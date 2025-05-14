# Fórmula Final y Optimizada: Promedio de Buchholz Truncado Final (PBT_final)

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

   c.  **Cálculo del PBT_final:**
       **PBT_final = (Suma_PGT × 10) / (N-1)**

---

## Resumen del Flujo Lógico para el Cálculo:

1.  Determinar **N** (número de oponentes únicos).
2.  Si **N = 0**, entonces **PBT_final = 0**.
3.  Si **N = 1**, entonces **PBT_final = (PG del único oponente) × 10**.
4.  Si **N >= 2**, entonces:
    *   Excluir el PG más bajo de la lista de PG de los oponentes.
    *   Calcular **Suma_PGT** (suma de los PG de los N-1 oponentes restantes).
    *   Calcular **PBT_final = (Suma_PGT × 10) / (N-1)**.