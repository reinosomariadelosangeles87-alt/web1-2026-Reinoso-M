# Sesión 15 · Integración de servicios de IA

**Módulo 4** · Zona 4: La Fortaleza de Componentes — última noche
**Lo que sale de esta noche:** una ruta de servidor que llama a un modelo de lenguaje sin exponer la clave, y la salida del modelo validada antes de mostrarla en pantalla
**Tu misión en curso:** Misión 07 — Marcador global con Next.js (100 XP · 9 h), que se entrega en la sesión 16

---

## Por qué pasar de usuario a integrador cambia las reglas

Usas asistentes de IA todos los días para hacer las misiones y los declaras en `IA.md` desde la sesión 1. Es decir: eres consumidor experto de modelos de lenguaje y nunca has visto qué hay del otro lado del cuadro de texto. Esta noche pasas de usuario a integrador, y ese cambio de rol trae tres responsabilidades que no tenías: la clave, la factura y la desconfianza.

El objetivo real de la sesión es estrecho a propósito: que salgas **incapaz de poner una clave de API en el cliente sin que te duela**, y con la certeza incómoda de que lo que devuelve un modelo es texto de un desconocido. Todo lo demás —qué proveedor, qué modelo, qué biblioteca— cambia cada seis meses y se aprende leyendo documentación. Esas dos cosas no.

Hay una trampa en la que es fácil caer con este tema, y conviene nombrarla: es divertidísimo escribir prompts y reírse de las respuestas, y de ahí no sale nada. **La gracia de esta noche no es que el modelo narre bien la partida; es que la clave no se pueda robar y que la respuesta no pueda romper la aplicación.** Igual con la conversación sobre qué modelo es mejor y cuál salió la semana pasada: hoy el proveedor es un detalle de implementación.

Antes de seguir, recupera la frontera de la semana pasada. **En la aplicación de Next.js que armaste, ¿qué archivos terminaron dentro del paquete de JavaScript que descarga el navegador, y qué archivos se quedaron en el servidor?** La respuesta: los componentes marcados con `"use client"` y todo lo que ellos importan viajan al navegador; los componentes de servidor y las rutas de `app/api/` no. Ten esa lista en dos columnas en la cabeza, porque es la herramienta de razonamiento de toda la noche.

Y ahora la pregunta que abre el tema real. No la respondas todavía: **si escribes tu clave de API en un archivo que tiene `"use client"` arriba, ¿dónde acaba esa clave?** En cuarenta minutos la vas a ver con tus propios ojos en la pestaña de Fuentes de las herramientas de desarrollo.

La noche en una frase: hoy metemos un modelo de lenguaje en el juego, y para hacerlo hay que aprender dos formas de desconfianza — desconfiar del navegador con nuestros secretos, y desconfiar del modelo con nuestra interfaz.

---

## Dónde vive la clave de API

Antes de la arquitectura, el problema en términos que tocan la billetera. **Si alguien encuentra tu clave de API, ¿qué es lo peor que puede hacer con ella, y a nombre de quién?**

La respuesta tiene dos partes y casi nunca aparece completa. Primero, puede hacer todas las llamadas que quiera. Segundo, y esto es lo que duele: **las pagas tú, porque la clave te identifica ante el proveedor.** No es un problema de privacidad abstracta, es una tarjeta de crédito prestada a internet.

Una clave de API es la llave de tu casa con tu dirección grabada en el llavero. Publicarla en el paquete de JavaScript es dejar ese llavero pegado en la puerta del edificio. No hace falta que nadie te odie para que entre; basta con que pase por ahí y le sirva.

### El paquete de JavaScript es público, y punto

Aquí hay que ser brutalmente claro, porque es el fallo de seguridad más común en trabajos de estudiantes, y no solo de estudiantes.

Todo lo que llega al navegador es público. No hay excepciones, no hay minificado que ayude, no hay variable de entorno con prefijo que lo salve. Cuando una herramienta de compilación arma el paquete de JavaScript, reemplaza las referencias a variables de entorno del cliente por su valor literal, en texto plano, dentro del archivo que cualquiera puede descargar. El navegador no puede ejecutar algo que no tiene, y por eso no puede haber secretos en el cliente: **si el código del navegador lo usa, el usuario lo tiene.**

La formulación que conviene repetir: *ofuscar no es proteger.* Minificar el código, poner la clave en un archivo con nombre raro, partirla en tres pedazos y concatenarla en tiempo de ejecución, codificarla en base64 —todo eso lo derrota una persona con las herramientas de desarrollo abiertas y diez minutos. La única protección real es que la clave nunca cruce la frontera.

