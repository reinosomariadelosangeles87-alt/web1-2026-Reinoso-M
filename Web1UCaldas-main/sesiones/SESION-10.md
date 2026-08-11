# Sesión 10 · Promesas, async/await y APIs REST

**Módulo 3** · Zona 3: La Sala de Máquinas
**Lo que sale de esta noche:** una petición real a una API pública funcionando, con los tres estados visibles en pantalla, y tu propia página rompiéndose al desconectar el wifi
**Tu misión al terminar:** Misión 04 — Trivia con API externa (100 XP · 10 h de trabajo autónomo). Esta noche también hay **Auditoría 2 cruzada** (25 XP)

---

## Por qué esta noche cambia lo que asumes de tu código

Hasta esta noche, todo lo que has programado ocurría dentro de tu máquina y era determinista. Un `filter` no falla. Un `addEventListener` no se cae porque el usuario esté en un ascensor.

Desde esta noche, tu programa depende de un servidor que no controlas, de una red que se corta, de un servicio que un día devuelve otra estructura de datos sin avisar. Así que el objetivo declarado es que sepas consumir una API REST con `fetch`, manejar promesas con `async`/`await` y coordinar varias peticiones con `Promise.all`; el objetivo real es más difícil de conseguir: **que empieces a escribir código que asume que las cosas van a fallar.**

La diferencia entre un ejercicio de clase y algo que le podrías mostrar a alguien no es la interfaz ni la cantidad de funcionalidades: es que uno se queda en blanco cuando el servidor responde mal y el otro dice "no pudimos cargar las preguntas, intenta de nuevo".

La sesión 9 fue la más importante conceptualmente; esta es la más importante en términos de oficio, y se apoya entera en la anterior. Cada vez que aparezca un `await`, tienes que poder señalar el dibujo de la semana pasada y decir en qué caja está cayendo ese callback. Si `async`/`await` se te queda como "la palabra que hay que poner para que funcione", perdimos las dos sesiones.

Hay dos ideas que si no salen de aquí instaladas te van a costar puntos en la Misión 04: que **`fetch` no rechaza en un 404** y que **tres `await` en fila cuando podrían ir en paralelo es un error de rendimiento, no una preferencia de estilo.**

Y un anzuelo para que lo lleves anotado mientras lees, sin resolverlo todavía:

```javascript
const r = await fetch("https://ejemplo.com/pregunta-que-no-existe");
// El servidor responde 404. ¿Entra al catch?
```

¿Levantarías la mano diciendo que sí? Casi todo el mundo lo hace. Anótalo y sigue leyendo.

---

## Promesas y `async`/`await`

Antes de leer lo que sigue, respondete esto desde tu propia experiencia: **¿alguna vez escribiste un `setTimeout` dentro de otro `setTimeout`? ¿Y tres, uno dentro de otro? ¿Cómo se veía eso?**

Así se ve el problema:

```javascript
cargarCategorias(function (categorias) {
  cargarPreguntas(categorias[0], function (preguntas) {
    cargarImagen(preguntas[0], function (imagen) {
      mostrar(imagen);
    }, manejarError);
  }, manejarError);
}, manejarError);
```

¿Dónde iría el manejo de errores si algo falla en el segundo nivel? Va en cada nivel, repetido, y si el error del tercer nivel tiene que llegar al primero hay que pasarlo a mano. Eso tiene nombre y lo vas a encontrar en internet con estas dos palabras: **callback hell**, o la pirámide de la perdición. Y el problema de fondo no es la estética de la indentación: es que **el error no puede propagarse.** En código síncrono, un error sube solo por la pila hasta que alguien lo atrapa. Con callbacks anidados, no sube: hay que llevarlo cargado.

Las promesas existen para eso.

### Qué es una promesa

Una promesa es **un objeto que representa un valor que todavía no tienes**. No es el valor: es el comprobante de que va a llegar un valor, o de que va a llegar una explicación de por qué no llegó.

La analogía es **el buzón que te dan en un restaurante de comidas rápidas.** Pides tu pedido y te entregan un aparatito que vibra. Ese aparato no es la comida. No puedes comértelo. Pero es real, lo tienes en la mano ahora mismo, puedes guardarlo en el bolsillo, pasárselo a un amigo, e incluso decirle a alguien "cuando esto vibre, ve tú a recogerlo". Tiene exactamente tres situaciones posibles: **está esperando** (*pending*), **vibró y hay comida** (*fulfilled*), o **vinieron a decirte que se acabó el pollo** (*rejected*). Y hay una propiedad crucial: **una vez que el buzón pasó de esperando a cualquiera de los otros dos estados, no vuelve atrás nunca.** La promesa se resuelve una sola vez y su resultado queda congelado. Si le vuelves a preguntar dos horas después, te da la misma respuesta.

