<div align="center">

# 🚀 XHTTP Relay ECO (VrcLIraniCore)

**A Low-Overhead & Hardened XHTTP Relay on Vercel Node Runtime**

[![Version](https://img.shields.io/badge/Version-1.0.0--eco-blue.svg?style=for-the-badge)]()
[![Runtime](https://img.shields.io/badge/Vercel-Node_Runtime-black.svg?style=for-the-badge&logo=vercel)]()
[![Profile](https://img.shields.io/badge/Profile-ECO_Throttle-2ea44f.svg?style=for-the-badge)]()

هدف این نسخه، کاهش فشار روی منابع با حفظ هسته امنیتی v1.3 است.
این پروژه عمدا سرعت را کنترل می‌کند تا مصرف CPU/Memory و هزینه پایین‌تر بماند.

</div>

---

> ⚠️ **هشدار مهم (Danger Zone)**
> لطفا این پروژه را Fork نکنید و برای استقرار امن از روش دستی (Vercel CLI) استفاده کنید.

---

## ✨ تغییرات نسخه ECO (Low-Overhead + Hardening)

- ⏱️ **Timeout پیشرفته:** `UPSTREAM_TIMEOUT_MS` با پیش‌فرض `120000`.
- 🛡️ **محدودیت متدها:** فقط `GET`, `HEAD`, `POST`.
- 🧹 **Header Filtering سخت‌گیرانه:** حذف headerهای hop-by-hop/proxy و headerهای پلتفرم.
- 🔑 **Auth امن:** فقط header `x-relay-key` (بدون query auth).
- 🛣️ **مسیر اجباری:** `RELAY_PATH` اجباری و Fail-Closed.
- 🐛 **Debug Logging:** لاگ بهتر برای timeout/error/duration.
- 🐢 **سرعت کنترل‌شده:** throttling واقعی روی upload/download.
- 📉 **مصرف پایین‌تر:** Node runtime + `128MB` + کنترل همزمانی.

---

## 🧠 معماری ECO

این نسخه روی Node Serverless اجرا می‌شود و سه لایه دارد:

1. **Core Security Layer**
- مسیر و متد و auth و timeout مثل v1.3

2. **Resource Control Layer**
- `MAX_INFLIGHT` برای محدود کردن درخواست همزمان
- جلوگیری از افزایش فشار لحظه‌ای روی instance

3. **Bandwidth Governor Layer**
- `MAX_UP_BPS` و `MAX_DOWN_BPS` برای محدودسازی نرخ انتقال
- خروجی عملی: سرعت پایدارتر و مصرف کمتر

---

## ⚙️ Deploy روی Vercel (CLI)

### 1) نصب CLI
```bash
npm i -g vercel
```

### 2) ورود
```bash
vercel login
```

### 3) Deploy اولیه
```bash
vercel deploy
```

### 4) تنظیم ENV
در `Settings -> Environment Variables` این مقادیر را ست کنید:

| متغیر | وضعیت | توضیح | پیش‌فرض |
| :--- | :---: | :--- | :--- |
| `TARGET_DOMAIN` | 🔴 اجباری | آدرس upstream | - |
| `RELAY_PATH` | 🔴 اجباری | مسیر مجاز relay (مثلا `/api`) | - |
| `RELAY_KEY` | ⚪ اختیاری | کلید auth از طریق header | خالی |
| `UPSTREAM_TIMEOUT_MS` | ⚪ اختیاری | تایم‌اوت upstream | `120000` |
| `MAX_INFLIGHT` | ⚪ اختیاری | حداکثر درخواست همزمان هر instance | `4` |
| `MAX_UP_BPS` | ⚪ اختیاری | سقف سرعت آپلود (bytes/sec) | `1572864` |
| `MAX_DOWN_BPS` | ⚪ اختیاری | سقف سرعت دانلود (bytes/sec) | `1572864` |

### 5) Deploy نهایی
```bash
vercel --prod
```

---

## 🎚️ تنظیم سرعت نزدیک به «نصف نسخه اصلی»

پروفایل پیش‌فرض ECO طوری تنظیم شده که سرعت را پایین نگه دارد.
اگر خواستی دقیق‌تر کالیبره کنی:

1. سرعت نسخه اصلی را تست کن (مثلا میانگین download Bps)
2. `MAX_DOWN_BPS` را روی حدود 50% همان عدد بگذار
3. `MAX_UP_BPS` را هم نزدیک 50% آپلود نسخه اصلی تنظیم کن
4. اگر latency spike داری، `MAX_INFLIGHT` را کمتر کن (مثلا `2`)

نمونه:
```text
MAX_UP_BPS=1200000
MAX_DOWN_BPS=1800000
MAX_INFLIGHT=2
```

---

## 💻 نمونه کانفیگ کلاینت

```text
vless://UUID-HERE@vercel.com:443?encryption=none&security=tls&sni=vercel.com&fp=chrome&alpn=h2&insecure=0&allowInsecure=0&type=xhttp&host=YOUR-VERCEL-DOMAIN&path=%2Fapi&mode=auto#XHTTP-ECO
```

---

## 🛠️ Status Codes

- `200` : اتصال موفق
- `403` : کلید `x-relay-key` اشتباه است
- `404` : مسیر با `RELAY_PATH` تطابق ندارد
- `405` : متد غیرمجاز است
- `500` : تنظیمات ENV ناقص یا اشتباه
- `502` : خطا در تونل به upstream
- `503` : فشار همزمانی بالاست (`MAX_INFLIGHT`)
- `504` : timeout به upstream

---

## 🔐 استفاده از RELAY_KEY

در صورت تنظیم `RELAY_KEY`، کلاینت باید header زیر را بفرستد:

```text
x-relay-key: YOUR_SECRET_KEY
```

---

## 📂 ساختار پروژه

```text
XHTTPRelayECO/
  api/
    index.js
  package.json
  vercel.json
  README.md
```

---

## License

MIT

