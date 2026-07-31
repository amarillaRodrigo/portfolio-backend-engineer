# PostulaDo — Flujo de Postulaciones con Análisis Asistido por LLM

**Rol.** Ingeniero backend (autor único).
**Stack.** NestJS · TypeScript · Prisma · PostgreSQL · Redis · BullMQ · SGLang (XGrammar + RadixAttention) · JWT · Passport.
**Estado.** Implementación de referencia pública, código en disco.

---

## Problema

Postularse a una oferta laboral es trabajo repetitivo: reformatear el CV, redactar una carta de presentación, resaltar la experiencia relevante, adaptar el tono. Las partes mecánicas son tediosas, pero la parte que realmente requiere oficio — elegir qué experiencia resaltar — necesita estructura.

La mayoría de las herramientas de "postulación con IA" esconden el modelo detrás de una caja negra y producen salidas genéricas. El reto fue diseñar un backend que permita al usuario inspeccionar lo que el modelo extrajo, regenerar cada sección en forma independiente, y mantener el control total sobre los datos subyacentes.

## Enfoque

Una API NestJS con arquitectura en capas:

- **Capa de auth** — JWT con Passport, guards por rol, ownership checks en cada recurso por usuario.
- **Persistencia** — Prisma sobre PostgreSQL. El esquema es la única fuente de verdad; la API lo refleja.
- **Trabajo asíncrono** — Redis + BullMQ para todo lo que no necesita estar en el request path (llamadas al LLM, scraping, generación de reportes).
- **Capa LLM** — SGLang (XGrammar + RadixAttention) para análisis estructurado de ofertas. Los outputs se validan contra un esquema antes de llegar al usuario.
- **Scraping** — Cheerio + OpenAI SDK para levantar texto plano de URLs y alimentar el pipeline de análisis.

Un interceptor global elimina los campos de contraseña de toda respuesta pública — seguridad en profundidad en la capa de serialización, no responsabilidad de cada controller.

## Arquitectura

```
Cliente HTTP
    │
    ▼
┌─────────────────────────────────────────────┐
│ API NestJS                                 │
│  ─ AuthModule (JWT + Passport + Roles)      │
│  ─ UsersModule (per-user, ownership guards) │
│  ─ ApplicationsModule (CRUD)                │
│  ─ AnalysisModule                           │
│     ├─ BullMQ producer (URL → job)          │
│     └─ BullMQ worker                        │
│         ├─ Scraping con Cheerio             │
│         ├─ Extracción estructurada SGLang   │
│         └─ Cover letter + CV + focus report│
└─────────────────────────────────────────────┘
    │                    │
    ▼                    ▼
PostgreSQL           Redis + BullMQ
   (Prisma)              │
                         ▼
                 Inferencia SGLang
```

## Decisiones clave

- **Extracción estructurada, no prosa libre.** El LLM devuelve un payload tipado (tecnologías, responsabilidades, años de experiencia, tono de la empresa) que se valida contra un esquema antes de consumirse. Entrada basura se rechaza, no se muestra.
- **Ownership guards como preocupación de primera clase.** Cada endpoint que toca un recurso por usuario verifica que el caller sea el dueño. Los roles son un eje separado — admin y owner no son el mismo check.
- **BullMQ para todo lo async, incluso lo chico.** Mantener el request path delgado hace que la API responda bien bajo carga y que el worker escale en forma independiente.
- **Ninguna contraseña en ninguna respuesta.** Un único interceptor global borra el campo de cada objeto saliente. La corrección está en el wire, no en cada controller.
- **Validación schema-first.** class-validator + class-transformer en cada DTO. El esquema de DB es el contrato; el DTO es el contrato de la API; ambos deben concordar.

## Resultados

- Los usuarios pueden pegar una URL y obtener un desglose estructurado de la oferta en una forma tipada, no un párrafo.
- La carta de presentación, la optimización del CV y el reporte de foco se regeneran en forma independiente — no hay que re-ejecutar el pipeline completo.
- Los tests de auth y ownership cubren los paths por usuario (ver `test/`).
- Separación limpia entre código de request-time y worker-time; razonar sobre la latencia es directo.

## Enlaces

- Repositorio: `D:\programacion\PostulaDo\postulado`
- Notas de arquitectura y testing: `docs/`
- Esquema Prisma: `prisma/schema.prisma`
- Entrypoint de la API: `src/main.ts`
