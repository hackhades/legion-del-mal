# Algoritmo del Sistema Suizo para Torneos de Dominó Profesional con Parejas Rotativas

## 1. Introducción y Principios Fundamentales

El presente algoritmo describe el funcionamiento del sistema suizo adaptado para torneos de dominó profesional con parejas rotativas. Este sistema garantiza:

- Competitividad equilibrada mediante emparejamientos por nivel de desempeño
- Rotación obligatoria de parejas en cada ronda
- Prevención de repetición de enfrentamientos
- Escalabilidad para torneos de diferentes tamaños

### 1.1 Definiciones Clave

- **Atleta**: Jugador individual que participa en el torneo
- **Pareja**: Dos atletas que juegan juntos en una ronda específica
- **Mesa**: Enfrentamiento entre dos parejas (4 atletas)
- **Pirámide**: Agrupación de atletas según su desempeño acumulado
- **Ventana de restricción**: Número de rondas que deben transcurrir antes de permitir repeticiones

## 2. Categorías de Torneo y Ventanas de Restricción

### 2.1 Categoría Compacta (28-36 participantes)
- **Restricción de compañeros**: +1 ronda (no pueden ser pareja en la ronda inmediatamente siguiente)
- **Restricción de rivales**: +2 rondas (no pueden enfrentarse en las siguientes 2 rondas)

### 2.2 Categoría Estándar (40-76 participantes)
- **Restricción de compañeros**: +2 rondas
- **Restricción de rivales**: +3 rondas

### 2.3 Categoría Internacional (80+ participantes)
- **Restricción de compañeros**: +3 rondas
- **Restricción de rivales**: +4 rondas

## 3. Proceso de Emparejamiento por Ronda

### 3.1 Primera Ronda

1. **Ordenamiento inicial**:
   - Si el administrador elige ordenamiento por ELO: ordenar atletas por rating
   - Si elige aleatorio: mezclar aleatoriamente la lista de atletas

2. **Formación de parejas**:
   - Emparejar consecutivamente: 1-2, 3-4, 5-6, etc.

3. **Formación de mesas**:
   - Enfrentar parejas consecutivas: (1-2 vs 3-4), (5-6 vs 7-8), etc.

### 3.2 Rondas Subsiguientes (2 en adelante)

#### Paso 1: Clasificación y Ordenamiento

1. **Calcular índices de desempeño** para cada atleta:
   - Puntos acumulados (victorias)
   - Win Rate (WR): porcentaje de victorias
   - Efectividad (EFF): fórmula compuesta que mide dominio relativo
   - Índice combinado = (Puntos × 1000) + (WR × 100) + (EFF × 10)

2. **Ordenar atletas** por índice combinado de mayor a menor

#### Paso 2: División en Pirámides

1. **Ordenar TODOS los atletas por clasificación general acumulada**
   - NO se separa por ganadores/perdedores de la última ronda
   - Se usa el índice de desempeño total acumulado desde el inicio del torneo
   - Un atleta puede haber perdido la última ronda y aún liderar la clasificación general

2. **Dividir la lista completa en dos pirámides**:
   - **Pirámide Superior**: Primera mitad de atletas (mejor clasificación general)
   - **Pirámide Inferior**: Segunda mitad de atletas (menor clasificación general)
   
3. **Ejemplo aclaratorio**:
   - Atleta A1: 3 victorias, 1 derrota = 3 puntos, WR 75%, EFF alta → Posición 1° general
   - Atleta B1: 4 victorias, 0 derrotas = 4 puntos, WR 100%, EFF media → Posición 2° general
   - A pesar de que A1 perdió la última ronda, sigue liderando por su efectividad acumulada

#### Paso 3: Formación de Parejas con Validación

Para cada pirámide (Pirámide Superior e Inferior):

1. **Calcular número de parejas del Lado A**:
   ```
   parejas_lado_A = floor(total_atletas_lado_A / 2)
   ```

2. **Dividir la pirámide**:
   - Lado A: Primera mitad de atletas (mejor desempeño)
   - Lado B: Segunda mitad de atletas (menor desempeño)

3. **Algoritmo de emparejamiento con validación**:
   
   a) **Intento inicial - Emparejamiento par-impar**:
      - Del Lado A: emparejar 1° con 3°, 2° con 4°, etc.
      - Del Lado B: mismo patrón
   
   b) **Validación de restricciones** para cada pareja propuesta:
      - Verificar que no fueron compañeros dentro de la ventana de restricción
      - Si falla, buscar siguiente candidato válido en el mismo lado
   
   c) **Mecanismo de intercambio**:
      - Si no hay candidato válido en el mismo lado, intercambiar con atleta del otro lado
      - Mantener el balance de niveles mediante intercambios recíprocos

#### Paso 4: Formación de Mesas con Validación

1. **Enfrentamiento cruzado**:
   - Parejas del Lado A enfrentan a parejas del Lado B
   - Primera pareja del Lado A vs Primera pareja del Lado B, etc.

2. **Validación de rivales**:
   - Verificar que ningún atleta de una pareja haya enfrentado a ningún atleta de la pareja rival dentro de la ventana de restricción
   - Si falla, rotar parejas del Lado B hasta encontrar enfrentamiento válido

3. **Numeración de mesas**:
   - Mesa 1: Pareja líder absoluta del torneo
   - Numeración ascendente según posición en la clasificación

## 4. Manejo de Situaciones Especiales

### 4.1 Número Impar de Participantes

