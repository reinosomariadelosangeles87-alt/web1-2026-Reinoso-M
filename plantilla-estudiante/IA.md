# Declaración de uso de IA

> Obligatoria en todas las prácticas. Si usaste un asistente, descríbelo aquí con
> precisión. Si NO usaste ninguno, escribe eso y explica cómo resolviste la parte
> más difícil por tu cuenta — también cuenta como declaración válida.
>
> Recuerda: código de IA sin declarar se califica en CERO y no admite reintento.
> Declararlo honestamente NO baja tu nota. Lo que se evalúa es tu capacidad de auditar.

 Herramientas que usé
Deepseek a través de la interfaz de escritorio/web. No usé GitHub Copilot ni Cursor.

 Qué le pedí
Cómo clonar el repositorio en mi computadora.

Cómo crear una rama (feat/practica-00), hacer cambios y subirlos con git push.

Cómo solucionar el error en GitHub donde main y feat/practica-00 aparecían como "idénticos" y no me dejaba abrir el Pull Request.

Cómo editar el archivo README.md

```
```

 Qué me devolvió
Me explicó paso a paso cada comando, por qué debía usarlo, y me ayudó a diagnosticar errores. Por ejemplo, cuando me salió el mensaje "There isn't anything to compare", me explicó que mi rama no tenía cambios porque solo la había creado pero no había editado archivos. Luego me recordó que debía hacer git add ., git commit y git push para ver los cambios reflejados en GitHub.

```javascript
```

 Qué estaba mal
El error más común fue que, al abrir el Pull Request, GitHub mostraba que main y feat/practica-00 eran idénticos. La razón fue que creé la rama pero no edité ningún archivo dentro de ella. Pensé que solo con crear la rama ya podía abrir el PR. No fue hasta que edité el README.md, guardé y subí los cambios que GitHub detectó diferencias y me permitió crear el PR.

También me costó entender que los cambios en Visual Studio Code no se ven automáticamente en GitHub hasta que no hago git push.

 Qué corregí y por qué
 No edité ningún archivo antes de intentar abrir el Pull Request.
 Edité el archivo README.md con mis datos personales, actualicé la tabla de progreso, hice git add ., git commit -m "docs(practica-00): completar perfil de jugador" y git push. Luego, GitHub ya detectó diferencias y pude abrir el PR sin problemas.

```javascript
```

 Qué escribí yo desde cero
No delegué completamente ninguna parte, porque fui yo quien escribió y guardó los cambios en VS Code. La IA me guió, pero yo ejecuté los comandos, edité el archivo manualmente y decidí qué datos poner en mi perfil. La IA me dio plantillas y sugerencias, pero yo ajusté los textos según mi experiencia personal.

 Reflexión
Definitivamente me ahorró tiempo. Sin la IA, probablemente habría pasado horas buscando en la documentación , y no habría entendido por qué no podía abrir el Pull Request. La IA me explicó el error paso a paso y en un lenguaje ás sencillo.

Lo que más valoro fue la ayuda para resolver el problema de "There isn't anything to compare". Aunque la guía de la misión decía que debía crear la rama y hacer el push, no mencionaba explícitamente que debía hacer cambios en la rama para que el PR tuviera sentido. La IA lo detectó rápido y me lo explicó con claridad.

Sí volvería a usarla para futuras misiones, pero tomando más en cuenta  que debo leer con atención los mensajes de error y no asumir que todo está bien. La IA me ayudó a aprender a leer mejor la terminal y los mensajes de GitHub.



 Misión 01: Ficha de personaje

 Herramientas que usé
Usé Deepseek desde la web, nada de GitHub Copilot ni Cursor.

 Qué le pedí a la IA
Le pedí ayuda con varias cosas: cómo hacer la estructura de una ficha de personaje en HTML sin usar CSS, cómo organizar el contenido con etiquetas como "header", "main", "section", "article" y "aside", y cómo corregir mis errores (como cambiar los "div" del formulario por `p`). También le pregunté cómo mantener el orden de los títulos ("h1", "h2", "h3"), 
cómo poner una imagen desde mi computadora y cómo validar el código en el W3C para que no tuviera errores.

 Qué me respondió
Me fue guiando paso a paso con ejemplos de código para cada parte: el nombre, la imagen con su "alt" bien descriptivo,
las tablas de estadísticas, la lista de habilidades, la historia con su jerarquía, y el formulario con sus etiquetas. 

Cuando tuve problemas con la imagen (no se veía porque era de Pinterest), me dijo que la descargara y la enlazara
desde mi carpeta con src="nanami.jpg". También me explicó que los "div" no estaban permitidos y que mejor usara "p" en
el formulario para cumplir con las reglas de la misión. Me recordó lo importante que es el "alt" y que cada "label"
tenga su "for" correspondiente.

 ¿Qué fue lo que falló?
Al principio, usé "div" en el formulario, pero la misión pedía usar elementos de sección en vez de "div". 
La imagen tampoco se veía porque el enlace de Pinterest estaba bloqueado, y yo pensaba que con solo poner la URL bastaba.
Además, el "alt" estaba incompleto (se quedaba en "vistiendo...") y no describía bien la imagen. 
También me faltaban filas en las tablas y el "tbody".

 Cómo lo arreglé y por qué
- Cambié los "div" del formulario por "p" para seguir la regla de usar elementos semánticos.
- Descargué la imagen y la guardé como "nanami.jpg" en la carpeta misiones/01-ficha-personaje/ , y cambié el "src" para 
que apunte a ese archivo. Así ya no dependo de internet.
- Completé el "alt" para que describa bien a Nanami: "Nanami Momozono, protagonista de Kamisama Kiss,
con cabello castaño claro y ojos marrones, vistiendo el uniforme escolar y sosteniendo un shikigami".
- Terminé las tablas con "thead" y "tbody", y puse todas las filas con sus atributos y niveles.
- Añadí "label" con "for" a cada campo del formulario ("nombre", "email", "asunto", "mensaje").

 ¿Qué hice yo por mi cuenta?
La IA me ayudó, pero yo hice varias cosas sola: elegí a Nanami Momozono de *Kamisama Kiss*, definí sus estadísticas 
como diosa y como humana, escribí su historia en tres párrafos (origen, desarrollo y madurez), elegí las habilidades 
y redacté sus descripciones. También descargué la imagen, la puse en la carpeta correcta, verifiqué que funcionara.

 Reflexión 
Esta misión me enseñó que HTML no es solo poner cosas en la pantalla, sino darle "sentido" a cada parte del contenido. 
La regla de no usar CSS no es un capricho, es como un examen: si tu página se ve ordenada sin estilos, significa que tu
estructura es buena. Si se ve como un bloque de texto plano, hay que mejorarla.

Aprendí que la semántica no es solo estética, es la base que usan los lectores de pantalla, los buscadores y 
los navegadores sin CSS. Una página bien hecha se ve decente incluso sin estilos, porque el navegador ya tiene sus 
propios estilos por defecto que respetan la estructura.

La verdad es que la IA me ahorró mucho tiempo. Sin ella, probablemente habría estado horas buscando en documentación y 
no habría entendido por qué mi imagen no se veía o por qué el formulario no cumplía con lo que pedían. La IA me explicó 
los errores paso a paso y con palabras más fáciles de entender.

Sí la volvería a usar para las próximas misiones, pero con más cuidado de leer bien los requisitos y no asumir que 
todo está bien solo porque parece que sí. La IA me ayudó a entender mejor los mensajes de error y a darme cuenta de 
que la estructura HTML es la base de todo.