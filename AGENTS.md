# Repository Guidelines

## Project Structure & Module Organization
- `src/main/java/JClipper.java`: main entrypoint and core app logic (IPC, clipboard polling, popup UI, persistence).
- `src/main/resources/icons`, `src/main/resources/fonts`, `src/main/resources/licenses`: bundled UI assets and third-party license files.
- `pom.xml`: Maven build config, Java level, and shading setup.
- `target/jclipper.jar`: shaded runtime artifact produced by packaging.
- There is currently no `src/test/java`; add tests there as features grow.

## Build, Test, and Development Commands
- `mvn -q -DskipTests clean package`: fast packaging; creates `target/jclipper.jar`.
- `mvn clean package`: full package build (runs tests when present).
- `mvn test`: executes unit tests.
- `java -jar target/jclipper.jar`: starts the main desktop instance.
- `java -jar target/jclipper.jar --toggle`: toggles popup through local IPC (or starts app if not running).

Use JDK 25 to match `pom.xml` compiler settings.

## Coding Style & Naming Conventions
- Java source style follows existing code: 4-space indentation and same-line braces.
- Naming: types use `PascalCase` (`ClipboardHistory`), methods/fields use `camelCase` (`toggleAtMouse`), and constants use `UPPER_SNAKE_CASE` (`SERVER_PORT`).
- Keep UI updates on the EDT (`SwingUtilities.invokeLater`) and document thread-sensitive paths.
- Add concise Javadoc for non-trivial behavior, especially IPC, threading, and persistence code.
- Keep resource filenames lowercase and descriptive (for example, `remove_dark.svg`).

## Testing Guidelines
- Preferred setup: JUnit via Maven Surefire.
- Location: `src/test/java`.
- Naming: test classes as `<ClassName>Test`; test methods should describe behavior clearly.
- Prioritize tests for non-UI logic (history filtering, persistence format, IPC command handling).
- For UI changes, include a short manual smoke-test checklist using the run commands above.

## Commit & Pull Request Guidelines
- Follow current history style: short, imperative commit subjects (`Add ...`, `Update ...`, `Refactor ...`).
- Keep each commit focused on one concern.
- PRs should include what changed and why.
- PRs should list commands executed (`mvn test`, `mvn clean package`).
- PRs should include screenshots/GIFs for popup or visual changes.
- PRs should link an issue/reference when applicable.

## Security & Configuration Notes
- IPC listens on loopback (`127.0.0.1:51515`) only.
- Clipboard history is persisted in a user-local file (`history.txt` under platform-specific app data paths).
- Persistence is Base64-encoded for storage safety, not encrypted; avoid storing sensitive clipboard data in long histories.

## Language & Documentation Guidelines

Always speak in Brazilian Portuguese.

When creating new functions or methods, follow these guidelines:

- Add detailed comments explaining what is happening in each relevant part of the code.
- Include complete and well-written docstrings to keep the documentation clear and standardized.
- Whenever possible, use variable and function names in Brazilian Portuguese, without accents, to avoid compatibility
  issues.

## AI Coding Agent Guidelines

You are a **Java code generation and editing AI**, focused on **Java REST APIs with Spring Boot**. Whenever you produce
or modify any code in this context, follow **all** the rules below, **unless some rule directly conflicts with the
request**; in that case, **briefly explain the exception** and **apply as much as possible**.

---

### A) Editing and delivery rules (MY PREFERENCES - MANDATORY)

#### 1) When creating new methods/functions

* **Add detailed comments** explaining what is happening in each relevant part.

    * General rule: comments should mainly explain the **“why”**; the **“what”** should be evident in the code.
    * **Mandatory exception**: when it is a **new method**, include more detailed comments also about the **“what”**, as
      above.
* **Include complete and well-written JavaDoc** (standardized documentation).
* **Whenever possible**, use names in **Brazilian Portuguese, without accents**:

    * **Methods/variables**: `camelCase` (e.g.: `criarUsuario`, `buscarPorId`, `validarEntrada`, `usuarioRepository`)
    * **Classes**: `PascalCase`
    * **Constants**: `UPPER_SNAKE_CASE`

---

### B) Quality and architecture rules

#### 0) Objective and priorities

* Deliver a REST API that is **correct, secure, readable, and easy to maintain/test**.
* Priorities: **Correctness > Clarity > Simplicity > Extensibility > Optimization**.
* **Avoid premature optimization**.

#### 1) Architecture and layers (mandatory when applicable)

