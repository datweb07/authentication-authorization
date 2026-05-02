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

---

# Xác thực và Phân quyền dựa trên Token (Token-based Authentication)
Giải thích về cơ chế xác thực (Authentication) và phân quyền (Authorization) sử dụng **Token** trong việc xây dựng Web API, thay thế cho cơ chế Cookie/Session truyền thống của các ứng dụng web thông thường.

Nếu người dùng hoặc ứng dụng gọi một API yêu cầu xác thực mà không cung cấp Token hoặc Token không hợp lệ, máy chủ (server) mặc định sẽ trả về lỗi **401 (Unauthorized)**.

Dưới đây là các khái niệm và thành phần chính khi làm việc với Token trong hệ thống API:

### 1. Cách thức gửi Token qua HTTP Header
Khác với Cookie được trình duyệt tự động lưu và đính kèm, Token khi gọi Web API phải được đính kèm thủ công vào mỗi lời gọi (HTTP Request) lên máy chủ.
* Gửi thông qua header có tên là `Authorization`.
* **Cú pháp quy ước:** `Authorization: <Scheme> <Token_Value>`
* **Ví dụ thực tế phổ biến nhất:** `Authorization: Bearer eyJhbGciOi...` 
  * `Bearer` là từ khóa chỉ định việc sử dụng JSON Web Token.
  * Cấu trúc là: Từ khóa Bearer -> khoảng trắng -> chuỗi ký tự Token.
* Việc gọi API với Token có tính độc lập hoàn toàn (stateless). Server sẽ xác thực trực tiếp trên từng request mà không cần lưu trữ hay liên kết chúng thông qua session.

### 2. Cấu trúc của JSON Web Token (JWT)
JWT là một chuỗi ký tự được mã hóa dạng Base64, bao gồm 3 phần chính được ngăn cách bởi dấu chấm (`.`):
* **Header:** Chứa thông tin cơ bản để mô tả về Token (loại token, thuật toán dùng để ký).
* **Payload:** Chứa nội dung dữ liệu thực tế mang theo, mỗi trường dữ liệu bên trong được gọi là một **Claim** (cặp key-value). 
  * Các Claim thường chứa: ID người dùng, đơn vị cấp phát (Issuer), Quyền hạn (Roles), Thời gian hết hạn,...
  * *Lưu ý:* Việc phân quyền ứng dụng thông qua việc đọc các dữ liệu (Claim) có sẵn bên trong Payload này được gọi là **Claim-based Authorization**.
* **Signature (Chữ ký):** Phần này được dùng để server đối chiếu, nhằm xác định Token có thực sự được cấp phát bởi máy chủ gốc hay không, và đảm bảo dữ liệu ở phần Payload không bị kẻ gian giả mạo hoặc thay đổi.

### 3. Phân loại Token và Cơ chế cấp mới
Token bản chất là tự thân mang đầy đủ thông tin (self-contained). Khi server nhận được Token, nó không cần phải gọi lại cơ sở dữ liệu để tra cứu. Do đó, Token **bắt buộc phải có thời gian hết hạn** để ngăn chặn việc Token bị dùng vĩnh viễn trong trường hợp bị lộ.
* **Access Token:** 
  * Dùng để đính kèm vào các API request hàng ngày để lấy dữ liệu.
  * Thường có thời gian sống (lifespan) rất ngắn (ví dụ: 5 - 10 phút) để đảm bảo an toàn.
* **Refresh Token:**
  * Dùng để xin cấp lại một *Access Token* mới khi nó bị hết hạn, nhờ đó người dùng không phải khó chịu vì phải gõ username/password đăng nhập lại liên tục.
  * Có thời gian sống dài hơn rất nhiều (vài ngày, vài tuần hoặc cả tháng).
  * *Tính năng nâng cao:* Nhiều hệ thống áp dụng cơ chế **Rotating Refresh Token**. Nghĩa là mỗi lần dùng Refresh Token cũ để xin Access Token mới, server cũng sẽ cấp kèm một Refresh Token mới toanh và hủy bỏ token cũ để tăng cường bảo mật.

### 4. Giao thức OAuth 2.0 và OpenID Connect
Đây là các tiêu chuẩn (protocol) định hình cách các hệ thống cấp phát và xác thực Token an toàn giữa nhiều bên với nhau.
* **OAuth 2.0:** 
  * Là một giao thức ủy quyền. Nó cho phép một ứng dụng bên thứ ba (Ví dụ: Duolingo) được phép thay mặt người dùng truy cập vào tài nguyên trên một hệ thống khác (Ví dụ: Facebook) mà **không cần người dùng phải giao Username/Password** cho bên thứ ba.
  * OAuth 2.0 chỉ tập trung vào việc cấp quyền (Authorization) thông qua việc sinh ra *Access Token* và *Refresh Token*. Nó không quan tâm người dùng đó cụ thể là ai.
