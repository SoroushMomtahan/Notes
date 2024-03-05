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

حالا این AppModule چیه ؟

