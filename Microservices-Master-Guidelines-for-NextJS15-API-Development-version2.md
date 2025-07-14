# EventfulHQ Microservices Master Guidelines - Complete API Development Reference

## Executive Summary

This document serves as the **definitive guide for AI-assisted AWS Lambda and API Gateway development** for the EventfulHQ B2B2C SaaS platform. EventfulHQ is a **WhatsApp-First, AI-First** global events platform supporting multi-domain routing, 10 international languages, and comprehensive third-party integrations. Every API endpoint must align with the MatDash frontend theme (NextJS 15) and follow these patterns to ensure **consistency**, **scalability**, **performance**, and **seamless deployment**.

## Table of Contents

1. [Platform Overview & Architecture](#platform-overview--architecture)
2. [Multi-Domain & Internationalization Strategy](#multi-domain--internationalization-strategy)
3. [Complete Tech Stack & Boundaries](#complete-tech-stack--boundaries)
4. [Domain-Driven Microservices with Routes](#domain-driven-microservices-with-routes)
5. [Database Architecture & Naming Conventions](#database-architecture--naming-conventions)
6. [Lambda Function Development Standards](#lambda-function-development-standards)
7. [API Gateway Configuration & Patterns](#api-gateway-configuration--patterns)
8. [Authentication & Authorization System](#authentication--authorization-system)
9. [Third-Party Integrations](#third-party-integrations)
10. [Frontend Integration Guidelines](#frontend-integration-guidelines)
11. [Swagger Documentation Standards](#swagger-documentation-standards)
12. [Performance & SLA Requirements](#performance--sla-requirements)
13. [Developer Portal Architecture](#developer-portal-architecture)
14. [SuperAdmin Management System](#superadmin-management-system)
15. [Service Communication Patterns](#service-communication-patterns)
16. [Error Handling & Logging](#error-handling--logging)
17. [Testing & Quality Standards](#testing--quality-standards)
18. [Deployment & CI/CD Pipeline](#deployment--cicd-pipeline)
19. [Frontend Components Wishlist](#frontend-components-wishlist)
20. [Best Practices & Code Examples](#best-practices--code-examples)

## Platform Overview & Architecture

### Core Platform Identity
- **Platform Name**: EventfulHQ
- **Primary Domain**: eventfulhq.com
- **Purpose**: Global B2B2C events marketplace connecting event organizers, artists, venues, and attendees
- **Architecture**: Serverless microservices on AWS Lambda with API Gateway

### Architectural Principles

1. **WhatsApp-First Design**
   - Primary authentication channel via WhatsApp OTP
   - Real-time notifications through WhatsApp Business API
   - Event sharing and RSVP management via WhatsApp
   - Customer support integration

2. **AI-First Implementation**
   - Multi-LLM support (AWS Bedrock + direct integrations)
   - Intelligent event recommendations
   - Automated content generation
   - Smart pricing and demand forecasting

3. **Multi-Domain Architecture**
   - Geographic routing based on user location
   - Custom domain support for white-labeled microsites
   - SEO-optimized landing pages per region

4. **Internationalization (i18n)**
   - 10 supported languages with smart defaults
   - RTL support for Arabic
   - Locale-specific content prioritization

5. **Event-Driven Communication**
   - AWS EventBridge for service orchestration
   - Asynchronous processing for scalability
   - Real-time updates via WebSockets

## Multi-Domain & Internationalization Strategy

### Geographic Domain Routing

```typescript
// Domain mapping configuration
export const REGIONAL_DOMAINS = {
  'US': 'eventfulamerica.com',
  'AE,SA,QA,KW,BH,OM': 'eventfularabia.com', // GCC Countries
  'IN': 'eventfulindia.com',
  'GB,FR,DE,IT,ES,NL,BE,AT,SE,DK,FI,NO,CH,IE,PT,GR,CZ,PL,HU,RO': 'eventfuleurope.com', // EU + UK
  'AU,NZ': 'eventfulaustralia.com',
  'DEFAULT': 'eventfulhq.com'
};

// Language configuration
export const SUPPORTED_LANGUAGES = {
  'en': 'English',
  'zh': 'Mandarin Chinese',
  'es': 'Spanish',
  'pt': 'Portuguese',
  'fr': 'French',
  'ja': 'Japanese',
  'ar': 'Standard Arabic',
  'hi': 'Hindi',
  'ru': 'Russian',
  'de': 'German'
};
```

### Domain Routing Lambda Function

```typescript
// services/domain-router/src/handlers/route-domain.ts
import { CloudFrontRequestEvent, CloudFrontRequestResult } from 'aws-lambda';
import { getCountryFromIP } from '../utils/geo-location';
import { REGIONAL_DOMAINS } from '../config/domains';

export const handler = async (
  event: CloudFrontRequestEvent
): Promise<CloudFrontRequestResult> => {
  const request = event.Records[0].cf.request;
  const headers = request.headers;
  
  // Check if user has domain preference in cookies
  const cookies = parseCookies(headers.cookie);
  if (cookies.preferredDomain) {
    return redirectToDomain(cookies.preferredDomain, request);
  }
  
  // Get user's country from CloudFront headers
  const country = headers['cloudfront-viewer-country']?.[0]?.value;
  
  // Find appropriate regional domain
  const regionalDomain = getRegionalDomain(country);
  
  if (regionalDomain !== getCurrentDomain(request)) {
    return redirectToDomain(regionalDomain, request);
  }
  
  return request;
};
```

## Complete Tech Stack & Boundaries

### Backend Stack (Serverless)
```yaml
Runtime: Node.js 20.x LTS
Framework: NestJS 10.x
API Gateway: AWS HTTP API v2
Functions: AWS Lambda
Database:
  Primary: PostgreSQL 15 (RDS Aurora Serverless v2)
  Cache: Redis (ElastiCache)
  Search: OpenSearch 2.x
  Analytics: Redshift Serverless
  NoSQL: DynamoDB (for real-time features)
ORM: TypeORM 0.3.x with strict naming conventions
API Docs: Swagger/OpenAPI 3.1 (auto-generated)
Message Queue: 
  - AWS SQS (standard operations)
  - AWS EventBridge (event bus)
  - AWS IoT Core (real-time updates)
File Storage: S3 with CloudFront CDN
Authentication: JWT + AWS Cognito
Monitoring: 
  - AWS X-Ray (tracing)
  - CloudWatch (logs/metrics)
  - DataDog (APM)
Infrastructure: AWS CDK v2
Container Registry: ECR for Lambda layers
```

### Frontend Stack (Aligned with MatDash)
```yaml
Framework: Next.js 15.0.0 (App Router)
UI Library: React 19.0.0
Theme: MatDash Pro (Material-UI based)
State Management: 
  - Zustand (client state)
  - TanStack Query (server state)
Styling: 
  - Material-UI v5
  - Tailwind CSS 3.4 (utilities)
  - Emotion (CSS-in-JS)
Forms: React Hook Form + Yup
i18n: next-i18next + i18next
Components: 
  - MUI Components
  - Custom EventfulHQ components
  - Storybook 7.x documentation
Testing:
  - Jest + React Testing Library
  - Playwright (E2E)
  - Cypress (integration)
Build Tools:
  - Turbo (monorepo)
  - ESBuild (bundling)
  - SWC (transpilation)
```

### AI & Integration Stack
```yaml
WhatsApp: WhatsApp Cloud API v18
AI Services:
  Primary: AWS Bedrock (Claude, Llama, Mistral)
  Secondary: Direct API integrations
    - OpenAI GPT-4
    - Anthropic Claude
    - Google Gemini
    - Meta Llama
    - Perplexity
    - Grok
    - Microsoft Copilot
Vector DB: Amazon OpenSearch with k-NN
Calendar: 
  - Google Calendar API
  - Microsoft Graph (Outlook)
  - CalDAV (Apple Calendar)
Payments:
  - Stripe Connect
  - Razorpay
  - PayPal
  - Square
Accounting:
  - QuickBooks API
  - Zoho Books API
CRM:
  - Zoho CRM
  - Zoho Desk
Communications:
  - Twilio (SMS/Voice)
  - Gupshup (WhatsApp)
  - SendGrid (Email)
Ticketing:
  - Ticketmaster API
  - Eventbrite API
  - StubHub API
  - BookMyShow API
  - District API
Automation:
  - Zapier Integration
  - n8n Webhooks
  - Make.com API
Job Boards:
  - LinkedIn API
  - Indeed API
  - Glassdoor API
  - Naukri API
  - Upwork API
```

## Domain-Driven Microservices with Routes

### Service Organization

All microservices are organized as **AppFeatures** - each service represents a controllable feature that can be enabled/disabled per subscription tier or geographic region.

### Complete Service Map with Routes

#### 1. Identity & Access Management Domain

##### AuthService (`/api/v1/auth`)
**Purpose**: WhatsApp-first authentication, JWT management, multi-domain session handling

```typescript
// Lambda: eventfulhq-auth-service
POST   /auth/whatsapp/send-otp         - Send OTP via WhatsApp
POST   /auth/whatsapp/verify-otp       - Verify WhatsApp OTP
POST   /auth/email/send-otp            - Send OTP via Email (fallback)
POST   /auth/email/verify-otp          - Verify Email OTP
POST   /auth/social/google             - Google OAuth
POST   /auth/social/apple              - Apple Sign In
POST   /auth/social/facebook           - Facebook Login
POST   /auth/refresh                   - Refresh tokens
POST   /auth/logout                    - Logout (all devices optional)
POST   /auth/biometric/register        - Register biometric (mobile)
POST   /auth/mfa/enable                - Enable 2FA
GET    /auth/session                   - Get current session
PUT    /auth/preferences/domain        - Update preferred domain
PUT    /auth/preferences/language      - Update preferred language
DELETE /auth/sessions/all              - Revoke all sessions
```

##### UserService (`/api/v1/users`)
**Purpose**: User profiles, preferences, multi-language settings, geographic preferences

```typescript
// Lambda: eventfulhq-user-service
GET    /users/profile                  - Get current user profile
GET    /users/profile/:userId          - Get user profile by ID
PUT    /users/profile                  - Update profile
POST   /users/profile/ai-enhance       - AI profile enhancement
GET    /users/public/:username         - Get public profile
POST   /users/verify/phone             - Verify phone number
POST   /users/verify/email             - Verify email
POST   /users/verify/documents         - KYC document verification
GET    /users/settings                 - Get user settings
PUT    /users/settings                 - Update settings
GET    /users/api-keys                 - List API keys
POST   /users/api-keys                 - Generate API key
DELETE /users/api-keys/:keyId          - Revoke API key
POST   /users/subscription/upgrade     - Upgrade subscription
GET    /users/subscription/status      - Subscription details
POST   /users/export-data              - GDPR data export
DELETE /users/account                  - Delete account
GET    /users/recommendations          - AI user recommendations
POST   /users/bulk-import              - Bulk user import (admin)
```

##### PermissionService (`/api/v1/permissions`)
**Purpose**: RBAC, SuperAdmin management, feature flags by region

```typescript
// Lambda: eventfulhq-permission-service
GET    /permissions/user/:userId       - Get user permissions
GET    /permissions/role/:roleId       - Get role permissions
POST   /permissions/assign             - Assign permission
DELETE /permissions/revoke             - Revoke permission
GET    /permissions/features           - Get feature flags
PUT    /permissions/features           - Update feature flags
GET    /permissions/superadmin/types   - List SuperAdmin types
POST   /permissions/superadmin/assign  - Assign SuperAdmin role
GET    /permissions/audit-log          - Permission change history
```

#### 2. Event Management Domain

##### EventService (`/api/v1/events`)
**Purpose**: Core event lifecycle, multi-language content, regional event management

```typescript
// Lambda: eventfulhq-event-service
POST   /events                         - Create event
GET    /events                         - List events (with filters)
GET    /events/:eventId                - Get event details
PUT    /events/:eventId                - Update event
DELETE /events/:eventId                - Cancel event
POST   /events/:eventId/publish        - Publish draft
POST   /events/:eventId/clone          - Clone event
GET    /events/:eventId/analytics      - Event analytics
POST   /events/:eventId/translate      - AI translate content
GET    /events/trending                - Trending events by region
GET    /events/recommendations         - AI recommendations
POST   /events/bulk-create             - Bulk import events
GET    /events/categories              - Event categories
GET    /events/search                  - Advanced search
POST   /events/:eventId/staff          - Add staff requirements
GET    /events/:eventId/staff          - Get staffing needs
POST   /events/:eventId/volunteers     - Request volunteers
PUT    /events/:eventId/seo            - Update SEO metadata
GET    /events/microsite/:slug         - Get microsite data
POST   /events/:eventId/qr-code        - Generate event QR
```

##### TicketingService (`/api/v1/tickets`)
**Purpose**: Ticket management, QR codes, wallet integration, dynamic pricing

```typescript
// Lambda: eventfulhq-ticketing-service
POST   /tickets/types                  - Create ticket type
GET    /tickets/event/:eventId         - Get event tickets
POST   /tickets/purchase               - Purchase tickets
GET    /tickets/validate/:qrCode       - Validate QR code
POST   /tickets/transfer               - Transfer ownership
GET    /tickets/user                   - User's tickets
POST   /tickets/refund                 - Process refund
GET    /tickets/:ticketId/qr           - Get ticket QR code
POST   /tickets/wallet/google          - Add to Google Wallet
POST   /tickets/wallet/apple           - Add to Apple Wallet
GET    /tickets/analytics              - Sales analytics
POST   /tickets/dynamic-pricing        - AI pricing engine
POST   /tickets/bulk-generate          - Bulk ticket creation
POST   /tickets/scan                   - Scan ticket (check-in)
GET    /tickets/export                 - Export attendee list
POST   /tickets/integrate/ticketmaster - Sync with Ticketmaster
POST   /tickets/integrate/eventbrite   - Sync with Eventbrite
```

##### CalendarService (`/api/v1/calendar`)
**Purpose**: Multi-calendar integration, availability management, timezone handling

```typescript
// Lambda: eventfulhq-calendar-service
GET    /calendar/availability          - Check availability
POST   /calendar/block                 - Block time slots
POST   /calendar/sync/google           - Sync Google Calendar
POST   /calendar/sync/outlook          - Sync Outlook Calendar
POST   /calendar/sync/apple            - Sync Apple Calendar
GET    /calendar/events                - Get calendar events
POST   /calendar/event                 - Create calendar event
PUT    /calendar/event/:eventId        - Update calendar event
DELETE /calendar/event/:eventId        - Remove from calendar
GET    /calendar/conflicts             - Check conflicts
POST   /calendar/bulk-sync             - Sync multiple events
GET    /calendar/suggestions           - AI time suggestions
POST   /calendar/subscribe             - Subscribe to calendar feed
GET    /calendar/ics/:eventId          - Export ICS file
```

##### VenueService (`/api/v1/venues`)
**Purpose**: Venue management, availability, virtual venues, 360° tours

```typescript
// Lambda: eventfulhq-venue-service
POST   /venues                         - Add venue
GET    /venues                         - List venues
GET    /venues/:venueId                - Venue details
PUT    /venues/:venueId                - Update venue
DELETE /venues/:venueId                - Remove venue
GET    /venues/:venueId/availability   - Check availability
POST   /venues/:venueId/book           - Book venue
GET    /venues/:venueId/calendar       - Venue calendar
POST   /venues/:venueId/tour-360       - Upload 360° tour
GET    /venues/search                  - Search venues
GET    /venues/trending                - Trending venues
POST   /venues/:venueId/review         - Add review
GET    /venues/:venueId/analytics      - Venue analytics
POST   /venues/virtual                 - Create virtual venue
GET    /venues/microsite/:slug         - Venue microsite
```

##### RSVPService (`/api/v1/rsvp`)
**Purpose**: RSVP management, guest lists, check-in system

```typescript
// Lambda: eventfulhq-rsvp-service
POST   /rsvp/create                    - Create RSVP
GET    /rsvp/event/:eventId            - Get event RSVPs
PUT    /rsvp/:rsvpId                   - Update RSVP
DELETE /rsvp/:rsvpId                   - Cancel RSVP
POST   /rsvp/bulk-invite               - Bulk invitations
GET    /rsvp/:rsvpId/qr                - Get RSVP QR code
POST   /rsvp/check-in                  - Check-in guest
GET    /rsvp/guest-list/:eventId       - Export guest list
POST   /rsvp/whatsapp/send             - Send via WhatsApp
GET    /rsvp/analytics/:eventId        - RSVP analytics
POST   /rsvp/reminders                 - Send reminders
```

#### 3. Creator & Talent Domain

##### ArtistService (`/api/v1/artists`)
**Purpose**: Artist profiles, portfolios, booking management, ratings

```typescript
// Lambda: eventfulhq-artist-service
POST   /artists/profile                - Create artist profile
GET    /artists                        - List artists
GET    /artists/:artistId              - Artist details
PUT    /artists/:artistId              - Update profile
GET    /artists/search                 - Search artists
GET    /artists/trending               - Trending artists
POST   /artists/:artistId/book         - Book artist
GET    /artists/:artistId/calendar     - Availability
POST   /artists/:artistId/media        - Upload media
GET    /artists/:artistId/reviews      - Get reviews
POST   /artists/:artistId/review       - Add review
GET    /artists/categories             - Artist categories
GET    /artists/recommendations        - AI recommendations
POST   /artists/:artistId/verify       - Verify artist
GET    /artists/microsite/:username    - Artist microsite
POST   /artists/ai-match               - AI matching for events
```

##### ExpertService (`/api/v1/experts`)
**Purpose**: Event experts, consultants, service providers

```typescript
// Lambda: eventfulhq-expert-service
POST   /experts/profile                - Create expert profile
GET    /experts                        - List experts
GET    /experts/:expertId              - Expert details
PUT    /experts/:expertId              - Update profile
GET    /experts/search                 - Search experts
GET    /experts/categories             - Expert categories
POST   /experts/:expertId/book         - Book expert
GET    /experts/:expertId/calendar     - Availability
POST   /experts/:expertId/consultation  - Schedule consultation
GET    /experts/trending               - Trending experts
GET    /experts/microsite/:username    - Expert microsite
```

##### TeamService (`/api/v1/teams`)
**Purpose**: Team management, collaboration, job board, mini-ATS

```typescript
// Lambda: eventfulhq-team-service
POST   /teams                          - Create team
GET    /teams                          - List teams
GET    /teams/:teamId                  - Team details
PUT    /teams/:teamId                  - Update team
DELETE /teams/:teamId                  - Delete team
POST   /teams/:teamId/members          - Add member
DELETE /teams/:teamId/members/:userId  - Remove member
PUT    /teams/:teamId/permissions      - Update permissions
GET    /teams/trending                 - Trending teams
GET    /teams/microsite/:slug          - Team microsite

# Job Board & ATS Features
POST   /teams/:teamId/jobs             - Post job
GET    /teams/:teamId/jobs             - List team jobs
GET    /teams/jobs                     - All job listings
GET    /teams/jobs/:jobId              - Job details
PUT    /teams/jobs/:jobId              - Update job
DELETE /teams/jobs/:jobId              - Remove job
POST   /teams/jobs/:jobId/apply        - Apply for job
GET    /teams/jobs/:jobId/applications - View applications
PUT    /teams/applications/:appId      - Update application status
POST   /teams/jobs/integrate/linkedin  - Post to LinkedIn
POST   /teams/jobs/integrate/indeed    - Post to Indeed
POST   /teams/jobs/integrate/glassdoor - Post to Glassdoor
POST   /teams/jobs/integrate/naukri    - Post to Naukri
GET    /teams/ats/pipeline             - ATS pipeline view
POST   /teams/ats/interview            - Schedule interview
```

#### 4. Communication Domain

##### MessagingService (`/api/v1/messages`)
**Purpose**: Real-time messaging, WhatsApp integration, group chats

```typescript
// Lambda: eventfulhq-messaging-service
POST   /messages/send                  - Send message
GET    /messages/conversations         - List conversations
GET    /messages/conversation/:id      - Get conversation
POST   /messages/whatsapp/send         - Send via WhatsApp
POST   /messages/whatsapp/template     - Send template message
GET    /messages/unread                - Unread count
PUT    /messages/read/:messageId       - Mark as read
POST   /messages/group                 - Create group
PUT    /messages/group/:groupId        - Update group
POST   /messages/group/:groupId/join   - Join group
DELETE /messages/:messageId            - Delete message
POST   /messages/ai-translate          - Translate message
POST   /messages/broadcast             - Broadcast message
GET    /messages/templates             - Message templates
```

##### NotificationService (`/api/v1/notifications`)
**Purpose**: Multi-channel notifications, preferences, delivery tracking

```typescript
// Lambda: eventfulhq-notification-service
POST   /notifications/send             - Send notification
GET    /notifications                  - User notifications
PUT    /notifications/read/:id         - Mark as read
PUT    /notifications/preferences      - Update preferences
POST   /notifications/bulk             - Bulk send
GET    /notifications/templates        - List templates
POST   /notifications/schedule         - Schedule notification
DELETE /notifications/:id              - Delete notification
GET    /notifications/delivery/:id     - Delivery status
POST   /notifications/test             - Test notification
PUT    /notifications/subscribe        - Subscribe to topics
DELETE /notifications/unsubscribe      - Unsubscribe
```

##### EmailService (`/api/v1/emails`)
**Purpose**: Transactional emails, campaigns, analytics

```typescript
// Lambda: eventfulhq-email-service
POST   /emails/send                    - Send email
POST   /emails/campaign                - Create campaign
GET    /emails/campaigns               - List campaigns
PUT    /emails/campaign/:id            - Update campaign
POST   /emails/template                - Create template
GET    /emails/templates               - List templates
GET    /emails/analytics/:campaignId   - Campaign analytics
POST   /emails/bulk                    - Bulk send
GET    /emails/bounces                 - Bounce list
PUT    /emails/unsubscribe             - Unsubscribe
POST   /emails/verify                  - Verify email
GET    /emails/domains                 - Verified domains
POST   /emails/domain/verify           - Verify domain
```

#### 5. Commerce Domain

##### PaymentService (`/api/v1/payments`)
**Purpose**: Multi-gateway payments, split payments, refunds

```typescript
// Lambda: eventfulhq-payment-service
POST   /payments/intent                - Create payment intent
POST   /payments/process               - Process payment
GET    /payments/:paymentId            - Payment status
POST   /payments/refund                - Process refund
GET    /payments/methods               - Available methods
POST   /payments/method/add            - Add payment method
DELETE /payments/method/:methodId      - Remove method
POST   /payments/split                 - Split payment
GET    /payments/history               - Payment history
POST   /payments/subscription          - Create subscription
PUT    /payments/subscription/:id      - Update subscription
DELETE /payments/subscription/:id      - Cancel subscription
POST   /payments/webhook/stripe        - Stripe webhook
POST   /payments/webhook/razorpay      - Razorpay webhook
GET    /payments/invoice/:invoiceId    - Get invoice
POST   /payments/payout                - Create payout
```

##### WalletService (`/api/v1/wallet`)
**Purpose**: Digital wallet, credits system, rewards

```typescript
// Lambda: eventfulhq-wallet-service
GET    /wallet/balance                 - Get balance
POST   /wallet/topup                   - Add funds
POST   /wallet/withdraw                - Withdraw funds
GET    /wallet/transactions            - Transaction history
POST   /wallet/transfer                - Transfer to user
GET    /wallet/statement               - Generate statement
POST   /wallet/credits/earn            - Earn credits
POST   /wallet/credits/redeem          - Redeem credits
GET    /wallet/rewards                 - Available rewards
POST   /wallet/rewards/claim           - Claim reward
GET    /wallet/referrals               - Referral earnings
```

##### BillingService (`/api/v1/billing`)
**Purpose**: Subscriptions, invoicing, usage tracking

```typescript
// Lambda: eventfulhq-billing-service
GET    /billing/subscription           - Current subscription
POST   /billing/subscribe              - New subscription
PUT    /billing/upgrade                - Upgrade plan
PUT    /billing/downgrade              - Downgrade plan
DELETE /billing/cancel                 - Cancel subscription
GET    /billing/invoices               - List invoices
GET    /billing/invoice/:id            - Download invoice
POST   /billing/usage                  - Report usage
GET    /billing/usage/current          - Current usage
POST   /billing/promo/apply            - Apply promo code
GET    /billing/tax                    - Tax calculation
POST   /billing/accounting/quickbooks  - Sync QuickBooks
POST   /billing/accounting/zoho        - Sync Zoho Books
```

#### 6. Content & Discovery Domain

##### ContentService (`/api/v1/content`)
**Purpose**: Content management, AI generation, SEO optimization

```typescript
// Lambda: eventfulhq-content-service
POST   /content/create                 - Create content
GET    /content/:contentId             - Get content
PUT    /content/:contentId             - Update content
DELETE /content/:contentId             - Delete content
GET    /content/feed                   - Content feed
POST   /content/ai-generate            - AI generate content
POST   /content/ai-enhance             - AI enhance content
POST   /content/translate              - Translate content
GET    /content/trending               - Trending content
POST   /content/:contentId/like        - Like content
POST   /content/:contentId/share       - Share content
POST   /content/:contentId/comment     - Add comment
GET    /content/seo/:slug              - SEO optimized page
POST   /content/import                 - Import content
GET    /content/export                 - Export content
```

##### DiscoveryService (`/api/v1/discovery`)
**Purpose**: Search, recommendations, trending content by region

```typescript
// Lambda: eventfulhq-discovery-service
GET    /discovery/search               - Universal search
GET    /discovery/trending/events      - Trending events
GET    /discovery/trending/artists     - Trending artists
GET    /discovery/trending/venues      - Trending venues
GET    /discovery/trending/teams       - Trending teams
GET    /discovery/trending/experts     - Trending experts
GET    /discovery/destinations         - Trending destinations
GET    /discovery/categories           - Browse by category
POST   /discovery/recommendations      - Personalized recommendations
GET    /discovery/near-me              - Nearby discoveries
POST   /discovery/ai-match             - AI matching
GET    /discovery/leaderboards         - Category leaderboards
GET    /discovery/filters              - Available filters
POST   /discovery/save-search          - Save search query
```

##### MediaService (`/api/v1/media`)
**Purpose**: File uploads, image/video processing, CDN management

```typescript
// Lambda: eventfulhq-media-service
POST   /media/upload                   - Upload file
POST   /media/upload/chunk             - Chunked upload
GET    /media/:mediaId                 - Get media URL
DELETE /media/:mediaId                 - Delete media
POST   /media/process                  - Process media
POST   /media/ai-enhance               - AI enhancement
POST   /media/ai-generate              - AI generation
GET    /media/gallery/:userId          - User gallery
POST   /media/optimize                 - Optimize for web
POST   /media/thumbnail                - Generate thumbnail
POST   /media/watermark                - Add watermark
GET    /media/usage                    - Storage usage
POST   /media/bulk-upload              - Bulk upload
POST   /media/import/url               - Import from URL
```

#### 7. Campaign Management Domain

##### CampaignService (`/api/v1/campaigns`)
**Purpose**: Marketing campaigns, influencer campaigns, analytics

```typescript
// Lambda: eventfulhq-campaign-service
POST   /campaigns                      - Create campaign
GET    /campaigns                      - List campaigns
GET    /campaigns/:campaignId          - Campaign details
PUT    /campaigns/:campaignId          - Update campaign
DELETE /campaigns/:campaignId          - Delete campaign
POST   /campaigns/:campaignId/launch   - Launch campaign
POST   /campaigns/:campaignId/pause    - Pause campaign
GET    /campaigns/:campaignId/analytics- Campaign analytics
POST   /campaigns/:campaignId/staff    - Add staff needs
GET    /campaigns/:campaignId/roi      - ROI analysis
POST   /campaigns/ai-optimize          - AI optimization
GET    /campaigns/templates            - Campaign templates
POST   /campaigns/:campaignId/clone    - Clone campaign
POST   /campaigns/influencer           - Create influencer campaign
GET    /campaigns/performance          - Performance metrics
```

#### 8. Integration Domain

##### IntegrationService (`/api/v1/integrations`)
**Purpose**: Third-party integrations, webhooks, API management

```typescript
// Lambda: eventfulhq-integration-service
GET    /integrations                   - List integrations
POST   /integrations/connect           - Connect integration
DELETE /integrations/:integrationId    - Disconnect
GET    /integrations/:integrationId    - Integration details
PUT    /integrations/:integrationId    - Update settings
POST   /integrations/webhook           - Register webhook
DELETE /integrations/webhook/:id       - Remove webhook
GET    /integrations/logs              - Integration logs
POST   /integrations/sync              - Manual sync
GET    /integrations/zapier/triggers  - Zapier triggers
GET    /integrations/zapier/actions   - Zapier actions
POST   /integrations/n8n/webhook       - n8n webhook
POST   /integrations/make/webhook      - Make.com webhook
GET    /integrations/api-usage         - API usage stats
```

##### WebhookService (`/api/v1/webhooks`)
**Purpose**: Webhook management, delivery tracking, retry logic

```typescript
// Lambda: eventfulhq-webhook-service
POST   /webhooks                       - Create webhook
GET    /webhooks                       - List webhooks
GET    /webhooks/:webhookId            - Webhook details
PUT    /webhooks/:webhookId            - Update webhook
DELETE /webhooks/:webhookId            - Delete webhook
POST   /webhooks/:webhookId/test      - Test webhook
GET    /webhooks/:webhookId/logs      - Delivery logs
POST   /webhooks/:webhookId/retry     - Retry failed delivery
PUT    /webhooks/:webhookId/pause     - Pause webhook
PUT    /webhooks/:webhookId/resume    - Resume webhook
GET    /webhooks/events                - Available events
```

#### 9. Analytics & Intelligence Domain

##### AnalyticsService (`/api/v1/analytics`)
**Purpose**: Platform analytics, reporting, business intelligence

```typescript
// Lambda: eventfulhq-analytics-service
GET    /analytics/dashboard            - Dashboard metrics
GET    /analytics/events/:eventId      - Event analytics
GET    /analytics/revenue              - Revenue analytics
GET    /analytics/users                - User analytics
GET    /analytics/geography            - Geographic analytics
POST   /analytics/report               - Generate report
GET    /analytics/reports              - List reports
GET    /analytics/export/:reportId     - Export report
POST   /analytics/schedule             - Schedule report
GET    /analytics/realtime             - Real-time metrics
GET    /analytics/trends               - Trend analysis
POST   /analytics/custom               - Custom query
GET    /analytics/funnel               - Conversion funnel
GET    /analytics/cohorts              - Cohort analysis
```

##### AIService (`/api/v1/ai`)
**Purpose**: AI operations, LLM management, model routing

```typescript
// Lambda: eventfulhq-ai-service
POST   /ai/complete                    - Text completion
POST   /ai/chat                        - Chat completion
POST   /ai/translate                   - Translation
POST   /ai/summarize                   - Summarization
POST   /ai/generate/content            - Content generation
POST   /ai/generate/image              - Image generation
POST   /ai/analyze/sentiment           - Sentiment analysis
POST   /ai/analyze/intent             - Intent detection
POST   /ai/recommendations             - Get recommendations
POST   /ai/embeddings                  - Generate embeddings
GET    /ai/models                      - Available models
PUT    /ai/models/preference           - Set model preference
GET    /ai/usage                       - AI usage stats
POST   /ai/fine-tune                   - Fine-tune model
```

#### 10. Platform Infrastructure Domain

##### MicrositeService (`/api/v1/microsites`)
**Purpose**: White-labeled microsites, custom domains, SEO pages

```typescript
// Lambda: eventfulhq-microsite-service
POST   /microsites/create              - Create microsite
GET    /microsites/:micrositeId        - Get microsite
PUT    /microsites/:micrositeId        - Update microsite
DELETE /microsites/:micrositeId        - Delete microsite
POST   /microsites/:micrositeId/domain - Add custom domain
GET    /microsites/verify/:domain      - Verify domain
POST   /microsites/:micrositeId/publish- Publish microsite
GET    /microsites/themes              - Available themes
POST   /microsites/:micrositeId/seo    - Update SEO settings
GET    /microsites/analytics/:id       - Microsite analytics
POST   /microsites/ai-optimize         - AI SEO optimization
GET    /microsites/templates           - Microsite templates
```

##### AdminService (`/api/v1/admin`)
**Purpose**: Platform administration, SuperAdmin features, system management

```typescript
// Lambda: eventfulhq-admin-service
GET    /admin/dashboard                - Admin dashboard
GET    /admin/users                    - List all users
PUT    /admin/users/:userId            - Admin update user
DELETE /admin/users/:userId            - Admin delete user
GET    /admin/events                   - List all events
PUT    /admin/events/:eventId          - Admin update event
DELETE /admin/events/:eventId          - Admin delete event
POST   /admin/broadcast                - System broadcast
GET    /admin/reports                  - System reports
POST   /admin/feature-flag             - Toggle feature
GET    /admin/audit-log                - System audit log
POST   /admin/maintenance              - Maintenance mode
GET    /admin/health                   - System health
POST   /admin/cache/clear              - Clear cache
GET    /admin/subscriptions            - All subscriptions
PUT    /admin/subscription/:id         - Override subscription
GET    /admin/revenue                  - Revenue overview
POST   /admin/refund                   - Admin refund
GET    /admin/support/tickets          - Support tickets
PUT    /admin/support/ticket/:id       - Update ticket
```

##### ConfigurationService (`/api/v1/config`)
**Purpose**: Dynamic configuration, feature flags, A/B testing

```typescript
// Lambda: eventfulhq-configuration-service
GET    /config/app                     - App configuration
GET    /config/features                - Feature flags
PUT    /config/features                - Update features
GET    /config/region/:region          - Region config
PUT    /config/region/:region          - Update region
GET    /config/languages               - Language settings
PUT    /config/languages               - Update languages
GET    /config/currencies              - Currency settings
PUT    /config/currencies              - Update currencies
GET    /config/integrations            - Integration configs
PUT    /config/integration/:name       - Update integration
GET    /config/sla                     - SLA thresholds
PUT    /config/sla                     - Update SLA
POST   /config/experiment              - Create A/B test
GET    /config/experiments             - List experiments
```

##### AuditService (`/api/v1/audit`)
**Purpose**: Audit logging, compliance, data retention

```typescript
// Lambda: eventfulhq-audit-service
POST   /audit/log                      - Create audit entry
GET    /audit/logs                     - Query audit logs
GET    /audit/user/:userId             - User audit trail
GET    /audit/entity/:type/:id         - Entity audit trail
GET    /audit/compliance               - Compliance report
POST   /audit/export                   - Export audit data
DELETE /audit/purge                    - Purge old logs
GET    /audit/retention                - Retention policy
PUT    /audit/retention                - Update retention
GET    /audit/search                   - Search audit logs
```

##### DeveloperPortalService (`/api/v1/developer`)
**Purpose**: API documentation, developer tools, API key management

```typescript
// Lambda: eventfulhq-developer-portal-service
GET    /developer/docs                 - API documentation
GET    /developer/sdks                 - Available SDKs
GET    /developer/apps                 - Developer apps
POST   /developer/apps                 - Create app
PUT    /developer/apps/:appId          - Update app
DELETE /developer/apps/:appId          - Delete app
GET    /developer/keys                 - API keys
POST   /developer/keys                 - Generate key
DELETE /developer/keys/:keyId          - Revoke key
GET    /developer/usage                - API usage
GET    /developer/limits               - Rate limits
POST   /developer/support              - Developer support
GET    /developer/webhooks             - Webhook docs
GET    /developer/examples             - Code examples
GET    /developer/playground           - API playground
POST   /developer/test                 - Test endpoint
```

## Database Architecture & Naming Conventions

### Database Per Service Pattern

Each microservice has its own database schema to ensure loose coupling and independent scaling.

### PostgreSQL Naming Conventions

```sql
-- Table names: plural, snake_case
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- Column names: snake_case
CREATE TABLE events (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    event_title VARCHAR(255) NOT NULL,
    event_date TIMESTAMP WITH TIME ZONE NOT NULL,
    max_capacity INTEGER DEFAULT 100,
    is_published BOOLEAN DEFAULT FALSE,
    created_by UUID REFERENCES users(id)
);

-- Index names: idx_tablename_columns
CREATE INDEX idx_events_event_date ON events(event_date);
CREATE INDEX idx_events_created_by_published ON events(created_by, is_published);

-- Foreign key names: fk_tablename_referencedtable
ALTER TABLE events 
ADD CONSTRAINT fk_events_users 
FOREIGN KEY (created_by) REFERENCES users(id);

-- Unique constraint names: uq_tablename_columns
ALTER TABLE users 
ADD CONSTRAINT uq_users_email 
UNIQUE (email);

-- Check constraint names: chk_tablename_condition
ALTER TABLE events 
ADD CONSTRAINT chk_events_capacity 
CHECK (max_capacity > 0);

-- Trigger names: trg_tablename_action
CREATE TRIGGER trg_events_update_timestamp
BEFORE UPDATE ON events
FOR EACH ROW
EXECUTE FUNCTION update_updated_at_column();

-- Function names: fn_action_description
CREATE FUNCTION fn_update_timestamp() 
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = CURRENT_TIMESTAMP;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- View names: vw_description
CREATE VIEW vw_active_events AS
SELECT * FROM events 
WHERE is_published = TRUE 
AND event_date > CURRENT_TIMESTAMP;

-- Materialized view names: mvw_description
CREATE MATERIALIZED VIEW mvw_event_statistics AS
SELECT 
    DATE_TRUNC('month', event_date) as month,
    COUNT(*) as event_count,
    SUM(max_capacity) as total_capacity
FROM events
GROUP BY DATE_TRUNC('month', event_date);
```

### TypeORM Entity Standards

```typescript
// Base entity with audit fields
@Entity('eventfulhq_base')
export abstract class BaseEntity {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @CreateDateColumn({ 
    type: 'timestamp with time zone',
    name: 'created_at' 
  })
  createdAt: Date;

  @UpdateDateColumn({ 
    type: 'timestamp with time zone',
    name: 'updated_at' 
  })
  updatedAt: Date;

  @Column({ 
    type: 'uuid', 
    name: 'created_by',
    nullable: true 
  })
  createdBy?: string;

  @Column({ 
    type: 'uuid', 
    name: 'updated_by',
    nullable: true 
  })
  updatedBy?: string;

  @DeleteDateColumn({ 
    type: 'timestamp with time zone',
    name: 'deleted_at' 
  })
  deletedAt?: Date;

  @VersionColumn({ name: 'version' })
  version: number;
}

// Example entity following conventions
@Entity('events')
@Index('idx_events_event_date', ['eventDate'])
@Index('idx_events_status_date', ['status', 'eventDate'])
export class Event extends BaseEntity {
  @Column({ 
    type: 'varchar', 
    length: 255, 
    name: 'event_title' 
  })
  eventTitle: string;

  @Column({ 
    type: 'text', 
    name: 'event_description' 
  })
  eventDescription: string;

  @Column({ 
    type: 'timestamp with time zone',
    name: 'event_date' 
  })
  eventDate: Date;

  @Column({ 
    type: 'integer', 
    name: 'max_capacity',
    default: 100 
  })
  maxCapacity: number;

  @Column({ 
    type: 'enum',
    enum: EventStatus,
    default: EventStatus.DRAFT 
  })
  status: EventStatus;

  @Column({ 
    type: 'jsonb',
    name: 'metadata',
    nullable: true 
  })
  metadata?: Record<string, any>;

  @ManyToOne(() => User, { lazy: true })
  @JoinColumn({ name: 'organizer_id' })
  organizer: User;

  @OneToMany(() => Ticket, ticket => ticket.event)
  tickets: Ticket[];
}
```

### Database Schemas by Service

```yaml
Databases:
  eventfulhq_auth:
    - users
    - user_sessions
    - oauth_providers
    - mfa_settings
    - api_keys
    
  eventfulhq_events:
    - events
    - event_categories
    - event_translations
    - event_staff
    - event_analytics
    
  eventfulhq_tickets:
    - tickets
    - ticket_types
    - ticket_transfers
    - ticket_validations
    - qr_codes
    
  eventfulhq_venues:
    - venues
    - venue_availability
    - venue_bookings
    - venue_amenities
    - venue_media
    
  eventfulhq_artists:
    - artist_profiles
    - artist_categories
    - artist_media
    - artist_bookings
    - artist_reviews
    
  eventfulhq_payments:
    - payment_intents
    - payment_methods
    - transactions
    - refunds
    - payouts
    
  eventfulhq_messaging:
    - conversations
    - messages
    - message_templates
    - whatsapp_sessions
    - broadcast_lists
    
  eventfulhq_content:
    - content_items
    - content_translations
    - content_media
    - content_seo
    - content_analytics
```

## Lambda Function Development Standards

### Lambda Function Template

```typescript
// Template for all Lambda functions
// src/handlers/[operation].handler.ts

import { APIGatewayProxyEventV2, APIGatewayProxyResultV2, Context } from 'aws-lambda';
import { Logger } from '@aws-lambda-powertools/logger';
import { Tracer } from '@aws-lambda-powertools/tracer';
import { Metrics } from '@aws-lambda-powertools/metrics';
import { ValidationError } from '@eventfulhq/common';
import middy from '@middy/core';
import cors from '@middy/http-cors';
import httpErrorHandler from '@middy/http-error-handler';
import httpJsonBodyParser from '@middy/http-json-body-parser';
import validator from '@middy/validator';
import { transpileSchema } from '@middy/validator/transpile';

// Initialize AWS Lambda Powertools
const logger = new Logger({ serviceName: 'eventfulhq-[service-name]' });
const tracer = new Tracer({ serviceName: 'eventfulhq-[service-name]' });
const metrics = new Metrics({ namespace: 'EventfulHQ', serviceName: '[service-name]' });

// Input validation schema
const inputSchema = {
  type: 'object',
  properties: {
    body: {
      type: 'object',
      properties: {
        // Define your input properties
      },
      required: ['field1', 'field2'],
      additionalProperties: false
    }
  }
};

// Business logic handler
const lambdaHandler = async (
  event: APIGatewayProxyEventV2,
  context: Context
): Promise<APIGatewayProxyResultV2> => {
  // Add correlation ID for distributed tracing
  const correlationId = event.headers['x-correlation-id'] || context.requestId;
  logger.appendKeys({ correlationId });
  
  // Extract user context from authorizer
  const userContext = event.requestContext.authorizer?.lambda;
  const { userId, userType, subscriptionTier, preferredLanguage } = userContext || {};
  
  logger.info('Processing request', {
    path: event.rawPath,
    method: event.requestContext.http.method,
    userId,
    userType
  });

  try {
    // Start X-Ray subsegment
    const subsegment = tracer.getSegment()?.addNewSubsegment('[operation-name]');
    
    // Parse and validate input
    const input = JSON.parse(event.body || '{}');
    
    // Add custom metrics
    metrics.addMetric('RequestCount', 1);
    metrics.addMetadata('operation', '[operation-name]');
    metrics.addMetadata('userId', userId);
    
    // TODO: Implement your business logic here
    const result = await processBusinessLogic(input, userContext);
    
    // Close subsegment
    subsegment?.close();
    
    // Return success response
    return {
      statusCode: 200,
      headers: {
        'Content-Type': 'application/json',
        'X-Correlation-ID': correlationId
      },
      body: JSON.stringify({
        success: true,
        data: result,
        timestamp: new Date().toISOString()
      })
    };
    
  } catch (error) {
    // Log error with context
    logger.error('Request failed', error as Error, {
      correlationId,
      userId,
      operation: '[operation-name]'
    });
    
    // Add error metric
    metrics.addMetric('ErrorCount', 1);
    
    // Determine error response
    if (error instanceof ValidationError) {
      return {
        statusCode: 400,
        headers: {
          'Content-Type': 'application/json',
          'X-Correlation-ID': correlationId
        },
        body: JSON.stringify({
          success: false,
          error: {
            code: 'VALIDATION_ERROR',
            message: error.message,
            details: error.details
          }
        })
      };
    }
    
    // Generic error response
    return {
      statusCode: 500,
      headers: {
        'Content-Type': 'application/json',
        'X-Correlation-ID': correlationId
      },
      body: JSON.stringify({
        success: false,
        error: {
          code: 'INTERNAL_ERROR',
          message: 'An internal error occurred',
          correlationId
        }
      })
    };
  } finally {
    // Publish metrics
    metrics.publishStoredMetrics();
  }
};

// Apply middleware
export const handler = middy(lambdaHandler)
  .use(httpJsonBodyParser())
  .use(validator({ 
    eventSchema: transpileSchema(inputSchema),
    ajvOptions: {
      strict: true,
      coerceTypes: true
    }
  }))
  .use(cors({
    origins: process.env.ALLOWED_ORIGINS?.split(',') || ['https://eventfulhq.com'],
    credentials: true
  }))
  .use(httpErrorHandler());

// Business logic implementation
async function processBusinessLogic(
  input: any, 
  userContext: any
): Promise<any> {
  // TODO: Implement actual business logic
  return {
    message: 'Success',
    processedAt: new Date().toISOString()
  };
}
```

### Database Connection Management

```typescript
// src/utils/database.ts
import { DataSource, DataSourceOptions } from 'typeorm';
import { SecretsManager } from '@aws-sdk/client-secrets-manager';
import { entities } from '../entities';

let dataSource: DataSource;
const secretsManager = new SecretsManager({ region: process.env.AWS_REGION });

async function getDatabaseCredentials(): Promise<any> {
  const secret = await secretsManager.getSecretValue({
    SecretId: process.env.DB_SECRET_ARN!
  });
  
  return JSON.parse(secret.SecretString!);
}

export async function getDataSource(): Promise<DataSource> {
  if (!dataSource || !dataSource.isInitialized) {
    const credentials = await getDatabaseCredentials();
    
    const options: DataSourceOptions = {
      type: 'postgres',
      host: process.env.DB_HOST!,
      port: parseInt(process.env.DB_PORT || '5432'),
      username: credentials.username,
      password: credentials.password,
      database: process.env.DB_NAME!,
      entities,
      synchronize: false,
      logging: process.env.STAGE === 'dev',
      ssl: {
        rejectUnauthorized: false
      },
      extra: {
        max: 1, // Lambda should use single connection
        idleTimeoutMillis: 10000,
        connectionTimeoutMillis: 2000
      }
    };
    
    dataSource = new DataSource(options);
    await dataSource.initialize();
  }
  
  return dataSource;
}

// Close connection before Lambda freezes
export async function closeDataSource(): Promise<void> {
  if (dataSource?.isInitialized) {
    await dataSource.destroy();
  }
}
```

### Environment Configuration

```typescript
// src/config/environment.ts
import { z } from 'zod';
import { SSMClient, GetParameterCommand } from '@aws-sdk/client-ssm';

const ssmClient = new SSMClient({ region: process.env.AWS_REGION });

// Environment schema
const envSchema = z.object({
  // AWS
  AWS_REGION: z.string().default('us-east-1'),
  STAGE: z.enum(['dev', 'staging', 'prod']),
  
  // Database
  DB_HOST: z.string(),
  DB_PORT: z.string().default('5432'),
  DB_NAME: z.string(),
  DB_SECRET_ARN: z.string(),
  
  // Redis
  REDIS_HOST: z.string(),
  REDIS_PORT: z.string().default('6379'),
  
  // WhatsApp
  WHATSAPP_API_URL: z.string(),
  WHATSAPP_ACCESS_TOKEN: z.string(),
  WHATSAPP_PHONE_NUMBER_ID: z.string(),
  
  // AI Services
  BEDROCK_MODEL_ID: z.string().default('anthropic.claude-3-sonnet'),
  OPENAI_API_KEY: z.string().optional(),
  
  // Frontend
  FRONTEND_URL: z.string(),
  ALLOWED_ORIGINS: z.string(),
  
  // Monitoring
  DATADOG_API_KEY: z.string().optional(),
  SENTRY_DSN: z.string().optional()
});

type Environment = z.infer<typeof envSchema>;

// Cache for SSM parameters
const paramCache = new Map<string, { value: string; expiry: number }>();

async function getParameter(name: string): Promise<string> {
  const cached = paramCache.get(name);
  if (cached && cached.expiry > Date.now()) {
    return cached.value;
  }
  
  const response = await ssmClient.send(
    new GetParameterCommand({
      Name: name,
      WithDecryption: true
    })
  );
  
  const value = response.Parameter?.Value || '';
  paramCache.set(name, {
    value,
    expiry: Date.now() + 300000 // 5 minutes
  });
  
  return value;
}

export async function getConfig(): Promise<Environment> {
  // Merge environment variables with SSM parameters
  const config = {
    ...process.env,
    WHATSAPP_ACCESS_TOKEN: await getParameter('/eventfulhq/whatsapp/access-token'),
    OPENAI_API_KEY: await getParameter('/eventfulhq/openai/api-key'),
    DATADOG_API_KEY: await getParameter('/eventfulhq/datadog/api-key')
  };
  
  return envSchema.parse(config);
}
```

### Error Handling Standards

```typescript
// src/errors/index.ts
export class BaseError extends Error {
  constructor(
    public code: string,
    message: string,
    public statusCode: number = 500,
    public details?: any
  ) {
    super(message);
    this.name = this.constructor.name;
    Error.captureStackTrace(this, this.constructor);
  }
}

export class ValidationError extends BaseError {
  constructor(message: string, details?: any) {
    super('VALIDATION_ERROR', message, 400, details);
  }
}

export class NotFoundError extends BaseError {
  constructor(resource: string, id?: string) {
    super(
      'NOT_FOUND',
      id ? `${resource} with id ${id} not found` : `${resource} not found`,
      404
    );
  }
}

export class UnauthorizedError extends BaseError {
  constructor(message = 'Unauthorized') {
    super('UNAUTHORIZED', message, 401);
  }
}

export class ForbiddenError extends BaseError {
  constructor(message = 'Forbidden') {
    super('FORBIDDEN', message, 403);
  }
}

export class ConflictError extends BaseError {
  constructor(message: string, details?: any) {
    super('CONFLICT', message, 409, details);
  }
}

export class TooManyRequestsError extends BaseError {
  constructor(message = 'Too many requests', retryAfter?: number) {
    super('TOO_MANY_REQUESTS', message, 429, { retryAfter });
  }
}

export class ServiceUnavailableError extends BaseError {
  constructor(service: string) {
    super('SERVICE_UNAVAILABLE', `${service} is temporarily unavailable`, 503);
  }
}
```

## API Gateway Configuration & Patterns

### API Gateway Structure

```yaml
# serverless.yml configuration
service: eventfulhq-api-gateway

provider:
  name: aws
  runtime: nodejs20.x
  stage: ${opt:stage, 'dev'}
  region: ${opt:region, 'us-east-1'}
  
  httpApi:
    cors:
      allowedOrigins:
        - https://eventfulhq.com
        - https://eventfulamerica.com
        - https://eventfularabia.com
        - https://eventfulindia.com
        - https://eventfuleurope.com
        - https://eventfulaustralia.com
      allowedHeaders:
        - Content-Type
        - Authorization
        - X-Correlation-ID
        - X-Preferred-Language
        - X-Preferred-Domain
      allowedMethods:
        - GET
        - POST
        - PUT
        - DELETE
        - OPTIONS
      allowCredentials: true
      maxAge: 86400
    
    authorizers:
      jwtAuthorizer:
        type: jwt
        identitySource: $request.header.Authorization
        issuerUrl: https://cognito-idp.${aws:region}.amazonaws.com/${self:custom.userPoolId}
        audience:
          - ${self:custom.userPoolClientId}

custom:
  userPoolId: ${cf:eventfulhq-auth-stack.UserPoolId}
  userPoolClientId: ${cf:eventfulhq-auth-stack.UserPoolClientId}
  
  # API Gateway stages
  stages:
    dev:
      throttle:
        rateLimit: 100
        burstLimit: 200
    staging:
      throttle:
        rateLimit: 500
        burstLimit: 1000
    prod:
      throttle:
        rateLimit: 2000
        burstLimit: 4000
        
  # Custom domain configuration
  customDomain:
    domainName: api.${self:custom.domains.${self:provider.stage}}
    certificateArn: ${self:custom.certificates.${self:provider.stage}}
    createRoute53Record: true
    endpointType: regional
    
  domains:
    dev: dev.eventfulhq.com
    staging: staging-api.eventfulhq.com
    prod: eventfulhq.com

functions:
  # Auth Service Routes
  authSendOTP:
    handler: services/auth/handlers/send-otp.handler
    events:
      - httpApi:
          path: /api/v1/auth/whatsapp/send-otp
          method: POST
          
  authVerifyOTP:
    handler: services/auth/handlers/verify-otp.handler
    events:
      - httpApi:
          path: /api/v1/auth/whatsapp/verify-otp
          method: POST
          
  # Event Service Routes
  createEvent:
    handler: services/events/handlers/create-event.handler
    events:
      - httpApi:
          path: /api/v1/events
          method: POST
          authorizer: jwtAuthorizer
          
  getEvents:
    handler: services/events/handlers/list-events.handler
    events:
      - httpApi:
          path: /api/v1/events
          method: GET
          
  # Add all other routes...

resources:
  Resources:
    # API Gateway usage plan for different tiers
    BasicUsagePlan:
      Type: AWS::ApiGateway::UsagePlan
      Properties:
        ApiStages:
          - ApiId: !Ref HttpApi
            Stage: ${self:provider.stage}
        Description: Basic tier - 100 requests per minute
        Quota:
          Limit: 10000
          Period: DAY
        Throttle:
          RateLimit: 100
          BurstLimit: 200
        UsagePlanName: EventfulHQ-Basic-${self:provider.stage}
        
    ProUsagePlan:
      Type: AWS::ApiGateway::UsagePlan
      Properties:
        ApiStages:
          - ApiId: !Ref HttpApi
            Stage: ${self:provider.stage}
        Description: Pro tier - 500 requests per minute
        Quota:
          Limit: 100000
          Period: DAY
        Throttle:
          RateLimit: 500
          BurstLimit: 1000
        UsagePlanName: EventfulHQ-Pro-${self:provider.stage}
        
    ProMaxUsagePlan:
      Type: AWS::ApiGateway::UsagePlan
      Properties:
        ApiStages:
          - ApiId: !Ref HttpApi
            Stage: ${self:provider.stage}
        Description: Pro Max tier - 2000 requests per minute
        Quota:
          Limit: 1000000
          Period: DAY
        Throttle:
          RateLimit: 2000
          BurstLimit: 4000
        UsagePlanName: EventfulHQ-ProMax-${self:provider.stage}
```

### Request/Response Transformations

```typescript
// API Gateway request transformation
export const requestTransformer = {
  // Add common headers
  headers: {
    'X-Correlation-ID': '$context.requestId',
    'X-Source-IP': '$context.identity.sourceIp',
    'X-User-Agent': '$context.identity.userAgent',
    'X-Request-Time': '$context.requestTime',
    'X-Stage': '$context.stage'
  },
  
  // Transform body
  body: {
    #set($body = $input.json('$'))
    "data": $body,
    "context": {
      "requestId": "$context.requestId",
      "stage": "$context.stage",
      "domainName": "$context.domainName",
      "path": "$context.path",
      "method": "$context.httpMethod",
      "sourceIp": "$context.identity.sourceIp",
      "userAgent": "$context.identity.userAgent",
      "requestTime": "$context.requestTime"
    }
  }
};

// Response transformation
export const responseTransformer = {
  // Success response
  200: {
    headers: {
      'Content-Type': 'application/json',
      'X-Correlation-ID': '$context.requestId',
      'Cache-Control': 'no-cache, no-store, must-revalidate'
    },
    body: {
      "success": true,
      "data": $input.json('$'),
      "metadata": {
        "timestamp": "$context.requestTime",
        "version": "1.0.0"
      }
    }
  },
  
  // Error response
  default: {
    headers: {
      'Content-Type': 'application/json',
      'X-Correlation-ID': '$context.requestId'
    },
    body: {
      "success": false,
      "error": {
        "code": "$context.error.responseType",
        "message": "$context.error.message",
        "requestId": "$context.requestId"
      }
    }
  }
};
```

## Authentication & Authorization System

### JWT Token Structure

```typescript
// Token payload interface
export interface JWTPayload {
  // Standard claims
  sub: string;          // User ID
  iat: number;          // Issued at
  exp: number;          // Expiration
  iss: string;          // Issuer (eventfulhq.com)
  aud: string[];        // Audiences
  
  // Custom claims
  email: string;
  phoneNumber?: string;
  whatsappNumber?: string;
  userType: 'fan' | 'artist' | 'expert' | 'venue' | 'organizer';
  subscriptionTier: 'free' | 'basic' | 'pro' | 'proMax' | 'enterprise';
  permissions: string[];
  preferredLanguage: string;
  preferredDomain: string;
  region: string;
  
  // SuperAdmin claims
  isSuperAdmin?: boolean;
  superAdminType?: SuperAdminType;
  superAdminPermissions?: string[];
  
  // Session info
  sessionId: string;
  deviceId?: string;
  loginMethod: 'whatsapp' | 'email' | 'google' | 'apple' | 'facebook';
}

// SuperAdmin types
export enum SuperAdminType {
  GLOBAL_ADMIN = 'GLOBAL_ADMIN',
  REGIONAL_ADMIN = 'REGIONAL_ADMIN',
  CONTENT_ADMIN = 'CONTENT_ADMIN',
  FINANCE_ADMIN = 'FINANCE_ADMIN',
  SUPPORT_ADMIN = 'SUPPORT_ADMIN',
  MARKETING_ADMIN = 'MARKETING_ADMIN',
  TECH_ADMIN = 'TECH_ADMIN',
  SECURITY_ADMIN = 'SECURITY_ADMIN',
  COMPLIANCE_ADMIN = 'COMPLIANCE_ADMIN',
  ANALYTICS_ADMIN = 'ANALYTICS_ADMIN',
  INTEGRATION_ADMIN = 'INTEGRATION_ADMIN'
}
```

### Custom Authorizer Lambda

```typescript
// services/auth/handlers/authorizer.ts
import { APIGatewayAuthorizerEvent, APIGatewayAuthorizerResult } from 'aws-lambda';
import jwt from 'jsonwebtoken';
import jwksClient from 'jwks-rsa';

const client = jwksClient({
  jwksUri: `https://cognito-idp.${process.env.AWS_REGION}.amazonaws.com/${process.env.USER_POOL_ID}/.well-known/jwks.json`,
  cache: true,
  cacheMaxAge: 600000, // 10 minutes
  rateLimit: true,
  jwksRequestsPerMinute: 10
});

export const handler = async (
  event: APIGatewayAuthorizerEvent
): Promise<APIGatewayAuthorizerResult> => {
  try {
    const token = event.authorizationToken?.replace('Bearer ', '');
    
    if (!token) {
      throw new Error('No token provided');
    }
    
    // Decode token to get header
    const decoded = jwt.decode(token, { complete: true });
    if (!decoded || !decoded.header.kid) {
      throw new Error('Invalid token');
    }
    
    // Get signing key
    const key = await client.getSigningKey(decoded.header.kid);
    const signingKey = key.getPublicKey();
    
    // Verify token
    const payload = jwt.verify(token, signingKey, {
      algorithms: ['RS256'],
      issuer: `https://cognito-idp.${process.env.AWS_REGION}.amazonaws.com/${process.env.USER_POOL_ID}`,
      audience: process.env.USER_POOL_CLIENT_ID
    }) as JWTPayload;
    
    // Check subscription tier permissions for the route
    const routePermissions = getRoutePermissions(event.methodArn);
    if (!hasPermission(payload, routePermissions)) {
      throw new Error('Insufficient permissions');
    }
    
    // Return policy
    return generatePolicy(payload.sub, 'Allow', event.methodArn, {
      userId: payload.sub,
      email: payload.email,
      userType: payload.userType,
      subscriptionTier: payload.subscriptionTier,
      permissions: JSON.stringify(payload.permissions),
      preferredLanguage: payload.preferredLanguage,
      preferredDomain: payload.preferredDomain,
      region: payload.region,
      isSuperAdmin: payload.isSuperAdmin?.toString() || 'false',
      superAdminType: payload.superAdminType || ''
    });
    
  } catch (error) {
    console.error('Authorization failed:', error);
    throw new Error('Unauthorized');
  }
};

function generatePolicy(
  principalId: string,
  effect: 'Allow' | 'Deny',
  resource: string,
  context?: Record<string, string>
): APIGatewayAuthorizerResult {
  return {
    principalId,
    policyDocument: {
      Version: '2012-10-17',
      Statement: [
        {
          Action: 'execute-api:Invoke',
          Effect: effect,
          Resource: resource
        }
      ]
    },
    context
  };
}

function getRoutePermissions(methodArn: string): string[] {
  // Extract route from ARN
  const arnParts = methodArn.split(':');
  const apiGatewayArn = arnParts[5].split('/');
  const method = apiGatewayArn[0];
  const route = apiGatewayArn.slice(2).join('/');
  
  // Define route permissions
  const routePermissions: Record<string, string[]> = {
    'POST /api/v1/events': ['events:create'],
    'PUT /api/v1/events/*': ['events:update'],
    'DELETE /api/v1/events/*': ['events:delete'],
    'POST /api/v1/admin/*': ['admin:access'],
    // Add all route permissions...
  };
  
  return routePermissions[`${method} ${route}`] || [];
}

function hasPermission(payload: JWTPayload, required: string[]): boolean {
  // SuperAdmins have all permissions
  if (payload.isSuperAdmin) {
    return true;
  }
  
  // Check if user has required permissions
  return required.every(perm => payload.permissions.includes(perm));
}
```

### API Key Management

```typescript
// services/users/handlers/api-keys.ts
import crypto from 'crypto';
import { DynamoDBClient } from '@aws-sdk/client-dynamodb';
import { DynamoDBDocumentClient, PutCommand, QueryCommand } from '@aws-sdk/lib-dynamodb';

const client = new DynamoDBClient({});
const docClient = DynamoDBDocumentClient.from(client);

export interface APIKey {
  keyId: string;
  userId: string;
  key: string;
  name: string;
  permissions: string[];
  rateLimit: number;
  expiresAt?: Date;
  lastUsedAt?: Date;
  createdAt: Date;
  isActive: boolean;
}

export async function generateAPIKey(
  userId: string,
  name: string,
  permissions: string[],
  expiresInDays?: number
): Promise<APIKey> {
  const keyId = crypto.randomUUID();
  const key = `ehq_${crypto.randomBytes(32).toString('hex')}`;
  const hashedKey = crypto.createHash('sha256').update(key).digest('hex');
  
  const apiKey: APIKey = {
    keyId,
    userId,
    key: hashedKey, // Store hashed version
    name,
    permissions,
    rateLimit: 1000, // Default rate limit
    createdAt: new Date(),
    isActive: true
  };
  
  if (expiresInDays) {
    apiKey.expiresAt = new Date(Date.now() + expiresInDays * 24 * 60 * 60 * 1000);
  }
  
  // Store in DynamoDB
  await docClient.send(new PutCommand({
    TableName: 'eventfulhq-api-keys',
    Item: apiKey
  }));
  
  // Return with actual key (only time it's shown)
  return { ...apiKey, key };
}

export async function validateAPIKey(key: string): Promise<APIKey | null> {
  const hashedKey = crypto.createHash('sha256').update(key).digest('hex');
  
  const response = await docClient.send(new QueryCommand({
    TableName: 'eventfulhq-api-keys',
    IndexName: 'key-index',
    KeyConditionExpression: '#key = :key',
    ExpressionAttributeNames: {
      '#key': 'key'
    },
    ExpressionAttributeValues: {
      ':key': hashedKey
    }
  }));
  
  if (!response.Items || response.Items.length === 0) {
    return null;
  }
  
  const apiKey = response.Items[0] as APIKey;
  
  // Check if expired
  if (apiKey.expiresAt && new Date(apiKey.expiresAt) < new Date()) {
    return null;
  }
  
  // Check if active
  if (!apiKey.isActive) {
    return null;
  }
  
  // Update last used
  await docClient.send(new PutCommand({
    TableName: 'eventfulhq-api-keys',
    Item: {
      ...apiKey,
      lastUsedAt: new Date()
    }
  }));
  
  return apiKey;
}
```

## Third-Party Integrations

### WhatsApp Cloud API Integration

```typescript
// services/integrations/whatsapp/client.ts
import axios, { AxiosInstance } from 'axios';
import { Logger } from '@aws-lambda-powertools/logger';

const logger = new Logger({ serviceName: 'whatsapp-integration' });

export class WhatsAppClient {
  private client: AxiosInstance;
  private phoneNumberId: string;
  
  constructor() {
    this.phoneNumberId = process.env.WHATSAPP_PHONE_NUMBER_ID!;
    this.client = axios.create({
      baseURL: `https://graph.facebook.com/v18.0/${this.phoneNumberId}`,
      headers: {
        'Authorization': `Bearer ${process.env.WHATSAPP_ACCESS_TOKEN}`,
        'Content-Type': 'application/json'
      },
      timeout: 30000
    });
    
    this.setupInterceptors();
  }
  
  private setupInterceptors(): void {
    this.client.interceptors.request.use(
      (config) => {
        logger.info('WhatsApp API Request', {
          method: config.method,
          url: config.url,
          data: config.data
        });
        return config;
      },
      (error) => {
        logger.error('WhatsApp API Request Error', error);
        return Promise.reject(error);
      }
    );
    
    this.client.interceptors.response.use(
      (response) => {
        logger.info('WhatsApp API Response', {
          status: response.status,
          data: response.data
        });
        return response;
      },
      (error) => {
        logger.error('WhatsApp API Response Error', {
          status: error.response?.status,
          data: error.response?.data
        });
        return Promise.reject(error);
      }
    );
  }
  
  async sendOTP(phoneNumber: string, otp: string): Promise<void> {
    await this.client.post('/messages', {
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
    });
  }
  
  async sendEventInvitation(
    phoneNumber: string,
    eventTitle: string,
    eventDate: string,
    eventUrl: string
  ): Promise<void> {
    await this.client.post('/messages', {
      messaging_product: 'whatsapp',
      recipient_type: 'individual',
      to: phoneNumber,
      type: 'template',
      template: {
        name: 'event_invitation',
        language: { code: 'en' },
        components: [
          {
            type: 'header',
            parameters: [
              { type: 'text', text: eventTitle }
            ]
          },
          {
            type: 'body',
            parameters: [
              { type: 'text', text: eventDate },
              { type: 'text', text: eventUrl }
            ]
          },
          {
            type: 'button',
            sub_type: 'url',
            index: '0',
            parameters: [
              { type: 'text', text: eventUrl.split('/').pop() }
            ]
          }
        ]
      }
    });
  }
  
  async sendMessage(phoneNumber: string, message: string): Promise<void> {
    await this.client.post('/messages', {
      messaging_product: 'whatsapp',
      recipient_type: 'individual',
      to: phoneNumber,
      type: 'text',
      text: { body: message }
    });
  }
  
  async sendInteractiveButtons(
    phoneNumber: string,
    body: string,
    buttons: Array<{ id: string; title: string }>
  ): Promise<void> {
    await this.client.post('/messages', {
      messaging_product: 'whatsapp',
      recipient_type: 'individual',
      to: phoneNumber,
      type: 'interactive',
      interactive: {
        type: 'button',
        body: { text: body },
        action: {
          buttons: buttons.map(btn => ({
            type: 'reply',
            reply: { id: btn.id, title: btn.title }
          }))
        }
      }
    });
  }
}
```

### Google Calendar Integration

```typescript
// services/integrations/google/calendar.ts
import { google, calendar_v3 } from 'googleapis';
import { OAuth2Client } from 'google-auth-library';

export class GoogleCalendarService {
  private oauth2Client: OAuth2Client;
  private calendar: calendar_v3.Calendar;
  
  constructor() {
    this.oauth2Client = new google.auth.OAuth2(
      process.env.GOOGLE_CLIENT_ID,
      process.env.GOOGLE_CLIENT_SECRET,
      process.env.GOOGLE_REDIRECT_URL
    );
    
    this.calendar = google.calendar({ 
      version: 'v3', 
      auth: this.oauth2Client 
    });
  }
  
  setUserTokens(tokens: any): void {
    this.oauth2Client.setCredentials(tokens);
  }
  
  async createEvent(eventData: {
    summary: string;
    description: string;
    location: string;
    startTime: Date;
    endTime: Date;
    attendees?: string[];
    conferenceData?: boolean;
  }): Promise<calendar_v3.Schema$Event> {
    const event: calendar_v3.Schema$Event = {
      summary: eventData.summary,
      description: eventData.description,
      location: eventData.location,
      start: {
        dateTime: eventData.startTime.toISOString(),
        timeZone: 'UTC'
      },
      end: {
        dateTime: eventData.endTime.toISOString(),
        timeZone: 'UTC'
      },
      attendees: eventData.attendees?.map(email => ({ email })),
      reminders: {
        useDefault: false,
        overrides: [
          { method: 'email', minutes: 24 * 60 },
          { method: 'popup', minutes: 60 }
        ]
      }
    };
    
    if (eventData.conferenceData) {
      event.conferenceData = {
        createRequest: {
          requestId: crypto.randomUUID(),
          conferenceSolutionKey: { type: 'hangoutsMeet' }
        }
      };
    }
    
    const response = await this.calendar.events.insert({
      calendarId: 'primary',
      requestBody: event,
      conferenceDataVersion: eventData.conferenceData ? 1 : undefined,
      sendNotifications: true
    });
    
    return response.data;
  }
  
  async checkAvailability(
    startTime: Date,
    endTime: Date,
    timeZone = 'UTC'
  ): Promise<boolean> {
    const response = await this.calendar.freebusy.query({
      requestBody: {
        timeMin: startTime.toISOString(),
        timeMax: endTime.toISOString(),
        timeZone,
        items: [{ id: 'primary' }]
      }
    });
    
    const busy = response.data.calendars?.primary?.busy || [];
    return busy.length === 0;
  }
  
  async syncEvents(
    startDate: Date,
    endDate: Date
  ): Promise<calendar_v3.Schema$Event[]> {
    const response = await this.calendar.events.list({
      calendarId: 'primary',
      timeMin: startDate.toISOString(),
      timeMax: endDate.toISOString(),
      singleEvents: true,
      orderBy: 'startTime'
    });
    
    return response.data.items || [];
  }
}
```

### Stripe Payment Integration

```typescript
// services/integrations/stripe/client.ts
import Stripe from 'stripe';
import { Logger } from '@aws-lambda-powertools/logger';

const logger = new Logger({ serviceName: 'stripe-integration' });

export class StripeService {
  private stripe: Stripe;
  
  constructor() {
    this.stripe = new Stripe(process.env.STRIPE_SECRET_KEY!, {
      apiVersion: '2023-10-16',
      typescript: true,
      telemetry: false
    });
  }
  
  async createPaymentIntent(params: {
    amount: number;
    currency: string;
    customerId?: string;
    metadata?: Record<string, string>;
    applicationFeeAmount?: number;
    transferData?: {
      destination: string;
      amount?: number;
    };
  }): Promise<Stripe.PaymentIntent> {
    try {
      const paymentIntent = await this.stripe.paymentIntents.create({
        amount: Math.round(params.amount * 100), // Convert to cents
        currency: params.currency.toLowerCase(),
        customer: params.customerId,
        metadata: {
          ...params.metadata,
          platform: 'eventfulhq',
          timestamp: new Date().toISOString()
        },
        payment_method_types: ['card'],
        capture_method: 'automatic',
        application_fee_amount: params.applicationFeeAmount,
        transfer_data: params.transferData
      });
      
      logger.info('Payment intent created', {
        paymentIntentId: paymentIntent.id,
        amount: params.amount,
        currency: params.currency
      });
      
      return paymentIntent;
    } catch (error) {
      logger.error('Failed to create payment intent', error as Error);
      throw error;
    }
  }
  
  async createConnectedAccount(params: {
    email: string;
    country: string;
    type: 'express' | 'standard';
    businessProfile: {
      name: string;
      productDescription: string;
      supportEmail: string;
      url?: string;
    };
    capabilities?: {
      cardPayments?: boolean;
      transfers?: boolean;
    };
  }): Promise<Stripe.Account> {
    const account = await this.stripe.accounts.create({
      type: params.type,
      country: params.country,
      email: params.email,
      capabilities: {
        card_payments: { requested: params.capabilities?.cardPayments ?? true },
        transfers: { requested: params.capabilities?.transfers ?? true }
      },
      business_profile: {
        name: params.businessProfile.name,
        product_description: params.businessProfile.productDescription,
        support_email: params.businessProfile.supportEmail,
        url: params.businessProfile.url
      },
      settings: {
        payouts: {
          schedule: {
            interval: 'daily',
            delay_days: 2
          }
        }
      }
    });
    
    return account;
  }
  
  async createAccountLink(
    accountId: string,
    refreshUrl: string,
    returnUrl: string
  ): Promise<string> {
    const accountLink = await this.stripe.accountLinks.create({
      account: accountId,
      refresh_url: refreshUrl,
      return_url: returnUrl,
      type: 'account_onboarding'
    });
    
    return accountLink.url;
  }
  
  async createSubscription(params: {
    customerId: string;
    priceId: string;
    trialDays?: number;
    metadata?: Record<string, string>;
  }): Promise<Stripe.Subscription> {
    return await this.stripe.subscriptions.create({
      customer: params.customerId,
      items: [{ price: params.priceId }],
      trial_period_days: params.trialDays,
      metadata: params.metadata,
      payment_behavior: 'default_incomplete',
      payment_settings: {
        save_default_payment_method: 'on_subscription'
      },
      expand: ['latest_invoice.payment_intent']
    });
  }
  
  async processRefund(params: {
    paymentIntentId: string;
    amount?: number;
    reason?: string;
    metadata?: Record<string, string>;
  }): Promise<Stripe.Refund> {
    return await this.stripe.refunds.create({
      payment_intent: params.paymentIntentId,
      amount: params.amount ? Math.round(params.amount * 100) : undefined,
      reason: params.reason as Stripe.RefundCreateParams.Reason,
      metadata: params.metadata
    });
  }
}
```

### Zapier Integration

```typescript
// services/integrations/zapier/handlers.ts
import { APIGatewayProxyEventV2, APIGatewayProxyResultV2 } from 'aws-lambda';
import { z } from 'zod';

// Zapier webhook authentication
const authenticateZapier = (event: APIGatewayProxyEventV2): boolean => {
  const apiKey = event.headers['x-api-key'];
  return apiKey === process.env.ZAPIER_API_KEY;
};

// Trigger: New Event Created
export const newEventTrigger = async (
  event: APIGatewayProxyEventV2
): Promise<APIGatewayProxyResultV2> => {
  if (!authenticateZapier(event)) {
    return {
      statusCode: 401,
      body: JSON.stringify({ error: 'Unauthorized' })
    };
  }
  
  // Zapier sends a subscription request first
  if (event.requestContext.http.method === 'DELETE') {
    // Handle unsubscribe
    return { statusCode: 200, body: JSON.stringify({ success: true }) };
  }
  
  // For GET requests, return sample data
  if (event.requestContext.http.method === 'GET') {
    return {
      statusCode: 200,
      body: JSON.stringify([
        {
          id: 'event-123',
          title: 'Summer Music Festival',
          date: '2024-07-15T18:00:00Z',
          location: 'Central Park, New York',
          organizer: 'John Doe',
          category: 'Music',
          ticketsAvailable: 500,
          price: 50,
          currency: 'USD',
          createdAt: '2024-01-15T10:00:00Z'
        }
      ])
    };
  }
  
  // For POST requests (new subscription), store webhook URL
  const body = JSON.parse(event.body || '{}');
  const webhookUrl = body.hookUrl;
  
  // TODO: Store webhook URL in database
  
  return {
    statusCode: 200,
    body: JSON.stringify({ 
      success: true,
      message: 'Webhook registered successfully'
    })
  };
};

// Action: Create Event
const createEventSchema = z.object({
  title: z.string(),
  description: z.string(),
  date: z.string(),
  location: z.object({
    address: z.string(),
    city: z.string(),
    country: z.string()
  }),
  capacity: z.number(),
  price: z.number(),
  currency: z.string()
});

export const createEventAction = async (
  event: APIGatewayProxyEventV2
): Promise<APIGatewayProxyResultV2> => {
  if (!authenticateZapier(event)) {
    return {
      statusCode: 401,
      body: JSON.stringify({ error: 'Unauthorized' })
    };
  }
  
  try {
    const input = createEventSchema.parse(JSON.parse(event.body || '{}'));
    
    // TODO: Create event in database
    
    return {
      statusCode: 200,
      body: JSON.stringify({
        id: 'event-456',
        ...input,
        status: 'draft',
        createdAt: new Date().toISOString()
      })
    };
  } catch (error) {
    return {
      statusCode: 400,
      body: JSON.stringify({ 
        error: 'Invalid input',
        details: error
      })
    };
  }
};

// Search Action: Find Artists
export const searchArtistsAction = async (
  event: APIGatewayProxyEventV2
): Promise<APIGatewayProxyResultV2> => {
  if (!authenticateZapier(event)) {
    return {
      statusCode: 401,
      body: JSON.stringify({ error: 'Unauthorized' })
    };
  }
  
  const { category, location, available_date } = JSON.parse(event.body || '{}');
  
  // TODO: Search artists in database
  
  return {
    statusCode: 200,
    body: JSON.stringify([
      {
        id: 'artist-789',
        name: 'The Jazz Ensemble',
        category: 'Jazz',
        location: 'New York',
        rating: 4.8,
        price_range: '$500-$1000',
        availability: true
      }
    ])
  };
};
```

## Frontend Integration Guidelines

### API Client Setup (Next.js 15)

```typescript
// lib/api/client.ts
import { QueryClient } from '@tanstack/react-query';
import axios, { AxiosInstance, InternalAxiosRequestConfig } from 'axios';
import { getSession, signOut } from 'next-auth/react';

// Create axios instance
export const apiClient: AxiosInstance = axios.create({
  baseURL: process.env.NEXT_PUBLIC_API_URL || 'https://api.eventfulhq.com',
  timeout: 30000,
  headers: {
    'Content-Type': 'application/json'
  }
});

// Request interceptor
apiClient.interceptors.request.use(
  async (config: InternalAxiosRequestConfig) => {
    // Get session
    const session = await getSession();
    
    if (session?.accessToken) {
      config.headers.Authorization = `Bearer ${session.accessToken}`;
    }
    
    // Add correlation ID
    config.headers['X-Correlation-ID'] = crypto.randomUUID();
    
    // Add language preference
    const language = localStorage.getItem('preferredLanguage') || 'en';
    config.headers['X-Preferred-Language'] = language;
    
    // Add domain preference
    const domain = localStorage.getItem('preferredDomain') || 'eventfulhq.com';
    config.headers['X-Preferred-Domain'] = domain;
    
    return config;
  },
  (error) => Promise.reject(error)
);

// Response interceptor
apiClient.interceptors.response.use(
  (response) => response,
  async (error) => {
    const originalRequest = error.config;
    
    if (error.response?.status === 401 && !originalRequest._retry) {
      originalRequest._retry = true;
      
      try {
        // Try to refresh token
        const session = await getSession();
        if (session) {
          // Token refresh is handled by NextAuth
          return apiClient(originalRequest);
        }
      } catch (refreshError) {
        // Refresh failed, sign out
        await signOut({ redirect: true, callbackUrl: '/login' });
      }
    }
    
    return Promise.reject(error);
  }
);

// React Query client
export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 5 * 60 * 1000, // 5 minutes
      cacheTime: 10 * 60 * 1000, // 10 minutes
      retry: 3,
      retryDelay: (attemptIndex) => Math.min(1000 * 2 ** attemptIndex, 30000),
      refetchOnWindowFocus: false
    },
    mutations: {
      retry: 1
    }
  }
});
```

### API Hooks (React Query)

```typescript
// hooks/api/useEvents.ts
import { useQuery, useMutation, useInfiniteQuery } from '@tanstack/react-query';
import { apiClient } from '@/lib/api/client';
import { Event, CreateEventDto, UpdateEventDto } from '@/types/event';

// Fetch events with infinite scroll
export const useEvents = (filters?: {
  category?: string;
  location?: string;
  startDate?: string;
  endDate?: string;
}) => {
  return useInfiniteQuery({
    queryKey: ['events', filters],
    queryFn: async ({ pageParam = 1 }) => {
      const { data } = await apiClient.get('/api/v1/events', {
        params: {
          page: pageParam,
          limit: 20,
          ...filters
        }
      });
      return data;
    },
    getNextPageParam: (lastPage) => lastPage.nextPage,
    getPreviousPageParam: (firstPage) => firstPage.prevPage
  });
};

// Fetch single event
export const useEvent = (eventId: string) => {
  return useQuery({
    queryKey: ['event', eventId],
    queryFn: async () => {
      const { data } = await apiClient.get(`/api/v1/events/${eventId}`);
      return data.data as Event;
    },
    enabled: !!eventId
  });
};

// Create event
export const useCreateEvent = () => {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: async (eventData: CreateEventDto) => {
      const { data } = await apiClient.post('/api/v1/events', eventData);
      return data.data as Event;
    },
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['events'] });
    }
  });
};

// Update event
export const useUpdateEvent = () => {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: async ({ 
      eventId, 
      data 
    }: { 
      eventId: string; 
      data: UpdateEventDto 
    }) => {
      const response = await apiClient.put(`/api/v1/events/${eventId}`, data);
      return response.data.data as Event;
    },
    onSuccess: (data) => {
      queryClient.invalidateQueries({ queryKey: ['event', data.id] });
      queryClient.invalidateQueries({ queryKey: ['events'] });
    }
  });
};

// Delete event
export const useDeleteEvent = () => {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: async (eventId: string) => {
      await apiClient.delete(`/api/v1/events/${eventId}`);
    },
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['events'] });
    }
  });
};

// AI-powered event recommendations
export const useEventRecommendations = () => {
  return useQuery({
    queryKey: ['event-recommendations'],
    queryFn: async () => {
      const { data } = await apiClient.get('/api/v1/events/recommendations');
      return data.data as Event[];
    }
  });
};

// Trending events by region
export const useTrendingEvents = (region?: string) => {
  return useQuery({
    queryKey: ['trending-events', region],
    queryFn: async () => {
      const { data } = await apiClient.get('/api/v1/events/trending', {
        params: { region }
      });
      return data.data as Event[];
    }
  });
};
```

### Component Integration Example

```tsx
// app/(dashboard)/events/page.tsx
'use client';

import { useState } from 'react';
import { useEvents, useCreateEvent } from '@/hooks/api/useEvents';
import { 
  Box, 
  Container, 
  Grid, 
  Typography, 
  Button, 
  Skeleton,
  Alert,
  Fab
} from '@mui/material';
import { Add as AddIcon } from '@mui/icons-material';
import { EventCard } from '@/components/events/EventCard';
import { EventFilters } from '@/components/events/EventFilters';
import { CreateEventDialog } from '@/components/events/CreateEventDialog';
import { InfiniteScroll } from '@/components/common/InfiniteScroll';
import { PageBreadcrumbs } from '@/components/common/PageBreadcrumbs';
import { useTranslation } from 'next-i18next';

export default function EventsPage() {
  const { t } = useTranslation('events');
  const [filters, setFilters] = useState({});
  const [createDialogOpen, setCreateDialogOpen] = useState(false);
  
  const {
    data,
    fetchNextPage,
    hasNextPage,
    isFetchingNextPage,
    isLoading,
    isError,
    error
  } = useEvents(filters);
  
  const createEventMutation = useCreateEvent();
  
  const events = data?.pages.flatMap(page => page.data) ?? [];
  
  const handleCreateEvent = async (eventData: any) => {
    try {
      await createEventMutation.mutateAsync(eventData);
      setCreateDialogOpen(false);
    } catch (error) {
      console.error('Failed to create event:', error);
    }
  };
  
  return (
    <Container maxWidth="xl">
      <PageBreadcrumbs />
      
      <Box sx={{ mb: 4 }}>
        <Typography variant="h3" component="h1" gutterBottom>
          {t('events.title')}
        </Typography>
        <Typography variant="body1" color="text.secondary">
          {t('events.description')}
        </Typography>
      </Box>
      
      <EventFilters 
        filters={filters} 
        onChange={setFilters} 
      />
      
      {isError && (
        <Alert severity="error" sx={{ mb: 3 }}>
          {t('events.error.loading')}
        </Alert>
      )}
      
      <InfiniteScroll
        onLoadMore={fetchNextPage}
        hasMore={hasNextPage || false}
        isLoading={isFetchingNextPage}
      >
        <Grid container spacing={3}>
          {isLoading ? (
            // Loading skeletons
            Array.from({ length: 6 }).map((_, index) => (
              <Grid item xs={12} sm={6} md={4} key={index}>
                <Skeleton variant="rectangular" height={320} />
              </Grid>
            ))
          ) : events.length === 0 ? (
            <Grid item xs={12}>
              <Alert severity="info">
                {t('events.empty')}
              </Alert>
            </Grid>
          ) : (
            events.map((event) => (
              <Grid item xs={12} sm={6} md={4} key={event.id}>
                <EventCard event={event} />
              </Grid>
            ))
          )}
        </Grid>
      </InfiniteScroll>
      
      <Fab
        color="primary"
        aria-label={t('events.create')}
        sx={{
          position: 'fixed',
          bottom: 24,
          right: 24
        }}
        onClick={() => setCreateDialogOpen(true)}
      >
        <AddIcon />
      </Fab>
      
      <CreateEventDialog
        open={createDialogOpen}
        onClose={() => setCreateDialogOpen(false)}
        onSubmit={handleCreateEvent}
        isLoading={createEventMutation.isLoading}
      />
    </Container>
  );
}
```

### State Management (Zustand)

```typescript
// stores/useAppStore.ts
import { create } from 'zustand';
import { persist, createJSONStorage } from 'zustand/middleware';
import { Language, Domain } from '@/types/app';

interface AppState {
  // User preferences
  language: Language;
  domain: Domain;
  theme: 'light' | 'dark' | 'system';
  
  // UI state
  sidebarOpen: boolean;
  notifications: Notification[];
  
  // Actions
  setLanguage: (language: Language) => void;
  setDomain: (domain: Domain) => void;
  setTheme: (theme: 'light' | 'dark' | 'system') => void;
  toggleSidebar: () => void;
  addNotification: (notification: Notification) => void;
  removeNotification: (id: string) => void;
}

export const useAppStore = create<AppState>()(
  persist(
    (set) => ({
      // Initial state
      language: 'en',
      domain: 'eventfulhq.com',
      theme: 'system',
      sidebarOpen: true,
      notifications: [],
      
      // Actions
      setLanguage: (language) => set({ language }),
      setDomain: (domain) => set({ domain }),
      setTheme: (theme) => set({ theme }),
      toggleSidebar: () => set((state) => ({ 
        sidebarOpen: !state.sidebarOpen 
      })),
      addNotification: (notification) => set((state) => ({
        notifications: [...state.notifications, notification]
      })),
      removeNotification: (id) => set((state) => ({
        notifications: state.notifications.filter(n => n.id !== id)
      }))
    }),
    {
      name: 'eventfulhq-app-store',
      storage: createJSONStorage(() => localStorage),
      partialize: (state) => ({
        language: state.language,
        domain: state.domain,
        theme: state.theme
      })
    }
  )
);
```

## Swagger Documentation Standards

### Swagger Configuration

```typescript
// src/swagger/config.ts
import { DocumentBuilder, SwaggerModule } from '@nestjs/swagger';
import { INestApplication } from '@nestjs/common';

export function setupSwagger(app: INestApplication, serviceName: string): void {
  const config = new DocumentBuilder()
    .setTitle(`EventfulHQ ${serviceName} API`)
    .setDescription(`
      EventfulHQ - AI-First, WhatsApp-First Global Events Platform
      
      Service: ${serviceName}
      Version: 1.0.0
      Environment: ${process.env.STAGE}
      
      ## Authentication
      All endpoints require JWT authentication unless marked as public.
      Include the token in the Authorization header: Bearer <token>
      
      ## Rate Limiting
      - Free tier: 100 requests/minute
      - Basic tier: 500 requests/minute  
      - Pro tier: 1000 requests/minute
      - Pro Max tier: 2000 requests/minute
      - Enterprise: Custom limits
      
      ## Multi-Language Support
      Include X-Preferred-Language header with one of:
      en, zh, es, pt, fr, ja, ar, hi, ru, de
      
      ## Regional Domains
      - Global: eventfulhq.com
      - Americas: eventfulamerica.com
      - Arabia: eventfularabia.com
      - India: eventfulindia.com
      - Europe: eventfuleurope.com
      - Australia: eventfulaustralia.com
    `)
    .setVersion('1.0.0')
    .setContact(
      'EventfulHQ API Team',
      'https://developer.eventfulhq.com',
      'api@eventfulhq.com'
    )
    .setLicense(
      'Proprietary',
      'https://eventfulhq.com/terms'
    )
    .addBearerAuth(
      {
        type: 'http',
        scheme: 'bearer',
        bearerFormat: 'JWT',
        description: 'Enter JWT token'
      },
      'JWT-auth'
    )
    .addApiKey(
      {
        type: 'apiKey',
        in: 'header',
        name: 'X-API-Key',
        description: 'API Key for external integrations'
      },
      'API-Key'
    )
    .addServer(`https://api.eventfulhq.com`, 'Production')
    .addServer(`https://staging-api.eventfulhq.com`, 'Staging')
    .addServer(`http://localhost:3000`, 'Development')
    .addTag('Authentication', 'WhatsApp-first authentication')
    .addTag('Events', 'Event management')
    .addTag('Tickets', 'Ticketing system')
    .addTag('Artists', 'Artist profiles')
    .addTag('Payments', 'Payment processing')
    .addTag('AI', 'AI-powered features')
    .addTag('WhatsApp', 'WhatsApp integration')
    .addTag('Admin', 'Admin operations')
    .addTag('Public', 'Public endpoints')
    .build();

  const document = SwaggerModule.createDocument(app, config, {
    operationIdFactory: (
      controllerKey: string,
      methodKey: string
    ) => methodKey
  });

  SwaggerModule.setup(`api/docs/${serviceName}`, app, document, {
    swaggerOptions: {
      persistAuthorization: true,
      docExpansion: 'none',
      filter: true,
      showRequestDuration: true,
      syntaxHighlight: {
        activate: true,
        theme: 'monokai'
      },
      tryItOutEnabled: true,
      requestInterceptor: (req: any) => {
        req.headers['X-Correlation-ID'] = crypto.randomUUID();
        return req;
      }
    },
    customSiteTitle: `EventfulHQ ${serviceName} API Documentation`,
    customfavIcon: '/favicon.ico',
    customCssUrl: '/swagger-custom.css'
  });
}
```

### API Documentation Decorators

```typescript
// Example controller with comprehensive Swagger documentation
import { 
  Controller, 
  Get, 
  Post, 
  Put, 
  Delete, 
  Body, 
  Param, 
  Query, 
  UseGuards,
  HttpStatus,
  HttpCode
} from '@nestjs/common';
import {
  ApiTags,
  ApiOperation,
  ApiResponse,
  ApiBearerAuth,
  ApiQuery,
  ApiParam,
  ApiBody,
  ApiExtraModels,
  ApiPropertyOptional,
  getSchemaPath
} from '@nestjs/swagger';

@ApiTags('Events')
@ApiBearerAuth('JWT-auth')
@Controller('api/v1/events')
@ApiExtraModels(Event, CreateEventDto, UpdateEventDto, ErrorResponse)
export class EventController {
  
  @Post()
  @HttpCode(HttpStatus.CREATED)
  @ApiOperation({
    summary: 'Create new event',
    description: `
      Creates a new event with AI-powered content generation.
      
      **Features:**
      - Automatic content translation to 10 languages
      - AI-generated descriptions and tags
      - Smart pricing recommendations
      - Automatic SEO optimization
      
      **Required Permissions:**
      - events:create
      - For paid events: payments:setup
      
      **Subscription Tiers:**
      - Free: 5 events/month
      - Basic: 20 events/month
      - Pro: 100 events/month
      - Pro Max: Unlimited
    `
  })
  @ApiBody({
    type: CreateEventDto,
    description: 'Event creation data',
    examples: {
      basicEvent: {
        summary: 'Basic event',
        value: {
          title: 'Summer Music Festival',
          description: 'Annual summer music celebration',
          eventDate: '2024-07-15T18:00:00Z',
          location: {
            address: '123 Main St',
            city: 'New York',
            state: 'NY',
            country: 'USA',
            coordinates: {
              lat: 40.7128,
              lng: -74.0060
            }
          },
          capacity: 500,
          ticketTypes: [
            {
              name: 'General Admission',
              price: 50,
              quantity: 400
            },
            {
              name: 'VIP',
              price: 150,
              quantity: 100
            }
          ]
        }
      },
      aiGeneratedEvent: {
        summary: 'AI-generated content',
        value: {
          title: 'Corporate Team Building',
          eventType: 'corporate',
          eventDate: '2024-08-20T09:00:00Z',
          expectedAttendees: 50,
          budget: {
            min: 5000,
            max: 10000,
            currency: 'USD'
          },
          preferences: ['team building', 'outdoor', 'adventure'],
          aiGenerate: true
        }
      }
    }
  })
  @ApiResponse({
    status: HttpStatus.CREATED,
    description: 'Event created successfully',
    schema: {
      allOf: [
        { $ref: getSchemaPath(Event) },
        {
          properties: {
            qrCode: { 
              type: 'string',
              description: 'QR code for event check-in'
            },
            micrositeUrl: {
              type: 'string',
              description: 'Event microsite URL'
            }
          }
        }
      ]
    }
  })
  @ApiResponse({
    status: HttpStatus.BAD_REQUEST,
    description: 'Invalid input data',
    type: ErrorResponse
  })
  @ApiResponse({
    status: HttpStatus.UNAUTHORIZED,
    description: 'Authentication required'
  })
  @ApiResponse({
    status: HttpStatus.FORBIDDEN,
    description: 'Insufficient permissions or quota exceeded'
  })
  @ApiResponse({
    status: HttpStatus.TOO_MANY_REQUESTS,
    description: 'Rate limit exceeded',
    headers: {
      'X-RateLimit-Limit': {
        description: 'Request limit per minute',
        schema: { type: 'integer' }
      },
      'X-RateLimit-Remaining': {
        description: 'Remaining requests',
        schema: { type: 'integer' }
      },
      'X-RateLimit-Reset': {
        description: 'Reset timestamp',
        schema: { type: 'integer' }
      }
    }
  })
  async createEvent(
    @Body() createEventDto: CreateEventDto,
    @CurrentUser() user: User
  ): Promise<Event> {
    return this.eventService.create(createEventDto, user);
  }

  @Get()
  @ApiOperation({
    summary: 'List events',
    description: 'Get paginated list of events with advanced filtering'
  })
  @ApiQuery({
    name: 'page',
    required: false,
    type: Number,
    description: 'Page number (default: 1)'
  })
  @ApiQuery({
    name: 'limit',
    required: false,
    type: Number,
    description: 'Items per page (default: 20, max: 100)'
  })
  @ApiQuery({
    name: 'category',
    required: false,
    enum: EventCategory,
    description: 'Filter by category'
  })
  @ApiQuery({
    name: 'location',
    required: false,
    type: String,
    description: 'Filter by location (city, state, or country)'
  })
  @ApiQuery({
    name: 'startDate',
    required: false,
    type: String,
    description: 'Filter by start date (ISO 8601)'
  })
  @ApiQuery({
    name: 'endDate',
    required: false,
    type: String,
    description: 'Filter by end date (ISO 8601)'
  })
  @ApiQuery({
    name: 'priceMin',
    required: false,
    type: Number,
    description: 'Minimum price filter'
  })
  @ApiQuery({
    name: 'priceMax',
    required: false,
    type: Number,
    description: 'Maximum price filter'
  })
  @ApiQuery({
    name: 'search',
    required: false,
    type: String,
    description: 'Search in title and description'
  })
  @ApiQuery({
    name: 'sortBy',
    required: false,
    enum: ['date', 'price', 'popularity', 'created'],
    description: 'Sort field (default: date)'
  })
  @ApiQuery({
    name: 'sortOrder',
    required: false,
    enum: ['asc', 'desc'],
    description: 'Sort order (default: asc)'
  })
  @ApiResponse({
    status: HttpStatus.OK,
    description: 'Events retrieved successfully',
    schema: {
      type: 'object',
      properties: {
        data: {
          type: 'array',
          items: { $ref: getSchemaPath(Event) }
        },
        pagination: {
          type: 'object',
          properties: {
            total: { type: 'number' },
            page: { type: 'number' },
            limit: { type: 'number' },
            totalPages: { type: 'number' }
          }
        }
      }
    }
  })
  async getEvents(
    @Query() query: GetEventsDto
  ): Promise<PaginatedResponse<Event>> {
    return this.eventService.findAll(query);
  }

  @Get(':eventId')
  @ApiOperation({
    summary: 'Get event details',
    description: 'Retrieve detailed information about a specific event'
  })
  @ApiParam({
    name: 'eventId',
    type: 'string',
    format: 'uuid',
    description: 'Event unique identifier'
  })
  @ApiResponse({
    status: HttpStatus.OK,
    description: 'Event details retrieved',
    type: Event
  })
  @ApiResponse({
    status: HttpStatus.NOT_FOUND,
    description: 'Event not found'
  })
  async getEvent(
    @Param('eventId') eventId: string
  ): Promise<Event> {
    return this.eventService.findById(eventId);
  }
}
```

### DTO Documentation

```typescript
// dto/create-event.dto.ts
import { ApiProperty, ApiPropertyOptional } from '@nestjs/swagger';
import { Type } from 'class-transformer';
import {
  IsString,
  IsDate,
  IsNumber,
  IsBoolean,
  IsArray,
  IsObject,
  IsEnum,
  IsOptional,
  ValidateNested,
  Min,
  Max,
  MinLength,
  MaxLength
} from 'class-validator';

export class LocationDto {
  @ApiProperty({
    description: 'Street address',
    example: '123 Main Street'
  })
  @IsString()
  @MinLength(5)
  @MaxLength(200)
  address: string;

  @ApiProperty({
    description: 'City name',
    example: 'New York'
  })
  @IsString()
  city: string;

  @ApiPropertyOptional({
    description: 'State or province',
    example: 'NY'
  })
  @IsString()
  @IsOptional()
  state?: string;

  @ApiProperty({
    description: 'Country name',
    example: 'USA'
  })
  @IsString()
  country: string;

  @ApiPropertyOptional({
    description: 'Postal/ZIP code',
    example: '10001'
  })
  @IsString()
  @IsOptional()
  postalCode?: string;

  @ApiPropertyOptional({
    description: 'GPS coordinates',
    type: 'object',
    properties: {
      lat: { type: 'number', example: 40.7128 },
      lng: { type: 'number', example: -74.0060 }
    }
  })
  @IsObject()
  @IsOptional()
  coordinates?: {
    lat: number;
    lng: number;
  };
}

export class TicketTypeDto {
  @ApiProperty({
    description: 'Ticket type name',
    example: 'General Admission'
  })
  @IsString()
  name: string;

  @ApiProperty({
    description: 'Price in smallest currency unit',
    example: 5000,
    minimum: 0
  })
  @IsNumber()
  @Min(0)
  price: number;

  @ApiProperty({
    description: 'Available quantity',
    example: 100,
    minimum: 1
  })
  @IsNumber()
  @Min(1)
  quantity: number;

  @ApiPropertyOptional({
    description: 'Ticket description',
    example: 'Standard entry with access to all areas'
  })
  @IsString()
  @IsOptional()
  description?: string;
}

export class CreateEventDto {
  @ApiProperty({
    description: 'Event title',
    example: 'Summer Music Festival 2024',
    minLength: 5,
    maxLength: 100
  })
  @IsString()
  @MinLength(5)
  @MaxLength(100)
  title: string;

  @ApiPropertyOptional({
    description: 'Detailed event description',
    example: 'Join us for an unforgettable summer music experience...',
    maxLength: 5000
  })
  @IsString()
  @IsOptional()
  @MaxLength(5000)
  description?: string;

  @ApiProperty({
    description: 'Event date and time (ISO 8601)',
    example: '2024-07-15T18:00:00Z',
    type: 'string',
    format: 'date-time'
  })
  @IsDate()
  @Type(() => Date)
  eventDate: Date;

  @ApiProperty({
    description: 'Event location details',
    type: LocationDto
  })
  @ValidateNested()
  @Type(() => LocationDto)
  location: LocationDto;

  @ApiProperty({
    description: 'Maximum capacity',
    example: 500,
    minimum: 1,
    maximum: 1000000
  })
  @IsNumber()
  @Min(1)
  @Max(1000000)
  capacity: number;

  @ApiProperty({
    description: 'Event category',
    enum: EventCategory,
    example: EventCategory.MUSIC
  })
  @IsEnum(EventCategory)
  category: EventCategory;

  @ApiPropertyOptional({
    description: 'Event subcategory',
    example: 'Rock'
  })
  @IsString()
  @IsOptional()
  subcategory?: string;

  @ApiPropertyOptional({
    description: 'Ticket types and pricing',
    type: [TicketTypeDto],
    isArray: true
  })
  @IsArray()
  @ValidateNested({ each: true })
  @Type(() => TicketTypeDto)
  @IsOptional()
  ticketTypes?: TicketTypeDto[];

  @ApiPropertyOptional({
    description: 'Enable AI content generation',
    example: true,
    default: false
  })
  @IsBoolean()
  @IsOptional()
  aiGenerate?: boolean = false;

  @ApiPropertyOptional({
    description: 'Enable WhatsApp notifications',
    example: true,
    default: true
  })
  @IsBoolean()
  @IsOptional()
  whatsappNotifications?: boolean = true;

  @ApiPropertyOptional({
    description: 'Custom metadata',
    type: 'object',
    additionalProperties: true
  })
  @IsObject()
  @IsOptional()
  metadata?: Record<string, any>;
}
```

## Performance & SLA Requirements

### Service Level Objectives (SLOs)

```yaml
Global SLAs:
  Availability:
    Target: 99.9% (43.2 minutes downtime/month)
    Measurement: 5-minute intervals
    
  Response Time:
    P50: < 100ms
    P95: < 300ms
    P99: < 500ms
    
  Error Rate:
    Target: < 0.1%
    Critical Services: < 0.01%
    
Service-Specific SLAs:
  AuthService:
    Availability: 99.99%
    Response Time:
      WhatsApp OTP: < 200ms (P95)
      Token Refresh: < 50ms (P95)
    Error Rate: < 0.01%
    
  PaymentService:
    Availability: 99.99%
    Response Time:
      Payment Intent: < 500ms (P95)
      Refund: < 300ms (P95)
    Error Rate: < 0.001%
    Success Rate: > 95%
    
  EventService:
    Availability: 99.9%
    Response Time:
      Create: < 300ms (P95)
      List: < 200ms (P95)
      Search: < 500ms (P95)
    Error Rate: < 0.1%
    
  TicketingService:
    Availability: 99.95%
    Response Time:
      Purchase: < 1000ms (P95)
      Validate: < 100ms (P95)
      QR Generation: < 200ms (P95)
    Error Rate: < 0.05%
    
  WhatsAppService:
    Availability: 99.9%
    Response Time:
      Send Message: < 500ms (P95)
      Template Message: < 300ms (P95)
    Delivery Rate: > 98%
    Error Rate: < 0.1%
    
  AIService:
    Availability: 99.5%
    Response Time:
      Text Generation: < 3000ms (P95)
      Translation: < 1000ms (P95)
      Recommendations: < 500ms (P95)
    Error Rate: < 1%
    
  SearchService:
    Availability: 99.9%
    Response Time:
      Simple Search: < 200ms (P95)
      Complex Search: < 500ms (P95)
      Autocomplete: < 50ms (P95)
    Error Rate: < 0.1%
```

### Performance Monitoring Implementation

```typescript
// src/monitoring/performance.ts
import { Metrics, MetricUnits } from '@aws-lambda-powertools/metrics';
import { Tracer } from '@aws-lambda-powertools/tracer';
import { Logger } from '@aws-lambda-powertools/logger';
import CloudWatch from '@aws-sdk/client-cloudwatch';

export class PerformanceMonitor {
  private metrics: Metrics;
  private tracer: Tracer;
  private logger: Logger;
  private cloudWatch: CloudWatchClient;
  
  constructor(serviceName: string) {
    this.metrics = new Metrics({ 
      namespace: 'EventfulHQ',
      serviceName 
    });
    this.tracer = new Tracer({ serviceName });
    this.logger = new Logger({ serviceName });
    this.cloudWatch = new CloudWatchClient({});
  }
  
  // Track API response time
  async trackResponseTime(
    operation: string,
    duration: number,
    statusCode: number
  ): Promise<void> {
    // Add metrics
    this.metrics.addMetric('ResponseTime', MetricUnits.Milliseconds, duration);
    this.metrics.addMetadata('operation', operation);
    this.metrics.addMetadata('statusCode', statusCode.toString());
    
    // Check SLA compliance
    const slaThreshold = this.getSLAThreshold(operation);
    if (duration > slaThreshold) {
      this.logger.warn('SLA violation detected', {
        operation,
        duration,
        threshold: slaThreshold,
        violation: duration - slaThreshold
      });
      
      this.metrics.addMetric('SLAViolation', MetricUnits.Count, 1);
    }
    
    // Publish metrics
    this.metrics.publishStoredMetrics();
  }
  
  // Track error rates
  trackError(operation: string, error: Error): void {
    this.metrics.addMetric('ErrorCount', MetricUnits.Count, 1);
    this.metrics.addMetadata('operation', operation);
    this.metrics.addMetadata('errorType', error.constructor.name);
    this.metrics.addMetadata('errorMessage', error.message);
    
    this.logger.error('Operation failed', error, {
      operation,
      errorType: error.constructor.name
    });
  }
  
  // Track business metrics
  trackBusinessMetric(
    metricName: string,
    value: number,
    unit: MetricUnits,
    metadata?: Record<string, string>
  ): void {
    this.metrics.addMetric(metricName, unit, value);
    
    if (metadata) {
      Object.entries(metadata).forEach(([key, val]) => {
        this.metrics.addMetadata(key, val);
      });
    }
  }
  
  // Create CloudWatch alarm for SLA monitoring
  async createSLAAlarm(
    alarmName: string,
    metricName: string,
    threshold: number,
    evaluationPeriods: number = 2
  ): Promise<void> {
    await this.cloudWatch.send(new PutMetricAlarmCommand({
      AlarmName: `EventfulHQ-${alarmName}`,
      ComparisonOperator: 'GreaterThanThreshold',
      EvaluationPeriods: evaluationPeriods,
      MetricName: metricName,
      Namespace: 'EventfulHQ',
      Period: 300, // 5 minutes
      Statistic: 'Average',
      Threshold: threshold,
      ActionsEnabled: true,
      AlarmActions: [process.env.SNS_ALERT_TOPIC_ARN!],
      AlarmDescription: `SLA violation alert for ${metricName}`,
      TreatMissingData: 'notBreaching'
    }));
  }
  
  private getSLAThreshold(operation: string): number {
    const thresholds: Record<string, number> = {
      'auth.sendOTP': 200,
      'auth.verifyOTP': 100,
      'auth.refresh': 50,
      'events.create': 300,
      'events.list': 200,
      'events.search': 500,
      'payments.process': 1000,
      'tickets.purchase': 1000,
      'tickets.validate': 100
    };
    
    return thresholds[operation] || 500; // Default threshold
  }
}

// Usage in Lambda handler
export const performanceWrapper = (
  handler: Function,
  operation: string
) => {
  return async (event: any, context: any) => {
    const monitor = new PerformanceMonitor(context.functionName);
    const startTime = Date.now();
    
    try {
      const result = await handler(event, context);
      const duration = Date.now() - startTime;
      
      await monitor.trackResponseTime(
        operation,
        duration,
        result.statusCode || 200
      );
      
      return result;
    } catch (error) {
      monitor.trackError(operation, error as Error);
      throw error;
    }
  };
};
```

### Lambda Cold Start Optimization

```typescript
// src/optimizations/cold-start.ts
import { readFileSync } from 'fs';
import { join } from 'path';

// 1. Pre-initialize connections outside handler
let dbConnection: DataSource;
let redisClient: RedisClient;
let initialized = false;

async function initialize(): Promise<void> {
  if (initialized) return;
  
  console.time('cold-start-init');
  
  // Initialize database connection
  dbConnection = await getDataSource();
  
  // Initialize Redis connection
  redisClient = await getRedisClient();
  
  // Pre-warm critical paths
  await prewarmCriticalPaths();
  
  initialized = true;
  console.timeEnd('cold-start-init');
}

// 2. Minimize dependencies
// Use dynamic imports for heavy libraries
const heavyOperations = {
  async generatePDF(): Promise<any> {
    const PDFDocument = (await import('pdfkit')).default;
    return new PDFDocument();
  },
  
  async processImage(): Promise<any> {
    const sharp = (await import('sharp')).default;
    return sharp;
  },
  
  async generateExcel(): Promise<any> {
    const ExcelJS = (await import('exceljs')).default;
    return new ExcelJS.Workbook();
  }
};

// 3. Lambda provisioned concurrency configuration
export const lambdaConfig = {
  memorySize: 1024, // Optimal for most use cases
  timeout: 30,
  environment: {
    NODE_OPTIONS: '--enable-source-maps --max-old-space-size=896',
    AWS_NODEJS_CONNECTION_REUSE_ENABLED: '1'
  },
  provisionedConcurrency: {
    production: {
      AuthService: 5,
      PaymentService: 3,
      EventService: 3,
      TicketingService: 3
    },
    staging: {
      AuthService: 1,
      PaymentService: 1,
      EventService: 1,
      TicketingService: 1
    }
  }
};

// 4. Webpack configuration for tree shaking
export const webpackConfig = {
  mode: 'production',
  target: 'node',
  externals: {
    'aws-sdk': 'aws-sdk',
    '@aws-sdk/client-s3': '@aws-sdk/client-s3',
    '@aws-sdk/client-dynamodb': '@aws-sdk/client-dynamodb'
  },
  optimization: {
    minimize: true,
    sideEffects: false,
    usedExports: true
  },
  resolve: {
    extensions: ['.ts', '.js'],
    alias: {
      '@': '/opt/nodejs/node_modules/@eventfulhq'
    }
  }
};

// 5. Pre-warm critical paths
async function prewarmCriticalPaths(): Promise<void> {
  // Pre-compile regex patterns
  const patterns = {
    email: /^[^\s@]+@[^\s@]+\.[^\s@]+$/,
    phone: /^\+?[1-9]\d{1,14}$/,
    uuid: /^[0-9a-f]{8}-[0-9a-f]{4}-4[0-9a-f]{3}-[89ab][0-9a-f]{3}-[0-9a-f]{12}$/i
  };
  
  // Pre-load configuration
  const config = await getConfig();
  
  // Pre-establish HTTP agent
  const agent = new https.Agent({
    keepAlive: true,
    maxSockets: 50
  });
}
```

## Developer Portal Architecture

### Developer Portal API

```typescript
// services/developer-portal/handlers/portal.ts
import { APIGatewayProxyEventV2, APIGatewayProxyResultV2 } from 'aws-lambda';
import { DeveloperPortalService } from '../services/developer-portal.service';

const service = new DeveloperPortalService();

// Get API documentation
export const getDocumentation = async (
  event: APIGatewayProxyEventV2
): Promise<APIGatewayProxyResultV2> => {
  const { version = 'latest' } = event.queryStringParameters || {};
  
  const docs = await service.getDocumentation(version);
  
  return {
    statusCode: 200,
    headers: {
      'Content-Type': 'application/json',
      'Cache-Control': 'public, max-age=3600'
    },
    body: JSON.stringify(docs)
  };
};

// Create developer app
export const createApp = async (
  event: APIGatewayProxyEventV2
): Promise<APIGatewayProxyResultV2> => {
  const userId = event.requestContext.authorizer?.lambda.userId;
  const appData = JSON.parse(event.body || '{}');
  
  const app = await service.createApp(userId, appData);
  
  return {
    statusCode: 201,
    body: JSON.stringify({
      appId: app.appId,
      clientId: app.clientId,
      clientSecret: app.clientSecret, // Only shown once
      webhookUrl: app.webhookUrl,
      redirectUris: app.redirectUris,
      scopes: app.scopes,
      createdAt: app.createdAt
    })
  };
};

// Generate API key
export const generateAPIKey = async (
  event: APIGatewayProxyEventV2
): Promise<APIGatewayProxyResultV2> => {
  const userId = event.requestContext.authorizer?.lambda.userId;
  const { appId } = event.pathParameters || {};
  const { name, permissions, expiresInDays } = JSON.parse(event.body || '{}');
  
  const apiKey = await service.generateAPIKey(
    userId,
    appId!,
    name,
    permissions,
    expiresInDays
  );
  
  return {
    statusCode: 201,
    body: JSON.stringify({
      keyId: apiKey.keyId,
      key: apiKey.key, // Only shown once
      name: apiKey.name,
      permissions: apiKey.permissions,
      expiresAt: apiKey.expiresAt,
      createdAt: apiKey.createdAt
    })
  };
};
```

### SDK Generation

```typescript
// services/developer-portal/sdk-generator.ts
import { OpenAPIV3 } from 'openapi-types';
import { generateSDK } from '@eventfulhq/sdk-generator';

export class SDKGenerator {
  async generateTypeScriptSDK(
    openApiSpec: OpenAPIV3.Document
  ): Promise<string> {
    const sdkCode = `
// EventfulHQ TypeScript SDK
// Auto-generated from OpenAPI specification
// Version: ${openApiSpec.info.version}

import axios, { AxiosInstance } from 'axios';

export interface EventfulHQConfig {
  apiKey?: string;
  accessToken?: string;
  baseURL?: string;
  timeout?: number;
}

export class EventfulHQ {
  private client: AxiosInstance;
  
  constructor(config: EventfulHQConfig) {
    this.client = axios.create({
      baseURL: config.baseURL || 'https://api.eventfulhq.com',
      timeout: config.timeout || 30000,
      headers: {
        'X-API-Key': config.apiKey,
        'Authorization': config.accessToken ? \`Bearer \${config.accessToken}\` : undefined,
        'Content-Type': 'application/json',
        'User-Agent': 'EventfulHQ-SDK/1.0.0'
      }
    });
  }
  
  // Events API
  events = {
    create: async (data: CreateEventDto): Promise<Event> => {
      const response = await this.client.post('/api/v1/events', data);
      return response.data;
    },
    
    list: async (params?: EventQueryParams): Promise<PaginatedEvents> => {
      const response = await this.client.get('/api/v1/events', { params });
      return response.data;
    },
    
    get: async (eventId: string): Promise<Event> => {
      const response = await this.client.get(\`/api/v1/events/\${eventId}\`);
      return response.data;
    },
    
    update: async (eventId: string, data: UpdateEventDto): Promise<Event> => {
      const response = await this.client.put(\`/api/v1/events/\${eventId}\`, data);
      return response.data;
    },
    
    delete: async (eventId: string): Promise<void> => {
      await this.client.delete(\`/api/v1/events/\${eventId}\`);
    }
  };
  
  // Tickets API
  tickets = {
    create: async (eventId: string, data: CreateTicketDto): Promise<Ticket> => {
      const response = await this.client.post(\`/api/v1/events/\${eventId}/tickets\`, data);
      return response.data;
    },
    
    purchase: async (data: PurchaseTicketDto): Promise<TicketPurchase> => {
      const response = await this.client.post('/api/v1/tickets/purchase', data);
      return response.data;
    },
    
    validate: async (qrCode: string): Promise<TicketValidation> => {
      const response = await this.client.get(\`/api/v1/tickets/validate/\${qrCode}\`);
      return response.data;
    }
  };
  
  // Artists API
  artists = {
    list: async (params?: ArtistQueryParams): Promise<PaginatedArtists> => {
      const response = await this.client.get('/api/v1/artists', { params });
      return response.data;
    },
    
    get: async (artistId: string): Promise<Artist> => {
      const response = await this.client.get(\`/api/v1/artists/\${artistId}\`);
      return response.data;
    },
    
    book: async (artistId: string, data: BookArtistDto): Promise<Booking> => {
      const response = await this.client.post(\`/api/v1/artists/\${artistId}/book\`, data);
      return response.data;
    }
  };
  
  // WhatsApp API
  whatsapp = {
    sendMessage: async (data: SendWhatsAppMessageDto): Promise<MessageStatus> => {
      const response = await this.client.post('/api/v1/messages/whatsapp/send', data);
      return response.data;
    },
    
    sendTemplate: async (data: SendTemplateMessageDto): Promise<MessageStatus> => {
      const response = await this.client.post('/api/v1/messages/whatsapp/template', data);
      return response.data;
    }
  };
  
  // AI API
  ai = {
    generateContent: async (data: GenerateContentDto): Promise<GeneratedContent> => {
      const response = await this.client.post('/api/v1/ai/generate/content', data);
      return response.data;
    },
    
    getRecommendations: async (type: RecommendationType): Promise<Recommendations> => {
      const response = await this.client.post('/api/v1/ai/recommendations', { type });
      return response.data;
    },
    
    translate: async (data: TranslateDto): Promise<TranslatedContent> => {
      const response = await this.client.post('/api/v1/ai/translate', data);
      return response.data;
    }
  };
}

// Export types
${generateTypeDefinitions(openApiSpec)}
`;
    
    return sdkCode;
  }
  
  async generatePythonSDK(
    openApiSpec: OpenAPIV3.Document
  ): Promise<string> {
    // Python SDK generation logic
    return `# EventfulHQ Python SDK implementation...`;
  }
  
  async generatePHPSDK(
    openApiSpec: OpenAPIV3.Document
  ): Promise<string> {
    // PHP SDK generation logic
    return `<?php // EventfulHQ PHP SDK implementation...`;
  }
}
```

### Interactive API Playground

```typescript
// Frontend component for API playground
// components/developer/ApiPlayground.tsx
import React, { useState } from 'react';
import {
  Box,
  Paper,
  TextField,
  Button,
  Select,
  MenuItem,
  Typography,
  Tabs,
  Tab,
  CircularProgress,
  Alert
} from '@mui/material';
import { Prism as SyntaxHighlighter } from 'react-syntax-highlighter';
import { vscDarkPlus } from 'react-syntax-highlighter/dist/esm/styles/prism';
import { useApiPlayground } from '@/hooks/useApiPlayground';

export function ApiPlayground() {
  const [endpoint, setEndpoint] = useState('/api/v1/events');
  const [method, setMethod] = useState('GET');
  const [headers, setHeaders] = useState({
    'Authorization': 'Bearer YOUR_TOKEN',
    'Content-Type': 'application/json'
  });
  const [body, setBody] = useState('');
  const [response, setResponse] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  const [tab, setTab] = useState(0);
  
  const { executeRequest, generateCode } = useApiPlayground();
  
  const handleExecute = async () => {
    setLoading(true);
    setError(null);
    
    try {
      const result = await executeRequest({
        endpoint,
        method,
        headers,
        body: body ? JSON.parse(body) : undefined
      });
      
      setResponse(result);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  };
  
  const codeExamples = generateCode({ endpoint, method, headers, body });
  
  return (
    <Box sx={{ display: 'flex', gap: 3, height: '100vh' }}>
      {/* Request Builder */}
      <Paper sx={{ flex: 1, p: 3, overflow: 'auto' }}>
        <Typography variant="h5" gutterBottom>
          API Playground
        </Typography>
        
        <Box sx={{ display: 'flex', gap: 2, mb: 3 }}>
          <Select
            value={method}
            onChange={(e) => setMethod(e.target.value)}
            sx={{ minWidth: 120 }}
          >
            <MenuItem value="GET">GET</MenuItem>
            <MenuItem value="POST">POST</MenuItem>
            <MenuItem value="PUT">PUT</MenuItem>
            <MenuItem value="DELETE">DELETE</MenuItem>
          </Select>
          
          <TextField
            fullWidth
            value={endpoint}
            onChange={(e) => setEndpoint(e.target.value)}
            placeholder="/api/v1/events"
          />
        </Box>
        
        <Typography variant="h6" gutterBottom>
          Headers
        </Typography>
        <TextField
          fullWidth
          multiline
          rows={4}
          value={JSON.stringify(headers, null, 2)}
          onChange={(e) => {
            try {
              setHeaders(JSON.parse(e.target.value));
            } catch {}
          }}
          sx={{ mb: 3 }}
        />
        
        {method !== 'GET' && (
          <>
            <Typography variant="h6" gutterBottom>
              Request Body
            </Typography>
            <TextField
              fullWidth
              multiline
              rows={8}
              value={body}
              onChange={(e) => setBody(e.target.value)}
              placeholder={JSON.stringify({
                title: 'Summer Festival',
                eventDate: '2024-07-15T18:00:00Z'
              }, null, 2)}
              sx={{ mb: 3 }}
            />
          </>
        )}
        
        <Button
          variant="contained"
          onClick={handleExecute}
          disabled={loading}
          fullWidth
        >
          {loading ? <CircularProgress size={24} /> : 'Execute Request'}
        </Button>
        
        {error && (
          <Alert severity="error" sx={{ mt: 2 }}>
            {error}
          </Alert>
        )}
      </Paper>
      
      {/* Response/Code Examples */}
      <Paper sx={{ flex: 1, p: 3, overflow: 'auto' }}>
        <Tabs value={tab} onChange={(e, v) => setTab(v)}>
          <Tab label="Response" />
          <Tab label="cURL" />
          <Tab label="JavaScript" />
          <Tab label="Python" />
          <Tab label="PHP" />
        </Tabs>
        
        <Box sx={{ mt: 3 }}>
          {tab === 0 && response && (
            <SyntaxHighlighter language="json" style={vscDarkPlus}>
              {JSON.stringify(response, null, 2)}
            </SyntaxHighlighter>
          )}
          
          {tab === 1 && (
            <SyntaxHighlighter language="bash" style={vscDarkPlus}>
              {codeExamples.curl}
            </SyntaxHighlighter>
          )}
          
          {tab === 2 && (
            <SyntaxHighlighter language="javascript" style={vscDarkPlus}>
              {codeExamples.javascript}
            </SyntaxHighlighter>
          )}
          
          {tab === 3 && (
            <SyntaxHighlighter language="python" style={vscDarkPlus}>
              {codeExamples.python}
            </SyntaxHighlighter>
          )}
          
          {tab === 4 && (
            <SyntaxHighlighter language="php" style={vscDarkPlus}>
              {codeExamples.php}
            </SyntaxHighlighter>
          )}
        </Box>
      </Paper>
    </Box>
  );
}
```

## SuperAdmin Management System

### SuperAdmin Types and Permissions

```typescript
// types/superadmin.ts
export enum SuperAdminType {
  GLOBAL_ADMIN = 'GLOBAL_ADMIN',               // Full platform access
  REGIONAL_ADMIN = 'REGIONAL_ADMIN',           // Regional management
  CONTENT_ADMIN = 'CONTENT_ADMIN',             // Content moderation
  FINANCE_ADMIN = 'FINANCE_ADMIN',             // Financial operations
  SUPPORT_ADMIN = 'SUPPORT_ADMIN',             // Customer support
  MARKETING_ADMIN = 'MARKETING_ADMIN',         // Marketing campaigns
  TECH_ADMIN = 'TECH_ADMIN',                   // Technical operations
  SECURITY_ADMIN = 'SECURITY_ADMIN',           // Security management
  COMPLIANCE_ADMIN = 'COMPLIANCE_ADMIN',       // Legal compliance
  ANALYTICS_ADMIN = 'ANALYTICS_ADMIN',         // Analytics access
  INTEGRATION_ADMIN = 'INTEGRATION_ADMIN'      // Third-party integrations
}

export interface SuperAdminPermissions {
  // Global permissions
  canAccessAllRegions: boolean;
  canModifySystemSettings: boolean;
  canViewAllData: boolean;
  canDeleteAnyContent: boolean;
  canImpersonateUsers: boolean;
  canAccessFinancials: boolean;
  canManageIntegrations: boolean;
  
  // Service-specific permissions
  services: {
    [serviceName: string]: {
      read: boolean;
      write: boolean;
      delete: boolean;
      admin: boolean;
    };
  };
  
  // Regional restrictions
  allowedRegions?: string[];
  
  // Time-based restrictions
  accessHours?: {
    start: string; // "09:00"
    end: string;   // "18:00"
    timezone: string;
  };
  
  // IP restrictions
  allowedIPs?: string[];
}

// Permission matrix
export const SUPERADMIN_PERMISSIONS: Record<SuperAdminType, SuperAdminPermissions> = {
  [SuperAdminType.GLOBAL_ADMIN]: {
    canAccessAllRegions: true,
    canModifySystemSettings: true,
    canViewAllData: true,
    canDeleteAnyContent: true,
    canImpersonateUsers: true,
    canAccessFinancials: true,
    canManageIntegrations: true,
    services: {
      '*': { read: true, write: true, delete: true, admin: true }
    }
  },
  
  [SuperAdminType.REGIONAL_ADMIN]: {
    canAccessAllRegions: false,
    canModifySystemSettings: false,
    canViewAllData: false,
    canDeleteAnyContent: true,
    canImpersonateUsers: false,
    canAccessFinancials: false,
    canManageIntegrations: false,
    services: {
      'events': { read: true, write: true, delete: true, admin: false },
      'users': { read: true, write: true, delete: false, admin: false },
      'content': { read: true, write: true, delete: true, admin: false }
    }
  },
  
  [SuperAdminType.CONTENT_ADMIN]: {
    canAccessAllRegions: true,
    canModifySystemSettings: false,
    canViewAllData: false,
    canDeleteAnyContent: true,
    canImpersonateUsers: false,
    canAccessFinancials: false,
    canManageIntegrations: false,
    services: {
      'content': { read: true, write: true, delete: true, admin: true },
      'media': { read: true, write: true, delete: true, admin: true },
      'events': { read: true, write: true, delete: false, admin: false }
    }
  },
  
  // ... other admin types
};
```

### SuperAdmin API Implementation

```typescript
// services/admin/handlers/superadmin.ts
import { APIGatewayProxyEventV2, APIGatewayProxyResultV2 } from 'aws-lambda';
import { SuperAdminService } from '../services/superadmin.service';
import { validateSuperAdmin } from '../middleware/superadmin.middleware';

const service = new SuperAdminService();

// System-wide broadcast
export const systemBroadcast = validateSuperAdmin(
  [SuperAdminType.GLOBAL_ADMIN, SuperAdminType.MARKETING_ADMIN],
  async (event: APIGatewayProxyEventV2): Promise<APIGatewayProxyResultV2> => {
    const { title, message, channels, targetRegions } = JSON.parse(event.body || '{}');
    
    await service.sendBroadcast({
      title,
      message,
      channels, // ['email', 'whatsapp', 'push', 'in-app']
      targetRegions: targetRegions || ['all'],
      sentBy: event.requestContext.authorizer?.lambda.userId
    });
    
    return {
      statusCode: 200,
      body: JSON.stringify({ success: true })
    };
  }
);

// User impersonation
export const impersonateUser = validateSuperAdmin(
  [SuperAdminType.GLOBAL_ADMIN, SuperAdminType.SUPPORT_ADMIN],
  async (event: APIGatewayProxyEventV2): Promise<APIGatewayProxyResultV2> => {
    const { userId } = event.pathParameters || {};
    const adminUserId = event.requestContext.authorizer?.lambda.userId;
    
    // Create impersonation session
    const session = await service.createImpersonationSession(
      adminUserId,
      userId!
    );
    
    // Log for audit
    await service.logAdminAction({
      adminId: adminUserId,
      action: 'USER_IMPERSONATION',
      targetUserId: userId,
      timestamp: new Date(),
      ip: event.requestContext.http.sourceIp
    });
    
    return {
      statusCode: 200,
      body: JSON.stringify({
        sessionToken: session.token,
        expiresAt: session.expiresAt
      })
    };
  }
);

// Financial override
export const overridePayment = validateSuperAdmin(
  [SuperAdminType.GLOBAL_ADMIN, SuperAdminType.FINANCE_ADMIN],
  async (event: APIGatewayProxyEventV2): Promise<APIGatewayProxyResultV2> => {
    const { paymentId, action, reason } = JSON.parse(event.body || '{}');
    
    const result = await service.overridePayment(
      paymentId,
      action, // 'refund', 'void', 'capture'
      reason,
      event.requestContext.authorizer?.lambda.userId
    );
    
    return {
      statusCode: 200,
      body: JSON.stringify(result)
    };
  }
);

// Platform configuration
export const updatePlatformConfig = validateSuperAdmin(
  [SuperAdminType.GLOBAL_ADMIN, SuperAdminType.TECH_ADMIN],
  async (event: APIGatewayProxyEventV2): Promise<APIGatewayProxyResultV2> => {
    const { configKey, configValue, region } = JSON.parse(event.body || '{}');
    
    await service.updateConfiguration(
      configKey,
      configValue,
      region || 'global'
    );
    
    return {
      statusCode: 200,
      body: JSON.stringify({ success: true })
    };
  }
);

// LLM configuration
export const configureLLM = validateSuperAdmin(
  [SuperAdminType.GLOBAL_ADMIN, SuperAdminType.TECH_ADMIN],
  async (event: APIGatewayProxyEventV2): Promise<APIGatewayProxyResultV2> => {
    const { provider, config } = JSON.parse(event.body || '{}');
    
    await service.configureLLMProvider({
      provider, // 'bedrock', 'openai', 'anthropic', etc.
      apiKey: config.apiKey,
      modelId: config.modelId,
      maxTokens: config.maxTokens,
      temperature: config.temperature,
      quotaLimit: config.quotaLimit,
      costLimit: config.costLimit
    });
    
    return {
      statusCode: 200,
      body: JSON.stringify({ success: true })
    };
  }
);
```

### SuperAdmin Dashboard

```tsx
// app/(admin)/superadmin/dashboard/page.tsx
'use client';

import React from 'react';
import {
  Grid,
  Card,
  CardContent,
  Typography,
  Box,
  Button,
  Alert,
  Tabs,
  Tab
} from '@mui/material';
import {
  TrendingUp,
  Warning,
  People,
  AttachMoney,
  CloudQueue,
  Security
} from '@mui/icons-material';
import { useSuperAdminDashboard } from '@/hooks/useSuperAdminDashboard';
import { MetricCard } from '@/components/admin/MetricCard';
import { SystemHealth } from '@/components/admin/SystemHealth';
import { RealtimeMetrics } from '@/components/admin/RealtimeMetrics';
import { AdminAuditLog } from '@/components/admin/AdminAuditLog';

export default function SuperAdminDashboard() {
  const { data, isLoading } = useSuperAdminDashboard();
  const [tab, setTab] = useState(0);
  
  if (isLoading) return <LoadingScreen />;
  
  return (
    <Box>
      <Typography variant="h3" gutterBottom>
        SuperAdmin Dashboard
      </Typography>
      
      {/* System Alerts */}
      {data.alerts.map((alert) => (
        <Alert 
          key={alert.id} 
          severity={alert.severity}
          sx={{ mb: 2 }}
          action={
            <Button size="small" color="inherit">
              Acknowledge
            </Button>
          }
        >
          {alert.message}
        </Alert>
      ))}
      
      {/* Key Metrics */}
      <Grid container spacing={3} sx={{ mb: 4 }}>
        <Grid item xs={12} sm={6} md={3}>
          <MetricCard
            title="Total Users"
            value={data.metrics.totalUsers}
            change={data.metrics.userGrowth}
            icon={<People />}
          />
        </Grid>
        <Grid item xs={12} sm={6} md={3}>
          <MetricCard
            title="Revenue (MTD)"
            value={`${data.metrics.monthlyRevenue.toLocaleString()}`}
            change={data.metrics.revenueGrowth}
            icon={<AttachMoney />}
          />
        </Grid>
        <Grid item xs={12} sm={6} md={3}>
          <MetricCard
            title="Active Events"
            value={data.metrics.activeEvents}
            change={data.metrics.eventGrowth}
            icon={<TrendingUp />}
          />
        </Grid>
        <Grid item xs={12} sm={6} md={3}>
          <MetricCard
            title="System Health"
            value={`${data.metrics.systemHealth}%`}
            status={data.metrics.systemHealth > 95 ? 'success' : 'warning'}
            icon={<CloudQueue />}
          />
        </Grid>
      </Grid>
      
      {/* Detailed Views */}
      <Card>
        <Tabs value={tab} onChange={(e, v) => setTab(v)}>
          <Tab label="System Health" />
          <Tab label="Real-time Metrics" />
          <Tab label="Audit Log" />
          <Tab label="Configuration" />
          <Tab label="Integrations" />
        </Tabs>
        
        <CardContent>
          {tab === 0 && <SystemHealth />}
          {tab === 1 && <RealtimeMetrics />}
          {tab === 2 && <AdminAuditLog />}
          {tab === 3 && <PlatformConfiguration />}
          {tab === 4 && <IntegrationManager />}
        </CardContent>
      </Card>
    </Box>
  );
}
```

## Service Communication Patterns

### Event-Driven Architecture

```typescript
// packages/shared/events/event-bus.ts
import { EventBridge } from '@aws-sdk/client-eventbridge';
import { z } from 'zod';

export class EventBus {
  private eventBridge: EventBridge;
  private source: string;
  
  constructor(serviceName: string) {
    this.eventBridge = new EventBridge({});
    this.source = `eventfulhq.${serviceName}`;
  }
  
  async publish<T extends z.ZodSchema>(
    eventType: string,
    schema: T,
    data: z.infer<T>
  ): Promise<void> {
    // Validate data against schema
    const validatedData = schema.parse(data);
    
    const event = {
      Source: this.source,
      DetailType: eventType,
      Detail: JSON.stringify({
        ...validatedData,
        eventId: crypto.randomUUID(),
        timestamp: new Date().toISOString(),
        correlationId: data.correlationId || crypto.randomUUID()
      }),
      EventBusName: process.env.EVENT_BUS_NAME || 'eventfulhq-event-bus'
    };
    
    await this.eventBridge.putEvents({
      Entries: [event]
    });
  }
  
  // Event schemas
  static schemas = {
    UserCreated: z.object({
      userId: z.string().uuid(),
      email: z.string().email(),
      userType: z.enum(['fan', 'artist', 'expert', 'venue', 'organizer']),
      region: z.string(),
      correlationId: z.string().optional()
    }),
    
    EventCreated: z.object({
      eventId: z.string().uuid(),
      title: z.string(),
      organizerId: z.string().uuid(),
      eventDate: z.string().datetime(),
      location: z.object({
        city: z.string(),
        country: z.string()
      }),
      correlationId: z.string().optional()
    }),
    
    PaymentCompleted: z.object({
      paymentId: z.string().uuid(),
      userId: z.string().uuid(),
      amount: z.number(),
      currency: z.string(),
      type: z.enum(['ticket', 'subscription', 'booking']),
      metadata: z.record(z.any()).optional(),
      correlationId: z.string().optional()
    }),
    
    TicketPurchased: z.object({
      ticketId: z.string().uuid(),
      eventId: z.string().uuid(),
      userId: z.string().uuid(),
      quantity: z.number(),
      totalAmount: z.number(),
      correlationId: z.string().optional()
    })
  };
}

// Usage example
const eventBus = new EventBus('event-service');

await eventBus.publish(
  'EVENT_CREATED',
  EventBus.schemas.EventCreated,
  {
    eventId: event.id,
    title: event.title,
    organizerId: event.organizerId,
    eventDate: event.eventDate.toISOString(),
    location: {
      city: event.location.city,
      country: event.location.country
    }
  }
);
```

### Service Registry Pattern

```typescript
// packages/shared/discovery/service-registry.ts
import { DynamoDB } from '@aws-sdk/client-dynamodb';
import { DynamoDBDocument } from '@aws-sdk/lib-dynamodb';

export interface ServiceEndpoint {
  serviceName: string;
  version: string;
  endpoint: string;
  healthCheck: string;
  region: string;
  status: 'healthy' | 'unhealthy' | 'degraded';
  lastHeartbeat: Date;
  metadata?: Record<string, any>;
}

export class ServiceRegistry {
  private docClient: DynamoDBDocument;
  private tableName = 'eventfulhq-service-registry';
  
  constructor() {
    const client = new DynamoDB({});
    this.docClient = DynamoDBDocument.from(client);
  }
  
  async register(service: ServiceEndpoint): Promise<void> {
    await this.docClient.put({
      TableName: this.tableName,
      Item: {
        ...service,
        ttl: Math.floor(Date.now() / 1000) + 300 // 5 minutes TTL
      }
    });
  }
  
  async discover(serviceName: string, region?: string): Promise<ServiceEndpoint[]> {
    const params = {
      TableName: this.tableName,
      KeyConditionExpression: 'serviceName = :serviceName',
      ExpressionAttributeValues: {
        ':serviceName': serviceName
      }
    };
    
    if (region) {
      params.FilterExpression = 'region = :region';
      params.ExpressionAttributeValues[':region'] = region;
    }
    
    const result = await this.docClient.query(params);
    return result.Items as ServiceEndpoint[];
  }
  
  async heartbeat(serviceName: string, endpoint: string): Promise<void> {
    await this.docClient.update({
      TableName: this.tableName,
      Key: { serviceName, endpoint },
      UpdateExpression: 'SET lastHeartbeat = :now, #status = :status',
      ExpressionAttributeNames: {
        '#status': 'status'
      },
      ExpressionAttributeValues: {
        ':now': new Date().toISOString(),
        ':status': 'healthy'
      }
    });
  }
}
```

## Error Handling & Logging

### Centralized Error Handler

```typescript
// packages/shared/errors/error-handler.ts
import { Logger } from '@aws-lambda-powertools/logger';
import { Metrics } from '@aws-lambda-powertools/metrics';
import { SQS } from '@aws-sdk/client-sqs';

export class ErrorHandler {
  private logger: Logger;
  private metrics: Metrics;
  private sqs: SQS;
  private dlqUrl: string;
  
  constructor(serviceName: string) {
    this.logger = new Logger({ serviceName });
    this.metrics = new Metrics({ namespace: 'EventfulHQ', serviceName });
    this.sqs = new SQS({});
    this.dlqUrl = process.env.ERROR_DLQ_URL!;
  }
  
  async handle(error: Error, context: any): Promise<void> {
    // Log error with context
    this.logger.error('Error occurred', error, {
      errorType: error.constructor.name,
      errorMessage: error.message,
      stackTrace: error.stack,
      context
    });
    
    // Record metrics
    this.metrics.addMetric('ErrorCount', 1);
    this.metrics.addMetadata('errorType', error.constructor.name);
    this.metrics.addMetadata('operation', context.operation);
    
    // Send to DLQ for critical errors
    if (this.isCriticalError(error)) {
      await this.sendToDLQ(error, context);
    }
    
    // Alert on high error rates
    if (await this.checkErrorRate()) {
      await this.triggerAlert(error, context);
    }
  }
  
  private isCriticalError(error: Error): boolean {
    const criticalErrors = [
      'PaymentError',
      'SecurityError',
      'DataIntegrityError',
      'SystemError'
    ];
    
    return criticalErrors.includes(error.constructor.name);
  }
  
  private async sendToDLQ(error: Error, context: any): Promise<void> {
    await this.sqs.sendMessage({
      QueueUrl: this.dlqUrl,
      MessageBody: JSON.stringify({
        error: {
          name: error.name,
          message: error.message,
          stack: error.stack
        },
        context,
        timestamp: new Date().toISOString(),
        service: this.logger.getServiceName()
      })
    });
  }
  
  private async checkErrorRate(): Promise<boolean> {
    // Check if error rate exceeds threshold
    // Implementation depends on metrics storage
    return false;
  }
  
  private async triggerAlert(error: Error, context: any): Promise<void> {
    // Send alert via SNS/PagerDuty/Slack
    // Implementation depends on alerting service
  }
}
```

### Structured Logging

```typescript
// packages/shared/logging/logger.ts
import { Logger as PowertoolsLogger } from '@aws-lambda-powertools/logger';
import { LogLevel } from '@aws-lambda-powertools/logger/lib/types';

export class Logger extends PowertoolsLogger {
  constructor(serviceName: string) {
    super({
      serviceName,
      logLevel: (process.env.LOG_LEVEL as LogLevel) || 'INFO',
      sampleRateValue: parseFloat(process.env.LOG_SAMPLE_RATE || '0.1')
    });
  }
  
  // Log API request
  logRequest(
    method: string,
    path: string,
    userId?: string,
    correlationId?: string
  ): void {
    this.info('API Request', {
      method,
      path,
      userId,
      correlationId,
      timestamp: new Date().toISOString()
    });
  }
  
  // Log API response
  logResponse(
    statusCode: number,
    duration: number,
    correlationId?: string
  ): void {
    this.info('API Response', {
      statusCode,
      duration,
      correlationId,
      timestamp: new Date().toISOString()
    });
  }
  
  // Log business event
  logBusinessEvent(
    eventType: string,
    data: any,
    userId?: string
  ): void {
    this.info('Business Event', {
      eventType,
      data,
      userId,
      timestamp: new Date().toISOString()
    });
  }
  
  // Log integration call
  logIntegration(
    service: string,
    operation: string,
    success: boolean,
    duration: number,
    error?: any
  ): void {
    const level = success ? 'info' : 'error';
    this[level]('Integration Call', {
      service,
      operation,
      success,
      duration,
      error,
      timestamp: new Date().toISOString()
    });
  }
}
```

## Testing & Quality Standards

### Unit Testing Template

```typescript
// Example unit test
// services/events/tests/event.service.spec.ts
import { Test, TestingModule } from '@nestjs/testing';
import { EventService } from '../event.service';
import { getRepositoryToken } from '@nestjs/typeorm';
import { Event } from '../entities/event.entity';
import { Repository } from 'typeorm';
import { EventBus } from '@eventfulhq/shared-events';
import { AIService } from '@eventfulhq/ai-service';

describe('EventService', () => {
  let service: EventService;
  let repository: Repository<Event>;
  let eventBus: EventBus;
  let aiService: AIService;
  
  const mockRepository = {
    create: jest.fn(),
    save: jest.fn(),
    findOne: jest.fn(),
    find: jest.fn(),
    update: jest.fn(),
    softDelete: jest.fn()
  };
  
  const mockEventBus = {
    publish: jest.fn()
  };
  
  const mockAIService = {
    generateContent: jest.fn(),
    translate: jest.fn(),
    generateTags: jest.fn()
  };
  
  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [
        EventService,
        {
          provide: getRepositoryToken(Event),
          useValue: mockRepository
        },
        {
          provide: EventBus,
          useValue: mockEventBus
        },
        {
          provide: AIService,
          useValue: mockAIService
        }
      ]
    }).compile();
    
    service = module.get<EventService>(EventService);
    repository = module.get<Repository<Event>>(getRepositoryToken(Event));
    eventBus = module.get<EventBus>(EventBus);
    aiService = module.get<AIService>(AIService);
  });
  
  afterEach(() => {
    jest.clearAllMocks();
  });
  
  describe('createEvent', () => {
    it('should create event with AI-generated content', async () => {
      // Arrange
      const createEventDto = {
        title: 'Test Event',
        eventDate: new Date('2024-07-15'),
        location: {
          city: 'New York',
          country: 'USA'
        },
        capacity: 100,
        aiGenerate: true
      };
      
      const aiGeneratedContent = {
        description: 'AI generated description...',
        tags: ['music', 'summer', 'festival']
      };
      
      const savedEvent = {
        id: 'event-123',
        ...createEventDto,
        ...aiGeneratedContent
      };
      
      mockAIService.generateContent.mockResolvedValue(aiGeneratedContent);
      mockRepository.create.mockReturnValue(savedEvent);
      mockRepository.save.mockResolvedValue(savedEvent);
      
      // Act
      const result = await service.createEvent(createEventDto, 'user-123');
      
      // Assert
      expect(aiService.generateContent).toHaveBeenCalledWith({
        type: 'event',
        data: createEventDto
      });
      
      expect(repository.create).toHaveBeenCalledWith({
        ...createEventDto,
        ...aiGeneratedContent,
        createdBy: 'user-123'
      });
      
      expect(eventBus.publish).toHaveBeenCalledWith(
        'EVENT_CREATED',
        expect.any(Object),
        expect.objectContaining({
          eventId: savedEvent.id,
          title: savedEvent.title
        })
      );
      
      expect(result).toEqual(savedEvent);
    });
    
    it('should validate event date is in the future', async () => {
      // Arrange
      const createEventDto = {
        title: 'Past Event',
        eventDate: new Date('2020-01-01'),
        location: { city: 'NYC', country: 'USA' },
        capacity: 100
      };
      
      // Act & Assert
      await expect(
        service.createEvent(createEventDto, 'user-123')
      ).rejects.toThrow('Event date must be in the future');
    });
    
    it('should enforce capacity limits by subscription tier', async () => {
      // Test implementation
    });
  });
  
  describe('searchEvents', () => {
    it('should search events with filters', async () => {
      // Test implementation
    });
    
    it('should apply regional filtering', async () => {
      // Test implementation
    });
  });
});
```

### Integration Testing

```typescript
// Example integration test
// services/events/tests/event.integration.spec.ts
import { Test } from '@nestjs/testing';
import { INestApplication } from '@nestjs/common';
import request from 'supertest';
import { AppModule } from '../app.module';
import { DataSource } from 'typeorm';

