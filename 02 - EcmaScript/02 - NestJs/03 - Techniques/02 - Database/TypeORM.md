فریمورک nest بصورت داخلی از سه ORM برای ارتباط با دیتابیس پشتیبانی می کند و برای این سه orm پکیج هایی رو توسعه داده است:

`@nestjs/typeorm` , `@nestjs/sequelize` , `@nestjs/mongoose`

## TypeORM Integration---------------

sql-server

```ts
TypeOrmModule.forRoot({  
  host: "localhost",  
  port: 1433,  
  type: "mssql",  
  username: "sa",  
  password: "1234@abcd",  
  database: "CatDB",  
  synchronize: true,  
  autoLoadEntities:true,  
  options:{  
    encrypt:false  
  }
```

mongo - Db

```ts
TypeOrmModule.forRoot({  
  host: "localhost",  
  port: 27017,  
  type: "mongodb",  
  database: "CatDB",  
  synchronize: true,  
  autoLoadEntities:true,  
  // useUnifiedTopology: true,  
  // useNewUrlParser: true,})
```

```ts
@Entity()  
export class Cat {  
  @ObjectIdColumn()  
  id:ObjectId;  
  
  @Column()  
  name:string;  
  
  @Column()  
  age:number;  
  
  @Column()  
  breed:string;  
}
```

