# Sesión 16 · Git avanzado y colaboración real

**Módulo 5** · Zona 5: La Cumbre
**Lo que sale de esta noche:** un conflicto de fusión provocado a propósito y resuelto hablando con otra persona, tu equipo de tres armado y el tema del proyecto final definido
**Lo que entregas hoy:** Misión 07 — Marcador global con Next.js (100 XP · 9 h de trabajo autónomo) y la **Auditoría 4** cruzada (25 XP)

---

## Por qué la colaboración es el tema del cierre

Llegas a esta sesión con quince semanas de trabajo individual encima. Todo lo que hiciste hasta ahora fue en tu propio repositorio, en tu propia rama, sin nadie más tocando los archivos. Usaste Git como una máquina de guardar versiones, que es una fracción minúscula de lo que Git es. Esta noche Git deja de ser tu copia de seguridad y se convierte en lo que realmente es: **un protocolo de coordinación entre personas que no están en la misma habitación.**

En el papel el tema de hoy son ramas, fusiones, conflictos, `rebase` y revisión de código. Cinco comandos que están en la documentación y que puedes buscar cuando los necesites. El tema real es otro, y es el que le da nombre al módulo: **un conflicto de fusión es una conversación pendiente entre dos personas, y no un problema técnico que se resuelve solo con el teclado.** Los comandos están escritos en alguna parte. La conducta no está en ninguna documentación, y es lo que va a decidir si tu proyecto final en equipo de tres llega a puerto o se convierte en tres carpetas separadas.

Hay un detalle de calendario que hace esta noche particular: lo que se explica al principio lo vas a necesitar al final, cuando tu nombre esté al lado de dos nombres más en un repositorio compartido. Se entrega la Misión 07, se hace la auditoría cruzada y se forman los equipos, todo el mismo día. La teoría de la colaboración y el inicio de la colaboración real ocurren a dos horas de distancia.

Antes de seguir, una advertencia sobre cómo estudiar esto: si terminas la noche con quince invocaciones de `git` copiadas y nada más, no aprendiste el tema. La mitad valiosa es la de conducta — cómo se resuelve un conflicto, cómo se escribe un comentario en una revisión, cómo se recibe una crítica — y no se aprende memorizando comandos.

Un chiste que resume el módulo y que conviene tomarse en serio: **un commit que no está en el remoto no existe para nadie más que para tu disco duro.** Antes de dar por entregada la Misión 07, abre tu repositorio en el navegador y verifica que la rama está ahí. En sesiones anteriores hubo gente que juraba haber subido y tenía los commits solo en local.

---

## Ramas, fusiones y conflictos

Antes de leer lo que sigue, respondete esto: **un compañero y tú editan el mismo archivo, la misma línea, cada uno en su rama, y los dos hacen push. ¿Cuál de las dos versiones debería quedar, y cómo lo decide Git?**

Los criterios que se te van a ocurrir son razonables: la más nueva, la del que subió primero, la que tenga más líneas. Todos están equivocados por la misma razón, y esa razón es la respuesta del tema: **Git no lo decide, porque Git no puede saberlo.** Git sabe que dos personas cambiaron la misma línea. No sabe qué hace el programa, no sabe cuál de los dos cambios es el correcto, no sabe si las dos intenciones son compatibles. Lo único honesto que puede hacer es parar y avisar.

Vale la pena que te quedes con la frase exacta: *un conflicto no es un error de Git. Es Git negándose a inventar una respuesta que no tiene.*

### Qué es una rama, de verdad

Antes de los conflictos hay que desmontar la idea de rama como carpeta paralela, porque casi todo el mundo la tiene así.

Una rama en Git es un apuntador con nombre a un commit. Nada más. `main` es un archivo de treinta y nueve bytes que contiene el identificador de un commit. Crear una rama es escribir ese archivo, y por eso es instantáneo aunque el proyecto tenga cinco mil archivos: no se copia nada. Cuando haces un commit estando en esa rama, el apuntador avanza al commit nuevo.

La analogía que funciona: el historial del proyecto es una lista de reproducción de canciones donde cada canción apunta a la anterior, y una rama es un marcador de página, un adhesivo pegado en una canción concreta. Cambiar de rama no cambia la lista: mueve el adhesivo y pone en pantalla lo que corresponde a esa posición. Borrar una rama arranca el adhesivo, y las canciones siguen ahí. Por eso borrar una rama fusionada no pierde nada, y por eso borrar una rama sin fusionar sí puede dejar commits huérfanos.

De esa definición sale por qué la fusión es posible. Cuando fusionas, Git busca el commit del que salieron las dos ramas —el ancestro común— y compara: qué cambió en tu rama desde ese punto, y qué cambió en la otra. Si los cambios están en archivos distintos, o en líneas distintas del mismo archivo, los aplica los dos y no te pregunta nada. Esa es la mayoría de los casos, y es la razón de que la gente crea que fusionar es automático. Lo es, hasta que no.

