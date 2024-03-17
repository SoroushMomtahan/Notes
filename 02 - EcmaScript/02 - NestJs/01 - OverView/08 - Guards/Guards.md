>[!tip]
>**کلمات کلیدی این بخش:**
>`@Injectable()`
>
>`CanActive`
>
>`canActive()`
>
>`ExecutionContext`
>
>`@UseGuards()`
>
>`useGlobalGuards()`
>
>`APP_GUARD`

---

کلاس هایی که اینترفیس `CanActive` رو پیاده سازی کردند و بالاشون از دکوراتور `@Injectable()` استفاده شده است.

![](Pasted%20image%2020240302201632.png)

این Guard های قصه ما که نقش محافظ یا نگهبان رو بازی می کنن ، تنها یه مسئولیت (Single Responsibility) دارند.

آنها با توجه به شرایطی مشخص (permissions ، roles ، ACLs) ، بررسی می کنن که درخواست رسیده توسط route handler مربوطه رسیدگی بشه یا نه ، با اینکار به نوعی از route handler های ما حفاظت میکنند.
به این کار هایی که Gurad ها برامون انجام می دهند **Authorization** (اجازه ، مجوز) گویند.

این Authorization پسر عمویی به اسم Authentication (احراز هویت) داره که معمولا با Authorization همکاری میکنه.

این مسئله authorization و authentication در برنامه های Express ای معمولا توسط middleware ها انجام می شد.

مدیریت کردن authentication توسط middleware ها (چسباندن token به request) کار خوبیه چون این کار مربوط به route خاصی نمی شود.

اما مشکلی که وجود داره اینه که middleware ها بی عقل هستند یعنی نمی دونن با صدا کردن next() قراره کدام controller و در ادامه router handler ای اجرا شود.

محافظان (Guards) با دسترسی به نمونه ای از کلاس `ExecutionContext` می تونن بفهمن که بعد از آنها چه اتفاقی قراره بیفته

این Guard ها بسیار شبیه به exception filter ها ، pipes ها و interceptor ها هستند و به شما اجازه می دهند تا منطقی که می خواهید رو در نقطه مناسب چرخه request/response قرار دهید.
این کار به تمیز شدن کد کمک می کند.

>[!tip]
>این Guard ها بعد از همه middleware ها و قبل از interceptor ها و pipe ها اجرا می شوند.

## Authorization Guard-----------------

همانطور که گفته شد Authorization بسیار مورد استفاده ی Guard هاست چون مشخص می کنند که یک route handler باید در اختیار کاربری که احراز هویت شده قرار گیرد یا نه 

پس authorization یک منطق هست که درون guard ها قرار میگیره.

کلاس `AuthGaurd` رو ایجاد می کنیم ، اینطور تصور شده که کاربر قبلا احراز هویت شده و اکنون می خواهیم بررسی کنیم که کاربر به route handler مربوطه دسترسی داشته باشد یا نه که این مورد میشه جز authorization که در واقع یک منطق است و ما این منطق رو در متد `validateRequest()` پیاده سازی کردیم و در AuthGurard فقط ازش استفاده کردیم:

`auth.guard.ts`
```typescript
import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common';
import { Observable } from 'rxjs';

@Injectable()
export class AuthGuard implements CanActivate {
  canActivate(
    context: ExecutionContext,
  ): boolean | Promise<boolean> | Observable<boolean> {
    const request = context.switchToHttp().getRequest();
    return validateRequest(request);
  }
}
```

همه Guard ها باید متد `canActive()` رو پیاده سازی کنند ، این متد یه boolean برمیگردونه که مشخص میکنه کاربر یا request زده شده به router handler مربوطه دسترسی داشته باشه (true) یا دسترسی نداشته باشه (false)

## Excecution context-----------------

متد `canActive()` تنها یه پارامتر آرگومان ورودی میگیره و اونم نمونه ای از کلاس `ExecutionContext` هست. کلاس `ExecutionContext` از `ArgumentsHost` ارث بری کرده ، توی قسمت exception filter درباره `ArgumentsHost` خوندیم.

حالا کلاس `ExecutionContext` اومده در کنار متد هایی که `ArgumentHost` داشته یه سری helper method اضافه کرده که جزئیات بیشتری درباره پروسه در حال اجرا که شامل request , ... هست به ما میده ، این `ExecutionContext` کمک میکنه تا بتونیم guard های عمومی تری بنویسیم که روی مجموعه ای از controller ها کار میکنه 

## Role-based authentication-----------------

در این قسمت می خواهیم guard ای رو بسازیم که بر اساس نقش به کاربران مجوز دسترسی میده

فعلا در ابتدای کار تنها یه guard ای ساختیم که به همه کاربران اجازه دسترسی میده و در ادامه کاملش می کنیم:

