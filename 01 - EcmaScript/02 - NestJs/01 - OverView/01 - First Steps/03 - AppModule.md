در واقع AppModule به عنوان RottModule ما شناخته میشه که تمام smallModule های موجود در برنامه ما رو در برمیگیره ، در واقع smallModule های برنامه ما ، در این ماژول ثبت می شوند تا به سیستم nest شناسونده بشن.

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

