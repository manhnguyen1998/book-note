- Thông thường, khi một sự kiện nào đó phát sinh ở một thẻ HTML, có thể listen ở đâu đó nhờ js
- Với component, có thể dùng EventEmitter và @Output decorator

```js
export interface Author {
  id: number;
  firstName: string;
  lastName: string;
  email: string;
  gender: string;
  ipAddress: string;
}
```

```js
import { Component, OnInit } from '@angular/core';
import { authors } from '../authors';
@Component({
  selector: 'app-author-list',
  template: `<app-author-detail
    *ngFor="let author of authors"
    [author]="author"
  ></app-author-detail>`,
  styles: [``],
})
export class AuthorListComponent implements OnInit {
  authors = authors;
  constructor() {}
  ngOnInit() {}
}
```
```ts
import { Component, OnInit, Input } from '@angular/core';
import { Author } from '../authors';
@Component({
  selector: 'app-author-detail',
  template: `
    <div *ngIf="author">
      <strong>{{ author.firstName }} {{ author.lastName }}</strong>
      <button (click)="handleDelete()">x</button>
    </div>
  `,
  styles: [``],
})
export class AuthorDetailComponent implements OnInit {
  @Input() author: Author;
  constructor() {}
  ngOnInit() {}
  handleDelete() {}
}
```
- Khi muốn delete author trong Author Detail component -> không nên làm, vì có thể delete xong sẽ không work. Vì thông tin này không phải của Author Detail component, nên sẽ không được phép modify, mà sẽ cần phải gửi một event cho parent component để báo rằng chúng ta muốn xoá phần tử đó -> Cần dùng Event Emitter và @Output decorator
```ts
export class AuthorDetailComponent implements OnInit {
  @Input() author: Author;
  @Output() deleteAuthor = new EventEmitter<Author>();
  constructor() {}
  ngOnInit() {}
  handleDelete() {
    this.deleteAuthor.emit(this.author);
  }
}
```


-> Ở parent, có thể listen và handle 
```ts
@Component({
  selector: 'app-author-list',
  template: `<app-author-detail
    *ngFor="let author of authors"
    [author]="author"
    (deleteAuthor)="handleDelete($event)"
  >
  </app-author-detail>`,
  styles: [``],
})
export class AuthorListComponent implements OnInit {
  authors = authors;
  constructor() {}
  ngOnInit() {}
  handleDelete(author: Author) {
    this.authors = this.authors.filter((item) => item.id !== author.id);
  }
}
```