# Guia basica para instalar ARCH linux con Hyprland
Esta es una guia basica donde tu podras tener tu arch + hyperland totalmente personalizado, se te ira dando el paso a paso para instalarlo en dual boot con windows
## Instalacion
Suponiendo que ya tienes una usb booteada con arch linux vamos al primer paso

## NOTA!

En la bios trata de tener:
<ul>
    <li> EUFI : ON </li>
    <li> Secure boot : OFF </li>
    <li> Fast Boot: OFF </li>
</ul>

### Conectar a internet
---

### Cable LAN
Aqui no tiene mucho que hacer pues arch linux ya te detecta por defecto el internet por cable
```
ping -c 4 archlinux.org
```
Con este comando podemos ver si estamos conectados a internet

---

### Wifi
En este paso si debemos conectarnos mediante comandos
#### Abrir iwctl
```
iwctl
```
Te saldra algo muy parecido a:
```
[iwd]#
```
#### Ver tu tarjeta de red
```
device list
```
Podras ver algo:
```
wlan0
wlp2s0
```
#### Escanear redes
```
station wlan0 scan
```
### Listar redes  disponibles
```
station wlan0 get-networks
```
### Conectarte a la red
```
station wlan0 connect NOMBRE_DE_RED
```
### Nos salimos del iwctl
```
exit
```
### Finalmente verificamos la conexion
```
ping -c 4 archlinux.org
```
---

### Configuramos el teclado respecto a nuestras distribucion
```
loadkeys us-acentos
```
Yo uso us-acentos porque mi teclado es de distribucion US trata de buscar la tuya el latino es <code>la-latin1</code>
## Particionado
Ingresamos a la herramienta de particionado
```
cfdisk /dev/nvme0n1
```
Nota: si tu disco es ssd aparece como sda
Puedes verificar cual es tu disco usando el comando:
```
lsblk
```

| Particion | Espacio | Tipo| Nombre|
| --- | --- | --- |---|
| nvme0n1p5 | 512M | EFI System | Efi Linux |
| nvme0m1p6 | 50-100G | Linux Filesystem | root|
| nvme0n1p7| Resto almacenamiento | Linux Filesystem | home|
| nvme0n1p8 | 6-10G | Linux Swap | swap|

Nota: usamos una particion efi para evitar corromper el arranque de windows igualmente separamos el home del root para mayor seguridad, la particion de swap es opcional, el nombre de las particiones de la tablla anterior es solo una guia didactica porque puede variar

### Formateo de particiones
```
mkfs.fat -F32 /dev/nvme0n1p5
mkfs.ext4 /dev/nvme0n1p6
mkfs.ext4 /dev/nvme0n1p7
mkswap /dev/nvme0n1p8
swapon /dev/nvme0n1p8
```
### Montaje de las particiones
```
mount /dev/nvme0n1p6 /mnt

mkdir /mnt/home
mount /dev/nvme0n1p7 /mnt/home

mkdir /mnt/boot
mount /dev/nvme0n1p5 /mnt/boot
```

## Instalacion del sistema base

```
pacstrap /mnt base linux linux-firmware nano networkmanager sudo

```
### Creamos la tabla de particiones

```
genfstab -U /mnt >> /mnt/etc/fstab
```
### Ingresamos a la raiz de nuevo sistema con permisos de superusuario
```
arch-chroot /mnt
```

### Condiguracion de la zona horaria

```
ln -sf /usr/share/zoneinfo/America/Bogota /etc/localtime
hwclock --systohc
```
### Configuracion de idioma
```
nano /etc/locale.gen
```

Ahi encontraremos una lista de idiomas busca y descomenta:

```
en_US.UTF-8 UTF-8
es_CO.UTF-8 UTF-8
```

Guarda con <code>Ctrl+O</code>  y presiona <code>Enter</code>, Salte con <code>Ctrl+X</code>