### Los marcadores de conflicto, línea por línea

Cuando los dos cambios tocan la misma región, Git deja el archivo en un estado que hay que aprender a leer, porque es donde se produce el pánico. Este es el aspecto exacto que tiene, con código del tipo de juego que ya construiste:

```typescript
// src/combate.ts — así queda el archivo cuando hay conflicto
export function calcularDanio(ataque: number, defensa: number) {
<<<<<<< HEAD
  // Tu versión: la que ya tenías en tu rama actual.
  return Math.max(1, ataque - defensa);
=======
  // La versión que llega: la de la rama que estás fusionando.
  return Math.max(0, ataque - defensa * 0.5);
>>>>>>> rama-de-tu-companero
}
```

Los tres marcadores, uno por uno, porque la confusión de cuál es cuál es constante. Entre `<<<<<<< HEAD` y `=======` está **lo que tú tenías** en la rama donde estás parado. Entre `=======` y `>>>>>>>` está **lo que llega** de la otra rama, y a la derecha de las últimas flechas aparece de dónde llega. Los marcadores son texto que Git escribió en tu archivo: no son sintaxis de nada, son basura deliberada que rompe la compilación para que no puedas ignorarla.

Ese último punto explica el diseño. Git podría haber puesto el conflicto en un archivo aparte y dejar el tuyo compilando. No lo hace a propósito: **quiere que el proyecto esté roto hasta que un humano decida**, porque un conflicto sin resolver que compila es un conflicto que se va a subir sin resolver.

Resolver significa borrar los tres marcadores y dejar el código que debe quedar. Y ojo con esto, porque es lo que menos se espera: la solución correcta muchas veces **no es ninguna de las dos versiones**, es una tercera que combina las dos intenciones. En el ejemplo de arriba, si uno quería garantizar daño mínimo de 1 y el otro quería que la defensa mitigara a la mitad, la respuesta probablemente es `Math.max(1, ataque - defensa * 0.5)`, que no estaba en ningún lado. Eso solo se descubre entendiendo qué quería cada uno, y entender qué quería el otro es, literalmente, hablar con él.

### Lo que no se hace

Este es el párrafo más importante de la sesión, así que léelo despacio.

Lo que vas a sentir la primera vez que te pase, a las once de la noche, con el proyecto sin compilar, es la tentación de borrar el bloque del otro y quedarte con el tuyo. Los marcadores desaparecen, el proyecto compila, la sensación es de haber resuelto el problema. Y lo que acaba de pasar es que **deshiciste el trabajo de otra persona sin avisarle.** Tu compañero va a descubrir dentro de tres días que su cambio ya no está, y lo peor no es el tiempo perdido: lo peor es que el historial dice que él lo escribió y que tú lo resolviste, así que la conversación no empieza bien.

Piénsalo como un documento compartido: es como si dos personas escribieran un párrafo distinto en el mismo lugar de un informe y una de ellas borrara el del otro para que el formato quedara bonito. Nadie llamaría a eso resolver un conflicto de formato.

La regla operativa es corta: **si no sabes por qué el otro escribió eso, no lo borres, preguntale.** Un mensaje de tres líneas ahorra dos días.

Y de ahí sale una consecuencia de calendario. Por eso importa tanto que tus ramas sean cortas y que las sincronices con `main` seguido. Un conflicto de dos horas es una pregunta rápida entre dos personas que se acuerdan de lo que estaban haciendo. Un conflicto de dos semanas involucra doscientas líneas, tres archivos y dos personas que ya no recuerdan por qué tomaron esas decisiones. **El tamaño del conflicto no depende de lo difícil que sea el problema, sino de cuánto tiempo lo dejaste crecer.**

```bash
# La rutina que evita conflictos grandes. Todos los días, no una vez por semana.
git switch main
git pull                      # traer lo que otros subieron
git switch mi-rama
git merge main                # traer main a mi rama, no al contrario
# Si hay conflicto ahora, es pequeño y reciente. Ese es el punto.
```

### `rebase` frente a `merge`

Las dos operaciones sirven para lo mismo —incorporar el trabajo de una rama en otra— y producen historiales distintos.

`merge` crea un commit nuevo con dos padres y conserva la forma real de lo que pasó: dos líneas de trabajo que existieron en paralelo y se juntaron en un punto. El historial queda con la verdad histórica, incluidas sus bifurcaciones.

`rebase` hace otra cosa: toma tus commits, los levanta, actualiza la base de tu rama al último commit de `main` y vuelve a aplicar los tuyos encima, uno por uno. El resultado es un historial lineal, como si hubieras empezado tu trabajo hoy en vez de la semana pasada. Es más fácil de leer y es mentira, en el sentido de que reescribe cómo ocurrieron las cosas.

