# Sesión 5 · Flexbox y Grid

**Módulo 2** · Zona 2: El Taller de Diseño
**Lo que sale de esta noche:** un tablero de ocho por ocho hecho con Grid en menos de quince líneas, y tu primera auditoría cruzada escrita en el Pull Request de un compañero
**Tu misión en curso:** Misión 02 — Tablero de juego adaptable (100 XP · 9 h autónomas), que se entrega antes de la sesión 7
**Esta noche se evalúa:** Auditoría 1 cruzada (25 XP), en el cierre

---

## Por qué elegir bien importa más que saber las propiedades

Flexbox y Grid se pueden aprender de memoria en una tarde de videos, y eso no resuelve el problema. El problema no es no conocer `justify-content`: es intentar maquetar la página entera con Flexbox anidando cinco contenedores, o usar Grid para poner tres botones en fila. En los dos casos vas a pelear contra la herramienta.

Así que el objetivo de la noche es más específico que "aprender los dos": **saber cuál de los dos usar antes de escribir la primera línea, y saber por qué.** La frase que ordena el módulo, y que vale la pena tener a la vista mientras trabajas: **Grid para el esqueleto de la página, Flexbox para lo que va dentro de cada bloque.**

Hay una segunda cosa que esta noche instala y es más profunda que el CSS. Vienes de años de programación imperativa: para acomodar sesenta y cuatro casillas, la intuición dice calcular la posición de cada una. Flexbox y Grid son **declarativos**: describes la forma del contenedor y el navegador calcula las posiciones. Ese cambio de "yo calculo dónde va cada cosa" a "yo describo la estructura y el sistema resuelve" es el mismo salto que vas a hacer en la sesión 7 con los métodos de arreglo en lugar de ciclos, y el mismo de la sesión 12 con React en lugar de manipular el DOM a mano. Es la primera vez que lo ves y es la más fácil de entender de las tres, porque aquí el resultado se ve en pantalla de inmediato.

Un aviso antes de arrancar: esta noche es la **Auditoría 1** y vale 25 XP. Necesitas tu propia rama subida a GitHub para que otro pueda revisarla, y sin rama subida se pierden los puntos. Si no la has subido, hazlo hoy mismo.

---

## Flexbox: una dimensión a la vez

Antes de leer lo que sigue, respondete esto: **¿cómo centrabas una caja dentro de otra antes de esta clase?**

Las respuestas suelen ser un catálogo de la historia del CSS: `margin: 0 auto`, `text-align: center`, `position: absolute` con `top: 50%` y un `transform` para compensar, tres `<br>`, o "buscaba en internet cada vez".

Y ahora el dato que le da peso al tema: **centrar algo vertical y horizontalmente fue durante quince años un problema difícil de verdad en el web, con artículos enteros dedicados a ello.** Hoy son tres líneas.

```css
.pantalla-inicio {
  display: flex;
  justify-content: center;  /* en el eje principal */
  align-items: center;      /* en el eje transversal */
}
```

### La idea central y los dos ejes

**Flexbox reparte espacio a lo largo de una línea.** No de una cuadrícula, no de dos dimensiones: una línea. Todo lo demás son consecuencias de esa decisión.

La analogía que funciona mejor es la fila del banco: hay una fila, hay gente en ella, y lo único que hay que decidir es cómo se distribuye la gente a lo largo de la fila —pegados al frente, pegados al fondo, repartidos con el mismo espacio entre ellos— y cómo se alinean respecto al ancho del pasillo. **Flexbox es el sistema para administrar una fila.**

De ahí sale la distinción donde todo el mundo se pierde la primera vez: los dos ejes. El eje principal es la dirección de la fila, y por defecto es horizontal. El eje transversal es el perpendicular. Entonces `justify-content` trabaja siempre sobre el eje principal y `align-items` siempre sobre el transversal.

La razón por la que esto confunde es que **cuando cambias `flex-direction` a `column`, los dos ejes rotan y las dos propiedades cambian de significado sin cambiar de nombre.** Compruébalo: pon `flex-direction: column` en el ejemplo del centrado y mira cómo `justify-content` ahora manda vertical. La regla mnemotécnica: `justify` va con la dirección, `align` va cruzado, y la dirección la eliges tú.

