# Sesión 7 · JavaScript moderno y programación funcional

**Módulo 3** · Zona 3: La Sala de Máquinas
**Lo que sale de esta noche:** el módulo de lógica pura de tu juego de memoria, barajado con Fisher-Yates y sin una sola mutación de estado compartido
**Tu misión al terminar:** esta noche no se asigna misión nueva. La Misión 03 se asigna en la sesión 8, y lo que escribas hoy es la mitad difícil de esa misión, hecha con anticipación (presupuesta 1 h 30 min, que salen de las 10 h de la Misión 03 y no se suman)

---

## Por qué empezamos por aquí

Ya escribiste marcado y estilos durante dos módulos, y el semestre pasado probablemente programaste con ciclos anidados y variables que se modifican en el lugar. Ese estilo funciona perfectamente mientras tu programa tiene un solo flujo: empieza, hace cosas, termina.

En cuatro semanas vas a tener eventos que llegan cuando quieren, animaciones corriendo sesenta veces por segundo y respuestas de red que aparecen en un orden impredecible. En ese mundo, **un estado que cualquiera puede modificar en cualquier momento es la fábrica de errores que no se pueden reproducir.**

Así que el objetivo declarado de esta sesión es que manejes la sintaxis moderna de JavaScript —`let` y `const`, funciones flecha, desestructuración, propagación y los métodos de arreglo—, y el objetivo real es otro: **que dejes de pensar en JavaScript como "C con llaves distintas"**. Esta no es "la sesión de sintaxis". Es la sesión donde se instala la disciplina que hace posible depurar código asíncrono.

Una advertencia honesta antes de arrancar, porque la honestidad compra paciencia: este módulo recibe el doble de horas que los demás porque el asincronismo es donde se estrella casi todo el mundo. Si en algún momento de las próximas cinco semanas sientes que no entiendes nada, es exactamente lo que le pasa a todos y no es señal de que estés perdido. Mira la tabla de niveles y fíjate en el nivel 4: **Domador del asincronismo, 700 XP, el nivel que equivale a aprobar.** No es casualidad que el nivel de aprobación lleve ese nombre. Desde esta noche empiezas a subir esa cuesta.

---

## Dónde vive una variable

Antes de leer lo que sigue, respondete esto sin ejecutarlo. **¿Qué imprime este fragmento?**

```javascript
const botones = [];
for (var i = 0; i < 3; i++) {
  botones.push(function () {
    console.log("soy el botón", i);
  });
}
botones.forEach(fn => fn());
```

Si dijiste `0, 1, 2`, estás con la mayoría. Imprime **`3, 3, 3`**.

Ahora cambia `var` por `let`, una sola palabra, y vuelve a ejecutarlo: `0, 1, 2`. Esa diferencia de tres letras es la puerta de entrada a todo el bloque, y conviene que la sientas antes de leer la explicación, porque nadie olvida un resultado que contradijo su predicción.

### Por qué pasa: ámbito de función frente a ámbito de bloque

`var` tiene **ámbito de función**. No importa dónde la declares, la variable pertenece a la función completa que la contiene. En el ejemplo hay una sola `i` para todo el bucle, y las tres funciones que guardaste en el arreglo no capturaron el valor de `i`: capturaron **la variable misma**. Cuando las ejecutas, el bucle ya terminó y esa única `i` vale 3.

`let` tiene **ámbito de bloque**. Cada iteración del `for` crea una `i` nueva, y cada función captura la suya. Tres variables distintas, tres valores distintos.

Aquí entra el término que vas a leer en toda la documentación: eso que hace la función cuando "se lleva" la variable con la que nació se llama **closure**. Una función en JavaScript no guarda copias de los valores que usa; guarda una referencia al entorno donde fue creada, y ese entorno sigue vivo mientras la función siga viva. Es un mecanismo potentísimo y es la causa de la mitad de las sorpresas del lenguaje.

Vas a encontrar también la palabra **hoisting**, así que conviene saber qué significa. Las declaraciones con `var` se "elevan" al inicio de la función con valor `undefined`, así que puedes leer una `var` antes de escribirla y obtener `undefined` en lugar de un error. Con `let` y `const` la variable también se eleva, pero queda en una zona donde leerla es un error explícito: la **temporal dead zone**. Que un error salte en la línea correcta en lugar de propagar un `undefined` silencioso es exactamente la clase de mejora que justifica el cambio.

