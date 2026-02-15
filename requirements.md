# Speech Play - Requirements Document
## The Therapy Compliance Engine

---

## 1. Project Overview

### 1.1 Brief Description

Speech Play is a therapy compliance platform designed for Indian speech-language pathology clinics. The system combines a gamified mobile application, WhatsApp integration, and a therapist dashboard to transform home practice compliance from 30% to 80%+ through automated reminders, AI-powered speech validation, and real-time progress tracking.

### 1.2 Problem Statement

Children receiving speech therapy spend only 1 hour per week with therapists but 167 hours at home. Current paper-based homework systems suffer from:

- Lost worksheets and unclear instructions
- No accountability or progress tracking
- 30% compliance rate leading to extended therapy duration
- Parents lack visibility into child's progress
- Therapists cannot identify struggling students until next session
- India faces severe SLP shortage (1:2000+ ratio) requiring efficiency tools

### 1.3 Solution Overview

Speech Play provides:

- **Gamified Mobile App**: Children complete speech exercises as games with instant AI feedback
- **Offline-First Architecture**: Works on 2G networks, syncs when online
- **WhatsApp Integration**: Automated reminders and progress reports via India's primary messaging platform
- **Therapist Dashboard**: Real-time compliance tracking, bulk assignment, and progress analytics
- **Parent Interface**: PWA-based progress monitoring without app installation
- **Multilingual Support**: Hindi, Tamil, Telugu, Marathi, Bengali with regional voice prompts

### 1.4 Target Users

**Primary Users:**
- Speech-Language Pathologists (SLPs) at Indian clinics
- Parents of children (ages 3-12) in speech therapy
- Children undergoing speech therapy

**Secondary Users:**
- Clinic administrators
- Therapy coordinators

---

## 2. Functional Requirements

### 2.1 Therapist Dashboard Requirements

#### FR-T1: Exercise Library Management
**User Story**: As a therapist, I want to access a comprehensive library of speech exercises categorized by phoneme, difficulty, and language, so I can quickly assign appropriate homework.

**Acceptance Criteria:**
- Library contains 500+ exercises covering all phonemes
- Filters by: phoneme, difficulty (beginner/intermediate/advanced), language, age group
- Search functionality with autocomplete
- Preview exercise with audio/visual demonstration
- Custom exercise creation with template support
- Bulk import from CSV/Excel

#### FR-T2: Assignment Creation
**User Story**: As a therapist, I want to assign exercises to individual students or groups with customizable schedules, so I can personalize therapy plans.

**Acceptance Criteria:**
- Assign to individual student or multiple students simultaneously
- Set frequency (daily, 3x/week, custom)
- Set duration (1 week, 2 weeks, ongoing)
- Add custom instructions and notes
- Set target accuracy threshold (e.g., 80% correct)
- Schedule assignments for future dates

#### FR-T3: Real-Time Compliance Dashboard
**User Story**: As a therapist, I want to see which students practiced today with accuracy scores, so I can identify who needs intervention.

**Acceptance Criteria:**
- Dashboard shows: student name, last practice time, completion %, accuracy %
- Color-coded status: green (completed), yellow (partial), red (not started)
- Filter by: date range, compliance status, accuracy range
- Sort by: name, compliance rate, accuracy, last activity
- Export data to PDF/Excel
- Refresh data in real-time (WebSocket updates)

#### FR-T4: Progress Analytics
**User Story**: As a therapist, I want to view longitudinal progress charts for each student, so I can measure therapy effectiveness.

**Acceptance Criteria:**
- Line charts showing accuracy trends over time
- Bar charts comparing weekly practice frequency
- Phoneme-specific progress breakdown
- Comparison against clinic averages
- Predictive analytics flagging at-risk students
- Downloadable progress reports for parents

#### FR-T5: Communication Tools
**User Story**: As a therapist, I want to send messages to parents via WhatsApp, so I can provide feedback without phone calls.

**Acceptance Criteria:**
- Send individual or bulk messages via WhatsApp Business API
- Pre-defined message templates (encouragement, reminders, concerns)
- View message delivery status
- Receive parent replies in dashboard
- Attach progress reports to messages

### 2.2 Child Mobile App Requirements

