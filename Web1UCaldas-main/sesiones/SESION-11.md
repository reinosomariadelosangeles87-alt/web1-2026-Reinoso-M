# Sesión 11 · TypeScript como seguridad estructural

**Módulo 3** · Zona 3: La Sala de Máquinas
**Lo que sale de esta noche:** tu Misión 04 compilando en modo estricto, sin un solo `any`, y la lista escrita de los errores que el compilador te encontró en código que creías correcto
**Tu misión al terminar:** Misión 05 — Migración a TypeScript (100 XP · 8 h de trabajo autónomo). Esta noche también hay **Quiz 4** (15 XP)

---

## Por qué cerramos el módulo con tipos

El objetivo declarado es que escribas TypeScript: tipos primitivos, `interface`, `type`, uniones, genéricos básicos, inferencia y modo estricto. Es un objetivo razonable y se cumple casi solo, porque la sintaxis de TypeScript es poca y tú ya escribes JavaScript desde la sesión 7.

El objetivo real es otro, y es el que cierra el módulo entero: **que entiendas que un tipo es una especificación ejecutable.** En la sesión 3 te pedí la disciplina de *specs before code* —escribir qué tiene que hacer el programa antes de escribir el programa— y probablemente lo leíste con la paciencia con la que se lee a un profesor que pide documentación. Esta noche esa idea deja de ser una recomendación de proceso y se convierte en algo que el computador verifica. Una especificación en prosa dentro de un `README.md` no puede impedir que el código la contradiga. Un tipo sí.

Y ahí está la segunda mitad del argumento, la que le da urgencia. Trabajas con asistentes de IA todos los días, y lo que un asistente produce mejor que nadie es **código plausible**: código que se lee bien, que respeta los estilos del lenguaje, que tiene nombres razonables y que a veces está mal. Contra el código plausible, la revisión visual es un instrumento pésimo, porque el código plausible está diseñado —sin intención, pero diseñado— para pasar la revisión visual. El compilador no tiene ese punto ciego. **El compilador no se deja convencer por código que se ve bien.** Un modelo puede escribir una función que compila y que trata un `undefined` como si fuera un número; el tipo lo atrapa antes de que el programa corra.

Esta es la última sesión del módulo 3, el más largo y más difícil del curso. Y algo que conviene decir de frente: React, en el módulo 4, asume que el asincronismo ya se entiende. Si llegas al `useEffect` sin haber digerido las promesas, no vas a fallar en React: vas a fallar en el módulo 3 con dos semanas de retraso y sin saber por qué.

Un arco para ubicarte: la sesión 7 fue inmutabilidad y métodos de arreglo, la 8 fueron eventos y el bucle de juego, la 9 y la 10 fueron asincronismo y consumo de API. Cuatro sesiones donde el lenguaje te dio libertad total y ninguna red de seguridad. *Lo que hicimos hasta hoy fue aprender a movernos rápido; lo de esta noche es aprender a no caernos.*

El **Quiz 4 (15 XP)** de esta noche cubre las sesiones 9 y 10: `fetch`, promesas, `async/await`, manejo de errores con `try/catch` y por qué un `await` dentro de un `forEach` no espera nada. Presta atención a la pregunta del `fetch` que falla, porque es la puerta al tema de hoy: si tu función declara que devuelve `Pregunta[]` y en realidad puede devolver un objeto de error del servidor, la firma está mintiendo, y esta noche vas a aprender a no mentir en las firmas.

---

## Tipos como especificación ejecutable

Antes de leer lo que sigue, mira esta función y respondete: **¿tiene un error?**

```javascript
function danio(atacante, objetivo) {
  const base = atacante.fuerza * 2;
  const reducido = base - objetivo.defensa;
  return Math.max(1, reducido);
}
```

La respuesta es que no: la función está bien escrita. Ahora llámala como se llama de verdad en un juego, donde los datos vienen de un arreglo, de una API o de una selección del usuario:

```javascript
const heroe = { nombre: "Kael", fuerza: 12, defensa: 4 };
const enemigos = [{ nombre: "Gólem", fuerza: 9, defensa: 7 }];

const objetivo = enemigos.find((e) => e.nombre === "Golem");  // sin tilde, no lo encuentra
console.log(danio(heroe, objetivo));
```

`TypeError: Cannot read properties of undefined`. Ese es el caso amable, porque truena. Ahora el caso cruel, que es el que de verdad te pasa:

```javascript
const objetivo = { nombre: "Gólem", vida: 30 };  // olvidamos `defensa`
console.log(danio(heroe, objetivo));  // NaN
```

`NaN`. Y aquí está lo importante: **`NaN` no truena.** El resultado se va corriendo por tu programa:

```javascript
let vidaEnemigo = 30;
vidaEnemigo = vidaEnemigo - danio(heroe, objetivo);  // NaN
console.log(vidaEnemigo <= 0);   // false: el enemigo es inmortal
console.log("Vida:", vidaEnemigo);  // "Vida: NaN" en la pantalla del jugador
```

