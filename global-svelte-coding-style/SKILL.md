---
name: global-svelte-coding-style
description: Global coding and architecture baseline for local Svelte and SvelteKit projects, including Svelte 5, TypeScript, Prisma, Zod, security, UI, PWA, documentation, and testing. Load when creating, changing, reviewing, or designing code in this ecosystem.
---

# Global Svelte Coding Style

Use this skill as the global baseline for the local Svelte and SvelteKit projects. These rules keep the code safe, searchable, consistent, accessible, and maintainable without adding abstractions that are not yet needed.

These rules are defaults, not an excuse to ignore the project contract. Before changing existing code, analyze the folder structure, package manager, SvelteKit version, naming patterns, formatter configuration, and the nearest writing rhythm. Follow the local patterns when they do not conflict with correctness, security, or the framework contract.

## 1. Decision priority

If rules collide, use the following order:

1. Correctness, security, privacy, and data integrity.
2. The SvelteKit contract and the server/client boundary.
3. Conformance with the project's SRS, PRD, design, and domain.
4. Consistency with the nearest existing structure and patterns.
5. Local clarity, accessibility, and ease of change.
6. Reuse and performance optimization that has evidence.

Do not do mass refactoring just to tidy up style. When touching old code, fix risky local patterns if the change stays within scope. Do not leave behind aliases, compatibility shims, empty folders, placeholders, no-op branches, or implementation TODOs.

If the requirements, SRS/PRD, database, library version, or project conventions conflict with each other and cannot be determined from the repository, stop at the ambiguous decision and ask for confirmation. Do not guess business contracts or pick a database technology without a basis.

## 2. Package manager, runtime, and dependency checks

### Bun is the required default

Prefer Bun for every project action:

- Initialization: `bun create`, `bunx sv`, or an official generator run through `bunx`.
- Installing and updating dependencies: `bun install`, `bun add`, `bun remove`, `bun update`.
- Running scripts: `bun run <script>`.
- Running a CLI without a global install: `bunx <package-or-command>`.
- Do not switch to `npm`, `pnpm`, or `yarn` when Bun can perform that action.

Keep `bun.lock`/`bun.lockb` in line with the project's Bun version. Do not mix package manager lockfiles. Before adding a dependency, check whether that functionality is already provided by SvelteKit, Svelte, shadcn-svelte, or existing utilities.

### Deprecated, compatibility, and vulnerability

Before integrating a module, check the official documentation, peer dependencies, release notes, and compatibility with the project's SvelteKit/Svelte version. Use the available Bun commands, for example `bun outdated`, `bun pm ls`, `bunx <package> --help`, and official CLIs through `bunx`. Do not continue with a deprecated, abandoned, or incompatible dependency without confirmation and a written reason.

Check the dependency tree and lockfile before and after changes. Run `bun audit` if the project's Bun version supports it; if it is unavailable, use a scanner that can be run through `bunx` and record the command and the advisory sources. Do not declare a dependency safe merely because the install succeeded.

If an RCE, prototype pollution, arbitrary file write, credential leak, supply-chain issue, or another relevant vulnerability is found:

1. Stop the risky integration or upgrade.
2. Tell the user first before continuing.
3. Include the package name, affected version, safe version when known, advisory/CVE, impact description, severity, and the reason that package entered the dependency graph.
4. Include a minimal, safe, local reproduction example to show the vulnerability condition; do not run an exploit against a real system.
5. Explain the mitigation: upgrade, downgrade, patch, removal, lockfile override, server-side isolation, or package replacement.

Do not hide audit results, lower severity, delete the lockfile, or swap scanners so the results look clean. Do not put credentials, tokens, active exploits, or malicious payloads in the repository.

## 3. Technology baseline

Use or keep the following technologies when they fit the project:

- SvelteKit and Svelte 5 Runes with strict TypeScript.
- Bun as the runtime and package manager.
- Tailwind CSS and shadcn-svelte for UI primitives.
- `zod` for runtime validation.
- Prisma with a configuration that matches the Prisma version actually installed.
- MySQL or PostgreSQL as the relational database.
- Vitest for unit/integration and Playwright for browser/e2e behavior.
- MDsveX for documentation rendered by SvelteKit.
- PWA with `manifest.json` and `src/service-worker.ts` when product needs support it.

