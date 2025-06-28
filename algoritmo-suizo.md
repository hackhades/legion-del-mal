# Algoritmo Suizo para Torneos de Dominó Profesional 
## Modalidad: Parejas Rotativas – Pseudocódigo Técnico con Explicaciones Embebidas

---
> Este documento contiene la versión **canónica** del algoritmo suizo que regirá los emparejamientos en el sistema.  
> El pseudocódigo está comentado en Markdown para facilitar la auditoría humana.

### Leyenda y Convenciones
- Los bloques de código usan un lenguaje neutro estilo "pseudocode".  
- Comentarios largos de explicación se colocan **fuera** del bloque de código.  
- Comentarios cortos de una línea dentro del bloque se indican con `//`.

---
## 1. Constantes y Parámetros Globales
```pseudocode
CONSTANTES:
    CATEGORIA_COMPACTA_MAX      = 36     // participantes
    CATEGORIA_ESTANDAR_MAX      = 76
    CATEGORIA_INTERNACIONAL_MIN  = 77     // Internacional ≥ 77

    VENTANA_COMP_COMPACTA       = 3       // rondas
    VENTANA_COMP_ESTANDAR       = 4
    VENTANA_COMP_INTERNACIONAL  = 5

    VENTANA_RIVAL_COMPACTA       = 1       // rivales pueden repetirse después de 1 ronda
    VENTANA_RIVAL_ESTANDAR       = 2
    VENTANA_RIVAL_INTERNACIONAL  = 3

    VENTANA_RIVAL_BASE          = 1       // constante para todos

    CONTING_MAX_INTENTOS_BASE   = 10   // ≤ 32 atletas
    CONTING_MAX_INTENTOS_EXT    = 12   // > 32 atletas
    CONTING_MAX_DISTANCIA       = 5

    NIVEL_CALIDAD_OBJETIVO      = 0.85    // 85 %
```

---
## 2. Estructuras de Datos
> Se simplifican para la descripción; el sistema real puede mapearlas a tablas SQL o clases TS.
```pseudocode
ESTRUCTURA Jugador {
    id: string
    ranking: integer        // posición global recibida
    victorias: integer      // acumuladas
    estado: enum { ACTIVO, BYE, RETIRADO, INCOMPARECENTE }
    historial_comp: Lista<{ id: string, ronda: int }>
    historial_riv: Lista<{ id: string, ronda: int }>
    bye_anterior: bool      // true si recibió BYE en la ronda previa
}

ESTRUCTURA Pareja {
    jugador1: Jugador
    jugador2: Jugador
}

ESTRUCTURA Mesa {
    numero: int
    pareja_a: Pareja
    pareja_b: Pareja
}

ESTRUCTURA ResultadoRonda {
    mesas: Lista<Mesa>
    jugadores_bye: Lista<Jugador>
    contingencias: Lista<{ nivel: int, descripcion: string }>
    calidad: float
}
```

---
## 2.1 Preparación de Lista Inicial
```pseudocode
FUNCION PrepararListaInicial(jugadores: Lista<Jugador>, modo_inicio: string, semilla: int) -> Lista<Jugador>
    SI (modo_inicio == "ELO")
        ORDENAR_DESC(jugadores, j => j.elo)           // mejor ELO primero
    SINO  // ALEATORIO
        SI (semilla != NULO) INICIALIZAR_RANDOM(semilla)
        MEZCLAR_ALEATORIO(jugadores)
    FIN SI
    RETORNAR jugadores
FIN FUNCION
```

## 2.2 Motor Principal del Torneo
```pseudocode
FUNCION EjecutarTorneo(jugadores: Lista<Jugador>, rondas_totales: int, modo_inicio: string, semilla: int)
    // Preparar lista inicial según modo elegido (ELO o ALEATORIO)
    resultados <- []

    SI (modo_inicio == "ELO")
        // Ordenar por ELO y generar ronda 1 con algoritmo piramidal
        ORDENAR_DESC(jugadores, j => j.elo)
        resultado_r1 <- GenerarRondaInicialELO(jugadores, 1)
    SINO
        lista_r1 <- PrepararListaInicial(jugadores, modo_inicio, semilla)
        resultado_r1 <- GenerarRonda(lista_r1, 1)
    FIN SI
    resultados.ANADIR(resultado_r1)

    // Generar rondas subsiguientes
    POR ronda DESDE 2 HASTA rondas_totales
        // El módulo de clasificación externo retorna lista ordenada por victorias y desempates
        lista_ordenada <- ObtenerClasificacionOrdenada()
        resultado <- GenerarRonda(lista_ordenada, ronda)
        resultados.ANADIR(resultado)
    FIN POR

    RETORNAR resultados
FIN FUNCION
```

