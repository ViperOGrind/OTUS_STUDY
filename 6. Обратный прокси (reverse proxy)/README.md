**Результаты выполнения ДЗ №4.**
Обратный прокси (reverse proxy)

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
   На сервере переразвернута ОС и произведена дефолтная  ANGIE после предыдущего ДЗ.
   
4. В директории /srv/www/wordpress размещены файлы из архива с Wordpress. Директориям, поддиректориям и файлам назначен владелец - пользователь и группа www-data, установлены права 744 для файлов и 755 для директорий.
   
   ![PERMISSIONS](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/6.%20Обратный%20прокси%20(reverse%20proxy)/Artifacts/files_perm.png)

5. Установлен сервер Apache2, отдающий контент.

   ![APACHE2_WP](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/6.%20Обратный%20прокси%20(reverse%20proxy)/Artifacts/Apache2_WP.png)

7. Написан следующий конфиг для Angie в качестве обратного прокси.
   
   [ANGIE_RP](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/6.%20Обратный%20прокси%20(reverse%20proxy)/Artifacts/Angie_RP-conf.png)

8. На проверочном хосте, в файле hosts сделана запись вида: 10.254.254.124 wrdprs.study.local, переходим в браузере по соответствующему FQDN.

   ![WORDPRESS_WEB](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/6.%20Обратный%20прокси%20(reverse%20proxy)/Artifacts/WP.png)

9. Проверяем, что ANGIE статические запросы обрабатывает сам, а динамические отправляет на Apache2, слушающий адрес 127.0.0.1:80.

   ![SEP_REQ](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/6.%20Обратный%20прокси%20(reverse%20proxy)/Artifacts/Angie_RP_WP.png)

