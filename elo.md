# 🏆 Sistema ELO para Torneos de Dominó (Parejas Rotativas)

## 📌 **Fórmulas Principales**

### 1. **Cálculo del ELO del Equipo**
\[
R_{\text{eq}} = \frac{R_{\text{Jugador 1}} + R_{\text{Jugador 2}}}{2}
\]

### 2. **Probabilidad Esperada de Victoria**
\[
E_{\text{eq}} = \frac{1}{1 + 10^{(R_{\text{rival}} - R_{\text{propio}})/400}}
\]

### 3. **Actualización del ELO Individual**
\[
R' = R + K \cdot (S - E_{\text{eq}}) + \text{Bono\_Aplastante}
\]
- **\(S\)**: 1 (victoria) / 0 (derrota).
- **Bono_Aplastante**: Ver sección 4.

---

## 🔥 **Bono por Victoria Aplastante**
**Condiciones**:  
1. El equipo ganador **alcanza o supera el puntaje objetivo** (ej. 100/200 puntos).  
2. La **brecha porcentual** cumple:  
   \[
   \text{Brecha\%} = \left(\frac{\text{Puntos Ganador} - \text{Puntos Perdedor}}{\text{Puntos Ganador}}\right) \times 100\%
   \]
   - **+3 ELO** → 100% (ej. 100-0).  
   - **+2 ELO** → ≥90% (ej. 100-10).  
   - **+1 ELO** → ≥80% (ej. 100-20).  

**Ejemplo**:  
- Ganador: 100-15 → Brecha = 85% → **+1 ELO**.  
- Ganador: 84-16 → **No aplica** (no alcanzó 100 puntos).  

---

## ⏳ **Ajustes por Inactividad**
### 1. **Inactividad ≥6 meses**
- **Primeras 5 partidas de regreso**:  
  - \(K = 30\) (sin importar su ELO).  
- **Objetivo**: Reajustar rápido su ELO real.  

### 2. **Inactividad ≥12 meses**
- **Decaimiento mensual**:  
  \[
  \text{ELO} = \text{ELO} \times 0.98 \quad (\text{-2\% por mes, máximo -10\%})
  \]
  - Ejemplo: 2000 ELO → 1960 tras 1 año.  

---

## ⚠️ **Casos Especiales**
### 1. **Abandono (Conducta Antideportiva)**
- **Jugador que abandona**:  
  \[
  R' = R - 7 \quad (\text{Penalización fija})
  \]
- **Equipo contrario**: Sin cambios en su ELO.  

### 2. **Inasistencia (Walkover)**
- **Equipo presente**: Sin cambios.  
- **Equipo ausente**: Sin cambios.  

---

## 🎯 **Parámetros Clave**
| **Concepto**          | **Valor**                               |
|-----------------------|-----------------------------------------|
| ELO inicial           | 1200 (o 1500)                           |
| Factor K (nuevos)     | 40 (<1600 ELO o <30 partidas)           |
| Factor K (intermedios)| 25 (1600-2000 ELO)                      |
| Factor K (élite)      | 10 (>2000 ELO)                          |
| Penalización abandono | -7 ELO                                  |
| Bono aplastante       | +1/+2/+3 ELO (según brecha)             |

---

## 📝 **Ejemplo Integral**
**Partida a 100 puntos**:  
- **Equipo A** (1700 ELO promedio) vs **Equipo B** (1800 ELO promedio).  
- **Resultado**: A gana 100-15.  
- **Cálculos**:  
  1. \(E_{\text{eq}} = 0.36\) (36% de probabilidad).  
  2. **Brecha%**: 85% → **+1 ELO**.  
  3. **Actualización** (jugador con \(R = 1500\), \(K = 25\)):  
     \[
     R' = 1500 + 25 \cdot (1 - 0.36) + 1 = 1517
     \]

---

## 📌 **Reglas de Oro**
1. **Transparencia**: Todos los parámetros deben ser públicos.  
2. **Consistencia**: Aplicar siempre las mismas reglas.  
3. **Registro histórico**: Guardar datos de partidas y ELOs.  