---
title: Лаба 2
tags:
  - layout
  - http
  - json
  - async
---

## Вступление

Данная лабораторная работа посвящена созданию базового каркаса приложения и первому подключению к реальному API.

**Что будем делать:**

- Вёрстка базового layout приложения
- Написание HTTP-запроса к API
- Десериализация JSON
- Вывод данных в ListBox

После этой лабы на экране появятся *реальные* данные с сервера. Пусть выглядят убого, но это "Мстители", а не "Item 1".

---

## Техническое задание

[Требования к программе, необходимые для сдачи.](../tasks/task2.md)

---

## Git-воркфлоу

Перед началом работы создай ветку:

```bash
git checkout -b feature/lab-2-skeleton
```

После завершения — Pull Request в `main`.

---

## Шаг 1: Вёрстка базового окна

Создаём простой layout: слева — список фильмов, справа — место для деталей (пока пустое).

```xml
<Window x:Class="CinemaKiosk.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="CinemaKiosk" Height="600" Width="800">
    <Grid>
        <Grid.ColumnDefinitions>
            <ColumnDefinition Width="300"/>
            <ColumnDefinition Width="*"/>
        </Grid.ColumnDefinitions>
        
        <!-- Левая панель: список фильмов -->
        <ListBox x:Name="MoviesListBox" Grid.Column="0"/>
        
        <!-- Правая панель: детали (пока пусто) -->
        <Border Grid.Column="1" Background="#F5F5F5">
            <TextBlock Text="Выберите фильм" 
                       HorizontalAlignment="Center" 
                       VerticalAlignment="Center"/>
        </Border>
    </Grid>
</Window>
```

---

## Шаг 2: Создание модели данных

Создай папку `Models` и добавь класс `MovieDto.cs`:

```csharp
public class MovieDto
{
    public int Id { get; set; }
    public string Title { get; set; }
    public string Description { get; set; }
    public string PosterUrl { get; set; }
    public int Duration { get; set; }
    
    public override string ToString()
    {
        return Title; // Для отображения в ListBox
    }
}
```

!!! note "Про DTO"
    DTO (Data Transfer Object) — это класс для передачи данных. Поля должны соответствовать JSON от API.

---

## Шаг 3: Установка NuGet-пакета

Для работы с JSON установи пакет `Newtonsoft.Json`:

1. **Tools** → **NuGet Package Manager** → **Manage NuGet Packages for Solution**
2. Найди `Newtonsoft.Json`
3. Установи

Или через Package Manager Console:

```powershell
Install-Package Newtonsoft.Json
```

---

## Шаг 4: Создание HTTP-клиента

Создай папку `Services` и добавь класс `ApiService.cs`:

```csharp
using System.Net.Http;
using System.Threading.Tasks;
using Newtonsoft.Json;
using System.Collections.Generic;

public class ApiService
{
    private readonly HttpClient _client;
    private const string BaseUrl = "https://your-api-url.com/api";
    
    public ApiService()
    {
        _client = new HttpClient();
    }
    
    public async Task<List<MovieDto>> GetMoviesAsync()
    {
        var response = await _client.GetStringAsync($"{BaseUrl}/movies");
        var movies = JsonConvert.DeserializeObject<List<MovieDto>>(response);
        return movies;
    }
}
```

---

## Шаг 5: Загрузка данных при старте

В `MainWindow.xaml.cs` добавь метод загрузки:

```csharp
public partial class MainWindow : Window
{
    private readonly ApiService _apiService;
    
    public MainWindow()
    {
        InitializeComponent();
        _apiService = new ApiService();
        
        // Загружаем данные при старте
        Loaded += MainWindow_Loaded;
    }
    
    private async void MainWindow_Loaded(object sender, RoutedEventArgs e)
    {
        await LoadMovies();
    }
    
    private async Task LoadMovies()
    {
        try
        {
            var movies = await _apiService.GetMoviesAsync();
            MoviesListBox.ItemsSource = movies;
        }
        catch (Exception ex)
        {
            MessageBox.Show($"Ошибка загрузки: {ex.Message}");
        }
    }
}
```

---

## Шаг 6: Проверка результата

Запусти приложение. В ListBox должны появиться названия фильмов с сервера.

Если видишь данные — поздравляю, ты подключился к реальному API!

---

## Возможные проблемы

### Ошибка сети
```
System.Net.Http.HttpRequestException
```
**Решение:** Проверь URL API и подключение к интернету.

### Ошибка десериализации
```
Newtonsoft.Json.JsonSerializationException
```
**Решение:** Проверь, что поля в `MovieDto` соответствуют JSON от API.

---

## Коммит и Push

```bash
git add .
git commit -m "feat: add basic layout and HTTP client"
git push origin feature/lab-2-skeleton
```

Создай Pull Request на GitHub!

---

## Полезные ссылки

- [HttpClient Documentation](https://docs.microsoft.com/en-us/dotnet/api/system.net.http.httpclient)
- [Newtonsoft.Json](https://www.newtonsoft.com/json)
- [Async/Await в C#](../topics/asyncawait.md)

