برای ایجاد controller ها از class ها و decorator ها استفاده می کنیم ، decorator ها امکان دسترسی کلاس ها به `metaData` ها رو فراهم می کنند و همچنین Decorator ها nest رو مجاب به ایجاد سیستم مسیریابی می کند یا به بیان دیگر درخواست ها به controller های مربوطه گره می خورد.

```tsx
import { Controller, Get } from "@nestjs/common";

@Controller("users")
export default class UserController {
  @Get()
  getUsers():string{
    return 'this route get all users'
  }
}
```

## Routing——————————-

 این `@GET` به Nest می گوید که یک handler (همان controller) برای endpoint ای که httpRequestMethod اش از نوع get هست ، بساز.
 
 میشه به این دکوراتور path هم پاس داد.
```tsx
@Get('test')
```
## Response Object—————————-

Standard (recommended)————-

این nest تا حد زیادی مسئولیت serialized کردن response رو برعهده می گیرد ؛ به این صورت که اگر controller ما بخواهد `array` یا `object` برگرداند ، nest آن را بصورت اتوماتیک به json سریالایز می کند و اگر controller بخواهد js primitive type مانند `string`, `number`, `boolean` همان مقادیر را بدون دستکاری و یا سریالایز کردن برمی گرداند.

در واقع شعار nest این هست که شما مقداری که می خواید رو return کنید و بقیه کار ها رو به عهده ما بگذارید.

بصورت پیش فرض همه درخواست های زده شده به یک controller دارای statusCode `200` هستند بجز درخواست زده شده به controller ای که httpRequestMethod POST هست که دارای statusCode `201` هست.

 می توان این رفتار پیش فرض statusCode رو با استفاده از دیکورتور `@HttpCode(...)` تغییر داد

Library-specific——————-

میشه کاری کنیم که بشه از سیستم ارسال response خود express استفاده کنیم که البته پیشنهاد نمیشه ، مگر برای برای موارد خاص

```tsx
import { Controller, Get, Res } from "@nestjs/common";
import { Response } from "express";

@Controller("users")
export default class UserController {
  @Get()
  getUsers(@Res() response: Response): Response {
    return response.status(200).json({ message: "hello" });
  }
}
```

هنگامی که response object را در controller مربوطه تزریق می کنیم ، در واقع مسئولیت فرستادن response رو از دوش nest برداتیم و وظیفه این کار رو به عهده گرفتیم ، پس اگر در این شرایط response نفرستیم ، برنامه هنگ می کند.

## Request Object——————————-

ممکنه نیاز به request object داشته باشیم ، پس نیاز به تزریق آن در controller مربوطه با استفاده از دکوراتور @Req داریم 

```tsx
import { Controller, Get, Req } from '@nestjs/common';
import { Request } from 'express';

@Controller('cats')
export class CatsController {
  @Get()
  findAll(@Req() request: Request): string {
    return 'This action returns all cats';
  }
}
```

خود request object مقادیری داره که با تزریق اونا می تونیم بهشون دسترسی داشته باشیم.

|@Request(), @Req()|req|
|---|---|
|@Response(), @Res()*|res|
|@Next()|next|
|@Session()|req.session|
|@Param(key?: string)|req.params / req.params[key]|
|@Body(key?: string)|req.body / req.body[key]|
|@Query(key?: string)|req.query / req.query[key]|
|@Headers(name?: string)|req.headers / req.headers[name]|
|@Ip()|req.ip|
|@HostParam()|req.hosts|

## resource ——————————-

<aside> 💡 به entity ها در api نویسی resource می گن.

</aside>

```tsx
import { Controller, Get, Post } from '@nestjs/common';

@Controller('cats')
export class CatsController {
  @Post()
  create(): string {
    return 'This action adds a new cat';
  }

  @Get()
  findAll(): string {
    return 'This action returns all cats';
  }
}
```

این nest برای همه http method های استاندارد دکوراتور داره:

`@Get()`, `@Post()`, `@Put()`, `@Delete()`, `@Patch()`, `@Options()`, and `@Head()`. In addition, `@All()` defines an endpoint that handles all of them.

## **Route wildcards———————-**

```tsx
@Get('ab*cd')
findAll() {
  return 'This route uses a wildcard';
}
```

<aside> 💡 استفاده از کاراکتر های `?`, `+`, `*`, and `()` در PATH به عنوان `string patterns` در نظر گرفته می شود. اما کاراکتر های DASH و DOT (- ، .) جزئی از رشته رشته string ای PATH در نظر گرفته می شود.

</aside>

<aside> 💡 این wildCard ها فقط در Express پشتیبانی می شوند.

</aside>

## **Status code——————————-**

```tsx
import { HttpCode } from "@nestjs/common";
@Post()
@HttpCode(204)
create() {
  return 'This action adds a new cat';
}
```

