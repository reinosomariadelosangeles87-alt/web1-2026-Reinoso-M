# Sesión 12 · React: componentes, props y estado

**Módulo 4** · Zona 4: La Fortaleza de Componentes
**Lo que sale de esta noche:** tu primer componente propio funcionando y una respuesta escrita a la pregunta "¿dónde vive este estado?"
**Tu misión en curso:** Misión 06 — Mazmorra por turnos en React (100 XP · 10 h de trabajo autónomo), que se entrega en la sesión 14

---

## Por qué React aparece justo ahora

Vienes de la Misión 03, el juego de memoria, escrito con JavaScript puro y manipulación directa del DOM. Si te pasó lo que le pasó a casi todo el mundo, en algún momento tuviste el bug de las tres cartas volteadas: la regla decía máximo dos y de pronto había tres, o una carta quedaba volteada para siempre, o el contador decía dos y en pantalla había una.

Ese bug casi siempre existe por la misma razón. El estado del juego estaba repartido entre variables sueltas, clases de CSS en los elementos del DOM y a veces un arreglo aparte, y en algún momento esas versiones de la verdad dejaron de coincidir. Busca en tu propio código de la Misión 03 las líneas donde le preguntabas a la pantalla en qué estado estaba el juego: algo como `if (carta.classList.contains('volteada'))` o `document.querySelectorAll('.volteada').length === 2`.

Ahora cuenta cuántas fuentes de verdad hay en ese archivo sobre cuántas cartas están volteadas. Casi siempre son dos o tres. Ese es el problema entero del módulo 3, y es exactamente el que React resuelve.

Entonces esta noche no vas a aprender una librería. Vas a aprender una regla: **un solo lugar guarda la verdad, y la pantalla es un reflejo de ese lugar.** El DOM miente; el estado no. Y el objetivo real de la sesión es más estrecho de lo que suena el temario: que salgas de aquí capaz de mirar una interfaz y decidir **dónde tiene que vivir cada pedazo de estado**. Todo lo demás de React se aprende leyendo documentación. Eso no.

Esta noche son cuatro cosas y ninguna más: componentes, props, estado y listas. No vamos a tocar `useEffect` ni Next.js todavía.

---

## El estado es la verdad y la interfaz es su consecuencia

Antes de leer lo que sigue, respondete esto: **con el código que ya escribiste en la Misión 03, si tienes que cambiar el número de vidas del jugador de 3 a 2, ¿qué pasos tienes que ejecutar?**

La secuencia real es más o menos esta: modificas la variable, buscas el elemento del DOM que muestra las vidas, le cambias el texto, y si las vidas llegaron a cero también deshabilitas los botones y muestras la pantalla de fin de partida. Cuatro o cinco pasos, todos manuales, todos con la posibilidad de que olvides uno.

La frase que resume React es esta: **cambias el dato y describes cómo se ve la interfaz para cada dato posible; nunca ejecutas los pasos intermedios.** Cambias el 3 por el 2 y la pantalla se acomoda sola. La diferencia con lo que venías haciendo no es de sintaxis: es de responsabilidad. Tú declaras el "qué", React se encarga del "cómo".

### El componente es una función que devuelve una descripción

Un componente de React es una función de JavaScript que recibe un objeto de datos y devuelve una descripción de lo que debería aparecer en pantalla. No devuelve elementos del DOM: devuelve una descripción de elementos. La distinción parece pedante y es la clave de todo lo demás.

```jsx
// Un componente es una función. Recibe datos, devuelve descripción.
// Por convención el nombre empieza en mayúscula: así React sabe
// que es un componente tuyo y no una etiqueta HTML.
function BarraDeVida({ actual, maximo }) {
  const porcentaje = (actual / maximo) * 100;
  return (
    <div className="barra">
      <div className="relleno" style={{ width: `${porcentaje}%` }} />
      <span>{actual} / {maximo}</span>
    </div>
  );
}
```

Hay tres cosas en ese bloque que conviene mirar despacio.

La primera es que **eso que parece HTML no es HTML: es JSX**, y el navegador no lo entiende. Una herramienta de compilación lo traduce a llamadas de función antes de que el navegador lo vea.