La analogía: `merge` es la fotografía del cruce de caminos, con las dos rutas visibles. `rebase` es volver a escribir el diario de viaje como si hubieras ido en línea recta. El diario reescrito se lee mejor. Si alguien más ya leyó el original y tomó notas, reescribirlo le arruina las notas.

De ahí sale la única regla dura de Git que conviene que memorices esta noche: **`rebase` solo sobre commits que nadie más ha visto.** Si tu rama solo está en tu máquina, o si es tuya y nadie más trabaja en ella, `rebase` para limpiar antes de abrir el pull request es una cortesía con quien va a revisar tu código. Si la rama es compartida o ya está en el remoto y alguien la descargó, `rebase` cambia los identificadores de los commits, y esa persona se encuentra con un historial que ya no coincide con el que tenía. La única salida entonces es un `push --force`, y esa es la forma más rápida de borrarle el trabajo a un compañero que existe en Git.

```bash
# Uso legítimo: limpiar MI rama antes de que alguien la revise.
git switch mi-rama
git rebase main               # mis commits quedan encima de main, historial lineal
# Si aparece conflicto, se resuelve commit por commit:
git add archivo-resuelto.ts
git rebase --continue
# Y si se complica de más, siempre hay salida:
git rebase --abort            # deja todo como estaba antes de empezar
```

Fíjate en `git rebase --abort`, porque es la razón de que puedas experimentar sin miedo. Mucha gente le tiene pánico a `rebase` porque nadie le dijo que hay un botón para deshacerlo.

Para tu proyecto final en equipo de tres, la política del curso te ahorra la discusión: `merge` para traer `main` a la rama de cada uno, `rebase` opcional para limpiar tu propia rama antes del pull request, y nunca `push --force` sobre una rama compartida. Anótala en algún lado, porque va a haber una noche en que la necesites y no te vas a acordar.

### Las ideas que hay que llevarse

**Un conflicto es una conversación pendiente, no un error.** Git no falló, ni tu compañero hizo algo mal, ni hay que arreglar nada antes de hablar. Dos personas cambiaron lo mismo porque las dos estaban trabajando, que es lo que se supone que pasa en un equipo. El único fallo posible aquí es resolverlo en silencio.

**El tamaño de un conflicto es función del tiempo, no de la dificultad.** Ramas cortas y sincronización diaria no son burocracia: son la única técnica conocida para que los conflictos sean pequeños. Todo lo demás son formas de sufrir con estilo.

**Una rama es un apuntador y el historial es inmutable, salvo cuando lo reescribes.** De esa frase sale todo: por qué crear ramas es gratis, por qué borrar una rama fusionada no pierde nada, por qué `rebase` es peligroso en compartido, y por qué una clave de API que entró en un commit sigue ahí después de "borrarla".

### Ponte a prueba

*Si crear una rama es instantáneo y no copia archivos, ¿por qué existe la costumbre de tener pocas ramas?* No existe: es una intuición traída de herramientas anteriores a Git, donde ramificar sí era caro. La respuesta correcta es que las ramas deben ser muchas y cortas. Si tu hábito es trabajar toda la semana en una sola rama gigante, viene de ahí y conviene desarmarlo.

*Si Git ve que los dos cambiamos la misma línea pero escribimos exactamente lo mismo, ¿hay conflicto?* No, porque el resultado es idéntico y no hay nada que decidir. El conflicto es sobre la ambigüedad, no sobre la coincidencia.

*¿Puede haber un conflicto que Git no detecte y que rompa el programa igual?* Sí, y es el caso más peligroso: alguien renombra una función en un archivo y tú agregas una llamada a la función vieja en otro. Git fusiona sin quejarse porque las líneas no se solapan, y el proyecto no compila. Se llama conflicto semántico, y es el argumento de por qué el pull request y el code review existen aunque la fusión sea limpia.

---

## Práctica en clase: provocar un conflicto a propósito

Este tramo es de teclado y es en parejas, con un repositorio compartido de verdad. No es una demostración que mires: vas a tener tu propio conflicto, con tu propio código, y lo vas a resolver hablando.

La razón de hacerlo así vale decirla antes de empezar, porque cambia la actitud con la que uno lo hace: **la primera vez que te pasa un conflicto de fusión conviene que no sea la primera vez.** Si te pasa hoy, con un archivo de juguete, con el docente caminando entre los puestos y tu compañero sentado al lado, aprendes el procedimiento sin adrenalina. Si te pasa por primera vez el sábado antes de la entrega del proyecto final, vas a aprender el procedimiento con adrenalina y probablemente le vas a borrar el trabajo a alguien.

Estos son los pasos, en orden, y están escritos para que puedas repetirlos por tu cuenta con cualquier compañero.

