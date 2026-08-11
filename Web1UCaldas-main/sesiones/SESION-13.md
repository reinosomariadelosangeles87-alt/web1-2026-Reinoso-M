# Sesión 13 · Hooks y ciclo de vida

**Módulo 4** · Zona 4: La Fortaleza de Componentes
**Lo que sale de esta noche:** un `useEffect` con limpieza funcionando, y una fuga de memoria vista con tus propios ojos en las herramientas del navegador
**Lo que se evalúa hoy:** Auditoría 3 cruzada (25 XP)
**Tu misión en curso:** Misión 06 — Mazmorra por turnos en React (100 XP · 10 h de trabajo autónomo), que se entrega en la sesión 14

---

## Por qué esta noche son dos temas y no seis

El temario dice `useState`, `useEffect`, `useRef`, `useReducer`, `useMemo` y `useCallback`. Suena a seis cosas. En realidad son dos las que deciden si tu mazmorra funciona o te arruina el fin de semana: **el arreglo de dependencias** y **la función de limpieza**.

La razón es empírica. Un `useEffect` con dependencias mal declaradas produce bugs que no se pueden reproducir a voluntad, del tipo "a veces el enemigo se mueve dos veces". Un `useEffect` sin limpieza produce fugas que no se notan en cinco minutos de prueba y sí se notan en veinte de partida. Ninguno de los dos se detecta leyendo el código con calma; los dos se previenen entendiendo la regla.

Y hay un tercer objetivo, más de actitud, que se juega al final: que salgas con **miedo sano a optimizar antes de medir**. Vas a leer en internet que hay que envolver todo en `useMemo`. Es falso y hace daño.

Antes de seguir, una pregunta que probablemente ya te tocó: **¿intentaste hacer que el enemigo de tu mazmorra se mueva solo, con un temporizador, y algo salió raro?** Si el enemigo se movía cada vez más rápido, o si seguía moviéndose después de morir, esos son exactamente los dos bugs de esta noche y ya los tienes en tu propio código.

---

## `useState` a fondo: el estado del render es una instantánea

Antes de leer la explicación, respondete esto mirando el código:

```javascript
// El jugador recibe un golpe crítico: tres puntos de daño, uno por uno.
function recibirGolpe() {
  setVida(vida - 1);
  setVida(vida - 1);
  setVida(vida - 1);
}
// Si la vida era 20, ¿en cuánto queda?
```

La respuesta que da todo el mundo es 17. La respuesta real es 19. Vale la pena quedarse incómodo un momento antes de seguir leyendo, porque esa incomodidad es la puerta al concepto.

La explicación es que `vida` no es una variable que puedas leer y reescribir. Es una **instantánea** del valor que tenía el estado en el render actual, y dentro de esa función vale 20 en las tres líneas. Las tres llamadas dicen lo mismo: "en el próximo render, la vida es 19". React no aplica los cambios uno por uno en el momento; los **agrupa por lote** y vuelve a renderizar una sola vez al final del manejador del evento, con el último valor que le dijeron.

La forma correcta es esta:

```javascript
// Actualización con función: React te pasa el valor MÁS RECIENTE
// de la cola, no la instantánea de tu render.
function recibirGolpe() {
  setVida((v) => v - 1);
  setVida((v) => v - 1);
  setVida((v) => v - 1);  // ahora sí: 17
}
```

Piénsalo como notas que dejas pegadas. `setVida(19)` es dejar una nota que dice "la vida queda en 19". `setVida(v => v - 1)` es dejar una nota que dice "réstale uno a lo que haya". Si dejas tres notas del primer tipo, todas dicen lo mismo y solo importa una. Si dejas tres del segundo tipo, se aplican en orden y se acumulan.

La regla práctica: **cuando el valor nuevo depende del anterior, usa la forma de función.** Contador de puntaje, vida, munición, turno. En un juego, casi todo es ese caso.

Y el corolario que te va a ahorrar horas: **el estado no cambia inmediatamente después de llamar al setter.** Si escribes `setVida(19); console.log(vida);` vas a ver el valor viejo y vas a creer que el setter no funcionó. No falló nada: estás leyendo la instantánea del render en el que estás. El valor nuevo existe en el render siguiente.

