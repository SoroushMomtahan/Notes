## You Arent Gonna Need It
""شما به آن نیاز ندارد""
ذهن برنامه نویس بصورت general کار می کند ، یعنی هر چیز رو می خواد در بهترین حالت خود داشته باشد ؛
اما باید این بهترین خواهی با نگاه به پروژه باشه ، مثلا برای پروژه ای که قرار نیست توسعه زیادی داشته باشه ، ایجاد تمهیداتی برای اینکه مثلا شاید در آینده رفتیم رو nosql پس از الان زیر ساختشو درست کنیم ، کار اشتباهی هست.
`users/IUser.repository.ts`
```ts
export interface IUserRepository{}
```

`users/sqlDb.ts`
```ts
export class SqlDb implement IUserRepository{}
```

`user/noSqlDb.ts`
```ts
export class NoSqlDb implement IUserRepository{}
```
کار هایی که بالا انجام دادیم ، برای پروژه های بزرگ خوبه ، اما برای پروژه های کوچک کار اضافی هست.
