---
title: Лекция 6.1 - Навигация и архитектура приложения
---

# Навигация и архитектура приложения

## Введение

Приложение киоска — это не один экран. Это **путешествие** пользователя:

1. Выбор фильма
2. Выбор сеанса
3. Выбор мест
4. Оплата
5. Получение билета

Каждый шаг — отдельный экран. Как организовать переходы между ними?

---

## Паттерны навигации

### Stack-based (Стековая навигация)

Экраны складываются в **стек**. Новый экран — наверх. Кнопка "Назад" — убираем верхний.

```
        ┌─────────────┐
        │   Билет     │ ← Текущий экран
        ├─────────────┤
        │   Оплата    │
        ├─────────────┤
        │   Места     │
        ├─────────────┤
        │   Сеансы    │
        ├─────────────┤
        │   Фильмы    │ ← Начальный экран
        └─────────────┘
```

**Примеры:** Мобильные приложения, мастера настройки

### Tab-based (Вкладки)

Верхний уровень — вкладки. Каждая вкладка может иметь свой стек.

```
┌─────────────────────────────────────────┐
│ [Фильмы] [Мои билеты] [Профиль]         │
├─────────────────────────────────────────┤
│                                         │
│         Контент вкладки                 │
│                                         │
└─────────────────────────────────────────┘
```

**Примеры:** Банковские приложения, соцсети

### Drawer (Боковое меню)

Меню выезжает сбоку.

```
┌───┬─────────────────────────────────┐
│   │                                 │
│ M │      Контент                    │
│ E │                                 │
│ N │                                 │
│ U │                                 │
│   │                                 │
└───┴─────────────────────────────────┘
```

**Примеры:** Админ-панели, сложные приложения с множеством разделов

### Wizard (Мастер)

Пошаговый процесс с индикатором прогресса.

```
┌─────────────────────────────────────────┐
│  1. Фильм  →  2. Сеанс  →  3. Места  →  4. Оплата
│     ●           ●            ○              ○
├─────────────────────────────────────────┤
│                                         │
│         Текущий шаг                     │
│                                         │
│              [Далее]                    │
└─────────────────────────────────────────┘
```

**Примеры:** Процесс покупки, регистрация

---

## Навигация в WPF

### Frame и Page

**Frame** — контейнер для навигации. **Page** — страницы внутри.

```xml
<!-- MainWindow.xaml -->
<Window>
    <Frame x:Name="MainFrame" NavigationUIVisibility="Hidden"/>
</Window>
```

```csharp
// Переход на страницу
MainFrame.Navigate(new MoviesPage());

// С параметром
MainFrame.Navigate(new SessionsPage(movieId));
```

### Пример структуры страниц

```
CinemaKiosk/
├── Pages/
│   ├── MoviesPage.xaml       ← Список фильмов
│   ├── SessionsPage.xaml     ← Сеансы фильма
│   ├── HallPage.xaml         ← Карта зала
│   ├── PaymentPage.xaml      ← Оплата
│   └── TicketPage.xaml       ← Билет
├── MainWindow.xaml           ← Frame-контейнер
└── App.xaml
```

### Создание Page

```xml
<Page x:Class="CinemaKiosk.Pages.MoviesPage"
      xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
      xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
      Title="Выбор фильма">
    <Grid>
        <!-- Контент страницы -->
        <ListBox x:Name="MoviesListBox" 
                 SelectionChanged="MoviesListBox_SelectionChanged"/>
    </Grid>
</Page>
```

```csharp
public partial class MoviesPage : Page
{
    public MoviesPage()
    {
        InitializeComponent();
        Loaded += MoviesPage_Loaded;
    }
    
    private async void MoviesPage_Loaded(object sender, RoutedEventArgs e)
    {
        var movies = await _api.GetMoviesAsync();
        MoviesListBox.ItemsSource = movies;
    }
    
    private void MoviesListBox_SelectionChanged(object sender, SelectionChangedEventArgs e)
    {
        if (MoviesListBox.SelectedItem is Movie movie)
        {
            // Переход на следующую страницу
            NavigationService.Navigate(new SessionsPage(movie.Id));
        }
    }
}
```

---

## Передача данных между страницами

### Способ 1: Через конструктор

