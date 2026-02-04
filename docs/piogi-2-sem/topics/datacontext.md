> `DataContext` — это свойство, которое указывает, откуда берутся данные для привязок в элементах интерфейса. Это как корзина с фруктами, из которой элементы UI берут нужные им данные.

## Как работает DataContext?

1. `DataContext` задается для элемента (например, для окна) и автоматически передается всем дочерним элементам
2. Когда мы используем привязку `{Binding Name}`, WPF ищет свойство `Name` в объекте, который хранится в `DataContext`

## Простой пример установки DataContext

### Шаг 1: Создаем класс с данными

```csharp
public class Room
{
    public string Name { get; set; }
    public double Temperature { get; set; }
}
```

### Шаг 2: Устанавливаем DataContext в коде

```csharp
public partial class MainWindow : Window
{
    public MainWindow()
    {
        InitializeComponent();

        // Создаем комнату
        Room livingRoom = new Room 
        { 
            Name = "Гостиная", 
            Temperature = 22.5 
        };

        // Устанавливаем DataContext для окна
        this.DataContext = livingRoom;
    }
}
```

Пояснение:
1. Создается новый объект класса `Room` с именем `livingRoom`. Ему присваиваются начальные значения для имени ("Гостиная") и температуры (22.5).
2. **Установка `DataContext`:** `DataContext` окна `MainWindow` устанавливается равным созданному объекту `livingRoom`. Это связывает данные, содержащиеся в объекте `livingRoom`, с элементами управления в окне, позволяя им отображать и редактировать эти данные. В дальнейшем, изменения в `livingRoom` будут автоматически отражаться в интерфейсе, и наоборот.
### Шаг 3: Используем привязки в XAML

```xml
<Window x:Class="WpfApp.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="Информация о комнате" Height="200" Width="300">
    <StackPanel Margin="10">
        <TextBlock Text="Название комнаты:" FontWeight="Bold"/>
        <TextBlock Text="{Binding Name}" Margin="0,0,0,10"/>

        <TextBlock Text="Температура:" FontWeight="Bold"/>
        <TextBlock Text="{Binding Temperature, StringFormat={0}°C}"/>
    </StackPanel>
</Window>
```

В этом примере:

- Мы создали объект `Room` в коде
- Установили его как `DataContext` для окна
- В XAML используем привязки к свойствам `Name` и `Temperature`

## Наследование DataContext

Важно понимать, что `DataContext` наследуется от родительских элементов к дочерним:

```xml
<Grid>
    <!-- Эти TextBlock наследуют DataContext от Window -->
    <TextBlock Text="{Binding Name}" HorizontalAlignment="Left" VerticalAlignment="Top" Margin="10"/>
    <TextBlock Text="{Binding Temperature}" HorizontalAlignment="Right" VerticalAlignment="Top" Margin="10"/>
</Grid>
```

## DataContext и списки элементов

Рассмотрим пример со списком комнат:
### Шаг 1: Создаем список комнат в коде

```csharp
public partial class MainWindow : Window
{
    public List<Room> Rooms { get; set; }

    public MainWindow()
    {
        InitializeComponent();

        // Создаем список комнат
        Rooms = new List<Room>
        {
            new Room { Name = "Гостиная", Temperature = 22.5 },
            new Room { Name = "Спальня", Temperature = 20.0 },
            new Room { Name = "Кухня", Temperature = 24.5 }
        };

        // Устанавливаем DataContext для окна
        this.DataContext = this;  // this - это сам класс MainWindow
    }
}
```

### Шаг 2: Используем ItemsSource и привязки

```xml
<Window x:Class="WpfApp.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="Список комнат" Height="300" Width="400">
    <Grid Margin="10">
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="*"/>
        </Grid.RowDefinitions>

        <TextBlock Text="Выберите комнату:" Grid.Row="0" Margin="0,0,0,5"/>

        <ListBox Grid.Row="1" ItemsSource="{Binding Rooms}">
            <ListBox.ItemTemplate>
                <DataTemplate>
                    <!-- Внутри DataTemplate DataContext автоматически меняется 
                         на текущий элемент списка (Room) -->
                    <StackPanel Orientation="Horizontal">
                        <TextBlock Text="{Binding Name}" FontWeight="Bold" Margin="0,0,5,0"/>
                        <TextBlock Text="{Binding Temperature, StringFormat={0}°C}"/>
                    </StackPanel>
                </DataTemplate>
            </ListBox.ItemTemplate>
        </ListBox>
    </Grid>
</Window>
```
В этом примере:

- `DataContext` окна установлен на сам класс `MainWindow`
- `ListBox` использует привязку `ItemsSource="{Binding Rooms}"` для отображения списка комнат
- Внутри `DataTemplate` контекст данных автоматически меняется на текущий элемент из списка `Rooms`

## Установка DataContext в XAML

Можно установить `DataContext` прямо в XAML:

```xml
<Window x:Class="WpfApp.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:local="clr-namespace:WpfApp"
        Title="Комнаты" Height="300" Width="400">

    <!-- Создаем экземпляр класса Room прямо в XAML -->
    <Window.DataContext>
        <local:Room Name="Гостиная" Temperature="22.5"/>
    </Window.DataContext>

    <StackPanel Margin="10">
        <TextBlock Text="{Binding Name}" FontSize="18" FontWeight="Bold"/>
        <TextBlock Text="{Binding Temperature, StringFormat=Температура: {0}°C}" Margin="0,10,0,0"/>
    </StackPanel>
</Window>
```

## Один из главных принципов DataContext

**Важно запомнить:** `DataContext` определяет, где WPF будет искать свойства, указанные в привязках `{Binding ...}`.

## Простой пример работы со свойствами элементов

Иногда нужно привязаться к свойствам других элементов UI, а не к `DataContext`. Для этого используется `ElementName`:

```xml
<StackPanel Margin="10">
    <Slider x:Name="temperatureSlider" Minimum="15" Maximum="30" Value="22" Margin="0,0,0,10"/>
    <TextBlock Text="{Binding Value, ElementName=temperatureSlider, StringFormat=Выбранная температура: {0}°C}"/>
</StackPanel>
```

В этом примере `TextBlock` привязан к значению `Slider` через `ElementName`, а не через `DataContext`.

## Вывод

`DataContext` — это:

- Контейнер с данными, который определяет, где WPF ищет свойства для привязок
- Он наследуется от родителя к дочерним элементам
- Можно установить как в коде, так и в XAML
- Внутри шаблонов данных (`DataTemplate`) он автоматически меняется на текущий элемент списка