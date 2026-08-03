# Deployment Guide — Render.com

## Зміст

- [Підготовка проєкту](#підготовка-проєкту)
- [Крок 1: Підготовка репозиторію](#крок-1-підготовка-репозиторію)
- [Крок 2: Реєстрація та налаштування Render](#крок-2-реєстрація-та-налаштування-render)
- [Крок 3: Створення PostgreSQL бази даних](#крок-3-створення-postgresql-бази-даних)
- [Крок 4: Створення Web Service](#крок-4-створення-web-service)
- [Крок 5: Налаштування змінних оточення](#крок-5-налаштування-змінних-оточення)
- [Крок 6: Ініціалізація таблиць бази даних](#крок-6-ініціалізація-таблиць-бази-даних)
- [Перевірка деплою](#перевірка-деплою)
- [Оновлення додатку](#оновлення-додатку)
- [Усунення проблем](#усунення-проблем)

---

## Підготовка проєкту

Перед деплоєм переконайтесь, що:

- Файл `.env` **не потрапляє** до Git репозиторію (він має бути у `.gitignore`)
- Файл `.env.example` **присутній** у репозиторії як шаблон змінних
- У `package.json` є скрипт `"start": "node src/index.js"`

### Перевірка `.gitignore`

Відкрийте `.gitignore` і переконайтесь, що є рядки:

```
.env
.env.local
```

---

## Крок 1: Підготовка репозиторію

1. Ініціалізуйте Git репозиторій (якщо ще немає):
   ```bash
   git init
   git add .
   git commit -m "initial commit"
   ```

2. Опублікуйте на GitHub:
   ```bash
   # Створіть репозиторій на github.com, потім:
   git remote add origin https://github.com/<ваш-username>/<назва-репо>.git
   git push -u origin main
   ```

---

## Крок 2: Реєстрація та налаштування Render

1. Перейдіть на [render.com](https://render.com) і зареєструйтесь
2. Натисніть **"Sign in with GitHub"** — це дозволить Render автоматично бачити ваші репозиторії

---

## Крок 3: Створення PostgreSQL бази даних

> Render надає безкоштовну PostgreSQL БД (обмеження: 90 днів на безкоштовному плані).

1. У дашборді натисніть **"New +"** → **"PostgreSQL"**
2. Заповніть форму:
   - **Name**: `accounting-db` (або будь-яка назва)
   - **Database**: `accounting`
   - **User**: `accounting_user`
   - **Region**: Frankfurt (EU Central) — найближче до України
   - **Plan**: Free
3. Натисніть **"Create Database"**
4. Зачекайте 1-2 хвилини поки БД створиться
5. **Збережіть дані підключення** зі сторінки БД — вони знадобляться на наступному кроці:
   - **Host** (Internal Host для з'єднання всередині Render)
   - **Port**: `5432`
   - **Database**
   - **Username**
   - **Password**

---

## Крок 4: Створення Web Service

1. У дашборді натисніть **"New +"** → **"Web Service"**
2. Оберіть **"Build and deploy from a Git repository"**
3. Підключіть ваш GitHub репозиторій
4. Заповніть налаштування:

   | Поле | Значення |
   |------|----------|
   | **Name** | `accounting-api` |
   | **Region** | Frankfurt (EU Central) |
   | **Branch** | `main` |
   | **Runtime** | Node |
   | **Build Command** | `npm install --omit=dev` |
   | **Start Command** | `node src/index.js` |
   | **Plan** | Free |

5. **Не натискайте "Create Web Service" ще** — спочатку налаштуйте змінні оточення (Крок 5)

---

## Крок 5: Налаштування змінних оточення

На тій самій сторінці створення Web Service прокрутіть вниз до розділу **"Environment Variables"** і додайте наступні змінні:

| Ключ | Значення |
|------|----------|
| `PORT` | `3000` |
| `POSTGRES_HOST` | Internal Host з даних Render PostgreSQL |
| `POSTGRES_PORT` | `5432` |
| `POSTGRES_USER` | Username з даних Render PostgreSQL |
| `POSTGRES_PASSWORD` | Password з даних Render PostgreSQL |
| `POSTGRES_DB` | Database з даних Render PostgreSQL |
| `POSTGRES_DIALECT` | `postgres` |
| `NODE_ENV` | `production` |

> **Важливо:** Використовуйте **Internal Host** (не External), щоб з'єднання між сервісами всередині Render було швидшим і безкоштовним.

Після заповнення натисніть **"Create Web Service"**.

---

## Крок 6: Ініціалізація таблиць бази даних

Після першого деплою потрібно одноразово створити таблиці в БД.

### Варіант A: Через Render Shell (рекомендовано)

1. Відкрийте ваш Web Service на Render
2. Перейдіть у вкладку **"Shell"**
3. Виконайте команду:
   ```bash
   node src/setupDb.js
   ```
4. Ви маєте побачити:
   ```
   --- The table for the todos model was just created! ---
   --- The table for the users model was just created! ---
   --- The table for the expenses model was just created! ---
   ```

### Варіант B: Через зовнішнє підключення

Використайте **External Database URL** з налаштувань Render PostgreSQL і підключіться через `psql` або будь-який GUI (TablePlus, DBeaver, pgAdmin).

---

## Перевірка деплою

Після успішного деплою Render надасть URL виду `https://accounting-api.onrender.com`.

Перевірте роботу API:

```bash
# Перевірка доступності сервера
curl https://accounting-api.onrender.com/todos

# Або відкрийте у браузері
https://accounting-api.onrender.com/users
```

Очікуваний результат — JSON відповідь (`[]` якщо БД порожня).

---

## Оновлення додатку

Render автоматично деплоїть нову версію щоразу, коли ви пушите зміни до гілки `main`:

```bash
git add .
git commit -m "feat: опис змін"
git push origin main
```

Render автоматично:
1. Виявляє новий коміт
2. Запускає Build Command
3. Замінює попередню версію новою (без downtime)

---

## Усунення проблем

### ❌ `Error: Dialect needs to be explicitly supplied`
**Причина:** Змінна `POSTGRES_DIALECT` відсутня або порожня.  
**Рішення:** Додайте `POSTGRES_DIALECT=postgres` у Environment Variables на Render.

---

### ❌ `Error: connect ECONNREFUSED` або `Connection refused`
**Причина:** Неправильний хост або БД ще не готова.  
**Рішення:** Переконайтесь, що використовуєте **Internal Host** (не External) з налаштувань Render PostgreSQL.

---

### ❌ Сервер "засинає" через 15 хвилин
**Причина:** Безкоштовний план Render автоматично зупиняє неактивні сервіси.  
**Рішення:** При наступному запиті сервер "прокинеться" (займає ~30 секунд). Для постійної роботи перейдіть на платний план або використовуйте зовнішній "пінгер" (наприклад, [UptimeRobot](https://uptimerobot.com)).

---

### ❌ Таблиці не створені — помилки `relation does not exist`
**Причина:** Не виконано `setupDb.js` після деплою.  
**Рішення:** Виконайте `node src/setupDb.js` через Shell у Render (Крок 6).

---

### 📋 Перевірка логів

Логи додатку доступні у вкладці **"Logs"** вашого Web Service на Render — це перше місце для діагностики будь-яких проблем.
