بعضی از کار ها وجود داره که برنامه باید منتظر بمونه تا آن کار ها تموم بشن ، 

برای اجرا این امر در provider ها می تونین از  `await / async` در کنار سینتکس `useFactory` استفاده کنید.

```typescript
{
  provide: 'ASYNC_CONNECTION',
  useFactory: async () => {
    const connection = await createConnection(options);
    return connection;
  },
}
```
