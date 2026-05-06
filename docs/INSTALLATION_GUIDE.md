# دليل التثبيت الشامل - ACE-Step UI

## المتطلبات الأساسية

| الأداة | الإصدار المطلوب | التحقق |
|---|---|---|
| Node.js | >= 18 | `node -v` |
| npm | >= 9 | `npm -v` |
| Python | 3.11 أو 3.12 | `python --version` |
| uv | أحدث إصدار | `uv --version` |
| Git | أي إصدار | `git --version` |

---

## خطوات التثبيت الكاملة

### الخطوة 1 — استنساخ مشروع الواجهة

```bash
git clone https://github.com/fspecii/ace-step-ui
cd ace-step-ui
```

المجلد الرئيسي للمشروع: `/workspaces/ace-step-ui`

---

### الخطوة 2 — استنساخ محرك ACE-Step 1.5 داخل مجلد الواجهة

> **مهم:** يجب أن يكون ACE-Step-1.5 **داخل** مجلد ace-step-ui وليس بجانبه.

```bash
# تأكد أنك داخل مجلد ace-step-ui
cd /workspaces/ace-step-ui

# استنسخ ACE-Step داخل المشروع
git clone https://github.com/ace-step/ACE-Step-1.5
```

الهيكل النهائي:
```
ace-step-ui/
├── ACE-Step-1.5/     <-- هنا بالضبط
├── server/
├── components/
└── ...
```

---

### الخطوة 3 — إعداد البيئة الافتراضية لـ ACE-Step

```bash
cd ACE-Step-1.5

# إنشاء البيئة الافتراضية (تُنشأ باسم venv وليس .venv)
uv venv venv

# تفعيل البيئة الافتراضية
source venv/bin/activate

# تثبيت الحزم
uv pip install -e .

# العودة إلى المجلد الرئيسي
cd ..
```

---

### الخطوة 4 — تشغيل سكريبت الإعداد

```bash
# من مجلد ace-step-ui
./setup.sh
```

هذا السكريبت يقوم بـ:
- التحقق من وجود ACE-Step-1.5
- إنشاء ملف `.env` بالمسارات الصحيحة
- تثبيت تبعيات الواجهة الأمامية (`npm install`)
- تثبيت تبعيات الواجهة الخلفية (`server/npm install`)
- تهيئة قاعدة البيانات

---

### الخطوة 5 — تشغيل المشروع الكامل

```bash
# من مجلد ace-step-ui
./start-all.sh
```

---

## التحقق من نجاح التشغيل

بعد تشغيل `./start-all.sh`، تحقق من السجلات:

```bash
# تحقق من حالة كل خدمة
tail -5 logs/api.log       # يجب أن ترى: Uvicorn running on http://127.0.0.1:8001
tail -5 logs/backend.log   # يجب أن ترى: ACE-Step UI Server running on http://localhost:3001
tail -5 logs/frontend.log  # يجب أن ترى: Local: http://localhost:3000/
```

---

## المشاكل الشائعة والحلول

### مشكلة: `uv run acestep-api` يفشل

**السبب:** البيئة الافتراضية تُنشأ باسم `venv` وليس `.venv` التي يتوقعها `uv run` افتراضيًا.

**الحل المطبق في `start-all.sh`:** استخدام المسار المباشر:
```bash
$ACESTEP_PATH/venv/bin/acestep-api --port 8001
```

---

### مشكلة: `ACESTEP_PATH` خاطئ

**السبب:** الملف `.env` كان يحتوي على:
```
ACESTEP_PATH=/workspaces/ace-step-ui   # خاطئ - مجلد الواجهة وليس ACE-Step
```

**الحل:** يجب أن يكون:
```
ACESTEP_PATH=/workspaces/ace-step-ui/ACE-Step-1.5
```

---

### مشكلة: ملف `.env` لا يُقرأ عند التشغيل

**السبب:** `start-all.sh` كان لا يستدعي `source .env`.

**الحل المطبق:** إضافة هذا في بداية السكريبت:
```bash
if [ -f ".env" ]; then
    set -o allexport
    source .env
    set +o allexport
fi
```
