**Результаты выполнения ДЗ №5.**

Оптимизация производительности веб-сервисов

**HOST**
1. Используется VMWare Workstation для развёртывания ВМ.
2. Базовая ОС - Ubuntu 24.04 server minimized
3. Конфигурация ВМ:

   ```
   CPU: 8 core

   RAM: 8 Gb

   DISK: 50 Gb

   NET: bridged 10.254.254.124/27
   ```

   Используется свежая инсталляция от предыдущего ДЗ.
   
4. Произведена настройка сетевого стека ОС

   [NETWORK](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/10.%20Оптимизация%20производительности%20веб-сервисов/Artifacts/sysctl.conf)
   
   а) Установлен параметр **net.ipv4.tcp_slow_start_after_idle = 0** для оптимизации скорости ответа сервера,
   
   б) Установлен параметр **net.ipv4.tcp_mtu_probing = 1** для оптимизации MTU на пути прохождения сетевых пакетов,
   
   в) Установлен параметр **net.ipv4.tcp_notsent_lowat = 55506** для оптимизации скорости ответа сервера,
   
   г) Установлен параметр **net.ipv4.ip_local_port_range = 1024 64535** для увеличения количества используемых локальных портов,
   
   д) Установлен параметр **net.ipv4.tcp_tw_reuse = 1** для включения переиспользования соединений,
   
   е) Параметр **#net.netfilter.nf_conntrack_max_ = 1048576** не используется, т.к. загрузка сервера на тестовом стенде по трафику минимальна,
   
   ж) Параметр **#net.core.netdev_max_backlog = 10000** не используется, т.к. сетевой адаптер сервера на тестовом стенде поддерживает макимальную скорость 1000 Мбит/с,
   
   з) Параметр **#net.core.somaxconn = 81920** не используется, т.к. на сервере слишком мало соединений,
   
   и) Параметр **#net.ipv4.tcp_congestion_control = bbr** не используется, т.к. запросов из сетей с высокими потерями не поступает,
   
   к) Параметр **#net.core.default_qdisc = fq** не используется, т.к. запросов из сетей с высокими потерями не поступает.
   

5. Настройки в основном файле конфигурации angie.conf:

   [ANGIE.CONF](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/10.%20Оптимизация%20производительности%20веб-сервисов/Artifacts/angie.conf)

   а) Директива **worker_processes  1;** - т.к. ANGIE выступает как прокси-сервер для Apache2, обслуживающего веб-приложение Wordpress, то выставляем ограничение в 1 процесса воркеров для ANGIE, которыq будtт занимать одно из 8 ядер процессора, в то время как остальные ядра будут свободны для Apache2 и других сервисов,
   
   б) Директива **worker_rlimit_nofile 65535;** - для современных сайтов рекомендуется увеличивать значение по умолчанию в 1000, т.к. каждое открытие сайта это множество запросов к серверу, также данное значение соответствует ulimit ОС,
   
   в) Директива **worker_cpu_affinity 1;** - привязываем процесс воркера к первому ядру процессора,
   

   Подключаем модуль Brotli для поддержки эффективного сжатия:
   
    **load_module modules/ngx_http_brotli_filter_module.so;**
   
    **load_module modules/ngx_http_brotli_static_module.so;**

   г) Директива **worker_connections 4096;** - устанавливает рассчетное количество соединений для воркера,

   д) Директива **use epoll;** - устанавливает механизм обработки соединений специфичный для Linux систем,

   е) Директива **multi_accept on;** - устанавливает режим, при котором воркер будет принимать сразу все новые соединения,

   ж) Директива **aio threads;** - устанавливает режим, при котором читать и отправлять файлы можно в многопоточном режиме, не блокируя при этом рабочий процесс,

   з) Директива **output_buffers 4 32k;** - устанавливает число и размер буферов, используемых при чтении ответа с диска,

   и) Директива **keepalive_timeout  65;** - устанавливает таймаут, в течение которого неактивное постоянное соединение с сервером группы не будет закрыто,

   к) Директива **server 127.0.0.1:1443 max_fails=3 fail_timeout=30s;** - устанавливает адрес бэкенд сервера, задает число неудачных попыток связи с сервером, в течение заданного времени для того, чтобы сервер считался недоступным, после чего он будет проверен через то же время,

   л) Директива **keepalive_timeout 60s;** - задает таймаут, в течение которого неактивное постоянное соединение с сервером группы не будет закрыто,

   м) Директива **keepalive_requests 1000;** - задает максимальное число запросов, которые можно сделать по одному постоянному соединению,

   н) Директива **keepalive 32;** - задает кэш соединений для группы серверов, устанавливая максимальное число неактивных постоянных соединений, которые будут сохраняться в кэше каждого рабочего процесса. При превышении этого числа - наиболее старые давно неиспользуемые соединения закрываются.