### Cómo se reparte el espacio sobrante y el faltante

Aquí entran tres propiedades que se escriben en los hijos y no en el contenedor, y esa distinción es la causa de la mitad de los errores del tema.

`flex-grow` dice cuánto del espacio sobrante toma este elemento; con `flex-grow: 1` en todos, se lo reparten en partes iguales, y con `2` en uno de ellos, ese se lleva el doble. `flex-shrink` dice qué tanto está dispuesto a encogerse cuando falta espacio, y `flex-shrink: 0` es la forma de decir "este no se encoge nunca", que es exactamente lo que quieres para un icono o un avatar. `flex-basis` es el tamaño de partida antes de crecer o encoger.

Y `gap`, que hay que celebrar. Antes de que existiera, separar elementos de una fila se hacía con márgenes en los hijos y un truco para quitarle el margen al último, y era feo. Hoy se declara una vez en el contenedor y ya. La conexión con la sesión pasada: **dentro de un contenedor flex los márgenes no colapsan, así que `gap` no es solo más cómodo, es más predecible.**

### El límite de `flex-wrap`, que es la trampa del tema

Con `wrap`, cuando no cabe, los elementos pasan a una línea nueva, y eso parece resolver la maquetación en dos dimensiones. **No la resuelve.** Cada línea calcula sus tamaños de forma independiente, así que los elementos de la segunda fila no se alinean con los de la primera.

Pruébalo con seis cajas de contenido distinto y `wrap` activado. Las columnas quedan desalineadas, y eso no es un error de configuración: es que Flexbox nunca prometió columnas. **Ese desalineamiento es la mejor justificación de por qué existe Grid.**

### Dónde sí es la herramienta correcta

En el contexto de tu juego: la barra de navegación con el logo a la izquierda y los enlaces a la derecha, la fila de botones de control, el panel de puntajes donde el nombre del jugador se estira y el número no, la pantalla de inicio con el título centrado, y la lista de jugadores con su avatar al lado del nombre.

### Las ideas que hay que llevarse

**Flexbox es para una dimensión, y "una dimensión" no significa "horizontal": significa una línea a la vez.** Cuando te descubras peleando por alinear cosas de dos filas distintas, no estás usando mal Flexbox: estás usando la herramienta equivocada.

**El contenedor manda sobre la distribución, los hijos mandan sobre su propia flexibilidad.** `display: flex`, `justify-content`, `align-items`, `gap` y `flex-direction` van en el padre. `flex-grow`, `flex-shrink` y `flex-basis` van en el hijo. Cuando algo no responde, la primera pregunta es si la propiedad está escrita en el elemento correcto, porque una propiedad flex en el elemento equivocado no da error: simplemente no hace nada, y eso vuelve a la lección de la sesión 4 sobre el CSS que falla en silencio.

**El tamaño de un elemento flex no es el que declaraste, es el que negoció el contenedor.** Esto explica el desconcierto de poner `width: 200px` y ver 150 en pantalla: `flex-shrink` está haciendo su trabajo. No es un error, es el sistema funcionando. Y la forma de decir "esto no se toca" es `flex-shrink: 0`.

### Ponte a prueba

*Si le pongo `flex-grow: 1` a los tres hijos, ¿quedan del mismo tamaño?* La respuesta es "no necesariamente", y es una buena trampa: `flex-grow` reparte el espacio **sobrante**, no el total, así que un hijo con más contenido queda más grande. Para tamaños iguales hace falta `flex-basis: 0`. Esta pregunta separa a quien memorizó de quien entendió.

*¿Qué pasa con un contenedor flex si le pongo un solo hijo?* Piénsalo y vas a llegar a que el centrado de tres líneas del comienzo es precisamente ese caso: Flexbox no necesita muchos elementos para ser útil.

---

## Práctica en clase: el tablero de ajedrez, primero a la mala

