از انواع و اقسام ORM ها در nest میشه استفاده کرد اما فریمورک nest بصورت داخلی از سه ORM برای ارتباط با دیتابیس پشتیبانی می کند و برای این سه orm پکیج هایی رو توسعه داده است:

`@nestjs/typeorm` , `@nestjs/sequelize` , `@nestjs/mongoose`

این به این معنا نیست که نمیشه از ORM های دیگه در nest استفاده کرد اما استفاده از ORM های بالا در nest راحت تره و برای مثال برای استفاده از Prisma در nest نیاز به پیکربندی های بیشتریست.

## TypeORM Integration---------------

پیشنهاد اول nest استفاده از TypeORM است ، به علت اینکه از ابتدا با TS نوشته شده ، خیلی با سیستم nest هماهنگ هست.

در این قسمت از TypeORM و دیتابیس MySQL استفاده کردیم:

```bash
npm install --save @nestjs/typeorm typeorm mysql2
```

`app.module.ts`
```typescript
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';

@Module({
  imports: [
    TypeOrmModule.forRoot({
      type: 'mysql',
      host: 'localhost',
      port: 3306,
      username: 'root',
      password: 'root',
      database: 'test',
      entities: [],
      synchronize: true,
    }),
  ],
})
export class AppModule {}
```

از کد بالا میشه نتیجه گرفت که `TypeOrmModule` یه DynamicModule هست و یه سری option رو در run-time میگیره و اونارو به جای مناسب تزریق میکنه.

>[!warning]
>از `synchronize: true` تنها در محیط development استفاده شود ؛ استفاده از این property در محیط production ، باعث از دست رفتن داده در این محیط می شود.

گویا یه کلاسی در TypeOrm به نام `DataSource` وجود داره که آرگومان های constructor این کلاس رو میشه به option متد forRoot داد ، لازم به ذکر هست که option متد forRoot از همه آرگومان های این constructor در قالب object ای که شامل property هست ، پشتیبانی میکنه ، به علاوه اینکه این آقای option از یه سری property های دیگه که nest اونارو توسعه داده هم پشتیبانی میکنه:

|   |   |
|---|---|
|`retryAttempts`|Number of attempts to connect to the database (default: `10`)|
|`retryDelay`|Delay between connection retry attempts (ms) (default: `3000`)|
|`autoLoadEntities`|If `true`, entities will be loaded automatically (default: `false`)|
حالا وقتی این تزریق وابستگی انجام شد ، می تونیم به دو شی از کلاس های `DataSource` و `EntityManager` اعلام نیازمندی کنیم:

`app.module.ts`
```typescript
import { DataSource } from 'typeorm';

@Module({
  imports: [TypeOrmModule.forRoot(), UsersModule],
})
export class AppModule {
  constructor(private dataSource: DataSource) {}
}
```

## Repository Pattern-----------------

![](./Images/Pasted%20image%2020240310202857%201.png)

در TypeORM ، هر **موجودیت** Repository شخصی خودشو داره که این repository از `DataSource` بدست میاد. در واقع Repository یه abstract (انتزاع) از Data Source بوجود آورده و دارای متد های کاربردی فراوانی هست.

گفتیم هر موجودیت repository خاص خودشو داره ، پس برای اینکه Repositpry داشته باشیم ، قبلش باید یه Enitiy داشته باشیم ، پس بیایم یه Entity به اسم `User` ایجاد کنیم:

`user.entity.ts`
```typescript
import { Entity, Column, PrimaryGeneratedColumn } from 'typeorm';

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  firstName: string;

  @Column()
  lastName: string;

  @Column({ default: true })
  isActive: boolean;
}
```

اگه بخوایم یکم کد بالا رو از نظر مفهومی بررسی کنیم ، `User` یه موجودیته پس بالاش دکوراتور `@Entity()` رو گذاشتیم ، حالا هر property از این کلاس نقش یه ستون از جدول رو دارند ، پس بالای هر property دکوراتور `@Column()` گذاشتیم ، کلید در جدول هم یه ستونه اما ستونی که خودش مقداردهی میشه پس بالاش دکوراتور `@PrimaryGeneratedColumn()` تا یه کلید در نظر گرفته بشه.

همه column ها بصورت پیش فرض `required` هستند.

فایل `user.entity.ts` رو در فایل `users` ایجاد کردیم ، هر چند که می تونیم این فایل هر جای دیگری تعریف کنیم ، اما چون این فایل با ماژول `UserModule` ارتباط داره ، بهتره اونو در پوشه `users` تعریف کنیم.

در حالت عادی هر نامی که entity class ما داشته باشه lowerCase میشه و به عنوان نام Table در database در نظر گرفته میشه مثلا الان نام table ای که در دیتابیس ساخته میشه `user` هست ، اگه بخوایم نام دیگه ای برای table مون در نظر بگیریم باید در پارامتر اول دکوراتور `@Entity()`نام دلخواه مون رو بدیم:

```ts
import { Entity, Column, PrimaryGeneratedColumn } from 'typeorm';

@Entity('users') // sql table === users
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  firstName: string;

  @Column()
  lastName: string;

  @Column({ default: true })
  isActive: boolean;
}
```

حالا برای اینکه بتونیم از این entity استفاده کنیم کنیم ، ابتدا باید این موجودیت رو به `TypeOrmModule` بشناسونیم ، برای اینکار از property ی `entities` که آرایه ای از موجودیت ها رو میگیره ، استفاده می کنیم:

`app.module.ts`
```typescript
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { User } from './users/user.entity';

@Module({
  imports: [
    TypeOrmModule.forRoot({
      type: 'mysql',
      host: 'localhost',
      port: 3306,
      username: 'root',
      password: 'root',
      database: 'test',
      entities: [User],
      synchronize: true,
    }),
  ],
})
export class AppModule {}
```

حالا علاوه بر این ، باید در `UsersModule` هم مشخص کنیم که در ماژول مربوطه ، قراره از چه موجودیتی استفاده بشه:

`users.module.ts`
```typescript
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { UsersService } from './users.service';
import { UsersController } from './users.controller';
import { User } from './user.entity';

@Module({
  imports: [TypeOrmModule.forFeature([User])],
  providers: [UsersService],
  controllers: [UsersController],
})
export class UsersModule {}
```

حالا می تونیم به `UserRepository`  در `UsersService` اعلام نیازمندی کنیم:
`users.service.ts`
```typescript
import { Injectable } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { User } from './user.entity';

@Injectable()
export class UsersService {
  constructor(
    @InjectRepository(User)
    private usersRepository: Repository<User>,
  ) {}

  findAll(): Promise<User[]> {
    return this.usersRepository.find();
  }

  findOne(id: number): Promise<User | null> {
    return this.usersRepository.findOneBy({ id });
  }

  async remove(id: number): Promise<void> {
    await this.usersRepository.delete(id);
  }
}
```

حالا اگه بخوایم از این repository ای که برای user ایجاد شده ، خارج از ماژول `UsersModule` استفاده کنیم ، باید `TypeOrmModule` در `UsersModule` رو export کنیم:

`users.module.ts`
```typescript
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { User } from './user.entity';

@Module({
  imports: [TypeOrmModule.forFeature([User])],
  exports: [TypeOrmModule]
})
export class UsersModule {}
```

حالا می تونیم `UsersModule` رو در `UsersHttpModule` وارد (import) کنیم و به راحتی از دکوراتور `@InjectRepository(User)` در service مربوطه ، استفاده کنیم:

`users-http.module.ts`
```typescript
import { Module } from '@nestjs/common';
import { UsersModule } from './users.module';
import { UsersService } from './users.service';
import { UsersController } from './users.controller';

@Module({
  imports: [UsersModule],
  providers: [UsersService],
  controllers: [UsersController]
})
export class UserHttpModule {}
```

## Relations-----------------

از Relations برای ایجاد وابستگی بین دو یا چند table استفاده می شود. در واقع Relation  بالای Property های اشتراکی قرار میگیره.

|   |   |
|---|---|
|`One-to-one`|هر سطر از جدول مرجه تنها و تنها با با یک سطر از جدول خارجی در ارتباط هست `@OneToOne()`
|`One-to-many / Many-to-one`|هر سطر از جدول مرجع ، با یک یا بیشتر از یک سطر از جدول خارجی در ارتباط هست `@OneTOMany()` & `ManyToOne()`|
|`Many-to-many`|هر سطر از جدول مرجع با چندین سطر از جدول خارجی در ارتباط هست و هر سطر از جدول خارجی نیز با چندین سطر از جدول مرجه در ارتباط هست `@ManyToMany()`|

حالا می خوایم مشخص کنیم که هر user می تونه چندین photo داشته باشه:

`user.entity.ts`
```typescript
import { Entity, Column, PrimaryGeneratedColumn, OneToMany } from 'typeorm';
import { Photo } from '../photos/photo.entity';

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  firstName: string;

  @Column()
  lastName: string;

  @Column({ default: true })
  isActive: boolean;

  @OneToMany(type => Photo, photo => photo.user)
  photos: Photo[];
}
```

## Auto-load Entities----------------

اینکه بخوایم در `AppModule` موجودیت ها رو با استفاده از property ی `entites` مشخص کنیم ، کار خسته کننده ای خواهد بود ، علاوه بر این ، این کار باعث میشه اصل ماژولاریته بودن رو نقض کنیم. برای رفع این مشکل ، می تونیم property ی `autoLoadEntities` رو در option متد `forRoot()` روی `true` قرار بدیم ، با این کار تمام entity های موجود در پروژه بصورت خودکار load می شوند.

`app.module.ts`
```typescript
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';

@Module({
  imports: [
    TypeOrmModule.forRoot({
      ...
      autoLoadEntities: true,
    }),
  ],
})
export class AppModule {}
```

#تکمیل_شود 

یه warning وجود داره که الان متوجش نشدم.

