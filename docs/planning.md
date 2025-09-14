# Implementation Plan - HR Dashboard with LINE OA Integration

## System Architecture Overview

```
                       ┌─────────────────┐
                       │  HR Dashboard   │
                       │  (Next.js 15)   │
                       │  + AI Analytics │
                       └─────────────────┘
                                |
                                | tRPC
                                v
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   LINE OA Bot   │<-->│  Backend API     │<-->│   AI Service    │
│                 │    │  (Express.js +   │    │  (Gemini API +  │
│                 │    │   tRPC Server)   │    │  NLP Pipeline)  │
└─────────────────┘    └──────────────────┘    └─────────────────┘
                                |                        |
                                v                        v
                       ┌──────────────────────────────────────┐
                       │         MongoDB Atlas                │
                       │  ┌─────────────┐ ┌─────────────────┐ │
                       │  │ Complaints  │ │  AI Analytics   │ │
                       │  │    Data     │ │     Data        │ │
                       │  └─────────────┘ └─────────────────┘ │
                       └──────────────────────────────────────┘
```

## Project Structure

```
irc_p002_ai_complain/
├── apps/
│   ├── web/                    # HR Dashboard (Next.js)
│   │   ├── app/
│   │   │   ├── dashboard/      # Dashboard pages
│   │   │   │   ├── complaints/ # Complaint management UI
│   │   │   │   ├── reports/    # Reports and analytics
│   │   │   │   └── settings/   # Settings and configuration
│   │   │   ├── components/     # Reusable UI components
│   │   │   │   ├── ui/         # Base UI components (shadcn)
│   │   │   │   ├── complaint/  # Complaint-specific components
│   │   │   │   ├── charts/     # Data visualization components
│   │   │   │   └── layout/     # Layout components
│   │   │   ├── lib/           # Utilities and configurations
│   │   │   │   ├── trpc.ts    # tRPC client configuration
│   │   │   │   ├── utils.ts   # General utilities
│   │   │   │   └── constants.ts # Application constants
│   │   │   ├── types/         # TypeScript type definitions
│   │   │   └── hooks/         # Custom React hooks
│   │   ├── public/            # Static assets
│   │   ├── styles/            # Global styles
│   │   └── server/            # tRPC API routes
│   │       └── routers/       # tRPC router definitions
│   └── backend/               # Backend API + LINE Bot + AI Service
│       ├── src/
│       │   ├── line/          # LINE OA Bot handlers
│       │   │   ├── controllers/ # LINE webhook controllers
│       │   │   ├── services/   # LINE bot business logic
│       │   │   └── config/     # LINE configuration
│       │   ├── ai/            # AI Analysis Service
│       │   │   ├── services/   # AI processing services
│       │   │   ├── processors/ # NLP processors
│       │   │   ├── models/     # AI model configurations
│       │   │   └── utils/      # AI utilities
│       │   ├── trpc/          # tRPC server setup
│       │   │   ├── routers/    # tRPC routers
│       │   │   ├── procedures/ # tRPC procedures
│       │   │   └── context.ts  # tRPC context
│       │   ├── services/      # Business logic services
│       │   ├── middleware/    # Express middleware
│       │   ├── utils/         # Backend utilities
│       │   └── types/         # Backend type definitions
│       ├── package.json
│       └── server.js
├── packages/
│   ├── database/              # Shared database schemas and utilities
│   │   ├── models/            # Mongoose models
│   │   ├── schemas/           # Database schemas
│   │   └── migrations/        # Database migrations
│   ├── shared/                # Shared utilities and types
│   │   ├── types/             # Shared TypeScript types
│   │   ├── constants/         # Shared constants
│   │   └── utils/             # Shared utility functions
│   └── ui/                    # Shared UI components
│       ├── components/        # Reusable components
│       └── styles/            # Shared styles
├── docs/                      # Documentation
├── package.json               # Root package.json (monorepo)
└── turbo.json                 # Turbo configuration
```

## Implementation Phases

### Phase 1: Foundation Setup (Week 1)

#### 1.1 Project Infrastructure
- **Monorepo Setup**: Configure Turbo.js for monorepo management
- **Package Configuration**: Setup shared packages for types, database, and UI
- **Environment Configuration**: Setup development and production environments
- **Database Setup**: Configure MongoDB Atlas connection and initial collections

