# **Algoritmo para Emparejamiento en Torneos de Dominó Profesional con Sistema Suizo en Parejas Rotativas**

---

---

## **Contexto**

En el sistema de dominó profesional con **SvelteKit** y **Svelte 5** que estoy desarrollando sobre gestión de torneos y manejo de rankings, la modalidad más utilizada es el **sistema suizo**. Los torneos a nivel internacional con **ELO** se juegan siempre en **parejas rotativas**, es decir, la clasificación es individual pero en cada ronda se empareja con un compañero nuevo, lo mismo sucede con los rivales.

Necesito que me ayudes a desarrollar el algoritmo, pero no en código de programación todavía, ya que la idea es entenderlo primero antes de plasmarlo en código, sobre todo porque el sistema será auditado y debe conservar su naturaleza en lenguaje humano para que los auditores de la federación puedan aprobarlo. 

### **Reglas Fundamentales**

Es vital subrayar que **en dominó profesional no existen empates**. Cada partida arroja obligatoriamente un ganador y un perdedor. Una partida equivale a una ronda y se establece un puntaje objetivo que, a nivel internacional y profesional, suelen ser **200 puntos**. La pareja que logre alcanzar o incluso superar dicha cifra será la pareja ganadora.

**Importante:** Existe un límite de tiempo por ronda. Si una mesa demora más del tiempo estipulado, gana quien tenga más puntos en ese preciso instante, indistintamente de si no llegaron a alcanzar el puntaje objetivo. En caso de empate, se juega una sola mano adicional para poder desempatar.

### **Criterios de Clasificación**

A lo largo de todo el torneo, el **número total de victorias** actúa como criterio primario e inapelable para ordenar la lista descendente que el algoritmo recibe en cada ronda. Los índices auxiliares calculados por la tabla de clasificación (**WR2, EFF** y otros) solo se emplean como desempate **secundario** si dos atletas acumulan exactamente la misma cantidad de victorias.

### **Alcance del Algoritmo**

En referencia a los índices que influyen como criterio de desempate y demás índices, se debe hacer énfasis en que el **algoritmo suizo no debe evaluar dichos criterios**, ya que no es su función. La función real que queremos es la de gestionar emparejamientos y enfrentamientos según la esencia de la modalidad en sí.

En teoría, el algoritmo recibe ronda tras ronda la lista ya ordenada con sus posiciones respectivas desde la tabla de clasificación.

### **Escalabilidad del Torneo**

En todo torneo élite de esta categoría, la cantidad ideal de participantes suele ser **28 atletas como mínimo**. A continuación, crearemos un esquema sobre cómo se jugaría ronda por ronda, con la prioridad de que el algoritmo o esquema sea escalable para conservar el mismo patrón incluso con una mayor participación de atletas.

---

## **Primera Ronda**

### **Configuración Inicial**

El administrador o anfitrión del torneo decidirá si la primera ronda se juega ordenando por **ELO** o aleatoriamente. En este algoritmo en específico, no tomaremos en cuenta el **ELO** aún. Partiremos de la base de que todos los participantes tienen un nivel similar y, por lo tanto, en la primera ronda se emparejarán compañeros y rivales de manera aleatoria.

### **Lista de Participantes**

Recibimos desde la tabla de clasificación la siguiente lista desordenada de atletas según su inscripción:

```
M2, K1, D2, F1, G1, I2, E1, N2, C2, L1, B1, J2, H1, K2, B2, D1, A2, E2, L2, F2, I1, G2, C1, N1, J1, A1, M1, H2
```

### **Proceso de Ordenamiento Aleatorio**

Aplicamos un algoritmo que posicione a los atletas al azar sin ningún criterio en particular. Supongamos que el listado queda de la siguiente manera:

```
A1, A2, C1, C2, E1, E2, G1, G2, I1, I2, K1, K2, M1, M2, B1, B2, D1, D2, F1, F2, H1, H2, J1, J2, L1, L2, N1, N2
```

> **Nota:** Este orden aleatorio solo ocurre en la primera ronda.

### **Aplicación de las Reglas de Emparejamiento**

Aplicamos la **Regla 1*** (descrita más adelante) para emparejar compañeros y plegamos el bloque para confrontar paralelamente ambas mitades, respetando la **Regla 2*** (descrita más adelante).

**Emparejamientos resultantes:**

| Mesa | Pareja 1 | vs | Pareja 2 |
|------|----------|----|----------|
| 1    | A1 - A2  | vs | B1 - B2  |
| 2    | C1 - C2  | vs | D1 - D2  |
| 3    | E1 - E2  | vs | F1 - F2  |
| 4    | G1 - G2  | vs | H1 - H2  |
| 5    | I1 - I2  | vs | J1 - J2  |
| 6    | K1 - K2  | vs | L1 - L2  |
| 7    | M1 - M2  | vs | N1 - N2  |

### **Resultados de la Primera Ronda**

Los ganadores de la primera ronda fueron:

