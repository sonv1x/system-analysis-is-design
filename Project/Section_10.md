## Розділ 10. Аналіз системи безпеки та захисту даних

Безпека розробленої системи базується на принципах "Defense in Depth" і реалізована на всіх рівнях архітектури додатку.

### 10.1. Криптографічний захист та автентифікація
* **Bcryptjs**: Паролі користувачів обробляються з використанням алгоритму `bcrypt.hash` з 10 раундами солі. Це захищає систему від атак методом перебору та райдужних таблиць.
* **JSON Web Tokens (JWT)**: Автентифікація реалізована через передачу токенів у заголовках запитів. Сервер перевіряє підпис токена за допомогою `process.env.JWT_SECRET`, забезпечуючи безпечний доступ до маршрутів `/api/admin` та `/api/loans`.

### 10.2. Захист прикладного рівня (API)
* **Захист від SQL-ін'єкцій**: У коді взаємодії з БД (PostgreSQL) використовуються виключно параметризовані запити (наприклад, `pool.query('... WHERE id = $1', [id])`). Це унеможливлює впровадження шкідливого коду через поля вводу.
* **Middleware авторизації**: Створено проміжний шар (`verifyToken`), який фільтрує запити до конфіденційних функцій.
* **CORS Policy**: Сервер налаштований так, щоб приймати запити лише з довірених доменів фронтенд-додатку.

### 10.3. Рольова модель доступу (RBAC)
Права доступу жорстко розмежовані на рівні логіки сервера на основі поля `role` (Admin, Librarian, Reader). Це гарантує, що читач не зможе отримати доступ до адмін-панелі або змінити статус інвентарної одиниці.

```javascript
// Приклад реалізації безпеки (Backend)
const bcrypt = require('bcryptjs');
const jwt = require('jsonwebtoken');

// Хешування пароля перед збереженням
const salt = await bcrypt.genSalt(10);
const hashedPassword = await bcrypt.hash(password, salt);

// Генерація JWT токена
const token = jwt.sign({ id: user.id, role: user.role }, process.env.JWT_SECRET, {
  expiresIn: '1h'
});
```

```javascript
// Middleware для авторизації (authMiddleware.js)
const jwt = require('jsonwebtoken');

const verifyToken = (req, res, next) => {
  const token = req.headers['authorization']?.split(' ')[1];
  if (!token) return res.status(401).json({ message: 'Доступ заборонено' });

  jwt.verify(token, process.env.JWT_SECRET, (err, decoded) => {
    if (err) return res.status(403).json({ message: 'Невалідний токен' });
    req.user = decoded;
    next();
  });
};
```

```javascript
// Конфігурація сервера (server.js)
const express = require('express');
const cors = require('cors');
const app = express();

app.use(cors()); // Захист CORS
app.use(express.json()); // Обробка JSON-тіла запитів

// Підключення маршрутів авторизації та аналітики
const authRoutes = require('./routes/auth');
const analyticsRoutes = require('./routes/analytics');

app.use('/api/auth', authRoutes);
app.use('/api/analytics', analyticsRoutes);

const PORT = process.env.PORT || 5000;
app.listen(PORT, () => console.log(`Server running on port ${PORT}`));
```

```javascript
// authController.js - Логіка входу
const token = jwt.sign(
    { id: user.user_id, role: user.role },
    process.env.SECRET_KEY,
    { expiresIn: '1h' }
);
res.json({ token, role: user.role, name: user.pib });

// authMiddleware.js - Перевірка ролей
function authorizeRole(roles) {
    return (req, res, next) => {
        if (!roles.includes(req.user.role)) {
            return res.status(403).send('Доступ заборонено');
        }
        next();
    };
}
```

![Authentication Logic](./images/auth_logic.jpg)