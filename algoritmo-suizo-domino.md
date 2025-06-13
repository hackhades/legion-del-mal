# Algoritmo de Emparejamiento Sistema Suizo para Torneos de Dominó Profesional
## Modalidad: Parejas Rotativas

### 1. INTRODUCCIÓN Y PRINCIPIOS FUNDAMENTALES

El presente algoritmo está diseñado para gestionar torneos de dominó profesional utilizando el sistema suizo con parejas rotativas. La clasificación es individual, pero en cada ronda los jugadores forman nuevas parejas y enfrentan nuevos rivales.

#### 1.1 Objetivos del Algoritmo
- Garantizar emparejamientos justos basados en el rendimiento actual
- Evitar repetición de compañeros y rivales según ventanas establecidas
- Mantener la integridad competitiva del sistema suizo
- Ser escalable desde 20 hasta 100+ participantes
- Proporcionar transparencia para auditorías federativas

### 2. CATEGORÍAS DE TORNEO Y PARÁMETROS

El algoritmo se adapta según el número de participantes:

#### 2.1 CATEGORÍA COMPACTA (20-36 participantes)
- **Ventana de no-repetición compañeros**: +1 ronda
- **Ventana de no-repetición rivales**: +2 rondas
- **Flexibilidad**: Alta (permite ajustes tempranos)

#### 2.2 CATEGORÍA ESTÁNDAR (40-76 participantes)
- **Ventana de no-repetición compañeros**: +2 rondas
- **Ventana de no-repetición rivales**: +3 rondas
- **Flexibilidad**: Media

#### 2.3 CATEGORÍA INTERNACIONAL (80+ participantes)
- **Ventana de no-repetición compañeros**: +3 rondas
- **Ventana de no-repetición rivales**: +4 rondas
- **Flexibilidad**: Baja (estricta adherencia a las reglas)

### 3. ESTRUCTURA DE DATOS REQUERIDA

El algoritmo requiere mantener los siguientes registros:

1. **Tabla de Clasificación Actual**
   - Posición, Jugador, Puntos, WR2 (Win Rate), EFF (Efectividad)
   - Índices discriminatorios secundarios

2. **Historial de Emparejamientos**
   - Ronda, Jugador, Compañero, Rival1, Rival2, Resultado

3. **Estado del Torneo**
   - Ronda actual, Total de rondas, Categoría, Jugadores activos

### 4. PROCESO DE EMPAREJAMIENTO POR RONDA

#### 4.1 RONDA 1 - Emparejamiento Inicial

**Paso 1**: Determinar método de emparejamiento inicial
- Opción A: Aleatorio (recomendado para torneos sin ELO)
- Opción B: Por ELO (serpentina o segmentado)

**Paso 2**: Formar parejas iniciales
- Emparejar jugadores de 2 en 2 secuencialmente
- Si hay jugadores impares, aplicar BYE al último

**Paso 3**: Crear enfrentamientos
- Emparejar parejas consecutivas: Pareja1 vs Pareja2, Pareja3 vs Pareja4, etc.

#### 4.2 RONDAS 2-9 - Emparejamiento por Sistema Suizo

**Paso 1**: Actualizar Clasificación
- Calcular puntos, WR2, EFF y demás índices
- Ordenar jugadores descendentemente por rendimiento total

**Paso 2**: División en Pirámides
- **Pirámide Principal (PP)**: Primera mitad de jugadores
- **Pirámide Secundaria (PS)**: Segunda mitad de jugadores

**Paso 3**: Subdivisión de Pirámides
Para cada pirámide:
- **Lado A**: Primera mitad (mejores de la pirámide)
- **Lado B**: Segunda mitad (menores de la pirámide)

**Paso 4**: Cálculo de Parejas por Lado
```
parejas_lado_A = floor(jugadores_lado_A / 2)
Si resultado es decimal, restar 0.5
```

**Paso 5**: Validación de Restricciones

Para cada posible emparejamiento, verificar:

**Restricción de Compañeros**:
- No fueron compañeros en las últimas X rondas (según categoría)
- X = 1 (Compacta), 2 (Estándar), 3 (Internacional)

**Restricción de Rivales**:
- No fueron rivales en las últimas Y rondas (según categoría)
- Y = 2 (Compacta), 3 (Estándar), 4 (Internacional)

**Paso 6**: Algoritmo de Emparejamiento

```
PARA cada lado de cada pirámide:
  1. Ordenar jugadores por posición
  2. Intentar emparejar por método PAR-IMPAR:
     - Jugador 1 con Jugador 2
     - Jugador 3 con Jugador 4
     - etc.
  3. SI algún emparejamiento viola restricciones:
     - Buscar siguiente jugador válido
     - Intercambiar posiciones si es necesario
  4. SI no hay solución válida:
     - Aplicar PROTOCOLO DE CONTINGENCIA
```

