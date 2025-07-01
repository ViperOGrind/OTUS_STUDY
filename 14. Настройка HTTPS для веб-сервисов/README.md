**Результаты выполнения ДЗ №5.**

Оптимизация производительности веб-сервисов

**HOST**
1. Используется VMWare Workstation для развёртывания ВМ.
   
2. Базовая ОС - Ubuntu 24.04 server minimized
   
3. Базовая конфигурация ВМ reverse-proxy (х2):

   ```
   CPU: 4 core

   RAM: 4 Gb

   DISK: 20 Gb

   NET: bridged 10.254.254.124/27
   ```

4. Базовая конфигурация ВМ WAF1 (x2):

   ```
   CPU:

   RAM:

   DISK:

   NET:
   ```

5. Базовая конфигурация ВМ WAF2 (x2):

   ```
   CPU:

   RAM:

   DISK:

   NET:
   ```

6. Базовая конфигурация ВМ SRV01 (x1):

   ```
   CPU:

   RAM:

   DISK:

   NET:
   ```

7. MediaWiKi развернута в докер-контейнере (с использованием веб-панели управления средой контейнеризации Portainer),
   
8. Выделено 2 домена для MediaWiKi (для реализации демо-стенда под каждый WAF),
    
9. Для reverse-proxy и WAF собраны кластеры высокой доступности (HA на Keepalived с контролем работоспособности процесса Angie/Nginx),
    
10. Общая схема прохождения трафика и терминирования SSL:

    Интернет -> RP01|RP02 -> WAF1|WAF2 -> SRV01L (Angie принимает запросы на IP 10.28.75.11 порты: 80 и 443, проксирует на 127.0.0.1 порты: 80(WiKi), 443(WiKi), 9080(Portainer), 9443(Portainer)

11. Терминирование SSL производится на Reverse proxy (конфигурация для обеих нод кластера высокой доступности одинаковая):

    [ANGIE.CONF (RP)](https://github.com/ViperOGrind/OTUS_STUDY/blob/main/14.%20%D0%9D%D0%B0%D1%81%D1%82%D1%80%D0%BE%D0%B9%D0%BA%D0%B0%20HTTPS%20%D0%B4%D0%BB%D1%8F%20%D0%B2%D0%B5%D0%B1-%D1%81%D0%B5%D1%80%D0%B2%D0%B8%D1%81%D0%BE%D0%B2/Artifacts/angie_rp.conf)

12. Резльтаты проверки с помощью общедоступного сервиса [Qalis SSL Labs](https://www.ssllabs.com/ssltest/analyze.html?d=wiki-pt.df-lab.ru)