### La ruta de servidor como intermediaria

La solución tiene una sola forma, y son tres cajas con dos flechas: navegador, tu servidor, proveedor del modelo. El navegador nunca habla con el proveedor. Le habla a tu servidor, y tu servidor —que sí tiene la clave, porque vive en una máquina que tú controlas— habla con el proveedor y devuelve el resultado.

```typescript
// app/api/narrar/route.ts
// Este archivo NO viaja al navegador. Nunca. Ni un byte.
// Es el único lugar del proyecto donde la clave puede aparecer.

export async function POST(request: Request) {
  // La clave se lee del entorno del servidor, no de un literal.
  const clave = process.env.CLAVE_MODELO;
  if (!clave) {
    // Fallar temprano y ruidoso: mejor un 500 claro que un bug silencioso.
    return Response.json({ error: "Configuración incompleta" }, { status: 500 });
  }

  const { partida } = await request.json();

  const respuesta = await fetch("https://api.proveedor.example/v1/mensajes", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      // La clave sale de aquí hacia el proveedor: servidor a servidor.
      Authorization: `Bearer ${clave}`,
    },
    body: JSON.stringify({
      modelo: "modelo-pequeno",
      // Presupuesto de salida: sin esto, una respuesta larga cuesta más.
      max_tokens: 200,
      mensajes: [
        {
          rol: "usuario",
          contenido: `Narra en dos frases esta partida: ${JSON.stringify(partida)}`,
        },
      ],
    }),
  });

  const datos = await respuesta.json();
  // Ojo: todavía NO hemos validado nada de lo que devolvió el modelo.
  return Response.json({ narracion: datos.contenido });
}
```

Detente en el nombre de la variable de entorno, porque ahí está la trampa. En Next.js, las variables que empiezan por `NEXT_PUBLIC_` se inyectan en el paquete del cliente **a propósito y por diseño**; es la forma de pasar datos no secretos, como la URL pública de la aplicación. Cualquier otra variable se queda en el servidor. Por eso `CLAVE_MODELO` está bien y `NEXT_PUBLIC_CLAVE_MODELO` es un desastre con un nombre que además te avisa. Ese prefijo se pone por desesperación, a la una de la mañana, porque "así por fin funcionó". Funciona, sí: funciona para todo el mundo.

El otro archivo que hay que nombrar: `.env.local` va en `.gitignore`, y en el repositorio se sube `.env.example` con los nombres de las variables y sin los valores. Y si la clave llega a entrar en un commit, borrarla en el commit siguiente **no la borra**: sigue en el historial, y el historial es público. Lo único que sirve es rotar la clave en el panel del proveedor y dar la vieja por perdida. Es la única vez en el semestre en que Git y seguridad se cruzan así, y la semana entrante el historial es el tema central.

### Respuestas en flujo y por qué importan

Con la arquitectura clara, el segundo tema es la espera. Una llamada a un modelo de lenguaje no responde en cincuenta milisegundos como una consulta a base de datos: responde en dos, cinco, quince segundos, y el tiempo depende de cuánto texto genere.

Aquí entra el flujo, el *streaming*. El proveedor no tiene que esperar a terminar la respuesta completa para empezar a mandarla: puede enviarla por pedazos, a medida que la produce. Tu servidor los reenvía, y el navegador los pinta apareciendo. El total de tiempo es el mismo o incluso un poco mayor; lo que cambia es **cuándo el usuario ve la primera palabra**, y eso lo cambia todo.

Es la diferencia entre un mesero que te trae los platos a medida que salen de la cocina y uno que espera a que esté todo listo para traerlo en una sola bandeja. La comida tarda igual. Sentado en la mesa, la experiencia no se parece en nada: con la bandeja te preguntas si te olvidaron, con los platos sabes que la cocina está trabajando.

### Las ideas que hay que llevarse

**Si el código del navegador puede leerlo, el usuario puede leerlo.** No es una regla de Next.js ni de React: es una propiedad de cómo funciona la web. La consecuencia práctica es que la pregunta "¿dónde pongo la clave?" tiene una sola respuesta correcta, y la pregunta "¿cómo la escondo en el cliente?" no tiene ninguna.

**Una ruta de servidor no es solo un pasamanos: es el lugar donde pones las reglas.** Además de guardar la clave, es donde limitas el tamaño del prompt, el número de llamadas por sesión, el presupuesto de tokens de salida y qué se le puede pedir al modelo. Si el navegador hablara directo con el proveedor, no tendrías dónde poner ninguna de esas reglas. La clave es el motivo obvio de la ruta; el control es el motivo bueno.

