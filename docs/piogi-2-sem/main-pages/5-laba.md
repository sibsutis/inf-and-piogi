---
title: Лаба 5
tags:
  - styles
  - resources
  - themes
  - xaml
---

## Вступление

Данная лабораторная работа посвящена стилизации приложения: вынос стилей в ресурсы, создание тем оформления.

**Что будем делать:**

- Вынос стилей в `App.xaml`
- Создание `ResourceDictionary`
- Оформление кнопок (скругления, тени)
- Реализация тёмной темы

У тебя уже есть работающее, но страшное приложение с реальными данными. Пора наводить красоту. Рефакторить *работающий* код приятнее!

---

## Техническое задание

[Требования к программе, необходимые для сдачи.](../tasks/task5.md)

---

## Git-воркфлоу

```bash
git checkout -b feature/lab-5-styles
```

---

## Шаг 1: Создание ResourceDictionary

1. **ПКМ по проекту** → **Add** → **New Item** → **Resource Dictionary (WPF)**
2. Название: `Styles.xaml`

Подключаем словарь в `App.xaml`:

```xml
<Application x:Class="CinemaKiosk.App"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             StartupUri="MainWindow.xaml">
    <Application.Resources>
        <ResourceDictionary>
            <ResourceDictionary.MergedDictionaries>
                <ResourceDictionary Source="Styles.xaml"/>
            </ResourceDictionary.MergedDictionaries>
        </ResourceDictionary>
    </Application.Resources>
</Application>
```

---

## Шаг 2: Базовые стили для кнопок

В `Styles.xaml`:

```xml
<ResourceDictionary xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
                    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
    
    <!-- Основная кнопка -->
    <Style x:Key="PrimaryButton" TargetType="Button">
        <Setter Property="Background" Value="#007ACC"/>
        <Setter Property="Foreground" Value="White"/>
        <Setter Property="FontSize" Value="14"/>
        <Setter Property="Padding" Value="20,10"/>
        <Setter Property="BorderThickness" Value="0"/>
        <Setter Property="Cursor" Value="Hand"/>
        <Setter Property="Template">
            <Setter.Value>
                <ControlTemplate TargetType="Button">
                    <Border Background="{TemplateBinding Background}"
                            CornerRadius="5"
                            Padding="{TemplateBinding Padding}">
                        <ContentPresenter HorizontalAlignment="Center" 
                                          VerticalAlignment="Center"/>
                    </Border>
                </ControlTemplate>
            </Setter.Value>
        </Setter>
        <Style.Triggers>
            <Trigger Property="IsMouseOver" Value="True">
                <Setter Property="Background" Value="#005A9E"/>
            </Trigger>
            <Trigger Property="IsPressed" Value="True">
                <Setter Property="Background" Value="#004080"/>
            </Trigger>
        </Style.Triggers>
    </Style>
    
    <!-- Второстепенная кнопка -->
    <Style x:Key="SecondaryButton" TargetType="Button">
        <Setter Property="Background" Value="Transparent"/>
        <Setter Property="Foreground" Value="#007ACC"/>
        <Setter Property="BorderBrush" Value="#007ACC"/>
        <Setter Property="BorderThickness" Value="1"/>
        <Setter Property="Padding" Value="20,10"/>
        <Setter Property="Cursor" Value="Hand"/>
        <Setter Property="Template">
            <Setter.Value>
                <ControlTemplate TargetType="Button">
                    <Border Background="{TemplateBinding Background}"
                            BorderBrush="{TemplateBinding BorderBrush}"
                            BorderThickness="{TemplateBinding BorderThickness}"
                            CornerRadius="5"
                            Padding="{TemplateBinding Padding}">
                        <ContentPresenter HorizontalAlignment="Center" 
                                          VerticalAlignment="Center"/>
                    </Border>
                </ControlTemplate>
            </Setter.Value>
        </Setter>
    </Style>
    
</ResourceDictionary>
```

---

## Шаг 3: Применение стилей

Используем стили в XAML:

```xml
<Button Content="Купить билеты" Style="{StaticResource PrimaryButton}"/>
<Button Content="Отмена" Style="{StaticResource SecondaryButton}"/>
```

---

## Шаг 4: Цветовые переменные

Добавляем цвета как ресурсы для переиспользования:

