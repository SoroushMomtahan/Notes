
## Provider-Scope--------------

می خوایم تعیین کنیم که هر نمونه که توسط IOC Container ساخته میشه ، چه طول عمری داشته باشه ، در طول برنامه فقط یک نمونه ساخته بشه `DEFAULT` 
و یا با هر درخواست جدیدی که میاد یه نمونه جدید ساخته بشه `REQUEST` 
و یا به ازای هر بار نیازمندی به یک سرویس یک نمونه جدید از آن سرویس ساخته بشه `TRANSIENT`

|             |                                                   |
| ----------- | ------------------------------------------------- |
| `DEFAULT`   | یک نمونه در کل برنامه                             |
| `REQUEST`   | یک نمونه جدید به ازای هر درخواست                  |
| `TRANSIENT` | یک نمونه جدید به ازای هر بار نیازمندی به آن سرویس |

>[!tip]
>توصیه میشه که از حالت دیفالت یعنی `SINGLETON` در همه موارد استفاده بشه چون تنها یک نمونه از سرویس در کل برنامه ساخته میشه و برای دفعات بعد این نمونه کش میشه که همین به performance برنامه کمک میکنه.

## Usage-------------

طریقه تعیین محدوده یک نمونه از سرویس به شکل زیر هست:

```typescript
import { Injectable, Scope } from '@nestjs/common';

@Injectable({ scope: Scope.REQUEST })
export class CatsService {}
```

برای custom-provider ها هم به شکل زیر محدوده رو مشخص می کنیم:

لازم به ذکره که این روش با روش بالا هیچ فرقی ندارد.

```typescript
{
  provide: 'CACHE_MANAGER',
  useClass: CacheManager,
  scope: Scope.TRANSIENT,
}
```

>[!warning]
>نکته اینکه Websocket Gateways ها نباید از provider هایی با محدوده درخواست استفاده کنند.

برای controller ها نیز می تونیم scope تعریف کنیم:

```typescript
@Controller({
  path: 'cats',
  scope: Scope.REQUEST,
})
export class CatsController {}
```

## Scope Hierarchy----------------

سلسله مراتب در scope ها رو با یه مثال جمع می کنیم ، سه کلاس زیر رو در نظر بگیرید که دارای گراف زیر هستند:

`CatsController` <- `CatsService` <- `CatsRepository`

اگه `CatsService` دارای محدوده درخواست (بقیه رو محدوده singleton در نظر بگیرید) باشه ، در این صورت همه بالا دستی های خودش رو دارای محدوده request میکنه ، پس یعنی `CatsController` در محدوده request قرار میگیره

در مورد محدوده `SINGLETON` و `TRANSIANT` این مورد وجود ندارد و باعث تغییر محدوده دیگری نمی شوند.

## Request Provider---------------

اگر یک class دارای محدوده request شود و بخواهیم از شی request پکیج express در این کلاس استفاده کنیم ، خیالمون راحته که به ازای هر درخواست یک شی جدید به امضای متد مربوطه پاس داده میشه:

```typescript
import { Injectable, Scope, Inject } from '@nestjs/common';
import { REQUEST } from '@nestjs/core';
import { Request } from 'express';

@Injectable({ scope: Scope.REQUEST })
export class CatsService {
  constructor(@Req() private request: Request) {}
}
```

با اینحال اگر از محدوده request استفاده نمی کنیم و می خواهیم `@Req()` به ازای هر درخواست یه نمونه جدید بهمون بده از دکوراتور `@Inject(REQUEST)` استفاده می کنیم:

```typescript
import { Injectable, Scope, Inject } from '@nestjs/common';
import { REQUEST } from '@nestjs/core';
import { Request } from 'express';

@Injectable()
export class CatsService {
  constructor(@Inject(REQUEST) private request: Request) {}
}
```

اگر از qraph-ql استفاده می کنیم باید به شکل زیر عمل کنیم:

```typescript
import { Injectable, Scope, Inject } from '@nestjs/common';
import { CONTEXT } from '@nestjs/graphql';

@Injectable()
export class CatsService {
  constructor(@Inject(CONTEXT) private context) {}
}
```

##  Inquirer Provider----------------

اگر بخوایم در کلاس تزریقی اسم کلاسی که به کلاس تزریقی نیازمنده رو بفهمیم از دکوراتور `@Inject(INQUIRE)` استفاده می کنیم:

```typescript
import { Inject, Injectable, Scope } from '@nestjs/common';
import { INQUIRER } from '@nestjs/core';

@Injectable({ scope: Scope.TRANSIENT })
export class HelloService {
  constructor(@Inject(INQUIRER) private parentClass: object) {}

  sayHello(message: string) {
    console.log(`${this.parentClass?.constructor?.name}: ${message}`);
  }
}
```

```typescript
import { Injectable } from '@nestjs/common';
import { HelloService } from './hello.service';

@Injectable()
export class AppService {
  constructor(private helloService: HelloService) {}

  getRoot(): string {
    this.helloService.sayHello('My name is getRoot'); 
    // console -> AppService: My name is getRoot
    
    return 'Hello world!';
  }
}
```

## Performance----------------

درسته که می گیم استفاده از محدوده request کارایی برنامه رو پایین میاره ، اما این پایین اومدن کارایی کمتر از 5 درصده.

## Durable providers-----------------

#تکمیل_شود 

این قسمت بعدا کامل می شود........