- **Mesa 1:** A1 - A2
- **Mesa 2:** C1 - C2
- **Mesa 3:** E1 - E2
- **Mesa 4:** G1 - G2
- **Mesa 5:** I1 - I2
- **Mesa 6:** K1 - K2
- **Mesa 7:** M1 - M2

---

## **Criterios de Desempate y Ordenamiento**

### **Proceso de Ordenamiento Inicial**

Después de que la tabla de clasificación aplica criterios de desempate con índices discriminatorios secundarios, el algoritmo recibe la lista completa ordenada en forma descendente con todos los atletas según su eficiencia y desempeño ronda tras ronda. Esto es necesario porque pueden existir atletas empatados en número de victorias, pero con diferentes posiciones debido a los puntajes obtenidos (puntos a favor y en contra por partida).

### **Lista Ordenada Recibida**

El algoritmo recibe la siguiente lista ya ordenada:

```
A1, A2, C1, C2, M1, M2, G1, G2, I1, I2, E1, E2, K1, K2, L1, L2, F1, F2, J1, J2, H1, H2, N1, N2, D1, D2, B1, B2
```

### **División en Bloques por Número de Victorias**

El algoritmo divide la lista en bloques según el número de victorias:

- **Bloque 1 (1 victoria):**
  ```
  A1, A2, C1, C2, M1, M2, G1, G2, I1, I2, E1, E2, K1, K2
  ```
  
- **Bloque 0 (0 victorias):**
  ```
  L1, L2, F1, F2, J1, J2, H1, H2, N1, N2, D1, D2, B1, B2
  ```

### **Consideraciones sobre Empates**

Es importante destacar que:

- **A1** y **A2** tienen los mismos valores ya que fueron pareja y han jugado una sola ronda.
- Esta misma lógica aplica para los demás pares (C1-C2, M1-M2, etc.).
- **A1** y **A2** están empatados en la primera posición del primer bloque.
- **C1** y **C2** ocupan la segunda posición, y así sucesivamente.

> **Nota:** Este tipo de empates son comunes después de la ronda 1 y antes de la ronda 2. No es común que dos atletas mantengan los mismos valores después de la ronda 2, gracias a los índices de desempate en la tabla de clasificación.

### **Verificación de Mesas Completas**

Antes de proceder con el bloque de atletas con más victorias (en este caso, 1 victoria), el algoritmo debe verificar si el bloque generará mesas incompletas. Por ejemplo:

- Si dividimos el bloque 14 atletas entre 4 (para formar mesas de 4 jugadores), obtenemos 3.5 mesas, lo cual no es un número entero.
- Necesitamos un número par de atletas para cumplir con el requisito de 4 jugadores por mesa.

### **Ajuste de Bloques**

Al detectar esta incongruencia, el algoritmo debe:

1. **Identificar** a los 2 atletas mejor posicionados del bloque contiguo inmediato (en este caso, el bloque con 0 victorias): **L1** y **L2**.

2. **Verificar** si estos atletas fueron compañeros de los últimos atletas del bloque 1 en la ronda anterior:
   - ¿**L1** fue compañero de **K1** o **K2**? → No
   - ¿**L2** fue compañero de **K1** o **K2**? → No

3. **Integrar** a **L1** y **L2** al bloque superior, ubicándolos en las últimas posiciones.

> **Caso Especial:** Si hubiera coincidencias, el algoritmo continuaría buscando candidatos válidos en el bloque 0 hasta encontrar una combinación que no viole la Regla 1.

### **Bloque Superior Ajustado**

Finalmente, el bloque superior quedaría así:

```
A1, A2, C1, C2, M1, M2, G1, G2, I1, I2, E1, E2, K1, K2, L1, L2
```

### **Consideraciones Finales**

- Este ajuste garantiza que el número total de atletas en el bloque sea par, permitiendo la formación de mesas completas de 4 jugadores.
- Estos escenarios son excepcionales y ocurren principalmente cuando la división del número de atletas entre 4 no da como resultado un número entero.
- Cuando la división es par, las mesas se completan normalmente sin necesidad de ajustes.

### **Proceso de Emparejamiento**

En la ronda 2, el algoritmo comienza a emparejar compañeros siguiendo estos pasos:

1. Si el contiguo inmediato infringe la **Regla 1**, se avanza al siguiente puesto hasta encontrar un candidato válido.
2. Una vez formadas todas las parejas, se crean los **bloques por puntos**.
3. Cada bloque se **pliega** para crear los enfrentamientos:
   - La mitad superior del bloque se enfrenta ordenadamente contra la mitad inferior.
   - La pareja en primer lugar de la mitad superior enfrenta a la pareja en primer lugar de la mitad inferior.
   - El segundo contra el segundo, y así sucesivamente, manteniendo el orden jerárquico.

### **Ejemplo de Emparejamiento**

Aplicando la **Regla 1** y manteniendo las posiciones jerárquicas, se forman las siguientes parejas:

```
A1 - C1, A2 - C2, M1 - G1, M2 - G2, I1 - E1, I2 - E2, K1 - L1, K2 - L2
```

---

## **Conformación de Sistema de Plegado**

### **División del Bloque 1**

