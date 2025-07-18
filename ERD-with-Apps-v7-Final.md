

## **EventfulHQ ERD Schema v7.0: The Final Production Blueprint**

### **Preface**

* **Author**: Gemini, The Cognitive Engine of EventfulHQ  
* **Version**: 7.0  
* **Date**: 18 July 2025  
* **Description**: This document presents the definitive enterprise-grade Entity Relationship Diagram (ERD) for EventfulHQ. It represents a mature Domain-Driven Design (DDD) architecture with 13 distinct domains, engineered for global scale, regulatory compliance, and enterprise integration capabilities, fully supporting our specified technology stack (NestJS, PostgreSQL, Redis, GraphQL/Apollo, Next.js 15).

---

### **Architectural Philosophy & Core Business Logic**

* **AI-First Architecture**: Every domain integrates with AI services, with usage and models tracked for AIOps.  
* **WhatsApp-First Communication**: Native WhatsApp Business API integration is modeled for templates, conversations, and messaging.  
* **Global Scale**: Multi-language, multi-currency, and multi-timezone support is built into the core (EntityTranslations, Destinations).  
* **Enterprise Security**: Banking-grade security with comprehensive, immutable audit trails.  
* **SaaS Model**: The schema fully supports our three-tier subscription model:  
  * **Base**: Free  
  * **Pro**: $9.99 / ₹999 per month (999 Credits)  
  * **ProMax**: $99.99 / ₹9999 per month (9999 Credits)  
  * An annual payment option with a 15% discount is managed at the billing layer.  
* **Platform Economy**: The financial system is built on a universal credit model where **1 USD \= 50 INR \= 1 Credit**.

---

### **ERD Conventions & Standards**

* **PK**: Primary Key (UUID format unless specified otherwise)  
* **FK**: Foreign Key  
* **P\_FK**: Polymorphic Foreign Key (...\_id, ...\_type)  
* **\-\>**: Denotes a Foreign Key relationship  
* **\[idx\]**: Recommended database index  
* **\[unique\]**: Unique constraint  
* **\[BaseEntity\]**: Inherits a full suite of audit and soft-delete fields: created\_at, updated\_at, created\_by\_user\_id, updated\_by\_user\_id, version, is\_deleted, deleted\_at, deleted\_by\_user\_id.

---

### ***Schema 1: Security & Governance Domain***

***Architectural Philosophy:** This domain is the security and compliance kernel of the platform. It is designed with a **Security-First** and **Compliance-by-Design** approach, incorporating comprehensive auditing, session management, data governance, and API security. It is structured to be a distinct, authoritative service within our domain-driven architecture.*

---

#### ***Users `[BaseEntity]`***

* ***Core Purpose:** The foundational record for every human, handling authentication, core identity, and archetype classification.*  
* ***Definition & Business Logic:** This table is the single source of truth for a person's identity on the platform. It links their secure authentication credentials (`email`, `whatsapp_number`) to their primary function (`primary_archetype`) and commercial status (`subscription_tier`). The strict `UNIQUE` constraints on email and WhatsApp numbers, combined with a mandatory two-factor verification flow upon registration, are the first line of defense against spam and fraudulent accounts. The `[BaseEntity]` implementation with soft-delete ensures that user data can be deactivated for compliance without breaking historical referential integrity in other tables like audit logs.*  
* ***Attributes:** | Column Name | Data Type | Constraints & Description | | :--- | :--- | :--- | | **user\_id** | `UUID` | **Primary Key**, `DEFAULT gen_random_uuid()`. The unique identifier for the user record. | | **name** | `VARCHAR(255)` | The full, legal name of the user, used for contracts and official communication. | | **email** | `VARCHAR(255)` | **UNIQUE, NOT NULL, \[idx\]**. The user's verified email address, serving as a primary login identifier. | | **email\_verified** | `BOOLEAN` | `DEFAULT FALSE`. A flag that is set to `true` only after successful completion of the email OTP verification loop. | | **whatsapp\_number** | `VARCHAR(20)` | **UNIQUE**. The user's verified WhatsApp number, a cornerstone of our WhatsApp-First features and 2FA. | | **whatsapp\_verified**| `BOOLEAN` | `DEFAULT FALSE`. A flag that is set to `true` only after successful completion of the WhatsApp OTP verification loop. | | **primary\_archetype**| `ENUM` | **NOT NULL**. Defines the user's primary role on the platform: `'Artist'`, `'Expert'`, `'Fan'`, `'SuperAdmin'`. This determines which specialized profile table is linked. | | **admin\_tier\_id** | `UUID` | **Foreign Key** \-\> `admin_tiers(tier_id)`. Nullable. Only populated if `primary_archetype` is 'SuperAdmin', assigning a specific rank from the governance hierarchy. | | **...** | | Includes all `[BaseEntity]` audit and soft-delete fields (`created_at`, `updated_at`, `is_deleted`, etc.). |*  
* ***Key Relationships & Cardinality:***  
  * *`Users` **(1) \--- (1)** `UserProfiles` (`ArtistProfiles`, `ExpertProfiles`, or `FanProfiles`)*  
  * *`Users` **(1) \--- (N)** `TeamMemberships`*  
  * *`Users` **(1) \--- (N)** `UserSessions`*  
  * *`Users` **(1) \--- (N)** `UserConsents`*  
* ***Recommended Indexes:** `CREATE INDEX idx_users_email ON users(email) WHERE NOT is_deleted;`, `CREATE INDEX idx_users_archetype ON users(primary_archetype) WHERE NOT is_deleted;`*

---

#### ***AdminTiers***

* ***Core Purpose:** Defines the strict, 11-tier administrative hierarchy for platform governance.*  
* ***Definition & Business Logic:** This table is the backbone of our internal Role-Based Access Control (RBAC) for `SuperAdmins`. It is not a user-configurable table but is managed by the Platform Governors. The `tier_level` provides a clear numerical hierarchy, ensuring a higher-level admin can manage lower-level ones. The `permissions` and `scope_restrictions` are JSONB fields that allow for highly flexible and granular control over what an admin can see and do (e.g., a 'Regional Governor' would have a scope restriction of `{"region": "APAC"}`).*  
* ***Attributes:** | Column Name | Data Type | Constraints & Description | | :--- | :--- | :--- | | **tier\_id** | `UUID` | **Primary Key**, `DEFAULT gen_random_uuid()`. Unique ID for the administrative tier. | | **tier\_name** | `VARCHAR(100)` | **UNIQUE, NOT NULL**. The human-readable name of the tier (e.g., 'Platform Governor', 'Financial Governor'). | | **tier\_level** | `INTEGER` | **UNIQUE, NOT NULL**. The numerical rank (1-11) for hierarchical checks. | | **permissions** | `JSONB` | **NOT NULL**. Details the specific actions this tier is allowed to perform (e.g., `{"can_issue_refunds": true}`). | | **scope\_restrictions**| `JSONB` | Defines limitations, such as geographic or functional scopes (e.g., `{"region": "APAC"}`). |*  
* ***Key Relationships & Cardinality:***  
  * *`AdminTiers` **(1) \--- (N)** `Users`*

---

#### ***UserConsents***

* ***Core Purpose:** Explicitly tracks user consent for data processing, essential for GDPR and global compliance.*  
* ***Definition & Business Logic:** This table provides an immutable, timestamped record of a user's consent decisions. It is not a single "accept all" checkbox but a granular log of each specific `consent_type`. This allows users to opt-in to marketing while opting-out of analytics, for example. The `withdrawn_at` field is crucial for GDPR's "right to withdraw consent," providing a clear audit trail that the user's request was honored.*  
* ***Attributes:** | Column Name | Data Type | Constraints & Description | | :--- | :--- | :--- | | **consent\_id** | `UUID` | **Primary Key**. | | **user\_id** | `UUID` | **Foreign Key** \-\> `users(user_id)`, **\[idx\]**. The user this consent record belongs to. | | **consent\_type** | `ENUM` | **NOT NULL**. The specific type of consent, e.g., `'MARKETING_EMAILS'`, `'ANALYTICS_TRACKING'`. | | **granted** | `BOOLEAN` | **NOT NULL**. Current status of the consent. `true` if granted, `false` if withdrawn. | | **granted\_at** | `TIMESTAMPTZ`| The timestamp when consent was initially granted. | | **withdrawn\_at** | `TIMESTAMPTZ`| Nullable. The timestamp when consent was withdrawn. |*  
* ***Key Relationships & Cardinality:***  
  * *`Users` **(1) \--- (N)** `UserConsents`*

---

#### ***UserSessions***

* ***Core Purpose:** Manages active user sessions for enhanced security, allowing for features like "log out all other devices."*  
* ***Definition & Business Logic:** This table tracks every active login across all devices (web, mobile). The `device_fingerprint` and `ip_address` are used by the security service to detect suspicious activity, such as logins from multiple locations in a short time. The `is_revoked` flag allows a user or a `SuperAdmin` to remotely terminate a specific session, a critical feature for enterprise security.*  
* ***Attributes:** | Column Name | Data Type | Constraints & Description | | :--- | :--- | :--- | | **session\_id** | `UUID` | **Primary Key**. | | **user\_id** | `UUID` | **Foreign Key** \-\> `users(user_id)`. The user who owns this session. | | **device\_fingerprint**| `VARCHAR(255)` | A unique hash representing the user's browser and device combination. | | **ip\_address** | `INET` | The IP address from which the session is active. | | **last\_active** | `TIMESTAMPTZ`| Timestamp of the last user activity, used to time out idle sessions. | | **expires\_at** | `TIMESTAMPTZ`| **NOT NULL**. The timestamp when this session's token will automatically expire. | | **is\_revoked** | `BOOLEAN` | `DEFAULT FALSE`. Set to `true` to forcibly and immediately end a session. |*  
* ***Key Relationships & Cardinality:***  
  * *`Users` **(1) \--- (N)** `UserSessions`*

---

#### ***ApiKeys***

* ***Core Purpose:** Manages secure, scoped API access for third-party integrations, essential for our B2B and enterprise clients.*  
* ***Definition & Business Logic:** This table allows `Teams` to generate API keys for programmatic access to their data. Security is paramount: the actual key is shown only once upon creation, and only its `key_hash` is stored for verification. The `permissions` array enforces the principle of least privilege, ensuring a key generated for reading event data cannot be used to modify financial records.*  
* ***Attributes:** | Column Name | Data Type | Constraints & Description | | :--- | :--- | :--- | | **api\_key\_id** | `UUID` | **Primary Key**. | | **team\_id** | `UUID` | **Foreign Key** \-\> `teams(team_id)`. API keys are owned by `Teams`. | | **key\_hash** | `VARCHAR(255)` | **NOT NULL**. A secure bcrypt or Argon2 hash of the API key. | | **permissions** | `TEXT[]` | An array defining the scope (e.g., `['events:read', 'bookings:write']`). | | **rate\_limit\_per\_minute**| `INTEGER` | The number of requests allowed per minute to prevent abuse. |*  
* ***Key Relationships & Cardinality:***  
  * *`Teams` **(1) \--- (N)** `ApiKeys`*

---

#### ***SecurityAuditLog***

* ***Core Purpose:** An immutable, append-only log of all critical security-related events for forensic analysis and compliance.*  
* ***Definition & Business Logic:** This table is the platform's security diary. It is designed to be written to by various services but should be non-modifiable by almost any process. Every failed login, password reset, and permission change is recorded here. In the event of a security incident, this log is the primary source of truth for understanding what happened, when, and from where.*  
* ***Attributes:** | Column Name | Data Type | Constraints & Description | | :--- | :--- | :--- | | **audit\_id** | `BIGSERIAL` | **Primary Key**. An auto-incrementing integer for strict chronological ordering. | | **user\_id** | `UUID` | **Foreign Key** \-\> `users(user_id)`. Nullable for system-level events. | | **action\_type**| `ENUM` | **NOT NULL**. The type of security event, e.g., `'LOGIN_SUCCESS'`, `'PERMISSION_CHANGED'`. | | **ip\_address** | `INET` | The source IP address of the action. | | **additional\_data**| `JSONB` | Stores context-specific data, like the permission that was changed. |*  
* ***Key Relationships & Cardinality:***  
  * *`Users` **(1) \--- (N)** `SecurityAuditLog`*

---

#### ***EntityAuditLog***

* ***Core Purpose:** A universal "time machine" for platform data, recording the "before" and "after" state for every significant data change.*  
* ***Definition & Business Logic:** This is one of the most critical tables for data integrity and dispute resolution. Using database triggers, every `UPDATE` or `SOFT_DELETE` on a major business entity (like `Events` or `Bookings`) automatically creates a record here. The `old_values` and `new_values` JSONB blobs provide a complete snapshot, allowing support staff or admins to see the exact history of any record.*  
* ***Attributes:** | Column Name | Data Type | Constraints & Description | | :--- | :--- | :--- | | **audit\_id** | `BIGSERIAL` | **Primary Key**. | | **entity\_id** | `UUID` | **P\_FK, \[idx\]**. The ID of the record that was changed. | | **entity\_type**| `VARCHAR(50)` | **P\_FK, \[idx\]**. The table name of the record (e.g., 'events'). | | **operation** | `ENUM` | `'CREATE'`, `'UPDATE'`, `'SOFT_DELETE'`. | | **changed\_by\_user\_id**| `UUID` | **Foreign Key** \-\> `users(user_id)`. The user who performed the action. | | **old\_values** | `JSONB` | A JSON snapshot of the record before the change. | | **new\_values** | `JSONB` | A JSON snapshot of the record after the change. |*  
* ***Key Relationships & Cardinality:***  
  * *Any major entity **(1) \--- (N)** `EntityAuditLog` (via polymorphic relationship).*

---

#### ***DataGovernance (`DataRetentionPolicies` & `DataExportRequests`)***

* ***Core Purpose:** Provides the infrastructure for managing the data lifecycle and complying with global regulations like GDPR.*  
* ***Definition & Business Logic:** `DataRetentionPolicies` is a configuration table used by automated cleanup jobs to enforce our data lifecycle policy (e.g., "delete inactive user sessions after 90 days"). `DataExportRequests` operationalizes the "right to data portability," creating a trackable, auditable queue for user data export requests.*  
* ***Attributes & Relationships:***  
  * ***DataRetentionPolicies:** `policy_id` (PK), `entity_type`, `retention_period_days`.*  
  * ***DataExportRequests:** `request_id` (PK), `user_id` (FK) \-\> `Users`, `status`, `export_data_url`.*  
  * *`Users` **(1) \--- (N)** `DataExportRequests`.*

---

#### ***AiUsageLogs***

* ***Core Purpose:** Provides detailed auditing and cost-tracking for every call made to an external AI/LLM provider.*  
* ***Definition & Business Logic:** This is the financial and operational meter for our AI-First strategy. Every time a feature (like a **Genesis** tool) calls an external LLM, a record is created here. This allows us to track costs precisely, bill users for on-demand usage via the `Wallet`, monitor which models are most effective, and identify potential abuse of the AI systems.*  
* ***Attributes:** | Column Name | Data Type | Constraints & Description | | :--- | :--- | :--- | | **usage\_id** | `UUID` | **Primary Key**. | | **user\_id** | `UUID` | **Foreign Key** \-\> `users(user_id)`. The end-user who triggered the operation. | | **team\_id** | `UUID` | **Foreign Key** \-\> `teams(team_id)`. The team whose subscription credits are being used. | | **llm\_provider** | `ENUM` | `'OpenAI'`, `'Google'`, `'Anthropic'`, etc. | | **tokens\_consumed**| `INTEGER` | **NOT NULL**. Total input \+ output tokens for billing. | | **cost\_credits**| `DECIMAL` | **NOT NULL**. The cost of this operation in EventfulHQ credits. | | **operation\_type**| `ENUM` | `'content_generation'`, `'translation'`, `'analysis'`, etc. The type of AI task. |*  
* ***Key Relationships & Cardinality:***  
  * *`Users` **(1) \--- (N)** `AiUsageLogs`*  
  * *`Teams` **(1) \--- (N)** `AiUsageLogs`*

---

### ***Schema 2: Identity & Collaboration Domain***

***Architectural Philosophy:** This domain is the authoritative source for all group and collaborative structures on the platform. It defines what a `Team` is, who belongs to it, and the specialized attributes of our core user archetypes (`Artist`, `Expert`, `Fan`). It is designed to be highly scalable to support millions of teams and profiles, from informal C2C committees to large enterprise clients, and includes the full `[BaseEntity]` definition for auditable, soft-deletable records.*

---

#### ***Teams `[BaseEntity]`***

* ***Core Purpose:** The `Teams` entity is the core collaborative and commercial unit for any group. It acts as a central owner for projects, financial accounts, brand identities, and API access.*  
* ***Definition & Business Logic:** This table represents any formal or informal group of users working together. The `team_type` allows the platform to understand the team's nature—a `'Corporate'` team has different needs and will be recommended different features than an artist's `'Collective'`. A `Team` is the primary entity for our B2B users, owning `Events`, `Campaigns`, and managing subscriptions. The link to a `creator_user_id` and the full `[BaseEntity]` fields ensure a complete audit trail of the team's lifecycle and management.*  
* ***Attributes:** | Column Name | Data Type | Constraints & Description | | :--- | :--- | :--- | | **team\_id** | `UUID` | **Primary Key**, `DEFAULT gen_random_uuid()`. A unique, system-generated identifier for the team. | | **team\_name** | `VARCHAR(255)` | **NOT NULL**. The public-facing name of the team. | | **team\_type** | `ENUM` | **NOT NULL**. Classifies the team's operational structure, e.g., `'event_organizer'`, `'venue'`, `'agency'`, `'corporate'`, etc.. | | **description** | `TEXT` | A detailed description of the team, its mission, and services, used for discovery and profiles. | | **website\_url** | `VARCHAR(500)` | The team's official website. | | **logo\_url** | `VARCHAR(500)` | URL for the team's logo, managed by the Media & Content Domain. | | **verification\_status**| `ENUM` | `DEFAULT 'unverified'`. The trust level of the team, e.g., `'unverified'`, `'pending'`, `'verified'`. Verified teams are prioritized in search results. | | **...** | | Includes all `[BaseEntity]` audit and soft-delete fields as specified (`created_at`, `is_deleted`, etc.).. |*  
* ***Key Relationships & Cardinality:***  
  * *`Teams` **(1) \--- (N)** `TeamMemberships`*  
  * *`Teams` **(1) \--- (1)** `Wallets` (via polymorphic link)*  
  * *`Teams` **(1) \--- (N)** `Events`, `Tours`, `Campaigns` (as owner)*  
  * *`Users` **(1) \--- (N)** `Teams` (as creator)*  
