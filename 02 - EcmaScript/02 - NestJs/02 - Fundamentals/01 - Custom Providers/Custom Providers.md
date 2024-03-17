>[!tip]
>در این بخش:
>- مبانی ***تزریق وابستگی*** رو یاد آوری می کنیم.
>- یاد میگیریم که چطوری یه ***توکن*** رو به provider ای مرتبط کنیم.
>- یاد می گیریم که با استفاده از سه سینتکس `useClass` و `useValue` و `useFactory` و `useExisting` عرضه کنندگان (provider) ***سفارشی*** بسازیم.
>- یاد میگیریم که provider ها تنها برای service ها کاربرد ندارند یعنی provider ها علاوه بر اینکه می تونن service ای رو return کنند ، می تونن ***هر چیز دیگه ای*** رو هم return کنند و مقدار return شده به جایی که باید تزریق بشه.
>- یاد میگیریم چطور custom provider ها رو از یک ماژول ***export*** کنیم (با دو روش).

در بخش overview و قسمت provider درباره چیستی provider ها و نحوه استفاده و به نوعی تزریق آنها آشنا شدیم ، در این بخش بصورت مفصل تر درباره provider ها می خونیم و پرونده ش رو بطور کامل می بندیم اما قبلش مروری برمواردی که در بخش overview و قسمت provider خواندیم ، می کنیم.

## DI Fundamentals----------------
مبانی تزریق وابستگی------------

![](./Images/Pasted%20image%2020240311121110.png)

تزریق وابستگی یه تکنیک از مبحث وارونگی کنترل (`IOC`) هست که در آن وظیفه نمونه سازی از کلاس های مورد نیاز رو برعهده `IOC Container` می گذارید ؛ IOC Container ظرفی هست که کلاس هایی که می خوایم ازشون نمونه ساخته بشه رو بهش می دیم و برامون نمونه میسازه

در گام اول یه provider تعریف کردیم:

`cats.service.ts`
```typescript
import { Injectable } from '@nestjs/common';
import { Cat } from './interfaces/cat.interface';

@Injectable()
export class CatsService {
  private readonly cats: Cat[] = [];

  findAll(): Cat[] {
    return this.cats;
  }
}
```

سپس controller ای که به provider ای که تعریف کردیم نیازمند بود ، نیازمندی (وابستگی) شو در constructor اعلام می کرد:

`cats.controller.ts`
```typescript
import { Controller, Get } from '@nestjs/common';
import { CatsService } from './cats.service';
import { Cat } from './interfaces/cat.interface';

@Controller('cats')
export class CatsController {
  constructor(private catsService: CatsService) {}

  @Get()
  async findAll(): Promise<Cat[]> {
    return this.catsService.findAll();
  }
}
```

در گام آخر هم provider مربوطه رو در IOC Container ثبت (Register) کردیم:

`app.module.ts`
```typescript
import { Module } from '@nestjs/common';
import { CatsController } from './cats/cats.controller';
import { CatsService } from './cats/cats.service';

@Module({
  controllers: [CatsController],
  providers: [CatsService],
})
export class AppModule {}
```

در 3 قسمت بالا به ترتیب چه اتفاقی می افتد ؟

1- در `cats.service.ts` دکوراتور `@Injectable()` باعث میشه  IOC Container بتونه کلاس `CatsService` رو به عنوان یه کلاس تزریقی مدیریت کنه.

2- در `cats.conteroller.ts` کلاس `CatController` وابستگی خودش رو به تزریق `cats.service.ts` در سازنده با استفاده از **توکن** `CatsService` اعلام میکنه.

```typescript
 constructor(private catsService: CatsService)
```

3- در `app.module.ts` ، توکن `CatsService` رو به کلاس `CatsService` نسبت دادیم (در ادامه همین قسمت درباره چگونه وابسته کردن توکن به کلاس ، بیشتر می خونیم)

حالا وقتی IOC Container می خواد از `CatsController` نمونه بسازه ، اول به نیازمندی های این نمونه نگاه میکنه ، CatsController نیازمندی شو با توکن `CatsService` مشخص کرده ، IOC Container داخل provider های موجود میگیرده ببینه که اولا همچین توکنی وجود داره و دوما اگه وجود داره چه چیزی بهش گره خورده (مثلا در این مثال میبینه که توکن مورد نظر وجود داره و این توکن با کلاس `CatsController` گره خورده) حال با فرض اینکه در حالت `SINGLTON` قرار داریم ، IOC Container Nest یا یه نمونه از کلاس `CatsService` میسازه و `cache` می کنه و یا اگر نمونه از قبل ساخته شده بود cache این نمونه موجود رو برمیگردونه.

