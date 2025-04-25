1. ¿Qué función cumple plt.fignum_exists(self.fig.number) en el ciclo principal?

plt.fignum_exists(self.fig.number) se usa en el ciclo principal para verificar si la ventana de la figura de matplotlib sigue abierta. Si la figura ha sido cerrada por el usuario, este método devuelve False y el ciclo se detiene. Esto asegura que el programa siga ejecutándose y actualizando la gráfica solo mientras la ventana de la gráfica esté activa. Si la ventana se cierra, el ciclo termina y se finaliza el monitoreo de la temperatura.

2. ¿Por qué se usa time.sleep(self.intervalo) y qué pasa si se quita?

El método time.sleep(self.intervalo) se usa para pausar la ejecución del ciclo durante un tiempo específico, definido por el valor de self.intervalo. Esto permite que la lectura de la temperatura y la actualización de la gráfica no se realicen de forma continua, sino que haya un intervalo de tiempo entre cada iteración (en este caso, en segundos). Si se quita, el programa trataría de leer la temperatura y actualizar la gráfica lo más rápido posible, lo que podría hacer que el programa consuma demasiados recursos del sistema y, posiblemente, cause un sobrecarga o un comportamiento errático debido a la falta de una pausa entre las actualizaciones.

3. ¿Qué ventaja tiene usar __init__ para inicializar listas y variables?

El uso de __init__ para inicializar listas y variables dentro de la clase tiene varias ventajas. Principalmente, asegura que las variables estén configuradas correctamente cada vez que se crea una nueva instancia del objeto. Además, la inicialización en __init__ proporciona un punto centralizado donde se pueden definir los valores predeterminados para los atributos de la clase. Esto hace que el código sea más modular y reutilizable, ya que cada vez que se crea un nuevo objeto, se garantiza que comience con los mismos valores de inicialización.

4. ¿Qué se está midiendo con self.inicio = time.time()?

self.inicio = time.time() mide el tiempo actual en segundos desde el "epoch" (el 1 de enero de 1970, 00:00:00 UTC). Este valor se usa para marcar el momento en que comienza la ejecución del monitoreo. Al restar self.inicio del tiempo actual en cada actualización, se obtiene el tiempo transcurrido desde que se inició el monitoreo. Este valor de tiempo transcurrido es lo que se utiliza para actualizar la gráfica y también para gestionar la duración de los datos que se almacenan.

5. ¿Qué hace exactamente subprocess.check_output(...)?

subprocess.check_output(...) ejecuta un comando en el sistema operativo y captura su salida estándar. En este caso, se está ejecutando el comando vcgencmd measure_temp, que obtiene la temperatura de la CPU en un Raspberry Pi. El comando se ejecuta en el sistema como si fuera un proceso de línea de comandos y la salida (el texto con la temperatura) se captura. La función devuelve esta salida como un string, que luego se procesa (con decode("utf-8") y strip()) para extraer la temperatura en formato numérico.

6. ¿Por qué se almacena ahora = time.time() - self.inicio en lugar del tiempo absoluto?

Se almacena ahora = time.time() - self.inicio para obtener el tiempo transcurrido desde el inicio del monitoreo. Si se usara el tiempo absoluto, el valor dependería del momento específico en que se ejecute el programa y no reflejaría el tiempo que ha pasado desde el comienzo del monitoreo. Al almacenar el tiempo transcurrido, podemos tener una representación consistente del tiempo en la gráfica, que siempre comienza desde cero y avanza a medida que pasa el tiempo.

7. ¿Por qué se usa self.ax.clear() antes de graficar?

self.ax.clear() se utiliza para borrar el contenido anterior de la gráfica antes de volver a dibujarla. Dado que el programa está actualizando la gráfica en tiempo real, esta función asegura que cada vez que se grafique la nueva información, la visualización previa se elimine. Si no se hiciera esto, las nuevas líneas de la gráfica se superpondrían a las anteriores, haciendo que la visualización sea confusa y desordenada.

8. ¿Qué captura el bloque try...except dentro de leer_temperatura()?

El bloque try...except captura posibles excepciones que podrían ocurrir al intentar ejecutar el comando vcgencmd. Por ejemplo, si el comando no está disponible o si hay problemas de permisos, el bloque except manejaría estos errores, evitando que el programa se bloquee o termine inesperadamente. En este caso, se captura cualquier error relacionado con la ejecución del comando y se muestra un mensaje de error en la consola.

9. ¿Cómo podría modificar el script para guardar las temperaturas en un archivo .csv?

Para guardar las temperaturas en un archivo .csv, puedes agregar código dentro del método actualizar_datos() para escribir los datos en el archivo.


```python
import matplotlib.pyplot as plt
import time
import subprocess
import csv

class MonitorTemperaturaRPI:
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

    def leer_temperatura(self):
        try:
            salida = subprocess.check_output(["vcgencmd", "measure_temp"]).decode("utf-8")
            temp_str = salida.strip().replace("temp=", "").replace("'C", "")
            return float(temp_str)
        except Exception as e:
            print("Error leyendo temperatura:", e)
            return None

    def guardar_en_csv(self, tiempo, temperatura):
        # Guardar los datos en el archivo CSV
        with open(self.archivo_salida, mode='a', newline='') as file:
            writer = csv.writer(file)
            writer.writerow([tiempo, temperatura])

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

    def graficar(self):
        self.ax.clear()  # Limpiar la gráfica
        self.ax.plot(self.tiempos, self.temperaturas, color='red')  # Graficar la temperatura
        self.ax.set_title("Temperatura CPU Raspberry Pi")
        self.ax.set_xlabel("Tiempo transcurrido (s)")
        self.ax.set_ylabel("Temperatura (°C)")
        self.ax.grid(True)
        self.fig.canvas.draw()  # Dibujar la nueva gráfica
        self.fig.canvas.flush_events()  # Actualizar la figura en la pantalla

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


if __name__ == "__main__":
    # Crear una instancia del monitor de temperatura
    monitor = MonitorTemperaturaRPI()
    monitor.ejecutar()  # Ejecutar el monitoreo
