---
title: Лекция 5.3 - Триггеры и состояния UI
---

# Триггеры и состояния UI

## Введение

Когда вы наводите мышку на кнопку — она меняет цвет. Когда нажимаете — становится темнее. Когда кнопка неактивна — серая и тусклая.

Это **состояния** элемента управления. В WPF они реализуются через **триггеры**.

---

## Конечные автоматы (State Machines)

### Концепция

Элемент UI можно представить как **конечный автомат** — систему с определёнными состояниями и переходами между ними.

**Состояния кнопки:**

```
            MouseEnter
    ┌────────────────────────┐
    │                        ▼
┌───────┐              ┌───────────┐
│Normal │              │  Hover    │
└───┬───┘              └─────┬─────┘
    │                        │
    │ MouseDown              │ MouseDown
    ▼                        ▼
┌───────────┐          ┌───────────┐
│ Pressed   │◀────────▶│  Pressed  │
└───────────┘ MouseUp  │  +Hover   │
                       └───────────┘
```

### Отдельное состояние: Disabled

```
    IsEnabled = false
┌───────┐ ──────────────▶ ┌───────────┐
│ Any   │                 │ Disabled  │
│ State │ ◀────────────── │           │
└───────┘  IsEnabled=true └───────────┘
```

---

## Триггеры в WPF

### Типы триггеров

| Тип | Срабатывает на |
|-----|----------------|
| **Trigger** | Изменение свойства элемента |
| **DataTrigger** | Изменение привязанных данных |
| **EventTrigger** | Событие (для анимаций) |
| **MultiTrigger** | Комбинация нескольких условий |
| **MultiDataTrigger** | Комбинация условий по данным |

### Property Trigger

Срабатывает при изменении свойства элемента:

```xml
<Style TargetType="Button">
    <Setter Property="Background" Value="#3498db"/>
    <Setter Property="Foreground" Value="White"/>
    
    <Style.Triggers>
        <!-- При наведении мыши -->
        <Trigger Property="IsMouseOver" Value="True">
            <Setter Property="Background" Value="#2980b9"/>
        </Trigger>
        
        <!-- При нажатии -->
        <Trigger Property="IsPressed" Value="True">
            <Setter Property="Background" Value="#1a5276"/>
        </Trigger>
        
        <!-- Когда неактивна -->
        <Trigger Property="IsEnabled" Value="False">
            <Setter Property="Opacity" Value="0.5"/>
        </Trigger>
    </Style.Triggers>
</Style>
```

### Data Trigger

Срабатывает при определённом значении привязанных данных:

```xml
<Style TargetType="Border">
    <Style.Triggers>
        <!-- Если статус = "Error" -->
        <DataTrigger Binding="{Binding Status}" Value="Error">
            <Setter Property="Background" Value="#ffebee"/>
            <Setter Property="BorderBrush" Value="#e74c3c"/>
        </DataTrigger>
        
        <!-- Если статус = "Success" -->
        <DataTrigger Binding="{Binding Status}" Value="Success">
            <Setter Property="Background" Value="#e8f5e9"/>
            <Setter Property="BorderBrush" Value="#2ecc71"/>
        </DataTrigger>
    </Style.Triggers>
</Style>
```

### MultiTrigger

Срабатывает при выполнении **всех** условий:

```xml
<Style TargetType="Button">
    <Style.Triggers>
        <MultiTrigger>
            <MultiTrigger.Conditions>
                <Condition Property="IsMouseOver" Value="True"/>
                <Condition Property="IsEnabled" Value="True"/>
            </MultiTrigger.Conditions>
            <MultiTrigger.Setters>
                <Setter Property="Background" Value="Gold"/>
                <Setter Property="Cursor" Value="Hand"/>
            </MultiTrigger.Setters>
        </MultiTrigger>
    </Style.Triggers>
</Style>
```

### Event Trigger

Срабатывает на событие, обычно для **анимаций**:

```xml
<Button Content="Анимация">
    <Button.Triggers>
        <EventTrigger RoutedEvent="MouseEnter">
            <BeginStoryboard>
                <Storyboard>
                    <DoubleAnimation 
                        Storyboard.TargetProperty="Width"
                        To="150" 
                        Duration="0:0:0.2"/>
                </Storyboard>
            </BeginStoryboard>
        </EventTrigger>
        
        <EventTrigger RoutedEvent="MouseLeave">
            <BeginStoryboard>
                <Storyboard>
                    <DoubleAnimation 
                        Storyboard.TargetProperty="Width"
                        To="100" 
                        Duration="0:0:0.2"/>
                </Storyboard>
            </BeginStoryboard>
        </EventTrigger>
    </Button.Triggers>
</Button>
```

---

## Visual States — современный подход

### Проблема триггеров

Триггеры мощные, но при сложной логике становятся громоздкими:
- Много вложенных условий
- Сложно управлять переходами
- Анимации разбросаны по разным триггерам

### VisualStateManager

**VisualStateManager (VSM)** — современный способ управления состояниями:

```xml
<ControlTemplate TargetType="Button">
    <Border x:Name="border" Background="#3498db">
        <VisualStateManager.VisualStateGroups>
            <VisualStateGroup x:Name="CommonStates">
                <VisualState x:Name="Normal"/>
                
                <VisualState x:Name="MouseOver">
                    <Storyboard>
                        <ColorAnimation 
                            Storyboard.TargetName="border"
                            Storyboard.TargetProperty="(Border.Background).(SolidColorBrush.Color)"
                            To="#2980b9" 
                            Duration="0:0:0.2"/>
                    </Storyboard>
                </VisualState>
                
                <VisualState x:Name="Pressed">
                    <Storyboard>
                        <ColorAnimation 
                            Storyboard.TargetName="border"
                            Storyboard.TargetProperty="(Border.Background).(SolidColorBrush.Color)"
                            To="#1a5276" 
                            Duration="0:0:0.1"/>
                    </Storyboard>
                </VisualState>
                
                <VisualState x:Name="Disabled">
                    <Storyboard>
                        <DoubleAnimation 
                            Storyboard.TargetName="border"
                            Storyboard.TargetProperty="Opacity"
                            To="0.5" 
                            Duration="0:0:0.2"/>
                    </Storyboard>
                </VisualState>
            </VisualStateGroup>
        </VisualStateManager.VisualStateGroups>
        
        <ContentPresenter HorizontalAlignment="Center" VerticalAlignment="Center"/>
    </Border>
</ControlTemplate>
```

### Переключение состояний из кода

```csharp
// Переключить состояние
VisualStateManager.GoToState(myButton, "Pressed", true);

// Или для элемента внутри шаблона
VisualStateManager.GoToElementState(border, "MouseOver", true);
```

---

## Анимации и переходы

### Зачем нужны анимации

Анимации — не просто украшение:

1. **Обратная связь** — пользователь видит, что действие сработало
2. **Контекст** — показывает, откуда что появилось
3. **Естественность** — реальные объекты не меняются мгновенно
4. **Внимание** — привлекает к важным изменениям

### Типы анимаций

| Анимация | Что анимирует |
|----------|---------------|
| DoubleAnimation | Числовые свойства (Width, Opacity) |
| ColorAnimation | Цвета |
| ThicknessAnimation | Margin, Padding, BorderThickness |
| PointAnimation | Координаты |

### Пример: Плавное появление

```xml
<Window.Resources>
    <Storyboard x:Key="FadeIn">
        <DoubleAnimation 
            Storyboard.TargetProperty="Opacity"
            From="0" To="1" 
            Duration="0:0:0.3"/>
    </Storyboard>
</Window.Resources>

<Border x:Name="ContentBorder" Opacity="0">
    <!-- Контент -->
</Border>
```

```csharp
// Запустить анимацию
var storyboard = (Storyboard)FindResource("FadeIn");
storyboard.Begin(ContentBorder);
```