---

## `useEffect`: el hook que hay que respetar

Primero para qué sirve, porque el mal uso viene de no tener claro el propósito. Un componente puro calcula y devuelve una descripción. Pero hay cosas que hay que hacer y que no son eso: suscribirse a un evento del navegador, arrancar un temporizador, abrir una conexión, medir un elemento ya pintado, pedir datos a un servidor. Todo eso son **efectos**: acciones que tocan el mundo de fuera de React. `useEffect` es el lugar donde van, y es el único lugar.

La anatomía tiene tres partes y conviene nombrarlas todas:

```javascript
useEffect(() => {
  // 1. El efecto: corre DESPUÉS de que React pintó la pantalla.
  const id = setInterval(() => moverEnemigos(), 1000);

  // 2. La limpieza: corre antes del siguiente efecto y al desmontar.
  return () => clearInterval(id);

  // 3. Las dependencias: de qué valores depende este efecto.
}, [velocidad]);
```

**Cuándo corre.** Después del render y después de que el navegador pintó, no durante. Esto importa: si dentro del efecto llamas a un setter, provocas otro render. A veces es legítimo, y es la fuente del ciclo infinito cuando las dependencias están mal.

### El arreglo de dependencias

Aquí hay que ir despacio, porque es el tema de la noche. El arreglo le dice a React de qué valores depende el efecto para saber cuándo volver a ejecutarlo. Después de cada render, React compara cada elemento del arreglo con el del render anterior; si alguno cambió, limpia el efecto viejo y ejecuta el nuevo.

Sin arreglo, el efecto corre **después de cada render**. Casi siempre es un error, y si dentro hay un setter es un ciclo infinito garantizado. Con arreglo vacío `[]`, el efecto corre **una sola vez**, al montar el componente, y se limpia al desmontar: es lo correcto para suscribirse a un evento global o abrir una conexión que no depende de nada. Con valores dentro `[a, b]`, el efecto corre al montar y **cada vez que `a` o `b` cambien**.

Y la regla, que es regla y no recomendación: **el arreglo tiene que contener todos los valores reactivos que el efecto usa por dentro.** Todo estado, toda prop, toda variable calculada en el cuerpo del componente. Si mientes en el arreglo —si omites algo para que el efecto corra menos veces— el efecto va a trabajar con valores viejos, y ese es el bug del enemigo que se mueve según un estado que ya no existe.

Textualmente, porque esto cambia la actitud: **el arreglo de dependencias no es una palanca para controlar cuándo corre el efecto. Es una declaración de qué usa el efecto.** Si el efecto corre demasiadas veces, el problema no es el arreglo: es que el efecto está mal diseñado, o que ese código no debería ser un efecto.

### La función de limpieza

El otro tema de la noche. Todo lo que abras hay que cerrarlo: temporizadores, escuchadores de eventos, conexiones, peticiones en vuelo. Si no lo cierras, sigue vivo después de que el componente desapareció, y eso es una fuga de memoria.

La limpieza es colgar el teléfono. Si abres cinco llamadas y no cuelgas ninguna, no es que la sexta no funcione: es que las cinco anteriores siguen hablando al mismo tiempo. En un juego eso se ve como el enemigo que se mueve el doble de rápido. No es que un intervalo vaya más rápido; es que hay dos intervalos.

Un detalle que evita una hora de persecución de fantasmas: en modo de desarrollo, React monta, desmonta y vuelve a montar cada componente a propósito, precisamente para que las limpiezas faltantes se vuelvan visibles de inmediato. Si un efecto sin limpieza produce el doble de algo desde el primer segundo, no es un bug de React: es React delatándote.

### Las ideas que hay que llevarse

**El estado del render es una instantánea, no una variable.** Dentro de una función, un valor de estado no cambia nunca. Cambia entre renders. Todo el desconcierto con `useState` sale de no haber aceptado esta frase.

