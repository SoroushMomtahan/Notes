>[!tip]
>عرضه کنندگان ، سرویس (cat.service) عرضه می کنند.

## عرضه کنندگان یا ارائه دهندگان————————-

بسیاری از کلاس ها در nest می تونن به عنوان یه ارائه دهنده (Provider) در نظر گرفته بشن

**ایده اصلی** ارائه دهندگان اینه که می تونن به عنوان یه وابستگی تزریق بشن.

این به این معنی هست که اشیا می تونن روابط مختلفی با هم داشته باشن و مدیریت **سیم کشی** این روابط رو تا حد زیادی میشه به فریمورک واگذار کرد.

![](Pasted%20image%2020240228190720.png)

![](./Images/Pasted%20image%2020240228190720.png)
---

---

مثلا اینطوری در نظر بگیرید که یه کمپانی میاد با یه سری عرضه کننده قرار داد می بنده تا نیازمندی هاشو تامین کنه و محصولی که می خواد رو عرضه کنه

پس اگه هر ماژولی رو یه کمپانی در نظر بگیریم ، این کمپانی یه سری نیازمندی داره که با استفاده از عرضه کننده ها آن نیازمندی ها رو تامین میکنه ، حالا اگه اون عرضه کننده بعد از مدتی نتونه نیازمندی مربوطه رو تامین کنه ، کنار میره و تامین کندده ای دیگر جاشو می گیره.

بعضی provider ها می تونن داخلی و بعضی هم خارجی باشن یعنی یا خودمون اونا رو توسعه داده باشیم و یا یه لایبرری باشن.

>[!tip]
>در واقع controller ها یا کنترل کننده ها درخواست های http رو مدیریت می کنند و وظایف پیچیده تر رو به provider ها واگذار می کنند.

این providers یا ارائه دهندگان ، کلاس های جاوا اسکریپت ساده ای هستند که به عنوان ارائه دهنده در یک ماژول اعلام می شوند.

درباره module ها در بخش Modules صحبت می شود.

## **inversion of control** (**IoC**)————————-

در برنامه نویسی سنتی ، این برنامه نویس بود که لایبرری و یا کتابخانه های آماده ای رو صدا میزد تا در کد خود از آن استفاده کند.

الان در کنار مفهوم بالا ، مفهمومی به نام فریمورک آمده است ؛ در این مفهوم ، این فریمورک است که کد های برنامه نویس رو فراخوانی می کنه

به این مفهوم جدید **وارونگی کنترل** گویند.

برنامه نویس library رو صدا میزنه ، framework کد های برنامه نویس و همچنین لایبرری ها رو

میشه گفت فریمورک یه جورایی حکم برنامه نویس نیمه خودکار رو داره یعنی در واقع framework داره نقش یه برنامه نویس رو با فراخوانی لایبرری هایی انجام میده.

## **Services—————————-**

**cats.service.ts—————-**

```tsx
import { Injectable } from '@nestjs/common';
import { Cat } from './interfaces/cat.interface';

@Injectable()
export class CatsService {
  private readonly cats: Cat[] = [];

  create(cat: Cat) {
    this.cats.push(cat);
  }

  findAll(): Cat[] {
    return this.cats;
  }
}
```

**interfaces/cat.interface.ts——————-**

```tsx
export interface Cat {
  name: string;
  age: number;
  breed: string;
}
```

دکوراتور **`@Injectable()`** اعلام میکنه که کلاس CatsService ، **کلاسی تزریقی** است که می تونه توسط **nest ioc container** مدیریت بشه.

اینکه میگیم این کلاس تزریقی میتونه توسط **nest ioc container** مدیریت بشه یعنی :

ابتدا ما در زمان برنامه نویسی خود **تنها اعلام می کنیم** که می خواهیم از فلان کلاس تزریقی استفاده کنیم به زبان دیگر اعلام می کنیم که آقا این کلاس تزریقی به ما تزریق شود (حتی اعلام می کنیم به کجامون تزریق بشه مثلا میگیم به constructor مون تزریق بشه ) که به این روش از برنامه نویسی ، برنامه نویسی اعلامی هم میگن.

حالا در برنامه نویسی مدرن همونطور که گفتیم این framework هست که بسته به نیازش کد های ما رو حین اجرا کردن کد های خودش صدا میزنه

این framework وقتی میاد controller ما رو صدا بزنه ، میبینه که ای دل قافل این controller نیاز به تزریق چیزی داره ، بنابراین میره تو provider هاش و چیزی که آن کنترل بهش نیاز داره رو پیدا میکنه و به کنترل تزریق میکنه (این تزریق کردن هم یعنی میره یه نمونه از کلاس تزریقی میسازه و به controller مربوطه میده)

حالا یه cats.service.ts که بالاتر تعریف شده ، در واقع یه عرضه کننده یا کلاس تزریقی هست که می تونه cat رو ذخیره و همچنین بازیابی کنه

