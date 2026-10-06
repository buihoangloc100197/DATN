# NỀN TẢNG SAAS AI DỊCH THUẬT & KHỚP KHẨU HÌNH VIDEO (LIP-SYNC)
> **Đề tài Tốt nghiệp:** Nghiên cứu & Xây dựng Hệ thống SaaS AI All-in-One Tự động Dịch thuật Đa ngôn ngữ, Nhân bản Giọng nói và Khớp Khẩu hình Video.

---

## 📌 TỔNG QUAN HỆ THỐNG
Dự án cung cấp giải pháp toàn diện cho việc bản địa hóa video (Video Localization) tự động hóa thông qua chuỗi xử lý AI đa tầng:
1. **Frontend & Tầng BFF (Next.js 14 App Router):**
   - **React Server Components (RSC):** Fetch và render dữ liệu 100% tại Server, **triệt tiêu hoàn toàn API requests trên tab Network của F12 DevTools**.
   - **Hashed Server Actions:** Thao tác ghi dữ liệu không lộ REST URL công khai, ẩn danh tên hàm và payload schema.
   - **Bảo mật Chống Soi DevTools (Anti-F12):** Tắt Source Maps trên Production, nén & biến đổi mã nguồn (Mangle), mã hóa Payload 2 chiều (AES-256-GCM).
   - **Ký số Request HMAC (Anti-Replay):** Đính kèm `X-Signature`, `X-Timestamp` (hiệu lực 30s), và `X-Nonce` (Redis TTL 60s) nhằm chặn đứng 100% hành vi Copy as cURL từ F12 về Postman để chạy lại.
2. **Kiến trúc Mở rộng Tự động Liên kết Module (Plug-and-Play Extensibility):**
   - **Nguyên lý Mở - Đóng (Open-Closed Principle):** Khi sửa code hoặc thêm module mới (như tạo phụ đề hoạt hình, tự động chèn nhạc nền BGM, nhận diện cảm xúc), hệ thống **tự động liên kết module mới với module cũ mà không cần sửa một dòng code nào trong các module cũ**.
   - **Khung Hợp đồng Chuẩn (Canonical Data Model - CDM):** Mọi module kế thừa giao diện `IPipelineModule` và chia sẻ cấu trúc `CanonicalPipelineContext` bất biến.
   - **Kiến trúc Hướng Sự kiện (Event-Driven Architecture):** Các module tách rời 100% (Loose Coupling), giao tiếp thông qua Redis Event Bus.
   - **Cấu hình Pipeline Động qua Metadata CSDL (Metadata-Driven Pipeline):** Thứ tự và danh sách các module được điều khiển bằng bảng `pipeline_definitions` và `pipeline_step_configs` trên Microsoft SQL Server 2022. Thêm module mới chỉ cần thêm 1 dòng cấu hình trong CSDL.
   - **Adapter Tương thích Ngược & Fallback An toàn:** Tự động chuyển đổi định dạng dữ liệu $v1 \leftrightarrow v2$ và tự động chuyển về module cũ trong vòng 50ms nếu module mới gặp sự cố.
3. **Backend & Cơ sở dữ liệu Cốt lõi:**
   - **Database Engine:** **Microsoft SQL Server 2022 (MS SQL Server 2022)**, quản trị và kiểm soát qua **SSMS (SQL Server Management Studio)**.
   - **Bảo mật Mã nguồn T-SQL:** **100% Stored Procedures, Functions & Triggers được biên dịch với `WITH ENCRYPTION`**, mã hóa vĩnh viễn trong CSDL, chống đọc lén qua `sp_helptext` hay `sys.sql_modules`.
   - **Native Row-Level Security (RLS):** Lọc dữ liệu tự động tại tầng Engine thông qua `SESSION_CONTEXT` và Security Predicates.
   - **Transactions & Khóa Bi quan:** Sử dụng `WITH (UPDLOCK, ROWLOCK)` trong các Stored Procedure (`sp_create_project_secure`, `sp_complete_project_and_charge`) loại trừ triệt để lỗi tranh chấp Race Condition.
   - **Database Triggers Bảo vệ:** `trg_protect_quota_direct_tamper` (chặn đứng mọi hành vi sửa trực tiếp hạn mức Quota từ SQL Console), `trg_projects_status_audit` (tự động ghi log chuyển trạng thái dự án vào bảng kiểm toán).
