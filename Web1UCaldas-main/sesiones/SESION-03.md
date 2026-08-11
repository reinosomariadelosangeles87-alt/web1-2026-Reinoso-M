# Sesión 3 · SEO técnico y asistentes de código

**Módulo 1** · Zona 1: El Puerto de Entrada
**Lo que sale de esta noche:** tu ficha auditada con herramientas objetivas y la regla de *specs before code* instalada para el resto del semestre
**Lo que se entrega:** Misión 01 — Ficha de personaje (100 XP · 7 h), que ya conoces desde la sesión 1

Esta noche arranca con **Quiz 1**: tres preguntas, 15 XP, en los primeros diez minutos, sin repaso previo. Entra lo de la sesión 2 y la lectura previa.

---

## Por qué SEO y asistentes de IA caben en la misma noche

Parecen dos temas sin relación, y los une una sola idea: **una máquina solo entiende de tu trabajo lo que le declares explícitamente.** El buscador entiende tu página por sus metadatos y su estructura; el asistente de IA entiende tu problema por la especificación que escribes. En los dos casos, lo implícito se pierde, y lo que se pierde se rellena con adivinanzas.

La segunda mitad de la noche es, en la práctica, la más importante del curso completo. La regla de *specs before code* que se establece hoy gobierna las siete misiones siguientes, y es la competencia que la asignatura declara no negociable.

Hay además una razón para ponerlas en este orden. El SEO técnico es un tema donde puedes verificar tu trabajo con herramientas objetivas, y funciona como calentamiento para el tema difícil, que es aprender a desconfiar de un texto que suena bien.

---

## SEO técnico: lo que sí controlas

Antes de leer lo que sigue, respondete esto: **Google no le paga a nadie para leer tu página. ¿Cómo decide entonces de qué trata, y por qué la pondría antes que otra?**

La respuesta intuitiva menciona palabras clave y enlaces. El reencuadre útil tiene tres pasos, y el orden importa. Primero el buscador tiene que **descubrir** que la página existe, casi siempre por un enlace o un sitemap. Después tiene que **rastrearla**, es decir pedirla como haría cualquier navegador, con el mismo HTTP de la sesión 1. Y después tiene que **indexarla**, o sea decidir de qué trata y guardarla. Solo al final hay un ranking.

La mayoría de los problemas reales de SEO no son de ranking: son de que la página no se descubre, no se rastrea o no se indexa. De ahí la frase que ordena todo el tema: **el SEO técnico no es adivinar el algoritmo, es no bloquearle el paso al rastreador.**

### El título y la descripción

El `<title>` es lo que aparece como enlace azul en el resultado de búsqueda y en la pestaña del navegador. Único por página, descriptivo, con lo específico primero. El error clásico es un sitio de treinta páginas donde las treinta se llaman "Inicio", que es como publicar un libro donde todos los capítulos se titulan "capítulo".

La `<meta name="description">` no influye directamente en la posición, y aun así importa: es el texto que se lee bajo el enlace y decide si alguien hace clic o sigue de largo. Vale la pena separar esas dos cosas en tu cabeza, porque el mito de que la meta descripción "sube el ranking" está por todas partes.

### La estructura, que es la misma de la sesión pasada

Un solo `<h1>`, jerarquía sin saltos, elementos de sección, texto alternativo en las imágenes. Aquí se cierra el círculo del módulo: **todo lo que hiciste la semana pasada por accesibilidad te sirve idéntico para posicionamiento, porque un rastreador y un lector de pantalla son el mismo tipo de consumidor.** Ninguno de los dos ve la pantalla; los dos leen la estructura. Es la mejor noticia del módulo: no son dos trabajos, es uno.

### Las URLs

`/juegos/memoria` frente a `/p?id=4827`. Legible para una persona, legible en un enlace compartido, estable en el tiempo. Y hay una conexión directa con la sesión 1: cuando una URL cambia, el `301` es la forma de decirle al buscador y al navegador que el contenido se mudó y que el enlace viejo debe actualizarse. Sin el 301, todo lo acumulado por esa dirección se pierde.

### El rendimiento

Google declaró la velocidad como factor de posicionamiento, así que aquí no hay especulación. No hace falta entrar en detalle todavía, porque el módulo 5 tiene una sesión entera sobre Core Web Vitals; basta con quedarte con que la página lenta pierde posiciones y usuarios al mismo tiempo, y que las dos pérdidas ocurren por la misma causa.

