>[!tip]
>میایم component (controller , services) های مرتبط با هم رو می پیچیم دور یه چیزی به اسم module
>با این کار componet هایی که feature های نزدیک بهم دارند رو می پیچیم دور یه feature module

```ts
@Injectable()  
export class AppService {  
  getHello(): string {  
    return 'Hello World!';  
  }  
}
```

با استفاده از دکوراتور `@Injectable()` رفتار کلاس `AppService` رو گسترش دادیم و همچنین به nest گفتیم که کلاس `AppService` رو به عنوان یه کلاس تزریقی در نظر بگیر.

