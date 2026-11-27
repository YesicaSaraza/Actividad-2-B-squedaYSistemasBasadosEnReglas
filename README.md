# Actividad-2-B-squedaYSistemasBasadosEnReglas
import heapq
import math

# 1. BASE DE CONOCIMIENTO (HECHOS DEL SISTEMA)
# Coordenadas aproximadas para que el programa sepa qué tan lejos está cada estación
coordenadas = {
    "San_Mateo": (0, 0),
    "Terreros": (1, 1),
    "Leon_XIII": (2, 2),
    "La_Despensa": (3, 3),
    "Bosa": (4, 3),
    "Guatoque_Veraguas": (11, 8),
    "Ricaurte": (12, 9),
    "Paloquemao": (12, 10),
    "Calle_85": (14, 17),
}

# Tramos reales: (Origen, Destino, Ruta, Minutos_sin_trancon)
tramos = [
    # Opción 1: K43 (No para en Bosa) + B11 en Ricaurte
    ("San_Mateo", "Terreros", "K43", 6),
    ("Terreros", "Leon_XIII", "K43", 5),
    ("Leon_XIII", "La_Despensa", "K43", 5),
    ("La_Despensa", "Ricaurte", "K43", 23),
    ("Ricaurte", "Calle_85", "B11", 25),

    # Opción 2: E42 (Sí para en Bosa) + B11 en Paloquemao
    ("La_Despensa", "Bosa", "E42", 5),
    ("Bosa", "Paloquemao", "E42", 23),
    ("Paloquemao", "Calle_85", "B11", 22),

    # Opción 3: L41 + Ruta 8 (Para en todas, sirve los domingos)
    ("San_Mateo", "Bosa", "L41", 16),
    ("Bosa", "Guatoque_Veraguas", "L41", 22),
    ("Guatoque_Veraguas", "Calle_85", "Ruta_8", 43),
]


# 2. REGLAS LÓGICAS (CONDICIONES REALES)
def evaluar_viaje(ruta, ruta_anterior, minutos_base, dia, es_hora_pico):
    # Regla 1: El B11 no trabaja los domingos ni festivos
    if ruta == "B11" and dia == "domingo":
        return None  # Camino bloqueado

    # Regla 2: Si es hora pico, el trancón en Soacha aumenta el tiempo un 48%
    tiempo = minutos_base
    if es_hora_pico and ruta in ["K43", "E42", "L41"]:
        tiempo = round(minutos_base * 1.48)
    elif es_hora_pico:
        tiempo = round(minutos_base * 1.15)

    # Regla 3: Si cambiamos de bus, sumamos tiempo de espera (transbordo)
    hubo_transbordo = (ruta_anterior is not None) and (ruta_anterior != ruta)
    if hubo_transbordo:
        tiempo += 10 if es_hora_pico else 4

    return tiempo, hubo_transbordo


# 3. BÚSQUEDA HEURÍSTICA (ALGORITMO A*)
def distancia_estimada(estacion_a, estacion_b):
    """Brújula: calcula qué tan cerca estamos de la meta."""
    x1, y1 = coordenadas[estacion_a]
    x2, y2 = coordenadas[estacion_b]
    return math.sqrt((x2 - x1) ** 2 + (y2 - y1) ** 2)


def buscar_mejor_ruta(inicio, destino, dia="martes", es_hora_pico=True):
    # Cola de prioridad: guarda (puntaje_total, minutos_reales, estacion, ruta_actual, historial)
    cola = [(0, 0, inicio, None, [])]

    while cola:
        puntaje, minutos, actual, ruta_actual, camino = heapq.heappop(cola)

        if actual == destino:
            return minutos, camino

        for orig, dest, ruta, min_base in tramos:
            if orig == actual:
                resultado = evaluar_viaje(ruta, ruta_actual, min_base, dia, es_hora_pico)
                if resultado is not None:
                    costo_tramo, transbordo = resultado
                    nuevo_tiempo = minutos + costo_tramo
                    # Fórmula A*: tiempo real + distancia que falta
                    prioridad = nuevo_tiempo + distancia_estimada(dest, destino)
                    nuevo_camino = camino + [(orig, dest, ruta, costo_tramo, transbordo)]
                    heapq.heappush(cola, (prioridad, nuevo_tiempo, dest, ruta, nuevo_camino))

    return None


# 4. PRUEBAS DEL SISTEMA
def mostrar_resultado(titulo, resultado):
    print(f"--- {titulo} ---")
    minutos, camino = resultado
    print(f"Tiempo total estimado: {minutos} minutos")
    for orig, dest, ruta, costo, transbordo in camino:
        aviso = " [TRANSBORDO]" if transbordo else ""
        print(f"  De {orig} a {dest} en {ruta} ({costo} min){aviso}")
    print()


if __name__ == "__main__":
    mostrar_resultado("PRUEBA 1: Martes en HORA PICO", buscar_mejor_ruta("San_Mateo", "Calle_85", "martes", True))
    mostrar_resultado("PRUEBA 2: Martes sin trancón (Hora Valle)", buscar_mejor_ruta("San_Mateo", "Calle_85", "martes", False))
    mostrar_resultado("PRUEBA 3: Domingo (B11 no opera)", buscar_mejor_ruta("San_Mateo", "Calle_85", "domingo", False))
