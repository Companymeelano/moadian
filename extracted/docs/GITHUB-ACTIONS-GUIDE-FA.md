# راهنمای GitHub Actions - ساخت خودکار APK

## تاریخ: ۸ مهر ۱۴۰۵ (۳۰ سپتامبر ۲۰۲۶)

---

## خلاصه

این پروژه دارای **دو workflow** برای ساخت APK است:

| # | نام | هدف | زمان |
|---|---|---|---|
| ۱ | `build-apk.yml` | ساخت همه ۴ flavor (visitor/store/staff/tax) | ~۲۰ دقیقه |
| ۲ | `build-tax-only.yml` ⭐ | فقط flavor مودیان (tax) | ~۵-۷ دقیقه |

---

## 🚀 راه‌اندازی سریع

### مرحله ۱: فعال‌سازی در repository

1. به آدرس زیر بروید:
   ```
   https://github.com/Companymeelano/moadian/settings/actions
   ```

2. در بخش **General → Actions permissions**:
   - ✅ **Allow all actions and reusable workflows** را انتخاب کنید
   - روی **Save** کلیک کنید

### مرحله ۲: (اختیاری) تنظیم کلید امضای اختصاصی

اگر می‌خواهید APK با کلید خودتان امضا شود:

1. یک کلید JKS بسازید:
   ```bash
   keytool -genkey -v -keystore release.jks -keyalg RSA -keysize 2048 -validity 10000 -alias meelano
   ```

2. کلید را به Base64 تبدیل کنید:
   ```bash
   base64 -w 0 release.jks > release.jks.b64
   ```

3. در GitHub:
   - `Settings → Secrets and variables → Actions → New repository secret`
   - ۴ secret اضافه کنید:
     - `MEELANO_KEYSTORE_BASE64` = محتوای `release.jks.b64`
     - `MEELANO_KEYSTORE_PASSWORD` = رمز کلید
     - `MEELANO_KEY_ALIAS` = `meelano`
     - `MEELANO_KEY_PASSWORD` = رمز کلید

4. **اگر این secrets را تنظیم نکنید:** workflow از کلید پیش‌فرض `derakhshan-install` استفاده می‌کند (مناسب برای استفاده داخلی)

### مرحله ۳: اجرای workflow

#### روش ۱: خودکار (پیشنهادی)
هر push به شاخه `arena/**` یا `main` workflow را فعال می‌کند (فقط اگر فایل‌های مرتبط تغییر کرده باشند).

#### روش ۲: دستی
1. به آدرس زیر بروید:
   ```
   https://github.com/Companymeelano/moadian/actions/workflows/build-tax-only.yml
   ```
2. روی **Run workflow** کلیک کنید
3. شاخه و flavor را انتخاب کنید
4. روی **Run workflow** (دکمه سبز) کلیک کنید

---

## 📥 دانلود APK پس از Build

### روش ۱: از Artifacts (موقت)
1. به صفحه Actions بروید
2. روی run موفق کلیک کنید
3. در پایین صفحه، بخش **Artifacts** را ببینید
4. روی `MEELANO-Tax-APK-v6.0.1` کلیک کنید
5. فایل ZIP دانلود می‌شود

### روش ۲: از Releases
1. به آدرس زیر بروید:
   ```
   https://github.com/Companymeelano/moadian/releases
   ```
2. آخرین release با عنوان **"MEELANO Tax v6.0.1 (build X)"** را پیدا کنید
3. فایل APK را دانلود کنید

### روش ۳: لینک مستقیم از شاخه
پس از موفقیت‌آمیز بودن build، فایل‌ها در پوشه `apk/` شاخه قرار می‌گیرند:

```
https://github.com/Companymeelano/moadian/raw/arena/01a0f11a-moadian/apk/MEELANO-Tax-v6.0.1-release.apk
https://github.com/Companymeelano/moadian/raw/arena/01a0f11a-moadian/apk/MEELANO-Tax-v6.0.1-debug.apk
https://github.com/Companymeelano/moadian/raw/arena/01a0f11a-moadian/apk/latest-tax.json
```

---

## 🔍 بررسی وضعیت Build

### وضعیت‌های ممکن:

| نماد | وضعیت | معنی |
|---|---|---|
| 🟢 | Success | Build موفقیت‌آمیز - APK آماده دانلود |
| 🟡 | In progress | در حال اجرا |
| 🔴 | Failed | خطا - روی آن کلیک کنید تا لاگ را ببینید |
| ⚪ | Queued | در صف انتظار |

