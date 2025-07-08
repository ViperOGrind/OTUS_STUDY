**Результаты выполнения ДЗ №7.**

Балансировка нагрузки (HTTP)

**HOST**
1. Используется корпоративное облако с изолированной инфраструктурой под демо стенд для развёртывания ВМ.
   
2. Базовая ОС для Reverse proxy/WAF1/Server - Ubuntu 24.04 server minimized + Angie 1.9.1, базовая ОС WAF2 - Debian 11 + Nginx 1.26,
   
3. Базовая конфигурация ВМ reverse-proxy (х2):

   ```
   CPU: 4 core

   RAM: 4 Gb

   DISK: 20 Gb
   ```

4. Базовая конфигурация ВМ WAF1 (x2):

   ```
   CPU: 4 core

   RAM: 6 Gb

   DISK: 250 Gb
   ```

5. Базовая конфигурация ВМ WAF2 (x2):

   ```
   CPU: 4 core

   RAM: 32 Gb

   DISK: 100 Gb
   ```

6. Базовая конфигурация ВМ SRV01 (x1):

   ```
   CPU: 2 core

   RAM: 4 Gb

   DISK: 20 Gb
   ```

7. Произведена конфигурация различных методов балансировки для upstream серверов в конфигурационном файле angie.conf.

   [ANGIE.CONF (RP)](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/16.%20%D0%91%D0%B0%D0%BB%D0%B0%D0%BD%D1%81%D0%B8%D1%80%D0%BE%D0%B2%D0%BA%D0%B0%20%D0%BD%D0%B0%D0%B3%D1%80%D1%83%D0%B7%D0%BA%D0%B8%20(HTTP)/Artifacts/angie.conf)

