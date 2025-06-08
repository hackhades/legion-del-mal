# 🏆 Tabla de Clasificación – Dominó Profesional

Esta tabla está diseñada para torneos oficiales, de élite e internacionales, y contiene los principales índices y métricas recomendadas para el contexto profesional del dominó.

---

## 📊 Columnas y Significado

| Columna         | Abreviatura | ¿Qué representa? / Fórmula                                                                 |
|-----------------|-------------|-------------------------------------------------------------------------------------------|
| **Posición**        | POS         | Puesto en la clasificación general.                                                      |
| **Jugador/Club**    |             | Nombre del jugador o club.                                                               |
| **País**            |             | País de origen del jugador o club.                                                       |
| **Partidas Jugadas**| PJ          | Número total de partidas disputadas.                                                     |
| **Victorias**       | V           | Cantidad de partidas ganadas.                                                            |
| **Derrotas**        | D           | Cantidad de partidas perdidas.                                                           |
| **Índice de Efectividad** | EFF     | Medida de eficiencia y rendimiento global (puede variar según reglamento).                             |
| **Índice de Dominio Normalizado** | IDN % | [(PF - PC)/(PF + PC)] × 100. Refleja el dominio relativo normalizado sobre el total jugado.|
| **Performance de Oponentes** | μTBz | Suma o promedio de los puntos de los rivales enfrentados (Buchholz truncado).           |
| **Puntos a Favor**  | PF          | Total de puntos anotados.                                                                |
| **Puntos en Contra**| PC          | Total de puntos recibidos.                                                               |
| **Diferencia de Puntos** | DIFF    | PF - PC. Saldo neto de puntos.                                                          |
| **Win Rate**        | WR       | (Victorias / Partidas Jugadas) × 100. Porcentaje de partidas ganadas.                   |
| **Indice de Margen por Victoria**  | MOV | (Victorias_amplias / Partidas Jugadas) |
| **Índice de Ataque Relativo** | IAR  | (PF_jugador / Media_Ataque_Ajustada) × 100. (%) del Poder ofensivo relativo al promedio.       |
| **Promedio de Puntos por Partida** | AVG | PF / PJ. Promedio de puntos a favor por partida jugada.                                 |
| **ELO Rating**      | ELO         | Valoración internacional basada en el sistema ELO.                                       |

---

## 📝 Ejemplo Visual de Tabla

| POS | Jugador/Club | País | PJ | V | D | EFF | IDN (%) | Prom_TBz | PF  | PC  | DIFF | PF/PC | WR (%) | ELO | ICV | IAR (%) | AVG |
|-----|--------------|------|----|---|---|-----|---------|----------|-----|-----|------|-------|--------|-----|----|---------|-----|
|  1  | Ana Team     | ESP  |  7 | 6 | 1 | 0.83|  14.3   |   36.4   |1400 |1200 | +200 | 1.17  | 85.7   |2100 | 80 | 120     | 200 |
|  2  | Juan Club    | VEN  |  7 | 5 | 2 | 0.71|   5.0   |   32.1   |1300 |1250 |  +50 | 1.04  | 71.4   |2080 | 65 | 104     |185.7|
| ... | ...          | ...  | ...|...|...| ... |   ...   |   ...    | ... | ... |  ... |  ...  | ...    | ... | ... | ...     | ... |

---

## 💡 Notas de Interpretación

- **EFF:** Puede representar eficiencia táctica, estratégica y rendimiento continuo durante del torneo.
- **IDN (%):** Valores positivos indican dominio neto, negativos dominio adverso.
- **Prom_TBz:** Fuerza de Calendario, útil como criterio de desempate en sistema suizo.
- **IAR (%):** >100% indica ataque superior al promedio ajustado del torneo.
- **AVG:** Mide la capacidad ofensiva media por ronda.
- **Victorias amplias** Victorias con un margen mayor a 50% = 1-(PC/PF) > 0.50

---

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


> **Recomendación:** Puedes adaptar el formato y agregar colores o iconos para destacar posiciones, mejores rendimientos o líderes en cada índice.