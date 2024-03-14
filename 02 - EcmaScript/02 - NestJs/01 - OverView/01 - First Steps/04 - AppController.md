```ts
@Controller()  
export class AppController {  
  constructor(private readonly appService: AppService) {}  
  
  @Post()  
  getHello() {  
    return this.appService.getHello();  
  }  
}
```

با استفاده از دکوراتور `@Controller()` رفتار کلاس `AppController` رو گسترش دادیم و به nest گفتیم که کلاس `AppController` رو یک controller در نظر بگیر.

با استفاده از دکوراتور `@Post()` رفتار متد `getHello()` رو گسترش دادیم و همچنین به nest گفتیم که این متد برای مدیریت کردن درخواست های از نوع post هست.

