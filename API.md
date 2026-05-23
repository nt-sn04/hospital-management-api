# 📡 API.md — Endpoint dokumentatsiyasi

Hospital Management API'ning **barcha endpointlari** shu fayldan topiladi.

> **Base URL:** `http://127.0.0.1:8000/api/`
>
> **Format:** JSON
>
> **Auth:** ❌ Yo'q (MVP versiyasi)

---

## 📑 Bo'limlar

1. [Umumiy qoidalar](#-umumiy-qoidalar)
2. [Doctors](#-doctors)
3. [Patients](#-patients)
4. [Appointments](#-appointments)
5. [Prescriptions](#-prescriptions)
6. [Laboratory](#-laboratory)
7. [Dashboard](#-dashboard)
8. [Error formatlar](#-error-formatlar)

---

## 🧭 Umumiy qoidalar

### HTTP Status Codes

| Code  | Ma'nosi                                       |
| ----- | --------------------------------------------- |
| `200` | OK — so'rov muvaffaqiyatli                    |
| `201` | Created — yangi resurs yaratildi              |
| `204` | No Content — o'chirildi (response body yo'q)  |
| `400` | Bad Request — validation xatosi               |
| `404` | Not Found — resurs topilmadi                  |
| `500` | Server Error — kod xatosi                     |

### Request headers

Har bir POST / PUT so'rovida:

```
Content-Type: application/json
```

### Sana / vaqt formati

Hamma `DateTime` field'lar ISO 8601 formatida:

```
2026-05-25T14:00:00
```

---

## 👨‍⚕️ Doctors

### 1. Doctorlar ro'yxati

**`GET /api/doctors/`**

#### Query parameters

| Parametr           | Misol           | Tavsif                                  |
| ------------------ | --------------- | --------------------------------------- |
| `specialization`   | `Cardiologist`  | Specialization nomi bo'yicha filter     |

#### Misol so'rov

```bash
curl http://127.0.0.1:8000/api/doctors/?specialization=Cardiologist
```

#### Response — `200 OK`

```json
[
    {
        "id": 1,
        "full_name": "Ali Karimov",
        "specialization": 2,
        "specialization_name": "Cardiologist",
        "phone": "+998901234567",
        "room_number": "204",
        "experience_years": 8,
        "consultation_fee": "150000.00"
    },
    {
        "id": 3,
        "full_name": "Dilshod Yusupov",
        "specialization": 2,
        "specialization_name": "Cardiologist",
        "phone": "+998935551122",
        "room_number": "206",
        "experience_years": 12,
        "consultation_fee": "200000.00"
    }
]
```

---

### 2. Yangi doctor yaratish

**`POST /api/doctors/`**

#### Request body

```json
{
    "full_name": "Sherzod Nazarov",
    "specialization": 1,
    "phone": "+998901111111",
    "room_number": "301",
    "experience_years": 5,
    "consultation_fee": "120000.00"
}
```

#### Response — `201 Created`

```json
{
    "id": 5,
    "full_name": "Sherzod Nazarov",
    "specialization": 1,
    "specialization_name": "Neurologist",
    "phone": "+998901111111",
    "room_number": "301",
    "experience_years": 5,
    "consultation_fee": "120000.00"
}
```

---

### 3. Doctor detail

**`GET /api/doctors/{id}/`**

#### Response — `200 OK`

```json
{
    "id": 1,
    "full_name": "Ali Karimov",
    "specialization": 2,
    "specialization_name": "Cardiologist",
    "phone": "+998901234567",
    "room_number": "204",
    "experience_years": 8,
    "consultation_fee": "150000.00"
}
```

---

### 4. Doctorni yangilash

**`PUT /api/doctors/{id}/`**

#### Request body

To'liq obyekt yuboring (PATCH emas, PUT).

```json
{
    "full_name": "Ali Karimov",
    "specialization": 2,
    "phone": "+998901234567",
    "room_number": "205",
    "experience_years": 9,
    "consultation_fee": "180000.00"
}
```

#### Response — `200 OK`

Yangilangan obyekt qaytadi.

---

### 5. Doctorni o'chirish

**`DELETE /api/doctors/{id}/`**

#### Response — `204 No Content`

Response body yo'q.

---

### 6. Doctor schedule

**`GET /api/doctors/{id}/schedule/`**

Doctorning kelajakdagi appointmentlari.

#### Response — `200 OK`

```json
{
    "doctor": "Ali Karimov",
    "upcoming_appointments": [
        {
            "id": 12,
            "patient_name": "Nodira Saidova",
            "appointment_time": "2026-05-25T14:00:00",
            "status": "CONFIRMED"
        },
        {
            "id": 15,
            "patient_name": "Olim Mahmudov",
            "appointment_time": "2026-05-25T15:30:00",
            "status": "PENDING"
        }
    ]
}
```

---

## 🧑 Patients

### 1. Patientlar ro'yxati

**`GET /api/patients/`**

#### Response — `200 OK`

```json
[
    {
        "id": 1,
        "full_name": "Nodira Saidova",
        "phone": "+998935559988",
        "gender": "F",
        "birth_date": "1992-03-15",
        "blood_group": "A+",
        "address": "Toshkent sh., Yunusobod tum."
    }
]
```

---

### 2. Yangi patient yaratish

**`POST /api/patients/`**

#### Request body

```json
{
    "full_name": "Olim Mahmudov",
    "phone": "+998934447755",
    "gender": "M",
    "birth_date": "1985-11-20",
    "blood_group": "O+",
    "address": "Samarqand sh."
}
```

#### Response — `201 Created`

```json
{
    "id": 7,
    "full_name": "Olim Mahmudov",
    "phone": "+998934447755",
    "gender": "M",
    "birth_date": "1985-11-20",
    "blood_group": "O+",
    "address": "Samarqand sh."
}
```

#### Validation qoidalari

| Field         | Qoida                                              |
| ------------- | -------------------------------------------------- |
| `gender`      | Faqat `"M"` yoki `"F"`                             |
| `blood_group` | `A+`, `A-`, `B+`, `B-`, `AB+`, `AB-`, `O+`, `O-`   |
| `birth_date`  | `YYYY-MM-DD` formatda                              |

---

### 3. Patient detail

**`GET /api/patients/{id}/`**

#### Response — `200 OK`

Yuqoridagi formatda bitta obyekt qaytadi.

---

### 4. Patient yangilash

**`PUT /api/patients/{id}/`**

`POST` bilan bir xil format.

---

### 5. Patient o'chirish

**`DELETE /api/patients/{id}/`**

Response — `204 No Content`.

---

### 6. Patient History ⭐

**`GET /api/patients/{id}/history/`**

Patient haqida **hamma narsa**: appointments, prescriptions, lab orders.

#### Response — `200 OK`

```json
{
    "id": 1,
    "full_name": "Nodira Saidova",
    "phone": "+998935559988",
    "gender": "F",
    "birth_date": "1992-03-15",
    "blood_group": "A+",
    "appointments": [
        {
            "id": 12,
            "doctor_name": "Ali Karimov",
            "appointment_time": "2026-05-20T10:00:00",
            "status": "COMPLETED",
            "notes": "Bosh og'rig'i",
            "prescription": {
                "diagnosis": "Migrain",
                "advice": "Suvni ko'p iching, charchamang",
                "items": [
                    {
                        "medicine_name": "Paracetamol 500mg",
                        "dosage": "Kuniga 2 mahal",
                        "duration_days": 5,
                        "instruction": "Ovqatdan keyin"
                    }
                ]
            },
            "lab_orders": [
                {
                    "id": 4,
                    "items": [
                        {
                            "test_name": "Blood Test",
                            "is_done": true,
                            "result": "Norm"
                        }
                    ]
                }
            ]
        }
    ]
}
```

---

## 📅 Appointments

> **Bu modul — projectning yuragi.** E'tibor bilan o'qing.

### 1. Appointmentlar ro'yxati

**`GET /api/appointments/`**

#### Query parameters

| Parametr   | Misol         | Tavsif                       |
| ---------- | ------------- | ---------------------------- |
| `status`   | `PENDING`     | Status bo'yicha filter       |
| `doctor`   | `3`           | Doctor ID bo'yicha filter    |

#### Misol so'rov

```bash
curl "http://127.0.0.1:8000/api/appointments/?status=PENDING&doctor=3"
```

#### Response — `200 OK`

```json
[
    {
        "id": 12,
        "patient": 1,
        "patient_name": "Nodira Saidova",
        "doctor": 3,
        "doctor_name": "Dilshod Yusupov",
        "appointment_time": "2026-05-25T14:00:00",
        "status": "PENDING",
        "notes": "Bosh og'rig'i",
        "created_at": "2026-05-23T09:15:00"
    }
]
```

---

### 2. Yangi appointment yaratish

**`POST /api/appointments/`**

#### Request body

```json
{
    "patient": 1,
    "doctor": 3,
    "appointment_time": "2026-05-25T14:00:00",
    "notes": "Bosh og'rig'i"
}
```

#### Response — `201 Created`

```json
{
    "id": 18,
    "patient": 1,
    "doctor": 3,
    "appointment_time": "2026-05-25T14:00:00",
    "notes": "Bosh og'rig'i"
}
```

#### ⚠️ Validation qoidalari

**1. Doctor double booking** — agar shu doctor shu vaqtda boshqa appointment'i bo'lsa (status `PENDING` yoki `CONFIRMED`):

```json
{
    "appointment_time": ["Doctor bu vaqtda band."]
}
```

→ `400 Bad Request`

**2. O'tmishdagi vaqt** — appointment vaqti o'tmishda bo'lsa:

```json
{
    "appointment_time": ["Appointment vaqti o'tmishda bo'lishi mumkin emas."]
}
```

→ `400 Bad Request`

---

### 3. Appointment detail

**`GET /api/appointments/{id}/`**

#### Response — `200 OK`

```json
{
    "id": 12,
    "patient": 1,
    "patient_name": "Nodira Saidova",
    "doctor": 3,
    "doctor_name": "Dilshod Yusupov",
    "appointment_time": "2026-05-25T14:00:00",
    "status": "PENDING",
    "notes": "Bosh og'rig'i",
    "created_at": "2026-05-23T09:15:00"
}
```

---

### 4. Appointmentni yangilash

**`PUT /api/appointments/{id}/`**

Vaqtni o'zgartirish double booking validation'dan o'tadi.

---

### 5. Appointmentni o'chirish

**`DELETE /api/appointments/{id}/`**

Response — `204 No Content`.

> 💡 **Best practice:** appointment'ni o'chirish o'rniga `cancel` qiling — tarix saqlanadi.

---

### 6. Cancel appointment

**`POST /api/appointments/{id}/cancel/`**

Request body kerak emas.

#### Response — `200 OK`

```json
{
    "id": 12,
    "status": "CANCELLED",
    "patient_name": "Nodira Saidova",
    "doctor_name": "Dilshod Yusupov",
    "appointment_time": "2026-05-25T14:00:00"
}
```

#### ⚠️ Xato holatlar

`COMPLETED` appointment'ni cancel qilib bo'lmaydi:

```json
{
    "detail": "Tugagan appointmentni cancel qilib bo'lmaydi."
}
```

→ `400 Bad Request`

Allaqachon cancel qilingan bo'lsa:

```json
{
    "detail": "Allaqachon cancel qilingan."
}
```

→ `400 Bad Request`

---

### 7. Complete appointment

**`POST /api/appointments/{id}/complete/`**

#### Response — `200 OK`

```json
{
    "id": 12,
    "status": "COMPLETED",
    "patient_name": "Nodira Saidova",
    "doctor_name": "Dilshod Yusupov",
    "appointment_time": "2026-05-25T14:00:00"
}
```

#### ⚠️ Qoida

Faqat `CONFIRMED` holatdagi appointment'ni complete qilish mumkin:

```json
{
    "detail": "Faqat CONFIRMED appointmentni complete qilish mumkin."
}
```

→ `400 Bad Request`

---

### 8. Bugungi appointmentlar

**`GET /api/appointments/today/`**

#### Response — `200 OK`

Bugungi appointmentlar ro'yxati (yuqoridagi `GET /appointments/` formati bilan bir xil).

---

## 💊 Prescriptions

### 1. Prescription yaratish

**`POST /api/appointments/{appointment_id}/prescription/`**

#### Request body

```json
{
    "diagnosis": "Migrain",
    "advice": "Suvni ko'p iching, charchamang",
    "items": [
        {
            "medicine": 1,
            "dosage": "Kuniga 2 mahal",
            "duration_days": 5,
            "instruction": "Ovqatdan keyin"
        },
        {
            "medicine": 4,
            "dosage": "Kuniga 1 mahal kechqurun",
            "duration_days": 7,
            "instruction": "Yotishdan oldin"
        }
    ]
}
```

#### Response — `201 Created`

```json
{
    "id": 5,
    "appointment": 12,
    "diagnosis": "Migrain",
    "advice": "Suvni ko'p iching, charchamang",
    "created_at": "2026-05-23T11:30:00",
    "items": [
        {
            "id": 8,
            "medicine": 1,
            "medicine_name": "Paracetamol 500mg",
            "dosage": "Kuniga 2 mahal",
            "duration_days": 5,
            "instruction": "Ovqatdan keyin"
        },
        {
            "id": 9,
            "medicine": 4,
            "medicine_name": "Vitamin B12",
            "dosage": "Kuniga 1 mahal kechqurun",
            "duration_days": 7,
            "instruction": "Yotishdan oldin"
        }
    ]
}
```

#### ⚠️ Validation qoidalari

**1. Appointment `COMPLETED` bo'lishi shart:**

```json
{
    "detail": "Faqat tugagan appointmentga retsept yozish mumkin."
}
```

→ `400 Bad Request`

**2. Bitta appointmentga faqat bitta prescription:**

```json
{
    "detail": "Bu appointment uchun retsept allaqachon mavjud."
}
```

→ `400 Bad Request`

**3. `items` array bo'sh bo'lmasligi kerak** — kamida bitta medicine.

---

### 2. Medicine ro'yxati (yordamchi endpoint)

**`GET /api/medicines/`**

#### Response — `200 OK`

```json
[
    {
        "id": 1,
        "name": "Paracetamol 500mg",
        "manufacturer": "Yuvaschem",
        "price": "8500.00"
    },
    {
        "id": 2,
        "name": "Ibuprofen 200mg",
        "manufacturer": "Remedy Group",
        "price": "12000.00"
    }
]
```

---

## 🧪 Laboratory

### 1. Lab order yaratish

**`POST /api/appointments/{appointment_id}/lab-order/`**

#### Request body

Faqat test ID'larini ro'yxat qilib yuboring:

```json
{
    "tests": [1, 2, 5]
}
```

#### Response — `201 Created`

```json
{
    "id": 10,
    "appointment": 12,
    "patient_name": "Nodira Saidova",
    "created_at": "2026-05-23T11:45:00",
    "items": [
        {
            "id": 14,
            "test": 1,
            "test_name": "Umumiy qon tahlili",
            "result": "",
            "is_done": false
        },
        {
            "id": 15,
            "test": 2,
            "test_name": "Siydik tahlili",
            "result": "",
            "is_done": false
        },
        {
            "id": 16,
            "test": 5,
            "test_name": "MRI",
            "result": "",
            "is_done": false
        }
    ]
}
```

#### ⚠️ Validation qoidalari

**1. `tests` array bo'sh bo'lmasligi kerak:**

```json
{
    "tests": ["This list may not be empty."]
}
```

→ `400 Bad Request`

**2. Mavjud bo'lmagan test ID:**

```json
{
    "tests": ["Quyidagi test ID'lari topilmadi: [99]"]
}
```

→ `400 Bad Request`

---

### 2. Lab test ro'yxati (yordamchi endpoint)

**`GET /api/lab-tests/`**

#### Response — `200 OK`

```json
[
    {
        "id": 1,
        "name": "Umumiy qon tahlili",
        "price": "45000.00"
    },
    {
        "id": 2,
        "name": "Siydik tahlili",
        "price": "30000.00"
    }
]
```

---

## 📊 Dashboard

### 1. Bugungi statistika

**`GET /api/dashboard/today/`**

#### Response — `200 OK`

```json
{
    "date": "2026-05-23",
    "appointments": 24,
    "completed": 18,
    "cancelled": 2,
    "pending": 4,
    "unique_patients": 21
}
```

---

### 2. Top doctorlar

**`GET /api/dashboard/top-doctors/`**

Eng ko'p `COMPLETED` appointment'ga ega 5 ta doctor.

#### Response — `200 OK`

```json
[
    {
        "id": 3,
        "full_name": "Dilshod Yusupov",
        "completed_appointments": 47
    },
    {
        "id": 1,
        "full_name": "Ali Karimov",
        "completed_appointments": 35
    },
    {
        "id": 5,
        "full_name": "Sherzod Nazarov",
        "completed_appointments": 28
    }
]
```

---

## ❌ Error formatlar

### Validation xatosi — `400 Bad Request`

Field bo'yicha:

```json
{
    "phone": ["Bu maydon majburiy."],
    "consultation_fee": ["A valid number is required."]
}
```

Object-level (umumiy):

```json
{
    "non_field_errors": ["Doctor bu vaqtda band."]
}
```

Yoki custom:

```json
{
    "detail": "Faqat tugagan appointmentga retsept yozish mumkin."
}
```

---

### Resurs topilmadi — `404 Not Found`

```json
{
    "detail": "Not found."
}
```

---

### Server xatosi — `500 Internal Server Error`

Bu code'da xato borligini bildiradi. Productionga chiqishdan oldin tuzating.

---

## 🗂 Endpointlar umumiy ro'yxati

| Method   | URL                                                | Tavsif                  |
| -------- | -------------------------------------------------- | ----------------------- |
| `GET`    | `/api/doctors/`                                    | Doctor list             |
| `POST`   | `/api/doctors/`                                    | Doctor yaratish         |
| `GET`    | `/api/doctors/{id}/`                               | Doctor detail           |
| `PUT`    | `/api/doctors/{id}/`                               | Doctor yangilash        |
| `DELETE` | `/api/doctors/{id}/`                               | Doctor o'chirish        |
| `GET`    | `/api/doctors/{id}/schedule/`                      | Doctor schedule         |
| `GET`    | `/api/patients/`                                   | Patient list            |
| `POST`   | `/api/patients/`                                   | Patient yaratish        |
| `GET`    | `/api/patients/{id}/`                              | Patient detail          |
| `PUT`    | `/api/patients/{id}/`                              | Patient yangilash       |
| `DELETE` | `/api/patients/{id}/`                              | Patient o'chirish       |
| `GET`    | `/api/patients/{id}/history/`                      | Patient history         |
| `GET`    | `/api/appointments/`                               | Appointment list        |
| `POST`   | `/api/appointments/`                               | Appointment yaratish    |
| `GET`    | `/api/appointments/today/`                         | Bugungi appointmentlar  |
| `GET`    | `/api/appointments/{id}/`                          | Appointment detail      |
| `PUT`    | `/api/appointments/{id}/`                          | Appointment yangilash   |
| `DELETE` | `/api/appointments/{id}/`                          | Appointment o'chirish   |
| `POST`   | `/api/appointments/{id}/cancel/`                   | Cancel                  |
| `POST`   | `/api/appointments/{id}/complete/`                 | Complete                |
| `POST`   | `/api/appointments/{id}/prescription/`             | Prescription yaratish   |
| `GET`    | `/api/medicines/`                                  | Medicine list           |
| `POST`   | `/api/appointments/{id}/lab-order/`                | Lab order yaratish      |
| `GET`    | `/api/lab-tests/`                                  | Lab test list           |
| `GET`    | `/api/dashboard/today/`                            | Bugungi statistika      |
| `GET`    | `/api/dashboard/top-doctors/`                      | Top 5 doctor            |

---

## 🧪 Postman uchun maslahat

Test qilish uchun shu tartibni tavsiya qilamiz:

1. **Specialization** va **Medicine**, **LabTest** ma'lumotlarini admin paneldan qo'shing.
2. `POST /api/doctors/` — bir nechta doctor yarating.
3. `POST /api/patients/` — bir nechta patient yarating.
4. `POST /api/appointments/` — appointment yarating.
5. Shu vaqtga **xuddi shu doctor** uchun yana bitta appointment yaratib ko'ring — `400` kelishi kerak.
6. Birinchi appointment'ni `PUT` orqali status'ini `CONFIRMED` qiling.
7. `POST /api/appointments/{id}/complete/` — tugating.
8. `POST /api/appointments/{id}/prescription/` — retsept yozing.
9. `GET /api/patients/{id}/history/` — hammasini ko'ring.
10. `GET /api/dashboard/today/` — statistika tekshiring.

Agar bu 10 ta qadam **xatosiz** o'tsa — API'ngiz to'g'ri ishlayapti. ✅
