# Sesión 9 · Módulos de ES y el bucle de eventos

**Módulo 3** · Zona 3: La Sala de Máquinas
**Lo que sale de esta noche:** tu juego partido en módulos que se importan entre sí, y la predicción del orden `1 4 3 2` hecha en papel, fallada y entendida
**Tu misión al terminar:** esta noche no se asigna misión nueva. La Misión 03 sigue en curso, y esta noche hay **Quiz 3** (15 XP)

---

## Por qué esta es la sesión más importante del curso

No es una exageración para que prestes atención. Es una descripción de la estructura del semestre.

Todo lo que viene después —las promesas de la sesión 10, el `fetch` de la Misión 04, los efectos de React en el módulo 4, cualquier cosa que hagas con un servidor— es una consecuencia directa de cómo el navegador decide qué código correr y cuándo. Si el event loop te queda claro esta noche, las cinco sesiones siguientes se sienten como aplicaciones de una sola idea. Si no queda claro, cada una se siente como magia nueva, y ahí es donde se pierde la gente.

Así que la instrucción más importante para ti al estudiar esta sesión es: **ve despacio.** Es mejor salir con la pila de llamadas, las dos colas y el orden `1 4 3 2` entendidos de verdad que con diez conceptos a medias.

La otra cosa que se resuelve esta noche es más silenciosa: dejar de escribir un archivo `main.js` de cuatrocientas líneas. Los módulos no son un tema de sintaxis, son el tema de cómo se organiza un programa que ya no cabe en la cabeza de una sola persona a la vez.

Y una frase para dejar flotando hasta después de los módulos, porque es el anzuelo de la noche: **el navegador no llama a tus manejadores de evento cuando ocurre el clic; los llama cuando puede.**

---

## Módulos de ES

Antes de leer lo que sigue, abre tu `main.js` de la Misión 03 y mira cuántas líneas tiene. Si pasa de doscientas, hazte esta pregunta concreta: ¿dónde está la función que compara dos cartas? Cuenta cuánto scroll te toma encontrarla. Ese scroll es el argumento entero de este bloque.

Ahora un escenario de dos archivos, sin módulos, cargados con dos etiquetas `<script>`:

```javascript
// tablero.js
let cartas = [];
function reiniciar() { cartas = []; }

// puntaje.js
let cartas = 0;   // otra persona del equipo usó el mismo nombre para otra cosa
```

¿Qué pasa cuando el navegador carga los dos archivos en ese orden? Hay **una sola** `cartas` en el ámbito global, el segundo archivo pisó la del primero, y el error va a aparecer en `reiniciar()`, en un archivo que nadie tocó. Ese es el problema que los módulos resuelven, y conviene verlo como problema antes de ver la solución.

### Un archivo, un ámbito, un contrato

Un módulo de ES es un archivo de JavaScript con dos propiedades que lo cambian todo. La primera es que **tiene su propio ámbito**: lo que declaras dentro de un módulo no existe fuera de él, punto. No hay colisión posible entre dos módulos que usen el mismo nombre, porque esos dos nombres viven en habitaciones distintas. La segunda es que **lo que sale del módulo lo decides tú, explícitamente, con `export`**, y lo que entra lo pides explícitamente con `import`.

La analogía es **la cocina de un restaurante**. Dentro de la cocina hay cuchillos, ollas, medio kilo de cebolla picada, un cocinero de mal humor y un desastre organizado que a nadie de la sala le interesa. Lo único que sale por la ventanilla son platos terminados, y la ventanilla es angosta a propósito. Un módulo es eso: un desorden privado del que solo salen platos. `export` es la ventanilla. Y la parte importante de la analogía es la que suele pasarse por alto: **el mesero no puede entrar a revolver las ollas.** Si el módulo de puntaje no exporta su variable interna, nadie de afuera la puede modificar, y por lo tanto cuando el puntaje esté mal solo hay un archivo donde buscar.

Las exportaciones **nombradas** son las que se usan el noventa por ciento del tiempo:

```javascript
// mazo.js
export const PAREJAS_POR_DEFECTO = 8;

export function crearMazo(parejas = PAREJAS_POR_DEFECTO) {
  // ...
}

export function barajar(lista) {
  // ...
}
```

