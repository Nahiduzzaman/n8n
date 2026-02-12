# n8n Developer Quick Start Guide

> **Fast-track guide to start contributing to n8n**

This is a condensed version of the full [Developer Contribution Guide](./DEVELOPER_CONTRIBUTION_GUIDE.md). Use this for quick reference and common tasks.

---

## 🚀 Quick Setup (5 minutes)

```bash
# 1. Prerequisites
corepack enable
corepack prepare pnpm@10.22.0 --activate

# 2. Clone & Install
git clone https://github.com/n8n-io/n8n.git
cd n8n
pnpm install

# 3. Build
pnpm build > build.log 2>&1

# 4. Start Development
# Terminal 1: Backend
cd packages/cli && pnpm dev

# Terminal 2: Frontend
cd packages/editor-ui && pnpm dev
```

Access at: http://localhost:5678

---

## 📁 Project Structure

```
n8n/
├── packages/
│   ├── @n8n/api-types/      # 🔗 Shared types (Frontend ↔ Backend)
│   ├── @n8n/design-system/  # 🎨 Vue component library
│   ├── @n8n/i18n/          # 🌍 Translations
│   ├── cli/                 # 🔧 Backend (Express + TypeORM)
│   ├── editor-ui/           # 💻 Frontend (Vue 3 + Pinia)
│   ├── nodes-base/          # 🔌 Built-in nodes
│   └── workflow/            # ⚙️ Core workflow engine
```

---

## 🎯 Common Tasks

### Add New API Endpoint

**1. Define types** (`packages/@n8n/api-types/src/dto/`):
```typescript
export const createUserSchema = z.object({
  email: z.string().email(),
  firstName: z.string(),
});
export type CreateUserDto = z.infer<typeof createUserSchema>;
```

**2. Create controller** (`packages/cli/src/controllers/`):
```typescript
@RestController('/users')
export class UsersController {
  @Post('/')
  async createUser(@Body payload: CreateUserDto) {
    return await this.userService.create(payload);
  }
}
```

**3. Add frontend API** (`packages/editor-ui/src/api/`):
```typescript
export async function createUser(data: CreateUserDto) {
  return await makeRestApiRequest('POST', '/users', data);
}
```

### Add Translation

**1. Add to** `packages/@n8n/i18n/src/locales/en.json`:
```json
{
  "users": {
    "create": "Create User",
    "success": "User created successfully"
  }
}
```

**2. Use in component**:
```typescript
const i18n = useI18n();
const title = i18n.baseText('users.create');
```

### Create Vue Component

```vue
<script setup lang="ts">
interface Props {
  title: string;
}
const props = defineProps<Props>();
const emit = defineEmits<{ save: [value: string] }>();
</script>

<template>
  <div class="wrapper">
    <h2>{{ title }}</h2>
  </div>
</template>

<style scoped>
.wrapper {
  padding: var(--spacing--md);
  color: var(--color--text);
}
</style>
```

### Write Unit Test

```typescript
import { mock } from 'jest-mock-extended';

describe('UserService', () => {
  let service: UserService;
  let mockRepo: jest.Mocked<UserRepository>;

  beforeEach(() => {
    mockRepo = mock<UserRepository>();
    service = new UserService(mockRepo);
  });

  it('should create user', async () => {
    mockRepo.save.mockResolvedValue({ id: '1', email: 'test@test.com' });
    
    const result = await service.create({ email: 'test@test.com' });
    
    expect(result.id).toBe('1');
  });
});
```

---

## 🎨 Frontend Patterns

### CSS Variables (Always Use!)
```scss
// ✅ CORRECT
.component {
  padding: var(--spacing--md);
  color: var(--color--text);
  font-size: var(--font-size--sm);
}

// ❌ WRONG
.component {
  padding: 20px;
  color: #333;
}
```

### Pinia Store
```typescript
export const useUsersStore = defineStore('users', () => {
  const users = ref<User[]>([]);
  
  const allUsers = computed(() => users.value);
  
  async function fetchUsers() {
    users.value = await usersApi.getUsers();
  }
  
  return { users, allUsers, fetchUsers };
});
```

---

## 🔧 Backend Patterns

### Controller → Service → Repository

```typescript
// Controller (HTTP layer)
@RestController('/users')
export class UsersController {
  constructor(private userService: UserService) {}
  
  @Get('/:id')
  async getUser(@Param('id') id: string) {
    return await this.userService.findById(id);
  }
}

// Service (Business logic)
@Service()
export class UserService {
  constructor(private userRepo: UserRepository) {}
  
  async findById(id: string) {
    return await this.userRepo.findOneBy({ id });
  }
}

// Repository (Data access)
@Service()
export class UserRepository extends Repository<User> {
  async findByEmail(email: string) {
    return await this.findOne({ where: { email } });
  }
}
```

### Error Handling
```typescript
// ✅ CORRECT
throw new BadRequestError('Invalid input');
throw new NotFoundError('User not found');
throw new UnexpectedError('Unexpected error');

// ❌ WRONG (deprecated)
throw new ApplicationError('Error');
```

---

## ✅ Code Quality Checklist

### TypeScript
- [ ] No `any` types (use `unknown` or proper types)
- [ ] No `as` casting (use type guards)
- [ ] Proper interfaces defined