### Los datos estructurados

Es marcado adicional que le dice explícitamente al buscador "esto es una receta, esto es un producto, esto es un videojuego con esta calificación", y es lo que produce esos resultados enriquecidos con estrellas y precios. El punto conceptual es el mismo de todo el tema: **lo que la máquina no puede inferir hay que declararlo.**

### Y lo que no controlas

El algoritmo cambia, la competencia cambia, y nadie fuera de Google conoce los pesos. Desconfía de cualquiera que venda "posición garantizada", porque lo único garantizable es el trabajo técnico. Hacer bien lo controlable y dejar de perseguir rumores es la postura profesional.

### Las ideas que hay que llevarse

**Accesibilidad y SEO son el mismo trabajo visto desde dos ángulos.** Un `alt` útil sirve a la persona ciega y a la búsqueda de imágenes. Una jerarquía correcta sirve al lector de pantalla y al indexador. Un `<title>` descriptivo sirve al resultado de búsqueda y a quien tiene veinte pestañas abiertas. Cuando dudes cuál priorizar, la respuesta es que no hay que elegir.

**Si el contenido no está en la respuesta, el buscador puede no verlo.** Esta idea hoy se planta y en el módulo 4 se cobra. Si toda la página se construye con JavaScript después de cargar, hay un momento en que el HTML que llegó estaba prácticamente vacío. Los buscadores modernos ejecutan JavaScript, pero con costos y demoras, y no siempre completan el trabajo. Es exactamente la razón por la que existen el renderizado en servidor y la generación estática, que son la sesión 14. Por ahora toma nota de que el HTML que llega en la respuesta importa; en el módulo 4 vas a decidir qué se renderiza dónde y esta será la razón.

### Ponte a prueba

*Si tu juego está publicado y nadie lo enlaza desde ninguna parte, ¿cómo se enteraría Google de que existe?* La respuesta lleva a sitemaps y enlaces, y a que "publicado" no es lo mismo que "descubrible".

*¿Por qué un sitio puede posicionar bien y ser inaccesible, si son el mismo trabajo?* Es una buena pregunta para matizar con honestidad: la superposición es grande pero no total, y hay criterios de accesibilidad —contraste, foco, teclado— que a un buscador le dan igual.

---

## Cómo trabajar con un asistente sin volverte su secretario

### Qué es realmente lo que tienes enfrente

Empecemos desarmando la metáfora equivocada. Un asistente de código no es un compilador ni un buscador: es un sistema que predice la continuación más probable de un texto, entrenado con enormes cantidades de código público. Eso explica de golpe sus dos caras. Es extraordinariamente bueno en lo que aparece muchas veces escrito de la misma forma, y es poco confiable en lo que es específico de tu problema, porque tu problema no está en el corpus.

De ahí sale la división que conviene tener presente. **Lo hace bien:** código repetitivo y andamiaje, conversiones de formato, recordar sintaxis olvidada, explicar un mensaje de error, escribir las pruebas de los casos obvios. **Lo hace mal:** la lógica del dominio, que no conoce; los casos borde, que por definición son raros y por lo tanto poco frecuentes en el entrenamiento; la seguridad; las decisiones de arquitectura; y en general cualquier cosa donde "parece correcto" y "es correcto" se separan.

Esto no es opinión del profesor, y vale la pena que sepas de dónde sale. Prather y colegas, en el informe de grupo de trabajo de ITiCSE de 2023, documentan que estas herramientas resuelven la mayoría de los ejercicios introductorios típicos, y advierten sobre el riesgo específico de la **dependencia sin comprensión**: el estudiante avanza en las tareas y no construye el modelo mental. Denny y colegas, en *Communications of the ACM* en 2024, plantean el reencuadre que este curso adopta como propio: si generar código es barato, lo que hay que enseñar y evaluar es **especificar, leer y verificar** código. Este curso está diseñado alrededor de esa tesis, y por eso la regla del `IA.md` no es desconfianza: es el diseño del programa.

### Specs before code, como regla operativa

Esta es la regla que se va a repetir el resto del semestre, así que memorízala en su forma final:

