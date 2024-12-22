# Лабораторная работа: Управление процессами в Linux

## Цель работы:
Научиться управлять процессами в Linux: запускать, останавливать и мониторить их. Освоить использование сигналов для управления процессами, организовать взаимодействие между процессами через пайпы и файлы, а также разрабатывать скрипты для автоматизации управления процессами.

## Задание: "Организация многоуровневой обработки данных с управлением процессами"

Предстоит разработать систему обработки текстовых данных, которая включает:  
1. **Процесс-генератор**: генерирует случайные строки текста и передает их в именованный канал.  
2. **Процесс-фильтр**: считывает данные из канала, фильтрует строки (например, выбирает строки, содержащие определенное слово), и записывает результат в другой канал.  
3. **Процесс-логгер**: записывает отфильтрованные данные в файл.  
4. **Процесс-менеджер**: управляет всей системой:  
   - Запускает и останавливает процессы.  
   - Увеличивает/уменьшает скорость генерации строк через сигналы.  
   - Проверяет состояние процессов.  
## Реализация:

### 1. Подготовка окружения

1. Убедитесь, что вы работаете в Linux-окружении (например, в WSL, Ubuntu или другой дистрибуции).  
2. Создайте рабочую папку для лабораторной работы:  
   ```bash
   mkdir process_lab
   cd process_lab
   ```

3. Создайте необходимые файлы:
   ```bash
   touch generator.sh filter.sh logger.sh manager.sh
   ```
### 2. Реализация процессов

#### Процесс-генератор (`generator.sh`)  
Генерирует случайные строки текста с регулируемой частотой.

Содержание файла `generator.sh`:
```bash
#!/bin/bash

# Именованный канал для передачи данных
FIFO_PATH="/tmp/input_fifo"

# Проверка существования канала
if [[ ! -p $FIFO_PATH ]]; then
    mkfifo $FIFO_PATH
fi

# Начальная задержка (1 секунда)
DELAY=1

# Обработчик сигналов
trap 'DELAY=$((DELAY > 1 ? DELAY - 1 : 1))' SIGUSR1
trap 'DELAY=$((DELAY + 1))' SIGUSR2
trap 'echo "Terminating generator"; exit 0' SIGTERM

# Генерация строк
while true; do
    RANDOM_TEXT="$(date +%T) RandomWord$RANDOM"
    echo "$RANDOM_TEXT" > $FIFO_PATH
    sleep $DELAY
done
```

Сделайте файл исполняемым:  
```bash
chmod +x generator.sh
```

#### Процесс-фильтр (`filter.sh`)  
Считывает строки, фильтрует по ключевому слову и передает в другой канал.

Содержание файла `filter.sh`:
```bash
#!/bin/bash

# Именованные каналы
INPUT_FIFO="/tmp/input_fifo"
OUTPUT_FIFO="/tmp/output_fifo"

# Проверка существования каналов
if [[ ! -p $INPUT_FIFO ]]; then
    echo "Input FIFO does not exist. Run generator.sh first."
    exit 1
fi

if [[ ! -p $OUTPUT_FIFO ]]; then
    mkfifo $OUTPUT_FIFO
fi

# Фильтрация данных
FILTER_KEYWORD="RandomWord"

trap 'echo "Terminating filter"; exit 0' SIGTERM

while true; do
    if read LINE < $INPUT_FIFO; then
        if [[ $LINE == *"$FILTER_KEYWORD"* ]]; then
            echo "$LINE" > $OUTPUT_FIFO
        fi
    fi
done
```

Сделайте файл исполняемым:  
```bash
chmod +x filter.sh
```


#### Процесс-логгер (`logger.sh`)  
Сохраняет данные в файл.

Содержание файла `logger.sh`:
```bash
#!/bin/bash

# Именованный канал
OUTPUT_FIFO="/tmp/output_fifo"
LOG_FILE="/tmp/filtered_log.txt"

# Проверка существования канала
if [[ ! -p $OUTPUT_FIFO ]]; then
    echo "Output FIFO does not exist. Run filter.sh first."
    exit 1
fi

# Запись в лог
trap 'echo "Terminating logger"; exit 0' SIGTERM

while true; do
    if read LINE < $OUTPUT_FIFO; then
        echo "$(date +%T) $LINE" >> $LOG_FILE
    fi
done
```

Сделайте файл исполняемым:  
```bash
chmod +x logger.sh
```

### 3. Процесс-менеджер

Скрипт `manager.sh` управляет запуском, остановкой и мониторингом системы.

Содержание файла `manager.sh`:
```bash
#!/bin/bash

# Функции для управления процессами
start_system() {
    echo "Starting system..."
    ./generator.sh &
    GEN_PID=$!
    echo $GEN_PID > gen.pid

    ./filter.sh &
    FILT_PID=$!
    echo $FILT_PID > filt.pid

    ./logger.sh &
    LOG_PID=$!
    echo $LOG_PID > log.pid

    echo "System started. PIDs: Generator=$GEN_PID, Filter=$FILT_PID, Logger=$LOG_PID"
}

stop_system() {
    echo "Stopping system..."
    kill $(cat gen.pid filt.pid log.pid)
    rm -f gen.pid filt.pid log.pid
    echo "System stopped."
}

status_system() {
    echo "Checking system status..."
    for PID_FILE in gen.pid filt.pid log.pid; do
        if [[ -f $PID_FILE ]]; then
            PID=$(cat $PID_FILE)
            if ps -p $PID > /dev/null; then
                echo "$PID_FILE: Running (PID=$PID)"
            else
                echo "$PID_FILE: Not running"
            fi
        else
            echo "$PID_FILE: Not found"
        fi
    done
}

log_system() {
    echo "Log file contents:"
    cat /tmp/filtered_log.txt
}

# Обработка команд
case $1 in
    start)
        start_system
        ;;
    stop)
        stop_system
        ;;
    status)
        status_system
        ;;
    log)
        log_system
        ;;
    *)
        echo "Usage: $0 {start|stop|status|log}"
        exit 1
        ;;
esac
```

Сделайте файл исполняемым:  
```bash
chmod +x manager.sh
```

### 4. Запуск системы

1. **Запуск системы:**  
   ```bash
   ./manager.sh start
   ```
2. **Управление частотой генерации:**  
   Уменьшить задержку:  
   ```bash
   kill -SIGUSR1 $(cat gen.pid)
   ```
   Увеличить задержку:  
   ```bash
   kill -SIGUSR2 $(cat gen.pid)
   ```

3. **Проверка состояния процессов:**  
   ```bash
   ./manager.sh status
   ```

4. **Просмотр лога:**  
   ```bash
   ./manager.sh log
   ```

5. **Остановка системы:**  
   ```bash
   ./manager.sh stop
   ```

