---
title: Лекция 4.2 - Компонентный подход и переиспользование
---

# Компонентный подход и переиспользование

## Введение: Проблема копипаста

Представьте: вам нужно отобразить информацию о месте в зале кинотеатра. Вы пишете XAML для одного места, потом копируете 100 раз для всех мест, меняя координаты...

Через неделю дизайнер просит изменить цвет занятых мест. Вы меняете в 100 местах. Пропускаете одно. Баг.

**Компонентный подход** решает эту проблему: один раз описываем компонент, используем многократно.

---

## Принцип DRY

### Don't Repeat Yourself

**DRY (Don't Repeat Yourself)** — фундаментальный принцип разработки:

> "Каждый фрагмент знания должен иметь единственное, непротиворечивое представление в системе."

**Нарушение DRY:**
```csharp
// В одном месте
if (seat.IsOccupied)
{
    seatButton.Background = Brushes.DarkGray;
    seatButton.IsEnabled = false;
}

// В другом месте (копипаст!)
if (seat.IsOccupied)
{
    seatButton.Background = Brushes.DarkGray;
    seatButton.IsEnabled = false;
}
```

Если нужно изменить цвет занятого места — нужно найти и изменить везде.

**Соблюдение DRY:**
```csharp
void UpdateSeatVisual(Button seatButton, Seat seat)
{
    if (seat.IsOccupied)
    {
        seatButton.Background = Brushes.DarkGray;
        seatButton.IsEnabled = false;
    }
}

// Используем везде
UpdateSeatVisual(seat1Button, seat1);
UpdateSeatVisual(seat2Button, seat2);
```

Изменение в одном месте — применяется везде.

---

## Атомарный дизайн

### Методология Brad Frost

**Atomic Design** — методология создания дизайн-систем, предложенная Брэдом Фростом. Идея: строить интерфейс как химические соединения — от атомов к молекулам, организмам и страницам.

### Уровни атомарного дизайна

```
┌─────────────────────────────────────────────────────────────────┐
│  Pages (Страницы)                                                │
│  Конкретные экземпляры шаблонов с реальными данными             │
├─────────────────────────────────────────────────────────────────┤
│  Templates (Шаблоны)                                             │
│  Каркасы страниц, определяющие структуру                        │
├─────────────────────────────────────────────────────────────────┤
│  Organisms (Организмы)                                           │
│  Группы молекул: хедер, карточка товара, форма                  │
├─────────────────────────────────────────────────────────────────┤
│  Molecules (Молекулы)                                            │
│  Группы атомов: поле поиска, элемент меню                       │
├─────────────────────────────────────────────────────────────────┤
│  Atoms (Атомы)                                                   │
│  Базовые элементы: кнопка, иконка, поле ввода                   │
└─────────────────────────────────────────────────────────────────┘
```

### Примеры в контексте киноприложения

**Атомы:**
- Кнопка
- Иконка
- Текстовый блок
- Изображение

**Молекулы:**
- SeatControl (кнопка + номер + состояние)
- MovieCard (постер + название + длительность)

**Организмы:**
- HallGrid (сетка из SeatControl + экран + легенда)
- MovieList (список MovieCard + фильтр + сортировка)

**Шаблоны:**
- Страница выбора фильма (шапка + MovieList + детали)
- Страница выбора мест (шапка + HallGrid + кнопка покупки)

**Страницы:**
- Конкретная страница с данными "Мстители", "Зал 1", etc.

---

## UserControl — пользовательский элемент управления

### Что такое UserControl

**UserControl** — способ создания переиспользуемых компонентов в WPF. Это комбинация XAML-разметки и C#-кода, которую можно использовать как единый элемент.

### Создание UserControl

1. **ПКМ по проекту** → **Add** → **User Control (WPF)**
2. Название: `SeatControl.xaml`

**SeatControl.xaml:**
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

**SeatControl.xaml.cs:**
```csharp
public partial class SeatControl : UserControl
{
    public int Row { get; set; }
    public int Number { get; set; }
    public bool IsSelected { get; private set; }
    public bool IsOccupied { get; set; }
    
    public event Action<SeatControl> SeatClicked;
    
    public SeatControl()
    {
        InitializeComponent();
        MouseLeftButtonDown += OnClick;
    }
    
    public void Initialize(int row, int number, bool isOccupied)
    {
        Row = row;
        Number = number;
        IsOccupied = isOccupied;
        SeatNumber.Text = number.ToString();
        UpdateVisual();
    }
    
    private void OnClick(object sender, MouseButtonEventArgs e)
    {
        if (IsOccupied) return;
        
        IsSelected = !IsSelected;
        UpdateVisual();
        SeatClicked?.Invoke(this);
    }
    
    private void UpdateVisual()
    {
        if (IsOccupied)
            SeatBorder.Background = Brushes.DarkGray;
        else if (IsSelected)
            SeatBorder.Background = Brushes.LightGreen;
        else
            SeatBorder.Background = Brushes.LightGray;
    }
}
```

### Использование UserControl

```xml
<Window xmlns:controls="clr-namespace:CinemaKiosk.Controls">
    <StackPanel>
        <controls:SeatControl x:Name="Seat1"/>
        <controls:SeatControl x:Name="Seat2"/>
    </StackPanel>
</Window>
```

```csharp
Seat1.Initialize(1, 1, false);  // Ряд 1, Место 1, свободно
Seat2.Initialize(1, 2, true);   // Ряд 1, Место 2, занято
```

---

## UserControl vs Custom Control

### Когда использовать UserControl

**UserControl** — когда нужна конкретная комбинация существующих элементов:

- Форма ввода данных
- Карточка товара
- Виджет погоды
- Любой "составной" компонент

**Плюсы:**
- Быстро создать
- XAML + Code-behind
- Понятная структура

**Минусы:**
- Нельзя полностью переопределить внешний вид через Template
- Ограниченная настраиваемость

### Когда использовать Custom Control

**Custom Control** — когда создаёте новый элемент управления с полной гибкостью стилизации:

- Новый тип кнопки
- Кастомный слайдер
- Элемент, который будет использоваться в разных проектах с разными стилями

**Плюсы:**
- Полная поддержка тем и стилей
- Можно переопределить ControlTemplate
- Профессиональный подход

**Минусы:**
- Сложнее создавать
- Больше кода

### Пример Custom Control

```csharp
public class RatingControl : Control
{
    static RatingControl()
    {
        DefaultStyleKeyProperty.OverrideMetadata(typeof(RatingControl),
            new FrameworkPropertyMetadata(typeof(RatingControl)));
    }
    
    public static readonly DependencyProperty RatingProperty =
        DependencyProperty.Register("Rating", typeof(int), typeof(RatingControl),
            new PropertyMetadata(0));
    
    public int Rating
    {
        get => (int)GetValue(RatingProperty);
        set => SetValue(RatingProperty, value);
    }
}
```

**Generic.xaml:**
```xml
<Style TargetType="{x:Type local:RatingControl}">
    <Setter Property="Template">
        <Setter.Value>
            <ControlTemplate TargetType="{x:Type local:RatingControl}">
                <!-- Внешний вид определяется здесь -->
                <!-- Можно переопределить в другом стиле -->
            </ControlTemplate>
        </Setter.Value>
    </Setter>
</Style>
```

---

## События — коммуникация между компонентами

### Проблема

Как родительский элемент узнает, что произошло внутри дочернего компонента?

```
┌─────────────────────────────────────┐
│           HallPage                   │
│                                     │
│  ┌─────────────────────────────┐   │
│  │         HallGrid             │   │
│  │  ┌─────┐ ┌─────┐ ┌─────┐    │   │
│  │  │Seat │ │Seat │ │Seat │    │   │  ← Клик!
│  │  └─────┘ └─────┘ └─────┘    │   │
│  └─────────────────────────────┘   │
│                                     │
│  [Кнопка "Купить"]  ← Должна знать, │
│                       что выбрано   │
└─────────────────────────────────────┘
```

### Решение: События (Events)

Компонент определяет событие:

```csharp
public partial class SeatControl : UserControl
{
    // Определяем событие
    public event Action<SeatControl> SeatClicked;
    
    private void OnClick(object sender, MouseButtonEventArgs e)
    {
        // Вызываем событие
        SeatClicked?.Invoke(this);
    }
}
```

Родитель подписывается:

```csharp
public partial class HallPage : Page
{
    private List<SeatControl> _selectedSeats = new List<SeatControl>();
    
    private void CreateSeat(int row, int number)
    {
        var seat = new SeatControl();
        seat.Initialize(row, number, false);
        
        // Подписываемся на событие
        seat.SeatClicked += OnSeatClicked;
        
        SeatsGrid.Children.Add(seat);
    }
    
    private void OnSeatClicked(SeatControl seat)
    {
        if (seat.IsSelected)
            _selectedSeats.Add(seat);
        else
            _selectedSeats.Remove(seat);
        
        UpdateTotalPrice();
    }
}
```

### Routed Events — продвинутые события WPF

WPF поддерживает **маршрутизируемые события** — они могут "всплывать" вверх по дереву элементов:

```csharp
// Определение Routed Event
public static readonly RoutedEvent SeatClickedEvent = 
    EventManager.RegisterRoutedEvent(
        "SeatClicked",
        RoutingStrategy.Bubble,  // Всплывает вверх
        typeof(RoutedEventHandler),
        typeof(SeatControl));

public event RoutedEventHandler SeatClicked
{
    add => AddHandler(SeatClickedEvent, value);
    remove => RemoveHandler(SeatClickedEvent, value);
}

// Вызов
private void OnClick(object sender, MouseButtonEventArgs e)
{
    RaiseEvent(new RoutedEventArgs(SeatClickedEvent, this));
}
```

Теперь можно подписаться на уровне родителя:

```xml
<UniformGrid local:SeatControl.SeatClicked="OnAnySeatClicked">
    <!-- Все SeatControl внутри -->
</UniformGrid>
```

---

## Dependency Properties в UserControl

### Зачем нужны

Чтобы использовать Binding с вашим UserControl, свойства должны быть Dependency Properties:

```xml
<!-- Не сработает с обычным свойством! -->
<controls:SeatControl IsOccupied="{Binding Seat.IsBooked}"/>
```

### Как добавить

```csharp
public partial class SeatControl : UserControl
{
    public static readonly DependencyProperty IsOccupiedProperty =
        DependencyProperty.Register(
            nameof(IsOccupied),
            typeof(bool),
            typeof(SeatControl),
            new PropertyMetadata(false, OnIsOccupiedChanged));
    
    public bool IsOccupied
    {
        get => (bool)GetValue(IsOccupiedProperty);
        set => SetValue(IsOccupiedProperty, value);
    }
    
    private static void OnIsOccupiedChanged(DependencyObject d, DependencyPropertyChangedEventArgs e)
    {
        var control = (SeatControl)d;
        control.UpdateVisual();
    }
}
```

Теперь Binding работает:

```xml
<controls:SeatControl IsOccupied="{Binding IsBooked}"/>
```

---

## Сравнение с веб-фреймворками

### React

```jsx
// React компонент
function SeatControl({ row, number, isOccupied, onClick }) {
    const [isSelected, setIsSelected] = useState(false);
    
    const handleClick = () => {
        if (!isOccupied) {
            setIsSelected(!isSelected);
            onClick?.();
        }
    };
    
    return (
        <div 
            className={`seat ${isOccupied ? 'occupied' : ''} ${isSelected ? 'selected' : ''}`}
            onClick={handleClick}
        >
            {number}
        </div>
    );
}
```

### Vue

```vue
<!-- Vue компонент -->
<template>
    <div :class="seatClasses" @click="handleClick">
        {{ number }}
    </div>
</template>

<script>
export default {
    props: ['row', 'number', 'isOccupied'],
    data() {
        return { isSelected: false };
    },
    computed: {
        seatClasses() {
            return {
                seat: true,
                occupied: this.isOccupied,
                selected: this.isSelected
            };
        }
    },
    methods: {
        handleClick() {
            if (!this.isOccupied) {
                this.isSelected = !this.isSelected;
                this.$emit('click');
            }
        }
    }
};
</script>
```

### Общие принципы

Несмотря на разный синтаксис, принципы одинаковы:

| Концепция | WPF | React | Vue |
|-----------|-----|-------|-----|
| Компонент | UserControl | Function/Class | SFC |
| Входные данные | Properties/DP | props | props |
| Внутреннее состояние | Fields | useState | data |
| Уведомление родителя | Events | Callbacks | $emit |
| Стилизация | Styles/Resources | CSS/Styled | Scoped CSS |

---

## Связь с лабораторной работой

### Лаба 4: UserControls

В лабораторной работе 4 вы создадите:

1. **SeatControl** — компонент для отображения места:
   ```csharp
   public partial class SeatControl : UserControl
   {
       public int Row { get; set; }
       public int Number { get; set; }
       public bool IsSelected { get; private set; }
       public event Action<SeatControl> OnSeatClicked;
   }
   ```

2. **Динамическую генерацию** — создание компонентов из данных API:
   ```csharp
   foreach (var seat in hall.Seats)
   {
       var control = new SeatControl(seat.Row, seat.Number, seat.IsOccupied);
       control.OnSeatClicked += HandleSeatClick;
       SeatsGrid.Children.Add(control);
   }
   ```

3. **Коммуникацию** — события для связи с родителем

---

## Заключение

**Компонентный подход** — это не просто организация кода, это **способ думать** об интерфейсе:

- Разбивай на независимые части
- Каждая часть делает одну вещь хорошо
- Переиспользуй везде, где нужно
- Общайся через чёткие интерфейсы (свойства и события)

Эти принципы универсальны и применимы в любой технологии: WPF, React, Vue, Flutter, SwiftUI.

---

## Полезные ресурсы

- [UserControl Overview](https://docs.microsoft.com/en-us/dotnet/desktop/wpf/controls/how-to-create-a-usercontrol)
- [Custom Control Overview](https://docs.microsoft.com/en-us/dotnet/desktop/wpf/controls/control-authoring-overview)
- [Atomic Design by Brad Frost](https://bradfrost.com/blog/post/atomic-web-design/)
- [Dependency Properties](https://docs.microsoft.com/en-us/dotnet/desktop/wpf/advanced/dependency-properties-overview)
