# PostulaDo — Job Application Workflow with LLM-Assisted Analysis

**Role.** Backend engineer (sole author).
**Stack.** NestJS · TypeScript · Prisma · PostgreSQL · Redis · BullMQ · SGLang (XGrammar + RadixAttention) · JWT · Passport.
**Status.** Public reference implementation, code on disk.

---

## Problem

Applying to a job posting is repetitive work: reformat the CV, draft a cover letter, highlight relevant experience, adapt the tone. The boring parts of this process are mechanical, but the part that actually requires craft — picking which experience to surface — needs structure.

Most "AI apply" tools hide the model behind a black box and produce generic output. The challenge was to design a backend that lets the user inspect what the model extracted, regenerate any section independently, and keep full ownership of the underlying data.

## Approach

A NestJS API with a layered architecture:

- **Auth layer** — JWT with Passport, role-based guards, ownership checks on every per-user resource.
- **Persistence** — Prisma over PostgreSQL. Schema is the single source of truth; the API reflects it.
- **Async work** — Redis + BullMQ for everything that doesn't need to be in the request path (LLM calls, scraping, report generation).
- **LLM layer** — SGLang (XGrammar + RadixAttention) for structured job-offer analysis. Outputs are validated against a schema before the user ever sees them.
- **Scraping** — Cheerio + OpenAI SDK to lift plain text from URLs and feed it into the analysis pipeline.

A global interceptor strips password fields from every public response — defense in depth at the serialization layer, not a per-controller responsibility.

## Architecture

```
HTTP client
    │
    ▼
┌─────────────────────────────────────────────┐
│ NestJS API                                  │
│  ─ AuthModule (JWT + Passport + Roles)      │
│  ─ UsersModule (per-user, ownership guards) │
│  ─ ApplicationsModule (CRUD)                │
│  ─ AnalysisModule                           │
│     ├─ BullMQ producer (URL → job)          │
│     └─ BullMQ worker                        │
│         ├─ Cheerio scrape                   │
│         ├─ SGLang structured extraction     │
│         └─ Cover letter + CV + focus report│
└─────────────────────────────────────────────┘
    │                    │
    ▼                    ▼
PostgreSQL           Redis + BullMQ
   (Prisma)              │
                         ▼
                   SGLang inference
```

## Key Decisions

- **Structured extraction, not free-form prose.** The LLM returns a typed payload (technologies, responsibilities, years of experience, company tone) that is validated against a schema before being consumed. Garbage-in is rejected, not displayed.
- **Ownership guards as a first-class concern.** Every endpoint that touches a per-user resource verifies the caller owns it. Roles are a separate axis — admin and owner are not the same check.
- **BullMQ for everything async, even the small stuff.** Keeping the request path thin means the API stays responsive under load and the worker can scale independently.
- **No passwords in any response.** A single global interceptor deletes the field from every outgoing object. The fix is at the wire, not at every controller.
- **Schema-first validation.** class-validator + class-transformer on every DTO. The DB schema is the contract; the DTO is the API contract; both must agree.

## Results

- Users can paste a URL and get a structured breakdown of the offer in a typed shape, not a paragraph.
- Cover letter, CV optimization, and focus report are regenerated independently — no full re-run.
- Auth and ownership testing cover the per-user paths (see `test/`).
- Clean separation between request-time and worker-time code; reasoning about latency is straightforward.

## Links

- Repository: `D:\programacion\PostulaDo\postulado`
- Architecture & testing notes: `docs/`
- Prisma schema: `prisma/schema.prisma`
- API entrypoint: `src/main.ts`
