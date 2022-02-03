# Развертывание проекта Python/Django на сервере с помощью Docker

Docker - отличный способ управлять зависимостями в масштабе всей системы для ваших приложений. 
Django - это простой способ запуска веб-приложений. Эти две технологии вместе составляют отличную пару. 

Какой минимум знаний нужен, чтобы работать с Docker?
Docker - это мини виртуальная машина с минимизированным функционалом зависимостей и пакетов, которые нужны твоему проекту.
Надо знать, чтобы бывают контейнеры и images. Images - это список инструкций для сборки контейнера, как образ для виртуалки, 
а контейнер - это запущенный образ.

Некоторые сервера не поддерживают технологию виртуализации, которая нужна для Docker. Например, OpenVZ не подходит для Docker, 
а KVM подходит. Если у вас не запустится докер на сервере, то уточните у хостера, подерживает ли ваш сервер технологию Docker.

Давайте составим туториал, как задеплоить Django c помощью Docker.

1. Установите docker-compose

        sudo apt install docker-compose

2. Обратите внимание на структуру вашего проекта, т.к. это важно при указании путей до файлов в файле Dockerfile, например 
структура моего проекта выглядит так:

        mysite
        |   .idea
        |   .gitignore
        |   README.md
        └───mysite
        │   │   app
        │   └───mysite
        │   │   │   __init__.py
        │   │   │   asgi.py
        │   │   │   setting.py
        │   │   │   url.py
        │   │   │   wsgi.py
        │   │   manage.py
        │   │   Pipfile
        │   │   Pipfile.lock
        │   │   requirements.txt
        │   │   test_uwsgi_deploy.ru

3. Откройте терминал, перейдите в папку с проектом, где находятся manage.py и requirements.txt. Установите pipenv, 
который сам подтянет зависимости в Pipfile.
    
        cd mysite/mysite
        pipenv install
    
4. Если ваш проект изпользует БД, примените миграции

        python manage.py makemigrations
        python manage.py migrate
    
5. Создайте свой Dockerfile командой `touch Dockerfile`. Для примера выложу свой. Синтаксис этого файла лучше всего изучить на официальном сайте.

 
        # Base Image
        FROM python:3.8
        
        # create and set working directory
        RUN mkdir /app
        WORKDIR /app
        
        # Add current directory code to working directory
        ADD . /app/
        
        # set default environment variables
        ENV PYTHONUNBUFFERED 1
        ENV LANG C.UTF-8
        ENV DEBIAN_FRONTEND=noninteractive
        
        # set project environment variables
        # grab these via Python's os.environ
        # these are 100% optional here
        ENV PORT=8888
        
        # Install system dependencies
        RUN apt-get update && apt-get install -y --no-install-recommends \
                tzdata \
                python3-setuptools \
                python3-pip \
                python3-dev \
                python3-venv \
                git \
                && \
            apt-get clean && \
            rm -rf /var/lib/apt/lists/*
        
        
        # install environment dependencies
        RUN pip3 install --upgrade pip
        RUN pip3 install pipenv
        
        # Install project dependencies
        RUN pipenv install --skip-lock --system --dev
        
        EXPOSE 8888
        CMD gunicorn mysite.wsgi:application --bind 0.0.0.0:$PORT
        
6. Создайте свой образ Docker.

        sudo docker build -t simple-django-on-docker -f Dockerfile .
    
7. Запустите свой контейнер.

        sudo docker run -it -p 80:8888 simple-django-on-docker
    
8. Теперь откройте http://localhost. Используйте Ctrl + C, чтобы отменить запуск контейнера.

# Готово)

Полезные команды:

Список запущенных контейнеров

    docker ps -a
    
Остановка контейнера

    docker stop <ID>

Удаление контейнера

    docker rm <ID>
    
Удаление images

    docker rmi <ID>
    
Выгрузка образа в файловую систему в архиве

    docker save -o simple-django-on-docker.tar simple-django-on-docker
    
Загрузка образа из архива в среду Docker

    docker load -i simple-django-on-docker.tar
