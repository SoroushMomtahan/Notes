از انواع و اقسام ORM ها در nest میشه استفاده کرد اما فریمورک nest بصورت داخلی از سه ORM برای ارتباط با دیتابیس پشتیبانی می کند و برای این سه orm پکیج هایی رو توسعه داده است:

`@nestjs/typeorm` , `@nestjs/sequelize` , `@nestjs/mongoose`

این به این معنا نیست که نمیشه از ORM های دیگه در nest استفاده کرد اما استفاده از ORM های بالا در nest راحت تره و برای مثال برای استفاده از Prisma در nest نیاز به پیکربندی های بیشتریست.

## Sequalize Integration-----------------

برای استفاده از پکیج sequalize ، علاوه بر نصب این پکیج ، باید پکیج
sequalize-typescript ([sequelize/sequelize-typescript: Decorators and some other features for sequelize (github.com)](https://github.com/sequelize/sequelize-typescript) 
هم نصب بشه:

```bash
$ npm install --save @nestjs/sequelize sequelize sequelize-typescript mysql2
$ npm install --save-dev @types/sequelize
```

پس از نصب پکیج های مورد نیاز می تونیم `SequalizeModule` رو import کنیم:

`app.module.ts`
```typescript
import { Module } from '@nestjs/common';
import { SequelizeModule } from '@nestjs/sequelize';

@Module({
  imports: [
    SequelizeModule.forRoot({
      dialect: 'mysql',
      host: 'localhost',
      port: 3306,
      username: 'root',
      password: 'root',
      database: 'test',
      models: [],
    }),
  ],
})
export class AppModule {}
```

متد `forRoot()` از همه property های پیکربندی ای که پکیج sequalize برای پیکربندی ارائه کرده ، پشتیبانی میکنه ، به علاوه یه سری propeerty های پیکربندی نیز وجود داره که nest اونا ارائه داده:

|   |   |
|---|---|
|`retryAttempts`|Number of attempts to connect to the database (default: `10`)|
|`retryDelay`|Delay between connection retry attempts (ms) (default: `3000`)|
|`autoLoadModels`|If `true`, models will be loaded automatically (default: `false`)|
|`keepConnectionAlive`|If `true`, connection will not be closed on the application shutdown (default: `false`)|
|`synchronize`|If `true`, automatically loaded models will be synchronized (default: `true`)|

وقتی موارد بالا انجام شد همه چیز برای اعلام نیازمندی آماده است:

`app.service.ts`
```typescript
import { Injectable } from '@nestjs/common';
import { Sequelize } from 'sequelize-typescript';

@Injectable()
export class AppService {
  constructor(private sequelize: Sequelize) {}
}
```

## Models--------------------



## Relations--------------------

## Auto-Load Models--------------------

## Sequalize Transactions----------------

#تکمیل_شود 

## Migrations--------------------

#تکمیل_شود 

## Multiple Databases-----------------

#تکمیل_شود 

## Testing---------------

#تکمیل_شود 

## Async Configuration----------------

