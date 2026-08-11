# Sesión 4 · La cascada, el modelo de caja y la especificidad

**Módulo 2** · Zona 2: El Taller de Diseño
**Lo que sale de esta noche:** tu ficha de personaje con estilos, una paleta declarada en variables, y la capacidad de explicar por qué una regla CSS no se está aplicando sin recurrir a `!important`
**Tu misión en curso:** Misión 02 — Tablero de juego adaptable (100 XP · 9 h autónomas), que se entrega antes de la sesión 7

---

## Por qué esta noche define el módulo entero

Llevas tres semanas sin CSS, con la ficha de personaje fea a propósito, y llevas tres semanas queriendo que se vea bien. Esa ansiedad acumulada es tu mejor recurso de la noche y la vas a gastar entera en la práctica. Pero antes hay algo que decidir sobre cómo vas a trabajar el CSS el resto del semestre.

Hay dos formas de relacionarse con esta tecnología. Una es tratarla como una colección de trucos que se prueban hasta que algo se ve bien: escribes una propiedad, no pasa nada, escribes otra, tampoco, agregas `!important`, funciona, sigues adelante y no aprendiste nada. Tres semanas después tienes un archivo de mil líneas donde cada cambio rompe dos cosas en otra parte, y para ese punto la única salida es empezar de nuevo.

La otra es tratarla como **un sistema de reglas con resultados predecibles**. Esa es la diferencia entre alguien que puede depurar un estilo y alguien que solo puede pelear con él, y es lo que esta noche te va a dar.

Hay una segunda razón para tomárselo en serio: **la cascada es el primer sistema de resolución de conflictos que estudias en la carrera, y no es el último.** La misma forma de pensar —hay reglas que compiten, hay un orden de desempate fijo, y si no lo conoces el resultado te parece aleatorio— reaparece en las precedencias de operadores, en las reglas de resolución de nombres de un lenguaje, en la propagación de eventos del módulo 3 y en las reglas de enrutamiento del módulo 4. Aprender a leer un desempate documentado en lugar de adivinar es la competencia real de esta sesión.

---

## Por qué tu regla no se aplica

Antes de leer lo que sigue, sé honesto contigo mismo: **cuando escribes una regla de CSS, no pasa nada, y no entiendes por qué, ¿qué haces?**

La respuesta sincera suele ser alguna combinación de probar otra cosa, mover la regla más abajo, agregarle `!important`, borrarla y volverla a escribir, o recargar con caché limpio.

Todas esas son formas de adivinar, y no hace falta adivinar, porque **la respuesta está siempre en una de cuatro cosas y esas cuatro cosas tienen un orden fijo.**

La analogía que lo aclara es el desempate de una liga de fútbol. Cuando dos equipos terminan con los mismos puntos, nadie tira una moneda: hay un orden de criterios escrito de antemano —puntos, diferencia de gol, goles a favor, enfrentamiento directo— y se aplican en ese orden hasta que uno gana. La cascada es exactamente eso para reglas de CSS que quieren pintar la misma propiedad del mismo elemento. **No hay azar, hay un desempate documentado**, y lo que sigue es leerlo.

### Primero, el origen

Los estilos no vienen todos del mismo lado. El navegador trae los suyos, que son la razón por la que un `<h1>` sin CSS ya se ve grande y un enlace ya se ve azul y subrayado. Encima van los del autor, o sea los tuyos. Y hay un tercer origen que casi nadie considera: los del usuario, que alguien configura en su navegador porque necesita todo más grande o con más contraste.

En condiciones normales los del autor ganan a los del navegador, y por eso tu CSS funciona. La parte que sorprende: **cuando el usuario declara algo con `!important`, gana incluso a los estilos del autor.** Ese caso existe precisamente para que una persona con baja visión pueda forzar el tamaño de letra que necesita sin que un diseñador se lo impida. Es la primera pista de la noche de que `!important` no es "el nivel más alto" sino una palanca con reglas propias.

### Segundo, la especificidad

Aquí está el 80% de la confusión del tema, así que despacio. El navegador cuenta cuántos selectores de cada tipo tiene la regla y arma una terna de tres números:

```css
/* especificidad 0-0-1 · una etiqueta */
p { color: blue; }

/* especificidad 0-1-0 · una clase · GANA sobre la anterior */
.texto { color: red; }

/* especificidad 1-0-0 · un id · GANA sobre las dos */
#unico { color: green; }
```

