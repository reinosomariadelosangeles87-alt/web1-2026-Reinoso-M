# Sesión 18 · Rendimiento, auditoría y sustentaciones

**Módulo 5** · Zona 5: La Cumbre · última sesión del semestre
**Lo que sale de esta noche:** saber medir el rendimiento de una aplicación web y actuar sobre lo medido, y las primeras sustentaciones del proyecto final
**Tu misión final:** Misión 08 — Proyecto final (100 XP + 110 XP de sustentación · 5 h de trabajo autónomo individual, más el trabajo en equipo)

---

## Por qué el rendimiento cierra el curso

El tema técnico de esta noche es medir: saber cuánto tarda tu aplicación en aparecer, en responder y en dejar de moverse, y saber qué hacer con esos números. Es lo último que falta para que lo que construiste sea presentable delante de alguien que no sea tu profesor.

Y hay una honestidad que conviene decir de entrada: esta teoría llega tarde para aplicarla al proyecto que vas a defender esta misma noche. No importa. Tómala como **criterio de evaluación** y no como tarea de hoy: estás aprendiendo a leer un informe que se va a mirar cuando se califique tu proyecto, y que vas a necesitar en tu primer trabajo, donde alguien te va a mostrar un informe de Lighthouse y va a esperar que sepas qué hacer con él.

La segunda mitad de la noche son las sustentaciones. Si te toca hoy, salta al apartado sobre cómo prepararte y vuelve después.

---

## Core Web Vitals y cómo medir

Antes de leer lo que sigue, respondete esto: **¿cuándo está "cargada" una página?**

Piénsalo un minuto, porque la pregunta parece trivial y no lo es. ¿Cuando llegó el HTML? ¿Cuando se ve algo? ¿Cuando se ve el contenido principal? ¿Cuando se puede hacer clic y responde? Cada respuesta es una métrica distinta, y esa ambigüedad es exactamente por qué existen los Core Web Vitals: hubo que ponerse de acuerdo en qué medir.

De ahí sale la primera corrección a la intuición: **"cargó rápido" no es una cosa, son al menos tres cosas distintas**, y una página puede ser excelente en una y terrible en otra.

### Las tres métricas

**LCP · Largest Contentful Paint.** Mide cuánto tarda en aparecer el elemento de contenido más grande del área visible, que normalmente es la imagen principal o el bloque de texto del encabezado. Es la aproximación estándar a "¿cuándo el usuario siente que la página ya está?". Bueno por debajo de **2.5 segundos**.

Lo que suele empeorarlo: imágenes enormes sin optimizar, fuentes que bloquean el renderizado, y un servidor que tarda en responder el HTML inicial.

**INP · Interaction to Next Paint.** Mide cuánto tarda la página en responder visualmente cuando interactúas: un clic, un toque, una tecla. Bueno por debajo de **200 milisegundos**.

Aquí es donde el módulo 3 vuelve con fuerza, y vale decirlo explícitamente: **el INP malo casi siempre es la pila de llamadas bloqueada.** Si un manejador de eventos hace un cálculo pesado de forma sincrónica, el navegador no puede pintar nada hasta que termine, y el usuario percibe la interfaz como pegada. Es literalmente el event loop de la sesión 9, medido en milisegundos y con nombre comercial.

**CLS · Cumulative Layout Shift.** Mide cuánto se mueve el contenido mientras la página carga. No mide velocidad sino estabilidad. Bueno por debajo de **0.1**, que es un número sin unidades porque es una fracción del área visible desplazada.

La forma de entenderlo es una experiencia que ya viviste: vas a tocar un botón y en ese instante aparece un banner arriba, empuja todo hacia abajo, y terminas tocando otra cosa. Todos lo odiamos, aunque casi nadie sabía que tenía nombre.

La causa habitual es una imagen o un anuncio sin dimensiones declaradas: el navegador no sabe cuánto espacio reservar, así que maqueta sin ella y luego reacomoda todo cuando llega.

### Las ideas que hay que llevarse

**El número de Lighthouse es un resumen, no el objetivo.** Da una cifra del 1 al 100 y es tentador perseguir el 100 como si fuera un puntaje de videojuego. No lo hagas: **lo útil del informe es la lista de recomendaciones concretas**, no el número. Un 78 con las tres métricas en verde es mejor que un 95 con el CLS en rojo.

**Medir en tu máquina engaña.** Desarrollas en un portátil decente, con wifi de campus y el servidor corriendo en local. Tus usuarios reales pueden estar en un teléfono de gama media con datos móviles. Lighthouse tiene opciones para simular esas condiciones, y hay que usarlas: es la diferencia entre medir y engañarte.

### Ponte a prueba

