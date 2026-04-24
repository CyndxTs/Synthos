# 🎹 Synthos

Synthos es un escenario musical embebido para el **ATmega16**. Carga una canción, la reproduce y la muestra en tiempo real sobre una matriz LED. Todo en AVR Assembly, sin librerías externas.

---

## 🎬 Puesta en Escena

El proyecto se desarrolla en **Microchip Studio** y se simula en **SimulIDE**.

1. Compila el proyecto en Microchip Studio y carga el `.hex` al escenario en SimulIDE.
2. Inicia la simulación y presiona cualquier tecla (`A`, `S`, `D`, `F`...) para activar `pinA` e iniciar la canción.
3. La canción comienza a sonar y las notas se visualizan en tiempo real en el panel LED.
4. Al terminar, el sistema queda en espera — presiona nuevamente para repetir.

---

## 📊 Visualizador de Notas

El visualizador divide las notas en **8 barras**. El tamaño de cada grupo se calcula tomando el múltiplo de 8 más cercano por encima del total de notas, agrupándolas en el orden en que fueron declaradas, no por frecuencia.

**Ejemplo:** 21 notas → múltiplo superior = 24 → cada barra agrupa **3 notas**.

La altura de cada barra depende de la duración de la nota, llegando al máximo con una Cuarta. Si la misma nota se repite consecutivamente, el nivel se mantiene, lo que permite representar duraciones compuestas como `Cuarta + Dieciseisava`.

| Duración     | Nivel     |
|--------------|-----------|
| Cuarta       | Máximo    |
| Octava       | Mitad     |
| Dieciseisava | Bajo      |

---

## 🗂️ Seccionamiento Musical

En vez de escribir cada nota seguida, las canciones se agrupan en **secciones reutilizables**, lo que reduce el código en aproximadamente un **70%**.

Cada nota de sección ocupa 1 byte: los **5 bits superiores** indican la posición de la nota y los **3 bits inferiores** su duración, dando un límite de **32 notas musicales**. Estas no tienen que ser contiguas en el piano, por lo que pueden cubrir más de 4 octavas sin problema.

```
[ 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 ]
\______5 bits______/ \__3 bits__/
      Posición         Duración
```

| Bits    | Qué indica                              |
|---------|-----------------------------------------|
| `[7:3]` | Posición de la nota (máx. 32 notas)     |
| `[2:0]` | Duración de la nota (8 valores posibles)|

---

## 🎼 Patrón Musical

El patrón define en qué orden se tocan las secciones, con un máximo de **256 entradas** en RAM. Cada entrada tiene dos datos:

| Campo      | Qué indica                                   |
|------------|----------------------------------------------|
| Posición   | Desde qué nota de la sección se empieza      |
| Duración   | Cuántas notas se tocan desde ahí (inclusive) |

---

## 🎶 Canciones Extra

Las canciones disponibles están en la carpeta **`MDX/`**, listas para copiar y pegar en el `main`.

En el repositorio también hay un **Excel** con el mapeo de notas, frecuencias y valores OCR1, útil para componer o adaptar canciones al sistema.

---

## ⚙️ Limitaciones

| Parámetro                          | Límite        |
|------------------------------------|---------------|
| Notas musicales máximas            | 32            |
| Entradas en el patrón              | 256           |
| Barras del visualizador            | 8             |
| Duración que satura el nivel       | Cuarta        |
| Análisis de frecuencias automático | No soportado  |

---

*By: Argedia & CyndxTs*
