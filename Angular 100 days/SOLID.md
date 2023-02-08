# SOLID là gì 
Tập hợp của 5 nguyên tắc sau:
- Single responsibility principle
- Open/closed principle
- Liskov substitution principle
- Interface segregation principle
- Dependency inversion principle

1. Single responsibility principle
- **Một class chỉ nên giữ 1 trách nhiệm duy nhất**
```js
public class ReportManager()
{
   public void ReadDataFromDB();
   public void ProcessData();
   public void PrintReport();
}
```

Vd như class trên nắm tới 3 vai trò: đọc, xử lý, in... Khi DB thay đổi, cũng phải sửa đổi class này, dần dần sau sẽ càng phình to ra -> nên tách ra 3 class nhỏ hơn 

2. Open/Closed principle
- **Có thể thoải mái mở rộng 1 class, nhưng không được sửa đổi bên trong class đó** 
-> Mỗi khi thêm một chức năng cho chương trình, nên viết class mở rộng cũ bằng cách kế thừa hoặc sửa đổi class cũ, không nên viết lại nó 

3. Liskov Substitution Principle
- **Trong một chương trình, các object của class con có thể thay thế class cha mà không làm thay đổi tính đúng đắn của chương trình** 

-> Vd như có class Vịt, các class VịtBầu, VỊtXiêm có thể kế thừa class này. Nhưng nếu viết class VịtChạyPin, vì vịt không có pin nên sẽ không chạy được, gây lỗi -> Đây là một dạng vi phạm, làm hỏng tính đúng đắn của chương trình 

4. Interface Segregation Principle
Thay vì dùng một interface lớn, ta nên tách thành nhiều interface nhỏ, với nhiều mục đích cụ thể 
Vd như một interface quá lớn có 100 method, việc implements sẽ khá cực, và không dùng hết được 

5. Dependency inversion Principle 
- Các module cấp cao không nên phụ thuộc vào các modules cấp thấp 
	- Cả 2 nên phụ thuộc vào abstraction 
- Interface (abstraction) không nên phụ thuộc vào chi tiết, mà ngược lại..
	- (Các class giao tiếp với nhau qua interface, không phải thông qua implementation)

-> Vd: đèn tròn và huỳnh quang đều có đuôi tròn, có thể thay thế cho nhau. Đây chính là sự phụ thuộc vào abstraction, đuôi đèn là interface, implementation là bóng tròn và bóng huỳnh quang.

-> Trong code chỉ cần quan tâm tới interface, kết nối với DB chỉ cần gọi hàm Get, Save của Interface IDataAccess, Khi thay database, chỉ cần thay implementation




