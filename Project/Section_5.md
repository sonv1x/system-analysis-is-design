## Розділ 5. Відображення User Stories у UML-нотації

У цьому розділі представлено діаграми послідовності (Sequence Diagrams) для двох ключових сценаріїв взаємодії користувача із системою.

### 5.1. Діаграма послідовності: Резервування книги Читачем

Цей сценарій демонструє процес онлайн-бронювання книги через веб-інтерфейс.

```mermaid
sequenceDiagram
    actor Reader as Читач
    participant UI as Веб-інтерфейс
    participant API as API Сервер
    participant DB as База Даних

    Reader->>UI: Вибирає книгу та натискає "Зарезервувати"
    UI->>API: POST /reservations (BookID, UserID)
    API->>DB: Перевірка статусу примірника
    DB-->>API: Статус примірника (Available)
    
    alt Примірник доступний
        API->>DB: Створення запису про резерв та оновлення статусу
        DB-->>API: Транзакція успішна
        API-->>UI: 201 Created
        UI-->>Reader: Повідомлення: "Книгу успішно зарезервовано"
    else Примірник вже зайнятий
        API-->>UI: 400 Bad Request
        UI-->>Reader: Повідомлення: "Вибачте, цей примірник недоступний"
    end
```
### 5.2. Діаграма послідовності: Видача книги Бібліотекарем

Цей сценарій описує робочий процес бібліотекаря при видачі книги користувачу та реєстрацію події в системі.

```mermaid
sequenceDiagram
    actor Librarian as Бібліотекар
    participant API as API Сервер
    participant DB as База Даних
    participant Notification as NotificationService

    Librarian->>API: POST /loans (ReaderID, BookID)
    API->>DB: Зміна статусу примірника на "Issued"
    API->>DB: Створення запису в таблиці обігу (due_date)
    DB-->>API: Транзакція успішна
    
    par Асинхронне сповіщення
        API->>Notification: Надіслати email-підтвердження Читачу
        Notification-->>API: Статус доставки (OK)
    end

    API-->>Librarian: Підтвердження: Видача зафіксована  
    
