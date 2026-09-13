# 09. Docker. Basic concept


## 📂 Структура проекта




## 🛠️ Homework Assignment 1: Docker Installation and Basic Commands



```bash
docker --version
Docker version 29.8.0, build 88096ef
```

#### 2. Запуск тестового контейнера hello-world
```bash
docker run hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

```

#### 3. Просмотр списка контейнеров
Команда `docker ps` выводит только работающие в данный момент контейнеры. Так как `hello-world` успешно выполнил задачу и завершился, список активных контейнеров пуст:
```
docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```
Чтобы увидеть завершенный контейнер, использовалась команда с флагом `-a`:
```bash
docker ps -a
CONTAINER ID   IMAGE                COMMAND           CREATED              STATUS                          PORTS     NAMES
02f52f8c1b57   hello-world:latest   "/hello"          About a minute ago   Exited (0) About a minute ago             stupefied_lovelace
```

---

## 🏗️ Homework Assignment 2: Building a Docker Image with Dockerfile

Был создан базовый образ для веб-приложения на **Python (Flask)**.

### История команд сборки и запуска:
1. **Сборка образа:**
```bash
docker build -t my-flask-app:v1

   IMAGE                ID             DISK USAGE   CONTENT SIZE   EXTRA
my-flask-app:v1      0d2b63ec67ac        212MB           52MB 
```
2. **Запуск контейнера в фоновом режиме с пробросом портов:**
```bash
docker run -d -p 80:5000 --name my_flask my-flask-app:v1
3188e3318c137c8198bb69f742c82636cc7e49c05156b60539b14b20a5e2e4dd
```
3. **Проверка работы контейнера (`docker ps`):**
```bash
docker ps
CONTAINER ID   IMAGE             COMMAND           CREATED         STATUS         PORTS                                     NAMES
3188e3318c13   my-flask-app:v1   "python app.py"   6 minutes ago   Up 6 minutes   0.0.0.0:80->5000/tcp, [::]:80->5000/tcp   my_flask
```
4. **Доступ к приложению:**
``` 
curl localhost
<h1>Hello from Docker Container! 🚀</h1>
```
---

## 🚀 Homework Assignment 3: Docker Build Automation (GitHub Actions)

В данном задании реализована **многоэтапная (Multi-stage) сборка** Docker-образа и настроен полноценный CI/CD процесс с отправкой готового образа на Docker Hub и уведомлением в Slack.

### Преимущества разработанной Multi-stage сборки:
1. **Минимальный размер финального образа:** Все тяжелые инструменты сборки, кэш менеджера пакетов `pip` и сборочные утилиты (пакет `wheel`) остаются на изолированном этапе `builder`. В финальный runtime-образ `runner` копируются исключительно скомпилированные библиотеки и код приложения.
2. **Безопасность:** В продакшн-образе отсутствуют лишние инструменты разработки, что уменьшает потенциальную поверхность атаки (attack surface).
3. **Быстрота развертывания:** Легковесный финальный образ быстрее скачивается на целевые сервера (deploy-ноды).

### Конфигурация Multi-stage Dockerfile (`homework3/Dockerfile`):
```dockerfile
# === STAGE 1: Build stage ===
FROM python:3.10-slim AS builder
WORKDIR /build
RUN pip install --no-cache-dir wheel && \
    pip install --no-cache-dir --user flask

# === STAGE 2: Runtime stage ===
FROM python:3.10-slim AS runner
WORKDIR /app
# Копируем только зависимости из этапа сборки
COPY --from=builder /root/.local /root/.local
COPY app.py .
ENV PATH=/root/.local/bin:\$PATH
EXPOSE 5000
CMD ["python", "app.py"]
```

### Автоматизация через GitHub Actions:
Пайплайн настроен на автоматический запуск при каждом `push` или создании `Pull Request` в ветку `master` (или `main`).

**Основные шаги воркфлоу:**
1. **Checkout code** — клонирование кода репозитория на воркер.
2. **Log in to Docker Hub** — безопасная авторизация с использованием `secrets.DOCKERHUB_USERNAME` и `secrets.DOCKERHUB_TOKEN`.
3. **Build and Push Docker image** — сборка многоэтапного образа с контекстом из папки `./homework3` и отправка тега `latest` в репозиторий Docker Hub.
4. **Slack Notification** — отправка интерактивного сообщения в канал Slack через входящий вебхук (`secrets.SLACK_WEBHOOK`):
   * При успешном завершении: отправляется зеленая карточка со статусом **Docker Build Success!** и именем собранного образа
 ```
Actions URL                    Commit
Build and Push Docker Image    3b8305
Docker Build Success!
Image: maksimsolap/flask-app:latest successfully built and pushed.
 ```    
   * При падении сборки: отправляется красная карточка **Docker Build Failed ❌** со ссылкой на коммит для быстрого дебага.