El bloque 1 (denominado temporalmente como "1" por contener a los jugadores con 1 victoria) se divide en dos subgrupos: la mitad superior y la mitad inferior.

#### **Mitad Superior**
```
A1 - C1, A2 - C2, M1 - G1, M2 - G2
```

#### **Mitad Inferior**
```
I1 - E1, I2 - E2, K1 - L1, K2 - L2
```

### **Creación de Mesas**

El algoritmo cierra los pliegues paralelamente, creando así las mesas con sus respectivos enfrentamientos:

| Mesa | Partido |
|------|---------|
| 1    | **A1 – C1** vs **I1 – E1** |
| 2    | **A2 – C2** vs **I2 – E2** |
| 3    | **M1 – G1** vs **K1 – L1** |
| 4    | **M2 – G2** vs **K2 – L2** |

### **Explicación del Proceso**

1. **Ordenamiento Jerárquico**: Las parejas se ordenan según su posición en la clasificación general.
2. **División en Mitades**: Se divide el bloque en dos mitades iguales.
3. **Emparejamiento Cruzado**: La primera pareja de la mitad superior se enfrenta a la primera de la mitad inferior, y así sucesivamente.
4. **Asignación de Mesas**: Se asignan números de mesa de forma ascendente según el rendimiento de las parejas.

---

## **Emparejamiento del Bloque Inferior en Ronda 2**

### **Proceso de Emparejamiento**

Aplicamos la misma dinámica y los mismos criterios para encontrar compañeros, rivales y conformar las mesas faltantes.

#### **Lista de Atletas del Bloque 0**
```
F1, F2, J1, J2, H1, H2, N1, N2, D1, D2, B1, B2
```

#### **Formación de Parejas**
Siguiendo la **Regla 1**, se forman las siguientes parejas:
```
F1 - J1, F2 - J2, H1 - N1, H2 - N2, D1 - B1, D2 - B2
```

#### **División en Mitades**

- **Mitad Superior:**
  ```
  F1 - J1, F2 - J2, H1 - N1
  ```

- **Mitad Inferior:**
  ```
  H2 - N2, D1 - B1, D2 - B2
  ```

### **Creación de Mesas**

El algoritmo cierra los pliegues paralelamente, creando así las siguientes mesas con sus respectivos enfrentamientos:

| Mesa | Partido |
|------|---------|
| 1    | **F1 – J1** vs **H2 – N2** |
| 2    | **F2 – J2** vs **D1 – B1** |
| 3    | **H1 – N1** vs **D2 – B2** |

### **Consideraciones**

- Se mantiene el orden jerárquico en la asignación de mesas.
- Se respeta la **Regla 1** para evitar emparejamientos recientes.
- La numeración de mesas continúa desde la última asignada en el bloque anterior.

---

## **Enumeración de Mesas**

### **Asignación de Números de Mesa**

Una vez conformados todos los enfrentamientos, procedemos a enumerar las mesas para la Ronda 2. La numeración sigue un orden ascendente según el rendimiento de las parejas, donde la **Mesa 1** siempre corresponde al enfrentamiento con los atletas de mejor desempeño.

### **Normativa de Asignación**

- **Criterio Principal:** Los atletas con mejor desempeño ocupan las mesas con números más bajos.
- **Jerarquía:** La numeración refleja el orden jerárquico de los equipos según su rendimiento acumulado.
- **Universalidad:** Este criterio es estándar en torneos profesionales de dominó a nivel mundial.

### **Mesas de la Ronda 2**

| Mesa | Partido |
|------|---------|
| **1** | **A1 – C1** vs **I1 – E1** |
| **2** | **A2 – C2** vs **I2 – E2** |
| **3** | **M1 – G1** vs **K1 – L1** |
| **4** | **M2 – G2** vs **K2 – L2** |
| **5** | **F1 – J1** vs **H2 – N2** |
| **6** | **F2 – J2** vs **D1 – B1** |
| **7** | **H1 – N1** vs **D2 – B2** |

### **Consideraciones**

- La numeración es crucial para el desarrollo del torneo, ya que las mesas con números más bajos suelen recibir mayor atención.
- Los árbitros más experimentados suelen ser asignados a las mesas con números más bajos.
- Este orden se mantendrá en rondas posteriores, ajustándose según el rendimiento de los participantes.

---

## **Reglas Obligatorias del Torneo**

### **Regla 1: Restricción de Reemparejamiento de Compañeros**

#### **Descripción**
Un atleta no puede volver a formar pareja con un excompañero hasta que hayan transcurrido al menos **2 rondas completas** desde su último emparejamiento.

#### **Aplicación Práctica**
- **Período de Enfriamiento:** 2 rondas completas
- **Flexibilidad:** En torneos pequeños, se puede ser flexible considerando el número limitado de participantes.
- **Fórmula de Cálculo:**
  ```
  Ronda de Nuevo Emparejamiento = Ronda de Último Emparejamiento + 2
  ```

#### **Ejemplo**
En un torneo de 9 rondas:
- Si **A1** y **A2** fueron pareja en la **Ronda 1**
- **No podrán** volver a ser pareja en las rondas 2 y 3
- **Podrán** volver a ser pareja a partir de la **Ronda 4**

