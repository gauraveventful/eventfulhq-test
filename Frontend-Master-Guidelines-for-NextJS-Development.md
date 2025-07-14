# Master Frontend Architecture Guidelines for EventfulHQ Platform
**Enterprise-Grade Next.js 15 SaaS Implementation with Microservices**  
**Version 4.0 - Complete Edition with SuperAdmin, Developer Portal & SEO Features**  
**Last Updated: January 2025**

## Table of Contents

1. [Executive Summary](#executive-summary)
2. [MatDash Theme Context & Capabilities](#matdash-theme-context--capabilities)
3. [Core Architecture Principles](#core-architecture-principles)
4. [Technology Stack & Dependencies](#technology-stack--dependencies)
5. [Project Structure & Nomenclature](#project-structure--nomenclature)
6. [Multi-Language Implementation](#multi-language-implementation)
7. [Multi-Domain Regional Routing](#multi-domain-regional-routing)
8. [SuperAdmin Access System](#superadmin-access-system)
9. [Developer Portal & API Documentation](#developer-portal--api-documentation)
10. [Landing Pages & Microsites](#landing-pages--microsites)
11. [Bot Management & SEO Strategy](#bot-management--seo-strategy)
12. [State Management & Data Flow](#state-management--data-flow)
13. [Form Handling with Yup & Zod](#form-handling-with-yup--zod)
14. [Storybook Integration](#storybook-integration)
15. [API Integration & Code Generation](#api-integration--code-generation)
16. [Authentication & Security Patterns](#authentication--security-patterns)
17. [Performance Optimization & SLA Compliance](#performance-optimization--sla-compliance)
18. [Testing Strategy](#testing-strategy)
19. [Vercel Deployment & Edge Functions](#vercel-deployment--edge-functions)
20. [Component Enhancement Patterns](#component-enhancement-patterns)
21. [Lambda Function Requirements](#lambda-function-requirements)
22. [API Gateway Configuration](#api-gateway-configuration)
23. [Production Readiness Checklist](#production-readiness-checklist)

## Executive Summary

This document serves as the definitive architectural blueprint and API development prompt for the EventfulHQ platform frontend. It provides comprehensive context about the MatDash theme capabilities, limitations, and enhancement opportunities, enabling precise Lambda function and API Gateway development that seamlessly integrates with the frontend architecture.

### Document Purpose

- **Primary**: Serve as a complete prompt for AI-assisted API and Lambda function development
- **Secondary**: Establish frontend architecture patterns and conventions including SuperAdmin, Developer Portal, and SEO features
- **Tertiary**: Document MatDash theme capabilities and planned enhancements

### Key Architectural Decisions

- **Base Theme**: MatDash Next.js template (modified)
- **UI Library**: ShadCN/UI (migrating from Flowbite-react)
- **Framework**: Next.js 15.0.3 with App Router and React 19
- **Validation**: Yup (primary) and Zod (secondary) for form validation
- **Documentation**: Storybook 8.0 for component library
- **State Management**: Zustand (global) + React Query (server state)
- **Deployment**: Vercel with Edge Functions
- **Backend**: AWS Lambda + API Gateway with NestJS
- **Authentication**: WhatsApp OTP (primary) + JWT with Role-Based Access Control
- **Admin System**: Multi-tier admin system with 11 admin types
- **SEO Strategy**: Dynamic landing pages with bot management

## MatDash Theme Context & Capabilities

### Current MatDash Implementation

The MatDash theme provides a solid foundation with the following capabilities:

#### Strengths

**Pre-built Dashboard Layouts**
- Vertical sidebar (default)
- Horizontal navigation
- Mini sidebar variant
- RTL support structure

**Component Library (Flowbite-based)**
- 50+ pre-styled components
- Consistent design language
- Responsive by default
- Dark mode support

**Built-in Features**
- Authentication pages (login, register, forgot password)
- Dashboard analytics widgets
- Data visualization (charts)
- Email and chat applications
- Calendar implementation
- Kanban board

**Technical Foundation**
- Next.js 15 App Router structure
- TypeScript configuration
- Tailwind CSS setup
- Basic API structure

#### Limitations Requiring Enhancement

**Component Customization**
- Flowbite components have limited customization
- No component composition patterns
- Missing advanced form components
- No built-in accessibility features beyond basics

**State Management**
- No global state solution
- Basic React Context usage
- No server state caching strategy
- Missing optimistic updates

**Internationalization**
- No i18n support out of the box
- Hardcoded English strings
- No RTL implementation for Arabic
- Missing locale detection

**API Integration**
- No standardized API client
- Missing error handling patterns
- No type generation from OpenAPI
- Basic fetch usage without interceptors

**Testing Infrastructure**
- No test setup included
- Missing component documentation
- No Storybook integration
- No E2E test configuration

**Performance Optimization**
- No code splitting strategy
- Missing image optimization
- No caching implementation
- Basic bundle configuration

#### Enhancement Opportunities

**Component System Upgrade**
```typescript
// FROM: MatDash Flowbite component
import { Button } from 'flowbite-react';

// TO: Enhanced ShadCN component with variants
import { Button } from '@/components/ui/button';
// Provides: Better TypeScript, accessibility, customization
```

**API Client Architecture**
```typescript
// FROM: Basic fetch in MatDash
const data = await fetch('/api/events').then(r => r.json());

// TO: Type-safe API client with interceptors
const { data } = await eventService.findAll({ 
  status: 'published',
  limit: 20 
});
// Provides: Type safety, error handling, caching, retry logic
```

**Form Handling Enhancement**
```typescript
// FROM: Basic form in MatDash
<form onSubmit={handleSubmit}>
  <input name="email" />
</form>

// TO: React Hook Form with Yup validation
<Form {...form}>
  <FormField
    name="email"
    rules={emailValidation}
    render={({ field }) => (
      <FormInput {...field} />
    )}
  />
</Form>
// Provides: Validation, error handling, type safety
```

## Core Architecture Principles

### 1. WhatsApp-First Design
Every feature must integrate WhatsApp as the primary channel:
- Authentication: OTP via WhatsApp Business API
- Notifications: Real-time updates through WhatsApp
- Sharing: Deep links for WhatsApp sharing
- Support: WhatsApp chat integration

### 2. AI-First Features
Leverage AI throughout the platform:
- Smart Recommendations: ML-based event suggestions
- Content Generation: AI-powered descriptions
- Image Enhancement: Automatic image optimization
- Chatbot Support: NLP-based customer service

### 3. Microservices Architecture
- 30+ Domain Services: Each handling specific business logic
- Event-Driven: AWS EventBridge for service communication
- API Gateway: Centralized routing and rate limiting
- Service Mesh: AWS App Mesh for observability
- **All Microservices are AppFeatures**: Each service represents a controllable feature

### 4. Performance SLAs
- Page Load: LCP < 2.5s, FID < 100ms, CLS < 0.1
- API Response: p50 < 200ms, p95 < 500ms, p99 < 1s
- Availability: 99.9% uptime SLA
- Error Rate: < 0.5% error threshold

### 5. SEO-First Architecture
- Dynamic landing pages for cities and countries
- Microsites for users, teams, and events
- Bot management for optimal crawling
- Structured data implementation

## Technology Stack & Dependencies

### Core Framework
```json
{
  "next": "15.0.3",
  "react": "19.0.0-rc-66855b96-20241106",
  "react-dom": "19.0.0-rc-66855b96-20241106",
  "typescript": "^5.3.3",
  "node": ">=18.17.0"
}
```

### UI & Styling
```json
{
  "@shadcn/ui": "latest",
  "tailwindcss": "^3.4.1",
  "@radix-ui/react-accordion": "^1.1.2",
  "@radix-ui/react-dialog": "^1.0.5",
  "@radix-ui/react-dropdown-menu": "^2.0.6",
  "@radix-ui/react-select": "^2.0.0",
  "class-variance-authority": "^0.7.0",
  "clsx": "^2.1.0",
  "tailwind-merge": "^2.2.0",
  "lucide-react": "^0.309.0",
  "@tabler/icons-react": "^3.19.0"
}
```

### Form Handling & Validation
```json
{
  "react-hook-form": "^7.54.0",
  "yup": "^1.3.3",
  "@hookform/resolvers": "^3.3.4",
  "zod": "^3.22.4"
}
```

### State & Data Management
```json
{
  "@tanstack/react-query": "^5.17.0",
  "@tanstack/react-query-devtools": "^5.17.0",
  "zustand": "^4.5.0",
  "immer": "^10.0.3",
  "axios": "^1.6.5",
  "swr": "^2.2.5"
}
```

### Admin & Developer Portal
```json
{
  "@monaco-editor/react": "^4.6.0",
  "swagger-ui-react": "^5.10.0",
  "@apidevtools/swagger-parser": "^10.1.0",
  "react-json-view": "^1.21.3",
  "@tanstack/react-table": "^8.11.0"
}
```

### SEO & Analytics
```json
{
  "next-seo": "^6.4.0",
  "next-sitemap": "^4.2.0",
  "@next/third-parties": "^14.0.4",
  "react-adsense": "^0.1.0",
  "schema-dts": "^1.1.2"
}
```

### Documentation & Testing
```json
{
  "storybook": "^8.0.0",
  "@storybook/addon-essentials": "^8.0.0",
  "@storybook/addon-interactions": "^8.0.0",
  "@storybook/addon-links": "^8.0.0",
  "@storybook/blocks": "^8.0.0",
  "@storybook/nextjs": "^8.0.0",
  "@storybook/react": "^8.0.0",
  "@storybook/test": "^8.0.0",
  "@testing-library/react": "^14.1.2",
  "@testing-library/jest-dom": "^6.2.0",
  "@testing-library/user-event": "^14.5.2",
  "vitest": "^1.2.0",
  "playwright": "^1.41.0",
  "msw": "^2.1.0"
}
```

### Internationalization
```json
{
  "next-intl": "^3.5.0",
  "@formatjs/intl": "^2.9.0",
  "accept-language-parser": "^1.5.0",
  "negotiator": "^0.6.3"
}
```

### Development Tools
```json
{
  "@openapitools/openapi-generator-cli": "^2.7.0",
  "orval": "^6.23.0",
  "prettier": "^3.2.4",
  "eslint": "^8.56.0",
  "eslint-config-next": "15.0.3",
  "husky": "^8.0.3",
  "lint-staged": "^15.2.0"
}
```

## Project Structure & Nomenclature

### Complete Directory Structure
```
eventfulhq-frontend/
├── .storybook/                      # Storybook configuration
│   ├── main.ts
│   ├── preview.tsx
│   └── preview-head.html
├── src/
│   ├── app/                         # Next.js 15 App Router
│   │   ├── [locale]/               # Internationalization root
│   │   │   ├── (auth)/             # Public auth routes
│   │   │   │   ├── login/
│   │   │   │   │   ├── page.tsx
│   │   │   │   │   ├── layout.tsx
│   │   │   │   │   └── whatsapp-otp-modal.tsx
│   │   │   │   ├── register/
│   │   │   │   │   └── page.tsx
│   │   │   │   └── forgot-password/
│   │   │   │       └── page.tsx
│   │   │   ├── (dashboard)/        # Protected dashboard routes
│   │   │   │   ├── layout.tsx      # Dashboard layout with sidebar
│   │   │   │   ├── page.tsx        # Dashboard home
│   │   │   │   ├── events/
│   │   │   │   │   ├── page.tsx
│   │   │   │   │   ├── create/
│   │   │   │   │   ├── [eventId]/
│   │   │   │   │   │   ├── page.tsx
│   │   │   │   │   │   ├── edit/
│   │   │   │   │   │   ├── analytics/
│   │   │   │   │   │   └── attendees/
│   │   │   │   │   └── loading.tsx
│   │   │   │   ├── rfp/            # RFP Marketplace
│   │   │   │   ├── artists/
│   │   │   │   ├── venues/
│   │   │   │   ├── analytics/
│   │   │   │   └── settings/
│   │   │   ├── (superadmin)/       # SuperAdmin routes
│   │   │   │   ├── layout.tsx       # SuperAdmin layout
│   │   │   │   ├── super-admin/
│   │   │   │   │   ├── page.tsx     # SuperAdmin dashboard
│   │   │   │   │   ├── admins/      # Admin management
│   │   │   │   │   │   ├── page.tsx
│   │   │   │   │   │   ├── create/
│   │   │   │   │   │   └── [adminId]/
│   │   │   │   │   ├── features/    # Feature control
│   │   │   │   │   │   ├── page.tsx
│   │   │   │   │   │   └── [featureId]/
│   │   │   │   │   ├── geography/   # Geography control
│   │   │   │   │   │   ├── page.tsx
│   │   │   │   │   │   └── [regionId]/
│   │   │   │   │   ├── permissions/ # Permission matrix
│   │   │   │   │   ├── config/      # App configuration
│   │   │   │   │   ├── monitoring/  # System monitoring
│   │   │   │   │   └── audit-logs/  # Audit trail
│   │   │   ├── (developers)/        # Developer portal
│   │   │   │   ├── layout.tsx
│   │   │   │   ├── developers/
│   │   │   │   │   ├── page.tsx     # Developer home
│   │   │   │   │   ├── docs/        # API documentation
│   │   │   │   │   │   ├── page.tsx
│   │   │   │   │   │   ├── [service]/
│   │   │   │   │   │   └── playground/
│   │   │   │   │   ├── api-keys/    # API key management
│   │   │   │   │   ├── webhooks/    # Webhook config
│   │   │   │   │   ├── usage/       # Usage analytics
│   │   │   │   │   └── billing/     # API billing
│   │   │   ├── (landing)/           # Landing pages
│   │   │   │   ├── [country]/       # Country pages
│   │   │   │   │   ├── page.tsx
│   │   │   │   │   └── [city]/      # City pages
│   │   │   │   │       └── page.tsx
│   │   │   │   ├── u/               # User microsites
│   │   │   │   │   └── [username]/
│   │   │   │   │       └── page.tsx
│   │   │   │   ├── t/               # Team microsites
│   │   │   │   │   └── [teamSlug]/
│   │   │   │   │       └── page.tsx
│   │   │   │   └── e/               # Event microsites
│   │   │   │       └── [eventSlug]/
│   │   │   │           └── page.tsx
│   │   │   └── layout.tsx
│   │   ├── api/                     # API Routes & BFF
│   │   │   ├── bff/                # Backend for Frontend
│   │   │   │   ├── auth/
│   │   │   │   ├── events/
│   │   │   │   ├── analytics/
│   │   │   │   └── admin/          # Admin BFF endpoints
│   │   │   ├── webhooks/
│   │   │   │   ├── whatsapp/
│   │   │   │   ├── stripe/
│   │   │   │   └── eventbridge/
│   │   │   ├── public/             # Public API endpoints
│   │   │   │   ├── v1/             # Versioned public API
│   │   │   │   └── docs/           # API docs endpoint
│   │   │   ├── seo/                # SEO endpoints
│   │   │   │   ├── sitemap/
│   │   │   │   ├── robots/
│   │   │   │   └── schema/
│   │   │   ├── health/
│   │   │   └── monitoring/
│   │   ├── layout.tsx              # Root layout
│   │   ├── error.tsx               # Error boundary
│   │   ├── global-error.tsx        # Global error handler
│   │   ├── not-found.tsx           # 404 page
│   │   └── robots.txt              # Dynamic robots.txt
│   ├── components/
│   │   ├── ui/                     # ShadCN UI components
│   │   │   ├── accordion.tsx
│   │   │   ├── alert-dialog.tsx
│   │   │   ├── button.tsx
│   │   │   ├── card.tsx
│   │   │   ├── dialog.tsx
│   │   │   ├── form.tsx
│   │   │   ├── input.tsx
│   │   │   ├── select.tsx
│   │   │   └── [30+ components]
│   │   ├── features/               # Feature-specific components
│   │   │   ├── events/
│   │   │   ├── whatsapp/
│   │   │   ├── rfp/
│   │   │   ├── analytics/
│   │   │   ├── admin/              # Admin components
│   │   │   │   ├── permission-matrix/
│   │   │   │   ├── admin-table/
│   │   │   │   └── feature-toggles/
│   │   │   ├── developers/         # Developer portal components
│   │   │   │   ├── api-explorer/
│   │   │   │   ├── code-samples/
│   │   │   │   └── api-key-manager/
│   │   │   ├── landing/            # Landing page components
│   │   │   │   ├── city-hero/
│   │   │   │   ├── trending-grid/
│   │   │   │   └── adsense-banner/
│   │   │   └── seo/                # SEO components
│   │   │       ├── structured-data/
│   │   │       ├── meta-tags/
│   │   │       └── bot-detector/
│   │   ├── layouts/                # Layout components
│   │   │   ├── dashboard-layout/
│   │   │   ├── auth-layout/
│   │   │   ├── landing-layout/
│   │   │   ├── superadmin-layout/
│   │   │   ├── developer-layout/
│   │   │   └── microsite-layout/
│   │   └── shared/                 # Shared components
│   ├── features/                    # Feature modules (DDD)
│   │   ├── events/
│   │   │   ├── components/
│   │   │   ├── hooks/
│   │   │   ├── services/
│   │   │   ├── types/
│   │   │   ├── utils/
│   │   │   └── validations/
│   │   ├── admin/                   # Admin feature module
│   │   │   ├── components/
│   │   │   ├── hooks/
│   │   │   ├── services/
│   │   │   ├── types/
│   │   │   └── constants/
│   │   ├── developers/              # Developer feature module
│   │   ├── landing/                 # Landing pages module
│   │   ├── seo/                     # SEO module
│   │   └── [other features]/
│   ├── hooks/                       # Global hooks
│   │   ├── use-admin-permissions.ts
│   │   ├── use-bot-detection.ts
│   │   └── use-adsense.ts
│   ├── lib/                         # Core libraries
│   │   ├── api/
│   │   │   ├── client.ts
│   │   │   ├── interceptors.ts
│   │   │   └── error-handler.ts
│   │   ├── auth/
│   │   │   ├── rbac.ts             # Role-based access control
│   │   │   └── permissions.ts
│   │   ├── i18n/
│   │   ├── seo/
│   │   │   ├── bot-detector.ts
│   │   │   ├── structured-data.ts
│   │   │   └── sitemap-generator.ts
│   │   └── utils/
│   ├── services/                    # API service layer
│   │   ├── generated/              # OpenAPI generated
│   │   ├── admin-service.ts
│   │   ├── developer-service.ts
│   │   ├── base-service.ts
│   │   └── service-registry.ts
│   ├── stores/                      # Zustand stores
│   │   ├── admin-store.ts
│   │   └── developer-store.ts
│   ├── types/                       # TypeScript types
│   │   ├── admin.types.ts
│   │   ├── developer.types.ts
│   │   └── permissions.types.ts
│   ├── validations/                 # Yup schemas
│   │   ├── event.schema.ts
│   │   ├── user.schema.ts
│   │   ├── admin.schema.ts
│   │   └── common.schema.ts
│   ├── locales/                     # Translation files
│   └── styles/                      # Global styles
├── public/                          # Static assets
│   ├── ads.txt                     # AdSense verification
│   └── api-docs/                   # Static API docs
├── stories/                         # Storybook stories
│   ├── components/
│   ├── features/
│   └── pages/
├── tests/                           # Test files
│   ├── unit/
│   ├── integration/
│   └── e2e/
├── .env.example
├── .eslintrc.json
├── .prettierrc
├── next.config.mjs
├── tailwind.config.ts
├── tsconfig.json
├── vitest.config.ts
└── vercel.json
```

### Naming Conventions

#### File Naming Standards
```typescript
// ✅ CORRECT File Names
page.tsx                    // Next.js pages
layout.tsx                  // Layout components
loading.tsx                 // Loading states
error.tsx                   // Error boundaries
event-card.tsx             // Component files (kebab-case)
event-card.stories.tsx     // Storybook stories
event-card.test.tsx        // Test files
use-event-data.ts          // Hooks (use- prefix)
event.service.ts           // Service files
event.schema.ts            // Yup validation schemas
event.types.ts             // Type definitions
admin-permissions.constants.ts // Constants files

// ❌ INCORRECT
EventCard.tsx              // Don't use PascalCase for files
eventCard.tsx              // Don't use camelCase
event_card.tsx             // Don't use snake_case
```

#### Component Naming
```typescript
// ✅ CORRECT
export function EventCard() {}              // PascalCase for components
export function useEventData() {}           // camelCase for hooks
export const EventContext = createContext() // Context naming
export const ADMIN_ROLES = {} as const      // UPPER_SNAKE_CASE for constants

interface EventCardProps {                  // Props interface
  event: Event;
  onSelect?: (event: Event) => void;
}

// ❌ INCORRECT
export function eventCard() {}              // Don't use camelCase
export function UseEventData() {}           // Don't capitalize 'use'
```

## Multi-Language Implementation

### Supported Languages
1. English (en) - Default
2. Mandarin Chinese (zh) - 中文
3. Spanish (es) - Español
4. Portuguese (pt) - Português
5. French (fr) - Français
6. Japanese (ja) - 日本語
7. Standard Arabic (ar) - العربية (RTL)
8. Hindi (hi) - हिन्दी
9. Russian (ru) - Русский

### IP-Based Language Detection with User Preference
```typescript
// middleware.ts
import { NextRequest, NextResponse } from 'next/server';
import { match } from '@formatjs/intl-localematcher';
import Negotiator from 'negotiator';

const locales = ['en', 'zh', 'es', 'pt', 'fr', 'ja', 'ar', 'hi', 'ru'];
const defaultLocale = 'en';

// IP to country mapping
const countryToLanguage: Record<string, string> = {
  // China, Hong Kong, Taiwan, Singapore
  CN: 'zh', HK: 'zh', TW: 'zh', SG: 'zh',
  // Spanish speaking countries
  ES: 'es', MX: 'es', AR: 'es', CO: 'es', CL: 'es', PE: 'es',
  // Portuguese speaking countries
  BR: 'pt', PT: 'pt', AO: 'pt', MZ: 'pt',
  // French speaking countries
  FR: 'fr', BE: 'fr', CA: 'fr', CH: 'fr', SN: 'fr',
  // Japan
  JP: 'ja',
  // Arabic speaking countries
  SA: 'ar', AE: 'ar', EG: 'ar', KW: 'ar', QA: 'ar', MA: 'ar',
  // India (Hindi)
  IN: 'hi',
  // Russian speaking countries
  RU: 'ru', BY: 'ru', KZ: 'ru', UA: 'ru',
};

export async function middleware(request: NextRequest) {
  const pathname = request.nextUrl.pathname;
  
  // Skip if locale already in path
  const pathnameHasLocale = locales.some(
    (locale) => pathname.startsWith(`/${locale}/`) || pathname === `/${locale}`
  );

  if (pathnameHasLocale) return;

  // 1. Check saved user preference
  const userPreferredLocale = request.cookies.get('preferred-locale')?.value;
  if (userPreferredLocale && locales.includes(userPreferredLocale)) {
    return NextResponse.redirect(
      new URL(`/${userPreferredLocale}${pathname}`, request.url)
    );
  }

  // 2. Check Accept-Language header
  const headers = { 'accept-language': request.headers.get('accept-language') || '' };
  const languages = new Negotiator({ headers }).languages();
  const headerLocale = match(languages, locales, defaultLocale);

  // 3. IP-based detection using Vercel's geo data
  const country = request.geo?.country || 
                  request.headers.get('x-vercel-ip-country') || 
                  '';
  const ipBasedLocale = countryToLanguage[country] || null;

  // Determine final locale (priority: saved > header > IP > default)
  const locale = headerLocale || ipBasedLocale || defaultLocale;

  // Redirect with locale
  const response = NextResponse.redirect(
    new URL(`/${locale}${pathname}`, request.url)
  );

  // Save preference for future visits
  response.cookies.set('preferred-locale', locale, {
    maxAge: 60 * 60 * 24 * 365, // 1 year
    sameSite: 'lax',
    secure: process.env.NODE_ENV === 'production',
  });

  return response;
}

export const config = {
  matcher: [
    '/((?!api|_next/static|_next/image|favicon.ico|.*\\..*).*)',
  ],
};
```

### Event Service Lambda Example
```typescript
// lambdas/event-service/create-event.ts
import { APIGatewayProxyEvent, APIGatewayProxyResult } from 'aws-lambda';
import { DynamoDBClient } from '@aws-sdk/client-dynamodb';
import { EventBridgeClient, PutEventsCommand } from '@aws-sdk/client-eventbridge';
import { S3Client, PutObjectCommand } from '@aws-sdk/client-s3';
import { v4 as uuidv4 } from 'uuid';
import * as yup from 'yup';
import { authenticate, authorize } from '/opt/nodejs/auth';
import { logger } from '/opt/nodejs/logging';
import { metrics } from '/opt/nodejs/monitoring';

const dynamodb = new DynamoDBClient({});
const eventbridge = new EventBridgeClient({});
const s3 = new S3Client({});

// Validation schema
const createEventSchema = yup.object({
  title: yup.string().required().min(5).max(100),
  description: yup.string().required().min(20).max(5000),
  date: yup.date().required().min(new Date()),
  venue: yup.object({
    name: yup.string().required(),
    address: yup.string().required(),
    city: yup.string().required(),
    country: yup.string().required(),
    coordinates: yup.object({
      lat: yup.number().required().min(-90).max(90),
      lng: yup.number().required().min(-180).max(180),
    }).required(),
  }).required(),
  capacity: yup.number().required().positive().integer(),
  price: yup.number().required().min(0),
  currency: yup.string().required().oneOf(['USD', 'EUR', 'GBP', 'INR', 'AED', 'SAR']),
  images: yup.array().of(yup.string()).min(1).max(10),
});

export const handler = async (
  event: APIGatewayProxyEvent
): Promise<APIGatewayProxyResult> => {
  const requestId = event.requestContext.requestId;
  const startTime = Date.now();
  
  logger.info('Creating event', { requestId, path: event.path });
  
  try {
    // 1. Authenticate user
    const user = await authenticate(event.headers.authorization);
    if (!user) {
      return {
        statusCode: 401,
        body: JSON.stringify({
          error: {
            code: 'UNAUTHORIZED',
            message: 'Authentication required',
            requestId,
          },
        }),
      };
    }
    
    // 2. Parse and validate request
    const body = JSON.parse(event.body || '{}');
    const validatedData = await createEventSchema.validate(body, { 
      abortEarly: false,
      stripUnknown: true,
    });
    
    // 3. Authorize action
    const canCreate = await authorize(user, 'events', 'create', {
      venue: validatedData.venue,
    });
    
    if (!canCreate) {
      return {
        statusCode: 403,
        body: JSON.stringify({
          error: {
            code: 'FORBIDDEN',
            message: 'You do not have permission to create events in this location',
            requestId,
          },
        }),
      };
    }
    
    // 4. Process images
    const processedImages = await processImages(validatedData.images);
    
    // 5. Create event
    const eventId = uuidv4();
    const eventData = {
      id: eventId,
      ...validatedData,
      images: processedImages,
      organizerId: user.id,
      organizerName: user.name,
      status: 'draft',
      createdAt: new Date().toISOString(),
      updatedAt: new Date().toISOString(),
      slug: generateSlug(validatedData.title),
    };
    
    // 6. Save to database
    await dynamodb.putItem({
      TableName: process.env.EVENTS_TABLE,
      Item: marshall(eventData),
      ConditionExpression: 'attribute_not_exists(id)',
    });
    
    // 7. Emit event
    await eventbridge.send(new PutEventsCommand({
      Entries: [{
        Source: 'event-service',
        DetailType: 'EventCreated',
        Detail: JSON.stringify({
          eventId,
          organizerId: user.id,
          title: validatedData.title,
          date: validatedData.date,
          venue: validatedData.venue,
        }),
        EventBusName: process.env.EVENT_BUS_NAME,
      }],
    }));
    
    // 8. Record metrics
    const duration = Date.now() - startTime;
    metrics.recordMetric('EventCreated', 1);
    metrics.recordMetric('EventCreationDuration', duration);
    
    logger.info('Event created successfully', { 
      requestId, 
      eventId, 
      duration,
    });
    
    // 9. Return response
    return {
      statusCode: 201,
      headers: {
        'Content-Type': 'application/json',
        'X-Request-ID': requestId,
        'Location': `/events/${eventId}`,
      },
      body: JSON.stringify({
        data: eventData,
        meta: {
          requestId,
          timestamp: new Date().toISOString(),
          version: '1.0',
        },
      }),
    };
    
  } catch (error) {
    logger.error('Failed to create event', { 
      requestId, 
      error: error.message,
      stack: error.stack,
    });
    
    // Handle validation errors
    if (error.name === 'ValidationError') {
      return {
        statusCode: 400,
        body: JSON.stringify({
          error: {
            code: 'VALIDATION_ERROR',
            message: 'Invalid request data',
            details: error.errors,
            requestId,
          },
        }),
      };
    }
    
    // Handle duplicate events
    if (error.code === 'ConditionalCheckFailedException') {
      return {
        statusCode: 409,
        body: JSON.stringify({
          error: {
            code: 'DUPLICATE_EVENT',
            message: 'An event with this ID already exists',
            requestId,
          },
        }),
      };
    }
    
    // Generic error
    return {
      statusCode: 500,
      body: JSON.stringify({
        error: {
          code: 'INTERNAL_ERROR',
          message: 'An error occurred while creating the event',
          requestId,
        },
      }),
    };
  }
};

// Helper functions
async function processImages(imageUrls: string[]): Promise<ProcessedImage[]> {
  const processedImages = [];
  
  for (const url of imageUrls) {
    // Download image
    const imageBuffer = await downloadImage(url);
    
    // Process variants
    const variants = await Promise.all([
      resizeImage(imageBuffer, { width: 1920, height: 1080, format: 'webp' }),
      resizeImage(imageBuffer, { width: 800, height: 600, format: 'webp' }),
      resizeImage(imageBuffer, { width: 400, height: 300, format: 'webp' }),
      resizeImage(imageBuffer, { width: 150, height: 150, format: 'webp', fit: 'cover' }),
    ]);
    
    // Upload to S3
    const uploadPromises = variants.map((variant, index) => {
      const key = `events/${uuidv4()}/image-${index}.webp`;
      return s3.send(new PutObjectCommand({
        Bucket: process.env.IMAGES_BUCKET,
        Key: key,
        Body: variant.buffer,
        ContentType: 'image/webp',
        CacheControl: 'public, max-age=31536000',
      }));
    });
    
    await Promise.all(uploadPromises);
    
    processedImages.push({
      original: url,
      large: `${process.env.CDN_URL}/${variants[0].key}`,
      medium: `${process.env.CDN_URL}/${variants[1].key}`,
      small: `${process.env.CDN_URL}/${variants[2].key}`,
      thumbnail: `${process.env.CDN_URL}/${variants[3].key}`,
    });
  }
  
  return processedImages;
}

function generateSlug(title: string): string {
  return title
    .toLowerCase()
    .replace(/[^a-z0-9]+/g, '-')
    .replace(/^-+|-+$/g, '')
    .substring(0, 50);
}
```

### WhatsApp OTP Lambda
```typescript
// lambdas/auth-service/whatsapp-otp.ts
import { APIGatewayProxyEvent, APIGatewayProxyResult } from 'aws-lambda';
import { DynamoDBClient } from '@aws-sdk/client-dynamodb';
import { SecretsManagerClient, GetSecretValueCommand } from '@aws-sdk/client-secrets-manager';
import { createHash, randomInt } from 'crypto';
import { WhatsAppBusinessAPI } from 'whatsapp-business-api';
import { logger } from '/opt/nodejs/logging';
import { rateLimiter } from '/opt/nodejs/rate-limiter';

const dynamodb = new DynamoDBClient({});
const secretsManager = new SecretsManagerClient({});

let whatsappClient: WhatsAppBusinessAPI;

export const handler = async (
  event: APIGatewayProxyEvent
): Promise<APIGatewayProxyResult> => {
  const requestId = event.requestContext.requestId;
  const { phoneNumber } = JSON.parse(event.body || '{}');
  
  try {
    // 1. Validate phone number
    if (!isValidPhoneNumber(phoneNumber)) {
      return {
        statusCode: 400,
        body: JSON.stringify({
          error: {
            code: 'INVALID_PHONE_NUMBER',
            message: 'Please provide a valid phone number with country code',
            requestId,
          },
        }),
      };
    }
    
    // 2. Rate limiting
    const rateLimitKey = `otp:${phoneNumber}`;
    const isAllowed = await rateLimiter.check(rateLimitKey, {
      max: 3,
      window: 300, // 5 minutes
    });
    
    if (!isAllowed) {
      return {
        statusCode: 429,
        body: JSON.stringify({
          error: {
            code: 'RATE_LIMIT_EXCEEDED',
            message: 'Too many OTP requests. Please try again later.',
            requestId,
          },
        }),
      };
    }
    
    // 3. Generate OTP
    const otp = generateOTP();
    const sessionId = generateSessionId();
    const expiresAt = new Date(Date.now() + 5 * 60 * 1000); // 5 minutes
    
    // 4. Store OTP session
    await dynamodb.putItem({
      TableName: process.env.OTP_SESSIONS_TABLE,
      Item: {
        sessionId: { S: sessionId },
        phoneNumber: { S: phoneNumber },
        otpHash: { S: hashOTP(otp) },
        attempts: { N: '0' },
        maxAttempts: { N: '3' },
        expiresAt: { N: expiresAt.getTime().toString() },
        createdAt: { S: new Date().toISOString() },
      },
    });
    
    // 5. Send OTP via WhatsApp
    const whatsapp = await getWhatsAppClient();
    await whatsapp.sendMessage({
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
              { type: 'text', text: '5 minutes' },
            ],
          },
        ],
      },
    });
    
    logger.info('OTP sent successfully', { 
      requestId, 
      sessionId,
      phoneNumber: maskPhoneNumber(phoneNumber),
    });
    
    return {
      statusCode: 200,
      body: JSON.stringify({
        data: {
          sessionId,
          expiresAt: expiresAt.toISOString(),
          maxAttempts: 3,
        },
        meta: {
          requestId,
          timestamp: new Date().toISOString(),
        },
      }),
    };
    
  } catch (error) {
    logger.error('Failed to send OTP', {
      requestId,
      error: error.message,
      phoneNumber: maskPhoneNumber(phoneNumber),
    });
    
    return {
      statusCode: 500,
      body: JSON.stringify({
        error: {
          code: 'OTP_SEND_FAILED',
          message: 'Failed to send OTP. Please try again.',
          requestId,
        },
      }),
    };
  }
};

// Helper functions
function generateOTP(): string {
  return randomInt(100000, 999999).toString();
}

function generateSessionId(): string {
  return createHash('sha256')
    .update(`${Date.now()}-${Math.random()}`)
    .digest('hex');
}

function hashOTP(otp: string): string {
  return createHash('sha256')
    .update(otp + process.env.OTP_SALT)
    .digest('hex');
}

function isValidPhoneNumber(phoneNumber: string): boolean {
  // Basic validation - should use libphonenumber-js in production
  return /^\+?[1-9]\d{1,14}$/.test(phoneNumber);
}

function maskPhoneNumber(phoneNumber: string): string {
  const length = phoneNumber.length;
  return phoneNumber.substring(0, 3) + '***' + phoneNumber.substring(length - 4);
}

async function getWhatsAppClient(): Promise<WhatsAppBusinessAPI> {
  if (!whatsappClient) {
    const secret = await secretsManager.send(new GetSecretValueCommand({
      SecretId: process.env.WHATSAPP_API_SECRET,
    }));
    
    const { apiKey, phoneNumberId } = JSON.parse(secret.SecretString!);
    
    whatsappClient = new WhatsAppBusinessAPI({
      apiKey,
      phoneNumberId,
    });
  }
  
  return whatsappClient;
}
```

## API Gateway Configuration

### API Gateway Setup
```yaml
# serverless.yml configuration for API Gateway
service: eventfulhq-api

provider:
  name: aws
  runtime: nodejs18.x
  stage: ${opt:stage, 'dev'}
  region: ${opt:region, 'us-east-1'}
  apiGateway:
    restApiId: ${cf:eventfulhq-infra.ApiGatewayRestApiId}
    restApiRootResourceId: ${cf:eventfulhq-infra.ApiGatewayRestApiRootResourceId}
    binaryMediaTypes:
      - 'image/*'
      - 'application/pdf'
      - 'application/octet-stream'
  environment:
    STAGE: ${self:provider.stage}
    REGION: ${self:provider.region}

custom:
  cors:
    origin:
      - https://eventfulhq.com
      - https://eventfulamerica.com
      - https://eventfularabia.com
      - https://eventfulindia.com
      - https://eventfuleurope.com
      - https://eventfulaustralia.com
      - http://localhost:3000
    headers:
      - Content-Type
      - X-Request-ID
      - Authorization
      - X-API-Key
      - X-Region
      - Accept-Language
    allowCredentials: true
  
  authorizer:
    name: jwtAuthorizer
    type: REQUEST
    identitySource: method.request.header.Authorization
    resultTtlInSeconds: 300

functions:
  # Auth endpoints
  whatsappOtp:
    handler: dist/lambdas/auth/whatsapp-otp.handler
    events:
      - http:
          path: /auth/whatsapp-otp
          method: post
          cors: ${self:custom.cors}
  
  verifyOtp:
    handler: dist/lambdas/auth/verify-otp.handler
    events:
      - http:
          path: /auth/verify-otp
          method: post
          cors: ${self:custom.cors}
  
  # Event endpoints
  createEvent:
    handler: dist/lambdas/events/create.handler
    events:
      - http:
          path: /events
          method: post
          cors: ${self:custom.cors}
          authorizer: ${self:custom.authorizer}
  
  listEvents:
    handler: dist/lambdas/events/list.handler
    events:
      - http:
          path: /events
          method: get
          cors: ${self:custom.cors}
          request:
            parameters:
              querystrings:
                page: false
                limit: false
                search: false
                category: false
                city: false
                startDate: false
                endDate: false
  
  getEvent:
    handler: dist/lambdas/events/get.handler
    events:
      - http:
          path: /events/{eventId}
          method: get
          cors: ${self:custom.cors}
          request:
            parameters:
              paths:
                eventId: true
  
  # Admin endpoints
  listAdmins:
    handler: dist/lambdas/admin/list-admins.handler
    events:
      - http:
          path: /admin/admins
          method: get
          cors: ${self:custom.cors}
          authorizer: ${self:custom.authorizer}
  
  updateFeature:
    handler: dist/lambdas/admin/update-feature.handler
    events:
      - http:
          path: /admin/features/{featureId}
          method: patch
          cors: ${self:custom.cors}
          authorizer: ${self:custom.authorizer}
  
  # Developer portal endpoints
  createApiKey:
    handler: dist/lambdas/developer/create-api-key.handler
    events:
      - http:
          path: /developer/api-keys
          method: post
          cors: ${self:custom.cors}
          authorizer: ${self:custom.authorizer}
  
  getUsageStats:
    handler: dist/lambdas/developer/usage-stats.handler
    events:
      - http:
          path: /developer/usage
          method: get
          cors: ${self:custom.cors}
          authorizer: ${self:custom.authorizer}
  
  # WebSocket endpoints
  websocketConnect:
    handler: dist/lambdas/websocket/connect.handler
    events:
      - websocket:
          route: $connect
          authorizer:
            name: websocketAuthorizer
            identitySource:
              - 'route.request.querystring.token'
  
  websocketDisconnect:
    handler: dist/lambdas/websocket/disconnect.handler
    events:
      - websocket:
          route: $disconnect
  
  websocketMessage:
    handler: dist/lambdas/websocket/message.handler
    events:
      - websocket:
          route: $default

resources:
  Resources:
    # API Gateway configuration
    ApiGatewayRestApi:
      Type: AWS::ApiGateway::RestApi
      Properties:
        Name: EventfulHQ API
        Description: Main API for EventfulHQ platform
        EndpointConfiguration:
          Types:
            - REGIONAL
        Policy:
          Version: '2012-10-17'
          Statement:
            - Effect: Allow
              Principal: '*'
              Action: 'execute-api:Invoke'
              Resource: '*'
    
    # Request validator
    RequestValidator:
      Type: AWS::ApiGateway::RequestValidator
      Properties:
        Name: RequestValidator
        RestApiId: !Ref ApiGatewayRestApi
        ValidateRequestBody: true
        ValidateRequestParameters: true
    
    # API Key for developer portal
    ApiKey:
      Type: AWS::ApiGateway::ApiKey
      Properties:
        Name: EventfulHQ-Developer-Key
        Description: API key for developer portal access
        Enabled: true
    
    # Usage plan
    UsagePlan:
      Type: AWS::ApiGateway::UsagePlan
      Properties:
        UsagePlanName: EventfulHQ-Standard
        Description: Standard usage plan for EventfulHQ API
        ApiStages:
          - ApiId: !Ref ApiGatewayRestApi
            Stage: ${self:provider.stage}
        Throttle:
          BurstLimit: 5000
          RateLimit: 2000
        Quota:
          Limit: 1000000
          Period: MONTH
    
    # WAF Web ACL
    WebACL:
      Type: AWS::WAFv2::WebACL
      Properties:
        Name: EventfulHQ-WAF
        Scope: REGIONAL
        DefaultAction:
          Allow: {}
        Rules:
          - Name: RateLimitRule
            Priority: 1
            Statement:
              RateBasedStatement:
                Limit: 2000
                AggregateKeyType: IP
            Action:
              Block: {}
            VisibilityConfig:
              SampledRequestsEnabled: true
              CloudWatchMetricsEnabled: true
              MetricName: RateLimitRule
          - Name: SQLInjectionRule
            Priority: 2
            Statement:
              ManagedRuleGroupStatement:
                VendorName: AWS
                Name: AWSManagedRulesSQLiRuleSet
            OverrideAction:
              None: {}
            VisibilityConfig:
              SampledRequestsEnabled: true
              CloudWatchMetricsEnabled: true
              MetricName: SQLInjectionRule
```

### API Gateway Custom Authorizer
```typescript
// lambdas/authorizers/jwt-authorizer.ts
import { APIGatewayAuthorizerResult, APIGatewayTokenAuthorizerEvent } from 'aws-lambda';
import { verify } from 'jsonwebtoken';
import { SecretsManagerClient, GetSecretValueCommand } from '@aws-sdk/client-secrets-manager';
import { DynamoDBClient, GetItemCommand } from '@aws-sdk/client-dynamodb';

const secretsManager = new SecretsManagerClient({});
const dynamodb = new DynamoDBClient({});

let jwtSecret: string;

export const handler = async (
  event: APIGatewayTokenAuthorizerEvent
): Promise<APIGatewayAuthorizerResult> => {
  try {
    // Extract token
    const token = event.authorizationToken.replace('Bearer ', '');
    
    // Get JWT secret
    if (!jwtSecret) {
      const secret = await secretsManager.send(new GetSecretValueCommand({
        SecretId: process.env.JWT_SECRET_ID,
      }));
      jwtSecret = secret.SecretString!;
    }
    
    // Verify token
    const decoded = verify(token, jwtSecret) as any;
    
    // Check if token is blacklisted
    const blacklistCheck = await dynamodb.send(new GetItemCommand({
      TableName: process.env.TOKEN_BLACKLIST_TABLE,
      Key: {
        token: { S: token },
      },
    }));
    
    if (blacklistCheck.Item) {
      throw new Error('Token is blacklisted');
    }
    
    // Build policy
    const policy: APIGatewayAuthorizerResult = {
      principalId: decoded.sub,
      policyDocument: {
        Version: '2012-10-17',
        Statement: [
          {
            Action: 'execute-api:Invoke',
            Effect: 'Allow',
            Resource: event.methodArn,
          },
        ],
      },
      context: {
        userId: decoded.sub,
        email: decoded.email,
        role: decoded.role,
        permissions: JSON.stringify(decoded.permissions || []),
      },
    };
    
    return policy;
    
  } catch (error) {
    throw new Error('Unauthorized');
  }
};
```

### API Gateway Response Templates
```typescript
// api-gateway/response-templates.ts
export const responseTemplates = {
  // Success response template
  success: {
    'application/json': `
      #set($inputRoot = $input.path('

### Language Switcher Component
```typescript
// components/features/i18n/language-switcher.tsx
'use client';

import { useLocale } from 'next-intl';
import { useRouter, usePathname } from 'next/navigation';
import { useTransition } from 'react';
import {
  Select,
  SelectContent,
  SelectItem,
  SelectTrigger,
  SelectValue,
} from '@/components/ui/select';
import { Globe } from 'lucide-react';

const languages = [
  { code: 'en', name: 'English', flag: '🇺🇸', dir: 'ltr' },
  { code: 'zh', name: '中文', flag: '🇨🇳', dir: 'ltr' },
  { code: 'es', name: 'Español', flag: '🇪🇸', dir: 'ltr' },
  { code: 'pt', name: 'Português', flag: '🇧🇷', dir: 'ltr' },
  { code: 'fr', name: 'Français', flag: '🇫🇷', dir: 'ltr' },
  { code: 'ja', name: '日本語', flag: '🇯🇵', dir: 'ltr' },
  { code: 'ar', name: 'العربية', flag: '🇸🇦', dir: 'rtl' },
  { code: 'hi', name: 'हिन्दी', flag: '🇮🇳', dir: 'ltr' },
  { code: 'ru', name: 'Русский', flag: '🇷🇺', dir: 'ltr' },
];

export function LanguageSwitcher() {
  const locale = useLocale();
  const router = useRouter();
  const pathname = usePathname();
  const [isPending, startTransition] = useTransition();

  const handleLanguageChange = (newLocale: string) => {
    // Save preference
    document.cookie = `preferred-locale=${newLocale};max-age=31536000;path=/;samesite=lax${
      process.env.NODE_ENV === 'production' ? ';secure' : ''
    }`;

    // Update HTML dir attribute for RTL languages
    const newLang = languages.find(l => l.code === newLocale);
    document.documentElement.dir = newLang?.dir || 'ltr';

    // Navigate to new locale
    startTransition(() => {
      const segments = pathname.split('/');
      segments[1] = newLocale;
      router.push(segments.join('/'));
    });
  };

  const currentLanguage = languages.find(lang => lang.code === locale);

  return (
    <Select 
      value={locale} 
      onValueChange={handleLanguageChange}
      disabled={isPending}
    >
      <SelectTrigger className="w-[180px]">
        <Globe className="mr-2 h-4 w-4" />
        <SelectValue>
          {currentLanguage && (
            <span className="flex items-center">
              <span className="mr-2">{currentLanguage.flag}</span>
              {currentLanguage.name}
            </span>
          )}
        </SelectValue>
      </SelectTrigger>
      <SelectContent>
        {languages.map((language) => (
          <SelectItem key={language.code} value={language.code}>
            <span className="flex items-center">
              <span className="mr-2 text-lg">{language.flag}</span>
              <span>{language.name}</span>
            </span>
          </SelectItem>
        ))}
      </SelectContent>
    </Select>
  );
}
```

## State Management & Data Flow

### Zustand Store Architecture
```typescript
// stores/use-app-store.ts
import { create } from 'zustand';
import { devtools, persist } from 'zustand/middleware';
import { immer } from 'zustand/middleware/immer';

interface AppState {
  // User state
  user: User | null;
  isAuthenticated: boolean;
  permissions: Permission[];
  
  // UI state
  sidebarOpen: boolean;
  theme: 'light' | 'dark' | 'system';
  locale: string;
  
  // Feature flags
  features: Record<string, boolean>;
  
  // Actions
  setUser: (user: User | null) => void;
  toggleSidebar: () => void;
  setTheme: (theme: 'light' | 'dark' | 'system') => void;
  setLocale: (locale: string) => void;
  updateFeatureFlag: (flag: string, enabled: boolean) => void;
  reset: () => void;
}

export const useAppStore = create<AppState>()(
  devtools(
    persist(
      immer((set) => ({
        // Initial state
        user: null,
        isAuthenticated: false,
        permissions: [],
        sidebarOpen: true,
        theme: 'system',
        locale: 'en',
        features: {},
        
        // Actions
        setUser: (user) =>
          set((state) => {
            state.user = user;
            state.isAuthenticated = !!user;
            state.permissions = user?.permissions || [];
          }),
          
        toggleSidebar: () =>
          set((state) => {
            state.sidebarOpen = !state.sidebarOpen;
          }),
          
        setTheme: (theme) =>
          set((state) => {
            state.theme = theme;
          }),
          
        setLocale: (locale) =>
          set((state) => {
            state.locale = locale;
          }),
          
        updateFeatureFlag: (flag, enabled) =>
          set((state) => {
            state.features[flag] = enabled;
          }),
          
        reset: () =>
          set((state) => {
            state.user = null;
            state.isAuthenticated = false;
            state.permissions = [];
            state.features = {};
          }),
      })),
      {
        name: 'app-storage',
        partialize: (state) => ({
          theme: state.theme,
          locale: state.locale,
          sidebarOpen: state.sidebarOpen,
        }),
      }
    )
  )
);

// Separate stores for different domains
export const useEventStore = create<EventState>()(
  devtools(
    immer((set) => ({
      // Event specific state and actions
    }))
  )
);

export const useAdminStore = create<AdminState>()(
  devtools(
    immer((set) => ({
      // Admin specific state and actions
    }))
  )
);
```

### React Query Configuration
```typescript
// lib/react-query.ts
import {
  QueryClient,
  QueryClientProvider,
  QueryCache,
  MutationCache,
} from '@tanstack/react-query';
import { toast } from 'sonner';

// Create a custom query client with global error handling
export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 5 * 60 * 1000, // 5 minutes
      gcTime: 10 * 60 * 1000, // 10 minutes (formerly cacheTime)
      retry: (failureCount, error: any) => {
        // Don't retry on 4xx errors
        if (error?.response?.status >= 400 && error?.response?.status < 500) {
          return false;
        }
        return failureCount < 3;
      },
      refetchOnWindowFocus: false,
    },
    mutations: {
      retry: false,
    },
  },
  queryCache: new QueryCache({
    onError: (error: any) => {
      // Global query error handling
      if (error?.response?.status === 401) {
        // Handle unauthorized
        window.location.href = '/login';
      } else if (error?.response?.status === 403) {
        toast.error('You do not have permission to perform this action');
      } else if (error?.response?.status >= 500) {
        toast.error('Server error. Please try again later.');
      }
    },
  }),
  mutationCache: new MutationCache({
    onError: (error: any) => {
      // Global mutation error handling
      const message = error?.response?.data?.message || 'An error occurred';
      toast.error(message);
    },
  }),
});

// Custom hooks for common queries
export function useUser() {
  return useQuery({
    queryKey: ['user'],
    queryFn: userService.getCurrentUser,
    staleTime: Infinity, // User data rarely changes
  });
}

export function useEvents(filters?: EventFilters) {
  return useQuery({
    queryKey: ['events', filters],
    queryFn: () => eventService.getEvents(filters),
  });
}

export function useEvent(eventId: string) {
  return useQuery({
    queryKey: ['events', eventId],
    queryFn: () => eventService.getEvent(eventId),
    enabled: !!eventId,
  });
}

// Optimistic updates example
export function useUpdateEvent() {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: eventService.updateEvent,
    onMutate: async (updatedEvent) => {
      // Cancel outgoing refetches
      await queryClient.cancelQueries({ queryKey: ['events', updatedEvent.id] });
      
      // Snapshot previous value
      const previousEvent = queryClient.getQueryData(['events', updatedEvent.id]);
      
      // Optimistically update
      queryClient.setQueryData(['events', updatedEvent.id], updatedEvent);
      
      // Return context with snapshot
      return { previousEvent };
    },
    onError: (err, updatedEvent, context) => {
      // Rollback on error
      if (context?.previousEvent) {
        queryClient.setQueryData(
          ['events', updatedEvent.id],
          context.previousEvent
        );
      }
    },
    onSettled: (data, error, variables) => {
      // Always refetch after error or success
      queryClient.invalidateQueries({ queryKey: ['events', variables.id] });
    },
  });
}
```

### Server State Synchronization
```typescript
// hooks/use-realtime-sync.ts
import { useEffect } from 'react';
import { useQueryClient } from '@tanstack/react-query';
import { io, Socket } from 'socket.io-client';

let socket: Socket;

export function useRealtimeSync() {
  const queryClient = useQueryClient();
  
  useEffect(() => {
    // Initialize WebSocket connection
    socket = io(process.env.NEXT_PUBLIC_WS_URL!, {
      auth: {
        token: localStorage.getItem('auth-token'),
      },
    });
    
    // Listen for real-time updates
    socket.on('event:updated', (data) => {
      // Update React Query cache
      queryClient.setQueryData(['events', data.id], data);
      
      // Invalidate list queries
      queryClient.invalidateQueries({ 
        queryKey: ['events'],
        exact: false,
      });
    });
    
    socket.on('event:deleted', (data) => {
      // Remove from cache
      queryClient.removeQueries({ queryKey: ['events', data.id] });
      
      // Invalidate list queries
      queryClient.invalidateQueries({ 
        queryKey: ['events'],
        exact: false,
      });
    });
    
    socket.on('notification', (notification) => {
      // Handle real-time notifications
      toast.info(notification.message);
    });
    
    return () => {
      socket.disconnect();
    };
  }, [queryClient]);
  
  return socket;
}

// Hook for subscribing to specific channels
export function useRealtimeChannel(channel: string, handler: (data: any) => void) {
  const socket = useRealtimeSync();
  
  useEffect(() => {
    if (!socket) return;
    
    socket.on(channel, handler);
    
    return () => {
      socket.off(channel, handler);
    };
  }, [socket, channel, handler]);
}
```

## Form Handling with Yup & Zod

### Yup Schema Examples
```typescript
// validations/event.schema.ts
import * as yup from 'yup';
import { isValidPhoneNumber } from 'libphonenumber-js';

// Custom validators
yup.addMethod(yup.string, 'phone', function(message = 'Invalid phone number') {
  return this.test('phone', message, function(value) {
    if (!value) return true;
    return isValidPhoneNumber(value);
  });
});

// Event creation schema
export const createEventSchema = yup.object({
  title: yup
    .string()
    .required('Event title is required')
    .min(5, 'Title must be at least 5 characters')
    .max(100, 'Title must not exceed 100 characters'),
    
  description: yup
    .string()
    .required('Description is required')
    .min(20, 'Description must be at least 20 characters')
    .max(5000, 'Description must not exceed 5000 characters'),
    
  date: yup
    .date()
    .required('Event date is required')
    .min(new Date(), 'Event date must be in the future'),
    
  endDate: yup
    .date()
    .nullable()
    .when('date', (date, schema) => {
      return date ? schema.min(date, 'End date must be after start date') : schema;
    }),
    
  venue: yup.object({
    name: yup.string().required('Venue name is required'),
    address: yup.string().required('Venue address is required'),
    city: yup.string().required('City is required'),
    country: yup.string().required('Country is required'),
    coordinates: yup.object({
      lat: yup.number().required().min(-90).max(90),
      lng: yup.number().required().min(-180).max(180),
    }).required('Venue coordinates are required'),
  }).required('Venue information is required'),
  
  capacity: yup
    .number()
    .required('Capacity is required')
    .positive('Capacity must be positive')
    .integer('Capacity must be a whole number')
    .max(1000000, 'Capacity seems unrealistic'),
    
  price: yup
    .number()
    .required('Price is required')
    .min(0, 'Price cannot be negative')
    .max(100000, 'Price seems too high'),
    
  currency: yup
    .string()
    .required('Currency is required')
    .oneOf(['USD', 'EUR', 'GBP', 'INR', 'AED', 'SAR']),
    
  categories: yup
    .array()
    .of(yup.string())
    .min(1, 'Select at least one category')
    .max(5, 'Maximum 5 categories allowed'),
    
  images: yup
    .array()
    .of(
      yup.object({
        url: yup.string().url('Invalid image URL'),
        alt: yup.string(),
      })
    )
    .min(1, 'At least one image is required')
    .max(10, 'Maximum 10 images allowed'),
    
  whatsappRSVP: yup.object({
    enabled: yup.boolean(),
    phoneNumber: yup.string().when('enabled', {
      is: true,
      then: (schema) => schema.required('WhatsApp number is required').phone(),
    }),
    message: yup.string().when('enabled', {
      is: true,
      then: (schema) => schema.required('RSVP message is required'),
    }),
  }),
  
  isPublished: yup.boolean(),
  
  tickets: yup.array().of(
    yup.object({
      name: yup.string().required('Ticket name is required'),
      price: yup.number().required().min(0),
      quantity: yup.number().required().positive().integer(),
      description: yup.string(),
    })
  ),
});

// User registration schema
export const userRegistrationSchema = yup.object({
  name: yup
    .string()
    .required('Name is required')
    .matches(/^[a-zA-Z\s]+$/, 'Name can only contain letters and spaces'),
    
  email: yup
    .string()
    .required('Email is required')
    .email('Invalid email address'),
    
  phoneNumber: yup
    .string()
    .required('Phone number is required')
    .phone('Invalid phone number'),
    
  password: yup
    .string()
    .required('Password is required')
    .min(8, 'Password must be at least 8 characters')
    .matches(
      /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]/,
      'Password must contain uppercase, lowercase, number and special character'
    ),
    
  confirmPassword: yup
    .string()
    .required('Please confirm your password')
    .oneOf([yup.ref('password')], 'Passwords must match'),
    
  acceptTerms: yup
    .boolean()
    .required('You must accept the terms and conditions')
    .oneOf([true], 'You must accept the terms and conditions'),
});

// Dynamic field validation
export const createDynamicSchema = (fields: FormField[]) => {
  const shape: Record<string, any> = {};
  
  fields.forEach((field) => {
    let validator = yup.string();
    
    if (field.required) {
      validator = validator.required(`${field.label} is required`);
    }
    
    if (field.type === 'email') {
      validator = validator.email('Invalid email');
    }
    
    if (field.type === 'number') {
      validator = yup.number();
      if (field.min !== undefined) {
        validator = validator.min(field.min);
      }
      if (field.max !== undefined) {
        validator = validator.max(field.max);
      }
    }
    
    if (field.pattern) {
      validator = validator.matches(field.pattern, field.patternMessage);
    }
    
    shape[field.name] = validator;
  });
  
  return yup.object(shape);
};
```

### Zod Schema Examples (Secondary)
```typescript
// validations/admin.schema.ts
import { z } from 'zod';

// Admin creation schema using Zod
export const createAdminSchema = z.object({
  name: z.string().min(2, 'Name must be at least 2 characters'),
  email: z.string().email('Invalid email address'),
  phoneNumber: z.string().regex(/^\+?[1-9]\d{1,14}$/, 'Invalid phone number'),
  type: z.enum([
    'SUPER_ADMIN',
    'PLATFORM_ADMIN',
    'REGIONAL_ADMIN',
    'COUNTRY_ADMIN',
    'CITY_ADMIN',
    'CATEGORY_ADMIN',
    'CONTENT_ADMIN',
    'SUPPORT_ADMIN',
    'FINANCE_ADMIN',
    'MARKETING_ADMIN',
    'DEVELOPER_ADMIN',
  ]),
  permissions: z.array(z.object({
    resource: z.string(),
    actions: z.array(z.enum(['READ', 'WRITE', 'UPDATE', 'DELETE', 'PUBLISH', 'EXPORT', 'CONFIGURE'])),
    conditions: z.record(z.any()).optional(),
  })),
  geographyAccess: z.array(z.object({
    type: z.enum(['GLOBAL', 'REGION', 'COUNTRY', 'CITY']),
    identifier: z.string(),
    permissions: z.array(z.string()),
  })),
  featureAccess: z.array(z.object({
    microservice: z.string(),
    permissions: z.array(z.string()),
    limits: z.record(z.number()).optional(),
  })),
});

// API request validation
export const apiRequestSchema = z.object({
  method: z.enum(['GET', 'POST', 'PUT', 'DELETE', 'PATCH']),
  endpoint: z.string().url(),
  headers: z.record(z.string()).optional(),
  body: z.any().optional(),
  queryParams: z.record(z.string()).optional(),
});

// Type inference from Zod schemas
export type CreateAdminInput = z.infer<typeof createAdminSchema>;
export type APIRequest = z.infer<typeof apiRequestSchema>;
```

### React Hook Form Integration
```typescript
// components/features/events/create-event-form.tsx
'use client';

import { useForm } from 'react-hook-form';
import { yupResolver } from '@hookform/resolvers/yup';
import { createEventSchema } from '@/validations/event.schema';
import { Form, FormControl, FormDescription, FormField, FormItem, FormLabel, FormMessage } from '@/components/ui/form';
import { Input } from '@/components/ui/input';
import { Textarea } from '@/components/ui/textarea';
import { Button } from '@/components/ui/button';
import { Calendar } from '@/components/ui/calendar';
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from '@/components/ui/select';
import { Switch } from '@/components/ui/switch';
import { toast } from 'sonner';

type FormData = yup.InferType<typeof createEventSchema>;

export function CreateEventForm() {
  const form = useForm<FormData>({
    resolver: yupResolver(createEventSchema),
    defaultValues: {
      title: '',
      description: '',
      date: undefined,
      endDate: undefined,
      venue: {
        name: '',
        address: '',
        city: '',
        country: '',
        coordinates: { lat: 0, lng: 0 },
      },
      capacity: 100,
      price: 0,
      currency: 'USD',
      categories: [],
      images: [],
      whatsappRSVP: {
        enabled: false,
        phoneNumber: '',
        message: '',
      },
      isPublished: false,
      tickets: [],
    },
  });

  const { watch, setValue } = form;
  const whatsappEnabled = watch('whatsappRSVP.enabled');

  const onSubmit = async (data: FormData) => {
    try {
      const response = await eventService.createEvent(data);
      toast.success('Event created successfully!');
      router.push(`/events/${response.id}`);
    } catch (error) {
      toast.error('Failed to create event');
    }
  };

  return (
    <Form {...form}>
      <form onSubmit={form.handleSubmit(onSubmit)} className="space-y-8">
        <FormField
          control={form.control}
          name="title"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Event Title</FormLabel>
              <FormControl>
                <Input placeholder="Summer Music Festival" {...field} />
              </FormControl>
              <FormDescription>
                Choose a catchy title for your event
              </FormDescription>
              <FormMessage />
            </FormItem>
          )}
        />

        <FormField
          control={form.control}
          name="description"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Description</FormLabel>
              <FormControl>
                <Textarea
                  placeholder="Describe your event..."
                  className="min-h-[120px]"
                  {...field}
                />
              </FormControl>
              <FormMessage />
            </FormItem>
          )}
        />

        <div className="grid gap-4 md:grid-cols-2">
          <FormField
            control={form.control}
            name="date"
            render={({ field }) => (
              <FormItem className="flex flex-col">
                <FormLabel>Start Date</FormLabel>
                <Calendar
                  mode="single"
                  selected={field.value}
                  onSelect={field.onChange}
                  disabled={(date) => date < new Date()}
                  initialFocus
                />
                <FormMessage />
              </FormItem>
            )}
          />

          <FormField
            control={form.control}
            name="endDate"
            render={({ field }) => (
              <FormItem className="flex flex-col">
                <FormLabel>End Date (Optional)</FormLabel>
                <Calendar
                  mode="single"
                  selected={field.value}
                  onSelect={field.onChange}
                  disabled={(date) => date < watch('date')}
                />
                <FormMessage />
              </FormItem>
            )}
          />
        </div>

        <div className="space-y-4">
          <h3 className="text-lg font-semibold">Venue Information</h3>
          
          <FormField
            control={form.control}
            name="venue.name"
            render={({ field }) => (
              <FormItem>
                <FormLabel>Venue Name</FormLabel>
                <FormControl>
                  <Input placeholder="Madison Square Garden" {...field} />
                </FormControl>
                <FormMessage />
              </FormItem>
            )}
          />

          <FormField
            control={form.control}
            name="venue.address"
            render={({ field }) => (
              <FormItem>
                <FormLabel>Address</FormLabel>
                <FormControl>
                  <Input placeholder="4 Pennsylvania Plaza" {...field} />
                </FormControl>
                <FormMessage />
              </FormItem>
            )}
          />

          <div className="grid gap-4 md:grid-cols-2">
            <FormField
              control={form.control}
              name="venue.city"
              render={({ field }) => (
                <FormItem>
                  <FormLabel>City</FormLabel>
                  <FormControl>
                    <Input placeholder="New York" {...field} />
                  </FormControl>
                  <FormMessage />
                </FormItem>
              )}
            />

            <FormField
              control={form.control}
              name="venue.country"
              render={({ field }) => (
                <FormItem>
                  <FormLabel>Country</FormLabel>
                  <Select onValueChange={field.onChange} defaultValue={field.value}>
                    <FormControl>
                      <SelectTrigger>
                        <SelectValue placeholder="Select country" />
                      </SelectTrigger>
                    </FormControl>
                    <SelectContent>
                      <SelectItem value="US">United States</SelectItem>
                      <SelectItem value="GB">United Kingdom</SelectItem>
                      <SelectItem value="IN">India</SelectItem>
                      <SelectItem value="AE">UAE</SelectItem>
                      {/* Add more countries */}
                    </SelectContent>
                  </Select>
                  <FormMessage />
                </FormItem>
              )}
            />
          </div>
        </div>

        <div className="grid gap-4 md:grid-cols-3">
          <FormField
            control={form.control}
            name="capacity"
            render={({ field }) => (
              <FormItem>
                <FormLabel>Capacity</FormLabel>
                <FormControl>
                  <Input
                    type="number"
                    placeholder="500"
                    {...field}
                    onChange={(e) => field.onChange(parseInt(e.target.value))}
                  />
                </FormControl>
                <FormMessage />
              </FormItem>
            )}
          />

          <FormField
            control={form.control}
            name="price"
            render={({ field }) => (
              <FormItem>
                <FormLabel>Price</FormLabel>
                <FormControl>
                  <Input
                    type="number"
                    placeholder="0"
                    {...field}
                    onChange={(e) => field.onChange(parseFloat(e.target.value))}
                  />
                </FormControl>
                <FormMessage />
              </FormItem>
            )}
          />

          <FormField
            control={form.control}
            name="currency"
            render={({ field }) => (
              <FormItem>
                <FormLabel>Currency</FormLabel>
                <Select onValueChange={field.onChange} defaultValue={field.value}>
                  <FormControl>
                    <SelectTrigger>
                      <SelectValue />
                    </SelectTrigger>
                  </FormControl>
                  <SelectContent>
                    <SelectItem value="USD">USD</SelectItem>
                    <SelectItem value="EUR">EUR</SelectItem>
                    <SelectItem value="GBP">GBP</SelectItem>
                    <SelectItem value="INR">INR</SelectItem>
                    <SelectItem value="AED">AED</SelectItem>
                    <SelectItem value="SAR">SAR</SelectItem>
                  </SelectContent>
                </Select>
                <FormMessage />
              </FormItem>
            )}
          />
        </div>

        <div className="space-y-4">
          <FormField
            control={form.control}
            name="whatsappRSVP.enabled"
            render={({ field }) => (
              <FormItem className="flex flex-row items-center justify-between rounded-lg border p-4">
                <div className="space-y-0.5">
                  <FormLabel className="text-base">
                    WhatsApp RSVP
                  </FormLabel>
                  <FormDescription>
                    Allow attendees to RSVP via WhatsApp
                  </FormDescription>
                </div>
                <FormControl>
                  <Switch
                    checked={field.value}
                    onCheckedChange={field.onChange}
                  />
                </FormControl>
              </FormItem>
            )}
          />

          {whatsappEnabled && (
            <>
              <FormField
                control={form.control}
                name="whatsappRSVP.phoneNumber"
                render={({ field }) => (
                  <FormItem>
                    <FormLabel>WhatsApp Number</FormLabel>
                    <FormControl>
                      <Input placeholder="+1234567890" {...field} />
                    </FormControl>
                    <FormDescription>
                      Include country code
                    </FormDescription>
                    <FormMessage />
                  </FormItem>
                )}
              />

              <FormField
                control={form.control}
                name="whatsappRSVP.message"
                render={({ field }) => (
                  <FormItem>
                    <FormLabel>RSVP Message Template</FormLabel>
                    <FormControl>
                      <Textarea
                        placeholder="Hi! I'd like to RSVP for {event_name}"
                        {...field}
                      />
                    </FormControl>
                    <FormDescription>
                      Use {'{event_name}'} as placeholder
                    </FormDescription>
                    <FormMessage />
                  </FormItem>
                )}
              />
            </>
          )}
        </div>

        <div className="flex justify-end gap-4">
          <Button variant="outline" type="button">
            Save as Draft
          </Button>
          <Button type="submit">
            Create Event
          </Button>
        </div>
      </form>
    </Form>
  );
}
```

## Storybook Integration

### Storybook Configuration
```typescript
// .storybook/main.ts
import type { StorybookConfig } from '@storybook/nextjs';

const config: StorybookConfig = {
  stories: [
    '../stories/**/*.stories.@(js|jsx|ts|tsx|mdx)',
    '../src/**/*.stories.@(js|jsx|ts|tsx|mdx)',
  ],
  addons: [
    '@storybook/addon-essentials',
    '@storybook/addon-interactions',
    '@storybook/addon-links',
    '@storybook/addon-a11y',
    '@storybook/addon-viewport',
    '@storybook/addon-docs',
    '@storybook/addon-controls',
    'storybook-addon-next-router',
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
```

### Storybook Preview Configuration
```typescript
// .storybook/preview.tsx
import React from 'react';
import type { Preview } from '@storybook/react';
import { Inter } from 'next/font/google';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { NextIntlClientProvider } from 'next-intl';
import { ThemeProvider } from '@/components/theme-provider';
import '../src/styles/globals.css';

const inter = Inter({ subsets: ['latin'] });

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      retry: false,
      refetchOnWindowFocus: false,
    },
  },
});

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
    viewport: {
      viewports: {
        mobile: {
          name: 'Mobile',
          styles: {
            width: '375px',
            height: '667px',
          },
        },
        tablet: {
          name: 'Tablet',
          styles: {
            width: '768px',
            height: '1024px',
          },
        },
        desktop: {
          name: 'Desktop',
          styles: {
            width: '1440px',
            height: '900px',
          },
        },
      },
    },
  },
  decorators: [
    (Story) => (
      <QueryClientProvider client={queryClient}>
        <NextIntlClientProvider locale="en" messages={{}}>
          <ThemeProvider
            attribute="class"
            defaultTheme="system"
            enableSystem
            disableTransitionOnChange
          >
            <div className={inter.className}>
              <Story />
            </div>
          </ThemeProvider>
        </NextIntlClientProvider>
      </QueryClientProvider>
    ),
  ],
  globalTypes: {
    locale: {
      name: 'Locale',
      description: 'Internationalization locale',
      defaultValue: 'en',
      toolbar: {
        icon: 'globe',
        items: [
          { value: 'en', title: 'English' },
          { value: 'es', title: 'Spanish' },
          { value: 'fr', title: 'French' },
          { value: 'ar', title: 'Arabic' },
          { value: 'zh', title: 'Chinese' },
        ],
        showName: true,
      },
    },
    theme: {
      name: 'Theme',
      description: 'Theme for components',
      defaultValue: 'light',
      toolbar: {
        icon: 'circlehollow',
        items: ['light', 'dark', 'system'],
        showName: true,
      },
    },
  },
};

export default preview;
```

### Component Story Examples
```typescript
// stories/components/ui/button.stories.tsx
import type { Meta, StoryObj } from '@storybook/react';
import { Button } from '@/components/ui/button';
import { Mail, Loader2 } from 'lucide-react';

const meta = {
  title: 'UI/Button',
  component: Button,
  parameters: {
    layout: 'centered',
    docs: {
      description: {
        component: 'A flexible button component with multiple variants and sizes.',
      },
    },
  },
  tags: ['autodocs'],
  argTypes: {
    variant: {
      control: { type: 'select' },
      options: ['default', 'destructive', 'outline', 'secondary', 'ghost', 'link'],
      description: 'Button variant style',
    },
    size: {
      control: { type: 'select' },
      options: ['default', 'sm', 'lg', 'icon'],
      description: 'Button size',
    },
    disabled: {
      control: 'boolean',
      description: 'Disabled state',
    },
    asChild: {
      control: 'boolean',
      description: 'Render as child component',
    },
  },
} satisfies Meta<typeof Button>;

export default meta;
type Story = StoryObj<typeof meta>;

export const Default: Story = {
  args: {
    children: 'Button',
  },
};

export const Variants: Story = {
  render: () => (
    <div className="flex flex-wrap gap-4">
      <Button variant="default">Default</Button>
      <Button variant="destructive">Destructive</Button>
      <Button variant="outline">Outline</Button>
      <Button variant="secondary">Secondary</Button>
      <Button variant="ghost">Ghost</Button>
      <Button variant="link">Link</Button>
    </div>
  ),
};

export const Sizes: Story = {
  render: () => (
    <div className="flex items-center gap-4">
      <Button size="sm">Small</Button>
      <Button size="default">Default</Button>
      <Button size="lg">Large</Button>
      <Button size="icon">
        <Mail className="h-4 w-4" />
      </Button>
    </div>
  ),
};

export const WithIcon: Story = {
  render: () => (
    <div className="flex gap-4">
      <Button>
        <Mail className="mr-2 h-4 w-4" />
        Login with Email
      </Button>
      <Button variant="outline">
        <Mail className="mr-2 h-4 w-4" />
        Contact Us
      </Button>
    </div>
  ),
};

export const Loading: Story = {
  render: () => (
    <div className="flex gap-4">
      <Button disabled>
        <Loader2 className="mr-2 h-4 w-4 animate-spin" />
        Please wait
      </Button>
      <Button variant="outline" disabled>
        <Loader2 className="mr-2 h-4 w-4 animate-spin" />
        Loading...
      </Button>
    </div>
  ),
};

export const Disabled: Story = {
  render: () => (
    <div className="flex gap-4">
      <Button disabled>Disabled Default</Button>
      <Button variant="outline" disabled>
        Disabled Outline
      </Button>
      <Button variant="secondary" disabled>
        Disabled Secondary
      </Button>
    </div>
  ),
};

export const AsChild: Story = {
  render: () => (
    <Button asChild>
      <a href="https://eventfulhq.com" target="_blank" rel="noopener noreferrer">
        Open EventfulHQ
      </a>
    </Button>
  ),
};
```

### Feature Component Story
```typescript
// stories/features/events/event-card.stories.tsx
import type { Meta, StoryObj } from '@storybook/react';
import { EventCard } from '@/components/features/events/event-card';
import { mockEvent } from '@/test/mocks/event';

const meta = {
  title: 'Features/Events/EventCard',
  component: EventCard,
  parameters: {
    layout: 'padded',
  },
  tags: ['autodocs'],
  decorators: [
    (Story) => (
      <div className="max-w-md">
        <Story />
      </div>
    ),
  ],
} satisfies Meta<typeof EventCard>;

export default meta;
type Story = StoryObj<typeof meta>;

export const Default: Story = {
  args: {
    event: mockEvent,
  },
};

export const WithWhatsAppRSVP: Story = {
  args: {
    event: {
      ...mockEvent,
      whatsappRSVP: {
        enabled: true,
        phoneNumber: '+1234567890',
        message: 'Hi! I would like to RSVP for {event_name}',
      },
    },
  },
};

export const SoldOut: Story = {
  args: {
    event: {
      ...mockEvent,
      isSoldOut: true,
    },
  },
};

export const FreeEvent: Story = {
  args: {
    event: {
      ...mockEvent,
      price: 0,
    },
  },
};

export const Loading: Story = {
  args: {
    event: mockEvent,
    isLoading: true,
  },
};

export const Interactive: Story = {
  args: {
    event: mockEvent,
    onSelect: (event) => {
      console.log('Selected event:', event);
    },
    onShare: (event) => {
      console.log('Share event:', event);
    },
  },
  play: async ({ canvasElement, step }) => {
    const canvas = within(canvasElement);
    
    await step('Hover over card', async () => {
      await userEvent.hover(canvas.getByRole('article'));
    });
    
    await step('Click share button', async () => {
      await userEvent.click(canvas.getByRole('button', { name: /share/i }));
    });
  },
};
```

## API Integration & Code Generation

### OpenAPI Code Generation Setup
```typescript
// orval.config.ts
import { defineConfig } from 'orval';

export default defineConfig({
  eventfulhq: {
    input: {
      target: './openapi/eventfulhq.yaml',
      validation: true,
    },
    output: {
      mode: 'tags-split',
      target: './src/services/generated',
      schemas: './src/types/generated',
      client: 'react-query',
      httpClient: 'axios',
      override: {
        mutator: {
          path: './src/lib/api/client.ts',
          name: 'apiClient',
        },
        query: {
          useQuery: true,
          useInfinite: true,
          useInfiniteQueryParam: 'cursor',
          options: {
            staleTime: 5 * 60 * 1000,
          },
        },
        operations: {
          listEvents: {
            query: {
              useInfinite: true,
              useInfiniteQueryParam: 'page',
            },
          },
        },
      },
    },
    hooks: {
      afterAllFilesWrite: 'prettier --write',
    },
  },
  // Additional services
  whatsapp: {
    input: {
      target: './openapi/whatsapp-service.yaml',
    },
    output: {
      target: './src/services/generated/whatsapp',
      client: 'react-query',
    },
  },
  admin: {
    input: {
      target: './openapi/admin-service.yaml',
    },
    output: {
      target: './src/services/generated/admin',
      client: 'react-query',
    },
  },
});
```

### API Client Configuration
```typescript
// lib/api/client.ts
import axios, { AxiosError, AxiosRequestConfig } from 'axios';
import { toast } from 'sonner';

const BASE_URL = process.env.NEXT_PUBLIC_API_URL || 'https://api.eventfulhq.com';

// Create axios instance
export const apiClient = axios.create({
  baseURL: BASE_URL,
  timeout: 30000,
  headers: {
    'Content-Type': 'application/json',
  },
});

// Request interceptor
apiClient.interceptors.request.use(
  (config) => {
    // Add auth token
    const token = localStorage.getItem('auth-token');
    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }
    
    // Add request ID for tracing
    config.headers['X-Request-ID'] = generateRequestId();
    
    // Add locale
    const locale = localStorage.getItem('locale') || 'en';
    config.headers['Accept-Language'] = locale;
    
    // Add region
    const region = localStorage.getItem('preferred-region') || 'global';
    config.headers['X-Region'] = region;
    
    return config;
  },
  (error) => {
    return Promise.reject(error);
  }
);

// Response interceptor
apiClient.interceptors.response.use(
  (response) => {
    // Log successful responses in development
    if (process.env.NODE_ENV === 'development') {
      console.log('API Response:', {
        url: response.config.url,
        status: response.status,
        data: response.data,
      });
    }
    
    return response;
  },
  async (error: AxiosError) => {
    const originalRequest = error.config as AxiosRequestConfig & { _retry?: boolean };
    
    // Handle 401 - Token expired
    if (error.response?.status === 401 && !originalRequest._retry) {
      originalRequest._retry = true;
      
      try {
        const refreshToken = localStorage.getItem('refresh-token');
        const response = await axios.post(`${BASE_URL}/auth/refresh`, {
          refreshToken,
        });
        
        const { accessToken, refreshToken: newRefreshToken } = response.data;
        localStorage.setItem('auth-token', accessToken);
        localStorage.setItem('refresh-token', newRefreshToken);
        
        // Retry original request
        return apiClient(originalRequest);
      } catch (refreshError) {
        // Refresh failed, redirect to login
        localStorage.clear();
        window.location.href = '/login';
        return Promise.reject(refreshError);
      }
    }
    
    // Handle other errors
    handleApiError(error);
    
    return Promise.reject(error);
  }
);

// Error handler
function handleApiError(error: AxiosError<any>) {
  if (!error.response) {
    // Network error
    toast.error('Network error. Please check your connection.');
    return;
  }
  
  const { status, data } = error.response;
  
  switch (status) {
    case 400:
      toast.error(data.message || 'Invalid request');
      break;
    case 403:
      toast.error('You do not have permission to perform this action');
      break;
    case 404:
      toast.error('Resource not found');
      break;
    case 429:
      toast.error('Too many requests. Please try again later.');
      break;
    case 500:
      toast.error('Server error. Please try again later.');
      break;
    default:
      toast.error(data.message || 'An error occurred');
  }
}

// Custom API client wrapper for mutations
export async function apiMutator<T = any>(
  config: AxiosRequestConfig
): Promise<T> {
  const response = await apiClient(config);
  return response.data;
}

// Generate unique request ID
function generateRequestId(): string {
  return `${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;
}

// Typed API endpoints
export const API_ENDPOINTS = {
  // Auth
  auth: {
    login: '/auth/login',
    register: '/auth/register',
    refresh: '/auth/refresh',
    logout: '/auth/logout',
    whatsappOtp: '/auth/whatsapp-otp',
    verifyOtp: '/auth/verify-otp',
  },
  
  // Events
  events: {
    list: '/events',
    detail: (id: string) => `/events/${id}`,
    create: '/events',
    update: (id: string) => `/events/${id}`,
    delete: (id: string) => `/events/${id}`,
    publish: (id: string) => `/events/${id}/publish`,
    attendees: (id: string) => `/events/${id}/attendees`,
  },
  
  // RFP
  rfp: {
    list: '/rfp',
    create: '/rfp',
    detail: (id: string) => `/rfp/${id}`,
    proposals: (id: string) => `/rfp/${id}/proposals`,
    submitProposal: (id: string) => `/rfp/${id}/proposals`,
  },
  
  // Admin
  admin: {
    admins: '/admin/admins',
    features: '/admin/features',
    permissions: '/admin/permissions',
    geography: '/admin/geography',
    audit: '/admin/audit-logs',
  },
  
  // Developer
  developer: {
    apiKeys: '/developer/api-keys',
    usage: '/developer/usage',
    webhooks: '/developer/webhooks',
    docs: '/developer/docs',
  },
} as const;
```

### Service Layer Architecture
```typescript
// services/base-service.ts
import { apiClient } from '@/lib/api/client';
import { AxiosRequestConfig } from 'axios';

export interface PaginatedResponse<T> {
  data: T[];
  meta: {
    total: number;
    page: number;
    limit: number;
    totalPages: number;
  };
}

export interface QueryParams {
  page?: number;
  limit?: number;
  sort?: string;
  order?: 'asc' | 'desc';
  search?: string;
  filters?: Record<string, any>;
}

export abstract class BaseService {
  protected endpoint: string;

  constructor(endpoint: string) {
    this.endpoint = endpoint;
  }

  protected async get<T>(
    path: string = '',
    config?: AxiosRequestConfig
  ): Promise<T> {
    const response = await apiClient.get<T>(`${this.endpoint}${path}`, config);
    return response.data;
  }

  protected async post<T, D = any>(
    path: string = '',
    data?: D,
    config?: AxiosRequestConfig
  ): Promise<T> {
    const response = await apiClient.post<T>(`${this.endpoint}${path}`, data, config);
    return response.data;
  }

  protected async put<T, D = any>(
    path: string = '',
    data?: D,
    config?: AxiosRequestConfig
  ): Promise<T> {
    const response = await apiClient.put<T>(`${this.endpoint}${path}`, data, config);
    return response.data;
  }

  protected async patch<T, D = any>(
    path: string = '',
    data?: D,
    config?: AxiosRequestConfig
  ): Promise<T> {
    const response = await apiClient.patch<T>(`${this.endpoint}${path}`, data, config);
    return response.data;
  }

  protected async delete<T>(
    path: string = '',
    config?: AxiosRequestConfig
  ): Promise<T> {
    const response = await apiClient.delete<T>(`${this.endpoint}${path}`, config);
    return response.data;
  }

  protected buildQueryString(params: QueryParams): string {
    const query = new URLSearchParams();
    
    if (params.page) query.append('page', params.page.toString());
    if (params.limit) query.append('limit', params.limit.toString());
    if (params.sort) query.append('sort', params.sort);
    if (params.order) query.append('order', params.order);
    if (params.search) query.append('search', params.search);
    
    if (params.filters) {
      Object.entries(params.filters).forEach(([key, value]) => {
        if (value !== undefined && value !== null && value !== '') {
          query.append(key, value.toString());
        }
      });
    }
    
    return query.toString() ? `?${query.toString()}` : '';
  }
}

// Event service implementation
export class EventService extends BaseService {
  constructor() {
    super('/events');
  }

  async getEvents(params?: QueryParams): Promise<PaginatedResponse<Event>> {
    const queryString = this.buildQueryString(params || {});
    return this.get<PaginatedResponse<Event>>(queryString);
  }

  async getEvent(id: string): Promise<Event> {
    return this.get<Event>(`/${id}`);
  }

  async createEvent(data: CreateEventInput): Promise<Event> {
    return this.post<Event>('', data);
  }

  async updateEvent(id: string, data: UpdateEventInput): Promise<Event> {
    return this.put<Event>(`/${id}`, data);
  }

  async deleteEvent(id: string): Promise<void> {
    return this.delete<void>(`/${id}`);
  }

  async publishEvent(id: string): Promise<Event> {
    return this.post<Event>(`/${id}/publish`);
  }

  async unpublishEvent(id: string): Promise<Event> {
    return this.post<Event>(`/${id}/unpublish`);
  }

  async getEventAttendees(
    id: string,
    params?: QueryParams
  ): Promise<PaginatedResponse<Attendee>> {
    const queryString = this.buildQueryString(params || {});
    return this.get<PaginatedResponse<Attendee>>(`/${id}/attendees${queryString}`);
  }

  async sendWhatsAppInvites(
    id: string,
    data: { phoneNumbers: string[]; message: string }
  ): Promise<void> {
    return this.post<void>(`/${id}/whatsapp-invites`, data);
  }
}

// Service registry
export const eventService = new EventService();
export const userService = new UserService();
export const adminService = new AdminService();
export const developerService = new DeveloperService();
export const rfpService = new RFPService();
export const analyticsService = new AnalyticsService();
```

## Authentication & Security Patterns

### WhatsApp OTP Authentication
```typescript
// lib/auth/whatsapp-auth.ts
import { apiClient } from '@/lib/api/client';
import { z } from 'zod';

// Validation schemas
const phoneSchema = z.string().regex(/^\+?[1-9]\d{1,14}$/, 'Invalid phone number');
const otpSchema = z.string().length(6, 'OTP must be 6 digits');

export interface WhatsAppAuthState {
  phoneNumber: string;
  sessionId: string;
  expiresAt: Date;
  attempts: number;
  maxAttempts: number;
}

export class WhatsAppAuth {
  private static SESSION_KEY = 'whatsapp-auth-session';
  
  static async sendOTP(phoneNumber: string): Promise<WhatsAppAuthState> {
    // Validate phone number
    const validatedPhone = phoneSchema.parse(phoneNumber);
    
    try {
      const response = await apiClient.post('/auth/whatsapp-otp', {
        phoneNumber: validatedPhone,
      });
      
      const session: WhatsAppAuthState = {
        phoneNumber: validatedPhone,
        sessionId: response.data.sessionId,
        expiresAt: new Date(response.data.expiresAt),
        attempts: 0,
        maxAttempts: response.data.maxAttempts || 3,
      };
      
      // Store session
      sessionStorage.setItem(this.SESSION_KEY, JSON.stringify(session));
      
      return session;
    } catch (error) {
      throw new Error('Failed to send OTP');
    }
  }
  
  static async verifyOTP(otp: string): Promise<AuthResponse> {
    // Get session
    const sessionStr = sessionStorage.getItem(this.SESSION_KEY);
    if (!sessionStr) {
      throw new Error('No active OTP session');
    }
    
    const session: WhatsAppAuthState = JSON.parse(sessionStr);
    
    // Check expiration
    if (new Date() > new Date(session.expiresAt)) {
      sessionStorage.removeItem(this.SESSION_KEY);
      throw new Error('OTP session expired');
    }
    
    // Validate OTP format
    const validatedOTP = otpSchema.parse(otp);
    
    try {
      const response = await apiClient.post('/auth/verify-otp', {
        sessionId: session.sessionId,
        otp: validatedOTP,
      });
      
      // Clear session
      sessionStorage.removeItem(this.SESSION_KEY);
      
      // Store tokens
      const { accessToken, refreshToken, user } = response.data;
      localStorage.setItem('auth-token', accessToken);
      localStorage.setItem('refresh-token', refreshToken);
      localStorage.setItem('user', JSON.stringify(user));
      
      return response.data;
    } catch (error: any) {
      // Update attempts
      session.attempts += 1;
      
      if (session.attempts >= session.maxAttempts) {
        sessionStorage.removeItem(this.SESSION_KEY);
        throw new Error('Maximum OTP attempts exceeded');
      }
      
      sessionStorage.setItem(this.SESSION_KEY, JSON.stringify(session));
      
      throw new Error(
        error.response?.data?.message || 
        `Invalid OTP. ${session.maxAttempts - session.attempts} attempts remaining`
      );
    }
  }
  
  static async resendOTP(): Promise<WhatsAppAuthState> {
    const sessionStr = sessionStorage.getItem(this.SESSION_KEY);
    if (!sessionStr) {
      throw new Error('No active OTP session');
    }
    
    const session: WhatsAppAuthState = JSON.parse(sessionStr);
    return this.sendOTP(session.phoneNumber);
  }
  
  static clearSession(): void {
    sessionStorage.removeItem(this.SESSION_KEY);
  }
}

// WhatsApp OTP Component
export function WhatsAppOTPModal({ 
  isOpen, 
  onClose, 
  phoneNumber,
  onSuccess 
}: WhatsAppOTPModalProps) {
  const [otp, setOtp] = useState(['', '', '', '', '', '']);
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);
  const [session, setSession] = useState<WhatsAppAuthState | null>(null);
  const inputRefs = useRef<(HTMLInputElement | null)[]>([]);
  
  useEffect(() => {
    if (isOpen && phoneNumber) {
      sendOTP();
    }
  }, [isOpen, phoneNumber]);
  
  const sendOTP = async () => {
    setIsLoading(true);
    setError(null);
    
    try {
      const session = await WhatsAppAuth.sendOTP(phoneNumber);
      setSession(session);
      toast.success('OTP sent to your WhatsApp');
    } catch (error: any) {
      setError(error.message);
      toast.error('Failed to send OTP');
    } finally {
      setIsLoading(false);
    }
  };
  
  const handleOTPChange = (index: number, value: string) => {
    if (!/^\d*$/.test(value)) return;
    
    const newOtp = [...otp];
    newOtp[index] = value;
    setOtp(newOtp);
    
    // Auto-focus next input
    if (value && index < 5) {
      inputRefs.current[index + 1]?.focus();
    }
    
    // Auto-submit if all digits entered
    if (newOtp.every(digit => digit) && newOtp.join('').length === 6) {
      verifyOTP(newOtp.join(''));
    }
  };
  
  const handleKeyDown = (index: number, e: KeyboardEvent<HTMLInputElement>) => {
    if (e.key === 'Backspace' && !otp[index] && index > 0) {
      inputRefs.current[index - 1]?.focus();
    }
  };
  
  const verifyOTP = async (otpCode?: string) => {
    const code = otpCode || otp.join('');
    if (code.length !== 6) {
      setError('Please enter all 6 digits');
      return;
    }
    
    setIsLoading(true);
    setError(null);
    
    try {
      const response = await WhatsAppAuth.verifyOTP(code);
      toast.success('Successfully authenticated!');
      onSuccess(response);
      onClose();
    } catch (error: any) {
      setError(error.message);
      setOtp(['', '', '', '', '', '']);
      inputRefs.current[0]?.focus();
    } finally {
      setIsLoading(false);
    }
  };
  
  return (
    <Dialog open={isOpen} onOpenChange={onClose}>
      <DialogContent className="sm:max-w-md">
        <DialogHeader>
          <DialogTitle>Enter WhatsApp OTP</DialogTitle>
          <DialogDescription>
            We've sent a 6-digit code to {phoneNumber}
          </DialogDescription>
        </DialogHeader>
        
        <div className="space-y-6 py-4">
          <div className="flex justify-center gap-2">
            {otp.map((digit, index) => (
              <Input
                key={index}
                ref={(el) => (inputRefs.current[index] = el)}
                type="text"
                inputMode="numeric"
                maxLength={1}
                value={digit}
                onChange={(e) => handleOTPChange(index, e.target.value)}
                onKeyDown={(e) => handleKeyDown(index, e)}
                className="w-12 h-12 text-center text-lg font-semibold"
                disabled={isLoading}
              />
            ))}
          </div>
          
          {error && (
            <Alert variant="destructive">
              <AlertDescription>{error}</AlertDescription>
            </Alert>
          )}
          
          {session && (
            <div className="text-center text-sm text-muted-foreground">
              <p>
                Code expires in{' '}
                <CountdownTimer
                  expiresAt={session.expiresAt}
                  onExpire={() => {
                    setError('OTP expired. Please request a new one.');
                    WhatsAppAuth.clearSession();
                  }}
                />
              </p>
            </div>
          )}
          
          <div className="flex flex-col gap-2">
            <Button
              onClick={() => verifyOTP()}
              disabled={isLoading || otp.some(d => !d)}
              className="w-full"
            >
              {isLoading ? (
                <>
                  <Loader2 className="mr-2 h-4 w-4 animate-spin" />
                  Verifying...
                </>
              ) : (
                'Verify OTP'
              )}
            </Button>
            
            <Button
              variant="outline"
              onClick={sendOTP}
              disabled={isLoading}
              className="w-full"
            >
              Resend OTP
            </Button>
          </div>
        </div>
      </DialogContent>
    </Dialog>
  );
}
```

### Role-Based Access Control (RBAC)
```typescript
// lib/auth/rbac.ts
import { AdminType, Action, Permission } from '@/types/admin.types';

export class RBAC {
  private static instance: RBAC;
  private permissions: Map<string, Set<string>> = new Map();
  
  private constructor() {
    this.initializePermissions();
  }
  
  static getInstance(): RBAC {
    if (!RBAC.instance) {
      RBAC.instance = new RBAC();
    }
    return RBAC.instance;
  }
  
  private initializePermissions() {
    // Define default permissions for each admin type
    const defaultPermissions: Record<AdminType, Permission[]> = {
      [AdminType.SUPER_ADMIN]: [
        { resource: '*', actions: [Action.READ, Action.WRITE, Action.DELETE, Action.CONFIGURE] },
      ],
      [AdminType.PLATFORM_ADMIN]: [
        { resource: 'events', actions: [Action.READ, Action.WRITE, Action.DELETE] },
        { resource: 'users', actions: [Action.READ, Action.WRITE] },
        { resource: 'analytics', actions: [Action.READ] },
      ],
      [AdminType.REGIONAL_ADMIN]: [
        { resource: 'events', actions: [Action.READ, Action.WRITE], conditions: { region: 'assigned' } },
        { resource: 'users', actions: [Action.READ], conditions: { region: 'assigned' } },
      ],
      // ... other admin types
    };
    
    // Build permission map
    Object.entries(defaultPermissions).forEach(([adminType, permissions]) => {
      const key = this.getPermissionKey(adminType as AdminType);
      const permSet = new Set<string>();
      
      permissions.forEach(perm => {
        perm.actions.forEach(action => {
          permSet.add(`${perm.resource}:${action}`);
        });
      });
      
      this.permissions.set(key, permSet);
    });
  }
  
  hasPermission(
    adminType: AdminType,
    resource: string,
    action: Action,
    context?: Record<string, any>
  ): boolean {
    const key = this.getPermissionKey(adminType);
    const permissions = this.permissions.get(key);
    
    if (!permissions) return false;
    
    // Check wildcard permission
    if (permissions.has(`*:${action}`)) return true;
    
    // Check specific permission
    return permissions.has(`${resource}:${action}`);
  }
  
  private getPermissionKey(adminType: AdminType): string {
    return `admin:${adminType}`;
  }
}

// Permission hook
export function usePermissions() {
  const { user } = useAppStore();
  const rbac = RBAC.getInstance();
  
  const hasPermission = useCallback(
    (resource: string, action: Action, context?: Record<string, any>) => {
      if (!user || !user.adminType) return false;
      return rbac.hasPermission(user.adminType, resource, action, context);
    },
    [user, rbac]
  );
  
  const hasAnyPermission = useCallback(
    (permissions: Array<{ resource: string; action: Action }>) => {
      return permissions.some(({ resource, action }) => 
        hasPermission(resource, action)
      );
    },
    [hasPermission]
  );
  
  const hasAllPermissions = useCallback(
    (permissions: Array<{ resource: string; action: Action }>) => {
      return permissions.every(({ resource, action }) => 
        hasPermission(resource, action)
      );
    },
    [hasPermission]
  );
  
  return {
    hasPermission,
    hasAnyPermission,
    hasAllPermissions,
  };
}

// Protected route component
export function ProtectedRoute({ 
  children, 
  resource, 
  action,
  fallback = <Navigate to="/unauthorized" replace />
}: ProtectedRouteProps) {
  const { hasPermission } = usePermissions();
  
  if (!hasPermission(resource, action)) {
    return fallback;
  }
  
  return <>{children}</>;
}

// Permission guard component
export function PermissionGuard({ 
  children, 
  resource, 
  action,
  fallback = null
}: PermissionGuardProps) {
  const { hasPermission } = usePermissions();
  
  if (!hasPermission(resource, action)) {
    return <>{fallback}</>;
  }
  
  return <>{children}</>;
}
```

### Security Headers & CSP
```typescript
// middleware/security-headers.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function securityHeadersMiddleware(request: NextRequest) {
  const response = NextResponse.next();
  
  // Security headers
  response.headers.set('X-Frame-Options', 'DENY');
  response.headers.set('X-Content-Type-Options', 'nosniff');
  response.headers.set('X-XSS-Protection', '1; mode=block');
  response.headers.set('Referrer-Policy', 'strict-origin-when-cross-origin');
  response.headers.set('Permissions-Policy', 'camera=(), microphone=(), geolocation=()');
  
  // Content Security Policy
  const csp = [
    "default-src 'self'",
    "script-src 'self' 'unsafe-inline' 'unsafe-eval' https://cdn.jsdelivr.net https://cdnjs.cloudflare.com https://pagead2.googlesyndication.com",
    "style-src 'self' 'unsafe-inline' https://fonts.googleapis.com",
    "font-src 'self' https://fonts.gstatic.com",
    "img-src 'self' data: https: blob:",
    "connect-src 'self' https://api.eventfulhq.com wss://ws.eventfulhq.com https://pagead2.googlesyndication.com",
    "frame-src 'self' https://googleads.g.doubleclick.net",
    "object-src 'none'",
    "base-uri 'self'",
    "form-action 'self'",
    "frame-ancestors 'none'",
    "upgrade-insecure-requests",
  ].join('; ');
  
  response.headers.set('Content-Security-Policy', csp);
  
  // HSTS for production
  if (process.env.NODE_ENV === 'production') {
    response.headers.set(
      'Strict-Transport-Security',
      'max-age=63072000; includeSubDomains; preload'
    );
  }
  
  return response;
}
```

## Performance Optimization & SLA Compliance

### Next.js Performance Configuration
```typescript
// next.config.mjs
import { withSentryConfig } from '@sentry/nextjs';
import million from 'million/compiler';

/** @type {import('next').NextConfig} */
const nextConfig = {
  // React compiler for optimization
  experimental: {
    reactCompiler: true,
    optimizePackageImports: [
      '@shadcn/ui',
      'lucide-react',
      '@tabler/icons-react',
      'recharts',
      'd3',
    ],
    turbo: {
      resolveAlias: {
        '@': './src',
      },
    },
  },
  
  // Image optimization
  images: {
    domains: ['cdn.eventfulhq.com'],
    formats: ['image/avif', 'image/webp'],
    deviceSizes: [640, 750, 828, 1080, 1200, 1920, 2048, 3840],
    imageSizes: [16, 32, 48, 64, 96, 128, 256, 384],
  },
  
  // Bundle optimization
  webpack: (config, { isServer }) => {
    // Tree shaking
    config.optimization.usedExports = true;
    config.optimization.sideEffects = false;
    
    // Module federation for microfrontends
    if (!isServer) {
      config.optimization.splitChunks = {
        chunks: 'all',
        cacheGroups: {
          default: false,
          vendors: false,
          framework: {
            chunks: 'all',
            name: 'framework',
            test: /(?<!node_modules.*)[\\/]node_modules[\\/](react|react-dom|scheduler|prop-types|use-subscription)[\\/]/,
            priority: 40,
            enforce: true,
          },
          lib: {
            test(module) {
              return module.size() > 160000 &&
                /node_modules[/\\]/.test(module.identifier());
            },
            name(module) {
              const hash = crypto.createHash('sha1');
              hash.update(module.identifier());
              return hash.digest('hex').substring(0, 8);
            },
            priority: 30,
            minChunks: 1,
            reuseExistingChunk: true,
          },
          commons: {
            name: 'commons',
            minChunks: 2,
            priority: 20,
          },
          shared: {
            name(module, chunks) {
              return crypto
                .createHash('sha1')
                .update(chunks.reduce((acc, chunk) => acc + chunk.name, ''))
                .digest('hex') + (isServer ? '-server' : '');
            },
            priority: 10,
            minChunks: 2,
            reuseExistingChunk: true,
          },
        },
      };
    }
    
    return config;
  },
  
  // Output optimization
  output: 'standalone',
  compress: true,
  poweredByHeader: false,
  generateEtags: true,
  
  // Compiler optimizations
  compiler: {
    removeConsole: process.env.NODE_ENV === 'production',
    reactRemoveProperties: process.env.NODE_ENV === 'production',
  },
};

// Apply Million.js compiler for React optimization
const millionConfig = {
  auto: true,
  rsc: true,
};

// Sentry configuration
const sentryWebpackPluginOptions = {
  silent: true,
  widenClientFileUpload: true,
};

export default million.next(
  withSentryConfig(nextConfig, sentryWebpackPluginOptions),
  millionConfig
);
```

### Performance Monitoring
```typescript
// lib/performance/monitoring.ts
import { getCLS, getFID, getLCP, getFCP, getTTFB } from 'web-vitals';

export class PerformanceMonitor {
  private static instance: PerformanceMonitor;
  private metrics: Map<string, number> = new Map();
  
  private constructor() {
    this.initializeVitals();
    this.setupResourceObserver();
    this.setupLongTaskObserver();
  }
  
  static getInstance(): PerformanceMonitor {
    if (!PerformanceMonitor.instance) {
      PerformanceMonitor.instance = new PerformanceMonitor();
    }
    return PerformanceMonitor.instance;
  }
  
  private initializeVitals() {
    // Core Web Vitals
    getCLS(this.sendToAnalytics);
    getFID(this.sendToAnalytics);
    getLCP(this.sendToAnalytics);
    
    // Additional metrics
    getFCP(this.sendToAnalytics);
    getTTFB(this.sendToAnalytics);
  }
  
  private setupResourceObserver() {
    if ('PerformanceObserver' in window) {
      const observer = new PerformanceObserver((list) => {
        for (const entry of list.getEntries()) {
          if (entry.entryType === 'resource') {
            this.trackResourceTiming(entry as PerformanceResourceTiming);
          }
        }
      });
      
      observer.observe({ entryTypes: ['resource'] });
    }
  }
  
  private setupLongTaskObserver() {
    if ('PerformanceObserver' in window) {
      const observer = new PerformanceObserver((list) => {
        for (const entry of list.getEntries()) {
          console.warn('Long task detected:', {
            duration: entry.duration,
            startTime: entry.startTime,
          });
          
          // Report long tasks
          this.sendToAnalytics({
            name: 'long-task',
            value: entry.duration,
            id: generateId(),
            entries: [entry],
          });
        }
      });
      
      try {
        observer.observe({ entryTypes: ['longtask'] });
      } catch (e) {
        // Long task observer not supported
      }
    }
  }
  
  private trackResourceTiming(entry: PerformanceResourceTiming) {
    const duration = entry.responseEnd - entry.startTime;
    
    // Track slow resources
    if (duration > 1000) {
      console.warn('Slow resource:', {
        name: entry.name,
        duration,
        type: entry.initiatorType,
      });
    }
    
    // Categorize by type
    const category = this.getResourceCategory(entry);
    const currentTotal = this.metrics.get(category) || 0;
    this.metrics.set(category, currentTotal + duration);
  }
  
  private getResourceCategory(entry: PerformanceResourceTiming): string {
    if (entry.name.includes('/api/')) return 'api';
    if (entry.initiatorType === 'script') return 'script';
    if (entry.initiatorType === 'css') return 'css';
    if (entry.initiatorType === 'img') return 'image';
    return 'other';
  }
  
  private sendToAnalytics = (metric: any) => {
    // Send to analytics service
    if (window.gtag) {
      window.gtag('event', metric.name, {
        value: Math.round(metric.value),
        metric_id: metric.id,
        metric_value: metric.value,
        metric_delta: metric.delta,
      });
    }
    
    // Send to custom monitoring
    fetch('/api/monitoring/metrics', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        metric: metric.name,
        value: metric.value,
        timestamp: Date.now(),
        url: window.location.href,
        userAgent: navigator.userAgent,
      }),
    }).catch(() => {
      // Fail silently
    });
  };
  
  measureApiCall(endpoint: string, duration: number) {
    this.sendToAnalytics({
      name: 'api-call',
      value: duration,
      id: generateId(),
      entries: [],
      endpoint,
    });
  }
  
  getMetrics(): Record<string, number> {
    return Object.fromEntries(this.metrics);
  }
}

// Performance hook
export function usePerformance() {
  const monitor = PerformanceMonitor.getInstance();
  
  const measureApiCall = useCallback((endpoint: string, duration: number) => {
    monitor.measureApiCall(endpoint, duration);
  }, [monitor]);
  
  const getMetrics = useCallback(() => {
    return monitor.getMetrics();
  }, [monitor]);
  
  return {
    measureApiCall,
    getMetrics,
  };
}
```

### Image Optimization Component
```typescript
// components/shared/optimized-image.tsx
'use client';

import Image from 'next/image';
import { useState, useEffect } from 'react';
import { cn } from '@/lib/utils';

interface OptimizedImageProps {
  src: string;
  alt: string;
  width?: number;
  height?: number;
  priority?: boolean;
  className?: string;
  objectFit?: 'contain' | 'cover' | 'fill' | 'none' | 'scale-down';
  quality?: number;
  sizes?: string;
  onLoad?: () => void;
  placeholder?: 'blur' | 'empty';
  blurDataURL?: string;
}

export function OptimizedImage({
  src,
  alt,
  width,
  height,
  priority = false,
  className,
  objectFit = 'cover',
  quality = 75,
  sizes,
  onLoad,
  placeholder = 'blur',
  blurDataURL,
}: OptimizedImageProps) {
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState(false);
  
  // Generate blur data URL if not provided
  const shimmer = (w: number, h: number) => `
    <svg width="${w}" height="${h}" version="1.1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink">
      <defs>
        <linearGradient id="g">
          <stop stop-color="#333" offset="20%" />
          <stop stop-color="#222" offset="50%" />
          <stop stop-color="#333" offset="70%" />
        </linearGradient>
      </defs>
      <rect width="${w}" height="${h}" fill="#333" />
      <rect id="r" width="${w}" height="${h}" fill="url(#g)" />
      <animate xlink:href="#r" attributeName="x" from="-${w}" to="${w}" dur="1s" repeatCount="indefinite"  />
    </svg>`;
  
  const toBase64 = (str: string) =>
    typeof window === 'undefined'
      ? Buffer.from(str).toString('base64')
      : window.btoa(str);
  
  const defaultBlurDataURL = `data:image/svg+xml;base64,${toBase64(
    shimmer(width || 700, height || 475)
  )}`;
  
  // Cloudinary optimization
  const getOptimizedSrc = (src: string) => {
    if (src.includes('cloudinary')) {
      const parts = src.split('/upload/');
      if (parts.length === 2) {
        const transformations = [
          'f_auto', // Auto format
          'q_auto', // Auto quality
          'dpr_auto', // Auto DPR
          width ? `w_${width}` : '',
          height ? `h_${height}` : '',
        ].filter(Boolean).join(',');
        
        return `${parts[0]}/upload/${transformations}/${parts[1]}`;
      }
    }
    return src;
  };
  
  const optimizedSrc = getOptimizedSrc(src);
  
  if (error) {
    return (
      <div 
        className={cn(
          'bg-muted flex items-center justify-center',
          className
        )}
        style={{ width, height }}
      >
        <span className="text-muted-foreground text-sm">
          Failed to load image
        </span>
      </div>
    );
  }
  
  return (
    <div className={cn('relative overflow-hidden', className)}>
      <Image
        src={optimizedSrc}
        alt={alt}
        width={width}
        height={height}
        priority={priority}
        quality={quality}
        sizes={sizes || '(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw'}
        className={cn(
          'duration-700 ease-in-out',
          isLoading ? 'scale-110 blur-2xl grayscale' : 'scale-100 blur-0 grayscale-0',
          objectFit === 'cover' && 'object-cover',
          objectFit === 'contain' && 'object-contain',
          className
        )}
        onLoad={() => {
          setIsLoading(false);
          onLoad?.();
        }}
        onError={() => {
          setError(true);
          setIsLoading(false);
        }}
        placeholder={placeholder}
        blurDataURL={blurDataURL || defaultBlurDataURL}
      />
    </div>
  );
}

// Lazy load component
export function LazyImage({ threshold = 0.1, ...props }: OptimizedImageProps & { threshold?: number }) {
  const [isInView, setIsInView] = useState(false);
  const imgRef = useRef<HTMLDivElement>(null);
  
  useEffect(() => {
    const observer = new IntersectionObserver(
      (entries) => {
        entries.forEach((entry) => {
          if (entry.isIntersecting) {
            setIsInView(true);
            observer.disconnect();
          }
        });
      },
      { threshold }
    );
    
    if (imgRef.current) {
      observer.observe(imgRef.current);
    }
    
    return () => observer.disconnect();
  }, [threshold]);
  
  return (
    <div ref={imgRef}>
      {isInView ? (
        <OptimizedImage {...props} />
      ) : (
        <div 
          className={cn('bg-muted animate-pulse', props.className)}
          style={{ width: props.width, height: props.height }}
        />
      )}
    </div>
  );
}
```

### Code Splitting & Dynamic Imports
```typescript
// lib/performance/dynamic-imports.ts
import dynamic from 'next/dynamic';
import { ComponentType } from 'react';

// Dynamic import with loading state
export function createDynamicComponent<P = {}>(
  importFn: () => Promise<{ default: ComponentType<P> }>,
  options?: {
    loading?: ComponentType;
    ssr?: boolean;
  }
) {
  return dynamic(importFn, {
    loading: options?.loading || (() => <div>Loading...</div>),
    ssr: options?.ssr ?? true,
  });
}

// Pre-configured dynamic imports
export const DynamicEventCard = createDynamicComponent(
  () => import('@/components/features/events/event-card').then(mod => ({ default: mod.EventCard }))
);

export const DynamicSwaggerUI = createDynamicComponent(
  () => import('swagger-ui-react'),
  { ssr: false }
);

export const DynamicMonacoEditor = createDynamicComponent(
  () => import('@monaco-editor/react'),
  { ssr: false }
);

export const DynamicCharts = createDynamicComponent(
  () => import('@/components/features/analytics/charts')
);

// Route-based code splitting
export const DynamicAdminDashboard = createDynamicComponent(
  () => import('@/components/features/admin/super-admin-dashboard').then(mod => ({ default: mod.SuperAdminDashboard }))
);

export const DynamicDeveloperPortal = createDynamicComponent(
  () => import('@/components/features/developers/developer-dashboard').then(mod => ({ default: mod.DeveloperDashboard }))
);

// Lazy load heavy libraries
export const loadD3 = () => import('d3');
export const loadThree = () => import('three');
export const loadChartJS = () => import('chart.js');
```

## Testing Strategy

### Unit Testing Configuration
```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config';
import react from '@vitejs/plugin-react';
import path from 'path';

export default defineConfig({
  plugins: [react()],
  test: {
    environment: 'jsdom',
    globals: true,
    setupFiles: ['./tests/setup.ts'],
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html'],
      exclude: [
        'node_modules/',
        'tests/',
        '**/*.d.ts',
        '**/*.config.*',
        '**/*.stories.*',
        'src/services/generated/**',
      ],
      thresholds: {
        branches: 80,
        functions: 80,
        lines: 80,
        statements: 80,
      },
    },
  },
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
});

// tests/setup.ts
import '@testing-library/jest-dom';
import { cleanup } from '@testing-library/react';
import { afterEach, vi } from 'vitest';
import { server } from './mocks/server';

// Establish API mocking before all tests
beforeAll(() => server.listen({ onUnhandledRequest: 'error' }));

// Reset any request handlers that we may add during the tests,
// so they don't affect other tests
afterEach(() => {
  cleanup();
  server.resetHandlers();
});

// Clean up after the tests are finished
afterAll(() => server.close());

// Mock next/navigation
vi.mock('next/navigation', () => ({
  useRouter: () => ({
    push: vi.fn(),
    replace: vi.fn(),
    prefetch: vi.fn(),
    back: vi.fn(),
  }),
  usePathname: () => '/test-path',
  useSearchParams: () => new URLSearchParams(),
}));

// Mock next-intl
vi.mock('next-intl', () => ({
  useTranslations: () => (key: string) => key,
  useLocale: () => 'en',
}));
```

### Component Testing Examples
```typescript
// tests/components/event-card.test.tsx
import { describe, it, expect, vi } from 'vitest';
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { EventCard } from '@/components/features/events/event-card';
import { mockEvent } from '@/tests/mocks/event';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';

const queryClient = new QueryClient({
  defaultOptions: {
    queries: { retry: false },
    mutations: { retry: false },
  },
});

const renderWithProviders = (component: React.ReactElement) => {
  return render(
    <QueryClientProvider client={queryClient}>
      {component}
    </QueryClientProvider>
  );
};

describe('EventCard', () => {
  it('renders event information correctly', () => {
    renderWithProviders(<EventCard event={mockEvent} />);
    
    expect(screen.getByText(mockEvent.title)).toBeInTheDocument();
    expect(screen.getByText(mockEvent.venue.name)).toBeInTheDocument();
    expect(screen.getByText(`${mockEvent.price}`)).toBeInTheDocument();
  });
  
  it('shows sold out badge when event is sold out', () => {
    const soldOutEvent = { ...mockEvent, isSoldOut: true };
    renderWithProviders(<EventCard event={soldOutEvent} />);
    
    expect(screen.getByText('Sold Out')).toBeInTheDocument();
  });
  
  it('displays WhatsApp RSVP button when enabled', () => {
    const whatsappEvent = {
      ...mockEvent,
      whatsappRSVP: {
        enabled: true,
        phoneNumber: '+1234567890',
        message: 'RSVP for {event_name}',
      },
    };
    renderWithProviders(<EventCard event={whatsappEvent} />);
    
    expect(screen.getByText('RSVP via WhatsApp')).toBeInTheDocument();
  });
  
  it('calls onSelect when card is clicked', async () => {
    const onSelect = vi.fn();
    renderWithProviders(<EventCard event={mockEvent} onSelect={onSelect} />);
    
    const card = screen.getByRole('article');
    await userEvent.click(card);
    
    expect(onSelect).toHaveBeenCalledWith(mockEvent);
  });
  
  it('shares event when share button is clicked', async () => {
    const onShare = vi.fn();
    renderWithProviders(<EventCard event={mockEvent} onShare={onShare} />);
    
    const shareButton = screen.getByRole('button', { name: /share/i });
    await userEvent.click(shareButton);
    
    expect(onShare).toHaveBeenCalledWith(mockEvent);
  });
  
  it('displays loading skeleton when isLoading is true', () => {
    renderWithProviders(<EventCard event={mockEvent} isLoading />);
    
    expect(screen.getByTestId('event-card-skeleton')).toBeInTheDocument();
  });
});
```

### Integration Testing
```typescript
// tests/integration/event-creation.test.tsx
import { describe, it, expect, beforeEach } from 'vitest';
import { render, screen, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { CreateEventPage } from '@/app/[locale]/(dashboard)/events/create/page';
import { server } from '@/tests/mocks/server';
import { rest } from 'msw';

describe('Event Creation Flow', () => {
  beforeEach(() => {
    // Reset MSW handlers
    server.resetHandlers();
  });
  
  it('completes full event creation flow', async () => {
    const user = userEvent.setup();
    render(<CreateEventPage />);
    
    // Fill in event details
    await user.type(screen.getByLabelText('Event Title'), 'Test Music Festival');
    await user.type(screen.getByLabelText('Description'), 'An amazing outdoor music experience');
    
    // Select date
    await user.click(screen.getByLabelText('Start Date'));
    await user.click(screen.getByText('15')); // Select 15th of current month
    
    // Fill venue information
    await user.type(screen.getByLabelText('Venue Name'), 'Central Park');
    await user.type(screen.getByLabelText('Address'), '5th Avenue');
    await user.type(screen.getByLabelText('City'), 'New York');
    
    // Set capacity and price
    await user.clear(screen.getByLabelText('Capacity'));
    await user.type(screen.getByLabelText('Capacity'), '500');
    await user.clear(screen.getByLabelText('Price'));
    await user.type(screen.getByLabelText('Price'), '75');
    
    // Enable WhatsApp RSVP
    await user.click(screen.getByLabelText('WhatsApp RSVP'));
    await user.type(screen.getByLabelText('WhatsApp Number'), '+1234567890');
    
    // Mock successful API response
    server.use(
      rest.post('/api/events', (req, res, ctx) => {
        return res(
          ctx.status(201),
          ctx.json({
            id: '123',
            ...req.body,
          })
        );
      })
    );
    
    // Submit form
    await user.click(screen.getByRole('button', { name: 'Create Event' }));
    
    // Verify success
    await waitFor(() => {
      expect(screen.getByText('Event created successfully!')).toBeInTheDocument();
    });
  });
  
  it('shows validation errors for invalid input', async () => {
    const user = userEvent.setup();
    render(<CreateEventPage />);
    
    // Try to submit without filling required fields
    await user.click(screen.getByRole('button', { name: 'Create Event' }));
    
    // Check for validation errors
    await waitFor(() => {
      expect(screen.getByText('Event title is required')).toBeInTheDocument();
      expect(screen.getByText('Description is required')).toBeInTheDocument();
      expect(screen.getByText('Event date is required')).toBeInTheDocument();
    });
  });
});
```

### E2E Testing with Playwright
```typescript
// tests/e2e/event-flow.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Event Management Flow', () => {
  test.beforeEach(async ({ page }) => {
    // Login before each test
    await page.goto('/login');
    await page.fill('input[name="phoneNumber"]', '+1234567890');
    await page.click('button:has-text("Send OTP")');
    
    // Fill OTP
    for (let i = 0; i < 6; i++) {
      await page.fill(`input[name="otp-${i}"]`, '1');
    }
    
    await page.waitForURL('/dashboard');
  });
  
  test('creates and publishes an event', async ({ page }) => {
    // Navigate to create event
    await page.goto('/events/create');
    
    // Fill event form
    await page.fill('input[name="title"]', 'Playwright Test Event');
    await page.fill('textarea[name="description"]', 'Test event created by Playwright');
    
    // Select date
    await page.click('button[aria-label="Start Date"]');
    await page.click('button:has-text("15")');
    
    // Fill venue
    await page.fill('input[name="venue.name"]', 'Test Venue');
    await page.fill('input[name="venue.address"]', '123 Test Street');
    await page.fill('input[name="venue.city"]', 'Test City');
    
    // Submit
    await page.click('button:has-text("Create Event")');
    
    // Wait for redirect
    await page.waitForURL(/\/events\/[\w-]+$/);
    
    // Verify event created
    await expect(page.locator('h1')).toContainText('Playwright Test Event');
    
    // Publish event
    await page.click('button:has-text("Publish Event")');
    
    // Verify published
    await expect(page.locator('text=Published')).toBeVisible();
  });
  
  test('searches and filters events', async ({ page }) => {
    await page.goto('/events');
    
    // Search
    await page.fill('input[placeholder="Search events..."]', 'Music');
    await page.press('input[placeholder="Search events..."]', 'Enter');
    
    // Apply filters
    await page.click('button:has-text("Filters")');
    await page.selectOption('select[name="category"]', 'music');
    await page.fill('input[name="minPrice"]', '0');
    await page.fill('input[name="maxPrice"]', '100');
    await page.click('button:has-text("Apply Filters")');
    
    // Verify results
    await expect(page.locator('[data-testid="event-card"]')).toHaveCount(10);
  });
});
```

### Performance Testing
```typescript
// tests/performance/event-list.perf.ts
import { test, expect } from '@playwright/test';

test.describe('Performance Tests', () => {
  test('event list page loads within performance budget', async ({ page }) => {
    // Start measuring
    await page.goto('/events');
    
    // Measure Core Web Vitals
    const metrics = await page.evaluate(() => {
      return new Promise((resolve) => {
        let lcp, fid, cls;
        
        new PerformanceObserver((list) => {
          const entries = list.getEntries();
          entries.forEach((entry) => {
            if (entry.name === 'largest-contentful-paint') {
              lcp = entry.startTime;
            }
          });
        }).observe({ entryTypes: ['largest-contentful-paint'] });
        
        new PerformanceObserver((list) => {
          const entries = list.getEntries();
          entries.forEach((entry) => {
            if (!fid) {
              fid = entry.processingStart - entry.startTime;
            }
          });
        }).observe({ entryTypes: ['first-input'] });
        
        let clsValue = 0;
        new PerformanceObserver((list) => {
          for (const entry of list.getEntries()) {
            if (!entry.hadRecentInput) {
              clsValue += entry.value;
            }
          }
          cls = clsValue;
        }).observe({ entryTypes: ['layout-shift'] });
        
        // Wait for metrics
        setTimeout(() => {
          resolve({ lcp, fid, cls });
        }, 5000);
      });
    });
    
    // Assert performance budgets
    expect(metrics.lcp).toBeLessThan(2500); // LCP < 2.5s
    expect(metrics.fid || 0).toBeLessThan(100); // FID < 100ms
    expect(metrics.cls || 0).toBeLessThan(0.1); // CLS < 0.1
  });
  
  test('API response times meet SLA', async ({ page, request }) => {
    const endpoints = [
      '/api/events',
      '/api/users/me',
      '/api/analytics/dashboard',
    ];
    
    for (const endpoint of endpoints) {
      const start = Date.now();
      const response = await request.get(endpoint);
      const duration = Date.now() - start;
      
      expect(response.status()).toBe(200);
      expect(duration).toBeLessThan(500); // p95 < 500ms
    }
  });
});
```

## Vercel Deployment & Edge Functions

### Vercel Configuration
```json
// vercel.json
{
  "buildCommand": "npm run build",
  "outputDirectory": ".next",
  "framework": "nextjs",
  "regions": ["iad1", "sfo1", "sin1", "syd1", "fra1", "gru1"],
  "functions": {
    "app/api/bff/*": {
      "maxDuration": 30
    },
    "app/api/webhooks/*": {
      "maxDuration": 60
    },
    "app/[locale]/(landing)/[country]/[city]/page.tsx": {
      "runtime": "edge"
    },
    "app/robots.txt/route.ts": {
      "runtime": "edge"
    },
    "app/sitemap.xml/route.ts": {
      "runtime": "edge"
    }
  },
  "crons": [
    {
      "path": "/api/cron/sitemap-refresh",
      "schedule": "0 3 * * *"
    },
    {
      "path": "/api/cron/cache-warm",
      "schedule": "*/15 * * * *"
    }
  ],
  "env": {
    "NEXT_PUBLIC_API_URL": "@api-url",
    "NEXT_PUBLIC_WS_URL": "@ws-url",
    "NEXT_PUBLIC_ADSENSE_CLIENT_ID": "@adsense-client-id"
  },
  "headers": [
    {
      "source": "/(.*)",
      "headers": [
        {
          "key": "X-Frame-Options",
          "value": "DENY"
        },
        {
          "key": "X-Content-Type-Options",
          "value": "nosniff"
        },
        {
          "key": "Referrer-Policy",
          "value": "strict-origin-when-cross-origin"
        }
      ]
    },
    {
      "source": "/api/(.*)",
      "headers": [
        {
          "key": "Cache-Control",
          "value": "no-store, max-age=0"
        }
      ]
    }
  ],
  "rewrites": [
    {
      "source": "/api/v1/:path*",
      "destination": "https://api.eventfulhq.com/v1/:path*"
    }
  ],
  "redirects": [
    {
      "source": "/events",
      "destination": "/en/events",
      "permanent": false
    }
  ]
}
```

### Edge Function Examples
```typescript
// app/api/edge/geo-redirect/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { geolocation } from '@vercel/edge';

export const runtime = 'edge';
export const dynamic = 'force-dynamic';

export async function GET(request: NextRequest) {
  const { country, city, region } = geolocation(request);
  
  // Determine best regional domain
  const regionalDomain = getRegionalDomain(country);
  
  // Return redirect response
  return NextResponse.json({
    country,
    city,
    region,
    recommendedDomain: regionalDomain,
    currentDomain: request.headers.get('host'),
  });
}

function getRegionalDomain(country?: string): string {
  if (!country) return 'eventfulhq.com';
  
  const regionMap: Record<string, string> = {
    US: 'eventfulamerica.com',
    CA: 'eventfulamerica.com',
    MX: 'eventfulamerica.com',
    BR: 'eventfulamerica.com',
    AE: 'eventfularabia.com',
    SA: 'eventfularabia.com',
    KW: 'eventfularabia.com',
    IN: 'eventfulindia.com',
    GB: 'eventfuleurope.com',
    FR: 'eventfuleurope.com',
    DE: 'eventfuleurope.com',
    AU: 'eventfulaustralia.com',
    NZ: 'eventfulaustralia.com',
  };
  
  return regionMap[country] || 'eventfulhq.com';
}
```

### ISR & Cache Strategy
```typescript
// app/[locale]/(landing)/[country]/[city]/page.tsx
export const revalidate = 3600; // Revalidate every hour
export const dynamicParams = true; // Allow dynamic segments

// Generate static params for popular cities
export async function generateStaticParams() {
  const cities = await landingService.getTopCities(100);
  
  return cities.map((city) => ({
    country: city.countryCode.toLowerCase(),
    city: city.slug,
  }));
}

// Cache configuration for different page types
export const cacheConfig = {
  // Static pages
  landing: {
    revalidate: 86400, // 24 hours
    tags: ['landing'],
  },
  
  // Dynamic user pages
  userProfile: {
    revalidate: 300, // 5 minutes
    tags: (username: string) => ['user', `user:${username}`],
  },
  
  // Event pages
  event: {
    revalidate: 60, // 1 minute for real-time updates
    tags: (eventId: string) => ['event', `event:${eventId}`],
  },
};

// On-demand revalidation
export async function revalidateEvent(eventId: string) {
  try {
    await fetch(`${process.env.DEPLOYMENT_URL}/api/revalidate`, {
      method: 'POST',
      headers: {
        'x-revalidate-secret': process.env.REVALIDATE_SECRET!,
      },
      body: JSON.stringify({
        tags: [`event:${eventId}`],
      }),
    });
  } catch (error) {
    console.error('Failed to revalidate:', error);
  }
}
```

### Environment-Specific Configuration
```typescript
// lib/config/environment.ts
interface EnvironmentConfig {
  apiUrl: string;
  wsUrl: string;
  cdnUrl: string;
  analyticsId: string;
  sentryDsn: string;
  features: Record<string, boolean>;
}

const configs: Record<string, EnvironmentConfig> = {
  development: {
    apiUrl: 'http://localhost:3001',
    wsUrl: 'ws://localhost:3001',
    cdnUrl: 'http://localhost:3000',
    analyticsId: '',
    sentryDsn: '',
    features: {
      whatsappAuth: true,
      rfpMarketplace: true,
      aiRecommendations: true,
      developerPortal: true,
      superAdmin: true,
    },
  },
  
  staging: {
    apiUrl: 'https://api-staging.eventfulhq.com',
    wsUrl: 'wss://ws-staging.eventfulhq.com',
    cdnUrl: 'https://cdn-staging.eventfulhq.com',
    analyticsId: process.env.NEXT_PUBLIC_GA_STAGING!,
    sentryDsn: process.env.NEXT_PUBLIC_SENTRY_DSN_STAGING!,
    features: {
      whatsappAuth: true,
      rfpMarketplace: true,
      aiRecommendations: true,
      developerPortal: true,
      superAdmin: true,
    },
  },
  
  production: {
    apiUrl: 'https://api.eventfulhq.com',
    wsUrl: 'wss://ws.eventfulhq.com',
    cdnUrl: 'https://cdn.eventfulhq.com',
    analyticsId: process.env.NEXT_PUBLIC_GA_ID!,
    sentryDsn: process.env.NEXT_PUBLIC_SENTRY_DSN!,
    features: {
      whatsappAuth: true,
      rfpMarketplace: true,
      aiRecommendations: false, // Gradual rollout
      developerPortal: true,
      superAdmin: true,
    },
  },
};

export const config = configs[process.env.NEXT_PUBLIC_ENV || 'development'];

// Feature flag helper
export function isFeatureEnabled(feature: keyof typeof config.features): boolean {
  return config.features[feature] || false;
}
```

## Component Enhancement Patterns

### MatDash to ShadCN Migration Pattern
```typescript
// migration/button-migration.tsx
// BEFORE: MatDash Flowbite Button
import { Button as FlowbiteButton } from 'flowbite-react';

export function OldButton({ children, onClick, variant }) {
  return (
    <FlowbiteButton
      color={variant === 'danger' ? 'failure' : variant}
      onClick={onClick}
    >
      {children}
    </FlowbiteButton>
  );
}

// AFTER: ShadCN Button with enhanced features
import { Button } from '@/components/ui/button';
import { cn } from '@/lib/utils';
import { Loader2 } from 'lucide-react';

interface EnhancedButtonProps {
  children: React.ReactNode;
  onClick?: () => void;
  variant?: 'default' | 'destructive' | 'outline' | 'secondary' | 'ghost' | 'link';
  size?: 'default' | 'sm' | 'lg' | 'icon';
  loading?: boolean;
  disabled?: boolean;
  className?: string;
  asChild?: boolean;
}

export function EnhancedButton({
  children,
  onClick,
  variant = 'default',
  size = 'default',
  loading = false,
  disabled = false,
  className,
  asChild = false,
}: EnhancedButtonProps) {
  return (
    <Button
      variant={variant}
      size={size}
      onClick={onClick}
      disabled={disabled || loading}
      className={cn(className)}
      asChild={asChild}
    >
      {loading && <Loader2 className="mr-2 h-4 w-4 animate-spin" />}
      {children}
    </Button>
  );
}

// Migration utility
export function migrateButtonProps(oldProps: any): EnhancedButtonProps {
  const variantMap = {
    primary: 'default',
    danger: 'destructive',
    failure: 'destructive',
    success: 'secondary',
    warning: 'outline',
  };
  
  return {
    ...oldProps,
    variant: variantMap[oldProps.color] || 'default',
    loading: oldProps.isProcessing,
  };
}
```

### Form Enhancement Pattern
```typescript
// components/enhanced/form-field.tsx
import { forwardRef } from 'react';
import { useFormContext } from 'react-hook-form';
import { FormControl, FormDescription, FormField, FormItem, FormLabel, FormMessage } from '@/components/ui/form';
import { Input } from '@/components/ui/input';
import { Textarea } from '@/components/ui/textarea';
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from '@/components/ui/select';
import { cn } from '@/lib/utils';

interface EnhancedFormFieldProps {
  name: string;
  label?: string;
  description?: string;
  placeholder?: string;
  type?: 'text' | 'email' | 'password' | 'number' | 'tel' | 'url' | 'textarea' | 'select';
  options?: Array<{ value: string; label: string }>;
  disabled?: boolean;
  className?: string;
  required?: boolean;
  autoComplete?: string;
}

export const EnhancedFormField = forwardRef<
  HTMLInputElement | HTMLTextAreaElement | HTMLButtonElement,
  EnhancedFormFieldProps
>(({
  name,
  label,
  description,
  placeholder,
  type = 'text',
  options = [],
  disabled = false,
  className,
  required = false,
  autoComplete,
}, ref) => {
  const { control } = useFormContext();
  
  return (
    <FormField
      control={control}
      name={name}
      render={({ field }) => (
        <FormItem className={className}>
          {label && (
            <FormLabel>
              {label}
              {required && <span className="text-destructive ml-1">*</span>}
            </FormLabel>
          )}
          <FormControl>
            {type === 'textarea' ? (
              <Textarea
                {...field}
                ref={ref as any}
                placeholder={placeholder}
                disabled={disabled}
                className="min-h-[100px]"
              />
            ) : type === 'select' ? (
              <Select
                onValueChange={field.onChange}
                defaultValue={field.value}
                disabled={disabled}
              >
                <FormControl>
                  <SelectTrigger ref={ref as any}>
                    <SelectValue placeholder={placeholder} />
                  </SelectTrigger>
                </FormControl>
                <SelectContent>
                  {options.map((option) => (
                    <SelectItem key={option.value} value={option.value}>
                      {option.label}
                    </SelectItem>
                  ))}
                </SelectContent>
              </Select>
            ) : (
              <Input
                {...field}
                ref={ref as any}
                type={type}
                placeholder={placeholder}
                disabled={disabled}
                autoComplete={autoComplete}
              />
            )}
          </FormControl>
          {description && <FormDescription>{description}</FormDescription>}
          <FormMessage />
        </FormItem>
      )}
    />
  );
});

EnhancedFormField.displayName = 'EnhancedFormField';

// Usage example
export function EnhancedEventForm() {
  const form = useForm({
    resolver: yupResolver(eventSchema),
  });
  
  return (
    <Form {...form}>
      <form onSubmit={form.handleSubmit(onSubmit)} className="space-y-6">
        <EnhancedFormField
          name="title"
          label="Event Title"
          placeholder="Summer Music Festival"
          required
        />
        
        <EnhancedFormField
          name="description"
          label="Description"
          type="textarea"
          placeholder="Describe your event..."
          required
        />
        
        <EnhancedFormField
          name="category"
          label="Category"
          type="select"
          placeholder="Select a category"
          options={[
            { value: 'music', label: 'Music' },
            { value: 'sports', label: 'Sports' },
            { value: 'arts', label: 'Arts & Culture' },
          ]}
          required
        />
        
        <Button type="submit">Create Event</Button>
      </form>
    </Form>
  );
}
```

### Data Table Enhancement Pattern
```typescript
// components/enhanced/data-table.tsx
import {
  ColumnDef,
  ColumnFiltersState,
  SortingState,
  VisibilityState,
  flexRender,
  getCoreRowModel,
  getFilteredRowModel,
  getPaginationRowModel,
  getSortedRowModel,
  useReactTable,
} from '@tanstack/react-table';
import {
  Table,
  TableBody,
  TableCell,
  TableHead,
  TableHeader,
  TableRow,
} from '@/components/ui/table';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import {
  DropdownMenu,
  DropdownMenuCheckboxItem,
  DropdownMenuContent,
  DropdownMenuTrigger,
} from '@/components/ui/dropdown-menu';
import { ChevronDown } from 'lucide-react';

interface DataTableProps<TData, TValue> {
  columns: ColumnDef<TData, TValue>[];
  data: TData[];
  searchKey?: string;
  showColumnVisibility?: boolean;
  showPagination?: boolean;
  pageSize?: number;
}

export function EnhancedDataTable<TData, TValue>({
  columns,
  data,
  searchKey,
  showColumnVisibility = true,
  showPagination = true,
  pageSize = 10,
}: DataTableProps<TData, TValue>) {
  const [sorting, setSorting] = useState<SortingState>([]);
  const [columnFilters, setColumnFilters] = useState<ColumnFiltersState>([]);
  const [columnVisibility, setColumnVisibility] = useState<VisibilityState>({});
  const [rowSelection, setRowSelection] = useState({});
  
  const table = useReactTable({
    data,
    columns,
    onSortingChange: setSorting,
    onColumnFiltersChange: setColumnFilters,
    getCoreRowModel: getCoreRowModel(),
    getPaginationRowModel: getPaginationRowModel(),
    getSortedRowModel: getSortedRowModel(),
    getFilteredRowModel: getFilteredRowModel(),
    onColumnVisibilityChange: setColumnVisibility,
    onRowSelectionChange: setRowSelection,
    state: {
      sorting,
      columnFilters,
      columnVisibility,
      rowSelection,
    },
    initialState: {
      pagination: {
        pageSize,
      },
    },
  });
  
  return (
    <div className="w-full">
      <div className="flex items-center py-4 gap-2">
        {searchKey && (
          <Input
            placeholder={`Search by ${searchKey}...`}
            value={(table.getColumn(searchKey)?.getFilterValue() as string) ?? ''}
            onChange={(event) =>
              table.getColumn(searchKey)?.setFilterValue(event.target.value)
            }
            className="max-w-sm"
          />
        )}
        {showColumnVisibility && (
          <DropdownMenu>
            <DropdownMenuTrigger asChild>
              <Button variant="outline" className="ml-auto">
                Columns <ChevronDown className="ml-2 h-4 w-4" />
              </Button>
            </DropdownMenuTrigger>
            <DropdownMenuContent align="end">
              {table
                .getAllColumns()
                .filter((column) => column.getCanHide())
                .map((column) => {
                  return (
                    <DropdownMenuCheckboxItem
                      key={column.id}
                      className="capitalize"
                      checked={column.getIsVisible()}
                      onCheckedChange={(value) =>
                        column.toggleVisibility(!!value)
                      }
                    >
                      {column.id}
                    </DropdownMenuCheckboxItem>
                  );
                })}
            </DropdownMenuContent>
          </DropdownMenu>
        )}
      </div>
      <div className="rounded-md border">
        <Table>
          <TableHeader>
            {table.getHeaderGroups().map((headerGroup) => (
              <TableRow key={headerGroup.id}>
                {headerGroup.headers.map((header) => {
                  return (
                    <TableHead key={header.id}>
                      {header.isPlaceholder
                        ? null
                        : flexRender(
                            header.column.columnDef.header,
                            header.getContext()
                          )}
                    </TableHead>
                  );
                })}
              </TableRow>
            ))}
          </TableHeader>
          <TableBody>
            {table.getRowModel().rows?.length ? (
              table.getRowModel().rows.map((row) => (
                <TableRow
                  key={row.id}
                  data-state={row.getIsSelected() && 'selected'}
                >
                  {row.getVisibleCells().map((cell) => (
                    <TableCell key={cell.id}>
                      {flexRender(
                        cell.column.columnDef.cell,
                        cell.getContext()
                      )}
                    </TableCell>
                  ))}
                </TableRow>
              ))
            ) : (
              <TableRow>
                <TableCell
                  colSpan={columns.length}
                  className="h-24 text-center"
                >
                  No results.
                </TableCell>
              </TableRow>
            )}
          </TableBody>
        </Table>
      </div>
      {showPagination && (
        <div className="flex items-center justify-end space-x-2 py-4">
          <div className="flex-1 text-sm text-muted-foreground">
            {table.getFilteredSelectedRowModel().rows.length} of{' '}
            {table.getFilteredRowModel().rows.length} row(s) selected.
          </div>
          <div className="space-x-2">
            <Button
              variant="outline"
              size="sm"
              onClick={() => table.previousPage()}
              disabled={!table.getCanPreviousPage()}
            >
              Previous
            </Button>
            <Button
              variant="outline"
              size="sm"
              onClick={() => table.nextPage()}
              disabled={!table.getCanNextPage()}
            >
              Next
            </Button>
          </div>
        </div>
      )}
    </div>
  );
}
```

## Lambda Function Requirements

### Lambda Function Architecture Context
```typescript
// Lambda functions need to handle the following frontend requirements:

interface LambdaRequirements {
  // Performance Requirements
  performance: {
    coldStart: '<1s for critical paths',
    warmResponse: '<200ms p50, <500ms p95',
    timeout: '30s default, 60s for heavy operations',
    memory: '256MB default, up to 3GB for processing',
    concurrency: '1000 concurrent executions',
  };
  
  // Security Requirements
  security: {
    authentication: 'JWT validation required',
    authorization: 'RBAC with 11 admin types',
    encryption: 'All data encrypted in transit and at rest',
    cors: 'Configured for all EventfulHQ domains',
    rateLimit: 'By user, IP, and API key',
  };
  
  // Integration Requirements
  integration: {
    database: 'RDS Proxy for connection pooling',
    cache: 'ElastiCache Redis for session and data cache',
    queue: 'SQS for async processing',
    events: 'EventBridge for inter-service communication',
    storage: 'S3 for media and documents',
  };
  
  // Response Format
  responseFormat: {
    success: {
      status: 200,
      body: {
        data: 'T',
        meta: {
          requestId: 'string',
          timestamp: 'ISO 8601',
          version: 'string',
        },
      },
    },
    error: {
      status: 'number',
      body: {
        error: {
          code: 'string',
          message: 'string',
          details: 'object',
          requestId: 'string',
        },
      },
    },
    pagination: {
      data: 'T[]',
      meta: {
        total: 'number',
        page: 'number',
        limit: 'number',
        totalPages: 'number',
      },
    },
  };
  
  // Feature-Specific Requirements
  features: {
    whatsappAuth: {
      otpGeneration: '6-digit numeric',
      otpExpiry: '5 minutes',
      maxAttempts: 3,
      sessionManagement: 'Redis with TTL',
    },
    eventManagement: {
      imageProcessing: 'Resize, optimize, generate thumbnails',
      geoLocation: 'Store and query by coordinates',
      realtimeUpdates: 'WebSocket via API Gateway',
      bulkOperations: 'Batch processing up to 1000 items',
    },
    adminSystem: {
      auditLogging: 'All admin actions logged',
      permissionChecks: 'Granular resource-action validation',
      geographyFilters: 'Region/country/city level access',
      featureFlags: 'Dynamic microservice control',
    },
    developerPortal: {
      apiKeyGeneration: 'Secure random 32 chars',
      usageTracking: 'Per endpoint, per key metrics',
      rateLimiting: 'Configurable per API key',
      webhookDelivery: 'Retry with exponential backoff',
    },
  };
}

// Lambda function template
export const lambdaHandler = async (event: APIGatewayProxyEvent): Promise<APIGatewayProxyResult> => {
  // Request ID for tracing
  const requestId = event.requestContext.requestId;
  
  try {
    // 1. Parse and validate input
    const body = JSON.parse(event.body || '{}');
    const validated = await schema.validate(body);
    
    // 2. Authenticate
    const user = await authenticate(event.headers.authorization);
    
    // 3. Authorize
    await authorize(user, resource, action);
    
    // 4. Execute business logic
    const result = await businessLogic(validated, user);
    
    // 5. Format response
    return {
      statusCode: 200,
      headers: {
        'Content-Type': 'application/json',
        'X-Request-ID': requestId,
        'Cache-Control': 'no-cache',
      },
      body: JSON.stringify({
        data: result,
        meta: {
          requestId,
          timestamp: new Date().toISOString(),
          version: '1.0',
        },
      }),
    };
  } catch (error) {
    // Error handling
    return handleError(error, requestId);
  }
};
```

### Lambda Layer Requirements
```typescript
// Shared Lambda layers for common functionality

// Layer 1: Core utilities
export const coreLayer = {
  name: 'eventfulhq-core',
  description: 'Core utilities and helpers',
  compatibleRuntimes: ['nodejs18.x', 'nodejs20.x'],
  contents: [
    'auth/jwt-validator',
    'auth/rbac',
    'database/connection-pool',
    'cache/redis-client',
    'monitoring/metrics',
    'logging/structured-logger',
    'validation/schemas',
    'utils/date-time',
    'utils/crypto',
    'middleware/error-handler',
    'middleware/request-validator',
  ],
};

// Layer 2: AWS SDK clients
export const awsLayer = {
  name: 'eventfulhq-aws',
  description: 'Optimized AWS SDK clients',
  compatibleRuntimes: ['nodejs18.x', 'nodejs20.x'],
  contents: [
    '@aws-sdk/client-dynamodb',
    '@aws-sdk/client-s3',
    '@aws-sdk/client-sqs',
    '@aws-sdk/client-eventbridge',
    '@aws-sdk/client-ses',
    '@aws-sdk/client-secrets-manager',
    '@aws-sdk/rds-signer',
  ],
};

// Layer 3: Third-party integrations
export const integrationsLayer = {
  name: 'eventfulhq-integrations',
  description: 'Third-party service integrations',
  compatibleRuntimes: ['nodejs18.x', 'nodejs20.x'],
  contents: [
    'whatsapp-business-api',
    'stripe',
    'sendgrid/mail',
    'twilio',
    'openai',
    'google-maps-services-js',
    'libphonenumber-js',
  ],
};

## Multi-Domain Regional Routing

### Regional Domain Configuration
EventfulHQ supports multiple regional domains that automatically redirect users based on their geographic location while respecting user preferences.

#### Domain Mapping
- **Global**: eventfulhq.com (default)
- **Americas**: eventfulamerica.com (USA, Canada, Mexico, South America)
- **Arabia**: eventfularabia.com (GCC Countries: UAE, Saudi Arabia, Kuwait, Qatar, Bahrain, Oman)
- **India**: eventfulindia.com (India, Bangladesh, Sri Lanka, Nepal)
- **Europe**: eventfuleurope.com (EU Countries + UK)
- **Australia**: eventfulaustralia.com (Australia, New Zealand)

### Regional Routing Middleware
```typescript
// middleware-regional.ts
import { NextRequest, NextResponse } from 'next/server';

interface RegionalConfig {
  domain: string;
  countries: string[];
  defaultLocale: string;
  supportedLocales: string[];
}

const regionalConfigs: Record<string, RegionalConfig> = {
  america: {
    domain: 'eventfulamerica.com',
    countries: ['US', 'CA', 'MX', 'BR', 'AR', 'CL', 'CO', 'PE'],
    defaultLocale: 'en',
    supportedLocales: ['en', 'es', 'pt'],
  },
  arabia: {
    domain: 'eventfularabia.com',
    countries: ['AE', 'SA', 'KW', 'QA', 'BH', 'OM', 'EG', 'JO', 'LB'],
    defaultLocale: 'ar',
    supportedLocales: ['ar', 'en'],
  },
  india: {
    domain: 'eventfulindia.com',
    countries: ['IN', 'BD', 'LK', 'NP', 'PK'],
    defaultLocale: 'hi',
    supportedLocales: ['hi', 'en'],
  },
  europe: {
    domain: 'eventfuleurope.com',
    countries: ['GB', 'FR', 'DE', 'IT', 'ES', 'NL', 'BE', 'PL', 'PT', 'SE', 'DK', 'FI', 'NO', 'IE', 'AT', 'CH'],
    defaultLocale: 'en',
    supportedLocales: ['en', 'fr', 'es', 'pt', 'ru'],
  },
  australia: {
    domain: 'eventfulaustralia.com',
    countries: ['AU', 'NZ'],
    defaultLocale: 'en',
    supportedLocales: ['en'],
  },
};

// Reverse mapping for quick lookup
const countryToRegion: Record<string, string> = {};
Object.entries(regionalConfigs).forEach(([region, config]) => {
  config.countries.forEach(country => {
    countryToRegion[country] = region;
  });
});

export async function regionalMiddleware(request: NextRequest) {
  const hostname = request.headers.get('host') || '';
  const pathname = request.nextUrl.pathname;
  
  // Get user's country from geo data
  const userCountry = request.geo?.country || 
                     request.headers.get('x-vercel-ip-country') || 
                     '';
  
  // Check if user has a saved regional preference
  const savedRegion = request.cookies.get('preferred-region')?.value;
  const savedDomain = request.cookies.get('preferred-domain')?.value;
  
  // If user has saved preferences, respect them
  if (savedDomain && hostname !== savedDomain) {
    const url = new URL(request.url);
    url.hostname = savedDomain;
    return NextResponse.redirect(url);
  }
  
  // Determine the appropriate region for the user
  const userRegion = countryToRegion[userCountry] || 'global';
  const targetConfig = regionalConfigs[userRegion];
  
  // If current domain doesn't match user's region and no saved preference
  if (!savedRegion && targetConfig && hostname !== targetConfig.domain && hostname === 'eventfulhq.com') {
    const url = new URL(request.url);
    url.hostname = targetConfig.domain;
    
    const response = NextResponse.redirect(url);
    
    // Save the auto-detected region (but as a soft preference)
    response.cookies.set('detected-region', userRegion, {
      maxAge: 60 * 60 * 24 * 30, // 30 days
      sameSite: 'lax',
      secure: process.env.NODE_ENV === 'production',
    });
    
    return response;
  }
  
  return NextResponse.next();
}
```

### Region Selector Component
```typescript
// components/features/regional/region-selector.tsx
'use client';

import { useState, useEffect } from 'react';
import { useRouter } from 'next/navigation';
import {
  Dialog,
  DialogContent,
  DialogDescription,
  DialogHeader,
  DialogTitle,
} from '@/components/ui/dialog';
import {
  Select,
  SelectContent,
  SelectItem,
  SelectTrigger,
  SelectValue,
} from '@/components/ui/select';
import { Button } from '@/components/ui/button';
import { GlobeIcon } from 'lucide-react';
import { toast } from 'sonner';

interface Region {
  id: string;
  name: string;
  domain: string;
  flag: string;
  description: string;
}

const regions: Region[] = [
  {
    id: 'global',
    name: 'Global',
    domain: 'eventfulhq.com',
    flag: '🌍',
    description: 'Access events worldwide',
  },
  {
    id: 'america',
    name: 'Americas',
    domain: 'eventfulamerica.com',
    flag: '🇺🇸',
    description: 'Events in North & South America',
  },
  {
    id: 'arabia',
    name: 'Arabia',
    domain: 'eventfularabia.com',
    flag: '🇸🇦',
    description: 'Events in GCC & Middle East',
  },
  {
    id: 'india',
    name: 'India',
    domain: 'eventfulindia.com',
    flag: '🇮🇳',
    description: 'Events in India & South Asia',
  },
  {
    id: 'europe',
    name: 'Europe',
    domain: 'eventfuleurope.com',
    flag: '🇪🇺',
    description: 'Events across Europe',
  },
  {
    id: 'australia',
    name: 'Australia',
    domain: 'eventfulaustralia.com',
    flag: '🇦🇺',
    description: 'Events in Australia & Oceania',
  },
];

export function RegionSelector() {
  const router = useRouter();
  const [showDialog, setShowDialog] = useState(false);
  const [selectedRegion, setSelectedRegion] = useState<string>('global');
  const [currentDomain, setCurrentDomain] = useState<string>('');

  useEffect(() => {
    // Get current domain
    const domain = window.location.hostname;
    setCurrentDomain(domain);
    
    // Find current region
    const currentRegion = regions.find(r => r.domain === domain);
    if (currentRegion) {
      setSelectedRegion(currentRegion.id);
    }
    
    // Check if this is first visit without preference
    const hasPreference = document.cookie.includes('preferred-region');
    const detectedRegion = document.cookie.includes('detected-region');
    
    if (!hasPreference && detectedRegion && domain === 'eventfulhq.com') {
      setShowDialog(true);
    }
  }, []);

  const handleRegionChange = (regionId: string) => {
    const region = regions.find(r => r.id === regionId);
    if (!region) return;
    
    // Save preference
    document.cookie = `preferred-region=${regionId};max-age=31536000;path=/;domain=.eventfulhq.com;samesite=lax${
      process.env.NODE_ENV === 'production' ? ';secure' : ''
    }`;
    document.cookie = `preferred-domain=${region.domain};max-age=31536000;path=/;domain=.eventfulhq.com;samesite=lax${
      process.env.NODE_ENV === 'production' ? ';secure' : ''
    }`;
    
    // Redirect to new domain
    if (region.domain !== currentDomain) {
      window.location.href = `https://${region.domain}${window.location.pathname}`;
    }
    
    toast.success(`Switched to ${region.name} region`);
  };

  const currentRegion = regions.find(r => r.id === selectedRegion);

  return (
    <>
      <Button
        variant="ghost"
        size="sm"
        onClick={() => setShowDialog(true)}
        className="flex items-center gap-2"
      >
        <GlobeIcon className="h-4 w-4" />
        <span className="hidden sm:inline">
          {currentRegion?.flag} {currentRegion?.name}
        </span>
        <span className="sm:hidden">{currentRegion?.flag}</span>
      </Button>

      <Dialog open={showDialog} onOpenChange={setShowDialog}>
        <DialogContent className="sm:max-w-md">
          <DialogHeader>
            <DialogTitle>Select Your Region</DialogTitle>
            <DialogDescription>
              Choose your preferred region to see relevant events and content
            </DialogDescription>
          </DialogHeader>
          
          <div className="space-y-4 py-4">
            <Select value={selectedRegion} onValueChange={setSelectedRegion}>
              <SelectTrigger>
                <SelectValue />
              </SelectTrigger>
              <SelectContent>
                {regions.map((region) => (
                  <SelectItem key={region.id} value={region.id}>
                    <div className="flex items-center gap-3">
                      <span className="text-2xl">{region.flag}</span>
                      <div>
                        <div className="font-medium">{region.name}</div>
                        <div className="text-xs text-muted-foreground">
                          {region.description}
                        </div>
                      </div>
                    </div>
                  </SelectItem>
                ))}
              </SelectContent>
            </Select>
            
            <div className="text-sm text-muted-foreground">
              Your selection will be saved and you'll be redirected to {regions.find(r => r.id === selectedRegion)?.domain}
            </div>
            
            <div className="flex justify-end gap-2">
              <Button
                variant="outline"
                onClick={() => setShowDialog(false)}
              >
                Cancel
              </Button>
              <Button
                onClick={() => {
                  handleRegionChange(selectedRegion);
                  setShowDialog(false);
                }}
              >
                Save & Continue
              </Button>
            </div>
          </div>
        </DialogContent>
      </Dialog>
    </>
  );
}
```

### Next.js Configuration for Multi-Domain
```javascript
// next.config.mjs
const domains = [
  'eventfulhq.com',
  'eventfulamerica.com',
  'eventfularabia.com',
  'eventfulindia.com',
  'eventfuleurope.com',
  'eventfulaustralia.com',
];

/** @type {import('next').NextConfig} */
const nextConfig = {
  experimental: {
    appDir: true,
  },
  images: {
    domains: [...domains, 'cdn.eventfulhq.com'],
    remotePatterns: [
      {
        protocol: 'https',
        hostname: '**.eventfulhq.com',
      },
    ],
  },
  async headers() {
    return [
      {
        source: '/:path*',
        headers: [
          {
            key: 'X-Frame-Options',
            value: 'DENY',
          },
          {
            key: 'Content-Security-Policy',
            value: `frame-ancestors 'none'; default-src 'self' ${domains.join(' ')};`,
          },
        ],
      },
    ];
  },
  async rewrites() {
    return {
      beforeFiles: [
        // API rewrites for regional domains
        ...domains.map((domain) => ({
          source: '/api/:path*',
          destination: `https://api.${domain}/api/:path*`,
          has: [
            {
              type: 'host',
              value: domain,
            },
          ],
        })),
      ],
    };
  },
  async redirects() {
    return [
      {
        source: '/events',
        destination: '/en/events',
        permanent: false,
      },
    ];
  },
};

export default nextConfig;
```

## SuperAdmin Access System

### Admin User Types (11 Types)
```typescript
// types/admin.types.ts
export enum AdminType {
  SUPER_ADMIN = 'SUPER_ADMIN',                 // Full system access
  PLATFORM_ADMIN = 'PLATFORM_ADMIN',           // Platform-wide management
  REGIONAL_ADMIN = 'REGIONAL_ADMIN',           // Regional management
  COUNTRY_ADMIN = 'COUNTRY_ADMIN',             // Country-level management
  CITY_ADMIN = 'CITY_ADMIN',                   // City-level management
  CATEGORY_ADMIN = 'CATEGORY_ADMIN',           // Event category management
  CONTENT_ADMIN = 'CONTENT_ADMIN',             // Content moderation
  SUPPORT_ADMIN = 'SUPPORT_ADMIN',             // Customer support
  FINANCE_ADMIN = 'FINANCE_ADMIN',             // Financial operations
  MARKETING_ADMIN = 'MARKETING_ADMIN',         // Marketing campaigns
  DEVELOPER_ADMIN = 'DEVELOPER_ADMIN',         // API and developer relations
}

export interface AdminUser {
  id: string;
  email: string;
  phoneNumber: string;
  name: string;
  type: AdminType;
  permissions: Permission[];
  geographyAccess: GeographyAccess[];
  featureAccess: FeatureAccess[];
  isActive: boolean;
  isTwoFactorEnabled: boolean;
  lastLogin: Date;
  createdAt: Date;
  createdBy: string;
}

export interface Permission {
  resource: string;      // Microservice name as AppFeature
  actions: Action[];     // Read, Write, Delete, etc.
  conditions?: Record<string, any>;
}

export interface GeographyAccess {
  type: 'GLOBAL' | 'REGION' | 'COUNTRY' | 'CITY';
  identifier: string;    // e.g., 'US', 'New York', 'ASIA'
  permissions: Action[];
}

export interface FeatureAccess {
  microservice: string;  // All 30+ microservices
  permissions: Action[];
  limits?: Record<string, number>;
}

export enum Action {
  READ = 'READ',
  WRITE = 'WRITE',
  UPDATE = 'UPDATE',
  DELETE = 'DELETE',
  PUBLISH = 'PUBLISH',
  EXPORT = 'EXPORT',
  CONFIGURE = 'CONFIGURE',
}
```

### SuperAdmin Dashboard Components
```typescript
// components/features/admin/super-admin-dashboard.tsx
'use client';

import { useState } from 'react';
import { useTranslations } from 'next-intl';
import { Tabs, TabsContent, TabsList, TabsTrigger } from '@/components/ui/tabs';
import { Card, CardContent, CardDescription, CardHeader, CardTitle } from '@/components/ui/card';
import { AdminManagement } from './admin-management';
import { FeatureControl } from './feature-control';
import { GeographyControl } from './geography-control';
import { PermissionMatrix } from './permission-matrix';
import { SystemConfiguration } from './system-configuration';
import { AuditLogs } from './audit-logs';
import { SystemMonitoring } from './system-monitoring';
import { Shield, Users, Globe, Settings, Activity, FileText, Map } from 'lucide-react';

export function SuperAdminDashboard() {
  const t = useTranslations('admin');
  const [activeTab, setActiveTab] = useState('admins');

  return (
    <div className="space-y-6">
      <div>
        <h1 className="text-3xl font-bold">{t('superAdmin.title')}</h1>
        <p className="text-muted-foreground">{t('superAdmin.description')}</p>
      </div>

      <Tabs value={activeTab} onValueChange={setActiveTab} className="space-y-4">
        <TabsList className="grid w-full grid-cols-7">
          <TabsTrigger value="admins" className="flex items-center gap-2">
            <Users className="h-4 w-4" />
            <span className="hidden sm:inline">{t('tabs.admins')}</span>
          </TabsTrigger>
          <TabsTrigger value="features" className="flex items-center gap-2">
            <Shield className="h-4 w-4" />
            <span className="hidden sm:inline">{t('tabs.features')}</span>
          </TabsTrigger>
          <TabsTrigger value="geography" className="flex items-center gap-2">
            <Globe className="h-4 w-4" />
            <span className="hidden sm:inline">{t('tabs.geography')}</span>
          </TabsTrigger>
          <TabsTrigger value="permissions" className="flex items-center gap-2">
            <Map className="h-4 w-4" />
            <span className="hidden sm:inline">{t('tabs.permissions')}</span>
          </TabsTrigger>
          <TabsTrigger value="config" className="flex items-center gap-2">
            <Settings className="h-4 w-4" />
            <span className="hidden sm:inline">{t('tabs.config')}</span>
          </TabsTrigger>
          <TabsTrigger value="monitoring" className="flex items-center gap-2">
            <Activity className="h-4 w-4" />
            <span className="hidden sm:inline">{t('tabs.monitoring')}</span>
          </TabsTrigger>
          <TabsTrigger value="audit" className="flex items-center gap-2">
            <FileText className="h-4 w-4" />
            <span className="hidden sm:inline">{t('tabs.audit')}</span>
          </TabsTrigger>
        </TabsList>

        <TabsContent value="admins" className="space-y-4">
          <AdminManagement />
        </TabsContent>

        <TabsContent value="features" className="space-y-4">
          <FeatureControl />
        </TabsContent>

        <TabsContent value="geography" className="space-y-4">
          <GeographyControl />
        </TabsContent>

        <TabsContent value="permissions" className="space-y-4">
          <PermissionMatrix />
        </TabsContent>

        <TabsContent value="config" className="space-y-4">
          <SystemConfiguration />
        </TabsContent>

        <TabsContent value="monitoring" className="space-y-4">
          <SystemMonitoring />
        </TabsContent>

        <TabsContent value="audit" className="space-y-4">
          <AuditLogs />
        </TabsContent>
      </Tabs>
    </div>
  );
}
```

### Admin Management Interface
```typescript
// components/features/admin/admin-management.tsx
'use client';

import { useState } from 'react';
import { useQuery, useMutation } from '@tanstack/react-query';
import { DataTable } from '@/components/ui/data-table';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import {
  Select,
  SelectContent,
  SelectItem,
  SelectTrigger,
  SelectValue,
} from '@/components/ui/select';
import { Badge } from '@/components/ui/badge';
import {
  Dialog,
  DialogContent,
  DialogDescription,
  DialogHeader,
  DialogTitle,
  DialogTrigger,
} from '@/components/ui/dialog';
import { AdminType } from '@/types/admin.types';
import { adminService } from '@/services/admin-service';
import { CreateAdminForm } from './create-admin-form';
import { PermissionEditor } from './permission-editor';
import { Plus, Edit, Trash, Shield, Key } from 'lucide-react';
import { toast } from 'sonner';

export function AdminManagement() {
  const [search, setSearch] = useState('');
  const [typeFilter, setTypeFilter] = useState<AdminType | 'ALL'>('ALL');
  const [showCreateDialog, setShowCreateDialog] = useState(false);
  const [selectedAdmin, setSelectedAdmin] = useState(null);

  const { data: admins, isLoading } = useQuery({
    queryKey: ['admins', search, typeFilter],
    queryFn: () => adminService.getAdmins({ search, type: typeFilter }),
  });

  const { mutate: deleteAdmin } = useMutation({
    mutationFn: adminService.deleteAdmin,
    onSuccess: () => {
      toast.success('Admin deleted successfully');
      queryClient.invalidateQueries({ queryKey: ['admins'] });
    },
  });

  const { mutate: toggleTwoFactor } = useMutation({
    mutationFn: adminService.toggleTwoFactor,
    onSuccess: () => {
      toast.success('Two-factor authentication updated');
      queryClient.invalidateQueries({ queryKey: ['admins'] });
    },
  });

  const columns = [
    {
      accessorKey: 'name',
      header: 'Name',
      cell: ({ row }) => (
        <div>
          <p className="font-medium">{row.original.name}</p>
          <p className="text-sm text-muted-foreground">{row.original.email}</p>
        </div>
      ),
    },
    {
      accessorKey: 'type',
      header: 'Admin Type',
      cell: ({ row }) => (
        <Badge variant={row.original.type === 'SUPER_ADMIN' ? 'destructive' : 'default'}>
          {row.original.type.replace('_', ' ')}
        </Badge>
      ),
    },
    {
      accessorKey: 'geographyAccess',
      header: 'Geography Access',
      cell: ({ row }) => {
        const access = row.original.geographyAccess;
        if (access.some(a => a.type === 'GLOBAL')) {
          return <Badge variant="outline">Global</Badge>;
        }
        return (
          <div className="flex flex-wrap gap-1">
            {access.slice(0, 3).map((geo, idx) => (
              <Badge key={idx} variant="secondary" className="text-xs">
                {geo.identifier}
              </Badge>
            ))}
            {access.length > 3 && (
              <Badge variant="secondary" className="text-xs">
                +{access.length - 3}
              </Badge>
            )}
          </div>
        );
      },
    },
    {
      accessorKey: 'featureAccess',
      header: 'Features',
      cell: ({ row }) => {
        const features = row.original.featureAccess;
        return (
          <div className="flex flex-wrap gap-1">
            {features.slice(0, 3).map((feature, idx) => (
              <Badge key={idx} variant="outline" className="text-xs">
                {feature.microservice}
              </Badge>
            ))}
            {features.length > 3 && (
              <Badge variant="outline" className="text-xs">
                +{features.length - 3}
              </Badge>
            )}
          </div>
        );
      },
    },
    {
      accessorKey: 'status',
      header: 'Status',
      cell: ({ row }) => (
        <div className="flex items-center gap-2">
          <Badge variant={row.original.isActive ? 'success' : 'destructive'}>
            {row.original.isActive ? 'Active' : 'Inactive'}
          </Badge>
          {row.original.isTwoFactorEnabled && (
            <Shield className="h-4 w-4 text-green-500" />
          )}
        </div>
      ),
    },
    {
      accessorKey: 'lastLogin',
      header: 'Last Login',
      cell: ({ row }) => (
        <span className="text-sm text-muted-foreground">
          {new Date(row.original.lastLogin).toLocaleDateString()}
        </span>
      ),
    },
    {
      id: 'actions',
      cell: ({ row }) => (
        <div className="flex items-center gap-2">
          <Button
            variant="ghost"
            size="icon"
            onClick={() => setSelectedAdmin(row.original)}
          >
            <Edit className="h-4 w-4" />
          </Button>
          <Button
            variant="ghost"
            size="icon"
            onClick={() => toggleTwoFactor(row.original.id)}
          >
            <Key className="h-4 w-4" />
          </Button>
          <Button
            variant="ghost"
            size="icon"
            onClick={() => deleteAdmin(row.original.id)}
            disabled={row.original.type === 'SUPER_ADMIN'}
          >
            <Trash className="h-4 w-4" />
          </Button>
        </div>
      ),
    },
  ];

  return (
    <div className="space-y-4">
      <div className="flex items-center justify-between">
        <div className="flex items-center gap-4">
          <Input
            placeholder="Search admins..."
            value={search}
            onChange={(e) => setSearch(e.target.value)}
            className="w-64"
          />
          <Select value={typeFilter} onValueChange={setTypeFilter}>
            <SelectTrigger className="w-48">
              <SelectValue />
            </SelectTrigger>
            <SelectContent>
              <SelectItem value="ALL">All Types</SelectItem>
              {Object.values(AdminType).map((type) => (
                <SelectItem key={type} value={type}>
                  {type.replace(/_/g, ' ')}
                </SelectItem>
              ))}
            </SelectContent>
          </Select>
        </div>
        <Dialog open={showCreateDialog} onOpenChange={setShowCreateDialog}>
          <DialogTrigger asChild>
            <Button>
              <Plus className="h-4 w-4 mr-2" />
              Add Admin
            </Button>
          </DialogTrigger>
          <DialogContent className="max-w-2xl">
            <DialogHeader>
              <DialogTitle>Create New Admin</DialogTitle>
              <DialogDescription>
                Add a new admin user with specific permissions and access controls
              </DialogDescription>
            </DialogHeader>
            <CreateAdminForm onSuccess={() => setShowCreateDialog(false)} />
          </DialogContent>
        </Dialog>
      </div>

      <DataTable
        columns={columns}
        data={admins?.data || []}
        loading={isLoading}
      />

      {selectedAdmin && (
        <PermissionEditor
          admin={selectedAdmin}
          open={!!selectedAdmin}
          onOpenChange={() => setSelectedAdmin(null)}
        />
      )}
    </div>
  );
}
```

### Feature Control (Microservices as AppFeatures)
```typescript
// components/features/admin/feature-control.tsx
'use client';

import { useState } from 'react';
import { useQuery, useMutation } from '@tanstack/react-query';
import { Card, CardContent, CardDescription, CardHeader, CardTitle } from '@/components/ui/card';
import { Switch } from '@/components/ui/switch';
import { Button } from '@/components/ui/button';
import { Badge } from '@/components/ui/badge';
import { Slider } from '@/components/ui/slider';
import { Input } from '@/components/ui/input';
import { Label } from '@/components/ui/label';
import {
  Accordion,
  AccordionContent,
  AccordionItem,
  AccordionTrigger,
} from '@/components/ui/accordion';
import { adminService } from '@/services/admin-service';
import { Microservice } from '@/types/admin.types';
import { Settings, AlertCircle, Activity } from 'lucide-react';
import { toast } from 'sonner';

// All 30+ microservices
const MICROSERVICES = [
  { id: 'event-service', name: 'Event Service', category: 'Core' },
  { id: 'user-service', name: 'User Service', category: 'Core' },
  { id: 'whatsapp-service', name: 'WhatsApp Service', category: 'Communication' },
  { id: 'rfp-service', name: 'RFP Service', category: 'Marketplace' },
  { id: 'analytics-service', name: 'Analytics Service', category: 'Analytics' },
  { id: 'payment-service', name: 'Payment Service', category: 'Financial' },
  { id: 'notification-service', name: 'Notification Service', category: 'Communication' },
  { id: 'search-service', name: 'Search Service', category: 'Core' },
  { id: 'recommendation-service', name: 'Recommendation Service', category: 'AI' },
  { id: 'media-service', name: 'Media Service', category: 'Content' },
  { id: 'chat-service', name: 'Chat Service', category: 'Communication' },
  { id: 'booking-service', name: 'Booking Service', category: 'Core' },
  { id: 'review-service', name: 'Review Service', category: 'Social' },
  { id: 'social-service', name: 'Social Service', category: 'Social' },
  { id: 'email-service', name: 'Email Service', category: 'Communication' },
  { id: 'sms-service', name: 'SMS Service', category: 'Communication' },
  { id: 'calendar-service', name: 'Calendar Service', category: 'Core' },
  { id: 'ticketing-service', name: 'Ticketing Service', category: 'Core' },
  { id: 'venue-service', name: 'Venue Service', category: 'Marketplace' },
  { id: 'artist-service', name: 'Artist Service', category: 'Marketplace' },
  { id: 'sponsor-service', name: 'Sponsor Service', category: 'Marketplace' },
  { id: 'streaming-service', name: 'Streaming Service', category: 'Content' },
  { id: 'translation-service', name: 'Translation Service', category: 'AI' },
  { id: 'moderation-service', name: 'Moderation Service', category: 'AI' },
  { id: 'fraud-service', name: 'Fraud Detection', category: 'Security' },
  { id: 'audit-service', name: 'Audit Service', category: 'Security' },
  { id: 'backup-service', name: 'Backup Service', category: 'Infrastructure' },
  { id: 'monitoring-service', name: 'Monitoring Service', category: 'Infrastructure' },
  { id: 'queue-service', name: 'Queue Service', category: 'Infrastructure' },
  { id: 'cache-service', name: 'Cache Service', category: 'Infrastructure' },
];

export function FeatureControl() {
  const [selectedCategory, setSelectedCategory] = useState<string>('all');
  
  const { data: features, isLoading } = useQuery({
    queryKey: ['features'],
    queryFn: adminService.getFeatures,
  });

  const { mutate: updateFeature } = useMutation({
    mutationFn: adminService.updateFeature,
    onSuccess: () => {
      toast.success('Feature updated successfully');
      queryClient.invalidateQueries({ queryKey: ['features'] });
    },
  });

  const categories = ['all', ...new Set(MICROSERVICES.map(m => m.category))];
  const filteredServices = selectedCategory === 'all' 
    ? MICROSERVICES 
    : MICROSERVICES.filter(m => m.category === selectedCategory);

  return (
    <div className="space-y-6">
      <div className="flex items-center gap-4">
        {categories.map((category) => (
          <Button
            key={category}
            variant={selectedCategory === category ? 'default' : 'outline'}
            size="sm"
            onClick={() => setSelectedCategory(category)}
          >
            {category.charAt(0).toUpperCase() + category.slice(1)}
          </Button>
        ))}
      </div>

      <div className="grid gap-4 md:grid-cols-2 lg:grid-cols-3">
        {filteredServices.map((service) => {
          const feature = features?.find(f => f.id === service.id);
          
          return (
            <Card key={service.id}>
              <CardHeader className="pb-3">
                <div className="flex items-center justify-between">
                  <CardTitle className="text-base">{service.name}</CardTitle>
                  <Switch
                    checked={feature?.isEnabled || false}
                    onCheckedChange={(enabled) => 
                      updateFeature({ id: service.id, isEnabled: enabled })
                    }
                  />
                </div>
                <CardDescription className="text-xs">
                  {service.category} • {service.id}
                </CardDescription>
              </CardHeader>
              <CardContent className="space-y-4">
                <div className="flex items-center justify-between text-sm">
                  <span className="text-muted-foreground">Status</span>
                  <Badge variant={feature?.status === 'healthy' ? 'success' : 'destructive'}>
                    {feature?.status || 'unknown'}
                  </Badge>
                </div>
                
                <div className="space-y-2">
                  <Label className="text-xs">Rate Limit (req/min)</Label>
                  <div className="flex items-center gap-2">
                    <Slider
                      value={[feature?.rateLimit || 1000]}
                      onValueChange={([value]) => 
                        updateFeature({ id: service.id, rateLimit: value })
                      }
                      max={10000}
                      step={100}
                      className="flex-1"
                    />
                    <span className="text-xs w-12 text-right">
                      {feature?.rateLimit || 1000}
                    </span>
                  </div>
                </div>

                <Accordion type="single" collapsible>
                  <AccordionItem value="config">
                    <AccordionTrigger className="text-xs">
                      Advanced Configuration
                    </AccordionTrigger>
                    <AccordionContent className="space-y-3">
                      <div className="space-y-2">
                        <Label className="text-xs">Timeout (ms)</Label>
                        <Input
                          type="number"
                          value={feature?.timeout || 30000}
                          onChange={(e) => 
                            updateFeature({ 
                              id: service.id, 
                              timeout: parseInt(e.target.value) 
                            })
                          }
                        />
                      </div>
                      
                      <div className="space-y-2">
                        <Label className="text-xs">Max Retries</Label>
                        <Input
                          type="number"
                          value={feature?.maxRetries || 3}
                          onChange={(e) => 
                            updateFeature({ 
                              id: service.id, 
                              maxRetries: parseInt(e.target.value) 
                            })
                          }
                        />
                      </div>

                      <div className="space-y-2">
                        <Label className="text-xs">Feature Flags</Label>
                        <div className="space-y-1">
                          {feature?.flags?.map((flag) => (
                            <div key={flag.name} className="flex items-center justify-between">
                              <span className="text-xs">{flag.name}</span>
                              <Switch
                                checked={flag.enabled}
                                onCheckedChange={(enabled) => 
                                  updateFeature({ 
                                    id: service.id, 
                                    flags: feature.flags.map(f => 
                                      f.name === flag.name ? { ...f, enabled } : f
                                    )
                                  })
                                }
                              />
                            </div>
                          ))}
                        </div>
                      </div>
                    </AccordionContent>
                  </AccordionItem>
                </Accordion>
              </CardContent>
            </Card>
          );
        })}
      </div>
    </div>
  );
}
```

### Permission Matrix Component
```typescript
// components/features/admin/permission-matrix.tsx
'use client';

import { useState } from 'react';
import { useQuery } from '@tanstack/react-query';
import {
  Table,
  TableBody,
  TableCell,
  TableHead,
  TableHeader,
  TableRow,
} from '@/components/ui/table';
import { Checkbox } from '@/components/ui/checkbox';
import { Badge } from '@/components/ui/badge';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import {
  Select,
  SelectContent,
  SelectItem,
  SelectTrigger,
  SelectValue,
} from '@/components/ui/select';
import { adminService } from '@/services/admin-service';
import { AdminType, Action } from '@/types/admin.types';
import { Save, Download } from 'lucide-react';
import { toast } from 'sonner';

export function PermissionMatrix() {
  const [search, setSearch] = useState('');
  const [selectedAdminType, setSelectedAdminType] = useState<AdminType>(AdminType.PLATFORM_ADMIN);
  const [modifiedPermissions, setModifiedPermissions] = useState<Record<string, any>>({});

  const { data: permissions, isLoading } = useQuery({
    queryKey: ['permission-matrix', selectedAdminType],
    queryFn: () => adminService.getPermissionMatrix(selectedAdminType),
  });

  const handlePermissionChange = (feature: string, action: Action, enabled: boolean) => {
    setModifiedPermissions(prev => ({
      ...prev,
      [`${feature}-${action}`]: enabled,
    }));
  };

  const handleSave = async () => {
    try {
      await adminService.updatePermissionMatrix(selectedAdminType, modifiedPermissions);
      toast.success('Permissions updated successfully');
      setModifiedPermissions({});
    } catch (error) {
      toast.error('Failed to update permissions');
    }
  };

  const handleExport = () => {
    const data = permissions?.map(p => ({
      Feature: p.feature,
      ...Object.fromEntries(
        Object.values(Action).map(action => [
          action,
          p.permissions.includes(action) ? 'Yes' : 'No'
        ])
      ),
    }));
    
    const csv = [
      Object.keys(data[0]).join(','),
      ...data.map(row => Object.values(row).join(','))
    ].join('\n');
    
    const blob = new Blob([csv], { type: 'text/csv' });
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = `permissions-${selectedAdminType}.csv`;
    a.click();
  };

  const filteredPermissions = permissions?.filter(p => 
    p.feature.toLowerCase().includes(search.toLowerCase())
  );

  return (
    <div className="space-y-4">
      <div className="flex items-center justify-between">
        <div className="flex items-center gap-4">
          <Input
            placeholder="Search features..."
            value={search}
            onChange={(e) => setSearch(e.target.value)}
            className="w-64"
          />
          <Select value={selectedAdminType} onValueChange={setSelectedAdminType}>
            <SelectTrigger className="w-48">
              <SelectValue />
            </SelectTrigger>
            <SelectContent>
              {Object.values(AdminType).map((type) => (
                <SelectItem key={type} value={type}>
                  {type.replace(/_/g, ' ')}
                </SelectItem>
              ))}
            </SelectContent>
          </Select>
        </div>
        <div className="flex items-center gap-2">
          <Button
            variant="outline"
            size="sm"
            onClick={handleExport}
          >
            <Download className="h-4 w-4 mr-2" />
            Export CSV
          </Button>
          <Button
            size="sm"
            onClick={handleSave}
            disabled={Object.keys(modifiedPermissions).length === 0}
          >
            <Save className="h-4 w-4 mr-2" />
            Save Changes
          </Button>
        </div>
      </div>

      <div className="border rounded-lg">
        <Table>
          <TableHeader>
            <TableRow>
              <TableHead className="w-64">Feature / Microservice</TableHead>
              {Object.values(Action).map((action) => (
                <TableHead key={action} className="text-center">
                  {action}
                </TableHead>
              ))}
            </TableRow>
          </TableHeader>
          <TableBody>
            {filteredPermissions?.map((permission) => (
              <TableRow key={permission.feature}>
                <TableCell className="font-medium">
                  <div>
                    <p>{permission.featureName}</p>
                    <p className="text-xs text-muted-foreground">{permission.feature}</p>
                  </div>
                </TableCell>
                {Object.values(Action).map((action) => {
                  const key = `${permission.feature}-${action}`;
                  const isEnabled = modifiedPermissions[key] ?? 
                    permission.permissions.includes(action);
                  const isModified = key in modifiedPermissions;
                  
                  return (
                    <TableCell key={action} className="text-center">
                      <div className="flex items-center justify-center">
                        <Checkbox
                          checked={isEnabled}
                          onCheckedChange={(checked) => 
                            handlePermissionChange(permission.feature, action, !!checked)
                          }
                        />
                        {isModified && (
                          <Badge variant="outline" className="ml-2 text-xs">
                            Modified
                          </Badge>
                        )}
                      </div>
                    </TableCell>
                  );
                })}
              </TableRow>
            ))}
          </TableBody>
        </Table>
      </div>
    </div>
  );
}
```

## Developer Portal & API Documentation

### Developer Portal Homepage
```typescript
// app/[locale]/(developers)/developers/page.tsx
import { Metadata } from 'next';
import { DeveloperDashboard } from '@/components/features/developers/developer-dashboard';
import { APIQuickStart } from '@/components/features/developers/api-quickstart';
import { APIStats } from '@/components/features/developers/api-stats';

export const metadata: Metadata = {
  title: 'EventfulHQ Developer Portal',
  description: 'Build amazing applications with EventfulHQ APIs',
};

export default function DevelopersPage() {
  return (
    <div className="container py-8 space-y-8">
      <div className="text-center space-y-4">
        <h1 className="text-4xl font-bold">EventfulHQ Developer Portal</h1>
        <p className="text-xl text-muted-foreground max-w-2xl mx-auto">
          Access our comprehensive APIs to build innovative event management solutions
        </p>
      </div>

      <APIStats />
      <APIQuickStart />
      <DeveloperDashboard />
    </div>
  );
}
```

### API Documentation Component
```typescript
// components/features/developers/api-documentation.tsx
'use client';

import { useState, useEffect } from 'react';
import dynamic from 'next/dynamic';
import { Tabs, TabsContent, TabsList, TabsTrigger } from '@/components/ui/tabs';
import { Card, CardContent, CardDescription, CardHeader, CardTitle } from '@/components/ui/card';
import { Button } from '@/components/ui/button';
import { Badge } from '@/components/ui/badge';
import { CodeBlock } from './code-block';
import { APIPlayground } from './api-playground';
import { useQuery } from '@tanstack/react-query';
import { developerService } from '@/services/developer-service';
import { Book, Code, Play, Download } from 'lucide-react';

// Dynamic import for Swagger UI
const SwaggerUI = dynamic(() => import('swagger-ui-react'), { 
  ssr: false,
  loading: () => <div>Loading API documentation...</div>
});

// Import Swagger UI CSS
import 'swagger-ui-react/swagger-ui.css';

export function APIDocumentation() {
  const [selectedService, setSelectedService] = useState('event-service');
  const [selectedLanguage, setSelectedLanguage] = useState('javascript');

  const { data: apiSpec } = useQuery({
    queryKey: ['api-spec', selectedService],
    queryFn: () => developerService.getAPISpec(selectedService),
  });

  const { data: sdks } = useQuery({
    queryKey: ['sdks'],
    queryFn: developerService.getSDKs,
  });

  const services = [
    { id: 'event-service', name: 'Event API', version: 'v1' },
    { id: 'user-service', name: 'User API', version: 'v1' },
    { id: 'rfp-service', name: 'RFP API', version: 'v1' },
    { id: 'analytics-service', name: 'Analytics API', version: 'v1' },
    { id: 'whatsapp-service', name: 'WhatsApp API', version: 'v1' },
    // ... other services
  ];

  const codeSamples = {
    javascript: `// Install the SDK
npm install @eventfulhq/sdk

// Initialize the client
import { EventfulHQ } from '@eventfulhq/sdk';

const client = new EventfulHQ({
  apiKey: 'your-api-key',
  region: 'global' // or 'america', 'arabia', 'india', 'europe', 'australia'
});

// Create an event
const event = await client.events.create({
  title: 'Summer Music Festival',
  description: 'A fantastic outdoor music experience',
  date: '2025-07-15T18:00:00Z',
  venue: {
    name: 'Central Park',
    address: 'New York, NY',
    coordinates: { lat: 40.7829, lng: -73.9654 }
  },
  capacity: 5000,
  price: 75,
  currency: 'USD'
});

// Send WhatsApp invitations
await client.whatsapp.sendInvitations({
  eventId: event.id,
  phoneNumbers: ['+1234567890', '+0987654321'],
  message: 'You are invited to the Summer Music Festival!'
});`,
    python: `# Install the SDK
pip install eventfulhq

# Initialize the client
from eventfulhq import EventfulHQ

client = EventfulHQ(
    api_key='your-api-key',
    region='global'  # or 'america', 'arabia', 'india', 'europe', 'australia'
)

# Create an event
event = client.events.create(
    title='Summer Music Festival',
    description='A fantastic outdoor music experience',
    date='2025-07-15T18:00:00Z',
    venue={
        'name': 'Central Park',
        'address': 'New York, NY',
        'coordinates': {'lat': 40.7829, 'lng': -73.9654}
    },
    capacity=5000,
    price=75,
    currency='USD'
)

# Send WhatsApp invitations
client.whatsapp.send_invitations(
    event_id=event.id,
    phone_numbers=['+1234567890', '+0987654321'],
    message='You are invited to the Summer Music Festival!'
)`,
    curl: `# Get your API key from the developer portal
API_KEY="your-api-key"

# Create an event
curl -X POST https://api.eventfulhq.com/v1/events \\
  -H "Authorization: Bearer $API_KEY" \\
  -H "Content-Type: application/json" \\
  -d '{
    "title": "Summer Music Festival",
    "description": "A fantastic outdoor music experience",
    "date": "2025-07-15T18:00:00Z",
    "venue": {
      "name": "Central Park",
      "address": "New York, NY",
      "coordinates": {"lat": 40.7829, "lng": -73.9654}
    },
    "capacity": 5000,
    "price": 75,
    "currency": "USD"
  }'

# Send WhatsApp invitations
curl -X POST https://api.eventfulhq.com/v1/events/{eventId}/whatsapp-invite \\
  -H "Authorization: Bearer $API_KEY" \\
  -H "Content-Type: application/json" \\
  -d '{
    "phoneNumbers": ["+1234567890", "+0987654321"],
    "message": "You are invited to the Summer Music Festival!"
  }'`,
  };

  return (
    <div className="space-y-6">
      <div className="flex items-center justify-between">
        <div className="flex items-center gap-4">
          <h2 className="text-2xl font-bold">API Documentation</h2>
          <Badge variant="outline">API v1</Badge>
        </div>
        <div className="flex items-center gap-2">
          <Button variant="outline" size="sm">
            <Download className="h-4 w-4 mr-2" />
            Download OpenAPI Spec
          </Button>
          <Button variant="outline" size="sm">
            <Book className="h-4 w-4 mr-2" />
            API Changelog
          </Button>
        </div>
      </div>

      <Tabs defaultValue="reference" className="space-y-4">
        <TabsList className="grid w-full grid-cols-4">
          <TabsTrigger value="quickstart">Quick Start</TabsTrigger>
          <TabsTrigger value="reference">API Reference</TabsTrigger>
          <TabsTrigger value="playground">Playground</TabsTrigger>
          <TabsTrigger value="sdks">SDKs</TabsTrigger>
        </TabsList>

        <TabsContent value="quickstart" className="space-y-4">
          <Card>
            <CardHeader>
              <CardTitle>Getting Started</CardTitle>
              <CardDescription>
                Start building with EventfulHQ APIs in minutes
              </CardDescription>
            </CardHeader>
            <CardContent className="space-y-4">
              <div className="space-y-2">
                <h3 className="font-semibold">1. Get your API key</h3>
                <p className="text-sm text-muted-foreground">
                  Sign in to the developer portal and create a new API key from the dashboard
                </p>
              </div>

              <div className="space-y-2">
                <h3 className="font-semibold">2. Choose your integration method</h3>
                <div className="flex gap-2">
                  {['javascript', 'python', 'curl'].map((lang) => (
                    <Button
                      key={lang}
                      variant={selectedLanguage === lang ? 'default' : 'outline'}
                      size="sm"
                      onClick={() => setSelectedLanguage(lang)}
                    >
                      {lang.charAt(0).toUpperCase() + lang.slice(1)}
                    </Button>
                  ))}
                </div>
              </div>

              <div className="space-y-2">
                <h3 className="font-semibold">3. Make your first API call</h3>
                <CodeBlock
                  language={selectedLanguage}
                  code={codeSamples[selectedLanguage]}
                />
              </div>
            </CardContent>
          </Card>
        </TabsContent>

        <TabsContent value="reference" className="space-y-4">
          <div className="flex gap-4">
            <div className="w-64 space-y-2">
              <h3 className="font-semibold text-sm">Services</h3>
              {services.map((service) => (
                <Button
                  key={service.id}
                  variant={selectedService === service.id ? 'secondary' : 'ghost'}
                  size="sm"
                  className="w-full justify-start"
                  onClick={() => setSelectedService(service.id)}
                >
                  {service.name}
                  <Badge variant="outline" className="ml-auto">
                    {service.version}
                  </Badge>
                </Button>
              ))}
            </div>
            <div className="flex-1">
              <Card>
                <CardContent className="p-0">
                  {apiSpec && (
                    <SwaggerUI
                      spec={apiSpec}
                      docExpansion="list"
                      defaultModelsExpandDepth={1}
                      tryItOutEnabled={true}
                    />
                  )}
                </CardContent>
              </Card>
            </div>
          </div>
        </TabsContent>

        <TabsContent value="playground" className="space-y-4">
          <APIPlayground />
        </TabsContent>

        <TabsContent value="sdks" className="space-y-4">
          <div className="grid gap-4 md:grid-cols-2 lg:grid-cols-3">
            {sdks?.map((sdk) => (
              <Card key={sdk.language}>
                <CardHeader>
                  <CardTitle className="flex items-center justify-between">
                    {sdk.name}
                    <Badge>{sdk.version}</Badge>
                  </CardTitle>
                  <CardDescription>{sdk.description}</CardDescription>
                </CardHeader>
                <CardContent className="space-y-4">
                  <CodeBlock
                    language="bash"
                    code={sdk.installCommand}
                  />
                  <div className="flex gap-2">
                    <Button variant="outline" size="sm" asChild>
                      <a href={sdk.githubUrl} target="_blank" rel="noopener">
                        <Code className="h-4 w-4 mr-2" />
                        GitHub
                      </a>
                    </Button>
                    <Button variant="outline" size="sm" asChild>
                      <a href={sdk.docsUrl} target="_blank" rel="noopener">
                        <Book className="h-4 w-4 mr-2" />
                        Docs
                      </a>
                    </Button>
                  </div>
                </CardContent>
              </Card>
            ))}
          </div>
        </TabsContent>
      </Tabs>
    </div>
  );
}
```

### API Key Management
```typescript
// components/features/developers/api-key-manager.tsx
'use client';

import { useState } from 'react';
import { useQuery, useMutation } from '@tanstack/react-query';
import { Card, CardContent, CardDescription, CardHeader, CardTitle } from '@/components/ui/card';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { Label } from '@/components/ui/label';
import { Badge } from '@/components/ui/badge';
import {
  Dialog,
  DialogContent,
  DialogDescription,
  DialogHeader,
  DialogTitle,
  DialogTrigger,
} from '@/components/ui/dialog';
import {
  Table,
  TableBody,
  TableCell,
  TableHead,
  TableHeader,
  TableRow,
} from '@/components/ui/table';
import { developerService } from '@/services/developer-service';
import { Copy, Eye, EyeOff, Key, Trash, Plus } from 'lucide-react';
import { toast } from 'sonner';

export function APIKeyManager() {
  const [showCreateDialog, setShowCreateDialog] = useState(false);
  const [newKeyName, setNewKeyName] = useState('');
  const [selectedScopes, setSelectedScopes] = useState<string[]>([]);
  const [visibleKeys, setVisibleKeys] = useState<Set<string>>(new Set());

  const { data: apiKeys, isLoading } = useQuery({
    queryKey: ['api-keys'],
    queryFn: developerService.getAPIKeys,
  });

  const { mutate: createKey } = useMutation({
    mutationFn: developerService.createAPIKey,
    onSuccess: (data) => {
      toast.success('API key created successfully');
      // Show the key once for copying
      navigator.clipboard.writeText(data.key);
      toast.info('API key copied to clipboard. This is the only time you can see it!');
      setShowCreateDialog(false);
      setNewKeyName('');
      setSelectedScopes([]);
      queryClient.invalidateQueries({ queryKey: ['api-keys'] });
    },
  });

  const { mutate: deleteKey } = useMutation({
    mutationFn: developerService.deleteAPIKey,
    onSuccess: () => {
      toast.success('API key deleted successfully');
      queryClient.invalidateQueries({ queryKey: ['api-keys'] });
    },
  });

  const scopes = [
    { id: 'events:read', name: 'Read Events' },
    { id: 'events:write', name: 'Write Events' },
    { id: 'users:read', name: 'Read Users' },
    { id: 'users:write', name: 'Write Users' },
    { id: 'analytics:read', name: 'Read Analytics' },
    { id: 'whatsapp:send', name: 'Send WhatsApp Messages' },
    { id: 'rfp:manage', name: 'Manage RFPs' },
    { id: 'payments:process', name: 'Process Payments' },
  ];

  const toggleKeyVisibility = (keyId: string) => {
    setVisibleKeys(prev => {
      const newSet = new Set(prev);
      if (newSet.has(keyId)) {
        newSet.delete(keyId);
      } else {
        newSet.add(keyId);
      }
      return newSet;
    });
  };

  return (
    <Card>
      <CardHeader>
        <div className="flex items-center justify-between">
          <div>
            <CardTitle>API Keys</CardTitle>
            <CardDescription>
              Manage your API keys for accessing EventfulHQ APIs
            </CardDescription>
          </div>
          <Dialog open={showCreateDialog} onOpenChange={setShowCreateDialog}>
            <DialogTrigger asChild>
              <Button>
                <Plus className="h-4 w-4 mr-2" />
                Create API Key
              </Button>
            </DialogTrigger>
            <DialogContent>
              <DialogHeader>
                <DialogTitle>Create New API Key</DialogTitle>
                <DialogDescription>
                  Generate a new API key with specific permissions
                </DialogDescription>
              </DialogHeader>
              <div className="space-y-4">
                <div className="space-y-2">
                  <Label>Key Name</Label>
                  <Input
                    placeholder="My App API Key"
                    value={newKeyName}
                    onChange={(e) => setNewKeyName(e.target.value)}
                  />
                </div>
                <div className="space-y-2">
                  <Label>Permissions</Label>
                  <div className="space-y-2">
                    {scopes.map((scope) => (
                      <div key={scope.id} className="flex items-center space-x-2">
                        <Checkbox
                          id={scope.id}
                          checked={selectedScopes.includes(scope.id)}
                          onCheckedChange={(checked) => {
                            if (checked) {
                              setSelectedScopes([...selectedScopes, scope.id]);
                            } else {
                              setSelectedScopes(selectedScopes.filter(s => s !== scope.id));
                            }
                          }}
                        />
                        <Label htmlFor={scope.id} className="text-sm">
                          {scope.name}
                        </Label>
                      </div>
                    ))}
                  </div>
                </div>
                <Button
                  className="w-full"
                  onClick={() => createKey({ name: newKeyName, scopes: selectedScopes })}
                  disabled={!newKeyName || selectedScopes.length === 0}
                >
                  Create API Key
                </Button>
              </div>
            </DialogContent>
          </Dialog>
        </div>
      </CardHeader>
      <CardContent>
        <Table>
          <TableHeader>
            <TableRow>
              <TableHead>Name</TableHead>
              <TableHead>Key</TableHead>
              <TableHead>Permissions</TableHead>
              <TableHead>Last Used</TableHead>
              <TableHead>Created</TableHead>
              <TableHead></TableHead>
            </TableRow>
          </TableHeader>
          <TableBody>
            {apiKeys?.map((apiKey) => (
              <TableRow key={apiKey.id}>
                <TableCell className="font-medium">{apiKey.name}</TableCell>
                <TableCell>
                  <div className="flex items-center gap-2">
                    <code className="text-xs bg-muted px-2 py-1 rounded">
                      {visibleKeys.has(apiKey.id) 
                        ? apiKey.key 
                        : `${apiKey.key.slice(0, 12)}...`}
                    </code>
                    <Button
                      variant="ghost"
                      size="icon"
                      className="h-6 w-6"
                      onClick={() => toggleKeyVisibility(apiKey.id)}
                    >
                      {visibleKeys.has(apiKey.id) ? (
                        <EyeOff className="h-3 w-3" />
                      ) : (
                        <Eye className="h-3 w-3" />
                      )}
                    </Button>
                    <Button
                      variant="ghost"
                      size="icon"
                      className="h-6 w-6"
                      onClick={() => {
                        navigator.clipboard.writeText(apiKey.key);
                        toast.success('API key copied to clipboard');
                      }}
                    >
                      <Copy className="h-3 w-3" />
                    </Button>
                  </div>
                </TableCell>
                <TableCell>
                  <div className="flex flex-wrap gap-1">
                    {apiKey.scopes.slice(0, 2).map((scope) => (
                      <Badge key={scope} variant="secondary" className="text-xs">
                        {scope}
                      </Badge>
                    ))}
                    {apiKey.scopes.length > 2 && (
                      <Badge variant="secondary" className="text-xs">
                        +{apiKey.scopes.length - 2}
                      </Badge>
                    )}
                  </div>
                </TableCell>
                <TableCell>
                  {apiKey.lastUsed ? (
                    <span className="text-sm text-muted-foreground">
                      {new Date(apiKey.lastUsed).toLocaleDateString()}
                    </span>
                  ) : (
                    <span className="text-sm text-muted-foreground">Never</span>
                  )}
                </TableCell>
                <TableCell>
                  <span className="text-sm text-muted-foreground">
                    {new Date(apiKey.createdAt).toLocaleDateString()}
                  </span>
                </TableCell>
                <TableCell>
                  <Button
                    variant="ghost"
                    size="icon"
                    onClick={() => deleteKey(apiKey.id)}
                  >
                    <Trash className="h-4 w-4" />
                  </Button>
                </TableCell>
              </TableRow>
            ))}
          </TableBody>
        </Table>
      </CardContent>
    </Card>
  );
}
```

## Landing Pages & Microsites

### Dynamic City Landing Page
```typescript
// app/[locale]/(landing)/[country]/[city]/page.tsx
import { Metadata } from 'next';
import { notFound } from 'next/navigation';
import { CityHero } from '@/components/features/landing/city-hero';
import { TrendingEvents } from '@/components/features/landing/trending-events';
import { TrendingArtists } from '@/components/features/landing/trending-artists';
import { TrendingVenues } from '@/components/features/landing/trending-venues';
import { AdSenseBanner } from '@/components/features/landing/adsense-banner';
import { StructuredData } from '@/components/features/seo/structured-data';
import { landingService } from '@/services/landing-service';

interface CityPageProps {
  params: {
    locale: string;
    country: string;
    city: string;
  };
}

export async function generateMetadata({ params }: CityPageProps): Promise<Metadata> {
  const cityData = await landingService.getCityData(params.country, params.city);
  
  if (!cityData) return {};
  
  return {
    title: `Events in ${cityData.name} - ${cityData.eventCount} Upcoming Events | EventfulHQ`,
    description: `Discover ${cityData.eventCount} amazing events happening in ${cityData.name}. Find concerts, festivals, workshops, and more. Book tickets for the best events in ${cityData.name}.`,
    openGraph: {
      title: `Events in ${cityData.name}`,
      description: `Discover amazing events in ${cityData.name}`,
      images: [cityData.heroImage],
      type: 'website',
    },
    alternates: {
      canonical: `https://eventfulhq.com/${params.locale}/${params.country}/${params.city}`,
      languages: {
        'en': `https://eventfulhq.com/en/${params.country}/${params.city}`,
        'es': `https://eventfulhq.com/es/${params.country}/${params.city}`,
        // ... other languages
      },
    },
  };
}

export async function generateStaticParams() {
  // Generate static pages for top cities
  const topCities = await landingService.getTopCities();
  
  return topCities.map((city) => ({
    country: city.countryCode.toLowerCase(),
    city: city.slug,
  }));
}

export default async function CityLandingPage({ params }: CityPageProps) {
  const cityData = await landingService.getCityData(params.country, params.city);
  
  if (!cityData) {
    notFound();
  }
  
  const [trendingEvents, trendingArtists, trendingVenues] = await Promise.all([
    landingService.getTrendingEvents(cityData.id),
    landingService.getTrendingArtists(cityData.id),
    landingService.getTrendingVenues(cityData.id),
  ]);
  
  const structuredData = {
    '@context': 'https://schema.org',
    '@type': 'City',
    name: cityData.name,
    geo: {
      '@type': 'GeoCoordinates',
      latitude: cityData.coordinates.lat,
      longitude: cityData.coordinates.lng,
    },
    containsPlace: trendingVenues.map(venue => ({
      '@type': 'EventVenue',
      name: venue.name,
      address: venue.address,
    })),
    event: trendingEvents.map(event => ({
      '@type': 'Event',
      name: event.title,
      startDate: event.date,
      location: {
        '@type': 'Place',
        name: event.venue.name,
        address: event.venue.address,
      },
      offers: {
        '@type': 'Offer',
        price: event.price,
        priceCurrency: event.currency,
        availability: event.isSoldOut ? 'SoldOut' : 'InStock',
      },
    })),
  };
  
  return (
    <>
      <StructuredData data={structuredData} />
      
      <div className="min-h-screen">
        <CityHero city={cityData} />
        
        <div className="container py-8 space-y-12">
          {/* AdSense Banner - Top */}
          <AdSenseBanner 
            slot="city-landing-top" 
            format="horizontal"
            className="mb-8"
          />
          
          {/* Trending Events Section */}
          <section>
            <h2 className="text-3xl font-bold mb-6">
              Trending Events in {cityData.name}
            </h2>
            <TrendingEvents events={trendingEvents} city={cityData} />
          </section>
          
          {/* AdSense Banner - Middle */}
          <AdSenseBanner 
            slot="city-landing-middle" 
            format="rectangle"
            className="my-8"
          />
          
          {/* Trending Artists Section */}
          <section>
            <h2 className="text-3xl font-bold mb-6">
              Popular Artists in {cityData.name}
            </h2>
            <TrendingArtists artists={trendingArtists} />
          </section>
          
          {/* Trending Venues Section */}
          <section>
            <h2 className="text-3xl font-bold mb-6">
              Top Venues in {cityData.name}
            </h2>
            <TrendingVenues venues={trendingVenues} />
          </section>
          
          {/* AdSense Banner - Bottom */}
          <AdSenseBanner 
            slot="city-landing-bottom" 
            format="horizontal"
            className="mt-8"
          />
          
          {/* SEO Content */}
          <section className="prose max-w-none">
            <h3>About Events in {cityData.name}</h3>
            <p>{cityData.description}</p>
            <p>
              With over {cityData.eventCount} upcoming events, {cityData.name} is 
              a vibrant hub for entertainment and culture. From live music concerts 
              to art exhibitions, food festivals to tech conferences, there's always 
              something exciting happening in {cityData.name}.
            </p>
          </section>
        </div>
      </div>
    </>
  );
}
```

### User Microsite (Without Ads for Premium Users)
```typescript
// app/[locale]/(landing)/u/[username]/page.tsx
import { Metadata } from 'next';
import { notFound } from 'next/navigation';
import { UserProfile } from '@/components/features/landing/user-profile';
import { UserEvents } from '@/components/features/landing/user-events';
import { UserStats } from '@/components/features/landing/user-stats';
import { ShareButtons } from '@/components/features/landing/share-buttons';
import { AdSenseBanner } from '@/components/features/landing/adsense-banner';
import { landingService } from '@/services/landing-service';
import { userService } from '@/services/user-service';

interface UserMicrositeProps {
  params: {
    locale: string;
    username: string;
  };
}

export async function generateMetadata({ params }: UserMicrositeProps): Promise<Metadata> {
  const user = await userService.getUserByUsername(params.username);
  
  if (!user) return {};
  
  return {
    title: `${user.name} - Event Organizer | EventfulHQ`,
    description: user.bio || `Check out events organized by ${user.name} on EventfulHQ`,
    openGraph: {
      title: user.name,
      description: user.bio,
      images: [user.avatar],
      type: 'profile',
    },
  };
}

export default async function UserMicrosite({ params }: UserMicrositeProps) {
  const user = await userService.getUserByUsername(params.username);
  
  if (!user) {
    notFound();
  }
  
  const [userEvents, userStats] = await Promise.all([
    landingService.getUserEvents(user.id),
    landingService.getUserStats(user.id),
  ]);
  
  const isPremium = user.subscription?.tier === 'premium';
  const customTheme = user.micrositeTheme || 'default';
  
  return (
    <div className={`min-h-screen theme-${customTheme}`}>
      <UserProfile user={user} stats={userStats} />
      
      <div className="container py-8 space-y-8">
        {/* No ads for premium users */}
        {!isPremium && (
          <AdSenseBanner 
            slot="user-microsite-top" 
            format="horizontal"
          />
        )}
        
        <section>
          <div className="flex items-center justify-between mb-6">
            <h2 className="text-2xl font-bold">Upcoming Events</h2>
            <ShareButtons 
              url={`https://eventfulhq.com/u/${user.username}`}
              title={`Check out events by ${user.name}`}
            />
          </div>
          <UserEvents events={userEvents} showOrganizer={false} />
        </section>
        
        {!isPremium && userEvents.length > 6 && (
          <AdSenseBanner 
            slot="user-microsite-middle" 
            format="rectangle"
          />
        )}
        
        <UserStats stats={userStats} />
        
        {!isPremium && (
          <AdSenseBanner 
            slot="user-microsite-bottom" 
            format="horizontal"
          />
        )}
      </div>
    </div>
  );
}
```

### Event Microsite Component
```typescript
// app/[locale]/(landing)/e/[eventSlug]/page.tsx
import { Metadata } from 'next';
import { notFound } from 'next/navigation';
import { EventHero } from '@/components/features/landing/event-hero';
import { EventDetails } from '@/components/features/landing/event-details';
import { TicketWidget } from '@/components/features/landing/ticket-widget';
import { WhatsAppRSVP } from '@/components/features/landing/whatsapp-rsvp';
import { AdSenseBanner } from '@/components/features/landing/adsense-banner';
import { StructuredData } from '@/components/features/seo/structured-data';
import { eventService } from '@/services/event-service';

interface EventMicrositeProps {
  params: {
    locale: string;
    eventSlug: string;
  };
}

export async function generateMetadata({ params }: EventMicrositeProps): Promise<Metadata> {
  const event = await eventService.getEventBySlug(params.eventSlug);
  
  if (!event) return {};
  
  return {
    title: `${event.title} - ${new Date(event.date).toLocaleDateString()} | EventfulHQ`,
    description: event.description,
    openGraph: {
      title: event.title,
      description: event.description,
      images: [event.coverImage],
      type: 'website',
    },
  };
}

export default async function EventMicrosite({ params }: EventMicrositeProps) {
  const event = await eventService.getEventBySlug(params.eventSlug);
  
  if (!event) {
    notFound();
  }
  
  const isPremiumEvent = event.organizer.subscription?.tier === 'premium';
  const customTheme = event.micrositeTheme || event.organizer.micrositeTheme || 'default';
  
  const structuredData = {
    '@context': 'https://schema.org',
    '@type': 'Event',
    name: event.title,
    description: event.description,
    startDate: event.date,
    endDate: event.endDate,
    location: {
      '@type': 'Place',
      name: event.venue.name,
      address: {
        '@type': 'PostalAddress',
        streetAddress: event.venue.address,
        addressLocality: event.venue.city,
        addressCountry: event.venue.country,
      },
      geo: {
        '@type': 'GeoCoordinates',
        latitude: event.venue.coordinates.lat,
        longitude: event.venue.coordinates.lng,
      },
    },
    offers: {
      '@type': 'Offer',
      url: `https://eventfulhq.com/e/${event.slug}`,
      price: event.price,
      priceCurrency: event.currency,
      availability: event.isSoldOut 
        ? 'https://schema.org/SoldOut' 
        : 'https://schema.org/InStock',
      validFrom: new Date().toISOString(),
    },
    performer: event.artists?.map(artist => ({
      '@type': 'Person',
      name: artist.name,
    })),
    organizer: {
      '@type': 'Organization',
      name: event.organizer.name,
      url: `https://eventfulhq.com/u/${event.organizer.username}`,
    },
  };
  
  return (
    <>
      <StructuredData data={structuredData} />
      
      <div className={`min-h-screen theme-${customTheme}`}>
        <EventHero event={event} />
        
        <div className="container py-8">
          <div className="grid gap-8 lg:grid-cols-3">
            <div className="lg:col-span-2 space-y-8">
              <EventDetails event={event} />
              
              {!isPremiumEvent && (
                <AdSenseBanner 
                  slot="event-microsite-content" 
                  format="rectangle"
                />
              )}
              
              {/* Event Description */}
              <section>
                <h2 className="text-2xl font-bold mb-4">About This Event</h2>
                <div className="prose max-w-none">
                  {event.description}
                </div>
              </section>
              
              {/* Artists/Performers */}
              {event.artists && event.artists.length > 0 && (
                <section>
                  <h2 className="text-2xl font-bold mb-4">Featuring</h2>
                  <div className="grid gap-4 sm:grid-cols-2">
                    {event.artists.map(artist => (
                      <ArtistCard key={artist.id} artist={artist} />
                    ))}
                  </div>
                </section>
              )}
            </div>
            
            <div className="space-y-6">
              {/* Ticket Widget */}
              <TicketWidget event={event} />
              
              {/* WhatsApp RSVP */}
              {event.isWhatsAppRSVP && (
                <WhatsAppRSVP event={event} />
              )}
              
              {!isPremiumEvent && (
                <AdSenseBanner 
                  slot="event-microsite-sidebar" 
                  format="vertical"
                />
              )}
              
              {/* Venue Information */}
              <VenueInfo venue={event.venue} />
              
              {/* Share Buttons */}
              <ShareButtons 
                url={`https://eventfulhq.com/e/${event.slug}`}
                title={event.title}
                description={event.description}
              />
            </div>
          </div>
        </div>
        
        {!isPremiumEvent && (
          <AdSenseBanner 
            slot="event-microsite-bottom" 
            format="horizontal"
            className="mt-8"
          />
        )}
      </div>
    </>
  );
}
```

### AdSense Integration Component
```typescript
// components/features/landing/adsense-banner.tsx
'use client';

import { useEffect } from 'react';
import { usePathname } from 'next/navigation';
import Script from 'next/script';

interface AdSenseBannerProps {
  slot: string;
  format?: 'horizontal' | 'vertical' | 'rectangle' | 'auto';
  className?: string;
  responsive?: boolean;
}

export function AdSenseBanner({ 
  slot, 
  format = 'auto',
  className = '',
  responsive = true 
}: AdSenseBannerProps) {
  const pathname = usePathname();
  
  useEffect(() => {
    try {
      // Push ads after route change
      (window.adsbygoogle = window.adsbygoogle || []).push({});
    } catch (err) {
      console.error('AdSense error:', err);
    }
  }, [pathname]);
  
  const formatStyles = {
    horizontal: { width: '728px', height: '90px' },
    vertical: { width: '300px', height: '600px' },
    rectangle: { width: '336px', height: '280px' },
    auto: { width: '100%', height: 'auto' },
  };
  
  const style = formatStyles[format];
  
  return (
    <>
      <Script
        id="adsense-script"
        async
        src={`https://pagead2.googlesyndication.com/pagead/js/adsbygoogle.js?client=${process.env.NEXT_PUBLIC_ADSENSE_CLIENT_ID}`}
        crossOrigin="anonymous"
        strategy="afterInteractive"
      />
      <div className={`adsense-container ${className}`}>
        <ins
          className="adsbygoogle"
          style={{
            display: 'block',
            width: style.width,
            height: style.height,
            ...(responsive && { 
              width: '100%',
              height: 'auto',
            }),
          }}
          data-ad-client={process.env.NEXT_PUBLIC_ADSENSE_CLIENT_ID}
          data-ad-slot={slot}
          data-ad-format={format}
          data-full-width-responsive={responsive ? 'true' : 'false'}
        />
      </div>
    </>
  );
}

// AdSense configuration for different page types
export const adSlots = {
  'city-landing-top': '1234567890',
  'city-landing-middle': '2345678901',
  'city-landing-bottom': '3456789012',
  'country-landing-top': '4567890123',
  'country-landing-sidebar': '5678901234',
  'user-microsite-top': '6789012345',
  'user-microsite-middle': '7890123456',
  'user-microsite-bottom': '8901234567',
  'event-microsite-content': '9012345678',
  'event-microsite-sidebar': '0123456789',
  'event-microsite-bottom': '1234567890',
};
```

## Bot Management & SEO Strategy

### Bot Detection Middleware
```typescript
// middleware/bot-detection.ts
import { NextRequest, NextResponse } from 'next/server';
import { botPatterns, seoBotsPatterns, llmBotsPatterns } from '@/lib/seo/bot-patterns';

export async function botDetectionMiddleware(request: NextRequest) {
  const userAgent = request.headers.get('user-agent') || '';
  const ip = request.ip || request.headers.get('x-forwarded-for') || '';
  
  // Check if it's a bot
  const isBot = botPatterns.some(pattern => pattern.test(userAgent));
  const isSEOBot = seoBotsPatterns.some(pattern => pattern.test(userAgent));
  const isLLMBot = llmBotsPatterns.some(pattern => pattern.test(userAgent));
  
  // Log bot access for analytics
  if (isBot) {
    await logBotAccess({
      userAgent,
      ip,
      path: request.nextUrl.pathname,
      type: isSEOBot ? 'seo' : isLLMBot ? 'llm' : 'other',
      timestamp: new Date(),
    });
  }
  
  // Block unauthorized bots
  if (isBot && !isSEOBot && !isLLMBot) {
    // Check rate limiting for suspicious bots
    const rateLimit = await checkBotRateLimit(ip);
    
    if (rateLimit.exceeded) {
      return new NextResponse('Too Many Requests', { status: 429 });
    }
    
    // Return minimal response for unauthorized bots
    return new NextResponse('Access Denied', { 
      status: 403,
      headers: {
        'X-Robots-Tag': 'noindex, nofollow',
      }
    });
  }
  
  // For SEO and LLM bots, add special headers and optimizations
  if (isSEOBot || isLLMBot) {
    const response = NextResponse.next();
    
    // Add bot-specific headers
    response.headers.set('X-Bot-Type', isSEOBot ? 'seo' : 'llm');
    response.headers.set('Cache-Control', 'public, max-age=3600');
    
    // For LLM bots, add structured data hints
    if (isLLMBot) {
      response.headers.set('X-LLM-Hints', 'structured-data-available');
    }
    
    return response;
  }
  
  return NextResponse.next();
}

// Bot patterns configuration
export const botPatterns = [
  // Common crawlers and scrapers
  /bot|crawler|spider|scraper|curl|wget|python-requests/i,
  // Specific bad bots
  /ahrefsbot|semrushbot|dotbot|rogerbot|facebookexternalhit|ia_archiver/i,
  // Vulnerability scanners
  /nikto|sqlmap|openvas|nessus|w3af/i,
];

export const seoBotsPatterns = [
  // Search engine bots
  /googlebot|bingbot|slurp|duckduckbot|baiduspider|yandexbot/i,
  // Social media bots
  /linkedinbot|whatsapp|telegrambot/i,
  // SEO tool bots (allowed)
  /screaming frog|deepcrawl|oncrawl/i,
];

export const llmBotsPatterns = [
  // AI/LLM crawlers
  /gpt-?bot|claude-?bot|anthropic|openai/i,
  /chatgpt|bard|llama|perplexity/i,
  // AI research bots
  /commoncrawl|ccbot/i,
];

// Rate limiting for bots
async function checkBotRateLimit(ip: string): Promise<{ exceeded: boolean }> {
  const key = `bot-rate-limit:${ip}`;
  const limit = 100; // requests per hour
  const window = 3600; // 1 hour in seconds
  
  // Implementation would use Redis or similar
  // This is a simplified example
  return { exceeded: false };
}

// Log bot access for analytics
async function logBotAccess(data: {
  userAgent: string;
  ip: string;
  path: string;
  type: string;
  timestamp: Date;
}) {
  // Log to analytics service
  console.log('Bot access:', data);
}
```

### Dynamic Robots.txt Generation
```typescript
// app/robots.txt/route.ts
import { NextRequest } from 'next/server';

export async function GET(request: NextRequest) {
  const host = request.headers.get('host') || 'eventfulhq.com';
  const protocol = process.env.NODE_ENV === 'production' ? 'https' : 'http';
  
  const robotsTxt = `# EventfulHQ Robots.txt
# Generated dynamically based on domain

User-agent: *
Allow: /
Disallow: /api/
Disallow: /admin/
Disallow: /super-admin/
Disallow: /_next/
Disallow: /static/
Crawl-delay: 1

# Search engines
User-agent: Googlebot
Allow: /
Crawl-delay: 0

User-agent: Bingbot
Allow: /
Crawl-delay: 0

User-agent: Slurp
Allow: /
Crawl-delay: 1

User-agent: DuckDuckBot
Allow: /
Crawl-delay: 1

# Social media
User-agent: LinkedInBot
Allow: /

User-agent: WhatsApp
Allow: /

User-agent: Twitterbot
Allow: /

# AI/LLM Bots
User-agent: GPTBot
Allow: /
Crawl-delay: 2

User-agent: Claude-Web
Allow: /
Crawl-delay: 2

User-agent: CCBot
Allow: /
Crawl-delay: 5

# Bad bots
User-agent: AhrefsBot
Disallow: /

User-agent: SemrushBot
Disallow: /

User-agent: DotBot
Disallow: /

User-agent: MJ12bot
Disallow: /

# Sitemaps
Sitemap: ${protocol}://${host}/sitemap.xml
Sitemap: ${protocol}://${host}/sitemap-events.xml
Sitemap: ${protocol}://${host}/sitemap-users.xml
Sitemap: ${protocol}://${host}/sitemap-venues.xml
Sitemap: ${protocol}://${host}/sitemap-cities.xml
Sitemap: ${protocol}://${host}/sitemap-countries.xml
`;

  return new Response(robotsTxt, {
    headers: {
      'Content-Type': 'text/plain',
      'Cache-Control': 'public, max-age=3600',
    },
  });
}
```

### Dynamic Sitemap Generation
```typescript
// app/sitemap.xml/route.ts
import { NextRequest } from 'next/server';
import { sitemapService } from '@/services/sitemap-service';

export async function GET(request: NextRequest) {
  const host = request.headers.get('host') || 'eventfulhq.com';
  const protocol = process.env.NODE_ENV === 'production' ? 'https' : 'http';
  
  // Generate sitemap index
  const sitemaps = [
    'sitemap-static.xml',
    'sitemap-events.xml',
    'sitemap-users.xml',
    'sitemap-venues.xml',
    'sitemap-cities.xml',
    'sitemap-countries.xml',
  ];
  
  const sitemapIndex = `<?xml version="1.0" encoding="UTF-8"?>
<sitemapindex xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
${sitemaps.map(sitemap => `  <sitemap>
    <loc>${protocol}://${host}/${sitemap}</loc>
    <lastmod>${new Date().toISOString()}</lastmod>
  </sitemap>`).join('\n')}
</sitemapindex>`;

  return new Response(sitemapIndex, {
    headers: {
      'Content-Type': 'application/xml',
      'Cache-Control': 'public, max-age=3600',
    },
  });
}

// Dynamic event sitemap
export async function generateEventSitemap() {
  const events = await sitemapService.getPublishedEvents();
  
  const sitemap = `<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9"
        xmlns:image="http://www.google.com/schemas/sitemap-image/1.1">
${events.map(event => `  <url>
    <loc>https://eventfulhq.com/e/${event.slug}</loc>
    <lastmod>${event.updatedAt}</lastmod>
    <changefreq>daily</changefreq>
    <priority>0.8</priority>
    <image:image>
      <image:loc>${event.coverImage}</image:loc>
      <image:title>${event.title}</image:title>
    </image:image>
  </url>`).join('\n')}
</urlset>`;

  return sitemap;
}
```

### SEO Monitoring Dashboard
```typescript
// components/features/seo/seo-monitoring.tsx
'use client';

import { useQuery } from '@tanstack/react-query';
import { Card, CardContent, CardDescription, CardHeader, CardTitle } from '@/components/ui/card';
import { Tabs, TabsContent, TabsList, TabsTrigger } from '@/components/ui/tabs';
import { Progress } from '@/components/ui/progress';
import { Badge } from '@/components/ui/badge';
import {
  LineChart,
  Line,
  BarChart,
  Bar,
  XAxis,
  YAxis,
  CartesianGrid,
  Tooltip,
  ResponsiveContainer,
} from 'recharts';
import { seoService } from '@/services/seo-service';
import { Bot, Search, TrendingUp, AlertCircle } from 'lucide-react';

export function SEOMonitoring() {
  const { data: seoData } = useQuery({
    queryKey: ['seo-monitoring'],
    queryFn: seoService.getMonitoringData,
  });

  const { data: botTraffic } = useQuery({
    queryKey: ['bot-traffic'],
    queryFn: seoService.getBotTraffic,
  });

  return (
    <div className="space-y-6">
      <div className="grid gap-4 md:grid-cols-2 lg:grid-cols-4">
        <Card>
          <CardHeader className="flex flex-row items-center justify-between space-y-0 pb-2">
            <CardTitle className="text-sm font-medium">SEO Score</CardTitle>
            <Search className="h-4 w-4 text-muted-foreground" />
          </CardHeader>
          <CardContent>
            <div className="text-2xl font-bold">{seoData?.seoScore}/100</div>
            <Progress value={seoData?.seoScore} className="mt-2" />
          </CardContent>
        </Card>

        <Card>
          <CardHeader className="flex flex-row items-center justify-between space-y-0 pb-2">
            <CardTitle className="text-sm font-medium">Indexed Pages</CardTitle>
            <TrendingUp className="h-4 w-4 text-muted-foreground" />
          </CardHeader>
          <CardContent>
            <div className="text-2xl font-bold">
              {seoData?.indexedPages.toLocaleString()}
            </div>
            <p className="text-xs text-muted-foreground">
              +{seoData?.newPagesThisWeek} this week
            </p>
          </CardContent>
        </Card>

        <Card>
          <CardHeader className="flex flex-row items-center justify-between space-y-0 pb-2">
            <CardTitle className="text-sm font-medium">Bot Traffic</CardTitle>
            <Bot className="h-4 w-4 text-muted-foreground" />
          </CardHeader>
          <CardContent>
            <div className="text-2xl font-bold">
              {botTraffic?.total.toLocaleString()}
            </div>
            <div className="flex gap-2 mt-2">
              <Badge variant="secondary" className="text-xs">
                SEO: {botTraffic?.seo}
              </Badge>
              <Badge variant="secondary" className="text-xs">
                LLM: {botTraffic?.llm}
              </Badge>
            </div>
          </CardContent>
        </Card>

        <Card>
          <CardHeader className="flex flex-row items-center justify-between space-y-0 pb-2">
            <CardTitle className="text-sm font-medium">Issues</CardTitle>
            <AlertCircle className="h-4 w-4 text-muted-foreground" />
          </CardHeader>
          <CardContent>
            <div className="text-2xl font-bold">{seoData?.issues.total}</div>
            <div className="flex gap-2 mt-2">
              <Badge variant="destructive" className="text-xs">
                Critical: {seoData?.issues.critical}
              </Badge>
              <Badge variant="outline" className="text-xs">
                Warnings: {seoData?.issues.warnings}
              </Badge>
            </div>
          </CardContent>
        </Card>
      </div>

      <Tabs defaultValue="traffic" className="space-y-4">
        <TabsList>
          <TabsTrigger value="traffic">Bot Traffic</TabsTrigger>
          <TabsTrigger value="crawl">Crawl Stats</TabsTrigger>
          <TabsTrigger value="performance">Performance</TabsTrigger>
          <TabsTrigger value="issues">Issues</TabsTrigger>
        </TabsList>

        <TabsContent value="traffic" className="space-y-4">
          <Card>
            <CardHeader>
              <CardTitle>Bot Traffic Over Time</CardTitle>
              <CardDescription>
                SEO and LLM bot visits to your site
              </CardDescription>
            </CardHeader>
            <CardContent>
              <ResponsiveContainer width="100%" height={300}>
                <LineChart data={botTraffic?.timeline}>
                  <CartesianGrid strokeDasharray="3 3" />
                  <XAxis dataKey="date" />
                  <YAxis />
                  <Tooltip />
                  <Line 
                    type="monotone" 
                    dataKey="seo" 
                    stroke="#8884d8" 
                    name="SEO Bots"
                  />
                  <Line 
                    type="monotone" 
                    dataKey="llm" 
                    stroke="#82ca9d" 
                    name="LLM Bots"
                  />
                  <Line 
                    type="monotone" 
                    dataKey="blocked" 
                    stroke="#ff7c7c" 
                    name="Blocked"
                  />
                </LineChart>
              </ResponsiveContainer>
            </CardContent>
          </Card>

          <div className="grid gap-4 md:grid-cols-2">
            <Card>
              <CardHeader>
                <CardTitle>Top SEO Bots</CardTitle>
              </CardHeader>
              <CardContent>
                <div className="space-y-2">
                  {botTraffic?.topSeoBots.map((bot) => (
                    <div key={bot.name} className="flex items-center justify-between">
                      <span className="text-sm">{bot.name}</span>
                      <div className="flex items-center gap-2">
                        <span className="text-sm text-muted-foreground">
                          {bot.visits.toLocaleString()}
                        </span>
                        <Progress value={bot.percentage} className="w-20" />
                      </div>
                    </div>
                  ))}
                </div>
              </CardContent>
            </Card>

            <Card>
              <CardHeader>
                <CardTitle>Blocked Bots</CardTitle>
              </CardHeader>
              <CardContent>
                <div className="space-y-2">
                  {botTraffic?.blockedBots.map((bot) => (
                    <div key={bot.name} className="flex items-center justify-between">
                      <span className="text-sm">{bot.name}</span>
                      <Badge variant="destructive" className="text-xs">
                        {bot.attempts} attempts
                      </Badge>
                    </div>
                  ))}
                </div>
              </CardContent>
            </Card>
          </div>
        </TabsContent>

        <TabsContent value="crawl" className="space-y-4">
          <Card>
            <CardHeader>
              <CardTitle>Crawl Statistics</CardTitle>
            </CardHeader>
            <CardContent>
              <ResponsiveContainer width="100%" height={300}>
                <BarChart data={seoData?.crawlStats}>
                  <CartesianGrid strokeDasharray="3 3" />
                  <XAxis dataKey="pageType" />
                  <YAxis />
                  <Tooltip />
                  <Bar dataKey="crawled" fill="#8884d8" name="Crawled" />
                  <Bar dataKey="indexed" fill="#82ca9d" name="Indexed" />
                </BarChart>
              </ResponsiveContainer>
            </CardContent>
          </Card>
        </TabsContent>
      </Tabs>
    </div>
  );
}
```))
      {
        "data": $input.json('$.data'),
        "meta": {
          "requestId": "$context.requestId",
          "timestamp": "$context.requestTime",
          "path": "$context.path",
          "version": "1.0"
        }
      }
    `,
  },
  
  // Error response template
  error: {
    'application/json': `
      #set($inputRoot = $input.path('

### Language Switcher Component
```typescript
// components/features/i18n/language-switcher.tsx
'use client';

import { useLocale } from 'next-intl';
import { useRouter, usePathname } from 'next/navigation';
import { useTransition } from 'react';
import {
  Select,
  SelectContent,
  SelectItem,
  SelectTrigger,
  SelectValue,
} from '@/components/ui/select';
import { Globe } from 'lucide-react';

const languages = [
  { code: 'en', name: 'English', flag: '🇺🇸', dir: 'ltr' },
  { code: 'zh', name: '中文', flag: '🇨🇳', dir: 'ltr' },
  { code: 'es', name: 'Español', flag: '🇪🇸', dir: 'ltr' },
  { code: 'pt', name: 'Português', flag: '🇧🇷', dir: 'ltr' },
  { code: 'fr', name: 'Français', flag: '🇫🇷', dir: 'ltr' },
  { code: 'ja', name: '日本語', flag: '🇯🇵', dir: 'ltr' },
  { code: 'ar', name: 'العربية', flag: '🇸🇦', dir: 'rtl' },
  { code: 'hi', name: 'हिन्दी', flag: '🇮🇳', dir: 'ltr' },
  { code: 'ru', name: 'Русский', flag: '🇷🇺', dir: 'ltr' },
];

export function LanguageSwitcher() {
  const locale = useLocale();
  const router = useRouter();
  const pathname = usePathname();
  const [isPending, startTransition] = useTransition();

  const handleLanguageChange = (newLocale: string) => {
    // Save preference
    document.cookie = `preferred-locale=${newLocale};max-age=31536000;path=/;samesite=lax${
      process.env.NODE_ENV === 'production' ? ';secure' : ''
    }`;

    // Update HTML dir attribute for RTL languages
    const newLang = languages.find(l => l.code === newLocale);
    document.documentElement.dir = newLang?.dir || 'ltr';

    // Navigate to new locale
    startTransition(() => {
      const segments = pathname.split('/');
      segments[1] = newLocale;
      router.push(segments.join('/'));
    });
  };

  const currentLanguage = languages.find(lang => lang.code === locale);

  return (
    <Select 
      value={locale} 
      onValueChange={handleLanguageChange}
      disabled={isPending}
    >
      <SelectTrigger className="w-[180px]">
        <Globe className="mr-2 h-4 w-4" />
        <SelectValue>
          {currentLanguage && (
            <span className="flex items-center">
              <span className="mr-2">{currentLanguage.flag}</span>
              {currentLanguage.name}
            </span>
          )}
        </SelectValue>
      </SelectTrigger>
      <SelectContent>
        {languages.map((language) => (
          <SelectItem key={language.code} value={language.code}>
            <span className="flex items-center">
              <span className="mr-2 text-lg">{language.flag}</span>
              <span>{language.name}</span>
            </span>
          </SelectItem>
        ))}
      </SelectContent>
    </Select>
  );
}
```

## State Management & Data Flow

### Zustand Store Architecture
```typescript
// stores/use-app-store.ts
import { create } from 'zustand';
import { devtools, persist } from 'zustand/middleware';
import { immer } from 'zustand/middleware/immer';

interface AppState {
  // User state
  user: User | null;
  isAuthenticated: boolean;
  permissions: Permission[];
  
  // UI state
  sidebarOpen: boolean;
  theme: 'light' | 'dark' | 'system';
  locale: string;
  
  // Feature flags
  features: Record<string, boolean>;
  
  // Actions
  setUser: (user: User | null) => void;
  toggleSidebar: () => void;
  setTheme: (theme: 'light' | 'dark' | 'system') => void;
  setLocale: (locale: string) => void;
  updateFeatureFlag: (flag: string, enabled: boolean) => void;
  reset: () => void;
}

export const useAppStore = create<AppState>()(
  devtools(
    persist(
      immer((set) => ({
        // Initial state
        user: null,
        isAuthenticated: false,
        permissions: [],
        sidebarOpen: true,
        theme: 'system',
        locale: 'en',
        features: {},
        
        // Actions
        setUser: (user) =>
          set((state) => {
            state.user = user;
            state.isAuthenticated = !!user;
            state.permissions = user?.permissions || [];
          }),
          
        toggleSidebar: () =>
          set((state) => {
            state.sidebarOpen = !state.sidebarOpen;
          }),
          
        setTheme: (theme) =>
          set((state) => {
            state.theme = theme;
          }),
          
        setLocale: (locale) =>
          set((state) => {
            state.locale = locale;
          }),
          
        updateFeatureFlag: (flag, enabled) =>
          set((state) => {
            state.features[flag] = enabled;
          }),
          
        reset: () =>
          set((state) => {
            state.user = null;
            state.isAuthenticated = false;
            state.permissions = [];
            state.features = {};
          }),
      })),
      {
        name: 'app-storage',
        partialize: (state) => ({
          theme: state.theme,
          locale: state.locale,
          sidebarOpen: state.sidebarOpen,
        }),
      }
    )
  )
);

// Separate stores for different domains
export const useEventStore = create<EventState>()(
  devtools(
    immer((set) => ({
      // Event specific state and actions
    }))
  )
);

export const useAdminStore = create<AdminState>()(
  devtools(
    immer((set) => ({
      // Admin specific state and actions
    }))
  )
);
```

### React Query Configuration
```typescript
// lib/react-query.ts
import {
  QueryClient,
  QueryClientProvider,
  QueryCache,
  MutationCache,
} from '@tanstack/react-query';
import { toast } from 'sonner';

// Create a custom query client with global error handling
export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 5 * 60 * 1000, // 5 minutes
      gcTime: 10 * 60 * 1000, // 10 minutes (formerly cacheTime)
      retry: (failureCount, error: any) => {
        // Don't retry on 4xx errors
        if (error?.response?.status >= 400 && error?.response?.status < 500) {
          return false;
        }
        return failureCount < 3;
      },
      refetchOnWindowFocus: false,
    },
    mutations: {
      retry: false,
    },
  },
  queryCache: new QueryCache({
    onError: (error: any) => {
      // Global query error handling
      if (error?.response?.status === 401) {
        // Handle unauthorized
        window.location.href = '/login';
      } else if (error?.response?.status === 403) {
        toast.error('You do not have permission to perform this action');
      } else if (error?.response?.status >= 500) {
        toast.error('Server error. Please try again later.');
      }
    },
  }),
  mutationCache: new MutationCache({
    onError: (error: any) => {
      // Global mutation error handling
      const message = error?.response?.data?.message || 'An error occurred';
      toast.error(message);
    },
  }),
});

// Custom hooks for common queries
export function useUser() {
  return useQuery({
    queryKey: ['user'],
    queryFn: userService.getCurrentUser,
    staleTime: Infinity, // User data rarely changes
  });
}

export function useEvents(filters?: EventFilters) {
  return useQuery({
    queryKey: ['events', filters],
    queryFn: () => eventService.getEvents(filters),
  });
}

export function useEvent(eventId: string) {
  return useQuery({
    queryKey: ['events', eventId],
    queryFn: () => eventService.getEvent(eventId),
    enabled: !!eventId,
  });
}

// Optimistic updates example
export function useUpdateEvent() {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: eventService.updateEvent,
    onMutate: async (updatedEvent) => {
      // Cancel outgoing refetches
      await queryClient.cancelQueries({ queryKey: ['events', updatedEvent.id] });
      
      // Snapshot previous value
      const previousEvent = queryClient.getQueryData(['events', updatedEvent.id]);
      
      // Optimistically update
      queryClient.setQueryData(['events', updatedEvent.id], updatedEvent);
      
      // Return context with snapshot
      return { previousEvent };
    },
    onError: (err, updatedEvent, context) => {
      // Rollback on error
      if (context?.previousEvent) {
        queryClient.setQueryData(
          ['events', updatedEvent.id],
          context.previousEvent
        );
      }
    },
    onSettled: (data, error, variables) => {
      // Always refetch after error or success
      queryClient.invalidateQueries({ queryKey: ['events', variables.id] });
    },
  });
}
```

### Server State Synchronization
```typescript
// hooks/use-realtime-sync.ts
import { useEffect } from 'react';
import { useQueryClient } from '@tanstack/react-query';
import { io, Socket } from 'socket.io-client';

let socket: Socket;

export function useRealtimeSync() {
  const queryClient = useQueryClient();
  
  useEffect(() => {
    // Initialize WebSocket connection
    socket = io(process.env.NEXT_PUBLIC_WS_URL!, {
      auth: {
        token: localStorage.getItem('auth-token'),
      },
    });
    
    // Listen for real-time updates
    socket.on('event:updated', (data) => {
      // Update React Query cache
      queryClient.setQueryData(['events', data.id], data);
      
      // Invalidate list queries
      queryClient.invalidateQueries({ 
        queryKey: ['events'],
        exact: false,
      });
    });
    
    socket.on('event:deleted', (data) => {
      // Remove from cache
      queryClient.removeQueries({ queryKey: ['events', data.id] });
      
      // Invalidate list queries
      queryClient.invalidateQueries({ 
        queryKey: ['events'],
        exact: false,
      });
    });
    
    socket.on('notification', (notification) => {
      // Handle real-time notifications
      toast.info(notification.message);
    });
    
    return () => {
      socket.disconnect();
    };
  }, [queryClient]);
  
  return socket;
}

// Hook for subscribing to specific channels
export function useRealtimeChannel(channel: string, handler: (data: any) => void) {
  const socket = useRealtimeSync();
  
  useEffect(() => {
    if (!socket) return;
    
    socket.on(channel, handler);
    
    return () => {
      socket.off(channel, handler);
    };
  }, [socket, channel, handler]);
}
```

## Form Handling with Yup & Zod

### Yup Schema Examples
```typescript
// validations/event.schema.ts
import * as yup from 'yup';
import { isValidPhoneNumber } from 'libphonenumber-js';

// Custom validators
yup.addMethod(yup.string, 'phone', function(message = 'Invalid phone number') {
  return this.test('phone', message, function(value) {
    if (!value) return true;
    return isValidPhoneNumber(value);
  });
});

// Event creation schema
export const createEventSchema = yup.object({
  title: yup
    .string()
    .required('Event title is required')
    .min(5, 'Title must be at least 5 characters')
    .max(100, 'Title must not exceed 100 characters'),
    
  description: yup
    .string()
    .required('Description is required')
    .min(20, 'Description must be at least 20 characters')
    .max(5000, 'Description must not exceed 5000 characters'),
    
  date: yup
    .date()
    .required('Event date is required')
    .min(new Date(), 'Event date must be in the future'),
    
  endDate: yup
    .date()
    .nullable()
    .when('date', (date, schema) => {
      return date ? schema.min(date, 'End date must be after start date') : schema;
    }),
    
  venue: yup.object({
    name: yup.string().required('Venue name is required'),
    address: yup.string().required('Venue address is required'),
    city: yup.string().required('City is required'),
    country: yup.string().required('Country is required'),
    coordinates: yup.object({
      lat: yup.number().required().min(-90).max(90),
      lng: yup.number().required().min(-180).max(180),
    }).required('Venue coordinates are required'),
  }).required('Venue information is required'),
  
  capacity: yup
    .number()
    .required('Capacity is required')
    .positive('Capacity must be positive')
    .integer('Capacity must be a whole number')
    .max(1000000, 'Capacity seems unrealistic'),
    
  price: yup
    .number()
    .required('Price is required')
    .min(0, 'Price cannot be negative')
    .max(100000, 'Price seems too high'),
    
  currency: yup
    .string()
    .required('Currency is required')
    .oneOf(['USD', 'EUR', 'GBP', 'INR', 'AED', 'SAR']),
    
  categories: yup
    .array()
    .of(yup.string())
    .min(1, 'Select at least one category')
    .max(5, 'Maximum 5 categories allowed'),
    
  images: yup
    .array()
    .of(
      yup.object({
        url: yup.string().url('Invalid image URL'),
        alt: yup.string(),
      })
    )
    .min(1, 'At least one image is required')
    .max(10, 'Maximum 10 images allowed'),
    
  whatsappRSVP: yup.object({
    enabled: yup.boolean(),
    phoneNumber: yup.string().when('enabled', {
      is: true,
      then: (schema) => schema.required('WhatsApp number is required').phone(),
    }),
    message: yup.string().when('enabled', {
      is: true,
      then: (schema) => schema.required('RSVP message is required'),
    }),
  }),
  
  isPublished: yup.boolean(),
  
  tickets: yup.array().of(
    yup.object({
      name: yup.string().required('Ticket name is required'),
      price: yup.number().required().min(0),
      quantity: yup.number().required().positive().integer(),
      description: yup.string(),
    })
  ),
});

// User registration schema
export const userRegistrationSchema = yup.object({
  name: yup
    .string()
    .required('Name is required')
    .matches(/^[a-zA-Z\s]+$/, 'Name can only contain letters and spaces'),
    
  email: yup
    .string()
    .required('Email is required')
    .email('Invalid email address'),
    
  phoneNumber: yup
    .string()
    .required('Phone number is required')
    .phone('Invalid phone number'),
    
  password: yup
    .string()
    .required('Password is required')
    .min(8, 'Password must be at least 8 characters')
    .matches(
      /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]/,
      'Password must contain uppercase, lowercase, number and special character'
    ),
    
  confirmPassword: yup
    .string()
    .required('Please confirm your password')
    .oneOf([yup.ref('password')], 'Passwords must match'),
    
  acceptTerms: yup
    .boolean()
    .required('You must accept the terms and conditions')
    .oneOf([true], 'You must accept the terms and conditions'),
});

// Dynamic field validation
export const createDynamicSchema = (fields: FormField[]) => {
  const shape: Record<string, any> = {};
  
  fields.forEach((field) => {
    let validator = yup.string();
    
    if (field.required) {
      validator = validator.required(`${field.label} is required`);
    }
    
    if (field.type === 'email') {
      validator = validator.email('Invalid email');
    }
    
    if (field.type === 'number') {
      validator = yup.number();
      if (field.min !== undefined) {
        validator = validator.min(field.min);
      }
      if (field.max !== undefined) {
        validator = validator.max(field.max);
      }
    }
    
    if (field.pattern) {
      validator = validator.matches(field.pattern, field.patternMessage);
    }
    
    shape[field.name] = validator;
  });
  
  return yup.object(shape);
};
```

### Zod Schema Examples (Secondary)
```typescript
// validations/admin.schema.ts
import { z } from 'zod';

// Admin creation schema using Zod
export const createAdminSchema = z.object({
  name: z.string().min(2, 'Name must be at least 2 characters'),
  email: z.string().email('Invalid email address'),
  phoneNumber: z.string().regex(/^\+?[1-9]\d{1,14}$/, 'Invalid phone number'),
  type: z.enum([
    'SUPER_ADMIN',
    'PLATFORM_ADMIN',
    'REGIONAL_ADMIN',
    'COUNTRY_ADMIN',
    'CITY_ADMIN',
    'CATEGORY_ADMIN',
    'CONTENT_ADMIN',
    'SUPPORT_ADMIN',
    'FINANCE_ADMIN',
    'MARKETING_ADMIN',
    'DEVELOPER_ADMIN',
  ]),
  permissions: z.array(z.object({
    resource: z.string(),
    actions: z.array(z.enum(['READ', 'WRITE', 'UPDATE', 'DELETE', 'PUBLISH', 'EXPORT', 'CONFIGURE'])),
    conditions: z.record(z.any()).optional(),
  })),
  geographyAccess: z.array(z.object({
    type: z.enum(['GLOBAL', 'REGION', 'COUNTRY', 'CITY']),
    identifier: z.string(),
    permissions: z.array(z.string()),
  })),
  featureAccess: z.array(z.object({
    microservice: z.string(),
    permissions: z.array(z.string()),
    limits: z.record(z.number()).optional(),
  })),
});

// API request validation
export const apiRequestSchema = z.object({
  method: z.enum(['GET', 'POST', 'PUT', 'DELETE', 'PATCH']),
  endpoint: z.string().url(),
  headers: z.record(z.string()).optional(),
  body: z.any().optional(),
  queryParams: z.record(z.string()).optional(),
});

// Type inference from Zod schemas
export type CreateAdminInput = z.infer<typeof createAdminSchema>;
export type APIRequest = z.infer<typeof apiRequestSchema>;
```

### React Hook Form Integration
```typescript
// components/features/events/create-event-form.tsx
'use client';

import { useForm } from 'react-hook-form';
import { yupResolver } from '@hookform/resolvers/yup';
import { createEventSchema } from '@/validations/event.schema';
import { Form, FormControl, FormDescription, FormField, FormItem, FormLabel, FormMessage } from '@/components/ui/form';
import { Input } from '@/components/ui/input';
import { Textarea } from '@/components/ui/textarea';
import { Button } from '@/components/ui/button';
import { Calendar } from '@/components/ui/calendar';
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from '@/components/ui/select';
import { Switch } from '@/components/ui/switch';
import { toast } from 'sonner';

type FormData = yup.InferType<typeof createEventSchema>;

export function CreateEventForm() {
  const form = useForm<FormData>({
    resolver: yupResolver(createEventSchema),
    defaultValues: {
      title: '',
      description: '',
      date: undefined,
      endDate: undefined,
      venue: {
        name: '',
        address: '',
        city: '',
        country: '',
        coordinates: { lat: 0, lng: 0 },
      },
      capacity: 100,
      price: 0,
      currency: 'USD',
      categories: [],
      images: [],
      whatsappRSVP: {
        enabled: false,
        phoneNumber: '',
        message: '',
      },
      isPublished: false,
      tickets: [],
    },
  });

  const { watch, setValue } = form;
  const whatsappEnabled = watch('whatsappRSVP.enabled');

  const onSubmit = async (data: FormData) => {
    try {
      const response = await eventService.createEvent(data);
      toast.success('Event created successfully!');
      router.push(`/events/${response.id}`);
    } catch (error) {
      toast.error('Failed to create event');
    }
  };

  return (
    <Form {...form}>
      <form onSubmit={form.handleSubmit(onSubmit)} className="space-y-8">
        <FormField
          control={form.control}
          name="title"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Event Title</FormLabel>
              <FormControl>
                <Input placeholder="Summer Music Festival" {...field} />
              </FormControl>
              <FormDescription>
                Choose a catchy title for your event
              </FormDescription>
              <FormMessage />
            </FormItem>
          )}
        />

        <FormField
          control={form.control}
          name="description"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Description</FormLabel>
              <FormControl>
                <Textarea
                  placeholder="Describe your event..."
                  className="min-h-[120px]"
                  {...field}
                />
              </FormControl>
              <FormMessage />
            </FormItem>
          )}
        />

        <div className="grid gap-4 md:grid-cols-2">
          <FormField
            control={form.control}
            name="date"
            render={({ field }) => (
              <FormItem className="flex flex-col">
                <FormLabel>Start Date</FormLabel>
                <Calendar
                  mode="single"
                  selected={field.value}
                  onSelect={field.onChange}
                  disabled={(date) => date < new Date()}
                  initialFocus
                />
                <FormMessage />
              </FormItem>
            )}
          />

          <FormField
            control={form.control}
            name="endDate"
            render={({ field }) => (
              <FormItem className="flex flex-col">
                <FormLabel>End Date (Optional)</FormLabel>
                <Calendar
                  mode="single"
                  selected={field.value}
                  onSelect={field.onChange}
                  disabled={(date) => date < watch('date')}
                />
                <FormMessage />
              </FormItem>
            )}
          />
        </div>

        <div className="space-y-4">
          <h3 className="text-lg font-semibold">Venue Information</h3>
          
          <FormField
            control={form.control}
            name="venue.name"
            render={({ field }) => (
              <FormItem>
                <FormLabel>Venue Name</FormLabel>
                <FormControl>
                  <Input placeholder="Madison Square Garden" {...field} />
                </FormControl>
                <FormMessage />
              </FormItem>
            )}
          />

          <FormField
            control={form.control}
            name="venue.address"
            render={({ field }) => (
              <FormItem>
                <FormLabel>Address</FormLabel>
                <FormControl>
                  <Input placeholder="4 Pennsylvania Plaza" {...field} />
                </FormControl>
                <FormMessage />
              </FormItem>
            )}
          />

          <div className="grid gap-4 md:grid-cols-2">
            <FormField
              control={form.control}
              name="venue.city"
              render={({ field }) => (
                <FormItem>
                  <FormLabel>City</FormLabel>
                  <FormControl>
                    <Input placeholder="New York" {...field} />
                  </FormControl>
                  <FormMessage />
                </FormItem>
              )}
            />

            <FormField
              control={form.control}
              name="venue.country"
              render={({ field }) => (
                <FormItem>
                  <FormLabel>Country</FormLabel>
                  <Select onValueChange={field.onChange} defaultValue={field.value}>
                    <FormControl>
                      <SelectTrigger>
                        <SelectValue placeholder="Select country" />
                      </SelectTrigger>
                    </FormControl>
                    <SelectContent>
                      <SelectItem value="US">United States</SelectItem>
                      <SelectItem value="GB">United Kingdom</SelectItem>
                      <SelectItem value="IN">India</SelectItem>
                      <SelectItem value="AE">UAE</SelectItem>
                      {/* Add more countries */}
                    </SelectContent>
                  </Select>
                  <FormMessage />
                </FormItem>
              )}
            />
          </div>
        </div>

        <div className="grid gap-4 md:grid-cols-3">
          <FormField
            control={form.control}
            name="capacity"
            render={({ field }) => (
              <FormItem>
                <FormLabel>Capacity</FormLabel>
                <FormControl>
                  <Input
                    type="number"
                    placeholder="500"
                    {...field}
                    onChange={(e) => field.onChange(parseInt(e.target.value))}
                  />
                </FormControl>
                <FormMessage />
              </FormItem>
            )}
          />

          <FormField
            control={form.control}
            name="price"
            render={({ field }) => (
              <FormItem>
                <FormLabel>Price</FormLabel>
                <FormControl>
                  <Input
                    type="number"
                    placeholder="0"
                    {...field}
                    onChange={(e) => field.onChange(parseFloat(e.target.value))}
                  />
                </FormControl>
                <FormMessage />
              </FormItem>
            )}
          />

          <FormField
            control={form.control}
            name="currency"
            render={({ field }) => (
              <FormItem>
                <FormLabel>Currency</FormLabel>
                <Select onValueChange={field.onChange} defaultValue={field.value}>
                  <FormControl>
                    <SelectTrigger>
                      <SelectValue />
                    </SelectTrigger>
                  </FormControl>
                  <SelectContent>
                    <SelectItem value="USD">USD</SelectItem>
                    <SelectItem value="EUR">EUR</SelectItem>
                    <SelectItem value="GBP">GBP</SelectItem>
                    <SelectItem value="INR">INR</SelectItem>
                    <SelectItem value="AED">AED</SelectItem>
                    <SelectItem value="SAR">SAR</SelectItem>
                  </SelectContent>
                </Select>
                <FormMessage />
              </FormItem>
            )}
          />
        </div>

        <div className="space-y-4">
          <FormField
            control={form.control}
            name="whatsappRSVP.enabled"
            render={({ field }) => (
              <FormItem className="flex flex-row items-center justify-between rounded-lg border p-4">
                <div className="space-y-0.5">
                  <FormLabel className="text-base">
                    WhatsApp RSVP
                  </FormLabel>
                  <FormDescription>
                    Allow attendees to RSVP via WhatsApp
                  </FormDescription>
                </div>
                <FormControl>
                  <Switch
                    checked={field.value}
                    onCheckedChange={field.onChange}
                  />
                </FormControl>
              </FormItem>
            )}
          />

          {whatsappEnabled && (
            <>
              <FormField
                control={form.control}
                name="whatsappRSVP.phoneNumber"
                render={({ field }) => (
                  <FormItem>
                    <FormLabel>WhatsApp Number</FormLabel>
                    <FormControl>
                      <Input placeholder="+1234567890" {...field} />
                    </FormControl>
                    <FormDescription>
                      Include country code
                    </FormDescription>
                    <FormMessage />
                  </FormItem>
                )}
              />

              <FormField
                control={form.control}
                name="whatsappRSVP.message"
                render={({ field }) => (
                  <FormItem>
                    <FormLabel>RSVP Message Template</FormLabel>
                    <FormControl>
                      <Textarea
                        placeholder="Hi! I'd like to RSVP for {event_name}"
                        {...field}
                      />
                    </FormControl>
                    <FormDescription>
                      Use {'{event_name}'} as placeholder
                    </FormDescription>
                    <FormMessage />
                  </FormItem>
                )}
              />
            </>
          )}
        </div>

        <div className="flex justify-end gap-4">
          <Button variant="outline" type="button">
            Save as Draft
          </Button>
          <Button type="submit">
            Create Event
          </Button>
        </div>
      </form>
    </Form>
  );
}
```

## Storybook Integration

### Storybook Configuration
```typescript
// .storybook/main.ts
import type { StorybookConfig } from '@storybook/nextjs';

const config: StorybookConfig = {
  stories: [
    '../stories/**/*.stories.@(js|jsx|ts|tsx|mdx)',
    '../src/**/*.stories.@(js|jsx|ts|tsx|mdx)',
  ],
  addons: [
    '@storybook/addon-essentials',
    '@storybook/addon-interactions',
    '@storybook/addon-links',
    '@storybook/addon-a11y',
    '@storybook/addon-viewport',
    '@storybook/addon-docs',
    '@storybook/addon-controls',
    'storybook-addon-next-router',
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
```

### Storybook Preview Configuration
```typescript
// .storybook/preview.tsx
import React from 'react';
import type { Preview } from '@storybook/react';
import { Inter } from 'next/font/google';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { NextIntlClientProvider } from 'next-intl';
import { ThemeProvider } from '@/components/theme-provider';
import '../src/styles/globals.css';

const inter = Inter({ subsets: ['latin'] });

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      retry: false,
      refetchOnWindowFocus: false,
    },
  },
});

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
    viewport: {
      viewports: {
        mobile: {
          name: 'Mobile',
          styles: {
            width: '375px',
            height: '667px',
          },
        },
        tablet: {
          name: 'Tablet',
          styles: {
            width: '768px',
            height: '1024px',
          },
        },
        desktop: {
          name: 'Desktop',
          styles: {
            width: '1440px',
            height: '900px',
          },
        },
      },
    },
  },
  decorators: [
    (Story) => (
      <QueryClientProvider client={queryClient}>
        <NextIntlClientProvider locale="en" messages={{}}>
          <ThemeProvider
            attribute="class"
            defaultTheme="system"
            enableSystem
            disableTransitionOnChange
          >
            <div className={inter.className}>
              <Story />
            </div>
          </ThemeProvider>
        </NextIntlClientProvider>
      </QueryClientProvider>
    ),
  ],
  globalTypes: {
    locale: {
      name: 'Locale',
      description: 'Internationalization locale',
      defaultValue: 'en',
      toolbar: {
        icon: 'globe',
        items: [
          { value: 'en', title: 'English' },
          { value: 'es', title: 'Spanish' },
          { value: 'fr', title: 'French' },
          { value: 'ar', title: 'Arabic' },
          { value: 'zh', title: 'Chinese' },
        ],
        showName: true,
      },
    },
    theme: {
      name: 'Theme',
      description: 'Theme for components',
      defaultValue: 'light',
      toolbar: {
        icon: 'circlehollow',
        items: ['light', 'dark', 'system'],
        showName: true,
      },
    },
  },
};

export default preview;
```

### Component Story Examples
```typescript
// stories/components/ui/button.stories.tsx
import type { Meta, StoryObj } from '@storybook/react';
import { Button } from '@/components/ui/button';
import { Mail, Loader2 } from 'lucide-react';

const meta = {
  title: 'UI/Button',
  component: Button,
  parameters: {
    layout: 'centered',
    docs: {
      description: {
        component: 'A flexible button component with multiple variants and sizes.',
      },
    },
  },
  tags: ['autodocs'],
  argTypes: {
    variant: {
      control: { type: 'select' },
      options: ['default', 'destructive', 'outline', 'secondary', 'ghost', 'link'],
      description: 'Button variant style',
    },
    size: {
      control: { type: 'select' },
      options: ['default', 'sm', 'lg', 'icon'],
      description: 'Button size',
    },
    disabled: {
      control: 'boolean',
      description: 'Disabled state',
    },
    asChild: {
      control: 'boolean',
      description: 'Render as child component',
    },
  },
} satisfies Meta<typeof Button>;

export default meta;
type Story = StoryObj<typeof meta>;

export const Default: Story = {
  args: {
    children: 'Button',
  },
};

export const Variants: Story = {
  render: () => (
    <div className="flex flex-wrap gap-4">
      <Button variant="default">Default</Button>
      <Button variant="destructive">Destructive</Button>
      <Button variant="outline">Outline</Button>
      <Button variant="secondary">Secondary</Button>
      <Button variant="ghost">Ghost</Button>
      <Button variant="link">Link</Button>
    </div>
  ),
};

export const Sizes: Story = {
  render: () => (
    <div className="flex items-center gap-4">
      <Button size="sm">Small</Button>
      <Button size="default">Default</Button>
      <Button size="lg">Large</Button>
      <Button size="icon">
        <Mail className="h-4 w-4" />
      </Button>
    </div>
  ),
};

export const WithIcon: Story = {
  render: () => (
    <div className="flex gap-4">
      <Button>
        <Mail className="mr-2 h-4 w-4" />
        Login with Email
      </Button>
      <Button variant="outline">
        <Mail className="mr-2 h-4 w-4" />
        Contact Us
      </Button>
    </div>
  ),
};

export const Loading: Story = {
  render: () => (
    <div className="flex gap-4">
      <Button disabled>
        <Loader2 className="mr-2 h-4 w-4 animate-spin" />
        Please wait
      </Button>
      <Button variant="outline" disabled>
        <Loader2 className="mr-2 h-4 w-4 animate-spin" />
        Loading...
      </Button>
    </div>
  ),
};

export const Disabled: Story = {
  render: () => (
    <div className="flex gap-4">
      <Button disabled>Disabled Default</Button>
      <Button variant="outline" disabled>
        Disabled Outline
      </Button>
      <Button variant="secondary" disabled>
        Disabled Secondary
      </Button>
    </div>
  ),
};

export const AsChild: Story = {
  render: () => (
    <Button asChild>
      <a href="https://eventfulhq.com" target="_blank" rel="noopener noreferrer">
        Open EventfulHQ
      </a>
    </Button>
  ),
};
```

### Feature Component Story
```typescript
// stories/features/events/event-card.stories.tsx
import type { Meta, StoryObj } from '@storybook/react';
import { EventCard } from '@/components/features/events/event-card';
import { mockEvent } from '@/test/mocks/event';

const meta = {
  title: 'Features/Events/EventCard',
  component: EventCard,
  parameters: {
    layout: 'padded',
  },
  tags: ['autodocs'],
  decorators: [
    (Story) => (
      <div className="max-w-md">
        <Story />
      </div>
    ),
  ],
} satisfies Meta<typeof EventCard>;

export default meta;
type Story = StoryObj<typeof meta>;

export const Default: Story = {
  args: {
    event: mockEvent,
  },
};

export const WithWhatsAppRSVP: Story = {
  args: {
    event: {
      ...mockEvent,
      whatsappRSVP: {
        enabled: true,
        phoneNumber: '+1234567890',
        message: 'Hi! I would like to RSVP for {event_name}',
      },
    },
  },
};

export const SoldOut: Story = {
  args: {
    event: {
      ...mockEvent,
      isSoldOut: true,
    },
  },
};

export const FreeEvent: Story = {
  args: {
    event: {
      ...mockEvent,
      price: 0,
    },
  },
};

export const Loading: Story = {
  args: {
    event: mockEvent,
    isLoading: true,
  },
};

export const Interactive: Story = {
  args: {
    event: mockEvent,
    onSelect: (event) => {
      console.log('Selected event:', event);
    },
    onShare: (event) => {
      console.log('Share event:', event);
    },
  },
  play: async ({ canvasElement, step }) => {
    const canvas = within(canvasElement);
    
    await step('Hover over card', async () => {
      await userEvent.hover(canvas.getByRole('article'));
    });
    
    await step('Click share button', async () => {
      await userEvent.click(canvas.getByRole('button', { name: /share/i }));
    });
  },
};
```

## API Integration & Code Generation

### OpenAPI Code Generation Setup
```typescript
// orval.config.ts
import { defineConfig } from 'orval';

export default defineConfig({
  eventfulhq: {
    input: {
      target: './openapi/eventfulhq.yaml',
      validation: true,
    },
    output: {
      mode: 'tags-split',
      target: './src/services/generated',
      schemas: './src/types/generated',
      client: 'react-query',
      httpClient: 'axios',
      override: {
        mutator: {
          path: './src/lib/api/client.ts',
          name: 'apiClient',
        },
        query: {
          useQuery: true,
          useInfinite: true,
          useInfiniteQueryParam: 'cursor',
          options: {
            staleTime: 5 * 60 * 1000,
          },
        },
        operations: {
          listEvents: {
            query: {
              useInfinite: true,
              useInfiniteQueryParam: 'page',
            },
          },
        },
      },
    },
    hooks: {
      afterAllFilesWrite: 'prettier --write',
    },
  },
  // Additional services
  whatsapp: {
    input: {
      target: './openapi/whatsapp-service.yaml',
    },
    output: {
      target: './src/services/generated/whatsapp',
      client: 'react-query',
    },
  },
  admin: {
    input: {
      target: './openapi/admin-service.yaml',
    },
    output: {
      target: './src/services/generated/admin',
      client: 'react-query',
    },
  },
});
```

### API Client Configuration
```typescript
// lib/api/client.ts
import axios, { AxiosError, AxiosRequestConfig } from 'axios';
import { toast } from 'sonner';

const BASE_URL = process.env.NEXT_PUBLIC_API_URL || 'https://api.eventfulhq.com';

// Create axios instance
export const apiClient = axios.create({
  baseURL: BASE_URL,
  timeout: 30000,
  headers: {
    'Content-Type': 'application/json',
  },
});

// Request interceptor
apiClient.interceptors.request.use(
  (config) => {
    // Add auth token
    const token = localStorage.getItem('auth-token');
    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }
    
    // Add request ID for tracing
    config.headers['X-Request-ID'] = generateRequestId();
    
    // Add locale
    const locale = localStorage.getItem('locale') || 'en';
    config.headers['Accept-Language'] = locale;
    
    // Add region
    const region = localStorage.getItem('preferred-region') || 'global';
    config.headers['X-Region'] = region;
    
    return config;
  },
  (error) => {
    return Promise.reject(error);
  }
);

// Response interceptor
apiClient.interceptors.response.use(
  (response) => {
    // Log successful responses in development
    if (process.env.NODE_ENV === 'development') {
      console.log('API Response:', {
        url: response.config.url,
        status: response.status,
        data: response.data,
      });
    }
    
    return response;
  },
  async (error: AxiosError) => {
    const originalRequest = error.config as AxiosRequestConfig & { _retry?: boolean };
    
    // Handle 401 - Token expired
    if (error.response?.status === 401 && !originalRequest._retry) {
      originalRequest._retry = true;
      
      try {
        const refreshToken = localStorage.getItem('refresh-token');
        const response = await axios.post(`${BASE_URL}/auth/refresh`, {
          refreshToken,
        });
        
        const { accessToken, refreshToken: newRefreshToken } = response.data;
        localStorage.setItem('auth-token', accessToken);
        localStorage.setItem('refresh-token', newRefreshToken);
        
        // Retry original request
        return apiClient(originalRequest);
      } catch (refreshError) {
        // Refresh failed, redirect to login
        localStorage.clear();
        window.location.href = '/login';
        return Promise.reject(refreshError);
      }
    }
    
    // Handle other errors
    handleApiError(error);
    
    return Promise.reject(error);
  }
);

// Error handler
function handleApiError(error: AxiosError<any>) {
  if (!error.response) {
    // Network error
    toast.error('Network error. Please check your connection.');
    return;
  }
  
  const { status, data } = error.response;
  
  switch (status) {
    case 400:
      toast.error(data.message || 'Invalid request');
      break;
    case 403:
      toast.error('You do not have permission to perform this action');
      break;
    case 404:
      toast.error('Resource not found');
      break;
    case 429:
      toast.error('Too many requests. Please try again later.');
      break;
    case 500:
      toast.error('Server error. Please try again later.');
      break;
    default:
      toast.error(data.message || 'An error occurred');
  }
}

// Custom API client wrapper for mutations
export async function apiMutator<T = any>(
  config: AxiosRequestConfig
): Promise<T> {
  const response = await apiClient(config);
  return response.data;
}

// Generate unique request ID
function generateRequestId(): string {
  return `${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;
}

// Typed API endpoints
export const API_ENDPOINTS = {
  // Auth
  auth: {
    login: '/auth/login',
    register: '/auth/register',
    refresh: '/auth/refresh',
    logout: '/auth/logout',
    whatsappOtp: '/auth/whatsapp-otp',
    verifyOtp: '/auth/verify-otp',
  },
  
  // Events
  events: {
    list: '/events',
    detail: (id: string) => `/events/${id}`,
    create: '/events',
    update: (id: string) => `/events/${id}`,
    delete: (id: string) => `/events/${id}`,
    publish: (id: string) => `/events/${id}/publish`,
    attendees: (id: string) => `/events/${id}/attendees`,
  },
  
  // RFP
  rfp: {
    list: '/rfp',
    create: '/rfp',
    detail: (id: string) => `/rfp/${id}`,
    proposals: (id: string) => `/rfp/${id}/proposals`,
    submitProposal: (id: string) => `/rfp/${id}/proposals`,
  },
  
  // Admin
  admin: {
    admins: '/admin/admins',
    features: '/admin/features',
    permissions: '/admin/permissions',
    geography: '/admin/geography',
    audit: '/admin/audit-logs',
  },
  
  // Developer
  developer: {
    apiKeys: '/developer/api-keys',
    usage: '/developer/usage',
    webhooks: '/developer/webhooks',
    docs: '/developer/docs',
  },
} as const;
```

### Service Layer Architecture
```typescript
// services/base-service.ts
import { apiClient } from '@/lib/api/client';
import { AxiosRequestConfig } from 'axios';

export interface PaginatedResponse<T> {
  data: T[];
  meta: {
    total: number;
    page: number;
    limit: number;
    totalPages: number;
  };
}

export interface QueryParams {
  page?: number;
  limit?: number;
  sort?: string;
  order?: 'asc' | 'desc';
  search?: string;
  filters?: Record<string, any>;
}

export abstract class BaseService {
  protected endpoint: string;

  constructor(endpoint: string) {
    this.endpoint = endpoint;
  }

  protected async get<T>(
    path: string = '',
    config?: AxiosRequestConfig
  ): Promise<T> {
    const response = await apiClient.get<T>(`${this.endpoint}${path}`, config);
    return response.data;
  }

  protected async post<T, D = any>(
    path: string = '',
    data?: D,
    config?: AxiosRequestConfig
  ): Promise<T> {
    const response = await apiClient.post<T>(`${this.endpoint}${path}`, data, config);
    return response.data;
  }

  protected async put<T, D = any>(
    path: string = '',
    data?: D,
    config?: AxiosRequestConfig
  ): Promise<T> {
    const response = await apiClient.put<T>(`${this.endpoint}${path}`, data, config);
    return response.data;
  }

  protected async patch<T, D = any>(
    path: string = '',
    data?: D,
    config?: AxiosRequestConfig
  ): Promise<T> {
    const response = await apiClient.patch<T>(`${this.endpoint}${path}`, data, config);
    return response.data;
  }

  protected async delete<T>(
    path: string = '',
    config?: AxiosRequestConfig
  ): Promise<T> {
    const response = await apiClient.delete<T>(`${this.endpoint}${path}`, config);
    return response.data;
  }

  protected buildQueryString(params: QueryParams): string {
    const query = new URLSearchParams();
    
    if (params.page) query.append('page', params.page.toString());
    if (params.limit) query.append('limit', params.limit.toString());
    if (params.sort) query.append('sort', params.sort);
    if (params.order) query.append('order', params.order);
    if (params.search) query.append('search', params.search);
    
    if (params.filters) {
      Object.entries(params.filters).forEach(([key, value]) => {
        if (value !== undefined && value !== null && value !== '') {
          query.append(key, value.toString());
        }
      });
    }
    
    return query.toString() ? `?${query.toString()}` : '';
  }
}

// Event service implementation
export class EventService extends BaseService {
  constructor() {
    super('/events');
  }

  async getEvents(params?: QueryParams): Promise<PaginatedResponse<Event>> {
    const queryString = this.buildQueryString(params || {});
    return this.get<PaginatedResponse<Event>>(queryString);
  }

  async getEvent(id: string): Promise<Event> {
    return this.get<Event>(`/${id}`);
  }

  async createEvent(data: CreateEventInput): Promise<Event> {
    return this.post<Event>('', data);
  }

  async updateEvent(id: string, data: UpdateEventInput): Promise<Event> {
    return this.put<Event>(`/${id}`, data);
  }

  async deleteEvent(id: string): Promise<void> {
    return this.delete<void>(`/${id}`);
  }

  async publishEvent(id: string): Promise<Event> {
    return this.post<Event>(`/${id}/publish`);
  }

  async unpublishEvent(id: string): Promise<Event> {
    return this.post<Event>(`/${id}/unpublish`);
  }

  async getEventAttendees(
    id: string,
    params?: QueryParams
  ): Promise<PaginatedResponse<Attendee>> {
    const queryString = this.buildQueryString(params || {});
    return this.get<PaginatedResponse<Attendee>>(`/${id}/attendees${queryString}`);
  }

  async sendWhatsAppInvites(
    id: string,
    data: { phoneNumbers: string[]; message: string }
  ): Promise<void> {
    return this.post<void>(`/${id}/whatsapp-invites`, data);
  }
}

// Service registry
export const eventService = new EventService();
export const userService = new UserService();
export const adminService = new AdminService();
export const developerService = new DeveloperService();
export const rfpService = new RFPService();
export const analyticsService = new AnalyticsService();
```

## Authentication & Security Patterns

### WhatsApp OTP Authentication
```typescript
// lib/auth/whatsapp-auth.ts
import { apiClient } from '@/lib/api/client';
import { z } from 'zod';

// Validation schemas
const phoneSchema = z.string().regex(/^\+?[1-9]\d{1,14}$/, 'Invalid phone number');
const otpSchema = z.string().length(6, 'OTP must be 6 digits');

export interface WhatsAppAuthState {
  phoneNumber: string;
  sessionId: string;
  expiresAt: Date;
  attempts: number;
  maxAttempts: number;
}

export class WhatsAppAuth {
  private static SESSION_KEY = 'whatsapp-auth-session';
  
  static async sendOTP(phoneNumber: string): Promise<WhatsAppAuthState> {
    // Validate phone number
    const validatedPhone = phoneSchema.parse(phoneNumber);
    
    try {
      const response = await apiClient.post('/auth/whatsapp-otp', {
        phoneNumber: validatedPhone,
      });
      
      const session: WhatsAppAuthState = {
        phoneNumber: validatedPhone,
        sessionId: response.data.sessionId,
        expiresAt: new Date(response.data.expiresAt),
        attempts: 0,
        maxAttempts: response.data.maxAttempts || 3,
      };
      
      // Store session
      sessionStorage.setItem(this.SESSION_KEY, JSON.stringify(session));
      
      return session;
    } catch (error) {
      throw new Error('Failed to send OTP');
    }
  }
  
  static async verifyOTP(otp: string): Promise<AuthResponse> {
    // Get session
    const sessionStr = sessionStorage.getItem(this.SESSION_KEY);
    if (!sessionStr) {
      throw new Error('No active OTP session');
    }
    
    const session: WhatsAppAuthState = JSON.parse(sessionStr);
    
    // Check expiration
    if (new Date() > new Date(session.expiresAt)) {
      sessionStorage.removeItem(this.SESSION_KEY);
      throw new Error('OTP session expired');
    }
    
    // Validate OTP format
    const validatedOTP = otpSchema.parse(otp);
    
    try {
      const response = await apiClient.post('/auth/verify-otp', {
        sessionId: session.sessionId,
        otp: validatedOTP,
      });
      
      // Clear session
      sessionStorage.removeItem(this.SESSION_KEY);
      
      // Store tokens
      const { accessToken, refreshToken, user } = response.data;
      localStorage.setItem('auth-token', accessToken);
      localStorage.setItem('refresh-token', refreshToken);
      localStorage.setItem('user', JSON.stringify(user));
      
      return response.data;
    } catch (error: any) {
      // Update attempts
      session.attempts += 1;
      
      if (session.attempts >= session.maxAttempts) {
        sessionStorage.removeItem(this.SESSION_KEY);
        throw new Error('Maximum OTP attempts exceeded');
      }
      
      sessionStorage.setItem(this.SESSION_KEY, JSON.stringify(session));
      
      throw new Error(
        error.response?.data?.message || 
        `Invalid OTP. ${session.maxAttempts - session.attempts} attempts remaining`
      );
    }
  }
  
  static async resendOTP(): Promise<WhatsAppAuthState> {
    const sessionStr = sessionStorage.getItem(this.SESSION_KEY);
    if (!sessionStr) {
      throw new Error('No active OTP session');
    }
    
    const session: WhatsAppAuthState = JSON.parse(sessionStr);
    return this.sendOTP(session.phoneNumber);
  }
  
  static clearSession(): void {
    sessionStorage.removeItem(this.SESSION_KEY);
  }
}

// WhatsApp OTP Component
export function WhatsAppOTPModal({ 
  isOpen, 
  onClose, 
  phoneNumber,
  onSuccess 
}: WhatsAppOTPModalProps) {
  const [otp, setOtp] = useState(['', '', '', '', '', '']);
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);
  const [session, setSession] = useState<WhatsAppAuthState | null>(null);
  const inputRefs = useRef<(HTMLInputElement | null)[]>([]);
  
  useEffect(() => {
    if (isOpen && phoneNumber) {
      sendOTP();
    }
  }, [isOpen, phoneNumber]);
  
  const sendOTP = async () => {
    setIsLoading(true);
    setError(null);
    
    try {
      const session = await WhatsAppAuth.sendOTP(phoneNumber);
      setSession(session);
      toast.success('OTP sent to your WhatsApp');
    } catch (error: any) {
      setError(error.message);
      toast.error('Failed to send OTP');
    } finally {
      setIsLoading(false);
    }
  };
  
  const handleOTPChange = (index: number, value: string) => {
    if (!/^\d*$/.test(value)) return;
    
    const newOtp = [...otp];
    newOtp[index] = value;
    setOtp(newOtp);
    
    // Auto-focus next input
    if (value && index < 5) {
      inputRefs.current[index + 1]?.focus();
    }
    
    // Auto-submit if all digits entered
    if (newOtp.every(digit => digit) && newOtp.join('').length === 6) {
      verifyOTP(newOtp.join(''));
    }
  };
  
  const handleKeyDown = (index: number, e: KeyboardEvent<HTMLInputElement>) => {
    if (e.key === 'Backspace' && !otp[index] && index > 0) {
      inputRefs.current[index - 1]?.focus();
    }
  };
  
  const verifyOTP = async (otpCode?: string) => {
    const code = otpCode || otp.join('');
    if (code.length !== 6) {
      setError('Please enter all 6 digits');
      return;
    }
    
    setIsLoading(true);
    setError(null);
    
    try {
      const response = await WhatsAppAuth.verifyOTP(code);
      toast.success('Successfully authenticated!');
      onSuccess(response);
      onClose();
    } catch (error: any) {
      setError(error.message);
      setOtp(['', '', '', '', '', '']);
      inputRefs.current[0]?.focus();
    } finally {
      setIsLoading(false);
    }
  };
  
  return (
    <Dialog open={isOpen} onOpenChange={onClose}>
      <DialogContent className="sm:max-w-md">
        <DialogHeader>
          <DialogTitle>Enter WhatsApp OTP</DialogTitle>
          <DialogDescription>
            We've sent a 6-digit code to {phoneNumber}
          </DialogDescription>
        </DialogHeader>
        
        <div className="space-y-6 py-4">
          <div className="flex justify-center gap-2">
            {otp.map((digit, index) => (
              <Input
                key={index}
                ref={(el) => (inputRefs.current[index] = el)}
                type="text"
                inputMode="numeric"
                maxLength={1}
                value={digit}
                onChange={(e) => handleOTPChange(index, e.target.value)}
                onKeyDown={(e) => handleKeyDown(index, e)}
                className="w-12 h-12 text-center text-lg font-semibold"
                disabled={isLoading}
              />
            ))}
          </div>
          
          {error && (
            <Alert variant="destructive">
              <AlertDescription>{error}</AlertDescription>
            </Alert>
          )}
          
          {session && (
            <div className="text-center text-sm text-muted-foreground">
              <p>
                Code expires in{' '}
                <CountdownTimer
                  expiresAt={session.expiresAt}
                  onExpire={() => {
                    setError('OTP expired. Please request a new one.');
                    WhatsAppAuth.clearSession();
                  }}
                />
              </p>
            </div>
          )}
          
          <div className="flex flex-col gap-2">
            <Button
              onClick={() => verifyOTP()}
              disabled={isLoading || otp.some(d => !d)}
              className="w-full"
            >
              {isLoading ? (
                <>
                  <Loader2 className="mr-2 h-4 w-4 animate-spin" />
                  Verifying...
                </>
              ) : (
                'Verify OTP'
              )}
            </Button>
            
            <Button
              variant="outline"
              onClick={sendOTP}
              disabled={isLoading}
              className="w-full"
            >
              Resend OTP
            </Button>
          </div>
        </div>
      </DialogContent>
    </Dialog>
  );
}
```

### Role-Based Access Control (RBAC)
```typescript
// lib/auth/rbac.ts
import { AdminType, Action, Permission } from '@/types/admin.types';

export class RBAC {
  private static instance: RBAC;
  private permissions: Map<string, Set<string>> = new Map();
  
  private constructor() {
    this.initializePermissions();
  }
  
  static getInstance(): RBAC {
    if (!RBAC.instance) {
      RBAC.instance = new RBAC();
    }
    return RBAC.instance;
  }
  
  private initializePermissions() {
    // Define default permissions for each admin type
    const defaultPermissions: Record<AdminType, Permission[]> = {
      [AdminType.SUPER_ADMIN]: [
        { resource: '*', actions: [Action.READ, Action.WRITE, Action.DELETE, Action.CONFIGURE] },
      ],
      [AdminType.PLATFORM_ADMIN]: [
        { resource: 'events', actions: [Action.READ, Action.WRITE, Action.DELETE] },
        { resource: 'users', actions: [Action.READ, Action.WRITE] },
        { resource: 'analytics', actions: [Action.READ] },
      ],
      [AdminType.REGIONAL_ADMIN]: [
        { resource: 'events', actions: [Action.READ, Action.WRITE], conditions: { region: 'assigned' } },
        { resource: 'users', actions: [Action.READ], conditions: { region: 'assigned' } },
      ],
      // ... other admin types
    };
    
    // Build permission map
    Object.entries(defaultPermissions).forEach(([adminType, permissions]) => {
      const key = this.getPermissionKey(adminType as AdminType);
      const permSet = new Set<string>();
      
      permissions.forEach(perm => {
        perm.actions.forEach(action => {
          permSet.add(`${perm.resource}:${action}`);
        });
      });
      
      this.permissions.set(key, permSet);
    });
  }
  
  hasPermission(
    adminType: AdminType,
    resource: string,
    action: Action,
    context?: Record<string, any>
  ): boolean {
    const key = this.getPermissionKey(adminType);
    const permissions = this.permissions.get(key);
    
    if (!permissions) return false;
    
    // Check wildcard permission
    if (permissions.has(`*:${action}`)) return true;
    
    // Check specific permission
    return permissions.has(`${resource}:${action}`);
  }
  
  private getPermissionKey(adminType: AdminType): string {
    return `admin:${adminType}`;
  }
}

// Permission hook
export function usePermissions() {
  const { user } = useAppStore();
  const rbac = RBAC.getInstance();
  
  const hasPermission = useCallback(
    (resource: string, action: Action, context?: Record<string, any>) => {
      if (!user || !user.adminType) return false;
      return rbac.hasPermission(user.adminType, resource, action, context);
    },
    [user, rbac]
  );
  
  const hasAnyPermission = useCallback(
    (permissions: Array<{ resource: string; action: Action }>) => {
      return permissions.some(({ resource, action }) => 
        hasPermission(resource, action)
      );
    },
    [hasPermission]
  );
  
  const hasAllPermissions = useCallback(
    (permissions: Array<{ resource: string; action: Action }>) => {
      return permissions.every(({ resource, action }) => 
        hasPermission(resource, action)
      );
    },
    [hasPermission]
  );
  
  return {
    hasPermission,
    hasAnyPermission,
    hasAllPermissions,
  };
}

// Protected route component
export function ProtectedRoute({ 
  children, 
  resource, 
  action,
  fallback = <Navigate to="/unauthorized" replace />
}: ProtectedRouteProps) {
  const { hasPermission } = usePermissions();
  
  if (!hasPermission(resource, action)) {
    return fallback;
  }
  
  return <>{children}</>;
}

// Permission guard component
export function PermissionGuard({ 
  children, 
  resource, 
  action,
  fallback = null
}: PermissionGuardProps) {
  const { hasPermission } = usePermissions();
  
  if (!hasPermission(resource, action)) {
    return <>{fallback}</>;
  }
  
  return <>{children}</>;
}
```

### Security Headers & CSP
```typescript
// middleware/security-headers.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function securityHeadersMiddleware(request: NextRequest) {
  const response = NextResponse.next();
  
  // Security headers
  response.headers.set('X-Frame-Options', 'DENY');
  response.headers.set('X-Content-Type-Options', 'nosniff');
  response.headers.set('X-XSS-Protection', '1; mode=block');
  response.headers.set('Referrer-Policy', 'strict-origin-when-cross-origin');
  response.headers.set('Permissions-Policy', 'camera=(), microphone=(), geolocation=()');
  
  // Content Security Policy
  const csp = [
    "default-src 'self'",
    "script-src 'self' 'unsafe-inline' 'unsafe-eval' https://cdn.jsdelivr.net https://cdnjs.cloudflare.com https://pagead2.googlesyndication.com",
    "style-src 'self' 'unsafe-inline' https://fonts.googleapis.com",
    "font-src 'self' https://fonts.gstatic.com",
    "img-src 'self' data: https: blob:",
    "connect-src 'self' https://api.eventfulhq.com wss://ws.eventfulhq.com https://pagead2.googlesyndication.com",
    "frame-src 'self' https://googleads.g.doubleclick.net",
    "object-src 'none'",
    "base-uri 'self'",
    "form-action 'self'",
    "frame-ancestors 'none'",
    "upgrade-insecure-requests",
  ].join('; ');
  
  response.headers.set('Content-Security-Policy', csp);
  
  // HSTS for production
  if (process.env.NODE_ENV === 'production') {
    response.headers.set(
      'Strict-Transport-Security',
      'max-age=63072000; includeSubDomains; preload'
    );
  }
  
  return response;
}
```

## Performance Optimization & SLA Compliance

### Next.js Performance Configuration
```typescript
// next.config.mjs
import { withSentryConfig } from '@sentry/nextjs';
import million from 'million/compiler';

/** @type {import('next').NextConfig} */
const nextConfig = {
  // React compiler for optimization
  experimental: {
    reactCompiler: true,
    optimizePackageImports: [
      '@shadcn/ui',
      'lucide-react',
      '@tabler/icons-react',
      'recharts',
      'd3',
    ],
    turbo: {
      resolveAlias: {
        '@': './src',
      },
    },
  },
  
  // Image optimization
  images: {
    domains: ['cdn.eventfulhq.com'],
    formats: ['image/avif', 'image/webp'],
    deviceSizes: [640, 750, 828, 1080, 1200, 1920, 2048, 3840],
    imageSizes: [16, 32, 48, 64, 96, 128, 256, 384],
  },
  
  // Bundle optimization
  webpack: (config, { isServer }) => {
    // Tree shaking
    config.optimization.usedExports = true;
    config.optimization.sideEffects = false;
    
    // Module federation for microfrontends
    if (!isServer) {
      config.optimization.splitChunks = {
        chunks: 'all',
        cacheGroups: {
          default: false,
          vendors: false,
          framework: {
            chunks: 'all',
            name: 'framework',
            test: /(?<!node_modules.*)[\\/]node_modules[\\/](react|react-dom|scheduler|prop-types|use-subscription)[\\/]/,
            priority: 40,
            enforce: true,
          },
          lib: {
            test(module) {
              return module.size() > 160000 &&
                /node_modules[/\\]/.test(module.identifier());
            },
            name(module) {
              const hash = crypto.createHash('sha1');
              hash.update(module.identifier());
              return hash.digest('hex').substring(0, 8);
            },
            priority: 30,
            minChunks: 1,
            reuseExistingChunk: true,
          },
          commons: {
            name: 'commons',
            minChunks: 2,
            priority: 20,
          },
          shared: {
            name(module, chunks) {
              return crypto
                .createHash('sha1')
                .update(chunks.reduce((acc, chunk) => acc + chunk.name, ''))
                .digest('hex') + (isServer ? '-server' : '');
            },
            priority: 10,
            minChunks: 2,
            reuseExistingChunk: true,
          },
        },
      };
    }
    
    return config;
  },
  
  // Output optimization
  output: 'standalone',
  compress: true,
  poweredByHeader: false,
  generateEtags: true,
  
  // Compiler optimizations
  compiler: {
    removeConsole: process.env.NODE_ENV === 'production',
    reactRemoveProperties: process.env.NODE_ENV === 'production',
  },
};

// Apply Million.js compiler for React optimization
const millionConfig = {
  auto: true,
  rsc: true,
};

// Sentry configuration
const sentryWebpackPluginOptions = {
  silent: true,
  widenClientFileUpload: true,
};

export default million.next(
  withSentryConfig(nextConfig, sentryWebpackPluginOptions),
  millionConfig
);
```

### Performance Monitoring
```typescript
// lib/performance/monitoring.ts
import { getCLS, getFID, getLCP, getFCP, getTTFB } from 'web-vitals';

export class PerformanceMonitor {
  private static instance: PerformanceMonitor;
  private metrics: Map<string, number> = new Map();
  
  private constructor() {
    this.initializeVitals();
    this.setupResourceObserver();
    this.setupLongTaskObserver();
  }
  
  static getInstance(): PerformanceMonitor {
    if (!PerformanceMonitor.instance) {
      PerformanceMonitor.instance = new PerformanceMonitor();
    }
    return PerformanceMonitor.instance;
  }
  
  private initializeVitals() {
    // Core Web Vitals
    getCLS(this.sendToAnalytics);
    getFID(this.sendToAnalytics);
    getLCP(this.sendToAnalytics);
    
    // Additional metrics
    getFCP(this.sendToAnalytics);
    getTTFB(this.sendToAnalytics);
  }
  
  private setupResourceObserver() {
    if ('PerformanceObserver' in window) {
      const observer = new PerformanceObserver((list) => {
        for (const entry of list.getEntries()) {
          if (entry.entryType === 'resource') {
            this.trackResourceTiming(entry as PerformanceResourceTiming);
          }
        }
      });
      
      observer.observe({ entryTypes: ['resource'] });
    }
  }
  
  private setupLongTaskObserver() {
    if ('PerformanceObserver' in window) {
      const observer = new PerformanceObserver((list) => {
        for (const entry of list.getEntries()) {
          console.warn('Long task detected:', {
            duration: entry.duration,
            startTime: entry.startTime,
          });
          
          // Report long tasks
          this.sendToAnalytics({
            name: 'long-task',
            value: entry.duration,
            id: generateId(),
            entries: [entry],
          });
        }
      });
      
      try {
        observer.observe({ entryTypes: ['longtask'] });
      } catch (e) {
        // Long task observer not supported
      }
    }
  }
  
  private trackResourceTiming(entry: PerformanceResourceTiming) {
    const duration = entry.responseEnd - entry.startTime;
    
    // Track slow resources
    if (duration > 1000) {
      console.warn('Slow resource:', {
        name: entry.name,
        duration,
        type: entry.initiatorType,
      });
    }
    
    // Categorize by type
    const category = this.getResourceCategory(entry);
    const currentTotal = this.metrics.get(category) || 0;
    this.metrics.set(category, currentTotal + duration);
  }
  
  private getResourceCategory(entry: PerformanceResourceTiming): string {
    if (entry.name.includes('/api/')) return 'api';
    if (entry.initiatorType === 'script') return 'script';
    if (entry.initiatorType === 'css') return 'css';
    if (entry.initiatorType === 'img') return 'image';
    return 'other';
  }
  
  private sendToAnalytics = (metric: any) => {
    // Send to analytics service
    if (window.gtag) {
      window.gtag('event', metric.name, {
        value: Math.round(metric.value),
        metric_id: metric.id,
        metric_value: metric.value,
        metric_delta: metric.delta,
      });
    }
    
    // Send to custom monitoring
    fetch('/api/monitoring/metrics', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        metric: metric.name,
        value: metric.value,
        timestamp: Date.now(),
        url: window.location.href,
        userAgent: navigator.userAgent,
      }),
    }).catch(() => {
      // Fail silently
    });
  };
  
  measureApiCall(endpoint: string, duration: number) {
    this.sendToAnalytics({
      name: 'api-call',
      value: duration,
      id: generateId(),
      entries: [],
      endpoint,
    });
  }
  
  getMetrics(): Record<string, number> {
    return Object.fromEntries(this.metrics);
  }
}

// Performance hook
export function usePerformance() {
  const monitor = PerformanceMonitor.getInstance();
  
  const measureApiCall = useCallback((endpoint: string, duration: number) => {
    monitor.measureApiCall(endpoint, duration);
  }, [monitor]);
  
  const getMetrics = useCallback(() => {
    return monitor.getMetrics();
  }, [monitor]);
  
  return {
    measureApiCall,
    getMetrics,
  };
}
```

### Image Optimization Component
```typescript
// components/shared/optimized-image.tsx
'use client';

import Image from 'next/image';
import { useState, useEffect } from 'react';
import { cn } from '@/lib/utils';

interface OptimizedImageProps {
  src: string;
  alt: string;
  width?: number;
  height?: number;
  priority?: boolean;
  className?: string;
  objectFit?: 'contain' | 'cover' | 'fill' | 'none' | 'scale-down';
  quality?: number;
  sizes?: string;
  onLoad?: () => void;
  placeholder?: 'blur' | 'empty';
  blurDataURL?: string;
}

export function OptimizedImage({
  src,
  alt,
  width,
  height,
  priority = false,
  className,
  objectFit = 'cover',
  quality = 75,
  sizes,
  onLoad,
  placeholder = 'blur',
  blurDataURL,
}: OptimizedImageProps) {
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState(false);
  
  // Generate blur data URL if not provided
  const shimmer = (w: number, h: number) => `
    <svg width="${w}" height="${h}" version="1.1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink">
      <defs>
        <linearGradient id="g">
          <stop stop-color="#333" offset="20%" />
          <stop stop-color="#222" offset="50%" />
          <stop stop-color="#333" offset="70%" />
        </linearGradient>
      </defs>
      <rect width="${w}" height="${h}" fill="#333" />
      <rect id="r" width="${w}" height="${h}" fill="url(#g)" />
      <animate xlink:href="#r" attributeName="x" from="-${w}" to="${w}" dur="1s" repeatCount="indefinite"  />
    </svg>`;
  
  const toBase64 = (str: string) =>
    typeof window === 'undefined'
      ? Buffer.from(str).toString('base64')
      : window.btoa(str);
  
  const defaultBlurDataURL = `data:image/svg+xml;base64,${toBase64(
    shimmer(width || 700, height || 475)
  )}`;
  
  // Cloudinary optimization
  const getOptimizedSrc = (src: string) => {
    if (src.includes('cloudinary')) {
      const parts = src.split('/upload/');
      if (parts.length === 2) {
        const transformations = [
          'f_auto', // Auto format
          'q_auto', // Auto quality
          'dpr_auto', // Auto DPR
          width ? `w_${width}` : '',
          height ? `h_${height}` : '',
        ].filter(Boolean).join(',');
        
        return `${parts[0]}/upload/${transformations}/${parts[1]}`;
      }
    }
    return src;
  };
  
  const optimizedSrc = getOptimizedSrc(src);
  
  if (error) {
    return (
      <div 
        className={cn(
          'bg-muted flex items-center justify-center',
          className
        )}
        style={{ width, height }}
      >
        <span className="text-muted-foreground text-sm">
          Failed to load image
        </span>
      </div>
    );
  }
  
  return (
    <div className={cn('relative overflow-hidden', className)}>
      <Image
        src={optimizedSrc}
        alt={alt}
        width={width}
        height={height}
        priority={priority}
        quality={quality}
        sizes={sizes || '(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw'}
        className={cn(
          'duration-700 ease-in-out',
          isLoading ? 'scale-110 blur-2xl grayscale' : 'scale-100 blur-0 grayscale-0',
          objectFit === 'cover' && 'object-cover',
          objectFit === 'contain' && 'object-contain',
          className
        )}
        onLoad={() => {
          setIsLoading(false);
          onLoad?.();
        }}
        onError={() => {
          setError(true);
          setIsLoading(false);
        }}
        placeholder={placeholder}
        blurDataURL={blurDataURL || defaultBlurDataURL}
      />
    </div>
  );
}

// Lazy load component
export function LazyImage({ threshold = 0.1, ...props }: OptimizedImageProps & { threshold?: number }) {
  const [isInView, setIsInView] = useState(false);
  const imgRef = useRef<HTMLDivElement>(null);
  
  useEffect(() => {
    const observer = new IntersectionObserver(
      (entries) => {
        entries.forEach((entry) => {
          if (entry.isIntersecting) {
            setIsInView(true);
            observer.disconnect();
          }
        });
      },
      { threshold }
    );
    
    if (imgRef.current) {
      observer.observe(imgRef.current);
    }
    
    return () => observer.disconnect();
  }, [threshold]);
  
  return (
    <div ref={imgRef}>
      {isInView ? (
        <OptimizedImage {...props} />
      ) : (
        <div 
          className={cn('bg-muted animate-pulse', props.className)}
          style={{ width: props.width, height: props.height }}
        />
      )}
    </div>
  );
}
```

### Code Splitting & Dynamic Imports
```typescript
// lib/performance/dynamic-imports.ts
import dynamic from 'next/dynamic';
import { ComponentType } from 'react';

// Dynamic import with loading state
export function createDynamicComponent<P = {}>(
  importFn: () => Promise<{ default: ComponentType<P> }>,
  options?: {
    loading?: ComponentType;
    ssr?: boolean;
  }
) {
  return dynamic(importFn, {
    loading: options?.loading || (() => <div>Loading...</div>),
    ssr: options?.ssr ?? true,
  });
}

// Pre-configured dynamic imports
export const DynamicEventCard = createDynamicComponent(
  () => import('@/components/features/events/event-card').then(mod => ({ default: mod.EventCard }))
);

export const DynamicSwaggerUI = createDynamicComponent(
  () => import('swagger-ui-react'),
  { ssr: false }
);

export const DynamicMonacoEditor = createDynamicComponent(
  () => import('@monaco-editor/react'),
  { ssr: false }
);

export const DynamicCharts = createDynamicComponent(
  () => import('@/components/features/analytics/charts')
);

// Route-based code splitting
export const DynamicAdminDashboard = createDynamicComponent(
  () => import('@/components/features/admin/super-admin-dashboard').then(mod => ({ default: mod.SuperAdminDashboard }))
);

export const DynamicDeveloperPortal = createDynamicComponent(
  () => import('@/components/features/developers/developer-dashboard').then(mod => ({ default: mod.DeveloperDashboard }))
);

// Lazy load heavy libraries
export const loadD3 = () => import('d3');
export const loadThree = () => import('three');
export const loadChartJS = () => import('chart.js');
```

## Testing Strategy

### Unit Testing Configuration
```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config';
import react from '@vitejs/plugin-react';
import path from 'path';

export default defineConfig({
  plugins: [react()],
  test: {
    environment: 'jsdom',
    globals: true,
    setupFiles: ['./tests/setup.ts'],
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html'],
      exclude: [
        'node_modules/',
        'tests/',
        '**/*.d.ts',
        '**/*.config.*',
        '**/*.stories.*',
        'src/services/generated/**',
      ],
      thresholds: {
        branches: 80,
        functions: 80,
        lines: 80,
        statements: 80,
      },
    },
  },
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
});

// tests/setup.ts
import '@testing-library/jest-dom';
import { cleanup } from '@testing-library/react';
import { afterEach, vi } from 'vitest';
import { server } from './mocks/server';

// Establish API mocking before all tests
beforeAll(() => server.listen({ onUnhandledRequest: 'error' }));

// Reset any request handlers that we may add during the tests,
// so they don't affect other tests
afterEach(() => {
  cleanup();
  server.resetHandlers();
});

// Clean up after the tests are finished
afterAll(() => server.close());

// Mock next/navigation
vi.mock('next/navigation', () => ({
  useRouter: () => ({
    push: vi.fn(),
    replace: vi.fn(),
    prefetch: vi.fn(),
    back: vi.fn(),
  }),
  usePathname: () => '/test-path',
  useSearchParams: () => new URLSearchParams(),
}));

// Mock next-intl
vi.mock('next-intl', () => ({
  useTranslations: () => (key: string) => key,
  useLocale: () => 'en',
}));
```

### Component Testing Examples
```typescript
// tests/components/event-card.test.tsx
import { describe, it, expect, vi } from 'vitest';
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { EventCard } from '@/components/features/events/event-card';
import { mockEvent } from '@/tests/mocks/event';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';

const queryClient = new QueryClient({
  defaultOptions: {
    queries: { retry: false },
    mutations: { retry: false },
  },
});

const renderWithProviders = (component: React.ReactElement) => {
  return render(
    <QueryClientProvider client={queryClient}>
      {component}
    </QueryClientProvider>
  );
};

describe('EventCard', () => {
  it('renders event information correctly', () => {
    renderWithProviders(<EventCard event={mockEvent} />);
    
    expect(screen.getByText(mockEvent.title)).toBeInTheDocument();
    expect(screen.getByText(mockEvent.venue.name)).toBeInTheDocument();
    expect(screen.getByText(`${mockEvent.price}`)).toBeInTheDocument();
  });
  
  it('shows sold out badge when event is sold out', () => {
    const soldOutEvent = { ...mockEvent, isSoldOut: true };
    renderWithProviders(<EventCard event={soldOutEvent} />);
    
    expect(screen.getByText('Sold Out')).toBeInTheDocument();
  });
  
  it('displays WhatsApp RSVP button when enabled', () => {
    const whatsappEvent = {
      ...mockEvent,
      whatsappRSVP: {
        enabled: true,
        phoneNumber: '+1234567890',
        message: 'RSVP for {event_name}',
      },
    };
    renderWithProviders(<EventCard event={whatsappEvent} />);
    
    expect(screen.getByText('RSVP via WhatsApp')).toBeInTheDocument();
  });
  
  it('calls onSelect when card is clicked', async () => {
    const onSelect = vi.fn();
    renderWithProviders(<EventCard event={mockEvent} onSelect={onSelect} />);
    
    const card = screen.getByRole('article');
    await userEvent.click(card);
    
    expect(onSelect).toHaveBeenCalledWith(mockEvent);
  });
  
  it('shares event when share button is clicked', async () => {
    const onShare = vi.fn();
    renderWithProviders(<EventCard event={mockEvent} onShare={onShare} />);
    
    const shareButton = screen.getByRole('button', { name: /share/i });
    await userEvent.click(shareButton);
    
    expect(onShare).toHaveBeenCalledWith(mockEvent);
  });
  
  it('displays loading skeleton when isLoading is true', () => {
    renderWithProviders(<EventCard event={mockEvent} isLoading />);
    
    expect(screen.getByTestId('event-card-skeleton')).toBeInTheDocument();
  });
});
```

### Integration Testing
```typescript
// tests/integration/event-creation.test.tsx
import { describe, it, expect, beforeEach } from 'vitest';
import { render, screen, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { CreateEventPage } from '@/app/[locale]/(dashboard)/events/create/page';
import { server } from '@/tests/mocks/server';
import { rest } from 'msw';

describe('Event Creation Flow', () => {
  beforeEach(() => {
    // Reset MSW handlers
    server.resetHandlers();
  });
  
  it('completes full event creation flow', async () => {
    const user = userEvent.setup();
    render(<CreateEventPage />);
    
    // Fill in event details
    await user.type(screen.getByLabelText('Event Title'), 'Test Music Festival');
    await user.type(screen.getByLabelText('Description'), 'An amazing outdoor music experience');
    
    // Select date
    await user.click(screen.getByLabelText('Start Date'));
    await user.click(screen.getByText('15')); // Select 15th of current month
    
    // Fill venue information
    await user.type(screen.getByLabelText('Venue Name'), 'Central Park');
    await user.type(screen.getByLabelText('Address'), '5th Avenue');
    await user.type(screen.getByLabelText('City'), 'New York');
    
    // Set capacity and price
    await user.clear(screen.getByLabelText('Capacity'));
    await user.type(screen.getByLabelText('Capacity'), '500');
    await user.clear(screen.getByLabelText('Price'));
    await user.type(screen.getByLabelText('Price'), '75');
    
    // Enable WhatsApp RSVP
    await user.click(screen.getByLabelText('WhatsApp RSVP'));
    await user.type(screen.getByLabelText('WhatsApp Number'), '+1234567890');
    
    // Mock successful API response
    server.use(
      rest.post('/api/events', (req, res, ctx) => {
        return res(
          ctx.status(201),
          ctx.json({
            id: '123',
            ...req.body,
          })
        );
      })
    );
    
    // Submit form
    await user.click(screen.getByRole('button', { name: 'Create Event' }));
    
    // Verify success
    await waitFor(() => {
      expect(screen.getByText('Event created successfully!')).toBeInTheDocument();
    });
  });
  
  it('shows validation errors for invalid input', async () => {
    const user = userEvent.setup();
    render(<CreateEventPage />);
    
    // Try to submit without filling required fields
    await user.click(screen.getByRole('button', { name: 'Create Event' }));
    
    // Check for validation errors
    await waitFor(() => {
      expect(screen.getByText('Event title is required')).toBeInTheDocument();
      expect(screen.getByText('Description is required')).toBeInTheDocument();
      expect(screen.getByText('Event date is required')).toBeInTheDocument();
    });
  });
});
```

### E2E Testing with Playwright
```typescript
// tests/e2e/event-flow.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Event Management Flow', () => {
  test.beforeEach(async ({ page }) => {
    // Login before each test
    await page.goto('/login');
    await page.fill('input[name="phoneNumber"]', '+1234567890');
    await page.click('button:has-text("Send OTP")');
    
    // Fill OTP
    for (let i = 0; i < 6; i++) {
      await page.fill(`input[name="otp-${i}"]`, '1');
    }
    
    await page.waitForURL('/dashboard');
  });
  
  test('creates and publishes an event', async ({ page }) => {
    // Navigate to create event
    await page.goto('/events/create');
    
    // Fill event form
    await page.fill('input[name="title"]', 'Playwright Test Event');
    await page.fill('textarea[name="description"]', 'Test event created by Playwright');
    
    // Select date
    await page.click('button[aria-label="Start Date"]');
    await page.click('button:has-text("15")');
    
    // Fill venue
    await page.fill('input[name="venue.name"]', 'Test Venue');
    await page.fill('input[name="venue.address"]', '123 Test Street');
    await page.fill('input[name="venue.city"]', 'Test City');
    
    // Submit
    await page.click('button:has-text("Create Event")');
    
    // Wait for redirect
    await page.waitForURL(/\/events\/[\w-]+$/);
    
    // Verify event created
    await expect(page.locator('h1')).toContainText('Playwright Test Event');
    
    // Publish event
    await page.click('button:has-text("Publish Event")');
    
    // Verify published
    await expect(page.locator('text=Published')).toBeVisible();
  });
  
  test('searches and filters events', async ({ page }) => {
    await page.goto('/events');
    
    // Search
    await page.fill('input[placeholder="Search events..."]', 'Music');
    await page.press('input[placeholder="Search events..."]', 'Enter');
    
    // Apply filters
    await page.click('button:has-text("Filters")');
    await page.selectOption('select[name="category"]', 'music');
    await page.fill('input[name="minPrice"]', '0');
    await page.fill('input[name="maxPrice"]', '100');
    await page.click('button:has-text("Apply Filters")');
    
    // Verify results
    await expect(page.locator('[data-testid="event-card"]')).toHaveCount(10);
  });
});
```

### Performance Testing
```typescript
// tests/performance/event-list.perf.ts
import { test, expect } from '@playwright/test';

test.describe('Performance Tests', () => {
  test('event list page loads within performance budget', async ({ page }) => {
    // Start measuring
    await page.goto('/events');
    
    // Measure Core Web Vitals
    const metrics = await page.evaluate(() => {
      return new Promise((resolve) => {
        let lcp, fid, cls;
        
        new PerformanceObserver((list) => {
          const entries = list.getEntries();
          entries.forEach((entry) => {
            if (entry.name === 'largest-contentful-paint') {
              lcp = entry.startTime;
            }
          });
        }).observe({ entryTypes: ['largest-contentful-paint'] });
        
        new PerformanceObserver((list) => {
          const entries = list.getEntries();
          entries.forEach((entry) => {
            if (!fid) {
              fid = entry.processingStart - entry.startTime;
            }
          });
        }).observe({ entryTypes: ['first-input'] });
        
        let clsValue = 0;
        new PerformanceObserver((list) => {
          for (const entry of list.getEntries()) {
            if (!entry.hadRecentInput) {
              clsValue += entry.value;
            }
          }
          cls = clsValue;
        }).observe({ entryTypes: ['layout-shift'] });
        
        // Wait for metrics
        setTimeout(() => {
          resolve({ lcp, fid, cls });
        }, 5000);
      });
    });
    
    // Assert performance budgets
    expect(metrics.lcp).toBeLessThan(2500); // LCP < 2.5s
    expect(metrics.fid || 0).toBeLessThan(100); // FID < 100ms
    expect(metrics.cls || 0).toBeLessThan(0.1); // CLS < 0.1
  });
  
  test('API response times meet SLA', async ({ page, request }) => {
    const endpoints = [
      '/api/events',
      '/api/users/me',
      '/api/analytics/dashboard',
    ];
    
    for (const endpoint of endpoints) {
      const start = Date.now();
      const response = await request.get(endpoint);
      const duration = Date.now() - start;
      
      expect(response.status()).toBe(200);
      expect(duration).toBeLessThan(500); // p95 < 500ms
    }
  });
});
```

## Vercel Deployment & Edge Functions

### Vercel Configuration
```json
// vercel.json
{
  "buildCommand": "npm run build",
  "outputDirectory": ".next",
  "framework": "nextjs",
  "regions": ["iad1", "sfo1", "sin1", "syd1", "fra1", "gru1"],
  "functions": {
    "app/api/bff/*": {
      "maxDuration": 30
    },
    "app/api/webhooks/*": {
      "maxDuration": 60
    },
    "app/[locale]/(landing)/[country]/[city]/page.tsx": {
      "runtime": "edge"
    },
    "app/robots.txt/route.ts": {
      "runtime": "edge"
    },
    "app/sitemap.xml/route.ts": {
      "runtime": "edge"
    }
  },
  "crons": [
    {
      "path": "/api/cron/sitemap-refresh",
      "schedule": "0 3 * * *"
    },
    {
      "path": "/api/cron/cache-warm",
      "schedule": "*/15 * * * *"
    }
  ],
  "env": {
    "NEXT_PUBLIC_API_URL": "@api-url",
    "NEXT_PUBLIC_WS_URL": "@ws-url",
    "NEXT_PUBLIC_ADSENSE_CLIENT_ID": "@adsense-client-id"
  },
  "headers": [
    {
      "source": "/(.*)",
      "headers": [
        {
          "key": "X-Frame-Options",
          "value": "DENY"
        },
        {
          "key": "X-Content-Type-Options",
          "value": "nosniff"
        },
        {
          "key": "Referrer-Policy",
          "value": "strict-origin-when-cross-origin"
        }
      ]
    },
    {
      "source": "/api/(.*)",
      "headers": [
        {
          "key": "Cache-Control",
          "value": "no-store, max-age=0"
        }
      ]
    }
  ],
  "rewrites": [
    {
      "source": "/api/v1/:path*",
      "destination": "https://api.eventfulhq.com/v1/:path*"
    }
  ],
  "redirects": [
    {
      "source": "/events",
      "destination": "/en/events",
      "permanent": false
    }
  ]
}
```

### Edge Function Examples
```typescript
// app/api/edge/geo-redirect/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { geolocation } from '@vercel/edge';

export const runtime = 'edge';
export const dynamic = 'force-dynamic';

export async function GET(request: NextRequest) {
  const { country, city, region } = geolocation(request);
  
  // Determine best regional domain
  const regionalDomain = getRegionalDomain(country);
  
  // Return redirect response
  return NextResponse.json({
    country,
    city,
    region,
    recommendedDomain: regionalDomain,
    currentDomain: request.headers.get('host'),
  });
}

function getRegionalDomain(country?: string): string {
  if (!country) return 'eventfulhq.com';
  
  const regionMap: Record<string, string> = {
    US: 'eventfulamerica.com',
    CA: 'eventfulamerica.com',
    MX: 'eventfulamerica.com',
    BR: 'eventfulamerica.com',
    AE: 'eventfularabia.com',
    SA: 'eventfularabia.com',
    KW: 'eventfularabia.com',
    IN: 'eventfulindia.com',
    GB: 'eventfuleurope.com',
    FR: 'eventfuleurope.com',
    DE: 'eventfuleurope.com',
    AU: 'eventfulaustralia.com',
    NZ: 'eventfulaustralia.com',
  };
  
  return regionMap[country] || 'eventfulhq.com';
}
```

### ISR & Cache Strategy
```typescript
// app/[locale]/(landing)/[country]/[city]/page.tsx
export const revalidate = 3600; // Revalidate every hour
export const dynamicParams = true; // Allow dynamic segments

// Generate static params for popular cities
export async function generateStaticParams() {
  const cities = await landingService.getTopCities(100);
  
  return cities.map((city) => ({
    country: city.countryCode.toLowerCase(),
    city: city.slug,
  }));
}

// Cache configuration for different page types
export const cacheConfig = {
  // Static pages
  landing: {
    revalidate: 86400, // 24 hours
    tags: ['landing'],
  },
  
  // Dynamic user pages
  userProfile: {
    revalidate: 300, // 5 minutes
    tags: (username: string) => ['user', `user:${username}`],
  },
  
  // Event pages
  event: {
    revalidate: 60, // 1 minute for real-time updates
    tags: (eventId: string) => ['event', `event:${eventId}`],
  },
};

// On-demand revalidation
export async function revalidateEvent(eventId: string) {
  try {
    await fetch(`${process.env.DEPLOYMENT_URL}/api/revalidate`, {
      method: 'POST',
      headers: {
        'x-revalidate-secret': process.env.REVALIDATE_SECRET!,
      },
      body: JSON.stringify({
        tags: [`event:${eventId}`],
      }),
    });
  } catch (error) {
    console.error('Failed to revalidate:', error);
  }
}
```

### Environment-Specific Configuration
```typescript
// lib/config/environment.ts
interface EnvironmentConfig {
  apiUrl: string;
  wsUrl: string;
  cdnUrl: string;
  analyticsId: string;
  sentryDsn: string;
  features: Record<string, boolean>;
}

const configs: Record<string, EnvironmentConfig> = {
  development: {
    apiUrl: 'http://localhost:3001',
    wsUrl: 'ws://localhost:3001',
    cdnUrl: 'http://localhost:3000',
    analyticsId: '',
    sentryDsn: '',
    features: {
      whatsappAuth: true,
      rfpMarketplace: true,
      aiRecommendations: true,
      developerPortal: true,
      superAdmin: true,
    },
  },
  
  staging: {
    apiUrl: 'https://api-staging.eventfulhq.com',
    wsUrl: 'wss://ws-staging.eventfulhq.com',
    cdnUrl: 'https://cdn-staging.eventfulhq.com',
    analyticsId: process.env.NEXT_PUBLIC_GA_STAGING!,
    sentryDsn: process.env.NEXT_PUBLIC_SENTRY_DSN_STAGING!,
    features: {
      whatsappAuth: true,
      rfpMarketplace: true,
      aiRecommendations: true,
      developerPortal: true,
      superAdmin: true,
    },
  },
  
  production: {
    apiUrl: 'https://api.eventfulhq.com',
    wsUrl: 'wss://ws.eventfulhq.com',
    cdnUrl: 'https://cdn.eventfulhq.com',
    analyticsId: process.env.NEXT_PUBLIC_GA_ID!,
    sentryDsn: process.env.NEXT_PUBLIC_SENTRY_DSN!,
    features: {
      whatsappAuth: true,
      rfpMarketplace: true,
      aiRecommendations: false, // Gradual rollout
      developerPortal: true,
      superAdmin: true,
    },
  },
};

export const config = configs[process.env.NEXT_PUBLIC_ENV || 'development'];

// Feature flag helper
export function isFeatureEnabled(feature: keyof typeof config.features): boolean {
  return config.features[feature] || false;
}
```

## Component Enhancement Patterns

### MatDash to ShadCN Migration Pattern
```typescript
// migration/button-migration.tsx
// BEFORE: MatDash Flowbite Button
import { Button as FlowbiteButton } from 'flowbite-react';

export function OldButton({ children, onClick, variant }) {
  return (
    <FlowbiteButton
      color={variant === 'danger' ? 'failure' : variant}
      onClick={onClick}
    >
      {children}
    </FlowbiteButton>
  );
}

// AFTER: ShadCN Button with enhanced features
import { Button } from '@/components/ui/button';
import { cn } from '@/lib/utils';
import { Loader2 } from 'lucide-react';

interface EnhancedButtonProps {
  children: React.ReactNode;
  onClick?: () => void;
  variant?: 'default' | 'destructive' | 'outline' | 'secondary' | 'ghost' | 'link';
  size?: 'default' | 'sm' | 'lg' | 'icon';
  loading?: boolean;
  disabled?: boolean;
  className?: string;
  asChild?: boolean;
}

export function EnhancedButton({
  children,
  onClick,
  variant = 'default',
  size = 'default',
  loading = false,
  disabled = false,
  className,
  asChild = false,
}: EnhancedButtonProps) {
  return (
    <Button
      variant={variant}
      size={size}
      onClick={onClick}
      disabled={disabled || loading}
      className={cn(className)}
      asChild={asChild}
    >
      {loading && <Loader2 className="mr-2 h-4 w-4 animate-spin" />}
      {children}
    </Button>
  );
}

// Migration utility
export function migrateButtonProps(oldProps: any): EnhancedButtonProps {
  const variantMap = {
    primary: 'default',
    danger: 'destructive',
    failure: 'destructive',
    success: 'secondary',
    warning: 'outline',
  };
  
  return {
    ...oldProps,
    variant: variantMap[oldProps.color] || 'default',
    loading: oldProps.isProcessing,
  };
}
```

### Form Enhancement Pattern
```typescript
// components/enhanced/form-field.tsx
import { forwardRef } from 'react';
import { useFormContext } from 'react-hook-form';
import { FormControl, FormDescription, FormField, FormItem, FormLabel, FormMessage } from '@/components/ui/form';
import { Input } from '@/components/ui/input';
import { Textarea } from '@/components/ui/textarea';
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from '@/components/ui/select';
import { cn } from '@/lib/utils';

interface EnhancedFormFieldProps {
  name: string;
  label?: string;
  description?: string;
  placeholder?: string;
  type?: 'text' | 'email' | 'password' | 'number' | 'tel' | 'url' | 'textarea' | 'select';
  options?: Array<{ value: string; label: string }>;
  disabled?: boolean;
  className?: string;
  required?: boolean;
  autoComplete?: string;
}

export const EnhancedFormField = forwardRef<
  HTMLInputElement | HTMLTextAreaElement | HTMLButtonElement,
  EnhancedFormFieldProps
>(({
  name,
  label,
  description,
  placeholder,
  type = 'text',
  options = [],
  disabled = false,
  className,
  required = false,
  autoComplete,
}, ref) => {
  const { control } = useFormContext();
  
  return (
    <FormField
      control={control}
      name={name}
      render={({ field }) => (
        <FormItem className={className}>
          {label && (
            <FormLabel>
              {label}
              {required && <span className="text-destructive ml-1">*</span>}
            </FormLabel>
          )}
          <FormControl>
            {type === 'textarea' ? (
              <Textarea
                {...field}
                ref={ref as any}
                placeholder={placeholder}
                disabled={disabled}
                className="min-h-[100px]"
              />
            ) : type === 'select' ? (
              <Select
                onValueChange={field.onChange}
                defaultValue={field.value}
                disabled={disabled}
              >
                <FormControl>
                  <SelectTrigger ref={ref as any}>
                    <SelectValue placeholder={placeholder} />
                  </SelectTrigger>
                </FormControl>
                <SelectContent>
                  {options.map((option) => (
                    <SelectItem key={option.value} value={option.value}>
                      {option.label}
                    </SelectItem>
                  ))}
                </SelectContent>
              </Select>
            ) : (
              <Input
                {...field}
                ref={ref as any}
                type={type}
                placeholder={placeholder}
                disabled={disabled}
                autoComplete={autoComplete}
              />
            )}
          </FormControl>
          {description && <FormDescription>{description}</FormDescription>}
          <FormMessage />
        </FormItem>
      )}
    />
  );
});

EnhancedFormField.displayName = 'EnhancedFormField';

// Usage example
export function EnhancedEventForm() {
  const form = useForm({
    resolver: yupResolver(eventSchema),
  });
  
  return (
    <Form {...form}>
      <form onSubmit={form.handleSubmit(onSubmit)} className="space-y-6">
        <EnhancedFormField
          name="title"
          label="Event Title"
          placeholder="Summer Music Festival"
          required
        />
        
        <EnhancedFormField
          name="description"
          label="Description"
          type="textarea"
          placeholder="Describe your event..."
          required
        />
        
        <EnhancedFormField
          name="category"
          label="Category"
          type="select"
          placeholder="Select a category"
          options={[
            { value: 'music', label: 'Music' },
            { value: 'sports', label: 'Sports' },
            { value: 'arts', label: 'Arts & Culture' },
          ]}
          required
        />
        
        <Button type="submit">Create Event</Button>
      </form>
    </Form>
  );
}
```

### Data Table Enhancement Pattern
```typescript
// components/enhanced/data-table.tsx
import {
  ColumnDef,
  ColumnFiltersState,
  SortingState,
  VisibilityState,
  flexRender,
  getCoreRowModel,
  getFilteredRowModel,
  getPaginationRowModel,
  getSortedRowModel,
  useReactTable,
} from '@tanstack/react-table';
import {
  Table,
  TableBody,
  TableCell,
  TableHead,
  TableHeader,
  TableRow,
} from '@/components/ui/table';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import {
  DropdownMenu,
  DropdownMenuCheckboxItem,
  DropdownMenuContent,
  DropdownMenuTrigger,
} from '@/components/ui/dropdown-menu';
import { ChevronDown } from 'lucide-react';

interface DataTableProps<TData, TValue> {
  columns: ColumnDef<TData, TValue>[];
  data: TData[];
  searchKey?: string;
  showColumnVisibility?: boolean;
  showPagination?: boolean;
  pageSize?: number;
}

export function EnhancedDataTable<TData, TValue>({
  columns,
  data,
  searchKey,
  showColumnVisibility = true,
  showPagination = true,
  pageSize = 10,
}: DataTableProps<TData, TValue>) {
  const [sorting, setSorting] = useState<SortingState>([]);
  const [columnFilters, setColumnFilters] = useState<ColumnFiltersState>([]);
  const [columnVisibility, setColumnVisibility] = useState<VisibilityState>({});
  const [rowSelection, setRowSelection] = useState({});
  
  const table = useReactTable({
    data,
    columns,
    onSortingChange: setSorting,
    onColumnFiltersChange: setColumnFilters,
    getCoreRowModel: getCoreRowModel(),
    getPaginationRowModel: getPaginationRowModel(),
    getSortedRowModel: getSortedRowModel(),
    getFilteredRowModel: getFilteredRowModel(),
    onColumnVisibilityChange: setColumnVisibility,
    onRowSelectionChange: setRowSelection,
    state: {
      sorting,
      columnFilters,
      columnVisibility,
      rowSelection,
    },
    initialState: {
      pagination: {
        pageSize,
      },
    },
  });
  
  return (
    <div className="w-full">
      <div className="flex items-center py-4 gap-2">
        {searchKey && (
          <Input
            placeholder={`Search by ${searchKey}...`}
            value={(table.getColumn(searchKey)?.getFilterValue() as string) ?? ''}
            onChange={(event) =>
              table.getColumn(searchKey)?.setFilterValue(event.target.value)
            }
            className="max-w-sm"
          />
        )}
        {showColumnVisibility && (
          <DropdownMenu>
            <DropdownMenuTrigger asChild>
              <Button variant="outline" className="ml-auto">
                Columns <ChevronDown className="ml-2 h-4 w-4" />
              </Button>
            </DropdownMenuTrigger>
            <DropdownMenuContent align="end">
              {table
                .getAllColumns()
                .filter((column) => column.getCanHide())
                .map((column) => {
                  return (
                    <DropdownMenuCheckboxItem
                      key={column.id}
                      className="capitalize"
                      checked={column.getIsVisible()}
                      onCheckedChange={(value) =>
                        column.toggleVisibility(!!value)
                      }
                    >
                      {column.id}
                    </DropdownMenuCheckboxItem>
                  );
                })}
            </DropdownMenuContent>
          </DropdownMenu>
        )}
      </div>
      <div className="rounded-md border">
        <Table>
          <TableHeader>
            {table.getHeaderGroups().map((headerGroup) => (
              <TableRow key={headerGroup.id}>
                {headerGroup.headers.map((header) => {
                  return (
                    <TableHead key={header.id}>
                      {header.isPlaceholder
                        ? null
                        : flexRender(
                            header.column.columnDef.header,
                            header.getContext()
                          )}
                    </TableHead>
                  );
                })}
              </TableRow>
            ))}
          </TableHeader>
          <TableBody>
            {table.getRowModel().rows?.length ? (
              table.getRowModel().rows.map((row) => (
                <TableRow
                  key={row.id}
                  data-state={row.getIsSelected() && 'selected'}
                >
                  {row.getVisibleCells().map((cell) => (
                    <TableCell key={cell.id}>
                      {flexRender(
                        cell.column.columnDef.cell,
                        cell.getContext()
                      )}
                    </TableCell>
                  ))}
                </TableRow>
              ))
            ) : (
              <TableRow>
                <TableCell
                  colSpan={columns.length}
                  className="h-24 text-center"
                >
                  No results.
                </TableCell>
              </TableRow>
            )}
          </TableBody>
        </Table>
      </div>
      {showPagination && (
        <div className="flex items-center justify-end space-x-2 py-4">
          <div className="flex-1 text-sm text-muted-foreground">
            {table.getFilteredSelectedRowModel().rows.length} of{' '}
            {table.getFilteredRowModel().rows.length} row(s) selected.
          </div>
          <div className="space-x-2">
            <Button
              variant="outline"
              size="sm"
              onClick={() => table.previousPage()}
              disabled={!table.getCanPreviousPage()}
            >
              Previous
            </Button>
            <Button
              variant="outline"
              size="sm"
              onClick={() => table.nextPage()}
              disabled={!table.getCanNextPage()}
            >
              Next
            </Button>
          </div>
        </div>
      )}
    </div>
  );
}
```

## Lambda Function Requirements

### Lambda Function Architecture Context
```typescript
// Lambda functions need to handle the following frontend requirements:

interface LambdaRequirements {
  // Performance Requirements
  performance: {
    coldStart: '<1s for critical paths',
    warmResponse: '<200ms p50, <500ms p95',
    timeout: '30s default, 60s for heavy operations',
    memory: '256MB default, up to 3GB for processing',
    concurrency: '1000 concurrent executions',
  };
  
  // Security Requirements
  security: {
    authentication: 'JWT validation required',
    authorization: 'RBAC with 11 admin types',
    encryption: 'All data encrypted in transit and at rest',
    cors: 'Configured for all EventfulHQ domains',
    rateLimit: 'By user, IP, and API key',
  };
  
  // Integration Requirements
  integration: {
    database: 'RDS Proxy for connection pooling',
    cache: 'ElastiCache Redis for session and data cache',
    queue: 'SQS for async processing',
    events: 'EventBridge for inter-service communication',
    storage: 'S3 for media and documents',
  };
  
  // Response Format
  responseFormat: {
    success: {
      status: 200,
      body: {
        data: 'T',
        meta: {
          requestId: 'string',
          timestamp: 'ISO 8601',
          version: 'string',
        },
      },
    },
    error: {
      status: 'number',
      body: {
        error: {
          code: 'string',
          message: 'string',
          details: 'object',
          requestId: 'string',
        },
      },
    },
    pagination: {
      data: 'T[]',
      meta: {
        total: 'number',
        page: 'number',
        limit: 'number',
        totalPages: 'number',
      },
    },
  };
  
  // Feature-Specific Requirements
  features: {
    whatsappAuth: {
      otpGeneration: '6-digit numeric',
      otpExpiry: '5 minutes',
      maxAttempts: 3,
      sessionManagement: 'Redis with TTL',
    },
    eventManagement: {
      imageProcessing: 'Resize, optimize, generate thumbnails',
      geoLocation: 'Store and query by coordinates',
      realtimeUpdates: 'WebSocket via API Gateway',
      bulkOperations: 'Batch processing up to 1000 items',
    },
    adminSystem: {
      auditLogging: 'All admin actions logged',
      permissionChecks: 'Granular resource-action validation',
      geographyFilters: 'Region/country/city level access',
      featureFlags: 'Dynamic microservice control',
    },
    developerPortal: {
      apiKeyGeneration: 'Secure random 32 chars',
      usageTracking: 'Per endpoint, per key metrics',
      rateLimiting: 'Configurable per API key',
      webhookDelivery: 'Retry with exponential backoff',
    },
  };
}

// Lambda function template
export const lambdaHandler = async (event: APIGatewayProxyEvent): Promise<APIGatewayProxyResult> => {
  // Request ID for tracing
  const requestId = event.requestContext.requestId;
  
  try {
    // 1. Parse and validate input
    const body = JSON.parse(event.body || '{}');
    const validated = await schema.validate(body);
    
    // 2. Authenticate
    const user = await authenticate(event.headers.authorization);
    
    // 3. Authorize
    await authorize(user, resource, action);
    
    // 4. Execute business logic
    const result = await businessLogic(validated, user);
    
    // 5. Format response
    return {
      statusCode: 200,
      headers: {
        'Content-Type': 'application/json',
        'X-Request-ID': requestId,
        'Cache-Control': 'no-cache',
      },
      body: JSON.stringify({
        data: result,
        meta: {
          requestId,
          timestamp: new Date().toISOString(),
          version: '1.0',
        },
      }),
    };
  } catch (error) {
    // Error handling
    return handleError(error, requestId);
  }
};
```

### Lambda Layer Requirements
```typescript
// Shared Lambda layers for common functionality

// Layer 1: Core utilities
export const coreLayer = {
  name: 'eventfulhq-core',
  description: 'Core utilities and helpers',
  compatibleRuntimes: ['nodejs18.x', 'nodejs20.x'],
  contents: [
    'auth/jwt-validator',
    'auth/rbac',
    'database/connection-pool',
    'cache/redis-client',
    'monitoring/metrics',
    'logging/structured-logger',
    'validation/schemas',
    'utils/date-time',
    'utils/crypto',
    'middleware/error-handler',
    'middleware/request-validator',
  ],
};

// Layer 2: AWS SDK clients
export const awsLayer = {
  name: 'eventfulhq-aws',
  description: 'Optimized AWS SDK clients',
  compatibleRuntimes: ['nodejs18.x', 'nodejs20.x'],
  contents: [
    '@aws-sdk/client-dynamodb',
    '@aws-sdk/client-s3',
    '@aws-sdk/client-sqs',
    '@aws-sdk/client-eventbridge',
    '@aws-sdk/client-ses',
    '@aws-sdk/client-secrets-manager',
    '@aws-sdk/rds-signer',
  ],
};

// Layer 3: Third-party integrations
export const integrationsLayer = {
  name: 'eventfulhq-integrations',
  description: 'Third-party service integrations',
  compatibleRuntimes: ['nodejs18.x', 'nodejs20.x'],
  contents: [
    'whatsapp-business-api',
    'stripe',
    'sendgrid/mail',
    'twilio',
    'openai',
    'google-maps-services-js',
    'libphonenumber-js',
  ],
};

## Multi-Domain Regional Routing

### Regional Domain Configuration
EventfulHQ supports multiple regional domains that automatically redirect users based on their geographic location while respecting user preferences.

#### Domain Mapping
- **Global**: eventfulhq.com (default)
- **Americas**: eventfulamerica.com (USA, Canada, Mexico, South America)
- **Arabia**: eventfularabia.com (GCC Countries: UAE, Saudi Arabia, Kuwait, Qatar, Bahrain, Oman)
- **India**: eventfulindia.com (India, Bangladesh, Sri Lanka, Nepal)
- **Europe**: eventfuleurope.com (EU Countries + UK)
- **Australia**: eventfulaustralia.com (Australia, New Zealand)

### Regional Routing Middleware
```typescript
// middleware-regional.ts
import { NextRequest, NextResponse } from 'next/server';

interface RegionalConfig {
  domain: string;
  countries: string[];
  defaultLocale: string;
  supportedLocales: string[];
}

const regionalConfigs: Record<string, RegionalConfig> = {
  america: {
    domain: 'eventfulamerica.com',
    countries: ['US', 'CA', 'MX', 'BR', 'AR', 'CL', 'CO', 'PE'],
    defaultLocale: 'en',
    supportedLocales: ['en', 'es', 'pt'],
  },
  arabia: {
    domain: 'eventfularabia.com',
    countries: ['AE', 'SA', 'KW', 'QA', 'BH', 'OM', 'EG', 'JO', 'LB'],
    defaultLocale: 'ar',
    supportedLocales: ['ar', 'en'],
  },
  india: {
    domain: 'eventfulindia.com',
    countries: ['IN', 'BD', 'LK', 'NP', 'PK'],
    defaultLocale: 'hi',
    supportedLocales: ['hi', 'en'],
  },
  europe: {
    domain: 'eventfuleurope.com',
    countries: ['GB', 'FR', 'DE', 'IT', 'ES', 'NL', 'BE', 'PL', 'PT', 'SE', 'DK', 'FI', 'NO', 'IE', 'AT', 'CH'],
    defaultLocale: 'en',
    supportedLocales: ['en', 'fr', 'es', 'pt', 'ru'],
  },
  australia: {
    domain: 'eventfulaustralia.com',
    countries: ['AU', 'NZ'],
    defaultLocale: 'en',
    supportedLocales: ['en'],
  },
};

// Reverse mapping for quick lookup
const countryToRegion: Record<string, string> = {};
Object.entries(regionalConfigs).forEach(([region, config]) => {
  config.countries.forEach(country => {
    countryToRegion[country] = region;
  });
});

export async function regionalMiddleware(request: NextRequest) {
  const hostname = request.headers.get('host') || '';
  const pathname = request.nextUrl.pathname;
  
  // Get user's country from geo data
  const userCountry = request.geo?.country || 
                     request.headers.get('x-vercel-ip-country') || 
                     '';
  
  // Check if user has a saved regional preference
  const savedRegion = request.cookies.get('preferred-region')?.value;
  const savedDomain = request.cookies.get('preferred-domain')?.value;
  
  // If user has saved preferences, respect them
  if (savedDomain && hostname !== savedDomain) {
    const url = new URL(request.url);
    url.hostname = savedDomain;
    return NextResponse.redirect(url);
  }
  
  // Determine the appropriate region for the user
  const userRegion = countryToRegion[userCountry] || 'global';
  const targetConfig = regionalConfigs[userRegion];
  
  // If current domain doesn't match user's region and no saved preference
  if (!savedRegion && targetConfig && hostname !== targetConfig.domain && hostname === 'eventfulhq.com') {
    const url = new URL(request.url);
    url.hostname = targetConfig.domain;
    
    const response = NextResponse.redirect(url);
    
    // Save the auto-detected region (but as a soft preference)
    response.cookies.set('detected-region', userRegion, {
      maxAge: 60 * 60 * 24 * 30, // 30 days
      sameSite: 'lax',
      secure: process.env.NODE_ENV === 'production',
    });
    
    return response;
  }
  
  return NextResponse.next();
}
```

### Region Selector Component
```typescript
// components/features/regional/region-selector.tsx
'use client';

import { useState, useEffect } from 'react';
import { useRouter } from 'next/navigation';
import {
  Dialog,
  DialogContent,
  DialogDescription,
  DialogHeader,
  DialogTitle,
} from '@/components/ui/dialog';
import {
  Select,
  SelectContent,
  SelectItem,
  SelectTrigger,
  SelectValue,
} from '@/components/ui/select';
import { Button } from '@/components/ui/button';
import { GlobeIcon } from 'lucide-react';
import { toast } from 'sonner';

interface Region {
  id: string;
  name: string;
  domain: string;
  flag: string;
  description: string;
}

const regions: Region[] = [
  {
    id: 'global',
    name: 'Global',
    domain: 'eventfulhq.com',
    flag: '🌍',
    description: 'Access events worldwide',
  },
  {
    id: 'america',
    name: 'Americas',
    domain: 'eventfulamerica.com',
    flag: '🇺🇸',
    description: 'Events in North & South America',
  },
  {
    id: 'arabia',
    name: 'Arabia',
    domain: 'eventfularabia.com',
    flag: '🇸🇦',
    description: 'Events in GCC & Middle East',
  },
  {
    id: 'india',
    name: 'India',
    domain: 'eventfulindia.com',
    flag: '🇮🇳',
    description: 'Events in India & South Asia',
  },
  {
    id: 'europe',
    name: 'Europe',
    domain: 'eventfuleurope.com',
    flag: '🇪🇺',
    description: 'Events across Europe',
  },
  {
    id: 'australia',
    name: 'Australia',
    domain: 'eventfulaustralia.com',
    flag: '🇦🇺',
    description: 'Events in Australia & Oceania',
  },
];

export function RegionSelector() {
  const router = useRouter();
  const [showDialog, setShowDialog] = useState(false);
  const [selectedRegion, setSelectedRegion] = useState<string>('global');
  const [currentDomain, setCurrentDomain] = useState<string>('');

  useEffect(() => {
    // Get current domain
    const domain = window.location.hostname;
    setCurrentDomain(domain);
    
    // Find current region
    const currentRegion = regions.find(r => r.domain === domain);
    if (currentRegion) {
      setSelectedRegion(currentRegion.id);
    }
    
    // Check if this is first visit without preference
    const hasPreference = document.cookie.includes('preferred-region');
    const detectedRegion = document.cookie.includes('detected-region');
    
    if (!hasPreference && detectedRegion && domain === 'eventfulhq.com') {
      setShowDialog(true);
    }
  }, []);

  const handleRegionChange = (regionId: string) => {
    const region = regions.find(r => r.id === regionId);
    if (!region) return;
    
    // Save preference
    document.cookie = `preferred-region=${regionId};max-age=31536000;path=/;domain=.eventfulhq.com;samesite=lax${
      process.env.NODE_ENV === 'production' ? ';secure' : ''
    }`;
    document.cookie = `preferred-domain=${region.domain};max-age=31536000;path=/;domain=.eventfulhq.com;samesite=lax${
      process.env.NODE_ENV === 'production' ? ';secure' : ''
    }`;
    
    // Redirect to new domain
    if (region.domain !== currentDomain) {
      window.location.href = `https://${region.domain}${window.location.pathname}`;
    }
    
    toast.success(`Switched to ${region.name} region`);
  };

  const currentRegion = regions.find(r => r.id === selectedRegion);

  return (
    <>
      <Button
        variant="ghost"
        size="sm"
        onClick={() => setShowDialog(true)}
        className="flex items-center gap-2"
      >
        <GlobeIcon className="h-4 w-4" />
        <span className="hidden sm:inline">
          {currentRegion?.flag} {currentRegion?.name}
        </span>
        <span className="sm:hidden">{currentRegion?.flag}</span>
      </Button>

      <Dialog open={showDialog} onOpenChange={setShowDialog}>
        <DialogContent className="sm:max-w-md">
          <DialogHeader>
            <DialogTitle>Select Your Region</DialogTitle>
            <DialogDescription>
              Choose your preferred region to see relevant events and content
            </DialogDescription>
          </DialogHeader>
          
          <div className="space-y-4 py-4">
            <Select value={selectedRegion} onValueChange={setSelectedRegion}>
              <SelectTrigger>
                <SelectValue />
              </SelectTrigger>
              <SelectContent>
                {regions.map((region) => (
                  <SelectItem key={region.id} value={region.id}>
                    <div className="flex items-center gap-3">
                      <span className="text-2xl">{region.flag}</span>
                      <div>
                        <div className="font-medium">{region.name}</div>
                        <div className="text-xs text-muted-foreground">
                          {region.description}
                        </div>
                      </div>
                    </div>
                  </SelectItem>
                ))}
              </SelectContent>
            </Select>
            
            <div className="text-sm text-muted-foreground">
              Your selection will be saved and you'll be redirected to {regions.find(r => r.id === selectedRegion)?.domain}
            </div>
            
            <div className="flex justify-end gap-2">
              <Button
                variant="outline"
                onClick={() => setShowDialog(false)}
              >
                Cancel
              </Button>
              <Button
                onClick={() => {
                  handleRegionChange(selectedRegion);
                  setShowDialog(false);
                }}
              >
                Save & Continue
              </Button>
            </div>
          </div>
        </DialogContent>
      </Dialog>
    </>
  );
}
```

### Next.js Configuration for Multi-Domain
```javascript
// next.config.mjs
const domains = [
  'eventfulhq.com',
  'eventfulamerica.com',
  'eventfularabia.com',
  'eventfulindia.com',
  'eventfuleurope.com',
  'eventfulaustralia.com',
];

/** @type {import('next').NextConfig} */
const nextConfig = {
  experimental: {
    appDir: true,
  },
  images: {
    domains: [...domains, 'cdn.eventfulhq.com'],
    remotePatterns: [
      {
        protocol: 'https',
        hostname: '**.eventfulhq.com',
      },
    ],
  },
  async headers() {
    return [
      {
        source: '/:path*',
        headers: [
          {
            key: 'X-Frame-Options',
            value: 'DENY',
          },
          {
            key: 'Content-Security-Policy',
            value: `frame-ancestors 'none'; default-src 'self' ${domains.join(' ')};`,
          },
        ],
      },
    ];
  },
  async rewrites() {
    return {
      beforeFiles: [
        // API rewrites for regional domains
        ...domains.map((domain) => ({
          source: '/api/:path*',
          destination: `https://api.${domain}/api/:path*`,
          has: [
            {
              type: 'host',
              value: domain,
            },
          ],
        })),
      ],
    };
  },
  async redirects() {
    return [
      {
        source: '/events',
        destination: '/en/events',
        permanent: false,
      },
    ];
  },
};

export default nextConfig;
```

## SuperAdmin Access System

### Admin User Types (11 Types)
```typescript
// types/admin.types.ts
export enum AdminType {
  SUPER_ADMIN = 'SUPER_ADMIN',                 // Full system access
  PLATFORM_ADMIN = 'PLATFORM_ADMIN',           // Platform-wide management
  REGIONAL_ADMIN = 'REGIONAL_ADMIN',           // Regional management
  COUNTRY_ADMIN = 'COUNTRY_ADMIN',             // Country-level management
  CITY_ADMIN = 'CITY_ADMIN',                   // City-level management
  CATEGORY_ADMIN = 'CATEGORY_ADMIN',           // Event category management
  CONTENT_ADMIN = 'CONTENT_ADMIN',             // Content moderation
  SUPPORT_ADMIN = 'SUPPORT_ADMIN',             // Customer support
  FINANCE_ADMIN = 'FINANCE_ADMIN',             // Financial operations
  MARKETING_ADMIN = 'MARKETING_ADMIN',         // Marketing campaigns
  DEVELOPER_ADMIN = 'DEVELOPER_ADMIN',         // API and developer relations
}

export interface AdminUser {
  id: string;
  email: string;
  phoneNumber: string;
  name: string;
  type: AdminType;
  permissions: Permission[];
  geographyAccess: GeographyAccess[];
  featureAccess: FeatureAccess[];
  isActive: boolean;
  isTwoFactorEnabled: boolean;
  lastLogin: Date;
  createdAt: Date;
  createdBy: string;
}

export interface Permission {
  resource: string;      // Microservice name as AppFeature
  actions: Action[];     // Read, Write, Delete, etc.
  conditions?: Record<string, any>;
}

export interface GeographyAccess {
  type: 'GLOBAL' | 'REGION' | 'COUNTRY' | 'CITY';
  identifier: string;    // e.g., 'US', 'New York', 'ASIA'
  permissions: Action[];
}

export interface FeatureAccess {
  microservice: string;  // All 30+ microservices
  permissions: Action[];
  limits?: Record<string, number>;
}

export enum Action {
  READ = 'READ',
  WRITE = 'WRITE',
  UPDATE = 'UPDATE',
  DELETE = 'DELETE',
  PUBLISH = 'PUBLISH',
  EXPORT = 'EXPORT',
  CONFIGURE = 'CONFIGURE',
}
```

### SuperAdmin Dashboard Components
```typescript
// components/features/admin/super-admin-dashboard.tsx
'use client';

import { useState } from 'react';
import { useTranslations } from 'next-intl';
import { Tabs, TabsContent, TabsList, TabsTrigger } from '@/components/ui/tabs';
import { Card, CardContent, CardDescription, CardHeader, CardTitle } from '@/components/ui/card';
import { AdminManagement } from './admin-management';
import { FeatureControl } from './feature-control';
import { GeographyControl } from './geography-control';
import { PermissionMatrix } from './permission-matrix';
import { SystemConfiguration } from './system-configuration';
import { AuditLogs } from './audit-logs';
import { SystemMonitoring } from './system-monitoring';
import { Shield, Users, Globe, Settings, Activity, FileText, Map } from 'lucide-react';

export function SuperAdminDashboard() {
  const t = useTranslations('admin');
  const [activeTab, setActiveTab] = useState('admins');

  return (
    <div className="space-y-6">
      <div>
        <h1 className="text-3xl font-bold">{t('superAdmin.title')}</h1>
        <p className="text-muted-foreground">{t('superAdmin.description')}</p>
      </div>

      <Tabs value={activeTab} onValueChange={setActiveTab} className="space-y-4">
        <TabsList className="grid w-full grid-cols-7">
          <TabsTrigger value="admins" className="flex items-center gap-2">
            <Users className="h-4 w-4" />
            <span className="hidden sm:inline">{t('tabs.admins')}</span>
          </TabsTrigger>
          <TabsTrigger value="features" className="flex items-center gap-2">
            <Shield className="h-4 w-4" />
            <span className="hidden sm:inline">{t('tabs.features')}</span>
          </TabsTrigger>
          <TabsTrigger value="geography" className="flex items-center gap-2">
            <Globe className="h-4 w-4" />
            <span className="hidden sm:inline">{t('tabs.geography')}</span>
          </TabsTrigger>
          <TabsTrigger value="permissions" className="flex items-center gap-2">
            <Map className="h-4 w-4" />
            <span className="hidden sm:inline">{t('tabs.permissions')}</span>
          </TabsTrigger>
          <TabsTrigger value="config" className="flex items-center gap-2">
            <Settings className="h-4 w-4" />
            <span className="hidden sm:inline">{t('tabs.config')}</span>
          </TabsTrigger>
          <TabsTrigger value="monitoring" className="flex items-center gap-2">
            <Activity className="h-4 w-4" />
            <span className="hidden sm:inline">{t('tabs.monitoring')}</span>
          </TabsTrigger>
          <TabsTrigger value="audit" className="flex items-center gap-2">
            <FileText className="h-4 w-4" />
            <span className="hidden sm:inline">{t('tabs.audit')}</span>
          </TabsTrigger>
        </TabsList>

        <TabsContent value="admins" className="space-y-4">
          <AdminManagement />
        </TabsContent>

        <TabsContent value="features" className="space-y-4">
          <FeatureControl />
        </TabsContent>

        <TabsContent value="geography" className="space-y-4">
          <GeographyControl />
        </TabsContent>

        <TabsContent value="permissions" className="space-y-4">
          <PermissionMatrix />
        </TabsContent>

        <TabsContent value="config" className="space-y-4">
          <SystemConfiguration />
        </TabsContent>

        <TabsContent value="monitoring" className="space-y-4">
          <SystemMonitoring />
        </TabsContent>

        <TabsContent value="audit" className="space-y-4">
          <AuditLogs />
        </TabsContent>
      </Tabs>
    </div>
  );
}
```

### Admin Management Interface
```typescript
// components/features/admin/admin-management.tsx
'use client';

import { useState } from 'react';
import { useQuery, useMutation } from '@tanstack/react-query';
import { DataTable } from '@/components/ui/data-table';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import {
  Select,
  SelectContent,
  SelectItem,
  SelectTrigger,
  SelectValue,
} from '@/components/ui/select';
import { Badge } from '@/components/ui/badge';
import {
  Dialog,
  DialogContent,
  DialogDescription,
  DialogHeader,
  DialogTitle,
  DialogTrigger,
} from '@/components/ui/dialog';
import { AdminType } from '@/types/admin.types';
import { adminService } from '@/services/admin-service';
import { CreateAdminForm } from './create-admin-form';
import { PermissionEditor } from './permission-editor';
import { Plus, Edit, Trash, Shield, Key } from 'lucide-react';
import { toast } from 'sonner';

export function AdminManagement() {
  const [search, setSearch] = useState('');
  const [typeFilter, setTypeFilter] = useState<AdminType | 'ALL'>('ALL');
  const [showCreateDialog, setShowCreateDialog] = useState(false);
  const [selectedAdmin, setSelectedAdmin] = useState(null);

  const { data: admins, isLoading } = useQuery({
    queryKey: ['admins', search, typeFilter],
    queryFn: () => adminService.getAdmins({ search, type: typeFilter }),
  });

  const { mutate: deleteAdmin } = useMutation({
    mutationFn: adminService.deleteAdmin,
    onSuccess: () => {
      toast.success('Admin deleted successfully');
      queryClient.invalidateQueries({ queryKey: ['admins'] });
    },
  });

  const { mutate: toggleTwoFactor } = useMutation({
    mutationFn: adminService.toggleTwoFactor,
    onSuccess: () => {
      toast.success('Two-factor authentication updated');
      queryClient.invalidateQueries({ queryKey: ['admins'] });
    },
  });

  const columns = [
    {
      accessorKey: 'name',
      header: 'Name',
      cell: ({ row }) => (
        <div>
          <p className="font-medium">{row.original.name}</p>
          <p className="text-sm text-muted-foreground">{row.original.email}</p>
        </div>
      ),
    },
    {
      accessorKey: 'type',
      header: 'Admin Type',
      cell: ({ row }) => (
        <Badge variant={row.original.type === 'SUPER_ADMIN' ? 'destructive' : 'default'}>
          {row.original.type.replace('_', ' ')}
        </Badge>
      ),
    },
    {
      accessorKey: 'geographyAccess',
      header: 'Geography Access',
      cell: ({ row }) => {
        const access = row.original.geographyAccess;
        if (access.some(a => a.type === 'GLOBAL')) {
          return <Badge variant="outline">Global</Badge>;
        }
        return (
          <div className="flex flex-wrap gap-1">
            {access.slice(0, 3).map((geo, idx) => (
              <Badge key={idx} variant="secondary" className="text-xs">
                {geo.identifier}
              </Badge>
            ))}
            {access.length > 3 && (
              <Badge variant="secondary" className="text-xs">
                +{access.length - 3}
              </Badge>
            )}
          </div>
        );
      },
    },
    {
      accessorKey: 'featureAccess',
      header: 'Features',
      cell: ({ row }) => {
        const features = row.original.featureAccess;
        return (
          <div className="flex flex-wrap gap-1">
            {features.slice(0, 3).map((feature, idx) => (
              <Badge key={idx} variant="outline" className="text-xs">
                {feature.microservice}
              </Badge>
            ))}
            {features.length > 3 && (
              <Badge variant="outline" className="text-xs">
                +{features.length - 3}
              </Badge>
            )}
          </div>
        );
      },
    },
    {
      accessorKey: 'status',
      header: 'Status',
      cell: ({ row }) => (
        <div className="flex items-center gap-2">
          <Badge variant={row.original.isActive ? 'success' : 'destructive'}>
            {row.original.isActive ? 'Active' : 'Inactive'}
          </Badge>
          {row.original.isTwoFactorEnabled && (
            <Shield className="h-4 w-4 text-green-500" />
          )}
        </div>
      ),
    },
    {
      accessorKey: 'lastLogin',
      header: 'Last Login',
      cell: ({ row }) => (
        <span className="text-sm text-muted-foreground">
          {new Date(row.original.lastLogin).toLocaleDateString()}
        </span>
      ),
    },
    {
      id: 'actions',
      cell: ({ row }) => (
        <div className="flex items-center gap-2">
          <Button
            variant="ghost"
            size="icon"
            onClick={() => setSelectedAdmin(row.original)}
          >
            <Edit className="h-4 w-4" />
          </Button>
          <Button
            variant="ghost"
            size="icon"
            onClick={() => toggleTwoFactor(row.original.id)}
          >
            <Key className="h-4 w-4" />
          </Button>
          <Button
            variant="ghost"
            size="icon"
            onClick={() => deleteAdmin(row.original.id)}
            disabled={row.original.type === 'SUPER_ADMIN'}
          >
            <Trash className="h-4 w-4" />
          </Button>
        </div>
      ),
    },
  ];

  return (
    <div className="space-y-4">
      <div className="flex items-center justify-between">
        <div className="flex items-center gap-4">
          <Input
            placeholder="Search admins..."
            value={search}
            onChange={(e) => setSearch(e.target.value)}
            className="w-64"
          />
          <Select value={typeFilter} onValueChange={setTypeFilter}>
            <SelectTrigger className="w-48">
              <SelectValue />
            </SelectTrigger>
            <SelectContent>
              <SelectItem value="ALL">All Types</SelectItem>
              {Object.values(AdminType).map((type) => (
                <SelectItem key={type} value={type}>
                  {type.replace(/_/g, ' ')}
                </SelectItem>
              ))}
            </SelectContent>
          </Select>
        </div>
        <Dialog open={showCreateDialog} onOpenChange={setShowCreateDialog}>
          <DialogTrigger asChild>
            <Button>
              <Plus className="h-4 w-4 mr-2" />
              Add Admin
            </Button>
          </DialogTrigger>
          <DialogContent className="max-w-2xl">
            <DialogHeader>
              <DialogTitle>Create New Admin</DialogTitle>
              <DialogDescription>
                Add a new admin user with specific permissions and access controls
              </DialogDescription>
            </DialogHeader>
            <CreateAdminForm onSuccess={() => setShowCreateDialog(false)} />
          </DialogContent>
        </Dialog>
      </div>

      <DataTable
        columns={columns}
        data={admins?.data || []}
        loading={isLoading}
      />

      {selectedAdmin && (
        <PermissionEditor
          admin={selectedAdmin}
          open={!!selectedAdmin}
          onOpenChange={() => setSelectedAdmin(null)}
        />
      )}
    </div>
  );
}
```

### Feature Control (Microservices as AppFeatures)
```typescript
// components/features/admin/feature-control.tsx
'use client';

import { useState } from 'react';
import { useQuery, useMutation } from '@tanstack/react-query';
import { Card, CardContent, CardDescription, CardHeader, CardTitle } from '@/components/ui/card';
import { Switch } from '@/components/ui/switch';
import { Button } from '@/components/ui/button';
import { Badge } from '@/components/ui/badge';
import { Slider } from '@/components/ui/slider';
import { Input } from '@/components/ui/input';
import { Label } from '@/components/ui/label';
import {
  Accordion,
  AccordionContent,
  AccordionItem,
  AccordionTrigger,
} from '@/components/ui/accordion';
import { adminService } from '@/services/admin-service';
import { Microservice } from '@/types/admin.types';
import { Settings, AlertCircle, Activity } from 'lucide-react';
import { toast } from 'sonner';

// All 30+ microservices
const MICROSERVICES = [
  { id: 'event-service', name: 'Event Service', category: 'Core' },
  { id: 'user-service', name: 'User Service', category: 'Core' },
  { id: 'whatsapp-service', name: 'WhatsApp Service', category: 'Communication' },
  { id: 'rfp-service', name: 'RFP Service', category: 'Marketplace' },
  { id: 'analytics-service', name: 'Analytics Service', category: 'Analytics' },
  { id: 'payment-service', name: 'Payment Service', category: 'Financial' },
  { id: 'notification-service', name: 'Notification Service', category: 'Communication' },
  { id: 'search-service', name: 'Search Service', category: 'Core' },
  { id: 'recommendation-service', name: 'Recommendation Service', category: 'AI' },
  { id: 'media-service', name: 'Media Service', category: 'Content' },
  { id: 'chat-service', name: 'Chat Service', category: 'Communication' },
  { id: 'booking-service', name: 'Booking Service', category: 'Core' },
  { id: 'review-service', name: 'Review Service', category: 'Social' },
  { id: 'social-service', name: 'Social Service', category: 'Social' },
  { id: 'email-service', name: 'Email Service', category: 'Communication' },
  { id: 'sms-service', name: 'SMS Service', category: 'Communication' },
  { id: 'calendar-service', name: 'Calendar Service', category: 'Core' },
  { id: 'ticketing-service', name: 'Ticketing Service', category: 'Core' },
  { id: 'venue-service', name: 'Venue Service', category: 'Marketplace' },
  { id: 'artist-service', name: 'Artist Service', category: 'Marketplace' },
  { id: 'sponsor-service', name: 'Sponsor Service', category: 'Marketplace' },
  { id: 'streaming-service', name: 'Streaming Service', category: 'Content' },
  { id: 'translation-service', name: 'Translation Service', category: 'AI' },
  { id: 'moderation-service', name: 'Moderation Service', category: 'AI' },
  { id: 'fraud-service', name: 'Fraud Detection', category: 'Security' },
  { id: 'audit-service', name: 'Audit Service', category: 'Security' },
  { id: 'backup-service', name: 'Backup Service', category: 'Infrastructure' },
  { id: 'monitoring-service', name: 'Monitoring Service', category: 'Infrastructure' },
  { id: 'queue-service', name: 'Queue Service', category: 'Infrastructure' },
  { id: 'cache-service', name: 'Cache Service', category: 'Infrastructure' },
];

export function FeatureControl() {
  const [selectedCategory, setSelectedCategory] = useState<string>('all');
  
  const { data: features, isLoading } = useQuery({
    queryKey: ['features'],
    queryFn: adminService.getFeatures,
  });

  const { mutate: updateFeature } = useMutation({
    mutationFn: adminService.updateFeature,
    onSuccess: () => {
      toast.success('Feature updated successfully');
      queryClient.invalidateQueries({ queryKey: ['features'] });
    },
  });

  const categories = ['all', ...new Set(MICROSERVICES.map(m => m.category))];
  const filteredServices = selectedCategory === 'all' 
    ? MICROSERVICES 
    : MICROSERVICES.filter(m => m.category === selectedCategory);

  return (
    <div className="space-y-6">
      <div className="flex items-center gap-4">
        {categories.map((category) => (
          <Button
            key={category}
            variant={selectedCategory === category ? 'default' : 'outline'}
            size="sm"
            onClick={() => setSelectedCategory(category)}
          >
            {category.charAt(0).toUpperCase() + category.slice(1)}
          </Button>
        ))}
      </div>

      <div className="grid gap-4 md:grid-cols-2 lg:grid-cols-3">
        {filteredServices.map((service) => {
          const feature = features?.find(f => f.id === service.id);
          
          return (
            <Card key={service.id}>
              <CardHeader className="pb-3">
                <div className="flex items-center justify-between">
                  <CardTitle className="text-base">{service.name}</CardTitle>
                  <Switch
                    checked={feature?.isEnabled || false}
                    onCheckedChange={(enabled) => 
                      updateFeature({ id: service.id, isEnabled: enabled })
                    }
                  />
                </div>
                <CardDescription className="text-xs">
                  {service.category} • {service.id}
                </CardDescription>
              </CardHeader>
              <CardContent className="space-y-4">
                <div className="flex items-center justify-between text-sm">
                  <span className="text-muted-foreground">Status</span>
                  <Badge variant={feature?.status === 'healthy' ? 'success' : 'destructive'}>
                    {feature?.status || 'unknown'}
                  </Badge>
                </div>
                
                <div className="space-y-2">
                  <Label className="text-xs">Rate Limit (req/min)</Label>
                  <div className="flex items-center gap-2">
                    <Slider
                      value={[feature?.rateLimit || 1000]}
                      onValueChange={([value]) => 
                        updateFeature({ id: service.id, rateLimit: value })
                      }
                      max={10000}
                      step={100}
                      className="flex-1"
                    />
                    <span className="text-xs w-12 text-right">
                      {feature?.rateLimit || 1000}
                    </span>
                  </div>
                </div>

                <Accordion type="single" collapsible>
                  <AccordionItem value="config">
                    <AccordionTrigger className="text-xs">
                      Advanced Configuration
                    </AccordionTrigger>
                    <AccordionContent className="space-y-3">
                      <div className="space-y-2">
                        <Label className="text-xs">Timeout (ms)</Label>
                        <Input
                          type="number"
                          value={feature?.timeout || 30000}
                          onChange={(e) => 
                            updateFeature({ 
                              id: service.id, 
                              timeout: parseInt(e.target.value) 
                            })
                          }
                        />
                      </div>
                      
                      <div className="space-y-2">
                        <Label className="text-xs">Max Retries</Label>
                        <Input
                          type="number"
                          value={feature?.maxRetries || 3}
                          onChange={(e) => 
                            updateFeature({ 
                              id: service.id, 
                              maxRetries: parseInt(e.target.value) 
                            })
                          }
                        />
                      </div>

                      <div className="space-y-2">
                        <Label className="text-xs">Feature Flags</Label>
                        <div className="space-y-1">
                          {feature?.flags?.map((flag) => (
                            <div key={flag.name} className="flex items-center justify-between">
                              <span className="text-xs">{flag.name}</span>
                              <Switch
                                checked={flag.enabled}
                                onCheckedChange={(enabled) => 
                                  updateFeature({ 
                                    id: service.id, 
                                    flags: feature.flags.map(f => 
                                      f.name === flag.name ? { ...f, enabled } : f
                                    )
                                  })
                                }
                              />
                            </div>
                          ))}
                        </div>
                      </div>
                    </AccordionContent>
                  </AccordionItem>
                </Accordion>
              </CardContent>
            </Card>
          );
        })}
      </div>
    </div>
  );
}
```

### Permission Matrix Component
```typescript
// components/features/admin/permission-matrix.tsx
'use client';

import { useState } from 'react';
import { useQuery } from '@tanstack/react-query';
import {
  Table,
  TableBody,
  TableCell,
  TableHead,
  TableHeader,
  TableRow,
} from '@/components/ui/table';
import { Checkbox } from '@/components/ui/checkbox';
import { Badge } from '@/components/ui/badge';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import {
  Select,
  SelectContent,
  SelectItem,
  SelectTrigger,
  SelectValue,
} from '@/components/ui/select';
import { adminService } from '@/services/admin-service';
import { AdminType, Action } from '@/types/admin.types';
import { Save, Download } from 'lucide-react';
import { toast } from 'sonner';

export function PermissionMatrix() {
  const [search, setSearch] = useState('');
  const [selectedAdminType, setSelectedAdminType] = useState<AdminType>(AdminType.PLATFORM_ADMIN);
  const [modifiedPermissions, setModifiedPermissions] = useState<Record<string, any>>({});

  const { data: permissions, isLoading } = useQuery({
    queryKey: ['permission-matrix', selectedAdminType],
    queryFn: () => adminService.getPermissionMatrix(selectedAdminType),
  });

  const handlePermissionChange = (feature: string, action: Action, enabled: boolean) => {
    setModifiedPermissions(prev => ({
      ...prev,
      [`${feature}-${action}`]: enabled,
    }));
  };

  const handleSave = async () => {
    try {
      await adminService.updatePermissionMatrix(selectedAdminType, modifiedPermissions);
      toast.success('Permissions updated successfully');
      setModifiedPermissions({});
    } catch (error) {
      toast.error('Failed to update permissions');
    }
  };

  const handleExport = () => {
    const data = permissions?.map(p => ({
      Feature: p.feature,
      ...Object.fromEntries(
        Object.values(Action).map(action => [
          action,
          p.permissions.includes(action) ? 'Yes' : 'No'
        ])
      ),
    }));
    
    const csv = [
      Object.keys(data[0]).join(','),
      ...data.map(row => Object.values(row).join(','))
    ].join('\n');
    
    const blob = new Blob([csv], { type: 'text/csv' });
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = `permissions-${selectedAdminType}.csv`;
    a.click();
  };

  const filteredPermissions = permissions?.filter(p => 
    p.feature.toLowerCase().includes(search.toLowerCase())
  );

  return (
    <div className="space-y-4">
      <div className="flex items-center justify-between">
        <div className="flex items-center gap-4">
          <Input
            placeholder="Search features..."
            value={search}
            onChange={(e) => setSearch(e.target.value)}
            className="w-64"
          />
          <Select value={selectedAdminType} onValueChange={setSelectedAdminType}>
            <SelectTrigger className="w-48">
              <SelectValue />
            </SelectTrigger>
            <SelectContent>
              {Object.values(AdminType).map((type) => (
                <SelectItem key={type} value={type}>
                  {type.replace(/_/g, ' ')}
                </SelectItem>
              ))}
            </SelectContent>
          </Select>
        </div>
        <div className="flex items-center gap-2">
          <Button
            variant="outline"
            size="sm"
            onClick={handleExport}
          >
            <Download className="h-4 w-4 mr-2" />
            Export CSV
          </Button>
          <Button
            size="sm"
            onClick={handleSave}
            disabled={Object.keys(modifiedPermissions).length === 0}
          >
            <Save className="h-4 w-4 mr-2" />
            Save Changes
          </Button>
        </div>
      </div>

      <div className="border rounded-lg">
        <Table>
          <TableHeader>
            <TableRow>
              <TableHead className="w-64">Feature / Microservice</TableHead>
              {Object.values(Action).map((action) => (
                <TableHead key={action} className="text-center">
                  {action}
                </TableHead>
              ))}
            </TableRow>
          </TableHeader>
          <TableBody>
            {filteredPermissions?.map((permission) => (
              <TableRow key={permission.feature}>
                <TableCell className="font-medium">
                  <div>
                    <p>{permission.featureName}</p>
                    <p className="text-xs text-muted-foreground">{permission.feature}</p>
                  </div>
                </TableCell>
                {Object.values(Action).map((action) => {
                  const key = `${permission.feature}-${action}`;
                  const isEnabled = modifiedPermissions[key] ?? 
                    permission.permissions.includes(action);
                  const isModified = key in modifiedPermissions;
                  
                  return (
                    <TableCell key={action} className="text-center">
                      <div className="flex items-center justify-center">
                        <Checkbox
                          checked={isEnabled}
                          onCheckedChange={(checked) => 
                            handlePermissionChange(permission.feature, action, !!checked)
                          }
                        />
                        {isModified && (
                          <Badge variant="outline" className="ml-2 text-xs">
                            Modified
                          </Badge>
                        )}
                      </div>
                    </TableCell>
                  );
                })}
              </TableRow>
            ))}
          </TableBody>
        </Table>
      </div>
    </div>
  );
}
```

## Developer Portal & API Documentation

### Developer Portal Homepage
```typescript
// app/[locale]/(developers)/developers/page.tsx
import { Metadata } from 'next';
import { DeveloperDashboard } from '@/components/features/developers/developer-dashboard';
import { APIQuickStart } from '@/components/features/developers/api-quickstart';
import { APIStats } from '@/components/features/developers/api-stats';

export const metadata: Metadata = {
  title: 'EventfulHQ Developer Portal',
  description: 'Build amazing applications with EventfulHQ APIs',
};

export default function DevelopersPage() {
  return (
    <div className="container py-8 space-y-8">
      <div className="text-center space-y-4">
        <h1 className="text-4xl font-bold">EventfulHQ Developer Portal</h1>
        <p className="text-xl text-muted-foreground max-w-2xl mx-auto">
          Access our comprehensive APIs to build innovative event management solutions
        </p>
      </div>

      <APIStats />
      <APIQuickStart />
      <DeveloperDashboard />
    </div>
  );
}
```

### API Documentation Component
```typescript
// components/features/developers/api-documentation.tsx
'use client';

import { useState, useEffect } from 'react';
import dynamic from 'next/dynamic';
import { Tabs, TabsContent, TabsList, TabsTrigger } from '@/components/ui/tabs';
import { Card, CardContent, CardDescription, CardHeader, CardTitle } from '@/components/ui/card';
import { Button } from '@/components/ui/button';
import { Badge } from '@/components/ui/badge';
import { CodeBlock } from './code-block';
import { APIPlayground } from './api-playground';
import { useQuery } from '@tanstack/react-query';
import { developerService } from '@/services/developer-service';
import { Book, Code, Play, Download } from 'lucide-react';

// Dynamic import for Swagger UI
const SwaggerUI = dynamic(() => import('swagger-ui-react'), { 
  ssr: false,
  loading: () => <div>Loading API documentation...</div>
});

// Import Swagger UI CSS
import 'swagger-ui-react/swagger-ui.css';

export function APIDocumentation() {
  const [selectedService, setSelectedService] = useState('event-service');
  const [selectedLanguage, setSelectedLanguage] = useState('javascript');

  const { data: apiSpec } = useQuery({
    queryKey: ['api-spec', selectedService],
    queryFn: () => developerService.getAPISpec(selectedService),
  });

  const { data: sdks } = useQuery({
    queryKey: ['sdks'],
    queryFn: developerService.getSDKs,
  });

  const services = [
    { id: 'event-service', name: 'Event API', version: 'v1' },
    { id: 'user-service', name: 'User API', version: 'v1' },
    { id: 'rfp-service', name: 'RFP API', version: 'v1' },
    { id: 'analytics-service', name: 'Analytics API', version: 'v1' },
    { id: 'whatsapp-service', name: 'WhatsApp API', version: 'v1' },
    // ... other services
  ];

  const codeSamples = {
    javascript: `// Install the SDK
npm install @eventfulhq/sdk

// Initialize the client
import { EventfulHQ } from '@eventfulhq/sdk';

const client = new EventfulHQ({
  apiKey: 'your-api-key',
  region: 'global' // or 'america', 'arabia', 'india', 'europe', 'australia'
});

// Create an event
const event = await client.events.create({
  title: 'Summer Music Festival',
  description: 'A fantastic outdoor music experience',
  date: '2025-07-15T18:00:00Z',
  venue: {
    name: 'Central Park',
    address: 'New York, NY',
    coordinates: { lat: 40.7829, lng: -73.9654 }
  },
  capacity: 5000,
  price: 75,
  currency: 'USD'
});

// Send WhatsApp invitations
await client.whatsapp.sendInvitations({
  eventId: event.id,
  phoneNumbers: ['+1234567890', '+0987654321'],
  message: 'You are invited to the Summer Music Festival!'
});`,
    python: `# Install the SDK
pip install eventfulhq

# Initialize the client
from eventfulhq import EventfulHQ

client = EventfulHQ(
    api_key='your-api-key',
    region='global'  # or 'america', 'arabia', 'india', 'europe', 'australia'
)

# Create an event
event = client.events.create(
    title='Summer Music Festival',
    description='A fantastic outdoor music experience',
    date='2025-07-15T18:00:00Z',
    venue={
        'name': 'Central Park',
        'address': 'New York, NY',
        'coordinates': {'lat': 40.7829, 'lng': -73.9654}
    },
    capacity=5000,
    price=75,
    currency='USD'
)

# Send WhatsApp invitations
client.whatsapp.send_invitations(
    event_id=event.id,
    phone_numbers=['+1234567890', '+0987654321'],
    message='You are invited to the Summer Music Festival!'
)`,
    curl: `# Get your API key from the developer portal
API_KEY="your-api-key"

# Create an event
curl -X POST https://api.eventfulhq.com/v1/events \\
  -H "Authorization: Bearer $API_KEY" \\
  -H "Content-Type: application/json" \\
  -d '{
    "title": "Summer Music Festival",
    "description": "A fantastic outdoor music experience",
    "date": "2025-07-15T18:00:00Z",
    "venue": {
      "name": "Central Park",
      "address": "New York, NY",
      "coordinates": {"lat": 40.7829, "lng": -73.9654}
    },
    "capacity": 5000,
    "price": 75,
    "currency": "USD"
  }'

# Send WhatsApp invitations
curl -X POST https://api.eventfulhq.com/v1/events/{eventId}/whatsapp-invite \\
  -H "Authorization: Bearer $API_KEY" \\
  -H "Content-Type: application/json" \\
  -d '{
    "phoneNumbers": ["+1234567890", "+0987654321"],
    "message": "You are invited to the Summer Music Festival!"
  }'`,
  };

  return (
    <div className="space-y-6">
      <div className="flex items-center justify-between">
        <div className="flex items-center gap-4">
          <h2 className="text-2xl font-bold">API Documentation</h2>
          <Badge variant="outline">API v1</Badge>
        </div>
        <div className="flex items-center gap-2">
          <Button variant="outline" size="sm">
            <Download className="h-4 w-4 mr-2" />
            Download OpenAPI Spec
          </Button>
          <Button variant="outline" size="sm">
            <Book className="h-4 w-4 mr-2" />
            API Changelog
          </Button>
        </div>
      </div>

      <Tabs defaultValue="reference" className="space-y-4">
        <TabsList className="grid w-full grid-cols-4">
          <TabsTrigger value="quickstart">Quick Start</TabsTrigger>
          <TabsTrigger value="reference">API Reference</TabsTrigger>
          <TabsTrigger value="playground">Playground</TabsTrigger>
          <TabsTrigger value="sdks">SDKs</TabsTrigger>
        </TabsList>

        <TabsContent value="quickstart" className="space-y-4">
          <Card>
            <CardHeader>
              <CardTitle>Getting Started</CardTitle>
              <CardDescription>
                Start building with EventfulHQ APIs in minutes
              </CardDescription>
            </CardHeader>
            <CardContent className="space-y-4">
              <div className="space-y-2">
                <h3 className="font-semibold">1. Get your API key</h3>
                <p className="text-sm text-muted-foreground">
                  Sign in to the developer portal and create a new API key from the dashboard
                </p>
              </div>

              <div className="space-y-2">
                <h3 className="font-semibold">2. Choose your integration method</h3>
                <div className="flex gap-2">
                  {['javascript', 'python', 'curl'].map((lang) => (
                    <Button
                      key={lang}
                      variant={selectedLanguage === lang ? 'default' : 'outline'}
                      size="sm"
                      onClick={() => setSelectedLanguage(lang)}
                    >
                      {lang.charAt(0).toUpperCase() + lang.slice(1)}
                    </Button>
                  ))}
                </div>
              </div>

              <div className="space-y-2">
                <h3 className="font-semibold">3. Make your first API call</h3>
                <CodeBlock
                  language={selectedLanguage}
                  code={codeSamples[selectedLanguage]}
                />
              </div>
            </CardContent>
          </Card>
        </TabsContent>

        <TabsContent value="reference" className="space-y-4">
          <div className="flex gap-4">
            <div className="w-64 space-y-2">
              <h3 className="font-semibold text-sm">Services</h3>
              {services.map((service) => (
                <Button
                  key={service.id}
                  variant={selectedService === service.id ? 'secondary' : 'ghost'}
                  size="sm"
                  className="w-full justify-start"
                  onClick={() => setSelectedService(service.id)}
                >
                  {service.name}
                  <Badge variant="outline" className="ml-auto">
                    {service.version}
                  </Badge>
                </Button>
              ))}
            </div>
            <div className="flex-1">
              <Card>
                <CardContent className="p-0">
                  {apiSpec && (
                    <SwaggerUI
                      spec={apiSpec}
                      docExpansion="list"
                      defaultModelsExpandDepth={1}
                      tryItOutEnabled={true}
                    />
                  )}
                </CardContent>
              </Card>
            </div>
          </div>
        </TabsContent>

        <TabsContent value="playground" className="space-y-4">
          <APIPlayground />
        </TabsContent>

        <TabsContent value="sdks" className="space-y-4">
          <div className="grid gap-4 md:grid-cols-2 lg:grid-cols-3">
            {sdks?.map((sdk) => (
              <Card key={sdk.language}>
                <CardHeader>
                  <CardTitle className="flex items-center justify-between">
                    {sdk.name}
                    <Badge>{sdk.version}</Badge>
                  </CardTitle>
                  <CardDescription>{sdk.description}</CardDescription>
                </CardHeader>
                <CardContent className="space-y-4">
                  <CodeBlock
                    language="bash"
                    code={sdk.installCommand}
                  />
                  <div className="flex gap-2">
                    <Button variant="outline" size="sm" asChild>
                      <a href={sdk.githubUrl} target="_blank" rel="noopener">
                        <Code className="h-4 w-4 mr-2" />
                        GitHub
                      </a>
                    </Button>
                    <Button variant="outline" size="sm" asChild>
                      <a href={sdk.docsUrl} target="_blank" rel="noopener">
                        <Book className="h-4 w-4 mr-2" />
                        Docs
                      </a>
                    </Button>
                  </div>
                </CardContent>
              </Card>
            ))}
          </div>
        </TabsContent>
      </Tabs>
    </div>
  );
}
```

### API Key Management
```typescript
// components/features/developers/api-key-manager.tsx
'use client';

import { useState } from 'react';
import { useQuery, useMutation } from '@tanstack/react-query';
import { Card, CardContent, CardDescription, CardHeader, CardTitle } from '@/components/ui/card';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { Label } from '@/components/ui/label';
import { Badge } from '@/components/ui/badge';
import {
  Dialog,
  DialogContent,
  DialogDescription,
  DialogHeader,
  DialogTitle,
  DialogTrigger,
} from '@/components/ui/dialog';
import {
  Table,
  TableBody,
  TableCell,
  TableHead,
  TableHeader,
  TableRow,
} from '@/components/ui/table';
import { developerService } from '@/services/developer-service';
import { Copy, Eye, EyeOff, Key, Trash, Plus } from 'lucide-react';
import { toast } from 'sonner';

export function APIKeyManager() {
  const [showCreateDialog, setShowCreateDialog] = useState(false);
  const [newKeyName, setNewKeyName] = useState('');
  const [selectedScopes, setSelectedScopes] = useState<string[]>([]);
  const [visibleKeys, setVisibleKeys] = useState<Set<string>>(new Set());

  const { data: apiKeys, isLoading } = useQuery({
    queryKey: ['api-keys'],
    queryFn: developerService.getAPIKeys,
  });

  const { mutate: createKey } = useMutation({
    mutationFn: developerService.createAPIKey,
    onSuccess: (data) => {
      toast.success('API key created successfully');
      // Show the key once for copying
      navigator.clipboard.writeText(data.key);
      toast.info('API key copied to clipboard. This is the only time you can see it!');
      setShowCreateDialog(false);
      setNewKeyName('');
      setSelectedScopes([]);
      queryClient.invalidateQueries({ queryKey: ['api-keys'] });
    },
  });

  const { mutate: deleteKey } = useMutation({
    mutationFn: developerService.deleteAPIKey,
    onSuccess: () => {
      toast.success('API key deleted successfully');
      queryClient.invalidateQueries({ queryKey: ['api-keys'] });
    },
  });

  const scopes = [
    { id: 'events:read', name: 'Read Events' },
    { id: 'events:write', name: 'Write Events' },
    { id: 'users:read', name: 'Read Users' },
    { id: 'users:write', name: 'Write Users' },
    { id: 'analytics:read', name: 'Read Analytics' },
    { id: 'whatsapp:send', name: 'Send WhatsApp Messages' },
    { id: 'rfp:manage', name: 'Manage RFPs' },
    { id: 'payments:process', name: 'Process Payments' },
  ];

  const toggleKeyVisibility = (keyId: string) => {
    setVisibleKeys(prev => {
      const newSet = new Set(prev);
      if (newSet.has(keyId)) {
        newSet.delete(keyId);
      } else {
        newSet.add(keyId);
      }
      return newSet;
    });
  };

  return (
    <Card>
      <CardHeader>
        <div className="flex items-center justify-between">
          <div>
            <CardTitle>API Keys</CardTitle>
            <CardDescription>
              Manage your API keys for accessing EventfulHQ APIs
            </CardDescription>
          </div>
          <Dialog open={showCreateDialog} onOpenChange={setShowCreateDialog}>
            <DialogTrigger asChild>
              <Button>
                <Plus className="h-4 w-4 mr-2" />
                Create API Key
              </Button>
            </DialogTrigger>
            <DialogContent>
              <DialogHeader>
                <DialogTitle>Create New API Key</DialogTitle>
                <DialogDescription>
                  Generate a new API key with specific permissions
                </DialogDescription>
              </DialogHeader>
              <div className="space-y-4">
                <div className="space-y-2">
                  <Label>Key Name</Label>
                  <Input
                    placeholder="My App API Key"
                    value={newKeyName}
                    onChange={(e) => setNewKeyName(e.target.value)}
                  />
                </div>
                <div className="space-y-2">
                  <Label>Permissions</Label>
                  <div className="space-y-2">
                    {scopes.map((scope) => (
                      <div key={scope.id} className="flex items-center space-x-2">
                        <Checkbox
                          id={scope.id}
                          checked={selectedScopes.includes(scope.id)}
                          onCheckedChange={(checked) => {
                            if (checked) {
                              setSelectedScopes([...selectedScopes, scope.id]);
                            } else {
                              setSelectedScopes(selectedScopes.filter(s => s !== scope.id));
                            }
                          }}
                        />
                        <Label htmlFor={scope.id} className="text-sm">
                          {scope.name}
                        </Label>
                      </div>
                    ))}
                  </div>
                </div>
                <Button
                  className="w-full"
                  onClick={() => createKey({ name: newKeyName, scopes: selectedScopes })}
                  disabled={!newKeyName || selectedScopes.length === 0}
                >
                  Create API Key
                </Button>
              </div>
            </DialogContent>
          </Dialog>
        </div>
      </CardHeader>
      <CardContent>
        <Table>
          <TableHeader>
            <TableRow>
              <TableHead>Name</TableHead>
              <TableHead>Key</TableHead>
              <TableHead>Permissions</TableHead>
              <TableHead>Last Used</TableHead>
              <TableHead>Created</TableHead>
              <TableHead></TableHead>
            </TableRow>
          </TableHeader>
          <TableBody>
            {apiKeys?.map((apiKey) => (
              <TableRow key={apiKey.id}>
                <TableCell className="font-medium">{apiKey.name}</TableCell>
                <TableCell>
                  <div className="flex items-center gap-2">
                    <code className="text-xs bg-muted px-2 py-1 rounded">
                      {visibleKeys.has(apiKey.id) 
                        ? apiKey.key 
                        : `${apiKey.key.slice(0, 12)}...`}
                    </code>
                    <Button
                      variant="ghost"
                      size="icon"
                      className="h-6 w-6"
                      onClick={() => toggleKeyVisibility(apiKey.id)}
                    >
                      {visibleKeys.has(apiKey.id) ? (
                        <EyeOff className="h-3 w-3" />
                      ) : (
                        <Eye className="h-3 w-3" />
                      )}
                    </Button>
                    <Button
                      variant="ghost"
                      size="icon"
                      className="h-6 w-6"
                      onClick={() => {
                        navigator.clipboard.writeText(apiKey.key);
                        toast.success('API key copied to clipboard');
                      }}
                    >
                      <Copy className="h-3 w-3" />
                    </Button>
                  </div>
                </TableCell>
                <TableCell>
                  <div className="flex flex-wrap gap-1">
                    {apiKey.scopes.slice(0, 2).map((scope) => (
                      <Badge key={scope} variant="secondary" className="text-xs">
                        {scope}
                      </Badge>
                    ))}
                    {apiKey.scopes.length > 2 && (
                      <Badge variant="secondary" className="text-xs">
                        +{apiKey.scopes.length - 2}
                      </Badge>
                    )}
                  </div>
                </TableCell>
                <TableCell>
                  {apiKey.lastUsed ? (
                    <span className="text-sm text-muted-foreground">
                      {new Date(apiKey.lastUsed).toLocaleDateString()}
                    </span>
                  ) : (
                    <span className="text-sm text-muted-foreground">Never</span>
                  )}
                </TableCell>
                <TableCell>
                  <span className="text-sm text-muted-foreground">
                    {new Date(apiKey.createdAt).toLocaleDateString()}
                  </span>
                </TableCell>
                <TableCell>
                  <Button
                    variant="ghost"
                    size="icon"
                    onClick={() => deleteKey(apiKey.id)}
                  >
                    <Trash className="h-4 w-4" />
                  </Button>
                </TableCell>
              </TableRow>
            ))}
          </TableBody>
        </Table>
      </CardContent>
    </Card>
  );
}
```

## Landing Pages & Microsites

### Dynamic City Landing Page
```typescript
// app/[locale]/(landing)/[country]/[city]/page.tsx
import { Metadata } from 'next';
import { notFound } from 'next/navigation';
import { CityHero } from '@/components/features/landing/city-hero';
import { TrendingEvents } from '@/components/features/landing/trending-events';
import { TrendingArtists } from '@/components/features/landing/trending-artists';
import { TrendingVenues } from '@/components/features/landing/trending-venues';
import { AdSenseBanner } from '@/components/features/landing/adsense-banner';
import { StructuredData } from '@/components/features/seo/structured-data';
import { landingService } from '@/services/landing-service';

interface CityPageProps {
  params: {
    locale: string;
    country: string;
    city: string;
  };
}

export async function generateMetadata({ params }: CityPageProps): Promise<Metadata> {
  const cityData = await landingService.getCityData(params.country, params.city);
  
  if (!cityData) return {};
  
  return {
    title: `Events in ${cityData.name} - ${cityData.eventCount} Upcoming Events | EventfulHQ`,
    description: `Discover ${cityData.eventCount} amazing events happening in ${cityData.name}. Find concerts, festivals, workshops, and more. Book tickets for the best events in ${cityData.name}.`,
    openGraph: {
      title: `Events in ${cityData.name}`,
      description: `Discover amazing events in ${cityData.name}`,
      images: [cityData.heroImage],
      type: 'website',
    },
    alternates: {
      canonical: `https://eventfulhq.com/${params.locale}/${params.country}/${params.city}`,
      languages: {
        'en': `https://eventfulhq.com/en/${params.country}/${params.city}`,
        'es': `https://eventfulhq.com/es/${params.country}/${params.city}`,
        // ... other languages
      },
    },
  };
}

export async function generateStaticParams() {
  // Generate static pages for top cities
  const topCities = await landingService.getTopCities();
  
  return topCities.map((city) => ({
    country: city.countryCode.toLowerCase(),
    city: city.slug,
  }));
}

export default async function CityLandingPage({ params }: CityPageProps) {
  const cityData = await landingService.getCityData(params.country, params.city);
  
  if (!cityData) {
    notFound();
  }
  
  const [trendingEvents, trendingArtists, trendingVenues] = await Promise.all([
    landingService.getTrendingEvents(cityData.id),
    landingService.getTrendingArtists(cityData.id),
    landingService.getTrendingVenues(cityData.id),
  ]);
  
  const structuredData = {
    '@context': 'https://schema.org',
    '@type': 'City',
    name: cityData.name,
    geo: {
      '@type': 'GeoCoordinates',
      latitude: cityData.coordinates.lat,
      longitude: cityData.coordinates.lng,
    },
    containsPlace: trendingVenues.map(venue => ({
      '@type': 'EventVenue',
      name: venue.name,
      address: venue.address,
    })),
    event: trendingEvents.map(event => ({
      '@type': 'Event',
      name: event.title,
      startDate: event.date,
      location: {
        '@type': 'Place',
        name: event.venue.name,
        address: event.venue.address,
      },
      offers: {
        '@type': 'Offer',
        price: event.price,
        priceCurrency: event.currency,
        availability: event.isSoldOut ? 'SoldOut' : 'InStock',
      },
    })),
  };
  
  return (
    <>
      <StructuredData data={structuredData} />
      
      <div className="min-h-screen">
        <CityHero city={cityData} />
        
        <div className="container py-8 space-y-12">
          {/* AdSense Banner - Top */}
          <AdSenseBanner 
            slot="city-landing-top" 
            format="horizontal"
            className="mb-8"
          />
          
          {/* Trending Events Section */}
          <section>
            <h2 className="text-3xl font-bold mb-6">
              Trending Events in {cityData.name}
            </h2>
            <TrendingEvents events={trendingEvents} city={cityData} />
          </section>
          
          {/* AdSense Banner - Middle */}
          <AdSenseBanner 
            slot="city-landing-middle" 
            format="rectangle"
            className="my-8"
          />
          
          {/* Trending Artists Section */}
          <section>
            <h2 className="text-3xl font-bold mb-6">
              Popular Artists in {cityData.name}
            </h2>
            <TrendingArtists artists={trendingArtists} />
          </section>
          
          {/* Trending Venues Section */}
          <section>
            <h2 className="text-3xl font-bold mb-6">
              Top Venues in {cityData.name}
            </h2>
            <TrendingVenues venues={trendingVenues} />
          </section>
          
          {/* AdSense Banner - Bottom */}
          <AdSenseBanner 
            slot="city-landing-bottom" 
            format="horizontal"
            className="mt-8"
          />
          
          {/* SEO Content */}
          <section className="prose max-w-none">
            <h3>About Events in {cityData.name}</h3>
            <p>{cityData.description}</p>
            <p>
              With over {cityData.eventCount} upcoming events, {cityData.name} is 
              a vibrant hub for entertainment and culture. From live music concerts 
              to art exhibitions, food festivals to tech conferences, there's always 
              something exciting happening in {cityData.name}.
            </p>
          </section>
        </div>
      </div>
    </>
  );
}
```

### User Microsite (Without Ads for Premium Users)
```typescript
// app/[locale]/(landing)/u/[username]/page.tsx
import { Metadata } from 'next';
import { notFound } from 'next/navigation';
import { UserProfile } from '@/components/features/landing/user-profile';
import { UserEvents } from '@/components/features/landing/user-events';
import { UserStats } from '@/components/features/landing/user-stats';
import { ShareButtons } from '@/components/features/landing/share-buttons';
import { AdSenseBanner } from '@/components/features/landing/adsense-banner';
import { landingService } from '@/services/landing-service';
import { userService } from '@/services/user-service';

interface UserMicrositeProps {
  params: {
    locale: string;
    username: string;
  };
}

export async function generateMetadata({ params }: UserMicrositeProps): Promise<Metadata> {
  const user = await userService.getUserByUsername(params.username);
  
  if (!user) return {};
  
  return {
    title: `${user.name} - Event Organizer | EventfulHQ`,
    description: user.bio || `Check out events organized by ${user.name} on EventfulHQ`,
    openGraph: {
      title: user.name,
      description: user.bio,
      images: [user.avatar],
      type: 'profile',
    },
  };
}

export default async function UserMicrosite({ params }: UserMicrositeProps) {
  const user = await userService.getUserByUsername(params.username);
  
  if (!user) {
    notFound();
  }
  
  const [userEvents, userStats] = await Promise.all([
    landingService.getUserEvents(user.id),
    landingService.getUserStats(user.id),
  ]);
  
  const isPremium = user.subscription?.tier === 'premium';
  const customTheme = user.micrositeTheme || 'default';
  
  return (
    <div className={`min-h-screen theme-${customTheme}`}>
      <UserProfile user={user} stats={userStats} />
      
      <div className="container py-8 space-y-8">
        {/* No ads for premium users */}
        {!isPremium && (
          <AdSenseBanner 
            slot="user-microsite-top" 
            format="horizontal"
          />
        )}
        
        <section>
          <div className="flex items-center justify-between mb-6">
            <h2 className="text-2xl font-bold">Upcoming Events</h2>
            <ShareButtons 
              url={`https://eventfulhq.com/u/${user.username}`}
              title={`Check out events by ${user.name}`}
            />
          </div>
          <UserEvents events={userEvents} showOrganizer={false} />
        </section>
        
        {!isPremium && userEvents.length > 6 && (
          <AdSenseBanner 
            slot="user-microsite-middle" 
            format="rectangle"
          />
        )}
        
        <UserStats stats={userStats} />
        
        {!isPremium && (
          <AdSenseBanner 
            slot="user-microsite-bottom" 
            format="horizontal"
          />
        )}
      </div>
    </div>
  );
}
```

### Event Microsite Component
```typescript
// app/[locale]/(landing)/e/[eventSlug]/page.tsx
import { Metadata } from 'next';
import { notFound } from 'next/navigation';
import { EventHero } from '@/components/features/landing/event-hero';
import { EventDetails } from '@/components/features/landing/event-details';
import { TicketWidget } from '@/components/features/landing/ticket-widget';
import { WhatsAppRSVP } from '@/components/features/landing/whatsapp-rsvp';
import { AdSenseBanner } from '@/components/features/landing/adsense-banner';
import { StructuredData } from '@/components/features/seo/structured-data';
import { eventService } from '@/services/event-service';

interface EventMicrositeProps {
  params: {
    locale: string;
    eventSlug: string;
  };
}

export async function generateMetadata({ params }: EventMicrositeProps): Promise<Metadata> {
  const event = await eventService.getEventBySlug(params.eventSlug);
  
  if (!event) return {};
  
  return {
    title: `${event.title} - ${new Date(event.date).toLocaleDateString()} | EventfulHQ`,
    description: event.description,
    openGraph: {
      title: event.title,
      description: event.description,
      images: [event.coverImage],
      type: 'website',
    },
  };
}

export default async function EventMicrosite({ params }: EventMicrositeProps) {
  const event = await eventService.getEventBySlug(params.eventSlug);
  
  if (!event) {
    notFound();
  }
  
  const isPremiumEvent = event.organizer.subscription?.tier === 'premium';
  const customTheme = event.micrositeTheme || event.organizer.micrositeTheme || 'default';
  
  const structuredData = {
    '@context': 'https://schema.org',
    '@type': 'Event',
    name: event.title,
    description: event.description,
    startDate: event.date,
    endDate: event.endDate,
    location: {
      '@type': 'Place',
      name: event.venue.name,
      address: {
        '@type': 'PostalAddress',
        streetAddress: event.venue.address,
        addressLocality: event.venue.city,
        addressCountry: event.venue.country,
      },
      geo: {
        '@type': 'GeoCoordinates',
        latitude: event.venue.coordinates.lat,
        longitude: event.venue.coordinates.lng,
      },
    },
    offers: {
      '@type': 'Offer',
      url: `https://eventfulhq.com/e/${event.slug}`,
      price: event.price,
      priceCurrency: event.currency,
      availability: event.isSoldOut 
        ? 'https://schema.org/SoldOut' 
        : 'https://schema.org/InStock',
      validFrom: new Date().toISOString(),
    },
    performer: event.artists?.map(artist => ({
      '@type': 'Person',
      name: artist.name,
    })),
    organizer: {
      '@type': 'Organization',
      name: event.organizer.name,
      url: `https://eventfulhq.com/u/${event.organizer.username}`,
    },
  };
  
  return (
    <>
      <StructuredData data={structuredData} />
      
      <div className={`min-h-screen theme-${customTheme}`}>
        <EventHero event={event} />
        
        <div className="container py-8">
          <div className="grid gap-8 lg:grid-cols-3">
            <div className="lg:col-span-2 space-y-8">
              <EventDetails event={event} />
              
              {!isPremiumEvent && (
                <AdSenseBanner 
                  slot="event-microsite-content" 
                  format="rectangle"
                />
              )}
              
              {/* Event Description */}
              <section>
                <h2 className="text-2xl font-bold mb-4">About This Event</h2>
                <div className="prose max-w-none">
                  {event.description}
                </div>
              </section>
              
              {/* Artists/Performers */}
              {event.artists && event.artists.length > 0 && (
                <section>
                  <h2 className="text-2xl font-bold mb-4">Featuring</h2>
                  <div className="grid gap-4 sm:grid-cols-2">
                    {event.artists.map(artist => (
                      <ArtistCard key={artist.id} artist={artist} />
                    ))}
                  </div>
                </section>
              )}
            </div>
            
            <div className="space-y-6">
              {/* Ticket Widget */}
              <TicketWidget event={event} />
              
              {/* WhatsApp RSVP */}
              {event.isWhatsAppRSVP && (
                <WhatsAppRSVP event={event} />
              )}
              
              {!isPremiumEvent && (
                <AdSenseBanner 
                  slot="event-microsite-sidebar" 
                  format="vertical"
                />
              )}
              
              {/* Venue Information */}
              <VenueInfo venue={event.venue} />
              
              {/* Share Buttons */}
              <ShareButtons 
                url={`https://eventfulhq.com/e/${event.slug}`}
                title={event.title}
                description={event.description}
              />
            </div>
          </div>
        </div>
        
        {!isPremiumEvent && (
          <AdSenseBanner 
            slot="event-microsite-bottom" 
            format="horizontal"
            className="mt-8"
          />
        )}
      </div>
    </>
  );
}
```

### AdSense Integration Component
```typescript
// components/features/landing/adsense-banner.tsx
'use client';

import { useEffect } from 'react';
import { usePathname } from 'next/navigation';
import Script from 'next/script';

interface AdSenseBannerProps {
  slot: string;
  format?: 'horizontal' | 'vertical' | 'rectangle' | 'auto';
  className?: string;
  responsive?: boolean;
}

export function AdSenseBanner({ 
  slot, 
  format = 'auto',
  className = '',
  responsive = true 
}: AdSenseBannerProps) {
  const pathname = usePathname();
  
  useEffect(() => {
    try {
      // Push ads after route change
      (window.adsbygoogle = window.adsbygoogle || []).push({});
    } catch (err) {
      console.error('AdSense error:', err);
    }
  }, [pathname]);
  
  const formatStyles = {
    horizontal: { width: '728px', height: '90px' },
    vertical: { width: '300px', height: '600px' },
    rectangle: { width: '336px', height: '280px' },
    auto: { width: '100%', height: 'auto' },
  };
  
  const style = formatStyles[format];
  
  return (
    <>
      <Script
        id="adsense-script"
        async
        src={`https://pagead2.googlesyndication.com/pagead/js/adsbygoogle.js?client=${process.env.NEXT_PUBLIC_ADSENSE_CLIENT_ID}`}
        crossOrigin="anonymous"
        strategy="afterInteractive"
      />
      <div className={`adsense-container ${className}`}>
        <ins
          className="adsbygoogle"
          style={{
            display: 'block',
            width: style.width,
            height: style.height,
            ...(responsive && { 
              width: '100%',
              height: 'auto',
            }),
          }}
          data-ad-client={process.env.NEXT_PUBLIC_ADSENSE_CLIENT_ID}
          data-ad-slot={slot}
          data-ad-format={format}
          data-full-width-responsive={responsive ? 'true' : 'false'}
        />
      </div>
    </>
  );
}

// AdSense configuration for different page types
export const adSlots = {
  'city-landing-top': '1234567890',
  'city-landing-middle': '2345678901',
  'city-landing-bottom': '3456789012',
  'country-landing-top': '4567890123',
  'country-landing-sidebar': '5678901234',
  'user-microsite-top': '6789012345',
  'user-microsite-middle': '7890123456',
  'user-microsite-bottom': '8901234567',
  'event-microsite-content': '9012345678',
  'event-microsite-sidebar': '0123456789',
  'event-microsite-bottom': '1234567890',
};
```

## Bot Management & SEO Strategy

### Bot Detection Middleware
```typescript
// middleware/bot-detection.ts
import { NextRequest, NextResponse } from 'next/server';
import { botPatterns, seoBotsPatterns, llmBotsPatterns } from '@/lib/seo/bot-patterns';

export async function botDetectionMiddleware(request: NextRequest) {
  const userAgent = request.headers.get('user-agent') || '';
  const ip = request.ip || request.headers.get('x-forwarded-for') || '';
  
  // Check if it's a bot
  const isBot = botPatterns.some(pattern => pattern.test(userAgent));
  const isSEOBot = seoBotsPatterns.some(pattern => pattern.test(userAgent));
  const isLLMBot = llmBotsPatterns.some(pattern => pattern.test(userAgent));
  
  // Log bot access for analytics
  if (isBot) {
    await logBotAccess({
      userAgent,
      ip,
      path: request.nextUrl.pathname,
      type: isSEOBot ? 'seo' : isLLMBot ? 'llm' : 'other',
      timestamp: new Date(),
    });
  }
  
  // Block unauthorized bots
  if (isBot && !isSEOBot && !isLLMBot) {
    // Check rate limiting for suspicious bots
    const rateLimit = await checkBotRateLimit(ip);
    
    if (rateLimit.exceeded) {
      return new NextResponse('Too Many Requests', { status: 429 });
    }
    
    // Return minimal response for unauthorized bots
    return new NextResponse('Access Denied', { 
      status: 403,
      headers: {
        'X-Robots-Tag': 'noindex, nofollow',
      }
    });
  }
  
  // For SEO and LLM bots, add special headers and optimizations
  if (isSEOBot || isLLMBot) {
    const response = NextResponse.next();
    
    // Add bot-specific headers
    response.headers.set('X-Bot-Type', isSEOBot ? 'seo' : 'llm');
    response.headers.set('Cache-Control', 'public, max-age=3600');
    
    // For LLM bots, add structured data hints
    if (isLLMBot) {
      response.headers.set('X-LLM-Hints', 'structured-data-available');
    }
    
    return response;
  }
  
  return NextResponse.next();
}

// Bot patterns configuration
export const botPatterns = [
  // Common crawlers and scrapers
  /bot|crawler|spider|scraper|curl|wget|python-requests/i,
  // Specific bad bots
  /ahrefsbot|semrushbot|dotbot|rogerbot|facebookexternalhit|ia_archiver/i,
  // Vulnerability scanners
  /nikto|sqlmap|openvas|nessus|w3af/i,
];

export const seoBotsPatterns = [
  // Search engine bots
  /googlebot|bingbot|slurp|duckduckbot|baiduspider|yandexbot/i,
  // Social media bots
  /linkedinbot|whatsapp|telegrambot/i,
  // SEO tool bots (allowed)
  /screaming frog|deepcrawl|oncrawl/i,
];

export const llmBotsPatterns = [
  // AI/LLM crawlers
  /gpt-?bot|claude-?bot|anthropic|openai/i,
  /chatgpt|bard|llama|perplexity/i,
  // AI research bots
  /commoncrawl|ccbot/i,
];

// Rate limiting for bots
async function checkBotRateLimit(ip: string): Promise<{ exceeded: boolean }> {
  const key = `bot-rate-limit:${ip}`;
  const limit = 100; // requests per hour
  const window = 3600; // 1 hour in seconds
  
  // Implementation would use Redis or similar
  // This is a simplified example
  return { exceeded: false };
}

// Log bot access for analytics
async function logBotAccess(data: {
  userAgent: string;
  ip: string;
  path: string;
  type: string;
  timestamp: Date;
}) {
  // Log to analytics service
  console.log('Bot access:', data);
}
```

### Dynamic Robots.txt Generation
```typescript
// app/robots.txt/route.ts
import { NextRequest } from 'next/server';

export async function GET(request: NextRequest) {
  const host = request.headers.get('host') || 'eventfulhq.com';
  const protocol = process.env.NODE_ENV === 'production' ? 'https' : 'http';
  
  const robotsTxt = `# EventfulHQ Robots.txt
# Generated dynamically based on domain

User-agent: *
Allow: /
Disallow: /api/
Disallow: /admin/
Disallow: /super-admin/
Disallow: /_next/
Disallow: /static/
Crawl-delay: 1

# Search engines
User-agent: Googlebot
Allow: /
Crawl-delay: 0

User-agent: Bingbot
Allow: /
Crawl-delay: 0

User-agent: Slurp
Allow: /
Crawl-delay: 1

User-agent: DuckDuckBot
Allow: /
Crawl-delay: 1

# Social media
User-agent: LinkedInBot
Allow: /

User-agent: WhatsApp
Allow: /

User-agent: Twitterbot
Allow: /

# AI/LLM Bots
User-agent: GPTBot
Allow: /
Crawl-delay: 2

User-agent: Claude-Web
Allow: /
Crawl-delay: 2

User-agent: CCBot
Allow: /
Crawl-delay: 5

# Bad bots
User-agent: AhrefsBot
Disallow: /

User-agent: SemrushBot
Disallow: /

User-agent: DotBot
Disallow: /

User-agent: MJ12bot
Disallow: /

# Sitemaps
Sitemap: ${protocol}://${host}/sitemap.xml
Sitemap: ${protocol}://${host}/sitemap-events.xml
Sitemap: ${protocol}://${host}/sitemap-users.xml
Sitemap: ${protocol}://${host}/sitemap-venues.xml
Sitemap: ${protocol}://${host}/sitemap-cities.xml
Sitemap: ${protocol}://${host}/sitemap-countries.xml
`;

  return new Response(robotsTxt, {
    headers: {
      'Content-Type': 'text/plain',
      'Cache-Control': 'public, max-age=3600',
    },
  });
}
```

### Dynamic Sitemap Generation
```typescript
// app/sitemap.xml/route.ts
import { NextRequest } from 'next/server';
import { sitemapService } from '@/services/sitemap-service';

export async function GET(request: NextRequest) {
  const host = request.headers.get('host') || 'eventfulhq.com';
  const protocol = process.env.NODE_ENV === 'production' ? 'https' : 'http';
  
  // Generate sitemap index
  const sitemaps = [
    'sitemap-static.xml',
    'sitemap-events.xml',
    'sitemap-users.xml',
    'sitemap-venues.xml',
    'sitemap-cities.xml',
    'sitemap-countries.xml',
  ];
  
  const sitemapIndex = `<?xml version="1.0" encoding="UTF-8"?>
<sitemapindex xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
${sitemaps.map(sitemap => `  <sitemap>
    <loc>${protocol}://${host}/${sitemap}</loc>
    <lastmod>${new Date().toISOString()}</lastmod>
  </sitemap>`).join('\n')}
</sitemapindex>`;

  return new Response(sitemapIndex, {
    headers: {
      'Content-Type': 'application/xml',
      'Cache-Control': 'public, max-age=3600',
    },
  });
}

// Dynamic event sitemap
export async function generateEventSitemap() {
  const events = await sitemapService.getPublishedEvents();
  
  const sitemap = `<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9"
        xmlns:image="http://www.google.com/schemas/sitemap-image/1.1">
${events.map(event => `  <url>
    <loc>https://eventfulhq.com/e/${event.slug}</loc>
    <lastmod>${event.updatedAt}</lastmod>
    <changefreq>daily</changefreq>
    <priority>0.8</priority>
    <image:image>
      <image:loc>${event.coverImage}</image:loc>
      <image:title>${event.title}</image:title>
    </image:image>
  </url>`).join('\n')}
</urlset>`;

  return sitemap;
}
```

### SEO Monitoring Dashboard
```typescript
// components/features/seo/seo-monitoring.tsx
'use client';

import { useQuery } from '@tanstack/react-query';
import { Card, CardContent, CardDescription, CardHeader, CardTitle } from '@/components/ui/card';
import { Tabs, TabsContent, TabsList, TabsTrigger } from '@/components/ui/tabs';
import { Progress } from '@/components/ui/progress';
import { Badge } from '@/components/ui/badge';
import {
  LineChart,
  Line,
  BarChart,
  Bar,
  XAxis,
  YAxis,
  CartesianGrid,
  Tooltip,
  ResponsiveContainer,
} from 'recharts';
import { seoService } from '@/services/seo-service';
import { Bot, Search, TrendingUp, AlertCircle } from 'lucide-react';

export function SEOMonitoring() {
  const { data: seoData } = useQuery({
    queryKey: ['seo-monitoring'],
    queryFn: seoService.getMonitoringData,
  });

  const { data: botTraffic } = useQuery({
    queryKey: ['bot-traffic'],
    queryFn: seoService.getBotTraffic,
  });

  return (
    <div className="space-y-6">
      <div className="grid gap-4 md:grid-cols-2 lg:grid-cols-4">
        <Card>
          <CardHeader className="flex flex-row items-center justify-between space-y-0 pb-2">
            <CardTitle className="text-sm font-medium">SEO Score</CardTitle>
            <Search className="h-4 w-4 text-muted-foreground" />
          </CardHeader>
          <CardContent>
            <div className="text-2xl font-bold">{seoData?.seoScore}/100</div>
            <Progress value={seoData?.seoScore} className="mt-2" />
          </CardContent>
        </Card>

        <Card>
          <CardHeader className="flex flex-row items-center justify-between space-y-0 pb-2">
            <CardTitle className="text-sm font-medium">Indexed Pages</CardTitle>
            <TrendingUp className="h-4 w-4 text-muted-foreground" />
          </CardHeader>
          <CardContent>
            <div className="text-2xl font-bold">
              {seoData?.indexedPages.toLocaleString()}
            </div>
            <p className="text-xs text-muted-foreground">
              +{seoData?.newPagesThisWeek} this week
            </p>
          </CardContent>
        </Card>

        <Card>
          <CardHeader className="flex flex-row items-center justify-between space-y-0 pb-2">
            <CardTitle className="text-sm font-medium">Bot Traffic</CardTitle>
            <Bot className="h-4 w-4 text-muted-foreground" />
          </CardHeader>
          <CardContent>
            <div className="text-2xl font-bold">
              {botTraffic?.total.toLocaleString()}
            </div>
            <div className="flex gap-2 mt-2">
              <Badge variant="secondary" className="text-xs">
                SEO: {botTraffic?.seo}
              </Badge>
              <Badge variant="secondary" className="text-xs">
                LLM: {botTraffic?.llm}
              </Badge>
            </div>
          </CardContent>
        </Card>

        <Card>
          <CardHeader className="flex flex-row items-center justify-between space-y-0 pb-2">
            <CardTitle className="text-sm font-medium">Issues</CardTitle>
            <AlertCircle className="h-4 w-4 text-muted-foreground" />
          </CardHeader>
          <CardContent>
            <div className="text-2xl font-bold">{seoData?.issues.total}</div>
            <div className="flex gap-2 mt-2">
              <Badge variant="destructive" className="text-xs">
                Critical: {seoData?.issues.critical}
              </Badge>
              <Badge variant="outline" className="text-xs">
                Warnings: {seoData?.issues.warnings}
              </Badge>
            </div>
          </CardContent>
        </Card>
      </div>

      <Tabs defaultValue="traffic" className="space-y-4">
        <TabsList>
          <TabsTrigger value="traffic">Bot Traffic</TabsTrigger>
          <TabsTrigger value="crawl">Crawl Stats</TabsTrigger>
          <TabsTrigger value="performance">Performance</TabsTrigger>
          <TabsTrigger value="issues">Issues</TabsTrigger>
        </TabsList>

        <TabsContent value="traffic" className="space-y-4">
          <Card>
            <CardHeader>
              <CardTitle>Bot Traffic Over Time</CardTitle>
              <CardDescription>
                SEO and LLM bot visits to your site
              </CardDescription>
            </CardHeader>
            <CardContent>
              <ResponsiveContainer width="100%" height={300}>
                <LineChart data={botTraffic?.timeline}>
                  <CartesianGrid strokeDasharray="3 3" />
                  <XAxis dataKey="date" />
                  <YAxis />
                  <Tooltip />
                  <Line 
                    type="monotone" 
                    dataKey="seo" 
                    stroke="#8884d8" 
                    name="SEO Bots"
                  />
                  <Line 
                    type="monotone" 
                    dataKey="llm" 
                    stroke="#82ca9d" 
                    name="LLM Bots"
                  />
                  <Line 
                    type="monotone" 
                    dataKey="blocked" 
                    stroke="#ff7c7c" 
                    name="Blocked"
                  />
                </LineChart>
              </ResponsiveContainer>
            </CardContent>
          </Card>

          <div className="grid gap-4 md:grid-cols-2">
            <Card>
              <CardHeader>
                <CardTitle>Top SEO Bots</CardTitle>
              </CardHeader>
              <CardContent>
                <div className="space-y-2">
                  {botTraffic?.topSeoBots.map((bot) => (
                    <div key={bot.name} className="flex items-center justify-between">
                      <span className="text-sm">{bot.name}</span>
                      <div className="flex items-center gap-2">
                        <span className="text-sm text-muted-foreground">
                          {bot.visits.toLocaleString()}
                        </span>
                        <Progress value={bot.percentage} className="w-20" />
                      </div>
                    </div>
                  ))}
                </div>
              </CardContent>
            </Card>

            <Card>
              <CardHeader>
                <CardTitle>Blocked Bots</CardTitle>
              </CardHeader>
              <CardContent>
                <div className="space-y-2">
                  {botTraffic?.blockedBots.map((bot) => (
                    <div key={bot.name} className="flex items-center justify-between">
                      <span className="text-sm">{bot.name}</span>
                      <Badge variant="destructive" className="text-xs">
                        {bot.attempts} attempts
                      </Badge>
                    </div>
                  ))}
                </div>
              </CardContent>
            </Card>
          </div>
        </TabsContent>

        <TabsContent value="crawl" className="space-y-4">
          <Card>
            <CardHeader>
              <CardTitle>Crawl Statistics</CardTitle>
            </CardHeader>
            <CardContent>
              <ResponsiveContainer width="100%" height={300}>
                <BarChart data={seoData?.crawlStats}>
                  <CartesianGrid strokeDasharray="3 3" />
                  <XAxis dataKey="pageType" />
                  <YAxis />
                  <Tooltip />
                  <Bar dataKey="crawled" fill="#8884d8" name="Crawled" />
                  <Bar dataKey="indexed" fill="#82ca9d" name="Indexed" />
                </BarChart>
              </ResponsiveContainer>
            </CardContent>
          </Card>
        </TabsContent>
      </Tabs>
    </div>
  );
}
```))
      {
        "error": {
          "code": "$inputRoot.errorType",
          "message": "$inputRoot.errorMessage",
          "requestId": "$context.requestId",
          "timestamp": "$context.requestTime"
        }
      }
    `,
  },
  
  // Validation error template
  validationError: {
    'application/json': `
      {
        "error": {
          "code": "VALIDATION_ERROR",
          "message": "Request validation failed",
          "details": $input.json('$.errors'),
          "requestId": "$context.requestId"
        }
      }
    `,
  },
};

// Request/Response models
export const apiModels = {
  // Error model
  errorModel: {
    type: 'object',
    required: ['error'],
    properties: {
      error: {
        type: 'object',
        required: ['code', 'message'],
        properties: {
          code: { type: 'string' },
          message: { type: 'string' },
          details: { type: 'object' },
          requestId: { type: 'string' },
        },
      },
    },
  },
  
  // Pagination model
  paginationModel: {
    type: 'object',
    required: ['data', 'meta'],
    properties: {
      data: {
        type: 'array',
        items: { type: 'object' },
      },
      meta: {
        type: 'object',
        required: ['total', 'page', 'limit', 'totalPages'],
        properties: {
          total: { type: 'integer' },
          page: { type: 'integer' },
          limit: { type: 'integer' },
          totalPages: { type: 'integer' },
        },
      },
    },
  },
};
```

## Production Readiness Checklist

### Pre-Launch Checklist
```typescript
// production-checklist.ts
export const productionReadinessChecklist = {
  // Performance
  performance: {
    'Core Web Vitals': {
      'LCP < 2.5s': false,
      'FID < 100ms': false,
      'CLS < 0.1': false,
      'TTFB < 800ms': false,
    },
    'Bundle Size': {
      'Main bundle < 250KB': false,
      'Vendor bundle < 500KB': false,
      'Code splitting implemented': false,
      'Tree shaking enabled': false,
    },
    'API Response Times': {
      'p50 < 200ms': false,
      'p95 < 500ms': false,
      'p99 < 1s': false,
      'Error rate < 0.5%': false,
    },
    'Caching': {
      'CDN configured': false,
      'Browser caching headers': false,
      'API response caching': false,
      'Redis cache implemented': false,
    },
  },
  
  // Security
  security: {
    'Authentication': {
      'JWT implementation secure': false,
      'Token refresh mechanism': false,
      'Session management': false,
      'Password policies enforced': false,
    },
    'Authorization': {
      'RBAC implemented': false,
      'Admin access controls': false,
      'API key management': false,
      'Rate limiting active': false,
    },
    'Data Protection': {
      'HTTPS everywhere': false,
      'CSP headers configured': false,
      'XSS protection': false,
      'SQL injection prevention': false,
    },
    'Compliance': {
      'GDPR compliance': false,
      'Data encryption at rest': false,
      'Data encryption in transit': false,
      'Audit logging enabled': false,
    },
  },
  
  // Infrastructure
  infrastructure: {
    'Deployment': {
      'CI/CD pipeline': false,
      'Blue-green deployment': false,
      'Rollback procedures': false,
      'Environment separation': false,
    },
    'Monitoring': {
      'Application monitoring': false,
      'Infrastructure monitoring': false,
      'Log aggregation': false,
      'Alerting configured': false,
    },
    'Scaling': {
      'Auto-scaling configured': false,
      'Load balancing': false,
      'Database connection pooling': false,
      'CDN distribution': false,
    },
    'Backup & Recovery': {
      'Database backups': false,
      'Disaster recovery plan': false,
      'RTO/RPO defined': false,
      'Backup testing': false,
    },
  },
  
  // Testing
  testing: {
    'Unit Tests': {
      'Coverage > 80%': false,
      'Critical paths tested': false,
      'Mocking implemented': false,
      'CI integration': false,
    },
    'Integration Tests': {
      'API integration tests': false,
      'Database integration tests': false,
      'Third-party service tests': false,
      'Error scenarios covered': false,
    },
    'E2E Tests': {
      'Critical user flows': false,
      'Cross-browser testing': false,
      'Mobile testing': false,
      'Performance testing': false,
    },
    'Security Tests': {
      'Penetration testing': false,
      'OWASP compliance': false,
      'Dependency scanning': false,
      'Secret scanning': false,
    },
  },
  
  // Documentation
  documentation: {
    'Technical Docs': {
      'API documentation': false,
      'Architecture diagrams': false,
      'Deployment guide': false,
      'Troubleshooting guide': false,
    },
    'User Docs': {
      'User manual': false,
      'Admin guide': false,
      'Developer portal docs': false,
      'FAQ section': false,
    },
    'Operations': {
      'Runbook created': false,
      'Incident response plan': false,
      'On-call procedures': false,
      'SLA documentation': false,
    },
  },
  
  // Features
  features: {
    'Core Features': {
      'Event management': false,
      'User authentication': false,
      'WhatsApp integration': false,
      'Payment processing': false,
    },
    'Admin Features': {
      'SuperAdmin portal': false,
      'Admin management': false,
      'Feature toggles': false,
      'Audit logs': false,
    },
    'Developer Features': {
      'Developer portal': false,
      'API key management': false,
      'Documentation site': false,
      'Usage analytics': false,
    },
    'SEO Features': {
      'Landing pages': false,
      'Microsites': false,
      'Bot management': false,
      'Sitemap generation': false,
    },
  },
};

// Validation script
export async function validateProductionReadiness(): Promise<{
  ready: boolean;
  issues: string[];
}> {
  const issues: string[] = [];
  
  // Check each category
  for (const [category, items] of Object.entries(productionReadinessChecklist)) {
    for (const [subcategory, checks] of Object.entries(items)) {
      for (const [check, status] of Object.entries(checks)) {
        if (!status) {
          issues.push(`${category} > ${subcategory} > ${check}`);
        }
      }
    }
  }
  
  return {
    ready: issues.length === 0,
    issues,
  };
}
```

### Production Deployment Script
```bash
#!/bin/bash
# deploy-production.sh

set -e

echo "🚀 Starting EventfulHQ Production Deployment"

# 1. Run tests
echo "📋 Running tests..."
npm run test:ci
npm run test:e2e

# 2. Build application
echo "🏗️ Building application..."
npm run build

# 3. Run security checks
echo "🔒 Running security checks..."
npm audit --production
npm run security:scan

# 4. Validate environment
echo "✅ Validating environment..."
node scripts/validate-env.js

# 5. Deploy infrastructure
echo "🏗️ Deploying infrastructure..."
terraform plan -out=tfplan
terraform apply tfplan

# 6. Deploy Lambda functions
echo "⚡ Deploying Lambda functions..."
serverless deploy --stage production

# 7. Deploy to Vercel
echo "▲ Deploying to Vercel..."
vercel --prod

# 8. Run smoke tests
echo "🔥 Running smoke tests..."
npm run test:smoke

# 9. Update monitoring
echo "📊 Updating monitoring..."
node scripts/update-monitoring.js

# 10. Notify team
echo "📢 Notifying team..."
node scripts/notify-deployment.js

echo "✅ Deployment completed successfully!"
```

### Monitoring Configuration
```typescript
// monitoring/datadog-config.ts
export const datadogConfig = {
  // RUM (Real User Monitoring)
  rum: {
    applicationId: process.env.DD_APPLICATION_ID,
    clientToken: process.env.DD_CLIENT_TOKEN,
    site: 'datadoghq.com',
    service: 'eventfulhq-frontend',
    env: process.env.NODE_ENV,
    version: process.env.APP_VERSION,
    sampleRate: 100,
    trackInteractions: true,
    defaultPrivacyLevel: 'mask-user-input',
  },
  
  // APM (Application Performance Monitoring)
  apm: {
    enabled: true,
    environment: process.env.NODE_ENV,
    service: 'eventfulhq-api',
    version: process.env.APP_VERSION,
    sampleRate: 1.0,
    tags: {
      team: 'platform',
      product: 'eventfulhq',
    },
  },
  
  // Custom metrics
  metrics: [
    {
      name: 'event.created',
      type: 'counter',
      tags: ['category', 'region'],
    },
    {
      name: 'api.response.time',
      type: 'histogram',
      tags: ['endpoint', 'method', 'status'],
    },
    {
      name: 'whatsapp.otp.sent',
      type: 'counter',
      tags: ['country', 'success'],
    },
    {
      name: 'admin.action',
      type: 'counter',
      tags: ['admin_type', 'action', 'resource'],
    },
  ],
  
  // Alerts
  alerts: [
    {
      name: 'High Error Rate',
      query: 'avg(last_5m):sum:trace.error{service:eventfulhq-api} > 0.05',
      message: 'Error rate is above 5% @slack-platform-alerts',
      tags: ['severity:high', 'team:platform'],
    },
    {
      name: 'Slow API Response',
      query: 'avg(last_5m):avg:trace.api.response.time{service:eventfulhq-api} > 1000',
      message: 'API response time is above 1s @slack-platform-alerts',
      tags: ['severity:medium', 'team:platform'],
    },
  ],
};
```

### Final Production Environment Variables
```bash
# .env.production
# API Configuration
NEXT_PUBLIC_API_URL=https://api.eventfulhq.com
NEXT_PUBLIC_WS_URL=wss://ws.eventfulhq.com
NEXT_PUBLIC_CDN_URL=https://cdn.eventfulhq.com

# Authentication
JWT_SECRET_ID=eventfulhq/jwt-secret
WHATSAPP_API_SECRET=eventfulhq/whatsapp-api
OTP_SALT=eventfulhq/otp-salt

# Database
DATABASE_URL=postgres://username:password@rds-endpoint:5432/eventfulhq
REDIS_URL=redis://elasticache-endpoint:6379

# AWS Resources
EVENTS_TABLE=eventfulhq-events-prod
USERS_TABLE=eventfulhq-users-prod
IMAGES_BUCKET=eventfulhq-images-prod
EVENT_BUS_NAME=eventfulhq-events-prod

# Monitoring
DD_APPLICATION_ID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
DD_CLIENT_TOKEN=pubxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
SENTRY_DSN=https://xxxxxx@sentry.io/xxxxxx

# Analytics
NEXT_PUBLIC_GA_ID=G-XXXXXXXXXX
NEXT_PUBLIC_ADSENSE_CLIENT_ID=ca-pub-xxxxxxxxxxxxxxxx

# Feature Flags
ENABLE_WHATSAPP_AUTH=true
ENABLE_RFP_MARKETPLACE=true
ENABLE_AI_RECOMMENDATIONS=true
ENABLE_DEVELOPER_PORTAL=true
ENABLE_SUPER_ADMIN=true

# Regional Domains
DOMAIN_GLOBAL=eventfulhq.com
DOMAIN_AMERICA=eventfulamerica.com
DOMAIN_ARABIA=eventfularabia.com
DOMAIN_INDIA=eventfulindia.com
DOMAIN_EUROPE=eventfuleurope.com
DOMAIN_AUSTRALIA=eventfulaustralia.com
```

---

## Conclusion

This comprehensive frontend architecture document serves as the complete guide for building and deploying the EventfulHQ platform. It covers all aspects from the initial MatDash theme enhancement to production deployment, including:

1. **Enhanced UI Components**: Migration from Flowbite to ShadCN/UI
2. **Multi-Language Support**: 9 languages with automatic detection
3. **Regional Routing**: 6 regional domains with geo-targeting
4. **SuperAdmin System**: 11 admin types with granular permissions
5. **Developer Portal**: Complete API documentation and management
6. **SEO-Optimized Landing Pages**: Dynamic city/country pages with bot management
7. **Microsite Generation**: User, team, and event microsites
8. **Performance Optimization**: Meeting all SLA requirements
9. **Security Implementation**: WhatsApp OTP, RBAC, and comprehensive security measures
10. **Production Readiness**: Complete deployment and monitoring setup

The architecture is designed to scale globally while maintaining excellent performance, security, and user experience across all regions and languages.

### Language Switcher Component
```typescript
// components/features/i18n/language-switcher.tsx
'use client';

import { useLocale } from 'next-intl';
import { useRouter, usePathname } from 'next/navigation';
import { useTransition } from 'react';
import {
  Select,
  SelectContent,
  SelectItem,
  SelectTrigger,
  SelectValue,
} from '@/components/ui/select';
import { Globe } from 'lucide-react';

const languages = [
  { code: 'en', name: 'English', flag: '🇺🇸', dir: 'ltr' },
  { code: 'zh', name: '中文', flag: '🇨🇳', dir: 'ltr' },
  { code: 'es', name: 'Español', flag: '🇪🇸', dir: 'ltr' },
  { code: 'pt', name: 'Português', flag: '🇧🇷', dir: 'ltr' },
  { code: 'fr', name: 'Français', flag: '🇫🇷', dir: 'ltr' },
  { code: 'ja', name: '日本語', flag: '🇯🇵', dir: 'ltr' },
  { code: 'ar', name: 'العربية', flag: '🇸🇦', dir: 'rtl' },
  { code: 'hi', name: 'हिन्दी', flag: '🇮🇳', dir: 'ltr' },
  { code: 'ru', name: 'Русский', flag: '🇷🇺', dir: 'ltr' },
];

export function LanguageSwitcher() {
  const locale = useLocale();
  const router = useRouter();
  const pathname = usePathname();
  const [isPending, startTransition] = useTransition();

  const handleLanguageChange = (newLocale: string) => {
    // Save preference
    document.cookie = `preferred-locale=${newLocale};max-age=31536000;path=/;samesite=lax${
      process.env.NODE_ENV === 'production' ? ';secure' : ''
    }`;

    // Update HTML dir attribute for RTL languages
    const newLang = languages.find(l => l.code === newLocale);
    document.documentElement.dir = newLang?.dir || 'ltr';

    // Navigate to new locale
    startTransition(() => {
      const segments = pathname.split('/');
      segments[1] = newLocale;
      router.push(segments.join('/'));
    });
  };

  const currentLanguage = languages.find(lang => lang.code === locale);

  return (
    <Select 
      value={locale} 
      onValueChange={handleLanguageChange}
      disabled={isPending}
    >
      <SelectTrigger className="w-[180px]">
        <Globe className="mr-2 h-4 w-4" />
        <SelectValue>
          {currentLanguage && (
            <span className="flex items-center">
              <span className="mr-2">{currentLanguage.flag}</span>
              {currentLanguage.name}
            </span>
          )}
        </SelectValue>
      </SelectTrigger>
      <SelectContent>
        {languages.map((language) => (
          <SelectItem key={language.code} value={language.code}>
            <span className="flex items-center">
              <span className="mr-2 text-lg">{language.flag}</span>
              <span>{language.name}</span>
            </span>
          </SelectItem>
        ))}
      </SelectContent>
    </Select>
  );
}
```

## State Management & Data Flow

### Zustand Store Architecture
```typescript
// stores/use-app-store.ts
import { create } from 'zustand';
import { devtools, persist } from 'zustand/middleware';
import { immer } from 'zustand/middleware/immer';

interface AppState {
  // User state
  user: User | null;
  isAuthenticated: boolean;
  permissions: Permission[];
  
  // UI state
  sidebarOpen: boolean;
  theme: 'light' | 'dark' | 'system';
  locale: string;
  
  // Feature flags
  features: Record<string, boolean>;
  
  // Actions
  setUser: (user: User | null) => void;
  toggleSidebar: () => void;
  setTheme: (theme: 'light' | 'dark' | 'system') => void;
  setLocale: (locale: string) => void;
  updateFeatureFlag: (flag: string, enabled: boolean) => void;
  reset: () => void;
}

export const useAppStore = create<AppState>()(
  devtools(
    persist(
      immer((set) => ({
        // Initial state
        user: null,
        isAuthenticated: false,
        permissions: [],
        sidebarOpen: true,
        theme: 'system',
        locale: 'en',
        features: {},
        
        // Actions
        setUser: (user) =>
          set((state) => {
            state.user = user;
            state.isAuthenticated = !!user;
            state.permissions = user?.permissions || [];
          }),
          
        toggleSidebar: () =>
          set((state) => {
            state.sidebarOpen = !state.sidebarOpen;
          }),
          
        setTheme: (theme) =>
          set((state) => {
            state.theme = theme;
          }),
          
        setLocale: (locale) =>
          set((state) => {
            state.locale = locale;
          }),
          
        updateFeatureFlag: (flag, enabled) =>
          set((state) => {
            state.features[flag] = enabled;
          }),
          
        reset: () =>
          set((state) => {
            state.user = null;
            state.isAuthenticated = false;
            state.permissions = [];
            state.features = {};
          }),
      })),
      {
        name: 'app-storage',
        partialize: (state) => ({
          theme: state.theme,
          locale: state.locale,
          sidebarOpen: state.sidebarOpen,
        }),
      }
    )
  )
);

// Separate stores for different domains
export const useEventStore = create<EventState>()(
  devtools(
    immer((set) => ({
      // Event specific state and actions
    }))
  )
);

export const useAdminStore = create<AdminState>()(
  devtools(
    immer((set) => ({
      // Admin specific state and actions
    }))
  )
);
```

### React Query Configuration
```typescript
// lib/react-query.ts
import {
  QueryClient,
  QueryClientProvider,
  QueryCache,
  MutationCache,
} from '@tanstack/react-query';
import { toast } from 'sonner';

// Create a custom query client with global error handling
export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 5 * 60 * 1000, // 5 minutes
      gcTime: 10 * 60 * 1000, // 10 minutes (formerly cacheTime)
      retry: (failureCount, error: any) => {
        // Don't retry on 4xx errors
        if (error?.response?.status >= 400 && error?.response?.status < 500) {
          return false;
        }
        return failureCount < 3;
      },
      refetchOnWindowFocus: false,
    },
    mutations: {
      retry: false,
    },
  },
  queryCache: new QueryCache({
    onError: (error: any) => {
      // Global query error handling
      if (error?.response?.status === 401) {
        // Handle unauthorized
        window.location.href = '/login';
      } else if (error?.response?.status === 403) {
        toast.error('You do not have permission to perform this action');
      } else if (error?.response?.status >= 500) {
        toast.error('Server error. Please try again later.');
      }
    },
  }),
  mutationCache: new MutationCache({
    onError: (error: any) => {
      // Global mutation error handling
      const message = error?.response?.data?.message || 'An error occurred';
      toast.error(message);
    },
  }),
});

// Custom hooks for common queries
export function useUser() {
  return useQuery({
    queryKey: ['user'],
    queryFn: userService.getCurrentUser,
    staleTime: Infinity, // User data rarely changes
  });
}

export function useEvents(filters?: EventFilters) {
  return useQuery({
    queryKey: ['events', filters],
    queryFn: () => eventService.getEvents(filters),
  });
}

export function useEvent(eventId: string) {
  return useQuery({
    queryKey: ['events', eventId],
    queryFn: () => eventService.getEvent(eventId),
    enabled: !!eventId,
  });
}

// Optimistic updates example
export function useUpdateEvent() {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: eventService.updateEvent,
    onMutate: async (updatedEvent) => {
      // Cancel outgoing refetches
      await queryClient.cancelQueries({ queryKey: ['events', updatedEvent.id] });
      
      // Snapshot previous value
      const previousEvent = queryClient.getQueryData(['events', updatedEvent.id]);
      
      // Optimistically update
      queryClient.setQueryData(['events', updatedEvent.id], updatedEvent);
      
      // Return context with snapshot
      return { previousEvent };
    },
    onError: (err, updatedEvent, context) => {
      // Rollback on error
      if (context?.previousEvent) {
        queryClient.setQueryData(
          ['events', updatedEvent.id],
          context.previousEvent
        );
      }
    },
    onSettled: (data, error, variables) => {
      // Always refetch after error or success
      queryClient.invalidateQueries({ queryKey: ['events', variables.id] });
    },
  });
}
```

### Server State Synchronization
```typescript
// hooks/use-realtime-sync.ts
import { useEffect } from 'react';
import { useQueryClient } from '@tanstack/react-query';
import { io, Socket } from 'socket.io-client';

let socket: Socket;

export function useRealtimeSync() {
  const queryClient = useQueryClient();
  
  useEffect(() => {
    // Initialize WebSocket connection
    socket = io(process.env.NEXT_PUBLIC_WS_URL!, {
      auth: {
        token: localStorage.getItem('auth-token'),
      },
    });
    
    // Listen for real-time updates
    socket.on('event:updated', (data) => {
      // Update React Query cache
      queryClient.setQueryData(['events', data.id], data);
      
      // Invalidate list queries
      queryClient.invalidateQueries({ 
        queryKey: ['events'],
        exact: false,
      });
    });
    
    socket.on('event:deleted', (data) => {
      // Remove from cache
      queryClient.removeQueries({ queryKey: ['events', data.id] });
      
      // Invalidate list queries
      queryClient.invalidateQueries({ 
        queryKey: ['events'],
        exact: false,
      });
    });
    
    socket.on('notification', (notification) => {
      // Handle real-time notifications
      toast.info(notification.message);
    });
    
    return () => {
      socket.disconnect();
    };
  }, [queryClient]);
  
  return socket;
}

// Hook for subscribing to specific channels
export function useRealtimeChannel(channel: string, handler: (data: any) => void) {
  const socket = useRealtimeSync();
  
  useEffect(() => {
    if (!socket) return;
    
    socket.on(channel, handler);
    
    return () => {
      socket.off(channel, handler);
    };
  }, [socket, channel, handler]);
}
```

## Form Handling with Yup & Zod

### Yup Schema Examples
```typescript
// validations/event.schema.ts
import * as yup from 'yup';
import { isValidPhoneNumber } from 'libphonenumber-js';

// Custom validators
yup.addMethod(yup.string, 'phone', function(message = 'Invalid phone number') {
  return this.test('phone', message, function(value) {
    if (!value) return true;
    return isValidPhoneNumber(value);
  });
});

// Event creation schema
export const createEventSchema = yup.object({
  title: yup
    .string()
    .required('Event title is required')
    .min(5, 'Title must be at least 5 characters')
    .max(100, 'Title must not exceed 100 characters'),
    
  description: yup
    .string()
    .required('Description is required')
    .min(20, 'Description must be at least 20 characters')
    .max(5000, 'Description must not exceed 5000 characters'),
    
  date: yup
    .date()
    .required('Event date is required')
    .min(new Date(), 'Event date must be in the future'),
    
  endDate: yup
    .date()
    .nullable()
    .when('date', (date, schema) => {
      return date ? schema.min(date, 'End date must be after start date') : schema;
    }),
    
  venue: yup.object({
    name: yup.string().required('Venue name is required'),
    address: yup.string().required('Venue address is required'),
    city: yup.string().required('City is required'),
    country: yup.string().required('Country is required'),
    coordinates: yup.object({
      lat: yup.number().required().min(-90).max(90),
      lng: yup.number().required().min(-180).max(180),
    }).required('Venue coordinates are required'),
  }).required('Venue information is required'),
  
  capacity: yup
    .number()
    .required('Capacity is required')
    .positive('Capacity must be positive')
    .integer('Capacity must be a whole number')
    .max(1000000, 'Capacity seems unrealistic'),
    
  price: yup
    .number()
    .required('Price is required')
    .min(0, 'Price cannot be negative')
    .max(100000, 'Price seems too high'),
    
  currency: yup
    .string()
    .required('Currency is required')
    .oneOf(['USD', 'EUR', 'GBP', 'INR', 'AED', 'SAR']),
    
  categories: yup
    .array()
    .of(yup.string())
    .min(1, 'Select at least one category')
    .max(5, 'Maximum 5 categories allowed'),
    
  images: yup
    .array()
    .of(
      yup.object({
        url: yup.string().url('Invalid image URL'),
        alt: yup.string(),
      })
    )
    .min(1, 'At least one image is required')
    .max(10, 'Maximum 10 images allowed'),
    
  whatsappRSVP: yup.object({
    enabled: yup.boolean(),
    phoneNumber: yup.string().when('enabled', {
      is: true,
      then: (schema) => schema.required('WhatsApp number is required').phone(),
    }),
    message: yup.string().when('enabled', {
      is: true,
      then: (schema) => schema.required('RSVP message is required'),
    }),
  }),
  
  isPublished: yup.boolean(),
  
  tickets: yup.array().of(
    yup.object({
      name: yup.string().required('Ticket name is required'),
      price: yup.number().required().min(0),
      quantity: yup.number().required().positive().integer(),
      description: yup.string(),
    })
  ),
});

// User registration schema
export const userRegistrationSchema = yup.object({
  name: yup
    .string()
    .required('Name is required')
    .matches(/^[a-zA-Z\s]+$/, 'Name can only contain letters and spaces'),
    
  email: yup
    .string()
    .required('Email is required')
    .email('Invalid email address'),
    
  phoneNumber: yup
    .string()
    .required('Phone number is required')
    .phone('Invalid phone number'),
    
  password: yup
    .string()
    .required('Password is required')
    .min(8, 'Password must be at least 8 characters')
    .matches(
      /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]/,
      'Password must contain uppercase, lowercase, number and special character'
    ),
    
  confirmPassword: yup
    .string()
    .required('Please confirm your password')
    .oneOf([yup.ref('password')], 'Passwords must match'),
    
  acceptTerms: yup
    .boolean()
    .required('You must accept the terms and conditions')
    .oneOf([true], 'You must accept the terms and conditions'),
});

// Dynamic field validation
export const createDynamicSchema = (fields: FormField[]) => {
  const shape: Record<string, any> = {};
  
  fields.forEach((field) => {
    let validator = yup.string();
    
    if (field.required) {
      validator = validator.required(`${field.label} is required`);
    }
    
    if (field.type === 'email') {
      validator = validator.email('Invalid email');
    }
    
    if (field.type === 'number') {
      validator = yup.number();
      if (field.min !== undefined) {
        validator = validator.min(field.min);
      }
      if (field.max !== undefined) {
        validator = validator.max(field.max);
      }
    }
    
    if (field.pattern) {
      validator = validator.matches(field.pattern, field.patternMessage);
    }
    
    shape[field.name] = validator;
  });
  
  return yup.object(shape);
};
```

### Zod Schema Examples (Secondary)
```typescript
// validations/admin.schema.ts
import { z } from 'zod';

// Admin creation schema using Zod
export const createAdminSchema = z.object({
  name: z.string().min(2, 'Name must be at least 2 characters'),
  email: z.string().email('Invalid email address'),
  phoneNumber: z.string().regex(/^\+?[1-9]\d{1,14}$/, 'Invalid phone number'),
  type: z.enum([
    'SUPER_ADMIN',
    'PLATFORM_ADMIN',
    'REGIONAL_ADMIN',
    'COUNTRY_ADMIN',
    'CITY_ADMIN',
    'CATEGORY_ADMIN',
    'CONTENT_ADMIN',
    'SUPPORT_ADMIN',
    'FINANCE_ADMIN',
    'MARKETING_ADMIN',
    'DEVELOPER_ADMIN',
  ]),
  permissions: z.array(z.object({
    resource: z.string(),
    actions: z.array(z.enum(['READ', 'WRITE', 'UPDATE', 'DELETE', 'PUBLISH', 'EXPORT', 'CONFIGURE'])),
    conditions: z.record(z.any()).optional(),
  })),
  geographyAccess: z.array(z.object({
    type: z.enum(['GLOBAL', 'REGION', 'COUNTRY', 'CITY']),
    identifier: z.string(),
    permissions: z.array(z.string()),
  })),
  featureAccess: z.array(z.object({
    microservice: z.string(),
    permissions: z.array(z.string()),
    limits: z.record(z.number()).optional(),
  })),
});

// API request validation
export const apiRequestSchema = z.object({
  method: z.enum(['GET', 'POST', 'PUT', 'DELETE', 'PATCH']),
  endpoint: z.string().url(),
  headers: z.record(z.string()).optional(),
  body: z.any().optional(),
  queryParams: z.record(z.string()).optional(),
});

// Type inference from Zod schemas
export type CreateAdminInput = z.infer<typeof createAdminSchema>;
export type APIRequest = z.infer<typeof apiRequestSchema>;
```

### React Hook Form Integration
```typescript
// components/features/events/create-event-form.tsx
'use client';

import { useForm } from 'react-hook-form';
import { yupResolver } from '@hookform/resolvers/yup';
import { createEventSchema } from '@/validations/event.schema';
import { Form, FormControl, FormDescription, FormField, FormItem, FormLabel, FormMessage } from '@/components/ui/form';
import { Input } from '@/components/ui/input';
import { Textarea } from '@/components/ui/textarea';
import { Button } from '@/components/ui/button';
import { Calendar } from '@/components/ui/calendar';
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from '@/components/ui/select';
import { Switch } from '@/components/ui/switch';
import { toast } from 'sonner';

type FormData = yup.InferType<typeof createEventSchema>;

export function CreateEventForm() {
  const form = useForm<FormData>({
    resolver: yupResolver(createEventSchema),
    defaultValues: {
      title: '',
      description: '',
      date: undefined,
      endDate: undefined,
      venue: {
        name: '',
        address: '',
        city: '',
        country: '',
        coordinates: { lat: 0, lng: 0 },
      },
      capacity: 100,
      price: 0,
      currency: 'USD',
      categories: [],
      images: [],
      whatsappRSVP: {
        enabled: false,
        phoneNumber: '',
        message: '',
      },
      isPublished: false,
      tickets: [],
    },
  });

  const { watch, setValue } = form;
  const whatsappEnabled = watch('whatsappRSVP.enabled');

  const onSubmit = async (data: FormData) => {
    try {
      const response = await eventService.createEvent(data);
      toast.success('Event created successfully!');
      router.push(`/events/${response.id}`);
    } catch (error) {
      toast.error('Failed to create event');
    }
  };

  return (
    <Form {...form}>
      <form onSubmit={form.handleSubmit(onSubmit)} className="space-y-8">
        <FormField
          control={form.control}
          name="title"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Event Title</FormLabel>
              <FormControl>
                <Input placeholder="Summer Music Festival" {...field} />
              </FormControl>
              <FormDescription>
                Choose a catchy title for your event
              </FormDescription>
              <FormMessage />
            </FormItem>
          )}
        />

        <FormField
          control={form.control}
          name="description"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Description</FormLabel>
              <FormControl>
                <Textarea
                  placeholder="Describe your event..."
                  className="min-h-[120px]"
                  {...field}
                />
              </FormControl>
              <FormMessage />
            </FormItem>
          )}
        />

        <div className="grid gap-4 md:grid-cols-2">
          <FormField
            control={form.control}
            name="date"
            render={({ field }) => (
              <FormItem className="flex flex-col">
                <FormLabel>Start Date</FormLabel>
                <Calendar
                  mode="single"
                  selected={field.value}
                  onSelect={field.onChange}
                  disabled={(date) => date < new Date()}
                  initialFocus
                />
                <FormMessage />
              </FormItem>
            )}
          />

          <FormField
            control={form.control}
            name="endDate"
            render={({ field }) => (
              <FormItem className="flex flex-col">
                <FormLabel>End Date (Optional)</FormLabel>
                <Calendar
                  mode="single"
                  selected={field.value}
                  onSelect={field.onChange}
                  disabled={(date) => date < watch('date')}
                />
                <FormMessage />
              </FormItem>
            )}
          />
        </div>

        <div className="space-y-4">
          <h3 className="text-lg font-semibold">Venue Information</h3>
          
          <FormField
            control={form.control}
            name="venue.name"
            render={({ field }) => (
              <FormItem>
                <FormLabel>Venue Name</FormLabel>
                <FormControl>
                  <Input placeholder="Madison Square Garden" {...field} />
                </FormControl>
                <FormMessage />
              </FormItem>
            )}
          />

          <FormField
            control={form.control}
            name="venue.address"
            render={({ field }) => (
              <FormItem>
                <FormLabel>Address</FormLabel>
                <FormControl>
                  <Input placeholder="4 Pennsylvania Plaza" {...field} />
                </FormControl>
                <FormMessage />
              </FormItem>
            )}
          />

          <div className="grid gap-4 md:grid-cols-2">
            <FormField
              control={form.control}
              name="venue.city"
              render={({ field }) => (
                <FormItem>
                  <FormLabel>City</FormLabel>
                  <FormControl>
                    <Input placeholder="New York" {...field} />
                  </FormControl>
                  <FormMessage />
                </FormItem>
              )}
            />

            <FormField
              control={form.control}
              name="venue.country"
              render={({ field }) => (
                <FormItem>
                  <FormLabel>Country</FormLabel>
                  <Select onValueChange={field.onChange} defaultValue={field.value}>
                    <FormControl>
                      <SelectTrigger>
                        <SelectValue placeholder="Select country" />
                      </SelectTrigger>
                    </FormControl>
                    <SelectContent>
                      <SelectItem value="US">United States</SelectItem>
                      <SelectItem value="GB">United Kingdom</SelectItem>
                      <SelectItem value="IN">India</SelectItem>
                      <SelectItem value="AE">UAE</SelectItem>
                      {/* Add more countries */}
                    </SelectContent>
                  </Select>
                  <FormMessage />
                </FormItem>
              )}
            />
          </div>
        </div>

        <div className="grid gap-4 md:grid-cols-3">
          <FormField
            control={form.control}
            name="capacity"
            render={({ field }) => (
              <FormItem>
                <FormLabel>Capacity</FormLabel>
                <FormControl>
                  <Input
                    type="number"
                    placeholder="500"
                    {...field}
                    onChange={(e) => field.onChange(parseInt(e.target.value))}
                  />
                </FormControl>
                <FormMessage />
              </FormItem>
            )}
          />

          <FormField
            control={form.control}
            name="price"
            render={({ field }) => (
              <FormItem>
                <FormLabel>Price</FormLabel>
                <FormControl>
                  <Input
                    type="number"
                    placeholder="0"
                    {...field}
                    onChange={(e) => field.onChange(parseFloat(e.target.value))}
                  />
                </FormControl>
                <FormMessage />
              </FormItem>
            )}
          />

          <FormField
            control={form.control}
            name="currency"
            render={({ field }) => (
              <FormItem>
                <FormLabel>Currency</FormLabel>
                <Select onValueChange={field.onChange} defaultValue={field.value}>
                  <FormControl>
                    <SelectTrigger>
                      <SelectValue />
                    </SelectTrigger>
                  </FormControl>
                  <SelectContent>
                    <SelectItem value="USD">USD</SelectItem>
                    <SelectItem value="EUR">EUR</SelectItem>
                    <SelectItem value="GBP">GBP</SelectItem>
                    <SelectItem value="INR">INR</SelectItem>
                    <SelectItem value="AED">AED</SelectItem>
                    <SelectItem value="SAR">SAR</SelectItem>
                  </SelectContent>
                </Select>
                <FormMessage />
              </FormItem>
            )}
          />
        </div>

        <div className="space-y-4">
          <FormField
            control={form.control}
            name="whatsappRSVP.enabled"
            render={({ field }) => (
              <FormItem className="flex flex-row items-center justify-between rounded-lg border p-4">
                <div className="space-y-0.5">
                  <FormLabel className="text-base">
                    WhatsApp RSVP
                  </FormLabel>
                  <FormDescription>
                    Allow attendees to RSVP via WhatsApp
                  </FormDescription>
                </div>
                <FormControl>
                  <Switch
                    checked={field.value}
                    onCheckedChange={field.onChange}
                  />
                </FormControl>
              </FormItem>
            )}
          />

          {whatsappEnabled && (
            <>
              <FormField
                control={form.control}
                name="whatsappRSVP.phoneNumber"
                render={({ field }) => (
                  <FormItem>
                    <FormLabel>WhatsApp Number</FormLabel>
                    <FormControl>
                      <Input placeholder="+1234567890" {...field} />
                    </FormControl>
                    <FormDescription>
                      Include country code
                    </FormDescription>
                    <FormMessage />
                  </FormItem>
                )}
              />

              <FormField
                control={form.control}
                name="whatsappRSVP.message"
                render={({ field }) => (
                  <FormItem>
                    <FormLabel>RSVP Message Template</FormLabel>
                    <FormControl>
                      <Textarea
                        placeholder="Hi! I'd like to RSVP for {event_name}"
                        {...field}
                      />
                    </FormControl>
                    <FormDescription>
                      Use {'{event_name}'} as placeholder
                    </FormDescription>
                    <FormMessage />
                  </FormItem>
                )}
              />
            </>
          )}
        </div>

        <div className="flex justify-end gap-4">
          <Button variant="outline" type="button">
            Save as Draft
          </Button>
          <Button type="submit">
            Create Event
          </Button>
        </div>
      </form>
    </Form>
  );
}
```

## Storybook Integration

### Storybook Configuration
```typescript
// .storybook/main.ts
import type { StorybookConfig } from '@storybook/nextjs';

const config: StorybookConfig = {
  stories: [
    '../stories/**/*.stories.@(js|jsx|ts|tsx|mdx)',
    '../src/**/*.stories.@(js|jsx|ts|tsx|mdx)',
  ],
  addons: [
    '@storybook/addon-essentials',
    '@storybook/addon-interactions',
    '@storybook/addon-links',
    '@storybook/addon-a11y',
    '@storybook/addon-viewport',
    '@storybook/addon-docs',
    '@storybook/addon-controls',
    'storybook-addon-next-router',
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
```

### Storybook Preview Configuration
```typescript
// .storybook/preview.tsx
import React from 'react';
import type { Preview } from '@storybook/react';
import { Inter } from 'next/font/google';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { NextIntlClientProvider } from 'next-intl';
import { ThemeProvider } from '@/components/theme-provider';
import '../src/styles/globals.css';

const inter = Inter({ subsets: ['latin'] });

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      retry: false,
      refetchOnWindowFocus: false,
    },
  },
});

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
    viewport: {
      viewports: {
        mobile: {
          name: 'Mobile',
          styles: {
            width: '375px',
            height: '667px',
          },
        },
        tablet: {
          name: 'Tablet',
          styles: {
            width: '768px',
            height: '1024px',
          },
        },
        desktop: {
          name: 'Desktop',
          styles: {
            width: '1440px',
            height: '900px',
          },
        },
      },
    },
  },
  decorators: [
    (Story) => (
      <QueryClientProvider client={queryClient}>
        <NextIntlClientProvider locale="en" messages={{}}>
          <ThemeProvider
            attribute="class"
            defaultTheme="system"
            enableSystem
            disableTransitionOnChange
          >
            <div className={inter.className}>
              <Story />
            </div>
          </ThemeProvider>
        </NextIntlClientProvider>
      </QueryClientProvider>
    ),
  ],
  globalTypes: {
    locale: {
      name: 'Locale',
      description: 'Internationalization locale',
      defaultValue: 'en',
      toolbar: {
        icon: 'globe',
        items: [
          { value: 'en', title: 'English' },
          { value: 'es', title: 'Spanish' },
          { value: 'fr', title: 'French' },
          { value: 'ar', title: 'Arabic' },
          { value: 'zh', title: 'Chinese' },
        ],
        showName: true,
      },
    },
    theme: {
      name: 'Theme',
      description: 'Theme for components',
      defaultValue: 'light',
      toolbar: {
        icon: 'circlehollow',
        items: ['light', 'dark', 'system'],
        showName: true,
      },
    },
  },
};

export default preview;
```

### Component Story Examples
```typescript
// stories/components/ui/button.stories.tsx
import type { Meta, StoryObj } from '@storybook/react';
import { Button } from '@/components/ui/button';
import { Mail, Loader2 } from 'lucide-react';

const meta = {
  title: 'UI/Button',
  component: Button,
  parameters: {
    layout: 'centered',
    docs: {
      description: {
        component: 'A flexible button component with multiple variants and sizes.',
      },
    },
  },
  tags: ['autodocs'],
  argTypes: {
    variant: {
      control: { type: 'select' },
      options: ['default', 'destructive', 'outline', 'secondary', 'ghost', 'link'],
      description: 'Button variant style',
    },
    size: {
      control: { type: 'select' },
      options: ['default', 'sm', 'lg', 'icon'],
      description: 'Button size',
    },
    disabled: {
      control: 'boolean',
      description: 'Disabled state',
    },
    asChild: {
      control: 'boolean',
      description: 'Render as child component',
    },
  },
} satisfies Meta<typeof Button>;

export default meta;
type Story = StoryObj<typeof meta>;

export const Default: Story = {
  args: {
    children: 'Button',
  },
};

export const Variants: Story = {
  render: () => (
    <div className="flex flex-wrap gap-4">
      <Button variant="default">Default</Button>
      <Button variant="destructive">Destructive</Button>
      <Button variant="outline">Outline</Button>
      <Button variant="secondary">Secondary</Button>
      <Button variant="ghost">Ghost</Button>
      <Button variant="link">Link</Button>
    </div>
  ),
};

export const Sizes: Story = {
  render: () => (
    <div className="flex items-center gap-4">
      <Button size="sm">Small</Button>
      <Button size="default">Default</Button>
      <Button size="lg">Large</Button>
      <Button size="icon">
        <Mail className="h-4 w-4" />
      </Button>
    </div>
  ),
};

export const WithIcon: Story = {
  render: () => (
    <div className="flex gap-4">
      <Button>
        <Mail className="mr-2 h-4 w-4" />
        Login with Email
      </Button>
      <Button variant="outline">
        <Mail className="mr-2 h-4 w-4" />
        Contact Us
      </Button>
    </div>
  ),
};

export const Loading: Story = {
  render: () => (
    <div className="flex gap-4">
      <Button disabled>
        <Loader2 className="mr-2 h-4 w-4 animate-spin" />
        Please wait
      </Button>
      <Button variant="outline" disabled>
        <Loader2 className="mr-2 h-4 w-4 animate-spin" />
        Loading...
      </Button>
    </div>
  ),
};

export const Disabled: Story = {
  render: () => (
    <div className="flex gap-4">
      <Button disabled>Disabled Default</Button>
      <Button variant="outline" disabled>
        Disabled Outline
      </Button>
      <Button variant="secondary" disabled>
        Disabled Secondary
      </Button>
    </div>
  ),
};

export const AsChild: Story = {
  render: () => (
    <Button asChild>
      <a href="https://eventfulhq.com" target="_blank" rel="noopener noreferrer">
        Open EventfulHQ
      </a>
    </Button>
  ),
};
```

### Feature Component Story
```typescript
// stories/features/events/event-card.stories.tsx
import type { Meta, StoryObj } from '@storybook/react';
import { EventCard } from '@/components/features/events/event-card';
import { mockEvent } from '@/test/mocks/event';

const meta = {
  title: 'Features/Events/EventCard',
  component: EventCard,
  parameters: {
    layout: 'padded',
  },
  tags: ['autodocs'],
  decorators: [
    (Story) => (
      <div className="max-w-md">
        <Story />
      </div>
    ),
  ],
} satisfies Meta<typeof EventCard>;

export default meta;
type Story = StoryObj<typeof meta>;

export const Default: Story = {
  args: {
    event: mockEvent,
  },
};

export const WithWhatsAppRSVP: Story = {
  args: {
    event: {
      ...mockEvent,
      whatsappRSVP: {
        enabled: true,
        phoneNumber: '+1234567890',
        message: 'Hi! I would like to RSVP for {event_name}',
      },
    },
  },
};

export const SoldOut: Story = {
  args: {
    event: {
      ...mockEvent,
      isSoldOut: true,
    },
  },
};

export const FreeEvent: Story = {
  args: {
    event: {
      ...mockEvent,
      price: 0,
    },
  },
};

export const Loading: Story = {
  args: {
    event: mockEvent,
    isLoading: true,
  },
};

export const Interactive: Story = {
  args: {
    event: mockEvent,
    onSelect: (event) => {
      console.log('Selected event:', event);
    },
    onShare: (event) => {
      console.log('Share event:', event);
    },
  },
  play: async ({ canvasElement, step }) => {
    const canvas = within(canvasElement);
    
    await step('Hover over card', async () => {
      await userEvent.hover(canvas.getByRole('article'));
    });
    
    await step('Click share button', async () => {
      await userEvent.click(canvas.getByRole('button', { name: /share/i }));
    });
  },
};
```

## API Integration & Code Generation

### OpenAPI Code Generation Setup
```typescript
// orval.config.ts
import { defineConfig } from 'orval';

export default defineConfig({
  eventfulhq: {
    input: {
      target: './openapi/eventfulhq.yaml',
      validation: true,
    },
    output: {
      mode: 'tags-split',
      target: './src/services/generated',
      schemas: './src/types/generated',
      client: 'react-query',
      httpClient: 'axios',
      override: {
        mutator: {
          path: './src/lib/api/client.ts',
          name: 'apiClient',
        },
        query: {
          useQuery: true,
          useInfinite: true,
          useInfiniteQueryParam: 'cursor',
          options: {
            staleTime: 5 * 60 * 1000,
          },
        },
        operations: {
          listEvents: {
            query: {
              useInfinite: true,
              useInfiniteQueryParam: 'page',
            },
          },
        },
      },
    },
    hooks: {
      afterAllFilesWrite: 'prettier --write',
    },
  },
  // Additional services
  whatsapp: {
    input: {
      target: './openapi/whatsapp-service.yaml',
    },
    output: {
      target: './src/services/generated/whatsapp',
      client: 'react-query',
    },
  },
  admin: {
    input: {
      target: './openapi/admin-service.yaml',
    },
    output: {
      target: './src/services/generated/admin',
      client: 'react-query',
    },
  },
});
```

### API Client Configuration
```typescript
// lib/api/client.ts
import axios, { AxiosError, AxiosRequestConfig } from 'axios';
import { toast } from 'sonner';

const BASE_URL = process.env.NEXT_PUBLIC_API_URL || 'https://api.eventfulhq.com';

// Create axios instance
export const apiClient = axios.create({
  baseURL: BASE_URL,
  timeout: 30000,
  headers: {
    'Content-Type': 'application/json',
  },
});

// Request interceptor
apiClient.interceptors.request.use(
  (config) => {
    // Add auth token
    const token = localStorage.getItem('auth-token');
    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }
    
    // Add request ID for tracing
    config.headers['X-Request-ID'] = generateRequestId();
    
    // Add locale
    const locale = localStorage.getItem('locale') || 'en';
    config.headers['Accept-Language'] = locale;
    
    // Add region
    const region = localStorage.getItem('preferred-region') || 'global';
    config.headers['X-Region'] = region;
    
    return config;
  },
  (error) => {
    return Promise.reject(error);
  }
);

// Response interceptor
apiClient.interceptors.response.use(
  (response) => {
    // Log successful responses in development
    if (process.env.NODE_ENV === 'development') {
      console.log('API Response:', {
        url: response.config.url,
        status: response.status,
        data: response.data,
      });
    }
    
    return response;
  },
  async (error: AxiosError) => {
    const originalRequest = error.config as AxiosRequestConfig & { _retry?: boolean };
    
    // Handle 401 - Token expired
    if (error.response?.status === 401 && !originalRequest._retry) {
      originalRequest._retry = true;
      
      try {
        const refreshToken = localStorage.getItem('refresh-token');
        const response = await axios.post(`${BASE_URL}/auth/refresh`, {
          refreshToken,
        });
        
        const { accessToken, refreshToken: newRefreshToken } = response.data;
        localStorage.setItem('auth-token', accessToken);
        localStorage.setItem('refresh-token', newRefreshToken);
        
        // Retry original request
        return apiClient(originalRequest);
      } catch (refreshError) {
        // Refresh failed, redirect to login
        localStorage.clear();
        window.location.href = '/login';
        return Promise.reject(refreshError);
      }
    }
    
    // Handle other errors
    handleApiError(error);
    
    return Promise.reject(error);
  }
);

// Error handler
function handleApiError(error: AxiosError<any>) {
  if (!error.response) {
    // Network error
    toast.error('Network error. Please check your connection.');
    return;
  }
  
  const { status, data } = error.response;
  
  switch (status) {
    case 400:
      toast.error(data.message || 'Invalid request');
      break;
    case 403:
      toast.error('You do not have permission to perform this action');
      break;
    case 404:
      toast.error('Resource not found');
      break;
    case 429:
      toast.error('Too many requests. Please try again later.');
      break;
    case 500:
      toast.error('Server error. Please try again later.');
      break;
    default:
      toast.error(data.message || 'An error occurred');
  }
}

// Custom API client wrapper for mutations
export async function apiMutator<T = any>(
  config: AxiosRequestConfig
): Promise<T> {
  const response = await apiClient(config);
  return response.data;
}

// Generate unique request ID
function generateRequestId(): string {
  return `${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;
}

// Typed API endpoints
export const API_ENDPOINTS = {
  // Auth
  auth: {
    login: '/auth/login',
    register: '/auth/register',
    refresh: '/auth/refresh',
    logout: '/auth/logout',
    whatsappOtp: '/auth/whatsapp-otp',
    verifyOtp: '/auth/verify-otp',
  },
  
  // Events
  events: {
    list: '/events',
    detail: (id: string) => `/events/${id}`,
    create: '/events',
    update: (id: string) => `/events/${id}`,
    delete: (id: string) => `/events/${id}`,
    publish: (id: string) => `/events/${id}/publish`,
    attendees: (id: string) => `/events/${id}/attendees`,
  },
  
  // RFP
  rfp: {
    list: '/rfp',
    create: '/rfp',
    detail: (id: string) => `/rfp/${id}`,
    proposals: (id: string) => `/rfp/${id}/proposals`,
    submitProposal: (id: string) => `/rfp/${id}/proposals`,
  },
  
  // Admin
  admin: {
    admins: '/admin/admins',
    features: '/admin/features',
    permissions: '/admin/permissions',
    geography: '/admin/geography',
    audit: '/admin/audit-logs',
  },
  
  // Developer
  developer: {
    apiKeys: '/developer/api-keys',
    usage: '/developer/usage',
    webhooks: '/developer/webhooks',
    docs: '/developer/docs',
  },
} as const;
```

### Service Layer Architecture
```typescript
// services/base-service.ts
import { apiClient } from '@/lib/api/client';
import { AxiosRequestConfig } from 'axios';

export interface PaginatedResponse<T> {
  data: T[];
  meta: {
    total: number;
    page: number;
    limit: number;
    totalPages: number;
  };
}

export interface QueryParams {
  page?: number;
  limit?: number;
  sort?: string;
  order?: 'asc' | 'desc';
  search?: string;
  filters?: Record<string, any>;
}

export abstract class BaseService {
  protected endpoint: string;

  constructor(endpoint: string) {
    this.endpoint = endpoint;
  }

  protected async get<T>(
    path: string = '',
    config?: AxiosRequestConfig
  ): Promise<T> {
    const response = await apiClient.get<T>(`${this.endpoint}${path}`, config);
    return response.data;
  }

  protected async post<T, D = any>(
    path: string = '',
    data?: D,
    config?: AxiosRequestConfig
  ): Promise<T> {
    const response = await apiClient.post<T>(`${this.endpoint}${path}`, data, config);
    return response.data;
  }

  protected async put<T, D = any>(
    path: string = '',
    data?: D,
    config?: AxiosRequestConfig
  ): Promise<T> {
    const response = await apiClient.put<T>(`${this.endpoint}${path}`, data, config);
    return response.data;
  }

  protected async patch<T, D = any>(
    path: string = '',
    data?: D,
    config?: AxiosRequestConfig
  ): Promise<T> {
    const response = await apiClient.patch<T>(`${this.endpoint}${path}`, data, config);
    return response.data;
  }

  protected async delete<T>(
    path: string = '',
    config?: AxiosRequestConfig
  ): Promise<T> {
    const response = await apiClient.delete<T>(`${this.endpoint}${path}`, config);
    return response.data;
  }

  protected buildQueryString(params: QueryParams): string {
    const query = new URLSearchParams();
    
    if (params.page) query.append('page', params.page.toString());
    if (params.limit) query.append('limit', params.limit.toString());
    if (params.sort) query.append('sort', params.sort);
    if (params.order) query.append('order', params.order);
    if (params.search) query.append('search', params.search);
    
    if (params.filters) {
      Object.entries(params.filters).forEach(([key, value]) => {
        if (value !== undefined && value !== null && value !== '') {
          query.append(key, value.toString());
        }
      });
    }
    
    return query.toString() ? `?${query.toString()}` : '';
  }
}

// Event service implementation
export class EventService extends BaseService {
  constructor() {
    super('/events');
  }

  async getEvents(params?: QueryParams): Promise<PaginatedResponse<Event>> {
    const queryString = this.buildQueryString(params || {});
    return this.get<PaginatedResponse<Event>>(queryString);
  }

  async getEvent(id: string): Promise<Event> {
    return this.get<Event>(`/${id}`);
  }

  async createEvent(data: CreateEventInput): Promise<Event> {
    return this.post<Event>('', data);
  }

  async updateEvent(id: string, data: UpdateEventInput): Promise<Event> {
    return this.put<Event>(`/${id}`, data);
  }

  async deleteEvent(id: string): Promise<void> {
    return this.delete<void>(`/${id}`);
  }

  async publishEvent(id: string): Promise<Event> {
    return this.post<Event>(`/${id}/publish`);
  }

  async unpublishEvent(id: string): Promise<Event> {
    return this.post<Event>(`/${id}/unpublish`);
  }

  async getEventAttendees(
    id: string,
    params?: QueryParams
  ): Promise<PaginatedResponse<Attendee>> {
    const queryString = this.buildQueryString(params || {});
    return this.get<PaginatedResponse<Attendee>>(`/${id}/attendees${queryString}`);
  }

  async sendWhatsAppInvites(
    id: string,
    data: { phoneNumbers: string[]; message: string }
  ): Promise<void> {
    return this.post<void>(`/${id}/whatsapp-invites`, data);
  }
}

// Service registry
export const eventService = new EventService();
export const userService = new UserService();
export const adminService = new AdminService();
export const developerService = new DeveloperService();
export const rfpService = new RFPService();
export const analyticsService = new AnalyticsService();
```

## Authentication & Security Patterns

### WhatsApp OTP Authentication
```typescript
// lib/auth/whatsapp-auth.ts
import { apiClient } from '@/lib/api/client';
import { z } from 'zod';

// Validation schemas
const phoneSchema = z.string().regex(/^\+?[1-9]\d{1,14}$/, 'Invalid phone number');
const otpSchema = z.string().length(6, 'OTP must be 6 digits');

export interface WhatsAppAuthState {
  phoneNumber: string;
  sessionId: string;
  expiresAt: Date;
  attempts: number;
  maxAttempts: number;
}

export class WhatsAppAuth {
  private static SESSION_KEY = 'whatsapp-auth-session';
  
  static async sendOTP(phoneNumber: string): Promise<WhatsAppAuthState> {
    // Validate phone number
    const validatedPhone = phoneSchema.parse(phoneNumber);
    
    try {
      const response = await apiClient.post('/auth/whatsapp-otp', {
        phoneNumber: validatedPhone,
      });
      
      const session: WhatsAppAuthState = {
        phoneNumber: validatedPhone,
        sessionId: response.data.sessionId,
        expiresAt: new Date(response.data.expiresAt),
        attempts: 0,
        maxAttempts: response.data.maxAttempts || 3,
      };
      
      // Store session
      sessionStorage.setItem(this.SESSION_KEY, JSON.stringify(session));
      
      return session;
    } catch (error) {
      throw new Error('Failed to send OTP');
    }
  }
  
  static async verifyOTP(otp: string): Promise<AuthResponse> {
    // Get session
    const sessionStr = sessionStorage.getItem(this.SESSION_KEY);
    if (!sessionStr) {
      throw new Error('No active OTP session');
    }
    
    const session: WhatsAppAuthState = JSON.parse(sessionStr);
    
    // Check expiration
    if (new Date() > new Date(session.expiresAt)) {
      sessionStorage.removeItem(this.SESSION_KEY);
      throw new Error('OTP session expired');
    }
    
    // Validate OTP format
    const validatedOTP = otpSchema.parse(otp);
    
    try {
      const response = await apiClient.post('/auth/verify-otp', {
        sessionId: session.sessionId,
        otp: validatedOTP,
      });
      
      // Clear session
      sessionStorage.removeItem(this.SESSION_KEY);
      
      // Store tokens
      const { accessToken, refreshToken, user } = response.data;
      localStorage.setItem('auth-token', accessToken);
      localStorage.setItem('refresh-token', refreshToken);
      localStorage.setItem('user', JSON.stringify(user));
      
      return response.data;
    } catch (error: any) {
      // Update attempts
      session.attempts += 1;
      
      if (session.attempts >= session.maxAttempts) {
        sessionStorage.removeItem(this.SESSION_KEY);
        throw new Error('Maximum OTP attempts exceeded');
      }
      
      sessionStorage.setItem(this.SESSION_KEY, JSON.stringify(session));
      
      throw new Error(
        error.response?.data?.message || 
        `Invalid OTP. ${session.maxAttempts - session.attempts} attempts remaining`
      );
    }
  }
  
  static async resendOTP(): Promise<WhatsAppAuthState> {
    const sessionStr = sessionStorage.getItem(this.SESSION_KEY);
    if (!sessionStr) {
      throw new Error('No active OTP session');
    }
    
    const session: WhatsAppAuthState = JSON.parse(sessionStr);
    return this.sendOTP(session.phoneNumber);
  }
  
  static clearSession(): void {
    sessionStorage.removeItem(this.SESSION_KEY);
  }
}

// WhatsApp OTP Component
export function WhatsAppOTPModal({ 
  isOpen, 
  onClose, 
  phoneNumber,
  onSuccess 
}: WhatsAppOTPModalProps) {
  const [otp, setOtp] = useState(['', '', '', '', '', '']);
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);
  const [session, setSession] = useState<WhatsAppAuthState | null>(null);
  const inputRefs = useRef<(HTMLInputElement | null)[]>([]);
  
  useEffect(() => {
    if (isOpen && phoneNumber) {
      sendOTP();
    }
  }, [isOpen, phoneNumber]);
  
  const sendOTP = async () => {
    setIsLoading(true);
    setError(null);
    
    try {
      const session = await WhatsAppAuth.sendOTP(phoneNumber);
      setSession(session);
      toast.success('OTP sent to your WhatsApp');
    } catch (error: any) {
      setError(error.message);
      toast.error('Failed to send OTP');
    } finally {
      setIsLoading(false);
    }
  };
  
  const handleOTPChange = (index: number, value: string) => {
    if (!/^\d*$/.test(value)) return;
    
    const newOtp = [...otp];
    newOtp[index] = value;
    setOtp(newOtp);
    
    // Auto-focus next input
    if (value && index < 5) {
      inputRefs.current[index + 1]?.focus();
    }
    
    // Auto-submit if all digits entered
    if (newOtp.every(digit => digit) && newOtp.join('').length === 6) {
      verifyOTP(newOtp.join(''));
    }
  };
  
  const handleKeyDown = (index: number, e: KeyboardEvent<HTMLInputElement>) => {
    if (e.key === 'Backspace' && !otp[index] && index > 0) {
      inputRefs.current[index - 1]?.focus();
    }
  };
  
  const verifyOTP = async (otpCode?: string) => {
    const code = otpCode || otp.join('');
    if (code.length !== 6) {
      setError('Please enter all 6 digits');
      return;
    }
    
    setIsLoading(true);
    setError(null);
    
    try {
      const response = await WhatsAppAuth.verifyOTP(code);
      toast.success('Successfully authenticated!');
      onSuccess(response);
      onClose();
    } catch (error: any) {
      setError(error.message);
      setOtp(['', '', '', '', '', '']);
      inputRefs.current[0]?.focus();
    } finally {
      setIsLoading(false);
    }
  };
  
  return (
    <Dialog open={isOpen} onOpenChange={onClose}>
      <DialogContent className="sm:max-w-md">
        <DialogHeader>
          <DialogTitle>Enter WhatsApp OTP</DialogTitle>
          <DialogDescription>
            We've sent a 6-digit code to {phoneNumber}
          </DialogDescription>
        </DialogHeader>
        
        <div className="space-y-6 py-4">
          <div className="flex justify-center gap-2">
            {otp.map((digit, index) => (
              <Input
                key={index}
                ref={(el) => (inputRefs.current[index] = el)}
                type="text"
                inputMode="numeric"
                maxLength={1}
                value={digit}
                onChange={(e) => handleOTPChange(index, e.target.value)}
                onKeyDown={(e) => handleKeyDown(index, e)}
                className="w-12 h-12 text-center text-lg font-semibold"
                disabled={isLoading}
              />
            ))}
          </div>
          
          {error && (
            <Alert variant="destructive">
              <AlertDescription>{error}</AlertDescription>
            </Alert>
          )}
          
          {session && (
            <div className="text-center text-sm text-muted-foreground">
              <p>
                Code expires in{' '}
                <CountdownTimer
                  expiresAt={session.expiresAt}
                  onExpire={() => {
                    setError('OTP expired. Please request a new one.');
                    WhatsAppAuth.clearSession();
                  }}
                />
              </p>
            </div>
          )}
          
          <div className="flex flex-col gap-2">
            <Button
              onClick={() => verifyOTP()}
              disabled={isLoading || otp.some(d => !d)}
              className="w-full"
            >
              {isLoading ? (
                <>
                  <Loader2 className="mr-2 h-4 w-4 animate-spin" />
                  Verifying...
                </>
              ) : (
                'Verify OTP'
              )}
            </Button>
            
            <Button
              variant="outline"
              onClick={sendOTP}
              disabled={isLoading}
              className="w-full"
            >
              Resend OTP
            </Button>
          </div>
        </div>
      </DialogContent>
    </Dialog>
  );
}
```

### Role-Based Access Control (RBAC)
```typescript
// lib/auth/rbac.ts
import { AdminType, Action, Permission } from '@/types/admin.types';

export class RBAC {
  private static instance: RBAC;
  private permissions: Map<string, Set<string>> = new Map();
  
  private constructor() {
    this.initializePermissions();
  }
  
  static getInstance(): RBAC {
    if (!RBAC.instance) {
      RBAC.instance = new RBAC();
    }
    return RBAC.instance;
  }
  
  private initializePermissions() {
    // Define default permissions for each admin type
    const defaultPermissions: Record<AdminType, Permission[]> = {
      [AdminType.SUPER_ADMIN]: [
        { resource: '*', actions: [Action.READ, Action.WRITE, Action.DELETE, Action.CONFIGURE] },
      ],
      [AdminType.PLATFORM_ADMIN]: [
        { resource: 'events', actions: [Action.READ, Action.WRITE, Action.DELETE] },
        { resource: 'users', actions: [Action.READ, Action.WRITE] },
        { resource: 'analytics', actions: [Action.READ] },
      ],
      [AdminType.REGIONAL_ADMIN]: [
        { resource: 'events', actions: [Action.READ, Action.WRITE], conditions: { region: 'assigned' } },
        { resource: 'users', actions: [Action.READ], conditions: { region: 'assigned' } },
      ],
      // ... other admin types
    };
    
    // Build permission map
    Object.entries(defaultPermissions).forEach(([adminType, permissions]) => {
      const key = this.getPermissionKey(adminType as AdminType);
      const permSet = new Set<string>();
      
      permissions.forEach(perm => {
        perm.actions.forEach(action => {
          permSet.add(`${perm.resource}:${action}`);
        });
      });
      
      this.permissions.set(key, permSet);
    });
  }
  
  hasPermission(
    adminType: AdminType,
    resource: string,
    action: Action,
    context?: Record<string, any>
  ): boolean {
    const key = this.getPermissionKey(adminType);
    const permissions = this.permissions.get(key);
    
    if (!permissions) return false;
    
    // Check wildcard permission
    if (permissions.has(`*:${action}`)) return true;
    
    // Check specific permission
    return permissions.has(`${resource}:${action}`);
  }
  
  private getPermissionKey(adminType: AdminType): string {
    return `admin:${adminType}`;
  }
}

// Permission hook
export function usePermissions() {
  const { user } = useAppStore();
  const rbac = RBAC.getInstance();
  
  const hasPermission = useCallback(
    (resource: string, action: Action, context?: Record<string, any>) => {
      if (!user || !user.adminType) return false;
      return rbac.hasPermission(user.adminType, resource, action, context);
    },
    [user, rbac]
  );
  
  const hasAnyPermission = useCallback(
    (permissions: Array<{ resource: string; action: Action }>) => {
      return permissions.some(({ resource, action }) => 
        hasPermission(resource, action)
      );
    },
    [hasPermission]
  );
  
  const hasAllPermissions = useCallback(
    (permissions: Array<{ resource: string; action: Action }>) => {
      return permissions.every(({ resource, action }) => 
        hasPermission(resource, action)
      );
    },
    [hasPermission]
  );
  
  return {
    hasPermission,
    hasAnyPermission,
    hasAllPermissions,
  };
}

// Protected route component
export function ProtectedRoute({ 
  children, 
  resource, 
  action,
  fallback = <Navigate to="/unauthorized" replace />
}: ProtectedRouteProps) {
  const { hasPermission } = usePermissions();
  
  if (!hasPermission(resource, action)) {
    return fallback;
  }
  
  return <>{children}</>;
}

// Permission guard component
export function PermissionGuard({ 
  children, 
  resource, 
  action,
  fallback = null
}: PermissionGuardProps) {
  const { hasPermission } = usePermissions();
  
  if (!hasPermission(resource, action)) {
    return <>{fallback}</>;
  }
  
  return <>{children}</>;
}
```

### Security Headers & CSP
```typescript
// middleware/security-headers.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function securityHeadersMiddleware(request: NextRequest) {
  const response = NextResponse.next();
  
  // Security headers
  response.headers.set('X-Frame-Options', 'DENY');
  response.headers.set('X-Content-Type-Options', 'nosniff');
  response.headers.set('X-XSS-Protection', '1; mode=block');
  response.headers.set('Referrer-Policy', 'strict-origin-when-cross-origin');
  response.headers.set('Permissions-Policy', 'camera=(), microphone=(), geolocation=()');
  
  // Content Security Policy
  const csp = [
    "default-src 'self'",
    "script-src 'self' 'unsafe-inline' 'unsafe-eval' https://cdn.jsdelivr.net https://cdnjs.cloudflare.com https://pagead2.googlesyndication.com",
    "style-src 'self' 'unsafe-inline' https://fonts.googleapis.com",
    "font-src 'self' https://fonts.gstatic.com",
    "img-src 'self' data: https: blob:",
    "connect-src 'self' https://api.eventfulhq.com wss://ws.eventfulhq.com https://pagead2.googlesyndication.com",
    "frame-src 'self' https://googleads.g.doubleclick.net",
    "object-src 'none'",
    "base-uri 'self'",
    "form-action 'self'",
    "frame-ancestors 'none'",
    "upgrade-insecure-requests",
  ].join('; ');
  
  response.headers.set('Content-Security-Policy', csp);
  
  // HSTS for production
  if (process.env.NODE_ENV === 'production') {
    response.headers.set(
      'Strict-Transport-Security',
      'max-age=63072000; includeSubDomains; preload'
    );
  }
  
  return response;
}
```

## Performance Optimization & SLA Compliance

### Next.js Performance Configuration
```typescript
// next.config.mjs
import { withSentryConfig } from '@sentry/nextjs';
import million from 'million/compiler';

/** @type {import('next').NextConfig} */
const nextConfig = {
  // React compiler for optimization
  experimental: {
    reactCompiler: true,
    optimizePackageImports: [
      '@shadcn/ui',
      'lucide-react',
      '@tabler/icons-react',
      'recharts',
      'd3',
    ],
    turbo: {
      resolveAlias: {
        '@': './src',
      },
    },
  },
  
  // Image optimization
  images: {
    domains: ['cdn.eventfulhq.com'],
    formats: ['image/avif', 'image/webp'],
    deviceSizes: [640, 750, 828, 1080, 1200, 1920, 2048, 3840],
    imageSizes: [16, 32, 48, 64, 96, 128, 256, 384],
  },
  
  // Bundle optimization
  webpack: (config, { isServer }) => {
    // Tree shaking
    config.optimization.usedExports = true;
    config.optimization.sideEffects = false;
    
    // Module federation for microfrontends
    if (!isServer) {
      config.optimization.splitChunks = {
        chunks: 'all',
        cacheGroups: {
          default: false,
          vendors: false,
          framework: {
            chunks: 'all',
            name: 'framework',
            test: /(?<!node_modules.*)[\\/]node_modules[\\/](react|react-dom|scheduler|prop-types|use-subscription)[\\/]/,
            priority: 40,
            enforce: true,
          },
          lib: {
            test(module) {
              return module.size() > 160000 &&
                /node_modules[/\\]/.test(module.identifier());
            },
            name(module) {
              const hash = crypto.createHash('sha1');
              hash.update(module.identifier());
              return hash.digest('hex').substring(0, 8);
            },
            priority: 30,
            minChunks: 1,
            reuseExistingChunk: true,
          },
          commons: {
            name: 'commons',
            minChunks: 2,
            priority: 20,
          },
          shared: {
            name(module, chunks) {
              return crypto
                .createHash('sha1')
                .update(chunks.reduce((acc, chunk) => acc + chunk.name, ''))
                .digest('hex') + (isServer ? '-server' : '');
            },
            priority: 10,
            minChunks: 2,
            reuseExistingChunk: true,
          },
        },
      };
    }
    
    return config;
  },
  
  // Output optimization
  output: 'standalone',
  compress: true,
  poweredByHeader: false,
  generateEtags: true,
  
  // Compiler optimizations
  compiler: {
    removeConsole: process.env.NODE_ENV === 'production',
    reactRemoveProperties: process.env.NODE_ENV === 'production',
  },
};

// Apply Million.js compiler for React optimization
const millionConfig = {
  auto: true,
  rsc: true,
};

// Sentry configuration
const sentryWebpackPluginOptions = {
  silent: true,
  widenClientFileUpload: true,
};

export default million.next(
  withSentryConfig(nextConfig, sentryWebpackPluginOptions),
  millionConfig
);
```

### Performance Monitoring
```typescript
// lib/performance/monitoring.ts
import { getCLS, getFID, getLCP, getFCP, getTTFB } from 'web-vitals';

export class PerformanceMonitor {
  private static instance: PerformanceMonitor;
  private metrics: Map<string, number> = new Map();
  
  private constructor() {
    this.initializeVitals();
    this.setupResourceObserver();
    this.setupLongTaskObserver();
  }
  
  static getInstance(): PerformanceMonitor {
    if (!PerformanceMonitor.instance) {
      PerformanceMonitor.instance = new PerformanceMonitor();
    }
    return PerformanceMonitor.instance;
  }
  
  private initializeVitals() {
    // Core Web Vitals
    getCLS(this.sendToAnalytics);
    getFID(this.sendToAnalytics);
    getLCP(this.sendToAnalytics);
    
    // Additional metrics
    getFCP(this.sendToAnalytics);
    getTTFB(this.sendToAnalytics);
  }
  
  private setupResourceObserver() {
    if ('PerformanceObserver' in window) {
      const observer = new PerformanceObserver((list) => {
        for (const entry of list.getEntries()) {
          if (entry.entryType === 'resource') {
            this.trackResourceTiming(entry as PerformanceResourceTiming);
          }
        }
      });
      
      observer.observe({ entryTypes: ['resource'] });
    }
  }
  
  private setupLongTaskObserver() {
    if ('PerformanceObserver' in window) {
      const observer = new PerformanceObserver((list) => {
        for (const entry of list.getEntries()) {
          console.warn('Long task detected:', {
            duration: entry.duration,
            startTime: entry.startTime,
          });
          
          // Report long tasks
          this.sendToAnalytics({
            name: 'long-task',
            value: entry.duration,
            id: generateId(),
            entries: [entry],
          });
        }
      });
      
      try {
        observer.observe({ entryTypes: ['longtask'] });
      } catch (e) {
        // Long task observer not supported
      }
    }
  }
  
  private trackResourceTiming(entry: PerformanceResourceTiming) {
    const duration = entry.responseEnd - entry.startTime;
    
    // Track slow resources
    if (duration > 1000) {
      console.warn('Slow resource:', {
        name: entry.name,
        duration,
        type: entry.initiatorType,
      });
    }
    
    // Categorize by type
    const category = this.getResourceCategory(entry);
    const currentTotal = this.metrics.get(category) || 0;
    this.metrics.set(category, currentTotal + duration);
  }
  
  private getResourceCategory(entry: PerformanceResourceTiming): string {
    if (entry.name.includes('/api/')) return 'api';
    if (entry.initiatorType === 'script') return 'script';
    if (entry.initiatorType === 'css') return 'css';
    if (entry.initiatorType === 'img') return 'image';
    return 'other';
  }
  
  private sendToAnalytics = (metric: any) => {
    // Send to analytics service
    if (window.gtag) {
      window.gtag('event', metric.name, {
        value: Math.round(metric.value),
        metric_id: metric.id,
        metric_value: metric.value,
        metric_delta: metric.delta,
      });
    }
    
    // Send to custom monitoring
    fetch('/api/monitoring/metrics', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        metric: metric.name,
        value: metric.value,
        timestamp: Date.now(),
        url: window.location.href,
        userAgent: navigator.userAgent,
      }),
    }).catch(() => {
      // Fail silently
    });
  };
  
  measureApiCall(endpoint: string, duration: number) {
    this.sendToAnalytics({
      name: 'api-call',
      value: duration,
      id: generateId(),
      entries: [],
      endpoint,
    });
  }
  
  getMetrics(): Record<string, number> {
    return Object.fromEntries(this.metrics);
  }
}

// Performance hook
export function usePerformance() {
  const monitor = PerformanceMonitor.getInstance();
  
  const measureApiCall = useCallback((endpoint: string, duration: number) => {
    monitor.measureApiCall(endpoint, duration);
  }, [monitor]);
  
  const getMetrics = useCallback(() => {
    return monitor.getMetrics();
  }, [monitor]);
  
  return {
    measureApiCall,
    getMetrics,
  };
}
```

### Image Optimization Component
```typescript
// components/shared/optimized-image.tsx
'use client';

import Image from 'next/image';
import { useState, useEffect } from 'react';
import { cn } from '@/lib/utils';

interface OptimizedImageProps {
  src: string;
  alt: string;
  width?: number;
  height?: number;
  priority?: boolean;
  className?: string;
  objectFit?: 'contain' | 'cover' | 'fill' | 'none' | 'scale-down';
  quality?: number;
  sizes?: string;
  onLoad?: () => void;
  placeholder?: 'blur' | 'empty';
  blurDataURL?: string;
}

export function OptimizedImage({
  src,
  alt,
  width,
  height,
  priority = false,
  className,
  objectFit = 'cover',
  quality = 75,
  sizes,
  onLoad,
  placeholder = 'blur',
  blurDataURL,
}: OptimizedImageProps) {
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState(false);
  
  // Generate blur data URL if not provided
  const shimmer = (w: number, h: number) => `
    <svg width="${w}" height="${h}" version="1.1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink">
      <defs>
        <linearGradient id="g">
          <stop stop-color="#333" offset="20%" />
          <stop stop-color="#222" offset="50%" />
          <stop stop-color="#333" offset="70%" />
        </linearGradient>
      </defs>
      <rect width="${w}" height="${h}" fill="#333" />
      <rect id="r" width="${w}" height="${h}" fill="url(#g)" />
      <animate xlink:href="#r" attributeName="x" from="-${w}" to="${w}" dur="1s" repeatCount="indefinite"  />
    </svg>`;
  
  const toBase64 = (str: string) =>
    typeof window === 'undefined'
      ? Buffer.from(str).toString('base64')
      : window.btoa(str);
  
  const defaultBlurDataURL = `data:image/svg+xml;base64,${toBase64(
    shimmer(width || 700, height || 475)
  )}`;
  
  // Cloudinary optimization
  const getOptimizedSrc = (src: string) => {
    if (src.includes('cloudinary')) {
      const parts = src.split('/upload/');
      if (parts.length === 2) {
        const transformations = [
          'f_auto', // Auto format
          'q_auto', // Auto quality
          'dpr_auto', // Auto DPR
          width ? `w_${width}` : '',
          height ? `h_${height}` : '',
        ].filter(Boolean).join(',');
        
        return `${parts[0]}/upload/${transformations}/${parts[1]}`;
      }
    }
    return src;
  };
  
  const optimizedSrc = getOptimizedSrc(src);
  
  if (error) {
    return (
      <div 
        className={cn(
          'bg-muted flex items-center justify-center',
          className
        )}
        style={{ width, height }}
      >
        <span className="text-muted-foreground text-sm">
          Failed to load image
        </span>
      </div>
    );
  }
  
  return (
    <div className={cn('relative overflow-hidden', className)}>
      <Image
        src={optimizedSrc}
        alt={alt}
        width={width}
        height={height}
        priority={priority}
        quality={quality}
        sizes={sizes || '(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw'}
        className={cn(
          'duration-700 ease-in-out',
          isLoading ? 'scale-110 blur-2xl grayscale' : 'scale-100 blur-0 grayscale-0',
          objectFit === 'cover' && 'object-cover',
          objectFit === 'contain' && 'object-contain',
          className
        )}
        onLoad={() => {
          setIsLoading(false);
          onLoad?.();
        }}
        onError={() => {
          setError(true);
          setIsLoading(false);
        }}
        placeholder={placeholder}
        blurDataURL={blurDataURL || defaultBlurDataURL}
      />
    </div>
  );
}

// Lazy load component
export function LazyImage({ threshold = 0.1, ...props }: OptimizedImageProps & { threshold?: number }) {
  const [isInView, setIsInView] = useState(false);
  const imgRef = useRef<HTMLDivElement>(null);
  
  useEffect(() => {
    const observer = new IntersectionObserver(
      (entries) => {
        entries.forEach((entry) => {
          if (entry.isIntersecting) {
            setIsInView(true);
            observer.disconnect();
          }
        });
      },
      { threshold }
    );
    
    if (imgRef.current) {
      observer.observe(imgRef.current);
    }
    
    return () => observer.disconnect();
  }, [threshold]);
  
  return (
    <div ref={imgRef}>
      {isInView ? (
        <OptimizedImage {...props} />
      ) : (
        <div 
          className={cn('bg-muted animate-pulse', props.className)}
          style={{ width: props.width, height: props.height }}
        />
      )}
    </div>
  );
}
```

### Code Splitting & Dynamic Imports
```typescript
// lib/performance/dynamic-imports.ts
import dynamic from 'next/dynamic';
import { ComponentType } from 'react';

// Dynamic import with loading state
export function createDynamicComponent<P = {}>(
  importFn: () => Promise<{ default: ComponentType<P> }>,
  options?: {
    loading?: ComponentType;
    ssr?: boolean;
  }
) {
  return dynamic(importFn, {
    loading: options?.loading || (() => <div>Loading...</div>),
    ssr: options?.ssr ?? true,
  });
}

// Pre-configured dynamic imports
export const DynamicEventCard = createDynamicComponent(
  () => import('@/components/features/events/event-card').then(mod => ({ default: mod.EventCard }))
);

export const DynamicSwaggerUI = createDynamicComponent(
  () => import('swagger-ui-react'),
  { ssr: false }
);

export const DynamicMonacoEditor = createDynamicComponent(
  () => import('@monaco-editor/react'),
  { ssr: false }
);

export const DynamicCharts = createDynamicComponent(
  () => import('@/components/features/analytics/charts')
);

// Route-based code splitting
export const DynamicAdminDashboard = createDynamicComponent(
  () => import('@/components/features/admin/super-admin-dashboard').then(mod => ({ default: mod.SuperAdminDashboard }))
);

export const DynamicDeveloperPortal = createDynamicComponent(
  () => import('@/components/features/developers/developer-dashboard').then(mod => ({ default: mod.DeveloperDashboard }))
);

// Lazy load heavy libraries
export const loadD3 = () => import('d3');
export const loadThree = () => import('three');
export const loadChartJS = () => import('chart.js');
```

## Testing Strategy

### Unit Testing Configuration
```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config';
import react from '@vitejs/plugin-react';
import path from 'path';

export default defineConfig({
  plugins: [react()],
  test: {
    environment: 'jsdom',
    globals: true,
    setupFiles: ['./tests/setup.ts'],
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html'],
      exclude: [
        'node_modules/',
        'tests/',
        '**/*.d.ts',
        '**/*.config.*',
        '**/*.stories.*',
        'src/services/generated/**',
      ],
      thresholds: {
        branches: 80,
        functions: 80,
        lines: 80,
        statements: 80,
      },
    },
  },
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
});

// tests/setup.ts
import '@testing-library/jest-dom';
import { cleanup } from '@testing-library/react';
import { afterEach, vi } from 'vitest';
import { server } from './mocks/server';

// Establish API mocking before all tests
beforeAll(() => server.listen({ onUnhandledRequest: 'error' }));

// Reset any request handlers that we may add during the tests,
// so they don't affect other tests
afterEach(() => {
  cleanup();
  server.resetHandlers();
});

// Clean up after the tests are finished
afterAll(() => server.close());

// Mock next/navigation
vi.mock('next/navigation', () => ({
  useRouter: () => ({
    push: vi.fn(),
    replace: vi.fn(),
    prefetch: vi.fn(),
    back: vi.fn(),
  }),
  usePathname: () => '/test-path',
  useSearchParams: () => new URLSearchParams(),
}));

// Mock next-intl
vi.mock('next-intl', () => ({
  useTranslations: () => (key: string) => key,
  useLocale: () => 'en',
}));
```

### Component Testing Examples
```typescript
// tests/components/event-card.test.tsx
import { describe, it, expect, vi } from 'vitest';
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { EventCard } from '@/components/features/events/event-card';
import { mockEvent } from '@/tests/mocks/event';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';

const queryClient = new QueryClient({
  defaultOptions: {
    queries: { retry: false },
    mutations: { retry: false },
  },
});

const renderWithProviders = (component: React.ReactElement) => {
  return render(
    <QueryClientProvider client={queryClient}>
      {component}
    </QueryClientProvider>
  );
};

describe('EventCard', () => {
  it('renders event information correctly', () => {
    renderWithProviders(<EventCard event={mockEvent} />);
    
    expect(screen.getByText(mockEvent.title)).toBeInTheDocument();
    expect(screen.getByText(mockEvent.venue.name)).toBeInTheDocument();
    expect(screen.getByText(`${mockEvent.price}`)).toBeInTheDocument();
  });
  
  it('shows sold out badge when event is sold out', () => {
    const soldOutEvent = { ...mockEvent, isSoldOut: true };
    renderWithProviders(<EventCard event={soldOutEvent} />);
    
    expect(screen.getByText('Sold Out')).toBeInTheDocument();
  });
  
  it('displays WhatsApp RSVP button when enabled', () => {
    const whatsappEvent = {
      ...mockEvent,
      whatsappRSVP: {
        enabled: true,
        phoneNumber: '+1234567890',
        message: 'RSVP for {event_name}',
      },
    };
    renderWithProviders(<EventCard event={whatsappEvent} />);
    
    expect(screen.getByText('RSVP via WhatsApp')).toBeInTheDocument();
  });
  
  it('calls onSelect when card is clicked', async () => {
    const onSelect = vi.fn();
    renderWithProviders(<EventCard event={mockEvent} onSelect={onSelect} />);
    
    const card = screen.getByRole('article');
    await userEvent.click(card);
    
    expect(onSelect).toHaveBeenCalledWith(mockEvent);
  });
  
  it('shares event when share button is clicked', async () => {
    const onShare = vi.fn();
    renderWithProviders(<EventCard event={mockEvent} onShare={onShare} />);
    
    const shareButton = screen.getByRole('button', { name: /share/i });
    await userEvent.click(shareButton);
    
    expect(onShare).toHaveBeenCalledWith(mockEvent);
  });
  
  it('displays loading skeleton when isLoading is true', () => {
    renderWithProviders(<EventCard event={mockEvent} isLoading />);
    
    expect(screen.getByTestId('event-card-skeleton')).toBeInTheDocument();
  });
});
```

### Integration Testing
```typescript
// tests/integration/event-creation.test.tsx
import { describe, it, expect, beforeEach } from 'vitest';
import { render, screen, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { CreateEventPage } from '@/app/[locale]/(dashboard)/events/create/page';
import { server } from '@/tests/mocks/server';
import { rest } from 'msw';

describe('Event Creation Flow', () => {
  beforeEach(() => {
    // Reset MSW handlers
    server.resetHandlers();
  });
  
  it('completes full event creation flow', async () => {
    const user = userEvent.setup();
    render(<CreateEventPage />);
    
    // Fill in event details
    await user.type(screen.getByLabelText('Event Title'), 'Test Music Festival');
    await user.type(screen.getByLabelText('Description'), 'An amazing outdoor music experience');
    
    // Select date
    await user.click(screen.getByLabelText('Start Date'));
    await user.click(screen.getByText('15')); // Select 15th of current month
    
    // Fill venue information
    await user.type(screen.getByLabelText('Venue Name'), 'Central Park');
    await user.type(screen.getByLabelText('Address'), '5th Avenue');
    await user.type(screen.getByLabelText('City'), 'New York');
    
    // Set capacity and price
    await user.clear(screen.getByLabelText('Capacity'));
    await user.type(screen.getByLabelText('Capacity'), '500');
    await user.clear(screen.getByLabelText('Price'));
    await user.type(screen.getByLabelText('Price'), '75');
    
    // Enable WhatsApp RSVP
    await user.click(screen.getByLabelText('WhatsApp RSVP'));
    await user.type(screen.getByLabelText('WhatsApp Number'), '+1234567890');
    
    // Mock successful API response
    server.use(
      rest.post('/api/events', (req, res, ctx) => {
        return res(
          ctx.status(201),
          ctx.json({
            id: '123',
            ...req.body,
          })
        );
      })
    );
    
    // Submit form
    await user.click(screen.getByRole('button', { name: 'Create Event' }));
    
    // Verify success
    await waitFor(() => {
      expect(screen.getByText('Event created successfully!')).toBeInTheDocument();
    });
  });
  
  it('shows validation errors for invalid input', async () => {
    const user = userEvent.setup();
    render(<CreateEventPage />);
    
    // Try to submit without filling required fields
    await user.click(screen.getByRole('button', { name: 'Create Event' }));
    
    // Check for validation errors
    await waitFor(() => {
      expect(screen.getByText('Event title is required')).toBeInTheDocument();
      expect(screen.getByText('Description is required')).toBeInTheDocument();
      expect(screen.getByText('Event date is required')).toBeInTheDocument();
    });
  });
});
```

### E2E Testing with Playwright
```typescript
// tests/e2e/event-flow.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Event Management Flow', () => {
  test.beforeEach(async ({ page }) => {
    // Login before each test
    await page.goto('/login');
    await page.fill('input[name="phoneNumber"]', '+1234567890');
    await page.click('button:has-text("Send OTP")');
    
    // Fill OTP
    for (let i = 0; i < 6; i++) {
      await page.fill(`input[name="otp-${i}"]`, '1');
    }
    
    await page.waitForURL('/dashboard');
  });
  
  test('creates and publishes an event', async ({ page }) => {
    // Navigate to create event
    await page.goto('/events/create');
    
    // Fill event form
    await page.fill('input[name="title"]', 'Playwright Test Event');
    await page.fill('textarea[name="description"]', 'Test event created by Playwright');
    
    // Select date
    await page.click('button[aria-label="Start Date"]');
    await page.click('button:has-text("15")');
    
    // Fill venue
    await page.fill('input[name="venue.name"]', 'Test Venue');
    await page.fill('input[name="venue.address"]', '123 Test Street');
    await page.fill('input[name="venue.city"]', 'Test City');
    
    // Submit
    await page.click('button:has-text("Create Event")');
    
    // Wait for redirect
    await page.waitForURL(/\/events\/[\w-]+$/);
    
    // Verify event created
    await expect(page.locator('h1')).toContainText('Playwright Test Event');
    
    // Publish event
    await page.click('button:has-text("Publish Event")');
    
    // Verify published
    await expect(page.locator('text=Published')).toBeVisible();
  });
  
  test('searches and filters events', async ({ page }) => {
    await page.goto('/events');
    
    // Search
    await page.fill('input[placeholder="Search events..."]', 'Music');
    await page.press('input[placeholder="Search events..."]', 'Enter');
    
    // Apply filters
    await page.click('button:has-text("Filters")');
    await page.selectOption('select[name="category"]', 'music');
    await page.fill('input[name="minPrice"]', '0');
    await page.fill('input[name="maxPrice"]', '100');
    await page.click('button:has-text("Apply Filters")');
    
    // Verify results
    await expect(page.locator('[data-testid="event-card"]')).toHaveCount(10);
  });
});
```

### Performance Testing
```typescript
// tests/performance/event-list.perf.ts
import { test, expect } from '@playwright/test';

test.describe('Performance Tests', () => {
  test('event list page loads within performance budget', async ({ page }) => {
    // Start measuring
    await page.goto('/events');
    
    // Measure Core Web Vitals
    const metrics = await page.evaluate(() => {
      return new Promise((resolve) => {
        let lcp, fid, cls;
        
        new PerformanceObserver((list) => {
          const entries = list.getEntries();
          entries.forEach((entry) => {
            if (entry.name === 'largest-contentful-paint') {
              lcp = entry.startTime;
            }
          });
        }).observe({ entryTypes: ['largest-contentful-paint'] });
        
        new PerformanceObserver((list) => {
          const entries = list.getEntries();
          entries.forEach((entry) => {
            if (!fid) {
              fid = entry.processingStart - entry.startTime;
            }
          });
        }).observe({ entryTypes: ['first-input'] });
        
        let clsValue = 0;
        new PerformanceObserver((list) => {
          for (const entry of list.getEntries()) {
            if (!entry.hadRecentInput) {
              clsValue += entry.value;
            }
          }
          cls = clsValue;
        }).observe({ entryTypes: ['layout-shift'] });
        
        // Wait for metrics
        setTimeout(() => {
          resolve({ lcp, fid, cls });
        }, 5000);
      });
    });
    
    // Assert performance budgets
    expect(metrics.lcp).toBeLessThan(2500); // LCP < 2.5s
    expect(metrics.fid || 0).toBeLessThan(100); // FID < 100ms
    expect(metrics.cls || 0).toBeLessThan(0.1); // CLS < 0.1
  });
  
  test('API response times meet SLA', async ({ page, request }) => {
    const endpoints = [
      '/api/events',
      '/api/users/me',
      '/api/analytics/dashboard',
    ];
    
    for (const endpoint of endpoints) {
      const start = Date.now();
      const response = await request.get(endpoint);
      const duration = Date.now() - start;
      
      expect(response.status()).toBe(200);
      expect(duration).toBeLessThan(500); // p95 < 500ms
    }
  });
});
```

## Vercel Deployment & Edge Functions

### Vercel Configuration
```json
// vercel.json
{
  "buildCommand": "npm run build",
  "outputDirectory": ".next",
  "framework": "nextjs",
  "regions": ["iad1", "sfo1", "sin1", "syd1", "fra1", "gru1"],
  "functions": {
    "app/api/bff/*": {
      "maxDuration": 30
    },
    "app/api/webhooks/*": {
      "maxDuration": 60
    },
    "app/[locale]/(landing)/[country]/[city]/page.tsx": {
      "runtime": "edge"
    },
    "app/robots.txt/route.ts": {
      "runtime": "edge"
    },
    "app/sitemap.xml/route.ts": {
      "runtime": "edge"
    }
  },
  "crons": [
    {
      "path": "/api/cron/sitemap-refresh",
      "schedule": "0 3 * * *"
    },
    {
      "path": "/api/cron/cache-warm",
      "schedule": "*/15 * * * *"
    }
  ],
  "env": {
    "NEXT_PUBLIC_API_URL": "@api-url",
    "NEXT_PUBLIC_WS_URL": "@ws-url",
    "NEXT_PUBLIC_ADSENSE_CLIENT_ID": "@adsense-client-id"
  },
  "headers": [
    {
      "source": "/(.*)",
      "headers": [
        {
          "key": "X-Frame-Options",
          "value": "DENY"
        },
        {
          "key": "X-Content-Type-Options",
          "value": "nosniff"
        },
        {
          "key": "Referrer-Policy",
          "value": "strict-origin-when-cross-origin"
        }
      ]
    },
    {
      "source": "/api/(.*)",
      "headers": [
        {
          "key": "Cache-Control",
          "value": "no-store, max-age=0"
        }
      ]
    }
  ],
  "rewrites": [
    {
      "source": "/api/v1/:path*",
      "destination": "https://api.eventfulhq.com/v1/:path*"
    }
  ],
  "redirects": [
    {
      "source": "/events",
      "destination": "/en/events",
      "permanent": false
    }
  ]
}
```

### Edge Function Examples
```typescript
// app/api/edge/geo-redirect/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { geolocation } from '@vercel/edge';

export const runtime = 'edge';
export const dynamic = 'force-dynamic';

export async function GET(request: NextRequest) {
  const { country, city, region } = geolocation(request);
  
  // Determine best regional domain
  const regionalDomain = getRegionalDomain(country);
  
  // Return redirect response
  return NextResponse.json({
    country,
    city,
    region,
    recommendedDomain: regionalDomain,
    currentDomain: request.headers.get('host'),
  });
}

function getRegionalDomain(country?: string): string {
  if (!country) return 'eventfulhq.com';
  
  const regionMap: Record<string, string> = {
    US: 'eventfulamerica.com',
    CA: 'eventfulamerica.com',
    MX: 'eventfulamerica.com',
    BR: 'eventfulamerica.com',
    AE: 'eventfularabia.com',
    SA: 'eventfularabia.com',
    KW: 'eventfularabia.com',
    IN: 'eventfulindia.com',
    GB: 'eventfuleurope.com',
    FR: 'eventfuleurope.com',
    DE: 'eventfuleurope.com',
    AU: 'eventfulaustralia.com',
    NZ: 'eventfulaustralia.com',
  };
  
  return regionMap[country] || 'eventfulhq.com';
}
```

### ISR & Cache Strategy
```typescript
// app/[locale]/(landing)/[country]/[city]/page.tsx
export const revalidate = 3600; // Revalidate every hour
export const dynamicParams = true; // Allow dynamic segments

// Generate static params for popular cities
export async function generateStaticParams() {
  const cities = await landingService.getTopCities(100);
  
  return cities.map((city) => ({
    country: city.countryCode.toLowerCase(),
    city: city.slug,
  }));
}

// Cache configuration for different page types
export const cacheConfig = {
  // Static pages
  landing: {
    revalidate: 86400, // 24 hours
    tags: ['landing'],
  },
  
  // Dynamic user pages
  userProfile: {
    revalidate: 300, // 5 minutes
    tags: (username: string) => ['user', `user:${username}`],
  },
  
  // Event pages
  event: {
    revalidate: 60, // 1 minute for real-time updates
    tags: (eventId: string) => ['event', `event:${eventId}`],
  },
};

// On-demand revalidation
export async function revalidateEvent(eventId: string) {
  try {
    await fetch(`${process.env.DEPLOYMENT_URL}/api/revalidate`, {
      method: 'POST',
      headers: {
        'x-revalidate-secret': process.env.REVALIDATE_SECRET!,
      },
      body: JSON.stringify({
        tags: [`event:${eventId}`],
      }),
    });
  } catch (error) {
    console.error('Failed to revalidate:', error);
  }
}
```

### Environment-Specific Configuration
```typescript
// lib/config/environment.ts
interface EnvironmentConfig {
  apiUrl: string;
  wsUrl: string;
  cdnUrl: string;
  analyticsId: string;
  sentryDsn: string;
  features: Record<string, boolean>;
}

const configs: Record<string, EnvironmentConfig> = {
  development: {
    apiUrl: 'http://localhost:3001',
    wsUrl: 'ws://localhost:3001',
    cdnUrl: 'http://localhost:3000',
    analyticsId: '',
    sentryDsn: '',
    features: {
      whatsappAuth: true,
      rfpMarketplace: true,
      aiRecommendations: true,
      developerPortal: true,
      superAdmin: true,
    },
  },
  
  staging: {
    apiUrl: 'https://api-staging.eventfulhq.com',
    wsUrl: 'wss://ws-staging.eventfulhq.com',
    cdnUrl: 'https://cdn-staging.eventfulhq.com',
    analyticsId: process.env.NEXT_PUBLIC_GA_STAGING!,
    sentryDsn: process.env.NEXT_PUBLIC_SENTRY_DSN_STAGING!,
    features: {
      whatsappAuth: true,
      rfpMarketplace: true,
      aiRecommendations: true,
      developerPortal: true,
      superAdmin: true,
    },
  },
  
  production: {
    apiUrl: 'https://api.eventfulhq.com',
    wsUrl: 'wss://ws.eventfulhq.com',
    cdnUrl: 'https://cdn.eventfulhq.com',
    analyticsId: process.env.NEXT_PUBLIC_GA_ID!,
    sentryDsn: process.env.NEXT_PUBLIC_SENTRY_DSN!,
    features: {
      whatsappAuth: true,
      rfpMarketplace: true,
      aiRecommendations: false, // Gradual rollout
      developerPortal: true,
      superAdmin: true,
    },
  },
};

export const config = configs[process.env.NEXT_PUBLIC_ENV || 'development'];

// Feature flag helper
export function isFeatureEnabled(feature: keyof typeof config.features): boolean {
  return config.features[feature] || false;
}
```

## Component Enhancement Patterns

### MatDash to ShadCN Migration Pattern
```typescript
// migration/button-migration.tsx
// BEFORE: MatDash Flowbite Button
import { Button as FlowbiteButton } from 'flowbite-react';

export function OldButton({ children, onClick, variant }) {
  return (
    <FlowbiteButton
      color={variant === 'danger' ? 'failure' : variant}
      onClick={onClick}
    >
      {children}
    </FlowbiteButton>
  );
}

// AFTER: ShadCN Button with enhanced features
import { Button } from '@/components/ui/button';
import { cn } from '@/lib/utils';
import { Loader2 } from 'lucide-react';

interface EnhancedButtonProps {
  children: React.ReactNode;
  onClick?: () => void;
  variant?: 'default' | 'destructive' | 'outline' | 'secondary' | 'ghost' | 'link';
  size?: 'default' | 'sm' | 'lg' | 'icon';
  loading?: boolean;
  disabled?: boolean;
  className?: string;
  asChild?: boolean;
}

export function EnhancedButton({
  children,
  onClick,
  variant = 'default',
  size = 'default',
  loading = false,
  disabled = false,
  className,
  asChild = false,
}: EnhancedButtonProps) {
  return (
    <Button
      variant={variant}
      size={size}
      onClick={onClick}
      disabled={disabled || loading}
      className={cn(className)}
      asChild={asChild}
    >
      {loading && <Loader2 className="mr-2 h-4 w-4 animate-spin" />}
      {children}
    </Button>
  );
}

// Migration utility
export function migrateButtonProps(oldProps: any): EnhancedButtonProps {
  const variantMap = {
    primary: 'default',
    danger: 'destructive',
    failure: 'destructive',
    success: 'secondary',
    warning: 'outline',
  };
  
  return {
    ...oldProps,
    variant: variantMap[oldProps.color] || 'default',
    loading: oldProps.isProcessing,
  };
}
```

### Form Enhancement Pattern
```typescript
// components/enhanced/form-field.tsx
import { forwardRef } from 'react';
import { useFormContext } from 'react-hook-form';
import { FormControl, FormDescription, FormField, FormItem, FormLabel, FormMessage } from '@/components/ui/form';
import { Input } from '@/components/ui/input';
import { Textarea } from '@/components/ui/textarea';
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from '@/components/ui/select';
import { cn } from '@/lib/utils';

interface EnhancedFormFieldProps {
  name: string;
  label?: string;
  description?: string;
  placeholder?: string;
  type?: 'text' | 'email' | 'password' | 'number' | 'tel' | 'url' | 'textarea' | 'select';
  options?: Array<{ value: string; label: string }>;
  disabled?: boolean;
  className?: string;
  required?: boolean;
  autoComplete?: string;
}

export const EnhancedFormField = forwardRef<
  HTMLInputElement | HTMLTextAreaElement | HTMLButtonElement,
  EnhancedFormFieldProps
>(({
  name,
  label,
  description,
  placeholder,
  type = 'text',
  options = [],
  disabled = false,
  className,
  required = false,
  autoComplete,
}, ref) => {
  const { control } = useFormContext();
  
  return (
    <FormField
      control={control}
      name={name}
      render={({ field }) => (
        <FormItem className={className}>
          {label && (
            <FormLabel>
              {label}
              {required && <span className="text-destructive ml-1">*</span>}
            </FormLabel>
          )}
          <FormControl>
            {type === 'textarea' ? (
              <Textarea
                {...field}
                ref={ref as any}
                placeholder={placeholder}
                disabled={disabled}
                className="min-h-[100px]"
              />
            ) : type === 'select' ? (
              <Select
                onValueChange={field.onChange}
                defaultValue={field.value}
                disabled={disabled}
              >
                <FormControl>
                  <SelectTrigger ref={ref as any}>
                    <SelectValue placeholder={placeholder} />
                  </SelectTrigger>
                </FormControl>
                <SelectContent>
                  {options.map((option) => (
                    <SelectItem key={option.value} value={option.value}>
                      {option.label}
                    </SelectItem>
                  ))}
                </SelectContent>
              </Select>
            ) : (
              <Input
                {...field}
                ref={ref as any}
                type={type}
                placeholder={placeholder}
                disabled={disabled}
                autoComplete={autoComplete}
              />
            )}
          </FormControl>
          {description && <FormDescription>{description}</FormDescription>}
          <FormMessage />
        </FormItem>
      )}
    />
  );
});

EnhancedFormField.displayName = 'EnhancedFormField';

// Usage example
export function EnhancedEventForm() {
  const form = useForm({
    resolver: yupResolver(eventSchema),
  });
  
  return (
    <Form {...form}>
      <form onSubmit={form.handleSubmit(onSubmit)} className="space-y-6">
        <EnhancedFormField
          name="title"
          label="Event Title"
          placeholder="Summer Music Festival"
          required
        />
        
        <EnhancedFormField
          name="description"
          label="Description"
          type="textarea"
          placeholder="Describe your event..."
          required
        />
        
        <EnhancedFormField
          name="category"
          label="Category"
          type="select"
          placeholder="Select a category"
          options={[
            { value: 'music', label: 'Music' },
            { value: 'sports', label: 'Sports' },
            { value: 'arts', label: 'Arts & Culture' },
          ]}
          required
        />
        
        <Button type="submit">Create Event</Button>
      </form>
    </Form>
  );
}
```

### Data Table Enhancement Pattern
```typescript
// components/enhanced/data-table.tsx
import {
  ColumnDef,
  ColumnFiltersState,
  SortingState,
  VisibilityState,
  flexRender,
  getCoreRowModel,
  getFilteredRowModel,
  getPaginationRowModel,
  getSortedRowModel,
  useReactTable,
} from '@tanstack/react-table';
import {
  Table,
  TableBody,
  TableCell,
  TableHead,
  TableHeader,
  TableRow,
} from '@/components/ui/table';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import {
  DropdownMenu,
  DropdownMenuCheckboxItem,
  DropdownMenuContent,
  DropdownMenuTrigger,
} from '@/components/ui/dropdown-menu';
import { ChevronDown } from 'lucide-react';

interface DataTableProps<TData, TValue> {
  columns: ColumnDef<TData, TValue>[];
  data: TData[];
  searchKey?: string;
  showColumnVisibility?: boolean;
  showPagination?: boolean;
  pageSize?: number;
}

export function EnhancedDataTable<TData, TValue>({
  columns,
  data,
  searchKey,
  showColumnVisibility = true,
  showPagination = true,
  pageSize = 10,
}: DataTableProps<TData, TValue>) {
  const [sorting, setSorting] = useState<SortingState>([]);
  const [columnFilters, setColumnFilters] = useState<ColumnFiltersState>([]);
  const [columnVisibility, setColumnVisibility] = useState<VisibilityState>({});
  const [rowSelection, setRowSelection] = useState({});
  
  const table = useReactTable({
    data,
    columns,
    onSortingChange: setSorting,
    onColumnFiltersChange: setColumnFilters,
    getCoreRowModel: getCoreRowModel(),
    getPaginationRowModel: getPaginationRowModel(),
    getSortedRowModel: getSortedRowModel(),
    getFilteredRowModel: getFilteredRowModel(),
    onColumnVisibilityChange: setColumnVisibility,
    onRowSelectionChange: setRowSelection,
    state: {
      sorting,
      columnFilters,
      columnVisibility,
      rowSelection,
    },
    initialState: {
      pagination: {
        pageSize,
      },
    },
  });
  
  return (
    <div className="w-full">
      <div className="flex items-center py-4 gap-2">
        {searchKey && (
          <Input
            placeholder={`Search by ${searchKey}...`}
            value={(table.getColumn(searchKey)?.getFilterValue() as string) ?? ''}
            onChange={(event) =>
              table.getColumn(searchKey)?.setFilterValue(event.target.value)
            }
            className="max-w-sm"
          />
        )}
        {showColumnVisibility && (
          <DropdownMenu>
            <DropdownMenuTrigger asChild>
              <Button variant="outline" className="ml-auto">
                Columns <ChevronDown className="ml-2 h-4 w-4" />
              </Button>
            </DropdownMenuTrigger>
            <DropdownMenuContent align="end">
              {table
                .getAllColumns()
                .filter((column) => column.getCanHide())
                .map((column) => {
                  return (
                    <DropdownMenuCheckboxItem
                      key={column.id}
                      className="capitalize"
                      checked={column.getIsVisible()}
                      onCheckedChange={(value) =>
                        column.toggleVisibility(!!value)
                      }
                    >
                      {column.id}
                    </DropdownMenuCheckboxItem>
                  );
                })}
            </DropdownMenuContent>
          </DropdownMenu>
        )}
      </div>
      <div className="rounded-md border">
        <Table>
          <TableHeader>
            {table.getHeaderGroups().map((headerGroup) => (
              <TableRow key={headerGroup.id}>
                {headerGroup.headers.map((header) => {
                  return (
                    <TableHead key={header.id}>
                      {header.isPlaceholder
                        ? null
                        : flexRender(
                            header.column.columnDef.header,
                            header.getContext()
                          )}
                    </TableHead>
                  );
                })}
              </TableRow>
            ))}
          </TableHeader>
          <TableBody>
            {table.getRowModel().rows?.length ? (
              table.getRowModel().rows.map((row) => (
                <TableRow
                  key={row.id}
                  data-state={row.getIsSelected() && 'selected'}
                >
                  {row.getVisibleCells().map((cell) => (
                    <TableCell key={cell.id}>
                      {flexRender(
                        cell.column.columnDef.cell,
                        cell.getContext()
                      )}
                    </TableCell>
                  ))}
                </TableRow>
              ))
            ) : (
              <TableRow>
                <TableCell
                  colSpan={columns.length}
                  className="h-24 text-center"
                >
                  No results.
                </TableCell>
              </TableRow>
            )}
          </TableBody>
        </Table>
      </div>
      {showPagination && (
        <div className="flex items-center justify-end space-x-2 py-4">
          <div className="flex-1 text-sm text-muted-foreground">
            {table.getFilteredSelectedRowModel().rows.length} of{' '}
            {table.getFilteredRowModel().rows.length} row(s) selected.
          </div>
          <div className="space-x-2">
            <Button
              variant="outline"
              size="sm"
              onClick={() => table.previousPage()}
              disabled={!table.getCanPreviousPage()}
            >
              Previous
            </Button>
            <Button
              variant="outline"
              size="sm"
              onClick={() => table.nextPage()}
              disabled={!table.getCanNextPage()}
            >
              Next
            </Button>
          </div>
        </div>
      )}
    </div>
  );
}
```

## Lambda Function Requirements

### Lambda Function Architecture Context
```typescript
// Lambda functions need to handle the following frontend requirements:

interface LambdaRequirements {
  // Performance Requirements
  performance: {
    coldStart: '<1s for critical paths',
    warmResponse: '<200ms p50, <500ms p95',
    timeout: '30s default, 60s for heavy operations',
    memory: '256MB default, up to 3GB for processing',
    concurrency: '1000 concurrent executions',
  };
  
  // Security Requirements
  security: {
    authentication: 'JWT validation required',
    authorization: 'RBAC with 11 admin types',
    encryption: 'All data encrypted in transit and at rest',
    cors: 'Configured for all EventfulHQ domains',
    rateLimit: 'By user, IP, and API key',
  };
  
  // Integration Requirements
  integration: {
    database: 'RDS Proxy for connection pooling',
    cache: 'ElastiCache Redis for session and data cache',
    queue: 'SQS for async processing',
    events: 'EventBridge for inter-service communication',
    storage: 'S3 for media and documents',
  };
  
  // Response Format
  responseFormat: {
    success: {
      status: 200,
      body: {
        data: 'T',
        meta: {
          requestId: 'string',
          timestamp: 'ISO 8601',
          version: 'string',
        },
      },
    },
    error: {
      status: 'number',
      body: {
        error: {
          code: 'string',
          message: 'string',
          details: 'object',
          requestId: 'string',
        },
      },
    },
    pagination: {
      data: 'T[]',
      meta: {
        total: 'number',
        page: 'number',
        limit: 'number',
        totalPages: 'number',
      },
    },
  };
  
  // Feature-Specific Requirements
  features: {
    whatsappAuth: {
      otpGeneration: '6-digit numeric',
      otpExpiry: '5 minutes',
      maxAttempts: 3,
      sessionManagement: 'Redis with TTL',
    },
    eventManagement: {
      imageProcessing: 'Resize, optimize, generate thumbnails',
      geoLocation: 'Store and query by coordinates',
      realtimeUpdates: 'WebSocket via API Gateway',
      bulkOperations: 'Batch processing up to 1000 items',
    },
    adminSystem: {
      auditLogging: 'All admin actions logged',
      permissionChecks: 'Granular resource-action validation',
      geographyFilters: 'Region/country/city level access',
      featureFlags: 'Dynamic microservice control',
    },
    developerPortal: {
      apiKeyGeneration: 'Secure random 32 chars',
      usageTracking: 'Per endpoint, per key metrics',
      rateLimiting: 'Configurable per API key',
      webhookDelivery: 'Retry with exponential backoff',
    },
  };
}

// Lambda function template
export const lambdaHandler = async (event: APIGatewayProxyEvent): Promise<APIGatewayProxyResult> => {
  // Request ID for tracing
  const requestId = event.requestContext.requestId;
  
  try {
    // 1. Parse and validate input
    const body = JSON.parse(event.body || '{}');
    const validated = await schema.validate(body);
    
    // 2. Authenticate
    const user = await authenticate(event.headers.authorization);
    
    // 3. Authorize
    await authorize(user, resource, action);
    
    // 4. Execute business logic
    const result = await businessLogic(validated, user);
    
    // 5. Format response
    return {
      statusCode: 200,
      headers: {
        'Content-Type': 'application/json',
        'X-Request-ID': requestId,
        'Cache-Control': 'no-cache',
      },
      body: JSON.stringify({
        data: result,
        meta: {
          requestId,
          timestamp: new Date().toISOString(),
          version: '1.0',
        },
      }),
    };
  } catch (error) {
    // Error handling
    return handleError(error, requestId);
  }
};
```

### Lambda Layer Requirements
```typescript
// Shared Lambda layers for common functionality

// Layer 1: Core utilities
export const coreLayer = {
  name: 'eventfulhq-core',
  description: 'Core utilities and helpers',
  compatibleRuntimes: ['nodejs18.x', 'nodejs20.x'],
  contents: [
    'auth/jwt-validator',
    'auth/rbac',
    'database/connection-pool',
    'cache/redis-client',
    'monitoring/metrics',
    'logging/structured-logger',
    'validation/schemas',
    'utils/date-time',
    'utils/crypto',
    'middleware/error-handler',
    'middleware/request-validator',
  ],
};

// Layer 2: AWS SDK clients
export const awsLayer = {
  name: 'eventfulhq-aws',
  description: 'Optimized AWS SDK clients',
  compatibleRuntimes: ['nodejs18.x', 'nodejs20.x'],
  contents: [
    '@aws-sdk/client-dynamodb',
    '@aws-sdk/client-s3',
    '@aws-sdk/client-sqs',
    '@aws-sdk/client-eventbridge',
    '@aws-sdk/client-ses',
    '@aws-sdk/client-secrets-manager',
    '@aws-sdk/rds-signer',
  ],
};

// Layer 3: Third-party integrations
export const integrationsLayer = {
  name: 'eventfulhq-integrations',
  description: 'Third-party service integrations',
  compatibleRuntimes: ['nodejs18.x', 'nodejs20.x'],
  contents: [
    'whatsapp-business-api',
    'stripe',
    'sendgrid/mail',
    'twilio',
    'openai',
    'google-maps-services-js',
    'libphonenumber-js',
  ],
};

## Multi-Domain Regional Routing

### Regional Domain Configuration
EventfulHQ supports multiple regional domains that automatically redirect users based on their geographic location while respecting user preferences.

#### Domain Mapping
- **Global**: eventfulhq.com (default)
- **Americas**: eventfulamerica.com (USA, Canada, Mexico, South America)
- **Arabia**: eventfularabia.com (GCC Countries: UAE, Saudi Arabia, Kuwait, Qatar, Bahrain, Oman)
- **India**: eventfulindia.com (India, Bangladesh, Sri Lanka, Nepal)
- **Europe**: eventfuleurope.com (EU Countries + UK)
- **Australia**: eventfulaustralia.com (Australia, New Zealand)

### Regional Routing Middleware
```typescript
// middleware-regional.ts
import { NextRequest, NextResponse } from 'next/server';

interface RegionalConfig {
  domain: string;
  countries: string[];
  defaultLocale: string;
  supportedLocales: string[];
}

const regionalConfigs: Record<string, RegionalConfig> = {
  america: {
    domain: 'eventfulamerica.com',
    countries: ['US', 'CA', 'MX', 'BR', 'AR', 'CL', 'CO', 'PE'],
    defaultLocale: 'en',
    supportedLocales: ['en', 'es', 'pt'],
  },
  arabia: {
    domain: 'eventfularabia.com',
    countries: ['AE', 'SA', 'KW', 'QA', 'BH', 'OM', 'EG', 'JO', 'LB'],
    defaultLocale: 'ar',
    supportedLocales: ['ar', 'en'],
  },
  india: {
    domain: 'eventfulindia.com',
    countries: ['IN', 'BD', 'LK', 'NP', 'PK'],
    defaultLocale: 'hi',
    supportedLocales: ['hi', 'en'],
  },
  europe: {
    domain: 'eventfuleurope.com',
    countries: ['GB', 'FR', 'DE', 'IT', 'ES', 'NL', 'BE', 'PL', 'PT', 'SE', 'DK', 'FI', 'NO', 'IE', 'AT', 'CH'],
    defaultLocale: 'en',
    supportedLocales: ['en', 'fr', 'es', 'pt', 'ru'],
  },
  australia: {
    domain: 'eventfulaustralia.com',
    countries: ['AU', 'NZ'],
    defaultLocale: 'en',
    supportedLocales: ['en'],
  },
};

// Reverse mapping for quick lookup
const countryToRegion: Record<string, string> = {};
Object.entries(regionalConfigs).forEach(([region, config]) => {
  config.countries.forEach(country => {
    countryToRegion[country] = region;
  });
});

export async function regionalMiddleware(request: NextRequest) {
  const hostname = request.headers.get('host') || '';
  const pathname = request.nextUrl.pathname;
  
  // Get user's country from geo data
  const userCountry = request.geo?.country || 
                     request.headers.get('x-vercel-ip-country') || 
                     '';
  
  // Check if user has a saved regional preference
  const savedRegion = request.cookies.get('preferred-region')?.value;
  const savedDomain = request.cookies.get('preferred-domain')?.value;
  
  // If user has saved preferences, respect them
  if (savedDomain && hostname !== savedDomain) {
    const url = new URL(request.url);
    url.hostname = savedDomain;
    return NextResponse.redirect(url);
  }
  
  // Determine the appropriate region for the user
  const userRegion = countryToRegion[userCountry] || 'global';
  const targetConfig = regionalConfigs[userRegion];
  
  // If current domain doesn't match user's region and no saved preference
  if (!savedRegion && targetConfig && hostname !== targetConfig.domain && hostname === 'eventfulhq.com') {
    const url = new URL(request.url);
    url.hostname = targetConfig.domain;
    
    const response = NextResponse.redirect(url);
    
    // Save the auto-detected region (but as a soft preference)
    response.cookies.set('detected-region', userRegion, {
      maxAge: 60 * 60 * 24 * 30, // 30 days
      sameSite: 'lax',
      secure: process.env.NODE_ENV === 'production',
    });
    
    return response;
  }
  
  return NextResponse.next();
}
```

### Region Selector Component
```typescript
// components/features/regional/region-selector.tsx
'use client';

import { useState, useEffect } from 'react';
import { useRouter } from 'next/navigation';
import {
  Dialog,
  DialogContent,
  DialogDescription,
  DialogHeader,
  DialogTitle,
} from '@/components/ui/dialog';
import {
  Select,
  SelectContent,
  SelectItem,
  SelectTrigger,
  SelectValue,
} from '@/components/ui/select';
import { Button } from '@/components/ui/button';
import { GlobeIcon } from 'lucide-react';
import { toast } from 'sonner';

interface Region {
  id: string;
  name: string;
  domain: string;
  flag: string;
  description: string;
}

const regions: Region[] = [
  {
    id: 'global',
    name: 'Global',
    domain: 'eventfulhq.com',
    flag: '🌍',
    description: 'Access events worldwide',
  },
  {
    id: 'america',
    name: 'Americas',
    domain: 'eventfulamerica.com',
    flag: '🇺🇸',
    description: 'Events in North & South America',
  },
  {
    id: 'arabia',
    name: 'Arabia',
    domain: 'eventfularabia.com',
    flag: '🇸🇦',
    description: 'Events in GCC & Middle East',
  },
  {
    id: 'india',
    name: 'India',
    domain: 'eventfulindia.com',
    flag: '🇮🇳',
    description: 'Events in India & South Asia',
  },
  {
    id: 'europe',
    name: 'Europe',
    domain: 'eventfuleurope.com',
    flag: '🇪🇺',
    description: 'Events across Europe',
  },
  {
    id: 'australia',
    name: 'Australia',
    domain: 'eventfulaustralia.com',
    flag: '🇦🇺',
    description: 'Events in Australia & Oceania',
  },
];

export function RegionSelector() {
  const router = useRouter();
  const [showDialog, setShowDialog] = useState(false);
  const [selectedRegion, setSelectedRegion] = useState<string>('global');
  const [currentDomain, setCurrentDomain] = useState<string>('');

  useEffect(() => {
    // Get current domain
    const domain = window.location.hostname;
    setCurrentDomain(domain);
    
    // Find current region
    const currentRegion = regions.find(r => r.domain === domain);
    if (currentRegion) {
      setSelectedRegion(currentRegion.id);
    }
    
    // Check if this is first visit without preference
    const hasPreference = document.cookie.includes('preferred-region');
    const detectedRegion = document.cookie.includes('detected-region');
    
    if (!hasPreference && detectedRegion && domain === 'eventfulhq.com') {
      setShowDialog(true);
    }
  }, []);

  const handleRegionChange = (regionId: string) => {
    const region = regions.find(r => r.id === regionId);
    if (!region) return;
    
    // Save preference
    document.cookie = `preferred-region=${regionId};max-age=31536000;path=/;domain=.eventfulhq.com;samesite=lax${
      process.env.NODE_ENV === 'production' ? ';secure' : ''
    }`;
    document.cookie = `preferred-domain=${region.domain};max-age=31536000;path=/;domain=.eventfulhq.com;samesite=lax${
      process.env.NODE_ENV === 'production' ? ';secure' : ''
    }`;
    
    // Redirect to new domain
    if (region.domain !== currentDomain) {
      window.location.href = `https://${region.domain}${window.location.pathname}`;
    }
    
    toast.success(`Switched to ${region.name} region`);
  };

  const currentRegion = regions.find(r => r.id === selectedRegion);

  return (
    <>
      <Button
        variant="ghost"
        size="sm"
        onClick={() => setShowDialog(true)}
        className="flex items-center gap-2"
      >
        <GlobeIcon className="h-4 w-4" />
        <span className="hidden sm:inline">
          {currentRegion?.flag} {currentRegion?.name}
        </span>
        <span className="sm:hidden">{currentRegion?.flag}</span>
      </Button>

      <Dialog open={showDialog} onOpenChange={setShowDialog}>
        <DialogContent className="sm:max-w-md">
          <DialogHeader>
            <DialogTitle>Select Your Region</DialogTitle>
            <DialogDescription>
              Choose your preferred region to see relevant events and content
            </DialogDescription>
          </DialogHeader>
          
          <div className="space-y-4 py-4">
            <Select value={selectedRegion} onValueChange={setSelectedRegion}>
              <SelectTrigger>
                <SelectValue />
              </SelectTrigger>
              <SelectContent>
                {regions.map((region) => (
                  <SelectItem key={region.id} value={region.id}>
                    <div className="flex items-center gap-3">
                      <span className="text-2xl">{region.flag}</span>
                      <div>
                        <div className="font-medium">{region.name}</div>
                        <div className="text-xs text-muted-foreground">
                          {region.description}
                        </div>
                      </div>
                    </div>
                  </SelectItem>
                ))}
              </SelectContent>
            </Select>
            
            <div className="text-sm text-muted-foreground">
              Your selection will be saved and you'll be redirected to {regions.find(r => r.id === selectedRegion)?.domain}
            </div>
            
            <div className="flex justify-end gap-2">
              <Button
                variant="outline"
                onClick={() => setShowDialog(false)}
              >
                Cancel
              </Button>
              <Button
                onClick={() => {
                  handleRegionChange(selectedRegion);
                  setShowDialog(false);
                }}
              >
                Save & Continue
              </Button>
            </div>
          </div>
        </DialogContent>
      </Dialog>
    </>
  );
}
```

### Next.js Configuration for Multi-Domain
```javascript
// next.config.mjs
const domains = [
  'eventfulhq.com',
  'eventfulamerica.com',
  'eventfularabia.com',
  'eventfulindia.com',
  'eventfuleurope.com',
  'eventfulaustralia.com',
];

/** @type {import('next').NextConfig} */
const nextConfig = {
  experimental: {
    appDir: true,
  },
  images: {
    domains: [...domains, 'cdn.eventfulhq.com'],
    remotePatterns: [
      {
        protocol: 'https',
        hostname: '**.eventfulhq.com',
      },
    ],
  },
  async headers() {
    return [
      {
        source: '/:path*',
        headers: [
          {
            key: 'X-Frame-Options',
            value: 'DENY',
          },
          {
            key: 'Content-Security-Policy',
            value: `frame-ancestors 'none'; default-src 'self' ${domains.join(' ')};`,
          },
        ],
      },
    ];
  },
  async rewrites() {
    return {
      beforeFiles: [
        // API rewrites for regional domains
        ...domains.map((domain) => ({
          source: '/api/:path*',
          destination: `https://api.${domain}/api/:path*`,
          has: [
            {
              type: 'host',
              value: domain,
            },
          ],
        })),
      ],
    };
  },
  async redirects() {
    return [
      {
        source: '/events',
        destination: '/en/events',
        permanent: false,
      },
    ];
  },
};

export default nextConfig;
```

## SuperAdmin Access System

### Admin User Types (11 Types)
```typescript
// types/admin.types.ts
export enum AdminType {
  SUPER_ADMIN = 'SUPER_ADMIN',                 // Full system access
  PLATFORM_ADMIN = 'PLATFORM_ADMIN',           // Platform-wide management
  REGIONAL_ADMIN = 'REGIONAL_ADMIN',           // Regional management
  COUNTRY_ADMIN = 'COUNTRY_ADMIN',             // Country-level management
  CITY_ADMIN = 'CITY_ADMIN',                   // City-level management
  CATEGORY_ADMIN = 'CATEGORY_ADMIN',           // Event category management
  CONTENT_ADMIN = 'CONTENT_ADMIN',             // Content moderation
  SUPPORT_ADMIN = 'SUPPORT_ADMIN',             // Customer support
  FINANCE_ADMIN = 'FINANCE_ADMIN',             // Financial operations
  MARKETING_ADMIN = 'MARKETING_ADMIN',         // Marketing campaigns
  DEVELOPER_ADMIN = 'DEVELOPER_ADMIN',         // API and developer relations
}

export interface AdminUser {
  id: string;
  email: string;
  phoneNumber: string;
  name: string;
  type: AdminType;
  permissions: Permission[];
  geographyAccess: GeographyAccess[];
  featureAccess: FeatureAccess[];
  isActive: boolean;
  isTwoFactorEnabled: boolean;
  lastLogin: Date;
  createdAt: Date;
  createdBy: string;
}

export interface Permission {
  resource: string;      // Microservice name as AppFeature
  actions: Action[];     // Read, Write, Delete, etc.
  conditions?: Record<string, any>;
}

export interface GeographyAccess {
  type: 'GLOBAL' | 'REGION' | 'COUNTRY' | 'CITY';
  identifier: string;    // e.g., 'US', 'New York', 'ASIA'
  permissions: Action[];
}

export interface FeatureAccess {
  microservice: string;  // All 30+ microservices
  permissions: Action[];
  limits?: Record<string, number>;
}

export enum Action {
  READ = 'READ',
  WRITE = 'WRITE',
  UPDATE = 'UPDATE',
  DELETE = 'DELETE',
  PUBLISH = 'PUBLISH',
  EXPORT = 'EXPORT',
  CONFIGURE = 'CONFIGURE',
}
```

### SuperAdmin Dashboard Components
```typescript
// components/features/admin/super-admin-dashboard.tsx
'use client';

import { useState } from 'react';
import { useTranslations } from 'next-intl';
import { Tabs, TabsContent, TabsList, TabsTrigger } from '@/components/ui/tabs';
import { Card, CardContent, CardDescription, CardHeader, CardTitle } from '@/components/ui/card';
import { AdminManagement } from './admin-management';
import { FeatureControl } from './feature-control';
import { GeographyControl } from './geography-control';
import { PermissionMatrix } from './permission-matrix';
import { SystemConfiguration } from './system-configuration';
import { AuditLogs } from './audit-logs';
import { SystemMonitoring } from './system-monitoring';
import { Shield, Users, Globe, Settings, Activity, FileText, Map } from 'lucide-react';

export function SuperAdminDashboard() {
  const t = useTranslations('admin');
  const [activeTab, setActiveTab] = useState('admins');

  return (
    <div className="space-y-6">
      <div>
        <h1 className="text-3xl font-bold">{t('superAdmin.title')}</h1>
        <p className="text-muted-foreground">{t('superAdmin.description')}</p>
      </div>

      <Tabs value={activeTab} onValueChange={setActiveTab} className="space-y-4">
        <TabsList className="grid w-full grid-cols-7">
          <TabsTrigger value="admins" className="flex items-center gap-2">
            <Users className="h-4 w-4" />
            <span className="hidden sm:inline">{t('tabs.admins')}</span>
          </TabsTrigger>
          <TabsTrigger value="features" className="flex items-center gap-2">
            <Shield className="h-4 w-4" />
            <span className="hidden sm:inline">{t('tabs.features')}</span>
          </TabsTrigger>
          <TabsTrigger value="geography" className="flex items-center gap-2">
            <Globe className="h-4 w-4" />
            <span className="hidden sm:inline">{t('tabs.geography')}</span>
          </TabsTrigger>
          <TabsTrigger value="permissions" className="flex items-center gap-2">
            <Map className="h-4 w-4" />
            <span className="hidden sm:inline">{t('tabs.permissions')}</span>
          </TabsTrigger>
          <TabsTrigger value="config" className="flex items-center gap-2">
            <Settings className="h-4 w-4" />
            <span className="hidden sm:inline">{t('tabs.config')}</span>
          </TabsTrigger>
          <TabsTrigger value="monitoring" className="flex items-center gap-2">
            <Activity className="h-4 w-4" />
            <span className="hidden sm:inline">{t('tabs.monitoring')}</span>
          </TabsTrigger>
          <TabsTrigger value="audit" className="flex items-center gap-2">
            <FileText className="h-4 w-4" />
            <span className="hidden sm:inline">{t('tabs.audit')}</span>
          </TabsTrigger>
        </TabsList>

        <TabsContent value="admins" className="space-y-4">
          <AdminManagement />
        </TabsContent>

        <TabsContent value="features" className="space-y-4">
          <FeatureControl />
        </TabsContent>

        <TabsContent value="geography" className="space-y-4">
          <GeographyControl />
        </TabsContent>

        <TabsContent value="permissions" className="space-y-4">
          <PermissionMatrix />
        </TabsContent>

        <TabsContent value="config" className="space-y-4">
          <SystemConfiguration />
        </TabsContent>

        <TabsContent value="monitoring" className="space-y-4">
          <SystemMonitoring />
        </TabsContent>

        <TabsContent value="audit" className="space-y-4">
          <AuditLogs />
        </TabsContent>
      </Tabs>
    </div>
  );
}
```

### Admin Management Interface
```typescript
// components/features/admin/admin-management.tsx
'use client';

import { useState } from 'react';
import { useQuery, useMutation } from '@tanstack/react-query';
import { DataTable } from '@/components/ui/data-table';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import {
  Select,
  SelectContent,
  SelectItem,
  SelectTrigger,
  SelectValue,
} from '@/components/ui/select';
import { Badge } from '@/components/ui/badge';
import {
  Dialog,
  DialogContent,
  DialogDescription,
  DialogHeader,
  DialogTitle,
  DialogTrigger,
} from '@/components/ui/dialog';
import { AdminType } from '@/types/admin.types';
import { adminService } from '@/services/admin-service';
import { CreateAdminForm } from './create-admin-form';
import { PermissionEditor } from './permission-editor';
import { Plus, Edit, Trash, Shield, Key } from 'lucide-react';
import { toast } from 'sonner';

export function AdminManagement() {
  const [search, setSearch] = useState('');
  const [typeFilter, setTypeFilter] = useState<AdminType | 'ALL'>('ALL');
  const [showCreateDialog, setShowCreateDialog] = useState(false);
  const [selectedAdmin, setSelectedAdmin] = useState(null);

  const { data: admins, isLoading } = useQuery({
    queryKey: ['admins', search, typeFilter],
    queryFn: () => adminService.getAdmins({ search, type: typeFilter }),
  });

  const { mutate: deleteAdmin } = useMutation({
    mutationFn: adminService.deleteAdmin,
    onSuccess: () => {
      toast.success('Admin deleted successfully');
      queryClient.invalidateQueries({ queryKey: ['admins'] });
    },
  });

  const { mutate: toggleTwoFactor } = useMutation({
    mutationFn: adminService.toggleTwoFactor,
    onSuccess: () => {
      toast.success('Two-factor authentication updated');
      queryClient.invalidateQueries({ queryKey: ['admins'] });
    },
  });

  const columns = [
    {
      accessorKey: 'name',
      header: 'Name',
      cell: ({ row }) => (
        <div>
          <p className="font-medium">{row.original.name}</p>
          <p className="text-sm text-muted-foreground">{row.original.email}</p>
        </div>
      ),
    },
    {
      accessorKey: 'type',
      header: 'Admin Type',
      cell: ({ row }) => (
        <Badge variant={row.original.type === 'SUPER_ADMIN' ? 'destructive' : 'default'}>
          {row.original.type.replace('_', ' ')}
        </Badge>
      ),
    },
    {
      accessorKey: 'geographyAccess',
      header: 'Geography Access',
      cell: ({ row }) => {
        const access = row.original.geographyAccess;
        if (access.some(a => a.type === 'GLOBAL')) {
          return <Badge variant="outline">Global</Badge>;
        }
        return (
          <div className="flex flex-wrap gap-1">
            {access.slice(0, 3).map((geo, idx) => (
              <Badge key={idx} variant="secondary" className="text-xs">
                {geo.identifier}
              </Badge>
            ))}
            {access.length > 3 && (
              <Badge variant="secondary" className="text-xs">
                +{access.length - 3}
              </Badge>
            )}
          </div>
        );
      },
    },
    {
      accessorKey: 'featureAccess',
      header: 'Features',
      cell: ({ row }) => {
        const features = row.original.featureAccess;
        return (
          <div className="flex flex-wrap gap-1">
            {features.slice(0, 3).map((feature, idx) => (
              <Badge key={idx} variant="outline" className="text-xs">
                {feature.microservice}
              </Badge>
            ))}
            {features.length > 3 && (
              <Badge variant="outline" className="text-xs">
                +{features.length - 3}
              </Badge>
            )}
          </div>
        );
      },
    },
    {
      accessorKey: 'status',
      header: 'Status',
      cell: ({ row }) => (
        <div className="flex items-center gap-2">
          <Badge variant={row.original.isActive ? 'success' : 'destructive'}>
            {row.original.isActive ? 'Active' : 'Inactive'}
          </Badge>
          {row.original.isTwoFactorEnabled && (
            <Shield className="h-4 w-4 text-green-500" />
          )}
        </div>
      ),
    },
    {
      accessorKey: 'lastLogin',
      header: 'Last Login',
      cell: ({ row }) => (
        <span className="text-sm text-muted-foreground">
          {new Date(row.original.lastLogin).toLocaleDateString()}
        </span>
      ),
    },
    {
      id: 'actions',
      cell: ({ row }) => (
        <div className="flex items-center gap-2">
          <Button
            variant="ghost"
            size="icon"
            onClick={() => setSelectedAdmin(row.original)}
          >
            <Edit className="h-4 w-4" />
          </Button>
          <Button
            variant="ghost"
            size="icon"
            onClick={() => toggleTwoFactor(row.original.id)}
          >
            <Key className="h-4 w-4" />
          </Button>
          <Button
            variant="ghost"
            size="icon"
            onClick={() => deleteAdmin(row.original.id)}
            disabled={row.original.type === 'SUPER_ADMIN'}
          >
            <Trash className="h-4 w-4" />
          </Button>
        </div>
      ),
    },
  ];

  return (
    <div className="space-y-4">
      <div className="flex items-center justify-between">
        <div className="flex items-center gap-4">
          <Input
            placeholder="Search admins..."
            value={search}
            onChange={(e) => setSearch(e.target.value)}
            className="w-64"
          />
          <Select value={typeFilter} onValueChange={setTypeFilter}>
            <SelectTrigger className="w-48">
              <SelectValue />
            </SelectTrigger>
            <SelectContent>
              <SelectItem value="ALL">All Types</SelectItem>
              {Object.values(AdminType).map((type) => (
                <SelectItem key={type} value={type}>
                  {type.replace(/_/g, ' ')}
                </SelectItem>
              ))}
            </SelectContent>
          </Select>
        </div>
        <Dialog open={showCreateDialog} onOpenChange={setShowCreateDialog}>
          <DialogTrigger asChild>
            <Button>
              <Plus className="h-4 w-4 mr-2" />
              Add Admin
            </Button>
          </DialogTrigger>
          <DialogContent className="max-w-2xl">
            <DialogHeader>
              <DialogTitle>Create New Admin</DialogTitle>
              <DialogDescription>
                Add a new admin user with specific permissions and access controls
              </DialogDescription>
            </DialogHeader>
            <CreateAdminForm onSuccess={() => setShowCreateDialog(false)} />
          </DialogContent>
        </Dialog>
      </div>

      <DataTable
        columns={columns}
        data={admins?.data || []}
        loading={isLoading}
      />

      {selectedAdmin && (
        <PermissionEditor
          admin={selectedAdmin}
          open={!!selectedAdmin}
          onOpenChange={() => setSelectedAdmin(null)}
        />
      )}
    </div>
  );
}
```

### Feature Control (Microservices as AppFeatures)
```typescript
// components/features/admin/feature-control.tsx
'use client';

import { useState } from 'react';
import { useQuery, useMutation } from '@tanstack/react-query';
import { Card, CardContent, CardDescription, CardHeader, CardTitle } from '@/components/ui/card';
import { Switch } from '@/components/ui/switch';
import { Button } from '@/components/ui/button';
import { Badge } from '@/components/ui/badge';
import { Slider } from '@/components/ui/slider';
import { Input } from '@/components/ui/input';
import { Label } from '@/components/ui/label';
import {
  Accordion,
  AccordionContent,
  AccordionItem,
  AccordionTrigger,
} from '@/components/ui/accordion';
import { adminService } from '@/services/admin-service';
import { Microservice } from '@/types/admin.types';
import { Settings, AlertCircle, Activity } from 'lucide-react';
import { toast } from 'sonner';

// All 30+ microservices
const MICROSERVICES = [
  { id: 'event-service', name: 'Event Service', category: 'Core' },
  { id: 'user-service', name: 'User Service', category: 'Core' },
  { id: 'whatsapp-service', name: 'WhatsApp Service', category: 'Communication' },
  { id: 'rfp-service', name: 'RFP Service', category: 'Marketplace' },
  { id: 'analytics-service', name: 'Analytics Service', category: 'Analytics' },
  { id: 'payment-service', name: 'Payment Service', category: 'Financial' },
  { id: 'notification-service', name: 'Notification Service', category: 'Communication' },
  { id: 'search-service', name: 'Search Service', category: 'Core' },
  { id: 'recommendation-service', name: 'Recommendation Service', category: 'AI' },
  { id: 'media-service', name: 'Media Service', category: 'Content' },
  { id: 'chat-service', name: 'Chat Service', category: 'Communication' },
  { id: 'booking-service', name: 'Booking Service', category: 'Core' },
  { id: 'review-service', name: 'Review Service', category: 'Social' },
  { id: 'social-service', name: 'Social Service', category: 'Social' },
  { id: 'email-service', name: 'Email Service', category: 'Communication' },
  { id: 'sms-service', name: 'SMS Service', category: 'Communication' },
  { id: 'calendar-service', name: 'Calendar Service', category: 'Core' },
  { id: 'ticketing-service', name: 'Ticketing Service', category: 'Core' },
  { id: 'venue-service', name: 'Venue Service', category: 'Marketplace' },
  { id: 'artist-service', name: 'Artist Service', category: 'Marketplace' },
  { id: 'sponsor-service', name: 'Sponsor Service', category: 'Marketplace' },
  { id: 'streaming-service', name: 'Streaming Service', category: 'Content' },
  { id: 'translation-service', name: 'Translation Service', category: 'AI' },
  { id: 'moderation-service', name: 'Moderation Service', category: 'AI' },
  { id: 'fraud-service', name: 'Fraud Detection', category: 'Security' },
  { id: 'audit-service', name: 'Audit Service', category: 'Security' },
  { id: 'backup-service', name: 'Backup Service', category: 'Infrastructure' },
  { id: 'monitoring-service', name: 'Monitoring Service', category: 'Infrastructure' },
  { id: 'queue-service', name: 'Queue Service', category: 'Infrastructure' },
  { id: 'cache-service', name: 'Cache Service', category: 'Infrastructure' },
];

export function FeatureControl() {
  const [selectedCategory, setSelectedCategory] = useState<string>('all');
  
  const { data: features, isLoading } = useQuery({
    queryKey: ['features'],
    queryFn: adminService.getFeatures,
  });

  const { mutate: updateFeature } = useMutation({
    mutationFn: adminService.updateFeature,
    onSuccess: () => {
      toast.success('Feature updated successfully');
      queryClient.invalidateQueries({ queryKey: ['features'] });
    },
  });

  const categories = ['all', ...new Set(MICROSERVICES.map(m => m.category))];
  const filteredServices = selectedCategory === 'all' 
    ? MICROSERVICES 
    : MICROSERVICES.filter(m => m.category === selectedCategory);

  return (
    <div className="space-y-6">
      <div className="flex items-center gap-4">
        {categories.map((category) => (
          <Button
            key={category}
            variant={selectedCategory === category ? 'default' : 'outline'}
            size="sm"
            onClick={() => setSelectedCategory(category)}
          >
            {category.charAt(0).toUpperCase() + category.slice(1)}
          </Button>
        ))}
      </div>

      <div className="grid gap-4 md:grid-cols-2 lg:grid-cols-3">
        {filteredServices.map((service) => {
          const feature = features?.find(f => f.id === service.id);
          
          return (
            <Card key={service.id}>
              <CardHeader className="pb-3">
                <div className="flex items-center justify-between">
                  <CardTitle className="text-base">{service.name}</CardTitle>
                  <Switch
                    checked={feature?.isEnabled || false}
                    onCheckedChange={(enabled) => 
                      updateFeature({ id: service.id, isEnabled: enabled })
                    }
                  />
                </div>
                <CardDescription className="text-xs">
                  {service.category} • {service.id}
                </CardDescription>
              </CardHeader>
              <CardContent className="space-y-4">
                <div className="flex items-center justify-between text-sm">
                  <span className="text-muted-foreground">Status</span>
                  <Badge variant={feature?.status === 'healthy' ? 'success' : 'destructive'}>
                    {feature?.status || 'unknown'}
                  </Badge>
                </div>
                
                <div className="space-y-2">
                  <Label className="text-xs">Rate Limit (req/min)</Label>
                  <div className="flex items-center gap-2">
                    <Slider
                      value={[feature?.rateLimit || 1000]}
                      onValueChange={([value]) => 
                        updateFeature({ id: service.id, rateLimit: value })
                      }
                      max={10000}
                      step={100}
                      className="flex-1"
                    />
                    <span className="text-xs w-12 text-right">
                      {feature?.rateLimit || 1000}
                    </span>
                  </div>
                </div>

                <Accordion type="single" collapsible>
                  <AccordionItem value="config">
                    <AccordionTrigger className="text-xs">
                      Advanced Configuration
                    </AccordionTrigger>
                    <AccordionContent className="space-y-3">
                      <div className="space-y-2">
                        <Label className="text-xs">Timeout (ms)</Label>
                        <Input
                          type="number"
                          value={feature?.timeout || 30000}
                          onChange={(e) => 
                            updateFeature({ 
                              id: service.id, 
                              timeout: parseInt(e.target.value) 
                            })
                          }
                        />
                      </div>
                      
                      <div className="space-y-2">
                        <Label className="text-xs">Max Retries</Label>
                        <Input
                          type="number"
                          value={feature?.maxRetries || 3}
                          onChange={(e) => 
                            updateFeature({ 
                              id: service.id, 
                              maxRetries: parseInt(e.target.value) 
                            })
                          }
                        />
                      </div>

                      <div className="space-y-2">
                        <Label className="text-xs">Feature Flags</Label>
                        <div className="space-y-1">
                          {feature?.flags?.map((flag) => (
                            <div key={flag.name} className="flex items-center justify-between">
                              <span className="text-xs">{flag.name}</span>
                              <Switch
                                checked={flag.enabled}
                                onCheckedChange={(enabled) => 
                                  updateFeature({ 
                                    id: service.id, 
                                    flags: feature.flags.map(f => 
                                      f.name === flag.name ? { ...f, enabled } : f
                                    )
                                  })
                                }
                              />
                            </div>
                          ))}
                        </div>
                      </div>
                    </AccordionContent>
                  </AccordionItem>
                </Accordion>
              </CardContent>
            </Card>
          );
        })}
      </div>
    </div>
  );
}
```

### Permission Matrix Component
```typescript
// components/features/admin/permission-matrix.tsx
'use client';

import { useState } from 'react';
import { useQuery } from '@tanstack/react-query';
import {
  Table,
  TableBody,
  TableCell,
  TableHead,
  TableHeader,
  TableRow,
} from '@/components/ui/table';
import { Checkbox } from '@/components/ui/checkbox';
import { Badge } from '@/components/ui/badge';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import {
  Select,
  SelectContent,
  SelectItem,
  SelectTrigger,
  SelectValue,
} from '@/components/ui/select';
import { adminService } from '@/services/admin-service';
import { AdminType, Action } from '@/types/admin.types';
import { Save, Download } from 'lucide-react';
import { toast } from 'sonner';

export function PermissionMatrix() {
  const [search, setSearch] = useState('');
  const [selectedAdminType, setSelectedAdminType] = useState<AdminType>(AdminType.PLATFORM_ADMIN);
  const [modifiedPermissions, setModifiedPermissions] = useState<Record<string, any>>({});

  const { data: permissions, isLoading } = useQuery({
    queryKey: ['permission-matrix', selectedAdminType],
    queryFn: () => adminService.getPermissionMatrix(selectedAdminType),
  });

  const handlePermissionChange = (feature: string, action: Action, enabled: boolean) => {
    setModifiedPermissions(prev => ({
      ...prev,
      [`${feature}-${action}`]: enabled,
    }));
  };

  const handleSave = async () => {
    try {
      await adminService.updatePermissionMatrix(selectedAdminType, modifiedPermissions);
      toast.success('Permissions updated successfully');
      setModifiedPermissions({});
    } catch (error) {
      toast.error('Failed to update permissions');
    }
  };

  const handleExport = () => {
    const data = permissions?.map(p => ({
      Feature: p.feature,
      ...Object.fromEntries(
        Object.values(Action).map(action => [
          action,
          p.permissions.includes(action) ? 'Yes' : 'No'
        ])
      ),
    }));
    
    const csv = [
      Object.keys(data[0]).join(','),
      ...data.map(row => Object.values(row).join(','))
    ].join('\n');
    
    const blob = new Blob([csv], { type: 'text/csv' });
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = `permissions-${selectedAdminType}.csv`;
    a.click();
  };

  const filteredPermissions = permissions?.filter(p => 
    p.feature.toLowerCase().includes(search.toLowerCase())
  );

  return (
    <div className="space-y-4">
      <div className="flex items-center justify-between">
        <div className="flex items-center gap-4">
          <Input
            placeholder="Search features..."
            value={search}
            onChange={(e) => setSearch(e.target.value)}
            className="w-64"
          />
          <Select value={selectedAdminType} onValueChange={setSelectedAdminType}>
            <SelectTrigger className="w-48">
              <SelectValue />
            </SelectTrigger>
            <SelectContent>
              {Object.values(AdminType).map((type) => (
                <SelectItem key={type} value={type}>
                  {type.replace(/_/g, ' ')}
                </SelectItem>
              ))}
            </SelectContent>
          </Select>
        </div>
        <div className="flex items-center gap-2">
          <Button
            variant="outline"
            size="sm"
            onClick={handleExport}
          >
            <Download className="h-4 w-4 mr-2" />
            Export CSV
          </Button>
          <Button
            size="sm"
            onClick={handleSave}
            disabled={Object.keys(modifiedPermissions).length === 0}
          >
            <Save className="h-4 w-4 mr-2" />
            Save Changes
          </Button>
        </div>
      </div>

      <div className="border rounded-lg">
        <Table>
          <TableHeader>
            <TableRow>
              <TableHead className="w-64">Feature / Microservice</TableHead>
              {Object.values(Action).map((action) => (
                <TableHead key={action} className="text-center">
                  {action}
                </TableHead>
              ))}
            </TableRow>
          </TableHeader>
          <TableBody>
            {filteredPermissions?.map((permission) => (
              <TableRow key={permission.feature}>
                <TableCell className="font-medium">
                  <div>
                    <p>{permission.featureName}</p>
                    <p className="text-xs text-muted-foreground">{permission.feature}</p>
                  </div>
                </TableCell>
                {Object.values(Action).map((action) => {
                  const key = `${permission.feature}-${action}`;
                  const isEnabled = modifiedPermissions[key] ?? 
                    permission.permissions.includes(action);
                  const isModified = key in modifiedPermissions;
                  
                  return (
                    <TableCell key={action} className="text-center">
                      <div className="flex items-center justify-center">
                        <Checkbox
                          checked={isEnabled}
                          onCheckedChange={(checked) => 
                            handlePermissionChange(permission.feature, action, !!checked)
                          }
                        />
                        {isModified && (
                          <Badge variant="outline" className="ml-2 text-xs">
                            Modified
                          </Badge>
                        )}
                      </div>
                    </TableCell>
                  );
                })}
              </TableRow>
            ))}
          </TableBody>
        </Table>
      </div>
    </div>
  );
}
```

## Developer Portal & API Documentation

### Developer Portal Homepage
```typescript
// app/[locale]/(developers)/developers/page.tsx
import { Metadata } from 'next';
import { DeveloperDashboard } from '@/components/features/developers/developer-dashboard';
import { APIQuickStart } from '@/components/features/developers/api-quickstart';
import { APIStats } from '@/components/features/developers/api-stats';

export const metadata: Metadata = {
  title: 'EventfulHQ Developer Portal',
  description: 'Build amazing applications with EventfulHQ APIs',
};

export default function DevelopersPage() {
  return (
    <div className="container py-8 space-y-8">
      <div className="text-center space-y-4">
        <h1 className="text-4xl font-bold">EventfulHQ Developer Portal</h1>
        <p className="text-xl text-muted-foreground max-w-2xl mx-auto">
          Access our comprehensive APIs to build innovative event management solutions
        </p>
      </div>

      <APIStats />
      <APIQuickStart />
      <DeveloperDashboard />
    </div>
  );
}
```

### API Documentation Component
```typescript
// components/features/developers/api-documentation.tsx
'use client';

import { useState, useEffect } from 'react';
import dynamic from 'next/dynamic';
import { Tabs, TabsContent, TabsList, TabsTrigger } from '@/components/ui/tabs';
import { Card, CardContent, CardDescription, CardHeader, CardTitle } from '@/components/ui/card';
import { Button } from '@/components/ui/button';
import { Badge } from '@/components/ui/badge';
import { CodeBlock } from './code-block';
import { APIPlayground } from './api-playground';
import { useQuery } from '@tanstack/react-query';
import { developerService } from '@/services/developer-service';
import { Book, Code, Play, Download } from 'lucide-react';

// Dynamic import for Swagger UI
const SwaggerUI = dynamic(() => import('swagger-ui-react'), { 
  ssr: false,
  loading: () => <div>Loading API documentation...</div>
});

// Import Swagger UI CSS
import 'swagger-ui-react/swagger-ui.css';

export function APIDocumentation() {
  const [selectedService, setSelectedService] = useState('event-service');
  const [selectedLanguage, setSelectedLanguage] = useState('javascript');

  const { data: apiSpec } = useQuery({
    queryKey: ['api-spec', selectedService],
    queryFn: () => developerService.getAPISpec(selectedService),
  });

  const { data: sdks } = useQuery({
    queryKey: ['sdks'],
    queryFn: developerService.getSDKs,
  });

  const services = [
    { id: 'event-service', name: 'Event API', version: 'v1' },
    { id: 'user-service', name: 'User API', version: 'v1' },
    { id: 'rfp-service', name: 'RFP API', version: 'v1' },
    { id: 'analytics-service', name: 'Analytics API', version: 'v1' },
    { id: 'whatsapp-service', name: 'WhatsApp API', version: 'v1' },
    // ... other services
  ];

  const codeSamples = {
    javascript: `// Install the SDK
npm install @eventfulhq/sdk

// Initialize the client
import { EventfulHQ } from '@eventfulhq/sdk';

const client = new EventfulHQ({
  apiKey: 'your-api-key',
  region: 'global' // or 'america', 'arabia', 'india', 'europe', 'australia'
});

// Create an event
const event = await client.events.create({
  title: 'Summer Music Festival',
  description: 'A fantastic outdoor music experience',
  date: '2025-07-15T18:00:00Z',
  venue: {
    name: 'Central Park',
    address: 'New York, NY',
    coordinates: { lat: 40.7829, lng: -73.9654 }
  },
  capacity: 5000,
  price: 75,
  currency: 'USD'
});

// Send WhatsApp invitations
await client.whatsapp.sendInvitations({
  eventId: event.id,
  phoneNumbers: ['+1234567890', '+0987654321'],
  message: 'You are invited to the Summer Music Festival!'
});`,
    python: `# Install the SDK
pip install eventfulhq

# Initialize the client
from eventfulhq import EventfulHQ

client = EventfulHQ(
    api_key='your-api-key',
    region='global'  # or 'america', 'arabia', 'india', 'europe', 'australia'
)

# Create an event
event = client.events.create(
    title='Summer Music Festival',
    description='A fantastic outdoor music experience',
    date='2025-07-15T18:00:00Z',
    venue={
        'name': 'Central Park',
        'address': 'New York, NY',
        'coordinates': {'lat': 40.7829, 'lng': -73.9654}
    },
    capacity=5000,
    price=75,
    currency='USD'
)

# Send WhatsApp invitations
client.whatsapp.send_invitations(
    event_id=event.id,
    phone_numbers=['+1234567890', '+0987654321'],
    message='You are invited to the Summer Music Festival!'
)`,
    curl: `# Get your API key from the developer portal
API_KEY="your-api-key"

# Create an event
curl -X POST https://api.eventfulhq.com/v1/events \\
  -H "Authorization: Bearer $API_KEY" \\
  -H "Content-Type: application/json" \\
  -d '{
    "title": "Summer Music Festival",
    "description": "A fantastic outdoor music experience",
    "date": "2025-07-15T18:00:00Z",
    "venue": {
      "name": "Central Park",
      "address": "New York, NY",
      "coordinates": {"lat": 40.7829, "lng": -73.9654}
    },
    "capacity": 5000,
    "price": 75,
    "currency": "USD"
  }'

# Send WhatsApp invitations
curl -X POST https://api.eventfulhq.com/v1/events/{eventId}/whatsapp-invite \\
  -H "Authorization: Bearer $API_KEY" \\
  -H "Content-Type: application/json" \\
  -d '{
    "phoneNumbers": ["+1234567890", "+0987654321"],
    "message": "You are invited to the Summer Music Festival!"
  }'`,
  };

  return (
    <div className="space-y-6">
      <div className="flex items-center justify-between">
        <div className="flex items-center gap-4">
          <h2 className="text-2xl font-bold">API Documentation</h2>
          <Badge variant="outline">API v1</Badge>
        </div>
        <div className="flex items-center gap-2">
          <Button variant="outline" size="sm">
            <Download className="h-4 w-4 mr-2" />
            Download OpenAPI Spec
          </Button>
          <Button variant="outline" size="sm">
            <Book className="h-4 w-4 mr-2" />
            API Changelog
          </Button>
        </div>
      </div>

      <Tabs defaultValue="reference" className="space-y-4">
        <TabsList className="grid w-full grid-cols-4">
          <TabsTrigger value="quickstart">Quick Start</TabsTrigger>
          <TabsTrigger value="reference">API Reference</TabsTrigger>
          <TabsTrigger value="playground">Playground</TabsTrigger>
          <TabsTrigger value="sdks">SDKs</TabsTrigger>
        </TabsList>

        <TabsContent value="quickstart" className="space-y-4">
          <Card>
            <CardHeader>
              <CardTitle>Getting Started</CardTitle>
              <CardDescription>
                Start building with EventfulHQ APIs in minutes
              </CardDescription>
            </CardHeader>
            <CardContent className="space-y-4">
              <div className="space-y-2">
                <h3 className="font-semibold">1. Get your API key</h3>
                <p className="text-sm text-muted-foreground">
                  Sign in to the developer portal and create a new API key from the dashboard
                </p>
              </div>

              <div className="space-y-2">
                <h3 className="font-semibold">2. Choose your integration method</h3>
                <div className="flex gap-2">
                  {['javascript', 'python', 'curl'].map((lang) => (
                    <Button
                      key={lang}
                      variant={selectedLanguage === lang ? 'default' : 'outline'}
                      size="sm"
                      onClick={() => setSelectedLanguage(lang)}
                    >
                      {lang.charAt(0).toUpperCase() + lang.slice(1)}
                    </Button>
                  ))}
                </div>
              </div>

              <div className="space-y-2">
                <h3 className="font-semibold">3. Make your first API call</h3>
                <CodeBlock
                  language={selectedLanguage}
                  code={codeSamples[selectedLanguage]}
                />
              </div>
            </CardContent>
          </Card>
        </TabsContent>

        <TabsContent value="reference" className="space-y-4">
          <div className="flex gap-4">
            <div className="w-64 space-y-2">
              <h3 className="font-semibold text-sm">Services</h3>
              {services.map((service) => (
                <Button
                  key={service.id}
                  variant={selectedService === service.id ? 'secondary' : 'ghost'}
                  size="sm"
                  className="w-full justify-start"
                  onClick={() => setSelectedService(service.id)}
                >
                  {service.name}
                  <Badge variant="outline" className="ml-auto">
                    {service.version}
                  </Badge>
                </Button>
              ))}
            </div>
            <div className="flex-1">
              <Card>
                <CardContent className="p-0">
                  {apiSpec && (
                    <SwaggerUI
                      spec={apiSpec}
                      docExpansion="list"
                      defaultModelsExpandDepth={1}
                      tryItOutEnabled={true}
                    />
                  )}
                </CardContent>
              </Card>
            </div>
          </div>
        </TabsContent>

        <TabsContent value="playground" className="space-y-4">
          <APIPlayground />
        </TabsContent>

        <TabsContent value="sdks" className="space-y-4">
          <div className="grid gap-4 md:grid-cols-2 lg:grid-cols-3">
            {sdks?.map((sdk) => (
              <Card key={sdk.language}>
                <CardHeader>
                  <CardTitle className="flex items-center justify-between">
                    {sdk.name}
                    <Badge>{sdk.version}</Badge>
                  </CardTitle>
                  <CardDescription>{sdk.description}</CardDescription>
                </CardHeader>
                <CardContent className="space-y-4">
                  <CodeBlock
                    language="bash"
                    code={sdk.installCommand}
                  />
                  <div className="flex gap-2">
                    <Button variant="outline" size="sm" asChild>
                      <a href={sdk.githubUrl} target="_blank" rel="noopener">
                        <Code className="h-4 w-4 mr-2" />
                        GitHub
                      </a>
                    </Button>
                    <Button variant="outline" size="sm" asChild>
                      <a href={sdk.docsUrl} target="_blank" rel="noopener">
                        <Book className="h-4 w-4 mr-2" />
                        Docs
                      </a>
                    </Button>
                  </div>
                </CardContent>
              </Card>
            ))}
          </div>
        </TabsContent>
      </Tabs>
    </div>
  );
}
```

### API Key Management
```typescript
// components/features/developers/api-key-manager.tsx
'use client';

import { useState } from 'react';
import { useQuery, useMutation } from '@tanstack/react-query';
import { Card, CardContent, CardDescription, CardHeader, CardTitle } from '@/components/ui/card';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { Label } from '@/components/ui/label';
import { Badge } from '@/components/ui/badge';
import {
  Dialog,
  DialogContent,
  DialogDescription,
  DialogHeader,
  DialogTitle,
  DialogTrigger,
} from '@/components/ui/dialog';
import {
  Table,
  TableBody,
  TableCell,
  TableHead,
  TableHeader,
  TableRow,
} from '@/components/ui/table';
import { developerService } from '@/services/developer-service';
import { Copy, Eye, EyeOff, Key, Trash, Plus } from 'lucide-react';
import { toast } from 'sonner';

export function APIKeyManager() {
  const [showCreateDialog, setShowCreateDialog] = useState(false);
  const [newKeyName, setNewKeyName] = useState('');
  const [selectedScopes, setSelectedScopes] = useState<string[]>([]);
  const [visibleKeys, setVisibleKeys] = useState<Set<string>>(new Set());

  const { data: apiKeys, isLoading } = useQuery({
    queryKey: ['api-keys'],
    queryFn: developerService.getAPIKeys,
  });

  const { mutate: createKey } = useMutation({
    mutationFn: developerService.createAPIKey,
    onSuccess: (data) => {
      toast.success('API key created successfully');
      // Show the key once for copying
      navigator.clipboard.writeText(data.key);
      toast.info('API key copied to clipboard. This is the only time you can see it!');
      setShowCreateDialog(false);
      setNewKeyName('');
      setSelectedScopes([]);
      queryClient.invalidateQueries({ queryKey: ['api-keys'] });
    },
  });

  const { mutate: deleteKey } = useMutation({
    mutationFn: developerService.deleteAPIKey,
    onSuccess: () => {
      toast.success('API key deleted successfully');
      queryClient.invalidateQueries({ queryKey: ['api-keys'] });
    },
  });

  const scopes = [
    { id: 'events:read', name: 'Read Events' },
    { id: 'events:write', name: 'Write Events' },
    { id: 'users:read', name: 'Read Users' },
    { id: 'users:write', name: 'Write Users' },
    { id: 'analytics:read', name: 'Read Analytics' },
    { id: 'whatsapp:send', name: 'Send WhatsApp Messages' },
    { id: 'rfp:manage', name: 'Manage RFPs' },
    { id: 'payments:process', name: 'Process Payments' },
  ];

  const toggleKeyVisibility = (keyId: string) => {
    setVisibleKeys(prev => {
      const newSet = new Set(prev);
      if (newSet.has(keyId)) {
        newSet.delete(keyId);
      } else {
        newSet.add(keyId);
      }
      return newSet;
    });
  };

  return (
    <Card>
      <CardHeader>
        <div className="flex items-center justify-between">
          <div>
            <CardTitle>API Keys</CardTitle>
            <CardDescription>
              Manage your API keys for accessing EventfulHQ APIs
            </CardDescription>
          </div>
          <Dialog open={showCreateDialog} onOpenChange={setShowCreateDialog}>
            <DialogTrigger asChild>
              <Button>
                <Plus className="h-4 w-4 mr-2" />
                Create API Key
              </Button>
            </DialogTrigger>
            <DialogContent>
              <DialogHeader>
                <DialogTitle>Create New API Key</DialogTitle>
                <DialogDescription>
                  Generate a new API key with specific permissions
                </DialogDescription>
              </DialogHeader>
              <div className="space-y-4">
                <div className="space-y-2">
                  <Label>Key Name</Label>
                  <Input
                    placeholder="My App API Key"
                    value={newKeyName}
                    onChange={(e) => setNewKeyName(e.target.value)}
                  />
                </div>
                <div className="space-y-2">
                  <Label>Permissions</Label>
                  <div className="space-y-2">
                    {scopes.map((scope) => (
                      <div key={scope.id} className="flex items-center space-x-2">
                        <Checkbox
                          id={scope.id}
                          checked={selectedScopes.includes(scope.id)}
                          onCheckedChange={(checked) => {
                            if (checked) {
                              setSelectedScopes([...selectedScopes, scope.id]);
                            } else {
                              setSelectedScopes(selectedScopes.filter(s => s !== scope.id));
                            }
                          }}
                        />
                        <Label htmlFor={scope.id} className="text-sm">
                          {scope.name}
                        </Label>
                      </div>
                    ))}
                  </div>
                </div>
                <Button
                  className="w-full"
                  onClick={() => createKey({ name: newKeyName, scopes: selectedScopes })}
                  disabled={!newKeyName || selectedScopes.length === 0}
                >
                  Create API Key
                </Button>
              </div>
            </DialogContent>
          </Dialog>
        </div>
      </CardHeader>
      <CardContent>
        <Table>
          <TableHeader>
            <TableRow>
              <TableHead>Name</TableHead>
              <TableHead>Key</TableHead>
              <TableHead>Permissions</TableHead>
              <TableHead>Last Used</TableHead>
              <TableHead>Created</TableHead>
              <TableHead></TableHead>
            </TableRow>
          </TableHeader>
          <TableBody>
            {apiKeys?.map((apiKey) => (
              <TableRow key={apiKey.id}>
                <TableCell className="font-medium">{apiKey.name}</TableCell>
                <TableCell>
                  <div className="flex items-center gap-2">
                    <code className="text-xs bg-muted px-2 py-1 rounded">
                      {visibleKeys.has(apiKey.id) 
                        ? apiKey.key 
                        : `${apiKey.key.slice(0, 12)}...`}
                    </code>
                    <Button
                      variant="ghost"
                      size="icon"
                      className="h-6 w-6"
                      onClick={() => toggleKeyVisibility(apiKey.id)}
                    >
                      {visibleKeys.has(apiKey.id) ? (
                        <EyeOff className="h-3 w-3" />
                      ) : (
                        <Eye className="h-3 w-3" />
                      )}
                    </Button>
                    <Button
                      variant="ghost"
                      size="icon"
                      className="h-6 w-6"
                      onClick={() => {
                        navigator.clipboard.writeText(apiKey.key);
                        toast.success('API key copied to clipboard');
                      }}
                    >
                      <Copy className="h-3 w-3" />
                    </Button>
                  </div>
                </TableCell>
                <TableCell>
                  <div className="flex flex-wrap gap-1">
                    {apiKey.scopes.slice(0, 2).map((scope) => (
                      <Badge key={scope} variant="secondary" className="text-xs">
                        {scope}
                      </Badge>
                    ))}
                    {apiKey.scopes.length > 2 && (
                      <Badge variant="secondary" className="text-xs">
                        +{apiKey.scopes.length - 2}
                      </Badge>
                    )}
                  </div>
                </TableCell>
                <TableCell>
                  {apiKey.lastUsed ? (
                    <span className="text-sm text-muted-foreground">
                      {new Date(apiKey.lastUsed).toLocaleDateString()}
                    </span>
                  ) : (
                    <span className="text-sm text-muted-foreground">Never</span>
                  )}
                </TableCell>
                <TableCell>
                  <span className="text-sm text-muted-foreground">
                    {new Date(apiKey.createdAt).toLocaleDateString()}
                  </span>
                </TableCell>
                <TableCell>
                  <Button
                    variant="ghost"
                    size="icon"
                    onClick={() => deleteKey(apiKey.id)}
                  >
                    <Trash className="h-4 w-4" />
                  </Button>
                </TableCell>
              </TableRow>
            ))}
          </TableBody>
        </Table>
      </CardContent>
    </Card>
  );
}
```

## Landing Pages & Microsites

### Dynamic City Landing Page
```typescript
// app/[locale]/(landing)/[country]/[city]/page.tsx
import { Metadata } from 'next';
import { notFound } from 'next/navigation';
import { CityHero } from '@/components/features/landing/city-hero';
import { TrendingEvents } from '@/components/features/landing/trending-events';
import { TrendingArtists } from '@/components/features/landing/trending-artists';
import { TrendingVenues } from '@/components/features/landing/trending-venues';
import { AdSenseBanner } from '@/components/features/landing/adsense-banner';
import { StructuredData } from '@/components/features/seo/structured-data';
import { landingService } from '@/services/landing-service';

interface CityPageProps {
  params: {
    locale: string;
    country: string;
    city: string;
  };
}

export async function generateMetadata({ params }: CityPageProps): Promise<Metadata> {
  const cityData = await landingService.getCityData(params.country, params.city);
  
  if (!cityData) return {};
  
  return {
    title: `Events in ${cityData.name} - ${cityData.eventCount} Upcoming Events | EventfulHQ`,
    description: `Discover ${cityData.eventCount} amazing events happening in ${cityData.name}. Find concerts, festivals, workshops, and more. Book tickets for the best events in ${cityData.name}.`,
    openGraph: {
      title: `Events in ${cityData.name}`,
      description: `Discover amazing events in ${cityData.name}`,
      images: [cityData.heroImage],
      type: 'website',
    },
    alternates: {
      canonical: `https://eventfulhq.com/${params.locale}/${params.country}/${params.city}`,
      languages: {
        'en': `https://eventfulhq.com/en/${params.country}/${params.city}`,
        'es': `https://eventfulhq.com/es/${params.country}/${params.city}`,
        // ... other languages
      },
    },
  };
}

export async function generateStaticParams() {
  // Generate static pages for top cities
  const topCities = await landingService.getTopCities();
  
  return topCities.map((city) => ({
    country: city.countryCode.toLowerCase(),
    city: city.slug,
  }));
}

export default async function CityLandingPage({ params }: CityPageProps) {
  const cityData = await landingService.getCityData(params.country, params.city);
  
  if (!cityData) {
    notFound();
  }
  
  const [trendingEvents, trendingArtists, trendingVenues] = await Promise.all([
    landingService.getTrendingEvents(cityData.id),
    landingService.getTrendingArtists(cityData.id),
    landingService.getTrendingVenues(cityData.id),
  ]);
  
  const structuredData = {
    '@context': 'https://schema.org',
    '@type': 'City',
    name: cityData.name,
    geo: {
      '@type': 'GeoCoordinates',
      latitude: cityData.coordinates.lat,
      longitude: cityData.coordinates.lng,
    },
    containsPlace: trendingVenues.map(venue => ({
      '@type': 'EventVenue',
      name: venue.name,
      address: venue.address,
    })),
    event: trendingEvents.map(event => ({
      '@type': 'Event',
      name: event.title,
      startDate: event.date,
      location: {
        '@type': 'Place',
        name: event.venue.name,
        address: event.venue.address,
      },
      offers: {
        '@type': 'Offer',
        price: event.price,
        priceCurrency: event.currency,
        availability: event.isSoldOut ? 'SoldOut' : 'InStock',
      },
    })),
  };
  
  return (
    <>
      <StructuredData data={structuredData} />
      
      <div className="min-h-screen">
        <CityHero city={cityData} />
        
        <div className="container py-8 space-y-12">
          {/* AdSense Banner - Top */}
          <AdSenseBanner 
            slot="city-landing-top" 
            format="horizontal"
            className="mb-8"
          />
          
          {/* Trending Events Section */}
          <section>
            <h2 className="text-3xl font-bold mb-6">
              Trending Events in {cityData.name}
            </h2>
            <TrendingEvents events={trendingEvents} city={cityData} />
          </section>
          
          {/* AdSense Banner - Middle */}
          <AdSenseBanner 
            slot="city-landing-middle" 
            format="rectangle"
            className="my-8"
          />
          
          {/* Trending Artists Section */}
          <section>
            <h2 className="text-3xl font-bold mb-6">
              Popular Artists in {cityData.name}
            </h2>
            <TrendingArtists artists={trendingArtists} />
          </section>
          
          {/* Trending Venues Section */}
          <section>
            <h2 className="text-3xl font-bold mb-6">
              Top Venues in {cityData.name}
            </h2>
            <TrendingVenues venues={trendingVenues} />
          </section>
          
          {/* AdSense Banner - Bottom */}
          <AdSenseBanner 
            slot="city-landing-bottom" 
            format="horizontal"
            className="mt-8"
          />
          
          {/* SEO Content */}
          <section className="prose max-w-none">
            <h3>About Events in {cityData.name}</h3>
            <p>{cityData.description}</p>
            <p>
              With over {cityData.eventCount} upcoming events, {cityData.name} is 
              a vibrant hub for entertainment and culture. From live music concerts 
              to art exhibitions, food festivals to tech conferences, there's always 
              something exciting happening in {cityData.name}.
            </p>
          </section>
        </div>
      </div>
    </>
  );
}
```

### User Microsite (Without Ads for Premium Users)
```typescript
// app/[locale]/(landing)/u/[username]/page.tsx
import { Metadata } from 'next';
import { notFound } from 'next/navigation';
import { UserProfile } from '@/components/features/landing/user-profile';
import { UserEvents } from '@/components/features/landing/user-events';
import { UserStats } from '@/components/features/landing/user-stats';
import { ShareButtons } from '@/components/features/landing/share-buttons';
import { AdSenseBanner } from '@/components/features/landing/adsense-banner';
import { landingService } from '@/services/landing-service';
import { userService } from '@/services/user-service';

interface UserMicrositeProps {
  params: {
    locale: string;
    username: string;
  };
}

export async function generateMetadata({ params }: UserMicrositeProps): Promise<Metadata> {
  const user = await userService.getUserByUsername(params.username);
  
  if (!user) return {};
  
  return {
    title: `${user.name} - Event Organizer | EventfulHQ`,
    description: user.bio || `Check out events organized by ${user.name} on EventfulHQ`,
    openGraph: {
      title: user.name,
      description: user.bio,
      images: [user.avatar],
      type: 'profile',
    },
  };
}

export default async function UserMicrosite({ params }: UserMicrositeProps) {
  const user = await userService.getUserByUsername(params.username);
  
  if (!user) {
    notFound();
  }
  
  const [userEvents, userStats] = await Promise.all([
    landingService.getUserEvents(user.id),
    landingService.getUserStats(user.id),
  ]);
  
  const isPremium = user.subscription?.tier === 'premium';
  const customTheme = user.micrositeTheme || 'default';
  
  return (
    <div className={`min-h-screen theme-${customTheme}`}>
      <UserProfile user={user} stats={userStats} />
      
      <div className="container py-8 space-y-8">
        {/* No ads for premium users */}
        {!isPremium && (
          <AdSenseBanner 
            slot="user-microsite-top" 
            format="horizontal"
          />
        )}
        
        <section>
          <div className="flex items-center justify-between mb-6">
            <h2 className="text-2xl font-bold">Upcoming Events</h2>
            <ShareButtons 
              url={`https://eventfulhq.com/u/${user.username}`}
              title={`Check out events by ${user.name}`}
            />
          </div>
          <UserEvents events={userEvents} showOrganizer={false} />
        </section>
        
        {!isPremium && userEvents.length > 6 && (
          <AdSenseBanner 
            slot="user-microsite-middle" 
            format="rectangle"
          />
        )}
        
        <UserStats stats={userStats} />
        
        {!isPremium && (
          <AdSenseBanner 
            slot="user-microsite-bottom" 
            format="horizontal"
          />
        )}
      </div>
    </div>
  );
}
```

### Event Microsite Component
```typescript
// app/[locale]/(landing)/e/[eventSlug]/page.tsx
import { Metadata } from 'next';
import { notFound } from 'next/navigation';
import { EventHero } from '@/components/features/landing/event-hero';
import { EventDetails } from '@/components/features/landing/event-details';
import { TicketWidget } from '@/components/features/landing/ticket-widget';
import { WhatsAppRSVP } from '@/components/features/landing/whatsapp-rsvp';
import { AdSenseBanner } from '@/components/features/landing/adsense-banner';
import { StructuredData } from '@/components/features/seo/structured-data';
import { eventService } from '@/services/event-service';

interface EventMicrositeProps {
  params: {
    locale: string;
    eventSlug: string;
  };
}

export async function generateMetadata({ params }: EventMicrositeProps): Promise<Metadata> {
  const event = await eventService.getEventBySlug(params.eventSlug);
  
  if (!event) return {};
  
  return {
    title: `${event.title} - ${new Date(event.date).toLocaleDateString()} | EventfulHQ`,
    description: event.description,
    openGraph: {
      title: event.title,
      description: event.description,
      images: [event.coverImage],
      type: 'website',
    },
  };
}

export default async function EventMicrosite({ params }: EventMicrositeProps) {
  const event = await eventService.getEventBySlug(params.eventSlug);
  
  if (!event) {
    notFound();
  }
  
  const isPremiumEvent = event.organizer.subscription?.tier === 'premium';
  const customTheme = event.micrositeTheme || event.organizer.micrositeTheme || 'default';
  
  const structuredData = {
    '@context': 'https://schema.org',
    '@type': 'Event',
    name: event.title,
    description: event.description,
    startDate: event.date,
    endDate: event.endDate,
    location: {
      '@type': 'Place',
      name: event.venue.name,
      address: {
        '@type': 'PostalAddress',
        streetAddress: event.venue.address,
        addressLocality: event.venue.city,
        addressCountry: event.venue.country,
      },
      geo: {
        '@type': 'GeoCoordinates',
        latitude: event.venue.coordinates.lat,
        longitude: event.venue.coordinates.lng,
      },
    },
    offers: {
      '@type': 'Offer',
      url: `https://eventfulhq.com/e/${event.slug}`,
      price: event.price,
      priceCurrency: event.currency,
      availability: event.isSoldOut 
        ? 'https://schema.org/SoldOut' 
        : 'https://schema.org/InStock',
      validFrom: new Date().toISOString(),
    },
    performer: event.artists?.map(artist => ({
      '@type': 'Person',
      name: artist.name,
    })),
    organizer: {
      '@type': 'Organization',
      name: event.organizer.name,
      url: `https://eventfulhq.com/u/${event.organizer.username}`,
    },
  };
  
  return (
    <>
      <StructuredData data={structuredData} />
      
      <div className={`min-h-screen theme-${customTheme}`}>
        <EventHero event={event} />
        
        <div className="container py-8">
          <div className="grid gap-8 lg:grid-cols-3">
            <div className="lg:col-span-2 space-y-8">
              <EventDetails event={event} />
              
              {!isPremiumEvent && (
                <AdSenseBanner 
                  slot="event-microsite-content" 
                  format="rectangle"
                />
              )}
              
              {/* Event Description */}
              <section>
                <h2 className="text-2xl font-bold mb-4">About This Event</h2>
                <div className="prose max-w-none">
                  {event.description}
                </div>
              </section>
              
              {/* Artists/Performers */}
              {event.artists && event.artists.length > 0 && (
                <section>
                  <h2 className="text-2xl font-bold mb-4">Featuring</h2>
                  <div className="grid gap-4 sm:grid-cols-2">
                    {event.artists.map(artist => (
                      <ArtistCard key={artist.id} artist={artist} />
                    ))}
                  </div>
                </section>
              )}
            </div>
            
            <div className="space-y-6">
              {/* Ticket Widget */}
              <TicketWidget event={event} />
              
              {/* WhatsApp RSVP */}
              {event.isWhatsAppRSVP && (
                <WhatsAppRSVP event={event} />
              )}
              
              {!isPremiumEvent && (
                <AdSenseBanner 
                  slot="event-microsite-sidebar" 
                  format="vertical"
                />
              )}
              
              {/* Venue Information */}
              <VenueInfo venue={event.venue} />
              
              {/* Share Buttons */}
              <ShareButtons 
                url={`https://eventfulhq.com/e/${event.slug}`}
                title={event.title}
                description={event.description}
              />
            </div>
          </div>
        </div>
        
        {!isPremiumEvent && (
          <AdSenseBanner 
            slot="event-microsite-bottom" 
            format="horizontal"
            className="mt-8"
          />
        )}
      </div>
    </>
  );
}
```

### AdSense Integration Component
```typescript
// components/features/landing/adsense-banner.tsx
'use client';

import { useEffect } from 'react';
import { usePathname } from 'next/navigation';
import Script from 'next/script';

interface AdSenseBannerProps {
  slot: string;
  format?: 'horizontal' | 'vertical' | 'rectangle' | 'auto';
  className?: string;
  responsive?: boolean;
}

export function AdSenseBanner({ 
  slot, 
  format = 'auto',
  className = '',
  responsive = true 
}: AdSenseBannerProps) {
  const pathname = usePathname();
  
  useEffect(() => {
    try {
      // Push ads after route change
      (window.adsbygoogle = window.adsbygoogle || []).push({});
    } catch (err) {
      console.error('AdSense error:', err);
    }
  }, [pathname]);
  
  const formatStyles = {
    horizontal: { width: '728px', height: '90px' },
    vertical: { width: '300px', height: '600px' },
    rectangle: { width: '336px', height: '280px' },
    auto: { width: '100%', height: 'auto' },
  };
  
  const style = formatStyles[format];
  
  return (
    <>
      <Script
        id="adsense-script"
        async
        src={`https://pagead2.googlesyndication.com/pagead/js/adsbygoogle.js?client=${process.env.NEXT_PUBLIC_ADSENSE_CLIENT_ID}`}
        crossOrigin="anonymous"
        strategy="afterInteractive"
      />
      <div className={`adsense-container ${className}`}>
        <ins
          className="adsbygoogle"
          style={{
            display: 'block',
            width: style.width,
            height: style.height,
            ...(responsive && { 
              width: '100%',
              height: 'auto',
            }),
          }}
          data-ad-client={process.env.NEXT_PUBLIC_ADSENSE_CLIENT_ID}
          data-ad-slot={slot}
          data-ad-format={format}
          data-full-width-responsive={responsive ? 'true' : 'false'}
        />
      </div>
    </>
  );
}

// AdSense configuration for different page types
export const adSlots = {
  'city-landing-top': '1234567890',
  'city-landing-middle': '2345678901',
  'city-landing-bottom': '3456789012',
  'country-landing-top': '4567890123',
  'country-landing-sidebar': '5678901234',
  'user-microsite-top': '6789012345',
  'user-microsite-middle': '7890123456',
  'user-microsite-bottom': '8901234567',
  'event-microsite-content': '9012345678',
  'event-microsite-sidebar': '0123456789',
  'event-microsite-bottom': '1234567890',
};
```

## Bot Management & SEO Strategy

### Bot Detection Middleware
```typescript
// middleware/bot-detection.ts
import { NextRequest, NextResponse } from 'next/server';
import { botPatterns, seoBotsPatterns, llmBotsPatterns } from '@/lib/seo/bot-patterns';

export async function botDetectionMiddleware(request: NextRequest) {
  const userAgent = request.headers.get('user-agent') || '';
  const ip = request.ip || request.headers.get('x-forwarded-for') || '';
  
  // Check if it's a bot
  const isBot = botPatterns.some(pattern => pattern.test(userAgent));
  const isSEOBot = seoBotsPatterns.some(pattern => pattern.test(userAgent));
  const isLLMBot = llmBotsPatterns.some(pattern => pattern.test(userAgent));
  
  // Log bot access for analytics
  if (isBot) {
    await logBotAccess({
      userAgent,
      ip,
      path: request.nextUrl.pathname,
      type: isSEOBot ? 'seo' : isLLMBot ? 'llm' : 'other',
      timestamp: new Date(),
    });
  }
  
  // Block unauthorized bots
  if (isBot && !isSEOBot && !isLLMBot) {
    // Check rate limiting for suspicious bots
    const rateLimit = await checkBotRateLimit(ip);
    
    if (rateLimit.exceeded) {
      return new NextResponse('Too Many Requests', { status: 429 });
    }
    
    // Return minimal response for unauthorized bots
    return new NextResponse('Access Denied', { 
      status: 403,
      headers: {
        'X-Robots-Tag': 'noindex, nofollow',
      }
    });
  }
  
  // For SEO and LLM bots, add special headers and optimizations
  if (isSEOBot || isLLMBot) {
    const response = NextResponse.next();
    
    // Add bot-specific headers
    response.headers.set('X-Bot-Type', isSEOBot ? 'seo' : 'llm');
    response.headers.set('Cache-Control', 'public, max-age=3600');
    
    // For LLM bots, add structured data hints
    if (isLLMBot) {
      response.headers.set('X-LLM-Hints', 'structured-data-available');
    }
    
    return response;
  }
  
  return NextResponse.next();
}

// Bot patterns configuration
export const botPatterns = [
  // Common crawlers and scrapers
  /bot|crawler|spider|scraper|curl|wget|python-requests/i,
  // Specific bad bots
  /ahrefsbot|semrushbot|dotbot|rogerbot|facebookexternalhit|ia_archiver/i,
  // Vulnerability scanners
  /nikto|sqlmap|openvas|nessus|w3af/i,
];

export const seoBotsPatterns = [
  // Search engine bots
  /googlebot|bingbot|slurp|duckduckbot|baiduspider|yandexbot/i,
  // Social media bots
  /linkedinbot|whatsapp|telegrambot/i,
  // SEO tool bots (allowed)
  /screaming frog|deepcrawl|oncrawl/i,
];

export const llmBotsPatterns = [
  // AI/LLM crawlers
  /gpt-?bot|claude-?bot|anthropic|openai/i,
  /chatgpt|bard|llama|perplexity/i,
  // AI research bots
  /commoncrawl|ccbot/i,
];

// Rate limiting for bots
async function checkBotRateLimit(ip: string): Promise<{ exceeded: boolean }> {
  const key = `bot-rate-limit:${ip}`;
  const limit = 100; // requests per hour
  const window = 3600; // 1 hour in seconds
  
  // Implementation would use Redis or similar
  // This is a simplified example
  return { exceeded: false };
}

// Log bot access for analytics
async function logBotAccess(data: {
  userAgent: string;
  ip: string;
  path: string;
  type: string;
  timestamp: Date;
}) {
  // Log to analytics service
  console.log('Bot access:', data);
}
```

### Dynamic Robots.txt Generation
```typescript
// app/robots.txt/route.ts
import { NextRequest } from 'next/server';

export async function GET(request: NextRequest) {
  const host = request.headers.get('host') || 'eventfulhq.com';
  const protocol = process.env.NODE_ENV === 'production' ? 'https' : 'http';
  
  const robotsTxt = `# EventfulHQ Robots.txt
# Generated dynamically based on domain

User-agent: *
Allow: /
Disallow: /api/
Disallow: /admin/
Disallow: /super-admin/
Disallow: /_next/
Disallow: /static/
Crawl-delay: 1

# Search engines
User-agent: Googlebot
Allow: /
Crawl-delay: 0

User-agent: Bingbot
Allow: /
Crawl-delay: 0

User-agent: Slurp
Allow: /
Crawl-delay: 1

User-agent: DuckDuckBot
Allow: /
Crawl-delay: 1

# Social media
User-agent: LinkedInBot
Allow: /

User-agent: WhatsApp
Allow: /

User-agent: Twitterbot
Allow: /

# AI/LLM Bots
User-agent: GPTBot
Allow: /
Crawl-delay: 2

User-agent: Claude-Web
Allow: /
Crawl-delay: 2

User-agent: CCBot
Allow: /
Crawl-delay: 5

# Bad bots
User-agent: AhrefsBot
Disallow: /

User-agent: SemrushBot
Disallow: /

User-agent: DotBot
Disallow: /

User-agent: MJ12bot
Disallow: /

# Sitemaps
Sitemap: ${protocol}://${host}/sitemap.xml
Sitemap: ${protocol}://${host}/sitemap-events.xml
Sitemap: ${protocol}://${host}/sitemap-users.xml
Sitemap: ${protocol}://${host}/sitemap-venues.xml
Sitemap: ${protocol}://${host}/sitemap-cities.xml
Sitemap: ${protocol}://${host}/sitemap-countries.xml
`;

  return new Response(robotsTxt, {
    headers: {
      'Content-Type': 'text/plain',
      'Cache-Control': 'public, max-age=3600',
    },
  });
}
```

### Dynamic Sitemap Generation
```typescript
// app/sitemap.xml/route.ts
import { NextRequest } from 'next/server';
import { sitemapService } from '@/services/sitemap-service';

export async function GET(request: NextRequest) {
  const host = request.headers.get('host') || 'eventfulhq.com';
  const protocol = process.env.NODE_ENV === 'production' ? 'https' : 'http';
  
  // Generate sitemap index
  const sitemaps = [
    'sitemap-static.xml',
    'sitemap-events.xml',
    'sitemap-users.xml',
    'sitemap-venues.xml',
    'sitemap-cities.xml',
    'sitemap-countries.xml',
  ];
  
  const sitemapIndex = `<?xml version="1.0" encoding="UTF-8"?>
<sitemapindex xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
${sitemaps.map(sitemap => `  <sitemap>
    <loc>${protocol}://${host}/${sitemap}</loc>
    <lastmod>${new Date().toISOString()}</lastmod>
  </sitemap>`).join('\n')}
</sitemapindex>`;

  return new Response(sitemapIndex, {
    headers: {
      'Content-Type': 'application/xml',
      'Cache-Control': 'public, max-age=3600',
    },
  });
}

// Dynamic event sitemap
export async function generateEventSitemap() {
  const events = await sitemapService.getPublishedEvents();
  
  const sitemap = `<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9"
        xmlns:image="http://www.google.com/schemas/sitemap-image/1.1">
${events.map(event => `  <url>
    <loc>https://eventfulhq.com/e/${event.slug}</loc>
    <lastmod>${event.updatedAt}</lastmod>
    <changefreq>daily</changefreq>
    <priority>0.8</priority>
    <image:image>
      <image:loc>${event.coverImage}</image:loc>
      <image:title>${event.title}</image:title>
    </image:image>
  </url>`).join('\n')}
</urlset>`;

  return sitemap;
}
```

### SEO Monitoring Dashboard
```typescript
// components/features/seo/seo-monitoring.tsx
'use client';

import { useQuery } from '@tanstack/react-query';
import { Card, CardContent, CardDescription, CardHeader, CardTitle } from '@/components/ui/card';
import { Tabs, TabsContent, TabsList, TabsTrigger } from '@/components/ui/tabs';
import { Progress } from '@/components/ui/progress';
import { Badge } from '@/components/ui/badge';
import {
  LineChart,
  Line,
  BarChart,
  Bar,
  XAxis,
  YAxis,
  CartesianGrid,
  Tooltip,
  ResponsiveContainer,
} from 'recharts';
import { seoService } from '@/services/seo-service';
import { Bot, Search, TrendingUp, AlertCircle } from 'lucide-react';

export function SEOMonitoring() {
  const { data: seoData } = useQuery({
    queryKey: ['seo-monitoring'],
    queryFn: seoService.getMonitoringData,
  });

  const { data: botTraffic } = useQuery({
    queryKey: ['bot-traffic'],
    queryFn: seoService.getBotTraffic,
  });

  return (
    <div className="space-y-6">
      <div className="grid gap-4 md:grid-cols-2 lg:grid-cols-4">
        <Card>
          <CardHeader className="flex flex-row items-center justify-between space-y-0 pb-2">
            <CardTitle className="text-sm font-medium">SEO Score</CardTitle>
            <Search className="h-4 w-4 text-muted-foreground" />
          </CardHeader>
          <CardContent>
            <div className="text-2xl font-bold">{seoData?.seoScore}/100</div>
            <Progress value={seoData?.seoScore} className="mt-2" />
          </CardContent>
        </Card>

        <Card>
          <CardHeader className="flex flex-row items-center justify-between space-y-0 pb-2">
            <CardTitle className="text-sm font-medium">Indexed Pages</CardTitle>
            <TrendingUp className="h-4 w-4 text-muted-foreground" />
          </CardHeader>
          <CardContent>
            <div className="text-2xl font-bold">
              {seoData?.indexedPages.toLocaleString()}
            </div>
            <p className="text-xs text-muted-foreground">
              +{seoData?.newPagesThisWeek} this week
            </p>
          </CardContent>
        </Card>

        <Card>
          <CardHeader className="flex flex-row items-center justify-between space-y-0 pb-2">
            <CardTitle className="text-sm font-medium">Bot Traffic</CardTitle>
            <Bot className="h-4 w-4 text-muted-foreground" />
          </CardHeader>
          <CardContent>
            <div className="text-2xl font-bold">
              {botTraffic?.total.toLocaleString()}
            </div>
            <div className="flex gap-2 mt-2">
              <Badge variant="secondary" className="text-xs">
                SEO: {botTraffic?.seo}
              </Badge>
              <Badge variant="secondary" className="text-xs">
                LLM: {botTraffic?.llm}
              </Badge>
            </div>
          </CardContent>
        </Card>

        <Card>
          <CardHeader className="flex flex-row items-center justify-between space-y-0 pb-2">
            <CardTitle className="text-sm font-medium">Issues</CardTitle>
            <AlertCircle className="h-4 w-4 text-muted-foreground" />
          </CardHeader>
          <CardContent>
            <div className="text-2xl font-bold">{seoData?.issues.total}</div>
            <div className="flex gap-2 mt-2">
              <Badge variant="destructive" className="text-xs">
                Critical: {seoData?.issues.critical}
              </Badge>
              <Badge variant="outline" className="text-xs">
                Warnings: {seoData?.issues.warnings}
              </Badge>
            </div>
          </CardContent>
        </Card>
      </div>

      <Tabs defaultValue="traffic" className="space-y-4">
        <TabsList>
          <TabsTrigger value="traffic">Bot Traffic</TabsTrigger>
          <TabsTrigger value="crawl">Crawl Stats</TabsTrigger>
          <TabsTrigger value="performance">Performance</TabsTrigger>
          <TabsTrigger value="issues">Issues</TabsTrigger>
        </TabsList>

        <TabsContent value="traffic" className="space-y-4">
          <Card>
            <CardHeader>
              <CardTitle>Bot Traffic Over Time</CardTitle>
              <CardDescription>
                SEO and LLM bot visits to your site
              </CardDescription>
            </CardHeader>
            <CardContent>
              <ResponsiveContainer width="100%" height={300}>
                <LineChart data={botTraffic?.timeline}>
                  <CartesianGrid strokeDasharray="3 3" />
                  <XAxis dataKey="date" />
                  <YAxis />
                  <Tooltip />
                  <Line 
                    type="monotone" 
                    dataKey="seo" 
                    stroke="#8884d8" 
                    name="SEO Bots"
                  />
                  <Line 
                    type="monotone" 
                    dataKey="llm" 
                    stroke="#82ca9d" 
                    name="LLM Bots"
                  />
                  <Line 
                    type="monotone" 
                    dataKey="blocked" 
                    stroke="#ff7c7c" 
                    name="Blocked"
                  />
                </LineChart>
              </ResponsiveContainer>
            </CardContent>
          </Card>

          <div className="grid gap-4 md:grid-cols-2">
            <Card>
              <CardHeader>
                <CardTitle>Top SEO Bots</CardTitle>
              </CardHeader>
              <CardContent>
                <div className="space-y-2">
                  {botTraffic?.topSeoBots.map((bot) => (
                    <div key={bot.name} className="flex items-center justify-between">
                      <span className="text-sm">{bot.name}</span>
                      <div className="flex items-center gap-2">
                        <span className="text-sm text-muted-foreground">
                          {bot.visits.toLocaleString()}
                        </span>
                        <Progress value={bot.percentage} className="w-20" />
                      </div>
                    </div>
                  ))}
                </div>
              </CardContent>
            </Card>

            <Card>
              <CardHeader>
                <CardTitle>Blocked Bots</CardTitle>
              </CardHeader>
              <CardContent>
                <div className="space-y-2">
                  {botTraffic?.blockedBots.map((bot) => (
                    <div key={bot.name} className="flex items-center justify-between">
                      <span className="text-sm">{bot.name}</span>
                      <Badge variant="destructive" className="text-xs">
                        {bot.attempts} attempts
                      </Badge>
                    </div>
                  ))}
                </div>
              </CardContent>
            </Card>
          </div>
        </TabsContent>

        <TabsContent value="crawl" className="space-y-4">
          <Card>
            <CardHeader>
              <CardTitle>Crawl Statistics</CardTitle>
            </CardHeader>
            <CardContent>
              <ResponsiveContainer width="100%" height={300}>
                <BarChart data={seoData?.crawlStats}>
                  <CartesianGrid strokeDasharray="3 3" />
                  <XAxis dataKey="pageType" />
                  <YAxis />
                  <Tooltip />
                  <Bar dataKey="crawled" fill="#8884d8" name="Crawled" />
                  <Bar dataKey="indexed" fill="#82ca9d" name="Indexed" />
                </BarChart>
              </ResponsiveContainer>
            </CardContent>
          </Card>
        </TabsContent>
      </Tabs>
    </div>
  );
}
```