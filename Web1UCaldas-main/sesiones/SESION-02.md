# Sesión 2 · HTML5 semántico y accesibilidad

**Módulo 1** · Zona 1: El Puerto de Entrada
**Lo que sale de esta noche:** el esqueleto semántico de tu ficha de personaje pasando el validador del W3C, y la página recorrida con teclado al menos una vez
**Tu misión en curso:** Misión 01 — Ficha de personaje (100 XP · 7 h de trabajo autónomo), que se entrega antes de la sesión 3

---

## Por qué el marcado importa más de lo que parece

Casi con seguridad llegas con esta intuición: el HTML es la cosa que produce lo que aparece en pantalla, y por lo tanto cualquier marcado que dé el resultado visual correcto está bien. Es una intuición razonable y es falsa, y esta noche está dedicada a romperla.

El HTML no es una plantilla de dibujo: es una estructura de datos que consumen máquinas. Y ese cambio de mentalidad es el más importante del módulo 1, porque la Misión 01 lo evalúa directamente y porque en el módulo 4, cuando armes componentes de React, cada componente va a devolver marcado. El marcado malo se multiplica por la cantidad de veces que reutilices el componente.

Antes de seguir, respondete esto: **de todos los que van a "leer" tu página, ¿cuántos lo van a hacer mirando una pantalla?**

La respuesta intuitiva es "todos". La real es que por tu HTML pasan el lector de pantalla de una persona ciega, que no ve nada y recorre el documento saltando de región en región; el rastreador de Google, que no tiene ojos y decide de qué trata la página leyendo el marcado; el modo lectura del navegador, que intenta adivinar dónde está el contenido y dónde el adorno; el traductor automático; el asistente de voz; y tu propio celular cuando compartes el enlace por WhatsApp y aparece una tarjeta con título e imagen.

De ahí sale la frase que ordena la noche: **el ojo humano es solo uno de los consumidores de tu HTML, y es el único que puede perdonar un marcado malo.** Todos los demás no perdonan nada, porque lo único que tienen es la estructura.

---

## Marcar no es decorar

### Lo que significa "semántico"

Una etiqueta semántica dice qué es una cosa, no cómo se ve. `<nav>` no significa "una fila de enlaces horizontal"; significa "aquí hay navegación", y podría verse como una columna, un menú desplegable o nada en absoluto. `<h1>` no significa "texto grande y en negrita"; significa "este es el título de este contenido", y que el navegador lo pinte grande es una consecuencia, no la definición.

Esa distinción entre significado y apariencia sostiene todo lo demás, así que vale la pena leerla despacio.

Una analogía para fijarla: **un `div` es una caja de mudanza sin rotular.** Sirve perfectamente para llevar cosas de un lado a otro, pero cuando llegas al apartamento nuevo tienes cuarenta cajas idénticas y para encontrar la cafetera tienes que abrirlas todas. `<header>`, `<main>`, `<nav>` y `<footer>` son las mismas cajas con el rótulo puesto. El camión no las carga distinto; la diferencia aparece cuando alguien que no las empacó tiene que encontrar algo.

### Los elementos de sección

`<header>` es lo introductorio de la página o de una sección. `<nav>` marca bloques de navegación. `<main>` es el contenido principal, y va uno solo por documento. `<article>` es algo que tendría sentido por sí solo si lo sacaras de la página y lo publicaras aparte. `<section>` es un agrupamiento temático que normalmente lleva su propio encabezado. `<aside>` es contenido tangencial y `<footer>` es el cierre.

La distinción que más cuesta es `<article>` frente a `<section>`, y la prueba práctica que mejor funciona es preguntarte: *si esto apareciera solo, en un lector de noticias, sin nada alrededor, ¿seguiría teniendo sentido?* Si la respuesta es sí, es un `<article>`. La ficha de un personaje completa es un `<article>`; su bloque de estadísticas es una `<section>`.

### La jerarquía de encabezados

Esto es uno de los cinco puntos que califica la rúbrica, así que conviene ser directo. Los encabezados forman un índice, igual que la tabla de contenidos de un libro. Un lector de pantalla ofrece un atajo para listar todos los encabezados de la página y saltar al que interese: es la forma normal de explorar un documento largo cuando no puedes verlo. Si los niveles están mal, ese índice queda inservible.

