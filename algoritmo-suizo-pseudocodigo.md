# Algoritmo Sistema Suizo Dominó - Pseudocódigo Técnico

## Estructuras de Datos

```
ESTRUCTURA Jugador {
    id: entero
    nombre: texto
    puntosAcumulados: decimal
    indiceWR: decimal
    indiceEFF: decimal
    posicionGeneral: entero
    historialCompaneros: lista<{jugadorId, ronda}>
    historialRivales: lista<{jugadorId, ronda}>
    activo: booleano
}

ESTRUCTURA Pareja {
    jugador1: Jugador
    jugador2: Jugador
    sumaPosiciones: entero
}

ESTRUCTURA Mesa {
    numero: entero
    pareja1: Pareja
    pareja2: Pareja
    ronda: entero
}

ESTRUCTURA ConfiguracionTorneo {
    categoria: enum {COMPACTA, ESTANDAR, INTERNACIONAL}
    ventanaCompaneros: entero
    ventanaRivales: entero
    totalJugadores: entero
    rondaActual: entero
    permitirBye: booleano
}
```

## Algoritmo Principal

```
FUNCIÓN emparejarRonda(jugadores: lista<Jugador>, config: ConfiguracionTorneo): lista<Mesa>
    // Validar entrada
    SI jugadores.longitud < 4 ENTONCES
        LANZAR_ERROR "Mínimo 4 jugadores requeridos"
    FIN_SI
    
    // Filtrar solo jugadores activos
    jugadoresActivos = FILTRAR(jugadores, j => j.activo)
    
    // Manejar número impar con BYE
    SI jugadoresActivos.longitud ES IMPAR ENTONCES
        jugadorBye = jugadoresActivos[ULTIMO]
        ASIGNAR_BYE(jugadorBye, config.rondaActual)
        jugadoresActivos.ELIMINAR(jugadorBye)
    FIN_SI
    
    // Primera ronda - emparejamiento aleatorio
    SI config.rondaActual == 1 ENTONCES
        RETORNAR emparejarPrimeraRonda(jugadoresActivos)
    FIN_SI
    
    // Rondas posteriores - sistema suizo
    piramides = formarPiramides(jugadoresActivos)
    parejas = formarParejas(piramides, config)
    mesas = formarMesas(parejas, config)
    
    RETORNAR mesas
FIN_FUNCIÓN
```

## Formación de Pirámides

```
FUNCIÓN formarPiramides(jugadores: lista<Jugador>): {PP: Piramide, PS: Piramide}
    // Ordenar por posición general (ya viene ordenado del sistema)
    totalJugadores = jugadores.longitud
    puntoCorte = totalJugadores / 2
    
    // Crear pirámides
    piramidePrincipal = jugadores[0:puntoCorte]
    piramideSecundaria = jugadores[puntoCorte:totalJugadores]
    
    // Dividir cada pirámide en lados
    PP = dividirEnLados(piramidePrincipal)
    PS = dividirEnLados(piramideSecundaria)
    
    RETORNAR {PP: PP, PS: PS}
FIN_FUNCIÓN

FUNCIÓN dividirEnLados(jugadores: lista<Jugador>): Piramide
    total = jugadores.longitud
    tamanoLadoA = PISO(total / 2)
    
    // Si resultado es impar, ajustar para parejas completas
    SI (tamanoLadoA / 2) TIENE_DECIMALES ENTONCES
        tamanoLadoA = tamanoLadoA - 1
    FIN_SI
    
    RETORNAR {
        ladoA: jugadores[0:tamanoLadoA],
        ladoB: jugadores[tamanoLadoA:total]
    }
FIN_FUNCIÓN
```

## Formación de Parejas con Validación

