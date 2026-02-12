# n8n Architecture & Contribution Flow

> **Visual guide to n8n's architecture and development workflow**

## System Architecture

```mermaid
graph TB
    subgraph "Frontend Layer"
        UI[Vue 3 Editor UI<br/>packages/editor-ui]
        DS[Design System<br/>@n8n/design-system]
        I18N[i18n Translations<br/>@n8n/i18n]
        STORES[Pinia Stores<br/>State Management]
    end

    subgraph "API Layer"
        TYPES[@n8n/api-types<br/>Shared TypeScript Interfaces]
    end

    subgraph "Backend Layer"
        CTRL[Controllers<br/>HTTP Endpoints]
        SVC[Services<br/>Business Logic]
        REPO[Repositories<br/>Data Access]
        DB[(Database<br/>SQLite/PostgreSQL)]
    end

    subgraph "Core Engine"
        WF[Workflow Engine<br/>packages/workflow]
        CORE[Execution Core<br/>packages/core]
        NODES[Nodes<br/>packages/nodes-base]
    end

    UI --> DS
    UI --> I18N
    UI --> STORES
    UI --> TYPES
    TYPES --> CTRL
    CTRL --> SVC
    SVC --> REPO
    REPO --> DB
    SVC --> CORE
    CORE --> WF
    CORE --> NODES

    style UI fill:#42b883
    style DS fill:#42b883
    style TYPES fill:#ffd700
    style CTRL fill:#68a063
    style SVC fill:#68a063
    style REPO fill:#68a063
    style WF fill:#ff6b6b
    style CORE fill:#ff6b6b
```

## Request Flow

```mermaid
sequenceDiagram
    participant User
    participant Vue Component
    participant Pinia Store
    participant API Client
    participant Controller
    participant Service
    participant Repository
    participant Database

    User->>Vue Component: Interact with UI
    Vue Component->>Pinia Store: Dispatch Action
    Pinia Store->>API Client: HTTP Request
    API Client->>Controller: POST /api/users
    Controller->>Service: createUser(dto)
    Service->>Repository: save(user)
    Repository->>Database: INSERT
    Database-->>Repository: User Entity
    Repository-->>Service: User
    Service-->>Controller: User
    Controller-->>API Client: 200 OK + User
    API Client-->>Pinia Store: Response Data
    Pinia Store-->>Vue Component: Update State
    Vue Component-->>User: UI Update
```

## Component Architecture

```mermaid
graph LR
    subgraph "Vue Component"
        PROPS[Props]
        STATE[Reactive State]
        COMPUTED[Computed Properties]
        METHODS[Methods]
        EMIT[Emits]
    end

    subgraph "Composables"
        I18N_COMP[useI18n]
        TOAST[useToast]
        STORE[useStore]
    end

    PROPS --> COMPUTED
    STATE --> COMPUTED
    COMPUTED --> TEMPLATE[Template]
    METHODS --> EMIT
    I18N_COMP --> METHODS
    TOAST --> METHODS
    STORE --> STATE

    style PROPS fill:#e3f2fd
    style STATE fill:#fff3e0
    style COMPUTED fill:#f3e5f5
    style METHODS fill:#e8f5e9
```

## Backend Module Structure

```mermaid
graph TD
    subgraph "Controller Layer"
        C1[UsersController<br/>@RestController]
        C2[WorkflowsController<br/>@RestController]
    end

    subgraph "Service Layer"
        S1[UserService<br/>@Service]
        S2[WorkflowService<br/>@Service]
        S3[EventService<br/>@Service]
    end

    subgraph "Repository Layer"
        R1[UserRepository<br/>extends Repository]
        R2[WorkflowRepository<br/>extends Repository]
    end

    subgraph "Database"
        E1[User Entity]
        E2[Workflow Entity]
    end

    C1 --> S1
    C2 --> S2
    S1 --> R1
    S2 --> R2
    S1 --> S3
    S2 --> S3
    R1 --> E1
    R2 --> E2

    style C1 fill:#4caf50
    style C2 fill:#4caf50
    style S1 fill:#2196f3
    style S2 fill:#2196f3
    style S3 fill:#2196f3
    style R1 fill:#ff9800
    style R2 fill:#ff9800
```

## Development Workflow