* ***Recommended Indexes:** `CREATE INDEX idx_teams_type ON teams(team_type) WHERE NOT is_deleted;`*

---

#### ***TeamMemberships (Junction Table)***

* ***Core Purpose:** This table resolves the critical many-to-many relationship between `Users` and `Teams`. It is the definitive record of who belongs to which team and in what capacity.*  
* ***Definition & Business Logic:** This is a crucial junction table for collaboration. It defines a user's role within the context of a specific team. A user could be an `'Owner'` of their own small agency `Team` while also being a `'Member'` of a larger `Team` they collaborate with. The `invitation_status` and `invited_by_user_id` fields provide a clear, auditable trail for how and when users join a team. The `UNIQUE` constraint on (`user_id`, `team_id`) ensures a user cannot have multiple roles within the same team.*  
* ***Attributes:** | Column Name | Data Type | Constraints & Description | | :--- | :--- | :--- | | **membership\_id** | `UUID` | **Primary Key**, `DEFAULT gen_random_uuid()`. The unique identifier for the membership record. | | **user\_id** | `UUID` | **Foreign Key** \-\> `users(user_id)`, **\[idx\]**. Links to the `User` who is a member. Part of a `UNIQUE` composite key. | | **team\_id** | `UUID` | **Foreign Key** \-\> `teams(team_id)`, **\[idx\]**. Links to the `Team` they belong to. Part of a `UNIQUE` composite key. | | **role** | `ENUM` | **NOT NULL**, `DEFAULT 'member'`. The user's role and base level of permission within that specific team, e.g., `'owner'`, `'admin'`, `'manager'`, `'member'`. | | **invitation\_status**| `ENUM` | `DEFAULT 'pending'`. Tracks the state of the user's invitation, e.g., `'pending'`, `'accepted'`, `'declined'`. | | **invited\_by\_user\_id**| `UUID` | **Foreign Key** \-\> `users(user_id)`. Audits which user sent the invitation. | | **joined\_at** | `TIMESTAMPTZ`| Nullable. The timestamp when the user accepted the invitation and officially joined the team. | | **is\_active** | `BOOLEAN` | `DEFAULT TRUE`. Allows an admin to suspend a member's access without deleting the record. |*  
* ***Key Relationships & Cardinality:***  
  * *`Users` **(1) \--- (N)** `TeamMemberships`*  
  * *`Teams` **(1) \--- (N)** `TeamMemberships`*  
* ***Recommended Indexes:** `CREATE INDEX idx_teammemberships_user_team ON team_memberships(user_id, team_id);`*

---

#### ***UserProfiles (1:1 with Users)***

* ***Core Purpose:** This group of tables extends the base `Users` entity to hold the rich, specialized data for each user archetype.*  
* ***Definition & Business Logic:** This 1:1 relationship is an architectural choice for performance and data clarity. It keeps the core `Users` table lean and fast for authentication, while offloading detailed, archetype-specific data to these dedicated tables. A `PRIMARY KEY` constraint on `user_id` in each of these tables enforces the strict 1:1 relationship, ensuring a user can only have one specialized profile.*

---

##### ***ArtistProfiles***

* ***Definition & Business Logic:** This table holds all the professional information for a `User` with the `'Artist'` archetype. It contains the data needed for the **Spotlight** marketplace, including their skills, pricing, and technical requirements. The `technical_rider_url` is a key field that our AI can parse to auto-populate budget line items in the **Ledger** app, delivering on our "low-click" promise.*  
* ***Attributes:** | Column Name | Data Type | Constraints & Description | | :--- | :--- | :--- | | **user\_id** | `UUID` | **Primary Key**, **Foreign Key** \-\> `users(user_id)`. This enforces the 1:1 relationship. | | **artist\_name**| `VARCHAR(255)`| The artist's public stage name, which may differ from their legal name in the `Users` table. | | **bio** | `TEXT` | A detailed biography used on their public profile and for event organizers to review. | | **genre\_primary**| `VARCHAR(100)`| The main genre of the artist (e.g., 'Electronic', 'Rock', 'Keynote Speaker'). | | **technical\_rider\_url**| `VARCHAR(500)`| A URL to the artist's PDF technical rider. | | **base\_fee\_credits** | `DECIMAL` | The artist's typical starting fee in platform credits, used for filtering in the marketplace. | | **is\_verified** | `BOOLEAN` | `DEFAULT FALSE`. A flag set by `SuperAdmins` to indicate a high-profile, authenticated artist. |*

---

##### ***ExpertProfiles***

* ***Definition & Business Logic:** This table holds the data for a `User` with the `'Expert'` archetype, powering the **Craftwork** marketplace. The `core_competencies` array is the most critical field, containing tags from our global `Expert Taxonomy`. This array is the primary data source for our AI's "Competency-to-Task Matching" algorithm, which intelligently connects an expert's specific skills to an organizer's needs.*  
* ***Attributes:** | Column Name | Data Type | Constraints & Description | | :--- | :--- | :--- | | **user\_id** | `UUID` | **Primary Key**, **Foreign Key** \-\> `users(user_id)`. This enforces the 1:1 relationship. | | **professional\_title**| `VARCHAR(255)`| The expert's professional title (e.g., 'Senior Event Producer', 'Lighting Designer'). | | **company\_name** | `VARCHAR(255)`| The name of the expert's company, if they operate as a business. | | **core\_competencies**| `VARCHAR(100)[]`| **NOT NULL**. An array of skills from our taxonomy (e.g., `'corporate_event_management'`, `'risk_assessment'`). | | **hourly\_rate\_credits**| `DECIMAL` | The expert's standard hourly rate in platform credits. | | **portfolio\_url**| `VARCHAR(500)`| A link to their external portfolio or website. | | **is\_green\_certified**| `BOOLEAN` | `DEFAULT FALSE`. A flag used by the **Veridian** app to identify sustainable service providers. |*

---

##### ***FanProfiles***

* ***Definition & Business Logic:** This table holds the data for a `User` with the `'Fan'` archetype. It is the core of our community and gamification system. The `contribution_score` and `reward_points` are updated by the **Rewards** engine based on user actions across the platform (e.g., writing reviews, creating C2C events). The `loyalty_tier` is derived from the `contribution_score` and unlocks special perks, creating a powerful incentive for users to engage with and enrich the ecosystem.*  
* ***Attributes:** | Column Name | Data Type | Constraints & Description | | :--- | :--- | :--- | | **user\_id** | `UUID` | **Primary Key**, **Foreign Key** \-\> `users(user_id)`. This enforces the 1:1 relationship. | | **display\_name** | `VARCHAR(100)`| The fan's public-facing username in the community. | | **bio** | `TEXT` | A short bio for their community profile. | | **favorite\_genres**| `VARCHAR(100)[]`| An array of genres used to power recommendations in **Spotlight** and **Stream**. | | **contribution\_score**| `INTEGER` | `DEFAULT 0`. The user's non-redeemable "experience points" (XP). | | **reward\_points**| `INTEGER` | `DEFAULT 0`. The user's redeemable points for use in the **Rewards** store. | | **loyalty\_tier**| `ENUM` | `DEFAULT 'bronze'`. The fan's community status, e.g., `'bronze'`, `'silver'`, `'gold'`. | | **dietary\_restrictions**| `TEXT[]` | An array of dietary needs used by the **Feast** app for aggregated, anonymized reporting to caterers. | | **accessibility\_needs**| `TEXT[]` | An array of accessibility needs to help organizers and venues accommodate them better. |*

---

### ***Schema 3: Finance Domain***

***Architectural Philosophy:** This domain is the financial kernel of the platform. It is designed with banking-grade principles, ensuring every transaction is atomic, auditable, and immutable. It provides the foundation for all monetization features, from ticket sales and SaaS subscriptions to on-demand AI tool usage. The schema separates the concept of a `Wallet` (internal value store) from `PaymentMethods` (external funding sources) and `Invoices` (billing documents), creating a clean, enterprise-ready financial architecture.*

---

#### ***Wallets `[BaseEntity]`***

* ***Core Purpose:** The `Wallets` entity is a universal financial account that can be owned by either a `User` or a `Team`, holding the platform's native credits.*  
* ***Definition & Business Logic:** This table is the central ledger for all stored value within the EventfulHQ ecosystem. Every transactional entity (`User` or `Team`) is issued a `Wallet` to create a clean financial boundary. The polymorphic ownership (`owner_id`, `owner_type`) allows a single table to manage balances for both individuals and organizations. The separation of balances (`available`, `pending`, `frozen`) is critical for handling complex workflows like escrow for bookings, where funds are held (`pending`) until a service is completed. All updates to balance fields must be atomic to prevent race conditions and ensure data integrity.*  
* ***Attributes:** | Column Name | Data Type | Constraints & Description | | :--- | :--- | :--- | | **wallet\_id** | `UUID` | **Primary Key**, `DEFAULT gen_random_uuid()`. Unique ID for the financial account. | | **user\_id** | `UUID` | **Foreign Key** \-\> `users(user_id)`. The owning user. Part of a `CHECK` constraint to ensure single ownership type. | | **team\_id** | `UUID` | **Foreign Key** \-\> `teams(team_id)`. The owning team. Part of a `CHECK` constraint. | | **wallet\_type** | `ENUM` | **NOT NULL**, `DEFAULT 'user_personal'`. Categorizes the wallet's purpose (e.g., `'user_personal'`, `'team_business'`, `'escrow'`). | | **currency\_code** | `VARCHAR(3)` | **NOT NULL**, `DEFAULT 'USD'`. The ISO currency code this wallet operates in. | | **available\_balance**| `DECIMAL(15,2)`| **NOT NULL**, `DEFAULT 0`. The balance that is immediately available for spending. | | **pending\_balance**| `DECIMAL(15,2)`| **NOT NULL**, `DEFAULT 0`. Funds that are incoming but not yet cleared (e.g., in-transit payouts). | | **frozen\_balance** | `DECIMAL(15,2)`| **NOT NULL**, `DEFAULT 0`. Funds held for a specific purpose, like a security deposit or in escrow for a booking. | | **status** | `ENUM` | `DEFAULT 'active'`. The operational status of the wallet (`'active'`, `'suspended'`, `'closed'`). | | **...** | | Includes all `[BaseEntity]` audit and soft-delete fields (`created_at`, `is_deleted`, etc.). |*  
* ***Key Relationships & Cardinality:***  
  * *`Users` **(1) \--- (1..N)** `Wallets` (A User has at least one personal wallet, potentially more for different currencies).*  
  * *`Teams` **(1) \--- (1..N)** `Wallets` (A Team has at least one business wallet).*  
  * *`Wallets` **(1) \--- (N)** `Transactions` (as either source or destination).*  
* ***Recommended Indexes:** `UNIQUE(user_id, currency_code)`, `UNIQUE(team_id, currency_code)`.*

---

#### ***Transactions `[BaseEntity]`***

* ***Core Purpose:** This table is the immutable, append-only ledger of all financial activity on the platform, providing a perfect, verifiable audit trail.*  
* ***Definition & Business Logic:** Following Event Sourcing principles, this table provides the complete, chronological history of every credit that moves through the system. It is designed to be the single source of truth for all financial reporting and dispute resolution. The `transaction_type` enum allows for clear categorization of fund movements, distinguishing between a simple `'transfer'`, a `'fee'` taken by the platform, or an `'escrow_hold'`. The link to an `external_transaction_id` from a payment gateway like Stripe provides end-to-end traceability from an external charge to internal credit allocation.*  
* ***Attributes:** | Column Name | Data Type | Constraints & Description | | :--- | :--- | :--- | | **transaction\_id** | `UUID` | **Primary Key**, `DEFAULT gen_random_uuid()`. | | **transaction\_reference**| `VARCHAR(100)`| **UNIQUE, NOT NULL**. A human-readable, unique reference code for the transaction. | | **transaction\_type** | `ENUM` | **NOT NULL**. The business logic category of the transaction (e.g., `'credit'`, `'debit'`, `'escrow_hold'`, `'refund'`). | | **from\_wallet\_id** | `UUID` | **Foreign Key** \-\> `wallets(wallet_id)`. The source of the funds. Can be NULL for credit purchases. | | **to\_wallet\_id** | `UUID` | **Foreign Key** \-\> `wallets(wallet_id)`. The destination of the funds. Can be NULL for credit spends. | | **amount** | `DECIMAL(15,2)`| **NOT NULL**. The gross amount of the transaction. | | **platform\_fee** | `DECIMAL(15,2)`| `DEFAULT 0`. The fee retained by EventfulHQ for facilitating the transaction. | | **processing\_fee** | `DECIMAL(15,2)`| `DEFAULT 0`. The fee passed on from the external payment gateway. | | **related\_entity\_id**| `UUID` | **P\_FK**. Polymorphic link to the entity that triggered this transaction (e.g., a `booking_id`, `subscription_id`). | | **related\_entity\_type**| `ENUM` | **P\_FK**. The type of the related entity. | | **status** | `ENUM` | **NOT NULL**, `DEFAULT 'pending'`. The lifecycle status of the transaction (`'pending'`, `'processing'`, `'completed'`, `'failed'`). | | **external\_transaction\_id**| `VARCHAR(255)`| The transaction ID from an external gateway like Stripe or PayPal for reconciliation. | | **...** | | Includes all `[BaseEntity]` audit and soft-delete fields. |*  
* ***Key Relationships & Cardinality:***  
  * *`Wallets` **(1) \--- (N)** `Transactions`.*  
  * *`Bookings`, `Subscriptions`, `Tickets`, etc. **(1) \--- (1..N)** `Transactions` (via polymorphic link).*  
* ***Recommended Indexes:** `CREATE INDEX idx_transactions_from_wallet ON transactions(from_wallet_id);`, `CREATE INDEX idx_transactions_to_wallet ON transactions(to_wallet_id);`, `CREATE INDEX idx_transactions_entity ON transactions(related_entity_id, related_entity_type);`*

---

#### ***PaymentMethods `[BaseEntity]`***

* ***Core Purpose:** Securely stores tokenized payment methods (credit cards, bank accounts) for users and teams, enabling seamless checkout and subscription billing.*  
* ***Definition & Business Logic:** This table **never** stores raw, sensitive payment information. It holds a non-sensitive reference (`external_id`) which is a token provided by a secure payment gateway (e.g., Stripe, Razorpay). This architecture outsources PCI compliance to the payment provider. A `User` or `Team` can add multiple payment methods and designate one as the `is_default` for automatic billing of their SaaS `Subscription`.*  
* ***Attributes:** | Column Name | Data Type | Constraints & Description | | :--- | :--- | :--- | | **payment\_method\_id** | `UUID` | **Primary Key**, `DEFAULT gen_random_uuid()`. | | **user\_id** | `UUID` | **Foreign Key** \-\> `users(user_id)`. The owning user. Part of a `CHECK` constraint. | | **team\_id** | `UUID` | **Foreign Key** \-\> `teams(team_id)`. The owning team. Part of a `CHECK` constraint. | | **method\_type** | `ENUM` | **NOT NULL**. The type of payment method, e.g., `'card'`, `'bank_account'`. | | **provider** | `ENUM` | **NOT NULL**. The payment gateway that holds the actual details, e.g., `'stripe'`, `'razorpay'`. | | **external\_id** | `VARCHAR(255)`| **NOT NULL**. The token/ID from the provider that represents the stored payment method. | | **last\_four\_digits**| `VARCHAR(4)` | The last four digits of the card or account number for display purposes. | | **brand** | `VARCHAR(50)` | The card brand, e.g., 'Visa', 'Mastercard'. | | **is\_default** | `BOOLEAN` | `DEFAULT FALSE`. Indicates if this is the default payment method for the owner. | | **status** | `ENUM` | `DEFAULT 'active'`. Status of the payment method, e.g., `'active'`, `'expired'`, `'removed'`. | | **...** | | Includes all `[BaseEntity]` audit and soft-delete fields. |*  
* ***Key Relationships & Cardinality:***  
  * *`Users` **(1) \--- (N)** `PaymentMethods`.*  
  * *`Teams` **(1) \--- (N)** `PaymentMethods`.*  
  * *`Subscriptions` **(N) \--- (1)** `PaymentMethods`.*

---

#### ***Invoices `[BaseEntity]`***

* ***Core Purpose:** Manages the creation, status tracking, and archival of invoices for B2B transactions and SaaS subscription billing.*  
* ***Definition & Business Logic:** This table provides the formal billing document for platform services. It's crucial for our B2B clients who require detailed invoices for their accounting. The structure supports billing from the platform (for SaaS subscriptions) and can be extended to allow billing between users (e.g., a `Venue` team invoicing a corporate `Team` for services). The `status` enum (`'draft'`, `'sent'`, `'paid'`, `'overdue'`) drives the automated billing and reminder workflows.*  
* ***Attributes:** | Column Name | Data Type | Constraints & Description | | :--- | :--- | :--- | | **invoice\_id** | `UUID` | **Primary Key**, `DEFAULT gen_random_uuid()`. | | **invoice\_number** | `VARCHAR(100)`| **UNIQUE, NOT NULL**. A human-readable, sequential invoice number. | | **billed\_to\_user\_id** | `UUID` | **Foreign Key** \-\> `users(user_id)`. The user receiving the invoice. | | **billed\_to\_team\_id** | `UUID` | **Foreign Key** \-\> `teams(team_id)`. The team receiving the invoice. | | **subtotal** | `DECIMAL(15,2)`| **NOT NULL**. The total amount before taxes and discounts. | | **tax\_amount** | `DECIMAL(15,2)`| `DEFAULT 0`. Applicable taxes. | | **total\_amount** | `DECIMAL(15,2)`| **NOT NULL**. The final amount due. | | **currency\_code** | `VARCHAR(3)` | **NOT NULL**, `DEFAULT 'USD'`. | | **status** | `ENUM` | `DEFAULT 'draft'`. The current status of the invoice lifecycle. | | **due\_date** | `DATE` | **NOT NULL**. The date by which payment is due. | | **paid\_at** | `TIMESTAMPTZ`| Nullable. Timestamp when the invoice was fully paid. | | **pdf\_url** | `VARCHAR(500)`| A secure URL to the generated PDF version of the invoice. | | **...** | | Includes all `[BaseEntity]` audit and soft-delete fields. |*  
* ***Key Relationships & Cardinality:***  
  * *`Subscriptions` **(1) \--- (N)** `Invoices`.*  
  * *`Users` or `Teams` **(1) \--- (N)** `Invoices` (as the billed party).*

