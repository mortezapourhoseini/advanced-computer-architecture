# معماری کامپیوتر پیشرفته — پاسخ دو سؤال اول

> **منبع اسلایدها:** COD Ch.1 (Computer Abstractions and Technology) و COD Ch.2 (The Role of Performance)، برگرفته از کتاب Patterson & Hennessy.
> هر دو سؤال دقیقاً به **فصل «کارایی» (Performance)** مربوط‌اند: معادله‌ی کارایی، CPI، MIPS، بنچمارک‌ها، SPEC و قانون Amdahl.

**برداشت من از سؤال‌ها** (اگر استاد منظور دیگری داشت بگو تا اصلاح کنم):

1. **Debug در برابر Release در Visual Studio:** هدف استاد این است که ببینی **یک برنامه‌ی واحد روی یک سخت‌افزار واحد**، فقط با عوض شدن تنظیمات کامپایلر، چقدر سرعتش فرق می‌کند، و این تفاوت را با معادله‌ی `CPU time = IC × CPI × T` توضیح بدهی. این همان جمله‌ی اسلاید Summary است: *«compiler enhancements that lower CPI and/or instruction count»*.
2. **بنچمارک گوشی‌های هوشمند:** یک تحقیق که توضیح بدهد گوشی‌ها با چه بنچمارک‌هایی سنجیده می‌شوند، هر کدام چه چیزی را اندازه می‌گیرند، چطور گزارش می‌شوند و چطور «دور زده» می‌شوند. این نسخه‌ی موبایلیِ بحث SPEC در اسلایدهاست.

---

## فهرست