### Easing Functions — естественность движения

По умолчанию анимация линейная. Easing Functions делают её естественнее:

```xml
<DoubleAnimation To="200" Duration="0:0:0.5">
    <DoubleAnimation.EasingFunction>
        <!-- Замедление в конце -->
        <CubicEase EasingMode="EaseOut"/>
    </DoubleAnimation.EasingFunction>
</DoubleAnimation>
```

| Easing | Эффект |
|--------|--------|
| **EaseIn** | Медленно начинает, быстро заканчивает |
| **EaseOut** | Быстро начинает, медленно заканчивает |
| **EaseInOut** | Медленно → быстро → медленно |

---

## Accessibility — доступность

### Почему состояния важны для доступности

Screen readers (программы для незрячих) полагаются на состояния элементов:

- `IsEnabled=False` → "Кнопка неактивна"
- `IsSelected=True` → "Выбрано"
- `IsExpanded=True` → "Развёрнуто"

### Рекомендации

1. **Не полагайтесь только на цвет**
   ```xml
   <!-- Плохо: только цвет показывает ошибку -->
   <TextBox BorderBrush="Red"/>
   
   <!-- Хорошо: цвет + иконка + текст -->
   <StackPanel>
       <TextBox BorderBrush="Red"/>
       <StackPanel Orientation="Horizontal">
           <Image Source="error-icon.png"/>
           <TextBlock Text="Обязательное поле"/>
       </StackPanel>
   </StackPanel>
   ```

2. **Focus visible** — выделяйте фокус клавиатуры
   ```xml
   <Trigger Property="IsFocused" Value="True">
       <Setter Property="BorderBrush" Value="#3498db"/>
       <Setter Property="BorderThickness" Value="2"/>
   </Trigger>
   ```

3. **Достаточная область клика** — минимум 44x44 пикселей

---

## Практические примеры

### Пример 1: Карточка с hover-эффектом

```xml
<Style x:Key="HoverCard" TargetType="Border">
    <Setter Property="Background" Value="White"/>
    <Setter Property="BorderBrush" Value="#e0e0e0"/>
    <Setter Property="BorderThickness" Value="1"/>
    <Setter Property="CornerRadius" Value="8"/>
    <Setter Property="Padding" Value="16"/>
    <Setter Property="Effect">
        <Setter.Value>
            <DropShadowEffect BlurRadius="4" ShadowDepth="1" Opacity="0.1"/>
        </Setter.Value>
    </Setter>
    
    <Style.Triggers>
        <Trigger Property="IsMouseOver" Value="True">
            <Setter Property="Effect">
                <Setter.Value>
                    <DropShadowEffect BlurRadius="16" ShadowDepth="4" Opacity="0.2"/>
                </Setter.Value>
            </Setter>
            <Setter Property="RenderTransform">
                <Setter.Value>
                    <TranslateTransform Y="-2"/>
                </Setter.Value>
            </Setter>
        </Trigger>
    </Style.Triggers>
</Style>
```

### Пример 2: Место в зале с состояниями