```javascript
// main.js
import { crearMazo, barajar, PAREJAS_POR_DEFECTO } from "./mazo.js";
```

Tres detalles que te van a morder. Las llaves del `import` **no son un objeto**, son una lista de nombres, aunque se parezcan; los nombres tienen que coincidir exactamente con lo exportado, porque no es una asignación, es una referencia por nombre. La ruta tiene que empezar con `./` o `../` y **tiene que llevar la extensión `.js`** en el navegador, a diferencia de lo que vas a ver en tutoriales de Node o de empaquetadores. Y si el nombre choca con algo que ya tienes, se renombra en el punto de importación con `as`:

```javascript
import { barajar as barajarCartas } from "./mazo.js";
```

Después está la exportación **por defecto**:

```javascript
// juego.js
export default class Juego { /* ... */ }

// main.js
import Juego from "./juego.js";       // sin llaves, y el nombre lo elige quien importa
```

Un módulo puede tener cuantas exportaciones nombradas quiera y **como máximo una por defecto**. La diferencia práctica es quién pone el nombre: con la nombrada lo pone quien exporta, con la default lo pone quien importa. Eso último suena cómodo y es justamente el problema: el mismo módulo termina importado como `Juego` en un archivo, `juego` en otro y `GameEngine` en un tercero, y buscar en el proyecto quién usa esa clase se vuelve imposible. La regla del curso, que se califica en la revisión de código: **exportaciones nombradas por defecto —perdón por el retruécano—, y `export default` solo cuando el módulo entero es una sola cosa**, como una clase o un componente. En el módulo 4 con React vas a ver la convención opuesta y está bien; ahí el archivo *es* el componente.

Y el detalle que hace que nada funcione si lo olvidas: en el navegador, un módulo se carga con el atributo `type="module"`.

```html
<script type="module" src="./main.js"></script>
```

Ese atributo cambia más de lo que parece. Un script con `type="module"` se carga en modo diferido por defecto —no bloquea el análisis del HTML y se ejecuta cuando el documento está listo, así que `document.querySelector` ya encuentra los elementos—, corre siempre en modo estricto sin que lo pidas, y su ámbito superior no es el global. Además, y esto es lo que te va a frenar: **los módulos no funcionan abriendo el archivo con doble clic**: el protocolo `file://` los bloquea por seguridad. Necesitas un servidor local, aunque sea la extensión Live Server o `python3 -m http.server`.

### Las ideas que hay que llevarse

**Cada módulo tiene su propio ámbito, y eso mata la clase de error más difícil de encontrar.** Antes de los módulos, cualquier archivo podía pisar una variable de cualquier otro, y el error aparecía lejos de la causa. Con módulos, si una variable está mal, el sospechoso está en el mismo archivo o entró por un `import` que puedes leer en la primera línea.

**`export` es un contrato, no un trámite.** Lo que exportas es lo que prometes mantener; lo que no exportas puedes reescribir mañana sin romperle nada a nadie. Pensar dos veces antes de exportar algo es diseño, no burocracia.

**Un módulo se importa una sola vez, aunque lo importes en diez archivos.** El navegador lo evalúa la primera vez y guarda el resultado; los otros nueve `import` reciben lo mismo. Eso significa que si un módulo tiene estado —una variable que cambia—, ese estado es compartido por todo el programa. Es útil y es peligroso.

### Ponte a prueba

*Si un módulo declara `let intentos = 0` y no lo exporta, ¿puede otro módulo cambiarlo?*

*¿Dónde debería vivir la función que dibuja las cartas en el DOM: en el módulo del mazo o en otro?*

*Si dos módulos se importan mutuamente, ¿qué crees que pasa?* Existen las dependencias circulares, el sistema de módulos las tolera a medias, y si te aparece un `undefined` inexplicable al importar, eso es lo primero que hay que revisar.

---

## Práctica en clase: partir el juego en piezas

Este recorrido se hace sobre la Misión 03 que ya tienes escrita, que es lo que lo hace valioso: no es un ejercicio inventado, es refactorizar tu propio código.

