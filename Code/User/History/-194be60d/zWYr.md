# Guia basica para instalar ARCH linux con Hyprland
Esta es una guia basica donde tu podras tener tu arch + hyperland totalmente personalizado, se te ira dando el paso a paso para instalarlo en dual boot con windows
## 1. Instalacion
Suponiendo que ya tienes una usb booteada con linux vamos al primer paso

### 1.1 Conectar a internet

#### Cable LAN
Aqui no tiene mucho que hacer pues arch linux ya te detecta por defecto el internet por cable
```
ping -c 4 archlinux.org
```
Con este comando podemos ver si estamos conectados a internet
#### Wifi
En este paso si debemos conectarnos mediante comandos
##### 1.1.1 Abrir iwctl
```
iwctl
```
Te saldra algo muy parecido a:
```
[iwd]#
```
##### 1.1.2 Ver tu tarjeta de red
```
device list
```
Podras ver algo:
```
wlan0
wlp2s0
```
#### 1.1.3 Escanear redes
```
station wlan0 scan
```
#### 1.1.4 Listar redes  disponibles
```