describe('EventController (integration)', () => {
  let app: INestApplication;
  let dataSource: DataSource;
  let authToken: string;
  
  beforeAll(async () => {
    const moduleRef = await Test.createTestingModule({
      imports: [AppModule]
    }).compile();
    
    app = moduleRef.createNestApplication();
    await app.init();
    
    dataSource = moduleRef.get<DataSource>(DataSource);
    
    // Get auth token
    const authResponse = await request(app.getHttpServer())
      .post('/api/v1/auth/whatsapp/verify-otp')
      .send({
        phoneNumber: '+1234567890',
        otp: '123456' // Test OTP
      });
    
    authToken = authResponse.body.accessToken;
  });
  
  afterAll(async () => {
    await dataSource.destroy();
    await app.close();
  });
  
  describe('POST /api/v1/events', () => {
    it('should create event successfully', async () => {
      const response = await request(app.getHttpServer())
        .post('/api/v1/events')
        .set('Authorization', `Bearer ${authToken}`)
        .set('X-Preferred-Language', 'en')
        .send({
          title: 'Integration Test Event',
          description: 'Test description',
          eventDate: '2024-12-25T18:00:00Z',
          location: {
            address: '123 Test St',
            city: 'New York',
            state: 'NY',
            country: 'USA',
            coordinates: {
              lat: 40.7128,
              lng: -74.0060
            }
          },
          capacity: 100,
          category: 'music',
          ticketTypes: [
            {
              name: 'General',
              price: 50,
              quantity: 80
            },
            {
              name: 'VIP',
              price: 150,
              quantity: 20
            }
          ]
        })
        .expect(201);
      
      expect(response.body).toMatchObject({
        id: expect.any(String),
        title: 'Integration Test Event',
        status: 'draft',
        qrCode: expect.any(String),
        micrositeUrl: expect.stringContaining('eventfulhq.com')
      });
      
      // Verify event was saved to database
      const savedEvent = await dataSource
        .getRepository('Event')
        .findOne({ where: { id: response.body.id } });
      
      expect(savedEvent).toBeTruthy();
      expect(savedEvent.title).toBe('Integration Test Event');
    });
    
    it('should enforce rate limiting', async () => {
      // Send multiple requests to trigger rate limit
      const requests = Array(101).fill(null).map(() =>
        request(app.getHttpServer())
          .get('/api/v1/events')
          .set('Authorization', `Bearer ${authToken}`)
      );
      
      const responses = await Promise.all(requests);
      const rateLimited = responses.filter(r => r.status === 429);
      
      expect(rateLimited.length).toBeGreaterThan(0);
      expect(rateLimited[0].headers['x-ratelimit-limit']).toBe('100');
      expect(rateLimited[0].headers['x-ratelimit-remaining']).toBe('0');
    });
  });
});
```

### Performance Testing

```typescript
// performance-tests/load-test.js
import http from 'k6/http';
import { check, sleep } from 'k6';
import { Rate } from 'k6/metrics';

