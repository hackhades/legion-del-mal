# Pseudocódigo - Algoritmo Sistema Suizo Dominó Profesional

## Estructuras de Datos

```
ESTRUCTURA Jugador {
    id: entero
    nombre: texto
    puntos: decimal
    wr2: decimal          // Win Rate
    eff: decimal          // Efectividad
    indicesSecundarios: arreglo[decimal]
    activo: booleano
    tuvoBye: booleano
}

ESTRUCTURA Emparejamiento {
    ronda: entero
    jugadorId: entero
    compañeroId: entero
    rival1Id: entero
    rival2Id: entero
    resultado: enum(VICTORIA, DERROTA, BYE)
    puntosObtenidos: entero
}

ESTRUCTURA Mesa {
    numero: entero
    pareja1: Pareja
    pareja2: Pareja
}

ESTRUCTURA Pareja {
    jugador1: Jugador
    jugador2: Jugador
}

ESTRUCTURA ConfiguracionTorneo {
    totalRondas: entero
    rondaActual: entero
    categoria: enum(COMPACTA, ESTANDAR, INTERNACIONAL)
    ventanaCompañeros: entero
    ventanaRivales: entero
    metodoRonda1: enum(ALEATORIO, POR_ELO)
}
```

## Algoritmo Principal

```
ALGORITMO SistemaSwizoDomino(jugadores, configuracion) {
    
    // Inicialización
    historialEmparejamientos = []
    
    PARA ronda = 1 HASTA configuracion.totalRondas {
        
        SI ronda == 1 {
            mesas = generarRonda1(jugadores, configuracion.metodoRonda1)
        } SINO {
            // Actualizar clasificación
            actualizarClasificacion(jugadores, historialEmparejamientos)
            
            // Generar emparejamientos
            mesas = generarRondaSuizo(jugadores, historialEmparejamientos, configuracion, ronda)
        }
        
        // Registrar emparejamientos
        registrarEmparejamientos(mesas, historialEmparejamientos, ronda)
        
        // Mostrar mesas
        mostrarMesas(mesas)
    }
}
```

## Generación Ronda 1

```
FUNCION generarRonda1(jugadores, metodo) {
    jugadoresActivos = filtrar(jugadores, j => j.activo)
    
    SI metodo == ALEATORIO {
        mezclar(jugadoresActivos)
    } SINO SI metodo == POR_ELO {
        ordenar(jugadoresActivos, j => j.elo, DESCENDENTE)
    }
    
    // Manejar número impar
    SI longitud(jugadoresActivos) % 2 != 0 {
        ultimo = jugadoresActivos[longitud(jugadoresActivos) - 1]
        ultimo.tuvoBye = verdadero
        eliminar(jugadoresActivos, ultimo)
    }
    
    // Formar parejas y mesas
    mesas = []
    PARA i = 0 HASTA longitud(jugadoresActivos) - 1 PASO 4 {
        pareja1 = Pareja(jugadoresActivos[i], jugadoresActivos[i+1])
        pareja2 = Pareja(jugadoresActivos[i+2], jugadoresActivos[i+3])
        mesas.agregar(Mesa(i/4 + 1, pareja1, pareja2))
    }
    
    RETORNAR mesas
}
```

## Generación Rondas Sistema Suizo

