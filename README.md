# plusone-creative

Плагін для Claude Code, який автоматизує створення Instagram-креативів для клієнтів Plusone. Витягує візуальний стиль з конкретного Figma-проекту, генерує дудли і фото через Higgsfield, складає готові кадри назад у Figma.

> Призначено для дизайнерів. Запускається через Cowork tab у Claude Desktop. Деталі установки і перший запуск — у [HANDOFF.md](HANDOFF.md).

---

## Що робить

Запускаєш одну команду у Cowork — отримуєш N готових постів у стилі вказаного Figma-проекту:

1. Аналізує референсний Figma-файл — дудли, фото, палітру, типографіку, шаблон.
2. Генерує асети через Higgsfield — прозорі PNG-дудли (gpt-image-1.5) + фото (nano-banana).
3. Складає N кадрів у Figma — дублює шаблон, наливає тексти, проставляє image-fills.
4. Перевіряє кожен кадр — overflow, безшовність стилів, заповнені плейсхолдери.
5. Звітує тобі — лінки на готові кадри + список того, що варто дотягнути руками.

---

## Передумови

Перед першим запуском:

- **Claude Desktop** Pro або Max (без Free — потрібен доступ до Cowork).
- **Підключений Figma connector** у Claude Desktop → Customize → Connectors → Figma → OAuth.
- **Підключений Higgsfield connector** там же → Higgsfield → OAuth.
- **Доступ до Figma-проекту**, який буде референсом стилю (твій логін Figma має його бачити).
- **Баланс Higgsfield** > `2N` кредитів (приблизно 1 кредит за дудл + 1 за фото).

---

## Установка

В терміналі (PowerShell):

```powershell
claude plugin marketplace add HoshovskyiV/plusone-creative
claude plugin install plusone-creative@plusone-creative
```

Перевірка установки:

```powershell
claude plugin list
```

В списку має бути `plusone-creative`. Якщо нема — перевір що ти залогінений у GitHub через `gh auth status` і що в Claude Desktop увімкнений CLI (Customize → Use Claude Code).

---

## 5 готових промпт-шаблонів для Cowork

Скопіюй, заміни `[…]` на свої дані, встав у Cowork-чат.

### 1. Батч постів у стилі проекту

```
Зроби 10 інстаграм-постів у стилі проекту [Figma URL].
Бриф — у файлі [path/to/brief.md].
Формат 1080×1080. Мова українська.
```

### 2. Адаптація шаблону для нового клієнта

```
Адаптуй шаблон [Figma URL ноду шаблону] під клієнта [назва].
Контент-план у [path/to/content.csv].
Збережи готові кадри на окремій сторінці.
```

### 3. Карусель story (9:16)

```
Згенеруй карусель з 5 сторіс у стилі [Figma URL].
Тема: [одне речення].
Формат 1080×1920.
Кожна сторіс має заголовок, підзаголовок, дудл і фото.
```

### 4. Ребренд існуючої серії

```
Візьми пости з фрейму [Figma URL фрейму батчу] і перероби їх у стилі проекту [інший Figma URL].
Залиш тексти, поміняй дудли + фото + палітру під новий стиль.
```

### 5. Швидкий single-post

```
Один інстаграм-пост у стилі [Figma URL].
Хедлайн: "[текст]"
Підзаголовок: "[текст]"
Концепція дудла: [одне речення]
Концепція фото: [одне речення]
```

---

## Що відбувається під капотом

Workflow завжди один і той же (10 кроків — детально у `skills/plusone-creative/SKILL.md`):

```
parse brief
  → identify Figma URL
    → style-analyzer (агент, читає Figma)
      → parse content brief
        → побудувати матрицю N×4
          → asset-generator (агент, паралельні Higgsfield calls)
            → frame-composer (агент, пише у Figma)
              → validator (агент, перевіряє кадри)
                → retry loop (якщо є проблеми)
                  → звіт користувачу
```

Якщо щось ламається на якомусь кроці — Claude скаже на якому і що робити.

---

## Структура плагіна

```
plusone-creative/
├── .claude-plugin/
│   ├── plugin.json           # маніфест плагіна
│   └── marketplace.json      # маніфест marketplace (одноплагінного)
├── .mcp.json                 # remote MCP servers (Figma + Higgsfield)
├── skills/
│   ├── plusone-creative/     # головний skill з 10-крокового workflow
│   │   ├── SKILL.md
│   │   └── references/       # 4 reference docs (style, doodle, prompts, conventions)
│   ├── figma-use/            # офіційний skill для use_figma write-операцій
│   └── figma-generate-design/# офіційний skill для design generation
├── agents/                   # 4 sub-agents
│   ├── style-analyzer.md
│   ├── asset-generator.md
│   ├── frame-composer.md
│   └── validator.md
├── README.md                 # цей файл
├── HANDOFF.md                # покрокова інструкція для дизайнера
└── CHANGELOG.md
```

---

## Зворотний зв'язок і апдейти

Якщо щось не працює або хочеться змінити поведінку — кажи (Telegram або голосом, як завжди), Vasyl ітерує. Логі помилок з Claude Desktop теж шли.

Перший пілот — це v0.1.0. Після фідбеку — v0.2.0 з усіма фіксами.

---

## Ліцензія

MIT. Внутрішній інструмент Plusone.
