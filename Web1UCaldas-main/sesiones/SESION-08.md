# Sesión 8 · El DOM, eventos y el bucle de juego

**Módulo 3** · Zona 3: La Sala de Máquinas
**Lo que sale de esta noche:** un bucle de juego corriendo con `requestAnimationFrame` y delta de tiempo, y tu tablero de memoria jugable de principio a fin
**Tu misión al terminar:** Misión 03 — Juego de memoria (100 XP · 10 h de trabajo autónomo)

---

## Por qué esta noche cambia tu forma de programar

Hasta ahora todo tu código ha sido una secuencia: empieza, hace cosas, termina. Esta noche escribes por primera vez código que **no controla cuándo se ejecuta**.

Un manejador de evento es una función que dejas guardada y que el navegador llamará cuando quiera, tal vez nunca, tal vez tres veces en doscientos milisegundos mientras otra cosa está a medio terminar. Ese cambio de modelo mental es más grande que cualquier sintaxis nueva, y es tu primera colisión frontal con la asincronía. Es mejor tenerla aquí, con ayuda al lado, que el domingo a medianoche.

Y hay una segunda cosa que se instala hoy: **el estado del juego es tuyo y el DOM es solo el dibujo.** Si esta noche te acostumbras a preguntarle al DOM en qué estado está el juego, en la sesión 12 con React vas a tener que desaprenderlo con dolor.

Una frase para dejar colgada toda la noche, porque es el requisito de la misión que va a costarte más que todo el resto junto: **no se pueden voltear tres cartas a la vez.** Todavía no te voy a decir por qué es difícil.

---

## Qué es exactamente el DOM

Antes de leer lo que sigue, respondete esto: **cuando escribes `<div>` en un archivo HTML, ¿qué es exactamente lo que manipula JavaScript después? ¿El archivo? ¿El texto? ¿Otra cosa?**

Casi todo el mundo dice "el HTML", y es la respuesta equivocada. Lo que casi nunca se dice explícitamente es esto: **el archivo HTML se lee una sola vez, al comienzo, y a partir de ahí no vuelve a importar.** Lo que el navegador construyó al leerlo es una estructura de objetos en memoria —el DOM— y eso es lo único que JavaScript toca. El archivo en disco no cambia nunca. Si recargas, el navegador vuelve a leer el archivo y todo lo que JavaScript había hecho desaparece.

La analogía: el HTML es **el plano de una casa** y el DOM es **la casa construida**. Puedes tumbar una pared en la casa y el plano no se entera. Y si mandas construir otra casa con el mismo plano, sale sin el cambio.

### Seleccionar y modificar, en un minuto

Esta parte es vocabulario. `querySelector` devuelve el primer elemento que coincide con un selector de CSS, o `null`; `querySelectorAll` devuelve una lista de todos. Esa lista **no es un arreglo**, y por eso a veces `map` no está disponible; la solución es `[...document.querySelectorAll(".carta")]`.

Para crear contenido, `document.createElement` y `appendChild`. Para el texto, `textContent`. Y aquí detente diez segundos: **`textContent` inserta texto y `innerHTML` interpreta marcado.** Si el contenido viene de algo que escribió un usuario o de una API, `innerHTML` es una puerta de entrada para inyectar código. La regla del curso: `textContent` siempre, `innerHTML` solo cuando tú generaste la cadena entera y sabes qué hay dentro. En el módulo 4, cuando muestres la salida de un modelo de lenguaje, esta misma regla vuelve con más peso.

Para cambiar apariencia usa `classList.add`, `remove` y `toggle`, en lugar de escribir `elemento.style.background` a mano. La razón no es estética: es que **la apariencia se declara en CSS y JavaScript solo cambia el nombre del estado.** Eso mantiene una frontera limpia.

### El objeto del evento y la propagación

Cuando registras un manejador con `addEventListener`, el navegador te llama pasándote un objeto que describe lo que pasó. Los campos que importan hoy son tres: `event.target`, que es el elemento donde el evento **se originó**; `event.currentTarget`, que es el elemento donde **está registrado el manejador**; y `event.type`. La diferencia entre `target` y `currentTarget` es la base de todo lo que viene después, así que no la pases rápido.

