# wb-mqtt-erc22

MQTT-драйвер для ERC22-485 конвертера. Читает данные с последовательного порта и публикует их в MQTT брокер.

## Описание

Программа подключается к ERC22-485 конвертеру через последовательный порт (RS485/USB), получает данные о нажатии кнопок и публикует их в MQTT брокер в формате, совместимом с Wiren Board.

## Установка

### 1. Копирование файлов

```bash
# Скопировать исполняемый файл
sudo cp wb-mqtt-erc22 /usr/bin/wb-mqtt-erc22
sudo chmod +x /usr/bin/wb-mqtt-erc22

# Скопировать файл сервиса systemd
sudo cp debian/wb-mqtt-erc22.service /lib/systemd/system/

# Скопировать конфигурационный файл
sudo cp wb-mqtt-erc22.conf /etc/wb-mqtt-erc22.conf
```

### 2. Настройка конфигурации

Отредактируйте файл `/etc/wb-mqtt-erc22.conf` для вашей системы:

```bash
sudo nano /etc/wb-mqtt-erc22.conf
```

Пример правильной конфигурации:
```json
{
    "debug": false,
    "verbose": false,
    "ports": [
        {
            "path": "/dev/ttyRS485-1",
            "baud_rate": 9600,
            "parity": "N",
            "data_bits": 8,
            "stop_bits": 1,
            "enabled": true
        }
    ]
}
```

### 3. Определение порта устройства

Найдите правильный порт для вашего ERC22 устройства:

```bash

# Или все последовательные порты
ls /dev/tty*

# Подключите устройство и выполните команду снова, чтобы увидеть новый порт
```

### 4. Активация сервиса

```bash
# Перезагрузить конфигурацию systemd
sudo systemctl daemon-reload

# Включить автозапуск сервиса
sudo systemctl enable wb-mqtt-erc22.service

# Запустить сервис
sudo systemctl start wb-mqtt-erc22.service
```

## Управление сервисом

### Основные команды

```bash
# Проверить статус сервиса
sudo systemctl status wb-mqtt-erc22.service

# Запустить сервис
sudo systemctl start wb-mqtt-erc22.service

# Остановить сервис
sudo systemctl stop wb-mqtt-erc22.service

# Перезапустить сервис
sudo systemctl restart wb-mqtt-erc22.service

# Отключить автозапуск
sudo systemctl disable wb-mqtt-erc22.service
```

### Просмотр логов

```bash
# Посмотреть логи в реальном времени
sudo journalctl -u wb-mqtt-erc22.service -f

# Посмотреть все логи сервиса
sudo journalctl -u wb-mqtt-erc22.service

# Посмотреть последние 50 строк логов
sudo journalctl -u wb-mqtt-erc22.service -n 50

# Логи за последние 10 минут
sudo journalctl -u wb-mqtt-erc22.service --since "10 minutes ago"
```

## Проверка работы

### 1. Проверка сервиса

```bash
# Убедиться, что сервис в списке активных
systemctl list-units --type=service | grep wb-mqtt-erc22

# Проверить, что процесс запущен
ps aux | grep wb-mqtt-erc22
```

### 2. Ручной запуск для диагностики

Если сервис не работает, можно запустить программу вручную для диагностики:

```bash
# Остановить сервис
sudo systemctl stop wb-mqtt-erc22.service

# Запустить программу вручную
sudo /usr/bin/wb-mqtt-erc22

# Запустить программу вручную с другим конфигурационным файлом
sudo /usr/bin/wb-mqtt-erc22 /root/wb-mqtt-erc22.conf
```

При ручном запуске вы увидите все сообщения программы в консоли.

## Конфигурация

### Параметры порта

В файле `/etc/wb-mqtt-erc22.conf` можно настроить:

- `path` - путь к последовательному порту (например: `/dev/ttyRS485-1`)
- `baud_rate` - скорость передачи (по умолчанию: 9600)
- `parity` - контроль четности: `N` (нет), `E` (четная), `O` (нечетная)
- `data_bits` - биты данных (обычно 8)
- `stop_bits` - стоп-биты (обычно 1)
- `enabled` - включить/отключить порт (`true`/`false`)

### Несколько портов

Можно настроить работу с несколькими ERC22 устройствами:

```json
{
    "debug": false,
    "verbose": true,
    "ports": [
        {
            "path": "/dev/ttyRS485-1",
            "enabled": true
        },
        {
            "path": "/dev/ttyRS485-2", 
            "enabled": true
        }
    ]
}
```

После изменения файла сервиса выполните:
```bash
sudo systemctl daemon-reload
sudo systemctl restart wb-mqtt-erc22.service
```

### Режимы отладки

- `debug: true` - включает вывод сырых данных от устройства в MQTT
- `verbose: true` - включает подробные сообщения в журнал

## MQTT топики

Программа публикует данные в следующие топики:

- `/devices/erc22_N/controls/XXXXXXXX` - состояние кнопки (где N - номер устройства, XXXXXXXX - ID кнопки)
- `/devices/erc22_N/meta` - метаданные устройства
- `/devices/erc22_N/meta/error` - ошибки устройства
- `/devices/erc22_N/meta/debug` - отладочная информация

## Решение проблем

### Сервис не запускается

1. Проверьте логи: `sudo journalctl -u wb-mqtt-erc22.service`
2. Проверьте конфигурацию: `cat /etc/wb-mqtt-erc22.conf`
3. Проверьте права доступа к порту: `ls -la /dev/tty*`
4. Запустите вручную для диагностики


## Системные требования

- Linux с systemd
- Python 3.6+
- Библиотеки Python: `pyserial`, `paho-mqtt`, `wb_common` (специфика Wiren Board)
- Доступ к последовательному порту
- MQTT брокер (например, Mosquitto)