#### 1.2 Database Layer
```javascript
// packages/database/models/complaint-session.js
import mongoose from 'mongoose';

const ChatLogSchema = new mongoose.Schema({
  timestamp: { type: Date, required: true },
  direction: { type: String, enum: ['user', 'bot'], required: true },
  message_type: { type: String, enum: ['text', 'image', 'file', 'command'], required: true },
  message: { type: String, required: true }
}, { _id: false });

const ComplaintSessionSchema = new mongoose.Schema({
  _id: { type: String, required: true },
  complaint_id: { type: String, required: true, unique: true },
  user_id: { type: String, required: true, index: true },
  status: { type: String, enum: ['open', 'submitted'], default: 'open', index: true },
  start_time: { type: Date, required: true },
  end_time: { type: Date },
  department: { type: String },
  chat_logs: { type: [ChatLogSchema], required: true },
  created_at: { type: Date, default: Date.now },
  updated_at: { type: Date, default: Date.now }
});

// Indexes
ComplaintSessionSchema.index({ status: 1, start_time: -1 });
ComplaintSessionSchema.index({ user_id: 1, start_time: -1 });
ComplaintSessionSchema.index({ complaint_id: 1 }, { unique: true });

export default mongoose.model('ComplaintSession', ComplaintSessionSchema);
```

#### 1.3 Simple AI Analytics Database Schema
```javascript
// packages/database/models/ai-analytics.js
import mongoose from 'mongoose';

const ComplaintAnalyticsSchema = new mongoose.Schema({
  complaint_id: { type: String, required: true, unique: true, index: true },
  user_id: { type: String, required: true, index: true },

  // Simple AI Analysis Results
  sentiment: { type: String, enum: ['positive', 'negative', 'neutral'], required: true },
  confidence_score: { type: Number, min: 0, max: 1, required: true },
  tags_keywords: [{ type: String }], // Main topics/keywords

  // Processing Metadata
  ai_model_version: { type: String, required: true },
  created_at: { type: Date, default: Date.now }
});

// Basic indexes
ComplaintAnalyticsSchema.index({ sentiment: 1, created_at: -1 });
ComplaintAnalyticsSchema.index({ tags_keywords: 1 });

export default mongoose.model('ComplaintAnalytics', ComplaintAnalyticsSchema);
```

#### 1.4 Simplified Shared Types
```typescript
// packages/shared/types/complaint.ts
export interface ChatLog {
  timestamp: Date;
  direction: 'user' | 'bot';
  message_type: 'text' | 'image' | 'file' | 'command';
  message: string;
}

export interface ComplaintSession {
  _id: string;
  complaint_id: string;
  user_id: string;
  status: 'open' | 'submitted';
  start_time: Date;
  end_time?: Date;
  department?: string;
  chat_logs: ChatLog[];
  created_at: Date;
  updated_at: Date;
}

export interface Employee {
  _id: string;
  display_name: string;
  department?: string;
  active: boolean;
  created_at: Date;
  updated_at: Date;
}

// Simple AI Analytics Types
export interface ComplaintAnalytics {
  complaint_id: string;
  user_id: string;
  sentiment: 'positive' | 'negative' | 'neutral';
  confidence_score: number;
  tags_keywords: string[];
  ai_model_version: string;
  created_at: Date;
}
```

### Phase 2: Backend with tRPC and LINE Bot (Week 2)

#### 2.1 tRPC Server Setup
```typescript
// apps/backend/src/trpc/context.ts
import { CreateNextContextOptions } from '@trpc/server/adapters/next';
import { connectDB } from '@packages/database';

export async function createTRPCContext(opts: CreateNextContextOptions) {
  await connectDB();

  return {
    req: opts.req,
    res: opts.res,
  };
}

export type Context = Awaited<ReturnType<typeof createTRPCContext>>;
```

#### 2.2 tRPC Router Configuration
```typescript
// apps/backend/src/trpc/router.ts
import { router } from './trpc';
import { complaintRouter } from './routers/complaint';
import { reportRouter } from './routers/report';

export const appRouter = router({
  complaint: complaintRouter,
  report: reportRouter,
});

export type AppRouter = typeof appRouter;
```

