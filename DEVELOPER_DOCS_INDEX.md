# n8n Developer Documentation Index

> **Complete guide to contributing to n8n's Frontend and Backend**

Welcome to the n8n developer documentation! This collection of guides will help you understand the codebase, make architectural decisions, and contribute effectively to the n8n workflow automation platform.

---

## 📚 Documentation Overview

### 🚀 [Developer Quick Start Guide](./DEVELOPER_QUICK_START.md)
**Perfect for:** First-time contributors, quick reference

**What you'll learn:**
- 5-minute setup guide
- Common development tasks
- Essential commands
- Code patterns and anti-patterns
- Quick troubleshooting

**Start here if:** You want to start contributing immediately

---

### 📖 [Developer Contribution Guide](./DEVELOPER_CONTRIBUTION_GUIDE.md)
**Perfect for:** Comprehensive understanding, detailed implementation

**What you'll learn:**
- Complete architecture overview
- Frontend development (Vue 3, Pinia, CSS)
- Backend development (Express, TypeORM, DI)
- Testing strategies (Jest, Vitest, Playwright)
- Code quality standards
- Git workflow and PR process

**Start here if:** You want in-depth knowledge of the entire system

---

### 🏗️ [Architecture Diagram Guide](./ARCHITECTURE_DIAGRAM.md)
**Perfect for:** Visual learners, understanding system design

**What you'll learn:**
- System architecture diagrams
- Request flow visualizations
- Component relationships
- Data flow patterns
- CI/CD pipeline
- Testing strategy

**Start here if:** You prefer visual explanations and diagrams

---

### 🎯 [Developer Decision Guide](./DEVELOPER_DECISION_GUIDE.md)
**Perfect for:** Making architectural choices, solving specific problems

**What you'll learn:**
- When to use what (components, stores, services)
- Error handling decisions
- Testing strategy selection
- Performance optimization
- Common scenarios and solutions
- Decision checklists

**Start here if:** You need help making implementation decisions

---

## 🗺️ Learning Paths

### Path 1: Complete Beginner

```
1. Quick Start Guide (30 min)
   ↓
2. Contribution Guide - Getting Started section (20 min)
   ↓
3. Architecture Diagrams - System Overview (15 min)
   ↓
4. Pick a small issue and start coding!
```

### Path 2: Frontend Developer

```
1. Quick Start Guide - Frontend section (15 min)
   ↓
2. Contribution Guide - Frontend Development (45 min)
   ↓
3. Architecture Diagrams - Component Architecture (15 min)
   ↓
4. Decision Guide - Frontend Decisions (20 min)
   ↓
5. Build a component!
```

### Path 3: Backend Developer

```
1. Quick Start Guide - Backend section (15 min)
   ↓
2. Contribution Guide - Backend Development (45 min)
   ↓
3. Architecture Diagrams - Backend Module Structure (15 min)
   ↓
4. Decision Guide - Backend Decisions (20 min)
   ↓
5. Build an API endpoint!
```

### Path 4: Full-Stack Developer

```
1. Quick Start Guide (30 min)
   ↓
2. Contribution Guide - Complete read (2 hours)
   ↓
3. Architecture Diagrams - All sections (30 min)
   ↓
4. Decision Guide - All sections (45 min)
   ↓
5. Build a full feature!
```

---

## 🎓 Topic-Specific Guides

### Frontend Topics

| Topic | Quick Start | Full Guide | Diagrams | Decisions |
|-------|-------------|------------|----------|-----------|
| Vue Components | ✅ | ✅ | ✅ | ✅ |
| Pinia Stores | ✅ | ✅ | ✅ | ✅ |
| CSS Variables | ✅ | ✅ | - | ✅ |
| i18n | ✅ | ✅ | - | ✅ |
| Testing | ✅ | ✅ | ✅ | ✅ |

### Backend Topics

| Topic | Quick Start | Full Guide | Diagrams | Decisions |
|-------|-------------|------------|----------|-----------|
| Controllers | ✅ | ✅ | ✅ | ✅ |
| Services | ✅ | ✅ | ✅ | ✅ |
| Repositories | ✅ | ✅ | ✅ | ✅ |
| Error Handling | ✅ | ✅ | ✅ | ✅ |
| Testing | ✅ | ✅ | ✅ | ✅ |

### Cross-Cutting Topics

| Topic | Quick Start | Full Guide | Diagrams | Decisions |
|-------|-------------|------------|----------|-----------|
| API Types | ✅ | ✅ | ✅ | ✅ |
| TypeScript | ✅ | ✅ | - | ✅ |
| Git Workflow | ✅ | ✅ | ✅ | - |
| Performance | ✅ | ✅ | - | ✅ |

