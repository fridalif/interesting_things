## 1.  Подготовка приложения на Go

 - Крутиться будет на 127.0.0.1 на произвольном порту (пусть 9999)
 - Для того чтобы Go принимал запросы с внешней сети необходимо добавить следующие строчки

```Go
router := http.NewServeMux()
c := cors.New(cors.Options{
		AllowedOrigins:   []string{"http(-s)://<IP:PORT/Domain>(Который в NGINX)"},
		AllowCredentials: true,
		AllowedMethods:   []string{"GET", "POST", "DELETE", "PUT", "OPTIONS"},
		AllowedHeaders:   []string{"Content-Type", "Authorization", "Refresh"},
	})

corsedRouter := c.Handler(router)
http.ListenAndServe("127.0.0.1:9999", corsedRouter)
```

- Далее билдим проект и ставим на фоновую работу

```Bash
go build
./projectname > ./backend_app_log 2>./backend_error_log &
```

- Если необходимо выключить работу сервера то вводим команды

```Bash
ss -tulpnet
#Находим pid процессаа занимающего наш порт(в моём случае 9999)
kill <pid>
```
## 2. Подготовка Next.js

- Крутиться будет на 127.0.0.1 на произвольном порту (пусть 3000)
- Для того чтобы сделать релизную версию проекта необходимо в директории проекта выполнить команду

```Bash
npx next build
```

- Ставим проект на фон

```Shell

npx next start -H 127.0.0.1 -p 3000 > ./frontend_app_log 2>./frontend_error_log &

```

- Если необходимо выключить работу сервера то вводим команды

```Bash
ss -tulpnet
#Находим pid процессаа занимающего наш порт(в моём случае 3000)
kill <pid>
```

## 3. Подготовка и настройка NGINX

- Установка nginx

```Bash
sudo apt intall nginx
```

- Создание своего конфига
```Bash
touch /etc/nginx/sites-available/myproject(- это произвольное название)
```

- Открываем данный конфиг и вбиваем следующие настройки

```myproject
server{
	listen <Порт который будет прослушиваться>;
	server_name <Ip/domain который будет прослушиваться>;
	location /api {
		proxy_pass http://127.0.0.1:9999(- это порт backenda)/api;( - это маршрут на бэке (location и это маршрут можно произваольно менять))
	}

	location /media {
		alias /path/to/media; (- путь к папке с медиа)

		location ~ ^/media/profiles/(?<id_profile>\d+)/(?<image>.+\.(png|jpg|jpeg|webp))$ {
	        expires 30d;  # Кэшировать на 30 дней
	        add_header Cache-Control "public, no-transform";
	
	        # Оптимизация для статических файлов
	        sendfile on;
	        tcp_nopush on;
	        tcp_nodelay on;
	
	        # Включаем gzip для изображений (если поддерживается)
	        gzip on;
	        gzip_types image/jpeg image/png image/webp;
	    }
	}
	
	location / {
		proxy_pass http://127.0.0.1:3000; (- путь к nginx)
	}
}
```

- Рестартим nginx и проверяем работоспособность

```Bash
service nginx restart
```


## 4. Создание сервиса в Linux




## P.S.
- При использовании веб-сокетов в конфиг nginx необходимо добавить:
```myproject

location /api/websocket_login {
    proxy_set_header   X-Forwarded-For $remote_addr;
	proxy_set_header   Host $http_host;
    proxy_set_header Upgrade websocket;
    proxy_set_header Connection Upgrade;
    proxy_pass         "http://127.0.0.1:9999/api/websocket_login";
}
location /api/websocket {
    proxy_set_header   X-Forwarded-For $remote_addr;
    proxy_set_header   Host $http_host;
    proxy_set_header Upgrade websocket;
    proxy_set_header Connection Upgrade;
    proxy_pass         "http://127.0.0.1:9999/api/websocket";
}

```