**El arreglo de dependencias es una declaración honesta de lo que el efecto usa.** Mentir ahí no hace que el código funcione mejor; hace que falle de forma intermitente, que es peor que fallar siempre.

**Todo lo que se abre en un efecto se cierra en la limpieza.** Sin excepciones. Y si no sabes qué cerrar, es señal de que no entendiste qué abriste.

**Muchos efectos no deberían ser efectos.** Si estás usando `useEffect` para calcular un valor derivado de otro estado, o para reaccionar a un clic del usuario, ese código va en el cuerpo del componente o en el manejador del evento. Un efecto es para sincronizar React con algo de fuera de React, no para orquestar lógica interna.

### Ponte a prueba

*Si el estado es una instantánea, ¿cómo hace `setVida(v => v - 1)` para saber el valor real?* Porque la función se ejecuta después, cuando React procesa la cola, no cuando tú la escribes.

*Tienes un efecto con `[]` que hace `fetch` y guarda el resultado con un setter. ¿Es correcto?* Casi. Falta cancelar la petición en la limpieza, porque si el componente desaparece mientras la respuesta viaja, el setter se llama sobre algo que ya no existe. Ese hilo lo retomamos en la sesión 15.

*¿Por qué un efecto corre infinitamente si dentro pones `setContador(contador + 1)` sin dependencias?* Reconstruye el ciclo en voz alta: render, efecto, cambio de estado, render, efecto. Cuando lo dices tú, no se te olvida.

---

## Práctica en clase: el enemigo que no deja de moverse

Este recorrido se hace sobre el proyecto de la Misión 06 que ya tienes. La estructura es deliberada: **primero provocas la fuga, después la ves en las herramientas del navegador, y solo entonces la arreglas.** Está escrito paso por paso para que puedas repetirlo por tu cuenta.

**Primero, el enemigo con intervalo escrito mal a propósito.** Crea un componente `Enemigo` que se mueve cada segundo.

```tsx
function Enemigo({ id, velocidadMs }: { id: string; velocidadMs: number }) {
  const [posicion, setPosicion] = useState({ x: 0, y: 0 });

  // VERSIÓN ROTA: sin limpieza y sin dependencias correctas.
  useEffect(() => {
    setInterval(() => {
      setPosicion((p) => ({ x: p.x + 1, y: p.y }));
    }, velocidadMs);
  });  // ← sin arreglo: corre después de CADA render

  return <span style={{ left: posicion.x * 32 }}>👹</span>;
}
```

Ejecútalo y observa qué pasa. En segundos el enemigo se acelera de forma absurda. Antes de leer la explicación, intenta decir por qué. La razón es que cada render crea un intervalo nuevo y ningún intervalo se cancela, así que hay diez, cien, mil intervalos empujando la misma posición.

**Segundo, hazlo visible.** Abre las herramientas de desarrollo y ve a la pestaña **Rendimiento**. Graba diez segundos y mira la densidad de tareas de JavaScript creciendo. Después ve a la pestaña **Memoria**, toma una instantánea del montón, deja correr treinta segundos, toma otra y compáralas. El número crece sin que nadie haya hecho nada.

Si tu máquina no da para el perfilador de memoria, hay una versión más brutal y que convence igual: un contador global fuera del componente.

```javascript
// Fuera del componente, solo para la demostración.
let intervalosCreados = 0;

useEffect(() => {
  intervalosCreados += 1;
  console.log("Intervalos vivos:", intervalosCreados);
  setInterval(/* ... */);
});
```

La consola escupiendo "Intervalos vivos: 47" en cinco segundos convence a cualquiera.

**Tercero, agrega `[]` y descubre que arreglaste la mitad.** El enemigo deja de acelerarse. Celébralo un segundo y después rómpelo: agrega en el componente padre un control para cambiar `velocidadMs` de 1000 a 300. Cámbialo y observa que **el enemigo sigue moviéndose a un movimiento por segundo**. El efecto se quedó con el valor viejo de la prop porque prometió no depender de nada.

Este es el momento central de la noche: **el arreglo vacío no significa "corre una vez"; significa "este efecto no usa ningún valor que cambie". Si eso es mentira, el efecto trabaja con datos del pasado.**