Los tres estados se llaman `pending`, `fulfilled` y `rejected`. Y la palabra que los engloba cuando ya no están pendientes es `settled`, que vas a necesitar cuando aparezca `allSettled`.

Lo que hace que una promesa sea mejor que un callback es que **el registro de lo que quieres hacer con el resultado está separado de la operación**. Con `.then` dices qué hacer si llega bien, con `.catch` qué hacer si llega mal, y lo escribes después, encadenado, no anidado:

```javascript
obtenerCategorias()
  .then((categorias) => obtenerPreguntas(categorias[0]))  // devolver una promesa aplana la cadena
  .then((preguntas) => mostrar(preguntas))
  .catch((error) => mostrarError(error));                 // UN catch para toda la cadena
```

Dos cosas que merecen que te detengas. La primera: **si dentro de un `.then` devuelves otra promesa, la cadena espera por ella** y el siguiente `.then` recibe su valor, no la promesa. Eso es lo que convierte la pirámide en una lista. La segunda, y es la que resuelve el problema de fondo: **un solo `.catch` al final atrapa el error de cualquier eslabón de la cadena.** El error se propaga solo, como en el código síncrono. Eso es lo que las promesas realmente arreglaron.

### `async`/`await`: la misma cosa, escrita para leerse

Sin misterio: **`async`/`await` es azúcar sintáctica sobre las promesas. No hay un mecanismo nuevo. Es la misma maquinaria con otra ortografía.**

```javascript
async function iniciarPartida() {
  try {
    const categorias = await obtenerCategorias();
    const preguntas  = await obtenerPreguntas(categorias[0]);
    mostrar(preguntas);
  } catch (error) {
    mostrarError(error);
  }
}
```

La equivalencia, pieza por pieza: `await` es `.then`, el `catch` del `try` es `.catch`, y una función marcada `async` **siempre devuelve una promesa**, aunque adentro escribas `return 5`; quien la llame recibe una promesa que se resuelve en 5. Ese último punto es el que produce más errores en las misiones: llamas una función `async` sin `await` y te preguntas por qué obtuviste `Promise { <pending> }` en lugar de datos.

Y ahora la parte que amarra esta sesión con la anterior. **Cuando el código llega a un `await`, ¿qué está haciendo el hilo de JavaScript durante esos cien milisegundos?**

La respuesta que se da siempre es "esperando". Es la respuesta equivocada, y es la equivocada más importante de la noche. **Nada espera.** `await` no bloquea nada. Lo que hace es más raro y más elegante: la función se **suspende**, su marco sale de la pila, el resto del programa sigue corriendo con total normalidad —los clics responden, las animaciones siguen—, y cuando la promesa se resuelve, **el resto de la función se encola como una microtarea.** La fila de arriba del dibujo de la semana pasada. Ahí es donde vive todo lo que viene después de un `await`.

La analogía que cierra esto: **`await` es poner un marcador en el libro y salir del cuarto.** No es quedarse mirando la página. El libro queda abierto exactamente donde lo dejaste, otra persona puede usar el cuarto mientras tanto, y cuando vuelves retomas en la misma palabra. La sensación de "el código se detuvo aquí" es una ilusión perfectamente construida por el lenguaje para que puedas leerlo de arriba abajo, y es una ilusión útil. Pero es una ilusión, y las consecuencias de creerla al pie de la letra son el tema de la segunda mitad de esta sesión.

Pruébalo, porque verlo vale más que la explicación:

```javascript
async function demo() {
  console.log("antes del await");
  await new Promise((r) => setTimeout(r, 3000));   // "espera" 3 segundos
  console.log("después del await");
}
demo();
console.log("¡yo corrí sin esperar a nadie!");
// antes del await
// ¡yo corrí sin esperar a nadie!
// (tres segundos después) después del await
```

Y mientras corren esos tres segundos, haz clic en un botón de la página. Responde. Compáralo con el experimento del `while` de la semana pasada, donde nada respondía. **Misma duración, comportamiento opuesto**, y la diferencia es si la pila estaba ocupada o libre.

### Las ideas que hay que llevarse

**Una promesa tiene tres estados y se resuelve una sola vez.** No es un flujo de eventos, no se puede reiniciar y no cambia de opinión. Si necesitas volver a pedir algo, haces una promesa nueva; no reutilizas la anterior.

