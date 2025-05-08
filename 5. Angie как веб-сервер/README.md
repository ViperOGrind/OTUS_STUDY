**Результаты выполнения ДЗ №3.**
ANGIE как веб-сервер

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
   На сервере восстановлена дефолтная конфигурация ANGIE после предыдущего ДЗ.
   
4. В директории /var/www/html/new_site размещены файлы из архива к ДЗ. Директориям, поддиректориям и файлам назначен владелец - пользователь и группа angie, установлены права 755.
   
   ![SITE](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/5.%20Angie%20как%20веб-сервер/Artifacts/site.png)

5. Написан следующий конфиг для сайта.
   
   [SITE config](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/5.%20Angie%20как%20веб-сервер/Artifacts/test5.study.local.conf)

   Проверен синтаксис конфига, запущен ANGIE.

   ![ANGIE pre-check and start](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/5.%20Angie%20как%20веб-сервер/Artifacts/ANGIE_status.png)

6. На проверочном хосте, в файле hosts сделана запись вида: 10.254.254.125 test5.study.local, переходим в браузере по соответствующему FQDN.

   ![SITE web](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/5.%20Angie%20как%20веб-сервер/Artifacts/test5.study.local_root.png)

7. Проверяем постоянный редирект.

   ![301](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/5.%20Angie%20как%20веб-сервер/Artifacts/test5.study.local_301.png)

8. Проверяем временный редирект.

   ![302](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/5.%20Angie%20как%20веб-сервер/Artifacts/test5.study.local_302.png)

9. Проверяем внутренний редирект.

   ![INTERNAL_REDIRECT](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/5.%20Angie%20как%20веб-сервер/Artifacts/test5.study.local_internal_redirect.png)

10. Проверяем страницу ошибки.

    ![404](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/5.%20Angie%20как%20веб-сервер/Artifacts/test5.study.local_error_page.png)

11. Отключаем отдачу версии ANGIE.

    ![SERVER TOKENS OFF](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/5.%20Angie%20как%20веб-сервер/Artifacts/test5.study.local_root_server_tokens_off.png)