**Cuarto, la versión correcta, con las tres partes en su lugar.**

```tsx
useEffect(() => {
  const id = setInterval(() => {
    setPosicion((p) => ({ x: p.x + 1, y: p.y }));
  }, velocidadMs);

  // Se cancela el intervalo viejo antes de crear el nuevo,
  // y también cuando el enemigo muere y desaparece del árbol.
  return () => clearInterval(id);
}, [velocidadMs]);  // declara honestamente lo que usa
```

Repite la prueba: cambia la velocidad y el enemigo cambia de ritmo, sin duplicarse. Después saca al enemigo de la lista con un botón y comprueba que el movimiento se detiene de verdad.

Hay un detalle que se pasa por alto y conviene notar: `setPosicion` no está en el arreglo y no hace falta, porque React garantiza que la función del setter es estable entre renders. Y `posicion` tampoco está, precisamente porque usaste la forma de función. Ese patrón resuelve el noventa por ciento de los efectos con temporizador en un juego.

**Quinto, el escuchador de teclado con limpieza.** Es el movimiento del jugador con las flechas, y el otro efecto que vas a necesitar en la Misión 06.

```tsx
useEffect(() => {
  function alPresionar(evento: KeyboardEvent) {
    if (evento.key === "ArrowUp") mover(0, -1);
    // ...
  }
  window.addEventListener("keydown", alPresionar);
  return () => window.removeEventListener("keydown", alPresionar);
}, [mover]);
```

Prueba esto y observa qué pasa: quita el `removeEventListener`, navega a otra vista y vuelve. Las teclas mueven al personaje dos casillas por pulsación, porque hay dos escuchadores. Es el bug que más vas a sufrir en la misión.

**Sexto, `useRef` para lo que no debe redibujar.** Este es el hook que en juegos es imprescindible y que casi nadie usa a tiempo. El problema es concreto: quieres guardar el instante del último fotograma para calcular el delta de tiempo —lo mismo que hiciste en la sesión 8 con `requestAnimationFrame`—, pero ese valor cambia sesenta veces por segundo y no debe provocar ni un solo render.

```tsx
const ultimoFotograma = useRef(performance.now());
const idAnimacion = useRef<number | null>(null);

useEffect(() => {
  function bucle(ahora: number) {
    const delta = ahora - ultimoFotograma.current;
    ultimoFotograma.current = ahora;   // mutar .current NO renderiza
    actualizarFisica(delta);
    idAnimacion.current = requestAnimationFrame(bucle);
  }
  idAnimacion.current = requestAnimationFrame(bucle);

  return () => {
    if (idAnimacion.current !== null) cancelAnimationFrame(idAnimacion.current);
  };
}, []);
```

`useState` es un tablero de anuncios: cuando escribes ahí, todo el mundo se entera y la pantalla se rehace. `useRef` es una libreta en tu bolsillo: puedes escribir lo que quieras y nadie se entera. Sirve para lo que necesitas recordar entre renders pero que no se dibuja: identificadores de temporizadores, referencias a elementos del DOM, el instante anterior, si ya se disparó algo una vez.

La regla de decisión, y vale escribirla en alguna parte: **si el dato se dibuja, es estado. Si no se dibuja, es referencia.** Si guardas la vida en un `ref`, la barra de vida no se va a actualizar y vas a perder media hora buscando por qué.

Cuatro dudas que aparecen siempre. No se pueden llamar hooks dentro de un `if` o de un ciclo porque React identifica cada hook por el **orden** en que se llamó dentro del componente, no por su nombre; si el orden cambia entre renders, React asocia tu estado al hook equivocado, y de ahí viene la regla de que los hooks van siempre arriba y siempre en el mismo orden. No se puede poner una función `async` directa en `useEffect`, porque una función `async` devuelve una promesa y React espera la función de limpieza o nada; se resuelve declarando una función interna y llamándola, y en la sesión 15 lo vas a usar en serio. Meter el estado del juego completo en un `ref` para "no renderizar tanto" es una trampa atractiva y no funciona: si no renderiza, no se ve. Y si el perfilador de memoria se comporta raro en tu máquina, usa el contador global; enseña lo mismo.

