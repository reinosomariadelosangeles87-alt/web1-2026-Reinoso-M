# Programación Web 1 — Guía del curso

**Código:** 235G8F · **Créditos:** 3 · **Semestre:** 2026-2
Universidad de Caldas · Facultad de Inteligencia Artificial e Ingenierías
Departamento de Sistemas e Informática · Ingeniería en Informática, cuarto semestre

> **Horario:** un bloque semanal de 6:00 pm a 10:00 pm (4 horas con descanso).
> **Modalidad:** teórico-práctica. **Nota aprobatoria:** 3.0. **No habilitable.**

---

## Bienvenida: esto no es un curso de clases magistrales

Vas a construir videojuegos. Nueve de ellos, uno tras otro, cada uno usando una tecnología nueva, y todos van a quedar publicados en internet con una URL que puedas mostrarle a quien quieras. El último lo vas a sustentar como si fuera un producto real, porque a esas alturas lo va a ser.

La razón de usar juegos no es que sea entretenido, aunque lo sea. Es que un videojuego te obliga a resolver, en pequeño, casi todos los problemas difíciles del desarrollo web: manejar estado que cambia sesenta veces por segundo, responder a eventos impredecibles del usuario, pedir datos a un servidor sin congelar la pantalla, guardar progreso, y hacer que todo eso funcione igual en un celular y en un monitor de escritorio. Un formulario de contacto no te enseña nada de eso. Un juego sí.

Hay una segunda razón, más incómoda. Puedes pedirle a una inteligencia artificial que te escriba un formulario de contacto y probablemente funcione. Cuando le pides un juego, el código que produce suele compilar, correr, y estar sutilmente mal: el personaje atraviesa las paredes, el puntaje se reinicia al azar, la animación va al doble de velocidad en máquinas rápidas. Ese es exactamente el terreno donde vas a aprender a auditar lo que la máquina escribe en lugar de aceptarlo. Es la competencia que este curso considera no negociable.

### Cómo funciona la sesión de cuatro horas

Sé que llegas después de trabajar o de un día completo de clase, y que cuatro horas de teoría a las nueve de la noche no le sirven a nadie. La sesión está diseñada en consecuencia:

| Tramo | Duración | Qué pasa |
|---|---|---|
| Apertura y repaso | 20 min | Revisamos lo entregado, resolvemos dudas del PR de la semana |
| Bloque teórico 1 | 30 min | Concepto nuevo, con diagramas. Nunca más largo que esto |
| Taller guiado | 55 min | Escribes código conmigo en pantalla, paso a paso |
| **Descanso** | **20 min** | En serio, descansa. Levántate de la silla |
| Bloque teórico 2 | 25 min | Segundo concepto, o profundización del primero |
| Taller autónomo | 70 min | Trabajas en tu misión, con asesoría disponible |
| Cierre y auditoría | 20 min | Revisión cruzada de código entre compañeros, asignación de la semana |

De las cuatro horas, casi dos y media son de teclado. Eso es deliberado y no va a cambiar.

### El presupuesto de horas, completo

El PIAA de la asignatura fija 72 horas presenciales y 72 no presenciales, en relación 1 a 1. Todo el material de este curso está presupuestado contra esas horas, sin excepción:

| Concepto | Horas |
|---|---|
| Horas teóricas presenciales (T) | 36 |
| Horas prácticas presenciales (P) | 36 |
| **Total presencial** | **72** (18 sesiones × 4 h) |
| Trabajo autónomo no presencial (NP) | 72 (4 h semanales × 18 semanas) |
| **Total de dedicación del curso** | **144 horas** |

Las 21 semanas del calendario institucional incluyen inducción, receso y semana de sustentaciones finales; las 18 sesiones de clase se distribuyen dentro de ese periodo.

Reparto por módulo:

| Módulo | Tema | T | P | Sesiones | NP |
|---|---|---|---|---|---|
| 1 | Fundamentos de la red y herramientas asistidas | 6 | 6 | 1–3 | 12 |
| 2 | Ingeniería de interfaces y frameworks de utilidad | 6 | 6 | 4–6 | 12 |
| 3 | Interactividad algorítmica y asincronismo | 10 | 10 | 7–11 | 20 |
| 4 | Arquitectura de meta-frameworks y convergencia | 8 | 8 | 12–15 | 16 |
| 5 | Gestión de versiones, despliegue y proyecto | 6 | 6 | 16–18 | 12 |
| | **Totales** | **36** | **36** | **18** | **72** |

El módulo 3 recibe la mayor tajada porque el asincronismo en JavaScript es, con diferencia, lo que más cuesta y lo que más se usa después. Si algo se retrasa en el semestre, se recorta de otro lado antes que de ahí.

### En qué se van tus 72 horas no presenciales

Estas son las horas que trabajas por fuera de clase, y están presupuestadas misión por misión. Si una misión te está tomando bastante más de lo indicado, no es que seas lento: es que algo se atascó y conviene que me escribas antes de perder la noche entera.

| Misión | Horas estimadas | Reparto sugerido |
|---|---|---|
| 00 · Registro de jugador | 1 | Se hace en clase; solo terminar el perfil |
| 01 · Ficha de personaje | 7 | 1 planear · 4 marcar · 1 validar · 1 documentar |
| 02 · Tablero adaptable | 9 | 1 planear · 3 CSS a mano · 3 Tailwind · 1 probar · 1 comparar |
| 03 · Juego de memoria | 10 | 2 planear · 6 programar · 1 probar · 1 documentar |
| 04 · Trivia con API | 10 | 2 planear · 5 programar · 2 manejar errores · 1 documentar |
| 05 · Migración a TypeScript | 8 | 1 configurar · 5 migrar · 2 documentar los errores |
| 06 · Mazmorra en React | 10 | 3 diseñar componentes · 6 programar · 1 documentar |
| 07 · Marcador con Next.js | 9 | 2 decidir renderizado · 5 programar · 1 desplegar · 1 documentar |
| 08 · Proyecto final | 5 | Trabajo individual; el resto es en equipo y en clase |
| Lectura previa a cada sesión | 3 | ~10 min antes de cada una de las 18 sesiones |
| **Total** | **72** | |

Las horas de lectura previa son reales y cuentan: la metodología del curso asume que llegas a la sesión habiendo visto el material, porque de otro modo los 30 minutos de teoría no alcanzan.

### Sobre la asistencia

El reglamento es estricto y conviene tenerlo claro desde hoy. Con **11 horas** de inasistencia justificada (15%) o **18 horas** entre justificadas y no justificadas (25%) se reprueba la asignatura. Como cada sesión vale 4 horas, eso significa que **perder tres sesiones completas ya te pone en el límite**. Avisa con antelación si vas a faltar.