const errorRate = new Rate('errors');

export const options = {
  stages: [
    { duration: '2m', target: 100 },  // Ramp up to 100 users
    { duration: '5m', target: 100 },  // Stay at 100 users
    { duration: '2m', target: 200 },  // Ramp up to 200 users
    { duration: '5m', target: 200 },  // Stay at 200 users
    { duration: '2m', target: 0 },    // Ramp down to 0 users
  ],
  thresholds: {
    http_req_duration: ['p(95)<500'], // 95% of requests under 500ms
    errors: ['rate<0.1'],             // Error rate under 10%
  },
};

const BASE_URL = 'https://api.eventfulhq.com';

export default function () {
  // Test event listing
  const listResponse = http.get(`${BASE_URL}/api/v1/events`, {
    headers: {
      'Authorization': `Bearer ${__ENV.AUTH_TOKEN}`,
      'X-Preferred-Language': 'en',
    },
  });
  
  check(listResponse, {
    'list events status is 200': (r) => r.status === 200,
    'list events response time < 500ms': (r) => r.timings.duration < 500,
  });
  
  errorRate.add(listResponse.status !== 200);
  
  sleep(1);
  
  // Test event creation
  const createResponse = http.post(
    `${BASE_URL}/api/v1/events`,
    JSON.stringify({
      title: `Load Test Event ${Date.now()}`,
      eventDate: '2024-12-31T23:59:59Z',
      location: {
        city: 'New York',
        country: 'USA',
      },
      capacity: 100,
    }),
    {
      headers: {
        'Authorization': `Bearer ${__ENV.AUTH_TOKEN}`,
        'Content-Type': 'application/json',
      },
    }
  );
  
  check(createResponse, {
    'create event status is 201': (r) => r.status === 201,
    'create event response time < 1000ms': (r) => r.timings.duration < 1000,
  });
  
  errorRate.add(createResponse.status !== 201);
  
  sleep(1);
}
```

## Deployment & CI/CD Pipeline

### GitHub Actions Workflow

```yaml
# .github/workflows/deploy.yml
name: Deploy to AWS

