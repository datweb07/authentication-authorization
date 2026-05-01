# Cookie, Session và Token-based authentication
Tập trung giải thích **bản chất cốt lõi dựa trên giao thức HTTP** về cách các hệ thống web xác thực và phân quyền người dùng. Đây là kiến thức nền tảng bắt buộc phải nắm vững trước khi code.

### 1. Phân biệt Authentication và Authorization
Rất nhiều lập trình viên mới hay nhầm lẫn hai khái niệm này:
*   **Authentication (Xác thực):** Trả lời câu hỏi *"Bạn là ai?"*. (Ví dụ: Hành động bạn nhập Username và Password để đăng nhập vào hệ thống).
*   **Authorization (Phân quyền/Ủy quyền):** Trả lời câu hỏi *"Bạn được phép làm gì?"*. (Ví dụ: Sau khi đăng nhập thành công, hệ thống kiểm tra xem bạn là Admin hay User thường, bạn có quyền xóa bài viết hay chỉ được xem).

### 2. Giao thức HTTP và vấn đề "Stateless"
*   Giao thức HTTP bản chất là **Stateless (Phi trạng thái)**. Nghĩa là server xử lý xong một request là sẽ "quên" luôn client đó là ai. 
*   Nếu không có cơ chế lưu trữ lại trạng thái, mỗi lần bạn bấm sang một trang mới trên website, hệ thống sẽ lại bắt bạn đăng nhập lại. Để giải quyết vấn đề này, người ta sinh ra **Cookie**, **Session** và **Token**.

### 3. Cơ chế Cookie và Session
**Cookie:**
*   Là một đoạn dữ liệu nhỏ (vài KB) được Server yêu cầu Client (Trình duyệt) lưu trữ lại ở dưới máy của người dùng.
*   Mỗi khi Client gửi một request mới lên Server, nó sẽ tự động đính kèm các Cookie này theo. Nhờ đó, Server biết được request này đến từ ai.

**Session:**
*   Lưu toàn bộ thông tin nhạy cảm ở phía Client (bằng Cookie) là rất nguy hiểm (dễ bị hack/sửa đổi). Do đó, người ta sinh ra Session.
*   **Cách hoạt động:** Khi bạn đăng nhập thành công, Server sẽ tạo ra một vùng nhớ chứa thông tin của bạn (gọi là Session) và sinh ra một **Session ID** (một chuỗi mã ngẫu nhiên). Server chỉ gửi cái **Session ID** này về cho Client để lưu vào **Cookie**.
*   Các lần request sau, Client chỉ gửi Session ID lên. Server lấy Session ID này dò trong bộ nhớ/database của mình để biết bạn là ai.

### 4. Token-based Authentication (Xác thực dựa trên Token)
*   **Nhược điểm của Session:** Khi hệ thống scale (mở rộng) lên nhiều Server, Server A lưu Session của bạn nhưng Server B thì không. Nếu request của bạn bị điều hướng sang Server B, bạn sẽ bị bắt đăng nhập lại (trừ khi dùng các giải pháp đồng bộ Session rất tốn kém).
*   **Giải pháp Token (thường dùng JWT - JSON Web Token):**
    *   Thay vì lưu trạng thái ở Server, sau khi đăng nhập thành công, Server sẽ gom thông tin của bạn (như ID, Role), **mã hóa** và **ký (sign)** bằng một khóa bí mật, tạo thành một chuỗi gọi là **Token**.
    *   Server gửi Token này cho Client lưu lại (thường lưu ở Local Storage hoặc Cookie).
    *   Lần sau gửi request, Client đính kèm Token này vào `Header` (thường là `Authorization: Bearer <token>`).
    *   Server không cần tìm trong bộ nhớ nữa, chỉ cần dùng khóa bí mật để giải mã và kiểm tra chữ ký của Token là biết bạn là ai và có quyền gì.
*   **Ưu điểm:** Cực kỳ phù hợp cho Web API, các ứng dụng Mobile, và hệ thống Microservices vì nó hoàn toàn "Stateless" (Server không cần tốn RAM để nhớ người dùng).