فرآیند تامین وابسته ها بسیار پیچیده هست یه گراف وابستگی ایجاد میشه ، مثلا در مثال پایین اگه خود `CatsService` نیازمند یه سری وابستگی باشه ، اول باید این وبستگی ها تامین بشه ، گراف وابستگی (**Dependency Graph**) این تضمین رو میده که وابستگی ها از  **پایین به بالا** تامین بشن.

## Standard Providers----------------


بیاییم به دکوراتور `@Module()` درون `app.module.ts` دقیق تر نگاه کنیم:

```typescript
@Module({
  controllers: [CatsController],
  providers: [CatsService],
})
```

این property ی provider آرایه ای از provider ها رو میگیره ؛ تا حالا provider ها رو براساس نام کلاس هاشون به این آرایه دادیم اما در واقع syntax ای که تا حالا ازش استفاده می کردیم یه short-hand برای syntax پایین بوده:

```typescript
providers: [
  {
    provide: CatsService,
    useClass: CatsService,
  },
];
```

در ساز و کار بالا ، توکن `CatsService`  رو به کلاس `CatsService` نسبت دادیم ، اون حرکتی که قبلا میزدیم ، خلاصه شده ی این حرکت برای زمانی هست که نام توکن با نام کلاس یکسانه.

## non-class based provider token------------

تا اینجا از نام کلاس برای تعریف نام توکن ها استفاده کردیم ؛ این الگو دقیقا برای زمانی کاربرد داره که نام توکن با نام کلاس یکسانه (constructor based injection) اما بعضی وقت ها شما می خواید بصورت داینامیک تر ، یعنی با استفاده از یک `string-value` ، توکن وابستگی رو مشخص کنید.

```typescript
import { connection } from './connection';

@Module({
  providers: [
    {
      provide: 'CONNECTION',
      useValue: connection,
    },
  ],
})
export class AppModule {}
```

>[!tip]
>اگر از string-value برای ایجاد توکن استفاده می کنید می توان از `enum` و یا `symbols` هم استفاده کرد.

قبلا درباره نحوه اعلام نیازمندی به یک provider در روش استاندارد صحبت کردیم :

```ts
 constructor(private catsService: CatsService)
```

حالا زمانی که از `string-value` برای ایجاد توکن استفاده می کنیم ، نحوه اعلام نیازمندی به اون توکن هم فرق می کنه و در این شرایط باید از دکوراتور `@Inject()` استفاده کنیم:

```typescript
@Injectable()
export class CatsRepository {
  constructor(@Inject('CONNECTION') connection: Connection) {}
}
```

برای تمیز تر شدن کدمون بهتره این `stirng-value` ها رو در یک فایل جدا مانند `constants.ts` تعریف کنیم.

```typescript
export const CONFIG_OPTIONS = 'CONFIG_OPTIONS';
```
## Custom Providers-----------------

حالا اگه خواسته های ما فراتر از استانداردی که برای provider ها ارئه شده ، باشد ، چه کنیم ؟

در زیر چند نمونه از خواسته ها رو نوشتیم:

- زمانی می خواهید یک نمونه شخصی سازی شده رو با نمونه ای که nest میسازه جایگزین کنید.
- زمانی که می خواهید از یک کلاس یکسان ، دارای دو توکن متفاوت باشد.
- زمانی که می خواید کلاس مربوطه رو با یه mock version از کلاس (برای کار های testing) جایگزین کنید.

سیستم nest برای این موارد custom provider ها رو ارائه داده

### Value Provider : `useValue`

سینتکس `useValue` برای زمانی که می خواهیم یک ثابت رو تزریق کنیم ، یک object از کلاسی رو برای کار های تستی تزریق کنیم و یا از یک لایبرری خارجی در کانتینر nest استفاده کنیم ، کاربرد داره

```typescript
import { CatsService } from './cats.service';

const mockCatsService = {
  /* mock implementation
  ...
  */
};

@Module({
  imports: [CatsModule],
  providers: [
    {
      provide: CatsService,
      useValue: mockCatsService,
    },
  ],
})
export class AppModule {}
```

>[!tip]
>در این مثال از شی `mockCatsService` استفاده کردیم که interface ای یکسان با کلاس `CatsService` دارد ، بنابراین با استفاده از این روش حتی می تونیم نمونه ای از یک کلاس رو هم به useValue نسبت بدیم.


### Class Providers : `UseClass`

سینتکس `useClass` اجازه میده تا بتونیم بصورت داینامیک class ای که می خوایم رو به توکن مربوطه نسبت بدیم.

