..
Лабораторная работа №4

Тема: Анализ набора данных с помощью SQL
Предметная область: образовательная платформа
СУБД: SQLite
Файл базы: dataset.sqlite
Шаг 1. Загрузка базы и первичная проверка

В SandboxSQL загружена SQLite-база. Выполнена проверка числа строк во всех шести таблицах.

Количество записей.
sql

SELECT 'groups' AS table_name, COUNT(*) AS rows_count FROM groups
UNION ALL SELECT 'students', COUNT(*) FROM students
UNION ALL SELECT 'teachers', COUNT(*) FROM teachers
UNION ALL SELECT 'courses', COUNT(*) FROM courses
UNION ALL SELECT 'enrollments', COUNT(*) FROM enrollments
UNION ALL SELECT 'results', COUNT(*) FROM results;

Результат:

https://../images/step01_counts.png

Вывод: все шесть таблиц доступны, всего 2147 записей.
Шаг 2. Изучение структуры и формулирование аналитических вопросов

База содержит шесть связанных таблиц:
Таблица	Назначение	Основные поля	Записей
groups	Учебные группы	id, group_name, study_year	12
students	Студенты	id, full_name, group_id, admission_year	240
teachers	Преподаватели	id, full_name, department	15
courses	Курсы	id, course_name, hours, teacher_id	30
enrollments	Записи студентов на курсы	id, student_id, course_id, enrolled_at, status	1000
results	Итоговые оценки завершённых записей	id, enrollment_id, score, completed_at	850

Связи: students.group_id → groups.id; courses.teacher_id → teachers.id; enrollments.student_id → students.id; enrollments.course_id → courses.id; results.enrollment_id → enrollments.id. В каждой таблице id — первичный ключ. Для results.enrollment_id задана уникальность: одной записи на курс соответствует максимум одна оценка.

Просмотр данных. Просмотр первых пяти студентов.
sql

SELECT * FROM students LIMIT 5;

Результат:

https://../images/step02_students_data.png

Структура таблицы. Просмотр полей и типов данных таблицы students.
sql

PRAGMA table_info(students);

Результат:

https://../images/step02_students_schema.png

Внешние ключи. Просмотр связей таблицы enrollments.
sql

PRAGMA foreign_key_list(enrollments);

Результат:

https://../images/step02_enrollments_fk.png

Первичные и внешние ключи. Связи между таблицами

В базе данных используются первичные ключи (PRIMARY KEY) для идентификации записей и внешние ключи (FOREIGN KEY) для связи таблиц.

    groups.id — первичный ключ учебной группы.

    students.id — первичный ключ студента.

    teachers.id — первичный ключ преподавателя.

    courses.id — первичный ключ курса.

    enrollments.id — первичный ключ записи на курс.

    results.id — первичный ключ оценки.

Связи между таблицами:

    students.group_id → groups.id — каждый студент относится к конкретной группе.

    courses.teacher_id → teachers.id — каждый курс закреплён за конкретным преподавателем.

    enrollments.student_id → students.id — запись на курс сделана конкретным студентом.

    enrollments.course_id → courses.id — запись сделана на конкретный курс.

    results.enrollment_id → enrollments.id — оценка привязана к конкретной записи на курс.

Таким образом, таблица enrollments связывает студентов и курсы, а таблица results позволяет получить информацию об успеваемости студентов по конкретным курсам.

Сформулированные аналитические вопросы:

    (Вариант 2) Какие учебные группы показывают лучшие средние результаты обучения? Сравните группы с учётом числа оценок. (groups, students, enrollments, results)

    Какие курсы имеют самую высокую и самую низкую среднюю оценку? (courses, enrollments, results)

    Сколько студентов записалось на каждый курс и какой процент успешно завершил обучение? (courses, enrollments)

    Какие преподаватели ведут наибольшее количество курсов? (teachers, courses)

    Как соотносятся длительность курса (hours) и средний балл по нему? (courses, enrollments, results)

    Есть ли студенты, которые записались на курсы, но не получили ни одной оценки? (students, enrollments, results)

    Какие группы имеют наибольшее количество отличных оценок (>= 90)? (groups, students, enrollments, results)

Шаг 3. Фильтрация и сортировка

Фильтрация по длительности. Отбираем курсы длительностью не менее 60 часов.
sql

SELECT id, course_name, hours FROM courses WHERE hours >= 60 ORDER BY hours DESC, course_name ASC;

Результат:

https://../images/step03_courses.png

Сортировка студентов. Выводим студентов, поступивших в 2025 году, по алфавиту.
sql

SELECT id, full_name, admission_year FROM students WHERE admission_year = 2025 ORDER BY full_name ASC LIMIT 10;

Результат:

https://../images/step03_students.png

Вывод: WHERE отбирает нужные строки, ORDER BY сортирует, LIMIT ограничивает объём вывода.
Шаг 4. Агрегирование и группировка

Подсчёт записей. Определяем общее число записей на курсы.
sql

SELECT COUNT(*) AS total_enrollments FROM enrollments;

Результат:

https://../images/step04_count.png

Агрегатные функции. Рассчитываем минимальную, максимальную, среднюю и суммарную продолжительность курсов.
sql

SELECT MIN(hours) AS min_hours, MAX(hours) AS max_hours, ROUND(AVG(hours), 2) AS avg_hours, SUM(hours) AS total_hours FROM courses;

Результат:

https://../images/step04_aggregates.png

GROUP BY и HAVING. Группируем записи по статусу и оставляем статусы, встречающиеся более 100 раз.
sql

SELECT status, COUNT(*) AS enrollments_count FROM enrollments GROUP BY status HAVING COUNT(*) > 100 ORDER BY enrollments_count DESC;

