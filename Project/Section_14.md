## Розділ 14. Визначення циклів у системі

У розробленій системі автоматизації бібліотеки виділено кілька типів циклів: життєві цикли об'єктів та замкнені контури керування.

**14.1. Життєвий цикл примірника книги (`book_items`)**
Це основний операційний цикл системи, що відображає рух активів:
1. **Надходження**: Створення запису в БД зі статусом `Available`.
2. **Видача**: Перехід у статус `Issued` при створенні запису в таблиці `loans`.
3. **Експлуатація**: Період перебування книги у читача (контролюється полем `due_date`).
4. **Повернення**: Оновлення статусу на `Available` та закриття запису в `loans`.
5. **Списання**: Видалення або зміна статусу на `Archived`.

**14.2. Контур зворотного зв'язку (Overdue Loop)**
Система реалізує циклічну перевірку заборгованостей:
* **Запит**: Функція `overdue-check` сканує таблицю `loans`.
* **Дія**: Якщо `current_date > due_date`, система маркує позику як протерміновану.
* **Результат**: Відображення в `Admin Dashboard` (через `Chart.js`), що спонукає бібліотекаря до адміністративного впливу.

```javascript
// Виконання циклу перевірки заборгованостей (overdue-check)
const handleCheckOverdue = () => {
  setIsLoading(true);
  api.post('/api/analytics/overdue-check')
    .then(res => {
      setMessage({ type: 'success', text: res.data.message });
      // Оновлення звітів після перевірки
      api.get('/api/analytics/overdue').then(res => setOverdue(res.data));
      api.get('/api/analytics/stats').then(res => setStats(res.data));
    })
    .catch(err => {
      setMessage({ type: 'error', text: 'Помилка виконання перевірки' });
    })
    .finally(() => setIsLoading(false));
};
```

* **Код автоматичного Email-сповіщення боржників з notificationService.js**:

```javascript
async sendDueReminder(email, bookTitle, dueDate) {
    const mailOptions = {
        from: 'zorinmisa554@gmail.com',
        to: email,
        subject: 'Нагадування про повернення книги',
        text: `Термін повернення книги "${bookTitle}" (дата: ${dueDate}) вже минув.`
    };
    await transporter.sendMail(mailOptions);
}
```

![Reservation Success](./images/reservation_success.png)
![Email Confirmation](./images/email_confirm.png)
![Email Conflict](./images/email_conflict.png)