**Lo mínimo que debería quedarte hecho esta noche:** el escuchador de teclado con limpieza, el movimiento del jugador funcionando, y al menos un enemigo con temporizador y limpieza correcta. Si vas adelantado, pasa el estado del juego a `useReducer`.

Y hay una auditoría propia que conviene hacer ya: **revisa cada `useEffect` que hayas escrito y responde por escrito dos preguntas.** ¿Qué valores usa este efecto por dentro, y están todos en el arreglo? ¿Qué abre este efecto, y dónde se cierra? Si la respuesta a la segunda es "nada", tiene que estar justificado, no asumido. Esas respuestas van al `README.md` de la práctica y son parte de los 4 XP específicos de efectos.

---

## `useReducer`: cuando el estado tiene reglas

Empieza por el síntoma, no por la solución. Cuando tu `App` tiene ocho `useState` y cada acción del juego toca cinco de ellos a la vez, la lógica se dispersa en los manejadores de eventos y se vuelve imposible razonar sobre qué transiciones son válidas. Atacar a un enemigo, por ejemplo, cambia la vida del enemigo, la vida del jugador si contraataca, el turno, el registro de combate y posiblemente el inventario.

`useReducer` invierte la estructura: en lugar de que cada manejador sepa cómo modificar el estado, cada manejador **describe qué pasó** y una sola función decide cómo cambia el estado.

```tsx
type Accion =
  | { tipo: "ATACAR"; idEnemigo: string }
  | { tipo: "MOVER"; dx: number; dy: number }
  | { tipo: "USAR_OBJETO"; idObjeto: string };

function reductor(estado: EstadoJuego, accion: Accion): EstadoJuego {
  switch (accion.tipo) {
    case "ATACAR": {
      // Toda la regla de combate vive AQUÍ, en un solo lugar,
      // y es una función pura: mismo estado + misma acción = mismo resultado.
      const enemigos = estado.enemigos.map((e) =>
        e.id === accion.idEnemigo ? { ...e, vida: e.vida - estado.ataque } : e
      );
      return { ...estado, enemigos, turno: "enemigo" };
    }
    default:
      return estado;
  }
}

const [estado, despachar] = useReducer(reductor, estadoInicial);
// En el componente: despachar({ tipo: "ATACAR", idEnemigo: "orco-1" })
```

`useState` es como llevar la contabilidad editando el saldo. `useReducer` es como llevarla registrando transacciones y calculando el saldo. Lo segundo es más ceremonioso y es lo que permite auditar, deshacer y probar.

Dos ventajas concretas que te van a importar. Una: el reductor es una función pura sin nada de React, así que se puede probar con Vitest sin montar ningún componente, exactamente como la misión secundaria del módulo 3. Dos: si mañana quieres un botón de "deshacer turno", con un reductor eso es guardar el estado anterior en una pila, y con ocho `useState` es una pesadilla.

Y la advertencia honesta: `useReducer` no es "mejor" que `useState`. Para la vida del jugador, `useState` es lo correcto. La señal de que hay que cambiar es cuando varias piezas de estado cambian **juntas y por las mismas razones**.

---

## `useMemo` y `useCallback`: no optimices antes de medir

Aquí hay que ser severo, porque el consenso de internet en este tema es malo.

Los dos hooks hacen lo mismo conceptualmente: recordar un resultado entre renders para no volver a calcularlo. `useMemo` recuerda el valor que devuelve una función; `useCallback` recuerda la función misma. Ambos reciben un arreglo de dependencias con la misma semántica que `useEffect`.

```tsx
// Solo se recalcula el camino cuando cambia el mapa o el destino.
const camino = useMemo(
  () => calcularRutaMasCorta(mapa, origen, destino),
  [mapa, origen, destino]
);
```