1. **Asignación de BYE**:
   - El atleta con peor desempeño recibe BYE (descanso)
   - Recibe puntos equivalentes a un empate
   - No afecta sus índices de efectividad

### 4.2 Abandono o Inasistencia Durante el Torneo

1. **Retirar al atleta** del pool de emparejamiento
2. **Si genera número impar**, aplicar regla de BYE
3. **Actualizar histórico** marcando como "no jugado" para no afectar estadísticas de rivales

### 4.3 Imposibilidad de Emparejamiento Válido

Aplicar en orden hasta resolver:

1. **Relajación temporal de restricciones**:
   - Reducir ventana de restricción en 1 ronda solo para casos problemáticos
   - Priorizar restricción de rivales sobre restricción de compañeros

2. **Intercambio entre pirámides**:
   - Permitir que últimos clasificados de pirámide superior intercambien con primeros de pirámide inferior
   - Máximo 2 atletas por pirámide

3. **Emparejamiento forzado documentado**:
   - Como último recurso, permitir repetición documentando la excepción
   - Requiere aprobación del director del torneo

## 5. Algoritmo de Validación de Restricciones

### 5.1 Estructura de Datos Históricos

Mantener para cada atleta:
- Lista de ex-compañeros con número de ronda
- Lista de ex-rivales con número de ronda

### 5.2 Función de Validación de Compañero

```
VALIDAR_COMPAÑERO(atleta1, atleta2, ronda_actual):
    última_ronda_juntos = buscar_última_ronda_como_compañeros(atleta1, atleta2)
    
    SI última_ronda_juntos existe:
        ventana = obtener_ventana_restricción_compañeros()
        SI (ronda_actual - última_ronda_juntos) <= ventana:
            RETORNAR FALSO
    
    RETORNAR VERDADERO
```

### 5.3 Función de Validación de Rivales

```
VALIDAR_RIVALES(pareja1, pareja2, ronda_actual):
    PARA cada atleta en pareja1:
        PARA cada atleta en pareja2:
            última_ronda_enfrentados = buscar_última_ronda_como_rivales(atleta1, atleta2)
            
            SI última_ronda_enfrentados existe:
                ventana = obtener_ventana_restricción_rivales()
                SI (ronda_actual - última_ronda_enfrentados) <= ventana:
                    RETORNAR FALSO
    
    RETORNAR VERDADERO
```

## 6. Consideraciones para la Escalabilidad

### 6.1 Optimización por Tamaño de Torneo

- **28-36 jugadores**: Aplicar algoritmo completo con validaciones estrictas
- **40-76 jugadores**: Implementar caché de emparejamientos válidos
- **80+ jugadores**: Usar algoritmo de sectores para reducir espacio de búsqueda

### 6.2 Prevención de Bucles Infinitos

1. **Límite de intentos**: Máximo 100 iteraciones por emparejamiento
2. **Degradación gradual**: Relajar restricciones progresivamente
3. **Registro de intentos fallidos**: Para análisis post-torneo

## 7. Métricas de Calidad del Emparejamiento

### 7.1 Indicadores de Éxito

- **Tasa de emparejamientos válidos**: > 95% sin relajación
- **Distribución de enfrentamientos**: Desviación estándar < 15%
- **Tiempo de cálculo**: < 30 segundos por ronda

### 7.2 Alertas de Calidad

- Más de 5% de emparejamientos con relajación
- Atletas con más de 2 BYEs
- Tiempo de cálculo > 60 segundos

## 8. Ejemplo Práctico de Aplicación

### Ronda 2 con 28 jugadores (7 mesas)

1. **Clasificación general después de ronda 1**:
   - Se ordenan TODOS los 28 atletas por su índice de desempeño acumulado
   - NO se separan por resultado de la última ronda
   - Ejemplo: A1 (1 victoria, índice 1850) lidera aunque otros tengan igual número de victorias

2. **División en pirámides por clasificación general**:
   - Pirámide Superior: Atletas posición 1-14 en clasificación general
   - Pirámide Inferior: Atletas posición 15-28 en clasificación general
   - Si hay mesas impares, ajustar moviendo los últimos 2 de pirámide superior

3. **Emparejamiento con validación**:
   - Dentro de cada pirámide, formar parejas respetando restricciones
   - Enfrentar parejas de Pirámide Superior vs Pirámide Inferior
   - Mesa 1 siempre para la pareja con mejor clasificación general

### Ejemplo de Clasificación General en Ronda 5

| Pos | Atleta | V-D | Puntos | WR% | EFF | Índice | Última |
|-----|--------|-----|--------|-----|-----|--------|---------|
| 1   | A1     | 3-1 | 3.0    | 75  | 92  | 3920   | Perdió  |
| 2   | C2     | 4-0 | 4.0    | 100 | 85  | 4850   | Ganó    |
| 3   | M1     | 3-1 | 3.0    | 75  | 88  | 3880   | Ganó    |
| ... | ...    | ... | ...    | ... | ... | ...    | ...     |

A pesar de que A1 perdió en ronda 4, mantiene el liderato por su efectividad acumulada.

## 9. Conclusión y Garantías

Este algoritmo garantiza:

1. **Integridad competitiva**: Emparejamientos justos por nivel
2. **Variedad de enfrentamientos**: Rotación sistemática
3. **Transparencia**: Proceso auditable y reproducible
4. **Flexibilidad**: Adaptable a diferentes tamaños y situaciones

El sistema ha sido diseñado para mantener el espíritu del sistema suizo mientras se adapta a las particularidades del dominó profesional con parejas rotativas.
