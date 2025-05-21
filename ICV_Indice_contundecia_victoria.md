# 📊 Resumen Breve: Fórmula EFF para Torneos Pequeños y Grandes

---

## 🏆 EFF para Torneos Pequeños (≤12 jugadores)

**Objetivo:** Medir el rendimiento global priorizando victorias, margen de puntos y potencia ofensiva relativa, ideal cuando el Buchholz pierde relevancia.

### **Fórmula:**
\[
EFF_{pequeño} = 0.45 \times Win\_Rate + 0.35 \times DNP_{norm} + 0.20 \times IAR_{norm}
\]

- **Win_Rate:** % de partidas ganadas.  
  \[
  Win\_Rate = (Victorias / Partidas\ Jugadas) \times 100
  \]
- **DNP_norm:** Dominio Neto por Partida, normalizado a [0,100].  
  \[
  DNP = (PF - PC) / PJ
  \]
  \[
  DNP_{norm} = \frac{DNP - DNP_{min}}{DNP_{max} - DNP_{min}} \times 100
  \]
- **IAR_norm:** Índice de Ataque Relativo, normalizado a [0,100].  
  \[
  IAR = PF / Media\_Ataque
  \]
  \[
  IAR_{norm} = \frac{IAR - IAR_{min}}{IAR_{max} - IAR_{min}} \times 100
  \]

**Notas:**
- Todos los componentes ponderados están en [0,100], asegurando comparación justa.
- Media_Ataque se calcula como el promedio de PF de todos los jugadores activos (sin exclusiones para grupos pequeños).

---

## 🥇 EFF para Torneos Grandes (≥12 jugadores)

**Objetivo:** Medir eficiencia considerando victorias, dominio ajustado y fuerza de oponentes.

### **Fórmula:**
\[
EFF_{grande} = 0.45 \times Win\_Rate + 0.35 \times DRP + 0.20 \times PBT_{norm}
\]

- **Win_Rate:** (igual que antes).
- **DRP:** Diferencia Relativa de Puntos.  
  \[
  DRP = \frac{PF - PC}{Umbral \times PJ} \times 100
  \]
  donde  
  \[
  Umbral = 200 \times \left(1 + \frac{1}{3} \frac{PF}{200 \times PJ}\right)
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
  DNP_{norm} = \frac{DNP - DNP_{min}}{DNP_{max} - DNP_{min}} \times 100
  \]
- **IAR_norm:**  
  \[
  IAR_{norm} = \frac{IAR - IAR_{min}}{IAR_{max} - IAR_{min}} \times 100
  \]
- **DRP:**  
  \[
  DRP = \frac{PF - PC}{Umbral \times PJ} \times 100
  \]

---

## 🚀 **Conclusión Final**

- **Torneos pequeños:** Usa `EFF_pequeño` (Win Rate, DNP_norm, IAR_norm).  
  → Simple, justo, sin depender de la calidad de rivales.
- **Torneos grandes:** Usa `EFF_grande` (Win Rate, DRP, PBT_norm).  
  → Preciso, considera dificultad real del torneo.

¡Ideal para implementar, documentar y asegurar equidad en cualquier formato competitivo!