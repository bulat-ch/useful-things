Для начала надо арендовать сервер можно и VPS, например я брал у Селекека. Там же у них можно взять и домен. 

Берём XRAY отсюда https://github.com/XTLS/Xray-install
А именно:
```
bash -c "$(curl -L https://github.com/XTLS/Xray-install/raw/main/install-release.sh)" @ install
```

Включаем сервис XRAY:
```
systemctl enable xray
```

Ставим 3X-UI https://github.com/MHSanaei/3x-ui
А именно командой:
```
bash <(curl -Ls https://raw.githubusercontent.com/mhsanaei/3x-ui/master/install.sh)
```
В процессе попросит установить логин, пароль и порт. Можете самостоятельно указать или просто нажать Enter и они сгенерируются автоматически случайными. Запомните их.

Далее ставим nginx:
```
sudo apt update && sudo apt install nginx -y
```

Устанавливаем Certbot для получения бесплатного SSL-сертификата:
```
sudo apt install certbot python3-certbot-nginx -y
```

Запрашиваем сертификат:
```
sudo certbot --nginx -d ваш.домен.com
```

Создаём конфиг файл nginx для X-UI:
```
sudo nano /etc/nginx/sites-available/x-ui.conf
```

В содержимое вставляем такое, где <ваш.домен.com>, собственно, ваш домен и <порт X-UI> - порт, который Вы выбрали при установке X-UI:
```
server {
	listen 80;
	server_name <ваш.домен.com>;
	return 301 https://$server_name$request_uri;
	}

server {
	listen 443 ssl http2;
	server_name <ваш.домен.com>;

	ssl_certificate /etc/letsencrypt/live/<ваш.домен.com>/fullchain.pem;
	ssl_certificate_key /etc/letsencrypt/live/<ваш.домен.com>/privkey.pem;

	location / 	{
			proxy_pass http://127.0.0.1:<порт X-UI>;  # Ваш порт X-UI
			proxy_set_header Host $host;
			proxy_set_header X-Real-IP $remote_addr;
			proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
			proxy_set_header Upgrade $http_upgrade;
			proxy_set_header Connection "upgrade";
			}
	}
```

Проверьте существующие конфиги Nginx:		
```
ls /etc/nginx/sites-enabled/
```

Если есть лишний конфиг (например, default), отключите его:
```
rm /etc/nginx/sites-enabled/конфликтующий_файл.conf
```
Certbot автоматически обновляет сертификаты, но можно проверить:
```
certbot renew --dry-run
```

Открываем нужные порты:
```
ufw allow 443/tcp
```
```
ufw allow 80/tcp
```
```
ufw allow 22/tcp
```

Активируем конфиг созданием символьной ссылки:
```
ln -s /etc/nginx/sites-available/x-ui.conf /etc/nginx/sites-enabled/
```

Проверяем конфиг: 
```
nginx -t
```

Перезагружаем nginx:
```
systemctl restart nginx
```

Включаем фаервол:
```
ufw enable
```

Можно польоваться.


Для безопасности:

1.
Можно изменить порт SSH:
```
nano /etc/ssh/sshd_config
```
Меняем порт 22 в строке "Port 22" на нужный Вам.

Открываем его:
```
ufw allow <порт для SSH>/tcp
```

Закрываем старый:
```
ufw deny 22/tcp
```

Проверяем фаервол:
```
ufw status
```

Закрываем все лишние порты, если есть:
```
ufw deny <лишний порт>/tcp  
```

Перезагружаем демона SSH:
```
systemctl restart sshd
```

2.
Устанавливаем fail2ban для защиты от брутфорса:
```
apt install fail2ban
```