```
FUNCION generarRondaSuizo(jugadores, historial, config, rondaActual) {
    jugadoresActivos = filtrar(jugadores, j => j.activo)
    
    // Determinar categoría y ventanas
    categoria = determinarCategoria(longitud(jugadoresActivos))
    ventanas = obtenerVentanas(categoria)
    
    // Ordenar por clasificación
    ordenar(jugadoresActivos, j => (j.puntos, j.wr2, j.eff), DESCENDENTE)
    
    // Dividir en pirámides
    mitad = longitud(jugadoresActivos) / 2
    piramidePrincipal = jugadoresActivos[0:mitad]
    piramideSecundaria = jugadoresActivos[mitad:]
    
    // Generar emparejamientos con validación
    mesas = []
    intentos = 0
    exito = falso
    
    MIENTRAS NO exito Y intentos < 3 {
        mesasTentativas = []
        jugadoresUsados = []
        
        // Procesar pirámide principal
        resultadoPP = procesarPiramide(
            piramidePrincipal, 
            historial, 
            ventanas, 
            rondaActual,
            jugadoresUsados
        )
        
        SI resultadoPP.exito {
            mesasTentativas.concatenar(resultadoPP.mesas)
            
            // Manejar sobrantes de PP
            SI longitud(resultadoPP.sobrantes) > 0 {
                // Insertar sobrantes al inicio de PS
                piramideSecundaria = resultadoPP.sobrantes + piramideSecundaria
            }
            
            // Procesar pirámide secundaria
            resultadoPS = procesarPiramide(
                piramideSecundaria,
                historial,
                ventanas,
                rondaActual,
                jugadoresUsados
            )
            
            SI resultadoPS.exito {
                mesasTentativas.concatenar(resultadoPS.mesas)
                exito = verdadero
                mesas = mesasTentativas
            }
        }
        
        SI NO exito {
            // Aplicar protocolo de contingencia
            ventanas = aplicarContingencia(ventanas, intentos + 1)
            intentos++
        }
    }
    
    // Numerar mesas
    PARA i = 0 HASTA longitud(mesas) - 1 {
        mesas[i].numero = i + 1
    }
    
    RETORNAR mesas
}
```

## Procesamiento de Pirámide

```
FUNCION procesarPiramide(jugadores, historial, ventanas, rondaActual, jugadoresUsados) {
    // Dividir en lados
    mitadPiramide = longitud(jugadores) / 2
    ladoA = jugadores[0:mitadPiramide]
    ladoB = jugadores[mitadPiramide:]
    
    // Calcular parejas por lado
    parejasLadoA = piso(longitud(ladoA) / 2)
    SI longitud(ladoA) % 2 != 0 {
        parejasLadoA = parejasLadoA
    }
    
    // Formar parejas con validación
    parejasA = []
    parejasB = []
    sobrantes = []
    
    // Emparejar Lado A
    resultadoA = emparejarLado(ladoA, historial, ventanas, rondaActual, jugadoresUsados)
    SI NO resultadoA.exito {
        RETORNAR {exito: falso}
    }
    parejasA = resultadoA.parejas
    
    // Emparejar Lado B
    resultadoB = emparejarLado(ladoB, historial, ventanas, rondaActual, jugadoresUsados)
    SI NO resultadoB.exito {
        RETORNAR {exito: falso}
    }
    parejasB = resultadoB.parejas
    
    // Manejar desbalance
    SI longitud(parejasA) != longitud(parejasB) {
        SI longitud(parejasA) > longitud(parejasB) {
            sobrantes = desemparejar(parejasA[longitud(parejasA) - 1])
            eliminar(parejasA, ultimo)
        } SINO {
            sobrantes = desemparejar(parejasB[longitud(parejasB) - 1])
            eliminar(parejasB, ultimo)
        }
    }
    
    // Crear mesas
    mesas = []
    PARA i = 0 HASTA longitud(parejasA) - 1 {
        SI validarEnfrentamiento(parejasA[i], parejasB[i], historial, ventanas, rondaActual) {
            mesas.agregar(Mesa(0, parejasA[i], parejasB[i]))
        } SINO {
            // Buscar pareja alternativa válida
            parejaAlternativa = buscarParejaValida(parejasA[i], parejasB, historial, ventanas, rondaActual, i)
            SI parejaAlternativa != nulo {
                intercambiar(parejasB, i, parejaAlternativa.indice)
                mesas.agregar(Mesa(0, parejasA[i], parejasB[i]))
            } SINO {
                RETORNAR {exito: falso}
            }
        }
    }
    
    RETORNAR {
        exito: verdadero,
        mesas: mesas,
        sobrantes: sobrantes
    }
}
```

## Validaciones

