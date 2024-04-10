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

فریمورک nest ، فرآیند نمونه سازی از class هایی که به این provider نیازمند اند را تا زمان انجام شدن این provider به تعویق می اندازد. به بیان دیگر تا زمانی که فرآیند async متد useFactory انجام نشود ، از کلاس هایی که به این provider نیازمند اند ، نمونه ساخته نمی شود.

