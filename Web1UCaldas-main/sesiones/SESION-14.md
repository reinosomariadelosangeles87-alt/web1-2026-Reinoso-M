# Sesión 14 · Next.js y estrategias de renderizado

**Módulo 4** · Zona 4: La Fortaleza de Componentes
**Lo que sale de esta noche:** un proyecto de Next.js corriendo, con una ruta creada por sistema de archivos, y la capacidad de señalar en tu propio código qué componente corre en el servidor y qué componente corre en el navegador
**Lo que se entrega hoy:** Misión 06 — Mazmorra por turnos en React (100 XP · 10 h) · también hay Quiz 5 (15 XP)

---

## Por qué el renderizado deja de ser una sola decisión

Todo lo que has hecho hasta esta noche —el juego de memoria en JavaScript puro, la mazmorra en React con Vite— vive entero en el navegador. Una sola estrategia de renderizado, tomada sin darte cuenta, para toda la aplicación. Esta noche descubres que hay tres, que cada una cuesta algo distinto, y que en el App Router de Next.js se eligen componente por componente.

Ese es el cambio mental grande: **deja de pensar el renderizado como una propiedad de la aplicación y empieza a pensarlo como una decisión por pedazo de pantalla.** Si de esta noche te llevas solo dos cosas —qué cuesta cada estrategia, y dónde va `"use client"`— la sesión hizo su trabajo.

Vas a querer hablar de despliegue, de rutas dinámicas, de bases de datos, de autenticación. Todo eso llega; anótalo como deuda y sigue. Y traes una ventaja: acabas de entregar la mazmorra, ya sufriste la arquitectura de componentes con tus propias manos, y ya sabes lo que es un estado en el nivel equivocado. La frontera nueva de esta noche es la misma clase de decisión, un nivel más arriba.

Empieza por una comprobación que puedes hacer ahora mismo. Abre tu mazmorra corriendo en Vite, haz clic derecho y elige **"Ver código fuente de la página"** —el código fuente, no el inspector de elementos, y la distinción importa. Antes de mirar, respondete: **¿dónde está la mazmorra que estás viendo en pantalla?**

Esto es todo lo que hay:

```html
<!doctype html>
<html lang="es">
  <head><title>Mazmorra de Caldas</title></head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.tsx"></script>
  </body>
</html>
```

Un `div` vacío. El juego que estás viendo no está en el documento que el servidor mandó. Abre ahora el inspector de elementos al lado, donde sí está todo el árbol: dos vistas del mismo momento, una vacía y una llena. La diferencia entre las dos es esta sesión entera.

Y la pregunta que hace que importe: ¿qué ve un buscador que quiera indexar esta página? ¿Qué ve alguien con una conexión mala en el segundo y medio que tarda el JavaScript en descargar? Ve un `div` vacío. ¿Tiene que ser así?

---

## Qué agrega un meta-framework sobre React

React resuelve un problema y solo uno: describir interfaces en función del estado. No dice nada sobre cómo se navega entre pantallas, cómo se optimizan las imágenes, cómo se parte el código en pedazos para no descargarlo todo de una vez, ni cómo se trae información del servidor. Todo eso hay que decidirlo, elegir librerías y configurarlo. En la mazmorra de la Misión 06 no lo notaste porque el juego es una sola pantalla; en cuanto haya cinco, se nota.

Un **meta-framework** es lo que se construye encima de React para tomar esas decisiones por ti. Next.js aporta tres cosas.

La primera es el **enrutamiento por sistema de archivos**. La estructura de carpetas *es* la estructura de URLs, sin un archivo de configuración de rutas en el medio:

```
app/
  page.tsx                    →  /
  juego/page.tsx              →  /juego
  puntajes/page.tsx           →  /puntajes
  juego/[nivel]/page.tsx      →  /juego/3   (segmento dinámico)
  layout.tsx                  →  envuelve todo lo de abajo
```

Los nombres son obligatorios: `page.tsx` y `layout.tsx` no son convenciones sugeridas, son las palabras que el framework busca. Y `layout.tsx` no se vuelve a montar al navegar entre páginas hermanas, que es lo que hace que el HUD del juego no parpadee al cambiar de pantalla.

La segunda es la **optimización automática**: partir el código por ruta para que la página de puntajes no descargue el motor del juego, procesar imágenes, precargar lo que se va a necesitar. Cosas que hacen falta y que nadie configura por gusto.

