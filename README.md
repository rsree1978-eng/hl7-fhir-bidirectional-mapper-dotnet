# HL7 v2 to FHIR Mapper – Complete .NET Core Project (.NET 8)

## 1. Project Overview

This project converts **HL7 v2.x messages (ADT, ORM, ORU)** into **FHIR R4 resources** using **ASP.NET Core (.NET 8)**. 

**Key Goals**

* Parse HL7 v2 messages
* Map segments → FHIR resources
* Validate against FHIR R4
* Expose REST APIs
* Optional: MLLP listener for real HL7 feeds

---

## 2. Tech Stack

* **.NET 8 / ASP.NET Core Web API**
* **HL7 Parser**: NHapi (HL7 v2)
* **FHIR SDK**: Hl7.Fhir.R4
* **JSON Serialization**: System.Text.Json
* **Validation**: FHIR Validator
* **Optional**: Mirth Connect, Docker, SQL Server

---

## 3. High-Level Architecture

```
HL7 v2 Message
      ↓
NHapi Parser
      ↓
HL7 Domain Model
      ↓
Mapping Layer
      ↓
FHIR Resources (Patient, Encounter, Observation)
      ↓
FHIR Validation
      ↓
REST Response / FHIR Server
```

---

## 4. Project Structure

```
HL7ToFHIR/
│
├── HL7ToFHIR.API
│   ├── Controllers
│   │   └── Hl7Controller.cs
│   ├── Services
│   │   ├── Hl7ParserService.cs
│   │   ├── FhirMappingService.cs
│   │   └── FhirValidationService.cs
│   ├── Models
│   │   └── Hl7MessageDto.cs
│   └── Program.cs
│
├── HL7ToFHIR.Mappings
│   ├── PatientMapper.cs
│   ├── EncounterMapper.cs
│   └── ObservationMapper.cs
│
├── HL7ToFHIR.Common
│   └── Constants.cs
│
└── HL7ToFHIR.Tests
    └── MappingTests.cs
```

---

## 5. NuGet Packages

```
dotnet add package NHapi
 dotnet add package Hl7.Fhir.R4
 dotnet add package Hl7.Fhir.Validation
```

---

## 6. HL7 Input Example (ADT^A01)

```
MSH|^~\&|HIS|RIH|EKG|EKG|202401011200||ADT^A01|MSG00001|P|2.5
PID|1||12345^^^Hospital||Ravi^Kumar||19800101|M
PV1|1|I|WARD^101^1
```

---

## 7. API Controller

**POST /api/hl7/convert**

* Input: HL7 v2 message (raw text)
* Output: FHIR Bundle (JSON)

---

## 8. HL7 Parsing Service

**Responsibility**: Convert raw HL7 → NHapi message object

Key Concepts:

* PipeParser
* Terser for field access

---

## 9. Patient Mapping (PID → Patient)

| HL7 Field | FHIR Field         |
| --------- | ------------------ |
| PID-3     | Patient.identifier |
| PID-5     | Patient.name       |
| PID-7     | Patient.birthDate  |
| PID-8     | Patient.gender     |

Mapping Rules:

* Gender: M → male, F → female
* DOB format: yyyyMMdd → ISO date

---

## 10. Encounter Mapping (PV1 → Encounter)

| HL7 Field | FHIR Field             |
| --------- | ---------------------- |
| PV1-2     | Encounter.class        |
| PV1-3     | Encounter.location     |
| PV1-44    | Encounter.period.start |

---

## 11. Observation Mapping (OBX → Observation)

| HL7 Field | FHIR Field                    |
| --------- | ----------------------------- |
| OBX-3     | Observation.code              |
| OBX-5     | Observation.value             |
| OBX-14    | Observation.effectiveDateTime |

---

## 12. Bundle Creation

All generated resources are wrapped into a **FHIR Bundle (type = collection)**:

* Patient
* Encounter
* Observations

---

## 13. Validation

FHIR validation ensures:

* Resource schema correctness
* Required fields present
* Terminology correctness (optional)

---

## 14. Sample Output

* `Bundle`

  * `Patient`
  * `Encounter`
  * `Observation`

Returned as **FHIR R4 JSON**.

---

## 15. Bidirectional Mapping (HL7 v2 ↔ FHIR)

This project supports **two-way interoperability**:

* **HL7 v2 → FHIR R4** (inbound clinical feeds)
* **FHIR R4 → HL7 v2** (legacy system compatibility)

### 15.1 HL7 v2 → FHIR (Already Covered)

Inbound HL7 messages (ADT, ORM, ORU) are parsed using **NHapi**, mapped into:

* Patient
* Encounter
* Observation

Wrapped inside a **FHIR Bundle (collection)**.

---

### 15.2 FHIR → HL7 v2 (NEW)

FHIR resources are converted back into HL7 v2 messages using:

* NHapi message builders
* Controlled segment population

Supported outbound messages:

* **FHIR Patient → ADT^A01**
* **FHIR Observation → ORU^R01**

---

## 16. FHIR → HL7 Mapping Rules

### Patient → PID Segment

| FHIR Field         | HL7 Field |
| ------------------ | --------- |
| Patient.identifier | PID-3     |
| Patient.name       | PID-5     |
| Patient.birthDate  | PID-7     |
| Patient.gender     | PID-8     |

Gender Mapping:

* male → M
* female → F
* other/unknown → U

---

### Encounter → PV1 Segment

| FHIR Field             | HL7 Field |
| ---------------------- | --------- |
| Encounter.class        | PV1-2     |
| Encounter.location     | PV1-3     |
| Encounter.period.start | PV1-44    |

---

### Observation → OBX Segment

| FHIR Field                    | HL7 Field |
| ----------------------------- | --------- |
| Observation.code              | OBX-3     |
| Observation.value             | OBX-5     |
| Observation.effectiveDateTime | OBX-14    |

---

## 17. Outbound HL7 Message Flow

```
FHIR JSON
   ↓
FHIR Parsing (Hl7.Fhir.R4)
   ↓
FHIR Domain Model
   ↓
HL7 Mapping Layer
   ↓
NHapi Message Builder
   ↓
HL7 v2 Message
```

---

## 18. API Endpoints (Bidirectional)

### HL7 → FHIR

`POST /api/hl7/convert-to-fhir`

* Input: Raw HL7 v2
* Output: FHIR Bundle

### FHIR → HL7

`POST /api/fhir/convert-to-hl7`

* Input: FHIR Bundle / Resource
* Output: HL7 v2 message

---
