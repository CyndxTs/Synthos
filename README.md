# 🎹 Synthos

Synthos es un escenario musical embebido para el ATmega16. Carga una canción, la reproduce y la muestra en tiempo real sobre una matriz LED. Todo en AVR Assembly, sin librerías externas.

1. Abre SimulIDE y carga el archivo de escenario `.simu` del proyecto.
2. Presiona cualquiera de las teclas conectadas al pin A (`A`, `S`, `D`, `F`, `G`, `H`, `J`, `K`) para activar el sistema e iniciar la canción.
3. La canción comienza a sonar y las notas se visualizan en tiempo real en el panel LED.
4. Al terminar, el sistema queda en espera. Presiona nuevamente para repetir.

---

## 🗂️ Seccionamiento Musical

En lugar de escribir cada nota de forma individual, las canciones se organizan en secciones reutilizables. Esto permite estructurar patrones repetidos sin duplicar código, reduciendo el tamaño del programa en aproximadamente un 70%.

Cada nota dentro de una sección se codifica en un solo byte: los 5 bits superiores almacenan la posición de la nota dentro del conjunto declarado, y los 3 bits inferiores almacenan su duración. Esto da un límite de 32 notas musicales distintas por proyecto, aunque no es necesario que sean contiguas en el piano, por lo que pueden abarcar más de 4 octavas sin problema.

```
[ 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 ]
\______5 bits______/ \__3 bits__/
      Posición         Duración
```

Los 3 bits de duración permiten representar 8 valores, desde la semifusa (sesentaycuatroava) hasta la media, siendo la cuarta la duración que satura el visualizador al nivel máximo. Las duraciones más cortas producen niveles proporcionalmente menores.

Si la misma nota se repite de forma consecutiva, el nivel del visualizador se mantiene, lo que permite representar duraciones compuestas como `Cuarta + Dieciseisava` sin perder la continuidad visual.

---

## 🎵 Patrón Musical

Luego del seccionamiento, el programa necesita saber de qué manera se tocarán las secciones. Por ello se define como 'patrón' a la secuencia de secciones a reproducir.

---

## 📊 Visualizador de Notas

El visualizador representa la actividad musical en tiempo real mediante 8 barras verticales sobre la matriz LED. Cada barra corresponde a un grupo de notas del conjunto declarado, agrupadas en el orden en que fueron definidas, no por frecuencia ni altura.

El tamaño de cada grupo se determina tomando el múltiplo de 8 inmediatamente superior al total de notas del proyecto y dividiendo ese valor entre 8.

```
Ejemplo: 21 notas → múltiplo superior más cercano = 24 → cada barra agrupa 3 notas
```

La altura de cada barra refleja la duración de la nota activa. Una cuarta lleva la barra al nivel máximo; duraciones más cortas producen niveles proporcionalmente menores. Cuando la misma nota se repite consecutivamente, el nivel se sostiene, permitiendo visualizar duraciones compuestas sin interrupciones.

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
- 📁 **`Canciones/`** — Canciones listas para copiar en formato `.txt`.
- 📊 *`Compresor.xlsx`* — Hoja de cálculo con las notas, sus frecuencias y parámetros de configuración del timer. Útil para definir el conjunto de notas a usar, y calcular sus valores comprimidos.