*Si el INP de mi juego es malo, ¿qué parte del módulo 3 debería revisar?* El bloque de eventos, y en concreto qué tan pesados son tus manejadores. Si uno de ellos hace un cálculo largo de forma sincrónica, ahí está el problema.

*¿Por qué declarar el ancho y alto de una imagen mejora el CLS aunque no cambie nada visualmente?* Porque le permite al navegador reservar el espacio antes de tener el archivo, y así no tiene que reacomodar la maqueta cuando la imagen llega.

---

## Práctica en clase: auditar un proyecto real con Lighthouse

Traes el informe de Lighthouse de tu propio proyecto. Estos son los pasos, y están escritos para que puedas repetirlos sobre cualquier proyecto tuyo cuando quieras.

**Primero, corre Lighthouse.** Herramientas de desarrollo, pestaña Lighthouse, **modo móvil**, y ejecuta. Mientras carga, ten presente que está simulando a propósito un dispositivo más lento y una red peor que la tuya.

**Segundo, lee el número y déjalo pasar.** Míralo, anótalo, y pasa a lo siguiente de forma deliberada. El número no es el trabajo.

**Tercero, baja a las oportunidades.** Ahí está el contenido real. Lee las tres primeras recomendaciones y para cada una hazte la misma pregunta: *¿cuánto cuesta arreglar esto y cuánto mejora?* Casi siempre hay una que es trivial —declarar dimensiones de imágenes, añadir `loading="lazy"`— y otra que implica rediseñar algo. Aprender a distinguirlas es la habilidad que te llevas de este ejercicio.

**Cuarto, provoca un CLS visible.** Con tu proyecto abierto, limita la red a 3G lento en la pestaña Red y recarga. Vas a ver el contenido saltando mientras cargan las imágenes. Verlo ocurrir en tu propio proyecto convence mucho más que el diagrama.

**Quinto, arregla una cosa y vuelve a medir.** Si tienes una imagen sin dimensiones, agrégalas, recarga y mira el CLS bajar. Un arreglo medido de principio a fin, en dos minutos, es lo que deja la lección clavada: medir, cambiar, volver a medir.

**Sexto, revisa la sección de accesibilidad.** Lighthouse también audita accesibilidad, y esto cierra el círculo con la sesión 2. Si tu proyecto tiene botones sin nombre accesible o contraste insuficiente, ahí sale. La insignia ♿ del curso se conseguía justamente con esto.

**Dos cosas que te van a confundir.** Tu puntaje va a cambiar entre ejecuciones, porque Lighthouse mide en condiciones simuladas sobre una máquina real con carga variable; las variaciones de unos puntos son normales y no significan nada, lo que importa son las tendencias y las recomendaciones. Y tu proyecto va a puntuar mejor en escritorio que en móvil, que es lo esperado: el perfil móvil simula un procesador más lento y una red peor, y **es el que refleja la realidad de más usuarios.**

---

## Optimizar lo que la medición señaló

Antes de leer lo que sigue, respondete esto: **de todo lo que Lighthouse le recomendó a tu proyecto, ¿qué harías primero si solo tuvieras una hora?**

Esa es la pregunta profesional real. No se optimiza todo; se optimiza lo que más devuelve por el esfuerzo invertido. Y hay cuatro palancas que casi siempre valen la pena.

**Imágenes.** Es casi siempre la mayor ganancia por el menor esfuerzo. Sírvelas en formatos modernos, en el tamaño en que se van a mostrar y no en el original de la cámara, y **siempre con `width` y `height` declarados** para que el navegador reserve el espacio y no haya salto de maqueta. Una imagen de dos megabytes escalada por CSS a doscientos píxeles de ancho sigue costando dos megabytes de descarga.

**Carga diferida.** Lo que no se ve al abrir la página no tiene por qué descargarse todavía. `loading="lazy"` en imágenes que están más abajo es una línea de código y quita peso de la carga inicial. La excepción importante: **nunca lo pongas en la imagen principal**, porque es justo la que define el LCP y diferirla lo empeora.

**División del paquete.** Enviar todo el JavaScript de la aplicación en la primera carga significa que el navegador tiene que descargar, analizar y ejecutar código de pantallas que el usuario quizá nunca visite. Los meta-frameworks dividen el bundle por rutas automáticamente, y ahí está buena parte del valor de usar Next.js en lugar de armar todo a mano.

**Fuentes tipográficas.** Una fuente web puede bloquear el renderizado del texto: el navegador tiene el texto y el espacio, pero no lo pinta porque espera la fuente. El resultado es una página en blanco durante un segundo. Se resuelve con `font-display` y precargando la fuente crítica.

### Las ideas que hay que llevarse

**Optimizar sin medir antes es adivinar.** Es la versión de rendimiento de lo que ya viste con `useMemo` en la sesión 13. Primero mides, luego cambias una cosa, luego vuelves a medir. Si no puedes demostrar que mejoró, no sabes si mejoró.