**Cada llamada cuesta dinero y ninguna es instantánea.** Estas dos propiedades no aparecen en ninguna otra API que hayas usado en el curso, y cambian el diseño de la interfaz. Cuesta dinero, entonces hay que poner límites. Tarda, entonces hay que mostrar progreso, permitir cancelar y decidir qué se hace cuando falla. Un botón que llama al modelo y no hace nada más durante ocho segundos es un botón roto, aunque el código sea correcto.

### Ponte a prueba

*¿Podría alguien seguir gastando tu cuota aunque la clave esté en el servidor?* Sí, llamando a tu ruta `/api/narrar` mil veces por segundo. La clave está a salvo, la factura no. Esconder la clave resuelve un problema, no todos: por eso hacen falta límites por sesión y un tope duro.

*Si el flujo tarda lo mismo en total, ¿por qué molestarse?* Porque el usuario no mide segundos, mide incertidumbre. Es la distinción entre rendimiento medido y rendimiento percibido, y la vas a necesitar el resto de tu carrera.

*¿Qué pasa si el usuario cierra la pestaña a mitad de una respuesta en flujo?* Que hay que cancelar la petición del lado del servidor, porque si no sigues pagando tokens que nadie va a leer.

---

## Práctica en clase: robar una clave en dos minutos

Primero se rompe, y el arreglo aparece como consecuencia. El riesgo de exponer una clave es abstracto hasta que la ves escrita en la pantalla. Después de este recorrido deja de ser abstracto.

**Primero, la aplicación mal construida.** Escríbela tú mismo, que es más contundente que abrir un archivo ya hecho.

```tsx
// app/narrador-malo/page.tsx
"use client";
// Este archivo viaja al navegador. Lo que escribamos aquí, viaja también.

import { useState } from "react";

// Clave de mentiras, con una forma reconocible a propósito.
// En una aplicación real esto vendría de NEXT_PUBLIC_ALGO, que es igual de malo.
const CLAVE = "sk-demo-CALDAS-2026-NUNCA-HAGAS-ESTO-99";

export default function NarradorMalo() {
  const [texto, setTexto] = useState("");

  async function narrar() {
    const r = await fetch("https://api.proveedor.example/v1/mensajes", {
      method: "POST",
      headers: { Authorization: `Bearer ${CLAVE}` }, // el navegador llama directo
      body: JSON.stringify({ mensajes: [{ rol: "usuario", contenido: "hola" }] }),
    });
    setTexto(JSON.stringify(await r.json()));
  }

  return <button onClick={narrar}>Narrar la partida</button>;
}
```

Usa una clave falsa, obviamente, pero con una forma que se parezca a las de verdad y con un texto reconocible dentro. Cuando aparezca en el paquete de JavaScript, reconocerla a simple vista es parte del efecto.

**Segundo, el robo.** Levanta la aplicación, abre las herramientas de desarrollo, ve a la pestaña **Fuentes** y usa la búsqueda global del panel —`Ctrl+Shift+F` en el panel de fuentes, no en la página— con el texto `sk-demo`.

Ahí está: la clave completa, en un archivo con nombre de hash, en texto plano, dentro del paquete que el servidor le entregó al navegador. No hiciste nada ilegal, no usaste ninguna herramienta especial, no hubo *hacking*. Usaste una función de búsqueda. Cuenta cuánto tardaste.

Ahora adelántate a la objeción obvia: haz la versión de producción con `npm run build && npm start` y repite la búsqueda. Con el código minificado, la clave sigue estando ahí, intacta, porque minificar cambia los nombres de las variables y no el valor de las cadenas de texto. Ese es el momento en que cae la ficha de que ofuscar no es proteger.

Mira también la pestaña **Red**: la petición sale del navegador con la cabecera `Authorization` completa y visible. Dos caminos independientes para conseguir lo mismo.

**Tercero, mueve la clave al servidor.** Crea `app/api/narrar/route.ts` con la estructura de arriba, pon la clave en `.env.local`, y haz que el componente de cliente llame a tu propio servidor. Repite la búsqueda en Fuentes: `sk-demo` ya no aparece. Ese contraste —la misma búsqueda, dos resultados— es lo que hay que recordar.

