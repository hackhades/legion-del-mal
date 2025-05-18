# Índice de Ataque Relativo (IAR) - Documentación Técnica

## 📌 Definición
**IAR** (_Índice de Ataque Relativo_) mide la fuerza ofensiva de un jugador en comparación con un promedio ajustado del torneo, excluyendo:
- **Outliers dominantes** (jugadores con PF anormalmente altos).
- **Outliers débiles** (jugadores con PF muy bajos *que sí completaron el torneo*).

---

## 🧮 Fórmula Base
```python
IAR = PF_jugador / Media_Ataque_Ajustada


Cálculo de la Media Ajustada
Ordenar a todos los jugadores por PF (de mayor a menor).

Exclusiones:

✅ Primer jugador (mayor PF).

✅ Último jugador (menor PF) solo si completó todas las rondas.

❌ Si el último tuvo inasistencias, excluir también al penúltimo.

Media_Ajustada = (Suma_PF_total - PF_primer - PF_último[? - PF_penúltimo]) / (N_jugadores - 2[o 3])


📊 Ejemplos Prácticos
Caso 1: Último jugador activo
python
PF_torneo = [500, 300, 250, 200, 150]  # Último jugó todas las rondas
Exclusiones: 500 (1°) y 150 (último).

Media ajustada: (300 + 250 + 200) / 3 = 250

IAR (PF=200): 200 / 250 = 0.8 → Ataque inferior al promedio.

Caso 2: Último jugador inactivo
python
PF_torneo = [500, 300, 250, 200, 50]  # Último abandonó
Exclusiones: 500 (1°), 50 (último) y 200 (penúltimo).

Media ajustada: (300 + 250) / 2 = 275

IAR (PF=250): 250 / 275 ≈ 0.91 → Evita penalizar por el abandonante.



def calcular_IAR(PF_jugador, lista_PF_todos, inasistencias_ultimo=False):
    # Ordenar jugadores por PF (descendente)
    lista_ordenada = sorted(lista_PF_todos, reverse=True)
    N = len(lista_ordenada)
    
    # Casos mínimos
    if N <= 2:
        return PF_jugador / (sum(lista_ordenada) / N) if N > 0 else 0
    
    # Lógica de exclusión
    if not inasistencias_ultimo:
        suma_ajustada = sum(lista_ordenada[1:-1])  # Excluye 1° y último
        media_ajustada = suma_ajustada / (N - 2)
    else:
        suma_ajustada = sum(lista_ordenada[1:-2])  # Excluye 1° y últimos 2
        media_ajustada = suma_ajustada / (N - 3)
    
    return round(PF_jugador / media_ajustada, 2)



Parámetros:
Parámetro	Tipo	Descripción
PF_jugador	float	Puntos a favor del jugador a evaluar
lista_PF_todos	list[float]	Lista de PF de todos los jugadores
inasistencias_ultimo	bool	Si True, excluye también al penúltimo
