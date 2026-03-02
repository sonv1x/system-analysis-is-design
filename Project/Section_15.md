## Розділ 15. Визначення артефакту системи

Відповідно до класифікації системних об'єктів та результатів моделювання, розроблена система відповідає артефакту **«Інформаційна модель керування ресурсами» (Digital Twin of Process)**.

**15.1. Чому це артефакт управління?**
Система не просто зберігає дані, а є цифровим відображенням реальних процесів бібліотеки:
* **Цифровий двійник фонду**: Кожен запис у таблиці `books` та `book_items` точно відповідає фізичній одиниці зберігання.
* **Транзакційна модель**: Кожна дія (видача/повернення) фіксується як подія, що змінює стан усієї системи.

**15.2. Відповідність технічним критеріям**
* **Структурність**: Система має чітку схему даних (ER-діаграма), що визначає правила взаємодії об'єктів.
* **Функціональність**: Наявність аналітичного модуля (`stats` та `analytics/overdue`) перетворює систему з простого сховища на інструмент підтримки прийняття рішень для адміністратора.


* **Код глобального стану AuthContext.js, що робить систему цілісною**:

```javascript
export const AuthProvider = ({ children }) => {
    const [user, setUser] = useState(null);
    const navigate = useNavigate();

    const login = async (email, password) => {
        try {
            const response = await api.post('/auth/login', { email, password });
            const { token, role, name } = response.data;
            localStorage.setItem('token', token);
            setUser({ name, role });
            if (role === 'Administrator') navigate('/admin');
            else if (role === 'Librarian') navigate('/librarian');
            else navigate('/catalog');
        } catch (error) {
            console.error("Помилка логіну:", error);
            throw error;
        }
    }; 

    const logout = () => {
        localStorage.removeItem('token');
        setUser(null);
        navigate('/login');
    };

    return (
        <AuthContext.Provider value={{ user, login, logout }}>
            {children}
        </AuthContext.Provider>
    );
};
```

![Reader Catalog](./images/reader_catalog.jpg)