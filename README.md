# 🧬 Bioinformatics Toolkit

Herramienta visual interactiva de bioinformática que implementa tres algoritmos clásicos con visualización estilo pizarra: **Needleman-Wunsch**, **Smith-Waterman** y **Burrows-Wheeler Transform + FM-Index**.

---

## ✨ Características

- **Sin instalación** — un solo archivo HTML, funciona directamente en el navegador
- **Tres modos** seleccionables con pestañas
- **Visualización completa paso a paso** de cada algoritmo
- **Parámetros configurables** (match, mismatch, gap)
- **Estilo oscuro** inspirado en pizarra de marcador

---

## 🚀 Uso rápido

1. Abre `alignment_visualizer.html` en cualquier navegador moderno (Chrome, Firefox, Edge, Safari)
2. Selecciona el modo con las pestañas superiores
3. Ingresa tus secuencias / texto / patrón
4. Haz clic en el botón **▶ ALINEAR** o **▶ TRANSFORMAR Y BUSCAR**

No requiere servidor, backend ni conexión a internet.

---

## 🔬 Modos disponibles

### 🟢 NW — Needleman-Wunsch (Alineamiento Global)

Alinea dos secuencias completas penalizando todos los gaps desde el inicio hasta el final.

**Entradas:**
| Campo | Descripción | Ejemplo |
|-------|-------------|---------|
| Secuencia Horizontal (seq1) | Primera secuencia (eje X de la matriz) | `GCATTA` |
| Secuencia Vertical (seq2) | Segunda secuencia (eje Y de la matriz) | `GCATA` |
| Match | Puntuación por bases iguales | `+1` |
| Mismatch | Penalización por bases distintas | `-1` |
| Gap | Penalización por inserción/deleción | `-1` |

**Salida:**
- Alineamiento final con `|` (match), `·` (mismatch), `-` (gap)
- Matriz DP completa con las 3 operaciones por celda: `↖ A+1=B`, `↑ A-2=B`, `← A-2=B`
- Camino de traceback resaltado en amarillo
- Puntaje final en círculo rojo

**Traceback:** desde la celda `[m][n]` (esquina inferior derecha) hasta `[0][0]`.

---

### 🔴 SW — Smith-Waterman (Alineamiento Local)

Encuentra la subsecuencia de mayor similitud entre dos secuencias, ignorando regiones de baja similitud.

Idéntica interfaz a NW, con las siguientes diferencias algorítmicas:

- Las celdas **nunca bajan de 0** (`max(..., 0)`)
- El **traceback comienza en la celda de mayor score** (no en la esquina)
- El traceback **termina al llegar a una celda con valor 0**
- Ideal para encontrar dominios conservados o motivos locales

---

### 🔵 BWT + FM-Index (Búsqueda de Patrones)

Implementa la **Transformada de Burrows-Wheeler** completa junto con el **FM-Index** para búsqueda eficiente de patrones en un texto genómico.

**Entradas:**
| Campo | Descripción | Ejemplo |
|-------|-------------|---------|
| Texto / Genoma | Secuencia donde buscar | `GATTACA` |
| Patrón | Subcadena a encontrar | `ATT` |

**Salida — 4 pasos visuales:**

#### Paso 1 — Rotaciones y BWT
- Tabla con todas las `n+1` rotaciones del texto (con centinela `$`)
- Rotaciones ordenadas lexicográficamente (`$` < letras)
- Columna **F** (primera, en verde) y columna **L / BWT** (última, en rojo)
- Array de sufijos **SA[i]** para cada rotación
- Fila original destacada en azul

#### Paso 2 — Tablas FM-Index
- **Tabla C**: conteo acumulativo de caracteres en F que son lexicográficamente menores que `c`
  ```
  C[c] = | { i : T[i] < c } |
  ```
- **Tabla Occ**: Occ[c][i] = número de ocurrencias de `c` en BWT[0..i-1]

#### Paso 3 — Búsqueda hacia atrás (Backward Search)
El patrón se procesa **de derecha a izquierda** usando la fórmula:
```
lo = C[ch] + Occ[ch][lo]
hi = C[ch] + Occ[ch][hi]
```
La tabla muestra por cada carácter: rango de entrada → cálculo detallado → rango de salida → estado ✓/✗.