### **Regla 2: Restricción de Reencuentros entre Rivales**

#### **Descripción**
Un atleta no puede enfrentarse nuevamente a los mismos rivales hasta que hayan transcurrido al menos **1 ronda completa** desde su último encuentro.

#### **Aplicación Práctica**
- **Período de Enfriamiento:** 1 ronda completa
- **Recomendación:** Aunque técnicamente podrían volver a enfrentarse después de 2 rondas, lo ideal es evitarlo cuando sea posible.
- **Fórmula de Cálculo:**
  ```
  Ronda de Nuevo Enfrentamiento = Ronda del Último Enfrentamiento + 1
  ```

#### **Ejemplo**
En un torneo de 9 rondas:
- Si **A1** se enfrentó a **B1** y **B2** en la **Ronda 1**
- **No podrá** volver a enfrentarse a ellos en la **Ronda 2**
- **Podría** volver a enfrentárseles a partir de la **Ronda 3** (aunque se recomienda evitarlo si es posible)

### **Consideraciones Generales**

| Aspecto | Regla 1 (Compañeros) | Regla 2 (Rivales) |
|---------|----------------------|-------------------|
| **Período de Enfriamiento** | 2 rondas | 1 rondas |
| **Flexibilidad** | Alta (especialmente en torneos pequeños) | Media (preferible evitar) |
| **Afecta a** | Parejas del mismo equipo | Equipos rivales |
| **Objetivo** | Promover diversidad de alianzas | Garantizar variedad de oponentes |

### **Excepciones y Flexibilidad**

1. **Torneos Pequeños:**
   - En torneos con pocos participantes, las reglas pueden relajarse ligeramente para permitir más combinaciones posibles.
   
2. **Fases Finales:**
   - En rondas avanzadas, especialmente en la final, estas reglas pueden flexibilizarse para priorizar los enfrentamientos más justos y emocionantes.

3. **Desempates:**
   - En caso de empates técnicos, el comité organizador puede hacer excepciones justificadas, siempre informando a todos los participantes.

---

# **Análisis de Ronda 3**

## **Resumen de Resultados de la Ronda 2**

Los ganadores de la segunda ronda fueron los siguientes equipos:

| Equipo | 
|--------|
| A1 – C1 |
| A2 – C2 |
| M1 – G1 |
| M2 – G2 |
| F1 – J1 |
| F2 – J2 |
| H1 – N1 |

## **Lista Ordenada para la Ronda 3**

El algoritmo recibe la siguiente lista completa ordenada según el rendimiento:

```
A2, C2, A1, C1, M2, G2, G1, M1, F1, J1, F2, J2, K1, K2, H1, N1, E2, I2, E1, I1, L1, L2, D2, B2, D1, B1, H2, N2
```

## **Consideraciones Generales**

A partir de la **Ronda 3**, el algoritmo sigue un patrón consistente que se mantendrá hasta el final del torneo. Las rondas 1 y 2 tuvieron reglas especiales que ya no aplican.

## **División en Bloques por Número de Victorias**

El algoritmo analiza y divide a los atletas en bloques según su número de victorias:

| Bloque | Victorias | Cantidad de Atletas | Atletas |
|--------|-----------|---------------------|---------|
| **Bloque 2** | 2 victorias | 8 | A2, C2, A1, C1, M2, G2, G1, M1 |
| **Bloque 1** | 1 victoria | 12 | F1, J1, F2, J2, K1, K2, H1, N1, E2, I2, E1, I1 |
| **Bloque 0** | 0 victorias | 8 | L1, L2, D2, B2, D1, B1, H2, N2 |

---

# **Procesamiento del Bloque 2 (2 Victorias)**

## **1. Verificación de División**

- **Cálculo:** 8 atletas ÷ 4 = 2 mesas exactas
- **Resultado:** La división es exacta (número par), por lo que no es necesario traer atletas del bloque inferior.

## **2. Proceso de Emparejamiento**

### **Aplicación de la Regla 1**
El algoritmo busca formar parejas verificando que no hayan sido compañeros en las 2 rondas anteriores.

#### **Primer Intento**

1. **Emparejamiento de A2**
   - Posibles compañeros: C2, A1, C1, M2, G2, G1, M1
   - Verificación:
     - A2 con C2: ❌ Fueron compañeros en Ronda 2
     - A2 con A1: ❌ Fueron compañeros en Ronda 1
     - A2 con C1: ✓ No han sido compañeros
   - **Pareja formada:** A2 - C1

2. **Emparejamiento de C2**
   - Posibles compañeros restantes: M2, G2, G1, M1
   - Verificación:
     - C2 con M2: ✓ No han sido compañeros
   - **Pareja formada:** C2 - M2

3. **Emparejamiento de A1**
   - Posibles compañeros restantes: G2, G1, M1
   - Verificación:
     - A1 con G2: ✓ No han sido compañeros
   - **Pareja formada:** A1 - G2

4. **Emparejamiento de G1 y M1**
   - Verificación:
     - G1 con M1: ❌ Fueron compañeros en Ronda 2
   - **Problema detectado:** No se puede formar esta pareja