La regla operativa del curso, que vas a ver repetida hasta el final del semestre: **`const` por defecto, `let` solo cuando el valor realmente tenga que cambiar, `var` nunca.**

Y de una vez la confusión más común: `const` no significa que el valor sea inmutable, significa que **el nombre no se puede reasignar**. Un `const` que apunta a un arreglo permite perfectamente hacerle `push`. Escríbelo tú mismo en la consola:

```javascript
const mazo = ["A", "B"];
mazo.push("C");        // válido: el arreglo cambió, el nombre sigue apuntando al mismo arreglo
// mazo = [];          // error: esto sí es reasignar el nombre
```

Esa distinción entre *reasignar el nombre* y *modificar el contenido* es la que sostiene todo lo que viene después.

### Funciones flecha y el `this` que nadie entiende

La parte fácil es la sintaxis. Una función flecha es una función más corta, y cuando el cuerpo es una sola expresión, el `return` es implícito:

```javascript
const dobleDanio = (base) => base * 2;                 // return implícito
const nombreLargo = (h) => h.nombre.length > 10;       // devuelve un booleano
const crearOro = (cantidad) => ({ tipo: "oro", cantidad }); // ¡paréntesis para devolver un objeto!
```

Fíjate en ese último detalle porque te va a morder: si devuelves un objeto literal con return implícito, hay que envolverlo en paréntesis, porque de otro modo las llaves se leen como el cuerpo de la función.

Y ahora la parte que importa de verdad. **La función flecha no es solo sintaxis corta: cambia el significado de `this`.**

En una función normal, `this` funciona como la palabra **"aquí"** dicha por teléfono: no significa nada por sí sola, significa lo que sea el lugar de quien está hablando en ese momento. Si te llamo desde Manizales, "aquí" es Manizales; si te llamo desde Pereira, "aquí" es Pereira. La misma palabra, dos lugares, y no lo decide la palabra sino **quién la pronuncia**. En una función normal, `this` lo decide **quien llama a la función**, no quien la escribió.

En una función flecha, `this` funciona como **una dirección escrita en un sobre**: quedó fija cuando alguien la escribió y no cambia porque el sobre pase de mano en mano. La flecha no tiene su propio `this`; toma prestado el del lugar donde fue *escrita*, para siempre.

Prueba esto y observa qué pasa:

```javascript
const jugador = {
  nombre: "Nyx",
  vida: 100,
  recibirDanio(cantidad) {
    this.vida -= cantidad;                  // aquí `this` es jugador: lo llamó jugador.recibirDanio()
    console.log("vida ahora:", this.vida);   // 80

    setTimeout(function () {
      // ¿quién llamó a esta función? El temporizador, no jugador.
      console.log(this.nombre, "queda con", this.vida); // undefined undefined
    }, 500);
  }
};
jugador.recibirDanio(20);
```

`undefined undefined`. La razón es que nadie llamó a esa función interna "a través de" `jugador`; la llamó el temporizador del navegador. El "aquí" del que habla se lo puso otro.

Cambia únicamente `function ()` por `() =>` y vuelve a ejecutar. Funciona. La flecha no preguntó quién la llamaba: se quedó con el `this` del sitio donde fue escrita, que es el interior de `recibirDanio`, donde `this` era `jugador`.

Y ahora el error simétrico, que es el que más aparece en las entregas:

```javascript
const enemigo = {
  nombre: "Gólem de piedra",
  // MAL: como método, la flecha toma el `this` del módulo, no del objeto
  presentarse: () => console.log("Soy", this.nombre),  // undefined
  // BIEN
  atacar() { console.log(this.nombre, "ataca"); }
};
```

La regla práctica: **flecha para funciones que se pasan como argumento —callbacks, manejadores de eventos, temporizadores, argumentos de `map`—; función normal para métodos de un objeto.** No es una regla de estilo, es una consecuencia directa de cómo se resuelve `this`.

### Las ideas que hay que llevarse

**El ámbito es de bloque, y eso hace que el código sea predecible.** Una variable declarada dentro de un `if`, un `for` o un par de llaves existe únicamente ahí. Cuando dentro de un mes tengas un bucle de juego con diez variables temporales, esa contención es lo que evita que se pisen entre sí.