```typescript
const configServiceProvider = {
  provide: ConfigService,
  useClass:
    process.env.NODE_ENV === 'development'
      ? DevelopmentConfigService
      : ProductionConfigService,
};

@Module({
  providers: [configServiceProvider],
})
export class AppModule {}
```

### Factory Providers : `useFactory`

این syntax یعنی `useFactory` اجازه میده تا بتونیم provider های داینامیکی رو بسازیم.

در واقع provider های واقعی یه متد `useFactory` دارند که چیزی رو return میکنه و در واقع همین چیزی که return میشه تزریق میشه به جایی که نیازه:

```ts
const myProviderFactory={  
  provide: 'MY_PROVIDER',  
  useFactory: ()=>{  
    return `this is my Provider factory`;  
  }  
}  
@Module({  
  controllers:[CatsController],  
  providers:[myProviderFactory]  
})  
export class CatsModule {  
  
}
```

```ts
@Controller('cats')  
export class CatsController {  
  
  constructor(@Inject('MY_PROVIDER') private readonly value:string) {  
  }  @Get()  
  findOne() {  
    return this.value  // this is my Provider factory
  }  
}
```

یه factory function ساده می تونه به provider های دیگه نیاز نداشته باشه و در مقایل یه factory function کمی پیچیده تر میتونه نیاز به provider هایی داشته باشه 

سینتکس factory function می تونه به صورت زیر باشه :
- یه property به اسم `useFactory` که یه متده و میتونه پارامتر ورودی تحت عنوان provider داشته باشه (برای زمانی که provider مربوطه خود نیازمند provider های دیگریست)
- یه property به اسم `inject` که اختیاری هست و پارامتر های ورودی متد `useFactory` رو تامین میکنه.

```typescript
const connectionProvider = {
  provide: 'CONNECTION',
  useFactory: (optionsProvider: OptionsProvider, optionalProvider?: string) => {
    const options = optionsProvider.get();
    return new DatabaseConnection(options);
  },
  inject: [OptionsProvider, { token: 'SomeOptionalProvider', optional: true }],
  //       \_____________/            \__________________/
  //        This provider              The provider with this
  //        is mandatory.              token can resolve to `undefined`.
};

@Module({
  providers: [
    connectionProvider,
    OptionsProvider,
    // { provide: 'SomeOptionalProvider', useValue: 'anything' },
  ],
})
export class AppModule {}
```

### Alias Provider : `useExisting`
ارائه دهنده با نام مستعار-----------

سینتکس `useExisting` اجازه میده تا بتونیم از provider موجود در قالب یه نام دیگه استفاده کنیم(یه نام دیگه برای provider موجود بگذاریم) 

این روش دو راه برای دسترسی به یک provider یکسان بوجود میاره

```typescript
@Injectable()
class LoggerService {
  /* implementation details */
}

const loggerAliasProvider = {
  provide: 'AliasedLoggerService',
  useExisting: LoggerService,
};

@Module({
  providers: [LoggerService, loggerAliasProvider],
})
export class AppModule {}
```

اگه هر دو وابستگی singleton scope باشند یک نمونه برای هر دو در طول برنامه ساخته میشه.

## non-service based provider

با اینکه در اکثر موارد provider ها برای service ها عرضه میشن اما هیچ محدودیتی در این زمینه وجود نداره و provider ها می تونن هر value رو return کنن:

```typescript
const configFactory = {
  provide: 'CONFIG',
  useFactory: () => {
    return process.env.NODE_ENV === 'development' ? devConfig : prodConfig;
  },
};

@Module({
  providers: [configFactory],
})
export class AppModule {}
```

## Export Custom Providers

برای export کردن custom provider ها یا میشه از نام توکن و یا نام full provider object که شی custom provider در اون تعریف شده استفاده کرد:

1- نام توکن

```typescript
const connectionFactory = {
  provide: 'CONNECTION',
  useFactory: (optionsProvider: OptionsProvider) => {
    const options = optionsProvider.get();
    return new DatabaseConnection(options);
  },
  inject: [OptionsProvider],
};

@Module({
  providers: [connectionFactory],
  exports: ['CONNECTION'],
})
export class AppModule {}
```

2- نام شی provider

```typescript
const connectionFactory = {
  provide: 'CONNECTION',
  useFactory: (optionsProvider: OptionsProvider) => {
    const options = optionsProvider.get();
    return new DatabaseConnection(options);
  },
  inject: [OptionsProvider],
};

@Module({
  providers: [connectionFactory],
  exports: [connectionFactory],
})
export class AppModule {}
```

