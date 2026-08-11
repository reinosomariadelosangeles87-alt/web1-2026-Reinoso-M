# Sesión 6 · Mobile-first y Tailwind CSS

**Módulo 2** · Zona 2: El Taller de Diseño · cierre del módulo
**Lo que sale de esta noche:** la interfaz de tu juego reorganizándose de verdad en tres tamaños de pantalla, y la comparación entre CSS a mano y Tailwind empezada por escrito
**Lo que se entrega:** Misión 02 — Tablero de juego adaptable (100 XP · 9 h autónomas)

Esta noche arranca con **Quiz 2**: tres preguntas de aplicación, 15 XP, en los primeros diez minutos. Entra lo de las sesiones 4 y 5.

---

## Por qué esta noche es sobre decidir, no sobre sintaxis

Los dos temas de hoy parecen sueltos —diseño adaptable con enfoque mobile-first, y clases de utilidad con Tailwind— y los une algo concreto: **las dos mitades son decisiones, no técnicas.**

Mobile-first no es una sintaxis nueva. Las media queries son las mismas en los dos sentidos. Es un **orden de trabajo** que te obliga a priorizar. Y Tailwind no es "el CSS mejorado", es un intercambio con un costo real: se gana mantenibilidad a cambio de un HTML lleno de clases. Si sales de esta noche sabiendo escribir `md:grid-cols-2` pero sin poder decir cuándo Tailwind es exagerado, no aprendiste lo que había que aprender.

Y de ahí sale por qué la Misión 02 se entrega dos veces. No es para que hagas doble trabajo: es para que la comparación del `README.md` esté escrita **desde tu experiencia y no desde lo que dice internet**. Esos 4 XP de la comparación reflexiva son los más honestos del módulo, porque no se pueden conseguir leyendo.

Esta es también la última noche del módulo 2, así que hay una función de cierre. Lo que aprendiste en tres semanas —la cascada, la caja, las unidades relativas, las variables, Grid y Flexbox— es exactamente lo que hace que la versión mobile-first sea tres líneas y no medio archivo.

---

## Mobile-first: por qué de pequeño a grande

Antes de leer lo que sigue, respondete la pregunta de la lectura previa: **¿por qué diseñar primero para el celular y después para el escritorio, y no al contrario, que parece más natural porque trabajas en un computador?**

La respuesta que sale casi siempre es alguna versión de "porque hay mucha gente en celular". Es cierta y es la menos interesante de las tres razones. Faltan dos, y las dos que faltan son las que importan para escribir código.

Y antes de las razones, una demostración que vale más que cualquier argumento: abre tu Misión 01 —o cualquier página que hayas hecho— y **arrastra el borde del navegador hasta 360 píxeles de ancho.** Mira el desastre: el texto desbordado, el tablero cortado, los botones apilados a medias. Ese es el ancho en el que la mayoría de la gente va a jugar tu juego, y es el ancho para el que no diseñaste.

### Primera razón: empezar por lo pequeño obliga a priorizar

En 360 píxeles de ancho no cabe todo, así que hay que decidir qué es esencial **antes** de tener espacio para adornos. Es una restricción que funciona como herramienta de diseño.

La analogía que se queda: **es como empacar para un viaje con una maleta de mano en lugar de una grande.** Con la maleta grande uno mete todo y no piensa; con la de mano hay que decidir qué se usa de verdad. Y lo interesante es que el resultado suele ser mejor incluso en la maleta grande, porque ya sabes qué es importante. Una interfaz diseñada primero para el celular llega al escritorio sabiendo qué es lo principal; una diseñada primero para el escritorio llega al celular con veinte cosas y sin jerarquía.

### Segunda razón: añadir es más fácil que quitar, y en CSS eso es literal

Aquí la razón deja de ser filosófica y se vuelve técnica. Mobile-first se escribe con `min-width`: la base es el móvil, sin ninguna media query, y cada punto de quiebre **agrega** reglas para pantallas más grandes.