on:
  push:
    branches: [main, staging, develop]
  pull_request:
    branches: [main]

env:
  AWS_REGION: us-east-1
  NODE_VERSION: 20.x

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run linting
        run: npm run lint
      
      - name: Run tests
        run: npm run test:ci
        env:
          CI: true
      
      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          token: ${{ secrets.CODECOV_TOKEN }}
  
  build:
    needs: test
    runs-on: ubuntu-latest
    strategy:
      matrix:
        service: [
          auth-service,
          event-service,
          ticket-service,
          payment-service,
          user-service,
          messaging-service,
          ai-service
        ]
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Build service
        run: npm run build:${{ matrix.service }}
      
      - name: Package Lambda
        run: |
          cd services/${{ matrix.service }}
          zip -r ../../${{ matrix.service }}.zip dist/ node_modules/
      
      - name: Upload artifact
        uses: actions/upload-artifact@v3
        with:
          name: ${{ matrix.service }}-package
          path: ${{ matrix.service }}.zip
  
  deploy:
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main' || github.ref == 'refs/heads/staging'
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ env.AWS_REGION }}
      
      - name: Download artifacts
        uses: actions/download-artifact@v3
      
      - name: Deploy to AWS
        run: |
          STAGE=${{ github.ref == 'refs/heads/main' && 'prod' || 'staging' }}
          npm run deploy:all -- --stage=$STAGE
      
      - name: Run smoke tests
        run: npm run test:smoke -- --stage=$STAGE
      
      - name: Notify deployment
        uses: 8398a7/action-slack@v3
        with:
          status: ${{ job.status }}
          text: 'Deployment to ${{ env.STAGE }} completed'
          webhook_url: ${{ secrets.SLACK_WEBHOOK }}
        if: always()