**`const` congela el nombre, no el contenido.** Es la fuente de la mitad de las discusiones inútiles sobre inmutabilidad. Lo que hace que un objeto no cambie no es la palabra `const`: es que tu código no lo modifique.

**En una función normal, `this` lo decide quien llama; en una flecha, quien escribe.** Si te llevas una sola frase de la noche, que sea esta.

### Ponte a prueba

*Si `const` no impide modificar un arreglo, ¿para qué sirve entonces?*

*¿Por qué un manejador de evento escrito con `function` puede necesitar `this` y uno con flecha no?*

*¿Qué pasa si usas una flecha como método de un objeto que se usa dentro de un `setInterval`?* La respuesta corta es "nada bueno", y entender por qué es combinar las dos reglas de arriba.

---

## Práctica en clase: el inventario del héroe

Este recorrido lo hacemos juntos en clase, pero está aquí para que puedas repetirlo solo. Es de consola y de un solo archivo `inventario.js` que corres con el navegador o con Node. No lo copies y pegues: escríbelo, porque teclear el punto y coma también enseña.

Parte de este arreglo:

```javascript
const inventario = [
  { nombre: "Espada oxidada",  tipo: "arma",    peso: 3.5, valor: 12,  rareza: 1 },
  { nombre: "Poción de vida",  tipo: "consumo", peso: 0.4, valor: 30,  rareza: 2 },
  { nombre: "Escudo de roble", tipo: "armadura",peso: 5.0, valor: 45,  rareza: 1 },
  { nombre: "Amuleto de Nyx",  tipo: "reliquia",peso: 0.1, valor: 500, rareza: 5 },
  { nombre: "Poción de maná",  tipo: "consumo", peso: 0.4, valor: 25,  rareza: 2 }
];
```

**Primero, el ciclo de toda la vida.** Escribe la versión que habrías escrito el semestre pasado: filtrar los consumibles con un `for` clásico, un arreglo vacío y un `push`. Cuenta las líneas. Cinco, y tres de ellas son andamiaje: el índice, la comparación, el incremento. Ninguna de esas tres dice nada sobre el problema.

```javascript
const consumibles = [];
for (let i = 0; i < inventario.length; i++) {
  if (inventario[i].tipo === "consumo") {
    consumibles.push(inventario[i]);
  }
}
```

**Segundo, la misma cosa con `filter`.** Una línea, y la línea dice literalmente el problema: quiero los que cumplen esta condición.

```javascript
const consumibles = inventario.filter((objeto) => objeto.tipo === "consumo");
```

Esta es la idea que gobierna todo el recorrido: el ciclo describe *cómo* recorrer; el método describe *qué* quieres. Eso es todo lo que significa "programación funcional" a este nivel.

**Tercero, `map` y la trampa de creer que modifica.** Sube todos los precios un 20 %:

```javascript
const conInflacion = inventario.map((objeto) => ({ ...objeto, valor: objeto.valor * 1.2 }));
console.log(inventario[0].valor);   // 12  ← el original NO cambió
console.log(conInflacion[0].valor); // 14.4
```

Detente en el `...objeto`. Ese es el operador de **propagación** (*spread*), y lo que hace es copiar las propiedades del objeto original en uno nuevo; la clave `valor` que escribes después gana porque va al final. Esa es la forma canónica de "cambiar" un objeto sin cambiarlo, y la vas a escribir mil veces en el módulo 4 con React.

**Cuarto, comete el error a propósito.** Escribe esta versión y déjala en el archivo:

```javascript
// MAL, y a propósito: esto muta el inventario original
const conInflacionMal = inventario.map((objeto) => {
  objeto.valor = objeto.valor * 1.2;   // ¡modifica el objeto que ya estaba en el arreglo!
  return objeto;
});
console.log(inventario[0].valor);      // 14.4 ← el "original" cambió
```

Ahora ejecútalo dos veces seguidas en la misma consola y mira el valor: sube otra vez, a 17.28. Ese es el punto. **Una función que ensucia lo que recibe da un resultado distinto cada vez que la llamas, y eso es exactamente lo que significa "no determinista".** Cuando en tres semanas tengas un juego donde el puntaje se reinicia solo, la causa va a ser un primo de este error. Deja el bloque comentado con un rótulo `// ERROR DEL TALLER — no borrar`.

