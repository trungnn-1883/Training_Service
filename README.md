# Training_Service

## I. Khái niệm

### 1. Service là gì ?

Là 1 thành phần của Android, tạo bởi hệ thống có thể thực hiện các hoạt động lâu dài trong background và không cung cấp giao diện người dùng, có thể hoạt động khi ứng dụng bị hủy

Service chạy trong main thread của process chứa nó (process có thể chứa nhiều thread). Service không tạo 1 thread riêng của nó cũng như không chạy trong 1 process độc lập trừ khi bạn tạo ra. Nó sẽ chạy trong Main UI Thread. 

Vì thế nếu bạn muốn thực hiện 1 tác vụ nặng, tốn CPU, đợi kết quả, ... như nghe nhạc hoặc mạng, nên tạo 1 thread trong service để làm những việc đó, tránh lỗi ARN.

### 2. Phân loại service

Trước kia người ta thường chia làm 2 loại Started Service và Bound Service. Nhưng giờ đã có cách phân loại mới như sau.

- **Foreground service**: thực hiện một số thao tác người dùng **có thể chú ý**, thấy rõ ràng
Ví dụ như một ứng dụng chơi nhạc, download, ... có thể thực hiện nhiệm vụ và control nó
bằng foreground service. Foreground service phải hiển thị một **Notification**. Chạy khi gọi **startForeGroundService()**

- **Background service**: thực hiện hành động người dùng **ko cần chú ý trực tiếp**. Từ Android 8.0, service chạy trong background bị giới hạn

- **Bound service**: cung cấp 1 giao diện Client - Server cho phép các thành phần tương tác với nó: gửi yêu cầu, nhận kết quả và thậm chí là IPC (inter-process communication) - giao tiếp qua nhiều tiến trình

<img src="https://codingwithmitch.s3.amazonaws.com/static/blog/9/open_bluetooth_connection.png" width="650">

### 3. Unbounded service (foreground + background service)

### a. Sử dụng

Để chạy một tác vụ lâu dài, có thể ko cần ràng buộc với thành phần giao diện kể cả khi thành phần đó bị hủy. Ở Android 8.0 đang giới hạn cho service chạy nền, nhưng việc sử dụng vẫn tương tự

### b. Vòng đời

<img src="https://o7planning.org/vi/10421/cache/images/i/1174191.png" width="350">

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

- Start service bằng startService() hoặc startForeGroundService() - sử dụng bắt đầu ở Android 8.0

- Sau khi đã startService rồi thì những lần sau khi gọi lại chạy sẽ vào onStartCommand

- Một context gọi startService():

+ Nếu service đó chưa được tạo sẽ gọi onCreate() -> onStartCommand()

+ Nếu service đó đã được tạo -> ko gọi onCreate mà gọi onStartCommand() luôn

=> Start bao lần thì chỉ có duy nhất 1 instance của service được tạo ra

### 4. Bound service

### a. Sử dụng

Kiểu mô hình client - server.

<img src="https://codingwithmitch.s3.amazonaws.com/static/blog/9/open_bluetooth_connection.png" width="650">

Client sẽ là các đối tượng gọi bind tới service

Server chính là service, ở đây sẽ thực hiện các tác vụ, rồi trả về kết quả cho client

### b. Vòng đời

<img src="https://pasteboard.co/IBz68sk.png" width="650">

### c. Cách khởi tạo

Có 3 cách

- **Extend lại Binder class**: nếu service là private trong app của bạn và chạy cùng process với người dùng, nên tạo interface extend lại class Binder và trả về instance trog onBInd(). Người dùng giờ có thể nhận instance này rồi gọi trực tiếp đến phương thức public trong nó hoặc trong Service 

```
class PlayMusicService : Service() {

    val mIbinder: IBinder = TestBinder()

    //  Extend lại class Binder
    inner class TestBinder : Binder() {
        fun getService(): Service = this@PlayMusicService
    }

    // Trả về instance trog onBInd()
    override fun onBind(intent: Intent): IBinder? {
        return mIbinder
    }
    
   fun getNumber() = 5
    
    //
    Tạo đối tượng ServiceConnection bên client để lấy đối tượng mIbinder rồi sử dụng
    
```

- **Sử dụng Messenger**: giúp làm việc giữa các process với nhau. Service định nghĩa Handler mà đáp lại tới mỗi loại Message khác nhau. Handler này là nền tảng cho Messenger có thể chia sẻ IBinder với client, cho phép client gửi lệnh tới service sử dụng Message. Service cũng có thể gửi message cho client khi client cũng định nghĩa 1 Messenger của chính nó.

