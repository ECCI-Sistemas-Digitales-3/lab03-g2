# Monitor de Temperatura en Raspberry Pi

Este script permite monitorear la temperatura de la CPU en una Raspberry Pi en tiempo real, visualizarla en una gráfica interactiva y almacenar los datos en un archivo CSV para su posterior análisis.

---

## Importaciones

```python
import matplotlib.pyplot as plt
import time
import subprocess
import csv
```

1. `matplotlib.pyplot as plt`: Se utiliza para generar gráficos y visualizaciones interactivas.
2. `time`: Proporciona funciones para medir el tiempo transcurrido y hacer pausas.
3. `subprocess`: Permite ejecutar comandos del sistema operativo desde Python.
4. `csv`: Facilita la escritura de datos en formato CSV.

---

## Clase MonitorTemperaturaRPI
La clase `MonitorTemperaturaRPI` encapsula toda la lógica del monitoreo: configuración, lectura de datos, guardado, graficación y control del flujo.

### 🔧 Constructor (`__init__`)
```python
def __init__(self, duracion_max=60, intervalo=0.5, archivo_salida="temperaturas.csv"):
    ...
```

**Parámetros:**
- `duracion_max`: Tiempo máximo que se mantiene visible en la gráfica.
- `intervalo`: Intervalo de muestreo en segundos.
- `archivo_salida`: Ruta y nombre del archivo CSV donde se guardan los datos.

**Inicialización:**
- Se definen listas para tiempo y temperatura.
- Se guarda la hora de inicio.
- Se crea una gráfica interactiva con Matplotlib.
- Se abre (o crea) el archivo CSV y se escribe la cabecera si es necesario.

---

### Método `leer_temperatura()`
```python
def leer_temperatura(self):
    ...
```

**Funcionamiento:**
- Ejecuta el comando `vcgencmd measure_temp`.
- Extrae y convierte la salida a un número flotante.
- Devuelve la temperatura actual de la CPU.

**Control de errores:**
- Captura cualquier excepción que ocurra y muestra un mensaje si hay errores de lectura.

---

### Método `guardar_en_csv()`
```python
def guardar_en_csv(self, tiempo, temperatura):
    ...
```

Guarda el tiempo y la temperatura en una fila del archivo CSV. Se abre en modo “append” para no sobreescribir los datos anteriores.

---

### Método `actualizar_datos()`
```python
def actualizar_datos(self):
    ...
```

- Calcula el tiempo relativo desde el inicio del monitoreo.
- Lee la temperatura actual.
- Agrega los datos a las listas y los guarda en el CSV.
- Elimina datos antiguos si exceden la duración máxima permitida en la gráfica.

---

### Método `graficar()`
```python
def graficar(self):
    ...
```

- Limpia la gráfica actual.
- Traza la curva de temperatura.
- Añade etiquetas, título y cuadrícula.
- Dibuja y actualiza en tiempo real usando Matplotlib interactivo.

---

### Método `ejecutar()`
```python
def ejecutar(self):
    ...
```

- Inicia un bucle que continúa mientras la ventana de la gráfica esté abierta.
- Actualiza los datos y redibuja la gráfica en cada iteración.
- Se detiene si se cierra la ventana o el usuario presiona `Ctrl+C`.
- Al final, apaga el modo interactivo y cierra la gráfica.

---

## Ejecución del Script
```python
if __name__ == "__main__":
    monitor = MonitorTemperaturaRPI()
    monitor.ejecutar()
```

Este bloque garantiza que el script se ejecute únicamente si es el archivo principal. Crea una instancia del monitor y llama al método `ejecutar()` para iniciar el monitoreo.

---

## Preguntas Clave

**¿Qué función cumple `plt.fignum_exists(self.fig.number)` en el ciclo principal?**
- Verifica si la figura de la gráfica aún existe (no ha sido cerrada por el usuario). Permite detener el bucle adecuadamente.

**¿Por qué se usa `time.sleep(self.intervalo)` y qué pasa si se quita?**
- Establece una pausa entre lecturas. Si se omite, el script consumirá muchos recursos al ejecutarse sin descanso.

**¿Qué ventaja tiene usar `__init__` para inicializar listas y variables?**
- Agrupa y organiza la inicialización de todos los atributos de la clase. Mejora la legibilidad y reutilización del código.

**¿Qué se está midiendo con `self.inicio = time.time()`?**
- El momento exacto en que empieza el monitoreo. Se usa para calcular el tiempo relativo transcurrido.

**¿Qué hace exactamente `subprocess.check_output(...)`?**
- Ejecuta un comando del sistema (en este caso, para obtener la temperatura) y devuelve su salida como texto.

**¿Por qué se almacena `ahora = time.time() - self.inicio` en lugar del tiempo absoluto?**
- Para mostrar un tiempo relativo fácil de interpretar en la gráfica (desde que comenzó el monitoreo).

**¿Por qué se usa `self.ax.clear()` antes de graficar?**
- Para evitar que la gráfica se sobrecargue con múltiples líneas superpuestas.

**¿Qué captura el bloque `try...except` dentro de `leer_temperatura()`?**
- Cualquier error al ejecutar el comando o procesar su salida. Esto previene que el programa se detenga inesperadamente.

**¿Cómo modificar el script para guardar temperaturas en un CSV?**
- Ya está implementado en el método `guardar_en_csv()`.

---