```mermaid
graph TB
    START[Start Development] --> BRANCH[Create Branch<br/>N8N-1234-feature]
    BRANCH --> DECIDE{What to Build?}
    
    DECIDE -->|API Endpoint| API_FLOW
    DECIDE -->|UI Component| UI_FLOW
    DECIDE -->|Full Feature| FULL_FLOW
    
    subgraph "API Development"
        API_FLOW[1. Define Types<br/>@n8n/api-types]
        API_FLOW --> API_CTRL[2. Create Controller<br/>packages/cli/controllers]
        API_CTRL --> API_SVC[3. Implement Service<br/>packages/cli/services]
        API_SVC --> API_TEST[4. Write Tests]
    end
    
    subgraph "UI Development"
        UI_FLOW[1. Create Component<br/>packages/editor-ui]
        UI_FLOW --> UI_I18N[2. Add Translations<br/>@n8n/i18n]
        UI_I18N --> UI_STORE[3. Update Store<br/>if needed]
        UI_STORE --> UI_TEST[4. Write Tests]
    end
    
    subgraph "Full Feature"
        FULL_FLOW[1. Define API Types]
        FULL_FLOW --> FULL_BE[2. Backend Implementation]
        FULL_BE --> FULL_FE[3. Frontend Implementation]
        FULL_FE --> FULL_TEST[4. E2E Tests]
    end
    
    API_TEST --> QUALITY
    UI_TEST --> QUALITY
    FULL_TEST --> QUALITY
    
    QUALITY[Code Quality Checks]
    QUALITY --> LINT[pnpm lint]
    QUALITY --> TYPE[pnpm typecheck]
    QUALITY --> TEST[pnpm test]
    
    TEST --> PR[Create Pull Request]
    PR --> REVIEW[Code Review]
    REVIEW --> MERGE[Merge to Master]

    style START fill:#4caf50
    style QUALITY fill:#ff9800
    style PR fill:#2196f3
    style MERGE fill:#4caf50
```

## Testing Strategy

```mermaid
graph TB
    subgraph "Unit Tests"
        UT1[Controller Tests<br/>Mock Services]
        UT2[Service Tests<br/>Mock Repositories]
        UT3[Component Tests<br/>Mock Stores]
    end

    subgraph "Integration Tests"
        IT1[API Tests<br/>Real Database]
        IT2[Store Tests<br/>Real API Calls]
    end

    subgraph "E2E Tests"
        E2E1[Playwright Tests<br/>Full User Flows]
    end

    UT1 --> IT1
    UT2 --> IT1
    UT3 --> IT2
    IT1 --> E2E1
    IT2 --> E2E1

    style UT1 fill:#e3f2fd
    style UT2 fill:#e3f2fd
    style UT3 fill:#e3f2fd
    style IT1 fill:#fff3e0
    style IT2 fill:#fff3e0
    style E2E1 fill:#f3e5f5
```

## Monorepo Package Dependencies

```mermaid
graph TD
    API_TYPES[@n8n/api-types]
    WORKFLOW[n8n-workflow]
    CORE[n8n-core]
    CLI[n8n CLI]
    EDITOR[editor-ui]
    DESIGN[design-system]
    I18N[@n8n/i18n]
    NODES[nodes-base]
    DI[@n8n/di]
    CONFIG[@n8n/config]

    API_TYPES --> CLI
    API_TYPES --> EDITOR
    WORKFLOW --> CORE
    WORKFLOW --> NODES
    CORE --> CLI
    NODES --> CLI
    DESIGN --> EDITOR
    I18N --> EDITOR
    DI --> CLI
    CONFIG --> CLI

    style API_TYPES fill:#ffd700
    style CLI fill:#68a063
    style EDITOR fill:#42b883
    style DESIGN fill:#42b883
    style CORE fill:#ff6b6b
```

## Data Flow: Creating a User

```mermaid
sequenceDiagram
    autonumber
    participant FE as Frontend Component
    participant Store as Pinia Store
    participant API as API Client
    participant Ctrl as UsersController
    participant Svc as UserService
    participant Repo as UserRepository
    participant DB as Database
    participant Event as EventService

    FE->>Store: dispatch('createUser', data)
    Store->>API: POST /api/users
    API->>Ctrl: @Post('/') createUser(dto)
    Ctrl->>Svc: create(dto)
    Svc->>Repo: create(user)
    Repo->>Repo: validate entity
    Repo->>DB: INSERT INTO users
    DB-->>Repo: User record
    Repo-->>Svc: User entity
    Svc->>Event: emit('user-created')
    Svc-->>Ctrl: User
    Ctrl-->>API: 200 OK + User
    API-->>Store: Response data
    Store->>Store: Update state
    Store-->>FE: User created
    FE->>FE: Show success toast
```

## Node Development Flow

