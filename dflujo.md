```mermaid
flowchart TD
    A[INICIO] --> B[Inicializar MonitorTemperaturaRPI<br>- Configurar duración, intervalo, archivo CSV<br>- Configurar gráfica]
    B --> C[Abrir archivo CSV<br>- Verificar si está vacío<br>- Escribir cabecera si es necesario]
    C --> D{¿Existen condiciones<br>para seguir monitoreando?}
    D -- Sí --> E[Actualizar datos:<br>- Leer temperatura<br>- Agregar tiempo<br>- Guardar en CSV<br>- Limpiar datos antiguos si exceden duración]
    E --> F[Graficar datos en tiempo real<br>- Actualizar gráfica]
    F --> G{¿Interrupción de teclado?}
    G -- No --> D
    G -- Sí --> H[Finalizar ejecución<br>- Apagar modo interactivo<br>- Cerrar gráfica]
    D -- No --> H
    H --> I[FIN]
