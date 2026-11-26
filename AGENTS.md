# AGENTS.md — Contexto y Reglas del Proyecto `spring-app`

Archivo de referencia para sesiones de desarrollo y agentes en este repositorio Spring Boot.

---

## 1. Stack Técnico y Versiones Exactas

| Componente | Valor / Versión |
| --- | --- |
| **Proyecto (`groupId:artifactId`)** | `com.ivan.course.antigravity.springboot:spring-app` (`0.0.1-SNAPSHOT`) |
| **Empaquetado** | `jar` |
| **Java / JDK** | **Java 25** (`<java.version>25</java.version>`, entorno local: OpenJDK Temurin `25.0.5.1` LTS) |
| **Spring Boot** | **4.2.0** (`spring-boot-starter-parent:4.2.0`, GA estable) |
| **Build tool** | **Maven** (local: Apache Maven `3.10.2`; Maven Wrapper `3.3.4` → Maven `3.10.1`) |
| **Configuración** | `src/main/resources/application.properties` (`spring.application.name=spring-app`) |

### Dependencias Actuales (`pom.xml`)
- `org.springframework.boot:spring-boot-starter-webmvc` — Spring MVC y servidor embebido Apache Tomcat.
- `org.springframework.boot:spring-boot-devtools` (`scope: runtime`, `optional: true`) — Reinicio rápido y utilidades de desarrollo.
- `org.springframework.boot:spring-boot-starter-webmvc-test` (`scope: test`) — Soporte de pruebas unitarias y de integración para Spring MVC / Spring Boot.
- Plugin de build: `org.springframework.boot:spring-boot-maven-plugin`.

---

## 2. Estructura de Paquetes y Convenciones de Nombres

- **Paquete base**: `com.ivan.course.antigravity.springboot`
- **Clase principal**: `com.ivan.course.antigravity.springboot.SpringAppApplication` (`@SpringBootApplication`)
- **Prueba base de contexto**: `com.ivan.course.antigravity.springboot.SpringAppApplicationTests` (`@SpringBootTest`)
- **Recursos**:
  - `src/main/resources/application.properties`
  - `src/main/resources/static/`
  - `src/main/resources/templates/`

### Organización de Paquetes Recomendada
Organizar por capas dentro del paquete base (o por submódulo de dominio si el proyecto crece: `com.ivan.course.antigravity.springboot.<dominio>.*`):

```text
com.ivan.course.antigravity.springboot
├── SpringAppApplication.java
├── config/          # Clases @Configuration y propiedades tipadas (@ConfigurationProperties)
├── controller/      # Controladores REST (@RestController)
├── dto/             # Records de entrada/salida (*Request, *Response)
├── exception/       # Excepciones de dominio y GlobalExceptionHandler (@RestControllerAdvice)
├── mapper/          # Conversión entre modelos/entidades y DTOs
├── model/           # Modelos de dominio / entidades
├── repository/      # Acceso a datos (@Repository)
└── service/         # Lógica de negocio (@Service)
```

### Convenciones de Nombres
- **Clases y records (`PascalCase`)**: `UserController`, `UserService`, `UserRepository`, `CreateUserRequest`, `UserResponse`, `ResourceNotFoundException`, `GlobalExceptionHandler`.
- **Métodos y variables (`camelCase`)**; **constantes (`UPPER_SNAKE_CASE`)**.
- **Endpoints REST**: minúsculas, sustantivos en plural y kebab-case (ej. `/api/v1/users`, `/api/v1/order-items/{id}`).
- **Pruebas**: espejo del paquete en `src/test/java`, sufijo `*Test` (unitarias) o `*Tests` / `*IT` (integración).

---

## 3. Reglas de Desarrollo y Buenas Prácticas (Spring Boot + Java 25)

1. **Inyección de dependencias por constructor**:
   - Usar siempre campos `private final` inyectados por constructor (sin `@Autowired` en campos; con un único constructor Spring inyecta automáticamente).
2. **Separación de capas y uso obligatorio de DTOs**:
   - Los controladores solo manejan transporte HTTP (rutas, parámetros, validación de entrada y códigos de estado) y delegan toda la lógica de negocio a `service/`.
   - Usar Java `record` inmutables en `dto/` para contratos de entrada (`*Request`) y salida (`*Response`). Nunca exponer entidades de persistencia ni modelos internos directamente en las respuestas REST.
