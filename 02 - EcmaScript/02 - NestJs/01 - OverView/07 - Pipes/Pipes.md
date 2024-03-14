بطور کلی Pipe ها کلاس هایی هستند که اینترفیس `PipeTransform` رو پیاده سازی کردند و بالاشون از دکوراتور `@Injectable` استفاده شده است.

![](Pasted%20image%2020240302175353.png)

این Pipe ها دو جا کاربرد دارند:

- برای تبدیل داده (**`Transformation`**) : تبدیل داده ی ورودی به نوع دلخواه (به عنوان مثال تبدیل number به String)
- اعتبارسنجی (**`Validation`**) : ارزیابی داده ورودی به این صورت که اگر داده valid بود داده عبور داده شود و در غیر اینصورت throw شود یه exception

این pipe ها روی پارامتر های ورودی controller ها کار می کنند.

سیستم Nest درست قبل از اینکه controller method بخواد invoke بشه میاد Pipe ها رو صدا میزنه و Pipe پارامتر (های) ورودی Controller رو میگیره و روی آنها عملیات انجام میده

پس از انجام هرگونه عملیات تبدیل و یا اعتبارسنجی آرگومان های مورد نیاز به controller داده می شود.

سیستم Nest چندین Pipe داخلی دارد ؛ ضمن اینکه می تونیم custom pipe های خودمون رو هم بسازیم.

>[!tip]
 >اگر pipe ها exception تولید کنند ، توسط سیستم مدیریت خطای نست (Global Exception filter) این خطای پرتاب شده از Pipe ها مدیریت می شود.
 
## Built-in Pipes-------------

این nest دارای 9 تا pipe بصورت داخلی می باشد :

- `ValidationPipe`
- `ParseIntPipe`
- `ParseFloatPipe`
- `ParseBoolPipe`
- `ParseArrayPipe`
- `ParseUUIDPipe`
- `ParseEnumPipe`
- `DefaultValuePipe`
- `ParseFilePipe`

اکنون می خواهیم مثالی از `ParseIntPipe` بزنیم ، لازم به ذکره که pipe هایی که با Parse شروع می شوند همگی پیاده سازی یکسانی دارند.
## Binding Pipes--------------

روش پایین رو اتصال pipe به متد گویند:

```typescript
@Get(':id')
async findOne(@Param('id', ParseIntPipe) id: number) {
  return this.catsService.findOne(id);
}
```

حال دو حالت پیش میاد : 

- یا number و یا string ای که تنها شامل عدد هست به عنوان پارامتر ورودی دریافت میشه که این باعث میشه درخواست از لوله به سلامت عبور کنه و controller مربوطه invoke بشه 
- حالت دوم اینکه که به هر دلیلی درخواست نتونه از خط لوله به سلامت عبور کنه و وسط راه به مشکل بخوره ، که در این صورت pipe یک exception رو throw می کنه

مثلا فرض کنید ، عبارت بالا به عنوان id داده میشه :

```bash
GET localhost:3000/abc
```

در این صورت خطای زیر به عنوان response فرستاده می شود:

```json
{
  "statusCode": 400,
  "message": "Validation failed (numeric string is expected)",
  "error": "Bad Request"
}
```

در مثال بالا خود کلاس `ParseIntPipe` رو به پارامتر دوم دکوراتور پاس دادیم تا وظیفه نمونه سازی و مدیریت طول عمر نمونه رو بر عهده nest بذاریم
البته که میشه یک نمونه هم پاس داد ، اما باید به این نکته توجه کرد که زمانی بهتر است نمونه پاس بدیم که می خواهیم رفتار pipe مربوطه رو با دادن option هایی تغییر دهیم :

```typescript
@Get(':id')
async findOne(
  @Param('id', new ParseIntPipe({ errorHttpStatusCode: HttpStatus.NOT_ACCEPTABLE }))
  id: number,
) {
  return this.catsService.findOne(id);
}
```

