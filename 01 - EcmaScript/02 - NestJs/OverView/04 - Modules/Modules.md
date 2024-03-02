ماژول یک کلاسه که بالاش دکوراتور `@Module()` وجود داره.

این @Module() ابرداده (meta data) هایی می سازد که به ساختار بندی برنامه های nest ای کمک می کند.

![](./Images/Pasted%20image%2020240228201324.png)

هر برنامه دست کم یک ماژول رو داره و اونم ماژول ریشه (root module) هست.

این root module نقطه شروعی هست که nest بواسطه اون گراف برنامه (application graph ) رو میسازه و بواسطه همین root module ارتباط و وابستگی میان دیگر module ها و provider ها رو پیدا میکنه.

این module ها برای سازماندهی component های برنامه ما بسیار کاربردی اند.

پس یک برنامه می تونه شامل چندین ماژول باشه که درون هر ماژول **مجموعه از قابلیت های مرتبط** با هم وجود داره.

در واقع میشه گفت مجموعه ای از قابلیت های مرتبط با هم رو پیچاندیمشون دور یه چیزی به اسم ماژول.

دکوراتور `@Module()` یه object میگیره که این object خودش property هایی داره که module مورد نظر رو توصیف میکنه:

|Module Properties|Descriptions|
|---|---|
|providers|شامل مجموعه ای از عرضه کنندگان هست که سیستم تزریق کننده نست وظیفه نمونه سازی و به اشتراک گذاری عرضه کنندگان حداقل در سراسر ماژول را دارد ، در واقع سیستم تزریق کننده نست همون وظیفه تزریق این عرضه کنندگان حداقل در سراسر آن ماژول رو دارد.|
|controllers|شامل مجموعه ای از کنترل کننده هاست که سیستم نست باید از آنها نمونه بسازد|
|imports|شامل لیستی از ماژول های دیگر است که عرضه کنندگان صادر شده ایی دارند که در ماژول فعلی مورد نیاز هست|
|exports|شامل زیر مجموعه ای از عرضه کنندگان ماژول فعلی که لازمه در ماژول های دیگر نیز استفاده شوند|


ماژول ها می تونن provider های داخلی خود را export کنند ، با این کار ماژول های دیگر می تونن به Provider های export شده آن ماژول هنگام import دسترسی داشته باشند.

>[!tip]
>وقتی ماژولی رو در ماژول دیگری import می کنیم ، اگر آن ماژول provider های export شده ای در قسمت export اش داشته باشد ، این provider ها در ماژول مقصد قابل استفاده خواهند بود.

>[!tip]
>پس نتیجه می شود که یک provider یا باید در imports های یک module باشد و یا در providers ها ؛ در غیر این صورت nest توانایی تامین این provider ها رو نخواهد داشت.

## **Feature modules————————-**

از آنجایی که catsController و catsService ارتباط نزدیکی با هم دارند و به نوعی از یک دامنه هستند ، بهتر آنها را به یک `feature module` انتقال بدیم.

`cats/cats.module.ts`

```tsx
import { Module } from '@nestjs/common';
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';

@Module({
  controllers: [CatsController],
  providers: [CatsService],
})
export class CatsModule {}
```

> برای ایجاد ماژول بوسیله `CLI` از دستور زیر استفاده می کنیم:

```tsx
$ nest g module cats
```

حالا باید این ماژول رو به root module بشناسونیم تا سیستم نست بتونه از اون استفاده کنه:

`app.module.ts`

```tsx
import { Module } from '@nestjs/common';
import { CatsModule } from './cats/cats.module';

@Module({
  imports: [CatsModule],
})
export class AppModule {}
```

![](./Images/Pasted%20image%2020240228195149.png)

## **Shared modules—————————-**

**هر module بصورت پیش فرض shared module است**

در nest بصورت پیش فرض module ها singleton اند بدین ترتیب در زمان استفاده از یک provider می دانیم که تنها یک نمونه از provider مربوطه ساخته شده است و در اختیار تزریق شوندگان قرار گرفته است.

![](./Images/Pasted%20image%2020240228195302.png)

>[!tip]
> مثلا در تصویر بالا تنها یک نمونه از provider های shared module در اختیار دیگر module ها قرار گرفته است.

این نکته حائز اهمیته که **هر module بصورت پیش فرض shared module است** ؛ یعنی از ماژول ساخته شده می تونیم در هر ماژول دیگری استفاده کنیم.