<aside> 💡 برای زمانی که ممکن است شرایطی پیش بیاد که statusCode داینامیک باشد ، می تونید شی Response رو تزریق کنید و از اون استفاده کنید.

</aside>

## **Headers——————————-**

<aside> 💡 دو روش برای اضافه کردن header وجود داره :

- اضافه کردن دکوراتور
- یا استفاده از library-specific (تزریق response و استفاده از متد header() آن `res.header()`)

</aside>

```tsx
@Post()
@Header('Cache-Control', 'none')
create() {
  return 'This action adds a new cat';
}
```

## **Redirection——————————-**

برای redirection هم مثل headers دو روش `@Redirect()` و یا `res.redirect()` وجود دارد.

دکوراتور @Redirect() دو پارامتر `url` و `statusCode` رو به عنوان ورودی میگیره که اختیاری هم هستند.

دیفالت statusCode `302` به معنای Found (قبلا جابه جایی موقت)

```tsx
@Get()
@Redirect('<https://nestjs.com>', 301)
```

بررسی شود؟

> HINT Routes with parameters should be declared after any static paths. This prevents the parameterized paths from intercepting traffic destined for the static paths.
> 
> ```
> HttpRedirectResponse
> ```
> 
> ```
> @nestjs/common
> ```

Returned values will override any arguments passed to the `@Redirect()` decorator. For example:

## Route Queries—————————-

```tsx
@Get('docs')
@Redirect('<https://docs.nestjs.com>', 302)
getDocs(@Query('version') version) {
  if (version && version === '5') {
    return { url: '<https://docs.nestjs.com/v5/>' };
  }
}

```

در روش بالا اگر 1000 تا query هم در ورودی وارد بشه فقط اونی که اسمش version هست گرفته میشه ، اگه بخوایم همه query ها دریافت بشن دکوراتور query نباید پارامتر ورودی ای داشته باشه و یا می تونیم از library-specific استفاده کنیم.

```tsx
  @Get()
  getHello(@Query() query:string[]) {
    return query;
  }
```

## **Route parameters———————-**

```tsx
  @Get(":id")
  getHello(@Param() param:string) {
    return param;
  }
```

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/5e86db0c-3d3f-4a3d-a68a-5ec0b47f544c/42f391c3-9cd6-47e6-b5a0-c4e834c47566/Untitled.png)

