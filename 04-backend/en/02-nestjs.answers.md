# NestJS — solutions

> 🌐 Russian version: [02-nestjs.answers.md](../ru/02-nestjs.answers.md)

---

## Question 1
Nest gives: **modularity** (structure from `@Module`), a **DI container** (injection
instead of `new`), **TS-first + decorators**, unified mechanisms for **validation**
(pipes), **authorization** (guards), **cross-cutting logic** (interceptors), and
**error handling** (exception filters). The point — impose an architectural skeleton
so a big project doesn't sprawl; under the hood it's still Express/Fastify.

## Question 2
A **provider** is a class with `@Injectable()` that the container can create and
inject. Dependencies are declared in the **constructor**, and Nest supplies the
instances (builds the dependency graph). The **DIP** link: the class depends on an
abstraction and doesn't create the dependency itself (`new`) but receives it from
outside — that's "dependency inversion" + dependency injection as the technique.

## Question 3
- **imports** — other modules whose exports are needed;
- **controllers** — HTTP handlers;
- **providers** — this module's injectable services;
- **exports** — providers visible outside.
A non-exported provider is **private to the module**: available inside, but other
modules can't inject it (boundary encapsulation).

## Question 4
By default — **singleton** (one instance for the whole app, created at startup).
`REQUEST` scope is needed when a service needs state of a **specific request** (e.g.
the current user's data/trace). It's more expensive: an instance is created **per
request** and "infects" the whole dependency chain with the scope — more allocations,
slower.

## Question 5
```
Middleware → Guards → Interceptors(pre) → Pipes → Handler → Interceptors(post) → Exception filters
```
- **Middleware** — early (logs, cors);
- **Guards** — let through or not (authorization, `canActivate`);
- **Interceptors** — a wrapper around the call (timing, response transformation,
  cache);
- **Pipes** — validation and transformation of input (DTO);
- **Handler** — the controller method;
- **Exception filters** — turn exceptions into an HTTP error response.

## Question 6
- (a) admin role → a **Guard** (authorization);
- (b) body validation → a **Pipe** (`ValidationPipe` + DTO);
- (c) timing log → an **Interceptor** (wraps the call, measures before/after);
- (d) exception → 400 JSON → an **Exception filter**.
Each concern is its own layer; mixing them (validate in a guard, authorize in a pipe)
is an anti-pattern.

## Question 7
A controller is a **coordination** layer (the Controller role from GRASP), not
business logic. In it: request parsing (`@Param/@Body`), calling the service,
returning the result. There should be no: business rules, DB access, complex
computation — all that lives in provider services. A thin controller = high cohesion,
easy to test and to reuse the logic outside HTTP.