Ahora la propagación. Un evento no ocurre "en un elemento": **hace un viaje completo por el árbol, de arriba abajo y de abajo arriba.** Piénsalo como una gota de agua que cae en un edificio: primero baja desde el techo por el hueco del ascensor buscando el piso donde cayó —esa bajada es la fase de **captura**—, luego toca el piso exacto —la fase de **objetivo**—, y luego el sonido del golpe sube de piso en piso hasta el techo, y en cada piso alguien puede oírlo —esa subida es el **burbujeo**, el *bubbling*.

Lo que esto significa en la práctica: **si haces clic en un botón dentro de un `div` dentro del `body`, el clic también "pasa" por el `div` y por el `body`.** Un manejador registrado en el contenedor se entera del clic en el botón. Eso no es un accidente del diseño: es lo que hace posible la delegación de eventos, y es intencional.

Existen `event.stopPropagation()` para cortar el burbujeo y `event.preventDefault()` para cancelar la acción por defecto del navegador —que un enlace navegue, que un formulario se envíe—. Son cosas distintas que se confunden todo el tiempo: uno detiene el viaje del evento, el otro detiene lo que el navegador iba a hacer.

### Delegación de eventos

Piensa primero en el problema. Un tablero de memoria de dieciséis cartas: la reacción natural es recorrer las cartas y ponerle un `addEventListener` a cada una. Dieciséis manejadores. Funciona.

Ahora dos preguntas que rompen ese planteamiento. Primera: *¿qué pasa si el tablero es de cien cartas?* Cien manejadores, cien referencias vivas en memoria. Segunda, y esta es la que importa: *¿qué pasa si las cartas se crean después, al empezar una partida nueva?* Los manejadores estaban atados a los elementos viejos, que ya no existen. Las cartas nuevas nacen muertas y hay que volver a registrar todo.

La delegación invierte el planteamiento: **un solo manejador en el contenedor, que aprovecha el burbujeo y decide qué hacer mirando `event.target`.**

```javascript
const tablero = document.querySelector("#tablero");

// UN manejador para todas las cartas, presentes y futuras
tablero.addEventListener("click", (evento) => {
  // el clic pudo caer en la carta o en algo dentro de ella:
  // closest sube por el árbol hasta encontrar una carta, o devuelve null
  const carta = evento.target.closest(".carta");
  if (!carta) return;                    // el clic cayó en el espacio vacío del tablero
  manejarClicEnCarta(Number(carta.dataset.id));
});
```

Lo que hace que la delegación sea robusta es `closest`. Si la carta contiene un `<span>` con el símbolo, el clic se origina en el `span`, no en la carta, y `event.target` es el `span`. `closest` sube por los ancestros hasta encontrar el que coincide con el selector. Sin `closest`, la delegación falla en cuanto las cartas tienen algo adentro, que es siempre.

Fíjate también en `dataset`, porque es el puente entre el DOM y tu estado: un atributo `data-id="7"` en el HTML se lee como `carta.dataset.id` en JavaScript, y siempre llega como texto, de ahí el `Number`.

### Las ideas que hay que llevarse

**Tu código no decide cuándo se ejecuta.** Un manejador es una función que dejas depositada. El navegador la llama cuando ocurre el evento, cuantas veces ocurra, sin preguntarte si estabas ocupado. Todo el resto del módulo 3 es aprender a convivir con eso.

**El evento viaja por el árbol, y eso es una herramienta.** El burbujeo no es un efecto secundario que haya que tolerar: es lo que permite escuchar en un solo lugar lo que pasa en cien.

**El estado vive en tu programa; el DOM es el dibujo del estado.** Si para saber algo del juego tienes que leer una clase de CSS, la información está en el lugar equivocado. Dibujar es una función de una sola dirección: del estado hacia la pantalla, nunca al revés.

### Ponte a prueba

*Si registras el manejador en el contenedor y el usuario hace clic en el borde vacío del tablero, ¿qué recibe tu función?* Piensa en la línea `if (!carta) return`, que es la que todo el mundo olvida.

*¿Por qué la delegación funciona con elementos que todavía no existen?*

*Si el usuario hace clic tres veces en doscientos milisegundos, ¿cuántas veces corre tu manejador?* Deja la respuesta rondando: es la trampa de la práctica de esta noche.

---

## Práctica en clase: del clic al tablero, con delegación

Este recorrido lo hacemos juntos en clase y está aquí para que puedas repetirlo solo. Se trabaja sobre el archivo de lógica pura de la sesión 7. Hoy le pones la piel.

