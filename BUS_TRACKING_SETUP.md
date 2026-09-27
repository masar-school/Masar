# ميزة التتبع الحي لمشرفة/سائق الباص — دليل التنفيذ

هذا الملف يشرح كل خطوة متبقية عليك لتفعيل ميزة "الخريطة الحية" الجديدة، بعد أن أصبحت
كل ملفات الكود (`Code.gs`، `index.html`، `Masar-staff.html`، `Masar-supervisor.html`)
جاهزة ومُعدَّلة.

**⚠️ نقطة مهمة جداً:** رابط خرائط غوغل القديم (`Live_Location_URL` بورقة `Supervisors`)
**لم يُلغَ ولم يُعدَّل إطلاقاً** — بقي يعمل بالضبط كما كان، كخيار احتياطي. الميزة الجديدة
(الخريطة الحية داخل التطبيق) أُضيفت **بجانبه**، منفصلة تماماً.

---

## 1) تعديلات مطلوبة يدوياً على Google Sheet (قبل أي نشر)

### أ) ورقة `Supervisors` — إضافة عمودين جديدين
أضف بأقصى يمين الصفوف الموجودة (أو أي مكان، المهم يكون بالسطر الأول اسم العمود بالضبط):

| Username | Password |
|---|---|

- هذا حساب دخول **منفصل تماماً** عن حسابات `Teacher_Accounts` — خاص بتطبيق المشرفة فقط.
- عبّي `Username`/`Password` لكل مشرفة إما يدوياً بالشيت، أو من لوحة الإدارة (تبويب مشرفي
  الباص أصبح فيه الآن حقلا اسم مستخدم/كلمة سر لكل مشرفة).

### ب) ورقة جديدة كاملة باسم `Supervisor_Trips`
أنشئ ورقة جديدة بنفس جدول "Masar School AppSheet-Claude"، بهذا الترتيب بالضبط بالسطر الأول:

| Trip_ID | Supervisor_ID | Supervisor_Name | Start_Time | End_Time | Status |
|---|---|---|---|---|---|

اتركها فارغة تماماً بعد السطر الأول — الكود يعبّيها تلقائياً بكل رحلة.

---

## 2) النشر (نفس طريقتكم المعتادة)

1. **`Code.gs`**: الصقه بمحرر Apps Script (استبدال كامل) ← Deploy ← Manage deployments ←
   ✏️ ← New version ← Deploy. **لا تستخدم "+ New Deployment"**.
2. **`index.html`** و**`Masar-staff.html`**: ارفعهما على GitHub بنفس الاسمين بالضبط
   (استبدال الموجود).
3. **`Masar-supervisor.html`** و**`manifest-supervisor.json`**: ملفان جديدان تماماً —
   ارفعهما بنفس مجلد `Masar/` على GitHub (بجانب باقي الملفات).
4. افتح كل صفحة بـ **Ctrl+Shift+R** بعد الرفع.

بعد هذا، صفحة المشرفة تشتغل فوراً كصفحة ويب عادية على:
`https://masar-school.github.io/Masar/Masar-supervisor.html`
— جربها بمتصفح الجوال قبل أي تغليف أندرويد، للتأكد إن تسجيل الدخول وبدء/إنهاء الرحلة
وإرسال الموقع تعمل صح.

**ملاحظة عن الأيقونة:** ملف `manifest-supervisor.json` يستخدم مؤقتاً نفس أيقونة تطبيق
الأهالي (`icon-parent-192.png`) كحل سريع. لما يجهز "شعار مسار" المخصص لتطبيق السائق،
بس بدّل اسم الملف بداخل `manifest-supervisor.json` وارفع الصورة الجديدة (يفضّل مقاس
512×512 فعلي، مو نفس صورة الـ192 مكرّرة).

---

## 3) لماذا التغليف بأندرويد ضروري (تذكير سريع)

صفحة ويب عادية (حتى لو PWA مثبّتة) بتوقف إرسال GPS فور ما المشرفة تفتح واتساب أو تصغّر
الشاشة — قيد من نظام التشغيل نفسه، مو مشكلة بالكود. الحل الوحيد الموثوق: تطبيق أندرويد
حقيقي بصلاحية **Foreground Service** (نفس أسلوب واتساب وتطبيقات التوصيل).

---

## 4) تغليف أندرويد — خطوات Bubblewrap

> ⚠️ هذا الجزء يحتاج جهاز كمبيوتر عندك فيه اتصال إنترنت و Node.js — ما ينفّذ من هون.

### أ) التثبيت (مرة واحدة)
```bash
npm install -g @bubblewrap/cli
```
سيطلب منك تثبيت Android SDK وJava (JDK) تلقائياً أول مرة — وافق واتركه يحمّلهم.

### ب) توليد المشروع
```bash
bubblewrap init --manifest=https://masar-school.github.io/Masar/manifest-supervisor.json
```
سيسألك عدة أسئلة، أهمها:
- **Application ID (package name):** `com.masarschool.busdriver`
- **App name:** مسار - تتبع الباص
- **Signing key:** اختر "Create new key" أول مرة — **احتفظ بملف الـ keystore وكلمة سره
  بمكان آمن جداً**، لأنك رح تحتاجه لأي تحديث مستقبلي لنفس التطبيق (فقدانه = ما تقدر
  تحدّث التطبيق أبداً، لازم تنزّله من جديد بمعرّف مختلف).

بعد الانتهاء، الأداة تطبع لك **SHA-256 fingerprint** الخاص بمفتاح التوقيع — انسخه،
رح تحتاجه بالخطوة الجاية.