**El rendimiento es una decisión de producto, no solo técnica.** Ese carrusel de imágenes pesa un megabyte y hace que la página tarde dos segundos más. La pregunta correcta no es cómo optimizarlo, es si vale lo que cuesta. En tu proyecto final, un juego que carga en cinco segundos pierde jugadores antes de que empiecen.

### Ponte a prueba

*Si `loading="lazy"` es bueno, ¿por qué no ponerlo en todas las imágenes?* Porque la imagen principal define el LCP y diferirla lo empeora justo en la métrica que más pesa. Es el mejor ejemplo de que las reglas de rendimiento tienen contexto y copiarlas sin entenderlas produce el efecto contrario.

---

## Misión 08 · Proyecto final (100 XP + 110 XP de sustentación)

Un juego web construido en equipo de tres. Los requisitos son de dos tipos y los dos pesan.

**Los requisitos técnicos:** Next.js con TypeScript, estado complejo bien manejado, persistencia real, al menos una API externa consumida, integración de un servicio de IA, accesibilidad verificada, Core Web Vitals en verde y un pipeline de CI/CD activo. Es, literalmente, el semestre completo en un solo repositorio.

**Los requisitos de proceso:** cada integrante con commits verificables en el historial, todo el trabajo entrando por Pull Request con revisión de un compañero, un tablero de tareas visible, y un `AUDITORIA.md` que documente qué generó la IA, qué estaba mal en lo que generó y cómo lo corrigieron.

Y aquí está la parte que conviene no malinterpretar: **los requisitos de proceso importan igual que los técnicos. Un juego perfecto con un solo commit de una sola persona no aprueba.** No es una amenaza retórica: el historial de Git se lee como un documento cuando se califica, y cuenta con precisión cómo trabajó el equipo.

Sobre la IA, la regla del curso es la misma de siempre y aquí es donde más se juega: todo código generado se declara en `IA.md`. Lo que no se declara se califica en **0 sin reintento**, y declararlo **no baja la nota**. El `AUDITORIA.md` es la versión ampliada de eso, y vale 30 de los 100 XP de la misión más 20 de los 110 de la sustentación: es el documento con más peso de todo el proyecto.

---

## Cómo prepararte para tu sustentación

La sustentación son veinte minutos por equipo: cinco de demostración jugando en vivo, siete de recorrido por las decisiones de arquitectura, tres sobre la auditoría de IA y cinco de preguntas. Vale 110 XP, más que cualquier misión del semestre. Estas son las cosas que de verdad cambian el resultado.

**Cualquiera del equipo tiene que poder explicar cualquier parte del código.** Esto no es un consejo de estilo, es la mecánica de la evaluación: **las preguntas se dirigen a integrantes al azar**, y quien recibe la pregunta es quien responde. Si el que escribió el sistema de combate es el único que lo entiende, el equipo pierde puntos cuando le preguntan al otro. La consecuencia práctica es que necesitas dedicar tiempo, antes de la sustentación, a que los tres se cuenten mutuamente qué hicieron y por qué. No es tiempo perdido: es la parte que vale 40 de los 110 XP.

**Ten la URL abierta y probada antes de empezar.** No desplegando en el momento, no instalando dependencias, no confiando en el wifi. Abre tu proyecto en una pestaña, juega una partida completa, déjala ahí. Si tu equipo llega diciendo que está desplegando, pasa al final del orden, y eso no le conviene a nadie.

**Ensaya la demostración de cinco minutos.** Cinco minutos jugando en vivo se pasan rapidísimo, y es fácil quemarlos mostrando la pantalla de inicio y explicando el menú. Decide de antemano qué tres cosas quieres que se vean —la mecánica principal, la persistencia funcionando, la integración con la API o con la IA— y practica el recorrido con un reloj. Un equipo que ensayó se nota en el primer minuto.

**Prepara respuestas sobre decisiones, no sobre sintaxis.** Las preguntas buenas son del tipo "¿por qué esta página es estática y esta otra se renderiza en el servidor?", no "¿qué hace `useMemo`?". Lo que se mide es comprensión, no memoria. Así que antes de la sustentación repasa cada elección de arquitectura que tomaron y ten lista la razón. Y no tiene que ser la elección que habría hecho el profesor: tiene que tener una razón defendible.

**Ten un error concreto de la IA listo para contar.** En los tres minutos de auditoría te van a pedir **un** error específico que generó la IA y cómo lo detectaron. Si la respuesta es genérica —"a veces se equivocaba"— esos 20 XP no están. Si cuentas el caso con detalle, qué pidieron, qué devolvió el modelo, por qué estaba mal y cómo lo descubrieron, están completos. Ese caso está en tu `AUDITORIA.md` si lo escribiste bien; si no lo escribiste, esta es la razón para hacerlo.

