# Automated Room Reservation & Airtable Ingestion System

An event-driven **n8n** workflow that captures customer reservation submissions through a web form and persists guest details into a structured **Airtable** database.

**Category:** Business Process Automation / Form & Database Integration Workflows

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Problem Statement](#2-problem-statement)
3. [Proposed Solution](#3-proposed-solution)
4. [System Architecture](#4-system-architecture)
5. [Technologies & Tools](#5-technologies--tools)
6. [Workflow Execution](#6-workflow-execution)
7. [Core Features](#7-core-features)
8. [AI Implementation](#8-ai-implementation)
9. [Data Flow](#9-data-flow)
10. [Error Handling & Validation](#10-error-handling--validation)
11. [Challenges & Technical Solutions](#11-challenges--technical-solutions)
12. [Business Value](#12-business-value)
13. [Limitations & Future Improvements](#13-limitations--future-improvements)
14. [Technical Project Summary](#14-technical-project-summary)
15. [Setup & Deployment](#15-setup--deployment)

---

## 1. Project Overview

| Item | Details |
|------|---------|
| **Project Name** | Automated Room Reservation & Airtable Ingestion System |
| **Category** | Business Process Automation / Form & Database Integration Workflows |
| **Description** | An automated n8n workflow that captures customer reservation submissions via a web form and persists guest details into a structured Airtable database. |
| **Main Purpose** | Streamline room booking collection by capturing web form entries in real time and logging them directly into a cloud database without manual data entry. |
| **Target Users** | Hotels, bed & breakfasts, event spaces, property managers, and hospitality businesses requiring automated booking capture and database synchronization. |

---

## 2. Problem Statement

Manual processing of room reservations creates operational delays, increases the risk of transcription errors (e.g., mistyped customer names or incorrect room allocations), and requires staff to continuously monitor form responses. Without direct database integration, booking requests remain trapped in static form logs or inbox alerts, resulting in slower response times and potential double-booking errors.

---

## 3. Proposed Solution

The workflow uses an n8n **Form Trigger** node that hosts a user-facing reservation web form titled "Reservation". On submission, n8n parses the input and writes it to the database:

1. **Form Interface:** Captures the guest's name (`Your Name `) and desired room tier (`Choose room` dropdown: `single room`, `deluxe`, `suite`).
2. **Database Ingestion:** Maps the form inputs dynamically and creates a new record in a designated Airtable base (`sheet one`, Base ID: `<airtable-base-id>`) under `Table 1` (`<airtable-table-id>`).

---

## 4. System Architecture

### Overview

The workflow runs as a 1-to-1 sequential pipeline:

- **Trigger:** `n8n-nodes-base.formTrigger` (v2.6), acting as a web form endpoint at `/<form-path>`.
- **Destination:** `n8n-nodes-base.airtable` (v2.2), creating a database record via Personal Access Token authentication.

### Workflow Diagram

```mermaid
graph TD
    Guest[Guest / User] -->|Fills in form| Form[n8n Form Trigger: Reservation]
    Form -->|"Your Name", "Choose room"| Airtable[Airtable Node: Create Record]
    Airtable -->|REST API / PAT| DB[(Airtable Base: sheet one / Table 1)]
```

### ASCII View

```text
+---------------------------------------+
|  Guest / User Interface               |
|  n8n Hosted Web Form ("Reservation")  |
+---------------------------------------+
                    |
                    | Form Submission Event (POST)
                    v
+---------------------------------------+
|  n8n Form Trigger Node                |
|  - Ingests: "Your Name ", "Choose room"|
+---------------------------------------+
                    |
                    | Direct Data Payload Pipeline
                    v
+---------------------------------------+
|  Airtable API Node                    |
|  (Operation: Create Record)           |
+---------------------------------------+
                    |
                    | API Call / Bearer Token
                    v
+---------------------------------------+
|  Airtable Cloud Database              |
|  - Base: "sheet one"                  |
|  - Table: "Table 1"                   |
|  - Columns: Name, room                |
+---------------------------------------+
```

---

## 5. Technologies & Tools

| Technology | Role in Project | Why Used (Implementation Evidence) |
|------------|-----------------|------------------------------------|
| **n8n** | Workflow orchestration platform | Coordinates form hosting, input parsing, variable mapping, and credential routing. |
| **n8n Form Trigger** | Interactive ingestion interface | Serves a built-in web form with required-field validation and dropdown choices. |
| **Airtable API** | Cloud relational database / sink | Stores structured guest reservation records in base `<airtable-base-id>` (`Table 1`). |
| **Airtable Token Auth** | Authentication mechanism | Secures database interactions using an Airtable Personal Access Token (stored as an n8n credential). |
| **n8n Expressions** | Data mapping engine | Binds form fields to database columns: `={{ $json['Your Name '] }}`, `={{ $json['Choose room'] }}`. |

---

## 6. Workflow Execution

```text
[ Web Form Submission ] ──► ( On form submission ) ──► ( Airtable: Create a record )
```

### Step 1: Ingestion & Form Trigger

- **Component:** `n8n-nodes-base.formTrigger` (On form submission)
- **Operation:** Listens for user submissions on webhook path `<form-path>`.
- **Form schema:**
  - **Title:** "Reservation"
  - **Description:** "Please Choose your room :"
  - **Field 1:** Text input labeled `Your Name ` (required, placeholder: `ex: john wick`).
  - **Field 2:** Dropdown labeled `Choose room` (required, options: `single room`, `deluxe`, `suite`).
- **Input:** User input submitted via the web UI.
- **Output:** JSON object containing the parsed form fields under the root `$json`.
- **Necessity:** Primary entry point and interface for guest data collection.

### Step 2: Database Ingestion

- **Component:** `n8n-nodes-base.airtable` (Create a record)
- **Operation:** Sends a REST API request to Airtable to create a new record.
- **Target storage:** Base `<airtable-base-id>` (`sheet one`), table `<airtable-table-id>` (`Table 1`).
- **Field mapping:**

  | Airtable Column | Mapped Expression |
  |-----------------|-------------------|
  | `Name` | `={{ $json['Your Name '] }}` |
  | `room` | `={{ $json['Choose room'] }}` |

- **Output:** Airtable API confirmation payload containing the created record ID and timestamps.
- **Necessity:** Persists incoming bookings into a central database for hotel operations.

---

## 7. Core Features

- **Built-in Web Form Hosting:** A functional, no-code web form with no external hosting or third-party form builder required.
- **Required-Field Validation:** Enforces `Your Name ` and `Choose room` before any downstream logic runs.
- **Predefined Options:** The dropdown prevents invalid room tier selections.
- **Dynamic Field Extraction:** Maps n8n JSON keys directly into matching database columns.
- **Token-Based Authentication:** Uses Airtable API tokens for secure cloud database insertion.

---

## 8. AI Implementation

**Status:** Not implemented.

This workflow is purely deterministic. Data passes straight from a structured form into a database table, with no AI or natural language processing components.

---

## 9. Data Flow

```text
[ Guest User ]
      │ Enters name & selects room in web form
      ▼
[ Form Trigger Node ] ──► Parses submission into $json payload:
      │                    - $json['Your Name ']
      │                    - $json['Choose room']
      ▼
[ Airtable Node ] ──────► Maps parameters & sends request to Airtable REST API
      │
      ▼
[ Airtable Database ] ──► Persists record under Table 1 (Name, room)
```

---

## 10. Error Handling & Validation

- **Data Validation:** Enforced at the form level (`requiredField: true` on both inputs). The dropdown restricts input to `single room`, `deluxe`, or `suite`.
- **Type Safety:** `attemptToConvertTypes` and `convertFieldsToString` are set to `false`, keeping raw string values from the form.
- **Duplicate Detection:** Not implemented. Submitting the same name and room repeatedly creates duplicate Airtable records.
- **Fallback & Retry Logic:** Default n8n failure behavior (execution halts if the Airtable token is invalid or a network request fails).

---

## 11. Challenges & Technical Solutions

| Challenge | Solution |
|-----------|----------|
| Ensuring room selections match valid room types without manual cleaning. | Configured strict dropdown values (`single room`, `deluxe`, `suite`) in the Form Trigger node, eliminating free-text entries. |
| Handling field names with trailing whitespace (`Your Name `). | Used exact bracketed key expressions (`$json['Your Name ']`) in the Airtable mapping to prevent reference errors. |

---

## 12. Business Value

- **Process Automation:** Removes manual entry of room booking data, saving labor hours per reservation.
- **Data Consistency:** Prevents invalid room categories or incomplete records from reaching the reservations database.
- **Low Friction:** Direct web-to-database entry lets staff see new reservations right away.

---

## 13. Limitations & Future Improvements

### Current Limitations

- **Confirmation Notifications:** No email or SMS confirmation is sent to the guest.
- **Availability Checks:** Does not check Airtable for room availability before accepting a submission.
- **Duplicate Protection:** Does not check for existing active reservations under the same guest name.

### Planned Enhancements

- [ ] **Automated email confirmations:** Add a Gmail or SendGrid node after ingestion to send a booking summary to the guest.
- [ ] **Availability & inventory logic:** Add a lookup node before record creation to verify room availability.
- [ ] **Date pickers:** Extend the form with check-in and check-out date fields.

---

## 14. Technical Project Summary

An event-driven n8n workflow that streamlines hotel reservations. The system hosts a custom form, validates guest inputs (Name, room), and immediately writes structured records into an Airtable cloud database using Personal Access Token authentication.

---

## 15. Setup & Deployment

### Configuration Requirements

- **Form Trigger node:** Enabled, with an active form URL.
- **Airtable credentials:** A Personal Access Token with the `data.records:write` scope, with access to the target base.
- **Database setup:** An Airtable base containing a table with text fields `Name` and `room`.

### Form Submission Example

```json
{
  "Your Name ": "John Doe",
  "Choose room": "deluxe"
}
```

### Deployment Steps

1. Import `workflow.json` into your n8n workspace.
2. Authenticate your Airtable account with a Personal Access Token.
3. Select your target Airtable Base ID and Table ID in the Airtable node.
4. Activate the workflow and share the generated Form URL with users.

### Tech Stack

- **Orchestration:** n8n Workflow Automation Engine
- **Database:** Airtable API
- **Trigger Type:** HTTP form submission (`n8n-nodes-base.formTrigger`)
