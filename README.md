# Kittygram

Kittygram — социальная сеть для обмена фотографиями любимых питомцев. 
Можно добавлять фотографии, достижения, а также рассказать всем пользователям, как зовут Ваших питомцев, какого они цвета и когда родились.

## Технологии
- Python
- React
- Django
- nginx
- gunicorn

## Установка проекта.
<br>1. Клонируем репозиторий:
```
git clone git@github.com:mutsolgov/api_yamdb.git
```

<br>2. Создаем и активируем виртуальное окружение:
```
python3 -m venv venv
```
```
source venv/bin/activate
```

<br>3. Установливаем зависимости из файла requirements.txt:
```
pip install -r requirements.txt 
```

<br>4. Перейходим в папку backend и выполняем миграции:
```
cd infra_sprint1/backend/
```
```
python manage.py migrate
```

<br>5. Создаем суперпользователя:
```
python manage.py createsuperuser
```

<br>6. В файле infra_sprint1/backend/kittygram_backend/settings.py в переменную ALLOWED_HOSTS добавляем локальные адреса, а также доменное имя или внешний IP:
```
ALLOWED_HOSTS = ['IP-сервера', '127.0.0.1', 'localhost', 'ваш_домен']
```

<br>7. В том же файле settings.py поменяйте значение переменной DEBUG с True на False:
```
DEBUG = False
```
<br>8. Подготавливаем бэкенд-приложение для сбора статики. В файле settings.py укажите директорию, куда эту статику нужно сложить,
укажите новое значение для константы STATIC_URL, добавьте константу STATIC_ROOT и
сохраните изменения в файле:
```
STATIC_URL = 'static_backend'
STATIC_ROOT = BASE_DIR / 'static_backend' 
```
<br> 9. Собераем статику бэкенд-приложения:
```
python manage.py collectstatic
```
<br> 10. Скопируйте директорию static_backend/ в директорию /var/www/название_проекта/:
```
sudo cp -r infra_sprint1/backend/static_backend/ /var/www/kittygram/
```

<br>11. Перейти в infra_sprint1/frontend и установить зависимости для frontend-приложения
```
npm i
```
<br> 12. Из директории с фронтенд-приложением выполните команду:
```
npm run build
```

<br> 13. Скопируйте статику фронтенд-приложения в директорию по умолчанию (точка после build важна — будет скопировано содержимое директории):

```
sudo cp -r infra_sprint1/frontend/build/. /var/www/kittygram/
``````


<br>14. Для backend-приложения установите WSGI-сервер gunicorn:
```
pip install gunicorn
```

<br>15. Создайте файл конфигурации для автозапуска WSGi-сервера
```
sudo nano /etc/systemd/system/gunicorn.service
```

<br>16. Внесите в файл gunicorn.service следующие настройки:
```
[Unit]
Description=gunicorn daemon 
After=network.target 

[Service]
User=имя_пользователя_в_системе
WorkingDirectory=/home/<имя-пользователя>/infra_sprint1/backend/
ExecStart=/home/<имя-пользователя>/infra_sprint1/venv/bin/gunicorn --bind 0.0.0.0:8000 backend.wsgi

[Install]
WantedBy=multi-user.target 
```

<br>17. Запустите созданную службу и внесите её в автозапуск:
```
sudo systemctl start gunicorn
```
```
sudo systemctl enable gunicorn
```

<br>18. Установите веб-сервер и запустите его:
```
sudo apt install nginx -y
```
```
sudo systemctl start nginx 
```
<br>19. Перейдите в файл конфигурации веб-сервера и измените его настройки на следующие:
```
sudo nano /etc/nginx/sites-enabled/default 
```
```
server {

    server_name <публичный-IP-адрес> <доменное-имя>;

    location /api/ {
        proxy_pass http://127.0.0.1:8000;
    }

    location /admin/ {
        proxy_pass http://127.0.0.1:8000;
    }

    location /media/ {
        alias /var/www/kittygram/media/;
    }

    location / {
    root    /var/www/kittygram;
    index   index.html index.htm;
    try_files  $uri /index.html;
    }

}
```
<br>20. Сохраните изменения в файле, закройте его и проверьте на корректность:
```
sudo nginx -t
```
<br>21. Перезагрузите конфигурацию Nginx:
```
sudo systemctl reload nginx
```

<br>22. Настройка файрвола ufw
<br> Файрвол установит правило, по которому будут закрыты все порты, кроме тех, которые
вы явно укажете:


- Активируйте разрешение принимать запросы только на порты 80, 443 и 22:
```
sudo ufw allow 'Nginx Full'
```
```
sudo ufw allow OpenSSH
```
<br>23. Включите файрвол:
```
sudo ufw enable
```
<br> 24. Получение и настройка SSL-сертификата:
- Находясь на сервере, установите certbot:
```
sudo apt install snapd

sudo snap install core; sudo snap refresh core

sudo snap install --classic certbot

sudo ln -s /snap/bin/certbot /usr/bin/certbot
```
<br> 25. Запустите certbot и получите SSL-сертификат:
```
sudo certbot --nginx
```

<br>26. Создайте папку для медиафайлов в директории веб-сервера, измените права доступа:
```
cd /var/www/kittygram/
```
```
mkdir media
```
```
sudo chown -R <имя_пользователя> /var/www/kittygram/media/
```
<br> 27. Создаем файл .env и вводим свои данные.
```
SECRET_KEY = (SECRET_KEY)
```

## Автор
Муцольгов Магомед