**Antes de pedirle código a un asistente, escribe en una frase qué debe hacer, con qué entradas, qué debe devolver y qué pasa cuando algo sale mal. Si no puedes escribir eso, todavía no sabes lo que quieres.**

Y el corolario, que es la parte que muerde: si no puedes escribir la especificación, el asistente te va a dar algo que parece correcto, y no vas a tener con qué darte cuenta. La petición vaga produce doscientas líneas que casi funcionan, y depurar código ajeno que casi funciona cuesta más que haberlo escrito.

Hay un beneficio secundario que conviene nombrar: la especificación es reutilizable. Sirve como prompt, sirve como comentario de documentación, sirve como lista de casos de prueba y, cuando llegues a TypeScript en la sesión 11, buena parte de ella se vuelve literalmente una firma de tipos. **Los tipos son una especificación ejecutable**, y esa frase se planta hoy para cobrarla en el módulo 3.

### El protocolo de las tres preguntas

Esto lo puedes aplicar mecánicamente esta misma noche. Ante cualquier bloque de código generado, tres preguntas antes de pegarlo.

La primera es *¿puedo explicar línea por línea qué hace esto?* Si no, no entra al proyecto. No por moral, por consecuencia práctica: cuando falle en la semana 12 vas a tener que arreglarlo y no vas a saber por dónde.

La segunda es *¿en qué caso esto se rompe?* Fuérzate a nombrar uno. El arreglo vacío, el nombre con tilde, la respuesta que llega a medias, el doble clic rápido. Los casos borde son donde el modelo falla con más frecuencia y son los que nadie pregunta.

La tercera es *¿esto es lo más simple que resuelve mi problema?* Los asistentes tienden a generar de más: dependencias que no necesitas, abstracciones para un caso que no tienes, configuración para un escenario que no es el tuyo. Código extra es deuda extra.

Estas tres preguntas son, literalmente, el contenido de la columna "qué estaba mal" de tu `IA.md`.

### Las ideas que hay que llevarse

**El modelo repite lo frecuente, no lo correcto.** Es la explicación de fondo de casi todos sus fallos, y la vas a ver en pantalla dentro de un rato con un ejemplo concreto.

**La especificación no mejora al modelo, mejora tu capacidad de revisar lo que te da.** Sin nada contra qué comparar la respuesta, "se ve bien" se vuelve el único criterio, y "se ve bien" es exactamente el criterio que estos sistemas están optimizados para satisfacer.

### Ponte a prueba

*Si le pides "hazme un formulario de contacto bonito y optimizado" y te devuelve sesenta líneas, ¿con qué criterio decides si están bien?*

*El asistente te da una función que funciona en tu prueba. Nombra un caso borde en el que se rompería, sin ejecutarla.*

---

## Práctica en clase: auditar tu ficha y escribir tu primera especificación

Esta práctica tiene dos mitades y la segunda se hace sobre el resultado de la primera. La mitad inicial es SEO y accesibilidad medidos con herramientas; la segunda es tu primer contacto formal con un asistente de código.

**Corre Lighthouse sobre tu ficha.** Abre las herramientas del navegador, ve a la pestaña Lighthouse y corre la auditoría con las categorías de SEO y accesibilidad. En clase lo hacemos sobre la entrega de alguien real, decente pero no perfecta, porque un 100 no enseña nada.

**Lee el informe y desármalo.** Cada fallo trae qué está mal y qué elemento lo causa. Recórrelos y arréglalos uno por uno en el editor. Casi siempre aparece un fallo de contraste o de `alt` que jurabas tener bien. Y hay una parte incómoda que conviene saber desde hoy: **Lighthouse detecta lo automatizable, que es más o menos un tercio de WCAG.** Un 100 sobre 100 no significa accesible; significa que no hay errores de los que una máquina puede encontrar sola. Lo demás se comprueba navegando con teclado y leyendo. Dicho de otro modo: la misión secundaria pide ese 100, y el número no es la meta.

**Rompe y arregla el título, en vivo.** Escribe un `<title>Inicio</title>` a propósito, corre Lighthouse otra vez, mira el aviso y arréglalo a algo descriptivo. Es un cambio de una línea con efecto medible, y te muestra el ciclo completo de medir, cambiar, medir.

