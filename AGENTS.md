<Context>
    <Philosophy>
        This project is not merely an application; it is a rigid academic framework designed to enforce software engineering mastery. 
        It rejects "easy" solutions in favor of "correct" solutions. 
        The primary objective is to internalize SOLID principles, strict OOP, and modular Clean Architecture.
        
        We prioritize:
        1. Maintainability over Speed.
        2. Explicitness over "Magic".
        3. Interfaces over Implementation.
    </Philosophy>
    <Scope>
        A full-stack enterprise application orchestrating a Python FastAPI backend and an Angular (Modern) frontend. 
        The system mimics a high-scale environment where code readability, testability, and decoupling are paramount.
    </Scope>
    <Actors>
        - The Architect (You): Defines the contracts (Interfaces/ABCs).
        - The Builder (You): Implements the logic behind the contracts.
        - The Consumer (You): Consumes the logic via strictly typed APIs and ViewModels.
    </Actors>
</Context>

<Architecture>
    <Backend_Pattern>
        CLEAN ARCHITECTURE (Onion Architecture)
        
        [ External Request ]
               ⬇
        [ Interface Layer ] -> (Routers, CLI)
               ⬇
        [ Application Layer ] -> (Use Cases, Services, DTOs)
               ⬇
        [ Domain Layer ] -> (Entities, Interface Definitions/ABCs)
               ⬆
        [ Infrastructure Layer ] -> (DB Implementations, 3rd Party APIs)
    </Backend_Pattern>

    <Frontend_Pattern>
        MVVM (Model-View-ViewModel) + SIGNAL ARCHITECTURE
        
        [ View (Template) ]
               ⬇ (Triggers Action)
        [ ViewModel (Component) ] -> Holds State (Signals), Handles UI Logic
               ⬇ (Calls)
        [ Model (Service/Repository) ] -> Handles HTTP, Data Transformation
               ⬇ (Returns Observable/Signal)
        [ View (Template) ] -> Updates via Signal bindings (No manual change detection)
    </Frontend_Pattern>
</Architecture>

<Requirements>
    <Backend_Tech_Stack>
        - Language: Python 3.11+
        - Framework: FastAPI (strictly for routing only)
        - Validation: Pydantic V2 (Strict Mode)
        - DI Framework: Native FastAPI Depends or specialized container (e.g., Dependency Injector)
        - Logging: Structlog (JSON format, context-aware)
        - Testing: Pytest (100% coverage on Domain/Application layers)
    </Backend_Tech_Stack>

    <Frontend_Tech_Stack>
        - Framework: Angular v21+ (Standalone Components, Signals)
        - Styling: SCSS (Strict BEM or Modular), Design Tokens
        - State: Signals (Native) + RxJS (for complex streams only)
        - HTTP: HttpClient with Interceptors for Logging/Auth
    </Frontend_Tech_Stack>

    <Database_Infrastructure>
        - Database: PostgreSQL
        - Migration: Flyway (Java-based CLI)
        - Strategy: SQL-First. No ORM Auto-generation.
        - Driver: AsyncPG / SQLAlchemy (Async)
    </Database_Infrastructure>
</Requirements>

