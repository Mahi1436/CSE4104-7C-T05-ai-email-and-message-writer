# AI Email & Message Writer — Implementation Plan

**Team:** CSE4104-7C-T05  
**Course:** CSE 4104 – Software Development III (Fall 2026)

## 1. Project Overview

AI Email & Message Writer is a web application that helps users create, refine, proofread, and reply to emails or short messages with the assistance of Google Gemini AI. It supports different writing purposes, tones, and lengths, and offers draft history and user profile settings.

## 2. Current Project Status

The project foundation is already in place:

- Backend: Express server, CORS, dotenv, MongoDB connection, and `/api/health` route.
- Frontend: Vite + React 19 scaffold.
- Repository structure: `frontend/`, `backend/`, `diagrams/`, and `documents/`.

## 3. Technology Stack

| Layer | Technology |
|---|---|
| Frontend | React 19, Vite, React Router, Axios, TailwindCSS |
| Backend | Node.js, Express 5, Mongoose |
| Database | MongoDB (Atlas for production) |
| AI | Google Gemini API (`@google/generative-ai`) |
| Authentication | JWT, bcryptjs |
| Password recovery email | Nodemailer |
| Deployment | Vercel (frontend), Render (backend) |

## 4. Implementation Roadmap

### Part 1 — Project Setup & Infrastructure

**Goal:** Prepare a shared, clean project skeleton.

#### Backend
- Install `bcryptjs`, `jsonwebtoken`, `nodemailer`, `@google/generative-ai`, `express-rate-limit`, and `express-validator`.
- Create `src/models/`, `src/middleware/`, `src/services/`, and `src/utils/`.
- Configure Morgan and Helmet.
- Implement centralized error handling.
- Add rate limiting for `/api/ai/*`.
- Create/update `.env.example` with required variables.
- Ensure `.gitignore` excludes `node_modules`, `.env`, and `dist`.

#### Frontend
- Install `react-router-dom`, `axios`, `tailwindcss`, and `lucide-react`.
- Configure TailwindCSS.
- Create `src/components/`, `src/pages/`, `src/context/`, `src/services/`, and `src/hooks/`.
- Create an Axios base instance in `src/services/api.js`.
- Add placeholder pages: Home, Login, Register, Writer, History, Profile, and Admin.
- Configure routes in `App.jsx`.

**Deliverable:** A clean, runnable skeleton with routes and both servers starting successfully.

### Part 2 — User Authentication

**Goal:** Provide secure registration, login, logout, and password recovery.

#### User Model
Fields:
- `name`
- `email`
- `password` (hashed)
- `role` (`user` or `admin`)
- `isBlocked`
- `designation`
- `institution`
- `studentId`
- `phone`
- `defaultTone`
- `createdAt`

#### Backend API
- `POST /api/auth/register` — validate, hash password, save user, and return JWT.
- `POST /api/auth/login` — verify credentials and return JWT.
- `POST /api/auth/forgot-password` — generate reset token and send email.
- `POST /api/auth/reset-password` — validate token and update password.

#### Frontend
- Registration form (name, email, password, confirm password).
- Login form (email and password) with Forgot Password link.
- Forgot Password and Reset Password pages.
- `AuthContext.jsx` to manage user state and login/logout helpers.
- `PrivateRoute.jsx` to protect authenticated pages.
- Axios interceptor to attach the bearer token.

**Deliverable:** Users can register, log in, log out, and recover passwords; protected pages require authentication.

### Part 3 — AI Message Writer (Core Feature)

**Goal:** Generate emails and short messages from user-provided ideas.

#### Backend
- Create `src/services/gemini.service.js`.
- Implement `generateMessage({ type, purpose, tone, length, idea })`.
- Implement `refineMessage({ text, action })` for shorter, more formal, friendlier, or grammar-corrected output.
- Create `POST /api/ai/generate` and `POST /api/ai/refine` (authenticated).
- Track usage counts.

#### Frontend
- Writer page with idea/context input (maximum 1,000 characters).
- Dropdowns for type (Email/Short Message), purpose, tone, and length.
- Generate button, loading state, and result panel.
- Editable result including subject and body where relevant.
- Copy and Regenerate actions.
- Quick Refine actions: Shorter, More Formal, Friendlier, Fix Grammar.
- Encapsulate state and API calls in `useWriter.js`.
- Support Bangla/Banglish input.

**Deliverable:** Authenticated users can generate, edit, refine, copy, and regenerate content.

### Part 4 — Smart Reply

**Goal:** Generate suggested replies to a message the user received.

#### Backend
- Implement `generateReplies({ receivedMessage, tone })`.
- Return 2–3 labelled suggestions.
- Create authenticated `POST /api/ai/smart-reply`.
- Track usage.

#### Frontend
- Textarea to paste a received email/message.
- Tone selector (Formal, Professional, Friendly, Persuasive).
- Generate Replies button.
- Display 2–3 suggestion cards with Select and Copy actions.
- Allow a selected suggestion to be edited before copying.

**Deliverable:** Users receive 2–3 reply suggestions and can edit and copy their preferred reply.

### Part 5 — AI Proofreading

**Goal:** Correct grammar and improve writing at different levels.

#### Backend
- Implement `proofread({ text, level })`.
- Levels: `fix_only`, `fix_improve`, and `rewrite`.
- Return corrected text and a list of original/corrected changes.
- Create authenticated `POST /api/ai/proofread`.

#### Frontend
- Text input.
- Modes: Fix Mistakes Only, Fix and Improve, Rewrite Professionally.
- Proofread action and result panel.
- Display changes as Original → Corrected pairs.
- Copy corrected text.