```javascript
// Esto que escribes...
<span className="vidas">{actual}</span>

// ...se compila aproximadamente a esto:
React.createElement("span", { className: "vidas" }, actual);

// Que en tiempo de ejecución produce un objeto plano, algo como:
// { type: "span", props: { className: "vidas", children: 3 } }
```

Esa última línea es la que hace clic. **Un componente devuelve objetos de JavaScript, no nodos del navegador.** Cuando aceptas eso, el DOM virtual deja de ser magia.

La segunda es mecánica: se escribe `className` y no `class`, porque `class` es palabra reservada de JavaScript y JSX es JavaScript, y las llaves son la puerta para entrar en modo JavaScript dentro del marcado. Lo que sí importa de verdad es que dentro de las llaves va una **expresión**, no una instrucción. Puedes poner una suma, una llamada a función o un ternario; no puedes poner un `if` ni un `for`. Esa restricción es la que explica por qué el renderizado condicional se escribe como se escribe, y lo vas a ver en la práctica.

La tercera es que la función tiene que ser **pura** respecto de lo que dibuja: con los mismos datos de entrada debe devolver la misma descripción, y no debe modificar nada de fuera mientras lo hace. Un componente que dentro del cuerpo hace `document.title = "..."` o empuja algo a un arreglo global está mintiendo sobre lo que es, y React se reserva el derecho de llamarlo dos veces para delatarlo. Por la misma razón, si pones un `Math.random()` en el cuerpo, tu componente deja de ser puro y va a dibujar distinto en cada render sin que ningún dato haya cambiado. La aleatoriedad y las llamadas a red no van ahí.

### El DOM virtual y la reconciliación, sin misticismo

El ciclo son tres movimientos. Cuando un dato cambia, React vuelve a llamar a tu componente y obtiene una descripción nueva del árbol. Ahora tiene dos árboles de objetos planos: el de antes y el de ahora. Los compara —eso es la **reconciliación**— y calcula el conjunto mínimo de operaciones reales sobre el DOM que hacen falta para que el navegador refleje el árbol nuevo. Si en un tablero de sesenta y cuatro casillas cambió el contenido de una, React toca una sola casilla.

Piénsalo como el control de versiones que ya usas todos los días. Git no manda el archivo completo cada vez; calcula el diff y aplica el diff. React hace lo mismo, y por la misma razón económica: comparar dos objetos en memoria es baratísimo, tocar el DOM es caro. Por eso puede permitirse el lujo conceptual de "volver a dibujar todo" y seguir siendo rápido.

Un matiz que previene una confusión clásica: **"volver a renderizar" no significa "volver a pintar en pantalla".** Renderizar es ejecutar tu función y producir la descripción. Pintar es lo que hace el navegador después, y solo con lo que efectivamente cambió. Cuando en la sesión 13 hablemos de rendimiento, esta distinción es la que te va a evitar optimizar fantasmas.

### Las ideas que hay que llevarse

**El estado es la única fuente de verdad, y la pantalla es una función de él.** La fórmula es `interfaz = f(estado)`, y toda la arquitectura de React sale de tomarse esa igualdad en serio. Si la pantalla dice algo que el estado no dice, tienes un bug; y al contrario, si nunca lees la pantalla para tomar decisiones, ese bug es imposible.

**Los datos bajan por props y las notificaciones suben por funciones.** Las props son de solo lectura: un componente hijo no puede modificar lo que recibió, igual que una función no reescribe los argumentos de quien la llamó. Si el hijo necesita provocar un cambio, el padre le pasa una función y el hijo la invoca. Las props son un memorando que baja por el organigrama, y las funciones de retorno son el formulario que el subalterno llena y devuelve. El memorando no se corrige con lápiz; se pide uno nuevo.

**El estado vive en el ancestro común más cercano de todos los componentes que lo necesitan, y ni un nivel más arriba.** Esta es la frase de la noche. Un nivel más abajo y vas a tener que duplicarlo, con lo cual vuelves a tener dos verdades. Un nivel más arriba y renderizas de más y arrastras props por componentes a los que no les interesa. En la Misión 06 esta decisión vale 6 de los 20 XP específicos del módulo, y es lo primero que se va a mirar al calificar.