**Quinto, `reduce`, despacio.** `reduce` es el que cuesta y merece tres minutos de calma. La analogía que funciona: **`reduce` es la bola de nieve rodando por la ladera.** Empieza con un tamaño inicial, pasa por cada objeto del camino y se lleva algo; lo que llega al final es la bola completa. El primer parámetro es la bola, el segundo es el objeto que está pisando.

```javascript
const pesoTotal = inventario.reduce((acumulado, objeto) => acumulado + objeto.peso, 0);
// 9.4
```

No olvides el `0` del final, que es justo lo que todo el mundo olvida. Sin valor inicial, `reduce` toma el primer elemento del arreglo como bola de nieve, y si el primer elemento es un objeto y tú estabas sumando números, obtienes `[object Object]0.4` y una tarde perdida.

`reduce` no solo suma: agrupar por tipo es el mismo mecanismo con un objeto como bola de nieve.

```javascript
const porTipo = inventario.reduce((grupos, objeto) => {
  const yaHay = grupos[objeto.tipo] ?? [];
  return { ...grupos, [objeto.tipo]: [...yaHay, objeto.nombre] };
}, {});
// { arma: [...], consumo: [...], armadura: [...], reliquia: [...] }
```

**Sexto, `find`, `some` y `every` en un minuto cada uno.** `find` devuelve el primer elemento que cumple, o `undefined`; `some` responde "¿hay al menos uno?"; `every` responde "¿todos?". Lo importante es que los tres **cortan en cuanto tienen la respuesta**, a diferencia de `filter`, que siempre recorre todo. Y que `find` devuelve el objeto mientras `filter` devuelve un arreglo de un elemento: confusión clasiquísima.

```javascript
const reliquia   = inventario.find((o) => o.rareza === 5);        // el objeto
const hayArmas   = inventario.some((o) => o.tipo === "arma");     // true
const todoLigero = inventario.every((o) => o.peso < 4);           // false
```

**Séptimo, el segundo error a propósito: `sort` muta.** Prueba esto y observa:

```javascript
const masValiosos = inventario.sort((a, b) => b.valor - a.valor);
console.log(inventario[0].nombre); // "Amuleto de Nyx" ← ¡el inventario se reordenó solo!
```

Antes de seguir, respondete: ¿`masValiosos` e `inventario` son dos arreglos o uno? Compruébalo: `masValiosos === inventario` da `true`. Son el mismo. `sort` es uno de los pocos métodos que **ordena en el lugar y además devuelve el mismo arreglo**, así que parece funcional y no lo es. La corrección es de tres caracteres:

```javascript
const masValiosos = [...inventario].sort((a, b) => b.valor - a.valor); // copia primero
```

Y de paso, el clásico: ejecuta `[10, 9, 100].sort()` y mira el resultado, `[10, 100, 9]`. Sin función comparadora, `sort` convierte todo a texto y compara alfabéticamente. Es un error que aparece en tablas de puntajes todos los semestres.

**Octavo, desestructuración, para cerrar.** Reescribe una función para que la firma diga qué necesita:

```javascript
// Antes: hay que leer el cuerpo para saber qué campos usa
function describir(objeto) {
  return `${objeto.nombre} (${objeto.tipo}) pesa ${objeto.peso}`;
}

// Después: la firma es la documentación, y `rareza` tiene valor por defecto
function describir({ nombre, tipo, peso, rareza = 1 }) {
  return `${nombre} (${tipo}) pesa ${peso}, rareza ${rareza}`;
}
```

Y la desestructuración de arreglos con resto, que vas a necesitar para barajar:

```javascript
const [primero, segundo, ...resto] = inventario;
const { posicion: { x, y } } = { posicion: { x: 4, y: 7 } };  // anidada
```

### Tres dudas que te van a salir en este recorrido

Te vas a preguntar si `map` es más lento que un `for`. La respuesta honesta: en microcomparaciones sí, marginalmente, y es completamente irrelevante para todo lo que vas a escribir este semestre. Optimizar antes de medir es perder el tiempo, y en la sesión 18 vas a medir de verdad.

Vas a intentar usar `forEach` esperando que devuelva algo. Compruébalo: `const x = arreglo.forEach(...)` deja `x` en `undefined`. `forEach` existe para provocar efectos, no para producir valores; si quieres un valor de vuelta, el método es otro.