```
FUNCIÓN formarParejas(piramides: {PP, PS}, config: ConfiguracionTorneo): lista<Pareja>
    parejas = []
    jugadoresUsados = conjunto_vacio()
    
    // Procesar Pirámide Principal
    parejasPP = emparejarPiramide(piramides.PP, jugadoresUsados, config)
    parejas.AGREGAR_TODOS(parejasPP)
    
    // Manejar sobrantes de PP si hay mesas impares
    sobrantesPP = obtenerJugadoresSinPareja(piramides.PP, jugadoresUsados)
    
    // Procesar Pirámide Secundaria con sobrantes
    SI sobrantesPP.longitud > 0 ENTONCES
        // Insertar sobrantes al inicio de PS
        piramides.PS.ladoA.INSERTAR_AL_INICIO(sobrantesPP)
    FIN_SI
    
    parejasPS = emparejarPiramide(piramides.PS, jugadoresUsados, config)
    parejas.AGREGAR_TODOS(parejasPS)
    
    RETORNAR parejas
FIN_FUNCIÓN

FUNCIÓN emparejarPiramide(piramide: Piramide, jugadoresUsados: conjunto, config: ConfiguracionTorneo): lista<Pareja>
    parejas = []
    
    // Emparejar Lado A
    parejas.AGREGAR_TODOS(emparejarLado(piramide.ladoA, jugadoresUsados, config))
    
    // Emparejar Lado B
    parejas.AGREGAR_TODOS(emparejarLado(piramide.ladoB, jugadoresUsados, config))
    
    RETORNAR parejas
FIN_FUNCIÓN
```

## Algoritmo de Emparejamiento con Restricciones

```
FUNCIÓN emparejarLado(jugadores: lista<Jugador>, jugadoresUsados: conjunto, config: ConfiguracionTorneo): lista<Pareja>
    parejas = []
    disponibles = FILTRAR(jugadores, j => NO jugadoresUsados.CONTIENE(j.id))
    
    MIENTRAS disponibles.longitud >= 2 HACER
        jugador1 = disponibles[0]
        parejaBuscada = buscarParejaValida(jugador1, disponibles[1:], config)
        
        SI parejaBuscada != NULO ENTONCES
            parejas.AGREGAR(crearPareja(jugador1, parejaBuscada))
            jugadoresUsados.AGREGAR(jugador1.id)
            jugadoresUsados.AGREGAR(parejaBuscada.id)
            disponibles.ELIMINAR(jugador1)
            disponibles.ELIMINAR(parejaBuscada)
        SINO
            // Activar sistema de contingencia
            parejaBuscada = aplicarContingencia(jugador1, disponibles, config)
            SI parejaBuscada != NULO ENTONCES
                parejas.AGREGAR(crearPareja(jugador1, parejaBuscada))
                jugadoresUsados.AGREGAR(jugador1.id)
                jugadoresUsados.AGREGAR(parejaBuscada.id)
                disponibles.ELIMINAR(jugador1)
                disponibles.ELIMINAR(parejaBuscada)
            SINO
                // Marcar para manejo especial
                disponibles.ELIMINAR(jugador1)
            FIN_SI
        FIN_SI
    FIN_MIENTRAS
    
    RETORNAR parejas
FIN_FUNCIÓN
```

## Búsqueda de Pareja Válida

```
FUNCIÓN buscarParejaValida(jugador: Jugador, candidatos: lista<Jugador>, config: ConfiguracionTorneo): Jugador
    PARA CADA candidato EN candidatos HACER
        SI validarCompanero(jugador, candidato, config) ENTONCES
            RETORNAR candidato
        FIN_SI
    FIN_PARA
    
    RETORNAR NULO
FIN_FUNCIÓN

FUNCIÓN validarCompanero(jugador1: Jugador, jugador2: Jugador, config: ConfiguracionTorneo): booleano
    // Verificar restricción de compañeros
    PARA CADA historial EN jugador1.historialCompaneros HACER
        SI historial.jugadorId == jugador2.id ENTONCES
            rondasTranscurridas = config.rondaActual - historial.ronda
            SI rondasTranscurridas <= config.ventanaCompaneros ENTONCES
                RETORNAR FALSO
            FIN_SI
        FIN_SI
    FIN_PARA
    
    RETORNAR VERDADERO
FIN_FUNCIÓN
```

