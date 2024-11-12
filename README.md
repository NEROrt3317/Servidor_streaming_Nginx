# Servidor_streaming_Nginx
ESto es un proyecto de servicios telematicos donde se va implementar un servidor streaming con Nginx y  RTMP

```
```
` `

1. Actualizar el sistema
Primero, asegúrate de que el sistema esté actualizado.
```
sudo dnf update -y
```

2. Instalar las dependencias
Para compilar `NGINX` con el módulo `RTMP`, instala las herramientas de desarrollo y dependencias necesarias.
```
sudo dnf install -y epel-release
sudo dnf install -y git gcc gcc-c++ make zlib-devel pcre-devel openssl-devel 
```
NOTA: FFMPEG NO ESTA DISPONIBLE EN LOS REPOSITORIOS ORIGINALES DE ROCKY LINUX
2.1 INSTALACION DE FFMPEG
## Habilitar los repositorios de EPEL y RPM Fusion
### Habilitar EPEL (Extra Packages for Enterprise Linux):
```
sudo dnf install -y https://mirrors.rpmfusion.org/free/el/rpmfusion-free-release-8.noarch.rpm
sudo dnf install -y https://mirrors.rpmfusion.org/nonfree/el/rpmfusion-nonfree-release-8.noarch.rpm
```
### Instalar ffmpeg
Con los repositorios adicionales habilitados, puedes instalar ffmpeg directamente:
```
sudo dnf install -y ffmpeg ffmpeg-devel
```
###   Verificar la instalación
Para confirmar que ffmpeg se instaló correctamente, ejecuta:
```
ffmpeg -version
```
3. Descargar y compilar `NGINX` con el módulo `RTMP`
El módulo `RTMP` no está incluido de forma predeterminada en `NGINX`, por lo que necesitas descargarlo y compilarlo con `NGINX`.

### Descargar `NGINX` y el módulo `RTMP`:
```
cd /usr/local/src
wget http://nginx.org/download/nginx-1.22.1.tar.gz
git clone https://github.com/arut/nginx-rtmp-module.git
```
### Descomprimir `NGINX`:
```
tar -zxvf nginx-1.22.1.tar.gz
cd nginx-1.22.1
```
### Configurar y compilar `NGINX` con el módulo `RTMP`:
```
./configure --with-http_ssl_module --add-module=../nginx-rtmp-module
make
sudo make install
```
4. Configurar `NGINX` para el streaming `RTMP`
Edita el archivo de configuración de NGINX para añadir soporte RTMP. El archivo de configuración suele estar en `/usr/local/nginx/conf/nginx.conf`.

### Abrir el archivo de configuración:
```
sudo nano /usr/local/nginx/conf/nginx.conf
```
### Agregar la configuración RTMP: Añade el siguiente bloque RTMP dentro del archivo nginx.conf:
```
rtmp {
    server {
        listen 1935;
        chunk_size 4096;

        application live {
            live on;
            record off;

            # Configuración para archivos de video locales
            allow play all;
        }
    }
}

http {
    include mime.types;
    default_type application/octet-stream;

    server {
        listen 8080;
        location / {
            root html;
        }

        # Configuración para el acceso HTTP al stream RTMP
        location /live {
            types {
                application/vnd.apple.mpegurl m3u8;
                video/mp2t ts;
            }
            alias /var/www/live;
            add_header Cache-Control no-cache;
        }
    }
}
```
#### Explicacion 
`listen 1935`: especifica el puerto para RTMP (puedes cambiarlo si necesitas).
`application live`: define la aplicación RTMP llamada "live" para transmitir.
`live on`: permite la transmisión en vivo.
`record off`: desactiva la grabación.
El bloque HTTP configura el acceso a la transmisión desde un navegador a través del puerto 8080.

### Crear la carpeta de streaming HTTP:
```
sudo mkdir -p /var/www/live
```
5. Iniciar NGINX
Después de guardar la configuración, inicia el servidor NGINX.
```
/usr/local/nginx/sbin/nginx
```
6. Transmitir un video local
Usa `ffmpeg` para enviar el archivo de video al servidor `RTMP` configurado en `NGINX`.
### Ejecutar el streaming:
```
ffmpeg -re -i /ruta/a/tu/video.mp4 -c copy -f flv rtmp://localhost/live/stream
```
#### Explicacion
`-re`: simula el tiempo real al leer desde el archivo de entrada.
`-i` /ruta/a/tu/video.mp4: especifica el archivo de video local.
`-c copy`: mantiene el códec original del video.
`-f flv`: formato FLV para RTMP.
`rtmp`://localhost/live/stream: URL de destino de RTMP.
7. Validar la transmisión desde un cliente (VLC)
- Abre VLC en tu equipo cliente.
- Ve a Medios > Abrir ubicación de red.
- Ingresa la URL de transmisión RTMP:
```
rtmp://[IP_DEL_SERVIDOR]/live/stream
```
Nota: Reemplaza `[IP_DEL_SERVIDOR]` con la IP real de tu servidor Rocky Linux.
8. Alternativa de visualización con `youtube-dl`
Para verificar la transmisión en un navegador web puedes usar youtube-dl para generar un enlace reproducible en otras herramientas de video.
### Instalar youtube-dl:
```
sudo dnf install -y youtube-dl
```
### Obtener el enlace de streaming:
```
youtube-dl -g rtmp://localhost/live/stream
```
