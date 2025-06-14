# Algoritmo para Emparejamiento en Torneos de Dominó Profesional con Sistema Suizo en Parejas Rotativas

---

## Contexto

En el sistema de dominó profesional con SvelteKit y Svelte 5 que estoy desarrollando sobre gestión de torneos y manejo de rankings, la modalidad más utilizada es el sistema suizo. Los torneos a nivel internacional con ELO se juegan siempre en parejas rotativas, es decir, la clasificación es individual pero en cada ronda se empareja con un compañero nuevo, lo mismo sucede con los rivales. Necesito que me ayudes a desarrollar el algoritmo, pero no en código de programación todavía, ya que la idea es entenderlo primero antes de plasmarlo en código, sobretodo porque el sistema será auditado y debe conservar su naturaleza en lenguaje humano para que los auditores de la federación puedan aprobarlo. 

El sistema suizo se caracteriza por ordenar en pirámides, pero como el dominó profesional e internacional se juega en parejas, supongo que tendremos que manejar 2 pirámides, pirámide principal (PP) y pirámide secundaria (PS). En todo torneo élite de esta categoría la cantidad ideal de participantes suele ser 28 atletas como mínimo. Entonces tratemos de crear un esquema sobre cómo se jugaría ronda por ronda y como prioridad el algoritmo o esquema debe ser escalable de tal modo que conserve el mismo patrón para cuando haya una mayor participación de atletas.

---

## Primera Ronda

El administrador o anfitrión del torneo decidirá si la primera ronda se juega ordenando por ELO o aleatoriamente, pero en este algoritmo en específico no quiero tomar en cuenta el ELO aún. Partamos en que todos tienen nivel similar y por lo tanto en la primera ronda se emparejarán compañeros y rivales de manera aleatoria. Por lo que las parejas quedarían del siguiente modo:

- **A1 – A2 vs B1 - B2**
- **C1 - C2 vs D1 - D2**
- **E1 - E2 vs F1 - F2**
- **G1 - G2 vs H1 - H2**
- **I1 - I2 vs J1 - J2**
- **K1 - K2 vs L1 - L2**
- **M1 - M2 vs N1 - N2**

Los ganadores de la primera ronda fueron:

- **A1 - A2** 
- **C1 - C2** 
- **E1 - E2** 
- **G1 - G2** 
- **I1 - I2** 
- **K1 - K2** 
- **M1 - M2**

---

## Criterios de Desempate y Ordenamiento

Después de aplicar criterios de desempate con índices como win rate "WR2", efectividad "EFF" (fórmula compuesta que mide el dominio) y otros indices discriminatorios secundarios, ordenamos la lista de ganadores según sus índices de desempeño en forma descendente:

- **A1, A2, C1, C2, M1, M2, G1, G2, I1, I2, E1, E2, K1, K2**

Se sobreentiende que A1 y A2 tienen los mismos valores ya que fueron pareja y han jugado apenas una ronda, por eso están empatados, el mismo ejemplo aplica para los demás también.

---

## División en Pirámides

Dividimos la lista total de jugadores en una primera mitad (PP) con los atletas con puntajes mas sobresalientes y estos a su vez se subdividen entre lados (los lados de la piramide), es decir, en 2 grupos:

- **Lista ganadores con mejor desempeño (Lado A):**  
  A1, A2, C1, C2, M1, M2, G1

- **Lista de ganadores con más bajo desempeño (Lado B):**  
  G2, I1, I2, E1, E2, K1, K2

---

## Reglas Obligatorias

1. **Regla 1:** Un atleta no puede volver a coincidir con un excompañero hasta al menos 3 rondas posteriores. Es decir, si fueron pareja en ronda 5 entonces en ronda 6 y ronda 7 no se pueden volver a emparejar como compañeros de equipo, en ronda 8 sí se podría ser flexible y sí podrían volver a coincidir, aunque dependerá de la cantidad de participantes. A mayor participación de jugadores en el torneo será más fácil evitar coincidencias pero por eso se debe ser flexible en caso que se trate de un torneo pequeño.  
   *Ejemplo:* En un hipotético torneo de 9 rondas, el jugador A1 después de ronda 1 no podrá volver a coincidir como compañero con el jugador A2 hasta la ronda 4 y ronda 7.

2. **Regla 2:** Un atleta no puede enfrentar a ninguno de los rivales de la ronda actual hasta por lo menos 3 rondas posteriores. Es decir, si fueron rivales en ronda 5 entonces en ronda 6 y ronda 7 no se pueden volver a enfrentar como rivales de equipo, en ronda 8 sí se podría ser flexible y sí podrían volver a coincidir aunque lo ideal sería evitarlo.  
   *Ejemplo:* En un hipotético torneo de 9 rondas, el jugador A1 después de ronda 1 no podrá volver a enfrentar como rivales contra los jugadores B1 y B2 hasta la ronda 4 y ronda 7.