**La bisagra: escribe la especificación antes de pedir nada.** Cierra el navegador y abre un archivo de texto. Nadie escribe un prompt hasta haber escrito una especificación. Esta es la forma, sobre algo concreto y pequeño como el bloque de metadatos de tu ficha:

```
Objetivo: bloque <head> completo para la ficha de personaje "Kael".
Entradas: nombre del personaje, una frase de resumen, la URL de la imagen.
Salida esperada: title único y descriptivo, meta description de 150 caracteres
  como máximo, meta charset, meta viewport, lang="es" en el html.
Restricciones: HTML5 válido, sin librerías, sin CSS, sin JavaScript.
Casos borde: si el nombre trae comillas o tildes, debe quedar bien escapado.
```

Escribir esas cinco líneas te obliga a decidir qué quieres **antes** de tener un texto plausible en pantalla empujándote a aceptarlo.

**Pide lo mismo mal, a propósito.** Este es el ejercicio más valioso del semestre. Pídele al asistente algo vago del estilo *"hazme el head de mi página de personaje bonito y optimizado para SEO"*. Va a devolver algo largo, con aspecto profesional, y con basura dentro. Lo que aparece casi siempre y hay que cazar:

Una `<meta name="keywords">`, que Google dejó de usar hace más de una década y que sigue apareciendo en el código generado porque está en millones de páginas viejas del corpus de entrenamiento. Es el ejemplo perfecto de que **el modelo repite lo frecuente, no lo correcto**.

Etiquetas de Open Graph o Twitter con URLs inventadas, dominios de ejemplo o rutas de imagen que no existen en tu proyecto.

A veces, un bloque de datos estructurados con campos que no corresponden al tipo declarado.

Deja los errores en pantalla un rato antes de borrarlos, y trata de decidir cuáles son sospechosos por tu cuenta. Es muy probable que defiendas la `meta keywords` porque suena razonable. Ese momento —creer que algo está bien porque suena bien— es exactamente lo que hay que aprender a detectar.

**Pide lo mismo bien.** Ahora pega la especificación de cinco líneas como prompt y compara las dos respuestas lado a lado. La segunda es más corta, más aburrida, y auditable en treinta segundos porque hay una lista contra la que verificarla.

**Escribe el `IA.md` en el momento.** Con lo que acabó de pasar en pantalla, llena las cuatro casillas: qué pedí, qué me dio, qué estaba mal, qué corregí. Toma dos minutos y es la prueba de que la exigencia no es burocrática: acabas de ver que hay algo real que declarar. Y si documentas un error real generado por IA con su corrección, te ganas la insignia 🔍 **Cazador de alucinaciones**, que da un día extra de plazo y es acumulable.

### Dudas que van a aparecer

**¿Usar IA está permitido?** Sí, está permitido y es parte del curso. Lo que no está permitido es no declararlo. Todo código generado se declara en `IA.md`: **sin declarar se califica en 0 y no admite reintento; declararlo NO baja la nota.** La razón es concreta: lo que se evalúa es tu capacidad de auditar, y si no hay declaración no hay nada que evaluar.

**¿El asistente "aprende" de mi corrección?** No, ni en la conversación siguiente ni en tu cuenta: cada conversación arranca sin memoria de las anteriores salvo que la herramienta la reinyecte. Es una analogía que ya tienes: es el mismo sin estado de HTTP de la sesión 1, y el contexto que la herramienta manda en cada turno cumple el papel de la cookie.

---

## Lo que se entrega esta noche

La **Misión 01 · Ficha de personaje** se entrega esta noche antes de la medianoche. Ya conoces el enunciado, la rúbrica y el presupuesto de horas desde la sesión 1, así que aquí solo lo administrativo: las correcciones y reintentos van dentro de la semana siguiente a recibir comentarios, y recuperan hasta el 80% del XP perdido. Si tu Pull Request tiene observaciones, ese rato es dinero.

Tu siguiente misión, la **Misión 02 · Tablero de juego adaptable**, queda asignada desde hoy y se entrega antes de la medianoche del día anterior a la sesión 7, con un presupuesto de **9 horas de trabajo autónomo**: una de planear, tres de CSS a mano, tres de Tailwind, una de probar y una de comparar. Se entrega dos veces el mismo diseño, y la comparación escrita en el `README.md` sobre cuál mantendrías dentro de un año vale 4 XP de los 20 específicos del módulo. No es un párrafo de relleno. El enunciado completo llega en la sesión 6.

