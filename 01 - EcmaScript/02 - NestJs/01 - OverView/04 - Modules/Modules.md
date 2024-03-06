ماژول یک کلاسه که بالاش دکوراتور `@Module()` وجود داره.

این @Module() ابرداده (meta data) هایی به class Module اضافه می کند و با این کار قابلیت های کلاس module رو افزایش می دهد.

![](Pasted%20image%2020240228201324.png)

هر برنامه دست کم یک ماژول رو داره و اونم ماژول ریشه (`root module`) هست.

این root module نقطه شروعی هست که nest بواسطه اون گراف برنامه (application graph ) رو میسازه و بواسطه همین root module ارتباط و وابستگی میان دیگر module ها و provider ها رو پیدا میکنه.

این module ها برای سازماندهی component های برنامه ما بسیار کاربردی اند.

پس یک برنامه می تونه شامل چندین ماژول باشه که درون هر ماژول **مجموعه از قابلیت های مرتبط** با هم وجود داره که به این قابلیت های مرتبط با هم component گوییم ، در واقع component های مرتبط با هم رو میپیچیمشون دور یه چیزی به اسم ماژول

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

از آنجایی که catsController و catsService ارتباط نزدیکی با هم دارند و به نوعی از یک دامنه هستند ، بهتره آنها را به یک `feature module`  انتقال بدیم feature module ها اون دسته از feature (قابلیت) هایی که مرتبط با هم اند رو سازماندهی میکنه ، بخاطر همینه که اسمش **feature** module هست.

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

برای ایجاد ماژول بوسیله `CLI` از دستور زیر استفاده می کنیم:

```tsx
$ nest generate | g module | mo [name(e.g cats)] 
```

حالا باید این ماژول رو به root module بشناسونیم تا سیستم nest بتونه اونو شناسایی کنه ازش استفاده کنه:

`app.module.ts`
```tsx
import { Module } from '@nestjs/common';
import { CatsModule } from './cats/cats.module';

@Module({
  imports: [CatsModule],
})
export class AppModule {}
```

![](Pasted%20image%2020240228195149.png)

## **Shared modules—————————-**

**هر module بصورت پیش فرض shared module است**

در nest بصورت پیش فرض module ها singleton اند بدین ترتیب در زمان استفاده از یک provider می دانیم که تنها یک نمونه از provider مربوطه ساخته شده است و در اختیار تزریق شوندگان قرار گرفته است.

به بیان دیگر تنها یک نمونه از ماژول مربوطه در کل پروژه **Shared** می شود ، به همین خاطره که میگوییم ماژول ها در nest بصورت پیش فرض shared Module اند که البته می تونیم این ویژگی ماژول ها رو تغییر بدیم.

![](Pasted%20image%2020240228195302.png)

>[!tip]
> مثلا در تصویر بالا تنها یک نمونه از provider های shared module ای فرضی در اختیار دیگر module ها قرار گرفته است.

این نکته حائز اهمیته که **هر module بصورت پیش فرض shared module است** ؛ یعنی از ماژول ساخته شده می تونیم در هر ماژول دیگری استفاده کنیم.

اما زمانی که بخواهیم از provider های یک ماژول در ماژولی دیگر که آن را import کرده ، استفاده کنیم ، provider مربوطه در ماژول مبدا باید ابتدا export شده باشد.

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

حالا اگه catsModule رو در ماژول دیگری در قسمت import اش بنویسیم بدین ترتیب می تونیم به CatsService دسترسی داشته باشیم.

## **Module re-exporting————————-**