### Requisito previo

Fundamentos de las Tecnologías de la Información y la Comunicación (316G8F). Se asume que manejas lógica de programación básica, variables, condicionales, ciclos y funciones. No se asume nada de HTML, CSS, JavaScript, Git ni línea de comandos.

---

## La campaña: cómo se gana la nota jugando

El curso está estructurado como una campaña de cinco zonas. Avanzas acumulando **XP** (puntos de experiencia) que se convierten directamente en tu nota. No hay puntos decorativos: cada XP que ganas es nota real.

### Niveles

| Nivel | XP acumulado | Título | Equivale a |
|---|---|---|---|
| 1 | 0–199 | Aprendiz de la red | — |
| 2 | 200–449 | Maquetador | 1.5 |
| 3 | 450–699 | Artesano de interfaces | 2.4 |
| 4 | 700–899 | Domador del asincronismo | 3.0 ✅ |
| 5 | 900–1049 | Arquitecto de componentes | 3.7 |
| 6 | 1050–1150 | Ingeniero de despliegue | 4.5 |
| 7 | 1151+ | **Leyenda de la web** | 5.0 |

El XP total disponible es de **1200 puntos**. Necesitas **700** para aprobar, que es aproximadamente el 58% — ligeramente por encima del 3.0/5.0 estricto, deliberadamente, porque hay XP opcional de sobra para compensar una mala semana.

### De dónde sale el XP

| Fuente | Cantidad | XP | Subtotal |
|---|---|---|---|
| Misiones (prácticas 0 a 8) | 9 | 100 c/u | 900 |
| Quices de apertura de sesión | 6 | 15 c/u | 90 |
| Auditorías de código a un compañero | 4 | 25 c/u | 100 |
| Sustentación del proyecto final | 1 | 110 | 110 |
| **Total obligatorio** | | | **1200** |
| Misiones secundarias (opcionales) | 6 | 25 c/u | +150 |

Las misiones secundarias sirven para recuperar terreno o para llegar a Leyenda. Están descritas al final de cada módulo y son estrictamente opcionales.

### Insignias

Las insignias no dan XP: dan reconocimiento público en la tabla de la clase y, tres de ellas, ventajas concretas.

| Insignia | Cómo se consigue | Ventaja |
|---|---|---|
| 🎯 **Primera sangre** | Primer PR de la clase aprobado sin correcciones | — |
| 🧹 **Cero deuda** | Cuatro misiones seguidas sin comentarios de estilo | — |
| 🔍 **Cazador de alucinaciones** | Documentar un error real generado por IA, con la corrección | +1 día de plazo, acumulable |
| ♿ **Acceso para todos** | Una misión con 100/100 en accesibilidad | — |
| ⚡ **Velocista** | Core Web Vitals en verde en el proyecto final | — |
| 🤝 **Buen colega** | Reconocido en dos auditorías por ayudar de verdad | Descarta tu peor quiz |
| 🏗️ **Refactorizador** | Mejorar código propio de un módulo anterior, con medición | — |
| 👑 **Speedrun** | Entregar tres misiones antes del plazo, todas aprobadas | Elige tema del proyecto libre |

### Reglas de la campaña

Las misiones se entregan **antes de la medianoche del día anterior a la sesión**, para que yo pueda revisarlas y devolverte comentarios a tiempo. Cada día de retraso cuesta 10 XP de esa misión, hasta un máximo de 40; después de cuatro días la misión queda en cero, pero **igual debes entregarla** para tener derecho a presentar la siguiente. Nadie queda bloqueado por una mala semana.

Puedes **reintentar** cualquier misión una vez, dentro de la semana siguiente a recibir los comentarios de tu entrega, y recuperar hasta el 80% del XP que perdiste. Esto premia que corrijas y aprendas en lugar de seguir de largo.

Sobre el uso de IA: **está permitido y es parte del curso**, con una condición que no admite excepciones. Todo código generado por un asistente se declara en el archivo `IA.md` de la misión, indicando qué le pediste, qué te devolvió, qué estaba mal y qué corregiste. Una misión con código de IA no declarado se califica en **cero** y no admite reintento. No es un castigo moral: es que la competencia que estoy evaluando es precisamente tu capacidad de auditar, y sin la declaración no hay nada que evaluar.

---

## Modelo de entregas en GitHub

### Tu repositorio

Creas **un solo repositorio** para todo el semestre, público, llamado exactamente:

```
web1-2026-tuapellido
```

Por ejemplo: `web1-2026-ramirez`. Si hay dos apellidos iguales en la clase, agrega la inicial del nombre: `web1-2026-ramirez-a`.

La estructura es fija y no negociable, porque es lo que me permite calificar treinta repositorios sin perderme:

```
web1-2026-tuapellido/
├── README.md                 ← tu perfil de jugador, XP y enlaces a cada misión
├── practica-00-registro/
│   ├── README.md             ← qué hiciste, cómo correrlo, qué aprendiste
│   ├── IA.md                 ← declaración de uso de IA (obligatorio siempre)
│   └── src/
├── practica-01-ficha/
├── practica-02-tablero/
├── practica-03-memoria/
├── practica-04-trivia/
├── practica-05-tipos/
├── practica-06-mazmorra/
├── practica-07-marcador/
├── practica-08-proyecto/
└── .github/workflows/validar.yml   ← te lo doy yo, no lo modifiques
```

### El flujo de cada entrega

Cada misión sigue el mismo ciclo, siempre. Al terminar el semestre lo vas a tener automatizado en los dedos, y es exactamente el flujo que se usa en cualquier empresa de software:

```bash
# 1. Rama nueva desde main, siempre actualizada
git switch main
git pull
git switch -c feat/practica-03

# 2. Trabajas, con commits pequeños y descriptivos
git add .
git commit -m "feat(memoria): voltear carta al hacer clic"
git commit -m "fix(memoria): evitar voltear tres cartas a la vez"

# 3. Subes la rama
git push -u origin feat/practica-03

# 4. Abres el Pull Request en GitHub con la plantilla
# 5. GitHub Actions valida la estructura automáticamente
# 6. Recibes comentarios y corriges si hace falta
# 7. Haces merge y etiquetas la versión
git switch main && git pull
git tag practica-03-v1 && git push --tags
```

Los mensajes de commit siguen [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/): `feat` para funcionalidad nueva, `fix` para corrección, `docs` para documentación, `refactor` para reestructurar sin cambiar comportamiento, `style` para formato. No es capricho; es el estándar que permite generar historiales de cambios automáticamente.

