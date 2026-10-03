# CityCare Hospital Management System (HMS)

A terminal-based Hospital Management System for **CityCare Hospital**, written in pure
Python 3.10+ (standard library only) with a clean object-oriented design.
All data lives in Python objects in memory: no database, files, GUI or third-party packages.

**Status:** Release 1 complete (`your-name-v1.0`). Releases 2 and 3 are planned in
[`docs/project-plan.md`](docs/project-plan.md).

## How to run

Run everything from this folder (`Capstone 0 - Git and Python/`):

```bash
python -m hms.main                          # start the application with sample data
python -m unittest discover -s tests -t .   # run all unit tests (standard library only)
```

Python 3.10 or newer is required. Currency is printed as `Rs`.

## What you can do (menu)

```
===== CityCare Hospital Management System =====
 1. Register patient (OPD / IPD)        8. Admit patient to ward
 2. Add doctor / nurse                  9. Discharge patient
 3. View all doctors                   10. Add prescription
 4. View all patients                  11. Generate bill
 5. Search patient by ID or name       12. View ward occupancy
 6. Book appointment                   13. View all bills (sorted by amount)
 7. Cancel appointment                 14. Complete appointment  (extra)
 0. Exit
```

Rules of the terminal: non-numeric input prints `Invalid choice`; every business error
prints `Error: <message>`; IDs work in any case (`p003` = `P003`); the program never crashes on bad input.

Sample data: 4 doctors (D004 is a senior doctor), 2 nurses, 1 receptionist, 3 wards
(General 10 beds, Private 5, ICU 4 - **ICU starts full**), 8 patients (P001-P008) and 3 appointments.

### Sample session

```
Enter choice: 6
Patient ID: P003
Doctor ID: D001
Available slots for Dr. Meera Rao (Cardiology): 10:00, 11:00, 15:00
Choose slot: 11:00
Appointment A004 booked for Anil Kumar with Dr. Meera Rao at 11:00.

Enter choice: 6
Patient ID: P005
Doctor ID: D001
Choose slot: 11:00
Error: Dr. Meera Rao is not available at 11:00.

Enter choice: 8
Patient ID: P002
Ward type (General/Private/ICU): ICU
Error: ICU ward is full (4/4 beds occupied).
```

Try this too: `9` -> `P001` -> `D004` (senior doctor approves discharge), then `11` -> `P001` to see an itemised IPD bill.

## Billing rules (as implemented)

| Patient | Formula |
|---|---|
| OutPatient | consultation fee + Rs 50 x medicine x day |
| InPatient | (days x ward rate) + doctor fee per visit + medicines + **5% service tax** |
| Age 60+ | 10% discount on the **final** amount (after tax), either type |

Ward rates: General Rs 1,000, Private Rs 3,500, ICU Rs 8,000 per day.
Worked example (IPD, 3 days General, one doctor visit of Rs 800, one medicine for 5 days, age 40):
(3 x 1000 + 800 + 250) = 4,050 + 5% tax = **Rs 4,252.50**; for a 70-year-old: 4,252.50 x 0.9 = **Rs 3,827.25**.

Assumptions: a same-day admission counts as 1 day; a doctor fee is charged per non-cancelled
appointment; items already billed are never charged again (appointments, prescriptions and
stay-days carry a billed marker, so interim bills only charge new days).

## Project layout

```
Capstone 0 - Git and Python/
├── README.md  CHANGELOG.md  .gitignore
├── docs/
│   ├── project-plan.md              milestones, sprints, dates, tags
│   └── sprint-notes/                sprint-01.md ... sprint-09.md
├── hms/
│   ├── main.py                      terminal menu only
│   ├── models/                      base, person, patient, staff, ward, medicine, ambulance
│   ├── services/                    hospital, appointment, billing
│   ├── exceptions/                  exceptions.py
│   └── data/                        seed_data.py
└── tests/                           unittest scripts (105 tests)
```

## Module -> design principles