**Primero, dibujar el tablero desde el estado.** Escribe una función `dibujar(estado)` que vacíe el contenedor y lo reconstruya a partir del arreglo de cartas. Es una función de una sola dirección: recibe estado, produce DOM, no lee nada del DOM.

```javascript
function dibujar(estado) {
  tablero.textContent = "";                      // vaciar
  for (const carta of estado.cartas) {
    const elemento = document.createElement("button");
    elemento.className = "carta " + carta.estado;   // oculta | volteada | emparejada
    elemento.dataset.id = carta.id;
    elemento.textContent = carta.estado === "oculta" ? "?" : carta.simbolo;
    tablero.appendChild(elemento);
  }
  marcador.textContent = `Intentos: ${estado.intentos}`;
}
```

Fíjate en que las cartas son `<button>` y no `<div>`. Un `button` es enfocable con el teclado y anunciable por un lector de pantalla; un `div` con un clic encima no lo es. Es la accesibilidad del módulo 1 apareciendo gratis por haber elegido bien el elemento.

**Segundo, comete el primer error a propósito: un manejador por carta.** Escríbelo así, deliberadamente:

```javascript
// MAL, y a propósito
document.querySelectorAll(".carta").forEach((elemento) => {
  elemento.addEventListener("click", () => voltear(elemento.dataset.id));
});
```

Pruébalo. Funciona, y es importante que veas que funciona, porque el error tiene que ser convincente antes de ser corregido. Ahora rómpelo: llama a `dibujar(estado)` otra vez —simulando "partida nueva"— y haz clic. **Nada responde.** Los manejadores estaban en los elementos viejos, que `textContent = ""` acaba de destruir.

Deja el bloque comentado con el rótulo `// ERROR DEL TALLER 1 — manejadores que mueren al redibujar` y reemplázalo por la delegación de arriba. Repite la prueba: redibuja, haz clic, funciona. Ese contraste es el argumento entero de la delegación y no necesita más palabras.

**Tercero, el segundo error a propósito, y este es el corazón de la noche: las tres cartas.** Escribe el manejador de la forma ingenua, la que escribirías en casa:

```javascript
function manejarClicEnCarta(id) {
  estado = voltear(estado, id);
  dibujar(estado);

  if (estado.volteadas.length === 2) {
    const [a, b] = estado.volteadas.map((i) => estado.cartas.find((c) => c.id === i));
    // esperamos 800 ms para que el jugador alcance a ver las dos cartas
    setTimeout(() => {
      estado = resolverPareja(estado, a, b);
      dibujar(estado);
    }, 800);
  }
}
```

Pruébalo despacio, haciendo clic con calma: funciona perfecto. Ahora **haz clic muy rápido en cuatro cartas distintas y observa qué pasa.** Se voltean tres, cuatro, el contador de intentos se desordena, y en algún momento una carta se queda boca arriba para siempre.

Detente aquí, porque este es el momento más importante de la sesión. Durante esos 800 milisegundos de espera, **tu programa no está haciendo nada, pero el usuario sí.** Los clics siguen llegando y el navegador sigue llamando a tu manejador, porque el manejador no tiene forma de saber que hay una animación en curso. El `setTimeout` no pausa el mundo: solo agenda un trabajo para después y devuelve el control inmediatamente.

Vale la pena repetírtelo: **el tiempo de espera de tu animación es un hueco en el que caben los clics del usuario.**

La solución tiene una línea:

```javascript
function manejarClicEnCarta(id) {
  if (estado.bloqueado) return;          // ← la línea que vale 5 XP de la rúbrica
  estado = voltear(estado, id);
  dibujar(estado);

  if (estado.volteadas.length === 2) {
    estado = { ...estado, bloqueado: true };     // cierro la puerta
    const [a, b] = /* ... */;
    setTimeout(() => {
      estado = { ...resolverPareja(estado, a, b), bloqueado: false };  // la abro
      dibujar(estado);
    }, 800);
  }
}
```

Ahí es donde se cobra el campo `bloqueado` que sembraste la semana pasada. No apareció por magia: lo pusiste cuando diseñaste el estado sin tocar el DOM, y ahora resulta que era la pieza que faltaba. Diseñar el estado bien primero es lo que hace que este bug tenga una solución de una línea en lugar de una tarde de parches.

