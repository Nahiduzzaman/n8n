# n8n Developer Contribution Guide

> **Comprehensive guide for contributing to n8n's Frontend and Backend**

This guide provides detailed patterns, conventions, and best practices for contributing to the n8n workflow automation platform. Whether you're working on the Vue.js frontend or the Node.js backend, this document will help you understand the codebase structure and development workflows.

---

## Table of Contents

- [Project Overview](#project-overview)
- [Getting Started](#getting-started)
- [Architecture Overview](#architecture-overview)
- [Frontend Development](#frontend-development)
- [Backend Development](#backend-development)
- [Testing Guidelines](#testing-guidelines)
- [Code Quality & Standards](#code-quality--standards)
- [Git Workflow & Pull Requests](#git-workflow--pull-requests)
- [Common Development Tasks](#common-development-tasks)
- [Troubleshooting](#troubleshooting)

---

## Project Overview

n8n is a workflow automation platform built with:
- **Frontend**: Vue 3 + TypeScript + Vite + Pinia
- **Backend**: Node.js + TypeScript + Express + TypeORM
- **Architecture**: Monorepo managed by pnpm workspaces with Turbo build orchestration
- **Testing**: Jest (unit) + Playwright (E2E) + Vitest (frontend)
- **Code Quality**: Biome (formatting) + ESLint + lefthook git hooks

---

## Getting Started

### Prerequisites

- **Node.js**: v22.16 or newer
- **pnpm**: v10.22 or newer (install via corepack)
- **Build tools**: gcc, g++, make (Linux), Xcode Command Line Tools (macOS)

### Installation

```bash
# Enable corepack
corepack enable
corepack prepare pnpm@10.22.0 --activate

# Clone the repository
git clone https://github.com/n8n-io/n8n.git
cd n8n

# Install dependencies
pnpm install

# Build all packages (redirect output to avoid clutter)
pnpm build > build.log 2>&1

# Check build status
tail -n 20 build.log
```

### Development Commands

```bash
# Full stack development (resource-intensive)
pnpm dev

# Backend only (excludes frontend packages)
pnpm dev:be

# Frontend only (editor-ui + design-system)
pnpm dev:fe

# AI/LangChain nodes development
pnpm dev:ai

# Start production build
pnpm start
```

### Recommended Development Setup

**For most development work (API + Frontend):**

```bash
# Terminal 1: Backend (CLI)
cd packages/cli
pnpm dev

# Terminal 2: Frontend (Editor UI)
cd packages/editor-ui
pnpm dev
```

---

## Architecture Overview

### Monorepo Structure

```
n8n/
├── packages/
│   ├── @n8n/
│   │   ├── api-types/          # Shared TypeScript interfaces (FE ↔ BE)
│   │   ├── design-system/      # Vue component library
│   │   ├── i18n/              # Internationalization
│   │   ├── config/            # Centralized configuration
│   │   └── di/                # Dependency injection container
│   ├── cli/                   # Express server, REST API, CLI
│   ├── editor-ui/             # Vue 3 frontend application
│   ├── workflow/              # Core workflow interfaces
│   ├── core/                  # Workflow execution engine
│   ├── nodes-base/            # Built-in nodes for integrations
│   └── @n8n/nodes-langchain/  # AI/LangChain nodes
├── .github/                   # CI/CD workflows
└── docker/                    # Docker configurations
```

### Key Architectural Patterns

1. **Dependency Injection**: Uses `@n8n/di` for IoC container
2. **Controller-Service-Repository**: Backend follows MVC-like pattern
3. **Event-Driven**: Internal event bus for decoupled communication
4. **State Management**: Frontend uses Pinia stores
5. **Design System**: Centralized reusable components in `@n8n/design-system`

---

## Frontend Development

### Technology Stack

- **Framework**: Vue 3 (Composition API)
- **State Management**: Pinia
- **Build Tool**: Vite
- **Styling**: CSS Variables + SCSS
- **Component Library**: `@n8n/design-system`
- **Testing**: Vitest + Playwright (E2E)

### Directory Structure

```
packages/editor-ui/
├── src/
│   ├── components/          # Vue components
│   ├── stores/             # Pinia stores
│   ├── views/              # Page-level components
│   ├── composables/        # Composition API utilities
│   ├── plugins/            # Vue plugins
│   ├── api/                # API client methods
│   └── constants/          # Constants and enums
```

### Frontend Development Patterns

#### 1. Component Structure

```vue
<script setup lang="ts">
import { ref, computed } from 'vue';
import { useI18n } from '@/composables/useI18n';
import { useToast } from '@/composables/useToast';

// Props
interface Props {
  title: string;
  isActive?: boolean;
}

const props = withDefaults(defineProps<Props>(), {
  isActive: false,
});

// Emits
const emit = defineEmits<{
  update: [value: string];
  close: [];
}>();

// Composables
const i18n = useI18n();
const toast = useToast();

// State
const internalValue = ref('');

// Computed
const displayTitle = computed(() => {
  return i18n.baseText('component.title', { interpolate: { title: props.title } });
});

// Methods
const handleUpdate = () => {
  emit('update', internalValue.value);
};
</script>

<template>
  <div class="component-wrapper">
    <h2>{{ displayTitle }}</h2>
    <!-- Component content -->
  </div>
</template>

<style lang="scss" scoped>
.component-wrapper {
  padding: var(--spacing--md);
  background: var(--color--background);
}
</style>
```

#### 2. Pinia Store Pattern

```typescript
import { defineStore } from 'pinia';
import { computed, ref } from 'vue';
import type { IUser } from '@n8n/api-types';
import * as usersApi from '@/api/users';

export const useUsersStore = defineStore('users', () => {
  // State
  const users = ref<IUser[]>([]);
  const currentUserId = ref<string | null>(null);

  // Getters
  const currentUser = computed(() => {
    return users.value.find((user) => user.id === currentUserId.value);
  });

  const allUsers = computed(() => users.value);

  // Actions
  async function fetchUsers() {
    try {
      const data = await usersApi.getUsers();
      users.value = data.items;
    } catch (error) {
      throw new Error('Failed to fetch users');
    }
  }

  async function updateUser(userId: string, data: Partial<IUser>) {
    await usersApi.updateUser(userId, data);
    const index = users.value.findIndex((u) => u.id === userId);
    if (index !== -1) {
      users.value[index] = { ...users.value[index], ...data };
    }
  }

  return {
    // State
    users,
    currentUserId,
    // Getters
    currentUser,
    allUsers,
    // Actions
    fetchUsers,
    updateUser,
  };
});
```

#### 3. CSS Variables & Styling

**Always use CSS variables** - never hardcode values:

```scss
// ✅ CORRECT
.component {
  padding: var(--spacing--md);
  color: var(--color--text);
  font-size: var(--font-size--sm);
  border-radius: var(--radius);
}

// ❌ WRONG
.component {
  padding: 20px;
  color: #333;
  font-size: 14px;
  border-radius: 4px;
}
```

**Available CSS Variables:**

```css
/* Spacing */
--spacing--5xs: 2px
--spacing--4xs: 4px
--spacing--3xs: 6px
--spacing--2xs: 8px
--spacing--xs: 12px
--spacing--sm: 16px
--spacing--md: 20px
--spacing--lg: 24px
--spacing--xl: 32px

/* Typography */
--font-size--3xs: 10px
--font-size--2xs: 12px
--font-size--xs: 13px
--font-size--sm: 14px
--font-size--md: 16px
--font-size--lg: 18px

/* Colors */
--color--primary
--color--success
--color--warning
--color--danger
--color--text
--color--background
--color--foreground
```

#### 4. Internationalization (i18n)

**All UI text MUST use i18n**:

```typescript
// In component
import { useI18n } from '@/composables/useI18n';

const i18n = useI18n();

// Simple translation
const title = i18n.baseText('users.title');

// With interpolation
const message = i18n.baseText('users.welcome', {
  interpolate: { name: userName },
});
```

**Adding translations** (`packages/@n8n/i18n/src/locales/en.json`):

```json
{
  "users": {
    "title": "User Management",
    "welcome": "Welcome, {name}!",
    "deleteConfirm": "Are you sure you want to delete this user?"
  }
}
```

#### 5. Debounce Timing

Use centralized constants instead of hardcoding:

```typescript
import { DEBOUNCE_TIME, getDebounceTime } from '@/app/constants';
import { useDebounceFn } from '@vueuse/core';

const debouncedSearch = useDebounceFn(() => {
  // Search logic
}, getDebounceTime(DEBOUNCE_TIME.INPUT.SEARCH));
```

#### 6. Testing Attributes

**data-testid must be a single value** (no spaces):

```vue
<!-- ✅ CORRECT -->
<button data-testid="save-button">Save</button>

<!-- ❌ WRONG -->
<button data-testid="save button">Save</button>
<button data-testid="save-button submit-button">Save</button>
```

### Frontend Development Workflow

1. **Create/modify components** in `packages/editor-ui/src/components/`
2. **Add reusable components** to `@n8n/design-system` for consistency
3. **Create Pinia stores** for state management
4. **Add i18n translations** to `@n8n/i18n`
5. **Use CSS variables** for all styling
6. **Write Vitest tests** for components
7. **Run typecheck** before committing: `pnpm typecheck`

---

## Backend Development

### Technology Stack

- **Runtime**: Node.js + TypeScript
- **Framework**: Express 5
- **ORM**: TypeORM
- **Database**: SQLite (dev) / PostgreSQL (production)
- **Queue**: Bull (Redis-based)
- **Testing**: Jest + Supertest + nock

### Directory Structure

```
packages/cli/
├── src/
│   ├── controllers/        # REST API endpoints (decorated with @RestController)
│   ├── services/          # Business logic layer
│   ├── databases/
│   │   ├── entities/      # TypeORM entities
│   │   └── repositories/  # Data access layer
│   ├── middlewares/       # Express middlewares
│   ├── decorators/        # Custom decorators (@Get, @Post, etc.)
│   ├── errors/            # Custom error classes
│   └── events/            # Event bus system
```

### Backend Development Patterns

#### 1. Controller Pattern

Controllers handle HTTP requests using decorators:

```typescript
import {
  RestController,
  Get,
  Post,
  Patch,
  Delete,
  Body,
  Param,
  Query,
  GlobalScope,
} from '@n8n/decorators';
import { Response } from 'express';
import type { AuthenticatedRequest } from '@n8n/db';
import { UserService } from '@/services/user.service';
import { BadRequestError } from '@/errors/response-errors/bad-request.error';
import { NotFoundError } from '@/errors/response-errors/not-found.error';

@RestController('/users')
export class UsersController {
  constructor(
    private readonly userService: UserService,
    private readonly logger: Logger,
  ) {}

  @Get('/')
  @GlobalScope('user:list')
  async listUsers(
    req: AuthenticatedRequest,
    _res: Response,
    @Query listQueryOptions: UsersListFilterDto,
  ) {
    const users = await this.userService.findAll(listQueryOptions);
    return { items: users, count: users.length };
  }

  @Get('/:id')
  @GlobalScope('user:read')
  async getUser(
    req: AuthenticatedRequest,
    _res: Response,
    @Param('id') id: string,
  ) {
    const user = await this.userService.findById(id);
    if (!user) {
      throw new NotFoundError('User not found');
    }
    return user;
  }

  @Post('/')
  @GlobalScope('user:create')
  async createUser(
    req: AuthenticatedRequest,
    _res: Response,
    @Body payload: CreateUserDto,
  ) {
    return await this.userService.create(payload);
  }

  @Patch('/:id')
  @GlobalScope('user:update')
  async updateUser(
    req: AuthenticatedRequest,
    _res: Response,
    @Param('id') id: string,
    @Body payload: UpdateUserDto,
  ) {
    return await this.userService.update(id, payload);
  }

  @Delete('/:id')
  @GlobalScope('user:delete')
  async deleteUser(
    req: AuthenticatedRequest,
    _res: Response,
    @Param('id') id: string,
  ) {
    await this.userService.delete(id);
    return { success: true };
  }
}
```

#### 2. Service Pattern

Services contain business logic:

```typescript
import { Service } from '@n8n/di';
import { Logger } from '@n8n/backend-common';
import { UserRepository, User } from '@n8n/db';
import { EventService } from '@/events/event.service';
import { InternalServerError } from '@/errors/response-errors/internal-server.error';

@Service()
export class UserService {
  constructor(
    private readonly logger: Logger,
    private readonly userRepository: UserRepository,
    private readonly eventService: EventService,
  ) {}

  async findAll(options?: ListOptions): Promise<User[]> {
    try {
      return await this.userRepository.find(options);
    } catch (error) {
      this.logger.error('Failed to fetch users', { error });
      throw new InternalServerError('Failed to fetch users');
    }
  }

  async findById(id: string): Promise<User | null> {
    return await this.userRepository.findOneBy({ id });
  }

  async create(data: CreateUserDto): Promise<User> {
    const user = this.userRepository.create(data);
    const savedUser = await this.userRepository.save(user);

    this.eventService.emit('user-created', {
      userId: savedUser.id,
      email: savedUser.email,
    });

    return savedUser;
  }

  async update(id: string, data: Partial<User>): Promise<User> {
    const user = await this.findById(id);
    if (!user) {
      throw new NotFoundError('User not found');
    }

    Object.assign(user, data);
    return await this.userRepository.save(user);
  }

  async delete(id: string): Promise<void> {
    await this.userRepository.delete({ id });
    this.eventService.emit('user-deleted', { userId: id });
  }

  getManager() {
    return this.userRepository.manager;
  }
}
```

#### 3. Repository Pattern

Repositories handle data access (TypeORM):

```typescript
import { Service } from '@n8n/di';
import { DataSource, Repository } from '@n8n/typeorm';
import { User } from '../entities/User';

@Service()
export class UserRepository extends Repository<User> {
  constructor(dataSource: DataSource) {
    super(User, dataSource.manager);
  }

  async findByEmail(email: string): Promise<User | null> {
    return await this.findOne({
      where: { email },
      relations: ['role', 'authIdentities'],
    });
  }

  async findManyByEmail(emails: string[]): Promise<User[]> {
    return await this.find({
      where: { email: In(emails) },
    });
  }

  buildUserQuery(options: UsersListFilterDto) {
    const query = this.createQueryBuilder('user')
      .leftJoinAndSelect('user.role', 'role')
      .leftJoinAndSelect('user.authIdentities', 'authIdentities');

    if (options.filter) {
      query.where('user.email LIKE :filter', {
        filter: `%${options.filter}%`,
      });
    }

    if (options.select) {
      query.select(options.select.map((field) => `user.${field}`));
    }

    return query;
  }
}
```

#### 4. DTO Validation (Zod)

Define DTOs in `packages/@n8n/api-types`:

```typescript
import { z } from 'zod';

// Schema definition
export const createUserSchema = z.object({
  email: z.string().email(),
  firstName: z.string().min(1).max(50),
  lastName: z.string().min(1).max(50),
  role: z.enum(['global:admin', 'global:member', 'global:chatUser']),
});

// Type inference
export type CreateUserDto = z.infer<typeof createUserSchema>;

// For use in controllers
export class CreateUserRequestDto extends createUserSchema {}
```

#### 5. Error Handling

**Use proper error classes** (NOT `ApplicationError` - it's deprecated):

```typescript
import { UnexpectedError, OperationalError, UserError } from 'n8n-workflow';
import { BadRequestError } from '@/errors/response-errors/bad-request.error';
import { NotFoundError } from '@/errors/response-errors/not-found.error';
import { ForbiddenError } from '@/errors/response-errors/forbidden.error';

// User-facing errors (4xx)
throw new BadRequestError('Invalid input data');
throw new NotFoundError('User not found');
throw new ForbiddenError('Insufficient permissions');

// Operational errors (expected errors)
throw new OperationalError('Database connection failed');

// User errors (user-caused issues)
throw new UserError('Invalid workflow configuration');

// Unexpected errors (bugs)
throw new UnexpectedError('Unexpected state encountered');
```

#### 6. Dependency Injection

Use `@n8n/di` for dependency injection:

```typescript
import { Service } from '@n8n/di';

@Service()
export class MyService {
  constructor(
    private readonly logger: Logger,
    private readonly userRepository: UserRepository,
    private readonly otherService: OtherService,
  ) {}

  // Service methods
}
```

### Backend Development Workflow

1. **Define API types** in `packages/@n8n/api-types`
2. **Create/modify controllers** in `packages/cli/src/controllers/`
3. **Implement services** in `packages/cli/src/services/`
4. **Create repositories** if needed in `packages/cli/src/databases/repositories/`
5. **Write Jest tests** with proper mocks
6. **Run typecheck**: `cd packages/cli && pnpm typecheck`
7. **Run tests**: `cd packages/cli && pnpm test`

---

## Testing Guidelines

### Unit Tests (Jest)

**Location**: Same directory as source file with `.test.ts` extension

```typescript
import { mock } from 'jest-mock-extended';
import { UserService } from '../user.service';
import { UserRepository } from '@/databases/repositories/user.repository';
import { EventService } from '@/events/event.service';
import { Logger } from '@n8n/backend-common';

describe('UserService', () => {
  let userService: UserService;
  let mockUserRepository: jest.Mocked<UserRepository>;
  let mockEventService: jest.Mocked<EventService>;
  let mockLogger: jest.Mocked<Logger>;

  beforeEach(() => {
    mockUserRepository = mock<UserRepository>();
    mockEventService = mock<EventService>();
    mockLogger = mock<Logger>();

    userService = new UserService(
      mockLogger,
      mockUserRepository,
      mockEventService,
    );
  });

  describe('findById', () => {
    it('should return user when found', async () => {
      const mockUser = { id: '123', email: 'test@example.com' };
      mockUserRepository.findOneBy.mockResolvedValue(mockUser);

      const result = await userService.findById('123');

      expect(result).toEqual(mockUser);
      expect(mockUserRepository.findOneBy).toHaveBeenCalledWith({ id: '123' });
    });

    it('should return null when user not found', async () => {
      mockUserRepository.findOneBy.mockResolvedValue(null);

      const result = await userService.findById('999');

      expect(result).toBeNull();
    });
  });

  describe('create', () => {
    it('should create user and emit event', async () => {
      const createData = { email: 'new@example.com', firstName: 'John' };
      const mockUser = { id: '456', ...createData };

      mockUserRepository.create.mockReturnValue(mockUser);
      mockUserRepository.save.mockResolvedValue(mockUser);

      const result = await userService.create(createData);

      expect(result).toEqual(mockUser);
      expect(mockEventService.emit).toHaveBeenCalledWith('user-created', {
        userId: '456',
        email: 'new@example.com',
      });
    });
  });
});
```

### HTTP Mocking (nock)

```typescript
import nock from 'nock';

describe('External API calls', () => {
  beforeEach(() => {
    nock.cleanAll();
  });

  it('should fetch data from external API', async () => {
    nock('https://api.example.com')
      .get('/users/123')
      .reply(200, { id: '123', name: 'John' });

    const result = await fetchUserFromExternalApi('123');

    expect(result).toEqual({ id: '123', name: 'John' });
  });

  it('should handle API errors', async () => {
    nock('https://api.example.com')
      .get('/users/999')
      .reply(404, { error: 'Not found' });

    await expect(fetchUserFromExternalApi('999')).rejects.toThrow('Not found');
  });
});
```

### Frontend Tests (Vitest)

```typescript
import { describe, it, expect, vi } from 'vitest';
import { mount } from '@vue/test-utils';
import { createTestingPinia } from '@pinia/testing';
import UserList from '../UserList.vue';

describe('UserList.vue', () => {
  it('renders user list correctly', () => {
    const wrapper = mount(UserList, {
      global: {
        plugins: [createTestingPinia({ createSpy: vi.fn })],
      },
      props: {
        users: [
          { id: '1', name: 'John' },
          { id: '2', name: 'Jane' },
        ],
      },
    });

    expect(wrapper.findAll('[data-testid="user-item"]')).toHaveLength(2);
  });

  it('emits delete event when delete button clicked', async () => {
    const wrapper = mount(UserList, {
      global: {
        plugins: [createTestingPinia()],
      },
      props: {
        users: [{ id: '1', name: 'John' }],
      },
    });

    await wrapper.find('[data-testid="delete-button"]').trigger('click');

    expect(wrapper.emitted('delete')).toBeTruthy();
    expect(wrapper.emitted('delete')?.[0]).toEqual(['1']);
  });
});
```

### E2E Tests (Playwright)

```typescript
import { test, expect } from '@playwright/test';

test.describe('User Management', () => {
  test('should create new user', async ({ page }) => {
    await page.goto('/users');

    await page.click('[data-testid="add-user-button"]');
    await page.fill('[data-testid="email-input"]', 'test@example.com');
    await page.fill('[data-testid="name-input"]', 'Test User');
    await page.click('[data-testid="save-button"]');

    await expect(page.locator('text=User created successfully')).toBeVisible();
  });
});
```

### Running Tests

```bash
# Backend unit tests
cd packages/cli
pnpm test

# Frontend tests
cd packages/editor-ui
pnpm test

# E2E tests
pnpm --filter=n8n-playwright test:local

# E2E tests with UI
pnpm --filter=n8n-playwright test:local --ui

# Run specific test
pnpm test UserService.test.ts
```

---

## Code Quality & Standards

### TypeScript Best Practices

```typescript
// ✅ CORRECT - Use proper types
function processUser(user: User): string {
  return user.name;
}

// ❌ WRONG - Never use 'any'
function processUser(user: any): any {
  return user.name;
}

// ✅ CORRECT - Use type guards
function isUser(obj: unknown): obj is User {
  return typeof obj === 'object' && obj !== null && 'id' in obj;
}

if (isUser(data)) {
  console.log(data.id); // TypeScript knows data is User
}

// ❌ WRONG - Avoid 'as' casting (except in tests)
const user = data as User;
```

### Linting & Formatting

```bash
# Run linter (from package directory)
cd packages/cli
pnpm lint

# Fix linting issues
pnpm lint:fix

# Format code
pnpm format

# Type checking
pnpm typecheck
```

### Pre-commit Checks

The project uses `lefthook` for git hooks. Before committing:

1. **Lint your changes**
2. **Run typecheck**
3. **Run relevant tests**
4. **Format code**

---

## Git Workflow & Pull Requests

### Branch Naming

Use Linear ticket naming convention:

```bash
# Create branch from Linear ticket
git checkout -b N8N-1234-add-user-management

# Or use the branch name suggested by Linear
```

### Commit Messages

Follow [Conventional Commits](https://www.conventionalcommits.org/):

```bash
# Format: <type>(<scope>): <summary>

git commit -m "feat(editor): Add user management UI"
git commit -m "fix(core): Resolve workflow execution bug"
git commit -m "docs(API): Update authentication documentation"
git commit -m "test(cli): Add user service tests"
```

### Pull Request Title Convention

```
<type>(<scope>): <summary>

Types:
- feat: New feature (appears in changelog)
- fix: Bug fix (appears in changelog)
- perf: Performance improvement (appears in changelog)
- docs: Documentation only
- refactor: Code refactoring
- test: Adding/updating tests
- build: Build system changes
- ci: CI/CD changes
- chore: Maintenance tasks

Scopes:
- API: Public API changes
- core: Backend/private API changes
- editor: Frontend UI changes
- <Node Name>: Specific node changes

Examples:
- feat(editor): Add user role management interface
- fix(core): Resolve memory leak in workflow execution
- perf(API): Optimize user list endpoint
- feat(Slack Node): Add support for file uploads
```

### Pull Request Checklist

- [ ] PR title follows conventions
- [ ] Tests included (unit + integration/E2E)
- [ ] Documentation updated (if needed)
- [ ] Translations added (if UI changes)
- [ ] Typecheck passes (`pnpm typecheck`)
- [ ] Linter passes (`pnpm lint`)
- [ ] Linear ticket linked in PR description
- [ ] PR is small and focused (single feature/fix)

### Creating a Pull Request

```bash
# 1. Ensure you're on latest master
git checkout master
git pull upstream master

# 2. Create feature branch
git checkout -b N8N-1234-feature-name

# 3. Make changes and commit
git add .
git commit -m "feat(editor): Add feature"

# 4. Push to your fork
git push origin N8N-1234-feature-name

# 5. Create PR using GitHub CLI
gh pr create --draft --title "feat(editor): Add feature" --body "
## Summary
Description of changes

## Related Linear tickets, Github issues, and Community forum posts
https://linear.app/n8n/issue/N8N-1234

## Review / Merge checklist
- [x] PR title and summary are descriptive
- [x] Tests included
- [ ] Docs updated
"
```

---

## Common Development Tasks

### 1. Adding a New API Endpoint

**Step 1**: Define types in `packages/@n8n/api-types`:

```typescript
// packages/@n8n/api-types/src/dto/users/create-user.dto.ts
import { z } from 'zod';

export const createUserSchema = z.object({
  email: z.string().email(),
  firstName: z.string().min(1),
  lastName: z.string().min(1),
});

export type CreateUserDto = z.infer<typeof createUserSchema>;
```

**Step 2**: Create controller in `packages/cli/src/controllers`:

```typescript
// packages/cli/src/controllers/users.controller.ts
import { RestController, Post, Body } from '@n8n/decorators';
import { CreateUserDto } from '@n8n/api-types';

@RestController('/users')
export class UsersController {
  constructor(private readonly userService: UserService) {}

  @Post('/')
  async createUser(@Body payload: CreateUserDto) {
    return await this.userService.create(payload);
  }
}
```

**Step 3**: Implement service logic:

```typescript
// packages/cli/src/services/user.service.ts
@Service()
export class UserService {
  async create(data: CreateUserDto): Promise<User> {
    // Implementation
  }
}
```

**Step 4**: Add frontend API call:

```typescript
// packages/editor-ui/src/api/users.ts
import { makeRestApiRequest } from '@/utils/apiUtils';
import type { CreateUserDto } from '@n8n/api-types';

export async function createUser(data: CreateUserDto) {
  return await makeRestApiRequest('POST', '/users', data);
}
```

**Step 5**: Use in component:

```typescript
import { createUser } from '@/api/users';

const handleCreateUser = async () => {
  await createUser({
    email: 'test@example.com',
    firstName: 'John',
    lastName: 'Doe',
  });
};
```

### 2. Adding Internationalization

**Step 1**: Add translations to `packages/@n8n/i18n/src/locales/en.json`:

```json
{
  "users": {
    "create": {
      "title": "Create User",
      "success": "User created successfully",
      "error": "Failed to create user"
    }
  }
}
```

**Step 2**: Use in component:

```vue
<script setup lang="ts">
import { useI18n } from '@/composables/useI18n';

const i18n = useI18n();
const title = i18n.baseText('users.create.title');
</script>

<template>
  <h1>{{ title }}</h1>
</template>
```

### 3. Creating a Reusable Component

**Step 1**: Create in `@n8n/design-system`:

```vue
<!-- packages/@n8n/design-system/src/components/N8nButton/N8nButton.vue -->
<script setup lang="ts">
interface Props {
  type?: 'primary' | 'secondary' | 'danger';
  size?: 'small' | 'medium' | 'large';
  disabled?: boolean;
}

const props = withDefaults(defineProps<Props>(), {
  type: 'primary',
  size: 'medium',
  disabled: false,
});

const emit = defineEmits<{
  click: [event: MouseEvent];
}>();
</script>

<template>
  <button
    :class="['n8n-button', `n8n-button--${type}`, `n8n-button--${size}`]"
    :disabled="disabled"
    @click="emit('click', $event)"
  >
    <slot />
  </button>
</template>

<style lang="scss" scoped>
.n8n-button {
  padding: var(--spacing--sm);
  border-radius: var(--radius);
  font-size: var(--font-size--sm);

  &--primary {
    background: var(--color--primary);
    color: white;
  }

  &--secondary {
    background: var(--color--secondary);
  }

  &--danger {
    background: var(--color--danger);
  }
}
</style>
```

**Step 2**: Export from design system:

```typescript
// packages/@n8n/design-system/src/components/index.ts
export { default as N8nButton } from './N8nButton/N8nButton.vue';
```

**Step 3**: Use in editor-ui:

```vue
<script setup lang="ts">
import { N8nButton } from '@n8n/design-system';
</script>

<template>
  <N8nButton type="primary" @click="handleClick">
    Save
  </N8nButton>
</template>
```

### 4. Working with Workflow Traversal

```typescript
import {
  getParentNodes,
  getChildNodes,
  mapConnectionsByDestination,
} from 'n8n-workflow';

// Finding parent nodes (requires inverted connections)
const connectionsByDestination = mapConnectionsByDestination(workflow.connections);
const parents = getParentNodes(connectionsByDestination, 'NodeName', 'main', 1);

// Finding child nodes (uses connections directly)
const children = getChildNodes(workflow.connections, 'NodeName', 'main', 1);
```

---

## Troubleshooting

### Build Issues

```bash
# Clean build artifacts
pnpm clean

# Rebuild everything
pnpm build > build.log 2>&1
tail -n 50 build.log

# Check for errors
grep -i error build.log
```

### Type Errors

```bash
# Run typecheck in specific package
cd packages/cli
pnpm typecheck

# Build dependencies first if cross-package types changed
cd packages/@n8n/api-types
pnpm build

cd packages/cli
pnpm typecheck
```

### Test Failures

```bash
# Run tests with verbose output
pnpm test --verbose

# Run specific test file
pnpm test UserService.test.ts

# Update snapshots
pnpm test -u
```

### Development Server Issues

```bash
# Clear n8n data
rm -rf ~/.n8n

# Use different data folder
N8N_USER_FOLDER=~/.n8n-dev pnpm dev

# Check ports
lsof -i :5678  # Backend
lsof -i :5173  # Frontend dev server
```

### Hot Reload Not Working

```bash
# Enable hot reload for nodes
N8N_DEV_RELOAD=true pnpm dev

# Restart dev server
# Ctrl+C then pnpm dev
```

---

## Additional Resources

- **Documentation**: https://docs.n8n.io
- **Community Forum**: https://community.n8n.io
- **Contributing Guide**: [CONTRIBUTING.md](./CONTRIBUTING.md)
- **PR Conventions**: [.github/pull_request_title_conventions.md](./.github/pull_request_title_conventions.md)
- **Code of Conduct**: [CODE_OF_CONDUCT.md](./CODE_OF_CONDUCT.md)

---

## Quick Reference

### Essential Commands

```bash
# Development
pnpm dev                    # Full stack
pnpm dev:be                 # Backend only
pnpm dev:fe                 # Frontend only

# Building
pnpm build                  # Build all packages
pnpm typecheck              # Type checking

# Testing
pnpm test                   # All tests
pnpm test:affected          # Changed files only
pnpm lint                   # Lint code
pnpm format                 # Format code

# Package-specific
cd packages/cli && pnpm dev
cd packages/editor-ui && pnpm dev
```

### File Locations

- **API Types**: `packages/@n8n/api-types/src/`
- **Controllers**: `packages/cli/src/controllers/`
- **Services**: `packages/cli/src/services/`
- **Components**: `packages/editor-ui/src/components/`
- **Stores**: `packages/editor-ui/src/stores/`
- **Translations**: `packages/@n8n/i18n/src/locales/`
- **Design System**: `packages/@n8n/design-system/src/components/`

---

**Happy Contributing! 🚀**

For questions, reach out on the [community forum](https://community.n8n.io) or check the [documentation](https://docs.n8n.io).
