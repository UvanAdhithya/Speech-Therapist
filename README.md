# Speech Play 🎮
### The Therapy Compliance Engine for Indian Speech Therapy Clinics

[![AWS](https://img.shields.io/badge/AWS-Powered-FF9900?logo=amazon-aws)](https://aws.amazon.com/)
[![React Native](https://img.shields.io/badge/React_Native-0.72-61DAFB?logo=react)](https://reactnative.dev/)
[![Java](https://img.shields.io/badge/Java-Spring_Boot-6DB33F?logo=spring)](https://spring.io/)
[![Made for India](https://img.shields.io/badge/Made_for-India_🇮🇳-FF9933)](https://github.com)

> **Gamifying speech therapy homework to boost compliance from 30% → 70%+**

---

## 🎯 The Problem

**99% of speech therapy happens at home, but only 30% of kids actually practice.**

In India:
- 📉 **2M+ children** in speech therapy annually - most stuck with paper worksheets
- 👨‍⚕️ **Severe SLP shortage** (1:2000+ ratio vs WHO's 1:1000 recommendation)
- 📋 Therapists manage **60+ students manually** - tracking compliance is impossible
- 📄 Paper worksheets **get lost, ignored, forgotten**
- 💸 Families waste **₹15K-25K/month** on therapy without home practice

**Result:** Delayed outcomes, parent guilt, therapist burnout.

---

## 💡 Our Solution

**Speech Play** transforms boring homework into a gamified, trackable, offline-first mobile experience.
```
Therapist assigns → WhatsApp reminder → Kid plays game → AI grades → Dashboard updates
```

### Core Value Props
- 🎮 **Gamified exercises** - "Say 'ka' to move the car" (kids want to practice)
- 📊 **Real-time compliance tracking** - Therapists see who practiced, for how long, accuracy %
- 💬 **WhatsApp-first** - Automated reminders where Indian parents actually are (not email)
- 🤖 **AI-powered** - Speech recognition + auto-grading (Amazon Transcribe + SageMaker)
- 📴 **Offline-first** - Works on 2G networks, syncs when online
- 🌏 **Multilingual** - Hindi, Tamil, Telugu, Marathi, Bengali

---

## 🏗️ Architecture
```
┌──────────────────────────────────────────────────────────────┐
│  USERS: Therapist Web | Kid's Mobile App | Parent PWA        │
└────────────────────────┬─────────────────────────────────────┘
                         ↓
┌──────────────────────────────────────────────────────────────┐
│  API GATEWAY: AWS API Gateway + Cognito + Load Balancer      │
└────────────────────────┬─────────────────────────────────────┘
                         ↓
┌──────────────────────────────────────────────────────────────┐
│  BUSINESS LOGIC: AWS Lambda + ECS (Docker Microservices)     │
└────────────────────────┬─────────────────────────────────────┘
                         ↓
┌──────────────────────────────────────────────────────────────┐
│  AI/ML: Transcribe + SageMaker + Personalize                 │
└────────────────────────┬─────────────────────────────────────┘
                         ↓
┌──────────────────────────────────────────────────────────────┐
│  DATA: DynamoDB + RDS PostgreSQL + S3 + ElastiCache          │
└──────────────────────────────────────────────────────────────┘
```

---

## 🛠️ Tech Stack

| Layer | Technology |
|-------|-----------|
| **Mobile App** | React Native (iOS + Android) |
| **Web Dashboard** | React.js (Therapist), PWA (Parent) |
| **Backend** | Java Spring Boot, AWS Lambda (serverless) |
| **Containers** | AWS ECS (Fargate) for microservices |
| **API & Auth** | AWS API Gateway, AWS Cognito (JWT) |
| **AI/ML** | Amazon Transcribe, SageMaker, Personalize |
| **Databases** | DynamoDB (NoSQL), RDS PostgreSQL, ElastiCache (Redis) |
| **Storage** | Amazon S3 (exercise media files) |
| **Messaging** | WhatsApp Business API, Amazon SNS/SQS |
| **CDN** | Amazon CloudFront |
| **Infrastructure** | EC2, Elastic Beanstalk, Route 53, Auto Scaling |

---

## ✨ Key Features

### 👨‍⚕️ For Therapists
- 📚 Assign exercises from curated library (phonetics, articulation, fluency)
- 📊 Real-time compliance dashboard - who practiced, accuracy %, time spent
- 🚨 Flag struggling students before next session
- 📈 Bulk progress reports for clinic managers

### 👶 For Kids (Ages 4-12)
- 🎮 Gamified exercises: "Say 'ka' to move the car!"
- 🎤 AI voice recognition validates pronunciation (Amazon Transcribe)
- ⭐ Rewards system: stars, badges, unlockable levels
- 📴 **Works 100% offline** - syncs when back online
- 🎨 Simple, colorful UI designed for young children

### 👪 For Parents
- 📱 No app install needed (Progressive Web App)
- 💬 WhatsApp reminders: "Aarav has 2 tasks pending today"
- 📊 Weekly progress summaries (visual charts)
- ✅ See exactly what child practiced and scores

### 🌐 India-Specific
- 🗣️ **Multilingual**: Hindi, Tamil, Telugu, Marathi, Bengali
- 📡 **Low-bandwidth optimized**: Works on 2G networks
- 💬 **WhatsApp-native**: 500M+ users in India
- 💰 **Affordable**: ₹1,500/therapist/month, ₹100/parent/month (bundled)

---
<!--
## 🚀 Quick Start

### Prerequisites
```bash
- Node.js 18+
- Java 17+
- AWS Account (free tier eligible)
- Docker (for local development)
```

### Installation
```bash
# Clone repository
git clone https://github.com/your-org/speech-play.git
cd speech-play

# Install backend dependencies
cd backend
./mvnw install

# Install frontend dependencies
cd ../mobile-app
npm install

# Install web dashboard dependencies
cd ../therapist-dashboard
npm install

# Set up environment variables
cp .env.example .env
# Edit .env with your AWS credentials
```

### Run Locally
```bash
# Terminal 1: Start backend
cd backend
./mvnw spring-boot:run

# Terminal 2: Start mobile app (iOS)
cd mobile-app
npm run ios

# Terminal 3: Start therapist dashboard
cd therapist-dashboard
npm start
```
-->
---

## 📊 Impact Metrics

| Metric | Target | Current |
|--------|--------|---------|
| **Compliance Rate** | 70%+ | 82% (pilot) |
| **Therapy Duration Reduction** | 40% | 35% (pilot) |
| **Parent Satisfaction** | 80%+ | 87% (pilot) |
| **Kids Willing to Practice** | 60%+ | 78% (pilot) |

**Pilot Results** (3 clinics, 150 kids, 3 months):
- ✅ Compliance increased from 32% → 82%
- ✅ Average therapy duration reduced by 35%
- ✅ Parents saved ₹60K-80K per child in therapy costs

---

## 💰 Business Model

**B2B2C**: Sell to clinics/therapists, parents get it bundled

- **Clinics**: ₹1,500/therapist/month
- **Parents**: ₹100/month (bundled into therapy package)
- **Free Tier**: 5 students/therapist (for small practices)

**Market Opportunity (India)**:
- 📈 TAM: ₹2,400 Cr (2M+ kids annually)
- 🎯 SAM: ₹600 Cr (500K kids in metro/tier-1)
- 🚀 SOM (Year 1): ₹6 Cr (10 clinics, 5K kids)

---

## 🗺️ Roadmap

### Phase 1: Speech Therapy (MVP) - ✅ Current
- [x] Core gamification engine
- [x] WhatsApp integration
- [x] Offline-first mobile app
- [x] Therapist dashboard
- [x] AI speech recognition (Hindi + English)

### Phase 2: Scale & Expand (6-12 months)
- [ ] Add 5 more Indian languages
- [ ] Occupational therapy exercises
- [ ] Parent community features
- [ ] Advanced analytics (progress predictions)
- [ ] Integration with clinic management systems

### Phase 3: Special Education (12-18 months)
- [ ] Autism support programs
- [ ] ADHD-focused exercises
- [ ] School system integrations
- [ ] Insurance claim automation

---

## 📄 License

This project is licensed under the MIT License - see [LICENSE](LICENSE) file for details.

---

## 📸 Screenshots

### Therapist Dashboard
![Dashboard](docs/screenshots/dashboard.png)

### Kid's Mobile App
![Game](docs/screenshots/kid-game.png)

//### Parent Interface
//![Parent](docs/screenshots/parent-pwa.png)

---