Ahora la parte importante. **Envolver algo en `useMemo` no es gratis.** Cuesta memoria para guardar el valor, cuesta comparar el arreglo de dependencias en cada render, y cuesta legibilidad. Para un cálculo trivial —una suma, un filtro sobre diez elementos— el envoltorio cuesta más que el cálculo. Estás pagando por no hacer un trabajo que era más barato que el pago.

La regla del curso, que vuelve en la sesión 18 con Lighthouse en la mano: **primero mides, después optimizas.** Medir significa abrir el perfilador de React, grabar la interacción que se siente lenta y ver qué componente tarda cuántos milisegundos. Si no tienes un número antes y un número después, no optimizaste: cambiaste el código y te sentiste productivo.

Los casos donde sí se justifica son dos y conviene reconocerlos. Uno, un cálculo genuinamente caro sobre datos grandes, como una búsqueda de camino en una cuadrícula grande recalculada en cada render. Dos, un valor que se pasa como dependencia de un `useEffect`, donde una referencia nueva en cada render hace que el efecto corra siempre. Ese segundo caso es la razón más frecuente y legítima para `useCallback`, y conecta directo con el taller: si le pasas al efecto una función recreada en cada render, la declaras en las dependencias y el efecto se reinicia sin parar.

La frase para cuando alguien te diga que hay que memorizar todo: **el código legible es la optimización por defecto; lo demás se justifica con un número.**

### Las ideas que hay que llevarse

**`useReducer` no es mejor que `useState`; es para cuando varias piezas cambian juntas y por las mismas razones.** La señal es la lógica dispersa en cinco manejadores, no la cantidad de líneas.

**Un reductor es una función pura y por eso se puede probar sin montar nada.** Esa es su ventaja menos vistosa y la más valiosa.

**Optimizar antes de medir es perder el tiempo, y a veces empeorarlo.** Las comparaciones de dependencias también cuestan.

### Ponte a prueba

*Tienes ocho `useState` en `App` y cada acción del juego toca cinco. ¿Qué cambia si pasas a `useReducer`?* Que la regla de qué transiciones son válidas queda en un solo lugar y se puede probar aparte.

*Envolviste una suma de dos números en `useMemo`. ¿Ganaste algo?* No: perdiste memoria, una comparación por render y legibilidad, a cambio de no hacer una suma.

*Tu efecto se reinicia en cada render y las dependencias parecen correctas. ¿Qué sospechas?* Que una de las dependencias es una función u objeto recreado en cada render. Ese es el caso legítimo de `useCallback`.

---

## Tu evaluación de esta noche: Auditoría 3 cruzada (25 XP)

Esta noche haces la tercera auditoría cruzada del semestre, y el tema es hooks. Abres la rama de la mazmorra del compañero que te toque y escribes tu revisión como **comentario en el Pull Request**, no como archivo aparte. Se piden al menos dos comentarios sustantivos.

Lo que tienes que responder: por cada `useEffect` del código que revisas, si el arreglo de dependencias declara todo lo que el efecto usa, y si lo que se abre se cierra. Si hay algún dato guardado en estado que no se dibuja y debería ser una referencia, o al revés. Si hay estado duplicado que podría derivarse. Y una cosa que la otra persona hizo bien, dicha concretamente y no como cortesía.

Los 25 XP se reparten así: identificar al menos un problema real y explicar por qué es un problema (10), proponer una corrección concreta y no genérica (8), el tono —crítica al código y nunca a la persona (4), y reconocer algo bien hecho de forma específica (3).

Ojo con lo que se califica aquí, porque suele malentenderse: **se evalúa la calidad de TU revisión, no la del código que revisas.** Un comentario que dice "esto está mal" no vale nada; uno que dice "este efecto usa `velocidad` pero el arreglo está vacío, así que va a quedarse con el valor inicial" vale un puesto de trabajo. Revisar código es una habilidad profesional y se entrena; en la sesión 16 volvemos sobre eso. Y dos reconocimientos por ayudar de verdad habilitan la insignia 🤝 **Buen colega**, que descarta tu peor quiz.

---

## Errores que probablemente vas a cometer

