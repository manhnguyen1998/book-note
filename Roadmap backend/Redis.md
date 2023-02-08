# Redis 
- là 1 trong các hệ quản trị dữ liệu Nosql

- Mọi chuyện bắt đầu từ antirez:
- Server của antirez nhận 1 lượng lớn thông tin từ trang web khác nhau thông qua js tracker, lưu trữ n page view cho từng trang và hiển thị theo thời gian thực cho user. TUy nhiên lượng page view tăng đến hàng nghìn page/s, -> không thể tìm ra các nào thực sự tối ưu cho việc thiết kế DB.

-> Anh ta nhận ra rằng việc lưu trữ danh sách bị giới hạn các bản ghi k phải vấn đề khó. Lưu trữ thông tin trên RAM và quản lý các page views dưới dạng native data với thời gian pop và push là hằng số được ra đời


# Redis の特徴
1。Data model

It is different from Mysql or Postgre, Redis have no table. It save data as key-value type.
Below are types of data in redis:
- STRING: string, integer, float. Redis can work with all string, a part of string, add/subtract integer, float
- LIST: linked list of strings. Redis support these method like push, pop from both side of list, trim by offset, read 1 or more items of list, search and delete value
- SET: set of string (non-sort). support read, delete, add... check existence of value in set. Intersect/union/different....
- HASH: save hash table of key-value, key is random
- ZSET: (sorted set): is a list, each value is map of 1 string and 1 floating-point

2. Master/slave replica
3. Clustering
If use Mysql, you must pay fee for this function. But in Nosql, it's free. 
4. In-memory
Redis does not save data in hard disk, it save data in ram.
But if server is down, are these data is gone?
-> Redis have it's way to save data 

# Redis persistence
Redis save data by key-value pair, but it also need save data on hard disk
1. RDB (Redis database file)
- RDB thực hiện tạo và sao lưu snapshot của DB vào ổ cứng sau mỗi khoảng thời gian nhất định 

### Ưu điểm 
- RDB cho phép người dùng lưu các version khác nhau của DB, rất thuận tiện khi có sự cố xảy ra
- Bằng việc lưu trữ data vào 1 file cố định, người dùng có thể dễ dàng chuyển data đến các center khác, hoặc chuyển dến Amazon S3
- RDB tối ưu hoá hiệu năng của Redis, Tiến trình redis chính sẽ chỉ làm các công việc trên Ram, bao gồm các thao tác cơ bản được yêu cầu từ phía client như thêm/đọc/xoá. Trong đó 1 tiến trình con sẽ đảm nhiệm các thao tác disk I/O. Cách tổ chức này giúp tối đa hiệu năng của Redis
- Khi restart server, dùng RDB làm việc với lượng data lớn sẽ có tốc độ cao hơn AOF
### Nhược 
RDB không phải lựa chọn tối ưu nếu muốn giảm thiểu tối đa nguy cơ mất dữ liệu. Thông thường, set up RDB snapshot trong 5 phút 1 lần, nếu có sự cố, Redis không thể hoạt động thì dữ liệu cuối cùng sẽ bị mất 
RDB cần dùng fork() để tạo tiến trình con phục vụ cho thao tác disk I/O. Trong trường hợp dữ liệu quá lớn, quá trình fork() có thể tốn ghời gian và server sẽ không thể đáp ứng được request từ client trong vài ms hoặc thậm chí là 1s với hiệu năng CPU.

# AOF (Append Only File)
## Cách làm việc 
AOF lưu lại tất cả các thao tác write mà server nhận được, các thao tác này sẽ chạy lại khi restart server hoặc tái thiết database

## Ưu điểm 
DÙng AOF sẽ đảm bảo dataset được bền vững so với RDB. Người dùng có thể config để Redis ghi log theo từng câu query hoặc mỗi giây 1 lần 
Redis ghi log AOF theo kiểu thêm vào cuối file sẵn có, do đó tiến trình seek trên file có sẵn là không cần thiết. 
Redis cung cấp tiến trình chạy nền, cho phép ghi lại file AOF khi dung lượng file quá lớn. Trong khi server vẫn thực hiện thao tác trên file cũ, 1 file hoàn toàn mới được tạo ra với số lượng tối thiểu operation phục vụ cho việc tạo dataset hiện tại. Và 1 khi file mới được ghi xong, Redis sẽ chuyển sang thực hiện thao tác ghi log trên file mới.

## Nhược điểm

File AOF thường lớn hơn file RDB với cùng 1 dataset.

AOF có thể chậm hơn RDB tùy theo cách thức thiết lập khoảng thời gian cho việc sao lưu vào ổ cứng. Tuy nhiên, nếu thiết lập log 1 giây 1 lần có thể đạt hiệu năng tương đương với RDB.

Developer của Redis đã từng gặp phải bug với AOF (mặc dù là rất hiếm), đó là lỗi AOF không thể tái tạo lại chính xác dataset khi restart Redis. Lỗi này chưa gặp phải khi làm việc với RDB bao giờ.

