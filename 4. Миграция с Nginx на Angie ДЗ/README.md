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
   ![NGINX version](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/4.%20Миграция%20с%20Nginx%20на%20Angie%20ДЗ/Artifacts/HOST/NGINX_v.png)

   **ANGIE**
   ```
   apt-get install -y ca-certificates curl
   sudo curl -o /etc/apt/trusted.gpg.d/angie-signing.gpg https://angie.software/keys/angie-signing.gpg
   echo "deb https://download.angie.software/angie/$(. /etc/os-release && echo "$ID/$VERSION_ID $VERSION_CODENAME") main" | sudo tee /etc/apt/sources.list.d/angie.list > /dev/null
   sudo apt update && sudo apt install angie -y
   angie -v
   ```
   ![ANGIE version](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/4.%20Миграция%20с%20Nginx%20на%20Angie%20ДЗ/Artifacts/HOST/ANGIE_v.png)
   
5. На сервер залит конфиг из ДЗ.
    
   ![Server NGINX config1](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/4.%20Миграция%20с%20Nginx%20на%20Angie%20ДЗ/Artifacts/HOST/NGINX_study_conf1.png)

   ![Server NGINX config2](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/4.%20Миграция%20с%20Nginx%20на%20Angie%20ДЗ/Artifacts/HOST/NGINX_study_conf2.png)

   ![Server NGINX config3](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/4.%20Миграция%20с%20Nginx%20на%20Angie%20ДЗ/Artifacts/HOST/NGINX_study_conf3.png)

   ![Server NGINX config4](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/4.%20Миграция%20с%20Nginx%20на%20Angie%20ДЗ/Artifacts/HOST/NGINX_study_conf4.png)

   ![Server NGINX config5](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/4.%20Миграция%20с%20Nginx%20на%20Angie%20ДЗ/Artifacts/HOST/NGINX_study_conf5.png)

   Предварительно делаем полный бэкап директории /etc/angie

   ![MOVING NGINX contents to ANGIE dir](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/4.%20Миграция%20с%20Nginx%20на%20Angie%20ДЗ/Artifacts/HOST/ANGIE_fullbackup.png)

   Приступаем к переносу файлов и каталогов

   В конфиге nginx.conf настроено проксирование на 127.0.0.1:\[9000-90003\], чтобы сервер при запуске отдал index.html, необходимо закомментировать строку proxy_pass "http:\/\/backend;"
   и раскомментировать "try_files $uri $uri/ =404;" т.к. порты 9000-9003 не слушаются.

   ![Server NGINX respond](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/4.%20Миграция%20с%20Nginx%20на%20Angie%20ДЗ/Artifacts/HOST/NGINX_conf_serve_index.html.png)

   ![Server NGINX respond](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/4.%20Миграция%20с%20Nginx%20на%20Angie%20ДЗ/Artifacts/HOST/NGINX_test_web.png)

6. Т.к. модуль BROTLI в NGINX устанавливается из официальных репозиториев только в платной версии, а OSS версию NGINX необходимо компилировать из исходного кода вручную, для запуска NGINX OSS на сервере,
   в конфигурационном файле были закомментированы строки с подгрузкой модуля BROTLI и соответствующие директивы (при миграции конфигурации, данный модуль был учтён и установлен для ANGIE).
   
   ![NGINX.conf brotli commented out 1](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/4.%20Миграция%20с%20Nginx%20на%20Angie%20ДЗ/Artifacts/HOST/NGINX_conf_brotli1.png)
   
   ![NGINX.conf brotli commented out 2](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/4.%20Миграция%20с%20Nginx%20на%20Angie%20ДЗ/Artifacts/HOST/NGINX_conf_brotli2.png)
   
7. Для упрощения миграции конфигурации файлы конфигурации (nginx.conf и angie.conf) были перенесены в отдельную директорию и сравнены с помощью diff.
    
   ![Move nginx.conf & angie.conf](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/4.%20Миграция%20с%20Nginx%20на%20Angie%20ДЗ/Artifacts/HOST/nginx.conf-angie.conf.png)

   [Diff nginx.conf & angie.conf](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/4.%20Миграция%20с%20Nginx%20на%20Angie%20ДЗ/Artifacts/HOST/nginx.conf-angie.conf_diff.txt)
   
8. Открываем конфиги на редактирование в vimdiff и, сравнивая значения в директивах, переносим необходимые из конфига nginx.conf в angie.conf. Отсутствующие директивы и блоки также переносим, предварительно
    сверившись с документацией angie и убедившись в том, что переносимые директивы поддерживаются.
    
    ![VIMDIFF NGINX/ANGIE](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/4.%20Миграция%20с%20Nginx%20на%20Angie%20ДЗ/Artifacts/HOST/nginx.conf-angie.conf_vimdiff.png)

   После переноса всех необходимых директив и значений из nginx.conf, копируем основной конфиг angie.conf в /etc/angie/angie.conf (оригинальный файл был забэкаплен ранее).

   ```
   cp ./angie.conf /etc/angie/angie.conf
   ```
    
