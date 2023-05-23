# AI-based messenger suggestions - one-click answers (backend)
## О проекте
Веб-сервер для автоматических подсказок для ответа в чате в один клик на основе AI. Интегрируйте AI-подсказки в ваше приложение.
Используется в моем проекте VKGPT в качестве серверной части.

VK GPT - кастомный клиент мессенджера [социальной сети ВКонтакте](https://vk.com). Основная фича - автоматические подсказки на основе AI, которые позволяют пользователю отвечать на сообщения в один клик.
## Использование
Серверная часть проекта не использует какие-либо библиотеки, привязанные к AI-провайдерам (такими как openai или gpt4free). AI-агент написан с нуля [в файле ai.py](https://github.com/nightadmin/ai-messenger-suggestions/blob/master/ai.py) и реализует интерфейс ChatCompletions API OpenAI. Это позволяет использовать альтернативные реализации AI-провайдеров, которые используют ту же схему взаимодействия, что и OpenAI (например, free.churchless.tech - на момент написания README этот сервис перестал работать). Для замены API-провайдера достаточно передать параметр с endpoint вашего сервиса:
- https://api.openai.com/v1/chat/completions - API OpenAI
- https://free.churchless.tech/v1/chat/completions - альтернативная реализация
- другой endpoint для вашего сервиса

Если ваш API-провайдер требует авторизацию по токену (например, OpenAI), токен также можно передать в объявлении класса:
```python
TOKEN = open("token.txt", encoding="utf-8").read()
agent = AIAgent("https://api.openai.com/v1/chat/completions", TOKEN)
```
## Prompt
Prompt для генерации подсказок хранится в файле prompt.txt. AI-провайдер передает текст prompt как сообщение с role=system, а сообщения пользователей с role=user, что помогает защититься от атак типа prompt injection, **если модель AI (в проекте используется gpt-3.5-turbo) поддерживает это**. Например, некоторые старые модели OpenAI не гарантируют приоритизацию сообщениям с role=system, поэтому важно понимать, что эта защита - не панацея.
## Идея реализации
Backend-часть приложения не является моделью, генерирующей подсказки. Вместо этого AI-агент требует отвечать в заданном формате JSON (который валидируется и отправляется клиенту в случае отсутствия ошибок). Использование такого подхода достаточно надежно, с учетом защиты от prompt injection и валидации выходного JSON.

Данный репозиторий содержит реализацию простейшего веб-сервера на основе FastAPI, который взаимодействует с API AI-провайдера описанным способом и всегда возвращает корректный ответ.
## Схема запроса / ответа
Документация доступна на localhost:8000/docs.
### POST /predict

#### Request
Необходимо отправить POST-запрос с массивом объектов типа "сообщение". Описание объектов сообщения:

```python
class Message(BaseModel):
    from_name: str
    text: str
```

Пример запроса:
```json
[
   {
      "from_name": "User 1",
      "text": "Message 1 text"
   }
]
```

#### Response

Ответ сервера содержит поле ok и дополнительные поля, в зависимости от успеха.

При ok=true:
```json
{
   "ok": true,
   "suggestions": [
      "Подсказка1",
      "Подсказка2"
   ]
}
При ok=false может быть возвращено поле exception с текстовым описанием ошибки.
```json
{
   "ok": false,
   "exception":"internal error"
}
```

```json
{
   "ok": false
}
```

### Пример запроса

Пример запроса:
```bash
curl -X 'POST' \
  'http://localhost:8000/predict' \
  -H 'accept: application/json' \
  -H 'Content-Type: application/json' \
  -d '[
  {
    "from_name": "Mark",
    "text": "Привет! Как дела?"
  },
  {
    "from_name": "Ilya",
    "text": "Привет, отлично! Как сам?"
  }
]'
```

```json
{
  "ok": true,
  "suggestions": [
    "Прекрасно!",
    "Очень хорошо",
    "Все отлично!"
  ]
}
```

## Новости проекта
Следите за новостями:
- ВКонтакте: https://vk.com/vkgptapp
- Мой Telegram-канал (поиск по хештегу VKGPT): https://t.me/difhel_b