**Cuarto, el contraste con el DOM.** Pregúntate cómo habrías resuelto lo mismo si el estado viviera en las clases de CSS. La respuesta es contando elementos con `document.querySelectorAll(".volteada").length`, y funciona hasta que la animación de CSS quita la clase antes de que tu código la cuente. La versión con estado propio no tiene esa carrera. No hace falta implementarla: basta con ver dónde está el problema.

**Quinto, la limpieza.** `addEventListener` tiene contraparte, `removeEventListener`, y para que funcione hay que pasar **la misma referencia de función**, no una flecha nueva escrita igual. Prueba esto y observa que no quita nada:

```javascript
tablero.addEventListener("click", (e) => manejar(e));
tablero.removeEventListener("click", (e) => manejar(e));   // NO quita nada: es otra función
```

Es la semilla de las fugas de memoria que en el módulo 4, con la función de limpieza de `useEffect`, vas a volver a encontrar.

### Tres cosas que te van a pasar en este recorrido

Te va a parecer que el juego "va lento" al redibujar todo el tablero en cada clic. Para dieciséis cartas es irrelevante, y redibujar todo es mucho más fácil de razonar que actualizar quirúrgicamente. Ese razonamiento —"redibujo todo conceptualmente y que el sistema optimice"— es exactamente la idea sobre la que está construido React.

Vas a poner el `<script>` en el `<head>` y vas a obtener `null` en todos los `querySelector`. Es un buen error: el script corrió antes de que el navegador construyera esa parte del DOM, así que los elementos no existían. Se arregla poniendo el script al final del `<body>` o con el atributo `defer`. La causa profunda —en qué orden pasan las cosas— es la sesión 9 completa.

Y te vas a preguntar si `dataset.id` es número o texto. Siempre texto. Comparar `"3" === 3` da `false`, lo cual produce un bug silencioso en el que la carta correcta nunca se encuentra. En la sesión 11 el compilador de TypeScript te habría avisado de esto antes de ejecutar nada.

---

## El bucle de juego y el delta de tiempo

Un programa normal termina. Un juego no: **un juego es un ciclo que se repite hasta que alguien lo detenga**, y en cada repetición hace tres cosas en el mismo orden. Primero **lee la entrada**: qué teclas están presionadas, dónde está el puntero. Segundo **actualiza el estado**: mueve al jugador, mueve a los enemigos, revisa colisiones, cuenta puntos. Tercero **dibuja**: pinta el estado actual en la pantalla. Y vuelve a empezar.

Lo importante es la separación entre la segunda y la tercera: **actualizar y dibujar son trabajos distintos y no deben mezclarse.** Si dentro de la función que mueve al enemigo también estás pintando, no puedes cambiar el dibujo sin tocar la lógica ni probar la lógica sin dibujar. Es lo mismo que hiciste esta noche con `voltear` y `dibujar`, ahora a sesenta veces por segundo.

### Nunca animes con `setInterval`

Esto es lo que escribirías por instinto:

```javascript
// MAL, y ahora vamos a ver por qué
let x = 0;
setInterval(() => {
  x += 5;                       // "5 píxeles cada vez"
  nave.style.left = x + "px";
}, 16);                         // "16 ms ≈ 60 veces por segundo"
```

Parece razonable y tiene tres problemas graves.

**El primero: `setInterval` no garantiza nada.** Los 16 milisegundos son una petición, no un contrato. Si el hilo está ocupado, el disparo se retrasa. Y como los disparos se agendan de todos modos, se pueden **acumular**: cuando el navegador se libera, corren varios seguidos de golpe y la animación da un salto.

**El segundo: el navegador no está pintando cuando tú quieres.** El navegador pinta cuando puede, sincronizado con el refresco de la pantalla. Tu `setInterval` cambia posiciones en momentos que no coinciden con ese refresco, así que algunos cambios se pintan y otros se sobrescriben antes de ser vistos. El resultado es el *jank*: movimiento que se siente entrecortado aunque los números sean correctos.

**El tercero, y el que hay que subrayar: la velocidad del juego depende de la máquina.** Si el intervalo dispara 60 veces por segundo, la nave avanza 300 píxeles por segundo. Si la máquina está cargada y solo dispara 40 veces, avanza 200. Si es una pantalla de 144 Hz y el motor le da más disparos, avanza más. **El mismo código produce un juego distinto en cada computador.**