**Usar el arreglo de dependencias como interruptor en lugar de como declaración.** Es el error madre del que salen casi todos los demás. El efecto corre más veces de las que quieres, entonces vacías el arreglo hasta que "deja de molestar". El efecto deja de correr, sí, y también deja de ver los valores nuevos. El resultado es el bug más frustrante del curso, porque el código se ve razonable, no lanza ningún error y simplemente hace lo correcto con datos equivocados. El cambio de fondo es de modelo mental: el arreglo describe, no controla. Si el efecto corre demasiado, la pregunta es por qué depende de tantas cosas, o si eso debía ser un efecto.

**Olvidar la función de limpieza y no notarlo porque "funciona".** Un intervalo sin cancelar, un escuchador sin quitar, una petición sin abortar. En una prueba de dos minutos nada de eso se nota: la memoria crece despacio y los duplicados tardan en acumularse. Se nota en una partida larga, en el celular de otra persona, o el día de la sustentación con el proyector encendido. Lo que ayuda es la costumbre mecánica: en el momento de escribir la línea que abre algo, escribe inmediatamente el `return` que lo cierra, antes de seguir con la lógica. Y ayuda haberlo visto una vez en el perfilador de memoria, que es exactamente para eso que existe la práctica de esta noche.

**Leer el estado justo después de llamar al setter.** Vas a escribir `setPuntaje(puntaje + 10)` y en la línea siguiente un `console.log(puntaje)` que muestra el valor viejo, y vas a concluir que el setter falló. De ahí salen las soluciones inventadas: llamarlo dos veces, meterlo en un `setTimeout`, guardar una copia en una variable global. Todas empeoran el código. La explicación es la de la instantánea: dentro de este render, ese valor no va a cambiar nunca, porque es una constante capturada. El valor nuevo vive en el render siguiente.

**Memorizar todo con `useMemo` y `useCallback` sin haber medido nada.** Llega de algún video que dice que es buena práctica, y termina en componentes donde cada valor y cada función están envueltos, el archivo es ilegible y el rendimiento es igual o peor, porque las comparaciones de dependencias también cuestan. Además suele venir acompañado de dependencias mal declaradas dentro de los memos, que es el error uno con otra máscara. La corrección es de proceso: ten un número antes y un número después. En el proyecto final, cualquier `useMemo` sin una medición que lo justifique en el `README.md` cuenta como deuda de calidad, no como optimización.

**Sobre la IA en este tema.** Todo lo que generes con un asistente va declarado en `IA.md`; sin declarar, la entrega se califica en 0 y no hay reintento. Declararlo no baja la nota, porque lo que se evalúa es tu capacidad de auditarlo. En hooks el patrón de fallo más frecuente son efectos con dependencias incompletas y sin limpieza, precisamente porque el asistente escribe cada fragmento aislado y no sabe qué más hay en tu componente.

---

## Fuentes de esta sesión

- Meta. *Built-in React Hooks*. https://react.dev/reference/react/hooks
- Meta. *React: Quick Start*. https://react.dev/learn

La referencia de hooks tiene, en cada página, una sección de problemas comunes con el título exacto del síntoma: "mi efecto corre dos veces", "mi efecto corre en un ciclo infinito". Úsala así: cuando algo falle, busca el síntoma ahí antes de buscar en foros. Está escrito por la gente que escribió el hook.

---

## Antes de la sesión 14

Lee la página de introducción del App Router de Next.js, solo hasta la sección de estructura de carpetas, y dale una ojeada al documento de componentes de servidor y cliente. Diez minutos. Lo que quiero es que llegues con una pregunta concreta ya formulada: *si un componente corre en el servidor, ¿puede tener `useState`?* No la respondas todavía; llega con la duda.

La semana entrante se **entrega la Misión 06** antes de la medianoche del día anterior, con las 10 horas de presupuesto que ya conoces. Si ya vas bien encaminado, usa el tiempo restante en los cuatro XP de efectos, que son los que casi nadie se acuerda de revisar. Y hay **Quiz 5** de apertura, 15 XP, sobre React y hooks: preguntas de aplicación, no de definiciones, del tipo "aquí hay un `useEffect` roto, di qué le falta".
