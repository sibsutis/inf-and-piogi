---
title: Лекция 3 - Асинхронное программирование
doc-status:
  - in_work
doc-type:
  - lection
---

# Асинхронное программирование

## Введение: Почему интерфейс "зависает"

Вы наверняка видели такое: нажимаете кнопку в приложении, и оно "замирает" — окно белеет, ничего не реагирует, появляется надпись "Не отвечает". Через несколько секунд всё оживает.

Что произошло? Приложение выполняло долгую операцию (загрузка файла, запрос к серверу) **в главном потоке**, который отвечает за отрисовку интерфейса.

**Асинхронное программирование** решает эту проблему: долгие операции выполняются "в фоне", а интерфейс остаётся отзывчивым.

---

## Однопоточная модель UI

### Как работает Windows-приложение

Каждое приложение с графическим интерфейсом имеет **главный поток (UI Thread)**, который:

1. Обрабатывает события (клики, нажатия клавиш)
2. Перерисовывает окно (60 раз в секунду)
3. Обновляет элементы интерфейса

```
┌────────────────────────────────────────┐
│           Message Queue                 │
│  [Click] [KeyPress] [Paint] [Timer]... │
└──────────────────┬─────────────────────┘
                   │
                   ▼
┌────────────────────────────────────────┐
│              UI Thread                  │
│  while (GetMessage(&msg)) {             │
│      TranslateMessage(&msg);            │
│      DispatchMessage(&msg);             │
│  }                                      │
└────────────────────────────────────────┘
```

### Проблема синхронного кода

Если в обработчике кнопки выполняется долгая операция:

```csharp
private void Button_Click(object sender, RoutedEventArgs e)
{
    // Этот код блокирует UI на 5 секунд!
    Thread.Sleep(5000);
    
    // Или загрузка данных синхронно
    var data = httpClient.GetStringAsync(url).Result; // ПЛОХО!
}
```

==Гифка живого примеры==

Пока `Button_Click` не завершится, UI Thread не сможет:
- Перерисовать окно
- Обработать другие клики
- Анимировать элементы

Результат: "замёрзшее" приложение "не отвечает", сносим через диспетчер задач.

### Демонстрация

```csharp
// ПЛОХОЙ КОД - UI зависнет
private void LoadButton_Click(object sender, RoutedEventArgs e)
{
    StatusLabel.Content = "Загрузка...";
    
    // Имитация долгой операции
    // Типа получаем данные, но сервер отвечает 30 секунд
    Thread.Sleep(30000);
    
    StatusLabel.Content = "Готово!";
}
```

Парадокс: даже `StatusLabel.Content = "Загрузка..."` не появится на экране, потому что перерисовка произойдёт только после завершения обработчика!

---

## Эволюция асинхронного кода

### Эпоха 1: Callbacks (Обратные вызовы)

Старейший способ асинхронности — передать функцию, которая вызовется по завершении:

```csharp
// Callback hell — ад обратных вызовов
webClient.DownloadStringAsync(url1, (result1) => {
    webClient.DownloadStringAsync(url2, (result2) => {
        webClient.DownloadStringAsync(url3, (result3) => {
            // И так далее...
            // Код уходит вправо, сложно читать и отлаживать
        });
    });
});
```

**Проблемы:**
- Код "уезжает" вправо (пирамида смерти)
- Сложная обработка ошибок
- Трудно следить за последовательностью

### Эпоха 2: Events (События)

Альтернатива — использовать события (привет от делегатов):

```csharp
webClient.DownloadStringCompleted += (sender, e) => {
    var data = e.Result;
    // Обработка результата
};
webClient.DownloadStringAsync(url);
```

**Проблемы:**
- Код разбросан (запуск в одном месте, обработка в другом)
- Сложно компоновать несколько операций

### Эпоха 3: Task и Promises

==Разжевать про такси подробно==

.NET 4.0 представил `Task` — объект, представляющий будущий результат:

```csharp
Task<string> task = httpClient.GetStringAsync(url);

// Можно продолжить цепочкой
task.ContinueWith(t => {
    var result = t.Result;
    // Использовать результат
});
```

