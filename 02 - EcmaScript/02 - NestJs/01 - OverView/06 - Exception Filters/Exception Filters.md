>[!tip]
>**کلمات کلیدی این بخش:**
>`@Catch()`
>
>`ExceptionFilter`
>
>`catch()`
>
>`ArgumentHost`
>
>`HttpException`
>
>`@UseFilters()`
>
>`useGlobalFilters()`
>
>`APP_FILTER`
>

---

بطور کلی nest لایه ای بصورت داخلی برای مدیریت خطا (Built-in Exception Layer) داره که مسئولیت مدیریت کردن خطا های handler نشده و خطا های از نوع کلاس `HttpException` در برنامه رو بر عهده داره.

![](./Images/Pasted%20image%2020240228202035.png)

همانطور که گفتیم این لایه (**global exception filter**) علاوه بر مدیریت کردن خطا های handler نشده ، خطا هایی رو که از نوع کلاس `HtppException` و یا زیر کلاس های آن هستند رو نیز handler میکنه.

حالا اگه یه خطایی رخ بده که از جنس این کلاس (HttpException) یا زیر مجموعه هاش نیستند ، سیستم توکار **global exception filter** پیام خطایی با محتوای زیر رو نشون میده:

```json
{
  "statusCode": 500,
  "message": "Internal server error"
}
```

>[!tip]
>این global exception filter تا حد زیادی از کتابخانه `http-errors` پشتیبانی میکنه.

## Throwing standard exceptions----------

سیستم nest به صورت داخلی از کتابخانه `HttpException` برای زمانی که خطایی رخ میده ، استفاده میکنه.

به عنوان مثال فرض کنید ، زمانی که در `CatsController` متد `findAll()` رو صدا می زنیم ، به ارور بخوریم:

`CatsController`
```typescript
@Get()
async findAll() {
  throw new HttpException('Forbidden', HttpStatus.FORBIDDEN);
}
```

>[!tip]
  >لازم به ذکره که `HttpStatus` یه helper enum (فهرست کمک کننده) هست.

حالا وقتی client این endpoint رو صدا میزنه ، با خطای زیر مواجع میشه:

```json
{
  "statusCode": 403,
  "message": "Forbidden"
}
```

حالا سازنده `HttpException` دو تا پارامتر ورودی ، مورد نیازشه :
- آرگومان اول اسمش `response` هست که بدنه json ای که می خوایم به عنوان response بفرستیم رو مشخص میکنه و می تونه از نوع `string` و یا `object` باشه.
- آرگومان دوم `status` هست ، که یه http status code رو میگیره.

حالا  json ای است که به عنوان response فرستاده میشه ، بدنه اش (Json response body) بصورت پیش فرض شامل دو تا property هست:
- ویژگی اول `statusCode` هست که مقدارش  از مقداری که به پارمتر دوم سازنده ی HttpException می دهیم ، گرفته می شود.
- ویژگی دوم `message` هست که مقدارش از پارامتر اولی که به سازنده ی HttpException می دهیم ، گرفته می شود (البته در صورتی که string داده باشیم.)

>[!tip]
>یعنی در واقع اگر بخواهیم تنها بخش message رو override کنیم ، کافیه به پارامتر اول سازنده HttpException یک string بدیم و اما اگر بخوایم کل بدنه response رو تغییر بدیم ، به  پارامتر اول سازنده HttpException یک object می دهیم.

`cats.controller.ts`
```typescript
@Get()
async findAll() {
  try {
    await this.service.findAll()
  } catch (error) { 
    throw new HttpException({
      status: HttpStatus.FORBIDDEN,
      error: 'This is a custom message',
    }, HttpStatus.FORBIDDEN, {
      cause: error
    });
  }
}
```

```json
{
  "status": 403,
  "error": "This is a custom message"
}
```

پارامتر سوم هم که اختیاریه و می تونه یه object با دو تا property به نام `cause` و `description` بگیره.

## Custom exceptions---------------