El enemigo es inmortal, la barra de vida dibuja una anchura de `NaN%` que el navegador ignora en silencio, y el error que vas a ver es *"la barra de vida no se mueve"*, tres pantallas y cuarenta minutos después de la línea que realmente estaba mal. ¿Cuánto tiempo te tomaría encontrar eso? Si ya te pasó en la Misión 04, ya sabes la respuesta.

Ahora la misma cosa con un tipo:

```typescript
interface Combatiente {
  nombre: string;
  fuerza: number;
  defensa: number;
  vida: number;
}

function danio(atacante: Combatiente, objetivo: Combatiente): number {
  const base = atacante.fuerza * 2;
  const reducido = base - objetivo.defensa;
  return Math.max(1, reducido);
}

const objetivo = { nombre: "Gólem", vida: 30 };
danio(heroe, objetivo);
// ^ Error: falta la propiedad 'fuerza' y falta 'defensa' en el tipo del argumento.
```

El error aparece **mientras lo escribes**, en la línea exacta, sin ejecutar nada, sin abrir el navegador y sin haber escrito una sola prueba. Esa es la sesión entera en una imagen.

### Un tipo es un contrato que alguien verifica

Cuando escribiste la especificación de la Misión 03 en el `README.md`, escribiste frases como *"la función `voltear` recibe el estado del juego y el id de una carta, y devuelve un estado nuevo"*. Es una especificación perfectamente buena, y tiene un defecto fatal: **nadie la lee excepto un humano, y ningún humano la revisa cada vez que el código cambia.** El código puede alejarse de esa frase durante semanas y la frase seguirá ahí, tranquila, mintiendo.

La analogía es el contrato de arrendamiento frente al acuerdo de palabra. Un acuerdo de palabra sobre el arriendo puede ser tan detallado como quieras; el problema no es el detalle, es que cuando hay desacuerdo no hay a quién acudir. Un contrato firmado dice lo mismo, pero hay un tercero —la ley— dispuesto a verificarlo sin que le importe cuál de las dos partes tiene mejores argumentos. **El tipo es el contrato, y el compilador es el tercero que lo verifica sin dejarse convencer.**

De ahí sale la frase que quiero que te lleves:

> *Un tipo es una especificación que el computador puede verificar. Todo lo demás que escribas sobre tu código es una intención.*

Esto cierra un círculo que empezó en la sesión 3: lo que te pedí entonces era escribir la especificación antes del código y confiar en tu disciplina para respetarla. Esta noche dejas de depender de la disciplina.

### Los tipos que necesitas esta noche, en orden de utilidad

Los **primitivos** son `string`, `number`, `boolean`, `null` y `undefined`. Un detalle que ahorra confusión si vienes de C: en TypeScript no hay `int` ni `float`, hay `number`, porque JavaScript solo tiene un tipo numérico.

Los **arreglos** se escriben `number[]` o `Combatiente[]`. Los **objetos** se describen con `interface` o con `type`. Las dos formas describen la forma de un objeto y para casi todo son intercambiables:

```typescript
// Con interface: la forma canónica para describir la forma de un objeto.
interface Enemigo {
  id: string;
  nombre: string;
  vida: number;
  fuerza: number;
  botin?: string;        // el `?` significa: puede no venir
  readonly especie: string;  // no se puede reasignar después de crear el objeto
}

// Con type: lo mismo, pero además puede nombrar cosas que no son objetos.
type Elemento = "fuego" | "hielo" | "rayo";      // una unión de literales
type Coordenada = { x: number; y: number };
type Tablero = Coordenada[][];
```

La regla práctica del curso, para que no la discutas: **`interface` para describir la forma de objetos, `type` para todo lo demás** —uniones, alias de primitivos, tipos de funciones—. La diferencia técnica real es que `interface` se puede reabrir y extender desde otro archivo y `type` no, algo que no te va a importar este semestre.

Las **uniones** son la herramienta más útil de la noche, porque modelan algo que en JavaScript solo vivía en tu cabeza:

```typescript
// El estado de una carta no es "un string": es exactamente uno de tres valores.
type EstadoCarta = "oculta" | "volteada" | "emparejada";

let estado: EstadoCarta = "volteada";
estado = "voltada";   // Error de escritura atrapado al escribirlo, no en producción
```

Detente en ese ejemplo treinta segundos, porque es el que convence a los escépticos. En la Misión 03 varios tuvieron un bug donde una carta nunca se emparejaba, y la causa fue una cadena de texto mal escrita en un solo lugar. Con una unión de literales, ese bug no existe. No se detecta antes: **no existe.**

El otro caso donde la unión gana es el del `fetch` de la semana pasada:

```typescript
// Modela honestamente que la petición puede fallar.
type Resultado =
  | { ok: true; preguntas: Pregunta[] }
  | { ok: false; mensaje: string };

function procesar(r: Resultado) {
  if (r.ok) {
    console.log(r.preguntas.length);  // aquí TypeScript SABE que existe `preguntas`
  } else {
    console.log(r.mensaje);           // y aquí sabe que existe `mensaje`
  }
}
```

Eso se llama **estrechamiento** (*narrowing*): el compilador sigue tu `if` y entiende que dentro de esa rama el tipo es más específico. Prueba a leer `r.preguntas` antes del `if` y mira el error. Ese error es un regalo y no una molestia: te está obligando a manejar el caso de fallo que el `fetch` de la sesión 10 te dejaba ignorar.

