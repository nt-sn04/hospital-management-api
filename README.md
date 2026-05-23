# 🏥 Hospital Management API

Kichik klinika uchun **REST API** backend. Django + DRF asosida qurilgan o'quv projecti.

> Bu project sizning DRF darslarida o'rgangan barcha narsalaringizni **bitta real loyihada** birlashtirib amaliyot qilish uchun mo'ljallangan.

---

## 📌 Project haqida

Tasavvur qiling — sizni kichik xususiy klinika backend developer qilib ishga oldi. Klinikaga kerak:

- doctorlar ro'yxatini boshqarish
- patientlarni qayd qilish
- appointment (qabul) yozish
- doctor patientga retsept (prescription) yozishi
- laboratoriya testlari buyurtma qilish
- kun davomidagi statistikani ko'rsatish

Siz shu sistemaning **REST API** qismini yozasiz. Frontend boshqa odam yozadi — sizning vazifangiz toza, ishonchli va mantiqli API berish.

---

## 🎯 Nimani o'rganasiz?

Bu project sizga quyidagi mavzularni **amaliyotda** mustahkamlaydi:

### Models & ORM
- `ForeignKey`, `ManyToManyField`, `OneToOneField`
- `related_name` va reverse relation
- `select_related`, `prefetch_related` (query optimization)
- `annotate`, `aggregate`, `Count` (dashboard uchun)

### Serializers
- `ModelSerializer`
- **Nested serializer** (read va write)
- `SerializerMethodField`
- `validate()` va `validate_<field>()` methodlari
- `create()` va `update()` ni override qilish

### Views
- `APIView` asosida ishlash
- `GET`, `POST`, `PUT`, `DELETE`
- URL parameters (`pk`) va query parameters (`?status=...`)
- Custom action endpointlar (`/cancel/`, `/complete/`)

### Architecture
- Appga to'g'ri bo'lish (`doctors`, `patients`, `appointments` ...)
- URL routinglarni toza tashkil qilish
- Business logic'ni view'dan ajratish

---

## 🛠 Texnologiyalar

| Texnologiya  | Versiya         |
| ------------ | --------------- |
| Python       | 3.10+           |
| Django       | 5.x             |
| DRF          | 3.14+           |
| Database     | SQLite (MVP)    |

> ⚠️ Bu projectda **authentication YO'Q**. Hali auth o'rganmagansiz — keyingi modulda. Hozircha API'lar ochiq.

---

## 📂 Project tuzilishi

```
hospital_api/
├── hospital_api/         # project settings
│   ├── settings.py
│   ├── urls.py
│   └── ...
│
├── doctors/              # Doctor + Specialization
│   ├── models.py
│   ├── serializers.py
│   ├── views.py
│   └── urls.py
│
├── patients/             # Patient + History API
│   ├── models.py
│   ├── serializers.py
│   ├── views.py
│   └── urls.py
│
├── appointments/         # CORE — appointment booking
│   ├── models.py
│   ├── serializers.py
│   ├── views.py
│   └── urls.py
│
├── prescriptions/        # Prescription + Medicine
│   ├── models.py
│   ├── serializers.py
│   ├── views.py
│   └── urls.py
│
├── laboratory/           # LabTest + LabOrder
│   ├── models.py
│   ├── serializers.py
│   ├── views.py
│   └── urls.py
│
├── dashboard/            # statistika endpointlari
│   ├── views.py
│   └── urls.py
│
├── manage.py
├── requirements.txt
├── README.md
├── GUIDE.md              # qadamma-qadam yo'riqnoma
└── API.md                # to'liq API documentation
```

---

## ⚡ Quick Start

### 1. Repoyni clone qiling

```bash
git clone <repo-url>
cd hospital_api
```

### 2. Virtual environment yarating

```bash
python -m venv venv
source venv/bin/activate        # Linux / macOS
venv\Scripts\activate           # Windows
```

### 3. Kerakli paketlarni o'rnating

```bash
pip install -r requirements.txt
```

### 4. Migratsiyalarni qiling

```bash
python manage.py makemigrations
python manage.py migrate
```

### 5. Test data tayyorlang

Admin panelga kirish uchun superuser yarating (faqat data qo'shish uchun):

```bash
python manage.py createsuperuser
python manage.py runserver
```

Endi `http://127.0.0.1:8000/admin/` orqali bir nechta `Specialization`, `Doctor`, `Patient`, `Medicine`, `LabTest` qo'shib qo'ying — testlash uchun kerak bo'ladi.

### 6. API'ni sinab ko'ring

```bash
curl http://127.0.0.1:8000/api/doctors/
```

To'liq API ro'yxati — [`API.md`](./API.md) faylida.

---

## 🗺 Asosiy modullar

| Modul             | Asosiy vazifa                             | Murakkablik   |
| ----------------- | ----------------------------------------- | ------------- |
| **Doctors**       | doctor CRUD + schedule                    | ⭐⭐            |
| **Patients**      | patient CRUD + history endpoint           | ⭐⭐⭐           |
| **Appointments**  | booking + double-booking validation       | ⭐⭐⭐⭐ (CORE)   |
| **Prescriptions** | nested writable serializer + transaction  | ⭐⭐⭐⭐          |
| **Laboratory**    | M2M through, nested response              | ⭐⭐⭐           |
| **Dashboard**     | aggregation, annotate, ordering           | ⭐⭐⭐           |

---

## 📖 Boshlash uchun qayerga qarash kerak?

1. **Birinchi** — [`GUIDE.md`](./GUIDE.md) ni o'qing. Qaysi tartibda yozish, har bir modulda nimaga e'tibor berish — hammasi yozilgan.
2. **Keyin** — [`API.md`](./API.md) ni o'qing. Har bir endpoint qanday ishlashi, request/response formati shu yerda.
3. **Yozib boshlang** — modulma-modul, GUIDE'da ko'rsatilgan tartibda.

---

## ⚠️ Eslatmalar

- **Auth yo'q** — bu o'qish maqsadidagi MVP. Production'ga chiqmaydi.
- **Validatsiyalarga e'tibor bering** — bu projectning eng muhim qismi.
- **Query optimization** — `N+1 problem`ga tushib qolmang. `select_related` va `prefetch_related` ni o'rganib chiqing.
- **Test data yarating** — admin panelda kamida 3 ta doctor, 5 ta patient, 10 ta medicine qo'shing.

---

## 🎓 Bu project tugagandan keyin nima qila olasiz?

- Real Django + DRF projectni noldan ko'tara olasiz
- Nested serializerlar bilan ishlashni biladigan bo'lasiz
- Business logic'ni view'da to'g'ri yozadigan bo'lasiz
- ORM'ni samarali ishlatadigan bo'lasiz
- Keyingi qadamga — **authentication, permissions, JWT** — tayyor bo'lasiz

Omad! 💪