The database must stay close to MySQL or PostgreSQL. A project that has not chosen a database MUST be confirmed with the user, either through a direct question or the SRS/PRD document provided. For now MySQL may be the default only if the project context confirms it. Do not change the Prisma provider, migrations, or column types silently.

## 4. Folder structure and local-based components

Prefer local-based components: code used by only one route or one feature stays close to its consumer. Do not force developers to jump to a global folder to find a page-specific component. Move code to `src/lib/` only after it is truly shared and its contract is stable.

Suggested feature structure:

```text
src/routes/(app)/<area>/<feature>/
├── +page.svelte             # composition and UI state
├── +page.server.ts          # thin load/action and route wiring
├── components/              # feature-specific presentation components
├── services/                # use case, workflow, persistence
├── schemas/                 # boundary validation
├── types/                   # feature compile-time types
├── utils/                   # local pure helpers
└── adapters/                # optional; external transport and mapping when needed

src/lib/
├── components/              # stable cross-feature components
├── server/                  # server-only module
├── utils/                   # cross-feature utilities
└── ...

src/docs/                    # MDsveX documentation source, parallel to src/lib and src/routes
```

Create a folder only when a file needs it. Keep the SvelteKit folder conventions (`+page.svelte`, `+page.server.ts`, `+server.ts`, `+layout.svelte`, `hooks.server.ts`). Do not create `_`-prefixed helper folders just out of other habits; follow the project's existing pattern.

Layer responsibilities:

| Layer             | May have                                            | Must not have                                                                    |
| ----------------- | --------------------------------------------------- | -------------------------------------------------------------------------------- |
| `+page.svelte`    | render, presentation state, navigation, feedback    | Prisma, private env, secrets, sensitive business rules                           |
| `+page.server.ts` | route parsing, load/action, service calls           | long database workflows or markup                                                |
| `+server.ts`      | HTTP method, auth, body parsing, response           | markup or UI state                                                               |
| `components/`     | markup, typed props, callbacks, accessibility       | database queries or secrets                                                      |
| `services/`       | use case, business rule, transaction, orchestration | Svelte markup or visual details                                                  |
| `schemas/`        | input validation and normalization                  | side effects, queries, logging                                                   |
| `types/`          | compile-time contract                               | utility functions or side effects                                                |
| `adapters/`       | API communication and external mapping when needed  | becoming mandatory without a need basis or spreading external payloads to the UI |

## 5. Components, icons, and visual design

### shadcn-svelte

Aim to use shadcn-svelte on every page for buttons, inputs, forms, dialogs, sheets, tables, dropdowns, tabs, cards, alerts, tooltips, and other available UI primitives. Use the existing Bits/`cn` helper before creating a new primitive. Custom markup is still allowed when the library component does not fit semantically, in accessibility, or in design needs; explain the local decision.

### Semantic markup and DOM structure

Use shadcn-svelte primitives with appropriate semantic HTML. Prefer `<main>`, `<header>`, `<nav>`, `<section>`, `<article>`, `<aside>`, `<footer>`, `<form>`, `<fieldset>`, `<ul>`, and `<ol>` according to the meaning of the content. Avoid raw HTML that replaces a shadcn-svelte component when the component is available, but do not use `<div>` as a substitute for semantic elements.

Avoid `div soup` and nested `<div>`s that have no clear layout or accessibility responsibility. Every wrapper MUST have a reason that can be explained. Split components only when it improves readability, real reuse, or accessibility; do not create wrappers and decorative components with no value.

### Icon

Prefer `@hugeicons/core-free-icons` and `@hugeicons/svelte`. Do not draw SVG by hand or mix several icon libraries without a design/compatibility reason. An icon-only control MUST have an accessible label or title. An icon MUST NOT be the only carrier of important information.

### Page style

