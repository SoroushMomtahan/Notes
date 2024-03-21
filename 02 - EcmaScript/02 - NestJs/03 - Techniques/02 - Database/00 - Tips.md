sql-server typeorm

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

mongo - Db typeorm

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

