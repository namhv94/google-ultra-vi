# google-ultra

Google One có ưu đãi dùng thử AI Ultra miễn phí 12 tháng, không hiển thị trên bất kỳ trang nào, chỉ tồn tại trong giá trị trả về của thư viện thanh toán Play (BillingClient). Script này gộp chuỗi «kiểm tra offer → mở trang thanh toán Play → xác nhận 0 đồng → kích hoạt» thành một lệnh duy nhất.

[DISCLAIMER.md](DISCLAIMER.md)

⚠️ **Cảnh báo:** Không chạy script trên tài khoản chính — rủi ro vi phạm ToS và bị khóa tài khoản; sau 12 tháng sẽ bị thu $99.99/tháng nếu không hủy.

Nguyên lý là mượn chính tiến trình của Google One để gọi BillingClient, không cần Xposed, không cần sửa APK.

## Hai phiên bản

**Phiên bản đơn giản**: một trang web, mở bằng cách double-click `web/index.html`, đăng nhập → liên kết thẻ → bấm một cái là xong. Không cần cài Python, không cần root, không cần cáp dữ liệu.

**Phiên bản script**: `ultra0.py`, cần một điện thoại đã root và frida. Ưu điểm là lấy offer từ client thật, không phụ thuộc vân tay trình duyệt, chạy được hàng loạt.

Thử phiên bản đơn giản trước, không được mới dùng script.

## Nhận được gì

Nhận được **Ultra 5x, 20TB, 12 tháng $0**, hủy trước khi hết hạn thì không phải trả tiền. Sau khi kích hoạt, trang đăng ký của Play sẽ ghi «sẽ thu 2027-11-17 US$99.99/tháng» — đó là giá sau khi dùng thử kết thúc, không phải thu ngay.

Bậc 30TB (20x) cũng có thể tìm thấy offer, nhưng Play sẽ kiểm tra lại lúc thanh toán, hầu hết tài khoản sẽ hiện «bạn không đủ điều kiện dùng thử miễn phí». Script này sẽ không bấm vào đó.

## Cần gì

Một điện thoại Android đã root, kết nối USB với máy tính. Magisk, KernelSU, APatch đều được, dùng máy ảo cũng được.

Trên máy tính cần Python 3.9 trở lên, cộng thêm hai gói:

```bash
pip3 install frida
```

adb dùng platform-tools trong Android SDK. Sau khi cài, `adb devices` thấy điện thoại là được. Nếu chưa cài: trên Mac dùng `brew install android-platform-tools`, trên Windows tải gói nén platform-tools, giải nén rồi thêm thư mục vào PATH.

Trên điện thoại bật «Tùy chọn nhà phát triển → Gỡ lỗi USB», cắm vào rồi chọn Cho phép trong hộp thoại hiện lên.

## Cách dùng

### Dùng phiên bản đơn giản

Mã nguồn ở `web/index.html`, một file duy nhất, tải về rồi double-click là dùng được, trình duyệt sẽ mở một trang:

1. Bấm «Đi đăng nhập», đăng nhập bằng tài khoản Google cần kích hoạt
2. Bấm «Đi liên kết thẻ», liên kết một thẻ, để trong thẻ 1 đô la là được, dùng thử không trừ tiền
3. Quay lại trang, bấm vào thẻ «Ultra 20 TB»

Sau khi chuyển đến trang thanh toán của Google, xác nhận số tiền là 0 rồi bấm đăng ký.

### Dùng phiên bản script

Trước tiên tự kiểm tra môi trường: script sẽ kiểm tra thiết bị, root, frida-server, đẩy module thanh toán, đồng thời tắt luôn các proxy và Clash chặn mạng Play:

```bash
python3 ultra0.py --check
```

Tất cả xanh là có thể chạy tài khoản.

Ghi tài khoản vào một file text, mỗi dòng một tài khoản, các trường cách nhau bằng bốn dấu gạch ngang:

```
email----mật khẩu----email phụ----khóa 2FA----số thẻ----hạn dùng----CVC----tên chủ thẻ----mã bưu điện
```

Các mục sau có thể để trống. Tài khoản mới mua thường không có 2FA, cứ để trống:

```
abc123@gmail.com----mypassword----
```

Nếu có thẻ:

```
abc123@gmail.com----mypassword--------4242424242424242----10/29----326----ZHANG SAN----97201
```

Nếu không điền địa chỉ, mặc định dùng `123 Main St, Portland, OR 97201`, Oregon không thu thuế tiêu thụ.

Chạy:

```bash
python3 ultra0.py accounts.txt
```

Cũng có thể chạy một tài khoản:

```bash
python3 ultra0.py --account "abc@gmail.com:mật khẩu:khóa 2FA"
```

Để xem kết quả, mở trang đăng ký để xác nhận:

```
Play Store → Avatar → Thanh toán và đăng ký → Đăng ký
```

## Bên trong script làm gì

1. Thêm tài khoản vào điện thoại (chưa có thì thêm, có rồi thì bỏ qua)
2. Chuyển tài khoản Play. Sau khi chuyển luôn đọc lại từ bảng avatar để đối chiếu, nếu không khớp thì thoát ngay, không chạy nhầm sang tài khoản khác
3. Khởi động Google One, dùng Frida gắn vào, `DexClassLoader` tải module thanh toán
4. `queryProductDetailsAsync` truy vấn `g1.play.20tb`, lấy về 5 offer
5. Chọn offer có thời gian dùng thử dài nhất (`eft-12month`, P1Y), gọi `launchBillingFlow` trên luồng chính
6. Theo dõi trang thanh toán. Chỉ bấm đăng ký khi thấy giá là 0 hoặc «dùng thử miễn phí 12 tháng»; nếu thấy «không đủ điều kiện dùng thử miễn phí» thì đóng ngay, tuyệt đối không bấm
7. Play đôi khi yêu cầu nhập lại mật khẩu, script tự điền
8. Mở trang đăng ký để đối chiếu, ghi kết quả vào `ultra0-report/`

## Khi thất bại thì sao

**Tài khoản không đủ điều kiện.** Play sẽ báo «bạn không đủ điều kiện dùng thử miễn phí, tuy nhiên bạn vẫn có thể đăng ký». Mỗi tài khoản về cơ bản chỉ có một lần: chỉ cần bắt đầu dùng thử, dù chưa xác nhận, offer cũng biến mất. Vì vậy đừng thử trên tài khoản chính.

**Không có phương thức thanh toán.** Đăng ký bắt buộc phải có thẻ. Trang thêm thẻ của Play là WebView, không điền được bằng adb, script sẽ dừng lại và nhắc bạn điền trên điện thoại rồi nhấn Enter. Mỗi thiết bị chỉ cần liên kết thủ công một lần, sau đó mọi tài khoản dùng chung thẻ này.

**Chưa chuyển được tài khoản Play.** Script sẽ in ra tài khoản đang đăng nhập thực tế rồi thoát. Lúc này hãy tự chuyển trong Play, rồi thêm `--no-switch` và chạy lại để script chỉ kiểm tra, không cố chuyển nữa:

```bash
python3 ultra0.py accounts.txt --no-switch
```

**Không hiện trang thanh toán.** Chạy lại một lần. Nếu hai lần liên tiếp vẫn không hiện, kiểm tra mạng của thiết bị: `settings get global http_proxy`, nếu không phải `null` thì xóa; các VPN như Clash chiếm quyền DNS cũng khiến Play không kết nối được.

## Một số tham số

```bash
--plan 5x      # chỉ kích hoạt 20TB
--plan 20x     # chỉ truy vấn 30TB (đa số tài khoản không lấy được dùng thử)
--plan auto    # mặc định, truy vấn cả hai bậc, chọn thời gian dùng thử dài nhất
--no-add       # tài khoản đã có trên điện thoại, bỏ qua bước thêm tài khoản
--no-switch    # không tự chuyển tài khoản, chỉ kiểm tra tài khoản hiện tại
```

## Một vài lưu ý

Sau khi dùng thử nhớ vào Play hủy đăng ký, nếu không khi hết hạn sẽ bị trừ $99.99.

Script chỉ chạy cục bộ, mật khẩu tài khoản và số thẻ không kết nối mạng, cũng không tải lên đâu. File báo cáo nằm ở `ultra0-report/`, bên trong có chi tiết offer và kết quả từng bước.

Đoạn code này chỉ khai thác việc «client có thể lấy được offer dùng thử» trong quy trình thanh toán của Play, không sửa đổi bất kỳ dữ liệu phía server nào.

## Cấu trúc kho

```
web/index.html       phiên bản đơn giản: trang web một file
simple/ultra.py      bản tối giản của script (kèm module nhúng)
simple/README.md     hướng dẫn bản script
ultra0.py            bản đầy đủ của script (hàng loạt, chọn bậc, chuyển tài khoản)
```

## Giấy phép

MIT, xem [LICENSE](LICENSE).