4. **AI Pipeline & Xử lý Âm thanh / Hình ảnh:**
   - **Speech-to-Text (STT):** OpenAI Whisper Large-v3 (Trích xuất âm thanh, khử nhiễu FFmpeg `afftdn`, gắn timestamps từng từ).
   - **Translation:** GPT-4o / DeepL API (Dịch thuật bảo toàn ngữ cảnh và mốc thời gian phân đoạn).
   - **Voice Cloning:** Coqui XTTS-v2 / ElevenLabs (Tách giọng mẫu Demucs, tổng hợp âm thanh đa ngôn ngữ giữ nguyên sắc thái giọng gốc).
   - **Lip-sync AI:** Wav2Lip-HQ / MuseTalk kết hợp CodeFormer (Điều chỉnh cử động môi, phục hồi khuôn mặt nét cao 1080p).
5. **Hạ tầng & Điều phối:** Redis Cluster + BullMQ/Celery + Cloud GPU (RunPod/AWS EC2) hỗ trợ Auto-scaling Scale-to-Zero.

---

## 📂 HỒ SƠ TÀI LIỆU KỸ THUẬT & HƯỚNG DẪN AI AGENT
Hệ thống tài liệu hoàn chỉnh sẵn sàng cho cả Kỹ sư và **AI Coding Agent tự động thực thi (Autonomous Run)**:
* 📘 **[Đặc tả Kỹ thuật & Kiến trúc Hệ thống Chi tiết](file:///c:/Users/LHUser/Desktop/DATN/docs/SOFTWARE_REQUIREMENTS_AND_ARCHITECTURE_SPECIFICATION.md)**
* 🤖 **[Chỉ thị Thực thi Tự trị dành riêng cho AI Agent (Runtime Directive)](file:///c:/Users/LHUser/Desktop/DATN/docs/AI_AGENT_INSTRUCTIONS.md)**

Nội dung cốt lõi:
* **Kiến trúc 4 tầng Clean Architecture + Mô hình BFF**: Phân tách ranh giới Domain Entities, Use Cases, Interface Adapters, Frameworks/Drivers, cô lập hoàn toàn Client khỏi Database.
* **Kiến trúc Mở rộng Cắm-Rút (Plug-and-Play Extensibility)**: Thiết kế Event-Driven Pipeline, Canonical Data Model, Dynamic Pipeline Orchestrator, Schema Evolution và tự động fallback.
* **Chiến lược Phòng thủ Chống Lộ API trên F12 DevTools**: Chi tiết 4 tầng phòng vệ (RSC Zero Client Fetch, Hashed Server Actions, AES-256-GCM Payload Encryption, HMAC Anti-Replay Tokens).
* **Mã nguồn T-SQL SQL Server 2022 Toàn diện**:
  - DDL Tables (`users`, `refresh_tokens`, `pipeline_definitions`, `pipeline_step_configs`, `projects`, `project_audit_history`, `project_segments`, `job_logs`, `transactions`).
  - Native Row-Level Security Policy (`SESSION_CONTEXT`).
  - Audit & Protection Triggers (`trg_protect_quota_direct_tamper`, `trg_projects_status_audit`) có `WITH ENCRYPTION`.
  - Bộ 6 Stored Procedures nghiệp vụ có `WITH ENCRYPTION`, `TRANSACTION` và `UPDLOCK, ROWLOCK`.
* **Sổ tay Tự trị cho AI Agent (Playbook)**: 6 quy tắc bất biến (Guardrails), cây thư mục chuẩn hóa, lộ trình triển khai 6 giai đoạn, code mẫu mở rộng module và bộ lệnh CLI tự kiểm thử (Self-testing Loop).