### Ponte a prueba

*Si un componente devuelve objetos y no elementos del DOM, ¿podría React dibujar en algo que no sea un navegador?* Sí, y esa es exactamente la razón de que exista React Native. Si esa respuesta te parece obvia, entendiste la separación entre describir y pintar.

*En el juego de memoria, el estado "esta carta está volteada", ¿debería vivir en la carta o en el tablero?* En el tablero, porque la regla de "máximo dos volteadas" necesita ver todas las cartas a la vez. Es el ejemplo perfecto de por qué el estado sube: la regla de negocio determina el nivel.

*¿Qué pasa si dentro del cuerpo de un componente escribes `Math.random()`?* Que el componente deja de ser puro y va a dibujar distinto en cada render sin que ningún dato haya cambiado.

---

## Práctica en clase: construir el HUD de la mazmorra

Vas a construir el panel de estado —el HUD— de la mazmorra que entregas en la Misión 06. El recorrido está diseñado para arrancar de un componente único y horrible y refactorizarlo hasta que la arquitectura aparezca sola. Lo hacemos juntos en el salón, y queda escrito aquí para que puedas repetirlo por tu cuenta.

**Primero, arranca el proyecto.** Vite es lo más rápido y no introduce todavía ningún concepto de Next.js.

```bash
npm create vite@latest practica-06-mazmorra -- --template react-ts
cd practica-06-mazmorra
npm install
npm run dev
```

Borra el contenido de ejemplo de `App.tsx` sin ceremonia. Empezar de cero está permitido.

**Segundo, escribe el componente monolítico.** Todo dentro, con datos fijos. Feo y a propósito.

```tsx
function App() {
  // Datos fijos por ahora: todavía no hay estado.
  const jugador = { nombre: "Kael", vida: 18, vidaMaxima: 25, oro: 40 };
  const inventario = ["Espada oxidada", "Poción menor", "Llave de hierro"];

  return (
    <main>
      <h1>Mazmorra de Caldas</h1>
      <p>{jugador.nombre}</p>
      <p>{jugador.vida} / {jugador.vidaMaxima}</p>
      <p>Oro: {jugador.oro}</p>
      <ul>
        <li>{inventario[0]}</li>
        <li>{inventario[1]}</li>
        <li>{inventario[2]}</li>
      </ul>
    </main>
  );
}
```

Antes de arreglarlo, mira qué está mal. Lo más evidente es que el inventario está escrito a mano, elemento por elemento: si mañana hay cinco objetos, hay que editar el JSX. Eso es lo que motiva las listas.

**Tercero, la lista con `map`, y omite la clave a propósito.** Reemplaza los tres `li` por un `map` sin `key`, deja la aplicación corriendo y abre la consola del navegador. Prueba esto y observa qué pasa: vas a ver la advertencia *"Each child in a list should have a unique key prop"*. Vale la pena provocarla una vez, porque ese mensaje lo vas a volver a ver el resto de tu vida profesional.

```tsx
<ul>
  {inventario.map((objeto) => (
    <li>{objeto}</li>  // falta key: React se queja en consola
  ))}
</ul>
```

**Cuarto, cae en la trampa del índice como clave y mírala fallar.** "Arréglalo" con `key={indice}`. La advertencia desaparece. El problema no. Para verlo, convierte el inventario en estado, agrega un campo de cantidad por cada objeto y un botón que borre el primer elemento del arreglo.

```tsx
const [inventario, setInventario] = useState([
  { id: "esp-01", nombre: "Espada oxidada", cantidad: 1 },
  { id: "poc-01", nombre: "Poción menor", cantidad: 3 },
  { id: "lla-01", nombre: "Llave de hierro", cantidad: 1 },
]);

// Versión ROTA a propósito: la clave es la posición, no el objeto.
{inventario.map((objeto, indice) => (
  <li key={indice}>
    {objeto.nombre}
    <input type="number" defaultValue={objeto.cantidad} />
  </li>
))}
```

Escribe un valor distinto en cada campo y borra el primer objeto. **Los valores que quedan en los campos no corresponden a los objetos que quedan.** Eso es todo lo que necesitas ver.

