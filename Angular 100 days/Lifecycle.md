# NgOnInit vs Constructor
- Angular được xây dựng xung quanh Component và Directive, một thời điểm có thể có 1 component được khởi tạo, một thời điểm khác chúng được xoá khỏi view....
- Constructor là hàm tạo của một class, function đặc biệt mà khi khởi tạo một instance thì sẽ được tự động chạy, **duy nhất một lần**
- NgOninit là life-cycle method, sẽ được Angular tự động gọi khi component được khởi tạo, sau khi constructor chạy, sau khi các input đã được binding
	- -> nếu binding cho một property ở template của component cha -> constructor của component con sẽ chưa nhận được giá trị đã binding, nhưng ngOnInit đã nhận được 
- Thực tế, Angular khuyến cáo hạn chế code ở constructor, constructor làm càng ít nhiệm vụ càng tốt, hãy để ngOnInit làm nhiệm vụ còn lại 

# ngOnChanges
- Giả sử trong trường hợp không biết người dùng binding những dữ liệu có hợp lệ không, muốn validate Input thì có cách nào?
	- Dễ dàng validate lần đầu ở trong ngOnInit được, nhưng các lần sau ngOnInit không chạy lại thì không phải là giải pháp toàn diện 
- -> Sử dụng ngOnChanges
	- ngOnChanges sẽ chạy lại mỗi khi có một input nào đó thay đổi, nó sẽ được tự động gọi, -> có thể validate property progress

```js
export class ProgressBarComponent implements OnInit, OnChanges {
  @Input() backgroundColor: string;
  @Input() progressColor: string;
  @Input() progress = 0;

  constructor() {}

  ngOnChanges(changes: SimpleChanges) {
    if ('progress' in changes) {
      if (typeof changes['progress'].currentValue !== 'number') {
        const progress = Number(changes['progress'].currentValue);
        if (Number.isNaN(progress)) {
          this.progress = 0;
        } else {
          this.progress = progress;
        }
      }
    }
  }

  ngOnInit() {}
}
```
- Trong trường hợp không thích dùng ngOnChanges, có thể dùng getter/setter 
```js
export class ProgressBarComponent implements OnInit {
  @Input() backgroundColor: string;
  @Input() progressColor: string;
  private $progress = 0;
  @Input()
  get progress(): number {
    return this.$progress;
  }
  set progress(value: number) {
    if (typeof value !== 'number') {
      const progress = Number(value);
      if (Number.isNaN(progress)) {
        this.$progress = 0;
      } else {
        this.$progress = progress;
      }
    } else {
      this.$progress = value;
    }
  }

  constructor() {}

  ngOnInit() {}
}

## [](https://github.com/angular-vietnam/100-days-of-angular/blob/master/Day007-Component-Interaction-01.md#summary)
```