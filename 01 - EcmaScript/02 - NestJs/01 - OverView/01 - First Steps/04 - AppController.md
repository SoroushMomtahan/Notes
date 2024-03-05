```ts
@Controller()  
export class AppController {  
  constructor(private readonly appService: AppService) {}  
  
  @Post()  
  getHello() {  
    return this.appService.getHello();  
  }  
}
```