`roles.guard.ts`
```typescript
import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common';
import { Observable } from 'rxjs';

@Injectable()
export class RolesGuard implements CanActivate {
  canActivate(
    context: ExecutionContext,
  ): boolean | Promise<boolean> | Observable<boolean> {
    return true;
  }
}
```

## Binding Guards-----------------

همانند exception filter ها و pipe ها یک guard میتونه **controller-scoped**, method-scoped, or global-scoped باشه.

در پایین با استفاده از دکوراتور `@UseGuards()` یه RolesGaurd رو به CatsController متصل کردیم:

```typescript
@Controller('cats')
@UseGuards(RolesGuard)
export class CatsController {}
```

در ضمن به این دکوراتور می تونیم مجموعه ای از guard ها که با کاما از هم جدا شدند ، رو بدیم.

در مثال بالا وظیفه نمونه سازی از کلاس `RoleGaurd` رو برعهده فریموریک گذاشتیم ، هر چند که می تونیم این کار رو خودمون هم انجام بدیم:

```typescript
@Controller('cats')
@UseGuards(new RolesGuard())
export class CatsController {}
```

اگه بخوایم `RoleGaurd` تنها روی یه متد (router handler) خاص از controller مون اعمال بشه هم براحتی می تونیم این کار رو انجام بدیم

همچنین می تونیم از یه guard بصورت global استفاده کنیم (با استفاده از متد `useGlobalGuards()`) تا حفاظ مربوطه روی همه route ها اعمال شود:

```typescript
const app = await NestFactory.create(AppModule);
app.useGlobalGuards(new RolesGuard());
```

>[!warning]
 >توی برنامه های هیبریدی نمی تونیم از متد `useGlobalGuards()` استفاده کنیم.
 
 این Global Gaurd هایی که بیرون از module تعریف می شوند (با استفاده از `useGlobalGaurds()`) نمی تونن وابستگی بهشون تزریق بشه و باید حتما نمونه بهشون پاس داده بشه

برای حل این مشکل می تونیم global gaurd ها رو در یک ماژول تعریف کنیم ، در این حالت فارغ از اینکه این guard در کدام ماژول تعریف شده باشد guard مربوطه Global است و روی همه ماژول ها اعمال می شود.

`app.module.ts`
```typescript
import { Module } from '@nestjs/common';
import { APP_GUARD } from '@nestjs/core';

@Module({
  providers: [
    {
      provide: APP_GUARD,
      useClass: RolesGuard,
    },
  ],
})
export class AppModule {}
```

## Setting roles per handler-----------------

تعیین نقش برای هر کنترل کننده

چطور می تونیم به هر مسیر یه نقش بدیم؟

اینجاست که ابرداده های سفارشی (**Custom metadata**) وارد بازی می شوند.

فریموریک nest این امکان رو فراهم کرده تا متادیتا های سفارشی رو به route handler های خود متصل کنیم.

یکی از راه های ایجاد custom metadata با استفاده از  `Reflector` و روش دیگه با استفاده از دکوراتور داخلی `@SetMetaData()` هست.

`roles.decorator.ts`
```ts
import { Reflector } from '@nestjs/core';

export const Roles = Reflector.createDecorator<string[]>();
```

`cats.controller.ts`
```typescript
@Post()
@Roles(['admin'])
async create(@Body() createCatDto: CreateCatDto) {
  this.catsService.create(createCatDto);
}
```


## Putting it all together-----------------

خب الان یه route handler ای داریم که یه metadata رو بهش وصل کردیم و تو این metadata ، نقش هایی که مورد نیاز route handler فعلی هست رو بهش دادیم و الان می خوایم ورود هر user به این route handler رو مشروط به نتیجه مقایسه نقش های این route handler با نقش هایی که کاربر دارد کنیم.

برای اینکار دوباره توسط `Reflector` نقش های مورد نیاز route handler مربوطه رو میگیریم :

`roles.guard.ts`
```typescript
import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common';
import { Reflector } from '@nestjs/core';
import { Roles } from './roles.decorator';

@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    const roles = this.reflector.get(Roles, context.getHandler());
    if (!roles) {
      return true;
    }
    const request = context.switchToHttp().getRequest();
    const user = request.user;
    return matchRoles(roles, user.roles);
  }
}
```

در پارامتر اول متد `get` ،  اون متادیتایی که می خواد بگیره رو مشخص کرده و در پارامتر دومش اینکه از کدام handler این متادیتا گرفته بشه ، مشخص شده.

متد `matchRoles` هم منطق ما برای مقایسه نقش های کاربر درخواست زده با نقش های مورد نیاز handler مربوطه هست.

این request.user هم قبلا در بخش احراز هویت به request متصل شده.

وقتی کاربر با امتیازات ناکافی درخواستی رو به یک endpoint می زند ، مسلما gaurd یک false رو return میکنه و nest بصورت اتوماتیک respone زیر رو میفرسته:

```typescript
{
  "statusCode": 403,
  "message": "Forbidden resource",
  "error": "Forbidden"
}
```

