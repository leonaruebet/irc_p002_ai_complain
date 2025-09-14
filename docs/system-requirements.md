# System Requirements Document

## HR Complaint Management System with AI Analytics

**Version**: 1.0
**Date**: September 14, 2025
**Project**: irc_p002_ai_complain

---

## Executive Summary

This document outlines the comprehensive system requirements for an HR complaint management system that integrates with LINE Official Account for employee complaint submission and provides AI-powered analytics for HR teams.

### Key Features
- 💬 **LINE OA Integration**: Employees submit complaints via familiar chat interface
- 🤖 **AI Analytics**: Automatic sentiment analysis and keyword tagging using Google Gemini
- 📊 **Visual Dashboard**: Real-time analytics with sentiment trends and word clouds
- 🔒 **Secure Architecture**: MongoDB Atlas with tRPC type-safe APIs
- ⚡ **Modern Stack**: Next.js 15, Express.js, TypeScript throughout

---

## 1. Functional Requirements

### 1.1 Employee Complaint Submission (LINE OA)

#### 1.1.1 Core Commands
| Command | Purpose | Expected Behavior |
|---------|---------|-------------------|
| `/complain` | Start new complaint session | Bot responds with guidance, creates session in DB |
| `/submit` | Finalize complaint | Bot confirms submission, triggers AI analysis |
| Free text | Add complaint details | Messages stored in chat logs |

#### 1.1.2 User Experience Flow
```
1. Employee sends /complain → Session starts
2. Employee describes issue → Messages logged
3. Employee sends /submit → Complaint submitted
4. Bot confirms with complaint ID → AI analysis begins
```

#### 1.1.3 Validation Rules
- Only registered LINE users can submit complaints
- Each user can have max 1 active (open) session at a time
- Sessions automatically timeout after 24 hours without submission
- Minimum complaint text: 10 characters (excluding commands)

### 1.2 AI Analysis Engine

#### 1.2.1 Sentiment Analysis
- **Provider**: Google Gemini 1.5 Flash
- **Classifications**: Positive, Negative, Neutral
- **Confidence Score**: 0.0 - 1.0 scale
- **Processing Trigger**: When complaint status changes to "submitted"
- **Fallback**: Default to "neutral" with 0.5 confidence if AI fails

#### 1.2.2 Keyword Extraction
- **Count**: 3-5 relevant keywords per complaint
- **Categories**: Predefined workplace categories
  - Management, Workload, Harassment, Salary, Environment
  - Communication, Safety, Discrimination, Career Development
- **Language**: Support for English and Thai text
- **Quality Control**: Filter out stop words and irrelevant terms

### 1.3 HR Dashboard

#### 1.3.1 Complaint Management
- **List View**: Paginated complaints with filtering by status, department, date
- **Detail View**: Complete conversation history with timestamps
- **Search**: By complaint ID, date range, keywords
- **Export**: CSV export for reporting (future enhancement)

#### 1.3.2 AI Analytics Dashboard
- **Sentiment Overview**: Cards showing positive/negative/neutral distribution
- **Word Cloud**: Visual representation of most common complaint topics
- **Keyword Chart**: Bar chart of top complaint categories
- **Trend Analysis**: Sentiment patterns over time (monthly view)

### 1.4 Data Management

#### 1.4.1 Employee Registry
- **Data Source**: Manual entry via admin interface
- **Required Fields**: LINE user ID, display name
- **Optional Fields**: Department, employee status
- **Maintenance**: Add/update employee records as needed

#### 1.4.2 Audit & Compliance
- **LINE Events**: Raw webhook data stored for 60 days (debugging)
- **Complaint History**: 7-year retention for compliance
- **AI Analysis**: Permanent storage linked to complaints
- **Data Export**: Ability to extract user's complaint data (GDPR)

---

## 2. Non-Functional Requirements

### 2.1 Performance Requirements