**Primero, levanta el servidor y falla a propósito.** Antes de tocar el JavaScript, agrega `type="module"` al `<script>` y abre el `index.html` con doble clic. La consola va a mostrar el error de CORS por `file://`. Léelo completo, en voz alta si estás solo. Después levanta Live Server y funciona. Esto no es una pérdida de tiempo: es la vacuna contra la media hora que ibas a perder esta semana, y te entrena para leer un mensaje de error como información y no como castigo.

**Segundo, decide los cortes antes de mover un archivo.** Dibuja cuatro cajas en un papel y decide qué va en cada una. La división que se exige en la Misión 04 es esta: `mazo.js` sabe de cartas y de azar, y no sabe que existe un navegador. `estado.js` sabe en qué situación está la partida y devuelve estados nuevos. `vista.js` sabe pintar un estado en el DOM y no decide nada. `main.js` no sabe casi nada: conecta los tres anteriores y escucha los eventos. La frase que resume la regla: **si un módulo importa `document`, no puede contener reglas del juego; y si contiene reglas del juego, no puede importar `document`.**

**Tercero, extrae `mazo.js`.** Corta `crearMazo` y `barajar` del archivo grande, pégalas en el nuevo archivo, ponles `export` y agrega el `import` en `main.js`. Recarga. Funciona.

**Cuarto, comete el error del `export` olvidado.** Extrae ahora `estado.js` con la función `voltear`, pero **no le pongas `export`**. Escribe el `import` en `main.js` y recarga. El error que aparece es este:

```
Uncaught SyntaxError: The requested module './estado.js'
does not provide an export named 'voltear'
```

Antes de arreglarlo, lee lo que el mensaje realmente dice: el módulo se encontró y se leyó bien —si la ruta estuviera mal, el error sería otro, un 404—, lo que no existe es la exportación. Aprender a distinguir esos dos errores te ahorra tardes enteras. Agrega el `export` y funciona.

**Quinto, comete el error del ámbito de módulo.** Este es el más instructivo de la noche. Conecta el manejador de clic con un atributo en el HTML, como se hacía antes:

```html
<!-- MAL, y a propósito -->
<div class="carta" onclick="voltearCarta(3)">…</div>
```

```javascript
// main.js, cargado con type="module"
function voltearCarta(id) { /* ... */ }
```

Haz clic y observa: `Uncaught ReferenceError: voltearCarta is not defined`. La causa es la confirmación práctica de todo lo anterior: el atributo `onclick` del HTML busca la función en el ámbito **global**, y tu función no está ahí, está dentro del módulo, que tiene su propio ámbito. No es que no exista: es que el HTML no tiene permiso para verla. Ninguna cantidad de recargas lo va a arreglar. La solución es la correcta desde el principio, `addEventListener` desde el módulo, y de paso se muere una mala costumbre del semestre pasado sin que nadie tuviera que prohibirla.

**Sexto, extrae `vista.js` y mira la dirección de las flechas.** Cuando termines, dibuja las dependencias: `main` importa a los tres; `vista` importa el estado solo para leerlo; `mazo` y `estado` no importan a nadie. Todas las flechas apuntan hacia adentro, hacia lo que no sabe nada del navegador. Ese dibujo, y no la cantidad de archivos, es lo que hace que un proyecto sea mantenible.

**Séptimo, el módulo con estado compartido.** Compruébalo en treinta segundos: un módulo se evalúa una sola vez.

```javascript
// contador.js
console.log("módulo contador evaluado");   // aparece UNA vez, no dos
export let clics = 0;
export function sumar() { clics++; }
```

Impórtalo desde dos archivos distintos y mira que el `console.log` aparece una sola vez. Ahí está la idea del módulo como singleton, que en el módulo 4 se va a llamar "estado global" y te va a parecer familiar.

### Cuatro dudas que te van a salir

Te vas a preguntar por qué en los tutoriales de Node los `import` no llevan `.js` ni `./`. Porque ahí hay un resolutor de módulos y un empaquetador haciendo trabajo extra por ti. En el navegador puro, la ruta es una ruta de verdad, la que el servidor tiene que poder resolver a un archivo. Acostúmbrate a la versión explícita; entender la versión con azúcar después es trivial, al revés no.

