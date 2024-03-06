در زبان فارسی دکوراتور رو می تونیم **شکل دهنده** معنی کنیم ؛ پس بالای هر چیزی که دکوراتور می گذاریم یعنی می خوایم اون چیز رو سر و شکل بدیم ؛ این سر و شکل دادن در برنامه نویسی یعنی اضافه کردن قابلیت یا قابلیت هایی به چیزی (property ، متد ، کلاس و...) بدون دستکاری مستقیم آن چیز.

فریمورک nest ، حول قابلیتی تحت عنوان `decorators` ساخته شده
قابلیت decorators قابلیتی نام آشنا در اکثر زبان های برنامه نویسی هست.

در es2016 دکوراتور ها یه اصطلاح هستند که یه تابع رو بر می گردونن و می تونن شامل **هدف** ، **نام** و property ی توصیف کننده ای در قالب **آرگومان** باشند.

برای بکار گیری یک دکوراتور باید از پیشوند `@` استفاده کرد و دکوراتور رو بالای چیزی که می خواد شکل و سر و وضعی به خود بگیره قرار دهیم.

همانطور که بالا تر هم گفتیم از دکوراتور ها میشه بالا سر property و method و class استفاده کرد.

## Param Decorators----------------

نست مجموعه ای از Param Decorator های کاربردی ای رو برای router handler ها ارائه کرده که می تونن با هم کار کنند:

|   |   |
|---|---|
|`@Request(), @Req()`|`req`|
|`@Response(), @Res()`|`res`|
|`@Next()`|`next`|
|`@Session()`|`req.session`|
|`@Param(param?: string)`|`req.params` / `req.params[param]`|
|`@Body(param?: string)`|`req.body` / `req.body[param]`|
|`@Query(param?: string)`|`req.query` / `req.query[param]`|
|`@Headers(param?: string)`|`req.headers` / `req.headers[param]`|
|`@Ip()`|`req.ip`|
|`@HostParam()`|`req.hosts`|
علاوه بر اینها ، شما می توانید دکوراتور اختصاصی (custom decorator) خودتون رو بسازید.

یکی از کار هایی که در دنیای node.js انجام میشه متصل کردن داده های مورد نیاز به شی request هست:

```typescript
const user = req.user;
```

می تونیم برای شفاف تر شدن کدمون برای اینکار دکوراتوری به اسم `@USer()` بسازیم که `req.user` رو بر میگردونه

```typescript
@Get()
async findOne(@User() user: UserEntity) {
  console.log(user);
}
```

`user.decorator.ts`
```typescript
export const User = createParamDecorator(
  (data: unknown, ctx: ExecutionContext) => {
    const request = ctx.switchToHttp().getRequest();
    return request.user;
  },
);
```

## Passing Data----------------

وقتی آرگومانی رو به دکوراتور خود پاس می دهیم ، این یعنی رفتار دکوراتور ما بستگی به شرط یا شرط هایی داره.

این آرگومان داده شده به دکوراتور در factory function قابل دریافته

مثلا فرض کنید ، یه request به router handler ما ارسال شده که دارای req.user هست:

`req.user`
```json
{
  "id": 101,
  "firstName": "Alan",
  "lastName": "Turing",
  "email": "alan@email.com",
  "roles": ["admin"]
}
```

حالا در نظر بگیرید که router handler ما تنها نیاز به property ی firstName دارد

در این حالت میایم نام property ای که می خوایم رو به عنوان پارامتر ورودی به دکوراتور `@User()` می دیم

```typescript
@Get()
async findOne(@User('firstName') firstName: string) {
  console.log(`Hello ${firstName}`);
}
```

`user.decorator.ts`
```typescript
import { createParamDecorator, ExecutionContext } from '@nestjs/common';

export const User = createParamDecorator(
  (data: string, ctx: ExecutionContext) => {
    const request = ctx.switchToHttp().getRequest();
    const user = request.user;

    return data ? user?.[data] : user;
  },
);
```

>[!tip]
>متد `createParamDecorator<T>()` بصورت generic هست ، T مشخص کننده نوع data هست ، پس یعنی می تونیم به این صورت type آرگوان ورودی دکوراتور رو مشخص کنیم ، یه راه دیگه برای مشخص کردن نوع data بصورت `createParamDecorator((data: string, ctx) => ...)` هست ، اگر هیچ یک از این روش ها استفاده نشده بود data از نوع any در نظر گرفته می شود.

## Working With Pipes----------------

نست با custom params decorator ها درست مثل param های داخلی خودش یعنی  (`@Body()`, `@Param()` and `@Query()`) رفتار میکنه یعنی custom param decorator هایی که ما می نویسیم رو به چشم بچه های خودش بهشون نگاه میکنه (:

از این رو می تونیم برای custom param decorator ها pipe در نظر بگیریم:

```typescript
@Get()
async findOne(
  @User(new ValidationPipe({ validateCustomDecorators: true }))
  user: UserEntity,
) {
  console.log(user);
}
```

>[!tip]
 >نکته اینکه برای اینکار `validateCustomDecorators`باید حتما `true`در نظر گرفته بشه ، این به این خاطره که `ValidationPipe` بصورت پیش فرض custom param decorator ها رو اعتبارسنجی نمیکنه.
 
## Decorator Composition----------------

نست یه helper method ارائه می ده که با استفاده از اون میشه چندین دکوراتور رو با هم ترکیب کرد مثلا فرض کنید می خواهید همه دکوراتور های مربوط به احراز هویت رو در قالب یه دکوراتور در بیارید:

`auth.decorator.ts`
```typescript
import { applyDecorators } from '@nestjs/common';

export function Auth(...roles: Role[]) {
  return applyDecorators(
    SetMetadata('roles', roles),
    UseGuards(AuthGuard, RolesGuard),
    ApiBearerAuth(),
    ApiUnauthorizedResponse({ description: 'Unauthorized' }),
  );
}
```

حالا می تونیم از دکوراتور `@Auth()` براحتی استفاده کنیم:

```typescript
@Get('users')
@Auth('admin')
findAllUsers() {}
```