### ج) ملف التحقق `assetlinks.json` (إلزامي)
أنشئ هذا الملف بجهازك بالمحتوى التالي (استبدل `PASTE_SHA256_HERE` بالقيمة اللي طلعت
بالخطوة السابقة):
```json
[{
  "relation": ["delegate_permission/common.handle_all_urls"],
  "target": {
    "namespace": "android_app",
    "package_name": "com.masarschool.busdriver",
    "sha256_cert_fingerprints": ["PASTE_SHA256_HERE"]
  }
}]
```
**مهم:** لازم يترفع بالضبط على:
`https://masar-school.github.io/.well-known/assetlinks.json`
— يعني **بجذر نطاق `masar-school.github.io` نفسه**، مو جوا مجلد `Masar/`. إذا مستودع
GitHub Pages عندك اسمه `masar-school.github.io`، أنشئ مجلد `.well-known` بأعلى مستوى
بالمستودع (بجانب مجلد `Masar/`، مو بداخله) وحط الملف فيه.

بدون هذا الملف بالمكان الصحيح، التطبيق سيفتح بشريط عنوان متصفح ظاهر بالأعلى (يشتغل
لكن ما يبدو كتطبيق حقيقي بالكامل).

### د) إضافة Foreground Service (الجزء الأهم — يحتاج فتح المشروع بـ Android Studio)
Bubblewrap يولّد لك "غلاف" فاضي فقط — خدمة تتبع الموقع بالخلفية لازم تُضاف يدوياً:

1. افتح المجلد اللي ولّده Bubblewrap بـ **Android Studio**.
2. أضف بملف `AndroidManifest.xml`:
```xml
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_BACKGROUND_LOCATION" />
<uses-permission android:name="android.permission.FOREGROUND_SERVICE" />
<uses-permission android:name="android.permission.FOREGROUND_SERVICE_LOCATION" />
```
وسجّل خدمة جديدة (مثال `LocationService`) بنوع `foregroundServiceType="location"`.

3. أنشئ ملف Kotlin جديد (مثال مبسّط — الفكرة العامة، يحتاج تعديل بسيط حسب إصدار
   Android Studio عندك):
```kotlin
class LocationService : Service() {
    private lateinit var fusedClient: FusedLocationProviderClient
    private var username = ""
    private var password = ""

    override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
        username = intent?.getStringExtra("username") ?: ""
        password = intent?.getStringExtra("password") ?: ""
        startForeground(1, buildNotification()) // إشعار ثابت "مسار: جاري تتبع الرحلة"
        fusedClient = LocationServices.getFusedLocationProviderClient(this)
        val request = LocationRequest.create().apply {
            interval = 12000
            priority = Priority.PRIORITY_HIGH_ACCURACY
        }
        fusedClient.requestLocationUpdates(request, object : LocationCallback() {
            override fun onLocationResult(result: LocationResult) {
                val loc = result.lastLocation ?: return
                postLocationToServer(loc.latitude, loc.longitude, username, password)
            }
        }, Looper.getMainLooper())
        return START_STICKY
    }

    private fun postLocationToServer(lat: Double, lng: Double, u: String, p: String) {
        // HTTP POST عادي إلى API_URL بنفس body الموجود بـ Masar-supervisor.html:
        // { action: "updateLocation", username: u, password: p, lat: lat, lng: lng }
        // استخدم OkHttp أو HttpURLConnection على Thread منفصل (ممنوع بالـ Main Thread)
    }
    override fun onBind(intent: Intent?): IBinder? = null
}
```
4. اربطها بصفحة الويب عبر **JavascriptInterface**: لما المستخدم يضغط "ابدأ الرحلة"
   بصفحة `Masar-supervisor.html`، بدل الاعتماد على `watchPosition` بالجافاسكربت وحده
   (يتوقف بالخلفية)، استدعي دالة تُشغّل `LocationService` الأصلية من كوتلن (تمرّر لها
   username/password المخزّنين). هذا يخلي إرسال الموقع مستمر حتى لو المشرفة فتحت
   واتساب، لأن الخدمة تعمل خارج WebView تماماً.

5. زر "إنهاء الرحلة" بالصفحة يستدعي بالمقابل دالة توقف `LocationService`
   (`stopService`).

> هذا الجزء (خطوة د) هو الوحيد اللي يحتاج فعلاً كتابة كود أندرويد أصلي (Kotlin) —
> باقي التطبيق (تسجيل الدخول، قائمة الطلاب، الإحصائيات) شغال 100% من نفس صفحة الويب
> بدون أي تعديل إضافي.

### هـ) البناء والتوزيع
```bash
bubblewrap build
```
ينتج ملف `app-release-signed.apk` — انسخه لهاتف كل مشرفة/سائق وثبّته يدوياً (لازم تفعيل
"السماح بمصادر غير معروفة" بإعدادات أندرويد أول مرة). لا حاجة لرفعه على Google Play.

---

## 6) خيار وسط أسرع (إذا حاب تجرب قبل استثمار وقت بـ Kotlin)
جرّب أولاً **المستوى 1** بدون أي تطبيق أندرويد إطلاقاً: افتحي `Masar-supervisor.html`
كصفحة PWA عادية بمتصفح كروم بالجوال، واطلبي من المشرفات الالتزام بإبقاء الشاشة مفتوحة
وعدم الرد على واتساب أثناء الرحلة. إذا انضبطن عملياً، توفر عليك كل تعقيد خطوة (د) أعلاه.
إذا فشل الانضباط (المتوقع حسب كلامك)، انتقل مباشرة لخطوة (د).
