# Esports Tournament Management System

A full-stack web application designed to streamline esports tournament operations. It provides role-specific dashboards, automated brackets, manual payment verification, and real-time updates built on a modern React + Firebase architecture.

---

## 📚 Table of Contents

* Overview
* Tech Stack
* Architecture
* Key Features
* Project Structure
* Core Workflows
* Data Model
* Frontend Overview
* Backend Overview
* Environment & Configuration
* Setup & Development
* Security Rules
* Deployment
* Troubleshooting Notes
* Roadmap

---

## 📝 Overview

**Purpose:** Provide a complete end-to-end system for organizing and participating in esports tournaments.

**Core modules:**

* Admin, Organizer, Player, and Viewer dashboards
* Firebase-backed backend (Auth, Firestore, Cloud Functions, Storage, Messaging)
* Automated single-elimination brackets
* UPI-based payment proof verification
* Real-time notifications and audit logging

**Data:** Firestore collections include users, teams, tournaments, registrations, matches, brackets, notifications, highlights, and auditLogs.

---

## 🧰 Tech Stack

### **Frontend**

* React 18, TypeScript, Vite
* Tailwind CSS, tailwind-merge, tailwindcss-animate
* shadcn/ui (Radix UI primitives), lucide-react
* React Router v6
* TanStack React Query
* React Hook Form + Zod
* Firebase Web SDK (Auth, Firestore, Storage, Functions, Messaging)

### **Backend**

* Firebase Functions v5 (Node 20)
* firebase-admin v12

### **Tooling**

* ESLint v9, TypeScript v5
* PostCSS
* @vitejs/plugin-react-swc

---

## 🏗 Architecture

* The client initializes Firebase via `src/services/firebase.ts`.
* Auth state and roles are loaded using custom hooks under `src/hooks/useAuth.ts`.
* Firestore is accessed client-side; privileged tasks use callable Cloud Functions.
* Backend logic (brackets, payments, scheduling, role updates) lives under `functions/src/*`.
* Notifications are stored in Firestore and optionally delivered via FCM.

---

## 🎮 Key Features

* Role-based dashboards for Admin, Organizer, Player, Viewer
* Full tournament lifecycle: draft → registration → live → completed
* Player registration with UPI payment proof + organizer approval
* Automated single-elimination bracket generation
* Automatic match progression and tournament completion
* Scheduled room credential reveal (15 minutes before matches)
* Real-time notifications (Firestore + FCM)
* Complete audit logging for sensitive actions
* Modern, responsive UI with skeletons, toasts, spinners

---

## 📁 Project Structure

```
├── functions/
│   └── src/
│       ├── index.ts          # Function exports
│       ├── auth.ts           # User creation + claims + audit
│       ├── admin.ts          # Admin role updates
│       ├── brackets.ts       # Bracket generation logic
│       ├── payments.ts       # Payment verification
│       ├── scheduler.ts      # Room credential scheduler
│       ├── stats.ts          # Stats + match progression
│       └── types.ts          # Shared backend types
├── src/
│   ├── components/
│   │   ├── auth/ProtectedRoute.tsx
│   │   └── ui/*              # shadcn/ui components
│   ├── hooks/
│   │   ├── useAuth.ts        # Auth provider + actions
│   │   └── use-auth.ts       # Role + permissions helpers
│   ├── pages/
│   │   ├── DashboardRouter.tsx
│   │   ├── AdminDashboard.tsx
│   │   ├── OrganizerDashboard.tsx
│   │   ├── PlayerDashboard.tsx
│   │   └── ViewerDashboard.tsx
│   ├── services/
│   │   ├── firebase.ts       # Firebase setup
│   │   ├── cloudFunctions.ts # Callable helpers
│   │   └── tournaments.ts    # Tournament service
│   └── types/index.ts        # Frontend types
├── firestore.rules
├── storage.rules
├── firebase.json
└── env.example
```

---

## 🔄 Core Workflows