| Metric | Requirement | Measurement |
|--------|-------------|-------------|
| LINE Response Time | < 2 seconds | Bot reply to user commands |
| Dashboard Load Time | < 3 seconds | Initial page load with 50 complaints |
| AI Processing Time | < 30 seconds | Gemini analysis per complaint |
| Database Query Time | < 1 second | Most dashboard queries |
| Concurrent Users | 100 simultaneous | HR dashboard users |

### 2.2 Scalability Requirements

#### 2.2.1 Current Capacity (Phase 1)
- **Employees**: 500-1,000 LINE users
- **Complaints**: 50-200 per month
- **HR Users**: 5-20 concurrent dashboard users
- **Data Storage**: ~1GB per year
- **AI API Calls**: ~200-800 per month (Gemini)

#### 2.2.2 Growth Planning (Phase 2)
- **Employees**: Up to 5,000 LINE users
- **Complaints**: Up to 1,000 per month
- **HR Users**: Up to 50 concurrent
- **Data Storage**: ~10GB per year
- **AI API Calls**: Up to 5,000 per month

### 2.3 Reliability Requirements

| Component | Availability | Recovery Time | Backup Strategy |
|-----------|-------------|---------------|----------------|
| LINE Bot | 99.9% uptime | < 5 minutes | Auto-restart, health checks |
| HR Dashboard | 99.5% uptime | < 15 minutes | Graceful degradation |
| Database | 99.9% uptime | < 1 minute | MongoDB Atlas auto-failover |
| AI Processing | 95% success rate | Retry failed jobs | Fallback to default values |

### 2.4 Security Requirements

#### 2.4.1 Data Protection
- **Encryption at Rest**: MongoDB Atlas encryption enabled
- **Encryption in Transit**: HTTPS/TLS for all API communications
- **PII Protection**: LINE user IDs are pseudonymous
- **Access Control**: Database authentication required

#### 2.4.2 Network Security
- **LINE Webhook**: Signature verification for incoming webhooks
- **API Security**: Input validation with Zod schemas
- **CORS Policy**: Restricted to frontend domain only
- **Rate Limiting**: Prevent API abuse (future enhancement)

---

## 3. Technical Requirements

### 3.1 Technology Stack

#### 3.1.1 Frontend (HR Dashboard)
```typescript
Framework: Next.js 15 with App Router
Language: TypeScript 5.0+
Styling: TailwindCSS 3.0 + shadcn/ui components
State Management: React Query (via tRPC)
Charts: Recharts for analytics visualization
Build Tool: Turbo (monorepo management)
```

#### 3.1.2 Backend (API + LINE Bot + AI)
```typescript
Runtime: Node.js 18+ LTS
Framework: Express.js 4.0
API Layer: tRPC 10.0 (type-safe APIs)
LINE SDK: @line/bot-sdk latest
AI Service: Google Generative AI SDK
Validation: Zod schema validation
Database: Mongoose ODM for MongoDB
```

#### 3.1.3 Database & Infrastructure
```yaml
Database: MongoDB Atlas (managed)
AI Service: Google Gemini 1.5 Flash
Deployment: Railway (containerized)
CI/CD: GitHub Actions
Monitoring: Railway built-in + custom logging
```

### 3.2 System Architecture

#### 3.2.1 High-Level Architecture
```
LINE OA ↔ Express.js Backend ↔ MongoDB Atlas
            ↕                    ↕
       tRPC APIs            AI Analytics
            ↕
    Next.js Dashboard
```

#### 3.2.2 Data Flow
1. **Complaint Submission**: LINE → Backend → MongoDB
2. **AI Processing**: MongoDB → Gemini API → MongoDB Analytics
3. **Dashboard**: Next.js → tRPC → MongoDB → Dashboard

### 3.3 Database Design