Y vas a descubrir que la copia con `...` "no funciona" cuando hay objetos anidados. La propagación copia **un nivel**. Si un objeto tiene dentro otro objeto, la copia y el original comparten ese objeto interno. Es la diferencia entre copia superficial y copia profunda, y hoy alcanza con que sepas que existe y que en la Misión 03 tu estado debe ser plano por esta razón. Sí, existe `structuredClone` y hace copia profunda; hoy no lo necesitas.

---

## Inmutabilidad y estado compartido

Este bloque no trae sintaxis nueva: le pone nombre y justificación a lo que acabas de hacer con las manos, que es el orden correcto de aprender las cosas.

Tienes dos versiones de la misma función: una que devuelve un arreglo nuevo y otra que modifica el que recibió. Las dos "funcionan" si las corres una vez. La diferencia aparece cuando el programa crece.

La analogía es **la pizarra de la sala**. Imagina un tablero de juego dibujado en una pizarra en el centro del salón, y cinco personas con marcador: la que mueve al jugador, la que mueve a los enemigos, la que dibuja, la que cuenta el puntaje y la que guarda la partida. Todas escriben sobre la misma pizarra, cuando les toca, sin avisar. Cuando el puntaje aparezca mal, la pregunta "¿quién lo cambió?" no tiene respuesta, porque cualquiera pudo. Ahora imagina la alternativa: cada persona recibe una foto de la pizarra, hace su cambio sobre **una foto nueva** y la entrega. Ahora sí hay respuesta, porque cada versión tiene un autor.

Eso es la inmutabilidad, y la razón de fondo es esta: **si nadie modifica lo que recibe, el estado tiene historia, y un error con historia se puede depurar.** No es una preferencia estética de programadores funcionales; es lo que hace posible reproducir un bug.

Ahora la información que vas a necesitar mañana, y conviene tenerla clara. Los métodos que **devuelven algo nuevo y dejan el original intacto** son `map`, `filter`, `reduce`, `slice`, `concat` y la propagación con `...`. Los que **modifican el arreglo en el lugar** son `push`, `pop`, `shift`, `unshift`, `splice`, `sort` y `reverse`. Los dos últimos son los traicioneros, porque además devuelven el arreglo y por eso se cuelan en cadenas que parecen funcionales.

Existen versiones que no mutan de esos dos: `toSorted` y `toReversed`, incorporadas a la especificación en ediciones recientes de ECMAScript. Revísalas en MDN antes de usarlas, porque el soporte depende del entorno; mientras tanto `[...arreglo].sort()` funciona en todas partes.

### El costo, dicho sin idealizar

La inmutabilidad no es gratuita. Copiar cuesta memoria y tiempo. Si en el bucle de juego de la semana que viene copias un arreglo de mil partículas sesenta veces por segundo, lo vas a notar. La regla razonable, y la que se califica en este curso: **el estado del juego se maneja de forma inmutable; los cálculos internos de un bucle caliente pueden mutar variables locales que nadie más ve.** La diferencia clave es la palabra *compartido*. Mutar una variable local dentro de una función que nadie más toca no le hace daño a nadie. Mutar el objeto que otras cuatro funciones también leen es el problema.

### Lo que esto significa para tu nota

De los 20 XP específicos del módulo en la Misión 03, cinco se van en que **el estado del juego esté centralizado y no repartido en el DOM**. Concretamente: si para saber si una carta está volteada tu código pregunta `carta.classList.contains("volteada")`, tu estado vive en el DOM, no en tu programa, y lo que sabes del juego depende de lo que el navegador esté dibujando. Cuando en el módulo 4 llegues a React, esta idea se llama "el DOM es una función del estado" y te va a parecer obvia. Hoy es una semilla.

### Las ideas que hay que llevarse

**Los métodos de arreglo devuelven un arreglo NUEVO.** `map`, `filter` y `reduce` no tocan lo que reciben. Si tu código depende de que algo cambie "en el lugar", estás usando el método equivocado o el diseño equivocado.

**Mutar estado compartido produce errores no deterministas.** El mismo código con la misma entrada da resultados distintos según cuántas veces se haya corrido antes. Eso es lo que hace que un bug sea imposible de reproducir, y por lo tanto imposible de arreglar con confianza.