## 2.3 Generación de Ronda 1 con ELO – Algoritmo Piramidal
```pseudocode
FUNCION GenerarRondaInicialELO(jugadores_ordenados: Lista<Jugador>, ronda: int) -> ResultadoRonda
    // 0. Filtrar atletas activos y ausentes (INCOMPARECENTE)
    //    y asignar BYE a los de menor ELO si el total no es múltiplo de 4
    activos <- FILTRAR(jugadores_ordenados, j => j.estado == ACTIVO)
    ausentes <- FILTRAR(jugadores_ordenados, j => j.estado == INCOMPARECENTE)

    // Ajustar BYE por menor ELO en caso de que activos % 4 != 0
    resto <- TAMANO(activos) % 4
    bye_elo <- []
    SI (resto != 0)
        bye_elo <- ExtraerUltimos(activos, resto)   // menor ELO están al final
    FIN SI

    // 1. Formar parejas contiguas (A1-A2, B1-B2, ...) usando solo activos restantes
    parejas_elo <- []
    PARA i DESDE 0 HASTA TAMANO(activos)-1 PASO 2
        parejas_elo.ANADIR({activos[i], activos[i+1]})
    FIN PARA

    // 2. Dividir en Pirámide Principal (PP) y Secundaria (PS)
    mitad <- TAMANO(parejas_elo)/2
    PP <- parejas_elo[0:mitad]
    PS <- parejas_elo[mitad:]

    // 3. Subdividir cada pirámide en lados A y B
    ladoA_PP <- PP[0:CEIL(TAMANO(PP)/2)]
    ladoB_PP <- PP[CEIL(TAMANO(PP)/2):]
    ladoA_PS <- PS[0:CEIL(TAMANO(PS)/2)]
    ladoB_PS <- PS[CEIL(TAMANO(PS)/2):]

    // 4. Asegurar paridad (mover pares top de PS a PP si algún lado PP es impar)
    SI ( (TAMANO(ladoA_PP) % 2) != 0 )
        mover <- ExtraerPrimeros(ladoA_PS, 1)
        ladoA_PP.ANADIR(mover)
    FIN SI
    SI ( (TAMANO(ladoB_PP) % 2) != 0 )
        mover <- ExtraerPrimeros(ladoB_PS, 1)
        ladoB_PP.ANADIR(mover)
    FIN SI

    // 5. Crear mesas PP (Mesa 1..n) y luego PS
    mesas <- []
    PARA i DESDE 0 HASTA TAMANO(ladoA_PP)-1
        mesas.ANADIR(CrearMesa(ladoA_PP[i], ladoB_PP[i]))
    FIN PARA
    PARA i DESDE 0 HASTA TAMANO(ladoA_PS)-1
        mesas.ANADIR(CrearMesa(ladoA_PS[i], ladoB_PS[i]))
    FIN PARA

    // 6. Gestionar BYE: ausentes + posibles faltantes de mod 4
    bye_auto <- GestionarBYE(activos, mesas, ronda)  // probablemente vacío en R1 con ELO
    bye_total <- CONCATENAR(bye_auto, ausentes, bye_elo)

    calidad <- CalcularCalidad(mesas, activos, bye_total)
    retornar <- ResultadoRonda{mesas: mesas, jugadores_bye: bye_total, contingencias: [], calidad: calidad}
    RETORNAR retornar
FIN FUNCION
```

## 2.4 Contingencia Nivel 3 – Transfusión de Bloques
```pseudocode
FUNCION TransfundirParejas(superior: Lista<Pareja>, inferior: Lista<Pareja>, idx: int, ronda: int, ventana_riv: int) -> Mesa | NULO
    pa <- superior[idx]
    original_pb <- inferior[idx]

    // Buscar pareja en inferior que no viole reglas con pa
    PARA j DESDE idx+1 HASTA TAMANO(inferior)-1
        q <- inferior[j]
        SI (
            NO SonCompanerosRecientes(pa, q)
            Y NO SonRivalesRecientes(pa, q, ronda, ventana_riv)
        )
            // Intercambiar posiciones: q sube, original_pb baja
            inferior[j] <- original_pb
            inferior[idx] <- q
            RETURN CrearMesa(pa, q)   // Mesa válida tras transfusión
        FIN SI
    FIN PARA
    RETORNAR NULO
FIN FUNCION
```

## 3. Flujo Principal por Ronda
### Explicación breve
1. Recibe lista ordenada de atletas activos.  
2. Divide en bloques según victorias.  
3. Ajusta paridad de cada bloque (múltiplos de 4).  
4. Forma parejas con ventana de compañeros.  
5. Plega mitades y valida rivales.  
6. Aplica contingencias escalonadas si hay violaciones.  
7. Asigna mesas, BYE y calcula métrica de calidad.

