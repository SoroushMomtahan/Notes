در بخش overview و قسمت module درباره چیستی module ها صحبت کردیم ، در این قسمت می خواهیم نکات تکمیلی درباره module ها یعنی نحوه نوشتن dynamic module رو بررسی کنیم و پرونده module ها رو بطور کل ببندیم.

```ts
import { Module } from '@nestjs/common';
import { createConnection } from 'typeorm';

@Module ({
	providers: [
		provide: 'CONNECTION',
		useValue: createConnection ({
			type: 'postgres',
			host: 'localhost',
			port: 5432,
		}),
	],
});
export class DatabaseModule {}
```

این یک ماژول دیتابیسی هست ، اینکه دیتابیس در چه port ای هست و نام و نوع دیتابیس چیست رو به صورت ایستا دادیم. اما می خواهیم کاری کنیم که هر ماژولی که از ماژول دیتابیسی ما بخواهد استفاده کند ، بصورت داینامیک بتواند این مقادیر رو تنظیم کند ، اینجاست که dynamic module ها خود نمایی می کنند.

```ts
@Module({})
export class DatabaseModule {
	static register(options: ConnectionOptions): DynamicModule {
		return {
			module: DatabaseModule,
			providers: [
				{
					provide: 'CONNECTION',
					useValue: createConnection(options),
				},
			],
		}
	}
}
```

حالا می تونیم در هر جایی از این ماژول استفاده کنیم و option های مورد نیاز را بصورت داینامیک بدیم:

```ts
@Module ({
	imports: [
		DatabaseModule.register({
		type: 'postgres',
		host: 'localhost',
		password: 'password',
		port: 5432,
		}),
		CoffeesModule,
	],
	providers: [CoffeeRatingService],
})
export class CoffeeRatingModule {}
```

این کلیات داینامیک ماژول ها بود ، حالا در قسمت های بعدی با جزئیات بیشتر همه این مراحل توضیح داده خواهد شد.

## Introduction--------------

تا اینجا تنها `static module` ها رو دیدیم.

```typescript
import { Module } from '@nestjs/common';
import { UsersService } from './users.service';

@Module({
  providers: [UsersService],
  exports: [UsersService],
})
export class UsersModule {}
```

اگر یک ماژول رو به شکل بالا تعریف کنیم ، در واقع یک static module نوشته ایم ، پس `UsersModule` یک ماژول Static هست.

با تعریف یک ماژول بصورت static ، دیگه امکانی برای پیکربندی ماژول import شده در ماژول مصرف کننده فراهم نمیشه ؛ یعنی اگه `UsersModules` رو در ماژولی import کنیم ، امکان پیکربندی آن وجود نخواهد داشت.

ما می تونیم یه module رو dynamic تعریف کنیم تا مصرف کننده آن ماژول بتونه پیکربندی هایی رو روی ماژول مربوطه انجام بده و به نوعی ماژول رو سفارشی کنه. (منظورمان از فراهم شدن امکان شخصی سازی روی یک ماژول ، شخصی سازی روی provider های آن ماژول هست ؛ که این شخصس سازی میتونه شامل اضافه کردن پیکربندی هایی به provider مربوطه و یا تزریق یه provider دیگه به provider مربوطه باشه که در ادامه همه این موارد رو بررسی می کنیم )

## Dynamic Module use case------------------

با تعریف ماژول ها به شکل static ، ماژول های مصرف کننده نمی تونن کنترل و **نفوذی** روی provider های ماژول میزبان داشته باشند.

فرض کنید یک provider یه سری تنظیمات عمومی داره که قبلا نوشته شده ، حالا  این provider بسته به ماژولی که اونو مصرف میکنه ، ممکنه نیاز به یه سری تنظیمات اختصاصی برای ماژول مصرف کننده داشته باشه.

یه مثال خوب در این باره **Configuration Module** ها هستند ، این ماژول بسته به ماژولی که در آن قرار میگیرد ، می تواند پیکربندی های متفاوتی رو بواسطه Dynamic Module بودنش ، قبول کنه. مثلا پارامتر های مورد نیاز یک ماژول رو می توان به این ماژول داد تا آن پارامتر ها رو برای ماژول مربوطه لود کنه و آن ماژول بتونه در component های خود از آن پارامتر ها که configuration module ، آنها رو لود کرده ، استفاده کنه ، حالا همین configuration module وقتی در محیط دیگری قرار میگیره می تونه parameter های دیگری رو برای ماژول هدف لود کنه (درباره confguration module ، در قسمت Technique و بخش configuration می خوانیم).