Vas a querer importar dentro de un `if` para "cargar solo lo que necesitas". El `import` estático va siempre arriba y se resuelve antes de que corra una línea de tu código —por eso el navegador puede saber qué archivos pedir sin ejecutar nada—. La versión dinámica existe, se escribe `import("./algo.js")` y **devuelve una promesa**. Guarda esa palabra: la semana que viene es el tema entero.

Vas a tener un error de mayúsculas en el nombre de un archivo. En Windows funciona y al subirlo a GitHub Pages, que corre sobre Linux, deja de funcionar. Es el bug más frustrante de todo el módulo porque "en mi máquina sí funciona". Revisa que el `import` diga exactamente lo que dice el archivo, letra por letra.

Y puedes terminar con nueve módulos de doce líneas. Partir no es un fin en sí mismo. Un módulo se justifica cuando tiene una responsabilidad que puedes nombrar en una frase corta sin usar la palabra "y".

---

## La pila, las colas y el event loop

Antes de leer una línea más, respondete esto: **JavaScript ejecuta una sola cosa a la vez, en un solo hilo. Entonces, ¿cómo puede una página tener una animación corriendo, un temporizador contando y responder a tus clics, todo a la vez, si solo puede hacer una cosa?**

Las dos respuestas que se dan siempre son "hilos" y "en paralelo", y las dos son incorrectas. La respuesta correcta es que **no lo hace todo a la vez; lo hace todo muy rápido, por turnos, y quien reparte los turnos es el event loop.**

Este bloque conviene estudiarlo con un lápiz en la mano, dibujando las cajas mientras lees. El punto entero es ver el movimiento: cosas que entran, cosas que salen, cajas que se vacían. Un diagrama terminado no muestra movimiento.

### El dibujo, en cuatro piezas

**La pila de llamadas (call stack).** Una columna vertical, abierta por arriba. Aquí está el único lugar donde se ejecuta tu código. Cuando llamas a una función, se apila un marco encima; cuando la función retorna, ese marco se quita. Es LIFO: **lo último que entró es lo primero que sale.** La analogía es la pila de platos del lavaplatos: solo puedes tocar el de arriba, y para llegar al de abajo tienes que sacar todos los de encima. Dibuja una llamada anidada de tres niveles y bórrala en orden inverso, marco por marco, para ver el mecanismo.

Y aquí la frase que sostiene todo: **mientras haya algo en la pila, nada más puede correr. Nada. Ni un clic, ni una animación, ni un temporizador que ya venció.**

**Las Web APIs.** Una caja al lado, y márcala como "esto NO es JavaScript". Aquí viven `setTimeout`, el sistema de eventos del DOM, `fetch`, `requestAnimationFrame`. Son capacidades que el navegador —escrito en C++— le presta al motor de JavaScript. Cuando tu código llama a `setTimeout(fn, 1000)`, la función `setTimeout` se apila, le entrega el encargo al navegador, y **retorna inmediatamente**. La pila queda libre. El navegador cuenta el segundo por su cuenta, en su propio hilo, sin que tu programa espere.

La analogía: **eres el único cocinero de una cocina, pero tienes temporizadores de horno.** No te quedas mirando el horno veinte minutos; pones el temporizador y sigues picando. El temporizador cuenta solo. Eso no te convierte en dos cocineros: sigues siendo uno, y por eso, cuando el temporizador suene, no vas a poder atenderlo hasta que sueltes el cuchillo.

**La cola de macrotareas (task queue).** Una fila horizontal, con entrada por un lado y salida por el otro. Cuando el navegador termina un encargo —venció el temporizador, ocurrió el clic—, **no ejecuta tu callback**: lo pone a hacer fila aquí. Es FIFO, el primero que llegó es el primero que sale. Y aquí va la corrección del malentendido número uno: `setTimeout(fn, 0)` **no significa "ejecuta ya"**. Significa "ponlo en la fila ya". Cuánto tarde en correr depende de qué haya delante y de cuándo se libere la pila. El número es un mínimo, no una promesa.

**La cola de microtareas (microtask queue).** Otra fila, dibujada **arriba** de la anterior, más corta pero con una flecha más gruesa hacia la pila. Aquí van los callbacks de las promesas: todo lo que registres con `.then`, `.catch`, `.finally`, y todo lo que siga a un `await`. Y aquí va la regla que es el corazón del módulo:

> **Las microtareas se vacían POR COMPLETO antes de tomar la siguiente macrotarea.**

Por completo significa por completo: si al procesar una microtarea se generan tres microtareas nuevas, esas tres también se procesan en el mismo turno, antes de mirar la cola de abajo. La cola de microtareas no se atiende "una por vuelta"; se drena hasta el fondo.

**Y el event loop, que es lo más simple de todo.** Una flecha circular alrededor del conjunto. El event loop es un vigilante con una rutina de tres pasos que repite para siempre: *¿la pila está vacía? Si no, espero.* *¿Hay microtareas? Las paso todas a la pila, hasta que no quede ninguna.* *¿Hay macrotareas? Tomo UNA, la paso a la pila, y vuelvo al paso uno.*

Eso es todo. No hay más. A estas alturas esperabas que fuera complicado, y la sorpresa de que sea tan poco es lo que hace que se te quede: **el event loop no es un algoritmo inteligente, es un mesero con una regla de prioridad.**

La analogía completa que lo amarra es **la sala de urgencias.** Hay un solo médico (la pila). Hay una fila de triage rojo (microtareas) y una fila de triage verde (macrotareas). La enfermera de la puerta —el event loop— tiene una sola regla: no interrumpe nunca al médico con un paciente adentro, y cuando el médico se libera, **primero pasa a todos los rojos que haya, y solo cuando no queda ni un rojo pasa un verde, uno solo, y vuelve a revisar los rojos.** De ahí se sigue todo, incluyendo la parte incómoda: si llegan rojos sin parar, los verdes no pasan nunca.

### El reto: predícelo en papel antes de ejecutarlo

Este es el ejercicio central de la sesión y lo tenías de tarea. **Escríbelo en papel, con lápiz, antes de tocar el `Enter`.** Quien ejecuta antes de predecir no aprende nada, solo lee un resultado.

```javascript
console.log("1");
setTimeout(() => console.log("2"), 0);
Promise.resolve().then(() => console.log("3"));
console.log("4");
```

¿Ya lo escribiste? Bien. Imprime **1 4 3 2**.

```javascript
console.log("1");                                  // síncrono: corre ya
setTimeout(() => console.log("2"), 0);             // macrotarea: a la fila de abajo
Promise.resolve().then(() => console.log("3"));    // microtarea: a la fila de arriba
console.log("4");                                  // síncrono: corre ya
// Salida: 1 4 3 2
```

Ahora lo importante, que es qué revela tu predicción. Hay tres respuestas típicas y cada una dice algo distinto de ti.

Si predijiste **1-2-3-4**, leíste el archivo de arriba abajo como si fuera una receta, y todavía no has separado "escribir una instrucción" de "ejecutar una instrucción".

Si predijiste **1-4-2-3**, ya entendiste que lo asíncrono va al final, que es medio camino, pero **crees que hay una sola cola**. Esa es la parte importante: ese error es exactamente el diagnóstico de que no distingues las dos colas, y es la razón por la que este ejercicio existe. Saber qué es lo que no sabes es media clase.

Recorre la ejecución moviendo cosas entre las cajas que dibujaste. `console.log("1")` se apila, imprime, se desapila. `setTimeout` se apila, entrega el encargo, se desapila; el navegador ve un retardo de cero, así que pone el callback en la cola de macrotareas casi de inmediato. `Promise.resolve()` ya está resuelta, así que el `.then` va derecho a la cola de microtareas. `console.log("4")` se apila, imprime, se desapila. **Ahora la pila está vacía y entra el event loop:** revisa microtareas, encuentra el `3`, lo ejecuta. Vuelve a revisar microtareas, no hay más. Ahora sí baja a las macrotareas y toma el `2`.

Un dato práctico que te va a interesar: **esta es la pregunta más frecuente en entrevistas técnicas de frontend.** No es folclore; es un clásico porque en una sola línea revela si alguien entiende el modelo de ejecución o si aprendió recetas. Si entiendes esta noche por qué es 1-4-3-2, tienes una respuesta que la mayoría de candidatos no tiene.

### Las ideas que hay que llevarse

