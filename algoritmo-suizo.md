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
    /* Internacional: > 76 */

    VENTANA_COMP_COMPACTA       = 3       // rondas
    VENTANA_COMP_ESTANDAR       = 4
    VENTANA_COMP_INTERNACIONAL  = 5

    VENTANA_RIVAL_BASE          = 1       // constante para todos

    CONTING_MAX_INTENTOS        = 10
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
    estado: enum { ACTIVO, BYE, RETIRADO }
    historial_comp: Lista<{ id: string, ronda: int }>
    historial_riv: Lista<{ id: string, ronda: int }>
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
    activos <- FILTRAR(jugadores_ordenados, j => j.estado == ACTIVO)
    categoria <- DeterminarCategoria(TAMANO(activos))
    ventana_comp <- ObtenerVentanaCompaneros(categoria, ronda)

    bloques <- DividirEnBloquesPorVictorias(activos)
    AjustarParidadBloques(bloques)           // añade atletas del bloque inferior si es impar

    enfrentamientos_total <- []
    conting_log <- []
    POR bloque EN bloques
        parejas <- FormarParejas(bloque, ventana_comp, ronda, conting_log)
        mesas_bloque <- PlegarBloque(parejas, ronda, conting_log)
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
FUNCION FormarParejas(bloque: Lista<Jugador>, ventana: int, ronda: int, log: Lista)
    sin_pareja <- COPIA(bloque)
    parejas <- []

    MIENTRAS (TAMANO(sin_pareja) >= 2)
        j1 <- sin_pareja[0]
        candidato <- BuscarParejaValida(j1, sin_pareja[1:], ventana, ronda)
        SI (candidato == NULO)
            // Contingencia Nivel 1: Búsqueda extendida
            candidato <- BuscarExtendido(j1, sin_pareja, ventana, ronda)
        FIN SI
        SI (candidato == NULO)
            // Contingencia Nivel 2: Intercambio
            exito <- IntercambiarPareja(j1, parejas, ventana, ronda)
            SI (NO exito)
                log.ANADIR({nivel:2, descripcion:"Fallo intercambio – se reduce ventana"})
                ventana_reducida <- MAX(2, ventana-1)
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
FUNCION PlegarBloque(parejas: Lista<Pareja>, ronda: int, log: Lista) -> Lista<Mesa>
    mitad <- TAMANO(parejas) / 2
    superior <- parejas[0:mitad]
    inferior <- parejas[mitad:]
    mesas <- []

    PARA i DESDE 0 HASTA TAMANO(superior)-1
        pa <- superior[i]
        pb <- inferior[i]
        SI (SonRivalesRecientes(pa, pb, ronda))
            // Contingencia – rotación interna inferior
            exito <- RotarInferior(inferior, i, ventana=1)
            SI (NO exito)
                log.ANADIR({nivel:3, descripcion:"Transfusión pirámide"})
                // Lógica de transfusión omitida por brevedad
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
    restantes.ORDENAR_POR(j => UltimaRondaBYE(j))
    bye_lista <- []
    MIENTRAS (TAMANO(restantes) % 2 != 0)  // dejar tamaño par para futura ronda
        j <- restantes[0]
        j.estado <- BYE
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