Esta es probablemente la práctica más memorable del semestre, y funciona por una razón: **la primera mitad se hace mal a propósito, y el dolor es real, no simulado.** No te la saltes por ganar tiempo. Si haces el tablero con Grid de entrada, el resultado es un truco bonito; si lo haces con posiciones absolutas primero, Grid se convierte en un alivio y eso no se olvida.

**Escribe el HTML de las casillas y toma una decisión semántica antes de escribirla.** Vas a necesitar sesenta y cuatro elementos, y lo primero que se le ocurre a cualquiera es una tabla. Aquí se cobra la sesión 2. Con una `<table>` funcionaría visualmente, y el problema es que **un tablero de ajedrez no es información tabular, es una superficie de juego, y marcarlo como tabla es mentirle al lector de pantalla.** Quien navegue con lector de pantalla va a oír "tabla de ocho columnas por ocho filas" y va a intentar leerla celda por celda como si fuera una hoja de cálculo. Es exactamente el mismo error que los `div` de la sesión 2, pero en el otro sentido: allá faltaba semántica, aquí sobra semántica equivocada.

```html
<!-- El tablero es una superficie, no una tabla de datos.
     El rol y la etiqueta le dicen al lector de pantalla
     qué es esto sin prometerle filas y columnas de datos. -->
<div class="tablero" role="grid" aria-label="Tablero de ajedrez">
  <div class="casilla"></div>
  <div class="casilla"></div>
  <!-- ... hasta 64 -->
</div>
```

**Hazlo con posiciones absolutas, en serio y hasta que duela.** Este es el tramo importante. Pon el contenedor en `position: relative` y empieza a colocar casillas con `position: absolute`, `top` y `left` calculados a mano. Escribe las primeras cuatro calculando en voz alta: la casilla de la fila 0 columna 0 va en `top: 0; left: 0`, la de la fila 0 columna 1 va en `left: 60px`, la de la fila 1 columna 0 va en `top: 60px`.

Cuando lleves seis o siete casillas escritas, detente y cuenta cuántas te faltan. Esa cuenta es la mitad del punto. El golpe real es el siguiente: **arrastra el borde de la ventana del navegador para angostarla.** El tablero no hace nada. Sigue midiendo 480 píxeles porque cada casilla tiene su coordenada clavada en píxeles. Y ahora las dos peores preguntas posibles: *¿y si el tablero tiene que medir el 90% del ancho de la pantalla?* Recalcular las sesenta y cuatro coordenadas. *¿Y si el juego pasa a ser de diez por diez?* Se rehace todo.

Deja eso en pantalla un momento sin arreglarlo. **Ese es el estado mental exacto en el que Grid tiene que llegar.**

**Borra todo y hazlo con Grid.** Selecciona el bloque completo de posiciones absolutas y bórralo —el gesto importa— y escribe esto:

```css
.tablero {
  display: grid;
  grid-template-columns: repeat(8, 1fr);
  grid-template-rows: repeat(8, 1fr);
  aspect-ratio: 1;              /* cuadrado siempre, sin calcular nada */
  width: min(90vw, 600px);      /* nunca más de 600, nunca desborda */
  gap: 2px;
}

.casilla:nth-child(odd)  { background: var(--casilla-clara); }
.casilla:nth-child(even) { background: var(--casilla-oscura); }
```

Guarda, angosta la ventana otra vez y mira el tablero adaptarse sin que hayas escrito una sola media query. Aquí está una de las frases que más vale la pena llevarse del semestre: **el código más corto suele ser también el más correcto semánticamente, y no es coincidencia.** Cuando la herramienta corresponde al problema, la solución es breve. Cuando la solución es kilométrica, casi siempre es porque estás forzando una herramienta que no era.

**Encuentra el bug del ajedrez, que es un regalo.** El CSS de arriba tiene un error real y muy famoso: con ocho columnas, `nth-child(odd)` produce **franjas verticales, no un tablero de ajedrez**, porque la paridad se reinicia perfectamente alineada en cada fila. Guarda y míralo. Antes de seguir leyendo, trata de explicar por qué pasa. La pista es que ocho es par.

El arreglo es el patrón de nueve:

```css
/* Con 8 columnas la paridad simple falla: da franjas.
   El patrón que sí alterna en cuadrícula par es cada 9. */
.casilla:nth-child(18n+1),  .casilla:nth-child(18n+3),
.casilla:nth-child(18n+5),  .casilla:nth-child(18n+7),
.casilla:nth-child(18n+10), .casilla:nth-child(18n+12),
.casilla:nth-child(18n+14), .casilla:nth-child(18n+16) {
  background: var(--casilla-clara);
}
```

Y la parte honesta: esto es feo. En el módulo 3, cuando generes las casillas con JavaScript, se resuelve poniéndole una clase a cada casilla al crearla, que es lo que haría cualquiera en un proyecto real. **Reconocer cuándo una solución de CSS puro se está volviendo retorcida es también parte del oficio**, y este es un buen ejemplo de que "se puede hacer con CSS" no siempre significa "se debe hacer con CSS".

**Sube un nivel: la maqueta de la página completa con áreas nombradas.** Sal del tablero y arma la pantalla entera. Aquí es donde se ve para qué sirven las áreas nombradas, porque el CSS se vuelve un dibujo:

```css
.pantalla-juego {
  display: grid;
  grid-template-areas:
    "cabecera cabecera"
    "tablero  puntajes"
    "controles controles";
  grid-template-columns: 1fr 20rem;
  gap: var(--espacio-base);
}

.cabecera  { grid-area: cabecera; }
.tablero   { grid-area: tablero; }
.puntajes  { grid-area: puntajes; }
.controles { grid-area: controles; }
```

Pregúntate qué habría que cambiar para que en el celular todo quede en una sola columna. La respuesta —redefinir `grid-template-areas` con una columna— es exactamente el contenido de la sesión 6, así que déjala pendiente.

**Mete Flexbox dentro de Grid, para cerrar el círculo.** Toma el panel de puntajes, que ya está colocado por Grid, y acomoda su contenido con Flexbox: el nombre del jugador a la izquierda con `flex-grow: 1` y el número a la derecha con `flex-shrink: 0`. **Grid puso el bloque en la página, Flexbox reparte lo que va adentro del bloque. No compitieron en ningún momento.**

**Y si te queda tiempo, la galería que se acomoda sola.** Prueba `repeat(auto-fit, minmax(8rem, 1fr))` sobre una galería de cartas y angosta la ventana. Las columnas cambian de cantidad solas, sin una sola media query. Es el mejor argumento a favor de Grid y el gancho perfecto para la sesión 6.

### Dudas que van a aparecer

**¿Por qué no usar una tabla si funciona?** Es la misma conversación de la sesión 2 y vale la pena cerrarla: una tabla es correcta para datos tabulares y el tablero no es datos tabulares. Y si el argumento semántico no te convence, hay uno práctico: la rúbrica de la Misión 02 da 6 de los 20 XP específicos a que el tablero use Grid y no una tabla ni posiciones absolutas.

**¿Cuál es la diferencia entre `1fr` y `auto`?** `auto` es "el tamaño que necesite el contenido", `1fr` es "una parte del espacio disponible, sin importar el contenido". Para casillas de un tablero quieres `1fr`, porque una casilla con una pieza dentro no debe ser más grande que una vacía. Y una nota: **`fr` es la unidad que no existía antes de Grid**, y es la que hace que las cuadrículas se adapten sin cálculos.

**¿`grid-template-columns` o `grid-column`?** Es el mismo par que en Flexbox: la primera va en el contenedor y define la cuadrícula, la segunda va en el hijo y dice qué celdas ocupa. Cuando algo no responde, revisar en qué elemento está escrita la propiedad sigue siendo la primera pregunta.

**¿Grid reemplaza a Flexbox?** No. Vuelve a la regla del tablero y ponle un ejemplo de tu proyecto a cada lado.

---

## Grid: dos dimensiones y una cuadrícula explícita

### La diferencia de fondo

Ahora que ya lo viste funcionando, la distinción con precisión: **Flexbox coloca elementos a lo largo de una línea y la estructura la determina el contenido. Grid define una cuadrícula primero y después coloca el contenido dentro de ella.** El orden es lo que cambia. En Flexbox, tres elementos producen tres huecos. En Grid, la cuadrícula de ocho por ocho existe aunque no haya nada dentro.