9. Копируем из каталога /etc/nginx/ каталоги и файлы в /etc/angie/, если в целевом каталоге копируемый файл или каталог уже присутствует, сравниваем и оставляем тот, который необходим и достаточен для штатной
    работы ANGIE и опубликованных веб-ресурсов.

    ![MOVING NGINX contents to ANGIE dir](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/4.%20Миграция%20с%20Nginx%20на%20Angie%20ДЗ/Artifacts/HOST/nginx.conf-angie.conf_mime.types.png)

    ![MOVING NGINX contents to ANGIE dir](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/4.%20Миграция%20с%20Nginx%20на%20Angie%20ДЗ/Artifacts/HOST/nginx.conf-angie.conf_mime.types_diff.png)

    ![MOVING NGINX contents to ANGIE dir](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/4.%20Миграция%20с%20Nginx%20на%20Angie%20ДЗ/Artifacts/HOST/ANGIE_cp_mime.types.png)

    ![MOVING NGINX contents to ANGIE dir](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/4.%20Миграция%20с%20Nginx%20на%20Angie%20ДЗ/Artifacts/HOST/ANGIE_snippets.png)

    Т.к. директория /etc/nginx/modules-enabled отсутствует, то можно в конфигурационном файле ANGIE её закомментировать. Но для обратной совместимости этого не делаем - отсутствующие директории ANGIE проигнорирует.

    ![MOVING NGINX contents to ANGIE dir](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/4.%20Миграция%20с%20Nginx%20на%20Angie%20ДЗ/Artifacts/HOST/ANGIE_modules-ena.png)

    ![MOVING NGINX contents to ANGIE dir](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/4.%20Миграция%20с%20Nginx%20на%20Angie%20ДЗ/Artifacts/HOST/ANGIE_sites-ena.png)

    ![MOVING NGINX contents to ANGIE dir](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/4.%20Миграция%20с%20Nginx%20на%20Angie%20ДЗ/Artifacts/HOST/ANGIE_fastcgi2.png)
    
    ![MOVING NGINX contents to ANGIE dir](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/4.%20Миграция%20с%20Nginx%20на%20Angie%20ДЗ/Artifacts/HOST/ANGIE_fastcgi1.png)

    ![MOVING NGINX contents to ANGIE dir](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/4.%20Миграция%20с%20Nginx%20на%20Angie%20ДЗ/Artifacts/HOST/NGINX_ANGIE_transfer_files.png)
    
10. Проверяем наличие и устанавливаем необходимые модули в ANGIE. Помним, что в учебном конфиге были закомментированы строки относящиеся к модулю brotli. Устанавливаем модуль angie-module-brotli из репозитория
    ANGIE.
    
    ![ANGIE modules list](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/4.%20Миграция%20с%20Nginx%20на%20Angie%20ДЗ/Artifacts/HOST/ANGIE_module-brotli.png)
    
    ![ANGIE brotli installed](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/4.%20Миграция%20с%20Nginx%20на%20Angie%20ДЗ/Artifacts/HOST/ANGIE_module-brotli_installed.png)
    
    ![ANGIE brotli start success](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/4.%20Миграция%20с%20Nginx%20на%20Angie%20ДЗ/Artifacts/HOST/ANGIE_module-brotli_launch_success.png)
    
11. Производим предварительную проверку работоспособности ANGIE с полученной конфигурацией. Устраняем выявленные ошибки.
    
    ![New ANGIE.CONF syntax check](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/4.%20Миграция%20с%20Nginx%20на%20Angie%20ДЗ/Artifacts/HOST/ANGIE_test_error.png)

    ![New ANGIE.CONF syntax error](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/4.%20Миграция%20с%20Nginx%20на%20Angie%20ДЗ/Artifacts/HOST/ANGIE_test_error_typo.png)

    ![New ANGIE.CONF syntax success](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/4.%20Миграция%20с%20Nginx%20на%20Angie%20ДЗ/Artifacts/HOST/ANGIE_test_success.png)

    ![New ANGIE.CONF syntax success](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/4.%20Миграция%20с%20Nginx%20на%20Angie%20ДЗ/Artifacts/HOST/ANGIE_launch_success.png)
    
12. Временно указываем порт 8080 в конфигурации для обслуживаемых доменов с целью запуска ANGIE и проверки штатной работы опубликованных ресурсов.
    
    ![ANGIE check config and published web resources](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/4.%20Миграция%20с%20Nginx%20на%20Angie%20ДЗ/Artifacts/HOST/ANGIE_port_change_for_test.png)

    ![ANGIE check config and published web resources](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/4.%20Миграция%20с%20Nginx%20на%20Angie%20ДЗ/Artifacts/HOST/ANGIE_test_web.png)
    
13. После подтверждения штатной работы ANGIE и опубликованных веб-ресурсов, меняем порт в конфигурации ANGIE, выключаем NGINX и запускаем ANGIE одной командой.
    
    ![ANGIE replace NGINX](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/4.%20Миграция%20с%20Nginx%20на%20Angie%20ДЗ/Artifacts/HOST/Switch_NGINX_to_ANGIE.png)

    ```
    systemctl stop nginx && systemctl start angie
    systemctl status nginx angie
    ```

    ![ANGIE serving 80](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/4.%20Миграция%20с%20Nginx%20на%20Angie%20ДЗ/Artifacts/HOST/ANGIE_80_serving.png)

    Отключаем автозагрузку NGINX и включаем ANGIE.

    ![ANGIE serving 80](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/4.%20Миграция%20с%20Nginx%20на%20Angie%20ДЗ/Artifacts/HOST/Switch_NGINX_to_ANGIE_final.png)
    
14. Дополнительно мониторим работу опубликованных веб-ресурсов. Помним, что браузер закэшировал страницу и необходимо очистить кэш, чтобы она корректно подгрузилась.

    ![ANGIE stoped serving 8080](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/4.%20Миграция%20с%20Nginx%20на%20Angie%20ДЗ/Artifacts/HOST/ANGIE_8080_not_responding.png)

    ![ANGIE serving 80](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/4.%20Миграция%20с%20Nginx%20на%20Angie%20ДЗ/Artifacts/HOST/ANGIE_80_serving.png)