- Use a consistent border-radius. Define a radius token or scale in the theme and then use it consistently; vary it only if UI/UX principles require it.
- Use the Poppins font when the project has no established brand font. Do not use fonts that look garish, dated, or unfitting for the product context.
- Use gradients based on the SRS/PRD, brand, product context, contrast, and color scheme theory. Do not pick Indigo/Blue at random. Document the reason for the primary color when the visual decision is not obvious.
- Ensure contrast, focus state, keyboard navigation, reduced motion, and interaction target size are usable.
- Avoid emoji in the UI, code, error messages, file names, and technical documentation.
- Do not stack badges above section headings. Badges are only used when they give real status information, especially on shadcn-svelte table components or profile pages when they are truly needed.
- Every page still has meaningful loading, empty, error, and success feedback.

## 6. Naming, TypeScript, and formatting

- Files and folders lowercase kebab-case, except for special SvelteKit files.
- The component `modal-action.svelte` uses the symbol `ModalAction`.
- The service `form.service.ts` uses the class `FormService`.
- The schema `<entity>.schema.ts` uses one clear constant name.
- Technical identifiers in English; labels and messages follow the application language.
- Function/variable camelCase; booleans use `is`, `has`, `can`, or `should`.
- Type/interface PascalCase without the `I` prefix; place type/interface in `types/`, not in a service/schema.
- Scalar config constants use `SCREAMING_SNAKE_CASE` when the project pattern uses it.
- Keep `strict: true`; do not use `any`, `@ts-ignore`, or `@ts-expect-error` without a documented reason.
- An untrusted boundary starts as `unknown`, is validated with Zod, then narrowed.
- Use `satisfies` for config and `z.infer` for types derived from a schema.
- Use the virtual types from `./$types`; do not import types from a physical route file.
- Public/exported functions in routes, services, adapters, utils, and test helpers have an explicit return type.
- Do not run an automatic formatter after any code modification. Prettier, Biome, editor formatters, and other formatters are FORBIDDEN after code is modified, changed, edited, written, or updated. Format code manually and keep the local style without adding formatter workarounds.
- The import/export order inside curly braces MUST be alphabetical A-Z, including `import { afterEach, beforeEach, describe, it, vi } from "vitest";`.
- The import/export statement order follows the source: packages or aliases starting with `@`, then sources without `@` and without `$`, then aliases starting with `$`; each group is sorted A-Z. For example the package `@hugeicons/svelte` comes before `svelte`, and `svelte` comes before `$lib`/`$routes`.
- Use the full file/package path for imports and exports. For internal modules, prefer the full alias such as `$routes/(app)/.../form.service.ts` or `$lib/...`; avoid relative paths such as `./form.service.ts` when a project alias is available. Do not shorten paths until the layer/feature intent is no longer visible. The only exception is a REQUIRED SvelteKit virtual module such as `./$types` or a pattern mandated by the toolchain.
- Keep `import type` in the same group and do not sacrifice the order of the curly brace members.

Apply KISS, YAGNI, and DRY in a balanced way. Do not create helpers, wrappers, type aliases, or abstractions before there is a real need. A function of only 1–3 lines or a variable used only once, including a `$derived` that is not reused, is merged into its consumer's expression if it does not reduce readability, debuggability, accessibility, or type safety. Do not inline logic that has side effects, an error boundary, an important domain name, or reuse potential.

## 7. Services, schemas, and state utilities

Services and state utilities based on `*.svelte.ts` MUST be OOP and use classes with clear responsibilities. Use static methods only for stateless operations without dependencies; use instances and dependency injection for clients, adapters, clocks, repositories, or boundaries that need to be swapped during tests. Do not create an empty class just as a namespace.

OOP files such as `*.service.ts` and `*.svelte.ts` are STRICTLY FORBIDDEN from having `function`, `const`, `let`, helpers, constants, or executable code outside the class other than the necessary imports. All helpers, constants, and behavior MUST become class members or methods. Do not create new top-level declarations to work around this rule.

Methods/functions in service files and `*.svelte.ts` are sorted alphabetically A-Z by method/function name. This order applies inside the class and across top-level declarations. Do not merge unrelated types/interfaces, schemas, or helpers into those files.

