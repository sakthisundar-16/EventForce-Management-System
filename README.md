# 🎯 EventForce Management System — Salesforce CRM

![Platform](https://img.shields.io/badge/Platform-Salesforce%20Lightning-00A1E0?style=for-the-badge&logo=salesforce)
![Backend](https://img.shields.io/badge/Backend-Apex%20%26%20Flows-blue?style=for-the-badge)
![Test Coverage](https://img.shields.io/badge/Code%20Coverage-100%25-brightgreen?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Completed-success?style=for-the-badge)

## 📌 1. Project Overview
**EventForce Management System** is an end-to-end Salesforce CRM application designed to help event planning agencies manage **client bookings, vendor coordination, venue reservations, and post-event feedback collection** within a unified platform. 

The system eliminates manual spreadsheet tracking by automating event scheduling conflict checks, cancellation approvals, and automated client email reminders, while delivering real-time operational insights via Lightning Reports and Dashboards.

---

## 🏗️ 2. Requirement Analysis & Planning (Epic 1)

### Functional Scope
- Centralized management of Venues, Vendors, Client Event Bookings, and Customer Feedback.
- Automated conflict detection to prevent double-booking venues on overlapping dates/times.
- Multi-step approval workflow for handling event cancellation requests.
- Automated 24-hour prior event reminder emails and post-event feedback survey notifications.
- Executive dashboards tracking venue utilization, revenue metrics, and customer satisfaction scores.

### Stakeholder Mapping
| Stakeholder Role | Access Level | Core Responsibilities |
| :--- | :--- | :--- |
| **System Administrator** | Full System Access | Manages org configuration, security, deployment, and user provisioning. |
| **Event Manager** | Read/Write/Approve | Reviews and approves/rejects event cancellation requests; monitors executive dashboards. |
| **Event Planner** | Create/Read/Edit | Creates client bookings, reserves venues, assigns vendors, and tracks event execution. |
| **Venue / Vendor Coordinator** | Read/Edit (Venues & Vendors) | Updates venue availability, capacity, pricing, and vendor contract statuses. |

---

## 🗄️ 3. Data Model & Architecture (Epic 2)

### Custom Objects & Schema
| Object Label | API Name | Record Name Type | Relationships & Key Custom Fields |
| :--- | :--- | :--- | :--- |
| **Venue** | `Venue__c` | `Text` (Venue Name) | • `Capacity__c` (Number)<br>• `Location_Address__c` (Text Area)<br>• `Cost_Per_Day__c` (Currency)<br>• `Availability_Status__c` (Picklist: *Available, Under Maintenance, Booked*) |
| **Vendor** | `Vendor__c` | `Text` (Vendor Name) | • `Service_Category__c` (Picklist: *Catering, Decor, AV/Sound, Photography, Security*)<br>• `Contact_Email__c` (Email)<br>• `Phone__c` (Phone)<br>• `Rate__c` (Currency) |
| **Event Booking** | `Event_Booking__c` | `Auto Number` (`EVT-{0000}`) | • `Event_Name__c` (Text - Required)<br>• `Client_Contact__c` (Lookup → `Contact`)<br>• `Venue__c` (Lookup → `Venue__c`)<br>• `Event_Start_Date__c` (Date/Time)<br>• `Event_End_Date__c` (Date/Time)<br>• `Expected_Attendees__c` (Number)<br>• `Status__c` (Picklist: *Draft, Scheduled, Confirmed, Cancellation Requested, Cancelled, Completed*)<br>• `Total_Budget__c` (Currency) |
| **Event Vendor** *(Junction)* | `Event_Vendor__c` | `Auto Number` (`EV-{0000}`) | • `Event_Booking__c` (Master-Detail → `Event_Booking__c`)<br>• `Vendor__c` (Master-Detail → `Vendor__c`)<br>• `Contract_Amount__c` (Currency)<br>• `Assignment_Status__c` (Picklist: *Assigned, Confirmed, Completed*) |
| **Event Feedback** | `Event_Feedback__c` | `Auto Number` (`FB-{0000}`) | • `Event_Booking__c` (Master-Detail → `Event_Booking__c`)<br>• `Client__c` (Lookup → `Contact`)<br>• `Rating__c` (Picklist: *1 to 5 Stars*)<br>• `Comments__c` (Long Text Area) |

---

## ⚙️ 4. Backend Development & Process Automation

### A. Data Validation Rules (`Event_Booking__c`)
1. **`Future_Start_Date_Required`**
   - **Formula:** `ISNEW() && Event_Start_Date__c < NOW()`
   - **Error Message:** *"Event Start Date must be scheduled in the future."*
2. **`Valid_Event_Duration`**
   - **Formula:** `Event_End_Date__c <= Event_Start_Date__c`
   - **Error Message:** *"Event End Date/Time must be later than the Event Start Date/Time."*
3. **`Check_Venue_Capacity`**
   - **Formula:** `Expected_Attendees__c > Venue__r.Capacity__c`
   - **Error Message:** *"Expected attendees cannot exceed the selected Venue's maximum capacity."*

### B. Apex Trigger & Handler (Venue Double-Booking Prevention)
- **`EventBookingTrigger.trigger`**: Fires on `before insert` and `before update` of `Event_Booking__c`.
- **`EventBookingHandler.cls`**: Bulkified Apex handler that queries active bookings for the requested `Venue__c` and blocks overlapping time slots (`Start_A < End_B && End_A > Start_B`) using `addError()`.

### C. Declarative Automation (Flows & Approval Process)
- **Cancellation Approval Process:** Triggered automatically when `Status__c` is set to `Cancellation Requested`. Routes to the Event Manager for approval. Upon approval, updates `Status__c` to `Cancelled`; upon rejection, reverts `Status__c` to `Confirmed`.
- **Client Reminder Scheduled Flow:** Record-Triggered Flow on `Event_Booking__c` with a scheduled path running **1 Day Before `Event_Start_Date__c`** to dispatch an automated reminder email to `Client_Contact__c`.
- **Post-Event Feedback Flow:** Record-Triggered Flow that triggers when `Status__c` changes to `Completed`, sending a feedback survey link to the client.

---

## 🎨 5. UI/UX Development & Analytics (Epic 3)

### Lightning Application & Page Customizations
- **Lightning App:** `EventForce Management` configured with branded utility bar and navigation tabs (`Home`, `Event Bookings`, `Venues`, `Vendors`, `Event Vendors`, `Event Feedbacks`, `Reports`, `Dashboards`).
- **Lightning Record Pages:** Upgraded `Event_Booking__c` record page using **Dynamic Forms** and conditional visibility (displaying Cancellation Reason fields only when `Status__c = 'Cancellation Requested'`).
- **Compact Layouts:** Configured key summary fields (`Event_Name__c`, `Venue__c`, `Event_Start_Date__c`, `Status__c`) on record highlights panels.

### Reports & Executive Dashboard
- **Events by Status Report:** Summary report grouping bookings by `Status__c`.
- **Venue Utilization & Revenue Report:** Summary report analyzing total bookings and `Total_Budget__c` per `Venue__c`.
- **Client Satisfaction Report:** Summary report tracking average client ratings from `Event_Feedback__c`.
- **EventForce Executive Dashboard:** Interactive Lightning Dashboard combining Donut, Horizontal Bar, and Gauge components for real-time KPI monitoring.

---

## 🔐 6. Data Migration, Testing & Security (Epic 4)

### Security Model
- **Organization-Wide Defaults (OWD):**
  - `Event_Booking__c` & `Event_Feedback__c`: **Private**
  - `Venue__c` & `Vendor__c`: **Public Read-Only**
- **Role Hierarchy:** `Executive Director` → `Event Manager` → `Event Planner`.
- **Permission Sets:** `Event_Planner_Access` grants CRED permissions on booking and vendor junction objects.

### Unit Testing
- **`EventBookingHandlerTest.cls`**: Verifies positive single bookings, bulkified non-overlapping bookings, and negative overlapping venue bookings using `Database.insert(record, false)` assertions. Achieves **100% code coverage**.

---

## 📂 7. Repository Structure (Epic 5)
```text
EventForce-Management-System/
├── force-app/
│   └── main/
│       └── default/
│           ├── classes/
│           │   ├── EventBookingHandler.cls
│           │   ├── EventBookingHandler.cls-meta.xml
│           │   ├── EventBookingHandlerTest.cls
│           │   └── EventBookingHandlerTest.cls-meta.xml
│           ├── triggers/
│           │   ├── EventBookingTrigger.trigger
│           │   └── EventBookingTrigger.trigger-meta.xml
│           └── objects/
│               ├── Venue__c/
│               ├── Vendor__c/
│               ├── Event_Booking__c/
│               ├── Event_Vendor__c/
│               └── Event_Feedback__c/
├── data/
│   ├── sample_venues.csv
│   └── sample_vendors.csv
└── README.md