* **OpenID Connect (OIDC):**
  * Là một bộ tiêu chuẩn mở rộng được xây dựng đè lên trên nền tảng của OAuth 2.0.
  * Nó bổ sung thêm tính năng định danh người dùng (Authentication).
  * OIDC sinh ra thêm một loại token thứ 3 được gọi là **ID Token**. Token này chứa các thông tin hồ sơ cơ bản (họ tên, email, ảnh đại diện...) để ứng dụng bên thứ ba có thể đọc và biết chính xác người đang đăng nhập là ai.

---

# Giải thích luồng OAuth2 qua ví dụ thực tế

Ví dụ thực tế: **Bạn (User)** đang muốn đăng nhập vào **FPT Shop (Client)** thông qua trung gian quản lý là **Supabase (Auth)** liên kết với tài khoản **Google**.

### 1. Các thành phần tham gia (Actors)

* **User:** Chính là bạn đang ngồi trước màn hình máy tính/điện thoại.
* **Client:** Trình duyệt đang mở trang web **FPT Shop**.
* **Auth (Authorization server):** **Supabase**, đóng vai trò là hệ thống backend quản lý việc đăng nhập cho FPT Shop.
* **Google (Resource Server)**

### 2. Giải thích luồng hoạt động (Flow)

#### **Bước 1: Bắt đầu yêu cầu** *(Từ Client -> Auth)*
* **Thực tế:** Bạn vào trang web FPT Shop và bấm vào nút **"Đăng nhập bằng Google"**.
* **Hệ thống:** Trình duyệt FPT Shop (Client) gửi một yêu cầu đến máy chủ Supabase (Auth), báo rằng: *"Có khách hàng muốn đăng nhập, hãy chuẩn bị luồng xác thực qua Google giúp tôi"*.

#### **Bước 2: Yêu cầu người dùng ủy quyền** *(Từ Auth -> User)*
* **Thực tế:** Màn hình của bạn bị chuyển hướng (redirect) rời khỏi FPT Shop, chuyển sang trang đăng nhập của Google. Bạn sẽ thấy thông báo: *"FPT Shop muốn truy cập vào tên và địa chỉ email của bạn"*.
* **Hệ thống:** Supabase/Google hiển thị giao diện đăng nhập và màn hình xin phép (Consent screen) cho bạn.

#### **Bước 3: Người dùng đồng ý** *(Từ User -> Auth)*
* **Thực tế:** Bạn nhập tài khoản, mật khẩu Gmail và bấm nút **"Cho phép" (Allow)**.
* **Hệ thống:** Bạn gửi trực tiếp sự đồng ý của mình cho Google và Supabase. 
    > **Lưu ý quan trọng:** FPT Shop hoàn toàn không biết mật khẩu của bạn, họ chỉ đứng ngoài chờ.

#### **Bước 4: Trả mã xác nhận về cho Client** *(Từ Auth -> Client)*
* **Thực tế:** Màn hình load một chút, sau đó bạn được tự động chuyển hướng quay trở lại trang web FPT Shop.
* **Hệ thống:** Google xác nhận bạn hợp lệ, nó gửi cho Supabase một cái mã (**Authorization Code**). Supabase đính kèm mã này vào đường link và đẩy trình duyệt của bạn quay về lại FPT Shop.

#### **Bước 5: Đổi mã lấy "Chìa khóa"** *(Giữa Client ↔ Auth)*
* **Thực tế:** Bạn thấy màn hình FPT Shop đang xoay xoay chữ *"Đang xử lý đăng nhập..."*.
* **Hệ thống:** Ở dưới ngầm, FPT Shop lấy Mã xác nhận (Code) vừa nhận được, gọi API đến Supabase để đổi lấy một **Access Token** (Chìa khóa truy cập). Bước trao đổi ngầm này giúp hacker không lấy cắp được token thông qua trình duyệt.

#### **Bước 6: Lấy thông tin** *(Từ Client -> Google)*
* **Thực tế:** FPT Shop hiện dòng chữ *"Chào mừng Nguyễn Văn A"*, kèm theo avatar Google của bạn. Quá trình đăng nhập hoàn tất.
* **Hệ thống:** Khi đã cầm Access Token, FPT Shop (thông qua Supabase) có quyền gọi các API của Google để lấy thông tin hồ sơ (Tên, Email, Avatar...) theo đúng những gì bạn đã cho phép.