La comparación es de izquierda a derecha y **no se suma**. Este es el malentendido más caro del tema: **la terna no es un número decimal, es una jerarquía.** Un id vale más que cualquier cantidad de clases, y una clase vale más que cualquier cantidad de etiquetas. Diez clases pierden contra un id. Ciento cincuenta etiquetas pierden contra una clase.

La analogía que lo deja claro: es como la antigüedad en una empresa donde primero se mira el cargo, después los años y después la fecha de ingreso. Un pasante con veinte años de antigüedad sigue siendo pasante frente a un director recién llegado. Sumar no aplica; comparar por niveles sí.

Los casos que vas a encontrar: las pseudoclases como `:hover` cuentan como clase. Los atributos entre corchetes como `[disabled]` cuentan como clase. Los pseudoelementos como `::before` cuentan como etiqueta. El selector universal `*` no aporta nada. Y `:where()` es la excepción útil: no aporta especificidad, y por eso sirve para escribir bases de estilo que sean fáciles de sobrescribir después.

### Tercero, el orden de aparición

Si dos reglas tienen el mismo origen y la misma especificidad, gana la última que el navegador leyó. Es el criterio más simple y el más subestimado, porque explica dos situaciones reales: que un estilo funcione o no según en qué parte del archivo lo escribiste, y que el orden en que enlazas las hojas de estilo en el `<head>` importe.

La consecuencia práctica: **si tu hoja propia va antes que la de un framework, el framework te va a pisar todo.** Eso te va a pasar en la sesión 6 con Tailwind si no lo tienes presente.

### Cuarto, la herencia

La herencia no es un criterio de desempate sino un mecanismo distinto, y por eso conviene separarla. Solo entra en juego cuando **ninguna** regla apunta a ese elemento para esa propiedad. Entonces algunas propiedades toman el valor del padre y otras no.

Las que tienen que ver con texto se heredan: `color`, `font-family`, `font-size`, `line-height`, `text-align`. Las que tienen que ver con la caja no: `border`, `padding`, `margin`, `background`, `width`. La razón es puro sentido común: si el borde se heredara, poner un borde a un contenedor le pondría borde a cada palabra de adentro.

El detalle que te ahorra dolores: cuando no entiendas de dónde salió un color que nadie escribió para ese elemento, casi siempre bajó del padre.

### Y `!important`, con franqueza

Sí, gana. Gana a todo lo del mismo origen sin importar la especificidad. Y precisamente por eso es una mala herramienta de trabajo diario: **cuando lo necesitas, casi siempre es señal de un problema anterior.**

El problema anterior suele ser uno de tres: le pusiste un id a algo que no lo necesitaba y ahora no puedes sobrescribirlo, tienes selectores tan largos que ya no puedes competir contra ti mismo, o estás intentando pisar un framework en el orden equivocado. Ninguno de los tres se arregla con `!important`; los tres se esconden con `!important`. Y el costo llega después: **el único remedio contra un `!important` es otro `!important`**, y esa escalada termina en un archivo donde nadie puede predecir nada.

Los dos casos donde sí se justifica: una utilidad de una sola línea que por definición debe ganar siempre, y sobrescribir el estilo en línea de una librería de terceros que no puedes editar.

### Las ideas que hay que llevarse

Son tres y las tres reaparecen en las dos sesiones siguientes.

**El CSS no falla en silencio por capricho: falla por una razón que se puede leer.** Cuando una regla no se aplica, hay cuatro candidatos y se revisan en orden. ¿La regla llega al elemento, o el selector está mal escrito y no apunta a nada? ¿Hay otra regla con más especificidad? ¿Hay otra regla igual más abajo? ¿O esa propiedad la está heredando y ninguna regla la toca? Esas cuatro preguntas son un procedimiento, y el panel de estilos del navegador las contesta las cuatro en diez segundos.

**Especificidad baja y estable es mejor que especificidad alta.** Suena al revés de lo intuitivo. Lo intuitivo es "si le pongo más selectores, funciona más". Lo cierto es que cada punto de especificidad que agregas hoy es un punto que tienes que superar mañana. Un archivo donde casi todo son clases simples es un archivo donde cualquier cosa se puede sobrescribir con otra clase simple. Un archivo lleno de ids y selectores de cinco niveles es un archivo donde solo se puede trabajar con `!important`.

