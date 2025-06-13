# Algoritmo de Emparejamiento Sistema Suizo para Dominó Profesional
## Parejas Rotativas - Versión para Auditoría

### 1. Introducción y Principios Fundamentales

El presente algoritmo gestiona el emparejamiento de jugadores en torneos de dominó profesional bajo el sistema suizo con parejas rotativas. La clasificación es individual, pero en cada ronda se forman nuevas parejas que compiten contra otras parejas.

#### 1.1 Objetivos Principales
- Mantener la integridad competitiva del sistema suizo
- Evitar repetición de compañeros en rondas consecutivas
- Minimizar repetición de rivales durante todo el torneo
- Garantizar emparejamientos justos según nivel de desempeño
- Adaptarse a diferentes tamaños de torneo

#### 1.2 Categorías de Torneo
- **COMPACTA**: ≤ 36 participantes
- **ESTÁNDAR**: 40-76 participantes  
- **INTERNACIONAL**: 80+ participantes

### 2. Estructura de Pirámides

#### 2.1 Concepto de Pirámides
En cada ronda, todos los jugadores se dividen en dos pirámides basándose en su clasificación general actual:

- **Pirámide Principal (PP)**: Primera mitad de jugadores con mejor desempeño
- **Pirámide Secundaria (PS)**: Segunda mitad de jugadores

Cada pirámide se subdivide en dos lados:
- **Lado A**: Mejor mitad dentro de cada pirámide
- **Lado B**: Segunda mitad dentro de cada pirámide

#### 2.2 Cálculo de División
1. Total de jugadores en pirámide ÷ 2 = jugadores por lado
2. Si el resultado tiene decimales, restar 0.5 para obtener número entero de parejas

### 3. Reglas de Emparejamiento

#### 3.1 Restricciones Obligatorias

**Regla de Compañeros (RC):**
- Un jugador NO puede repetir compañero hasta cumplirse una ventana mínima de rondas
- Ventana varía según categoría del torneo

**Regla de Rivales (RR):**
- Un jugador NO puede enfrentar a los mismos rivales hasta cumplirse una ventana mínima
- Ventana más estricta que la de compañeros

#### 3.2 Ventanas por Categoría

| Categoría | Ventana Compañeros | Ventana Rivales |
|-----------|-------------------|-----------------|
| COMPACTA | +1 ronda | +2 rondas |
| ESTÁNDAR | +2 rondas | +3 rondas |
| INTERNACIONAL | +3 rondas | +4 rondas |

### 4. Proceso de Emparejamiento por Ronda

#### 4.1 Primera Ronda
1. Ordenar jugadores aleatoriamente o por ELO (decisión del administrador)
2. Formar parejas consecutivas: A1-A2, B1-B2, C1-C2...
3. Enfrentar parejas: (A1-A2 vs B1-B2), (C1-C2 vs D1-D2)...

#### 4.2 Rondas Posteriores

**Paso 1: Obtener Clasificación**
- Recibir lista ordenada de jugadores según índices (WR, EFF, etc.)
- La lista ya viene ordenada descendentemente por el sistema de clasificación

**Paso 2: Formar Pirámides**
1. Dividir lista total en dos mitades
2. Primera mitad → Pirámide Principal (PP)
3. Segunda mitad → Pirámide Secundaria (PS)

**Paso 3: Dividir en Lados**
1. Cada pirámide se divide en Lado A y Lado B
2. Aplicar fórmula de cálculo para determinar tamaño de cada lado

**Paso 4: Emparejamiento Base**
1. En PP Lado A: emparejar posición 1 con 3, 2 con 4, 5 con 7...
2. En PP Lado B: continuar patrón par-impar
3. Repetir proceso en PS

**Paso 5: Validación de Restricciones**
Para cada pareja propuesta, verificar:
- ¿Los jugadores fueron compañeros dentro de la ventana prohibida?
- Para cada enfrentamiento, verificar:
  - ¿Algún jugador enfrentó a sus rivales propuestos dentro de la ventana?

**Paso 6: Manejo de Conflictos**
Si se detectan violaciones:
1. Intentar intercambio con siguiente jugador disponible del mismo lado
2. Si no es posible, buscar en el lado opuesto de la misma pirámide
3. Como último recurso, buscar en la otra pirámide

### 5. Casos Especiales

#### 5.1 Mesas Impares
Cuando el número de mesas es impar:
1. La última pareja de PP sin rival se desempareja
2. Estos jugadores se emparejan con los mejores de PS
3. Enfrentan a la pareja más fuerte disponible de PS Lado B

