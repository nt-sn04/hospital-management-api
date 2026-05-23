# 📘 GUIDE.md — Qadamma-qadam yo'riqnoma

Salom, talaba! 👋

Bu fayl — sizning **ustozingiz yonida o'tirib** loyihani yozib boryotgandek qadamma-qadam yo'l-yo'riq beradi. Har bir bo'limda:

- **Nima qilamiz**
- **Nega aynan shunday**
- **Qaysi DRF mavzusi mustahkamlanmoqda**
- **Tipik xatoliklar**

ko'rsatilgan.

> 💡 **Qoida:** modulni o'tkazib yubormang. Tartib — bu loyihaning qo'shni-qo'shni bo'g'inlari. `Appointment`'ni yozish uchun `Doctor` va `Patient` kerak; `Prescription` uchun `Appointment` kerak.

---

## 📋 Tartib

1. [Project setup](#1-project-setup)
2. [Doctors app](#2-doctors-app)
3. [Patients app](#3-patients-app)
4. [Appointments app — CORE](#4-appointments-app--core)
5. [Prescriptions app](#5-prescriptions-app)
6. [Laboratory app](#6-laboratory-app)
7. [Dashboard app](#7-dashboard-app)
8. [Yakuniy tekshirish](#8-yakuniy-tekshirish)

---

## 1. Project setup

### Maqsad

Django projectni va apparni to'g'ri tashkil qilish.

### Qadamlar

```bash
django-admin startproject hospital_api .
python manage.py startapp doctors
python manage.py startapp patients
python manage.py startapp appointments
python manage.py startapp prescriptions
python manage.py startapp laboratory
python manage.py startapp dashboard
```

`settings.py`:

```python
INSTALLED_APPS = [
    # ...
    'rest_framework',

    'doctors',
    'patients',
    'appointments',
    'prescriptions',
    'laboratory',
    'dashboard',
]
```

`hospital_api/urls.py`:

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/', include('doctors.urls')),
    path('api/', include('patients.urls')),
    path('api/', include('appointments.urls')),
    path('api/', include('prescriptions.urls')),
    path('api/', include('laboratory.urls')),
    path('api/dashboard/', include('dashboard.urls')),
]
```

### 🎯 Maslahat

URL'larni `api/` prefiks bilan boshlang. Bu — production amaliyot. Keyinchalik versiyalash (`api/v1/`) ham osonroq bo'ladi.

---

## 2. Doctors app

### Maqsad

Doctor va Specialization modellari + CRUD + filtering + schedule endpoint.

### Models

```python
# doctors/models.py

class Specialization(models.Model):
    name = models.CharField(max_length=100, unique=True)

    def __str__(self):
        return self.name


class Doctor(models.Model):
    full_name = models.CharField(max_length=150)
    specialization = models.ForeignKey(
        Specialization,
        on_delete=models.PROTECT,
        related_name='doctors'
    )
    phone = models.CharField(max_length=20)
    room_number = models.CharField(max_length=10)
    experience_years = models.PositiveIntegerField(default=0)
    consultation_fee = models.DecimalField(max_digits=10, decimal_places=2)

    def __str__(self):
        return f"Dr. {self.full_name}"
```

### 🔍 E'tibor bering

- `on_delete=models.PROTECT` — agar bir specialization'ga doctor biriktirilgan bo'lsa, uni o'chirishga ruxsat bermaymiz. **CASCADE qilmang** — bu xavfli.
- `related_name='doctors'` — keyinchalik `specialization.doctors.all()` deb yoza olamiz.
- `DecimalField` — pulni hech qachon `FloatField` bilan saqlamang. Yumalash xatosi bo'ladi.

### Serializer

```python
# doctors/serializers.py

class DoctorSerializer(serializers.ModelSerializer):
    specialization_name = serializers.CharField(
        source='specialization.name',
        read_only=True
    )

    class Meta:
        model = Doctor
        fields = [
            'id', 'full_name', 'specialization', 'specialization_name',
            'phone', 'room_number', 'experience_years', 'consultation_fee'
        ]
```

### 🎯 Nima uchun `specialization_name` qo'shdik?

Frontend uchun. `specialization` — bu ID (raqam), lekin foydalanuvchiga `"Cardiologist"` deb ko'rsatish kerak. Shu bilan birga `specialization` ID'ni POST/PUT'da yuborish uchun qoldiramiz.

### Views — APIView bilan

```python
# doctors/views.py

class DoctorListCreateView(APIView):
    def get(self, request):
        qs = Doctor.objects.select_related('specialization').all()

        # Filtering
        spec = request.query_params.get('specialization')
        if spec:
            qs = qs.filter(specialization__name__iexact=spec)

        serializer = DoctorSerializer(qs, many=True)
        return Response(serializer.data)

    def post(self, request):
        serializer = DoctorSerializer(data=request.data)
        serializer.is_valid(raise_exception=True)
        serializer.save()
        return Response(serializer.data, status=201)


class DoctorDetailView(APIView):
    def get_object(self, pk):
        return get_object_or_404(Doctor, pk=pk)

    def get(self, request, pk):
        doctor = self.get_object(pk)
        serializer = DoctorSerializer(doctor)
        return Response(serializer.data)

    def put(self, request, pk):
        doctor = self.get_object(pk)
        serializer = DoctorSerializer(doctor, data=request.data)
        serializer.is_valid(raise_exception=True)
        serializer.save()
        return Response(serializer.data)

    def delete(self, request, pk):
        doctor = self.get_object(pk)
        doctor.delete()
        return Response(status=204)
```

### ⚠️ Tipik xato

`Doctor.objects.all()` — bu **N+1 query** muammosini chaqiradi, chunki har bir doctor uchun specialization alohida so'rov bilan olinadi. Yechim:

```python
Doctor.objects.select_related('specialization').all()
```

`select_related` — `ForeignKey` uchun. `prefetch_related` — `ManyToMany` va reverse uchun.

### Schedule endpoint

```python
class DoctorScheduleView(APIView):
    def get(self, request, pk):
        doctor = get_object_or_404(Doctor, pk=pk)
        upcoming = doctor.appointments.filter(
            appointment_time__gte=timezone.now(),
            status__in=['PENDING', 'CONFIRMED']
        ).order_by('appointment_time')

        serializer = AppointmentShortSerializer(upcoming, many=True)
        return Response({
            'doctor': doctor.full_name,
            'upcoming_appointments': serializer.data
        })
```

### 🎓 Bu modulda mustahkamlandi

- ✅ `ForeignKey` va `related_name`
- ✅ `ModelSerializer` + extra read-only field
- ✅ `APIView` + custom logic
- ✅ Query parameter bilan filtering
- ✅ `select_related` — query optimization

---

## 3. Patients app

### Maqsad

Patient CRUD + **History endpoint** (eng muhim qism).

### Model

```python
# patients/models.py

class Patient(models.Model):
    GENDER_CHOICES = [
        ('M', 'Male'),
        ('F', 'Female'),
    ]
    BLOOD_GROUPS = [
        ('A+', 'A+'), ('A-', 'A-'),
        ('B+', 'B+'), ('B-', 'B-'),
        ('AB+', 'AB+'), ('AB-', 'AB-'),
        ('O+', 'O+'), ('O-', 'O-'),
    ]

    full_name = models.CharField(max_length=150)
    phone = models.CharField(max_length=20)
    gender = models.CharField(max_length=1, choices=GENDER_CHOICES)
    birth_date = models.DateField()
    blood_group = models.CharField(max_length=3, choices=BLOOD_GROUPS)
    address = models.TextField()

    def __str__(self):
        return self.full_name
```

### 🎯 Choices nima uchun?

`gender = "X"` deb yozib qo'yib bo'lmasin uchun. Bu — **data integrity**. Validatsiya database darajasida ham bo'lishi kerak.

### CRUD views

`Doctor`ga o'xshash. O'zingiz yozing.

### History endpoint — DIQQAT BILAN O'QING

Bu endpoint bitta `patient` haqida **hamma narsani** qaytaradi: appointments, prescriptions, lab orders.

```python
# patients/views.py

class PatientHistoryView(APIView):
    def get(self, request, pk):
        patient = (
            Patient.objects
            .prefetch_related(
                'appointments__doctor',
                'appointments__prescription__items__medicine',
                'appointments__lab_order__items__test',
            )
            .filter(pk=pk)
            .first()
        )
        if not patient:
            return Response({'detail': 'Patient not found'}, status=404)

        serializer = PatientHistorySerializer(patient)
        return Response(serializer.data)
```

### Serializer

```python
# patients/serializers.py

class PatientHistorySerializer(serializers.ModelSerializer):
    appointments = AppointmentDetailSerializer(many=True, read_only=True)

    class Meta:
        model = Patient
        fields = [
            'id', 'full_name', 'phone', 'gender',
            'birth_date', 'blood_group', 'appointments'
        ]
```

Bu yerda `appointments` — reverse relation. `Appointment` modelida `related_name='appointments'` qilib qo'ygan bo'lsangiz, shunday ishlatasiz.

### ⚠️ Eng katta xato

`prefetch_related` ishlatmaslik. Agar patient'da 20 ta appointment bo'lsa, har biri uchun alohida doctor, prescription, items — JAMI **100+ SQL query** bo'lishi mumkin. `prefetch_related` bilan — 4-5 ta query.

### Buni qanday tekshirish?

`settings.py`'ga qo'shing (faqat development uchun):

```python
LOGGING = {
    'version': 1,
    'loggers': {
        'django.db.backends': {
            'level': 'DEBUG',
            'handlers': ['console'],
        },
    },
    'handlers': {
        'console': {'class': 'logging.StreamHandler'},
    },
}
```

Endi har bir SQL query terminalda ko'rinadi. Sonini sanang — 5 tadan oshmasin.

### 🎓 Bu modulda mustahkamlandi

- ✅ `choices` (data integrity)
- ✅ **Nested serializer** (reverse relation orqali)
- ✅ `prefetch_related` — chuqur level (`__` orqali)
- ✅ Catta API response'ni to'g'ri tashkil qilish

---

## 4. Appointments app — CORE

> Bu — projectning **yuragi**. Vaqtingizning 40% shu modulga ketadi va shunday bo'lishi kerak.

### Model

```python
# appointments/models.py

class Appointment(models.Model):
    STATUS_CHOICES = [
        ('PENDING', 'Pending'),
        ('CONFIRMED', 'Confirmed'),
        ('COMPLETED', 'Completed'),
        ('CANCELLED', 'Cancelled'),
    ]

    patient = models.ForeignKey(
        'patients.Patient',
        on_delete=models.PROTECT,
        related_name='appointments'
    )
    doctor = models.ForeignKey(
        'doctors.Doctor',
        on_delete=models.PROTECT,
        related_name='appointments'
    )
    appointment_time = models.DateTimeField()
    status = models.CharField(
        max_length=20,
        choices=STATUS_CHOICES,
        default='PENDING'
    )
    notes = models.TextField(blank=True)
    created_at = models.DateTimeField(auto_now_add=True)

    class Meta:
        ordering = ['-appointment_time']
```

### 🎯 Nima uchun `'patients.Patient'` string sifatida?

Circular import'dan qochish uchun. App boshqa appdagi modelni `string` sifatida `'app_name.ModelName'` orqali ham bog'lay oladi.

### Eng muhim qism — Double booking validation

```python
# appointments/serializers.py

class AppointmentCreateSerializer(serializers.ModelSerializer):
    class Meta:
        model = Appointment
        fields = ['id', 'patient', 'doctor', 'appointment_time', 'notes']

    def validate_appointment_time(self, value):
        if value < timezone.now():
            raise serializers.ValidationError(
                "Appointment vaqti o'tmishda bo'lishi mumkin emas."
            )
        return value

    def validate(self, attrs):
        doctor = attrs['doctor']
        appointment_time = attrs['appointment_time']

        # Doctor shu vaqtda band bo'lmasligi kerak
        # Tahminan ±30 daqiqa oralig'ida boshqa appointment bo'lmasin
        clash = Appointment.objects.filter(
            doctor=doctor,
            appointment_time=appointment_time,
            status__in=['PENDING', 'CONFIRMED']
        )

        # Update bo'lsa, o'zini chiqarib tashlaymiz
        if self.instance:
            clash = clash.exclude(pk=self.instance.pk)

        if clash.exists():
            raise serializers.ValidationError({
                'appointment_time': "Doctor bu vaqtda band."
            })

        return attrs
```

### 🚨 Bu yerdagi 3 ta MUHIM detal

**1. `validate_<field>` vs `validate`**

| Qaysi   | Qachon ishlatamiz                                  |
| ------- | -------------------------------------------------- |
| `validate_field` | Faqat bitta field tekshirilsa            |
| `validate`       | Bir nechta field birga tekshirilsa       |

Double booking — `doctor` + `appointment_time` ikkalasiga ham qaragan. Demak `validate` (object-level).

**2. `self.instance` nima?**

`Update` paytida (PUT) — `self.instance` mavjud. `Create` paytida (POST) — `None`. Bu yordamida update'da o'zini "band" deb hisoblamaslik.

**3. `status__in=['PENDING', 'CONFIRMED']`**

`CANCELLED` yoki `COMPLETED` appointmentlar bo'sh vaqt hisoblanadi. Faqat aktiv appointmentlarni tekshiramiz.

### View

```python
# appointments/views.py

class AppointmentListCreateView(APIView):
    def get(self, request):
        qs = Appointment.objects.select_related('patient', 'doctor').all()

        # Filtering
        status_param = request.query_params.get('status')
        if status_param:
            qs = qs.filter(status=status_param)

        doctor_id = request.query_params.get('doctor')
        if doctor_id:
            qs = qs.filter(doctor_id=doctor_id)

        serializer = AppointmentDetailSerializer(qs, many=True)
        return Response(serializer.data)

    def post(self, request):
        serializer = AppointmentCreateSerializer(data=request.data)
        serializer.is_valid(raise_exception=True)
        serializer.save()
        return Response(serializer.data, status=201)
```

### Custom action endpointlar

```python
class AppointmentCancelView(APIView):
    def post(self, request, pk):
        appointment = get_object_or_404(Appointment, pk=pk)

        if appointment.status == 'COMPLETED':
            return Response(
                {'detail': "Tugagan appointmentni cancel qilib bo'lmaydi."},
                status=400
            )
        if appointment.status == 'CANCELLED':
            return Response(
                {'detail': "Allaqachon cancel qilingan."},
                status=400
            )

        appointment.status = 'CANCELLED'
        appointment.save()
        return Response(AppointmentDetailSerializer(appointment).data)


class AppointmentCompleteView(APIView):
    def post(self, request, pk):
        appointment = get_object_or_404(Appointment, pk=pk)

        if appointment.status != 'CONFIRMED':
            return Response(
                {'detail': "Faqat CONFIRMED appointmentni complete qilish mumkin."},
                status=400
            )

        appointment.status = 'COMPLETED'
        appointment.save()
        return Response(AppointmentDetailSerializer(appointment).data)
```

### 🎯 Status transition qoidalari

Bu bizning **business logic**'imiz:

```
PENDING    ─→ CONFIRMED  ─→ COMPLETED
   │              │
   └──→ CANCELLED ◄┘
```

- `COMPLETED` dan boshqa hech qaysi holatga o'tmaydi (yakuniy)
- `CANCELLED` dan ham orqaga qaytarib bo'lmaydi
- `COMPLETED` faqat `CONFIRMED` dan kelishi mumkin

### `today/` endpoint

```python
class AppointmentTodayView(APIView):
    def get(self, request):
        today = timezone.localdate()
        qs = Appointment.objects.select_related('patient', 'doctor').filter(
            appointment_time__date=today
        )
        serializer = AppointmentDetailSerializer(qs, many=True)
        return Response(serializer.data)
```

### URL routing

```python
# appointments/urls.py

urlpatterns = [
    path('appointments/', AppointmentListCreateView.as_view()),
    path('appointments/today/', AppointmentTodayView.as_view()),
    path('appointments/<int:pk>/', AppointmentDetailView.as_view()),
    path('appointments/<int:pk>/cancel/', AppointmentCancelView.as_view()),
    path('appointments/<int:pk>/complete/', AppointmentCompleteView.as_view()),
]
```

### ⚠️ URL tartibiga e'tibor bering

`appointments/today/` ni `appointments/<int:pk>/` dan **oldin** qo'ying. Aks holda Django `today` so'zini `pk` deb o'qishga urinadi va `ValueError` chiqaradi.

### 🎓 Bu modulda mustahkamlandi

- ✅ `validate()` — object-level validation
- ✅ Business logic in view
- ✅ Status transition (state machine)
- ✅ Custom action endpointlar
- ✅ Query parameter filtering (multiple)
- ✅ `self.instance` create vs update farqi

---

## 5. Prescriptions app

### Maqsad

**Writable nested serializer** — bu yerda eng qiyin DRF mavzusini o'rganasiz.

### Models

```python
# prescriptions/models.py

class Medicine(models.Model):
    name = models.CharField(max_length=150)
    manufacturer = models.CharField(max_length=150)
    price = models.DecimalField(max_digits=10, decimal_places=2)


class Prescription(models.Model):
    appointment = models.OneToOneField(
        'appointments.Appointment',
        on_delete=models.PROTECT,
        related_name='prescription'
    )
    diagnosis = models.CharField(max_length=255)
    advice = models.TextField(blank=True)
    created_at = models.DateTimeField(auto_now_add=True)


class PrescriptionItem(models.Model):
    prescription = models.ForeignKey(
        Prescription,
        on_delete=models.CASCADE,
        related_name='items'
    )
    medicine = models.ForeignKey(Medicine, on_delete=models.PROTECT)
    dosage = models.CharField(max_length=100)
    duration_days = models.PositiveIntegerField()
    instruction = models.CharField(max_length=255, blank=True)
```

### 🎯 Diqqat: Nima uchun `OneToOneField`?

Bitta appointment'ga faqat **bitta** prescription. Bir necha marta retsept yozish kerak emas — bitta appointment'da bitta retsept.

### 🎯 `CASCADE` vs `PROTECT`

- `Prescription` o'chsa, uning `items` ham o'chsin → `CASCADE`
- `Medicine` o'chmasin agar u biron prescription'da ishlatilgan bo'lsa → `PROTECT`

### Writable Nested Serializer

```python
# prescriptions/serializers.py

class PrescriptionItemSerializer(serializers.ModelSerializer):
    medicine_name = serializers.CharField(source='medicine.name', read_only=True)

    class Meta:
        model = PrescriptionItem
        fields = [
            'id', 'medicine', 'medicine_name',
            'dosage', 'duration_days', 'instruction'
        ]


class PrescriptionSerializer(serializers.ModelSerializer):
    items = PrescriptionItemSerializer(many=True)

    class Meta:
        model = Prescription
        fields = ['id', 'appointment', 'diagnosis', 'advice', 'items', 'created_at']
        read_only_fields = ['appointment', 'created_at']

    def create(self, validated_data):
        items_data = validated_data.pop('items')

        with transaction.atomic():
            prescription = Prescription.objects.create(**validated_data)
            for item in items_data:
                PrescriptionItem.objects.create(prescription=prescription, **item)

        return prescription
```

### 🚨 Bu yerda 3 ta MUHIM detal

**1. `items_data = validated_data.pop('items')`**

`Prescription.objects.create()` chaqirilganda `items` kerak emas — u boshqa modelga tegishli. Shuning uchun uni alohida olib qo'yamiz.

**2. `transaction.atomic()` nima uchun?**

Agar 3 ta item yaratayotganda 2-tasi xato bersa — birinchisi DBga yozib bo'lingan bo'ladi. Bu — yarim ishlangan ma'lumot. `transaction.atomic()` bilan: yoki **hammasi** yoziladi, yoki **hech qaysisi**.

```python
from django.db import transaction
```

**3. `read_only_fields = ['appointment']`**

`appointment` URL'dan keladi (`/appointments/{id}/prescription/`), request body'dan emas. Shuning uchun read-only.

### View

```python
# prescriptions/views.py

class PrescriptionCreateView(APIView):
    def post(self, request, appointment_id):
        appointment = get_object_or_404(Appointment, pk=appointment_id)

        # 1-qoida: Faqat COMPLETED appointmentga retsept yoziladi
        if appointment.status != 'COMPLETED':
            return Response(
                {'detail': "Faqat tugagan appointmentga retsept yozish mumkin."},
                status=400
            )

        # 2-qoida: Bitta appointmentga faqat bitta prescription
        if hasattr(appointment, 'prescription'):
            return Response(
                {'detail': "Bu appointment uchun retsept allaqachon mavjud."},
                status=400
            )

        serializer = PrescriptionSerializer(data=request.data)
        serializer.is_valid(raise_exception=True)
        serializer.save(appointment=appointment)

        return Response(serializer.data, status=201)
```

### 🎯 `serializer.save(appointment=appointment)` nima?

`save()` ga qo'shimcha argument berib, uni `validated_data` ga qo'shamiz. Shu sabab `appointment` URL'dan kelyapti, request body'dan emas.

### 🎓 Bu modulda mustahkamlandi

- ✅ `OneToOneField` va `ForeignKey` to'g'ri tanlash
- ✅ **Writable nested serializer**
- ✅ `create()` override qilish
- ✅ `transaction.atomic()` — data integrity
- ✅ Business rule validation (faqat COMPLETED'ga)
- ✅ `serializer.save(extra=value)` pattern

---

## 6. Laboratory app

### Maqsad

`ManyToMany` through table + nested response.

### Models

```python
# laboratory/models.py

class LabTest(models.Model):
    name = models.CharField(max_length=150)
    price = models.DecimalField(max_digits=10, decimal_places=2)


class LabOrder(models.Model):
    appointment = models.ForeignKey(
        'appointments.Appointment',
        on_delete=models.PROTECT,
        related_name='lab_orders'
    )
    created_at = models.DateTimeField(auto_now_add=True)


class LabOrderItem(models.Model):
    lab_order = models.ForeignKey(
        LabOrder,
        on_delete=models.CASCADE,
        related_name='items'
    )
    test = models.ForeignKey(LabTest, on_delete=models.PROTECT)
    result = models.TextField(blank=True)
    is_done = models.BooleanField(default=False)
```

### 🎯 Nima uchun M2M emas, `through` model?

DRF darslarda `ManyToManyField` ko'rgansiz. Lekin agar har bir testni alohida `result`, `is_done` bilan saqlash kerak bo'lsa — `through` model orqali boramiz (`LabOrderItem`).

Bu — **production amaliyot**. Shunchaki `ManyToManyField` qo'yib qo'yib, keyin "voy, men har bir testning natijasini saqlay olmayman" — bu klassik xato.

### Serializer

```python
# laboratory/serializers.py

class LabOrderItemSerializer(serializers.ModelSerializer):
    test_name = serializers.CharField(source='test.name', read_only=True)

    class Meta:
        model = LabOrderItem
        fields = ['id', 'test', 'test_name', 'result', 'is_done']


class LabOrderCreateSerializer(serializers.Serializer):
    tests = serializers.ListField(
        child=serializers.IntegerField(),
        allow_empty=False
    )

    def validate_tests(self, value):
        existing = set(LabTest.objects.filter(id__in=value).values_list('id', flat=True))
        missing = set(value) - existing
        if missing:
            raise serializers.ValidationError(
                f"Quyidagi test ID'lari topilmadi: {sorted(missing)}"
            )
        return value


class LabOrderDetailSerializer(serializers.ModelSerializer):
    items = LabOrderItemSerializer(many=True, read_only=True)
    patient_name = serializers.CharField(
        source='appointment.patient.full_name',
        read_only=True
    )

    class Meta:
        model = LabOrder
        fields = ['id', 'appointment', 'patient_name', 'items', 'created_at']
```

### 🎯 Nima uchun ikkita serializer?

- `LabOrderCreateSerializer` — POST uchun (input format). Foydalanuvchi faqat test ID'lari ro'yxatini yuboradi.
- `LabOrderDetailSerializer` — response uchun (output format). To'liq ma'lumot bilan qaytaramiz.

Bu **production amaliyot** — `input` va `output` shakli har doim ham bir xil emas.

### View

```python
# laboratory/views.py

class LabOrderCreateView(APIView):
    def post(self, request, appointment_id):
        appointment = get_object_or_404(Appointment, pk=appointment_id)

        serializer = LabOrderCreateSerializer(data=request.data)
        serializer.is_valid(raise_exception=True)
        test_ids = serializer.validated_data['tests']

        with transaction.atomic():
            order = LabOrder.objects.create(appointment=appointment)
            LabOrderItem.objects.bulk_create([
                LabOrderItem(lab_order=order, test_id=tid)
                for tid in test_ids
            ])

        # Response'da boshqa serializer ishlatamiz
        order = LabOrder.objects.select_related(
            'appointment__patient'
        ).prefetch_related('items__test').get(pk=order.pk)

        return Response(LabOrderDetailSerializer(order).data, status=201)
```

### 🎯 `bulk_create` nima uchun?

Agar 5 ta test yuborilsa va biz `for` loopda `LabOrderItem.objects.create(...)` chaqirsak — **5 ta SQL INSERT** bo'ladi. `bulk_create` bilan — **bitta INSERT**.

Kichik miqdorda farq sezilmaydi, lekin amaliyotni hozirdan o'rgangan ma'qul.

### 🎓 Bu modulda mustahkamlandi

- ✅ `through` model (M2M alternative)
- ✅ Alohida input/output serializerlar
- ✅ `ListField` — array validation
- ✅ `bulk_create` — performance
- ✅ Custom validation (`validate_tests`)

---

## 7. Dashboard app

### Maqsad

ORM aggregation, annotate, ordering.

### Today's Statistics

```python
# dashboard/views.py

class TodayStatsView(APIView):
    def get(self, request):
        today = timezone.localdate()

        today_appointments = Appointment.objects.filter(
            appointment_time__date=today
        )

        stats = today_appointments.aggregate(
            total=Count('id'),
            completed=Count('id', filter=Q(status='COMPLETED')),
            cancelled=Count('id', filter=Q(status='CANCELLED')),
            pending=Count('id', filter=Q(status='PENDING')),
        )

        unique_patients = today_appointments.values('patient').distinct().count()

        return Response({
            'date': today,
            'appointments': stats['total'],
            'completed': stats['completed'],
            'cancelled': stats['cancelled'],
            'pending': stats['pending'],
            'unique_patients': unique_patients,
        })
```

### 🎯 `Count('id', filter=Q(...))` nima?

Pythonda 4 ta alohida query yozish o'rniga **bitta** SQL query bilan barcha statistikani olamiz. Bu **professional** ORM ishlatish.

### Top Doctors

```python
class TopDoctorsView(APIView):
    def get(self, request):
        doctors = (
            Doctor.objects
            .annotate(
                completed_count=Count(
                    'appointments',
                    filter=Q(appointments__status='COMPLETED')
                )
            )
            .order_by('-completed_count')[:5]
        )

        data = [
            {
                'id': d.id,
                'full_name': d.full_name,
                'completed_appointments': d.completed_count,
            }
            for d in doctors
        ]
        return Response(data)
```

### 🎓 Bu modulda mustahkamlandi

- ✅ `aggregate()` — umumiy statistika
- ✅ `annotate()` — har bir obyekt uchun statistika
- ✅ `Count` + `Q` filter
- ✅ `order_by` + slice
- ✅ `values().distinct().count()`

---

## 8. Yakuniy tekshirish

Project tugadi deb hisoblashdan oldin shu checklist'ni o'ting:

### ✅ Funksionallik

- [ ] Doctor yarata olaman, list, detail, filter ishlaydi
- [ ] Patient yarata olaman, history endpoint to'liq ishlaydi
- [ ] Appointment yaratganda double booking validation ishlaydi
- [ ] Cancel va Complete endpointlar status'ni to'g'ri o'zgartiradi
- [ ] Faqat COMPLETED appointment uchun prescription yozish mumkin
- [ ] Bitta appointmentga ikki marta prescription yozib bo'lmaydi
- [ ] Lab order yaratish ishlaydi va nested response qaytaradi
- [ ] Dashboard endpointlari to'g'ri raqamlar beradi

### ✅ Code Quality

- [ ] Har bir app o'z `serializers.py`, `views.py`, `urls.py` ga ega
- [ ] `select_related` va `prefetch_related` ishlatilgan
- [ ] Validation logiclar serializer ichida, view ichida emas
- [ ] Business rules (status transition, double booking) tushunarli yozilgan
- [ ] HTTP status code'lar to'g'ri (201 create, 204 delete, 400 validation, 404 not found)

### ✅ Manual test

Postman yoki cURL bilan har bir endpoint'ni sinab ko'ring:

1. `POST /api/doctors/` — yangi doctor
2. `POST /api/patients/` — yangi patient
3. `POST /api/appointments/` — appointment yarating
4. Yana shu vaqtga **xuddi shu doctor** uchun appointment yarating — **xato kelishi kerak**
5. `POST /api/appointments/{id}/complete/` — completed
6. `POST /api/appointments/{id}/prescription/` — retsept yozing
7. Xuddi shu appointment uchun yana retsept yozing — **xato kelishi kerak**
8. `GET /api/patients/{id}/history/` — hammasi ko'rinadimi?

Agar hammasi ishlasa — siz **DRF'da intermediate darajadagi backend developer**siz. 🎉

---

## 💡 Umumiy maslahatlar

1. **Modulma-modul yozing.** `doctors` to'liq ishlamaguncha `appointments`'ga o'tmang.

2. **Har bir endpoint yozilgandan keyin sinab ko'ring.** Postman'da darhol test qiling. "Hammasini yozib bo'lib keyin sinaymiz" — bu xato.

3. **Migration'lar bilan ehtiyot bo'ling.** Model o'zgartirsangiz darhol `makemigrations` va `migrate`. Eski migration'larni o'chirib qo'ymang.

4. **Error message'lar uzbekcha bo'lsin yoki inglizcha — lekin BIR XIL.** Aralashtirmang.

5. **N+1 problem'ni har doim eslab turing.** Listda har bir element uchun related model'ga murojaat bo'lsa — `select_related` / `prefetch_related` qo'ying.

6. **`raise_exception=True` ishlatib turing.** Validation xatosini qo'lda return qilish o'rniga DRF o'zi 400 qaytaradi.

7. **Imports'ni toza saqlang.** Har bir faylning yuqorisida tartibli imports.

Omad, kelajakdagi backend developer! 🚀
