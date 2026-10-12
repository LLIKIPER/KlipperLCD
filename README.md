# KlipperLCD (для дисплея Elegoo Neptune 3 Pro)

Хотите установить Klipper на Neptune 3 Pro, но при этом сохранить возможность пользоваться штатным сенсорным дисплеем?

Обратите внимание на этот сервис на Python для дисплея Elegoo Neptune 3 Pro! Он работает совместно с Klipper3d и Moonraker.

В данном проекте используется плата **BIGTREETECH BTT PI V1.2**

**В отличии от форка, сдесть реализовано включение/выключение принтера, по параметру "Fan control" в настройках** (На данный момент на таком пункте, планирую изменить прошивку)

## Внешний вид и интерфейс
<p float="left">
    <img src="img/boot_screen.PNG" height="400">
    <img src="img/main_screen.PNG" height="400">
    <img src="img/about_screen.PNG" height="400">
</p>

## Что потребуется?
* Принтер Elegoo Neptune 3 Pro с дисплеем.
* BIGTREETECH BTT PI V1.2 или другой одноплатный компьютер (SBC) для запуска Klipper. Для установки Klipper, Moonraker и выбранного веб-интерфейса ([Fluidd](https://docs.fluidd.xyz/) или [Mainsail](https://docs.mainsail.xyz/))
* Небольшая переделка подключения дисплея: его необходимо соединить c USB–UART.
* После этого следуйте инструкции ниже, чтобы задействовать сенсорный дисплей Neptune 3 Pro.

## Подключение через преобразователь USB–UART

Всё достаточно просто: не забудьте перекрестить линии RX и TX дисплея и преобразователя USB–UART.

| Преобразователь USB–UART | Дисплей |
| --- | --- |
| 5V | 5V (чёрный провод) |
| GND | GND (красный провод) |
| TXD | RX (зелёный провод) |
| RXD | TX (жёлтый провод) |

<p float="left">
    <img src="img/USB_conn.png" height="400">
    <img src="img/LCD_conn.png" height="400">
</p>

> **Внимание! Проверьте обязательно, что на выходе +5V у вас 5-5.5V. При плохом блоке питания, может быть 4,5V и плата основная плата будет работать, а вот USB-Uart будет отваливаться! Что и произошло со мной.**


## Обновление прошивки дисплея

1. Скопируйте файл прошивки дисплея `LCD/20240125.tft` в корень карты microSD, отформатированной в FAT32.
2. Убедитесь, что питание дисплея выключено.
3. Вставьте карту microSD в **слот дисплея**. Для этого потребуется снять заднюю крышку. **В самом дисплее есть разъем под карту!**
4. Включите дисплей и дождитесь появления сообщения `Update Successed!`.

Более подробная инструкция по обновлению прошивки дисплея доступна [на сайте Elegoo](https://www.elegoo.com/blogs/3d-printing/elegoo-neptune-3-pro-plus-max-fdm-3d-printer-support-files).


## Запуск сервиса KlipperLCD

* Подключитесь к Raspberry Pi по SSH.

### API-сокет Klipper

* Убедитесь, что API-сокет Klipper включён, проверив аргументы запуска Klipper.

Команда:

```bash
cat ~/printer_data/systemd/klipper.env
```

Пример вывода:

```text
KLIPPER_ARGS="/home/biqu/klipper/klippy/klippy.py /home/biqu/printer_data/config/printer.cfg -I /home/biqu/printer_data/comms/klippy.serial -l /home/biqu/printer_data/logs/klippy.log -a /home/biqu/printer_data/comms/klippy.sock"
```

В переменной `KLIPPER_ARGS` должен присутствовать аргумент `-a /home/biqu/printer_data/comms/klippy.sock`. Если его нет, добавьте в файл `klipper.env`.

### Установка зависимостей

```bash
sudo apt-get install python3-pip git
pip install pyserial
```

### Получение исходного кода

```bash
git clone https://github.com/LLIKIPER/KlipperLCD
cd KlipperLCD
```

### Настройка кода

* Откройте `main.py` и найдите объявление класса `KlipperLCD`:

```python
class KlipperLCD ():
    def __init__(self):
        ...
        LCD("/dev/serial/by-id/usb-1a86_USB_Serial-if00-port0", callback=self.lcd_callback)
        ...
        PrinterData('XXXXXX', URL=("127.0.0.1"), klippy_sock='/home/biqu/printer_data/comms/klippy.sock')
```

>  **Примечание:**
> * при подключении дисплея через преобразователь USB–UART устройство устройство можно получить через команду `"sudo ls /dev/serial/by-id/usb*"`.
> * Если сокет Klipper называется иначе или расположен по другому пути, измените значение `klippy_sock` (`"/home/biqu/printer_data/comms/klippy.sock"`) на фактический путь к файлу сокета.

### Запуск программы

Когда сенсорный дисплей подключён к Raspberry Pi, API-сокет Klipper включён, а класс `KlipperLCD` настроен с учётом вашего подключения, можно запускать программу:

```bash
python3 main.py
```

Поздравляем! Теперь можно пользоваться сенсорным дисплеем.

### Автоматический запуск KlipperLCD при загрузке системы

Если `main.py` находится не по пути `/home/biqu/KlipperLCD/main.py` или используется пользователь, отличный от `biqu`, откройте `KlipperLCD.service` и измените параметры в соответствии со своей системой.

Включите автоматический запуск сервиса при загрузке:

```bash
sudo chmod +x main.py
sudo chmod +x KlipperLCD.service
sudo cp KlipperLCD.service /etc/systemd/system/KlipperLCD.service
sudo chmod 644 /etc/systemd/system/KlipperLCD.service
sudo systemctl daemon-reload
sudo systemctl enable KlipperLCD.service
sudo systemctl start KlipperLCD.service
```

## Консоль

Консоль включена по умолчанию. Чтобы открыть её, нажмите в верхней центральной части главного экрана либо на область миниатюры модели во время печати.

Через консоль можно отправлять команды и просматривать все ответы G-code и сообщения Klipper, которые обычно отображаются на вкладке консоли в Mainsail или Fluidd.

<p float="left">
    <img src="img/console.PNG" height="400">
    <img src="img/console_key.PNG" height="400">
    <img src="img/console_num.PNG" height="400">
</p>

## Миниатюры моделей

KlipperLCD также поддерживает отображение миниатюр моделей!

Чтобы включить создание миниатюр в слайсере, воспользуйтесь [этой инструкцией](https://klipperscreen.readthedocs.io/en/latest/Thumbnails/).

<p float="left">
    <img src="img/thumb1.png" height="400">
    <img src="img/thumb2.png" height="400">
</p>