**Primero, el repositorio compartido.** Uno de los dos crea un repositorio en GitHub y agrega al otro como colaborador con permiso de escritura. El segundo acepta la invitación —hay que revisar el correo, y ahí se van cinco minutos— y los dos clonan.

```bash
# Los dos, cada uno en su máquina
git clone https://github.com/USUARIO/taller-conflicto.git
cd taller-conflicto
```

El que creó el repositorio escribe un archivo mínimo con contenido del juego, lo sube a `main`, y el otro hace `git pull`. Confirmen entre los dos que ven exactamente lo mismo antes de seguir; sin ese punto de partida común el ejercicio no funciona.

```typescript
// src/combate.ts
export function calcularDanio(ataque: number, defensa: number) {
  return ataque - defensa;
}
```

**Segundo, cada uno en su rama, y el mismo cambio.** Aquí viene la parte deliberada. Cada uno crea su rama y **modifica la misma línea, la del `return`**, con una regla de daño distinta. No copien las versiones de aquí: inventa tu propia regla de balanceo. Que las dos sean defendibles es lo que hace que la conversación de después sea real.

```bash
# Estudiante A
git switch -c balanceo-danio-minimo
# edita el return: Math.max(1, ataque - defensa)
git commit -am "fix(combate): garantizar daño mínimo de 1"
git push -u origin balanceo-danio-minimo

# Estudiante B, al mismo tiempo
git switch -c balanceo-defensa-parcial
# edita el MISMO return: ataque - defensa * 0.5
git commit -am "feat(combate): la defensa mitiga solo la mitad"
git push -u origin balanceo-defensa-parcial
```

Fíjate en la forma de esos mensajes de commit, porque es una convención que vas a usar el resto del curso: `tipo(alcance): descripción en imperativo`. Se llama Conventional Commits, cabe en una página de especificación, y se aprende usándola más que leyéndola. Úsala desde hoy.

**Tercero, el primer pull request pasa sin novedad.** El estudiante A abre su pull request contra `main`, GitHub dice que se puede fusionar sin conflictos, y lo fusiona. Todo bien, sensación de que Git es fácil.

**Cuarto, el segundo se encuentra el conflicto.** El estudiante B abre su pull request y GitHub muestra el aviso: no se puede fusionar automáticamente. Ese momento es el objetivo del ejercicio. Ahora B trae `main` a su rama en local, que es donde se resuelve de verdad:

```bash
# Estudiante B
git switch main
git pull                  # ya trae el cambio de A
git switch balanceo-defensa-parcial
git merge main            # aquí aparece el conflicto
# CONFLICT (content): Merge conflict in src/combate.ts
git status                # lee esto: dice exactamente qué archivo y qué hacer
```

Lee `git status` de verdad, no lo saltes. Es el comando más subestimado de Git y en un conflicto dice literalmente qué archivos están sin resolver y cuáles son los pasos siguientes. **La mitad del pánico con Git se cura leyendo lo que Git ya te está diciendo.**

**Quinto, la resolución hablando.** Aquí está la instrucción que hace que esto sea distinto de un tutorial: **B no puede tocar el archivo hasta haberle preguntado a A por qué escribió su versión.** En voz alta, girando la silla. Treinta segundos de conversación. Después resuelven juntos, y lo esperable es que la solución sea una tercera versión que combina las dos intenciones.

```typescript
// Resuelto: no es la versión de A ni la de B, es la que quieren los dos.
export function calcularDanio(ataque: number, defensa: number) {
  return Math.max(1, ataque - defensa * 0.5);
}
```

```bash
git add src/combate.ts
git commit                # Git propone un mensaje de merge; explica la decisión
git push
# El pull request en GitHub se actualiza solo y ya se puede fusionar.
```

**Sexto, mira qué pasa si subes los marcadores.** Vas a ver en vivo el resultado de "resolver" un conflicto borrando solo la línea `=======` y dejando las otras dos: el archivo queda en GitHub con los `<<<<<<< HEAD` visibles en la web y el proyecto no compila. Y el dato que conviene guardar: esto está en repositorios públicos de empresas reales, se busca en GitHub y aparecen miles de casos. Resolver conflictos con prisa y sin leer es un problema de la industria, no de estudiantes de cuarto semestre.

**Séptimo, cambien los papeles.** Repitan el ejercicio invirtiendo los roles, en otro archivo, para que los dos hayan estado en el lado incómodo. Si solo fusionaste sin conflicto, no aprendiste el procedimiento.

### Tres cosas que probablemente te confundan durante el ejercicio

El permiso de colaborador falla seguido. Casi siempre es que la invitación está en el correo sin aceptar, o que clonaron por HTTPS y no tienen credenciales configuradas. Salida rápida si se atasca: trabajen los dos sobre una sola máquina con dos carpetas clonadas del mismo remoto. El conflicto sale igual.

