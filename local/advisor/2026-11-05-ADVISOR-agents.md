# Агенти Блоку 0

<!-- advisor:pair audit/advisor/2026-11-05-ADVISOR-agents.xml -->

- Status: partial
- Input: `local/notes/agents-block-0-update.txt`
- Updated: 2026-11-05

## Decision

Блок 0 тепер має шість ролей. Чотири аналітики пишуть попередні англійські документи — опис продукту, архітектуру, error handling і concurrency — кожен зі своїм скілом і правилом. Їх не можна викликати вручну: лише formalizer через `/formalizer` з одним xml або txt, після чого він перевіряє цілісність і конфлікти. Без input formalizer лише робить цю перевірку і зберігає рапорт xml та md у стилі advisor. translator лишається окремим і викликається як `/translator`. Наявні п’ять агентів і код продукту не змінюються.

## Next steps

1. Agent: додати formalizer, translator, чотирьох аналітиків, їхні скіли й правила. Готово.
2. Developer: запускати `/formalizer` з xml або txt, доки рапорт не містить `integrity: ok` і `conflicts: none`; розбіжності записати в `local/notes/block-0-resolutions.txt`.
3. Agent: підтвердити англійські файли і чистий рапорт.
4. Developer: запустити `/translator`.
5. Agent: підтвердити українські чернетки в `local/block-0/`.
6. Developer: покласти чотири українські файли на Google Drive і записати `local/notes/block-0-drive.txt`.
7. Agent: підтвердити запис про Drive.
8. Developer: схвалити Блок 0 рядком `template: approved` у `local/notes/block-0-template.txt`.
9. Agent: підтвердити схвалення шаблону.

## Agent log

Крок 1 виконано: додано шість ролей, чотири скіли й чотири правила. Крок 2 лишається заблокованим, крок 3 не почато.

Запустіть /formalizer з одним xml або txt файлом, а якщо рапорт містить конфлікт або рядок integrity: failed, спочатку запишіть local/notes/block-0-resolutions.txt.

## Developer steps
<!-- advisor:developer-steps -->

None.