کلا pipe هایی که با parse شروع میشن برای queryString ها و parameter ها و body ها کاربرد دارند.

برای مثال برای queryString هم بصورت زیر عمل می کنیم :

```typescript
@Get()
async findOne(@Query('id', ParseIntPipe) id: number) {
  return this.catsService.findOne(id);
}
```

یا به عنوان مثال در مثال پایین چک کردیم که پارامتر ورودی `uuid` هست یا نه :

```typescript
@Get(':uuid')
async findOne(@Param('uuid', new ParseUUIDPipe()) uuid: string) {
  return this.catsService.findOne(uuid);
}
```

>[!tip]
>وقتی از uuid استفاده می کنیم بصورت خودکار uuid به ورژن 3 , 4 و یا 5 تجزیه می شود ؛ برای اینکه بخواهیم مشخص کنیم uuid به ورزن خاصی parse شود شماره ورژن رو هنگام ساخت نمونه ، به عنوان option پاس می دهیم.

باید به این نکته توجه کرد که اتصال (Binding) لوله های اعتبارسنجی (Validation Pipe) کمی متفاوت از اتصال لوله های خانواده Parse* هستند.

## Custom Pipes--------------

می تونیم pipe های اختصاصی خودمون رو توسعه بدیم ، برای اینکار کلاس مربوطه باید اینترفیس `PipeTransform` رو پیاده سازی کند و از دکوراتور `@Injectable()` بالای کلاس نوشته شده استفاده کنیم تا بدین ترتیب مشخص کنیم که این کلاس یک کلاس تزریقی هست.

در مثال پایین یک لوله validation ساختیم که در آن یک مقدار رو گرفتیم و همونو return کردیم :

`validation.pipe.ts`
```typescript
import { PipeTransform, Injectable, ArgumentMetadata } from '@nestjs/common';

@Injectable()
export class ValidationPipe implements PipeTransform {
  transform(value: any, metadata: ArgumentMetadata) {
    return value;
  }
}
```

>[!tip]
>کلاس `PipeTransform<T, R>` یه کلاس Generic هست که T نشان دهنده Type value و R نشان دهنده type خروجی متد transform() هست.

همه pipe ها باید اینترفیس PipeTransform رو پیاده سازی کنن ، در این اینترفیس متد transform() وجود داره که باید پیاده سازی بشه 

این متد یعنی متد transform() دو پارامتر ورودی میگیره:
- پارمتر اول `value`
- پارامتر دوم `metadata`

پارامتر اول یعنی value همان آرگومان ورودی route handler مربوطه هست و پارامتر دوم یعنی metadata یک پیاده سازی از اینترفیس `ArgumentMetaData` هست.

حالا اینترفیس `ArgumentMetaData` شامل property های زیر هست :

```typescript
export interface ArgumentMetadata {
  type: 'body' | 'query' | 'param' | 'custom';
  metatype?: Type<unknown>;
  data?: string;
}
```

این property ها **آرگومان های پردازش فعلی** رو توصیف می کنند.