```css
/* Base: móvil. Sin media query. Todo en una columna. */
.pantalla-juego {
  display: grid;
  grid-template-areas:
    "cabecera"
    "tablero"
    "puntajes"
    "controles";
  grid-template-columns: 1fr;
}

/* Tableta y arriba: el panel de puntajes se va al lado. */
@media (min-width: 48rem) {
  .pantalla-juego {
    grid-template-areas:
      "cabecera  cabecera"
      "tablero   puntajes"
      "controles controles";
    grid-template-columns: 1fr 20rem;
  }
}
```

Detente en esas dos declaraciones de `grid-template-areas`, porque son el punto: **eso es la interfaz reorganizándose de verdad, no encogiéndose.** El panel de puntajes no es una columna angosta en el celular: en el celular está abajo, a todo el ancho, donde sí se puede leer. Y todo el cambio son cuatro líneas, gracias a las áreas nombradas de la sesión 5. Aquí es donde el trabajo de la semana pasada se cobra.

Ahora el camino contrario, el de `max-width`, para que veas por qué es peor. Con desktop-first la base es la pantalla grande y cada media query tiene que **desarmar** lo que la base declaró: quitar columnas, quitar posiciones, quitar tamaños. Y como las reglas de la base siguen aplicándose, terminan pisándose entre sí y llega el momento en que agregas `!important` para ganarle a tu propia base. **Ese `!important` es el síntoma de haber empezado por el lado equivocado**, y conecta directo con la sesión 4.

### Tercera razón: es donde está la gente

La mayoría del tráfico web mundial es móvil, y tu juego se va a jugar en un teléfono. Es la razón que ya conocías, y con eso basta.

### Los puntos de quiebre en `rem` y no en `px`

Este tramo es corto y es de los que más valen. Un punto de quiebre en `px` es una medida absoluta, y un punto de quiebre en `rem` se calcula sobre el tamaño de letra base del navegador, **que el usuario pudo haber cambiado**.

La consecuencia es exactamente la que quieres: cuando alguien configura el navegador con letra más grande porque no ve bien, el contenido necesita más espacio para el mismo texto, así que lo correcto es que el diseño cambie a la disposición de pantalla angosta antes que en la configuración normal. Con `px` eso no pasa, y quien puso letra grande recibe una maqueta de escritorio con texto que no cabe.

Los tres valores de referencia del curso: **48rem para tableta y 75rem para escritorio**, con el móvil como base sin media query.

Y la parte que se califica, dicha con franqueza: **los puntos de quiebre se eligen por el contenido, no por la lista de dispositivos de moda.** No tienes que copiar los anchos de un iPhone concreto, porque ese teléfono va a cambiar y tu contenido no. El procedimiento correcto es angostar la ventana lentamente y **poner un punto de quiebre donde algo se ve mal**: cuando la línea de texto se vuelve demasiado larga para leerla, cuando el tablero deja de ser cuadrado, cuando dos columnas se aprietan. Eso es un punto de quiebre razonado, y esos son 6 XP de la rúbrica.

### Dos herramientas que aún no conoces

La primera es una línea de HTML: sin `<meta name="viewport" content="width=device-width, initial-scale=1">` en el `<head>`, **ninguna media query funciona en un celular real**, porque el teléfono finge ser una pantalla ancha y escala todo hacia abajo. Va en el HTML y no en el CSS, y es la causa número uno de "en el emulador funciona y en mi teléfono no".

La segunda es el modo de dispositivo de las herramientas del navegador, que sirve para desarrollar y que **no reemplaza probar en un teléfono real**, porque el emulador no reproduce el tamaño del dedo, la latencia ni el brillo del sol.

### Las ideas que hay que llevarse

**Adaptarse no es encogerse.** Una maqueta que en el celular es la misma de escritorio con todo más pequeño no es adaptable: es ilegible con buena voluntad. Adaptarse significa que los bloques cambian de lugar, que lo secundario baja, y que lo esencial queda accesible con un pulgar. La rúbrica de la Misión 02 dice esto con todas las palabras y vale 6 XP.

