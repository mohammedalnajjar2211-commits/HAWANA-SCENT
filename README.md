# HAWANA SCENT — hawanascent.com

متجر عربي RTL لأجهزة التعطير والزيوت العطرية، قابل للنشر على GitHub Pages.

## 1. تعديل المنتجات
افتح `products.json` وغيّر `name` و`description` و`price` (عدد صحيح بالدينار العراقي). جميع الأسعار **0 عمداً** إلى أن تُدخل أسعارك الحقيقية؛ عند السعر 0 يتعطل زر إضافة المنتج. استبدل صور SVG التصورية في `assets/` بصور منتجاتك الحقيقية وحدث حقل `image`. الصور الحالية **تصورية وليست صور منتجات حقيقية**.

**مهم جداً:** بعد تغيير الأسعار، انسخ نفس `products.json` إلى `worker/catalog.json` ثم أعد نشر Worker. السعر الحقيقي يتحقق منه الخادم وليس المتصفح.

## 2. النشر على GitHub Pages
أنشئ مستودعاً جديداً `hawanascent`، وارفع **محتويات هذا المجلد** إلى الجذر (لا ترفع ZIP كما هو). من Settings → Pages → Deploy from a branch → main → / (root). ملف `CNAME` يحوي `hawanascent.com`. اربط DNS بحسب القيم التي تعرضها GitHub Pages لعنوان النطاق، مع تفعيل HTTPS. لا تضع مفاتيح API في GitHub أو `config.js`.

## 3. الدفع — Cloudflare Worker + D1
يحتاج حساب SwiftPayIQ مفعّلاً ومفتاح API (صلاحية `invoices:write`) وسر توقيع Webhook. يحتاج حساب Cloudflare لتشغيل Worker وD1.

داخل `worker/`:

```bash
npx wrangler login
npx wrangler d1 create hawana-orders
```

ضع `database_id` الناتج في `wrangler.toml`، ثم:

```bash
npx wrangler d1 execute hawana-orders --remote --file=./schema.sql
npx wrangler secret put SWIFTPAY_API_KEY
npx wrangler secret put SWIFTPAY_WEBHOOK_SECRET
npx wrangler deploy
```

أدخل رابط Worker الناتج في `config.js` تحت `apiBase` وغيّر `paymentsEnabled` إلى `true` **فقط بعد اختبار الربط**. سجّل عنوان `https://YOUR-WORKER.workers.dev/api/webhook` في لوحة SwiftPayIQ. تحقق من إعداد وضع الاختبار/الإنتاج لدى المزود.

## 4. اختبار قبل إطلاق الدفع
- اختبار إنشاء فاتورة من منتج واحد ومتعدد، وإدخال عنوان وهاتف عراقي.
- اختبار Webhook بتوقيع صحيح وخاطئ، وحدث مكرر، واختلاف المبلغ.
- التأكد من أن الطلب يصبح `paid` في D1 فقط بعد إشعار موثق.
- ضبط سياسة الشحن والاسترجاع، وإضافة الصور والمواصفات والأسعار الحقيقية.

**ملاحظة:** الموقع لا يرسل رسائل واتساب تلقائياً، بل يفتح رسالة جاهزة للعميل. لا يوجد نظام مخزون أو لوحة إدارة أو حسابات مستخدمين. تحتاج مراجعة الشروط القانونية والتجارية قبل البيع الفعلي. GitHub Pages يستضيف الواجهة فقط، أما الدفع فينفذ عبر Worker آمن.
