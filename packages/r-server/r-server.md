# Конфигурация удаленного сервера для разворачивания Shiny-приложений и выполнения R скриптов

## Ubuntu 24.04 (noble)

### Установка R
```bash
# update indices
sudo apt update -qq
sudo apt full-upgrade

# install two helper packages we need
sudo apt install --no-install-recommends software-properties-common dirmngr
# add the signing key (by Michael Rutter) for these repos
# To verify key, run gpg --show-keys /etc/apt/trusted.gpg.d/cran_ubuntu_key.asc
# Fingerprint: E298A3A825C0D65DFD57CBB651716619E084DAB9
wget -qO- https://cloud.r-project.org/bin/linux/ubuntu/marutter_pubkey.asc | sudo tee -a /etc/apt/trusted.gpg.d/cran_ubuntu_key.asc

# add the repo from CRAN (then record will be added to /etc/apt/sources.list)
sudo add-apt-repository "deb https://cloud.r-project.org/bin/linux/ubuntu $(lsb_release -cs)-cran40/"

# install R itself
sudo apt install --no-install-recommends r-base
sudo apt install r-base-dev                       ; для компиляции пакетов из исходников
````
Для установки и компиляции пакетов могут потребоваться пакеты Ubuntu из репозиториев "backports", которые нужно добавить в файл */etc/apt/sources.list*.

В репозиториях Ubuntu доступен ряд пакетов R с именами, начинающимися с `r-cran-`. Часть этих пакетов поддерживаются на CRAN в актуальном состоянии и входят в состав пакета `r-recommended`. Другая часть пакетов `r-cran-*` обновляется только в релизах Ubuntu. При необходимости обновить один из этих пакетов (например, `r-cran-foo`), нужно убедиться в наличии всех необходимых зависимостей:
```bash
sudo apt-get build-dep r-cran-foo
```

>Пакеты $R$, входящие в состав `r-base` и `r-recommended`, устанавливаются в каталог */usr/lib/R/library* и обновляются обычным обновлениями пакетов Ubuntu.
>Остальные пакеты $R$, доступные как предкомпилированные пакеты Debian `r-cran-\*` и `r-bioc-*`, устанавливаются в каталог */usr/lib/R/site-library*. Установка этих пакетов требует установки пакета `r-base-dev`.
> Пакеты, которые компилируются на сервере, устанавливаются в каталог */usr/local/lib/R/site-library/*.


### Установка RStudio Server
```bash
sudo apt install gdebi-core
wget https://download2.rstudio.org/server/jammy/amd64/rstudio-server-2024.12.1-563-amd64.deb
sudo gdebi rstudio-server-2024.12.1-563-amd64.deb
````
После установки автоматически будет запущен демон `rstudio-server.service`, а сам RStudio Server будет доступен по адресу *http://server-address:8787*.

**Web Links**
* [Download RStudio Server](https://posit.co/download/rstudio-server)

### Установка Shiny Server
При наличии менее 3Gb RAM необходимо обеспечить достаточный объем раздела подкачки:
```bash
cd /
sudo dd if=/dev/zero of=swapfile bs=1M count=3000  ; создать в корне файл размером 3Gb
sudo mkswap swapfile                               ; преобразовать файл в swapfile
sudo swapon swapfile                               ; включить своп
echo "/swapfile none swap sw 0 0" >> /etc/fstab    ; делаем своп доступным при загрузке системы
cat /proc/meminfo                                  ; проверяем в выводе наличие swap-файла
```

> It is important to change the default R package library directory from a *user based* file path to a *system based* file path directory.\
> В конце файла */etc/R/Renviron* закомментировать верхнюю строку `R_LIBS_USER=${R_LIBS_USER:-'%U'}` и снять комментарий (если установлен) со следующей строки `R_LIBS_SITE=${R_LIBS_SITE:-'/usr/local/lib/R/site-library:%S'}`

До установки Shiny-сервера необходимо установить R и пакет `shiny`:
```bash
sudo -i R
> .libPaths()  ; должны быть 3 каталога: /usr/local/lib/R/site-library и /usr/lib/R/{site-library,library}
> install.packages(c('shiny', 'rmarkdown', 'dplyr', 'devtools'))
```
**UPD:** Установку пакета `shiny` рекомендуется делать из командной строки:
```bash
sudo su - -c "R -e "install.packages('shiny', repos='http://cran.rstudio.com/')""
```

Создаем пользователя `rstudio`:
```bash
sudo adduser <user_name>
```

Устанавливаем Shiny-сервер:
```bash
sudo apt install r-base
sudo su - -c "R -e \"install.packages('shiny', repos='https://cran.rstudio.com/')\""
sudo apt install gdebi-core
wget https://download3.rstudio.org/ubuntu-20.04/x86_64/shiny-server-1.5.23.1030-amd64.deb
sudo gdebi shiny-server-1.5.23.1030-amd64.deb
```

После установки автоматически будет запущен демон `shiny-server.service`, а сам Shiny Server будет доступен по адресу *http://server-address:3838*.

**Web Links**
* [Install Shiny Server for R on Ubuntu the Right Way](https://www.r-bloggers.com/2015/10/install-shiny-server-for-r-on-ubuntu-the-right-way)
* [Deploy RStudio And Shiny Server On Ubuntu](https://www.john-mcallister.com/deploy-rstudio-and-shiny-server-on-ubuntu)
* [How to install Shiny server on Ubuntu 22.04](https://www.r-bloggers.com/2023/06/tutorial-how-to-install-shiny-server-on-ubuntu-22-04)
* [Get your Shiny apps online | Posit](https://posit.co/products/open-source/shiny-server)

* [Posit Workbench Administrator Guide](https://docs.posit.co/ide/server-pro/)
