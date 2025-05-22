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
- Media_Ataque se calcula como el promedio de PF de todos los jugadores activos (sin excluir menor ni mayor ya que hay pocos jugadores).

---

## 🥇 Calcular IAR

**Objetivo:** Esto escala la desviación para que la máxima desviación observada (positiva o negativa) se mapee a +100 o -100 respectivamente.

  Simulación del IAR_final_component
  
  Supongamos un torneo pequeño con 5 jugadores (A, B, C, D, E) y hemos calculado sus PF y la Media de Ataque Simple:
  
  PF_A = 1500
  
  PF_B = 1200
  
  PF_C = 1000
  
  PF_D = 700
  
  PF_E = 600
  
  Suma_PF = 1500+1200+1000+700+600 = 5000
  
  Media_Ataque_Simple = 5000 / 5 = 1000
  
  Paso 1: Calcular IAR_crudo_j = PF_j / Media_Ataque_Simple
  
  IAR_crudo_A = 1500 / 1000 = 1.5
  
  IAR_crudo_B = 1200 / 1000 = 1.2
  
  IAR_crudo_C = 1000 / 1000 = 1.0 (Exactamente la media)
  
  IAR_crudo_D = 700 / 1000 = 0.7
  
  IAR_crudo_E = 600 / 1000 = 0.6
  
  Paso 2: Calcular IAR_desviacion_j = IAR_crudo_j - 1.0
  
  IAR_desviacion_A = 1.5 - 1.0 = +0.5
  
  IAR_desviacion_B = 1.2 - 1.0 = +0.2
  
  IAR_desviacion_C = 1.0 - 1.0 = 0.0
  
  IAR_desviacion_D = 0.7 - 1.0 = -0.3
  
  IAR_desviacion_E = 0.6 - 1.0 = -0.4
  
  Paso 3: Calcular Max_Abs_Desv_IAR_Torneo = max(abs(IAR_desviacion_j))
  
  Max_Abs_Desv_IAR_Torneo = max(0.5, 0.2, 0.0, 0.3, 0.4) = 0.5
  
  Paso 4: Calcular IAR_final_component
  
  Fórmula: IAR_final_component = (IAR_desviacion_j / Max_Abs_Desv_IAR_Torneo) * 100
  
  
  IAR_final_component_B: (0.2 / 0.5) * 100 = 0.4 * 100 = +40
  
  Interpretación: B estuvo por encima de la media, su desviación fue el 40% de la máxima desviación absoluta observada.
    
  Resultados del IAR_final_component:
  
  A: +100
  
  B: +40
  
  C: 0
  
  D: -60
  
  E: -80

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
  - IAR_componente_EFF_escalado_a_cero_en_1.0 se calcula como:

  - IAR_crudo_j = PF_j / Media_Ataque

  - IAR_desviacion_j = IAR_crudo_j - 1.0
 
  - Calcular Max_Abs_Desviacion_Observada_en_torneo de todos los IAR_desviacion_j.
 
 
  - IAR_componente_EFF_escalado_a_cero_en_1.0 = (IAR_desviacion_j / Max_Abs_Desviacion_Observada_en_torneo) * 100
  

Análisis
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