```csharp
// MoviesPage → SessionsPage
NavigationService.Navigate(new SessionsPage(movieId: 42));

// SessionsPage
public partial class SessionsPage : Page
{
    private readonly int _movieId;
    
    public SessionsPage(int movieId)
    {
        InitializeComponent();
        _movieId = movieId;
    }
}
```

**Плюсы:** Явная зависимость, типобезопасность
**Минусы:** Для каждой комбинации параметров — свой конструктор

### Способ 2: Через свойства

```csharp
var page = new SessionsPage();
page.MovieId = 42;
NavigationService.Navigate(page);
```

### Способ 3: NavigationService с параметром

```csharp
// Отправка
NavigationService.Navigate(new Uri("SessionsPage.xaml?movieId=42", UriKind.Relative));

// Получение (в OnNavigatedTo)
protected override void OnNavigatedTo(NavigationEventArgs e)
{
    var query = NavigationService.CurrentSource.ToString();
    // Парсинг movieId из query string
}
```

### Способ 4: Shared State / Service

```csharp
// Общий сервис
public class BookingContext
{
    public Movie SelectedMovie { get; set; }
    public Session SelectedSession { get; set; }
    public List<Seat> SelectedSeats { get; set; } = new();
}

// В каждой странице
public partial class SessionsPage : Page
{
    private readonly BookingContext _context;
    
    public SessionsPage(BookingContext context)
    {
        _context = context;
        // Используем _context.SelectedMovie
    }
}
```

**Плюсы:** Удобно для многошагового процесса
**Минусы:** Глобальное состояние, сложнее тестировать

---

## Жизненный цикл страницы

### События навигации

```csharp
public partial class HallPage : Page
{
    public HallPage()
    {
        InitializeComponent();
    }
    
    // Вызывается при переходе НА страницу
    protected override void OnNavigatedTo(NavigationEventArgs e)
    {
        base.OnNavigatedTo(e);
        
        // Загрузка данных
        await LoadHallAsync();
    }
    
    // Вызывается при уходе СО страницы
    protected override void OnNavigatingFrom(NavigatingCancelEventArgs e)
    {
        base.OnNavigatingFrom(e);
        
        // Можно отменить навигацию
        if (_hasUnsavedChanges)
        {
            var result = MessageBox.Show(
                "Вы уверены? Выбранные места будут сброшены.",
                "Подтверждение",
                MessageBoxButton.YesNo);
            
            if (result == MessageBoxResult.No)
            {
                e.Cancel = true;  // Отмена навигации
            }
        }
    }
    
    // Вызывается после ухода со страницы
    protected override void OnNavigatedFrom(NavigationEventArgs e)
    {
        base.OnNavigatedFrom(e);
        
        // Очистка ресурсов
        _cancellationTokenSource?.Cancel();
    }
}
```

### Диаграмма жизненного цикла

```
┌──────────────────┐
│  Constructor     │
└────────┬─────────┘
         ▼
┌──────────────────┐
│    Loaded        │
└────────┬─────────┘
         ▼
┌──────────────────┐
│ OnNavigatedTo    │ ← Загрузка данных
└────────┬─────────┘
         │
         │ (пользователь работает)
         │
         ▼
┌──────────────────┐
│OnNavigatingFrom  │ ← Можно отменить
└────────┬─────────┘
         ▼
┌──────────────────┐
│ OnNavigatedFrom  │ ← Очистка
└──────────────────┘
```

---

## Навигация "Назад"

### Встроенная история

Frame автоматически ведёт историю:

```csharp
// Назад
if (NavigationService.CanGoBack)
{
    NavigationService.GoBack();
}

// Вперёд
if (NavigationService.CanGoForward)
{
    NavigationService.GoForward();
}
```

### Кнопка "Назад"

```xml
<Button Content="← Назад" 
        Click="BackButton_Click"
        IsEnabled="{Binding CanGoBack, ElementName=MainFrame}"/>
```

```csharp
private void BackButton_Click(object sender, RoutedEventArgs e)
{
    if (MainFrame.CanGoBack)
        MainFrame.GoBack();
}
```

### Очистка истории (для киоска)

После завершения покупки нужно вернуться на начало:

```csharp
private void CompleteBooking()
{
    // Очищаем историю, чтобы следующий пользователь не видел предыдущие экраны
    while (NavigationService.CanGoBack)
    {
        NavigationService.RemoveBackEntry();
    }
    
    NavigationService.Navigate(new MoviesPage());
}
```