#### FR-C1: Gamified Exercise Interface
**User Story**: As a child, I want to play fun games that help me practice speech, so I enjoy doing homework.

**Acceptance Criteria:**
- Each exercise presented as interactive game (e.g., "Say KA to move car")
- Visual feedback: animations, particle effects on success
- Audio feedback: encouraging sounds, character voices
- Progress bar showing exercise completion
- Retry mechanism for incorrect attempts (max 3 tries)
- Skip option (marked as incomplete)

#### FR-C2: Voice Recognition & Validation
**User Story**: As a child, I want the app to tell me if I said the sound correctly, so I know I'm practicing right.

**Acceptance Criteria:**
- Record audio using device microphone
- Process audio through Amazon Transcribe (Indian languages)
- Compare transcription against expected phoneme using ML model
- Provide instant feedback: correct/incorrect with visual cue
- Show accuracy score (0-100%)
- Replay recorded audio for self-review

#### FR-C3: Rewards System
**User Story**: As a child, I want to earn stars and unlock levels, so I feel motivated to practice daily.

**Acceptance Criteria:**
- Earn stars for: completing exercises (1 star), high accuracy (2 stars), daily streaks (bonus stars)
- Unlock badges: "3-Day Streak", "Perfect Score", "100 Exercises"
- Level progression system (Level 1-50)
- Virtual rewards: character customization, themes, stickers
- Leaderboard (optional, privacy-controlled by parent)

#### FR-C4: Offline Capability
**User Story**: As a child in a low-connectivity area, I want to practice exercises without internet, so I can complete homework anytime.

**Acceptance Criteria:**
- Download all assigned exercises on WiFi
- Store exercises locally (IndexedDB/SQLite)
- Practice fully offline with voice recognition
- Queue completed exercises for upload
- Sync automatically when online
- Show sync status indicator

#### FR-C5: Multilingual Interface
**User Story**: As a child who speaks Tamil, I want instructions in Tamil, so I understand what to do.

**Acceptance Criteria:**
- Support languages: Hindi, Tamil, Telugu, Marathi, Bengali, English
- Voice prompts in selected language
- Text instructions in selected language
- Language selection on first launch
- Parent can change language in settings

### 2.3 Parent Interface Requirements

#### FR-P1: Progress Dashboard (PWA)
**User Story**: As a parent, I want to see my child's weekly practice completion without installing an app, so I can monitor progress easily.

**Acceptance Criteria:**
- Accessible via web browser (no app install)
- Shows: weekly completion %, daily practice log, accuracy trends
- Calendar view with color-coded practice days
- Detailed exercise-level breakdown
- Works on mobile and desktop browsers
- Offline viewing of cached data

#### FR-P2: WhatsApp Notifications
**User Story**: As a parent, I want daily WhatsApp reminders, so I remember to help my child practice.

**Acceptance Criteria:**
- Receive daily reminder at customizable time (default 6 PM)
- Reminder includes: child name, pending exercises, quick link to app
- Weekly summary every Sunday with: completion %, accuracy %, therapist feedback
- Motivational messages on milestones
- Opt-out option for reminders

#### FR-P3: Communication with Therapist
**User Story**: As a parent, I want to message the therapist via WhatsApp, so I can ask questions conveniently.

**Acceptance Criteria:**
- Two-way messaging via WhatsApp Business API
- View therapist's messages in PWA
- Receive notifications for new therapist messages
- Attach photos/videos of child practicing
- Message history stored for 90 days

### 2.4 WhatsApp Integration Requirements

#### FR-W1: Automated Reminders
**User Story**: As the system, I want to send automated reminders via WhatsApp, so parents receive timely notifications.

**Acceptance Criteria:**
- Daily reminders sent at parent-specified time
- Reminder content: child name, exercise count, motivational message
- Include deep link to open app directly to exercises
- Delivery confirmation tracking
- Fallback to SMS if WhatsApp delivery fails

#### FR-W2: Weekly Summary Reports
**User Story**: As a parent, I want weekly progress summaries via WhatsApp, so I understand my child's improvement.

**Acceptance Criteria:**
- Sent every Sunday at 10 AM
- Contains: completion rate, accuracy %, exercises completed, therapist comments
- Visual chart image embedded in message
- Comparison with previous week
- Actionable recommendations