```

### Serverless Framework Deployment

```yaml
# serverless.yml
service: eventfulhq-${self:custom.serviceName}

frameworkVersion: '3'

provider:
  name: aws
  runtime: nodejs20.x
  stage: ${opt:stage, 'dev'}
  region: ${opt:region, 'us-east-1'}
  
  tracing:
    lambda: true
    apiGateway: true
  
  logs:
    restApi:
      accessLogging: true
      format: 'requestId: $context.requestId'
      executionLogging: true
      level: INFO
      fullExecutionData: true
  
  environment:
    STAGE: ${self:provider.stage}
    SERVICE_NAME: ${self:custom.serviceName}
    EVENT_BUS_NAME: eventfulhq-event-bus-${self:provider.stage}
    
  iam:
    role:
      statements:
        - Effect: Allow
          Action:
            - dynamodb:Query
            - dynamodb:Scan
            - dynamodb:GetItem
            - dynamodb:PutItem
            - dynamodb:UpdateItem
            - dynamodb:DeleteItem
          Resource:
            - arn:aws:dynamodb:${aws:region}:${aws:accountId}:table/eventfulhq-*
        
        - Effect: Allow
          Action:
            - events:PutEvents
          Resource:
            - arn:aws:events:${aws:region}:${aws:accountId}:event-bus/eventfulhq-*
        
        - Effect: Allow
          Action:
            - xray:PutTraceSegments
            - xray:PutTelemetryRecords
          Resource: '*'

