# POPRS
Результат работы в этом семестре: 

Для повышения качества печати необходимо откалибровать 3Д-принтер внутренним методом Input shaper в Klipper. 
Метод управления используется для уменьшения вибраций и артефактов на поверхности печати, таких как «звон», «эхо» или «призраки» (ringing, ghosting). 
Он также известен как компенсация резонанса.

# 1. Подключить акселерометр ADXL345 к Orange Pi:

Пин ADXL – Пин Orange Pi

VCC(3v) – Out 3.3 V (1 pin)

GND – GND (6 pin)

SDA – MOSI (19 pin)

SDO – MISO (21 pin)

SCL – CLK (23 pin)

CS – CS (24 pin)

# 2. Активируем SPI интерфейс:

sudo apt update

sudo apt install python3-numpy python3-matplotlib libatlas-base-dev -y

cd ~/klipper/

sudo cp "./scripts/klipper-mcu-start.sh" /etc/init.d/klipper_mcu

sudo update-rc.d klipper_mcu defaults

cd ~/klipper/

make menuconfig

Выбираем Micro-controller Architecture –> Linux Process

sudo service klipper stop

make flash

sudo service klipper start

Включаем SPI-dev1:

1)sudo armbian-config

2)Пароль

3)System

4)Hardware

5)spi-spidev1 пробелом устанавливаем звездочку

Включаем SPI-dev1:

sudo nano /boot/armbianEnv.txt

overlays=spi-spidev1

param_spidev_spi_bus=1

param_spidev_spi_cs=0

# 3. Изменения в конфигурации printer.cfg:

Микроконтроллер для Orange Pi

[mcu host]

serial: /tmp/klipper_host_mcu

Настройка ADXL345

[adxl345]

cs_pin: host:None

spi_bus: spidev1.0

axes_map: y,x,z

Тестер резонансов
[resonance_tester]

accel_chip: adxl345

probe_points:
    200, 200, 10

# 4. Проверка работы акселерометра:

ACCELEROMETER_QUERY

Получаем:

Recv: // adxl345 values (x, y, z): 170.719200, 241.438400, 28.196800

# 5. Калибровка:

Запускаем калибровку по X:

TEST_RESONANCES AXIS=X

Запускаем калибровку по Y:

TEST_RESONANCES AXIS=Y

Строим графики:

~/klipper/scripts/calibrate_shaper.py /tmp/resonances_x_*.csv -o ~/klipper/shaper_calibrate_x.png

~/klipper/scripts/calibrate_shaper.py /tmp/resonances_y_*.csv -o ~/klipper/shaper_calibrate_y.png

# 6. Интегрируем предложенную конфигурацию [input_shaper] в printer.cfg:

[input_shaper]

shaper_freq_x: 79.2

shaper_type_x: mzv

shaper_freq_y: 34.6

shaper_type_y: mzv

[printer]

max_accel: 3000
