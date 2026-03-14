# Personal Assistant — Технічна документація

> **Версія:** 1.0.0
> **Python:** 3.10+
> **Автори:** Олександр, Марія, Емма, В'ячеслав

---

## 1. Загальний опис

**Personal Assistant** — це консольний «персональний помічник» для керування контактами та нотатками. Додаток підтримує два режими роботи:

- **Класичний режим** (`--classic`) — текстовий інтерфейс командного рядка з REPL-циклом, кольоровим виводом і підказками.
- **TUI-режим** (за замовчуванням) — повноекранний псевдографічний інтерфейс на базі бібліотеки `asciimatics` з навігацією, формами та grid-таблицями.

Усі дані (контакти, нотатки) зберігаються в JSON-файлах на диску та автоматично відновлюються при наступному запуску.

---

## 2. Встановлення

### Вимоги

- Python 3.10 або новіший
- pip

### Крок 1 — Клонування репозиторію

```bash
git clone https://github.com/vstrelnikof/goit-pycore-hw-final.git
cd goit-pycore-hw-final
```

### Крок 2 — Створення та активація віртуального середовища

```bash
# macOS / Linux
python -m venv .venv
source .venv/bin/activate

# Windows (cmd)
python -m venv .venv
.venv\Scripts\activate

# Windows (PowerShell)
python -m venv .venv
.venv\Scripts\Activate.ps1
```

### Крок 3 — Встановлення залежностей

```bash
# Звичайне встановлення
pip install .

# Режим розробки (зміни у коді підхоплюються без перевстановлення)
pip install -e .

# З dev-залежностями (тести, linting)
pip install -e ".[dev]"
```

### Крок 4 — Запуск

```bash
# TUI-режим (за замовчуванням)
personal-assistant

# Класичний консольний режим
personal-assistant --classic

# Якщо встановлення не виконувалось
python main.py
python main.py --classic
```

---

## 3. Конфігурація

Налаштування задаються у файлі `config.yaml` у корені проекту. Аргументи командного рядка мають вищий пріоритет.

### config.yaml

```yaml
app:
  log_level: 20          # 10=DEBUG, 20=INFO, 30=WARNING, 40=ERROR, 50=CRITICAL
  theme: "default"       # default / monochrome / green / bright
  classic: false         # true — запуск у класичному режимі
  app_data_paths:
    address_book: data/contacts.json
    notes: data/notes.json
```

| Параметр | Тип | За замовч. | Опис |
|---|---|---|---|
| `log_level` | int | `20` (INFO) | Рівень логування у файл `assistant.log` |
| `theme` | string | `"default"` | Тема TUI-інтерфейсу |
| `classic` | bool | `false` | Режим класичного CLI |
| `app_data_paths.address_book` | string | `data/contacts.json` | Шлях до файлу контактів |
| `app_data_paths.notes` | string | `data/notes.json` | Шлях до файлу нотаток |

### Аргументи командного рядка

| Аргумент | Опис |
|---|---|
| `--classic` | Увімкнути класичний CLI-режим |
| `--theme <назва>` | Вибір теми TUI (`default`, `monochrome`, `green`, `bright`) |
| `--log-level <число>` | Рівень логування |
| `--config <профіль>` | Альтернативний конфіг-файл або профіль (`dev`, `demo`) |
| `--create-fakes-contacts <N>` | Згенерувати N фейкових контактів перед запуском |
| `--create-fakes-notes <N>` | Згенерувати N фейкових нотаток перед запуском |

### Профілі конфігурації

Можна створювати файли `config.<профіль>.yaml` і вибирати їх через `--config`:

```bash
python main.py --config dev      # використовує config.dev.yaml
python main.py --config demo     # використовує config.demo.yaml
```

---

## 4. Архітектура та структура проекту