Los **genéricos**, en versión básica y sin teoría de tipos, son un tipo con un hueco:

```typescript
// Sin genérico habría que escribir una función por cada tipo de cosa.
function primero<T>(lista: T[]): T | undefined {
  return lista[0];
}

const e = primero<Enemigo>(enemigos);  // e es Enemigo | undefined
const n = primero([1, 2, 3]);          // n es number | undefined, inferido solo
```

La analogía: **el genérico es el molde de la máquina expendedora.** La máquina no sabe qué producto va a dispensar, pero sí garantiza que lo que sale por la ranura es del mismo tipo que lo que metiste en la bandeja. `T` es "el mismo tipo que entró". Y fíjate en el `| undefined` del retorno: es el compilador recordándote que un arreglo vacío no tiene primer elemento, que es precisamente el bug del `find` con el que abrimos.

Y ahora la parte que baja la resistencia: la **inferencia**. **No hay que anotar todo.** La queja número uno contra TypeScript es que parece el doble de escritura, y esa queja viene de gente que anota lo que no hace falta.

```typescript
let vida = 100;                    // inferido: number. Anotar `: number` es ruido.
const nombres = ["Kael", "Nyx"];   // inferido: string[]
const enemigo = enemigos.find((e) => e.vida > 0);  // inferido: Enemigo | undefined

// Anota SOLO en las fronteras: parámetros de funciones,
// y el retorno cuando quieras que la firma sea el contrato.
function curar(objetivo: Combatiente, cantidad: number): Combatiente {
  return { ...objetivo, vida: Math.min(objetivo.vida + cantidad, 100) };
}
```

La regla: **anota las fronteras, deja que el interior se infiera.** Los parámetros de una función son una frontera porque los llena alguien más. Una variable local que inicializas en la línea siguiente no es frontera de nada.

### Las ideas que hay que llevarse

**Un tipo es una especificación ejecutable, y es la única clase de documentación que no puede quedar desactualizada.** Si el código deja de cumplir el tipo, el código no compila. La única forma de que el tipo mienta es que alguien lo cambie a propósito, y eso aparece en el diff.

**El compilador no se deja convencer por código que se ve bien.** Es la razón por la que TypeScript es la mejor defensa disponible contra el código plausible pero incorrecto, venga de un asistente de IA o de ti mismo a las once de la noche. La revisión visual premia el código que parece correcto; el verificador de tipos premia el que lo es.

**El error más barato es el que aparece mientras escribes; el más caro es el que aparece en producción.** Entre esos dos extremos hay una escala, y cada paso hacia la derecha multiplica el costo. Un tipo mueve errores desde el extremo caro al extremo barato, y eso es literalmente todo lo que hace.

### Ponte a prueba

*Si TypeScript se borra al compilar y en el navegador solo corre JavaScript, ¿de qué sirvió?* Sirvió antes de correr. Los tipos no protegen en tiempo de ejecución, protegen en tiempo de escritura.

*¿Qué tipo tiene lo que devuelve `await fetch(...).then(r => r.json())`?* La respuesta es `any`, y es incómoda a propósito: el hueco por donde entra la mentira está justo en la frontera con el mundo exterior.

*En tu juego de memoria, ¿qué habría cambiado si `EstadoCarta` hubiera sido una unión de literales en lugar de `string`?* Relee tu propio código de hace tres semanas con ojos nuevos.

---

## Práctica en clase: la función `danio` que devolvía `NaN`

Este recorrido se hace en un proyecto nuevo, con el editor abierto. La idea es empezar con un archivo de JavaScript que **funciona** —que corre, que no truena, que se ve bien— y convertirlo en TypeScript paso a paso, dejando que el compilador vaya señalando los problemas que llevaban ahí todo el tiempo. Lo que hace valioso el ejercicio es que el compilador encuentre cosas que nadie te anunció.

Monta el proyecto:

```bash
npm create vite@latest taller-11-tipos -- --template vanilla-ts
cd taller-11-tipos
npm install
npx tsc --noEmit --watch   # el compilador vigilando en una terminal aparte
```

Deja esa última terminal visible en una esquina de la pantalla toda la clase. Ver los errores aparecer y desaparecer en tiempo real es la mitad del valor del taller.

**Primero, el módulo de combate en JavaScript, escrito para funcionar.** Escríbelo entero, sin tipos, y córrelo para comprobar que funciona. Es importante que funcione: no vas a arreglar código roto, vas a descubrir que el código bueno tenía huecos.

```javascript
// combate.js — funciona, y tiene tres bombas de tiempo
export function crearCombatiente(nombre, fuerza, defensa) {
  return { nombre, fuerza, defensa, vida: 100 };
}

export function danio(atacante, objetivo) {
  return Math.max(1, atacante.fuerza * 2 - objetivo.defensa);
}

export function atacar(atacante, objetivo) {
  const golpe = danio(atacante, objetivo);
  return { ...objetivo, vida: objetivo.vida - golpe };
}

export function buscarPorNombre(lista, nombre) {
  return lista.find((c) => c.nombre === nombre);
}
```