**`async`/`await` no es un mecanismo distinto de las promesas, es otra forma de escribirlas.** Se pueden mezclar en el mismo archivo sin problema, y vas a tener que leer las dos formas porque la documentación y el código ajeno usan ambas.

**Lo que va después de un `await` es una microtarea.** Esta frase conecta las dos sesiones del módulo. `await` no detiene el programa: suspende una función y encola su continuación en la fila de arriba.

### Ponte a prueba

*Si una función `async` no tiene ningún `await` dentro, ¿sigue devolviendo una promesa?* Sí, y entender por qué separa la palabra `async` (que habla del valor de retorno) de la palabra `await` (que habla de suspender).

*¿Puedes usar `await` fuera de una función `async`?* En un módulo de ES, en el nivel superior, sí, y eso se llama top-level await. En un script clásico, no. La decisión de la semana pasada de usar módulos te dio algo a cambio.

*Si tienes una promesa guardada en una variable y le haces `await` dos veces, ¿se pide dos veces?* No. La promesa ya está resuelta y devuelve el mismo valor al instante. Esta pregunta prepara la secundaria opcional de caché de la Misión 04.

---

## Práctica en clase: la primera petición honesta

Este recorrido se hace en un archivo nuevo, `api.js`, con el navegador y **la pestaña de red abierta todo el tiempo**. La pestaña de red es el instrumento de esta noche, como la consola fue la de la sesión 7. Ténla abierta.

Usa una API pública sin autenticación, de las que devuelven preguntas de trivia o datos de prueba, y verifícala antes de empezar.

**Primero, la petición mínima, y la primera sorpresa.** Escribe esto y ejecútalo:

```javascript
const respuesta = await fetch(URL_API);
console.log(respuesta);        // un objeto Response, NO los datos
console.log(respuesta.json);   // una función
```

¿Dónde están los datos? No están. Lo que llegó es un objeto `Response` con el estado, los encabezados y el cuerpo **sin leer todavía**. Explora en el objeto las propiedades `ok`, `status` y `body`. Hace falta un segundo paso porque el cuerpo puede ser enorme y puede venir llegando por partes, así que leerlo y convertirlo de texto a objetos de JavaScript es una operación que **también devuelve una promesa**.

```javascript
const datos = await respuesta.json();   // segundo await, y es obligatorio
```

**Segundo, prueba el error del `.json()` sin `await`.** Quítale el `await` y observa: en consola aparece `Promise { <pending> }` seguido de `undefined` cuando intentas leer una propiedad. Es el error número uno de la semana entrante y verlo ahora con nombre y apellido te ahorra una hora.

**Tercero, los tres estados, en la interfaz y no en la consola.** Aquí está el corazón de la práctica.

```javascript
// vista.js — tres funciones, tres estados, y NINGUNA decide nada
export function mostrarCargando() { /* pinta el spinner, oculta lo demás */ }
export function mostrarError(mensaje) { /* pinta el mensaje y un botón de reintentar */ }
export function mostrarPreguntas(preguntas) { /* pinta el juego */ }
```

```javascript
// main.js
async function cargar() {
  mostrarCargando();                        // 1. ANTES de pedir, no después
  try {
    const preguntas = await obtenerPreguntas();
    mostrarPreguntas(preguntas);            // 2. éxito
  } catch (error) {
    console.error(error);                   // el detalle técnico va a la consola
    mostrarError("No pudimos cargar las preguntas. Revisa tu conexión.");  // 3. el humano ve esto
  }
}
```

Dos cosas que se califican. El `mostrarCargando()` va **antes** del `await`, no dentro del `try` después de la petición; si va después ya no indica nada. Y el mensaje que ve el usuario **no es el mensaje del error**: un `TypeError: Failed to fetch` no le dice nada a nadie. El detalle técnico va a la consola para ti; a la pantalla va una frase en español que dice qué pasó y qué puede hacer.

**Cuarto, el momento del wifi.** Este es el ejercicio que más se recuerda del semestre. Con la página cargada y funcionando, **apaga el wifi de tu portátil** y recarga. Observa la página, no la consola, durante quince segundos.

Lo que casi siempre pasa: **la página se queda en blanco y no dice absolutamente nada.** Ni un mensaje, ni un spinner, ni una pista. Solo blanco. ¿Qué pensaría un usuario cualquiera frente a esa pantalla? Pensaría que la aplicación está mal hecha, y tendría razón, porque desde afuera un error silencioso y un programa roto son indistinguibles.

