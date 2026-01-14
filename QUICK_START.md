# 🚀 VPN Bot System - Быстрый старт

Полнофункциональная система для продажи VPN подписок через Telegram бота с интеграцией 3X-UI панели и веб админ-панелью.

## 📋 Содержание

- [Быстрая установка](#-быстрая-установка)
- [Ручная установка](#-ручная-установка)
- [Настройка](#-настройка)
- [Запуск](#-запуск)
- [Управление](#-управление)
- [Устранение неполадок](#-устранение-неполадок)

## ⚡ Быстрая установка

### Автоматический установщик (Рекомендуется)

```bash
# Скачиваем установщик
curl -sSL https://raw.githubusercontent.com/your-repo/vpn-bot-system/main/installer/install.sh -o install.sh

# Делаем исполняемым
chmod +x install.sh

# Запускаем с вашим доменом и email
sudo ./install.sh -d your-domain.com -e your-email@domain.com
```

### Docker установка

```bash
# Клонируем репозиторий
git clone https://github.com/your-repo/vpn-bot-system.git
cd vpn-bot-system

# Настраиваем переменные окружения
cp .env.example .env
nano .env  # Настройте переменные

# Запускаем через Docker
docker-compose up -d

# Инициализируем базу данных
docker-compose exec vpnbot python database/init_db.py
```

## 🔧 Ручная установка

### Требования

- **OS:** Ubuntu 20.04+ / Debian 10+ / CentOS 8+
- **RAM:** Минимум 1GB, рекомендуется 2GB
- **Диск:** Минимум 10GB свободного места
- **Python:** 3.8+
- **Node.js:** 18+
- **PostgreSQL:** 13+
- **Redis:** 6+

### Шаг 1: Установка зависимостей

#### Ubuntu/Debian:
```bash
sudo apt update
sudo apt install -y python3 python3-pip python3-venv nodejs npm postgresql postgresql-contrib redis-server nginx git curl
```

#### CentOS/RHEL:
```bash
sudo yum update -y
sudo yum install -y python3 python3-pip nodejs npm postgresql-server postgresql-contrib redis nginx git curl
```

### Шаг 2: Клонирование проекта

```bash
git clone https://github.com/your-repo/vpn-bot-system.git
cd vpn-bot-system
```

### Шаг 3: Настройка PostgreSQL

```bash
# Запускаем PostgreSQL
sudo systemctl start postgresql
sudo systemctl enable postgresql

# Создаем базу данных и пользователя
sudo -u postgres psql << EOF
CREATE USER vpnbot WITH PASSWORD 'your_secure_password';
CREATE DATABASE vpnbot_db OWNER vpnbot;
GRANT ALL PRIVILEGES ON DATABASE vpnbot_db TO vpnbot;
\q
EOF
```

### Шаг 4: Настройка Python окружения

```bash
# Создаем виртуальное окружение
python3 -m venv venv
source venv/bin/activate

# Устанавливаем зависимости
pip install -r requirements.txt
```

### Шаг 5: Настройка админ-панели

```bash
# Переходим в директорию админ-панели
cd admin-panel

# Устанавливаем Node.js зависимости
npm install

# Возвращаемся в корень
cd ..
```

### Шаг 6: Инициализация базы данных

```bash
# Активируем виртуальное окружение
source venv/bin/activate

# Инициализируем базу данных
python database/init_db.py
```

## ⚙️ Настройка

### Создание .env файла

```bash
cp .env.example .env
nano .env
```

### Обязательные настройки в .env:

```env
# Telegram Bot
BOT_TOKEN=123456789:AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
ADMIN_USER_ID=123456789

# База данных
DATABASE_URL=postgresql://vpnbot:your_password@localhost:5432/vpnbot_db

# 3X-UI панель (ОБЯЗАТЕЛЬНО!)
XRAY_PANEL_URL=https://your-panel.domain.com
XRAY_USERNAME=admin
XRAY_PASSWORD=admin_password

# Домен
DOMAIN=your-domain.com
SSL_EMAIL=admin@your-domain.com

# Безопасность (сгенерируется автоматически или задайте свои)
JWT_SECRET_KEY=your-jwt-secret-key
API_SECRET_KEY=your-api-secret-key
```

### Настройка платежных систем:

```env
# Stripe (банковские карты)
STRIPE_SECRET_KEY=sk_live_your_stripe_secret_key

# Криптовалюты (CoinPayments)
COINPAYMENTS_PRIVATE_KEY=your_coinpayments_private_key
COINPAYMENTS_PUBLIC_KEY=your_coinpayments_public_key

# Российские платежи
YOOMONEY_TOKEN=your_yoomoney_token
QIWI_SECRET_KEY=your_qiwi_secret_key
```

### Настройка 3X-UI панели

1. **Установите 3X-UI панель** на ваш VPN сервер:
   ```bash
   bash <(curl -Ls https://raw.githubusercontent.com/MHSanaei/3x-ui/master/install.sh)
   ```

2. **Настройте inbound'ы** в панели:
   - Создайте inbound'ы для разных серверов
   - Используйте протоколы VLESS или VMess
   - Включите TLS если используете доменные имена

3. **Включите API** в настройках панели

4. **Добавьте URL панели** в .env файл

### Настройка Nginx (опционально)

```bash
# Копируем конфигурацию
sudo cp installer/nginx.conf /etc/nginx/sites-available/vpnbot
sudo ln -s /etc/nginx/sites-available/vpnbot /etc/nginx/sites-enabled/
sudo rm /etc/nginx/sites-enabled/default

# Тестируем конфигурацию
sudo nginx -t

# Перезагружаем Nginx
sudo systemctl reload nginx
```

### SSL сертификат (рекомендуется)

```bash
# Устанавливаем Certbot
sudo apt install certbot python3-certbot-nginx

# Получаем сертификат
sudo certbot --nginx -d your-domain.com

# Настраиваем автоматическое обновление
sudo crontab -e
# Добавьте строку:
0 12 * * * /usr/bin/certbot renew --quiet
```

## 🚀 Запуск

### Системный запуск (рекомендуется)

#### Создание systemd сервисов:

1. **VPN Bot Service:**
```bash
sudo nano /etc/systemd/system/vpnbot.service
```

```ini
[Unit]
Description=VPN Telegram Bot
After=network.target postgresql.service redis.service

[Service]
Type=simple
User=root
WorkingDirectory=/path/to/vpn-bot-system
Environment=PATH=/path/to/vpn-bot-system/venv/bin
ExecStart=/path/to/vpn-bot-system/venv/bin/python bot/main.py
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

2. **API Service:**
```bash
sudo nano /etc/systemd/system/vpnbot-api.service
```

```ini
[Unit]
Description=VPN Bot API Server
After=network.target postgresql.service redis.service

[Service]
Type=simple
User=root
WorkingDirectory=/path/to/vpn-bot-system
Environment=PATH=/path/to/vpn-bot-system/venv/bin
ExecStart=/path/to/vpn-bot-system/venv/bin/python api/server.py
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

3. **Admin Panel Service:**
```bash
sudo nano /etc/systemd/system/vpnbot-admin.service
```

```ini
[Unit]
Description=VPN Bot Admin Panel
After=network.target

[Service]
Type=simple
User=root
WorkingDirectory=/path/to/vpn-bot-system/admin-panel
ExecStart=/usr/bin/npm start
Restart=always
RestartSec=5
Environment=NODE_ENV=production

[Install]
WantedBy=multi-user.target
```

#### Запуск сервисов:

```bash
# Перезагружаем systemd
sudo systemctl daemon-reload

# Включаем автозапуск
sudo systemctl enable vpnbot vpnbot-api vpnbot-admin

# Запускаем сервисы
sudo systemctl start vpnbot vpnbot-api vpnbot-admin

# Проверяем статус
sudo systemctl status vpnbot vpnbot-api vpnbot-admin
```

### Ручной запуск (для разработки)

#### Терминал 1 - База данных и Redis:
```bash
sudo systemctl start postgresql redis-server
```

#### Терминал 2 - Telegram Bot:
```bash
cd vpn-bot-system
source venv/bin/activate
python bot/main.py
```

#### Терминал 3 - API Server:
```bash
cd vpn-bot-system
source venv/bin/activate
python api/server.py
```

#### Терминал 4 - Admin Panel:
```bash
cd vpn-bot-system/admin-panel
npm start
```

### Docker запуск

```bash
# Запуск всех сервисов
docker-compose up -d

# Просмотр логов
docker-compose logs -f vpnbot

# Остановка
docker-compose down
```

## 📱 Доступ к системе

После запуска у вас будет доступ к:

- **Telegram Bot:** `@your_bot_username`
- **Admin Panel:** `https://your-domain.com` или `http://localhost:3000`
- **API:** `https://your-domain.com/api` или `http://localhost:8000`

## 🔧 Управление

### Полезные команды

```bash
# Статус сервисов
sudo systemctl status vpnbot vpnbot-api vpnbot-admin

# Перезапуск бота
sudo systemctl restart vpnbot

# Просмотр логов
sudo journalctl -f -u vpnbot

# Обновление системы
git pull origin main
pip install -r requirements.txt --upgrade
cd admin-panel && npm install
sudo systemctl restart vpnbot vpnbot-api vpnbot-admin
```

### Бэкап базы данных

```bash
# Создание бэкапа
pg_dump -h localhost -U vpnbot vpnbot_db > backup_$(date +%Y%m%d_%H%M%S).sql

# Восстановление бэкапа
psql -h localhost -U vpnbot vpnbot_db < backup_file.sql
```

### Мониторинг

```bash
# Просмотр активных подключений
sudo netstat -tlnp | grep -E ':(8000|3000|5432|6379)'

# Использование ресурсов
htop
df -h
free -h

# Логи Nginx
sudo tail -f /var/log/nginx/access.log
sudo tail -f /var/log/nginx/error.log
```

## 📊 Первоначальная настройка

### 1. Создание Telegram бота

1. Напишите [@BotFather](https://t.me/BotFather)
2. Выполните команду `/newbot`
3. Следуйте инструкциям
4. Скопируйте токен в файл `.env`

### 2. Получение Admin User ID

1. Напишите [@userinfobot](https://t.me/userinfobot)
2. Скопируйте ваш ID в `.env` файл как `ADMIN_USER_ID`

### 3. Настройка платежей

#### Stripe (для банковских карт):
1. Зарегистрируйтесь на [stripe.com](https://stripe.com)
2. Получите API ключи в Dashboard
3. Добавьте в `.env` файл

#### CoinPayments (для криптовалют):
1. Зарегистрируйтесь на [coinpayments.net](https://coinpayments.net)
2. Создайте API ключи
3. Добавьте в `.env` файл

### 4. Тестирование системы

```bash
# Проверка подключения к 3X-UI
curl -X POST "https://your-panel.com/login" \
  -H "Content-Type: application/json" \
  -d '{"username":"admin","password":"admin"}'

# Проверка API
curl http://localhost:8000/health

# Проверка админ-панели
curl http://localhost:3000
```

## 🚨 Устранение неполадок

### Частые проблемы

#### 1. Бот не отвечает на сообщения

```bash
# Проверьте статус бота
sudo systemctl status vpnbot

# Проверьте логи
sudo journalctl -f -u vpnbot

# Проверьте токен бота
grep BOT_TOKEN .env
```

#### 2. Не подключается к 3X-UI панели

```bash
# Проверьте доступность панели
curl -I https://your-panel.com

# Проверьте учетные данные
grep XRAY_ .env

# Проверьте логи
sudo journalctl -f -u vpnbot | grep -i xray
```

#### 3. Не работают платежи

```bash
# Проверьте настройки платежных систем
grep -E "(STRIPE|COIN)" .env

# Проверьте логи платежей
sudo journalctl -f -u vpnbot | grep -i payment
```

#### 4. Админ-панель не загружается

```bash
# Проверьте статус
sudo systemctl status vpnbot-admin

# Проверьте порт
netstat -tlnp | grep :3000

# Проверьте логи
sudo journalctl -f -u vpnbot-admin
```

#### 5. Проблемы с базой данных

```bash
# Проверьте статус PostgreSQL
sudo systemctl status postgresql

# Проверьте подключение
psql -h localhost -U vpnbot vpnbot_db -c "SELECT 1;"

# Пересоздайте базу
python database/init_db.py
```

### Перезапуск всей системы

```bash
# Остановка всех сервисов
sudo systemctl stop vpnbot vpnbot-api vpnbot-admin

# Перезапуск базы данных и Redis
sudo systemctl restart postgresql redis-server

# Запуск всех сервисов
sudo systemctl start vpnbot vpnbot-api vpnbot-admin

# Проверка статуса
sudo systemctl status vpnbot vpnbot-api vpnbot-admin
```

### Логи системы

```bash
# Все логи бота
sudo journalctl -f -u vpnbot

# Логи с фильтрацией
sudo journalctl -f -u vpnbot | grep ERROR

# Логи за последний час
sudo journalctl -u vpnbot --since "1 hour ago"

# Логи в файлы
sudo journalctl -u vpnbot > /tmp/vpnbot.log
```

## 🔒 Безопасность

### Рекомендации

1. **Используйте сильные пароли** для всех сервисов
2. **Настройте файрвол:**
   ```bash
   sudo ufw enable
   sudo ufw allow 22,80,443/tcp
   ```
3. **Регулярно обновляйте систему:**
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```
4. **Настройте бэкапы:**
   ```bash
   # Добавьте в cron
   0 2 * * * /path/to/backup_script.sh
   ```
5. **Мониторьте логи** на предмет подозрительной активности

### Важные файлы для бэкапа

- `.env` - Переменные окружения
- `database/` - Дамп базы данных
- `configs/` - Конфигурационные файлы пользователей
- `logs/` - Логи системы

## 📞 Поддержка

- 📧 **Email:** support@your-domain.com
- 💬 **Telegram:** @your_support_username  
- 📋 **Issues:** [GitHub Issues](https://github.com/your-repo/vpn-bot-system/issues)
- 📖 **Документация:** [Wiki](https://github.com/your-repo/vpn-bot-system/wiki)

## 🎯 Дальнейшие шаги

1. **Настройте мониторинг** (Prometheus + Grafana)
2. **Добавьте больше серверов** в 3X-UI панель
3. **Настройте маркетинговые каналы**
4. **Добавьте дополнительные способы оплаты**
5. **Настройте автоматические бэкапы**

---

## 📝 Changelog

### v1.0.0 (2024-01-20)
- ✅ Первый релиз системы
- ✅ Telegram бот с полным функционалом
- ✅ Веб админ-панель
- ✅ Интеграция с 3X-UI
- ✅ Система платежей
- ✅ Автоустановщик

---

⭐ **Если проект помог вам, поставьте звездочку на GitHub!**

🚀 **Удачного использования VPN Bot System!**