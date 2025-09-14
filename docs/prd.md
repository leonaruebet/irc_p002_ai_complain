## Product Requirments Documentation

---

## 1. Objective  
Enable employees to log complaints through **LINE OA chat commands** (`/complain`, `/submit`) with conversations stored in MongoDB. HR can access a **secure dashboard** (Next.js app) to view, track, and report complaints.

---

## 2. Goals & Non-Goals  

### Goals  
- Provide a **chat-first complaint submission flow** integrated with LINE OA.  
- Store all conversations with **user_id + timestamp** in MongoDB.  
- Ensure HR has **enterprise-grade secure access** to the dashboard.  
- HR can:  
  - View complaints by status.  
  - Review chat logs.  
  - Generate summary reports.  

### Non-Goals  
- No access for executives or employees beyond HR.  
- No anonymous submissions (LINE user_id always logged).  
- No AI classification in MVP.  

---

## 3. User Stories  

### Employee (via LINE OA)  
- As an employee, I can type `/complain` to start a session.  
- As an employee, I can type freely with the bot.  
- As an employee, I can type `/submit` to finalize my complaint.  
- As an employee, I receive acknowledgment & updates.  

### HR (via Dashboard)  
- As HR, I can view a list of complaints.  
- As HR, I can filter complaints (by date, status, employee).  
- As HR, I can view complaint chat logs.  
- As HR, I can update status and add resolution notes.  
- As HR, I can generate reports.  

---

## 4. Functional Requirements  

### LINE OA Bot  
- Commands supported: `/complain`, `/submit`.  
- Capture employee messages (with `user_id`, `timestamp`).  
- Send acknowledgment with complaint ID.  

### Backend (API Layer)  
- **Session management** between `/complain` and `/submit`.  
- **MongoDB Collections**:  
  - `complaint_sessions` (session_id, user_id, status).  
  - `complaint_logs` (messages).  
- APIs:  
  - `POST /complaints/start`  
  - `POST /complaints/log`  
  - `POST /complaints/submit`  
  - `GET /complaints`  
  - `GET /complaints/:id`  
  - `PATCH /complaints/:id`  

### HR Dashboard (Next.js)  
- Features:  
  - Complaint list table.  
  - Complaint detail page with chat log.  
  - Status & resolution notes.  
  - Reports (volume, resolution time, trends).  

---

## 5. Non-Functional Requirements  

- **Scalability** 
  - MongoDB Atlas with auto-scaling. 
  - Backend supports 10k+ complaints/year.  

- **Reliability**  
  - 99.9% uptime target.  
  - Daily backups for MongoDB.  

---

## 6. Tech Stack  

- **Frontend (HR Dashboard):**  
  - Next.js 15  
  - TailwindCSS + shadcn/ui  


- **Backend (API):**  
  - Node.js + Express 
  - LINE Messaging API SDK  
  - REST APIs with JWT tokens issued by IdP  

- **Database:**  
  - MongoDB Atlas (Collections: `complaint_sessions`, `complaint_logs`, `hr_actions`)  

- **Authentication:**  
  - **SSO with IdP (Okta, Azure AD, Google Workspace)**  
  - **OIDC (OpenID Connect) + OAuth2.0**  
  - MFA enforcement  
  - Role-based access control  

- **Deployment:**  
  - Railway 

---

## 7. Success Metrics  

- **Adoption**: >80% complaints submitted via LINE OA.  
- **Efficiency**: 30% faster HR resolution time.  
- **System Reliability**: >99.9% uptime.  

---

## 8. Future Enhancements  

- AI classification for severity tagging.  
- Anonymous mode (mask user_id).  
- Slack/Email notifications for new complaints.  
- Export reports (PDF/CSV).  

---
