# Лабораторна робота № 6
# ІО-46 Шеремета Артем,Попович Мілана
## Тема: Міграції схем за допомогою Prisma ORM
---
## Цілі
Використати Prisma ORM для керування схемами та дослідити, як Prisma може аналізувати та змінювати схему вашої бази даних.
Застосування міграцій - генерування та застосування змін схеми (таблиць, стовпців, зв'язків) за допомогою prisma migrate.
Моделювання за допомогою файлів схеми Prisma - визначення таблиць та зв'язків у schema.prisma та перегляд їхнього відображення в PostgreSQL.
Виконати базові запити Prisma - вставити та запитати дані за допомогою клієнта Prisma (через Prisma Studio або простий скрипт) для перевірки змін.
---
## Результати
Спочатку було ініціалізовано середовище Prisma ORM та налаштовано підключення до нашої існуючої бази даних, розробленої під час лабораторної роботи №5.
<img width="575" height="644" alt="image" src="https://github.com/user-attachments/assets/b1fd22b1-a684-41a6-b66d-cb1c310817ae" />

Команда ініціалізації автоматично згенерувала необхідні конфігураційні файли:

.env — містить рядок підключення (DATABASE_URL), через який ми надаємо Prisma безпечний доступ безпосередньо до нашої PostgreSQL бази.

schema.prisma — основний файл схеми, куди за допомогою команди npx prisma db pull було імпортовано (інтроспектовано) нашу поточну структуру таблиць.
---
## 1. Додавання Нової таблиці 
додаємо таблицю Гуртожитку 
Код:
model dormitory {
  dormitory_id Int       @id @default(autoincrement())
  number       Int
  address      String    @db.VarChar(150)
  students     student[] 
}

До 
<img width="1026" height="708" alt="image" src="https://github.com/user-attachments/assets/fb753bab-e574-4178-9e57-f8b045db3ffc" />

після
<img width="393" height="458" alt="image" src="https://github.com/user-attachments/assets/59748765-1bd6-4c32-add4-e8f62e114871" />


## 2. Оновлення таблиці
оновлюємо таблиці студент та додаємо туди рядки про гуртожиток
код:
dormitory_id   Int?
  dormitory      dormitory? @relation(fields: [dormitory_id], references: [dormitory_id])

До
<img width="710" height="311" alt="image" src="https://github.com/user-attachments/assets/f3dcfdc2-a315-4c18-86e1-2fe3a1675ac4" />

після
<img width="408" height="393" alt="image" src="https://github.com/user-attachments/assets/a25879ea-f9a6-49b9-9dda-de0413bfeb9b" />
