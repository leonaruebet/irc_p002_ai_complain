# API Documentation

## HR Complaint Management System - tRPC API Reference

This document provides comprehensive API documentation for the complaint management system with AI analytics integration.

**API Type**: tRPC with Express.js backend (matches planning.md)
**Base URL**: `http://localhost:3001/trpc` (development)
**Authentication**: None (internal system)
**Data Format**: JSON

---

## API Router Structure (matches planning.md)

```typescript
export const appRouter = router({
  complaint: complaintRouter,
  report: reportRouter, // includes analytics sub-router
});
```

### Response Format
All tRPC APIs return TypeScript-safe responses with automatic serialization:

```typescript
// Success Response
{
  result: {
    data: T  // Actual response data
  }
}

// Error Response
{
  error: {
    message: string,
    code: "INTERNAL_SERVER_ERROR" | "BAD_REQUEST" | "NOT_FOUND",
    data: {
      code: string,
      httpStatus: number
    }
  }
}
```

---

## Complaint Management APIs

### 1. Get All Complaints

**Procedure**: `complaint.getAll`
**Type**: Query
**Purpose**: Retrieve paginated list of complaints with filtering

#### Input Schema
```typescript
{
  page?: number;        // Default: 1
  limit?: number;       // Default: 50, Max: 100
  status?: string;      // "open" | "submitted"
  department?: string;  // Filter by department
}
```

#### Response Schema
```typescript
{
  complaints: Array<{
    _id: string;
    complaint_id: string;      // "CMP-2025-09-14-0001"
    user_id: string;           // LINE userId
    department?: string;
    status: "open" | "submitted";
    start_time: Date;
    end_time?: Date;
  }>;
  pagination: {
    page: number;
    limit: number;
    total: number;
    pages: number;
  };
}
```

#### Example Usage
```typescript
// Frontend (React)
const { data, isLoading, error } = trpc.complaint.getAll.useQuery({
  page: 1,
  limit: 25,
  status: "submitted",
  department: "Operations"
});

// Response
{
  complaints: [
    {
      _id: "sess_20250914_ABC123",
      complaint_id: "CMP-2025-09-14-0001",
      user_id: "Ucb71e2...1234",
      department: "Operations",
      status: "submitted",
      start_time: "2025-09-14T10:02:34.000Z",
      end_time: "2025-09-14T10:15:55.000Z"
    }
  ],
  pagination: {
    page: 1,
    limit: 25,
    total: 145,
    pages: 6
  }
}
```

---

### 2. Get Complaint by ID

**Procedure**: `complaint.getById`
**Type**: Query
**Purpose**: Retrieve complete complaint details including chat logs

#### Input Schema
```typescript
{
  id: string;  // complaint_id (e.g., "CMP-2025-09-14-0001")
}
```

#### Response Schema
```typescript
{
  _id: string;
  complaint_id: string;
  user_id: string;
  status: "open" | "submitted";
  start_time: Date;
  end_time?: Date;
  department?: string;
  chat_logs: Array<{
    timestamp: Date;
    direction: "user" | "bot";
    message_type: "text" | "image" | "file" | "command";
    message: string;
  }>;
  created_at: Date;
  updated_at: Date;
}
```

#### Example Usage
```typescript
// Frontend (React)
const { data: complaint, isLoading } = trpc.complaint.getById.useQuery({
  id: "CMP-2025-09-14-0001"
});

// Response
{
  _id: "sess_20250914_ABC123",
  complaint_id: "CMP-2025-09-14-0001",
  user_id: "Ucb71e2...1234",
  status: "submitted",
  start_time: "2025-09-14T10:02:34.000Z",
  end_time: "2025-09-14T10:15:55.000Z",
  department: "Operations",
  chat_logs: [
    {
      timestamp: "2025-09-14T10:02:35.000Z",
      direction: "user",
      message_type: "command",
      message: "/complain"
    },
    {
      timestamp: "2025-09-14T10:06:12.000Z",
      direction: "user",
      message_type: "text",
      message: "My manager repeatedly assigns unpaid overtime work."
    }
  ],
  created_at: "2025-09-14T10:02:34.000Z",
  updated_at: "2025-09-14T10:15:55.000Z"
}
```