---

## 🔍 Quick Reference

### I want to...

| Goal | Document | Section |
|------|----------|---------|
| Set up my dev environment | [Quick Start](./DEVELOPER_QUICK_START.md) | Quick Setup |
| Understand the architecture | [Architecture](./ARCHITECTURE_DIAGRAM.md) | System Architecture |
| Create a Vue component | [Contribution Guide](./DEVELOPER_CONTRIBUTION_GUIDE.md) | Frontend Development |
| Add an API endpoint | [Quick Start](./DEVELOPER_QUICK_START.md) | Add New API Endpoint |
| Write tests | [Contribution Guide](./DEVELOPER_CONTRIBUTION_GUIDE.md) | Testing Guidelines |
| Decide where to put code | [Decision Guide](./DEVELOPER_DECISION_GUIDE.md) | When to Use What |
| Fix a bug | [Decision Guide](./DEVELOPER_DECISION_GUIDE.md) | Scenario 2 |
| Optimize performance | [Decision Guide](./DEVELOPER_DECISION_GUIDE.md) | Performance Considerations |
| Add translations | [Quick Start](./DEVELOPER_QUICK_START.md) | Add Translation |
| Create a PR | [Contribution Guide](./DEVELOPER_CONTRIBUTION_GUIDE.md) | Git Workflow & Pull Requests |

---

## 📋 Checklists

### Before You Start Coding

- [ ] Read [Quick Start Guide](./DEVELOPER_QUICK_START.md)
- [ ] Set up development environment
- [ ] Understand the feature/bug you're working on
- [ ] Check [Decision Guide](./DEVELOPER_DECISION_GUIDE.md) for architectural choices
- [ ] Review existing similar code

### Before You Create a PR

- [ ] Code follows TypeScript best practices (no `any`, no `as`)
- [ ] All UI text uses i18n
- [ ] CSS uses variables (no hardcoded values)
- [ ] Tests written and passing
- [ ] Linting passes: `pnpm lint`
- [ ] Type checking passes: `pnpm typecheck`
- [ ] PR title follows conventions
- [ ] Linear ticket linked in description

### Code Review Checklist

- [ ] Code is clear and maintainable
- [ ] Tests cover edge cases
- [ ] Error handling is appropriate
- [ ] Performance considerations addressed
- [ ] Documentation updated if needed
- [ ] No security vulnerabilities introduced

---

## 🎯 Common Tasks Quick Links

### Frontend