#### Paso 4 — Ocurrencias en el texto original
- Texto completo con todas las ocurrencias resaltadas en **fondo amarillo (#FFFF00) + texto negro negrita**
- Cada ocurrencia en su propia línea con posición anotada
- Regla de posiciones con `↑` y el patrón impreso debajo de cada coincidencia
- Posiciones finales en **números circulados grandes** ① ② ③ en rojo

---

## 🎨 Guía visual de colores

| Color | Significado |
|-------|-------------|
| 🟡 Amarillo `#FFFF00` | Ocurrencia del patrón (BWT) / camino de traceback (NW/SW) |
| 🔴 Rojo `#ff3c3c` | Operación elegida / valor máximo de celda / posiciones encontradas |
| 🟢 Verde `#39ff14` | Match entre caracteres / columna F (BWT) |
| 🔵 Azul `#4da6ff` | Modo BWT / cálculos del FM-Index |
| 🟣 Púrpura `#c084fc` | Tabla Occ |
| ⚫ Gris | Valores no elegidos / posiciones no relevantes |

---

## 📐 Detalles de implementación

### Needleman-Wunsch
```
dp[i][j] = max(
  dp[i-1][j-1] + sub(seq2[i], seq1[j]),   ↖ diagonal
  dp[i-1][j]   + gap,                       ↑ arriba
  dp[i][j-1]   + gap                        ← izquierda
)
Inicialización: dp[i][0] = i * gap,  dp[0][j] = j * gap
```

### Smith-Waterman
```
dp[i][j] = max(
  dp[i-1][j-1] + sub(seq2[i], seq1[j]),
  dp[i-1][j]   + gap,
  dp[i][j-1]   + gap,
  0                                          ← nunca negativo
)
Inicialización: toda la primera fila y columna = 0
```

### BWT
```
BWT(T) = última columna de la matriz de rotaciones ordenadas
       = T[(SA[i] - 1 + n) mod n]  para cada i
```

### FM-Index Backward Search
```
Para cada c = pattern[k] (k desde len-1 hasta 0):
  lo_new = C[c] + Occ[c][lo]
  hi_new = C[c] + Occ[c][hi]
  si lo_new >= hi_new → patrón no encontrado

Posiciones = { SA[i] + 1 : lo ≤ i < hi }   (1-based)
```

---

## ⚙️ Parámetros por defecto

| Parámetro | Valor | Descripción |
|-----------|-------|-------------|
| Match | `+1` | Bases idénticas |
| Mismatch | `-1` | Bases distintas |
| Gap | `-2` | Inserción o deleción |

Todos los parámetros son enteros y pueden ser positivos o negativos.

---

## 🧪 Ejemplos de prueba

### Alineamiento global (NW)
```
seq1: GCATTA
seq2: GCATA
Match: +1 | Mismatch: -1 | Gap: -2

Resultado:
Cadena 1: G C A T T A
Cadena 2: G C A - T A
Score: 3
```

### Alineamiento local (SW)
```
seq1: TGTTACGG
seq2: GGTTGACTA
Match: +3 | Mismatch: -3 | Gap: -2
```

### BWT + búsqueda
```
Texto:   BANANA
Patrón:  ANA
→ Posiciones: ② ④
```

---

## 🌐 Compatibilidad

| Navegador | Versión mínima |
|-----------|---------------|
| Chrome / Chromium | 80+ |
| Firefox | 75+ |
| Edge | 80+ |
| Safari | 13.1+ |

No se requieren dependencias externas. Las fuentes se cargan desde Google Fonts (requiere internet solo para tipografías; sin internet se usará monospace del sistema).

---

## 📁 Estructura del proyecto

```
alignment_visualizer.html   ← archivo único, todo incluido
README.md                   ← este archivo
```

Todo el código (HTML + CSS + JavaScript) está autocontenido en un único archivo de ~500 líneas.

---

## 📚 Referencias

- Needleman, S.B. & Wunsch, C.D. (1970). *A general method applicable to the search for similarities in the amino acid sequence of two proteins.* Journal of Molecular Biology, 48(3), 443–453.
- Smith, T.F. & Waterman, M.S. (1981). *Identification of common molecular subsequences.* Journal of Molecular Biology, 147(1), 195–197.
- Burrows, M. & Wheeler, D.J. (1994). *A block-sorting lossless data compression algorithm.* Technical Report 124, Digital Equipment Corporation.
- Ferragina, P. & Manzini, G. (2000). *Opportunistic data structures with applications.* Proceedings of the 41st Annual Symposium on Foundations of Computer Science, 390–398.
