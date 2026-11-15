# Informe Técnico de Auditoría y Remediación de Accesibilidad Web — Spring PetClinic (Angular)

## 1. Portada y contexto
* **Proyecto auditado:** Spring PetClinic (`spring-petclinic-angular`)
* **Entorno de ejecución:** macOS · Node.js / npm · Frontend Angular (`http://localhost:4200`) · Backend REST Docker (`springcommunity/spring-petclinic-rest` en puerto `9966`)
* **Rama de trabajo Git:** `a11y-remediacion`
* **Herramientas utilizadas:** `@axe-core/cli` (`axe-core v4.14.0`) sobre navegador Firefox (`148.0`), Firefox DevTools (Inspector, Console y Accessibility Tree) e inspección estática de plantillas HTML/Angular.
* **Marco de referencia:** WCAG 2.2 Nivel AA y norma europea EN 301 549.

---

## 2. Alcance y metodología
La auditoría se acotó a tres vistas representativas de los flujos principales de la aplicación:
1. **Home (`/petclinic/`):** Pantalla de aterrizaje y navegación global (`welcome.component.html` y `app.component.html`).
2. **Owners (`/petclinic/owners`):** Consulta y listado tabular de propietarios (`owner-list.component.html`).
3. **Add Owner (`/petclinic/owners/add`):** Formulario de creación de nuevos propietarios con validación reactiva (`owner-add.component.html`).

**Metodología aplicada:**
1. **Línea base automática (*baseline*):** Ejecución de `@axe-core/cli` con motor Firefox sobre las tres URLs antes de aplicar cambios (`evidencias/json-antes/`).
2. **Validación manual (5 comprobaciones):** Pruebas sin ratón (`Tab`, `Shift+Tab`, `Enter`, `Espacio`), verificación de orden e indicador visual de foco, inspección de asociación `<label>`/`<input>` y gestión de errores dinámicos, y revisión de jerarquía de encabezados y textos `alt`.
3. **Triaje y priorización:** Cruce de hallazgos automáticos y manuales para confirmar reproducibilidad e impacto real, seleccionando 5 problemas documentados (`A11Y-01` a `A11Y-05`).
4. **Análisis crítico asistido por IA:** Evaluación de propuestas de remediación mediante un prompt acotado, aplicando filtro humano para descartar soluciones ARIA incorrectas o referencias rotas.
5. **Remediación incremental y re-test:** Implementación de 3 correcciones mediante commits atómicos en Git y repetición de pruebas automáticas (`evidencias/json-despues/`) y manuales.

---

## 3. Baseline automática: resultados `axe-core` por página
Ejecución realizada con `@axe-core/cli` (`axe-core v4.14.0`, Firefox `148.0`) antes de modificar código:

| Página | Violaciones | Incomplete | Principales problemas detectados | Prioridad inicial |
|---|---|---|---|---|
| **Home** (`/petclinic/`) | `0` | `0` | Ninguna violación detectada automáticamente. | **Baja** (requiere verificación manual de navegación y contenido no textual). |
| **Owners** (`/petclinic/owners`) | `1` | `0` | `page-has-heading-one` (impacto `moderate`, tags `cat.semantics, best-practice`): ausencia de encabezado `<h1>`. | **Media** (buena práctica semántica; vista de consulta con tabla dinámica). |
| **Add Owner** (`/petclinic/owners/add`) | `1` | `0` | `page-has-heading-one` (impacto `moderate`, tags `cat.semantics, best-practice`): ausencia de encabezado `<h1>`. | **Media-Alta** (formulario crítico de entrada de datos donde los errores dinámicos deben auditarse manualmente). |

> **Nota metodológica:** La regla `page-has-heading-one` pertenece a la categoría `best-practice` de Deque. Obtener `0` o `1` alerta automática no certifica conformidad WCAG 2.2 AA, ya que `axe-core` en reposo no dispara validaciones dinámicas de formularios ni evalúa la calidad semántica real de los textos `alt` o tablas mal anidadas.

---

## 4. Validación manual: cinco comprobaciones y evidencias

