---
name: skill_backend
description: >
  Validación de buenas prácticas de desarrollo Java/Spring Boot, arquitectura limpia,
  código limpio, documentación y estándares del proyecto ParKing.
  Úsala al revisar PRs, generar código o auditar el backend.
---

## 1. Buenas prácticas de desarrollo — Java + Spring Boot

### 1.1 Java

- **Nombres**: clases en `PascalCase`, métodos/variables en `camelCase`, constantes en `UPPER_SNAKE_CASE`.
- **Sin números mágicos**: toda constante literal debe tener nombre (excepto 0, 1, -1, HTTP codes).
- **Líneas**: máximo 150 caracteres.
- **Sin System.out.println**: usar logging (SLF4J/Logback).
- **Métodos cortos**: un método = una responsabilidad. Máximo ~30 líneas.
- **Exceptions**: usar excepciones específicas, nunca `Exception` genérica. Lanzar `IllegalArgumentException`, `ResourceNotFoundException`, etc.
- **Optional**: usar `Optional<T>` para retornos que pueden ser vacíos, nunca `null`.
- **Records para DTOs**: usar `record` de Java para DTOs inmutables cuando sea posible.

### 1.2 Spring Boot

- **Inyección**: usar constructor injection (no `@Autowired` en campos).
- **Controllers**: anotaciones REST (`@RestController`, `@RequestMapping`). Sin lógica de negocio.
- **Services**: anotar con `@Service` y `@Transactional` en métodos de escritura.
- **Repositories**: extender `JpaRepository` o interfaces de Spring Data.
- **Properties**: valores de configuración en `application.properties` / `application.yml`, no hardcodeados.
- **Validación**: usar `@Valid` + Bean Validation (`@NotBlank`, `@Size`, `@Pattern`, etc.) en los DTOs de entrada.

### 1.3 Testing

- Nombrar tests con patrón `methodName_expectedBehavior` (ej. `login_withValidCredentials_returnsToken`).
- Usar Mockito para aislar la capa bajo test.
- No usar `@SpringBootTest` a menos que sea necesario (preferir slices como `@WebMvcTest`, `@DataJpaTest`).

---

## 2. Arquitectura del proyecto — ParKing

### 2.1 Estilo arquitectónico

**Layered Architecture (Arquitectura en Capas)** — Monolítica REST API con Spring Boot 4.0.6.

### 2.2 Capas y dependencias

```
Controller  →  Service  →  Repository  →  JPA / PostgreSQL
    │              │              │
    ▼              ▼              ▼
   DTOs        Entidades      Spring Data
```

| Capa | Paquete | Depende de | Responsabilidad |
|---|---|---|---|
| **Controller** | `controller/` | `service/`, `dto/` | Endpoints REST, HTTP request/response |
| **Service** | `service/` | `repository/`, `entity/` | Lógica de negocio, validación, transacciones |
| **Repository** | `repository/` | `entity/` | Acceso a base de datos vía Spring Data JPA |
| **Entity** | `entity/` | — | Modelo de dominio → tablas de BD |
| **DTO** | `dto/` | — | Objetos de transferencia request/response |
| **Security** | `security/` | — | Filtro JWT, utilería de tokens |
| **Config** | `config/` | `security/` | Configuración de Spring Security y filtros |

### 2.3 Dependencias externas

| Dependencia | Propósito |
|---|---|
| `spring-boot-starter-data-jpa` | ORM con Hibernate + JPA |
| `spring-boot-starter-webmvc` | Servidor REST embebido (Tomcat) |
| `spring-boot-starter-security` | Autenticación y autorización |
| `spring-boot-starter-validation` | Bean Validation (`@Valid`) |
| `postgresql` | Driver JDBC PostgreSQL 16 |
| `jjwt-api / jjwt-impl / jjwt-jackson` | Creación/validación de JWT |
| `maven-checkstyle-plugin` | Reglas de estilo de código |

### 2.4 Flujo de datos