Y el detalle que nadie espera: si el usuario cambia de pestaña, el navegador limita agresivamente los temporizadores en segundo plano. Al volver, el juego está retrasado o hace un salto. Eso explica el "el temporizador no se pausa al minimizar la pestaña" que aparece en la plantilla de PR del curso.

### `requestAnimationFrame`

`requestAnimationFrame` invierte quién manda. Con `setInterval` tú le dices al navegador cuándo quieres correr; con `requestAnimationFrame` le pides al navegador que te avise **justo antes de que vaya a pintar el siguiente fotograma**.

La analogía: `setInterval` es **poner una alarma cada 16 minutos para llegar al bus**. A veces llegas antes y esperas, a veces el bus ya se fue. `requestAnimationFrame` es **que el conductor te llame cuando esté llegando a la esquina**. Nunca pierdes el bus y nunca esperas de más.

Las tres consecuencias prácticas: el navegador sincroniza tu función con el refresco real de la pantalla; si la pestaña deja de estar visible, **deja de llamarte** en lugar de acumular trabajo; y como te pasa una marca de tiempo, puedes saber exactamente cuánto pasó desde el fotograma anterior. Eso último es la puerta al delta de tiempo.

```javascript
let anterior = 0;

function bucle(ahora) {          // el navegador te pasa la marca de tiempo, en milisegundos
  const delta = (ahora - anterior) / 1000;   // segundos transcurridos desde el fotograma previo
  anterior = ahora;

  actualizar(delta);
  dibujar();

  requestAnimationFrame(bucle);  // pido el siguiente fotograma
}
requestAnimationFrame(bucle);    // arranco
```

No hay `while` ni `for`: el bucle se sostiene porque cada llamada pide la siguiente. Si dejas de pedirla, el juego se detiene, y eso es precisamente cómo se implementa la pausa.

### El delta de tiempo, que es la idea de la noche

**Un juego sin delta de tiempo razona en "píxeles por fotograma"; un juego con delta razona en "píxeles por segundo".** La diferencia es la misma que hay entre decir "avanzo un paso cada vez que pestañeo" y "avanzo a cinco kilómetros por hora". La primera medida depende de con qué frecuencia pestañees; la segunda no depende de nada.

```javascript
// MAL: depende de la máquina
x += 5;

// BIEN: 300 píxeles por segundo, en cualquier máquina
const VELOCIDAD = 300;
x += VELOCIDAD * delta;
```

Los números convencen, así que hazlos: a 60 fotogramas por segundo, `delta` vale aproximadamente 0.0167 y el desplazamiento por fotograma es 5 píxeles. A 30 fotogramas por segundo, `delta` vale 0.0333 y el desplazamiento por fotograma es 10 píxeles. **El doble de avance en la mitad de fotogramas: el mismo movimiento por segundo.** El juego se ve más fluido en la máquina rápida y más brusco en la lenta, pero **es el mismo juego**, y eso es exactamente lo que buscas.

Un detalle que va a fallar en tu proyecto: el primer fotograma. Si `anterior` arranca en 0, el primer `delta` es enorme —el valor completo del reloj del navegador— y todo salta al infinito en el primer instante. Se resuelve inicializando `anterior` en el primer fotograma, o poniéndole un techo al delta: `const delta = Math.min((ahora - anterior) / 1000, 0.05)`. Ese techo además te protege del salto al volver de otra pestaña.

### Las ideas que hay que llevarse

**Nunca animes con `setInterval`.** No garantiza el intervalo, no está sincronizado con el pintado, acumula disparos y hace que tu juego corra a velocidades distintas en máquinas distintas. Para animar, `requestAnimationFrame`, siempre.

**El movimiento se expresa en unidades por segundo, no por fotograma.** Multiplica por el delta y tu juego corre igual en la máquina lenta del compañero y en la tuya con 144 Hz.

**No acumules incrementos: mide contra una referencia.** Es la misma lección del delta aplicada a cualquier medición de tiempo.

### Ponte a prueba

*Si tu juego se ve bien en tu máquina y al doble de velocidad en otra, ¿qué línea de tu código es la culpable?*

*¿Por qué el bucle de `requestAnimationFrame` no necesita un `while`?*

*¿Qué pasa con el delta la primera vez que corre el bucle, y qué dos formas hay de arreglarlo?*

---

## Práctica en clase: tu propio bucle

