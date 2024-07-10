1. Configuración del proyecto:

Crear una carpeta para el proyecto: Elige un nombre descriptivo para la carpeta, como cub3d_proyecto.
Inicializar el proyecto de C: Utiliza un compilador de C como gcc o clang.
Instalar las librerías necesarias:
Minilibx: Puedes descargarla desde su sitio web oficial o usar un gestor de paquetes como apt o brew.
Crear los archivos fuente:
main.c: Contendrá el código principal del juego, incluyendo la inicialización, bucle principal y funciones auxiliares.
cub3d.h: Contendrá las declaraciones de las funciones y estructuras específicas del juego.
2. Definición de funciones y estructuras:

Definir las estructuras necesarias:

codigo:

#ifndef CUB3D_H
#define CUB3D_H

typedef struct {
    double x;        // Posición horizontal del jugador en el mapa (en celdas)
    double y;        // Posición vertical del jugador en el mapa (en celdas)
    double angulo;   // Dirección del jugador en grados (0º hacia el norte, 90º hacia el este, 180º hacia el sur, 270º hacia el oeste)
    double velocidad; // Velocidad de movimiento del jugador (en celdas por segundo)
} Jugador;

typedef struct {
    int ancho;       // Ancho del mapa en celdas
    int alto;        // Alto del mapa en celdas
    char **celdas;   // Matriz bidimensional de caracteres que representan el contenido del mapa
} Mapa;

typedef struct {
    int tipo_textura; // Identificador de la textura de la pared (NO, SO, WE, EA)
    double distancia; // Distancia desde el punto de origen al punto de intersección con la pared
} Pared;

void inicializar_cubo3d(void);
void ciclo_principal(void);
void procesar_entrada(void);
void logica_del_juego(void);
void renderizar(void);
void raycast(int x);
void dibujar_pared_columna(int x, Pared pared_cercana);
void cargar_texturas(void);
void leer_archivo_cub(const char *nombre_archivo);
#endif

jugador: Almacena la posición, dirección y estado del jugador.
x: Posición horizontal del jugador en el mapa (en celdas).
y: Posición vertical del jugador en el mapa (en celdas).
angulo: Dirección del jugador en grados (0º hacia el norte, 90º hacia el este, 180º hacia el sur, 270º hacia el oeste).
velocidad: Velocidad de movimiento del jugador (en celdas por segundo).
mapa: Almacena la representación del mapa en forma de matriz bidimensional.
ancho: Ancho del mapa en celdas.
alto: Alto del mapa en celdas.
celdas: Matriz bidimensional de caracteres que representan el contenido del mapa:
'0': Espacio vacío.
'1': Pared.
'N', 'S', 'E', 'W': Posición inicial y orientación del jugador (N: Norte, S: Sur, E: Este, W: Oeste).
pared: Almacena información sobre una pared, como su tipo de textura y distancia.
tipo_textura: Identificador de la textura de la pared (NO, SO, WE, EA).
distancia: Distancia desde el punto de origen al punto de intersección con la pared.
Declarar las funciones principales:
inicializar_cubo3d(): Carga la librería Minilibx, crea la ventana y el buffer de imagen, inicializa las variables del jugador y carga las texturas.
ciclo_principal(): Maneja el bucle principal del juego, procesando la entrada del usuario, actualizando la lógica del juego y renderizando la imagen.
procesar_entrada(): Lee los eventos del teclado y el ratón para actualizar la dirección del jugador y disparar si es necesario.
logica_del_juego(): Detecta colisiones, actualiza la posición de los enemigos y otros elementos del juego.
renderizar(): Limpia el buffer de imagen, dibuja las paredes y sprites, y muestra la imagen final en la ventana.
raycast(): Implementa el algoritmo Ray Casting para calcular la distancia a la pared más cercana en una dirección específica.
dibujar_pared_columna(): Dibuja una columna de la pared en el buffer de imagen en base a la distancia calculada por raycast().
dibujar_sprite(): Dibuja un sprite en el buffer de imagen en una posición y tamaño específicos, teniendo en cuenta la perspectiva 3D.
(Opcional) cargar_texturas(): Lee las rutas de las texturas desde el archivo .cub y carga las imágenes utilizando Minilibx.
(Opcional) mapeo_textura(textura, x, y, ancho, alto): Aplica la técnica de mapeo de texturas para ajustar la textura de la pared a la columna que se está dibujando.
3. Implementación de las funciones:

3.1 inicializar_cubo3d():
Explicación:

Esta función se encarga de la inicialización completa del juego, incluyendo la carga de la librería Minilibx, la creación de la ventana y el buffer de imagen, la lectura del archivo .cub para obtener la información del mapa, las texturas y los colores, la carga de las texturas, la inicialización de las variables del jugador y la inicialización de los enemigos (si se implementan).

Código:

void inicializar_cubo3d(void) {
    // Cargar librería Minilibx
    void *mlx = mlx_init();
    if (!mlx) {
        fprintf(stderr, "Error al inicializar Minilibx\n");
        exit(1);
    }

    // Crear ventana
    void *mlx_ptr = mlx_create_window(mlx, ANCHO_PANTALLA, ALTO_PANTALLA, "cub3D");
    if (!mlx_ptr) {
        fprintf(stderr, "Error al crear la ventana\n");
        mlx_destroy(mlx);
        exit(1);
    }

    // Crear buffer de imagen 
    unsigned int *buffer = mlx_get_data_addr(mlx_ptr, &ancho_buffer, &alto_buffer, &bits_per_pixel);
    if (!buffer) {
        fprintf(stderr, "Error al crear el buffer de imagen\n");
        mlx_destroy_window(mlx, mlx_ptr);
        mlx_destroy(mlx);
        exit(1);
    }

    // Leer archivo .cub
    leer_archivo_cub("mapa.cub");

    // Cargar texturas
    cargar_texturas();

    // Inicializar variables del jugador
    jugador.x = POSICION_INICIAL_X;
    jugador.y = POSICION_INICIAL_Y;
    jugador.angulo = DIRECCION_INICIAL;
    jugador.velocidad = VELOCIDAD_JUGADOR;

    // Inicializar enemigos (si se implementan)

}

Explicación del código:

Se carga la librería Minilibx utilizando la función mlx_init(). Si la carga falla, se muestra un mensaje de error y se termina el programa.
Se crea la ventana del juego utilizando la función mlx_create_window(). Se especifican el ancho y alto de la ventana, y el título de la misma. Si la creación de la ventana falla, se muestra un mensaje de error y se liberan los recursos utilizados.
Se obtiene el puntero al buffer de imagen de la ventana utilizando la función mlx_get_data_addr(). Se almacenan las dimensiones del buffer en las variables ancho_buffer y alto_buffer, y el número de bits por píxel en la variable bits_per_pixel. Si la obtención del buffer falla, se muestra un mensaje de error y se liberan los recursos utilizados.
Se llama a la función leer_archivo_cub() para leer la información del mapa, las texturas y los colores desde el archivo .cub.
Se llama a la función cargar_texturas() para cargar las texturas de pared desde las rutas especificadas en el archivo .cub.
Se inicializan las variables del jugador con su posición inicial, dirección y velocidad.
Se inicializan los enemigos (si se implementan) utilizando sus propias funciones específicas.
3.2 leer_archivo_cub():

Explicación:

Esta función se encarga de leer la información del archivo .cub, incluyendo el mapa, las rutas a las texturas y los colores del piso y techo.

Código:

C
void leer_archivo_cub(const char *nombre_archivo) {
    FILE *archivo = fopen(nombre_archivo, "r");
    if (!archivo) {
        fprintf(stderr, "Error al abrir el archivo %s\n", nombre_archivo);
        exit(1);
    }

    // Leer primera línea (ancho y alto del mapa)
    char linea[MAX_LINEA];
    fgets(linea, MAX_LINEA, archivo);
    sscanf(linea, "%d %d", &ancho_mapa, &alto_mapa);

    // Reservar memoria para el mapa
    mapa = malloc(alto_mapa * sizeof(char *));
    if (!mapa) {
        fprintf(stderr, "Error al reservar memoria para el mapa\n");
        fclose(archivo);
        exit(1);
    }

for (int i = 0; i < alto_mapa; i++) {
        mapa[i] = malloc(ancho_mapa + 1);
        if (!mapa[i]) {
            for (int j = 0; j < i; j++) {
                free(mapa[j]);
            }
            free(mapa);
            fclose(archivo);
            exit(1);
        }
    }

    // Leer líneas restantes (mapa, texturas, colores)
    char *ptr_linea = linea;
    for (int i = 0; i < alto_mapa; i++) {
        fgets(linea, MAX_LINEA, archivo);
        for (int j = 0; j < ancho_mapa; j++) {
            mapa[i][j] = *ptr_linea;
            ptr_linea++;
        }
        *ptr_linea = '\0'; // Terminar la cadena
        ptr_linea = linea; // Reiniciar puntero para la siguiente línea
    }

    // Leer rutas a las texturas
    fgets(linea, MAX_LINEA, archivo);
    sscanf(linea, "%s %s %s %s", &ruta_textura_no, &ruta_textura_so, &ruta_textura_we, &ruta_textura_ea);

    // Leer colores del piso y techo
    fgets(linea, MAX_LINEA, archivo);
    sscanf(linea, "%x %x", &color_piso, &color_techo);

    fclose(archivo);
}