<aside> 💡 اگه تایپ param رو any بزاریم می تونیم به [param.id](http://param.id) دسترسی داشته باشیم.

</aside>

ولی خب در Ts بهتره به شیوه پایین عمل کنیم:

```tsx
interface Param{
  id:string
}

@Get(":id")
getHello(@Param() param:Param) {
	return param.id;
}
```

حالا اگه بخوایم مستقیم به id دسترسی داشته باشیم :

```tsx
@Get(":id")
getHello(@Param('id') id:number) {
	return id;
}
```

## **Sub-Domain Routing—————————-**

```tsx
@Controller({ host: 'admin.example.com' })
export class AdminController {
  @Get()
  index(): string {
    return 'Admin page';
  }
}
```

```tsx
@Controller({ host: 'localhost' })
export class AdminController {
  @Get()
  index(): string {
    return 'Admin page';
  }
}
```

```tsx
// بررسی شود
@Controller({ host: 'account.example.com' })
export class AccountController {
  @Get()
  getInfo(@HostParam('account') account: string) {
    return account;
  }
}

```

## **Scopes————————-**

در این باره در بخش [**Injection scopes**](https://www.notion.so/Injection-scopes-150d6b9770b042eab3fcfcd6059ef540?pvs=21) می خوانیم.

## **Asynchronicity—————————-**

همه async function ها یه Promise رو return می کنند.

```tsx
@Get()
async findAll(): Promise<any[]> {
  return [];
}
```

خود nest بصورت داخلی از RxJs هم پشتیبانی میکنه:

[Observable | RxJS API Document (reactivex.io)](https://reactivex.io/rxjs/class/es6/Observable.js~Observable.html)

```tsx
@Get()
findAll(): Observable<any[]> {
  return of([]);
}
```

## Error Handling————————-

## **Request payloads—————————-**

**محموله های درخواست————-**

محموله های یا داده هایی که قرار است از سمت client ارسال شوند باید در سمت بک اند درون ظرفی ریخته شوند ؛ به ظرفی که برای این داده ها در نظر گرفته شده data transfer object یا `DTO` گویند که می تواند از جنس کلاس یا interface باشد

برای اینکه interface در js وجود ندارد بهتر است این ظرف رو از جنس کلاس در نظر بگیریم تا nest بتواند در زمان اجرا نیز به متادیتا هایی دسترسی داشته باشد تا ضمن این دسترسی بتواند درخواست ارسال شده را validate کند تا اگر data ی ارسالی خارج از ظرف Dto تشکیل شده بود به نوعی اجازه ورود به کنترل مربوطه رو ندهد.

**create-cat.dto.ts**

```tsx
export class CreateCatDto {
  name: string;
  age: number;
  breed: string;
}
```

**cats.controller.ts**

```tsx
@Post()
async create(@Body() createCatDto: CreateCatDto) {
  return 'This action adds a new cat';
}
```

طریقه کار validation payload ارسالی از request به این صورته که `ValidationPipe` ای وجود داره که یک white list ای داره ، که در این whiteList تنها property هایی مجاز اند دریافت شوند که جز property های dto class ما هستند.(البته تنظیماتی هم باید اعمال بشه)

## **Handling errors—————————-**

در این باره در بخش [**[Exception filters](https://docs.nestjs.com/exception-filters)**]([https://www.notion.so/Exception-filters-cc650dd36d6b4c45a9eb25b013bca93f?pvs=21](https://www.notion.so/Exception-filters-cc650dd36d6b4c45a9eb25b013bca93f?pvs=21)) می خونیم.

## **Full resource sample————————-**

```tsx
import { Controller, Get, Query, Post, Body, Put, Param, Delete } from '@nestjs/common';
import { CreateCatDto, UpdateCatDto, ListAllEntities } from './dto';

@Controller('cats')
export class CatsController {
  @Post()
  create(@Body() createCatDto: CreateCatDto) {
    return 'This action adds a new cat';
  }

  @Get()
  findAll(@Query() query: ListAllEntities) {
    return `This action returns all cats (limit: ${query.limit} items)`;
  }

  @Get(':id')
  findOne(@Param('id') id: string) {
    return `This action returns a #${id} cat`;
  }

  @Put(':id')
  update(@Param('id') id: string, @Body() updateCatDto: UpdateCatDto) {
    return `This action updates a #${id} cat`;
  }

  @Delete(':id')
  remove(@Param('id') id: string) {
    return `This action removes a #${id} cat`;
  }
}
```

<aside> 💡 بسیاری از کار های تکراری مانند ساخت controller برای resource ها رو بوسیله cli بصورت خودکار میشه انجام داد.

</aside>

## **Getting up and running————————-**

وقتی یک controoler می سازیم ، nest از وجود این controoler اطلاعی ندارد و نمی تواند نمونه ای از controller ایجاد شده توسط ما بسازد

در نست controller ها همیشه به فایل ماژول تعلق دارند و آنها را در آرایه از controller ها در دکوراتور `@module` تعریف می کنیم.

```tsx
import { Module } from '@nestjs/common';
import { CatsController } from './cats/cats.controller';

@Module({
  controllers: [CatsController],
})
export class AppModule {}
```

دکوراتور @Module() میاد controller ها رو به metaData کلاس `AppModule` ضمیمه می کنه و این باعث میشه تا nest بفهمه چه controller هایی نصب شده.

## **Library-specific approach———————-**

تاکنون در مورد روش استاندارد Nest برای دستکاری پاسخ ها بحث کرده ایم. راه دوم برای دستکاری پاسخ استفاده از یک شی پاسخ خاص کتابخانه است. برای تزریق یک شی پاسخ خاص، باید از دکوراتور @Res() استفاده کنیم. برای نشان دادن تفاوت ها، اجازه دهید CatsController را به صورت زیر بازنویسی کنیم:

```tsx
import { Controller, Get, Post, Res, HttpStatus } from '@nestjs/common';
import { Response } from 'express';

@Controller('cats')
export class CatsController {
  @Post()
  create(@Res() res: Response) {
    res.status(HttpStatus.CREATED).send();
  }

  @Get()
  findAll(@Res() res: Response) {
     res.status(HttpStatus.OK).json([]);
  }
}
```

اگرچه این رویکرد کار می کند، و در واقع با ارائه کنترل کامل شی پاسخ (دستکاری سرصفحه ها، ویژگی های خاص کتابخانه، و غیره) انعطاف پذیری بیشتری را از برخی جهات اجازه می دهد، اما باید با احتیاط از آن استفاده کرد. به طور کلی، رویکرد بسیار کمتر واضح است و دارای معایبی است. نقطه ضعف اصلی این است که کد شما وابسته به پلتفرم می شود (زیرا کتابخانه های زیربنایی ممکن است API های متفاوتی روی شی پاسخ داشته باشند)، و آزمایش آن سخت تر است (شما باید شی پاسخ را مسخره کنید و غیره).

همچنین همانطور که بالا تر گفته شده با این کار ویژگی های خودکار nest برای reponse رو از دست می دهیم که برای رفع این مشکل می توانیم گزینه `passthrough` را به صورت زیر روی true قرار دهید:

```tsx
@Get()
findAll(@Res({ passthrough: true }) res: Response) {
  res.status(HttpStatus.OK);
  return [];
}
```

با این کار تنها کنترل کامل response object که شامل set cookies or headers conditions و… است رو برعهده می گیریم اما بقیه کار ها رو به framework واگذار می کنیم.