Lo que acaba de pasar es esto: la clave es la promesa que le haces a React sobre la identidad de cada elemento. Con `key={0}` le dijiste "este es el elemento cero". Al borrar el primero, la poción pasa a ser el elemento cero, React cree que es el mismo de antes y le conserva el estado interno del DOM —el texto del input— aunque el contenido cambió. Es como identificar a las personas por el puesto en que se sentaron en vez de por su cédula: si alguien se va, todos los demás cambian de identidad. Arréglalo con `key={objeto.id}` y repite la operación. Ahora sí.

**Quinto, sube el estado.** Extrae `BarraDeVida`, `Inventario` y `PanelDeJugador` a componentes propios. El estado del jugador se queda en `App` y baja por props. Escribe las firmas con tipos, que ya los traes del módulo 3.

```tsx
type Objeto = { id: string; nombre: string; cantidad: number };

type PanelProps = {
  nombre: string;
  vida: number;
  vidaMaxima: number;
  inventario: Objeto[];
  onUsarObjeto: (id: string) => void;  // la notificación sube
};
```

**Sexto, prueba a mutar una prop y observa qué pasa.** Dentro de `Inventario`, intenta `props.inventario.push(...)` o `inventario.sort()`. La pantalla no se actualiza, o se actualiza a destiempo y mal. Es el error conceptual más caro del módulo: mutar en lugar de reemplazar. Corrígelo devolviendo un arreglo nuevo con `map` o `filter`. No es un tema nuevo: es la inmutabilidad de la sesión 7, ahora con consecuencias visibles.

**Séptimo, el renderizado condicional y su trampa.** Hay tres formas y conviene tenerlas las tres en pantalla al mismo tiempo.

```tsx
{vida <= 0 && <PantallaDeDerrota />}          {/* mostrar o nada */}
{vida > 5 ? <Normal /> : <Advertencia />}      {/* uno u otro */}
{objetos.length === 0 && <p>Mochila vacía</p>} {/* CUIDADO */}
```

Cambia `objetos.length === 0` por `objetos.length &&` y mira la pantalla: aparece un **cero suelto pintado ahí**. `&&` devuelve el valor de la izquierda cuando es falso, y `0` es falso pero también es algo que se puede dibujar. Es un bug que vas a cometer, y que se diagnostica en dos segundos si ya lo viste una vez.

Cuatro dudas que aparecen siempre en este taller y conviene resolver de una vez. Un componente devuelve un solo valor porque es una función, y `<>...</>` es la forma de agrupar hermanos sin ensuciar el DOM con un `div` de más. `class` no funciona porque JSX es JavaScript. `onClick={manejar}` pasa la función y `onClick={manejar()}` la ejecuta durante el render y pasa su resultado, así que si dentro había un `setState` acabas de crear un ciclo infinito; probablemente lo logres esta noche, y cuando pase ya sabes qué fue. Y si te preguntas cómo se guarda el progreso o cómo se llama a una API, eso es la sesión 13 y la 15: anótalo como deuda y no lo improvises hoy.

**Antes de seguir programando en tu misión, escribe la arquitectura.** En el `README.md` de `practica-06-mazmorra/` va el árbol de componentes y una tabla de estado con tres columnas: el nombre del dato, qué componentes lo necesitan, y dónde decidiste ponerlo. Quien empieza a programar sin eso rehace todo el sábado. Lo que debería quedarte hecho esta noche, en orden, es el proyecto creado y corriendo, el árbol escrito, la cuadrícula de la mazmorra dibujada con `map` anidado y claves estables, y el HUD mostrando datos fijos. El movimiento del personaje y el combate son para la casa; hoy lo que importa es la estructura.

---

## Dónde vive el estado

Este es el bloque corto en contenido y decisivo en consecuencias.

El estado en React tiene tres propiedades. Es **local a la instancia del componente**: dos `BarraDeVida` en pantalla tienen dos estados independientes, igual que dos objetos de la misma clase tienen atributos independientes. Es **privado**: nadie de fuera puede leerlo ni escribirlo. Y es **persistente entre renders**: React lo guarda por ti, asociado a la posición de ese componente en el árbol. De esa última propiedad sale algo que sorprende la primera vez: si un componente desaparece del árbol y después vuelve, su estado se perdió.