Vuelve a poner el wifi, agrega el `try`/`catch` con el estado de error si no lo tenías, y **repite el experimento con el wifi apagado**. Ahora aparece el mensaje. Esa diferencia entre las dos versiones es la mitad de la nota de la Misión 04.

Mira también en la pestaña de red qué se ve cuando no hay red: la petición marcada como fallida, sin código de estado. Y fíjate en qué error atrapó el `catch`: un `TypeError`, que es lo que `fetch` lanza cuando la petición **no salió**. Guarda ese dato, porque en la sección siguiente es la clave de la trampa.

**Quinto, el estado de carga que dura demasiado.** Simula una red lenta desde la pestaña de red con la limitación de velocidad. Es la única forma de ver tu propio spinner durante más de un pestañeo, y suele revelar que el spinner nunca se ocultaba o que aparecía y desaparecía tan rápido que producía un parpadeo feo.

### Cuatro cosas que se te van a atravesar

CORS, y va a aparecer como un muro. Si eliges una API que responde con un error de política de origen cruzado, esto es lo que pasa: el navegador, por seguridad, no le deja a tu página leer la respuesta de otro dominio a menos que ese dominio lo autorice con un encabezado. **No es un error de tu código y no lo puedes arreglar desde tu código.** La solución en este curso es elegir otra API, de las que sí autorizan. Y el dato importante: un fallo de CORS **sí hace que la promesa se rechace**, así que sí entra al `catch`.

Qué significa que la API sea "REST". Que hay una dirección por cada recurso, que se le pide con un verbo del protocolo HTTP —`GET` para leer, `POST` para crear— y que la respuesta suele venir en JSON. En este curso solo vas a hacer `GET`, y si necesitas pasar parámetros van en la URL.

La confusión entre JSON y un objeto de JavaScript. JSON es **texto** con una forma parecida, y por eso hay que convertirlo. La diferencia se vuelve visible cuando te llega un JSON mal formado y el `.json()` truena con un error de sintaxis. Ese error sí entra al `catch`.

Y `axios`, que lo vas a ver en algún tutorial. Es una biblioteca, hace lo mismo con menos ceremonia, y **entre otras cosas sí rechaza en los códigos de error** —lo cual es justamente la razón por la que hay que entender primero por qué `fetch` no lo hace—. En este curso se usa `fetch`, que está en el navegador y no hay que instalar.

---

## `fetch`, códigos de estado y `Promise.all`

### La trampa: `fetch` no rechaza en un 404 ni en un 500

Vuelve al anzuelo del comienzo. La respuesta es: **no entra al `catch`. La promesa se cumple.**

Compruébalo con una URL que sepas que da 404:

```javascript
try {
  const r = await fetch(URL_QUE_NO_EXISTE);
  console.log("¿llegué aquí?", r.ok, r.status);   // llegué aquí. false, 404
  const datos = await r.json();                    // y aquí truena, con un error absurdo
} catch (e) {
  console.log("el catch dice:", e.message);        // algo sobre JSON inesperado
}
```

El razonamiento es coherente aunque incómodo. **`fetch` promete entregarte una respuesta HTTP, no promete que la respuesta te guste.** Desde el punto de vista del protocolo, un 404 es un éxito rotundo: preguntaste algo, el servidor te entendió, te contestó correctamente y su contestación fue "eso no existe". La comunicación funcionó perfectamente. Un 500 es lo mismo: el servidor te informó con toda claridad que se rompió por dentro. En los dos casos hubo pregunta, hubo respuesta, hubo entendimiento.

La analogía: **`fetch` es el mensajero, no el remitente.** Si le pides que lleve una carta y vuelve con un sobre que dice "el destinatario no vive aquí", el mensajero **hizo su trabajo**. Fue, entregó, trajo respuesta. No tiene sentido regañarlo. El mensajero solo falla si nunca llegó a la puerta: si la dirección no existe en el mapa, si el camino estaba cerrado, si se le acabó la gasolina. Eso —y solo eso— es lo que hace rechazar la promesa de `fetch`: que la petición **no haya salido o no haya vuelto**. Sin red, DNS que no resuelve, CORS que bloquea la lectura, la petición cancelada. Todo lo demás, incluidos todos los códigos de error del servidor, son respuestas exitosas del protocolo y **el `catch` nunca se entera**.