```mermaid
graph TB
    START[Create Node] --> DESC[Define Node Description<br/>INodeTypeDescription]
    DESC --> PROPS[Define Properties<br/>Parameters & Options]
    PROPS --> DECIDE{Node Type?}
    
    DECIDE -->|Programmatic| EXEC[Implement execute()<br/>Custom Logic]
    DECIDE -->|Declarative| ROUTING[Define Routing<br/>requestDefaults]
    DECIDE -->|Trigger| TRIGGER[Implement trigger()<br/>or webhook()]
    
    EXEC --> CREDS[Add Credentials<br/>if needed]
    ROUTING --> CREDS
    TRIGGER --> CREDS
    
    CREDS --> TEST[Write Tests<br/>Unit + Workflow]
    TEST --> ICON[Add Icon SVG]
    ICON --> DOCS[Add Documentation]
    DOCS --> DONE[Node Complete]

    style START fill:#4caf50
    style DONE fill:#4caf50
    style TEST fill:#ff9800
```

## CI/CD Pipeline

```mermaid
graph LR
    PUSH[Push to Branch] --> LINT[Linting<br/>ESLint + Biome]
    LINT --> TYPE[Type Check<br/>TypeScript]
    TYPE --> UNIT[Unit Tests<br/>Jest]
    UNIT --> E2E[E2E Tests<br/>Playwright]
    E2E --> BUILD[Build<br/>All Packages]
    BUILD --> DEPLOY{Deploy?}
    
    DEPLOY -->|PR| PREVIEW[Preview Environment]
    DEPLOY -->|Master| PROD[Production]

    style PUSH fill:#2196f3
    style LINT fill:#ff9800
    style TYPE fill:#ff9800
    style UNIT fill:#ff9800
    style E2E fill:#ff9800
    style BUILD fill:#4caf50
    style PROD fill:#4caf50
```

## State Management Flow (Pinia)

```mermaid
graph TB
    COMP[Vue Component] -->|dispatch| ACTION[Store Action]
    ACTION -->|async call| API[API Request]
    API -->|response| ACTION
    ACTION -->|commit| STATE[Store State]
    STATE -->|computed| GETTER[Store Getter]
    GETTER -->|reactive| COMP

    style COMP fill:#42b883
    style ACTION fill:#ffd700
    style STATE fill:#ff6b6b
    style GETTER fill:#4caf50
```

## Error Handling Flow

```mermaid
graph TB
    ERROR[Error Occurs] --> TYPE{Error Type?}
    
    TYPE -->|User Input| USER_ERROR[UserError<br/>400 Bad Request]
    TYPE -->|Not Found| NOT_FOUND[NotFoundError<br/>404 Not Found]
    TYPE -->|Permissions| FORBIDDEN[ForbiddenError<br/>403 Forbidden]
    TYPE -->|Expected| OPERATIONAL[OperationalError<br/>500 Internal]
    TYPE -->|Unexpected| UNEXPECTED[UnexpectedError<br/>500 Internal]
    
    USER_ERROR --> LOG[Log Error]
    NOT_FOUND --> LOG
    FORBIDDEN --> LOG
    OPERATIONAL --> LOG
    UNEXPECTED --> LOG
    
    LOG --> RESPONSE[Send Error Response]
    RESPONSE --> CLIENT[Client Handles Error]
    CLIENT --> TOAST[Show Toast Message]

    style USER_ERROR fill:#ff9800
    style NOT_FOUND fill:#ff9800
    style FORBIDDEN fill:#f44336
    style OPERATIONAL fill:#ff9800
    style UNEXPECTED fill:#f44336
```

---

## Key Takeaways

### Frontend Development
1. **Components** → Use Vue 3 Composition API
2. **State** → Manage with Pinia stores
3. **Styling** → Always use CSS variables
4. **Text** → All UI text must use i18n
5. **Reusability** → Pure components go in design-system

### Backend Development
1. **Structure** → Controller → Service → Repository
2. **DI** → Use `@Service()` and constructor injection
3. **Types** → Define DTOs in `@n8n/api-types`
4. **Errors** → Use proper error classes (not ApplicationError)
5. **Events** → Use EventService for decoupled communication

### Testing
1. **Unit** → Mock all dependencies
2. **Integration** → Test with real database
3. **E2E** → Test full user flows with Playwright
4. **Coverage** → Aim for high coverage on critical paths

### Code Quality
1. **TypeScript** → No `any`, no `as` casting
2. **Linting** → Run before committing
3. **Formatting** → Use Biome
4. **Tests** → Required for all changes

---

For detailed implementation examples, see [DEVELOPER_CONTRIBUTION_GUIDE.md](./DEVELOPER_CONTRIBUTION_GUIDE.md)
