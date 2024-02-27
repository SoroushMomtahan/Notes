این middleware می تونه یه function و یا class باشه که قبل از Router Handler (ها) صدا زده میشه

![Alt Middleware](https://raw.githubusercontent.com/SoroushMomtahan/Notes/main/Public/Images/EcmaScript/NestJs/OverView/01.png?raw=true)

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

> پکیج Body-Parser
> 
> <aside> 💡 وقتی از Adapter پیش فرض nest یعنی express استفاده می کنیم ، بصورت پیش فرض json و urlencoded از پکیج body-parser رجیستر می شوند. حال اگر بخواهیم خودمان این دو middleware پیش فرض رو شخصی سازی کنیم باید این دو middleware رو با false کردن فلگ bodyParser غیر فعال کنیم.
> 
> </aside>

```tsx
async function bootstrap() {
  const app = await NestFactory.create(AppModule, {bodyParser:false});
  await app.listen(3000);
}
bootstrap();
```

## **Route wildcards—————————-**

داخل متد forRoutes می تونیم از string patern ها استفاده کنیم.

اگر از fastify به جای express استفاده می کنیم این string patern ها در قالب پکیج [path-to-regexp](https://github.com/pillarjs/path-to-regexp#parameters) عرض می شوند.

```tsx
forRoutes({ path: 'ab*cd', method: RequestMethod.ALL });
```

## **Middleware consumer——————————-**

## **Excluding routes——————————-**

## **Functional middleware————————-**

## **Multiple middleware—————————-**

## **Global middleware—————————-**