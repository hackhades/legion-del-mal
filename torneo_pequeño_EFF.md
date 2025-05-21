# 📊 Resumen Breve: Fórmula EFF para Torneos Pequeños y Grandes

---

## 🏆 EFF para Torneos Pequeños (≤12 jugadores)

**Objetivo:** Medir el rendimiento global priorizando victorias, margen de puntos y potencia ofensiva relativa, ideal cuando el Buchholz pierde relevancia.

### **Fórmula:**
\[
EFF_pequeño = 0.45 * Win_Rate + 0.35 * DNP_norm + 0.20 * IAR_norm
\]

- **Win_Rate:** % de partidas ganadas.  
  \[
  Win_Rate = (Victorias / Partidas Jugadas) * 100
  \]
- **DNP_norm:** Dominio Neto por Partida, normalizado a [0,100].  
  \[
  DNP = (PF - PC) / PJ
  \]
  \[
  DNP_norm = (DNP - DNP_min) / (DNP_max - DNP_min) * 100
  \]
- **IAR_norm:** Índice de Ataque Relativo, normalizado a [0,100].  
  \[
  IAR = PF / Media_Ataque
  \]
  \[
  IAR_norm = (IAR - IAR_min) / (IAR_max - IAR_min) * 100
  \]

**Notas:**
- Todos los componentes ponderados están en [0,100], asegurando comparación justa.
- Media_Ataque se calcula como el promedio de PF de todos los jugadores activos (sin exclusiones para grupos pequeños).

---

## 🥇 EFF para Torneos Grandes (≥12 jugadores)

**Objetivo:** Medir eficiencia considerando victorias, dominio ajustado y fuerza de oponentes.

### **Fórmula:**
\[
EFF_grande = 0.45 * Win_Rate + 0.35 * DRP + 0.20 * PBT_norm
\]

- **Win_Rate:** (igual que antes).
- **DRP:** Diferencia Relativa de Puntos.  
  \[
  DRP = (PF - PC) / (Umbral * PJ) * 100
  \]
  donde  
  \[
  Umbral = 200 * (1 + 1/3 * PF / (200 * PJ))
  \]
- **PBT_norm:** Performance de Oponentes (Buchholz truncado), normalizado a [0,100].

---

## 📌 Comparativa Clave

| Característica              | EFF Pequeño             | EFF Grande                  |
|-----------------------------|:-----------------------:|:---------------------------:|
| Dependencia de rivales      | Baja (sólo ataque)      | Alta (Buchholz y DRP)       |
| Robustez a abandonos        | Alta                    | Media                       |
| Mejor para                  | ≤12 jugadores           | ≥12 jugadores               |
| Ventaja principal           | Simplicidad, equidad    | Precisión, ajuste real      |

---

## 🧮 Fórmulas Subyacentes

- **DNP_norm:**  
  \[
  DNP_norm = (DNP - DNP_min) / (DNP_max - DNP_min) * 100
  \]
- **IAR_norm:**  
  \[
  IAR_norm = (IAR - IAR_min) / (IAR_max - IAR_min) * 100
  \]
- **DRP:**  
  \[
  DRP = (PF - PC) / (Umbral * PJ) * 100
  \]

---

## 🚀 **Conclusión Final**

- **Torneos pequeños:** Usa `EFF_pequeño` (Win Rate, DNP_norm, IAR_norm).  
  → Simple, justo, sin depender de la calidad de rivales.
- **Torneos grandes:** Usa `EFF_grande` (Win Rate, DRP, PBT_norm).  
  → Preciso, considera dificultad real del torneo.

¡Ideal para implementar, documentar y asegurar equidad en cualquier formato competitivo!