Лучше, но всё ещё громоздко.

### Эпоха 4: async/await (C# 5.0)

**async/await** — синтаксический сахар, который позволяет писать асинхронный код как синхронный:

```csharp
private async void LoadButton_Click(object sender, RoutedEventArgs e)
{
    StatusLabel.Content = "Загрузка...";
    
    // Не блокирует UI Thread!
    var data = await httpClient.GetStringAsync(url);
    
    StatusLabel.Content = "Готово!";
}
```

**Магия:** код выглядит последовательным, но UI не зависает!

---

## Как работает async/await

### Ключевые слова

**`async`** — модификатор метода, разрешающий использовать `await` внутри.

**`await`** — оператор, который:
1. Проверяет, завершена ли операция
2. Если нет — "приостанавливает" метод и возвращает управление
3. Когда операция завершится — продолжает выполнение с места остановки

### Что происходит под капотом

```csharp
async Task<string> LoadDataAsync()
{
    Console.WriteLine("1: Начало");
    
    var result = await httpClient.GetStringAsync(url);
    // ^-- Здесь метод "засыпает"
    
    Console.WriteLine("2: Продолжение"); // Выполнится позже
    return result;
}
```

Компилятор превращает это в **конечный автомат (state machine)**:

```
┌────────────────────────────────────────────────────────┐
│  State 0: Начало                                        │
│    ├── Console.WriteLine("1: Начало")                  │
│    ├── Запустить GetStringAsync                         │
│    └── Вернуть Task (метод "приостановлен")            │
└────────────────────────────────────────────────────────┘
                          │
                          ▼ (когда HTTP завершится)
┌────────────────────────────────────────────────────────┐
│  State 1: Продолжение                                   │
│    ├── Console.WriteLine("2: Продолжение")             │
│    └── Вернуть результат                                │
└────────────────────────────────────────────────────────┘
```

### Важно: await не создаёт новый поток

`await` **не создаёт новый поток**. Он освобождает текущий поток для других задач. Когда асинхронная операция завершается, продолжение выполняется (обычно) в том же контексте.

---

## Task vs Thread

### Thread (Поток)

==Показать пример анимации на потоке как плохой пример==

**Thread** — это поток выполнения операционной системы. Создание потока дорого (выделение стека, переключение контекста).

```csharp
// Создание нового потока — тяжёлая операция
var thread = new Thread(() => {
    // Работа в отдельном потоке
});
thread.Start();
```

### Task (Задача)

**Task** — абстракция над асинхронной операцией. Может использовать пул потоков, а может не использовать потоки вообще (например, ожидание I/O).

```csharp
// Task может не создавать новый поток
Task<string> task = httpClient.GetStringAsync(url);
// Пока ждём ответ сервера — потоков не занято
```

### I/O-bound vs CPU-bound

**I/O-bound** (ограниченные вводом-выводом):
- Сетевые запросы
- Чтение/запись файлов
- Запросы к базе данных

Для них `async/await` идеален — **не нужен отдельный поток**, просто ожидание.

**CPU-bound** (ограниченные процессором):
- Сложные вычисления
- Обработка изображений
- Сжатие данных

Для них нужен **отдельный поток**, чтобы не блокировать UI:

```csharp
// CPU-bound: используем Task.Run для переноса в другой поток
var result = await Task.Run(() => {
    // Тяжёлые вычисления
    return CalculateSomethingComplex();
});
```

---

## Паттерны асинхронности

### 1. Последовательное выполнение

```csharp
async Task LoadAllDataAsync()
{
    var movies = await GetMoviesAsync();      // Ждём
    var sessions = await GetSessionsAsync();  // Потом ждём
    var seats = await GetSeatsAsync();        // Потом ждём
    // Общее время = сумма времён всех операций
}
```

### 2. Параллельное выполнение (Task.WhenAll)