`form.service.ts` has a stricter rule: after the imports and the required class declaration, provide only the `create`, `delete`, and `update` operations as the form requires. Do not put `type`, `interface`, schema, utilities, unrelated queries, or extra methods in it. If soft delete is used, the `delete` method must follow the meaning of the project contract and is better given a domain name such as `deactivate` when it does not actually delete data.

A `*.schema.ts` file has as much as possible only one exported schema constant and does not export a `type` or `interface`. Validation must be strict and reflect the form input and the database invariants: trim, format, enum, range, the required FormData coercion, optional blank normalization, and `.strict()` for objects that must not accept foreign fields. Put the inferred result type in `types/` or the consumer file.

Do not create a new `type` or `interface` when an official type from the package, SvelteKit, the Prisma Client from `bunx prisma generate`, or a related library is already available. For a Prisma model, use the generated model/type such as `Prisma.<Model>GetPayload`, `Prisma.<Model>CreateInput`, or `Prisma.<Model>UpdateInput` according to the boundary; do not copy model fields into an identical feature type/interface. Create a new type/interface only when it forms an API contract, an external payload, or a DTO that genuinely differs from the model; document that API contract in English JSDoc.

## 8. Svelte 5 and SvelteKit

- New code uses Svelte 5 Runes.
- Props typed through `$props()`; use `$bindable()` only when the child really mutates the prop.
- Use `$state` for state read by the template, `$derived` for derived values, and `$effect` only for side effects with cleanup.
- Use `fly` from `svelte/transition` when a transition helps user orientation or makes the page feel more alive:
  `import { fly } from "svelte/transition";`
- Use `fly` sensibly with smooth duration, direction, and distance. Do not add a transition to every element or make content hard to access; respect `prefers-reduced-motion` and do not sacrifice loading/error feedback.
- Use `onclick` and modern event attributes, not `on:click`, in new code.
- Use `{#snippet}` and `{@render}` for composition as well as a unique domain key on each block.
- A SvelteKit route module MUST follow the contract of its own file type, and only export official framework APIs that are actually needed. No specific export is always required. `+page.ts` may export `load`, page options (`prerender`, `ssr`, `csr`, `trailingSlash`, `config` when supported by the SvelteKit version), and `entries` for dynamic routes when needed; `+page.server.ts` may export the server-only equivalents, `load`, and `actions`. `actions` is only valid in `+page.server.ts`.
- `+layout.ts` and `+layout.server.ts` may export `load` and the official, relevant layout/page options, but not `actions`. `+server.ts` exports the official HTTP handlers (`GET`, `POST`, `PUT`, `PATCH`, `DELETE`, `OPTIONS`, `HEAD`, or `fallback`) and official endpoint options such as `prerender`, `trailingSlash`, `config`, and `entries` when supported and needed; do not mix `load` or `actions` into it. Always match the export list against the documentation and generated types of the project's SvelteKit version.
- `load` may hold route logic directly without being REQUIRED to call a service/controller. `actions` may hold the local declarations needed inside the handler and may call a service/controller as the architecture requires; it is not REQUIRED to always be inline. Avoid helpers, per-request mutable state, and reusable/complex business logic at the module level when not needed; move reusable or complex logic to the appropriate layer.
- Do not import private env, Prisma, bcrypt, or `.server.ts` into the client.
- Auth/RBAC MUST be enforced on the server, not only by hiding the UI.
- Form mutations are revalidated on the server; client validation is only early feedback.
- An endpoint reads JSON as `unknown`, validates it, and only then uses the parsed data.
- Use semantic statuses: 400 malformed, 401 unauthenticated, 403 forbidden/CSRF, 404 missing, 409 conflict, 500 unexpected.
- Keep redirects/framework errors that must be re-thrown. Do not turn them into 500.
- Do not send stack traces, queries, tokens, credentials, or raw database errors to the client. Humanize error messages so they are safe, short, and actionable; technical details go to the structured logger.