**El estilo en línea existe y no es una solución.** Un `style="color: red"` en el HTML gana a cualquier regla de la hoja excepto a un `!important`. Es tentador cuando algo no funciona y es la peor decisión posible, porque saca el estilo del lugar donde se puede buscar, reutilizar y cambiar. Vale la pena saberlo hoy porque en el módulo 4, con React, el estilo en línea vuelve disfrazado de objeto de JavaScript y sigue siendo la misma mala idea.

### Ponte a prueba

*Si tu CSS funciona en tu máquina y no en la de un compañero, ¿cuál de los cuatro criterios podría estar cambiando?* Piensa en el orden de carga de las hojas y en los estilos del usuario: "en mi máquina sí funciona" tiene explicaciones concretas.

*¿Por qué el navegador trae estilos propios si podría no traer ninguno?* Conecta con la sesión 2: los estilos por defecto son la razón por la que un documento bien marcado se ve razonable sin CSS, que es exactamente lo que comprobaste en la Misión 01.

*Si `!important` gana siempre, ¿por qué no usarlo todo el tiempo y ahorrarse el problema?* Llega solo a la respuesta: si todo es importante, nada es importante, y quedas otra vez con el desempate por orden de aparición pero ahora sin poder salirte.

---

## Práctica en clase: predecir el color antes de abrir el navegador

La primera parte de esta práctica se hace **en papel, con el computador cerrado**, y eso no es un capricho: el ejercicio pierde todo su valor si abres el archivo y miras el resultado. El punto no es acertar; el punto es que tu error de predicción sea visible y tuyo.

**Predice en papel.** Mira este marcado y este CSS, y escribe en tu cuaderno de qué color queda el nombre del personaje. Sin discutir con nadie.

```html
<div id="panel">
  <p class="nombre destacado">Kael, guerrero élfico</p>
</div>
```

```css
p { color: blue; }
.nombre { color: red; }
#panel p { color: green; }
.destacado { color: orange; }
```

El favorito equivocado suele ser naranja, porque es lo que sale si razonas por orden de aparición sin haber comparado la especificidad primero. Y ahí es donde el ejercicio hace su trabajo: **si te equivocaste así, aplicaste el criterio número tres antes que el número dos.** Recorre el conteo: `p` es 0-0-1, `.nombre` es 0-1-0, `.destacado` es 0-1-0 y gana a `.nombre` por orden, y `#panel p` es 1-0-1 y gana a todos. Queda verde. Ahora sí abre el navegador y compruébalo.

**Agrega una regla más y vuelve a predecir.** Escribe `.destacado { color: purple !important; }` al final del archivo. Vas a ver que el `!important` en una clase con especificidad 0-1-0 le gana a un id con 1-0-1. Y ahora la pregunta que importa: *si quisieras volver a poner verde, ¿qué te toca hacer?* La única respuesta es otro `!important` con más especificidad. Ese es todo el argumento sobre `!important` en noventa segundos. Bórralo y sigue.

**Provoca el error del selector que no apunta a nada.** Escribe una regla para `.casilla-activa` cuando en el HTML la clase se llama `casilla_activa`. Guarda. No pasa nada, y no hay ningún mensaje de error en ninguna parte. Esa es la parte importante: **el CSS no avisa cuando un selector no encuentra nada, porque un selector que no coincide con nada es perfectamente legal.** Ahora abre las herramientas del navegador, selecciona el elemento, y fíjate en que la regla simplemente no aparece en el panel de estilos. Aprende a leer eso: si la regla no está en la lista, no es que perdió, es que nunca llegó. Es un diagnóstico completamente distinto y lleva a arreglar cosas distintas.

**Usa el panel de estilos para leer la cascada.** Con el elemento seleccionado, hay tres cosas que el panel ya te está diciendo y que casi nadie mira. Las reglas aparecen ordenadas de la que gana hacia abajo. Las propiedades perdedoras aparecen tachadas, y el tachón es la cascada dibujada. Y más abajo hay una sección de propiedades heredadas del padre, que es donde vas a encontrar el color que nadie escribió. Esta pestaña es a la maquetación lo que el depurador va a ser a JavaScript en el módulo 3: **la diferencia entre suponer y ver.**