Te vas a preguntar por qué le toca a B resolver el conflicto y no a A, si A "no hizo nada mal". No es un castigo ni una asignación de culpa: es que `main` se movió después de que B empezó, así que a B le toca actualizarse. El que llega segundo se adapta, y a lo largo de un proyecto todos son el que llega segundo alguna vez.

Y vas a confundir el `merge` de `main` a tu rama con el de tu rama a `main`. Piénsalo por la dirección: traes `main` a tu rama para descubrir los conflictos en tu terreno, donde si rompes algo no molestas a nadie. Solo cuando tu rama está limpia y actualizada la fusionas hacia `main`. **`main` es lo que funciona; los experimentos se hacen en tu casa.**

Si además te aparece un conflicto durante un `rebase`, vas a ver que el mensaje es distinto y aparece varias veces. La diferencia es esta: en `merge` resuelves una vez, al final; en `rebase` resuelves commit por commit, porque Git está reaplicando tu trabajo en orden. Y ya sabes que existe `git rebase --abort`.

---

## El code review como práctica profesional

Antes de leer lo que sigue, respondete esto: **de las auditorías que hiciste este semestre, ¿cuál comentario que recibiste te sirvió de verdad, y cuál te molestó? ¿Qué tenían de distinto?**

Si lo pensás con calma, la diferencia casi nunca es la severidad —los comentarios duros pueden servir muchísimo— sino otra cosa: los que sirvieron hablaban del código y explicaban por qué; los que molestaron hablaban de la persona o eran veredictos sin argumento.

Esa distinción es todo el tema, y de ahí salen cuatro reglas.

### Se comenta el código, no a la persona

La formulación es sencilla y la diferencia es enorme. Compara las dos versiones del mismo hallazgo:

> *"No entendiste inmutabilidad."*
>
> *"Esta función muta el arreglo que recibe por parámetro, así que quien la llame va a ver su arreglo modificado sin esperarlo."*

Las dos señalan el mismo problema. La primera es un diagnóstico sobre una persona, es imposible de responder salvo defendiéndose, y no dice qué hacer. La segunda describe un hecho verificable sobre unas líneas concretas, cualquiera puede comprobarlo, y contiene implícitamente la corrección.

Un comentario de code review es un informe de laboratorio, no una calificación de conducta. El informe dice "esta muestra tiene tal concentración"; no dice "el técnico es descuidado". Y el informe se puede discutir con datos, que es justo lo que quieres que pase.

El efecto secundario es el argumento pragmático, por si el argumento humano no te convence: **un comentario sobre el código se arregla; un comentario sobre la persona se defiende.** Si tu objetivo es que el código mejore, el segundo tipo de comentario es simplemente menos eficaz.

### Regla y preferencia no son lo mismo, y hay que decir cuál es cuál

Esta es la que más discusiones inútiles ahorra en un equipo de tres.

Hay hallazgos que son reglas: esto rompe con un arreglo vacío, esta clave de API está en el cliente, esta función muta su entrada, este `useEffect` no limpia su suscripción. Son verificables y no se negocian.

Y hay hallazgos que son preferencias: yo habría usado `map` en vez de un bucle, yo pondría este componente en otro archivo, yo nombraría esta variable distinto. Pueden ser buenos consejos y no son obligaciones.

El error no es tener preferencias: es enunciarlas con el mismo tono que las reglas, porque entonces quien recibe la revisión no sabe qué es obligatorio y acaba haciendo todo o discutiendo todo. La solución es una palabra al principio del comentario:

> *"Preferencia mía, tómalo o déjalo: me parece que esto se leería mejor con un `filter`."*
>
> *"Esto sí hay que cambiarlo: si `enemigos` llega vacío, `enemigos[0]` es `undefined` y la línea siguiente lanza."*

Dos comentarios, dos pesos distintos, y quien los recibe sabe exactamente en qué gastar su tiempo. En un equipo de tres con una semana para entregar, eso es tiempo real.

### Preguntar antes de afirmar

Esta es la que convierte una revisión en una conversación. Compara otra vez:

> *"Esto falla con arreglos vacíos."*
>
> *"¿Qué pasa si `enemigos` llega vacío?"*

La afirmación asume que tienes razón. La pregunta averigua. Y la pregunta gana en los dos escenarios posibles: si tenías razón, el autor lo descubre él mismo, que es la forma en la que se aprende de verdad; y si no tenías razón —porque el caso ya estaba manejado tres líneas arriba, o porque por construcción el arreglo nunca llega vacío— acabas de aprender algo del código en vez de haber quedado en evidencia. **La pregunta te protege de tu propia lectura incompleta**, y en un code review tu lectura es siempre más incompleta que la de quien escribió el código.