### Guarda tu configuacion
```
locale-gen
echo "LANG=es_CO.UTF-8" > /etc/locale.conf
```

### Nombre el hostname

```
echo arch > /etc/hostname
```

## Configuracion de usuarios y contrasenas
```
passwd
```
Esta es la contrasena del superusuario o root.

### Crea tu usario
```
useradd -m -G wheel jxhn
passwd jxhn
```
Recomendacion: Evita ingresar mayusculas para evitar conflictos en un futuro.

### Configuracion del sudo
```
EDITOR=nano visudo
```
Buscala la linea <code> %wheel ALL=(ALL) ALL</code> y descomentala.

Guarda con <code>Ctrl+O</code>  y presiona <code>Enter</code>, Salte con <code>Ctrl+X</code>.

### Activa el NetworkManager
```
systemctl enable NetworkManager
```

## Configuracion del bootloader
Descagamos el grub
```
pacman -S grub efibootmgr os-prober
```
Y lo instalamos
```
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=ARCH
```
## Configuracion windows

```
nano /etc/default/grub
```
Agregamos o descomentamos:
```
GRUB_DISABLE_OS_PROBER=false
```

Guardamos la configuracion
```
grub-mkconfig -o /boot/grub/grub.cfg
```

### Configuracion de Nvidia

Descargamos las herramientas:
```
pacman -S nvidia-dkms nvidia-utils nvidia-settings mesa vulkan-icd-loader
```

### Descargamos hyprland
```
pacman -S hyprland waybar kitty dunst xdg-desktop-portal-hyprland thunar
```

###  Configuracion del audio

```
pacman -S pipewire pipewire-pulse pavucontrol
```
### Login manager

```
pacman -S sddm
systemctl enable sddm
```

## Configuracion de nvidia y hyprland

```
nano ~/.config/hypr/hyprland.conf
```
Y copiamos esto:
```
env = WLR_NO_HARDWARE_CURSORS,1
env = GBM_BACKEND,nvidia-drm
env = LIBVA_DRIVER_NAME,nvidia
```
## Finalizamos y reiniciamos

```
exit
umount -R /mnt
reboot
```
Nota: Cuando el sistema apague quitamos el usb y cuando inicie ingresamos con nuestro usuario y clave
### Configuracion modo kernel nvidia
```
nano /etc/default/grub
```

Buscamos la linea:
```
GRUB_CMDLINE_LINUX_DEFAULT
```
Y la modificamos por:
```
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash nvidia_drm.modeset=1"
```

Guardamos:
```
grub-mkconfig -o /boot/grub/grub.cfg
```
### Regeneramos initramfs
```
mkinitcpio -P
```
Reiniciamos
```
reboot
```
Activamos el sonido
```
systemctl --user enable --now pipewire
```

## Descargamos los dotfiles

```
git clone https://github.com/JxhnTuring/dotfiles.git
```
Copiamos la configuracion:
```
cp -r dotfiles/* .config
```
Instalamos la fuente:
```
sudo pacman -S ttf-jetbrains-mono-nerd
```

### Configuracion de kitty + zsh
Instalamos zsh
```
sudo pacman -S zsh fastfetch
```
Cambiamos shell por defecto:
```
chsh -s /bin/zsh
```

Instalamos el yay
```
sudo pacman -S --needed base-devel 
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si
```
Instalamos el oh my posh

```
yay -S oh-my-posh
```
Configuramos el zsh con el oh my posh

```
nano ~/.zshrc
```
Y pegamos
```
eval "$(oh-my-posh init zsh --config /usr/share/oh-my-posh/themes/hul10.omp.json)"
fastfectch
```
cerramos seccion con Super(Windows)+M

## Personalizacion del grub 
```
git clone https://github.com/vinceliuice/grub2-themes.git
cd grub2-themes
sudo ./install.sh -t whitesur
```

Poner fondo de la luna
```
cp
```
