# Changelog

All notable changes to the CityCare HMS are recorded here (newest first).

## [v1.0] / [Sprint 9] - 2026-10-03
### Added
- Constraint tests (permitted modules only, folder structure, no storage/UI libraries).
- Full README: how to run, billing rules, module-to-design-principle mapping, Git workflow.
### Release 1 summary
- All nine modules delivered across 5 milestones and 9 sprints; 105 unit tests passing.

## [Sprint 8] - 2026-10-03
### Added
- `hms/data/seed_data.py`: 4 doctors (1 senior), 2 nurses, 1 receptionist, 3 wards, 8 patients, 3 appointments, 1 prescription (ICU starts full).
- `hms/main.py`: menu-driven terminal application (options 1-13 from the brief, plus 14 "Complete appointment").
- `Hospital.hire_doctor()`, `hire_nurse()`, `create_patient_from_string()` so the menu never builds staff objects itself.
- Friendly handling of invalid menu input, every `HospitalError` and every `ValueError` (`Error: <message>`).

## [Sprint 7] - 2026-10-03
### Added
- `__str__` / `__repr__` for `Patient`, `Doctor`, `Appointment` (phone never appears in `repr`).
- `Person.__eq__` / `__hash__` by ID; `Ward.__len__`, `__contains__`, `__bool__`; `Bill.__lt__`, `__add__`, `__radd__`.
- `Patient.from_string("Ravi,34,M,9876543210,O+")` classmethod and `Hospital.is_valid_phone()` staticmethod.
### Changed
- `Hospital.bills_sorted()` now sorts with `Bill.__lt__`.

## [Sprint 6] - 2026-10-03
### Added
- `Hospital` class owning patients, doctors, nurses, wards, appointments and bills.
- Operations: `register_patient`, `book_appointment`, `cancel_appointment`, `complete_appointment`, `admit_patient`, `discharge_patient`, `add_prescription`, `generate_bill`, `search_patient`, `free_slots`, `ward_report`, `bills_sorted`.
- Errors raised from `Hospital`: `PatientNotFoundError`, `DoctorNotAvailableError`, `WardFullError`, `InvalidOperationError` (+ doctor/appointment/ward not-found errors).
- IDs are accepted in any case.

## [Sprint 5] - 2026-10-03
### Added
- `Appointment` (patient, doctor, slot, `AppointmentStatus` enum SCHEDULED/COMPLETED/CANCELLED) with guarded `complete()` / `cancel()`.
- `Prescription` (doctor, patient, list of `Medicine`) and `Bill` / `BillLineItem` (bill id, patient, line items, total, receipt).

## [Sprint 4] - 2026-10-03
### Added
- `Person` is now an abstract base class (`get_details()`, `get_role()`); `Billable` abstract class with `calculate_bill()`.
- Per-class `get_details()` for patients, doctors, nurses, receptionists and `Ambulance`; `print_all()` works on any mix without `isinstance`.
- Billing rules: OPD (fee + Rs 50/medicine/day), IPD (ward days + doctor fee + medicines + 5% tax), 10% senior discount applied after tax.
### Changed
- Phone numbers are masked in `get_details()` output.

## [Sprint 3] - 2026-10-03
### Added
- `Patient` base class (blood group, medical history, prescriptions, appointments) with `OutPatient` and `InPatient` (ward, bed, admission date).
- `Doctor` (specialisation enum, fee, validated slots), `SeniorDoctor` (can approve discharges), `Nurse` (shift enum, ward) and `Receptionist`.
- Admission/discharge rule checks on patients (outpatients cannot be admitted or discharged).

## [Sprint 2] - 2026-10-03
### Added
- `Person` with private `__age` / `__phone` exposed via validated properties (age 0-120, phone exactly 10 digits, `ValueError` with clear messages).
- Automatic IDs from a class-level counter (`P001`, `D001`, `N001` ...), not consumed when validation fails.
- Phone masking helper for safe display.

## [Sprint 1] - 2026-10-03
### Added
- Exception hierarchy: `HospitalError` and its subclasses.
- `CityCareEntity` base class holding the shared `hospital_name`.
- `Medicine` (name, dosage, days, Rs 50/day charge) and `Ward` (beds, `available_beds()`, `is_full()`).
- Unit tests for the above.
