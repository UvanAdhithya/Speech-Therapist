# Speech Play - Design Document
## The Therapy Compliance Engine - Technical Architecture

---

## 1. System Architecture

### 1.1 High-Level Architecture Overview

Speech Play follows a cloud-native, microservices-based architecture deployed on AWS. The system is designed for high availability, scalability, and offline-first operation to serve users across India's diverse network conditions.

```
┌─────────────────────────────────────────────────────────────────────┐
│                          USER LAYER                                  │
├─────────────────┬─────────────────┬─────────────────┬───────────────┤
│  React Native   │   React.js      │      PWA        │   WhatsApp    │
│  Mobile App     │   Therapist     │   Parent        │   Business    │
│  (iOS/Android)  │   Dashboard     │   Interface     │   API         │
└────────┬────────┴────────┬────────┴────────┬────────┴───────┬───────┘
         │                 │                 │                │
         └─────────────────┴─────────────────┴────────────────┘
                                    │
         ┌──────────────────────────┴──────────────────────────┐
         │              CDN LAYER (CloudFront)                  │
         └──────────────────────────┬──────────────────────────┘
                                    │
         ┌──────────────────────────┴──────────────────────────┐
         │         API GATEWAY LAYER (AWS API Gateway)          │
         │  - REST APIs          - WebSocket APIs               │
         │  - Rate Limiting      - Request Validation           │
         └──────────────────────────┬──────────────────────────┘
                                    │
         ┌──────────────────────────┴──────────────────────────┐
         │      AUTHENTICATION LAYER (AWS Cognito)              │
         │  - User Pools         - JWT Tokens                   │
         │  - MFA                - OAuth 2.0                    │
         └──────────────────────────┬──────────────────────────┘
                                    │
         ┌──────────────────────────┴──────────────────────────┐
         │         LOAD BALANCER (Elastic Load Balancer)        │
         └──────────────────────────┬──────────────────────────┘
                                    │
    ┌────────────────────────────────────────────────────────────┐
    │              BUSINESS LOGIC LAYER                          │
    ├──────────────┬──────────────┬──────────────┬──────────────┤
    │ User Service │Exercise Svc  │Progress Svc  │Notification  │
    │ (ECS)        │(ECS)         │(Lambda)      │Service (SNS) │
    └──────┬───────┴──────┬───────┴──────┬───────┴──────┬───────┘
           │              │              │              │
    ┌──────┴──────────────┴──────────────┴──────────────┴───────┐
    │                  AI/ML LAYER                               │
    ├──────────────┬──────────────┬──────────────────────────────┤
    │  Transcribe  │  SageMaker   │      Personalize             │
    │  (Speech)    │  (Grading)   │      (Reminders)             │
    └──────┬───────┴──────┬───────┴──────────────┬───────────────┘
           │              │                      │
    ┌──────┴──────────────┴──────────────────────┴───────────────┐
    │                    DATA LAYER                               │
    ├──────────────┬──────────────┬──────────────┬───────────────┤
    │  DynamoDB    │  RDS         │     S3       │ ElastiCache   │
    │  (NoSQL)     │  (PostgreSQL)│  (Media)     │ (Redis)       │
    └──────────────┴──────────────┴──────────────┴───────────────┘
```

### 1.2 Layer-by-Layer Breakdown

#### 1.2.1 User Layer

**React Native Mobile App (iOS/Android):**
- Offline-first architecture using IndexedDB/SQLite
- Voice recording using native APIs
- Local ML model for offline speech recognition
- Background sync using React Native Background Fetch
- Push notifications via Firebase Cloud Messaging

**React.js Therapist Dashboard:**
- Server-side rendering for fast initial load
- Real-time updates via WebSocket
- Data visualization using Recharts
- Responsive design (desktop/tablet)
- Export functionality (PDF/Excel)

**Progressive Web App (Parent Interface):**
- Service worker for offline capability
- Installable on home screen
- Push notifications via Web Push API
- Responsive mobile-first design
- Minimal JavaScript bundle (< 200 KB)

**WhatsApp Business API:**
- Cloud API integration (official)
- Webhook for incoming messages
- Template messages for notifications
- Rich media support (images, PDFs)
- Delivery status tracking

#### 1.2.2 CDN Layer (Amazon CloudFront)

- Global edge locations for low latency
- Cache static assets (images, audio, videos)
- Cache API responses (GET requests, 5-minute TTL)
- Gzip/Brotli compression
- Custom domain with SSL/TLS certificates

#### 1.2.3 API Gateway Layer

**AWS API Gateway:**
- REST API endpoints for CRUD operations
- WebSocket API for real-time updates
- Request/response transformation
- CORS configuration
- API versioning (/v1, /v2)

**Features:**
- Rate limiting: 1000 requests/minute per user
- Request validation using JSON Schema
- API key management for mobile apps
- Usage plans and throttling
- CloudWatch logging and metrics

#### 1.2.4 Authentication Layer (AWS Cognito)

**User Pools:**
- Separate pools for therapists, parents, children
- Email/phone verification
- Password policies and MFA
- Custom attributes (clinicId, role, language)
- Lambda triggers for custom workflows

**Identity Pools:**
- Federated identities for social login (future)
- Temporary AWS credentials for S3 access
- Fine-grained IAM permissions

#### 1.2.5 Load Balancer Layer

**Application Load Balancer (ALB):**
- Distributes traffic across ECS tasks
- Health checks for target groups
- SSL/TLS termination
- Path-based routing (/api/users → User Service)
- Sticky sessions for WebSocket connections


#### 1.2.6 Business Logic Layer

**Microservices Architecture:**

1. **User Service (Spring Boot on ECS)**
   - User registration and profile management
   - Clinic and therapist management
   - Role-based access control
   - Audit logging

2. **Exercise Service (Spring Boot on ECS)**
   - Exercise library CRUD operations
   - Exercise categorization and search
   - Custom exercise creation
   - Exercise versioning

3. **Assignment Service (Spring Boot on ECS)**
   - Assignment creation and management
   - Assignment scheduling
   - Bulk assignment operations
   - Assignment status tracking

4. **Progress Service (AWS Lambda)**
   - Completion tracking
   - Progress calculation
   - Analytics aggregation
   - Report generation

5. **Notification Service (AWS Lambda + SNS/SQS)**
   - WhatsApp message sending
   - Push notification delivery
   - Email notifications
   - SMS fallback

6. **AI Service (AWS Lambda)**
   - Speech-to-text processing
   - Auto-grading logic
   - Recommendation engine
   - Predictive analytics

**Inter-Service Communication:**
- Synchronous: REST APIs via API Gateway
- Asynchronous: SNS/SQS for event-driven workflows
- Service mesh: AWS App Mesh for observability

#### 1.2.7 AI/ML Layer

**Amazon Transcribe:**
- Real-time speech-to-text conversion
- Support for Hindi, Tamil, Telugu, Marathi, Bengali, English
- Custom vocabulary for therapy terms
- Confidence scores for accuracy

**Amazon SageMaker:**
- ML model for phoneme matching
- Training pipeline using labeled data
- Model versioning and A/B testing
- Batch inference for bulk grading

**Amazon Personalize:**
- Recommendation engine for optimal reminder times
- User behavior analysis
- Personalized exercise suggestions
- Churn prediction

#### 1.2.8 Data Layer

**Amazon DynamoDB:**
- Single-digit millisecond latency
- Auto-scaling based on load
- Global tables for multi-region (future)
- Point-in-time recovery
- DynamoDB Streams for change data capture

**Amazon RDS PostgreSQL:**
- Multi-AZ deployment for high availability
- Read replicas for analytics queries
- Automated backups (7-day retention)
- Encryption at rest (AES-256)

**Amazon S3:**
- Versioning enabled
- Lifecycle policies (move to Glacier after 90 days)
- Server-side encryption (SSE-S3)
- CloudFront integration for fast delivery

**Amazon ElastiCache Redis:**
- In-memory caching for hot data
- Session storage for WebSocket connections
- Leaderboard data (sorted sets)
- Cache-aside pattern

---

## 2. Component Design

### 2.1 Therapist Dashboard (React.js)

**Architecture:**
```
src/
├── components/
│   ├── Dashboard/
│   │   ├── ComplianceWidget.tsx
│   │   ├── StudentList.tsx
│   │   └── ProgressChart.tsx
│   ├── Exercises/
│   │   ├── ExerciseLibrary.tsx
│   │   ├── ExercisePreview.tsx
│   │   └── CustomExerciseForm.tsx
│   └── Assignments/
│       ├── AssignmentForm.tsx
│       └── BulkAssignment.tsx
├── services/
│   ├── api.service.ts
│   ├── websocket.service.ts
│   └── auth.service.ts
├── store/
│   ├── slices/
│   │   ├── userSlice.ts
│   │   ├── exerciseSlice.ts
│   │   └── progressSlice.ts
│   └── store.ts
└── utils/
    ├── dateHelpers.ts
    └── chartHelpers.ts
```

**Key Features:**
- Redux Toolkit for state management
- React Query for server state caching
- WebSocket connection for real-time updates
- Recharts for data visualization
- React Hook Form for form handling
- Material-UI for component library


### 2.2 Kid's Mobile App (React Native)

**Architecture:**
```
src/
├── screens/
│   ├── HomeScreen.tsx
│   ├── ExerciseScreen.tsx
│   ├── RewardsScreen.tsx
│   └── ProfileScreen.tsx
├── components/
│   ├── GameEngine/
│   │   ├── VoiceRecorder.tsx
│   │   ├── AnimationPlayer.tsx
│   │   └── FeedbackOverlay.tsx
│   └── Rewards/
│       ├── StarDisplay.tsx
│       ├── BadgeCollection.tsx
│       └── LevelProgress.tsx
├── services/
│   ├── offline.service.ts
│   ├── sync.service.ts
│   ├── voice.service.ts
│   └── ml.service.ts
├── store/
│   ├── exerciseSlice.ts
│   ├── rewardSlice.ts
│   └── syncSlice.ts
└── utils/
    ├── audioHelpers.ts
    └── storageHelpers.ts
```