### مشاهده لاگ:
1. روی run کلیک کنید
2. روی job (مثلاً `build-tax`) کلیک کنید
3. روی هر step کلیک کنید تا جزئیات را ببینید

---

## 🛠️ رفع مشکلات رایج

### مشکل ۱: Workflow اجرا نمی‌شود

**علت:** GitHub Actions در repository غیرفعال است.

**راه‌حل:**
1. `Settings → Actions → General`
2. **Allow all actions** را فعال کنید

### مشکل ۲: خطای Gradle "out of memory"

**علت:** Heap size کم است.

**راه‌حل:** در workflow خط زیر را تغییر دهید:
```yaml
-Dorg.gradle.jvmargs=-Xmx4g
```
به:
```yaml
-Dorg.gradle.jvmargs=-Xmx6g
```

### مشکل ۳: خطای "Permission denied" در push

**علت:** GitHub Actions اجازه push ندارد.

**راه‌حل:**
1. `Settings → Actions → General → Workflow permissions`
2. **Read and write permissions** را انتخاب کنید
3. ✅ **Allow GitHub Actions to create and approve pull requests** را فعال کنید

### مشکل ۴: خطای "keystore not found"

**علت:** کلید امضا در secrets تنظیم نشده.

**راه‌حل:** یا secrets را تنظیم کنید (مرحله ۲)، یا بگذارید workflow از کلید پیش‌فرض استفاده کند.

### مشکل ۵: خطای compile

**علت:** خطای کد Java.

**راه‌حل:**
1. لاگ build را ببینید
2. خط اول با `error:` را پیدا کنید
3. فایل و شماره خط مشخص است
4. کد را اصلاح کنید و push کنید
5. workflow دوباره اجرا می‌شود

---

## 📊 زمان‌بندی Build

| مرحله | زمان تخمینی |
|---|---|
| Checkout source | ۱۰-۲۰ ثانیه |
| Setup Java 17 | ۲۰-۴۰ ثانیه |
| Setup Android SDK | ۳۰-۶۰ ثانیه |
| Setup Gradle | ۲۰-۴۰ ثانیه |
| Gradle dependencies download | ۱-۳ دقیقه |
| Build debug APK | ۱-۲ دقیقه |
| Build release APK | ۱-۲ دقیقه |
| Verify + Upload | ۲۰-۴۰ ثانیه |
| **مجموع** | **۵-۸ دقیقه** |

> **نکته:** Gradle cache پس از اولین build سریع‌تر می‌شود.

---

## 🔄 Workflow خودکار

workflow جدید `build-tax-only.yml` در این شرایط **خودکار** اجرا می‌شود:

✅ تغییر در `MEELANO-Android/app/src/tax/**`
✅ تغییر در `MEelanoTax*.java` (کلاس‌های مالیاتی)
✅ تغییر در خود workflow
✅ push به `arena/**` یا `main`

اگر فقط فایل مستندات (`*.md`) تغییر کند، workflow اجرا **نمی‌شود** (صرفه‌جویی در زمان).

---

## 📋 وضعیت فعلی

| مورد | وضعیت |
|---|---|
| workflow `build-apk.yml` | ✅ موجود (قبلی - همه flavors) |
| workflow `build-tax-only.yml` | ✅ اضافه شد (فقط tax) |
| Push به GitHub | ✅ انجام شد (commit `404d54e`) |
| اجرای خودکار workflow | ⏳ منتظر push بعدی یا اجرای دستی |

---

## 🎯 مراحل بعدی برای دریافت APK

### گزینه A: اجرای دستی (سریع‌ترین)
1. مرورگر را باز کنید
2. به این آدرس بروید:
   ```
   https://github.com/Companymeelano/moadian/actions/workflows/build-tax-only.yml
   ```
3. روی **Run workflow** کلیک کنید
4. شاخه `arena/01a0f11a-moadian` را انتخاب کنید
5. روی **Run workflow** (سبز) کلیک کنید
6. ~۵-۷ دقیقه صبر کنید
7. APK را از Artifacts دانلود کنید

### گزینه B: منتظر push بعدی
- هر push کاری به `arena/01a0f11a-moadian` workflow را فعال می‌کند
- یک push کوچک (مثلاً تغییر در README) کافی است

### گزینه C: از من بخواهید
- بگویید "push کوچک برای فعال‌سازی workflow"
- یک commit خالی push می‌کنم
- workflow خودکار اجرا می‌شود

---

**ساخته شده توسط:** Arena Agent
**تاریخ:** ۸ مهر ۱۴۰۵