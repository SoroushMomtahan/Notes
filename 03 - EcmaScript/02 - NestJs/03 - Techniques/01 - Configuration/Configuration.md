برنامه ها بسته به محیطی که در اون قرار میگیرند ،  تنظیمات پیکر بندی متفاوتی خواهند داشت ، مثلا تنظیمات دیتابیس در محیط development با محیط production معمولا فرق می کنه. 

در nodejs با استفاده از `proccess.env` می تونیم به متغیر های محیطی دسترسی پیدا کنیم.

بهترین راه حل برای استفاده از متغیر های محیطی استفاده از یک `ConfigModule` هست ، این ماژول یک سرویس به نام `ConfigService` داره که می تونیم از این سرویس برای بدست آوردن متغیر محیطی مورد نظر استفاده کنیم.

فریمورک nest برای راحتی کار یک module با همین نام توسعه داده که باید اونو نصب کنیم:

```bash
$ npm i --save @nestjs/config
```

پکیج `@nestjs/config` بصورت داخلی از پکیج `dotenv` استفاده می کند.

خب حالا باید از این ماژول داینامیک استفاده کنیم ، بهترین جا برای استفاده از این ماژول ، ماژول `AppModule` هست ، از این ماژول میشه در ماژول های دیگه هم استفاده کرد که بعدا درباره آن توضیح داده خواهد شد:

`app.module.ts`
```typescript
import { Module } from '@nestjs/common';
import { ConfigModule } from '@nestjs/config';

@Module({
  imports: [ConfigModule.forRoot()],
})
export class AppModule {}
```

با ثبت ماژول `ConfigModule` ، این ماژول یه فایل `.env` رو از محل پیش فرض پروژه (root directory) بلند 
میکنه :) و محتویاتش رو می خونه و تجزیه میکنه ، 

## Custom Env File Path---------------

حالا اگه بخوایم این محل پیش فرض رو عوض کنیم از property ی `envFilePath` استفاده می کنیم:

```typescript
ConfigModule.forRoot({
  envFilePath: '.development.env',
});
```

می تونیم چندین مسیر هم به عنوان مسیر فایل env بدیم:

```typescript
ConfigModule.forRoot({
  envFilePath: ['.env.development.local', '.env.development'],
});
```

در این حالت ، اگه یک variable بین دو فایل یکسان بود ، ***اولویت با محتویات فایل اول است***.

## Disable Env File--------------

اگر می خواهید فایل env در نظر گرفته نشه و بجاش یه همچین چیزی `export DATABASE_USER=test` در نظر گرفته بشه property ی `ignoreEnvFile` رو true می دیم:

```typescript
ConfigModule.forRoot({
  ignoreEnvFile: true,
});
```

## Use Config Module--------------

اگر می خواید از `ConfigModule` در ماژول های دیگر استفاده کنید ، باید `ConfigModule` در ماژول مورد نظر import شود ، اما برای راحتی کار می توان property ی `isGlobal` رو در `ConfigModule` تنظیم کرد تا این ماژول global شود و نیاز به import کردن در ماژول های دیگر رو نداشته باشه:

```typescript
ConfigModule.forRoot({
  isGlobal: true,
});
```

## Custom Configuration Files-------------

بسته به نیاز پروژه میشه فایل های پیکربندی سفارشی ایجاد کرد که در واقع یه تابع هست که یک شی js ای برمیگردونه ، بدین ترتیب میشه property های تودر تو ایجاد کرد و تعدادی از متغیر های محیطی که مربوط به ویژگی ای خاص هستند رو در یک فایل قرار داد:

`config/configuration.ts`
```typescript
export default () => ({
  port: parseInt(process.env.PORT, 10) || 3000,
  database: {
    host: process.env.DATABASE_HOST,
    port: parseInt(process.env.DATABASE_PORT, 10) || 5432
  }
});
```

حال با استفاده از property ی `load` میشه این فایل رو به `ConfigModule` شناسوند:

```typescript
import configuration from './config/configuration';

@Module({
  imports: [
    ConfigModule.forRoot({
      load: [configuration],
    }),
  ],
})
export class AppModule {}
```

#تکمیل_شود 

برای custom configuration file میشه از فایل YAML هم استفاده کرد ؛ برای اطلاعات بیشتر درباره استفاده از Yamlبه داکیومنت nest مراجعه شود.