- [بخش ۱: Debug در برابر Release](#بخش-۱-debug-در-برابر-release)
  - [۱.۱ اصلاً Build Configuration چیست؟](#۱۱-اصلاً-build-configuration-چیست)
  - [۱.۲ جدول تفاوت تنظیمات در Visual Studio (C/C++)](#۱۲-جدول-تفاوت-تنظیمات-در-visual-studio-cc)
  - [۱.۳ ارتباط با درس: معادله‌ی کارایی](#۱۳-ارتباط-با-درس-معادلهی-کارایی)
  - [۱.۴ آزمایش واقعی: همان کد، دو کامپایل](#۱۴-آزمایش-واقعی-همان-کد-دو-کامپایل)
  - [۱.۵ تله‌ی MIPS](#۱۵-تلهی-mips)
  - [۱.۶ نکات عملی و باگ‌های «فقط در Release»](#۱۶-نکات-عملی-و-باگهای-فقط-در-release)
  - [۱.۷ اگر پروژه C# بود](#۱۷-اگر-پروژه-c-بود)
  - [۱.۸ خودت تکرارش کن](#۱۸-خودت-تکرارش-کن)
- [بخش ۲: تحقیق — بنچمارک‌های گوشی هوشمند](#بخش-۲-تحقیق--بنچمارکهای-گوشی-هوشمند)
  - [۲.۱ چرا بنچمارک گوشی سخت‌تر از PC است؟](#۲۱-چرا-بنچمارک-گوشی-سختتر-از-pc-است)
  - [۲.۲ دسته‌بندی بنچمارک‌ها](#۲۲-دستهبندی-بنچمارکها)
  - [۲.۳ معرفی بنچمارک‌های اصلی](#۲۳-معرفی-بنچمارکهای-اصلی)
  - [۲.۴ امتیاز چطور ساخته می‌شود؟ (میانگین هندسی)](#۲۴-امتیاز-چطور-ساخته-میشود-میانگین-هندسی)
  - [۲.۵ تقلب در بنچمارک](#۲۵-تقلب-در-بنچمارک)
  - [۲.۶ ارتباط با مفاهیم درس](#۲۶-ارتباط-با-مفاهیم-درس)
  - [۲.۷ روش درست بنچمارک گرفتن از گوشی](#۲۷-روش-درست-بنچمارک-گرفتن-از-گوشی)
  - [۲.۸ جمع‌بندی تحقیق](#۲۸-جمعبندی-تحقیق)
- [منابع](#منابع)

---

## بخش ۱: Debug در برابر Release

### ۱.۱ اصلاً Build Configuration چیست؟

وقتی در Visual Studio پروژه می‌سازی، دو «پیکربندی ساخت» به صورت پیش‌فرض ساخته می‌شود. **کد منبع یکی است**؛ فقط تنظیمات کامپایلر و لینکر فرق دارد:

| | **Debug** | **Release** |
|---|---|---|
| هدف | پیدا کردن باگ هنگام توسعه | تحویل نهایی به کاربر و **اندازه‌گیری کارایی** |
| بهینه‌سازی | **خاموش** | **روشن** (حداکثر سرعت) |
| اطلاعات دیباگ | کامل؛ هر خط کد ↔ دستورات ماشین | معمولاً فقط فایل `.pdb` جدا |
| بررسی‌های زمان اجرا | زیاد (پشته، متغیرهای مقداردهی‌نشده، heap) | تقریباً هیچ |
| `assert()` | فعال | حذف می‌شود (`NDEBUG`) |
| سرعت اجرا | کند (گاهی ۱۰ تا ۵۰ برابر) | سریع |
| حجم فایل | بزرگ‌تر | کوچک‌تر |

**چرا در Debug بهینه‌سازی خاموش است؟** چون بهینه‌سازی رابطه‌ی «خط کد ↔ دستور ماشین» را به هم می‌ریزد. کامپایلر متغیرها را در رجیستر نگه می‌دارد یا حذف می‌کند، حلقه‌ها را باز می‌کند و ترتیب دستورات را عوض می‌کند. در نتیجه وقتی breakpoint می‌گذاری، دیباگر نمی‌تواند مقدار متغیر را نشان بدهد یا خط به خط جلو برود.

### ۱.۲ جدول تفاوت تنظیمات در Visual Studio (C/C++)

مسیر: `Project → Properties → Configuration Properties → C/C++` و `Linker`

| تنظیم | Debug (پیش‌فرض) | Release (پیش‌فرض) | معنی |
|---|---|---|---|
| Optimization | `/Od` | `/O2` | `/Od` یعنی بدون بهینه‌سازی؛ `/O2` یعنی حداکثر سرعت |
| Debug Information | `/ZI` (Edit & Continue) | `/Zi` | تولید اطلاعات دیباگ |
| Basic Runtime Checks | `/RTC1` | — | چک کردن خرابی پشته و متغیر مقداردهی‌نشده در زمان اجرا |
| Runtime Library | `/MDd` (نسخه‌ی دیباگ CRT) | `/MD` | کتابخانه‌ی دیباگ، heap را با الگوهای خاص پر می‌کند و چک می‌کند |
| Preprocessor | `_DEBUG` | `NDEBUG` | `assert` در Release حذف می‌شود |
| Intrinsic Functions | — | `/Oi` | جایگزینی توابعی مثل `memcpy` با دستورات مستقیم |
| Whole Program Opt. | — | `/GL` + لینکر `/LTCG` | بهینه‌سازی بین فایل‌ها (inline کردن توابع فایل‌های دیگر) |
| Function-level linking | — | `/Gy` + `/OPT:REF` و `/OPT:ICF` | حذف کد بلااستفاده و ادغام توابع یکسان |
| Just My Code | `/JMC` | — | کمک به دیباگر |
| Incremental Linking | بله | خیر | لینک سریع‌تر هنگام توسعه |

### ۱.۳ ارتباط با درس: معادله‌ی کارایی

از اسلاید **Performance Equation II**:

```
CPU time = Instruction Count (IC) × CPI × Clock cycle time (T)
```

وقتی Debug و Release را **روی یک کامپیوتر** اجرا می‌کنی:

| عامل | چه اتفاقی می‌افتد؟ | چرا؟ |
|---|---|---|
| **T** (زمان سیکل) | **ثابت** | سخت‌افزار عوض نشده |
| **IC** (تعداد دستورات) | در Release **خیلی کمتر** | حذف load/store اضافه، inline کردن، برداری‌سازی (SIMD)، حذف کد مرده، محاسبه‌ی ثابت‌ها هنگام کامپایل |
| **CPI** | در Release معمولاً **کمتر** | در Debug، متغیرها مدام در حافظه نوشته و دوباره خوانده می‌شوند. این یک **زنجیره‌ی وابستگی از طریق حافظه** می‌سازد و پردازنده باید منتظر بماند |

**نتیجه‌ی مهم:** کامپایلر هم IC را تغییر می‌دهد و هم CPI را. این دقیقاً موضوع اسلایدهای *CPI Example II* و *MIPS Example* است که در آن‌ها دو کامپایلر کد متفاوتی برای یک برنامه تولید می‌کنند.

**بهینه‌سازی‌های اصلی Release، به زبان ساده:**

- **Register allocation:** متغیرها به جای حافظه در رجیستر می‌مانند. دسترسی به رجیستر حدود ۱ سیکل است، ولی load/store از حافظه چند سیکل طول می‌کشد. این همان جمله‌ی اسلاید است: *«Accessing memory takes more time than accessing registers»*.
- **Inlining:** فراخوانی تابع (call/ret، ذخیره‌ی رجیسترها، کار با پشته) حذف می‌شود و بدنه‌ی تابع مستقیم جای فراخوانی قرار می‌گیرد.
- **Vectorization (SIMD):** یک دستور `paddd` چهار عدد صحیح را با هم جمع می‌کند. پس IC حدود ۴ برابر کم می‌شود.
- **Loop optimizations:** باز کردن حلقه (unrolling) و بیرون بردن محاسبات تکراری از حلقه.
- **Constant folding / Dead code elimination:** محاسبه‌ی عبارت‌های ثابت هنگام کامپایل و حذف کدی که نتیجه‌اش استفاده نمی‌شود.

### ۱.۴ آزمایش واقعی: همان کد، دو کامپایل

من این آزمایش را خودم اجرا کردم. چون Visual Studio در لینوکس نیست، از `gcc` استفاده کردم. معادلِ Debug می‌شود `-O0` و معادلِ Release می‌شود `-O2`؛ مفهوم دقیقاً همان `/Od` در برابر `/O2` است.

```c
int sum(const int *a, int n) {
    int s = 0;
    for (int i = 0; i < n; i++) s += a[i];
    return s;
}
// main: آرایه‌ای با ۱۰ میلیون عدد، ۱۰۰ بار جمع زده می‌شود
```

**کد اسمبلی x86-64 حلقه در حالت Debug (`-O0`):**

```asm
loop:
  mov  eax, DWORD PTR [rbp-0x4]    ; i را از حافظه بخوان
  cdqe
  lea  rdx, [rax*4]
  mov  rax, QWORD PTR [rbp-0x18]   ; آدرس a را از حافظه بخوان
  add  rax, rdx
  mov  eax, DWORD PTR [rax]        ; a[i]
  add  DWORD PTR [rbp-0x8], eax    ; s += a[i]  ← خواندن و نوشتن s در حافظه
  add  DWORD PTR [rbp-0x4], 0x1    ; i++        ← خواندن و نوشتن i در حافظه
  mov  eax, DWORD PTR [rbp-0x4]    ; دوباره i را بخوان
  cmp  eax, DWORD PTR [rbp-0x1c]   ; n را هم از حافظه بخوان
  jl   loop
```

→ **۱۱ دستور برای هر عنصر**، که بیشترشان با حافظه کار دارند.

**در حالت Release (`-O2`)، کامپایلر تابع را داخل `main` آورد (inline) و برداری کرد:**

```asm
loop:
  movdqu xmm4, XMMWORD PTR [rax]   ; ۴ عدد را با هم بخوان
  add    rax, 0x10
  paddd  xmm0, xmm4                ; ۴ جمع با یک دستور
  cmp    rbx, rax
  jne    loop
```

→ **۵ دستور برای هر ۴ عنصر، یعنی ۱٫۲۵ دستور برای هر عنصر**. متغیر `s` در رجیستر `xmm0` می‌ماند.

**نتایج اندازه‌گیری‌شده** (پردازنده‌ی Intel Xeon با کلاک ۲٫۱ GHz، GCC 13.3؛ تعداد دستورات با `valgrind --tool=cachegrind` شمرده شده):

| | Debug (`-O0`) | Release (`-O2`) | نسبت |
|---|---|---|---|
| تعداد دستورات برای یک بار جمع ۱۰ میلیون عنصر | حدود ۱۱۰ میلیون | حدود ۱۲٫۵ میلیون | **IC حدود ۸٫۸ برابر کمتر** |
| زمان ۱۰۰ بار تکرار (سه اجرا) | ۲٫۲۶ تا ۲٫۳۱ ثانیه | ۰٫۱۱ تا ۰٫۱۹ ثانیه | **حدود ۱۲ تا ۲۱ برابر سریع‌تر** |

**تحلیل با فرمول درس:** چون T ثابت است:

```
Time_Debug / Time_Release = (IC_D / IC_R) × (CPI_D / CPI_R)
       ≈ 15            =      8.8      ×   CPI_D / CPI_R
⇒  CPI_D / CPI_R ≈ 1.4 تا 2.4
```

یعنی **فقط کم شدن دستورات، کل اختلاف را توضیح نمی‌دهد**. در Debug، CPI هم بدتر است. دلیلش این است که `s` و `i` در هر تکرار در حافظه نوشته و بلافاصله دوباره خوانده می‌شوند (store→load). هر تکرار باید منتظر تمام شدن تکرار قبلی بماند. اسم این وابستگی در فصل pipeline می‌شود *data hazard*.

> **نکته‌ی جانبی برای روش‌شناسی:** زمان Release در سه اجرا بین ۰٫۱۱ و ۰٫۱۹ ثانیه نوسان داشت (به خاطر ماشین مجازی، پهنای باند حافظه و فرکانس متغیر). برای همین **باید چند بار اندازه بگیری و میانه یا میانگین را گزارش کنی**. این نکته را در بخش ۲ هم لازم داریم.

### ۱.۵ تله‌ی MIPS

از اسلاید *MIPS Example* و *Pitfall*:

```
MIPS = IC / (Execution time × 10^6)
```

برنامه‌ی Debug تعداد خیلی زیادی دستور ساده اجرا می‌کند. پس **ممکن است MIPS آن بالا باشد، با اینکه برنامه کندتر است**. در آزمایش ما:

- Debug: حدود ۱۱۰ میلیون دستور در ۰٫۰۲۳ ثانیه (برای هر بار تکرار) ≈ **۴۸۰۰ MIPS**
- Release: حدود ۱۲٫۵ میلیون دستور در حدود ۰٫۰۰۱۵ ثانیه ≈ **۸۳۰۰ MIPS**

در این مثال هنوز Release برنده است، ولی اختلاف MIPS (حدود ۱٫۷ برابر) خیلی کمتر از اختلاف واقعی سرعت (حدود ۱۵ برابر) است. اگر بهینه‌سازی فقط IC را کم کرده بود و CPI را دست نزده بود، MIPS دو حالت تقریباً یکسان درمی‌آمد، در حالی که یکی چند برابر سریع‌تر بود.
→ **MIPS معیار کارایی نیست؛ تنها معیار معتبر «زمان اجرا»ست.** (همان جمله‌ی اسلاید: *Performance is determined by execution time*.)

### ۱.۶ نکات عملی و باگ‌های «فقط در Release»

1. **هیچ‌وقت برنامه‌ی Debug را بنچمارک نکن.** اعداد بی‌معنی می‌شوند. SPEC هم برای همین الزام می‌کند **فلگ‌های کامپایلر گزارش شوند** (اسلاید *SPEC CPU2000 reporting: Report compiler flag setting*).
2. **باگ‌هایی که فقط در Release ظاهر می‌شوند:**
   - **متغیر مقداردهی‌نشده:** در Debug، حافظه با الگوهای ثابت (مثل `0xCC`) پر می‌شود و `/RTC1` خطا می‌دهد. در Release مقدار تصادفی است.
   - **Undefined Behavior** (مثل خواندن خارج از آرایه یا سرریز عدد صحیح علامت‌دار): بهینه‌ساز فرض می‌کند UB اتفاق نمی‌افتد و ممکن است کد را عوض کند.
   - **کد داخل `assert(...)`:** اگر داخل assert کار مفیدی انجام شود (مثلاً `assert(init()==0)`)، در Release اصلاً اجرا نمی‌شود.
   - **Race condition در برنامه‌های چندنخی:** در Debug به خاطر کندی دیده نمی‌شود.
3. **دیباگ کردن Release:** فایل `.pdb` را نگه دار. MSVC گزینه‌ی `/Zo` را برای دیباگ بهتر کد بهینه‌شده دارد.
4. **حالت سوم:** خیلی از پروژه‌ها یک پیکربندی `RelWithDebInfo` دارند (بهینه‌سازی روشن + اطلاعات دیباگ) که برای **profiling** بهترین است.

### ۱.۷ اگر پروژه C# بود

در C#/.NET، کامپایلر C# کد میانی (IL) می‌سازد و **کامپایلر JIT** در زمان اجرا آن را به کد ماشین تبدیل می‌کند. در Debug، فلگ optimize خاموش است و JIT هم بهینه‌سازی را محدود می‌کند (مثلاً متغیرها زنده نگه داشته می‌شوند تا در دیباگر دیده شوند). اگر برنامه زیر دیباگر اجرا شود، حتی Release هم ممکن است کامل بهینه نشود. برای بنچمارک در .NET از **BenchmarkDotNet** استفاده می‌شود که اجرای Debug را رد می‌کند.

### ۱.۸ خودت تکرارش کن

**روی Linux Mint خودت** (معادل دقیق Debug/Release):

```bash
gcc -O0 -g -o sum_debug   sum.c     # ≈ Debug
gcc -O2 -DNDEBUG -o sum_rel sum.c   # ≈ Release
./sum_debug ; ./sum_rel

# شمردن دستورات و سیکل‌ها و محاسبه‌ی مستقیم IPC = 1/CPI:
sudo apt install linux-tools-common linux-tools-$(uname -r)
perf stat -e instructions,cycles ./sum_debug
perf stat -e instructions,cycles ./sum_rel
# مقایسه‌ی کد اسمبلی:
objdump -d -M intel sum_rel | less
```

`perf stat` مستقیماً `instructions` (یعنی IC) و `cycles` را می‌دهد. پس **CPI = cycles / instructions** را خودت حساب می‌کنی و هر سه جزء فرمول را روی کامپیوتر خودت می‌بینی. این بهترین تمرین ممکن برای این فصل است.

**در Visual Studio (ویندوز):** از نوار ابزار `Debug` یا `Release` را انتخاب کن. با `Ctrl+F5` اجرا کن (بدون دیباگر). برای دیدن کد اسمبلی، روی یک breakpoint از مسیر `Debug → Windows → Disassembly` استفاده کن.

---

## بخش ۲: تحقیق — بنچمارک‌های گوشی هوشمند

### ۲.۱ چرا بنچمارک گوشی سخت‌تر از PC است؟

گوشی یک **SoC** (System on Chip) است که چند واحد پردازشی ناهمگون دارد:

- **CPU** با هسته‌های ناهمگون (big.LITTLE / DynamIQ): هسته‌های پرقدرت و کم‌مصرف
- **GPU** برای گرافیک و محاسبات
- **NPU** برای هوش مصنوعی، **DSP** و **ISP** (پردازش تصویر دوربین)
- حافظه‌ی **LPDDR** مشترک بین همه

چالش‌های مخصوص موبایل:

1. **Thermal throttling:** گوشی فن ندارد. بعد از چند دقیقه کار سنگین داغ می‌شود و فرکانس را پایین می‌آورد. پس **کارایی لحظه‌ای (peak) با کارایی پایدار (sustained) فرق دارد**.
2. **باتری و مصرف انرژی:** معیار مهم فقط سرعت نیست، بلکه **کارایی به ازای هر وات** (perf/W) هم هست.
3. **نرم‌افزار:** نسخه‌ی اندروید/iOS، درایور GPU و فریم‌ورک‌های AI روی نتیجه اثر زیادی دارند.
4. **تقلب سازنده‌ها:** بخش ۲.۵ را ببین.

### ۲.۲ دسته‌بندی بنچمارک‌ها

همان دسته‌بندی اسلاید درس، در دنیای موبایل:

| نوع (طبق اسلاید) | معادل موبایل |
|---|---|
| Real applications | تست واقعی: زمان باز شدن اپ، FPS در بازی‌هایی مثل Genshin Impact، عمر باتری |
| Small benchmarks / kernels (مثل Livermore Loops) | تست‌های کوچک CPU، مثل CoreMark |
| Benchmark suites (مثل SPEC) | **Geekbench، AnTuTu، 3DMark، MLPerf Mobile** |
| Synthetic / Composite | AnTuTu (یک عدد کلی از ترکیب CPU+GPU+Memory+UX) |

### ۲.۳ معرفی بنچمارک‌های اصلی

#### الف) Geekbench (شرکت Primate Labs) — استاندارد مقایسه‌ی CPU

- **Geekbench 7** در **۲۳ ژوئیه‌ی ۲۰۲۶** منتشر شد، حدود سه سال بعد از Geekbench 6. برای Android، iOS، Windows، macOS و Linux موجود است و برای استفاده‌ی شخصی رایگان است.
- **مهم‌ترین تغییرش، بازطراحی تست multi-core بود:** کارها فقط وقتی بین چند هسته پخش می‌شوند که **برنامه‌ی واقعی متناظر هم واقعاً چندنخی باشد**. مثلاً تست مرورگر HTML5 در بخش multi-core نیست، چون مرورگرها بیشتر به کارایی تک‌نخی وابسته‌اند. هدف این است که امتیاز، پردازنده‌ای را که فقط هسته‌ی بیشتری دارد بی‌دلیل بالا نبرد. این **عملاً همان قانون Amdahl است** (بخش ۲.۶).
- تست‌های جدید آن: کدگذاری ویدیو AV1، صدای Opus، زیرنویس خودکار با مدل Whisper، فیزیک بازی و ویرایش عکس پیشرفته. GPU benchmark هم بیشتر روی یادگیری ماشین و تولید محتوا تمرکز دارد. CUDA هم به APIهای OpenCL، Vulkan و Metal اضافه شده است.
- **امتیاز CPU نسبت به سیستم مرجعی با امتیاز ۲۵۰۰ نرمال می‌شود.** امتیاز دو برابر یعنی (در تئوری) کارایی دو برابر. **امتیاز Geekbench 7 با Geekbench 6 قابل مقایسه نیست.**
- نمونه‌ی امتیازهای Geekbench 7 گوشی‌ها (Notebookcheck، اوت تا اکتبر ۲۰۲۶):
  - **Single-core:** iPhone Air (۳۲۱۱)، iPhone 17 Pro (۳۱۷۵)، Galaxy S26 Ultra (۳۱۴۲)، Honor Magic8 Pro (۳۱۴۰)
  - **Multi-core:** Honor Magic8 Pro (۱۰۹۵۳)، Galaxy S26 Ultra (۱۰۳۹۸)، Vivo X300 Ultra (۹۳۰۰)، iPhone 17 Pro (۹۰۰۳)
- **Geekbench 6** (۲۰۲۳) هنوز در خیلی از مقاله‌ها و نشت‌های اطلاعاتی استفاده می‌شود. مثلاً Galaxy S26 Ultra با Snapdragon 8 Elite Gen 5 امتیاز single-core ۳۸۵۲ و multi-core ۱۱۷۳۸ گرفت. در GB6 تست رمزنگاری (crypto) حذف شد، چون به پردازنده‌هایی که شتاب‌دهنده‌ی سخت‌افزاری AES داشتند امتیاز نامتناسب می‌داد.
- **Geekbench AI** (نسخه‌ی ۱٫۰ از اوت ۲۰۲۴): CPU، GPU و **NPU** را با کارهای واقعی یادگیری ماشین می‌سنجد. می‌شود فریم‌ورک را هم انتخاب کرد (Core ML، QNN، OpenVINO و...). علاوه بر سرعت، **دقت خروجی** را هم اندازه می‌گیرد، چون می‌شود با کوانتیزه کردن شدید مدل سرعت را بالا برد و دقت را خراب کرد.

#### ب) AnTuTu — محبوب‌ترین بنچمارک ترکیبی در بازار اندروید

- یک **عدد کلی** از چهار بخش می‌سازد: **CPU، GPU، Memory، UX** (تجربه‌ی کاربری).
- **نسخه‌ی فعلی v11** است. **امتیاز v10 و v11 قابل مقایسه نیستند.**
- رتبه‌بندی ماهانه منتشر می‌کند (میانگین اجراهای کاربران، عمدتاً در چین). در دسامبر ۲۰۲۵، RedMagic 11 Pro+ با Snapdragon 8 Elite Gen 5 حدود **۴٬۱۱۸٬۸۲۸** امتیاز گرفت. در جدول nanoreview، صدر جدول مال iQOO 15 Ultra با حدود **۴٬۶۰۲٬۲۹۰** است. Dimensity 9500 مدیاتک هم نزدیک به همین سطح است (حدود ۴٫۰ میلیون).
- AnTuTu روی iOS هم نسخه‌ی v11 دارد.
- **انتقاد علمی:** وزن‌دهی بخش‌ها شفاف نیست و یک عدد کلی، نقاط ضعف را پنهان می‌کند. این دقیقاً هشدار اسلاید است: *«see through the marketing hype»*.

#### ج) 3DMark (شرکت UL Solutions) — استاندارد گرافیک و پایداری

| تست | چه می‌سنجد |
|---|---|
| **Wild Life / Wild Life Extreme** | گرافیک سبک بدون ray tracing، چندسکویی (Android، iOS، Windows، Mac) |
| **Steel Nomad Light** | سنگین‌ترین تست بدون ray tracing برای دستگاه‌های سبک؛ رزولوشن ۱۴۴۰p؛ حداقل ۸ گیگ RAM |
| **Solar Bay** | ray tracing سبک (Vulkan روی اندروید، Metal روی iOS) |
| **Solar Bay Extreme** | ray tracing سنگین‌تر (از نوامبر ۲۰۲۵) |
| **Stress Test** | **۲۰ بار تکرار** تست؛ نشان می‌دهد گوشی زیر بار طولانی چقدر افت می‌کند (**stability %**) |

**Stress Test مهم‌ترین تست برای فهم throttling است.** نسبت بدترین دور به بهترین دور، پایداری حرارتی را نشان می‌دهد.

#### د) MLPerf Mobile (کنسرسیوم MLCommons) — استاندارد صنعتی AI روی دستگاه

- **اولین بنچمارک متن‌باز و استاندارد صنعتی** برای سنجش کارایی **و دقت** هوش مصنوعی روی گوشی. سازندگان SoC، فریم‌ورک‌ها و دانشگاه‌ها با هم آن را ساخته‌اند. مقاله‌اش در **MLSys 2022** چاپ شده است.
- یک اپ اندروید مشترک دارد با مدل‌ها، داده‌ها، معیار کیفیت و **قوانین اجرای (run rules) ثابت**. هر سازنده یک **backend** برای سخت‌افزار خودش (NPU، DSP، GPU) می‌نویسد.
- تسک‌ها: Image Classification، Object Detection، Semantic Segmentation، Question Answering (و در نسخه‌های جدید تسک‌های بیشتر).
- **برای تو (Edge AI و بینایی ماشین) مهم‌ترین بنچمارک است**، چون منطق MLPerf (latency در برابر throughput، کوانتیزه‌سازی INT8 در برابر FP16، تأثیر software stack) مستقیماً به کار و مقاله‌هایت مربوط است. طبق مقاله، در دو دور اول فقط در شش ماه throughput حدود ۳ برابر و latency تا ۱۲ برابر بهتر شد، **فقط با بهبود نرم‌افزار و کامپایلر**. این دقیقاً نسخه‌ی AI بحث Debug/Release است.

#### هـ) بنچمارک‌های دیگر

| بنچمارک | کاربرد |
|---|---|
| **GFXBench** | تست‌های گرافیکی قدیمی‌تر؛ گزارش FPS به صورت on-screen و off-screen |
| **PCMark for Android (Work 3.0)** | کارهای روزمره (وب، ویرایش عکس و متن) + **تست عمر باتری** |
| **AI Benchmark (ETH Zürich)** | سنجش AI روی اندروید، رایج در مقاله‌های آکادمیک |
| **SPEC CPU** | بنچمارک **آکادمیک** CPU (همان SPEC اسلایدها). **SPEC CPU 2026** در سال ۲۰۲۶ منتشر شد و جای SPEC CPU 2017 را گرفت (۵۲ بنچمارک به جای ۴۳). برای دانشگاه‌های معتبر رایگان است. سایت‌های بررسی تخصصی گاهی SPEC را روی گوشی کامپایل و اجرا می‌کنند تا IPC و perf/W هسته‌ها را مقایسه کنند |

### ۲.۴ امتیاز چطور ساخته می‌شود؟ (میانگین هندسی)

از اسلاید *SPEC CPU2000 reporting: «Single number result – geometric mean of normalized ratios»*. Geekbench هم همین روش را دارد:

1. هر تست روی **ماشین مرجع** اجرا می‌شود: `Time_ref`
2. روی دستگاه تو اجرا می‌شود: `Time_dev`
3. نسبت نرمال‌شده: `ratio_i = Time_ref / Time_dev`
4. امتیاز کل = **میانگین هندسی**: `(ratio_1 × ratio_2 × … × ratio_n)^(1/n)`

**چرا هندسی و نه حسابی؟** مثال با دو تست:

| | تست ۱ | تست ۲ | میانگین حسابی | میانگین هندسی |
|---|---|---|---|---|
| گوشی A نسبت به مرجع | ۲٫۰ | ۰٫۵ | ۱٫۲۵ | **۱٫۰** |
| گوشی B نسبت به مرجع | ۰٫۵ | ۲٫۰ | ۱٫۲۵ | **۱٫۰** |

ویژگی کلیدی: **نسبت میانگین هندسی دو دستگاه به انتخاب ماشین مرجع بستگی ندارد**:

```
GM(A)/GM(B) = GM(Time_B / Time_A)
```

`Time_ref` از صورت و مخرج حذف می‌شود. با میانگین حسابی، عوض کردن ماشین مرجع می‌تواند ترتیب برنده‌ها را عوض کند.
⚠️ **عیبش:** میانگین هندسی با «زمان کل اجرا» متناسب نیست. اسلاید Summary می‌گوید *«total execution time is a consistent summary»*. این بحث کلاسیک SPEC را حتماً یاد بگیر.

### ۲.۵ تقلب در بنچمارک

اسلاید SPEC می‌گوید: *«Can still be abused…, by selective optimization!»*. در دنیای گوشی این اتفاق بارها افتاده است:

| سال | سازنده | چه کرد | نتیجه |
|---|---|---|---|
| ۲۰۲۱ | **OnePlus 9 / 9 Pro** | اپ‌های معروف (مثل کروم) **بر اساس نام اپ** کند می‌شدند، ولی بنچمارک‌ها نه | Geekbench هر دو را حذف (delist) کرد |
| ۲۰۲۲ | **Samsung (سرویس GOS)** | GOS حدود ۱۰ هزار اپ و بازی را throttle می‌کرد ولی **بنچمارک‌ها را استثنا کرده بود** | Geekbench چهار نسل Galaxy S (S10، S20، S21، S22) و بعداً Tab S8 را حذف کرد. طبق سیاستش، دستگاه حذف‌شده هرگز برنمی‌گردد |
| قبل‌تر | Huawei، Xiaomi، Meizu و... | موارد مشابه | حذف از جدول‌ها |
| **آوریل ۲۰۲۶** | **REDMAGIC 11 Pro / Pro+** | UL Solutions دید امتیاز اپ عمومی 3DMark تا **۲۴٪ بیشتر** از نسخه‌ی تغییرنام‌داده‌ی همان اپ است. گوشی اپ را **از روی نامش** تشخیص می‌داد و حالت پرقدرت را روشن می‌کرد، طوری که خاموش‌شدنی هم نبود | حذف از رتبه‌بندی 3DMark |

**الگوی مشترک:** تصمیم بر اساس **شناسه‌ی اپ** گرفته می‌شود، نه **رفتار اپ**. در نتیجه بنچمارک چیزی را می‌سنجد که کاربر واقعی هیچ‌وقت تجربه نمی‌کند.
**روش کشف تقلب:** اجرای نسخه‌ی **تغییرنام‌داده‌ی (stealth)** بنچمارک و مقایسه با نسخه‌ی عمومی. این از نظر علمی یک **آزمایش کنترل‌شده** است.

### ۲.۶ ارتباط با مفاهیم درس

| مفهوم درس | در بنچمارک گوشی |
|---|---|
| `Performance = 1 / Execution time` | امتیاز Geekbench با معکوس زمان اجرای تست‌ها متناسب است |
| `Time = IC × CPI × T` | امتیاز single-core ≈ **IPC × فرکانس**. دو گوشی با امتیاز single-core برابر (مثل A19 Pro و Snapdragon 8 Elite Gen 5 در GB6) ممکن است یکی IPC بالاتر و دیگری فرکانس بالاتر داشته باشد. **تمرین:** فرکانس هسته‌ی اصلی هر دو را پیدا کن و IPC نسبی را حساب کن |
| **قانون Amdahl** | چرا multi-core گوشی ۸ هسته‌ای ۸ برابر single-core نیست؟ چون بخشی از کار سریال است. تغییر multi-core در Geekbench 7 دقیقاً می‌خواهد این واقعیت را در امتیاز نشان بدهد |
| «Make the common case fast» | NPU و ISP برای کارهای رایج (عکاسی، AI) ساخته شده‌اند |
| Response time در برابر Throughput | در MLPerf: سناریوی **Single-stream** (latency) در برابر **Offline** (throughput) |
| نقش کامپایلر (سؤال ۱) | بهبود ۳ تا ۱۲ برابری MLPerf Mobile فقط با نرم‌افزار؛ اهمیت درایور و فریم‌ورک |
| Pitfall: یک عدد ≠ کارایی | امتیاز کلی AnTuTu؛ کارایی لحظه‌ای در برابر پایدار |
| انتخاب workload نماینده | Geekbench از کارهای واقعی (PDF، عکس، مرورگر، Whisper) استفاده می‌کند، نه کارهای مصنوعی |

### ۲.۷ روش درست بنچمارک گرفتن از گوشی

چک‌لیست (اگر استاد پروژه‌ی عملی خواست):

1. شارژ بالای ۸۰٪ و ترجیحاً **بدون شارژر** (شارژ کردن گرما تولید می‌کند)
2. دمای اتاق ثابت و حدود ۲۵ درجه؛ گوشی قبل از تست خنک باشد
3. حالت پرواز، بستن همه‌ی اپ‌ها، روشنایی صفحه‌ی ثابت
4. **چند بار اجرا** (مثلاً ۵ بار) و گزارش **میانه + انحراف معیار**. در آزمایش بخش ۱ دیدیم اعداد تا ۴۰٪ نوسان دارند
5. گزارش کامل **پیکربندی**: مدل، SoC، RAM، نسخه‌ی OS، نسخه‌ی بنچمارک. همان چیزی که SPEC الزام می‌کند (*Report precise description of machine*)
6. جدا کردن **peak** (یک اجرا) از **sustained** (Stress Test با ۲۰ بار تکرار)
7. در صورت امکان **مصرف انرژی** (perf/W) هم اندازه گرفته شود
8. فقط **نسخه‌های یکسان** بنچمارک با هم مقایسه شوند (GB6 ≠ GB7، AnTuTu v10 ≠ v11)

### ۲.۸ جمع‌بندی تحقیق

- **برای CPU:** Geekbench 7 (و برای مقایسه با منابع قدیمی‌تر، GB6). **برای GPU و پایداری حرارتی:** 3DMark (Steel Nomad Light، Solar Bay، Stress Test). **برای AI:** MLPerf Mobile و Geekbench AI. **برای عدد کلی بازاری:** AnTuTu v11، با احتیاط.
- **هیچ بنچمارکی به تنهایی کافی نیست.** کارایی به برنامه بستگی دارد. همان جمله‌ی اسلاید: *«Performance is specific to a particular program»*.
- بنچمارک‌ها مدام عوض می‌شوند (Geekbench 7 و SPEC CPU 2026 هر دو در ۲۰۲۶ آمدند)، چون workloadها عوض می‌شوند. این همان چیزی است که اسلاید *SPEC History* نشان می‌دهد.
- تقلب هنوز ادامه دارد (۲۰۲۶ هم دیدیم). پس اعداد را **نقادانه** بخوان.

---

## منابع

**Debug / Release**
- [Microsoft Learn — Set debug and release configurations](https://learn.microsoft.com/visualstudio/debugger/how-to-set-debug-and-release-configurations)
- [Microsoft Learn — Debug optimized code](https://learn.microsoft.com/visualstudio/debugger/how-to-debug-optimized-code)
- [Microsoft Learn — MSVC compiler options listed alphabetically](https://learn.microsoft.com/en-us/cpp/build/reference/compiler-options-listed-alphabetically)
- [Microsoft Learn (VS 2008) — How to: Set Debug and Release Configurations](https://learn.microsoft.com/sv-se/previous-versions/visualstudio/visual-studio-2008/wx0123s5(v=vs.90))
- [LearnCpp — Build configurations](https://www.learncpp.com/?p=1232)
- آزمایش عملی بخش ۱.۴: اجرای خودم با GCC 13.3 و Valgrind Cachegrind

**بنچمارک گوشی**
- [MacRumors — Geekbench 7 launches](https://macrumors.com/2026/07/23/geekbench-7-launch)
- [Notebookcheck — Geekbench 7 launches with CUDA support and a major multi-core overhaul](https://notebookcheck.net/Geekbench-7-launches-with-CUDA-support-and-a-major-multi-core-overhaul.1349869.0.html)
- [Macworld — Geekbench 7 is out](https://www.macworld.com/article/3198338/geekbench-7-is-out-now-with-new-modern-workloads-and-better-multi-core-tests.html)
- [Notebookcheck — New Geekbench: This is the fastest smartphone](https://www.notebookcheck.net/New-Geekbench-This-is-the-fastest-smartphone.1374777.0.html)
- [Notebookcheck — Galaxy S26 Ultra in Geekbench 6](https://www.notebookcheck.net/Benchmark-Galaxy-S26-Ultra-crushes-iPhone-17-Pro-harder-than-ever-in-Geekbench-6.1231556.0.html)
- [TechPowerUp — Geekbench AI 1.0](https://techpowerup.com/325619/geekbench-ai-hits-1-0-release-cpus-gpus-and-npus-finally-get-ai-benchmarking-solution)
- [The Tech Outlook — AnTuTu V11 rankings, December 2025](https://www.thetechoutlook.com/news/antutu-performance-rankings-for-december-2025-shared-redmagic-11-pro-oppo-reno15-pro-and-honor-magicpad-3-pro-13-3-lead-their-segments)
- [nanoreview — AnTuTu Benchmark 11 Rating](https://nanoreview.net/en/phone-list/antutu-rating)
- [UL Solutions — 3DMark](https://benchmarks.ul.com/3dmark) و [3DMark for iOS](https://benchmarks.ul.com/3dmark-ios)
- [UL Solutions — 3DMark delisting of REDMAGIC 11 Pro phones](https://benchmarks.ul.com/de/news/3dmark-delisting-of-redmagic-11-pro-phones)
- [Reddi et al., MLPerf Mobile Inference Benchmark, MLSys 2022 (arXiv)](https://arxiv.org/abs/2012.02328)
- [Android Authority — Geekbench delists Samsung flagships](https://www.androidauthority.com/geekbench-delists-samsung-flagships-3129230/)
- [TechRadar — OnePlus 9 delisted by Geekbench](https://www.techradar.com/news/oneplus-once-again-caught-trying-to-manipulate-benchmark-scores)
- [SPEC — SPEC CPU 2017 retirement](https://www.spec.org/cpu2017) و [SPEC CPU 2026 Overview](https://ftp.spec.org/cpu2026/docs/overview.html)
- [heise — SPEC CPU 2026](https://heise.de/-11284559)
- اسلایدهای درس: COD Ch.1 و Ch.2 (Patterson & Hennessy، تنظیم Dr. Sumanta Guha)