Use JWT or tokens only in crucial parts if the feature or SRS/PRD requires them, for example session/API credentials that are genuinely token-based. Do not add JWT merely as a decorative standard; choose the cookie session, token, expiry, rotation, revocation, audience, issuer, and storage based on the threat model. Tokens must not enter logs, URLs, HTML, or unnecessary responses.

Apply CSRF protection for cookie/session-based mutations when relevant, including SameSite cookies, origin/referer validation, and CSRF tokens as the architecture requires. Prevent XSS with Svelte's default escaping, URL/protocol validation, CSP/security headers, and official library sanitization before using `{@html}`. Avoid `{@html}` when the content does not genuinely need rich text; do not build HTML from user input.

For `.svelte` file changes, read the related Svelte skill and use the available editor/autofixer or MCP. Do not copy framework rules already provided by the tool when that only adds duplication.

## 9. Prisma, database, and data security

Use a single Prisma client singleton at the server boundary, for example `src/lib/server/prisma.ts` or the equivalent pattern in the existing project. Do not create `new PrismaClient()` in every service. Align the generator, adapter, datasource, migration, and Prisma API with the Prisma version installed; check the official documentation for that version through Bun before configuring.

- Queries select the minimum fields with a deliberate `select`/`include`.
- Apply `deletedAt: null` when the model uses soft delete.
- Use `$transaction` for atomic multi-write and use `tx` consistently inside the callback.
- Avoid N+1 queries and `Promise.all` for writes that have dependencies.
- Use `Decimal` for financial/precise-quantity values.
- Use UTC/ISO for persistence and identifiers.
- Audit logs are append-only and always sanitize sensitive fields.
- Do not store per-request mutable state in a shared server module.

Use appropriate hashing for passwords and sensitive data such as NIK. Do not store plaintext, reversible encoding, weak hashes, hardcoded secrets, or credentials in fixtures/logs/documentation. Minimize sensitive data, restrict access, and do not display sensitive values in responses.

Use a rate limiter for transactional endpoints, login, credential reset, upload, or operations that can be abused. The rate limit MUST be server-side, have the right key, and return a humanized message without leaking internal details.

JWT, tokens, CSRF, and XSS protection remain conditional. Choose only the mechanism that fits the feature, auth model, browser boundary, and SRS/PRD; document the decision and the threat it handles.

## 10. Zod and error handling

Every form, important query, JSON body, upload metadata, external response, and untrusted runtime configuration MUST have a Zod schema. Separate the create/update schema when the required fields or invariants differ. Do not display a raw `ZodError`; turn it into a field error that the UI can render.

Use a centralized logger with context (`entityId`, `operation`, `status`, `error`) and sanitize passwords, JWT/CSRF tokens, cookies, API tokens, connection strings, request credentials, and unsanitized payloads. Catch exceptions as `unknown`, preserve the cause/context, and humanize the message for the user. Do not create a catch that only returns `String(error)`.

## 11. MDsveX documentation and JSDoc

Aim for MDsveX integration matching the project's SvelteKit version. Put documents in `src/docs`, parallel to `src/lib` and `src/routes`, then build a dedicated documentation layout that has navigation, a sidebar/TOC, breadcrumbs when relevant, search/filter when available, code blocks, link state, a responsive layout, and accessibility. Its visuals follow modern technical documentation patterns such as Astro Documentation or Prisma Documentation without copying their brand.

Source documentation is written in simple, easy-to-understand English. Public API JSDoc is also written in plain English. When annotations are used, write the explanation on a new line after the annotation:

```ts
/**
 * Creates a customer after validating the submitted form data.
 *
 * @param input
 * The normalized customer fields received from the server boundary.
 * @returns
 * The persisted customer summary without sensitive fields.
 * @throws
 * When the customer conflicts with an existing unique record.
 * @example
 * const customer = await customerService.create(input);
 */
```

Use `@description`, `@param`, `@returns`, `@throws`, and `@example` when relevant. Do not write JSDoc that merely repeats the type name. Document the reason, input, output, failure mode, security boundary, and a genuinely understandable example.

## 12. PWA and service worker

