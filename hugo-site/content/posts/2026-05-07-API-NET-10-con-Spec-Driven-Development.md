---
title: "Construyendo una API .NET 10 con Spec-Driven Development, OpenSpec y Claude Code"
slug: "API-NET-10-con-Spec-Driven-Development"
date: 2026-05-07
tags: ["netcore", "sdd", "claude"]
---

> **Stack:** .NET 10 · C# 13 · SQLite · EF Core · xUnit · Swashbuckle · Claude Code · OpenSpec v1.3.1 · JetBrains Rider  
> **Repositorio:** [re-al-7/SSI.SSD.TodoApi](https://github.com/re-al-7/SSI.SSD.TodoApi)  
> **Framework:** [openspec.dev](https://openspec.dev/)

---

## ¿Por qué Spec-First?

El flujo default de desarrollo con IA es prompt-driven: describes lo que quieres, el agente genera código, iteras. Funciona para tareas aisladas. Se rompe para cualquier cosa con superficie real — múltiples endpoints, reglas de dominio, concerns transversales. El agente no tiene memoria de decisiones previas, no conoce las restricciones arquitectónicas, y no puede distinguir "lo que se decidió" de "lo que acabo de asumir".

El Spec-Driven Development (SDD) invierte esto. Primero escribes el contrato — qué debe hacer el sistema, qué no debe hacer, qué constituye éxito — y recién entonces le pides al agente que lo implemente. Las specs se "commitean" junto al código y se convierten en un documento vivo que sobrevive a los resets del context window.

> *"Las specs no son documentación escrita después. Son la fuente de verdad que el agente lee antes de escribir cualquier cosa."*

[OpenSpec](https://openspec.dev/) es un framework ligero que operacionaliza esta idea. No reemplaza tu IDE, tu lenguaje ni tu agente — agrega una capa de specs delante de ellos, con un CLI pequeño y slash commands que se integran directamente con Claude Code.

---

## La Capa de Contexto — Antes de Cualquier Feature

La primera inversión es un conjunto de archivos de contexto que Claude Code lee al inicio de cada sesión. Son la memoria permanente del proyecto — lo que le dirías a cualquier integrante nuevo el primer día.

```
TodoApi/
├── CLAUDE.md                  ← comportamiento del agente, reglas TDD/SDD
├── .claude/
│   ├── settings.json          ← comandos bash permitidos y denegados
│   └── settings.local.json    ← overrides personales (gitignored)
└── openspec/
    ├── CONTEXT.md             ← stack, config de runtime, estructura del proyecto
    ├── STANDARDS.md           ← naming, code style, prefijos de commit
    └── DATA-MODEL.md          ← entidades, campos, relaciones
```

Cada archivo tiene una sola responsabilidad:

- **`CONTEXT.md`** responde "¿qué estamos construyendo y cómo corre?" — versión de .NET, motor de base de datos, base path de la API, puerto.
- **`STANDARDS.md`** responde "¿cómo escribimos código?" — variables en `camelCase`, métodos en `PascalCase`, campos privados con `_camelCase`, patrón AAA en tests.
- **`DATA-MODEL.md`** es el esquema canónico: entidades, tipos de campos, constraints y relaciones.
- **`CLAUDE.md`** es específico de Claude Code — lo primero que lee el agente. Codifica el contrato de trabajo: siempre leer la spec antes de implementar, nunca escribir código sin un test que falle primero, usar `ProblemDetails` para todos los errores, commitear specs y código juntos.

> **Lección aprendida:** La configuración de runtime — puertos, launch profiles, `appsettings.json` — pertenece tanto a `CONTEXT.md` como a una `specs/infrastructure/spec.md` dedicada. En este proyecto, `launchSettings.json` y `appsettings.json` tuvieron que crearse manualmente porque no estaban declarados en ninguna spec. Si un archivo necesita existir para que el proyecto corra, necesita estar en una spec.

---

## Feature Specs — Just Enough, Just In Time

Las specs de features viven en `openspec/specs/{feature}/`. Cada feature recibe dos archivos: `spec.md` para requisitos y escenarios, y `tests.md` para el plan de tests correspondiente.

> **Importante:** No necesitás tener todas las specs escritas antes de escribir código. La regla es: escribí la spec y el plan de tests para el feature que vas a implementar ahora. Nada más.

Las specs usan lenguaje estilo RFC — **SHALL**, **SHALL NOT**, **MAY** — y escenarios Given/When/Then:

```markdown
### Requisito: Crear lista
El sistema SHALL permitir crear una lista de tareas con nombre.

#### Escenario: Nombre duplicado
- GIVEN una lista con el mismo nombre ya existe
- WHEN se llama POST /api/v1/lists
- THEN retornar 409 Conflict
```

El `tests.md` companion mapea cada escenario a un test nombrado antes de que exista una sola línea de implementación:

```markdown
### Escenario: Nombre duplicado
- [ ] `CreateList_DuplicateName_Returns409`
- [ ] `CreateList_DuplicateName_DoesNotCreateRecord`
```

Este es el punto de integración con TDD. Los nombres de tests en `tests.md` se convierten en los nombres de métodos en `tests/TodoApi.Tests/`. El agente escribe los tests que fallan primero, luego implementa hasta que pasen.

| Archivo | Responde a | Se escribe cuando |
|---|---|---|
| `CONTEXT.md` | ¿Qué construimos y cómo corre? | Una vez, al inicio |
| `STANDARDS.md` | ¿Cómo escribimos código? | Una vez, al inicio |
| `DATA-MODEL.md` | ¿Cómo es el esquema? | Antes del primer feature, se actualiza según necesidad |
| `specs/{feature}/spec.md` | ¿Qué debe hacer este feature? | Antes de implementar el feature |
| `specs/{feature}/tests.md` | ¿Cómo verificamos cada escenario? | Antes de implementar el feature |

---

## El Ciclo de Implementación

OpenSpec introduce tres slash commands que mapean directamente al ciclo SDD. Los tres corren dentro de una sesión de Claude Code (`claude`) desde la raíz del proyecto.

**Ciclo completo por feature:**

1. **`/opsx:propose`** — Claude lee las specs y genera un directorio de change con `proposal.md`, `design.md` y `tasks.md`
2. **Revisar el proposal en Rider** antes de aplicar — verificar scope, decisiones arquitectónicas, archivos a modificar
3. **`/opsx:apply`** — Claude implementa las tasks, escribiendo los tests que fallan primero, luego la implementación
4. **Verificar** — `dotnet build`, `dotnet test`, `dotnet run`
5. **`/opsx:archive`** — el change se archiva y los delta specs se mergean en el árbol principal de specs

El `proposal.md` no es una formalidad — es donde se detectan desalineaciones antes de que se conviertan en código.

---

## La Revisión del Proposal — Un Ejemplo Real

Al implementar la spec `todo-errors`, el propose inicial fue:

```
/opsx:propose "Global error handling middleware con ProblemDetails RFC 7807"
```

Claude generó un proposal razonable — pero el `proposal.md` contenía esta línea:

> *"Controllers continue to return `NotFound()` directly where applicable — the middleware handles the exception path."*

Esto es la **Opción B**: dos code paths para errores 404. La spec `todo-errors` exige un contrato ProblemDetails uniforme para todos los tipos de error, lo que requiere la **Opción A** — los services lanzan `NotFoundException`, el middleware la captura, y los controllers no tienen lógica de null-check en absoluto.

La corrección fue reemplazar el proposal con una intención más explícita:

```
/opsx:propose "Global error handling middleware con ProblemDetails RFC 7807 —
los services deben lanzar NotFoundException en lugar de retornar null,
controllers no deben retornar NotFound() directamente"
```

> **Regla para el texto del propose:** Si ya está en la spec, no necesitás repetirlo. Si *no* está en la spec pero tenés una preferencia arquitectónica específica, agrégalo al propose — o mejor, actualizá la spec primero. El texto del propose es para intención y constraints; la spec es para requisitos y escenarios.

El proposal resultante fue sustancialmente diferente. Claude refactorizó ambas interfaces de service para eliminar los nullable returns, reescribió las implementaciones para lanzar excepciones en lugar de retornar null, simplificó los controllers eliminando todo el boilerplate de null-check, y actualizó los tests de service existentes para hacer assert de `NotFoundException` en lugar de null. El cambio tocó nueve archivos — todos correctamente.

---

## Estructura Final del Proyecto

```
SSI.SSD.TodoApi/
├── CLAUDE.md
├── TodoApi.sln
├── .claude/
│   ├── settings.json
│   ├── settings.local.json
│   └── skills/
│       ├── openspec-propose/
│       ├── openspec-apply-change/
│       ├── openspec-archive-change/
│       ├── openspec-explore/
│       └── openspec-sync-specs/
├── openspec/
│   ├── config.yaml
│   ├── CONTEXT.md
│   ├── DATA-MODEL.md
│   ├── STANDARDS.md
│   ├── changes/archive/          ← 3 ciclos completados
│   └── specs/
│       ├── infrastructure/       spec.md  tests.md
│       ├── todo-list/            spec.md  tests.md
│       ├── todo-item/            spec.md  tests.md
│       └── todo-errors/          spec.md  tests.md
├── src/
│   ├── Controllers/
│   │   ├── ListsController.cs
│   │   └── ItemsController.cs
│   ├── Services/
│   │   ├── IListService.cs   ListService.cs
│   │   └── IItemService.cs   ItemService.cs
│   ├── Models/
│   │   ├── TodoList.cs
│   │   └── TodoItem.cs
│   ├── DTOs/
│   │   ├── CreateListRequest.cs   CreateItemRequest.cs
│   │   ├── UpdateListRequest.cs   UpdateItemRequest.cs
│   │   ├── ListResponse.cs        ListDetailResponse.cs
│   │   └── ItemResponse.cs
│   ├── Data/
│   │   ├── AppDbContext.cs
│   │   └── Migrations/
│   ├── Middleware/
│   │   └── ErrorHandlingMiddleware.cs
│   ├── Exceptions.cs
│   └── Program.cs
└── tests/TodoApi.Tests/
    ├── Lists/
    │   ├── ListServiceCreateTests.cs
    │   ├── ListServiceGetAllTests.cs
    │   ├── ListServiceGetByIdTests.cs
    │   ├── ListServiceUpdateTests.cs
    │   └── ListServiceDeleteTests.cs
    ├── Items/
    │   └── ItemServiceTests.cs
    └── Middleware/
        └── ErrorHandlingMiddlewareTests.cs
```

**Resultado de tests al final de los tres ciclos:**

```
total: 24+   failed: 0   succeeded: 24+   skipped: 0
Build succeeded in 4.1s
```

---

## Qué Hace Realmente OpenSpec

Vale ser precisos sobre qué aporta OpenSpec vs qué aporta Claude Code. OpenSpec no escribe código. No analiza tu codebase. Genera **skills** — archivos Markdown estructurados en `.claude/skills/` — que Claude Code detecta automáticamente.

Cada skill es un conjunto de instrucciones que le dice al agente cómo comportarse durante una fase específica del workflow. El skill `openspec-propose` dice: lee el contexto del proyecto, lee la spec relevante, produce un proposal con esta estructura. El skill `openspec-apply-change` dice: lee las tasks, implementa en orden, corré los tests después de cada cambio.

El CLI (`openspec init`, `openspec config`, `openspec update`) administra qué skills se generan según tu perfil:

| Perfil | Workflows disponibles |
|---|---|
| **core** | `propose` `explore` `apply` `archive` |
| **custom** | + `sync` `new` `continue` `ff` `verify` `bulk-archive` `onboard` |

> **Nota sobre `/opsx:sync`:** En OpenSpec v1.3.1, `sync` aparece en la documentación del core profile pero no se instala con `openspec config profile core` seguido de `openspec update`. Para obtenerlo usá el perfil custom: `openspec config profile custom --workflows propose,explore,apply,sync,archive` y luego `openspec update --force`.

---

## El openspec/config.yaml

Más allá de los archivos Markdown de contexto, OpenSpec soporta un `openspec/config.yaml` que inyecta contexto del proyecto directamente en cada invocación de skill. Es una alternativa más limpia a escribir contexto en Markdown plano — el agente lo lee automáticamente sin necesitar una referencia en `CLAUDE.md`:

```yaml
# openspec/config.yaml
schema: spec-driven
context: |
  Platform: .NET 10
  Language: C# 13
  Database: SQLite via EF Core
  API base path: /api/v1
  Port: 5020
  Error format: ProblemDetails RFC 7807
rules:
  proposal:
    - Identificar archivos a modificar, no solo a crear
    - Indicar si los tests existentes necesitan actualizarse
  specs:
    - Usar SHALL/SHALL NOT para requisitos
    - Usar Given/When/Then para escenarios
  tasks:
    - Incluir una task final de verificación con dotnet test
```

---

## La Integración SDD + TDD

SDD y TDD son complementarios, no competitivos. La spec define el *qué*. El plan de tests en `tests.md` define *cómo verificar el qué*. Los archivos `.cs` de test son la implementación de ese plan. El código de producción hace pasar los tests.

```
spec.md  →  tests.md  →  *Tests.cs (red)  →  implementación  →  *Tests.cs (green)
```

La convención de naming en `tests.md` importa. Los nombres de tests siguen el patrón `MethodName_Scenario_ExpectedResult`:

- `CreateList_DuplicateName_Returns409`
- `DeleteList_WithTasks_DeletesListAndAllTasks`
- `Middleware_UnhandledException_DoesNotExposeDetail`

Estos nombres vienen directamente de los escenarios de la spec, creando una línea trazable desde el requisito hasta el test hasta la implementación.

---

## Tradeoffs y Cuándo Funciona Mejor

SDD con OpenSpec agrega trabajo upfront. Escribir specs, planes de tests y archivos de contexto antes de tocar código es más lento que promptear directamente — en la primera hora. A lo largo de la vida de un proyecto, se recupera con creces: comportamiento predecible del agente, cero momentos de "¿cuál era la decisión arquitectónica acá?", y la capacidad de pasarle contexto a una sesión nueva de Claude Code sin tener que re-explicar nada.

**Este approach funciona mejor cuando:**

- Tienes múltiples features relacionados con reglas de dominio compartidas
- El proyecto abarca múltiples sesiones (el agente olvida; las specs no)
- Hay más de un desarrollador involucrado y la consistencia importa
- Estás refactorizando — los delta specs describen qué cambia sin reescribir todo

**Agrega menos valor en:**

- Scripts verdaderamente aislados
- Transformaciones de datos one-off
- Cualquier cosa donde el contexto completo entra cómodamente en un solo prompt

---

## Código Fuente

El proyecto completo — incluyendo todas las specs, archivos de contexto, código generado y tests — está disponible en [github.com/re-al-7/SSI.SSD.TodoApi](https://github.com/re-al-7/SSI.SSD.TodoApi).

El framework OpenSpec está en [openspec.dev](https://openspec.dev/).