Hay dos reglas concretas. La primera es que no se salta de nivel hacia abajo: después de un `<h2>` viene un `<h3>`, no un `<h4>`. Saltar equivale a un índice que anuncia un capítulo y de una vez una sub-sub-sección, sin decir de qué sección es hija. La segunda es la que rompe casi todo el mundo: **el nivel no se elige por el tamaño de la letra.** Si el `<h3>` se ve muy pequeño no se cambia a `<h2>`, se cambia el CSS. El nivel es estructura; el tamaño es presentación.

### El árbol de accesibilidad

Este es el concepto que amarra todo. El navegador, además del DOM, construye un segundo árbol pensado para las tecnologías de asistencia, donde cada elemento aparece con un rol, un nombre accesible y un estado. Cuando escribes `<button>`, el rol es "botón" gratis; cuando escribes `<div onclick>`, el rol es "genérico" y nadie que no vea la pantalla se va a enterar de que eso hace algo.

Dicho de una forma que conviene recordar: **la semántica no es documentación, es la entrada de un árbol que el navegador construye y que las herramientas de asistencia leen.**

### Las ideas que hay que llevarse

Son tres y las tres vuelven en el semestre.

**El HTML es una API, no una plantilla de dibujo.** Todo lo que no sea "un ojo mirando la pantalla" consume la estructura y nada más. Google, los lectores de pantalla, el modo lectura y las tarjetas de vista previa de las redes sociales leen lo mismo: etiquetas.

**Los elementos nativos traen comportamiento gratis, y reimplementarlo sale carísimo.** Un `<button>` responde al Enter y a la barra espaciadora, entra en el orden de tabulación, tiene foco visible, se anuncia como botón y funciona con control por voz. Un `<div>` con un escuchador de clic no hace nada de eso, y para igualarlo hay que agregar `tabindex`, un rol, el manejo de teclado y los estilos de foco: cuatro cosas que salen mal por separado. La regla en una frase: **si existe el elemento nativo, úsalo; solo se construye a mano lo que no existe.**

**`div` y `span` no son el enemigo.** Existen precisamente para cuando no hay significado que expresar, y a veces solo necesitas un contenedor para colgar un estilo o una cuadrícula. El error no es usar `div`, es usar `div` **cuando existía la etiqueta que decía algo**. Vale la pena tenerlo claro desde hoy, porque el error opuesto —convertir cada `div` en `<section>`— también cuenta como marcado mal puesto.

### Ponte a prueba

Si puedes responder estas tres sin releer, el bloque quedó.

*Si el CSS de una página no carga, ¿la página queda inservible?* Piensa en la idea de degradación: un documento bien marcado sigue siendo usable, feo pero navegable.

*¿Por qué `<b>` y `<strong>` se ven igual y no son lo mismo?* Uno pide negrita, el otro declara importancia, y el lector de pantalla puede cambiar la entonación con `<strong>` y no con `<b>`.

*Si un buscador tuviera que resumir tu ficha de personaje en una línea, ¿de dónde la sacaría?* Guarda tu respuesta: es el terreno del SEO técnico de la sesión 3.

---

## Práctica en clase: marcar la ficha de personaje

Este recorrido lo hacemos juntos en clase y sales de aquí con el esqueleto de tu Misión 01 empezado, pero está escrito para que puedas repetirlo solo. Necesitas dos cosas abiertas: tu archivo con Live Server o algo equivalente, y `validator.w3.org` con la opción de validar por carga de archivo.

**Escribe el documento mal a propósito.** Marca la cabecera de la ficha con `div` por todos lados y ponles clases con nombres visuales: `class="grande"`, `class="caja"`, `class="titulo-azul"`. Guarda y mira el resultado. Se ve mal, sí, pero funciona. Quédate con esa idea: esto funciona y está mal, y el resto de la práctica es sobre por qué.