**Deliverable:** Users can proofread text at three levels and review suggested changes.

### Part 6 — Draft History

**Goal:** Save, search, browse, and delete generated drafts.

#### Draft Model
- `userId` (reference to User)
- `type` (`email` or `message`)
- `purpose`
- `tone`
- `length`
- `idea`
- `subject`
- `body`
- `createdAt`

#### Backend API
- `POST /api/drafts` — save a draft.
- `GET /api/drafts` — paginated list with keyword search.
- `GET /api/drafts/:id` — retrieve a draft.
- `DELETE /api/drafts/:id` — delete one draft.
- `DELETE /api/drafts` — delete all drafts belonging to the user.
- Optionally allow saving drafts during generation.

#### Frontend
- History page with keyword/type search.
- Draft cards showing type, idea preview, and date.
- Expand drafts to view full subject/body.
- Copy and Delete actions.
- Clear All History with confirmation.
- Save to History action on the writer result.

**Deliverable:** Users can browse, search, copy, and delete their saved drafts.

### Part 7 — User Profile

**Goal:** Manage profile information and writing preferences.

#### Backend API
- `GET /api/profile` — return user details without password.
- `PUT /api/profile` — update permitted profile fields.
- `PUT /api/profile/change-password` — verify current password and change it.

#### Frontend
- Show name, email (read-only), designation, institution, student ID, and phone.
- Edit profile form.
- Default Tone preference.
- Change password form.
- Success/error notifications.

**Deliverable:** Users can update profile details and writing preferences.

### Part 8 — Admin Panel

**Goal:** Provide system usage overview and account management.

#### Stats Model
Daily statistics may include:
- `date`
- `totalGenerated`
- `failedRequests`
- `activeUsers`

#### Backend API
- `GET /api/admin/stats` — aggregates for today, 7 days, and 30 days.
- `GET /api/admin/users` — paginated/searchable user list.
- `PUT /api/admin/users/:id/block` — block or unblock a user.
- `DELETE /api/admin/users/:id` — delete a user and their drafts.
- Reject login attempts for blocked accounts.
- Protect admin routes with admin middleware.

#### Frontend
- Admin dashboard stat cards.
- Period selector (Today, Last 7 Days, Last 30 Days).
- User table with search and account status.
- Block/Unblock and Delete actions with confirmation.
- Admin-only route guard.

**Privacy requirement:** Admins may view usage counts and account metadata, but must not access users' private message content.

**Deliverable:** Admins can monitor system usage and manage accounts.

### Part 9 — UI Polish, Responsiveness & Error Handling

- Apply a consistent TailwindCSS design system.
- Test responsive layouts at 375px, 768px, and 1280px.
- Add loading indicators/skeletons for asynchronous operations.
- Add empty states for history and admin lists.
- Add toast notifications.
- Add a 404 page.
- Handle expired JWTs with logout and redirect.
- Show understandable Gemini/API error messages.
- Add inline validation to forms.
- Confirm destructive actions.

**Deliverable:** Responsive, polished UI with clear error handling.

### Part 10 — Testing & Deployment

#### Testing
- End-to-end manual testing of each feature.
- API testing using Postman and export the collection.
- Test empty inputs, long text, Bangla input, blocked-account login, and invalid/expired tokens.
- Cross-browser testing in Chrome, Firefox, and Edge.

#### Deployment
- Backend on Render.
- Frontend on Vercel.
- Production database on MongoDB Atlas.
- Configure environment variables and CORS.
- Use the deployed backend URL in frontend configuration.
- Smoke-test the live application.
- Update README with the deployment URL.

**Deliverable:** A deployed application that can be accessed through a public URL.

## 5. Development Order

1. Project setup and infrastructure.
2. Authentication.
3. Core AI writer.
4. Smart Reply, Proofreading, and Draft History (can be parallelized after the core writer is ready).
5. Profile.
6. Admin panel.
7. UI polish and responsive design.
8. Testing and deployment.

## 6. Package Installation

### Backend
```bash
npm install bcryptjs jsonwebtoken nodemailer @google/generative-ai express-rate-limit express-validator
```

### Frontend
```bash
npm install react-router-dom axios
npm install -D tailwindcss @tailwindcss/vite lucide-react
```

## 7. Proposed Backend Structure

```text
backend/src/
├── config/
│   └── db.js
├── controllers/
│   ├── auth.controller.js
│   ├── ai.controller.js
│   ├── draft.controller.js
│   ├── profile.controller.js
│   └── admin.controller.js
├── middleware/
│   ├── auth.js
│   ├── admin.js
│   └── errorHandler.js
├── models/
│   ├── User.js
│   ├── Draft.js
│   └── Stats.js
├── routes/
│   ├── health.routes.js
│   ├── auth.routes.js
│   ├── ai.routes.js
│   ├── draft.routes.js
│   ├── profile.routes.js
│   └── admin.routes.js
├── services/
│   └── gemini.service.js
└── index.js
```

## 8. Environment Variables

Use a `.env` file locally and configure equivalent secrets in deployment. Never commit real secrets.

```env
MONGO_URI=
JWT_SECRET=
GEMINI_API_KEY=
EMAIL_USER=
EMAIL_PASS=
```

## 9. Security Notes

- Hash passwords with bcryptjs.
- Protect private APIs with JWT authentication.
- Restrict admin endpoints using role checks.
- Block access for blocked accounts.
- Apply rate limiting to AI endpoints.
- Validate and sanitize inputs.
- Keep secrets out of source control.
- Admins must not see private draft/message contents.

---
