# CareerGuide — Full Project Documentation

> **Platform:** BiT (Bureau of Information Technology) Student Career Guidance System
> **Last Updated:** May 2026
> **Status:** Active Development

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Tech Stack](#2-tech-stack)
3. [Project Structure](#3-project-structure)
4. [User Roles & Access](#4-user-roles--access)
5. [Backend API](#5-backend-api)
6. [Web Frontend](#6-web-frontend)

7. [Mobile App](#7-mobile-app)
8. [Authentication & Security](#8-authentication--security)
9. [AI Integration](#9-ai-integration)
10. [Offline Support (Mobile)](#10-offline-support-mobile)
11. [Database Schema](#11-database-schema)
12. [Configuration & Deployment](#12-configuration--deployment)

---

## 1. Project Overview

CareerGuide is a full-stack career guidance platform built specifically for students at Bahir Dar Institute of Technology (BiT). It provides:

- **AI-powered career roadmaps and courses** tailored to individual student goals
- **Curated content** created and published by BiT academic staff
- **Teacher-student interaction** through resource sharing, feedback, and progress monitoring
- **Administrative governance** for user management, content moderation, and platform oversight

The platform runs as:
- A **React web application** (desktop-first, responsive)
- A **Flutter mobile application** (Android/iOS)
- A **PHP REST API backend** served via XAMPP/Apache

---

## 2. Tech Stack

### Backend
| Component | Technology |
|---|---|
| Language | PHP 8.0+ |
| Server | XAMPP Apache (port 80) |
| Database | MySQL (via PDO) |
| Auth | JWT (firebase/php-jwt) |
| Config | vlucas/phpdotenv |
| Architecture | Custom router, no framework, raw PDO queries |
| AI Providers | Groq (primary), OpenRouter (backup), Ollama (local) |

### Web Frontend
| Component | Technology |
|---|---|
| Framework | React 19 + TypeScript 5.8 |
| Build Tool | Vite 6 |
| Styling | Tailwind CSS 3 |
| Icons | lucide-react |
| Markdown | react-markdown |
| AI SDK | @google/genai + @google/generative-ai |
| Utilities | date-fns, uuid |

### Mobile App
| Component | Technology |
|---|---|
| Framework | Flutter / Dart (SDK ≥3.0) |
| State Management | flutter_riverpod 2.5 |
| Navigation | go_router 13 |
| HTTP Client | Dio 5.4 |
| Token Storage | flutter_secure_storage |
| Offline Cache | sqflite 2.3 + connectivity_plus |
| Markdown | flutter_markdown |
| File Handling | image_picker, file_picker |
| Utilities | intl, timeago, url_launcher, shared_preferences |

---

## 3. Project Structure

```
careerguide/
├── backend/                    # PHP REST API
│   ├── app/
│   │   ├── Controllers/        # Request handlers
│   │   ├── Models/             # Database models
│   │   ├── Services/           # AI service layer
│   │   │   └── AiProviders/    # Groq, OpenRouter, Ollama
│   │   ├── Helpers/            # JWTHelper
│   │   └── Interfaces/         # AiProviderInterface
│   ├── config/
│   │   └── database.php        # DB connection
│   ├── database/               # SQL migrations
│   ├── public/
│   │   └── index.php           # Entry point
│   ├── routes/
│   │   └── api.php             # All route definitions
│   ├── uploads/                # Uploaded files
│   └── .env                    # Environment config
│
├── frontend/                   # React web app
│   ├── components/
│   │   ├── Auth/               # Login, signup, forgot password
│   │   ├── Dashboard/          # All dashboard views
│   │   │   ├── Admin/          # Admin-specific views
│   │   │   ├── BiT/            # BiT-specific views
│   │   │   ├── Teacher/        # Teacher-specific views
│   │   │   └── common/         # Shared dashboard components
│   │   ├── Pages/              # Static pages (FAQ, Privacy, etc.)
│   │   └── common/             # Shared UI components
│   ├── services/               # API client, course/teacher services
│   ├── types.ts                # TypeScript type definitions
│   └── utils/
│       └── apiConfig.ts        # Centralized API base URL
│
└── mobile/                     # Flutter mobile app
    └── lib/
        ├── core/
        │   ├── cache/          # SQLite offline cache
        │   ├── constants/      # ApiConstants
        │   ├── models/         # Data models
        │   ├── network/        # Dio API client
        │   ├── providers/      # Auth, theme providers
        │   ├── router/         # GoRouter config
        │   └── theme/          # AppTheme, AppColors
        ├── features/
        │   ├── admin/          # Admin screens
        │   ├── auth/           # Auth screens
        │   ├── bit/            # BiT screens
        │   ├── shared/         # Notifications, support chat
        │   └── student/        # Student screens + providers
        └── shared/
            └── widgets/        # Reusable widgets
```

---

## 4. User Roles & Access

### Role Overview

| Role | Registration | Approval Required | Portal |
|---|---|---|---|
| `student` | Self-register | No — active immediately | Student Portal |
| `teacher` | Self-register | Yes — admin must approve | Teacher Portal |
| `admin` | Created by super-admin | No | Admin Portal |
| `bit` | Created by admin | No | BiT Portal |

### Account Status Flow
```
Register → pending (teacher/bit) → Admin approves → active
                                 → Admin rejects  → rejected
```
Students skip the pending state and are active immediately.

### Student Capabilities
- Browse and enroll in BiT-curated courses and roadmaps
- Generate AI-powered custom courses and roadmaps
- Track learning progress (lesson completion, course progress %)
- Take AI-generated assessments and quizzes
- Explore career paths and details
- Access teacher-uploaded course materials
- View notifications
- Chat with support (admin)
- Manage profile (name, email, academic year, department, phone)
- Forgot password / reset password

### Teacher Capabilities
- Upload and manage educational resources (documents, videos, links, notes)
- Upload course-specific materials per assigned course
- View students enrolled in their assigned courses
- Monitor student progress per course
- Send feedback to students
- View own resource statistics
- Manage profile (bio, expertise, languages, LinkedIn, website)
- Chat with students and admin via support
- Dark mode toggle

### BiT Staff Capabilities
- Full CRUD on curated roadmaps (3-level: Beginner/Medium/Advanced)
- Full CRUD on courses (modules, lessons, video/file content)
- Full CRUD on career entries
- Publish/unpublish roadmaps, courses, and careers
- View platform analytics (students, courses, roadmaps)

### Admin Capabilities
- View all users and change their roles
- Approve or reject teacher role requests
- Approve or reject educational resources
- View all support conversations
- View platform-wide analytics (total users, students, roadmaps, resources)
- Manage platform resources

---

## 5. Backend API

**Base URL:** `http://localhost/careerguide/backend/api`
**Auth:** Bearer JWT token in `Authorization` header

### 5.1 Authentication

| Method | Endpoint | Description |
|---|---|---|
| POST | `/auth/register` | Register new user (student/teacher) |
| POST | `/auth/login` | Login, returns JWT token + user object |
| POST | `/auth/logout` | Invalidate token |
| POST | `/auth/forgot-password` | Send password reset code to email |
| POST | `/auth/reset-password` | Reset password with code |

**Register payload:**
```json
{
  "name": "Abebe Bekele",
  "email": "abebe@bit.bdu.edu.et",
  "password": "SecurePass123",
  "role_preference": "student",
  "student_id": "BIT/123/14",
  "department": "Computer Science",
  "academic_year": "3rd Year",
  "graduation_year": 2026,
  "phone_number": "+251912345678"
}
```

**Login response:**
```json
{
  "token": "eyJ...",
  "user": {
    "id": 42,
    "name": "Abebe Bekele",
    "email": "abebe@bit.bdu.edu.et",
    "role": "student",
    "account_status": "active",
    "academic_year": "3rd Year",
    "department": "Computer Science"
  }
}
```

### 5.2 User

| Method | Endpoint | Description |
|---|---|---|
| GET | `/users/profile` | Get current user profile |
| PUT | `/users/profile` | Update profile |
| POST | `/users/profile/image` | Upload profile photo |
| DELETE | `/users/profile/image` | Remove profile photo |
| GET | `/users/stats` | Get stats (courses enrolled, lessons done) |
| GET | `/users/activity` | Get recent activity |
| GET | `/users/courses` | Get enrolled courses |
| GET | `/users/roadmaps` | Get saved AI roadmaps |

### 5.3 Courses

| Method | Endpoint | Description |
|---|---|---|
| GET | `/courses` | All courses (AI courses filtered per user) |
| GET | `/courses/:id` | Single course with enrollment data |
| POST | `/courses/generate` | AI-generate a custom course |
| POST | `/courses/:id/enroll` | Enroll in a course (409 = already enrolled) |
| DELETE | `/courses/:id/unenroll` | Unenroll from a course |
| PUT | `/courses/:id/progress` | Update lesson progress |

**AI isolation:** Courses with `author = 'AI Architect'` are only visible to the student who created them. BiT-created courses are visible to all.

### 5.4 Curated Roadmaps

| Method | Endpoint | Description |
|---|---|---|
| GET | `/curated-roadmaps` | Browse published roadmaps (with filters) |
| GET | `/curated-roadmaps/categories` | Get category list |
| GET | `/curated-roadmaps/:id` | Single roadmap detail |
| POST | `/curated-roadmaps/:id/enroll` | Enroll in roadmap |
| GET | `/curated-roadmaps/:id/resources` | Resources linked to roadmap |
| GET | `/curated-roadmaps/:id/courses` | Courses linked to roadmap |

### 5.5 AI Roadmaps (User-generated)

| Method | Endpoint | Description |
|---|---|---|
| POST | `/roadmaps/generate` | Generate AI roadmap (not saved) |
| POST | `/roadmaps` | Save generated roadmap |
| DELETE | `/roadmaps/:id` | Delete saved roadmap |

### 5.6 AI Endpoints

| Method | Endpoint | Description |
|---|---|---|
| POST | `/ai/career-suggestion` | Suggest careers based on interests |
| POST | `/ai/career-details` | Generate career detail content |
| POST | `/ai/lesson-content` | Generate lesson content on-demand |
| POST | `/ai/generate-assessment` | Generate quiz for a course |
| POST | `/ai/quiz-evaluate` | Evaluate quiz answers |

### 5.7 Assessments

| Method | Endpoint | Description |
|---|---|---|
| GET | `/assessments` | Get assessments for current student |
| GET | `/assessments/course/:id` | Assessments for a specific course |
| GET | `/assessments/:id` | Single assessment |
| POST | `/assessments` | Create assessment (teacher/admin) |
| POST | `/assessments/:id/submit` | Submit quiz answers |
| DELETE | `/assessments/:id` | Delete assessment |

### 5.8 Careers

| Method | Endpoint | Description |
|---|---|---|
| GET | `/careers` | All published careers (public) |
| GET | `/careers/categories` | Career categories |
| GET | `/careers/:id` | Single career detail |
| GET | `/bit/careers` | All careers (BiT admin) |
| POST | `/bit/careers` | Create career entry |
| PUT | `/bit/careers/:id` | Update career entry |
| DELETE | `/bit/careers/:id` | Delete career entry |
| POST | `/bit/careers/:id/publish` | Publish career |
| POST | `/bit/careers/:id/unpublish` | Unpublish career |

### 5.9 BiT Content Management

| Method | Endpoint | Description |
|---|---|---|
| GET/POST | `/bit/roadmaps` | List / create roadmaps |
| GET/PUT/DELETE | `/bit/roadmaps/:id` | Single roadmap CRUD |
| POST | `/bit/roadmaps/:id/publish` | Publish roadmap |
| GET/POST | `/bit/courses` | List / create standalone courses |
| GET/PUT/DELETE | `/bit/courses/:id` | Single course CRUD |
| GET | `/bit/analytics` | BiT dashboard analytics |

### 5.10 Teacher Endpoints

| Method | Endpoint | Description |
|---|---|---|
| GET | `/teacher/stats` | Teacher dashboard stats |
| GET | `/teacher/activity` | Recent activity feed |
| GET/PUT | `/teacher/profile` | Teacher profile |
| GET/PUT | `/teacher/settings` | Notification settings |
| GET | `/teacher/course-students` | Students grouped by assigned course |
| GET | `/teacher/students` | All monitored students |
| GET | `/teacher/students/:id/progress` | Student progress detail |
| POST | `/teacher/feedback` | Send feedback to student |
| GET | `/teacher/feedback/:studentId` | Feedback history |
| GET/POST | `/teacher/resources` | List / upload resources |
| PUT/DELETE | `/teacher/resources/:id` | Update / delete resource |
| GET/POST | `/teacher/courses/:id/materials` | Course-specific materials |
| PUT/DELETE | `/teacher/materials/:id` | Update / delete material |

### 5.11 Admin Endpoints

| Method | Endpoint | Description |
|---|---|---|
| GET | `/admin/users` | All users |
| PUT | `/admin/users/:id/role` | Change user role |
| GET | `/admin/approvals/pending` | Pending role requests |
| POST | `/admin/approvals/:id/approve` | Approve role request |
| POST | `/admin/approvals/:id/reject` | Reject role request |
| GET | `/admin/analytics` | Platform analytics |
| GET | `/admin/resources` | All resources |
| POST | `/admin/resources/:id/approve` | Approve resource |
| POST | `/admin/resources/:id/reject` | Reject resource |
| GET | `/admin/support/conversations` | All support conversations |
| GET | `/admin/support/messages/:userId` | Messages with a user |

### 5.12 Notifications

| Method | Endpoint | Description |
|---|---|---|
| GET | `/notifications` | Get all notifications |
| GET | `/notifications/unread-count` | Unread count |
| PUT | `/notifications/mark-all-read` | Mark all as read |
| PUT | `/notifications/:id/read` | Mark one as read |
| DELETE | `/notifications/:id` | Delete notification |

### 5.13 Support Chat

| Method | Endpoint | Description |
|---|---|---|
| GET | `/support/messages` | Get messages (authenticated user) |
| POST | `/support/messages` | Send message |
| GET | `/public/support/messages/:guestId` | Guest messages |
| POST | `/public/support/messages` | Guest send message |

### 5.14 File Upload & Serve

| Method | Endpoint | Description |
|---|---|---|
| POST | `/upload` | Upload file (course content) |
| GET | `/uploads/serve?file=X&type=Y` | Serve uploaded file |

---

## 6. Web Frontend

**URL:** `http://localhost/careerguide/frontend` (or Vite dev server)
**API Base:** `http://localhost/careerguide/backend/api` (configured in `utils/apiConfig.ts`)

### 6.1 Application Entry

The app starts at `App.tsx` which handles:
- Reading auth state from `localStorage` (`auth_token`, `user`)
- Routing to the correct dashboard based on user role
- Theme (light/dark) persistence

### 6.2 Public Pages

| Page | Route | Description |
|---|---|---|
| Landing | `/` | Hero, features, career explorer teaser |
| Login | `/login` | Email + password login |
| Sign Up | `/signup` | Role selection + registration |
| FAQ | `/faq` | Frequently asked questions |
| Privacy Policy | `/privacy` | Privacy policy |
| Terms of Service | `/terms` | Terms of service |
| User Guide | `/guide` | Platform usage guide |
| Public Chat | `/chat` | Guest support chat |

**Email validation:** Accepts any valid email domain (not Gmail-only). Supports `@bit.bdu.edu.et` and all other domains.

**Phone validation (signup):** `+251` is a static prefix. The user enters 9 digits. Valid prefixes: `9xx` (Ethiotelecom) or `7xx` (Safaricom Ethiopia).

### 6.3 Student Dashboard

Accessed after login with `role = 'student'`. Layout: fixed left sidebar + scrollable right content.

| Tab | Component | Description |
|---|---|---|
| Home | `DashboardHome` | Stats cards, quick actions, recent activity |
| Roadmaps | `CuratedRoadmapsView` | Browse BiT roadmaps + AI generator |
| Courses | `LibraryView` | My Courses + Browse All + AI generator |
| Careers | `CareersView` | Career cards with icon-based UI |
| Assessments | `AssessmentView` | Take AI-generated quizzes |
| Progress | `ProgressView` | Course completion tracking |
| Profile | `ProfileSettings` | Edit profile, photo, password |
| Support | `SupportChatView` | Chat with admin |

**Course View (`CourseView.tsx`):**
- Resizable sidebar with module/lesson tree
- Lesson content rendered from JSON blocks (text, image, video, file, link)
- YouTube embed support + uploaded video playback
- AI on-demand lesson content generation for `[CONTENT_PENDING]` lessons
- Progress tracking synced to backend
- "Finish Course" button turns green "✓ Finished" on last lesson
- AI Assessment generation button

**Roadmap View (`CuratedRoadmapsView.tsx`):**
- 3-level structure: Beginner / Medium / Advanced tabs
- Phase accordion with topic expansion
- Linked courses and resources per roadmap

### 6.4 Teacher Dashboard

Accessed after login with `role = 'teacher'` and `account_status = 'active'`.

| Tab | Component | Description |
|---|---|---|
| Dashboard | `TeacherOverview` | Stats (Total Resources, Active Students), quick actions, recent activity |
| My Resources | `TeacherResourcesView` | Upload/manage resources (Total + Approved stats) |
| Students | `TeacherStudentsView` | Students grouped by assigned course with progress |
| Support | `SupportChatView` | Chat with students and admin |
| Profile | `TeacherProfileView` | Profile editor with unified sidebar card |

**Teacher Profile sidebar card contains:**
- Avatar with verified badge
- Resources count + Years Experience
- Expertise areas (tags)
- Languages
- LinkedIn + Website links

**Students View features:**
- Courses accordion (expand/collapse per course)
- Per-student: name, email, academic year, department, progress bar, lessons done, enrolled date, status badge
- Click student → detail modal

**Pending teacher:** Shown a waiting screen until admin approves. Cannot access portal features.

### 6.5 BiT Dashboard

Accessed after login with `role = 'bit'`.

| Tab | Component | Description |
|---|---|---|
| Overview | `BiTOverview` | Analytics: roadmaps, courses, students |
| Roadmaps | `BiTRoadmapsView` | Create/edit/publish roadmaps |
| Courses | `BiTCoursesView` | Create/edit/publish courses |
| Careers | `BiTCareersView` | Create/edit/publish career entries |

**Roadmap structure:** Each roadmap has 3 levels (Beginner, Medium, Advanced), each with phases, each phase with topics and concepts.

**Course structure:** Modules → Lessons. Each lesson can have rich content blocks: text (markdown/HTML), images, videos (YouTube or uploaded), files (PDF, DOCX, etc.), links.

### 6.6 Admin Dashboard

Accessed after login with `role = 'admin'`.

| Tab | Component | Description |
|---|---|---|
| Overview | `AdminOverview` | Platform stats (Total Users, Roadmaps, Resources), activity feed, quick actions |
| Users | `AdminUsersView` | List all users, change roles |
| Approvals | `AdminApprovalsView` | Approve/reject teacher role requests |
| Resources | `AdminResourcesView` | Browse and moderate resources |
| Analytics | `AdminAnalyticsView` | Detailed platform analytics |
| Support | `AdminSupportView` | View all user conversations |

**Default admin credentials:**
- Admin: `admin@bit.bdu.edu.et` / `Admin@2024`
- BiT: `bit@bit.bdu.edu.et` / `Bit@2024`

### 6.7 Shared Components

| Component | Description |
|---|---|
| `NotificationBell` | Bell icon with live unread count badge |
| `UserAvatar` | Initials-based avatar with optional photo |
| `ConfirmModal` | Reusable confirm/cancel dialog |
| `Skeleton` / `CardGridSkeleton` | Loading placeholders |
| `SupportChatView` | Reusable chat widget (student, teacher, admin) |
| `RichTextEditor` | WYSIWYG editor for course content |

---

## 7. Mobile App

**Platform:** Android & iOS
**Min SDK:** Android 5.0+ / iOS 12+
**Entry point:** `lib/main.dart`

### 7.1 Network Configuration

The mobile app connects to the backend via LAN IP for physical devices:

```dart
// mobile/lib/core/constants/api_constants.dart
static const String _physicalDeviceIp = '10.161.174.178';
static const String _backendPath = '/careerguide/backend/api';
```

| Environment | Base URL |
|---|---|
| Physical device | `http://10.161.174.178/careerguide/backend/api` |
| Android emulator | `http://10.0.2.2/careerguide/backend/api` |
| Web (Chrome) | `http://localhost/careerguide/backend/api` |

> **To change the device IP:** Edit `_physicalDeviceIp` in `api_constants.dart`.

### 7.2 Navigation (GoRouter)

Routes are defined in `mobile/lib/core/router/app_router.dart`.

**Auth routes (no shell):**
```
/splash          → SplashScreen
/login           → LoginScreen
/register        → RegisterScreen
/onboarding      → OnboardingScreen
/pending         → PendingApprovalScreen
/forgot-password → ForgotPasswordScreen
```

**Student shell (`/student`):**
```
/student                    → StudentHomeScreen
/student/roadmaps           → RoadmapsScreen
/student/roadmaps/generate  → AiRoadmapGeneratorScreen
/student/roadmaps/:id       → RoadmapDetailScreen
/student/courses            → CoursesScreen
/student/courses/generate   → AiCourseGeneratorScreen
/student/courses/:id        → CourseDetailScreen
/student/assessments        → AssessmentsScreen
/student/assessments/:id    → AssessmentQuizScreen
/student/profile            → StudentProfileScreen
/student/progress           → ProgressScreen
/student/careers            → CareersScreen
```

**Admin shell (`/admin`):**
```
/admin            → AdminHomeScreen
/admin/users      → AdminUsersScreen
/admin/approvals  → AdminApprovalsScreen
/admin/resources  → AdminResourcesScreen
```

**BiT shell (`/bit`):**
```
/bit           → BitHomeScreen
/bit/roadmaps  → BitRoadmapsScreen
/bit/courses   → BitCoursesScreen
```

**Shared routes (no shell):**
```
/notifications  → NotificationsScreen
/support        → SupportChatScreen
```

**Route guards:**
- Unauthenticated users → `/login`
- Pending teachers → `/pending`
- Cross-role access → redirected to own portal
- Authenticated users on `/login` or `/register` → redirected to their portal

### 7.3 State Management (Riverpod)

All state is managed with `flutter_riverpod`. Key providers:

#### Auth (`core/providers/auth_provider.dart`)
```dart
authProvider          // StateNotifierProvider<AuthNotifier, AsyncValue<UserModel?>>
currentUserProvider   // Provider<UserModel?> — convenience accessor
```
- `login(email, password)` — POST `/auth/login`, stores JWT
- `register(data)` — POST `/auth/register`
- `logout()` — removes token immediately (fast logout), fires server call in background

#### Theme (`core/providers/theme_provider.dart`)
```dart
themeModeProvider  // StateNotifierProvider<ThemeModeNotifier, ThemeMode>
```
Persists light/dark preference via `shared_preferences`.

#### Student Providers (`features/student/providers/student_providers.dart`)
```dart
studentStatsProvider        // FutureProvider — courses enrolled, lessons done
enrolledCoursesProvider     // FutureProvider<List<CourseModel>>
allCoursesProvider          // FutureProvider<List<CourseModel>> — browse tab
curatedRoadmapsProvider     // FutureProvider.family — with filters
roadmapDetailProvider       // FutureProvider.family<String>
studentAssessmentsProvider  // FutureProvider<List<Map>>
recentActivityProvider      // FutureProvider<List<Map>>
enrollInCourse(ref, id)     // Future<bool> — handles 409 as success
```

#### Cached Providers (`core/cache/cached_providers.dart`)
```dart
cachedAllCoursesProvider          // Cache-first, falls back to SQLite offline
cachedEnrolledCoursesProvider     // Cache-first, 30-min TTL
cachedCuratedRoadmapsProvider     // Cache-first, 1-hour TTL
syncPendingMutations(ref)         // Flushes offline mutation queue on reconnect
```

#### Notification Badge (`shared/widgets/app_drawer.dart`)
```dart
drawerUnreadCountProvider  // FutureProvider<int> — live badge in drawer
```

### 7.4 Student Portal Screens

#### Home Screen (`student_home_screen.dart`)
- Welcome card with user initials
- Stats grid: Courses enrolled + Lessons Done (2-column, `childAspectRatio: 2.4`)
- Quick action tiles: Roadmaps, Courses, Assess, Progress
- Explore cards: Career Roadmaps, Curated Courses, Skill Assessments, Explore Careers
- Support chat shortcut
- Auto-syncs offline mutations on load and on connectivity restore

#### Courses Screen (`courses_screen.dart`)
Two tabs:
- **My Courses** — enrolled courses with progress bars
- **Browse All** — all BiT courses with Enroll Now / Enrolled button

Enroll button behavior:
- Shows spinner while enrolling
- Switches to green "✓ Enrolled" badge immediately on success
- 409 (already enrolled) also marks as enrolled
- State persists across tab switches (seeded from `enrolledCoursesProvider`)
- Pull-to-refresh invalidates both providers

AI Custom Course FAB navigates to `/student/courses/generate`.

#### Course Detail Screen (`course_detail_screen.dart`)
- Hero header with gradient (course title, level badge)
- Progress bar (if enrolled)
- Curriculum accordion (modules → lessons)
- Lesson viewer with:
  - AI on-demand content generation for `[CONTENT_PENDING]` lessons
  - Markdown + HTML rendering
  - Progress bar (lesson X of Y)
  - Next / Finish navigation
- Resources tab (teacher-uploaded materials)

#### Roadmaps Screen (`roadmaps_screen.dart`)
- Search + category/difficulty filters
- Roadmap cards with level badge, phase count, enrollment count
- Tap → `RoadmapDetailScreen`

#### Roadmap Detail Screen (`roadmap_detail_screen.dart`)
- Hero header
- Phase accordion with topic expansion
- HTML content rendered as plain text (strips tags, parses `<li>` items as bullets)
- No "Start This Path" button (removed)

#### AI Generators
- `AiRoadmapGeneratorScreen` — text input → AI generates roadmap → save
- `AiCourseGeneratorScreen` — text input → AI generates course → auto-enroll

#### Assessments Screen (`assessments_screen.dart`)
- Lists available assessments
- Tap → `AssessmentQuizScreen` (multiple choice, submit, see score)

#### Progress Screen (`progress_screen.dart`)
- Course completion overview
- Lesson counts per course

#### Careers Screen (`careers_screen.dart`)
- Career cards from BiT-published entries
- Tap → career detail with AI-generated description

#### Profile Screen (`student_profile_screen.dart`)
- Avatar with initials
- Account Info: Academic Year, Department, Email
- Settings: Dark Mode toggle, Notifications (tappable → `/notifications`)
- Logout button
- No Student ID shown

### 7.5 Admin Portal Screens

#### Admin Home (`admin_home_screen.dart`)
- Stats grid: Total Users, Students, Teachers, Pending approvals
- Quick action buttons: Approvals, Users

#### Admin Users (`admin_users_screen.dart`)
- List all users with role color badges
- Role colors: admin=indigo, bit=sky, student=blue (teacher removed)

#### Admin Approvals (`admin_approvals_screen.dart`)
- Pending teacher/bit role requests
- Approve / Reject buttons

#### Admin Resources (`admin_resources_screen.dart`)
- Browse all educational resources

### 7.6 BiT Portal Screens

#### BiT Home (`bit_home_screen.dart`)
- Stats: Roadmaps, Published, Courses, Students

#### BiT Roadmaps (`bit_roadmaps_screen.dart`)
- List all roadmaps with status
- Create / Edit / Delete / Publish

#### BiT Courses (`bit_courses_screen.dart`)
- List all courses
- Create / Edit / Delete

### 7.7 Shared Screens

#### Notifications (`notifications_screen.dart`)
- Full notification inbox
- Swipe-to-dismiss (deletes notification)
- "Mark all read" button
- Unread dot indicator per item
- Type-based icons and colors

#### Support Chat (`support_chat_screen.dart`)
- Real-time-style chat with admin
- Error state shows friendly empty state (no crash on 401)
- Messages fetched from `/support/messages`

### 7.8 Drawer Navigation

The `AppDrawer` widget is used across all portals. It contains:
- Logo + portal name
- User avatar + name + role label
- Navigation items (role-specific)
- Divider
- **Notifications** item with live unread count badge (red pill)
- **Support** item
- Theme toggle (Dark/Light Mode)
- Logout button

Notifications badge refreshes automatically after returning from the notifications screen.

### 7.9 Registration Flow

**Step 1 — Account Info:**
- Full Name (first + father name required, min 5 chars)
- Email Address
- Password (min 6 chars)

**Step 2 — Academic Info:**
- Student ID (optional)
- Department (dropdown): Cyber Security, Computer Science, Information Technology, Software Engineering, Information System, Other
- Academic Year (dropdown): 1st–4th Year always; 5th Year only for Software Engineering/Other; Postgraduate always
- Graduation Year (number, 2020–2035, default 2026)

**Phone field (Step 1):**
- `+251` is a static non-editable prefix
- User enters 9 digits
- Valid: `9xx` (Ethiotelecom) or `7xx` (Safaricom Ethiopia)
- Sent to backend as `+251XXXXXXXXX`

**Forgot Password flow:**
1. Enter email → backend sends reset code
2. Enter code + new password + confirm
3. Success screen → back to login

---

## 8. Authentication & Security

### JWT Flow
1. Client sends `POST /auth/login` with email + password
2. Backend validates credentials, returns `{ token, user }`
3. **Web:** Token stored in `localStorage` as `auth_token`
4. **Mobile:** Token stored in `flutter_secure_storage` (encrypted)
5. All subsequent requests include `Authorization: Bearer <token>`
6. Token expiry: 86400 seconds (24 hours), configurable via `JWT_EXPIRY` in `.env`

### Password Reset
1. `POST /auth/forgot-password` with `{ email }` — backend creates a time-limited token in `password_resets` table and (in production) emails it
2. `POST /auth/reset-password` with `{ email, token, password, password_confirmation }`
3. Table is auto-created by `AuthController` if it doesn't exist

### Role-Based Access Control
- Every protected endpoint calls `requireAuth(['role1', 'role2'])` in the controller
- Teachers with `account_status = 'pending'` are blocked from teacher endpoints
- AI-generated courses (`author = 'AI Architect'`) are filtered server-side — only visible to their creator

### Email Validation
- Web: regex `/^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/i` — accepts any domain
- Mobile: `contains('@')` check — accepts any domain
- Supports `@bit.bdu.edu.et`, `@gmail.com`, and all other domains

---

## 9. AI Integration

### Backend AI Service (`app/Services/`)

The backend uses a multi-provider AI system with automatic fallback:

```
AiManager → tries providers in order: Groq → OpenRouter → Ollama
```

| Provider | Model | Use |
|---|---|---|
| Groq | `llama-3.3-70b-versatile` | Primary (fast, free tier) |
| OpenRouter | `google/gemini-2.0-flash-001` | Backup |
| Ollama | `llama3` (local) | Local fallback |

**Configuration (`.env`):**
```env
AI_PROVIDER_ORDER=groq,openrouter,ollama
GROQ_API_KEY=gsk_...
OPENROUTER_API_KEY=sk-or-v1-...
OLLAMA_HOST=http://localhost:11434
```

### AI Features

#### Career Suggestion (`/ai/career-suggestion`)
- Input: student interests, skills, academic background
- Output: list of recommended career paths with descriptions

#### Career Details (`/ai/career-details`)
- Input: career name
- Output: detailed career description, required skills, salary range, growth prospects

#### Roadmap Generation (`/api/roadmaps/generate`)
- Input: `{ role: "Frontend Developer" }`
- Output: structured roadmap with phases, topics, concepts, resources
- Saved to `roadmaps` table with `user_id` (private to creator)

#### Course Generation (`/api/courses/generate`)
- Input: `{ role: "Data Science" }`
- Output: course with modules and lesson stubs (`[CONTENT_PENDING]`)
- Saved with `author = 'AI Architect'`, `created_by = user_id`
- Creator is auto-enrolled

#### Lesson Content Generation (`/ai/lesson-content`)
- Input: `{ course_id, lesson_title, module_title, course_title }`
- Output: rich content blocks (text, code, examples)
- Called on-demand when student opens a `[CONTENT_PENDING]` lesson
- Content cached in the course's `modules` JSON

#### Assessment Generation (`/ai/generate-assessment`)
- Input: `{ course_title, course_content, question_count }`
- Output: quiz with multiple-choice questions, correct answers, explanations
- Saved to `assessments` table

### Frontend AI (Google Generative AI)
The web frontend also uses `@google/genai` directly for:
- Career exploration details
- Real-time suggestions in the career explorer

---

## 10. Offline Support (Mobile)

### Architecture

```
Network available → fetch from API → cache in SQLite → return data
Network unavailable → read from SQLite cache → return cached data
Back online → flush pending mutations queue
```

### SQLite Database (`core/cache/local_db.dart`)

Two tables:

**`cache_entries`**
```sql
key       TEXT PRIMARY KEY
value     TEXT NOT NULL        -- JSON-encoded data
cached_at INTEGER NOT NULL     -- Unix timestamp
ttl_secs  INTEGER DEFAULT 3600 -- Time-to-live in seconds
```

**`sync_queue`**
```sql
id         INTEGER PRIMARY KEY AUTOINCREMENT
method     TEXT NOT NULL        -- POST or PUT
endpoint   TEXT NOT NULL
payload    TEXT                 -- JSON body
created_at INTEGER NOT NULL
```

### Cache TTLs

| Data | TTL |
|---|---|
| All courses (browse) | 1 hour |
| Enrolled courses | 30 minutes |
| Curated roadmaps | 1 hour |

### Sync Behavior

`syncPendingMutations(ref)` is called:
1. On `StudentHomeScreen` init (app startup)
2. When `connectivityProvider` emits `true` (device comes back online)

It iterates `sync_queue`, replays each POST/PUT request, and removes successful items.

### Connectivity Provider (`core/cache/connectivity_service.dart`)
```dart
connectivityProvider  // StreamProvider<bool> — true = online
isOnline()            // Future<bool> — one-shot check
```

---

## 11. Database Schema

### Core Tables

#### `users`
```sql
id, name, email, password (bcrypt), role (student|teacher|admin|bit),
account_status (active|pending|rejected), role_request, role_preference,
academic_year, student_id, department, graduation_year,
institution, years_experience, expertise_areas (JSON), qualifications (JSON),
certifications (JSON), bio, teaching_philosophy, languages (JSON),
linkedin_url, website_url, phone_number, profile_image,
xp, streak, created_at
```

#### `courses`
```sql
id, title, description, category, level (Beginner|Intermediate|Advanced),
modules (JSON), duration, author, created_by (FK users),
rating, enrolled_count, created_at
```

**Modules JSON structure:**
```json
[
  {
    "title": "Module Name",
    "lessons": [
      {
        "title": "Lesson Name",
        "duration": "15 min",
        "content": "...[CONTENT_PENDING] or rich JSON blocks"
      }
    ]
  }
]
```

#### `course_enrollments`
```sql
id, course_id (FK), user_id (FK), progress (0-100),
completed_lessons (JSON array of lesson titles), enrolled_at
UNIQUE KEY (course_id, user_id)
```

#### `curated_roadmaps`
```sql
id, title, description, category, difficulty_level (all|beginner|intermediate|advanced),
estimated_duration, phases (JSON), tags (JSON), thumbnail_url,
status (draft|published), views, enrollments, created_by (FK users), created_at
```

**Phases JSON (3-level structure):**
```json
[
  {
    "level": "beginner",
    "phases": [
      {
        "title": "Phase Name",
        "description": "...",
        "duration": "2 weeks",
        "topics": [
          { "title": "Topic", "description": "...", "concepts": ["..."] }
        ]
      }
    ]
  },
  { "level": "intermediate", "phases": [...] },
  { "level": "advanced", "phases": [...] }
]
```

#### `roadmaps` (AI-generated, user-private)
```sql
id, title, role, road_data (JSON), user_id (FK users), created_at
```

#### `educational_resources`
```sql
id, title, description, resource_type (document|video|link|note|article|tutorial),
file_path, external_url, category, tags (JSON), uploaded_by (FK users),
file_size, file_type, status (pending|approved|rejected),
course_id (FK), module_name, lesson_name, notes,
views, downloads, created_at
```

#### `careers`
```sql
id, title, description, icon, category, skills (JSON),
salary_range, growth_rate, status (draft|published),
created_by (FK users), created_at
```

#### `assessments`
```sql
id, course_id (FK), title, description, questions (JSON),
created_by (FK users), created_at
```

**Questions JSON:**
```json
[
  {
    "question": "What is...?",
    "options": ["A", "B", "C", "D"],
    "correct": 0,
    "explanation": "Because..."
  }
]
```

#### `assessment_submissions`
```sql
id, assessment_id (FK), user_id (FK), answers (JSON),
score, total_questions, submitted_at
```

#### `notifications`
```sql
id, user_id (FK), type, title, message, link, is_read, created_at
```

#### `support_messages`
```sql
id, user_id (FK nullable), guest_id, message, sender_type (user|admin),
created_at
```

#### `teacher_course_assignments`
```sql
id, teacher_id (FK users), course_id (FK courses),
status (pending|approved|rejected), notes, requested_at, approved_at, approved_by
UNIQUE KEY (teacher_id, course_id)
```

#### `password_resets`
```sql
id, email, token, created_at
-- Auto-created by AuthController if not exists
```

### Migration Files (run in order)

```
schema.sql                          → Base tables
role_approval_migration.sql         → Role approval workflow
update_user_roles.sql               → Add 'bit' role
bit_role_migration.sql              → BiT tables
course_system_migration.sql         → Course + enrollment system
assessments_migration.sql           → Assessments system
careers_migration.sql               → Careers table
admin_content_migration.sql         → Admin moderation
teacher_profile_migration.sql       → Teacher profile fields
enhanced_teacher_profile.sql        → Extended teacher profile
teacher_profile_settings_migration.sql → Teacher settings
teacher_materials_migration.sql     → Course materials
student_monitoring_system.sql       → Student tracking + feedback
resource_roadmap_matching.sql       → Resource-roadmap auto-matching
notifications_system.sql            → Notifications
support_chat_migration.sql          → Support messages
add_profile_image.sql               → Profile image column
seed_roadmaps.sql                   → Sample roadmap data
run_all_migrations.sql              → Master runner
```

---

## 12. Configuration & Deployment

### Backend `.env`

```env
# Database
DB_HOST=localhost
DB_PORT=3306
DB_NAME=careerguide
DB_USER=root
DB_PASS=

# JWT
JWT_SECRET=careerguide_secret_key_2024_development
JWT_EXPIRY=86400

# AI Providers
GROQ_API_KEY=gsk_...
GROQ_MODEL=llama-3.3-70b-versatile
OPENROUTER_API_KEY=sk-or-v1-...
OPENROUTER_MODEL=google/gemini-2.0-flash-001
OLLAMA_HOST=http://localhost:11434
OLLAMA_MODEL=llama3
AI_PROVIDER_ORDER=groq,openrouter,ollama

# App
APP_ENV=development
APP_URL=http://localhost/careerguide/backend/public
FRONTEND_URL=http://localhost:3000
```

### Web Frontend API Config

```typescript
// frontend/utils/apiConfig.ts
export const API_BASE_URL = 'http://localhost/careerguide/backend/api';
```

### Mobile API Config

```dart
// mobile/lib/core/constants/api_constants.dart
static const String _physicalDeviceIp = '10.161.174.178'; // ← update this
static const String _backendPath = '/careerguide/backend/api';
```

### Running the Project

**Backend (XAMPP):**
1. Start Apache and MySQL in XAMPP Control Panel
2. Place project at `C:\xampp\htdocs\careerguide\`
3. Run migrations: visit `http://localhost/careerguide/backend/public/run-all-migrations.php`
4. API available at `http://localhost/careerguide/backend/api`

**Web Frontend:**
```bash
cd frontend
npm install
npm run dev        # Development server (Vite)
npm run build      # Production build
```

**Mobile App:**
```bash
cd mobile
flutter pub get
flutter run        # Run on connected device/emulator
```

> For physical device: update `_physicalDeviceIp` in `api_constants.dart` to your PC's LAN IP (run `ipconfig` on Windows).

### File Upload Storage

Uploaded files are stored at:
```
backend/uploads/resources/documents/   ← PDFs, DOCX, etc.
backend/uploads/resources/videos/      ← MP4, MOV, etc.
```

Served via: `GET /api/uploads/serve?file=<filename>&type=resource`

The `buildFileUrl()` helper in `ResourceController` dynamically builds the correct URL using `$_SERVER['HTTP_HOST']`, so it works on both localhost (web) and LAN IP (mobile).

---

## 13. Known Limitations & Notes

| Area | Note |
|---|---|
| Teacher role (mobile) | Completely removed from mobile app. Teachers use web portal only. |
| AI lesson content | Generated on-demand per lesson. First open may be slow (1–3s). |
| Password reset email | Backend creates the reset token but email delivery requires SMTP config (not set up in dev). |
| OAuth | Google/GitHub OAuth routes exist in backend but are not fully implemented. |
| File size limit | 50MB per upload (configurable in `ResourceController::handleFileUpload`). |
| Offline sync | Only POST/PUT mutations are queued. DELETE operations are not queued offline. |
| SQLite cache | Cache is per-device. Clearing app data clears the cache. |
| BiT roadmap 5th Year | 5th Year academic year option only shown for Software Engineering and Other departments. |
| Phone validation | Only Ethiopian numbers accepted: Ethiotelecom (9xx) and Safaricom Ethiopia (7xx) after +251. |

---

*Documentation generated May 2026. For questions or updates, refer to the source code in `c:\xampp\htdocs\careerguide\`.*
