# IplEcommerce — One?Page Architecture

```mermaid
flowchart LR
  Client[Client\n(Web / Mobile / Postman)] -->|HTTP JSON| API[API\n`IplEcommerce.API` (ASP.NET Core)]
  API --> Controllers[Controllers]
  Controllers -->|uses| UnitOfWork[`UnitOfWork` / `IUnitOfWork`]
  UnitOfWork -->|coordinates| Repos[Repositories\n(`GenericRepository`, `ProductRepository`, `CartRepository`, `OrderRepository`)]
  Repos -->|EF Core| DbContext[`IplEcommerceDbContext` (EF Core)]
  DbContext -->|SQL| SQL[(SQL Server)]

  Domain[Domain Layer\n(Entities, Enums, Interfaces)] --- Repos
  Application[Application Layer\n(DTOs)] --- API
  Infrastructure[Infrastructure\n(EF, Repos, Migrations, Seeder)] --- DbContext
  API -.->|optional| Mediator[`MediatR`]
```

Key components (file pointers):

- Presentation (API): `src/Presentation/IplEcommerce.API`
  - Controllers: `OrdersController.cs`, `ProductsController.cs`, `CartController.cs`
  - Program/startup: `Program.cs`

- Application (DTOs): `src/Core/IplEcommerce.Application`
  - DTO definitions: `ApplicationDtos.cs`

- Domain: `src/Core/IplEcommerce.Domain`
  - Entities, enums and repository interfaces: `Entities/`, `Interfaces/`

- Infrastructure: `src/Infrastructure/IplEcommerce.Infrastructure`
  - EF DbContext: `Data/IplEcommerceDbContext.cs`
  - Repositories: `Repositories/GenericRepository.cs`, `ProductRepository.cs`, `OrderRepository.cs`, `CartRepository.cs`
  - Unit of Work: `UnitOfWork/UnitOfWork.cs`
  - Migrations & Seeder: `Migrations/`, `Data/DatabaseSeeder.cs`

Data flow summary:
- Client ? API controllers ? UnitOfWork ? repository calls ? EF Core (`IplEcommerceDbContext`) ? SQL Server.
- DTOs are used by controllers to return API payloads; domain entities live in Domain layer and are persisted by EF Core in Infrastructure.

Important design points & notes:
- Patterns used: Repository (generic + specific), Unit of Work, DTOs, Dependency Injection, (optionally) Mediator.
- Concurrency: stock decrement is implemented using an atomic SQL UPDATE in `ProductRepository.TryDecrementStockAsync` to avoid oversell.
- Performance considerations: prefer DTO projection (`Select` / `ProjectTo`) instead of materializing full entities; use `.AsNoTracking()` for read-only queries; batch `SaveChanges()` calls.

Suggested next improvements (short):
- Add endpoint idempotency for `CreateOrder` (Idempotency-Key) to prevent duplicate orders on retries.
- Add caching for read-heavy data (franchises, product metadata) via `IMemoryCache` or Redis.
- Introduce `RowVersion` for optimistic concurrency + retry as fallback.
- Add automated integration test for concurrent order creation.

(End of one?page architecture)
