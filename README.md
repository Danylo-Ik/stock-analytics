# Аналітична платформа фондового ринку

Цей проєкт — це масштабована система для збору, потокової обробки та аналізу даних фондового ринку США в режимі реального часу. Вона використовує дані з біржі Finnhub (WebSocket + REST).

**Стек технологій:** Apache Kafka, Apache Spark (Streaming + Batch), Apache Cassandra, FastAPI, Docker.

---

## Швидкий старт

### 1. Підготовка
Переконайтеся, що у вас встановлено **Docker** та **Docker Compose**. Також вам знадобиться безкоштовний API-ключ від [Finnhub](https://finnhub.io/).

### 2. Налаштування середовища
Створіть файл `.env` у кореневій директорії проєкту та додайте свій ключ:

```env
FINNHUB_API_KEY=ваш_ключ_finnhub
SYMBOLS=AAPL,MSFT,GOOGL,AMZN,NVDA,META,TSLA
```

**Запуск системи**
Щоб підняти всю інфраструктуру (брокер повідомлень, бази даних, кластер обчислень та API), виконайте команду:

```Bash
docker compose up -d
```
Примітка: При першому запуску ініціалізація Cassandra та Kafka може зайняти 1-2 хвилини. Завдання потокової обробки (streaming) почнуть генерувати результати, як тільки накопичиться достатньо даних у мікробатчах.

За потреби виконання завдань можна почати вручну через 
```Bash
docker exec -it streaming bash run_jobs.sh
```

### 3. Доступ до API
Аналітичні дані доступні через REST API, який автоматично генерує Swagger UI.

Після успішного запуску контейнерів перейдіть за посиланням:
http://localhost:8000/docs

Тут ви можете тестувати всі доступні ендпоінти:

- Отримання історичних свічок (OHLCV)

- Перегляд алертів волатильності та великих угод ("китів")

- Перевірка поточного спреду та моментуму ринку

- Доступ до погодинних звітів

**Корисні команди**
Перегляд стану всіх контейнерів:

```Bash
docker compose ps
```
Перегляд логів потокової обробки (щоб переконатися, що дані йдуть):

```Bash
docker compose logs -f streaming
```

**Перегляд таблиць Cassandra**

```Bash
docker exec -it cassandra cqlsh
```
```SQL
USE market_analytics;
DESCRIBE TABLES;
```
К-сть записів

```SQL
SELECT COUNT(*) FROM trades;
```
Whale Alerts

```SQL
SELECT symbol, alert_time, trade_size, deviation_percent FROM whale_alerts LIMIT 5;
```
Volatility Monitor

```SQL
SELECT symbol, window_start, volatility FROM volatility_windows LIMIT 10;
```
Market Momentum

```SQL
SELECT symbol, window_start, current_price, price_change_pct FROM market_momentum LIMIT 10;
```
Зупинка всієї системи (без видалення збережених даних):

```Bash
docker compose stop
```
Повне видалення контейнерів (дані в Cassandra збережуться, якщо налаштовано volume):

```Bash
docker compose down
```