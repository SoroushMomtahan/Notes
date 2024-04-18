
زبان های script ای مانند python و php و… زبان هایی بودند که functional بودند اما زبان هایی مثل c# و java و… زبان هایی بوند که از ابتدا بر اساس کلاس توسعه داده شدند.

---

```ts
if(0 === count){}
```

```ts
if(cont === 0){}
```

>[!tip]
>استفاده از عبارت دوم بهتر است
>
>

---

برای edit یک سایت بصورت زنده این کد رو در console مرورگر وارد می کنیم:

```ts
document.designMode = 'on'
```

---

هنگامی که از arrow function برای تعریف تابع استفاده می کنیم و می خواهیم یک object رو به عنوان return بفرستیم ، برای اینکه {} با بدنه تابع اشتباه گرفته نشه {} رو داخل () می گذاریم:

```ts
export default () => ({
	name: 'soroush'
})
```

---

#pagination

![](./Images/Pasted%20image%2020240402103947.png)

---

#validationPipe 

```ts
export default {
  validation: (validationCompile) => {
    return (req, res, next) => {
      const body = req.body;
      const validationResult = validationCompile(body);
      if (validationResult !== true) {
        return res.status(422).json({ message: 'Validation failed' });
      }
      next()
    }
  }
}
```

```ts
router.post('/country', validation(createOneValidation),countriesController.createOne);
```


#sendResponse #express 

```ts
export const successResponse = ({
  res,
  statusCode = 200,
  message = null,
  data = null
}) => {
  return res
    .status(statusCode)
    .json({ status: statusCode, success: true, message, data });
};
export const errorResponse = ({
  res,
  statusCode,
  message = null,
  data = null
}) => {
  return res
    .status(statusCode)
    .json({ status: statusCode, success: false, message, data });
};
```

```ts
export default (req, res, next) => { return errorResponse({ res, statusCode: 404, message: '404 | Page Not Found' }); };
```

---

#adminPanel

[https://github.com/evershopcommerce/evershop](https://github.com/evershopcommerce/evershop)

#mongoose #express 

```ts
schema.pre('save', async function (next) { try { this.password = bcrypt.hashSync(this.password); next(); } catch (err) { next(err); } }); schema.pre('updateOne', function (next) { try { const password = this.get('password'); if (password) { this.set({ password: bcrypt.hashSync(password) }); } next(); } catch (err) { next(err); } });
```

