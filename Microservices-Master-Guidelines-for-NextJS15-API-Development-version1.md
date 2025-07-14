# ArtistRSVP.com Master Architecture Guidelines - Complete Development Reference

## Executive Summary

This document serves as the comprehensive architectural and development guideline for the ArtistRSVP.com B2B2C SaaS platform. ArtistRSVP is an **AI-First, WhatsApp-First** platform that revolutionizes the global events industry. Every API endpoint and component integrates AI capabilities and WhatsApp functionality. All developers and AI assistants must follow these patterns to ensure **scalability**, **reusability**, **maintainability**, and **system stability**.

## Complete Table of Contents

1. [Overall Tech Stack and Boundaries](#overall-tech-stack-and-boundaries)
2. [AI-First & WhatsApp-First Architecture Overview](#ai-first-whatsapp-first-architecture-overview)
3. [Domain-Driven Microservices Organization with Routes](#domain-driven-microservices-organization-with-routes)
4. [Service Communication Patterns](#service-communication-patterns)
5. [Data Flow and API Architecture](#data-flow-and-api-architecture)
6. [Database Architecture](#database-architecture)
7. [TypeORM Standards and Best Practices](#typeorm-standards-and-best-practices)
8. [Monorepo Structure with All Services](#monorepo-structure-with-all-services)
9. [Shared Libraries and Code Reusability](#shared-libraries-and-code-reusability)
10. [Configuration Management](#configuration-management)
11. [Event-Driven Architecture](#event-driven-architecture)
12. [API Documentation Standards](#api-documentation-standards)
13. [Error Handling and Logging](#error-handling-and-logging)
14. [Frontend Development Guidelines](#frontend-development-guidelines)
15. [Mobile App Development Guidelines](#mobile-app-development-guidelines)
16. [Third-Party API Integrations](#third-party-api-integrations)
17. [Service Level Agreements](#service-level-agreements)
18. [AWS Lambda/API Gateway Development Guidelines](#aws-lambda-api-gateway-development-guidelines)
19. [Security and Authentication](#security-and-authentication)
20. [Monitoring and Observability](#monitoring-and-observability)
21. [Developer-Specific Best Practices](#developer-specific-best-practices)

## Overall Tech Stack and Boundaries

### Backend Stack (Serverless - No Docker)
- **Runtime**: Node.js 20.x LTS on AWS Lambda
- **Framework**: NestJS 10.x
- **Database**: 
  - PostgreSQL 15 (RDS) - Primary for most services
  - DynamoDB - Timeline data (Content Service)
  - Redis (ElastiCache) - Caching and sessions
  - Redshift - Analytics Service (data warehouse)
- **ORM**: TypeORM 0.3.x
- **GraphQL**: Apollo Server 4.x (Content Service, Discovery Service, Analytics Service)
- **API Gateway**: AWS API Gateway v2 (HTTP APIs)
- **API Documentation**: Swagger/OpenAPI 3.0
- **Message Queue**: AWS SQS, EventBridge
- **File Storage**: AWS S3, CloudFront CDN
- **Authentication**: JWT, AWS Cognito
- **Domain Management**: AWS Route53
- **Monitoring**: AWS X-Ray, CloudWatch
- **Infrastructure**: AWS CDK/Terraform

### Frontend Stack
- **Framework**: Next.js 15.0.0 (App Router)
- **UI Library**: React 19.0.0
- **State Management**: Zustand, TanStack Query
- **Forms**: React Hook Form 7.x with Yup validation
- **Internationalization**: i18next 23.x
- **Styling**: Tailwind CSS 3.4, shadcn/ui
- **Component Documentation**: Storybook 7.x
- **Testing**: Jest, React Testing Library, Playwright
- **Hosting**: Vercel Pro

### Mobile Stack
- **Framework**: React Native 0.73.x
- **Navigation**: React Navigation 6.x
- **State Management**: Zustand, TanStack Query
- **UI Components**: React Native Elements, Reanimated 3
- **Testing**: Jest, Detox
- **Distribution**: iOS App Store, Google Play Store

### AI & WhatsApp Integration
- **WhatsApp**: WhatsApp Cloud API
- **AI Providers**: OpenAI, Anthropic, Google AI, Meta AI, Perplexity, DeepSeek, Grok
- **LLM Orchestration**: LangChain, Custom AI Router
- **Vector Database**: Pinecone/Weaviate for embeddings

## AI-First & WhatsApp-First Architecture Overview

### Core Architectural Principles

1. **AI-First Design**
   - Every service includes an AI layer for intelligent processing
   - AI-powered recommendations, content generation, and automation
   - Multi-LLM orchestration for cost optimization
   - Embeddings-based search and similarity matching
   - Predictive analytics and demand forecasting

2. **WhatsApp-First Integration**
   - WhatsApp-based authentication (OTP)
   - RFP responses via WhatsApp
   - Event notifications and updates
   - Contract discussions and negotiations
   - Content creation through WhatsApp
   - Customer support via WhatsApp Business API

3. **Serverless-First**
   - All services run as AWS Lambda functions
   - Auto-scaling with zero infrastructure management
   - Pay-per-execution pricing model
   - Cold start optimization strategies

4. **Domain-Driven Design**
   - Services organized by business domains
   - Clear bounded contexts
   - Shared kernel for common functionality

5. **Event-Driven Communication**
   - Asynchronous messaging via AWS EventBridge
   - Event sourcing for audit trails
   - CQRS for read/write separation

## Domain-Driven Microservices Organization with Routes

### Identity & Access Domain

#### AuthService (`/api/v1/auth`)
**Scope**: WhatsApp-first authentication, JWT management, OAuth2 integration, session handling

**Core Routes**:
```
POST   /auth/whatsapp/send-otp         - Send OTP via WhatsApp
POST   /auth/whatsapp/verify-otp       - Verify WhatsApp OTP
POST   /auth/google/oauth              - Google OAuth (mandatory for calendar)
POST   /auth/refresh-token             - Refresh JWT token
GET    /auth/session/validate          - Validate current session
POST   /auth/logout                    - Logout user
POST   /auth/biometric/enable          - Enable biometric auth (mobile)
DELETE /auth/revoke-all                - Revoke all sessions
```

**Dependencies**: Calls UserProfileService, NotificationService
**Called By**: All services requiring authentication

#### UserProfileService (`/api/v1/users`)
**Scope**: User profiles, preferences, multi-tenant management, AI-powered profile enrichment

**Core Routes**:
```
GET    /users/profile/:userId          - Get user profile
PUT    /users/profile/:userId          - Update profile
POST   /users/profile/ai-enrich        - AI-enhance profile
GET    /users/microsite/:userId        - Get microsite data
POST   /users/subscription/upgrade     - Upgrade subscription tier
GET    /users/search                   - Search users with AI ranking
POST   /users/verify-documents         - Verify identity documents
PUT    /users/claim-profile            - Claim unclaimed profile
GET    /users/recommendations          - AI-powered user recommendations
POST   /users/bulk-import              - Import users from CSV/Excel
```

**Dependencies**: Calls MediaStorageService, AIService
**Called By**: All services needing user data

### Event Management Domain

#### EventService (`/api/v1/events`)
**Scope**: Core event lifecycle, scheduling, capacity management, AI-powered matching

**Core Routes**:
```
POST   /events/create                  - Create event
GET    /events/search                  - Search events with AI
GET    /events/:eventId                - Get event details
PUT    /events/:eventId                - Update event
DELETE /events/:eventId                - Cancel event
POST   /events/:eventId/publish        - Publish draft event
GET    /events/recommendations         - AI event recommendations
POST   /events/:eventId/clone          - Clone existing event
GET    /events/analytics/:eventId      - Event performance analytics
POST   /events/bulk-create             - Create multiple events
GET    /events/trending                - Trending events (AI-ranked)
```

**Dependencies**: Calls CalendarService, NotificationService, AIService
**Called By**: RFPService, TicketingService, DiscoveryService

#### CalendarService (`/api/v1/calendar`)
**Scope**: Google Calendar integration, availability management, timezone handling

**Core Routes**:
```
GET    /calendar/availability/:userId   - Check user availability
POST   /calendar/block-time            - Block calendar slots
PUT    /calendar/sync/:eventId         - Sync event to Google Calendar
POST   /calendar/create-event          - Create calendar event
DELETE /calendar/event/:eventId        - Remove from calendar
GET    /calendar/conflicts             - Check scheduling conflicts
POST   /calendar/bulk-sync             - Sync multiple events
GET    /calendar/suggestions           - AI-powered time suggestions
```

**Dependencies**: External Google Calendar API
**Called By**: EventService, RFPService, BookingService

#### TicketingService (`/api/v1/tickets`)
**Scope**: Ticket creation, sales, validation, QR codes, dynamic pricing

**Core Routes**:
```
POST   /tickets/create                 - Create ticket types
GET    /tickets/event/:eventId         - Get event tickets
POST   /tickets/purchase               - Purchase tickets
GET    /tickets/validate/:qrCode       - Validate ticket QR
POST   /tickets/transfer               - Transfer ticket ownership
GET    /tickets/user/:userId           - User's tickets
POST   /tickets/refund                 - Process refund
GET    /tickets/analytics              - Sales analytics
POST   /tickets/dynamic-pricing        - AI-based pricing
GET    /tickets/recommendations        - Recommended events
```

**Dependencies**: Calls PaymentService, EventService, NotificationService
**Called By**: Frontend applications, Mobile apps

### Communication Domain

#### CommunicationService (`/api/v1/messages`)
**Scope**: Real-time messaging, chat rooms, WhatsApp integration

**Core Routes**:
```
POST   /messages/send                  - Send message
GET    /messages/conversation/:id      - Get conversation
POST   /messages/whatsapp/send         - Send via WhatsApp
GET    /messages/unread                - Get unread count
PUT    /messages/read/:messageId       - Mark as read
POST   /messages/group/create          - Create group chat
DELETE /messages/:messageId            - Delete message
GET    /messages/search                - Search messages
POST   /messages/ai-translate          - AI translation
GET    /messages/sentiment              - Sentiment analysis
```

**Dependencies**: Calls WhatsApp API, AIService, NotificationService
**Called By**: All services needing messaging

#### NotificationService (`/api/v1/notifications`)
**Scope**: Push notifications, SMS, in-app alerts, notification preferences

**Core Routes**:
```
POST   /notifications/send             - Send notification
GET    /notifications/user/:userId     - User notifications
PUT    /notifications/read/:id         - Mark as read
PUT    /notifications/preferences      - Update preferences
POST   /notifications/bulk             - Bulk notifications
GET    /notifications/templates        - Notification templates
POST   /notifications/schedule         - Schedule notification
DELETE /notifications/:id              - Delete notification
GET    /notifications/analytics        - Delivery analytics
```

**Dependencies**: Calls WhatsApp API, Twilio, Firebase
**Called By**: All services

#### EmailService (`/api/v1/emails`)
**Scope**: Transactional emails, campaigns, templates, analytics

**Core Routes**:
```
POST   /emails/send                    - Send email
POST   /emails/campaign/create         - Create campaign
GET    /emails/templates               - Email templates
POST   /emails/template/create         - Create template
GET    /emails/analytics/:campaignId   - Campaign analytics
POST   /emails/bulk                    - Bulk email send
GET    /emails/bounces                 - Bounce management
PUT    /emails/unsubscribe             - Unsubscribe user
POST   /emails/verify                  - Verify email address
```

**Dependencies**: AWS SES, SendGrid
**Called By**: All services needing email

### Content & Discovery Domain

#### ContentService (`/api/v1/content`) - GraphQL
**Scope**: Content creation, publishing, feeds, AI-powered content generation

**GraphQL Schema**:
```graphql
type Query {
  # Content queries
  content(id: ID!): Content
  feed(userId: ID!, pagination: PaginationInput): ContentFeed!
  trending(timeRange: TimeRange!, limit: Int): [Content!]!
  search(query: String!, filters: ContentFilters): ContentSearchResult!
  recommendations(userId: ID!, limit: Int): [Content!]!
  
  # Comments and interactions
  comments(contentId: ID!, pagination: PaginationInput): CommentList!
  likes(contentId: ID!): LikeStats!
}

type Mutation {
  # Content management
  createContent(input: CreateContentInput!): Content!
  updateContent(id: ID!, input: UpdateContentInput!): Content!
  deleteContent(id: ID!): Boolean!
  
  # AI operations
  generateContent(input: AIContentInput!): Content!
  enhanceContent(id: ID!, options: EnhanceOptions!): Content!
  translateContent(id: ID!, targetLanguage: String!): Content!
  
  # Interactions
  likeContent(contentId: ID!): Like!
  unlikeContent(contentId: ID!): Boolean!
  commentOnContent(contentId: ID!, comment: String!): Comment!
  reportContent(contentId: ID!, reason: ReportReason!): Report!
}

type Subscription {
  # Real-time updates
  contentUpdated(contentId: ID!): Content!
  newComment(contentId: ID!): Comment!
  feedUpdate(userId: ID!): FeedUpdate!
}

type Content {
  id: ID!
  type: ContentType!
  title: String!
  body: String!
  media: [Media!]
  author: User!
  tags: [String!]
  likes: Int!
  comments: Int!
  shares: Int!
  aiGenerated: Boolean!
  createdAt: DateTime!
  updatedAt: DateTime!
}

enum ContentType {
  POST
  ARTICLE
  VIDEO
  IMAGE
  STORY
  REEL
}
```

**Dependencies**: Calls AIService, MediaStorageService, UserProfileService
**Called By**: Frontend apps, Mobile apps

#### DiscoveryService (`/api/v1/discovery`) - GraphQL
**Scope**: Search, trending algorithms, AI-powered discovery

**GraphQL Schema**:
```graphql
type Query {
  # Universal search
  search(query: String!, type: SearchType, filters: SearchFilters): SearchResult!
  
  # Discovery
  discoverArtists(filters: ArtistFilters, limit: Int): [Artist!]!
  discoverEvents(filters: EventFilters, limit: Int): [Event!]!
  discoverContent(userId: ID!, limit: Int): [Content!]!
  
  # Trending
  trendingArtists(timeRange: TimeRange!, limit: Int): [TrendingArtist!]!
  trendingEvents(timeRange: TimeRange!, limit: Int): [TrendingEvent!]!
  trendingHashtags(limit: Int): [Hashtag!]!
  
  # Recommendations
  recommendedForYou(userId: ID!, type: RecommendationType): Recommendations!
  similarArtists(artistId: ID!, limit: Int): [Artist!]!
  similarEvents(eventId: ID!, limit: Int): [Event!]!
}

type Mutation {
  # Feedback
  recordInteraction(input: InteractionInput!): Boolean!
  improveRecommendations(userId: ID!, feedback: FeedbackInput!): Boolean!
  
  # AI operations
  aiMatch(input: AIMatchInput!): MatchResult!
}

type SearchResult {
  artists: [Artist!]!
  events: [Event!]!
  content: [Content!]!
  teams: [Team!]!
  totalResults: Int!
  facets: SearchFacets!
}

type Recommendations {
  artists: [Artist!]!
  events: [Event!]!
  content: [Content!]!
  reason: String!
  confidence: Float!
}
```

**Dependencies**: Calls AIService, ElasticSearch
**Called By**: All discovery features

#### MediaStorageService (`/api/v1/media`)
**Scope**: File uploads, CDN integration, image/video processing

**Core Routes**:
```
POST   /media/upload                   - Upload file
GET    /media/:mediaId                 - Get media URL
DELETE /media/:mediaId                 - Delete media
POST   /media/process                  - Process media (resize, etc.)
GET    /media/gallery/:userId          - User's media gallery
POST   /media/ai-enhance               - AI image enhancement
POST   /media/generate                 - AI image generation
GET    /media/analytics                - Storage analytics
POST   /media/bulk-upload              - Bulk upload
PUT    /media/metadata/:mediaId        - Update metadata
```

**Dependencies**: AWS S3, CloudFront, AI image services
**Called By**: All services handling media

### Commerce Domain

#### WalletService (`/api/v1/wallet`)
**Scope**: Digital wallet, balance management, transaction history

**Core Routes**:
```
GET    /wallet/balance/:userId         - Get wallet balance
POST   /wallet/add-funds               - Add funds
POST   /wallet/withdraw                - Withdraw funds
GET    /wallet/transactions            - Transaction history
POST   /wallet/transfer                - Transfer to another user
GET    /wallet/statement               - Generate statement
POST   /wallet/refund                  - Process refund
GET    /wallet/pending                 - Pending transactions
POST   /wallet/verify-kyc              - KYC verification
GET    /wallet/limits                  - Transaction limits
```

**Dependencies**: Calls PaymentService
**Called By**: PaymentService, BillingService

#### PaymentService (`/api/v1/payments`)
**Scope**: Payment processing, multiple gateways, PCI compliance

**Core Routes**:
```
POST   /payments/intent                - Create payment intent
POST   /payments/process               - Process payment
GET    /payments/:paymentId            - Payment status
POST   /payments/refund                - Process refund
GET    /payments/methods               - Available methods
POST   /payments/save-card             - Save card for future
DELETE /payments/card/:cardId          - Remove saved card
POST   /payments/webhook               - Gateway webhooks
GET    /payments/history               - Payment history
POST   /payments/subscription          - Create subscription
```

**Dependencies**: Stripe, Razorpay, PayPal APIs
**Called By**: TicketingService, StoreService, BillingService

#### BillingService (`/api/v1/billing`)
**Scope**: Subscription management, invoicing, revenue recognition

**Core Routes**:
```
GET    /billing/subscription/:userId   - Get subscription
POST   /billing/subscribe              - Create subscription
PUT    /billing/upgrade                - Upgrade plan
DELETE /billing/cancel                 - Cancel subscription
GET    /billing/invoices               - List invoices
GET    /billing/invoice/:id            - Download invoice
POST   /billing/usage                  - Report usage
GET    /billing/revenue                - Revenue analytics
POST   /billing/discount               - Apply discount
GET    /billing/tax                    - Tax calculation
```

**Dependencies**: Calls PaymentService, integrates with Zoho Books/QuickBooks
**Called By**: UserProfileService, AdminService

#### StoreService (`/api/v1/store`)
**Scope**: Merchandise, digital goods, inventory management

**Core Routes**:
```
POST   /store/product/create           - Create product
GET    /store/products                 - List products
GET    /store/product/:productId       - Product details
POST   /store/cart/add                 - Add to cart
GET    /store/cart                     - View cart
POST   /store/checkout                 - Checkout
GET    /store/orders                   - Order history
POST   /store/review                   - Product review
GET    /store/inventory                - Check inventory
POST   /store/wishlist/add             - Add to wishlist
```

**Dependencies**: Calls PaymentService, WalletService, NotificationService
**Called By**: Frontend apps

### Collaboration Domain

#### TeamsService (`/api/v1/teams`)
**Scope**: Team creation, permissions, hierarchy, ensemble management

**Core Routes**:
```
POST   /teams/create                   - Create team
GET    /teams/:teamId                  - Get team details
PUT    /teams/:teamId                  - Update team
POST   /teams/:teamId/invite           - Invite member
DELETE /teams/:teamId/member/:userId   - Remove member
PUT    /teams/:teamId/permissions      - Update permissions
GET    /teams/user/:userId             - User's teams
POST   /teams/:teamId/project          - Create team project
GET    /teams/:teamId/analytics        - Team analytics
POST   /teams/merge                    - Merge teams
```

**Dependencies**: Calls UserProfileService, NotificationService
**Called By**: ProjectService, RFPService

#### ProjectService (`/api/v1/projects`)
**Scope**: Campaign management, milestones, deliverables, project collaboration

**Core Routes**:
```
POST   /projects/create                - Create project
GET    /projects/:projectId            - Project details
PUT    /projects/:projectId            - Update project
POST   /projects/:projectId/task       - Add task
GET    /projects/:projectId/tasks      - List tasks
PUT    /projects/task/:taskId          - Update task
POST   /projects/:projectId/milestone  - Add milestone
GET    /projects/:projectId/timeline   - Project timeline
POST   /projects/:projectId/file       - Upload file
GET    /projects/analytics             - Project analytics
```

**Dependencies**: Calls TeamsService, MediaStorageService
**Called By**: RFPService, EventService

#### ConnectionsService (`/api/v1/connections`)
**Scope**: Social graph, relationship management, network analytics

**Core Routes**:
```
POST   /connections/follow             - Follow user
DELETE /connections/unfollow           - Unfollow user
GET    /connections/followers/:userId  - Get followers
GET    /connections/following/:userId  - Get following
POST   /connections/block              - Block user
DELETE /connections/unblock            - Unblock user
GET    /connections/mutual/:userId     - Mutual connections
GET    /connections/suggestions        - AI suggestions
GET    /connections/network-stats      - Network analytics
POST   /connections/import             - Import contacts
```

**Dependencies**: Calls AIService for suggestions
**Called By**: UserProfileService, DiscoveryService

#### RFPService (`/api/v1/rfp`)
**Scope**: Request for Proposal marketplace, AI matching, automated workflows

**Core Routes**:
```
POST   /rfp/create                     - Create RFP
GET    /rfp/search                     - Search RFPs
GET    /rfp/:rfpId                     - RFP details
POST   /rfp/:rfpId/respond             - Submit response
PUT    /rfp/response/:responseId       - Update response
GET    /rfp/matches/:rfpId             - AI matches
POST   /rfp/whatsapp/respond           - WhatsApp response
POST   /rfp/ai-generate                - AI-generate RFP
GET    /rfp/analytics                  - RFP analytics
POST   /rfp/:rfpId/award               - Award RFP
```

**Dependencies**: Calls AIService, EventService, CalendarService, WhatsApp API
**Called By**: Frontend apps, WhatsApp webhooks

### Intelligence Domain

#### AIService (`/api/v1/ai`)
**Scope**: Core AI operations, ML models, recommendations, content generation

**Core Routes**:
```
POST   /ai/generate-content            - Generate content
POST   /ai/translate                   - Translate text
POST   /ai/summarize                   - Summarize content
POST   /ai/sentiment                   - Sentiment analysis
POST   /ai/recommendations/user        - User recommendations
POST   /ai/recommendations/event       - Event recommendations
POST   /ai/match/artist-event          - Match artists to events
POST   /ai/predict/demand              - Demand prediction
POST   /ai/optimize/pricing            - Price optimization
GET    /ai/usage/analytics             - AI usage analytics
```

**Dependencies**: OpenAI, Anthropic, Google AI APIs
**Called By**: All services requiring AI

#### AILabsService (`/api/v1/ai-labs`)
**Scope**: Premium AI services, custom GPTs, LLM wrappers, AI applications

**Core Routes**:
```
GET    /ai-labs/services               - Available AI services
POST   /ai-labs/custom-gpt/create      - Create custom GPT
POST   /ai-labs/image/generate         - Generate images
POST   /ai-labs/video/generate         - Generate videos
POST   /ai-labs/music/generate         - Generate music
POST   /ai-labs/voice/clone            - Voice cloning
POST   /ai-labs/subtitle/generate      - Generate subtitles
GET    /ai-labs/models                 - Available models
POST   /ai-labs/fine-tune              - Fine-tune model
GET    /ai-labs/usage                  - Usage analytics
```

**Dependencies**: Multiple AI providers
**Called By**: Premium tier users

#### AnalyticsService (`/api/v1/analytics`) - GraphQL
**Scope**: Business intelligence, reporting, data warehouse, real-time analytics

**GraphQL Schema**:
```graphql
type Query {
  # Platform Analytics
  platformMetrics(timeRange: TimeRange!): PlatformMetrics!
  userAnalytics(userId: ID!, timeRange: TimeRange!): UserAnalytics!
  eventAnalytics(eventId: ID!): EventAnalytics!
  revenueAnalytics(timeRange: TimeRange!): RevenueAnalytics!
  
  # Custom Reports
  generateReport(type: ReportType!, filters: ReportFilters!): Report!
  exportData(query: ExportQuery!): ExportJob!
  
  # Real-time Metrics
  activeUsers: Int!
  currentRevenue: Float!
  liveEvents: [Event!]!
  
  # Dashboards
  getDashboard(id: ID!): Dashboard!
  listDashboards(userId: ID!): [Dashboard!]!
}

type Mutation {
  createDashboard(input: DashboardInput!): Dashboard!
  updateDashboard(id: ID!, input: DashboardInput!): Dashboard!
  deleteDashboard(id: ID!): Boolean!
  scheduleReport(input: ScheduledReportInput!): ScheduledReport!
}

type Subscription {
  realtimeMetrics(metrics: [MetricType!]!): MetricUpdate!
  alertNotification(userId: ID!): Alert!
}

type PlatformMetrics {
  totalUsers: Int!
  activeUsers: Int!
  totalEvents: Int!
  totalRevenue: Float!
  growthRate: Float!
  churnRate: Float!
}
```

**Dependencies**: AWS Redshift, data pipeline services
**Called By**: Admin dashboard, reporting features

### Platform Infrastructure Domain

[Continuing with all remaining services...]

## Service Communication Patterns

### Communication Matrix

| Service | Calls These Services | Called By These Services |
|---------|---------------------|-------------------------|
| AuthService | UserProfileService, NotificationService | All services |
| UserProfileService | MediaStorageService, AIService | All services |
| EventService | CalendarService, NotificationService, AIService | RFPService, TicketingService |
| CalendarService | Google Calendar API | EventService, RFPService |
| TicketingService | PaymentService, EventService, NotificationService | Frontend apps |
| CommunicationService | WhatsApp API, AIService, NotificationService | All services |
| NotificationService | WhatsApp API, Twilio, Firebase | All services |
| EmailService | AWS SES, SendGrid | All services |
| ContentService | AIService, MediaStorageService | Frontend apps |
| DiscoveryService | AIService, ElasticSearch | Frontend apps |
| MediaStorageService | AWS S3, CloudFront | All media-handling services |
| WalletService | PaymentService | PaymentService, BillingService |
| PaymentService | Stripe, Razorpay, PayPal | TicketingService, StoreService |
| BillingService | PaymentService, Zoho/QuickBooks | UserProfileService |
| StoreService | PaymentService, WalletService | Frontend apps |
| TeamsService | UserProfileService, NotificationService | ProjectService, RFPService |
| ProjectService | TeamsService, MediaStorageService | RFPService, EventService |
| ConnectionsService | AIService | UserProfileService |
| RFPService | AIService, EventService, CalendarService | Frontend apps |
| AIService | External AI APIs | All services |
| AILabsService | Multiple AI providers | Premium features |
| AnalyticsService | Redshift | Admin features |

### Synchronous Communication Patterns

**Direct Service Calls**: For real-time operations
```typescript
// Example: TicketingService calling PaymentService
async purchaseTicket(ticketData: PurchaseTicketDto) {
  // Validate ticket availability
  const ticket = await this.ticketRepository.findOne(ticketData.ticketId);
  
  // Call PaymentService synchronously
  const payment = await this.paymentService.processPayment({
    amount: ticket.price,
    currency: ticket.currency,
    customerId: ticketData.userId,
    metadata: { ticketId: ticket.id }
  });
  
  // Update ticket status
  ticket.status = 'SOLD';
  await this.ticketRepository.save(ticket);
  
  return { ticket, payment };
}
```

### Asynchronous Communication Patterns

**Event-Driven Architecture**:
```typescript
// Publishing an event
await this.eventBridge.putEvents({
  Entries: [{
    Source: 'ticketing-service',
    DetailType: 'TicketPurchased',
    Detail: JSON.stringify({
      ticketId: ticket.id,
      userId: user.id,
      eventId: event.id,
      purchaseTime: new Date()
    })
  }]
});

// Subscribing to events in another service
@EventPattern('TicketPurchased')
async handleTicketPurchase(data: TicketPurchasedEvent) {
  await this.notificationService.sendPurchaseConfirmation(data);
  await this.analyticsService.recordPurchase(data);
}
```

## Data Flow and API Architecture

### Complete Request Flow

```
Mobile/Web Client
    ↓
CloudFront (CDN)
    ↓
Route53 (DNS)
    ↓
API Gateway (HTTP API)
    ↓
Lambda Authorizer (JWT Validation)
    ↓
Service Router
    ├── REST Endpoints → Lambda Functions → PostgreSQL
    └── GraphQL Endpoints → Apollo Server → DynamoDB/Redshift
         ↓
    EventBridge (Async Events)
         ↓
    SQS Queues → Lambda Consumers
```

### API Gateway Configuration

```yaml
# serverless.yml example
functions:
  authService:
    handler: apps/backend/auth-service/dist/lambda.handler
    events:
      - httpApi:
          path: /api/v1/auth/*
          method: ANY
          authorizer:
            name: jwtAuthorizer
            type: request
            identitySource: $request.header.Authorization
            
  graphqlContent:
    handler: apps/backend/content-service/dist/graphql.handler
    events:
      - httpApi:
          path: /graphql/content
          method: POST
          authorizer:
            name: jwtAuthorizer
```

## Database Architecture

### Database Per Service Pattern

| Service | Database Type | Purpose |
|---------|--------------|---------|
| AuthService | PostgreSQL | User credentials, sessions |
| UserProfileService | PostgreSQL | User profiles, preferences |
| EventService | PostgreSQL | Event data, schedules |
| CalendarService | PostgreSQL | Calendar entries, availability |
| TicketingService | PostgreSQL | Tickets, sales data |
| CommunicationService | PostgreSQL + Redis | Messages, real-time data |
| NotificationService | PostgreSQL | Notification logs, templates |
| EmailService | PostgreSQL | Email logs, campaigns |
| ContentService | DynamoDB | Time-series content data |
| DiscoveryService | ElasticSearch | Search indices |
| MediaStorageService | PostgreSQL + S3 | Media metadata |
| WalletService | PostgreSQL | Transactions, balances |
| PaymentService | PostgreSQL | Payment records |
| BillingService | PostgreSQL | Subscriptions, invoices |
| StoreService | PostgreSQL | Products, orders |
| TeamsService | PostgreSQL | Team data, permissions |
| ProjectService | PostgreSQL | Projects, tasks |
| ConnectionsService | PostgreSQL + Graph DB | Social connections |
| RFPService | PostgreSQL | RFPs, responses |
| AIService | PostgreSQL + Vector DB | AI logs, embeddings |
| AnalyticsService | Redshift | Data warehouse |

### Data Consistency Patterns

**Strong Consistency Required**:
- Financial transactions (PaymentService, WalletService)
- User authentication (AuthService)
- Inventory management (TicketingService, StoreService)

**Eventual Consistency Acceptable**:
- Analytics data (AnalyticsService)
- Social features (ConnectionsService)
- Content feeds (ContentService)
- Search indices (DiscoveryService)

## TypeORM Standards and Best Practices

### Base Entity Configuration

```typescript
// packages/shared-database/src/entities/base.entity.ts
import { 
  PrimaryGeneratedColumn, 
  CreateDateColumn, 
  UpdateDateColumn, 
  VersionColumn,
  DeleteDateColumn,
  BaseEntity as TypeORMBaseEntity 
} from 'typeorm';

export abstract class BaseEntity extends TypeORMBaseEntity {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @CreateDateColumn({ type: 'timestamp with time zone' })
  createdAt: Date;

  @UpdateDateColumn({ type: 'timestamp with time zone' })
  updatedAt: Date;

  @DeleteDateColumn({ type: 'timestamp with time zone' })
  deletedAt?: Date;

  @VersionColumn()
  version: number;
}

export abstract class AuditableEntity extends BaseEntity {
  @Column({ type: 'uuid', nullable: true })
  createdBy?: string;

  @Column({ type: 'uuid', nullable: true })
  updatedBy?: string;
}
```

### Entity Example with Best Practices

```typescript
// apps/backend/event-service/src/entities/event.entity.ts
import { Entity, Column, Index, ManyToOne, OneToMany } from 'typeorm';
import { AuditableEntity } from '@artistrsvp/shared-database';

@Entity('events')
@Index(['status', 'eventDate'])
@Index(['organizerId', 'status'])
export class Event extends AuditableEntity {
  @Column({ type: 'varchar', length: 255 })
  @Index()
  title: string;

  @Column({ type: 'text' })
  description: string;

  @Column({ type: 'timestamp with time zone' })
  @Index()
  eventDate: Date;

  @Column({ type: 'int', default: 100 })
  capacity: number;

  @Column({ type: 'decimal', precision: 10, scale: 2 })
  price: number;

  @Column({ type: 'varchar', length: 3 })
  currency: string;

  @Column({ type: 'jsonb', nullable: true })
  metadata: Record<string, any>;

  @Column({ 
    type: 'enum', 
    enum: ['DRAFT', 'PUBLISHED', 'CANCELLED', 'COMPLETED'],
    default: 'DRAFT' 
  })
  @Index()
  status: string;

  @ManyToOne(() => User, { lazy: true })
  organizer: User;

  @OneToMany(() => Ticket, ticket => ticket.event, { cascade: true })
  tickets: Ticket[];
}
```

### Repository Pattern

```typescript
// packages/shared-database/src/repositories/base.repository.ts
import { Repository, FindOptionsWhere, DeepPartial } from 'typeorm';
import { BaseEntity } from '../entities/base.entity';

export abstract class BaseRepository<T extends BaseEntity> {
  constructor(protected readonly repository: Repository<T>) {}

  async findById(id: string, relations?: string[]): Promise<T | null> {
    return this.repository.findOne({ 
      where: { id } as FindOptionsWhere<T>,
      relations 
    });
  }

  async findByIds(ids: string[]): Promise<T[]> {
    return this.repository.findByIds(ids);
  }

  async create(data: DeepPartial<T>): Promise<T> {
    const entity = this.repository.create(data);
    return this.repository.save(entity);
  }

  async update(id: string, data: DeepPartial<T>): Promise<T> {
    await this.repository.update(id, data);
    return this.findById(id);
  }

  async softDelete(id: string): Promise<void> {
    await this.repository.softDelete(id);
  }

  async restore(id: string): Promise<void> {
    await this.repository.restore(id);
  }
}
```

### Migration Best Practices

```typescript
// apps/backend/event-service/src/migrations/1234567890-CreateEventTable.ts
import { MigrationInterface, QueryRunner, Table, Index } from 'typeorm';

export class CreateEventTable1234567890 implements MigrationInterface {
  public async up(queryRunner: QueryRunner): Promise<void> {
    // Create extension if not exists
    await queryRunner.query(`CREATE EXTENSION IF NOT EXISTS "uuid-ossp"`);
    
    // Create table
    await queryRunner.createTable(
      new Table({
        name: 'events',
        columns: [
          {
            name: 'id',
            type: 'uuid',
            isPrimary: true,
            default: 'uuid_generate_v4()',
          },
          {
            name: 'title',
            type: 'varchar',
            length: '255',
          },
          {
            name: 'description',
            type: 'text',
          },
          {
            name: 'event_date',
            type: 'timestamp with time zone',
          },
          {
            name: 'capacity',
            type: 'int',
            default: 100,
          },
          {
            name: 'price',
            type: 'decimal',
            precision: 10,
            scale: 2,
          },
          {
            name: 'currency',
            type: 'varchar',
            length: '3',
          },
          {
            name: 'metadata',
            type: 'jsonb',
            isNullable: true,
          },
          {
            name: 'status',
            type: 'enum',
            enum: ['DRAFT', 'PUBLISHED', 'CANCELLED', 'COMPLETED'],
            default: "'DRAFT'",
          },
          {
            name: 'organizer_id',
            type: 'uuid',
          },
          {
            name: 'created_at',
            type: 'timestamp with time zone',
            default: 'CURRENT_TIMESTAMP',
          },
          {
            name: 'updated_at',
            type: 'timestamp with time zone',
            default: 'CURRENT_TIMESTAMP',
          },
          {
            name: 'deleted_at',
            type: 'timestamp with time zone',
            isNullable: true,
          },
          {
            name: 'version',
            type: 'int',
            default: 1,
          },
        ],
      }),
      true,
    );

    // Create indices
    await queryRunner.createIndex(
      'events',
      new Index({
        name: 'IDX_EVENT_STATUS_DATE',
        columnNames: ['status', 'event_date'],
      }),
    );

    await queryRunner.createIndex(
      'events',
      new Index({
        name: 'IDX_EVENT_ORGANIZER',
        columnNames: ['organizer_id', 'status'],
      }),
    );

    await queryRunner.createIndex(
      'events',
      new Index({
        name: 'IDX_EVENT_TITLE',
        columnNames: ['title'],
      }),
    );
  }

  public async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.dropTable('events');
  }
}
```

### Query Optimization for Serverless

```typescript
// Use query builder for complex queries
async findUpcomingEventsWithAvailability(): Promise<Event[]> {
  return this.eventRepository
    .createQueryBuilder('event')
    .leftJoinAndSelect('event.tickets', 'ticket')
    .where('event.status = :status', { status: 'PUBLISHED' })
    .andWhere('event.eventDate > :now', { now: new Date() })
    .andWhere('event.capacity > (SELECT COUNT(*) FROM tickets WHERE event_id = event.id AND status = :sold)', { sold: 'SOLD' })
    .orderBy('event.eventDate', 'ASC')
    .limit(50)
    .cache(60000) // Cache for 1 minute
    .getMany();
}

// Use raw queries for read-heavy operations
async getEventStatistics(eventId: string): Promise<EventStats> {
  const result = await this.dataSource.query(`
    SELECT 
      e.id,
      e.title,
      e.capacity,
      COUNT(DISTINCT t.id) FILTER (WHERE t.status = 'SOLD') as tickets_sold,
      COUNT(DISTINCT t.user_id) as unique_attendees,
      SUM(t.price) FILTER (WHERE t.status = 'SOLD') as total_revenue,
      e.capacity - COUNT(DISTINCT t.id) FILTER (WHERE t.status = 'SOLD') as available_tickets
    FROM events e
    LEFT JOIN tickets t ON t.event_id = e.id
    WHERE e.id = $1
    GROUP BY e.id, e.title, e.capacity
  `, [eventId]);
  
  return result[0];
}
```

## Monorepo Structure with All Services

```
artistrsvp-monorepo/
├── apps/                              # Applications
│   ├── backend/                       # Backend microservices (30 services)
│   │   ├── auth-service/              # Identity & Access Domain
│   │   ├── user-profile-service/
│   │   ├── event-service/             # Event Management Domain
│   │   ├── calendar-service/
│   │   ├── ticketing-service/
│   │   ├── communication-service/     # Communication Domain
│   │   ├── notification-service/
│   │   ├── email-service/
│   │   ├── content-service/           # Content & Discovery Domain
│   │   ├── discovery-service/
│   │   ├── media-storage-service/
│   │   ├── wallet-service/            # Commerce Domain
│   │   ├── payment-service/
│   │   ├── billing-service/
│   │   ├── store-service/
│   │   ├── teams-service/             # Collaboration Domain
│   │   ├── project-service/
│   │   ├── connections-service/
│   │   ├── rfp-service/
│   │   ├── ai-service/                # Intelligence Domain
│   │   ├── ai-labs-service/
│   │   ├── analytics-service/
│   │   ├── integration-service/       # Platform Infrastructure
│   │   ├── audit-service/
│   │   ├── configuration-service/
│   │   ├── admin-service/
│   │   ├── microsite-service/
│   │   ├── rewards-service/
│   │   ├── lms-service/
│   │   └── ads-service/
│   ├── frontend/                      # Frontend applications
│   │   ├── web-app/                   # Next.js main application
│   │   │   ├── src/
│   │   │   │   ├── app/               # App Router
│   │   │   │   ├── components/        # UI Components
│   │   │   │   ├── hooks/             # Custom hooks
│   │   │   │   ├── services/          # API clients
│   │   │   │   ├── stores/            # State management
│   │   │   │   └── utils/             # Utilities
│   │   │   └── public/
│   │   ├── admin-portal/              # Admin dashboard
│   │   ├── microsite-builder/         # Microsite editor
│   │   └── landing-pages/             # Marketing sites
│   └── mobile/                        # Mobile applications
│       ├── ios/                       # iOS specific
│       ├── android/                   # Android specific
│       └── src/                       # Shared React Native code
│           ├── components/
│           ├── screens/
│           ├── navigation/
│           ├── services/
│           └── utils/
├── packages/                          # Shared libraries
│   ├── shared-types/                  # TypeScript interfaces and types
│   ├── shared-auth/                   # Authentication utilities
│   ├── shared-database/               # Database utilities
│   ├── shared-events/                 # Event schemas
│   ├── shared-utils/                  # Common utilities
│   ├── shared-config/                 # Configuration
│   ├── shared-logging/                # Logging
│   ├── shared-testing/                # Testing utilities
│   ├── shared-ai/                     # AI utilities
│   ├── shared-whatsapp/               # WhatsApp utilities
│   ├── ui-components/                 # Shared UI components
│   └── api-client/                    # Frontend API client
├── services/                          # Third-party service integrations
│   ├── google-services/               # Google APIs
│   │   ├── maps/
│   │   ├── places/
│   │   └── calendar/
│   ├── payment-providers/             # Payment gateways
│   │   ├── stripe/
│   │   ├── razorpay/
│   │   └── paypal/
│   ├── communication-providers/       # Communication APIs
│   │   ├── whatsapp/
│   │   ├── twilio/
│   │   └── firebase/
│   ├── ai-providers/                  # AI services
│   │   ├── openai/
│   │   ├── anthropic/
│   │   ├── google-ai/
│   │   ├── meta-ai/
│   │   ├── perplexity/
│   │   ├── deepseek/
│   │   └── grok/
│   ├── business-tools/                # Business integrations
│   │   ├── zoho/
│   │   ├── quickbooks/
│   │   ├── zoom/
│   │   └── agora/
│   └── media-generation/              # Media AI services
│       ├── image/
│       ├── audio/
│       └── video/
├── infrastructure/                    # Infrastructure as Code
│   ├── terraform/
│   ├── aws-cdk/
│   └── scripts/
├── tools/                            # Build and deployment tools
├── docs/                             # Documentation
├── nx.json                          # NX workspace configuration
├── package.json                     # Root package.json
├── tsconfig.base.json              # Base TypeScript configuration
└── .env.example                    # Environment variables template
```

## Shared Libraries and Code Reusability

### Shared Types Package

```typescript
// packages/shared-types/src/index.ts
export * from './entities';
export * from './dtos';
export * from './enums';
export * from './interfaces';

// packages/shared-types/src/enums/user.enum.ts
export enum UserType {
  FAN = 'fan',
  EXPERT = 'expert',
  ARTIST = 'artist',
  ADMIN = 'admin'
}

export enum SubscriptionTier {
  BASIC = 'basic',
  PRO = 'pro',
  PRO_MAX = 'pro_max'
}

// packages/shared-types/src/interfaces/user.interface.ts
export interface IUser {
  id: string;
  email: string;
  name: string;
  userType: UserType;
  subscriptionTier: SubscriptionTier;
  whatsappNumber?: string;
  googleId?: string;
  metadata?: Record<string, any>;
  createdAt: Date;
  updatedAt: Date;
}
```

### Shared Authentication Package

```typescript
// packages/shared-auth/src/guards/jwt-auth.guard.ts
import { Injectable, CanActivate, ExecutionContext, UnauthorizedException } from '@nestjs/common';
import { JwtService } from '@nestjs/jwt';
import { ConfigService } from '@nestjs/config';
import { Reflector } from '@nestjs/core';

@Injectable()
export class JwtAuthGuard implements CanActivate {
  constructor(
    private jwtService: JwtService,
    private configService: ConfigService,
    private reflector: Reflector
  ) {}

  async canActivate(context: ExecutionContext): Promise<boolean> {
    const isPublic = this.reflector.get<boolean>('isPublic', context.getHandler());
    if (isPublic) return true;

    const request = context.switchToHttp().getRequest();
    const token = this.extractTokenFromHeader(request);
    
    if (!token) {
      throw new UnauthorizedException('No token provided');
    }

    try {
      const payload = await this.jwtService.verifyAsync(token, {
        secret: this.configService.get<string>('JWT_SECRET')
      });
      
      request['user'] = payload;
      return true;
    } catch (error) {
      throw new UnauthorizedException('Invalid token');
    }
  }

  private extractTokenFromHeader(request: any): string | undefined {
    const [type, token] = request.headers.authorization?.split(' ') ?? [];
    return type === 'Bearer' ? token : undefined;
  }
}

// packages/shared-auth/src/decorators/roles.decorator.ts
import { SetMetadata } from '@nestjs/common';
import { UserType } from '@artistrsvp/shared-types';

export const ROLES_KEY = 'roles';
export const Roles = (...roles: UserType[]) => SetMetadata(ROLES_KEY, roles);

// packages/shared-auth/src/decorators/current-user.decorator.ts
import { createParamDecorator, ExecutionContext } from '@nestjs/common';

export const CurrentUser = createParamDecorator(
  (data: unknown, ctx: ExecutionContext) => {
    const request = ctx.switchToHttp().getRequest();
    return request.user;
  },
);
```

### Shared WhatsApp Package

```typescript
// packages/shared-whatsapp/src/whatsapp.service.ts
import { Injectable } from '@nestjs/common';
import axios, { AxiosInstance } from 'axios';

@Injectable()
export class WhatsAppService {
  private client: AxiosInstance;
  private phoneNumberId: string;

  constructor(private configService: ConfigService) {
    this.phoneNumberId = this.configService.get('WHATSAPP_PHONE_NUMBER_ID');
    this.client = axios.create({
      baseURL: `https://graph.facebook.com/v18.0/${this.phoneNumberId}`,
      headers: {
        'Authorization': `Bearer ${this.configService.get('WHATSAPP_ACCESS_TOKEN')}`,
        'Content-Type': 'application/json',
      },
    });
  }

  async sendOTP(to: string, otp: string): Promise<any> {
    return this.sendTemplate(to, 'otp_template', 'en', [
      {
        type: 'body',
        parameters: [{ type: 'text', text: otp }]
      }
    ]);
  }

  async sendTemplate(
    to: string, 
    templateName: string, 
    languageCode: string, 
    components?: any[]
  ): Promise<any> {
    return this.client.post('/messages', {
      messaging_product: 'whatsapp',
      to,
      type: 'template',
      template: {
        name: templateName,
        language: { code: languageCode },
        components,
      },
    });
  }

  async sendMessage(to: string, message: string): Promise<any> {
    return this.client.post('/messages', {
      messaging_product: 'whatsapp',
      recipient_type: 'individual',
      to,
      type: 'text',
      text: { body: message }
    });
  }
}
```

## Configuration Management

### AWS Systems Manager Parameter Store Integration

```typescript
// packages/shared-config/src/parameter-store.service.ts
import { SSMClient, GetParameterCommand, GetParametersCommand } from '@aws-sdk/client-ssm';
import { Injectable } from '@nestjs/common';

@Injectable()
export class ParameterStoreService {
  private client: SSMClient;
  private cache: Map<string, { value: any; expiry: number }> = new Map();

  constructor() {
    this.client = new SSMClient({ region: process.env.AWS_REGION || 'us-east-1' });
  }

  async getParameter<T>(name: string, decrypt = false): Promise<T> {
    const cached = this.cache.get(name);
    if (cached && cached.expiry > Date.now()) {
      return cached.value;
    }

    const command = new GetParameterCommand({
      Name: name,
      WithDecryption: decrypt
    });

    const response = await this.client.send(command);
    const value = JSON.parse(response.Parameter?.Value || '{}');
    
    this.cache.set(name, {
      value,
      expiry: Date.now() + 300000 // 5 minutes
    });

    return value;
  }

  async getParameters(names: string[]): Promise<Record<string, any>> {
    const command = new GetParametersCommand({
      Names: names,
      WithDecryption: true
    });

    const response = await this.client.send(command);
    const parameters: Record<string, any> = {};

    response.Parameters?.forEach(param => {
      parameters[param.Name!] = JSON.parse(param.Value!);
    });

    return parameters;
  }
}
```

### Environment Configuration Schema

```typescript
// packages/shared-config/src/config.schema.ts
import { z } from 'zod';

export const ConfigSchema = z.object({
  // Environment
  NODE_ENV: z.enum(['development', 'staging', 'production']),
  SERVICE_NAME: z.string(),
  
  // AWS
  AWS_REGION: z.string().default('us-east-1'),
  AWS_LAMBDA_FUNCTION_NAME: z.string().optional(),
  
  // Database
  DATABASE_URL: z.string().url(),
  DATABASE_POOL_MAX: z.number().default(1),
  
  // Redis
  REDIS_URL: z.string().url().optional(),
  
  // Auth
  JWT_SECRET: z.string().min(32),
  JWT_ACCESS_EXPIRES_IN: z.string().default('15m'),
  JWT_REFRESH_EXPIRES_IN: z.string().default('7d'),
  
  // WhatsApp
  WHATSAPP_ACCESS_TOKEN: z.string(),
  WHATSAPP_PHONE_NUMBER_ID: z.string(),
  WHATSAPP_WEBHOOK_VERIFY_TOKEN: z.string(),
  
  // AI Services
  OPENAI_API_KEY: z.string().optional(),
  ANTHROPIC_API_KEY: z.string().optional(),
  GOOGLE_AI_API_KEY: z.string().optional(),
  
  // Logging
  LOG_LEVEL: z.enum(['debug', 'info', 'warn', 'error']).default('info'),
});

export type Config = z.infer<typeof ConfigSchema>;
```

## Event-Driven Architecture

### Event Bus Service

```typescript
// packages/shared-events/src/event-bus.service.ts
import { EventBridgeClient, PutEventsCommand } from '@aws-sdk/client-eventbridge';
import { Injectable } from '@nestjs/common';
import { v4 as uuidv4 } from 'uuid';

export interface DomainEvent {
  eventId: string;
  eventType: string;
  source: string;
  timestamp: Date;
  version: string;
  correlationId: string;
  detail: any;
}

@Injectable()
export class EventBusService {
  private client: EventBridgeClient;
  private eventBusName: string;

  constructor() {
    this.client = new EventBridgeClient({ region: process.env.AWS_REGION });
    this.eventBusName = process.env.EVENT_BUS_NAME || 'artistrsvp-event-bus';
  }

  async publish<T extends DomainEvent>(event: T): Promise<void> {
    const command = new PutEventsCommand({
      Entries: [{
        Source: event.source,
        DetailType: event.eventType,
        Detail: JSON.stringify({
          ...event.detail,
          eventId: event.eventId,
          correlationId: event.correlationId,
          timestamp: event.timestamp
        }),
        EventBusName: this.eventBusName,
        Time: new Date()
      }]
    });

    await this.client.send(command);
  }

  createEvent<T>(source: string, eventType: string, detail: T): DomainEvent {
    return {
      eventId: uuidv4(),
      eventType,
      source,
      timestamp: new Date(),
      version: '1.0',
      correlationId: uuidv4(),
      detail
    };
  }
}
```

### Event Schemas

```typescript
// packages/shared-events/src/schemas/index.ts
import { z } from 'zod';

// Base event schema
export const BaseEventSchema = z.object({
  eventId: z.string().uuid(),
  eventType: z.string(),
  source: z.string(),
  timestamp: z.date(),
  version: z.string(),
  correlationId: z.string().uuid(),
});

// User events
export const UserCreatedEventSchema = BaseEventSchema.extend({
  eventType: z.literal('USER_CREATED'),
  source: z.literal('user-profile-service'),
  detail: z.object({
    userId: z.string().uuid(),
    email: z.string().email(),
    userType: z.enum(['fan', 'expert', 'artist']),
    subscriptionTier: z.enum(['basic', 'pro', 'pro_max']),
    whatsappNumber: z.string().optional(),
  })
});

// RFP events
export const RFPCreatedEventSchema = BaseEventSchema.extend({
  eventType: z.literal('RFP_CREATED'),
  source: z.literal('rfp-service'),
  detail: z.object({
    rfpId: z.string().uuid(),
    creatorId: z.string().uuid(),
    eventDate: z.date(),
    budgetRange: z.object({
      min: z.number(),
      max: z.number(),
      currency: z.string()
    }),
    rfpType: z.enum(['manual', 'automated', 'concierge'])
  })
});

// Payment events
export const PaymentCompletedEventSchema = BaseEventSchema.extend({
  eventType: z.literal('PAYMENT_COMPLETED'),
  source: z.literal('payment-service'),
  detail: z.object({
    paymentId: z.string().uuid(),
    userId: z.string().uuid(),
    amount: z.number(),
    currency: z.string(),
    paymentMethod: z.string(),
    metadata: z.record(z.any()).optional()
  })
});

// Type exports
export type UserCreatedEvent = z.infer<typeof UserCreatedEventSchema>;
export type RFPCreatedEvent = z.infer<typeof RFPCreatedEventSchema>;
export type PaymentCompletedEvent = z.infer<typeof PaymentCompletedEventSchema>;
```

## API Documentation Standards

### Swagger/OpenAPI Configuration

```typescript
// src/main.ts
import { SwaggerModule, DocumentBuilder } from '@nestjs/swagger';
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  // Swagger configuration
  const config = new DocumentBuilder()
    .setTitle('ArtistRSVP API')
    .setDescription(`
      ArtistRSVP.com - AI-First, WhatsApp-First Events Platform
      
      ## Features
      - 🎵 Artist discovery and booking
      - 💬 WhatsApp-integrated workflows
      - 🤖 AI-powered recommendations
      - 📱 Real-time messaging
      - 💳 Multi-gateway payments
      - 🎓 Learning management system
      - 🏆 Rewards and gamification
      
      ## Authentication
      All endpoints require JWT authentication unless marked as public.
      Include the token in the Authorization header: Bearer <token>
      
      ## Rate Limiting
      - Free tier: 100 requests/minute
      - Pro tier: 500 requests/minute
      - Pro Max tier: 2000 requests/minute
      
      ## WhatsApp Integration
      Many endpoints support WhatsApp-based interactions.
      Look for the WhatsApp tag on endpoints.
      
      ## AI Features
      Endpoints marked with AI tag include intelligent processing.
    `)
    .setVersion('1.0.0')
    .setContact('ArtistRSVP Team', 'https://artistrsvp.com', 'api@artistrsvp.com')
    .setLicense('Proprietary', 'https://artistrsvp.com/terms')
    .addBearerAuth(
      {
        type: 'http',
        scheme: 'bearer',
        bearerFormat: 'JWT',
        description: 'Enter JWT token',
      },
      'JWT-auth'
    )
    .addServer('https://api.artistrsvp.com', 'Production')
    .addServer('https://staging-api.artistrsvp.com', 'Staging')
    .addServer('http://localhost:3000', 'Development')
    .addTag('Authentication', 'WhatsApp-first authentication')
    .addTag('Users', 'User profiles and management')
    .addTag('Events', 'Event creation and management')
    .addTag('RFP', 'Request for Proposal marketplace')
    .addTag('AI', 'AI-powered services')
    .addTag('WhatsApp', 'WhatsApp integration')
    .addTag('Payments', 'Payment processing')
    .build();

  const document = SwaggerModule.createDocument(app, config);
  SwaggerModule.setup('api/docs', app, document, {
    customSiteTitle: 'ArtistRSVP API Documentation',
    customfavIcon: '/favicon.ico',
    customCssUrl: '/swagger-custom.css',
  });

  await app.listen(3000);
}

bootstrap();
```

### API Endpoint Documentation Example

```typescript
// Example controller with Swagger decorators
import { ApiTags, ApiOperation, ApiResponse, ApiBearerAuth, ApiQuery } from '@nestjs/swagger';

@ApiTags('RFP')
@ApiBearerAuth('JWT-auth')
@Controller('rfp')
export class RFPController {
  @Post('create')
  @ApiOperation({ 
    summary: 'Create new RFP',
    description: 'Creates a new Request for Proposal with AI-powered artist matching'
  })
  @ApiResponse({ 
    status: 201, 
    description: 'RFP created successfully',
    type: RFPResponseDto 
  })
  @ApiResponse({ 
    status: 400, 
    description: 'Invalid input data' 
  })
  async createRFP(
    @Body() createRFPDto: CreateRFPDto,
    @CurrentUser() user: UserPayload
  ): Promise<RFPResponseDto> {
    return this.rfpService.create(createRFPDto, user.id);
  }

  @Get('search')
  @ApiOperation({ 
    summary: 'Search RFPs',
    description: 'Search RFPs with AI-powered relevance ranking'
  })
  @ApiQuery({ name: 'q', required: false, description: 'Search query' })
  @ApiQuery({ name: 'eventType', required: false, enum: EventType })
  @ApiQuery({ name: 'minBudget', required: false, type: Number })
  @ApiQuery({ name: 'maxBudget', required: false, type: Number })
  @ApiResponse({ 
    status: 200, 
    description: 'Search results',
    type: [RFPSearchResultDto] 
  })
  async searchRFPs(
    @Query() searchParams: RFPSearchDto
  ): Promise<RFPSearchResultDto[]> {
    return this.rfpService.search(searchParams);
  }
}
```

## Error Handling and Logging

### Global Exception Filter

```typescript
// packages/shared-utils/src/filters/global-exception.filter.ts
import { 
  ExceptionFilter, 
  Catch, 
  ArgumentsHost, 
  HttpException, 
  HttpStatus,
  Logger
} from '@nestjs/common';
import { Request, Response } from 'express';

@Catch()
export class GlobalExceptionFilter implements ExceptionFilter {
  private readonly logger = new Logger(GlobalExceptionFilter.name);

  catch(exception: unknown, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const request = ctx.getRequest<Request>();

    const status = exception instanceof HttpException
      ? exception.getStatus()
      : HttpStatus.INTERNAL_SERVER_ERROR;

    const message = exception instanceof HttpException
      ? exception.getResponse()
      : 'Internal server error';

    const errorResponse = {
      statusCode: status,
      timestamp: new Date().toISOString(),
      path: request.url,
      method: request.method,
      correlationId: request.headers['x-correlation-id'] || 'unknown',
      message: typeof message === 'string' ? message : (message as any).message,
      error: typeof message === 'object' ? message : undefined,
      stack: process.env.NODE_ENV === 'development' && exception instanceof Error 
        ? exception.stack 
        : undefined
    };

    this.logger.error(
      `${request.method} ${request.url} - ${status} - ${JSON.stringify(errorResponse)}`,
      exception instanceof Error ? exception.stack : undefined
    );

    response.status(status).json(errorResponse);
  }
}
```

### Structured Logging Service

```typescript
// packages/shared-logging/src/logger.service.ts
import { Injectable, LoggerService } from '@nestjs/common';
import winston from 'winston';
import { CloudWatchTransport } from 'winston-cloudwatch';

@Injectable()
export class StructuredLogger implements LoggerService {
  private logger: winston.Logger;

  constructor(serviceName: string) {
    const transports: winston.transport[] = [
      new winston.transports.Console({
        format: winston.format.combine(
          winston.format.colorize(),
          winston.format.timestamp(),
          winston.format.printf(({ timestamp, level, message, ...meta }) => {
            return `${timestamp} [${level}]: ${message} ${Object.keys(meta).length ? JSON.stringify(meta) : ''}`;
          })
        )
      })
    ];

    // Add CloudWatch transport in production
    if (process.env.NODE_ENV === 'production') {
      transports.push(
        new CloudWatchTransport({
          logGroupName: `/aws/lambda/${serviceName}`,
          logStreamName: new Date().toISOString().split('T')[0],
          awsRegion: process.env.AWS_REGION,
          messageFormatter: ({ level, message, ...meta }) => {
            return JSON.stringify({ level, message, ...meta, service: serviceName });
          }
        })
      );
    }

    this.logger = winston.createLogger({
      level: process.env.LOG_LEVEL || 'info',
      format: winston.format.json(),
      defaultMeta: { service: serviceName },
      transports
    });
  }

  log(message: string, context?: any) {
    this.logger.info(message, { context });
  }

  error(message: string, trace?: string, context?: any) {
    this.logger.error(message, { trace, context });
  }

  warn(message: string, context?: any) {
    this.logger.warn(message, { context });
  }

  debug(message: string, context?: any) {
    this.logger.debug(message, { context });
  }

  verbose(message: string, context?: any) {
    this.logger.verbose(message, { context });
  }
}
```

## Frontend Development Guidelines

### Next.js 15 Application Structure

```
apps/frontend/web-app/
├── src/
│   ├── app/                          # Next.js 15 App Router
│   │   ├── (auth)/                   # Auth group routes
│   │   │   ├── login/
│   │   │   │   ├── page.tsx
│   │   │   │   └── layout.tsx
│   │   │   ├── register/
│   │   │   └── forgot-password/
│   │   ├── (dashboard)/              # Protected routes
│   │   │   ├── layout.tsx           # Dashboard layout
│   │   │   ├── events/
│   │   │   ├── artists/
│   │   │   ├── rfp/
│   │   │   └── profile/
│   │   ├── (public)/                 # Public routes
│   │   │   ├── explore/
│   │   │   ├── about/
│   │   │   └── contact/
│   │   ├── api/                      # API routes
│   │   │   └── webhooks/
│   │   │       └── whatsapp/
│   │   ├── layout.tsx                # Root layout
│   │   ├── page.tsx                  # Home page
│   │   └── global-error.tsx          # Global error boundary
│   ├── components/                   # Component library
│   │   ├── ui/                       # Base UI components (shadcn/ui)
│   │   │   ├── button.tsx
│   │   │   ├── card.tsx
│   │   │   └── ...
│   │   ├── features/                 # Feature-specific components
│   │   │   ├── events/
│   │   │   ├── rfp/
│   │   │   └── artists/
│   │   ├── forms/                    # Form components
│   │   │   ├── EventForm.tsx
│   │   │   └── RFPForm.tsx
│   │   └── layouts/                  # Layout components
│   │       ├── Header.tsx
│   │       └── Footer.tsx
│   ├── hooks/                        # Custom React hooks
│   │   ├── useAuth.ts
│   │   ├── useWhatsApp.ts
│   │   └── useAI.ts
│   ├── lib/                          # Utility functions
│   │   ├── api-client.ts
│   │   ├── utils.ts
│   │   └── constants.ts
│   ├── services/                     # API service layer
│   │   ├── auth.service.ts
│   │   ├── event.service.ts
│   │   └── rfp.service.ts
│   ├── stores/                       # Zustand stores
│   │   ├── auth.store.ts
│   │   └── app.store.ts
│   ├── styles/                       # Global styles
│   │   └── globals.css
│   ├── types/                        # TypeScript types
│   │   └── index.ts
│   └── utils/                        # Helper functions
│       ├── format.ts
│       └── validation.ts
├── public/                           # Static assets
├── .storybook/                       # Storybook configuration
└── tests/                            # Test files
```

### API Client Architecture

```typescript
// packages/api-client/src/base-client.ts
import axios, { AxiosInstance, AxiosRequestConfig, AxiosError } from 'axios';
import { getSession } from 'next-auth/react';

export interface ApiError {
  message: string;
  statusCode: number;
  error?: any;
}

export class BaseAPIClient {
  protected client: AxiosInstance;
  private refreshPromise: Promise<string> | null = null;
  
  constructor(baseURL: string) {
    this.client = axios.create({
      baseURL: process.env.NEXT_PUBLIC_API_URL + baseURL,
      timeout: 30000,
      headers: {
        'Content-Type': 'application/json',
      }
    });

    this.setupInterceptors();
  }

  private setupInterceptors() {
    // Request interceptor for authentication
    this.client.interceptors.request.use(
      async (config) => {
        const session = await getSession();
        if (session?.accessToken) {
          config.headers.Authorization = `Bearer ${session.accessToken}`;
        }
        
        // Add correlation ID for tracing
        config.headers['X-Correlation-ID'] = this.generateCorrelationId();
        
        return config;
      },
      (error) => Promise.reject(error)
    );

    // Response interceptor for error handling and token refresh
    this.client.interceptors.response.use(
      (response) => response,
      async (error: AxiosError) => {
        const originalRequest = error.config as AxiosRequestConfig & { _retry?: boolean };
        
        if (error.response?.status === 401 && !originalRequest._retry) {
          originalRequest._retry = true;
          
          try {
            // Prevent multiple refresh calls
            if (!this.refreshPromise) {
              this.refreshPromise = this.refreshToken();
            }
            
            const newToken = await this.refreshPromise;
            this.refreshPromise = null;
            
            if (originalRequest.headers) {
              originalRequest.headers.Authorization = `Bearer ${newToken}`;
            }
            
            return this.client(originalRequest);
          } catch (refreshError) {
            // Redirect to login on refresh failure
            window.location.href = '/login';
            return Promise.reject(refreshError);
          }
        }
        
        return Promise.reject(this.handleError(error));
      }
    );
  }

  private async refreshToken(): Promise<string> {
    const response = await axios.post('/api/auth/refresh');
    return response.data.accessToken;
  }

  private handleError(error: AxiosError): ApiError {
    if (error.response) {
      return {
        message: (error.response.data as any)?.message || 'An error occurred',
        statusCode: error.response.status,
        error: error.response.data
      };
    }
    
    if (error.request) {
      return {
        message: 'No response from server',
        statusCode: 0
      };
    }
    
    return {
      message: error.message || 'An unknown error occurred',
      statusCode: 0
    };
  }

  private generateCorrelationId(): string {
    return `${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;
  }
}

// packages/api-client/src/services/rfp.service.ts
import { BaseAPIClient } from '../base-client';
import { RFP, CreateRFPDto, RFPResponse } from '@artistrsvp/shared-types';

export class RFPAPIClient extends BaseAPIClient {
  constructor() {
    super('/api/v1/rfp');
  }

  async createRFP(data: CreateRFPDto): Promise<RFP> {
    const response = await this.client.post<RFP>('/create', data);
    return response.data;
  }

  async searchRFPs(params: any): Promise<RFP[]> {
    const response = await this.client.get<RFP[]>('/search', { params });
    return response.data;
  }

  async getRFPDetails(rfpId: string): Promise<RFP> {
    const response = await this.client.get<RFP>(`/${rfpId}`);
    return response.data;
  }

  async submitResponse(rfpId: string, data: any): Promise<RFPResponse> {
    const response = await this.client.post<RFPResponse>(`/${rfpId}/respond`, data);
    return response.data;
  }

  async submitWhatsAppResponse(rfpId: string, message: string): Promise<RFPResponse> {
    const response = await this.client.post<RFPResponse>('/whatsapp/respond', {
      rfpId,
      message
    });
    return response.data;
  }
}
```

### React Hook Form with Yup Validation

```typescript
// components/forms/RFPForm.tsx
import { useForm } from 'react-hook-form';
import { yupResolver } from '@hookform/resolvers/yup';
import * as yup from 'yup';
import { useTranslation } from 'react-i18next';

const rfpSchema = yup.object({
  title: yup
    .string()
    .required('Title is required')
    .min(10, 'Title must be at least 10 characters')
    .max(100, 'Title must not exceed 100 characters'),
  
  description: yup
    .string()
    .required('Description is required')
    .min(50, 'Description must be at least 50 characters')
    .max(5000, 'Description must not exceed 5000 characters'),
  
  eventDate: yup
    .date()
    .required('Event date is required')
    .min(new Date(), 'Event date must be in the future'),
  
  budgetRange: yup.object({
    min: yup
      .number()
      .required('Minimum budget is required')
      .positive('Budget must be positive')
      .integer(),
    max: yup
      .number()
      .required('Maximum budget is required')
      .positive('Budget must be positive')
      .integer()
      .min(yup.ref('min'), 'Maximum budget must be greater than minimum'),
    currency: yup
      .string()
      .required('Currency is required')
      .oneOf(['USD', 'EUR', 'INR', 'GBP'])
  }),
  
  location: yup.object({
    address: yup.string().required('Address is required'),
    city: yup.string().required('City is required'),
    state: yup.string().required('State is required'),
    country: yup.string().required('Country is required'),
    postalCode: yup.string().required('Postal code is required'),
    coordinates: yup.object({
      lat: yup.number().required().min(-90).max(90),
      lng: yup.number().required().min(-180).max(180)
    })
  }),
  
  requirements: yup.array().of(
    yup.object({
      type: yup.string().required(),
      description: yup.string().required()
    })
  ).min(1, 'At least one requirement is needed'),
  
  rfpType: yup
    .string()
    .required()
    .oneOf(['manual', 'automated', 'concierge']),
  
  aiMatchingEnabled: yup.boolean(),
  whatsappNotifications: yup.boolean()
});

type RFPFormData = yup.InferType<typeof rfpSchema>;

export function RFPForm({ onSubmit }: { onSubmit: (data: RFPFormData) => void }) {
  const { t } = useTranslation();
  const [isLoadingLocation, setIsLoadingLocation] = useState(false);
  
  const {
    register,
    handleSubmit,
    control,
    formState: { errors, isSubmitting },
    watch,
    setValue,
    trigger
  } = useForm<RFPFormData>({
    resolver: yupResolver(rfpSchema),
    defaultValues: {
      rfpType: 'automated',
      aiMatchingEnabled: true,
      whatsappNotifications: true,
      budgetRange: {
        currency: 'USD'
      }
    }
  });

  const rfpType = watch('rfpType');

  // Auto-detect location
  const detectLocation = async () => {
    setIsLoadingLocation(true);
    try {
      const position = await getCurrentPosition();
      const address = await reverseGeocode(position.coords);
      
      setValue('location', {
        address: address.formatted,
        city: address.city,
        state: address.state,
        country: address.country,
        postalCode: address.postalCode,
        coordinates: {
          lat: position.coords.latitude,
          lng: position.coords.longitude
        }
      });
      
      await trigger('location');
    } catch (error) {
      toast.error(t('rfp.location.detection_failed'));
    } finally {
      setIsLoadingLocation(false);
    }
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)} className="space-y-6">
      <Card>
        <CardHeader>
          <CardTitle>{t('rfp.create.title')}</CardTitle>
          <CardDescription>{t('rfp.create.description')}</CardDescription>
        </CardHeader>
        
        <CardContent className="space-y-4">
          {/* Title Field */}
          <div className="space-y-2">
            <Label htmlFor="title">{t('rfp.fields.title')}</Label>
            <Input
              id="title"
              {...register('title')}
              placeholder={t('rfp.placeholders.title')}
              className={errors.title ? 'border-red-500' : ''}
            />
            {errors.title && (
              <p className="text-sm text-red-500">{errors.title.message}</p>
            )}
          </div>

          {/* Description Field */}
          <div className="space-y-2">
            <Label htmlFor="description">{t('rfp.fields.description')}</Label>
            <Textarea
              id="description"
              {...register('description')}
              placeholder={t('rfp.placeholders.description')}
              rows={5}
              className={errors.description ? 'border-red-500' : ''}
            />
            {errors.description && (
              <p className="text-sm text-red-500">{errors.description.message}</p>
            )}
          </div>

          {/* Event Date */}
          <div className="space-y-2">
            <Label htmlFor="eventDate">{t('rfp.fields.eventDate')}</Label>
            <Controller
              name="eventDate"
              control={control}
              render={({ field }) => (
                <DatePicker
                  selected={field.value}
                  onChange={field.onChange}
                  minDate={new Date()}
                  className={errors.eventDate ? 'border-red-500' : ''}
                  placeholderText={t('rfp.placeholders.eventDate')}
                />
              )}
            />
            {errors.eventDate && (
              <p className="text-sm text-red-500">{errors.eventDate.message}</p>
            )}
          </div>

          {/* Budget Range */}
          <div className="grid grid-cols-3 gap-4">
            <div className="space-y-2">
              <Label htmlFor="budgetMin">{t('rfp.fields.budgetMin')}</Label>
              <Input
                id="budgetMin"
                type="number"
                {...register('budgetRange.min', { valueAsNumber: true })}
                placeholder="1000"
                className={errors.budgetRange?.min ? 'border-red-500' : ''}
              />
              {errors.budgetRange?.min && (
                <p className="text-sm text-red-500">{errors.budgetRange.min.message}</p>
              )}
            </div>
            
            <div className="space-y-2">
              <Label htmlFor="budgetMax">{t('rfp.fields.budgetMax')}</Label>
              <Input
                id="budgetMax"
                type="number"
                {...register('budgetRange.max', { valueAsNumber: true })}
                placeholder="5000"
                className={errors.budgetRange?.max ? 'border-red-500' : ''}
              />
              {errors.budgetRange?.max && (
                <p className="text-sm text-red-500">{errors.budgetRange.max.message}</p>
              )}
            </div>
            
            <div className="space-y-2">
              <Label htmlFor="currency">{t('rfp.fields.currency')}</Label>
              <Select
                value={watch('budgetRange.currency')}
                onValueChange={(value) => setValue('budgetRange.currency', value)}
              >
                <SelectTrigger>
                  <SelectValue />
                </SelectTrigger>
                <SelectContent>
                  <SelectItem value="USD">USD</SelectItem>
                  <SelectItem value="EUR">EUR</SelectItem>
                  <SelectItem value="INR">INR</SelectItem>
                  <SelectItem value="GBP">GBP</SelectItem>
                </SelectContent>
              </Select>
            </div>
          </div>

          {/* Location */}
          <div className="space-y-2">
            <div className="flex justify-between items-center">
              <Label>{t('rfp.fields.location')}</Label>
              <Button
                type="button"
                variant="outline"
                size="sm"
                onClick={detectLocation}
                disabled={isLoadingLocation}
              >
                {isLoadingLocation ? (
                  <Loader2 className="h-4 w-4 animate-spin" />
                ) : (
                  <MapPin className="h-4 w-4" />
                )}
                {t('rfp.actions.detectLocation')}
              </Button>
            </div>
            
            <div className="grid grid-cols-2 gap-4">
              <Input
                {...register('location.address')}
                placeholder={t('rfp.placeholders.address')}
                className={errors.location?.address ? 'border-red-500' : ''}
              />
              <Input
                {...register('location.city')}
                placeholder={t('rfp.placeholders.city')}
                className={errors.location?.city ? 'border-red-500' : ''}
              />
              <Input
                {...register('location.state')}
                placeholder={t('rfp.placeholders.state')}
                className={errors.location?.state ? 'border-red-500' : ''}
              />
              <Input
                {...register('location.country')}
                placeholder={t('rfp.placeholders.country')}
                className={errors.location?.country ? 'border-red-500' : ''}
              />
            </div>
          </div>

          {/* RFP Type */}
          <div className="space-y-2">
            <Label>{t('rfp.fields.rfpType')}</Label>
            <RadioGroup
              value={rfpType}
              onValueChange={(value) => setValue('rfpType', value as any)}
            >
              <div className="flex items-center space-x-2">
                <RadioGroupItem value="manual" id="manual" />
                <Label htmlFor="manual" className="font-normal">
                  {t('rfp.types.manual')}
                  <span className="text-sm text-gray-500 block">
                    {t('rfp.types.manual_description')}
                  </span>
                </Label>
              </div>
              <div className="flex items-center space-x-2">
                <RadioGroupItem value="automated" id="automated" />
                <Label htmlFor="automated" className="font-normal">
                  {t('rfp.types.automated')}
                  <span className="text-sm text-gray-500 block">
                    {t('rfp.types.automated_description')}
                  </span>
                </Label>
              </div>
              <div className="flex items-center space-x-2">
                <RadioGroupItem value="concierge" id="concierge" />
                <Label htmlFor="concierge" className="font-normal">
                  {t('rfp.types.concierge')}
                  <Badge className="ml-2" variant="secondary">Pro</Badge>
                  <span className="text-sm text-gray-500 block">
                    {t('rfp.types.concierge_description')}
                  </span>
                </Label>
              </div>
            </RadioGroup>
          </div>

          {/* AI & WhatsApp Options */}
          <div className="space-y-4">
            <div className="flex items-center space-x-2">
              <Controller
                name="aiMatchingEnabled"
                control={control}
                render={({ field }) => (
                  <Switch
                    id="aiMatching"
                    checked={field.value}
                    onCheckedChange={field.onChange}
                  />
                )}
              />
              <Label htmlFor="aiMatching" className="font-normal">
                {t('rfp.options.aiMatching')}
                <span className="text-sm text-gray-500 block">
                  {t('rfp.options.aiMatching_description')}
                </span>
              </Label>
            </div>
            
            <div className="flex items-center space-x-2">
              <Controller
                name="whatsappNotifications"
                control={control}
                render={({ field }) => (
                  <Switch
                    id="whatsappNotifications"
                    checked={field.value}
                    onCheckedChange={field.onChange}
                  />
                )}
              />
              <Label htmlFor="whatsappNotifications" className="font-normal">
                {t('rfp.options.whatsappNotifications')}
                <span className="text-sm text-gray-500 block">
                  {t('rfp.options.whatsappNotifications_description')}
                </span>
              </Label>
            </div>
          </div>
        </CardContent>
        
        <CardFooter className="flex justify-between">
          <Button
            type="button"
            variant="outline"
            onClick={() => window.history.back()}
          >
            {t('common.cancel')}
          </Button>
          
          <Button
            type="submit"
            disabled={isSubmitting}
            className="min-w-[120px]"
          >
            {isSubmitting ? (
              <>
                <Loader2 className="mr-2 h-4 w-4 animate-spin" />
                {t('common.creating')}
              </>
            ) : (
              t('rfp.actions.create')
            )}
          </Button>
        </CardFooter>
      </Card>
    </form>
  );
}
```

### Internationalization with i18next

```typescript
// lib/i18n/config.ts
import i18n from 'i18next';
import { initReactI18next } from 'react-i18next';
import LanguageDetector from 'i18next-browser-languagedetector';
import Backend from 'i18next-http-backend';

// Import translations
import enCommon from './locales/en/common.json';
import enRFP from './locales/en/rfp.json';
import enEvents from './locales/en/events.json';
import esCommon from './locales/es/common.json';
import esRFP from './locales/es/rfp.json';
import esEvents from './locales/es/events.json';
import hiCommon from './locales/hi/common.json';
import hiRFP from './locales/hi/rfp.json';
import hiEvents from './locales/hi/events.json';

export const defaultNS = 'common';
export const resources = {
  en: {
    common: enCommon,
    rfp: enRFP,
    events: enEvents,
  },
  es: {
    common: esCommon,
    rfp: esRFP,
    events: esEvents,
  },
  hi: {
    common: hiCommon,
    rfp: hiRFP,
    events: hiEvents,
  },
} as const;

i18n
  .use(Backend)
  .use(LanguageDetector)
  .use(initReactI18next)
  .init({
    fallbackLng: 'en',
    debug: process.env.NODE_ENV === 'development',
    
    defaultNS,
    ns: ['common', 'rfp', 'events', 'artists', 'auth'],
    
    interpolation: {
      escapeValue: false, // React already escapes values
    },
    
    detection: {
      order: ['querystring', 'cookie', 'localStorage', 'navigator', 'htmlTag'],
      caches: ['localStorage', 'cookie'],
    },
    
    backend: {
      loadPath: '/locales/{{lng}}/{{ns}}.json',
    },
    
    react: {
      useSuspense: false,
    },
  });

export default i18n;

// lib/i18n/locales/en/rfp.json
{
  "create": {
    "title": "Create New RFP",
    "description": "Post your requirements and let AI match you with perfect artists"
  },
  "fields": {
    "title": "RFP Title",
    "description": "Description",
    "eventDate": "Event Date",
    "budgetMin": "Min Budget",
    "budgetMax": "Max Budget",
    "currency": "Currency",
    "location": "Location",
    "rfpType": "RFP Type"
  },
  "placeholders": {
    "title": "e.g., Wedding Reception - Need Live Band",
    "description": "Describe your event requirements in detail...",
    "eventDate": "Select event date",
    "address": "Street address",
    "city": "City",
    "state": "State/Province",
    "country": "Country"
  },
  "types": {
    "manual": "Manual",
    "manual_description": "Review and select artists yourself",
    "automated": "Automated",
    "automated_description": "AI automatically matches and notifies artists",
    "concierge": "Concierge",
    "concierge_description": "Our experts handle everything for you"
  },
  "options": {
    "aiMatching": "Enable AI Matching",
    "aiMatching_description": "Use AI to find the best artists for your event",
    "whatsappNotifications": "WhatsApp Notifications",
    "whatsappNotifications_description": "Receive updates via WhatsApp"
  },
  "actions": {
    "create": "Create RFP",
    "detectLocation": "Detect Location"
  },
  "location": {
    "detection_failed": "Could not detect your location. Please enter manually."
  }
}

// Usage in components
import { useTranslation } from 'react-i18next';

export function LanguageSelector() {
  const { i18n } = useTranslation();
  
  return (
    <Select
      value={i18n.language}
      onValueChange={(lng) => i18n.changeLanguage(lng)}
    >
      <SelectTrigger className="w-[140px]">
        <Globe className="h-4 w-4 mr-2" />
        <SelectValue />
      </SelectTrigger>
      <SelectContent>
        <SelectItem value="en">
          <span className="flex items-center">
            🇺🇸 English
          </span>
        </SelectItem>
        <SelectItem value="es">
          <span className="flex items-center">
            🇪🇸 Español
          </span>
        </SelectItem>
        <SelectItem value="hi">
          <span className="flex items-center">
            🇮🇳 हिन्दी
          </span>
        </SelectItem>
      </SelectContent>
    </Select>
  );
}
```

### Tailwind CSS with shadcn/ui

```typescript
// tailwind.config.ts
import type { Config } from 'tailwindcss';

const config: Config = {
  darkMode: ['class'],
  content: [
    './pages/**/*.{ts,tsx}',
    './components/**/*.{ts,tsx}',
    './app/**/*.{ts,tsx}',
    './src/**/*.{ts,tsx}',
  ],
  theme: {
    container: {
      center: true,
      padding: '2rem',
      screens: {
        '2xl': '1400px',
      },
    },
    extend: {
      colors: {
        border: 'hsl(var(--border))',
        input: 'hsl(var(--input))',
        ring: 'hsl(var(--ring))',
        background: 'hsl(var(--background))',
        foreground: 'hsl(var(--foreground))',
        primary: {
          DEFAULT: 'hsl(var(--primary))',
          foreground: 'hsl(var(--primary-foreground))',
        },
        secondary: {
          DEFAULT: 'hsl(var(--secondary))',
          foreground: 'hsl(var(--secondary-foreground))',
        },
        destructive: {
          DEFAULT: 'hsl(var(--destructive))',
          foreground: 'hsl(var(--destructive-foreground))',
        },
        muted: {
          DEFAULT: 'hsl(var(--muted))',
          foreground: 'hsl(var(--muted-foreground))',
        },
        accent: {
          DEFAULT: 'hsl(var(--accent))',
          foreground: 'hsl(var(--accent-foreground))',
        },
        popover: {
          DEFAULT: 'hsl(var(--popover))',
          foreground: 'hsl(var(--popover-foreground))',
        },
        card: {
          DEFAULT: 'hsl(var(--card))',
          foreground: 'hsl(var(--card-foreground))',
        },
        whatsapp: {
          DEFAULT: '#25D366',
          dark: '#128C7E',
          light: '#DCF8C6',
        },
      },
      borderRadius: {
        lg: 'var(--radius)',
        md: 'calc(var(--radius) - 2px)',
        sm: 'calc(var(--radius) - 4px)',
      },
      keyframes: {
        'accordion-down': {
          from: { height: '0' },
          to: { height: 'var(--radix-accordion-content-height)' },
        },
        'accordion-up': {
          from: { height: 'var(--radix-accordion-content-height)' },
          to: { height: '0' },
        },
        'slide-in': {
          from: { transform: 'translateX(-100%)' },
          to: { transform: 'translateX(0)' },
        },
        'fade-in': {
          from: { opacity: '0' },
          to: { opacity: '1' },
        },
      },
      animation: {
        'accordion-down': 'accordion-down 0.2s ease-out',
        'accordion-up': 'accordion-up 0.2s ease-out',
        'slide-in': 'slide-in 0.3s ease-out',
        'fade-in': 'fade-in 0.4s ease-out',
      },
      fontFamily: {
        sans: ['var(--font-inter)', 'system-ui', 'sans-serif'],
        display: ['var(--font-cal-sans)', 'system-ui', 'sans-serif'],
      },
    },
  },
  plugins: [
    require('tailwindcss-animate'),
    require('@tailwindcss/typography'),
    require('@tailwindcss/forms'),
  ],
};

export default config;

// styles/globals.css
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  :root {
    --background: 0 0% 100%;
    --foreground: 222.2 84% 4.9%;
    --card: 0 0% 100%;
    --card-foreground: 222.2 84% 4.9%;
    --popover: 0 0% 100%;
    --popover-foreground: 222.2 84% 4.9%;
    --primary: 222.2 47.4% 11.2%;
    --primary-foreground: 210 40% 98%;
    --secondary: 210 40% 96.1%;
    --secondary-foreground: 222.2 47.4% 11.2%;
    --muted: 210 40% 96.1%;
    --muted-foreground: 215.4 16.3% 46.9%;
    --accent: 210 40% 96.1%;
    --accent-foreground: 222.2 47.4% 11.2%;
    --destructive: 0 84.2% 60.2%;
    --destructive-foreground: 210 40% 98%;
    --border: 214.3 31.8% 91.4%;
    --input: 214.3 31.8% 91.4%;
    --ring: 222.2 84% 4.9%;
    --radius: 0.5rem;
  }

  .dark {
    --background: 222.2 84% 4.9%;
    --foreground: 210 40% 98%;
    --card: 222.2 84% 4.9%;
    --card-foreground: 210 40% 98%;
    --popover: 222.2 84% 4.9%;
    --popover-foreground: 210 40% 98%;
    --primary: 210 40% 98%;
    --primary-foreground: 222.2 47.4% 11.2%;
    --secondary: 217.2 32.6% 17.5%;
    --secondary-foreground: 210 40% 98%;
    --muted: 217.2 32.6% 17.5%;
    --muted-foreground: 215 20.2% 65.1%;
    --accent: 217.2 32.6% 17.5%;
    --accent-foreground: 210 40% 98%;
    --destructive: 0 62.8% 30.6%;
    --destructive-foreground: 210 40% 98%;
    --border: 217.2 32.6% 17.5%;
    --input: 217.2 32.6% 17.5%;
    --ring: 212.7 26.8% 83.9%;
  }
}

@layer components {
  .gradient-text {
    @apply bg-gradient-to-r from-primary to-purple-600 bg-clip-text text-transparent;
  }

  .glass-effect {
    @apply bg-white/80 backdrop-blur-md border border-white/20 shadow-xl;
  }

  .hover-scale {
    @apply transition-transform duration-200 hover:scale-105;
  }

  .whatsapp-button {
    @apply bg-whatsapp hover:bg-whatsapp-dark text-white font-medium py-2 px-4 rounded-lg flex items-center gap-2 transition-colors;
  }
}

// Custom shadcn/ui component example
// components/ui/button.tsx
import * as React from 'react';
import { Slot } from '@radix-ui/react-slot';
import { cva, type VariantProps } from 'class-variance-authority';
import { cn } from '@/lib/utils';

const buttonVariants = cva(
  'inline-flex items-center justify-center rounded-md text-sm font-medium ring-offset-background transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:pointer-events-none disabled:opacity-50',
  {
    variants: {
      variant: {
        default: 'bg-primary text-primary-foreground hover:bg-primary/90',
        destructive:
          'bg-destructive text-destructive-foreground hover:bg-destructive/90',
        outline:
          'border border-input bg-background hover:bg-accent hover:text-accent-foreground',
        secondary:
          'bg-secondary text-secondary-foreground hover:bg-secondary/80',
        ghost: 'hover:bg-accent hover:text-accent-foreground',
        link: 'text-primary underline-offset-4 hover:underline',
        whatsapp: 'bg-whatsapp text-white hover:bg-whatsapp-dark',
      },
      size: {
        default: 'h-10 px-4 py-2',
        sm: 'h-9 rounded-md px-3',
        lg: 'h-11 rounded-md px-8',
        icon: 'h-10 w-10',
      },
    },
    defaultVariants: {
      variant: 'default',
      size: 'default',
    },
  }
);

export interface ButtonProps
  extends React.ButtonHTMLAttributes<HTMLButtonElement>,
    VariantProps<typeof buttonVariants> {
  asChild?: boolean;
}

const Button = React.forwardRef<HTMLButtonElement, ButtonProps>(
  ({ className, variant, size, asChild = false, ...props }, ref) => {
    const Comp = asChild ? Slot : 'button';
    return (
      <Comp
        className={cn(buttonVariants({ variant, size, className }))}
        ref={ref}
        {...props}
      />
    );
  }
);
Button.displayName = 'Button';

export { Button, buttonVariants };
```

### Storybook Configuration

```typescript
// .storybook/main.ts
import type { StorybookConfig } from '@storybook/nextjs';

const config: StorybookConfig = {
  stories: [
    '../src/**/*.stories.@(js|jsx|ts|tsx|mdx)',
    '../src/**/*.story.@(js|jsx|ts|tsx|mdx)',
  ],
  addons: [
    '@storybook/addon-links',
    '@storybook/addon-essentials',
    '@storybook/addon-onboarding',
    '@storybook/addon-interactions',
    '@storybook/addon-a11y',
    '@storybook/addon-coverage',
    'storybook-addon-designs',
    'storybook-dark-mode',
  ],
  framework: {
    name: '@storybook/nextjs',
    options: {},
  },
  docs: {
    autodocs: 'tag',
  },
  staticDirs: ['../public'],
  features: {
    experimentalRSC: true,
  },
};

export default config;

// .storybook/preview.tsx
import type { Preview } from '@storybook/react';
import { ThemeProvider } from 'next-themes';
import '../src/styles/globals.css';
import { I18nextProvider } from 'react-i18next';
import i18n from '../src/lib/i18n/config';

const preview: Preview = {
  parameters: {
    actions: { argTypesRegex: '^on[A-Z].*' },
    controls: {
      matchers: {
        color: /(background|color)$/i,
        date: /Date$/,
      },
    },
    nextjs: {
      appDirectory: true,
    },
    darkMode: {
      darkClass: 'dark',
      lightClass: 'light',
      stylePreview: true,
    },
  },
  decorators: [
    (Story) => (
      <I18nextProvider i18n={i18n}>
        <ThemeProvider attribute="class" defaultTheme="system" enableSystem>
          <Story />
        </ThemeProvider>
      </I18nextProvider>
    ),
  ],
};

export default preview;

// Example component story
// components/ui/Button/Button.stories.tsx
import type { Meta, StoryObj } from '@storybook/react';
import { Button } from './Button';
import { Mail, Send } from 'lucide-react';

const meta = {
  title: 'UI/Button',
  component: Button,
  parameters: {
    layout: 'centered',
    design: {
      type: 'figma',
      url: 'https://www.figma.com/file/xxxxx',
    },
  },
  tags: ['autodocs'],
  argTypes: {
    variant: {
      control: 'select',
      options: ['default', 'destructive', 'outline', 'secondary', 'ghost', 'link', 'whatsapp'],
      description: 'The visual style of the button',
    },
    size: {
      control: 'select',
      options: ['default', 'sm', 'lg', 'icon'],
      description: 'The size of the button',
    },
    disabled: {
      control: 'boolean',
      description: 'Whether the button is disabled',
    },
    asChild: {
      control: 'boolean',
      description: 'Whether to render as child component',
    },
  },
} satisfies Meta<typeof Button>;

export default meta;
type Story = StoryObj<typeof meta>;

// Basic variants
export const Default: Story = {
  args: {
    children: 'Click me',
  },
};

export const Secondary: Story = {
  args: {
    variant: 'secondary',
    children: 'Secondary Action',
  },
};

export const Destructive: Story = {
  args: {
    variant: 'destructive',
    children: 'Delete',
  },
};

export const Outline: Story = {
  args: {
    variant: 'outline',
    children: 'Outline',
  },
};

export const Ghost: Story = {
  args: {
    variant: 'ghost',
    children: 'Ghost',
  },
};

export const Link: Story = {
  args: {
    variant: 'link',
    children: 'Link Button',
  },
};

export const WhatsApp: Story = {
  args: {
    variant: 'whatsapp',
    children: 'Send on WhatsApp',
  },
};

// Sizes
export const Small: Story = {
  args: {
    size: 'sm',
    children: 'Small Button',
  },
};

export const Large: Story = {
  args: {
    size: 'lg',
    children: 'Large Button',
  },
};

export const Icon: Story = {
  args: {
    size: 'icon',
    children: <Mail className="h-4 w-4" />,
  },
};

// With icons
export const WithIcon: Story = {
  args: {
    children: (
      <>
        <Mail className="mr-2 h-4 w-4" />
        Email Me
      </>
    ),
  },
};

export const WhatsAppWithIcon: Story = {
  args: {
    variant: 'whatsapp',
    children: (
      <>
        <Send className="mr-2 h-4 w-4" />
        Send Message
      </>
    ),
  },
};

// States
export const Loading: Story = {
  args: {
    disabled: true,
    children: (
      <>
        <span className="mr-2 h-4 w-4 animate-spin rounded-full border-2 border-background border-t-transparent" />
        Loading...
      </>
    ),
  },
};

export const Disabled: Story = {
  args: {
    disabled: true,
    children: 'Disabled Button',
  },
};

// Interactive story
export const AllVariants: Story = {
  render: () => (
    <div className="flex flex-wrap gap-4">
      <Button>Default</Button>
      <Button variant="secondary">Secondary</Button>
      <Button variant="destructive">Destructive</Button>
      <Button variant="outline">Outline</Button>
      <Button variant="ghost">Ghost</Button>
      <Button variant="link">Link</Button>
      <Button variant="whatsapp">WhatsApp</Button>
    </div>
  ),
};

// Dark mode story
export const DarkMode: Story = {
  parameters: {
    backgrounds: { default: 'dark' },
  },
  decorators: [
    (Story) => (
      <div className="dark">
        <Story />
      </div>
    ),
  ],
  render: () => (
    <div className="flex flex-wrap gap-4 p-8 bg-background">
      <Button>Default</Button>
      <Button variant="secondary">Secondary</Button>
      <Button variant="destructive">Destructive</Button>
      <Button variant="outline">Outline</Button>
      <Button variant="ghost">Ghost</Button>
    </div>
  ),
};

// Complex example
export const RFPActionButtons: Story = {
  render: () => (
    <div className="flex gap-4">
      <Button variant="outline">Save Draft</Button>
      <Button variant="secondary">Preview</Button>
      <Button variant="whatsapp">
        <Send className="mr-2 h-4 w-4" />
        Send via WhatsApp
      </Button>
      <Button>Publish RFP</Button>
    </div>
  ),
};
```

## Mobile App Development Guidelines

### React Native Project Structure

```
apps/mobile/
├── src/
│   ├── components/               # Shared components
│   │   ├── common/              # Common UI components
│   │   │   ├── Button.tsx
│   │   │   ├── Card.tsx
│   │   │   ├── Input.tsx
│   │   │   └── Loading.tsx
│   │   ├── forms/               # Form components
│   │   │   ├── RFPForm.tsx
│   │   │   └── EventForm.tsx
│   │   └── features/            # Feature-specific components
│   │       ├── events/
│   │       ├── artists/
│   │       └── rfp/
│   ├── screens/                 # Screen components
│   │   ├── auth/
│   │   │   ├── LoginScreen.tsx
│   │   │   ├── RegisterScreen.tsx
│   │   │   └── WhatsAppOTPScreen.tsx
│   │   ├── events/
│   │   │   ├── EventListScreen.tsx
│   │   │   └── EventDetailsScreen.tsx
│   │   ├── artists/
│   │   │   ├── ArtistListScreen.tsx
│   │   │   └── ArtistProfileScreen.tsx
│   │   ├── rfp/
│   │   │   ├── RFPListScreen.tsx
│   │   │   └── RFPDetailsScreen.tsx
│   │   └── profile/
│   │       └── ProfileScreen.tsx
│   ├── navigation/              # Navigation configuration
│   │   ├── AppNavigator.tsx
│   │   ├── AuthNavigator.tsx
│   │   ├── TabNavigator.tsx
│   │   └── types.ts
│   ├── services/                # API services
│   │   ├── api.service.ts
│   │   ├── auth.service.ts
│   │   └── whatsapp.service.ts
│   ├── stores/                  # State management
│   │   ├── auth.store.ts
│   │   └── app.store.ts
│   ├── hooks/                   # Custom hooks
│   │   ├── useAuth.ts
│   │   ├── useWhatsApp.ts
│   │   └── useOffline.ts
│   ├── utils/                   # Utility functions
│   │   ├── storage.ts
│   │   ├── permissions.ts
│   │   └── deeplink.ts
│   ├── types/                   # TypeScript types
│   └── constants/               # App constants
├── ios/                         # iOS specific code
│   ├── ArtistRSVP/
│   └── Podfile
├── android/                     # Android specific code
│   ├── app/
│   └── gradle/
├── __tests__/                   # Test files
├── .detoxrc.js                 # Detox E2E config
└── index.js                    # Entry point
```

### Navigation Setup with Deep Linking

```typescript
// navigation/linking.ts
import { LinkingOptions } from '@react-navigation/native';
import { RootStackParamList } from './types';

export const linking: LinkingOptions<RootStackParamList> = {
  prefixes: [
    'artistrsvp://',
    'https://artistrsvp.com',
    'https://*.artistrsvp.com',
  ],
  config: {
    screens: {
      Auth: {
        screens: {
          Login: 'login',
          Register: 'register',
          WhatsAppOTP: 'verify-otp/:sessionId',
        },
      },
      Main: {
        screens: {
          Home: '',
          Events: {
            screens: {
              EventList: 'events',
              EventDetails: 'events/:eventId',
            },
          },
          Artists: {
            screens: {
              ArtistList: 'artists',
              ArtistProfile: 'artists/:artistId',
            },
          },
          RFP: {
            screens: {
              RFPList: 'rfp',
              RFPDetails: 'rfp/:rfpId',
              RFPResponse: 'rfp/:rfpId/respond',
            },
          },
          Profile: 'profile',
        },
      },
    },
  },
  async getInitialURL() {
    // Handle WhatsApp deep links
    const url = await Linking.getInitialURL();
    if (url != null) {
      return url;
    }

    // Handle push notification deep links
    const message = await messaging().getInitialNotification();
    return message?.data?.deepLink;
  },
  subscribe(listener) {
    // Listen to incoming links from deep linking
    const linkingSubscription = Linking.addEventListener('url', ({ url }) => {
      listener(url);
    });

    // Listen to push notification interactions
    const unsubscribe = messaging().onNotificationOpenedApp((message) => {
      const url = message.data?.deepLink;
      if (url) {
        listener(url);
      }
    });

    return () => {
      linkingSubscription.remove();
      unsubscribe();
    };
  },
};

// navigation/AppNavigator.tsx
import { NavigationContainer } from '@react-navigation/native';
import { createNativeStackNavigator } from '@react-navigation/native-stack';
import { useAuthStore } from '@/stores/auth.store';
import { linking } from './linking';
import SplashScreen from 'react-native-splash-screen';

const Stack = createNativeStackNavigator<RootStackParamList>();

export function AppNavigator() {
  const { isAuthenticated, isLoading } = useAuthStore();
  const [isReady, setIsReady] = useState(false);

  useEffect(() => {
    async function prepare() {
      try {
        // Restore authentication state
        await useAuthStore.getState().restoreSession();
      } catch (e) {
        console.error('Failed to restore session:', e);
      } finally {
        setIsReady(true);
        SplashScreen.hide();
      }
    }

    prepare();
  }, []);

  if (!isReady || isLoading) {
    return null; // Splash screen is still visible
  }

  return (
    <NavigationContainer
      linking={linking}
      fallback={<LoadingScreen />}
      onStateChange={(state) => {
        // Track navigation analytics
        analytics().logScreenView({
          screen_name: getCurrentRouteName(state),
          screen_class: getCurrentRouteClass(state),
        });
      }}
    >
      <Stack.Navigator
        screenOptions={{
          headerShown: false,
          animation: 'slide_from_right',
        }}
      >
        {isAuthenticated ? (
          <>
            <Stack.Screen name="Main" component={MainNavigator} />
            <Stack.Screen 
              name="RFPResponse" 
              component={RFPResponseScreen}
              options={{
                presentation: 'modal',
                animation: 'slide_from_bottom',
              }}
            />
            <Stack.Screen 
              name="WhatsAppShare" 
              component={WhatsAppShareScreen}
              options={{
                presentation: 'transparentModal',
                animation: 'fade',
              }}
            />
          </>
        ) : (
          <Stack.Screen name="Auth" component={AuthNavigator} />
        )}
      </Stack.Navigator>
    </NavigationContainer>
  );
}
```

### Performance Optimization

```typescript
// components/OptimizedList.tsx
import React, { memo, useCallback, useMemo } from 'react';
import { FlashList } from '@shopify/flash-list';
import { RefreshControl, View, Text } from 'react-native';
import FastImage from 'react-native-fast-image';
import { useInfiniteQuery } from '@tanstack/react-query';

interface OptimizedListProps<T> {
  queryKey: string[];
  queryFn: (params: any) => Promise<{ data: T[]; nextPage?: number }>;
  renderItem: (item: T) => React.ReactElement;
  keyExtractor: (item: T) => string;
  estimatedItemSize: number;
  HeaderComponent?: React.ComponentType;
  EmptyComponent?: React.ComponentType;
}

export function OptimizedList<T>({
  queryKey,
  queryFn,
  renderItem,
  keyExtractor,
  estimatedItemSize,
  HeaderComponent,
  EmptyComponent,
}: OptimizedListProps<T>) {
  const {
    data,
    fetchNextPage,
    hasNextPage,
    isFetching,
    isFetchingNextPage,
    refetch,
    isLoading,
    error,
  } = useInfiniteQuery({
    queryKey,
   queryFn: ({ pageParam = 1 }) => queryFn({ page: pageParam }),
    getNextPageParam: (lastPage) => lastPage.nextPage,
    staleTime: 5 * 60 * 1000, // 5 minutes
    cacheTime: 10 * 60 * 1000, // 10 minutes
  });

  const flatData = useMemo(
    () => data?.pages.flatMap((page) => page.data) ?? [],
    [data]
  );

  const renderItemMemoized = useCallback(
    ({ item }: { item: T }) => renderItem(item),
    [renderItem]
  );

  const onEndReached = useCallback(() => {
    if (hasNextPage && !isFetchingNextPage) {
      fetchNextPage();
    }
  }, [hasNextPage, isFetchingNextPage, fetchNextPage]);

  const ListFooterComponent = useCallback(() => {
    if (!isFetchingNextPage) return null;
    return (
      <View className="py-4">
        <ActivityIndicator size="small" color="#6366f1" />
      </View>
    );
  }, [isFetchingNextPage]);

  if (isLoading) {
    return <LoadingScreen />;
  }

  if (error) {
    return <ErrorScreen error={error} onRetry={refetch} />;
  }

  return (
    <FlashList
      data={flatData}
      renderItem={renderItemMemoized}
      keyExtractor={keyExtractor}
      estimatedItemSize={estimatedItemSize}
      onEndReached={onEndReached}
      onEndReachedThreshold={0.5}
      refreshControl={
        <RefreshControl
          refreshing={isFetching && !isFetchingNextPage}
          onRefresh={refetch}
          tintColor="#6366f1"
        />
      }
      ListHeaderComponent={HeaderComponent}
      ListEmptyComponent={EmptyComponent}
      ListFooterComponent={ListFooterComponent}
      showsVerticalScrollIndicator={false}
      removeClippedSubviews
      maxToRenderPerBatch={10}
      updateCellsBatchingPeriod={50}
      windowSize={10}
    />
  );
}

// Example usage with optimized image component
const ArtistCard = memo(({ artist }: { artist: Artist }) => {
  return (
    <Pressable className="bg-white rounded-lg shadow-sm p-4 mb-3">
      <View className="flex-row">
        <FastImage
          source={{
            uri: artist.profileImage,
            priority: FastImage.priority.normal,
            cache: FastImage.cacheControl.immutable,
          }}
          className="w-16 h-16 rounded-full"
          resizeMode={FastImage.resizeMode.cover}
        />
        <View className="ml-4 flex-1">
          <Text className="text-lg font-semibold">{artist.name}</Text>
          <Text className="text-gray-600">{artist.genre}</Text>
          <View className="flex-row items-center mt-1">
            <Star className="w-4 h-4 text-yellow-500" />
            <Text className="ml-1 text-sm">{artist.rating}</Text>
          </View>
        </View>
      </View>
    </Pressable>
  );
});

Offline Support & Data Persistence
typescript// hooks/useOffline.ts
import NetInfo from '@react-native-community/netinfo';
import { useEffect, useState } from 'react';
import AsyncStorage from '@react-native-async-storage/async-storage';
import { useMutation, useQueryClient } from '@tanstack/react-query';

export function useOffline() {
  const [isOffline, setIsOffline] = useState(false);
  const queryClient = useQueryClient();

  useEffect(() => {
    const unsubscribe = NetInfo.addEventListener((state) => {
      setIsOffline(!state.isConnected);
      
      if (state.isConnected) {
        // Sync offline data when connection is restored
        syncOfflineData();
      }
    });

    return unsubscribe;
  }, []);

  const syncOfflineData = async () => {
    try {
      const offlineQueue = await AsyncStorage.getItem('@offline_queue');
      if (offlineQueue) {
        const queue = JSON.parse(offlineQueue);
        
        for (const request of queue) {
          await processOfflineRequest(request);
        }
        
        await AsyncStorage.removeItem('@offline_queue');
        queryClient.invalidateQueries();
      }
    } catch (error) {
      console.error('Failed to sync offline data:', error);
    }
  };

  const queueOfflineRequest = async (request: any) => {
    try {
      const existingQueue = await AsyncStorage.getItem('@offline_queue');
      const queue = existingQueue ? JSON.parse(existingQueue) : [];
      queue.push({
        ...request,
        timestamp: new Date().toISOString(),
        id: `${Date.now()}-${Math.random()}`,
      });
      await AsyncStorage.setItem('@offline_queue', JSON.stringify(queue));
    } catch (error) {
      console.error('Failed to queue offline request:', error);
    }
  };

  return {
    isOffline,
    queueOfflineRequest,
    syncOfflineData,
  };
}

// services/offline.service.ts
class OfflineService {
  private readonly CACHE_PREFIX = '@cache_';
  private readonly CACHE_EXPIRY = 24 * 60 * 60 * 1000; // 24 hours

  async cacheData(key: string, data: any, expiryMs?: number) {
    const cacheData = {
      data,
      timestamp: Date.now(),
      expiry: expiryMs || this.CACHE_EXPIRY,
    };
    
    await AsyncStorage.setItem(
      `${this.CACHE_PREFIX}${key}`,
      JSON.stringify(cacheData)
    );
  }

  async getCachedData<T>(key: string): Promise<T | null> {
    try {
      const cached = await AsyncStorage.getItem(`${this.CACHE_PREFIX}${key}`);
      if (!cached) return null;

      const { data, timestamp, expiry } = JSON.parse(cached);
      
      if (Date.now() - timestamp > expiry) {
        await AsyncStorage.removeItem(`${this.CACHE_PREFIX}${key}`);
        return null;
      }

      return data as T;
    } catch (error) {
      console.error('Failed to get cached data:', error);
      return null;
    }
  }

  async clearCache(pattern?: string) {
    const keys = await AsyncStorage.getAllKeys();
    const cacheKeys = keys.filter((key) => {
      if (!key.startsWith(this.CACHE_PREFIX)) return false;
      if (pattern) {
        return key.includes(pattern);
      }
      return true;
    });

    await AsyncStorage.multiRemove(cacheKeys);
  }
}

export const offlineService = new OfflineService();
WhatsApp Integration for Mobile
typescript// services/whatsapp-mobile.service.ts
import { Linking, Platform } from 'react-native';
import Share from 'react-native-share';
import { check, request, PERMISSIONS, RESULTS } from 'react-native-permissions';

export class WhatsAppMobileService {
  async shareToWhatsApp(message: string, phoneNumber?: string) {
    try {
      const whatsappInstalled = await this.isWhatsAppInstalled();
      
      if (!whatsappInstalled) {
        throw new Error('WhatsApp is not installed');
      }

      if (phoneNumber) {
        // Direct message to specific number
        const url = `whatsapp://send?phone=${phoneNumber}&text=${encodeURIComponent(message)}`;
        await Linking.openURL(url);
      } else {
        // Share via WhatsApp picker
        await Share.open({
          message,
          social: Share.Social.WHATSAPP,
        });
      }
    } catch (error) {
      console.error('WhatsApp share failed:', error);
      throw error;
    }
  }

  async shareMedia(options: {
    message?: string;
    url?: string;
    phoneNumber?: string;
  }) {
    try {
      const { message, url, phoneNumber } = options;
      
      if (phoneNumber && url) {
        // WhatsApp doesn't support direct media sharing to specific numbers
        // Fallback to message with link
        const fullMessage = message ? `${message}\n${url}` : url;
        await this.shareToWhatsApp(fullMessage, phoneNumber);
      } else if (url) {
        await Share.open({
          message,
          url,
          social: Share.Social.WHATSAPP,
        });
      }
    } catch (error) {
      console.error('WhatsApp media share failed:', error);
      throw error;
    }
  }

  async isWhatsAppInstalled(): Promise<boolean> {
    const whatsappURL = Platform.select({
      ios: 'whatsapp://',
      android: 'whatsapp://send?phone=',
    });

    return Linking.canOpenURL(whatsappURL!);
  }

  async requestContactsPermission(): Promise<boolean> {
    try {
      const permission = Platform.select({
        ios: PERMISSIONS.IOS.CONTACTS,
        android: PERMISSIONS.ANDROID.READ_CONTACTS,
      });

      const result = await request(permission!);
      return result === RESULTS.GRANTED;
    } catch (error) {
      console.error('Contacts permission request failed:', error);
      return false;
    }
  }

  formatWhatsAppLink(phoneNumber: string, message?: string): string {
    const cleanNumber = phoneNumber.replace(/\D/g, '');
    const encodedMessage = message ? encodeURIComponent(message) : '';
    return `https://wa.me/${cleanNumber}${message ? `?text=${encodedMessage}` : ''}`;
  }
}

export const whatsAppMobileService = new WhatsAppMobileService();

// components/WhatsAppButton.tsx
import React from 'react';
import { Pressable, Text, Alert } from 'react-native';
import { whatsAppMobileService } from '@/services/whatsapp-mobile.service';
import { Send } from 'lucide-react-native';

interface WhatsAppButtonProps {
  message: string;
  phoneNumber?: string;
  onSuccess?: () => void;
  onError?: (error: Error) => void;
}

export function WhatsAppButton({
  message,
  phoneNumber,
  onSuccess,
  onError,
}: WhatsAppButtonProps) {
  const handlePress = async () => {
    try {
      const isInstalled = await whatsAppMobileService.isWhatsAppInstalled();
      
      if (!isInstalled) {
        Alert.alert(
          'WhatsApp Not Installed',
          'Please install WhatsApp to use this feature.',
          [
            { text: 'Cancel', style: 'cancel' },
            {
              text: 'Install',
              onPress: () => {
                const storeUrl = Platform.select({
                  ios: 'https://apps.apple.com/app/whatsapp-messenger/id310633997',
                  android: 'market://details?id=com.whatsapp',
                });
                Linking.openURL(storeUrl!);
              },
            },
          ]
        );
        return;
      }

      await whatsAppMobileService.shareToWhatsApp(message, phoneNumber);
      onSuccess?.();
    } catch (error) {
      console.error('WhatsApp share error:', error);
      onError?.(error as Error);
    }
  };

  return (
    <Pressable
      onPress={handlePress}
      className="bg-whatsapp px-4 py-3 rounded-lg flex-row items-center justify-center active:opacity-80"
    >
      <Send size={20} color="white" className="mr-2" />
      <Text className="text-white font-semibold">Send on WhatsApp</Text>
    </Pressable>
  );
}
Push Notifications Setup
typescript// services/notifications.service.ts
import messaging, { FirebaseMessagingTypes } from '@react-native-firebase/messaging';
import notifee, { AndroidImportance, AndroidStyle } from '@notifee/react-native';
import { Platform } from 'react-native';
import AsyncStorage from '@react-native-async-storage/async-storage';

class NotificationService {
  private readonly FCM_TOKEN_KEY = '@fcm_token';

  async initialize() {
    // Request permissions
    await this.requestPermissions();
    
    // Get FCM token
    await this.getFCMToken();
    
    // Setup notification handlers
    this.setupNotificationHandlers();
    
    // Create notification channels for Android
    if (Platform.OS === 'android') {
      await this.createNotificationChannels();
    }
  }

  async requestPermissions(): Promise<boolean> {
    const authStatus = await messaging().requestPermission();
    const enabled =
      authStatus === messaging.AuthorizationStatus.AUTHORIZED ||
      authStatus === messaging.AuthorizationStatus.PROVISIONAL;

    if (enabled) {
      console.log('Push notifications authorized');
    }

    return enabled;
  }

  async getFCMToken(): Promise<string | null> {
    try {
      const token = await messaging().getToken();
      
      // Save token for comparison
      const savedToken = await AsyncStorage.getItem(this.FCM_TOKEN_KEY);
      
      if (token !== savedToken) {
        await AsyncStorage.setItem(this.FCM_TOKEN_KEY, token);
        // Send token to backend
        await this.sendTokenToServer(token);
      }

      return token;
    } catch (error) {
      console.error('Failed to get FCM token:', error);
      return null;
    }
  }

  private async sendTokenToServer(token: string) {
    try {
      await api.post('/notifications/register-device', {
        token,
        platform: Platform.OS,
        deviceInfo: {
          brand: DeviceInfo.getBrand(),
          model: DeviceInfo.getModel(),
          systemVersion: DeviceInfo.getSystemVersion(),
        },
      });
    } catch (error) {
      console.error('Failed to register device token:', error);
    }
  }

  private setupNotificationHandlers() {
    // Handle foreground notifications
    messaging().onMessage(async (remoteMessage) => {
      await this.displayNotification(remoteMessage);
    });

    // Handle background notification interactions
    messaging().setBackgroundMessageHandler(async (remoteMessage) => {
      console.log('Background notification:', remoteMessage);
    });

    // Handle notification opened from quit state
    messaging()
      .getInitialNotification()
      .then((remoteMessage) => {
        if (remoteMessage) {
          this.handleNotificationOpen(remoteMessage);
        }
      });

    // Handle notification opened from background state
    messaging().onNotificationOpenedApp((remoteMessage) => {
      this.handleNotificationOpen(remoteMessage);
    });

    // Token refresh listener
    messaging().onTokenRefresh(async (token) => {
      await AsyncStorage.setItem(this.FCM_TOKEN_KEY, token);
      await this.sendTokenToServer(token);
    });
  }

  private async createNotificationChannels() {
    await notifee.createChannel({
      id: 'default',
      name: 'Default Channel',
      importance: AndroidImportance.HIGH,
    });

    await notifee.createChannel({
      id: 'rfp',
      name: 'RFP Notifications',
      importance: AndroidImportance.HIGH,
      badge: true,
    });

    await notifee.createChannel({
      id: 'messages',
      name: 'Messages',
      importance: AndroidImportance.HIGH,
      badge: true,
    });

    await notifee.createChannel({
      id: 'events',
      name: 'Event Updates',
      importance: AndroidImportance.DEFAULT,
    });
  }

  async displayNotification(
    remoteMessage: FirebaseMessagingTypes.RemoteMessage
  ) {
    const { notification, data } = remoteMessage;
    
    if (!notification) return;

    const notificationOptions: any = {
      title: notification.title,
      body: notification.body,
      data,
      android: {
        channelId: data?.channelId || 'default',
        smallIcon: 'ic_notification',
        color: '#6366f1',
        importance: AndroidImportance.HIGH,
        pressAction: {
          id: 'default',
        },
      },
      ios: {
        foregroundPresentationOptions: {
          badge: true,
          sound: true,
          banner: true,
          list: true,
        },
      },
    };

    // Add image if present
    if (notification.android?.imageUrl) {
      notificationOptions.android.largeIcon = notification.android.imageUrl;
      notificationOptions.android.style = {
        type: AndroidStyle.BIGPICTURE,
        picture: notification.android.imageUrl,
      };
    }

    // Add actions based on notification type
    if (data?.type === 'rfp_match') {
      notificationOptions.android.actions = [
        {
          title: 'View RFP',
          pressAction: { id: 'view_rfp', launchActivity: 'default' },
        },
        {
          title: 'Quick Response',
          pressAction: { id: 'quick_response' },
          input: {
            allowFreeFormInput: true,
            placeholder: 'Type your response...',
          },
        },
      ];
    }

    await notifee.displayNotification(notificationOptions);
  }

  private handleNotificationOpen(
    remoteMessage: FirebaseMessagingTypes.RemoteMessage
  ) {
    const { data } = remoteMessage;

    if (data?.deepLink) {
      // Navigate to deep link
      Linking.openURL(data.deepLink);
    } else if (data?.screen) {
      // Navigate to specific screen
      navigationRef.current?.navigate(data.screen as any, data.params);
    }
  }

  async setBadgeCount(count: number) {
    if (Platform.OS === 'ios') {
      await notifee.setBadgeCount(count);
    }
  }

  async cancelAllNotifications() {
    await notifee.cancelAllNotifications();
  }
}

export const notificationService = new NotificationService();
Third-Party API Integrations
Integration Architecture
typescript// services/integrations/base-integration.service.ts
import axios, { AxiosInstance, AxiosError } from 'axios';
import { z } from 'zod';
import CircuitBreaker from 'opossum';
import { Logger } from '@nestjs/common';

export interface IntegrationConfig {
  name: string;
  baseURL: string;
  timeout: number;
  retries: number;
  circuitBreakerOptions?: CircuitBreaker.Options;
}

export abstract class BaseIntegrationService {
  protected client: AxiosInstance;
  protected logger: Logger;
  protected circuitBreaker: CircuitBreaker;
  
  constructor(protected config: IntegrationConfig) {
    this.logger = new Logger(config.name);
    
    this.client = axios.create({
      baseURL: config.baseURL,
      timeout: config.timeout,
      headers: this.getDefaultHeaders(),
    });

    this.setupInterceptors();
    this.setupCircuitBreaker();
  }

  protected abstract getDefaultHeaders(): Record<string, string>;
  protected abstract handleError(error: AxiosError): never;

  private setupInterceptors() {
    // Request interceptor
    this.client.interceptors.request.use(
      (config) => {
        config.headers['X-Request-ID'] = this.generateRequestId();
        config.metadata = { startTime: Date.now() };
        
        this.logger.debug(`${config.method?.toUpperCase()} ${config.url}`, {
          headers: config.headers,
          data: config.data,
        });
        
        return config;
      },
      (error) => {
        this.logger.error('Request error:', error);
        return Promise.reject(error);
      }
    );

    // Response interceptor
    this.client.interceptors.response.use(
      (response) => {
        const duration = Date.now() - response.config.metadata.startTime;
        
        this.logger.debug(`Response ${response.status} in ${duration}ms`, {
          url: response.config.url,
          data: response.data,
        });
        
        return response;
      },
      async (error: AxiosError) => {
        const originalRequest = error.config as any;
        
        if (error.response?.status === 429 && originalRequest._retry < this.config.retries) {
          originalRequest._retry = (originalRequest._retry || 0) + 1;
          
          const retryAfter = this.getRetryAfter(error.response);
          await this.delay(retryAfter);
          
          return this.client(originalRequest);
        }
        
        return Promise.reject(this.handleError(error));
      }
    );
  }

  private setupCircuitBreaker() {
    const options: CircuitBreaker.Options = this.config.circuitBreakerOptions || {
      timeout: 3000,
      errorThresholdPercentage: 50,
      resetTimeout: 30000,
      rollingCountTimeout: 10000,
      rollingCountBuckets: 10,
      name: this.config.name,
    };

    this.circuitBreaker = new CircuitBreaker(
      async (fn: Function) => fn(),
      options
    );

    this.circuitBreaker.on('open', () => {
      this.logger.warn(`Circuit breaker opened for ${this.config.name}`);
    });

    this.circuitBreaker.on('halfOpen', () => {
      this.logger.log(`Circuit breaker half-open for ${this.config.name}`);
    });

    this.circuitBreaker.on('close', () => {
      this.logger.log(`Circuit breaker closed for ${this.config.name}`);
    });
  }

  protected async makeRequest<T>(
    fn: () => Promise<T>,
    schema?: z.ZodSchema<T>
  ): Promise<T> {
    try {
      const response = await this.circuitBreaker.fire(fn);
      
      if (schema) {
        return schema.parse(response);
      }
      
      return response;
    } catch (error) {
      if (error instanceof CircuitBreaker.OpenCircuitError) {
        throw new Error(`Service ${this.config.name} is temporarily unavailable`);
      }
      throw error;
    }
  }

  private getRetryAfter(response: any): number {
    const retryAfter = response.headers['retry-after'];
    if (retryAfter) {
      return parseInt(retryAfter) * 1000;
    }
    return 1000; // Default 1 second
  }

  private delay(ms: number): Promise<void> {
    return new Promise(resolve => setTimeout(resolve, ms));
  }

  private generateRequestId(): string {
    return `${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;
  }
}

WhatsApp Cloud API Integration
typescript// services/integrations/whatsapp-cloud.service.ts
import { Injectable } from '@nestjs/common';
import { BaseIntegrationService } from './base-integration.service';
import { z } from 'zod';
import crypto from 'crypto';

const WhatsAppMessageSchema = z.object({
  messaging_product: z.literal('whatsapp'),
  recipient_type: z.enum(['individual', 'group']),
  to: z.string(),
  type: z.enum(['text', 'template', 'image', 'document', 'audio', 'video', 'location']),
  text: z.object({ body: z.string() }).optional(),
  template: z.object({
    name: z.string(),
    language: z.object({ code: z.string() }),
    components: z.array(z.any()).optional(),
  }).optional(),
});

@Injectable()
export class WhatsAppCloudService extends BaseIntegrationService {
  private phoneNumberId: string;
  private businessAccountId: string;
  private webhookVerifyToken: string;

  constructor(configService: ConfigService) {
    super({
      name: 'WhatsApp Cloud API',
      baseURL: `https://graph.facebook.com/v18.0`,
      timeout: 30000,
      retries: 3,
    });

    this.phoneNumberId = configService.get('WHATSAPP_PHONE_NUMBER_ID');
    this.businessAccountId = configService.get('WHATSAPP_BUSINESS_ACCOUNT_ID');
    this.webhookVerifyToken = configService.get('WHATSAPP_WEBHOOK_VERIFY_TOKEN');
  }

  protected getDefaultHeaders(): Record<string, string> {
    return {
      'Authorization': `Bearer ${process.env.WHATSAPP_ACCESS_TOKEN}`,
      'Content-Type': 'application/json',
    };
  }

  protected handleError(error: AxiosError): never {
    const errorData = error.response?.data as any;
    
    if (errorData?.error) {
      const whatsappError = errorData.error;
      throw new Error(
        `WhatsApp API Error: ${whatsappError.message} (Code: ${whatsappError.code})`
      );
    }
    
    throw new Error(`WhatsApp API Error: ${error.message}`);
  }

  // Send OTP via WhatsApp
  async sendOTP(phoneNumber: string, otp: string): Promise<void> {
    const message = {
      messaging_product: 'whatsapp',
      recipient_type: 'individual',
      to: phoneNumber,
      type: 'template',
      template: {
        name: 'otp_verification',
        language: { code: 'en' },
        components: [
          {
            type: 'body',
            parameters: [
              { type: 'text', text: otp },
              { type: 'text', text: '10 minutes' }
            ]
          }
        ]
      }
    };

    await this.makeRequest(
      () => this.client.post(`/${this.phoneNumberId}/messages`, message),
      WhatsAppMessageSchema
    );
  }

  // Send RFP notification
  async sendRFPNotification(data: {
    phoneNumber: string;
    rfpTitle: string;
    eventDate: string;
    budget: string;
    location: string;
    rfpId: string;
  }): Promise<void> {
    const message = {
      messaging_product: 'whatsapp',
      recipient_type: 'individual',
      to: data.phoneNumber,
      type: 'template',
      template: {
        name: 'rfp_notification',
        language: { code: 'en' },
        components: [
          {
            type: 'header',
            parameters: [
              { type: 'text', text: data.rfpTitle }
            ]
          },
          {
            type: 'body',
            parameters: [
              { type: 'text', text: data.eventDate },
              { type: 'text', text: data.budget },
              { type: 'text', text: data.location }
            ]
          },
          {
            type: 'button',
            sub_type: 'url',
            index: '0',
            parameters: [
              { type: 'text', text: data.rfpId }
            ]
          }
        ]
      }
    };

    await this.makeRequest(
      () => this.client.post(`/${this.phoneNumberId}/messages`, message)
    );
  }

  // Send interactive message with buttons
  async sendInteractiveMessage(data: {
    phoneNumber: string;
    body: string;
    buttons: Array<{ id: string; title: string }>;
  }): Promise<void> {
    const message = {
      messaging_product: 'whatsapp',
      recipient_type: 'individual',
      to: data.phoneNumber,
      type: 'interactive',
      interactive: {
        type: 'button',
        body: { text: data.body },
        action: {
          buttons: data.buttons.map(btn => ({
            type: 'reply',
            reply: { id: btn.id, title: btn.title }
          }))
        }
      }
    };

    await this.makeRequest(
      () => this.client.post(`/${this.phoneNumberId}/messages`, message)
    );
  }

  // Handle webhook verification
  verifyWebhook(query: any): { challenge: string } | null {
    const mode = query['hub.mode'];
    const token = query['hub.verify_token'];
    const challenge = query['hub.challenge'];

    if (mode === 'subscribe' && token === this.webhookVerifyToken) {
      return { challenge };
    }

    return null;
  }

  // Handle incoming webhook
  async handleWebhook(body: any, signature: string): Promise<void> {
    // Verify webhook signature
    if (!this.verifyWebhookSignature(body, signature)) {
      throw new Error('Invalid webhook signature');
    }

    const { entry } = body;
    
    for (const entryItem of entry) {
      const { changes } = entryItem;
      
      for (const change of changes) {
        if (change.field === 'messages') {
          await this.processMessages(change.value);
        }
      }
    }
  }

  private verifyWebhookSignature(body: any, signature: string): boolean {
    const payload = JSON.stringify(body);
    const expectedSignature = crypto
      .createHmac('sha256', process.env.WHATSAPP_APP_SECRET!)
      .update(payload)
      .digest('hex');
    
    return `sha256=${expectedSignature}` === signature;
  }

  private async processMessages(value: any): Promise<void> {
    const { messages, contacts } = value;
    
    if (!messages) return;

    for (const message of messages) {
      const contact = contacts.find((c: any) => c.wa_id === message.from);
      
      // Emit event for message processing
      await this.eventBus.publish({
        eventType: 'WHATSAPP_MESSAGE_RECEIVED',
        source: 'whatsapp-integration',
        detail: {
          messageId: message.id,
          from: message.from,
          contactName: contact?.profile?.name,
          type: message.type,
          text: message.text?.body,
          timestamp: new Date(parseInt(message.timestamp) * 1000),
        }
      });
    }
  }

  // Get message templates
  async getMessageTemplates(): Promise<any[]> {
    const response = await this.makeRequest(
      () => this.client.get(`/${this.businessAccountId}/message_templates`)
    );

    return response.data.data;
  }

  // Upload media
  async uploadMedia(file: Buffer, mimeType: string): Promise<string> {
    const formData = new FormData();
    formData.append('file', file, { contentType: mimeType });
    formData.append('messaging_product', 'whatsapp');

    const response = await this.makeRequest(
      () => this.client.post(`/${this.phoneNumberId}/media`, formData, {
        headers: { 'Content-Type': 'multipart/form-data' }
      })
    );

    return response.data.id;
  }
}

OpenAI Integration
typescript// services/integrations/openai.service.ts
import { Injectable } from '@nestjs/common';
import { BaseIntegrationService } from './base-integration.service';
import { z } from 'zod';

const CompletionResponseSchema = z.object({
  id: z.string(),
  object: z.string(),
  created: z.number(),
  model: z.string(),
  choices: z.array(z.object({
    index: z.number(),
    message: z.object({
      role: z.string(),
      content: z.string(),
    }),
    finish_reason: z.string(),
  })),
  usage: z.object({
    prompt_tokens: z.number(),
    completion_tokens: z.number(),
    total_tokens: z.number(),
  }),
});

@Injectable()
export class OpenAIService extends BaseIntegrationService {
  private apiKey: string;
  private organization: string;
  private defaultModel = 'gpt-4-turbo';

  constructor(configService: ConfigService) {
    super({
      name: 'OpenAI',
      baseURL: 'https://api.openai.com/v1',
      timeout: 60000,
      retries: 2,
      circuitBreakerOptions: {
        timeout: 30000,
        errorThresholdPercentage: 30,
      },
    });

    this.apiKey = configService.get('OPENAI_API_KEY');
    this.organization = configService.get('OPENAI_ORGANIZATION');
  }

  protected getDefaultHeaders(): Record<string, string> {
    return {
      'Authorization': `Bearer ${this.apiKey}`,
      'OpenAI-Organization': this.organization,
      'Content-Type': 'application/json',
    };
  }

  protected handleError(error: AxiosError): never {
    const errorData = error.response?.data as any;
    
    if (error.response?.status === 429) {
      throw new Error('OpenAI rate limit exceeded. Please try again later.');
    }
    
    if (errorData?.error) {
      throw new Error(`OpenAI Error: ${errorData.error.message}`);
    }
    
    throw new Error(`OpenAI Error: ${error.message}`);
  }

  // Generate RFP content
  async generateRFPContent(data: {
    eventType: string;
    date: string;
    location: string;
    guestCount: number;
    preferences: string[];
    budget: { min: number; max: number; currency: string };
  }): Promise<string> {
    const prompt = `Create a professional Request for Proposal (RFP) for the following event:
    
Event Type: ${data.eventType}
Date: ${data.date}
Location: ${data.location}
Guest Count: ${data.guestCount}
Budget: ${data.budget.currency} ${data.budget.min}-${data.budget.max}
Preferences: ${data.preferences.join(', ')}

Generate a comprehensive RFP that includes:
1. Event overview and objectives
2. Detailed requirements for artists/performers
3. Technical requirements
4. Timeline and milestones
5. Evaluation criteria
6. Submission guidelines

Make it professional, clear, and engaging.`;

    const response = await this.makeRequest(
      () => this.client.post('/chat/completions', {
        model: this.defaultModel,
        messages: [
          { role: 'system', content: 'You are an expert event planner creating professional RFPs.' },
          { role: 'user', content: prompt }
        ],
        temperature: 0.7,
        max_tokens: 2000,
      }),
      CompletionResponseSchema
    );

    return response.data.choices[0].message.content;
  }

  // Match artists to RFP
  async matchArtistsToRFP(data: {
    rfpRequirements: string;
    artistProfiles: Array<{
      id: string;
      name: string;
      bio: string;
      genres: string[];
      experience: string;
      rating: number;
      priceRange: { min: number; max: number };
    }>;
  }): Promise<Array<{ artistId: string; score: number; reasoning: string }>> {
    const prompt = `Analyze the following RFP requirements and rank the artists based on their fit:

RFP Requirements:
${data.rfpRequirements}

Artists:
${data.artistProfiles.map((artist, i) => `
${i + 1}. ${artist.name} (ID: ${artist.id})
- Genres: ${artist.genres.join(', ')}
- Experience: ${artist.experience}
- Rating: ${artist.rating}/5
- Price Range: $${artist.priceRange.min}-$${artist.priceRange.max}
- Bio: ${artist.bio}
`).join('\n')}

For each artist, provide:
1. Match score (0-100)
2. Brief reasoning (2-3 sentences)

Return as JSON array with format: [{ "artistId": "...", "score": 0-100, "reasoning": "..." }]`;

    const response = await this.makeRequest(
      () => this.client.post('/chat/completions', {
        model: this.defaultModel,
        messages: [
          { role: 'system', content: 'You are an AI matching system for events and artists.' },
          { role: 'user', content: prompt }
        ],
        temperature: 0.3,
        max_tokens: 1500,
        response_format: { type: 'json_object' },
      }),
      CompletionResponseSchema
    );

    const result = JSON.parse(response.data.choices[0].message.content);
    return result.matches || result;
  }

  // Generate event description
  async generateEventDescription(data: {
    title: string;
    type: string;
    date: string;
    venue: string;
    highlights: string[];
  }): Promise<{ description: string; tags: string[] }> {
    const response = await this.makeRequest(
      () => this.client.post('/chat/completions', {
        model: this.defaultModel,
        messages: [
          { 
            role: 'system', 
            content: 'Generate engaging event descriptions and relevant tags.' 
          },
          { 
            role: 'user', 
            content: `Create a compelling event description and tags for:
              Title: ${data.title}
              Type: ${data.type}
              Date: ${data.date}
              Venue: ${data.venue}
              Highlights: ${data.highlights.join(', ')}
              
              Return as JSON: { "description": "...", "tags": ["...", "..."] }`
          }
        ],
        temperature: 0.8,
        response_format: { type: 'json_object' },
      }),
      CompletionResponseSchema
    );

    return JSON.parse(response.data.choices[0].message.content);
  }

  // Generate embeddings for similarity search
  async generateEmbeddings(texts: string[]): Promise<number[][]> {
    const response = await this.makeRequest(
      () => this.client.post('/embeddings', {
        model: 'text-embedding-3-small',
        input: texts,
      })
    );

    return response.data.data.map((item: any) => item.embedding);
  }
}

Google Services Integration
typescript// services/integrations/google/google-calendar.service.ts
import { google, calendar_v3 } from 'googleapis';
import { OAuth2Client } from 'google-auth-library';
import { Injectable } from '@nestjs/common';

@Injectable()
export class GoogleCalendarService {
  private oauth2Client: OAuth2Client;
  private calendar: calendar_v3.Calendar;

  constructor(private configService: ConfigService) {
    this.oauth2Client = new google.auth.OAuth2(
      configService.get('GOOGLE_CLIENT_ID'),
      configService.get('GOOGLE_CLIENT_SECRET'),
      configService.get('GOOGLE_REDIRECT_URL')
    );

    this.calendar = google.calendar({ version: 'v3', auth: this.oauth2Client });
  }

  async setUserCredentials(tokens: any) {
    this.oauth2Client.setCredentials(tokens);
  }

  async createEvent(eventData: {
    summary: string;
    description: string;
    location: string;
    startDateTime: Date;
    endDateTime: Date;
    attendees?: string[];
    reminders?: Array<{ method: string; minutes: number }>;
  }): Promise<calendar_v3.Schema$Event> {
    const event: calendar_v3.Schema$Event = {
      summary: eventData.summary,
      description: eventData.description,
      location: eventData.location,
      start: {
        dateTime: eventData.startDateTime.toISOString(),
        timeZone: 'UTC',
      },
      end: {
        dateTime: eventData.endDateTime.toISOString(),
        timeZone: 'UTC',
      },
      attendees: eventData.attendees?.map(email => ({ email })),
      reminders: {
        useDefault: false,
        overrides: eventData.reminders || [
          { method: 'email', minutes: 24 * 60 },
          { method: 'popup', minutes: 60 },
        ],
      },
    };

    const response = await this.calendar.events.insert({
      calendarId: 'primary',
      requestBody: event,
      sendNotifications: true,
    });

    return response.data;
  }

  async checkAvailability(
    startTime: Date,
    endTime: Date,
    calendarIds: string[] = ['primary']
  ): Promise<Array<{ calendarId: string; busy: boolean; conflicts: any[] }>> {
    const response = await this.calendar.freebusy.query({
      requestBody: {
        timeMin: startTime.toISOString(),
        timeMax: endTime.toISOString(),
        items: calendarIds.map(id => ({ id })),
      },
    });

    const results = [];
    for (const [calendarId, calendar] of Object.entries(response.data.calendars || {})) {
      const busySlots = calendar.busy || [];
      const conflicts = busySlots.filter((slot: any) => {
        const slotStart = new Date(slot.start!);
        const slotEnd = new Date(slot.end!);
        return (
          (startTime >= slotStart && startTime < slotEnd) ||
          (endTime > slotStart && endTime <= slotEnd) ||
          (startTime <= slotStart && endTime >= slotEnd)
        );
      });

      results.push({
        calendarId,
        busy: conflicts.length > 0,
        conflicts,
      });
    }

    return results;
  }

  async syncEventUpdates(eventId: string, updates: Partial<calendar_v3.Schema$Event>) {
    const response = await this.calendar.events.patch({
      calendarId: 'primary',
      eventId,
      requestBody: updates,
      sendNotifications: true,
    });

    return response.data;
  }

  async deleteEvent(eventId: string, sendNotifications = true) {
    await this.calendar.events.delete({
      calendarId: 'primary',
      eventId,
      sendNotifications,
    });
  }

  async listUpcomingEvents(maxResults = 10): Promise<calendar_v3.Schema$Event[]> {
    const response = await this.calendar.events.list({
      calendarId: 'primary',
      timeMin: new Date().toISOString(),
      maxResults,
      singleEvents: true,
      orderBy: 'startTime',
    });

    return response.data.items || [];
  }
}

// services/integrations/google/google-maps.service.ts
import { Client } from '@googlemaps/google-maps-services-js';
import { Injectable } from '@nestjs/common';

@Injectable()
export class GoogleMapsService {
  private client: Client;
  private apiKey: string;

  constructor(private configService: ConfigService) {
    this.client = new Client({});
    this.apiKey = configService.get('GOOGLE_MAPS_API_KEY');
  }

  async geocodeAddress(address: string): Promise<{
    lat: number;
    lng: number;
    formattedAddress: string;
    placeId: string;
  }> {
    const response = await this.client.geocode({
      params: {
        address,
        key: this.apiKey,
      },
    });

    if (response.data.results.length === 0) {
      throw new Error('Address not found');
    }

    const result = response.data.results[0];
    return {
      lat: result.geometry.location.lat,
      lng: result.geometry.location.lng,
      formattedAddress: result.formatted_address,
      placeId: result.place_id,
    };
  }

  async reverseGeocode(lat: number, lng: number): Promise<{
    address: string;
    city: string;
    state: string;
    country: string;
    postalCode: string;
  }> {
    const response = await this.client.reverseGeocode({
      params: {
        latlng: { lat, lng },
        key: this.apiKey,
      },
    });

    if (response.data.results.length === 0) {
      throw new Error('Location not found');
    }

    const result = response.data.results[0];
    const components = result.address_components;

    return {
      address: result.formatted_address,
      city: this.extractComponent(components, 'locality'),
      state: this.extractComponent(components, 'administrative_area_level_1'),
      country: this.extractComponent(components, 'country'),
      postalCode: this.extractComponent(components, 'postal_code'),
    };
  }

  async getPlaceDetails(placeId: string): Promise<any> {
    const response = await this.client.placeDetails({
      params: {
        place_id: placeId,
        fields: [
          'name',
          'formatted_address',
          'geometry',
          'photos',
          'types',
          'website',
          'formatted_phone_number',
          'opening_hours',
          'rating',
          'user_ratings_total',
        ],
        key: this.apiKey,
      },
    });

    return response.data.result;
  }

  async searchNearbyVenues(
    location: { lat: number; lng: number },
    radius: number = 5000,
    type: string = 'establishment'
  ): Promise<any[]> {
    const response = await this.client.placesNearby({
      params: {
        location,
        radius,
        type,
        key: this.apiKey,
      },
    });

    return response.data.results;
  }

  async calculateDistance(
    origins: string[],
    destinations: string[],
    mode: 'driving' | 'walking' | 'bicycling' | 'transit' = 'driving'
  ): Promise<any> {
    const response = await this.client.distancematrix({
      params: {
        origins,
        destinations,
        mode,
        key: this.apiKey,
      },
    });

    return response.data;
  }

  private extractComponent(components: any[], type: string): string {
    const component = components.find(c => c.types.includes(type));
    return component?.long_name || '';
  }
}

Payment Gateway Integrations
typescript// services/integrations/payments/stripe.service.ts
import Stripe from 'stripe';
import { Injectable } from '@nestjs/common';

@Injectable()
export class StripeService {
  private stripe: Stripe;

  constructor(private configService: ConfigService) {
    this.stripe = new Stripe(configService.get('STRIPE_SECRET_KEY'), {
      apiVersion: '2023-10-16',
      typescript: true,
    });
  }

  async createPaymentIntent(data: {
    amount: number;
    currency: string;
    customerId?: string;
    metadata?: Record<string, string>;
    paymentMethodTypes?: string[];
  }): Promise<Stripe.PaymentIntent> {
    const paymentIntent = await this.stripe.paymentIntents.create({
      amount: Math.round(data.amount * 100), // Convert to cents
      currency: data.currency.toLowerCase(),
      customer: data.customerId,
      metadata: {
        ...data.metadata,
        platform: 'artistrsvp',
        timestamp: new Date().toISOString(),
      },
      payment_method_types: data.paymentMethodTypes || ['card'],
      capture_method: 'automatic',
    });

    return paymentIntent;
  }

  async createCustomer(data: {
    email: string;
    name: string;
    phone?: string;
    metadata?: Record<string, string>;
  }): Promise<Stripe.Customer> {
    return await this.stripe.customers.create({
      email: data.email,
      name: data.name,
      phone: data.phone,
      metadata: data.metadata,
    });
  }

  async createSubscription(data: {
    customerId: string;
    priceId: string;
    trialDays?: number;
    metadata?: Record<string, string>;
  }): Promise<Stripe.Subscription> {
    return await this.stripe.subscriptions.create({
      customer: data.customerId,
      items: [{ price: data.priceId }],
      trial_period_days: data.trialDays,
      metadata: data.metadata,
      payment_behavior: 'default_incomplete',
      expand: ['latest_invoice.payment_intent'],
    });
  }

  async createConnectAccount(data: {
    email: string;
    type: 'express' | 'standard';
    country: string;
    businessProfile: {
      name: string;
      product_description: string;
    };
  }): Promise<Stripe.Account> {
    return await this.stripe.accounts.create({
      type: data.type,
      country: data.country,
      email: data.email,
      capabilities: {
        card_payments: { requested: true },
        transfers: { requested: true },
      },
      business_profile: data.businessProfile,
    });
  }

  async createPaymentLink(data: {
    priceId: string;
    quantity?: number;
    metadata?: Record<string, string>;
    afterCompletion?: {
      type: 'redirect';
      redirect: { url: string };
    };
  }): Promise<Stripe.PaymentLink> {
    return await this.stripe.paymentLinks.create({
      line_items: [
        {
          price: data.priceId,
          quantity: data.quantity || 1,
        },
      ],
      metadata: data.metadata,
      after_completion: data.afterCompletion || {
        type: 'redirect',
        redirect: { url: `${process.env.FRONTEND_URL}/payment/success` },
      },
    });
  }

  async processRefund(data: {
    paymentIntentId: string;
    amount?: number;
    reason?: string;
    metadata?: Record<string, string>;
  }): Promise<Stripe.Refund> {
    return await this.stripe.refunds.create({
      payment_intent: data.paymentIntentId,
      amount: data.amount ? Math.round(data.amount * 100) : undefined,
      reason: data.reason as Stripe.RefundCreateParams.Reason,
      metadata: data.metadata,
    });
  }

  async handleWebhook(
    payload: string | Buffer,
    signature: string
  ): Promise<Stripe.Event> {
    const webhookSecret = this.configService.get('STRIPE_WEBHOOK_SECRET');
    
    try {
      const event = this.stripe.webhooks.constructEvent(
        payload,
        signature,
        webhookSecret
      );
      
      return event;
    } catch (error) {
      throw new Error(`Webhook signature verification failed: ${error.message}`);
    }
  }
}

// services/integrations/payments/razorpay.service.ts
import Razorpay from 'razorpay';
import crypto from 'crypto';
import { Injectable } from '@nestjs/common';

@Injectable()
export class RazorpayService {
  private razorpay: Razorpay;
  private keySecret: string;

  constructor(private configService: ConfigService) {
    this.razorpay = new Razorpay({
      key_id: configService.get('RAZORPAY_KEY_ID'),
      key_secret: configService.get('RAZORPAY_KEY_SECRET'),
    });
    
    this.keySecret = configService.get('RAZORPAY_KEY_SECRET');
  }

  async createOrder(data: {
    amount: number;
    currency: string;
    receipt: string;
    notes?: Record<string, string>;
  }): Promise<any> {
    const order = await this.razorpay.orders.create({
      amount: Math.round(data.amount * 100), // Convert to paise
      currency: data.currency,
      receipt: data.receipt,
      notes: {
        ...data.notes,
        platform: 'artistrsvp',
      },
    });

    return order;
  }

  async capturePayment(paymentId: string, amount: number): Promise<any> {
    return await this.razorpay.payments.capture(
      paymentId,
      Math.round(amount * 100)
    );
  }

  async createRefund(data: {
    paymentId: string;
    amount?: number;
    notes?: Record<string, string>;
  }): Promise<any> {
    return await this.razorpay.payments.refund(data.paymentId, {
      amount: data.amount ? Math.round(data.amount * 100) : undefined,
      notes: data.notes,
    });
  }

  async createVirtualAccount(data: {
    name: string;
    email: string;
    contact: string;
    description?: string;
  }): Promise<any> {
    return await this.razorpay.virtualAccounts.create({
      receivers: {
        types: ['bank_account'],
      },
      description: data.description || 'ArtistRSVP Wallet',
      customer_id: await this.createCustomer(data),
      notes: {
        name: data.name,
        email: data.email,
      },
    });
  }

  async createPayout(data: {
    accountNumber: string;
    amount: number;
    currency: string;
    mode: 'IMPS' | 'NEFT' | 'RTGS' | 'UPI';
    purpose: string;
    notes?: Record<string, string>;
  }): Promise<any> {
    return await this.razorpay.payouts.create({
      account_number: data.accountNumber,
      amount: Math.round(data.amount * 100),
      currency: data.currency,
      mode: data.mode,
      purpose: data.purpose,
      queue_if_low_balance: true,
      notes: data.notes,
    });
  }

  verifyWebhookSignature(
    body: string,
    signature: string,
    secret: string = this.keySecret
  ): boolean {
    const expectedSignature = crypto
      .createHmac('sha256', secret)
      .update(body)
      .digest('hex');

    return expectedSignature === signature;
  }

  private async createCustomer(data: {
    name: string;
    email: string;
    contact: string;
  }): Promise<string> {
    const customer = await this.razorpay.customers.create({
      name: data.name,
      email: data.email,
      contact: data.contact,
    });

    return customer.id;
  }
}
Service Level Agreements
SLA Configuration
typescript// packages/shared-config/src/sla.config.ts
export interface SLAConfig {
  service: string;
  endpoints: EndpointSLA[];
  availability: {
    target: number; // Percentage (e.g., 99.9)
    measurementPeriod: 'daily' | 'weekly' | 'monthly';
  };
  performance: {
    responseTime: ResponseTimeSLA;
    throughput: ThroughputSLA;
  };
  errorBudget: {
    errorRate: number; // Percentage
    period: 'hour' | 'day' | 'week';
  };
}

interface EndpointSLA {
  path: string;
  method: string;
  criticality: 'critical' | 'high' | 'medium' | 'low';
  responseTime: {
    p50: number; // milliseconds
    p95: number;
    p99: number;
  };
  errorRate: number; // Percentage
}

interface ResponseTimeSLA {
  p50: number;
  p95: number;
  p99: number;
}

interface ThroughputSLA {
  requestsPerSecond: number;
  concurrentConnections: number;
}

// Service-specific SLA definitions
export const serviceSLAs: Record<string, SLAConfig> = {
  'auth-service': {
    service: 'auth-service',
    endpoints: [
      {
        path: '/auth/whatsapp/send-otp',
        method: 'POST',
        criticality: 'critical',
        responseTime: { p50: 200, p95: 500, p99: 1000 },
        errorRate: 0.1,
      },
      {
        path: '/auth/whatsapp/verify-otp',
        method: 'POST',
        criticality: 'critical',
        responseTime: { p50: 100, p95: 300, p99: 500 },
        errorRate: 0.1,
      },
      {
        path: '/auth/refresh-token',
        method: 'POST',
        criticality: 'high',
        responseTime: { p50: 50, p95: 150, p99: 300 },
        errorRate: 0.5,
      },
    ],
    availability: {
      target: 99.9,
      measurementPeriod: 'monthly',
    },
    performance: {
      responseTime: { p50: 100, p95: 300, p99: 500 },
      throughput: {
        requestsPerSecond: 1000,
        concurrentConnections: 5000,
      },
    },
    errorBudget: {
      errorRate: 0.1,
      period: 'day',
    },
  },
  
  'rfp-service': {
    service: 'rfp-service',
    endpoints: [
      {
        path: '/rfp/create',
        method: 'POST',
        criticality: 'high',
        responseTime: { p50: 300, p95: 800, p99: 1500 },
        errorRate: 0.5,
      },
      {
        path: '/rfp/search',
        method: 'GET',
        criticality: 'medium',
        responseTime: { p50: 200, p95: 600, p99: 1200 },
        errorRate: 1.0,
      },
      {
        path: '/rfp/:id/respond',
        method: 'POST',
        criticality: 'high',
        responseTime: { p50: 400, p95: 1000, p99: 2000 },
        errorRate: 0.5,
      },
    ],
    availability: {
      target: 99.5,
      measurementPeriod: 'monthly',
    },
    performance: {
      responseTime: { p50: 300, p95: 800, p99: 1500 },
      throughput: {
        requestsPerSecond: 500,
        concurrentConnections: 2000,
      },
    },
    errorBudget: {
      errorRate: 0.5,
      period: 'day',
    },
  },
  
  'payment-service': {
    service: 'payment-service',
    endpoints: [
      {
        path: '/payments/process',
        method: 'POST',
        criticality: 'critical',
        responseTime: { p50: 500, p95: 2000, p99: 5000 },
        errorRate: 0.01,
      },
      {
        path: '/payments/refund',
        method: 'POST',
        criticality: 'critical',
        responseTime: { p50: 300, p95: 1000, p99: 3000 },
        errorRate: 0.01,
      },
    ],
    availability: {
      target: 99.99,
      measurementPeriod: 'monthly',
    },
    performance: {
      responseTime: { p50: 400, p95: 1500, p99: 4000 },
      throughput: {
        requestsPerSecond: 200,
        concurrentConnections: 1000,
      },
    },
    errorBudget: {
      errorRate: 0.01,
      period: 'hour',
    },
  },
};

// SLA monitoring implementation
export class SLAMonitor {
  private metrics: Map<string, ServiceMetrics> = new Map();

  constructor(
    private cloudWatch: CloudWatchClient,
    private slaConfig: SLAConfig
  ) {}

  async checkSLA(): Promise<SLAStatus> {
    const metrics = await this.getMetrics();
    const violations: SLAViolation[] = [];

    // Check availability
    if (metrics.availability < this.slaConfig.availability.target) {
      violations.push({
        type: 'availability',
        expected: this.slaConfig.availability.target,
        actual: metrics.availability,
        severity: 'critical',
      });
    }

    // Check response times
    for (const endpoint of this.slaConfig.endpoints) {
      const endpointMetrics = metrics.endpoints.get(endpoint.path);
      
      if (!endpointMetrics) continue;

      if (endpointMetrics.p99 > endpoint.responseTime.p99) {
        violations.push({
          type: 'response_time',
          endpoint: endpoint.path,
          expected: endpoint.responseTime.p99,
          actual: endpointMetrics.p99,
          severity: this.getSeverity(endpoint.criticality),
        });
      }

      if (endpointMetrics.errorRate > endpoint.errorRate) {
        violations.push({
          type: 'error_rate',
          endpoint: endpoint.path,
          expected: endpoint.errorRate,
          actual: endpointMetrics.errorRate,
          severity: this.getSeverity(endpoint.criticality),
        });
      }
    }

    return {
      service: this.slaConfig.service,
      status: violations.length === 0 ? 'healthy' : 'degraded',
      violations,
      metrics,
      timestamp: new Date(),
    };
  }

  private async getMetrics(): Promise<ServiceMetrics> {
    // Implementation to fetch metrics from CloudWatch
    // This is a simplified example
    const response = await this.cloudWatch.send(
      new GetMetricStatisticsCommand({
        Namespace: 'ArtistRSVP',
        MetricName: 'Latency',
        Dimensions: [
          { Name: 'ServiceName', Value: this.slaConfig.service },
        ],
        StartTime: new Date(Date.now() - 3600000), // Last hour
        EndTime: new Date(),
        Period: 300, // 5 minutes
        Statistics: ['Average', 'Maximum'],
      })
    );

    // Process and return metrics
    return {
      availability: 99.95, // Example
      endpoints: new Map(),
      errorRate: 0.05,
      throughput: 450,
    };
  }

  private getSeverity(criticality: string): 'critical' | 'high' | 'medium' | 'low' {
    return criticality as any;
  }
}

AWS Lambda/API Gateway Development Guidelines
Lambda Function Structure
typescript// apps/backend/[service-name]/src/lambda.ts
import { NestFactory } from '@nestjs/core';
import { ExpressAdapter } from '@nestjs/platform-express';
import serverlessExpress from '@vendia/serverless-express';
import express from 'express';
import { AppModule } from './app.module';
import { Handler, Context } from 'aws-lambda';
import { Logger } from '@nestjs/common';
import AWSXRay from 'aws-xray-sdk-core';

// Enable X-Ray tracing
AWSXRay.captureHTTPsGlobal(require('http'));
AWSXRay.captureHTTPsGlobal(require('https'));
AWSXRay.capturePromise();

let cachedServer: Handler;

async function bootstrapServer(): Promise<Handler> {
  const expressApp = express();
  const adapter = new ExpressAdapter(expressApp);
  
  const app = await NestFactory.create(AppModule, adapter, {
    logger: ['error', 'warn', 'log'],
  });

  // Global middleware
  app.enableCors({
    origin: process.env.ALLOWED_ORIGINS?.split(',') || '*',
    credentials: true,
  });

  // Trust proxy for API Gateway
  expressApp.set('trust proxy', true);

  await app.init();
  
  return serverlessExpress({ app: expressApp });
}

export const handler: Handler = async (
  event: any,
  context: Context,
  callback: any
) => {
  // Set X-Ray trace header
  if (process.env._X_AMZN_TRACE_ID) {
    AWSXRay.setTraceId(process.env._X_AMZN_TRACE_ID);
  }

  // Log cold start
  if (!cachedServer) {
    Logger.log('Cold start detected', 'Lambda');
  }

  // Reuse connection for container reuse
  context.callbackWaitsForEmptyEventLoop = false;

  if (!cachedServer) {
    cachedServer = await bootstrapServer();
  }

  return cachedServer(event, context, callback);
};

// Health check handler for ALB
export const health: Handler = async () => {
  return {
    statusCode: 200,
    body: JSON.stringify({ status: 'healthy', timestamp: new Date() }),
  };
};
Cold Start Optimization Strategies
typescript// packages/shared-lambda/src/optimizations.ts

// 1. Minimize bundle size
// webpack.config.js
module.exports = {
  mode: 'production',
  target: 'node',
  externals: {
    // Don't bundle AWS SDK - it's available in Lambda runtime
    'aws-sdk': 'aws-sdk',
    '@aws-sdk/client-s3': '@aws-sdk/client-s3',
    '@aws-sdk/client-dynamodb': '@aws-sdk/client-dynamodb',
  },
  optimization: {
    minimize: true,
    // Tree shaking
    usedExports: true,
    sideEffects: false,
  },
};

// 2. Lazy loading for heavy dependencies
export class OptimizedService {
  private _heavyLibrary: any;

  get heavyLibrary() {
    if (!this._heavyLibrary) {
      // Load only when needed
      this._heavyLibrary = require('heavy-library');
    }
    return this._heavyLibrary;
  }
}

// 3. Connection pooling for databases
import { DataSource } from 'typeorm';

let dataSource: DataSource;

export async function getDataSource(): Promise<DataSource> {
  if (!dataSource) {
    dataSource = new DataSource({
      type: 'postgres',
      host: process.env.DB_HOST,
      port: parseInt(process.env.DB_PORT || '5432'),
      username: process.env.DB_USERNAME,
      password: process.env.DB_PASSWORD,
      database: process.env.DB_NAME,
      ssl: { rejectUnauthorized: false },
      synchronize: false,
      logging: false,
      // Optimize for Lambda
      extra: {
        max: 1, // One connection per Lambda container
        idleTimeoutMillis: 30000,
        connectionTimeoutMillis: 2000,
      },
    });

    await dataSource.initialize();
  }

  return dataSource;
}

// 4. Provisioned concurrency configuration
// serverless.yml
functions:
  authService:
    handler: dist/lambda.handler
    memorySize: 1024 # Optimal for most use cases
    timeout: 30
    provisionedConcurrency: 2 # For critical services
    environment:
      NODE_OPTIONS: '--enable-source-maps'
    layers:
      - !Ref SharedDepsLambdaLayer # Shared dependencies layer

// 5. Lambda SnapStart for Java (if using Java services)
// template.yaml
Resources:
  MyFunction:
    Type: AWS::Serverless::Function
    Properties:
      SnapStart:
        ApplyOn: PublishedVersions

// 6. Warm-up strategy
import { Scheduled, ScheduleExpression } from '@nestjs/schedule';

@Injectable()
export class WarmUpService {
  private readonly logger = new Logger(WarmUpService.name);

  @Scheduled(ScheduleExpression.EVERY_5_MINUTES)
  async warmUpCriticalServices() {
    if (process.env.STAGE !== 'production') return;

    const criticalEndpoints = [
      '/api/v1/auth/health',
      '/api/v1/rfp/health',
      '/api/v1/payments/health',
    ];

    for (const endpoint of criticalEndpoints) {
      try {
        await axios.get(`${process.env.API_URL}${endpoint}`, {
          headers: { 'X-Warm-Up': 'true' },
        });
      } catch (error) {
        this.logger.error(`Warm-up failed for ${endpoint}`, error);
      }
    }
  }
}

// 7. Request/Response optimization
export class LambdaOptimizationInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    
    // Skip processing for warm-up requests
    if (request.headers['x-warm-up'] === 'true') {
      return of({ status: 'warm' });
    }

    // Enable HTTP keep-alive for external requests
    if (!request.agent) {
      request.agent = new https.Agent({
        keepAlive: true,
        maxSockets: 50,
      });
    }

    return next.handle();
  }
}

// 8. Caching strategies
import { CacheModule } from '@nestjs/cache-manager';
import * as redisStore from 'cache-manager-redis-store';

@Module({
  imports: [
    CacheModule.register({
      store: redisStore,
      host: process.env.REDIS_HOST,
      port: process.env.REDIS_PORT,
      ttl: 300, // 5 minutes default
      max: 100, // Maximum items in cache
      // Use in-memory cache as fallback
      fallbackStore: 'memory',
    }),
  ],
})
export class OptimizedCacheModule {}
API Gateway Configuration
typescript// infrastructure/api-gateway/api-gateway.yaml
openapi: 3.0.0
info:
  title: ArtistRSVP API Gateway
  version: 1.0.0
  description: |
    Central API Gateway for ArtistRSVP microservices.
    Handles routing, authentication, rate limiting, and monitoring.

x-amazon-apigateway-request-validators:
  all:
    validateRequestBody: true
    validateRequestParameters: true

x-amazon-apigateway-gateway-responses:
  DEFAULT_4XX:
    responseParameters:
      gatewayresponse.header.Access-Control-Allow-Origin: "'*'"
    responseTemplates:
      application/json: |
        {
          "message": $context.error.messageString,
          "type": "$context.error.responseType",
          "statusCode": $context.error.statusCode
        }
  DEFAULT_5XX:
    responseParameters:
      gatewayresponse.header.Access-Control-Allow-Origin: "'*'"
    responseTemplates:
      application/json: |
        {
          "message": "Internal server error",
          "type": "INTERNAL_ERROR",
          "statusCode": 500,
          "requestId": "$context.requestId"
        }

components:
  securitySchemes:
    BearerAuth:
      type: apiKey
      name: Authorization
      in: header
      x-amazon-apigateway-authtype: custom
      x-amazon-apigateway-authorizer:
        type: request
        authorizerUri: !Sub >-
          arn:aws:apigateway:${AWS::Region}:lambda:path/2015-03-31/functions/${AuthorizerFunction.Arn}/invocations
        authorizerResultTtlInSeconds: 300
        identitySource: method.request.header.Authorization

  schemas:
    Error:
      type: object
      properties:
        message:
          type: string
        code:
          type: string
        statusCode:
          type: integer

paths:
  /api/v1/auth/{proxy+}:
    x-amazon-apigateway-any-method:
      parameters:
        - name: proxy
          in: path
          required: true
          schema:
            type: string
      x-amazon-apigateway-integration:
        type: aws_proxy
        httpMethod: POST
        uri: !Sub >-
          arn:aws:apigateway:${AWS::Region}:lambda:path/2015-03-31/functions/${AuthServiceFunction.Arn}/invocations
        requestTemplates:
          application/json: |
            {
              "version": "2.0",
              "routeKey": "$context.httpMethod $context.path",
              "rawPath": "$context.path",
              "rawQueryString": "$context.queryString",
              "headers": $input.json('$.headers'),
              "requestContext": {
                "accountId": "$context.accountId",
                "apiId": "$context.apiId",
                "domainName": "$context.domainName",
                "domainPrefix": "$context.domainPrefix",
                "http": {
                  "method": "$context.httpMethod",
                  "path": "$context.path",
                  "protocol": "$context.protocol",
                  "sourceIp": "$context.identity.sourceIp",
                  "userAgent": "$context.identity.userAgent"
                },
                "requestId": "$context.requestId",
                "routeKey": "$context.routeKey",
                "stage": "$context.stage",
                "time": "$context.requestTime",
                "timeEpoch": $context.requestTimeEpoch
              },
              "body": "$util.escapeJavaScript($input.body)",
              "isBase64Encoded": false
            }

  /api/v1/rfp/{proxy+}:
    x-amazon-apigateway-any-method:
      security:
        - BearerAuth: []
      parameters:
        - name: proxy
          in: path
          required: true
          schema:
            type: string
      x-amazon-apigateway-integration:
        type: aws_proxy
        httpMethod: POST
        uri: !Sub >-
          arn:aws:apigateway:${AWS::Region}:lambda:path/2015-03-31/functions/${RFPServiceFunction.Arn}/invocations

# Rate limiting configuration
x-amazon-apigateway-usage-plan:
  - name: Basic
    description: Basic tier - 100 requests per minute
    apiStages:
      - apiId: !Ref ApiGateway
        stage: prod
    throttle:
      rateLimit: 100
      burstLimit: 200
    quota:
      limit: 10000
      period: DAY
      
  - name: Pro
    description: Pro tier - 500 requests per minute
    apiStages:
      - apiId: !Ref ApiGateway
        stage: prod
    throttle:
      rateLimit: 500
      burstLimit: 1000
    quota:
      limit: 100000
      period: DAY
      
  - name: ProMax
    description: Pro Max tier - 2000 requests per minute
    apiStages:
      - apiId: !Ref ApiGateway
        stage: prod
    throttle:
      rateLimit: 2000
      burstLimit: 4000
    quota:
      limit: 1000000
      period: DAY

# Custom authorizer
// infrastructure/lambda/authorizer.ts
import { APIGatewayAuthorizerResult, APIGatewayTokenAuthorizerEvent } from 'aws-lambda';
import jwt from 'jsonwebtoken';
import { SSMClient, GetParameterCommand } from '@aws-sdk/client-ssm';

const ssmClient = new SSMClient({ region: process.env.AWS_REGION });

let jwtSecret: string;

export const handler = async (
  event: APIGatewayTokenAuthorizerEvent
): Promise<APIGatewayAuthorizerResult> => {
  try {
    // Get JWT secret from Parameter Store
    if (!jwtSecret) {
      const command = new GetParameterCommand({
        Name: '/artistrsvp/jwt-secret',
        WithDecryption: true,
      });
      const response = await ssmClient.send(command);
      jwtSecret = response.Parameter?.Value!;
    }

    // Extract token
    const token = event.authorizationToken.replace('Bearer ', '');
    
    // Verify token
    const decoded = jwt.verify(token, jwtSecret) as any;
    
    // Generate policy
    return generatePolicy(decoded.sub, 'Allow', event.methodArn, {
      userId: decoded.sub,
      email: decoded.email,
      userType: decoded.userType,
      subscriptionTier: decoded.subscriptionTier,
    });
  } catch (error) {
    console.error('Authorization failed:', error);
    return generatePolicy('user', 'Deny', event.methodArn);
  }
};

function generatePolicy(
  principalId: string,
  effect: 'Allow' | 'Deny',
  resource: string,
  context?: Record<string, any>
): APIGatewayAuthorizerResult {
  return {
    principalId,
    policyDocument: {
      Version: '2012-10-17',
      Statement: [
        {
          Action: 'execute-api:Invoke',
          Effect: effect,
          Resource: resource,
        },
      ],
    },
    context,
  };
}
Lambda Layers for Shared Dependencies
typescript// infrastructure/lambda-layers/shared-deps/nodejs/package.json
{
  "name": "shared-deps-layer",
  "version": "1.0.0",
  "description": "Shared dependencies for Lambda functions",
  "dependencies": {
    "@nestjs/common": "^10.0.0",
    "@nestjs/core": "^10.0.0",
    "@nestjs/platform-express": "^10.0.0",
    "aws-sdk": "^2.1400.0",
    "axios": "^1.4.0",
    "class-transformer": "^0.5.1",
    "class-validator": "^0.14.0",
    "rxjs": "^7.8.1",
    "typeorm": "^0.3.17"
  }
}

// Build script
// infrastructure/scripts/build-layer.sh
#!/bin/bash
set -e

LAYER_NAME=$1
LAYER_DIR="infrastructure/lambda-layers/${LAYER_NAME}"

echo "Building Lambda layer: ${LAYER_NAME}"

# Clean previous builds
rm -rf ${LAYER_DIR}/nodejs/node_modules
rm -f ${LAYER_DIR}/${LAYER_NAME}.zip

# Install production dependencies
cd ${LAYER_DIR}/nodejs
npm ci --production

# Remove unnecessary files
find . -name "*.md" -delete
find . -name "*.ts" -delete
find . -name "*.map" -delete
find . -name "test" -type d -exec rm -rf {} +
find . -name "tests" -type d -exec rm -rf {} +
find . -name "docs" -type d -exec rm -rf {} +

# Create layer zip
cd ..
zip -r ${LAYER_NAME}.zip nodejs/

echo "Layer ${LAYER_NAME} built successfully"

// CDK deployment
import * as lambda from 'aws-cdk-lib/aws-lambda';
import * as cdk from 'aws-cdk-lib';

export class LambdaLayersStack extends cdk.Stack {
  public readonly sharedDepsLayer: lambda.LayerVersion;

  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    this.sharedDepsLayer = new lambda.LayerVersion(this, 'SharedDepsLayer', {
      code: lambda.Code.fromAsset('infrastructure/lambda-layers/shared-deps/shared-deps.zip'),
      compatibleRuntimes: [lambda.Runtime.NODEJS_20_X],
      description: 'Shared dependencies for all Lambda functions',
      layerVersionName: 'artistrsvp-shared-deps',
    });

    // Output layer ARN
    new cdk.CfnOutput(this, 'SharedDepsLayerArn', {
      value: this.sharedDepsLayer.layerVersionArn,
      exportName: 'SharedDepsLayerArn',
    });
  }
}
Developer-Specific Best Practices
Code Quality Standards
typescript// .eslintrc.js
module.exports = {
  parser: '@typescript-eslint/parser',
  parserOptions: {
    project: 'tsconfig.json',
    tsconfigRootDir: __dirname,
    sourceType: 'module',
  },
  plugins: ['@typescript-eslint/eslint-plugin', 'prettier', 'import'],
  extends: [
    'plugin:@typescript-eslint/recommended',
    'plugin:prettier/recommended',
    'plugin:import/errors',
    'plugin:import/warnings',
    'plugin:import/typescript',
  ],
  root: true,
  env: {
    node: true,
    jest: true,
  },
  ignorePatterns: ['.eslintrc.js', 'dist', 'node_modules'],
  rules: {
    '@typescript-eslint/interface-name-prefix': 'off',
    '@typescript-eslint/explicit-function-return-type': 'error',
    '@typescript-eslint/explicit-module-boundary-types': 'error',
    '@typescript-eslint/no-explicit-any': 'error',
    '@typescript-eslint/no-unused-vars': ['error', { argsIgnorePattern: '^_' }],
    '@typescript-eslint/naming-convention': [
      'error',
      {
        selector: 'interface',
        format: ['PascalCase'],
        custom: {
          regex: '^I[A-Z]',
          match: true,
        },
      },
    ],
    'import/order': [
      'error',
      {
        groups: ['builtin', 'external', 'internal', 'parent', 'sibling', 'index'],
        'newlines-between': 'always',
        alphabetize: {
          order: 'asc',
          caseInsensitive: true,
        },
      },
    ],
    'no-console': ['error', { allow: ['warn', 'error'] }],
    'prettier/prettier': 'error',
  },
};

// .prettierrc
{
  "singleQuote": true,
  "trailingComma": "es5",
  "tabWidth": 2,
  "semi": true,
  "printWidth": 100,
  "bracketSpacing": true,
  "arrowParens": "always",
  "endOfLine": "lf"
}

// commitlint.config.js
module.exports = {
  extends: ['@commitlint/config-conventional'],
  rules: {
    'type-enum': [
      2,
      'always',
      [
        'feat',     // New feature
        'fix',      // Bug fix
        'docs',     // Documentation
        'style',    // Formatting
        'refactor', // Code refactoring
        'perf',     // Performance improvements
        'test',     // Tests
        'chore',    // Maintenance
        'revert',   // Revert commits
        'build',    // Build system
        'ci',       // CI/CD
      ],
    ],
    'subject-case': [2, 'never', ['upper-case', 'pascal-case']],
    'header-max-length': [2, 'always', 100],
  },
};
Git Workflow
bash# .gitmessage
# Type: feat, fix, docs, style, refactor, perf, test, chore, revert
# Scope: service name or feature area
# Subject: imperative mood, no period at end

# <type>(<scope>): <subject>
# 
# <body>
# 
# <footer>

# Example:
# feat(rfp-service): add WhatsApp response capability
# 
# - Integrate WhatsApp Cloud API for RFP responses
# - Add message template management
# - Implement webhook handling for incoming messages
# 
# Closes #123

# Branch naming convention
feature/ARSVP-123-whatsapp-integration
bugfix/ARSVP-456-auth-token-expiry
hotfix/ARSVP-789-payment-processing
chore/ARSVP-321-update-dependencies

Development Environment Setup
bash# scripts/setup-dev.sh
#!/bin/bash
set -e

echo "Setting up ArtistRSVP development environment..."

# Check prerequisites
command -v node >/dev/null 2>&1 || { echo "Node.js is required but not installed. Aborting." >&2; exit 1; }
command -v docker >/dev/null 2>&1 || { echo "Docker is required but not installed. Aborting." >&2; exit 1; }

# Install Node.js dependencies
echo "Installing dependencies..."
npm install

# Set up git hooks
echo "Setting up git hooks..."
npm run prepare

# Copy environment files
echo "Setting up environment files..."
if [ ! -f .env ]; then
  cp .env.example .env
  echo "Please update .env with your configuration"
fi

# Start local services
echo "Starting local services..."
docker-compose -f docker-compose.dev.yml up -d

# Run database migrations
echo "Running database migrations..."
npm run migration:run

# Seed development data
echo "Seeding development data..."
npm run seed:dev

echo "Development environment setup complete!"
echo "Run 'npm run dev' to start the development servers"
Testing Standards
typescript// Example test file structure
// apps/backend/rfp-service/src/rfp.service.spec.ts
import { Test, TestingModule } from '@nestjs/testing';
import { RFPService } from './rfp.service';
import { getRepositoryToken } from '@nestjs/typeorm';
import { RFP } from './entities/rfp.entity';
import { Repository } from 'typeorm';
import { AIService } from '@artistrsvp/shared-ai';
import { EventBusService } from '@artistrsvp/shared-events';

describe('RFPService', () => {
  let service: RFPService;
  let repository: Repository<RFP>;
  let aiService: AIService;
  let eventBus: EventBusService;

  const mockRepository = {
    create: jest.fn(),
    save: jest.fn(),
    findOne: jest.fn(),
    find: jest.fn(),
  };

  const mockAIService = {
    generateRFPContent: jest.fn(),
    matchArtistsToRFP: jest.fn(),
  };

  const mockEventBus = {
    publish: jest.fn(),
  };

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [
        RFPService,
        {
          provide: getRepositoryToken(RFP),
          useValue: mockRepository,
        },
        {
          provide: AIService,
          useValue: mockAIService,
        },
        {
          provide: EventBusService,
          useValue: mockEventBus,
        },
      ],
    }).compile();

    service = module.get<RFPService>(RFPService);
    repository = module.get<Repository<RFP>>(getRepositoryToken(RFP));
    aiService = module.get<AIService>(AIService);
    eventBus = module.get<EventBusService>(EventBusService);
  });

  afterEach(() => {
    jest.clearAllMocks();
  });

  describe('createRFP', () => {
    it('should create an RFP with AI-generated content', async () => {
      // Arrange
      const createRFPDto = {
        eventType: 'Wedding',
        eventDate: new Date('2024-06-01'),
        location: 'Mumbai, India',
        guestCount: 200,
        budgetRange: { min: 50000, max: 100000, currency: 'INR' },
        preferences: ['Live Band', 'Classical Music'],
        aiGenerate: true,
      };

      const generatedContent = 'AI-generated RFP content...';
      const mockRFP = { id: 'rfp-123', ...createRFPDto };

      mockAIService.generateRFPContent.mockResolvedValue(generatedContent);
      mockRepository.create.mockReturnValue(mockRFP);
      mockRepository.save.mockResolvedValue(mockRFP);

      // Act
      const result = await service.createRFP(createRFPDto, 'user-123');

      // Assert
      expect(aiService.generateRFPContent).toHaveBeenCalledWith({
        eventType: createRFPDto.eventType,
        date: createRFPDto.eventDate.toISOString(),
        location: createRFPDto.location,
        guestCount: createRFPDto.guestCount,
        preferences: createRFPDto.preferences,
        budget: createRFPDto.budgetRange,
      });

      expect(repository.create).toHaveBeenCalledWith({
        ...createRFPDto,
        description: generatedContent,
        createdBy: 'user-123',
      });

      expect(eventBus.publish).toHaveBeenCalledWith(
        expect.objectContaining({
          eventType: 'RFP_CREATED',
          source: 'rfp-service',
        })
      );

      expect(result).toEqual(mockRFP);
    });

    it('should handle errors gracefully', async () => {
      // Arrange
      const createRFPDto = {
        eventType: 'Concert',
        eventDate: new Date('2024-07-01'),
        location: 'Delhi, India',
        guestCount: 1000,
        budgetRange: { min: 100000, max: 200000, currency: 'INR' },
        aiGenerate: true,
      };

      mockAIService.generateRFPContent.mockRejectedValue(
        new Error('AI service unavailable')
      );

      // Act & Assert
      await expect(
        service.createRFP(createRFPDto, 'user-123')
      ).rejects.toThrow('Failed to generate RFP content');

      expect(repository.save).not.toHaveBeenCalled();
      expect(eventBus.publish).not.toHaveBeenCalled();
    });
  });

  describe('matchArtists', () => {
    it('should return AI-matched artists for an RFP', async () => {
      // Test implementation
    });
  });
});

// E2E test example
// apps/backend/rfp-service/test/rfp.e2e-spec.ts
import { Test, TestingModule } from '@nestjs/testing';
import { INestApplication } from '@nestjs/common';
import * as request from 'supertest';
import { AppModule } from '../src/app.module';

describe('RFP Controller (e2e)', () => {
  let app: INestApplication;
  let authToken: string;

  beforeAll(async () => {
    const moduleFixture: TestingModule = await Test.createTestingModule({
      imports: [AppModule],
    }).compile();

    app = moduleFixture.createNestApplication();
    await app.init();

    // Get auth token
    const authResponse = await request(app.getHttpServer())
      .post('/api/v1/auth/login')
      .send({
        email: 'test@artistrsvp.com',
        password: 'testpassword',
      });

    authToken = authResponse.body.accessToken;
  });

  afterAll(async () => {
    await app.close();
  });

  describe('/api/v1/rfp (POST)', () => {
    it('should create a new RFP', async () => {
      const createRFPDto = {
        title: 'Wedding Reception - Need Live Band',
        eventType: 'Wedding',
        eventDate: '2024-06-01T18:00:00Z',
        location: {
          address: '123 Wedding Venue',
          city: 'Mumbai',
          state: 'Maharashtra',
          country: 'India',
          coordinates: {
            lat: 19.0760,
            lng: 72.8777,
          },
        },
        guestCount: 200,
        budgetRange: {
          min: 50000,
          max: 100000,
          currency: 'INR',
        },
        requirements: [
          {
            type: 'music',
            description: 'Live band with classical and Bollywood repertoire',
          },
        ],
        rfpType: 'automated',
        aiMatchingEnabled: true,
        whatsappNotifications: true,
      };

      const response = await request(app.getHttpServer())
        .post('/api/v1/rfp/create')
        .set('Authorization', `Bearer ${authToken}`)
        .send(createRFPDto)
        .expect(201);

      expect(response.body).toMatchObject({
        id: expect.any(String),
        title: createRFPDto.title,
        status: 'DRAFT',
        createdAt: expect.any(String),
      });
    });

    it('should validate required fields', async () => {
      const invalidRFP = {
        title: 'Test RFP', // Missing required fields
      };

      const response = await request(app.getHttpServer())
        .post('/api/v1/rfp/create')
        .set('Authorization', `Bearer ${authToken}`)
        .send(invalidRFP)
        .expect(400);

      expect(response.body).toMatchObject({
        statusCode: 400,
        message: expect.arrayContaining([
          expect.stringContaining('eventDate'),
          expect.stringContaining('location'),
        ]),
      });
    });
  });
});

Documentation Standards
typescript/**
 * RFP Service
 * 
 * Handles all RFP-related operations including creation, matching,
 * and response management. Integrates with AI services for intelligent
 * matching and WhatsApp for notifications.
 * 
 * @module RFPService
 * @requires AIService
 * @requires EventBusService
 * @requires WhatsAppService
 */
@Injectable()
export class RFPService {
  private readonly logger = new Logger(RFPService.name);

  constructor(
    @InjectRepository(RFP)
    private rfpRepository: Repository<RFP>,
    private aiService: AIService,
    private eventBus: EventBusService,
    private whatsappService: WhatsAppService
  ) {}

  /**
   * Creates a new RFP with optional AI-generated content
   * 
   * @param {CreateRFPDto} createRFPDto - RFP creation data
   * @param {string} userId - ID of the user creating the RFP
   * @returns {Promise<RFP>} Created RFP entity
   * @throws {BadRequestException} If validation fails
   * @throws {ServiceUnavailableException} If AI service is unavailable
   * 
   * @example
   * const rfp = await rfpService.createRFP({
   *   title: 'Wedding Reception',
   *   eventType: 'Wedding',
   *   eventDate: new Date('2024-06-01'),
   *   location: { city: 'Mumbai', country: 'India' },
   *   budgetRange: { min: 50000, max: 100000, currency: 'INR' },
   *   aiGenerate: true
   * }, 'user-123');
   */
  async createRFP(createRFPDto: CreateRFPDto, userId: string): Promise<RFP> {
    // Implementation
  }
}
Performance Monitoring
typescript// packages/shared-monitoring/src/performance.interceptor.ts
import { Injectable, NestInterceptor, ExecutionContext, CallHandler } from '@nestjs/common';
import { Observable } from 'rxjs';
import { tap } from 'rxjs/operators';
import { MetricsService } from './metrics.service';

@Injectable()
export class PerformanceInterceptor implements NestInterceptor {
  constructor(private metricsService: MetricsService) {}

  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    const response = context.switchToHttp().getResponse();
    const startTime = Date.now();

    return next.handle().pipe(
      tap({
        next: () => {
          const duration = Date.now() - startTime;
          const route = request.route?.path || request.url;
          
          this.metricsService.recordHttpRequest({
            method: request.method,
            route,
            statusCode: response.statusCode,
            duration,
          });

          // Log slow requests
          if (duration > 1000) {
            console.warn(`Slow request detected: ${request.method} ${route} took ${duration}ms`);
          }
        },
        error: (error) => {
          const duration = Date.now() - startTime;
          const route = request.route?.path || request.url;
          
          this.metricsService.recordHttpRequest({
            method: request.method,
            route,
            statusCode: error.status || 500,
            duration,
            error: error.message,
          });
        },
      })
    );
  }
}
Conclusion
This comprehensive architecture document provides the complete blueprint for building and maintaining the ArtistRSVP.com platform. It ensures consistency, scalability, and maintainability across all services and teams.
Key Takeaways

AI-First & WhatsApp-First: Every service integrates AI capabilities and WhatsApp functionality
Domain-Driven Design: Clear service boundaries with well-defined responsibilities
Serverless Architecture: Optimized for scalability and cost-effectiveness
Event-Driven Communication: Asynchronous patterns for better resilience
Comprehensive Testing: Unit, integration, and E2E testing standards
Performance Optimization: Cold start mitigation and caching strategies
Security by Design: Authentication, authorization, and data protection at every layer
Developer Experience: Clear guidelines, tooling, and documentation

Next Steps

Use this document as the single source of truth for all development
Keep it updated as the platform evolves
Ensure all developers and AI assistants follow these guidelines
Regular architecture reviews to maintain alignment

Remember: Every line of code should align with these architectural principles and patterns.
