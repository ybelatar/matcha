Product Requirements Document (PRD): Matcha Dating Web Application

1. Project Overview

Project Name: Matcha
Tagline: "Because love, too, can be industrialized."
Description: Matcha is a full-stack web application that facilitates romantic connections through user registration, profile management, intelligent matching, browsing, searching, real-time chat, and notifications. It emphasizes security, mobile-friendliness, and intelligent algorithms for profile suggestions based on location, interests, and fame rating. The app must handle user interactions like "likes" (mutual likes enable "connections" for chatting), profile views, blocking, and reporting.

Goals:

- Enable users to find and connect with potential partners artificially via algorithms.
- Ensure a seamless, error-free experience across browsers (latest Firefox and Chrome).
- Support at least 500 distinct profiles in the database for evaluation.
- Be secure, mobile-responsive, and real-time for key features (e.g., chat and notifications with ≤10s delay).
  Target Users: Adults seeking romantic connections (heterosexual, homosexual, bisexual).
  Key Metrics for Success: Zero errors/warnings/notices; no security vulnerabilities; functional on mobile; passes peer-evaluation criteria.

2. Scope and Constraints

- Mandatory Features: Registration/sign-in, user profiles, browsing, search, profile viewing, chat, notifications.
- Bonus Features: Only evaluated if mandatory is perfect; includes OmniAuth, photo gallery, interactive map, video/audio chat, date scheduling.
- General Constraints:
  - Error-free (no server/client-side errors, warnings, notices).
  - Any programming language allowed; micro-frameworks permitted (e.g., no full ORMs, validators, or user managers).
  - Relational or graph-oriented DB (e.g., PostgreSQL, Neo4j); manual queries required (no ORMs).
  - UI libraries allowed (e.g., React, Vue, Bootstrap).
  - Mobile-friendly layout (header, main, footer).
  - Compatible with latest Firefox/Chrome.
  - Minimum 500 profiles in DB for evaluation.
  - Web server: Apache, Nginx, or built-in.
  - All forms validated; secure against common vulnerabilities (e.g., SQL injection, XSS, plain-text passwords).
- Out of Scope: Non-dating features (e.g., payments, ads); non-specified integrations.

3. Technical Stack Recommendations

Based on spec suggestions and freedoms, here's a recommended stack (flexible; team can choose alternatives like Ruby/Sinatra or Python/Flask):

- Backend: Node.js with Express (micro-framework; includes router but no ORM/validators).
- Frontend: React.js (for dynamic UI) + Bootstrap (for responsive design).
- Database: PostgreSQL (relational; supports geospatial queries for location).
- Real-Time Features: WebSockets via Socket.io (for chat and notifications; ≤10s delay).
- Email Service: Nodemailer or SendGrid (for verification/reset emails).
- Authentication: JWT (JSON Web Tokens) for sessions; bcrypt for password hashing.
- Geolocation: Google Maps API or IP-based fallback (e.g., ipapi.co) for location detection.
- Other Tools: Git for version control; .env for secrets (exclude from Git); Prettier for code formatting (print width 80).
- Environment: Development on local machines; production-like setup with Docker for consistency.

4. Database Design

Use PostgreSQL for relational structure with geospatial extensions (PostGIS for location queries). No ORM—write manual SQL queries (build a lightweight query helper library if needed).

Schema Overview (Tables and Key Fields):

- Users (Core table; ~500 rows minimum for eval):

      - id: SERIAL PRIMARY KEY
      - username: VARCHAR(50) UNIQUE
      - email: VARCHAR(100) UNIQUE
      - first_name: VARCHAR(50)
      - last_name: VARCHAR(50)
      - password_hash: VARCHAR(255) (bcrypt hashed)
      - gender: ENUM('male', 'female', 'other')
      - sexual_preference: ENUM('heterosexual', 'homosexual', 'bisexual') DEFAULT 'bisexual'
      - biography: TEXT
      - fame_rating: INTEGER DEFAULT 0 (algorithmically calculated; e.g., based on likes/views/connections)
      - location: POINT (geospatial; e.g., latitude/longitude)
      - manual_location: POINT (optional override)
      - is_verified: BOOLEAN DEFAULT FALSE
      - last_online: TIMESTAMP
      - created_at: TIMESTAMP DEFAULT NOW()
      - updated_at: TIMESTAMP

- Interests (Tags; reusable):

      - id: SERIAL PRIMARY KEY
      - tag_name: VARCHAR(50) UNIQUE