**Key Technologies:**
- React Native 0.72+
- Redux Persist for offline storage
- React Native Voice for audio recording
- Lottie for animations
- React Native MMKV for fast storage
- TensorFlow Lite for on-device ML

**Offline-First Strategy:**
1. Download exercises on WiFi
2. Store in local SQLite database
3. Practice with local ML model
4. Queue completions in IndexedDB
5. Sync when online using background task

### 2.3 Parent Interface (PWA)

**Architecture:**
```
src/
├── pages/
│   ├── Dashboard.tsx
│   ├── Progress.tsx
│   └── Messages.tsx
├── components/
│   ├── ProgressCard.tsx
│   ├── CalendarView.tsx
│   └── ChatInterface.tsx
├── service-worker.ts
└── manifest.json
```

**PWA Features:**
- Service worker for offline caching
- Web App Manifest for installability
- Push notifications via Web Push API
- Background sync for message sending
- IndexedDB for offline data storage

**Performance Optimizations:**
- Code splitting by route
- Lazy loading images
- Prefetching critical resources
- Minimal third-party dependencies

### 2.4 Backend Services (Spring Boot Microservices)

**User Service:**
```java
@RestController
@RequestMapping("/api/v1/users")
public class UserController {
    
    @PostMapping("/register")
    public ResponseEntity<UserDTO> registerUser(@RequestBody UserRegistrationRequest request) {
        // Validate request
        // Create user in Cognito
        // Store user profile in DynamoDB
        // Send welcome notification
        return ResponseEntity.ok(userDTO);
    }
    
    @GetMapping("/{userId}")
    public ResponseEntity<UserDTO> getUser(@PathVariable String userId) {
        // Fetch from cache (Redis)
        // If not in cache, fetch from DynamoDB
        // Update cache
        return ResponseEntity.ok(userDTO);
    }
}
```

**Exercise Service:**
```java
@RestController
@RequestMapping("/api/v1/exercises")
public class ExerciseController {
    
    @GetMapping
    public ResponseEntity<Page<ExerciseDTO>> listExercises(
        @RequestParam(required = false) String phoneme,
        @RequestParam(required = false) String difficulty,
        @RequestParam(required = false) String language,
        Pageable pageable
    ) {
        // Query DynamoDB with filters
        // Apply pagination
        // Return results
        return ResponseEntity.ok(exercises);
    }
    
    @PostMapping
    public ResponseEntity<ExerciseDTO> createExercise(@RequestBody ExerciseRequest request) {
        // Validate request
        // Upload media to S3
        // Store metadata in DynamoDB
        // Invalidate cache
        return ResponseEntity.ok(exerciseDTO);
    }
}
```

**Progress Service (Lambda):**
```python
import boto3
from decimal import Decimal

dynamodb = boto3.resource('dynamodb')
completions_table = dynamodb.Table('Completions')

def lambda_handler(event, context):
    student_id = event['pathParameters']['studentId']
    
    # Query completions for student
    response = completions_table.query(
        IndexName='StudentIdIndex',
        KeyConditionExpression='studentId = :sid',
        ExpressionAttributeValues={':sid': student_id}
    )
    
    # Calculate metrics
    total_assigned = event['queryStringParameters'].get('totalAssigned', 0)
    completed = len(response['Items'])
    compliance_rate = (completed / total_assigned * 100) if total_assigned > 0 else 0
    
    # Calculate average accuracy
    accuracies = [item['accuracy'] for item in response['Items']]
    avg_accuracy = sum(accuracies) / len(accuracies) if accuracies else 0
    
    return {
        'statusCode': 200,
        'body': json.dumps({
            'studentId': student_id,
            'complianceRate': Decimal(str(compliance_rate)),
            'averageAccuracy': Decimal(str(avg_accuracy)),
            'totalCompleted': completed
        })
    }
```

### 2.5 AI/ML Pipeline

**Speech Recognition Flow:**
```
1. Mobile app records audio (Opus codec, 32 kbps)
2. Upload to S3 bucket (recordings/)
3. Trigger Lambda function via S3 event
4. Lambda invokes Amazon Transcribe
5. Transcribe returns transcription + confidence score
6. Lambda invokes SageMaker endpoint for phoneme matching
7. SageMaker returns accuracy score (0-100%)
8. Lambda stores result in DynamoDB
9. Lambda sends notification to mobile app via SNS
10. Mobile app displays feedback to child
```

**Auto-Grading Algorithm (SageMaker):**
```python
import sagemaker
from sagemaker.sklearn import SKLearnModel

class PhonemeGrader:
    def __init__(self):
        self.model = self.load_model()
    
    def grade(self, expected_phoneme, transcribed_text, confidence_score):
        # Extract phonemes from transcribed text
        transcribed_phonemes = self.extract_phonemes(transcribed_text)
        
        # Calculate similarity score
        similarity = self.calculate_similarity(expected_phoneme, transcribed_phonemes)
        
        # Adjust for confidence score
        adjusted_score = similarity * confidence_score
        
        # Apply threshold
        if adjusted_score >= 0.85:
            return {'grade': 'correct', 'score': adjusted_score * 100}
        elif adjusted_score >= 0.60:
            return {'grade': 'partial', 'score': adjusted_score * 100}
        else:
            return {'grade': 'incorrect', 'score': adjusted_score * 100}
    
    def extract_phonemes(self, text):
        # Use phoneme extraction library
        # Return list of phonemes
        pass
    
    def calculate_similarity(self, expected, actual):
        # Use Levenshtein distance or phonetic similarity
        # Return similarity score (0-1)
        pass
```

**Smart Reminder System (Amazon Personalize):**
```python
import boto3

personalize_runtime = boto3.client('personalize-runtime')

def get_optimal_reminder_time(user_id):
    # Get user's historical practice times
    response = personalize_runtime.get_recommendations(
        campaignArn='arn:aws:personalize:region:account:campaign/reminder-optimizer',
        userId=user_id,
        numResults=1
    )
    
    # Return recommended time (e.g., "18:00")
    return response['itemList'][0]['itemId']
```

---

## 3. Database Design

### 3.1 DynamoDB Schema

**Users Table:**
```
Partition Key: userId (String)
Attributes:
- email (String)
- phoneNumber (String)
- role (String) // "therapist", "parent", "child"
- clinicId (String)
- language (String)
- createdAt (Number)
- updatedAt (Number)

GSI: EmailIndex (email)
GSI: ClinicIdIndex (clinicId)
```

**Exercises Table:**
```
Partition Key: exerciseId (String)
Attributes:
- title (String)
- description (String)
- phoneme (String)
- difficulty (String) // "beginner", "intermediate", "advanced"
- language (String)
- audioUrl (String)
- imageUrl (String)
- instructions (String)
- createdBy (String) // therapistId
- createdAt (Number)

GSI: PhonemeIndex (phoneme, difficulty)
GSI: LanguageIndex (language)
```

**Assignments Table:**
```
Partition Key: assignmentId (String)
Attributes:
- therapistId (String)
- studentId (String)
- exerciseId (String)
- frequency (String) // "daily", "3x_week", "custom"
- startDate (Number)
- endDate (Number)
- targetAccuracy (Number)
- status (String) // "active", "completed", "cancelled"
- createdAt (Number)

GSI: TherapistIdIndex (therapistId, createdAt)
GSI: StudentIdIndex (studentId, status)
```

**Completions Table:**
```
Partition Key: completionId (String)
Sort Key: completedAt (Number)
Attributes:
- studentId (String)
- assignmentId (String)
- exerciseId (String)
- recordingUrl (String)
- transcription (String)
- accuracy (Number)
- attempts (Number)
- duration (Number) // seconds
- feedback (String)

GSI: StudentIdIndex (studentId, completedAt)
GSI: AssignmentIdIndex (assignmentId)
```

**Messages Table:**
```
Partition Key: conversationId (String) // therapistId_parentId
Sort Key: timestamp (Number)
Attributes:
- senderId (String)
- receiverId (String)
- messageType (String) // "text", "image", "report"
- content (String)
- mediaUrl (String)
- deliveryStatus (String) // "sent", "delivered", "read"
- whatsappMessageId (String)

GSI: SenderIdIndex (senderId, timestamp)
GSI: ReceiverIdIndex (receiverId, timestamp)
```


### 3.2 PostgreSQL Schema

**Clinics Table:**
```sql
CREATE TABLE clinics (
    clinic_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(255) NOT NULL,
    address TEXT,
    city VARCHAR(100),
    state VARCHAR(100),
    pincode VARCHAR(10),
    phone VARCHAR(20),
    email VARCHAR(255),
    subscription_tier VARCHAR(50), -- "free", "basic", "premium"
    subscription_start_date DATE,
    subscription_end_date DATE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_clinics_city ON clinics(city);
CREATE INDEX idx_clinics_subscription ON clinics(subscription_tier, subscription_end_date);
```

**Therapists Table:**
```sql
CREATE TABLE therapists (
    therapist_id UUID PRIMARY KEY,
    clinic_id UUID REFERENCES clinics(clinic_id),
    user_id VARCHAR(255) UNIQUE NOT NULL, -- Cognito userId
    first_name VARCHAR(100),
    last_name VARCHAR(100),
    email VARCHAR(255) UNIQUE,
    phone VARCHAR(20),
    license_number VARCHAR(50),
    specialization VARCHAR(100),
    years_of_experience INT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_therapists_clinic ON therapists(clinic_id);
CREATE INDEX idx_therapists_user ON therapists(user_id);
```