Con un matiz de honestidad: hay hallazgos donde la pregunta es teatro. Si la clave de API está en un componente de cliente, no preguntes "¿has pensado dónde vive esta clave?". Eso es condescendiente. Dilo directo, con el argumento. La pregunta es la herramienta por defecto, no una obligación.

### Reconocer lo que está bien

Esta es la que más se salta la gente porque parece decorativa. No lo es, y por dos razones distintas.

La primera es que enseña. Si alguien encontró una forma elegante de derivar el estado en vez de duplicarlo, decírselo —y decir por qué está bien— hace que lo repita y que lo pueda explicar. **Una revisión que solo señala fallos transmite el catálogo de lo que no hay que hacer, y ese catálogo es infinito.** Señalar lo bueno transmite dirección.

La segunda es que sostiene la práctica. Una revisión que solo señala fallos desmotiva, y una persona desmotivada abre el siguiente pull request más tarde, más grande y con menos ganas de que lo revisen. La calidad de las revisiones determina la frecuencia de las revisiones. Es como un profesor que solo marca en rojo: produce estudiantes que entregan menos. No es psicología blanda, es un incentivo que se puede medir.

### Cómo recibir una crítica a tu propio código

El otro lado, que en un equipo de tres vas a necesitar igual de rápido.

Tu código no eres tú. Que alguien encuentre un caso borde que no cubriste no dice nada sobre tu capacidad; dice que una segunda persona leyó el código, que es exactamente para lo que se pide la revisión. La alternativa a que un compañero encuentre el bug es que lo encuentre un usuario, y esa alternativa es peor para todos, sobre todo para ti.

Tres conductas concretas. **Agradece los hallazgos**, aunque escuezan, porque cada uno es trabajo que alguien hizo gratis sobre tu código. **Responde con argumentos, no con explicaciones de tu proceso**: si crees que el revisor se equivoca, dilo y muestra por qué —"ese caso está cubierto en la línea 12"— en vez de contar cuánto te costó escribirlo. Y **si el comentario no se entiende, pregunta** en lugar de adivinar y hacer un cambio que nadie pidió.

La frase con la que conviene quedarse: **una revisión sin ningún comentario no es un elogio, es una revisión que no se hizo.**

### Las ideas que hay que llevarse

**El objeto de una revisión es el código; el sujeto es el equipo.** Se describe lo que hace el código, no lo que le pasa a la persona que lo escribió, y el propósito no es filtrar sino que los tres sepan lo que hay en el repositorio.

**Decir el peso de cada comentario es parte del comentario.** Regla, preferencia o pregunta. Sin esa etiqueta, quien recibe la revisión no puede priorizar, y una revisión que no se puede priorizar se ignora entera.

**Un hallazgo específico y verificable vale más que diez juicios generales.** "Esto no está bien estructurado" no es accionable. "Este componente maneja el estado del combate y además dibuja el inventario, así que no se puede reutilizar en la pantalla de tienda" sí lo es. Esta idea es literalmente el criterio de calificación de la Auditoría 4.

### Ponte a prueba

*¿Puede un code review detectar el conflicto semántico —el que Git fusiona sin quejarse y rompe el programa?* Sí, y es una de las razones principales de que exista. Un humano que lee el cambio completo ve que se renombró una función y que quedó una llamada al nombre viejo. Ninguna fusión automática ve eso.

*Si el código funciona y pasa las pruebas, ¿queda algo que revisar?* Queda casi todo: si alguien más lo puede mantener, si duplica algo que ya existía, si los nombres dicen la verdad, si los casos borde están cubiertos. "Funciona" son 35 de 100 XP en la rúbrica del curso, y esa proporción no es arbitraria.

*¿Cuál es el tamaño ideal de un pull request para que la revisión sirva?* Pequeño, y la razón es medible: una revisión de cuarenta líneas produce comentarios concretos; una de mil produce "se ve bien". Es el mismo argumento del tamaño de los conflictos, aplicado a la atención humana, y es el que hay que llevarse al proyecto final.

---

## Auditoría 4 · 25 XP

La regla que rige esta auditoría, y que conviene que te quede grabada: **se califica la calidad de TU revisión, no la calidad del código revisado.** Si te toca revisar un proyecto excelente, "no había nada que decir" no es un descargo, es el trabajo sin hacer.

El mínimo son comentarios sustantivos sobre el pull request de la Misión 07 del compañero que te asignaron, al menos dos, comentados línea por línea en GitHub y no en un documento aparte. Y sobre la escala, sin rodeos: un comentario que dice "está bien", "buen trabajo" o "no encontré nada" vale **cero**, sin importar que el código revisado sea excelente, porque no aporta información que el autor no tuviera. Un comentario que identifica un caso borde no cubierto, señala una duplicación de estado, encuentra la clave de API donde no debe estar o detecta que la salida del modelo entra al estado sin validarse vale **la auditoría completa**, porque es un hallazgo que el autor no tenía y sobre el que puede actuar. Entre esos dos extremos está todo lo demás, y el criterio es siempre el mismo: **¿el autor sabe algo que no sabía antes de leer tu comentario?**

