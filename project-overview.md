# Industry Project Report — FeedBack Analytics Platform

**Dear Aspirer,**
*Project report is an inherent component of your Industry Project.*

---

## Project Header

| Field | Details |
|---|---|
| **Industry Project Title** | FeedBack Analytics Platform |
| **Name of the Company** | Tata Consultancy Services (TCS) |
| **Name of the Institute** | *(Your Institute Name)* |

| Start Date | End Date | Total Effort (hrs.) | Project Environment | Tools Used |
|---|---|---|---|---|
| April 2026 | April 2026 | ~80 hrs | Windows 11, Node.js 20, MySQL 8 | VS Code, Docker, Postman, Git |

---

## Table of Contents

1. [Acknowledgements](#1-acknowledgements)
2. [Objective and Scope](#2-objective-and-scope)
3. [Problem Statement](#3-problem-statement)
4. [Existing Approaches](#4-existing-approaches)
5. [Approach / Methodology](#5-approach--methodology)
6. [Workflow](#6-workflow)
7. [Assumptions](#7-assumptions)
8. [Implementation](#8-implementation)
9. [Solution Design](#9-solution-design)
10. [Challenges & Opportunities](#10-challenges--opportunities)
11. [Reflections on the Project](#11-reflections-on-the-project)
12. [Recommendations](#12-recommendations)
13. [Outcome / Conclusion](#13-outcome--conclusion)
14. [Enhancement Scope](#14-enhancement-scope)
15. [Link to Code and Executable File](#15-link-to-code-and-executable-file)
16. [Research Questions and Responses](#16-research-questions-and-responses)
17. [References](#17-references)

---

## 1. Acknowledgements

I extend my sincere gratitude to Tata Consultancy Services (TCS) for providing this industry project opportunity. Special thanks to the mentors and peers who offered guidance throughout the development lifecycle. This project deepened my understanding of full-stack application development, REST API design, and role-based access control in real-world scenarios.

---

## 2. Objective and Scope

**Objective:** Build a full-stack web application that enables users to submit product feedback and allows administrators to analyze aggregated feedback data through interactive dashboards.

**Scope:**
- User registration, login, and JWT-based session management
- Product catalog with user feedback submission (rating + comment)
- Admin panel with analytics: rating distribution, feedback trends, product-level analysis
- Role-based access control (User / Admin)
- Containerised deployment using Docker + MySQL

---

## 3. Problem Statement

Organizations often lack a centralized, structured mechanism to collect and analyze product feedback. Feedback is scattered across emails, spreadsheets, and informal channels — making it difficult to identify trends, low-performing products, or recurring issues. There is a need for a unified platform where end users can submit structured feedback and administrators can derive actionable insights from aggregated data.

---

## 4. Existing Approaches

| Approach | Limitation |
|---|---|
| Spreadsheet-based surveys (Google Forms) | No real-time analytics; no role separation |
| Generic CRM tools (Salesforce, Zendesk) | Expensive; over-engineered for small teams |
| Manual reporting via email | Error-prone; no single source of truth |
| Third-party survey tools (SurveyMonkey) | Limited customization; no product-level drill-down |

The FeedBack Analytics platform addresses these gaps with a purpose-built, lightweight, and self-hosted solution.

---

## 5. Approach / Methodology

**Architecture:** Decoupled full-stack (REST API + SPA)

### Tools and Technologies

| Layer | Technology |
|---|---|
| **Frontend** | React 18, Vite, Tailwind CSS, Recharts, Lucide Icons |
| **Backend** | Node.js, Express.js |
| **ORM** | Sequelize v6 |
| **Database** | MySQL 8.0 (Dockerized) |
| **Auth** | JWT (Access + Refresh tokens), bcryptjs |
| **Containerization** | Docker, Docker Compose |
| **Testing** | Jest, Supertest |
| **Dev Tools** | Nodemon, Postman, VS Code |

**Methodology:** Agile-inspired iterative development — schema design → API layer → frontend pages → integration → testing.

---

## 6. Workflow

```
User/Admin
    │
    ▼
Frontend (React + Vite) ─── HTTP/REST ──► Backend (Express.js)
                                                │
                               ┌────────────────┴───────────────┐
                               ▼                                ▼
                         Auth Routes                     Admin Routes
                     (/api/auth/*)               (/api/admin/products, users, feedbacks)
                               │
                               ▼
                         MySQL 8.0 (Docker)
                         [Users | Products | Feedbacks]
```

**Key Flows:**
1. **User Flow:** Register → Login → Browse Products → Submit Feedback → View History
2. **Admin Flow:** Admin Login → Dashboard Overview → Manage Products/Users/Feedbacks → View Analytics

---

## 7. Assumptions

- Single organization deployment (not multi-tenant)
- Admins are created via seeding scripts; not via public registration
- Each user can submit only one feedback per product
- Internet access is available for Docker image pulls and npm package installation
- MySQL 8.0 is the only supported database dialect

---

## 8. Implementation

### Data Collection
- User feedback collected via a structured form: **Star Rating (1–5)** + **Comment (text)**
- Product data seeded via `src/seed.js` with 8 sample products across 5 categories

### Database Schema

**Users Table**

| Column | Type | Notes |
|---|---|---|
| id | INT (PK, AI) | Primary key |
| name | VARCHAR | Required |
| email | VARCHAR (unique) | Validated |
| password | VARCHAR | bcrypt hashed |
| role | ENUM('user','admin') | Default: 'user' |
| refreshToken | TEXT | JWT refresh token |

**Products Table**

| Column | Type | Notes |
|---|---|---|
| id | INT (PK, AI) | — |
| name | VARCHAR | Required |
| category | VARCHAR | — |
| description | TEXT | — |
| imageUrl | VARCHAR | — |
| createdBy | INT (FK → Users) | Admin who created it |

**Feedbacks Table**

| Column | Type | Notes |
|---|---|---|
| id | INT (PK, AI) | — |
| userId | INT (FK → Users) | — |
| productId | INT (FK → Products) | — |
| rating | INT (1–5) | Required |
| comment | TEXT | Required |

### Processing Steps
1. User submits rating + comment → validated server-side → stored in `Feedbacks`
2. Admin dashboard queries aggregate functions: `AVG(rating)`, `COUNT(id)` grouped by product/date
3. Charts rendered client-side using **Recharts** (bar, pie, line)

### API Endpoints Summary

| Method | Route | Auth | Description |
|---|---|---|---|
| POST | `/api/auth/register` | Public | Register new user |
| POST | `/api/auth/login` | Public | User login |
| POST | `/api/auth/admin-login` | Public | Admin login |
| GET | `/api/products` | User/Admin | Get all products |
| POST | `/api/feedbacks` | User | Submit feedback |
| GET | `/api/admin/overview` | Admin | Dashboard stats |
| GET | `/api/admin/users` | Admin | List all users |
| GET | `/api/admin/products` | Admin | Manage products |
| GET | `/api/admin/feedbacks` | Admin | All feedbacks |

---

## 9. Solution Design

### Architecture Diagram

```
┌───────────────────────────────────┐
│         Frontend (React/Vite)      │
│  ┌─────────┐  ┌─────────────────┐ │
│  │  User   │  │  Admin Panel    │ │
│  │Dashboard│  │ (Dashboard,     │ │
│  │         │  │  Products,      │ │
│  │Feedback │  │  Users,         │ │
│  │Submit   │  │  Feedbacks,     │ │
│  └────┬────┘  │  Analytics)     │ │
│       │       └────────┬────────┘ │
└───────┼────────────────┼──────────┘
        │ REST API (JWT) │
┌───────▼────────────────▼──────────┐
│         Backend (Express.js)       │
│  Auth │ Products │ Feedbacks │ Admin│
│       Middleware (JWT Role Guard)  │
└───────────────────┬───────────────┘
                    │ Sequelize ORM
           ┌────────▼────────┐
           │   MySQL 8.0     │
           │  (Docker:3307)  │
           └─────────────────┘
```

### Security Design
- **Access Token:** Short-lived JWT (15 min) for API requests
- **Refresh Token:** Long-lived JWT stored in DB for session renewal
- **Password Hashing:** bcryptjs with salt rounds = 10
- **Role Guard Middleware:** Protects `/api/admin/*` routes — rejects non-admin tokens

---

## 10. Challenges & Opportunities

| Challenge | Resolution |
|---|---|
| JWT refresh token rotation | Stored refresh token in DB; validated on each refresh call |
| MySQL Docker connectivity | Used `DB_HOST=feedback_db` service name in Docker network |
| Role-based route protection | Created `authMiddleware` + `adminMiddleware` chain |
| CORS between Vite (5173) and Express (5000) | Configured `cors()` with explicit `CLIENT_URL` origin |

**Opportunities:**
- Sentiment analysis using NLP on feedback comments (AI layer)
- Email notifications for low-rated product alerts
- Export analytics as PDF/CSV reports

---

## 11. Reflections on the Project

This project reinforced the importance of designing a clean API contract before building the frontend. Implementing JWT refresh token rotation was the most technically challenging aspect, requiring careful state management on both server and client. Working with Sequelize associations (hasMany / belongsTo) and aggregate SQL queries through the ORM strengthened my understanding of relational data modeling.

The role-based access control system — while simple in this implementation — mirrors real-world enterprise patterns and gave practical exposure to security-first design thinking.

---

## 12. Recommendations

- **For Production:** Migrate to HTTPS, use Redis for refresh token store instead of DB column
- **Scalability:** Add pagination to all list endpoints; implement DB indexing on `productId`, `userId`
- **Security:** Implement rate-limiting on `/api/auth/*` endpoints to prevent brute-force attacks
- **UX:** Add email verification during registration for account integrity
- **Testing:** Increase unit test coverage for controllers beyond the current baseline

---

## 13. Outcome / Conclusion

The FeedBack Analytics Platform successfully delivers a production-grade, role-based web application that centralizes product feedback collection and analysis. The system demonstrates a complete full-stack implementation with secure authentication, relational data modeling, REST API design, and interactive data visualization. All core features — user registration/login, feedback submission, admin dashboard with charts, user/product management — are fully functional and integrated.

---

## 14. Enhancement Scope

| Enhancement | Priority |
|---|---|
| NLP-based sentiment tagging on comments | High |
| CSV/PDF export for admin reports | High |
| Email alerts for products with avg rating < 3 | Medium |
| Multi-language support (i18n) | Medium |
| Mobile app (React Native) | Low |
| OAuth2 social login (Google) | Low |
| Real-time feedback notifications (WebSockets) | Medium |

---

## 15. Link to Code and Executable File

| Resource | Link |
|---|---|
| **GitHub Repository** | `https://github.com/Akhil18-kuttu/FeedBack-Analytics` |
| **Local Dev — Backend** | `http://localhost:5000/api/health` |
| **Local Dev — Frontend** | `http://localhost:5173` |
| **Admin Login** | Email: `admin@tcs.com` / Password: `Admin@123` |
| **User Login** | Email: `akhil@example.com` / Password: `User@123` |

**Run Locally:**
```bash
# Start DB
docker compose up -d feedback_db

# Start backend
cd backend && npm run dev

# Start frontend
cd frontend && npm run dev
```

---

## 16. Research Questions and Responses

**Q1: How can feedback data be used to improve product quality?**
> By aggregating star ratings and analyzing comment sentiment over time, teams can identify declining product satisfaction trends and prioritize improvements. The platform's bar/line/pie charts make this immediately actionable for product managers.

**Q2: What is the most effective way to implement role-based access in a REST API?**
> Using JWT claims to encode the user role, combined with server-side middleware validation on protected routes, provides a stateless, scalable, and secure RBAC solution. This approach avoids session-based overhead while maintaining strong access control.

**Q3: How does containerization benefit project deployment?**
> Docker Compose ensures environment consistency across development and production. The MySQL service, backend, and frontend each run in isolated containers with defined dependencies, eliminating "works on my machine" issues.

---

## 17. References

1. Express.js Documentation — https://expressjs.com/
2. Sequelize ORM Documentation — https://sequelize.org/
3. React Documentation — https://react.dev/
4. JSON Web Tokens (JWT) — https://jwt.io/
5. Docker Documentation — https://docs.docker.com/
6. Recharts Library — https://recharts.org/
7. Tailwind CSS Documentation — https://tailwindcss.com/
8. bcryptjs npm package — https://www.npmjs.com/package/bcryptjs
9. MySQL 8.0 Reference Manual — https://dev.mysql.com/doc/refman/8.0/en/
10. Vite Build Tool — https://vitejs.dev/

---

*Note: This project report is the sole source for evaluating the quality of the project.*