**Cuarto, prueba el prefijo y observa qué pasa.** Cambia `CLAVE_MODELO` por `NEXT_PUBLIC_CLAVE_MODELO` en `.env.local` y en el componente de cliente, reinicia el servidor de desarrollo, y busca otra vez. Volvió. El prefijo no es decorativo y Next.js está haciendo exactamente lo que se le pidió. Quítalo y sigue.

**Quinto, la latencia y los tres estados de la petición.** Con la ruta funcionando, el botón todavía es un botón roto.

```tsx
"use client";
import { useState } from "react";

export default function Narrador({ partida }: { partida: unknown }) {
  const [estado, setEstado] = useState<"listo" | "cargando" | "error">("listo");
  const [narracion, setNarracion] = useState("");
  // El controlador nos permite cancelar la petición en curso.
  const [control, setControl] = useState<AbortController | null>(null);

  async function narrar() {
    const c = new AbortController();
    setControl(c);
    setEstado("cargando");
    try {
      const r = await fetch("/api/narrar", {
        method: "POST",
        body: JSON.stringify({ partida }),
        signal: c.signal, // esto es lo que hace posible cancelar
      });
      if (!r.ok) throw new Error("El narrador no respondió");
      const datos = await r.json();
      setNarracion(datos.narracion);
      setEstado("listo");
    } catch {
      // Plan B: el juego sigue funcionando sin narración.
      setNarracion("El cronista de la mazmorra está dormido.");
      setEstado("error");
    }
  }

  return (
    <section>
      <button onClick={narrar} disabled={estado === "cargando"}>
        {estado === "cargando" ? "El cronista escribe..." : "Narrar la partida"}
      </button>
      {estado === "cargando" && <button onClick={() => control?.abort()}>Cancelar</button>}
      <p>{narracion}</p>
    </section>
  );
}
```

El `disabled` no es cosmético. Prueba esto: quítalo, haz seis clics rápido y mira la pestaña Red. Seis peticiones idénticas saliendo en fila, seis llamadas pagadas por un botón mal hecho. Multiplica por los usuarios de un juego que le guste a alguien.

Y el plan B, explícito: **la narración es un adorno, la partida es el producto.** Si el modelo falla, el juego tiene que seguir jugándose. Una función que se cae y arrastra a toda la interfaz con ella es una dependencia mal aislada.

**Sexto, el flujo, si te alcanza el tiempo.** Convierte la ruta en una respuesta en flujo y mira el texto apareciendo palabra por palabra. Es la parte más vistosa y la menos importante; si lo anterior te tomó más de lo previsto, déjala para la misión, donde vale como secundaria.

Cuatro dudas que aparecen siempre. No basta con poner la clave en una variable de entorno "normal" y usarla desde el cliente, porque **no existen variables de entorno en el navegador**: lo que hay es un reemplazo de texto en tiempo de compilación, así que no lees una variable en tiempo de ejecución, incrustas un literal. Cifrar la clave en el cliente parece razonable y no funciona, porque para descifrar necesitas la llave de descifrado y esa llave también tendría que estar en el cliente: moviste el problema, no lo resolviste; *no puedes guardar un secreto en una máquina que no controlas*. El `.env.local` no es lo mismo que las variables de entorno del servicio donde despliegas: en desarrollo el archivo hace el trabajo, y al desplegar hay que volver a escribir esas variables en el panel del proveedor de hosting, porque `.env.local` no se sube al repositorio y por lo tanto no llega al servidor —eso es un dolor de cabeza garantizado en la sesión 17. Y sobre cuánto cuesta de verdad: una partida narrada es una fracción de centavo, pero haz la otra cuenta, la del bucle infinito —una llamada por render, sesenta renders por segundo, treinta minutos de una pestaña olvidada abierta. Ese número sí impresiona, y es la razón de que el tope duro no sea paranoia.

---

## La salida del modelo es entrada no confiable

Antes de leer, respondete: **en la sesión 1 quedó dicho que en la web nadie confía en nadie. ¿En qué categoría entra el texto que devolvió el modelo: es código tuyo, o es entrada de un desconocido?**

Dudar es el punto. Se siente como código propio porque tú escribiste el prompt, tú pagaste la llamada y tú controlas la ruta. Pero el texto lo escribió un sistema probabilístico que no controlas, que puede haber sido influido por lo que el usuario escribió en el juego, y que no tiene ninguna obligación de respetar el formato que le pediste.

La frase del bloque: **la salida de un modelo de lenguaje es entrada de usuario con buena ortografía.** Merece exactamente la misma desconfianza que un formulario abierto en internet. Ni más, ni menos, ni distinta.