La tercera, y la que hay que decir con el nombre completo porque es la filosofía del asunto: **convenciones en lugar de configuración**. Si el archivo se llama `page.tsx` es una página; si se llama `loading.tsx` es lo que se muestra mientras carga; si se llama `error.tsx` es lo que se muestra cuando algo truena. No hay que declararlo en ninguna parte. **Es la diferencia entre un edificio donde cada puerta tiene un letrero improvisado y uno donde todo el mundo sabe que la salida de emergencia está al final del pasillo.** Cuestas algo en libertad y ganas mucho en que cualquiera pueda entrar a tu proyecto y orientarse.

Y el costo, que lo vas a sentir: cuando quieras hacer algo que el framework no previó, pelear contra la convención es más difícil que configurar desde cero. Es un intercambio, no un regalo.

---

## Las tres estrategias, y qué cuesta cada una

Las tres se leen igual: quién arma el HTML, cuándo, qué se gana, qué se paga.

**CSR, renderizado en el cliente.** Es lo que acabas de ver en el código fuente de tu mazmorra. El servidor manda un HTML esencialmente vacío con una etiqueta `script`; el navegador descarga el JavaScript, lo ejecuta, y ese JavaScript construye la página. La ventaja es que es lo más simple de desplegar: son archivos estáticos, el servidor no piensa, cualquier hosting sirve. Después de la primera carga, navegar es instantáneo porque no hay más viajes al servidor por HTML. Lo que se paga son dos cosas concretas: **es malo para SEO**, porque un rastreador que no ejecute JavaScript ve el `div` vacío, y **la primera carga es lenta**, porque hay que descargar y ejecutar todo el JavaScript antes de que aparezca el primer pixel de contenido. Para un juego al que llegas desde un enlace directo y en el que pasas media hora, el intercambio es razonable. Para una página de la que dependen visitas de buscadores, no.

**SSR, renderizado en el servidor.** El servidor arma el HTML completo, con los datos ya dentro, y lo manda listo. El navegador lo pinta de inmediato; el JavaScript llega después y solo se encarga de **hidratar**, es decir enganchar los manejadores de eventos al HTML que ya está en pantalla. Se gana lo que se perdía: SEO, porque el rastreador recibe contenido de verdad, y datos frescos, porque el HTML se armó en el momento de la visita. Lo que se paga es **CPU por cada visita**: cada petición hace trabajar al servidor, y eso cuesta dinero y añade latencia entre el clic y el primer byte. Es el restaurante que cocina el plato cuando lo pides: fresco, y hay que esperar y pagar la cocina.

**SSG, generación estática.** El HTML se genera una sola vez, al compilar, y se sirve desde una CDN o desde el Edge, cerca del usuario. Es el más rápido de los tres por un margen amplio y con **cero cómputo por visita**: el servidor solo entrega un archivo que ya existía. Es ideal cuando el contenido cambia poco. Lo que se paga es la frescura: si el contenido cambió después del `build`, el usuario ve lo viejo hasta el siguiente despliegue. Siguiendo la analogía, es el pan de la mañana, horneado antes de que abra el local. Instantáneo, y no lo hicieron para ti.

Cómo se decide en la práctica, que es lo que vas a necesitar el sábado. El reglamento del juego, la página de instrucciones, el "acerca de": eso cambia una vez al mes y es SSG. La tabla de puntajes global, que tiene que mostrar lo que pasó hace un minuto y a la que se llega desde un enlace compartido: SSR. El tablero del juego, con la cuadrícula, el combate y el inventario, que necesita `useState` y responde a teclas: eso es y tiene que ser cliente. **La misma aplicación, tres estrategias, y ninguna es la correcta para todo.**

### En el App Router esto se decide por componente

Esta es la idea central, la que conviene llevarse aunque olvides todo lo demás.

En las versiones anteriores de los frameworks, y en las herramientas que ya conoces, esta elección era una propiedad de la aplicación entera. Elegías una plantilla, y toda tu aplicación era CSR. En el App Router de Next.js **la decisión se toma por componente**, y una misma página puede tener una parte generada al compilar, una parte armada en el servidor en el momento de la visita y una parte que solo existe en el navegador.

