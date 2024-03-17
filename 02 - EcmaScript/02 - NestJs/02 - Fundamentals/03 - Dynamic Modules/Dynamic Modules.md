در بخش overview و قسمت module درباره چیستی module ها صحبت کردیم ، در این قسمت می خواهیم نکات تکمیلی درباره module ها یعنی نحوه نوشتن dynamic module رو بررسی کنیم و پرونده module ها رو بطور کل ببندیم.

## Introduction--------------

تا اینجا تنها `static module` ها رو دیدیم.

```typescript
@Module({
  imports: [DogsModule],
  controllers: [CatsController],
  providers: [CatsService],
  exports: [CatsService]
})
```

اگر یک ماژول رو به شکل بالا تعریف کنیم ، در واقع یک static module نوشته ایم.

با تعریف یک ماژول بصورت static ، دیگه امکانی برای پیکربندی ماژول import شده در ماژول مصرف کننده فراهم نمیشه

ما می تونیم یه module رو dynamic تعریف کنیم تا مصرف کننده آن ماژول بتونه پیکربندی هایی رو روی ماژول مربوطه انجام بده و به نوعی ماژول رو سفارشی کنه.

## Example--------------

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

میشه این نتیجه رو گرفت که متد `register()` باید یه ماژولی رو Return کنه چون property ی imports ماژول قبول میکنه.

در حقیقت چیزی که متد `register()` برمیگردونه یه `DynamicModule` هست ، dynamic module ها در زمان run-time ساخته می شوند و تنها یک property اضافه تر از static-mosule ها به اسم ، `module` دارند ؛ این property نام ماژول رو مشخص میکنه که باید همنام با نام کلاس module مربوطه باشد ، ضمنا این property تنها property اجباری می باشد:

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

## Module Configuration--------------

حالا می خوایم بسته به نیاز ، یه سری options در ماژول مصرف کننده به ماژول import شده اضافه بشه و این option ها در provider مربوطه استفاده بشه:

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
```

##  Community guidelines-----------------

سه تا متد `forRoot()` و `forFeature()` و `register()` داریم که برای dynamic  کردن ماژول مورد استفاده قرار میگیرد.

متد `forRoot()` زمانی استفاده می شود که می خواهیم یک سری تنظیمات بر سر ماژول import شده انجام بدیم که برای همه برنامه این تنظیمات یکسانه.

متد `forFeature()` زمانی کاربرد داره که علاوه بر تنظیمات کلی که با استفاده از متد `forRoot()` اعمال کردیم ، می خواهیم یک سری تنظیمات خاص نیز اعمال کنیم.

متد `register()` زمانی که می خواهیم یک مازول در هر ماژول مورد استفاده تنظیمات خاصی رو داشته باشه از این متد استفاده می کنیم.


#تکمیل_شود 
# ادامه دارد ......