**Parents Table:**
```sql
CREATE TABLE parents (
    parent_id UUID PRIMARY KEY,
    user_id VARCHAR(255) UNIQUE NOT NULL,
    first_name VARCHAR(100),
    last_name VARCHAR(100),
    email VARCHAR(255),
    phone VARCHAR(20) UNIQUE,
    whatsapp_number VARCHAR(20),
    preferred_language VARCHAR(20),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_parents_phone ON parents(phone);
CREATE INDEX idx_parents_user ON parents(user_id);
```

**Children Table:**
```sql
CREATE TABLE children (
    child_id UUID PRIMARY KEY,
    user_id VARCHAR(255) UNIQUE NOT NULL,
    parent_id UUID REFERENCES parents(parent_id),
    therapist_id UUID REFERENCES therapists(therapist_id),
    first_name VARCHAR(100),
    date_of_birth DATE,
    gender VARCHAR(20),
    diagnosis TEXT,
    therapy_start_date DATE,
    therapy_goal TEXT,
    preferred_language VARCHAR(20),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_children_parent ON children(parent_id);
CREATE INDEX idx_children_therapist ON children(therapist_id);
```

**Subscriptions Table:**
```sql
CREATE TABLE subscriptions (
    subscription_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    clinic_id UUID REFERENCES clinics(clinic_id),
    plan_type VARCHAR(50), -- "monthly", "annual"
    amount DECIMAL(10, 2),
    currency VARCHAR(3) DEFAULT 'INR',
    status VARCHAR(50), -- "active", "cancelled", "expired"
    start_date DATE,
    end_date DATE,
    payment_method VARCHAR(50),
    razorpay_subscription_id VARCHAR(255),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_subscriptions_clinic ON subscriptions(clinic_id);
CREATE INDEX idx_subscriptions_status ON subscriptions(status, end_date);
```

**Audit Logs Table:**
```sql
CREATE TABLE audit_logs (
    log_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id VARCHAR(255),
    action VARCHAR(100), -- "login", "create_assignment", "view_progress"
    resource_type VARCHAR(50), -- "assignment", "exercise", "user"
    resource_id VARCHAR(255),
    ip_address INET,
    user_agent TEXT,
    timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_audit_logs_user ON audit_logs(user_id, timestamp);
CREATE INDEX idx_audit_logs_action ON audit_logs(action, timestamp);
```

### 3.3 S3 Bucket Structure

```
speech-play-production/
├── exercises/
│   ├── audio/
│   │   ├── {exerciseId}/
│   │   │   ├── hindi.opus
│   │   │   ├── tamil.opus
│   │   │   └── english.opus
│   ├── images/
│   │   ├── {exerciseId}/
│   │   │   ├── thumbnail.webp
│   │   │   └── full.webp
│   └── videos/
│       └── {exerciseId}/
│           └── demo.mp4
├── recordings/
│   ├── {userId}/
│   │   ├── {date}/
│   │   │   ├── {recordingId}.opus
│   │   │   └── {recordingId}_metadata.json
├── reports/
│   ├── {clinicId}/
│   │   ├── weekly/
│   │   │   └── {date}_report.pdf
│   │   └── monthly/
│   │       └── {date}_report.pdf
└── avatars/
    └── {userId}/
        └── profile.webp
```

**S3 Lifecycle Policies:**
- Recordings: Move to S3 Glacier after 90 days
- Reports: Move to S3 Glacier after 180 days
- Exercises: Never expire (permanent storage)

### 3.4 Cache Strategy (Redis)

**Cache Keys:**
```
user:{userId} → User profile (TTL: 1 hour)
exercise:{exerciseId} → Exercise details (TTL: 24 hours)
assignments:therapist:{therapistId} → Therapist's assignments (TTL: 5 minutes)
assignments:student:{studentId} → Student's assignments (TTL: 5 minutes)
progress:student:{studentId} → Student progress (TTL: 5 minutes)
leaderboard:clinic:{clinicId} → Clinic leaderboard (TTL: 1 hour)
session:{sessionId} → WebSocket session data (TTL: 30 minutes)
```

**Cache Invalidation:**
- On user update: Invalidate `user:{userId}`
- On assignment creation: Invalidate `assignments:therapist:{therapistId}` and `assignments:student:{studentId}`
- On completion submission: Invalidate `progress:student:{studentId}` and `leaderboard:clinic:{clinicId}`

---

## 4. API Design

### 4.1 REST Endpoints

**Authentication Endpoints:**

```http
POST /api/v1/auth/register
Content-Type: application/json

{
  "email": "therapist@example.com",
  "password": "SecurePass123!",
  "role": "therapist",
  "firstName": "Priya",
  "lastName": "Sharma",
  "clinicId": "clinic-uuid",
  "phone": "+919876543210"
}

Response 201:
{
  "userId": "user-uuid",
  "email": "therapist@example.com",
  "role": "therapist",
  "accessToken": "eyJhbGciOiJIUzI1NiIs...",
  "refreshToken": "eyJhbGciOiJIUzI1NiIs...",
  "expiresIn": 3600
}
```

```http
POST /api/v1/auth/login
Content-Type: application/json

{
  "email": "therapist@example.com",
  "password": "SecurePass123!"
}

Response 200:
{
  "userId": "user-uuid",
  "accessToken": "eyJhbGciOiJIUzI1NiIs...",
  "refreshToken": "eyJhbGciOiJIUzI1NiIs...",
  "expiresIn": 3600
}
```

**Exercise Endpoints:**

```http
GET /api/v1/exercises?phoneme=ka&difficulty=beginner&language=hindi&page=0&size=20
Authorization: Bearer {accessToken}

Response 200:
{
  "content": [
    {
      "exerciseId": "ex-uuid",
      "title": "Say KA",
      "description": "Practice the KA sound",
      "phoneme": "ka",
      "difficulty": "beginner",
      "language": "hindi",
      "audioUrl": "https://cdn.speechplay.com/exercises/audio/ex-uuid/hindi.opus",
      "imageUrl": "https://cdn.speechplay.com/exercises/images/ex-uuid/full.webp",
      "instructions": "Listen and repeat the sound KA"
    }
  ],
  "totalElements": 150,
  "totalPages": 8,
  "currentPage": 0
}
```

```http
POST /api/v1/exercises
Authorization: Bearer {accessToken}
Content-Type: application/json

{
  "title": "Say MA",
  "description": "Practice the MA sound",
  "phoneme": "ma",
  "difficulty": "beginner",
  "language": "hindi",
  "instructions": "Listen and repeat the sound MA",
  "audioFile": "base64_encoded_audio",
  "imageFile": "base64_encoded_image"
}

Response 201:
{
  "exerciseId": "ex-uuid",
  "title": "Say MA",
  "audioUrl": "https://cdn.speechplay.com/exercises/audio/ex-uuid/hindi.opus",
  "imageUrl": "https://cdn.speechplay.com/exercises/images/ex-uuid/full.webp"
}
```

**Assignment Endpoints:**

```http
POST /api/v1/assignments
Authorization: Bearer {accessToken}
Content-Type: application/json

{
  "therapistId": "therapist-uuid",
  "studentIds": ["student-uuid-1", "student-uuid-2"],
  "exerciseIds": ["ex-uuid-1", "ex-uuid-2"],
  "frequency": "daily",
  "startDate": "2026-02-15",
  "endDate": "2026-02-22",
  "targetAccuracy": 80,
  "instructions": "Practice these sounds 3 times each day"
}

Response 201:
{
  "assignmentIds": ["assign-uuid-1", "assign-uuid-2"],
  "studentsNotified": 2,
  "whatsappMessagesSent": 2
}
```

```http
GET /api/v1/assignments/student/{studentId}?status=active
Authorization: Bearer {accessToken}

Response 200:
{
  "assignments": [
    {
      "assignmentId": "assign-uuid",
      "exerciseId": "ex-uuid",
      "exerciseTitle": "Say KA",
      "frequency": "daily",
      "startDate": "2026-02-15",
      "endDate": "2026-02-22",
      "targetAccuracy": 80,
      "completionRate": 60,
      "averageAccuracy": 75,
      "status": "active"
    }
  ]
}
```

**Completion Endpoints:**

```http
POST /api/v1/completions
Authorization: Bearer {accessToken}
Content-Type: application/json

{
  "studentId": "student-uuid",
  "assignmentId": "assign-uuid",
  "exerciseId": "ex-uuid",
  "recordingFile": "base64_encoded_audio",
  "duration": 45,
  "attempts": 2
}

Response 201:
{
  "completionId": "comp-uuid",
  "transcription": "ka",
  "accuracy": 85,
  "feedback": "Great job! You pronounced it correctly.",
  "starsEarned": 2,
  "badgesUnlocked": ["3-day-streak"]
}
```

```http
GET /api/v1/completions/student/{studentId}?startDate=2026-02-01&endDate=2026-02-15
Authorization: Bearer {accessToken}

Response 200:
{
  "completions": [
    {
      "completionId": "comp-uuid",
      "exerciseTitle": "Say KA",
      "completedAt": "2026-02-15T18:30:00Z",
      "accuracy": 85,
      "attempts": 2,
      "duration": 45
    }
  ],
  "totalCompletions": 25,
  "averageAccuracy": 82
}
```

**Progress Endpoints:**