**Hasta hoy elegías el material con el que se construye toda la casa; ahora eliges el material de cada pared.** La cimentación puede ser prefabricada, la cocina se arma en sitio y la ventana tiene que abrirse y cerrarse. Nadie construye una casa entera de un solo material porque cada parte hace otra cosa.

Y la consecuencia incómoda, porque conviene que la sepas antes: si la decisión se toma por componente, entonces **hay que tomarla muchas veces**, y equivocarse es fácil. De eso trata la segunda mitad de la noche.

### Las ideas que hay que llevarse

**Las tres estrategias no se ordenan de peor a mejor: se ordenan por lo que cuestan.** CSR cuesta primera carga y SEO. SSR cuesta CPU por visita. SSG cuesta frescura. Elegir bien es saber cuál de esos tres costos te duele menos en esa pantalla concreta.

**En el App Router el renderizado es una decisión por componente, no por aplicación.** Es lo que hace posible que la página de reglas sea estática y el tablero sea interactivo en el mismo proyecto.

**Hidratar no es dibujar otra vez: es enganchar el comportamiento a un HTML que ya está en pantalla.** Entender esa palabra evita la mitad de las confusiones sobre por qué en SSR la página se ve antes de que funcione. Hay un intervalo en el que ves el botón y el botón todavía no responde, y eso no es un bug.

### Ponte a prueba

*Tu mazmorra de la Misión 06, ¿podría ser SSG?* La respuesta interesante es "en parte": la cuadrícula inicial y las reglas sí, el juego en curso no. Si llegaste ahí, entendiste que la decisión se fracciona.

*Si SSG es el más rápido y el más barato, ¿por qué no todo es SSG?* Porque cuesta frescura, y hay pantallas donde la frescura es el producto. La tabla de puntajes es el caso.

*¿Qué ve exactamente el rastreador de un buscador en una aplicación CSR?* Ya lo viste en pantalla al empezar la sesión: un `div` vacío.

---

## Práctica en clase: la misma página en CSR, SSR y SSG

La idea es construir **la misma pantalla tres veces** y comprobar la diferencia en el código fuente y en la pestaña de red, no en una diapositiva. La pantalla es la tabla de puntajes de la mazmorra, que es lo próximo que vas a necesitar de verdad.

```bash
npx create-next-app@latest fortaleza-14 --typescript --app --no-tailwind
cd fortaleza-14
npm run dev
```

De la estructura que se creó, por ahora solo te interesan `app/page.tsx` y `app/layout.tsx`. Los demás archivos pueden esperar.

**Primero, crea una ruta nueva sin configurar nada.** Haz `app/puntajes/page.tsx` con un párrafo dentro y navega a `/puntajes`. Funcionó. No hubo router, ni registro de rutas, ni importaciones. Es un momento pequeño y genuinamente convincente.

**Segundo, la versión CSR, que es la que ya sabes hacer.** Escríbela con `useEffect`, tal como lo hiciste en la sesión 13, porque partir de lo conocido es lo que hace visible el contraste.

```tsx
// app/puntajes/csr/page.tsx
"use client";

import { useEffect, useState } from "react";

interface Puntaje { jugador: string; puntos: number; nivel: number; }

export default function PuntajesCSR() {
  const [puntajes, setPuntajes] = useState<Puntaje[]>([]);
  const [cargando, setCargando] = useState(true);

  useEffect(() => {
    fetch("/api/puntajes")
      .then((r) => r.json())
      .then((datos: Puntaje[]) => setPuntajes(datos))
      .finally(() => setCargando(false));
  }, []);

  if (cargando) return <p>Cargando el salón de la fama...</p>;

  return (
    <ol>
      {puntajes.map((p) => (
        <li key={p.jugador}>{p.jugador} — {p.puntos}</li>
      ))}
    </ol>
  );
}
```

Ahora la comprobación, que es la parte que importa: **"Ver código fuente de la página"**. Los puntajes no están. Está el "Cargando el salón de la fama...". Eso es literalmente lo que recibe un rastreador. Abre además la pestaña de red y mira que hay dos viajes: uno por el HTML y otro por los datos, en serie, uno después del otro. Esa es la primera carga lenta, medida y no argumentada.

**Tercero, la versión SSR.** Borra el `"use client"`, borra el `useEffect`, borra el `useState`, y escribe la función `async`.