- User_Interests (Many-to-Many junction):

      - user_id: INTEGER REFERENCES Users(id)
      - interest_id: INTEGER REFERENCES Interests(id)

- Photos (Up to 5 per user; one profile pic):

      - id: SERIAL PRIMARY KEY
      - user_id: INTEGER REFERENCES Users(id)
      - url: VARCHAR(255)
      - is_profile_pic: BOOLEAN DEFAULT FALSE
      - uploaded_at: TIMESTAMP

- Likes (For "likes" and connections):

      - id: SERIAL PRIMARY KEY
      - liker_id: INTEGER REFERENCES Users(id)
      - likee_id: INTEGER REFERENCES Users(id)
      - created_at: TIMESTAMP
      - UNIQUE(liker_id, likee_id)  // Prevent duplicates

- Views (Profile visit history):

      - id: SERIAL PRIMARY KEY
      - viewer_id: INTEGER REFERENCES Users(id)
      - viewee_id: INTEGER REFERENCES Users(id)
      - viewed_at: TIMESTAMP

- Blocks (Blocked users):

      - id: SERIAL PRIMARY KEY
      - blocker_id: INTEGER REFERENCES Users(id)
      - blockee_id: INTEGER REFERENCES Users(id)
      - created_at: TIMESTAMP
      - UNIQUE(blocker_id, blockee_id)

- Reports (Fake account reports):

      - id: SERIAL PRIMARY KEY
      - reporter_id: INTEGER REFERENCES Users(id)
      - reported_id: INTEGER REFERENCES Users(id)
      - reason: TEXT
      - created_at: TIMESTAMP

- Messages (Chat history):

      - id: SERIAL PRIMARY KEY
      - sender_id: INTEGER REFERENCES Users(id)
      - receiver_id: INTEGER REFERENCES Users(id)
      - content: TEXT
      - sent_at: TIMESTAMP
      - is_read: BOOLEAN DEFAULT FALSE

- Notifications (Real-time events):

      - id: SERIAL PRIMARY KEY
      - user_id: INTEGER REFERENCES Users(id)
      - type: ENUM('like', 'view', 'message', 'match', 'unlike')
      - from_user_id: INTEGER REFERENCES Users(id)
      - content: TEXT
      - is_read: BOOLEAN DEFAULT FALSE
      - created_at: TIMESTAMP

  Queries: All manual (e.g., SELECT with JOINs for matching; geospatial queries like ST_Distance for proximity).

5. Feature Breakdown

Mandatory Features

IV.1 Registration and Signing-in

- Users register with: email, username, first/last name, password (reject common words; hash with bcrypt).
- Send verification email with unique link (e.g., JWT token).
- Login with username/password; logout from any page.
- Password reset via email.
- Validation: All fields required; email unique/valid; password strong.

IV.2 User Profile

- Complete post-login: gender, preferences, bio, tags (reusable from DB), up 5 photos (one profile pic).
- Edit anytime (including name/email).
- View profile visitors and likers.
- Public fame rating (define as: likes received \* 2 + views + connections; update algorithmically).
- GPS location (auto-detect via JS/IP; fallback to IP if opted out; manual override).

IV.3 Browsing

- Suggested profiles: Filter by preferences (e.g., hetero women see men; default bisexual).
- Intelligent sorting: Proximity (geospatial), shared tags (COUNT JOIN), fame rating.
- Prioritize same area.
- Sortable/filterable by: age (derived from optional birthdate?), location, fame, tags.

IV.4 Research (Advanced Search)

- Criteria: Age range, fame range, location, tags.
- Results sortable/filterable like browsing.

IV.5 Profile View

- Display all info except email/password.
- Log views in DB.
- Actions: Like/unlike (mutual = connected), check fame, online status/last connection, report fake, block (hides from searches/notifications).
- Show if liked/connected; disable chat if unliked/blocked.
- Require profile pic to like others.

IV.6 Chat

- Real-time (WebSockets; ≤10s delay) for connected users.
- Show new messages from any page (e.g., badge/counter).

IV.7 Notifications

- Real-time (≤10s) for: like, view, message, match (mutual like), unlike.
- Show unread count from any page (e.g., bell icon).

Bonus Features (Implement only after mandatory is perfect)

- OmniAuth (e.g., Google/Facebook login).
- Photo gallery: Drag-drop upload, edit (crop/rotate/filters) via library like Cropper.js.
- Interactive map: JS-based (Leaflet/Google Maps) with precise GPS.
- Video/audio chat: WebRTC integration.
- Date scheduling: Calendar UI for events.

6. Security and Compliance

