Nguyên nhân sâu xa của sự cố văng phiên đăng nhập (Session Conflict) Dưới góc độ cấu trúc dữ liệu trình duyệt: Tất cả các trình duyệt hiện đại như Google Chrome quản lý dữ liệu người dùng thông qua Thư mục dữ liệu người dùng (User Data Directory). Khi chỉ sử dụng 1 Profile mặc định (Default Profile), toàn bộ thông tin phiên làm việc bao gồm Session Cookies, Local Storage, IndexedDB, và Cache của tất cả các tab đều được ghi chung vào cùng một tập hợp tệp cơ sở dữ liệu SQLite Cookies.

Cơ chế gây ra lỗi (Mechanism of Conflict): Khi Bác sĩ đăng nhập hệ thống CarePulse, trình duyệt lưu một Session_Cookie xác thực định danh cho phiên làm việc của bác sĩ. Khi thực tập sinh mở tab mới để truy cập Cổng học tập PTIT, GitHub hoặc ứng dụng khác có chung cơ chế quản lý phiên hoặc chia sẻ hệ thống xác thực SSO (Single Sign-On), các request mới sẽ cập nhật, ghi đè (Overwrite) hoặc làm mới (Refresh/Invalidate) Cookie xác thực chung trong tệp Cookies đó. Hệ thống CarePulse nhận diện Cookie xác thực cũ đã bị thay đổi hoặc không hợp lệ, dẫn đến việc kích hoạt cơ chế bảo mật tự động Logout (Văng đăng xuất) ngay lập tức để bảo vệ tài khoản, làm gián đoạn quá trình kê đơn cho bệnh nhân.


Thiết kế Mô hình 3 Profile Chuyên dụng cho Máy trạm CarePulse

Profile 1: Work (Chuyên môn) Tên Profile: CarePulse_Clinical Mục đích sử dụng: Dành riêng cho Bác sĩ và Điều dưỡng trực: Kê đơn, xem bệnh án, tra cứu y tế, hội chẩn. Theme thị giác: Xanh Dương y tế (Medical Blue) - Phân biệt rõ môi trường làm việc chính thức. Dữ liệu lưu trữ độc lập: Cookies: Lưu token đăng nhập CarePulse, Cổng BHYT. History: Chỉ chứa lịch sử y tế. Bookmarks: Cây dấu trang y tế 01, 02, 03. Extensions: Tiện ích y khoa, đọc file DICOM/PDF.

Profile 2: Personal (Cá nhân) Tên Profile: Personal_Staff Mục đích sử dụng: Dành cho nhân viên giải trí nhẹ, tra cứu cá nhân trong giờ nghỉ (Email cá nhân, tin tức, Shopee). Theme thị giác: Xanh Lá cây (Sage Green) - Tạo cảm giác thư giãn, dễ nhận biết là vùng cá nhân. Dữ liệu lưu trữ độc lập: Cookies: Lưu tài khoản cá nhân, mạng xã hội. History: Lịch sử tra cứu cá nhân. Bookmarks: Trang cá nhân. Extensions: Chặn quảng cáo, dịch thuật cá nhân.

Profile 3: Sandbox/Test (Thực tập & Kiểm thử) Tên Profile: Sandbox_Trainee Mục đích sử dụng: Dành cho sinh viên thực tập, kỹ thuật viên chạy thử nghiệm tính năng mới của phần mềm CarePulse. Theme thị giác: Vàng Cam cảnh báo (Amber/Orange) - Nhắc nhở đây là môi trường test/thử nghiệm. Dữ liệu lưu trữ độc lập: Cookies: Xóa định kỳ, chứa token tài khoản test Admin/User. History: Lịch sử truy cập staging/test. Bookmarks: Link môi trường Test/Staging. Extensions: DevTools hỗ trợ test.

PHẦN 2: THIẾT KẾ HỆ THỐNG BOOKMARK VÀ QUY TRÌNH AN TOÀN


Cấu trúc Cây Thư mục Bookmark Chuẩn (Ctrl + Shift + O) Hệ thống Bookmark được sắp xếp theo đúng 3 thư mục chuẩn nghiệp vụ tại thanh dấu trang:

Thư mục 1: 01_CarePulse_Clinic (Phục vụ tác nghiệp lâm sàng & quản lý bệnh nhân) Hệ thống Quản lý Bệnh viện CarePulse HIS (carepulse.internal/login) Cổng Tra cứu Thông tin Thẻ BHYT Quốc gia (baohiemxahoi.gov.vn) Hệ thống Đơn thuốc Quốc gia & Kê đơn điện tử (donthuocquocgia.vn)