```xml
<!-- Цвета приложения -->
<Color x:Key="PrimaryColor">#007ACC</Color>
<Color x:Key="PrimaryDarkColor">#005A9E</Color>
<Color x:Key="BackgroundColor">#F5F5F5</Color>
<Color x:Key="TextColor">#333333</Color>

<SolidColorBrush x:Key="PrimaryBrush" Color="{StaticResource PrimaryColor}"/>
<SolidColorBrush x:Key="BackgroundBrush" Color="{StaticResource BackgroundColor}"/>
<SolidColorBrush x:Key="TextBrush" Color="{StaticResource TextColor}"/>
```

---

## Шаг 5: Создание тёмной темы

Создаём два файла: `LightTheme.xaml` и `DarkTheme.xaml`:

**LightTheme.xaml:**
```xml
<ResourceDictionary xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
                    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
    
    <Style x:Key="WindowStyle" TargetType="Window">
        <Setter Property="Background" Value="#FFFFFF"/>
    </Style>
    
    <SolidColorBrush x:Key="BackgroundBrush" Color="#FFFFFF"/>
    <SolidColorBrush x:Key="CardBrush" Color="#F5F5F5"/>
    <SolidColorBrush x:Key="TextBrush" Color="#333333"/>
    
</ResourceDictionary>
```

**DarkTheme.xaml:**
```xml
<ResourceDictionary xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
                    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
    
    <Style x:Key="WindowStyle" TargetType="Window">
        <Setter Property="Background" Value="#1E1E1E"/>
    </Style>
    
    <SolidColorBrush x:Key="BackgroundBrush" Color="#1E1E1E"/>
    <SolidColorBrush x:Key="CardBrush" Color="#2D2D2D"/>
    <SolidColorBrush x:Key="TextBrush" Color="#FFFFFF"/>
    
</ResourceDictionary>
```

---

## Шаг 6: Переключение темы

В `MainWindow.xaml` применяем стиль:

```xml
<Window Style="{DynamicResource WindowStyle}" ...>
```

!!! note "DynamicResource vs StaticResource"
    `DynamicResource` позволяет менять ресурс во время работы программы.

В коде переключения темы:

```csharp
private bool _isLightTheme = true;

private void ToggleTheme_Click(object sender, RoutedEventArgs e)
{
    var newThemeUri = _isLightTheme ? "DarkTheme.xaml" : "LightTheme.xaml";
    
    var newTheme = new ResourceDictionary
    {
        Source = new Uri(newThemeUri, UriKind.Relative)
    };
    
    // Удаляем старую тему
    var oldTheme = Application.Current.Resources.MergedDictionaries
        .FirstOrDefault(d => d.Source != null && 
            (d.Source.OriginalString.Contains("LightTheme") || 
             d.Source.OriginalString.Contains("DarkTheme")));
    
    if (oldTheme != null)
    {
        Application.Current.Resources.MergedDictionaries.Remove(oldTheme);
    }
    
    // Добавляем новую тему
    Application.Current.Resources.MergedDictionaries.Add(newTheme);
    
    _isLightTheme = !_isLightTheme;
}
```

---

## Шаг 7: Стили для карточек

Добавляем стиль для карточек фильмов:

```xml
<Style x:Key="MovieCard" TargetType="Border">
    <Setter Property="Background" Value="{DynamicResource CardBrush}"/>
    <Setter Property="CornerRadius" Value="10"/>
    <Setter Property="Padding" Value="15"/>
    <Setter Property="Margin" Value="5"/>
    <Setter Property="Effect">
        <Setter.Value>
            <DropShadowEffect BlurRadius="10" 
                              ShadowDepth="2" 
                              Opacity="0.2"/>
        </Setter.Value>
    </Setter>
</Style>
```

---

## Результат

После выполнения:

1. Все стили вынесены в ResourceDictionary
2. Нет inline-стилей в XAML-элементах
3. Приложение поддерживает светлую и тёмную тему
4. Кнопки имеют скругления, hover-эффекты, тени

---

## Коммит и Push

```bash
git add .
git commit -m "style: add themes and button styles"
git push origin feature/lab-5-styles
```

---

## Полезные ссылки

- [Создание и описание стиля](../topics/styles.md)
- [ResourceDictionary](https://docs.microsoft.com/en-us/dotnet/desktop/wpf/fundamentals/xaml-resources-define)
- [ControlTemplate](https://docs.microsoft.com/en-us/dotnet/desktop/wpf/controls/how-to-create-apply-template)

