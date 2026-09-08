# Al-Mozare3 RAG Backend (باك إند تطبيق المزارع الذكي) 🌾🇪🇬

باك إند محلي ذكي عالي الأداء مبني باستخدام **FastAPI** وتقنية **RAG (Retrieval-Augmented Generation)** مخصص لتطبيق **"المزارع" (Al-Mozare3)** للإرشاد الزراعي في جمهورية مصر العربية.

يعتمد النظام على **67 نشرة زراعية رسمية** (محاصيل حقلية، خضر، فاكهة، ودواجن ومواشي) مخزنة في مجلد `data/`، ويقوم بتقسيمها وتضمينها باستخدام **Google Gemini Embeddings (`models/text-embedding-004`)** وتخزينها محلياً في قاعدة بيانات المتجهات **ChromaDB**، مع توليد إجابات زراعية استشارية دقيقة ومبسطة باللهجة والمصطلحات الزراعية المصرية عبر **Google Gemini 1.5 Flash**.

---

## 🌟 مميزات النظام (Key Features)

1. **فهرسة تلقائية وتراكمية ذكية (Smart Incremental Ingestion):**
   - فحص مجلد `data/` واستخراج النصوص من ملفات الـ PDF والـ Word بدقة مع تتبع أرقام الصفحات الأصلية.
   - الفهرسة التراكمية: يتعرف النظام تلقائياً على الملفات التي تم تضمينها سابقاً في ChromaDB ويتجاوزها لتوفير استهلاك الكوتا وتسريع التشغيل الفوري.
2. **شخصية المستشار الزراعي المصري المخصص (Egyptian Farmer Persona):**
   - برومت هندسي متخصص يستقبل سياق المزارع وحقله (`farmer_name`, `crop`, `soil_type`, `irrigation_system`, `governorate`, `crop_age_days`).
   - تخصيص التوصيات بدقة حسب عمر النبات ونوع التربة ومصدر الري.
   - التزام صارم بالنشرات الرسمية مع منع الهلوسة (Strict Grounding Fallback).
3. **جاهزية كاملة لتطبيق فلاتر (Flutter Ready):**
   - تفعيل الـ CORS لكافة الطلبات والمنافذ.
   - دعم مباشر للمحاكي (Android Emulator: `10.0.2.2`) وأجهزة الهاتف الحقيقية عبر شبكة الـ Wi-Fi المحلية.

---

## 🏗️ هيكلية المشروع (Project Structure)

```text
al_mozare3_backend/
├── app/
│   ├── __init__.py
│   ├── config.py             # إعدادات النظام وقراءة .env (pydantic-settings)
│   ├── core/
│   │   ├── __init__.py
│   │   ├── gemini_client.py  # دوال الاتصال بنماذج Google Gemini والتضمين
│   │   └── prompts.py        # برومت المستشار الزراعي المصري وتخصيص سياق الفلاح
│   ├── services/
│   │   ├── __init__.py
│   │   ├── document_loader.py# استخراج النصوص وتقطيعها بفواصل عربية ذكية
│   │   ├── rag_service.py    # محرك البحث الشعاعي واسترجاع الإجابات
│   │   └── vector_store.py   # إدارة قاعدة بيانات المتجهات ChromaDB محلياً
│   ├── api/
│   │   ├── __init__.py
│   │   └── routes.py         # مسارات الـ API (/chat, /ingest, /status, /health)
│   ├── schemas/
│   │   ├── __init__.py
│   │   └── rag_schemas.py    # نماذج Pydantic للطلبات والردود وسياق المزارع
│   └── main.py               # تهيئة FastAPI وخادم التطبيق وإدارة دورة الحياة
├── data/                     # مجلد النشرات الزراعية (يحتوي على 67 ملف PDF رسمي)
├── chroma_db/                # مجلد تخزين المتجهات الدائم (يُنشأ تلقائياً)
├── .env.example              # ملف نموذج للمتغيرات البيئية
├── .env                      # ملف المتغيرات البيئية الفعلي
├── requirements.txt          # قائمة المكتبات المطلوبة
└── README.md                 # دليل التشغيل والتوثيق
```

---

## 🚀 التشغيل السريع (Quick Start)

### 1. تثبيت الاعتماديات (Dependencies)
تأكد من وجود Python 3.10 أو أحدث، ثم قم بتثبيت الحزم المطلوبة:
```bash
pip install -r requirements.txt
```