---

## Emparejamiento de Rondas Posteriores

Para mantener la dinamica del algoritmo suizo se debe evitar repetir compañeros y también evitar rivales en las 2 subsiguientes rondas. Con respecto a compañeros, el Atleta ya sea del lado A o del lado B se debe emparejar siempre con otro atleta el cual se encuentre mas contiguo a su posicion (ya que quiere decir que tienen nivel o puntaje similar), siempre y cuando se respeten las reglas 1 y 2 por lo que tendremos que aplicar condicionales basadas en esos criterios. Con respecto a los rivales, el sistema debe ser más riguroso aún y tratar en lo posible de que nunca se vuelvan a repetir rivales.

Ejemplo: En el lado A , comenzamos los emparejamientos de manera descendente, es decir, el primer emparejamiento seria el lider quien es A1 debemos emparejarlo con el jugador mas contiguo quien es A2 pero antes aplicamos condicionales y segun las reglas A1 y A2 coincidieron recientemente en la ronda anterior como compañeros, no pueden volver a repetir , entonces saltamos a A2 y buscamos al siguiente contiguo quien seria C1, aplicamos las condicionales y con C1 no hay coincidencias por lo tanto A1 sera compañero de C1. El mismo ejemplo aplica para confrontar rivales ya que la dinamica de decisiones seria la misma.   

---

## Determinación de las Parejas de Cada Lado

Antes de emparejar debemos definir cuántas parejas contendrá cada lado de la pirámide, sobretodo el primer lado, para ello dividimos el número total integrante del “lado A” entre 2, si el resultado es un número con decimales entonces restamos -0.50, para este caso el resultado de dividir 7 entre 2 nos dio un valor con decimales "3.50", entonces procedemos restando los -0.50 para obtener un total de 3 parejas para este “lado A”.

Al ser 28 atletas encontramos el primer inconveniente, si dividimos 28/4 nos dará un resultado de 7 lo que quiere decir que son 7 mesas, un número impar de mesas, por lo tanto en el ordenamiento debemos ser cuidadosos a la hora de organizar, y debemos hallar alguna fórmula como por ejemplo emparejar al penultimo del Lado A con el primer lugar del Lado B y al ultimo del Lado A con el sgundo del lado B,  En cambio, cuando las mesas son pares por ejemplo 8 mesas, se facilita el ordentamiento.

---

## Emparejamiento de Ganadores

Procedemos a emparejar ganadores:

- **Lado A PP:** A1 – C1, A2 – C2, M1 – G1. (cumplimos con el criterio de tener 3 parejas en el Lado A)
- **Lado B (Segundo lote de la pirámide principal):** M2 - I1, G2 – I2, E1 – K1, E2 – K2

Y procedemos a crear las mesas cerrando los lados de la piramide generando asi los siguientes enfrentamientos:

- **A1 – C1 vs M2 - I1**
- **A2 – C2 vs G2 – I2**
- **M1 – G1 vs E1 – K1**

Nos sobraría la pareja **E2 – K2**, esto ocurre por la misma razón que tenemos mesas impares. Es probable que con mesas con un número par no ocurra y el ordenamiento sea más sencillo. En este caso, al ser una pareja de la piramide principal quienes quedaron a la deriva, pues los vamos a desemparejar para luego emparejarlos sí o sí con los mejores integrantes de la piramide secundaria, del mismo modo los enfrentaremos también contra la pareja más fuerte del lado B de la piramide secundaria, para así respetar la esencia del sistema suizo, esto se debe lograr siempre y cuando compañeros y rivales no se hayan repetido en la ronda anterior o ronda previa a la anterior.

Luego aplicamos la misma dinámica que se utilizó en PP para la PS tambien.

---

## Emparejamiento de PS

Lista total de la piramide secundaria ya ordenada segun sus posiciones correspondientes:  
**L1, L2, F1, F2, J1, J2, H1, H2, N1, N2, D1, D2, B1, B2**

En teoría, en primera ronda L1 y L2 deberían ser los más fuertes, a pesar que perdieron deberían tener un mejor índice de efectividad ya que fueron quienes lograron bajar la calidad de victoria para K1 y K2 dejándolos como los últimos lugares de la piramide principal.