## Seprating Entity Defination------------------


علاوه بر اینکه میشه با استفاده از دکوراتور ها Entity مورد نظر رو تعریف کرد ، می توان از روش `Entity Schemas` هم اقدام به تعریف entity مورد نظر کرد.

لازم به ذکر هست که در این روش همچنان نیاز به تعریف فایل `user.entity.ts` داریم ، اما بجای استفاده از دکوراتور در این فایل ، فایلی دیگر ایجاد می کنیم و در آن `Entity Schemas` مورد نظر رو می نویسیم:

`user.entity.ts`
```typescript
import { Entity, Column, PrimaryGeneratedColumn, OneToMany } from 'typeorm';
import { Photo } from '../photos/photo.entity';

@Entity()
export class User {
  id: number;
  firstName: string;
  lastName: string;
  isActive: boolean;
  photos: Photo[];
}
```

`user.schema.ts`
```typescript
import { EntitySchema } from 'typeorm';
import { User } from './user.entity';

export const UserSchema = new EntitySchema<User>({
  name: 'User',
  target: User,
  columns: {
    id: {
      type: Number,
      primary: true,
      generated: true,
    },
    firstName: {
      type: String,
    },
    lastName: {
      type: String,
    },
    isActive: {
      type: Boolean,
      default: true,
    },
  },
  relations: {
    photos: {
      type: 'one-to-many',
      target: 'Photo', // the name of the PhotoSchema
    },
  },
});
```

>[!warning]
>اگر property ی `target` رو به کار ببریم ، property ی `name` باید string ای همنام با property ی target باشد ، اما اگر property ی `target` رو به کار ببریم ، property ی `name` می تواند هر string ای باشد.

حالا می تونیم از schema نوشته شده استفاده کنیم:

`users.module.ts`
```typescript
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { UserSchema } from './user.schema';
import { UsersController } from './users.controller';
import { UsersService } from './users.service';

@Module({
  imports: [TypeOrmModule.forFeature([UserSchema])],
  providers: [UsersService],
  controllers: [UsersController],
})
export class UsersModule {}
```

## TypeORM Transactions-----------------

#تکمیل_شود 

## Subscribers-------------------

#تکمیل_شود 

## Migrations-----------------

#تکمیل_شود 

## Multiple Databases---------------

#تکمیل_شود 

## Testing-------------

#تکمیل_شود 

## Async Configuration--------------

می تونیم option ای که قراره به ماژول `TypeOrmModule` پاس می دیم رو بصورت async پاس بدیم ، برای اینکار از متد `forRootAsync()` استفاده می کنیم ، این متد راه های مختلفی رو برای سر و کله زدن با پیکربندی های async به ما ارائه میده

یکی از این راه ها ، استفاده از `useFactory` function هست:

```typescript
TypeOrmModule.forRootAsync({
  useFactory: () => ({
    type: 'mysql',
    host: 'localhost',
    port: 3306,
    username: 'root',
    password: 'root',
    database: 'test',
    entities: [],
    synchronize: true,
  }),
});
```

در واقع متد `useFactory` برای ایجاد custom provider ها کاربرد داشت (بخش techniques قسمت custom provider ) و از طرف دیگه وقتی می خواستیم dynamic module ایجاد کنیم property های درون دکوراتور `@module()` رو درون متد static ای در کلاس ماژول مربوطه می نوشتیم و return می کردیم ، حالا برای اینکه ایجاد dynamic ماژول هایی که async باشند توسط خود برنامه نویس ، کار سختیه ، nest کلاس `ConfigurationModuleBuilder` رو در معرض دید قرار داده تا با استفاده از اون برنامه نویسان بتونن ماژول های داینامیک که async هستند ، بسازند. حالا این کلاس یعنی `ConfigurationModuleBuilder` همون ابزار های custom provider ها یعنی `useClass` و `useFactory` و `useExsiting` رو طبق فرآیندی کاری کرده که در متد های async در بطور مثال `forRootAsync()` در دسترس باشند.

```typescript
TypeOrmModule.forRootAsync({
  imports: [ConfigModule],
  useFactory: (configService: ConfigService) => ({
    type: 'mysql',
    host: configService.get('HOST'),
    port: +configService.get('PORT'),
    username: configService.get('USERNAME'),
    password: configService.get('PASSWORD'),
    database: configService.get('DATABASE'),
    entities: [],
    synchronize: true,
  }),
  inject: [ConfigService],
});
```

بدین ترتیب ، وقتی داریم از متد های static ای مثل `forRootAsync`  که مربوط به dynamic module ها استفاده می کنیم ، باید فرض کنیم که داریم خودمون پیاده سازی متد `forRootAsync` رو مانند کاری که در dynamic ماژول ها می کردیم ، می نویسیم.

همچنین میشه از سینتکس useClass هم استفاده کرد:

```typescript
TypeOrmModule.forRootAsync({
  useClass: TypeOrmConfigService,
});
```

#تکمیل_شود 

## Custom DataSource Factory----------------

#تکمیل_شود 