**Un solo hilo no significa una sola cosa a la vez en el sistema; significa una sola cosa a la vez en tu código.** El navegador tiene varios hilos y trabaja en paralelo con gusto. Lo que no puede es correr dos de tus funciones simultáneamente. Esa distinción es la que hace que la palabra "asíncrono" deje de ser mística: asíncrono no es paralelo, es *después*.

**El callback nunca se ejecuta cuando el evento ocurre, sino cuando la pila se libera.** Todo el asincronismo del navegador es esa frase. Un clic que ocurre mientras tu código corre no se pierde, pero tampoco se atiende: espera en la fila.

**Las microtareas siempre le ganan a las macrotareas, sin importar el orden en que se registraron ni el tiempo que se pidió.** Una promesa ya resuelta se atiende antes que un `setTimeout(fn, 0)` escrito tres líneas más arriba. Esto no es un detalle de implementación de un navegador: está en la especificación y es igual en todos.

### Ponte a prueba

*Si `setTimeout(fn, 1000)` promete mil milisegundos y tu pila está ocupada dos segundos, ¿cuándo corre `fn`?* A los dos segundos y algo, y el número mil no fue una mentira: fue un mínimo.

*¿Puede el event loop interrumpir una función a mitad de camino para atender un clic urgente?* No, jamás. Si tu función tarda tres segundos, el usuario tiene tres segundos de página muerta.

*Si dentro de un `.then` devuelves otra promesa que se resuelve al instante, ¿corre antes o después de un `setTimeout(fn, 0)` que ya estaba en fila?* Antes, porque la cola de microtareas se drena completa. Es la pregunta que separa a quien entendió la regla de quien la memorizó.

---

## Práctica en clase: el laboratorio del orden de ejecución

Esta práctica es distinta a todas las del semestre: **no se construye nada. Se predice y se comprueba.** Trabaja en un archivo `laboratorio.js` cargado con `type="module"`, y respeta la regla de oro: **primero la predicción en papel, con lápiz, y solo después el `Enter`.**

**Experimento 1: el del 1-4-3-2.** Ya lo hiciste arriba. Si tu predicción falló, vuelve a leer la sección de las cuatro piezas antes de seguir; los tres experimentos que vienen dependen de eso.

**Experimento 2: la pila bloqueada, o por qué animar con `while` es imposible.** Escribe un botón y esto:

```javascript
function calculoPesado() {
  const fin = Date.now() + 5000;
  let vueltas = 0;
  while (Date.now() < fin) { vueltas++; }   // cinco segundos de pila ocupada
  console.log("terminé, vueltas:", vueltas);
}

document.querySelector("#pesado").addEventListener("click", calculoPesado);
document.querySelector("#otro").addEventListener("click", () => console.log("clic en otro"));
```

Haz clic en `#pesado` y, mientras corre, haz clic seis o siete veces en `#otro`, intenta seleccionar texto de la página y mueve el cursor sobre los botones. **Nada responde.** La página está congelada: el texto no se selecciona, los estilos de `:hover` no cambian, el botón no se hunde. Y cuando termina, aparecen de golpe todos los `clic en otro`. Cuenta cuántos aparecen y compáralo con cuántos hiciste; casi siempre son menos, porque el navegador descarta clics repetidos en el mismo lugar.

Esto es la demostración física de la frase de la pila. Y de aquí sale la conexión con la sesión 8: **por eso es imposible animar con un bucle `while`.** Si esta semana intentaste algo como `while (x < 500) { x++; carta.style.left = x + "px"; }` esperando ver la carta desplazarse, no viste nada moverse: viste la carta en la posición inicial, la página congelada, y de repente la carta en 500. La razón ahora es evidente. El navegador **no puede pintar mientras la pila está ocupada**, porque pintar también necesita su turno. Tus quinientas asignaciones de estilo ocurrieron todas antes de que el navegador tuviera oportunidad de dibujar una sola vez. Animar no es "cambiar la posición muchas veces rápido": es **cambiarla una vez y devolverle el turno al navegador**, y eso es precisamente lo que hace `requestAnimationFrame`.

**Experimento 3: el orden con varias fuentes.** Predice en papel y comprueba:

