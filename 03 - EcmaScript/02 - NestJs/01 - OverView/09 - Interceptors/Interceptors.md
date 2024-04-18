>[!tip]
>**کلمات کلیدی این بخش:**
>`@Injectable()`
>
>`NestInterceptor`
>
>`intercept()`
>
>`CallHandler`
>
>`@UseInterceptors()`
>
>`useGlobalInterceptors()`
>
>`APP_INTERCEPTOR`

---

این Interceptor یه کلاس هست که که بالاش دکوراتور `@Injectable()` وجود داره و اینترفیس `NestInterceptor` رو پیاده سازی کرده.

![](Pasted%20image%2020240305181051.png)

این interceptors مجموعه ای از توانایی های کاربردی ای رو داره که از برنامه نویسی جنبه گرا [Aspect Oriented Programming](https://en.wikipedia.org/wiki/Aspect-oriented_programming) (AOP)  برای پیاد سازی این توانایی ها الهام گرفته.

برخی از این توانایی ها عبارت اند از:
- اتصال منطق اضافی (extra logic) به قبل / بعد متد اجرا شده.
- تبدیل نتیجه return شده از متد 
- تبدیل exception پرتاب شده از متد
- بسط (گسترش) دادن رفتار متد
- بسته به شرایط خاص یک متد رو به طور کامل لغو کنید (برای اهداف cache)
## Basics-----------------

هر interceptor باید متد `intercept()` رو پیاده سازی کنه. این متد دو تا آرگومان ورودی داره که آرگومان اول نمونه ای از کلاس `ExecutionContext` هست که این کلاس از `ArgumentHost` ارث بری کرده و چند تا helper function به متد های کلاس اصلی یعنی ArgumentHost اضافه کرده.
## Execution Context-----------------

همانطور که بالا هم گفتیم ، `ExecutionContext` از `ArgumentHost` ارث بری کرده و چندین helper function اضافه کرده که باعث میشه بتونیم interceptor های عمومی رو که بتونه طیف گسترده ای از controller ها و متد ها رو ساپورت کنه بسازیم.
## Call Handler-----------------

پارامتر دوم متد `intercept()` نمونه ای از `CallHandler` است که دارای متدی به اسم `handler()` هست.
با استفاده از این متد می تونیم route handler مربوطه رو در نقطه ای که می خواهیم صدا بزنیم.

اگه در بدنه متد `intercept()` متد `handler()` از `CallHandler` رو صدا نزنیم ، route handler مربوطه هیچگاه اجرا نمیشه.

رویکرد interceptor به این صورته که جریان request / response و بطور کلی route handler مربوطه رو دور خودش می پیچه ، بنابراین شما می تونین قبل و بعد از route handler مربوطه یه سری logic رو اضافه کنید.

در واقع نتیجه route handler مربوطه توسط متد handler() دریافت میشه و با استفاده از عملگر های قدرتمند [RxJs ](https://github.com/ReactiveX/rxjs)می تونیم دستکاری بیشتری روی پاسخ ای که route handler مربوطه برگردونده انجام بدیم.

## Aspect Interception-----------------

مثلا فرض کنید می خواهیم از تعامل کاربر با یک route handler بیایم و بوسیله interceptor ها log بگیریم:

`logging.interceptor.ts`
```typescript
import { Injectable, NestInterceptor, ExecutionContext, CallHandler } from '@nestjs/common';
import { Observable } from 'rxjs';
import { tap } from 'rxjs/operators';

@Injectable()
export class LoggingInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    console.log('Before...');

    const now = Date.now();
    return next
      .handle()
      .pipe(
        tap(() => console.log(`After... ${Date.now() - now}ms`)),
      );
  }
}
```

در اینترفیس `NestInterceptor<T, R>` این T نشان دهنده تایپ خروجی کنترل مربوطه (خروجی متد handler()) `Observable<T>` و R نشان دهنده تایپ خروجی intereceptor هست.

بدلیل اینکه خروجی متد handler() یک observable هست ، ما عملگر های گسترده ای برای دستکاری این جریان داریم.

در مثال بالا از عملگر `tap()` استفاده کردیم که یک تابع logging رو بعد از invoke شدن route handler ما اجرا میکنه ؛ این عملگر در فرآیند request / response تداخلی ایجاد نمیکنه.

## Binding Interceptors-----------------

برای اتصال interceptor ای که نوشتیم به route handler مربوطه از دکوراتور `@UseInterceptors()` استفاده می کنیم

مانند pipe ها و guard ها interceptor ها می تونن controller-scoped, method-scoped, or global-scoped باشند.

`cats.controller.ts`
```typescript
@UseInterceptors(LoggingInterceptor)
export class CatsController {}
```

با کاری که بالا انجام دادیم ، تمام router handler هایی که داخل CatsController تعریف شده از `LoggingInterceptor` استفاده می کنند ، به عنوان مثال اگه کاربری GET / cats رو صدا بزنه ، عبارت پایین در ترمینال نمایش داده میشه:

```typescript
Before...
After... 1ms
```

می تونیم نمونه ای از کلاس LoggingInterceptor که نوشتیم رو هم پاس بدیم:

`cats.controller.ts`
```typescript
@UseInterceptors(new LoggingInterceptor())
export class CatsController {}
```

بصورت global هم می تونیم از یه interceptor استفاده کنیم:

```typescript
const app = await NestFactory.create(AppModule);
app.useGlobalInterceptors(new LoggingInterceptor());
```

باید به این نکته توجه کرد که در متد `useGlobalInterceptors` نمی تونیم از تزریق استفاده کنیم و برای اینکار باید interceptor در ماژول تعریف بشه:

`app.module.ts`
```typescript
import { Module } from '@nestjs/common';
import { APP_INTERCEPTOR } from '@nestjs/core';

@Module({
  providers: [
    {
      provide: APP_INTERCEPTOR,
      useClass: LoggingInterceptor,
    },
  ],
})
export class AppModule {}
```

## Response Mapping-----------------

دستکاری response ارسال شده از route handler به متد handler()

می دونیم که خروجی متد handler() همون خروجی route handler ما هستند و یه observable هست ، بنابراین می تونیم براحتی با استفاده از عملگر `map()` از RxJs این response رو دستکاری کنیم:

>[!warning]
>اگر از @Res() برای ارسال response در route handler مربوطه استفاده کرده باشیم ، دیگه نمی تونیم از عملگر map() استفاده کنیم.

`transform.interceptor.ts`
```typescript
import { Injectable, NestInterceptor, ExecutionContext, CallHandler } from '@nestjs/common';
import { Observable } from 'rxjs';
import { map } from 'rxjs/operators';

export interface Response<T> {
  data: T;
}

@Injectable()
export class TransformInterceptor<T> implements NestInterceptor<T, Response<T>> {
  intercept(context: ExecutionContext, next: CallHandler): Observable<Response<T>> {
    return next.handle().pipe(map(data => ({ data })));
  }
}
```

پاسخی که به سمت کاربر ارسال میشه ، به شکل زیر خواهد بود:

```json
{
  "data": []
}
```

مثالی دیگر
به عنوان مثال فرض کنید می خواهیم اگر response ای که می خواهد به سمت کاربر ارسال شود ، `null` بود ، مقدارش با `''` عوض شود:

```typescript
import { Injectable, NestInterceptor, ExecutionContext, CallHandler } from '@nestjs/common';
import { Observable } from 'rxjs';
import { map } from 'rxjs/operators';

@Injectable()
export class ExcludeNullInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next
      .handle()
      .pipe(map(value => value === null ? '' : value ));
  }
}
```

## Exception Mapping-----------------

یکی دیگه از کار هایی که میشه با interceptor ها انجام داد ، گرفتن خطای احتمالی route handler ها و نمایش خطا یا هر چیزی که می خواهیم هست:

`errors.interceptor.ts`
```typescript
import {
  Injectable,
  NestInterceptor,
  ExecutionContext,
  BadGatewayException,
  CallHandler,
} from '@nestjs/common';
import { Observable, throwError } from 'rxjs';
import { catchError } from 'rxjs/operators';

@Injectable()
export class ErrorsInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next
      .handle()
      .pipe(
        catchError(err => throwError(() => new BadGatewayException())),
      );
  }
}
```

## Stream Overriding-----------------

به دلایل بسیاری ممکنه بخوایم بطور کامل از route handler جلوگیری کنیم و value خودمون رو بفرستیم یکی از موارد اینه که مثلا می خواهیم برای بالا رفتن سرعت ، بجای ارسال response بیایم cache موجود رو بفرستیم:

`cache.interceptor.ts`
```typescript
import { Injectable, NestInterceptor, ExecutionContext, CallHandler } from '@nestjs/common';
import { Observable, of } from 'rxjs';

@Injectable()
export class CacheInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const isCached = true;
    if (isCached) {
      return of([]);
    }
    return next.handle();
  }
}
```

ما اینجا منطق اینکه cache وجود دارد یا نه رو پیاده نکردیم و به یه `isCached = true` بسنده کردیم.

عملگر `of()` از Rxjs باعث می شود یک جریان (stream) جدید ایجاد بشه.
## More Operators-----------------

دستکاری stream با استفاده از عملگر های Rxjs توانایی های زیادی به ما داده.
مثلا فرض کنید می خواهیم برای یک route handler یه timeout تنظیم کنیم یعنی در صورتی که route handler نتونست بعد از مدتی response بفرسته ، یه exception پرتاب شود:

`timeout.interceptor.ts`
```typescript
import { Injectable, NestInterceptor, ExecutionContext, CallHandler, RequestTimeoutException } from '@nestjs/common';
import { Observable, throwError, TimeoutError } from 'rxjs';
import { catchError, timeout } from 'rxjs/operators';

@Injectable()
export class TimeoutInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      timeout(5000),
      catchError(err => {
        if (err instanceof TimeoutError) {
          return throwError(() => new RequestTimeoutException());
        }
        return throwError(() => err);
      }),
    );
  };
};
```

```ts
@Get()
async findAll() {
await new Promise(resolve => setTimeout(resolve, 5000));
	return this. coffeesService.findAll();
}
```