```csharp
async Task LoadAllDataFastAsync()
{
    // Запускаем все операции одновременно
    var moviesTask = GetMoviesAsync();
    var sessionsTask = GetSessionsAsync();
    var seatsTask = GetSeatsAsync();
    
    // Ждём завершения всех
    await Task.WhenAll(moviesTask, sessionsTask, seatsTask);
    
    // Получаем результаты
    var movies = moviesTask.Result;
    var sessions = sessionsTask.Result;
    var seats = seatsTask.Result;
    // Общее время = время самой долгой операции
}
```

### 3. Гонка (Task.WhenAny)



```csharp
async Task<string> GetFromFastestServerAsync()
{
    var task1 = httpClient.GetStringAsync("https://server1.com/data");
    var task2 = httpClient.GetStringAsync("https://server2.com/data");
    
    // Возвращает первый завершившийся
    // Остальные вернут ответ в пустоту, круто да? 
    // Как спросить домашка у всех в потоке, ответ получить в личку, а потом за игнорить ещё 10 ответов
    var firstCompleted = await Task.WhenAny(task1, task2);
    return await firstCompleted;
}
```

### 4. Fire-and-Forget (Запустить и забыть)

```csharp
// Осторожно! Ошибки могут потеряться
private void Button_Click(object sender, RoutedEventArgs e)
{
    // Не ждём завершения
    _ = SendAnalyticsAsync();  // _ показывает, что результат не нужен
}
```

### 5. Cancellation Token (Отмена)

```csharp
private CancellationTokenSource _cts;

private async void SearchButton_Click(object sender, RoutedEventArgs e)
{
    // Отменяем предыдущий поиск
    _cts?.Cancel();
    _cts = new CancellationTokenSource();
    
    try
    {
        var results = await SearchAsync(query, _cts.Token);
        DisplayResults(results);
    }
    catch (OperationCanceledException)
    {
        // Поиск был отменён — это нормально
    }
}

async Task<List<Movie>> SearchAsync(string query, CancellationToken token)
{
    // Проверяем, не отменили ли нас
    token.ThrowIfCancellationRequested();
    
    var response = await httpClient.GetAsync(url, token);
    // ...
}
```

---

## Типичные ошибки

### 1. async void

```csharp
// ПЛОХО: async void - нельзя отловить исключения
private async void LoadData()
{
    var data = await GetDataAsync(); // Если упадёт — приложение крашнется
}

// ХОРОШО: async Task
private async Task LoadDataAsync()
{
    var data = await GetDataAsync(); // Исключение можно обработать
}
```

**Исключение:** обработчики событий WPF обязаны быть `async void`:

```csharp
private async void Button_Click(object sender, RoutedEventArgs e)
{
    try
    {
        await LoadDataAsync();
    }
    catch (Exception ex)
    {
        MessageBox.Show($"Ошибка: {ex.Message}");
    }
}
```

### 2. .Result и .Wait() — Deadlock

```csharp
// ОЧЕНЬ ПЛОХО: может вызвать deadlock
private void Button_Click(object sender, RoutedEventArgs e)
{
    var result = GetDataAsync().Result;  // Deadlock!
    var result2 = GetDataAsync().Wait(); // Deadlock!
}
```

**Почему deadlock?**
1. `.Result` блокирует UI Thread
2. `await` внутри `GetDataAsync` хочет продолжить выполнение в UI Thread
3. Но UI Thread заблокирован `.Result`
4. Бесконечное ожидание

### 3. Забытый await

```csharp
// ПЛОХО: операция запустится, но результат не дождёмся
private async Task LoadAsync()
{
    GetDataAsync();  // Забыли await!
    // Код пойдёт дальше, не дожидаясь данных
    // Классическое "все правильно, но данных нет"
}

// ХОРОШО
private async Task LoadAsync()
{
    await GetDataAsync();
}
```

### 4. ConfigureAwait

В библиотечном коде рекомендуется использовать `ConfigureAwait(false)`:

```csharp
// В библиотеке — не нужно возвращаться в UI Thread
public async Task<string> GetDataAsync()
{
    var response = await httpClient.GetAsync(url).ConfigureAwait(false);
    var content = await response.Content.ReadAsStringAsync().ConfigureAwait(false);
    return content;
}
```