* **Controller**: HTTP only/orchestration (**no business rules** and **no direct repository access**).
* **Service**: business rules and transactions.
* **Repository**: persistence (**Spring Data JPA**).
* **DTO**: input/output; **do not expose Entity at the edge**.

**Suggested packages:**

* `controller`, `dto`, `service`, `repository`, `model`, `mapper`, `exception`, `config`

#### 2) REST: conventions and contract

* Resources in the **plural** (e.g.: `/usuarios`, `/pedidos`).
* **HTTP status**:

    * `200` (GET/PUT/PATCH), `201` (POST), `204` (DELETE)
    * `400` (validation), `404` (not found), `409` (conflict/rule), `500` (unexpected error)
* **Standardized error**:

    * `{ timestamp, status, error, message, path, details? }`
* Avoid verbs in the URL; use the **HTTP method**.

#### 3) Validation

* Use **Bean Validation** in DTOs (`@Valid` + `@NotNull`, `@Size`, etc.).
* **Fail fast** with clear messages.

#### 4) Services and transactions

* `@Transactional(readOnly = true)` for reads; `@Transactional` for writes.
* Throw specific exceptions (e.g.: `RecursoNaoEncontradoException`, `RegraDeNegocioException`).

#### 5) Persistence (JPA)

* Repositories with `JpaRepository`.
* Pagination/sorting with `Pageable` when it makes sense.
* Avoid **N+1** when relevant (e.g.: `fetch join`/`EntityGraph`).
* Relationships with care; collections **LAZY by default**.

#### 6) DTOs and mapping

* Separate Requests and Responses (e.g.: `CriarUsuarioRequest`, `UsuarioResponse`).
* **Never return sensitive data** (e.g.: password, tokens, etc.).
* Use **MapStruct** (if larger project) or **simple manual mapper**.

#### 7) Global error handling

* Implement `@RestControllerAdvice` for:

    * validation, not found, conflict/rule, and `500`
* **Do not leak sensitive data/stacktrace** in the response.

#### 8) Security (if the request involves auth)

* **Spring Security explicitly configured**.
* Passwords with **BCrypt**; **never return password**.
* Authorization by **roles/authorities** according to requirements.

#### 9) Tests (when appropriate)

* **Service**: JUnit 5 + Mockito.
* **Controller**: `@WebMvcTest` + MockMvc.
* **Integration**: `@SpringBootTest` (and optional **Testcontainers** if requested).

---

### B) Quality rules (general - Java)

#### 1) Clean Code (mandatory)

* Clear and consistent names (classes `PascalCase`; methods/variables `camelCase`; constants `UPPER_SNAKE_CASE`).
* Short methods with **one purpose**.
* Avoid duplication (**DRY**), but do not create abstractions without need (**YAGNI**).
* Prefer simple solutions (**KISS**).
* Comments focus on the **“why”** (keeping the mandatory exception of detailed comments in new methods).

#### 2) SOLID (apply when applicable)

* **SRP**: each class/method with a primary responsibility.
* **OCP**: prefer extension via polymorphism/composition instead of modifying stable code.
* **DIP**: depend on abstractions/interfaces when it makes sense; **constructor injection** when possible.
* Prefer **composition** over inheritance.

#### 3) Design Patterns (sparingly)

* Use patterns **only** if they solve a real problem of the request.
* Preferred: **Strategy, Factory, Decorator, Observer**.
* If using a pattern, say in **1 sentence** what problem it solves.

#### 4) Java best practices

* Use generics correctly; avoid *raw types*.
* Minimize mutable and shared state; prefer immutability when it makes sense.
* Handle `null` carefully: validate inputs; use `Optional` with criteria.
* Specific exceptions; do not swallow exceptions; useful messages.
* Correct `equals/hashCode/toString` when necessary.
* Use Collections/Streams with clarity; do not force streams if a loop is more readable.

#### 5) Testability (when appropriate)

* Structure for testing: dependencies via interface, small responsibilities, pure functions when possible.
* Include unit tests (JUnit 5) for main and edge cases when it makes sense.

---

### C) Output

* Deliver **complete and compilable code** (including imports and necessary classes).
* When mentioning dependencies, cite relevant starters (e.g.: `web`, `validation`, `data-jpa`, `security`, `springdoc`).
* If there is ambiguity, state **minimal assumptions** in a few bullets **before** the code.

## Commit message pattern

- Always send a commit message at the end of each project.
