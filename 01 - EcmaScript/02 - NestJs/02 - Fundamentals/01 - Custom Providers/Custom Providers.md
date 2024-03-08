```ts
const myProviderFactory={  
  provide: 'MY_PROVIDER',  
  useFactory: ()=>{  
    return `this is my Provider factory`;  
  }  
}  
@Module({  
  controllers:[CatsController],  
  providers:[myProviderFactory]  
})  
export class CatsModule {  
  
}
```

```ts
@Controller('cats')  
export class CatsController {  
  
  constructor(@Inject('MY_PROVIDER') private readonly value:string) {  
  }  @Get()  
  findOne() {  
    return this.value  
  }  
}
```

----

در قسمت بخش  [Providers](Providers.md) 