---

## 3. Non-Functional Requirements

### 3.1 Performance Requirements

#### NFR-P1: Response Time
- API response time: < 200ms (p95)
- Dashboard load time: < 2 seconds
- Mobile app launch time: < 1.5 seconds
- Voice recognition processing: < 3 seconds
- Real-time sync latency: < 500ms

#### NFR-P2: Scalability
- Support 10,000 concurrent users
- Handle 100,000 daily exercise submissions
- Process 50,000 voice recordings per day
- Scale to 500 clinics, 50,000 children within 12 months
- Auto-scaling based on load (AWS ECS/Lambda)

#### NFR-P3: Availability
- System uptime: 99.5% (excluding planned maintenance)
- Planned maintenance windows: Sunday 2-4 AM IST
- Graceful degradation: offline mode maintains core functionality
- Database replication for disaster recovery (RTO: 4 hours, RPO: 1 hour)

### 3.2 Security Requirements

#### NFR-S1: Authentication & Authorization
- Multi-factor authentication for therapists
- OAuth 2.0 with AWS Cognito
- Role-based access control (RBAC): Admin, Therapist, Parent, Child
- Session timeout: 30 minutes (therapist), 7 days (parent/child)
- Password requirements: 8+ characters, uppercase, lowercase, number, special char

#### NFR-S2: Data Encryption
- Data at rest: AES-256 encryption (S3, RDS, DynamoDB)
- Data in transit: TLS 1.3
- Voice recordings encrypted before storage
- PII data tokenization for analytics

#### NFR-S3: Compliance
- GDPR compliance for data privacy
- Indian IT Act 2000 compliance
- HIPAA-equivalent standards for health data
- Data residency: all data stored in AWS Mumbai region
- Right to deletion: complete data removal within 30 days

### 3.3 Offline Capability Requirements

#### NFR-O1: Offline Functionality
- Download exercises for 7 days in advance
- Store up to 100 exercises locally (approx. 500 MB)
- Voice recognition works offline using on-device model
- Queue up to 50 completed exercises for sync
- Automatic sync when WiFi detected

#### NFR-O2: Sync Mechanism
- Conflict resolution: server wins for assignments, client wins for completions
- Incremental sync (only changed data)
- Background sync using service workers (PWA)
- Sync status indicator in UI
- Retry failed syncs with exponential backoff

### 3.4 Multilingual Support Requirements

#### NFR-M1: Language Coverage
- Support 6 languages: Hindi, Tamil, Telugu, Marathi, Bengali, English
- All UI text translated
- Voice prompts recorded by native speakers
- Right-to-left text support (future: Urdu)

#### NFR-M2: Speech Recognition Accuracy
- Accuracy target: 85%+ for supported languages
- Support Indian English accents
- Handle code-switching (mixing languages)
- Continuous model improvement based on user data

### 3.5 Accessibility Requirements

#### NFR-A1: WCAG 2.1 Compliance
- Level AA compliance for all interfaces
- Screen reader support (NVDA, JAWS, TalkBack)
- Keyboard navigation for all features
- Color contrast ratio: 4.5:1 minimum
- Text resizing up to 200% without loss of functionality

#### NFR-A2: Inclusive Design
- Large touch targets (minimum 44x44 pixels)
- Simple language (reading level: Grade 5)
- Visual and audio feedback for all actions
- Support for motor impairments (voice-only navigation)

### 3.6 Low-Bandwidth Optimization

#### NFR-B1: Network Requirements
- Functional on 2G networks (50 kbps)
- Optimized for 3G (384 kbps) and 4G
- Image compression: WebP format, < 50 KB per image
- Audio compression: Opus codec, 32 kbps
- Lazy loading for non-critical resources

#### NFR-B2: Data Usage
- Average exercise download: < 200 KB
- Daily data usage: < 5 MB (with sync)
- Weekly data usage: < 30 MB
- Option to download on WiFi only

---

## 4. Technical Requirements

### 4.1 System Architecture

#### TR-A1: Microservices Architecture
- Service decomposition: User Service, Exercise Service, Progress Service, Notification Service, AI Service
- API Gateway for routing and load balancing
- Event-driven architecture using SNS/SQS
- Containerized deployment using Docker + ECS

