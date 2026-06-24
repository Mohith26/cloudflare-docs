# PLAN: Sandbox SDK next-branch migration documentation

Created: 2026-06-24
Last updated: 2026-06-24

## Goal

Replace the current 2026 deprecation stubs with a complete migration plan for the Sandbox SDK `next` branch. The docs should tell existing users to migrate to `@cloudflare/sandbox@next` as soon as possible, and tell new projects to start on `next` today.

The migration guide should cover five major topics:

1. Removal of HTTP and WebSocket transports.
2. Consolidation of `exec()`, `execStream()`, and `startProcess()` into one `exec()` API that aligns with the Containers `container.exec()` API.
3. Removal of the default session and the `enableDefaultSession` flag from `getSandbox()`.
4. Extraction of core functionality into plugins, specifically Code Interpreter, Git, and PTY support.
5. New extension API for extending the Sandbox SDK.

## Source material

Use these PRs as sources of truth before writing final copy:

- Sandbox SDK PR: [cloudflare/sandbox-sdk#765 — Make top-level sandbox execution stateless by default](https://github.com/cloudflare/sandbox-sdk/pull/765)
  - Base branch: `next`.
  - Key themes: no hidden default session, sessionless top-level execution, removal of `execStream()` public surface, execution runtime changes.
- Sandbox SDK PR: [cloudflare/sandbox-sdk#766 — Implement Sandbox extension framework](https://github.com/cloudflare/sandbox-sdk/pull/766)
  - Key source for the extension API public interface.
- Sandbox SDK PR: [cloudflare/sandbox-sdk#767 — Migrate code interpreter into a `SandboxExtension`](https://github.com/cloudflare/sandbox-sdk/pull/767)
  - Key source for plugin/extension migration patterns.
- Cloudflare Docs PR: [cloudflare/cloudflare-docs#31565 — containers: Add documentation for the newly added `container.exec()`](https://github.com/cloudflare/cloudflare-docs/pull/31565)
  - Use this to mirror the new Sandbox `exec()` shape where details are known.

## Content strategy

### Rename the guide and skill

Rename the current stub paths:

- From: `src/content/docs/sandbox/guides/2026-deprecation.mdx`
- To: `src/content/docs/sandbox/guides/migrating-to-next.mdx`

Rename the static agent skill:

- From: `public/sandbox/guides/2026-deprecation/SKILL.md`
- To: `public/sandbox/guides/migrating-to-next/SKILL.md`

Use uppercase `SKILL.md` to match Agent Skill conventions.

### Reframe the changelog

Update `src/content/changelog/sandbox/2026-06-09-deprecating-sandbox-sdk-features.mdx` so it no longer frames this as imminent removal from the current branch. Instead:

- Explain that deprecated APIs are marked in the primary branch.
- Explain that the replacement APIs are available on the `next` branch.
- Show the installation command:

```sh
npm install @cloudflare/sandbox@next
```

- State that existing projects should migrate as soon as possible.
- State that new projects should use `@cloudflare/sandbox@next` today.
- Link to `/sandbox/guides/migrating-to-next/` and `/sandbox/guides/migrating-to-next/SKILL.md`.
- Replace relative deadline language with the explicit target date `2026-07-31` (July 31, 2026). For user-facing prose, write “July 31, 2026” rather than “in 30 days.”
- Remove or rewrite Desktop-specific removal language unless it maps directly to PTY/plugin extraction.
  - `exposePort()` deprecation, unless separately confirmed as in scope.

## Target guide structure

File: `src/content/docs/sandbox/guides/migrating-to-next.mdx`

Recommended frontmatter:

```yaml
---
title: Migrate to Sandbox SDK next
pcx_content_type: how-to
sidebar:
  order: 99
description: Migrate existing Sandbox SDK projects to the next branch and update deprecated APIs.
products:
  - sandbox
---
```

Recommended opening:

- Briefly define the `next` branch.
- Tell existing projects to migrate as soon as possible.
- Tell new projects to install `@cloudflare/sandbox@next` today.
- Include a note that some interfaces are still stabilizing where the source PRs are still open.

### Section 1: Install the next branch

Required content:

- Installation command:

```sh
npm install @cloudflare/sandbox@next
```

- If the repo uses `pnpm` or `yarn`, show equivalent commands only if using the `PackageManagers` component is appropriate for this docs area.
- Explain that the package should be upgraded in both local development and deployment environments.
- Tell users to deploy the Worker and container image together when behavior spans SDK and container runtime.
- Add a short verification checklist:
  - package version resolves to `next`
  - Worker deploy succeeds
  - basic sandbox creation succeeds
  - one top-level `exec()` call succeeds

### Section 2: Migrate deprecated features

Create one subsection for each major topic.

#### HTTP and WebSocket transports

Required content:

- State that HTTP and WebSocket transports are removed in `next`.
- State what users should do before migrating:
  - On the primary branch, set `SANDBOX_TRANSPORT=rpc` to smoke-test current code on RPC.
  - Remove code or configuration that forces `http` or `websocket`.
- State what users should do after installing `next`:
  - Use the supported transport path only.
  - Remove obsolete transport toggles if they no longer apply.
- Link to `/sandbox/configuration/transport/`, but update that page too so it does not contradict the guide.

Acceptance criteria:

- [ ] The guide clearly says HTTP and WebSocket are not available on `next`.
- [ ] The guide explains how to test on RPC before switching package tags.
- [ ] The transport reference page has a deprecation note or `next` migration callout.

#### Execution API consolidation

Required content:

- State that `exec()`, `execStream()`, and `startProcess()` are being consolidated into a single `exec()` API.
- State that the new interface mirrors the Containers `container.exec()` model.
- Inline the relevant Containers-style `exec()` interface and behavior in this guide instead of sending readers to another page for the core API shape.
- Use PR #31565 as source material, but make the Sandbox guide self-contained.
- Mark exact Sandbox examples as TBD until the public Sandbox API shape is confirmed.
- Include a placeholder migration table:

| Current API                     | Next API                                 | Notes                                   |
| ------------------------------- | ---------------------------------------- | --------------------------------------- |
| `sandbox.exec(command)`         | `sandbox.exec(...)`                      | TBD: update when final interface lands. |
| `sandbox.execStream(command)`   | `sandbox.exec(...)` with streamed output | TBD.                                    |
| `sandbox.startProcess(command)` | `sandbox.exec(...)` process handle       | TBD.                                    |

Acceptance criteria:

- [ ] The guide does not publish guessed `exec()` signatures as final.
- [ ] The guide explicitly labels `exec()` migration examples as TBD where the interface is not finalized.
- [ ] The guide points readers to the Containers `exec()` model as the expected shape.
- [ ] `src/content/docs/sandbox/api/commands.mdx`, `guides/execute-commands.mdx`, and `guides/streaming-output.mdx` are updated or annotated so they do not conflict with the migration guide.

#### Default sessions and `enableDefaultSession`

Required content:

- State that hidden default sessions are removed on `next`.
- State that `enableDefaultSession` is removed from `getSandbox()`.
- Explain the replacement behavior:
  - Top-level operations are stateless/sessionless by default.
  - Persistent shell state requires an explicit session.
- Show old and new patterns:

```ts
// Before
const sandbox = getSandbox(env.Sandbox, "my-sandbox", {
	enableDefaultSession: false,
});
```

```ts
// After
const sandbox = getSandbox(env.Sandbox, "my-sandbox");
```

```ts
// Use an explicit session when shell state must persist.
const session = await sandbox.createSession();
await session.exec("cd /workspace");
await session.exec("pwd");
```

Acceptance criteria:

- [ ] The guide tells users to remove `enableDefaultSession` from `getSandbox()` options.
- [ ] The guide distinguishes stateless top-level calls from explicit session workflows.
- [ ] Existing session docs and sandbox options docs include a migration note.

#### Plugins: Code Interpreter, Git, and PTY support

Required content:

- State that Code Interpreter, Git, and PTY support are extracted from core SDK functionality into plugins/extensions.
- For each plugin, include:
  - current/core API name or docs page
  - next/plugin import path following `@cloudflare/sandbox/<plugin>`
  - `with<Plugin>` helper usage
  - before/after code example when available
- The pattern from PR #767 is:

```ts
import { with<Plugin> } from "@cloudflare/sandbox/<plugin>";

export class MySandbox extends Sandbox {
  <plugin> = with<Plugin>(this);
}
```

- Public interfaces should remain the same as the current core APIs, but namespaced on the sandbox instance through the plugin.
- Suggested subsections:
  - Code Interpreter plugin
  - Git plugin
  - PTY plugin

Acceptance criteria:

- [ ] The guide names all three extracted capabilities.
- [ ] Plugin import paths follow `@cloudflare/sandbox/<plugin>`.
- [ ] Plugin setup examples use `with<Plugin>(this)` on the Sandbox instance.
- [ ] Existing docs for interpreter, Git workflows, and terminal/PTY features link to the migration guide or include `next` callouts.

#### Extension API

Required content:

- Introduce the new Sandbox extension API.
- Explain when users should create an extension instead of adding helper code around the SDK.
- Pull public interfaces from PR #766.
- Include a minimal skeleton only after the API shape is confirmed.
- Include a “Port a helper into an extension” migration note if supported by PR #767.

Acceptance criteria:

- [ ] The guide includes the public extension API interfaces or clearly labels them TBD.
- [ ] The guide links extension concepts to the Code Interpreter migration example from PR #767.
- [ ] The docs use the `@cloudflare/sandbox/<plugin>` and `with<Plugin>(this)` pattern from PR #767, and avoid over-promising lifecycle hooks beyond that pattern.

### Section 3: Public interfaces

The final section should collect the public interfaces readers need during migration.

Include these subsections:

1. `exec()` API — TBD until the final Sandbox SDK interface lands. Inline the Containers-style API shape from PR #31565 so readers do not have to leave the migration guide for the expected execution model.
2. Extension API — source from PR #766.
3. Plugin APIs — Code Interpreter, Git, and PTY plugin interfaces, source from PR #767 and related SDK PRs.

This section can start with placeholders, but every placeholder should be explicit and actionable:

```mdx
:::note[Interface pending]
The final Sandbox SDK `exec()` TypeScript interface is still being finalized. This guide will be updated with the exact signature when the `next` branch API lands.
:::
```

Avoid vague stub language such as “published shortly.”

## Agent skill structure

File: `public/sandbox/guides/migrating-to-next/SKILL.md`

The skill should be useful even before the guide is complete. It should include:

- Frontmatter:

```yaml
---
name: sandbox-migrating-to-next
description: Migrate Cloudflare Sandbox SDK projects to @cloudflare/sandbox@next and update deprecated transports, execution APIs, sessions, plugins, and extensions.
---
```

- Install command for `@cloudflare/sandbox@next`.
- Checklist for scanning a codebase:
  - `SANDBOX_TRANSPORT`
  - `transport: "http"`
  - `transport: "websocket"`
  - `execStream(`
  - `startProcess(`
  - `enableDefaultSession`
  - code interpreter APIs
  - Git APIs
  - PTY/terminal APIs
- Migration instructions for each of the five major topics.
- Explicit `TBD` markers for unresolved `exec()` interfaces.
- Links to:
  - `/sandbox/guides/migrating-to-next/`
  - relevant SDK PRs

## Files likely touched

### Required

- `src/content/changelog/sandbox/2026-06-09-deprecating-sandbox-sdk-features.mdx`
- `src/content/docs/sandbox/guides/2026-deprecation.mdx` → rename to `src/content/docs/sandbox/guides/migrating-to-next.mdx`
- `public/sandbox/guides/2026-deprecation/SKILL.md` → rename to `public/sandbox/guides/migrating-to-next/SKILL.md`

### Strongly recommended companion updates

- `src/content/docs/sandbox/configuration/transport.mdx`
- `src/content/docs/sandbox/api/commands.mdx`
- `src/content/docs/sandbox/guides/execute-commands.mdx`
- `src/content/docs/sandbox/guides/streaming-output.mdx`
- `src/content/docs/sandbox/configuration/sandbox-options.mdx`
- `src/content/docs/sandbox/api/sessions.mdx`
- `src/content/docs/sandbox/concepts/sessions.mdx`
- `src/content/docs/sandbox/api/interpreter.mdx`
- `src/content/docs/sandbox/guides/code-execution.mdx`
- `src/content/docs/sandbox/guides/git-workflows.mdx`
- `src/content/docs/sandbox/api/terminal.mdx`
- `src/content/docs/sandbox/concepts/terminal.mdx`

### Search-and-replace cleanup

Replace old links:

- `/sandbox/guides/2026-deprecation/` → `/sandbox/guides/migrating-to-next/`
- `/sandbox/guides/2026-deprecation/SKILL.md` → `/sandbox/guides/migrating-to-next/SKILL.md`
- `https://developers.cloudflare.com/sandbox/guides/2026-deprecation/` → `/sandbox/guides/migrating-to-next/` in docs content; full URLs may remain inside the static skill if they are intended for external agent use.

Remove or revisit mentions of deprecated topics that are no longer in scope:

- `exposePort()` deprecation language
- Desktop-specific removal language, unless rewritten under PTY/plugin extraction
- relative deadline language; use `2026-07-31` / July 31, 2026 instead

## Implementation tasks

### Task 1: Rename files and update route references

**Description:** Rename the guide and static skill to the requested `migrating-to-next` paths, then update all links in the branch.

**Acceptance criteria:**

- [ ] `src/content/docs/sandbox/guides/migrating-to-next.mdx` exists.
- [ ] `src/content/docs/sandbox/guides/2026-deprecation.mdx` is removed.
- [ ] `public/sandbox/guides/migrating-to-next/SKILL.md` exists.
- [ ] `public/sandbox/guides/2026-deprecation/SKILL.md` is removed.
- [ ] No references to `/sandbox/guides/2026-deprecation/` remain.

**Verification:**

- [ ] `grep -R "2026-deprecation" src public`
- [ ] `pnpm run check`

**Dependencies:** None.

**Estimated scope:** Small.

### Task 2: Rewrite the changelog framing

**Description:** Update the changelog so it announces primary-branch deprecations and the `next` branch migration path, not 30-day removal.

**Acceptance criteria:**

- [ ] Includes `npm install @cloudflare/sandbox@next`.
- [ ] States existing projects should migrate as soon as possible.
- [ ] States new projects should use `next` today.
- [ ] Links to `/sandbox/guides/migrating-to-next/`.
- [ ] Removes the 30-day deadline language.
- [ ] Removes `exposePort()` unless separately confirmed in scope.

**Verification:**

- [ ] Review rendered changelog copy.
- [ ] `pnpm run check`.

**Dependencies:** Task 1.

**Estimated scope:** Small.

### Task 3: Build the migration guide foundation

**Description:** Create the full page structure, install section, scope notes, and placeholders for unresolved interfaces.

**Acceptance criteria:**

- [ ] Guide has the three requested major sections.
- [ ] Install section includes `npm install @cloudflare/sandbox@next`.
- [ ] Opening copy tells existing users to migrate and new users to start on `next`.
- [ ] TBDs are explicit and limited to genuinely unresolved API surfaces.

**Verification:**

- [ ] Read the page end to end for flow.
- [ ] `pnpm run check`.

**Dependencies:** Task 1.

**Estimated scope:** Medium.

### Task 4: Fill migration guidance for transports and sessions

**Description:** Write concrete migration steps for removed transports and default-session removal.

**Acceptance criteria:**

- [ ] HTTP/WebSocket migration path is clear.
- [ ] `enableDefaultSession` removal is clear.
- [ ] Explicit-session replacement pattern is shown.
- [ ] Affected reference pages include migration callouts.

**Verification:**

- [ ] Search for `SANDBOX_TRANSPORT`, `websocket`, `enableDefaultSession` and confirm updated contexts.
- [ ] `pnpm run check`.

**Dependencies:** Task 3.

**Estimated scope:** Medium.

### Task 5: Add execution API consolidation guidance

**Description:** Document the intended `exec()` consolidation without inventing finalized signatures.

**Acceptance criteria:**

- [ ] Migration table maps `exec()`, `execStream()`, and `startProcess()` to the new model.
- [ ] Exact code examples are included only if confirmed.
- [ ] TBD placeholders include the Containers-style `exec()` model inline and cite PR #31565 only as source material.
- [ ] Commands/streaming docs have matching deprecation or migration callouts.

**Verification:**

- [ ] Search for `execStream(` and `startProcess(` in Sandbox docs and confirm deprecation context where needed.
- [ ] `pnpm run check`.

**Dependencies:** Task 3.

**Estimated scope:** Medium.

### Task 6: Add plugin and extension API guidance

**Description:** Document the plugin extraction and extension API from SDK PRs #766 and #767.

**Acceptance criteria:**

- [ ] Code Interpreter, Git, and PTY support each have a migration subsection.
- [ ] Extension API section includes confirmed public interfaces or explicit TBD placeholders.
- [ ] Existing interpreter/Git/terminal docs point to the migration guide.
- [ ] Plugin import paths follow the `@cloudflare/sandbox/<plugin>` pattern from PR #767.

**Verification:**

- [ ] Cross-check against PR #766 and #767.
- [ ] `pnpm run check`.

**Dependencies:** Task 3.

**Estimated scope:** Medium.

### Task 7: Update the agent skill

**Description:** Rewrite the static skill so agents can help users scan and migrate projects to `next`.

**Acceptance criteria:**

- [ ] Skill frontmatter uses `sandbox-migrating-to-next`.
- [ ] Skill includes the `@cloudflare/sandbox@next` install command.
- [ ] Skill includes search terms for deprecated APIs.
- [ ] Skill mirrors the five migration topics.
- [ ] Skill links to the new migration guide.

**Verification:**

- [ ] Read the static skill as standalone Markdown.
- [ ] Confirm public path uses uppercase `SKILL.md`.

**Dependencies:** Tasks 3-6.

**Estimated scope:** Small.

### Task 8: Final cleanup and validation

**Description:** Format, validate, and remove unrelated untracked review artifacts.

**Acceptance criteria:**

- [ ] `PLAN.md` is either intentionally committed or removed before PR, depending on maintainer preference.
- [ ] Unrelated untracked files are removed: `review-accuracy.md`, `review-docs.md`, `review-userflow.md`.
- [ ] Formatting passes.
- [ ] Type/content checks pass.

**Verification:**

```sh
pnpm run format
pnpm run check
pnpm run lint
pnpm run format:core:check
```

For a local full validation pass only:

```sh
pnpm run build
```

**Dependencies:** Tasks 1-7.

**Estimated scope:** Small.

## Risks and mitigations

| Risk                                                  | Impact | Mitigation                                                                                                               |
| ----------------------------------------------------- | ------ | ------------------------------------------------------------------------------------------------------------------------ |
| `next` APIs are still changing while docs are written | High   | Label unsettled interfaces as TBD and link to source PRs. Do not publish guessed signatures.                             |
| Existing docs contradict the migration guide          | High   | Add callouts to affected API/reference pages in the same PR.                                                             |
| Static skill path casing breaks links                 | Medium | Use exactly `public/sandbox/guides/migrating-to-next/SKILL.md` and link to `/sandbox/guides/migrating-to-next/SKILL.md`. |
| Plugin package/import names are not final             | Medium | Keep placeholders until PR #766/#767 confirms names.                                                                     |
| Changelog overstates removal timing                   | Medium | Avoid relative deadline language. Frame as deprecation on primary branch and migration to `next`.                        |

## Open questions

- What is the final public `sandbox.exec()` TypeScript interface on `next`?
- What are the final plugin identifiers for the `@cloudflare/sandbox/<plugin>` import paths for Code Interpreter, Git, and PTY?
- Should `exposePort()` remain out of scope for this migration guide?
- Should the old `/sandbox/guides/2026-deprecation/` route receive a redirect, or is it safe to remove because it has not shipped?
- Should the static skill use root-relative docs links or full `https://developers.cloudflare.com/...` links for external agent portability?