### Por qué esto no es paranoia

Son tres modos de fallo, en orden de gravedad creciente, y el juego los hace visibles.

El primero es el formato roto. Le pediste al modelo un JSON con tres campos y te devolvió el JSON envuelto en un bloque de código con acentos graves, o con una frase amable antes —"¡Claro! Aquí tienes el resumen de la partida:"— o con una coma final que hace que `JSON.parse` reviente. No hay mala intención: hay un sistema que genera texto plausible, y "texto plausible" incluye la cortesía. Si tu código asume el formato, tu código se cae.

El segundo es la ruptura de la interfaz. El modelo devuelve texto larguísimo donde esperabas dos frases, y el panel de narración se desborda y tapa el tablero. O devuelve una respuesta vacía. O devuelve el nombre de un objeto que no existe en el inventario, y el código que lo busca encuentra `undefined` y falla tres funciones más adelante, lejos del origen. Esto es lo que más vas a sufrir en la Misión 07, y probablemente no lo vas a atribuir al modelo, porque el error aparece en otro archivo.

El tercero es el peligroso: contenido activo. Si el modelo devuelve una cadena con etiquetas de HTML y tú la insertas con `dangerouslySetInnerHTML` o con `innerHTML`, acabas de darle a un texto que no controlas permiso para ejecutar en tu página. Y hay un camino para que eso no sea casualidad: el jugador escribe el nombre de su personaje, ese nombre entra en el prompt, y el jugador aprovecha para escribir instrucciones dentro del nombre. El modelo, que no distingue tus instrucciones de los datos del usuario, obedece. Eso es **inyección de prompt**, y la lección arquitectónica es que cualquier texto que venga del usuario y entre en el prompt convierte la salida en un canal que el usuario influye.

La analogía que ata todo: el modelo es un pasante brillante, rapidísimo y con cero criterio sobre lo que le conviene a tu empresa. Le pides un informe y te lo entrega en tiempo récord, bien escrito, y de vez en cuando incluye un párrafo que le dictó por teléfono alguien que se hizo pasar por el jefe. No lo dejas fuera del equipo por eso. **Le revisas todo lo que entrega antes de mandarlo al cliente.**

### La validación como frontera, no como parche

Hay una buena noticia: en React el riesgo de contenido activo está en buena parte mitigado por defecto, porque JSX escapa el texto que interpolas, así que `<p>{narracion}</p>` pinta las etiquetas como texto y no las ejecuta. Y hay una mala: eso deja de valer cuando usas `dangerouslySetInnerHTML`, cuando guardas ese texto en la base de datos y otra parte del sistema lo pinta de otro modo, cuando lo metes en un atributo de estilo o en un `href`, o cuando lo pasas a una biblioteca de Markdown que sí genera HTML. El nombre `dangerouslySetInnerHTML` es largo y feo a propósito: es una advertencia con forma de API.

La forma correcta de tratarlo es poner la validación **en la frontera**, es decir en la ruta de servidor, justo cuando el texto llega del proveedor y antes de que exista en cualquier otra parte del sistema.

```typescript
// app/api/narrar/route.ts (continuación)
// Contrato explícito de lo que aceptamos del modelo.
// Si no cumple, no entra al sistema.

const MAX_LARGO = 400;

function sanearNarracion(bruto: unknown): string | null {
  // 1. ¿Es siquiera una cadena? El modelo pudo devolver null, un objeto, nada.
  if (typeof bruto !== "string") return null;

  // 2. Recortar y verificar que no esté vacía.
  const limpio = bruto.trim();
  if (limpio.length === 0) return null;

  // 3. Tope de largo: protege la interfaz y el presupuesto.
  if (limpio.length > MAX_LARGO) return null;

  // 4. Lista de permitidos, no de prohibidos: solo texto de narración.
  //    Es más seguro decir qué SÍ se acepta que intentar enumerar
  //    todas las formas de escribir algo peligroso.
  const soloTexto = /^[\p{L}\p{N}\s.,;:!?¡¿'"()-]+$/u;
  if (!soloTexto.test(limpio)) return null;

  return limpio;
}

export async function POST(request: Request) {
  // ... la llamada al proveedor, como antes ...
  const narracion = sanearNarracion(datos?.contenido);

  if (narracion === null) {
    // Fallar de forma segura: sin narración, no con narración dudosa.
    return Response.json({ narracion: null, motivo: "salida no válida" });
  }
  return Response.json({ narracion });
}
```

