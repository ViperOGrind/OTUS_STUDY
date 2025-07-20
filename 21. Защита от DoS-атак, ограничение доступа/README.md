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
   
4. Настроено ограничение по User-Agent для поддерживаемых браузеров:

    map $http_user_agent $drop_conn {
   
    default 0;
   
    ~*(Mozilla.+\s\(Windows.+\)\sAppleWebKit.+\sChrome.+) 1;
   
    }

5. Настроены зоны limit_req_zone (для /wp-admin.php и /xmlrpc.php):

   limit_req_zone $binary_remote_addr zone=admin_req:10m rate=1r/s;                               # Request limit
   
   limit_req_zone $binary_remote_addr zone=xmlrpc_req:10m rate=1r/s; 

7. В location /wp-admin.php и /xmlrpc.php настроен сброс соединения, в случае, если User-Agent клиента не соответствует заданному в map $http_user_agent $drop_conn {...}

           if ($drop_conn = 0) {
   
             return 444;
   
           }

8. В location /wp-admin.php и /xmlrpc.php настроена базовая аутентификация:

      auth_basic "Authenticate";
   
      auth_basic_user_file /etc/angie/.auth/htpasswd;

9. В location /wp-admin.php и /xmlrpc.php настроены лимиты количества запросов:

           limit_req zone=admin_req burst=5 nodelay;                                              # Set request limit 1r/s, burst 5r/s
   
           limit_req_log_level warn;                                                              # Request logging level WARN
   
           limit_req_status 406;                                                                  # Request limit return HTTP code 406 - Not Acceptable

10. Конфигурация:

    [wordpress.study.local]()