#### 5.2 Jugadores Impares (BYE)
Si hay número impar de jugadores:
1. El jugador con peor clasificación recibe BYE
2. Se le asigna victoria con puntaje promedio del torneo
3. No cuenta para restricciones de compañero/rival

#### 5.3 Abandonos Durante el Torneo
1. Si abandona antes de una ronda: aplicar BYE al rival sin pareja
2. Si abandona durante partida: victoria para rivales
3. Jugador que abandona se elimina de futuros emparejamientos

### 6. Sistema de Contingencia

#### 6.1 Nivel 1: Intercambio Local
- Buscar alternativas dentro del mismo lado de la pirámide
- Mantener integridad competitiva

#### 6.2 Nivel 2: Intercambio Inter-Lados
- Permitir intercambios entre Lado A y B de la misma pirámide
- Solo si Nivel 1 no encuentra solución

#### 6.3 Nivel 3: Transfusión Entre Pirámides
- Intercambiar últimos lugares de PP con primeros de PS
- Aplicar solo en casos extremos
- Máximo 2 parejas por ronda

#### 6.4 Nivel 4: Relajación de Ventanas
Si ningún nivel anterior funciona:
- Reducir ventanas en -1 ronda temporalmente
- Aplicar solo a los jugadores afectados
- Registrar excepción para auditoría

### 7. Algoritmo de Búsqueda de Pareja Válida

Para cada jugador sin pareja:
1. Crear lista de candidatos del mismo lado
2. Filtrar por restricción de compañeros
3. Ordenar por cercanía en clasificación
4. Seleccionar primer candidato válido
5. Si no hay candidatos, activar contingencia

### 8. Algoritmo de Validación de Mesa

Para cada mesa propuesta (4 jugadores):
1. Verificar que ningún jugador repita compañero dentro de ventana
2. Verificar que ningún jugador enfrente rival anterior dentro de ventana
3. Si hay violación, marcar mesa para reconfiguración
4. Aplicar sistema de contingencia si es necesario

### 9. Numeración de Mesas

1. Ordenar mesas por suma de posiciones de los 4 jugadores
2. Mesa con menor suma = Mesa 1
3. Continuar ascendentemente
4. La pareja líder siempre debe estar en Mesa 1

### 10. Consideraciones de Escalabilidad

#### 10.1 Complejidad del Algoritmo
- Rondas 1-3: Emparejamientos fluidos en todas las categorías
- Rondas 4-6: Posibles conflictos en categoría COMPACTA
- Rondas 7-9: Requiere flexibilidad incluso en categoría ESTÁNDAR

#### 10.2 Métricas de Éxito
- **Óptimo**: 100% emparejamientos sin violar reglas
- **Aceptable**: >85% emparejamientos válidos, <15% con contingencia Nivel 1-2
- **Crítico**: Requiere contingencia Nivel 3-4

### 11. Registro y Auditoría

El sistema debe registrar:
1. Todos los emparejamientos de cada ronda
2. Violaciones de reglas detectadas
3. Niveles de contingencia aplicados
4. Justificación de excepciones
5. Métricas de calidad del emparejamiento

### 12. Validación Final

Antes de confirmar una ronda:
1. Verificar que todos los jugadores activos estén emparejados
2. Confirmar que no hay violaciones sin resolver
3. Validar numeración correcta de mesas
4. Generar reporte de contingencias aplicadas

### 13. Ejemplo Práctico Completo

**Torneo de 28 jugadores - Ronda 2**

*Entrada:* Lista clasificada tras Ronda 1
```
[A1, A2, C1, C2, M1, M2, G1, G2, I1, I2, E1, E2, K1, K2, 
 L1, L2, F1, F2, J1, J2, H1, H2, N1, N2, D1, D2, B1, B2]
```

*Proceso:*
1. PP = primeros 14 jugadores
2. PS = últimos 14 jugadores
3. PP Lado A = 7 jugadores (3.5 parejas → 3 parejas)
4. Emparejamientos resultantes tras validación:
   - Mesa 1: A1-C1 vs M2-I1
   - Mesa 2: A2-C2 vs G2-I2
   - Mesa 3: M1-G1 vs E1-K1
   - Mesa 4: E2-L1 vs H1-N1
   - Mesa 5: K2-L2 vs H2-N2
   - Mesa 6: F1-J1 vs D1-B1
   - Mesa 7: F2-J2 vs D2-B2

### 14. Conclusión

Este algoritmo garantiza:
- Emparejamientos justos según nivel
- Mínima repetición de compañeros y rivales
- Adaptabilidad a diferentes tamaños de torneo
- Transparencia para auditoría
- Soluciones para casos excepcionales

La implementación debe ser determinista y reproducible para permitir verificación por parte de la federación.
