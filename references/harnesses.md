# Клиенты Workmesh

Проверяй механизмы на реально доступных инструментах и версии клиента. Используй поддерживаемый механизм ведущей сессии для вопросов. Для изоляции проверяй, что делегация начинает новый контекст или что отдельная сессия открывается с новым ID и доступным способом получить результат.

| Клиент | Форма | Нативное исполнение | Отдельная именованная сессия |
| --- | --- | --- | --- |
| Codex | `request_user_input_async` или `request_user_input` там, где инструмент доступен. В App Server форма обозначена событием `item/tool/requestUserInput`. | Обычный вызов субагента. Если есть настройка наследования истории, укажи режим без истории. | Разрешённый механизм клиента. Для подключённого App Server используй `thread/start` в том же проекте, `thread/name/set`, `turn/start` и чтение результата по ID. |
| OpenCode | `question` при доступном разрешении и поддержке формы интерфейсом. | `subagent` в V2 с дочерним `sessionID`. В V1 — `task` с `task_id`. | V2 TUI `/new` и переименование. Подключённый V2 API создаёт самостоятельную сессию через `POST /api/session`, передаёт задачу через `POST /api/session/{sessionID}/prompt` и читает события или сообщения. |
| Hermes Agent | `clarify` в ведущей сессии, если инструмент предоставлен. Например, формы исключены из пресетов ACP и API Server и недоступны делегированным воркерам. | `delegate_task` получает задание и контекст явно. Свежий запуск использует новый subagent ID. | В отдельном клиенте `hermes -c <name> --create-if-missing` с уникальным именем попытки. Если имя уже занято, сделай новое или возобнови именно ранее записанную попытку по `hermes --resume <id-or-title>`. |
| Pi | `question`, когда зарегистрировано проверенное расширение. В штатной сборке используется вопрос в чате. `ctx.ui.select`, `ctx.ui.input` и `ctx.ui.confirm` принадлежат API расширений. | Настроенное расширение, например upstream-пример `subagent`. Проверь его фактическую изоляцию и каналы возврата. | Отдельный экземпляр Pi в той же рабочей папке, новая сессия и `/name <name>`. Для интеграции через SDK возможен `SessionManager.create(cwd)`. Имя сессии задаёт `pi.setSessionName(name)` в расширении. |
| Claude Code | `AskUserQuestion` в ведущей сессии с подходящими разрешениями. У субагентов инструмент исключён. | `Agent` с указанной персоной и полным брифом. Для обязательного ревью выбирай новый запуск с чистым контекстом. | `claude --name <name> <brief>` в проекте при поддержке установленной версией. Для возобновления собственной попытки используй `claude --resume <session-id-or-name>`. |

При отсутствии формы задай вопрос в ведущем чате. Для отдельного исполнения передай бриф, разрешённые входы и формат результата, запиши ID сессии до сбора ответа. Вызов из отдельного клиента должен сохранять действующие разрешения и контекст проекта. Команды и API применяй только там, где такой способ доступен и разрешён. В Codex Desktop создание пользовательского чата должно быть явно запрошено. В активном родительском Hermes CLI команда `/new` переключает чат и может остановить фоновые задачи. В Pi `/new` запускай в отдельном клиенте при работающем родителе. Для ревью используй новый контекст с минимальным пакетом входов. Возобновляй сессию по ID только для продолжения той же попытки.

### Запуск и сбор результата в Pi

При разрешённом вызове CLI запускай процесс из рабочей папки проекта. Передай уникальный ID попытки и понятное отображаемое имя. CLI открывает старую сессию при повторе ID, поэтому для новой проверки заранее выдели новый ID. Сохрани финальный ответ из stdout и статус процесса, затем сверь возврат с заданием и версией артефакта.

```text
pi --session-id <unique-id> --name <display-name> --print <brief>
```

Для ручного варианта в отдельном Pi доступны `/name`, `/session` для идентификатора и `/copy` для последнего ответа. [Pi CLI](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/cli.md#sessions) · [Команды Pi](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/usage.md#continue-or-start-over)

## Документация

- Codex. [Субагенты](https://learn.chatgpt.com/docs/agent-configuration/subagents), [вопросы](https://learn.chatgpt.com/docs/app-server#toolrequestuserinput), [сессии App Server](https://learn.chatgpt.com/docs/app-server#threads).
- OpenCode. [Инструменты V2](https://opencode.ai/docs/tools/), [инструменты V1](https://opencode.ai/v1/docs/tools/), [TUI](https://opencode.ai/docs/cli/tui/#sessions), [API V2](https://opencode.ai/docs/api).
- Hermes Agent. [Clarify](https://hermes-agent.nousresearch.com/docs/reference/tools-reference#clarify-toolset), [наборы инструментов](https://hermes-agent.nousresearch.com/docs/reference/toolsets-reference), [делегация](https://hermes-agent.nousresearch.com/docs/user-guide/features/delegation), [CLI](https://hermes-agent.nousresearch.com/docs/user-guide/cli).
- Pi. [CLI и сессии](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/cli.md#sessions), [API расширений](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/extensions.md), [пример question](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/examples/extensions/question.ts), [пример subagent](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/examples/extensions/subagent/index.ts).
- Claude Code. [AskUserQuestion](https://code.claude.com/docs/en/tools-reference#askuserquestion-tool-behavior), [субагенты](https://code.claude.com/docs/en/sub-agents#available-tools), [CLI](https://code.claude.com/docs/en/cli-reference#cli-flags).