**Mobile-first es un orden de trabajo, no una sintaxis.** Las mismas media queries existen en los dos sentidos. Lo que cambia es cuál es la base y en qué dirección se agrega. Y esa dirección determina si el archivo crece sumando o crece desarmando.

**Los `rem` de la sesión 4 y las áreas nombradas de la sesión 5 son lo que hace que esto sea corto.** Si tu hoja está en píxeles fijos y sin variables, mobile-first cuesta medio archivo. Si está en `rem` con variables y con `grid-template-areas`, cuesta cuatro líneas. **El módulo entero venía preparando esta noche.**

### Ponte a prueba

*¿Probaste tu Misión 01 en tu propio teléfono?* Si la respuesta es no, esa es exactamente la razón por la que existe la práctica de esta noche.

*Si un punto de quiebre está en `px` y el usuario aumenta el tamaño de letra del navegador, ¿qué pasa?* El diseño no cambia y el texto deja de caber. Es la justificación completa del `rem` en una frase.

*¿Cuál es el ancho mínimo que vas a soportar, y quién lo decide?* Lo decides tú, y hay que escribirlo en el `README.md`, no dejar que se descubra al calificar.

---

## Práctica en clase: reorganizar la interfaz de verdad y después reescribirla con utilidades

Esta práctica tiene dos mitades con una bisagra. La primera es mobile-first sobre la maqueta que construiste la semana pasada. La segunda es tu primer contacto con Tailwind, **haciendo exactamente lo mismo**, para que la comparación sea real y no teórica.

Necesitas el editor con la maqueta de Grid de la sesión 5 y el navegador en modo de dispositivo.

**Provoca el error del viewport ausente.** Abre la maqueta en el modo de dispositivo con un teléfono seleccionado, y **antes** de agregar la etiqueta de viewport. La página aparece diminuta, como una miniatura de la versión de escritorio, y ninguna media query se activa. Lo desconcertante es que el CSS está correcto. Después agrega la línea en el `<head>`, recarga, y mira la página aparecer al tamaño correcto de golpe:

```html
<meta name="viewport" content="width=device-width, initial-scale=1">
```

La moraleja completa: **esa línea no es CSS, va en el HTML, y sin ella todo el trabajo de esta noche es invisible en un teléfono real.** Es el tipo de detalle que cuesta una tarde de frustración a quien no lo sabe.

**Encuentra tus puntos de quiebre angostando la ventana, sin decidirlos de antemano.** Empieza en escritorio y arrastra el borde despacio, anotando qué se rompe y en qué ancho. "Aquí el panel de puntajes ya está muy angosto". "Aquí el tablero dejó de ser cuadrado". "Aquí los botones se salen". Ese es el procedimiento entero, y hacerlo una vez sirve más que cualquier explicación: **los puntos de quiebre se encuentran mirando el contenido, no consultando una lista de teléfonos.**

**Invierte la maqueta a mobile-first.** Toma el CSS de la sesión 5, que casi seguro está escrito pensando en escritorio, y reescríbelo con la base en una columna y dos bloques de `min-width` en `48rem` y `75rem`. Vas a descubrir que la parte más difícil no es escribir las media queries sino **quitar de la base todo lo que asumía una pantalla ancha**. Ese trabajo de quitar es exactamente el que no habrías tenido que hacer si hubieras empezado por el celular, y sentirlo mientras lo haces es el argumento definitivo de todo el tema.

**Provoca el error del punto de quiebre en píxeles.** Deja un punto de quiebre en `768px` en lugar de `48rem`. Después ve a la configuración del navegador y cambia el tamaño de letra por defecto a 24 píxeles, que es lo que haría alguien con dificultad para leer. Recarga. La maqueta sigue en modo escritorio y el texto ya no cabe en las columnas. Cambia `768px` por `48rem`, recarga, y mira la maqueta pasar a una columna sola. **Esta demostración dura cuarenta segundos y es la única forma de que el argumento del `rem` deje de sonar teórico.** Devuelve la configuración a 16 después.