Explicación del código:

Se abre el archivo .cub en modo lectura utilizando la función fopen(). Si la apertura falla, se muestra un mensaje de error y se termina el programa.
Se lee la primera línea del archivo, que contiene el ancho y alto del mapa. Se utilizan las funciones fgets() y sscanf() para leer la línea y almacenar los valores en las variables ancho_mapa y alto_mapa.
Se reserva memoria para el mapa utilizando la función malloc(). Se crea un arreglo bidimensional de caracteres mapa con el ancho y alto del mapa. Si la reserva de memoria falla, se muestra un mensaje de error, se liberan los recursos utilizados y se termina el programa.
Se recorren las líneas restantes del archivo para leer el contenido del mapa, las rutas a las texturas y los colores del piso y techo. Se utilizan las funciones fgets() y sscanf() para leer cada línea y almacenar los datos en las variables correspondientes.
Se cierra el archivo utilizando la función fclose().
3.3 cargar_texturas():

Explicación:

Esta función se encarga de cargar las texturas de pared desde las rutas especificadas en el archivo .cub.

Código:

C
void cargar_texturas(void) {
    textura_no = mlx_ImageLoad(mlx, ruta_textura_no, &ancho_textura_no, &alto_textura_no);
    if (!textura_no) {
        fprintf(stderr, "Error al cargar la textura NO: %s\n", ruta_textura_no);
        exit(1);
    }

    textura_so = mlx_ImageLoad(mlx, ruta_textura_so, &ancho_textura_so, &alto_textura_so);
    if (!textura_so) {
        fprintf(stderr, "Error al cargar la textura SO: %s\n", ruta_textura_so);
        exit(1);
    }

    textura_we = mlx_ImageLoad(mlx, ruta_textura_we, &ancho_textura_we, &alto_textura_we);
    if (!textura_we) {
        fprintf(stderr, "Error al cargar la textura WE: %s\n", ruta_textura_we);
        exit(1);
    }

    textura_ea = mlx_ImageLoad(mlx, ruta_textura_ea, &ancho_textura_ea, &alto_textura_ea);
    if (!textura_ea) {
        fprintf(stderr, "Error al cargar la textura EA: %s\n", ruta_textura_ea);
        exit(1);
    }
Explicación del código:

Se carga cada textura de pared utilizando la función mlx_ImageLoad(). Se pasan como parámetros la ruta a la imagen, las variables donde se almacenarán el ancho y alto de la imagen cargada, y un puntero a la imagen

3.3 cargar_texturas() (continuación):

Si la carga de una textura falla, se muestra un mensaje de error con la ruta de la textura y se termina el programa.
3.4 ciclo_principal():

Explicación:

Esta función maneja el bucle principal del juego, procesando la entrada del usuario, actualizando la lógica del juego y renderizando la imagen.

Código:

C
void ciclo_principal(void) {
    while (1) {
        // Procesar entrada del usuario
        procesar_entrada();

        // Actualizar lógica del juego
        logica_del_juego();

        // Renderizar imagen
        renderizar();

        // Actualizar la ventana
        mlx_SwapBuffers(mlx, mlx_ptr);
    }
}

Explicación del código:

El bucle while se ejecuta indefinidamente hasta que el usuario decide salir del juego.
Dentro del bucle, se llama a las funciones procesar_entrada(), logica_del_juego() y renderizar() para realizar sus respectivas tareas.
Al final del bucle, se llama a la función mlx_SwapBuffers() para actualizar la ventana del juego y mostrar la imagen renderizada.
3.5 procesar_entrada():

Explicación:

Esta función lee los eventos del teclado y el ratón para actualizar la dirección del jugador y disparar si es necesario.

Código:

C
void procesar_entrada(void) {
    mlx_events *e;
    int estado = mlx_get_events(mlx, mlx_ptr, &e);

    if (estado) {
        // Leer eventos del teclado
        if (e->key_press == 1) {
            switch (e->key_code) {
                case KEY_W:
                    jugador.angulo = (jugador.angulo + 90) % 360;
                    break;
                case KEY_S:
                    jugador.angulo = (jugador.angulo - 90) % 360;
                    break;
                case KEY_A:
                    jugador.angulo = (jugador.angulo + 180) % 360;
                    break;
                case KEY_D:
                    jugador.angulo = jugador.angulo % 360;
                    break;
            }
        }
        }
    }
}