```http
GET /api/v1/progress/student/{studentId}
Authorization: Bearer {accessToken}

Response 200:
{
  "studentId": "student-uuid",
  "studentName": "Aarav Kumar",
  "complianceRate": 85,
  "averageAccuracy": 82,
  "totalExercisesCompleted": 45,
  "currentStreak": 7,
  "longestStreak": 12,
  "level": 5,
  "totalStars": 120,
  "badges": ["3-day-streak", "perfect-score", "100-exercises"],
  "weeklyProgress": [
    {"date": "2026-02-09", "completed": 5, "accuracy": 80},
    {"date": "2026-02-10", "completed": 6, "accuracy": 85},
    {"date": "2026-02-11", "completed": 4, "accuracy": 78}
  ],
  "phonemeProgress": [
    {"phoneme": "ka", "accuracy": 90, "attempts": 20},
    {"phoneme": "ga", "accuracy": 75, "attempts": 15}
  ]
}
```


### 4.2 WebSocket Channels

**Real-Time Dashboard Updates:**

```javascript
// Client connects to WebSocket
const ws = new WebSocket('wss://api.speechplay.com/ws/dashboard/therapist-uuid');

// Server sends real-time updates
{
  "type": "COMPLETION_SUBMITTED",
  "data": {
    "studentId": "student-uuid",
    "studentName": "Aarav Kumar",
    "exerciseTitle": "Say KA",
    "accuracy": 85,
    "completedAt": "2026-02-15T18:30:00Z"
  }
}

{
  "type": "STUDENT_ONLINE",
  "data": {
    "studentId": "student-uuid",
    "studentName": "Aarav Kumar",
    "status": "practicing"
  }
}

{
  "type": "COMPLIANCE_UPDATE",
  "data": {
    "studentId": "student-uuid",
    "complianceRate": 85,
    "change": +5
  }
}
```

**Real-Time Sync for Mobile App:**

```javascript
// Client connects to WebSocket
const ws = new WebSocket('wss://api.speechplay.com/ws/sync/user-uuid');

// Server sends sync updates
{
  "type": "NEW_ASSIGNMENT",
  "data": {
    "assignmentId": "assign-uuid",
    "exerciseId": "ex-uuid",
    "exerciseTitle": "Say MA",
    "dueDate": "2026-02-16"
  }
}

{
  "type": "ASSIGNMENT_UPDATED",
  "data": {
    "assignmentId": "assign-uuid",
    "status": "completed"
  }
}

{
  "type": "MESSAGE_RECEIVED",
  "data": {
    "messageId": "msg-uuid",
    "senderId": "therapist-uuid",
    "senderName": "Dr. Priya Sharma",
    "content": "Great progress this week!",
    "timestamp": "2026-02-15T19:00:00Z"
  }
}
```

### 4.3 Authentication Flow

**JWT Token Structure:**
```json
{
  "header": {
    "alg": "RS256",
    "typ": "JWT",
    "kid": "cognito-key-id"
  },
  "payload": {
    "sub": "user-uuid",
    "email": "therapist@example.com",
    "cognito:groups": ["therapists"],
    "custom:clinicId": "clinic-uuid",
    "custom:role": "therapist",
    "iss": "https://cognito-idp.ap-south-1.amazonaws.com/ap-south-1_XXXXX",
    "exp": 1708023600,
    "iat": 1708020000
  }
}
```

**Authentication Flow:**
```
1. User submits credentials → API Gateway
2. API Gateway forwards to Cognito
3. Cognito validates credentials
4. Cognito returns JWT tokens (access + refresh)
5. Client stores tokens securely
6. Client includes access token in Authorization header
7. API Gateway validates token with Cognito
8. If valid, request forwarded to backend services
9. If expired, client uses refresh token to get new access token
```

### 4.4 Rate Limiting Strategy

**Rate Limits by User Type:**

| User Type | Requests/Minute | Burst Limit |
|-----------|----------------|-------------|
| Child     | 100            | 150         |
| Parent    | 200            | 300         |
| Therapist | 500            | 750         |
| Admin     | 1000           | 1500        |

**Rate Limiting Implementation:**
- Token bucket algorithm in API Gateway
- Redis-based rate limiting for custom logic
- 429 Too Many Requests response with Retry-After header
- Exponential backoff on client side

---

## 5. Offline-First Architecture

### 5.1 Local Storage Strategy

**Mobile App (React Native):**

```javascript
// IndexedDB schema for exercises
const exerciseStore = {
  name: 'exercises',
  keyPath: 'exerciseId',
  indexes: [
    { name: 'phoneme', keyPath: 'phoneme' },
    { name: 'language', keyPath: 'language' }
  ]
};

// IndexedDB schema for completions queue
const completionsQueue = {
  name: 'completionsQueue',
  keyPath: 'queueId',
  autoIncrement: true,
  indexes: [
    { name: 'syncStatus', keyPath: 'syncStatus' }, // 'pending', 'syncing', 'synced'
    { name: 'createdAt', keyPath: 'createdAt' }
  ]
};

// Store exercise locally
async function downloadExercise(exerciseId) {
  const exercise = await fetchExercise(exerciseId);
  
  // Download audio file
  const audioBlob = await fetch(exercise.audioUrl).then(r => r.blob());
  exercise.audioBlob = audioBlob;
  
  // Download image file
  const imageBlob = await fetch(exercise.imageUrl).then(r => r.blob());
  exercise.imageBlob = imageBlob;
  
  // Store in IndexedDB
  await db.exercises.put(exercise);
}

// Queue completion for sync
async function queueCompletion(completion) {
  await db.completionsQueue.add({
    ...completion,
    syncStatus: 'pending',
    createdAt: Date.now()
  });
}
```

**PWA (Parent Interface):**

```javascript
// Service worker caching strategy
self.addEventListener('fetch', (event) => {
  if (event.request.url.includes('/api/v1/progress')) {
    // Network first, fallback to cache
    event.respondWith(
      fetch(event.request)
        .then(response => {
          const clonedResponse = response.clone();
          caches.open('api-cache').then(cache => {
            cache.put(event.request, clonedResponse);
          });
          return response;
        })
        .catch(() => caches.match(event.request))
    );
  } else if (event.request.url.includes('/static/')) {
    // Cache first, fallback to network
    event.respondWith(
      caches.match(event.request)
        .then(response => response || fetch(event.request))
    );
  }
});
```

### 5.2 Sync Mechanism

**Background Sync Strategy:**

```javascript
// Register background sync
if ('serviceWorker' in navigator && 'sync' in registration) {
  registration.sync.register('sync-completions');
}

// Handle background sync event
self.addEventListener('sync', (event) => {
  if (event.tag === 'sync-completions') {
    event.waitUntil(syncCompletions());
  }
});

async function syncCompletions() {
  const pendingCompletions = await db.completionsQueue
    .where('syncStatus').equals('pending')
    .toArray();
  
  for (const completion of pendingCompletions) {
    try {
      // Update status to syncing
      await db.completionsQueue.update(completion.queueId, {
        syncStatus: 'syncing'
      });
      
      // Upload recording to S3
      const recordingUrl = await uploadRecording(completion.recordingBlob);
      
      // Submit completion to API
      const response = await fetch('/api/v1/completions', {
        method: 'POST',
        headers: {
          'Authorization': `Bearer ${accessToken}`,
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({
          ...completion,
          recordingUrl
        })
      });
      
      if (response.ok) {
        // Mark as synced
        await db.completionsQueue.update(completion.queueId, {
          syncStatus: 'synced',
          syncedAt: Date.now()
        });
      } else {
        // Revert to pending
        await db.completionsQueue.update(completion.queueId, {
          syncStatus: 'pending'
        });
      }
    } catch (error) {
      console.error('Sync failed:', error);
      // Revert to pending for retry
      await db.completionsQueue.update(completion.queueId, {
        syncStatus: 'pending'
      });
    }
  }
}
```

### 5.3 Conflict Resolution

**Strategy: Last Write Wins (LWW) with Timestamps**

```javascript
async function resolveConflict(localData, serverData) {
  // For assignments: server always wins (therapist is source of truth)
  if (localData.type === 'assignment') {
    return serverData;
  }
  
  // For completions: client always wins (child's practice data)
  if (localData.type === 'completion') {
    return localData;
  }
  
  // For user preferences: compare timestamps
  if (localData.type === 'preference') {
    return localData.updatedAt > serverData.updatedAt ? localData : serverData;
  }
}
```

### 5.4 Data Consistency Guarantees

**Eventual Consistency Model:**
- Completions are eventually consistent (acceptable delay: up to 24 hours)
- Assignments are strongly consistent (immediate sync required)
- Progress metrics are eventually consistent (acceptable delay: up to 1 hour)

**Consistency Checks:**
```javascript
// Periodic consistency check
setInterval(async () => {
  const localCompletionCount = await db.completions.count();
  const serverCompletionCount = await fetchCompletionCount();
  
  if (localCompletionCount !== serverCompletionCount) {
    // Trigger full sync
    await performFullSync();
  }
}, 3600000); // Every hour
```

---

## 6. AI/ML Pipeline Design

### 6.1 Speech Recognition Flow

**Architecture:**
```
Mobile App → S3 Upload → Lambda Trigger → Transcribe → SageMaker → DynamoDB → SNS → Mobile App
```

**Detailed Flow:**

1. **Audio Recording (Mobile App)**
   ```javascript
   import Voice from '@react-native-voice/voice';
   
   async function recordAudio() {
     await Voice.start('hi-IN'); // Hindi
     // User speaks
     await Voice.stop();
     const results = await Voice.onSpeechResults();
     return results[0]; // Transcribed text
   }
   ```

2. **Upload to S3**
   ```javascript
   const recordingKey = `recordings/${userId}/${date}/${recordingId}.opus`;
   await Storage.put(recordingKey, audioBlob, {
     contentType: 'audio/opus',
     metadata: {
       userId,
       exerciseId,
       expectedPhoneme: 'ka'
     }
   });
   ```