**La inmutabilidad es una disciplina, no una palabra clave.** No la garantiza `const`. La garantiza que tu código copie en lugar de escribir encima.

### Ponte a prueba

*¿Por qué `datos.sort(...).map(...)` es una cadena peligrosa aunque se vea funcional?*

*Si `map` devuelve un arreglo nuevo, ¿por qué el ejemplo del cuarto paso sí modificó el original?*

*¿En qué caso concreto está bien mutar una variable dentro de una función?*

---

## Práctica en clase: la lógica pura del juego de memoria

El objetivo es que salgas de esta noche con un archivo funcionando: **la lógica pura del juego de memoria, sin una línea de DOM.** Esto todavía no es la Misión 03 —esa se asigna la semana entrante, cuando ya sepas manejar eventos—, es la mitad difícil de la Misión 03 hecha con anticipación y con ayuda al lado.

**Tarea 1: construir el mazo.** Escribe una función `crearMazo(cantidadDeParejas)` que devuelva un arreglo de objetos carta, cada uno con un identificador único, el símbolo de la pareja y el estado (`oculta`, `volteada`, `emparejada`). Dos cartas por pareja. Nada de DOM, nada de `document`. Pruébala imprimiendo el resultado en consola.

**Tarea 2: barajar de verdad.** Aquí está la sangre de la noche. Lo primero que se te va a ocurrir, y lo primero que devuelve cualquier asistente de IA cuando le pides "baraja este arreglo en JavaScript", es esto:

```javascript
const mezclado = mazo.sort(() => Math.random() - 0.5);  // MAL
```

No te lo voy a prohibir: **mídelo**. Corre este experimento, que cabe en diez líneas:

```javascript
// ¿Qué tan justo es el barajado? Barajamos [1,2,3,4] diez mil veces
// y contamos cuántas veces cada número cayó en la primera posición.
const conteo = { 1: 0, 2: 0, 3: 0, 4: 0 };
for (let i = 0; i < 10000; i++) {
  const barajado = [1, 2, 3, 4].sort(() => Math.random() - 0.5);
  conteo[barajado[0]]++;
}
console.log(conteo);  // muy lejos de 2500 en cada uno
```

Con un barajado justo, los cuatro conteos rondarían 2500. Con `sort` y una comparación aleatoria no lo hacen, y la razón es que `sort` asume un comparador **consistente** —que si a va antes de b, siempre va antes— y uno aleatorio rompe esa suposición, así que el resultado depende del algoritmo interno del motor. Ver el sesgo con tus propios números es infinitamente más convincente que leer la explicación.

Ahora implementa **Fisher-Yates** de verdad, recorriendo el arreglo de atrás hacia adelante e intercambiando con una posición aleatoria entre 0 y el índice actual, ambos inclusive. Y hazlo sobre una copia, no sobre el mazo original. Aquí tienes la firma y el criterio, no el cuerpo:

```javascript
// Devuelve un arreglo NUEVO barajado. No toca el original.
function barajar(lista) {
  const copia = [...lista];
  // recorre desde el final hacia el inicio;
  // en cada paso elige un índice aleatorio entre 0 y el actual, inclusive;
  // intercambia.
  return copia;
}
```

Los dos errores que vas a cometer aquí son elegir el índice aleatorio en todo el arreglo en lugar de en la parte no barajada, o dejar la última posición sin tocar. Vuelve a correr el experimento de los diez mil barajados con tu Fisher-Yates y mira los cuatro conteos cerca de 2500. Ese contraste es la lección entera.

Aprovecha el momento para hacer algo que vale más que un discurso: pídele a tu asistente de IA favorito "una función que baraje un arreglo en JavaScript" y audita lo que devuelva. Si devuelve el `sort` aleatorio, ya sabes por qué está mal, con tus propios números. Si devuelve un Fisher-Yates, revísalo buscando el error del rango. Ese es el argumento entero del curso en una escena, y por eso el `IA.md` no es burocracia: **todo código generado por un asistente se declara en `IA.md`, con qué pediste, qué te devolvió, qué estaba mal y qué corregiste. Sin declarar, la misión se califica en cero y no admite reintento. Declararlo NO baja la nota**, porque lo que se evalúa es tu capacidad de auditar. Y documentar un error real de IA con su corrección habilita la insignia 🔍 **Cazador de alucinaciones**, que da un día extra de plazo acumulable. Este ejercicio del barajado es candidato perfecto.

