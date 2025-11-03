# Automation Test Framework - Architecture & Design Documentation

## Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
  - [Monorepo Structure](#monorepo-structure)
  - [Service Orchestration](#service-orchestration)
  - [Library Organization](#library-organization)
  - [Testing Types Overview](#testing-types-overview)
  - [EAB Testing Architecture](#eab-testing-architecture)
- [Design Patterns](#design-patterns)
  - [AAA (Arrange-Act-Assert) Pattern](#aaa-arrange-act-assert-pattern)
  - [Builder Pattern](#builder-pattern)
  - [Factory Pattern](#factory-pattern)
  - [Facade Pattern](#facade-pattern)
  - [Repository Pattern](#repository-pattern)
- [Tech Stack & Dependencies](#tech-stack--dependencies)
  - [Core Technologies](#core-technologies)
  - [Library Dependencies by Domain](#library-dependencies-by-domain)
  - [Path Mapping Configuration](#path-mapping-configuration)
- [Architecture Diagrams](#architecture-diagrams)
  - [User Relationships Diagram](#user-relationships-diagram)
  - [Monorepo Structure](#monorepo-structure-diagram)
  - [Service Orchestration Flow](#service-orchestration-flow)
  - [Test User Lifecycle](#test-user-lifecycle)
  - [AAA Pattern Flow](#aaa-pattern-flow)
  - [Library Dependency Graph](#library-dependency-graph)
  - [CI/CD Pipeline](#cicd-pipeline)
  - [EAB Test Sharding Flow](#eab-test-sharding-flow)
- [Code Samples](#code-samples)
  - [TestUser Creation and Management](#testuser-creation-and-management)
  - [AAA Assistant Usage](#aaa-assistant-usage)
  - [Builder Pattern Examples](#builder-pattern-examples)
  - [Configuration Management](#configuration-management)
  - [EAB Testing Examples](#eab-testing-examples)
- [Complete Flows](#complete-flows)
  - [Test Execution Flow](#test-execution-flow)
  - [Test User Acquisition and Lifecycle](#test-user-acquisition-and-lifecycle)
  - [Configuration Loading and Override](#configuration-loading-and-override)
  - [Report Portal Integration](#report-portal-integration)
  - [EAB Test Execution Flow](#eab-test-execution-flow)
- [CI/CD Integration](#cicd-integration)
- [References](#references)

---

## Overview

This document provides a comprehensive overview of a sophisticated test automation framework built using a monorepo architecture. The framework orchestrates multiple testing services (API, Web Portal, EAB, Performance) while maintaining code reusability, consistency, and scalability through well-defined design patterns and architectural principles.

### Key Objectives

- **Unified Testing Infrastructure**: Integrate different testing types (API, UI, EAB, Performance) in a single repository
- **Code Reusability**: Share common utilities, types, and test assistants across different testing domains
- **Maintainability**: Organize code by domain and feature to minimize coupling and maximize cohesion
- **Scalability**: Support parallel test execution, dynamic test user management, and flexible configuration

---

## Architecture

### Monorepo Structure

The project uses **Nx** (Nrwl Extensions) as the monorepo management tool, enabling:

- **Dependency Management**: Automatic dependency graph computation and affected project detection
- **Code Sharing**: Type-safe imports across projects using TypeScript path mapping
- **Task Orchestration**: Parallel execution of linting, testing, and building tasks
- **Library Management**: Automated refactoring for library creation, renaming, moving, and removal

```
automation-test/
├── apps/                    # Test applications organized by testing type
│   ├── api/                 # API testing application
│   ├── web-portal/          # Web Portal UI testing application
│   ├── eab/                 # Enterprise Access Browser testing application
│   ├── performance/         # Performance testing application
│   ├── new-tenant/          # New tenant setup testing application
│   └── playwright.base.config.ts  # Base Playwright configuration
├── libs/                    # Shared libraries organized by domain
│   ├── api/                 # API-specific libraries
│   │   ├── type-api/        # API type definitions
│   │   ├── type-api-core/  # Core API types
│   │   └── util-api/       # API utility functions and builders
│   ├── eab/                 # EAB-specific libraries
│   │   ├── type-eab/        # EAB type definitions
│   │   └── util-eab/       # EAB utility functions
│   ├── web-portal/          # Web Portal-specific libraries
│   │   ├── type-web-portal/ # Web Portal type definitions
│   │   └── util-web-portal/ # Web Portal utility functions
│   └── shared/              # Cross-domain shared libraries
│       ├── feature-aaa/     # AAA pattern implementation
│       ├── feature-http-client/ # HTTP client utilities
│       ├── type-aaa/        # AAA type definitions
│       ├── type-auth/       # Authentication types
│       ├── type-error/      # Error type definitions
│       ├── type-node-process/ # Node.js process types
│       ├── util-aws-cognito/ # AWS Cognito utilities
│       ├── util-core/       # Core utilities (facade, logger, etc.)
│       └── util-playwright/ # Playwright-specific utilities
└── docs/                    # Documentation and resources
    └── resources/           # Diagrams and reference images
```

### Service Orchestration

The framework orchestrates different testing services through:

1. **Centralized Configuration**: Base Playwright configuration shared across apps with per-app overrides
2. **Shared Libraries**: Domain-specific and cross-domain libraries accessible via TypeScript path mapping
3. **Test User Repository**: Centralized test account management with database-backed locking mechanism
4. **Unified Reporting**: Report Portal integration for consolidated test result tracking

#### Configuration Hierarchy

```
┌──────────────────────────────────────────┐
│   playwright.base.config.ts              │  ← Base configuration
│   - Environment config loading            │
│   - Global test settings                  │
│   - Shared timeout values                 │
└───────────────┬───────────────────────────┘
                │
                ├────────────────────────┐
                │                        │
    ┌───────────▼────────────┐  ┌─────────▼──────────┐
    │ api/playwright.config │  │ web-portal/        │
    │ .ts                   │  │ playwright.config   │
    │                       │  │ .ts                 │
    │ - App-specific        │  │                     │
    │   reporters           │  │ - App-specific      │
    │ - Global setup        │  │   projects          │
    │ - Project configs     │  │ - Overrides         │
    └───────────────────────┘  └─────────────────────┘
```

### Library Organization

Libraries are organized by **domain** and **concern**:

#### Domain-Specific Libraries

- **`libs/api/`**: API testing utilities, types, and builders
- **`libs/eab/`**: Enterprise Access Browser testing utilities
- **`libs/web-portal/`**: Web Portal UI testing utilities

#### Shared Libraries

- **`libs/shared/feature-*`**: Feature implementations (AAA, HTTP client)
- **`libs/shared/type-*`**: Type definitions
- **`libs/shared/util-*`**: Utility functions
- **`libs/shared/error-*`**: Error handling

### Testing Types Overview

| Testing Type | Location | Description |
|-------------|----------|-------------|
| **API Testing** | `apps/api/` | GraphQL and RESTful API endpoint testing |
| **Web Portal Testing** | `apps/web-portal/` | End-to-end web application UI testing |
| **EAB Testing** | `apps/eab/` | Enterprise Access Browser automation testing |
| **Performance Testing** | `apps/performance/` | Performance and load testing |
| **New Tenant Testing** | `apps/new-tenant/` | Tenant setup and configuration testing |

### EAB Testing Architecture

EAB (Enterprise Access Browser) testing has a unique architecture that combines multiple components to automate native desktop application testing.

#### EAB Component Architecture

```
┌─────────────────────────────────────────────────────────┐
│                  EAB Test Architecture                   │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  ┌──────────────────────────────────────┐               │
│  │         Test Script                  │               │
│  │  (Playwright test.describe/test)     │               │
│  └──────────────┬───────────────────────┘               │
│                 │                                        │
│                 ▼                                        │
│  ┌──────────────────────────────────────┐               │
│  │      TestUser                         │               │
│  │  - useUnityAssistant()                │               │
│  │  - useMammothBrowserAssistant()       │               │
│  └──────┬─────────────────┬──────────────┘               │
│         │                 │                              │
│         │                 │                              │
│         ▼                 ▼                              │
│  ┌──────────────┐  ┌─────────────────────┐            │
│  │ Unity Assistant│  │ Mammoth Browser      │            │
│  │                │  │ Assistant           │            │
│  │ Unity Tray UI  │  │                     │            │
│  │ Operations     │  │ Browser Automation  │            │
│  └──────┬────────┘  └──────────┬──────────┘            │
│         │                      │                        │
│         │                      │                        │
│         ▼                      ▼                        │
│  ┌──────────────┐  ┌─────────────────────┐            │
│  │PlatformDriver│  │MammothBrowserDriver  │            │
│  │              │  │                      │            │
│  │ Windows:     │  │ CDP (Chrome DevTools │            │
│  │ WinAppDriver │  │ Protocol) Connection │            │
│  │              │  │                      │            │
│  │ macOS:       │  │ Remote Debugging     │            │
│  │ Appium Mac2  │  │ Port Management      │            │
│  └──────┬───────┘  └──────────┬──────────┘            │
│         │                      │                        │
│         └──────────┬───────────┘                        │
│                    │                                    │
│                    ▼                                    │
│         ┌─────────────────────┐                         │
│         │  EAB Application    │                         │
│         │  (Mammoth Browser)  │                         │
│         │                     │                         │
│         │  - Unity Tray       │                         │
│         │  - Browser Engine   │                         │
│         │  - CDP Endpoint     │                         │
│         └─────────────────────┘                         │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

#### Key Components

**1. PlatformDriver (Unity Tray Automation)**
- **Purpose**: Controls the Unity Tray application (system tray integration)
- **Platform Implementations**:
  - **WindowsDriver**: Uses WinAppDriver for Windows automation
  - **MacOSDriver**: Uses Appium Mac2 Driver for macOS automation
- **Responsibilities**:
  - Launch/terminate Unity Tray
  - Interact with Unity Tray UI elements
  - Execute system-level actions (keyboard shortcuts, window management)
  - Access desktop elements

**2. MammothBrowserDriver (Browser Automation)**
- **Purpose**: Controls the Mammoth Browser application via Chrome DevTools Protocol (CDP)
- **Connection Method**: CDP over WebSocket to `--remote-debugging-port`
- **Responsibilities**:
  - Initialize browser context
  - Manage browser tabs and pages
  - Execute JavaScript in browser context
  - Capture screenshots, videos
  - Monitor network traffic

**3. Unity Assistant**
- **Purpose**: Provides AAA pattern interface for Unity Tray operations
- **Components**: Arrange, Actions, Assert classes for Unity Tray interactions
- **Usage**: `testUser.useUnityAssistant()`

**4. MammothBrowserAssistant**
- **Purpose**: Provides AAA pattern interface for browser operations
- **Components**: Arrange, Actions, Assert classes for browser interactions
- **Usage**: `testUser.useMammothBrowserAssistant()`

#### EAB Test Structure Organization

EAB tests are organized by feature categories in a hierarchical structure:

```
apps/eab/tests/
├── 01-install/                    # Installation tests
│   ├── 01-010-001-install.win.msi.spec.ts
│   ├── 01-010-002-install.win.msix.spec.ts
│   └── 01-010-003-install.mac.spec.ts
├── 02-features/                   # Feature tests
│   ├── mammothBrowser/
│   │   ├── 010-login/            # Login feature tests
│   │   │   ├── 02-010-001-mammoth.login.spec.ts
│   │   │   └── 02-010-002-mammoth.login.loginPolicy.spec.ts
│   │   ├── 020-webDefault/       # Web default tests
│   │   │   ├── 02-020-001-mammoth.saas.spec.ts
│   │   │   ├── 02-020-002-mammoth.contentScanning.spec.ts
│   │   │   └── src/               # Shared test utilities
│   │   ├── 030-app/               # App launch tests
│   │   │   ├── 02-030-001-mammoth.app.rdp.spec.ts
│   │   │   ├── 02-030-002-mammoth.app.ssh.spec.ts
│   │   │   └── 02-030-003-mammoth.app.fileShare.spec.ts
│   │   └── 090-quit/              # Quit/cleanup tests
│   │       └── 02-090-001-mammoth.quit.spec.ts
│   └── unity/                     # Unity-specific tests
│       ├── 02-01-unity.login.spec.ts
│       └── 02-02-unity.quit.spec.ts
├── 03-logout/                     # Logout tests
│   └── 03-logout.spec.ts
└── 04-upgrade/                     # Upgrade tests
    ├── 04-010-upgrade.spec.ts
    └── 04-020-upgrade.msix.spec.ts
```

#### Test Naming Convention

EAB tests follow a structured naming pattern:
- **Format**: `{category}-{subcategory}-{sequence}-{feature}.spec.ts`
- **Example**: `02-010-001-mammoth.login.spec.ts`
  - `02`: Feature category (02 = features)
  - `010`: Subcategory (010 = login)
  - `001`: Sequence number
  - `mammoth.login`: Feature description

---

## Design Patterns

### AAA (Arrange-Act-Assert) Pattern

The framework implements a structured AAA pattern where each test assistant provides three distinct interfaces:

#### Pattern Structure

```typescript
// Example: LoginPolicyAPIAssistant structure
type TALoginPolicyAPI = {
  arrange: LoginPolicyAPIArrange;   // Setup and data preparation
  actions: LoginPolicyAPIActions;   // API operations execution
  assert: LoginPolicyAPIAssert;      // Result validation
  helper: LoginPolicyAPIArrange;    // Helper methods (same as arrange)
};
```

#### Arrange Phase

The `Arrange` class handles:
- **Input Payload Creation**: Generate create/update input payloads using builders
- **Preset Data Generation**: Create prerequisite test data
- **Expected Result Preparation**: Construct expected responses for validation

```typescript
// Example: Arrange phase
export class LoginPolicyAPIArrange extends TestAAA<LoginPolicyAPI> {
  @Step()
  async getCreateInputPayload(
    options?: Partial<TCreateLoginPolicyInput>
  ): Promise<TCreateLoginPolicyInput> {
    return await utilAPIForCreateInputOrBodyPayloadBuilder<
      TCreateLoginPolicyInput
    >(
      CreateLoginPolicyInputBuilder,
      options,
    );
  }

  @Step()
  async generatePresetLoginPolicyEntry(
    actions: LoginPolicyAPIActions,
  ): Promise<TCreateLoginPolicyResult> {
    const createInput = await this.getCreateInputPayload();
    return await actions.createLoginPolicy(createInput);
  }
}
```

#### Actions Phase

The `Actions` class executes:
- **CRUD Operations**: Create, Read, Update, Delete API calls
- **Result Tracking**: Store created resource IDs in TestUser notes
- **Error Handling**: Type-safe error result handling

```typescript
// Example: Actions phase
export class LoginPolicyAPIActions extends TestAAA<LoginPolicyAPI> {
  @Step()
  async createLoginPolicy(
    input: TCreateLoginPolicyInput,
  ): Promise<TCreateLoginPolicyResult> {
    const resp = await this.getApiService()
      .getGqlClient()
      .mutation<TCreateLoginPolicyResult, TCreateLoginPolicyErrorResult>(
        createLoginPolicyMutation,
        { input },
      );

    // Track created resources in TestUser notes
    if (smartTypeGuardian<TCreateLoginPolicyResult, TCreateLoginPolicyErrorResult>(
      resp,
      'createLoginPolicy'
    )) {
      this.getApiService()
        .getTestUser()
        .appendNote(NoteKey.LoginPolicyIds, [resp.createLoginPolicy.id]);
    }

    return resp;
  }
}
```

#### Assert Phase

The `Assert` class validates:
- **Response Structure**: Verify response properties exist
- **Data Consistency**: Compare actual vs expected results
- **Error Scenarios**: Validate error responses

```typescript
// Example: Assert phase
export class LoginPolicyAPIAssert {
  @Step()
  async loginPolicyCreatedOK(
    actualResult: TCreateLoginPolicyResult,
    expectedResult: TCreateLoginPolicyResult,
  ): Promise<void> {
    baseExpect(actualResult.createLoginPolicy, {
      message: AssertMessage.common.toBeTruthy('property createLoginPolicy'),
    }).toBeTruthy();

    this.assertLoginPolicyContent(
      actualResult.createLoginPolicy,
      expectedResult.createLoginPolicy,
    );
  }
}
```

### Builder Pattern

The framework extensively uses the Builder pattern for constructing complex objects:

#### Input Builders

Input builders generate request payloads with default values and customization options:

```typescript
// Example: Create Input Builder
export const CreateLoginPolicyInputBuilder = {
  async _default(): Promise<TCreateLoginPolicyInput> {
    return {
      name: await utilManager.faker().random.word(),
      description: await utilManager.faker().lorem.sentence(),
      isDenyAction: false,
      // ... other default properties
    };
  },
};

// Usage with customization
const createInput = await utilAPIForCreateInputOrBodyPayloadBuilder<
  TCreateLoginPolicyInput
>(
  CreateLoginPolicyInputBuilder,
  { isDenyAction: true } // Override default
);
```

#### Response Builders

Response builders construct expected API responses from input payloads:

```typescript
// Example: Response Builder
export const CreateLoginPolicyResponseBuilder = {
  async _default(
    input: TCreateLoginPolicyInput,
  ): Promise<TCreateLoginPolicyResult> {
    return {
      createLoginPolicy: {
        id: expect.any(String),
        name: input.name,
        description: input.description,
        isDenyAction: input.isDenyAction,
        // ... other mapped properties
      },
    };
  },
};
```

#### Builder Utility Functions

The framework provides utilities to simplify builder usage:

```typescript
// Generic builder function for create inputs
export async function utilAPIForCreateInputOrBodyPayloadBuilder<T>(
  builderCallBackFunction: { _default: () => Promise<T> },
  options?: Partial<T>,
): Promise<T> {
  const inputPayload = await builderCallBackFunction._default();
  
  if (options) {
    // Merge options into default payload
    utilManager.handler().object().smartObjUpdates({
      thisObj: inputPayload as object,
      otherObj: options,
      union: true,
    });
  }
  
  return inputPayload;
}
```

### Factory Pattern

The Factory pattern is used for TestUser creation through `TestUserGenerator`:

```typescript
// Example: TestUser Factory
export class TestUserGenerator {
  static async createAdminUser(
    options?: Partial<TUserBuilderOptions>
  ): Promise<TestUser> {
    return await TestUser.create({
      role: TEnumUserRole.ADMIN,
      ...options,
    });
  }

  static async createRandomUser(
    options?: Partial<TUserBuilderOptions>
  ): Promise<TestUser> {
    return await TestUser.create({
      role: TEnumUserRole.USER,
      username: utilManager.faker().internet.userName(),
      ...options,
    });
  }
}

// Usage in tests
const adminUser = await TestUserGenerator.createAdminUser();
const randomUser = await TestUserGenerator.createRandomUser();
```

### Facade Pattern

The Facade pattern is implemented through `utilManager`, providing a unified interface to various utilities:

```typescript
// Example: Utility Manager Facade
export const utilManager = {
  faker: () => utilFakerData,
  handler: () => ({
    object: () => utilObjectHandler,
    apiVersionHandler: () => utilVersionHandler,
    gmailClient: (oAuth2: TGmailOAuth2) => utilGmailClient(oAuth2),
    totp: () => new TOTPService(),
  }),
  logger: () => utilLogger,
  mapper: () => utilMapper,
  processNode: () => utilProcessNode,
  typeChecker: () => utilTypeChecker,
  reportPortal: () => utilReportPortal,
  // ... more utilities
};

// Usage
const randomName = utilManager.faker().internet.userName();
const env = utilManager.processNode().facade().getter().getNodeEnv();
```

### Repository Pattern

The Repository pattern manages test account lifecycle through `TestAccountRepository`:

```typescript
// Example: Test Account Repository
export class TestAccountRepository {
  /**
   * Acquires and locks a test account from the database
   * Supports hostname-based filtering and retry logic
   */
  async getAndLockTestAccount(
    options: TGetAndLockOptions
  ): Promise<TTestAccount> {
    // Implementation with retry logic, hostname filtering,
    // and database transaction management
  }

  /**
   * Releases locked test accounts
   * Supports targeted unlocking by username or bulk unlocking
   */
  async unlockTestAccounts(
    options: TUnlockTestAccountsOptions
  ): Promise<void> {
    // Implementation with transaction safety
  }
}
```

---

## Tech Stack & Dependencies

### Core Technologies

| Technology | Version | Purpose |
|-----------|--------|---------|
| **Nx** | 17.1.3 | Monorepo management and task orchestration |
| **Playwright** | ^1.52.0 | End-to-end testing framework |
| **TypeScript** | ^5.1.6 | Type-safe development |
| **GraphQL** | 16.8.1 | API query language |
| **Axios** | ^1.7.2 | HTTP client for RESTful APIs |
| **Node.js** | ^22.0.0 | Runtime environment |

### Library Dependencies by Domain

#### API Domain
```json
{
  "@graphql-codegen/cli": "^5.0.2",
  "@graphql-codegen/typescript": "^4.0.9",
  "graphql": "16.8.1",
  "graphql-request": "6.1.0",
  "gql-query-builder": "^3.8.0"
}
```

#### EAB Domain
```json
{
  "appium": "^2.17.1",
  "webdriverio": "^9.12.4",
  "robotjs": "^0.6.0",
  "node-pty": "^1.0.0"
}
```

#### Shared Utilities
```json
{
  "@aws-sdk/client-ec2": "^3.529.0",
  "amazon-cognito-identity-js": "^6.3.7",
  "date-fns": "^3.6.0",
  "@faker-js/faker": "^8.3.1",
  "mysql2": "^3.14.3"
}
```

### Path Mapping Configuration

TypeScript path mappings enable type-safe imports across the monorepo:

```json
{
  "compilerOptions": {
    "baseUrl": "./",
    "paths": {
      "@company-qa/api/type-api": ["libs/api/type-api/src/index.ts"],
      "@company-qa/api/util-api": ["libs/api/util-api/src/index.ts"],
      "@company-qa/shared/feature-aaa": ["libs/shared/feature-aaa/src/index.ts"],
      "@company-qa/shared/util-core": ["libs/shared/util-core/src/index.ts"],
      "@company-qa/web-portal/type-web-portal": ["libs/web-portal/type-web-portal/src/index.ts"]
    }
  }
}
```

---

## Architecture Diagrams

### User Relationships Diagram

The following diagram illustrates the relationships between TestUser, TestUserManager, and TestUserGenerator:

![TestUser Relationships](docs/resources/relationships-users-in-automation.jpeg)

*Diagram showing TestUser, TestUserManager, and TestUserGenerator relationships and lifecycle*

### Monorepo Structure Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                   automation-test                           │
│                     (Nx Monorepo)                           │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌────────────────────┐    ┌────────────────────┐             │
│  │      apps/         │    │      libs/         │             │
│  │                    │    │                    │             │
│  │  ┌──────────────┐ │    │  ┌──────────────┐ │             │
│  │  │  api/        │ │    │  │  api/        │ │             │
│  │  │              │ │    │  │              │ │             │
│  │  │  tests/      │ │    │  │  type-api/   │ │             │
│  │  │  config/     │ │    │  │  util-api/   │ │             │
│  │  └──────────────┘ │    │  └──────────────┘ │             │
│  │                    │    │                    │             │
│  │  ┌──────────────┐ │    │  ┌──────────────┐ │             │
│  │  │  web-portal/ │ │    │  │  eab/        │ │             │
│  │  │              │ │    │  │              │ │             │
│  │  │  tests/      │ │    │  │  type-eab/   │ │             │
│  │  │  config/     │ │    │  │  util-eab/   │ │             │
│  │  └──────────────┘ │    │  └──────────────┘ │             │
│  │                    │    │                    │             │
│  │  ┌──────────────┐ │    │  ┌──────────────┐ │             │
│  │  │  eab/        │ │    │  │  shared/     │ │             │
│  │  │              │ │    │  │              │ │             │
│  │  │  tests/      │ │    │  │  feature-aaa  │ │             │
│  │  │  config/     │ │    │  │  util-core   │ │             │
│  │  └──────────────┘ │    │  │  util-playwr │ │             │
│  │                    │    │  │  ...         │ │             │
│  │  ┌──────────────┐ │    │  └──────────────┘ │             │
│  │  │  performance/│ │    │                    │             │
│  │  └──────────────┘ │    │                    │             │
│  └────────────────────┘    └────────────────────┘             │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### Service Orchestration Flow

```
┌──────────────┐
│  Test Script │
└──────┬───────┘
       │
       ▼
┌─────────────────────────┐
│  Playwright Config      │
│  - Load base config     │
│  - Load app config      │
│  - Load env variables   │
└──────┬──────────────────┘
       │
       ▼
┌─────────────────────────┐
│  Test User Fixtures      │
│  - TestUserGenerator     │
│  - TestUserRepository   │
│  - Account acquisition   │
└──────┬──────────────────┘
       │
       ▼
┌─────────────────────────┐
│  AAA Assistant          │
│  - Arrange              │
│  - Actions              │
│  - Assert               │
└──────┬──────────────────┘
       │
       ▼
┌─────────────────────────┐
│  API Client             │
│  - GraphQL Client      │
│  - RESTful Client      │
└──────┬──────────────────┘
       │
       ▼
┌─────────────────────────┐
│  Report Portal          │
│  - Test results         │
│  - Attachments          │
└─────────────────────────┘
```

### Test User Lifecycle

```
                    ┌─────────────┐
                    │ Test Start  │
                    └──────┬──────┘
                           │
                           ▼
            ┌───────────────────────────┐
            │ TestUserGenerator        │
            │ - createAdminUser()      │
            │ - createRandomUser()     │
            └──────┬───────────────────┘
                   │
                   ▼
        ┌───────────────────────┐
        │ TestAccountRepository │
        │ - getAndLockAccount() │
        └──────┬────────────────┘
               │
               ▼
    ┌──────────────────────────────┐
    │ Database Account Pool        │
    │ - Query available accounts   │
    │ - Lock account (hostname)   │
    │ - Return account info       │
    └──────┬───────────────────────┘
           │
           ▼
┌──────────────────────┐
│ TestUser Instance    │
│ - username          │
│ - credentials       │
│ - Cognito session   │
└──────┬───────────────┘
       │
       ▼
┌──────────────────────┐
│ Test Execution       │
│ - useAPIAssistant()  │
│ - takeNote()         │
│ - appendNote()       │
└──────┬───────────────┘
       │
       ▼
┌──────────────────────┐
│ Test Cleanup         │
│ - Cleanup resources  │
│ - unlockAccount()    │
└──────────────────────┘
```

### AAA Pattern Flow

```
┌─────────────────────────────────────────────────────────┐
│                    Test Case                             │
└─────────────────────────────────────────────────────────┘
                            │
                            ▼
        ┌───────────────────────────────────┐
        │     ARRANGE Phase                 │
        │  ┌─────────────────────────┐    │
        │  │ Arrange Methods          │    │
        │  │ - getCreateInputPayload() │    │
        │  │ - generatePresetEntry()   │    │
        │  │ - getExpectedResult()     │    │
        │  └─────────────────────────┘    │
        └───────────────┬──────────────────┘
                        │
                        ▼
        ┌───────────────────────────────────┐
        │     ACTIONS Phase                 │
        │  ┌─────────────────────────┐    │
        │  │ Actions Methods          │    │
        │  │ - createResource()       │    │
        │  │ - updateResource()       │    │
        │  │ - getResource()          │    │
        │  │ - deleteResource()       │    │
        │  └─────────────────────────┘    │
        │         │                        │
        │         ▼                        │
        │  ┌─────────────┐                │
        │  │ API Client   │                │
        │  │ - GraphQL    │                │
        │  │ - RESTful    │                │
        │  └─────────────┘                │
        └───────────────┬──────────────────┘
                        │
                        ▼
        ┌───────────────────────────────────┐
        │     ASSERT Phase                  │
        │  ┌─────────────────────────┐    │
        │  │ Assert Methods           │    │
        │  │ - resourceCreatedOK()    │    │
        │  │ - resourceUpdatedOK()   │    │
        │  │ - resourceGotOK()       │    │
        │  │ - validateContent()     │    │
        │  └─────────────────────────┘    │
        └──────────────────────────────────┘
```

### Library Dependency Graph

```
┌─────────────────────────────────────────────────────────┐
│                      Apps Layer                         │
│                                                          │
│  ┌─────────┐  ┌────────────┐  ┌──────┐  ┌────────────┐ │
│  │  api/   │  │ web-portal/│  │ eab/ │  │performance/│ │
│  └────┬────┘  └──────┬──────┘  └───┬──┘  └──────┬─────┘ │
└───────┼──────────────┼──────────────┼─────────────┼──────┘
        │              │              │             │
        └──────────────┴──────────────┴─────────────┘
                         │
        ┌────────────────┼────────────────┐
        │                │                │
┌───────▼──────┐  ┌──────▼──────┐  ┌─────▼─────────┐
│ libs/api/    │  │ libs/eab/    │  │libs/web-portal│
│              │  │              │  │               │
│  type-api    │  │  type-eab    │  │  type-web-    │
│  util-api    │  │  util-eab    │  │  portal       │
└───────┬──────┘  └──────┬───────┘  └──────┬────────┘
        │                │                 │
        └────────────────┼─────────────────┘
                         │
        ┌────────────────▼─────────────────┐
        │        libs/shared/               │
        │                                   │
        │  ┌───────────────────────────┐   │
        │  │  feature-aaa              │   │
        │  │  feature-http-client      │   │
        │  │  util-core                │   │
        │  │  util-playwright          │   │
        │  │  util-aws-cognito         │   │
        │  │  type-aaa                 │   │
        │  │  type-auth                │   │
        │  └───────────────────────────┘   │
        └───────────────────────────────────┘
```

### CI/CD Pipeline

```
┌─────────────────────────────────────────────────────────┐
│                  Jenkins Pipeline                        │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  Stage 1: Prerequisite                                 │
│  ┌────────────────────────────────────────────┐         │
│  │ - Copy source files                       │         │
│  │ - Setup directories                       │         │
│  │ - Prepare environment                      │         │
│  └────────────────────────────────────────────┘         │
│                        │                                 │
│                        ▼                                 │
│  Stage 2: API Tests                                      │
│  ┌────────────────────────────────────────────┐         │
│  │ - Load credentials                         │         │
│  │ - Run API test suite                       │         │
│  │ - Generate HTML report                     │         │
│  └────────────────────────────────────────────┘         │
│                        │                                 │
│                        ▼                                 │
│  Stage 3: E2E Tests                                      │
│  ┌────────────────────────────────────────────┐         │
│  │ - Load credentials                         │         │
│  │ - Run Web Portal test suite                │         │
│  │ - Generate HTML report                     │         │
│  └────────────────────────────────────────────┘         │
│                        │                                 │
│                        ▼                                 │
│  Post Actions                                           │
│  ┌────────────────────────────────────────────┐         │
│  │ - Archive test reports                      │         │
│  │ - Publish HTML reports                      │         │
│  │ - Send notifications                        │         │
│  └────────────────────────────────────────────┘         │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### EAB Test Sharding Flow

EAB tests use Playwright's **shard feature** to distribute tests across multiple machines, with each machine running a single worker. This approach is necessary because EAB (Enterprise Access Browser) cannot run multiple instances on the same machine simultaneously.

**Key Characteristics:**
- **Sharding**: Tests are distributed across multiple machines using Playwright's `--shard=X/Y` flag
- **Single Worker Per Machine**: Each machine runs with `workers: 1` to avoid EAB instance conflicts
- **Machine-Level Isolation**: Each machine has its own EAB instance, Unity Tray, and test accounts
- **Parallel Execution**: Multiple machines execute their assigned shards in parallel

#### Test Distribution Strategy

```
┌─────────────────────────────────────────────────────────┐
│            EAB Test Sharding Architecture                │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  ┌────────────────────────────────────┐                │
│  │   Playwright Test Runner            │                │
│  │   (Coordinator)                     │                │
│  └──────────────┬─────────────────────┘                │
│                 │                                        │
│                 ▼                                        │
│  ┌────────────────────────────────────┐               │
│  │   Test Discovery                     │               │
│  │   - Scan tests/ directory            │               │
│  │   - Match: /.*spec.ts/               │               │
│  │   - Collect all test files           │               │
│  └──────────────┬─────────────────────┘                  │
│                 │                                        │
│                 ▼                                        │
│  ┌────────────────────────────────────┐                │
│  │   Shard Distribution                 │                │
│  │   - Total shards: N (e.g., 4)        │                │
│  │   - Distribute tests across shards   │                │
│  │   - Each shard runs on separate      │                │
│  │     machine                          │                │
│  └─────┬─────────────┬───────────┬─────┘                │
│        │             │           │                      │
│        ▼             ▼           ▼                      │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐    │
│  │ Machine 1    │ │ Machine 2    │ │ Machine N    │    │
│  │ (Shard 1/4)  │ │ (Shard 2/4)  │ │ (Shard N/4)  │    │
│  │              │ │              │ │              │    │
│  │ Worker: 1    │ │ Worker: 1    │ │ Worker: 1    │    │
│  │              │ │              │ │              │    │
│  │ ┌──────────┐ │ │ ┌──────────┐ │ │ ┌──────────┐ │    │
│  │ │ EAB Inst│ │ │ │ EAB Inst│ │ │ │ EAB Inst│ │    │
│  │ │ Unity   │ │ │ │ Unity   │ │ │ │ Unity   │ │    │
│  │ │ Platform│ │ │ │ Platform│ │ │ │ Platform│ │    │
│  │ │ Driver  │ │ │ │ Driver  │ │ │ │ Driver  │ │    │
│  │ └──────────┘ │ │ └──────────┘ │ │ └──────────┘ │    │
│  │              │ │              │ │              │    │
│  │ Tests A,B,C  │ │ Tests D,E,F  │ │ Tests X,Y,Z  │    │
│  └──────────────┘ └──────────────┘ └──────────────┘    │
│        │             │           │                      │
│        └─────────────┴───────────┘                      │
│                    │                                     │
│                    ▼                                     │
│       ┌─────────────────────────┐                        │
│       │  Resource Isolation      │                        │
│       │                          │                        │
│       │  Each Machine Has:       │                        │
│       │  - 1 Worker (required)    │                        │
│       │  - 1 EAB Instance         │                        │
│       │  - 1 Unity Tray Process   │                        │
│       │  - 1 PlatformDriver       │                        │
│       │  - 1 MammothBrowserDriver │                        │
│       │  - Test Accounts          │                        │
│       │    (hostname-filtered)    │                        │
│       └─────────────────────────┘                        │
│                                                          │
│  Note: Multiple workers on same machine is not          │
│        supported due to EAB instance limitations        │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

#### Test Project Configuration

EAB tests use a single Playwright project with pattern-based test matching:

```typescript
// apps/eab/playwright.config.ts
projects: [
  {
    name: 'e2e',
    testMatch: /.*spec.ts/,  // Matches all .spec.ts files
    use: {
      channel: 'chrome',
    },
    outputDir: 'out/e2e-ts-test-results/',
  },
]
```

#### Sharding Mechanism

**1. Test Discovery**
- Playwright scans `apps/eab/tests/` directory
- Matches files ending with `.spec.ts`
- Collects all test files for shard distribution

**2. Shard Distribution**
- Tests are distributed across N shards (configured via `--shard=X/Y` flag)
- Each shard runs on a separate machine
- Distribution algorithm balances test execution time across shards
- Example: `--shard=1/4` means machine 1 of 4 total machines

**3. Single Worker Per Machine**
- Each machine runs with `workers: 1` (configured in base config)
- This is required because EAB cannot run multiple instances on the same machine
- Tests within a shard run sequentially within the single worker

**4. Parallel Execution Across Machines**
- Multiple machines execute their assigned shards in parallel
- Each machine maintains isolated resources (EAB instance, Unity Tray, accounts)
- No resource sharing between machines

#### Execution Configuration

**Standard Configuration (Single Worker Per Machine):**
```
┌─────────────────────────────────────┐
│         Worker 1                    │
│                                     │
│  ┌─────────────────────────────┐   │
│  │  EAB Instance (Shared)      │   │
│  │  Unity Tray Process         │   │
│  │  PlatformDriver             │   │
│  └─────────────────────────────┘   │
│                                     │
│  Test Execution Order:              │
│  1. Test A (install)                │
│  2. Test B (login)                  │
│  3. Test C (webDefault)             │
│  4. Test D (app)                    │
│  5. Test E (quit)                   │
│                                     │
│  Resource Sharing:                  │
│  - EAB persists across tests        │
│  - Unity Tray persists              │
│  - Faster execution (no startup)    │
└─────────────────────────────────────┘
```

**Multiple Shards (Multiple Machines):**
```
┌─────────────────────────────────────────────────────────┐
│        Multiple Shards Across Multiple Machines         │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │ Machine 1    │  │ Machine 2    │  │ Machine 3    │  │
│  │ (Shard 1/3)  │  │ (Shard 2/3)  │  │ (Shard 3/3)  │  │
│  │              │  │              │  │              │  │
│  │ Worker: 1    │  │ Worker: 1    │  │ Worker: 1    │  │
│  │              │  │              │  │              │  │
│  │ EAB Inst 1  │  │ EAB Inst 2   │  │ EAB Inst 3   │  │
│  │ Unity 1     │  │ Unity 2      │  │ Unity 3      │  │
│  │              │  │              │  │              │  │
│  │ Test A       │  │ Test D       │  │ Test G       │  │
│  │ Test B       │  │ Test E       │  │ Test H       │  │
│  │ Test C       │  │ Test F       │  │ Test I       │  │
│  └──────────────┘  └──────────────┘  └──────────────┘  │
│                                                          │
│  Resource Isolation:                                     │
│  - Each machine has separate EAB instance               │
│  - Each machine has separate Unity Tray                 │
│  - Each machine has separate test accounts               │
│    (filtered by machine hostname)                       │
│  - No resource sharing between machines                 │
│                                                          │
│  Parallel Execution:                                     │
│  - Tests run simultaneously across machines             │
│  - Faster overall execution                             │
│  - Requires multiple machines                           │
│                                                          │
│  Command Example:                                        │
│  Machine 1: --shard=1/3                                 │
│  Machine 2: --shard=2/3                                 │
│  Machine 3: --shard=3/3                                 │
└─────────────────────────────────────────────────────────┘
```

#### Persistent Session Mode and Sharding

EAB supports persistent session mode, where EAB instance is shared across tests within the single worker on each machine:

**Persistent Mode (Standard Single Worker Configuration):**
```
┌─────────────────────────────────────┐
│         Worker 1 (Persistent Mode)  │
│                                     │
│  ┌─────────────────────────────┐   │
│  │  beforeAll:                 │   │
│  │  - Launch EAB once          │   │
│  │  - Initialize Unity Tray    │   │
│  └─────────────────────────────┘   │
│                                     │
│  Test Execution:                    │
│  ┌───────────────────────────┐   │
│  │  Test A: Use existing EAB   │   │
│  └───────────────────────────┘   │
│  ┌───────────────────────────┐   │
│  │  Test B: Use existing EAB  │   │
│  └───────────────────────────┘   │
│  ┌───────────────────────────┐   │
│  │  Test C: Use existing EAB  │   │
│  └───────────────────────────┘   │
│                                     │
│  ┌─────────────────────────────┐   │
│  │  afterAll:                  │   │
│  │  - Cleanup EAB              │   │
│  │  - Terminate Unity Tray     │   │
│  └─────────────────────────────┘   │
│                                     │
│  Benefits:                          │
│  - Faster execution (no per-test    │
│    EAB startup)                     │
│  - Reduced resource usage           │
│  - Shared state across tests        │
└─────────────────────────────────────┘
```

**Persistent Mode with Multiple Shards (Multiple Machines):**
```
┌─────────────────────────────────────────────────────────┐
│   Multiple Shards with Persistent Mode (Per Machine)     │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  ┌──────────────┐         ┌──────────────┐             │
│  │ Machine 1    │         │ Machine 2     │             │
│  │ (Shard 1/2)  │         │ (Shard 2/2)   │             │
│  │              │         │              │             │
│  │ Worker: 1    │         │ Worker: 1     │             │
│  │              │         │              │             │
│  │ EAB Inst 1   │         │ EAB Inst 2    │             │
│  │ (Persistent) │         │ (Persistent)  │             │
│  │              │         │              │             │
│  │ Tests A,B,C  │         │ Tests D,E,F   │             │
│  │ Share EAB 1  │         │ Share EAB 2   │             │
│  └──────────────┘         └──────────────┘             │
│                                                          │
│  Behavior:                                               │
│  - Each machine maintains its own persistent EAB        │
│  - Tests within a machine (shard) share EAB instance    │
│  - Tests across machines use different EAB instances    │
│  - Parallel execution with machine-level isolation       │
│                                                          │
│  Benefits:                                               │
│  - Faster execution (no per-test EAB startup)           │
│  - Reduced resource usage per machine                    │
│  - Scales horizontally with more machines               │
└─────────────────────────────────────────────────────────┘
```

#### Test Account Management Across Machines

Test accounts are managed using **hostname-based filtering**, which is **machine-level**. Since EAB tests use sharding with one worker per machine, each machine has a single worker that uses the machine's hostname for account filtering.

**Important Concepts:**
- **Hostname**: Extracted from the machine's OS hostname (e.g., VM-01, VM-02)
- **Shard**: A subset of tests assigned to a specific machine
- **Worker**: Single worker per machine (workers: 1 is required for EAB)
- **Account Filter**: Based on machine hostname, extracted as last 2 digits (e.g., "%01%" for VM-01)

```
┌─────────────────────────────────────────────────────────┐
│    Test Account Distribution (Hostname-Based)           │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  ┌──────────────────────────────────────┐               │
│  │    Database Account Pool             │               │
│  │    (Filtered by machine hostname)    │               │
│  └──────────────┬───────────────────────┘               │
│                 │                                         │
│                 ├──────────┬──────────┬──────────┐       │
│                 │          │          │          │       │
│                 ▼          ▼          ▼          ▼       │
│      ┌──────────────┐ ┌──────────────┐ ┌──────────────┐ │
│      │  Machine 1    │ │  Machine 2    │ │  Machine N    │ │
│      │  (Shard 1/3)  │ │  (Shard 2/3)  │ │  (Shard 3/3)  │ │
│      │               │ │               │ │               │ │
│      │  Hostname:    │ │  Hostname:    │ │  Hostname:    │ │
│      │  Extract "01" │ │  Extract "02" │ │  Extract "N" │ │
│      │  Filter: %01% │ │  Filter: %02% │ │  Filter: %N% │ │
│      │               │ │               │ │               │ │
│      │  ┌──────────┐ │ │  ┌──────────┐ │ │  ┌──────────┐ │ │
│      │  │ Worker:  │ │ │  │ Worker:  │ │ │  │ Worker:  │ │ │
│      │  │    1     │ │ │  │    1     │ │ │  │    1     │ │ │
│      │  └──────────┘ │ │  └──────────┘ │ │  └──────────┘ │ │
│      │               │ │               │ │               │ │
│      │  Account:     │ │  Account:     │ │  Account:     │ │
│      │  user-01      │ │  user-02      │ │  user-N       │ │
│      │  Locked: %01% │ │  Locked: %02% │ │  Locked: %N% │ │
│      └──────────────┘ └──────────────┘ └──────────────┘ │
│                 │          │          │                  │
│                 └──────────┴──────────┘                  │
│                          │                               │
│                          ▼                               │
│              ┌────────────────────────┐                  │
│              │   Test Execution       │                  │
│              │   (Parallel, Isolated) │                  │
│              └────────────────────────┘                  │
│                                                          │
│  Key Points:                                             │
│  - Hostname extracted from machine OS hostname           │
│  - All workers on same machine share same hostname       │
│  - Account filter: %{hostnameDigits}% (e.g., %01%)       │
│  - Accounts locked per account, not per worker           │
│  - Each machine (shard) uses accounts matching its       │
│    hostname pattern                                      │
└─────────────────────────────────────────────────────────┘
```

#### Sharding Execution Flow

```
1. Playwright Initialization
   │
   ├─> Load playwright.config.ts
   │    ├─> Load base config
   │    ├─> Load EAB-specific config
   │    └─> Configure workers: 1 (fixed for EAB)
   │
   ├─> Test Discovery
   │    ├─> Scan apps/eab/tests/ directory
   │    ├─> Match files: /.*spec.ts/
   │    └─> Collect all test files
   │
   ├─> Shard Distribution (if --shard=X/Y specified)
   │    ├─> Calculate total shards from --shard flag
   │    ├─> Distribute tests across shards
   │    ├─> Assign current shard to this machine
   │    └─> Balance test execution time across shards
   │
   ├─> Worker Initialization (Single Worker Per Machine)
   │    ├─> Global Setup
   │    │    ├─> Install platform packages (if needed)
   │    │    ├─> Verify Appium installation
   │    │    └─> Prepare test environment
   │    │
   │    └─> EAB Instance Setup (If Persistent Mode)
   │         ├─> Launch EAB with --remote-debugging-port
   │         ├─> Initialize PlatformDriver
   │         └─> Initialize MammothBrowserDriver
   │
   ├─> Test Execution (Within Single Worker)
   │    │
   │    ├─> For each test in shard's queue:
   │    │    ├─> Test Setup (beforeEach/beforeAll)
   │    │    │    └─> Initialize EAB (if not persistent)
   │    │    │
   │    │    ├─> Test Run
   │    │    │    ├─> Create TestUser
   │    │    │    │    ├─> Extract hostname from machine OS
   │    │    │    │    ├─> Build filter: %{hostnameDigits}%
   │    │    │    │    ├─> Acquire account: Query DB with hostname filter
   │    │    │    │    └─> Lock account in database
   │    │    │    ├─> Use Unity/MammothBrowser Assistant
   │    │    │    ├─> Execute test steps
   │    │    │    └─> Verify results
   │    │    │
   │    │    └─> Test Cleanup (afterEach/afterAll)
   │    │         ├─> Cleanup resources (if not persistent)
   │    │         └─> Unlock test accounts
   │    │
   │    └─> Worker Completion
   │         ├─> Cleanup EAB instance (if persistent)
   │         ├─> Unlock test accounts
   │         └─> Report results
   │
   └─> Global Teardown (Per Machine)
        ├─> Terminate EAB instance
        ├─> Cleanup Unity Tray process
        └─> Release machine resources
```

---

## Code Samples

### TestUser Creation and Management

#### Basic TestUser Creation

```typescript
import { testFixturesAPI as test } from '@company-qa/shared/util-playwright';
import { TestUserGenerator } from '@company-qa/shared/feature-aaa';

test.describe('Example Test Suite', () => {
  test('Create and use test users', async ({ adminUser }) => {
    // Admin user is provided via fixture
    const adminUserAPITA = adminUser.useAPIAssistant<
      UserApiManager,
      TAUserAPI
    >(UserApiManager);

    // Create a random user using generator
    const randomUser = await TestUserGenerator.createRandomUser({
      subDomainName: 'main',
    });

    // Register user in Cognito
    await randomUser.cognitoSignUp();

    // Use TestUserManager to track users
    TestUserManager.userIntoPlay(randomUser);

    // Perform test actions
    // ...

    // Cleanup: users are automatically cleaned up via TestUserManager
    TestUserManager.takeABow(randomUser);
  });
});
```

#### TestUser with Notes (Resource Tracking)

```typescript
test('Track created resources', async ({ adminUser }) => {
  const adminAPITA = await adminUser.useAPIAssistant<
    LoginPolicyApiManager,
    TALoginPolicyAPI
  >(LoginPolicyApiManager);

  // Arrange: Create input payload
  const createInput = await adminAPITA.arrange.getCreateInputPayload();

  // Actions: Create resource
  const actualResult = await adminAPITA.actions.createLoginPolicy(createInput);

  // The created ID is automatically stored in TestUser notes
  // Access via: adminUser.getNote(NoteKey.LoginPolicyIds)

  // Assert: Verify creation
  const expectedResult = await adminAPITA.arrange.getExpectedResultForCreatePolicyOK(
    createInput
  );
  await adminAPITA.assert.loginPolicyCreatedOK(actualResult, expectedResult);
});
```

### AAA Assistant Usage

#### Complete CRUD Example

```typescript
test.describe('LoginPolicy API Tests', () => {
  test('Create LoginPolicy @C1234 sanity', async ({ adminUser }) => {
    // Arrange
    const adminAPITA = await adminUser.useAPIAssistant<
      LoginPolicyApiManager,
      TALoginPolicyAPI
    >(LoginPolicyApiManager);

    const createInput = await adminAPITA.arrange.getCreateInputPayload({
      isDenyAction: false,
    });

    // Actions
    const actualResult = await adminAPITA.actions.createLoginPolicy(createInput);

    // Assert
    const expectedResult = await adminAPITA.helper.getExpectedResultForCreatePolicyOK(
      createInput
    );
    await adminAPITA.assert.loginPolicyCreatedOK(actualResult, expectedResult);
  });

  test('Update LoginPolicy @C1235 sanity', async ({ adminUser }) => {
    // Arrange
    const adminAPITA = await adminUser.useAPIAssistant<
      LoginPolicyApiManager,
      TALoginPolicyAPI
    >(LoginPolicyApiManager);

    // Preset: Create a policy first
    const presetPolicy = await adminAPITA.arrange.generatePresetLoginPolicyEntry(
      adminAPITA.actions
    );

    const updateInput = await adminAPITA.arrange.getUpdateInputPayload(
      presetPolicy,
      { isDenyAction: true }
    );

    // Actions
    const actualResult = await adminAPITA.actions.updateLoginPolicy(updateInput);

    // Assert
    const expectedResult = await adminAPITA.helper.getExpectedResultForUpdatePolicyOK(
      updateInput
    );
    await adminAPITA.assert.loginPolicyUpdatedOK(actualResult, expectedResult);
  });

  test('Delete LoginPolicy @C1236 sanity', async ({ adminUser }) => {
    // Arrange
    const adminAPITA = await adminUser.useAPIAssistant<
      LoginPolicyApiManager,
      TALoginPolicyAPI
    >(LoginPolicyApiManager);

    const presetPolicy = await adminAPITA.arrange.generatePresetLoginPolicyEntry(
      adminAPITA.actions
    );

    // Actions
    const deleteResult = await adminAPITA.actions.deleteLoginPolicy(
      presetPolicy.createLoginPolicy.id
    );

    // Assert
    await adminAPITA.assert.loginPolicyDeletedOK(
      deleteResult,
      presetPolicy
    );

    // Verify deletion: Get should return null
    const getResult = await adminAPITA.actions.getLoginPolicy(
      presetPolicy.createLoginPolicy.id
    );
    await adminAPITA.assert.loginPolicyGotNull(getResult);
  });
});
```

### Builder Pattern Examples

#### Create Input Builder

```typescript
// Input builder with default values
export const CreateLoginPolicyInputBuilder = {
  async _default(): Promise<TCreateLoginPolicyInput> {
    return {
      name: await utilManager.faker().random.word(),
      description: await utilManager.faker().lorem.sentence(),
      isDenyAction: false,
      conditions: {
        and: [
          {
            datetime: {
              daysOfWeek: ['Monday', 'Friday'],
              hours: { start: '09:00', end: '17:00' },
              timeZone: 'America/New_York',
            },
          },
        ],
      },
    };
  },
};

// Usage with customization
const createInput = await utilAPIForCreateInputOrBodyPayloadBuilder<
  TCreateLoginPolicyInput
>(
  CreateLoginPolicyInputBuilder,
  { 
    isDenyAction: true,  // Override default
    name: 'Custom Policy Name'  // Override default
  }
);
```

#### Response Builder

```typescript
// Response builder constructs expected result from input
export const CreateLoginPolicyResponseBuilder = {
  async _default(
    input: TCreateLoginPolicyInput,
  ): Promise<TCreateLoginPolicyResult> {
    return {
      createLoginPolicy: {
        id: expect.any(String),
        name: input.name,
        description: input.description,
        isDenyAction: input.isDenyAction,
        conditions: input.conditions,
        createdAt: expect.any(String),
        updatedAt: expect.any(String),
      },
    };
  },
};

// Usage in Arrange phase
const expectedResult = await utilAPIForInputOrResponseBuilder(
  CreateLoginPolicyResponseBuilder,
  actualResult.createLoginPolicy,  // Use actual result as base
  { /* additional overrides */ }
);
```

### Configuration Management

#### Environment Configuration Loading

```typescript
// Base configuration loader
export function loadEnvConfig(rootPath: string): void {
  const env = utilManager
    .processNode()
    .facade()
    .getter()
    .getNodeEnv();  // Get NODE_ENV, defaults to 'qa'

  const dotenvPath = path.resolve(
    rootPath,
    `./config/.env.${env}`
  );

  config({ path: dotenvPath });
}

// Global environment values (shared across apps)
export function getGlobalEnvValues() {
  return {
    tenant: {
      region: 'us-east-1',
      clientId: 'xxx',
      userPoolId: 'xxx',
      subs: [/* sub-tenant configs */]
    },
    hostNamePool: {
      BASE_URL: 'https://example.com',
      APP_SYNC: 'https://api.example.com/graphql',
      GRAPHQL_API: 'https://api.example.com/graphql',
    },
    testSettings: {
      API_VERSION: 'v4-3-0',
      LONG_TIMEOUT: 60000,
      NORMAL_TIMEOUT: 30000,
    },
  };
}
```

#### Playwright Configuration Inheritance

```typescript
// Base configuration (apps/playwright.base.config.ts)
export const baseConfig: PlaywrightTestConfig = {
  testDir: './tests',
  timeout: utilManager.processNode().facade().getter().getTimeouts().testDefault,
  expect: {
    timeout: utilManager.processNode().facade().getter().getTimeouts().expectDefault,
  },
  fullyParallel: true,
  retries: 0,
  workers: 1,
  use: {
    baseURL: utilManager.processNode().facade().getter().getWebPortalOldBaseUrl(),
    actionTimeout: utilManager.processNode().facade().getter().getTimeouts().action,
    trace: 'retain-on-failure',
    headless: true,
  },
};

// App-specific configuration (apps/api/playwright.config.ts)
module.exports = defineConfig({
  ...baseConfig,  // Inherit base config
  reporter: utilPlaywright.reporters.getPlaywrightReporter({
    launch: 'api',
    description: 'API E2E Test',
  }),
  globalSetup: require.resolve('./src/fixtures/globalSetup'),
  projects: [
    {
      name: 'api testing',
      testMatch: /.*\.(api\.ts|api\.js)$/,
      outputDir: 'out/api-test-results/',
    },
  ],
});
```

### EAB Testing Examples

#### Basic EAB Test with Unity Assistant

```typescript
import { testFixturesEAB as test } from '@company-qa/shared/util-playwright';
import { UnityTrayPage } from '@company-qa/shared/feature-aaa';

test.describe('Unity Tray Tests', () => {
  test('Login via Unity Tray @C2001 sanity', async ({ adminUser }) => {
    // Arrange: Get Unity Assistant
    const unityTrayTA = await adminUser.useUnityAssistant<
      UnityTrayPage,
      TAUnityTrayPage
    >(UnityTrayPage);

    // Actions: Open Unity Tray and perform login
    await unityTrayTA.actions.openTray();
    await unityTrayTA.actions.gotoAbout();

    // Assert: Verify Unity Tray is accessible
    await unityTrayTA.assert.trayOpenedOK();
  });
});
```

#### EAB Test with Mammoth Browser Assistant

```typescript
import { testFixturesEAB as test } from '@company-qa/shared/util-playwright';
import { MBAppLauncherPage } from '@company-qa/shared/feature-aaa';

test.describe('Mammoth Browser Tests', () => {
  test('Launch SaaS Application @C2002 sanity', async ({ adminUser }) => {
    // Arrange: Get Mammoth Browser Assistant
    const mbTA = await adminUser.useMammothBrowserAssistant<
      MBAppLauncherPage,
      TAMBAppLauncherPage
    >(MBAppLauncherPage);

    // Actions: Launch application
    await mbTA.actions.launchSaaSApp('example-app');

    // Assert: Verify application launched
    await mbTA.assert.appLaunchedOK('example-app');
  });
});
```

#### EAB Test with Persistent Session Mode

```typescript
import { testFixturesEAB as test } from '@company-qa/shared/util-playwright';
import { createHooksManager } from '@company-qa/shared/feature-aaa';

test.describe('EAB Persistent Session Tests', () => {
  // Configure persistent mode for all tests in this describe block
  test.use({
    eabSessionMode: {
      mode: 'persistent',
    },
  });

  // Configure parallel execution within worker
  test.describe.configure({ mode: 'parallel' });

  let hooksManager: ReturnType<typeof createHooksManager>;
  let globalUser: TestUser;

  test.beforeAll(async ({ }) => {
    // Initialize hooks manager
    hooksManager = createHooksManager(test);
    
    globalUser = await TestUserGenerator.createAdminUser();

    // Setup persistent EAB session
    hooksManager
      .initialize()
      .configure({
        eabSessionMode: 'persistent',
        enableLogging: true,
      })
      .beforeAll()
      .setupEabSession()
      .afterAll()
      .cleanupEabAndTestEntities();
  });

  test('First test using persistent EAB @C2003 sanity', async ({ }) => {
    const mbTA = await globalUser.useMammothBrowserAssistant<
      MBAppLauncherPage,
      TAMBAppLauncherPage
    >(MBAppLauncherPage);

    // EAB instance is already running from beforeAll
    await mbTA.actions.navigateToUrl('https://example.com');
    await mbTA.assert.pageLoadedOK();
  });

  test('Second test reusing same EAB @C2004 sanity', async ({ }) => {
    // This test reuses the same EAB instance from previous test
    const mbTA = await globalUser.useMammothBrowserAssistant<
      MBAppLauncherPage,
      TAMBAppLauncherPage
    >(MBAppLauncherPage);

    // Faster execution - no EAB startup overhead
    await mbTA.actions.navigateToUrl('https://another-site.com');
    await mbTA.assert.pageLoadedOK();
  });

  test.afterAll(async () => {
    // Cleanup persistent EAB instance
    await hooksManager.executeAfterAll();
  });
});
```

#### EAB Test with Content Scanning

```typescript
import { ContentScanningTestManager } from './content-scanning-test-manager';
import { testFixturesEAB as test } from '@company-qa/shared/util-playwright';

test.describe('Content Scanning Tests', () => {
  const contentScanningTestManager = new ContentScanningTestManager();

  // Get test data based on configuration
  const getTestData = () => {
    const config = utilManager
      .processNode()
      .facade()
      .getter()
      .getEabConfigHandler()
      .getContentScanningTestConfig();

    return contentScanningTestManager.getSensitiveTestDataWithOptions(config);
  };

  const testData = getTestData();

  testData.forEach(({ type, values, priority }) => {
    test(`Test ${type} detection - ${priority} priority`, async ({ adminUser }) => {
      // Arrange
      const mbTA = await adminUser.useMammothBrowserAssistant<
        MBAppLauncherPage,
        TAMBAppLauncherPage
      >(MBAppLauncherPage);

      const qaWebServerTA = await contentScanningTestManager.setupQAWebServerPage(
        adminUser,
        'Content Scanning Policy'
      );

      // Actions: Test copy functionality with sensitive content
      for (const value of values) {
        await contentScanningTestManager.testCopyFunctionalityWithSensitiveContent(
          qaWebServerTA,
          value
        );
      }

      // Assert: Verify content scanning logs
      const logs = await contentScanningTestManager.verifyContentScanningLogs({
        adminUser,
        policyName: 'Content Scanning Policy',
        appTags: [TEnumValuesLogAppTag.PageCopyBlock],
        eventTriggerTime: [],
        shouldContainBlock: true,
      });

      expect(logs.length).toBeGreaterThan(0);
    });
  });
});
```

---

## Complete Flows

### Test Execution Flow

```
1. Developer writes test script
   │
   ├─> Uses Playwright fixtures (adminUser, pendingRandomUser, etc.)
   │
   ├─> Test script imports AAA assistant
   │    import { UserApiManager } from '@company-qa/shared/feature-aaa';
   │
   ├─> Test execution begins
   │    │
   │    ├─> Playwright loads configuration
   │    │    ├─> Load base config (playwright.base.config.ts)
   │    │    ├─> Load app config (apps/api/playwright.config.ts)
   │    │    └─> Load environment variables (.env.{env})
   │    │
   │    ├─> Test fixtures are injected
   │    │    ├─> TestUserGenerator creates users
   │    │    ├─> TestAccountRepository acquires accounts from DB
   │    │    └─> Users are registered in TestUserManager
   │    │
   │    ├─> Test runs AAA pattern
   │    │    ├─> Arrange: Prepare input payloads
   │    │    ├─> Actions: Execute API calls
   │    │    └─> Assert: Validate results
   │    │
   │    └─> Cleanup phase
   │         ├─> Delete created resources (from TestUser notes)
   │         ├─> Unlock test accounts
   │         └─> Release resources
   │
   └─> Results reported
        ├─> Playwright HTML report
        └─> Report Portal (if configured)
```

### Test User Acquisition and Lifecycle

```
1. TestUserGenerator.createAdminUser() called
   │
   ├─> TestUser.create() invoked
   │    │
   │    ├─> Check if user provided in fixture options
   │    │    └─> If yes, use provided user
   │    │
   │    └─> If not, acquire from repository
   │         │
   │         ├─> TestAccountRepository.getAndLockTestAccount()
   │         │    │
   │         │    ├─> Determine hostname (VM-01, VM-02, etc.)
   │         │    │
   │         │    ├─> Build username filter (e.g., '%01%')
   │         │    │
   │         │    ├─> Query database with retry logic
   │         │    │    ├─> SELECT ... WHERE lock_flag = 0
   │         │    │    ├─> UPDATE ... SET lock_flag = 1
   │         │    │    └─> Return account info
   │         │    │
   │         │    └─> If no account found, retry with delay
   │         │
   │         ├─> TestUser instance created
   │         │    ├─> username, password stored
   │         │    ├─> Cognito session initialized (lazy)
   │         │    └─> Notes storage initialized
   │         │
   │         └─> TestUserManager.userIntoPlay(user)
   │              └─> User tracked in worker/parallel index map
   │
2. During test execution
   │
   ├─> User performs actions
   │    ├─> cognitoSignUp() - Register in Cognito
   │    ├─> useAPIAssistant() - Get AAA assistant (auto-auth)
   │    ├─> takeNote() - Store temporary values
   │    └─> appendNote() - Add to existing notes (e.g., resource IDs)
   │
3. Test cleanup
   │
   ├─> TestUserManager.takeABow(user) called (auto or manual)
   │    │
   │    ├─> Cleanup resources based on NoteKeys
   │    │    ├─> Delete in dependency order (e.g., ContentProfileIds before PolicyIds)
   │    │    └─> For each NoteKey, delete associated resources
   │    │
   │    ├─> TestAccountRepository.unlockTestAccounts()
   │    │    └─> UPDATE ... SET lock_flag = 0 WHERE username = ?
   │    │
   │    └─> Remove user from TestUserManager tracking
```

### Configuration Loading and Override

```
Configuration Hierarchy (bottom to top priority):

1. Global Environment Values (libs/shared/util-core/util-node-globalEnvValues.ts)
   - Tenant configurations
   - Hostname pools (URLs)
   - Test settings (timeouts, API versions)
   │
   └─> Used by: All apps via utilManager.processNode()

2. Base Playwright Config (apps/playwright.base.config.ts)
   - Base timeout values
   - Default test settings
   - Shared use options
   │
   └─> Used by: All app playwright.config.ts files

3. App Environment Config (apps/{app}/config/.env.{env})
   - App-specific environment variables
   - API endpoints
   - Credentials (via SHARED_PASSWORD)
   │
   └─> Loaded by: loadEnvConfig() in app's playwright.config.ts

4. App Playwright Config (apps/{app}/playwright.config.ts)
   - Reporter configuration
   - Project definitions
   - Global setup hooks
   │
   └─> Extends: Base config with app-specific overrides

5. Command Line Environment Variables
   - NODE_ENV={env}
   - SHARED_PASSWORD={password}
   - RP_API_KEY={key}
   - RP_MODE={mode}
   │
   └─> Highest priority, overrides all above

Override Flow:
┌─────────────────────────┐
│ Global Env Values       │  ← Base layer
└──────────┬──────────────┘
           │
           ▼
┌─────────────────────────┐
│ Base Playwright Config  │  ← Inherited
└──────────┬──────────────┘
           │
           ▼
┌─────────────────────────┐
│ App Env Config          │  ← App-specific
└──────────┬──────────────┘
           │
           ▼
┌─────────────────────────┐
│ App Playwright Config   │  ← Merged with base
└──────────┬──────────────┘
           │
           ▼
┌─────────────────────────┐
│ CLI Env Variables       │  ← Final overrides
└─────────────────────────┘
```

### Report Portal Integration

```
Test Execution
   │
   ├─> Playwright runs tests
   │    │
   │    ├─> Test steps tracked via @Step() decorator
   │    │    └─> Step metadata: name, message, timestamp
   │    │
   │    └─> Test results collected
   │         ├─> Pass/Fail status
   │         ├─> Screenshots (on failure)
   │         ├─> Videos (on failure)
   │         └─> Traces (on failure)
   │
   ├─> Report Portal Reporter (if RP_MODE != DISABLED)
   │    │
   │    ├─> Initialize Report Portal client
   │    │    └─> Use RP_API_KEY from environment
   │    │
   │    ├─> Start test launch
   │    │    ├─> Create launch with attributes
   │    │    │    ├─> testAppType: 'api' | 'web-portal' | 'eab'
   │    │    │    ├─> type: 'e2e'
   │    │    │    └─> env: 'qa' | 'dev' | 'demo' | 'prod'
   │    │    │
   │    │    └─> Mode determines visibility
   │    │         ├─> DEFAULT: Visible in main dashboard
   │    │         └─> DEBUG: Visible in debug tab only
   │    │
   │    ├─> For each test
   │    │    ├─> Create test item
   │    │    ├─> Report test steps
   │    │    ├─> Attach screenshots/videos
   │    │    └─> Report test result (PASSED/FAILED)
   │    │
   │    └─> Finish launch
   │
   └─> Results available in Report Portal
        └─> Accessible via web UI or API
```

### EAB Test Execution Flow

```
1. Test Command Execution
   │
   ├─> Nx command: npx nx run eab:test@default
   │    └─> Executes: npx playwright test --reporter=html --project=e2e
   │
   ├─> Playwright Initialization
   │    ├─> Load playwright.config.ts
   │    │    ├─> Load base config
   │    │    ├─> Fetch EAB build info from GitHub
   │    │    ├─> Configure Report Portal attributes
   │    │    └─> Setup web server for testing
   │    │
   │    └─> Global Setup
   │         ├─> Install platform packages (Appium drivers)
   │         ├─> Verify Appium server
   │         └─> Prepare EAB build (if globalSetup enabled)
   │
   ├─> Test Discovery and Sharding
   │    ├─> Scan apps/eab/tests/ directory
   │    ├─> Match files: /.*spec.ts/
   │    ├─> Collect test files
   │    └─> Distribute tests across shards (if --shard specified)
   │
   ├─> Worker Initialization (Single Worker Per Machine)
   │    └─> EAB Environment Setup (If Persistent Mode)
   │         ├─> Launch EAB with --remote-debugging-port=9222
   │         ├─> Initialize PlatformDriver
   │         │    ├─> Windows: Connect to WinAppDriver
   │         │    └─> macOS: Connect to Appium Mac2 Driver
   │         └─> Initialize MammothBrowserDriver
   │              └─> Connect via CDP to EAB instance
   │
   ├─> Test Execution (Within Single Worker)
   │    │
   │    ├─> For each test in shard queue:
   │    │    │
   │    │    ├─> Test Setup (beforeEach/beforeAll)
   │    │    │    ├─> Create TestUser (or reuse from fixture)
   │    │    │    │    ├─> Extract hostname from machine OS
   │    │    │    │    ├─> Build username filter: %{hostnameDigits}%
   │    │    │    │    ├─> Acquire account: Query DB WHERE lock_flag = 0 AND username LIKE filter
   │    │    │    │    └─> Lock account: UPDATE ... SET lock_flag = 1
   │    │    │    ├─> Register user in TestUserManager
   │    │    │    └─> Initialize EAB (if not persistent mode)
   │    │    │
   │    │    ├─> Test Run
   │    │    │    ├─> Get Unity Assistant (testUser.useUnityAssistant())
   │    │    │    │    └─> PlatformDriver controls Unity Tray
   │    │    │    │
   │    │    │    ├─> Get MammothBrowser Assistant (testUser.useMammothBrowserAssistant())
   │    │    │    │    └─> MammothBrowserDriver controls browser via CDP
   │    │    │    │
   │    │    │    ├─> Execute test steps
   │    │    │    │    ├─> Unity Tray operations (open, navigate, close)
   │    │    │    │    ├─> Browser operations (navigate, click, fill)
   │    │    │    │    └─> Content scanning verification
   │    │    │    │
   │    │    │    └─> Assert results
   │    │    │    │
   │    │    └─> Test Cleanup (afterEach/afterAll)
   │    │         ├─> Cleanup test resources (if not persistent)
   │    │         └─> Store test results
   │    │
   │    └─> Worker Completion
   │         ├─> Cleanup persistent EAB instance (if applicable)
   │         ├─> Unlock test accounts
   │         ├─> Terminate PlatformDriver connection
   │         └─> Report results to Playwright
   │
   ├─> Global Teardown (Per Machine)
   │    ├─> Terminate EAB instance
   │    ├─> Cleanup Unity Tray process
   │    └─> Release Appium connection
   │
   ├─> Result Aggregation
   │    ├─> Collect results from all machines/shards
   │    ├─> Generate HTML report
   │    └─> Send to Report Portal (if configured)
   │
   └─> Test Completion
        ├─> HTML report available
        └─> Report Portal updated (if enabled)
```

**Key Differences from API/Web Portal Tests:**

1. **Platform Drivers**: EAB tests require platform-specific drivers (WinAppDriver/Appium Mac2)
2. **CDP Connection**: MammothBrowserDriver connects via Chrome DevTools Protocol
3. **Native Application**: Tests interact with desktop application, not just browser
4. **Resource Isolation**: Each worker maintains separate EAB and Unity Tray instances
5. **Persistent Mode**: Optional mode to share EAB instance across tests within a worker
6. **Hostname-Based Account Assignment**: Test accounts assigned based on worker hostname

---

## CI/CD Integration

### Jenkins Pipeline Structure

The framework supports multiple Jenkins pipelines for different testing scenarios:

#### Main Pipeline (Jenkinsfile)
- **Schedule**: Daily at 8:30 AM
- **Stages**:
  1. Prerequisite: Copy files, setup directories
  2. API Tests: Execute API test suite
  3. E2E Tests: Execute Web Portal test suite
- **Post Actions**: Archive reports, publish HTML, send notifications

#### Platform-Specific Pipelines
- **Jenkinsfile.macos**: macOS-specific EAB tests
  - Runs on `macbook-slave` agent
  - Executes API and E2E tests on macOS
  - Uses `--headed` flag for E2E tests
- **Jenkinsfile.windows**: Windows-specific EAB tests
  - Runs on `windows-slave` agent
  - Executes Unity/Mammoth Browser tests on Windows
  - Filters tests with `-g "Mammoth Browser"` pattern
- **Jenkinsfile.idp**: Identity Provider tests
- **Jenkinsfile.auto**: Automated test execution

#### Pipeline Configuration Example

```groovy
pipeline {
    options {
        disableConcurrentBuilds()
        timeout(time: 1, unit: 'HOURS')
    }

    agent {
        label 'jenkins-slave'
    }

    stages {
        stage('Prerequisite') {
            steps {
                container('playwright') {
                    sh '''
                        cp -a playwright/src /app
                        cp -a playwright/tests /app
                        mkdir -p /app/out/api_html
                        mkdir -p /app/out/e2e_html
                    '''
                }
            }
        }

        stage('API Tests') {
            steps {
                container('playwright') {
                    withCredentials([
                        usernamePassword(
                            credentialsId: 'automation-test',
                            usernameVariable: 'ADMIN_USERNAME',
                            passwordVariable: 'ADMIN_PASSWORD'
                        )
                    ]) {
                        sh '''
                            cd /app
                            export PLAYWRIGHT_HTML_REPORT=out/api_html
                            npx playwright test --project="api testing"
                        '''
                    }
                }
            }
        }
    }

    post {
        always {
            publishHTML target: [
                reportName: 'Playwright result',
                reportDir: 'out',
                reportFiles: 'api_html/index.html, e2e_html/index.html',
                keepAll: true
            ]
        }
    }
}
```

---

## References

### Key Documentation Files

- **Main README**: `/automation-test/README.md` - Comprehensive project overview
- **Apps Documentation**: `/automation-test/apps/README.md` - App organization guide
- **Libraries Documentation**: `/automation-test/libs/README.md` - Library structure
- **API Testing Guide**: `/automation-test/apps/api/README.md` - API testing specifics
- **Web Portal Guide**: `/automation-test/apps/web-portal/README.md` - UI testing guide
- **EAB Guide**: `/automation-test/apps/eab/README.md` - EAB testing guide
- **Structure Guidelines**: `/automation-test/docs/structure_guideline.md` - Code organization rules

### Key Diagrams and Resources

- **User Relationships**: `docs/resources/relationships-users-in-automation.jpeg` - TestUser, TestUserManager, TestUserGenerator relationships
- **Configuration Examples**: Various screenshots in `docs/resources/` showing configuration setup

### Design Pattern References

- **AAA Pattern**: Based on standard Arrange-Act-Assert testing pattern
- **Builder Pattern**: Implementation inspired by GoF Builder pattern
- **Factory Pattern**: TestUser creation follows Factory method pattern
- **Facade Pattern**: utilManager provides unified interface to utilities
- **Repository Pattern**: TestAccountRepository follows data access repository pattern

---

## Conclusion

This automation test framework demonstrates a sophisticated approach to test automation through:

1. **Monorepo Architecture**: Efficient code sharing and dependency management using Nx
2. **Design Pattern Implementation**: Consistent application of established patterns (AAA, Builder, Factory, Facade, Repository)
3. **Service Orchestration**: Unified configuration and execution across multiple testing types
4. **Scalability**: Support for parallel execution, dynamic user management, and flexible configuration
5. **Maintainability**: Clear separation of concerns, domain-driven organization, and comprehensive documentation

The framework successfully integrates API, UI, EAB, and Performance testing into a cohesive, maintainable, and scalable testing infrastructure.