**Segundo, cambia la extensión a `.ts` y no toques nada más.** Un solo cambio de tres letras en el nombre del archivo y la terminal se llena de errores. Léelos. Van a ser todos de la misma familia: *"El parámetro 'atacante' tiene un tipo 'any' implícito."*

TypeScript no está inventando problemas: está diciendo *"no tengo idea de qué es esto y por lo tanto no puedo protegerte"*. Un `any` implícito no es un tipo, es una confesión de ignorancia del compilador.

**Tercero, la interfaz, y el primer descubrimiento.** Declara `Combatiente` y anota las firmas.

```typescript
export interface Combatiente {
  nombre: string;
  fuerza: number;
  defensa: number;
  vida: number;
}

export function danio(atacante: Combatiente, objetivo: Combatiente): number {
  return Math.max(1, atacante.fuerza * 2 - objetivo.defensa);
}

export function atacar(atacante: Combatiente, objetivo: Combatiente): Combatiente {
  const golpe = danio(atacante, objetivo);
  return { ...objetivo, vida: objetivo.vida - golpe };
}
```

Fíjate en la propagación tipada, que te va a servir todo el módulo 4: el `...objetivo` seguido de `vida:` produce un objeto que TypeScript verifica contra `Combatiente`. Si escribieras `vidas:` por error, el compilador te dice que sobra una propiedad y falta otra. La inmutabilidad de la sesión 7 y los tipos de esta noche se refuerzan entre sí: **copiar y verificar es más seguro que mutar y confiar.**

**Cuarto, el descubrimiento que nadie anunció: `find` puede no encontrar.** Anota `buscarPorNombre` y úsala.

```typescript
export function buscarPorNombre(lista: Combatiente[], nombre: string): Combatiente | undefined {
  return lista.find((c) => c.nombre === nombre);
}

const objetivo = buscarPorNombre(enemigos, "Golem");
atacar(heroe, objetivo);
//              ^ Error: 'Combatiente | undefined' no es asignable a 'Combatiente'.
```

Detente aquí de verdad. ¿El compilador está siendo molesto o está teniendo razón? Está teniendo razón: `find` **puede** no encontrar nada, siempre pudo, y en JavaScript nadie te lo recordaba. El error del principio —el `NaN` que hacía inmortal al enemigo— era exactamente este caso, y el tipo lo puso en la pantalla antes de ejecutar.

Hay dos formas correctas de resolverlo y cada una tiene su momento:

```typescript
// Opción A: manejarlo donde ocurre. Casi siempre es la correcta.
const objetivo = buscarPorNombre(enemigos, "Golem");
if (!objetivo) {
  console.warn("No hay enemigo con ese nombre; el ataque no ocurre.");
} else {
  atacar(heroe, objetivo);
}

// Opción B: que la función decida por ti y nunca devuelva undefined.
export function buscarOFallar(lista: Combatiente[], nombre: string): Combatiente {
  const encontrado = lista.find((c) => c.nombre === nombre);
  if (!encontrado) throw new Error(`No existe el combatiente ${nombre}`);
  return encontrado;
}
```

Lo que acaba de pasar en términos de diseño: **el tipo te forzó a decidir qué pasa cuando el enemigo no existe, y esa decisión era parte del juego que nunca habías tomado.** Los tipos no solo encuentran errores; encuentran decisiones que estabas evitando.

**Quinto, prueba el atajo del `!` y observa qué pasa.** Vas a descubrir —o tu asistente de IA te lo va a soplar— que el error desaparece con un signo de exclamación.

```typescript
atacar(heroe, objetivo!);   // MAL, y a propósito: "confía en mí, no es undefined"
```

Compila. No hay error. Ahora ejecútalo con un nombre mal escrito. Truena en tiempo de ejecución, igual que antes de TypeScript. Ese es el punto: **el operador `!` no arregla nada, apaga la alarma.** Es una promesa que le haces al compilador sin ninguna evidencia, y si te equivocas, vuelves al mundo del `NaN` silencioso. Déjalo comentado con el rótulo `// ERROR DEL TALLER — no borrar` y la razón escrita al lado.

**Sexto, uniones y estrechamiento con el registro de combate.** Modela algo que en la Misión 04 tenías con `if` sueltos y cadenas de texto:

```typescript
type Evento =
  | { tipo: "ataque"; atacante: string; objetivo: string; danio: number }
  | { tipo: "curacion"; objetivo: string; cantidad: number }
  | { tipo: "muerte"; objetivo: string };

function describir(evento: Evento): string {
  switch (evento.tipo) {
    case "ataque":
      return `${evento.atacante} golpea a ${evento.objetivo} por ${evento.danio}`;
    case "curacion":
      return `${evento.objetivo} recupera ${evento.cantidad}`;
    case "muerte":
      return `${evento.objetivo} ha caído`;
  }
}
```

Ahora la demostración que vale el tramo: agrega un cuarto caso al tipo, `{ tipo: "huida"; objetivo: string }`, y **no toques la función**. Si declaraste el retorno como `string`, el compilador se queja de que la función puede terminar sin devolver nada. **El tipo acaba de recordarte que dejaste un caso sin manejar, en un archivo que no habías abierto.** Eso es imposible de conseguir con documentación en prosa.