Prioritize PWA when the application needs require it. Follow the SvelteKit conventions for `manifest.json`, icon assets, `src/service-worker.ts`, registration, caching, offline fallback, update lifecycle, and SSR safety. Do not claim offline support if the queue/sync behavior has not genuinely been implemented.

Every behavior or function related to `self` in `src/service-worker.ts` is sorted alphabetically A-Z by function/handler name. Give it English JSDoc with the relevant annotations (`@description`, `@param`, `@returns`, `@throws`, `@example`). Ensure install, activate, fetch, message, cache invalidation, and update behavior do not swallow errors silently. Test the service worker behavior with an appropriate browser boundary.

## 13. Static assets and orphan files

Avoid orphan asset files in `static/`: group icons, images, fonts, and feature assets in meaningful folders. Root files may only be exempted when they are convention configuration such as `robots.txt`, `manifest.json`, or an equivalent important file.

Apply the same principle to `src/`: files MUST live in a clear layer/feature folder and not be scattered at the root without a reason. Do not move existing files in bulk just to satisfy this rule; when a feature is touched, tidy up the local safe paths and migrate all imports/callers.

## 14. CDN and cost burden

Do not add CDN links or dependencies automatically. If a CDN is considered, first calculate the financial and operational impact based on project data: asset size and monthly traffic, origin egress, request count, a realistic cache-hit ratio, CDN cost, request/origin fetch cost, region, invalidation, and additional security/compliance cost. Minimal estimate:

```text
origin savings   = (cached traffic × origin egress cost)
                   + (reduced origin requests × origin request cost)
net CDN cost     = CDN cost + egress to CDN + requests/invalidation + operational cost
net benefit      = origin savings - net CDN cost
```

Give the user confirmation before adding a CDN or a CDN URL, including the assumptions and the estimate range. If the net benefit is not material, not proven, or gets worse, use local assets/packages and avoid the CDN. Also consider supply-chain risk, CSP, SRI, availability, privacy, and cache invalidation; cost savings are not the only criterion.

## 15. Testing with Vitest and Playwright

Use Vitest for schemas, utilities, pure domain logic, services with mocked boundaries, workers, and API handlers. Use Playwright for real user behavior, accessibility, navigation, auth flows, and PWA surfaces.

Default structure:

```text
tests/
├── unit/<domain-or-layer>/*.test.ts
├── integration/*.test.ts
└── e2e/*.e2e.test.ts
```

The Vitest configuration follows the PPC pattern already referenced: the SvelteKit plugin, `$lib`/`$routes` aliases, an environment matching the boundary, coverage excludes for generated declarations, tests, docs, and schemas when that is indeed the project coverage policy, and includes for `tests/unit` and `tests/integration`. The Playwright configuration places tests in `tests/e2e`, uses the `.e2e` suffix, and runs the server through Bun. Do not copy the configuration verbatim; match the version and scripts of the project being worked on.

Test rules:

- `describe` names the feature/unit; `it` names the observable behavior.
- Use Arrange, Act, Assert and deterministic fixtures.
- Mock only boundaries: database, network, browser API, and clock.
- Test success, invalid input, not found, conflict, authorization, real errors, and important transitions.
- Test atomicity, rate limits, hashing boundaries, retry/dedupe, service worker lifecycle, and real item counts when that is a contract.
- Restore fetch, timers, navigator, storage, and mocks after the test.
- Assertions MUST fail on a wrong implementation; avoid generic `toBeDefined`, `not.toThrow`, `toBeTruthy`, and assertions that accept every outcome.
- Do not test source text, wiring, field-by-field copies, or incidental defaults.

## 16. Code change SOP