**La bisagra: el mismo bloque, ahora con utilidades.** Con un entorno de Tailwind ya funcionando —no lo instales sobre la marcha, eso se come veinte minutos y no enseña nada— toma **un solo componente pequeño**, el panel de puntajes por ejemplo, y escríbelo con clases de utilidad al lado de la versión a mano:

```html
<!-- A mano: el HTML está limpio y los estilos viven en otro archivo -->
<section class="panel-puntajes">
  <h2 class="panel-puntajes__titulo">Puntajes</h2>
</section>

<!-- Con utilidades: todo está aquí y no hay otro archivo que buscar -->
<section class="rounded-lg bg-slate-800 p-4 shadow">
  <h2 class="mb-2 text-lg font-semibold text-amber-400">Puntajes</h2>
</section>
```

No decidas todavía cuál es mejor. Escribe en dos columnas qué ganó y qué perdió cada versión. Lo que casi todo el mundo anota es que la segunda es más fea de leer y que en la primera hay que ir a buscar a otro archivo. **Esas dos observaciones son toda la teoría de la segunda mitad de la noche**, y llegan mucho mejor si las escribes tú.

**Prueba los prefijos de punto de quiebre.** En Tailwind el diseño adaptable son prefijos en la misma clase, y el sistema es mobile-first por diseño: una clase sin prefijo aplica en todos los tamaños, y `md:` aplica desde ese punto hacia arriba.

```html
<!-- Una columna en móvil; dos columnas desde tableta.
     La clase sin prefijo es la base: el sistema es
     mobile-first por diseño, no por convención. -->
<div class="grid grid-cols-1 gap-4 md:grid-cols-[1fr_20rem]">
```

Fíjate en lo que acaba de pasar: **el orden mobile-first no es opcional en Tailwind, está incorporado.** Es una de las razones honestas por las que a un equipo le conviene, y no es la razón que suele darse.

**Genera un componente con IA y audítalo.** Este es el cierre de la práctica y el más importante. Pídele a un asistente el panel de puntajes con Tailwind. Va a devolver algo que se ve bien de inmediato, y hay que reconocerlo con honestidad: **en maquetación los asistentes son genuinamente buenos**, porque es código repetitivo con patrones muy documentados. Y ahora audítalo con dos condiciones.

La primera condición es la semántica. Casi siempre devuelve `div` anidados donde iba una `<section>` con su encabezado, y un `<div onclick>` o un `<a>` sin destino donde iba un `<button>`. Se ve idéntico y es peor, y es exactamente el error de la sesión 2 reaparecido. **La razón es la que viste en la sesión 3: el modelo repite lo frecuente, y en el corpus hay muchísimo más `div` que `section`.**

La segunda condición es la accesibilidad. Revisa el contraste del texto sobre el fondo que eligió, que muy a menudo es un gris sobre gris que no llega a la relación 4.5 a 1 de nivel AA; busca los botones que solo tienen un icono y no tienen nombre accesible; y presiona Tab para ver si el foco es visible, porque es común que venga con el anillo de foco desactivado "porque queda más limpio". Corre Lighthouse ahí mismo sobre el componente generado y mira los fallos aparecer.

Cierra escribiendo el `IA.md` en el momento, con las cuatro casillas: qué pedí, qué me dio, qué estaba mal —los `div`, el contraste, el foco— y qué corregí. Documentar un error real generado por IA con su corrección habilita la insignia 🔍 **Cazador de alucinaciones**, que da un día extra de plazo y es acumulable.

### Dudas que van a aparecer

**¿Tailwind no es lo mismo que los estilos en línea que se descartaron en la sesión 4?** Es una excelente pregunta y merece respuesta completa. No es lo mismo por tres razones: las utilidades vienen de un sistema de diseño con una escala limitada y coherente, así que no se puede poner cualquier valor arbitrario; sí soportan estados y puntos de quiebre, que el estilo en línea no puede hacer; y no ganan la guerra de especificidad, porque son clases normales. Lo que sí comparten con el estilo en línea es el problema de legibilidad, y eso hay que concederlo.