Dividimos:  
Recordemos que tenemos en lista de espera a **E2** y **K2**, quienes por lógica deben ser los líderes de la PS pero a la misma vez tambien debemos ser equitativos, y seria injusto emparejarlos como compañeros del lado A para enfrentar a la pareja lider del lado B la cual en teoria son mas debiles por amplio margen, lo que contradeciria la esencia de un sistema suizo, por lo tanto lo que haremos sera separarlos para que ambos sean lideres en cada lado de la piramide y asi se enfrenten entre ellos mismos, lo que representaria un enfrentamiento mas equilibrado. Totalizamos jugadores de la PS y dividimos en partes iguales y llenamos cada lado segun su posicion correspondiente.

- **Lado A PS:** E2, L1, L2, F1, F2, J1, J2, H1
- **Lado B PS:** K2, H2, N1, N2, D1, D2, B1, B2

Emparejamos del mismo modo como se hizo en PP, por lo tanto obtenemos:  
**Lado A PS:** E2 – L1, L2 – F1, F2 – J1, J2 – H1
**Lado B PS:** K2 – H2, N1 – D1, N2 – D2, B1 – B2 (eventualidad inesperada)

**Caso Excepcional:** Despues del ordenamiento B1 y B2 quedaron como compañeros pero esto rompe la condicionales ya que en ronda anterior fueron pareja y por lo tanto NO pueden volver a coincidir, en una situacion asi el algoritmo debe ser capaz de identificar el problema y buscar a la pareja mas contigua (de nivel similar) y realizar un reordenamiento, la pareja de nivel mas similar en este caso seria N2 y D2, invertimos de manera reciproca la posicion de los atletas mas contiguos en cuestion quienes serian (D2 y B1), obteniendo el siguiente fragmento ordenado "N2, B1, D2, B2" , aplicamos los condicionales respectivas para confirmar que los emparejamientos si sean posibles, si condicionales se rompen de nuevo entonces repetimos ciclo otra vez con la pareja mas contigua de derecha a izquierda (orden ascendente) sino finalmente obtenemos el lado B:
**Lado B PS:** K2 – H2, N1 – D1, N2 – B1, D2 – B2

Cerramos los lados de la pirámide y creamos las mesas con sus respectivos enfrentamientos:

- **E2 – L1 vs K2 – H2**
- **L2 – F1 vs N1 – D1**
- **F2 – J1 vs N2 – B1**
- **J2 – H1 vs D2 – B2**

---

## Ejemplo de Aplicación de Condicionales

- ¿E2 fue compañero de L1? = No (su compañero fue E1)  

- ¿E2 fue rival de H2 y k2? = No (sus rivales fueron F1 y F2)
  
→ Entonces seguimos adelante, ejecutamos.

---

## Enumeración de Mesas

Finalmente ya tenemos nuestras mesas listas para la ronda 2, dichas mesas deben ser enumeradas por el algoritmo de manera ascendente en donde siempre la pareja líder de la pirámide con mejor desempeño ocupará la “Mesa 1”, eso es regla elemental en todos los torneos profesionales de dominó:

- **Mesa 1:** A1 – C1 vs M2 - I1
- **Mesa 2:** A2 – C2 vs G2 – I2
- **Mesa 3:** M1 – G1 vs E1 – K1
- **Mesa 4:** E2 – L1 vs K2 – H2
- **Mesa 5:** L2 – F1 vs N1 – D1
- **Mesa 6:** F2 – J1 vs N2 – B1
- **Mesa 7:** J2 – H1 vs D2 – B2

---

## Manejo de Inasistencias y "Byes"

El algoritmo obligatoriamente debe contemplar el siguiente escenario:
- En caso de inasistencia antes del torneo o abandono durante el desarrollo, entonces utilizar "byes":  
  Si hay un número impar de parejas, asignar un bye (descanso) a los atletas con menor desempeño que no logren completar una mesa completa de 4 integrantes.

---

## Propuesta para Reglas 1 y 2 según Categoría

- **CATEGORÍA COMPACTA (≤ 36 participantes):**
  - Compañeros: ventana de +3 ronda
  - Rivales: ventana de +3 rondas
  - Riesgo distorsión: ¿quizás ALTO si aplicamos más restricciones?

- **CATEGORÍA ESTÁNDAR (40-76 participantes):**
  - Compañeros: ventana de +4 rondas
  - Rivales: ventana de +4 rondas
  - Riesgo distorsión: ¿quizás MEDIO-BAJO con parejas en bucle infinito por espera de pareo?

- **CATEGORÍA INTERNACIONAL (80+ participantes):**
  - Compañeros: ventana de +5 rondas
  - Rivales: ventana de +5 rondas
  - Riesgo distorsión: ¿quizás MÍNIMO sin alteraciones y todos los jugadores dentro de su respectiva pirámide según corresponda?

---