```
Cliente HTTP
    → Controller (@RestController)
        → Service (@Service, @Transactional)
            → Repository (JpaRepository)
                → Entity (@Entity) → PostgreSQL
    ← DTO (respuesta)
```

---

## 3. MUST y SHOULD en revisiones de código

### 🔴 MUST HAVE (obligatorio)

- [ ] **No hay lógica de negocio en Controllers**. Los controllers solo delegan a Services y retornan respuestas.
- [ ] **Las queries JPA se hacen en Repositories**, nunca en Services o Controllers.
- [ ] **Los métodos tienen una sola responsabilidad**. Si hace más de una cosa, refactorizar.
- [ ] **No hay números mágicos (hardcodeados) a excepción de los códigos http y números comunes.**
- [ ] **No hay `System.out.println`**. Usar logger (`private static final Logger log = LoggerFactory.getLogger(...)`).
- [ ] **Los endpoints sensibles tienen autorización** configurada en `SecurityConfig.java`.
- [ ] **Los tests unitarios existen** para toda nueva lógica de negocio en Services.
- [ ] **Los DTOs de entrada tienen validación** con anotaciones `jakarta.validation`.
- [ ] **No hay warnings de Checkstyle** (`mvn checkstyle:check` debe pasar).
- [ ] **Las contraseñas se almacenan hasheadas** (SHA-256), nunca en texto plano.
- [ ] **No se exponen entidades JPA directamente** en respuestas — usar DTOs.

### 🟡 SHOULD HAVE (recomendado)

- [ ] **Uso de `Optional`** para retornos de repositorio que pueden ser vacíos.
- [ ] **Constructor injection** en lugar de `@Autowired` en campos.
- [ ] **Métodos con menos de 30 líneas** (ideal < 15).
- [ ] **Nombres de variables y métodos descriptivos** (evitar `a`, `tmp`, `data`).
- [ ] **Manejo de errores consistente**: usar `ResponseEntity` con códigos HTTP apropiados.
- [ ] **Tests con nombres descriptivos** siguiendo el patrón `methodName_expectedBehavior`.
- [ ] **No hay imports no utilizados** ni comentarios de código muerto.
- [ ] **Los commits son atómicos** y con mensajes descriptivos (no "fix", "update").

---

## 4. Estándares de documentación

### 4.1 API

- Documentar endpoints con `@Operation` (Swagger/OpenAPI) si se agrega dependencia.
- Los DTOs deben reflejar exactamente la estructura de la request/response.

### 4.2 Base de datos

- Los scripts SQL en `scripts/` deben reflejar el estado actual del esquema.
- Toda nueva tabla debe tener su `CREATE TABLE` en `init.sql`.
- Los datos de prueba van en `data_demo.sql` con comentarios del caso de uso.

### 4.3 Código

- `checkstyle.xml` es la fuente de verdad para estilo. No deshabilitar reglas sin justificación.
- No dejar código comentado. Si algo no se usa, eliminarlo.

---

## 5. Reglas de código limpio (vistas en clase)

1. **DRY** (Don't Repeat Yourself): si un bloque se repite 2+ veces, extraer a método.
2. **KISS** (Keep It Simple): priorizar solución simple sobre ingeniería excesiva.
3. **YAGNI** (You Ain't Gonna Need It): no agregar abstracciones "por si acaso".
4. **Principio de mínimo asombro**: el código debe hacer lo que su nombre indica.
5. **Separación de concerns**: cada capa se ocupa de lo suyo (Controller ≠ Service ≠ Repository).
6. **Fail fast**: validar entradas al inicio del método, no 50 líneas después.
7. **Composición sobre herencia**: preferir composición a herencia de clases.
8. **Nombres significativos**: `parkingLotService` > `service`, `calculateTotalCost()` > `calc()`.
9. **Funciones puras**: evitar efectos secundarios en métodos que prometen solo calcular/transformar.
10. **Manejo de nulls**: evitar retornar `null` — usar `Optional`, excepciones, o patrones como Null Object.