```tsx
// app/puntajes/ssr/page.tsx
// Sin "use client": esto es un componente de servidor.
interface Puntaje { jugador: string; puntos: number; nivel: number; }

async function traerPuntajes(): Promise<Puntaje[]> {
  const res = await fetch("https://ejemplo.api/puntajes", { cache: "no-store" });
  if (!res.ok) throw new Error(`El servidor respondió ${res.status}`);
  return res.json();
}

export default async function PuntajesSSR() {
  const puntajes = await traerPuntajes();   // el await está en el componente. Sí, se puede.
  return (
    <ol>
      {puntajes.map((p) => (
        <li key={p.jugador}>{p.jugador} — {p.puntos}</li>
      ))}
    </ol>
  );
}
```

Dos cosas aquí, y ninguna es menor. La primera: **el componente es `async` y tiene un `await` en el cuerpo.** En la sesión 12 quedó dicho que un componente debía ser puro y que no se hacían llamadas de red en el cuerpo. La contradicción es aparente y vale la pena resolverla: esa regla vale para componentes de cliente, que pueden ejecutarse muchas veces. Un componente de servidor corre una vez, en el servidor, para producir HTML. La segunda: **el código quedó más corto que la versión CSR.** Se fueron el estado de carga, el efecto y una variable. Cuenta las líneas.

Y la comprobación: código fuente de la página. **Los puntajes están ahí, en el HTML.** Un solo viaje en la pestaña de red. Ese es el argumento del SEO, visible.

**Cuarto, prueba esto y observa qué pasa.** Intenta agregar interactividad al componente de servidor: un botón para ordenar la tabla.

```tsx
// MAL, y a propósito: esto es un componente de servidor
export default async function PuntajesSSR() {
  const puntajes = await traerPuntajes();
  const [orden, setOrden] = useState("puntos");   // ← truena
  return <button onClick={() => setOrden("nivel")}>Ordenar</button>;
}
```

Lee el error completo: *"You're importing a component that needs `useState`. This React hook only works in a Client Component."* La reacción natural es poner `"use client"` arriba. Hazlo. Funciona, el error desaparece. **Y aquí hay que frenar en seco**, porque acabas de aprender el reflejo que te va a costar el módulo. Deja escrito en algún lado *"lo arreglamos mal"*, porque más adelante en esta misma sesión se explica por qué.

**Quinto, la versión SSG y la palabra `build`.** Cambia una sola cosa del componente de servidor: el `cache: "no-store"` por el comportamiento estático.

```tsx
// app/reglas/page.tsx — contenido que cambia una vez al mes: SSG
async function traerReglas() {
  // Sin "no-store": Next.js puede resolver esto en tiempo de build.
  const res = await fetch("https://ejemplo.api/reglas");
  return res.json();
}

export default async function Reglas() {
  const reglas = await traerReglas();
  return <article>{reglas.texto}</article>;
}
```

Corre `npm run build` y detente en la tabla que imprime al final, donde cada ruta aparece marcada como estática o dinámica. Esa tabla es la mejor herramienta de diagnóstico de la noche y hay que aprender a leerla: **si una ruta que creías estática aparece como dinámica, algo en el árbol la volvió dinámica y hay que averiguar qué.** Revisar esa tabla antes de entregar la Misión 07 te va a ahorrar preguntas.

**Sexto, mide en lugar de creer.** Con los tres builds hechos, compara en la pestaña de red el tamaño del JavaScript que se descarga y el tiempo hasta el primer contenido en pantalla. No hace falta rigor de laboratorio; hace falta que veas con tus propios números que la diferencia es real. Y fíjate en el dato que más sorprende: la ruta SSG descarga **menos JavaScript** que la CSR, porque el componente de servidor nunca se envía al navegador.

Cuatro dudas que aparecen siempre en este taller. En un componente de servidor se puede usar `await` y en uno de cliente no, porque el de servidor se ejecuta una vez para producir HTML y el de cliente puede ejecutarse muchas veces; esperar dentro de algo que se re-ejecuta sin control es la receta del bucle infinito, y para eso existía `useEffect`. Si pones un `console.log` en un componente de servidor no lo vas a encontrar en la consola del navegador: está en la terminal donde corre `npm run dev`, porque ese código corrió en el servidor, y esa sola experiencia enseña más sobre la frontera que un diagrama. Esto no reemplaza tener un backend: reemplaza la capa de API que hacía de intermediaria para leer datos, no la base de datos ni la lógica de negocio. Y si te asomas a *streaming*, `Suspense` o rutas paralelas, existen, resuelven un problema real de página lenta, y no son de hoy.