**Explora el panel del modelo de caja.** Pasa al recuadro de colores del panel, el que muestra contenido, relleno, borde y margen en capas. Selecciona una casilla del tablero o la tarjeta del personaje y ve pasando el cursor por cada capa: el navegador resalta en la página exactamente la zona que corresponde. Este medio minuto de correspondencia visual enseña el modelo de caja mejor que cualquier diagrama. Después lee en el panel el ancho total del elemento y compáralo con el `width` que está escrito en el CSS. Si no coinciden, guarda la sorpresa: eso se explica en la sección siguiente.

**Caza especificidad en un archivo real.** Abre el CSS de cualquier proyecto o plantilla que tengas a mano y busca el selector más específico del archivo. Vas a encontrar algo como `#contenido .lista li.item a:hover`. Pregúntate qué haría falta para sobrescribir eso desde otra parte y vas a llegar a una respuesta incómoda. **Ese selector no es potente, es frágil**, porque depende de que la estructura del HTML no cambie nunca.

### Dudas que van a aparecer

**¿Para qué existen los ids si no conviene usarlos para estilos?** Existen para identificar de forma única un elemento, y eso es utilísimo para el atributo `for` de una etiqueta de formulario —que ya usaste en la sesión 2—, para los enlaces internos con `#` y para seleccionar el elemento desde JavaScript en el módulo 3. Para estilos, casi siempre es mejor una clase, porque una clase se puede reutilizar y se puede sobrescribir.

**¿El orden de las clases en el atributo `class` cambia algo?** No cambia nada, y este malentendido es muy común: `class="a b"` y `class="b a"` producen exactamente el mismo resultado. Lo que decide es el orden en la **hoja de estilos**, no en el atributo.

**¿Y si resuelvo todo con selectores más largos?** Cada nivel que agregas es especificidad que después hay que superar, y además ata el estilo a la estructura del HTML: el día que muevas un elemento de contenedor, el estilo desaparece sin explicación.

**¿Conviene ordenar el CSS de menos a más específico?** Sí, y es una buena costumbre: primero lo general —etiquetas y variables—, después los componentes por clase, y al final los estados y las excepciones. En la sesión 6 vas a ver que Tailwind resuelve esto de otra manera, quitando la pregunta de en medio.

---

## La caja, las unidades y las variables

### Las cuatro capas

Aquí se cobra la pregunta de la lectura previa. **Todo elemento del documento es una caja rectangular con cuatro capas**, de adentro hacia afuera: el contenido, el relleno que lo separa del borde, el borde, y el margen que lo separa de los vecinos.

La analogía es un cuadro enmarcado y funciona perfecto porque cada pieza tiene un equivalente exacto. La foto es el contenido. El paspartú blanco que va entre la foto y el marco es el relleno: está dentro del cuadro, y si el fondo es de color, el relleno se pinta con ese color. El marco de madera es el borde. Y el espacio de pared que dejas entre este cuadro y el de al lado es el margen: no le pertenece a nadie, no se pinta, y solo existe en relación con los vecinos.

Con esa analogía la distinción que más cuesta —relleno o margen— se vuelve automática: **si lo que quieres es separar el contenido de su propio borde, es relleno; si lo que quieres es separarte de otro elemento, es margen.**

Y ahora la respuesta a la pregunta. Por defecto, `width` mide **solo el contenido**, así que una caja con `width: 300px`, `padding: 20px` y un borde de 2 píxeles ocupa 344 píxeles de ancho. Si respondiste 340 estabas más cerca y te olvidaste del borde. La analogía del absurdo, que se queda: **es como comprar un sofá anunciado en 200 centímetros y descubrir que el 200 era solo del cojín, sin contar los brazos.** Nadie mide muebles así, y sin embargo así mide el CSS por defecto.

De ahí sale la línea que casi todo el mundo escribe sin saber por qué:

```css
/* Al principio de la hoja, siempre. Ahora width incluye
   el relleno y el borde: 300px es 300px en pantalla. */
*, *::before, *::after {
  box-sizing: border-box;
}
```

