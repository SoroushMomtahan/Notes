نقطه شروع یک برنامه nest ای از main.ts هست.

```ts
import { NestFactory } from '@nestjs/core';  
import { AppModule } from './app.module';  
  
async function bootstrap() {  
  const app = await NestFactory.create(AppModule);  
  await app.listen(3000);  
}  
bootstrap();
```

کارخونه نست `NestFactory` قربون دستت بساز `create` برنامه رو با نگاه به `AppModule` و برنامه ای که ساختی رو بریزش تو ظرف `app` 

حالا به app میگیم به port 3000 گوش کن

بعد این دو تا کار رو می پیچیمش دور یه function که خودکفا `bootstrap()` هست و دستش جلو کسی دراز نیست.

در آخر هم اونی که خودکفا هست خودشو صدا میزنه 

حالا این `AppModule` چیه ؟

