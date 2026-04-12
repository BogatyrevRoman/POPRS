# POPRS
Результат работы в этом семестре: 

Для повышения качества печати необходимо откалибровать 3Д-принтер внутренним методом Input shaper в Klipper. 
Метод управления используется для уменьшения вибраций и артефактов на поверхности печати, таких как «звон», «эхо» или «призраки» (ringing, ghosting). 
Он также известен как компенсация резонанса.

# 1. Подключить акселерометр ADXL345 к Orange Pi:

Pin ADXL345 – Pin Octopus Pro (SPI)

VCC (3.3V) – 3.3V

GND – GND

CS – CS (PA15 pin)

SDO – MISO (PB4 pin)

SDA – MOSI (PB5 pin)

SCL – SCK (PB3 pin)

# 2. Редактируем printer.cfg:

Аппаратный SPI на Octopus Pro

    [adxl345]
    cs_pin: PB12
    spi_bus: spi1
    spi_speed: 2000000  

    [resonance_tester]
    accel_chip: adxl345
    probe_points:
        150, 150, 20   # точка на столе (X, Y, Z) в мм

# 3. Обход обязательного возвращения в HOME:

Редактируем printer.cfg

    [force_move]
    enable_force_move: True

Задаем положение в консоли:

    SET_KINEMATIC_POSITION X=150 Y=150 Z=20


# 4. Проверка работы акселерометра:

    ACCELEROMETER_QUERY

Получаем:

    Recv: // adxl345 values (x, y, z): 2220.617826, 9104.533087, 2274.896044

# 5. Калибровка:

Запускаем калибровку по X:

    TEST_RESONANCES AXIS=X

Запускаем калибровку по Y:

    TEST_RESONANCES AXIS=Y

Строим графики:

~/klipper/scripts/calibrate_shaper.py /tmp/resonances_x_*.csv -o ~/klipper/shaper_calibrate_x.png (Для оси X)
<img width="993" height="623" alt="Снимок экрана 2026-04-12 в 14 48 18" src="https://github.com/user-attachments/assets/2e30b3f9-5c5a-44e5-b9e1-8dd91639dff7" />
~/klipper/scripts/calibrate_shaper.py /tmp/resonances_y_*.csv -o ~/klipper/shaper_calibrate_y.png (Для оси Y)
<img width="993" height="623" alt="Снимок экрана 2026-04-12 в 14 49 22" src="https://github.com/user-attachments/assets/31889659-ba5e-46fc-bb0d-f388b6026a13" />



# 6. Интегрируем предложенную конфигурацию [input_shaper] в printer.cfg:

    [input_shaper]

    shaper_freq_x: 31.6

    shaper_type_x: mzv

    shaper_freq_y: 24.4

    shaper_type_y: zv

    [printer]

    max_accel: 1700
