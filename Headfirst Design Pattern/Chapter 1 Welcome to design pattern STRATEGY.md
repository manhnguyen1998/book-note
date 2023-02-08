# Problem 
![[Pasted image 20220819223542.png]]

Như hình trên, mọi class kế thừa Duck, nếu muốn thêm chức năng mới cho mọi con vịt -> RubberDuck cũng sẽ bị ảnh hưởng (fly) -> không đúng logic 


## Nguyên tắc thiết kế 
![[Pasted image 20220819223747.png]]

Nếu bạn có một số đoạn code đang thay đổi, xem xét tất cả những yêu cầu mới, một số hành vi cần phải tách ra khỏi những thứ không thay đổi 

Có một cách khác: lấy các thành phần thay đổi và đóng gói chúng, để sau này có thể thay đổi hoặc mở rộng các thành phần khác nhau mà không ảnh hưởng đến những phần còn lại 

## Tách những thứ thay đổi ra khỏi những thứ không thay đổi 
![[Pasted image 20220819224152.png]]

Tách các thứ có thể thay đổi của Duck là Fly() và Quack() ra lớp Duck Behavior

**Từ giờ trở đi, các hành vi của Vịt sẽ đặt trong một lớp riêng biệt, Một lớp implement một interface hành vi cụ thể.  
Theo cách đó, các lớp Vịt không cần phải biết bất kỳ implementation chi tiết nào thực hiện cho hành vi của nó.**

![[Pasted image 20220819224549.png]]

Sử dụng một interface để thực hiện từng hành vi
VD: FlyBehavior và QuackBehavior - mỗi lần cần một hành vi thì sẽ implement một trong nhũng interface đó 

![[Pasted image 20220819224728.png]]



Ví dụ đơn giản 
![[Pasted image 20220819225102.png]]

Nếu code như trên, 
Nhưng “**programming to an interface/supertype**” sẽ là:

```java
 Animal animal = new Dog();  
animal.makeSound();
```
Thậm chí, có thể gán như sau 
a = getAnimal();
a.makeSound();

## Implementing những hành vi của Duck 
Có 2 giao diện, FlyBehavior và QuackBehavior 
![[Pasted image 20220819225320.png]]

Với thiết kế này, các loại đối tượng khác có thể sử dụng lại các hành vi fly và quack, bởi vì chúng không còn bị che giấu trong các lớp Duck. Và có thể thêm các hành vi mới mà không cần sửa đổi bất kỳ hành vi hiện có nào hoặc sửa đổi vào bất kỳ lớp Duck nào sử dụng hành vi fly()


# STRATEGY
![[Pasted image 20220822232753.png]]


# Làm cách nào để sử dụng các mẫu thiết kế 
Các mẫu thiết kế không đi trực tiếp vào code, chúng đi vào não 