#### 2.3 Complaint tRPC Router
```typescript
// apps/backend/src/trpc/routers/complaint.ts
import { z } from 'zod';
import { router, publicProcedure } from '../trpc';
import { ComplaintSession } from '@packages/database/models/complaint-session';

export const complaintRouter = router({
  getAll: publicProcedure
    .input(z.object({
      page: z.number().default(1),
      limit: z.number().default(50),
      status: z.string().optional(),
      department: z.string().optional(),
    }))
    .query(async ({ input }) => {
      const { page, limit, status, department } = input;

      const filter: any = {};
      if (status) filter.status = status;
      if (department) filter.department = department;

      const skip = (page - 1) * limit;

      const [complaints, total] = await Promise.all([
        ComplaintSession.find(filter)
          .select('complaint_id user_id department status start_time end_time')
          .sort({ start_time: -1 })
          .skip(skip)
          .limit(limit)
          .lean(),
        ComplaintSession.countDocuments(filter)
      ]);

      return {
        complaints,
        pagination: {
          page,
          limit,
          total,
          pages: Math.ceil(total / limit)
        }
      };
    }),

  getById: publicProcedure
    .input(z.object({ id: z.string() }))
    .query(async ({ input }) => {
      const complaint = await ComplaintSession.findOne({
        complaint_id: input.id
      }).lean();

      if (!complaint) {
        throw new Error('Complaint not found');
      }

      return complaint;
    }),
});
```

#### 2.4 LINE Bot Integration
```javascript
// apps/backend/src/line/controllers/webhook.js
import { Client } from '@line/bot-sdk';
import { ComplaintService } from '../services/complaint.js';
import { config } from '../config/line.js';

const client = new Client(config);
const complaintService = new ComplaintService();

export async function handleWebhook(req, res) {
  const events = req.body.events;

  for (const event of events) {
    if (event.type === 'message' && event.message.type === 'text') {
      await handleTextMessage(event);
    }
  }

  res.status(200).end();
}

async function handleTextMessage(event) {
  const { replyToken, source, message } = event;
  const userId = source.userId;
  const text = message.text;

  try {
    if (text === '/complain') {
      await complaintService.startSession(userId);
      await client.replyMessage(replyToken, {
        type: 'text',
        text: 'Please describe your complaint. Type /submit when done.'
      });
    } else if (text === '/submit') {
      const complaintId = await complaintService.submitComplaint(userId);
      await client.replyMessage(replyToken, {
        type: 'text',
        text: `Your complaint has been submitted. ID: ${complaintId}`
      });
    } else {
      await complaintService.logMessage(userId, text);
      await client.replyMessage(replyToken, {
        type: 'text',
        text: 'Message received. Continue describing your complaint or type /submit to finish.'
      });
    }
  } catch (error) {
    console.error('Error handling message:', error);
  }
}
```

#### 2.5 Backend Server Setup
```typescript
// apps/backend/src/server.js
import express from 'express';
import cors from 'cors';
import { createExpressMiddleware } from '@trpc/server/adapters/express';
import { middleware } from '@line/bot-sdk';
import { appRouter } from './trpc/router.js';
import { createTRPCContext } from './trpc/context.js';
import { handleWebhook } from './line/controllers/webhook.js';
import { config as lineConfig } from './line/config/line.js';

const app = express();
const PORT = process.env.PORT || 3001;

// CORS setup
app.use(cors({
  origin: process.env.FRONTEND_URL || 'http://localhost:3000',
  credentials: true,
}));

// LINE webhook
app.use('/webhook', middleware(lineConfig), handleWebhook);

// tRPC middleware
app.use('/trpc', createExpressMiddleware({
  router: appRouter,
  createContext: createTRPCContext,
}));

app.listen(PORT, () => {
  console.log(`Backend server running on port ${PORT}`);
});
```

#### 2.6 Simple AI Analysis Service with Gemini
```typescript
// apps/backend/src/ai/services/gemini-analyzer.ts
import { GoogleGenerativeAI } from '@google/generative-ai';

const genAI = new GoogleGenerativeAI(process.env.GEMINI_API_KEY!);

export class GeminiAnalyzer {
  private model = genAI.getGenerativeModel({ model: 'gemini-1.5-flash' });

  async analyzeComplaint(complaintText: string) {
    const prompt = `
Analyze this employee complaint and return only a JSON response:

{
  "sentiment": "positive|negative|neutral",
  "confidence_score": 0.85,
  "tags_keywords": ["keyword1", "keyword2", "keyword3"]
}

Rules:
1. sentiment: overall emotional tone of the complaint
2. confidence_score: how confident you are in the sentiment (0-1)
3. tags_keywords: 3-5 main topics/keywords (e.g., "management", "workload", "harassment", "salary", "environment")