### **Solución: Reajuste de Emparejamiento**

El algoritmo debe retroceder y probar una combinación diferente.

1. **Reemparejamiento de A1**
   - Posibles compañeros: G1, M1
   - Verificación:
     - A1 con G1: ✓ No han sido compañeros
   - **Nueva pareja formada:** A1 - G1

2. **Emparejamiento de G2 y M1**
   - Verificación:
     - G2 con M1: ✓ No han sido compañeros
   - **Pareja formada:** G2 - M1

### **Parejas Finales del Bloque 2**

1. **A2 - C1**
2. **C2 - M2**
3. **A1 - G1**
4. **G2 - M1**

## **3. Sistema de Plegado**

### **División en Mitades**

| Mitad | Parejas |
|-------|---------|
| **Superior** | A2 - C1, C2 - M2 |
| **Inferior** | A1 - G1, G2 - M1 |

## **4. Validación de Enfrentamientos - Regla 2**

Antes de confirmar los enfrentamientos, el algoritmo debe verificar que ningún atleta se enfrente a rivales que ya enfrentó en la ronda anterior.

### **Enfrentamientos Propuestos**

1. **A2 - C1** vs **A1 - G1**
2. **C2 - M2** vs **G2 - M1**

### **Verificación de la Regla 2**

- **A2 - C1 vs A1 - G1**
  - A2 enfrentó a A1 en Ronda 1 ❌
  - **Solución:** Se requiere ajuste

- **C2 - M2 vs G2 - M1**
  - No hay enfrentamientos previos entre estos equipos ✓

### **Ajuste Necesario**

El algoritmo debe intercambiar los enfrentamientos para evitar la violación de la Regla 2:

1. **A2 - C1** vs **G2 - M1**
2. **C2 - M2** vs **A1 - G1**

### **Enfrentamientos Finales del Bloque 2**

| Mesa | Partido |
|------|---------|
| 1 | **A2 - C1** vs **G2 - M1** |
| 2 | **C2 - M2** vs **A1 - G1** |

### Validación Sistemática:

#### Enfrentamiento 1: **A2 - C1 vs A1 - G1**

**Verificaciones:**
- ¿A2 enfrentó a A1 en ronda 2? → No (A2 enfrentó a I2-E2) ✓
- ¿A2 enfrentó a G1 en ronda 2? → No ✓
- ¿C1 enfrentó a A1 en ronda 2? → No (si fueron compañeros pero no rivales) ✓
- ¿C1 enfrentó a G1 en ronda 2? → No (C1 enfrentó a I1-E1) ✓

**NO HAY CONFLICTO** El enfrentamiento A2-C1 vs A1-G2 es válido según la Regla 2.

#### Enfrentamiento 2: **C2 - M2 vs G2 - M1**

**Verificaciones:**
- ¿C2 enfrentó a G2 en ronda 2? → No (C2 enfrentó a I2-E2) ✓
- ¿C2 enfrentó a M1 en ronda 2? → No ✓
- ¿M2 enfrentó a G2 en ronda 2? → No (M2 enfrentó a K2-L2) ✓
- ¿M2 enfrentó a M1 en ronda 2? → No ✓

**NO HAY CONFLICTO** El enfrentamiento C2-M2 vs G2-M1 es válido según la Regla 2.

### Enfrentamientos Finales del Bloque 2
- **A2 - C1 vs G1 - M1**
- **C2 - M2 vs A1 - G2**

---

### Procesamiento Resumido del Bloque 1 (1 victoria)
Atletas: F1, J1, F2, J2, K1, K2, H1, N1, E2, I2, E1, I1  
Parejas (Regla 1):  
- **F1 – J2**  
- **J1 – F2**  
- **K1 – H1**  
- **K2 – N1**  
- **E2 – I1**  
- **I2 – E1**

Enfrentamientos (Sistema de plegado, Regla 2):  
- **F1 – J2 vs K2 – N1**  
- **J1 – F2 vs E2 – I1**  
- **K1 – H1 vs I2 – E1**

### Procesamiento Resumido del Bloque 0 (0 victorias)
Atletas: L1, L2, D2, B2, D1, B1, H2, N2  
Parejas (Regla 1):  
- **L1 – D2**  
- **L2 – B2**  
- **D1 – H2**  
- **B1 – N2**

Enfrentamientos (Sistema de plegado, Regla 2):  
- **L1 – D2 vs D1 – H2**  
- **L2 – B2 vs B1 – N2**

### Mesas Definitivas – Ronda 3
1. **A2 – C1 vs G1 – M1**  
2. **C2 – M2 vs A1 – G2**  
3. **F1 – J2 vs K2 – N1**  
4. **J1 – F2 vs E2 – I1**  
5. **K1 – H1 vs I2 – E1**  
6. **L1 – D2 vs D1 – H2**  
7. **L2 – B2 vs B1 – N2**

---

## Algoritmo Generalizado para Rondas 3 en Adelante

### Pasos del Algoritmo para rondas posteriores:

1. **Recepción de Lista Ordenada:** El algoritmo recibe la lista completa ordenada descendentemente por número de victorias y criterios de desempate secundarios.

2. **Separación en Bloques:** Divide automáticamente la lista en bloques según el número de victorias de cada atleta.

3. **Procesamiento por Bloque (de mayor a menor victorias):**

   a) **Verificación de División:** Si el número de atletas en el bloque dividido entre 4 no da un número par:
   - Trae los 2 mejores atletas del bloque inmediato inferior
   - Verifica que no violen la Regla 1 con los 2 últimos del bloque superior
   - Si hay violación, busca los siguientes candidatos válidos en bloque inferior contiguo

   b) **Emparejamiento:** Para cada atleta en orden jerárquico:
   - Busca el primer compañero disponible que no viole la Regla 1
   - Si no encuentra válido en posiciones inmediatas, avanza secuencialmente

   c) **Sistema de Plegado:**
   - Divide las parejas en mitad superior e inferior
   - Enfrenta paralelamente: 1° superior vs 1° inferior, 2° superior vs 2° inferior, etc.

4. **Verificación de Regla 2:** Antes de confirmar enfrentamientos, verificar que ninguna pareja haya enfrentado rivales de la ronda anterior o numero de rondas segun corresponda ventana por categoria.

5. **Enumeración de Mesas:** Numera ascendentemente comenzando por el atleta líder del torneo.

---

## Escenario Especial: Bloque Superior con Exactamente 2 Atletas

### REGLA DEL DUELO DE LÍDERES

Cuando el algoritmo detecta que el bloque superior contiene exactamente 2 atletas (los líderes absolutos del torneo), se activa un protocolo especial denominado "Duelo de Líderes". Este escenario típicamente ocurre en rondas intermedias o avanzadas (ronda 5, 6 o posteriores) cuando la competencia se ha polarizado y dos atletas se han destacado claramente del resto.

Protocolo de Ejecución:

1. Identificación del Escenario:

  - El algoritmo detecta que el bloque superior tiene exactamente 2 atletas
  - Estos atletas son los líderes absolutos del torneo con el mayor número de victorias

2. Prohibición de Emparejamiento:

  - Los 2 líderes NO pueden ser compañeros bajo ninguna circunstancia
  - Deben enfrentarse obligatoriamente como rivales en bandos opuestos
  - Esta regla prevalece sobre cualquier otra consideración del algoritmo


3. Búsqueda de Compañeros Válidos:

  - El algoritmo debe traer exactamente 2 atletas del bloque inmediato inferior
  - Estos atletas se convertirán en los compañeros de cada líder
  - Verificación Regla 1: Los candidatos no pueden haber sido compañeros de ninguno de los 2 líderes en las rondas anteriores según la ventana establecida
  - Compañero A y Compañero B NO deben haberse enfrentado en ronda anterior.
  - Si los primeros 2 candidatos violan la Regla 1, el algoritmo debe continuar buscando secuencialmente hasta encontrar 2 candidatos válidos, en caso de ser válidos verificar que no violen la Regla 2.
  

4. Configuración de Mesa:

  - Líder 1 + Compañero A vs Líder 2 + Compañero B
  - Esta mesa se convierte automáticamente en la Mesa 1 del torneo (mesa principal)
  - Representa el enfrentamiento más relevante de la ronda


5. Impacto en la Clasificación:

  - El resultado de esta mesa define quién mantiene o toma el liderazgo absoluto
  - Es un momento decisivo en la evolución del torneo


Ejemplo Práctico:
Situación: Ronda 6, Bloque Superior con 2 atletas (5 victorias c/u)

Líderes: A2, C1
Bloque Inferior: E1, I2, D1, D2, E2... (4 victorias c/u)

Proceso:

A2 y C1 NO pueden ser compañeros
Buscar compañeros: ¿A2 fue compañero de E1 en ventana? ¿C1 fue compañero de I2 en ventana? ¿E1 fue rival de I2 en ventana?
Si válidos → A2-E1 vs C1-I2 (Mesa 1)
Si inválidos → continuar con D1, D2, etc.

Resultado: El Duelo de Líderes garantiza que los mejores atletas se enfrenten directamente, manteniendo la competitividad y el espíritu del sistema suizo profesional.

---

## Escenario Especial: Bloque Inferior con 2 Atletas

### REGLA DEL FLOAT-UP DE COLISTAS

Este protocolo se activa cuando el **último bloque** contiene exactamente **dos atletas**.

1. **Identificación del escenario**  
   • Bloque inferior con 2 atletas.  
   • Verificar si ya se enfrentaron durante la ventana establecida.

2. **Opción rápida – Duelo directo**  
   Se usa solo si (a) los colistas **aún no se han enfrentado** y (b) ascenderlos rompería la paridad del bloque superior (|B<sub>sup</sub>| + 2 ≡ 2 mod 4).  
   • Los dos colistas permanecen juntos y juegan como rivales.  
   • Se seleccionan los dos atletas inmediatamente superiores (puestos *n-1* y *n*) como compañeros, siempre que respeten Regla 1 y Regla 2.  
   • Se forma una **única mesa de 4**; el resto del bloque superior se empareja con normalidad.

