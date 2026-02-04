---
title: Лекция 4.0 - Декларативное vs Императивное программирование
---

# Декларативное vs Императивное программирование

## Введение: Две философии

Представьте, что вы хотите добраться из точки А в точку Б.

**Императивный подход:** "Выйди из дома, поверни направо, пройди 100 метров, на светофоре поверни налево, пройди ещё 200 метров, войди в здание..."

**Декларативный подход:** "Мне нужно в библиотеку на улице Пушкина, 15."

В первом случае вы описываете **как** добраться (каждый шаг). Во втором — **что** вам нужно (конечный результат).

Эти две парадигмы лежат в основе всего программирования.

---

## Императивное программирование

### Определение

**Императивное программирование** — парадигма, где программа описывает **последовательность команд**, которые изменяют состояние программы.

Ключевое слово: **КАК**

### Характеристики

- Явное управление потоком выполнения (if, for, while)
- Изменяемое состояние (переменные меняются)
- Пошаговые инструкции
- Фокус на алгоритме решения

### Пример: Создание кнопки в WinForms (императивный стиль)

```csharp
// Код в C# — императивный подход
Button button = new Button();
button.Text = "Нажми меня";
button.Width = 100;
button.Height = 30;
button.Location = new Point(50, 50);
button.BackColor = Color.Blue;
button.ForeColor = Color.White;
button.Click += Button_Click;
this.Controls.Add(button);
```

Каждая строка — команда: "создай", "установи", "добавь".

### Пример: Фильтрация списка (императивный стиль)

```csharp
// Императивный подход
List<Movie> longMovies = new List<Movie>();
foreach (var movie in movies)
{
    if (movie.Duration > 120)
    {
        longMovies.Add(movie);
    }
}
```

Мы явно описываем **как** отфильтровать: создай список, пройди по каждому элементу, проверь условие, добавь в результат.

---

## Декларативное программирование

### Определение

**Декларативное программирование** — парадигма, где программа описывает **что должно быть достигнуто**, а не как это сделать.

Ключевое слово: **ЧТО**

### Характеристики

- Описание желаемого результата
- Минимум побочных эффектов
- Абстракция над деталями реализации
- Фокус на логике, а не на алгоритме

### Пример: Создание кнопки в WPF (декларативный стиль)

```xml
<!-- XAML — декларативный подход -->
<Button Content="Нажми меня"
        Width="100" Height="30"
        Margin="50"
        Background="Blue"
        Foreground="White"
        Click="Button_Click"/>
```

Мы описываем **что** хотим видеть: кнопку с таким-то текстом, такого-то размера, такого-то цвета.

### Пример: Фильтрация списка (декларативный стиль)

```csharp
// Декларативный подход с LINQ
var longMovies = movies.Where(m => m.Duration > 120);
```

Мы описываем **что** хотим получить: фильмы, где длительность больше 120.

---

## Сравнение парадигм

### Пример: Обновление UI

**Императивный подход (WinForms):**

```csharp
// Когда данные меняются — вручную обновляем UI
movie.Title = "Новое название";
titleLabel.Text = movie.Title;  // Нужно помнить обновить!
```

**Декларативный подход (WPF с Binding):**

```xml
<TextBlock Text="{Binding Title}"/>
```

```csharp
// Данные меняются — UI обновляется автоматически!
movie.Title = "Новое название";
// TextBlock обновится сам через Binding
```

### Таблица сравнения

| Аспект | Императивный | Декларативный |
|--------|--------------|---------------|
| Фокус | Как сделать | Что получить |
| Контроль | Полный | Абстрагированный |
| Код | Больше | Меньше |
| Читаемость | Детали видны | Намерение понятно |
| Гибкость | Максимальная | Ограниченная |
| Ошибки | Легко допустить | Сложнее |
| Примеры | C, Java (императивная часть) | SQL, XAML, HTML, CSS |

---

## XAML как декларативный язык

### Что такое XAML

**XAML (eXtensible Application Markup Language)** — декларативный язык разметки для описания пользовательского интерфейса в WPF, UWP, MAUI.

### Преимущества XAML

1. **Разделение ответственности**
   - Дизайнер работает с XAML
   - Программист работает с C#
   - Можно использовать Blend для дизайна

2. **Читаемость**
   ```xml
   <StackPanel>
       <TextBlock Text="Имя:"/>
       <TextBox x:Name="NameTextBox"/>
       <Button Content="OK" Click="OkButton_Click"/>
   </StackPanel>
   ```
   Структура интерфейса видна сразу.

3. **Инструменты**
   - Визуальный редактор в Visual Studio
   - Hot Reload — изменения видны без перезапуска
   - Валидация на этапе редактирования

