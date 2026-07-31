# Content Map

El portfolio se mapea a cuatro rutas. Cada ruta lista sus secciones, la intención del copy y los assets necesarios. Todas las decisiones de tipografía, color y espaciado viven en `identity-kit.md` y deben aplicarse en forma literal.

## Rutas

| Ruta | Propósito | Secciones |
|---|---|---|
| `/` | Hero — primera impresión | Hero |
| `/projects` | Trabajos seleccionados (grilla Bento) | Hero + grilla Bento |
| `/about` | Quién soy | About |
| `/contact` | Cómo contactarme | Contact |

El Hero de `/` y el hero interno de `/projects` comparten el mismo componente de headline pero usan copy distinto.

---

## `/` — Hero

**Propósito.** Nombre + posicionamiento en una línea. Visible en < 2 segundos.

| Sección | Intención de copy | Assets |
|---|---|---|
| Eyebrow | Nota de estado ("En construcción") | — |
| Headline H1 | Nombre: `Rodrigo` | — |
| Tagline | Una frase: "Backend engineer focused on distributed systems." | — |
| CTA | Link a `/projects` | — |

Spacing: `24px` entre eyebrow y headline; `48px` entre headline y tagline; `96px` de respiro vertical hasta la siguiente sección.

---

## `/projects` — Trabajos Seleccionados

**Propósito.** Dos proyectos reales, foco actual.

| Sección | Intención de copy | Assets |
|---|---|---|
| Hero | "Selected Works." + 2 frases de posicionamiento | — |
| Card featured | HieloPedido — link al case study | `hielopedido-proof.png` (diagrama de arquitectura o del manejador de idempotencia) |
| Card estándar | PostulaDo — link al case study | `postulado-proof.png` (árbol de llamadas de la API o esquema) |

**Layout Bento** (desktop):
- La card featured ocupa 12/12 columnas.
- Cada card estándar ocupa 6/12 columnas.
- Spacing: `gutter` de 24px entre cards.

**Anatomía de card** (por proyecto):
- Fila de tags (Java 25 / Spring Boot, NestJS / Prisma, etc.) — mayúsculas, estilo `label-xs`.
- Título del proyecto (H2 / H3 según el tamaño de la card).
- Una frase de descripción que enfatice la decisión de ingeniería, no la feature.
- Chips del stack técnico.
- Link "VIEW CASE STUDY" → lleva a la página del case study.

---

## `/about` — Quién Soy

**Propósito.** Contexto personal. Cinco líneas, sin relleno.

| Sección | Intención de copy | Assets |
|---|---|---|
| Eyebrow | "About" | — |
| Headline | "Rodrigo" | — |
| Lead | 24 años, Argentina, Téc. Sup. en Desarrollo de Software, Lic. en Ciencia de Datos (en curso) | — |
| What I do | Arquitecturas backend · testing comprehensivo · CI/CD · plataforma de feedback anónimo para comercios locales · app de tracking calórico con IA · boxeo / deportes de combate | — |
| Secondary | Intereses actuales: sistemas distribuidos, idempotencia, workflows asistidos por LLM | — |

Sin foto en esta página. Avatar opcional solo en el hero.

---

## `/contact` — Contacto

**Propósito.** Tres formas de contacto, sin formularios.

| Sección | Intención de copy | Assets |
|---|---|---|
| Eyebrow | "Contact" | — |
| Headline | "Let's talk." | — |
| Email | Email de trabajo | — |
| LinkedIn | URL del perfil | — |
| GitHub | URL del perfil | — |

Botón CTA sticky visible en todas las rutas, link a `/contact`.

---

## Assets (en `portfolio/assets/images/`)

| Archivo | Requerido para | Estado |
|---|---|---|
| `avatar.jpg` | Hero (opcional) | existe |
| `favicon.png` | `<head>` | existe |
| `hielopedido-proof.png` | Card de Projects | pendiente |
| `postulado-proof.png` | Card de Projects | pendiente |

---

## Cross-references

- `identity-kit.md` — tokens de tipografía, color, espaciado.
- `case-study-hielopedido.md` / `.en.md` — copy de la card featured.
- `case-study-postulado.md` / `.en.md` — copy de la card estándar.

---

## Eliminado del mockup original

- **Multi-Container Ops** — sin código en disco, removido de `/projects` y de este content map.

---

## Fases de build (sugeridas)

1. Rutas (`/`, `/projects`, `/about`, `/contact`) con copy placeholder.
2. Componentes (Navbar, Hero, BentoCard, Footer).
3. Vincular los case studies a las páginas de detalle (`/projects/hielopedido`, `/projects/postulado`).
4. Agregar los assets cuando se capturen las pruebas.
5. SEO + Open Graph metadata por ruta.