| ID | Comprobación | Página | Resultado | Evidencia y observaciones técnicas |
|---|---|---|---|---|
| **MAN-01** | Navegación con teclado (`Tab` / `Shift+Tab`) | Home, Owners, Add Owner | **Pass** | Todos los enlaces de la barra superior (`Home`, desplegables `Owners` y `Veterinarians`, `Pet Types`, `Specialties`), campos y botones son alcanzables y operables sin ratón ni trampas de teclado (WCAG 2.1.1 / 2.1.2). |
| **MAN-02** | Orden del foco | Home, Owners, Add Owner | **Pass** | La secuencia de tabulación sigue el orden lógico de lectura (izquierda a derecha en navegación y arriba abajo en formularios y tablas) sin saltos incoherentes (WCAG 2.4.3). |
| **MAN-03** | Foco visible | Home, Owners, Add Owner | **Pass** | Los controles interactivos y botones (`Back`, `Add Owner`, inputs y enlaces) muestran un contorno visual perceptible al recibir foco mediante teclado (`evidencias/capturas/man-03-foco-visible.png`) (WCAG 2.4.7). |
| **MAN-04** | Labels y errores del formulario | Add Owner | **Fail (parcial)** | Los `<label for="...">` enfocan correctamente sus `<input>` al hacer clic (WCAG 3.3.2), pero al disparar errores de validación los mensajes `<div class="invalid-feedback">` carecen de `aria-describedby` y `aria-invalid` en el `<input>` (WCAG 1.3.1 / 3.3.1). |
| **MAN-05** | Títulos (`h1`–`h6`) / `alt` | Home, Owners, Add Owner | **Fail** | Confirmado en DevTools Console que `/owners/add` solo contiene `H2: "New Owner"` sin ningún `<h1>` (`evidencias/capturas/man-05-titulos-alt.png`), y en Home `pets.png` usa `alt="pets logo"` para una fotografía de un perro y un gato (WCAG 1.1.1 / 1.3.1 / 2.4.6). |

---

## 5. Tabla consolidada de findings: impacto, prioridad y estado

| ID | Página | Problema | Origen | Impacto | ¿Confirmado? | Prioridad |
|---|---|---|---|---|---|---|
| **A11Y-01** | Owners (`/owners`) | Ausencia de encabezado principal `<h1>` (`<h2>Owners</h2>` en `owner-list.component.html`). | Automático (`axe-core`) + Manual (`MAN-05`) | Moderado (dificulta navegación rápida por encabezados en lectores de pantalla) | Sí | **Media** |
| **A11Y-02** | Add Owner (`/owners/add`) | Ausencia de encabezado principal `<h1>` (`<h2>New Owner</h2>` en `owner-add.component.html`). | Automático (`axe-core`) + Manual (`MAN-05`) | Moderado (desorientación estructural al entrar en el formulario) | Sí | **Media** |
| **A11Y-03** | Add Owner (`/owners/add`) | Mensajes de error (`.invalid-feedback`) sin vinculación programática (`aria-invalid` / `aria-describedby`) en los 5 campos. | Manual / Revisión HTML (`MAN-04`) | Alto (usuarios ciegos no reciben la descripción del error ni el motivo por el que `Add Owner` sigue deshabilitado) | Sí | **Alta** |
| **A11Y-04** | Owners (`/owners`) | Anidamiento HTML inválido de `<tr>` directamente dentro de `<td>` en la columna `Pets` de `#ownersTable`. | Manual / Revisión HTML | Alto (corrompe la estructura de filas y celdas anunciada por lectores de pantalla; WCAG 1.3.1) | Sí | **Alta** |
| **A11Y-05** | Home (`/`) | Alternativa textual inexacta (`alt="pets logo"`) en la imagen principal `pets.png` (`welcome.component.html`). | Manual / Revisión HTML (`MAN-05`) | Bajo-Moderado (describe un logotipo inexistente en lugar de la fotografía del perro y el gato; WCAG 1.1.1) | Sí | **Media** |

---

## 6. Análisis asistido por IA: propuesta y validación humana

### Prompt utilizado
```text
Analiza estos findings de accesibilidad.
Para cada problema:
- resume la evidencia;
- propón una prioridad;
- sugiere una remediación concreta;
- indica si necesita validación manual;
- no inventes criterios WCAG ni des por confirmado un problema sin evidencia.
Agrupa findings duplicados cuando corresponda.