3. **Lambda Processing**
   ```python
   import boto3
   
   s3 = boto3.client('s3')
   transcribe = boto3.client('transcribe')
   sagemaker_runtime = boto3.client('sagemaker-runtime')
   
   def lambda_handler(event, context):
       # Get S3 object details
       bucket = event['Records'][0]['s3']['bucket']['name']
       key = event['Records'][0]['s3']['object']['key']
       
       # Start transcription job
       job_name = f"transcribe-{uuid.uuid4()}"
       transcribe.start_transcription_job(
           TranscriptionJobName=job_name,
           Media={'MediaFileUri': f's3://{bucket}/{key}'},
           MediaFormat='opus',
           LanguageCode='hi-IN',
           Settings={
               'ShowSpeakerLabels': False,
               'MaxSpeakerLabels': 1
           }
       )
       
       # Wait for completion (or use async callback)
       while True:
           status = transcribe.get_transcription_job(TranscriptionJobName=job_name)
           if status['TranscriptionJob']['TranscriptionJobStatus'] in ['COMPLETED', 'FAILED']:
               break
           time.sleep(2)
       
       # Get transcription result
       transcript_uri = status['TranscriptionJob']['Transcript']['TranscriptFileUri']
       transcript = requests.get(transcript_uri).json()
       transcribed_text = transcript['results']['transcripts'][0]['transcript']
       confidence = transcript['results']['items'][0]['alternatives'][0]['confidence']
       
       # Invoke SageMaker for grading
       response = sagemaker_runtime.invoke_endpoint(
           EndpointName='phoneme-grader',
           ContentType='application/json',
           Body=json.dumps({
               'expected_phoneme': 'ka',
               'transcribed_text': transcribed_text,
               'confidence': confidence
           })
       )
       
       result = json.loads(response['Body'].read())
       
       # Store in DynamoDB
       dynamodb.put_item(
           TableName='Completions',
           Item={
               'completionId': str(uuid.uuid4()),
               'userId': metadata['userId'],
               'exerciseId': metadata['exerciseId'],
               'transcription': transcribed_text,
               'accuracy': Decimal(str(result['score'])),
               'grade': result['grade'],
               'completedAt': int(time.time())
           }
       )
       
       # Send notification via SNS
       sns.publish(
           TopicArn='arn:aws:sns:ap-south-1:account:completion-notifications',
           Message=json.dumps(result),
           MessageAttributes={
               'userId': {'DataType': 'String', 'StringValue': metadata['userId']}
           }
       )
       
       return result
   ```


### 6.2 Auto-Grading Algorithm (SageMaker)

**Model Training:**

```python
import sagemaker
from sagemaker.sklearn import SKLearn

# Training script
# train.py
import pandas as pd
from sklearn.ensemble import RandomForestClassifier
import joblib

def train():
    # Load training data
    df = pd.read_csv('/opt/ml/input/data/training/phoneme_data.csv')
    
    # Features: expected_phoneme, transcribed_phoneme, confidence, duration
    X = df[['expected_phoneme_encoded', 'transcribed_phoneme_encoded', 
            'confidence', 'duration', 'levenshtein_distance']]
    y = df['accuracy_score']
    
    # Train model
    model = RandomForestClassifier(n_estimators=100, max_depth=10)
    model.fit(X, y)
    
    # Save model
    joblib.dump(model, '/opt/ml/model/model.joblib')

if __name__ == '__main__':
    train()

# Deploy model
sklearn_estimator = SKLearn(
    entry_point='train.py',
    role='SageMakerRole',
    instance_type='ml.m5.xlarge',
    framework_version='1.0-1',
    py_version='py3'
)

sklearn_estimator.fit({'training': 's3://speech-play/training-data/'})

# Deploy endpoint
predictor = sklearn_estimator.deploy(
    initial_instance_count=1,
    instance_type='ml.t2.medium',
    endpoint_name='phoneme-grader'
)
```

**Inference:**

```python
# inference.py
import joblib
import json
import numpy as np
from Levenshtein import distance

def model_fn(model_dir):
    model = joblib.load(f'{model_dir}/model.joblib')
    return model

def input_fn(request_body, content_type):
    if content_type == 'application/json':
        data = json.loads(request_body)
        return data
    raise ValueError(f'Unsupported content type: {content_type}')

def predict_fn(input_data, model):
    expected = input_data['expected_phoneme']
    transcribed = input_data['transcribed_text']
    confidence = input_data['confidence']
    
    # Calculate Levenshtein distance
    lev_distance = distance(expected, transcribed)
    
    # Encode phonemes (simplified)
    expected_encoded = hash(expected) % 1000
    transcribed_encoded = hash(transcribed) % 1000
    
    # Prepare features
    features = np.array([[
        expected_encoded,
        transcribed_encoded,
        confidence,
        len(transcribed),
        lev_distance
    ]])
    
    # Predict accuracy score
    score = model.predict(features)[0]
    
    # Determine grade
    if score >= 85:
        grade = 'correct'
    elif score >= 60:
        grade = 'partial'
    else:
        grade = 'incorrect'
    
    return {
        'score': float(score),
        'grade': grade,
        'confidence': confidence,
        'levenshtein_distance': lev_distance
    }

def output_fn(prediction, accept):
    if accept == 'application/json':
        return json.dumps(prediction), accept
    raise ValueError(f'Unsupported accept type: {accept}')
```

### 6.3 Recommendation Engine (Amazon Personalize)

**Dataset Preparation:**

```python
import boto3
import pandas as pd

personalize = boto3.client('personalize')

# Prepare interactions dataset
interactions_df = pd.DataFrame({
    'USER_ID': ['user1', 'user1', 'user2'],
    'ITEM_ID': ['18:00', '18:30', '19:00'],  # Practice times
    'TIMESTAMP': [1708020000, 1708106400, 1708192800],
    'EVENT_TYPE': ['practice', 'practice', 'practice'],
    'EVENT_VALUE': [1.0, 1.0, 1.0]  # Completion indicator
})

# Upload to S3
interactions_df.to_csv('s3://speech-play/personalize/interactions.csv', index=False)

# Create dataset group
response = personalize.create_dataset_group(
    name='reminder-optimization',
    domain='ECOMMERCE'  # Using ecommerce domain for recommendation
)

# Create schema
schema = {
    "type": "record",
    "name": "Interactions",
    "namespace": "com.amazonaws.personalize.schema",
    "fields": [
        {"name": "USER_ID", "type": "string"},
        {"name": "ITEM_ID", "type": "string"},
        {"name": "TIMESTAMP", "type": "long"},
        {"name": "EVENT_TYPE", "type": "string"},
        {"name": "EVENT_VALUE", "type": "float"}
    ],
    "version": "1.0"
}

# Create dataset and import data
# ... (standard Personalize setup)

# Create solution (recommendation algorithm)
personalize.create_solution(
    name='reminder-time-recommender',
    datasetGroupArn=dataset_group_arn,
    recipeArn='arn:aws:personalize:::recipe/aws-user-personalization'
)

# Deploy campaign
personalize.create_campaign(
    name='reminder-optimizer',
    solutionVersionArn=solution_version_arn,
    minProvisionedTPS=1
)
```

**Getting Recommendations:**

```python
def get_optimal_reminder_time(user_id):
    personalize_runtime = boto3.client('personalize-runtime')
    
    response = personalize_runtime.get_recommendations(
        campaignArn='arn:aws:personalize:ap-south-1:account:campaign/reminder-optimizer',
        userId=user_id,
        numResults=3
    )
    
    # Returns top 3 recommended times
    recommended_times = [item['itemId'] for item in response['itemList']]
    return recommended_times[0]  # Best time
```

### 6.4 Model Training & Deployment

**CI/CD Pipeline for ML Models:**

```yaml
# .github/workflows/ml-pipeline.yml
name: ML Model Training and Deployment

on:
  push:
    paths:
      - 'ml/models/**'
      - 'ml/training-data/**'

jobs:
  train-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2
      
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v1
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ap-south-1
      
      - name: Train model
        run: |
          python ml/train.py --data s3://speech-play/training-data/
      
      - name: Evaluate model
        run: |
          python ml/evaluate.py --model-path /tmp/model.joblib
      
      - name: Deploy to SageMaker
        if: success()
        run: |
          python ml/deploy.py --endpoint phoneme-grader
```

---

## 7. Security Design

### 7.1 Authentication & Authorization

**AWS Cognito Configuration:**

```javascript
// Cognito User Pool configuration
const userPoolConfig = {
  UserPoolName: 'speech-play-users',
  Policies: {
    PasswordPolicy: {
      MinimumLength: 8,
      RequireUppercase: true,
      RequireLowercase: true,
      RequireNumbers: true,
      RequireSymbols: true
    }
  },
  MfaConfiguration: 'OPTIONAL',
  AccountRecoverySetting: {
    RecoveryMechanisms: [
      { Name: 'verified_email', Priority: 1 },
      { Name: 'verified_phone_number', Priority: 2 }
    ]
  },
  Schema: [
    { Name: 'email', Required: true, Mutable: false },
    { Name: 'phone_number', Required: true, Mutable: true },
    { Name: 'custom:role', AttributeDataType: 'String', Mutable: false },
    { Name: 'custom:clinicId', AttributeDataType: 'String', Mutable: false }
  ]
};
```

**Role-Based Access Control (RBAC):**