6. Настроим системные лимиты:

   [SYSCTL.CONF](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/10.%20Оптимизация%20производительности%20веб-сервисов/Artifacts/sysctl.conf)

   Открываем файл **/etc/security/limits.conf** на редактирование и прописываем значения:

   **soft nofile 65535**
   
   **hard nofile 65535**

   Применяем командой **ulimit -n 65535** (или перезагрузкой хоста).

7. Настраиваем ограничение использования ядер процессора и оперативной памяти для сервиса Apache2:

   [APACHE2.SERVICE](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/10.%20Оптимизация%20производительности%20веб-сервисов/Artifacts/apache2.service)

   **CPUAffinity=1-2**

   **MemoryMax=2G**

8. Настраиваем ограничение использования ядер процессора и оперативной памяти для сервиса PHP8.3-FPM (выделяем больше, т.к. PHP8.3-FPM будет обрабатывать динамические запросы):

   [PHP8.3-FPM.SERVICE](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/10.%20Оптимизация%20производительности%20веб-сервисов/Artifacts/php8.3-fpm.service)

   **CPUAffinity=3-5**

   **MemoryLimit=4G**

9. Настраиваем оптимизацию для сервиса PHP8.3-FPM:

   [MPM_EVENT.CONF](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/10.%20Оптимизация%20производительности%20веб-сервисов/Artifacts/mpm_event.conf)

   **StartServers             2**
   **MinSpareThreads         25**
   **MaxSpareThreads         75**
   **ThreadLimit             32**
   **ThreadsPerChild         32**
   **MaxRequestWorkers      128**
   **MaxConnectionsPerChild  10000**
   **ServerLimit             4**

10. Настраиваем ограничение использования ядер процессора и оперативной памяти для сервиса MariaDB:

   [MARIADB.SERVICE](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/10.%20Оптимизация%20производительности%20веб-сервисов/Artifacts/mariadb.service)

   **CPUAffinity=1**

   **MemoryMax=1G**

11. Настраиваем оптимизацию для сервиса MariaDB:

   [60-OPTIMIZED.CNF](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/10.%20Оптимизация%20производительности%20веб-сервисов/Artifacts/60-optimized.cnf)

   **thread_handling = pool-of-threads**

   **thread_pool_size = 1**

   **thread_pool_max_threads = 100**

   **innodb_buffer_pool_size = 1G**

   **innodb_buffer_pool_chunk_size = 128M**

   **innodb_read_io_threads = 2**

   **innodb_write_io_threads = 2**

   **innodb_flush_method = O_DIRECT**