| Module | Where | Principles used |
|---|---|---|
| 1 Core entities | `models/medicine.py`, `models/ward.py`, `models/base.py` | Classes and objects; **class attribute** (`CityCareEntity.hospital_name`) stored once and shared by all objects; encapsulated bed tracking |
| 2 Data protection | `models/person.py` | **Encapsulation**: private `__age` / `__phone`, `@property` getters/setters, validation with `ValueError`; class-level ID counters |
| 3 People hierarchy | `models/patient.py`, `models/staff.py` | **Inheritance**: single (`Patient` -> `OutPatient`/`InPatient`), multilevel (`Person` -> `Doctor` -> `SeniorDoctor`), `super().__init__()` |
| 4 Uniform behaviour | `get_details()`, `print_all()`, `calculate_bill()`, `models/ambulance.py` | **Polymorphism** and duck typing: `print_all()` has no `isinstance`; `Ambulance` is not a `Person` yet works with it; method overriding |
| 5 Contracts | `Person(ABC)`, `Billable(ABC)` | **Abstraction**: abstract base classes and `@abstractmethod`; `Person()` and `Patient()` cannot be instantiated |
| 6 Hospital operations | `services/hospital.py`, `appointment.py`, `billing.py`, `models/medicine.py` | **Composition**: `Hospital` owns everything; `Appointment` holds a patient + doctor; `Bill` holds line items; **Enum** for status |
| 7 Error handling | `exceptions/exceptions.py` | **Custom exception hierarchy** (`HospitalError` base); raised in `Hospital`, caught in the menu |
| 8 Usability | magic methods across models | `__str__`, `__repr__`, `__eq__`/`__hash__`, `__len__`, `__contains__`, `__lt__`, `__add__`; `@classmethod from_string`; `@staticmethod is_valid_phone` |
| 9 Terminal app | `main.py`, `data/seed_data.py` | Separation of concerns: menu only does I/O and calls `Hospital` methods |

### Design decisions worth knowing

- **No `isinstance` in business logic.** Rules that differ per type are methods on the objects
  (`check_can_admit()`, `check_can_discharge()`, `can_approve_discharge()`, `bill_line_items()`), so `Hospital` stays free of type checks.
- **Nothing half-done.** `admit_patient()` validates first and takes the bed last, so a failed admission never leaves the ward changed.
- **Sensitive data stays private.** Phone numbers are masked in `get_details()` and never appear in `repr()`.
- **Extras beyond the brief** (all documented): menu option 14 (complete an appointment, needed to make "cancel a completed appointment" reachable),
  extra `DoctorNotFoundError` / `AppointmentNotFoundError` / `WardNotFoundError` (all subclasses of `HospitalError`),
  `Receptionist`, and `Hospital.hire_doctor()` / `hire_nurse()`.

## Release 2 and 3 (planned)

Release 2 requires a deliberate redesign of Release 1: `OutPatient` / `InPatient` subclasses cannot model an OPD patient
who becomes IPD mid-visit, so the patient will **have a** `PatientStatus` instead of **being** a subclass. Reason for composition
over inheritance: an object cannot change its class at runtime, but it can swap the status object it holds, which keeps the same patient ID,
history and prescriptions. This section will be expanded with the actual change notes when Release 2 is built.
See the plan for milestones M6-M8.

## Git workflow used

- Work lives in `Projects/Individual/<your-name>/Week 1/Capstone 0 - Git and Python/`.
- First commit: folder structure + README + `.gitignore`; second commit: `docs/project-plan.md`.
- One branch per sprint item (`your-name/sprint-NN-topic`), merged to `main` through a pull request; never committed directly to `main` after the plan.
- Commit messages: `type(scope): summary` with types `feat`, `fix`, `refactor`, `docs`, `test`, `chore`.
- Tags: `your-name-sprint-01` ... `your-name-sprint-09`, release `your-name-v1.0`.
- `CHANGELOG.md` and one `docs/sprint-notes/sprint-NN.md` per sprint.