El comentario del punto 4 contiene el principio de seguridad más transferible de la noche. Una lista de prohibidos —"quita los `<script>`"— pierde siempre, porque quien ataca solo necesita encontrar una forma que no anticipaste y hay infinitas. Una lista de permitidos gana por construcción, porque lo que no está permitido no pasa, incluso lo que no se te ocurrió. Es la lógica del portero que tiene la lista de invitados en vez de la lista de personas vetadas.

Para estructuras más complejas que una cadena —un objeto con campos, que es lo que vas a necesitar si le pides al modelo datos y no prosa— lo razonable es un validador de esquemas en lugar de escribir las comprobaciones a mano. Existen y vale la pena buscarlos cuando llegues a ese punto.

### Las ideas que hay que llevarse

**Todo texto que cruza una frontera se valida al entrar, no al salir.** La ruta de servidor es la aduana. Si el texto malo entra al sistema y lo validas en cada pantalla donde se muestra, un día alguien agrega una pantalla nueva y se olvida. Validar una vez, en el borde, es una decisión de arquitectura; validar en cada uso es una lista de tareas que alguien va a incumplir.

**Nadie confía en nadie, y eso ahora incluye al modelo que tú mismo llamaste.** Es literalmente la frase de la sesión 1 aplicada a un actor que se siente propio y no lo es. Si esta idea queda instalada, la transferencia a cualquier API de terceros que uses después es automática.

**Fallar de forma segura significa quedarse sin la función, no seguir con datos dudosos.** Cuando la validación rechaza la salida, la respuesta correcta es "no hay narración esta vez", no "mostremos lo que llegó y esperemos". Un juego sin narración es un juego. Un juego con una interfaz reventada por texto no controlado es un bug reportado por un jugador.

### Ponte a prueba

*Si JSX ya escapa el texto, ¿para qué sanear en el servidor?* Porque el escapado de JSX protege una pantalla, y el saneado protege el sistema: la base de datos, el marcador global, el correo que alguien mande después con ese texto, y la pantalla que un compañero escriba el mes entrante sin conocer esta conversación.

*Si le pides al modelo "responde solo con JSON", ¿ya está resuelto el problema de formato?* Mejora mucho la probabilidad y no da garantía. El prompt es una petición, no un contrato. Pedirlo bien y verificarlo son dos cosas distintas y las dos necesarias.

*Si el nombre del personaje lo escribe el jugador y entra en el prompt, ¿quién está escribiendo el prompt en realidad?* Los dos. Con eso se ve la inyección de prompt sin necesidad de montar ningún ataque.

---

## Práctica en clase: la ruta de servidor de tu Misión 07

**Antes de escribir la llamada al modelo, escribe en el `README.md` la tabla de la frontera.** Tres columnas: qué dato o secreto, si vive en servidor o en cliente, y por qué. La clave de API tiene que aparecer en esa tabla con la palabra "servidor" y una justificación de una línea. Si todavía no puedes llenar esa tabla, no entendiste la sesión 14 ni la de hoy, y es mejor descubrirlo esta noche que el sábado.

Lo que debería quedarte hecho, en orden: la ruta `app/api/narrar/route.ts` respondiendo; la clave en `.env.local` con `.env.local` en el `.gitignore` y un `.env.example` sin valores; la búsqueda de la clave en la pestaña Fuentes dando cero resultados; el componente de cliente con estados de carga, error y cancelación; y la función de saneado con su tope de largo. El flujo y el pulido visual son para la casa.

Cuatro tropiezos habituales. Vas a quedarte con la clave en un componente de cliente porque "así funcionaba y no quise romperlo": no lo dejes para después, esos son 6 XP directos. Vas a ver `.env.local` apareciendo en `git status`, y ahí hay que parar todo y arreglar el `.gitignore` antes de que haya un commit, porque después es mucho más caro. Vas a llamar al modelo dentro de un `useEffect` sin arreglo de dependencias o con dependencias mal puestas, que es la forma más limpia de fabricar un bucle infinito pagado: es el mismo error de la sesión 13, y la pestaña Red te lo cuenta petición por petición. Y vas a mandar la salida del modelo directo al estado sin pasar por el saneado, con la excusa razonable de que "el modelo siempre devuelve texto normal"; la validación no existe para el caso normal.