#### TR-A2: Technology Stack

**Backend:**
- Java Spring Boot 3.x (REST APIs)
- AWS Lambda (serverless functions for notifications, data processing)
- AWS API Gateway (REST + WebSocket)
- AWS Cognito (authentication)

**Frontend:**
- React Native 0.72+ (mobile app - iOS/Android)
- React.js 18+ (therapist dashboard)
- Progressive Web App (parent interface)
- AWS Amplify (hosting, CI/CD)

**AI/ML:**
- Amazon Transcribe (speech-to-text, Indian languages)
- Amazon SageMaker (auto-grading ML model)
- Amazon Personalize (smart reminders)

**Databases:**
- Amazon DynamoDB (NoSQL, real-time sync)
- Amazon RDS PostgreSQL 14+ (structured data)
- Amazon S3 (media storage)
- Amazon ElastiCache Redis (caching)

**Infrastructure:**
- Amazon EC2 (compute)
- Amazon CloudFront (CDN)
- Elastic Load Balancer (ALB)
- Amazon Route 53 (DNS)
- AWS Elastic Beanstalk (deployment)

### 4.2 Integration Requirements

#### TR-I1: WhatsApp Business API
- Official WhatsApp Business API integration
- Message templates pre-approved by WhatsApp
- Webhook for incoming messages
- Delivery status tracking
- Rate limiting: 1000 messages/second

#### TR-I2: AWS Services Integration
- AWS SDK for Java (backend services)
- AWS Amplify SDK (frontend)
- AWS Mobile SDK (React Native)
- CloudWatch for monitoring and logging
- X-Ray for distributed tracing

### 4.3 Data Storage Requirements

#### TR-D1: DynamoDB Tables
- Users table (partition key: userId)
- Exercises table (partition key: exerciseId)
- Assignments table (partition key: assignmentId, GSI: therapistId, studentId)
- Completions table (partition key: completionId, GSI: studentId, date)
- Messages table (partition key: conversationId, sort key: timestamp)

#### TR-D2: PostgreSQL Schema
- Clinics, Therapists, Parents, Children (relational)
- Subscriptions and billing data
- Audit logs
- Analytics aggregations

#### TR-D3: S3 Bucket Structure
```
speech-play-media/
├── exercises/
│   ├── audio/
│   ├── images/
│   └── videos/
├── recordings/
│   └── {userId}/{date}/{recordingId}.opus
└── reports/
    └── {clinicId}/{date}/report.pdf
```

### 4.4 API Specifications

#### TR-API1: REST Endpoints

**Authentication:**
- `POST /api/v1/auth/login` - User login
- `POST /api/v1/auth/refresh` - Refresh token
- `POST /api/v1/auth/logout` - User logout

**Exercises:**
- `GET /api/v1/exercises` - List exercises (with filters)
- `GET /api/v1/exercises/{id}` - Get exercise details
- `POST /api/v1/exercises` - Create custom exercise (therapist only)

**Assignments:**
- `POST /api/v1/assignments` - Create assignment
- `GET /api/v1/assignments/therapist/{therapistId}` - Get therapist's assignments
- `GET /api/v1/assignments/student/{studentId}` - Get student's assignments
- `PUT /api/v1/assignments/{id}` - Update assignment

**Completions:**
- `POST /api/v1/completions` - Submit completed exercise
- `GET /api/v1/completions/student/{studentId}` - Get student's completions
- `POST /api/v1/completions/validate` - Validate speech recording

**Progress:**
- `GET /api/v1/progress/student/{studentId}` - Get student progress
- `GET /api/v1/progress/therapist/{therapistId}` - Get all students' progress

#### TR-API2: WebSocket Channels
- `/ws/dashboard/{therapistId}` - Real-time dashboard updates
- `/ws/sync/{userId}` - Real-time data sync

---

## 5. India-Specific Requirements

### 5.1 Low-Bandwidth Optimization

#### ISR-B1: Network Adaptation
- Detect network speed and adjust quality
- Preload exercises on WiFi
- Compress all assets (images: WebP, audio: Opus)
- Implement progressive image loading
- Cache aggressively using service workers

### 5.2 Regional Language Support