```
goit-pycore-hw-final/
│
├── main.py                    # Точка входу
├── config.yaml                # Конфігурація
│
├── models/                    # Моделі даних
│   ├── base_model.py          # Абстрактна базова модель
│   ├── contact.py             # Модель контакту
│   ├── note.py                # Модель нотатки
│   ├── contact_birthday.py    # NamedTuple для дня народження
│   ├── app_config.py          # Pydantic-модель конфігурації
│   ├── app_data_paths.py      # Pydantic-модель шляхів
│   └── table_row.py           # Рядок таблиці для TUI
│
├── services/                  # Бізнес-логіка
│   ├── base_service.py        # Абстрактний базовий сервіс
│   ├── address_book_service.py # Сервіс адресної книги
│   └── notes_service.py       # Сервіс нотаток
│
├── providers/                 # Постачальники даних
│   ├── config_provider.py     # Завантаження конфігурації
│   └── storage_provider.py    # JSON-сховище
│
├── utils/                     # Утиліти
│   ├── state.py               # AppState — контейнер стану
│   ├── validator.py           # Валідатори полів
│   ├── fake_data.py           # Генератор тестових даних
│   └── wrapped_func_formatter.py # Форматер логів
│
├── helpers/                   # Допоміжні функції
│   └── date_helpers.py        # Операції з датами
│
├── decorators/                # Декоратори
│   ├── log_action.py          # @log_action
│   └── log_command_action.py  # @log_command_action
│
├── enums/                     # Перелічення
│   ├── scene_type.py          # Типи TUI-сцен
│   └── theme_type.py          # Теми TUI
│
├── factories/                 # Фабрики
│   └── scene_factory.py       # Фабрика TUI-сцен
│
├── cli/
│   ├── classic/               # Класичний CLI-інтерфейс
│   │   ├── runner.py          # REPL-цикл
│   │   ├── dispatcher.py      # Роутинг команд
│   │   ├── handler.py         # Обробник підкоманд
│   │   ├── renderer.py        # Форматування виводу
│   │   ├── colors.py          # ANSI-кольори
│   │   └── forms/             # Консольні форми
│   │       ├── base_form.py
│   │       ├── contact_form.py
│   │       └── note_form.py
│   │
│   └── tui/                   # TUI-інтерфейс (asciimatics)
│       ├── base_frame.py      # Базовий фрейм
│       ├── forms/             # TUI-форми
│       │   ├── base_form.py
│       │   ├── contact_form.py
│       │   └── note_form.py
│       └── views/             # TUI-перегляди
│           ├── base_grid_view.py
│           ├── birthday_grid_view.py
│           ├── contact_grid_view.py
│           ├── dashboard_view.py
│           └── note_grid_view.py
│
├── data/                      # Файли даних
│   ├── contacts.json
│   └── notes.json
│
└── tests/                     # Автоматизовані тести
    ├── test_config_provider.py
    ├── test_date_helpers.py
    ├── test_dispatcher.py
    ├── test_fake_data.py
    ├── test_models.py
    ├── test_renderer.py
    ├── test_services.py
    ├── test_state.py
    ├── test_storage_provider.py
    ├── test_table_row.py
    └── test_validator.py
```

### Схема залежностей між шарами

```
main.py
  └─► providers/config_provider     (завантаження конфігу)
  └─► utils/state (AppState)
        └─► services/address_book_service
        │     └─► providers/storage_provider
        │     └─► models/contact
        └─► services/notes_service
              └─► providers/storage_provider
              └─► models/note
  └─► cli/classic/runner   або   cli/tui/* (режим з конфігу/аргументів)
```

---

## 5. Моделі даних

### 5.1 `BaseModel` (`models/base_model.py`)

Абстрактний датаклас, від якого успадковуються всі бізнес-моделі.

| Метод | Опис |
|---|---|
| `is_valid() → bool` | Перевіряє валідність об'єкта (абстрактний) |
| `to_dict() → dict` | Серіалізує об'єкт у словник для збереження |
| `from_dict(data) → Self` | Десеріалізує об'єкт зі словника |
| `_generate_id() → str` | Генерує унікальний UUID |

### 5.2 `Contact` (`models/contact.py`)

Модель контакту адресної книги.

| Поле | Тип | Опис |
|---|---|---|
| `id` | `str` | UUID контакту |
| `name` | `str` | Ім'я та прізвище |
| `phone` | `str` | Номер телефону (формат E.164, напр. `+380501234567`) |
| `email` | `str` | Електронна пошта |
| `address` | `str` | Адреса (довільний текст) |
| `birthday` | `str` | День народження у форматі `DD.MM.YYYY` |

### 5.3 `Note` (`models/note.py`)

Модель нотатки.

| Поле | Тип | Опис |
|---|---|---|
| `id` | `str` | UUID нотатки |
| `text` | `str` | Текст нотатки (може бути багаторядковим) |
| `tags` | `list[str]` | Список тегів (ключових слів) |

### 5.4 Формат зберігання (JSON)

Обидва файли мають однакову структуру — масив об'єктів:

```json
[
  {
    "id": "3f1a2b4c-...",
    "name": "Іван Петренко",
    "phone": "+380501234567",
    "email": "ivan@example.com",
    "address": "м. Київ, вул. Хрещатик, 1",
    "birthday": "15.03.1990"
  }
]
```

---

## 6. Сервіси