```javascript
// IAM policies for different roles
const therapistPolicy = {
  Version: '2012-10-17',
  Statement: [
    {
      Effect: 'Allow',
      Action: [
        'dynamodb:GetItem',
        'dynamodb:PutItem',
        'dynamodb:UpdateItem',
        'dynamodb:Query'
      ],
      Resource: [
        'arn:aws:dynamodb:ap-south-1:account:table/Exercises',
        'arn:aws:dynamodb:ap-south-1:account:table/Assignments',
        'arn:aws:dynamodb:ap-south-1:account:table/Completions'
      ],
      Condition: {
        'ForAllValues:StringEquals': {
          'dynamodb:LeadingKeys': ['${cognito-identity.amazonaws.com:sub}']
        }
      }
    },
    {
      Effect: 'Allow',
      Action: ['s3:GetObject', 's3:PutObject'],
      Resource: 'arn:aws:s3:::speech-play-production/exercises/*'
    }
  ]
};

const parentPolicy = {
  Version: '2012-10-17',
  Statement: [
    {
      Effect: 'Allow',
      Action: ['dynamodb:GetItem', 'dynamodb:Query'],
      Resource: [
        'arn:aws:dynamodb:ap-south-1:account:table/Assignments',
        'arn:aws:dynamodb:ap-south-1:account:table/Completions'
      ],
      Condition: {
        'ForAllValues:StringEquals': {
          'dynamodb:LeadingKeys': ['${cognito-identity.amazonaws.com:sub}']
        }
      }
    },
    {
      Effect: 'Allow',
      Action: ['s3:GetObject'],
      Resource: 'arn:aws:s3:::speech-play-production/exercises/*'
    }
  ]
};
```

### 7.2 Data Encryption

**Encryption at Rest:**
- DynamoDB: AWS-managed keys (default encryption)
- RDS PostgreSQL: AES-256 encryption enabled
- S3: Server-side encryption (SSE-S3)
- ElastiCache Redis: Encryption at rest enabled

**Encryption in Transit:**
- TLS 1.3 for all API communications
- Certificate management via AWS Certificate Manager
- HTTPS-only CloudFront distributions
- Encrypted WebSocket connections (WSS)

**Voice Recording Encryption:**

```javascript
// Client-side encryption before upload
import CryptoJS from 'crypto-js';

async function encryptAndUpload(audioBlob, userId) {
  // Generate encryption key (derived from user's Cognito identity)
  const encryptionKey = await deriveKey(userId);
  
  // Convert blob to base64
  const base64Audio = await blobToBase64(audioBlob);
  
  // Encrypt
  const encrypted = CryptoJS.AES.encrypt(base64Audio, encryptionKey).toString();
  
  // Upload to S3
  const key = `recordings/${userId}/${Date.now()}.enc`;
  await Storage.put(key, encrypted, {
    contentType: 'application/octet-stream',
    metadata: {
      encrypted: 'true',
      algorithm: 'AES-256'
    }
  });
  
  return key;
}
```

### 7.3 API Security

**AWS WAF Configuration:**

```javascript
const wafRules = [
  {
    Name: 'RateLimitRule',
    Priority: 1,
    Statement: {
      RateBasedStatement: {
        Limit: 2000,
        AggregateKeyType: 'IP'
      }
    },
    Action: { Block: {} }
  },
  {
    Name: 'SQLInjectionRule',
    Priority: 2,
    Statement: {
      ManagedRuleGroupStatement: {
        VendorName: 'AWS',
        Name: 'AWSManagedRulesSQLiRuleSet'
      }
    },
    Action: { Block: {} }
  },
  {
    Name: 'GeoBlockingRule',
    Priority: 3,
    Statement: {
      GeoMatchStatement: {
        CountryCodes: ['IN']  // Only allow traffic from India
      }
    },
    Action: { Allow: {} }
  }
];
```

**API Gateway Security:**

```javascript
// Request validation
const requestValidator = {
  validateRequestBody: true,
  validateRequestParameters: true
};

// CORS configuration
const corsConfig = {
  allowOrigins: ['https://app.speechplay.com', 'https://dashboard.speechplay.com'],
  allowMethods: ['GET', 'POST', 'PUT', 'DELETE', 'OPTIONS'],
  allowHeaders: ['Content-Type', 'Authorization', 'X-Api-Key'],
  maxAge: 3600
};

// Throttling
const throttleSettings = {
  rateLimit: 1000,  // requests per second
  burstLimit: 2000
};
```

### 7.4 GDPR/Indian Data Law Compliance

**Data Retention Policies:**

```javascript
const retentionPolicies = {
  userProfiles: 'Indefinite (until account deletion)',
  voiceRecordings: '90 days (then moved to Glacier)',
  completionData: '2 years',
  auditLogs: '7 years',
  messages: '90 days'
};

// Automated data deletion
async function deleteUserData(userId) {
  // Delete from DynamoDB
  await dynamodb.deleteItem({ TableName: 'Users', Key: { userId } });
  await dynamodb.query({
    TableName: 'Completions',
    IndexName: 'StudentIdIndex',
    KeyConditionExpression: 'studentId = :userId',
    ExpressionAttributeValues: { ':userId': userId }
  }).then(items => {
    items.forEach(item => dynamodb.deleteItem({ TableName: 'Completions', Key: { completionId: item.completionId } }));
  });
  
  // Delete from PostgreSQL
  await db.query('DELETE FROM children WHERE user_id = $1', [userId]);
  
  // Delete from S3
  const recordings = await s3.listObjectsV2({
    Bucket: 'speech-play-production',
    Prefix: `recordings/${userId}/`
  });
  await s3.deleteObjects({
    Bucket: 'speech-play-production',
    Delete: { Objects: recordings.Contents.map(obj => ({ Key: obj.Key })) }
  });
  
  // Anonymize audit logs
  await db.query('UPDATE audit_logs SET user_id = $1 WHERE user_id = $2', ['DELETED_USER', userId]);
}
```

**Consent Management:**

```javascript
const consentTypes = {
  dataCollection: 'Required for service',
  voiceRecording: 'Required for speech therapy',
  dataSharing: 'Optional (for research)',
  marketing: 'Optional'
};

// Store consent in DynamoDB
async function recordConsent(userId, consentType, granted) {
  await dynamodb.putItem({
    TableName: 'UserConsents',
    Item: {
      userId,
      consentType,
      granted,
      timestamp: Date.now(),
      ipAddress: req.ip
    }
  });
}
```

---

## 8. Deployment Architecture

### 8.1 AWS Infrastructure Setup

**VPC Configuration:**

```
VPC: 10.0.0.0/16
├── Public Subnets (for ALB, NAT Gateway)
│   ├── 10.0.1.0/24 (ap-south-1a)
│   └── 10.0.2.0/24 (ap-south-1b)
├── Private Subnets (for ECS, RDS, ElastiCache)
│   ├── 10.0.10.0/24 (ap-south-1a)
│   └── 10.0.11.0/24 (ap-south-1b)
└── Database Subnets (for RDS)
    ├── 10.0.20.0/24 (ap-south-1a)
    └── 10.0.21.0/24 (ap-south-1b)
```

**ECS Cluster Configuration:**

```yaml
# ecs-task-definition.json
{
  "family": "speech-play-backend",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "512",
  "memory": "1024",
  "containerDefinitions": [
    {
      "name": "user-service",
      "image": "account.dkr.ecr.ap-south-1.amazonaws.com/speech-play/user-service:latest",
      "portMappings": [
        {
          "containerPort": 8080,
          "protocol": "tcp"
        }
      ],
      "environment": [
        {"name": "SPRING_PROFILES_ACTIVE", "value": "production"},
        {"name": "AWS_REGION", "value": "ap-south-1"}
      ],
      "secrets": [
        {"name": "DB_PASSWORD", "valueFrom": "arn:aws:secretsmanager:ap-south-1:account:secret:db-password"},
        {"name": "JWT_SECRET", "valueFrom": "arn:aws:secretsmanager:ap-south-1:account:secret:jwt-secret"}
      ],
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/speech-play",
          "awslogs-region": "ap-south-1",
          "awslogs-stream-prefix": "user-service"
        }
      }
    }
  ]
}
```


### 8.2 CI/CD Pipeline

**GitHub Actions Workflow:**

```yaml
# .github/workflows/deploy.yml
name: Deploy to AWS

on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Set up JDK 17
        uses: actions/setup-java@v2
        with:
          java-version: '17'
      - name: Run tests
        run: ./gradlew test
      - name: Run integration tests
        run: ./gradlew integrationTest

  build-and-push:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v1
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ap-south-1
      
      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v1
      
      - name: Build, tag, and push image
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          ECR_REPOSITORY: speech-play/user-service
          IMAGE_TAG: ${{ github.sha }}
        run: |
          docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
          docker tag $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG $ECR_REGISTRY/$ECR_REPOSITORY:latest
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:latest

  deploy:
    needs: build-and-push
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to ECS
        run: |
          aws ecs update-service \
            --cluster speech-play-cluster \
            --service user-service \
            --force-new-deployment \
            --region ap-south-1
      
      - name: Wait for deployment
        run: |
          aws ecs wait services-stable \
            --cluster speech-play-cluster \
            --services user-service \
            --region ap-south-1

  deploy-frontend:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Set up Node.js
        uses: actions/setup-node@v2
        with:
          node-version: '18'
      
      - name: Install dependencies
        run: npm ci
        working-directory: ./frontend
      
      - name: Build
        run: npm run build
        working-directory: ./frontend
        env:
          REACT_APP_API_URL: https://api.speechplay.com
      
      - name: Deploy to Amplify
        uses: aws-amplify/amplify-cli-action@v0.3.0
        with:
          amplify_command: publish
          amplify_env: production
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          AWS_REGION: ap-south-1
```

### 8.3 Monitoring & Logging

**CloudWatch Configuration:**