3. **Manejo global de excepciones**:
   - Centralizar los errores en un `GlobalExceptionHandler` anotado con `@RestControllerAdvice` y `@ExceptionHandler`.
   - Devolver respuestas de error consistentes siguiendo RFC 9457 usando `org.springframework.http.ProblemDetail` (evitar filtrar stack traces o detalles internos al cliente).
4. **Validaciones**:
   - Validar contratos de entrada en los DTOs con Jakarta Bean Validation (`@Valid` en el controlador y anotaciones como `@NotBlank`, `@NotNull`, `@Email`, `@Positive`; añadir `spring-boot-starter-validation` en `pom.xml` cuando se incorporen).
   - Mantener las reglas de negocio e invariantes en la capa `service/`.
5. **Respuestas HTTP estándar**:
   - Usar `ResponseEntity<T>` con códigos semánticos:
     - `200 OK` (consultas y actualizaciones con cuerpo)
     - `201 Created` (creación, incluyendo cabecera `Location` cuando aplique)
     - `204 No Content` (eliminaciones o actualizaciones sin cuerpo)
     - `400 Bad Request` (errores de validación / formato)
     - `404 Not Found` (recurso inexistente)
     - `409 Conflict` (duplicados o conflictos de estado)
     - `500 Internal Server Error` (errores inesperados)
6. **Configuración y seguridad**:
   - Mantener la configuración en `application.properties` (y perfiles `application-{profile}.properties` si aplica).
   - No hardcodear credenciales, tokens ni URLs sensibles; inyectarlos vía variables de entorno (`${ENV_VAR}`).
7. **Java 25 y pruebas**:
   - Aprovechar `record`, pattern matching, `switch` expressions y `var` cuando mejoren la legibilidad.
   - Acompañar nuevas funcionalidades con pruebas automatizadas (JUnit 5, `@WebMvcTest` para controladores y `@SpringBootTest` para integración).

---

## 4. Comandos Útiles del Proyecto

En Windows PowerShell usa `.\mvnw.cmd` (en Linux/macOS usa `./mvnw`).

> **Nota sobre `.mvn/`**: Como `.mvn/` está excluido en `.gitignore`, en entornos recién clonados sin el directorio `.mvn/` utiliza `mvn` directamente (Maven instalado en el sistema).

```powershell
# Compilar el proyecto
.\mvnw.cmd clean compile
# o: mvn clean compile

# Ejecutar la aplicación en modo desarrollo (con DevTools activo)
.\mvnw.cmd spring-boot:run
# o: mvn spring-boot:run

# Ejecutar todas las pruebas unitarias y de contexto
.\mvnw.cmd test
# o: mvn test

# Ejecutar una clase de prueba específica
.\mvnw.cmd test -Dtest=SpringAppApplicationTests

# Empaquetar el JAR ejecutable (en target/)
.\mvnw.cmd clean package

# Empaquetar omitiendo pruebas (solo si ya se validaron)
.\mvnw.cmd clean package -DskipTests
```

---

## 5. Convenciones de Git

- **Rama principal**: `main`
- **Remoto `origin`**: `https://github.com/ivanrojasdev/spring-app.git`

### Conventional Commits
Todos los commits deben seguir el formato:

```text
<type>(<scope opcional>): <descripción breve en imperativo>
```

Tipos habituales:
- `feat:` nueva funcionalidad
- `fix:` corrección de errores
- `docs:` documentación (`README.md`, `AGENTS.md`, comentarios)
- `refactor:` refactorización sin cambios de comportamiento
- `test:` creación o ajuste de pruebas
- `build:` cambios en `pom.xml`, dependencias o empaquetado
- `chore:` tareas de mantenimiento, configuración de Git/CI

Ejemplo: `docs: add AGENTS.md with project context and development guidelines`

### Archivos Excluidos (`.gitignore`)
Nunca versionar artefactos generados, configuración local de IDEs ni secretos. `.gitignore` excluye explícitamente:
- **Build y wrapper local**: `target/`, `.mvn/`, `HELP.md`
- **IDEs**: `.idea`, `*.iml`, `*.iws`, `*.ipr`, `.vscode/`, `.classpath`, `.project`, `.settings`, `.apt_generated`, `.factorypath`, `.springBeans`, `.sts4-cache`, `build/`
- **Archivos sensibles**: `.env`, `.env.*`, `*.env`