**¿El archivo de CSS de Tailwind no queda gigantesco?** No, y la razón vale la pena: el compilador revisa los archivos del proyecto y genera solo las utilidades que están efectivamente usadas. La consecuencia práctica que sí importa es que **si construyes nombres de clase pegando cadenas en JavaScript, el compilador no las va a encontrar y esos estilos no van a existir.** Es un error real que vas a encontrar en el módulo 4 con React.

**¿Y cuando la misma combinación de doce clases se repite en veinte lugares?** Ese es el límite real del enfoque, y la solución es un componente, no una clase nueva. En el módulo 4, con React, eso se vuelve natural, y ahí es donde Tailwind empieza a tener más sentido del que tiene hoy.

---

## Clases de utilidad: qué problema resuelven de verdad

### El problema real, contado como sucede

No empecemos por Tailwind, empecemos por el problema, porque si no lo entiendes la solución te va a parecer una moda.

Así pasa en un proyecto real. Alguien escribe `.tarjeta` y funciona. Tres semanas después alguien necesita una tarjeta un poco distinta y escribe `.tarjeta-grande`. Después llega `.tarjeta-grande-oscura`. Un mes después el archivo tiene dos mil líneas y ocurre lo predecible: **alguien ve una clase que parece no usarse, quiere borrarla, y no se atreve.** No se atreve porque no tiene forma de saber en cuáles de las cuarenta plantillas del proyecto se usa. Entonces no la borra. Y como nadie borra nunca, el archivo solo crece.

Esa es la enfermedad y tiene nombre: **el CSS es global y el HTML es local.** Cuando tocas una clase, no sabes a quién estás afectando. Cuando tocas un elemento, sabes exactamente qué estás afectando. Toda la propuesta de las clases de utilidad sale de ahí.

La analogía que lo cierra: **el CSS tradicional es un armario compartido de un apartamento con seis roomies.** Es eficiente porque nadie compra dos veces la misma cosa, y nadie se atreve a botar nada porque quién sabe de quién es. Las utilidades son que cada uno tenga sus cosas en su cuarto: se repite más, y cuando alguien se va, se va con sus cosas y no queda basura de nadie.

Y de ahí el beneficio concreto: **borras el elemento y sus estilos se van con él.** No queda nada que limpiar porque no había nada compartido. Eso es lo que Tailwind resuelve de verdad, y no es "escribir menos CSS".

### El costo, dicho sin adornos

Ahora la otra columna, con la misma seriedad, porque es lo que distingue una decisión de una moda.

El HTML se llena de clases. Un elemento con doce utilidades es difícil de leer, y en un archivo con cincuenta elementos así hay que entrenar el ojo para encontrar la estructura entre el ruido. Es un costo real y quien diga que no lo es, está vendiendo algo.

Hay una curva de aprendizaje que no es trivial: hay que aprender los nombres del sistema, y `p-4` no es más expresivo que `padding: 1rem`, solo más corto. Y hay una dependencia de herramientas: el proyecto necesita un paso de compilación, y en un archivo suelto que se abre con doble clic eso no funciona.

Y el más importante para el semestre: **el duplicado se vuelve visible.** Cuando la misma combinación de doce clases aparece en veinte lugares, no hay un archivo que lo esconda: está ahí, veinte veces, a la vista. Eso puede leerse como una desventaja o como una virtud, y honestamente es las dos cosas. Es incómodo y también es información, porque acabas de descubrir que ahí había un componente que nadie había extraído.

En una frase: **es un intercambio real, no una mejora gratis.**

### Cuándo conviene y cuándo es exagerado

Conviene cuando hay un equipo, cuando el proyecto va a crecer, cuando ya existe un sistema de diseño con una escala definida y cuando hay componentes que se reutilizan mucho —y por eso encaja tan bien con React—. En esos escenarios el problema del CSS global es real y el costo del HTML ruidoso se paga con gusto.

Es exagerado en una página de tres secciones, en un proyecto de una sola persona con doscientas líneas de CSS, y cuando hay que entregar un archivo HTML suelto sin paso de compilación. Ahí Tailwind es traer una grúa para colgar un cuadro.

