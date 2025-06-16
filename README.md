### 💡 Что понадобится:

1. **API-ключ OpenAI**.
2. **Установить NuGet-пакет `RestSharp` или `HttpClient`**.
3. **Windows Forms App (.NET Framework или .NET Core)**.

---

### 📦 Шаг 1: Установка зависимостей

Если используете .NET Core / .NET 6+:

```bash
dotnet add package System.Text.Json
```

Или используйте `RestSharp`:

```bash
dotnet add package RestSharp
```

---

### 📝 Шаг 2: Код программы

```csharp
using System;
using System.IO;
using System.Net.Http;
using System.Text;
using System.Text.Json;
using System.Threading.Tasks;
using System.Windows.Forms;

namespace ChatGPTMarkdownSaver
{
    public partial class MainForm : Form
    {
        private readonly HttpClient _httpClient = new HttpClient();
        private const string OpenAIApiKey = "YOUR_API_KEY"; // 🔐 Замените на ваш API ключ

        public MainForm()
        {
            InitializeComponent();
            this.Text = "ChatGPT to Markdown";
            this.Width = 800;
            this.Height = 600;

            var inputBox = new TextBox
            {
                Multiline = true,
                Width = 760,
                Height = 300,
                Top = 10,
                Left = 10,
                ScrollBars = ScrollBars.Vertical
            };

            var sendButton = new Button
            {
                Text = "Отправить и сохранить",
                Width = 200,
                Top = 320,
                Left = 10
            };

            sendButton.Click += async (sender, args) =>
            {
                string prompt = inputBox.Text;
                string response = await GetChatGPTResponse(prompt);

                // Сохраняем в Markdown
                string filePath = $"chatgpt_{DateTime.Now:yyyyMMdd_HHmmss}.md";
                File.WriteAllText(filePath, $"# Вопрос\n\n{prompt}\n\n# Ответ\n\n{response}");

                MessageBox.Show($"Ответ сохранён в файл: {filePath}");
            };

            this.Controls.Add(inputBox);
            this.Controls.Add(sendButton);
        }

        private async Task<string> GetChatGPTResponse(string prompt)
        {
            var requestBody = new
            {
                model = "gpt-4",
                messages = new[]
                {
                    new { role = "user", content = prompt }
                }
            };

            var requestContent = new StringContent(
                JsonSerializer.Serialize(requestBody),
                Encoding.UTF8,
                "application/json"
            );

            _httpClient.DefaultRequestHeaders.Clear();
            _httpClient.DefaultRequestHeaders.Add("Authorization", $"Bearer {OpenAIApiKey}");

            var response = await _httpClient.PostAsync("https://api.openai.com/v1/chat/completions", requestContent);
            var responseString = await response.Content.ReadAsStringAsync();

            using JsonDocument doc = JsonDocument.Parse(responseString);
            var content = doc.RootElement
                .GetProperty("choices")[0]
                .GetProperty("message")
                .GetProperty("content")
                .GetString();

            return content;
        }
    }
}
```

---

### ⚙️ Шаг 3: Настройка формы

Если используете Windows Forms Designer, добавьте `TextBox` и `Button` и назовите их соответственно (`inputBox`, `sendButton`). Или используйте код выше — он программно создаёт элементы.

---

Отлично! Давайте улучшим программу:

---

## ✅ Обновления:

1. **Форматирование Markdown**:

   * Вопрос будет в `## Вопрос`
   * Ответ в `## Ответ`
   * Все блоки кода будут автоматически обернуты в тройные кавычки ` ``` `.

2. **Поддержка нескольких ответов**:

   * Количество ответов можно задать, например, 3.

---

### 🆕 Обновлённый метод `GetChatGPTResponse` и `Click` обработчик:

Замените в предыдущем коде эти два фрагмента:

---

### 📌 Новый `Click` обработчик кнопки:

```csharp
sendButton.Click += async (sender, args) =>
{
    string prompt = inputBox.Text;
    int numberOfResponses = 3;

    string[] responses = await GetMultipleChatGPTResponses(prompt, numberOfResponses);

    var sb = new StringBuilder();
    sb.AppendLine("## Вопрос\n");
    sb.AppendLine(prompt);
    sb.AppendLine();

    for (int i = 0; i < responses.Length; i++)
    {
        sb.AppendLine($"## Ответ {i + 1}\n");
        sb.AppendLine(FormatMarkdown(responses[i]));
        sb.AppendLine();
    }

    string filePath = $"chatgpt_{DateTime.Now:yyyyMMdd_HHmmss}.md";
    File.WriteAllText(filePath, sb.ToString());

    MessageBox.Show($"Ответы сохранены в файл: {filePath}");
};
```

---

### 📌 Метод `GetMultipleChatGPTResponses`:

```csharp
private async Task<string[]> GetMultipleChatGPTResponses(string prompt, int count)
{
    var responses = new string[count];

    for (int i = 0; i < count; i++)
    {
        responses[i] = await GetChatGPTResponse(prompt);
        await Task.Delay(500); // Маленькая пауза между запросами
    }

    return responses;
}
```

---

### 📌 Метод `FormatMarkdown`:

Добавьте этот метод в класс, чтобы обернуть код в markdown-блоки:

````csharp
private string FormatMarkdown(string content)
{
    var lines = content.Split('\n');
    var sb = new StringBuilder();
    bool inCodeBlock = false;

    foreach (var line in lines)
    {
        if (line.TrimStart().StartsWith("```"))
        {
            inCodeBlock = !inCodeBlock;
            sb.AppendLine(line);
        }
        else if (line.TrimStart().StartsWith("    ") || line.Trim().StartsWith("//") || line.Trim().Contains(";"))
        {
            if (!inCodeBlock)
            {
                sb.AppendLine("```csharp");
                inCodeBlock = true;
            }
            sb.AppendLine(line);
        }
        else
        {
            if (inCodeBlock)
            {
                sb.AppendLine("```");
                inCodeBlock = false;
            }
            sb.AppendLine(line);
        }
    }

    if (inCodeBlock)
    {
        sb.AppendLine("```");
    }

    return sb.ToString();
}
````

---

📄 **Итог**:

* У вас теперь будет файл `.md`, в котором:

  * Каждый ответ аккуратно размечен как `## Ответ X`.
  * Код автоматически выделяется в блоках ` ```csharp `.
  * Можно легко сравнить несколько вариантов ответа ChatGPT.

---

Если хотите добавить возможность выбрать количество ответов прямо в интерфейсе (например, через `NumericUpDown`) — могу добавить это тоже.


В OpenAI доступны несколько моделей, каждая с разными возможностями, скоростью и стоимостью. Вот актуальный список моделей **ChatGPT (по состоянию на июнь 2025)**:

---

## 🧠 **Основные модели ChatGPT**

| Модель          | Описание                                                  | Поддержка кода и мультимодальности |
| --------------- | --------------------------------------------------------- | ---------------------------------- |
| **gpt-4o**      | ⚡ Новейшая, быстрая, мультимодальная, дешевле GPT-4-turbo | ✅ (код, текст, изображение, аудио) |
| **gpt-4-turbo** | Мощная и быстрая версия GPT-4 (предыдущий флагман)        | ✅ (текст, код)                     |
| **gpt-4**       | Оригинальная GPT-4 (медленнее и дороже)                   | ✅ (только текст и код)             |
| **gpt-3.5**     | Быстрая, недорогая, но слабее GPT-4                       | ✅ (текст и код)                    |

---

## 📸 **Мультимодальные возможности**

| Модель          | Текст | Код | Картинки | Аудио | Видео |
| --------------- | ----- | --- | -------- | ----- | ----- |
| **gpt-4o**      | ✅     | ✅   | ✅        | ✅     | ❌     |
| **gpt-4-turbo** | ✅     | ✅   | ✅        | ❌     | ❌     |
| **gpt-3.5**     | ✅     | ✅   | ❌        | ❌     | ❌     |

---

## 💰 **Разница по стоимости и производительности**

| Модель          | Цена (примерная, API) | Скорость   | Контекст        |
| --------------- | --------------------- | ---------- | --------------- |
| **gpt-4o**      | Очень дёшево, быстро  | 🚀 Быстрая | до 128k токенов |
| **gpt-4-turbo** | Средняя цена          | 🏃‍♂️      | до 128k токенов |
| **gpt-3.5**     | Очень дёшево          | ⚡          | до 16k токенов  |

---

## 🔧 Пример использования моделей в API

```json
{
  "model": "gpt-4o", // можно заменить на gpt-4-turbo, gpt-4, gpt-3.5-turbo
  "messages": [
    { "role": "user", "content": "Привет, как дела?" }
  ]
}
```

---

### 🔄 Советы:

* Используйте **`gpt-4o`** для наилучшего баланса между качеством, стоимостью и возможностями.
* **`gpt-3.5`** подходит для дешёвых и простых задач.
* Старую **`gpt-4`** лучше не использовать — она дороже и хуже `gpt-4o`.

---

Хочешь пример кода на C# с выбором модели?


### 🛡 Безопасность

Никогда не коммитьте API-ключ в репозиторий. Лучше сохранять его в `.env` или `appsettings.json`.