3. **Float-up estándar**  
   En cualquier otro caso, ambos colistas **ascienden (up-float)** al bloque superior y se insertan en las **dos últimas posiciones** para no alterar la jerarquía.  
   • El bloque resultante se empareja según el procedimiento habitual.  
   • Marcar a ambos como *float-up* para evitar ascensos consecutivos.

4. **Contingencia de mesas impares tras el ascenso**  
   Después del ascenso, si el tamaño del nuevo bloque queda |B| ≡ 2 mod 4, se aplica una **transferencia compensatoria**:  
   • Los **dos últimos** del bloque ahora excedente descienden (down-float) al siguiente bloque; si no existe, reciben **BYE** conforme a la sección de inasistencias.  
   • Así se restablece la divisibilidad por 4 sin romper la lógica suiza.

5. **Impacto en la clasificación**  
   • Los colistas siguen ocupando las posiciones más bajas y no adelantan a jugadores con mejor puntuación.  
   • El sistema evita byes prematuros y repeticiones innecesarias.

**Ejemplo 1 (ascenso directo):**  
Bloque X-1: 10 jug. (2-2)  
Bloque X :   2 jug. (1-3)  
⇒ 10 + 2 = 12 (divisible por 4) → float-up estándar.

**Ejemplo 2 (contingencia por imparidad):**  
Bloque X-1: 11 jug. (2-2)  
Bloque X :   2 jug. (1-3)  
⇒ 11 + 2 = 13 (≡ 1 mod 4).  
  a) Si colistas no se han enfrentado → Duelo directo + 2 compañeros del propio bloque superior.  
  b) Si ya se enfrentaron → Float-up y luego down-float de compensación.

---

## Escenario ELO para primera ronda:

**Caso Excepcional:** Cuando en primera ronda el administrador del torneo no decide comenzar la ronda en modo aleatorio sino que decide que los jugadores se ordenen por ELO, entonces el algoritmo recibe la lista completa de jugadores ordenada de manera descendente segun su ELO, posicionados desde el ELO mas alto hasta el ELO mas bajo. 

En una situacion asi el algoritmo debe ser capaz de identificar la modalidad en cuestion y buscar a la pareja mas contigua (de nivel similar) y realizar el pareo antes de iniciar ronda 1.

Supongamos recibimos el siguiente listado:
**A1, A2, B1, B2, C1, C2, D1, D2, E1, E2, F1, F2, G1, G2, H1, H2, I1, I2, J1, J2, K1, K2, L1, L2, M1, M2, N1, N2**

Realizamos el pareo:

**A1 - A2, B1 - B2, C1 - C2, D1 - D2, E1 - E2, F1 - F2, G1 - G2, H1 - H2, I1 - I2, J1 - J2, K1 - K2, L1 - L2, M1 - M2, N1 - N2**

Dividimos la lista completa de jugadores en una primera mitad (las mejores parejas) y enviamos esa primera mitad para conformar la Piramide Principal (PP) y estos mismos a su vez se subdividen entre lados, es decir, en 2 grupos , la mitad superior sera el lado A de la piramide y la mitad inferior sera el lado B. 

Del mismo modo haremos con la ultima mitad del listado total de parejas, los enviamos para conformar la Piramide Secundaria (PS), el algoritmo tambien va a subdividir entre lados A y B con la mitad superior y mitad inferior respectivamente.  

Conformamos las piramides:

Lado A PP: A1, A2, B1, B2, C1, C2, D1 (incongruencia detectada) 

El algoritmo debe tener la capacidad de detectar si un lado es impar, si o si siempre cada lado debe tener un numero de atletas par. En el lado A comprobamos que no es par ya que 7 atletas es una cantidad impar, entonces el algoritmo debe subir desde PS a los 2 atletas lideres o mejor posicionados quienes serian H1 y H2, los ascendemos hasta PP y de este modo ya tendriamos un lado A de PP con parejas par. La siguiente estructura es la que debe generar el algoritmo, tal cual sin alucinaciones:

Lado A PP: A1 - A2, B1 - B2, C1 - C2, D1 - D2 
Lado B PP: E1 - E2, F1 - F2, G1 - G2, H1 - H2

Lado A PS: I1 - I2, J1 - J2, K1 - K2 
Lado B PS: L1 - L2, M1 - M2, N1 - N2

Cerramos los lados de cada pirámide y creamos las mesas con sus respectivos enfrentamientos:

- Mesa 1: **A1 – A2 vs E1 – E2**
- Mesa 2: **B1 – B2 vs F1 – F2**  
- Mesa 3: **C1 – C2 vs G1 – G2**
- Mesa 4: **D1 – D2 vs H1 – H2**
- Mesa 5: **I1 – I2 vs L1 – L2**
- Mesa 6: **J1 – J2 vs M1 – M2**
- Mesa 7: **K1 – K2 vs N1 – N2**

y asi bajo este esquema ya tenemos las mesas ordenadas para dar inicio al torneo, despues para las rondas posteriores se aplica la misma dinamica por bloques.