### 6.1 `AddressBookService` (`services/address_book_service.py`)

Відповідає за весь функціонал адресної книги.

| Метод | Опис |
|---|---|
| `add(form_data: dict) → Contact` | Додати новий контакт |
| `edit(index: int, form_data: dict) → Contact` | Редагувати контакт за індексом у відфільтрованому списку |
| `delete(index: int) → None` | Видалити контакт |
| `get_table_data(search: str) → TableData` | Отримати контакти у вигляді рядків таблиці (з пошуковим фільтром) |
| `get_birthday_table_data(days: int) → TableData` | Контакти з днями народження протягом `days` днів |
| `get_dashboard_birthdays() → list` | Список найближчих днів народження для dashboard |
| `save() → None` | Зберегти контакти в JSON |
| `reload() → None` | Перезавантажити контакти з JSON |

### 6.2 `NotesService` (`services/notes_service.py`)

Відповідає за весь функціонал нотаток.

| Метод | Опис |
|---|---|
| `add(form_data: dict) → Note` | Додати нотатку |
| `edit(index: int, form_data: dict) → Note` | Редагувати нотатку |
| `delete(index: int) → None` | Видалити нотатку |
| `get_table_data(search: str, desc: bool) → TableData` | Нотатки у вигляді рядків таблиці (пошук за текстом та тегами) |
| `save() → None` | Зберегти нотатки в JSON |
| `reload() → None` | Перезавантажити нотатки з JSON |

---

## 7. Постачальники (Providers)

### 7.1 `ConfigProvider` (`providers/config_provider.py`)

- Зчитує `config.yaml` (або профільний файл через `--config`)
- Парсить аргументи командного рядка через `argparse`
- Аргументи CLI мають пріоритет над значеннями з файлу
- Повертає об'єкт `AppConfig`

### 7.2 `StorageProvider` (`providers/storage_provider.py`)

- При ініціалізації створює JSON-файл (якщо не існує) з порожнім масивом `[]`
- `load_list()` — генератор, що повертає словники із файлу
- `save(data: list[dict])` — записує дані у файл
- Декорований `@log_action` для аудиту дій зі збереженням

---

## 8. Валідація

Клас `Validator` (`utils/validator.py`) містить статичні методи:

| Метод | Правило |
|---|---|
| `is_valid_phone(phone)` | E.164: `+` і від 10 до 15 цифр |
| `is_valid_email(email)` | Стандартний формат `user@domain.ext` |
| `is_valid_date(date)` | Формат `DD.MM.YYYY` та реальне існування дати |
| `is_valid_days(days)` | Рядок з лише цифрами (> 0) |
| `matches_search_term(value, term)` | Підтримує wildcard `*` та регістронезалежне порівняння |

---

## 9. Класичний CLI-режим

### Запуск

```bash
python main.py --classic
```

### Список команд

| Команда | Опис |
|---|---|
| `help` | Список усіх команд |
| `stats` / `dashboard` | Статистика: кількість контактів, нотаток, найближчі ДН |
| `contacts search [термін] [N]` | Показати/знайти контакти (wildcard `*` підтримується) |
| `contacts add` | Інтерактивне додавання контакту |
| `contacts edit N` | Редагувати контакт за номером із результатів search |
| `contacts delete N` | Видалити контакт |
| `birthdays [N]` | Дні народження на N найближчих днів (за замовч. 7) |
| `notes search [термін] [N]` | Показати/знайти нотатки за текстом або тегом |
| `notes add` | Інтерактивне додавання нотатки (підтримка багаторядкового тексту) |
| `notes show N` | Переглянути повний текст нотатки |
| `notes edit N` | Редагувати нотатку |
| `notes delete N` | Видалити нотатку |
| `exit` / `quit` | Завершити роботу |

### Особливості

- **Fuzzy matching:** якщо команду введено неправильно (наприклад `contacs`), система автоматично знаходить найближчу відому команду через `difflib`.
- **Wildcard-пошук:** символ `*` у пошуковому терміні замінює будь-яку кількість символів. Наприклад, `jo*n` знайде «john», «jon», «jordan».
- **Пагінація:** можна обмежити кількість відображених рядків: `contacts search 10`, `notes search python 5`.
- **Кольоровий вивід:** ANSI-кольори для заголовків, успіху, помилок та підказок (підтримка Windows через colorama).

---

## 10. TUI-режим (asciimatics)

### Запуск

```bash
python main.py
# або після встановлення:
personal-assistant
```

### Навігація