|   |   |
|---|---|
|`type`|Indicates whether the argument is a body `@Body()`, query `@Query()`, param `@Param()`, or a custom parameter (read more [here](https://docs.nestjs.com/custom-decorators)).|
|`metatype`|Provides the metatype of the argument, for example, `String`. Note: the value is `undefined` if you either omit a type declaration in the route handler method signature, or use vanilla JavaScript.|
|`data`|The string passed to the decorator, for example `@Body('string')`. It's `undefined` if you leave the decorator parenthesis empty.|
>[!warning]
>هنگام تبدیل ts به js ، اینترفیس ها ناپدید می شوند ، بنابراین اگر آرگومان ورودی یک controller رو از جنس interface تعیین کرده باشیم ، هنگام تبدیل به js متاتایپ این ورودی Controller ، از نوع object در نظر گرفته می شود. به همین خاطر پیشنهاد میشه که تایپ ورودی از جنس class باشد(مثل cat.dto.ts)

## Schema based validation-------------

حال می خواهیم یک اعتبارسنجی روی ورودی متد `create()` از `CatsController` انجام بدیم ، یعنی می خوایم بررسی کنیم ، چیزی که از body دریافت شده معتبر هست یا نه . اگر معتبر بود متد مربوطه invoke شود (invoke create method) و در غیر این صورت یک exeption پرتاب شود.

```typescript
@Post()
async create(@Body() createCatDto: CreateCatDto) {
  this.catsService.create(createCatDto);
}
```

`create-cat.dto.ts`
```typescript
export class CreateCatDto {
  name: string;
  age: number;
  breed: string;
}
```

می خواهیم مطمئن شویم که body دریافت شده از request معتبر است ؛ حال body از نوع CreateCatDto هست ، بنابراین باید هر یک از property های آن را از نظر معتبربودن یا نبودن بررسی کنیم.

ما می تونیم این کار رو در routeHandler مربوطه یعنی در بدنه متد create انجام دهیم ، اما انجام این کار اصل تک مسئولیتی (single responsibility principle) رو نقض میکنه.

راهکار دیگه ایجاد یک کلاس اعتبارسنجی (validator class) و واگذاری وظیفه اعتبارسنجی به اون کلاسه ؛ که این روش هم مشکلاتی داره از جمله اینکه باید همیشه یادمون باشه که نمونه ای کلاس اعتبارسنجی رو در ابتدا خط هر متدی صدا بزنیم.

چطوره از یک middleware عمومی ها استفاده کنیم ؟ middleware ها به دلیل اینکه از controller ای که می خواد بعدش اجرا بشه و همچنین پارامتر های ورودی این controller اطلاعی ندارن ، راهکار خوبی نیستند.

بهترین راه حل استفاده از validation Pipe هست.
## Object schema validation--------------

پس بستری که قراره اعتبارسنجی رو درش انجام بدیم (Validation Pipe) مشخص شد. 

حالا بیایم بررسی کنیم که از چه منطقی برای اعتبارسنجی استفاده کنیم ؛ یکی از منطق ها منطق **Schema-Based** هست. 

از کتابخانه [Zod]([Zod | Documentation](https://zod.dev/)) برای این کار استفاده می کنیم:

```bash
npm install --save zod
```

```typescript
import { PipeTransform, ArgumentMetadata, BadRequestException } from '@nestjs/common';
import { ZodSchema  } from 'zod';

export class ZodValidationPipe implements PipeTransform {
  constructor(private schema: ZodSchema) {}

  transform(value: unknown, metadata: ArgumentMetadata) {
    try {
      const parsedValue = this.schema.parse(value);
      return parsedValue;
    } catch (error) {
      throw new BadRequestException('Validation failed');
    }
  }
}
```

یک کلاس درست کردیم و مشخص کردیم که این کلاس به یک schema از جنس ZodSchema نیازمند هست (یه وابستگی تعریف کردیم که می بایست در constructor تزریق شود)
## Binding validation pipes---------------

حالا میایم ZodValidationPipe رو به وسیله دکوراتور `@UserPipe()` به routeHandler مربوطه متصل می کنیم.

`cats.controller.ts`
```typescript
@Post()
@UsePipes(new ZodValidationPipe(createCatSchema))
async create(@Body() createCatDto: CreateCatDto) {
  this.catsService.create(createCatDto);
}
```

```typescript
import { z } from 'zod';

export const createCatSchema = z
  .object({
    name: z.string(),
    age: z.number(),
    breed: z.string(),
  })
  .required();

export type CreateCatDto = z.infer<typeof createCatSchema>;
```

چون از Zod استفاده کردیم `CreateCatDto`  رو هم همینجا بسازیم ، بجای اینکه بخوایم به شکل پایین کلاسی جدا برای آن ایجاد کنیم:

```ts
export class CreateCatDto {
  name: string;
  age: number;
  breed: string;
}
```

>[!warning]
 >کتابخونه zod برای کار کردن نیاز به فعال کردن فلگ `strictNullChecks`در tsconfig.json دارد.
 
## Class validator---------------

>[!warning]
>این تکنیک یعنی class-validator فقط در ts کار می کند.

```bash
npm i --save class-validator class-transformer
```

در این روش بعد از نصب پکیج های مربوطه ، تنها کافیست چندین دکوراتور به کلاس `CreateCatDto` اضافه کنیم ، یعنی نیازی به ایجاد کلاسی جدا نداریم:

`create-cat.dto.ts`
```typescript
import { IsString, IsInt } from 'class-validator';

export class CreateCatDto {
  @IsString()
  name: string;

  @IsInt()
  age: number;

  @IsString()
  breed: string;
}
```

همانطور که در بالا مشخصه ، منطق اعتبارسنجی رو به کمک گذاشتن دکوراتور هایی بالای هر property از dto-class مون پیاده سازی کردیم

حالا باید بستری مناسب برای فرآیند اعتبارسنجی بوجود بیاریم ، این بستر رو می تونیم با کمک pipe ها و با ایجاد یه pipe سفارشی انجام بدیم:

`validation.pipe.ts`
```typescript
import { PipeTransform, Injectable, ArgumentMetadata, BadRequestException } from '@nestjs/common';
import { validate } from 'class-validator';
import { plainToInstance } from 'class-transformer';

@Injectable()
export class ValidationPipe implements PipeTransform<any> {
  async transform(value: any, { metatype }: ArgumentMetadata) {
    if (!metatype || !this.toValidate(metatype)) {
      return value;
    }
    const object = plainToInstance(metatype, value);
    const errors = await validate(object);
    if (errors.length > 0) {
      throw new BadRequestException('Validation failed');
    }
    return value;
  }

  private toValidate(metatype: Function): boolean {
    const types: Function[] = [String, Boolean, Number, Array, Object];
    return !types.includes(metatype);
  }
}
```

یادتونه که پکیج `class-transformer` رو ابتدا نصب کردیم ، این کلاس توسط سازنده خود `class-validator` توسعه داده شده و به همین منظور هماهنگی خوبی با پکیج class-validator داره 

حالا دلیل ستفاده از این پکیج اینه که هنگامی که مقداری از request به route handler مربوطه ارسال میشه ، نوع این value مشخص نیست پس باید ابتدا یه برچسبی به این داده ارسالی بزنیم تا مشخص شود نوعش چیه تا بشه اون نوعی که میگیره رو با دکوراتور هایی که رو سر property های dto-class مربوطه می گذاریم مقایسه کرد.

پس ، از متد `palinToInstance` از پکیج `class-transformer` استفاده کردیم ، این متد در پارامتر اولش نوعی که value باید داشته باشه رو میگیره و در پارامتر دوم خود value رو میگیره و به این صورت Value رو بر اساس پارامتر اول برجسب گذاری میکنه و نتیجه رو یک همان value اما با برچسب گذاری شده هست رو Return میکنه ، در گام بعد می تونیم این value رو برای ارزیابی معتبر بودن یا نبودن به متد `validate` بدیم.

>[!tip]
>البته خود `ValidationPipe` داخلی nest به خوبی از پس همه چی بر میاد و نیاز به نوشتن موارد بالا نبود.

----

در پایین هم pipe ای که نوشتیم رو متصل کردیم ، لازم به ذکر هست که این اتصال میتونه` global-scoped` یا `controller-scoped` یا `method-scoped` و یا مانند مثال پایین `parameter-scoped` باشه:

`cats.controller.ts`
```typescript
@Post()
async create(
  @Body(new ValidationPipe()) createCatDto: CreateCatDto,
) {
  this.catsService.create(createCatDto);
}
```

درباره دو بخش بالا در بخش مربوطه به طور مفصل توضیح داده خواهد شد....

## Global scoped pipes----------------

`main.ts`
```typescript
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.useGlobalPipes(new ValidationPipe());
  await app.listen(3000);
}
bootstrap();
```

>[!warning]
>از متد `useGlobalPipes()` در برنامه های هیبریدی `hybrid apps` که gateway , microservice ارائه میدهند ، نمی توان استفاده کرد. 

این global pipe ها در سراسر برنامه (برای کنترل ها و هر route handler ای) استفاده می شوند.

روی global pipe هایی که این شکلی تعریف می شوند امکان تزریق pipe مورد نظر وجود ندارد و باید دستی مانند بالا نمونه ساخته شود ، می تونیم global scope ها رو داخل ماژول ها استفاده کنیم تا نخواهیم از آنها شی بسازیم:

`app.module.ts`
```typescript
import { Module } from '@nestjs/common';
import { APP_PIPE } from '@nestjs/core';

@Module({
  providers: [
    {
      provide: APP_PIPE,
      useClass: ValidationPipe,
    },
  ],
})
export class AppModule {}
```

با اینکار فارغ از اینکه این pipe در کدام ماژول تعریف شده ، در سراسر پروژه مورد استفاده قرار میگیرد و روی هر کنترل و route handler ای اعمال می شود.

## The built-in validationPipe-------------

در بخش تکنیک و قسمت validation در این باره صحبت میشه.
## Transformation Use case----------------

در این قسمت گفته شده که می توان عملکرد داخلی transformation pipe ها رو شخصی سازی کرد.

بطور کلی از transformation pipe ها استفاده می کنیم چون شاید بخواهیم داده ارسالی از سمت client رو قبل رسیدن به دست route handler مربوطه تغییر بدیم مثلا به داده های ارسال نشده مقدار پیش فرض بدیم و یا داده string ای ارسال شده رو به integer تبدیل کنیم:

`parse-int.pipe.ts`
```typescript
import { PipeTransform, Injectable, ArgumentMetadata, BadRequestException } from '@nestjs/common';

@Injectable()
export class ParseIntPipe implements PipeTransform<string, number> {
  transform(value: string, metadata: ArgumentMetadata): number {
    const val = parseInt(value, 10);
    if (isNaN(val)) {
      throw new BadRequestException('Validation failed');
    }
    return val;
  }
}
```

```typescript
@Get(':id')
async findOne(@Param('id', new ParseIntPipe()) id) {
  return this.catsService.findOne(id);
}
```

یکی از کاربرد های دیگه ی transformation pipe ها تبدیل id ارسالی از client به user مربوطه هست.

```typescript
@Get(':id')
findOne(@Param('id', UserByIdPipe) userEntity: UserEntity) {
  return userEntity;
}
```

مخصوصا مورد بالا به تمیزی کد ما خیلی کمک میکنه.

## Providing defaults----------------

این parse pipe ها ، انتظار دارند که پارامتر value یه مقداری داشته باشه ، اگه این پارامتر `null` و یا `undefined` باشه parse pipe ها یه exception پرتاب میکنن

حالا برای اینکه به endpoint ها بتونیم این امکان رو بدیم که داده از دست رفته رو handle کنند یه pipe ای به اسم defaultValuePipe رو قبل از parsePipe ها قرار می دیم ، با اینکار اگر داده ارسالی از سمت کاربر null و یا undefined بود ، داده ارسالی ، این مقدار پیش فرض رو بجای null یا undefined به خود میگیرد:

```typescript
@Get()
async findAll(
  @Query('activeOnly', new DefaultValuePipe(false), ParseBoolPipe) activeOnly: boolean,
  @Query('page', new DefaultValuePipe(0), ParseIntPipe) page: number,
) {
  return this.catsService.findAll({ activeOnly, page });
}
```