Así se reparten los 110 XP:

| Criterio | XP | Qué se busca |
|---|---|---|
| Dominio técnico al responder | 40 | Que cualquier integrante pueda explicar cualquier parte |
| Decisiones de arquitectura y su justificación | 25 | Que las elecciones tengan razón, no que sean las esperadas |
| Profundidad de la auditoría de IA | 20 | Un error concreto, detectado y corregido, no una frase |
| Claridad de la comunicación | 15 | Que se entienda sin ser experto en tu código |
| Evidencia de trabajo en equipo real | 10 | Historial de commits y PRs de los tres |

---

## Lo que te llevas del curso

Empecemos por lo concreto y verificable. **Te llevas nueve juegos desplegados**, cada uno con URL pública y código en GitHub. No es un certificado: es un portafolio que puedes mandar a una vacante mañana. **Te llevas nueve archivos `IA.md`** donde documentaste errores reales de la máquina, que es el registro escrito de haber aprendido a desconfiar con criterio. Y **te llevas el flujo profesional en los dedos**: ramas, pull requests, revisiones cruzadas, integración continua. Llegas a tu primer trabajo sabiendo cómo se colabora, que es algo que mucha gente aprende a los golpes en el primer mes.

Y la idea de fondo, dicha sin solemnidad.

**La tecnología concreta de este curso va a cambiar.** Next.js será otra cosa en cinco años. El modelo de lenguaje que uses será mucho mejor que el de hoy, y probablemente escribirá código que hoy nos parecería impecable. Nada de eso vuelve inútil lo que aprendiste, porque lo que no cambia es saber leer una especificación cuando la documentación no alcanza, entender **por qué** algo falla en lugar de probar cosas hasta que funcione, y no aceptar código que no comprendes por más plausible que se vea.

Esa última parte es la que este curso quería dejarte, y es la razón por la que cada misión pedía un `IA.md`. No era burocracia: era entrenamiento para el momento en que la herramienta sea tan buena que dé pereza revisarla.

---

## Errores que probablemente vas a cometer

**Perseguir el 100 de Lighthouse.** Te vas a obsesionar con el número y vas a dedicar horas a ganar seis puntos que ningún usuario percibe, mientras el LCP sigue en cuatro segundos. El número es un resumen ponderado de varias cosas; las recomendaciones son el trabajo real. Si tienes una hora, gástala en la primera oportunidad de la lista y no en subir la cifra.

**Medir solo en tu propia máquina y en escritorio.** Tu portátil con wifi de campus no es el teléfono de gama media con datos móviles de tus usuarios. Si no usas el perfil móvil con red limitada, estás midiendo un escenario que casi nadie vive, y los números que obtengas no significan nada fuera de tu escritorio.

**Poner `loading="lazy"` en la imagen principal.** Vas a aplicar la regla sin el matiz y vas a empeorar el LCP precisamente en el elemento que lo define. Es el mejor ejemplo del semestre de que copiar una recomendación sin entender qué mide produce el efecto contrario al buscado.

**Creer que el rendimiento se arregla al final.** Vas a llegar con un proyecto que carga en seis segundos esperando resolverlo la noche anterior. Algunas decisiones —qué se renderiza en el servidor, cómo se divide el bundle, qué imágenes se cargan— son arquitectónicas, y cambiarlas al final significa rehacer. Medir temprano cuesta menos, y esa frase aplica a todo lo que aprendiste este semestre.

---

## Fuentes de esta sesión

- Google. *Web Vitals*. https://web.dev/articles/vitals
- MDN Web Docs. *Lazy loading*. https://developer.mozilla.org/en-US/docs/Web/Performance/Guides/Lazy_loading
- Denny, P., Prather, J., Becker, B. A., Finnie-Ansley, J., Hellas, A., Leinonen, J., Luxton-Reilly, A., Reeves, B. N., Santos, E. A., & Sarsa, S. (2024). Computing education in the era of generative AI. *Communications of the ACM, 67*(2), 56–67. https://doi.org/10.1145/3624720

---

## Después de esta sesión

Las sustentaciones de los equipos que no alcanzaron a presentar hoy continúan en la semana de finales institucional, con el mismo formato de veinte minutos y la misma rúbrica de 110 XP. El horario se publica en el aula virtual con al menos tres días de antelación, así que revísalo y confirma tu franja.

Si te toca sustentar en finales, tienes días extra y conviene usarlos en lo que más pesa: que los tres puedan explicar todo el código, que el `AUDITORIA.md` tenga casos concretos y que la demostración esté ensayada. No en agregar una funcionalidad más.

Y después, la parte que ya no es del curso: tienes nueve URLs públicas y nueve repositorios. Ponlos en un lugar donde alguien pueda encontrarlos.