**Comprueba que el navegador no entiende nada.** Abre las herramientas de desarrollo, ve a la pestaña de accesibilidad y busca el árbol de accesibilidad de ese fragmento. Todo aparece como genérico. No hay regiones, no hay encabezados, no hay nada para navegar. Este paso dura treinta segundos y es el más convincente de la noche.

**Traduce el mismo contenido a semántica.** Reemplaza los `div` por `header`, `nav`, `main`, `article`, `section` y `footer`, sin tocar el contenido ni agregar CSS. Guarda y recarga: la página cambia de aspecto sola, porque los estilos por defecto del navegador respetan la estructura. Vuelve al árbol de accesibilidad y ahora vas a ver regiones con nombre y una lista de encabezados. Mismo contenido, mismo esfuerzo, resultado distinto para todos los que no miran la pantalla.

**Rompe la jerarquía de encabezados a propósito.** Pon el nombre del personaje en `<h1>` y de una vez la sección de estadísticas en `<h4>`. Mira la pantalla: no se nota nada raro, solo cambia el tamaño de la letra. Ahora abre el panel de encabezados o una extensión de accesibilidad y mira el índice roto. Arréglalo a `<h2>`. El punto es que **este error es invisible al ojo y evidente a la máquina**, y por eso se valida en lugar de mirarse.

**Escribe la imagen y su texto alternativo.** Empieza con `alt="imagen"` y pregúntate qué información entrega eso a alguien que no ve la imagen. Ninguna, y es peor que nada, porque el lector de pantalla anuncia una imagen y después dice "imagen". Corrígelo a algo que describa lo que la imagen aporta en este contexto: `alt="Kael, guerrero élfico con armadura de placas y hacha a dos manos"`. Y aprende la regla del caso decorativo: si la imagen no aporta información, `alt=""` vacío es la respuesta correcta, porque le dice explícitamente al lector de pantalla que la salte. Un `alt` vacío es una decisión; un `alt` ausente es un olvido, y el navegador los trata distinto.

**Marca la tabla de estadísticas.** Aquí se aclara el malentendido de que "las tablas son malas". Las tablas son malas para maquetar y son exactamente lo correcto para datos tabulares, que es lo que son las estadísticas de un personaje. Escríbela con `<caption>`, `<thead>`, `<tbody>` y `<th scope="col">`. El `scope` se entiende con la escena del lector de pantalla leyendo celda por celda: sin encabezados asociados, quien escucha oye "12" y no sabe si es fuerza o destreza.

**Escribe el formulario y provoca el error del `label` suelto.** Pon primero un `<label>Nombre</label>` seguido de un `<input>`, sin `for` ni `id`. Se ve idéntico a un formulario correcto. Ahora haz clic sobre la palabra "Nombre": no pasa nada. Añade `for` e `id`, recarga y haz clic otra vez: el cursor salta al campo. **Ese clic que ahora funciona es la demostración física de que la asociación existe.** Y ten presente que el `placeholder` no reemplaza al `label`, porque desaparece al escribir y muchos lectores de pantalla no lo anuncian.

**Valida.** Sube el archivo al validador del W3C. Casi siempre queda algún error real: una etiqueta sin cerrar, un atributo mal escrito, un `<section>` anidado donde no iba. Arréglalo. Validar toma quince segundos y es un paso del proceso, no un examen final. La rúbrica de la Misión 01 da 5 XP por pasar el validador sin errores, y son los 5 XP más fáciles del semestre.

**Navega con teclado.** Guarda el ratón y recorre la página con Tab, mirando el foco moverse por los enlaces y los campos. Después vuelve al fragmento con `div` del principio y pásale Tab: el foco no entra. Ese contraste es el cierre de la práctica.

Cuando sigas solo con tu Misión 01, arranca con una instrucción que ahorra la mitad de los errores de jerarquía: **antes de escribir una etiqueta, escribe el índice de encabezados de tu ficha en un comentario al principio del archivo.** El nombre del personaje en `h1` y debajo la lista de las secciones con su nivel. Son cinco minutos bien invertidos.

### Dudas que van a aparecer

Vas a preguntarte si `<section>` y `<div>` no son lo mismo, dado que se ven igual. Se ven igual y no significan lo mismo: en el árbol de accesibilidad una `<section>` con encabezado aparece como región navegable y un `<div>` no aparece.

