---
title: Лаба 1
tags:
  - git
  - github
  - setup
---

## Вступление

Данная лабораторная работа посвящена настройке рабочего окружения и знакомству с системой контроля версий Git.

**Что будем делать:**

- Создание аккаунта на GitHub
- Настройка репозитория проекта
- Работа с `.gitignore`
- Первый коммит и push

!!! warning "Важно"
    Без сдачи этой лабораторной доступ к остальным **закрыт**. Нет в удалённом репо — нет лабы.

---

## Техническое задание

[Требования к программе, необходимые для сдачи.](task-1-1.md)

---

## Легенда

Ты — новый стажёр в IT-компании. Тебе выдали пустой репозиторий. Твоя задача — не засрать его мусором в первый же день.

---

## Шаг 1: Создание аккаунта на GitHub

Если у тебя ещё нет аккаунта:

1. Перейди на [github.com](https://github.com)
2. Нажми **Sign up**
3. Заполни форму регистрации
4. Подтверди email

---

## Шаг 2: Создание репозитория

1. На главной странице GitHub нажми **New repository**
2. Заполни поля:
    - **Repository name:** `CinemaKiosk_Фамилия` (например, `CinemaKiosk_Ivanov`)
    - **Description:** Проект киоска бронирования билетов
    - **Visibility:** Private
3. **НЕ** ставь галочки на "Add README" и "Add .gitignore" — сделаем это вручную
4. Нажми **Create repository**

---

## Шаг 3: Создание Solution в Visual Studio

1. Открой Visual Studio
2. **Create a new project** → **WPF Application**
3. Укажи путь к папке, которую будешь связывать с репозиторием
4. Название проекта: `CinemaKiosk`

---

## Шаг 4: Настройка .gitignore

Это **критически важный** шаг. Файл `.gitignore` указывает Git, какие файлы НЕ нужно отслеживать.

1. В корне проекта создай файл `.gitignore`
2. Скопируй содержимое из [gitignore.io](https://www.toptal.com/developers/gitignore/api/visualstudio) или используй шаблон:

```gitignore
# Visual Studio
.vs/
bin/
obj/
*.user
*.suo
*.cache

# Build results
[Dd]ebug/
[Rr]elease/

# NuGet
packages/
*.nupkg
```

!!! danger "Автоматическая пересдача"
    Если в репозитории окажутся папки `/bin`, `/obj` или `.vs` — это автоматическая пересдача. Гугли "Visual Studio gitignore".

---

## Шаг 5: Создание README.md

В корне проекта создай файл `README.md`:

```markdown
# CinemaKiosk

Проект киоска бронирования билетов в кинотеатр.

## Автор
- **ФИО:** Иванов Иван Иванович
- **Группа:** ИС-21

## Статус
- [ ] Лаба 1: Git Setup
- [ ] Лаба 2: Layout + HTTP
- [ ] Лаба 3: Data Binding
- [ ] Лаба 4: UserControls
- [ ] Лаба 5: Styles
- [ ] Лаба 6: Booking
```

---

## Шаг 6: Первый коммит и push

Открой терминал (Git Bash или PowerShell) в папке проекта:

```bash
# Инициализация репозитория
git init

# Добавление всех файлов
git add .

# Первый коммит
git commit -m "Initial commit: project setup"

# Связь с удалённым репозиторием
git remote add origin https://github.com/ТВОЙ_ЛОГИН/CinemaKiosk_Фамилия.git

# Push в main
git branch -M main
git push -u origin main
```

---

## Шаг 7: Приглашение преподавателя

1. Перейди в репозиторий на GitHub
2. **Settings** → **Collaborators**
3. Нажми **Add people**
4. Введи логин преподавателя
5. Отправь приглашение

---

## Проверка перед сдачей

Убедись, что:

- [ ] Репозиторий создан и приватный
- [ ] `.gitignore` настроен правильно
- [ ] В репозитории НЕТ папок `bin/`, `obj/`, `.vs/`
- [ ] Есть файл `README.md` с твоими данными
- [ ] Преподаватель получил приглашение в Collaborators

---

## Полезные ссылки

- [Git Documentation](https://git-scm.com/doc)
- [GitHub Docs](https://docs.github.com)
- [Генератор .gitignore](https://www.toptal.com/developers/gitignore)