<Constraints>
    <DO>
        <Universal>
            - DO ensure every single class has a Single Responsibility (SRP).
            - DO document every public method with Docstrings (Python) or JSDoc (TS).
        </Universal>

        <Backend>
            - DO define an Abstract Base Class (ABC) for *every* Repository.
            - DO inject dependencies via `__init__` in Services.
            - DO map all Database Exceptions to Domain Exceptions before they reach the Router.
            - DO use `UPPER_CASE` for global constants and configuration.
            - DO implement a global Exception Handler that returns standardized JSON error responses.
        </Backend>

        <Frontend>
            - DO create a "Design System" folder first (`src/app/theme`) containing:
                - `colors.ts` (CONSTANTS only)
                - `typography.scss` (Mixins)
                - `icons.registry.ts` (SVG definitions)
            - DO use Angular Signals (`computed`, `effect`) for all derived state.
            - DO decouple Components from API logic entirely (The Component should not know what HTTP is).
            - DO use "Smart" (Container) vs "Dumb" (Presentational) component patterns.
        </Frontend>
    </DO>

    <DO_NOT>
        <Backend>
            - DO NOT put business logic in `api/routes`. Routers are for parsing Request -> DTO only.
            - DO NOT expose ORM Entities (SQLAlchemy Models) to the outside world. Always map to Pydantic Response DTOs.
            - DO NOT use global variables for state.
            - DO NOT import `Infrastructure` modules into `Domain` modules. (Circular Dependency / Architectural Violation).
        </Backend>

        <Frontend>
            - DO NOT write logic inside the HTML Template (e.g., `{{ data.filter(...) }}`). compute it in the ViewModel.
            - DO NOT use `any`. Ever. Use `unknown` if strictly necessary, but prefer defined Interfaces.
            - DO NOT hardcode colors (e.g., `#FFFFFF`) in Component SCSS. Use CSS Variables/Design Tokens.
            - DO NOT import from `app.module` (Standalone components are mandatory).
        </Frontend>

        <Database>
             - DO NOT modify the database schema manually. If it's not in a `V__` Flyway script, it doesn't exist.
        </Database>
    </DO_NOT>

    <SUCCESS_METRICS>
        1. "The Swap Test": Can you swap the PostgreSQL implementation with a MongoDB implementation by changing ONLY the Infrastructure layer and DI config?
        2. "The Junior Test": Can a junior developer read the `Domain` folder and understand exactly what the business does without seeing a single line of SQL or HTTP code?
        3. "The Reskin Test": Can you change the entire color scheme of the app by modifying only `variables.scss`?
    </SUCCESS_METRICS>
</Constraints>

<Coding_Styles>
    <Python>
        <Naming>
            - Classes: `PascalCase` (e.g., `UserProfileService`)
            - Variables/Functions: `snake_case` (e.g., `get_user_by_id`)
            - Interfaces: Prefixed with I (e.g., `IUserRepository`) - *Optional but recommended for clarity in this strict environment.*
            - Private members: `_prefixed` (e.g., `_validate_email`)
        </Naming>
        <Formatting>
            - Tools: `Ruff` (Linter), `Black` (Formatter), `MyPy` (Type Checker).
            - Rules: Line length 88-100. Double quotes.
        </Formatting>
        <Example_Good>
            ```python
            class UserService:
                def __init__(self, user_repo: IUserRepository):
                    self._user_repo = user_repo

                async def create(self, cmd: CreateUserCommand) -> UserDto:
                    if not cmd.is_valid:
                        raise DomainValidationError("Invalid command")
                    return await self._user_repo.save(cmd)
            ```
        </Example_Good>
    </Python>

    <Angular_TypeScript>
        <Naming>
            - Files: `kebab-case` (e.g., `user-profile.component.ts`)
            - Classes: `PascalCase` (e.g., `UserProfileComponent`)
            - Observables: Suffix with `$` (e.g., `users$`)
            - Signals: No specific suffix, but prefer noun phrases (e.g., `currentUser`)
        </Naming>
        <Formatting>
            - Tools: `Prettier`, `ESLint`.
            - Rules: Single quotes. 2 Space indentation.
        </Formatting>
        <Example_Good>
            ```typescript
            @Component({ ... })
            export class UserListComponent {
                // Dependency Injection
                private userService = inject(UserService);
                
                // State (Signal)
                users = this.userService.getUsers(); 
                
                // Computed State
                userCount = computed(() => this.users().length);
            }
            ```
        </Example_Good>
    </Angular_TypeScript>
    
    <SQL>
        <Naming>
            - Tables: `snake_case`, plural (e.g., `users`, `order_items`)
            - Keys: `pk_table_name`, `fk_source_target`
        </Naming>
        <Keywords>
            - ALWAYS UPPERCASE (e.g., `SELECT * FROM users WHERE id = 1;`)
        </Keywords>
    </SQL>
</Coding_Styles>

<Workflow>
    <New_Feature_Checklist>
        1. **Database:** Write `V{N}__description.sql` in Flyway folder. Run migration.
        2. **Domain:** Define the Data Models (Entities) and Repository Interfaces (ABCs).
        3. **Application:** Create the Service class implementing the business logic. Define Pydantic DTOs for input/output.
        4. **Infrastructure:** Implement the Repository Interface using SQL/ORM.
        5. **Interface (API):** Create the Router, wire the Service (DI), and map Request -> DTO -> Service.
        6. **Frontend (Model):** specific Interface matching the API Response DTO.
        7. **Frontend (Service):** Add method to Data Service.
        8. **Frontend (View/ViewModel):** Build Component using Signals.
    </New_Feature_Checklist>
</Workflow>