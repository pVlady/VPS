# Автозапуск скриптов при загрузке системы
После создания скрипта командой `chmod +x script.sh` нужно сделать его запускаемым.
## Ubuntu
**Автозапуск с использованием `systemd`**\
   В каталоге */etc/systemd/system/* нужно создать службу, которая будет запускать скрипт, напр., файл *script.service* :
   ```ini
   [Unit]
   Description="Run Script After Restart"
   After=network.target
   [Service]
   ExecStart=/path/to/script.sh
   [Install]
   WantedBy=default.target
   ```
   Далее выполняем команды:
   ```bash
   sudo systemctl daemon-reload
   sudo systemctl enable myscript.service
   sudo systemctl start myscript.service
   ```
   
**Автозапуск с использованием *rc.local***\
   Файл *rc.local* — это скрипт, который выполняется в конце процесса загрузки системы.\
   Для выполнения скрипта в конце загрузки системы нужно в файле *rc.local* добавить строку запуска скрипта перед строкой `exit 0` и выполнить команды:
   ```bash
   sudo chmod +x /etc/rc.local
   sudo reboot
   ``` 
   
**Автозапуск с использованием *init.d***\
   Этот способ считается устаревшим.
   Для запуска скрипта нужно создать запускающий скрипт файл, сделать его исполняемым и поместить в каталог */etc/init.d/*:
   ```bash
   cd /path/to/script_directory
   ./script.sh
   ```
   Чтобы скрипт запускался при старте системы нужно выоплнить:
   ```bash
   sudo update-rc.d myscript defaults
   sudo reboot
   ```


