# Docker-compose MING (mosquitto) STACK

Docker-compose para hacer un deploy del siguiente Stack
1. MQTT (Mosquitto)
2. InfluxDB
3. Node-RED
4. Grafana


## 1. Crear volúmenes locales

### 1.1. Node-RED

```
mkdir -p /home/ubuntu/docker/nodered/data
```
### 1.2. Grafana

```
mkdir -p /home/ubuntu/docker/grafana/var/lib/grafana
```
### 1.3. InfluxDB
Volumen para versión 2.X
```
mkdir -p /home/ubuntu/docker/influxdb/var/lib/influxdb2 && mkdir -p /home/ubuntu/docker/influxdb/etc/influxdb2
```
Volumen para versión 1.X
```
mkdir -p /home/ubuntu/docker/influxdb/var/lib/influxdb && mkdir -p /home/ubuntu/docker/influxdb/etc/influxdb
```
### 1.4. Grafana

```
mkdir -p /home/ubuntu/docker/mosquitto/config && mkdir -p /home/ubuntu/docker/mosquitto/data && mkdir -p /home/ubuntu/docker/mosquitto/log
```
```
touch /home/ubuntu/docker/mosquitto/config/password_file
```
## 2. Crear usuario y grupo en el sistema para Mosquitto
```
groupadd mqtt -g 1883
```
```
useradd mqtt -u 1883 -g 1883 -m -s /bin/true
```
```
sudo chown -R mqtt.mqtt /home/ubuntu/docker/mosquitto
```
## 3. Configurar Mosquitto
```
sudo nano /home/ubuntu/docker/mosquitto/config/mosquitto.conf
```
Pegar el contenido en `mosquitto.conf`
```
# PUERTOS A ESUCHAR
# mqtt
listener 1883
# websockets
listener 9001
protocol websockets

# PERSISTENCIA DE DATOS
persistence true
# Intervalo de guardado [Segundos]
autosave_interval 1800
# Expiracion de persistencia de datos [dias]
persistent_client_expiration 1d
# Archivo de persistencia de datos
persistence_file /mosquitto/data/persistence.db
# persistence_location /mosquitto/data

# LOGS
# Tipo de logs
log_type information
# Marca de tiempo en logs
log_timestamp true
# Archivo destino de los logs
log_dest file /mosquitto/log/mosquitto.log

# AUTENTICACION
# Permitir o No permitir clientes sin contraseña
allow_anonymous true
# Archivo de contraseñas
password_file /mosquitto/config/password_file

# RESTRICCIÓN DE TOPICS (opcional)
# acl_file /mosquitto/data/mosquitto.aclfile
```

## 4. Docker Compose
### 4.1. Network
Revisar nombre de la network de nginx. Cambiar en `networks:npm_proxy` si es necesario
```
docker network ls
```
### 4.2. User
Revisar el número de id y grupo del usuario. Cambiar en `user` si es necesario
```
whoami
id
```

## 5. Asegurar Mosquitto
### 5.1. Crear usuario y contraseña
```
docker exec -it mqtt sh
```
En `usuario` poner el nombre de usuario que se desee. Posteriormente escribir la contraseña en la consola
```
mosquitto_passwd -c /mosquitto/config/password_file usuario
```
Salir de la consola de Mosquitto
```
exit
```
### 5.2. No permitir anónimos
```
sudo nano /home/ubuntu/docker/mosquitto/config/mosquitto.conf
```
```
allow_anonymous false
```
### 5.3. Cómo crear más usuarios en mosquitto
```
docker exec -it mosquitto mosquitto_passwd -b /mosquitto/config/password_file <SegundoUsuario> <contraseñaSegundoUsuario>
```