1. Understand the SRS/PRD, actors, roles, routes, inputs, outputs, errors, persistence, and the behavior being changed.
2. Analyze the project rhythm: package manager, framework version, sibling features, schemas, services, routes, components, tests, config, and naming.
3. Find every caller/export/reference before changing a contract.
4. Define the server/client boundary, DTO, schema, auth, rate limit, transaction, external integration when needed, and UX state.
5. Implement starting from the data boundary and schema, then types, adapters when needed, services, routes, components/pages, documentation, and tests.
6. Use Bun for installs, generators, scripts, compatibility checks, and audits.
7. Humanize errors, protect sensitive data, and do not send internal details to the user.
8. Run the relevant quality checks and verification. Automatic formatters remain FORBIDDEN:
   - `bun run check` for the SvelteKit, TypeScript, and accessibility validation configured in the repository.
   - `bun run typecheck` when the script is available, or the equivalent official repository typecheck command.
   - `bun run build` to ensure the application can be compiled and bundled.
   - `bunx eslint <changed-files>`
   - `bunx vitest run <targeted-test>` or `bunx playwright test <targeted-e2e>`
   - dependency audit through Bun or a Bun-compatible scanner when dependencies change.
     Run `typecheck`, `build`, and `bun run check` when relevant to the change; all three MAY and are RECOMMENDED to ensure no code has errors. Do not run `prettier`, `biome`, or any other automatic formatter.
9. For UI/PWA, do a smoke test on the actual surface; ensure accessibility, responsive layout, loading/error/empty state, cache/update behavior, and no-leak server boundary.
10. Review as a consumer and finish the cutover: migrate callers, remove obsolete paths, and do not leave placeholders, deprecated aliases, no-ops, credentials, or formatter workarounds.

If the work starts to leave the context of this skill, conflicts with SvelteKit conventions, does not match the SRS/PRD, or requires a decision that cannot be proven from the repository, give the user confirmation before choosing the implementation direction.

## 17. Definition of done

A change is ready to be considered done only if:

- Folders, files, and components are in the correct local feature layer.
- Skill metadata and dependency actions use the correct global names and commands.
- There is no server/private import into the client.
- Inputs, config, and external responses are strictly validated with Zod.
- Services and `*.svelte.ts` are OOP, methods/functions A-Z, and layers are not mixed.
- OOP files have no `function`, `const`, `let`, helper, constant, or executable code outside the class; only the necessary imports may sit outside the class.
- A route module only exports official SvelteKit framework APIs, valid for that file type, and genuinely needed; there is no two-export whitelist and no `load`/`actions` that is always required. Distinguish the `+page.server.ts`, `+page.ts`, `+layout.server.ts`, `+layout.ts`, and `+server.ts` contracts according to the documentation and generated types of the SvelteKit version. Local declarations needed inside the handler are allowed; still avoid helpers, per-request mutable state, and reusable/complex business logic at the module level when not needed.
- Imports/exports are sorted A-Z inside the curly braces, the source groups follow `@` then non-`@`/non-`$` then `$`, and full internal paths use aliases when available.
- Adapters are only created when the system or SRS/PRD needs them.
- Markup uses semantic tags, avoids `div soup`, and the `fly` transition is used subtly when relevant.
- KISS, YAGNI, and DRY are applied without sacrificing readability, accessibility, type safety, or observability.
- There are no duplicate types/interfaces from generated Prisma or official packages; new API contracts are documented.
- JWT/tokens, CSRF, and XSS protection are applied conditionally based on the threat model and feature needs.
- No CDN is added without a cost/benefit estimate and user confirmation.
- `form.service.ts` only has create/delete/update; the schema exports only one constant without type/interface.
- Prisma uses a singleton, the configuration version matches, queries are safe, and transaction boundaries are correct.
- Passwords/sensitive data are hashed, transactional endpoints have a rate limit, and errors are humanized.
- The UI uses shadcn-svelte and Hugeicons when appropriate, radius/font/color are consistent, without emoji and excessive badges.
- Documentation lives in `src/docs`, uses MDsveX/layout, and public API JSDoc is useful English.
- The PWA manifest/service worker follow the conventions, `self` behavior is sorted A-Z, and is documented when used.
- There are no orphan assets or source files without a convention reason.
- Vitest/Playwright prove important behavior, not merely wiring.
- The relevant checks, lints, and smoke/targeted tests have been run without an automatic formatter.
- There are no placeholders, no-ops, raw secrets, credentials, active exploits, unconfirmed deprecated dependencies, or unjustified formatter workarounds.