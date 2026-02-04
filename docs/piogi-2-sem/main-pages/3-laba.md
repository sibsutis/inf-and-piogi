---
title: Лаба 3
tags:
  - binding
  - datatemplate
  - images
  - xaml
---

## Вступление

Данная лабораторная работа посвящена превращению убогих строк в красивые карточки с использованием Data Binding и DataTemplate.

**Что будем делать:**

- Создание `DataTemplate` для карточек фильмов
- Загрузка изображений по URL
- Привязка выбранного элемента к детальной панели
- Работа с `Binding` и `DataContext`

После этой лабы вместо строк `ToString()` у тебя будут красивые карточки с постерами.

---

## Техническое задание

[Требования к программе, необходимые для сдачи.](../tasks/task3.md)

---

## Git-воркфлоу

```bash
git checkout -b feature/lab-3-binding
```

---

## Шаг 1: Создание DataTemplate для карточки

В `MainWindow.xaml` добавляем шаблон для отображения фильма:

```xml
<Window.Resources>
    <DataTemplate x:Key="MovieCardTemplate">
        <Border BorderBrush="Gray" BorderThickness="1" Margin="5" Padding="10">
            <Grid>
                <Grid.ColumnDefinitions>
                    <ColumnDefinition Width="80"/>
                    <ColumnDefinition Width="*"/>
                </Grid.ColumnDefinitions>
                
                <!-- Постер -->
                <Image Source="{Binding PosterUrl}" 
                       Width="70" Height="100" 
                       Stretch="UniformToFill"/>
                
                <!-- Информация -->
                <StackPanel Grid.Column="1" Margin="10,0,0,0">
                    <TextBlock Text="{Binding Title}" 
                               FontWeight="Bold" 
                               FontSize="14"/>
                    <TextBlock Text="{Binding Duration, StringFormat='{}{0} мин'}" 
                               Foreground="Gray"/>
                </StackPanel>
            </Grid>
        </Border>
    </DataTemplate>
</Window.Resources>
```

---

## Шаг 2: Применение шаблона к ListBox

Обновляем `ListBox`, чтобы он использовал наш шаблон:

```xml
<ListBox x:Name="MoviesListBox" 
         Grid.Column="0"
         ItemTemplate="{StaticResource MovieCardTemplate}"
         SelectionChanged="MoviesListBox_SelectionChanged"/>
```

Теперь вместо строк будут отображаться карточки с постерами!

---

## Шаг 3: Панель деталей фильма

Заменяем заглушку справа на панель с детальной информацией:

```xml
<!-- Правая панель: детали фильма -->
<Border Grid.Column="1" Padding="20">
    <StackPanel x:Name="DetailsPanel" 
                DataContext="{Binding SelectedItem, ElementName=MoviesListBox}">
        
        <!-- Заголовок -->
        <TextBlock Text="{Binding Title}" 
                   FontSize="24" 
                   FontWeight="Bold"/>
        
        <!-- Большой постер -->
        <Image Source="{Binding PosterUrl}" 
               MaxHeight="300" 
               Margin="0,10"/>
        
        <!-- Описание -->
        <TextBlock Text="{Binding Description}" 
                   TextWrapping="Wrap"/>
        
        <!-- Длительность -->
        <TextBlock Margin="0,10">
            <Run Text="Длительность: "/>
            <Run Text="{Binding Duration}"/>
            <Run Text=" мин"/>
        </TextBlock>
        
    </StackPanel>
</Border>
```

!!! note "Binding к SelectedItem"
    `DataContext="{Binding SelectedItem, ElementName=MoviesListBox}"` автоматически обновляет панель при выборе фильма.

---

## Шаг 4: Асинхронная загрузка изображений

WPF автоматически загружает изображения по URL, но для лучшего UX можно добавить placeholder:

```xml
<Image>
    <Image.Style>
        <Style TargetType="Image">
            <Setter Property="Source" Value="{Binding PosterUrl}"/>
            <Style.Triggers>
                <Trigger Property="Source" Value="{x:Null}">
                    <Setter Property="Source" Value="/Assets/placeholder.png"/>
                </Trigger>
            </Style.Triggers>
        </Style>
    </Image.Style>
</Image>
```

---

## Шаг 5: StringFormat для форматирования

Используй `StringFormat` для красивого отображения данных:

```xml
<!-- Длительность в часах и минутах -->
<TextBlock Text="{Binding Duration, StringFormat='Длительность: {0} мин'}"/>

<!-- Дата в нужном формате -->
<TextBlock Text="{Binding ReleaseDate, StringFormat='dd.MM.yyyy'}"/>

<!-- Цена с валютой -->
<TextBlock Text="{Binding Price, StringFormat='{}{0:N0} ₽'}"/>
```

[Подробнее про StringFormat](../topics/stringformat.md)

---

## Результат

После выполнения всех шагов:

1. Слева — список карточек с постерами
2. При клике на карточку — справа появляются детали
3. Всё работает автоматически через Binding

---

## Возможные проблемы

### Изображения не загружаются
- Проверь URL (должен быть валидный и доступный)
- Проверь HTTPS (некоторые HTTP-ссылки блокируются)

### Binding не работает
- Проверь `DataContext`
- Проверь названия свойств (регистр важен!)
- Убедись, что свойства публичные (`public`)

---

## Коммит и Push

```bash
git add .
git commit -m "feat: add DataTemplate and detail panel binding"
git push origin feature/lab-3-binding
```

---

## Полезные ссылки

- [Data Binding Overview](https://docs.microsoft.com/en-us/dotnet/desktop/wpf/data/data-binding-overview)
- [DataTemplate](../topics/binding.md)
- [StringFormat](../topics/stringformat.md)
- [DataContext](../topics/datacontext.md)