### Cómo se revisa tu entrega, en este orden

Cuando abro tu PR miro cinco cosas, y las miro rápido. Por eso la estructura importa tanto:

Primero se mira si **GitHub Actions pasó en verde**. Si está en rojo, la revisión ni empieza: la validación comprueba que existan las carpetas y los archivos obligatorios, que el HTML sea válido y que el proyecto compile. Segundo, si el **`IA.md` está lleno de verdad**, no con una frase genérica. Tercero, se **abre tu URL desplegada** y se juega. Si el juego no funciona, el resto es secundario. Cuarto, se **lee el código** buscando lo específico de la rúbrica del módulo. Y quinto, se revisa tu **historial de commits**: se espera ver un proceso, no un único commit gigante llamado "todo".

### La plantilla de tu Pull Request

Cada PR debe traer esto en la descripción, y el workflow verifica que estén las secciones:

```markdown
## Misión
Práctica 03 — Juego de memoria

## URL desplegada
https://mi-memoria.vercel.app

## Qué funciona
- Las cartas se voltean y se emparejan
- El puntaje sube y se guarda entre partidas

## Qué no alcancé a hacer
- El temporizador no se pausa al minimizar la pestaña

## Uso de IA
Ver IA.md. Resumen: le pedí la lógica de mezclado y me dio un
Fisher-Yates mal implementado que no barajaba la última posición.

## Lo más difícil
Entender por qué al hacer clic muy rápido se volteaban tres cartas.

## Autoevaluación
Estimo 85/100. Perdí puntos en el temporizador.
```

Esa autoevaluación no es un trámite. Comparo tu estimación con mi calificación, y aprender a estimar tu propio trabajo con precisión es una habilidad profesional real.

### Rúbrica general (100 XP por misión)

Todas las misiones se califican con la misma estructura, para que sepas siempre dónde estás parado:

| Criterio | XP | Qué busco concretamente |
|---|---|---|
| **Funciona** | 35 | El juego corre en la URL desplegada, sin errores en consola, y cumple lo que pedía la misión |
| **Calidad del código** | 25 | Nombres claros, funciones cortas, sin repetición evidente, sin código muerto ni comentado |
| **Lo específico del módulo** | 20 | Lo que ese módulo evalúa: semántica, responsive, tipos, componentes, según corresponda |
| **Proceso en Git** | 10 | Commits pequeños y descriptivos, rama correcta, PR con la plantilla completa |
| **Auditoría de IA** | 10 | `IA.md` real y reflexivo, con un error identificado y corregido |

Los 20 puntos "específicos del módulo" están detallados en cada misión más abajo.

---

## Módulo 1 — Fundamentos de la red y herramientas asistidas

**Zona 1: El Puerto de Entrada** · Sesiones 1 a 3 · 6 h teóricas + 6 h prácticas + 12 h autónomas

Antes de escribir una sola línea de interfaz necesitas entender qué pasa cuando escribes una dirección y presionas Enter. Este módulo es corto en apariencia y decisivo en la práctica: casi todos los errores confusos que vas a encontrar en el semestre se explican por algo de aquí.

### Sesión 1 — Arquitectura web y el protocolo HTTP (2 h T + 2 h P)

Qué pasa realmente entre tu navegador y un servidor. El modelo cliente-servidor, por qué HTTP no tiene estado y qué consecuencias tiene eso, los verbos (GET, POST, PUT, DELETE), los códigos de estado y por qué importa devolver el correcto, las cabeceras, y qué agrega HTTPS con TLS. Abrimos las herramientas de desarrollo del navegador y leemos peticiones reales en la pestaña de red.

Conceptos que quedan cubiertos: DNS, TCP, el ciclo petición-respuesta, idempotencia, caché.

**Fuentes**
- IETF. *RFC 9110: HTTP Semantics*. https://www.rfc-editor.org/rfc/rfc9110.html
- IETF. *RFC 9112: HTTP/1.1*. https://www.rfc-editor.org/rfc/rfc9112.html
- MDN Web Docs. *An overview of HTTP*. https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Overview

### Sesión 2 — HTML5 semántico y accesibilidad (2 h T + 2 h P)

La diferencia entre marcar contenido y decorarlo. Estructura de un documento, la jerarquía de encabezados y por qué saltarse un nivel rompe cosas, los elementos de sección, formularios y sus etiquetas, imágenes con texto alternativo que sirva. Introducción a WCAG 2.2 y a por qué la accesibilidad no es un extra opcional sino una obligación legal en buena parte del mundo.

**Fuentes**
- WHATWG. *HTML Living Standard*. https://html.spec.whatwg.org/multipage/
- W3C. *Web Content Accessibility Guidelines (WCAG) 2.2*. https://www.w3.org/TR/WCAG22/
- W3C. *How to Meet WCAG (Quick Reference)*. https://www.w3.org/WAI/WCAG22/quickref/
- MDN Web Docs. *HTML heading elements*. https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/Heading_Elements

### Sesión 3 — SEO técnico y asistentes de código (2 h T + 2 h P)

Cómo lee un buscador tu página y qué puedes hacer para que la entienda: metadatos, datos estructurados, URLs legibles, rendimiento como factor de posicionamiento. La segunda mitad es la introducción formal al uso de asistentes de IA: qué hacen bien (código repetitivo, conversiones de formato, andamiaje), qué hacen mal (lógica de dominio, casos borde, seguridad), y cómo se escribe una especificación antes de pedir código. Aquí establecemos la regla de *specs before code* que rige el resto del semestre.

**Fuentes**
- Google Search Central. *SEO Starter Guide*. https://developers.google.com/search/docs/fundamentals/seo-starter-guide
- Prather, J., Denny, P., Leinonen, J., et al. (2023). The robots are here: Navigating the generative AI revolution in computing education. *ITiCSE-WGR '23*, 108–159. https://doi.org/10.1145/3623762.3633499
- Denny, P., Prather, J., Becker, B. A., et al. (2024). Computing education in the era of generative AI. *Communications of the ACM, 67*(2), 56–67. https://doi.org/10.1145/3624720

### Misión 00 — Registro de jugador (25 XP, en clase)

Tu primera entrega es minúscula a propósito: crear el repositorio, configurar Git con tu nombre y correo, escribir un `README.md` con tu perfil de jugador y abrir tu primer Pull Request. Lo hacemos juntos en la sesión 1 para que nadie arranque el semestre trabado en la herramienta.

### Misión 01 — Ficha de personaje (100 XP)

