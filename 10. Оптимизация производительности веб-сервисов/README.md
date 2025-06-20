**Результаты выполнения ДЗ №5.**

Оптимизация производительности веб-сервисов

**HOST**
1. Используется VMWare Workstation для развёртывания ВМ.
2. Базовая ОС - Ubuntu 24.04 server minimized
3. Конфигурация ВМ:

   ```
   CPU: 4 core

   RAM: 4 Gb

   DISK: 20 Gb

   NET: bridged 10.254.254.124/27
   ```
   Используется инсталляция от предыдущего ДЗ.
   
4. Произведена настройка сетевого стека ОС

   ![NETWORK]()

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
   

6. Настройкив основном файле конфигурации angie.conf:

   ![ANGIE.CONF]()

   а) Директива **worker_processes  2;** - т.к. ANGIE выступает как прокси-сервер для Apache2, обслуживающего веб-приложение Wordpress, то выставляем ограничение в 2 процесса воркеров для ANGIE, которые будут занимать каждый по одному из 4 ядер процессора, в то время как остальные ядра будут свободны для Apache2,
   
   б) Директива **worker_rlimit_nofile 65535;** - для современных сайтов рекомендуется увеличивать значение по умолчанию в 1000, т.к. каждое открытие сайта это множество запросов к серверу, также данное значение соответствует ulimit ОС,
   
   в) Директива **worker_cpu_affinity auto;** - позволяет автоматически привязывать процессы воркеров к доступным ядрам процессоров, также можно ограничить процессоры, доступные для автоматической привязки с помощью специальной маски (необязательно): 01010101 (пример: **worker_cpu_affinity auto 01010101;**).
   

   Подключаем модуль Brotli для поддержки эффективного сжатия:
   
    **load_module modules/ngx_http_brotli_filter_module.so;**
   
    **load_module modules/ngx_http_brotli_static_module.so;**

   Настроим системные лимиты:

   Открываем файл **/etc/security/limits.conf** на редактирование и прописываем значения:

   **soft nofile 65535**
   
   **hard nofile 65535**

   Применяем командой **ulimit -n 65535** (или перезагрузкой хоста).

   Рассчитываем значение **worker_connections**

   Для 4GB оперативной памяти применяем следующую схему:

   I. Резервируем 1GB RAM для ОС и других сервисов,

   II. Оставшиеся 3GB (3072MB) остаются доступны для ANGIE,

   Приблизительно оцениваем безопасный объем памяти на соединение (берём среднее значение): **1,5KB на соединение**,

   Рассчитываем максимальное количество **Con(max) = 3072MB / 1,5KB ≈ 2M** соединений.

   Устанавливаем значение wroker_connections в конфигурационном файле.

   Для тестового стенда с 4 ядрами CPU, 2 из которых отдано воркерам ANGIE, это количество составит 32K одновременных соединений.

   Формула для расчета занчения следующая: **worker_connections = (Доступная RAM / Память на соединение) / количество процессов воркеров**

   Получаем следующее значение: (3072Mb = 3,145,728KB)

   **(3,145,728KB / 1,5KB) / 2 = 1 048 576** соединений. Данное значение является практическим максимальным лимитом, в реальных условиях сильно превышает безопасные значения и не используется.

   Также следует учесть накладные расходы на установление TLS соединения, скачки трафика, утилизацию сетевого стэка и ядра и др.

   Оптимальное рекомендуемое значение для тестового стенда 4CPU 4 Gb RAM при выделении для воркеров ANGIE 2 ядер процессора - **16384**.

   Также следует применить директивы **multi_accept** (воркер будет принимать сразу все новые соединения) и **use epoll** (оптимизация для linux серверов).

   Конфигурация будет выглядеть следующим образом:

   **worker_processes 2;**  # Задаем количество воркеров,
   
   **worker_rlimit_nofile 65535;** # соответствует значению ulimit
   
   **events {
       worker_connections 16384;      
       multi_accept on;               
       use epoll;                     
   }**

   Что даёт нам общее количество одновременных соединений: **16K × 2 = 32K** (с приоритетом на эффективное использование оперативной памяти).

   Также необходимо установить таймауты соединений в модуле **http {...}**

   **keepalive_timeout 120;** # таймаут - 2 минуты,
   **keepalive_requests 10000;** # количество запросов которые можно сделать по одному постоянному соединению.

   


































   