**Paso 7**: Manejo de Mesas Impares

Cuando el número de mesas es impar:
1. La última pareja del Lado A de PP queda pendiente
2. Esta pareja se desempareja
3. Sus integrantes lideran el Lado A de PS
4. Se reajustan los emparejamientos en PS

**Paso 8**: Creación de Enfrentamientos

```
Mesa = Pareja_Lado_A vs Pareja_Lado_B
```
- PP Lado A vs PP Lado B (mesas superiores)
- PS Lado A vs PS Lado B (mesas inferiores)

**Paso 9**: Numeración de Mesas
- Mesa 1: Líderes absolutos del torneo
- Numeración ascendente según posición en clasificación

### 5. PROTOCOLO DE CONTINGENCIA

Cuando no se pueden cumplir todas las restricciones:

#### Nivel 1 - Relajación Menor
- Reducir ventana de compañeros en 1 ronda
- Mantener ventana de rivales

#### Nivel 2 - Transfusión entre Pirámides
- Intercambiar últimos 2 jugadores de PP con primeros 2 de PS
- Reintenter emparejamiento

#### Nivel 3 - Relajación Mayor
- Reducir ambas ventanas en 1 ronda
- Priorizar no-repetición de rivales sobre compañeros

### 6. CASOS ESPECIALES

#### 6.1 BYE (Descanso)
- Se asigna cuando hay número impar de jugadores
- Otorga victoria automática con puntaje promedio
- Se asigna al jugador con peor clasificación que no haya tenido BYE

#### 6.2 Abandono Durante el Torneo
- El jugador se marca como inactivo
- Su compañero recibe BYE si no hay reemplazo
- Se recalculan emparejamientos si es necesario

#### 6.3 Llegadas Tardías
- Solo permitidas en Ronda 1
- Se insertan al final de la lista inicial

### 7. VALIDACIÓN Y AUDITORÍA

El algoritmo debe generar para cada ronda:

1. **Registro de Decisiones**
   - Método de división utilizado
   - Restricciones aplicadas
   - Contingencias activadas

2. **Métricas de Calidad**
   - Porcentaje de emparejamientos óptimos
   - Número de contingencias aplicadas
   - Distribución de enfrentamientos por nivel

3. **Trazabilidad**
   - Justificación de cada emparejamiento
   - Alternativas consideradas
   - Razones de rechazo de alternativas

### 8. CRITERIOS DE ÉXITO

El algoritmo se considera exitoso si:

1. **Integridad Competitiva**: >90% de emparejamientos respetan el orden de clasificación
2. **Cumplimiento de Restricciones**: >85% sin activar contingencias
3. **Escalabilidad**: Tiempo de cálculo <5 segundos para 100 jugadores
4. **Equidad**: Ningún jugador repite compañero/rival fuera de ventanas más de 1 vez

### 9. EJEMPLO PRÁCTICO - RONDA 2 CON 28 JUGADORES

**Clasificación tras Ronda 1** (ordenada por índices):
```
1-14: A1,A2,C1,C2,M1,M2,G1,G2,I1,I2,E1,E2,K1,K2 (PP)
15-28: L1,L2,F1,F2,J1,J2,H1,H2,N1,N2,D1,D2,B1,B2 (PS)
```

**División PP**:
- Lado A: A1,A2,C1,C2,M1,M2,G1 (7 jugadores → 3 parejas)
- Lado B: G2,I1,I2,E1,E2,K1,K2 (7 jugadores → 3 parejas + 1)

**Emparejamientos PP**:
- A1-C1, A2-C2, M1-G1 (Lado A)
- M2-I1, G2-I2, E1-K1 (Lado B)
- Sobrantes: E2, K2

**División PS** (con E2, K2 liderando):
- Lado A: E2,K2,L1,L2,F1,F2,J1,J2
- Lado B: H1,H2,N1,N2,D1,D2,B1,B2

**Resultado Final Ronda 2**:
```
Mesa 1: A1-C1 vs M2-I1
Mesa 2: A2-C2 vs G2-I2
Mesa 3: M1-G1 vs E1-K1
Mesa 4: E2-L1 vs H1-N1
Mesa 5: K2-L2 vs H2-N2
Mesa 6: F1-J1 vs D1-B1
Mesa 7: F2-J2 vs D2-B2
```

### 10. CONSIDERACIONES FINALES

Este algoritmo ha sido diseñado para:
- Mantener la esencia competitiva del sistema suizo
- Adaptarse a diferentes tamaños de torneo
- Proporcionar transparencia total para auditorías
- Minimizar intervenciones manuales
- Garantizar experiencia justa para todos los participantes

La implementación debe incluir logs detallados de cada decisión para facilitar la revisión por parte de las autoridades del torneo y garantizar la confianza de los participantes en el sistema.