## Formación de Mesas con Validación de Rivales

```
FUNCIÓN formarMesas(parejas: lista<Pareja>, config: ConfiguracionTorneo): lista<Mesa>
    mesas = []
    parejasUsadas = conjunto_vacio()
    numeroMesa = 1
    
    // Ordenar parejas por suma de posiciones
    parejas.ORDENAR_POR(p => p.sumaPosiciones)
    
    MIENTRAS hayParejasDisponibles(parejas, parejasUsadas) HACER
        pareja1 = siguienteParejaSinUsar(parejas, parejasUsadas)
        SI pareja1 == NULO ENTONCES SALIR
        
        pareja2 = buscarRivalValido(pareja1, parejas, parejasUsadas, config)
        
        SI pareja2 != NULO ENTONCES
            mesa = crearMesa(numeroMesa, pareja1, pareja2, config.rondaActual)
            mesas.AGREGAR(mesa)
            parejasUsadas.AGREGAR(pareja1)
            parejasUsadas.AGREGAR(pareja2)
            numeroMesa++
        SINO
            // Sistema de contingencia para rivales
            pareja2 = aplicarContingenciaRivales(pareja1, parejas, parejasUsadas, config)
            SI pareja2 != NULO ENTONCES
                mesa = crearMesa(numeroMesa, pareja1, pareja2, config.rondaActual)
                mesas.AGREGAR(mesa)
                parejasUsadas.AGREGAR(pareja1)
                parejasUsadas.AGREGAR(pareja2)
                numeroMesa++
            FIN_SI
        FIN_SI
    FIN_MIENTRAS
    
    RETORNAR mesas
FIN_FUNCIÓN
```

## Validación de Rivales

```
FUNCIÓN buscarRivalValido(pareja1: Pareja, parejas: lista<Pareja>, usadas: conjunto, config: ConfiguracionTorneo): Pareja
    candidatos = FILTRAR(parejas, p => NO usadas.CONTIENE(p) Y p != pareja1)
    
    PARA CADA candidato EN candidatos HACER
        SI validarEnfrentamiento(pareja1, candidato, config) ENTONCES
            RETORNAR candidato
        FIN_SI
    FIN_PARA
    
    RETORNAR NULO
FIN_FUNCIÓN

FUNCIÓN validarEnfrentamiento(pareja1: Pareja, pareja2: Pareja, config: ConfiguracionTorneo): booleano
    jugadores1 = [pareja1.jugador1, pareja1.jugador2]
    jugadores2 = [pareja2.jugador1, pareja2.jugador2]
    
    // Verificar cada combinación de rivales
    PARA CADA j1 EN jugadores1 HACER
        PARA CADA j2 EN jugadores2 HACER
            SI NO validarRival(j1, j2, config) ENTONCES
                RETORNAR FALSO
            FIN_SI
        FIN_PARA
    FIN_PARA
    
    RETORNAR VERDADERO
FIN_FUNCIÓN

FUNCIÓN validarRival(jugador1: Jugador, jugador2: Jugador, config: ConfiguracionTorneo): booleano
    PARA CADA historial EN jugador1.historialRivales HACER
        SI historial.jugadorId == jugador2.id ENTONCES
            rondasTranscurridas = config.rondaActual - historial.ronda
            SI rondasTranscurridas <= config.ventanaRivales ENTONCES
                RETORNAR FALSO
            FIN_SI
        FIN_SI
    FIN_PARA
    
    RETORNAR VERDADERO
FIN_FUNCIÓN
```

## Sistema de Contingencia