- **[Create Component](./DEVELOPER_QUICK_START.md#create-vue-component)**
- **[Add Translation](./DEVELOPER_QUICK_START.md#add-translation)**
- **[Create Store](./DEVELOPER_CONTRIBUTION_GUIDE.md#2-pinia-store-pattern)**
- **[Use CSS Variables](./DEVELOPER_CONTRIBUTION_GUIDE.md#3-css-variables--styling)**

### Backend

- **[Add API Endpoint](./DEVELOPER_QUICK_START.md#add-new-api-endpoint)**
- **[Create Service](./DEVELOPER_CONTRIBUTION_GUIDE.md#2-service-pattern)**
- **[Handle Errors](./DEVELOPER_CONTRIBUTION_GUIDE.md#5-error-handling)**
- **[Write Tests](./DEVELOPER_QUICK_START.md#write-unit-test)**

### Testing

- **[Unit Tests](./DEVELOPER_CONTRIBUTION_GUIDE.md#unit-tests-jest)**
- **[Integration Tests](./DEVELOPER_CONTRIBUTION_GUIDE.md#http-mocking-nock)**
- **[E2E Tests](./DEVELOPER_CONTRIBUTION_GUIDE.md#e2e-tests-playwright)**

---

## 🛠️ Essential Commands

```bash
# Setup
pnpm install
pnpm build > build.log 2>&1

# Development
pnpm dev              # Full stack
pnpm dev:be           # Backend only
pnpm dev:fe           # Frontend only

# Testing
pnpm test             # All tests
pnpm typecheck        # Type checking
pnpm lint             # Linting

# Package-specific
cd packages/cli && pnpm dev
cd packages/editor-ui && pnpm dev
```

---

## 📁 Key Directories

```
n8n/
├── packages/
│   ├── @n8n/api-types/          # Shared TypeScript types
│   ├── @n8n/design-system/      # Vue component library
│   ├── @n8n/i18n/              # Translations
│   ├── cli/                     # Backend (Express + TypeORM)
│   │   ├── src/controllers/    # API endpoints
│   │   ├── src/services/       # Business logic
│   │   └── src/databases/      # Data access
│   ├── editor-ui/               # Frontend (Vue 3 + Pinia)
│   │   ├── src/components/     # Vue components
│   │   ├── src/stores/         # Pinia stores
│   │   └── src/api/            # API clients
│   ├── nodes-base/              # Built-in nodes
│   └── workflow/                # Core workflow engine
```

---

## 🌟 Best Practices Summary

### TypeScript
✅ Use proper types, not `any`  
✅ Use type guards instead of `as` casting  
✅ Define shared types in `@n8n/api-types`

### Frontend
✅ All UI text must use i18n  
✅ Always use CSS variables  
✅ Pure components go in design-system  
✅ Use Pinia for shared state

### Backend
✅ Controller → Service → Repository pattern  
✅ Use dependency injection  
✅ Proper error classes (not ApplicationError)  
✅ DTOs defined in `@n8n/api-types`

### Testing
✅ Write tests for all changes  
✅ Mock external dependencies  
✅ Test edge cases  
✅ Aim for high coverage on critical paths

---

## 🆘 Getting Help

### Documentation
- **n8n Docs**: https://docs.n8n.io
- **Community Forum**: https://community.n8n.io
- **Contributing Guide**: [CONTRIBUTING.md](./CONTRIBUTING.md)

### Code Examples
- **Controllers**: `packages/cli/src/controllers/users.controller.ts`
- **Services**: `packages/cli/src/services/user.service.ts`
- **Components**: `packages/editor-ui/src/components/`
- **Stores**: `packages/editor-ui/src/stores/`

### Internal Guides
- **Node Development**: `packages/nodes-base/AGENTS.md`
- **Frontend Patterns**: `packages/frontend/AGENTS.md`
- **Backend Modules**: `packages/cli/AGENTS.md`

---

## 📊 Documentation Statistics

| Document | Pages | Reading Time | Best For |
|----------|-------|--------------|----------|
| Quick Start | ~8 | 30 min | Getting started fast |
| Contribution Guide | ~25 | 2 hours | Deep understanding |
| Architecture Diagrams | ~6 | 30 min | Visual learners |
| Decision Guide | ~15 | 1 hour | Making choices |

**Total Reading Time**: ~4 hours for complete mastery

---

## 🎓 Recommended Learning Order

### Day 1: Orientation (2-3 hours)
1. Read [Quick Start Guide](./DEVELOPER_QUICK_START.md) - 30 min
2. Set up dev environment - 30 min
3. Skim [Contribution Guide](./DEVELOPER_CONTRIBUTION_GUIDE.md) - 30 min
4. Review [Architecture Diagrams](./ARCHITECTURE_DIAGRAM.md) - 30 min
5. Explore codebase - 1 hour

### Day 2: Deep Dive (3-4 hours)
1. Read [Contribution Guide](./DEVELOPER_CONTRIBUTION_GUIDE.md) fully - 2 hours
2. Study [Decision Guide](./DEVELOPER_DECISION_GUIDE.md) - 1 hour
3. Review existing code patterns - 1 hour

### Day 3: Practice (4-6 hours)
1. Pick a small issue - 30 min
2. Implement solution - 3 hours
3. Write tests - 1 hour
4. Create PR - 30 min

---

## 🚀 Next Steps

1. **Choose your learning path** from the options above
2. **Read the relevant documentation** for your role
3. **Set up your development environment** using the Quick Start Guide
4. **Explore the codebase** to see patterns in action
5. **Pick a task** from Linear or GitHub issues
6. **Start contributing!**

---

## 📝 Document Maintenance

These guides are living documents. If you find:
- **Outdated information** - Create an issue or PR
- **Missing topics** - Suggest additions in the forum
- **Unclear explanations** - Request clarification

**Last Updated**: February 2026  
**Maintained by**: n8n Community

---

## 🎉 Welcome to the n8n Community!

We're excited to have you contribute to n8n. Whether you're fixing a bug, adding a feature, or improving documentation, every contribution matters.

**Happy Coding!** 🚀

---

## Quick Navigation

- 🚀 [Quick Start](./DEVELOPER_QUICK_START.md)
- 📖 [Full Guide](./DEVELOPER_CONTRIBUTION_GUIDE.md)
- 🏗️ [Architecture](./ARCHITECTURE_DIAGRAM.md)
- 🎯 [Decisions](./DEVELOPER_DECISION_GUIDE.md)
- 📚 [Contributing](./CONTRIBUTING.md)
- 🏠 [Main README](./README.md)