---

## Manejo de Inasistencias y "Byes"

El algoritmo obligatoriamente debe contemplar el siguiente escenario:
  - El algoritmo debe detectar cuando un jugador esta ausente o abandona, quizas por medio de alguna bandera que recibe por parte del sistema.
  - **BYE ≠ abandono.** Un atleta con BYE está presente y sigue compitiendo, pero en esa ronda no juega porque falta gente para completar su mesa de 4. Quien abandona obtiene 0 puntos; su ausencia provoca el BYE de otros.
  - Si el total de inscritos deja un resto 1 (mod 4) → 3 atletas reciben BYE.
  - Si resta 2 (mod 4) → 2 atletas reciben BYE.
  - Si resta 3 (mod 4) → 1 atleta recibe BYE.
  - Los BYE obtienen automáticamente el puntaje de descanso (≈ 50 % del puntaje objetivo) y se marcan con una bandera para que el sistema de clasificación los procese el puntaje debido.

---

## Propuesta para Reglas 1 y 2 según Categoría

- **CATEGORÍA COMPACTA (≤ 36 participantes):**
  - Compañeros: ventana de +2 rondas
  - Rivales: ventana de +1 ronda
  - Riesgo distorsión: ¿quizás ALTO si aplicamos más restricciones?

- **CATEGORÍA ESTÁNDAR (40-76 participantes):**
  - Compañeros: ventana de +3 rondas
  - Rivales: ventana de +2 rondas
  - Riesgo distorsión: ¿quizás MEDIO-BAJO con parejas en bucle infinito por espera de pareo?

- **CATEGORÍA INTERNACIONAL (80+ participantes):**
  - Compañeros: ventana de +4 rondas
  - Rivales: ventana de +3 rondas
  - Riesgo distorsión: ¿quizás MÍNIMO sin alteraciones y todos los jugadores dentro de su respectiva pirámide según corresponda?

---

## Posibles Problemas: Efecto Cascada

- Si una pareja líder en el bloque superior no encuentra rivales, debe bajar hasta el siguiente bloque y este suceso distorsiona completamente el sistema suizo.
- Crea "emparejamientos forzados" no competitivos.

---

## Flexibilidad y Mandamientos Excepcionales

En caso extremo que el algoritmo no logre encontrar pareja válida, lo idóneo es que no se pierda la base y estructura del “sistema suizo”. Por ende, se podría estudiar la posibilidad de aplicar mandamientos excepcionales tales como:

- **Primero:** Realizar una transfusión de atletas entre bloques, los últimos lugares del bloque superior intercambiados con los primeros lugares del bloque inferior. Se debe ser estrictamente selectivo y excluyente para no generar distorsiones.
- **Segundo:** Con respecto a las reglas 1 y 2, se podría aplicar una relajación progresiva en cuanto a las ventanas, como por ejemplo disminuir una ronda; es decir, en el escenario de “categoría estándar” se podría aplicar -1 ronda y las ventanas pasarían de ser +3/+2 a +2/+1 solo para los jugadores afectados, no para todos.

Pero dejando siempre claro que lo óptimo sería que el algoritmo funcionara sin estos mandamientos, que sean solo una utilidad de último recurso.

---

## Conclusiones y peticion a la IA

Dicho esto, como la IA con mayor potencial del mercado que eres tú, por supuesto debes profundizar con rigurosidad y desarrollar el algoritmo por mí, justo esa es la idea, por eso he tratado de darte el contexto necesario. El algoritmo que tú me generes debe tener la lógica y coherencia desde la ronda 1 hasta la ronda 9. No me decepciones ya que el algoritmo será auditado y no quiero hacer el ridículo por tu culpa.

Con el algoritmo ya finalizado debes hacer doble simulación interna para tu propio consumo, no a nivel visual, ni generar archivos extras, la simulación debe ser mentalmente para ti mismo como IA para que luego me indiques si se presentaron vicisitudes durante la simulación y sobre cuáles serían los posibles retos inesperados a presentarse, ya que necesitamos prever problemas para los debidos mantenimientos preventivos y no correctivos. Este algoritmo debe ser versátil y adaptable para poder superar cualquier prueba con rotundo éxito. Yo no necesito ver las simulaciones, la doble simulacion sera una sin jugadores "Bye" y otra simulacion si con 3 jugadores con estatus BYE y 1 abandono.

Debes generar 2 archivos, uno con el algoritmo en pseudocódigo tecnico y otro archivo en formato markdown explicando dicho algoritmo en un lenguaje más natural para humanos. La creación de 2 archivos es opcional, al menos que quieras crear todo dentro de un solo archivo. ¡Sorpréndeme!

Importante: NO generar ningun archivo hasta que la doble simulacion haya culminado con exito, repetir simulaciones necesarias hasta lograr el exito esperado y comprobar que nuestro algoritmo es optimo, luego de ello si generar el o los archivo(s).

---

**Nota:** Si necesitas más contexto me puedes preguntar y yo te voy a facilitar otro prompt contextual adicional para abordar la eventualidad.