## Use The `ConfigService`-----------------

خب حالا که این module رو import کردیم و اونو global در نظر گرفتیم ، وقتش رسیده تا در جایی که می خوایم از `ConfigService` استفاده کنیم یعنی به نوعی اعلام نیازمندی به این سرویس بکنیم:

```typescript
constructor(private configService: ConfigService) {}
```

حالا می تونیم هر جایی که می خوایم از متغیر های محیطی استفاده کنیم:

`.env`
```json
ATABASE_USER=test
DATABASE_PASSWORD=test
```

`config/configuration.ts`
```ts
export default () => ({
  port: parseInt(process.env.PORT, 10) || 3000,
  database: {
    host: process.env.DATABASE_HOST,
    port: parseInt(process.env.DATABASE_PORT, 10) || 5432
  }
});
```

`my-module.service.ts`
```typescript
// get an environment variable
const dbUser = this.configService.get<string>('DATABASE_USER');

// get a custom configuration value
const dbHost = this.configService.get<string>('database.host');
```

با استفاده از متد `configService.get()` می تونیم به مقادیر متغیر های محیطی دسترسی پیدا کنیم ، حتی اگر از یک فایل پیکربندی سفارشی هم استفاده کرده باشیم ، براحتی می تونیم بصورت تودر تو هم به مقادیر دسترسی داشته باشیم (مانند قسمت دوم کد بالا)

حتی میشه یه interface هم درست کنیم و به شکل زیر عمل کنیم:

```typescript
interface DatabaseConfig {
  host: string;
  port: number;
}

const dbConfig = this.configService.get<DatabaseConfig>('database');

// you can now use `dbConfig.port` and `dbConfig.host`
const port = dbConfig.port;
```

در کد بالا type خروجی متد `get` رو با استفاده از `generic` ای که براش در نظر گرفتیم مشخص کردیم ، پس در واقع generic ای که متد get میگیره type خروجی متد get رو مشخص میکنه.

متد `get()` همچنین یه پارامتر دوم اختیاری هم داره که یه مقدار میگیره تا اگه کلید داده شده در متغیر های محیطی وجود نداشت ، آن مقدار به عنوان خروجی متد get در نظر گرفته بشه:

```typescript
// use "localhost" when "database.host" is not defined
const dbHost = this.configService.get<string>('database.host', 'localhost');
```

اضافه کردن یه generic به `ConfigService` :

```typescript
interface EnvironmentVariables {
  PORT: number;
  TIMEOUT: string;
}

// somewhere in the code
constructor(private configService: ConfigService<EnvironmentVariables>) {
  const port = this.configService.get('PORT');

  // TypeScript Error: this is invalid as the URL property is not defined in EnvironmentVariables
  const url = this.configService.get('URL');
}
```

اینکار دو مزیت داره:
- مواظب هست که پارامتر اول داده شده به متد get جز property های موجود باشد
- مواظب هست که پارامتر دوم متد get که یه مقدار اختیاری هست از نوع property باشد

الان چون property ای به اسم `url` در `EnvironmentVariables` وجود ندارد ، از ما ایراد گرفته می شود.

`{ infer: true }`

زمانی که object های تو در تو داریم و می خواهیم از generic برای `ConfigService` استفاده کنیم ، باید این property به عنوان option به متد get داده شود تا مجوز استفاده از `.` رو برای object های تو در تو پیدا کنیم:

```typescript
interface EnvironmentVariables {
  database:{
	  PORT: number;
	  TIMEOUT: string;
  }
}

// somewhere in the code
constructor(private configService: ConfigService<EnvironmentVariables>) {

  //error
  const port = this.configService.get('database.PORT');

  // pass
  const port = this.configService.get('database.PORT', { infer: true });
}
```

#تکمیل_شود

یه چند تا نکته دیگه وجود داره که الان متوجه شون نشدم و بعدا تکمیل میشه...

## Configuration namespace-----------------

می تونیم به متد anonymous که برای config ساختیم  یه اسم بدیم ، به این صورت که این متد رو میپیچیمش دور یه یه متد دیگه به اسم `registerAs()` که آرگومان اولش یه اسم میگیره و آرگومان دومش متدمون رو به عنوان callback میگیره:

`config/database.config.ts`
```typescript
export default registerAs('database', () => ({
  host: process.env.DATABASE_HOST,
  port: process.env.DATABASE_PORT || 5432
}));
```

حالا مثل روال گذشته فایل config رو در `ConfigModule` لود می کنیم:

```typescript
import databaseConfig from './config/database.config';

@Module({
  imports: [
    ConfigModule.forRoot({
      load: [databaseConfig],
    }),
  ],
})
export class AppModule {}
```

در گام بعد ، پس از اعلام نیازمندی به `ConfigService` می تونیم به شکل زیر متغیر های محیطی رو بدست بیاریم:

```typescript
const dbHost = this.configService.get<string>('database.host');
```

**بهترین روش برای استفاده از فایل config**

می تونیم همون توکن `database` رو که به متد registerAt دادیم رو تزریق کنیم:

```typescript
constructor(
  @Inject(databaseConfig.KEY)
  private dbConfig: ConfigType<typeof databaseConfig>,
) {}
```

مزیتی که این روش داره اینه که از type checking قدرتمندی برخوردار میشیم:

![](./Images/Pasted%20image%2020240314191551.png)

## Cache Environment Variable-----------------

میشه قابلیت cache رو برای `ConfigModule` فعال کرد و با اینکار متد get میتونه با سرعت بیشتری متغیر های محیطی رو بدست بیاره

```typescript
ConfigModule.forRoot({
  cache: true,
});
```

ممکنه بسته به نیاز پروژه ، فایل های config زیادی داشته باشیم که برای ماژول های مختلفی هستند و نخواهیم در ابتدای کار همه آنها `load` شوند و بجای آن درون هر ماژول config مربوطه رو با استفاده از متد `forFeature()` وارد می کنیم:

```typescript
import databaseConfig from './config/database.config';

@Module({
  imports: [ConfigModule.forFeature(databaseConfig)],
})
export class DatabaseModule {}
```

#تکمیل_شود 

یه هشدار وجود داره که باید تکمیل بشه....

## Schema Validation----------------

می تونیم متغیر های محیطی رو قبل شروع برنامه اعتبار سنجی کنیم تا در صورتی که متغیری وجود ندارد یا مقدار آن مناسب نیست ، برنامه کلا اجرا نشه و خطا پرتاب شود:

روش های مختلفی برای اعتبارسنجی وجود داره ، یکی از این روش ها استفاده از پکیج `joi` هست:

```bash
npm install --save joi
```

متد `forRoot()` در option های خود property ای تحت عنوان `ValidationSchema` داره که با استفاده از اون می تونیم validation ای که با joi انجام دادیم رو به این property بدیم :

`app.module.ts`
```typescript
@Module({
  imports: [
    ConfigModule.forRoot({
      validationSchema: Joi.object({
        NODE_ENV: Joi.string()
          .valid('development', 'production', 'test', 'provision')
          .default('development'),
        PORT: Joi.number().port().default(3000),
      }),
    }),
  ],
})
export class AppModule {}
```

دو متغیر محیطی به اسم `NODE_ENV` و `PORT` داریم که تحت عنوان property به برای validate کردن به joi معرفی می کنیم.

خود پکیج joi یه سری option داره که در قالب یکی از property های `foorRoot` به اسم `validationOption` می تونیم اونا رو تنظیم کنیم:

`app.module.ts`
```typescript
import * as Joi from 'joi';

@Module({
  imports: [
    ConfigModule.forRoot({
      validationSchema: Joi.object({
        NODE_ENV: Joi.string()
          .valid('development', 'production', 'test', 'provision')
          .default('development'),
        PORT: Joi.number().port().default(3000),
      }),
      validationOptions: {
        allowUnknown: false,
        abortEarly: true,
      },
    }),
  ],
})
export class AppModule {}
```

پروپرتی `allUnknown` وقتی false بشه دیگه اجازه دریافت env هایی که در property های joi نیستند رو نمی ده

پروپرتی `abortEarly` با دریافت اولین خطا validation متوقف میشه و خطا پرتاب میشه.

## Custom Validate Function---------------

یکی از روش های دیگه برای اعتبارسنجی متغیر های محیطی ، استفاده از `class-validator` هست:

`env.validation.ts`
```typescript
import { plainToInstance } from 'class-transformer';
import { IsEnum, IsNumber, validateSync } from 'class-validator';

enum Environment {
  Development = "development",
  Production = "production",
  Test = "test",
  Provision = "provision",
}

class EnvironmentVariables {
  @IsEnum(Environment)
  NODE_ENV: Environment;

  @IsNumber()
  @Min(0)
  @Max(65535)
  PORT: number;
}

export function validate(config: Record<string, unknown>) {
  const validatedConfig = plainToInstance(
    EnvironmentVariables,
    config,
    { enableImplicitConversion: true },
  );
  const errors = validateSync(validatedConfig, { skipMissingProperties: false });

  if (errors.length > 0) {
    throw new Error(errors.toString());
  }
  return validatedConfig;
}
```

حالا می تونیم از متد validate استفاده کنیم:

`app.module.ts`
```typescript
import { validate } from './env.validation';

@Module({
  imports: [
    ConfigModule.forRoot({
      validate,
    }),
  ],
})
export class AppModule {}
```

## Custom Getter Function------------------

می تونیم عملکرد جدیدی رو با کمک متد get از ConfigService بوجود بیاریم:

```typescript
@Injectable()
export class ApiConfigService {
  constructor(private configService: ConfigService) {}

  get isAuthEnabled(): boolean {
    return this.configService.get('AUTH_ENABLED') === 'true';
  }
}
```

حالا می تونیم از این getter method بسته به نیاز استفاده کنیم:

`app.service.ts`
```typescript
@Injectable()
export class AppService {
  constructor(private apiConfigService: ApiConfigService) {
    if (apiConfigService.isAuthEnabled) {
      // Authentication is enabled
    }
  }
}
```

## Environment variable hook-------------

#تکمیل_شود 

## Conditional Module Configuration---------------

ممکنه بخوایم اگر یه شرطی برقرار شد آنوقت یه ماژول import شود و اینو در نظر بگیرید که اون شرط رو در متغیر های محیطی مشخص کرده باشیم 

خوشبختانه پکیج @nest/config ماژولی تحت عنوان `ConditionalModule` رو برای این کار در اختیارمون قرار داده :

```json
USE_CAT_MODULE=false
```

```typescript
@Module({
  imports: [ConfigModule.forRoot(), ConditionalModule.registerWhen(CatModule, 'USE_CAT_MODULE')],
})
export class AppModule {}
```

حالا بنابر شرط ، `CatModule` در AppModule نمی تواند import شود ، چون متغیر محیطی `USE_CAT_MODULE` مقدار false دارد.

همچنین می تونیم ، شرط های پیچیده تری نیز ایجاد کنیم:

```typescript
@Module({
  imports: [ConfigModule.forRoot(), ConditionalModule.registerWhen(FooBarModule, (env: NodeJS.ProcessEnv) => !!env['foo'] && !!env['bar'])],
})
export class AppModule {}
```

#تکمیل_شود 

باید مطمئن شوید که `ConfigModule` قبل از `ConditionalModule` نمونه سازی می شود برای اینکار میشه از environment variable hook استفاده کرد...

## Expanded Variable------------------

پکیج `@nest/config` درون خود از پکیج `dotenv-expand` (https://github.com/motdotla/dotenv-expand) استفاده میکنه.

با استفاده از این پکیج میشه در فایل `.env` به شکل زیر عمل کرد:

```json
APP_URL=mywebsite.com
SUPPORT_EMAIL=support@${APP_URL}
```

همچنین برای فعال شدن این قابلیت باید پروپرتی `expandsVariable` رو در option متد forRoot فعال کنیم:

`app.module.ts`
```typescript
@Module({
  imports: [
    ConfigModule.forRoot({
      // ...
      expandVariables: true,
    }),
  ],
})
export class AppModule {}
```

## Use in the `main.ts` -------------

می تونیم از متغیر های محیطی در فایل `main.ts` استفاده کنیم ، به این صورت که ابتدا با استفاده از `app.get()` میاییم `ConfigService` رو میگیریم و بعد از متد get اش استفاده می کنیم:

```typescript
const configService = app.get(ConfigService);
```

```typescript
const port = configService.get('PORT');
```