```javascript
console.log("A");
setTimeout(() => console.log("B"), 0);
setTimeout(() => console.log("C"), 100);
Promise.resolve().then(() => console.log("D")).then(() => console.log("E"));
queueMicrotask(() => console.log("F"));
console.log("G");
// Predice antes de ejecutar. Salida: A G D F E B C
```

La sorpresa buena aquí es que `E` sale después de `F` aunque `D` y `E` están en la misma cadena: cada `.then` de la cadena es una microtarea **nueva**, que se encola cuando la anterior termina, así que `F`, que ya estaba en fila, va antes. Si entiendes esto, entendiste que "drenar la cola" incluye lo que va llegando durante el drenaje.

**Experimento 4: el que da miedo, opcional si vas rápido.** Una microtarea que se reencola a sí misma:

```javascript
// OJO: esto congela la pestaña. Ten lista la forma de cerrarla.
setTimeout(() => console.log("¿llego algún día?"), 0);
function siempre() { Promise.resolve().then(siempre); }
siempre();
```

El `setTimeout` **nunca corre**. La cola de microtareas nunca se vacía, así que el event loop nunca baja a las macrotareas. Es la parte incómoda de la analogía de urgencias: si los rojos no dejan de llegar, los verdes no pasan jamás. Míralo una vez y cierra la pestaña. Sirve para que la palabra "por completo" deje de sonar a detalle.

**Cierre: documéntalo.** Agrega al `README.md` de tu Misión 03 una sección corta —cinco o seis líneas, en prosa— explicando con tus palabras por qué el primer experimento imprime 1 4 3 2. Escribirlo es lo que convierte "vi el resultado" en "lo entiendo". Y esa explicación escrita es, casualmente, un ensayo de la respuesta de entrevista.

La pregunta que tienes que hacerte al final de cada experimento no es "¿me funcionó?" sino **"¿por qué?"**. Y si tu explicación es "porque las promesas son más rápidas", corrígela ahora mismo: no son más rápidas, tienen **prioridad**. La velocidad no tiene nada que ver.

---

## Sobre la IA en esta sesión

Esta noche no hay misión nueva, pero hay una auditoría que vale la pena hacer. Pídele a un asistente "¿qué imprime este código?" con el ejemplo del 1-4-3-2. Casi siempre acierta, porque es un caso famosísimo. Ahora pídele algo menos famoso: el experimento 3, con la cadena de `.then` y el `queueMicrotask`. Ahí la tasa de error sube notablemente, y el resultado se entrega con la misma seguridad absoluta.

Ese contraste es la lección entera: **la confianza del texto generado no tiene relación con su corrección, y la única defensa es poder verificarlo tú.**

La regla completa, otra vez: todo código generado se declara en `IA.md`, con qué pediste, qué te devolvió, qué estaba mal y qué corregiste. **Sin declarar, la misión se califica en cero y no admite reintento; declararlo NO baja la nota.** Lo que se evalúa es tu capacidad de auditar, y hoy tuviste una clase entera de auditar predicciones.

Sobre entregas: la Misión 03 sigue en curso y se entrega según el cronograma. La refactorización en módulos que hiciste hoy cuenta para el punto de la rúbrica de organización del código, así que incorpórala ahora y no la dejes como "mejora pendiente". El reparto es el de siempre: funciona 35, calidad de código 25, específicos del módulo 20, proceso Git 10, auditoría IA 10. Y la refactorización de esta noche debería ser varios commits pequeños con mensajes que digan qué se movió, no un commit gigante llamado "cambios".

La semana que viene se asigna la **Misión 04**, el juego de trivia que consume una API real y es la más grande del módulo: diez horas de trabajo autónomo. Ve pensando en un tema para tu trivia.

Y una frase honesta antes de cerrar: esta noche fue la cuesta más empinada del semestre. Lo que viene es más trabajo, pero no es más difícil. La semana que viene no vas a aprender un tema nuevo: vas a aprender una sintaxis cómoda —`async`/`await`— para trabajar con la cola de arriba, la de las microtareas. La sesión 10 va a tener sentido en la medida en que esta noche haya tenido sentido.

---

## Errores que probablemente vas a cometer

