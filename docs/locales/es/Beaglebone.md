# Beaglebone

Este documento describe el proceso de ejecución de Klipper en un Beaglebone PRU.

## Creando una imagen del SO

Comience instalando la imagen [Debian 9.9 2019-08-03 4GB SD IoT](https://beagleboard.org/latest-images). Puede ejecutar la imagen desde una tarjeta micro-SD o desde la eMMC integrada. Si utiliza la eMMC, instálela ahora en la eMMC siguiendo las instrucciones del enlace anterior.

Then ssh into the Beaglebone machine (`ssh debian@beaglebone` -- password is `temppwd`) and install Klipper by running the following commands:

```
git clone https://github.com/Klipper3d/klipper
./klipper/scripts/install-beaglebone.sh
```

## Instala Octoprint

A continuación, puede instalar Octoprint:

```
git clone https://github.com/foosel/OctoPrint.git
cd OctoPrint/
virtualenv venv
./venv/bin/python setup.py install
```

Y configure OctoPrint para que se inicie al arrancar:

```
sudo cp ~/OctoPrint/scripts/octoprint.init /etc/init.d/octoprint
sudo chmod +x /etc/init.d/octoprint
sudo cp ~/OctoPrint/scripts/octoprint.default /etc/default/octoprint
sudo update-rc.d octoprint defaults
```

It is necessary to modify OctoPrint's **/etc/default/octoprint** configuration file. One must change the `OCTOPRINT_USER` user to `debian`, change `NICELEVEL` to `0`, uncomment the `BASEDIR`, `CONFIGFILE`, and `DAEMON` settings and change the references from `/home/pi/` to `/home/debian/`:

```
sudo nano /etc/default/octoprint
```

A continuación, inicie el servicio Octoprint:

```
sudo systemctl start octoprint
```

Make sure the OctoPrint web server is accessible - it should be at: <http://beaglebone:5000/>

## Compilar el código del microcontrolador

To compile the Klipper micro-controller code, start by configuring it for the "Beaglebone PRU":

```
cd ~/klipper/
make menuconfig
```

Para compilar e instalar el nuevo código del microcontrolador, ejecute:

```
sudo service klipper stop
make flash
sudo service klipper start
```

It is also necessary to compile and install the micro-controller code for a Linux host process. Configure it a second time for a "Linux process":

```
make menuconfig
```

Then install this micro-controller code as well:

```
sudo service klipper stop
make flash
sudo service klipper start
```

## Configuración restante

Complete the installation by configuring Klipper and Octoprint following the instructions in the main [Installation](Installation.md#configuring-klipper) document.

## Printing on the Beaglebone

Unfortunately, the Beaglebone processor can sometimes struggle to run OctoPrint well. Print stalls have been known to occur on complex prints (the printer may move faster than OctoPrint can send movement commands). If this occurs, consider using the "virtual_sdcard" feature (see [Config Reference](Config_Reference.md#virtual_sdcard) for details) to print directly from Klipper.
