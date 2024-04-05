![](./Images/Pasted%20image%2020240302195338.png)[A Quick Introduction to NestJS | Mitrais Blog](https://www.mitrais.com/news-updates/a-quick-introduction-to-nestjs/)


```ts
@Module(
{ 
	imports: [ 
		MongooseModule.forRoot("mongodb://localhost:27017/project-01"), 
		UsersModule, 
	], 
	 controllers: [AppController], 
	 providers: [AppService], 
 }) 
 export class AppModule {}
```

```ts
@Module({ imports: [ MongooseModule.forFeature([ { name: User.name, schema: UserSchema, }, ]), ], controllers: [UsersController], providers: [UsersService], }) export class UsersModule {}
```

```ts
constructor(@InjectModel(User.name) private userModel: Model<User>) {}
```

```ts
@Schema()
export class User {
  @Prop()
  firstName: string;

  @Prop()
  lastName: string;

  @Prop()
  email: string;

  @Prop()
  password: string;
}

export const UserSchema = SchemaFactory.createForClass(User);
```

![](./Images/Pasted%20image%2020240310113913.png)