---

### ***Schema 4: VETC Product Domain***

***Architectural Philosophy:** This domain defines the core "products" of the EventfulHQ platform: the **V**enues, **E**vents, **T**ours, and **C**ampaigns that our users create, manage, and promote. These entities are the central nodes of our knowledge graph. The schema uses typed foreign keys for ownership to enforce data integrity, as you recommended, and is fully integrated with our global taxonomy and translation frameworks to support a worldwide user base.*

---

#### ***EventCategories (Taxonomy Table)***

* ***Core Purpose:** This table serves as the single source of truth for our proprietary global events taxonomy, consisting of 15 main categories and 44 subcategories.*  
* ***Definition & Business Logic:** This is a crucial lookup table that powers the AI's classification and recommendation engines. When a user creates an event, its `category_id` links here, giving the platform deep contextual understanding of the event's nature. The `skills_required` array informs the **Craftwork** marketplace which `Expert` competencies are relevant for this type of event, creating a direct link between event type and the professionals needed to execute it.*  
* ***Attributes:** | Column Name | Data Type | Constraints & Description | | :--- | :--- | :--- | | **category\_id** | `UUID` | **Primary Key**, `DEFAULT gen_random_uuid()`. Unique ID for a specific subcategory. | | **category\_name** | `VARCHAR(100)` | **NOT NULL**. The name of the main category (e.g., 'Corporate & Business Events'). | | **subcategory\_name** | `VARCHAR(100)` | The name of the specific subcategory (e.g., 'Internal Corporate Events'). | | **taxonomy\_code** | `VARCHAR(20)` | **UNIQUE, NOT NULL**. A unique, human-readable code for programmatic access (e.g., 'CORP\_1\_1'). | | **skills\_required**| `TEXT[]` | An array of `core_competency` tags from the `ExpertProfiles` taxonomy that are typically required for this event type. |*  
* ***Key Relationships & Cardinality:***  
  * *`EventCategories` **(1) \--- (N)** `Events`*

---

#### ***Events (Event Editions) `[BaseEntity]`***

* ***Core Purpose:** The `Events` entity is the anchor point of the entire platform, representing a specific gathering at a specific time and place.*  
* ***Definition & Business Logic:** This is the central hub to which venues, talent, attendees, budgets, and campaigns are connected. The strict `CHECK` constraint on ownership ensures that every event has exactly one owner—either a `Team` or a `User`—preventing data ambiguity. The optional link to a `parent_event_brand_id` allows for the management of recurring event series (e.g., the 2024 and 2025 editions of "AI Innovate Conference" can both link to the same "AI Innovate" `Brand`).*  
* ***Attributes:** | Column Name | Data Type | Constraints & Description | | :--- | :--- | :--- | | **event\_edition\_id** | `UUID` | **Primary Key**, `DEFAULT gen_random_uuid()`. | | **edition\_title** | `VARCHAR(255)` | **NOT NULL**. The public-facing name of the event (e.g., "AI Innovate Conference 2025"). | | **category\_id** | `UUID` | **Foreign Key** \-\> `event_categories(category_id)`. **NOT NULL**. Links the event to its specific taxonomy classification. | | **status** | `ENUM` | `DEFAULT 'draft'`. The lifecycle status of the event, e.g., `'draft'`, `'published'`, `'completed'`, `'cancelled'`. | | **start\_datetime**| `TIMESTAMPTZ`| **NOT NULL**. The official start time of the event in UTC. | | **end\_datetime** | `TIMESTAMPTZ`| **NOT NULL**. The official end time of the event in UTC. `CONSTRAINT check_datetime CHECK (end_datetime > start_datetime)`. | | **owner\_team\_id** | `UUID` | **Foreign Key** \-\> `teams(team_id)`. Nullable. The ID of the `Team` that owns this event. | | **owner\_user\_id** | `UUID` | **Foreign Key** \-\> `users(user_id)`. Nullable. The ID of the `User` who owns this event. | | **tour\_edition\_id**| `UUID` | **Foreign Key** \-\> `tours(tour_id)`. Nullable. Links this event to a parent `Tour`. | | **parent\_event\_brand\_id**| `UUID` | **Foreign Key** \-\> `brands(brand_id)`. Nullable. Links this event to a recurring brand identity. | | **...** | | Includes all `[BaseEntity]` audit and soft-delete fields. |*  
* ***Key Relationships & Cardinality:***  
  * *`Events` **(1) \--- (N)** `Bookings`, `Sponsorships`, `Exhibitors`, `Tickets`, `EventStaff`.*  
  * *`Events` **(N) \--- (1)** `EventCategories`, `Tours`, `Brands`.*  
  * *`Teams` or `Users` **(1) \--- (N)** `Events` (as owner).*  
* ***Recommended Indexes:** `CREATE INDEX idx_events_start_datetime ON events(start_datetime) WHERE NOT is_deleted;`, `CREATE INDEX idx_events_category ON events(category_id) WHERE NOT is_deleted;`*

---

#### ***Venues `[BaseEntity]`***

* ***Core Purpose:** Represents any physical location where an event can be hosted, grounded by a mandatory link to the Google Places API for data integrity.*  
* ***Definition & Business Logic:** This table models the physical containers for experiences. The `google_place_id` was a foundational architectural decision to prevent duplicates and auto-populate address data. The ownership constraint (`owner_team_id` or `owner_user_id`) allows venue managers to claim and manage their profiles on the **Haven** marketplace.*  
* ***Attributes:** | Column Name | Data Type | Constraints & Description | | :--- | :--- | :--- | | **venue\_id** | `UUID` | **Primary Key**, `DEFAULT gen_random_uuid()`. | | **name** | `VARCHAR(255)` | **NOT NULL**. The common name of the venue. | | **venue\_type** | `ENUM` | **NOT NULL**. Classifies the venue, e.g., `'conference_center'`, `'outdoor_space'`, `'hotel'`. | | **max\_capacity** | `INTEGER` | **NOT NULL**. The maximum legal capacity of the venue. | | **address\_line1**| `VARCHAR(255)`| The physical street address. | | **country** | `VARCHAR(2)` | The ISO 3166-1 alpha-2 country code. | | **coordinates** | `POINT` | PostGIS data type for efficient geographical queries. | | **destination\_id** | `UUID` | **Foreign Key** \-\> `destinations(destination_id)`. Links the venue to its geographical `Destination`. | | **...** | | Includes all `[BaseEntity]` audit and soft-delete fields. |*  
* ***Key Relationships & Cardinality:***  
  * *`Venues` **(N) \--- (M)** `Events` (via a junction table).*  
  * *`Venues` **(N) \--- (1)** `Destinations`.*

---

#### ***Tours (Tour Editions) `[BaseEntity]`***

* ***Core Purpose:** Acts as a container for a series of linked events that form a single, packaged experience.*  
* ***Definition & Business Logic:** This entity allows organizers to manage complex, multi-day or multi-city experiences as a single product. For example, a "Concert Tour" is a `Tour` entity that serves as the parent for ten individual `Event` entities, one for each city's concert. This allows for master-level management of logistics, budget, and marketing.*  
* ***Attributes:** | Column Name | Data Type | Constraints & Description | | :--- | :--- | :--- | | **tour\_id** | UUID | **Primary Key**, DEFAULT gen\_random\_uuid(). | | **title** | VARCHAR(255) | **NOT NULL**. The public name of the tour. | | **status** | ENUM | DEFAULT 'planning'. The lifecycle status, e.g., 'planning', 'active', 'completed'. | | **start\_date** | DATE | **NOT NULL**. The start date of the entire tour. | | **end\_date** | DATE | **NOT NULL**. The end date of the entire tour. CONSTRAINT check\_tour\_dates CHECK (end\_date \>= start\_date). | | **owner\_team\_id** | UUID | **Foreign Key** \-\> teams(team\_id). Nullable. The owning Team. | | **owner\_user\_id** | UUID | **Foreign Key** \-\> users(user\_id). Nullable. The owning User. | | **...** | | Includes all \[BaseEntity\] audit and soft-delete fields. |*  
* ***Key Relationships & Cardinality:***  
  * *Tours **(1) \--- (N)** Events.*

---

#### ***Campaigns \[BaseEntity\]***

* ***Core Purpose:** The platform's communication engine, representing a purpose-driven initiative with a defined objective.*  
* ***Definition & Business Logic:** A Campaign is a flexible object designed to promote almost any other entity on the platform. The polymorphic association (associated\_entity\_id, associated\_entity\_type) is a key architectural feature that allows a user to launch a marketing campaign for their Event, a PR campaign for their Brand, or an influencer campaign for their Tour, all using the same underlying structure.*  
* ***Attributes:** | Column Name | Data Type | Constraints & Description | | :--- | :--- | :--- | | **campaign\_id** | UUID | **Primary Key**, DEFAULT gen\_random\_uuid(). | | **name** | VARCHAR(255) | **NOT NULL**. The name of the campaign. | | **campaign\_type** | ENUM | **NOT NULL**. The campaign classification, e.g., 'event\_promotion', 'brand\_activation'. | | **associated\_entity\_id**| UUID | **P\_FK, \[idx\]**. The ID of the entity being promoted. | | **associated\_entity\_type**| VARCHAR(50)| **P\_FK, \[idx\]**. The type of the entity being promoted, e.g., 'Event', 'Tour'. | | **...** | | Includes all \[BaseEntity\] audit and soft-delete fields. |*  
* ***Key Relationships & Cardinality:***  
  * *Campaigns **(N) \--- (1)** Events, Tours, Venues, Brands (via polymorphic link).*

---

#### ***EntityTranslations***

* ***Core Purpose:** The backbone of our multi-language support, allowing key text fields from VETC entities to be stored in multiple languages.*  
* ***Definition & Business Logic:** This table enables our platform to be truly global. When a user views an Event page, the application backend will query this table for translations of fields like 'title' and 'description' in the user's language\_preference. The UNIQUE constraint is critical to prevent duplicate translations for the same field on the same entity.*  
* ***Attributes:** | Column Name | Data Type | Constraints & Description | | :--- | :--- | :--- | | **translation\_id** | UUID | **Primary Key**, DEFAULT gen\_random\_uuid(). | | **entity\_id** | UUID | **P\_FK**. The ID of the record being translated. Part of a UNIQUE composite key. | | **entity\_type**| VARCHAR(50) | **P\_FK**. The table name of the record. Part of a UNIQUE composite key. | | **field\_name** | VARCHAR(100)| **NOT NULL**. The column name being translated (e.g., 'title', 'description'). Part of a UNIQUE composite key. | | **language\_code**| VARCHAR(5) | **NOT NULL**. The ISO language code (e.g., 'en', 'hi', 'ar'). Part of a UNIQUE composite key. | | **translated\_text**| TEXT | **NOT NULL**. The translated content. |*  
* ***Key Relationships & Cardinality:***  
  * *Any VETC entity **(1) \--- (N)** EntityTranslations.*

---

#### ***Destinations***

* ***Core Purpose:** The foundational geographical pillar, providing regional context for Venues and Events.*  
* ***Definition & Business Logic:** This entity models the geographical world. The addition of currency\_code, language\_codes, timezone, and region is an enterprise-grade enhancement that allows the platform to personalize the user experience based on location. For example, when viewing an Event in a Destination with region: 'europe', the platform can automatically display prices in EUR, even if the platform's default is USD.*  
* ***Attributes:** | Column Name | Data Type | Constraints & Description | | :--- | :--- | :--- | | **destination\_id** | UUID | **Primary Key**, DEFAULT gen\_random\_uuid(). | | **name** | VARCHAR(255) | The common name of the destination (e.g., "Udaipur"). | | **city** | VARCHAR(100) | The primary city. | | **country** | VARCHAR(2) | The ISO 3166-1 alpha-2 country code. | | **currency\_code**| VARCHAR(3) | **NOT NULL**. The primary local currency (e.g., 'INR', 'USD', 'AED'). | | **language\_codes**| VARCHAR(5)\[\]| An array of official languages spoken (e.g., {'en', 'hi'}). | | **timezone** | VARCHAR(50) | **NOT NULL**. The IANA time zone name (e.g., 'Asia/Kolkata'). | | **region** | ENUM | **NOT NULL**. The global region for analytics and regional management, e.g., 'americas', 'asia\_pacific'. |*  
* ***Key Relationships & Cardinality:***  
  * *Destinations **(1) \--- (N)** Venues.*

---

#### ***Brands \[BaseEntity\]***

* ***Core Purpose:** Allows a Team to manage multiple distinct commercial event identities.*  
* ***Definition & Business Logic:** This entity solves a key problem for large event companies. A single Team (e.g., "Superstruct Entertainment") can own multiple distinct festival Brands (e.g., "Sziget Festival," "Wacken Open Air"). This allows for brand-specific analytics, marketing, and sponsorship while maintaining a single administrative and financial owner at the Team level.*  
* ***Attributes:** | Column Name | Data Type | Constraints & Description | | :--- | :--- | :--- | | **brand\_id** | UUID | **Primary Key**, DEFAULT gen\_random\_uuid(). | | **brand\_name** | VARCHAR(255) | **NOT NULL**. The name of the brand (e.g., "Oktoberfest", "Sunburn Festival"). | | **owning\_team\_id** | UUID | **Foreign Key** \-\> teams(team\_id). **NOT NULL**. The Team that owns this brand. | | **...** | | Includes all \[BaseEntity\] audit and soft-delete fields. |*  
* ***Key Relationships & Cardinality:***  
  * *Teams **(1) \--- (N)** Brands.*  
  * *Brands **(1) \--- (N)** Events.*

---

### ***Schema 5: Communications Domain***

***Architectural Philosophy:** This domain is the central nervous system for all real-time interactions on the EventfulHQ platform. It is architected to be **WhatsApp-First**, providing a robust, scalable, and compliant integration with the WhatsApp Business API. It also supports our internal real-time chat, creating a unified communication layer. Every message is treated as an auditable event, and the schema is designed for high-throughput, low-latency delivery, with AI-processing capabilities built into its core to enable intelligent automation and analysis.*

---

#### ***WhatsappConversations***

* ***Core Purpose:** Manages and tracks ongoing WhatsApp Business API conversations, respecting WhatsApp's 24-hour session window.*  
* ***Definition & Business Logic:** This table acts as a container for a thread of messages with a specific user via their WhatsApp number. Its primary business logic is to manage the state of the conversation, particularly the session\_expires\_at timestamp, which is a critical constraint of the WhatsApp Business API. A "session" is initiated when a user messages in, opening a 24-hour window for free-form replies. Business-initiated messages outside this window require pre-approved templates. This table allows our system to track these sessions, route conversations to the correct agents or bots (assigned\_agent\_id), and categorize them for analytics (conversation\_category).*  
* ***Attributes:** | Column Name | Data Type | Constraints & Description | | :--- | :--- | :--- | | **conversation\_id** | UUID | **Primary Key**, DEFAULT gen\_random\_uuid(). Unique ID for the conversation thread. | | **whatsapp\_phone\_number**| VARCHAR(20)| **NOT NULL, \[idx\]**. The end-user's WhatsApp number that is part of the conversation. | | **user\_id** | UUID | **Foreign Key** \-\> users(user\_id), **\[idx\]**. Links the conversation to a registered EventfulHQ user, if one exists for the phone number. | | **status** | ENUM | DEFAULT 'active'. The current state of the conversation ('active', 'ended', 'blocked', 'archived'). | | **session\_expires\_at**| TIMESTAMPTZ| Nullable. The critical timestamp marking the end of WhatsApp's 24-hour session window for free-form replies. | | **last\_message\_at** | TIMESTAMPTZ| Timestamp of the last message received or sent, used for sorting and inactivity checks. | | **assigned\_agent\_id** | UUID | **Foreign Key** \-\> users(user\_id). Nullable. Assigns the conversation to a specific support agent or SuperAdmin. |*  
* ***Key Relationships & Cardinality:***  
  * *WhatsappConversations **(1) \--- (N)** WhatsappMessages*  
  * *Users **(1) \--- (N)** WhatsappConversations*  
* ***Recommended Indexes:** CREATE INDEX idx\_whatsapp\_conversations\_phone ON whatsapp\_conversations(whatsapp\_phone\_number);, CREATE INDEX idx\_whatsapp\_conversations\_status ON whatsapp\_conversations(status);*

---

#### ***WhatsappMessages***