**Tramo A: el bucle.** Un archivo aparte, `bucle.html`, con un cuadrado que se mueve de izquierda a derecha y rebota en los bordes. Tres requisitos: que use `requestAnimationFrame`, que la velocidad esté expresada en píxeles por segundo con delta, y que la barra espaciadora pause y reanude.

Y una prueba que no te puedes saltar, porque es lo que hace que la lección quede: **escribe primero la versión con `x += 5` sin delta, déjala corriendo, y luego abre las herramientas de desarrollo y limita el rendimiento de la CPU** (en el panel de rendimiento hay una opción de estrangulamiento, 4x o 6x). Con la versión sin delta, el cuadrado se arrastra. Con la versión con delta, avanza igual, solo con menos fotogramas. Mira las dos, una al lado de la otra, con la CPU frenada. Ese experimento vale más que toda la teoría de arriba.

Para la pausa, la implementación correcta es dejar de pedir fotogramas, y ahí hay una trampa: al reanudar, `anterior` quedó viejo, así que el primer delta después de la pausa es gigante y el cuadrado teletransporta. La solución es reiniciar `anterior` al reanudar. Descúbrelo y arréglalo; es un bug excelente.

**Tramo B: arranque de la Misión 03.** Con la lógica pura de la sesión 7 y la delegación de esta noche, el objetivo es tener un tablero jugable de principio a fin, aunque sea feo. Revisa cuatro cosas, en este orden: que el barajado sea Fisher-Yates y no `sort` con random; que haya **un solo** `addEventListener` en el contenedor y no uno por carta; que exista la guarda de bloqueo y que la pruebes haciendo clic como loco, que es la única prueba válida; y que la consola esté limpia, porque cero errores durante una partida completa vale 5 XP de la rúbrica y es lo primero que se revisa al calificar.

El temporizador conviene dejarlo para la casa, con una advertencia: si lo haces con `setInterval` de un segundo para mostrar el reloj, está bien —el reloj no es una animación—, pero no calcules el tiempo transcurrido sumando segundos en cada disparo, porque los disparos se retrasan y el reloj se atrasa. La forma correcta es guardar la marca de tiempo de inicio y calcular la diferencia contra `Date.now()` cada vez que dibujas.

---

## Tu misión de la semana: Juego de memoria (100 XP)

Construyes el clásico de voltear cartas y buscar parejas, **con JavaScript puro, sin librerías**.

Necesita: mezclado aleatorio correcto, control de estado para que no se puedan voltear tres cartas a la vez, contador de intentos, detección de victoria y temporizador.

### Por qué el detalle de las tres cartas

Ese requisito parece trivial y es donde casi todos fallan. La razón por la que está ahí es que **revela si entendiste que los eventos llegan mientras una animación está en curso.** Un juego que solo funciona si el jugador hace clic despacio no es un juego que funciona: es un juego que todavía no se ha probado. Y la solución no es un parche: es la consecuencia de haber diseñado el estado antes de tocar el DOM.

### Cómo se califica

| Criterio | XP |
|---|---|
| Funciona: el juego se puede jugar de principio a fin | 35 |
| Calidad del código: nombres claros, sin repetición ni código muerto | 25 |
| Específico del módulo (ver abajo) | 20 |
| Proceso en Git: commits pequeños y descriptivos, PR completo | 10 |
| Auditoría de IA: `IA.md` real y reflexivo | 10 |

Los 20 XP específicos de esta misión: el mezclado es un Fisher-Yates correcto y no un `sort` con random (5) —y ya mediste por qué—; el estado del juego está centralizado y no repartido en el DOM (5), o sea que si tienes que leer una clase de CSS para saber qué está pasando, pierdes estos puntos; usas delegación de eventos (5), un manejador y no dieciséis; y cero errores en consola durante una partida completa (5), que se comprueba abriendo la consola y jugando hasta ganar.

El commit `fix(memoria): evitar voltear tres cartas a la vez` es el que se quiere ver en tu historial, porque demuestra que encontraste el bug y lo resolviste en lugar de haberlo evitado por casualidad.