```pseudocode
FUNCION GenerarRonda(jugadores_ordenados: Lista<Jugador>, ronda: int) -> ResultadoRonda
    // Limpia bandera BYE anterior
    POR j EN jugadores_ordenados
        j.bye_anterior <- FALSO
    FIN POR

    activos <- FILTRAR(jugadores_ordenados, j => j.estado == ACTIVO)
    categoria <- DeterminarCategoria(TAMANO(activos))
    ventana_comp <- ObtenerVentanaCompaneros(categoria, ronda)
    ventana_riv <- ObtenerVentanaRivales(categoria)

    bloques <- DividirEnBloquesPorVictorias(activos)
    AjustarParidadBloques(bloques)           // añade atletas del bloque inferior si es impar

    enfrentamientos_total <- []
    conting_log <- []
    POR bloque EN bloques
        parejas <- FormarParejas(bloque, ventana_comp, ventana_riv, ronda, conting_log)
        mesas_bloque <- PlegarBloque(parejas, ronda, ventana_riv, conting_log)
        enfrentamientos_total.ANADIR(mesas_bloque)
    FIN POR

    NumerarMesas(enfrentamientos_total)      // Mesa 1 = líderes

    bye_asignados <- GestionarBYE(activos, enfrentamientos_total, ronda)

    calidad <- CalcularCalidad(enfrentamientos_total, activos, bye_asignados)

    REGISTRAR_LogRonda(ronda, enfrentamientos_total, conting_log, bye_asignados, calidad)

    RETORNAR ResultadoRonda {
        mesas: enfrentamientos_total,
        jugadores_bye: bye_asignados,
        contingencias: conting_log,
        calidad: calidad
    }
FIN FUNCION
```

---
## 4. Módulos Clave
### 4.1 División y Ajuste de Bloques
```pseudocode
FUNCION AjustarParidadBloques(bloques: Lista<Bloque>)
    POR i DESDE 0 HASTA TAMANO(bloques)-1
        bloque <- bloques[i]
        SI (TAMANO(bloque) % 4 == 0)
            CONTINUAR
        FIN SI
        deficit <- 4 - (TAMANO(bloque) % 4)
        j <- i + 1
        MIENTRAS (deficit > 0 Y j < TAMANO(bloques))
            mover <- ExtraerPrimeros(bloques[j], MIN(deficit, TAMANO(bloques[j])))
            bloque.ANADIR(mover)
            deficit <- 4 - (TAMANO(bloque) % 4)
            j++
        FIN MIENTRAS
    FIN POR
FIN FUNCION
```

### 4.2 Formación de Parejas con Ventanas y Contingencias
```pseudocode
FUNCION FormarParejas(bloque: Lista<Jugador>, ventana_comp: int, ventana_riv: int, ronda: int, log: Lista)
    sin_pareja <- COPIA(bloque)
    parejas <- []

    MIENTRAS (TAMANO(sin_pareja) >= 2)
        j1 <- sin_pareja[0]
        candidato <- BuscarParejaValida(j1, sin_pareja[1:], ventana_comp, ronda)
        SI (candidato == NULO)
            // Contingencia Nivel 1: Búsqueda extendida
            candidato <- BuscarExtendido(j1, sin_pareja, ventana_comp, ronda)
        FIN SI
        SI (candidato == NULO)
            // Contingencia Nivel 2: Intercambio
            exito <- IntercambiarPareja(j1, parejas, ventana_comp, ronda)
            SI (NO exito)
                log.ANADIR({nivel:2, descripcion:"Fallo intercambio – se reduce ventana"})
                ventana_reducida <- MAX(2, ventana_comp-1)
                candidato <- BuscarParejaValida(j1, sin_pareja[1:], ventana_reducida, ronda)
            FIN SI
        FIN SI

        SI (candidato != NULO)
            parejas.ANADIR({j1, candidato})
            sin_pareja.QUITAR(j1, candidato)
        SINO
            // caso extremo: marcar BYE (gestor BYE decidirá)
            sin_pareja.QUITAR(j1)
            j1.estado <- BYE_PENDIENTE
        FIN SI
    FIN MIENTRAS

    RETORNAR parejas
FIN FUNCION
```

