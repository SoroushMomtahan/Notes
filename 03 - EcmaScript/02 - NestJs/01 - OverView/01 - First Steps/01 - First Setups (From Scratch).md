می تونیم بدون استفاده از nest cli نیز یک پروژه nest رو اجرا کنیم:

```bash
npm init -y
```

```bash
npm i @nestjs/common @nestjs/core @nestjs/platform-express reflect-metadata typescript
```

![](./Images/Pasted%20image%2020240504233817.png)

`tsconfig.json`
```json
{
	"compileOptions":{
		"module": "commonjs",
		"target": "ES2021",
		"experimentalDecorators": true,
		"emitDecoratorMetadata": true
	}
}
```