Vas a tener la tentación de poner varios `<h1>`. Déjalo en uno por documento: es lo que espera un buscador y es lo que evalúa la rúbrica.

Sobre `<b>`, `<i>` y `<u>`: usa `<strong>` para importancia y `<em>` para énfasis. Si lo que quieres es negrita por diseño, eso es CSS.

Y sobre por qué no usar simplemente Bootstrap o una plantilla: porque una plantilla te da apariencia y esconde la estructura, y este mes se evalúa la estructura. En el módulo 2 llega Tailwind y esa conversación se retoma con argumentos.

---

## WCAG 2.2 y por qué no es opcional

### Los cuatro principios

WCAG está organizado en cuatro principios, y la mnemotecnia en inglés es POUR. Vale la pena recorrerlos con un ejemplo de tu propia ficha de personaje en cada uno, porque así dejan de sonar a burocracia.

**Perceptible.** La información tiene que poder percibirse por más de un sentido. En tu ficha, eso es el `alt` de la imagen del personaje y el contraste suficiente entre el texto y el fondo. El criterio de contraste en nivel AA pide una relación de 4.5 a 1 para texto normal y 3 a 1 para texto grande, y es el fallo más común en todo el web: el gris claro sobre blanco que se ve elegante en un monitor bueno y desaparece en un celular al sol.

**Operable.** Todo se tiene que poder usar sin ratón. Si tu formulario se puede llenar y enviar solo con el teclado, y el foco se ve siempre, ese principio está cubierto. WCAG 2.2 agregó criterios en esta línea, entre ellos que el foco no quede tapado por un elemento fijo y que los objetivos interactivos tengan un tamaño mínimo razonable, precisamente porque las interfaces modernas rompían eso todo el tiempo.

**Comprensible.** Lenguaje claro, comportamiento predecible y errores de formulario que digan qué hacer. Aquí cabe una observación práctica: declarar `lang="es"` en el `<html>` es una línea y cambia cómo pronuncia el sintetizador de voz. Es el criterio más fácil de cumplir del estándar entero y casi nadie lo cumple.

**Robusto.** Que funcione con las tecnologías de asistencia actuales y futuras, lo que en la práctica significa marcado válido y roles correctos. Es la conexión directa con el validador de la práctica.

Los criterios tienen tres niveles: A, AA y AAA. El nivel que se exige en la práctica, y el que piden casi todas las normativas del mundo, es **AA**. Conviene saberlo porque al ver el número AAA es fácil asumir que esa es la meta.

### La parte legal, que cambia la conversación

Aquí el tema deja de sonar a buena voluntad. En Colombia, la **Ley 1618 de 2013** y la **Resolución 1519 de 2020** exigen accesibilidad en los sitios de las entidades públicas. No es una recomendación de estilo: es una obligación con la que te vas a topar el día que trabajes en cualquier proyecto del Estado, de una universidad pública, de una alcaldía o de un contratista de cualquiera de esos.

Y hay que ser franco sobre el incentivo: **la accesibilidad casi nunca la pide el cliente, pero cuando la audita alguien y el sitio falla, el problema es del que lo construyó.** Rehacer un formulario accesible cuesta una tarde; rehacer una aplicación entera cuesta un trimestre. Es la misma lógica de la seguridad: barato al principio, carísimo después.

Abre el *Quick Reference* del W3C y filtra por nivel AA. Vas a ver que no es un documento para leer de corrido sino una herramienta de consulta con criterios concretos y técnicas sugeridas. La diferencia entre "leer WCAG" y "usar el quickref para revisar una pantalla" es la diferencia entre tenerle miedo al estándar y trabajar con él.

### Las ideas que hay que llevarse

**El nivel que se exige es AA, y la mayor parte se consigue con marcado correcto.** Si tu documento valida, tiene jerarquía coherente, textos alternativos útiles, etiquetas asociadas y `lang` declarado, ya cumpliste una buena parte sin haber escrito una línea de CSS.