| Клавіша / дія | Опис |
|---|---|
| `Enter` | Вибір пункту / активація кнопки |
| `Esc` | Повернутися назад / скасувати форму |
| `Tab` | Перехід між полями форми |
| Стрілки | Навігація у списках |
| Кнопки Create / Edit / Delete | Виклик відповідних дій |

### Екрани

| Екран | Опис |
|---|---|
| **Dashboard** | Головний екран: статистика (кількість контактів і нотаток, дата), найближчі дні народження, меню навігації |
| **Contacts Grid** | Повний список контактів із пошуковим полем, кнопками Create / Edit / Delete |
| **Contact Form** | Форма для створення/редагування контакту з inline-валідацією |
| **Notes Grid** | Список нотаток з пошуком за текстом і тегами, сортування (чекбокс) |
| **Note Form** | Форма для нотатки: багаторядковий текст, поле тегів через кому |
| **Birthdays Grid** | Таблиця контактів із найближчими ДН; поле для введення кількості днів |

### Теми

Вибір теми в `config.yaml` або через `--theme`:

| Тема | Опис |
|---|---|
| `default` | Стандартна кольорова схема |
| `monochrome` | Чорно-білий режим |
| `green` | Термінал у стилі «зелений-на-чорному» |
| `bright` | Яскраві кольори |

---

## 11. Логування

Усі дії програми записуються у файл `assistant.log` у корені проекту.

- Рівень логування задається через `log_level` у `config.yaml` або `--log-level`.
- Декоратор `@log_action` автоматично логує початок та кінець виконання будь-якої функції.
- Декоратор `@log_command_action` додатково фіксує час виконання команди.
- `WrappedFuncFormatter` забезпечує відображення імені оригінальної функції (а не декоратора) у логах.

---

## 12. Тестування

### Запуск тестів

```bash
# Усі тести
pytest

# З виводом деталей
pytest -v

# Лише певний модуль
pytest tests/test_services.py -v

# З покриттям
pytest --cov=. --cov-report=term-missing
```

### Покриття тестами

| Модуль | Тестовий файл |
|---|---|
| `providers/config_provider` | `tests/test_config_provider.py` |
| `providers/storage_provider` | `tests/test_storage_provider.py` |
| `helpers/date_helpers` | `tests/test_date_helpers.py` |
| `models/*` | `tests/test_models.py` |
| `models/table_row` | `tests/test_table_row.py` |
| `utils/validator` | `tests/test_validator.py` |
| `utils/fake_data` | `tests/test_fake_data.py` |
| `utils/state` | `tests/test_state.py` |
| `services/*` | `tests/test_services.py` |
| `cli/classic/dispatcher + handler` | `tests/test_dispatcher.py` |
| `cli/classic/renderer` | `tests/test_renderer.py` |

---

## 13. Залежності

### Основні (`requirements.txt`)

| Бібліотека | Призначення |
|---|---|
| `asciimatics` | TUI-інтерфейс (псевдографічні вікна) |
| `pydantic` | Валідація конфігураційних моделей |
| `pyyaml` | Читання `config.yaml` |
| `faker` | Генерація тестових даних (uk_UA) |
| `colorama` | ANSI-кольори на Windows |

### Dev-залежності (`requirements-dev.txt`)

| Бібліотека | Призначення |
|---|---|
| `pytest` | Фреймворк тестування |
| `pytest-cov` | Звіт покриття тестами |
| `ruff` | Linting та форматування коду |

---

## 14. Розробка та налаштування середовища

### Налаштування git-hooks

```bash
# macOS / Linux
bash scripts/setup-hooks.sh

# Windows (PowerShell)
./scripts/setup-hooks.ps1
```

### Генерація тестових даних

```bash
python main.py --create-fakes-contacts 20 --create-fakes-notes 15
```

### Структура коміту

Рекомендований формат повідомлення коміту:

```
<тип>: <короткий опис>

Типи: feat | fix | docs | test | refactor | chore
```

Приклади:

```
feat: add wildcard support to contact search
fix: correct leap-year birthday calculation
test: add coverage for NotesService.delete
docs: update installation guide for Windows
```

---

## 15. Розподіл роботи в команді

| Учасник | Роль | Відповідальність | % |
|---|---|---|---|
| **Олександр** | Scrum Master | `main.py`, конфігурація, логування, провайдери, enums, helpers | ~20% |
| **Марія** | Team Lead | Моделі, сервіси, валідація, стан застосунку, фабрики | ~20% |
| **Емма** | Developer | Класичний CLI (runner, dispatcher, handler, renderer, форми) | ~30% |
| **В'ячеслав** | Developer | TUI-інтерфейс (фрейми, форми, grid-views, dashboard) | ~30% |