Vale la pena que sepas por qué este instrumento existe y por qué es de los caros del semestre. **La Auditoría 4 es lo que evidencia el resultado de aprendizaje 3 del curso: integrarse en equipos de desarrollo.** La capacidad de trabajar en equipo no se puede evaluar con un examen individual, y tampoco con una nota de grupo, que premia al que se cuelga. Se evalúa mirando lo que una persona le aporta al trabajo de otra, y eso es exactamente un code review.

Cuatro cosas que te van a pasar mientras la haces. Vas a tener ganas de escribir puros elogios porque revisar a un compañero incomoda; recuerda que un hallazgo es un favor y que quedarse callado no es amabilidad. Vas a escribir reglas disfrazadas de preferencia y preferencias disfrazadas de regla; agrega la etiqueta al principio y léete el comentario en voz alta. Si te toca defenderte, vas a querer contar cómo lo escribiste en vez de señalar el código; vuelve a la línea concreta. Y vas a tender a comentar solo estilo —nombres, indentación, comillas— porque es lo más fácil de ver; empújate hacia la lógica preguntandote "¿qué pasa si esto recibe datos vacíos?", que es la pregunta que abre los hallazgos buenos.

Sobre la IA en las auditorías: pedirle a un asistente que revise el código de tu compañero es usar IA, así que **va declarado en `IA.md` como todo lo demás, y declararlo no baja la nota**. Lo que no declaras se califica en 0 sin reintento. Pero conviene que sepas qué vas a obtener: un asistente produce comentarios genéricos y plausibles —"considera manejar los errores", "podrías extraer esto a una función"— porque no tiene el contexto de la misión, ni la rúbrica, ni sabe qué se enseñó en la sesión 15. Esos comentarios son exactamente los que valen cero. La auditoría es el instrumento del curso que la IA hace peor que un estudiante que estuvo en clase, y eso debería decirte algo sobre para qué sirve venir.

---

## Tu equipo y tu proyecto final

Esta noche se forman los equipos: tres personas, ni dos ni cuatro, y una idea de juego por equipo. Cada equipo va a decir en voz alta tres cosas: quiénes lo forman, qué juego van a hacer y **qué es lo que lo hace difícil**. Esa tercera es la importante y conviene que te dé un poco de miedo: si nadie del equipo sabe qué parte va a costar, todavía no tienen un proyecto, tienen un título.

Tres criterios para juzgar tu propio tema. Que sea un juego que puedan terminar, porque un proyecto ambicioso a medias vale menos que uno modesto completo. Que tenga algo que persista —marcador, progreso, partidas guardadas— porque si no, no vas a poder demostrar la mitad de lo que aprendiste. Y que el trabajo se pueda partir en tres pedazos que avancen en paralelo sin pisarse, porque si los tres tienen que editar el mismo archivo para hacer cualquier cosa, van a tener un conflicto de fusión por hora y ya sabes lo que eso significa. Ese último criterio es la conexión directa con lo de esta noche: **la arquitectura de un proyecto determina cuántos conflictos va a tener el equipo.** No es casualidad que la sesión de conflictos y la de formación de equipos sean la misma.

La rúbrica de misión rige también para el proyecto final: funciona (35), calidad del código (25), específicos del módulo (20), proceso en Git (10) y auditoría de IA (10). Detente en esos diez de proceso en Git, que hasta ahora eran los más fáciles de conseguir trabajando solo y ahora son los que van a doler: commits atómicos con mensajes convencionales, ramas por funcionalidad, pull requests revisados por otro miembro del equipo y un `main` que siempre compila. En el proyecto final el historial de Git se lee como un documento, y un historial con tres commits llamados "cambios" cuenta una historia muy clara sobre cómo trabajó ese equipo.

Dos frases para cerrar. **Un conflicto no lo resuelve el que edita el archivo, lo resuelve el que habla con la otra persona.** Y la del módulo: hasta hoy Git fue tu cuaderno de apuntes; desde esta noche es el lugar donde tres personas se ponen de acuerdo, y esa es la única habilidad de todo el semestre que no se puede practicar solo.

Estamos en la sesión 16 de 21. Lo que queda es el proyecto final y el despliegue, es decir: lo único que vas a poder mostrarle a alguien.

---

## Errores que probablemente vas a cometer

