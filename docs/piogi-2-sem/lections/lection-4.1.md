---
title: Лекция 4.1 - Шаблоны и визуальное дерево WPF
doc-type:
  - lection
doc-status:
  - in_work
---

# Шаблоны и визуальное дерево WPF

## Введение

Когда вы создаёте кнопку в WPF, она выглядит определённым образом: прямоугольник с текстом, который меняет цвет при наведении. Но откуда WPF знает, **как** должна выглядеть кнопка?

Ответ: **шаблоны (Templates)**. Каждый элемент управления имеет шаблон, описывающий его внешний вид. И вы можете полностью его изменить!

---

## Два дерева WPF

### Logical Tree (Логическое дерево)

**Логическое дерево** — структура элементов, которую вы описываете в XAML:

```xml
<Window>
    <Grid>
        <StackPanel>
            <TextBlock Text="Заголовок"/>
            <Button Content="OK"/>
        </StackPanel>
    </Grid>
</Window>
```

```
Window
  └── Grid
        └── StackPanel
              ├── TextBlock
              └── Button
```

Это то, что вы видите и контролируете как разработчик.

### Visual Tree (Визуальное дерево)

**Визуальное дерево** — реальная структура элементов, которая используется для отрисовки. Она значительно сложнее:

```
Window
  └── Border
        └── AdornerDecorator
              └── ContentPresenter
                    └── Grid
                          └── StackPanel
                                ├── TextBlock
                                │     └── (внутренние элементы)
                                └── Button
                                      └── ButtonChrome
                                            └── ContentPresenter
                                                  └── TextBlock ("OK")
```

Кнопка в логическом дереве — один элемент. В визуальном дереве — несколько: `ButtonChrome`, `ContentPresenter`, `TextBlock`...

### Почему это важно

1. **Стилизация** — чтобы изменить внешний вид, нужно понимать визуальное дерево
2. **Производительность** — больше элементов в визуальном дереве = медленнее отрисовка
3. **Отладка** — инструменты показывают визуальное дерево

### Инструменты для просмотра

**Snoop** — бесплатный инструмент для инспекции WPF-приложений:
- Показывает оба дерева
- Позволяет изменять свойства в реальном времени
- Помогает отлаживать Binding

**Visual Studio Live Visual Tree** — встроенный инструмент (Debug → Windows → Live Visual Tree)

---

## DataTemplate — шаблон данных

### Проблема

Когда вы добавляете объект в ListBox, WPF не знает, как его отобразить:

```csharp
public class Movie
{
    public string Title { get; set; }
    public string PosterUrl { get; set; }
    public int Duration { get; set; }
}

// Что увидит пользователь?
listBox.Items.Add(new Movie { Title = "Мстители" });
```

По умолчанию WPF вызывает `ToString()`:

```
CinemaKiosk.Models.Movie
```

Не очень полезно.

### Решение: DataTemplate

**DataTemplate** описывает, как отображать объект определённого типа:

```xml
<DataTemplate x:Key="MovieTemplate">
    <Border BorderBrush="Gray" BorderThickness="1" Margin="5" Padding="10">
        <StackPanel Orientation="Horizontal">
            <Image Source="{Binding PosterUrl}" Width="50" Height="75"/>
            <StackPanel Margin="10,0,0,0">
                <TextBlock Text="{Binding Title}" FontWeight="Bold"/>
                <TextBlock Text="{Binding Duration, StringFormat='{}{0} мин'}"/>
            </StackPanel>
        </StackPanel>
    </Border>
</DataTemplate>
```

```xml
<ListBox ItemsSource="{Binding Movies}" 
         ItemTemplate="{StaticResource MovieTemplate}"/>
```

Теперь каждый `Movie` отображается красивой карточкой!

### Как работает DataTemplate

```
┌─────────────────────────────────────────────────┐
│                  ListBox                         │
│  ┌───────────────────────────────────────────┐  │
│  │ DataTemplate применяется к каждому Movie  │  │
│  │                                           │  │
│  │  ┌─────────────────────────────────────┐  │  │
│  │  │ [Poster] Title                      │  │  │
│  │  │          Duration                   │  │  │
│  │  └─────────────────────────────────────┘  │  │
│  │  ┌─────────────────────────────────────┐  │  │
│  │  │ [Poster] Title                      │  │  │
│  │  │          Duration                   │  │  │
│  │  └─────────────────────────────────────┘  │  │
│  └───────────────────────────────────────────┘  │
└─────────────────────────────────────────────────┘
```