Complaint text: "${complaintText}"

Return only valid JSON:`;

    try {
      const result = await this.model.generateContent(prompt);
      const response = await result.response;
      return JSON.parse(response.text());
    } catch (error) {
      console.error('Gemini analysis error:', error);
      return {
        sentiment: 'neutral',
        confidence_score: 0.5,
        tags_keywords: ['general']
      };
    }
  }
}
```

#### 2.7 Simple AI Processing Service
```typescript
// apps/backend/src/ai/services/complaint-processor.ts
import { ComplaintSession } from '@packages/database/models/complaint-session';
import { ComplaintAnalytics } from '@packages/database/models/ai-analytics';
import { GeminiAnalyzer } from './gemini-analyzer';

export class ComplaintProcessor {
  private analyzer = new GeminiAnalyzer();

  async processComplaint(complaintId: string): Promise<void> {
    try {
      const complaint = await ComplaintSession.findOne({ complaint_id: complaintId });
      if (!complaint) return;

      // Extract user messages
      const complaintText = complaint.chat_logs
        .filter(log => log.direction === 'user' && log.message_type === 'text')
        .map(log => log.message)
        .join(' ');

      if (!complaintText.trim()) return;

      // Analyze with Gemini
      const analysis = await this.analyzer.analyzeComplaint(complaintText);

      // Save to database
      await new ComplaintAnalytics({
        complaint_id: complaintId,
        user_id: complaint.user_id,
        sentiment: analysis.sentiment,
        confidence_score: analysis.confidence_score,
        tags_keywords: analysis.tags_keywords,
        ai_model_version: 'gemini-1.5-flash'
      }).save();

      console.log(`Processed complaint ${complaintId}: ${analysis.sentiment}`);
    } catch (error) {
      console.error(`Error processing ${complaintId}:`, error);
    }
  }
}
```

### Phase 3: Frontend tRPC Integration (Week 3)

#### 3.1 tRPC Client Setup
```typescript
// apps/web/lib/trpc.ts
import { createTRPCReact } from '@trpc/react-query';
import { httpBatchLink } from '@trpc/client';
import type { AppRouter } from '@backend/src/trpc/router';

export const trpc = createTRPCReact<AppRouter>();

export const trpcClient = trpc.createClient({
  links: [
    httpBatchLink({
      url: `${process.env.NEXT_PUBLIC_BACKEND_URL}/trpc`,
    }),
  ],
});
```

#### 3.2 tRPC Provider Setup
```typescript
// apps/web/app/providers.tsx
'use client';

import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { useState } from 'react';
import { trpc, trpcClient } from '@/lib/trpc';

export function Providers({ children }: { children: React.ReactNode }) {
  const [queryClient] = useState(() => new QueryClient());

  return (
    <trpc.Provider client={trpcClient} queryClient={queryClient}>
      <QueryClientProvider client={queryClient}>
        {children}
      </QueryClientProvider>
    </trpc.Provider>
  );
}
```

#### 3.3 Root Layout with Providers
```typescript
// apps/web/app/layout.tsx
import './globals.css';
import { Providers } from './providers';

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body>
        <Providers>
          {children}
        </Providers>
      </body>
    </html>
  );
}
```

### Phase 4: HR Dashboard Frontend (Week 4-5)

#### 4.1 Dashboard Layout
```typescript
// apps/web/app/dashboard/layout.tsx
import { Sidebar } from '@/components/layout/sidebar';
import { Header } from '@/components/layout/header';

export default function DashboardLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <div className="flex h-screen bg-gray-100">
      <Sidebar />
      <div className="flex-1 flex flex-col overflow-hidden">
        <Header />
        <main className="flex-1 overflow-y-auto p-6">
          {children}
        </main>
      </div>
    </div>
  );
}
```

#### 4.2 Complaints List Page
```typescript
// apps/web/app/dashboard/complaints/page.tsx
import { ComplaintsTable } from '@/components/complaint/complaints-table';
import { ComplaintsFilters } from '@/components/complaint/complaints-filters';
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card';

export default function ComplaintsPage() {
  return (
    <div className="space-y-6">
      <div className="flex items-center justify-between">
        <h1 className="text-3xl font-bold">Complaints Management</h1>
      </div>

      <Card>
        <CardHeader>
          <CardTitle>Filters</CardTitle>
        </CardHeader>
        <CardContent>
          <ComplaintsFilters />
        </CardContent>
      </Card>

      <Card>
        <CardHeader>
          <CardTitle>All Complaints</CardTitle>
        </CardHeader>
        <CardContent>
          <ComplaintsTable />
        </CardContent>
      </Card>
    </div>
  );
}
```