* ***Core Purpose:** Logs every individual inbound and outbound WhatsApp message, providing a complete, auditable history of the conversation.*  
* ***Definition & Business Logic:** This is the immutable log of all WhatsApp communication. The direction field is critical for understanding the flow of conversation. The message\_type enum supports the full richness of the WhatsApp platform, from simple text to interactive buttons and media. The status field ('sent', 'delivered', 'read') provides the delivery receipts essential for reliable communication. Crucially, the AI-processing fields (ai\_intent, ai\_sentiment, etc.) allow our system to analyze incoming messages in real-time to power chatbots, automate responses, or intelligently route conversations.*  
* ***Attributes:** | Column Name | Data Type | Constraints & Description | | :--- | :--- | :--- | | **message\_id** | UUID | **Primary Key**, DEFAULT gen\_random\_uuid(). | | **conversation\_id** | UUID | **Foreign Key** \-\> whatsapp\_conversations(conversation\_id), **NOT NULL**. Links the message to its parent conversation. | | **whatsapp\_message\_id**| VARCHAR(255)| **UNIQUE**. The unique ID provided by the WhatsApp API for each message, used for status tracking and reconciliation. | | **direction** | ENUM | **NOT NULL**. 'inbound' for user-to-platform, 'outbound' for platform-to-user. | | **message\_type** | ENUM | **NOT NULL**. The type of message, e.g., 'text', 'image', 'interactive', 'template'. | | **text\_content** | TEXT | The textual content of the message. | | **media\_url** | VARCHAR(500)| Nullable. A URL to the media file if the message type is image, video, etc. | | **template\_name**| VARCHAR(255)| Nullable. The name of the WhatsappTemplate used for an outbound message. | | **status** | ENUM | DEFAULT 'sent'. The delivery status of the message ('sent', 'delivered', 'read', 'failed'). | | **ai\_intent** | VARCHAR(100)| Nullable. The user's intent as classified by our AI (e.g., 'check\_booking\_status', 'request\_refund'). | | **ai\_sentiment** | ENUM | Nullable. The sentiment of the message ('positive', 'negative', 'neutral') as classified by our AI. | | **created\_at** | TIMESTAMPTZ| DEFAULT NOW(). The timestamp the message was recorded in our system. |*  
* ***Key Relationships & Cardinality:***  
  * *WhatsappMessages **(N) \--- (1)** WhatsappConversations*  
* ***Recommended Indexes:** CREATE INDEX idx\_whatsapp\_messages\_conversation ON whatsapp\_messages(conversation\_id);, CREATE INDEX idx\_whatsapp\_messages\_created\_at ON whatsapp\_messages(created\_at);*

---

#### ***WhatsappTemplates***

* ***Core Purpose:** Manages pre-approved WhatsApp Business message templates required for initiating conversations with users.*  
* ***Definition & Business Logic:** This table is the gateway to proactive, outbound WhatsApp communication. Due to WhatsApp's strict anti-spam policies, any message sent to a user more than 24 hours after their last message must be a pre-approved template. This table stores the content of these templates, their placeholder variables, and their approval whatsapp\_status from Meta. This allows our platform to programmatically send reliable, compliant notifications like ticket confirmations, event reminders, and security alerts.*  
* ***Attributes:** | Column Name | Data Type | Constraints & Description | | :--- | :--- | :--- | | **template\_id** | UUID | **Primary Key**, DEFAULT gen\_random\_uuid(). | | **team\_id** | UUID | **Foreign Key** \-\> teams(team\_id). Nullable. If the template is custom for a specific team. | | **template\_name**| VARCHAR(255)| **NOT NULL**. A unique, human-readable name for the template within our system. | | **category** | ENUM | **NOT NULL**. The WhatsApp-defined category, e.g., 'authentication', 'marketing', 'utility'. | | **language** | VARCHAR(10) | **NOT NULL**, DEFAULT 'en'. The language code of the template. | | **body\_text** | TEXT | **NOT NULL**. The template's body content, with placeholders like {{1}}, {{2}}. | | **whatsapp\_status**| ENUM | DEFAULT 'pending'. The approval status from Meta ('pending', 'approved', 'rejected'). | | **is\_active** | BOOLEAN | DEFAULT TRUE. Allows admins to enable or disable the use of a template. |*  
* ***Key Relationships & Cardinality:***  
  * *Teams **(1) \--- (N)** WhatsappTemplates (Teams can have their own custom templates).*  
  * *WhatsappTemplates **(1) \--- (N)** WhatsappMessages (A template can be used to generate many messages).*

---

#### ***InternalChat: Conversations & Messages***

* ***Core Purpose:** These tables support the on-platform, real-time **Chat** application for communication between registered users and teams.*  
* ***Definition & Business Logic:** This provides a self-contained messaging system for interactions that happen within the EventfulHQ web or mobile app, such as a Team collaborating internally on an Event or an organizer negotiating with a Venue. Unlike WhatsApp, this system does not have session windows or template restrictions. It is designed for persistent, project-based communication. The structure mirrors the WhatsApp tables for consistency.*

##### ***InternalConversations \[BaseEntity\]***

| *Column Name* | *Data Type* | *Constraints & Description* |
| :---- | :---- | :---- |
| ***conversation\_id*** | *UUID* | ***Primary Key**.* |
| ***title*** | *VARCHAR(255)* | *Nullable. An optional title for the conversation, e.g., "Logistics for AI Conf 2025".* |
| ***related\_entity\_id*** | *UUID* | ***P\_FK**. Polymorphic link to the entity this conversation is about (e.g., an event\_edition\_id, booking\_id).* |
| ***related\_entity\_type*** | *VARCHAR(50)* | ***P\_FK**. The type of the related entity.* |
| ***...*** |  | *Includes all \[BaseEntity\] audit and soft-delete fields.* |

##### ***ConversationParticipants (Junction Table)***

| *Column Name* | *Data Type* | *Constraints & Description* |
| :---- | :---- | :---- |
| ***conversation\_id*** | *UUID* | ***PK, Foreign Key** \-\> InternalConversations(conversation\_id).* |
| ***user\_id*** | *UUID* | ***PK, Foreign Key** \-\> users(user\_id).* |
| ***joined\_at*** | *TIMESTAMPTZ* | *DEFAULT NOW().* |

##### ***InternalMessages \[BaseEntity\]***

| *Column Name* | *Data Type* | *Constraints & Description* |
| :---- | :---- | :---- |
| ***message\_id*** | *UUID* | ***Primary Key**.* |
| ***conversation\_id*** | *UUID* | ***Foreign Key** \-\> InternalConversations(conversation\_id). **NOT NULL**.* |
| ***sender\_user\_id*** | *UUID* | ***Foreign Key** \-\> users(user\_id). **NOT NULL**. The user who sent the message.* |
| ***content*** | *TEXT* | *The body of the message.* |
| ***...*** |  | *Includes all \[BaseEntity\] audit and soft-delete fields.* |

* ***Key Relationships & Cardinality:***  
  * *InternalConversations **(N) \--- (M)** Users (via ConversationParticipants).*  
  * *InternalConversations **(1) \--- (N)** InternalMessages.*  
  * *Events, Bookings, etc. **(1) \--- (N)** InternalConversations (via polymorphic link).*

---

### **Schema 6: Content & Community Domain**

**Architectural Philosophy:** This domain is the vibrant heart of the EventfulHQ ecosystem, designed to capture, organize, and amplify all forms of content. It manages the full spectrum of community interaction, from ephemeral social Posts to structured educational Courses and high-quality Videos. The architecture supports a powerful flywheel: user engagement in the **Connect** app generates data and social proof, which in turn fuels the content and value of the **Academy** and **Stream** platforms, creating a deeply interconnected and self-enriching community experience.

---

#### **Posts \[BaseEntity\]**

* **Core Purpose:** This entity represents a single piece of user-generated content (e.g., a review, a photo, a comment) published to the "Social Wall" of another entity on the platform.  
* **Definition & Business Logic:** This is a versatile and central table for all social activity within the **Connect** app. The polymorphic relationship (related\_entity\_id, related\_entity\_type) allows a Post to be attached to an Event, Venue, Artist, or any other major entity, creating a dynamic, context-specific feed. The content\_type enum supports rich media, including polls and event shares. Moderation fields (is\_flagged, moderation\_status) are crucial for maintaining community health. Every post created is a signal that feeds into the user's FanProfile contribution score.  
* **Attributes:** | Column Name | Data Type | Constraints & Description | | :--- | :--- | :--- | | **post\_id** | UUID | **Primary Key**, DEFAULT gen\_random\_uuid(). | | **author\_user\_id** | UUID | **Foreign Key** \-\> users(user\_id). **NOT NULL**. The user who created the post. | | **content** | TEXT | **NOT NULL**. The main body of the post. | | **content\_type** | ENUM | DEFAULT 'text'. The format of the post, e.g., 'text', 'image', 'video', 'poll', 'review'. | | **media\_urls** | TEXT\[\] | An array of URLs pointing to associated media files stored in the Media & Content Domain. | | **related\_entity\_id**| UUID | **P\_FK**. Polymorphic link to the entity this post is related to (e.g., the Event being reviewed). | | **related\_entity\_type**| VARCHAR(50)| **P\_FK**. The type of the related entity. | | **like\_count** | INTEGER | DEFAULT 0\. A denormalized count for fast retrieval, updated by triggers. | | **comment\_count** | INTEGER | DEFAULT 0\. A denormalized count for fast retrieval. | | **...** | | *Includes all \[BaseEntity\] audit and soft-delete fields.* |  
* **Key Relationships & Cardinality:**  
  * Users **(1) \--- (N)** Posts  
  * Events, Venues, etc. **(1) \--- (N)** Posts (via polymorphic "wall" relationship)  
  * Posts **(1) \--- (N)** PostLikes  
  * Posts **(1) \--- (N)** PostComments  
* **Recommended Indexes:** CREATE INDEX idx\_posts\_author\_user ON posts(author\_user\_id) WHERE NOT is\_deleted;, CREATE INDEX idx\_posts\_published\_at ON posts(published\_at) WHERE NOT is\_deleted AND is\_published;  
  ---

  #### **PostLikes**

* **Core Purpose:** Tracks user likes and reactions on a Post, serving as a primary engagement metric.  
* **Definition & Business Logic:** A high-volume junction table that creates a record for each user reaction to a post. The UNIQUE constraint on (post\_id, user\_id) ensures a user can only react once per post. The reaction\_type allows for richer emotional feedback, providing deeper sentiment data for our **Analytics** engine. An application-level trigger on this table is responsible for incrementing the like\_count on the parent Posts table.  
* **Attributes:** | Column Name | Data Type | Constraints & Description | | :--- | :--- | :--- | | **like\_id** | UUID | **Primary Key**, DEFAULT gen\_random\_uuid(). | | **post\_id** | UUID | **Foreign Key** \-\> posts(post\_id). **NOT NULL**. Part of a UNIQUE composite key. | | **user\_id** | UUID | **Foreign Key** \-\> users(user\_id). **NOT NULL**. Part of a UNIQUE composite key. | | **reaction\_type**| ENUM | DEFAULT 'like'. The type of reaction, e.g., 'like', 'love', 'laugh', 'wow'. | | **created\_at** | TIMESTAMPTZ| DEFAULT NOW(). |  
* **Key Relationships & Cardinality:**  
  * Posts **(1) \--- (N)** PostLikes  
  * Users **(1) \--- (N)** PostLikes

  ---

  #### **PostComments \[BaseEntity\]**

* **Core Purpose:** Enables threaded conversations on Posts, fostering community dialogue.  
* **Definition & Business Logic:** This table stores comments linked to a parent Post. The key architectural feature is the self-referencing parent\_comment\_id, which allows for nested or threaded replies, creating an organized conversation flow. Like Posts, comments have moderation fields to ensure community safety. Triggers on this table update the comment\_count on the parent Posts record.  
* **Attributes:** | Column Name | Data Type | Constraints & Description | | :--- | :--- | :--- | | **comment\_id** | UUID | **Primary Key**, DEFAULT gen\_random\_uuid(). | | **post\_id** | UUID | **Foreign Key** \-\> posts(post\_id). **NOT NULL**. The post this comment belongs to. | | **author\_user\_id** | UUID | **Foreign Key** \-\> users(user\_id). **NOT NULL**. The user who wrote the comment. | | **content** | TEXT | **NOT NULL**. The body of the comment. | | **parent\_comment\_id**| UUID | **Foreign Key** \-\> post\_comments(comment\_id). Nullable. If this is a reply, it links to the parent comment, enabling threading. | | **...** | | *Includes all \[BaseEntity\] audit and soft-delete fields.* |  
* **Key Relationships & Cardinality:**  
  * Posts **(1) \--- (N)** PostComments  
  * Users **(1) \--- (N)** PostComments  
  * PostComments **(1) \--- (N)** PostComments (Self-referencing for replies).

  ---

  #### **Follows**

* **Core Purpose:** Powers the social graph of the platform, tracking the relationships between users.  
* **Definition & Business Logic:** This dedicated junction table is optimized for the high-volume read/write operations of a social network. It records a directional link from a follower\_user\_id to a following\_user\_id. The status field is crucial for implementing "follow requests" for private profiles, which require approval from the following\_user\_id.  
* **Attributes:** | Column Name | Data Type | Constraints & Description | | :--- | :--- | :--- | | **follow\_id** | UUID | **Primary Key**, DEFAULT gen\_random\_uuid(). | | **follower\_user\_id** | UUID | **Foreign Key** \-\> users(user\_id). **NOT NULL**. The user initiating the follow. Part of a UNIQUE composite key. | | **following\_user\_id**| UUID | **Foreign Key** \-\> users(user\_id). **NOT NULL**. The user being followed. Part of a UNIQUE composite key. | | **status** | ENUM | DEFAULT 'active'. The state of the follow relationship, e.g., 'active', 'pending', 'blocked'. | | **created\_at** | TIMESTAMPTZ| DEFAULT NOW(). |  
* **Key Relationships & Cardinality:**  
  * Users **(1) \--- (N)** Follows (as follower).  
  * Users **(1) \--- (N)** Follows (as following).

  ---

  #### **Courses \[BaseEntity\]**

* **Core Purpose:** Represents a single educational course or masterclass within the **Academy** application.  
* **Definition & Business Logic:** This entity defines a structured learning experience. Each Course is created by an instructor\_user\_id (who must be a verified Expert or Artist). The platform's business logic will enforce that only users with ProMax subscriptions can become instructors. The price\_credits attribute allows instructors to monetize their expertise, with the platform taking a commission on each sale processed through the **Wallet**.  
* **Attributes:** | Column Name | Data Type | Constraints & Description | | :--- | :--- | :--- | | **course\_id** | UUID | **Primary Key**, DEFAULT gen\_random\_uuid(). | | **title** | VARCHAR(255) | **NOT NULL**. The public title of the course. | | **description** | TEXT | A detailed description of the course content, target audience, and learning outcomes. | | **instructor\_user\_id**| UUID | **Foreign Key** \-\> users(user\_id). **NOT NULL**. The verified Expert or Artist teaching the course. | | **price\_credits**| DECIMAL(10,2)| DEFAULT 0\. The cost to enroll in the course. 0 indicates a free course. | | **difficulty\_level**| ENUM | DEFAULT 'beginner'. The level of the course, e.g., 'beginner', 'intermediate', 'advanced'. | | **...** | | *Includes all \[BaseEntity\] audit and soft-delete fields.* |  
* **Key Relationships & Cardinality:**  
  * Users (as Instructor) **(1) \--- (N)** Courses.  
  * Courses **(N) \--- (M)** Users (as Student) (via Enrollments).  
  * Courses **(1) \--- (N)** Certifications.

  ---

  #### **Enrollments**

* **Core Purpose:** A junction table that tracks a User's progress in a Course they have enrolled in.  
* **Definition & Business Logic:** This table is created when a User enrolls in a Course. It acts as the student's personal record, tracking their progress\_percentage. The application logic will update this percentage as the user completes modules and quizzes. The completed\_at timestamp is the trigger for issuing a Certification.  
* **Attributes:** | Column Name | Data Type | Constraints & Description | | :--- | :--- | :--- | | **enrollment\_id** | UUID | **Primary Key**, DEFAULT gen\_random\_uuid(). | | **user\_id** | UUID | **Foreign Key** \-\> users(user\_id). The student. Part of a UNIQUE composite key. | | **course\_id** | UUID | **Foreign Key** \-\> courses(course\_id). The course. Part of a UNIQUE composite key. | | **enrolled\_at** | TIMESTAMPTZ| DEFAULT NOW(). The timestamp of enrollment. | | **progress\_percentage**| INTEGER | DEFAULT 0\. The student's completion progress, from 0 to 100\. | | **completed\_at** | TIMESTAMPTZ| Nullable. Timestamp when progress reaches 100%. |  
* **Key Relationships & Cardinality:**  
  * Users **(1) \--- (N)** Enrollments.  
  * Courses **(1) \--- (N)** Enrollments.

  ---

  #### **Videos**

* **Core Purpose:** Represents a single video asset within the **Stream** application.  
* **Definition & Business Logic:** This table acts as the central registry for all video content. The polymorphic source link is critical, as it allows a Video to be associated with its origin—whether it's a recording from an Event, a promotional video for a Tour, or an educational module for a Course. This context is used by the AI to power recommendations and discovery.  
* **Attributes:** | Column Name | Data Type | Constraints & Description | | :--- | :--- | :--- | | **video\_id** | UUID | **Primary Key**, DEFAULT gen\_random\_uuid(). | | **title** | VARCHAR(255) | **NOT NULL**. The public title of the video. | | **description** | TEXT | A description of the video's content. | | **cdn\_url** | VARCHAR(500)| **NOT NULL**. The URL to the master video file on the Content Delivery Network. | | **duration\_seconds** | INTEGER | The length of the video in seconds. | | **uploader\_user\_id**| UUID | **Foreign Key** \-\> users(user\_id). **NOT NULL**. The user who uploaded the video. | | **source\_entity\_id**| UUID | **P\_FK**. The ID of the entity this video is from (e.g., an event\_edition\_id). | | **source\_entity\_type**| VARCHAR(50)| **P\_FK**. The type of the source entity. |  
* **Key Relationships & Cardinality:**  
  * Users **(1) \--- (N)** Videos.  
  * Events, Courses, etc. **(1) \--- (N)** Videos (via polymorphic link).  
  * Videos **(N) \--- (M)** Playlists (via a junction table).

  ---

  #### **AiEnhancedContent**

