# 42ciber

## pasos para clonar el repo

# enlace de descarga de las maqinas de [Dockerlabs.es](https://dockerlabs.es/#/)
- [hackthehieven]([hacktheheaven](https://mega.nz/file/9G0BCDTC#YMVrW5AmlCfRDFm5fZgz1FWZkHsHK-F0tgM3WcfZfzY)) -> hard >>>> https://home.mycloud.com/action/share/d357d797-014d-4abf-8d5b-99a01eec8d95
- [hidden]([hidden](https://mega.nz/file/EO8DzKgR#V3Vj8pWT6dUfWP03Zi2ZNs-o3uztnrTd1qGxvnn3oHo)) -> medium >>>> https://home.mycloud.com/action/share/e4ebf09b-cb56-4c5e-a345-ebdf54177ce4
- [injection]([injection](https://mega.nz/file/wLN2nQ7B#p0YzUFAsrE3ilnJ9HzMr1hfsUq2DPYiDHlIU_9IEizU)) -> easy >>>>  https://home.mycloud.com/action/share/0357c556-8565-45a8-b586-bc1c3cf86aca

# como desplegar laboratorio

Copiar el script de abajo y ejecutar 
```
./script.sh injection.tar 8080:80

```

# el escript a copiar 

```
#!/bin/bash

# Función para detener y eliminar contenedor e imagen asociados al TAR
detener_y_eliminar_contenedor() {
    local image_name="$1" # Recibe el nombre de la imagen como argumento
    local container_name="${image_name}_container"

    echo -e "\n\e[1;36m-> Buscando y eliminando contenedor previo '$container_name' (si existe)...\e[0m"
    # Comprueba si el contenedor existe (corriendo o parado)
    if [ "$(docker ps -a -q -f name=$container_name)" ]; then
        # Detiene (si está corriendo) y elimina el contenedor
        docker rm -f "$container_name" > /dev/null 2>&1
        if [ $? -eq 0 ]; then
            echo -e "\e[1;32m   Contenedor '$container_name' eliminado.\e[0m"
        else
             echo -e "\e[1;31m   Error al eliminar el contenedor '$container_name'. Puede que ya no existiera.\e[0m"
        fi
    else
        echo -e "\e[1;34m   No se encontró un contenedor previo llamado '$container_name'.\e[0m"
    fi

    echo -e "\e[1;36m-> Buscando y eliminando imagen previa '$image_name' (si existe)...\e[0m"
    # Comprueba si la imagen existe
    if [ "$(docker images -q $image_name)" ]; then
        docker rmi "$image_name" > /dev/null 2>&1
         if [ $? -eq 0 ]; then
            echo -e "\e[1;32m   Imagen '$image_name' eliminada.\e[0m"
         else
             # Podría fallar si un contenedor (quizás otro diferente al nuestro) aún la usa
             echo -e "\e[1;33m   No se pudo eliminar la imagen '$image_name'. Puede estar en uso por otro contenedor.\e[0m"
         fi
    else
        echo -e "\e[1;34m   No se encontró una imagen previa llamada '$image_name'.\e[0m"
    fi
}

# Variable global para el nombre de la imagen (necesaria en ctrl_c)
# Se inicializa vacía y se establece después de validar argumentos
IMAGE_NAME=""

# Función para manejar Ctrl+C
function ctrl_c() {
    echo -e "\n\n\e[1;31mInterrupción recibida (Ctrl+C).\e[0m"
    if [ -n "$IMAGE_NAME" ]; then
        echo -e "\e[1m\e[1;33mEliminando el laboratorio '$IMAGE_NAME', espere un momento...\e[0m"
        detener_y_eliminar_contenedor "$IMAGE_NAME" # Pasa el nombre de la imagen
        echo -e "\n\e[1;32mEl laboratorio '$IMAGE_NAME' ha sido eliminado del sistema.\e[0m"
    else
        echo -e "\e[1;34mNo se había iniciado ningún laboratorio para eliminar.\e[0m"
    fi
    exit 0
}

# Capturar la señal INT (Ctrl+C)
trap ctrl_c INT

# --- Inicio del Script Principal ---

# Verificar número de argumentos
if [ $# -ne 2 ]; then
    echo -e "\n\e[1;31mError: Número incorrecto de argumentos.\e[0m"
    echo "Uso: $0 <archivo_tar_imagen> <mapeo_puertos>"
    echo "Ejemplo: $0 mi_imagen_lab.tar 8080:80"
    echo "Donde <mapeo_puertos> es 'Puerto_Host:Puerto_Contenedor'."
    echo "  Puerto_Host: El puerto en tu máquina real para acceder al contenedor."
    echo "  Puerto_Contenedor: El puerto que la aplicación escucha DENTRO del contenedor."
    exit 1
fi

TAR_FILE="$1"
PORT_MAPPING="$2"

# Validar formato simple de mapeo de puertos (numero:numero)
if ! [[ "$PORT_MAPPING" =~ ^[0-9]+:[0-9]+$ ]]; then
    echo -e "\n\e[1;31mError: Formato de mapeo de puertos inválido: '$PORT_MAPPING'.\e[0m"
    echo "El formato debe ser 'Puerto_Host:Puerto_Contenedor', por ejemplo: 8080:80"
    exit 1
fi

# Verificar si el archivo TAR existe
if [ ! -f "$TAR_FILE" ]; then
    echo -e "\n\e[1;31mError: El archivo '$TAR_FILE' no existe o no es un archivo regular.\e[0m"
    exit 1
fi

# Extraer nombre de la imagen del archivo TAR (quitando .tar)
IMAGE_NAME=$(basename "$TAR_FILE" .tar)
# Extraer el puerto del host del mapeo
HOST_PORT=$(echo "$PORT_MAPPING" | cut -d: -f1)
# Definir nombre del contenedor
CONTAINER_NAME="${IMAGE_NAME}_container"

# Verificar si Docker está instalado
if ! command -v docker &> /dev/null; then
    echo -e "\033[1;36m\nDocker no está instalado. Intentando instalar Docker...\033[0m"
    # Comprobar si se puede usar sudo o ya es root
    if [ "$(id -u)" -ne 0 ] && ! command -v sudo &> /dev/null; then
        echo -e "\e[1;31mError: Se necesita 'sudo' para instalar Docker, pero no se encontró el comando.\e[0m"
        echo "Por favor, instala Docker manualmente o ejecuta el script como root/con sudo."
        exit 1
    fi
    # Usar sudo si no es root
    SUDO_CMD=""
    if [ "$(id -u)" -ne 0 ]; then
        SUDO_CMD="sudo"
    fi

    $SUDO_CMD apt update -y
    $SUDO_CMD apt install docker.io -y
    if [ $? -ne 0 ]; then
       echo -e "\e[1;31mError al instalar Docker. Por favor, verifique los errores e inténtelo de nuevo.\e[0m"
       exit 1
    fi
    echo -e "\033[1;36m\nHabilitando y reiniciando el servicio de docker. Espere un momento...\033[0m"
    sleep 5 # Reducido el sleep
    $SUDO_CMD systemctl restart docker && $SUDO_CMD systemctl enable docker
    if [ $? -eq 0 ]; then
        echo -e "\e[1;32mDocker ha sido instalado y habilitado correctamente.\e[0m"
        # Añadir usuario actual al grupo docker (requiere cerrar sesión y volver a entrar para efecto completo, pero útil para futuras ejecuciones)
        if [ -n "$SUDO_CMD" ]; then # Solo si usamos sudo
            echo -e "\e[1;33m\nAñadiendo usuario '$(whoami)' al grupo 'docker'. Necesitarás cerrar sesión y volver a entrar para ejecutar docker sin sudo.\e[0m"
            $SUDO_CMD usermod -aG docker "$(whoami)"
        fi
    else
        echo -e "\e[1;31mError al reiniciar/habilitar el servicio Docker. Por favor, verifique.\e[0m"
        exit 1
    fi
fi

# --- Despliegue ---
echo -e "\e[1;93m\n--- Iniciando Despliegue del Laboratorio '$IMAGE_NAME' ---\e[0m"

# 1. Limpiar instalaciones previas de este laboratorio
echo -e "\e[1;36m[Paso 1/4] Limpiando recursos previos (si existen)...\e[0m"
detener_y_eliminar_contenedor "$IMAGE_NAME"

# 2. Cargar la imagen desde el archivo TAR
echo -e "\e[1;36m\n[Paso 2/4] Cargando imagen '$IMAGE_NAME' desde '$TAR_FILE'...\e[0m"
docker load -i "$TAR_FILE" # Mostrar salida para ver progreso/errores
if [ $? -ne 0 ]; then
    echo -e "\e[91m\nError: Ha ocurrido un error al cargar la imagen desde '$TAR_FILE'.\e[0m"
    exit 1
fi
echo -e "\e[1;32m   Imagen cargada correctamente.\e[0m"

# 3. Ejecutar el contenedor con el mapeo de puertos
echo -e "\e[1;36m\n[Paso 3/4] Ejecutando contenedor '$CONTAINER_NAME' en segundo plano...\e[0m"
echo -e "   Mapeo de puertos solicitado: Host($HOST_PORT) -> Contenedor($(echo "$PORT_MAPPING" | cut -d: -f2))"
docker run -d --name "$CONTAINER_NAME" -p "$PORT_MAPPING" "$IMAGE_NAME" > /dev/null
if [ $? -ne 0 ]; then
    echo -e "\e[91m\nError: Ha ocurrido un error al ejecutar el contenedor '$CONTAINER_NAME'.\e[0m"
    echo "   Posibles causas: El puerto $HOST_PORT ya está en uso en el host, o hay un problema con la imagen."
    # Intentar limpiar la imagen cargada si falla el run
    detener_y_eliminar_contenedor "$IMAGE_NAME"
    exit 1
fi
echo -e "\e[1;32m   Contenedor ejecutándose.\e[0m"

# 4. Obtener la IP del HOST y mostrar información de acceso
echo -e "\e[1;36m\n[Paso 4/4] Obteniendo información de acceso...\e[0m"
# Intentar obtener la IP de la LAN del host (puede necesitar ajustes según el sistema)
HOST_IP=$(hostname -I | awk '{print $1}')
if [ -z "$HOST_IP" ]; then
    echo -e "\e[1;33m   No se pudo determinar automáticamente la IP del host.\e[0m"
    echo -e "\e[1;33m   Puedes encontrarla manualmente ejecutando 'ip addr' o 'hostname -I'.\e[0m"
    ACCESS_INFO="<IP_DEL_HOST>:$HOST_PORT"
else
    echo -e "\e[1;32m   IP del Host detectada: $HOST_IP\e[0m"
    ACCESS_INFO="$HOST_IP:$HOST_PORT"
fi

echo -e "\n\e[1;92m--- Laboratorio Desplegado Exitosamente ---"
echo -e "\e[1;96mPuedes acceder al servicio en:\e[0m \e[1;97mhttp://$ACCESS_INFO\e[0m"
echo -e "\e[1;96m(Asegúrate de que el servicio dentro del contenedor use HTTP si usas 'http://' para acceder)\e[0m"
echo -e "\e[1;96mEl contenedor '$CONTAINER_NAME' se está ejecutando en segundo plano.\e[0m"

echo -e "\n\e[1;91m>>> Presiona Ctrl+C cuando termines para detener y eliminar el laboratorio <<<\e[0m"

# Espera indefinida para que el script no termine y se pueda usar Ctrl+C
while true; do
    sleep 60 # Dormir un poco más para no consumir CPU innecesariamente
done


```