Y aquí lo honesto: **tu Misión 02 es del segundo grupo.** Es una página de tres secciones hecha por una persona. Con criterios de proyecto real, Tailwind está de más. Y sin embargo la misión pide las dos versiones, por una razón que no es la de conveniencia: **la única forma de tener una opinión propia sobre un intercambio es haber pagado los dos costos.** Después de esta misión vas a poder decir "yo escribí lo mismo de las dos maneras y esto es lo que sentí", y eso es una posición profesional. Sin la experiencia, lo único que puedes hacer es repetir el argumento de quien lo dijo más fuerte en internet.

### La generación asistida, como regla

**La maquetación es probablemente el terreno donde los asistentes son más útiles.** Es código repetitivo, con patrones documentadísimos, y el resultado se verifica mirando la pantalla, que es la forma de verificación más rápida que existe. Aprovéchalo, en serio.

Con dos condiciones que hay que verificar siempre, porque son los dos fallos que estos modelos cometen sistemáticamente. **Verificar la semántica**, porque tienden a producir `div` anidados donde iba un elemento de sección, y el resultado se ve igual y es peor. **Verificar la accesibilidad**, porque el contraste insuficiente, los botones sin nombre accesible y el foco invisible son fallos frecuentísimos y no se ven hasta que alguien no puede usar la página.

Y la observación que amarra el semestre: **esos dos fallos son exactamente los dos temas de la sesión 2.** Lo que aprendiste en el módulo 1 no era teoría de calentamiento: es la lista de verificación con la que auditas lo que un asistente te devuelve. Sin la sesión 2, no tendrías con qué revisar. Es la misma tesis del curso dicha otra vez: **generar es barato, verificar es la competencia.**

### Las ideas que hay que llevarse

**El problema que resuelven las utilidades no es escribir menos CSS: es que el CSS es global y por eso nadie se atreve a borrar nada.**

**Todo intercambio tiene dos columnas, y si solo puedes nombrar una de las dos, estás repitiendo un argumento ajeno.**

**En maquetación los asistentes son buenos, y sus dos fallos sistemáticos son semántica y accesibilidad: exactamente lo del módulo 1.**

### Ponte a prueba

*Nombra un costo real de tu opción preferida entre CSS a mano y Tailwind.* Si no puedes, todavía no tienes una opinión, tienes una preferencia heredada.

*¿En qué caso concreto de tu propio proyecto Tailwind te ahorró trabajo, y en cuál te lo costó?*

---

## Tu misión de la semana: Tablero de juego adaptable (100 XP)

Maquetas la interfaz completa de un juego de mesa **sin una sola línea de JavaScript**: el tablero en cuadrícula, el panel de puntajes, la zona de jugadores y los controles. Tiene que funcionar en tres tamaños de pantalla, y la versión móvil tiene que **reorganizarse de verdad, no solo encogerse**.

Y se entrega **dos veces**: una versión con CSS a mano y otra con Tailwind, en dos carpetas dentro de `practica-02-tablero/`. En el `README.md` argumentas cuál mantendrías dentro de un año.

### Por qué las dos versiones

Ya lo dijimos y vale repetirlo, porque es lo que hace que el doble trabajo tenga sentido: la única forma de tener una opinión propia sobre un intercambio es haber pagado los dos costos. La comparación no se puede conseguir leyendo, y por eso vale puntos.

### Cómo se califica

| Criterio | XP |
|---|---|
| Funciona: la interfaz cumple lo pedido en los tres tamaños | 35 |
| Calidad del código: nombres claros, sin repetición ni código muerto | 25 |
| Específico del módulo (ver abajo) | 20 |
| Proceso en Git: commits pequeños y descriptivos, PR completo | 10 |
| Auditoría de IA: `IA.md` real y reflexivo | 10 |

Los 20 XP específicos de esta misión: el tablero usa Grid y no una tabla ni posiciones absolutas (6), el diseño es mobile-first con puntos de quiebre razonados (6), no hay valores mágicos repetidos porque usas variables o utilidades (4), y la comparación entre los dos enfoques es reflexiva y no genérica (4).

