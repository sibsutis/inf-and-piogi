---
title: Лаба 4
tags:
  - usercontrol
  - components
  - dynamic-ui
---

## Вступление

Данная лабораторная работа посвящена созданию кастомных компонентов (UserControl) и динамической генерации UI на основе данных с API.

**Что будем делать:**

- Создание `UserControl` для места в зале
- Запрос матрицы мест с API
- Динамическая генерация карты зала
- Логика выбора/отмены мест

Это самая жёсткая лаба по алгоритмам отрисовки. Реальные данные покажут реальную геометрию залов.

---

## Техническое задание

[Требования к программе, необходимые для сдачи.](../tasks/task4.md)

---

## Git-воркфлоу

```bash
git checkout -b feature/lab-4-controls
```

---

## Концепция: Компонентно-ориентированное программирование

**UserControl** — пользовательский элемент управления, который позволяет создавать переиспользуемые компоненты из базовых элементов.

Можно сказать, что это визуальный аналог класса — мы выделяем основные свойства объекта в его визуальном представлении.

---

## Шаг 1: Создание UserControl для места

1. **ПКМ по проекту** → **Add** → **User Control (WPF)**
2. Название: `SeatControl.xaml`

```xml
<UserControl x:Class="CinemaKiosk.Controls.SeatControl"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             Width="40" Height="40">
    <Border x:Name="SeatBorder" 
            BorderBrush="Gray" 
            BorderThickness="1" 
            CornerRadius="5"
            Background="LightGray"
            Cursor="Hand">
        <TextBlock x:Name="SeatNumber" 
                   HorizontalAlignment="Center" 
                   VerticalAlignment="Center"
                   FontSize="12"/>
    </Border>
</UserControl>
```

---

## Шаг 2: Логика компонента

В `SeatControl.xaml.cs`:

```csharp
public partial class SeatControl : UserControl
{
    public int Row { get; set; }
    public int Number { get; set; }
    public bool IsSelected { get; private set; }
    public bool IsOccupied { get; set; }
    
    public event Action<SeatControl> OnSeatClicked;
    
    public SeatControl(int row, int number, bool isOccupied)
    {
        InitializeComponent();
        
        Row = row;
        Number = number;
        IsOccupied = isOccupied;
        
        SeatNumber.Text = number.ToString();
        
        UpdateVisual();
        
        MouseLeftButtonDown += SeatControl_Click;
    }
    
    private void SeatControl_Click(object sender, MouseButtonEventArgs e)
    {
        if (IsOccupied) return; // Занятое место нельзя выбрать
        
        IsSelected = !IsSelected;
        UpdateVisual();
        
        OnSeatClicked?.Invoke(this);
    }
    
    private void UpdateVisual()
    {
        if (IsOccupied)
        {
            SeatBorder.Background = Brushes.DarkGray;
        }
        else if (IsSelected)
        {
            SeatBorder.Background = Brushes.Green;
        }
        else
        {
            SeatBorder.Background = Brushes.LightGray;
        }
    }
}
```

---

## Шаг 3: Модель данных для мест

Добавь модель `SeatDto.cs`:

```csharp
public class SeatDto
{
    public int Row { get; set; }
    public int Number { get; set; }
    public bool IsOccupied { get; set; }
}

public class HallDto
{
    public int SessionId { get; set; }
    public int Rows { get; set; }
    public int SeatsPerRow { get; set; }
    public List<SeatDto> Seats { get; set; }
}
```

---

## Шаг 4: Метод API для получения мест

В `ApiService.cs`:

```csharp
public async Task<HallDto> GetSeatsAsync(int sessionId)
{
    var response = await _client.GetStringAsync($"{BaseUrl}/sessions/{sessionId}/seats");
    var hall = JsonConvert.DeserializeObject<HallDto>(response);
    return hall;
}
```

---

## Шаг 5: Создание страницы зала

Создай новое окно или Page для отображения зала:

```xml
<Page x:Class="CinemaKiosk.Pages.HallPage"
      xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
      xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
      Title="Выбор мест">
    <Grid>
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="*"/>
            <RowDefinition Height="Auto"/>
        </Grid.RowDefinitions>
        
        <!-- Экран -->
        <Border Grid.Row="0" Background="DarkBlue" Height="30" Margin="50,20">
            <TextBlock Text="ЭКРАН" Foreground="White" 
                       HorizontalAlignment="Center" VerticalAlignment="Center"/>
        </Border>
        
        <!-- Карта зала -->
        <UniformGrid x:Name="SeatsGrid" Grid.Row="1" Margin="20"/>
        
        <!-- Кнопка покупки -->
        <Button Grid.Row="2" Content="Купить билеты" 
                Height="40" Margin="20"
                Click="BuyButton_Click"/>
    </Grid>
</Page>
```

---

## Шаг 6: Динамическая генерация зала

В `HallPage.xaml.cs`:

```csharp
public partial class HallPage : Page
{
    private readonly ApiService _apiService;
    private readonly int _sessionId;
    private List<SeatControl> _selectedSeats = new List<SeatControl>();
    
    public HallPage(int sessionId)
    {
        InitializeComponent();
        _apiService = new ApiService();
        _sessionId = sessionId;
        
        Loaded += HallPage_Loaded;
    }
    
    private async void HallPage_Loaded(object sender, RoutedEventArgs e)
    {
        await LoadHall();
    }
    
    private async Task LoadHall()
    {
        var hall = await _apiService.GetSeatsAsync(_sessionId);
        
        SeatsGrid.Rows = hall.Rows;
        SeatsGrid.Columns = hall.SeatsPerRow;
        
        foreach (var seat in hall.Seats)
        {
            var seatControl = new SeatControl(seat.Row, seat.Number, seat.IsOccupied);
            seatControl.OnSeatClicked += SeatControl_Clicked;
            SeatsGrid.Children.Add(seatControl);
        }
    }
    
    private void SeatControl_Clicked(SeatControl seat)
    {
        if (seat.IsSelected)
        {
            _selectedSeats.Add(seat);
        }
        else
        {
            _selectedSeats.Remove(seat);
        }
        
        // Можно обновить UI: показать количество выбранных мест
    }
    
    private void BuyButton_Click(object sender, RoutedEventArgs e)
    {
        if (_selectedSeats.Count == 0)
        {
            MessageBox.Show("Выберите хотя бы одно место!");
            return;
        }
        
        // Логика покупки будет в следующей лабе
    }
}
```

---

## Визуальная схема

```
        ┌─────────────────────────────┐
        │          ЭКРАН              │
        └─────────────────────────────┘
        
        [1] [2] [3] [4] [5] [6] [7] [8]   Ряд 1
        [1] [2] [■] [■] [5] [6] [7] [8]   Ряд 2
        [1] [2] [3] [4] [5] [6] [7] [8]   Ряд 3
        [1] [2] [3] [4] [5] [6] [7] [8]   Ряд 4
        
        ■ = занято    [N] = свободно
```

---

## Коммит и Push

```bash
git add .
git commit -m "feat: add SeatControl and hall generation"
git push origin feature/lab-4-controls
```

---

## Полезные ссылки

- [UserControl Overview](../topics/UserControl.md)
- [События в C#](../topics/events.md)
- [UniformGrid](https://docs.microsoft.com/en-us/dotnet/api/system.windows.controls.primitives.uniformgrid)

