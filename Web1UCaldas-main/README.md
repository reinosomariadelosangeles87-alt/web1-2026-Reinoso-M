# Programación Web 1 · Base de conocimiento

**Código 235G8F** · 3 créditos · Semestre 2026-2
Universidad de Caldas · Facultad de Inteligencia Artificial e Ingenierías
Ingeniería en Informática, cuarto semestre

Aquí está todo el material del curso. Es tu punto de partida para cada sesión y tu referencia durante el semestre.

---

## Empieza por aquí

Si es tu primer día, lee estos dos documentos en este orden:

**[Guía del curso](GUIA-DEL-CURSO.md)** — cómo funciona la asignatura, el sistema de XP y niveles, las nueve misiones, el modelo de entregas en GitHub, el cronograma completo y las fuentes. Es el documento que define las reglas del juego, y conviene leerlo entero una vez antes de la primera sesión.

**[Sesión 1](sesiones/SESION-01.md)** — arquitectura web y protocolo HTTP. Tu primera noche.

---

## Qué hay en cada carpeta

### [`sesiones/`](sesiones/) — el contenido de las 18 clases

Un archivo por sesión con la teoría explicada, los ejemplos de código, las prácticas para que puedas repetirlas solo, tu misión de la semana con su rúbrica, y los errores que probablemente vas a cometer.

El [índice](sesiones/INDICE.md) te lleva a cualquier sesión y explica cómo está organizado cada archivo.

**Lee la sesión antes de la clase.** No hace falta que la domines: con haberla visto en diagonal, las cuatro horas de la noche rinden el doble.

### [`presentaciones/`](presentaciones/) — los cinco decks, uno por módulo

Las diapositivas que se proyectan en clase, para que puedas volver a ellas cuando repases. Están organizadas por módulo, no por sesión: cada deck cubre entre tres y cinco noches, y el encabezado de cada diapositiva indica a qué sesión pertenece.

### [`recursos/`](recursos/) — los once diagramas

Los diagramas del curso en PNG, por separado y en buena resolución. Están dentro de las presentaciones y de las sesiones, pero aquí los tienes sueltos por si quieres imprimirlos, usarlos para estudiar o pegarlos en tus propios apuntes.

El del bucle de eventos (`d06-event-loop.png`) es el que más vas a mirar.

### [`plantilla-de-entrega/`](plantilla-de-entrega/) — la estructura de tu repositorio

Esto no se lee: se copia. Es la base de tu repositorio de entregas, con el `README.md` de tu perfil de jugador, el `IA.md` que llenas en cada misión, la plantilla de Pull Request y el workflow de GitHub Actions que valida tus entregas.

En la sesión 1 lo vas a usar para crear tu propio repositorio.

---

## Cómo se aprueba el curso

El curso funciona como una campaña de cinco zonas. Acumulas **XP** y ese XP **es** tu nota, no un puntaje paralelo.

| | |
|---|---|
| XP total disponible | 1200 (más 150 opcionales) |
| XP para aprobar | 700 |
| Nota aprobatoria | 3.0 |
| Habilitable | No |

Ganas XP con nueve misiones de 100 puntos, seis quices de 15, cuatro auditorías de código de 25 y la sustentación final de 110. El detalle completo, con la tabla de niveles y las insignias, está en la [guía del curso](GUIA-DEL-CURSO.md).

### Tres reglas que conviene saber desde hoy

**Hay reintento.** Cualquier misión se puede repetir una vez y recuperar hasta el 80% del XP perdido. Una mala semana no te hunde el semestre.

**La IA está permitida y es parte del curso**, con una condición que no admite excepciones: todo código generado por un asistente se declara en el archivo `IA.md` de esa misión, explicando qué pediste, qué te devolvió, qué estaba mal y qué corregiste. Código de IA sin declarar se califica en **cero** y no admite reintento. Declararlo **no baja tu nota**: lo que se evalúa es tu capacidad de auditar, y sin la declaración no hay nada que evaluar.

**Las entregas van antes de la medianoche del día anterior a la sesión.** Cada día de retraso cuesta 10 XP, hasta un máximo de 40. Después de cuatro días la misión queda en cero, pero igual debes entregarla para tener derecho a presentar la siguiente.

---

## Lo que vas a construir

Nueve juegos, uno tras otro, cada uno con una tecnología nueva y todos publicados en internet con URL propia.

| # | Misión | Con qué | Horas |
|---|---|---|---|
| 00 | Registro de jugador | Git y GitHub | 1 |
| 01 | Ficha de personaje | HTML puro, sin CSS | 7 |
| 02 | Tablero adaptable | CSS y Tailwind | 9 |
| 03 | Juego de memoria | JavaScript | 10 |
| 04 | Trivia con API | fetch y async/await | 10 |
| 05 | Migración a TypeScript | TypeScript estricto | 8 |
| 06 | Mazmorra por turnos | React | 10 |
| 07 | Marcador global | Next.js y un LLM | 9 |
| 08 | Proyecto final | Todo, en equipos de 3 | 5 + equipo |

Las horas son el presupuesto de trabajo autónomo estimado, y suman las 72 horas no presenciales que fija el plan de la asignatura. Si una misión te está tomando bastante más de lo indicado, no es que seas lento: es que algo se atascó, y conviene preguntar en lugar de perder la noche.

---

## Cómo estudiar con este material

Cada archivo de sesión sigue la misma estructura, así que después de la primera ya sabes dónde buscar.

Si tienes **cinco minutos** antes de clase, lee la sección "Por qué este tema importa" y los apartados "Las ideas que hay que llevarse". Con eso llegas ubicado.

Si tienes **media hora**, lee la sesión completa y responde mentalmente las preguntas de "Ponte a prueba". Si alguna no la puedes responder, ya sabes qué preguntar en clase.

Si estás **atascado en una misión**, ve directo a "Errores que probablemente vas a cometer" de la sesión correspondiente. Son los cuatro fallos que aparecen todos los semestres en ese tema, y hay buenas probabilidades de que el tuyo esté ahí.

Si algo **no te cuadra**, ve a las fuentes al final de cada sesión. Son especificaciones oficiales y documentación de primera mano, no tutoriales. Acostumbrarte a leerlas es la habilidad que te va a servir cuando el framework de moda sea otro.

---

## Requisito previo

Fundamentos de las Tecnologías de la Información y la Comunicación (316G8F).

Se asume que manejas lógica de programación básica: variables, condicionales, ciclos y funciones. **No se asume nada** de HTML, CSS, JavaScript, Git ni línea de comandos.

---

## Lo que necesitas instalado

Antes de la primera sesión, ten esto funcionando en tu portátil:

**Git** — descarga en [git-scm.com](https://git-scm.com/). Verifica que funcione abriendo una terminal y escribiendo `git --version`.

**Visual Studio Code** — [code.visualstudio.com](https://code.visualstudio.com/)

**Un navegador reciente** — Chrome o Firefox.

**Una cuenta de GitHub** ya creada y con el correo confirmado. Usa un nombre de usuario que te sirva profesionalmente: este repositorio va a ser tu portafolio.

Llega con el portátil cargado.

---

*Material del curso Programación Web 1 (235G8F) · Universidad de Caldas · Semestre 2026-2*
