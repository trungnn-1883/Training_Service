# Training_Service

## I. Khái niệm

### 1. Service là gì ?

Là 1 thành phần của Android, tạo bởi hệ thống có thể thực hiện các hoạt động lâu dài trong background và không cung cấp giao diện người dùng, có thể hoạt động khi ứng dụng bị hủy

Tương tác với service qua:

- Một thành phần khác start nó

- Một thành phần khác liên kết (bind) với nó

### 2. Có 3 loại service

- **Foreground service**: thực hiện một số thao tác người dùng **có thể chú ý**, thấy rõ ràng
Ví dụ như một ứng dụng chơi nhạc, download, ... có thể thực hiện nhiệm vụ và control nó
bằng foreground service. Foreground service phải hiển thị một **Notification** 

Chạy khi gọi **startForeGroundService()**

- **Background service**: thực hiện hành động người dùng **ko cần chú ý trực tiếp**. Từ Android 8.0, service chạy trong background bị giới hạn

- **Bound service**: cung cấp 1 giao diện Client - Server cho phép các thành phần tương tác với nó: gửi yêu cầu, nhận kết quả và thậm chí là IPC (inter-process communication) - giao tiếp qua nhiều tiến trình

<img src="https://images.viblo.asia/5210d9be-e4a0-430b-8e7f-dd37c47e0678.png">

### Sơ đồ hoạt động

<img src="https://o7planning.org/vi/10421/cache/images/i/1172852.png" witdh="450">