---

## Basic Reporting APIs

### 3. Report Overview

**Procedure**: `report.overview`
**Type**: Query
**Purpose**: Get basic complaint statistics

#### Input Schema
```typescript
// No input parameters
{}
```

#### Response Schema
```typescript
{
  totalComplaints: number;
  openComplaints: number;
  submittedComplaints: number;
}
```

#### Example Usage
```typescript
// Frontend (React)
const { data: overview } = trpc.report.overview.useQuery();

// Response
{
  totalComplaints: 245,
  openComplaints: 12,
  submittedComplaints: 233
}
```

---

## AI Analytics APIs

### 4. Analytics Overview

**Procedure**: `report.analytics.overview`
**Type**: Query
**Purpose**: Get AI-powered complaint insights overview

#### Input Schema
```typescript
// No input parameters
{}
```

#### Response Schema
```typescript
{
  totalComplaints: number;
  sentimentStats: Array<{
    _id: "positive" | "negative" | "neutral";
    count: number;
  }>;
  topKeywords: Array<{
    _id: string;      // keyword
    count: number;    // frequency
  }>;
}
```

#### Example Usage
```typescript
// Frontend (React)
const { data: analytics } = trpc.report.analytics.overview.useQuery();

// Response
{
  totalComplaints: 233,
  sentimentStats: [
    { _id: "negative", count: 145 },
    { _id: "neutral", count: 67 },
    { _id: "positive", count: 21 }
  ],
  topKeywords: [
    { _id: "management", count: 67 },
    { _id: "workload", count: 45 },
    { _id: "overtime", count: 34 },
    { _id: "communication", count: 28 }
  ]
}
```

---

### 5. Word Cloud Data

**Procedure**: `report.analytics.wordCloud`
**Type**: Query
**Purpose**: Get formatted data for word cloud visualization

#### Input Schema
```typescript
// No input parameters
{}
```

#### Response Schema
```typescript
Array<{
  text: string;   // keyword
  value: number;  // frequency count
}>
```

#### Example Usage
```typescript
// Frontend (React)
const { data: wordCloudData } = trpc.report.analytics.wordCloud.useQuery();

// Response
[
  { text: "management", value: 67 },
  { text: "workload", value: 45 },
  { text: "overtime", value: 34 },
  { text: "communication", value: 28 },
  { text: "harassment", value: 23 },
  { text: "salary", value: 19 }
]
```

---

### 6. Sentiment Trends

**Procedure**: `report.analytics.sentimentTrends`
**Type**: Query
**Purpose**: Get sentiment analysis over time

#### Input Schema
```typescript
// No input parameters
{}
```

#### Response Schema
```typescript
Array<{
  _id: {
    sentiment: "positive" | "negative" | "neutral";
    month: number;    // 1-12
    year: number;     // e.g., 2025
  };
  count: number;
}>
```

#### Example Usage
```typescript
// Frontend (React)
const { data: trends } = trpc.report.analytics.sentimentTrends.useQuery();

// Response
[
  {
    _id: { sentiment: "negative", month: 8, year: 2025 },
    count: 23
  },
  {
    _id: { sentiment: "neutral", month: 8, year: 2025 },
    count: 12
  },
  {
    _id: { sentiment: "negative", month: 9, year: 2025 },
    count: 34
  }
]
```

---

## Error Handling

### Common Error Codes

#### INTERNAL_SERVER_ERROR
```typescript
{
  error: {
    message: "Failed to analyze complaint with AI",
    code: "INTERNAL_SERVER_ERROR",
    data: {
      code: "INTERNAL_SERVER_ERROR",
      httpStatus: 500
    }
  }
}
```

#### NOT_FOUND
```typescript
{
  error: {
    message: "Complaint not found",
    code: "NOT_FOUND",
    data: {
      code: "NOT_FOUND",
      httpStatus: 404
    }
  }
}
```

#### BAD_REQUEST
```typescript
{
  error: {
    message: "Invalid input: page must be a positive number",
    code: "BAD_REQUEST",
    data: {
      code: "BAD_REQUEST",
      httpStatus: 400
    }
  }
}
```