**Misión secundaria opcional (+25 XP):** un tema oscuro que respete `prefers-color-scheme`. Con las variables CSS de la sesión 4 ya declaradas, son veinte minutos.

**Plazo:** antes de la medianoche del día anterior a la sesión 7. Presupuesto estimado: **9 horas autónomas** repartidas en 1 de planear, 3 de CSS a mano, 3 de Tailwind, 1 de probar y 1 de comparar.

### Cómo trabajarla, en tres frentes

El primer frente es terminar la versión de CSS a mano en mobile-first, con los tres tamaños funcionando. La comprobación es concreta y conviene hacerla siempre: **abre el modo de dispositivo y recorre 360, 768 y 1280, verificando en cada uno que nada se desborda, que el tablero sigue cuadrado y que se puede llegar a todos los controles.** Y en el más angosto, que la maqueta se reorganizó y no solo se encogió.

El segundo frente es la versión con Tailwind. Empiézala mientras tengas ayuda a mano, para que un problema de instalación no te cueste la entrega.

El tercer frente es obligatorio aunque no termines nada: **escribe en el `README.md` tres observaciones concretas de la comparación mientras la experiencia está fresca.** No el párrafo final: tres observaciones. La razón es la misma por la que el `IA.md` se llena en el momento: los 4 XP de la comparación reflexiva no se pueden reconstruir de memoria una semana después. Y lo que no cuenta: "Tailwind es más rápido pero el HTML queda feo" es lo que dice internet y vale cero. Lo que cuenta es "en la versión a mano cambié el color de acento en un lugar y se actualizó todo; en la de Tailwind tuve que buscarlo en once elementos", porque eso te pasó a ti.

Mientras trabajas, hazte estas tres preguntas: *¿probé esto en 360 de ancho?*, *¿por qué elegí ese punto de quiebre?* y *¿esto se reorganizó o solo se encogió?*. La segunda es la que más revela, porque quien copió una lista de anchos de internet no la puede contestar y quien angostó la ventana sí.

Y la regla de IA, con el detalle de este módulo: la maquetación es donde los asistentes son buenos y donde más los vas a usar, así que el `IA.md` de esta misión probablemente sea el más largo del semestre hasta ahora. Todo lo generado se declara con qué pediste, qué devolvió, qué estaba mal y qué corregiste. **Sin declarar se califica en 0 y no admite reintento; declararlo NO baja la nota**, porque lo que se evalúa es tu capacidad de auditar. Y lo que hay que auditar en este módulo son las dos condiciones de esta noche: la semántica y la accesibilidad.

---

## Errores que probablemente vas a cometer

**Escribir desktop-first y llamarlo adaptable.** Es el error más común del tema y el más entendible, porque estás sentado frente a un monitor grande y es lo que ves. El síntoma es una hoja llena de `max-width` con una base pensada para pantalla ancha, y la consecuencia es que cada media query tiene que desarmar lo que la base ya declaró: quitar columnas, quitar anchos, quitar posiciones. Como las reglas de la base siguen activas, empiezan a pisarse entre sí y aparece el `!important` para ganarle a tu propio código, que es el mismo problema de la sesión 4 con otro origen. La corrección no es cambiar `max-width` por `min-width` y ya: es invertir el orden de trabajo, empezar por la ventana angosta y agregar hacia arriba. Cuesta una vez y después es más barato para siempre.

**Copiar una lista de puntos de quiebre de internet sin haber mirado tu propio contenido.** Aparecen los anchos de algún dispositivo de moda, en píxeles, en un proyecto cuyo contenido no tiene nada que ver. El problema de fondo es que un punto de quiebre no es una propiedad del teléfono, es una propiedad del contenido: es el ancho en el que **tu** maqueta se ve mal. La pregunta que lo destapa es sencilla y si copiaste no la puedes responder: *¿por qué ese número y no otro?* El procedimiento correcto cuesta dos minutos: angostar la ventana despacio y poner el quiebre donde algo se rompe. Y en `rem`, no en `px`, para que quien configuró letra grande reciba la maqueta que necesita.