حالا در cat controller اعلام می کنیم که به تزریق شدن این کلاس نیازمندیم و یا تنها اعلام می کنیم که به این عرضه کننده نیازمندیم:

```tsx
import { Controller, Get, Post, Body } from '@nestjs/common';
import { CreateCatDto } from './dto/create-cat.dto';
import { CatsService } from './cats.service';
import { Cat } from './interfaces/cat.interface';

@Controller('cats')
export class CatsController {
  constructor(private catsService: CatsService) {}

  @Post()
  async create(@Body() createCatDto: CreateCatDto) {
    this.catsService.create(createCatDto);
  }

  @Get()
  async findAll(): Promise<Cat[]> {
    return this.catsService.findAll();
  }
}
```

در کد بالا گفتیم که آقا این چیزی که می خوای تزریق کنی رو به constructor ما تزریق کن.

## **Dependency injection————————-**

به لطف typescript تزریق وابستگی یا همان dependency injection در nest بسیار راحت است چون با استفاده از تعیین type این مسئله حل می شود.

مقاله زیر در مستندات angular برای درک مفهوم تزریق وابستگی بسیار کاربردی هست.

[https://angular.dev/guide/di](https://angular.dev/guide/di)

همانطور که گفتیم nest وقتی می خواهد از catController ما استفاده کند می بینه که این کلاس نیاز به تزریق داره و از داخل provider هاش سرویس مناسب برای تزریق رو پیدا میکنه و اونو تزریق میکنه (یه نمونه از service میسازه و نمونه رو به constructor کنترل مربوطه پاس میده)

```tsx
constructor(private catsService: CatsService) {}
```

## Scopes—————————-

ارائه دهندگان به طور معمول دارای **طول عمر ("محدوده")** هستند که با چرخه عمر برنامه هماهنگ شده است. هنگامی که برنامه بوت استرپ می شود، هر وابستگی باید حل شود، و بنابراین هر ارائه دهنده باید نمونه سازی شود. به همین ترتیب، هنگامی که برنامه خاموش می شود، هر ارائه دهنده از بین می رود. با این حال، راه‌هایی وجود دارد که می‌توانید ارائه‌دهنده مادام‌العمر خود را نیز در محدوده درخواستی قرار دهید. در اینجا می توانید اطلاعات بیشتری در مورد این تکنیک ها بخوانید.

در این باره در بخش های بعدی صحبت میشه.

## **Custom providers———————-**

موارد بیشتر درباره provider ها مثل تزریق یه provider در provider دیگر رو در بخش های بعدی می خوانیم.

## **Optional providers————————-**

زمانی که تزریق کردن یک provider اختیاری است مثلا تعین می کنیم که اگر provider ای تزریق شد از اون استفاده کن و اگر provider ای تزریق نشد از موارد پیش فرض استفاده کن.

```tsx
import { Injectable, Optional, Inject } from '@nestjs/common';

@Injectable()
export class HttpService<T> {
  constructor(@Optional() @Inject('HTTP_OPTIONS') private httpClient: T) {}
}
```

در کد بالا باید یک ارائه دهنده سفارشی به نام HTTP_OPTIONS توسط فریمورک تزریق شود و از آنجایی که این ارائه دهنده اختیاریست از دکوراتور @Optional() پشت آن استفاده می کنیم.

اطلاعات بیشتر درباره provider ها در بخش های بعدی می خوانیم.

## **Property-based injection—————————-**

تنها زمانی که کلاس ما از یک کلاس مادر extend کرده است و آن کلاس مادر وابستگی هایی دارد بجای پاس دادن آن وابسستگی ها در super() می تونیم از **Property-based injection استفاده کنیم.**

پس زمانی که extend ای نداریم ، حتما باید از تزریق در constructor استفاده کنیم.

## **Provider registration——————————-**

حالا باید این cat provider ای که تعریف کردیم رو به nest بشناسونیم

ضمن اینکه این service ای که تعریف کردیم یه مصرف کننده بنام cat.controller.ts داره.

**app.module.ts————-**

```tsx
import { Module } from '@nestjs/common';
import { CatsController } from './cats/cats.controller';
import { CatsService } from './cats/cats.service';

@Module({
  controllers: [CatsController],
  providers: [CatsService],
})
export class AppModule {}
```

حالا nest می تونه وابستگی cat.controller رو حل کنه

>[!tip]
>پس میشه گفت یکی از وظایف فریمورک ها تامین وابستگی (تزریق وابستگی) هست.

**حالا ساختار پروژه ما نیز به صورت زیر خواهد بود:**

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/5e86db0c-3d3f-4a3d-a68a-5ec0b47f544c/df0c44de-a3bc-436d-936d-361d0e280451/Untitled.png)

## **Manual instantiation——————————-**

به هر دلیلی ممکنه نیاز باشه تا بصورت دستی و نه توسط فریمورک وابستگی تزریق بشه

که در بخش های بعدی در این باره توضیح داده شده.
![](Pasted%20image%2020240228191840.png)