#### 3.3.1 Collections Structure
```javascript
complaint_sessions: {
  complaint_id: "CMP-YYYY-MM-DD-NNNN",
  user_id: "LINE-USER-ID",
  chat_logs: [embedded array],
  status: "open|submitted"
}

complaint_analytics: {
  complaint_id: "reference",
  sentiment: "positive|negative|neutral",
  confidence_score: 0.85,
  tags_keywords: ["management", "workload"]
}
```

#### 3.3.2 Indexing Strategy
- Primary queries optimized with compound indexes
- TTL indexes for automatic data cleanup
- Text indexes for search functionality (future)

---

## 4. Integration Requirements

### 4.1 LINE Official Account

#### 4.1.1 Setup Requirements
- **LINE Business Account**: Verified business account required
- **Webhook URL**: `https://your-domain.railway.app/webhook`
- **Channel Access Token**: For sending replies to users
- **Channel Secret**: For webhook signature verification

#### 4.1.2 Message Types Supported
- **Text Messages**: Primary complaint content
- **Commands**: `/complain`, `/submit`
- **Rich Messages**: Future enhancement for guided forms
- **File Attachments**: Future enhancement for evidence

### 4.2 Google Gemini Integration

#### 4.2.1 API Requirements
- **Service**: Google AI Studio / Vertex AI
- **Model**: Gemini 1.5 Flash (cost-optimized)
- **API Key**: Required for authentication
- **Rate Limits**: Respect Google's API limits
- **Fallback**: Graceful degradation if AI service unavailable

#### 4.2.2 Prompt Engineering
```
Input: Employee complaint text (Thai/English)
Output: JSON with sentiment + keywords
Processing: Custom prompt for workplace context
Quality: Confidence scoring for reliability
```

---

## 5. Deployment Requirements

### 5.1 Environment Configuration

#### 5.1.1 Development Environment
```bash
Node.js: 18+ LTS
MongoDB: Local instance or Atlas free tier
LINE: Webhook tunneling (ngrok/localtunnel)
Gemini: Development API key
```

#### 5.1.2 Production Environment
```bash
Platform: Railway (recommended)
Domain: Custom domain with HTTPS
Database: MongoDB Atlas (shared/dedicated cluster)
Environment Variables: Secure secret management
Monitoring: Application and infrastructure monitoring
```

### 5.2 CI/CD Pipeline

#### 5.2.1 Automated Deployment
```yaml
Trigger: Git push to main branch
Build: Turbo build all packages
Test: Unit tests + integration tests
Deploy: Railway automatic deployment
Rollback: One-click rollback capability
```

#### 5.2.2 Quality Gates
- **TypeScript**: Zero type errors
- **Tests**: All tests passing
- **Linting**: ESLint + Prettier compliance
- **Build**: Successful production build

---

## 6. Operational Requirements

### 6.1 Monitoring & Logging

#### 6.1.1 Application Monitoring
- **API Response Times**: Track tRPC endpoint performance
- **Error Rates**: Monitor failed requests and exceptions
- **AI Processing**: Track Gemini API success/failure rates
- **User Activity**: LINE bot interaction metrics

#### 6.1.2 Infrastructure Monitoring
- **Database Performance**: MongoDB query performance
- **Server Resources**: CPU, memory, network usage
- **External APIs**: LINE and Gemini service availability
- **Storage Growth**: Database size and growth trends

### 6.2 Backup & Recovery

#### 6.2.1 Data Backup Strategy
```yaml
Database: MongoDB Atlas automatic backups (daily)
Application Data: Configuration and environment variables
Code: Git repository (GitHub/GitLab)
Dependencies: Package lock files committed
```

#### 6.2.2 Disaster Recovery
- **RTO (Recovery Time Objective)**: 4 hours
- **RPO (Recovery Point Objective)**: 24 hours
- **Backup Testing**: Monthly restore verification
- **Documentation**: Step-by-step recovery procedures

### 6.3 Maintenance Requirements

#### 6.3.1 Regular Maintenance
- **Security Updates**: Monthly dependency updates
- **Database Cleanup**: Quarterly old data archival
- **Performance Review**: Monthly query optimization
- **AI Model Updates**: Quarterly evaluation of new models