La analogía que lo cierra: **Flexbox es acomodar los muebles de una habitación ya construida; Grid es dibujar los planos de la casa antes de meter los muebles.** Y de ahí sale la regla de la noche sin necesidad de justificarla más: los planos primero, los muebles después.

### Las piezas que hay que dominar

`grid-template-columns` y `grid-template-rows` definen la cuadrícula. `repeat()` evita escribir la misma medida ocho veces y hace el CSS legible, además de que cambiar el tamaño del tablero pasa a ser cambiar un número. La unidad `fr` reparte el espacio disponible después de descontar los tamaños fijos y los `gap`, y es la razón por la que una cuadrícula en `fr` se adapta sin cálculos. `minmax()` pone un piso y un techo a una pista, y es lo que evita que una columna se vuelva ilegible al angostar.

Y la combinación de `repeat()` con `auto-fit` y `minmax()` produce el efecto que más impresiona: **una galería que cambia de cantidad de columnas sola, sin una sola media query.** Aquí conviene la distinción con `auto-fill`, porque la vas a encontrar: `auto-fit` colapsa las pistas vacías y estira las que hay, `auto-fill` las mantiene reservadas. Para una galería casi siempre quieres `auto-fit`.

Las áreas nombradas merecen énfasis aparte, porque tienen un beneficio que no es técnico: **el CSS se vuelve un dibujo de la página que otra persona puede leer.** Cuando alguien abra ese archivo en seis meses, `grid-template-areas` le cuenta la estructura sin tener que abrir el HTML. Eso es documentación que no se desactualiza, porque es el código. Y en la sesión 6 vas a descubrir el otro beneficio: reorganizar la página completa para el celular es reescribir esas tres líneas.

### La trampa de accesibilidad que hay que conocer hoy

Esto es exactamente el tipo de error que se cuela en la Misión 02: **Grid permite colocar los elementos en un orden visual distinto al orden del HTML, y el lector de pantalla y la navegación con Tab siguen el orden del HTML, no el visual.**

Si acomodas los controles arriba del tablero visualmente pero en el marcado van al final, alguien que navegue con teclado va a recorrer la página en un orden que no corresponde a lo que ve. La regla es sencilla: **el orden del HTML es el orden lógico, y Grid solo debería cambiar la presentación, no el sentido.** Es la misma idea de la sesión 2 con una herramienta nueva.

### Las ideas que hay que llevarse

**Grid y Flexbox no compiten: resuelven problemas distintos y se usan juntos todo el tiempo.** La pregunta correcta no es "cuál es mejor" sino "¿estoy acomodando a lo largo de una línea, o estoy definiendo filas y columnas?". Si la respuesta menciona filas **y** columnas, es Grid.

**Si estás anidando tres contenedores de Flexbox para lograr algo que se parece a una cuadrícula, detente: era Grid.** Este síntoma es concreto y verificable, y te sirve como alarma en la Misión 02.

**Lo que se declara es la estructura, no las posiciones.** No hay una sola coordenada en el CSS del tablero de Grid, y esa ausencia es el punto. En la sesión 7 vas a cambiar ciclos por `map` y `filter` por la misma razón, y en el módulo 4 vas a describir interfaces en lugar de manipular el DOM. **Declarar el qué y dejar el cómo al sistema es una idea que se repite en todo el semestre**, y esta noche es la primera vez que la ves con resultado inmediato en pantalla.

### Ponte a prueba

*Si Grid puede hacer una sola fila, ¿por qué existe Flexbox?* Grid puede, y para una barra de navegación con elementos de ancho desconocido Flexbox es más natural, porque no hay que declarar pistas para algo cuyo contenido manda. La respuesta madura es que ninguno reemplaza al otro y que elegir bien ahorra líneas.

*¿Qué pasa si pongo más de sesenta y cuatro casillas en un tablero de ocho por ocho?* Lleva a las filas implícitas: Grid crea pistas nuevas automáticamente. Es útil y también es la causa de un bug misterioso cuando aparece una fila que nadie declaró.