اما زمانی که بخواهیم از provider های یک ماژول در ماژولی دیگر که آن را import کرده ، استفاده کنیم ، provider مربوطه در ماژول مبدا باید ابتدا export شده باشند.

مثلا فرض کنید می خواهیم نمونه ساخته شده از catsService رو بین چند ماژول که دارند catsModule رو import می کنن به اشتراک بزاریم ، اولین کاری که باید انجام بدیم ، export کردن catsSerivce در ماژول catsModule هست.

```tsx
import { Module } from '@nestjs/common';
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';

@Module({
  controllers: [CatsController],
  providers: [CatsService],
  exports: [CatsService]
})
export class CatsModule {}
```

حالا اگه catsModule رو در ماژول دیگری در قسمت import اش بنویسیم بدین ترتیب می تونیم به CatService دسترسی داشته باشیم.

## **Module re-exporting————————-**

همانطور که در قسمت های قبل فهمیدیم یه ماژول می تونه Provider های داخلی شو (Internal Provider) export کنه ، علاوه بر این یک module میتونه Module هایی که import کرده رو هم export کنه تا با این کار بتونه از طریق خودش Provider های ماژول های دیگه رو به اشتراک بزاره.

```tsx
@Module({
  imports: [CommonModule],
  exports: [CommonModule],
})
export class CoreModule {}

```

## **Dependency injection—————————-**

این ModuleClass ها خودشون نمی تونن به عنوان تزریق شونده تزریق شوند به علت Circular Dependency اما میشه به ماژول ها چیزی تزریق کرد.(نمی توان ماژول ها رو به جایی تزریق کرد اما می توان به ماژول ها چیزی تزریق کرد)

`cats.module.ts`

```tsx
import { Module } from '@nestjs/common';
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';

@Module({
  controllers: [CatsController],
  providers: [CatsService],
})
export class CatsModule {
  constructor(private catsService: CatsService) {}
}
```

## **Global modules—————————-**

اگه نیاز داریم تا یه ماژول رو در ماژول های زیادی import کنیم تا به واسطه این import به provider های آن ماژول دسترسی پیدا کنیم ، بجای این کار می تونیم بالای ماژولی که در بسیاری از ماژول های دیگر قرار استفاده بشه دکوراتور @Global() قرار بدیم .

با این کار ماژول های دیگر بدون import این ماژول هم می تونن به provider های ماژول global شده ، دسترسی پیدا کنند ، البته در صورتی که ماژول global شده Provider هاشو export کرده باشه (مثل کد پایین که CatModule درسته که Global شده اما برای اینکه بقیه بتونن به CatsService ش دسترسی پیدا کنن اومده CatsService رو export کرده)

>[!tip]
>لازم به ذکر است بهترین راه استفاده از provider های یک module همان import کردن آن در ماژول مربوطه است و این روش یعنی Global کردن ماژول ، زیاد پیشنهاد نمیشه.

```tsx
import { Module, Global } from '@nestjs/common';
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';

@Global()
@Module({
  controllers: [CatsController],
  providers: [CatsService],
  exports: [CatsService],
})
export class CatsModule {}
```

این نکته نیز حائز اهمیته که provider های Angular بصورت global رجیستر می شوند یعنی یکبار تعریف می شوند و دیگه در همه جای پروژه در دسترس خواهند بود.

>[!tip]
>به ماژولی که global شده `module global-scoped` می گویند . این دسته از ماژول ها تنها باید یکبار register شوند و بعد از آن همه جا در دسترس اند. این یکبار رجیستر شدن می تونه در root module اتفاق بیفته و یا هر جای دیگه البته بسته به ساختار پروژه

## **Dynamic modules—————————**

با استفاده از این ویژگی میشه به راحتی ماژول های قابل تنظیمی ایجاد کرد که میتونه بصورت دینامیک register بشه و Provider ها رو پیکربندی کنه

این بخش بصورت کامل در قسمت های بعدی توضیح داده شده.

```tsx
import { Module, DynamicModule } from '@nestjs/common';
import { createDatabaseProviders } from './database.providers';
import { Connection } from './connection.provider';

@Module({
  providers: [Connection],
})
export class DatabaseModule {
  static forRoot(entities = [], options?): DynamicModule {
    const providers = createDatabaseProviders(options, entities);
    return {
      module: DatabaseModule,
      providers: providers,
      exports: providers,
    };
  }
}
```

ادامه دارد ….

ادامه ش نوشته شود.