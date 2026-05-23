```Markdown
# API Specifications - Module 1: Specimen Registration (LAB-REG)

يوثق هذا الملف نقاط الاتصال (Endpoints) الخاصة بالموديول الأول لنظام MediChain، والتي تتيح تسجيل العينات، فحص شروط المريض الطبية آلياً، وتوليد الباركود المركب الفريد.

---

## 1. Register a New Specimen (تسجيل عينة جديدة وتحقق آلي)

* **Endpoint:** `/api/specimens/register`
* **Method:** `POST`
* **Description:** يستقبل بيانات العينة من الواجهة، ويقوم بالتحقق الآلي من شرط الصيام للتحاليل المشروطة وتثبيت وقت السحب قبل الحفظ.

### Request Body (JSON)
| Field | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| `patient_id` | String (UUID) | Yes | معرف المريض صاحب العينة |
| `analysis_type_id` | String (UUID) | Yes | معرف نوع التحليل المطلوب |
| `collected_at` | String (ISO 8601) | Yes | وقت وسحب العينة الفعلي بدقة ثانية |
| `is_patient_fasting` | Boolean | Yes | هل المريض صائم فعلياً وقت السحب؟ |

### Success Response (`201 Created`)
```json
{
  "success": true,
  "message": "Specimen registered and routed successfully.",
  "data": {
    "specimen_id": "spc-98213-xyz",
    "status": "REGISTERED",
    "destination_lab_id": "lab-central-01",
    "collected_at": "2026-05-23T21:00:00Z"
  }
}
```