---

## La frontera servidor/cliente

Este es el bloque que decide si la Misión 07 te sale o la sufres. Y empieza recogiendo el error del taller: pusiste `"use client"` para que el `useState` funcionara, el error desapareció, y nadie preguntó qué más pasó.

Primero la regla base. **Por defecto, en el App Router, todo es componente de servidor.** No hay que escribir nada para que lo sea; se es por omisión. Y eso implica tres cosas.

La primera: **tu código corre en el servidor y nunca se envía al navegador.** No es que se envíe y no se ejecute: no se envía. Si un componente de servidor importa una librería de trescientos kilobytes para formatear fechas, el usuario descarga cero. Esa es la razón por la que la ruta SSG del taller pesaba menos que la CSR.

La segunda, que es la que más cambia la cabeza: **puede hablar con la base de datos directamente, sin una API en el medio.** Todo lo que has hecho hasta ahora asumía que el navegador pide y un servidor responde por HTTP, y que había que escribir ese servidor. Un componente de servidor ya está en el servidor. La consulta va en el cuerpo del componente.

```tsx
// Esto corre en el servidor. La consulta no viaja, el resultado sí, ya como HTML.
export default async function Ranking() {
  const filas = await db.query("SELECT jugador, puntos FROM puntajes ORDER BY puntos DESC LIMIT 10");
  return <ol>{filas.map((f) => <li key={f.jugador}>{f.jugador}: {f.puntos}</li>)}</ol>;
}
```

La tercera: **no puede tener interactividad.** No hay `useState`, no hay `useEffect`, no hay `onClick`, no hay `window` ni `localStorage`. Y la razón es sencilla: ese código ya terminó de ejecutarse cuando el usuario ve la página. No hay nadie escuchando.

De ahí sale para qué existe `"use client"`. No es una optimización ni una preferencia de estilo: es la declaración de que **este componente y todo lo que importe tienen que enviarse al navegador porque necesitan estar vivos ahí.** Y lo justifican exactamente tres cosas: que uses hooks de estado o efecto, que uses eventos del usuario como `onClick` u `onChange`, o que uses APIs del navegador como `window`, `document` o `localStorage`. Si no es ninguna de las tres, no lo pongas.

La imagen que conviene tener: **el componente de servidor es el programa impreso de una obra de teatro y el componente de cliente es el actor.** El programa se imprime antes, llega completo a las manos del espectador, no cambia y es baratísimo de repartir. El actor tiene que estar presente, respirar y reaccionar a lo que pasa en la sala, y eso cuesta. Nadie imprime el programa con actores dentro, y nadie pone un actor a hacer el trabajo de una hoja de papel. La pregunta correcta ante cada componente no es "¿lo hago cliente o servidor?", es **"¿esto tiene que reaccionar a algo?"**.

### El error más caro del módulo, y por qué es invisible

Ahora sí, lo que quedó escrito como *"lo arreglamos mal"*.

Cuando aparece el mensaje de *"this hook only works in a Client Component"*, la reacción natural —y la que un asistente de IA sugiere con muchísima frecuencia— es subir el `"use client"` hasta que el error desaparezca. Y como el `layout.tsx` de la raíz envuelve absolutamente todo, ponerlo ahí hace desaparecer todos los errores de una vez. Es la solución más eficiente posible para el síntoma.

Y es un desastre, porque **la directiva se hereda hacia abajo.** Todo lo que un componente de cliente importa se convierte en cliente. Un `"use client"` en la raíz convierte la aplicación entera en cliente: todo el código se envía al navegador, el HTML vuelve a ser un `div` vacío, se pierde el SEO, se pierde el acceso directo a datos, y te queda una aplicación de React con Vite pero con más configuración encima. **Perdiste el beneficio entero y no se dio ni un error.** Es apagar el interruptor general porque parpadeaba un bombillo.

La regla operativa: **`"use client"` va lo más abajo posible en el árbol.** No en la página; en el componente que efectivamente necesita el hook. Y el patrón que resuelve el noventa por ciento de los casos es aislar la parte interactiva en su propio componente.

