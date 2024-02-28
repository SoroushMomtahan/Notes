این middleware می تونه یه function و یا class باشه که قبل از Router Handler (ها) صدا زده میشه

![img-01.png (1481×314) (raw.githubusercontent.com)](https://raw.githubusercontent.com/SoroushMomtahan/Notes/main/01%20-%20EcmaScript/NestJs/OverView/05%20-%20Middlewares/Images/img-01.png)

این function می تونه به دو object به نام request و response دسترسی داشته باشه و همچنین میتونه به next function هم دسترسی داشته باشه.

می تونیم middleware های شخصی سازی شده ای رو با استفاده از روش `function` ای و یا `class` ای بوجود بیاریم.

**روش کلاسی ——————-**

اگر از روش کلاسی بریم ، باید از دکوراتور تزریقی `@Injectable()` بالای کلاس استفاده کنیم.

همچنین این کلاس باید interface ای به نام `NestMiddleware` رو implement کند.

`logger.middleware.ts`

```tsx
import { Injectable, NestMiddleware } from '@nestjs/common';
import { Request, Response, NextFunction } from 'express';

@Injectable()
export class LoggerMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    console.log('Request...');
    next();
  }
}

```

## **Dependency injection————————-**

درست مانند Controller ها و Provider ها ، به class Middleware ها نیز میشه چیزی تزریق کرد.

## **Applying middleware—————————-**

برخلاف controller ها و provider ها ، در `@Module()` جایی برای middleware ها وجود ندارد.

یک ModuleClass برای اینکه بتونه middleware داشته باشه باید دارای متد `configure()` باشه و `NestModule` interface رو برای خودش پیاده کرده باشه.

```tsx
import { Module, NestModule, MiddlewareConsumer } from '@nestjs/common';
import { LoggerMiddleware } from './common/middleware/logger.middleware';
import { CatsModule } from './cats/cats.module';

@Module({
  imports: [CatsModule],
})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer
      .apply(LoggerMiddleware)
      .forRoutes('cats');
  }
}
```

کلاس MiddlewareConsumer یا مصرف کننده متد هایی رو بصورت `chain` در اختیار ما میزاره

در متد `apply` می تونیم middleware یا middleware هایی که قراره استفاده کنیم رو بدیم (چند middleware با کاما از هم جدا میشن)

متد `forRoutes` هم می تونه string(s) ، controller(s) و یا object ای بگیره که مسیر (path) و نوع request method رو با استفاده از `enum RequestMethod` مشخص می کند.

**`app.module.ts`**

```tsx
import { Module, NestModule, RequestMethod, MiddlewareConsumer } from '@nestjs/common';
import { LoggerMiddleware } from './common/middleware/logger.middleware';
import { CatsModule } from './cats/cats.module';

@Module({
  imports: [CatsModule],
})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer
      .apply(LoggerMiddleware)
      .forRoutes({ path: 'cats', method: RequestMethod.GET });
  }
}
```

پکیج Body-Parser
>[!tip]
>وقتی از Adapter پیش فرض nest یعنی express استفاده می کنیم ، بصورت پیش فرض json و urlencoded از پکیج body-parser رجیستر می شوند. حال اگر بخواهیم خودمان این دو middleware پیش فرض رو شخصی سازی کنیم باید این دو middleware رو با false کردن فلگ bodyParser غیر فعال کنیم.
```tsx
async function bootstrap() {
  const app = await NestFactory.create(AppModule, {bodyParser:false});
  await app.listen(3000);
}
bootstrap();
```

## **Route wildcards—————————-**

داخل متد forRoutes می تونیم از string patern ها استفاده کنیم.

>[!warning]
>اگر از fastify به جای express استفاده می کنیم این string patern ها در قالب پکیج [path-to-regexp](https://github.com/pillarjs/path-to-regexp#parameters) عرض می شوند.

```tsx
forRoutes({ path: 'ab*cd', method: RequestMethod.ALL });
```

## **Middleware consumer——————————-**
این `MiddlewareConsumer` یه `Helper Class` هست و همانطور که قبلا هم گفتیم یه سری متد رو **رای مدیریت middleware**  بصورت زنجیره ای در اختیارمون میزاره.

در متد `apply` می تونیم middleware یا middleware هایی که قراره استفاده کنیم رو بدیم (چند middleware با کاما از هم جدا میشن)

متد `forRoutes` هم می تونه string(s) ، controller(s) و یا object ای بگیره که مسیر (path) و نوع request method رو با استفاده از `enum RequestMethod` مشخص می کند.
```typescript
import { Module, NestModule, MiddlewareConsumer } from '@nestjs/common';
import { LoggerMiddleware } from './common/middleware/logger.middleware';
import { CatsModule } from './cats/cats.module';
import { CatsController } from './cats/cats.controller';

@Module({
  imports: [CatsModule],
})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer
      .apply(LoggerMiddleware)
      .forRoutes(CatsController);
  }
}
```

## **Excluding routes——————————-**
گاهی وقت ها می خوایم روی اکثر مسیر ها و یا Controller ها بجز محدود مسیر و یا controller هایی یه middleware رو set کنیم ، در این شرایط می تونیم از متد `exclude` استفاده کنیم.
```typescript
consumer
  .apply(LoggerMiddleware)
  .exclude(
    { path: 'cats', method: RequestMethod.GET },
    { path: 'cats', method: RequestMethod.POST },
    'cats/(.*)',
  )
  .forRoutes(CatsController);
```
>[!tip]
>متد `exclude` از wild card parameter های پکیج  [path-to-regexp](https://github.com/pillarjs/path-to-regexp#parameters) پشتیبانی میکنه

## **Functional middleware————————-**
اگر نیازی نداریم که به middleware خود**option** ای پاس بدیم و یا **وابستگی** ای تزریق کنیم می تونیم از middleware بصورت function ای استفاده کنیم.
`logger.middleware.ts`
```typescript
import { Request, Response, NextFunction } from 'express';

export function logger(req: Request, res: Response, next: NextFunction) {
  console.log(`Request...`);
  next();
};
```

## **Multiple middleware—————————-**
همونطور که قبلا هم گفته شد ، می تونیم به متد `apply()` چندین middleware پاس بدیم.
```typescript
consumer.apply(cors(), helmet(), logger).forRoutes(CatsController);
```

## **Global middleware—————————-**
اگه بخوایم یه middleware برای همه route های register شده اعمال بشه می تونیم با استفاده از متد `use()` در فایل main.ts این کار رو انجام بدیم:
`main.ts`
```typescript
const app = await NestFactory.create(AppModule);
app.use(logger);
await app.listen(3000);
```
>[!tip]
>لازم به ذکره که متد `use()` تنها function middleware قبول می کنه ، بنابراین اگر به تزریق وابستگی نیاز داریم و به طبع از class Middleware استفاده می کنیم متد use قادر به دریافت اون نیست.
>بنابراین در صورتی که از class middleware استفاده می کنیم و می خواهید class middleware ای که ساخته اید بصورت global اعمال شود راهکار استفاده از متد `forRoutes('*')` هست:
```typescript
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer
      .apply(LoggerMiddleware)
      .forRoutes('*');
  }
}
```