#### 4.3 Complaints Table Component with tRPC
```typescript
// apps/web/components/complaint/complaints-table.tsx
'use client';

import { useState } from 'react';
import { useSearchParams } from 'next/navigation';
import {
  Table,
  TableBody,
  TableCell,
  TableHead,
  TableHeader,
  TableRow,
} from '@/components/ui/table';
import { Badge } from '@/components/ui/badge';
import { Button } from '@/components/ui/button';
import { trpc } from '@/lib/trpc';
import Link from 'next/link';

export function ComplaintsTable() {
  const [page, setPage] = useState(1);
  const searchParams = useSearchParams();
  const status = searchParams.get('status') || undefined;
  const department = searchParams.get('department') || undefined;

  const { data, isLoading, error } = trpc.complaint.getAll.useQuery({
    page,
    limit: 50,
    status,
    department,
  });

  if (isLoading) {
    return <div className="flex justify-center p-8">Loading...</div>;
  }

  if (error) {
    return <div className="flex justify-center p-8 text-red-500">Error: {error.message}</div>;
  }

  return (
    <div className="space-y-4">
      <Table>
        <TableHeader>
          <TableRow>
            <TableHead>Complaint ID</TableHead>
            <TableHead>User ID</TableHead>
            <TableHead>Department</TableHead>
            <TableHead>Status</TableHead>
            <TableHead>Submitted</TableHead>
            <TableHead>Actions</TableHead>
          </TableRow>
        </TableHeader>
        <TableBody>
          {data?.complaints.map((complaint) => (
            <TableRow key={complaint.complaint_id}>
              <TableCell className="font-medium">
                {complaint.complaint_id}
              </TableCell>
              <TableCell>{complaint.user_id}</TableCell>
              <TableCell>{complaint.department || 'N/A'}</TableCell>
              <TableCell>
                <Badge variant={complaint.status === 'submitted' ? 'default' : 'secondary'}>
                  {complaint.status}
                </Badge>
              </TableCell>
              <TableCell>
                {new Date(complaint.start_time).toLocaleDateString()}
              </TableCell>
              <TableCell>
                <Link href={`/dashboard/complaints/${complaint.complaint_id}`}>
                  <Button variant="outline" size="sm">
                    View Details
                  </Button>
                </Link>
              </TableCell>
            </TableRow>
          ))}
        </TableBody>
      </Table>

      {/* Pagination */}
      <div className="flex justify-between items-center">
        <p className="text-sm text-gray-700">
          Showing {((page - 1) * 50) + 1} to {Math.min(page * 50, data?.pagination.total || 0)} of {data?.pagination.total} results
        </p>
        <div className="flex gap-2">
          <Button
            variant="outline"
            size="sm"
            onClick={() => setPage(page - 1)}
            disabled={page === 1}
          >
            Previous
          </Button>
          <Button
            variant="outline"
            size="sm"
            onClick={() => setPage(page + 1)}
            disabled={page >= (data?.pagination.pages || 1)}
          >
            Next
          </Button>
        </div>
      </div>
    </div>
  );
}
```

#### 4.4 Complaint Detail Page with tRPC
```typescript
// apps/web/app/dashboard/complaints/[id]/page.tsx
'use client';

import { ComplaintDetails } from '@/components/complaint/complaint-details';
import { ChatLog } from '@/components/complaint/chat-log';
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card';
import { trpc } from '@/lib/trpc';
import { useParams } from 'next/navigation';

export default function ComplaintPage() {
  const params = useParams();
  const complaintId = params.id as string;

  const { data: complaint, isLoading, error } = trpc.complaint.getById.useQuery({
    id: complaintId
  });

  if (isLoading) {
    return <div className="flex justify-center p-8">Loading...</div>;
  }

  if (error) {
    return <div className="flex justify-center p-8 text-red-500">Error: {error.message}</div>;
  }

  if (!complaint) {
    return <div className="flex justify-center p-8">Complaint not found</div>;
  }

  return (
    <div className="space-y-6">
      <div className="flex items-center justify-between">
        <h1 className="text-3xl font-bold">Complaint {complaint.complaint_id}</h1>
      </div>

      <div className="grid grid-cols-1 lg:grid-cols-3 gap-6">
        <div className="lg:col-span-2">
          <Card>
            <CardHeader>
              <CardTitle>Conversation Log</CardTitle>
            </CardHeader>
            <CardContent>
              <ChatLog chatLogs={complaint.chat_logs} />
            </CardContent>
          </Card>
        </div>

        <div>
          <Card>
            <CardHeader>
              <CardTitle>Complaint Details</CardTitle>
            </CardHeader>
            <CardContent>
              <ComplaintDetails complaint={complaint} />
            </CardContent>
          </Card>
        </div>
      </div>
    </div>
  );
}
```

