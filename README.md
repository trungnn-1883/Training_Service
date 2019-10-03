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
bằng foreground service. Foreground service phải hiển thị một **Notification**. Chạy khi gọi **startForeGroundService()**

- **Background service**: thực hiện hành động người dùng **ko cần chú ý trực tiếp**. Từ Android 8.0, service chạy trong background bị giới hạn

- **Bound service**: cung cấp 1 giao diện Client - Server cho phép các thành phần tương tác với nó: gửi yêu cầu, nhận kết quả và thậm chí là IPC (inter-process communication) - giao tiếp qua nhiều tiến trình

<img src="https://images.viblo.asia/5210d9be-e4a0-430b-8e7f-dd37c47e0678.png" width="450">

### 3. Sơ đồ hoạt động


<img src="https://o7planning.org/vi/10421/cache/images/i/1172852.png" width="450">


### 4. Unbounded service (foreground + background service)

### a. Sử dụng

Để chạy một tác vụ lâu dài, có thể ko cần ràng buộc với thành phần giao diện kể cả khi thành phần đó bị hủy. ở Android 8.0 đang giới hạn cho service chạy nền, nhưng việc sử dụng vẫn tương tự

### b. Vòng đời

<img src="https://o7planning.org/vi/10421/cache/images/i/1174191.png" width="350">

Gọi bằng startService() hoặc startForeGroundService() - sử dụng bắt đầu ở Android 8.0

Vòng đời startService -> onCreate -> onStartCommand

Sau khi đã startService rồi thì khi gọi lại sẽ vào onStartCommand

Có các cờ trả về của hàm onStartCommand:

- **START_STICKY** bảo với hệ điều hành rằng cần tạo lại service khi có đủ bộ nhớ và call lại onStartCommand() với intent là null

- **START_NOT_STICKY**: chỉ áp dụng khi Service bị huỷ khi chưa thực hiện xong do điện thoại hết bộ nhớ hay vì một lí do nào đó thì lại bảo với Hệ điều hành rằng không cần tạo lại service

- **START_REDELEVER_INTENT**: nếu Service bị kill thì nó sẽ được khởi động lại với một Intent là Intent cuối cùng mà Service được nhận. Điều này thích hợp với các service đang thực hiện công việc muốn tiếp tục ngay tức thì như download fie chẳng hạn

- **START_STICKY_COMPATIBILITY**: giống như START_STICKY nhưng nó không chắc chắn, đảm bảo khởi động lại service

### c. stopService(intent) vs stopSeft() vs stopSeft(startId)

- **stopService(intent)**: để dừng service, nhưng gọi ở ngoài service muốn dừng

- **stopSeft()**: để dừng service, nhưng gọi ở bên trong service

- **stopSeft(startId)**: để dừng service hiện tại, nhưng chỉ khi startId trùng với id của service được start cuối cùng

Mỗi lần gọi startService() thì trong onStartCommand có paremater startId, sẽ được tăng thêm 1 đơn vị

### d. Lưu ý

- Một context gọi startService(), nếu service đó chưa được tạo sẽ gọi onCreate() -> onStartCommand()

- Nếu một context khác (service, activity, broadcast, ...) gọi tới service mà service đó đã chạy -> chỉ có onStartCommand() được gọi

=> Start bao lần thì chỉ có duy nhất 1 instance của service được tạo ra

### 5. Bound service

### a. Sử dụng

Kiểu mô hình client - server.

<img src="https://images.viblo.asia/5210d9be-e4a0-430b-8e7f-dd37c47e0678.png" width="450">

Client sẽ là các đối tượng gọi bind tới service

Server chính là service, ở đây sẽ thực hiện các tác vụ, rồi trả về kết quả cho client

### b. Vòng đời

### c. Lưu ý