در بیشتر موارد ، نیازی به custom exception نداریم و کلاس HttpException داخلی خود nest می تونه کار ما رو راه بندازه
اما به هر دلیل اگر نیاز به custom exception شد بهتره کلاس مورد نظر خود را با ارث بری از HttpException بنویسیم:

`forbidden.exception.ts`
```typescript
export class ForbiddenException extends HttpException {
  constructor() {
    super('Forbidden', HttpStatus.FORBIDDEN);
  }
}
```

>[!tip]
>همونطور که قبلا هم گفتیم ، سازنه کلاس HttpException به دو پارامتر ورودی برای ایجاد شدن نیاز داره ، بنابراین این دو پارامتر رو بوسیله متد super تامین می کنیم

حالا چون از HttpException ارث بری کردیم ، سیستم Exception filter داخلی nest این خطا رو می تونه رهگیری کنه

```typescript
@Get()
async findAll() {
  throw new ForbiddenException();
}
```

مثالی دیگر:

```ts
export class MyException extends  HttpException{  
  constructor(  
  
  ) {  
    super('MyException', HttpStatus.BAD_GATEWAY, {cause:'Hello', description:'ddd'});  
    console.log(this.stack);  
  }  
}
```

## Built-in HTTP exceptions---------------

فریمورک nest یه سری exception استاندارد داخلی که از HttpException ارث بری کردن رو ارائه کرده:

- `BadRequestException`
- `UnauthorizedException`
- `NotFoundException`
- `ForbiddenException`
- `NotAcceptableException`
- `RequestTimeoutException`
- `ConflictException`
- `GoneException`
- `HttpVersionNotSupportedException`
- `PayloadTooLargeException`
- `UnsupportedMediaTypeException`
- `UnprocessableEntityException`
- `InternalServerErrorException`
- `NotImplementedException`
- `ImATeapotException`
- `MethodNotAllowedException`
- `BadGatewayException`
- `ServiceUnavailableException`
- `GatewayTimeoutException`
- `PreconditionFailedException`

مثلا `BadRequestException` چون statusCode ش از قبل مشخص شده دیگه پارامتر statusCode نداره 
پارامتر سوم هم optional هست که یه object هست که می تونه دو ویژگی `cause` و `description` رو بگیره: 

```typescript
throw new BadRequestException('Something bad happened', { cause: new Error(), description: 'Some error description' })
```

```json
{
  "message": "Something bad happened",
  "error": "Some error description",
  "statusCode": 400,
}
```

مقدار property ی description تحت عنوان property ی error در خروجی چاپ میشه.

>[!tip]
>لازم به ذکره که این پارمتر سوم در کلاس پایه HttpEception نیز وجود داره.

## Custom Exception filters----------------

خود exception filter بطور خودکار هر error ای رو مدیریت میکنه ، با این حال می تونیم exception filter layer سفارشی خودمون رو بنویسیم.

مثلا فرض کنید ، می خواهیم این لایه یه logger داشته باشه و یا ساختار json ای که به عنوان response میفرستیم رو بسته به فاکتور های مختلف عوض کنیم

پس یعنی می تونیم کنترل کاملی روی جریان (flow) و محتوا (content) ارسالی خطا با تعریف یه exception filter اختصاصی  ، داشته باشیم.

---

بیایم یه exception filter بسازیم که مسئول catch کردن exception هایی که نمونه ای از کلاس HttpException اند ، می باشد.

این exception filter یه کلاسه که اینترفیس `ExceptionFilter` رو پیاده سازی کرده و بالاش از دکوراتور `@Catch()` استفاده شده (این دکوراتور رفتار کلاس زیرش رو گسترش میده و همچنین به سیستم nest میفهمونه که این کلاس قراره نقش یه exception filter رو بازی کنه ، این دکوراتور در پارامتر های ورودی ای که میگیره مشخص میکنه که این فیلتر برای چه نوع خطا هایی باشه)

`http-exception.filter.ts`
```typescript
import { ExceptionFilter, Catch, ArgumentsHost, HttpException } from '@nestjs/common';
import { Request, Response } from 'express';

@Catch(HttpException)
export class HttpExceptionFilter implements ExceptionFilter {
  catch(exception: HttpException, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const request = ctx.getRequest<Request>();
    const status = exception.getStatus();

    response
      .status(status)
      .json({
        statusCode: status,
        timestamp: new Date().toISOString(),
        path: request.url,
      });
  }
}
```