### 2. ضبط مفتاح Google Gemini API
قم بفتح ملف `.env` وضع مفتاحك الخاص:
```env
GEMINI_API_KEY=AIzaSy...ضع_مفتاحك_هنا...
EMBEDDING_MODEL=models/text-embedding-004
CHAT_MODEL=gemini-1.5-flash
CHROMA_DB_DIR=./chroma_db
DATA_DIR=./data
AUTO_INGEST_ON_STARTUP=true
```
*(يمكنك استخراج مفتاح مجاني من [Google AI Studio](https://aistudio.google.com/app/apikey))*

### 3. تشغيل خادم الباك إند
```bash
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```
أو عبر بايثون مباشرة:
```bash
python app/main.py
```

* سيعمل الخادم على: `http://localhost:8000`
* توثيق السواجر التفاعلي (Swagger UI): `http://localhost:8000/docs`

---

## 📡 مسارات الـ API (API Endpoints)

### 1. محادثة المستشار الزراعي (`POST /api/chat`)
إرسال سؤال المزارع مع السياق الاختياري لحقله:

**الطلب (Request Body):**
```json
{
  "query": "ما هي كمية السماد المطلوبة لري المحاياة في محصول القمح؟",
  "farmer_context": {
    "farmer_name": "عبدالرحمن",
    "crop": "قمح",
    "soil_type": "طينية",
    "irrigation_system": "ري بالغمر",
    "governorate": "كفر الشيخ",
    "crop_age_days": 40
  },
  "history": []
}
```

**الرد (Response):**
```json
{
  "answer": "أهلاً بك يا حاج عبدالرحمن في كفر الشيخ، وربنا يباركلك في محصول القمح.\n\nبناءً على النشرات الرسمية لمحصول القمح في الأراضي القديمة (الطينية):\n1. في رية المحاياة (تكون عادة بعد 21 إلى 25 يوماً من الزراعة أو عند عمر 30-40 يوماً حسب طبيعة الأرض):\n- يضاف ثلثا الجرعة السمادية الآزوتية المقررة (حوالي 50 إلى 75 كجم نيتروجين للفدان).\n- ما يعادل شيكارتين إلى 3 شكاير من يوريا (46.5%) أو نترات النشادر (33.5%).\n2. تنبيه هام: يوضع السماد تكبيشاً أو نثراً قبل الري مباشرة لتجنب تطاير الآزوت وتغذية النبات فوراً مع ماء الري.\n\nلو عندك أي استفسار آخر أنا في خدمتك!",
  "sources": [
    {
      "document": "القمح-فى-الأراضى-القديمة-نهائي_compressed.pdf",
      "page": 12,
      "snippet": "التسميد الآزوتي لمحصول القمح: يضاف بمعدل 75 وحدة آزوت للفدان في الأراضي القديمة..."
    }
  ],
  "relevant_chunks_count": 4
}
```

---

### 2. إعادة فهرسة المستندات يدوياً (`POST /api/ingest`)
إعادة فحص وتحديث النشرات الجديدة دون إعادة تشغيل السيرفر:
```bash
curl -X POST "http://localhost:8000/api/ingest" \
     -H "Content-Type: application/json" \
     -d '{"force_reindex": false}'
```

---

### 3. إحصائيات النظام وقاعدة المتجهات (`GET /api/status`)
```bash
curl "http://localhost:8000/api/status"
```
يرجع عدد الملفات المفهرسة وعدد المتجهات المخزنة في ChromaDB.

---

### 4. فحص الحالة (`GET /health`)
```bash
curl "http://localhost:8000/health"
```

---

## 📱 ربط الباك إند مع تطبيق فلاتر (Flutter Integration)

في تطبيقك `al_mozare3`، يمكنك استخدام كود الخدمة التالي للتواصل مع الباك إند:

```dart
import 'dart:convert';
import 'package:http/http.dart' as http;

class Mozare3RagService {
  // للتشغيل على محاكي الأندرويد (Android Emulator):
  static const String baseUrl = 'http://10.0.2.2:8000';
  
  // للتشغيل على هاتف حقيقي عبر نفس شبكة الواي فاي:
  // static const String baseUrl = 'http://192.168.1.X:8000';

  Future<Map<String, dynamic>> askAdvisor({
    required String query,
    Map<String, dynamic>? farmerContext,
  }) async {
    final url = Uri.parse('$baseUrl/api/chat');

    final payload = {
      'query': query,
      'farmer_context': farmerContext ?? {
        'crop': 'قمح',
        'soil_type': 'طينية',
      },
    };

    final response = await http.post(
      url,
      headers: {'Content-Type': 'application/json'},
      body: jsonEncode(payload),
    );

    if (response.statusCode == 200) {
      final data = jsonDecode(utf8.decode(response.bodyBytes));
      return {
        'answer': data['answer'],
        'sources': data['sources'],
      };
    } else {
      throw Exception('فشل الاتصال بالباك إند: ${response.statusCode}');
    }
  }
}
```
# Mozar3_backend
