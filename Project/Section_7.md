## Розділ 7. Побудова моделі даних

**7.1. Основні сутності та атрибути**
* **Users**: `user_id` (PK), `full_name`, `email`, `role`.
* **Books**: `book_id` (PK), `title`, `author`, `isbn`.
* **Inventory_Items**: `item_id` (PK), `book_id` (FK), `status` (Available, Reserved, Issued).
* **Loans**: `loan_id` (PK), `user_id` (FK), `item_id` (FK), `issue_date`, `due_date`.

**7.2. ER-діаграма**

```mermaid
erDiagram
    USER ||--o{ LOAN : "здійснює"
    BOOK ||--|{ INVENTORY_ITEM : "має"
    INVENTORY_ITEM ||--o{ LOAN : "бере участь у"
    
    USER {
        int user_id PK
        string full_name
        string role
    }
    BOOK {
        int book_id PK
        string title
        string author
    }
    LOAN {
        int loan_id PK
        date due_date
    }