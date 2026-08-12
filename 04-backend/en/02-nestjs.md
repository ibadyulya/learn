# NestJS

> 🌐 Russian version: [02-nestjs.md](../ru/02-nestjs.md)

A framework that brings **structure** to Node. Reads as a story; the core idea is
dependency injection, a direct continuation of
[SOLID](../../02-paradigms-and-design/en/03-solid.md) (DIP).

The through-line of the topic:

> **Nest is structure over Node/Express: an app is assembled from modules, and
> dependencies aren't created inside classes but injected by a DI container.
> Everything is built on two things — modules (organization) and providers
> (injectable services). A request passes through a pipeline of interceptors
> (guards → interceptors → pipes → handler), and each layer owns its cross-cutting
> concern.**

---

## Why Nest over Express

Bare Express gives routing and middleware but doesn't dictate structure — on a big
project that sprawls. Nest brings: **modularity**, **TS-first**, a **DI container**,
**decorators**, and unified mechanisms for validation, authorization, error handling.
Under the hood it's the same Express (or Fastify), but with an architectural skeleton.

---

## Modules — units of organization

`@Module` groups the related: controllers, providers, imports of other modules.
```ts
@Module({
  imports: [DatabaseModule],       // what we take from others
  controllers: [OrderController],  // HTTP handlers
  providers: [OrderService],       // injectable services
  exports: [OrderService],         // what we expose outward
})
export class OrderModule {}
```
An app is a tree of modules from the root `AppModule`. A module = a boundary
(encapsulation + high cohesion from
[coupling/cohesion](../../02-paradigms-and-design/en/06-coupling-cohesion-principles.md)):
a provider is visible outside only if explicitly exported.

---

## Providers and DI — the heart of Nest

A **provider** is a class (usually `@Injectable()`) that can be **injected**. Instead
of `new`, dependencies are declared in the constructor, and the container supplies
them:
```ts
@Injectable()
class OrderService {
  constructor(private repo: OrderRepository) {}   // injected by the container
}
```
This is direct **DIP**: `OrderService` depends on an abstraction rather than creating
the concretion itself. The container (IoC) builds the dependency graph and passes
instances. The benefit: swappability (a mock in tests), a unified lifecycle, loose
coupling.

By default providers are **singletons** (one instance per app); there's `REQUEST`
scope (new per request) and `TRANSIENT` — but singleton scope is the most common and
cheapest.

---

## Controllers and routing

`@Controller` + method decorators receive HTTP and **delegate** to services (the
Controller role from [GRASP](../../02-paradigms-and-design/en/04-grasp.md)):
```ts
@Controller('orders')
class OrderController {
  constructor(private orders: OrderService) {}
  @Get(':id') get(@Param('id') id: string) { return this.orders.find(id); }
  @Post()     create(@Body() dto: CreateOrderDto) { return this.orders.create(dto); }
}
```
The controller is thin: it parsed the request, called the service, returned the
result; no business logic in it.

---

## The request lifecycle — the pipeline

A request passes through layers in order, each a cross-cutting concern:
```
Middleware → Guards → Interceptors(pre) → Pipes → HANDLER → Interceptors(post) → Exception filters
```
- **Middleware** — the earliest layer (as in Express): logging, cors.
- **Guards** — "let it through or not": **authorization**/authentication
  (`canActivate` → true/false).
- **Interceptors** — wrap the call (the decorator pattern): timing logs, response
  transformation, cache — before and after the handler.
- **Pipes** — **validation and transformation** of input (`ValidationPipe` +
  class-validator checks the DTO; see
  [validation](./05-validation-error-handling.md)).
- **Handler** — the controller method.
- **Exception filters** — catch thrown exceptions and shape the HTTP error response.

Splitting into layers = clean roles: a guard doesn't validate, a pipe doesn't
authorize.

---

## Interview phrasing

> "Nest is a structural framework over Express/Fastify: an app of modules,
> dependencies injected by a DI container (direct DIP — classes depend on
> abstractions, the container supplies providers; singletons by default). A module is
> a boundary with imports/providers/exports. A controller is thin: received the
> request, delegated to a service. A request goes through the pipeline: middleware →
> guards (authorization) → interceptors (cross-cutting around the call) → pipes (DTO
> validation/transformation) → handler → exception filters (errors into an HTTP
> response). Each layer is its own cross-cutting concern, giving high cohesion and
> testability."

---

See [02-nestjs.tasks.md](./02-nestjs.tasks.md) — tasks. Solutions in
[02-nestjs.answers.md](./02-nestjs.answers.md).