**La accesibilidad es una propiedad de las decisiones de estructura, no una capa que se agrega al final.** Cuando la estructura ya está tomada, arreglarla significa rehacerla.

### Ponte a prueba

*¿Qué relación de contraste pide el nivel AA para texto normal, y por qué el gris claro sobre blanco es el fallo más frecuente del web?*

*Tu formulario se ve perfecto y no se puede enviar sin ratón. ¿Qué principio de WCAG estás incumpliendo?*

---

## Errores que probablemente vas a cometer

**Elegir el nivel del encabezado por el tamaño que se ve.** Es el error número uno y viene de años de usar procesadores de texto. Vas a poner `<h4>` porque `<h2>` "se ve muy grande". El nivel es estructura, el tamaño es CSS, y todavía no estamos en CSS. Cuando dudes, abre el índice de encabezados en las herramientas del navegador y mira si el documento tiene sentido leído como una tabla de contenidos.

**Confundir semántica con "poner más etiquetas".** Después de esta clase es tentador convertir cada `div` en `<section>`, incluidas las cajas que solo existían para agrupar visualmente. Es la sobrecorrección clásica y produce un árbol de accesibilidad lleno de regiones sin nombre, que es peor que un `div` honesto. La prueba es siempre la misma: si no puedes ponerle un encabezado que tenga sentido, no era una sección.

**Escribir textos alternativos que describen el archivo y no la información.** `alt="foto.jpg"`, `alt="imagen del personaje"`, `alt="captura"`. El criterio útil es preguntarte qué le dirías a alguien por teléfono si tuvieras que explicarle esa imagen en una frase, y escribir eso. Y ojo con el caso simétrico: las imágenes decorativas llevan `alt=""`, porque el vacío intencional es una respuesta válida.

**Creer que la accesibilidad es una capa que se agrega al final.** Es el mismo error que "primero lo hago funcionar, después lo hago seguro", y no se puede: la accesibilidad depende de decisiones de estructura, y cuando la estructura está tomada, arreglarlo significa rehacerlo. En el módulo 4, con componentes de React, el costo de rehacer se multiplica.

---

## Fuentes de esta sesión

- WHATWG. *HTML Living Standard*. https://html.spec.whatwg.org/multipage/
- W3C. *Web Content Accessibility Guidelines (WCAG) 2.2*. https://www.w3.org/TR/WCAG22/
- W3C. *How to Meet WCAG (Quick Reference)*. https://www.w3.org/WAI/WCAG22/quickref/
- MDN Web Docs. *HTML heading elements*. https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/Heading_Elements

El HTML Living Standard es la autoridad final sobre qué significa cada elemento, y es un documento vivo: no tiene versiones, se actualiza. No hace falta leerlo completo. El *Quick Reference* de WCAG sí es de uso diario y conviene tenerlo en marcadores desde hoy.

Sobre la Misión 01, que se entrega antes de la medianoche del día anterior a la sesión 3: lo de esta noche es exactamente lo que se califica ahí. Y recuerda la regla de IA del curso, que en esta misión se aplica completa. Todo código generado por un asistente se declara en `IA.md`, con qué le pediste, qué te devolvió, qué estaba mal y qué corregiste. **Sin declarar se califica en 0 y no admite reintento. Declararlo no baja la nota**, porque lo que se evalúa es tu capacidad de auditar, y sin declaración no hay nada que evaluar. El detalle práctico que te sirve hoy: para HTML semántico los asistentes producen `div` anidados que se ven bien y están mal, así que si usas uno, revisar el marcado y no el aspecto es literalmente donde están los puntos.

---

## Antes de la sesión 3

Lee la sección "Módulo 1, sesión 3" de `GUIA-DEL-CURSO.md` y la introducción del *SEO Starter Guide* de Google, solo la parte de cómo encuentra Google el contenido. Diez minutos.

Llega con una idea formada sobre esta pregunta, porque con ella abre la sesión: *si Google no puede pagarle a nadie para que lea tu página, ¿cómo decide de qué trata?*

Y ten presente que la sesión 3 empieza con **Quiz 1**: tres preguntas, 15 XP, en los primeros diez minutos, y entra lo de esta noche.