>[!tip]
>همه exception filter ها باید اینترفیس `ExceptionFilter<T<`رو پیاده سازی کنند.
>این پیاده سازی ، شما رو مجبور به ایجاد متدی به نام `catch(exception: T, host: ArgumentsHost)` می کند.

دکوراتور `@Catch(HttpException)` دو کار انجام میده:
- یه سری metaData مورد نیاز رو به کلاس HttpExceptionFilter اضافه می کند 
- و به nest می گوید این Custom exception filter نوشته شده قراره Exception هایی که از نوع کلاس HttpException اند رو filter کنه.
- همین دو کار و بس!

این دکوراتور می تونه یه پارامتر یا چندین پارامتر ورودی بگیره ، که چندین پارامتر به شما اجازه میده تعیین کنید این filter برای چه کلاس هایی از Exception باشد.

>[!tip]
>شاید تنها بخواهیم به رفتار exception filter داخلی خود nest چیزی اضافه کنیم و نخواهیم یه exception filter رو از صفر پیاده سازی کنیم ، در آخر همین بخش در همین رابطه توضیح داده شده.

## Arguments host------------

منبع دو شی request و response بالا مربوط به route handler ای هست که exception داده است.

کلاس `ArgumentHost` یه ابزار قدرتمند هست که در همه زمینه ها مورد استفاده قرار میگیرد و یکی از موارد استفاده اش در مثال بالا مشخص شده است که بوسیله نمونه ای از این کلاس و helper function هایش توانسته ایم به request و response دسترسی پیدا کنیم.

در باره این بخش در قسمت های بعدی بیشتر می خوانیم.
## Binding filters--------------

اتصال فیلتری که ساختیم 

حالا میایم فیلتری که ساختیم رو به متد یا متد های کنترلی (router handler) که می خوایم از این فیلتر استفاده کنند ، می دهیم.
برای این منظور دکوراتور `@UseFilters` رو بالای متد یا controlle مورد نظر اضافه می کنیم ، با این کار مشخص می کنیم که متد مورد نظر اگر به خطا خورد این فیلتر قراره خطا شو دریافت کنه response مناسب بفرسته:

`cats.controller.ts`
```typescript
@Post()
@UseFilters(new HttpExceptionFilter())
async create(@Body() createCatDto: CreateCatDto) {
  throw new ForbiddenException();
}
```

این دکوراتور یعنی `@UseFilters()` می تونه یک یا چند فیلتر رو بگیره.

همچنین می تونیم بجای پاس دادن نمونه ای از فیلتر مربوطه ، خود کلاس filter مربوطه رو پاس بدیم ، با این کار وظیفه نمونه سازی رو بر عهده فریمورک می گذاریم.

`cats.controller.ts`
```typescript
@Post()
@UseFilters(HttpExceptionFilter)
async create(@Body() createCatDto: CreateCatDto) {
  throw new ForbiddenException();
}
```

>[!tip]
>پیشنهاد میشه کلاس پاس بدیم تا اینکه بخوایم نمونه بسازیم ، چون nest می تونه یه نمونه بسازه و از اون در کل module استفاده کنه که این خودش باعث کاهش مصرف memory میشه چون نمونه ی کمتری ساخته میشه

در مثال بالا فیلتر مربوطه روی یک متد اعمال شده ، پس یعنی فیلتر مربوطه `method-scoped` بوده
انواع دیگه ای scoped برای filter ها رو داریم :
- فیلتر method-scoped
- فیلتر global-scoped
- فیلتر controller-scoped
- فیلتر resolver-scoped , gateway-scoped

مثالی از `controller-scoped` :

`cats.controller.ts`
```typescript
@UseFilters(new HttpExceptionFilter())
export class CatsController {}
```

مثالی از `global-scoped` :

`main.ts`
```typescript
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.useGlobalFilters(new HttpExceptionFilter());
  await app.listen(3000);
}
bootstrap();
```