DataTemplate **не изменяет** сам элемент управления (ListBox). Он описывает, как отображать **каждый элемент данных** внутри.

### DataType — автоматическое применение

Можно привязать шаблон к типу данных:

```xml
<Window.Resources>
    <DataTemplate DataType="{x:Type local:Movie}">
        <!-- Этот шаблон применится ко всем Movie автоматически -->
        <TextBlock Text="{Binding Title}"/>
    </DataTemplate>
</Window.Resources>
```

Теперь везде, где WPF встретит объект `Movie`, он применит этот шаблон.

---

## ControlTemplate — шаблон элемента управления

### Чем отличается от DataTemplate

- **DataTemplate** — как отображать **данные** (объекты)
- **ControlTemplate** — как выглядит **сам элемент управления** (кнопка, текстбокс)

### Пример: Кастомная кнопка

По умолчанию кнопка — серый прямоугольник. Давайте сделаем круглую:

```xml
<Style x:Key="CircleButton" TargetType="Button">
    <Setter Property="Template">
        <Setter.Value>
            <ControlTemplate TargetType="Button">
                <Grid>
                    <!-- Круг вместо прямоугольника -->
                    <Ellipse Fill="{TemplateBinding Background}"
                             Stroke="{TemplateBinding BorderBrush}"
                             StrokeThickness="2"/>
                    
                    <!-- Содержимое кнопки -->
                    <ContentPresenter HorizontalAlignment="Center"
                                      VerticalAlignment="Center"/>
                </Grid>
            </ControlTemplate>
        </Setter.Value>
    </Setter>
</Style>
```

```xml
<Button Style="{StaticResource CircleButton}" 
        Content="+" 
        Width="50" Height="50"
        Background="LightBlue"/>
```

Результат: круглая кнопка с текстом "+" в центре.

### TemplateBinding

**TemplateBinding** — связывает свойства шаблона со свойствами элемента:

```xml
<Ellipse Fill="{TemplateBinding Background}"/>
```

Это значит: "Используй значение `Background` кнопки как заливку эллипса."

Так внешний код может настраивать внешний вид:

```xml
<Button Style="{StaticResource CircleButton}" Background="Red"/>
<!-- Эллипс будет красным -->

<Button Style="{StaticResource CircleButton}" Background="Green"/>
<!-- Эллипс будет зелёным -->
```

### ContentPresenter

**ContentPresenter** — место, куда вставляется содержимое элемента:

```xml
<Button Content="Текст кнопки">
```

В шаблоне `ContentPresenter` отображает это "Текст кнопки".

Без `ContentPresenter` содержимое не появится:

```xml
<ControlTemplate TargetType="Button">
    <Ellipse Fill="Blue"/>
    <!-- Где текст? Его нет, потому что нет ContentPresenter -->
</ControlTemplate>
```

---

## Виртуализация списков

### Проблема больших списков

Представьте ListBox с 10 000 элементов. Если создать визуальные элементы для каждого:

- 10 000 TextBlock
- 10 000 Border
- ...

Это **очень медленно** и **жрёт память**.

### Решение: Виртуализация

WPF создаёт визуальные элементы **только для видимых** строк:

```
┌─────────────────────────────────────────┐
│  Элемент 0  ←── Реально существует      │
│  Элемент 1  ←── Реально существует      │
│  Элемент 2  ←── Реально существует      │
│  Элемент 3  ←── Реально существует      │
│  Элемент 4  ←── Реально существует      │
└─────────────────────────────────────────┘
   Элемент 5      (не создан)
   Элемент 6      (не создан)
   ...
   Элемент 9999   (не создан)
```

При прокрутке элементы **переиспользуются**:

```
Прокрутка вниз:
  Элемент 0 "уезжает" наверх → становится Элементом 5
  (меняется только DataContext, визуальный элемент тот же)
```

### Как включить виртуализацию

