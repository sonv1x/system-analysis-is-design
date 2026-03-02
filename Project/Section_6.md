## Розділ 6. Архітектура системи та залежності

**6.1. Логічна архітектурна модель**
Система базується на трирівневій архітектурі (Three-Tier Architecture):
* **Presentation Layer**: Реалізований на **React** для створення інтерактивного інтерфейсу користувача.
* **Business Logic Layer**: Побудований на **Node.js (Express)** для обробки запитів та керування логікою бібліотеки.
* **Data Layer**: Використовує **PostgreSQL** для збереження даних про фонд та користувачів.

**6.2. Зовнішні системи**
* **Email Service**: Інтеграція для автоматичного надсилання нагадувань боржникам.

**6.3. Діаграма компонентів**

```mermaid
graph TD
    User((Користувач)) --> Web[React Frontend]
    subgraph Server
        Web --> API[Node.js API]
        API --> Logic[Business Logic]
    end
    Logic --> DB[(PostgreSQL)]
    Logic --> Mail[Email Service] 
```


**6.4. Програмна реалізація точки входу (server.js)**
* **Головна точка входу бекенду `server.js` описує взаємодію модулів та запуск сервера**:

```javascript
const express = require('express');
const sequelize = require('./models').sequelize;
const authRoutes = require('./routes/authRoutes');
const apiRoutes = require('./routes/apiRoutes');

const app = express();
app.use(express.json());
app.use('/auth', authRoutes);
app.use('/api', apiRoutes);

const startServer = async () => {
    try {
        await sequelize.authenticate();
        await sequelize.sync({ alter: true });
        app.listen(3001, () => console.log('Сервер запущено на порту 3001'));
    } catch (error) {
        console.error('Помилка запуску:', error);
    }
};

startServer(); 
```

![ER Diagram](./images/er_diagram.png)