### XAML vs Код

Одно и то же можно написать двумя способами:

**XAML (декларативно):**
```xml
<Grid>
    <Grid.RowDefinitions>
        <RowDefinition Height="Auto"/>
        <RowDefinition Height="*"/>
    </Grid.RowDefinitions>
    
    <TextBlock Grid.Row="0" Text="Заголовок"/>
    <ListBox Grid.Row="1" ItemsSource="{Binding Items}"/>
</Grid>
```

**C# (императивно):**
```csharp
var grid = new Grid();
grid.RowDefinitions.Add(new RowDefinition { Height = GridLength.Auto });
grid.RowDefinitions.Add(new RowDefinition { Height = new GridLength(1, GridUnitType.Star) });

var textBlock = new TextBlock { Text = "Заголовок" };
Grid.SetRow(textBlock, 0);
grid.Children.Add(textBlock);

var listBox = new ListBox();
listBox.SetBinding(ListBox.ItemsSourceProperty, new Binding("Items"));
Grid.SetRow(listBox, 1);
grid.Children.Add(listBox);
```

Какой вариант проще читать и поддерживать?

---

## Data Binding — сердце декларативности WPF

### Проблема без Binding

```csharp
// Императивный подход — синхронизация вручную
public class MovieForm
{
    private Movie _movie;
    private TextBox _titleTextBox;
    
    public void LoadMovie(Movie movie)
    {
        _movie = movie;
        _titleTextBox.Text = movie.Title;  // UI из данных
    }
    
    public void SaveMovie()
    {
        _movie.Title = _titleTextBox.Text;  // Данные из UI
    }
}
```

Нужно помнить синхронизировать данные с UI. В большом проекте это приводит к багам.

### Решение с Binding

```xml
<TextBox Text="{Binding Title, Mode=TwoWay}"/>
```

```csharp
// Binding синхронизирует автоматически!
public class MovieForm
{
    public void LoadMovie(Movie movie)
    {
        DataContext = movie;  // Всё!
    }
}
```

**Binding** сам следит за изменениями и обновляет UI.

### Как работает Binding

```
┌─────────────────┐         ┌─────────────────┐
│    Источник     │         │      Цель       │
│   (ViewModel)   │◀───────▶│     (UI)        │
│   Movie.Title   │ Binding │  TextBox.Text   │
└─────────────────┘         └─────────────────┘
```

1. **OneWay** — данные → UI (по умолчанию для большинства свойств)
2. **TwoWay** — данные ↔ UI (для TextBox, CheckBox)
3. **OneWayToSource** — UI → данные
4. **OneTime** — один раз при загрузке

### INotifyPropertyChanged

Чтобы Binding работал при изменении данных, класс должен уведомлять об изменениях:

```csharp
public class Movie : INotifyPropertyChanged
{
    private string _title;
    
    public string Title
    {
        get => _title;
        set
        {
            if (_title != value)
            {
                _title = value;
                OnPropertyChanged(nameof(Title));
            }
        }
    }
    
    public event PropertyChangedEventHandler PropertyChanged;
    
    protected void OnPropertyChanged(string propertyName)
    {
        PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
    }
}
```

Теперь при изменении `movie.Title = "Новое"` — TextBox обновится автоматически!

---

## Паттерн Observer

### История

Data Binding в WPF — это реализация классического паттерна **Observer** (Наблюдатель).

**Observer** — поведенческий паттерн, где объект (Subject) оповещает зависимые объекты (Observers) об изменении своего состояния.

```
        ┌──────────────────┐
        │     Subject      │
        │  (Movie)         │
        └────────┬─────────┘
                 │ Notify
      ┌──────────┼──────────┐
      ▼          ▼          ▼
┌──────────┐ ┌──────────┐ ┌──────────┐
│Observer 1│ │Observer 2│ │Observer 3│
│(TextBox) │ │(Label)   │ │(Title)   │
└──────────┘ └──────────┘ └──────────┘
```

### В WPF

- **Subject** — объект с данными (реализует `INotifyPropertyChanged`)
- **Observer** — UI-элементы с `{Binding}`
- WPF автоматически подписывает/отписывает наблюдателей

---

## Введение в MVVM

### Проблема "толстого" code-behind

Без архитектуры весь код оказывается в code-behind:

```csharp
// MainWindow.xaml.cs — становится огромным
public partial class MainWindow : Window
{
    private HttpClient _client;
    private List<Movie> _movies;
    
    private async void LoadButton_Click(...)
    {
        var response = await _client.GetAsync(...);
        _movies = JsonConvert.Deserialize...
        MoviesListBox.ItemsSource = _movies;
        // Сотни строк логики...
    }
    
    private void SearchTextBox_TextChanged(...)
    {
        // Ещё логика...
    }
    
    // И так далее...
}
```