```
FUNCIÓN aplicarContingencia(jugador: Jugador, disponibles: lista<Jugador>, config: ConfiguracionTorneo): Jugador
    // Nivel 1: Buscar en todo el lado actual
    candidato = buscarEnTodoElLado(jugador, disponibles, config)
    SI candidato != NULO ENTONCES RETORNAR candidato
    
    // Nivel 2: Buscar en el otro lado de la misma pirámide
    candidato = buscarEnOtroLado(jugador, config)
    SI candidato != NULO ENTONCES RETORNAR candidato
    
    // Nivel 3: Transfusión entre pirámides
    candidato = buscarEnOtraPiramide(jugador, config)
    SI candidato != NULO ENTONCES RETORNAR candidato
    
    // Nivel 4: Relajar ventanas temporalmente
    configRelajada = relajarVentanas(config)
    candidato = buscarParejaValida(jugador, disponibles, configRelajada)
    SI candidato != NULO ENTONCES
        REGISTRAR_EXCEPCION("Ventanas relajadas para jugador " + jugador.id)
        RETORNAR candidato
    FIN_SI
    
    RETORNAR NULO
FIN_FUNCIÓN

FUNCIÓN relajarVentanas(config: ConfiguracionTorneo): ConfiguracionTorneo
    nuevaConfig = CLONAR(config)
    nuevaConfig.ventanaCompaneros = MAX(0, config.ventanaCompaneros - 1)
    nuevaConfig.ventanaRivales = MAX(0, config.ventanaRivales - 1)
    RETORNAR nuevaConfig
FIN_FUNCIÓN
```

## Configuración por Categoría

```
FUNCIÓN obtenerConfiguracion(totalJugadores: entero): ConfiguracionTorneo
    config = nuevo ConfiguracionTorneo()
    config.totalJugadores = totalJugadores
    
    SI totalJugadores <= 36 ENTONCES
        config.categoria = COMPACTA
        config.ventanaCompaneros = 1
        config.ventanaRivales = 2
    SINO SI totalJugadores <= 76 ENTONCES
        config.categoria = ESTANDAR
        config.ventanaCompaneros = 2
        config.ventanaRivales = 3
    SINO
        config.categoria = INTERNACIONAL
        config.ventanaCompaneros = 3
        config.ventanaRivales = 4
    FIN_SI
    
    config.permitirBye = VERDADERO
    RETORNAR config
FIN_FUNCIÓN
```

## Actualización de Historiales

```
FUNCIÓN actualizarHistoriales(mesas: lista<Mesa>, ronda: entero)
    PARA CADA mesa EN mesas HACER
        jugadores = [
            mesa.pareja1.jugador1,
            mesa.pareja1.jugador2,
            mesa.pareja2.jugador1,
            mesa.pareja2.jugador2
        ]
        
        // Actualizar compañeros
        actualizarCompanero(mesa.pareja1.jugador1, mesa.pareja1.jugador2, ronda)
        actualizarCompanero(mesa.pareja2.jugador1, mesa.pareja2.jugador2, ronda)
        
        // Actualizar rivales
        PARA CADA j1 EN [mesa.pareja1.jugador1, mesa.pareja1.jugador2] HACER
            PARA CADA j2 EN [mesa.pareja2.jugador1, mesa.pareja2.jugador2] HACER
                actualizarRival(j1, j2, ronda)
            FIN_PARA
        FIN_PARA
    FIN_PARA
FIN_FUNCIÓN

FUNCIÓN actualizarCompanero(jugador1: Jugador, jugador2: Jugador, ronda: entero)
    jugador1.historialCompaneros.AGREGAR({jugadorId: jugador2.id, ronda: ronda})
    jugador2.historialCompaneros.AGREGAR({jugadorId: jugador1.id, ronda: ronda})
FIN_FUNCIÓN

FUNCIÓN actualizarRival(jugador1: Jugador, jugador2: Jugador, ronda: entero)
    jugador1.historialRivales.AGREGAR({jugadorId: jugador2.id, ronda: ronda})
    jugador2.historialRivales.AGREGAR({jugadorId: jugador1.id, ronda: ronda})
FIN_FUNCIÓN
```