**Resolver el conflicto borrando el cambio del otro para que el proyecto compile.** Es el error más caro de la noche y no se comete por ignorancia técnica sino por prisa: el archivo está roto, los marcadores molestan, tu versión se entiende y la del otro no, y borrarla hace que todo vuelva a funcionar. El daño es doble. Se pierde trabajo ajeno, y se pierde en silencio, porque el historial registra que el conflicto fue resuelto y no que una de las dos intenciones desapareció. Tu compañero lo descubre días después, cuando su funcionalidad ya no está y nadie sabe cuándo dejó de estar. La corrección no es un comando: es preguntar antes de borrar. Y si el argumento humano no te alcanza, toma el egoísta: la próxima vez el que llega segundo vas a ser tú.

**Trabajar semanas en una sola rama gigante y fusionar al final.** Viene directamente de tu costumbre individual de todo el semestre, donde la rama es un trámite y no una unidad de trabajo. En equipo el resultado es predecible: al final del proyecto hay una fusión con conflictos en ocho archivos, ninguno de los tres recuerda por qué tomó cada decisión, y la resolución se hace a las dos de la mañana escogiendo versiones al azar hasta que compila. El tamaño del conflicto no depende de la dificultad del problema sino del tiempo transcurrido, y la técnica de prevención es aburrida y funciona: ramas de dos días, `git merge main` a diario, pull requests pequeños. En el proyecto final la rúbrica exige que el historial muestre ramas cortas.

**Subir los marcadores de conflicto sin darte cuenta.** Resuelves a medias, borras uno de los tres marcadores, no vuelves a compilar, haces commit con `-am` sin mirar el diff y subes. El archivo queda en el repositorio con `<<<<<<< HEAD` dentro, la aplicación no arranca para nadie y el mensaje del commit dice "resolver conflicto". La causa raíz no es el conflicto: es hacer commit sin leer lo que estás subiendo, un hábito que trabajando solo solo produce commits desordenados y en equipo rompe el trabajo de los demás. La rutina correcta después de cualquier resolución son tres pasos —`git diff` para leer lo que quedó, compilar, y solo entonces commit— y como comprobación final busca la cadena `<<<<<<< HEAD` en tu propio repositorio.

**Confundir el code review con un juicio sobre la persona, en cualquiera de los dos papeles.** Como revisor aparece así: comentás sobre quien escribió en vez de sobre lo escrito, o te incomoda señalarle algo a un compañero y lo resuelves escribiendo "todo bien", perdiendo los 25 XP. Como autor aparece como defensa: respondés al hallazgo contando el esfuerzo que te costó, o haces todos los cambios sin discutir ninguno, incluidos los que eran preferencias del revisor. Las dos formas tienen la misma causa, que es tratar el código como una extensión de tu identidad, y las dos se corrigen con la misma frase: la revisión mira el código, y el código no es nadie. Operativamente ayuda mucho obligarte a citar una línea concreta y decir si es regla o preferencia, porque un comentario anclado a una línea es casi imposible de escribir en segunda persona.

---

## Fuentes de esta sesión

- Chacon, S., & Straub, B. *Pro Git* (2ª ed.). https://git-scm.com/book/en/v2
- GitHub. *About pull requests*. https://docs.github.com/en/pull-requests/reference/pull-requests
- Conventional Commits 1.0.0. https://www.conventionalcommits.org/en/v1.0.0/
- Preston-Werner, T. *Semantic Versioning 2.0.0*. https://semver.org/

*Pro Git* es la fuente de esta noche y vale saber cómo leerla: los capítulos 3.1 a 3.3 explican ramas y fusiones con los mejores diagramas que existen sobre el tema, y el capítulo 7 tiene el material de `rebase` con las advertencias incluidas. Es un libro gratuito, escrito por gente del proyecto, y está traducido al español. Es la única fuente de Git que necesitas: la mayoría de los tutoriales de internet son una lista de comandos sin el modelo mental que hace que los comandos tengan sentido.

La documentación de GitHub sobre pull requests es referencia operativa y no lectura: ahí está cómo comentar una línea, cómo sugerir un cambio y cómo funcionan las revisiones aprobadas o con cambios solicitados. Te va a servir esta noche mientras haces la Auditoría 4.

Conventional Commits y Semantic Versioning son las dos convenciones que vas a usar en el proyecto final. Conventional Commits ya la aplicaste en la práctica —`fix(combate): ...`— y merece la pena que veas la especificación completa, que cabe en una página. Semantic Versioning es tema de la sesión 17 y hoy solo queda sembrado: los tres números de una versión no son decoración, son una promesa sobre qué se rompe al actualizar.

---

## Antes de la sesión 17

Lee la especificación de Semantic Versioning completa, que son diez minutos, y la sección de Conventional Commits que explica cómo se marca un cambio que rompe compatibilidad. Llega con esta pregunta contestada en la cabeza: si tu juego pasa de la versión 1.4.2 a la 2.0.0, ¿qué le estás diciendo exactamente a quien lo estaba usando? Ese es el punto de partida de la sesión 17, donde el tema es el despliegue y qué significa publicar algo que otra gente ya tiene funcionando.