Construyes la ficha de un personaje de videojuego como página HTML, **sin una sola línea de CSS**. Debe tener nombre, imagen con texto alternativo descriptivo, tabla de estadísticas, lista de habilidades, historia en varios párrafos con jerarquía de encabezados correcta, y un formulario de contacto con etiquetas asociadas a sus campos.

La restricción de no usar CSS es el punto pedagógico entero: una página bien marcada semánticamente se ve razonable sin estilos, y una mal marcada se ve como una lista de basura. Vas a notar la diferencia inmediatamente.

Los 20 XP específicos del módulo se reparten así: el documento pasa el validador del W3C sin errores (5), la jerarquía de encabezados es correcta y sin saltos (5), se usan elementos de sección en lugar de `div` genéricos (5), y todas las imágenes y campos de formulario tienen texto alternativo y etiquetas útiles (5).

**Misión secundaria (25 XP):** consigue 100/100 en accesibilidad con Lighthouse y adjunta la captura. Habilita la insignia ♿.

---

## Módulo 2 — Ingeniería de interfaces y frameworks de utilidad

**Zona 2: El Taller de Diseño** · Sesiones 4 a 6 · 6 h teóricas + 6 h prácticas + 12 h autónomas

Ahora que el contenido está bien estructurado, lo hacemos ver bien y funcionar en cualquier pantalla. El error más común en este módulo es tratar el CSS como una colección de trucos memorizados; lo vamos a tratar como el sistema de reglas que en realidad es.

### Sesión 4 — La cascada, el modelo de caja y la especificidad (2 h T + 2 h P)

Por qué tu regla no se aplica: la cascada, la herencia, la especificidad y el origen de los estilos. El modelo de caja y por qué `box-sizing: border-box` es casi siempre lo que quieres. Unidades relativas y absolutas, y cuándo usar cada una. Variables CSS para no repetir colores por todo el archivo.

**Fuentes**
- W3C. *CSS Cascading and Inheritance Level 5*. https://www.w3.org/TR/css-cascade-5/
- MDN Web Docs. *CSS box model*. https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_box_model

### Sesión 5 — Flexbox y Grid (2 h T + 2 h P)

Los dos sistemas de maquetación modernos, cuándo usar cada uno y por qué no compiten. Flexbox para distribuir en un eje: barras de navegación, filas de botones, centrado. Grid para maquetar en dos ejes: la estructura de la página, tableros de juego, galerías. Construimos un tablero de ajedrez con Grid en clase, que es el mejor ejercicio que conozco para que el concepto quede.

**Fuentes**
- W3C. *CSS Flexible Box Layout Module Level 1*. https://www.w3.org/TR/css-flexbox-1/
- W3C. *CSS Grid Layout Module Level 1*. https://www.w3.org/TR/css-grid-1/
- MDN Web Docs. *Basic concepts of flexbox*. https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Flexible_box_layout/Basic_concepts
- MDN Web Docs. *CSS grid layout*. https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Grid_layout

### Sesión 6 — Mobile-first y Tailwind CSS (2 h T + 2 h P)

Diseñar desde la pantalla pequeña hacia arriba, y por qué ese orden y no el contrario. Media queries y puntos de quiebre. Luego el cambio de paradigma de las clases de utilidad con Tailwind: qué problema resuelve realmente (el CSS que crece sin control y nadie se atreve a borrar), cuándo conviene y cuándo es exagerado. Cerramos con generación de componentes asistida por IA, que en maquetación es donde los asistentes son genuinamente buenos.

**Fuentes**
- Tailwind CSS. *Styling with utility classes*. https://tailwindcss.com/docs/styling-with-utility-classes
- Tailwind CSS. *Responsive design*. https://tailwindcss.com/docs/responsive-design
- MDN Web Docs. *Lazy loading*. https://developer.mozilla.org/en-US/docs/Web/Performance/Guides/Lazy_loading

### Misión 02 — Tablero de juego adaptable (100 XP)

Maquetas la interfaz completa de un juego de mesa, todavía sin JavaScript: tablero en cuadrícula, panel de puntajes, zona de jugadores, controles. Debe funcionar en tres tamaños de pantalla, y la versión móvil tiene que reorganizarse de verdad, no solo encogerse.

Se entrega **dos veces el mismo diseño**: una versión con CSS escrito a mano y otra con Tailwind. Comparar tus dos soluciones al mismo problema es donde está el aprendizaje real de este módulo, y en el `README.md` argumentas cuál preferirías mantener dentro de un año y por qué.

Los 20 XP específicos: el tablero usa Grid correctamente, no una tabla ni posiciones absolutas (6), el diseño es mobile-first con puntos de quiebre razonados (6), no hay valores mágicos repetidos, se usan variables o utilidades (4), y la comparación entre ambos enfoques es reflexiva y no genérica (4).

**Misión secundaria (25 XP):** añade un tema oscuro que respete `prefers-color-scheme`.

---

## Módulo 3 — Interactividad algorítmica y asincronismo

**Zona 3: La Sala de Máquinas** · Sesiones 7 a 11 · 10 h teóricas + 10 h prácticas + 20 h autónomas

Este es el módulo más largo y el más difícil, y recibe la mayor asignación de horas del curso por una razón: el asincronismo es donde casi todo el mundo se estrella, y es lo que vas a usar todos los días durante el resto de tu carrera. Si algo del semestre se retrasa, se recorta de otro lado antes que de aquí.

### Sesión 7 — JavaScript moderno y programación funcional (2 h T + 2 h P)

Declaraciones y ámbito (`let`, `const` y por qué `var` quedó atrás), funciones flecha y qué cambia con `this`, desestructuración, parámetros por defecto, propagación. Los métodos de arreglo que reemplazan la mayoría de los ciclos: `map`, `filter`, `reduce`, `find`, `some`, `every`. Inmutabilidad y por qué mutar el estado compartido causa errores que no se pueden reproducir.

**Fuentes**
- Ecma International. *ECMAScript Language Specification*. https://262.ecma-international.org/
- MDN Web Docs. *JavaScript Guide*. https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide

### Sesión 8 — El DOM, eventos y el bucle de juego (2 h T + 2 h P)

Seleccionar y modificar elementos, crear nodos, delegación de eventos y por qué es mejor que poner cien escuchadores. El objeto del evento, la propagación y cómo detenerla. Luego el bucle de juego con `requestAnimationFrame`, por qué nunca se anima con `setInterval`, y el uso del delta de tiempo para que el juego corra igual en una máquina lenta y en una rápida.