La consecuencia práctica es lo que hace que este tema sea la trampa más importante del módulo. Sin revisar el código de estado, tu programa **sigue adelante creyendo que todo salió bien**. Intenta convertir a JSON una página de error de HTML y truena con un `SyntaxError` sobre un `<` inesperado. O peor: el servidor devuelve un JSON de error con otra forma, la conversión funciona, y tu programa se cae treinta líneas más abajo al hacer `preguntas.length` sobre algo que no es un arreglo. **El error explota en un lugar completamente distinto de donde está la causa, y con un mensaje que no tiene ninguna relación con ella.** Eso es lo que convierte un bug de dos minutos en una tarde perdida: no que falle, sino que falle mintiendo sobre dónde.

La corrección son tres líneas y hay que escribirlas siempre:

```javascript
export async function obtenerPreguntas() {
  const r = await fetch(URL_API);
  if (!r.ok) {                                   // ok es true solo para 200–299
    throw new Error(`El servidor respondió ${r.status}`);   // lanzas TÚ el error
  }
  return r.json();
}
```

La idea de fondo es más grande que `fetch`: **cuando una función que llamas no considera error algo que para ti sí lo es, tienes que convertirlo en error tú mismo.** Ese `throw` es el que hace que el `catch` de quien llama se entere. Y ahora hay un solo lugar donde se decide qué cuenta como fallo.

Los códigos que vas a ver: el 200, que todo salió bien; el 404, que la dirección no corresponde a nada; el 429, que te vas a encontrar seguro porque significa que hiciste demasiadas peticiones muy rápido y aparece cuando recargas cuarenta veces seguidas probando; y el 500, que el problema es del servidor y no tuyo. Tu mensaje al usuario debería distinguir al menos entre "no hay conexión" y "el servicio está teniendo problemas".

### `Promise.all`: el error de rendimiento más común del curso

Esto que sigue es literalmente el error de rendimiento más común del curso, semestre tras semestre.

```javascript
// MAL: tres esperas encadenadas sin ninguna razón
const categorias = await fetch(URL_CATEGORIAS);   // 100 ms
const dificultad = await fetch(URL_DIFICULTAD);   // 100 ms — no necesita lo anterior
const preguntas  = await fetch(URL_PREGUNTAS);    // 100 ms — tampoco
// Total: 300 ms
```

```javascript
// BIEN: las tres salen al mismo tiempo
const [categorias, dificultad, preguntas] = await Promise.all([
  fetch(URL_CATEGORIAS),
  fetch(URL_DIFICULTAD),
  fetch(URL_PREGUNTAS)
]);
// Total: 100 ms — el tiempo de la más lenta, no la suma
```

**Tres `await fetch` en fila tardan 300 ms. `Promise.all` de los tres tarda 100 ms.** Es el mismo trabajo, la misma red, el mismo servidor, y un tercio del tiempo, y la diferencia es puramente una decisión de escritura.

La analogía es doméstica: **son tres lavadoras.** La versión mala es poner una carga, quedarse mirando media hora, sacarla, poner la segunda, esperar otra media hora. La versión buena es poner las tres al mismo tiempo y esperar una sola media hora. Nadie lava ropa de la primera forma. Y la razón por la que en código sí se hace es que `await` se **lee** como una espera y eso invita a ponerlos en fila sin pensar.

Ahora la regla completa, porque `Promise.all` no siempre es la respuesta: **usa `await` secuencial solo cuando la segunda petición NECESITA el resultado de la primera.** Si tienes que pedir la lista de categorías para saber qué categoría pedir después, no hay nada que paralelizar; la dependencia es real y la secuencia es correcta. Si las tres peticiones son independientes, ponerlas en fila es regalar tiempo. La pregunta que tienes que aprender a hacerte frente a cada `await`: **"¿lo que viene después necesita esto, o solo está esperando por costumbre?"**

Y el detalle del comportamiento ante fallos, que es el que decide cuál de las dos usar:

```javascript
// Promise.all: si UNA falla, se rechaza toda. No obtienes las que sí funcionaron.
// Sirve cuando necesitas las tres para poder mostrar algo: sin una, no hay juego.
const todo = await Promise.all([a(), b(), c()]);

// Promise.allSettled: nunca se rechaza. Devuelve el resultado de TODAS,
// cada una con su status ("fulfilled" o "rejected"). Sirve cuando
// puedes funcionar con resultados parciales.
const resultados = await Promise.allSettled([a(), b(), c()]);
// [{ status: "fulfilled", value: ... }, { status: "rejected", reason: ... }, ...]
```