### Frontend Error Handling
```typescript
const { data, error, isLoading } = trpc.complaint.getAll.useQuery({
  page: 1
});

if (error) {
  console.error('API Error:', error.message);
  // Handle specific error codes
  switch (error.data?.code) {
    case 'NOT_FOUND':
      // Show "No complaints found" message
      break;
    case 'INTERNAL_SERVER_ERROR':
      // Show "Please try again later" message
      break;
    default:
      // Generic error handling
  }
}
```

---

## Type Safety & Validation

### Input Validation (Zod Schemas)

#### Complaint Query Validation
```typescript
// Backend validation schema
const complaintQuerySchema = z.object({
  page: z.number().min(1).default(1),
  limit: z.number().min(1).max(100).default(50),
  status: z.enum(['open', 'submitted']).optional(),
  department: z.string().optional()
});
```

#### Frontend TypeScript Integration
```typescript
// Types are automatically inferred from backend
type ComplaintQueryInput = {
  page?: number;
  limit?: number;
  status?: "open" | "submitted";
  department?: string;
}

// Type-safe API calls
const queryInput: ComplaintQueryInput = {
  page: 1,
  status: "submitted"  // TypeScript validates enum values
};
```

---

## Rate Limiting & Performance

### Query Optimization

#### Efficient Pagination
```typescript
// Good: Use limit and skip for large datasets
trpc.complaint.getAll.useQuery({
  page: 5,
  limit: 50  // Maximum 100 to prevent overload
});

// Bad: Loading all data
trpc.complaint.getAll.useQuery({
  limit: 10000  // Will be capped at 100
});
```

#### Caching Strategy
```typescript
// Frontend caching with React Query
const { data } = trpc.complaint.getAll.useQuery(
  { page: 1, status: "submitted" },
  {
    staleTime: 5 * 60 * 1000,      // 5 minutes
    cacheTime: 10 * 60 * 1000,     // 10 minutes
    refetchOnWindowFocus: false,
  }
);
```

### Performance Recommendations

1. **Use Pagination**: Always specify page and limit for list queries
2. **Filter Early**: Apply status/department filters to reduce data transfer
3. **Cache Results**: Use React Query caching for repeated requests
4. **Avoid Polling**: Use real-time updates only when necessary

---

## Development & Testing

### Development Server
```bash
# Start backend with tRPC
cd apps/backend
npm run dev  # Runs on http://localhost:3001/trpc

# Start frontend
cd apps/web
npm run dev  # Runs on http://localhost:3000
```

### API Testing

#### Using tRPC Panel (Development)
```bash
# Install tRPC panel
npm install --save-dev @trpc/panel

# Add to backend server
app.use('/panel', trpcPanel(appRouter, { url: '/trpc' }));

# Access at: http://localhost:3001/panel
```

#### Manual Testing with curl
```bash
# Get all complaints
curl -X GET "http://localhost:3001/trpc/complaint.getAll?input=%7B%22page%22:1%7D"

# Get specific complaint
curl -X GET "http://localhost:3001/trpc/complaint.getById?input=%7B%22id%22:%22CMP-2025-09-14-0001%22%7D"
```

#### Frontend Testing
```typescript
// Test with React Query DevTools
import { ReactQueryDevtools } from '@tanstack/react-query-devtools';

function App() {
  return (
    <>
      <YourApp />
      <ReactQueryDevtools initialIsOpen={false} />
    </>
  );
}
```

---

## Production Considerations

### Monitoring & Logging
- Log all API requests with response times
- Monitor error rates by endpoint
- Track Gemini API usage and costs
- Set up alerts for high error rates

### Security
- Validate all inputs with Zod schemas
- Implement rate limiting per IP/user
- Add request logging for audit trail
- Consider API authentication for production

### Scaling
- Enable MongoDB connection pooling
- Add Redis caching layer for frequent queries
- Implement API response caching
- Consider CDN for static responses

---

## Migration & Versioning

### API Versioning Strategy
- Use semantic versioning for tRPC router
- Maintain backward compatibility when possible
- Deprecate old endpoints gradually
- Document breaking changes clearly

### Schema Evolution
- Add new fields as optional
- Use database migrations for required changes
- Version API responses for major changes
- Implement feature flags for gradual rollouts