**Séptimo, el genérico útil, no el genérico de ejemplo.** Algo que de verdad vas a usar en la Misión 05:

```typescript
// Trae JSON y lo devuelve con el tipo que el llamador declare.
// Nota el `unknown`: json() no sabe qué trajo, y decirlo es más honesto que `any`.
async function traerJson<T>(url: string): Promise<T> {
  const respuesta = await fetch(url);
  if (!respuesta.ok) throw new Error(`El servidor respondió ${respuesta.status}`);
  const datos: unknown = await respuesta.json();
  return datos as T;   // aquí TÚ estás afirmando la forma. Ver la advertencia de abajo.
}

interface RespuestaTrivia {
  results: { question: string; correct_answer: string; incorrect_answers: string[] }[];
}

const datos = await traerJson<RespuestaTrivia>("https://opentdb.com/api.php?amount=5");
datos.results.forEach((p) => console.log(p.question));  // autocompletado real
```

Y ahora la advertencia honesta, que es una de las ideas más importantes de la noche: ese `as T` es una **afirmación**, no una verificación. TypeScript te está creyendo. Si la API cambia el nombre de un campo mañana, tu tipo dice una cosa y los datos dicen otra, y el error vuelve a ser de tiempo de ejecución. **Los tipos protegen las fronteras internas de tu programa; la frontera con el mundo exterior hay que verificarla con código.** Existen librerías de validación de esquemas para eso y no las necesitas esta noche, pero el `IA.md` de la Misión 05 es buen lugar para dejar constancia de que sabes dónde está el hueco.

### Cinco dudas que te van a salir

Si TypeScript hace el programa más lento. No: los tipos se borran al compilar. Lo que se hace más lento es el `build`, no el juego. El navegador nunca ve un solo tipo.

Por qué el editor te marca error pero `npm run dev` funciona igual. Vite **transpila** —borra los tipos y sirve el archivo— pero no verifica. La verificación la hace `tsc`, y por eso tienes esa terminal con `--noEmit --watch` abierta. Un proyecto donde nadie corre `tsc` es un proyecto con tipos decorativos.

La diferencia entre `any` y `unknown`, que casi siempre aparece porque la viste en un resultado de IA. Con `any` puedes hacer cualquier cosa sin que nadie te pregunte; con `unknown` no puedes hacer nada hasta que compruebes qué es. `unknown` es "no sé qué es esto y lo voy a averiguar"; `any` es "no sé qué es esto y no me importa".

Si intentas una `interface` con una unión, no va a compilar. Uniones con `type`.

Y si te dan curiosidad las clases, los decoradores o los tipos condicionales: anótalo como deuda y sigue. Nada de eso hace falta para la Misión 05.

---

## `tsconfig.json`, modo estricto y el costo de `any`

Esta sección no trae sintaxis nueva: trae la configuración que decide si todo lo anterior sirve para algo o es decoración.