*Si acomodo los controles arriba con Grid pero en el HTML están al final, ¿qué se rompe?* El orden de tabulación, y con él el sentido de la página para quien no la ve.

---

## Cómo trabajar tu Misión 02 después de esta noche

El objetivo concreto para el final de la noche es preciso: **tener el esqueleto de la pantalla de juego colocado con Grid y al menos un bloque interno acomodado con Flexbox.** No estilos finos, no colores bonitos: la estructura.

Y una instrucción de arranque que ahorra más tiempo del que cuesta: **antes de escribir CSS, dibuja la maqueta en papel y escribe los nombres de las áreas de Grid al lado.** Cabecera, tablero, puntajes, controles. Cinco minutos de papel. Quien empieza a escribir CSS sin haber decidido la estructura termina con seis contenedores de Flexbox anidados, y arreglar eso cuesta más que planearlo.

Mientras trabajas, hazte estas tres preguntas: *¿esto es una línea o es una cuadrícula?*, *¿esa propiedad va en el contenedor o en el hijo?* y *¿qué pasa si angosto la ventana ahora?*. La tercera es la más productiva de todas porque encuentra los problemas antes de que los encuentre la calificación.

Si vas adelantado, prueba `repeat(auto-fit, minmax())` en la zona de jugadores, o ve directo al tema oscuro con variables de la misión secundaria de 25 XP.

Y no olvides subir tu rama antes del cierre de la noche, porque sin rama subida no hay 25 XP de auditoría.

---

## Tu auditoría de esta noche: Auditoría 1 cruzada (25 XP)

Esta es la primera de las cuatro auditorías cruzadas del semestre, y lo que se califica no es evidente, así que lee con atención.

Te va a tocar asignado el repositorio de un compañero. Abres el Pull Request de su Misión 02 y dejas **al menos dos comentarios sustantivos en el PR mismo**, no en un archivo aparte ni por WhatsApp. El comentario en el PR es el artefacto: es donde ocurre la revisión de código en la industria y es lo que se lee al calificar.

Y aquí está la parte que cambia todo: **lo que se califica es la calidad de TU revisión, no la calidad del código revisado.** Te puede tocar un proyecto excelente y sacar los 25 XP completos, y te puede tocar un proyecto desastroso y sacar cero. Son dos trabajos independientes.

Un comentario que dice "está bien, buen trabajo" **vale cero**. Un comentario que dice "el tablero se desborda por debajo de 380 píxeles de ancho porque el `width` está en píxeles fijos; con `min(90vw, 600px)` se resuelve" vale la auditoría completa, porque identificó un caso borde que el autor no cubrió, explicó la causa y propuso una corrección concreta.

Las preguntas que tu comentario tiene que responder son las de la rúbrica del módulo: si el tablero usa Grid o si usa una tabla o posiciones absolutas; si hay valores repetidos que debían ser variables; si al angostar la ventana algo se desborda o se vuelve ilegible; si hay Flexbox anidado donde correspondía Grid; y una cosa que el revisado hizo bien, dicha con concreción y no como cortesía.

Los 25 XP se reparten así: identificar al menos un problema real y explicar por qué es un problema (10), proponer una corrección concreta y no genérica (8), el tono —crítica al código y nunca a la persona (4), y reconocer algo bien hecho de forma específica (3).

Y hay un incentivo: dos reconocimientos por haber ayudado de verdad en las auditorías habilitan la insignia 🤝 **Buen colega**, que descarta el peor quiz del semestre.

---

## Errores que probablemente vas a cometer

**Elegir la herramienta por costumbre y no por la forma del problema.** El patrón es este: aprendes Flexbox primero, funciona para todo lo que has intentado, y entonces intentas maquetar la página entera con contenedores flex anidados. El resultado se ve casi bien y es imposible de mantener, porque cada ajuste en un nivel desacomoda los otros dos, y sobre todo porque las cosas de filas distintas nunca se alinean —Flexbox nunca prometió alinearlas—. El síntoma de alarma que puedes reconocer solo es el anidamiento: **si vas por el tercer contenedor flex dentro de otro y sigues persiguiendo una alineación, era Grid.** La otra cara del mismo error también aparece: usar Grid para poner tres botones en fila, declarando pistas para algo cuyo tamaño lo manda el contenido.