- Mandatory Protections:
  - Hash passwords (bcrypt).
  - Prevent SQL injection (prepared statements).
  - Validate/sanitize inputs (no XSS/HTML injection).
  - Secure file uploads (photos: size/type limits; store securely).
  - No plain-text credentials; use .env (gitignore).
  - Rate limiting on login/reset.
  - JWT for auth; HTTPS recommended.
- Additional: CSRF tokens, input validation libraries (e.g., Joi for Node), secure cookies.

7. Development Process

Phased approach with tasks. Total estimated timeline: 4-6 weeks (assuming 4 engineers).

Phase 1: Planning and Setup (Week 1; Effort: Low)

- Task 1.1 (P1): Design DB schema (as above); write migration scripts (manual SQL). (2 days)
- Task 1.2 (P2): Seed DB with 500+ fake profiles (use Faker.js; include varied data for testing). (1 day)
- Task 1.3 (P3): Define fame rating algorithm (e.g., cron job to update nightly). (1 day)

Phase 2: Backend Development (Weeks 1-2; Effort: High)

- Task 2.1 (P1): Implement auth routes: Register (email verification), login, logout, password reset. (3 days)
- Task 2.2 (P1): User profile endpoints: Create/update (including photos upload/validation), get visitors/likers. (2 days)
- Task 2.3 (P1): Location services: Auto-detect (IP/JS), manual override; geospatial queries. (2 days)
- Task 2.4 (P1): Browsing/Search endpoints: Intelligent matching (queries with filters/sorts). (3 days)
- Task 2.5 (P1): Profile view actions: Like/unlike, block, report; log views. (2 days)
- Task 2.6 (P1): Real-time: Set up Socket.io for chat and notifications. (2 days)
- Task 2.7 (P2): Implement security: Hashing, sanitization, validation on all endpoints. (Throughout; 2 days audit)

Phase 3: Frontend Development (Weeks 2-3; Effort: High)

- Task 3.1 (P1): Build responsive layout (header/footer/main) with Bootstrap/React. (2 days)
- Task 3.2 (P1): Auth pages: Register/login/reset forms with validation. (2 days)
- Task 3.3 (P1): Profile management: Edit form, photo upload (up to 5), tag selector (autocomplete from DB). (3 days)
- Task 3.4 (P1): Browsing/Search UI: Lists with filters/sorts; integrate API calls. (3 days)
- Task 3.5 (P1): Profile view page: Display info, actions (like/block), online status. (2 days)
- Task 3.6 (P1): Chat UI: Real-time messaging with Socket.io; new message badges. (2 days)
- Task 3.7 (P1): Notifications: Real-time updates; unread counter. (1 day)
- Task 3.8 (P2): Mobile responsiveness testing (media queries). (1 day)

Phase 4: Integration and Bonus (Weeks 3-4; Effort: Medium)

- Task 4.1 (P1): Integrate frontend with backend APIs (e.g., Axios for calls). (2 days)
- Task 4.2 (P1): Implement real-time features end-to-end (chat/notifications). (2 days)
- Task 4.3 (P2): Add bonuses if mandatory is complete (e.g., OmniAuth, photo editing). (3-5 days optional)
- Task 4.4 (P3): Optimize performance (e.g., pagination for lists). (1 day)

Phase 5: Testing and Quality Assurance (Week 4; Effort: Medium)

- Task 5.1 (P1): Unit tests: Backend (e.g., Jest for APIs, queries). (2 days)
- Task 5.2 (P1): Integration tests: End-to-end (e.g., Cypress for UI flows). (2 days)
- Task 5.3 (P1): Security audits: Scan for vulnerabilities (e.g., OWASP tools); test injections. (1 day)
- Task 5.4 (P1): Browser testing: Firefox/Chrome; mobile emulation. (1 day)
- Task 5.5 (P1): Load testing: Simulate 500 users; ensure no errors/warnings. (1 day)

Phase 6: Deployment and Submission (End of Week 4; Effort: Low)

- Task 6.1 (P1): Set up web server (e.g., Nginx for production-like). (1 day)
- Task 6.2 (P1): Deploy to local/test server; ensure .env is local-only. (1 day)
- Task 6.3 (P1): Prepare for peer-evaluation: Document setup, run error-free demo. (1 day)

8. Risks and Dependencies

- Risks: Real-time delays >10s (mitigate with efficient WebSockets); geolocation accuracy (use fallbacks).
- Dependencies: External APIs (email, maps) require keys in .env.
- Milestones: End of Phase 2: Backend APIs ready; End of Phase 4: Full app functional.
