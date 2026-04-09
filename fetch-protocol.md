# Протокол fetch-and-log (актуальный)

Обязателен для Частей 2–9. Вступил в силу с коммита `734ea74` (10.04.2026).

## Шаг 1 — перед началом каждой Части

```
python notary_agent.py init-part-draft <subtopic_id> <N>
```

Создаёт `draft-part-NN.md` с `[WEBFETCH-ДЕКЛАРАЦИЯ]` в первой строке.
Дописывай карточки в этот файл через Write/Edit tool.
Не создавай файл вручную — маркер должен быть первым.

## Шаг 2 — после каждого web_fetch, перед каждой карточкой

```
python notary_agent.py fetch-and-log <subtopic_id> <url> --title "заголовок страницы" --preview "первые 200 символов"
```

- `--title` и `--preview` берёшь из результата своего встроенного `web_fetch`.
- Локальный HTTP-запрос не делается — WinError 10013 исключён.
- Запись пишется в `research-log.jsonl` с `fetched_by_agent=true`.
- Старый вызов без `--title` сохранён как fallback, но использовать его не нужно.

## Шаг 3 — после завершения Части

```
python notary_agent.py promote-draft <subtopic_id> <N>
```

- Если `[WEBFETCH-ПОДТВЕРЖДЕНИЕ]` отсутствует — дописывается автоматически.
- Если `[WEBFETCH-ДЕКЛАРАЦИЯ]` отсутствует — hard error. Файл создан в обход `init-part-draft`. Исправь вручную или пересоздай через `init-part-draft`.

## Полный цикл одной Части

```
python notary_agent.py init-part-draft <id> <N>

# для каждого документа:
# 1. web_fetch → получить заголовок и превью
# 2. python notary_agent.py fetch-and-log <id> <url> --title "..." --preview "..."
# 3. написать карточку в draft-part-NN.md

python notary_agent.py promote-draft <id> <N>
```

## Почему изменился протокол

Предыдущий `fetch-and-log` делал локальный HTTP-запрос через `urllib`.
На Windows он падал с WinError 10013 (сетевой блок), `fetched_by_agent=true` не писался,
trusted-валидация блокировала захват.
Новый режим `--title --preview` использует данные от встроенного `web_fetch` агента,
который работает всегда.