Escríbela hoy y escríbela en todos tus proyectos del semestre. Y si te preguntas por qué no es el comportamiento por defecto, la razón es que el modelo de caja original se especificó así hace más de veinticinco años, y cambiar el valor por defecto ahora rompería una parte enorme del web que ya existe. **La compatibilidad hacia atrás es la razón por la que el web tiene decisiones viejas que nadie defendería hoy**, y este es el ejemplo más limpio que vas a ver. Es la misma razón por la que existen `<b>` y `<i>`, y la misma por la que vas a ver cosas raras en el módulo 3.

Una curiosidad que por ahora solo tienes que reconocer: el colapso de márgenes. Dos elementos apilados con 20 píxeles de margen entre ellos no dejan 40, dejan 20, porque los márgenes verticales adyacentes se combinan en el mayor de los dos. En la sesión 5 el problema desaparece: **dentro de un contenedor de Flexbox o de Grid los márgenes no colapsan, y para separar se usa `gap`.**

### Unidades relativas y absolutas

Esto decide si tu Misión 02 se adapta de verdad o solo se encoge, así que vale la pena la claridad.

El `px` es absoluto: 16 píxeles son 16 píxeles independientemente de todo. Sirve para lo que no debe escalar con nada: el grosor de un borde, un radio pequeño, una sombra.

El `rem` se calcula sobre el tamaño de letra de la raíz del documento, que por defecto son 16 píxeles pero que **el usuario puede cambiar en la configuración de su navegador**, y esa frase es todo el argumento. Cuando alguien con dificultad para leer pone el tamaño de letra en 24, una interfaz escrita en `rem` crece completa y sigue siendo usable; una escrita en `px` se queda igual e ignora la configuración. La analogía útil: **el `rem` es una escala, como los planos de un edificio en los que todo está en metros pero el metro lo define el usuario.** Usa `rem` para tamaños de letra, espaciados, anchos de contenedor y —esto se cobra en la sesión 6— para los puntos de quiebre.

El `em` se calcula sobre el tamaño de letra **del propio elemento**, así que se acumula al anidar: un `em` dentro de otro `em` se multiplica, y ahí es donde la gente se pierde. Tiene un uso donde es claramente mejor que `rem`: el relleno interno de un botón, porque así el botón crece en proporción a su propio texto sin que haya que calcular nada.

El porcentaje es relativo al contenedor padre, y la trampa es que **el porcentaje de qué depende de la propiedad**: en `width` es del ancho del padre, y en `padding` y `margin` también es del ancho, incluso cuando se aplica arriba y abajo. Ese detalle explica un truco viejo para mantener proporciones que hoy no hace falta, porque existe `aspect-ratio`, que vas a usar en el tablero de la sesión 5.

Y `vw` con `vh` son porcentajes de la ventana visible. Son muy útiles para una pantalla de inicio a pantalla completa y son una trampa para el texto: un tamaño de letra en `vw` puro se vuelve ilegible en una ventana angosta y absurdo en un monitor grande, además de que ignora la configuración del usuario. Si vas a usarlos, acótalos, y para eso está `clamp()`, que resuelve el problema de una sola vez:

```css
:root {
  /* Nunca menos de 1rem, nunca más de 2rem, y entre
     esos dos límites escala con el ancho de la ventana. */
  --texto-titulo: clamp(1rem, 4vw, 2rem);
}
```

### Variables CSS para no repetir colores

Las custom properties son lo que hace la diferencia entre la versión "a mano" de tu Misión 02 y un archivo inmantenible.

El problema es concreto y ya lo tienes: el color de acento del juego aparece en el borde del tablero, en el fondo de los botones, en el texto del panel de puntajes y en el estado activo de una casilla. Son ocho o diez lugares. El día que decidas cambiarlo, hay que buscar y reemplazar, y el reemplazo va a fallar en dos lugares porque en uno lo escribiste en hexadecimal y en otro con `rgb()`.

```css
:root {
  --color-acento: #e2b714;
  --color-fondo: #1a1a1a;
  --casilla-clara: #eeeeee;
  --casilla-oscura: #333333;
  --espacio-base: 0.5rem;
  --radio: 0.375rem;
}

.boton-jugar {
  background: var(--color-acento);
  padding: var(--espacio-base) calc(var(--espacio-base) * 2);
  border-radius: var(--radio);
}
```

