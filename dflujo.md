```mermaid
flowchart TD
    A[Inicio del programa] --> B[Inicializar objeto MonitorTemperatura<br>- Definir intervalo y duración<br>- Preparar archivo CSV<br>- Configurar gráfica]
    B --> C[Crear archivo CSV si no existe<br>- Escribir encabezado si está vacío]
    C --> D[Registrar hora de inicio time.time]
    D --> E{¿Ventana de gráfica abierta<br>y sin interrupción del usuario?}
    E -- Sí --> F[Leer sensor de temperatura<br>- Manejo de errores con try...except]
    F --> G[Calcular tiempo transcurrido<br>desde inicio]
    G --> H[Registrar datos:<br>- Agregar a lista<br>- Guardar en CSV]
    H --> I[Actualizar gráfica:<br>- Limpiar ejes<br>- Graficar temperatura vs tiempo]
    I --> J[Esperar intervalo definido sleep]
    J --> E
    E -- No --> K[Finalizar monitoreo<br>- Cerrar figura<br>- Terminar script]
    K --> L[Fin del programa]