В приложении WPF — **не** используйте, если обновляете UI:

```csharp
// В WPF приложении — нужен UI Thread для обновления элементов
private async void Button_Click(object sender, RoutedEventArgs e)
{
    var data = await GetDataAsync();  // Без ConfigureAwait
    DataLabel.Content = data;  // Обновление UI — нужен UI Thread
}
```

---

## Async в WPF

### Загрузка данных при старте

```csharp
public partial class MainWindow : Window
{
    public MainWindow()
    {
        InitializeComponent();
        Loaded += MainWindow_Loaded; //Событие полной загрузки окна, назначаем обработчик
    }
    
    private async void MainWindow_Loaded(object sender, RoutedEventArgs e)
    {
        try
        {
            LoadingIndicator.Visibility = Visibility.Visible;
            
            var movies = await _apiService.GetMoviesAsync();
            MoviesListBox.ItemsSource = movies;
        }
        catch (HttpRequestException ex)
        {
            MessageBox.Show($"Ошибка сети: {ex.Message}");
        }
        finally
        {
            LoadingIndicator.Visibility = Visibility.Collapsed;
        }
    }
}
```

### Прогресс-индикатор

```csharp
private async void DownloadButton_Click(object sender, RoutedEventArgs e)
{
    DownloadButton.IsEnabled = false;
    ProgressBar.Value = 0;
    
    var progress = new Progress<int>(percent => {
        ProgressBar.Value = percent;
    });
    
    await DownloadFileAsync(url, progress);
    
    DownloadButton.IsEnabled = true;
}

async Task DownloadFileAsync(string url, IProgress<int> progress)
{
    // Отчёт о прогрессе
    for (int i = 0; i <= 100; i += 10)
    {
        await Task.Delay(100);
        progress.Report(i);
    }
}
```

---

## Связь с лабораторными работами

### Лаба 2: Первый async-запрос

```csharp
public async Task<List<MovieDto>> GetMoviesAsync()
{
    var response = await _client.GetStringAsync($"{BaseUrl}/movies");
    var movies = JsonConvert.DeserializeObject<List<MovieDto>>(response);
    return movies;
}
```

### Лаба 3: Загрузка изображений

WPF автоматически загружает изображения асинхронно по URL:

```xml
<Image Source="{Binding PosterUrl}"/>
```

Под капотом происходит асинхронная загрузка, UI не блокируется.

### Лаба 6: POST-запрос с обработкой ошибок

```csharp
public async Task<BookingResponse> CreateBookingAsync(BookingRequest request)
{
    var json = JsonConvert.SerializeObject(request);
    var content = new StringContent(json, Encoding.UTF8, "application/json");
    
    var response = await _client.PostAsync($"{BaseUrl}/booking", content);
    var responseJson = await response.Content.ReadAsStringAsync();
    
    // Обработка разных статусов
    if (response.IsSuccessStatusCode)
    {
        return JsonConvert.DeserializeObject<BookingResponse>(responseJson);
    }
    
    throw new ApiException(response.StatusCode, responseJson);
}
```

---

## Заключение

**Async/await** — это не просто синтаксис, это **образ мышления**:

- UI Thread священен — не блокируй его
- I/O операции должны быть асинхронными
- Код выглядит синхронным, но работает асинхронно
- Обрабатывай ошибки и отмену

Навык написания асинхронного кода **критически важен** для современного разработчика. Без него невозможно создать отзывчивое приложение, работающее с сетью или файлами.

---

## Полезные ресурсы

- [Async/Await — Best Practices](https://docs.microsoft.com/en-us/archive/msdn-magazine/2013/march/async-await-best-practices-in-asynchronous-programming)
- [Stephen Cleary: Async in C#](https://blog.stephencleary.com/)
- [Task-based Asynchronous Pattern (TAP)](https://docs.microsoft.com/en-us/dotnet/standard/asynchronous-programming-patterns/task-based-asynchronous-pattern-tap)