```javascript
// CloudWatch Alarms
const alarms = [
  {
    AlarmName: 'HighAPILatency',
    MetricName: 'Latency',
    Namespace: 'AWS/ApiGateway',
    Statistic: 'Average',
    Period: 300,
    EvaluationPeriods: 2,
    Threshold: 1000,  // 1 second
    ComparisonOperator: 'GreaterThanThreshold',
    AlarmActions: ['arn:aws:sns:ap-south-1:account:alerts']
  },
  {
    AlarmName: 'HighErrorRate',
    MetricName: '5XXError',
    Namespace: 'AWS/ApiGateway',
    Statistic: 'Sum',
    Period: 300,
    EvaluationPeriods: 1,
    Threshold: 10,
    ComparisonOperator: 'GreaterThanThreshold',
    AlarmActions: ['arn:aws:sns:ap-south-1:account:alerts']
  },
  {
    AlarmName: 'LowDiskSpace',
    MetricName: 'FreeStorageSpace',
    Namespace: 'AWS/RDS',
    Statistic: 'Average',
    Period: 300,
    EvaluationPeriods: 1,
    Threshold: 10737418240,  // 10 GB
    ComparisonOperator: 'LessThanThreshold',
    AlarmActions: ['arn:aws:sns:ap-south-1:account:alerts']
  }
];

// Custom metrics
const customMetrics = {
  ComplianceRate: {
    namespace: 'SpeechPlay',
    metricName: 'ComplianceRate',
    dimensions: [{ Name: 'ClinicId', Value: 'clinic-uuid' }],
    value: 85,
    unit: 'Percent'
  },
  SpeechRecognitionAccuracy: {
    namespace: 'SpeechPlay',
    metricName: 'SpeechRecognitionAccuracy',
    dimensions: [{ Name: 'Language', Value: 'hindi' }],
    value: 87,
    unit: 'Percent'
  }
};
```

**Structured Logging:**

```java
// Spring Boot logging configuration
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import net.logstash.logback.argument.StructuredArguments;

@Service
public class UserService {
    private static final Logger logger = LoggerFactory.getLogger(UserService.class);
    
    public User createUser(UserRequest request) {
        logger.info("Creating user",
            StructuredArguments.kv("email", request.getEmail()),
            StructuredArguments.kv("role", request.getRole()),
            StructuredArguments.kv("clinicId", request.getClinicId())
        );
        
        try {
            User user = userRepository.save(request.toEntity());
            logger.info("User created successfully",
                StructuredArguments.kv("userId", user.getId())
            );
            return user;
        } catch (Exception e) {
            logger.error("Failed to create user",
                StructuredArguments.kv("email", request.getEmail()),
                StructuredArguments.kv("error", e.getMessage())
            );
            throw e;
        }
    }
}
```

**X-Ray Tracing:**

```java
// Enable X-Ray tracing
import com.amazonaws.xray.spring.aop.XRayEnabled;

@Service
@XRayEnabled
public class ExerciseService {
    
    @Autowired
    private DynamoDBMapper dynamoDBMapper;
    
    @Autowired
    private S3Client s3Client;
    
    public Exercise createExercise(ExerciseRequest request) {
        // X-Ray automatically traces DynamoDB and S3 calls
        Exercise exercise = new Exercise();
        exercise.setTitle(request.getTitle());
        
        // Upload audio to S3 (traced)
        String audioUrl = uploadAudio(request.getAudioFile());
        exercise.setAudioUrl(audioUrl);
        
        // Save to DynamoDB (traced)
        dynamoDBMapper.save(exercise);
        
        return exercise;
    }
}
```

### 8.4 Auto-Scaling Configuration

**ECS Auto-Scaling:**

```javascript
const ecsAutoScaling = {
  ServiceName: 'user-service',
  MinCapacity: 2,
  MaxCapacity: 10,
  TargetTrackingScalingPolicies: [
    {
      PolicyName: 'cpu-scaling',
      TargetValue: 70.0,
      PredefinedMetricSpecification: {
        PredefinedMetricType: 'ECSServiceAverageCPUUtilization'
      },
      ScaleInCooldown: 300,
      ScaleOutCooldown: 60
    },
    {
      PolicyName: 'memory-scaling',
      TargetValue: 80.0,
      PredefinedMetricSpecification: {
        PredefinedMetricType: 'ECSServiceAverageMemoryUtilization'
      },
      ScaleInCooldown: 300,
      ScaleOutCooldown: 60
    }
  ]
};
```

**DynamoDB Auto-Scaling:**

```javascript
const dynamoAutoScaling = {
  TableName: 'Completions',
  MinReadCapacity: 5,
  MaxReadCapacity: 100,
  MinWriteCapacity: 5,
  MaxWriteCapacity: 100,
  TargetTrackingScalingPolicies: [
    {
      PolicyName: 'read-scaling',
      TargetValue: 70.0,
      PredefinedMetricType: 'DynamoDBReadCapacityUtilization'
    },
    {
      PolicyName: 'write-scaling',
      TargetValue: 70.0,
      PredefinedMetricType: 'DynamoDBWriteCapacityUtilization'
    }
  ]
};
```

**Lambda Concurrency:**

```javascript
const lambdaConfig = {
  FunctionName: 'progress-calculator',
  ReservedConcurrentExecutions: 50,  // Reserve capacity
  ProvisionedConcurrencyConfig: {
    ProvisionedConcurrentExecutions: 10  // Keep warm
  }
};
```

---

## 9. UI/UX Design Principles

### 9.1 Mobile-First Design

**Design System:**

```javascript
// Design tokens
const designTokens = {
  colors: {
    primary: '#FF6B6B',      // Playful red
    secondary: '#4ECDC4',    // Calming teal
    success: '#95E1D3',      // Light green
    warning: '#FFE66D',      // Yellow
    error: '#FF6B6B',        // Red
    background: '#F7F7F7',   // Light gray
    text: '#2C3E50',         // Dark gray
    textLight: '#7F8C8D'     // Medium gray
  },
  typography: {
    fontFamily: 'Poppins, sans-serif',
    sizes: {
      xs: '12px',
      sm: '14px',
      md: '16px',
      lg: '20px',
      xl: '24px',
      xxl: '32px'
    },
    weights: {
      regular: 400,
      medium: 500,
      semibold: 600,
      bold: 700
    }
  },
  spacing: {
    xs: '4px',
    sm: '8px',
    md: '16px',
    lg: '24px',
    xl: '32px',
    xxl: '48px'
  },
  borderRadius: {
    sm: '4px',
    md: '8px',
    lg: '16px',
    full: '9999px'
  }
};
```

### 9.2 Gamification Elements (for Kids)

**Reward System:**

```javascript
const rewardSystem = {
  stars: {
    perExercise: 1,
    perfectScore: 2,
    dailyStreak: 1
  },
  badges: [
    { id: 'first-exercise', name: 'First Steps', requirement: 'Complete 1 exercise' },
    { id: '3-day-streak', name: 'Consistent Learner', requirement: '3-day practice streak' },
    { id: 'perfect-score', name: 'Perfect!', requirement: 'Score 100% on any exercise' },
    { id: '100-exercises', name: 'Century', requirement: 'Complete 100 exercises' },
    { id: 'all-phonemes', name: 'Master Speaker', requirement: 'Practice all phonemes' }
  ],
  levels: {
    xpPerLevel: 100,
    maxLevel: 50,
    rewards: {
      5: 'Unlock new avatar',
      10: 'Unlock new theme',
      20: 'Unlock special badge',
      50: 'Master certificate'
    }
  }
};
```

**Animation Guidelines:**

```javascript
// Lottie animations for feedback
const animations = {
  success: 'confetti.json',      // Celebration animation
  incorrect: 'try-again.json',   // Encouraging animation
  levelUp: 'level-up.json',      // Level progression
  badgeUnlock: 'badge.json',     // Badge unlock
  loading: 'loading.json'        // Loading state
};

// Animation timing
const animationDurations = {
  feedback: 1500,    // 1.5 seconds
  transition: 300,   // 0.3 seconds
  loading: 2000      // 2 seconds max
};
```

### 9.3 Simple Parent Interface (Low Tech Literacy)

**Design Principles:**
- Large, clear buttons (minimum 60px height)
- High contrast colors (WCAG AAA)
- Minimal text, maximum visuals
- Single-column layout on mobile
- No hidden menus or complex navigation

**Component Example:**

```jsx
// ProgressCard.tsx
import React from 'react';
import { Card, ProgressBar, Icon } from './components';

const ProgressCard = ({ childName, completionRate, accuracy }) => {
  return (
    <Card style={styles.card}>
      <Icon name="child" size={48} color="#4ECDC4" />
      <Text style={styles.childName}>{childName}</Text>
      
      <View style={styles.metric}>
        <Text style={styles.label}>Practice Completion</Text>
        <ProgressBar value={completionRate} color="#95E1D3" />
        <Text style={styles.value}>{completionRate}%</Text>
      </View>
      
      <View style={styles.metric}>
        <Text style={styles.label}>Accuracy</Text>
        <ProgressBar value={accuracy} color="#FFE66D" />
        <Text style={styles.value}>{accuracy}%</Text>
      </View>
      
      <Button 
        title="View Details" 
        onPress={() => navigate('Details')}
        style={styles.button}
      />
    </Card>
  );
};

const styles = {
  card: {
    padding: 24,
    borderRadius: 16,
    backgroundColor: '#FFFFFF',
    shadowColor: '#000',
    shadowOpacity: 0.1,
    shadowRadius: 10
  },
  childName: {
    fontSize: 24,
    fontWeight: 'bold',
    color: '#2C3E50',
    marginTop: 16
  },
  metric: {
    marginTop: 24
  },
  label: {
    fontSize: 14,
    color: '#7F8C8D',
    marginBottom: 8
  },
  value: {
    fontSize: 20,
    fontWeight: 'bold',
    color: '#2C3E50',
    marginTop: 8
  },
  button: {
    marginTop: 24,
    height: 60,
    borderRadius: 12,
    backgroundColor: '#4ECDC4'
  }
};
```