#### 4.5 Chat Log Component
```typescript
// apps/web/components/complaint/chat-log.tsx
import { ChatLog as ChatLogType } from '@packages/shared/types/complaint';
import { Card } from '@/components/ui/card';
import { Badge } from '@/components/ui/badge';

interface ChatLogProps {
  chatLogs: ChatLogType[];
}

export function ChatLog({ chatLogs }: ChatLogProps) {
  return (
    <div className="space-y-3 max-h-96 overflow-y-auto">
      {chatLogs.map((log, index) => (
        <div
          key={index}
          className={`flex ${log.direction === 'user' ? 'justify-end' : 'justify-start'}`}
        >
          <Card className={`max-w-xs px-3 py-2 ${
            log.direction === 'user'
              ? 'bg-blue-500 text-white'
              : 'bg-gray-100 text-gray-900'
          }`}>
            <div className="space-y-1">
              <div className="flex items-center gap-2 text-xs opacity-70">
                <Badge variant={log.direction === 'user' ? 'secondary' : 'default'} className="text-xs">
                  {log.direction}
                </Badge>
                <span>{new Date(log.timestamp).toLocaleTimeString()}</span>
              </div>
              <p className="text-sm">{log.message}</p>
              {log.message_type === 'command' && (
                <Badge variant="outline" className="text-xs">
                  {log.message_type}
                </Badge>
              )}
            </div>
          </Card>
        </div>
      ))}
    </div>
  );
}
```

### Phase 5: Data Visualization & Reports (Week 5)

#### 5.1 AI Analytics Dashboard
```typescript
// apps/web/app/dashboard/analytics/page.tsx
import { SentimentOverview } from '@/components/analytics/sentiment-overview';
import { WordCloudVisualization } from '@/components/analytics/word-cloud';
import { KeywordsChart } from '@/components/analytics/keywords-chart';
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card';

export default function AnalyticsPage() {
  return (
    <div className="space-y-6">
      <div className="flex items-center justify-between">
        <h1 className="text-3xl font-bold">AI Analytics</h1>
      </div>

      <SentimentOverview />

      <div className="grid grid-cols-1 lg:grid-cols-2 gap-6">
        <Card>
          <CardHeader>
            <CardTitle>Word Cloud</CardTitle>
          </CardHeader>
          <CardContent>
            <WordCloudVisualization />
          </CardContent>
        </Card>

        <Card>
          <CardHeader>
            <CardTitle>Top Keywords</CardTitle>
          </CardHeader>
          <CardContent>
            <KeywordsChart />
          </CardContent>
        </Card>
      </div>
    </div>
  );
}
```

#### 5.2 Simple Sentiment Overview Component
```typescript
// apps/web/components/analytics/sentiment-overview.tsx
'use client';

import { trpc } from '@/lib/trpc';
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card';

export function SentimentOverview() {
  const { data, isLoading } = trpc.report.analytics.overview.useQuery();

  if (isLoading) return <div>Loading analytics...</div>;

  const sentimentData = data?.sentimentStats || [];
  const total = sentimentData.reduce((sum, item) => sum + item.count, 0);

  return (
    <div className="grid grid-cols-3 gap-4">
      {sentimentData.map((item) => (
        <Card key={item._id}>
          <CardHeader className="pb-2">
            <CardTitle className="text-lg capitalize">{item._id}</CardTitle>
          </CardHeader>
          <CardContent>
            <div className="text-2xl font-bold">{item.count}</div>
            <p className="text-sm text-gray-600">
              {total > 0 ? Math.round((item.count / total) * 100) : 0}% of complaints
            </p>
          </CardContent>
        </Card>
      ))}
    </div>
  );
}
```

