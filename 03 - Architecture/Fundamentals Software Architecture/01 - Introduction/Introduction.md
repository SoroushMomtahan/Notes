چرا وقتی از شغل هایی مانند پرستاری صحبت می کنیم ، نقشه راه مشخصی برای آنها وجود داره اما برای شغل معماری نرم افزار نقشه راه مشخصی نیست؟

اول، صنعت تعریف خوبی از معماری نرم افزار ندارد.

کلا تعریف دقیقی از مهندسی نرم افزار وجود ندارد ، یه شخص معروف گفته مهندسی نرم افزار هر چی که می خواد باشه ولی می دونیم چیز مهمی هست.

شکل زیر ، دامنه معماری نرم افزار رو نشان میده:

![](Pasted%20image%2020240315131247.png)

قدیم معماران نرم افزار فقط به جنبه های فنی مانند : (modularity ، components ، patterns) توجه می کردند اما اکنون به دلیل بوجود آمدن سبک های جدید معماری مانند microservices ، نقش معماران نرم افزار گسترش پیدا کرده است.

همچنین یکی دیگه از علت هایی که تعریف درستی از معماری نرم افزار وجود نداره اینه که ، **هدف** معماری نرم افزار ، روز به روز در حال تغییر هست چون محیط زیست برنامه ها مددام در حال تحول هست.

تا وسط ص 22

module = package = namespace = مجموعه ای که درون آن ابزار هایی شبیه بهم وجود دارد

برنامه نویسی ماژولار قبل از برنامه نویسی شی گرا (مبتنی بر کلاس) ارائه شد.

زمانی رسید که برنامه نویسان دریافتند هیچ راهی برای سازماندهی ویژگی هایی مرتبط با هم در زبان های برنامه نویسی آن زمان وجود ندارد ، بنابراین در عصر کوتاهی (عصری که زود به پایان رسید) زبان هایی ماژولار ارائه شد.

همانطور که گفتیم ، دوران برنامه نویسی ماژولار ، کوتاه بود و زبانهای برنامه نویسی شی گرا به شدت محبوب شدند چون راه های جدیدی رو برای برنامه نویسی ارائه می دادند ، اما همچنان این زبان های شی گرا سعی کردند قابلیت ماژولار بودن رو به خود اضافه کنند ، بطور مثال زبان جاوا از مفهوم package برای بحث ماژولار استفاده میکنه و یا .net از مفهوم namespace استفاده میکنه.

منظور از ماژولاره کردن ایجاد یک جدایی منطقی هست نه فیزیکی .