**Fuentes**
- MDN Web Docs. *Anatomy of a video game*. https://developer.mozilla.org/en-US/docs/Games/Anatomy
- MDN Web Docs. *Window: requestAnimationFrame()*. https://developer.mozilla.org/en-US/docs/Web/API/Window/requestAnimationFrame
- MDN Web Docs. *Canvas API tutorial*. https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API/Tutorial

### Sesión 9 — Módulos, el bucle de eventos y concurrencia (2 h T + 2 h P)

Módulos de ES: `import`, `export` y cómo partir un programa en archivos que se puedan entender por separado. Después el corazón del módulo: la pila de llamadas, las colas de tareas y el bucle de eventos. Por qué JavaScript tiene un solo hilo y aun así no se congela. La diferencia entre microtareas y macrotareas, que explica el resultado de ese ejercicio clásico donde un `await` resuelto corre antes que un `setTimeout(0)`.

**Fuentes**
- MDN Web Docs. *JavaScript modules*. https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules
- MDN Web Docs. *JavaScript execution model*. https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Execution_model

### Sesión 10 — Promesas, async/await y APIs REST (2 h T + 2 h P)

Promesas: los tres estados, encadenamiento, manejo de errores. `async`/`await` como azúcar sintáctica sobre lo mismo, y qué sigue pasando por debajo. Ejecución en paralelo con `Promise.all` frente a esperas secuenciales innecesarias, que es el error de rendimiento más común que voy a ver en sus proyectos. Consumo de APIs REST con `fetch`, códigos de error, y qué hacer cuando la red falla, porque va a fallar.

**Fuentes**
- MDN Web Docs. *Using the Fetch API*. https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch
- MDN Web Docs. *Using Promises*. https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Using_promises

### Sesión 11 — TypeScript como seguridad estructural (2 h T + 2 h P)

Por qué un lenguaje con tipos sobre JavaScript, y qué clase de errores desaparecen. Tipos primitivos, interfaces y tipos, uniones, genéricos básicos, inferencia. Configuración de `tsconfig.json` y modo estricto. El argumento pedagógico central: los tipos son una especificación ejecutable, y son la mejor defensa que tienes contra el código plausible pero incorrecto que produce un asistente de IA.

**Fuentes**
- Microsoft. *The TypeScript Handbook*. https://www.typescriptlang.org/docs/handbook/intro.html

### Misión 03 — Juego de memoria (100 XP)

El clásico de voltear cartas y buscar parejas, con JavaScript puro y sin librerías. Necesita mezclado aleatorio correcto, control de estado para que no se puedan voltear tres cartas a la vez, contador de intentos, detección de victoria y un temporizador.

Ese detalle de las tres cartas parece trivial y es donde casi todos fallan: revela si entendiste que los eventos pueden llegar mientras una animación está en curso. Es la primera vez que te vas a topar de frente con la asincronía.

Los 20 XP específicos: el mezclado es un Fisher-Yates correcto y no un `sort` con random (5), el estado del juego está centralizado y no repartido en el DOM (5), se usa delegación de eventos (5), y no hay errores en consola durante una partida completa (5).

### Misión 04 — Trivia con API externa (100 XP)

Un juego de preguntas que consume una API pública real. Debe manejar el estado de carga, el error de red (pruébalo desconectando el wifi), preguntas con opciones, puntaje acumulado y un resumen final.

Aquí se califica con especial atención el manejo de errores, porque es lo que separa un ejercicio de clase de algo que podrías mostrar: qué ve el usuario cuando la API tarda cinco segundos, o devuelve un 500, o devuelve datos con un campo faltante.

Los 20 XP específicos: hay estados explícitos de carga, éxito y error, todos visibles en la interfaz (6), se usa `async`/`await` con `try`/`catch` real (5), las peticiones independientes van en paralelo con `Promise.all` cuando corresponde (5), y el código está partido en módulos con responsabilidades claras (4).

**Misión secundaria (25 XP):** agrega una caché con `sessionStorage` para no repetir peticiones idénticas.

### Misión 05 — Migración a TypeScript (100 XP)

Tomas tu propia misión 04 y la migras a TypeScript con modo estricto, sin usar `any` en ningún lado. Documentas en el `README.md` cada error que el compilador encontró en código que tú creías correcto.

Esa lista de errores es el entregable de verdad. Casi siempre aparecen dos o tres casos reales que habrían fallado en producción, y verlos en tu propio código convence mucho más que cualquier argumento que yo pueda dar en clase.

Los 20 XP específicos: cero usos de `any`, explícito o implícito (6), las respuestas de la API tienen interfaces declaradas (5), el modo estricto está activo y compila limpio (5), y la lista de errores encontrados es real y está comentada (4).

**Misión secundaria (25 XP):** añade pruebas unitarias con Vitest a la lógica de puntaje.

---

## Módulo 4 — Arquitectura de meta-frameworks y convergencia

**Zona 4: La Fortaleza de Componentes** · Sesiones 12 a 15 · 8 h teóricas + 8 h prácticas + 16 h autónomas

Hasta aquí manipulaste el DOM a mano y ya sentiste el dolor: el estado repartido por todos lados, la interfaz que se desincroniza de los datos. React existe porque ese dolor no escala. Este módulo es el salto a cómo se construye software web de verdad hoy.

### Sesión 12 — React: componentes, props y estado (2 h T + 2 h P)

Pensar en componentes en lugar de en páginas. JSX y qué compila. Props para que los datos bajen, estado para lo que cambia, y la regla de que el estado vive lo más arriba que sea necesario y no más. El DOM virtual y la reconciliación: por qué React puede volver a renderizar todo conceptualmente y aun así ser rápido. Renderizado condicional y listas con claves, y por qué usar el índice como clave causa errores raros.

**Fuentes**
- Meta. *React: Quick Start*. https://react.dev/learn
- Meta. *Built-in React Hooks*. https://react.dev/reference/react/hooks

### Sesión 13 — Hooks y ciclo de vida (2 h T + 2 h P)

`useState` a fondo, incluidas las actualizaciones por lote que confunden a todo el mundo la primera vez. `useEffect`: cuándo se ejecuta, el arreglo de dependencias, la función de limpieza y por qué olvidarla causa fugas de memoria. `useRef` para lo que no debe provocar renderizado, que en juegos es fundamental. `useReducer` cuando el estado se vuelve complejo, y `useMemo` con `useCallback` para optimizar, con la advertencia de que optimizar antes de medir es perder el tiempo.

### Sesión 14 — Next.js y estrategias de renderizado (2 h T + 2 h P)