Для `ListBox` и `ListView` виртуализация **включена по умолчанию**.

Для других контейнеров:

```xml
<ItemsControl VirtualizingPanel.IsVirtualizing="True"
              VirtualizingPanel.VirtualizationMode="Recycling">
    <ItemsControl.ItemsPanel>
        <ItemsPanelTemplate>
            <VirtualizingStackPanel/>
        </ItemsPanelTemplate>
    </ItemsControl.ItemsPanel>
</ItemsControl>
```

### Когда виртуализация ломается

1. **ScrollViewer вокруг ItemsControl:**
   ```xml
   <!-- ПЛОХО: виртуализация не работает -->
   <ScrollViewer>
       <ListBox ItemsSource="{Binding HugeList}"/>
   </ScrollViewer>
   ```
   ListBox думает, что может занять бесконечную высоту, и создаёт все элементы.

2. **Неправильный ItemsPanel:**
   ```xml
   <!-- ПЛОХО: WrapPanel не поддерживает виртуализацию -->
   <ListBox.ItemsPanel>
       <ItemsPanelTemplate>
           <WrapPanel/>
       </ItemsPanelTemplate>
   </ListBox.ItemsPanel>
   ```

---

## Dependency Properties

### Обычные свойства vs Dependency Properties

**Обычное свойство C#:**
```csharp
public class Person
{
    public string Name { get; set; }  // Просто хранит значение
}
```

**Dependency Property:**
```csharp
public static readonly DependencyProperty NameProperty =
    DependencyProperty.Register(
        "Name",
        typeof(string),
        typeof(Person),
        new PropertyMetadata("Default"));

public string Name
{
    get => (string)GetValue(NameProperty);
    set => SetValue(NameProperty, value);
}
```

### Зачем нужны Dependency Properties

1. **Data Binding** — работает только с DP
2. **Анимации** — можно анимировать только DP
3. **Стили** — можно устанавливать через Style/Setter
4. **Наследование значений** — дочерние элементы могут наследовать
5. **Значения по умолчанию** — задаются в метаданных

### Attached Properties

**Attached Properties** — DP, которые можно "прикреплять" к чужим элементам:

```xml
<Grid>
    <Button Grid.Row="1" Grid.Column="2"/>
</Grid>
```

`Grid.Row` и `Grid.Column` — это attached properties класса `Grid`, прикреплённые к `Button`.

Как определить:

```csharp
public static readonly DependencyProperty RowProperty =
    DependencyProperty.RegisterAttached(
        "Row",
        typeof(int),
        typeof(Grid),
        new PropertyMetadata(0));

public static int GetRow(DependencyObject obj)
{
    return (int)obj.GetValue(RowProperty);
}

public static void SetRow(DependencyObject obj, int value)
{
    obj.SetValue(RowProperty, value);
}
```

---

## Наследование и каскадность

### Наследование значений свойств

Некоторые свойства наследуются от родителя к детям:

```xml
<StackPanel FontSize="20" Foreground="Blue">
    <TextBlock Text="Первый"/>   <!-- FontSize=20, Foreground=Blue -->
    <TextBlock Text="Второй"/>   <!-- FontSize=20, Foreground=Blue -->
    <TextBlock Text="Третий" FontSize="14"/>  <!-- FontSize=14, Foreground=Blue -->
</StackPanel>
```

`FontSize` и `Foreground` наследуются. Но можно переопределить.

### Приоритет значений свойств

WPF имеет строгий порядок приоритетов (от высшего к низшему):

1. **Локальное значение** — `<Button Background="Red"/>`
2. **Триггеры стилей**
3. **Триггеры шаблонов**
4. **Сеттеры стилей** — `<Setter Property="Background" Value="Red"/>`
5. **Унаследованное значение**
6. **Значение по умолчанию**

### Аналогия с CSS

Если знаете CSS, это похоже на специфичность селекторов:

| WPF | CSS |
|-----|-----|
| Локальное значение | inline style |
| Стиль с x:Key | class selector |
| Стиль по TargetType | element selector |
| Наследование | inheritance |
| Значение по умолчанию | browser default |

---

## Практические примеры

### Пример 1: Кастомный ListBox с карточками