Y aquí la parte que las hace más que un ahorro de tecleo, y que las diferencia de las variables de un preprocesador: **las custom properties se heredan y viven en el navegador, no en un paso de compilación.** Eso significa que se pueden redefinir para una parte del árbol y todo lo de adentro cambia solo, que se pueden leer y escribir desde JavaScript en tiempo de ejecución, y que un tema oscuro completo puede ser cuatro líneas dentro de un bloque de `prefers-color-scheme`. Con nombre y apellido: **esa es exactamente la misión secundaria de 25 XP de este módulo**, y si entiendes esto hoy la resuelves en veinte minutos.

Y la regla que se califica: **si un valor aparece más de dos veces en tu hoja, es una variable.** Cuatro de los veinte XP específicos de la Misión 02 son literalmente eso.

### Las ideas que hay que llevarse

**El ancho que declaras no es el ancho que ocupa, hasta que escribes `border-box`.** Es la causa de los desbordes misteriosos y de las columnas que "casi" caben.

**`px` solo para bordes y sombras; `rem` para todo lo demás.** Es una costumbre que se instala una vez y te evita un fallo de accesibilidad real y una hoja que no se adapta.

**Un valor repetido es una variable esperando a ser declarada.** Y las variables de CSS no son azúcar sintáctica: viven en el navegador, se heredan y se pueden cambiar en tiempo de ejecución.

### Ponte a prueba

*Tienes tres tarjetas de `33%` con `padding: 1rem` y se van a la línea siguiente en lugar de llenar la fila. ¿Qué está pasando y con qué línea se arregla?*

*¿Por qué un punto de quiebre en `rem` sirve mejor a alguien que configuró su navegador con letra grande que uno en `px`?*

---

## Práctica en clase: darle estilos a tu ficha

Es la primera vez en el semestre que puedes hacer que algo se vea bien, y llevas tres semanas esperándolo. Si no entregaste la Misión 01, te toca maquetar la ficha de un compañero, que es peor y es justo, y además es una lección temprana sobre lo que se siente heredar código ajeno.

La instrucción de arranque no es negociable: **las primeras diez líneas del archivo son el reinicio y el bloque de variables, antes de cualquier otra cosa.** El `box-sizing: border-box` universal, el bloque `:root` con la paleta y las medidas, y nada más. No escribas una regla de componente antes de eso.

Mientras trabajas, hazte estas tres preguntas cada vez que algo no salga como esperabas: *¿por qué creo que esa regla no se está aplicando?*, *¿qué dice el panel de estilos sobre ese elemento?* y *¿este valor lo escribí ya en otra parte?*. La segunda es la más importante de la noche, porque te entrena en el procedimiento en lugar de darte el resultado.

Cuatro cosas van a pasarte y conviene que sepas qué hacer con cada una. Si te aparece el impulso de escribir tu primer `!important`, no lo escribas todavía: abre el panel de estilos y averigua contra qué regla estás perdiendo. Si algo se desborda cuando angostas la ventana, ese es un ancho fijo en píxeles pidiendo `rem` o un porcentaje. Si un espaciado no queda donde querías, abre el panel del modelo de caja y mira qué capa estás moviendo: casi siempre es `margin` donde iba `padding`. Y si un color ya lo escribiste en seis lugares, súbelo a `:root` ahora y no después.

Si terminas temprano, tienes dos caminos. Uno es empezar la maqueta base de la Misión 02 —solo la estructura y las variables, sin Grid todavía, que llega la semana entrante—. El otro, mejor: intenta el tema oscuro con `prefers-color-scheme` usando las variables que acabas de declarar. Son los 25 XP de la misión secundaria y esta noche ya tienes todo lo necesario.

Una cosa más, administrativa pero importante: en la **sesión 5 hay Auditoría 1**, cruzada, que vale **25 XP**, y cada semestre alguien la pierde por lo mismo. **Tienes que llegar con tu rama subida a GitHub, aunque esté incompleta.** Sin código subido no hay nada que auditar y no hay forma de recuperar esos puntos. Sube lo que tengas de esta práctica antes del fin de semana.

---

## Errores que probablemente vas a cometer

**Usar `!important` como primera herramienta de diagnóstico en lugar de última.** Es el error número uno del módulo y su mecánica es siempre igual: la regla no se aplica, no sabes por qué, agregas `!important`, funciona, y sigues adelante sin haber averiguado nada. El problema no es esa línea; es que la causa real —un id innecesario, un selector demasiado largo, un orden de carga equivocado— sigue ahí y va a volver a morderte tres archivos después. Y la deuda se acumula sola, porque lo único que le gana a un `!important` es otro. La disciplina que funciona: antes de escribirlo, abre el panel de estilos y di en voz alta contra qué regla estás perdiendo. Nueve de cada diez veces, al nombrar la regla ganadora ya no lo necesitas.

