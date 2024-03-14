#exception

هر جایی اگر بنابر شرطی نیاز به پرتاب exception بود ، همونجا پرتاب میکنیم 

`users.service.ts`
```ts
@Get()
find(@Param('id') id:string){
	const user = users.get();
	if(!user){
		throw new HttpException('user not found', HttpStatus.NOT_FOUND);
		// OR
		throw new NotFoundException('user not found');
	}
	return user
}
```


#CLI
برای ایجاد کلاس بوسیله Nest CLI:

```bash
nest generate | g class users/dto/create-user.dto --no-spec
```

#Dto

یکی از کار های مهم زمان ایجاد dto class ها اینه که همه property های تعریف شده در این کلاس readonly تعریف شوند:

`create-user.dto.ts`
```ts
export class CreateUserDto {
	readonly name: string;
	readonly age: number;
}
```

#Dto 

نیاز به `update-users.dto.ts` در کنار `create-user.dto.ts` داریم ، چون می خوایم property های کلاس `UpdateUserDto` رو اختیاری تعریف کنیم:

```ts
export class UpdateUserDto {
	readonly name?: string;
	readonly age?: number;
}
```

اما یه راه حل بهتر برای اینکار وجود داره ، property هایی که به `UpdateUserDto` دادیم ، دقیقا برابر با property های `CreateUserDto` هستند ، با این تفاوت که property های `UpdateUserDto` اختیاری اند.

خود nest یه پکیجی رو ارائه داده که میاد property های یک کلاس رو به class دیگه میده با این تفاوت که اونا رو اختیاری شده به اون کلاس میده ، خب این پکیج رو نصب می کنیم:

```bash
npm i @nest/mapped-types
```

حالا property های کلاس `UpdateUserDto` رو پاک می کنیم و در ادامه:

```ts
import {PartialType} from `@nest/mapped-types`
export class UpdateUserDto extends PartialType(CreateUserDto){}
```

#validationPipe

```ts
@Post()  
@UsePipes(new ValidationPipe({  
  whitelist: true,  
  forbidNonWhitelisted: true  
}))  
create(@Body() createCatDto:CreateCatDto){  
  return createCatDto  
}
```

این `whitelist` باعث میشه هر property به جز property های dto-class رو ignore کنه.
و `forbidNonWhitelisted` هم باعث میشه اگه property ای خارج از dto-class داده شد یه exception پرتاب بشه.

#validationPipe 

داده ورودی از `@Body()` و `@Param()` و `@Query()` از نوع تایپی که براش مشخص می کنیم نخواهد بود

```ts
@Post()  
@UsePipes(new ValidationPipe({  
  whitelist: true,  
  transform:true,  
  forbidNonWhitelisted: true  
}))  
create(@Body() createCatDto:CreateCatDto){  
  console.log(typeof createCatDto);  
  console.log(createCatDto instanceof CreateCatDto);  // false before set  transform:true,
  return createCatDto  
}  
  
@Get()  
find(@Param() param:number){  
  console.log(typeof param); // object  
  return param;  
}
```

#dependencyInjection