Empecemos con una afirmación que suena exagerada y no lo es: **TypeScript sin modo estricto es un lenguaje distinto, y bastante peor.** Sin `strict`, el compilador acepta parámetros sin tipo, deja pasar `null` y `undefined` en todas partes, y te deja creer que estás protegido cuando no lo estás. La mayoría de los tutoriales que vas a encontrar en internet están escritos sin modo estricto, porque así hay menos errores que explicar. Ese es exactamente el problema.

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "noUnusedLocals": true,
    "noEmit": true
  },
  "include": ["src"]
}
```

`strict: true` es un interruptor que enciende varias verificaciones a la vez, y de esas hay dos que importan. La primera es **`noImplicitAny`**: si el compilador no puede deducir el tipo de un parámetro, es un error, y no un silencioso `any`. La segunda, y la más valiosa del lenguaje entero, es **`strictNullChecks`**: `null` y `undefined` dejan de ser valores válidos para cualquier tipo y se convierten en algo que hay que declarar y manejar.

Esa segunda es la que produce el `Combatiente | undefined` del taller. La analogía: sin `strictNullChecks`, todos los tipos vienen con una cláusula invisible al final que dice *"...o nada"*. Un `number` puede ser un número o nada. Un `Combatiente` puede ser un combatiente o nada. Y como está invisible, nadie la maneja. Encender la opción es imprimir esa cláusula en letra grande, y de repente hay que decidir qué pasa cuando no hay nada. Eso duele las primeras horas y después ahorra semanas. La industria le llama a esto *el error del billón de dólares*, y el nombre no es una exageración de mercadeo: es la estimación de lo que ha costado que los lenguajes trataran "nada" como si fuera un valor cualquiera.

`noUncheckedIndexedAccess` tiene un efecto inmediato en un juego con cuadrículas:

```typescript
const tablero: string[][] = crearTablero(8, 8);
const casilla = tablero[10][3];   // con la opción activa: error, tablero[10] puede no existir
```

Sin esa opción, indexar fuera de rango devuelve `undefined` sin que nadie diga nada, que es la fuente clásica del bug "el personaje se sale del mapa y el juego se congela". Con la opción activa, el compilador te obliga a comprobar. Es la opción más molesta de la lista y la que más te va a salvar en la Misión 06.

### Por qué `any` desactiva todo el beneficio

Esta parte es el criterio principal de calificación de la Misión 05, así que va dicha con dureza.

`any` no significa "cualquier tipo". Significa **"apaga el verificador para este valor y para todo lo que salga de él"**. Y esa segunda mitad es la que nadie ve venir: el `any` se propaga.

```typescript
const datos: any = await respuesta.json();   // un solo `any`...
const preguntas = datos.results;             // ...también es any
const primera = preguntas[0];                // any
const texto = primera.pregunta;              // any (y el campo se llama `question`)
console.log(texto.toUpperCase());            // compila perfecto. Truena en ejecución.
```

Escríbelo y comprueba que **compila sin una sola advertencia**. Cuatro líneas después del `any`, estás leyendo un campo que no existe con un nombre inventado, y el compilador que hace media hora te salvó del `NaN` ahora no dice nada. La analogía: **un `any` es un guardia de seguridad al que le dijiste que no revise a este señor ni a nadie que venga con él.** No es una excepción puntual; es un agujero con descendencia.

Y aquí es donde esto se conecta con la IA de frente. Cuando un asistente encuentra un error de tipos que no sabe resolver, la salida más frecuente es anotar `any` o meter un `as`. El código empieza a compilar, tú celebras, y lo que realmente pasó es que se desactivó la verificación justo en el lugar donde había un problema real. **El error que el compilador estaba señalando sigue ahí; lo único que se fue es el aviso.** La regla operativa: cuando un asistente resuelva un error de tipos con `any` o con `as`, eso no es una solución, es una pista de que hay algo que ninguno de los dos entendió todavía. Y eso va en el `IA.md`.

La alternativa honesta: cuando de verdad no sabes qué llega, el tipo correcto es `unknown`, porque te obliga a comprobar antes de usar.

```typescript
const datos: unknown = await respuesta.json();
if (typeof datos === "object" && datos !== null && "results" in datos) {
  // aquí ya puedes seguir estrechando, con el compilador de tu lado
}
```

Es más largo. Es la diferencia entre revisar el paquete y firmar el recibo sin abrirlo.

### Las ideas que hay que llevarse

**Sin `strict: true`, TypeScript es una decoración.** Si abres un proyecto y el `tsconfig.json` no tiene modo estricto, lo primero que hay que hacer es encenderlo y ver qué aparece. Lo que aparece es la deuda que estaba escondida.

**`any` no es un tipo, es un permiso, y se hereda.** Un `any` en la frontera de la API contamina todo lo que se deriva de él. Por eso la Misión 05 exige cero, y por eso vale 6 de los 20 XP específicos.

**Los tipos verifican tu código, no los datos del mundo.** La frontera con la red, con el `localStorage` y con lo que escribe el usuario hay que validarla escribiendo código. Saber dónde termina la protección es parte de entender la herramienta.

### Ponte a prueba

*Si `strict` va a producir cincuenta errores en tu Misión 04, ¿eso significa que el modo estricto es un problema?* Los cincuenta errores ya estaban ahí, apagados. Encender la luz no crea las cucarachas.

*¿Por qué `as` compila y `!` compila, si los dos pueden estar equivocados?* Porque TypeScript decide creerte cuando afirmas algo explícitamente. Es una herramienta de escape deliberada, y usarla sin justificación es mentirle al único revisor que no se cansa.

*¿Dónde pondrías la validación de la respuesta de la API en tu juego de trivia?* Busca la idea de una única función frontera que valide una vez, en lugar de comprobaciones dispersas.

---

## Práctica en clase: migrar tu Misión 04 a TypeScript estricto

Esta práctica es literalmente el arranque de la Misión 05. No hay proyecto nuevo, no hay repositorio nuevo: se trabaja sobre el juego de trivia que ya entregaste, en una rama aparte.

La instrucción de arranque, y respétala: **primero renombra los archivos y enciende `strict`, y ANTES de arreglar nada copia la lista completa de errores a un archivo.** Ese archivo es el entregable que de verdad importa, y si empiezas a corregir sin copiarlo, lo pierdes.

```bash
git checkout -b mision-05-typescript
# renombrar .js a .ts, agregar tsconfig con strict
npx tsc --noEmit > errores-iniciales.txt
wc -l errores-iniciales.txt
```

Mira cuántos errores te salieron. Puede que pasen de cien, y eso está bien. La frase que quiero que te lleves de este momento: *esos errores no aparecieron hoy; los entregaste la semana pasada y la misión funcionó de todas formas. Funcionó por suerte, en el camino que probaste.*

Lo que debería quedar hecho esta noche, en orden: los archivos renombrados, el `tsconfig.json` con modo estricto, la lista de errores iniciales guardada, las interfaces de la respuesta de la API escritas, y la frontera del `fetch` tipada. La eliminación de todos los `any` y la redacción comentada de la lista es para la casa.

Cuatro cosas que te van a pasar en este tramo y conviene atajar de inmediato.

Vas a estar tentado de resolver todo con `any` para que la terminal se calle, casi siempre con la ayuda de un asistente. Antes de hacerlo, busca en tu propio código dónde estaba el error real que el `any` tapa, porque casi siempre hay uno. Esa búsqueda es lo más productivo que puedes hacer esta noche.

Vas a pelear con el resultado de `respuesta.json()`, que es `any` por diseño en la definición de la plataforma. Ahí es donde debes escribir tu `interface` de la respuesta de la API, y ahí es donde ganas 5 de los 20 XP específicos. Abre la respuesta cruda de la API en el navegador y léela campo por campo antes de escribir la interfaz. Adivinar los nombres de los campos es cómo se producen las interfaces que compilan y mienten.

Vas a ver `document.getElementById("puntaje").textContent = "0"` marcado en rojo, y te vas a frustrar. Es un buen error: `getElementById` devuelve `HTMLElement | null` porque el elemento puede no existir, y en un juego que arma HTML dinámicamente eso pasa de verdad. La solución razonable es una función auxiliar que busque y falle con un mensaje claro, no un `!` en cada línea.

Y vas a querer reescribir el juego entero desde cero "ahora que sabes hacerlo bien". No lo hagas. **El valor de esta misión está en migrar el código existente, porque solo así los errores que aparecen son errores que tú cometiste.** Un juego reescrito desde cero con tipos no te enseña nada sobre lo que estaba mal en el anterior.

---

## Tu misión de la semana: Migración a TypeScript (100 XP)

Tomas tu propia Misión 04 y la migras a TypeScript en modo estricto, sin usar `any`.

### Por qué el entregable real no son los tipos

Si no lo digo explícitamente vas a creer que la misión es "poner tipos". No lo es. **El entregable real es la lista de errores que el compilador te encontró en código que creías correcto.** Un archivo `ERRORES-ENCONTRADOS.md` con la lista inicial, y para cada error que resultó ser un problema de verdad, tres cosas: qué decía el compilador, por qué el código estaba mal, y en qué situación habría fallado con un usuario real.

Y algo que conviene saber de antemano para que lo busques: **casi siempre aparecen dos o tres casos que habrían fallado en producción.** El `find` que puede no encontrar. El campo de la API que a veces no viene. El `parseInt` de un valor que podía ser `null`. La casilla del arreglo fuera de rango. No son errores hipotéticos de ejercicio: son tus propios bugs, en tu propio juego, en código que revisaste y entregaste convencido. Esa es la razón por la que la misión está diseñada así: **verlos en tu propio código convence más que cualquier argumento externo.** Se puede repetir veinte veces que los tipos sirven y no pasa nada; si descubres que tu juego de trivia se rompía cuando la API devolvía menos preguntas de las pedidas, no necesitas que nadie te lo repita.

### Cómo se califica

| Criterio | XP |
|---|---|
| Funciona: el juego migrado sigue funcionando igual o mejor | 35 |
| Calidad del código: nombres claros, sin repetición ni código muerto | 25 |
| Específico del módulo (ver abajo) | 20 |
| Proceso en Git: commits pequeños y descriptivos, PR completo | 10 |
| Auditoría de IA: `IA.md` real y reflexivo | 10 |

Los 20 XP específicos de esta misión: cero `any` explícito o implícito (6); interfaces declaradas para las respuestas de la API (5); compila limpio en modo estricto (5); y la lista de errores encontrados es real y está comentada (4).

**Misión secundaria opcional (+25 XP):** pruebas unitarias con Vitest sobre la lógica de puntaje. No son puntos fáciles: son el complemento natural del argumento de la noche. Un tipo verifica la forma de los datos; una prueba verifica el comportamiento. Que el puntaje sea un `number` no garantiza que sea el número correcto. Dos o tres pruebas sobre la función que calcula puntos —respuesta correcta, respuesta incorrecta, racha— alcanzan.

Sobre la IA, en esta misión hay una tentación muy específica: **todo código generado por un asistente se declara en `IA.md`, con qué pediste, qué te devolvió, qué estaba mal y qué corregiste. Sin declarar, la misión se califica en cero y no admite reintento. Declararlo NO baja la nota**, porque lo que se evalúa es tu capacidad de auditar. Y aquí la auditoría tiene un blanco obvio: cada vez que un asistente resuelva un error de tipos con `any` o con `as`, eso es material para el `IA.md` y candidato directo a la insignia 🔍 **Cazador de alucinaciones**, que da un día extra de plazo acumulable.

**Plazo:** antes de la medianoche del día anterior a la sesión 12. Presupuesto: **8 horas** repartidas en 1 de configurar y renombrar, 5 de migrar y eliminar los `any`, y 2 de documentar los errores encontrados.

---

## Antes de cerrar el módulo: ¿qué quedó flojo?

Esta pregunta va en serio y vale la pena que te la respondas antes de la sesión 12, porque nadie más lo va a hacer por ti. No es "¿tengo alguna duda?", que es una pregunta a la que nunca nadie responde. Es concreta: ¿sigues sin tener claro qué hace exactamente un `await`? ¿Sabrías explicar por qué un `fetch` que devuelve 404 no entra en el `catch`? ¿Entiendes para qué sirve la cola de microtareas?

Si alguna respuesta es "no del todo", escríbelo y pide asesoría antes de la sesión 12. La razón es esta: **React, en el módulo 4, asume que el asincronismo ya se entiende.** El `useEffect` de la sesión 13 es una promesa esperando dentro de un componente que se puede desmontar antes de que la respuesta llegue. Quien llegue ahí sin dominar el módulo 3 no va a tener problemas con React: va a tener problemas del módulo 3 disfrazados de React, dos semanas más tarde, y ese disfraz es lo que hace que sean tan difíciles de diagnosticar.

El módulo que se llamaba **Domador del asincronismo, 700 XP** acaba esta noche. Si estás en ese nivel, lo estás porque te lo ganaste en la parte más difícil del semestre.

---

## Errores que probablemente vas a cometer

**Anotar absolutamente todo, incluso lo que se infiere solo.** Vas a escribir `const vida: number = 100` y `const nombres: string[] = ["Kael"]`, y a los diez minutos vas a concluir que TypeScript es escribir el doble para nada. El problema no es el lenguaje, es la costumbre traída de otros lenguajes donde el tipo va siempre a la izquierda. Anota las fronteras —los parámetros de las funciones y, cuando importa como contrato, el retorno— y deja que el interior lo deduzca el compilador. Un buen indicador: si al borrar una anotación no aparece ningún error nuevo, esa anotación era ruido.

**Usar `any` como analgésico para que la terminal se calle.** Es el error más costoso del tema y el más fácil de cometer a las once de la noche, sobre todo con un asistente ofreciéndolo. El daño tiene dos capas: la inmediata, que es perder la verificación en ese valor y en todo lo que se derive de él, y la más grave, que es haber apagado la única alarma que estaba señalando un problema real. El reflejo correcto: cuando aparezca la tentación del `any`, la pregunta no es "¿cómo silencio esto?" sino "¿qué es lo que este valor puede ser realmente?". Casi siempre la respuesta es una unión con `undefined`, y escribir esa unión es la corrección de verdad.

**Confundir el `as` y el `!` con soluciones.** Es el mismo error con mejor disfraz, porque `as` y `!` se ven técnicos y deliberados mientras `any` se ve perezoso. Pero las tres cosas hacen lo mismo desde el punto de vista de la garantía: le piden al compilador que crea una afirmación sin evidencia. Un `as RespuestaTrivia` sobre un `json()` no verifica un solo campo; solo cambia lo que el editor te va a autocompletar. Lee `as` como "yo me hago responsable de que esto sea así", y pregúntate si de verdad puedes hacerte responsable. En la frontera con una API que no controlas, la respuesta honesta es que no.

**Creer que los tipos protegen en tiempo de ejecución.** Vas a escribir una `interface` perfecta para la respuesta de la API, la API va a cambiar un nombre de campo o a devolver un arreglo vacío, el juego va a tronar, y tu reacción va a ser "pero yo puse los tipos". Es la confusión conceptual más importante de la noche. La imagen que ayuda: el tipo es el plano del edificio y el compilador es el revisor que comprueba que lo construido coincide con el plano; ninguno de los dos puede impedir que llegue un camión que el plano no contemplaba. Lo que protege en la frontera es código que valide, y saber dónde está esa frontera —el `fetch`, el `localStorage`, el `input` del usuario, la `URL`— es parte del oficio.

---

## Fuentes de esta sesión

- Microsoft. *The TypeScript Handbook*. https://www.typescriptlang.org/docs/handbook/intro.html
- Prather, J. et al. (2024). *The widening gap: The benefits and harms of generative AI for novice programmers*. ICER '24, 469-486. https://doi.org/10.1145/3632620.3671116

El *Handbook* de TypeScript es corto y está escrito para gente que ya sabe JavaScript, que es exactamente tu situación. Ve a las secciones de tipos básicos, objetos y estrechamiento; el resto es referencia que vas a consultar cuando la necesites. La documentación oficial es más clara que la mayoría de los tutoriales que vas a encontrar, precisamente porque no evita el modo estricto para simplificar los ejemplos.

El artículo de Prather y colegas sostiene el argumento pedagógico de la noche: la brecha entre estudiantes se ensancha con los asistentes de IA porque quienes ya entienden el código los usan como acelerador y quienes no lo entienden los usan como sustituto, y el segundo grupo produce código que funciona lo suficiente para pasar la entrega y no lo suficiente para sostenerse. La razón por la que te lo señalo: el modo estricto de TypeScript es una de las pocas herramientas que mueve a alguien del segundo grupo al primero sin que un profesor esté al lado, porque el compilador no se conforma con código que parece correcto.

---

## Antes de la sesión 12

Lee la sección "Módulo 4, sesión 12" de `GUIA-DEL-CURSO.md`, y la página *Thinking in React* de `react.dev/learn`, completa. Quince minutos. No importa si no entiendes el código de los ejemplos; lo que necesitas es llegar habiendo visto una vez la idea de que se empieza dibujando el árbol de componentes y decidiendo dónde vive cada dato, antes de escribir una línea.

Y llega con una respuesta a una sola pregunta: en tu juego de trivia, ¿cuántos pedazos de estado distintos hay y quién necesita ver cada uno?