custom:
  serviceName: ${env:SERVICE_NAME}
  
  prune:
    automatic: true
    number: 3
  
  warmup:
    default:
      enabled: ${self:custom.warmupEnabled.${self:provider.stage}}
      events:
        - schedule: rate(5 minutes)
      concurrency: 1
  
  warmupEnabled:
    dev: false
    staging: true
    prod: true
  
  customDomain:
    domainName: ${self:custom.domains.${self:provider.stage}}
    stage: ${self:provider.stage}
    certificateName: '*.eventfulhq.com'
    createRoute53Record: true
    endpointType: regional
    securityPolicy: tls_1_2
  
  domains:
    dev: dev-api.eventfulhq.com
    staging: staging-api.eventfulhq.com
    prod: api.eventfulhq.com

functions:
  - ${file(./functions.yml)}

plugins:
  - serverless-webpack
  - serverless-offline
  - serverless-domain-manager
  - serverless-prune-plugin
  - serverless-plugin-warmup

resources:
  - ${file(./resources/dynamodb.yml)}
  - ${file(./resources/s3.yml)}
  - ${file(./resources/cloudwatch.yml)}
```

## Frontend Components Wishlist

### Component Hierarchy and Naming Convention

```typescript
// Frontend components structure with MatDash theme integration
export const COMPONENT_WISHLIST = {
  // Layout Components
  layout: {
    'AppLayout': 'Main application layout wrapper',
    'DashboardLayout': 'Dashboard specific layout',
    'AuthLayout': 'Authentication pages layout',
    'PublicLayout': 'Public pages layout',
    'AdminLayout': 'SuperAdmin layout',
    'MobileLayout': 'Mobile responsive layout',
    'SideNavigation': 'Collapsible side navigation',
    'TopBar': 'Top navigation bar with user menu',
    'Footer': 'Application footer',
    'Breadcrumbs': 'Dynamic breadcrumb navigation'
  },
  
  // Authentication Components
  auth: {
    'WhatsAppLogin': 'WhatsApp OTP login form',
    'EmailLogin': 'Email/password login form',
    'SocialLogin': 'Social media login buttons',
    'RegisterForm': 'User registration form',
    'OTPInput': 'OTP input component',
    'BiometricAuth': 'Biometric authentication',
    'MFASetup': 'Multi-factor auth setup',
    'ForgotPassword': 'Password recovery form',
    'SessionManager': 'Session timeout handler',
    'PermissionGate': 'Permission-based rendering'
  },
  
  // Event Components
  events: {
    'EventCard': 'Event display card',
    'EventGrid': 'Event grid layout',
    'EventList': 'Event list view',
    'EventDetails': 'Detailed event view',
    'EventForm': 'Create/edit event form',
    'EventCalendar': 'Calendar view of events',
    'EventSearch': 'Advanced event search',
    'EventFilters': 'Event filter sidebar',
    'EventMap': 'Map view of events',
    'EventGallery': 'Event photo gallery',
    'EventSchedule': 'Event timeline/schedule',
    'EventStats': 'Event statistics dashboard',
    'TrendingEvents': 'Trending events carousel',
    'EventQRCode': 'Event QR code generator',
    'EventShare': 'Event sharing options',
    'EventStaff': 'Staff management for events',
    'EventMicrosite': 'Event microsite preview'
  },
  
  // Ticketing Components
  tickets: {
    'TicketSelector': 'Ticket type selector',
    'TicketPurchase': 'Ticket purchase flow',
    'TicketCart': 'Ticket shopping cart',
    'TicketCheckout': 'Checkout process',
    'TicketConfirmation': 'Purchase confirmation',
    'TicketQR': 'Ticket QR code display',
    'TicketWallet': 'Digital wallet integration',
    'TicketTransfer': 'Ticket transfer interface',
    'TicketRefund': 'Refund request form',
    'TicketScanner': 'QR code scanner',
    'SeatingChart': 'Interactive seating chart',
    'TicketAnalytics': 'Sales analytics'
  },
  
  // Artist/Talent Components
  artists: {
    'ArtistCard': 'Artist profile card',
    'ArtistGrid': 'Artist grid layout',
    'ArtistProfile': 'Detailed artist profile',
    'ArtistBooking': 'Artist booking form',
    'ArtistCalendar': 'Availability calendar',
    'ArtistMedia': 'Media gallery',
    'ArtistReviews': 'Reviews and ratings',
    'ArtistSearch': 'Artist search interface',
    'ArtistCategories': 'Category browser',
    'TrendingArtists': 'Trending artists',
    'ArtistRecommendations': 'AI recommendations',
    'ArtistPortfolio': 'Portfolio showcase'
  },
  
  // Messaging Components
  messaging: {
    'ChatInterface': 'Main chat interface',
    'ConversationList': 'List of conversations',
    'MessageThread': 'Message thread view',
    'MessageComposer': 'Message input area',
    'WhatsAppChat': 'WhatsApp integration',
    'VoiceMessage': 'Voice message recorder',
    'MediaMessage': 'Media sharing',
    'GroupChat': 'Group chat interface',
    'ChatNotifications': 'Chat notifications',
    'TranslateMessage': 'Message translation',
    'ChatSearch': 'Search in messages'
  },
  
  // Payment Components
  payments: {
    'PaymentForm': 'Payment input form',
    'PaymentMethods': 'Payment method selector',
    'CardInput': 'Credit card input',
    'StripeElements': 'Stripe payment elements',
    'RazorpayCheckout': 'Razorpay integration',
    'PaymentHistory': 'Transaction history',
    'InvoiceViewer': 'Invoice display',
    'RefundForm': 'Refund request form',
    'SubscriptionManager': 'Subscription management',
    'PricingPlans': 'Pricing plan selector',
    'PaymentAnalytics': 'Payment analytics',
    'SplitPayment': 'Split payment interface'
  },
  
  // Discovery Components
  discovery: {
    'UniversalSearch': 'Global search bar',
    'SearchResults': 'Search results display',
    'CategoryBrowser': 'Category navigation',
    'TrendingSection': 'Trending content',
    'RecommendationFeed': 'Personalized feed',
    'NearbyDiscovery': 'Location-based discovery',
    'FilterPanel': 'Advanced filters',
    'SavedSearches': 'Saved search management',
    'SearchSuggestions': 'Search autocomplete',
    'DiscoveryMap': 'Map-based discovery'
  },
  
  // Campaign Components
  campaigns: {
    'CampaignCard': 'Campaign display card',
    'CampaignForm': 'Create campaign form',
    'CampaignDashboard': 'Campaign management',
    'CampaignAnalytics': 'Performance metrics',
    'InfluencerList': 'Influencer management',
    'CampaignTimeline': 'Campaign schedule',
    'CampaignBudget': 'Budget tracker',
    'CampaignROI': 'ROI calculator',
    'CampaignTemplates': 'Template selector'
  },
  
  // Team Components
  teams: {
    'TeamCard': 'Team profile card',
    'TeamMembers': 'Member management',
    'TeamPermissions': 'Permission matrix',
    'TeamProjects': 'Project list',
    'TeamChat': 'Team communication',
    'TeamCalendar': 'Shared calendar',
    'TeamAnalytics': 'Team performance',
    'JobBoard': 'Job postings',
    'ApplicationTracker': 'ATS pipeline',
    'TeamMicrosite': 'Team microsite'
  },
  
  // User Components
  users: {
    'UserProfile': 'User profile view',
    'ProfileEditor': 'Profile edit form',
    'UserSettings': 'Settings panel',
    'APIKeyManager': 'API key management',
    'NotificationPreferences': 'Notification settings',
    'PrivacySettings': 'Privacy controls',
    'SubscriptionStatus': 'Subscription info',
    'UserActivity': 'Activity timeline',
    'UserBadges': 'Achievement badges',
    'ReferralProgram': 'Referral interface'
  },
  
  // Admin Components
  admin: {
    'AdminDashboard': 'Admin overview',
    'UserManagement': 'User administration',
    'ContentModeration': 'Content review',
    'SystemHealth': 'System monitoring',
    'RevenueReports': 'Financial reports',
    'AuditLog': 'Audit trail viewer',
    'FeatureFlags': 'Feature management',
    'BroadcastComposer': 'System announcements',
    'SupportTickets': 'Support management',
    'IntegrationManager': 'Third-party integrations',
    'LLMConfiguration': 'AI model settings',
    'ComplianceReports': 'Compliance dashboard'
  },
  
  // Common Components
  common: {
    'LoadingSpinner': 'Loading indicator',
    'ErrorBoundary': 'Error handling wrapper',
    'EmptyState': 'Empty state display',
    'InfiniteScroll': 'Infinite scroll wrapper',
    'Pagination': 'Pagination controls',
    'DataTable': 'Advanced data table',
    'DatePicker': 'Date/time picker',
    'FileUploader': 'File upload component',
    'ImageCropper': 'Image editing tool',
    'VideoPlayer': 'Video playback',
    'AudioPlayer': 'Audio playback',
    'ShareModal': 'Social sharing modal',
    'QRScanner': 'QR code scanner',
    'LanguageSelector': 'Language switcher',
    'CurrencySelector': 'Currency switcher',
    'CountrySelector': 'Country picker',
    'PhoneInput': 'International phone input',
    'AddressAutocomplete': 'Address suggestions',
    'MapPicker': 'Location selector',
    'ColorPicker': 'Color selection',
    'EmojiPicker': 'Emoji selector',
    'RichTextEditor': 'WYSIWYG editor',
    'MarkdownEditor': 'Markdown editor',
    'CodeEditor': 'Code syntax editor',
    'CSVImporter': 'CSV upload parser',
    'ExportDialog': 'Data export options',
    'PrintPreview': 'Print preview modal',
    'TourGuide': 'Onboarding tour',
    'Tooltip': 'Info tooltips',
    'ConfirmDialog': 'Confirmation modal',
    'SnackbarQueue': 'Toast notifications',
    'ProgressBar': 'Progress indicator',
    'Skeleton': 'Loading skeleton',
    'Badge': 'Status badges',
    'Chip': 'Tag chips',
    'Avatar': 'User avatars',
    'Rating': 'Star rating',
    'Slider': 'Range slider',
    'Toggle': 'Switch toggle',
    'Tabs': 'Tab navigation',
    'Accordion': 'Collapsible panels',
    'Timeline': 'Event timeline',
    'Stepper': 'Multi-step process',
    'Breadcrumb': 'Navigation path',
    'SpeedDial': 'Floating action menu',
    'BottomNavigation': 'Mobile bottom nav',
    'SwipeableViews': 'Swipeable panels',
    'VirtualList': 'Virtualized list',
    'Masonry': 'Masonry grid layout',
    'Parallax': 'Parallax scrolling',
    'AnimatedCounter': 'Number animation',
    'Confetti': 'Celebration effect'
  },
  
  // Mobile-Specific Components
  mobile: {
    'MobileHeader': 'Mobile app header',
    'MobileTabBar': 'Bottom tab navigation',
    'PullToRefresh': 'Pull refresh wrapper',
    'MobileDrawer': 'Slide-out drawer',
    'MobileSearch': 'Mobile search interface',
    'MobileFilters': 'Mobile filter sheet',
    'TouchGestures': 'Gesture handlers',
    'MobileCalendar': 'Mobile calendar view',
    'MobileUpload': 'Mobile file upload',
    'BiometricPrompt': 'Biometric auth prompt'
  },
  
  // SEO Components
  seo: {
    'MetaTags': 'Dynamic meta tags',
    'StructuredData': 'JSON-LD structured data',
    'OpenGraph': 'Open Graph tags',
    'TwitterCard': 'Twitter card meta',
    'Sitemap': 'Dynamic sitemap',
    'RobotsTxt': 'Robots.txt generator',
    'CanonicalUrl': 'Canonical URL handler',
    'HreflangTags': 'Multi-language SEO'
  },
  
  // Analytics Components  
  analytics: {
    'AnalyticsDashboard': 'Main analytics view',
    'MetricCard': 'Metric display card',
    'ChartWrapper': 'Chart container',
    'LineChart': 'Line chart component',
    'BarChart': 'Bar chart component',
    'PieChart': 'Pie chart component',
    'HeatMap': 'Heat map visualization',
    'FunnelChart': 'Conversion funnel',
    'GeoMap': 'Geographic data map',
    'RealtimeMetrics': 'Live data display',
    'DateRangePicker': 'Analytics date range',
    'ExportReport': 'Report export options',
    'CustomQuery': 'Custom analytics query'
  },
  
  // Integration Components
  integrations: {
    'GoogleCalendarSync': 'Google Calendar integration',
    'OutlookSync': 'Outlook integration',
    'AppleCalendarSync': 'Apple Calendar sync',
    'ZapierWebhook': 'Zapier integration',
    'N8NWebhook': 'n8n integration',
    'MakeWebhook': 'Make.com integration',
    'QuickBooksSync': 'QuickBooks integration',
    'ZohoBooksSync': 'Zoho Books sync',
    'LinkedInConnect': 'LinkedIn integration',
    'FacebookPixel': 'Facebook tracking',
    'GoogleAnalytics': 'GA4 integration',
    'Hotjar': 'Hotjar tracking',
    'Intercom': 'Intercom chat widget'
  },
  
  // Developer Portal Components
  developer: {
    'APIDocumentation': 'API docs viewer',
    'APIPlayground': 'Interactive API tester',
    'SDKDownload': 'SDK download section',
    'CodeExamples': 'Code snippet display',
    'APIKeyGenerator': 'API key creation',
    'WebhookManager': 'Webhook configuration',
    'RateLimitDisplay': 'Rate limit info',
    'ChangeLog': 'API changelog',
    'StatusPage': 'API status monitor',
    'SupportForm': 'Developer support'
  }
};

