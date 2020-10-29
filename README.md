# Мониторинг соединений к базам 1С Предприятия средствами Zabbix

Для получения информации используются консольная утилита RAC (клиент).

Для обнаружения баз используется LLD Zabbix, используемые версии Zabbix 3.4 - 4.2, ниже не получится, используются зависимые элементы данных.

Добавлена возможность работы с несколькоми кластерами 1С на наблюдаемом сервере. Для информационных баз указывается в каком кластере они живут.

#### Установка под Windows:

<details>
  <summary>1. Установить службу RAS</summary>
  
  <br>
  
  Для установки сервера RAS в качестве службы используется команда:
  `sc create "1C:Enterprise RAS" binpath= "C:\Program Files\1cv8\8.Х.Х.ХХХХ\bin\ras.exe cluster --service" displayname= "1C:Enterprise RAS" start= auto`

  Для запуска - команда:  
  `net start "1C:Enterprise RAS"`

---
</details>
<details>
  <summary>2. Указать системе путь к RAC</summary>
  
  <br>

  В системную переменную PATH необходимо добавить путь где находится исполняемый файл rac.exe:
  `c:\Program Files\1cv8\8.X.XX.XXXX\bin\`  
  <img src="/set_path.JPG" width="400">

  Правой кнопкой на Этот компьютер - Свойства - Дополнительные параметры системы (справа) - Переменные среды - раздел Системные переменные - выбрать переменную Path, изменить, откроется окно Изменение системной перемнной - в поле значение перемнной добавить в конец путь вида `c:\Program Files\1cv8\8.X.XX.XXXX\bin\`, т.е. путь где лежит используемая платформа.
  При использовании 32-х разрядной платформы путь будет вида: `c:\Program Files (x86)\1cv8\8.X.XX.XXXX\bin\`

---
</details>
<details>
  <summary>3. Установить Python  </summary>
  
  <br>
  
  Python можно установить с сайта https://www.python.org, проверьте что бы путь установки соответствовал пути прописанному в файле `1c_zabbix.conf`. Ожидается что интерпретатор будет находится по следующему пути:
  `c:\Program Files\Python37\python.exe`

---
</details>
<details>
  <summary>4. Установить скрипты и конфигурацию Zabbix</summary>
  
  <br>
  
  На сервер с установленным RAC и Zabbix agent необходимо положить файлы `*.py` из каталога `scripts` и файл конфигурации `1c_zabbix.conf`.
  Если в кластере используется администратор кластера, то в `1c_zabbix.conf` следует раскомментировать строки с указанием пользователя и пароля. (При использовании нескольких кластеров, соответственно, этот пользователь должен существовать во всех кластерах).

`ib_list.py` - скрипт возвращающий список баз с указанием uuid кластера  
`sess_list.py` - скрипт возвращает кол-во сессий на присланный uuid базы и uuid кластера

Настройки каталогов:
* zabbix agent - c:\zabbix  
* python - c:\Program Files\Python37\python.exe

---
</details>

#### Установка под Linux
<details>
  <summary>1. Установить службу RAS</summary>
  
  <br>
  
  Для дистрибутивов на основе `systemd` (Debian/Ubuntu, RHEL/CentOS, SLES/OpenSUSE) добавить файл `ras.service` в `/etc/systemd/system/`:  
```
[Unit]
Description=RAS
After=syslog.target
After=network.target

[Service]
Type=simple
User=usr1cv8
Group=grp1cv8
ExecStart=/opt/1C/v8.3/x86_64/ras cluster
KillMode=process

[Install]
WantedBy=multi-user.target
```
Выполнить команды:

`systemctl daemon-reload`  
`systemctl enable --now ras`

---
</details>
<details>
  <summary>2. Указать системе путь к RAC</summary>
  
  <br>
  
  Сделать ссылку:
  
  `ln -s /opt/1C/v8.3/x86_64/rac /usr/local/bin/`
  
  ---
</details>
<details>
  <summary>3. Установить Python  </summary>
  
  <br>
  
  Python на Linux обычно уже есть. Если все же нет - установить python3.
  
  Для Debian/Ubuntu:  
  `apt install python3`
  
  Для RHEL/CentOS:  
  `yum install python3`

---
</details>
<details>
  <summary>4. Установить скрипты и конфигурацию Zabbix</summary>
  
  <br>
  
  На сервер с установленным RAC и Zabbix agent необходимо положить файлы `*.py` из каталога scripts и файл конфигурации `1c_zabbix_linux.conf`. 
  Если в кластере используется администратор кластера, то в `1c_zabbix.conf` следует раскомментировать строки с указанием пользователя и пароля. (При использовании нескольких кластеров, соответственно, этот пользователь должен существовать во всех кластерах).

`ib_list.py` - скрипт возвращающий список баз с указанием uuid кластера  
`sess_list.py` - скрипт возвращает кол-во сессий на присланный uuid базы и uuid кластера

Настройки каталогов:
* zabbix agent - `/etc/zabbix/`

---
</details>

Шаблон для Zabbix 4.2: `1c-zabbix.xml`  
3.4 версию Zabbix более не использую, шаблона для нее у меня нет. Можно поробовать переделать xml под 3.4, но думаю проще обновиться.

Снимаются данные о количестве соединений в разбивке по видам: тонкий клиент, толстый клиент и т.д., число заблокированных объектов, время блокировки и объем данных.  
Для каждой базы создается отдельный график.

При обнаружении ошибок, возникающих вопросах, необходимости добавления новых данных в мониторинг - создайте [issue](https://github.com/kulpin74/zabbix-1c/issues/new).
