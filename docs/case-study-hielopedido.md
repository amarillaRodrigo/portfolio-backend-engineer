# HieloPedido — Gestión de Pedidos B2B Offline-First

**Rol.** Ingeniero backend (autor único de los servicios).
**Stack.** Java 25 · Spring Boot · PostgreSQL 15 · Docker · JWT (HMAC-SHA256).
**Estado.** Implementación de referencia lista para producción, código en disco.

---

## Problema

Los distribuidores mayoristas de hielo operan desde cámaras frigoríficas y camiones de reparto — entornos donde la conectividad móvil es poco confiable o inexistente. Los preventistas necesitan registrar pedidos en la calle y que lleguen al sistema central sin reconciliación manual al recuperar señal.

Un backend REST naïve rechazaría envíos duplicados en el reintento, perdería pedidos cuando la red cae a mitad de sincronización, y obligaría al cliente a implementar colas offline frágiles. El reto fue diseñar un backend que aceptara el mismo envío N veces de forma segura, preservara el orden tras la reconexión, y usara tokens que sobrevivan a una ventana offline larga sin convertirse en una vulnerabilidad.

## Enfoque

Dividí el sistema en dos microservicios Spring Boot detrás de un contrato claro:

- **sync-service** — punto de entrada. Valida JWT, aplica idempotencia, rota tokens de refresco y reenvía los payloads aceptados al servicio de pedidos.
- **order-service** — dueño de la base de datos transaccional. Gestiona pedidos, validación de stock y catálogo de productos.

El contrato cliente-servidor se construye alrededor de **idempotency keys** llevadas en cada escritura. El cliente genera la clave una vez por pedido lógico y reintenta con seguridad hasta que el servidor acusa recibo. El **patrón Outbox** propaga los pedidos confirmados a los consumidores downstream sin perder eventos ante un crash.

## Arquitectura

```
Cliente móvil Flutter
        │  (HTTPS, JWT bearer)
        ▼
┌──────────────────────────────┐
│ sync-service   (puerto 8080/81)│
│  ─ Validación JWT & refresh  │
│  ─ Middleware de idempotencia│
│  ─ Publicador Outbox         │
└──────────────────────────────┘
        │  (interno)
        ▼
┌──────────────────────────────┐
│ order-service   (puerto 8082)│
│  ─ Pedidos + stock + catálogo│
│  ─ PostgreSQL 15 (Docker)    │
└──────────────────────────────┘
```

## Decisiones clave

- **Rotación de refresh-token en cada uso.** Un refresh token filtrado es detectable en la siguiente request porque el anterior queda invalidado. Las ventanas offline no amplían la superficie de ataque más allá del TTL configurado.
- **Las idempotency keys viven en el borde de la API, no en la base de datos.** El sync service es dueño de la deduplicación; el order service confía en que lo que recibe es lógicamente único. Esto mantiene la base transaccional simple.
- **Tabla Outbox dentro de cada servicio.** Sin broker externo. Los eventos se escriben en la misma transacción que el cambio de estado de negocio, y un publicador los drena. Atomicidad por encima de throughput.
- **Dos bases de datos, no dos esquemas.** `sync_db` y `order_db` aisladas por Docker. Cada servicio es dueño de su esquema de punta a punta. Estrictamente enforced en el contrato OpenAPI.
- **JWT HMAC-SHA256 con secreto mínimo de 32 bytes.** Ambos servicios se niegan a arrancar si `JWT_SECRET` falta o es corto — una guarda fail-fast contra un footgun conocido de despliegue.

## Resultados

- Pérdida de conectividad de cualquier duración es recuperable sin intervención manual.
- Pedidos duplicados son imposibles en el borde de la API.
- La rotación JWT de refresh está verificada end-to-end con tests de integración (ver `docs/testing-integration.md`).
- Extensible a nuevas apps cliente (web admin, integraciones de terceros) sin cambiar el contrato de sync.

## Enlaces

- Repositorio: `D:\programacion\ice`
- Spec: `PRD.md`, `docs/ARCHITECTURE.md`, `docs/API_REFERENCE.md`
- OpenAPI público: `docs/openapi.json`