**Confundir adaptarse con encogerse.** La versión de celular queda siendo la de escritorio con todo más pequeño: el panel de puntajes es una columna de cuatro centímetros ilegible, el tablero cabe pero las casillas son del tamaño de una uña, y los controles están apretados en una fila. Técnicamente no se desborda nada, así que es fácil creer que está resuelto. La rúbrica dice explícitamente que la versión móvil tiene que reorganizarse de verdad, y la comprobación que puedes hacer solo es probarlo en tu propio teléfono con el pulgar: si no puedes presionar un botón sin acertar en un objetivo de cinco milímetros, la maqueta no está adaptada aunque quepa.

**Adoptar Tailwind como creencia y no como decisión, en cualquiera de las dos direcciones.** Vas a encontrar gente convencida de que Tailwind es el futuro y el CSS a mano es de otra época, y gente convencida de que Tailwind es ensuciar el HTML y que quien lo usa no sabe CSS. Los dos están repitiendo argumentos ajenos, y los dos se delatan igual: cuando se les pregunta qué costo tiene su opción preferida, no lo saben. La comparación del `README.md` está diseñada exactamente contra esto, y por eso los 4 XP se pierden con un párrafo genérico aunque la conclusión sea razonable. Lo que se califica es si la argumentación viene de haber escrito las dos versiones: cuál valor tuviste que cambiar en once lugares, cuál archivo te costó volver a entender, cuánto tardaste en cada una la segunda vez.

---

## Fuentes de esta sesión

- Tailwind CSS. *Styling with utility classes*. https://tailwindcss.com/docs/styling-with-utility-classes
- Tailwind CSS. *Responsive design*. https://tailwindcss.com/docs/responsive-design
- MDN Web Docs. *Lazy loading*. https://developer.mozilla.org/en-US/docs/Web/Performance/Guides/Lazy_loading

Las dos páginas de Tailwind son documentación de primera mano y hay que leerlas con una advertencia: son documentación **y** material de promoción del proyecto, así que presentan el enfoque con sus ventajas mejor argumentadas que sus costos. Eso no las hace menos útiles, pero sí obliga a leerlas sabiendo que la columna del costo la tienes que sostener tú.

La página de MDN sobre carga diferida conecta con el módulo 5: cuando la interfaz ya funciona en tres tamaños, la pregunta siguiente es cuánto pesa, y `loading="lazy"` en las imágenes es la mejora más grande por menos esfuerzo que existe en el web. Es una línea por imagen y la puedes aplicar en la Misión 02 aunque todavía no trabajemos rendimiento.

Y una nota de cierre del módulo. En tres sesiones aprendiste a saber por qué una regla no se aplica en lugar de adivinar, a elegir el sistema de maquetación según la forma del problema, y a diseñar de la pantalla pequeña hacia arriba. **Lo que hiciste esta noche en cuatro líneas de `grid-template-areas` habría costado medio archivo sin las variables, los `rem` y las áreas nombradas de las dos sesiones anteriores.** El módulo estaba construido para esta noche.

---

## Antes de la sesión 7

Lee la sección "Módulo 3, sesión 7" de `GUIA-DEL-CURSO.md` y la introducción de la *JavaScript Guide* de MDN, solo la parte de declaraciones y ámbito. Quince minutos esta vez, porque el módulo 3 es otra cosa y conviene entrar con algo leído.

Y llega con esta pregunta contestada como creas, porque con ella abre la sesión: *si dos partes de un programa pueden modificar la misma variable en cualquier momento, y algo sale mal, ¿cómo averiguas cuál de las dos lo hizo?* Respóndela aunque sea con una intuición. En la sesión 7 vas a descubrir por qué esa pregunta no tiene buena respuesta y por qué eso cambia la forma de escribir código.

Una última cosa, con seriedad: la sesión 7 abre el **módulo 3**, que es el más largo y el más difícil del curso, con cinco sesiones y el doble de horas que este por una razón. No hay quiz de apertura esa noche, pero sí hay lectura previa. **Llega descansado.**