Sobre la regla de IA, esta noche hay una vuelta de tuerca: vas a usar un asistente de IA para escribir código que llama a un modelo de IA. Todo lo generado va en `IA.md` como siempre, y declararlo no baja la nota; lo que se evalúa es la auditoría, y sin declarar la entrega se califica en 0 sin reintento. En este tema los asistentes tienen un patrón de error muy concreto y muy caro: si le pides "un componente de React que llame a la API de un modelo", una fracción alta de las veces te va a devolver la llamada desde el cliente con la clave en una constante o en una variable con prefijo público, porque es el ejemplo más frecuente en el material del que aprendió. Es decir: **el asistente reproduce el fallo de seguridad más común precisamente porque es el más común.** Si lo aceptas sin leerlo, pierdes los 6 XP de la clave. Si lo detectas, lo documentas en `IA.md` y explicas por qué está mal, eso es exactamente la insignia 🔍 **Cazador de alucinaciones** y es el tipo de auditoría que el curso quiere.

---

## Lo que viene: Misión 07 · Marcador global con Next.js (100 XP)

Esta noche no hay entrega, pero la Misión 07 se entrega en la **sesión 16**, la semana entrante, así que conviene tener la cuenta clara desde ya.

Consiste en migrar la mazmorra a Next.js y agregarle una tabla de puntajes global con persistencia real, más una ruta de servidor que consulta un modelo de lenguaje para narrar cada partida.

### Cómo se califica

| Criterio | XP |
|---|---|
| Funciona: el marcador persiste y la aplicación cumple lo pedido | 35 |
| Calidad del código: nombres claros, sin repetición ni código muerto | 25 |
| Específico del módulo (ver abajo) | 20 |
| Proceso en Git: commits pequeños y descriptivos, PR completo | 10 |
| Auditoría de IA: `IA.md` real y reflexivo | 10 |

Los 20 XP específicos son la traducción exacta de las sesiones 14 y 15: la estrategia de renderizado argumentada por página (6), la clave de API nunca en el cliente y verificable en el navegador (6), la frontera servidor/cliente bien trazada (4), y la salida del modelo validada antes de usarse (4).

Detente en el "verificable en el navegador" de esos segundos seis puntos, porque se califica de forma literal: **se abre tu aplicación, se va a la pestaña Fuentes, se busca la clave, y si aparece son cero de seis.** No hay explicación en el `README.md` que compense eso. Es la única parte de la rúbrica de todo el semestre que se evalúa con una búsqueda de texto, y conviene que sepas exactamente cómo va a pasar.

**Misión secundaria opcional (+25 XP):** la narración con respuesta en flujo (streaming).

**Plazo:** antes de la medianoche del día anterior a la sesión 16. Presupuesto estimado: **9 horas**, repartidas en 2 de decidir el renderizado de cada página, 5 de programar, 1 de desplegar y 1 de documentar. La narración es la parte pequeña y la más divertida, así que la trampa es gastar seis horas ahí y llegar sin marcador; el marcador es lo que vale los 35 XP de "funciona".

La semana entrante viene cargada: se entrega la Misión 07, hay **Auditoría 4** cruzada de 25 XP sobre el código de un compañero, y **se forman los equipos de tres del proyecto final**. Sobre esto último, algo concreto para esta semana: llega a la sesión 16 con una idea de con quién quieres trabajar y con dos o tres temas posibles de juego. El proyecto final es un juego en equipos de tres y lo que queda del semestre se va en construirlo, así que la semana que se pierda eligiendo tema es una semana que no se recupera.

Y como cierre del módulo 4, la frase que lo resume, porque esta es la última noche de La Fortaleza de Componentes: **una fortaleza no se define por lo que deja entrar, sino por dónde pone la puerta.** Toda la arquitectura de estas cuatro sesiones —componentes, estado, frontera de renderizado, rutas de servidor, validación de salidas— es la misma decisión repetida a distintas escalas: elegir con cuidado qué cruza y en qué dirección.

---

## Errores que probablemente vas a cometer

**Poner la clave en el cliente y creer que el prefijo público la protege.** Es el error número uno y llega por un camino casi siempre idéntico: la llamada desde el cliente falla, un asistente de IA o un tutorial sugiere agregar `NEXT_PUBLIC_` a la variable de entorno, y de repente funciona. Es fácil interpretar que funcionó porque estaba mal escrito y ahora está bien. Lo que pasó en realidad es que Next.js hizo exactamente lo que ese prefijo significa: publicar el valor en el paquete del navegador. La corrección no es cambiar el nombre de la variable, es cambiar el lugar de la llamada, y eso importa porque uno busca un arreglo de una línea y esto es un arreglo de arquitectura. La forma de que no se te olvide es hacer tú mismo la búsqueda en la pestaña Fuentes y encontrar tu propia clave.