### **User onboarding**

* Trigger: `onUserCreate`
* Creates `users/{uid}` with default role *player*.
* Assigns Firebase custom claims.
* Writes to `auditLogs`.

### **Authentication (frontend)**

Handled in `useAuth.ts`:

* Email/password login
* Google login
* Logout
* Profile update
* Loads Firestore profile on auth change

### **Organizer: Create tournament**

* Form in `CreateTournament.tsx`
* Writes tournament data (schedule, rules, banner, entry fee, etc.)

### **Player: Register + upload payment proof**

* Writes to `registrations` with UPI proof (image + UTR)
* Organizer reviews and approves/rejects

### **Organizer: Verify payment**

* Calls `verifyPayment`
* Updates status, writes audit log, sends notification

### **Bracket generation**

* Callable `generateBrackets`
* Validates participants
* Shuffles entries
* Pads to power-of-two
* Generates first round + brackets document
* Marks tournament as `live`

### **Match progression**

* On match completion, `recalcStats` updates:

  * winners' stats
  * tournament status
  * bracket progression

### **Room credential reveal**

* Scheduler function runs every 5 minutes
* Reveals room credentials ~15 minutes before match
* Sends notifications

---

## 🗃 Data Model (Collections)

* **users**: roles, stats, profile details, FCM token
* **teams**: roster, logos, captain, stats
* **tournaments**: core tournament metadata
* **registrations**: payment proof + verification status
* **matches**: match results and bracket slots
* **brackets**: single-elimination structure
* **highlights**: stored clips/links
* **notifications**: messages, status
* **auditLogs**: sensitive action tracking

---

## 🎨 Frontend Overview

### **Routing**

* `ProtectedRoute.tsx` handles auth + allowedRoles
* `DashboardRouter.tsx` selects dashboard by role

### **Hooks**

* `useAuth.ts` for auth state + actions
* `use-auth.ts` for role helpers like `isAdmin`, `isOrganizer`

### **Services**

* `firebase.ts` handles full Firebase initialization
* `tournaments.ts` contains CRUD for tournaments + matches
* `cloudFunctions.ts` handles callable functions like:

  * `generateBrackets`
  * `verifyPayment`
  * `setCustomClaims`

### **UI**

* shadcn/ui components
* lucide-react icons
* Toasts, skeleton loaders, spinners

---

## ⚙ Backend Overview

### **Cloud Functions**

Exported via `index.ts`:

* `onUserCreate`
* `setCustomClaims`
* `generateBrackets`
* `verifyPayment`
* `timeGateRoomCreds`
* `recalcStats`

---

## 🌐 Environment & Configuration

* Environment template: `env.example`
* Firebase config currently set in `firebase.ts`
* Update project credentials when switching Firebase projects
* Cloud Messaging requires VAPID key + permission prompt

---

## 🛠 Setup & Development

Install dependencies:

```
npm install
cd functions && npm install && cd ..
```

Authenticate & configure Firebase:

```
firebase login
firebase use
```

Run client:

```
npm run dev
```

Deploy Functions:

```
cd functions
npm run build
firebase deploy --only functions
```

Deploy Hosting:

```
npm run build
firebase deploy --only hosting
```

Full deploy:

```
firebase deploy
```

---

## 🔐 Security Rules

* Firestore and Storage security rules stored in `firestore.rules` and `storage.rules`
* Enforce role checks, ownership validation, and document structure consistency

Deploy rules:

```
firebase deploy --only firestore:rules,storage
```

---

## ⚠ Troubleshooting Notes

* Firebase project ID is hardcoded in `firebase.ts`
* FCM requires proper VAPID key + service worker support
* Role updates must be performed via `setCustomClaims`
* Brackets are single-elimination only

---

## 🗺 Roadmap

* Double-elimination and round-robin brackets
* Live stream integration
* Advanced analytics and leaderboards
* PWA/mobile support
* Tournament templates
* Emulator support via env flags