#تکمیل_شود 

>[!warning]
>متد `useGlobalFilters()` فیلتری بر روی gateway و hybrid-application ها ایجاد نمیکنه.

فیلتر های global-scoped بر روی کل application نظیر controller ها و route handler ها اعمال می شود.

همانطور که در قسمت های قبلی نیز گفته شده ، global-filter هایی که خارج از module تعریف میشن قادر به تزریق وابستگی نیستند و حتما باید نمونه به آنها پاس داده شود.

برای رفع این مشکل میشه از global filter ها در سطح module استفاده کرد:

`app.module.ts`
```typescript
import { Module } from '@nestjs/common';
import { APP_FILTER } from '@nestjs/core';

@Module({
  providers: [
    {
      provide: APP_FILTER,
      useClass: HttpExceptionFilter,
    },
  ],
})
export class AppModule {}
```

>[!tip]
>باید به این نکته توجه داشت که ، در این روش یعنی ثبت exception filter در سطح ماژول ، صرف نظر از اینکه filter مربوطه در کدام ماژول ثبت میشه ، این فیلتر global خواهد بود ، یعنی در ماژول های دیگه هم میشه از این فیلتر استفاده کرد ؛ پس در واقع بهترین جا برای ثبت filter ها آن ماژولی هست که از آن فیلتر استفاده کرده

>[!tip]
>لازم به ذکره که استفاده از property ای با نام useClass تنها راه ثبت یک custom filter نیست
>در ادامه و در بخش های آینده در این باره صحبت میشه

## Catch everything----------------

اگر بخوایم همه error ها ، صرف نظر از اینکه از چه نوعی هستند رو بگیریم ، به دکوراتور `@Catch()` پارامتر ورودی نمی دیم.

#تکمیل_شود 

ادامه بخش رو مثالی از Adapter زده که بعدا (وقتی Adapeter رو فهمیدیم ) کامل میشه

ادامه دارد.....

>[!warning]
 >وقتی می خوایم از چندین فیلتر در دکوراتور `@UserFilters()` استفاده کنیم ، اون فیلتری که همه exception ها رو می خواد بگیره باید اول نوشته بشه.

## Inheritance------------------

همانطور که دیدید در بالا custom exception filter اختصاصی خودمون رو ساختیم ، 
اما ممکنه بخواهیم از exception filter داخلی nest بهمراه شخصی سازی هایی استفاده کنیم 
در این حالت از کلاس `BaseExceptionFilter` ارث بری می کنیم:

`my-exceptions.filter.ts`
```typescript
import { Catch, ArgumentsHost } from '@nestjs/common';
import { BaseExceptionFilter } from '@nestjs/core';

@Catch()
export class MyExceptionsFilter extends BaseExceptionFilter {
  catch(exception: unknown, host: ArgumentsHost) {
    super.catch(exception, host);
  }
}
```

>[!warning]
>باید به این نکته توجه کرد که زمانی که می خواهیم MyExceptionsFilter برای Method-scoped و Controller-scoped ها استفاده کنیم ، نباید از آن نمونه بسازیم و این کار رو به فریمورک واگذار کنیم.

حالا اگر بخوایم این فیلتر نوشته شده رو global کنیم ، مانند گذشته دو راه وجود دارد:

- روش اول 

```typescript
async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  const { httpAdapter } = app.get(HttpAdapterHost);
  app.useGlobalFilters(new MyExceptionsFilter(httpAdapter));

  await app.listen(3000);
}
bootstrap();
```

>[!tip]
>روش اول مانند زمانی هست که یه custom exception filter می ساختیم اما یه فرقی داره و اونم اینه که باید `HttpAdapter` رو به سازنده نمونه ساخته شده از MyExceptionsFilter پاس بدیم

- روش دوم

```ts
import { Module } from '@nestjs/common';
import { APP_FILTER } from '@nestjs/core';

@Module({
  providers: [
    {
      provide: APP_FILTER,
      useClass: MyExceptionFilter,
    },
  ],
})
export class AppModule {}
```

