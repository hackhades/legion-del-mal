# Fórmula Final y Optimizada: Promedio de Buchholz Truncado Final (PBT_final)

    # 🏅 Método PBT Recomendado (TBz)

El sistema Buchholz es un método clásico de desempate en torneos suizos o ligas. Evalúa la "fuerza" de los oponentes que enfrentó cada jugador, sumando los puntos de torneo de todos sus rivales. El **Truncado Buchholz (TBz)** mejora este sistema al excluir el puntaje más bajo de los oponentes (evitando que un rival muy débil distorsione el cálculo). En algunos sistemas se excluye también el mayor, pero aquí solo se elimina el menor para simplificar.

Para cada jugador participante en el torneo, el PBT_final se calcula de la siguiente manera:

---

## Paso 1: Recopilar Datos de Oponentes

1.  Sea **N** el número total de oponentes *únicos* diferentes que el jugador ha enfrentado a lo largo del torneo.
2.  Obtener la lista de las **Partidas Ganadas (PG)** acumuladas en el torneo por cada uno de estos N oponentes únicos.
    *   `Lista_PG_Oponentes = [PG_op1, PG_op2, ..., PG_opN]`

---

## Paso 2: Calcular PBT_final según el valor de N

Se aplicará uno de los siguientes tres casos, dependiendo del valor de N:

### CASO A: Si N = 0
   *(El jugador no enfrentó oponentes únicos, por ejemplo, solo tuvo byes o no jugó ninguna partida).*

   **PBT_final = 0**

### CASO B: Si N = 1
   *(El jugador enfrentó a un solo oponente único).*

   **PBT_final = PG_op1 × 10**

   Donde:
   *   `PG_op1` = Total de Partidas Ganadas del único oponente enfrentado.

### CASO C: Si N >= 2
   *(El jugador enfrentó a dos o más oponentes únicos).*

   a.  **Truncamiento:**
       De la `Lista_PG_Oponentes`, se identifica y elimina el valor de Partidas Ganadas (PG) más bajo. Si varios oponentes comparten el mismo PG más bajo, solo se elimina la cuenta de uno de ellos.

   b.  **Suma de PG Truncados (Suma_PGT):**
       Se suman los PG de los `N-1` oponentes restantes en la lista truncada.

---

## Resumen del Flujo Lógico para el Cálculo:

1.  Determinar **N** (número de oponentes únicos).
2.  Si **N = 0**, entonces **PBT_final = 0**.
3.  Si **N = 1**, entonces **PBT_final = (PG del único oponente) × 10**.
4.  Si **N >= 2**, entonces:
    *   ver siguiente sección


---

## 📊 ¿Qué se considera "Puntos de Torneo"?

En este sistema, el punto de torneo más directo es **PG** (*Partidas Ganadas*), ya que no existen empates.

---

## 🧮 Cálculo paso a paso

### 1. **TBz crudo**

Para cada jugador:
- Suma los PG de todos sus oponentes.
- Resta el valor mínimo de esa lista (el oponente con menos PG).

\[
TBz_crudo = ∑ PG_oponentes - min(PG_oponentes)
\]

> **Nota:**  
> En algunos métodos, el TBz se escala para promedios (multiplica por 10 y divide por N-1, donde N es el número de oponentes), pero el método estándar recomendado es simplemente la suma truncada.

---

## 📝 Ejemplo usando listas de la Simulación 1

- **Jugador A:** Oponentes PG: `[5, 5, 4, 4, 3, 3, 2]`  
  Suma = 26, Mínimo = 2  
  **TBz_A_crudo = 26 - 2 = 24**

- **Jugador B:** Oponentes PG: `[5, 4, 4, 3, 3, 2, 1]`  
  Suma = 22, Mínimo = 1  
  **TBz_B_crudo = 22 - 1 = 21**

- **Jugador C:** Oponentes PG: `[4, 3, 2, 2, 1, 1, 0]`  
  Suma = 13, Mínimo = 0  
  **TBz_C_crudo = 13 - 0 = 13**

- **Jugador D:** Oponentes PG: `[2, 1, 1, 0, 0, 0, 0]`  
  Suma = 4, Mínimo = 0  
  **TBz_D_crudo = 4 - 0 = 4**

---

## ⚖️ Normalización Min-Max de TBz_crudo

Para comparar y visualizar el desempeño relativo, se recomienda **normalizar** los valores TBz_crudo al rango [0, 100] usando la siguiente fórmula:

\[
TBz_{norm} = \frac{TBz_{crudo} - TBz_{min}}{TBz_{max} - TBz_{min}} \times 100
\]

Donde:
- \(TBz_min\): El menor TBz_crudo del grupo.
- \(TBz_max\): El mayor TBz_crudo del grupo.

### **Ejemplo con los datos anteriores:**

- **Lista_TBz_crudos_Sim2:** `[24, 21, 13, 4]`
- **TBz_min:** `4`
- **TBz_max:** `24`
- **Rango_TBz:** `24 - 4 = 20`

\[
A_PBT_norm = (24 - 4) / 20 * 100 = 100%
B_PBT_norm = (21 - 4) / 20 * 100 = 85%
C_PBT_norm = (13 - 4) / 20 * 100 = 45%
D_PBT_norm = (4 - 4) / 20 * 100 = 0%
\]

---

## 📌 Resumen del procedimiento

1. Suma los PG de todos los oponentes de un jugador.
2. Excluye el menor valor de esa lista.
3. Obtén el **TBz crudo**.
4. Normaliza todos los TBz crudos usando la fórmula min-max para obtener un porcentaje comparativo.

**Ventajas:**  
- Refleja la "dureza" real de los rivales enfrentados.
- Evita distorsiones por oponentes débiles.
- Permite comparar el desempeño relativo de jugadores aunque hayan enfrentado rivales distintos.

---
