# Importaciones

```python
import matplotlib.pyplot as plt
import time
import subprocess
import csv
```
1. matplotlib.pyplot as plt: Importa la librería matplotlib.pyplot, que se usa para generar gráficos y visualizaciones.

2. time: Importa el módulo time, que proporciona funciones para trabajar con tiempos, como esperar una cierta cantidad de segundos o medir el tiempo transcurrido.

3. subprocess: Importa el módulo subprocess, que permite ejecutar comandos del sistema operativo desde el código Python. En este caso, se utiliza para ejecutar el comando que obtiene la temperatura de la Raspberry Pi.

4. csv: Importa el módulo csv que facilita la lectura y escritura de archivos CSV, que es el formato donde se guardarán los datos de temperatura.

# Clase MonitorTemperaturaRPI
La clase MonitorTemperaturaRPI gestiona todo el proceso de monitoreo de la temperatura de la Raspberry Pi, almacenando los datos en un archivo CSV y mostrando un gráfico interactivo.

Constructor (__init__)

```python
def __init__(self, duracion_max=60, intervalo=0.5, archivo_salida="temperaturas.csv"):
    self.duracion_max = duracion_max
    self.intervalo = intervalo
    self.tiempos = []
    self.temperaturas = []
    self.inicio = time.time()
    self.archivo_salida = archivo_salida

    # Inicialización de la gráfica en modo interactivo
    plt.ion()
    self.fig, self.ax = plt.subplots()

    # Abrir el archivo CSV en modo append y escribir la cabecera si es necesario
    with open(self.archivo_salida, mode='a', newline='') as file:
        writer = csv.writer(file)
        if file.tell() == 0:  # Solo escribir cabecera si el archivo está vacío
            writer.writerow(["Tiempo (s)", "Temperatura (°C)"])

```

1. def __init__(self, duracion_max=60, intervalo=0.5, archivo_salida="temperaturas.csv"):: Es el constructor de la clase. Recibe tres parámetros:

* duracion_max: La duración máxima de tiempo (en segundos) que se mostrará en la gráfica. Por defecto es 60 segundos.

* intervalo: El intervalo (en segundos) entre cada lectura de temperatura. Por defecto es 0.5 segundos.

* archivo_salida: El nombre del archivo CSV donde se almacenarán los datos de temperatura. Por defecto es "temperaturas.csv".

2. self.duracion_max = duracion_max: Asigna el valor de duracion_max al atributo de la instancia.

3. self.intervalo = intervalo: Asigna el valor de intervalo al atributo de la instancia.

4. self.tiempos = []: Inicializa una lista vacía que almacenará los tiempos (en segundos) cuando se registre la temperatura.

5. self.temperaturas = []: Inicializa una lista vacía que almacenará las temperaturas leídas.

6. self.inicio = time.time(): Guarda el tiempo de inicio, es decir, el momento exacto en que el monitoreo empieza.

7. self.archivo_salida = archivo_salida: Asigna el nombre del archivo CSV de salida a un atributo de la clase.

8. plt.ion(): Activa el modo interactivo de Matplotlib. Esto permite que el gráfico se actualice en tiempo real sin necesidad de bloquear el programa.

9. self.fig, self.ax = plt.subplots(): Crea una figura y un eje para el gráfico, utilizando Matplotlib.

10. with open(self.archivo_salida, mode='a', newline='') as file:: Abre el archivo CSV en modo de "añadir" ('a') para que los datos no sobreescriban los anteriores. newline='' asegura que las líneas se escriban correctamente en el archivo CSV.

11. writer = csv.writer(file): Crea un objeto csv.writer que se usa para escribir en el archivo.

12. if file.tell() == 0:: Verifica si el archivo está vacío. file.tell() devuelve la posición del cursor en el archivo. Si es 0, significa que el archivo está vacío.

13. writer.writerow(["Tiempo (s)", "Temperatura (°C)"]): Si el archivo está vacío, escribe una fila con los encabezados de las columnas: "Tiempo (s)" y "Temperatura (°C)".

# Método leer_temperatura

```python
def leer_temperatura(self):
    try:
        salida = subprocess.check_output(["vcgencmd", "measure_temp"]).decode("utf-8")
        temp_str = salida.strip().replace("temp=", "").replace("'C", "")
        return float(temp_str)
    except Exception as e:
        print("Error leyendo temperatura:", e)
        return None
```
1. subprocess.check_output(["vcgencmd", "measure_temp"]): Ejecuta el comando vcgencmd measure_temp en la Raspberry Pi. Este comando devuelve la temperatura de la CPU.

2. .decode("utf-8"): Convierte la salida del comando de bytes a una cadena de texto en formato UTF-8.

3. .strip(): Elimina los espacios en blanco al principio y al final de la cadena.

4. .replace("temp=", "").replace("'C", ""): Elimina las cadenas "temp=" y "C" para dejar solo el valor numérico de la temperatura.

5. return float(temp_str): Convierte el valor de la temperatura en una cadena a un número flotante y lo devuelve.

Si ocurre algún error durante este proceso (por ejemplo, si el comando falla), el except lo captura y muestra un mensaje de error.

# Método guardar_en_csv