// Component implementation priority
export const COMPONENT_PRIORITY = {
  critical: [
    'AppLayout',
    'WhatsAppLogin',
    'EventCard',
    'EventForm',
    'TicketPurchase',
    'PaymentForm',
    'UniversalSearch'
  ],
  high: [
    'UserProfile',
    'EventGrid',
    'ArtistCard',
    'ChatInterface',
    'AnalyticsDashboard'
  ],
  medium: [
    'CampaignForm',
    'TeamMembers',
    'APIDocumentation',
    'AdminDashboard'
  ],
  low: [
    'TourGuide',
    'Confetti',
    'Parallax'
  ]
};
```

## Best Practices & Code Examples

### API Response Standards

```typescript
// Standard API response format
export interface ApiResponse<T = any> {
  success: boolean;
  data?: T;
  error?: {
    code: string;
    message: string;
    details?: any;
  };
  metadata?: {
    timestamp: string;
    version: string;
    correlationId: string;
  };
  pagination?: {
    page: number;
    limit: number;
    total: number;
    totalPages: number;
  };
}

// Success response helper
export function successResponse<T>(
  data: T,
  metadata?: Partial<ApiResponse['metadata']>
): ApiResponse<T> {
  return {
    success: true,
    data,
    metadata: {
      timestamp: new Date().toISOString(),
      version: '1.0.0',
      correlationId: metadata?.correlationId || crypto.randomUUID(),
      ...metadata
    }
  };
}