**Confiar en el formato de la respuesta del modelo.** Vas a escribir `JSON.parse(respuesta)` directo, va a funcionar diez veces seguidas, y la número once el modelo va a anteponer una frase amable o a envolver el JSON en un bloque de código y la aplicación se cae. Y vas a reportar que "la API está fallando", cuando la API respondió con un 200 perfectamente válido. La causa profunda es tratar un sistema probabilístico como si fuera una función determinista, y el síntoma es que el error es intermitente, que es la clase de bug más frustrante. Lo que ayuda de verdad es envolver todo acceso a la respuesta en una función de validación que devuelva `null` en vez de reventar, y probarla a mano pasándole basura: un objeto, una cadena vacía, un texto de cinco mil caracteres. Si tu código sobrevive a esas tres, sobrevive al modelo.

**No manejar la latencia, y entonces multiplicar las llamadas.** Pones el botón, no deshabilitas nada, no muestras ningún indicador, y tú mismo haces cuatro clics porque crees que no pasó nada. Son cuatro llamadas pagadas y, peor, cuatro respuestas que van a llegar en orden impredecible y a sobrescribirse entre ellas en la interfaz, así que a veces vas a ver la respuesta de la primera pulsación después de la de la última. Es un problema de concurrencia disfrazado de problema de diseño visual, y esa doble naturaleza es lo importante: el `disabled` mientras hay una petición en curso no es cortesía con el usuario, es la forma más simple de garantizar que solo hay una respuesta posible. La variante grave del mismo error es la llamada dentro de un `useEffect` con dependencias mal declaradas, donde nadie hace clic y aun así hay cientos de peticiones.

**Insertar la salida del modelo con `dangerouslySetInnerHTML` para que "se vea bonita".** Le pides al modelo que use negritas o saltos de línea, la respuesta llega con etiquetas de HTML o con Markdown, y como en pantalla se ven las etiquetas literales buscas cómo interpretarlas. La primera solución que vas a encontrar es la peligrosa, y encima funciona a la primera, que es lo que la hace tan atractiva. "No lo hagas" no es un argumento suficiente, así que ten presente el camino completo del ataque: el jugador escribe algo en el nombre de su personaje, ese nombre entra en el prompt, la respuesta del modelo lleva el contenido activo y el navegador lo ejecuta en el contexto de tu aplicación, con acceso a lo que tu aplicación tenga. La alternativa correcta es tener el formato del lado del código y no del texto: pide al modelo prosa plana, y la negrita, los saltos y el estilo los pone tu componente.

---

## Fuentes de esta sesión

- OWASP. *OWASP Top Ten*. https://owasp.org/www-project-top-ten/
- Prather, J., et al. (2024). The widening gap: The benefits and harms of generative AI for novice programmers. *Proceedings of the 2024 ACM Conference on International Computing Education Research (ICER '24)*, 469-486. https://doi.org/10.1145/3632620.3671116

El Top Ten de OWASP es la referencia con la que hay que leer esta sesión, y vale la pena abrirlo treinta segundos: lo que viste esta noche —secretos expuestos en el cliente y entradas no validadas que llegan a la interfaz— no son curiosidades del ecosistema de la IA, son categorías clásicas de la lista, con un actor nuevo. Que el problema tenga veinte años ayuda a no tratarlo como novedad.

El artículo de Prather y colegas es el que justifica la regla de `IA.md` del curso, y es un estudio con observación directa de estudiantes novatos: documenta que la asistencia generativa amplía la distancia entre quien ya sabe evaluar el código que recibe y quien todavía no, y que el riesgo específico del principiante es aceptar código que funciona sin entender por qué. Esta noche viste el caso perfecto: un asistente que produce, con toda naturalidad, el fallo de seguridad más frecuente de la web. Por eso el curso no evalúa si usaste IA, sino si fuiste capaz de auditarla.

---

## Antes de la sesión 16

Lee los capítulos 3.1 y 3.2 de *Pro Git* —"Ramificaciones en Git: qué es una rama" y "Procedimientos básicos para ramificar y fusionar"—, disponibles en https://git-scm.com/book/en/v2. Veinte minutos, y no hace falta entenderlo todo. Lo que sí quiero es que llegues con una pregunta formulada: si una rama es solo un apuntador a un commit, ¿qué es exactamente lo que Git no puede decidir por sí solo cuando dos ramas cambiaron la misma línea? La respuesta a eso es la mitad de la sesión 16, y la otra mitad es qué se hace —hablando con una persona— cuando aparece.
