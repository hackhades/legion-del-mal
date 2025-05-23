# 📊 Métricas y Criterios para Torneos de Dominó (FID Estándar)

## 🔢 Métricas Básicas
| **Nombre**            | **Fórmula**                          | **Descripción**                                  |
|-----------------------|--------------------------------------|------------------------------------------------|
| **Partidas Jugadas (PJ)** | `PJ = ∑ Partidas`                | Total de partidas disputadas.                  |
| **Partidas Ganadas (PG)** | `PG = ∑ Victorias`               | Partidas ganadas.                              |
| **Puntos Favor (PF)**    | `PF = ∑ Puntos anotados`         | Puntos totales a favor.                        |
| **Puntos Contra (PC)**   | `PC = ∑ Puntos recibidos`        | Puntos totales en contra.                      |
| **Puntaje**   | `ptj_base = 100 o 200 ó mas`        | Puntaje objetivo por partida durante un torneo.            |

## 📈 Métricas de Rendimiento
| **Nombre**            | **Fórmula**                          | **Descripción**                                  |
|-----------------------|--------------------------------------|------------------------------------------------|
| **Diferencia (Diff)** | `Diff = PF - PC`                   | Balance neto de puntos.                        |
| **Average (Avg)**     | `Avg = PF / PJ`                    | Puntos promedio por partida.                   |
| **Win Rate (%)**      | `Win Rate = (PG / PJ) × 100`       | % de partidas ganadas.                         |
| **IAR**   | `(IAR_calculado - 1) * 100`                     | Mostrar como Diferencia Porcentual    |
| **IDN**   | `[(PF-PC)/(PF+PC)] × 100`                       | Proporción de dominio de puntos            |
| **Efectividad (Eff)** | `Eff = 0.45×Win_Rate + 0.35×DRP_norm[-100, 100] + 0.20×PBT_norm` | Combina victorias y dominio de puntos.        |
| **DRP**         | `[(PF-PC)/(umbral×PJ)] × 100`                                 | Rendimiento base normalizado               |
| **Umbral** | `umbral = ptj_base * (1 + (1/3 * (PF_total / (ptj_base × PJ))))` | denominador de DRP.        |
| **μTBz**      | `(Suma_PG_total_todos) / (N-1)`     | Fuerza de Calendario / N (número de oponentes únicos) - ultimo     |

# Paso: Promedio Buchholz truncado
-  Excluir el PG más bajo de la lista de PG de los oponentes.

-	Calcular Suma_PGT (suma de los PG de los N-1 oponentes restantes).

-	Calcular PBT_final = (Suma_PGT) / (N-1).


# Paso normalizar DRP \[-100,100\]

   (DRP Equitativo Normalizado y Escalado):

   -   Crudo: Se calcula el DRP_equitativo_crudo para cada jugador (considera PF, PC, PJ y un umbral dinámico ajustado por la ofensiva del jugador). Puede ser negativo.

   -   Escalado: Los DRP_equitativo_crudo de todos los jugadores se normalizan Min-Max a una escala de [-100, 100].

   -   Fórmula de escalado: Norm = (((Crudo - Min_Crudo) / Rango_Crudo) * 200) - 100



# 📌 Glosario

**DRP** = *Diferencia Relativa de Puntos* Métrica que mide eficiencia en el manejo de puntos

**μ** = *Promedio* , suma lineal / N total

**símbolo Σ** = *Sigma* , suma lineal   
 

## ⚖️ Criterios de Desempate (Orden de Prioridad)
1. **Partidas Ganadas (PG)**.  
2. **Efectividad (Eff)**.  
4. **μ Buchholz**.  
4. **IDN**.  
3. **Diferencia de Puntos (Diff)** (`PF - PC`).  
4. **Puntos a Favor (PF)**.  
4. **IAR**.  
4. **AVG**.  
4. **PF**.  
5. **Enfrentamiento Directo por ELO** (mayor ELO inicial, previo al torneo).  

# 📋 Criterios de Emparejamiento para Torneos de Dominó (Sistema Suizo con Parejas Rotativas)

## 🔄 Orden de Prioridades
1. **Puntos acumulados individuales**  
   - Agrupa jugadores con similar rendimiento (altos con altos, bajos con bajos)

2. **Diferencia de puntos (Diff)**  
   - Dentro del mismo grupo de puntos, empareja quienes tengan Diff similar (±20%)

3. **Rotación obligatoria de parejas**  
   - Nunca repetir la misma combinación de jugadores como pareja
   - Evitar que ex-compañeros se enfrenten en rondas consecutivas

4. **Efectividad (Eff) [Opcional]**  
   - Usar solo para desempates muy ajustados

## ⚠️ Restricciones Absolutas
- **Prohibido** repetir enfrentamientos entre las mismas 4 personas (ej: A+B vs C+D solo una vez)