### 9.4 Professional Therapist Dashboard

**Dashboard Layout:**

```
┌─────────────────────────────────────────────────────────────┐
│  Speech Play Dashboard                    [Profile] [Logout] │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │
│  │ Total       │  │ Compliance  │  │ Avg Accuracy│         │
│  │ Students    │  │ Rate        │  │             │         │
│  │    45       │  │    85%      │  │    82%      │         │
│  └─────────────┘  └─────────────┘  └─────────────┘         │
│                                                               │
│  ┌───────────────────────────────────────────────────────┐  │
│  │ Student Compliance (Real-Time)                        │  │
│  ├───────────────────────────────────────────────────────┤  │
│  │ Name          Last Practice  Completion  Accuracy     │  │
│  │ Aarav Kumar   2 hours ago    100%        85%  🟢      │  │
│  │ Priya Sharma  5 hours ago    80%         78%  🟡      │  │
│  │ Rohan Patel   2 days ago     20%         65%  🔴      │  │
│  └───────────────────────────────────────────────────────┘  │
│                                                               │
│  ┌───────────────────────────────────────────────────────┐  │
│  │ Weekly Progress Trends                                │  │
│  │ [Line Chart: Compliance Rate Over Time]              │  │
│  └───────────────────────────────────────────────────────┘  │
│                                                               │
└─────────────────────────────────────────────────────────────┘
```

### 9.5 Accessibility Guidelines

**WCAG 2.1 Level AA Compliance:**

```javascript
const accessibilityFeatures = {
  colorContrast: {
    normalText: '4.5:1',      // WCAG AA
    largeText: '3:1',         // WCAG AA
    uiComponents: '3:1'       // WCAG AA
  },
  textSizing: {
    minSize: '16px',
    scalable: true,
    maxScale: '200%'
  },
  touchTargets: {
    minSize: '44x44px',       // iOS guidelines
    spacing: '8px'
  },
  screenReader: {
    ariaLabels: true,
    semanticHTML: true,
    altText: true
  },
  keyboardNavigation: {
    tabIndex: true,
    focusIndicators: true,
    skipLinks: true
  }
};
```

---

## 10. Performance Optimization

### 10.1 CDN Strategy (CloudFront)

**Cache Behaviors:**

```javascript
const cloudFrontConfig = {
  origins: [
    {
      id: 'api-origin',
      domainName: 'api.speechplay.com',
      customHeaders: [
        { headerName: 'X-CDN-Request', headerValue: 'true' }
      ]
    },
    {
      id: 's3-origin',
      domainName: 'speech-play-production.s3.ap-south-1.amazonaws.com',
      s3OriginConfig: {
        originAccessIdentity: 'origin-access-identity/cloudfront/XXXXX'
      }
    }
  ],
  cacheBehaviors: [
    {
      pathPattern: '/api/v1/exercises',
      targetOriginId: 'api-origin',
      viewerProtocolPolicy: 'redirect-to-https',
      allowedMethods: ['GET', 'HEAD', 'OPTIONS'],
      cachedMethods: ['GET', 'HEAD'],
      compress: true,
      minTTL: 0,
      defaultTTL: 300,      // 5 minutes
      maxTTL: 3600,         // 1 hour
      forwardedValues: {
        queryString: true,
        headers: ['Authorization']
      }
    },
    {
      pathPattern: '/exercises/*',
      targetOriginId: 's3-origin',
      viewerProtocolPolicy: 'redirect-to-https',
      compress: true,
      minTTL: 86400,        // 1 day
      defaultTTL: 604800,   // 7 days
      maxTTL: 31536000      // 1 year
    }
  ]
};
```

### 10.2 Lazy Loading

**React Native:**

```javascript
// Lazy load screens
const HomeScreen = React.lazy(() => import('./screens/HomeScreen'));
const ExerciseScreen = React.lazy(() => import('./screens/ExerciseScreen'));
const RewardsScreen = React.lazy(() => import('./screens/RewardsScreen'));

// Lazy load images
import FastImage from 'react-native-fast-image';

<FastImage
  source={{
    uri: exerciseImageUrl,
    priority: FastImage.priority.normal,
    cache: FastImage.cacheControl.immutable
  }}
  resizeMode={FastImage.resizeMode.contain}
/>
```

### 10.3 Image Optimization

**Automated Image Processing:**

```javascript
// Lambda function for image optimization
const sharp = require('sharp');

exports.handler = async (event) => {
  const bucket = event.Records[0].s3.bucket.name;
  const key = event.Records[0].s3.object.key;
  
  // Download original image
  const originalImage = await s3.getObject({ Bucket: bucket, Key: key }).promise();
  
  // Generate WebP versions
  const thumbnail = await sharp(originalImage.Body)
    .resize(200, 200, { fit: 'cover' })
    .webp({ quality: 80 })
    .toBuffer();
  
  const full = await sharp(originalImage.Body)
    .resize(800, 800, { fit: 'inside' })
    .webp({ quality: 85 })
    .toBuffer();
  
  // Upload optimized versions
  await s3.putObject({
    Bucket: bucket,
    Key: key.replace('.png', '_thumbnail.webp'),
    Body: thumbnail,
    ContentType: 'image/webp'
  }).promise();
  
  await s3.putObject({
    Bucket: bucket,
    Key: key.replace('.png', '_full.webp'),
    Body: full,
    ContentType: 'image/webp'
  }).promise();
};
```

### 10.4 Database Query Optimization

**DynamoDB Best Practices:**

```javascript
// Use batch operations
const batchGetItems = async (exerciseIds) => {
  const params = {
    RequestItems: {
      'Exercises': {
        Keys: exerciseIds.map(id => ({ exerciseId: id }))
      }
    }
  };
  return await dynamodb.batchGetItem(params).promise();
};

// Use projection expressions to fetch only needed attributes
const getExercise = async (exerciseId) => {
  const params = {
    TableName: 'Exercises',
    Key: { exerciseId },
    ProjectionExpression: 'exerciseId, title, phoneme, audioUrl, imageUrl'
  };
  return await dynamodb.getItem(params).promise();
};

// Use GSI for efficient queries
const getAssignmentsByStudent = async (studentId) => {
  const params = {
    TableName: 'Assignments',
    IndexName: 'StudentIdIndex',
    KeyConditionExpression: 'studentId = :sid',
    ExpressionAttributeValues: { ':sid': studentId },
    Limit: 20
  };
  return await dynamodb.query(params).promise();
};
```

**PostgreSQL Optimization:**

```sql
-- Create indexes for frequent queries
CREATE INDEX idx_children_therapist_active ON children(therapist_id) WHERE therapy_start_date IS NOT NULL;
CREATE INDEX idx_subscriptions_active ON subscriptions(clinic_id, status) WHERE status = 'active';

-- Use materialized views for analytics
CREATE MATERIALIZED VIEW clinic_analytics AS
SELECT 
  c.clinic_id,
  c.name,
  COUNT(DISTINCT t.therapist_id) as total_therapists,
  COUNT(DISTINCT ch.child_id) as total_children,
  AVG(comp.accuracy) as avg_accuracy
FROM clinics c
LEFT JOIN therapists t ON c.clinic_id = t.clinic_id
LEFT JOIN children ch ON t.therapist_id = ch.therapist_id
LEFT JOIN completions comp ON ch.user_id = comp.student_id
GROUP BY c.clinic_id, c.name;

-- Refresh materialized view daily
REFRESH MATERIALIZED VIEW CONCURRENTLY clinic_analytics;
```

### 10.5 Caching Strategy

**Multi-Layer Caching:**

```javascript
// Layer 1: Browser cache (service worker)
// Layer 2: CDN cache (CloudFront)
// Layer 3: Application cache (Redis)
// Layer 4: Database

const getCachedExercise = async (exerciseId) => {
  // Check Redis cache
  const cached = await redis.get(`exercise:${exerciseId}`);
  if (cached) {
    return JSON.parse(cached);
  }
  
  // Fetch from DynamoDB
  const exercise = await dynamodb.getItem({
    TableName: 'Exercises',
    Key: { exerciseId }
  }).promise();
  
  // Store in Redis with 24-hour TTL
  await redis.setex(`exercise:${exerciseId}`, 86400, JSON.stringify(exercise.Item));
  
  return exercise.Item;
};
```

---

## 11. Cost Estimation & Optimization

**Monthly Cost Breakdown (for 5,000 active users):**

| Service | Usage | Cost (INR) |
|---------|-------|------------|
| EC2 (ECS) | 4 t3.medium instances | ₹8,000 |
| RDS PostgreSQL | db.t3.medium | ₹6,000 |
| DynamoDB | 10M reads, 5M writes | ₹3,000 |
| S3 | 500 GB storage, 1 TB transfer | ₹2,500 |
| CloudFront | 1 TB data transfer | ₹5,000 |
| Lambda | 10M invocations | ₹1,000 |
| Transcribe | 100 hours audio | ₹8,000 |
| SageMaker | ml.t2.medium endpoint | ₹4,000 |
| ElastiCache | cache.t3.micro | ₹2,000 |
| **Total** | | **₹39,500/month** |

**Revenue (5,000 users):**
- 50 therapists × ₹1,500 = ₹75,000
- 5,000 parents × ₹100 = ₹5,00,000
- **Total Revenue: ₹5,75,000/month**
- **Profit Margin: 93%**

---

**Document Version**: 3.0  
**Last Updated**: February 15, 2026  
**Authors**: Speech Play Engineering Team  
**Status**: Final for Hackathon Submission
