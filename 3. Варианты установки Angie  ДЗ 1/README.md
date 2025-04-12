**Результаты выполнения ДЗ №1.**
Варианты установки Angie

**HOST**
1. Используется VMWare Workstation для развёртывания ВМ.
2. Базовая ОС - Ubuntu 24.04 server minimized
3. Конфигурация ВМ:

   CPU: 4 core

   RAM: 4 Gb

   DISK: 20 Gb

   NET: bridged 10.254.254.124/27
   
5. [Написан простой скрипт](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/3.%20Варианты%20установки%20Angie%20%20ДЗ%201/Artifacts/HOST/Angie_install_script_code.txt) автоматизации установки
6. ![Результат выполнения скрипта](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/3.%20Варианты%20установки%20Angie%20%20ДЗ%201/Artifacts/HOST/Angie_install_script_result.png)
7. [В текстовом виде](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/3.%20Варианты%20установки%20Angie%20%20ДЗ%201/Artifacts/HOST/Angie_install_script_result.txt)
8. [Выведен список](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/3.%20Варианты%20установки%20Angie%20%20ДЗ%201/Artifacts/HOST/Angie_install_modules_list.txt) дополнительных модулей
9. [Установлены](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/3.%20Варианты%20установки%20Angie%20%20ДЗ%201/Artifacts/HOST/Angie_install_modules_console-light_brotli.txt) модули angie-module-console-light и angie-module-brotli
10. [Отредактирован](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/3.%20Варианты%20установки%20Angie%20%20ДЗ%201/Artifacts/HOST/Angie_artifacts_angie.conf.txt) файл конфигурации /etc/angie/angie.conf
11. [Создан файл](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/3.%20Варианты%20установки%20Angie%20%20ДЗ%201/Artifacts/HOST/Angie_artifacts_under_dev.conf.txt) конфигурации сервера в /etc/angie/http.d/under_dev.conf
12. [Проверена работоспособность](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/3.%20Варианты%20установки%20Angie%20%20ДЗ%201/Artifacts/HOST/Angie_artifacts_test_angie.txt) конфигурации ANGIE
13. Произведён перезапуск
    `
    $ systemctl reload angie
    `
14. и проверка штатной работы Angie
    `
    $ systemctl status angie
    `
15. Angie перезагрузил конфигурацию [успешно](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/3.%20Варианты%20установки%20Angie%20%20ДЗ%201/Artifacts/HOST/Angie_artifacts_reload_angie.txt)
16. Т.к. в локальной сети (сетевой интерфейс у ВМ в режиме bridged) уже используется reverse proxy (на базе ANGIE) для доступа к локальным сетевым ресурсам по FQDN - в конфигурации under_dev.conf было настроено ограничение по IP RP
17. На RP были внесены изменения в конфигурацию, позволяющие осуществить доступ к развернутому учебному стенду ANGIE по FQDN http://angie-oss.study.local/ :

    а) ![Добавлен upstream](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/3.%20Варианты%20установки%20Angie%20%20ДЗ%201/Artifacts/HOST/Angie_artifacts_reverse_proxy_upstreams.png)

    б) ![Написан файл конфигурации в /etc/angie/http.d/](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/3.%20Варианты%20установки%20Angie%20%20ДЗ%201/Artifacts/HOST/Angie_artifacts_reverse_proxy_config.png)
    
19. ![На проверочном хосте сделана запись в HOSTS](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/3.%20Варианты%20установки%20Angie%20%20ДЗ%201/Artifacts/HOST/Angie_artifacts_hosts_config.png)
20. ![Проверена доступность / через RP](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/3.%20Варианты%20установки%20Angie%20%20ДЗ%201/Artifacts/HOST/Angie_artifacts_angie-oss.study.local.png)
21. ![Проверена доступность /console через RP](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/3.%20Варианты%20установки%20Angie%20%20ДЗ%201/Artifacts/HOST/Angie_artifacts_angie-oss.study.local_console.png)
22. ![Проверена доступность кастомной страницы 404 через RP](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/3.%20Варианты%20установки%20Angie%20%20ДЗ%201/Artifacts/HOST/Angie_artifacts_angie-oss.study.local_test404.png)
23. Доступность кастомной страницы 404 означает также и доступность страницы 50х

**DOCKER**

1. На той же ВМ установлен Docker
   `
   # Add Docker's official GPG key:
   sudo apt-get update
   sudo apt-get install ca-certificates curl
   sudo install -m 0755 -d /etc/apt/keyrings
   sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
   sudo chmod a+r /etc/apt/keyrings/docker.asc

   # Add the repository to Apt sources:
   echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
   sudo apt-get update

   # Install Docker
   sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

2. [Для корректного проброса конфигураций, была создана директория /home/angie/conf с файлом angie.conf](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/3.%20Варианты%20установки%20Angie%20%20ДЗ%201/Artifacts/DOCKER/Angie_artifacts_docker_angie.conf.txt)
3. [Для того. чтобы не писать отдельные конфигурационные файлы под страничку-заглушку, была также создана папка /home/angie/conf/http.d в которой был создан файл конфигурации test.conf](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/3.%20Варианты%20установки%20Angie%20%20ДЗ%201/Artifacts/DOCKER/Angie_artifacts_docker_test.conf.txt)
4. [Для установки ANGIE в контейнере, выноса за пределы контейнера файла с конфигурациями и проброса портов (8080 на 80 и 8443 на 443) использовалась следующая команда](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/3.%20Варианты%20установки%20Angie%20%20ДЗ%201/Artifacts/DOCKER/Angie_artifacts_docker_deploy.txt)
5. ![Результат выполнения команды - запущен контейнер с ANGIE](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/3.%20Варианты%20установки%20Angie%20%20ДЗ%201/Artifacts/DOCKER/Angie_artifacts_docker_ps.png)
6. ![При этом конфигурация для console-light храниться в самом контейнере](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/3.%20Варианты%20установки%20Angie%20%20ДЗ%201/Artifacts/DOCKER/Angie_artifacts_docker_console-light.png)
7. На RP также были добавлены:
   а) ![Upstraem](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/3.%20Варианты%20установки%20Angie%20%20ДЗ%201/Artifacts/DOCKER/Angie_artifacts_reverse_proxy_upstreams_docker.png)
   б) [Написан файл конфигурации под FQDN angie-docker.study.local](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/3.%20Варианты%20установки%20Angie%20%20ДЗ%201/Artifacts/DOCKER/Angie_artifacts_reverse_proxy_docker_config.txt)
8. ![На проверочном хосте сделана запись в HOSTS](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/3.%20Варианты%20установки%20Angie%20%20ДЗ%201/Artifacts/DOCKER/Angie_artifacts_hosts.png)
9. ![Проверена доступность / по FQDN angie-docker.study.local](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/3.%20Варианты%20установки%20Angie%20%20ДЗ%201/Artifacts/DOCKER/Angie_artifacts_reverse_angie-docker.study.local.png)
10. ![Проверена доступность кастомной страницы 404](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/3.%20Варианты%20установки%20Angie%20%20ДЗ%201/Artifacts/DOCKER/Angie_artifacts_reverse_angie-docker.study.local_test404.png)
11. ![При этом, в контейнере с ANGIE работает собственная страница console-light](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/3.%20Варианты%20установки%20Angie%20%20ДЗ%201/Artifacts/DOCKER/Angie_artifacts_reverse_angie-docker.study.local_console.png)
