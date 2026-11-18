# CheckSync — Club & School Attendance Suite

![Architecture](https://img.shields.io/badge/Architecture-Single--File_Web_App-6B705C?style=for-the-badge)
![Backend](https://img.shields.io/badge/Backend-Firebase_Auth_%26_Firestore-CB997E?style=for-the-badge)
![Exports](https://img.shields.io/badge/Reports-CSV_%26_XLSX-A5A58D?style=for-the-badge)
![Compliance](https://img.shields.io/badge/Privacy-FERPA_Ready-2D6A4F?style=for-the-badge)

**CheckSync** is a zero-build, real-time attendance and absence-management web application designed for high school clubs, student organizations, and activity advisors. It bridges traditional spreadsheet attendance exports (such as 5-Star Students) with live QR code check-ins, real-time absence excuse approvals, chronic-absence alerts, and one-click Gmail BCC follow-ups.

---

## Table of Contents

- [Overview](#overview)
- [Key Features](#key-features)
  - [Advisor & Officer Portal](#advisor--officer-portal)
  - [Student QR Check-In Portal](#student-qr-check-in-portal)
  - [Security & Role Isolation](#security--role-isolation)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Getting Started & Deployment](#getting-started--deployment)
- [Firebase Configuration & Security Rules](#firebase-configuration--security-rules)
- [How It Works (Workflows)](#how-it-works-workflows)
- [Spreadsheet Compatibility](#spreadsheet-compatibility)
- [FERPA & Data Privacy](#ferpa--data-privacy)
- [License](#license)

---

## Overview

Managing club attendance often means juggling paper sign-in sheets, 5-Star CSV exports, scattered excuse emails, and manual absence tallies. **CheckSync** consolidates the entire workflow into a single self-contained HTML application backed by Firebase:

1. **Host Live QR Meetings:** Project a unique QR code and 6-character session code while watching **Present** and **Missing** roster dropdowns update live without refreshing the page.
2. **Evaluate Spreadsheet or QR Attendance:** Compare 5-Star `.csv`/`.xlsx` exports or live QR attendees against your master club roster in seconds.
3. **Automate Excuse Approvals:** Students submit pre-meeting excuses from a dropdown of their registered clubs; approving a request automatically marks the student **Excused** on the roster and recalculates attendance percentages.
4. **Follow Up in One Click:** Generate pre-filled Gmail BCC drafts for unexcused missing members using customizable templates.

---

## Key Features

### Advisor & Officer Portal
* **First-Time Advisor Setup Wizard:** Guided onboarding to configure Club Name, Advisor Name, credentials, and initial roster via **Spreadsheet Upload**, **Manual Entry**, or **Add Later**.
* **Live QR Meeting Host & Real-Time Feed:**
  * Generates a unique QR code, direct check-in link (`?checkin=CODE`), and 6-character manual session code.
  * Live Firestore listeners populate collapsible **Present** (tagged as *Roster Member* or *Guest / Unlisted*) and **Missing Members** (*Excused* vs. *Not Checked In*) dropdowns in real time.
  * One-click **Stop Session** locks check-ins immediately and transitions directly to attendance evaluation.
* **Daily Attendance Evaluation:**
  * Supports `.csv`, `.xlsx`, and `.xls` uploads (including 5-Star exports) as well as direct syncing from Live QR sessions.
  * Displays real-time stats (**Present**, **Excused**, **Unexcused**, and **Attendance Rate %**).
  * Automatically triggers a **Chronic Absence Alert (3+ Strikes)** banner when a missing student has accumulated 3 or more unexcused absences.
* **Absence Excuse Approval Queue:**
  * Real-time queue with live badge counters on the navigation tab and attendance view.
  * Advisors can attach an optional note and **Approve & Mark Excused** (automatically updating the roster and recalculating attendance) or **Deny / Re-open** requests.
* **Interactive Roster Editor:** Searchable member directory with individual add/remove controls, pre-excused toggles, bulk **Clear All Excused**, and replacement CSV/Excel imports.
* **History & Audit Reports:** Tracks cumulative unexcused absences per member, stores permanent session logs, and exports both **Daily Attendance CSVs** and **Full Semester Audit CSVs**.
* **Custom Email Templates & Gmail BCC Integration:** Customize subjects and bodies using dynamic placeholders (`{ClubName}`, `{Date}`, `{AdvisorName}`), review a checklist of missing students (with excused members automatically unchecked), and launch a pre-addressed Gmail BCC compose window.

### Student QR Check-In Portal
* **In-Browser QR Scanner & Session Code Entry:** Scan the projected meeting QR code using the device camera (`html5-qrcode`), open a direct check-in URL, or type the 6-character session code manually.
* **Registered-Club Absence Excuse Form:**
  * Automatically populates a dropdown of clubs associated with the student's **name** and **email** (synced from advisor rosters, student profiles, and QR check-in history).
  * Supports structured categories (*Athletics*, *Doctor / Medical Appointment*, *Academic / Exam Conflict*, *Family Emergency*, and *Other* with an animated explanation field).
* **Live Excuse Status Tracker:** Students see real-time updates (*Pending Review*, *Approved (Excused)*, or *Denied*), read advisor feedback notes, and can withdraw pending requests.

### Security & Role Isolation
* **Permanent Email-to-Role Locking:** Accounts are permanently bound to either `advisor` or `student` in Firestore (`email_roles` and `users`), preventing cross-portal logins and hiding portal switchers once authenticated.
* **Session Persistence:** Remembers the active portal across page reloads and token refreshes without accidental logouts.

---

## Tech Stack

| Layer | Technology | Purpose |
| :--- | :--- | :--- |
| **Frontend** | HTML5, CSS3, Vanilla JavaScript | Zero-build single-file application with responsive card UI |
| **Authentication** | Firebase Auth (v10.8.0 Compat) | Email/password sign-up and role-locked session management |
| **Database** | Cloud Firestore | Real-time `onSnapshot` listeners for QR sessions, rosters, and excuse queues |
| **Spreadsheet I/O** | [PapaParse](https://www.papaparse.com/) & [SheetJS (XLSX)](https://sheetjs.com/) | Client-side parsing of `.csv`, `.xlsx`, and `.xls` files |
| **QR Generation** | [QRCode.js](https://davidshimjs.github.io/qrcodejs/) | Dynamic QR code rendering for live meetings |
| **QR Scanning** | [html5-qrcode](https://github.com/mebjas/html5-qrcode) | Cross-browser camera barcode/QR scanning |

---

## Project Structure

CheckSync is intentionally packaged as a **single self-contained HTML file** so any school club or advisor can host it for free without Node.js, bundlers, or build steps:

```text
checksync/
├── index.html          # Complete application (UI, styles, logic, Firebase SDKs)
└── README.md           # Setup, deployment, and Firestore security rules guide
```

---

## Getting Started & Deployment

### 1. Clone the Repository
```bash
git clone https://github.com/your-username/checksync.git
cd checksync
```

### 2. Configure Your Firebase Project
1. Go to the [Firebase Console](https://console.firebase.google.com/) and create a new project.
2. Enable **Authentication → Sign-in method → Email/Password**.
3. Create a **Cloud Firestore** database in production mode.
4. Register a **Web App** in Project Settings and copy your `firebaseConfig` object into `index.html`:
   ```javascript
   var firebaseConfig = {
     apiKey: "YOUR_API_KEY",
     authDomain: "YOUR_PROJECT.firebaseapp.com",
     projectId: "YOUR_PROJECT_ID",
     storageBucket: "YOUR_PROJECT.firebasestorage.app",
     messagingSenderId: "YOUR_SENDER_ID",
     appId: "YOUR_APP_ID"
   };
   ```

### 3. Deploy Anywhere (Static Hosting)
Because CheckSync has zero build dependencies, you can deploy `index.html` immediately on:
* **GitHub Pages:** Push `index.html` to `main` and enable Pages under **Settings → Pages**.
* **Firebase Hosting:** Run `firebase init hosting` and `firebase deploy`.
* **Vercel / Netlify:** Drag and drop the folder or connect your GitHub repository.

> **Note:** Camera-based QR scanning (`html5-qrcode`) requires **HTTPS** (automatically provided by GitHub Pages, Firebase Hosting, Netlify, and Vercel) or `localhost`.

---

## Firebase Configuration & Security Rules

Copy and publish the following rules in **Firebase Console → Firestore Database → Rules** to enforce role isolation, allow students to look up their registered clubs by name/email, and secure live QR check-in sessions:

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    // 1. Role lookup index (checked before & during login/signup)
    match /email_roles/{emailKey} {
      allow read: if true;
      allow create, update: if request.auth != null
        && request.resource.data.uid == request.auth.uid;
    }

    // 2. User account & locked role metadata
    match /users/{userId} {
      allow read, write: if request.auth != null
        && request.auth.uid == userId;
    }

    // 3. Student profiles & associated registered clubs
    match /students/{studentId} {
      allow read, write: if request.auth != null
        && request.auth.uid == studentId;
    }

    // 4. Club documents (advisors manage their own club; signed-in users can read for roster matching)
    match /clubs/{clubId} {
      allow read: if request.auth != null;
      allow write: if request.auth != null
        && request.auth.uid == clubId;
    }

    // 5. Synced club roster index (allows students to look up their registered clubs by email/name)
    match /club_codes/{clubCode} {
      allow read: if request.auth != null;
      allow create: if request.auth != null
        && request.resource.data.clubUid == request.auth.uid;
      allow update, delete: if request.auth != null
        && resource.data.clubUid == request.auth.uid;
    }

    // 6. Live QR check-in sessions
    match /qr_sessions/{sessionCode} {
      allow read: if request.auth != null;
      allow create: if request.auth != null
        && request.resource.data.clubUid == request.auth.uid;
      allow update: if request.auth != null && (
        resource.data.clubUid == request.auth.uid ||
        (resource.data.active == true
          && request.resource.data.diff(resource.data).affectedKeys().hasOnly(['attendees']))
      );
      allow delete: if request.auth != null
        && resource.data.clubUid == request.auth.uid;
    }

    // 7. Student absence excuse requests & advisor approval queue
    match /excuse_requests/{requestId} {
      allow read: if request.auth != null && (
        resource.data.studentUid == request.auth.uid ||
        resource.data.clubUid == request.auth.uid
      );
      allow create: if request.auth != null
        && request.resource.data.studentUid == request.auth.uid;
      allow update: if request.auth != null
        && resource.data.clubUid == request.auth.uid;
      allow delete: if request.auth != null && (
        resource.data.studentUid == request.auth.uid ||
        resource.data.clubUid == request.auth.uid
      );
    }
  }
}
```

---

## How It Works (Workflows)

### Advisor Workflow
1. **Sign Up & Import Roster:** Create an Advisor account and upload a `.csv`/`.xlsx` file containing student emails (and optional full names). CheckSync automatically extracts emails and formats names.
2. **Host a Meeting:** Open **Live QR Meeting**, enter an optional meeting label, and click **Generate Unique QR & Session Code**. Project the screen for students.
3. **Close & Evaluate:** Click **Stop Session (Close Check-In)** once the meeting begins, then click **Evaluate Attendance from This Session** (or upload a 5-Star spreadsheet in **Attendance Check**).
4. **Save & Follow Up:** Click **Save to History** to log strikes, **Export CSV Report** for records, or **Email Missing** to open a BCC email draft in Gmail.

### Student Workflow
1. **Sign Up / Log In:** Create a Student account with your full name and school email.
2. **Check In:** Click **Scan Meeting QR Code** (or enter the 6-character code on the board). Your check-in timestamp appears immediately on the advisor's live dashboard.
3. **Submit an Excuse:** Select your registered club from the dropdown, pick the meeting date and reason category, and submit. Watch the status pill update live once your advisor reviews it.

---

## Spreadsheet Compatibility

CheckSync's parser automatically scans every row of uploaded `.csv`, `.xlsx`, and `.xls` files for valid email addresses (`user@domain.edu`) and adjacent full-name columns—no strict column header naming is required. If only an email address (e.g., `jordan.patel@school.org`) is present, CheckSync automatically derives the display name (`Jordan Patel`).

---

## FERPA & Data Privacy

* **Client-Side Spreadsheet Processing:** Uploaded CSV and Excel files are parsed locally in the browser via PapaParse and SheetJS; raw files are never uploaded to external servers.
* **End-of-Year Wipe:** Under **Email & Settings → FERPA Data Controls**, advisors can execute a one-click **FERPA End-of-Year Wipe** to permanently erase student rosters and attendance logs at the conclusion of the school year.

---

## License

This project is open-source and available under the [MIT License](LICENSE).