## Config Module Example----------------

فرض کنید ماژولی به اسم `ConfigModule` داریم ، می خوایم کاری کنیم که این ماژول بسته به جایی که ازش استفاده میشه ، بتونه آدرس فایل های .env متفاوتی رو دریافت کنه. مثلا فرض کنید که چندین فایل .env داریم که می خواهیم که می خواهیم از هر کدام در شرایط خاص خودشون استفاده کنیم ، با این روش هر فایل .env تنها زمانی که بهش نیازه لود میشه.

ابتدا `ConfigModule` رو یک ماژول static در نظر میگیریم:

```typescript
import { Module } from '@nestjs/common';
import { AppController } from './app.controller';
import { AppService } from './app.service';
import { ConfigModule } from './config/config.module';

@Module({
  imports: [ConfigModule],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

حالا فرض کنید `ConfigModule` یک ماژول Dynamic هست (فعلا وارد جزئیات نحوه داینامیک کردن نشدیم):

```typescript
import { Module } from '@nestjs/common';
import { AppController } from './app.controller';
import { AppService } from './app.service';
import { ConfigModule } from './config/config.module';

@Module({
  imports: [ConfigModule.register({ folder: './config' })],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

به تفاوت ها که نگاه می کنیم ، متوجه می شویم که به `ConfigModule` متدی تحت عنوان `register()` اضافه شده که یه object گرفته که درون اون folder شامل فایل config مشخص شده.

از طرفی میشه پیش بینی کرد که متد `register()` باید یک module رو return کنه ، چون عملا property ی imports ، تنها آرایه ای از module ها رو میتونه دریافت کنه . به بیان دقیق تر متد `register()` یه `DynamicModule` رو return میکنه که در ادامه با جزئیات آن آشنا می شویم.

این `DynamicModule` چیزی جز یه ماژول معمولی نیست ، با این تفاوت که در run-time ساخته می شود.

بیاید یه مرور دوباره به static ماژول ها داشته باشیم:

```typescript
@Module({
  imports: [DogsModule],
  controllers: [CatsController],
  providers: [CatsService],
  exports: [CatsService]
})
```

یک dynamic module نیز باید دقیقا همین property های static module رو داشته باشه به علاوه ی یک property ی `module` که مشخص کننده نام module هست و باید با نام Module یکی باشد.

مثال زیر داینامیک ماژول `ConfigModule` رو نشون میده:

```typescript
import { DynamicModule, Module } from '@nestjs/common';
import { ConfigService } from './config.service';

@Module({})
export class ConfigModule {
  static register(): DynamicModule {
    return {
      module: ConfigModule,
      providers: [ConfigService],
      exports: [ConfigService],
    };
  }
}
```

الان dynamic module ما کار خاصی نسبت به static module نمیکنه ، در ادامه configuration هایی رو به عنوان پارامتر ورودی به متد `register()` می دهیم و...

## Module Configuration-----------------

بیایم یه بار دیگه به ماژول مصرف کننده ی `ConfigModule` نگاه کنیم:

```typescript
import { Module } from '@nestjs/common';
import { AppController } from './app.controller';
import { AppService } from './app.service';
import { ConfigModule } from './config/config.module';

@Module({
  imports: [ConfigModule.register({ folder: './config' })],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

حالا باید این موضوع رو بررسی کنیم که چطور از option ای که به عنوان ورودی به متد `register()` دادیم ، استفاده کنیم؟

باید این نکته رو در نظر بگیریم که ما در واقع یه module رو در ماژول مصرف کندده import می کنیم تا ماژول مصرف کننده بتونه به Service های ماژول import شده دسترسی پیدا کنه ، پس وقتی option ای هم در ماژول مصرف کننده تنظیم می کنیم ، این option برای Service های ماژول import شده هست.

پس یعنی باید یه روشی باشه که بتونیم به option ای که در ماژول مصرف کننده تزریق کردیم ، در Service مربوطه دسترسی پیدا کنیم.

ابتدا بیایم کد های `ConfigService` رو ببینیم (در کد پایین ، چون هنوز نحوه دسترسی به option در Service مربوطه رو نمی دونیم ، option رو بصورت hard-code قرار دادیم):

```typescript
import { Injectable } from '@nestjs/common';
import * as dotenv from 'dotenv';
import * as fs from 'fs';
import * as path from 'path';
import { EnvConfig } from './interfaces';

@Injectable()
export class ConfigService {
  private readonly envConfig: EnvConfig;

  constructor() {
    const options = { folder: './config' };

    const filePath = `${process.env.NODE_ENV || 'development'}.env`;
    const envFile = path.resolve(__dirname, '../../', options.folder, filePath);
    this.envConfig = dotenv.parse(fs.readFileSync(envFile));
  }

  get(key: string): string {
    return this.envConfig[key];
  }
}
```

ما از dependency injection برای دسترسی به `option` در Service مربوطه استفاده می کنیم.
در واقع `ConfigModule` یه provider ای به اسم `ConfigService` داره که به `option` نیازمنده که این option در run-time قراره تامین بشه ، پس این option اول باید به عنوان provider به`ConfigModule` شناسونده بشه و در گام بعد `ConfigService` به این option اعلام نیازمندی کنه:

>[!tip]
>قبلا هم در قسمت Fundamentals و بخش Custom Providers گفته بودیم که provider ها فقط service نیستند و می تونن چیز های دیگری نیز باشند.

- اول ، تعریف `option` به عنوان provider:

```typescript
import { DynamicModule, Module } from '@nestjs/common';
import { ConfigService } from './config.service';

@Module({})
export class ConfigModule {
  static register(options: Record<string, any>): DynamicModule {
    return {
      module: ConfigModule,
      providers: [
        {
          provide: 'CONFIG_OPTIONS',
          useValue: options,
        },
        ConfigService,
      ],
      exports: [ConfigService],
    };
  }
}
```

- دوم ، اعلام نیازمندی به `option` در `ConfigService`:

```typescript
import * as dotenv from 'dotenv';
import * as fs from 'fs';
import * as path from 'path';
import { Injectable, Inject } from '@nestjs/common';
import { EnvConfig } from './interfaces';

@Injectable()
export class ConfigService {
  private readonly envConfig: EnvConfig;

  constructor(@Inject('CONFIG_OPTIONS') private options: Record<string, any>) {
    const filePath = `${process.env.NODE_ENV || 'development'}.env`;
    const envFile = path.resolve(__dirname, '../../', options.folder, filePath);
    this.envConfig = dotenv.parse(fs.readFileSync(envFile));
  }

  get(key: string): string {
    return this.envConfig[key];
  }
}
``````typescript
import * as dotenv from 'dotenv';
import * as fs from 'fs';
import * as path from 'path';
import { Injectable, Inject } from '@nestjs/common';
import { EnvConfig } from './interfaces';

@Injectable()
export class ConfigService {
  private readonly envConfig: EnvConfig;

  constructor(@Inject('CONFIG_OPTIONS') private options: Record<string, any>) {
    const filePath = `${process.env.NODE_ENV || 'development'}.env`;
    const envFile = path.resolve(__dirname, '../../', options.folder, filePath);
    this.envConfig = dotenv.parse(fs.readFileSync(envFile));
  }

  get(key: string): string {
    return this.envConfig[key];
  }
}
```

ما `CONFIG_OPTIONS` رو برای راحتی داخل provider به عنوان توکن ، تعریف کردیم اما بهتر بود فایلی تحت عنوان `constant.ts` تعریف کنیم و درون این فایل به شکل زیر عمل کنیم:

```typescript
export const CONFIG_OPTIONS = 'CONFIG_OPTIONS';
```

##  Community guidelines-----------------

سه تا متد `forRoot()` و `forFeature()` و `register()` داریم که برای dynamic  کردن ماژول مورد استفاده قرار میگیرد.

متد `forRoot()` زمانی استفاده می شود که می خواهیم یک سری تنظیمات بر سر ماژول import شده انجام بدیم که برای همه برنامه این تنظیمات یکسانه.

متد `forFeature()` زمانی کاربرد داره که علاوه بر تنظیمات کلی که با استفاده از متد `forRoot()` اعمال کردیم ، می خواهیم یک سری تنظیمات خاص نیز اعمال کنیم. وقتی داریم از متد `forFeature()` استفاده می کنیم در واقع تکنیک partial registeration رو به کار بردیم.

متد `register()` زمانی که می خواهیم یک ماژول در هر ماژول مورد استفاده تنظیمات خاصی رو داشته باشه از این متد استفاده می کنیم.

## Configurable Module Builder-------------------

ساخت ماژول های قابل تنظیم بصورت خودکار---------------

ایجاد ماژول های قابل تنظیم که بصورت async عمل کنند ، بسیار سخت هست ، از همین رو  nest کلاس `ConfigurationModuleBuilder` رو در معرض دید قرار داده تا با استفاده از این کلاس بشه در چند خط ماژول های پویا ایجاد کرد.

در ابتدای کار ، باید interface ای رو تعریف کنیم که پارامتر های ورودی متد register رو مشخص میکنه:

```typescript
export interface ConfigModuleOptions {
  folder: string;
}
```

حالا در کنار فایل ، `config.module.ts` یه فایل تحت عنوان `config.module-definition.ts` ایجاد میکنیم:

`config.module-definition.ts`
```typescript
import { ConfigurableModuleBuilder } from '@nestjs/common';
import { ConfigModuleOptions } from './interfaces/config-module-options.interface';

export const { ConfigurableModuleClass, MODULE_OPTIONS_TOKEN } =
  new ConfigurableModuleBuilder<ConfigModuleOptions>().build();
```

حالا در فایل `config.module.ts` از `ConfigurableModuleClass` استفاده می کنیم:

`config.module.ts`
```typescript
import { Module } from '@nestjs/common';
import { ConfigService } from './config.service';
import { ConfigurableModuleClass } from './config.module-definition';

@Module({
  providers: [ConfigService],
  exports: [ConfigService],
})
export class ConfigModule extends ConfigurableModuleClass {}
```

حالا در ماژول مصرف کننده می تونیم از دو متد `register()` و `registerAsync()` برای کار های نا همزمان استفاده کنیم:

```typescript
@Module({
  imports: [
    ConfigModule.register({ folder: './config' }),
    // or alternatively:
    // ConfigModule.registerAsync({
    //   useFactory: () => {
    //     return {
    //       folder: './config',
    //     }
    //   },
    //   inject: [...any extra dependencies...]
    // }),
  ],
})
export class AppModule {}
```

در گام آخر هم در `ConfigService` به این option اعلام نیازمندی می کنیم:

```typescript
@Injectable()
export class ConfigService {
  constructor(@Inject(MODULE_OPTIONS_TOKEN) private options: ConfigModuleOptions) { ... }
}
```

## Custom Method Key----------------

کلاس `ConfigurableModuleClass` بصورت پیش فرض متد `register()` رو عرضه میکنه و `registerAsync()` رو عرضه میکنه ، برای اینکه بتونیم از متد ها با نام های دیگه مثل `forRoot()` و `forFeature()` استفاده کنیم ، به شکل زیر عمل می کنیم:

`config.module-definition.ts`
```typescript
export const { ConfigurableModuleClass, MODULE_OPTIONS_TOKEN } =
  new ConfigurableModuleBuilder<ConfigModuleOptions>().setClassMethodName('forRoot').build();
```

حالا در ماژول مصرف کننده ، می تونیم از دو متد `forRoot()` و `forRootAsync()` استفاده کنیم:

```typescript
@Module({
  imports: [
    ConfigModule.forRoot({ folder: './config' }), // <-- note the use of "forRoot" instead of "register"
    // or alternatively:
    // ConfigModule.forRootAsync({
    //   useFactory: () => {
    //     return {
    //       folder: './config',
    //     }
    //   },
    //   inject: [...any extra dependencies...]
    // }),
  ],
})
export class AppModule {}
```

## Custom options factory class------------------

#تکمیل_شود 
## Extra options----------------

#تکمیل_شود 

## Extending auto-generated methods--------------

#تکمیل_شود 