La forma de elegir es una pregunta sobre el producto, no sobre el código: **¿puedo mostrar algo útil si me falta uno de los tres?** Si la respuesta es no, `all`, y con un solo `catch` te basta. Si es sí, `allSettled`, y pintas lo que llegó marcando lo que no. Un dato que sorprende: con `Promise.all`, las otras peticiones **no se cancelan** cuando una falla; siguen su curso, solo que ya nadie recoge su resultado.

### Las ideas que hay que llevarse

**`fetch` solo rechaza si la petición no salió; los códigos de error son respuestas exitosas.** Hay que revisar `r.ok` a mano y lanzar el error uno mismo. Sin eso, el `catch` es decorativo y el programa falla lejos de la causa.

**`await` en fila es secuencial y eso casi nunca es lo que quieres.** Un `await` solo se justifica antes de otro si el segundo depende del primero. Tres esperas independientes en fila son un bug de rendimiento con la misma seriedad que un bug de lógica.

**El manejo de errores no es una capa que se agrega al final.** Los tres estados —carga, éxito, error— son parte del diseño desde la primera línea. Un programa que solo contempla el camino feliz no está incompleto: está mal.

### Ponte a prueba

*Si el servidor devuelve un 500 y no revisas `r.ok`, ¿dónde va a explotar tu programa?*

*Tienes que pedir el perfil de un usuario y, con su id, sus partidas. ¿`Promise.all`?*

*Si una de tres peticiones falla y usas `Promise.all`, ¿qué le muestras al usuario?*

---

## Práctica en clase: el esqueleto de la trivia

El objetivo es concreto: **que no cierres el portátil esta noche sin una petición real funcionando, con los tres estados visibles.** No el juego completo —eso es la Misión 04 y son diez horas—, sino el esqueleto que ya conversa con el servidor. Lo demás es interfaz y lógica, y eso ya lo sabes hacer.

**Tarea 1: elegir la API y verificarla.** Abre la URL directamente en el navegador antes de escribir una línea de JavaScript, mira la forma del JSON que devuelve y **dibuja en papel** qué campos vas a usar. Diez minutos, no más. Es el paso que todo el mundo quiere saltarse y el que evita la mitad de los problemas: si no sabes qué forma tienen los datos, vas a estar depurando la forma de los datos en lugar de tu programa.

**Tarea 2: el módulo `api.js` con los cuatro elementos obligatorios.** Una función `async` que haga el `fetch`, revise `r.ok`, lance su propio error con el código de estado, y devuelva los datos ya normalizados a la forma que tu juego necesita. Normalizados quiere decir que el resto del programa no tenga que saber que la API llama `question` a lo que tú llamas `enunciado`. Ese módulo es la única parte del programa que sabe de dónde vienen los datos.

**Tarea 3: los tres estados en el HTML, de verdad.** Tres contenedores, uno por estado, y una función por estado que muestra uno y oculta los otros dos. Nada de dejar el error escondido en la consola. Prueba el estado de error de dos formas distintas: **desconectando el wifi** y **rompiendo la URL a propósito** para provocar un 404. Son dos caminos distintos hacia el mismo estado visible y conviene verificar los dos, porque el primero rechaza la promesa y el segundo no.

**Tarea 4: medir los 300 ms contra los 100 ms.** Este es obligatorio y se hace con la pestaña de red abierta. Escribe las dos versiones, la secuencial y la de `Promise.all`, con tres peticiones independientes, y **mide en la pestaña Red**. Lo que tienes que observar no es solo el número total: es la **forma del diagrama de barras**. En la versión secuencial las tres barras están escalonadas, cada una empieza donde termina la anterior. Con `Promise.all` las tres barras arrancan alineadas a la izquierda. Esa imagen se te queda mucho más que los números. Captúrala y pégala en el `README.md` de la Misión 04 con dos líneas explicando qué se ve.

Antes de cerrar, revisa exactamente cuatro cosas y no te distraigas con la estética, que hoy no importa: que exista el `if (!r.ok)`; que `mostrarCargando()` esté **antes** del `await` y no después; que el mensaje de error que se pinta esté escrito para un humano y no sea el `error.message` crudo; y que `api.js` no toque el DOM ni una vez. Si esas cuatro están, la Misión 04 ya tiene la estructura correcta y lo que falta es trabajo, no descubrimiento.

Dos cosas que te van a pasar. La primera, el 429: cuando varias personas recargan cuarenta veces la misma API en diez minutos, el servicio empieza a rechazar. No es tu código. Aprovecha para que ese caso también tenga su mensaje, y de paso ahí está la razón de la secundaria opcional de caché. La segunda, el `await` en el nivel superior del módulo: si escribes `const datos = await obtenerPreguntas()` fuera de toda función te va a funcionar, porque estás en un módulo. Está bien y es una de las cosas que ganaste la semana pasada, pero para poder reintentar necesitas que la carga esté dentro de una función a la que se pueda volver a llamar.

