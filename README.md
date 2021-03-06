extFilter
===========
Программа для блокирования сайтов из списка РКН с использованием DPDK.

Функционал
----------
Программа осуществляет блокировку сайтов путем анализа зеркалированного трафика от пользователей.
В случае нахождения вызываемого абонентом HTTP ресурса в списке блокировки, пользователю отсылается редирект на специальную страницу или сбрасывается соединение.
Блокировка HTTPS ресурсов осуществляется на основе имени сервера в client hello запросе от пользователя или ip адреса сервера, если имени сервера нет в client hello запросе.
В случае нахождения вызываемого абонентом HTTPS ресурса в списке блокировки, соединение с данными ресурсом будет сброшено.
Для отправки данных в сторону пользователя необходим настроенный ip интерфейс под управлением ядра ОС.
Дополнительно присутствует функция оповещения, которая позволяет информировать пользователей путем периодической переадресации на нужную страницу.

Требования
----------
Для сборки программы необходимы следующие библиотеки и программы:

- Poco >= 1.6
- DPDK = 17.05.01
- git

Сборка
------
- [Установить DPDK](http://dpdk.org/doc/quick-start)
- Сгенерировать configure
```bash
./autogen.sh
```
- Запустить configure
```bash
./configure --with-dpdk_target=<target> --with-dpdk_home=<path_to_compiled_dpdk>
```
- Скомпилировать программу
```bash
make
```

Настройка DPDK
--------------
Для работы DPDK необходимо настроить huge-pages и подключить необходимые сетевые адаптеры в DPDK.

Пример настройки для CentOS 7:

- Создаем в каталоге /usr/lib/tuned папку dpdk-tune

- Создаем в dpdk-tune файл tuned.conf:
```
[main]
include=latency-performance

[bootloader]
cmdline=isolcpus=1,2,3 default_hugepagesz=1G hugepagesz=1G hugepages=4
```
isolcpus=1,2,3 - Какие ядра освободить для работы dpdk/extfilter.
default_hugepagesz=1G hugepagesz=1G - Размер страницы памяти для dpdk/extfilter.
hugepages=4 - Количество страниц памяти для dpdk/extfilter (в данном примере под dpdk/extfilter будет выделено 4 Гигабайта памяти).

- Активируем профиль
```bash
tuned-adm profile dpdk-tune
```

- Отправляем сервер на перезагрузку.

- Загружаем необходимые драйвера
```bash
modprobe uio
insmod /path/to/dpdk/build/kmod/igb_uio.ko
```

- Подключаем сетевую карту к dpdk
```bash
/path/to/dpdk/usertools/dpdk-devbind.py --bind=igb_uio dev_pci_num
```
Получить dev_pci_num можно при помощи команды:
```bash
/path/to/dpdk/usertools/dpdk-devbind.py --status
```


Запуск
------
Параметры работы программы задаются в конфигурационном файле.
Для запуска программы необходимо указать путь к конфигурационному .ini файлу (--config-file в командой строке). Для запуска в режиме daemon необходимо указать ключи --daemon и --pidfile=/path/to/file.pid

Файлы списков блокировки
------------------------
Файлы с данными для блокировки (домены, url и т.д.) должны быть в формате [nfqfilter](https://github.com/max197616/nfqfilter).

Обновление списков блокировки
-----------------------------
Для загрузки обновленных списков блокировки без перезапуска программы необходимо подать сигнал HUP.


Поддержка проекта
------
Если вы используете программу и она вам нравится, то вы можете отблагодарить автора через Yandex.Money 410014706910423