* **Core Purpose:** Tracks AI-generated or \-enhanced content across the platform, providing an audit trail and a feedback loop for model performance.  
* **Definition & Business Logic:** This table is the output log for many **Genesis** tools. When a user enhances an Event description with AI, a record is created here storing the original\_content and the ai\_enhanced\_content. This serves multiple purposes: it allows users to revert to the original, provides an audit trail (model\_id), and creates a valuable dataset for fine-tuning our AI models based on which enhancements users approve and keep.  
* **Attributes:** | Column Name | Data Type | Constraints & Description | | :--- | :--- | :--- | | **enhancement\_id** | UUID | **Primary Key**, DEFAULT gen\_random\_uuid(). | | **entity\_id** | UUID | **P\_FK, NOT NULL**. The ID of the entity whose content was enhanced. | | **entity\_type**| VARCHAR(50)| **P\_FK, NOT NULL**. The type of the entity. | | **field\_name** | VARCHAR(100)| **NOT NULL**. The specific field that was enhanced (e.g., 'title', 'description'). | | **original\_content**| TEXT | **NOT NULL**. The original user-provided content. | | **enhanced\_content**| TEXT | **NOT NULL**. The new content generated by the AI. | | **model\_id** | UUID | **Foreign Key** \-\> ai\_models(model\_id). **NOT NULL**. The AI model that performed the enhancement. | | **is\_active** | BOOLEAN | DEFAULT FALSE. A flag indicating if the enhanced version is currently live on the entity's profile. |  
* **Key Relationships & Cardinality:**  
  * Any content-heavy entity (Events, Posts, etc.) **(1) \--- (N)** AiEnhancedContent.  
  * AiModels **(1) \--- (N)** AiEnhancedContent.

---

*Of course. Here is the highly detailed specification for **Schema 7: Operations & Analytics Domain**, following the exact same comprehensive format.*

---

### ***Schema 7: Operations & Analytics Domain***

***Architectural Philosophy:** This domain is the engine room of the platform, providing the essential infrastructure for modern DevOps, enterprise integrations, and high-performance analytics. The entities here are not typically interacted with directly by end-users but are critical for ensuring platform stability, manageability, and speed. The aggregation tables (UserStatistics, EventStatistics) are a direct implementation of your performance optimization recommendations, designed to provide lightning-fast dashboard loads for our **Analytics** application.*

---

#### ***WebhookSubscriptions***

* ***Core Purpose:** This entity powers our integration capabilities, allowing Teams to subscribe to real-time events on the platform and receive notifications at a specified URL.*  
* ***Definition & Business Logic:** This is a crucial feature for our enterprise clients, enabling them to build a connected software ecosystem. A Team can configure a webhook to be notified instantly when a specific action occurs (e.g., booking.created, ticket.sold). This allows them to sync EventfulHQ data with their own internal systems like CRMs, accounting software, or marketing automation platforms in real-time. The secret\_key is used for signature verification, ensuring that the receiving server can confirm the webhook payload genuinely originated from EventfulHQ.*  
* ***Attributes:** | Column Name | Data Type | Constraints & Description | | :--- | :--- | :--- | | **webhook\_id** | UUID | **Primary Key**, DEFAULT gen\_random\_uuid(). | | **team\_id** | UUID | **Foreign Key** \-\> teams(team\_id). **NOT NULL**. Webhooks are a team-level feature. | | **endpoint\_url** | VARCHAR(500)| **NOT NULL**. The URL to which the platform will send the POST request payload. | | **event\_types** | TEXT\[\] | **NOT NULL**. An array of specific events the team is subscribing to (e.g., \['booking.created', 'event.published'\]). | | **secret\_key** | VARCHAR(255)| **NOT NULL**. A secret key used to generate a signature for verifying the webhook's authenticity. | | **is\_active** | BOOLEAN | DEFAULT TRUE. A toggle to easily enable or disable the webhook without deleting it. | | **retry\_count** | INTEGER | DEFAULT 0. Tracks the number of failed delivery attempts for the last payload. | | **last\_delivery\_attempt**| TIMESTAMPTZ| The timestamp of the last attempted delivery, successful or not. |*  
* ***Key Relationships & Cardinality:***  
  * *Teams **(1) \--- (N)** WebhookSubscriptions*

---

#### ***SystemHealthChecks***

* ***Core Purpose:** This entity provides the data for our internal monitoring and status dashboards, ensuring platform stability.*  
* ***Definition & Business Logic:** This table is the pulse of the platform, accessible only to SuperAdmins in the 'Developer' or 'Platform Governor' tiers. Automated monitoring agents will populate this table every minute for every microservice. This allows our SRE (Site Reliability Engineering) team to create real-time dashboards, set up alerts for degraded performance (response\_time\_ms spikes) or downtime (status \= 'DOWN'), and ensure we meet our Service Level Agreements (SLAs) for enterprise customers.*  
* ***Attributes:** | Column Name | Data Type | Constraints & Description | | :--- | :--- | :--- | | **check\_id** | UUID | **Primary Key**, DEFAULT gen\_random\_uuid(). | | **service\_name**| VARCHAR(100)| **NOT NULL**. The name of the microservice being checked (e.g., 'wallet-service', 'auth-service'). | | **status** | ENUM | **NOT NULL**. The operational status ('HEALTHY', 'DEGRADED', 'DOWN'). | | **response\_time\_ms**| INTEGER | The latency of the service response in milliseconds. | | **error\_message**| TEXT | Nullable. Any error message returned by the service if the check failed. | | **checked\_at** | TIMESTAMPTZ| DEFAULT NOW(). The timestamp when the health check was performed. |*  
* ***Key Relationships & Cardinality:***  
  * *This is a standalone logging and monitoring table with no direct foreign key relationships to business entities.*

---

#### ***FeatureFlags***

* ***Core Purpose:** This entity is a cornerstone of our agile development strategy, allowing for gradual and targeted feature rollouts.*  
* ***Definition & Business Logic:** This table provides our product and engineering teams with fine-grained control over feature visibility without requiring new code deployments. A new feature can be created with is\_enabled: true but a rollout\_percentage: 1, allowing internal testing. It can then be rolled out to target\_user\_segments like 'ProMax' users first to gather feedback, before being gradually rolled out to 100% of users. This de-risks new releases and enables data-driven product development.*  
* ***Attributes:** | Column Name | Data Type | Constraints & Description | | :--- | :--- | :--- | | **flag\_id** | UUID | **Primary Key**, DEFAULT gen\_random\_uuid(). | | **flag\_name** | VARCHAR(100)| **UNIQUE, NOT NULL**. A human-readable, unique name for the feature (e.g., 'new-dashboard-ui'). | | **is\_enabled** | BOOLEAN | DEFAULT FALSE. The master switch for the feature. | | **rollout\_percentage**| INTEGER | DEFAULT 0. The percentage of users (0-100) for whom the feature is enabled. | | **target\_user\_segments**| TEXT\[\] | Nullable. An array of user segments to target (e.g., \['ProMax', 'Artists'\]). |*  
* ***Key Relationships & Cardinality:***  
  * *This is a standalone configuration table.*

---

#### ***MicroserviceCommunications***

* ***Core Purpose:** Logs all internal API calls between our microservices for diagnostic and performance monitoring purposes.*  
* ***Definition & Business Logic:** In a distributed microservices architecture, tracing a single user request across multiple services can be complex. This table acts as a distributed trace log. Every internal service-to-service call is logged with a shared request\_id. This allows developers to reconstruct the entire lifecycle of an operation, identify which service is causing a bottleneck (response\_time\_ms), and debug complex failures.*  
* ***Attributes:** | Column Name | Data Type | Constraints & Description | | :--- | :--- | :--- | | **communication\_id**| UUID | **Primary Key**, DEFAULT gen\_random\_uuid(). | | **source\_service**| VARCHAR(100)| **NOT NULL**. The name of the service initiating the call. | | **target\_service**| VARCHAR(100)| **NOT NULL**. The name of the service receiving the call. | | **request\_id** | UUID | **NOT NULL, \[idx\]**. A shared ID that traces a single operation across all services. | | **response\_time\_ms**| INTEGER | The time taken for the target service to respond. | | **status\_code** | INTEGER | The HTTP status code of the response (e.g., 200, 404, 500). | | **created\_at** | TIMESTAMPTZ| DEFAULT NOW(). |*  
* ***Key Relationships & Cardinality:***  
  * *This is a standalone logging table.*

---

#### ***UserStatistics (Pre-computed Aggregation Table)***

* ***Core Purpose:** A performance optimization layer for the **Analytics** dashboard, providing pre-calculated key metrics for each user.*  
* ***Definition & Business Logic:** Instead of running expensive COUNT and SUM queries across many tables every time a user profile or dashboard is loaded, this table stores the results. A background job (e.g., a nightly cron job) runs to update these metrics. This ensures that user-facing analytics are fast and responsive, a critical component of a good user experience, as you recommended.*  
* ***Attributes:** | Column Name | Data Type | Constraints & Description | | :--- | :--- | :--- | | **user\_id** | UUID | **Primary Key**, **Foreign Key** \-\> users(user\_id). A 1:1 relationship with the Users table. | | **events\_created**| INTEGER | DEFAULT 0. A denormalized count of events owned by the user. | | **events\_attended**| INTEGER | DEFAULT 0. A denormalized count of events the user has attended. | | **total\_earnings\_credits**| DECIMAL(15,2)| DEFAULT 0. For Experts and Artists, the total credits earned through bookings. | | **followers\_count**| INTEGER | DEFAULT 0. The total number of followers from the Follows table. | | **last\_updated** | TIMESTAMPTZ| The timestamp of the last background job update. |*  
* ***Key Relationships & Cardinality:***  
  * *Users **(1) \--- (1)** UserStatistics*

---

#### ***EventStatistics (Pre-computed Aggregation Table)***

* ***Core Purpose:** A performance optimization layer for event-specific dashboards in the **Analytics** app.*  
* ***Definition & Business Logic:** Similar to UserStatistics, this table pre-aggregates the most important performance indicators for an Event. This allows an event organizer to see their total revenue, attendee count, and social engagement at a glance without waiting for slow database queries. The last\_updated field allows the frontend to show if the data is fresh or currently being recalculated.*  
* ***Attributes:** | Column Name | Data Type | Constraints & Description | | :--- | :--- | :--- | | **event\_id** | UUID | **Primary Key**, **Foreign Key** \-\> events(event\_edition\_id). A 1:1 relationship with the Events table. | | **total\_revenue\_credits**| DECIMAL(15,2)| DEFAULT 0. The sum of all revenue from tickets, sponsorships, etc., from the Transactions table. | | **attendee\_count**| INTEGER | DEFAULT 0. The current number of registered attendees. | | **social\_engagement\_score**| INTEGER | DEFAULT 0. An aggregated score based on posts, comments, and likes on the event's social wall. | | **last\_updated** | TIMESTAMPTZ| The timestamp of the last background job update. |*  
* ***Key Relationships & Cardinality:***  
  * *Events **(1) \--- (1)** EventStatistics*

---

### ***Schema 8: Marketplace & Commerce Domain***

***Architectural Philosophy:** This domain is the transactional heart of the EventfulHQ ecosystem. It models the core business logic for all value exchanges, from hiring an Artist to securing a multi-million credit Sponsorship. Every entity in this domain is designed to be auditable, stateful, and deeply integrated with the **Finance Domain** to ensure transactional integrity. This schema powers the primary functionality of the **Spotlight**, **Craftwork**, **Alliance**, **Showcase**, and **Covenant** applications.*

---

#### ***Bookings \[BaseEntity\]***

* ***Core Purpose:** This is the central transaction table for the **Spotlight** and **Craftwork** marketplaces. It represents the formal hiring of a User (an Artist or Expert) for a specific Event.*  
* ***Definition & Business Logic:** This entity captures the entire lifecycle of a talent engagement. It begins as an 'inquiry' and moves through a state machine ('pending', 'confirmed', 'completed'). The table links the three primary actors: the Event, the talent\_user\_id being hired, and the booked\_by party (a User or Team). The financial terms, including all allowances, are explicitly defined, and the total\_amount\_credits is a generated column to ensure data consistency. Post-event, this table also stores the two-way ratings, providing crucial data for the marketplace's reputation system.*  
* ***Attributes:** | Column Name | Data Type | Constraints & Description | | :--- | :--- | :--- | | **booking\_id** | UUID | **Primary Key**, DEFAULT gen\_random\_uuid(). | | **booking\_reference**| VARCHAR(100)| **UNIQUE, NOT NULL**. A human-readable reference code for the booking. | | **event\_id** | UUID | **Foreign Key** \-\> events(event\_id), **NOT NULL, \[idx\]**. The event for which the talent is being booked. | | **talent\_user\_id** | UUID | **Foreign Key** \-\> users(user\_id), **NOT NULL, \[idx\]**. The Artist or Expert being booked. | | **booked\_by\_user\_id**| UUID | **Foreign Key** \-\> users(user\_id). Nullable. The individual user making the booking. | | **booked\_by\_team\_id** | UUID | **Foreign Key** \-\> teams(team\_id). Nullable. The team making the booking. | | **fee\_credits** | DECIMAL(15,2)| **NOT NULL**. The primary performance or service fee in platform credits. | | **total\_amount\_credits**| DECIMAL(15,2)| **Generated Column**. The sum of fee\_credits and all allowances, ensuring the total is always accurate. | | **status** | ENUM | **NOT NULL**, DEFAULT 'inquiry'. The lifecycle state of the booking, e.g., 'inquiry', 'confirmed', 'completed', 'cancelled'. | | **confirmed\_at** | TIMESTAMPTZ| Nullable. Timestamp when the booking status becomes 'confirmed'. | | **organizer\_rating** | INTEGER | CHECK (organizer\_rating BETWEEN 1 AND 5\). The organizer's rating of the talent, post-event. | | **artist\_rating** | INTEGER | CHECK (artist\_rating BETWEEN 1 AND 5\). The talent's rating of the organizer, post-event. | | **...** | | Includes all \[BaseEntity\] audit and soft-delete fields. | | **CONSTRAINT**| check\_booking\_party| CHECK ((booked\_by\_user\_id IS NOT NULL AND booked\_by\_team\_id IS NULL) OR (booked\_by\_user\_id IS NULL AND booked\_by\_team\_id IS NOT NULL)). |*  
* ***Key Relationships & Cardinality:***  
  * *Events **(1) \--- (N)** Bookings*  
  * *Users (as Talent) **(1) \--- (N)** Bookings*  
  * *Bookings **(1) \--- (N)** BookingMessages*  
  * *Bookings **(1) \--- (1)** Contracts*  
* ***Recommended Indexes:** CREATE INDEX idx\_bookings\_status ON bookings(status) WHERE NOT is\_deleted;*

---

#### ***BookingMessages***

* ***Core Purpose:** Manages the communication thread specifically for a booking negotiation, creating an auditable trail of all offers and agreements.*  
* ***Definition & Business Logic:** This table isolates the negotiation for a specific Booking from general chat. This is critical for legal and dispute resolution purposes. The message\_type enum distinguishes between a simple 'text' message and a binding 'offer' or 'acceptance'. The offer\_data JSONB can store the structured terms of an offer, providing a clear history of how the final fee\_credits in the parent Bookings table was reached.*  
* ***Attributes:** | Column Name | Data Type | Constraints & Description | | :--- | :--- | :--- | | **message\_id** | UUID | **Primary Key**, DEFAULT gen\_random\_uuid(). | | **booking\_id** | UUID | **Foreign Key** \-\> bookings(booking\_id), **NOT NULL, \[idx\]**. The booking this message belongs to. | | **sender\_user\_id** | UUID | **Foreign Key** \-\> users(user\_id), **NOT NULL**. The user who sent the message. | | **message\_type** | ENUM | DEFAULT 'text'. The type of message, e.g., 'text', 'offer', 'counter\_offer', 'acceptance'. | | **message\_text** | TEXT | **NOT NULL**. The content of the message. | | **offer\_data** | JSONB | Nullable. A structured JSON object containing the terms of an offer, e.g., {"fee": 5000, "expenses\_covered": true}. | | **is\_read** | BOOLEAN | DEFAULT FALSE. Tracks if the recipient has read the message. | | **created\_at** | TIMESTAMPTZ| DEFAULT NOW(). |*  
* ***Key Relationships & Cardinality:***  
  * *Bookings **(1) \--- (N)** BookingMessages*  
  * *Users **(1) \--- (N)** BookingMessages*

---

#### ***Contracts \[BaseEntity\]***

* ***Core Purpose:** Represents the legally binding agreement that formalizes a Booking, powering the **Covenant** app.*  
* ***Definition & Business Logic:** This entity serves as the single source of truth for a legal agreement. It is linked 1:1 with a Booking and is generated once the Booking status is 'confirmed'. The document\_url points to a securely stored, non-modifiable PDF. The status tracks the signature lifecycle, which can be updated via integrations with e-signature platforms. All versions and changes are tracked via the inherited \[BaseEntity\] fields.*  
* ***Attributes:** | Column Name | Data Type | Constraints & Description | | :--- | :--- | :--- | | **contract\_id** | UUID | **Primary Key**, DEFAULT gen\_random\_uuid(). | | **booking\_id** | UUID | **Foreign Key** \-\> bookings(booking\_id), **UNIQUE, NOT NULL**. Enforces a strict 1:1 relationship with a booking. | | **contract\_number**| VARCHAR(100)| **UNIQUE, NOT NULL**. A human-readable, unique reference for the contract. | | **status** | ENUM | DEFAULT 'draft'. The lifecycle status, e.g., 'draft', 'sent', 'signed', 'fulfilled'. | | **document\_url** | VARCHAR(500)| A secure URL pointing to the generated PDF contract. | | **template\_id**| UUID | **Foreign Key** \-\> contract\_templates(template\_id). The template used to generate this contract. | | **client\_signed\_at**| TIMESTAMPTZ| Timestamp of the client's signature. | | **artist\_signed\_at**| TIMESTAMPTZ| Timestamp of the artist's/expert's signature. | | **...** | | Includes all \[BaseEntity\] audit and soft-delete fields. |*  
* ***Key Relationships & Cardinality:***  
  * *Bookings **(1) \--- (1)** Contracts*  
  * *ContractTemplates **(1) \--- (N)** Contracts*

---

#### ***ContractTemplates***