**Tarea 3: la máquina de estados, sin DOM.** Escribe una función `voltear(estado, idCarta)` que reciba el estado del juego y devuelva un **estado nuevo**. Tiene que decidir, sin preguntarle nada al navegador: si la carta ya está emparejada, no pasa nada; si ya hay dos cartas volteadas, no pasa nada; si es la primera, se voltea; si es la segunda, se voltea y se compara.

```javascript
const estadoInicial = {
  cartas: barajar(crearMazo(8)),
  volteadas: [],       // ids de las cartas boca arriba ahora
  intentos: 0,
  bloqueado: false     // este campo te va a salvar la vida en la sesión 8
};
```

Ese campo `bloqueado` no lo vas a usar hoy. La semana que viene es lo que evita que se volteen tres cartas a la vez. Ponlo ahora y la solución de la próxima sesión te va a parecer inevitable en lugar de mágica.

Revisa dos cosas antes de cerrar el archivo: que ninguna función escriba en el objeto que recibió, y que `voltear` sea probable desde la consola sin abrir el navegador. Si puedes llamar a `voltear` tres veces seguidas en la consola y ver el estado evolucionar, la mitad difícil de la Misión 03 ya está hecha.

Termina este archivo mientras está fresco. Presupuesta **una hora y media**, y recuerda que esa hora y media sale del presupuesto de las diez horas de la Misión 03, no se suma. En la sesión 8 se asigna esa misión formalmente y en la sesión 9 hay **Quiz 3**.

---

## Errores que probablemente vas a cometer

**Creer que `const` hace inmutable el objeto.** Es el malentendido número uno y produce un falso sentido de seguridad: declaras todo con `const`, le haces `push` a los arreglos por todos lados y crees que estás escribiendo código inmutable. Cada vez que te aparezca la duda, repítete la frase: `const` protege el nombre, no el valor. Lo que protege el valor es que tu código no lo toque.

**Usar una función flecha como método de un objeto.** Escribes `metodo: () => this.algo` y obtienes `undefined` sin ningún error visible, que es lo peor que puede pasar. Como no truena, vas a buscar el problema en otro lado durante media hora. La señal de alarma que tienes que aprender a reconocer: si dentro de una función hay un `this` y esa función está escrita con flecha directamente dentro de un objeto, algo está mal.

**Encadenar métodos sobre un arreglo que `sort` ya mutó.** Se te va a aparecer como un bug de "la tabla de puntajes se reordena sola" o "el mazo ya venía barajado". Como `sort` devuelve el mismo arreglo, la cadena `datos.sort(...).map(...)` se ve perfectamente funcional y dejó `datos` reordenado para todo el resto del programa. Copia antes de ordenar, siempre, incluso cuando creas que no hace falta.

**Olvidar el valor inicial de `reduce`.** Sin el segundo argumento, `reduce` usa el primer elemento como acumulador. Si estabas sumando propiedades de objetos, el resultado es una concatenación de texto absurda o un `NaN`, y el mensaje de error no apunta a la causa. Escríbelo siempre, incluso cuando "no hace falta"; cuesta dos caracteres y elimina una clase entera de errores.

---

## Fuentes de esta sesión

- Ecma International. *ECMAScript Language Specification*. https://262.ecma-international.org/
- MDN Web Docs. *JavaScript Guide*. https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide

La especificación de ECMAScript es la autoridad final: cuando alguien afirme que "JavaScript hace tal cosa", ahí está escrito qué hace y en qué orden. No hace falta leerla completa, pero conviene que sepas que el lenguaje tiene un documento normativo y no es un conjunto de costumbres. La guía de MDN es la lectura pedagógica de ese documento y es la que vas a usar todos los días.

---

## Antes de la sesión 8

Lee la sección "Módulo 3, sesión 8" de `GUIA-DEL-CURSO.md` y dale una ojeada a la página *Anatomy of a video game* de MDN, solo la parte introductoria donde explica qué hace un bucle de juego. Diez minutos, no más. No importa si no entiendes el código todavía; lo que necesitas es llegar habiendo visto la idea de que un juego es un ciclo que se repite y no una secuencia de instrucciones que termina.
