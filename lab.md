# Лабораторная работа: Управление процессами в Linux


## Цель работы:
Научиться управлять процессами в Linux: запускать, останавливать и мониторить их. Освоить использование сигналов для управления процессами, организовать взаимодействие между процессами через пайпы и файлы, а также разрабатывать скрипты для автоматизации управления процессами.

### Пример
Создать процесс с бесконечным циклом, управлять его выполнением через команды `ps`, `kill`, `bg` и `fg`. Научиться останавливать и возобновлять выполнение процесса.


1. Создайте скрипт `example_loop.sh` с содержимым:  
   ```bash
   #!/bin/bash
   while true; do
       echo "Process is running..." >> /tmp/example.log
       sleep 2
   done
   ```
2. Сделайте скрипт исполняемым:  
   ```bash
   chmod +x example_loop.sh
   ```

3. Запустите процесс в фоновом режиме:  
   ```bash
   ./example_loop.sh &
   ```

4. Найдите PID процесса:  
   ```bash
   ps aux | grep example_loop.sh
   ```

5. Приостановите процесс (Ctrl+Z), а затем продолжите его выполнение в фоновом режиме:  
   ```bash
   bg <PID>
   ```

6. Переведите процесс в активный режим:  
   ```bash
   fg <PID>
   ```

7. Завершите процесс:  
   ```bash
   kill <PID>
   ```

8. Убедитесь, что процесс завершен:  
   ```bash
   ps aux | grep example_loop.sh
   ```
   

## Задание: "Организация многоуровневой обработки данных с управлением процессами"

Предстоит разработать систему обработки текстовых данных, которая включает:  
1. **Процесс-генератор**: генерирует случайные строки текста и передает их в именованный канал.  
2. **Процесс-фильтр**: считывает данные из канала, фильтрует строки (например, выбирает строки, содержащие определенное слово), и записывает результат в другой канал.  
3. **Процесс-логгер**: записывает отфильтрованные данные в файл.  
4. **Процесс-менеджер**: управляет всей системой:  
   - Запускает и останавливает процессы.  
   - Увеличивает/уменьшает скорость генерации строк через сигналы.  
   - Проверяет состояние процессов.  


### Как успешно сдать работу?

Создать свой репозиторий из шаблона этого. Как это делается - "Use this template" -> "Create a new repository" и сделайте его public. 

Находясь уже в своем репозитории - создайте новый файл формата .md и там оформляйте отчет. В отчете опишите все шаги которые вы делали, чтобы получить финальный результат работы.

Что вам нужно знать, чтобы успешно защитить работу:

Как создавать и управлять - докерфайлом (команды, оптимизация), образами (создание, запуск, принцип работы), контейнерами (запуск, отслеживание, объединение); виртуализация и контейнеризация. 

## Источники

1. [Источник где можно найти все](https://google.com)
2. [Управление процессами в Linux](https://losst.pro/upravlenie-protsessami-v-linux)  