#### ISR-L1: Language Requirements
- Hindi (Devanagari script)
- Tamil (Tamil script)
- Telugu (Telugu script)
- Marathi (Devanagari script)
- Bengali (Bengali script)
- English (Latin script)

#### ISR-L2: Voice Recognition
- Amazon Transcribe support for all 6 languages
- Custom vocabulary for speech therapy terms
- Accent adaptation for Indian English

### 5.3 WhatsApp-First Communication

#### ISR-W1: WhatsApp Priority
- WhatsApp as primary notification channel (90% of Indian users)
- SMS fallback for non-WhatsApp users
- Rich media support (images, PDFs in WhatsApp)
- Interactive buttons for quick actions

### 5.4 Pricing in INR

#### ISR-PR1: Pricing Structure
- Therapist subscription: ₹1,500/month
- Parent subscription: ₹100/month (bundled with clinic)
- Free tier: 1 therapist, 10 students, 30-day trial
- Annual discount: 20% off
- Payment methods: UPI, credit/debit cards, net banking

### 5.5 Data Compliance

#### ISR-C1: Indian Data Laws
- Data stored in AWS Mumbai region (ap-south-1)
- Compliance with IT Act 2000
- Compliance with Personal Data Protection Bill (when enacted)
- Data localization for sensitive health information
- Consent management for data collection

---

## 6. Success Metrics

### 6.1 Key Performance Indicators (KPIs)

#### KPI-1: Compliance Rate
- **Baseline**: 30% (paper-based)
- **Target**: 80%+ within 3 months
- **Measurement**: (Completed exercises / Assigned exercises) × 100

#### KPI-2: Therapy Duration Reduction
- **Baseline**: 12 months average therapy duration
- **Target**: 8 months average therapy duration
- **Measurement**: Time from first session to discharge

#### KPI-3: User Adoption
- **Month 1**: 10 clinics, 100 therapists, 1,000 children
- **Month 6**: 50 clinics, 500 therapists, 5,000 children
- **Month 12**: 100 clinics, 1,000 therapists, 10,000 children

#### KPI-4: Engagement Metrics
- Daily active users (DAU): 60%+ of enrolled children
- Average session duration: 15+ minutes
- Weekly retention rate: 85%+
- Parent dashboard views: 3+ times per week

### 6.2 Performance Benchmarks

#### PB-1: Technical Performance
- API response time (p95): < 200ms
- Voice recognition accuracy: 85%+
- App crash rate: < 0.5%
- Sync success rate: 99%+

#### PB-2: Business Performance
- Customer acquisition cost (CAC): < ₹5,000 per clinic
- Monthly recurring revenue (MRR) growth: 20%+ month-over-month
- Churn rate: < 5% monthly
- Net Promoter Score (NPS): 50+

### 6.3 User Satisfaction Targets

#### US-1: Satisfaction Scores
- Therapist satisfaction: 4.5/5 stars
- Parent satisfaction: 4.3/5 stars
- Child engagement score: 4.0/5 stars (parent-reported)

---

## 7. Assumptions and Constraints

### 7.1 Assumptions

- Target users have smartphones (Android 8+ or iOS 12+)
- Parents have WhatsApp installed (95% penetration in India)
- Clinics have internet connectivity for therapist dashboard
- Children can use touchscreen devices independently or with minimal parent help
- Therapists are willing to adopt digital tools

### 7.2 Constraints

- Budget: Hackathon MVP within ₹50,000 AWS credits
- Timeline: 3 months to MVP, 6 months to market launch
- Team size: 5 developers (2 backend, 2 frontend, 1 ML engineer)
- Regulatory: No medical device classification (wellness app)
- Competition: Existing paper-based systems, international apps (not India-optimized)

---

## 8. Out of Scope (Phase 1)

- Video calling for remote therapy sessions
- Therapist-to-therapist collaboration features
- Insurance claim integration
- Multi-clinic management for chains
- Advanced analytics (predictive models beyond basic flagging)
- Integration with electronic health records (EHR)
- Occupational therapy and special education modules (Phase 2)

---

**Document Version**: 5.2  
**Last Updated**: February 15, 2026  
**Authors**: Uvan Adhithya B
**Status**: Final Draft