Sobre la IA, sé consciente esta semana porque es tu primera misión con lógica de verdad: pedirle a un asistente el juego de memoria completo va a producir código que corre y que casi seguro tiene el bug de las tres cartas o un barajado sesgado. Está permitido usarlo. Lo que no admite excepción es la declaración en `IA.md`: qué pediste, qué te devolvió, qué estaba mal y qué corregiste. **Sin declarar, la misión se califica en cero y no admite reintento. Declararlo NO baja la nota**, porque lo que se evalúa es tu capacidad de auditar. Si documentas bien el error del barajado o el de las tres cartas con su corrección, ahí está la insignia 🔍 **Cazador de alucinaciones** y su día extra de plazo acumulable.

**Plazo:** antes de la medianoche del día anterior a la sesión 9. Presupuesto: **10 horas** repartidas en 2 de planear, 6 de programar, 1 de probar y 1 de documentar. Descuenta lo que ya hiciste en las sesiones 7 y 8: vas con ventaja. Si te está tomando mucho más, escribe en lugar de perder la noche.

En la sesión 9 hay **Quiz 3**. Y algo para despertarte la curiosidad: la semana entrante vas a entender por qué el `setTimeout` de 800 milisegundos dejó pasar los clics, y por qué eso es exactamente la misma razón por la que un `await` se ejecuta antes que un `setTimeout(0)`.

---

## Errores que probablemente vas a cometer

**Poner un escuchador por elemento y perderlos al redibujar.** Es el error que más aparece y el más frustrante, porque el código funcionó la primera vez. Cuando el tablero se regenera —partida nueva, nivel nuevo, filtro aplicado— los elementos son nuevos y los manejadores se quedaron con los viejos. El síntoma que vas a reportar es "deja de responder después de la primera partida", y la causa nunca está donde la vas a buscar. La delegación lo elimina de raíz.

**Guardar el estado del juego en clases de CSS.** Preguntar `classList.contains("volteada")` para saber si una carta está volteada es cómodo y es una trampa. El DOM lo modifican las animaciones, las transiciones y el navegador, así que la información que lees puede estar a medio camino entre dos estados. Además hace imposible probar tu lógica sin abrir el navegador. El estado se lee del objeto de estado, siempre.

**Animar con `setInterval` y mover en píxeles por fotograma.** El juego va a funcionar perfecto en la máquina en la que lo escribiste y al doble de velocidad o a la mitad en la del compañero. Como pruebas siempre en el mismo computador, el bug es invisible hasta que alguien más lo abre, y para entonces tu código está lleno de números ajustados a mano que no significan nada. Estrangular la CPU en las herramientas de desarrollo debería ser un paso de prueba obligatorio para ti.

**No proteger la ventana de la animación.** Vas a aceptar clics durante los 800 milisegundos en los que el juego está "esperando", y todo el estado se va a desordenar. Detrás hay el malentendido más profundo del módulo: creer que mientras un `setTimeout` está pendiente el programa está detenido. No lo está. El `setTimeout` agendó algo y devolvió el control inmediatamente, y ese hueco es tiempo en el que el usuario puede hacer lo que quiera.

---

## Fuentes de esta sesión

- MDN Web Docs. *Anatomy of a video game*. https://developer.mozilla.org/en-US/docs/Games/Anatomy
- MDN Web Docs. *Window: requestAnimationFrame()*. https://developer.mozilla.org/en-US/docs/Web/API/Window/requestAnimationFrame
- MDN Web Docs. *Canvas API tutorial*. https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API/Tutorial

*Anatomy of a video game* es la mejor lectura de una sola sentada que existe sobre este tema y explica con dibujos por qué `setInterval` no sirve para animar. La referencia de `requestAnimationFrame` es corta y conviene leerla completa, sobre todo la parte de qué contiene la marca de tiempo que recibes. El tutorial de Canvas queda como puerta abierta: la Misión 03 no lo necesita, pero si quieres hacer tu juego de memoria con Canvas en lugar de con elementos del DOM, ahí está todo lo que hace falta.

---

## Antes de la sesión 9

Lee la sección "Módulo 3, sesión 9" de `GUIA-DEL-CURSO.md` y la página de MDN sobre módulos de JavaScript, solo hasta la parte de `export` e `import` con nombre. Diez minutos.

Y una tarea de un minuto que sí es obligatoria: escribe en un papel, **antes de llegar a clase y sin ejecutarlo**, qué imprime este fragmento y en qué orden. Trae el papel.

```javascript
console.log("1");
setTimeout(() => console.log("2"), 0);
Promise.resolve().then(() => console.log("3"));
console.log("4");
```
