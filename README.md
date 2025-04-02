## Instalación de Excalidraw Auto-alojado

Hay dos métodos para instalar Excalidraw. Elige el que mejor se adapte a tus necesidades:

### Método 1: Usando Traefik (Recomendado para Producción)
Este método proporciona HTTPS, mejor seguridad y una configuración adecuada de proxy inverso.

#### Prerequisitos
- Docker y Docker Compose instalados
- Acceso Root/Administrador para modificar el archivo hosts

#### Pasos de Instalación

1. Crear un archivo `docker-compose.yml` con el siguiente contenido:
```yaml
version: '3.3'

services:
  traefik2:
    image: traefik:latest
    restart: always
    command:
      - "--log.level=DEBUG"
      - "--api.insecure=true"
      - "--providers.docker=true"
      - "--providers.docker.exposedbydefault=true"
      - "--entrypoints.web.address=:80"
      - "--entrypoints.websecure.address=:443"
      - "--entrypoints.web.http.redirections.entryPoint.to=websecure"
      - "--entrypoints.web.http.redirections.entryPoint.scheme=https"
    ports:
      - 80:80
      - 443:443
    networks:
      traefik:        
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    container_name: traefik

  excalidraw:
    image: excalidraw/excalidraw:latest
    restart: always
    networks:
      traefik:
    environment:
      - LOG_LEVEL=debug
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.excalidraw.tls=true"
      - "traefik.http.routers.excalidraw.rule=Host(`excalidraw.cloud.local`)"
      - "traefik.http.routers.excalidraw.entrypoints=websecure"
      - "traefik.http.services.excalidraw.loadbalancer.server.port=80"
    container_name: excalidraw

networks:
  traefik:
    driver: bridge
    name: traefik
    ipam:
      driver: default
      config:
        - subnet: 172.19.0.0/16
```

2. Configurar el dominio local en el archivo hosts:

Para Linux/Mac:
```bash
sudo nano /etc/hosts
```

Para Windows (ejecutar Notepad como administrador):
```
127.0.0.1    excalidraw.cloud.local
```

3. Iniciar los servicios:
```bash
docker-compose up -d
```

4. Acceder a Excalidraw en:
```
https://excalidraw.cloud.local
```

Nota: Deberás aceptar la advertencia del certificado auto-firmado en tu navegador.

### Beneficios Principales

1. **Sin Exposición de Puertos**: 
   - No es necesario exponer el puerto 8888
   - Más seguro ya que la aplicación solo es accesible a través de Traefik
   - Sin acceso directo al contenedor

2. **Capacidad de Trabajo Sin Internet**:
   - Una vez descargadas las imágenes, funciona completamente sin conexión
   - No requiere conexión a internet para operar
   - Todos los datos permanecen en tu red local

3. **Gestión Simple**:
   - Usa `docker-compose down` para detener todos los servicios
   - Usa `docker-compose up -d` para iniciar todos los servicios
   - Docker Compose maneja toda la configuración automáticamente

Nota: El comando `docker pull excalidraw/excalidraw:latest` no es necesario porque `docker-compose up -d` descargará automáticamente las imágenes requeridas si no están presentes localmente. Solo necesitas ejecutar `docker pull` manualmente si específicamente quieres actualizar a la última versión de la imagen.

### Solución de Problemas

Si no puedes acceder a la aplicación:
1. Verifica que los contenedores estén ejecutándose: `docker-compose ps`
2. Revisa los logs: `docker-compose logs`
3. Asegúrate de que el archivo hosts esté configurado correctamente
4. Limpia la caché del navegador si es necesario

### Nota Importante Sobre Puertos

Si bien esta configuración usa Traefik y HTTPS (`https://excalidraw.cloud.local`), también es posible acceder a Excalidraw directamente a través de:
```
http://localhost:8888
```

Sin embargo, se recomienda usar Traefik (el método descrito en esta guía) porque:
- Proporciona cifrado HTTPS
- Mayor seguridad a través del proxy inverso
- Sin exposición directa de puertos
- Configuración más profesional para entornos de producción


### Instalación Alternativa: Acceso Directo por Puerto (Sin Traefik)

### Método 2: Acceso Directo por Puerto (Configuración Simple)
Este método es más simple pero menos seguro, adecuado para desarrollo local, por las siguientes razones:
- No tiene cifrado HTTPS (usa HTTP plano)
- Expone directamente el puerto del contenedor (8888)
- No tiene la capa adicional de seguridad que proporciona el proxy inverso
- No tiene control de acceso ni filtrado de solicitudes

Este método no requiere el archivo docker-compose.yml. Solo necesitarás ejecutar estos comandos:

1. (Opcional) Descargar la imagen:
```bash
docker pull excalidraw/excalidraw:latest
```

2. Ejecutar el contenedor:
```bash
docker run -d -p 8888:80 --name excalidraw-instance excalidraw/excalidraw:latest
```

3. Acceder a Excalidraw en:
```
http://localhost:8888
```

4. Para gestionar el contenedor:
```bash
# Detener contenedor
docker stop excalidraw-instance

# Eliminar contenedor
docker rm excalidraw-instance

# Detener y eliminar en un solo comando
docker rm -f excalidraw-instance
```

### Solución de Problemas

Si ves que Excalidraw sigue funcionando en el puerto 8888 después de detener los contenedores, intenta:
1. Limpiar la caché del navegador
2. Verificar que no hay contenedores ejecutándose con: `docker ps`

### Comparación de Métodos

#### Beneficios del Método 1 (Traefik):
- Cifrado HTTPS
- Protección mediante proxy inverso
- Mayor seguridad
- Sin exposición directa de puertos
- Funciona sin internet después de la configuración inicial
- Configuración apropiada para producción

#### Beneficios del Método 2 (Puerto Directo):
- Configuración más simple
- Menos componentes
- Más fácil de entender
- Rápido de implementar para pruebas locales

Importante: El Método 2 es adecuado para:
- Desarrollo local
- Pruebas rápidas
- Entornos de aprendizaje
- Uso personal en una red local segura

No se recomienda para:
- Entornos de producción
- Servidores expuestos a internet
- Uso empresarial
- Datos sensibles
