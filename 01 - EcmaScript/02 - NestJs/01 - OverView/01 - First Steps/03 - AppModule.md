در واقع AppModule به عنوان Root module ما شناخته میشه که تمام small module های موجود در برنامه ما رو در برمیگیره ، در واقع small module های برنامه ما ، در این ماژول ثبت می شوند تا به سیستم nest شناسونده بشن.

```ts
import { Module } from '@nestjs/common';  
import { AppController } from './app.controller';  
import { AppService } from './app.service';  
  
@Module({  
  controllers: [AppController],  
  providers: [AppService],  
})  
export class AppModule {}
```

با استفاده از دکوراتور `@Module()` کلاس AppModule رو گسترش دادیم.