```tsx
// app/puntajes/page.tsx — SERVIDOR: trae los datos, no necesita interactividad
import { TablaOrdenable } from "./TablaOrdenable";

export default async function Puntajes() {
  const puntajes = await traerPuntajes();      // acceso a datos aquí arriba
  return (
    <section>
      <h1>Salón de la fama</h1>
      <TablaOrdenable puntajes={puntajes} />   {/* los datos bajan como props */}
    </section>
  );
}
```

```tsx
// app/puntajes/TablaOrdenable.tsx — CLIENTE: solo esto viaja al navegador
"use client";

import { useState } from "react";

export function TablaOrdenable({ puntajes }: { puntajes: Puntaje[] }) {
  const [orden, setOrden] = useState<"puntos" | "nivel">("puntos");
  const ordenados = [...puntajes].sort((a, b) => b[orden] - a[orden]);
  return (
    <>
      <button onClick={() => setOrden("nivel")}>Ordenar por nivel</button>
      <ol>{ordenados.map((p) => <li key={p.jugador}>{p.jugador} — {p[orden]}</li>)}</ol>
    </>
  );
}
```

Fíjate en que la copia antes del `sort` sigue siendo la de la sesión 7, que las props siguen bajando como en la sesión 12, y que el único cambio real es una línea que declara dónde vive cada mitad. **No es un paradigma nuevo: es la misma arquitectura con una frontera dibujada.**

Un límite práctico que te vas a encontrar: las props que cruzan la frontera del servidor al cliente tienen que ser serializables. Datos, sí; funciones, no. Si intentas pasar una función desde un componente de servidor a uno de cliente, el error es claro y ahora ya sabes por qué: hay que atravesar la red y una función no viaja.

### Las ideas que hay que llevarse

**Todo es servidor hasta que demuestres que necesita ser cliente.** La carga de la prueba está en la interactividad, no en el valor por omisión. La pregunta en cada componente es "¿esto tiene que reaccionar a algo?".

**`"use client"` se hereda hacia abajo, así que ponerlo arriba lo pone en todo.** Es la razón por la que va lo más abajo posible, y es lo que separa un proyecto de Next.js de un proyecto de React con pasos extra.

**El acceso a datos sube y la interactividad baja.** El componente de servidor de arriba trae la información; el componente de cliente de abajo la recibe como props y reacciona. Si respetas esa forma, la mayoría de las decisiones se toman solas.

### Ponte a prueba

*Si pones `"use client"` en el `layout.tsx` de la raíz, ¿qué error vas a ver?* Ninguno, y ese es exactamente el punto. Es un fallo silencioso de arquitectura, del que solo te enteras mirando el código fuente o la tabla del `build`.

*¿Puede un componente de cliente contener un componente de servidor?* No de la forma en que uno lo espera, y pensar por qué —el hijo tendría que renderizarse en un lugar al que el padre ya no tiene acceso— consolida la idea de que la frontera es real y no una etiqueta.

*En la mazmorra que entregas hoy, ¿qué componentes llevarían `"use client"` si la migraras?* Señálalo sobre tu propio árbol de componentes. Casi siempre la respuesta es "el tablero y el HUD sí, la pantalla de reglas y el marco no".

---

## Práctica en clase: migrar el tablero de puntajes a Next.js

El objetivo es concreto y acotado: **un proyecto de Next.js con tres rutas, donde cada una use una estrategia distinta y sepas justificar por qué.** Son tres tareas.

**Tarea 1: las tres rutas.** Una página de reglas del juego con contenido que cambia poco, resuelta como SSG. Una tabla de puntajes que muestre datos frescos, resuelta como SSR. Y una pantalla de juego con interactividad mínima —un contador, un botón— que sea cliente. Corre `npm run build` y comprueba en la tabla que cada ruta salió marcada como esperabas. Si no coincide, ahí está el ejercicio de verdad.

**Tarea 2: la frontera bien puesta.** En la tabla de puntajes, la carga de datos va en el componente de servidor y el botón de ordenar va en un componente de cliente aparte. La regla de aceptación es dura y fácil de verificar: **si hay un solo `"use client"` en un `layout.tsx`, la tarea está mal hecha.**

**Tarea 3: escribe la justificación.** En el `README.md` del proyecto, una tabla con tres filas —una por ruta— y tres columnas: qué estrategia usaste, por qué, y qué costo aceptaste al elegirla. Esa tercera columna es la que importa, porque es la que demuestra que entendiste que las tres estrategias cuestan algo. Sin esa columna la tarea no cuenta.