Qué agrega un meta-framework sobre React: enrutamiento por sistema de archivos, optimización automática, convenciones. Las tres estrategias de renderizado (CSR, SSR, SSG), qué cuesta cada una y cómo se elige. El App Router y la distinción entre componentes de servidor y de cliente, que es el concepto más nuevo y el que más confusión genera. Cuándo `use client` es necesario y cuándo es un parche por no haber pensado la frontera.

**Fuentes**
- Vercel. *Next.js App Router: Getting Started*. https://nextjs.org/docs/app/getting-started
- Vercel. *Server and Client Components*. https://nextjs.org/docs/app/getting-started/server-and-client-components

### Sesión 15 — Integración de servicios de IA (2 h T + 2 h P)

Consumir APIs de modelos de lenguaje desde una aplicación web: dónde vive la clave de API y por qué jamás en el cliente, rutas de servidor como intermediario, respuestas en flujo, manejo de latencia y de costos. El diseño de la experiencia cuando la respuesta tarda o falla. Y la parte crítica: validar y sanear lo que devuelve un modelo antes de mostrarlo o guardarlo, porque un LLM es una entrada de usuario no confiable.

**Fuentes**
- OWASP. *OWASP Top Ten*. https://owasp.org/www-project-top-ten/
- Prather, J., Reeves, B. N., Leinonen, J., et al. (2024). The widening gap: The benefits and harms of generative AI for novice programmers. *ICER '24*, 469–486. https://doi.org/10.1145/3632620.3671116

### Misión 06 — Mazmorra por turnos en React (100 XP)

Un juego de exploración por turnos: cuadrícula, personaje que se mueve, enemigos, combate por turnos, inventario, puntos de vida. Todo con componentes de React y estado bien ubicado.

El reto real no es el juego, es la arquitectura. Si pones todo el estado en el componente raíz vas a sufrir; si lo dispersas demasiado, se desincroniza. Encontrar el punto medio es el aprendizaje.

Los 20 XP específicos: los componentes tienen una sola responsabilidad clara y son reutilizables (6), el estado está en el nivel correcto del árbol, sin duplicación (6), las listas usan claves estables y no el índice (4), y los efectos tienen dependencias correctas y limpieza donde hace falta (4).

### Misión 07 — Marcador global con Next.js (100 XP)

Migras tu mazmorra a Next.js y le agregas una tabla de puntajes global con persistencia real y una ruta de servidor que consulta un modelo de lenguaje para generar la descripción narrativa de cada partida.

En el `README.md` justificas, para cada página, qué estrategia de renderizado elegiste y por qué. Esa justificación vale tanto como el código.

Los 20 XP específicos: la elección de CSR, SSR o SSG está argumentada por página (6), la clave de API no aparece nunca en el cliente, verificable en el navegador (6), la frontera entre componentes de servidor y cliente está bien trazada (4), y la salida del modelo se valida antes de usarse (4).

**Misión secundaria (25 XP):** implementa respuesta en flujo (streaming) para la narración.

---

## Módulo 5 — Gestión de versiones, despliegue y proyecto práctico

**Zona 5: La Cumbre** · Sesiones 16 a 18 · 6 h teóricas + 6 h prácticas + 12 h autónomas

Ya sabes construir. Este módulo es sobre trabajar en equipo sin pisarse, publicar sin romper nada y medir en lugar de suponer. Es lo que separa a alguien que programa de alguien contratable.

### Sesión 16 — Git avanzado y colaboración real (2 h T + 2 h P)

Ramas, fusiones y conflictos: cómo se resuelven de verdad, no borrando el archivo del otro. `rebase` frente a `merge` y cuándo usar cada uno. Revisión de código como práctica profesional: qué se comenta, cómo se comenta sin ser desagradable, cómo se recibe una crítica al propio código. Hacemos un ejercicio de conflicto provocado a propósito, en parejas, porque la primera vez que te pasa en un proyecto real conviene que no sea la primera vez.

**Fuentes**
- Chacon, S., & Straub, B. *Pro Git* (2ª ed.). https://git-scm.com/book/en/v2
- GitHub. *About pull requests*. https://docs.github.com/en/pull-requests/reference/pull-requests
- Conventional Commits. https://www.conventionalcommits.org/en/v1.0.0/
- Preston-Werner, T. *Semantic Versioning 2.0.0*. https://semver.org/

### Sesión 17 — Persistencia en el cliente y CI/CD (2 h T + 2 h P)

`localStorage` y `sessionStorage`: qué guardan, sus límites y qué nunca se guarda ahí. IndexedDB para datos estructurados y volumen. Después el despliegue: qué es un pipeline, qué es GitHub Actions, cómo se escribe un flujo que valide, pruebe, compile y publique. Infraestructura sin servidor y en el borde, y por qué eso cambia el cálculo de costos.

**Fuentes**
- MDN Web Docs. *Web Storage API*. https://developer.mozilla.org/en-US/docs/Web/API/Web_Storage_API
- MDN Web Docs. *IndexedDB API*. https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API
- GitHub. *Quickstart for GitHub Actions*. https://docs.github.com/en/actions/get-started/quickstart
- Vercel. *Deployments*. https://vercel.com/docs/deployments

### Sesión 18 — Rendimiento, auditoría y sustentaciones (2 h T + 2 h P)

Core Web Vitals y qué mide cada métrica. Auditoría con Lighthouse y cómo leer el informe sin obsesionarse con el número. Optimización de imágenes, carga diferida, división del paquete. La última parte de la sesión, y la primera de la semana de finales, son las sustentaciones.

**Fuentes**
- Google. *Web Vitals*. https://web.dev/articles/vitals

### Misión 08 — Proyecto final (100 XP + 110 XP de sustentación)

Un juego web completo, en equipos de tres, desplegado y funcionando. Los requisitos técnicos mínimos son: Next.js con TypeScript, estado complejo bien manejado, persistencia real, consumo de al menos una API externa, integración de un servicio de IA, accesibilidad verificada, Core Web Vitals en verde y pipeline de CI/CD activo.

Los requisitos de proceso importan igual: cada integrante con commits significativos y verificables, todo el trabajo mediante Pull Requests con revisión de un compañero, tablero de tareas visible, y un `AUDITORIA.md` que documente qué generó la IA en el proyecto, qué estaba mal y cómo lo corrigieron.

La sustentación dura veinte minutos por equipo: cinco de demostración jugando en vivo, siete de recorrido por las decisiones de arquitectura, tres sobre la auditoría de IA, y cinco de preguntas. **Se pregunta a integrantes al azar sobre cualquier parte del código**, así que repartirse el trabajo sin entender lo que hizo el otro no funciona.