* ***Core Purpose:** Stores reusable contract templates, enabling the "low-click" and automated contract generation features of the **Covenant** app.*  
* ***Definition & Business Logic:** This table acts as a library of pre-approved legal documents. Platform-provided templates (is\_public: true) are available to all users. Enterprise Teams can upload and manage their own proprietary templates (created\_by\_team\_id) for their specific needs. The placeholders JSONB field defines the variables (e.g., {{{artist\_name}}}) that the system will automatically populate from the Bookings and Users tables when generating a new contract.*  
* ***Attributes:** | Column Name | Data Type | Constraints & Description | | :--- | :--- | :--- | | **template\_id** | UUID | **Primary Key**, DEFAULT gen\_random\_uuid(). | | **template\_name**| VARCHAR(255)| **NOT NULL**. The name of the template. | | **template\_category**| ENUM | **NOT NULL**. e.g., 'artist\_performance', 'venue\_rental', 'sponsorship'. | | **template\_content**| TEXT | **NOT NULL**. The full legal text of the template with placeholders. | | **placeholders** | JSONB | **NOT NULL**. A JSON object defining the variables to be replaced, e.g., {"artist\_name": "string", "performance\_date": "date"}. | | **is\_public** | BOOLEAN | DEFAULT FALSE. true for standard templates provided by EventfulHQ. | | **created\_by\_team\_id**| UUID | **Foreign Key** \-\> teams(team\_id). Nullable. The team that owns this custom template. |*  
* ***Key Relationships & Cardinality:***  
  * *ContractTemplates **(1) \--- (N)** Contracts*  
  * *Teams **(1) \--- (N)** ContractTemplates*

---

#### ***Sponsorships \[BaseEntity\]***

* ***Core Purpose:** Powers the **Alliance** marketplace, representing a financial or in-kind sponsorship of an Event by a Brand.*  
* ***Definition & Business Logic:** This table formalizes the relationship between a sponsor and an event. It links an Event to a Brand and captures the key commercial terms, including the sponsorship\_tier and the amount\_credits. The status field allows the event organizer to manage the sponsorship sales pipeline from 'proposed' to 'confirmed'.*  
* ***Attributes:** | Column Name | Data Type | Constraints & Description | | :--- | :--- | :--- | | **sponsorship\_id** | UUID | **Primary Key**, DEFAULT gen\_random\_uuid(). | | **event\_edition\_id** | UUID | **Foreign Key** \-\> events(event\_edition\_id), **NOT NULL, \[idx\]**. | | **brand\_id** | UUID | **Foreign Key** \-\> brands(brand\_id), **NOT NULL, \[idx\]**. | | **sponsorship\_tier** | VARCHAR(100)| The level of sponsorship (e.g., 'Gold', 'Silver', 'Presenting Partner'). | | **amount\_credits**| DECIMAL(15,2)| **NOT NULL**. The value of the sponsorship in platform credits. | | **status** | ENUM | DEFAULT 'proposed'. The lifecycle state, e.g., 'proposed', 'confirmed', 'active'. | | **...** | | Includes all \[BaseEntity\] audit and soft-delete fields. |*  
* ***Key Relationships & Cardinality:***  
  * *Events **(1) \--- (N)** Sponsorships*  
  * *Brands **(1) \--- (N)** Sponsorships*

---

#### ***Exhibitors \[BaseEntity\]***

* ***Core Purpose:** Powers the **Showcase** marketplace by representing a Team officially registered to exhibit at a trade show Event.*  
* ***Definition & Business Logic:** This table manages the core data for an exhibition. It links the exhibiting Team to the specific Event, and records crucial operational details like the assigned booth\_number and booth\_size. The registration\_fee\_credits is captured to track revenue for the event organizer in their **Ledger**.*  
* ***Attributes:** | Column Name | Data Type | Constraints & Description | | :--- | :--- | :--- | | **exhibitor\_id** | UUID | **Primary Key**, DEFAULT gen\_random\_uuid(). | | **event\_edition\_id** | UUID | **Foreign Key** \-\> events(event\_edition\_id), **NOT NULL, \[idx\]**. | | **exhibiting\_team\_id**| UUID | **Foreign Key** \-\> teams(team\_id), **NOT NULL, \[idx\]**. The Team that is exhibiting. | | **booth\_number** | VARCHAR(50) | The assigned booth number on the event floor plan. | | **booth\_size** | VARCHAR(50) | The dimensions or type of the booth (e.g., '10x10', 'Island'). | | **registration\_fee\_credits**| DECIMAL(10,2)| The fee paid by the exhibitor to secure the booth. | | **...** | | Includes all \[BaseEntity\] audit and soft-delete fields. |*  
* ***Key Relationships & Cardinality:***  
  * *Events **(1) \--- (N)** Exhibitors*  
  * *Teams **(1) \--- (N)** Exhibitors*

---

### ***Schema 9: Event Operations Domain***

***Architectural Philosophy:** This domain operationalizes the event plan. It provides the essential entities for managing event execution, ticketing, access control, and staffing. The tables in this schema are designed for high transactional volume and real-time updates, forming the backbone of our **Monetization Suite**, the on-site **Conductor** app, and the **Brigade** staffing marketplace. Every entity includes full \[BaseEntity\] support for a complete audit trail of all operational changes.*

---

#### ***TicketTypes \[BaseEntity\]***

* ***Core Purpose:** Defines the different categories, pricing tiers, and rules for tickets available for a specific Event.*  
* ***Definition & Business Logic:** This table acts as a template for creating individual Tickets. It allows an event organizer to set up a sophisticated sales strategy with multiple tiers like 'Early Bird', 'VIP', and 'General Admission', each with its own price, quantity, and sales window. The quantity\_available is a generated column that automatically calculates remaining inventory (total\_quantity \- quantity\_sold), ensuring data consistency and preventing overselling. This entity is the core configuration for the event's ticketing page on its **Portal**.*  
* ***Attributes:** | Column Name | Data Type | Constraints & Description | | :--- | :--- | :--- | | **ticket\_type\_id** | UUID | **Primary Key**, DEFAULT gen\_random\_uuid(). | | **event\_id** | UUID | **Foreign Key** \-\> events(event\_id), **NOT NULL, \[idx\]**. The event this ticket type belongs to. | | **name** | VARCHAR(255) | **NOT NULL**. The public name of the ticket tier (e.g., "VIP Gold Pass"). | | **price\_credits** | DECIMAL(15,2)| **NOT NULL**. The cost of the ticket in platform credits. A value of 0 indicates a free ticket. | | **total\_quantity** | INTEGER | **NOT NULL**. The total number of tickets available for this tier. | | **quantity\_sold** | INTEGER | DEFAULT 0. A denormalized count of tickets sold, updated by triggers for performance. | | **quantity\_available**| INTEGER | **Generated Column** (total\_quantity \- quantity\_sold). Ensures real-time accuracy. | | **sale\_start\_datetime**| TIMESTAMPTZ| **NOT NULL**. The exact time when this ticket tier becomes available for purchase. | | **sale\_end\_datetime** | TIMESTAMPTZ| **NOT NULL**. The exact time when sales for this tier close. | | **included\_perks** | TEXT\[\] | An array of strings describing the benefits of this tier (e.g., 'Meet & Greet', 'Free Drink'). | | **...** | | Includes all \[BaseEntity\] audit and soft-delete fields. |*  
* ***Key Relationships & Cardinality:***  
  * *Events **(1) \--- (N)** TicketTypes*  
  * *TicketTypes **(1) \--- (N)** Tickets*  
* ***Recommended Indexes:** CREATE INDEX idx\_ticket\_types\_event ON ticket\_types(event\_id) WHERE NOT is\_deleted;*

---

#### ***Tickets \[BaseEntity\]***

* ***Core Purpose:** Represents a single, unique ticket instance purchased by a User for an Event.*  
* ***Definition & Business Logic:** This is the primary entity for event access control. Each record is a unique, ownable asset. The qr\_code and access\_token are unique identifiers used by the **Conductor** on-site app for scanning and check-in. The table maintains a clear chain of ownership from the purchaser\_user\_id to the current\_owner\_user\_id, which is updated via the TicketTransfers table. This creates an auditable history, which is crucial for managing entry and resolving disputes.*  
* ***Attributes:** | Column Name | Data Type | Constraints & Description | | :--- | :--- | :--- | | **ticket\_id** | UUID | **Primary Key**, DEFAULT gen\_random\_uuid(). | | **ticket\_number**| VARCHAR(100)| **UNIQUE, NOT NULL**. A human-readable unique identifier for the ticket. | | **event\_id** | UUID | **Foreign Key** \-\> events(event\_id), **NOT NULL, \[idx\]**. | | **ticket\_type\_id**| UUID | **Foreign Key** \-\> ticket\_types(ticket\_type\_id), **NOT NULL**. The category this ticket belongs to. | | **purchaser\_user\_id**| UUID | **Foreign Key** \-\> users(user\_id), **NOT NULL**. The user who originally bought the ticket. | | **current\_owner\_user\_id**| UUID | **Foreign Key** \-\> users(user\_id), **NOT NULL, \[idx\]**. The current holder of the ticket. | | **purchase\_price\_credits**| DECIMAL(15,2)| **NOT NULL**. The actual price paid for the ticket. | | **qr\_code** | VARCHAR(255)| **UNIQUE, NOT NULL**. The unique data for the QR code used for scanning at the gate. | | **check\_in\_status**| ENUM | DEFAULT 'not\_checked\_in'. The current status ('not\_checked\_in', 'checked\_in', 'no\_show'). | | **checked\_in\_at**| TIMESTAMPTZ| Nullable. The exact timestamp of the check-in scan. | | **status** | ENUM | DEFAULT 'active'. The lifecycle status of the ticket ('active', 'transferred', 'refunded'). | | **...** | | Includes all \[BaseEntity\] audit and soft-delete fields. |*  
* ***Key Relationships & Cardinality:***  
  * *TicketTypes **(1) \--- (N)** Tickets*  
  * *Users (as owner) **(1) \--- (N)** Tickets*  
  * *Tickets **(1) \--- (N)** TicketTransfers*  
* ***Recommended Indexes:** CREATE INDEX idx\_tickets\_qr\_code ON tickets(qr\_code) WHERE NOT is\_deleted;*

---

#### ***TicketTransfers \[BaseEntity\]***

* ***Core Purpose:** Creates a secure and auditable log of every ticket ownership transfer between users.*  
* ***Definition & Business Logic:** This table is essential for enabling a secure secondary market or simple gifting of tickets. When a user initiates a transfer, a record is created here with a 'pending' status and a unique transfer\_token. The recipient uses this token to claim the ticket, at which point the status becomes 'completed', and the current\_owner\_user\_id on the parent Tickets table is updated. This prevents fraud and provides a clear, traceable history of ownership for every ticket.*  
* ***Attributes:** | Column Name | Data Type | Constraints & Description | | :--- | :--- | :--- | | **transfer\_id** | UUID | **Primary Key**, DEFAULT gen\_random\_uuid(). | | **ticket\_id** | UUID | **Foreign Key** \-\> tickets(ticket\_id), **NOT NULL**. The ticket being transferred. | | **from\_user\_id**| UUID | **Foreign Key** \-\> users(user\_id), **NOT NULL**. The user initiating the transfer. | | **to\_user\_id** | UUID | **Foreign Key** \-\> users(user\_id). Nullable until the transfer is claimed. | | **to\_email** | VARCHAR(255)| Used to invite a non-registered user to accept the ticket. | | **status** | ENUM | DEFAULT 'pending'. The state of the transfer ('pending', 'completed', 'cancelled', 'expired'). | | **transfer\_token**| VARCHAR(255)| **UNIQUE, NOT NULL**. A secure, single-use token for claiming the transfer. | | **expires\_at** | TIMESTAMPTZ| **NOT NULL**. A deadline by which the recipient must accept the transfer. | | **...** | | Includes all \[BaseEntity\] audit and soft-delete fields. |*  
* ***Key Relationships & Cardinality:***  
  * *Tickets **(1) \--- (N)** TicketTransfers*  
  * *Users **(1) \--- (N)** TicketTransfers (as both sender and receiver).*

---

#### ***EventStaff \[BaseEntity\]***

* ***Core Purpose:** Manages staff assignments, roles, and schedules for a specific Event.*  
* ***Definition & Business Logic:** This table powers the **Brigade** marketplace and on-site operations. It connects a User to an Event with a specific role\_title and detailed responsibilities. This is more than a simple role; it's a mini-contract that includes shift times and payment details (hourly\_rate\_credits). The table is used by the **Conductor** app to display schedules and by the **Ledger** app to calculate payroll. The post-event performance\_rating feeds back into the user's Reliability Score in the **Brigade** app.*  
* ***Attributes:** | Column Name | Data Type | Constraints & Description | | :--- | :--- | :--- | | **staff\_assignment\_id**| UUID | **Primary Key**, DEFAULT gen\_random\_uuid(). | | **event\_id** | UUID | **Foreign Key** \-\> events(event\_id), **NOT NULL**. | | **user\_id** | UUID | **Foreign Key** \-\> users(user\_id), **NOT NULL**. | | **role\_title** | VARCHAR(255)| **NOT NULL**. The specific job title, e.g., "Stage Manager", "Security Guard". | | **role\_category**| ENUM | **NOT NULL**. High-level category, e.g., 'management', 'production', 'volunteer'. | | **assignment\_status**| ENUM | DEFAULT 'invited'. The status of the staff member's assignment. | | **shift\_start\_datetime**| TIMESTAMPTZ| **NOT NULL**. The start time of their assigned shift. | | **shift\_end\_datetime** | TIMESTAMPTZ| **NOT NULL**. The end time of their assigned shift. | | **performance\_rating**| DECIMAL(3,2)| Nullable. A post-event rating of the staff member's performance. | | **...** | | Includes all \[BaseEntity\] audit and soft-delete fields. |*  
* ***Key Relationships & Cardinality:***  
  * *Events **(1) \--- (N)** EventStaff*  
  * *Users **(1) \--- (N)** EventStaff*

---

#### ***EventRequirements \[BaseEntity\]***

* ***Core Purpose:** Tracks all technical and logistical requirements for an event, acting as a master supply chain checklist.*  
* ***Definition & Business Logic:** This table is a project management tool for event producers. It allows them to log every single item or service needed for an event, from 'technical' (e.g., sound system) to 'permits'. The status field tracks the procurement lifecycle from 'needed' to 'completed'. Crucially, the depends\_on\_requirement\_ids array allows for creating dependencies, which is essential for the AI in our **Timeline** app to perform critical path analysis.*  
* ***Attributes:** | Column Name | Data Type | Constraints & Description | | :--- | :--- | :--- | | **requirement\_id**| UUID | **Primary Key**, DEFAULT gen\_random\_uuid(). | | **event\_id** | UUID | **Foreign Key** \-\> events(event\_id), **NOT NULL**. | | **requirement\_type**| ENUM | **NOT NULL**. The category of the need, e.g., 'technical', 'catering', 'security'. | | **title** | VARCHAR(255)| **NOT NULL**. A concise name for the requirement (e.g., "Main Stage PA System"). | | **status** | ENUM | DEFAULT 'needed'. The procurement status, e.g., 'needed', 'quoted', 'booked'. | | **priority** | ENUM | DEFAULT 'medium'. The priority of the requirement ('low', 'medium', 'high', 'critical'). | | **assigned\_to\_user\_id**| UUID | **Foreign Key** \-\> users(user\_id). The team member responsible for sourcing this item. | | **estimated\_cost\_credits**| DECIMAL(15,2)| The budgeted cost, which links to a **LineItem** in the **Ledger** app. | | **depends\_on\_requirement\_ids**| UUID\[\] | An array of other requirement\_ids that must be completed first. | | **...** | | Includes all \[BaseEntity\] audit and soft-delete fields. |*  
* ***Key Relationships & Cardinality:***  
  * *Events **(1) \--- (N)** EventRequirements*  
  * *Users **(1) \--- (N)** EventRequirements (as assignee).*

---

#### ***EventChecklist \[BaseEntity\]***

* ***Core Purpose:** Provides a granular task management and checklist system for event planning, powering the **Timeline** app.*  
* ***Definition & Business Logic:** While EventRequirements is for procuring things, EventChecklist is for doing things. This table breaks down the entire event plan into actionable tasks. The due\_date and status fields are standard project management attributes. The dependency mapping (depends\_on\_item\_ids) is the most advanced feature, enabling a Gantt chart view and allowing the AI to flag delays in critical path items that could jeopardize the entire event timeline.*  
* ***Attributes:** | Column Name | Data Type | Constraints & Description | | :--- | :--- | :--- | | **checklist\_item\_id**| UUID | **Primary Key**, DEFAULT gen\_random\_uuid(). | | **event\_id** | UUID | **Foreign Key** \-\> events(event\_id), **NOT NULL**. | | **title** | VARCHAR(255)| **NOT NULL**. The name of the task (e.g., "Confirm Keynote Speaker Travel"). | | **category** | ENUM | **NOT NULL**. The phase of the event this task belongs to, e.g., 'planning', 'marketing', 'post\_event'. | | **assigned\_to\_user\_id**| UUID | **Foreign Key** \-\> users(user\_id). The team member responsible for this task. | | **status** | ENUM | DEFAULT 'not\_started'. The current status of the task ('not\_started', 'in\_progress', 'completed'). | | **due\_date** | TIMESTAMPTZ| Nullable. The deadline for completing the task. | | **depends\_on\_item\_ids**| UUID\[\] | An array of other checklist\_item\_ids that are prerequisites. | | **...** | | Includes all \[BaseEntity\] audit and soft-delete fields. |*  
* ***Key Relationships & Cardinality:***  
  * *Events **(1) \--- (N)** EventChecklist*  
  * *Users **(1) \--- (N)** EventChecklist (as assignee).*

---

### ***Schema 10: Subscription & Billing Domain***

***Architectural Philosophy:** This domain is the automated revenue engine of the EventfulHQ platform. It is designed to be highly flexible, allowing the business to easily define and modify subscription plans without code changes. The schema provides a robust, auditable system for managing the entire subscription lifecycle—from trial and activation to billing, usage tracking, and dunning. It is built to integrate seamlessly with the **Finance Domain** to ensure every charge is tracked and every invoice is accurate.*

