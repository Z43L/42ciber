# 42ciber

## pasos para clonar el repo
# enlacce al binario con el ctf
- ctf_game >>>> https://home.mycloud.com/action/share/a5c0cb0a-1926-4254-b050-2a9ecfdb599c
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

detener_y_eliminar_contenedor() {
    IMAGE_NAME="${TAR_FILE%.tar}"
    CONTAINER_NAME="${IMAGE_NAME}_container"

    if [ "$(docker ps -a -q -f name=$CONTAINER_NAME -f status=exited)" ]; then
        docker rm $CONTAINER_NAME > /dev/null
    fi

    if [ "$(docker ps -q -f name=$CONTAINER_NAME)" ]; then
        docker stop $CONTAINER_NAME > /dev/null
        docker rm $CONTAINER_NAME > /dev/null
    fi

    # Elimina la imagen si existe
    if [ "$(docker images -q $IMAGE_NAME)" ]; then
        docker rmi $IMAGE_NAME > /dev/null
    fi
}

# Función para manejar la señal SIGINT (Ctrl+C)
trap ctrl_c INT

function ctrl_c() {
    echo -e "\e[1mEliminando el laboratorio, espere un momento...\e[0m"
    detener_y_eliminar_contenedor
    echo -e "\nEl laboratorio ha sido eliminado por completo del sistema."
    exit 0
}

if [ $# -ne 2 ]; then
    echo "Uso: $0 <archivo_tar> <puerto_host:puerto_contenedor>"
    echo "Ejemplo: $0 mi_laboratorio.tar 8080:80"
    exit 1
fi

if ! command -v docker &> /dev/null; then
    echo -e "\033[1;36m\nDocker no está instalado. Instalando Docker...\033[0m"
    sudo apt update
    sudo apt install docker.io -y
    echo -e "\033[1;36m\nEstamos habilitando el servicio de docker. Espere un momento...\033[0m"
    sleep 10
    sudo systemctl restart docker && sudo systemctl enable docker
    if [ $? -eq 0 ]; then
        echo "Docker ha sido instalado correctamente."
    else
        echo "Error al instalar Docker. Por favor, verifique y vuelva a intentarlo."
        exit 1
    fi
fi

TAR_FILE="$1"
PORT_MAPPING="$2"

echo -e "\e[1;93m\nEstamos desplegando la máquina vulnerable, espere un momento.\e[0m"
detener_y_eliminar_contenedor
docker load -i "$TAR_FILE" > /dev/null


if [ $? -eq 0 ]; then

    IMAGE_NAME=$(basename "$TAR_FILE" .tar) # Obtiene el nombre del archivo sin la extensión .tar
    CONTAINER_NAME="${IMAGE_NAME}_container"

    docker run -d --name $CONTAINER_NAME -p "$PORT_MAPPING" $IMAGE_NAME /bin/bash -c "service apache2 start ; su - s4vitar -c 'cd /opt/web && php -S localhost:9999'; while true; do echo 'Alive'; sleep 60; done"> /dev/null

    IP_ADDRESS=$(docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' $CONTAINER_NAME)

    echo -e "\e[1;96m\nMáquina desplegada, su dirección IP es --> \e[0m\e[1;97m$IP_ADDRESS\e[0m"
    echo -e "\e[1;96mPuerto forwardeado (host:contenedor) --> \e[0m\e[1;97m$PORT_MAPPING\e[0m"
    echo -e "\e[1;91m\nPresiona Ctrl+C cuando termines con la máquina para eliminarla\e[0m"

else
    echo -e "\e[91m\nHa ocurrido un error al cargar el laboratorio en Docker.\e[0m"
    exit 1
fi

# Espera indefinida para que el script no termine
while true; do
    sleep 1
done
```