#### 5.3 Simple Word Cloud Component
```typescript
// apps/web/components/analytics/word-cloud.tsx
'use client';

import { trpc } from '@/lib/trpc';

export function WordCloudVisualization() {
  const { data: wordData, isLoading } = trpc.report.analytics.wordCloud.useQuery();

  if (isLoading) return <div>Loading word cloud...</div>;

  return (
    <div className="flex flex-wrap gap-2 p-4">
      {wordData?.map((word, index) => (
        <span
          key={word.text}
          className="inline-block px-3 py-1 bg-blue-100 text-blue-800 rounded-full"
          style={{
            fontSize: `${Math.min(Math.max(word.value * 2 + 10, 12), 24)}px`,
          }}
        >
          {word.text}
        </span>
      ))}
    </div>
  );
}
```

#### 5.4 Keywords Chart Component
```typescript
// apps/web/components/analytics/keywords-chart.tsx
'use client';

import { trpc } from '@/lib/trpc';

export function KeywordsChart() {
  const { data, isLoading } = trpc.report.analytics.overview.useQuery();

  if (isLoading) return <div>Loading keywords...</div>;

  const keywords = data?.topKeywords || [];

  return (
    <div className="space-y-2">
      {keywords.slice(0, 8).map((keyword, index) => (
        <div key={keyword._id} className="flex justify-between items-center">
          <span className="text-sm font-medium">{keyword._id}</span>
          <div className="flex items-center gap-2">
            <div
              className="bg-blue-500 h-2 rounded"
              style={{
                width: `${Math.max((keyword.count / keywords[0]?.count) * 100, 10)}px`,
              }}
            />
            <span className="text-sm text-gray-600 w-8 text-right">{keyword.count}</span>
          </div>
        </div>
      ))}
    </div>
  );
}
```

#### 5.2 Analytics tRPC Router
```typescript
// apps/backend/src/trpc/routers/analytics.ts
import { z } from 'zod';
import { router, publicProcedure } from '../trpc';
import { ComplaintSession } from '@packages/database/models/complaint-session';
import { ComplaintAnalytics } from '@packages/database/models/ai-analytics';

export const analyticsRouter = router({
  // Basic overview with AI insights
  overview: publicProcedure
    .query(async () => {
      const [totalComplaints, sentimentStats, topKeywords] = await Promise.all([
        ComplaintSession.countDocuments({ status: 'submitted' }),

        // Sentiment distribution
        ComplaintAnalytics.aggregate([
          { $group: { _id: '$sentiment', count: { $sum: 1 } } }
        ]),

        // Top keywords
        ComplaintAnalytics.aggregate([
          { $unwind: '$tags_keywords' },
          { $group: { _id: '$tags_keywords', count: { $sum: 1 } } },
          { $sort: { count: -1 } },
          { $limit: 10 }
        ])
      ]);

      return {
        totalComplaints,
        sentimentStats,
        topKeywords
      };
    }),

  // Word cloud data
  wordCloud: publicProcedure
    .query(async () => {
      const result = await ComplaintAnalytics.aggregate([
        { $unwind: '$tags_keywords' },
        { $group: {
            _id: '$tags_keywords',
            value: { $sum: 1 }
        }},
        { $sort: { value: -1 } },
        { $limit: 30 },
        { $project: {
            text: '$_id',
            value: 1,
            _id: 0
        }}
      ]);

      return result;
    }),

  // Sentiment trends over time
  sentimentTrends: publicProcedure
    .query(async () => {
      const result = await ComplaintAnalytics.aggregate([
        {
          $group: {
            _id: {
              sentiment: '$sentiment',
              month: { $month: '$created_at' },
              year: { $year: '$created_at' }
            },
            count: { $sum: 1 }
          }
        },
        { $sort: { '_id.year': 1, '_id.month': 1 } }
      ]);

      return result;
    })
});

// Update main router to include analytics
export const reportRouter = router({
  overview: publicProcedure.query(async () => {
    // Basic complaint counts
    const totalComplaints = await ComplaintSession.countDocuments({});
    const openComplaints = await ComplaintSession.countDocuments({ status: 'open' });
    const submittedComplaints = await ComplaintSession.countDocuments({ status: 'submitted' });

    return {
      totalComplaints,
      openComplaints,
      submittedComplaints
    };
  }),

  analytics: analyticsRouter
});
```

## Development Timeline

