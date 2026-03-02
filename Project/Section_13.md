## Розділ 13. Документування результатів проєкту

Фінальним результатом проєкту є повнофункціональна інформаційна система автоматизації бібліотеки, яка забезпечує повний цикл обліку книжкового фонду та взаємодії з читачами.

### 13.1. Технічні результати реалізації
* **Програмний продукт**: Розроблено багаторівневий веб-додаток. Бекенд-частина (Node.js/Express) забезпечує безпечну обробку даних через `bcryptjs` та `jsonwebtoken`. Фронтенд-частина (React) надає інтерактивний інтерфейс для управління бібліотекою.
* **База даних**: Спроектовано та впроваджено реляційну базу даних у PostgreSQL. Реалізовано 4 ключові таблиці (`users`, `books`, `book_items`, `loans`) з підтримкою каскадного видалення та цілісності посилань.
* **Аналітичний модуль**: Впроваджено систему візуалізації даних, яка за допомогою `Chart.js` відображає співвідношення виданих та доступних примірників у реальному часі.

### 13.2. Досягнуті показники автоматизації
* **Контроль заборгованостей**: Реалізовано алгоритм `overdue-check`, який автоматично ідентифікує протерміновані видачі, що значно знижує ризики втрати фонду.
* **Безпека**: Впроваджено рольову модель доступу (RBAC), що дозволяє чітко розмежовувати права між адміністраторами та бібліотекарями, захищаючи конфіденційні дані користувачів.
* **Масштабованість**: Архітектура системи дозволяє без додаткових змін інтегрувати нові модулі, такі як RFID-зчитувачі або Telegram-боти для сповіщень.



**Висновок**: Розроблена модель повністю відповідає поставленим завданням курсової роботи, забезпечуючи високу швидкість обробки інформації та надійність зберігання даних.

* **Візуалізація результатів**: Панель адміністратора
У системі реалізовано графічне відображення стану фонду за допомогою Chart.js.

```javascript
// Фрагмент коду (AdminPanel.js)
const chartData = {
  labels: ['Видані примірники', 'Доступні примірники'],
  datasets: [{
    label: 'Стан фонду',
    data: [stats.issuedCopies, (stats.totalCopies - stats.issuedCopies)],
    backgroundColor: ['rgba(255, 99, 132, 0.5)', 'rgba(75, 192, 192, 0.5)'],
    borderWidth: 1,
  }],
};
```

```javascript
// Ендпоінт для перевірки заборгованостей (analyticsRoutes.js)
router.post('/overdue-check', verifyToken, async (req, res) => {
  try {
    // SQL запит на пошук протермінованих книг
    const query = `
      SELECT loans.*, users.name, books.title 
      FROM loans 
      JOIN users ON loans.user_id = users.id
      JOIN book_items ON loans.item_id = book_items.id
      JOIN books ON book_items.book_id = books.id
      WHERE loans.due_date < CURRENT_DATE AND loans.return_date IS NULL
    `;
    const { rows } = await db.query(query);
    res.json({ success: true, count: rows.length, data: rows });
  } catch (err) {
    res.status(500).json({ message: 'Помилка сервера' });
  }
});
```

```javascript
// Бронювання книги 
const availableCopy = await db.Copy.findOne({
    where: { book_id: book_id, is_available: true, is_reserved: false }
});
if (availableCopy) {
    availableCopy.is_reserved = true;
    await availableCopy.save();
    await db.Reservation.create({ book_id, user_id, status: 'Active' });
}
```

![Librarian Issue Form](./images/librarian_issue.png)