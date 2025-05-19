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
| **Índice de Eficiencia** | EFF     | Medida de eficiencia y rendimiento global (puede variar según reglamento).                             |
| **Índice de Dominio Normalizado** | IDN | [(PF - PC)/(PF + PC)] × 100. Refleja el dominio relativo normalizado sobre el total jugado.|
| **Performance de Oponentes** | Prom_TBz | Suma o promedio de los puntos de los rivales enfrentados (Buchholz truncado).           |
| **Puntos a Favor**  | PF          | Total de puntos anotados.                                                                |
| **Puntos en Contra**| PC          | Total de puntos recibidos.                                                               |
| **Diferencia de Puntos** | DIFF    | PF - PC. Saldo neto de puntos.                                                          |
| **Win Rate**        | WR       | (Victorias / Partidas Jugadas) × 100. Porcentaje de partidas ganadas.                   |
| **ELO Rating**      | ELO         | Valoración internacional basada en el sistema ELO.                                       |
| **Índice de Ataque Relativo** | IAR  | (PF_jugador / Media_Ataque_Ajustada) × 100. (%) del Poder ofensivo relativo al promedio.       |
| **Dominio Neto por Partida** | DNP  | DIFF / PJ. Saldo neto de puntos por partida.                                            |
| **Promedio de Puntos por Partida** | AVG | PF / PJ. Promedio de puntos a favor por partida jugada.                                 |

---

## 📝 Ejemplo Visual de Tabla

| POS | Jugador/Club | País | PJ | V | D | EFF | IDN (%) | Prom_TBz | PF  | PC  | DIFF | PF/PC | WR (%) | ELO | IAR (%) | DNP | AVG |
|-----|--------------|------|----|---|---|-----|---------|----------|-----|-----|------|-------|--------|-----|---------|-----|-----|
|  1  | Ana Team     | ESP  |  7 | 6 | 1 | 0.83|  14.3   |   36.4   |1400 |1200 | +200 | 1.17  | 85.7   |2100 | 120     | 28.6| 200 |
|  2  | Juan Club    | VEN  |  7 | 5 | 2 | 0.71|   5.0   |   32.1   |1300 |1250 |  +50 | 1.04  | 71.4   |2080 | 104     | 7.1 |185.7|
| ... | ...          | ...  | ...|...|...| ... |   ...   |   ...    | ... | ... |  ... |  ...  | ...    | ... | ...     | ... | ... |

---

## 💡 Notas de Interpretación

- **EFF:** Puede representar eficiencia táctica, estratégica o basada en criterios propios del torneo.
- **IDN (%):** Valores positivos indican dominio neto, negativos dominio adverso.
- **Prom_TBz:** Útil como criterio de desempate en sistema suizo.
- **IAR (%):** >100% indica ataque superior al promedio ajustado del torneo.
- **DNP:** Muestra el saldo de puntos que el jugador obtiene por partida.
- **AVG:** Mide la capacidad ofensiva media por ronda.

---

> **Recomendación:** Puedes adaptar el formato y agregar colores o iconos para destacar posiciones, mejores rendimientos o líderes en cada índice.