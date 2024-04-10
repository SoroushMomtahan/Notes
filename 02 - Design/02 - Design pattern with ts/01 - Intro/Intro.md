>[!tip]
>فایل `.env` رو باید داخل gitignore قرار بدیم ، چون در git بهشون نیازی نیست ؛ اگر احیانا چیزی در .env هست که می خوایم در git قرار بگیره ، یه فایل به اسم `.env.example` ایجاد می کنیم و داخل اون آن دست از مواردی که لازمه رو می نویسیم.

`app.ts`
```typescript
export class Application{
    private app:Express
    constructor() {
        this.app = express();
    }
    public run(port:number):void{
        this.app.listen(port, ()=>{
           console.log(`run > http://localhost:${port}`); 
        });
    }
}

```

`server.ts`
```ts
const app = new Application();
app.run(process.env.PORT);
```

`@types/express/index.d.ts`
```ts
export {}
declare global {
    namespace Express{
        interface Request {
            authenticatedUser?: IUser;
        }
    }
}
```

`tsconfig.json`
```json
"typeRoots": ["node_modules/@types", "@types"],
```