## Manejo de Casos Especiales

```
FUNCIÓN manejarAbandono(jugador: Jugador, config: ConfiguracionTorneo)
    jugador.activo = FALSO
    
    // Si tiene pareja asignada, liberar a la pareja
    SI jugador.parejaActual != NULO ENTONCES
        compañero = jugador.parejaActual
        compañero.parejaActual = NULO
        
        // Buscar nueva pareja o asignar BYE
        SI hayJugadoresDisponibles() ENTONCES
            nuevaPareja = buscarParejaEmergencia(compañero, config)
            SI nuevaPareja != NULO ENTONCES
                emparejar(compañero, nuevaPareja)
            SINO
                ASIGNAR_BYE(compañero, config.rondaActual)
            FIN_SI
        SINO
            ASIGNAR_BYE(compañero, config.rondaActual)
        FIN_SI
    FIN_SI
FIN_FUNCIÓN

FUNCIÓN ASIGNAR_BYE(jugador: Jugador, ronda: entero)
    jugador.puntosAcumulados += PUNTOS_BYE
    jugador.historialByes.AGREGAR(ronda)
    REGISTRAR("Jugador " + jugador.id + " recibe BYE en ronda " + ronda)
FIN_FUNCIÓN
```

## Validación y Métricas

```
FUNCIÓN validarEmparejamiento(mesas: lista<Mesa>, config: ConfiguracionTorneo): ReporteValidacion
    reporte = nuevo ReporteValidacion()
    reporte.totalMesas = mesas.longitud
    reporte.violacionesCompaneros = 0
    reporte.violacionesRivales = 0
    reporte.contingenciasAplicadas = 0
    
    PARA CADA mesa EN mesas HACER
        // Validar compañeros
        SI NO validarCompanero(mesa.pareja1.jugador1, mesa.pareja1.jugador2, config) ENTONCES
            reporte.violacionesCompaneros++
        FIN_SI
        SI NO validarCompanero(mesa.pareja2.jugador1, mesa.pareja2.jugador2, config) ENTONCES
            reporte.violacionesCompaneros++
        FIN_SI
        
        // Validar enfrentamiento
        SI NO validarEnfrentamiento(mesa.pareja1, mesa.pareja2, config) ENTONCES
            reporte.violacionesRivales++
        FIN_SI
    FIN_PARA
    
    reporte.porcentajeExito = ((reporte.totalMesas * 4) - reporte.violacionesCompaneros - reporte.violacionesRivales) / (reporte.totalMesas * 4) * 100
    
    RETORNAR reporte
FIN_FUNCIÓN
```

## Función Principal del Sistema

```
FUNCIÓN ejecutarRonda(torneo: Torneo): ResultadoRonda
    // Obtener configuración según categoría
    config = obtenerConfiguracion(torneo.jugadores.longitud)
    config.rondaActual = torneo.rondaActual
    
    // Obtener jugadores ordenados por clasificación
    jugadoresOrdenados = torneo.obtenerClasificacionActual()
    
    // Realizar emparejamiento
    mesas = emparejarRonda(jugadoresOrdenados, config)
    
    // Validar resultado
    reporte = validarEmparejamiento(mesas, config)
    
    SI reporte.porcentajeExito < 85 ENTONCES
        ADVERTENCIA("Calidad de emparejamiento por debajo del umbral: " + reporte.porcentajeExito + "%")
    FIN_SI
    
    // Actualizar historiales
    actualizarHistoriales(mesas, config.rondaActual)
    
    // Preparar resultado
    resultado = nuevo ResultadoRonda()
    resultado.mesas = mesas
    resultado.reporte = reporte
    resultado.ronda = config.rondaActual
    
    RETORNAR resultado
FIN_FUNCIÓN
```