```xml
<Style x:Key="SeatStyle" TargetType="Border">
    <Setter Property="Width" Value="36"/>
    <Setter Property="Height" Value="36"/>
    <Setter Property="CornerRadius" Value="4"/>
    <Setter Property="Margin" Value="2"/>
    <Setter Property="Cursor" Value="Hand"/>
    
    <Style.Triggers>
        <!-- Свободное место -->
        <DataTrigger Binding="{Binding IsOccupied}" Value="False">
            <Setter Property="Background" Value="#e8f5e9"/>
            <Setter Property="BorderBrush" Value="#81c784"/>
        </DataTrigger>
        
        <!-- Занятое место -->
        <DataTrigger Binding="{Binding IsOccupied}" Value="True">
            <Setter Property="Background" Value="#ffebee"/>
            <Setter Property="BorderBrush" Value="#e57373"/>
            <Setter Property="Cursor" Value="Arrow"/>
        </DataTrigger>
        
        <!-- Выбранное место -->
        <DataTrigger Binding="{Binding IsSelected}" Value="True">
            <Setter Property="Background" Value="#bbdefb"/>
            <Setter Property="BorderBrush" Value="#42a5f5"/>
        </DataTrigger>
        
        <!-- Hover для свободных -->
        <MultiDataTrigger>
            <MultiDataTrigger.Conditions>
                <Condition Binding="{Binding IsOccupied}" Value="False"/>
                <Condition Binding="{Binding RelativeSource={RelativeSource Self}, Path=IsMouseOver}" Value="True"/>
            </MultiDataTrigger.Conditions>
            <MultiDataTrigger.Setters>
                <Setter Property="Background" Value="#c8e6c9"/>
            </MultiDataTrigger.Setters>
        </MultiDataTrigger>
    </Style.Triggers>
</Style>
```

### Пример 3: Кнопка загрузки

```xml
<Style x:Key="LoadingButton" TargetType="Button">
    <Setter Property="Template">
        <Setter.Value>
            <ControlTemplate TargetType="Button">
                <Grid>
                    <Border x:Name="MainBorder" 
                            Background="{TemplateBinding Background}"
                            CornerRadius="4">
                        <Grid>
                            <!-- Обычный контент -->
                            <ContentPresenter x:Name="Content" 
                                              HorizontalAlignment="Center"/>
                            
                            <!-- Индикатор загрузки (скрыт по умолчанию) -->
                            <StackPanel x:Name="LoadingPanel" 
                                        Orientation="Horizontal" 
                                        Visibility="Collapsed"
                                        HorizontalAlignment="Center">
                                <Ellipse Width="8" Height="8" Fill="White" Margin="2"/>
                                <Ellipse Width="8" Height="8" Fill="White" Margin="2"/>
                                <Ellipse Width="8" Height="8" Fill="White" Margin="2"/>
                            </StackPanel>
                        </Grid>
                    </Border>
                </Grid>
                
                <ControlTemplate.Triggers>
                    <Trigger Property="Tag" Value="Loading">
                        <Setter TargetName="Content" Property="Visibility" Value="Collapsed"/>
                        <Setter TargetName="LoadingPanel" Property="Visibility" Value="Visible"/>
                        <Setter Property="IsEnabled" Value="False"/>
                    </Trigger>
                </ControlTemplate.Triggers>
            </ControlTemplate>
        </Setter.Value>
    </Setter>
</Style>
```

```csharp
// Использование
BuyButton.Tag = "Loading";  // Показать индикатор
await ProcessPayment();
BuyButton.Tag = null;       // Вернуть обычное состояние
```

---

## Связь с лабораторной работой

### Лаба 5: Темы и состояния

В лабораторной работе вы:

1. **Создадите стили кнопок** с hover/pressed состояниями
2. **Реализуете переключение темы** через замену ResourceDictionary
3. **Стилизуете места в зале** — свободное/занятое/выбранное

---

## Заключение

**Триггеры и состояния** — это способ сделать UI **живым и отзывчивым**:

- **Триггеры** — декларативный способ менять свойства
- **Visual States** — организованное управление состояниями
- **Анимации** — плавные переходы между состояниями
- **Accessibility** — состояния помогают всем пользователям

Хороший UI реагирует на действия пользователя мгновенно и предсказуемо.

---

## Полезные ресурсы

- [Triggers in WPF](https://docs.microsoft.com/en-us/dotnet/desktop/wpf/controls/styles-templates-overview)
- [VisualStateManager](https://docs.microsoft.com/en-us/dotnet/api/system.windows.visualstatemanager)
- [Animation Overview](https://docs.microsoft.com/en-us/dotnet/desktop/wpf/graphics-multimedia/animation-overview)
- [Easing Functions](https://easings.net/) — визуализация функций
