# Dependencies Inversion và các định nghĩa khác 

![[Pasted image 20220817231324.png]]

- **Dependency Inversion**: Một nguyên lý thiết kế và viết code trong SOLID
- Inversion of Control: là một design pattern được tạo ra để code có thể tuân theo nguyên lý **Dependency Inversion**. Có nhiều cách để thực hiện nguyên lý này: ServiceLocator, Event, Delegate.... Depency Injection là một trong các cách đó
- **Dependency Injection** (DI): là một cách để hiện thực Inversion of Control Pattern( có thể coi nó như một design pattern riêng). **Các module phụ thuộc (dependency) sẽ được inject vào module cấp cao** 

## Dependency Injection
1. Các module không giao tiếp trực tiếp với nhau, mà thông qua interface. Module cấp thấp sẽ implement interface, module cấp cao sẽ gọi module cấp thấp thông qua interface
	1. Ví dụ: Để giao tiếp với database, ta có interface IDatabase, các module cấp thấp là XMLDatabase, SQLDatabase. Các module cấp cao là CustomerBusiness sẽ chỉ sử dụng IDatabase
2. VIệc khởi tạo các module cấp thấp sẽ do DI COntainer thực hiện. Ví dụ: trong module CustomerBusiness, ta sẽ không khởi tạo IDatabase db = new XMLDatabase(), việc này sẽ do DI Container thực hiện. Module CustomerBusiness sẽ không biết gì về module XMLDatabase hay SQLDatabase 
3. Việc Module nào gắn với interface nào sẽ được config trong code hoặc trong file XML.
4. DI được dùng để giảm sự phụ thuộc giữa các module, dễ dàng hơn trong việc thay đổi module, bảo trì code và testing 

## Các dạng DI
1. Constructor Injection: Các dependency sẽ được container truyền vào (inject) 1 class thông qua constructor của class đó. Đây là cách thông dụng nhất 
2. Setter Injection: Các dependency sẽ được truyền vào 1 class thông qua các hàm Setter
3. Interface Injection: Class cần inject sẽ implement 1 interface. Interface này sẽ chứa 1 hàm tên là Inject. Container sẽ injection dependency vào 1 class thông qua hàm Inject của interface đó. Đây là cách rườm rà và ít sử dụng nhất 

# Ưu điểm và khuyết điểm 
![[Pasted image 20220817232710.png]]