Cuatro tropiezos habituales mientras haces esto. Vas a poner `"use client"` en la raíz, o en tres archivos donde no hace falta, casi siempre porque un asistente lo puso: borra el que no necesites y lee el error que aparece; si no aparece ninguno, sobraba. Vas a escribir `useEffect` y `fetch` en un componente de servidor por memoria muscular de la sesión 13; recuerda que si estás en el servidor no necesitas esperar en el navegador, ya estás donde están los datos. Vas a usar `window` o `localStorage` en un componente de servidor y el error va a decir que no están definidos: ese código corrió donde no hay ventana, y si necesitas `localStorage`, eso es cliente por definición. Y vas a intentar pasar una función como prop del servidor al cliente: la solución es al contrario, la función se define dentro del componente de cliente y del servidor solo bajan datos.

Sobre la IA en este tema, hay un fallo muy específico y reconocible. La mayor parte del material sobre el que se entrenaron los asistentes es de la época anterior al App Router, así que producen código con la estructura antigua o ponen `"use client"` en todas partes por seguridad. Si le pides "una página de Next.js que traiga datos", hay una probabilidad alta de recibir `useEffect` con `fetch` en un componente de cliente, que funciona y desperdicia el framework entero. Todo lo generado va declarado en `IA.md`; sin declarar, la entrega se califica en 0 y no hay reintento. Declararlo no baja la nota, porque lo que se evalúa es tu capacidad de auditarlo, y encontrar y explicar este fallo vale la insignia 🔍 **Cazador de alucinaciones**.

---

## Tu misión de la semana: Mazmorra por turnos en React (100 XP)

Esta noche **se entrega la Misión 06**, así que lo primero es administrativo: confirma que la rama está subida, que el `IA.md` está diligenciado y que el `README.md` tiene la tabla de estado que se pidió en la sesión 12. Un repositorio sin `IA.md` se califica en cero y no admite reintento.

Lo que construyes es un juego de exploración por turnos con cuadrícula, un personaje que se mueve, enemigos, combate por turnos, inventario y puntos de vida, todo con componentes de React.

### Por qué el juego no es el reto

Muchos van a entregar un juego que funciona y a creer que eso era todo. **El reto real no era el juego, era la arquitectura.** En términos de lógica, esto es más simple que el juego de memoria del módulo 3. Lo difícil es otra cosa.

Si pusiste todo el estado en la raíz, cada tecla presionada vuelve a renderizar las sesenta y cuatro casillas, `App` tiene ochocientas líneas y cambiar el inventario obliga a leer el archivo entero. Y si lo dispersaste demasiado, la vida del jugador vive en dos sitios, el turno se desincroniza y aparece el bug que ya conoces del módulo 3 con sintaxis nueva. **Los dos extremos funcionan en la demostración y los dos se rompen cuando el juego crece un poco.** Encontrar el punto medio —lo más arriba que sea necesario y ni un nivel más— es el aprendizaje de la misión.

Y esa es exactamente la misma clase de decisión que la frontera servidor/cliente de esta noche, un nivel más arriba. Allá la pregunta era *¿quién necesita este dato?*; aquí es *¿esto tiene que reaccionar a algo?*. Las dos se resuelven poniendo la cosa lo más abajo posible en el árbol.

### Cómo se califica

| Criterio | XP |
|---|---|
| Funciona: el juego se juega y cumple lo pedido | 35 |
| Calidad del código: nombres claros, sin repetición ni código muerto | 25 |
| Específico del módulo (ver abajo) | 20 |
| Proceso en Git: commits pequeños y descriptivos, PR completo | 10 |
| Auditoría de IA: `IA.md` real y reflexivo | 10 |

Los 20 XP específicos son la traducción exacta de las sesiones 12 y 13: componentes con una sola responsabilidad clara y reutilizables (6), estado en el nivel correcto del árbol y sin duplicación (6), listas con claves estables y no el índice (4), y efectos con dependencias correctas y limpieza donde hace falta (4). Los doce puntos de los dos primeros criterios son los del equilibrio arquitectónico, y son lo que se audita entre compañeros.

**Plazo:** antes de la medianoche del día anterior a esta sesión. Presupuesto estimado: **10 horas**, repartidas en 3 de diseñar componentes, 6 de programar y 1 de documentar. Las tres horas de diseño son reales y son las que deciden la nota; quien las salta gasta seis en volver a hacer lo mismo.