---

## Auditoría 2 cruzada (25 XP)

Esta noche también hay auditoría cruzada. Vas a revisar el `api.js` de un compañero, en pareja, con esta lista de cuatro preguntas: *¿revisa `r.ok` y lanza su propio error?* *¿hay algún `await` en fila que podría ser `Promise.all`?* *¿qué ve el usuario si no hay red: un mensaje o una pantalla en blanco?* *¿`api.js` toca el DOM en algún punto?*

La entrega es en prosa: **tres hallazgos escritos, cada uno con la línea donde está y qué cambiarías**, firmados con tu nombre, en el repositorio del auditado como un issue o un archivo `auditoria-2.md`. Los 25 XP son para quien audita y se otorgan por la **calidad de los hallazgos**, no por encontrar muchos: un hallazgo bien explicado vale más que cinco "está bien".

Y el tono importa: se audita el código, no a la persona, y quien recibe una auditoría dura recibió un favor.

---

## Tu misión de la semana: Trivia con API externa (100 XP)

Construyes un juego de preguntas que consume una **API pública real**.

Necesita: estados explícitos y visibles de carga, éxito y error; preguntas con opciones seleccionables; puntaje acumulado; y una pantalla de resumen al final con el resultado de la partida.

### Por qué el manejo de errores se califica con dureza

No es una parte opcional que se revisa si sobra tiempo. Un juego bonito que se queda en blanco sin red pierde más puntos que un juego feo que explica qué pasó, y la razón no es capricho: **el manejo de errores es exactamente lo que separa un ejercicio de clase de algo que le podrías mostrar a alguien.** Cualquiera hace funcionar el camino feliz.

Lo que se evalúa concretamente es qué ve el usuario cuando la API tarda cinco segundos, cuando devuelve un 500 o cuando manda datos con un campo faltante. Pruébalo desconectando el wifi. Es la única prueba que vale.

### Cómo se califica

| Criterio | XP |
|---|---|
| Funciona: el juego se puede jugar de principio a fin | 35 |
| Calidad del código: nombres claros, sin repetición ni código muerto | 25 |
| Específico del módulo (ver abajo) | 20 |
| Proceso en Git: commits pequeños y descriptivos, PR completo | 10 |
| Auditoría de IA: `IA.md` real y reflexivo | 10 |

Los 20 XP específicos de esta misión: los tres estados son visibles en la interfaz (6), visibles en pantalla y no en la consola; `async`/`await` con `try`/`catch` real (5), y "real" significa que el `catch` hace algo, no que exista; `Promise.all` donde corresponde (5), lo que también implica no usarlo donde hay dependencia verdadera; y módulos con responsabilidades claras (4), con la regla de la sesión pasada: si un módulo importa `document`, no contiene reglas del juego.

**Misión secundaria opcional (+25 XP):** caché con `sessionStorage` para no repetir peticiones idénticas. Ya sentiste en carne propia por qué, con el 429: antes de pedir, revisa si ya tienes esa respuesta guardada; si la tienes, úsala; si no, pide y guarda. Es la primera vez en el curso que vas a escribir algo que hace una aplicación mediblemente mejor sin agregar ninguna funcionalidad visible.

Sobre la IA, esta misión te sirve el ejemplo perfecto en bandeja. Pídele a un asistente "una función que traiga preguntas de una API con fetch" y lee lo que devuelva. La apuesta es segura: **la mayoría de las veces el código no revisa `r.ok`.** Va a tener un `try`/`catch` que se ve muy responsable, va a tener `async`/`await` correcto, y va a estar ciego a los 404 y los 500 exactamente como el código que escribías hace dos horas. Ahí está el argumento entero del curso: el código generado es sintácticamente impecable y semánticamente incompleto, y **la única forma de notarlo es saber qué debería estar ahí.** La regla completa: todo código generado se declara en `IA.md`, con qué pediste, qué te devolvió, qué estaba mal y qué corregiste. **Sin declarar, la misión se califica en cero y no admite reintento. Declararlo NO baja la nota**, porque lo que se evalúa es tu capacidad de auditar. Este caso del `r.ok` faltante es candidato ideal para la insignia 🔍 **Cazador de alucinaciones**, con su día extra de plazo acumulable.

