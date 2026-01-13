# Unified Automation Framework

## Unified Automation AAA Training Materials (Audio, Video, Slides)

- 🎙️ **Audio Presentation**: [Listen here](docs/resources/%5BAudio%5DThe_Unified_Automation_AAA_02.mp4)


https://github.com/user-attachments/assets/332f707b-b06a-4754-8437-7ac9a100bf2d

- 📽️ **Video presentation**: [Watch here](docs/resources/%5BVideo%5DThe_Unified_Automation_AAA_02.mp4)

https://github.com/user-attachments/assets/fe3d455c-7648-491a-a2b7-27409a1608e6

- 📊 **Slides**: [Download PDF](docs/resources/%5BSlides%5DThe_Unified_Automation_AAA_02.pdf)

Welcome to the **Unified Automation Framework** documentation ("Automation AAA"). This repository serves as the architectural blueprint and documentation hub for our testing ecosystem.

## 🌟 Introduction

Our framework is designed to unify **API**, **Web**, and **Desktop** (Native) testing into a single, cohesive Monorepo using `Nx` and `Playwright`. It emphasizes:
-   **Stability**: Through strong typing and the AAA (Arrange-Act-Assert) pattern.
-   **Scalability**: Via Dockerized execution and sharding.
-   **Flexibility**: Testing complex DLP scenarios across operating systems.

## 📚 Documentation Index

### 🏗️ Architecture & Core Concepts
-   **[Architecture Overview](docs/architecture.md)**: Explore the Nx Monorepo structure, library organization, and dependency rules.
-   **[Test Assistants & AAA Pattern](docs/test-assistants.md)**: The core of the framework. Learn about the `TestUser` controller, the AAA interaction model, Page Object composition, and assistant lifecycle.
-   **[Framework Orchestration](docs/orchestration.md)**: Central `utilManager` facade, sharding coordination strategies, and utility categories.
-   **[Configuration Management](docs/configuration.md)**: Environment loading, Playwright config overrides, and facade pattern for configuration access.

### 👤 Test User & Authentication
-   **[Test User Management](docs/test-user-management.md)**: TestUser lifecycle, account pooling, and authentication management.
-   **[Token Management](docs/token-management.md)**: TestUserTokenProvider, automatic token refresh, retry logic with exponential backoff.

### 🎭 Driver Architecture
-   **[Drivers](docs/drivers.md)**: Dual-driver model (Platform Driver + Corporate Browser Driver), CDP integration, and lifecycle management.
-   **[Fixtures](docs/fixtures.md)**: Isolated vs persistent session modes, fixture helper classes, and attachment management.
-   **[Global Setup](docs/global-setup.md)**: SabGlobalSetupManager, Appium server management, and environment initialization.

### 🧪 Domain Testing Guides
-   **[SAB & Desktop Testing](docs/sab-testing.md)**: Deep dive into the "Dual-Driver" model (PlatformDriver + Corporate Browser) for testing native desktop apps.
-   **[API Testing](docs/api-testing.md)**: How to use Builder Patterns (`InputBuilder`, `ResponseBuilder`) for robust API automation.
-   **[Microsoft 365 Automation](docs/m365-automation.md)**: Strategies for handling complex Canvas, Iframes, and Context Menus in Office Online.

### 🛡️ Specialized Frameworks
-   **[Content Scanning Framework](docs/content-scanning-framework.md)**: An inside look at the Strategy-based framework for testing DLP/Data Security scenarios.

### 📊 Reporting & Utilities
-   **[Reporting & Logging](docs/reporting.md)**: ExecutionTracker, @Step decorator, Report Portal integration, and context-aware logging.
-   **[SAB Utilities](docs/utilities.md)**: File system operations, GitHub release management, installer handling, and QA web server.

### ⚙️ DevOps & Runtime
-   **[Infrastructure & CI/CD](docs/infrastructure.md)**: Docker containers, Windows support, sharding strategies, Report Portal, and Secrets management.

## 🚀 Quick Start (Blueprint)

This repository contains the *skeleton* code structure. To generate a new library matching our standards:

```bash
# Generate a new Feature library
npx nx g @nx/js:lib feature-my-logic --directory=libs/shared

# Generate a new Page Object (Component)
# (See manuals for template usage)
```
