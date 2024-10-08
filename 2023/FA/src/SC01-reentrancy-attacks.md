## آسیب‌پذیری: حملات بازگشت‌پذیر (Reentrancy attack)

### توضیحات:

حمله بازگشت مجدد یا Reentrancy attack درواقع نوعی آسیب پذیری در قراردادهای هوشمند هست به شکلی که با استفاده از تابعی به نام Fallback برای فراخوانی مجدد استفاده می‌کند. این امر می‌تواند منجر به از دست دادن دارایی ها در قرارداد‌های هوشمند شود. در زبان برنامه نویسی سالیدیتی تابعی به نام Fallback  وجود دارد که پارامتی نداشته و به صورت دلخواه برنامه نویس می‌تواند درون کد قراردهد.

### بررسی سناریو عادی:

طبق تصویر قطعه کدی که قرار دارد دارای دو تابع واریزی و برداشت می‌باشد. درواقع کاربر با فراخوانی تابع deposit می‌تواند مقدار اتر مورد نیاز را به قرارداد ارسال کند و با فراخوانی تابع withdraw می‌تواند مقدار اتر واریز شده را برداشت بزند. و در خط اخر (هایلایت شماره 3) مقدار بالانس کاربر در قرارداد برابر با 0 می‌شود.
(تصویر کد)


### بررسی سناریو حمله:

در سناریوی حمله، مهاجم با استفاده از یک قرارداد مخرب، آسیب‌پذیری موجود در قرارداد اصلی را هدف قرار می‌دهد. در اینجا، مهاجم با فراخوانی تابع `withdraw` قرارداد اصلی، سعی می‌کند وجوه خود را برداشت کند. اما به جای اینکه تابع `withdraw` تنها یک بار فراخوانی شود، مهاجم از آسیب پذیری بازگشت مجدد (Reentrancy) استفاده می‌کند تا این تابع را قبل از به‌روزرسانی موجودی کاربر در قرارداد اصلی، چندین بار فراخوانی کند.

نحوه عملکرد این حمله به این صورت است:

1. مهاجم ابتدا مقداری اتر به قرارداد اصلی واریز می‌کند.
2. سپس تابع `withdraw` را فراخوانی می‌کند تا وجوه خود را برداشت کند.
3. تابع `withdraw` ابتدا وجوه را به آدرس مهاجم ارسال می‌کند و سپس موجودی را به‌روزرسانی می‌کند.
4. در اینجا، تابع `fallback` قرارداد مهاجم فراخوانی می‌شود و درون این تابع، مهاجم دوباره تابع `withdraw` را قبل از اینکه موجودی به‌روزرسانی شود، فراخوانی می‌کند.
5. این فرایند تا زمانی که تمام وجوه قرارداد اصلی تخلیه شود، ادامه می‌یابد.


### شدت تاثیرگذاری آسیب‌پذیری‌:


- **خروج غیرمجاز وجوه:** مهاجمان با بهره‌گیری از این آسیب‌پذیری می‌توانند بیشتر از میزان مجاز وجوه برداشت کنند و حتی موجودی قرارداد را کاملاً تخلیه کنند.
- **فراخوانی غیرمجاز توابع:** مهاجم می‌تواند توابعی را به صورت غیرمجاز فراخوانی کند که این می‌تواند منجر به انجام عملیات ناخواسته در قرارداد یا سیستم‌های مرتبط شود.

### توصیه ها:

- **به‌روزرسانی موجودی‌ها قبل از فراخوانی کد خارجی:** همیشه اطمینان حاصل کنید که هر تغییر وضعیت داخلی قبل از فراخوانی قراردادهای خارجی انجام می‌شود. به عبارت دیگر، ابتدا موجودی‌ها یا کد داخلی را به‌روزرسانی کنید و سپس کد خارجی را فراخوانی کنید.
- **استفاده از Re-entrancy Guard:** از `ReentrancyGuard` یا محافظ‌های مشابه برای جلوگیری از بازگشت مجدد استفاده کنید، مانند کتابخانه‌های OpenZeppelin.


### مثال‌هایی از قراردادهای هوشمندی که قربانی حملات بازگشت‌پذیر شده‌اند:
1. [Rari Capital](https://etherscan.io/address/0xe16db319d9da7ce40b666dd2e365a4b8b3c18217#code) : A Comprehensive [Hack Analysis](https://blog.solidityscan.com/rari-capital-re-entrancy-vulnerability-analysis-25df2bbfc803)
2. [Orion Protocol](https://etherscan.io/address/0x98a877bb507f19eb43130b688f522a13885cf604#code) : A Comprehensive [Hack Analysis](https://blog.solidityscan.com/orion-protocol-hack-analysis-missing-reentrancy-protection-f9af6995acb3)
