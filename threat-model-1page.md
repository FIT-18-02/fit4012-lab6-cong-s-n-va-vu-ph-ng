# Threat Model - Lab 6 AES-CBC Socket

## Thông tin nhóm

- Thành viên 1: NGUYỄN VŨ PHƯƠNG
- Thành viên 2: TẠ CÔNG SƠN

## Assets

NGUYỄN VŨ PHƯƠNG : Liệt kê tài sản cần bảo vệ, ví dụ plaintext, AES key, IV, ciphertext, file đầu vào, file đầu ra và log.

Các tài sản quan trọng cần được bảo vệ trong hệ thống gồm:
Plaintext: dữ liệu gốc trước khi mã hóa, có thể chứa thông tin nhạy cảm.
AES Key: khóa bí mật dùng để mã hóa và giải mã dữ liệu.
IV (Initialization Vector): vector khởi tạo cho AES-CBC.
Ciphertext: dữ liệu sau khi mã hóa được gửi qua mạng.
Input file: file chứa dữ liệu đầu vào trước khi mã hóa.
Output file: file chứa dữ liệu sau khi giải mã.
Log files: file log lưu trạng thái hoạt động, lỗi hoặc dữ liệu debug của hệ thống.
Socket connection: kết nối TCP giữa Sender và Receiver.

## Attacker model

TẠ CÔNG SƠN : Mô tả đối tượng tấn công có thể nghe lén mạng LAN, bắt gói tin, sửa ciphertext, replay packet hoặc đọc log.

Đối tượng tấn công được giả định có khả năng:

Nghe lén lưu lượng mạng trong cùng LAN hoặc môi trường nội bộ.
Bắt và phân tích packet TCP giữa Sender và Receiver.
Chỉnh sửa ciphertext trong quá trình truyền.
Gửi lại packet cũ để thực hiện replay attack.
Truy cập file log nếu hệ thống lưu log không an toàn.
Giả mạo Sender để gửi dữ liệu không hợp lệ đến Receiver.

Attacker không cần quyền truy cập trực tiếp vào source code nhưng có khả năng tương tác với lưu lượng mạng.
## Threats

NGUYỄN VŨ PHƯƠNG : Nêu ít nhất 3 mối đe dọa cụ thể, ví dụ:
- Key disclosure do key/IV gửi plaintext.
- Tampering do ciphertext bị sửa.
- Replay attack do packet cũ bị gửi lại.
- Log leakage do key bị ghi vào log.
- No authentication do Receiver không xác thực Sender.

1. Key disclosure

AES key và IV được gửi qua key channel ở dạng plaintext. Nếu attacker nghe lén được packet, họ có thể lấy key và giải mã toàn bộ ciphertext.

2. Ciphertext tampering

Attacker có thể sửa đổi một phần ciphertext trong quá trình truyền. Điều này có thể làm dữ liệu giải mã bị sai hoặc gây lỗi padding.

3. Replay attack

Do hệ thống chưa sử dụng nonce hoặc timestamp, attacker có thể gửi lại packet cũ để Receiver xử lý lại dữ liệu trước đó.

4. Log leakage

Nếu key, IV hoặc plaintext bị ghi vào file log, attacker có thể đọc log để lấy thông tin nhạy cảm.

5. No authentication

Receiver chưa xác thực danh tính Sender, vì vậy attacker có thể giả mạo Sender và gửi dữ liệu độc hại.
## Mitigations

TẠ CÔNG SƠN : Nêu ít nhất 3 biện pháp giảm thiểu, ví dụ:
- Không gửi key plaintext trong hệ thống thật.
- Dùng TLS hoặc cơ chế trao đổi khóa an toàn.
- Dùng AES-GCM để có xác thực dữ liệu.
- Không ghi key thật vào log trong môi trường thật.
- Thêm nonce/timestamp để giảm replay.
- Thêm xác thực Sender.

1. Sử dụng cơ chế trao đổi khóa an toàn

Trong hệ thống thực tế cần dùng TLS, RSA hoặc Diffie-Hellman để trao đổi khóa thay vì gửi plaintext key qua mạng.

2. Sử dụng AES-GCM hoặc thêm MAC

AES-CBC chỉ bảo mật nội dung nhưng không kiểm tra tính toàn vẹn. Có thể dùng AES-GCM hoặc HMAC để phát hiện dữ liệu bị chỉnh sửa.

3. Không ghi key vào log

Trong môi trường thực tế không nên ghi AES key hoặc plaintext vào file log để tránh rò rỉ thông tin.

4. Thêm nonce hoặc timestamp

Mỗi packet nên có nonce hoặc timestamp để giảm nguy cơ replay attack.

5. Xác thực Sender

Có thể bổ sung chữ ký số, token hoặc cơ chế xác thực client để đảm bảo Receiver chỉ nhận dữ liệu từ nguồn hợp lệ.
## Residual risks

NGUYỄN VŨ PHƯƠNG : Nêu ít nhất 1 rủi ro còn lại, ví dụ hệ thống vẫn chưa an toàn vì key channel chỉ là mô phỏng, chưa có TLS, chưa có xác thực và chưa chống replay đầy đủ.

Mặc dù đã áp dụng một số biện pháp giảm thiểu, hệ thống vẫn còn nhiều rủi ro do đây chỉ là mô hình học tập. Key channel chưa được bảo vệ bằng TLS, chưa có cơ chế xác thực đầy đủ và chưa chống replay hoàn chỉnh. Vì vậy hệ thống chưa đủ an toàn để triển khai trong môi trường thực tế hoặc trên Internet công cộng.