**Plazo:** antes de la medianoche del día anterior a la sesión 11. Presupuesto: **10 horas** repartidas en 2 de planear, 5 de programar, 2 de manejo de errores y 1 de documentar. Fíjate en que el manejo de errores tiene su propio bloque de horas: eso es deliberado.

---

## Errores que probablemente vas a cometer

**No revisar `r.ok` y creer que el `try`/`catch` te protege.** Es el error central del tema y el más caro, porque produce la peor combinación posible: código que parece defensivo y no lo es. Tienes un `try`, tienes un `catch`, tienes `async`/`await` bien escrito, y tu programa se cae igual ante un 404 porque el `catch` nunca se enteró de nada. Lo que agrava el problema es que el error termina saliendo por otro lado —un `SyntaxError` de JSON, o un `undefined` treinta líneas después— así que vas a buscar la causa en el lugar equivocado durante mucho tiempo. Escribe el `if (!r.ok) throw` desde esta noche, sin excepción, hasta que salga sin pensarlo.

**Encadenar `await` innecesariamente y triplicar el tiempo de carga.** Vas a escribir tres, cuatro o cinco `await` en fila porque el código se lee bien así, sin preguntarte ni una vez si alguno depende del anterior. El resultado es una pantalla de carga tres veces más larga de lo necesario, y como funciona, nadie lo reporta como bug. Es invisible sin medir: nadie nota la diferencia entre 300 ms y 100 ms mirando la pantalla, pero la pestaña de red la muestra sin ambigüedad. Ante cada `await`, pregúntate si lo que viene después necesita ese resultado o solo está esperando por inercia.

**Olvidar el `await` del `.json()`, o llamar una función `async` sin `await`.** Los dos son la misma confusión de fondo: no distinguir la promesa del valor que contiene. El síntoma es inconfundible una vez que lo conoces —`Promise { <pending> }` en la consola, o un `undefined` al leer cualquier propiedad—. El patrón que tienes que aprender a reconocer: si en la consola aparece la palabra `Promise` donde esperabas datos, falta un `await` en alguna parte, y casi siempre es el inmediatamente anterior.

**Tratar el estado de error como algo que se agrega al final, si sobra tiempo.** Vas a construir el juego completo con la API funcionando, la interfaz terminada, todo bonito, y a dejar el manejo de errores como el último punto de la lista, que es el que nunca se hace. La consecuencia es que la primera vez que tu proyecto se vea desde otra red o el servicio se caiga, la aplicación se queda en blanco y parece que no funcionara. Lo que hay que corregir no es la técnica sino el orden de trabajo: **el estado de error se escribe primero, cuando todavía es fácil, y se prueba a propósito desconectando la red.** Un programa que solo contempla el camino feliz no está a medio hacer; está mal hecho, y en esta misión se califica como tal.

---

## Fuentes de esta sesión

- MDN Web Docs. *Using the Fetch API*. https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch
- MDN Web Docs. *Using Promises*. https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Using_promises

Las dos son lectura obligatoria y cada una sirve para algo distinto. *Using Promises* es la explicación conceptual: los estados, el encadenamiento, cómo se propagan los errores y por qué las promesas resolvieron un problema real de los callbacks. Es la que hay que leer entera y despacio, y la que aclara la mayoría de las dudas de la noche. *Using the Fetch API* es la referencia práctica, y tiene un párrafo que vale la sesión completa: el que dice explícitamente que la promesa de `fetch` **no** se rechaza ante respuestas de error HTTP y que hay que examinar `Response.ok`. Búscalo y léelo, porque cuando alguien discuta el tema en el canal del curso la respuesta es esa línea. La documentación ya avisó, y casi nadie la lee: eso, en sí mismo, es parte de lo que se enseña aquí.

---

## Antes de la sesión 11

Lee la sección "Módulo 3, sesión 11" de `GUIA-DEL-CURSO.md`, y de MDN la parte de *Using the Fetch API* que no alcanzaste a leer esta noche, en particular lo relativo a cancelar peticiones. Quince minutos.

Y una tarea de reflexión que no es de lectura: **apunta en el `README.md` de tu Misión 04 los tres errores que tu trivia puede sufrir y qué muestra en cada caso.** Sin red, el servicio caído o devolviendo 500, y datos que llegan con una forma distinta a la esperada. Tres líneas, en prosa, escritas antes de programar el manejo de esos errores y no después. Escribirlo primero es lo que hace que se implemente, y de paso es media rúbrica de la misión resuelta en cinco minutos de pensar.