Distribución de los 110 XP de sustentación: dominio técnico demostrado al responder (40), calidad de las decisiones de arquitectura y su justificación (25), profundidad de la auditoría de IA (20), claridad de la comunicación (15) y evidencia de trabajo en equipo real (10).

**Misión secundaria (25 XP):** modo multijugador local o efectos de sonido accesibles con control de volumen.

---

## Cronograma completo

Dieciocho sesiones de cuatro horas. Las fechas exactas se fijan al inicio del semestre según el calendario académico.

| Sesión | Módulo | Tema de la noche | Entrega esa semana |
|---|---|---|---|
| 1 | 1 | Arquitectura web y HTTP | Misión 00 (en clase) |
| 2 | 1 | HTML5 semántico y accesibilidad | — |
| 3 | 1 | SEO técnico y asistentes de código | **Misión 01** · Quiz 1 |
| 4 | 2 | Cascada, caja y especificidad | — |
| 5 | 2 | Flexbox y Grid | Auditoría 1 |
| 6 | 2 | Mobile-first y Tailwind | **Misión 02** · Quiz 2 |
| 7 | 3 | JavaScript moderno y funcional | — |
| 8 | 3 | DOM, eventos y bucle de juego | **Misión 03** |
| 9 | 3 | Módulos y bucle de eventos | Quiz 3 |
| 10 | 3 | Promesas, async/await y REST | **Misión 04** · Auditoría 2 |
| 11 | 3 | TypeScript | **Misión 05** · Quiz 4 |
| 12 | 4 | React: componentes y estado | — |
| 13 | 4 | Hooks y ciclo de vida | Auditoría 3 |
| 14 | 4 | Next.js y renderizado | **Misión 06** · Quiz 5 |
| 15 | 4 | Integración de servicios de IA | — |
| 16 | 5 | Git avanzado y colaboración | **Misión 07** · Auditoría 4 |
| 17 | 5 | Persistencia y CI/CD | Quiz 6 · avance de proyecto |
| 18 | 5 | Rendimiento y sustentaciones | **Misión 08** + sustentación |

Las semanas restantes del calendario de 21 corresponden a inducción institucional, receso y cierre de notas.

---

## Correspondencia con los resultados de aprendizaje

| RA del curso | Dónde se desarrolla | Cómo se evidencia |
|---|---|---|
| **R1** — Optimizar arquitectura y rendimiento, garantizando procesamiento y despliegue eficiente (RA 1) | Módulos 1, 3 y 5 | Misiones 01, 04, 05 y 08; Core Web Vitals en verde y auditoría de Lighthouse en el proyecto final |
| **R2** — Diseñar interfaces interactivas y escalables con meta-frameworks, apoyándose en IA (RA 2) | Módulos 2 y 4 | Misiones 02, 06 y 07; justificación por escrito de estrategias de renderizado |
| **R3** — Integrarse en equipos aplicando ágil, control de versiones y auditoría de código (RA 6) | Módulo 5 y todo el semestre | Historial de Pull Requests, cuatro auditorías cruzadas, `AUDITORIA.md` y sustentación en equipo |

---

## Referencias

### Documentación oficial y estándares

Todos los enlaces fueron verificados el 4 de agosto de 2026.

**Protocolos y red**
IETF. *RFC 9110: HTTP Semantics*. https://www.rfc-editor.org/rfc/rfc9110.html
IETF. *RFC 9112: HTTP/1.1*. https://www.rfc-editor.org/rfc/rfc9112.html
MDN Web Docs. *An overview of HTTP*. https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Overview

**Marcado y accesibilidad**
WHATWG. *HTML Living Standard*. https://html.spec.whatwg.org/multipage/
W3C. *Web Content Accessibility Guidelines (WCAG) 2.2*. https://www.w3.org/TR/WCAG22/
W3C. *How to Meet WCAG (Quick Reference)*. https://www.w3.org/WAI/WCAG22/quickref/
MDN Web Docs. *HTML heading elements*. https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/Heading_Elements
Google Search Central. *SEO Starter Guide*. https://developers.google.com/search/docs/fundamentals/seo-starter-guide

**Estilos y maquetación**
W3C. *CSS Cascading and Inheritance Level 5*. https://www.w3.org/TR/css-cascade-5/
W3C. *CSS Flexible Box Layout Module Level 1*. https://www.w3.org/TR/css-flexbox-1/
W3C. *CSS Grid Layout Module Level 1*. https://www.w3.org/TR/css-grid-1/
MDN Web Docs. *Basic concepts of flexbox*. https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Flexible_box_layout/Basic_concepts
MDN Web Docs. *CSS grid layout*. https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Grid_layout
Tailwind CSS. *Styling with utility classes*. https://tailwindcss.com/docs/styling-with-utility-classes
Tailwind CSS. *Responsive design*. https://tailwindcss.com/docs/responsive-design

**JavaScript y TypeScript**
Ecma International. *ECMAScript Language Specification*. https://262.ecma-international.org/
MDN Web Docs. *JavaScript modules*. https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules
MDN Web Docs. *JavaScript execution model*. https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Execution_model
MDN Web Docs. *Using the Fetch API*. https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch
Microsoft. *The TypeScript Handbook*. https://www.typescriptlang.org/docs/handbook/intro.html

**Desarrollo de juegos en la web**
MDN Web Docs. *Anatomy of a video game*. https://developer.mozilla.org/en-US/docs/Games/Anatomy
MDN Web Docs. *Canvas API tutorial*. https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API/Tutorial
MDN Web Docs. *Window: requestAnimationFrame()*. https://developer.mozilla.org/en-US/docs/Web/API/Window/requestAnimationFrame

**React y meta-frameworks**
Meta. *React: Quick Start*. https://react.dev/learn
Meta. *Built-in React Hooks*. https://react.dev/reference/react/hooks
Vercel. *Next.js App Router: Getting Started*. https://nextjs.org/docs/app/getting-started
Vercel. *Server and Client Components*. https://nextjs.org/docs/app/getting-started/server-and-client-components

