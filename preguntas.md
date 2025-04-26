1. ¿Qué función cumple plt.fignum_exists(self.fig.number) en el ciclo principal?

La función plt.fignum_exists(self.fig.number) verifica si la figura identificada por self.fig.number aún existe dentro del entorno de matplotlib.
Función en el ciclo principal: Permite condicionar la ejecución del bucle a la existencia de la ventana gráfica. Si el usuario la cierra, la función retorna False, provocando la salida controlada del ciclo de adquisición y visualización de datos.

2. ¿Por qué se usa time.sleep(self.intervalo) y qué pasa si se quita?

El método time.sleep(self.intervalo) introduce una pausa de duración definida (self.intervalo, en segundos) en cada iteración del bucle principal.
- Propósito:
Controla la frecuencia de muestreo, evita lecturas redundantes y actualizaciones excesivas del gráfico y reduce el uso innecesario del CPU.
- Sin esta instrucción:
El bucle se ejecutaría de forma continua y rápida, provocando, un alto consumo de recursos del sistema, saturación de la interfaz gráfica y posible bloqueo del sistema operativo o errores por exceso de lecturas.

3. ¿Qué ventaja tiene usar __init__ para inicializar listas y variables?

El método especial __init__ en Python es el constructor de una clase que permite establecer un estado inicial coherente del objeto, facilita la modularidad y reutilización del código, mejora la legibilidad y la mantenibilidad, al centralizar la inicialización de atributos (como listas, contadores, objetos Figure y Axes, etc.), evita errores por referencias a variables no definidas.

4. ¿Qué se está midiendo con self.inicio = time.time()?

Se registra el tiempo absoluto en segundos desde "epoch" UNIX (01/01/1970) justo al inicio del programa o del proceso de monitoreo. Permite calcular el tiempo transcurrido desde ese punto de referencia, como ahora = time.time() - self.inicio, este valor de tiempo transcurrido es lo que se utiliza para actualizar.

5. ¿Qué hace exactamente subprocess.check_output(...)?

Ejecuta un comando externo en una subrutina del sistema operativo y retorna su salida estándar (stdout) como una cadena de bytes.
Ejemplo de uso típico: Leer la temperatura de un sensor desde un script o comando del sistema (e.g., cat /sys/class/thermal/...).
Esto presenta diferentes vnetajas como: Integra scripts de shell o comandos específicos de hardware y permite mantener la portabilidad o utilizar herramientas ya existentes.

6. ¿Por qué se almacena ahora = time.time() - self.inicio en lugar del tiempo absoluto?

Se calcula un tiempo relativo transcurrido desde el inicio de la medición, en lugar de usar una marca de tiempo absoluta; esto mejora la interpretación de los datos, facilita la visualización de la evolución temporal en gráficos, evita valores de tiempo absolutos que podrían ser difíciles de representar gráficamente.

7. ¿Por qué se usa self.ax.clear() antes de graficar?

El método self.ax.clear() limpia todos los elementos gráficos actuales del eje antes de dibujar nuevos datos.
De esta forma evita la superposición de múltiples curvas o textos, asegura que cada gráfico represente únicamente los datos actuales y previene errores visuales o acumulación de elementos innecesarios.

8. ¿Qué captura el bloque try...except dentro de leer_temperatura()?

Este bloque intercepta cualquier excepción que pueda producirse al intentar leer la temperatura, típicamente mediante subprocess.check_output(...); esto identifica y registra una falla controlada y permite que el programa continúe funcionando sin detener la ejecución, identificando diferentes errores como:
- Archivo no encontrado.
- Sensor desconectado.
- Error de permisos o de ejecución del comando.

9. ¿Cómo podría modificar el script para guardar las temperaturas en un archivo .csv?

Para guardar las temperaturas en un archivo .csv, se puede incorporar una función como esta dentro de tu clase.

```python
import csv
from datetime import datetime

def guardar_csv(self, tiempo, temperatura):
    with open("datos_temperatura.csv", mode='a', newline='') as archivo:
        escritor = csv.writer(archivo)
        escritor.writerow([datetime.now().isoformat(), tiempo, temperatura])
```
Luego realizar el llamado con:
```python
self.guardar_csv(ahora, temperatura)
```
Esto permite un registro persistente con marcas de tiempo y facilidad de análisis posterior con Excel, Python (pandas), MATLAB, etc.

```python
import matplotlib.pyplot as plt
import time
import csv
import random

class MonitorTemperatura:
    def __init__(self, intervalo=1.0):
        self.intervalo = intervalo
        self.tiempos = []
        self.temperaturas = []
        self.inicio = time.time()

        self.fig, self.ax = plt.subplots()
        self.ax.set_title("Temperatura en tiempo real")
        self.ax.set_xlabel("Tiempo [s]")
        self.ax.set_ylabel("Temperatura [°C]")

        with open("datos_temperatura.csv", mode='w', newline='') as archivo:
            escritor = csv.writer(archivo)
            escritor.writerow(["timestamp", "tiempo_s", "temperatura_C"])

    def leer_temperatura(self):
        try:
            # Simulación, reemplaza con subprocess si lo necesitas
            temperatura = 20 + 5 * random.random()
            return temperatura
        except Exception as e:
            print("Error al leer temperatura:", e)
            return None

    def guardar_csv(self, tiempo, temperatura):
        with open("datos_temperatura.csv", mode='a', newline='') as archivo:
            escritor = csv.writer(archivo)
            escritor.writerow([time.strftime("%Y-%m-%d %H:%M:%S"), tiempo, temperatura])

    def graficar(self):
        while plt.fignum_exists(self.fig.number):
            temperatura = self.leer_temperatura()
            ahora = time.time() - self.inicio

            if temperatura is not None:
                self.tiempos.append(ahora)
                self.temperaturas.append(temperatura)
                self.guardar_csv(ahora, temperatura)

                self.ax.clear()
                self.ax.plot(self.tiempos, self.temperaturas, color='blue')
                self.ax.set_xlabel("Tiempo [s]")
                self.ax.set_ylabel("Temperatura [°C]")
                self.ax.set_title("Temperatura en tiempo real")
                plt.pause(0.01)

            time.sleep(self.intervalo)

if __name__ == "__main__":
    monitor = MonitorTemperatura(intervalo=2.0)  # cada 2 segundos
    monitor.graficar()