---

#### ***SubscriptionPlans***

* ***Core Purpose:** Defines the available SaaS subscription plans, their pricing, and the features and limits associated with each tier.*  
* ***Definition & Business Logic:** This table is the "menu" of our SaaS offerings, directly modeling our business logic. It stores the details for our **Base (Free)**, **Pro ($9.99 / 999 Credits)**, and **ProMax ($99.99 / 9999 Credits)** tiers. The monthly\_price\_credits and annual\_price\_credits columns allow us to manage our pricing and 15% annual discount centrally. The features and limits (max\_events\_per\_month, ai\_credits\_included) are stored in a flexible JSONB format, allowing the product team to easily adjust plan offerings. This table is the single source of truth for what a customer receives at each subscription level.*  
* ***Attributes:** | Column Name | Data Type | Constraints & Description | | :--- | :--- | :--- | | **plan\_id** | UUID | **Primary Key**, DEFAULT gen\_random\_uuid(). | | **plan\_name** | VARCHAR(100) | **UNIQUE, NOT NULL**. The public name of the plan (e.g., 'Base', 'Pro', 'ProMax'). | | **plan\_category** | ENUM | **NOT NULL**. The target for the plan, e.g., 'individual', 'team', 'enterprise'. | | **monthly\_price\_credits**| DECIMAL(10,2)| The cost of the plan per month in platform credits (e.g., 999.00 for Pro). | | **annual\_price\_credits** | DECIMAL(10,2)| The discounted cost for an annual subscription (e.g., (999 \* 12 \* 0.85)). | | **features** | JSONB | **NOT NULL**. A JSON object detailing the features included (e.g., {"ai\_enhanced\_content": true, "custom\_branding": false}). | | **max\_events\_per\_month**| INTEGER | Nullable. The quota for events a user/team on this plan can create per month. NULL means unlimited. | | **ai\_credits\_included** | INTEGER | The number of AI credits included per month before overage charges apply. | | **is\_active** | BOOLEAN | DEFAULT TRUE. Allows admins to gracefully retire old plans without deleting them. |*  
* ***Key Relationships & Cardinality:***  
  * *SubscriptionPlans **(1) \--- (N)** Subscriptions*

---

#### ***Subscriptions \[BaseEntity\]***

* ***Core Purpose:** Represents an active subscription linking a User or Team to a specific SubscriptionPlan.*  
* ***Definition & Business Logic:** This is the central table for managing a customer's relationship with EventfulHQ. A record is created here when a User or Team signs up for a paid plan. The status field drives the entire lifecycle, managed by automated billing jobs: a trialing subscription automatically transitions to active, an active one becomes past\_due after a failed payment, and eventually unpaid or cancelled. The current\_period\_start and current\_period\_end fields define the billing cycle, and auto\_renew controls the renewal logic.*  
* ***Attributes:** | Column Name | Data Type | Constraints & Description | | :--- | :--- | :--- | | **subscription\_id** | UUID | **Primary Key**, DEFAULT gen\_random\_uuid(). | | **team\_id** | UUID | **Foreign Key** \-\> teams(team\_id). Nullable. The Team that is subscribed. | | **user\_id** | UUID | **Foreign Key** \-\> users(user\_id). Nullable. The User who is subscribed. | | **plan\_id** | UUID | **Foreign Key** \-\> subscription\_plans(plan\_id), **NOT NULL**. The plan they are subscribed to. | | **status** | ENUM | **NOT NULL**, DEFAULT 'active'. The current state of the subscription (e.g., 'active', 'trialing', 'cancelled'). | | **current\_period\_start**| DATE | The start date of the current billing cycle. | | **current\_period\_end** | DATE | The end date of the current billing cycle. | | **auto\_renew** | BOOLEAN | DEFAULT TRUE. Determines if the subscription should automatically renew at the end of the period. | | **external\_subscription\_id**| VARCHAR(255)| The ID from the payment gateway (e.g., Stripe) for this subscription, for reconciliation. | | **...** | | Includes all \[BaseEntity\] audit and soft-delete fields. | | **CONSTRAINT**| check\_subscriber| CHECK ((team\_id IS NOT NULL AND user\_id IS NULL) OR (team\_id IS NULL AND user\_id IS NOT NULL)). Ensures each subscription has exactly one owner. |*  
* ***Key Relationships & Cardinality:***  
  * *SubscriptionPlans **(1) \--- (N)** Subscriptions*  
  * *Users **(1) \--- (N)** Subscriptions*  
  * *Teams **(1) \--- (N)** Subscriptions*  
  * *Subscriptions **(1) \--- (N)** Invoices*  
  * *Subscriptions **(1) \--- (N)** SubscriptionUsage*

---

#### ***Invoices \[BaseEntity\]***

* ***Core Purpose:** Represents a specific, itemized bill issued for a subscription period or a one-time purchase.*  
* ***Definition & Business Logic:** This table provides the formal billing document, which is critical for our B2B clients' accounting and record-keeping. An Invoice is generated automatically at the start of a new billing period for a Subscription. Its status ('draft', 'sent', 'paid', 'overdue') drives the dunning process (automated payment reminders) and financial reporting. The pdf\_url provides a link to a non-modifiable, customer-facing document for their records.*  
* ***Attributes:** | Column Name | Data Type | Constraints & Description | | :--- | :--- | :--- | | **invoice\_id** | UUID | **Primary Key**, DEFAULT gen\_random\_uuid(). | | **subscription\_id** | UUID | **Foreign Key** \-\> subscriptions(subscription\_id). Links the invoice to the subscription it's for. | | **amount\_credits** | DECIMAL(10,2)| **NOT NULL**. The total amount due in platform credits. | | **status** | ENUM | **NOT NULL**, DEFAULT 'draft'. The current status of the invoice lifecycle. | | **due\_date** | DATE | **NOT NULL**. The date by which payment is due. | | **paid\_date** | DATE | Nullable. The date the invoice was successfully paid. | | **invoice\_number**| VARCHAR(100)| **UNIQUE, NOT NULL**. A human-readable, sequential invoice number. | | **pdf\_url** | VARCHAR(500)| A secure URL to the generated PDF version of the invoice. | | **...** | | Includes all \[BaseEntity\] audit and soft-delete fields. |*  
* ***Key Relationships & Cardinality:***  
  * *Subscriptions **(1) \--- (N)** Invoices*

---

#### ***SubscriptionUsage***

* ***Core Purpose:** Tracks the consumption of metered resources (like AI credits or events created) against a subscription's limits.*  
* ***Definition & Business Logic:** This table is essential for enforcing the limits defined in SubscriptionPlans. For every metered action (e.g., a user on a 'Pro' plan creates a new Event), a record is created or updated here. A background job checks this table against the plan limits. If a limit is exceeded (limit\_exceeded: true), the system can trigger an automated workflow, such as blocking the creation of new events or initiating an overage charge, and notifying the user via the **Notify** engine.*  
* ***Attributes:** | Column Name | Data Type | Constraints & Description | | :--- | :--- | :--- | | **usage\_id** | UUID | **Primary Key**, DEFAULT gen\_random\_uuid(). | | **subscription\_id** | UUID | **Foreign Key** \-\> subscriptions(subscription\_id), **NOT NULL, \[idx\]**. | | **usage\_type** | ENUM | **NOT NULL**. The metered resource being tracked, e.g., 'events\_created', 'ai\_credits\_used'. | | **quantity\_used** | INTEGER | **NOT NULL**. The amount of the resource consumed in this specific instance. | | **cumulative\_usage**| INTEGER | **NOT NULL**. The running total of this usage\_type for the current billing period. | | **limit\_exceeded** | BOOLEAN | DEFAULT FALSE. A flag that is set to true when the cumulative usage surpasses the plan's limit. | | **related\_entity\_id**| UUID | Nullable. The ID of the entity that triggered the usage record (e.g., the event\_edition\_id that was created). | | **created\_at** | TIMESTAMPTZ| DEFAULT NOW(). |*  
* ***Key Relationships & Cardinality:***  
  * *Subscriptions **(1) \--- (N)** SubscriptionUsage*  
* ***Recommended Indexes:** CREATE INDEX idx\_subscription\_usage\_subscription\_type ON subscription\_usage(subscription\_id, usage\_type);*

---

### ***Schema 11: Media & Content Management Domain***

***Architectural Philosophy:** This domain is the centralized repository for all digital assets on the EventfulHQ platform. It is architected as a sophisticated Digital Asset Management (DAM) system, moving beyond simple file hosting. The schema separates the physical file (MediaFiles) from its contextual use (EntityMedia), which is a critical design pattern for efficiency and scalability. It's built for performance with CDN integration and intelligent processing, and serves as a foundational layer for our AI to analyze and tag visual content, powering smarter search and discovery across all applications.*

---

#### ***MediaFiles \[BaseEntity\]***

* ***Core Purpose:** Acts as the single source of truth for every unique media file (image, video, document) uploaded to the platform.*  
* ***Definition & Business Logic:** This table represents the physical asset itself. To ensure efficiency and prevent data duplication, a file\_hash (SHA-256) is generated upon upload. If a file with the same hash already exists, the system reuses the existing MediaFiles record instead of storing a new copy. The table stores critical metadata, including file\_size\_bytes, mime\_type, and dimensions. The processing\_status tracks the progress of background jobs that generate thumbnails and optimized variants for different devices, which is essential for frontend performance. The AI-powered fields (ai\_tags, ai\_description) are populated by our vision models to make all media assets searchable by their content.*  
* ***Attributes:** | Column Name | Data Type | Constraints & Description | | :--- | :--- | :--- | | **file\_id** | UUID | **Primary Key**, DEFAULT gen\_random\_uuid(). Unique ID for the media file record. | | **uploader\_user\_id**| UUID | **Foreign Key** \-\> users(user\_id), **NOT NULL**. The user who uploaded the file. | | **file\_hash** | VARCHAR(64) | **UNIQUE, NOT NULL**. The SHA-256 hash of the file content, used for deduplication. | | **original\_filename**| VARCHAR(255)| **NOT NULL**. The original name of the file as uploaded by the user. | | **file\_size\_bytes**| BIGINT | **NOT NULL**. The size of the file in bytes. | | **mime\_type** | VARCHAR(100)| **NOT NULL**. The MIME type of the file (e.g., 'image/jpeg', 'application/pdf'). | | **storage\_path** | VARCHAR(500)| **NOT NULL**. The path to the file in our cloud storage bucket (e.g., AWS S3). | | **cdn\_url** | VARCHAR(500)| The public URL for the file served via a Content Delivery Network for fast access. | | **media\_type** | ENUM | **NOT NULL**. The high-level category of the media, e.g., 'image', 'video', 'document'. | | **image\_width** | INTEGER | Nullable. The width of the image in pixels, populated after processing. | | **image\_height**| INTEGER | Nullable. The height of the image in pixels. | | **video\_duration\_seconds**| INTEGER | Nullable. The duration of the video in seconds. | | **processing\_status**| ENUM | DEFAULT 'uploaded'. The status of background processing ('uploaded', 'processing', 'completed', 'failed'). | | **variants** | JSONB | DEFAULT '{}'. A JSON object storing URLs for different optimized versions (e.g., {"thumbnail": "url", "medium": "url"}). | | **ai\_tags** | VARCHAR(100)\[\]| An array of tags automatically generated by an AI vision model (e.g., 'crowd', 'stage', 'conference'). | | **...** | | Includes all \[BaseEntity\] audit and soft-delete fields. |*  
* ***Key Relationships & Cardinality:***  
  * *Users **(1) \--- (N)** MediaFiles (as uploader).*  
  * *MediaFiles **(1) \--- (N)** EntityMedia (A single file can be used in many places).*  
* ***Recommended Indexes:** CREATE INDEX idx\_media\_files\_uploader ON media\_files(uploader\_user\_id) WHERE NOT is\_deleted;, CREATE INDEX idx\_media\_files\_hash ON media\_files(file\_hash) WHERE NOT is\_deleted;*

---

#### ***EntityMedia (Junction Table)***

* ***Core Purpose:** This polymorphic junction table associates a specific MediaFile with any other entity on the platform, defining its contextual usage.*  
* ***Definition & Business Logic:** This is the key to the DAM's flexibility. It allows a single uploaded file to be used in multiple contexts without duplication. For example, a sponsor's logo (MediaFiles record) can be linked to an Event with media\_usage: 'banner', to a Team with media\_usage: 'avatar', and to a Sponsorship record with media\_usage: 'attachment'. The display\_order is crucial for creating ordered photo galleries for Events or Venues. The is\_primary flag allows us to designate a main profile picture or banner for any entity.*  
* ***Attributes:** | Column Name | Data Type | Constraints & Description | | :--- | :--- | :--- | | **entity\_media\_id**| UUID | **Primary Key**, DEFAULT gen\_random\_uuid(). | | **entity\_id** | UUID | **P\_FK, NOT NULL, \[idx\]**. The ID of the entity this media is attached to (e.g., an event\_edition\_id). | | **entity\_type** | VARCHAR(50)| **P\_FK, NOT NULL, \[idx\]**. The type of the entity (e.g., 'Event', 'Venue', 'User'). | | **file\_id** | UUID | **Foreign Key** \-\> media\_files(file\_id), **NOT NULL**. The specific file being associated. | | **media\_usage** | ENUM | **NOT NULL**. The context of how the media is used, e.g., 'avatar', 'banner', 'gallery', 'document'. | | **display\_order**| INTEGER | DEFAULT 0. The sort order for media items within a gallery context. | | **alt\_text** | VARCHAR(255)| Text for accessibility (screen readers) and SEO. | | **is\_primary** | BOOLEAN | DEFAULT FALSE. Indicates if this is the primary image for a given media\_usage (e.g., the main profile picture). |*  
* ***Key Relationships & Cardinality:***  
  * *MediaFiles **(1) \--- (N)** EntityMedia*  
  * *Events, Venues, Users, etc. **(1) \--- (N)** EntityMedia (via polymorphic link).*  
* ***Recommended Indexes:** CREATE INDEX idx\_entity\_media\_entity ON entity\_media(entity\_id, entity\_type);, UNIQUE(entity\_id, entity\_type, media\_usage) WHERE is\_primary \= TRUE (ensures only one primary image per context).*

---

### ***Schema 12: Notification System Domain***

***Architectural Philosophy:** This domain is the centralized communication engine for the entire EventfulHQ platform, powering the **Notify** core service. It is designed around a scalable, template-driven architecture that separates content from delivery logic. This allows for consistent messaging, easy updates by non-technical staff, and a complete, auditable log of every notification sent to our users. The schema is built to support intelligent delivery, personalization, and user preference management, preventing notification fatigue and ensuring critical alerts are always received.*

---

#### ***NotificationTemplates***

* ***Core Purpose:** Manages a library of reusable, multi-channel message templates for all automated platform communications.*  
* ***Definition & Business Logic:** This table is a cornerstone of our operational efficiency and consistent branding. It allows SuperAdmins to create and manage the content of transactional notifications (e.g., "Welcome to EventfulHQ," "Booking Confirmed," "Password Reset") in one place. Each template can have variants for different channels (Email, WhatsApp, Push, SMS). The body\_template uses placeholders (e.g., {{{user\_name}}}, {{{event\_title}}}), which are dynamically populated with data when a notification is generated. This "write once, send anywhere" approach ensures that non-developers can manage communications without needing to change application code.*  
* ***Attributes:** | Column Name | Data Type | Constraints & Description | | :--- | :--- | :--- | | **template\_id** | UUID | **Primary Key**, DEFAULT gen\_random\_uuid(). | | **template\_name**| VARCHAR(100)| **UNIQUE, NOT NULL**. A unique internal name for the template (e.g., booking\_confirmation\_v2). | | **template\_category**| ENUM | **NOT NULL**. The business purpose of the template, e.g., 'transactional', 'marketing', 'alert', 'reminder'. | | **channels** | ENUM\[\] | **NOT NULL**. An array of supported channels, e.g., {'email', 'whatsapp', 'push'}. | | **subject\_template**| TEXT | Nullable. The subject line template for email notifications, with placeholders. | | **body\_template** | TEXT | **NOT NULL**. The main content template with placeholders for personalization. | | **whatsapp\_template\_name**| VARCHAR(100)| Nullable. The corresponding pre-approved template name from the WhatsappTemplates table, if applicable. | | **available\_variables**| JSONB | A JSON object that documents the placeholders available for this template, for developer reference (e.g., {"user\_name": "string", "event\_title": "string"}). | | **is\_active** | BOOLEAN | DEFAULT TRUE. Allows admins to enable or disable the use of a template. |*  
* ***Key Relationships & Cardinality:***  
  * *NotificationTemplates **(1) \--- (N)** Notifications*

---

#### ***Notifications***