همانطور که در قسمت های قبل فهمیدیم یه ماژول می تونه Provider های داخلی شو (Internal Provider) export کنه ، علاوه بر این یک module میتونه Module هایی که import کرده رو هم export کنه تا با این کار بتونه از طریق خودش Provider های ماژول های دیگه رو به اشتراک بزاره به این کار re-exporting (exxport دوباره) ماژول گویند.

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
  constructor(private CommonService: CommonService) {}
}
```

در مثال بالا به هر دلیلی ممکنه CatsModule به `CommonService` ای نیاز داشته باشه بنابراین در constructor این مازول اعلام کردیم که به همچین سرویسی نیازمندیم.
## **Global modules—————————-**

اگه نیاز داریم تا یه ماژول رو در ***ماژول های زیادی*** import کنیم تا به واسطه این import به provider های آن ماژول دسترسی پیدا کنیم ، بجای این کار می تونیم بالای ماژولی که در بسیاری از ماژول های دیگر قرار استفاده بشه دکوراتور `@Global()` قرار بدیم .

با این کار ماژول های دیگر بدون import این ماژول هم می تونن به provider های ماژول global شده ، دسترسی پیدا کنند ، البته در صورتی که ماژول global شده Provider هاشو export کرده باشه (مثل کد پایین که CatModule درسته که Global شده اما برای اینکه بقیه بتونن به CatsService ش دسترسی پیدا کنن اومده CatsService رو `export` کرده)

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

این نکته نیز حائز اهمیته که provider های Angular بصورت global رجیستر می شوند یعنی یکبار تعریف می شوند و دیگه در همه جای پروژه در دسترس خواهند بود اما در nest اینگونه نیست.

>[!tip]
>به ماژولی که global شده `module global-scoped` می گویند . این دسته از ماژول ها تنها باید یکبار register شوند و بعد از آن همه جا در دسترس اند. این یکبار رجیستر شدن می تونه در root module اتفاق بیفته و یا هر جای دیگه البته بسته به ساختار پروژه

## **Dynamic modules—————————**

با استفاده از این ویژگی میشه به راحتی ماژول های سفارشی ای رو ایجاد کرد که می تونن register (ثبت) بشن  و Provider ها شون رو بصورت داینامیک (پویا) پیکربندی کنند.

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

در واقع همون property هایی رو که توسط دکوراتور `@Module()` تنظیم میکردیم رو داریم توسط متد `forRoot()` تنظیم می کنیم ، با این تفاوت که یه سری پیکربندی هایی رو هم در نظر می گیریم

این ماژول ما `DatabaseModule` بصورت پیش فرض provider ای به اسم connection داره اما علاوه بر این با توجه به entities و options ای که به متد forRoot پاس داده می شود تعداد دیگری provider نیز expose (افشا) می شود یعنی در واقع return ای های متد forRoot میاد provider های پیش فرض که در دکوراتور `@Module()` تعریف شده اند رو گسترش (extend) میده.

 That's how both the statically declared `Connection` provider **and** the dynamically generated repository providers are exported from the module.

عبارت بالا یعنی میگه connect هم export میشه ؟


اگر می خواید dynamic module رو global تعریف شود ، property ای به اسم `global` رو `true` می گذاریم:

```typescript
{
  global: true,
  module: DatabaseModule,
  providers: providers,
  exports: providers,
}
```

حالا به شکل زیر می تونیم از این dynamic module در جایی که می خوایم استفاده کنیم:

```typescript
import { Module } from '@nestjs/common';
import { DatabaseModule } from './database/database.module';
import { User } from './users/entities/user.entity';

@Module({
  imports: [DatabaseModule.forRoot([User])],
})
export class AppModule {}
```

>[!tip]
>در کد بالا `DatabaseModule` رو imports کردیم و همچنین اونو پیکربندی هم کردیم

اگه بخوایم دوباره این ماژول رو export کنیم تا بشه در دیگر ماژول ها ازش استفاده کرد دیگر نیازی به تنظیم configuration نیست:

```typescript
import { Module } from '@nestjs/common';
import { DatabaseModule } from './database/database.module';
import { User } from './users/entities/user.entity';

@Module({
  imports: [DatabaseModule.forRoot([User])],
  exports: [DatabaseModule],
})
export class ExampleModule {}
```

با این کار همانوطر که می دونید هر ماژولی که ExampleModule رو import کنه به provider های export شده این ماژول (در این مثال DatabaseModule) دسترسی خواهد داشت.

می تونیم dynamic module های سفارشی هم با استفاده از `ConfigurableModuleBuilder` بسازیم که در بخش مربوطه درباره آن صحبت شده.

