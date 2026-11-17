# Tuyên bố miễn trừ trách nhiệm (Disclaimer)

Bản dịch/fork này **chỉ nhằm mục đích nghiên cứu**. **KHÔNG chạy bất kỳ script nào trong repo này trên máy thật** cũng như trên thiết bị có dữ liệu hoặc tài khoản thật.

## Cảnh báo rủi ro

- **APK unsigned được nhúng trong repo gốc.** Repo gốc nhúng một APK chưa ký với `classes.dex` dung lượng **105971 bytes**, SHA256 `38a1cd132f10b31c3f4ec85d341764c702148175e2db7e480a288468b39b7b74`, sau đó inject mã này vào tiến trình Google One bằng quyền root. Đây là **rủi ro bảo mật không thể kiểm chứng/audit được** — bạn không thể xác minh nội dung thực thi bên trong.
- **Lộ thông tin nhạy cảm.** Tệp `accounts.txt` chứa plaintext mật khẩu, mã 2FA, số thẻ và CVC. Tuyệt đối không mở, sao chép hay chia sẻ tệp này với dữ liệu thật.
- **Vi phạm Điều khoản dịch vụ.** Việc tự động hóa các offer dùng thử có thể **vi phạm Điều khoản Google** và dẫn đến **khóa tài khoản** hoặc **từ chối thanh toán**.
- **Chi phí sau dùng thử.** Sau khi hết hạn dùng thử, gói có giá **US$99.99/tháng** — bạn **phải hủy trước hạn** nếu không muốn bị tính phí.

## Nguồn gốc

Đây là **fork private của `gylive/google-ultra`**, được dịch sang tiếng Việt. **Giữ nguyên code gốc** — bản fork chỉ thay đổi nội dung trình bày/tài liệu, không chỉnh sửa logic của mã nguồn.

Người dùng tự chịu trách nhiệm hoàn toàn về bất kỳ hậu quả nào khi sử dụng repo này.