```
FUNCION validarEmparejamiento(jugador1, jugador2, historial, ventanas, rondaActual) {
    // Verificar que no fueron compañeros recientemente
    PARA ronda = max(1, rondaActual - ventanas.compañeros) HASTA rondaActual - 1 {
        emparejamientos = filtrar(historial, h => h.ronda == ronda)
        PARA emp EN emparejamientos {
            SI (emp.jugadorId == jugador1.id Y emp.compañeroId == jugador2.id) O
               (emp.jugadorId == jugador2.id Y emp.compañeroId == jugador1.id) {
                RETORNAR falso
            }
        }
    }
    RETORNAR verdadero
}

FUNCION validarEnfrentamiento(pareja1, pareja2, historial, ventanas, rondaActual) {
    jugadoresPareja1 = [pareja1.jugador1.id, pareja1.jugador2.id]
    jugadoresPareja2 = [pareja2.jugador1.id, pareja2.jugador2.id]
    
    // Verificar que no fueron rivales recientemente
    PARA ronda = max(1, rondaActual - ventanas.rivales) HASTA rondaActual - 1 {
        emparejamientos = filtrar(historial, h => h.ronda == ronda)
        PARA emp EN emparejamientos {
            SI emp.jugadorId EN jugadoresPareja1 {
                SI emp.rival1Id EN jugadoresPareja2 O emp.rival2Id EN jugadoresPareja2 {
                    RETORNAR falso
                }
            }
        }
    }
    RETORNAR verdadero
}
```

## Protocolo de Contingencia

```
FUNCION aplicarContingencia(ventanas, nivel) {
    nuevasVentanas = copiar(ventanas)
    
    SEGUN nivel {
        CASO 1:  // Relajación menor
            nuevasVentanas.compañeros = max(0, ventanas.compañeros - 1)
            
        CASO 2:  // Transfusión entre pirámides
            // Esto se maneja en el algoritmo principal intercambiando jugadores
            
        CASO 3:  // Relajación mayor
            nuevasVentanas.compañeros = max(0, ventanas.compañeros - 1)
            nuevasVentanas.rivales = max(1, ventanas.rivales - 1)
    }
    
    RETORNAR nuevasVentanas
}
```

## Funciones Auxiliares

```
FUNCION determinarCategoria(numeroJugadores) {
    SI numeroJugadores <= 36 {
        RETORNAR COMPACTA
    } SINO SI numeroJugadores <= 76 {
        RETORNAR ESTANDAR
    } SINO {
        RETORNAR INTERNACIONAL
    }
}

FUNCION obtenerVentanas(categoria) {
    SEGUN categoria {
        CASO COMPACTA:
            RETORNAR {compañeros: 1, rivales: 2}
        CASO ESTANDAR:
            RETORNAR {compañeros: 2, rivales: 3}
        CASO INTERNACIONAL:
            RETORNAR {compañeros: 3, rivales: 4}
    }
}

FUNCION emparejarLado(jugadores, historial, ventanas, rondaActual, jugadoresUsados) {
    parejas = []
    disponibles = copiar(jugadores)
    
    MIENTRAS longitud(disponibles) >= 2 {
        jugador1 = disponibles[0]
        eliminar(disponibles, 0)
        
        // Buscar compañero válido
        compañeroEncontrado = falso
        PARA i = 0 HASTA longitud(disponibles) - 1 {
            jugador2 = disponibles[i]
            
            SI validarEmparejamiento(jugador1, jugador2, historial, ventanas, rondaActual) Y
               jugador1.id NO EN jugadoresUsados Y jugador2.id NO EN jugadoresUsados {
                parejas.agregar(Pareja(jugador1, jugador2))
                jugadoresUsados.agregar(jugador1.id)
                jugadoresUsados.agregar(jugador2.id)
                eliminar(disponibles, i)
                compañeroEncontrado = verdadero
                ROMPER
            }
        }
        
        SI NO compañeroEncontrado {
            RETORNAR {exito: falso}
        }
    }
    
    RETORNAR {
        exito: verdadero,
        parejas: parejas
    }
}

FUNCION buscarParejaValida(parejaObjetivo, parejasCandidatas, historial, ventanas, rondaActual, indiceInicial) {
    PARA i = 0 HASTA longitud(parejasCandidatas) - 1 {
        SI i != indiceInicial {
            SI validarEnfrentamiento(parejaObjetivo, parejasCandidatas[i], historial, ventanas, rondaActual) {
                RETORNAR {indice: i, pareja: parejasCandidatas[i]}
            }
        }
    }
    RETORNAR nulo
}
```