Результат:

https://../images/step04_having.png

Вывод: в базе 1000 записей на курсы. HAVING отбирает группы после агрегирования; условие применяется к COUNT(*).
Шаг 5. Объединение связанных таблиц

Студенты и группы. Используем INNER JOIN, чтобы показать первых 10 студентов вместе с названиями групп.
sql

SELECT s.id, s.full_name, g.group_name FROM students AS s INNER JOIN groups AS g ON s.group_id = g.id ORDER BY s.id LIMIT 10;

Результат:

https://../images/step05_students_groups.png

Курсы и преподаватели. Используем INNER JOIN, чтобы показать первые 10 курсов и их преподавателей.
sql

SELECT c.course_name, t.full_name AS teacher_name FROM courses AS c INNER JOIN teachers AS t ON c.teacher_id = t.id ORDER BY c.id LIMIT 10;

Результат:

https://../images/step05_courses_teachers.png

Все преподаватели и курсы. Используем LEFT JOIN, чтобы сохранить в результате всех преподавателей.
sql

SELECT t.full_name AS teacher_name, c.course_name FROM teachers AS t LEFT JOIN courses AS c ON t.id = c.teacher_id ORDER BY t.full_name;

Результат:

https://../images/step05_teachers_leftjoin.png

Вывод: INNER JOIN выводит совпавшие записи, а LEFT JOIN сохраняет строки из левой таблицы даже при отсутствии соответствий.
Шаг 6. Проверка корректности данных

Проверка оценок. Проверяем, нет ли оценок меньше 0, больше 100 или отсутствующих.
sql

SELECT COUNT(*) AS invalid_scores FROM results WHERE score < 0 OR score > 100 OR score IS NULL;

Результат:

https://../images/step06_invalid.png

Проверка связей. Проверяем, нет ли оценок, ссылающихся на несуществующую запись на курс.
sql

SELECT COUNT(*) AS orphan_results FROM results AS r LEFT JOIN enrollments AS e ON r.enrollment_id = e.id WHERE e.id IS NULL;

Результат:

https://../images/step06_orphans.png

Вывод: оба запроса вернули 0. В рамках проведённых проверок нарушений не обнаружено. Это не означает, что проверены абсолютно все возможные ошибки в данных.
Шаг 7. Индивидуальное аналитическое исследование

Вариант: 2.

Исследовательский вопрос: какие учебные группы показывают лучшие средние результаты обучения, с учётом числа оценок?

Используемые таблицы: groups, students, enrollments, results.

Метод исследования: с помощью INNER JOIN, GROUP BY, COUNT и AVG определяем количество выставленных оценок и средний балл отдельно по учебным группам. Дополнительно рассчитываем общий средний балл по всей базе для сравнения.

Для ответа на исследовательский вопрос выполняются два основных SQL-запроса:

Аналитика по группам. Группируем данные по учебным группам.
sql

SELECT g.group_name, COUNT(r.score) AS grades_count, ROUND(AVG(r.score), 2) AS avg_score FROM groups AS g JOIN students AS s ON g.id = s.group_id JOIN enrollments AS e ON s.id = e.student_id JOIN results AS r ON e.id = r.enrollment_id WHERE r.score IS NOT NULL GROUP BY g.id, g.group_name ORDER BY avg_score DESC, grades_count DESC;

Результат:

https://../images/step07_groups_avg.png

Общий средний балл по базе. Сравнительный показатель.
sql

SELECT ROUND(AVG(score), 2) AS overall_avg_score FROM results WHERE score IS NOT NULL;

Результат:

https://../images/step07_overall_avg.png

Аналитические выводы:

    Наивысший средний балл показывает группа ИВТ-09 — 80.09 (учтено 75 оценок). Это превышает общий средний балл по базе (77.58) на 2.51 пункта.

    При сравнении важно учитывать столбец grades_count. Группа ИВТ-02 занимает второе место (79.87), но имеет меньшее количество оценок (61). Напротив, группа ИВТ-07 имеет максимальное число оценок (80), но её средний балл (76.95) ниже общего. Большой объем выборки сглаживает случайные отклонения.

    Наименьший средний результат демонстрирует группа ИВТ-12 (75.26). Разрыв между лучшей (ИВТ-09) и худшей (ИВТ-12) группой составляет 4.83 балла. Разброс средних баллов невелик, что указывает на относительно однородный уровень подготовки.

Итог

В работе изучена структура SQLite-базы из шести связанных таблиц, выполнены запросы с WHERE, ORDER BY, LIMIT, агрегатными функциями, GROUP BY, HAVING, INNER JOIN, LEFT JOIN; проведены проверки целостности и отдельное аналитическое исследование (Вариант 2).

Ограничения: анализ не учитывает сложность курсов, которые выбирали студенты разных групп. Кроме того, в расчет берутся только завершенные курсы с оценками; студенты, которые записались, но бросили курс, не влияют на средний балл группы, что может скрывать реальный процент отсева. Выводы относятся только к данному учебному набору данных.
Структура файлов на GitHub
text

lab4/
├── README.md
├── report/
│   └── analysis.md
└── images/
    ├── step01_counts.png
    ├── step02_students_data.png
    ├── step02_students_schema.png
    ├── step02_enrollments_fk.png
    ├── step03_courses.png
    ├── step03_students.png
    ├── step04_count.png
    ├── step04_aggregates.png
    ├── step04_having.png
    ├── step05_students_groups.png
    ├── step05_courses_teachers.png
    ├── step05_teachers_leftjoin.png
    ├── step06_invalid.png
    ├── step06_orphans.png
    ├── step07_groups_avg.png
    └── step07_overall_avg.png