```xml
<ListBox ItemsSource="{Binding Movies}">
    <!-- Меняем контейнер на WrapPanel -->
    <ListBox.ItemsPanel>
        <ItemsPanelTemplate>
            <WrapPanel/>
        </ItemsPanelTemplate>
    </ListBox.ItemsPanel>
    
    <!-- Шаблон карточки -->
    <ListBox.ItemTemplate>
        <DataTemplate>
            <Border Width="150" Height="220" Margin="10"
                    BorderBrush="Gray" BorderThickness="1" CornerRadius="5">
                <Grid>
                    <Grid.RowDefinitions>
                        <RowDefinition Height="*"/>
                        <RowDefinition Height="Auto"/>
                    </Grid.RowDefinitions>
                    
                    <Image Source="{Binding PosterUrl}" Stretch="UniformToFill"/>
                    
                    <Border Grid.Row="1" Background="#80000000" Padding="5">
                        <TextBlock Text="{Binding Title}" 
                                   Foreground="White" 
                                   TextWrapping="Wrap"/>
                    </Border>
                </Grid>
            </Border>
        </DataTemplate>
    </ListBox.ItemTemplate>
    
    <!-- Убираем выделение по умолчанию -->
    <ListBox.ItemContainerStyle>
        <Style TargetType="ListBoxItem">
            <Setter Property="Template">
                <Setter.Value>
                    <ControlTemplate TargetType="ListBoxItem">
                        <ContentPresenter/>
                    </ControlTemplate>
                </Setter.Value>
            </Setter>
        </Style>
    </ListBox.ItemContainerStyle>
</ListBox>
```

### Пример 2: Индикатор загрузки

```xml
<ControlTemplate x:Key="LoadingIndicator">
    <Grid>
        <Ellipse Width="50" Height="50" 
                 Stroke="DodgerBlue" StrokeThickness="4"
                 StrokeDashArray="3,2">
            <Ellipse.RenderTransform>
                <RotateTransform x:Name="RotateTransform"/>
            </Ellipse.RenderTransform>
            <Ellipse.Triggers>
                <EventTrigger RoutedEvent="Loaded">
                    <BeginStoryboard>
                        <Storyboard RepeatBehavior="Forever">
                            <DoubleAnimation 
                                Storyboard.TargetName="RotateTransform"
                                Storyboard.TargetProperty="Angle"
                                From="0" To="360" 
                                Duration="0:0:1"/>
                        </Storyboard>
                    </BeginStoryboard>
                </EventTrigger>
            </Ellipse.Triggers>
        </Ellipse>
    </Grid>
</ControlTemplate>
```

---

## Связь с лабораторной работой

### Лаба 3: DataTemplate

В лабораторной работе вы создадите:

1. **DataTemplate для карточки фильма:**
   ```xml
   <DataTemplate x:Key="MovieCardTemplate">
       <Border>
           <Grid>
               <Image Source="{Binding PosterUrl}"/>
               <TextBlock Text="{Binding Title}"/>
           </Grid>
       </Border>
   </DataTemplate>
   ```

2. **Панель деталей с Binding к SelectedItem:**
   ```xml
   <StackPanel DataContext="{Binding SelectedItem, ElementName=MoviesListBox}">
       <TextBlock Text="{Binding Title}"/>
       <Image Source="{Binding PosterUrl}"/>
   </StackPanel>
   ```

---

## Заключение

**Шаблоны** — мощнейший инструмент WPF:

- **DataTemplate** — как отображать данные
- **ControlTemplate** — как выглядит элемент управления
- **ItemsPanelTemplate** — как располагаются элементы в коллекции

Понимание **визуального дерева** помогает:
- Отлаживать проблемы отображения
- Оптимизировать производительность
- Создавать сложные кастомные элементы

---

## Полезные ресурсы

- [Data Templating Overview](https://docs.microsoft.com/en-us/dotnet/desktop/wpf/data/data-templating-overview)
- [Control Templates](https://docs.microsoft.com/en-us/dotnet/desktop/wpf/controls/how-to-create-apply-template)
- [Snoop WPF](https://github.com/snoopwpf/snoopwpf) — инструмент для инспекции
