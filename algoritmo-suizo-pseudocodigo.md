# Algoritmo Sistema Suizo Dominó - Pseudocódigo Técnico

```
ALGORITMO EmparejamientoSuizoDomino

CONSTANTES:
    VENTANA_COMPACTA = 3
    VENTANA_ESTANDAR = 4
    VENTANA_INTERNACIONAL = 5
    MAX_INTENTOS_BUSQUEDA = 10
    MAX_DISTANCIA_BUSQUEDA = 5

ESTRUCTURAS:
    Jugador {
        id: string
        ranking: integer
        puntos: float
        historial_companeros: Lista<{jugador_id, ronda}>
        historial_rivales: Lista<{jugador_id, ronda}>
        estado: enum(ACTIVO, BYE, RETIRADO)
        mesa_actual: integer
    }
    
    Mesa {
        numero: integer
        pareja_a: {jugador1: Jugador, jugador2: Jugador}
        pareja_b: {jugador1: Jugador, jugador2: Jugador}
    }
    
    ResultadoEmparejamiento {
        mesas: Lista<Mesa>
        jugadores_bye: Lista<Jugador>
        contingencias_aplicadas: Lista<{nivel, descripcion}>
        calidad_emparejamiento: float
    }

FUNCIÓN PRINCIPAL GenerarEmparejamientoRonda(
    jugadores_ordenados: Lista<Jugador>,
    ronda_actual: integer,
    total_rondas: integer
) -> ResultadoEmparejamiento:

    // Paso 1: Validación inicial
    jugadores_activos = FILTRAR(jugadores_ordenados, j => j.estado == ACTIVO)
    categoria = DeterminarCategoria(TAMAÑO(jugadores_activos))
    ventana_restriccion = ObtenerVentanaRestriccion(categoria, ronda_actual)
    
    // Paso 2: División en pirámides
    punto_division = TAMAÑO(jugadores_activos) / 2
    piramide_principal = jugadores_activos[0:punto_division]
    piramide_secundaria = jugadores_activos[punto_division:]
    
    // Paso 3: Manejo de mesas impares
    SI (TAMAÑO(jugadores_activos) % 4 != 0):
        sobrantes = ManejarMesasImpares(piramide_principal, piramide_secundaria)
    
    // Paso 4: Emparejamiento por pirámide
    mesas_pp = EmparejarPiramide(piramide_principal, ventana_restriccion, ronda_actual)
    mesas_ps = EmparejarPiramide(piramide_secundaria, ventana_restriccion, ronda_actual)
    
    // Paso 5: Consolidación y numeración
    todas_mesas = CONCATENAR(mesas_pp, mesas_ps)
    todas_mesas = NumerarMesas(todas_mesas)
    
    // Paso 6: Validación final
    resultado = ValidarYOptimizar(todas_mesas, jugadores_activos)
    
    RETORNAR resultado

FIN FUNCIÓN

FUNCIÓN EmparejarPiramide(
    jugadores: Lista<Jugador>,
    ventana: integer,
    ronda: integer
) -> Lista<Mesa>:

    // División en lados
    punto_medio = TAMAÑO(jugadores) / 2
    lado_a = jugadores[0:PISO(punto_medio)]
    lado_b = jugadores[PISO(punto_medio):]
    
    // Calcular parejas necesarias por lado
    parejas_lado_a = PISO(TAMAÑO(lado_a) / 2)
    parejas_lado_b = TECHO(TAMAÑO(lado_b) / 2)
    
    // Formar parejas en cada lado
    parejas_a = FormarParejas(lado_a, ventana, ronda)
    parejas_b = FormarParejas(lado_b, ventana, ronda)
    
    // Crear enfrentamientos
    mesas = CrearEnfrentamientos(parejas_a, parejas_b, ventana, ronda)
    
    RETORNAR mesas

FIN FUNCIÓN

FUNCIÓN FormarParejas(
    jugadores: Lista<Jugador>,
    ventana: integer,
    ronda: integer
) -> Lista<Pareja>:

    parejas = []
    jugadores_sin_pareja = COPIA(jugadores)
    intentos_fallidos = {}
    
    MIENTRAS (TAMAÑO(jugadores_sin_pareja) >= 2):
        jugador_actual = jugadores_sin_pareja[0]
        pareja_encontrada = FALSO
        intentos = 0
        
        // Buscar pareja válida
        PARA i DESDE 1 HASTA TAMAÑO(jugadores_sin_pareja):
            candidato = jugadores_sin_pareja[i]
            
            SI (EsCompaneroValido(jugador_actual, candidato, ventana, ronda)):
                parejas.AGREGAR({jugador_actual, candidato})
                jugadores_sin_pareja.ELIMINAR(jugador_actual)
                jugadores_sin_pareja.ELIMINAR(candidato)
                pareja_encontrada = VERDADERO
                ROMPER
            FIN SI
            
            intentos++
            SI (intentos > MAX_INTENTOS_BUSQUEDA):
                ROMPER
            FIN SI
        FIN PARA
        
        // Si no encuentra pareja, aplicar contingencia
        SI (NO pareja_encontrada):
            resultado_contingencia = AplicarContingencia(
                jugador_actual, 
                jugadores_sin_pareja,
                parejas,
                ventana,
                ronda
            )
            SI (resultado_contingencia.exito):
                parejas = resultado_contingencia.parejas_actualizadas
                jugadores_sin_pareja = resultado_contingencia.jugadores_restantes
            SINO:
                // Caso extremo: marcar para BYE
                jugador_actual.estado = BYE
                jugadores_sin_pareja.ELIMINAR(jugador_actual)
            FIN SI
        FIN SI
    FIN MIENTRAS
    
    RETORNAR parejas

FIN FUNCIÓN

FUNCIÓN EsCompaneroValido(
    jugador1: Jugador,
    jugador2: Jugador,
    ventana: integer,
    ronda: integer
) -> boolean:

    // Verificar historial de compañeros
    PARA historial EN jugador1.historial_companeros:
        SI (historial.jugador_id == jugador2.id):
            SI (ronda - historial.ronda < ventana):
                RETORNAR FALSO
            FIN SI
        FIN SI
    FIN PARA
    
    // Verificar que no se excedan enfrentamientos mutuos
    enfrentamientos_mutuos = ContarEnfrentamientosMutuos(jugador1, jugador2)
    SI (enfrentamientos_mutuos > 2):
        RETORNAR FALSO
    FIN SI
    
    RETORNAR VERDADERO

FIN FUNCIÓN

FUNCIÓN CrearEnfrentamientos(
    parejas_a: Lista<Pareja>,
    parejas_b: Lista<Pareja>,
    ventana: integer,
    ronda: integer
) -> Lista<Mesa>:

    mesas = []
    parejas_b_disponibles = COPIA(parejas_b)
    
    PARA pareja_a EN parejas_a:
        rival_encontrado = FALSO
        
        PARA pareja_b EN parejas_b_disponibles:
            SI (EsEnfrentamientoValido(pareja_a, pareja_b, ventana, ronda)):
                mesa = CrearMesa(pareja_a, pareja_b)
                mesas.AGREGAR(mesa)
                parejas_b_disponibles.ELIMINAR(pareja_b)
                rival_encontrado = VERDADERO
                ROMPER
            FIN SI
        FIN PARA
        
        SI (NO rival_encontrado):
            // Aplicar contingencia de enfrentamiento
            mesa = BuscarEnfrentamientoContingencia(
                pareja_a, 
                parejas_b_disponibles,
                ventana,
                ronda
            )
            SI (mesa != NULO):
                mesas.AGREGAR(mesa)
            FIN SI
        FIN SI
    FIN PARA
    
    RETORNAR mesas

FIN FUNCIÓN

FUNCIÓN EsEnfrentamientoValido(
    pareja_a: Pareja,
    pareja_b: Pareja,
    ventana: integer,
    ronda: integer
) -> boolean:

    // Verificar todos los cruces posibles
    jugadores_a = [pareja_a.jugador1, pareja_a.jugador2]
    jugadores_b = [pareja_b.jugador1, pareja_b.jugador2]
    
    PARA j_a EN jugadores_a:
        PARA j_b EN jugadores_b:
            SI (FueronRivalesRecientes(j_a, j_b, ventana, ronda)):
                RETORNAR FALSO
            FIN SI
        FIN PARA
    FIN PARA
    
    RETORNAR VERDADERO

FIN FUNCIÓN

FUNCIÓN AplicarContingencia(
    jugador: Jugador,
    jugadores_disponibles: Lista<Jugador>,
    parejas_formadas: Lista<Pareja>,
    ventana: integer,
    ronda: integer
) -> ResultadoContingencia:

    // NIVEL 1: Búsqueda extendida
    PARA distancia DESDE 1 HASTA MAX_DISTANCIA_BUSQUEDA:
        candidatos = ObtenerJugadoresADistancia(jugador, jugadores_disponibles, distancia)
        PARA candidato EN candidatos:
            SI (EsCompaneroValido(jugador, candidato, ventana, ronda)):
                RETORNAR ExitoContingencia(nivel=1, jugador, candidato)
            FIN SI
        FIN PARA
    FIN PARA
    
    // NIVEL 2: Intercambio de parejas
    PARA pareja EN parejas_formadas:
        SI (PuedeIntercambiar(jugador, pareja, ventana, ronda)):
            RETORNAR ExitoIntercambio(nivel=2, jugador, pareja)
        FIN SI
    FIN PARA
    
    // NIVEL 3: Transfusión entre pirámides
    SI (PermitidaTransfusion(ronda)):
        resultado = IntentarTransfusion(jugador, ventana, ronda)
        SI (resultado.exito):
            RETORNAR resultado
        FIN SI
    FIN SI
    
    // NIVEL 4: Flexibilización de ventanas
    ventana_reducida = ventana - 1
    SI (ventana_reducida >= 2):
        PARA candidato EN jugadores_disponibles:
            SI (EsCompaneroValido(jugador, candidato, ventana_reducida, ronda)):
                RETORNAR ExitoFlexibilizacion(nivel=4, jugador, candidato)
            FIN SI
        FIN PARA
    FIN SI
    
    RETORNAR FalloContingencia()

FIN FUNCIÓN

FUNCIÓN ManejarMesasImpares(
    piramide_principal: Lista<Jugador>,
    piramide_secundaria: Lista<Jugador>
) -> Lista<Jugador>:

    num_jugadores = TAMAÑO(piramide_principal) + TAMAÑO(piramide_secundaria)
    jugadores_sobrantes = num_jugadores % 4
    
    SI (jugadores_sobrantes == 0):
        RETORNAR []
    FIN SI
    
    // Transferir jugadores sobrantes de PP a PS
    sobrantes = []
    PARA i DESDE 1 HASTA jugadores_sobrantes:
        jugador = piramide_principal.ULTIMO()
        piramide_principal.ELIMINAR_ULTIMO()
        piramide_secundaria.INSERTAR_AL_INICIO(jugador)
        sobrantes.AGREGAR(jugador)
    FIN PARA
    
    RETORNAR sobrantes

FIN FUNCIÓN

FUNCIÓN ValidarYOptimizar(
    mesas: Lista<Mesa>,
    jugadores_activos: Lista<Jugador>
) -> ResultadoEmparejamiento:

    resultado = NuevoResultadoEmparejamiento()
    
    // Verificar cobertura completa
    jugadores_asignados = ExtraerJugadoresAsignados(mesas)
    jugadores_sin_mesa = DIFERENCIA(jugadores_activos, jugadores_asignados)
    
    SI (TAMAÑO(jugadores_sin_mesa) > 0):
        // Asignar BYE a jugadores sin mesa
        PARA jugador EN jugadores_sin_mesa:
            jugador.estado = BYE
            resultado.jugadores_bye.AGREGAR(jugador)
        FIN PARA
    FIN SI
    
    // Calcular calidad del emparejamiento
    total_emparejamientos = TAMAÑO(mesas) * 4
    emparejamientos_optimos = ContarEmparejamientosOptimos(mesas)
    resultado.calidad_emparejamiento = emparejamientos_optimos / total_emparejamientos
    
    resultado.mesas = mesas
    
    RETORNAR resultado

FIN FUNCIÓN

FUNCIÓN DeterminarCategoria(num_jugadores: integer) -> string:
    SI (num_jugadores <= 36):
        RETORNAR "COMPACTA"
    SINO SI (num_jugadores <= 76):
        RETORNAR "ESTANDAR"
    SINO:
        RETORNAR "INTERNACIONAL"
    FIN SI
FIN FUNCIÓN

FUNCIÓN ObtenerVentanaRestriccion(
    categoria: string,
    ronda_actual: integer
) -> integer:

    ventana_base = CASO categoria:
        "COMPACTA": VENTANA_COMPACTA
        "ESTANDAR": VENTANA_ESTANDAR
        "INTERNACIONAL": VENTANA_INTERNACIONAL
    FIN CASO
    
    // Permitir flexibilización en rondas finales
    SI (categoria == "COMPACTA" Y ronda_actual >= 6):
        ventana_base = ventana_base - 1
    SINO SI (categoria == "ESTANDAR" Y ronda_actual >= 7):
        ventana_base = ventana_base - 1
    SINO SI (categoria == "INTERNACIONAL" Y ronda_actual >= 8):
        ventana_base = ventana_base - 1
    FIN SI
    
    RETORNAR MAX(ventana_base, 2)

FIN FUNCIÓN

// Funciones auxiliares adicionales

FUNCIÓN RegistrarEmparejamiento(mesa: Mesa, ronda: integer):
    // Actualizar historiales de compañeros
    mesa.pareja_a.jugador1.historial_companeros.AGREGAR({
        jugador_id: mesa.pareja_a.jugador2.id,
        ronda: ronda
    })
    mesa.pareja_a.jugador2.historial_companeros.AGREGAR({
        jugador_id: mesa.pareja_a.jugador1.id,
        ronda: ronda
    })
    
    // Similar para pareja_b...
    
    // Actualizar historiales de rivales
    jugadores_a = [mesa.pareja_a.jugador1, mesa.pareja_a.jugador2]
    jugadores_b = [mesa.pareja_b.jugador1, mesa.pareja_b.jugador2]
    
    PARA j_a EN jugadores_a:
        PARA j_b EN jugadores_b:
            j_a.historial_rivales.AGREGAR({
                jugador_id: j_b.id,
                ronda: ronda
            })
            j_b.historial_rivales.AGREGAR({
                jugador_id: j_a.id,
                ronda: ronda
            })
        FIN PARA
    FIN PARA
FIN FUNCIÓN

FUNCIÓN GenerarReporteAuditoria(
    resultado: ResultadoEmparejamiento,
    ronda: integer
) -> string:

    reporte = "RONDA " + ronda + ":\n"
    reporte += "- Total jugadores: " + ContarJugadoresActivos() + "\n"
    reporte += "- Mesas generadas: " + TAMAÑO(resultado.mesas) + "\n"
    
    contingencias_por_nivel = AgruparContingenciasPorNivel(resultado.contingencias_aplicadas)
    PARA nivel DESDE 1 HASTA 4:
        reporte += "- Contingencias Nivel " + nivel + ": " + 
                   TAMAÑO(contingencias_por_nivel[nivel]) + " casos\n"
    FIN PARA
    
    reporte += "- Jugadores BYE: " + ListarJugadores(resultado.jugadores_bye) + "\n"
    reporte += "- Calidad del emparejamiento: " + 
               FORMATEAR_PORCENTAJE(resultado.calidad_emparejamiento) + "\n"
    
    RETORNAR reporte

FIN FUNCIÓN
```