#### 6.3.2 Support & Documentation
- **User Manual**: HR dashboard usage guide
- **API Documentation**: Developer reference
- **Troubleshooting**: Common issues and solutions
- **Change Log**: Version history and updates

---

## 7. Compliance & Legal Requirements

### 7.1 Data Privacy

#### 7.1.1 GDPR Compliance (if applicable)
- **Data Collection**: Minimal necessary data only
- **Consent**: Clear notification of data processing
- **Right to Access**: Users can request their data
- **Right to Deletion**: Ability to remove user data
- **Data Portability**: Export user complaint history

#### 7.1.2 Local Privacy Laws
- **Data Residency**: Store data in compliant regions
- **Employee Rights**: Clear complaint process documentation
- **Retention Policy**: Defined data retention periods
- **Access Controls**: Limit HR access to authorized personnel

### 7.2 Workplace Compliance

#### 7.2.1 HR Policies
- **Non-Retaliation**: Clear policy against complaint retaliation
- **Confidentiality**: Secure handling of sensitive complaints
- **Response Times**: Defined SLAs for complaint resolution
- **Audit Trail**: Complete record of complaint handling

#### 7.2.2 Record Keeping
- **Legal Requirements**: 7-year complaint record retention
- **Audit Support**: Easy data retrieval for legal/audit needs
- **Anonymization**: Option to anonymize old complaints
- **Export Capabilities**: Legal-compliant data export formats

---

## 8. Success Criteria

### 8.1 User Adoption Metrics

| Metric | Target | Measurement Period |
|--------|--------|-------------------|
| Employee Registration | 80% of eligible employees | 3 months post-launch |
| Complaint Submission Rate | 90% via LINE OA | 6 months post-launch |
| HR User Adoption | 100% of HR team | 1 month post-launch |
| System Satisfaction | 4/5 average rating | Ongoing surveys |

### 8.2 Performance Metrics

| Metric | Target | Acceptable Range |
|--------|--------|-----------------|
| LINE Bot Response Time | < 2 seconds | < 5 seconds |
| Dashboard Load Time | < 3 seconds | < 8 seconds |
| AI Processing Accuracy | > 85% relevant | > 75% acceptable |
| System Uptime | 99.9% | > 99% |

### 8.3 Business Impact

#### 8.3.1 Efficiency Gains
- **Complaint Processing Time**: 50% reduction from previous system
- **HR Administrative Work**: 30% reduction in manual processing
- **Response Quality**: Improved consistency with AI insights
- **Compliance Reporting**: Automated report generation

#### 8.3.2 Employee Experience
- **Accessibility**: 24/7 complaint submission via familiar interface
- **Anonymity**: Pseudonymous LINE IDs protect privacy
- **Feedback**: Clear complaint ID and status tracking
- **Responsiveness**: Faster acknowledgment and processing

---

## 9. Future Enhancements

### 9.1 Phase 2 Features (6-12 months)
- **Multi-language Support**: Thai language UI and AI analysis
- **Advanced Analytics**: Trend analysis and predictive insights
- **Mobile App**: Native mobile app for HR teams
- **Integration**: Connect with existing HR systems

### 9.2 Phase 3 Features (12-24 months)
- **Anonymous Mode**: Option for completely anonymous complaints
- **AI Routing**: Automatic complaint categorization and routing
- **Real-time Notifications**: Instant alerts for urgent complaints
- **Advanced Reporting**: Custom dashboards and scheduled reports

### 9.3 Scalability Roadmap
- **Multi-tenant**: Support for multiple organizations
- **API Gateway**: External integrations for partners
- **Microservices**: Break monolith into specialized services
- **Global Deployment**: Multi-region deployment for performance

---

This system requirements document serves as the foundation for developing a robust, scalable HR complaint management system that leverages modern technologies to improve workplace communication and HR efficiency.