Esta noche también hay **Quiz 5 (15 XP)** sobre las sesiones 12 y 13: props frente a estado, dónde vive el estado, claves en listas, el arreglo de dependencias de `useEffect` y por qué hace falta la función de limpieza. Preguntas de aplicación.

---

## Errores que probablemente vas a cometer

**Poner `"use client"` en la raíz para silenciar errores.** Es el error central del tema y el más difícil de detectar, porque no produce ningún síntoma: la aplicación funciona, se ve igual y no imprime advertencias. Lo que pasó es que la directiva se heredó hacia abajo por todo el árbol, todo el código se está enviando al navegador, el HTML volvió a salir vacío y tu aplicación es React con Vite disfrazado de Next.js. Contra esto hay dos hábitos de verificación baratos: mira el código fuente de la página en el navegador para comprobar que el contenido está ahí, y lee la tabla que imprime `npm run build` para ver qué rutas quedaron estáticas. Sin esos dos hábitos, el error puede sobrevivir hasta la entrega final.

**Escribir `useEffect` con `fetch` donde no hace falta.** Vienes de la sesión 13 y de años de tutoriales donde traer datos era eso, y además es lo que un asistente sugiere la mayoría de las veces. El resultado funciona, y ese es el problema: nadie lo corrige porque nada truena. Pero convierte una página que podía llegar completa desde el servidor en una que llega vacía, hace dos viajes en serie donde bastaba uno y arrastra un estado de carga que no era necesario. La señal es sencilla: si el `useEffect` solo trae datos y no reacciona a nada que haga el usuario, ese código está en el lugar equivocado.

**Creer que la elección de estrategia es una preferencia de estilo.** La pregunta "¿cuál es la mejor?" no tiene respuesta única, y cuando se elige por lo que se leyó que es más moderno, sale una tabla de puntajes generada al compilar que muestra datos de hace tres días, o una página de reglas que hace trabajar al servidor en cada visita para devolver un texto que no cambió en un mes. Lo que corrige el hábito es escribir el costo aceptado y no la ventaja: esa columna del `README.md` donde tienes que decir qué pierdes con la estrategia que elegiste es la que produce el aprendizaje, porque no se puede llenar sin haber comparado.

**Olvidar que el componente de servidor corre en otro lugar.** Aparece como confusión de diagnóstico más que como error de código: buscas tus `console.log` en la consola del navegador y no están, usas `window` y te dice que no está definido, o no entiendes por qué un `alert` no aparece. La raíz es que todavía piensas en un solo entorno de ejecución, como en todo lo que has escrito este semestre. Ayuda un andamio temporal: durante la primera semana, escribe en la primera línea de cada archivo un comentario que diga si eso corre en el servidor o en el navegador. Se lo puedes quitar en dos semanas, y mientras lo usas dejas de cometer esta familia entera de errores.

---

## Fuentes de esta sesión

- Vercel. *Next.js: App Router — Getting Started*. https://nextjs.org/docs/app/getting-started
- Vercel. *Next.js: Server and Client Components*. https://nextjs.org/docs/app/getting-started/server-and-client-components

La documentación del App Router es la única fuente que conviene usar en este tema, y vale la pena saber por qué. Next.js cambió de arquitectura, hay una cantidad enorme de material anterior al App Router circulando en internet, y ese material no está equivocado en sí mismo pero describe otro framework. Un tutorial de hace tres años sobre obtención de datos en Next.js te va a enseñar funciones que hoy no se usan, y peor, va a reforzar la costumbre de traer datos desde el cliente. La página de *Server and Client Components* es la lectura corta que resuelve el noventa por ciento de las dudas de esta noche, y la que hay que releer antes de decidir dónde va cada `"use client"`.

---

## Antes de la sesión 15

Lee la sección "Módulo 4, sesión 15" de `GUIA-DEL-CURSO.md`, y de la documentación del App Router únicamente la página de *Getting Started* en las partes de estructura de proyecto y de enrutamiento —lo de layouts y páginas. Quince minutos, no más. Lo que necesitas es llegar con la estructura de carpetas ya vista y con una pregunta formulada: si la carpeta es la URL, ¿cómo se hace una URL que cambia, como `/juego/nivel/7`? Con eso abre la próxima sesión, y llegar con la pregunta hecha vale más que llegar con la respuesta leída.
