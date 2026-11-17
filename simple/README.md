# Ultra miễn phí 1 năm · Bản tối giản

> [DISCLAIMER.md](../DISCLAIMER.md) — chỉ để đọc/tham khảo, không chạy trên tài khoản chính; nhớ hủy trước khi bị tính $99.99/tháng.

Một file, hai ba trăm dòng, làm đúng một việc: để Google One hiển thị offer dùng thử Ultra miễn phí 12 tháng, xác nhận 0 đồng, rồi kích hoạt.

Script không thay bạn khâu liên kết thẻ, cũng không đụng vào cài đặt tài khoản. Nó chỉ kéo cái offer mà giao diện không hiển thị ra.

## I. Chuẩn bị

**Điện thoại**: Android đã root, kết nối USB với máy tính. Magisk, KernelSU, APatch đều được, emulator cũng được.

**Máy tính**: Python 3.9 trở lên, cài một gói

```bash
pip3 install frida
```

**adb**: platform-tools trong Android SDK. Cài xong chạy `adb devices`, thấy một dãy số thiết bị là đúng. Không thấy thì mở «Tùy chọn nhà phát triển → Gỡ lỗi USB» trên điện thoại, cắm vào rồi nhấn Cho phép.

**frida-server trên điện thoại**: không nằm trong script, phải tự đưa lên một lần.

```bash
# server trùng phiên bản frida trên máy tính
curl -LO https://github.com/frida/frida/releases/download/16.5.9/frida-server-16.5.9-android-arm64.xz
xz -d frida-server-16.5.9-android-arm64.xz
adb push frida-server-16.5.9-android-arm64 /data/local/tmp/frida-server
adb shell "su -c 'chmod 755 /data/local/tmp/frida-server'"
adb shell "su -c '/data/local/tmp/frida-server &'"
```

Đổi số phiên bản thành đúng cái `frida --version` trên máy bạn, lệch là không kết nối được.

## II. Kiểm tra môi trường trước

```bash
python3 ultra.py --check
```

Script sẽ lần lượt kiểm tra: adb, thiết bị, root, proxy, Clash, frida-server, module billing và phiên bản frida.

Tất cả xanh thì tiếp tục. Dòng đỏ sẽ nói ngay cái gì đang thiếu.

## III. Làm hai việc trên điện thoại

Script không làm thay, vì làm tay hai bước này chắc chắn hơn tự động.

**1. Thêm tài khoản và chuyển sang tài khoản đó trong Play**

Cài đặt → Tài khoản → Thêm tài khoản → Google. Xong mở Play Store, nhấn avatar góc phải, chuyển sang tài khoản vừa thêm.

**2. Liên kết một thẻ**

Play → Avatar → Thanh toán và đăng ký → Phương thức thanh toán → Thêm thẻ tín dụng.

Đăng ký bắt buộc phải có phương thức thanh toán, không có thẻ thì Play sẽ từ chối. Mỗi điện thoại chỉ cần liên kết một lần, các tài khoản sau dùng chung thẻ đó.

## IV. Chạy

```bash
python3 ultra.py
```

Script sẽ làm bốn việc:

1. Liệt kê mọi tài khoản Google trên điện thoại, dừng lại chờ bạn nhấn Enter. **Nhìn kỹ, nếu không phải tài khoản muốn mở thì Ctrl+C, vào Play góc phải chuyển tài khoản rồi chạy lại**
2. Khởi chạy Google One và tiêm một module nhỏ vào
3. Truy vấn offer `g1.play.20tb`, chọn cái có thời gian dùng thử dài nhất
4. Mở trang thanh toán Play và theo dõi số tiền

Ở bước bốn sẽ dừng lại, trang xác nhận đăng ký của Play hiện lên. Script tự xử lý trong hai trường hợp:

- Thấy «Dùng thử miễn phí 12 tháng» hoặc `$0` → nhấn Đăng ký
- Thấy «Bạn không đủ điều kiện dùng thử miễn phí» → đóng lại, không nhấn gì cả

Đôi khi Play yêu cầu nhập lại mật khẩu. Chạy kèm mật khẩu:

```bash
python3 ultra.py --password «redacted-secret»
```

Không muốn nhấn Enter mỗi lần thì thêm `--yes`:

```bash
python3 ultra.py --yes
```

## V. Xác nhận kết quả

Sau khi mở xong, script tự mở trang đăng ký của Play.

Kết quả bình thường trông như thế này:

```
Google One
Sắp thanh toán    Ngày bắt đầu: 17/11/2027
            US$99.99/tháng
Phương thức thanh toán chính Mastercard-7558
```

Sang năm mới tính tiền nghĩa là 12 tháng này hoàn toàn miễn phí. Trang chủ Google One App cũng sẽ hiện «Gói hiện tại: Google AI Ultra, 20 TB».

## Câu hỏi thường gặp

**Báo không có root**

Script cần tiêm code vào tiến trình ứng dụng, không có root thì không làm được.

**Cứ đọc được tài khoản rỗng**

Play chưa tải xong. Mở Play bằng tay một lần, đợi trang chủ hiện ra rồi mới chạy.

**Trang thanh toán hiện rồi tắt ngay**

Chạy lại. Hai lần liên tiếp đều không được thì kiểm tra `adb shell settings get global http_proxy`, nếu không phải `null` thì xóa. Các VPN kiểu Clash sẽ đổi DNS, Play không kết nối được là ra thế này.

**Báo «không đủ điều kiện dùng thử miễn phí»**

Tài khoản này không đủ điều kiện, đổi tài khoản khác. Mỗi tài khoản cơ bản chỉ có một lần: dù bạn không xác nhận, offer cũng biến mất ngay khi dùng thử được khởi tạo. Vì đừng thử bằng tài khoản chính.

**Trang thanh toán hiện US$99.99**

Nghĩa là offer không được chọn, script sẽ tự bỏ cuộc, không trừ tiền.

## Sau khi mở xong

Trước hạn hãy vào Play hủy đăng ký, nếu không sẽ bị trừ $99.99. Đường dẫn: Play → Avatar → Thanh toán và đăng ký → Đăng ký → Google One → Hủy đăng ký.

Hủy không làm mất thời gian đã dùng, Ultra vẫn hoạt động đến hết thời hạn dùng thử.