```python
def guardar_en_csv(self, tiempo, temperatura):
    with open(self.archivo_salida, mode='a', newline='') as file:
        writer = csv.writer(file)
        writer.writerow([tiempo, temperatura])
```
1. open(self.archivo_salida, mode='a', newline=''): Abre el archivo CSV en modo "añadir" para agregar nuevos datos al final del archivo.

2. csv.writer(file): Crea un escritor CSV para agregar los datos al archivo.

3. writer.writerow([tiempo, temperatura]): Escribe una nueva fila con el tiempo y la temperatura que se pasan como parámetros.

# Método actualizar_datos

```python
def actualizar_datos(self):
    ahora = time.time() - self.inicio
    temp = self.leer_temperatura()
    if temp is not None:
        self.tiempos.append(ahora)
        self.temperaturas.append(temp)

        # Guardar la nueva temperatura en el archivo CSV
        self.guardar_en_csv(ahora, temp)

        # Eliminar datos que superen el tiempo máximo
        while self.tiempos and self.tiempos[0] < ahora - self.duracion_max:
            self.tiempos.pop(0)
            self.temperaturas.pop(0)
```

1. ahora = time.time() - self.inicio: Calcula el tiempo transcurrido desde el inicio del monitoreo.

2. temp = self.leer_temperatura(): Obtiene la temperatura actual de la CPU.

3. if temp is not None:: Si se ha obtenido una temperatura válida (no es None), se proceden a realizar las siguientes acciones:

* self.tiempos.append(ahora): Añade el tiempo actual a la lista de tiempos.

* self.temperaturas.append(temp): Añade la temperatura actual a la lista de temperaturas.

* self.guardar_en_csv(ahora, temp): Guarda los datos en el archivo CSV.

4. while self.tiempos and self.tiempos[0] < ahora - self.duracion_max:: Si los tiempos almacenados superan la duración máxima (duracion_max), elimina los datos más antiguos.

* self.tiempos.pop(0): Elimina el primer valor (el más antiguo) de la lista de tiempos.

* self.temperaturas.pop(0): Elimina el primer valor (el más antiguo) de la lista de temperaturas.

# Método graficar

```python
def graficar(self):
    self.ax.clear()  # Limpiar la gráfica
    self.ax.plot(self.tiempos, self.temperaturas, color='red')  # Graficar la temperatura
    self.ax.set_title("Temperatura CPU Raspberry Pi")
    self.ax.set_xlabel("Tiempo transcurrido (s)")
    self.ax.set_ylabel("Temperatura (°C)") self.ax.grid(True) self.fig.canvas.draw() # Dibujar la nueva gráfica self.fig.canvas.flush_events() # Actualizar la figura en la pantalla
```
1. self.ax.clear(): Limpia la gráfica anterior.
2. self.ax.plot(self.tiempos, self.temperaturas, color='red'): Grafica los datos de tiempo y temperatura con una línea roja.
3. self.ax.set_title("Temperatura CPU Raspberry Pi"): Establece el título del gráfico.
4. self.ax.set_xlabel("Tiempo transcurrido (s)"): Establece la etiqueta del eje X (tiempo).
5. self.ax.set_ylabel("Temperatura (°C)"): Establece la etiqueta del eje Y (temperatura).
6. self.ax.grid(True): Activa la cuadrícula en la gráfica.
7. self.fig.canvas.draw(): Dibuja la nueva gráfica.
8. self.fig.canvas.flush_events(): Actualiza la gráfica en la pantalla en modo interactivo.

#Método `ejecutar

```python
def ejecutar(self):
    try:
        while plt.fignum_exists(self.fig.number):
            self.actualizar_datos()  # Actualizar los datos de temperatura
            self.graficar()  # Graficar los datos
            time.sleep(self.intervalo)  # Esperar antes de la siguiente actualización

    except KeyboardInterrupt:
        print("Monitoreo interrumpido por el usuario.")

    finally:
        print("Monitoreo finalizado.")
        plt.ioff()  # Apagar el modo interactivo de matplotlib
        plt.close(self.fig)  # Cerrar la figura
```

1. while plt.fignum_exists(self.fig.number):: Inicia un bucle que continúa hasta que la figura de Matplotlib sea cerrada por el usuario.

2. self.actualizar_datos(): Actualiza los datos de tiempo y temperatura.

3. self.graficar(): Grafica los datos actualizados.

4. time.sleep(self.intervalo): Hace una pausa durante el tiempo especificado por el intervalo antes de continuar.

5. except KeyboardInterrupt:: Captura una interrupción manual (cuando el usuario presiona Ctrl+C) y muestra un mensaje indicando que el monitoreo se ha interrumpido.

6. finally:: Al finalizar el monitoreo, apaga el modo interactivo de Matplotlib y cierra la ventana del gráfico.

# Parte final del código
```python
if __name__ == "__main__":
    # Crear una instancia del monitor de temperatura
    monitor = MonitorTemperaturaRPI()
    monitor.ejecutar()  # Ejecutar el monitoreo
```
1. if __name__ == "__main__":: Esta línea asegura que el código se ejecute solo cuando se ejecuta directamente el archivo (no cuando se importa como módulo).

2. monitor = MonitorTemperaturaRPI(): Crea una instancia de la clase MonitorTemperaturaRPI.

3. monitor.ejecutar(): Ejecuta el monitoreo de la temperatura.