| Phase | Duration | Tasks | Dependencies |
|-------|----------|-------|--------------|
| Phase 1 | Week 1 | Foundation setup, database models, shared types | None |
| Phase 2 | Week 2 | Backend with tRPC server and LINE OA bot integration | Phase 1 |
| Phase 3 | Week 3 | Frontend tRPC client integration | Phase 1, 2 |
| Phase 4 | Week 4 | HR dashboard frontend components | Phase 1, 2, 3 |
| Phase 5 | Week 5 | Data visualization and reports | Phase 1, 2, 3, 4 |
| Testing | Week 6 | Integration testing, UAT | All phases |
| Deployment | Week 7 | Production deployment and monitoring | All phases |

## Technology Stack Details

### Frontend (HR Dashboard)
- **Framework**: Next.js 15 with App Router
- **Styling**: TailwindCSS + shadcn/ui components
- **State Management**: React hooks + tRPC React Query
- **AI Visualization**: Custom sentiment cards, word cloud, keyword charts
- **API Layer**: tRPC client with automatic TypeScript inference

### Backend (API + LINE Bot + AI)
- **Runtime**: Node.js 18+
- **Framework**: Express.js with tRPC
- **LINE SDK**: @line/bot-sdk
- **AI Service**: Google Gemini 1.5 Flash
- **Database ODM**: Mongoose
- **API Layer**: tRPC server with Zod validation

### Database
- **Primary**: MongoDB Atlas
- **Collections**: complaint_sessions, employees, complaint_analytics, line_events_raw
- **AI Analytics**: Sentiment analysis, keyword tagging, confidence scores
- **Indexing**: Optimized for read-heavy workloads and AI queries

### Deployment
- **Platform**: Railway
- **CI/CD**: GitHub Actions
- **Monitoring**: Built-in Railway monitoring
- **SSL**: Automatic HTTPS

## Security Considerations

1. **Data Encryption**: At-rest encryption in MongoDB
2. **Network Security**: HTTPS enforced, webhook validation
3. **Input Validation**: tRPC with Zod schema validation
4. **Data Retention**: TTL policies for temporary data

## Testing Strategy

### Unit Testing
- Backend services and API endpoints
- React components and hooks
- Database model validation

### Integration Testing
- LINE webhook to database flow
- tRPC API endpoints with database
- Frontend to backend data flow

### User Acceptance Testing
- HR dashboard workflows
- LINE OA complaint submission
- Reports and analytics accuracy

## Deployment Strategy

### Environment Setup
```bash
# Development
npm run dev

# Production build
npm run build

# Production start
npm run start
```

### Environment Variables
```env
# Database
MONGODB_URI=mongodb+srv://auth:auth@webapp-ireadcustomer.fyywwxu.mongodb.net/irc_platform_prod

# LINE
LINE_CHANNEL_ACCESS_TOKEN=your_access_token
LINE_CHANNEL_SECRET=your_channel_secret

# AI Service
GEMINI_API_KEY=your_gemini_api_key

# Backend
PORT=3001
FRONTEND_URL=http://localhost:3000

# Frontend
NEXT_PUBLIC_BACKEND_URL=http://localhost:3001
```

## ✅ AI Analytics Features Added

### 🧠 AI Analysis Capabilities
- **Sentiment Analysis**: Positive, negative, neutral classification with confidence scores
- **Keyword Tagging**: Automatic extraction of 3-5 relevant topics per complaint
- **Smart Categorization**: Topics like "management", "workload", "harassment", etc.

### 📊 Dashboard Visualizations
- **Sentiment Overview**: Distribution of positive/negative/neutral complaints
- **Word Cloud**: Visual representation of most common complaint keywords
- **Top Keywords Chart**: Bar chart showing frequency of complaint topics
- **Trend Analysis**: Sentiment patterns over time

### 🔧 Technical Implementation
- **Google Gemini Integration**: Simple AI analysis using Gemini 1.5 Flash
- **Background Processing**: Automatic analysis when complaints are submitted
- **tRPC Analytics API**: Type-safe endpoints for dashboard data
- **MongoDB Analytics Collection**: Optimized storage for AI insights

### 🚀 Simple & Lightweight
- **Minimal Complexity**: Just sentiment + keywords, no over-engineering
- **Fast Processing**: Efficient Gemini prompts for quick analysis
- **Clean UI**: Simple, intuitive visualizations for HR teams
- **Scalable Architecture**: Ready for future AI enhancements

This implementation plan provides a comprehensive roadmap for developing the HR dashboard with LINE OA integration and AI analytics, ensuring scalability, security, and maintainability throughout the development process.