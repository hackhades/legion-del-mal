# 📊 Métricas y Criterios para Torneos de Dominó (FID Estándar)

## 🔢 Métricas Básicas
| **Nombre**            | **Fórmula**                          | **Descripción**                                  |
|-----------------------|--------------------------------------|------------------------------------------------|
| **Partidas Jugadas (PJ)** | `PJ = ∑ Partidas`                | Total de partidas disputadas.                  |
| **Partidas Ganadas (PG)** | `PG = ∑ Victorias`               | Partidas ganadas.                              |
| **Puntos Favor (PF)**    | `PF = ∑ Puntos anotados`         | Puntos totales a favor.                        |
| **Puntos Contra (PC)**   | `PC = ∑ Puntos recibidos`        | Puntos totales en contra.                      |

## 📈 Métricas de Rendimiento
| **Nombre**            | **Fórmula**                          | **Descripción**                                  |
|-----------------------|--------------------------------------|------------------------------------------------|
| **Diferencia (Diff)** | `Diff = PF - PC`                   | Balance neto de puntos.                        |
| **Average (Avg)**     | `Avg = PF / PJ`                    | Puntos promedio por partida.                   |
| **Win Rate (%)**      | `Win Rate = (PG / PJ) × 100`       | % de partidas ganadas.                         |
| **Efectividad (Eff)** | `Eff = 0.7×Win_Rate + 0.3×DRP_Híbrida` | Combina victorias y dominio de puntos.        |
| **DRP_base**         | `[(PF-PC)/(Puntaje_Partida×PJ)] × 100`                                 | Rendimiento base normalizado               |
| **DRP_excedentes**   | `[(PF-PC)/(PF+PC)] × 100`                                              | Proporción de dominio de puntos            |
| **multiplicacion**   | `(DRP_base × DRP_excedentes)`                            | Para que siempre de positivo    |
| **DRP Híbrida**      | `√multiplicacion`                                         | Combinación equilibrada de rendimiento     |

# Paso 2: Versión Caso zapato
    Si PF == 0:  
        Eff = 0.7×Win_Rate + 1 ×DRP_Híbrida
    Si no:
        Eff = 0.7×Win_Rate + 0.3×DRP_Híbrida

# Paso 3: Escenario Efectividad Negativa
    Si PF < PC  
        -DRP_Híbrida
        Eff = 0.7×Win_Rate + 0.3×DRP_Híbrida_negativa 



# 📌 Definición de DRP en Dominó (Fórmulas y Significado)

## 🔎 **¿Qué es DRP?**
**DRP** = *Diferencia Relativa de Puntos*  
Métrica que mide **eficiencia en el manejo de puntos** 

**símbolo Σ** = *Sigma* , suma lineal   
 


## ⚖️ Criterios de Desempate (Orden de Prioridad)
1. **Partidas Ganadas (PG)**.  
2. **Efectividad (Eff)**.  
3. **Diferencia de Puntos (Diff)** (`PF - PC`).  
4. **Puntos a Favor (PF)**.  
5. **Enfrentamiento Directo** (si hubo partida entre los empatados).  

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