### 4.3 Plegado y Validación de Rivales
```pseudocode
FUNCION PlegarBloque(parejas: Lista<Pareja>, ronda: int, ventana_riv: int, log: Lista) -> Lista<Mesa>
    mitad <- TAMANO(parejas) / 2
    superior <- parejas[0:mitad]
    inferior <- parejas[mitad:]
    mesas <- []

    PARA i DESDE 0 HASTA TAMANO(superior)-1
        pa <- superior[i]
        pb <- inferior[i]
        SI (SonRivalesRecientes(pa, pb, ronda, ventana_riv))
            // Contingencia – rotación interna inferior
            exito <- RotarInferior(inferior, i, ventana=1)
            SI (NO exito)
                // Nivel 3 – Transfusión entre pirámides / bloques
                mesas_pir <- TransfundirParejas(superior, inferior, i, ronda, ventana_riv)
                SI (mesas_pir != NULO)
                    log.ANADIR({nivel:3, descripcion:"Transfusión pirámide aplicada"})
                    mesas.ANADIR(mesas_pir)
                    CONTINUAR  // saltamos añadir mesa estándar
                SINO
                    log.ANADIR({nivel:3, descripcion:"Transfusión fallida"})
                FIN SI
            FIN SI
        FIN SI
        mesas.ANADIR(CrearMesa(pa, pb))
    FIN PARA
    RETORNAR mesas
FIN FUNCION
```

### 4.4 BYE y Abandonos
```pseudocode
FUNCION GestionarBYE(activos: Lista<Jugador>, mesas: Lista<Mesa>, ronda: int) -> Lista<Jugador>
    asignados <- JugadoresEnMesas(mesas)
    restantes <- DIFERENCIA(activos, asignados)
    // priorizar quien NO haya tenido BYE recientemente
    // Orden: primero quienes NO tuvieron BYE en la ronda previa, luego peor ranking
    restantes.ORDENAR_POR(j => (j.bye_anterior ? 0 : 1, -j.ranking))
    bye_lista <- []
    resto <- TAMANO(activos) % 4
    byes_necesarios <- (resto == 0) ? 0 : 4 - resto
    contador <- 0
    MIENTRAS (contador < byes_necesarios)
        j <- restantes[0]
        j.estado <- BYE
        j.bye_anterior <- VERDADERO
        bye_lista.ANADIR(j)
        restantes.QUITAR(j)
    FIN MIENTRAS
    RETORNAR bye_lista
FIN FUNCION
```

---
## 5. Métrica de Calidad
```pseudocode
FUNCION CalcularCalidad(mesas: Lista<Mesa>, jugadores_activos: Lista<Jugador>, bye_lista: Lista<Jugador>) -> float
    total_activos <- TAMANO(jugadores_activos) - TAMANO(bye_lista)
    emparejamientos_totales <- total_activos
    emparejamientos_validos <- ContarEmparejamientosValidos(mesas)
    RETURN emparejamientos_validos / emparejamientos_totales
FIN FUNCION
```

---
## 6. Registro y Auditoría
> Cada ronda se persiste como un objeto JSON; esto permite auditorías posteriores.
```pseudocode
FUNCION REGISTRAR_LogRonda(ronda, mesas, conting, bye, calidad)
    log <- {
        ronda: ronda,
        mesas: mesas,
        contingencias: conting,
        jugadores_bye: MAPEAR(bye, j => j.id),
        calidad: calidad
    }
    ESCRIBIR_JSON_LINE("logs/torneo.log", log)
FIN FUNCION
```

---
## 7. Ventana Dinámica (Fase Final)
```pseudocode
FUNCION ObtenerVentanaCompaneros(categoria: string, ronda: int) -> int
    ventana_base <- CASO categoria:
        "COMPACTA": VENTANA_COMP_COMPACTA
        "ESTANDAR": VENTANA_COMP_ESTANDAR
        "INTERNACIONAL": VENTANA_COMP_INTERNACIONAL
    FIN CASO

    // Flexibilización progresiva en finales
    SI (categoria == "COMPACTA" Y ronda >= 7) ventana_base--
    SI (categoria == "ESTANDAR" Y ronda >= 8) ventana_base--
    SI (categoria == "INTERNACIONAL" Y ronda >= 9) ventana_base--
    RETORNAR MAX(2, ventana_base)
FIN FUNCION
```

---
## 7.1 Ventana de Rivales por Categoría
```pseudocode
FUNCION ObtenerVentanaRivales(categoria: string) -> int
    RETORNAR CASO categoria:
        "COMPACTA": VENTANA_RIVAL_COMPACTA
        "ESTANDAR": VENTANA_RIVAL_ESTANDAR
        "INTERNACIONAL": VENTANA_RIVAL_INTERNACIONAL
    FIN CASO
FIN FUNCION
```

---
## 8. Ejemplo de Salida (Ronda 2)
```text
RONDA 2
Mesa 1: A1 – C1  vs  I1 – E1
Mesa 2: A2 – C2  vs  I2 – E2
Mesa 3: M1 – G1  vs  K1 – L1
Mesa 4: M2 – G2  vs  K2 – L2
Mesa 5: F1 – J1  vs  H2 – N2
Mesa 6: F2 – J2  vs  D1 – B1
Mesa 7: H1 – N1  vs  D2 – B2
BYE: —
Calidad: 1.00
```

---
### Fin del Documento
```
