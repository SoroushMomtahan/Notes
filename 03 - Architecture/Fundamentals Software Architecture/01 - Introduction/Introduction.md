چرا وقتی از شغل هایی مانند پرستاری صحبت می کنیم ، نقشه راه مشخصی برای آنها وجود داره اما برای شغل معماری نرم افزار نقشه راه مشخصی نیست؟

اول، صنعت تعریف خوبی از معماری نرم افزار ندارد.

کلا تعریف دقیقی از مهندسی نرم افزار وجود ندارد ، یه شخص معروف گفته مهندسی نرم افزار هر چی که می خواد باشه ولی می دونیم چیز مهمی هست.

شکل زیر ، دامنه معماری نرم افزار رو نشان میده:

![](./Images/Pasted%20image%2020240315131247.png)

قدیم معماران نرم افزار فقط به جنبه های فنی مانند : (modularity ، components ، patterns) توجه می کردند اما اکنون به دلیل بوجود آمدن سبک های جدید معماری مانند microservices ، نقش معماران نرم افزار گسترش پیدا کرده است.

همچنین یکی دیگه از علت هایی که تعریف درستی از معماری نرم افزار وجود نداره اینه که ، **هدف** معماری نرم افزار ، روز به روز در حال تغییر هست چون محیط زیست برنامه ها مددام در حال تحول هست.

تا وسط ص 22

module = package = namespace = مجموعه ای که درون آن ابزار هایی شبیه یا مرتبط بهم وجود دارد

برنامه نویسی ماژولار قبل از برنامه نویسی شی گرا (مبتنی بر کلاس) ارائه شد.

زمانی رسید که برنامه نویسان دریافتند هیچ راهی برای سازماندهی ویژگی هایی مرتبط با هم در زبان های برنامه نویسی آن زمان وجود ندارد ، بنابراین در عصر کوتاهی (عصری که زود به پایان رسید) زبان هایی ماژولار ارائه شد.

همانطور که گفتیم ، دوران برنامه نویسی ماژولار ، کوتاه بود و زبانهای برنامه نویسی شی گرا به شدت محبوب شدند چون راه های جدیدی رو برای برنامه نویسی ارائه می دادند ، اما همچنان این زبان های شی گرا سعی کردند قابلیت ماژولار بودن رو به خود اضافه کنند ، بطور مثال زبان جاوا از مفهوم package برای بحث ماژولار استفاده میکنه و یا .net از مفهوم namespace استفاده میکنه.

منظور از ماژولاره کردن ایجاد یک جدایی منطقی هست نه فیزیکی .



---

سوالاتی که برای پیدا کردن module ها در یک سیستم نرم افزاری باید از خود بپرسیم:

Identifying the modules of a software system is a crucial step in designing a well-organized and maintainable architecture. Here are some questions to ask yourself during this process:

1. **What Are the High-Level Functionalities?**:
    
    - Consider the primary functionalities or features that the system needs to provide.
    - Ask:
        - What are the core responsibilities of the application?
        - What problem does it solve for users?
2. **What Are the Key Components?**:
    
    - Break down the system into major components or areas of responsibility.
    - Ask:
        - What are the major building blocks of the system?
        - Which components can be logically separated?
3. **What Are the Subsystems?**:
    
    - Identify subsystems that handle specific aspects of the system.
    - Ask:
        - Are there distinct areas like authentication, data storage, reporting, etc.?
        - Which parts can be encapsulated as separate units?
4. **What Are the Dependencies?**:
    
    - Consider interactions between different parts of the system.
    - Ask:
        - Which components rely on others?
        - Which modules need to communicate?
5. **What Are the Data Flows?**:
    
    - Analyze how data moves through the system.
    - Ask:
        - How does information flow from input to output?
        - Which modules handle data transformation?
6. **What Are the Interfaces?**:
    
    - Identify external interfaces (e.g., APIs, user interfaces).
    - Ask:
        - How does the system interact with external systems or users?
        - Which modules expose APIs?
7. **What Are the Common Patterns?**:
    
    - Recognize recurring patterns or functionalities.
    - Ask:
        - Are there common tasks (e.g., logging, error handling) that can be modularized?
        - Which modules can be reused across different parts of the system?
8. **What Are the Security Boundaries?**:
    
    - Consider security and access control.
    - Ask:
        - Which modules handle authentication, authorization, and encryption?
        - Where are security boundaries drawn?
9. **What Are the Performance Considerations?**:
    
    - Think about performance-critical areas.
    - Ask:
        - Which modules need optimization?
        - Are there bottlenecks that require separate treatment?
10. **What Are the Business Domains?**:
    
    - Align modules with real-world business concepts.
    - Ask:
        - Which parts of the system correspond to specific business domains?
        - How can we model these domains as modules?

Remember that module identification is an iterative process. Start with high-level concepts and gradually refine your understanding by asking detailed questions. Collaborate with stakeholders, domain experts, and other team members to ensure a comprehensive view of the system’s structure. 🧩🌟