**Confundir en qué elemento va cada propiedad, y no darte cuenta porque el CSS no avisa.** Escribes `justify-content` en el hijo o `flex-grow` en el contenedor, no pasa absolutamente nada, y concluyes que la propiedad "no funciona" o que el navegador no la soporta. Es el mismo problema de la sesión 4 con otra cara: una propiedad inválida para ese contexto se ignora en silencio. La costumbre que hay que instalar es mirar el panel de estilos, donde el navegador marca las propiedades que no está aplicando, y la regla mental es corta: **lo que reparte va en el contenedor, lo que se deja repartir va en el hijo.**

**Poner tamaños fijos en píxeles a los hijos de una cuadrícula, y anular con eso todo el beneficio.** Escribes `display: grid` con `repeat(8, 1fr)` —perfecto— y después le pones `width: 60px; height: 60px` a la casilla. La cuadrícula queda adaptable y el contenido no, así que el tablero se desborda en el celular o deja huecos en el escritorio, y la conclusión fácil es que Grid "no sirvió". Lo que hay que entender es que en una cuadrícula el tamaño lo negocia la pista, no el hijo, y que para mantener las casillas cuadradas existe `aspect-ratio`, que no requiere calcular nada. Este error cuesta puntos directamente en la Misión 02.

**Usar Grid para reordenar visualmente y romper el orden de lectura sin notarlo.** Es el más silencioso de los cuatro y el más difícil de detectar, porque en pantalla todo se ve correcto. Mueves los controles arriba del tablero con `grid-template-areas` mientras en el HTML siguen al final del documento, y quien navegue con Tab va a recorrer la interfaz saltando en un orden que no corresponde a nada de lo que ve. Es exactamente el mismo tipo de fallo de la sesión 2: invisible al ojo, evidente a la máquina. La comprobación cuesta quince segundos y conviene volverla rutina: **guarda el ratón y recorre la pantalla con Tab.** Si el foco salta de forma incoherente, el problema no es el CSS: es que el orden lógico del documento quedó mal y hay que arreglarlo en el HTML.

---

## Fuentes de esta sesión

- W3C. *CSS Flexible Box Layout Module Level 1*. https://www.w3.org/TR/css-flexbox-1/
- W3C. *CSS Grid Layout Module Level 1*. https://www.w3.org/TR/css-grid-1/
- MDN Web Docs. *Basic concepts of flexbox*. https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Flexible_box_layout/Basic_concepts
- MDN Web Docs. *CSS grid layout*. https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Grid_layout

Las dos especificaciones del W3C son la autoridad y no son lectura de estudio: sirven para resolver una duda puntual sobre cómo se calcula un tamaño cuando la documentación divulgativa se contradice. Las dos guías de MDN sí son de uso diario, y conviene tenerlas en marcadores desde esta noche, porque vas a volver a ellas cada vez que necesites recordar qué eje maneja cada propiedad.

Recuerda también el plazo de la Misión 02: antes de la medianoche del día anterior a la sesión 7, en sus dos versiones, con el presupuesto de **9 horas autónomas** repartidas en una de planear, tres de CSS a mano, tres de Tailwind, una de probar y una de comparar. La versión de Tailwind se hace después de la sesión 6, así que esta semana es la de CSS a mano.

---

## Antes de la sesión 6

Lee la sección "Módulo 2, sesión 6" de `GUIA-DEL-CURSO.md` y la página de Tailwind sobre diseño adaptable, solo la parte de los prefijos de punto de quiebre. Diez minutos, sin instalar nada.

Y llega con esta pregunta contestada, porque con ella abre la sesión y porque la respuesta intuitiva es la equivocada: *¿por qué diseñar primero para el celular y después para el escritorio, y no al contrario, que parece más natural porque tú trabajas en un computador?* Escríbelo en dos líneas.

La sesión 6 empieza con **Quiz 2**: 15 XP, tres preguntas, en los primeros diez minutos, sobre la cascada, la caja, Flexbox y Grid. Preguntas de aplicación y no de definiciones.