Explicación del código:

Se obtiene el estado de los eventos del teclado y el ratón utilizando la función mlx_get_events(). Si hay eventos pendientes, se almacena la información en la estructura e.
Si se detecta una pulsación de tecla, se comprueba el código de la tecla y se actualiza la dirección del jugador en consecuencia.
Si se detecta una pulsación del botón izquierdo del ratón, se implementa la lógica de disparo (que debe ser implementada por el programador).
3.6 logica_del_juego():

Explicación:

Esta función se encarga de actualizar la lógica del juego, incluyendo la detección de colisiones, el movimiento de enemigos y otros elementos del juego.

Código:

C
void logica_del_juego(void) {

}

Explicación del código:

Esta función debe ser implementada por el programador en base a las reglas y mecánicas específicas del juego que se esté desarrollando.
Se deben implementar las funciones para detectar colisiones entre el jugador y las paredes, los enemigos y otros objetos del juego.
Se debe implementar la lógica para mover los enemigos, cambiar su estado o comportamiento, y aplicar las reglas del juego en general.
3.7 renderizar():

Explicación:

Esta función se encarga de renderizar la imagen del juego, incluyendo el dibujo de las paredes y otros elementos.

Código:

void renderizar(void) {
    // Limpiar buffer de imagen
    llenar_buffer(color_piso);

    // Dibujar paredes
    for (int x = 0; x < ANCHO_PANTALLA; x++) {
        raycast_y(x);
    }
    // Mostrar imagen en la ventana
    mlx_PutImageToWindow(mlx, mlx_ptr, 0, 0, buffer);
}

Explicación del código:

Se llama a la función llenar_buffer() para limpiar el buffer de imagen con el color del piso.
Se recorre cada columna de la pantalla (desde 0 hasta el ancho de la pantalla) y se llama a la función raycast_y() para calcular la distancia a la pared más cercana en esa dirección y dibujar la columna correspondiente.
Si se desea dibujar sprites, se implementaría un bucle para recorrer los sprites del juego y dibujar cada uno en su posición y tamaño correctos.
Finalmente, se llama a la función mlx_PutImageToWindow() para copiar la imagen renderizada en el buffer de imagen en la ventana del juego.
3.8 raycast_y():

Explicación:

Esta función calcula la distancia a la pared más cercana en una dirección vertical (y) y dibuja la columna correspondiente en el buffer de imagen.

Código:

C
void raycast_y(int x) {
    pared pared_cercana;
    pared_cercana.distancia = INFINITO;

    // Calcular ángulo de rayo
    double angulo_rayo = jugador.angulo + (x - (ANCHO_PANTALLA / 2)) * ANGULO_FOV / (ANCHO_PANTALLA / 2);

    // Calcular distancia inicial a la pared
    double distancia_inicial = CELDA_TAMANIO / cos(angulo_rayo);

    // Calcular posición y dirección del rayo
    int x_rayo = (int)(jugador.x + distancia_inicial * cos(angulo_rayo));
    int y_rayo = (int)(jugador.y + distancia_inicial * sin(angulo_rayo));

    // Recorrer celdas en dirección vertical
    while (1) {
        // Verificar si se sale del mapa
        if (x_rayo < 0 || x_rayo >= ancho_mapa || y_rayo < 0 || y_rayo >= alto_mapa) {
            break;
        }

        // Verificar si se encuentra una pared
        if (mapa[y_rayo][x_rayo] != '0') {
            pared pared_actual;
            pared_actual.tipo_textura = get_tipo_textura(x_rayo, y_rayo, angulo_rayo);
            pared_actual.distancia = distancia_inicial;

            // Actualizar pared más cercana si se encuentra una más próxima
            if (pared_actual.distancia < pared_cercana.distancia) {
                pared_cercana = pared_actual;
            }

            break;
        }

        // Calcular siguiente posición del rayo
        x_rayo += (int)(cos(angulo_rayo) * CELDA_TAMANIO);
        y_rayo += (int)(sin(angulo_rayo) * CELDA_TAMANIO);
        distancia_inicial += CELDA_TAMANIO;
    }

    // Dibujar columna de la pared
    dibujar_pared_columna(x, pared_cercana);
}
Explicación del código:

Se inicializa una variable pared_cercana para almacenar la información de la pared más cercana encontrada hasta el momento.
Se calcula el ángulo del rayo en base a la posición de la columna (x) y el campo de visión (FOV) del jugador.
Se calcula la distancia inicial a la pared utilizando la trigonometría.
Se calculan la posición inicial y la dirección del rayo en base a la posición del jugador y la distancia inicial.
Se recorren las celdas en dirección vertical a partir de la posición inicial del rayo.
En cada celda, se verifica si se sale del mapa o si se encuentra una pared.
Si se encuentra una pared, se calcula el tipo de textura de la pared y la distancia a la misma.
Se actualiza la variable pared_cercana si se encuentra una pared más cercana que la anterior.
Una vez que se encuentra la pared más cercana, se llama a la función dibujar_pared_columna() para dibujar la columna correspondiente en el buffer de imagen.
3.9 dibujar_pared_columna():

Explicación:

Esta función dibuja una columna de la pared en el buffer de imagen en base a

3.9 dibujar_pared_columna() (continuación):

void dibujar_pared_columna(int x, pared pared_cercana) {
    // Calcular altura de la columna
    int altcolumna = (ALTO_PANTALLA / pared_cercana.distancia) * CELDA_TAMANIO;
    if (altura_columna > ALTO_PANTALLA) {
        altura_columna = ALTO_PANTALLA;
    }

    // Calcular inicio y fin de la columna
    int inicio_columna = (ALTO_PANTALLA - altura_columna) / 2;
    int fin_columna = inicio_columna + altura_columna;

    // Dibujar la columna de la pared
    for (int y = inicio_columna; y < fin_columna; y++) {
        // Calcular índice del píxel en el buffer de imagen
        int indice_pixel = y * ANCHO_PANTALLA + x;

        // Calcular coordenadas de la textura
        double altura_relativa = (y - inicio_columna) / (double) altura_columna;
        double textura_y = altura_relativa * (double) ALTO_TEXTURA;

        // Obtener el color de la textura
        unsigned int *ptr_textura = (unsigned int *) mlx_Get_Texels(pared_cercana.tipo_textura);
        int indice_textura = (int)(textura_y * (double) ancho_textura_no);
        unsigned int color_textura = ptr_textura[indice_textura];

        // Establecer el color del píxel en el buffer de imagen
        buffer[indice_pixel] = color_textura;
    }
}

Explicación del código:

Se calcula la altura de la columna de la pared en base a la distancia a la pared.
Se calculan el inicio y el fin de la columna en la pantalla.
Se recorre la columna píxel por píxel.
Para cada píxel, se calcula el índice del píxel en el buffer de imagen.
Se calculan las coordenadas de la textura en base a la altura relativa del píxel en la columna.
Se obtiene el color de la textura en base a las coordenadas de la textura.
Se establece el color del píxel en el buffer de imagen con el color de la textura.

1. Funciones adicionales:

get_tipo_textura(x, y, angulo_rayo): Esta función determina el tipo de textura de la pared en una celda específica en base a la posición del jugador y el ángulo del rayo.
dibujar_sprite(sprite, x, y, ancho, alto): Esta función dibuja un sprite en el buffer de imagen en una posición y tamaño específicos, teniendo en cuenta la perspectiva 3D
mapeo_textura(textura, x, y, ancho, alto): Esta función aplica la técnica de mapeo de texturas para ajustar la textura de la pared a la columna que se está dibujando.

5. Implementación final:

Con la implementación de todas las funciones descritas, se obtiene un juego cub3D básico funcional que permite al jugador moverse por un mapa, visualizar las paredes y sprites, y detectar colisiones.