Проблемы:
- Невозможно тестировать
- Сложно переиспользовать
- Логика смешана с UI

### MVVM — Model-View-ViewModel

**MVVM** разделяет приложение на три слоя:

```
┌─────────────────────────────────────────────────────┐
│                      View                            │
│  (XAML + минимальный code-behind)                   │
│  - Отображение данных                                │
│  - Реакция на действия пользователя                 │
└───────────────────────┬─────────────────────────────┘
                        │ DataBinding
                        │ Commands
┌───────────────────────▼─────────────────────────────┐
│                   ViewModel                          │
│  (C# класс)                                          │
│  - Данные для отображения                            │
│  - Команды для действий                              │
│  - Логика представления                              │
└───────────────────────┬─────────────────────────────┘
                        │
┌───────────────────────▼─────────────────────────────┐
│                     Model                            │
│  (C# классы + сервисы)                               │
│  - Бизнес-логика                                     │
│  - Доступ к данным (API, БД)                         │
│  - Сущности предметной области                       │
└─────────────────────────────────────────────────────┘
```

### Преимущества MVVM

1. **Тестируемость** — ViewModel можно тестировать без UI
2. **Разделение** — дизайнер работает с View, программист с ViewModel
3. **Переиспользование** — одна ViewModel для разных View
4. **Чистый код** — каждый слой делает своё дело

---

## DataContext — источник правды

### Что такое DataContext

**DataContext** — свойство, которое определяет контекст данных для привязок (Binding) в элементе и его потомках.

```csharp
public MainWindow()
{
    InitializeComponent();
    DataContext = new MainViewModel();  // Устанавливаем контекст
}
```

```xml
<!-- Все Binding в этом окне будут искать свойства в MainViewModel -->
<Window>
    <TextBlock Text="{Binding Title}"/>
    <ListBox ItemsSource="{Binding Movies}"/>
</Window>
```

### Наследование DataContext

DataContext наследуется вниз по дереву элементов:

```xml
<Window DataContext="{...ViewModel...}">        <!-- DataContext = ViewModel -->
    <Grid>                                       <!-- DataContext = ViewModel (наследуется) -->
        <StackPanel>                             <!-- DataContext = ViewModel (наследуется) -->
            <TextBlock Text="{Binding Title}"/> <!-- Binding к ViewModel.Title -->
        </StackPanel>
    </Grid>
</Window>
```

### Изменение DataContext

Можно менять контекст для части UI:

```xml
<ListBox ItemsSource="{Binding Movies}">
    <ListBox.ItemTemplate>
        <DataTemplate>
            <!-- Здесь DataContext = каждый Movie из списка -->
            <TextBlock Text="{Binding Title}"/>
        </DataTemplate>
    </ListBox.ItemTemplate>
</ListBox>
```

---

## Связь с лабораторной работой

### Лаба 3: Data Binding

В лабораторной работе 3 вы:

1. **Используете Binding для карточки фильма:**
   ```xml
   <TextBlock Text="{Binding Title}"/>
   <Image Source="{Binding PosterUrl}"/>
   ```

2. **Связываете выбор с деталями:**
   ```xml
   <StackPanel DataContext="{Binding SelectedItem, ElementName=MoviesListBox}">
   ```

3. **Форматируете данные:**
   ```xml
   <TextBlock Text="{Binding Duration, StringFormat='{}{0} мин'}"/>
   ```

### Почему это важно

Binding позволяет:
- Не синхронизировать UI с данными вручную
- Писать меньше кода
- Избегать багов синхронизации
- Легко менять UI, не трогая логику

---

## Заключение

**Декларативный подход** — это не просто синтаксис, это **способ думать о программе**:

- Описывай **что** хочешь, а не **как** это сделать
- Пусть фреймворк заботится о деталях
- Фокусируйся на логике, а не на низкоуровневых операциях

WPF построен вокруг декларативности: XAML, Binding, Styles, Templates — всё это инструменты, позволяющие описывать UI **декларативно**, а не создавать его **императивно**.

---

## Полезные ресурсы

- [Data Binding Overview](https://docs.microsoft.com/en-us/dotnet/desktop/wpf/data/data-binding-overview)
- [MVVM Pattern](https://docs.microsoft.com/en-us/archive/msdn-magazine/2009/february/patterns-wpf-apps-with-the-model-view-viewmodel-design-pattern)
- [INotifyPropertyChanged](https://docs.microsoft.com/en-us/dotnet/api/system.componentmodel.inotifypropertychanged)