Y hay algo que puedes hacer hoy mismo con lo aprendido esta noche: **escribe una especificación real de cada pieza que vas a necesitar en la Misión 02.** No código, la especificación. El tablero en cuadrícula, el panel de puntajes, los controles. Cinco líneas por pieza. Después somete cada una a las preguntas que las matan: *¿qué pasa si el tablero no es cuadrado?*, *¿de qué tamaño es la casilla en un celular de 360 píxeles?*, *¿qué se ve mientras no hay datos?*. Cuando no puedas responder, acabas de descubrir la parte del problema que no habías pensado, y ese descubrimiento es el objetivo del ejercicio.

Dos avisos más. En la sesión 4 empieza CSS, y vas a maquetar **tu propio HTML**; quien no entregó la Misión 01 maqueta el de otro, que es peor y es justo. Y en la sesión 5 hay **Auditoría 1**, cruzada, que vale 25 XP.

---

## Errores que probablemente vas a cometer

**Tratar el SEO como una lista de trucos y no como consecuencia de un buen marcado.** Es tentador buscar el atajo: cuántas veces repetir la palabra clave, qué meta agregar. Casi todo el SEO técnico que controlas es exactamente el trabajo de la sesión 2, y las técnicas de relleno de palabras clave llevan veinte años sin funcionar. Si necesitas una prueba, mira la `meta keywords` que el asistente te generó: es un fósil.

**Pegar código generado sin poder explicarlo, y darte cuenta tres semanas después.** Es el patrón que la literatura documenta como dependencia sin comprensión, y en este curso tiene una consecuencia física: el código que no entiendes reaparece en la misión siguiente, porque las misiones se construyen unas sobre otras. El caso concreto que te va a pasar: la Misión 05 pide migrar la Misión 04 a TypeScript, y quien no escribió su Misión 04 no puede migrarla.

**Escribir el `IA.md` después, de memoria y en genérico.** Va a salirte algo como "usé ChatGPT para ayudarme con el CSS" y nada más. No cumple, por una razón concreta: la columna que vale es "qué estaba mal", y esa no se puede reconstruir de memoria una semana después. El `IA.md` se llena **en el momento**, con copiar y pegar de lo que pasó. Toma dos minutos si lo haces ahí mismo y veinte si lo dejas para el final.

**Creer que un 100 en Lighthouse significa accesible o bien posicionado.** Las herramientas automáticas cubren lo automatizable, que en accesibilidad es una fracción del estándar. Un formulario puede tener 100 y ser imposible de llenar con teclado; una página puede tener 100 en SEO y no estar indexada porque nadie la enlaza. La regla: la herramienta encuentra errores, no certifica ausencia de errores.

---

## Fuentes de esta sesión

- Google Search Central. *SEO Starter Guide*. https://developers.google.com/search/docs/fundamentals/seo-starter-guide
- Prather, J., Denny, P., Leinonen, J., et al. (2023). The robots are here: Navigating the generative AI revolution in computing education. *ITiCSE-WGR '23*, 108–159. https://doi.org/10.1145/3623762.3633499
- Denny, P., Prather, J., Becker, B. A., et al. (2024). Computing education in the era of generative AI. *Communications of the ACM, 67*(2), 56–67. https://doi.org/10.1145/3624720

La guía de Google es la única fuente de primera mano sobre SEO técnico que vale citar; todo lo demás en internet es interpretación de ella o especulación. Los dos artículos son literatura revisada por pares y sostienen el diseño del curso: conviene que sepas que existen, porque cambian el estatus de la regla del `IA.md` de capricho a decisión fundamentada.

---

## Antes de la sesión 4

Lee la sección "Módulo 2, sesión 4" de `GUIA-DEL-CURSO.md` y la página de MDN sobre el modelo de caja, solo el diagrama y la explicación de las cuatro capas. Diez minutos.

Y llega con esta pregunta contestada como creas: *si le pongo `width: 300px` a una caja y además 20 píxeles de relleno, ¿cuánto mide la caja en pantalla?* Respóndela antes de buscar, porque en la sesión 4 vas a descubrir por qué la respuesta intuitiva es la equivocada.
