# 🎹 Synthos
Synthos es un escenario musical para el ATmega16. Carga una canción, la reproduce y la visualiza en tiempo real sobre una matriz LED. Todo escrito en AVR Assembly puro, sin librerías externas.
1. Abre SimulIDE y carga el archivo de escenario `.simu` del proyecto.
2. Presiona cualquiera de las teclas conectadas al pin A (`A`, `S`, `D`, `F`, `G`, `H`, `J`, `K`) para activar el sistema e iniciar la canción.
3. La canción comienza a sonar y las notas se visualizan en tiempo real en el panel LED.
4. Al terminar, el sistema queda en espera. Presiona nuevamente para repetir.
---
## 🗂️ Seccionamiento Musical
En lugar de escribir cada nota de forma individual, las canciones se organizan en secciones reutilizables. Esto permite estructurar patrones repetidos sin duplicar código, reduciendo el tamaño del programa en aproximadamente un 70%.

Cada nota dentro de una sección se codifica en un solo byte:
```
[ 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 ] | [ 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 ] | [ 7 | 6 | 5 | - - -
\______5 bits______/ \__3 bits__/
      Posición         Duración
\_____________1 byte____________/
           Nota musical
\_____________________________________Sección_Musical_____________________________________/
```
Los 5 bits superiores almacenan la posición de la nota dentro del conjunto declarado, lo que da un límite de 32 notas musicales distintas por proyecto. No es necesario que sean contiguas en el piano, por lo que pueden abarcar más de 4 octavas sin problema.

| Posición (5 bits) | Rango        |
|-------------------|--------------|
| 00000 – 11111     | 0 – 31       |
| Máximo de notas   | 32           |
| Octavas posibles  | Sin límite   |

Los 3 bits inferiores almacenan la duración de la nota, abarcando desde la semifusa hasta la media. Fueron calibrados para una frecuencia de 1 MHz, lo que resulta en un tempo de aproximadamente 120 BPM.

| Valor | Duración          | Tiempo aproximado |
|-------|-------------------|-------------------|
| 001   | Media             | ≈ 1000 ms         |
| 010   | Cuarta            | ≈ 500 ms          |
| 011   | Octava            | ≈ 250 ms          |
| 100   | Dieciseisava      | ≈ 125 ms          |
| 101   | Treintaidosava    | ≈ 62.5 ms         |
| 110   | Sesentaicuatroava | ≈ 31.25 ms        |
---
## 🎵 Patrón Musical
Luego del seccionamiento, el programa necesita saber de qué manera se tocarán las secciones. Para ello, se define como 'patrón' a una secuencia de entradas, donde cada una indica desde qué nota de una sección comenzar, y cuántas notas reproducir a partir de ahí.

Esto permite no solo encadenar secciones completas, sino también reutilizarlas desde distintos puntos de entrada o recortarlas, sin necesidad de declararlas nuevamente.

```
Posiciones:  [  0 |  0 |  6 |  0 | ... ]   ← Desde qué nota iniciar
Duraciones:  [  6 |  6 |  4 |  6 | ... ]   ← Cuántas notas reproducir
```

---
## 📊 Visualizador de Notas
El visualizador representa la actividad musical en tiempo real mediante 8 barras verticales sobre la matriz LED. Cada barra corresponde a un grupo de notas del conjunto declarado, agrupadas en el orden en que fueron definidas, no por frecuencia ni altura.
El tamaño de cada grupo se determina tomando el múltiplo de 8 inmediatamente superior al total de notas del proyecto y dividiendo ese valor entre 8.

| Notas declaradas | Múltiplo superior | Notas por barra |
|------------------|-------------------|-----------------|
| 1 – 8            | 8                 | 1               |
| 9 – 16           | 16                | 2               |
| 17 – 24          | 24                | 3               |
| 25 – 32          | 32                | 4               |

La altura de cada barra refleja la duración de la nota activa. La cuarta lleva la barra al nivel máximo (8), y cada valor más corto reduce ese nivel en 2, hasta llegar a 0.

| Duración          | Nivel |
|-------------------|-------|
| Media             | 8     |
| Cuarta            | 8     |
| Octava            | 4     |
| Dieciseisava      | 2     |
| Treintaidosava    | 0     |
| Sesentaicuatroava | 0     |

Si la misma nota se repite de forma consecutiva, el nivel del visualizador se mantiene, lo que permite representar duraciones compuestas como Cuarta + Dieciseisava sin perder la continuidad visual.

---
## ⚙️ Limitaciones
| Parámetro                           | Límite                    |
|-------------------------------------|---------------------------|
| Notas musicales                     | 32                        |
| Secciones del patrón                | 256                       |
| Barras del visualizador             | 8                         |
| Orden del visualizador              | Por declaración           |
---
## 🛠️ Recursos
- 🖥️ **Microchip Studio** — IDE para escribir y compilar el código AVR Assembly. Se usa para generar el `.hex` que se carga en el simulador.
- ⚡ **SimulIDE** — Simulador de circuitos electrónicos donde se monta y prueba el escenario. Se recomienda usar la versión **0.4.15**, incluida en el proyecto en formato `.zip`, ya que es la versión con soporte correcto para el ATmega16.
- 📁 **`Canciones/`** — Directorio de canciones listas para copiar en formato `.txt`.
- 📊 *`Compresor.xlsx`* — Hoja de cálculo para definir el conjunto de notas del proyecto. Incluye sus frecuencias, los valores OCR calculados para el timer y la codificación comprimida lista para usar en el programa.
