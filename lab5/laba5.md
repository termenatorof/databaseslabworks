
# Лабораторна робота 5: Нормалізація бази даних
**Виконали:** Шеремета Артем, Попович Мілана (Група ІО-46)

## 1. Початковий дизайн таблиць (Ненормалізована схема)

Для демонстрації процесу нормалізації, припустимо, що до створення поточної ER-діаграми всі дані про успішність студентів та їхні групи зберігалися в одній ненормалізованій зведеній відомості `student_records_draft`.

**Таблиця:** `student_records_draft`
**Стовпці:**
* `student_id` (PK)
* `first_name`
* `second_name`
* `group_name`
* `curator_name` (ПІБ куратора текстом)
* `faculty_name`
* `building`
* `courses_grades` (наприклад: "Організація баз даних: 95, ІПЗ: 94")

**Аналіз порушень:**
Поточна схема **не відповідає 1NF**, оскільки стовпець `courses_grades` містить множинні значення (повторювані групи) та не є атомарним. Крім того, присутні численні часткові та транзитивні залежності.

---

## 2. Функціональні залежності (ФЗ)

Якщо припустити, що ми розбили `courses_grades` на окремі атрибути `course_id`, `course_name` та `grade` для визначення залежностей, мінімальний набір функціональних залежностей для початкової предметної області буде наступним:

1. **ФЗ 1 (Повна залежність):** `{student_id, course_id} -> {grade}`
   *Оцінка залежить виключно від конкретного студента та конкретного курсу.*
2. **ФЗ 2 (Часткова залежність):** `student_id -> {first_name, second_name, group_name}`
   *Особисті дані студента та його група залежать лише від ідентифікатора студента.*
3. **ФЗ 3 (Часткова залежність):** `course_id -> {course_name}`
   *Назва дисципліни залежить лише від ID курсу.*
4. **ФЗ 4 (Транзитивна залежність):** `group_name -> {curator_name, faculty_name, building}`
   *Інформація про куратора та факультет залежить від групи, а не безпосередньо від студента.*
5. **ФЗ 5 (Транзитивна залежність):** `faculty_name -> {building}`
   *Номер корпусу залежить від факультету.*

---

## 3. Покрокове пояснення нормалізації

### Крок 1: Перехід до 1NF (Усунення повторюваних груп)
* **Проблема:** Поле `courses_grades` порушує вимогу атомарності атрибутів.
* **Рішення:** Розбиваємо множинне поле на окремі рядки. Тепер кожен запис містить лише один курс та одну оцінку. Первинний ключ стає складеним: `(student_id, course_id)`.
* **Результат (Таблиця в 1NF):** `student_1nf(student_id, course_id, first_name, second_name, group_name, curator_name, faculty_name, building, course_name, grade)`

### Крок 2: Перехід до 2NF (Усунення часткових залежностей)
* **Проблема:** Неключові атрибути (наприклад, `first_name` або `course_name`) залежать лише від частини складеного ключа (ФЗ 2 та ФЗ 3).
* **Рішення:** Декомпозуємо таблицю на три нові. Дані про студентів, дані про курси та таблицю зв'язку для оцінок.
* **Результат (Таблиці в 2NF):**
  * `students_2nf` (Ключ: `student_id`): `first_name, second_name, group_name, curator_name, faculty_name, building`
  * `courses_2nf` (Ключ: `course_id`): `course_name`
  * `enrollment` (Складений ключ: `student_id, course_id`): `grade`

### Крок 3: Перехід до 3NF (Усунення транзитивних залежностей)
* **Проблема:** У таблиці `students_2nf` атрибути `faculty_name` та `building` залежать від `group_name` (ФЗ 4, ФЗ 5). Також наявна аномалія оновлення: `curator_name` зберігається як звичайний текст, що дублює сутність викладача і може призвести до невідповідностей при зміні куратора.
* **Рішення:** Виокремлюємо факультети та групи у власні таблиці. Заміняємо текстове поле `curator_name` на зовнішній ключ `curator_id`, який посилається на таблицю викладачів (`teacher`).
* **Результат (Фінальні таблиці в 3NF):**
  * `faculty` (Ключ: `faculty_id`): `faculty_name, building`
  * `teacher` (Ключ: `teacher_id`): `first_name, second_name...`
  * `student_group` (Ключ: `group_id`): `group_name, start_year, faculty_id, curator_id`
  * `student` (Ключ: `student_id`): `first_name, second_name, group_id...`

---

## 4. Перероблений дизайн таблиць (SQL)

Нижче наведено команди створення (`CREATE TABLE`) для переглянутої схеми у 3NF. Зверніть увагу на таблицю `student_group`, де було усунуто текстове поле куратора і додано зовнішній ключ `curator_id`.

```sql
CREATE TABLE faculty(
    faculty_id SERIAL PRIMARY KEY,
    faculty_name VARCHAR(100) NOT NULL,
    building SMALLINT
);

CREATE TABLE teacher(
    teacher_id SERIAL PRIMARY KEY,
    first_name VARCHAR(25) NOT NULL,
    second_name VARCHAR(35) NOT NULL,
    phone_number CHAR(13) NOT NULL,
    email VARCHAR(64) UNIQUE NOT NULL,
    job teacher_role,
    status teacher_status NOT NULL,
    faculty_id INT NOT NULL REFERENCES faculty(faculty_id)
);

-- Таблиця нормалізована до 3NF: додано curator_id замість curator_name
CREATE TABLE student_group(
    group_id SERIAL PRIMARY KEY,
    group_name CHAR(7) NOT NULL CHECK (group_name LIKE '__-%'),
    start_year SMALLINT NOT NULL CHECK (start_year >= 1898),
    curator_id INT REFERENCES teacher(teacher_id) ON DELETE SET NULL,
    faculty_id INT NOT NULL REFERENCES faculty(faculty_id)
);

CREATE TABLE student(
    student_id SERIAL PRIMARY KEY,
    first_name VARCHAR(25) NOT NULL,
    second_name VARCHAR(35) NOT NULL,
    birth_date DATE NOT NULL,
    start_date DATE,
    end_date DATE,
    phone_number CHAR(13) NOT NULL,
    email VARCHAR(64) UNIQUE NOT NULL,
    course INT CHECK(course >= 1 AND course <= 6),
    status student_status NOT NULL,
    form_of_study student_form_of_study,
    finance_source student_finance_source,
    group_id INT NOT NULL REFERENCES student_group(group_id)
);