12. Оптимизации в конфигурации Wordpress для Angie:

   [WORDPRESS.CONF](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/10.%20Оптимизация%20производительности%20веб-сервисов/Artifacts/wordpress.conf)

   a) Редирект с HTTP на HTTPS,

   б) Настроена служба OCSP респондера на базе openssl-ocsp,

   в) Ограничение поддерживаемых SSL протоколов: **ssl_protocols TLSv1.2 TLSv1.3;**,

   г) Ограничение поддерживаемых SSL шифров: **ssl_ciphers 'ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384';**,

   д) Кэш SSL сессии: **ssl_session_cache shared:SSL:10m;**,

   е) Таймаут SSL сессии: **ssl_session_timeout 1d;**,

   ж) Отключены тикеты SSL сессий: **ssl_session_tickets off;**,

   з) Настроена проверка OCSP: **ssl_stapling on;** **ssl_stapling_verify on;**,

   и) Спрятаны заголовки сервера (вместо параметров в заголовках сервера устанавливаются заголовки прокси): 
   
  **proxy_hide_header X-Powered-By;
    proxy_hide_header Strict-Transport-Security;
    proxy_hide_header X-Frame-Options;
    proxy_hide_header X-Content-Type-Options;
    proxy_hide_header X-XSS-Protection;
    proxy_hide_header Referrer-Policy;
    proxy_hide_header Content-Security-Policy;**

  к) Установлены следующие заголови:

  **add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;
    add_header Content-Security-Policy "default-src 'self' https://wordpress.study.local;
        script-src 'self' 'unsafe-inline' 'unsafe-eval' 'wasm-unsafe-eval' 'inline-speculation-rules' https://wordpress.study.local;
        worker-src 'self' blob: https://wordpress.study.local;
        style-src 'unsafe-inline' https://wordpress.study.local;
        img-src 'unsafe-inline' data: blob: https://wordpress.study.local https://secure.gravatar.com https://ps.w.org;
        font-src 'unsafe-inline' data: https://wordpress.study.local;
        connect-src 'self' *.study.local;
        frame-src 'self' blob:;
        frame-ancestors 'self';
        form-action 'self';";**

   л) Настроено сжатие Brotli (в случае отсутствия поддержки Brotli в браузере будет использоваться Gzip):

  **brotli on;                   # enable Brotli
    brotli_static on;            # serve pre-compressed files if exist
    brotli_comp_level 4;		   # compression level
    brotli_types 			         # compression mime types
        text/plain 
        text/css 
        text/javascript 
        application/javascript 
        application/x-javascript 
        application/json 
        application/xml 
        application/xml+rss 
        text/xml 
        image/svg+xml
        font/woff
        font/woff2
        font/ttf
        font/otf
        image/webp
        image/bmp;**

  м) Сжатие Gzip:

  **gzip on;				# enable gzip
    gzip_static on;			# serve pre-compressed files if exist
    gzip_vary on;			# allow Header Vary: Accept-Encoding
    gzip_proxied any;			# allow all proxied requests to be gzipped
    gzip_comp_level 4;			# compression level
    gzip_types 
        text/plain
        text/css
        text/javascript
        application/javascript
        application/x-javascript  
        application/json 
        application/xml
        application/xml+rss
        text/xml
        image/svg+xml
        font/woff
        font/woff2
        font/ttf
        font/otf;**


  н) Оптимизация буферов для проксирования:

  **proxy_buffer_size 16k;
    proxy_buffers 4 32k;
    proxy_busy_buffers_size 64k;
    proxy_temp_file_write_size 64k;**

 о) Оптимизация прокси таймаутов:

  **proxy_connect_timeout 90;
    proxy_send_timeout 90;
    proxy_read_timeout 90;
    send_timeout 90;**

 п) Рут локейшн с настройкам кэша:

 **location / {
        proxy_pass https://wordpress_backend$request_uri;                                      # set proxy_pass to backend upstream for location
        proxy_cache wordpress_cache;                                                           # set cache zone for location
        proxy_cache_bypass $http_cache_control;                                                # set cache bypass for location
        proxy_no_cache $http_cache_control;                                                    # set no cache
        proxy_cache_revalidate on;                                                             # enable cache revalidate
        proxy_cache_lock on;                                                                   # enable cache lock
        proxy_cache_use_stale error timeout updating http_500 http_502 http_503 http_504;      # set cache stale for 50x error pages
        proxy_cache_background_update on;                                                      # enable cache background upgrade
        proxy_cache_methods GET HEAD;                                                          # set cache for GET and HEAD methods
    }**

р) Кэширование статических файлов:

**location ~* \.(jpg|jpeg|png|gif|ico|css|js|svg|woff|woff2)$ {
        proxy_pass https://wordpress_backend$request_uri;                                      # set proxy_pass to backend upstream for location
        expires 365d;                                                                          # set cache expiration for static files
        add_header Cache-Control "public, no-transform";                                       # set Cache-Control header
        access_log off;                                                                        # disable access log for static files
        proxy_cache wordpress_cache;                                                           # set cache zone for location
        proxy_cache_valid 200 302 365d;                                                        # set cache valid time for 200 and 302
    }**

с) Отключение логирования для favicon.ico и robots.txt:

**location = /favicon.ico {
        log_not_found off;
        access_log off;
    }**

**location = /robots.txt {
        log_not_found off;
        access_log off;
    }**

т) Запрет доступа к скрытым директориям:

**location ~ /\. {
        deny all;
        access_log off;
        log_not_found off;
    }**

13. Конфигурация Wrodpress для Apache2:

**# Disable server signature
ServerSignature Off
ServerTokens Prod

<VirtualHost 127.0.0.1:1443>
    ServerName wordpress.study.local
    ServerAlias www.wordpress.study.local
    
    DocumentRoot /var/www/wordpress
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/angie/.ssl/wordpress.study.local/wildcard.study.local.crt
    SSLCertificateKeyFile /etc/ssl/angie/.ssl/wordpress.study.local/wildcard.study.local.key
    
    <Directory /var/www/wordpress>
        Options FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
    
    <FilesMatch "\.php$">
        SetHandler "proxy:unix:/run/php/php8.3-fpm-wordpress.sock|fcgi://localhost"
    </FilesMatch>
    
    ErrorLog ${APACHE_LOG_DIR}/wordpress_error.log
    CustomLog ${APACHE_LOG_DIR}/wordpress_access.log combined
    
    # Security Headers
    Header always set X-Content-Type-Options "nosniff"
    Header always set X-Frame-Options "SAMEORIGIN"
    Header always set X-XSS-Protection "1; mode=block"
    Header always set Referrer-Policy "strict-origin-when-cross-origin"
    
</VirtualHost>**

 14. Скриншот работающего сайта:

     [wordpress.study.local](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/10.%20Оптимизация%20производительности%20веб-сервисов/Artifacts/Wordpress.png)

 15. Скриншот console light с кэшированием:

     [angie.study.local/console/](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/10.%20Оптимизация%20производительности%20веб-сервисов/Artifacts/Console_status.png)