**Subir la especificidad con selectores cada vez más largos, creyendo que eso es "más potente".** Vas a escribir `#contenido .tablero .fila .casilla.activa` y a creer que hiciste algo robusto. Hiciste lo contrario: ataste el estilo a una estructura exacta de HTML, así que el día que muevas la casilla un nivel el estilo desaparece sin ningún mensaje, y además te dejaste sin margen para sobrescribir nada desde afuera. Si quieres sentirlo de verdad, intenta sobrescribir tu propio selector kilométrico desde otra hoja. La regla que queda es que en CSS la especificidad se gasta, y conviene gastar lo mínimo.

**Creer que `width` es el ancho que va a medir el elemento en pantalla.** Este es el que produce los desbordes misteriosos y las columnas que "casi" caben. Pones tres tarjetas de `33%` con relleno, y en lugar de llenar la fila se van a la línea siguiente, y la conclusión fácil es que el CSS es impredecible. La conclusión correcta es que el ancho por defecto mide el contenido y el relleno se suma por fuera. Lo que lo arregla de raíz es el `border-box` universal escrito antes de cualquier otra regla, y lo que arregla tu modelo mental es haber pasado el cursor por las capas del panel de caja mirando qué se resalta en la página.

**Usar `px` para todo, incluido el texto y los espaciados, y descubrir el problema demasiado tarde.** Viene de que la maqueta se piensa en píxeles y de que es la unidad que aparece en cualquier ejemplo de internet. Tiene dos consecuencias y una de las dos te cuesta puntos. La primera es que la interfaz ignora el tamaño de letra que el usuario configuró en su sistema, que es un fallo de accesibilidad real y del mismo tipo que los de la sesión 2. La segunda llega en la sesión 6: una hoja escrita entera en píxeles fijos no se adapta, solo se encoge, y la Misión 02 pide explícitamente que la versión móvil se reorganice de verdad. La costumbre que instalas desde hoy: `px` solo para bordes y sombras, `rem` para todo lo demás.

---

## Fuentes de esta sesión

- W3C. *CSS Cascading and Inheritance Level 5*. https://www.w3.org/TR/css-cascade-5/
- MDN Web Docs. *CSS box model*. https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_box_model

La especificación del W3C es la autoridad final sobre el orden exacto del desempate, y es más legible de lo que su nombre sugiere: la sección de la cascada se lee en veinte minutos y contesta discusiones que en internet duran años. No hace falta leerla completa. La página de MDN sobre el modelo de caja sí conviene tenerla en marcadores, porque el diagrama de las cuatro capas es el que vas a querer volver a ver cada vez que confundas relleno con margen.

Y la regla de IA se aplica igual que siempre, con un detalle propio de este módulo: para CSS, los asistentes producen hojas larguísimas con propiedades redundantes, prefijos de navegador que ya no hacen falta y valores en píxeles por todos lados. Si usas uno, lo que hay que auditar es cuántas de esas líneas hacen algo. Todo se declara en `IA.md` con qué pediste, qué devolvió, qué estaba mal y qué corregiste. **Sin declarar se califica en 0 y no admite reintento; declararlo NO baja la nota**, porque lo que se evalúa es tu capacidad de auditar.

---

## Antes de la sesión 5

Lee la sección "Módulo 2, sesión 5" de `GUIA-DEL-CURSO.md` y la página de MDN sobre los conceptos básicos de flexbox, solo la parte de los dos ejes: el eje principal y el eje transversal. Diez minutos, y no intentes memorizar propiedades.

Y llega con una respuesta pensada a esta pregunta: *si tuvieras que acomodar sesenta y cuatro casillas en un cuadrado de ocho por ocho, y solo pudieras usar lo que sabes hoy, ¿cómo lo harías?* Escríbelo aunque sea en dos líneas. La sesión 5 empieza haciéndolo de la manera dolorosa a propósito, y el contraste solo funciona si llegaste habiendo pensado en la manera dolorosa por tu cuenta.