Đây là cách đơn giản nhất để thực hiện giao tiếp liên tiến trình (IPC), bởi vì Messager sắp hàng toàn bộ request vào một thread duy nhất vì thế bạn sẽ không phải thiết kế service dạng thread-safe (đồng bộ request - chỉ cho duy nhất 1 request truy suất vào tại 1 thời điểm)

```
class MessengerService : Service() {


    private lateinit var mMessenger: Messenger

    /**
     * Tạo 1 Handler để xử lý message
     */
    internal class IncomingHandler(
            context: Context,
            private val applicationContext: Context = context.applicationContext
    ) : Handler() {
        override fun handleMessage(msg: Message) {
            when (msg.what) {
                MSG_SAY_HELLO ->
                    Toast.makeText(applicationContext, "hello!", Toast.LENGTH_SHORT).show()
                else -> super.handleMessage(msg)
            }
        }
    }

    /**
     * Tạo đối tượng Messenger từ Handler trên, trả về 1 IBinder
     */
    override fun onBind(intent: Intent): IBinder? {
        Toast.makeText(applicationContext, "binding", Toast.LENGTH_SHORT).show()
        mMessenger = Messenger(IncomingHandler(this))
        return mMessenger.binder
    }
}

    // Tạo đối tượng ServiceConnection bên client để lấy đối tượng mIbinder rồi sử dụng
    
```

- **Sử dụng AIDL** (Android Interface Definition Language): chia nhỏ đối tượng thành dạng nguyên thủy mà hệ điều hành có thể hiểu được và sắp vào các process để thực hiện IPC. Messenger ở trên là dựa trên AIDL, nhưng chỉ là ở trong 1 thread. AIDL có thể thực hiện multi-thread, nhưng service phải là dạng thread-safe.

Hầu hết ứng dụng không nên sử dụng AIDL để tạo bound service, bởi vì nó có thể yêu cầu khả năng xử lý multi-thread và có thể dẫn tới những xử lý rất phức tạp. Tuy nhiên trong trường hợp bạn cần sử dụng, đó là khi muốn tương tác với app khác thì bạn có thể triển khai, nhưng chắc chắc hiểu về Bound service, AIDL và xử lý đầy đủ các trường hợp.

### d. Lưu ý

- Chỉ **activity, service và content provider** có thể bind tới service. **Broadcast receiver** không bind được. Từ fragment có thể bind tới service, bằng cách gọi getActivity() rồi bind tới. Nhưng cẩn thận vì đối tượng service lấy được dễ có thể bị null. 

- Nên bắt ngoại lệ **DeadObjectException**, được ném ra khi kết nối bị chết.

- Các đối tượng là tham chiếu được tính qua các tiến trình 

(Objects are reference counted across processes

Tham khảo: https://en.wikipedia.org/wiki/Reference_counting)

- Nên binding và unbinding theo thời điểm đối ngược nhau trong vòng đời của client, như:

+ Bind khi **onStart()**, unbind khi **onStop()**. Dùng khi tương tác với service khi activity đang được nhìn thấy

+ Bind khi **onCreate()**, unbind khi **onDestroy()**. Dùng khi tương tác với service kể cả khi nó đang chạy background. Nên cẩn thận khi sử dụng vì service sẽ chạy toàn bộ thời gian activity chạy (kể cả dưới background).

+ Không nên bind, unbind trong **onResume()**, **onPause()**

### 5. Thời điểm bị kill

Hệ thống Android stop một service chỉ khi bộ nhớ thấp và cần dùng cho activity cần focus:

- 1 service gắn với activity đang được focus, nó ít có khả năng bị kill

- 1 service chạy foreground, nó hiếm khi bị kill

- 1 service đã chạy lâu, được started thì dễ bị kill

### 6. Process vs Thread

- **Process**: nằm ở mức hệ thống, kiểm soát bởi hệ thống:

+ Khi ứng dụng chạy, hệ thống tạo ra 1 process và ứng dụng sẽ được chạy và quản lý trong process đó. 

+ Mỗi process sẽ có tài nguyên, bộ nhớ độc lập với nhau


- **Thread**: ở mức ứng dụng:

+ Khi một process được tạo, ứng dụng bên trong đó có thể tạo nhiều thread khác nhau. Đầu tiên, 1 thread sẽ được tạo ra, gọi là "main". Thread này sẽ chịu trách nhiệm vẽ, nhận sự kiện, tương tác với Android UI toolkit, .... Thread main này còn được gọi là UI thread. 

+ Thread chỉ hoạt động bên trong giới hạn của process, chia sẻ tài nguyên trong 1 process. 

Việc tạo ra multi-process hay multi-thread, người lập trình có thể quy định. Tuy nhiên phải xem xét cho kĩ lưỡng.



------------------------------------------------------------------------

Tham khảo: https://developer.android.com/guide/components/services



