## Posibles Problemas: Efecto Cascada

- Si una pareja líder en PP no encuentra rivales, debe bajar hasta pirámide secundaria y este suceso distorsiona completamente el sistema suizo.
- Crea "emparejamientos forzados" no competitivos.

---

## Dudas y Ajustes

Listo, hemos culminado. Ahora viene mi gran duda:  
¿Este algoritmo es escalable? ¿Funcionará el algoritmo para las siguientes rondas? Por ejemplo, si se juega un torneo de 9 rondas.  
Si el algoritmo requiere ajustes, entonces ¿cuáles ajustes son y cómo implementarlos?

---

## Flexibilidad y Mandamientos Excepcionales

En caso extremo que el algoritmo no logre encontrar pareja válida, lo idóneo es que no se pierda la base y estructura del “sistema suizo”. Por ende, se podría estudiar la posibilidad de aplicar mandamientos excepcionales tales como:

- **Primero:** Realizar una transfusión de atletas entre pirámides, los últimos lugares de la pirámide principal intercambiados con los primeros lugares de la segunda pirámide. Se debe ser estrictamente selectivo y excluyente para no generar distorsiones.
- **Segundo:** Con respecto a las reglas 1 y 2, se podría aplicar una relajación progresiva en cuanto a las ventanas, como por ejemplo disminuir una ronda; es decir, en el escenario de “categoría estándar” se podría aplicar -1 ronda y las ventanas pasarían de ser +3/+3 a +2/+2.

Pero dejando siempre claro que lo óptimo sería que el algoritmo funcionara sin estos mandamientos, que sean solo una utilidad de último recurso.

---

## Aclaración Sobre las Pirámides

Por último, las pirámides no se tratan sobre perdedores o ganadores de la ronda. En realidad, en cada ronda varía: nuestra base de datos ya contempla una tabla de clasificacion con las posiciones de los atletas enumeradas, de alli se toma la lista completa del torneo en forma descendente en base a sus índices WR, EFF entre otros mas, y a partir de dicho listado, ronda tras ronda, se divide y se forman dos nuevas pirámides. Es decir, te doy un ejemplo: el jugador A1 en la ronda 4 puede haber ganado tres veces consecutivas y haber perdido en la última ronda, y por haber perdido en la última ronda no quiere decir que automáticamente pasa a la pirámide secundaria, ya que por clasificación global es probable que aún vaya de líder a pesar de su única derrota; entonces debería estar en la primera pirámide e incluso como líder de todos aún, suponiendo que en la tabla general así se refleje.

En referencia a los indices que influyen como criterio de desempate y demas indices que no hacen falta mencionar ni describir, se debe hacer enfasis en que el algoritmo suizo no debe evaluar dichos criterios, ya que no es su funcion, la funcion real que queremos es la de gestionar emparejamientos segun la esencia de la modalidad en si. En teoria el algoritmo recibe ronda tras ronda ya la lista ordenada con sus posiciones respectivas desde la tabla de clasificacion.

---

## Conclusiones y peticion a la IA

Dicho esto, como la IA con mayor potencial del mercado que eres tú, por supuesto debes profundizar con rigurosidad y desarrollar el algoritmo por mí, justo esa es la idea, por eso he tratado de darte el contexto necesario. El algoritmo que tú me generes debe tener la lógica y coherencia desde la ronda 1 hasta la ronda 9. No me decepciones ya que el algoritmo será auditado y no quiero hacer el ridículo por tu culpa.

Con el algoritmo ya finalizado debes hacer una simulación interna para tu propio consumo, no a nivel visual, ni generar archivos extras, la simulación debe ser mentalmente para ti mismo como IA para que luego me indiques si se presentaron vicisitudes durante la simulación y sobre cuáles serían los posibles retos inesperados a presentarse, ya que necesitamos prever problemas para los debidos mantenimientos preventivos y no correctivos. Este algoritmo debe ser versátil y adaptable para poder superar cualquier prueba con rotundo éxito.

En caso de ser necesario, debes generar 2 archivos, uno con el algoritmo en pseudocódigo tecnico y otro archivo en formato markdown explicando dicho algoritmo en un lenguaje más natural para humanos. La creación de 2 archivos es opcional, al menos que quieras crear todo dentro de un solo archivo. ¡Sorpréndeme!

Importante: NO generar ningun archivo hasta que la simulacion haya culminado con exito, repetir simulaciones necesarias hasta lograr el exito esperado que concluya que nuestro algoritmo es optimo, luego si generar el o los archivo(s).

---

**Nota:** Si necesitas más contexto me puedes preguntar y yo te voy a facilitar otro prompt contextual adicional para abordar la eventualidad.