* ***Core Purpose:** An immutable log of every individual notification sent from the platform to a user, providing a complete delivery and engagement audit trail.*  
* ***Definition & Business Logic:** This table is the definitive record of all communications. When an action on the platform triggers a notification (e.g., a Booking is confirmed), a record is created here. It links the recipient\_user\_id to the template\_id used, and stores the final, personalized message content. The status field is critical for customer support and system monitoring; it tracks the entire delivery lifecycle from 'queued' to 'sent', 'delivered', and finally to engagement states like 'opened' or 'clicked'. This provides a clear audit trail to answer questions like, "Did this user receive the payment reminder?"*  
* ***Attributes:** | Column Name | Data Type | Constraints & Description | | :--- | :--- | :--- | | **notification\_id**| UUID | **Primary Key**, DEFAULT gen\_random\_uuid(). | | **recipient\_user\_id**| UUID | **Foreign Key** \-\> users(user\_id), **NOT NULL, \[idx\]**. The user who received the notification. | | **template\_id** | UUID | **Foreign Key** \-\> notification\_templates(template\_id). Nullable, for non-templated system alerts. | | **title** | VARCHAR(255)| The final, personalized title or subject of the notification. | | **message** | TEXT | **NOT NULL**. The final, personalized body content of the notification sent to the user. | | **channels\_sent**| ENUM\[\] | An array of channels this notification was sent through (e.g., {'email', 'push'}). | | **status** | ENUM | DEFAULT 'queued'. The delivery status of the notification, e.g., 'queued', 'sent', 'delivered', 'failed', 'opened'. | | **read\_at** | TIMESTAMPTZ| Nullable. Timestamp when the notification was marked as read by the user. | | **provider\_message\_id**| VARCHAR(255)| The unique message ID from the delivery provider (e.g., SendGrid, Twilio) for reconciliation. | | **related\_entity\_id**| UUID | **P\_FK**. The ID of the entity that triggered this notification (e.g., a booking\_id). | | **related\_entity\_type**| VARCHAR(50)| **P\_FK**. The type of the related entity. | | **created\_at** | TIMESTAMPTZ| DEFAULT NOW(). |*  
* ***Key Relationships & Cardinality:***  
  * *Users **(1) \--- (N)** Notifications*  
  * *NotificationTemplates **(1) \--- (N)** Notifications*  
  * *Events, Bookings, etc. **(1) \--- (N)** Notifications (via polymorphic link).*  
* ***Recommended Indexes:** CREATE INDEX idx\_notifications\_recipient ON notifications(recipient\_user\_id);, CREATE INDEX idx\_notifications\_status ON notifications(status);*

---

### ***Schema 13: Integration Management Domain***

***Architectural Philosophy:** This domain provides the framework for EventfulHQ to connect seamlessly with the third-party services that our clients already use, transforming our platform from a standalone application into a central, integrated hub. It is designed to be secure, with encrypted credential storage, and auditable, with detailed logs for every synchronization activity. This schema is the foundation for our **App Marketplace** and enables powerful, automated workflows between EventfulHQ and the broader SaaS ecosystem (CRMs, accounting software, marketing automation, etc.).*

---

#### ***ThirdPartyIntegrations \[BaseEntity\]***

* ***Core Purpose:** Manages the configuration, credentials, and status of connections between a Team and external third-party services.*  
* ***Definition & Business Logic:** This table acts as the master record for every integration a Team enables. For example, when a Team connects their Zoom account, a record is created here. Security is paramount; the credentials (like API keys or OAuth tokens) are always stored in an encrypted format. The status field ('active', 'error', 'revoked') is critical for the system to know whether it can reliably sync data. The table also stores synchronization settings, like auto\_sync\_enabled and sync\_frequency\_minutes, allowing teams to control how and when data is exchanged.*  
* ***Attributes:** | Column Name | Data Type | Constraints & Description | | :--- | :--- | :--- | | **integration\_id** | UUID | **Primary Key**, DEFAULT gen\_random\_uuid(). | | **team\_id** | UUID | **Foreign Key** \-\> teams(team\_id), **NOT NULL**. The Team that owns this integration. | | **service\_name** | VARCHAR(100)| **NOT NULL**. A unique identifier for the external service (e.g., 'zoom', 'stripe', 'calendly'). | | **service\_category** | ENUM | **NOT NULL**. The category of the service, e.g., 'video\_conferencing', 'payment', 'crm'. | | **credentials** | JSONB | **NOT NULL**. An encrypted JSON object containing the necessary API keys, tokens, or other secrets. **Never stored in plain text.** | | **config** | JSONB | A JSON object for user-configurable settings related to the integration (e.g., {"default\_zoom\_host": "user\_id"}). | | **status** | ENUM | **NOT NULL**, DEFAULT 'active'. The current health and status of the integration ('active', 'inactive', 'error', 'revoked'). | | **last\_sync\_at** | TIMESTAMPTZ| Nullable. The timestamp of the last successful data synchronization. | | **...** | | Includes all \[BaseEntity\] audit and soft-delete fields. | | **CONSTRAINT**| UNIQUE | UNIQUE(team\_id, service\_name). Ensures a team can only have one active integration per service. |*  
* ***Key Relationships & Cardinality:***  
  * *Teams **(1) \--- (N)** ThirdPartyIntegrations*  
  * *ThirdPartyIntegrations **(1) \--- (N)** SyncLogs*

---

#### ***SyncLogs***

* ***Core Purpose:** Provides a detailed, immutable audit trail of all data synchronization activities between EventfulHQ and third-party services.*  
* ***Definition & Business Logic:** This table is essential for debugging and monitoring the health of our integrations. Every time a sync job runs for a ThirdPartyIntegrations record, a new SyncLogs record is created. It captures the operation type (e.g., importing data from a CRM, exporting attendees to a marketing platform), the status of the job, and detailed metrics on the number of records processed, created, or failed. For developers and support staff, this log is the first place to look to diagnose any issues with a customer's integration.*  
* ***Attributes:** | Column Name | Data Type | Constraints & Description | | :--- | :--- | :--- | | **sync\_id** | UUID | **Primary Key**, DEFAULT gen\_random\_uuid(). | | **integration\_id** | UUID | **Foreign Key** \-\> third\_party\_integrations(integration\_id), **NOT NULL, \[idx\]**. Links the log to its parent integration. | | **operation** | VARCHAR(100)| The specific operation that was performed (e.g., 'import\_contacts', 'export\_attendees'). | | **status** | ENUM | **NOT NULL**. The final status of the sync job ('started', 'completed', 'failed', 'cancelled'). | | **started\_at** | TIMESTAMPTZ| DEFAULT NOW(). The timestamp when the sync job was initiated. | | **completed\_at**| TIMESTAMPTZ| Nullable. The timestamp when the sync job finished. | | **records\_processed**| INTEGER | DEFAULT 0. The total number of records the job attempted to process. | | **success\_count** | INTEGER | DEFAULT 0. The number of records that were successfully synchronized. | | **error\_count** | INTEGER | DEFAULT 0. The number of records that failed to sync. | | **error\_details**| JSONB | Nullable. A JSON object or array containing details about any errors that occurred during the sync. |*  
* ***Key Relationships & Cardinality:***  
  * *ThirdPartyIntegrations **(1) \--- (N)** SyncLogs*  
* ***Recommended Indexes:** CREATE INDEX idx\_sync\_logs\_integration ON sync\_logs(integration\_id);, CREATE INDEX idx\_sync\_logs\_status ON sync\_logs(status);*

---

### ***Schema 14: API & Integration Management Domain***

***Architectural Philosophy:** This domain is the gateway for external systems to interact with EventfulHQ data. It is designed with a **"Zero Trust"** security model, where every application and every request must be authenticated and authorized. The architecture supports multiple integration patterns, from simple no-code HTML embeds for our C2C users to sophisticated REST/GraphQL APIs for our B2B enterprise clients. Access to this domain's capabilities is a core feature of the **ProMax Tier**, providing a powerful incentive for our top-tier customers.*

---

#### ***DeveloperApplications***

* ***Core Purpose:** Manages the registration and configuration of third-party applications that intend to use the EventfulHQ API.*  
* ***Definition & Business Logic:** This is the starting point for any developer integration. Before a Team can get API keys, they must first register a DeveloperApplication. This is a standard industry practice (e.g., creating an app on the X/Twitter Developer Portal). This table acts as a master record for an integration, defining its name, scope, and the Team that owns it. This ensures that every API key is tied to a specific, known application, providing a clear audit trail and preventing anonymous API usage.*  
* ***Attributes:** | Column Name | Data Type | Constraints & Description | | :--- | :--- | :--- | | **app\_id** | UUID | **Primary Key**, DEFAULT gen\_random\_uuid(). Unique identifier for the developer application. | | **app\_name** | VARCHAR(255) | **NOT NULL**. The human-readable name of the application (e.g., "Acme Corp's CRM Connector"). | | **app\_description** | TEXT | A description of what the application does and how it uses EventfulHQ data. | | **owning\_team\_id**| UUID | **Foreign Key** \-\> teams(team\_id). **NOT NULL**. The ProMax Team that owns this application. | | **status** | ENUM | DEFAULT 'active'. The status of the application, e.g., 'active', 'suspended', 'under\_review'. | | **redirect\_uris**| TEXT\[\] | An array of allowed callback URLs for OAuth 2.0 flows, preventing phishing attacks. |*  
* ***Key Relationships & Cardinality:***  
  * *Teams **(1) \--- (N)** DeveloperApplications*  
  * *DeveloperApplications **(1) \--- (1)** ApiCredentials*

---

#### ***ApiCredentials***

* ***Core Purpose:** Securely stores and manages the authentication credentials (Client ID, Client Secret) associated with a DeveloperApplication.*  
* ***Definition & Business Logic:** This table is the vault for our API secrets. When a DeveloperApplication is created, a corresponding record is generated here. The client\_id is public, but the client\_secret is shown only once and its hash is stored, never the raw value. These credentials are used in a standard OAuth 2.0 flow to obtain short-lived access\_tokens. This is far more secure than using a single, long-lived static API key for all requests.*  
* ***Attributes:** | Column Name | Data Type | Constraints & Description | | :--- | :--- | :--- | | **credential\_id** | UUID | **Primary Key**, DEFAULT gen\_random\_uuid(). | | **app\_id** | UUID | **Foreign Key** \-\> developer\_applications(app\_id). **UNIQUE, NOT NULL**. A 1:1 link to the parent application. | | **client\_id** | VARCHAR(100)| **UNIQUE, NOT NULL**. The public identifier for the application. | | **client\_secret\_hash**| VARCHAR(255)| **NOT NULL**. A secure bcrypt or Argon2 hash of the client secret. | | **permissions\_scope**| TEXT\[\] | **NOT NULL**. An array of permission scopes this application is allowed to request (e.g., \['events:read', 'bookings:write'\]). |*  
* ***Key Relationships & Cardinality:***  
  * *DeveloperApplications **(1) \--- (1)** ApiCredentials*

---

#### ***EmbeddableWidgets***

* ***Core Purpose:** Manages the configuration of no-code, embeddable HTML snippets for users.*  
* ***Definition & Business Logic:** This table directly addresses the "embed as HTML" use case. A ProMax user can generate a widget for one of their entities (e.g., a ticket purchase box for an Event). This table stores the configuration for that specific widget instance, such as custom colors or layout choices. The widget\_token is a secure, public token used in the HTML snippet to fetch the correct data without exposing private API keys on a public website.*  
* ***Attributes:** | Column Name | Data Type | Constraints & Description | | :--- | :--- | :--- | | **widget\_id** | UUID | **Primary Key**, DEFAULT gen\_random\_uuid(). | | **owning\_team\_id**| UUID | **Foreign Key** \-\> teams(team\_id). **NOT NULL**. | | **widget\_type** | ENUM | **NOT NULL**. The type of widget, e.g., 'event\_tickets', 'artist\_schedule', 'venue\_calendar'. | | **target\_entity\_id**| UUID | **P\_FK, NOT NULL**. The ID of the Event, Artist, etc., this widget displays data for. | | **target\_entity\_type**| VARCHAR(50)| **P\_FK, NOT NULL**. | | **config\_json** | JSONB | A JSON object storing user customizations (e.g., {"color": "\#FF0000", "layout": "compact"}). | | **widget\_token**| VARCHAR(255)| **UNIQUE, NOT NULL**. A secure, public token used for authenticating the embed request. |*  
* ***Key Relationships & Cardinality:***  
  * *Teams **(1) \--- (N)** EmbeddableWidgets*  
  * *Events, Artists, etc. **(1) \--- (N)** EmbeddableWidgets (via polymorphic link).*

---

#### ***InboundWebhooks***

* ***Core Purpose:** Defines the endpoints and logic for receiving data from third-party automation platforms like Zapier and Make.com.*  
* ***Definition & Business Logic:** This table is the counterpart to WebhookSubscriptions. It allows users to create triggers for their automation workflows. For example, a user can create a "Create Event" webhook here. The system generates a unique endpoint\_url for them to use in Zapier. When Zapier sends a POST request to this URL, our system uses the target\_action and field\_mapping to securely create a new Event on behalf of the owning\_team\_id.*  
* ***Attributes:** | Column Name | Data Type | Constraints & Description | | :--- | :--- | :--- | | **inbound\_webhook\_id**| UUID | **Primary Key**. | | **owning\_team\_id**| UUID | **Foreign Key** \-\> teams(team\_id). **NOT NULL**. | | **platform** | ENUM | The originating platform, e.g., 'zapier', 'make', 'custom'. | | **endpoint\_url**| VARCHAR(255)| **UNIQUE, NOT NULL**. The unique URL our system listens on for this webhook. | | **target\_action**| VARCHAR(100)| **NOT NULL**. The action to perform (e.g., 'create\_event', 'add\_attendee'). | | **field\_mapping**| JSONB | A JSON object that maps incoming data fields to our internal entity attributes. | | **is\_active** | BOOLEAN | DEFAULT TRUE. |*  
* ***Key Relationships & Cardinality:***  
  * *Teams **(1) \--- (N)** InboundWebhooks*

---

#### ***ApiUsageLogs***

* ***Core Purpose:** Provides a detailed, auditable log of every single request made to the public-facing API.*  
* ***Definition & Business Logic:** This is a critical table for security, billing, and performance monitoring of our API. Every authenticated request using an ApiCredential is logged here. It records which application made the request, which endpoint was hit, the response status, and latency. This data is used to enforce the rate\_limit\_per\_minute defined in ApiCredentials and to bill Teams for API usage if we introduce a usage-based pricing model.*  
* ***Attributes:** | Column Name | Data Type | Constraints & Description | | :--- | :--- | :--- | | **log\_id** | BIGSERIAL | **Primary Key**. | | **credential\_id**| UUID | **Foreign Key** \-\> api\_credentials(credential\_id). **NOT NULL**. Which credential was used. | | **requesting\_ip**| INET | The IP address of the client making the request. | | **endpoint** | VARCHAR(255)| The API endpoint that was called (e.g., '/v1/events/123'). | | **status\_code** | INTEGER | The HTTP response code (e.g., 200, 403, 500). | | **response\_time\_ms**| INTEGER | The time taken to process the request. |*  
* ***Key Relationships & Cardinality:***  
  * *ApiCredentials **(1) \--- (N)** ApiUsageLogs*

---

### ***Answering Your Specific Questions***

* ***Where is the API key generated?** The long-lived credentials (client\_id and client\_secret) are generated in the **ApiCredentials** table when a ProMax Team registers a new **DeveloperApplication**.*  
* ***Where is its token?** The term "token" usually refers to the short-lived **access token**. This is not stored permanently in the database. It is generated on-the-fly via an OAuth 2.0 flow where the developer exchanges their client\_id and client\_secret for it. The active session associated with this token is then tracked in the **UserSessions** table in Schema 1\.*  
* ***When and how does it get refreshed?** The short-lived access token has an expiration time (e.g., 1 hour). Along with the access token, the developer also receives a long-lived **refresh token**. When the access token expires, the developer's application uses the refresh token to request a new access token from our authentication service without requiring the user to log in again. This is a standard, secure OAuth 2.0 pattern.*

---

### ***\#\# Microservice Interaction: A Connected Ecosystem***

*Yes, the architecture is explicitly designed for intelligent, seamless communication between microservices. They will collaborate to solve problems using a combination of three industry-standard patterns:*

1. ***Direct API Calls (for Synchronous Tasks):** For immediate, necessary actions, services will communicate directly.*  
   * ***Example:** When a user confirms a booking in the **Marketplace** service, it will make a direct, synchronous call to the **Finance** service to place an escrow hold in the Wallets table. The booking is not confirmed until the financial transaction succeeds.*  
2. ***Event-Driven Messaging (for Asynchronous Tasks):** For actions that don't need to block the user, services will communicate asynchronously via a message queue. This creates a resilient and decoupled system.*  
   * ***Example:** When an organizer uploads a new photo to an Event (handled by the **Media** service), an event.media.added message is published. The **Community** service listens for this message and creates a new Post on the event's social wall. The **Rewards** service also listens and asynchronously awards the user contribution points. The user's initial upload is instant, and the downstream effects happen reliably in the background.*  
3. ***Shared Data with Bounded Context:** While we have a unified ERD, our domain-driven design means each microservice "owns" its respective schema. A service can read data from another domain's database for context, but only the owning service can write to its tables.*  
   * ***Example:** The **AI** service can read Event and Venue data to generate a report, but it can only write to the AiUsageLogs or AiEnhancedContent tables, which it owns.*

---

### ***\#\# Vision Alignment: The Blueprint Realized***

*This final ERD v7.0 is the definitive data-level implementation of the EventfulHQ vision.*

* ***✅ AI-First:** The dedicated **AI & Machine Learning Domain** (AiModels, AiUsageLogs) and the AI-processing fields embedded in tables like WhatsappMessages make AI a core, auditable, and manageable part of the infrastructure, not just an add-on.*  
* ***✅ WhatsApp-First:** The dedicated **Communications Domain**, with its detailed tables for WhatsappConversations, Messages, and Templates, provides the deep integration required to deliver a true WhatsApp-native experience.*  
* ***✅ Unified B2B & B2C Ecosystem:** This is perfectly modeled by the flexible **Identity & Collaboration Domain**. The same Users table can support a C2C Fan planning a birthday party or a B2B Expert working for a corporate Team. The marketplaces are designed to serve both, using the same underlying Bookings and Sponsorships entities.*  
* ***✅ Low-Click & Intelligent Automation:** The entire ERD is built to support this. The ContractTemplates table allows for one-click contract generation. The EntityTranslations table automates multi-language support. The EventCategories table allows the AI to infer needs and make intelligent recommendations, all based on a single user selection.*  
* ***✅ The Interconnected Knowledge Graph:** The relationship graph you reviewed is the physical manifestation of our knowledge graph. The way a Booking links a User (as an Artist) to an Event, which is held at a Venue in a Destination, and is promoted by a Campaign run by a Team, is the core of our interconnected ecosystem.*

*The architecture is complete, vision-aligned, and ready.*

This v7.0 blueprint is the definitive data architecture for EventfulHQ. It is complete, robust, and directly aligned with the complex business requirements of a global, AI-powered enterprise platform.