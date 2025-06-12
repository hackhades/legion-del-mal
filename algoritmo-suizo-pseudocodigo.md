# Pseudocódigo del Sistema Suizo para Torneos de Dominó

```pseudocode
ESTRUCTURA Atleta:
    id: entero
    nombre: texto
    elo: entero
    puntos_totales: decimal
    victorias: entero
    derrotas: entero
    partidas_jugadas: entero
    win_rate: decimal
    efectividad: decimal
    indice_desempeno: decimal
    historial_companeros: lista<{atleta_id, ronda}>
    historial_rivales: lista<{atleta_id, ronda}>
    tiene_bye: booleano

ESTRUCTURA Pareja:
    atleta1: Atleta
    atleta2: Atleta
    id_temporal: texto

ESTRUCTURA Mesa:
    numero: entero
    pareja1: Pareja
    pareja2: Pareja
    resultado: {ganador: Pareja, puntos_p1: entero, puntos_p2: entero}

ESTRUCTURA ConfiguracionTorneo:
    total_participantes: entero
    categoria: enum {COMPACTA, ESTANDAR, INTERNACIONAL}
    ventana_companeros: entero
    ventana_rivales: entero
    ronda_actual: entero
    total_rondas: entero
    ordenamiento_inicial: enum {ELO, ALEATORIO}

// FUNCIONES PRINCIPALES

FUNCION inicializar_torneo(atletas: lista<Atleta>, config: ConfiguracionTorneo):
    // Determinar categoría y ventanas según número de participantes
    SI config.total_participantes <= 36:
        config.categoria = COMPACTA
        config.ventana_companeros = 1
        config.ventana_rivales = 2
    SINO SI config.total_participantes <= 76:
        config.categoria = ESTANDAR
        config.ventana_companeros = 2
        config.ventana_rivales = 3
    SINO:
        config.categoria = INTERNACIONAL
        config.ventana_companeros = 3
        config.ventana_rivales = 4
    
    RETORNAR config

FUNCION generar_emparejamiento_ronda(atletas: lista<Atleta>, config: ConfiguracionTorneo):
    SI config.ronda_actual == 1:
        RETORNAR emparejamiento_primera_ronda(atletas, config)
    SINO:
        RETORNAR emparejamiento_ronda_suiza(atletas, config)

// PRIMERA RONDA

FUNCION emparejamiento_primera_ronda(atletas: lista<Atleta>, config: ConfiguracionTorneo):
    atletas_ordenados = lista<Atleta>
    
    SI config.ordenamiento_inicial == ELO:
        atletas_ordenados = ordenar_por_elo(atletas)
    SINO:
        atletas_ordenados = mezclar_aleatoriamente(atletas)
    
    // Manejar número impar
    SI longitud(atletas_ordenados) MOD 2 != 0:
        ultimo_atleta = atletas_ordenados[ultimo]
        ultimo_atleta.tiene_bye = VERDADERO
        atletas_ordenados.remover(ultimo_atleta)
    
    parejas = lista<Pareja>
    PARA i = 0 HASTA longitud(atletas_ordenados) - 1 PASO 2:
        pareja = crear_pareja(atletas_ordenados[i], atletas_ordenados[i+1])
        parejas.agregar(pareja)
        actualizar_historial_companeros(atletas_ordenados[i], atletas_ordenados[i+1], 1)
    
    mesas = crear_mesas_desde_parejas(parejas, 1)
    RETORNAR mesas

FUNCION crear_mesas_desde_parejas(parejas: lista<Pareja>, ronda: entero):
    mesas = lista<Mesa>
    
    PARA i = 0 HASTA longitud(parejas) - 1 PASO 2:
        SI i + 1 < longitud(parejas):
            mesa = Mesa{
                numero: (i / 2) + 1,
                pareja1: parejas[i],
                pareja2: parejas[i + 1]
            }
            mesas.agregar(mesa)
            actualizar_historial_rivales(parejas[i], parejas[i + 1], ronda)
    
    RETORNAR mesas

// RONDAS SUBSIGUIENTES - SISTEMA SUIZO

FUNCION emparejamiento_ronda_suiza(atletas: lista<Atleta>, config: ConfiguracionTorneo):
    // Actualizar índices de desempeño
    PARA cada atleta EN atletas:
        atleta.win_rate = atleta.victorias / atleta.partidas_jugadas
        atleta.indice_desempeno = calcular_indice_desempeno(atleta)
    
    // Ordenar por desempeño GENERAL ACUMULADO (no solo última ronda)
    atletas_ordenados = ordenar_por_indice_desempeno(atletas)
    
    // Dividir en dos pirámides según clasificación general
    punto_division = longitud(atletas_ordenados) / 2
    piramide_superior = atletas_ordenados[0:punto_division]
    piramide_inferior = atletas_ordenados[punto_division:]
    
    // Manejar casos especiales de número impar de mesas
    SI longitud(atletas_ordenados) MOD 4 != 0:
        ajustar_para_mesas_impares(piramide_superior, piramide_inferior)
    
    // Generar emparejamientos para cada pirámide
    parejas_piramide_superior = generar_parejas_con_validacion(piramide_superior, config)
    parejas_piramide_inferior = generar_parejas_con_validacion(piramide_inferior, config)
    
    // Combinar y crear mesas
    todas_parejas = parejas_piramide_superior + parejas_piramide_inferior
    mesas = crear_mesas_con_validacion(todas_parejas, config)
    
    RETORNAR mesas

FUNCION generar_parejas_con_validacion(atletas: lista<Atleta>, config: ConfiguracionTorneo):
    parejas = lista<Pareja>
    atletas_disponibles = copia(atletas)
    intentos = 0
    max_intentos = 100
    
    MIENTRAS longitud(atletas_disponibles) >= 2 Y intentos < max_intentos:
        // Dividir en lados A y B
        punto_division = longitud(atletas_disponibles) / 2
        lado_a = atletas_disponibles[0:punto_division]
        lado_b = atletas_disponibles[punto_division:]
        
        // Intentar emparejar par-impar
        pareja_valida = FALSO
        
        PARA i EN rango(0, longitud(lado_a), 2):
            SI i + 1 < longitud(lado_a):
                atleta1 = lado_a[i]
                atleta2 = lado_a[i + 1]
                
                SI validar_companeros(atleta1, atleta2, config):
                    pareja = crear_pareja(atleta1, atleta2)
                    parejas.agregar(pareja)
                    atletas_disponibles.remover(atleta1)
                    atletas_disponibles.remover(atleta2)
                    actualizar_historial_companeros(atleta1, atleta2, config.ronda_actual)
                    pareja_valida = VERDADERO
                SINO:
                    // Buscar alternativa
                    alternativa = buscar_companero_alternativo(atleta1, lado_a, config)
                    SI alternativa != NULO:
                        pareja = crear_pareja(atleta1, alternativa)
                        parejas.agregar(pareja)
                        atletas_disponibles.remover(atleta1)
                        atletas_disponibles.remover(alternativa)
                        actualizar_historial_companeros(atleta1, alternativa, config.ronda_actual)
                        pareja_valida = VERDADERO
        
        SI NO pareja_valida:
            // Aplicar mecanismos de contingencia
            aplicar_contingencia(atletas_disponibles, parejas, config)
        
        intentos++
    
    RETORNAR parejas

FUNCION crear_mesas_con_validacion(parejas: lista<Pareja>, config: ConfiguracionTorneo):
    mesas = lista<Mesa>
    parejas_disponibles = copia(parejas)
    numero_mesa = 1
    
    // Dividir parejas en dos grupos por desempeño
    punto_division = longitud(parejas_disponibles) / 2
    grupo_superior = parejas_disponibles[0:punto_division]
    grupo_inferior = parejas_disponibles[punto_division:]
    
    PARA pareja_sup EN grupo_superior:
        mesa_creada = FALSO
        intentos = 0
        
        MIENTRAS NO mesa_creada Y intentos < longitud(grupo_inferior):
            pareja_inf = grupo_inferior[intentos]
            
            SI validar_rivales(pareja_sup, pareja_inf, config):
                mesa = Mesa{
                    numero: numero_mesa,
                    pareja1: pareja_sup,
                    pareja2: pareja_inf
                }
                mesas.agregar(mesa)
                grupo_inferior.remover(pareja_inf)
                actualizar_historial_rivales(pareja_sup, pareja_inf, config.ronda_actual)
                mesa_creada = VERDADERO
                numero_mesa++
            
            intentos++
        
        SI NO mesa_creada:
            // Aplicar relajación de restricciones
            pareja_inf = aplicar_relajacion_rivales(pareja_sup, grupo_inferior, config)
            SI pareja_inf != NULO:
                mesa = Mesa{
                    numero: numero_mesa,
                    pareja1: pareja_sup,
                    pareja2: pareja_inf
                }
                mesas.agregar(mesa)
                grupo_inferior.remover(pareja_inf)
                numero_mesa++
    
    RETORNAR mesas

// FUNCIONES DE VALIDACIÓN

FUNCION validar_companeros(atleta1: Atleta, atleta2: Atleta, config: ConfiguracionTorneo):
    PARA registro EN atleta1.historial_companeros:
        SI registro.atleta_id == atleta2.id:
            rondas_transcurridas = config.ronda_actual - registro.ronda
            SI rondas_transcurridas <= config.ventana_companeros:
                RETORNAR FALSO
    
    RETORNAR VERDADERO

FUNCION validar_rivales(pareja1: Pareja, pareja2: Pareja, config: ConfiguracionTorneo):
    atletas_pareja1 = [pareja1.atleta1, pareja1.atleta2]
    atletas_pareja2 = [pareja2.atleta1, pareja2.atleta2]
    
    PARA atleta1 EN atletas_pareja1:
        PARA atleta2 EN atletas_pareja2:
            PARA registro EN atleta1.historial_rivales:
                SI registro.atleta_id == atleta2.id:
                    rondas_transcurridas = config.ronda_actual - registro.ronda
                    SI rondas_transcurridas <= config.ventana_rivales:
                        RETORNAR FALSO
    
    RETORNAR VERDADERO

// FUNCIONES DE CONTINGENCIA

FUNCION aplicar_contingencia(atletas_disponibles: lista<Atleta>, parejas: lista<Pareja>, config: ConfiguracionTorneo):
    // Nivel 1: Relajación temporal
    config_relajada = copia(config)
    config_relajada.ventana_companeros = max(0, config.ventana_companeros - 1)
    
    // Reintentar con restricciones relajadas
    pareja = intentar_emparejar_con_config(atletas_disponibles[0], atletas_disponibles, config_relajada)
    
    SI pareja != NULO:
        parejas.agregar(pareja)
        RETORNAR VERDADERO
    
    // Nivel 2: Intercambio entre grupos
    SI permitir_intercambio_grupos():
        // Implementar lógica de intercambio
        RETORNAR aplicar_intercambio_grupos(atletas_disponibles, parejas, config)
    
    // Nivel 3: Emparejamiento forzado (último recurso)
    registrar_excepcion("Emparejamiento forzado en ronda " + config.ronda_actual)
    pareja_forzada = crear_pareja(atletas_disponibles[0], atletas_disponibles[1])
    parejas.agregar(pareja_forzada)
    RETORNAR VERDADERO

// FUNCIONES AUXILIARES

FUNCION calcular_indice_desempeno(atleta: Atleta):
    // Fórmula compuesta para índice de desempeño
    puntos_ponderados = atleta.puntos_totales * 1000
    wr_ponderado = atleta.win_rate * 100
    eff_ponderada = atleta.efectividad * 10
    
    RETORNAR puntos_ponderados + wr_ponderado + eff_ponderada

FUNCION actualizar_historial_companeros(atleta1: Atleta, atleta2: Atleta, ronda: entero):
    atleta1.historial_companeros.agregar({atleta_id: atleta2.id, ronda: ronda})
    atleta2.historial_companeros.agregar({atleta_id: atleta1.id, ronda: ronda})

FUNCION actualizar_historial_rivales(pareja1: Pareja, pareja2: Pareja, ronda: entero):
    // Actualizar para todos los cruces posibles
    pareja1.atleta1.historial_rivales.agregar({atleta_id: pareja2.atleta1.id, ronda: ronda})
    pareja1.atleta1.historial_rivales.agregar({atleta_id: pareja2.atleta2.id, ronda: ronda})
    pareja1.atleta2.historial_rivales.agregar({atleta_id: pareja2.atleta1.id, ronda: ronda})
    pareja1.atleta2.historial_rivales.agregar({atleta_id: pareja2.atleta2.id, ronda: ronda})
    
    // Recíproco
    pareja2.atleta1.historial_rivales.agregar({atleta_id: pareja1.atleta1.id, ronda: ronda})
    pareja2.atleta1.historial_rivales.agregar({atleta_id: pareja1.atleta2.id, ronda: ronda})
    pareja2.atleta2.historial_rivales.agregar({atleta_id: pareja1.atleta1.id, ronda: ronda})
    pareja2.atleta2.historial_rivales.agregar({atleta_id: pareja1.atleta2.id, ronda: ronda})

// MANEJO DE CASOS ESPECIALES

FUNCION ajustar_para_mesas_impares(piramide_superior: lista<Atleta>, piramide_inferior: lista<Atleta>):
    // Mover últimos atletas de pirámide superior a encabezar pirámide inferior
    SI longitud(piramide_superior) MOD 4 == 2:
        ultimos_dos = piramide_superior[-2:]
        piramide_superior.remover(ultimos_dos[0])
        piramide_superior.remover(ultimos_dos[1])
        piramide_inferior.insertar(0, ultimos_dos[0])
        piramide_inferior.insertar(1, ultimos_dos[1])

FUNCION manejar_bye(atleta: Atleta, ronda: entero):
    atleta.tiene_bye = VERDADERO
    atleta.puntos_totales += 0.5  // Medio punto por bye
    atleta.partidas_jugadas += 1
    // No afecta win_rate ni efectividad

// FUNCIÓN PRINCIPAL DEL TORNEO

FUNCION ejecutar_torneo(atletas: lista<Atleta>, total_rondas: entero, ordenamiento: enum):
    config = ConfiguracionTorneo{
        total_participantes: longitud(atletas),
        ronda_actual: 1,
        total_rondas: total_rondas,
        ordenamiento_inicial: ordenamiento
    }
    
    config = inicializar_torneo(atletas, config)
    historial_mesas = lista<lista<Mesa>>
    
    PARA ronda = 1 HASTA total_rondas:
        config.ronda_actual = ronda
        mesas = generar_emparejamiento_ronda(atletas, config)
        historial_mesas.agregar(mesas)
        
        // Simular resultados (en producción vendría de input real)
        PARA mesa EN mesas:
            procesar_resultado_mesa(mesa)
            actualizar_estadisticas_atletas(mesa)
    
    RETORNAR {
        clasificacion_final: ordenar_por_indice_desempeno(atletas),
        historial_completo: historial_mesas,
        estadisticas: generar_estadisticas_torneo(atletas, historial_mesas)
    }
}
```