**Creer que `setTimeout(fn, 0)` ejecuta la función inmediatamente.** Es el malentendido más extendido y el más caro, porque produce código que funciona por accidente. Vas a leer el cero como "ahora" y no como "encólalo con retardo mínimo", y de ahí sale el patrón de usar `setTimeout(fn, 0)` para "esperar a que el DOM esté listo" o para "arreglar" un problema de orden. A veces funciona, porque encolar una macrotarea sí mueve el código después de todo lo síncrono, y esa coincidencia es la que fija el malentendido. El antídoto es el experimento 2: cuando ves que un `setTimeout` de cero espera cinco segundos porque la pila estaba ocupada, el cero deja de significar "ya".

**No distinguir la cola de microtareas de la de macrotareas.** Se diagnostica solo: es predecir 1-4-2-3. Ya entendiste que lo asíncrono se posterga, pero manejas un modelo con una única fila donde el orden lo decide quién llegó primero. Con ese modelo, el `setTimeout` escrito antes del `.then` debería salir antes, y no sale. Usa la palabra "prioridad" en lugar de "velocidad", y vuelve a dibujar las dos filas cada vez que la confusión reaparezca. No basta con leerlo una vez: tienes que predecir tres o cuatro casos distintos hasta que la regla se te vuelva refleja.

**Poner un cálculo pesado en el hilo principal y culpar al navegador.** Se te va a aparecer como "mi juego se traba", "el navegador es lento" o "necesito una computadora mejor". Escribes un bucle que recorre diez mil elementos dentro de un manejador de clic, o generas un mazo enorme de forma síncrona, y concluyes que el problema es del entorno. La reacción correcta es abrir la pestaña de rendimiento y buscar la tarea larga en lugar de adivinar: el navegador te muestra exactamente qué función tuvo la pila ocupada y cuánto. Y la regla operativa, que se califica: **si una función va a tardar más de unos pocos milisegundos, hay que partirla o sacarla del hilo principal**, y ninguna cantidad de `setTimeout` sueltos arregla un cálculo que simplemente es demasiado grande.

**Olvidar `export`, o el `type="module"`, o la extensión `.js` en el `import`.** Son tres errores distintos con la misma consecuencia práctica —nada funciona— y hay que aprender a distinguirlos por el mensaje, porque cada uno dice algo diferente. Sin `type="module"`, el `import` produce un error de sintaxis inesperada, porque el navegador está leyendo el archivo como script clásico y `import` no es válido ahí. Sin `export`, el mensaje dice literalmente que el módulo no provee esa exportación, lo que significa que el archivo sí se encontró. Sin la extensión o con la ruta mal, aparece un 404 en la pestaña de red, que es la pista de que el archivo no existe donde lo estás pidiendo. Lee el mensaje completo antes de tocar nada; la mitad de las veces el mensaje ya trae la respuesta y lo que falta es el hábito de leerlo.

---

## Fuentes de esta sesión

- MDN Web Docs. *JavaScript modules*. https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules
- MDN Web Docs. *JavaScript execution model*. https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Execution_model

La guía de módulos de MDN hay que leerla completa, porque es corta, está escrita en orden y cubre exactamente los casos que usas en el curso, incluida la advertencia sobre servir los archivos por HTTP. La página del modelo de ejecución es distinta: es densa y describe con precisión el ciclo de ejecución hasta el final, las colas y el momento en que se procesan las microtareas. No esperes entenderla en una lectura. Vale la pena verla una vez para saber que las cajas que dibujaste no son una metáfora del profesor sino un mecanismo documentado, y volver a ella cuando algo no ocurra en el orden que esperabas. Es la clase de página que se entiende a la tercera visita, y la tercera visita llega sola durante la Misión 04.

---

## Antes de la sesión 10

Lee la sección "Módulo 3, sesión 10" de `GUIA-DEL-CURSO.md`, y de MDN la página *Using Promises*, solo hasta donde explica el encadenamiento con `.then`. Veinte minutos, no más, y con una instrucción concreta que hace toda la diferencia: **llega con una pregunta escrita.** No importa si es una pregunta ingenua; importa que hayas leído lo suficiente para tener una. La sesión 10 arranca respondiéndolas.

Y llega pensando en el tema de tu trivia para la Misión 04, porque vas a necesitar una API pública que la sirva y elegirla bien te ahorra horas.