---

## Single Page Application (SPA) подход

### Концепция

Вместо отдельных Page можно использовать **один Window с переключаемым контентом**:

```xml
<Window>
    <Grid>
        <ContentControl x:Name="MainContent"/>
    </Grid>
</Window>
```

```csharp
// "Навигация" — просто смена контента
MainContent.Content = new MoviesView();
MainContent.Content = new SessionsView();
```

### View вместо Page

```csharp
public partial class MoviesView : UserControl
{
    // Логика аналогична Page
}
```

### Плюсы SPA-подхода

- Больше контроля над переходами
- Легче реализовать анимации
- Проще shared state
- Не зависит от Frame/NavigationService

### Минусы

- Нет встроенной истории (нужно реализовать самим)
- Больше кода

---

## Полный User Flow

### Диаграмма приложения

```
┌───────────────┐
│   Фильмы      │
│   (MoviesPage)│
└───────┬───────┘
        │ Выбор фильма
        ▼
┌───────────────┐
│   Сеансы      │
│ (SessionsPage)│
└───────┬───────┘
        │ Выбор сеанса
        ▼
┌───────────────┐
│   Зал         │
│  (HallPage)   │
└───────┬───────┘
        │ Выбор мест + "Купить"
        ▼
┌───────────────┐     Ошибка     ┌───────────────┐
│   Обработка   │ ─────────────▶ │   Сообщение   │
│   платежа     │                │   об ошибке   │
└───────┬───────┘                └───────┬───────┘
        │ Успех                          │
        ▼                                │
┌───────────────┐                        │
│   Билет       │                        │
│ (TicketPage)  │                        │
└───────┬───────┘                        │
        │ "На главную"                   │
        ▼                                ▼
┌───────────────────────────────────────────┐
│               Фильмы (начало)              │
└───────────────────────────────────────────┘
```

### Реализация навигации

```csharp
// MainWindow.xaml.cs
public partial class MainWindow : Window
{
    private readonly BookingContext _context;
    private readonly ApiService _api;
    
    public MainWindow()
    {
        InitializeComponent();
        
        _api = new ApiService();
        _context = new BookingContext();
        
        // Начальная страница
        MainFrame.Navigate(new MoviesPage(_api, _context, NavigateToSessions));
    }
    
    private void NavigateToSessions(int movieId)
    {
        MainFrame.Navigate(new SessionsPage(_api, _context, movieId, NavigateToHall));
    }
    
    private void NavigateToHall(int sessionId)
    {
        MainFrame.Navigate(new HallPage(_api, _context, sessionId, NavigateToTicket, NavigateToMovies));
    }
    
    private void NavigateToTicket(BookingResponse booking)
    {
        MainFrame.Navigate(new TicketPage(booking, NavigateToMovies));
    }
    
    private void NavigateToMovies()
    {
        // Очищаем контекст и историю
        _context.Clear();
        while (MainFrame.CanGoBack)
            MainFrame.RemoveBackEntry();
        
        MainFrame.Navigate(new MoviesPage(_api, _context, NavigateToSessions));
    }
}
```

---

## Связь с лабораторной работой

### Лаба 6: Навигация

В лабораторной работе вы:

1. **Создадите несколько Page:**
   - MoviesPage → SessionsPage → HallPage → TicketPage

2. **Реализуете переходы:**
   ```csharp
   NavigationService.Navigate(new HallPage(sessionId));
   ```

3. **Организуете полный user flow:**
   - Выбор фильма → Выбор сеанса → Выбор мест → Билет

---

## Заключение

**Навигация** — это не просто переходы между экранами. Это **архитектура** приложения:

- Чёткий flow помогает пользователю
- Правильная передача данных упрощает код
- Жизненный цикл страниц — управление ресурсами

Продуманная навигация делает приложение **понятным** для пользователя и **поддерживаемым** для разработчика.

---

## Полезные ресурсы

- [Navigation Overview in WPF](https://docs.microsoft.com/en-us/dotnet/desktop/wpf/app-development/navigation-overview)
- [Page Class](https://docs.microsoft.com/en-us/dotnet/api/system.windows.controls.page)
- [NavigationService](https://docs.microsoft.com/en-us/dotnet/api/system.windows.navigation.navigationservice)
