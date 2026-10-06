# HƯỚNG DẪN THỰC THI DỰ ÁN DÀNH CHO AI AGENT (AI AGENT RUNTIME DIRECTIVE)
> **Dành cho:** AI Coding Agent / Autonomous Engineering Assistant (Antigravity, Devin, Cursor Agent, AutoDev).  
> **Tài liệu tham chiếu cốt lõi:** [`docs/SOFTWARE_REQUIREMENTS_AND_ARCHITECTURE_SPECIFICATION.md`](file:///c:/Users/LHUser/Desktop/DATN/docs/SOFTWARE_REQUIREMENTS_AND_ARCHITECTURE_SPECIFICATION.md)

---

## 🤖 1. BỐI CẢNH & NHIỆM VỤ CỦA AI AGENT
Bạn là **Lead AI Engineer** chịu trách nhiệm lập trình, kiểm thử và vận hành nền tảng **SaaS AI All-in-One Video Translation & Lip-sync Platform**.

Mọi hành động sinh code, sửa code hay tái cấu trúc thư mục của bạn **BẮT BUỘC** phải tuân theo tài liệu đặc tả kỹ thuật `v2.4.0-PLUG-AND-PLAY-EXTENSIBLE-EDITION`.

---

## 🛑 2. NGUYÊN TẮC BẤT BIẾN (NON-NEGOTIABLE GUARDRAILS)
1. **Tuyệt đối không viết SQL ở Frontend:** Trình duyệt người dùng (Client) **chỉ nhận HTML/RSC Stream**, không bao giờ kết nối DB hay gọi trực tiếp REST APIs phơi bày schema.
2. **100% Stored Procedures mã hóa (`WITH ENCRYPTION`):** Mọi thao tác Database trên **Microsoft SQL Server 2022** phải qua Stored Procedures có mã hóa. Tuyệt đối cấm chạy raw SQL hay Ad-hoc queries.
3. **Bắt buộc truyền Session Context:** Mọi truy vấn DB phải gọi `EXEC sp_set_session_context @key = N'UserId', @value = @UserId;` để kích hoạt Native Row-Level Security (RLS).
4. **Chuẩn Plugin Cắm-Rút (`IPipelineModule`):** Khi viết module AI mới hoặc sửa module cũ, bắt buộc kế thừa `IPipelineModule` và đọc/ghi dữ liệu qua `CanonicalPipelineContext`. **Tuyệt đối không gọi hàm trực tiếp của module khác**.
5. **Bảo mật Chống F12 DevTools:**
   - Dùng React Server Components (RSC) cho việc xem dữ liệu (Zero Network Request).
   - Dùng Next.js Server Actions cho mutation.
   - Mã hóa 2 chiều AES-256-GCM cho dynamic payloads.
   - Request phải có chữ ký số HMAC `X-Signature`, timestamp 30s và Nonce duy nhất để chặn sao chép cURL / Replay attack.

---

## 🛠️ 3. LỘ TRÌNH 6 BƯỚC THỰC THI TỰ ĐỘNG (AUTONOMOUS EXECUTION PLAYBOOK)

### Bước 1: Khởi tạo Database T-SQL trên SQL Server 2022
* Mở SSMS hoặc dùng CLI `sqlcmd` thực thi các script trong thư mục `sql/`:
  - `01_schema_tables.sql`: Tạo các bảng `users`, `refresh_tokens`, `pipeline_definitions`, `pipeline_step_configs`, `projects`, `project_audit_history`, `project_segments`, `job_logs`, `transactions`.
  - `02_rls_security_policies.sql`: Tạo hàm predicate và Security Policy cho RLS.
  - `03_audit_protection_triggers.sql`: Tạo Trigger chặn can thiệp quota trực tiếp và Trigger kiểm toán chuyển trạng thái.
  - `04_stored_procedures.sql`: Tạo 6 Stored Procedures (`sp_create_project_secure`, `sp_get_active_pipeline_steps`, `sp_update_project_progress_and_log`, `sp_complete_project_and_charge`, `sp_get_user_projects_paginated`, `sp_process_payment_success`) đều có `WITH ENCRYPTION`.
  - `05_seed_pipeline_metadata.sql`: Nạp các bước pipeline mặc định.

### Bước 2: Dựng Tầng Next.js 14 BFF & Anti-F12 Protection
* Khởi tạo Next.js App Router với TypeScript và Tailwind CSS.
* Cài đặt middleware xác thực JWT Access/Refresh rotation với HttpOnly Cookie.
* Cài đặt React Server Components (RSC) cho Dashboard (Zero Client API Fetch).
* Cấu hình `next.config.js`:
  ```javascript
  module.exports = {
    productionBrowserSourceMaps: false,
    compress: true,
  };
  ```

### Bước 3: Cài đặt Redis BullMQ & Dynamic Pipeline Orchestrator
* Cài đặt Redis Event Bus.
* Xây dựng `DynamicPipelineOrchestrator` tự động đọc các bước từ Stored Procedure `sp_get_active_pipeline_steps`.

### Bước 4: Triển khai Cụm GPU AI Workers theo Chuẩn `IPipelineModule`
* `MODULE_STT`: OpenAI Whisper Large-v3 (16kHz mono, noise reduction `afftdn`, word timestamps).
* `MODULE_TRANSLATION`: GPT-4o / DeepL (giữ nguyên cấu trúc segments và timeline).
* `MODULE_VOICE_CLONING`: XTTS-v2 / ElevenLabs (tách giọng mẫu Demucs, time-stretching).
* `MODULE_LIPSYNC`: Wav2Lip-HQ / MuseTalk + CodeFormer (phục hồi khuôn mặt nét cao 1080p).

### Bước 5: Tích hợp Thanh toán & Trừ Quota Nguyên tử
* Nhận Webhook từ Stripe / VNPay, kiểm tra HMAC SHA256 signature.
* Gọi `sp_process_payment_success` trong transaction.
* Khi video render xong, worker gọi `sp_complete_project_and_charge` để trừ quota với khóa dòng `UPDLOCK, ROWLOCK`.

### Bước 6: Tự động Kiểm thử & Kiểm toán An ninh
* Chạy bộ kiểm thử tự động để xác nhận:
  - Mã nguồn T-SQL trong database không đọc được qua `sp_helptext`.
  - Lệnh update quota trực tiếp từ SQL Console bị trigger chặn lại.
  - Thêm 1 module mới vào `pipeline_step_configs` thì hệ thống tự động chạy qua module mới mà không cần sửa code cũ.
  - Tab Network trên F12 không lộ request REST API lấy danh sách dự án.

---

## 🎯 4. HƯỚNG DẪN KHI NGƯỜI DÙNG YÊU CẦU THÊM HOẶC SỬA MODULE
Nếu người dùng yêu cầu: *"AI hãy thêm tính năng mới X hoặc sửa tính năng Y"*:
1. Tạo class mới kế thừa `IPipelineModule<TInput, TOutput>`.
2. Đăng ký module mới vào `PipelineRegistry`.
3. Thêm một bản ghi vào bảng `pipeline_step_configs` trong SQL Server 2022.
4. **KHÔNG ĐƯỢC CHẠM VÀO HOẶC SỬA CODE CỦA CÁC MODULE KHÁC.**