**Versionado, despliegue y rendimiento**
Chacon, S., & Straub, B. *Pro Git* (2ª ed.). Apress. https://git-scm.com/book/en/v2
GitHub. *About pull requests*. https://docs.github.com/en/pull-requests/reference/pull-requests
GitHub. *Quickstart for GitHub Actions*. https://docs.github.com/en/actions/get-started/quickstart
Conventional Commits 1.0.0. https://www.conventionalcommits.org/en/v1.0.0/
Preston-Werner, T. *Semantic Versioning 2.0.0*. https://semver.org/
Vercel. *Deployments*. https://vercel.com/docs/deployments
Google. *Web Vitals*. https://web.dev/articles/vitals
MDN Web Docs. *Web Storage API*. https://developer.mozilla.org/en-US/docs/Web/API/Web_Storage_API
MDN Web Docs. *IndexedDB API*. https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API
MDN Web Docs. *Lazy loading*. https://developer.mozilla.org/en-US/docs/Web/Performance/Guides/Lazy_loading
OWASP. *OWASP Top Ten*. https://owasp.org/www-project-top-ten/

### Fundamento pedagógico

El diseño de este curso se apoya en literatura revisada por pares. Todos los DOI fueron verificados el 4 de agosto de 2026.

**Aprendizaje basado en juegos y gamificación**
Deterding, S., Dixon, D., Khaled, R., & Nacke, L. (2011). From game design elements to gamefulness: Defining "gamification". *Proceedings of the 15th International Academic MindTrek Conference*, 9–15. https://doi.org/10.1145/2181037.2181040
Hamari, J., Koivisto, J., & Sarsa, H. (2014). Does gamification work? A literature review of empirical studies on gamification. *47th Hawaii International Conference on System Sciences*, 3025–3034. https://doi.org/10.1109/HICSS.2014.377
Sailer, M., & Homner, L. (2020). The gamification of learning: A meta-analysis. *Educational Psychology Review, 32*(1), 77–112. https://doi.org/10.1007/s10648-019-09498-w
Clark, D. B., Tanner-Smith, E. E., & Killingsworth, S. S. (2016). Digital games, design, and learning: A systematic review and meta-analysis. *Review of Educational Research, 86*(1), 79–122. https://doi.org/10.3102/0034654315582065
Hamari, J., Shernoff, D. J., Rowe, E., Coller, B., Asbell-Clarke, J., & Edwards, T. (2016). Challenging games help students learn: An empirical study on engagement, flow and immersion in game-based learning. *Computers in Human Behavior, 54*, 170–179. https://doi.org/10.1016/j.chb.2015.07.045

**Gamificación aplicada a la enseñanza de programación**
Zhan, Z., He, L., Tong, Y., Liang, X., Guo, S., & Lan, X. (2022). The effectiveness of gamification in programming education: Evidence from a meta-analysis. *Computers and Education: Artificial Intelligence, 3*, 100096. https://doi.org/10.1016/j.caeai.2022.100096
Zolezzi, D., Martini, L., Iacono, S., & Vercelli, G. V. (2026). Results of a gamification university experience for teaching web programming: The "Alice in Codeland" project. *IEEE Transactions on Education, 69*(2), 161–175. https://doi.org/10.1109/TE.2026.3668978
Costa, J. M. (2023). Using game concepts to improve programming learning: A multi-level meta-analysis. *Computer Applications in Engineering Education, 31*(4), 1098–1110. https://doi.org/10.1002/cae.22630
Di Nardo, V., Fino, R., Fiore, M., Mignogna, G., Mongiello, M., & Simeone, G. (2024). Usage of gamification techniques in software engineering education and training: A systematic review. *Computers, 13*(8), 196. https://doi.org/10.3390/computers13080196
Alhammad, M. M., & Moreno, A. M. (2018). Gamification in software engineering education: A systematic mapping. *Journal of Systems and Software, 141*, 131–150. https://doi.org/10.1016/j.jss.2018.03.065

**Motivación y compromiso**
Ryan, R. M., & Deci, E. L. (2000). Self-determination theory and the facilitation of intrinsic motivation, social development, and well-being. *American Psychologist, 55*(1), 68–78. https://doi.org/10.1037/0003-066X.55.1.68
Wang, W.-T., & Sari, M. K. (2025). Design fit in gamified online programming learning environment. *Learning and Instruction, 96*, 102087. https://doi.org/10.1016/j.learninstruc.2025.102087

**Inteligencia artificial generativa en educación en programación**
Prather, J., Denny, P., Leinonen, J., Becker, B. A., Albluwi, I., Craig, M., Keuning, H., Kiesler, N., Kohn, T., Luxton-Reilly, A., MacNeil, S., Petersen, A., Pettit, R., Reeves, B. N., & Savelka, J. (2023). The robots are here: Navigating the generative AI revolution in computing education. *ITiCSE-WGR '23*, 108–159. https://doi.org/10.1145/3623762.3633499
Denny, P., Prather, J., Becker, B. A., Finnie-Ansley, J., Hellas, A., Leinonen, J., Luxton-Reilly, A., Reeves, B. N., Santos, E. A., & Sarsa, S. (2024). Computing education in the era of generative AI. *Communications of the ACM, 67*(2), 56–67. https://doi.org/10.1145/3624720
Prather, J., Reeves, B. N., Leinonen, J., MacNeil, S., Randrianasolo, A. S., Becker, B. A., Kimmel, B., Wright, J., & Briggs, B. (2024). The widening gap: The benefits and harms of generative AI for novice programmers. *ICER '24*, 469–486. https://doi.org/10.1145/3632620.3671116
Finnie-Ansley, J., Denny, P., Becker, B. A., Luxton-Reilly, A., & Prather, J. (2022). The robots are coming: Exploring the implications of OpenAI Codex on introductory programming. *ACE '22*, 10–19. https://doi.org/10.1145/3511861.3511863
Alanazi, M., Soh, B., Samra, H., & Li, A. (2025). The influence of artificial intelligence tools on learning outcomes in computer programming: A systematic review and meta-analysis. *Computers, 14*(5), 185. https://doi.org/10.3390/computers14050185
Rubio-Manzano, C., Meza, J., Fernández-Santibáñez, R., & Vidal-Castro, C. (2025). *Teaching programming in the age of generative AI: Insights from literature, pedagogical proposals, and student perspectives* (preprint arXiv:2507.00108). https://doi.org/10.48550/arXiv.2507.00108

### Bibliografía del PIAA

Haverbeke, M. (2024). *Eloquent JavaScript: A modern introduction to programming* (4ª ed.). No Starch Press.
Flanagan, D. (2020). *JavaScript: The definitive guide* (7ª ed.). O'Reilly Media.
Cruse, D., & Boudreau, D. (2025). *Inclusive design for accessibility*. Packt.
Huyen, C. (2024). *AI engineering: Building applications with foundation models*. O'Reilly Media.

---

*Guía del curso Programación Web 1 (235G8F) · Universidad de Caldas · Semestre 2026-2*