Thư mục 2: 02_Pharma_Portal (Phục vụ tra cứu thuốc, tương tác thuốc & dược lâm sàng) Tra cứu Dược thư Quốc gia Việt Nam (duocthu.gov.vn) Cơ sở dữ liệu Tương tác thuốc Drugs.com (drugs.com/drug-interactions) Ngân hàng dữ liệu Dược Bộ Y tế - DrugBank Việt Nam (drugbank.vn)

Thư mục 3: 03_Medical_Tools (Phục vụ công cụ tính toán & hội chẩn y khoa) Công cụ Tính toán Liều lượng & Chỉ số Y khoa MedCalc (medcalc.com) Cổng Đọc & Xem Ảnh X-quang/CT DICOM Trực tuyến (dicomviewer.net) Trang tin Y học Lâm sàng & Hướng dẫn Điều trị UpToDate (uptodate.com)


Kỹ thuật Favicon-Only tiết kiệm diện tích thanh Bookmark gấp 3 lần

Rào cản ban đầu: Nếu để cả tên đầy đủ của trang web (ví dụ: Hệ thống Quản lý Bệnh viện CarePulse HIS), thanh Bookmark chỉ chứa được tối đa 5 đến 7 liên kết là bị đầy và tràn vào menu phụ.

Hướng dẫn thực hiện mẹo Favicon-Only: Bước 1: Mở Trình quản lý dấu trang bằng tổ hợp phím Ctrl + Shift + O (hoặc chuột phải vào thanh Bookmark). Bước 2: Tìm đến 4 trang web dùng hàng ngày bao gồm CarePulse, Cổng BHYT, Dược thư Quốc gia, Google Dịch. Bước 3: Nhấp chuột phải vào từng trang web và chọn Edit (Chỉnh sửa). Bước 4: Tại ô Name (Tên), xóa sạch toàn bộ chữ, để trống hoàn toàn và nhấn Save (Lưu).

Kết quả đạt được: Thanh công cụ lúc này chỉ hiển thị duy nhất Biểu tượng Logo (Favicon) đại diện của trang web đó như Biểu tượng Chữ thập y tế của CarePulse, Logo BHYT, Logo Google Dịch. Chiều rộng hiển thị của mỗi trang thu hẹp từ khoảng 200px xuống còn đúng 24px. Giúp không gian thanh Bookmark chứa được hơn 30 liên kết quan trọng ngay trên một màn hình mà không bao giờ bị tràn.


Xử lý Bẫy an ninh & Quy tắc quản trị Cloud Sync

Kịch bản đoàn thanh tra / khách mượn máy 30 phút: Chế độ kích hoạt BẮT BUỘC: Guest Profile (Hồ sơ Khách). Lý do lựa chọn Guest Profile thay vì Incognito (Vô danh): Incognito (Vô danh) vẫn nằm trong context của Profile hiện tại. Người mượn máy vẫn có thể nhìn thấy, truy cập và sử dụng toàn bộ Bookmarks, Extensions đã cài, thậm chí có thể mở lại các tab vừa đóng hoặc truy cập vào dữ liệu autofill nếu không cẩn thận. Guest Profile (Hồ sơ Khách) khởi tạo một môi trường duyệt web trắng tinh hoàn toàn độc lập (Clean State). Người mượn máy không thể nhìn thấy bất kỳ Bookmark, Lịch sử duyệt web, Mật khẩu đã lưu hay Cookie nào của phòng khám. Khi đóng cửa sổ Guest, toàn bộ dữ liệu duyệt web tạm thời của khách sẽ bị xóa sạch 100%.

Cảnh báo nguy cơ khi bật Cloud Sync tài khoản cá nhân trên máy trạm y tế: Nguy cơ rò rỉ dữ liệu bệnh nhân (HIPAA / Bảo mật thông tin Y tế): Nếu bác sĩ đăng nhập tài khoản Google cá nhân và bật Cloud Sync, toàn bộ dữ liệu lịch sử tra cứu bệnh án, thông tin bệnh nhân, Bookmark y tế sẽ tự động đồng bộ về máy tính cá nhân hoặc điện thoại cá nhân của bác sĩ đó, vi phạm nghiêm trọng quy định bảo mật dữ liệu y tế. Nguy cơ rò rỉ dữ liệu riêng tư ra phòng khám: Ngược lại, toàn bộ Mật khẩu đã lưu ở nhà, Lịch sử tìm kiếm cá nhân (mua sắm, giải trí, mạng xã hội) từ điện thoại cá nhân sẽ bị đồng bộ ngược lại máy trạm phòng khám. Khi hội chẩn hoặc dùng máy chiếu, các gợi ý cá nhân này sẽ tự động hiện lên trước mặt đồng nghiệp và đoàn thanh tra, gây mất an toàn thông tin và lộ lọt riêng tư. Quy tắc vàng: TUYỆT ĐỐI KHÔNG bật Cloud Sync tài khoản cá nhân trên Profile công việc (CarePulse_Clinical) của máy tính trạm dùng chung.
