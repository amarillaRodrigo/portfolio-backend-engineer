# Content Map

The portfolio maps to four routes. Each route lists its sections, the copy intent, and the assets needed. All typography, color, and spacing decisions live in `identity-kit.md` and must be applied verbatim.

## Routes

| Route | Purpose | Sections |
|---|---|---|
| `/` | Hero — first impression | Hero |
| `/projects` | Selected works (Bento grid) | Hero + Bento grid |
| `/about` | Who I am | About |
| `/contact` | How to reach me | Contact |

The Hero (`/`) and the in-route hero at `/projects` share the same headline component but use different copy.

---

## `/` — Hero

**Purpose.** Name + one-line positioning. Visible in < 2 seconds.

| Section | Copy intent | Assets |
|---|---|---|
| Eyebrow | Status note ("En construcción" / "Under construction") | — |
| Headline H1 | Name: `Rodrigo` | — |
| Tagline | One sentence: "Backend engineer focused on distributed systems." | — |
| CTA | Link to `/projects` | — |

Spacing: `24px` between eyebrow and headline; `48px` between headline and tagline; `96px` vertical breathing room to the next section.

---

## `/projects` — Selected Works

**Purpose.** Two real projects, current focus.

| Section | Copy intent | Assets |
|---|---|---|
| Hero | "Selected Works." + 2-sentence positioning | — |
| Featured card | HieloPedido — link to case study | `hielopedido-proof.png` (architecture or idempotency handler diagram) |
| Standard card | PostulaDo — link to case study | `postulado-proof.png` (API call tree or schema) |

**Bento layout** (desktop):
- Featured card spans 12/12 columns.
- Each standard card spans 6/12 columns.
- Spacing: `gutter` 24px between cards.

**Card anatomy** (per project):
- Tags row (Java 25 / Spring Boot, NestJS / Prisma, etc.) — uppercase `label-xs` style.
- Project title (H2 / H3 per card size).
- 1-sentence description emphasizing the engineering choice, not the feature.
- Tech stack chips.
- "VIEW CASE STUDY" link → routes to the case study page.

---

## `/about` — Who I Am

**Purpose.** Personal context. Five lines, no fluff.

| Section | Copy intent | Assets |
|---|---|---|
| Eyebrow | "About" | — |
| Headline | "Rodrigo" | — |
| Lead | Age 24, Argentina, Téc. Sup. en Desarrollo de Software, Lic. en Ciencia de Datos (in progress) | — |
| What I do | Backend architectures · comprehensive testing · CI/CD · anonymous feedback platform for local businesses · AI-driven calorie-tracking app · boxing/combat sports | — |
| Secondary | Currently interested in: distributed systems, idempotency, LLM-assisted workflows | — |

No photo on this page. Avatar optional in the hero only.

---

## `/contact` — Contact

**Purpose.** Three ways to reach me, no forms.

| Section | Copy intent | Assets |
|---|---|---|
| Eyebrow | "Contact" | — |
| Headline | "Let's talk." | — |
| Email | Work email | — |
| LinkedIn | Profile URL | — |
| GitHub | Profile URL | — |

Sticky CTA button visible across all routes, links to `/contact`.

---

## Assets (owned by `portfolio/assets/images/`)

| File | Required for | Status |
|---|---|---|
| `avatar.jpg` | Hero (optional) | exists |
| `favicon.png` | `<head>` | exists |
| `hielopedido-proof.png` | Projects card | pending |
| `postulado-proof.png` | Projects card | pending |

---

## Cross-references

- `identity-kit.md` — typography, color, spacing tokens.
- `case-study-hielopedido.md` / `.en.md` — Featured card copy.
- `case-study-postulado.md` / `.en.md` — Standard card copy.

---

## Dropped from original mockup

- **Multi-Container Ops** — no code on disk, removed from `/projects` and from this content map.

---

## Build phases (suggested)

1. Routes (`/`, `/projects`, `/about`, `/contact`) with placeholder copy.
2. Components (Navbar, Hero, BentoCard, Footer).
3. Wire case studies to detail pages (`/projects/hielopedido`, `/projects/postulado`).
4. Add assets when proofs are captured.
5. SEO + Open Graph metadata per route.