De ahí sale el procedimiento para decidir dónde va cada cosa, y conviene tenerlo como procedimiento porque es lo que vas a necesitar el sábado a las once de la noche.

Primero, lista qué datos cambian con el tiempo. En la mazmorra: posición del jugador, vida, inventario, enemigos vivos, turno actual, mensaje del registro de combate. Segundo, por cada dato, busca todos los componentes que lo necesitan para dibujarse. Tercero, encuentra el ancestro común más cercano de ese conjunto: ahí va el estado. Y cuarto, que es el paso que casi nadie hace, pregúntate si el dato es **derivado**: si se puede calcular a partir de otro estado, no es estado.

Ese cuarto punto merece su propio ejemplo, porque es el error que más veces aparece en las entregas.

```tsx
// MAL: dos estados que pueden contradecirse.
const [enemigos, setEnemigos] = useState(listaInicial);
const [enemigosVivos, setEnemigosVivos] = useState(listaInicial.length);
// Si alguna vez actualizas uno y olvidas el otro, la interfaz miente.

// BIEN: uno es la verdad, el otro se calcula.
const [enemigos, setEnemigos] = useState(listaInicial);
const enemigosVivos = enemigos.filter((e) => e.vida > 0).length;
// Imposible que se desincronicen: no hay dos cosas que sincronizar.
```

La regla es corta: **si puedes calcularlo, no lo guardes.** Cada pedazo de estado redundante es una oportunidad futura de que la interfaz mienta, y ya sabes a qué se parece una interfaz que miente.

El caso contrario también existe. Cuando el ancestro común queda muy arriba y el dato tiene que atravesar cinco componentes que no lo usan, eso se llama *prop drilling* y es una molestia real. Tiene solución —Context, y librerías de estado— y **para la mazmorra de la Misión 06 no la necesitas**. Meter Context ahora sería resolver un problema que todavía no tienes, y eso hace más daño que bien.

Así se lee la arquitectura de la mazmorra, para que la tengas de referencia: `App` guarda el estado del juego completo; `Tablero` recibe la cuadrícula y una función para mover; `Casilla` recibe qué hay en ella y no tiene estado propio; `PanelDeJugador` recibe vida e inventario; `RegistroDeCombate` recibe un arreglo de mensajes. Ni un solo componente hoja necesita estado, y eso está bien. Un componente sin estado es un componente que no puede desincronizarse.

Un detalle que parece contradecir lo del taller y no lo contradice: en la cuadrícula, una clave como `key={fila + "-" + columna}` está perfectamente bien, porque en una cuadrícula fija la posición **sí es** la identidad. El índice es mala clave cuando el orden puede cambiar, no siempre.

### Las ideas que hay que llevarse

**El estado vive lo más arriba que sea necesario y ni un nivel más.** Las dos mitades de esa frase pesan igual. Cuando te encuentres con un componente inflado de estado, la pregunta útil no es "¿está mal?" sino "¿quién más necesita este dato?"; si la respuesta es "solo este componente", el dato tiene que bajar.

**Si puedes calcularlo, no lo guardes.** Estado derivado es estado duplicado con otro nombre, y estado duplicado es el bug de la Misión 03 esperando a volver.

**Un componente sin estado no puede desincronizarse.** Que tus componentes hoja no tengan nada propio no es pobreza de diseño: es la garantía de que no hay dos verdades peleando.

### Ponte a prueba

*El registro de combate se muestra en un panel lateral y también en un resumen al final de la partida. ¿Dónde vive el arreglo de mensajes?* En el ancestro común más cercano de los dos, que en la estructura de arriba es `App`.

*Tienes `inventario` en estado y quieres mostrar el peso total de la mochila. ¿`useState` o cálculo?* Cálculo, siempre. El peso se deriva del inventario.

*Si borras un componente del árbol y lo vuelves a montar, ¿qué pasa con su estado?* Se perdió. React lo guardaba asociado a esa posición del árbol, y esa posición dejó de existir.

---

## Errores que probablemente vas a cometer