## Manejo de Casos Especiales

```
FUNCION manejarBye(jugador, ronda) {
    // Asignar victoria con puntaje promedio
    puntajePromedio = calcularPuntajePromedio()
    
    emparejamiento = Emparejamiento {
        ronda: ronda,
        jugadorId: jugador.id,
        compañeroId: -1,  // Sin compañero
        rival1Id: -1,     // Sin rivales
        rival2Id: -1,
        resultado: BYE,
        puntosObtenidos: puntajePromedio
    }
    
    jugador.puntos += puntajePromedio
    jugador.tuvoBye = verdadero
    
    RETORNAR emparejamiento
}

FUNCION manejarAbandono(jugadorId, jugadores) {
    jugador = buscar(jugadores, j => j.id == jugadorId)
    SI jugador != nulo {
        jugador.activo = falso
    }
}
```

## Registro y Auditoría

```
FUNCION registrarDecision(tipo, descripcion, parametros) {
    registro = {
        timestamp: obtenerTiempoActual(),
        tipo: tipo,
        descripcion: descripcion,
        parametros: parametros
    }
    
    agregarALog(registro)
}

FUNCION generarReporteRonda(ronda, mesas, contingenciasAplicadas) {
    reporte = {
        ronda: ronda,
        totalMesas: longitud(mesas),
        emparejamientosOptimos: contarEmparejamientosOptimos(mesas),
        contingenciasAplicadas: contingenciasAplicadas,
        distribucionNiveles: calcularDistribucion(mesas)
    }
    
    RETORNAR reporte
}
```

## Validación de Integridad

```
FUNCION validarIntegridadTorneo(historial, configuracion) {
    metricas = {
        porcentajeOptimos: 0,
        repeticionesIndebidas: 0,
        contingenciasActivadas: 0,
        equidadByes: verdadero
    }
    
    // Verificar emparejamientos óptimos
    totalEmparejamientos = 0
    emparejamientosOptimos = 0
    
    PARA emp EN historial {
        totalEmparejamientos++
        SI verificarOrdenClasificacion(emp) {
            emparejamientosOptimos++
        }
    }
    
    metricas.porcentajeOptimos = (emparejamientosOptimos / totalEmparejamientos) * 100
    
    // Verificar repeticiones
    PARA jugador EN jugadores {
        repeticiones = contarRepeticiones(jugador.id, historial, configuracion)
        SI repeticiones > 1 {
            metricas.repeticionesIndebidas++
        }
    }
    
    // Verificar BYEs
    byesPorJugador = contarByesPorJugador(historial)
    SI max(byesPorJugador) - min(byesPorJugador) > 1 {
        metricas.equidadByes = falso
    }
    
    RETORNAR metricas
}
```

## Ejemplo de Uso

```
// Configuración inicial
configuracion = ConfiguracionTorneo {
    totalRondas: 9,
    rondaActual: 1,
    categoria: ESTANDAR,
    ventanaCompañeros: 2,
    ventanaRivales: 3,
    metodoRonda1: ALEATORIO
}

// Lista de jugadores (28 participantes)
jugadores = []
PARA i = 1 HASTA 28 {
    jugadores.agregar(Jugador {
        id: i,
        nombre: "Jugador" + i,
        puntos: 0,
        wr2: 0,
        eff: 0,
        activo: verdadero,
        tuvoBye: falso
    })
}

// Ejecutar torneo
SistemaSwizoDomino(jugadores, configuracion)
```
