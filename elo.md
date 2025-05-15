# 🏆 Sistema ELO para Torneos de Dominó (Parejas Rotativas)

## 📌 **Fórmulas Principales**

### 1. **Cálculo del ELO del Equipo**
\[
ELO_pareja = (elo_jugador_A + elo_jugador_B) / 2
\]


### 2. **Probabilidad Esperada de Victoria**
\[
E_eq = 1 / (1 + 10^((R_rival - R_propio) / 400))
\]

>function calcularExpectativa(ELO_rivalPar, ELO_propioPar) {
>  
>  let exponente = (ELO_rivalPar - ELO_propioPar) / 400;
>
>  let potencia = Math.pow(10, exponente);
>
>  let potenciaTruncada = parseFloat(potencia.toFixed(3));
>
>  let expectativa = 1 / (1 + potenciaTruncada);
>
>  let E_eq = parseFloat(expectativa.toFixed(2)); 
>  
>  // toFixed() Redondea a decimales
>  // Usando Math.pow()
>  // 1O base de la exponenciacion
>  // let expectativa = 1 / (1 + 10 ** exponente); // Usando "**" seria igual que "^" 
>  
>  return E_eq;
>
>}


### 3. **Actualización del ELO Individual**
\[
ELO = ELO + K * (S - E_eq) + Bono_Aplastante
\]
- **\(S\)**: 1 (victoria) / 0 (derrota).
- **Bono_Aplastante**: Ver siguiente sección.

---

## 🔥 **Bono por Victoria Aplastante**
**Condiciones**:  
1. El equipo ganador **alcanza o supera el puntaje objetivo** (ej. 100/200 puntos).  
2. La **brecha porcentual** cumple:  
   \[
   Brecha = ((Puntos Ganador - Puntos Perdedor) / Puntos Ganador) * 100
   \]
   - **+3 ELO** → 100% (ej. 100-0).  
   - **+2 ELO** → ≥90% (ej. 100-10).  
   - **+1 ELO** → ≥80% (ej. 100-20).  

**Ejemplo**:  
- Ganador: 100-15 → Brecha = 85% → **+1 ELO**.  
- Ganador: 84-16 → **No aplica** (no alcanzó 100 puntos).  

---

## ⏳ **Ajustes por Inactividad**
### 1. **Inactividad ≥3 meses**
- **Primeras 5 partidas de regreso**:  
  - \(K = 32\) (sin importar su ELO).  
- **Objetivo**: Reajustar rápido su ELO real.  

### 2. **Inactividad ≥6 meses**
- **Decaimiento mensual**:  
  \[
    ELO = ELO * 0.98 (-2% por mes)  \]

  - Ejemplo 1: 2000 ELO → 1960 tras 1 mes. 
  - Ejemplo 2: 2000 ELO → 1569 tras 1 año. 
  - Valor_Final = Valor_Inicial * (1 - Tasa_Descuento)^Número_de_Periodos 
  - Valor_Final = 2000 * (1 - 0.02)^12 

---

## ⚠️ **Casos Especiales**
### 1. **Abandono (Conducta Antideportiva)**
- **Jugador que abandona**:  
  \[
  ELO = ELO - 7 (Penalización fija)
  \]
- **Equipo contrario**: Sin cambios en su ELO.  

### 2. **Inasistencia (Walkover)**
- **Equipo presente**: Sin cambios.  
- **Equipo ausente**: Sin cambios.  

---

## 🎯 **Parámetros Clave**
| **Concepto**          | **Valor**                               |
|-----------------------|-----------------------------------------|
| ELO inicial           | 1200                                    |
| Factor K (nuevos)     | 40 (<1600 ELO o <20 partidas)           |
| Factor K (inactivos)  | 32 (por 5 partidas)                     |
| Factor K (intermedios)| 20 (1600-2000 ELO)                      |
| Factor K (élite)      | 10 (>2000 ELO)                          |
| Penalización abandono | -7 ELO                                  |
| Bono aplastante       | +1/+2/+3 ELO (según brecha)             |

---

## 📝 **Ejemplo Integral**
**Partida a 100 puntos**:  
- **Equipo A** (1700 ELO promedio) vs **Equipo B** (1800 ELO promedio).  
- **Resultado**: A gana 105-15.  
- **S**: 1 por victoria.  
- **Cálculos**:  
  1. (E_eq = 0.36) (36% de probabilidad).  
  2. **Brecha%**: 85.71% → **+1 ELO**.  
  3. **K-factor** 20 jugador activo y establecido.  
  3. **ELO individual actual** 1500.  
     \[
     ELO = 1500 + 20 * (1 - 0.36) + 1 = 1513.8 ≈ 1514
     \]

---

## 📌 **Reglas de Oro**
1. **Transparencia**: Todos los parámetros deben ser públicos.  
2. **Consistencia**: Aplicar siempre las mismas reglas.  
3. **Registro histórico**: Guardar datos de partidas y ELOs.  