### Frontend
- [ ] All text uses i18n
- [ ] CSS variables used (no hardcoded values)
- [ ] `data-testid` is single value (no spaces)
- [ ] Components in design-system if reusable

### Backend
- [ ] DTOs defined in `@n8n/api-types`
- [ ] Proper error classes used
- [ ] Services use dependency injection
- [ ] Controllers use decorators

### Testing
- [ ] Unit tests written
- [ ] External dependencies mocked
- [ ] Tests pass: `pnpm test`
- [ ] Typecheck passes: `pnpm typecheck`

---

## 🧪 Testing Commands

```bash
# Backend tests
cd packages/cli && pnpm test

# Frontend tests
cd packages/editor-ui && pnpm test

# E2E tests
pnpm --filter=n8n-playwright test:local

# Specific test
pnpm test UserService.test.ts

# With coverage
COVERAGE_ENABLED=true pnpm test
```

---

## 🔍 Debugging

### Backend
```bash
# Clean database
rm -rf ~/.n8n

# Use different database
N8N_USER_FOLDER=~/.n8n-dev pnpm dev

# Enable hot reload
N8N_DEV_RELOAD=true pnpm dev
```

### Frontend
```bash
# Check dev server
lsof -i :5173

# Restart dev server
cd packages/editor-ui
pnpm dev
```

### Build Issues
```bash
# Clean and rebuild
pnpm clean
pnpm build > build.log 2>&1
tail -n 50 build.log
```

---

## 📝 Git Workflow

### Branch Naming
```bash
git checkout -b N8N-1234-feature-name
```

### Commit Format
```bash
git commit -m "feat(editor): Add user management"
git commit -m "fix(core): Resolve memory leak"
git commit -m "docs: Update API documentation"
```

### PR Title Format
```
feat(editor): Add user role management
fix(API): Resolve authentication issue
perf(core): Optimize workflow execution
```

### Create PR
```bash
gh pr create --draft --title "feat(editor): Add feature" --body "
## Summary
Description of changes

## Related Linear tickets
https://linear.app/n8n/issue/N8N-1234

## Checklist
- [x] Tests included
- [x] Translations added
- [x] Typecheck passes
"
```

---

## 🛠️ Essential Commands

```bash
# Development
pnpm dev                    # Full stack (resource-intensive)
pnpm dev:be                 # Backend only
pnpm dev:fe                 # Frontend only
pnpm dev:ai                 # AI nodes only

# Building
pnpm build                  # Build all
pnpm typecheck              # Type check

# Testing
pnpm test                   # All tests
pnpm test:affected          # Changed files
pnpm lint                   # Lint
pnpm lint:fix               # Fix linting
pnpm format                 # Format code

# Specific packages
cd packages/cli && pnpm dev
cd packages/editor-ui && pnpm dev
cd packages/nodes-base && pnpm dev
```

---

## 📚 Key Files to Know

| File | Purpose |
|------|---------|
| `AGENTS.md` | AI agent development guidelines |
| `CONTRIBUTING.md` | Full contribution guide |
| `packages/@n8n/api-types/` | Shared TypeScript types |
| `packages/cli/src/controllers/` | API endpoints |
| `packages/cli/src/services/` | Business logic |
| `packages/editor-ui/src/stores/` | State management |
| `packages/@n8n/i18n/src/locales/` | Translations |

---

## 🚨 Common Pitfalls

### ❌ Don't
- Use `any` type
- Hardcode CSS values (use variables)
- Use `ApplicationError` (deprecated)
- Create large PRs (keep them focused)
- Skip tests
- Forget i18n for UI text

### ✅ Do
- Use proper TypeScript types
- Use CSS variables
- Use `UnexpectedError`, `OperationalError`, `UserError`
- Keep PRs small and focused
- Write tests for all changes
- Add translations for all UI text

---

## 💡 Pro Tips

1. **Use selective development** - Don't run full `pnpm dev` if working on specific packages
2. **Build dependencies first** - If changing `@n8n/api-types`, build it before other packages
3. **Check linter locally** - Run `pnpm lint` before pushing
4. **Use hot reload wisely** - `N8N_DEV_RELOAD=true` adds overhead
5. **Mock external APIs** - Always use `nock` in tests
6. **Read existing code** - Check similar features for patterns

---

## 🆘 Getting Help

- **Documentation**: https://docs.n8n.io
- **Community Forum**: https://community.n8n.io
- **Full Guide**: [DEVELOPER_CONTRIBUTION_GUIDE.md](./DEVELOPER_CONTRIBUTION_GUIDE.md)
- **Node Development**: `packages/nodes-base/AGENTS.md`
- **Frontend Patterns**: `packages/frontend/AGENTS.md`

---

## 📖 Next Steps

1. ✅ Set up development environment
2. 📖 Read [DEVELOPER_CONTRIBUTION_GUIDE.md](./DEVELOPER_CONTRIBUTION_GUIDE.md)
3. 🔍 Explore codebase
4. 🎯 Pick a task from Linear
5. 💻 Start coding!
6. ✅ Write tests
7. 📝 Create PR

---

**Ready to contribute? Start with a small bug fix or documentation improvement to get familiar with the workflow!** 🚀