// Error response helper
export function errorResponse(
  code: string,
  message: string,
  statusCode: number,
  details?: any
): APIGatewayProxyResultV2 {
  return {
    statusCode,
    headers: {
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      success: false,
      error: {
        code,
        message,
        details
      },
      metadata: {
        timestamp: new Date().toISOString(),
        version: '1.0.0'
      }
    })
  };
}
```

### Multi-Language Content Handling

```typescript
// Multi-language content model
export interface MultiLangContent {
  en: string;
  zh?: string;
  es?: string;
  pt?: string;
  fr?: string;
  ja?: string;
  ar?: string;
  hi?: string;
  ru?: string;
  de?: string;
}

// Content translation service
export class ContentTranslationService {
  async translateContent(
    content: string,
    sourceLang: string,
    targetLangs: string[]
  ): Promise<MultiLangContent> {
    const translations: MultiLangContent = { [sourceLang]: content };
    
    // Batch translate to all target languages
    const translationPromises = targetLangs.map(async (lang) => {
      if (lang === sourceLang) return;
      
      const translated = await this.aiService.translate({
        text: content,
        sourceLang,
        targetLang: lang
      });
      
      translations[lang] = translated;
    });
    
    await Promise.all(translationPromises);
    
    return translations;
  }
  
  getContentByLanguage(
    content: MultiLangContent,
    preferredLang: string,
    fallbackLang = 'en'
  ): string {
    return content[preferredLang] || content[fallbackLang] || content.en;
  }
}
```

### Regional Content Filtering

```typescript
// Regional content filter
export class RegionalContentFilter {
  private regionMappings = {
    'US': ['US', 'CA', 'MX'],
    'GCC': ['AE', 'SA', 'QA', 'KW', 'BH', 'OM'],
    'IN': ['IN', 'BD', 'LK', 'NP'],
    'EU': ['GB', 'FR', 'DE', 'IT', 'ES', 'NL', 'BE', 'AT', 'SE', 'DK', 'FI', 'NO', 'CH', 'IE', 'PT', 'GR', 'CZ', 'PL', 'HU', 'RO'],
    'AU': ['AU', 'NZ']
  };
  
  filterByRegion<T extends { location: { country: string } }>(
    items: T[],
    userRegion: string
  ): T[] {
    const allowedCountries = this.regionMappings[userRegion] || [];
    
    return items.filter(item => 
      allowedCountries.includes(item.location.country)
    );
  }
  
  prioritizeByRegion<T extends { location: { country: string } }>(
    items: T[],
    userRegion: string
  ): T[] {
    const regional = this.filterByRegion(items, userRegion);
    const others = items.filter(item => !regional.includes(item));
    
    return [...regional, ...others];
  }
}
```

### WhatsApp Template Management

```typescript
// WhatsApp template registry
export const WHATSAPP_TEMPLATES = {
  OTP_VERIFICATION: {
    name: 'otp_verification',
    languages: ['en', 'es', 'pt', 'hi', 'ar'],
    parameters: ['otp', 'validity']
  },
  EVENT_INVITATION: {
    name: 'event_invitation',
    languages: ['en', 'es', 'pt', 'hi', 'ar'],
    parameters: ['eventName', 'eventDate', 'eventUrl']
  },
  TICKET_CONFIRMATION: {
    name: 'ticket_confirmation',
    languages: ['en', 'es', 'pt', 'hi', 'ar'],
    parameters: ['eventName', 'ticketCount', 'qrCode']
  },
  PAYMENT_RECEIPT: {
    name: 'payment_receipt',
    languages: ['en', 'es', 'pt', 'hi', 'ar'],
    parameters: ['amount', 'currency', 'transactionId']
  },
  ARTIST_BOOKING: {
    name: 'artist_booking',
    languages: ['en', 'es', 'pt', 'hi', 'ar'],
    parameters: ['artistName', 'eventDate', 'venue']
  }
};

// Template sender
export class WhatsAppTemplateSender {
  async sendTemplate(
    phoneNumber: string,
    templateKey: keyof typeof WHATSAPP_TEMPLATES,
    parameters: Record<string, string>,
    language = 'en'
  ): Promise<void> {
    const template = WHATSAPP_TEMPLATES[templateKey];
    
    // Validate parameters
    const missingParams = template.parameters.filter(
      param => !parameters[param]
    );
    
    if (missingParams.length > 0) {
      throw new Error(`Missing parameters: ${missingParams.join(', ')}`);
    }
    
    // Send via WhatsApp API
    await this.whatsappClient.sendTemplate(
      phoneNumber,
      template.name,
      language,
      template.parameters.map(param => ({
        type: 'text',
        text: parameters[param]
      }))
    );
  }
}
```

### Performance Optimization Patterns

```typescript
// 1. Request caching
export class CachedAPIHandler {
  private cache = new Map<string, { data: any; expiry: number }>();
  
  async handleRequest(
    key: string,
    ttl: number,
    fetcher: () => Promise<any>
  ): Promise<any> {
    const cached = this.cache.get(key);
    
    if (cached && cached.expiry > Date.now()) {
      return cached.data;
    }
    
    const data = await fetcher();
    
    this.cache.set(key, {
      data,
      expiry: Date.now() + ttl
    });
    
    return data;
  }
}

// 2. Batch processing
export class BatchProcessor {
  private queue: Array<{ id: string; resolver: Function }> = [];
  private timer: NodeJS.Timeout | null = null;
  
  async process<T>(id: string): Promise<T> {
    return new Promise((resolve) => {
      this.queue.push({ id, resolver: resolve });
      
      if (!this.timer) {
        this.timer = setTimeout(() => this.flush(), 10);
      }
    });
  }
  
  private async flush(): Promise<void> {
    const batch = [...this.queue];
    this.queue = [];
    this.timer = null;
    
    if (batch.length === 0) return;
    
    const ids = batch.map(item => item.id);
    const results = await this.batchFetch(ids);
    
    batch.forEach(({ id, resolver }) => {
      resolver(results[id]);
    });
  }
  
  private async batchFetch(ids: string[]): Promise<Record<string, any>> {
    // Implement batch fetching logic
    return {};
  }
}

// 3. Connection pooling
export class DatabaseConnectionPool {
  private pool: DataSource[] = [];
  private activeConnections = new Map<string, DataSource>();
  
  async getConnection(tenantId: string): Promise<DataSource> {
    if (this.activeConnections.has(tenantId)) {
      return this.activeConnections.get(tenantId)!;
    }
    
    let connection = this.pool.pop();
    
    if (!connection) {
      connection = await this.createConnection();
    }
    
    this.activeConnections.set(tenantId, connection);
    
    return connection;
  }
  
  async releaseConnection(tenantId: string): Promise<void> {
    const connection = this.activeConnections.get(tenantId);
    
    if (connection && this.pool.length < 5) {
      this.pool.push(connection);
    } else if (connection) {
      await connection.destroy();
    }
    
    this.activeConnections.delete(tenantId);
  }
  
  private async createConnection(): Promise<DataSource> {
    // Create new connection
    return new DataSource({
      // Configuration
    });
  }
}
```

## Conclusion

This comprehensive guide provides the complete blueprint for developing the EventfulHQ platform's microservices architecture. By following these guidelines, developers and AI assistants can ensure consistent, scalable, and maintainable code across all services.

### Key Principles to Remember

1. **WhatsApp-First**: Every feature should consider WhatsApp integration
2. **AI-Powered**: Leverage AI for intelligent automation and recommendations
3. **Multi-Domain**: Support regional domains with proper routing
4. **Multi-Language**: Full internationalization with 10 languages
5. **Serverless**: Optimize for AWS Lambda's constraints
6. **Event-Driven**: Use asynchronous patterns for scalability
7. **API-First**: Design APIs before implementation
8. **Documentation**: Auto-generate and maintain Swagger docs
9. **Performance**: Meet strict SLA requirements
10. **Security**: Implement defense in depth

### Next Steps

1. Use this document as the single source of truth
2. Keep it updated as the platform evolves
3. Ensure all code follows these patterns
4. Regular architecture reviews
5. Performance monitoring and optimization

Remember: **Every line of code should align with these architectural principles and be ready for production deployment.**