**Tratar el estado como una variable y mutarlo directamente.** Vas a escribir `inventario.push(objeto)` o `jugador.vida = jugador.vida - 5` y te vas a desesperar porque la pantalla no cambia. La causa es que React compara la referencia del objeto para decidir si algo cambió, y una mutación no cambia la referencia: es el mismo arreglo con contenido distinto, y React no tiene forma de notarlo. La corrección es siempre producir un valor nuevo con `map`, `filter` o propagación. La imagen que ayuda: no editas el estado, escribes uno nuevo encima. Y esto no es una rareza de React, es la inmutabilidad de la sesión 7 con consecuencias que ahora se ven.

**Usar el índice como clave por costumbre.** Lo vas a hacer porque hace desaparecer la advertencia de la consola, que es lo que uno interpreta como "arreglado". El daño no aparece hasta que la lista se reordena, se filtra o se le borra un elemento del medio, y entonces el bug es de los peores: el estado interno de los componentes queda pegado a la posición equivocada, la interfaz muestra datos cruzados y nada de eso apunta a la clave. La regla operativa es sencilla: si la lista puede cambiar de orden o de tamaño, necesitas un identificador que venga del dato, y si el dato no lo tiene, genéralo cuando lo creas y no cuando lo dibujas.

**Duplicar estado en lugar de derivarlo.** Vas a guardar `enemigos` y también `cantidadDeEnemigos`, o `inventario` y también `pesoTotal`, y tarde o temprano vas a actualizar uno y olvidar el otro. Es exactamente el mismo error del juego de memoria, migrado a React: dos fuentes de verdad que se contradicen. La defensa concreta es justificar por escrito cada `useState` en la tabla de estado del `README.md`, porque en el momento de escribir "este dato se puede calcular a partir de aquel" el problema se ve solo.

**Poner todo el estado en el componente raíz "por si acaso".** Es la sobrecorrección típica después de esta clase. Un `App` con mil líneas y veinte `useState` funciona, pero cada cambio vuelve a renderizar el árbol completo, cada componente hijo recibe quince props de las que usa dos, y nadie puede leer el archivo, empezando por ti dentro de dos semanas. La regla es "lo más arriba que sea necesario", y la segunda mitad no es decorativa.

**Sobre la IA en este tema.** El asistente está permitido y genera componentes de React con mucha soltura. Lo que genera mal, casi siempre, es precisamente la ubicación del estado: tiende a poner estado local en cada componente porque escribe cada archivo aislado. Si le pides la mazmorra completa de un tirón, vas a recibir cuatro componentes con estado duplicado que se desincroniza. Todo lo que generes va declarado en `IA.md`; sin declarar, la entrega se califica en 0 y no hay reintento. Declararlo no baja la nota, porque lo que se evalúa es tu capacidad de auditarlo, y encontrar exactamente este fallo vale la insignia 🔍 **Cazador de alucinaciones**.

---

## Fuentes de esta sesión

- Meta. *React: Quick Start*. https://react.dev/learn
- Meta. *Built-in React Hooks*. https://react.dev/reference/react/hooks

La documentación de React se reescribió completa para enseñar precisamente el enfoque de esta sesión —pensar en estado y no en manipulación del DOM—, así que es una fuente pedagógica y no solo una referencia de API. Los ejemplos interactivos de `react.dev/learn` valen más que cualquier tutorial de video, porque se pueden romper y volver a arreglar ahí mismo en el navegador.

---

## Antes de la sesión 13

Lee la página de `useState` y la de `useEffect` en la referencia de hooks, solo la sección de descripción de cada una, sin los ejemplos avanzados. Diez minutos. La idea no es que llegues sabiendo usarlos, sino que llegues habiendo visto la firma y con una pregunta ya formulada en la cabeza: *¿por qué esto necesita un arreglo al final?* Ese arreglo es el tema central de la próxima sesión.

Y ten presente lo administrativo: la semana entrante hay **Auditoría 3** cruzada, 25 XP, sobre el código de la mazmorra de un compañero. Sube tu rama aunque esté incompleta, porque sin código subido no hay nada que auditar y esos 25 puntos se pierden.
