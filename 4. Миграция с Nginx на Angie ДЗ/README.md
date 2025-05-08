**Результаты выполнения ДЗ №2.**
Миграция с NGINX на ANGIE

**HOST**
1. Используется VMWare Workstation для развёртывания ВМ.
2. Базовая ОС - Ubuntu 24.04 server minimized
3. Конфигурация ВМ:

   ```
   CPU: 4 core

   RAM: 4 Gb

   DISK: 20 Gb

   NET: bridged 10.254.254.125/27
   ```
   
4. Произведена установка NGINX Stable актуальной версии (1.28.0) и ANGIE 1.9.0

   **NGINX**
   ```
   sudo apt install curl gnupg2 ca-certificates lsb-release debian-archive-keyring
   curl https://nginx.org/keys/nginx_signing.key | gpg --dearmor | sudo tee /usr/share/keyrings/nginx-archive-keyring.gpg >/dev/null
   gpg --dry-run --quiet --no-keyring --import --import-options import-show /usr/share/keyrings/nginx-archive-keyring.gpg
   echo "deb [signed-by=/usr/share/keyrings/nginx-archive-keyring.gpg] http://nginx.org/packages/debian `lsb_release -cs` nginx" | sudo tee /etc/apt/sources.list.d/nginx.list
   echo -e "Package: *\nPin: origin nginx.org\nPin: release o=nginx\nPin-Priority: 900\n" | sudo tee /etc/apt/preferences.d/99nginx
   sudo apt update
   sudo apt install nginx
   nginx -v
   ```
   [NGINX version]()

   **ANGIE**
   ```
   apt-get install -y ca-certificates curl
   sudo curl -o /etc/apt/trusted.gpg.d/angie-signing.gpg https://angie.software/keys/angie-signing.gpg
   echo "deb https://download.angie.software/angie/$(. /etc/os-release && echo "$ID/$VERSION_ID $VERSION_CODENAME") main" | sudo tee /etc/apt/sources.list.d/angie.list > /dev/null
   sudo apt update && sudo apt install angie -y
   angie -v
   ```
   [ANGIE version]()
   
5. На сервер залит конфиг из ДЗ.
    
   [Server NGINX config]()

6. Т.к. модуль BROTLI в NGINX устанавливается из официальных репозиториев только в платной версии, а OSS версию NGINX необходимо компилировать из исходного кода вручную, для запуска NGINX OSS на сервере,
   в конфигурационном файле были закомментированы строки с подгрузкой модуля BROTLI и соответствующие директивы (при миграции конфигурации, данный модуль был учтён и установлен для ANGIE).
   
   [NGINX.conf brotli commented out 1]()
   [NGINX.conf brotli commented out 2]()
   
7. Для упрощения миграции конфигурации файлы конфигурации (nginx.conf и angie.conf) были перенесены в отдельную директорию и сравнены с помощью diff.
    
    [Move nginx.conf & angie.conf]()
   
8. Открываем конфиги на редактирование в vimdiff и, сравнивая значения в директивах, переносим необходимые из конфига nginx.conf в angie.conf. Отсутствующие директивы и блоки также переносим, предварительно
    сверившись с документацией angie и убедившись в том, что переносимые директивы поддерживаются.
    
    [VIMDIFF NGINX/ANGIE]()
    
9. Копируем из каталога /etc/nginx/ каталоги и файлы в /etc/angie/, если в целевом каталоге копируемый файл или каталог уже присутствует, сравниваем и оставляем тот, который необходим и достаточен для штатной
    работы ANGIE.

    [MOVING NGINX contents to ANGIE dir]()
    
10. Проверяем наличие и устанавливаем необходимые модули в ANGIE. Помним, что в учебном конфиге были закомментированы строки относящиеся к модулю brotli. Устанавливаем модуль angie-module-brotli из репозитория
    ANGIE.
    
    [NGINX modules list]()
    [Search ANGIE repository for analog modules]()
    [Install ANGIE modules]()
    
11. Производим предварительную проверку работоспособности ANGIE с полученной конфигурацией.
    
    [New ANGIE.CONF syntax check]()
    
12. Устраняем выявленные ошибки, если их нет - временно указываем порт 8080 в конфигурации для обслуживаемых доменов с целью запуска ANGIE и проверки штатной работы опубликованных ресурсов.
    
    [ANGIE check config and published web resources]()
    
13. После подтверждения штатной работы ANGIE и опубликованных веб-ресурсов, меняем порт в конфигурации ANGIE, выключаем NGINX и запускаем ANGIE одной командой.
    
    [ANGIE replace NGINX]()
    
14. Дополнительно мониторим работу опубликованных веб-ресурсов.
