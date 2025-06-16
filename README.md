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

### 🛡 Безопасность

Никогда не коммитьте API-ключ в репозиторий. Лучше сохранять его в `.env` или `appsettings.json`.

