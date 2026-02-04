---
title: Лекция 5.2 - Системы стилей и дизайн-системы
---

# Системы стилей и дизайн-системы

## Введение

Почему все приложения Google выглядят "по-гугловски"? Почему продукты Apple узнаваемы с первого взгляда? Почему интерфейс Windows отличается от macOS?

Ответ: **дизайн-системы** — наборы правил, компонентов и стилей, которые обеспечивают единообразие продуктов.

---

## Проблемы без системы стилей

### Типичная ситуация

Проект начинается. Разработчик создаёт кнопку:

```xml
<Button Background="#3498db" Foreground="White" Padding="10,5"/>
```

Через неделю другой разработчик создаёт ещё одну:

```xml
<Button Background="#2980b9" Foreground="White" Padding="15,8"/>
```

Месяц спустя — в проекте 50 кнопок, и ни одна не похожа на другую.

### Симптомы проблемы

1. **Визуальный хаос** — каждый элемент выглядит по-своему
2. **Дублирование** — одни и те же стили копируются везде
3. **Сложность изменений** — чтобы поменять цвет бренда, нужно найти 100 мест
4. **Медленная разработка** — каждый раз решаем "какой цвет использовать"

---

## Дизайн-системы: что это

### Определение

**Дизайн-система** — это набор:

- **Принципов** — философия дизайна
- **Компонентов** — готовые UI-элементы
- **Стилей** — цвета, типографика, отступы
- **Паттернов** — типичные решения
- **Документации** — как это всё использовать

### Известные дизайн-системы

**Material Design (Google):**
- Философия: "материальные" поверхности с тенями
- Яркие акцентные цвета
- Анимации и feedback

**Fluent Design (Microsoft):**
- Философия: "глубина, свет, движение, материал, масштаб"
- Акриловые эффекты, размытие
- Адаптивность

**Human Interface Guidelines (Apple):**
- Философия: простота и понятность
- Минимализм
- Нативный look & feel

**Bootstrap (Web):**
- Практичный подход
- Сетка из 12 колонок
- Готовые компоненты

---

## Цветовая теория

### Цветовая палитра

Любая дизайн-система начинается с **палитры цветов**:

```
Primary     ████ #3498db  — Основной цвет бренда
Secondary   ████ #2ecc71  — Вторичный цвет
Accent      ████ #e74c3c  — Акцентный цвет
Background  ████ #ecf0f1  — Фон
Surface     ████ #ffffff  — Поверхности (карточки)
Text        ████ #2c3e50  — Основной текст
TextSecond  ████ #7f8c8d  — Второстепенный текст
```

### Цветовые роли

| Роль | Использование |
|------|---------------|
| **Primary** | Кнопки действия, ссылки, выделение |
| **Secondary** | Второстепенные кнопки, декор |
| **Accent** | Важные уведомления, ошибки |
| **Background** | Фон приложения |
| **Surface** | Карточки, диалоги, меню |
| **On-**** | Цвет текста на соответствующем фоне |

### Контрастность и Accessibility

**WCAG** (Web Content Accessibility Guidelines) требует минимальный контраст:

- **4.5:1** для обычного текста
- **3:1** для крупного текста и иконок

**Инструменты проверки:**
- [Contrast Checker](https://webaim.org/resources/contrastchecker/)
- [Accessible Colors](https://accessible-colors.com/)

**Пример:**
```
Белый текст на #3498db → контраст 3.38:1 → FAIL для мелкого текста
Белый текст на #2980b9 → контраст 4.56:1 → PASS
```

---

## CSS vs XAML Styles

### Общие принципы

И CSS, и XAML Styles решают одну задачу — **отделение стилей от структуры**:

| Концепция | CSS | XAML |
|-----------|-----|------|
| Стиль | `.button { ... }` | `<Style TargetType="Button">` |
| Применение | `class="button"` | `Style="{StaticResource ...}"` |
| Переменные | `--primary: #3498db;` | `<Color x:Key="Primary">` |
| Наследование | каскадность | BasedOn |
| Селекторы | `.btn:hover` | `<Trigger Property="IsMouseOver">` |

### XAML Resources

В WPF стили и ресурсы определяются в **ResourceDictionary**:

```xml
<Window.Resources>
    <!-- Цвета -->
    <Color x:Key="PrimaryColor">#3498db</Color>
    <SolidColorBrush x:Key="PrimaryBrush" Color="{StaticResource PrimaryColor}"/>
    
    <!-- Стиль кнопки -->
    <Style x:Key="PrimaryButton" TargetType="Button">
        <Setter Property="Background" Value="{StaticResource PrimaryBrush}"/>
        <Setter Property="Foreground" Value="White"/>
        <Setter Property="Padding" Value="20,10"/>
    </Style>
</Window.Resources>
```

### StaticResource vs DynamicResource

**StaticResource** — значение читается один раз при загрузке:
```xml
<Button Background="{StaticResource PrimaryBrush}"/>
```
Быстрее, но не меняется в runtime.

**DynamicResource** — значение отслеживается и обновляется:
```xml
<Button Background="{DynamicResource PrimaryBrush}"/>
```
Медленнее, но позволяет менять стили на лету (например, переключение темы).

---

## Токены дизайна

### Что такое токены

**Токены** — это именованные переменные дизайн-системы:

```xml
<!-- Примитивы (raw values) -->
<Color x:Key="Blue500">#3498db</Color>
<Color x:Key="Blue700">#2980b9</Color>
<sys:Double x:Key="Spacing4">4</sys:Double>
<sys:Double x:Key="Spacing8">8</sys:Double>

<!-- Семантические токены -->
<Color x:Key="ColorPrimary" Color="{StaticResource Blue500}"/>
<Color x:Key="ColorPrimaryDark" Color="{StaticResource Blue700}"/>
<sys:Double x:Key="SpacingSmall">{StaticResource Spacing4}</sys:Double>
```

### Зачем нужны токены

1. **Абстракция** — `ColorPrimary` понятнее, чем `#3498db`
2. **Централизация** — изменение в одном месте применяется везде
3. **Темы** — один токен может иметь разные значения в разных темах

### Пример использования

```xml
<Button Margin="{StaticResource SpacingSmall}"
        Background="{StaticResource ColorPrimary}"/>
```

При ребрендинге достаточно изменить значение `ColorPrimary`.

---

## Dark Mode — больше чем инверсия

### Наивный подход

```csharp
// НЕ ДЕЛАЙТЕ ТАК
void ToggleDarkMode()
{
    foreach (var element in AllElements)
    {
        element.Background = InvertColor(element.Background);
        element.Foreground = InvertColor(element.Foreground);
    }
}
```

Проблемы:
- Синий станет оранжевым
- Красный ошибки станет голубым
- Изображения инвертируются

### Правильный подход: две палитры

**Light Theme:**
```xml
<ResourceDictionary>
    <SolidColorBrush x:Key="BackgroundBrush" Color="#FFFFFF"/>
    <SolidColorBrush x:Key="SurfaceBrush" Color="#F5F5F5"/>
    <SolidColorBrush x:Key="TextBrush" Color="#212121"/>
    <SolidColorBrush x:Key="PrimaryBrush" Color="#3498db"/>
</ResourceDictionary>
```

**Dark Theme:**
```xml
<ResourceDictionary>
    <SolidColorBrush x:Key="BackgroundBrush" Color="#121212"/>
    <SolidColorBrush x:Key="SurfaceBrush" Color="#1E1E1E"/>
    <SolidColorBrush x:Key="TextBrush" Color="#FFFFFF"/>
    <SolidColorBrush x:Key="PrimaryBrush" Color="#64B5F6"/>  <!-- Светлее! -->
</ResourceDictionary>
```

### Особенности тёмной темы

1. **Не чисто чёрный фон** — #121212 вместо #000000 (меньше утомляет глаза)
2. **Приглушённые цвета** — яркие цвета "кричат" на тёмном фоне
3. **Меньше теней** — тени плохо видны на тёмном
4. **Больше глубины через яркость** — светлые поверхности "выше"

### Переключение темы в WPF

```csharp
private void ToggleTheme()
{
    var dict = Application.Current.Resources.MergedDictionaries;
    
    // Удаляем текущую тему
    var currentTheme = dict.FirstOrDefault(d => 
        d.Source?.OriginalString.Contains("Theme") == true);
    if (currentTheme != null)
        dict.Remove(currentTheme);
    
    // Добавляем новую
    var newTheme = _isDarkMode ? "LightTheme.xaml" : "DarkTheme.xaml";
    dict.Add(new ResourceDictionary 
    { 
        Source = new Uri(newTheme, UriKind.Relative) 
    });
    
    _isDarkMode = !_isDarkMode;
}
```

---

## Организация стилей в проекте

### Структура папок

```
CinemaKiosk/
├── Themes/
│   ├── Colors.xaml         ← Цветовые токены
│   ├── Typography.xaml     ← Шрифты и размеры
│   ├── Buttons.xaml        ← Стили кнопок
│   ├── Cards.xaml          ← Стили карточек
│   ├── LightTheme.xaml     ← Светлая тема
│   └── DarkTheme.xaml      ← Тёмная тема
├── App.xaml                ← Подключение словарей
```

### App.xaml — точка сборки

```xml
<Application>
    <Application.Resources>
        <ResourceDictionary>
            <ResourceDictionary.MergedDictionaries>
                <ResourceDictionary Source="Themes/Colors.xaml"/>
                <ResourceDictionary Source="Themes/Typography.xaml"/>
                <ResourceDictionary Source="Themes/Buttons.xaml"/>
                <ResourceDictionary Source="Themes/Cards.xaml"/>
                <ResourceDictionary Source="Themes/LightTheme.xaml"/>
            </ResourceDictionary.MergedDictionaries>
        </ResourceDictionary>
    </Application.Resources>
</Application>
```

### Порядок важен!

Словари, добавленные позже, перезаписывают предыдущие:

```xml
<!-- Colors.xaml -->
<SolidColorBrush x:Key="TextBrush" Color="Black"/>

<!-- DarkTheme.xaml (добавлен после) -->
<SolidColorBrush x:Key="TextBrush" Color="White"/>  <!-- Перезапишет -->
```

---

## Практический пример: Система стилей для CinemaKiosk

### Colors.xaml

```xml
<ResourceDictionary>
    <!-- Примитивы -->
    <Color x:Key="Blue500">#3498db</Color>
    <Color x:Key="Blue700">#2980b9</Color>
    <Color x:Key="Green500">#2ecc71</Color>
    <Color x:Key="Red500">#e74c3c</Color>
    <Color x:Key="Gray100">#f5f5f5</Color>
    <Color x:Key="Gray900">#212121</Color>
    
    <!-- Семантические -->
    <SolidColorBrush x:Key="PrimaryBrush" Color="{StaticResource Blue500}"/>
    <SolidColorBrush x:Key="PrimaryDarkBrush" Color="{StaticResource Blue700}"/>
    <SolidColorBrush x:Key="SuccessBrush" Color="{StaticResource Green500}"/>
    <SolidColorBrush x:Key="ErrorBrush" Color="{StaticResource Red500}"/>
</ResourceDictionary>
```

### Buttons.xaml

```xml
<ResourceDictionary>
    <Style x:Key="BaseButton" TargetType="Button">
        <Setter Property="FontSize" Value="14"/>
        <Setter Property="Padding" Value="20,10"/>
        <Setter Property="BorderThickness" Value="0"/>
        <Setter Property="Cursor" Value="Hand"/>
    </Style>
    
    <Style x:Key="PrimaryButton" TargetType="Button" BasedOn="{StaticResource BaseButton}">
        <Setter Property="Background" Value="{DynamicResource PrimaryBrush}"/>
        <Setter Property="Foreground" Value="White"/>
        <Setter Property="Template">
            <Setter.Value>
                <ControlTemplate TargetType="Button">
                    <Border Background="{TemplateBinding Background}"
                            CornerRadius="5"
                            Padding="{TemplateBinding Padding}">
                        <ContentPresenter HorizontalAlignment="Center"/>
                    </Border>
                </ControlTemplate>
            </Setter.Value>
        </Setter>
    </Style>
    
    <Style x:Key="SecondaryButton" TargetType="Button" BasedOn="{StaticResource BaseButton}">
        <Setter Property="Background" Value="Transparent"/>
        <Setter Property="Foreground" Value="{DynamicResource PrimaryBrush}"/>
        <Setter Property="BorderBrush" Value="{DynamicResource PrimaryBrush}"/>
        <Setter Property="BorderThickness" Value="1"/>
    </Style>
</ResourceDictionary>
```

---

## Связь с лабораторной работой

### Лаба 5: Стилизация

В лабораторной работе вы:

1. **Создадите ResourceDictionary** с цветами и стилями
2. **Вынесете inline-стили** в ресурсы
3. **Реализуете переключение темы** светлая/тёмная
4. **Стилизуете кнопки** со скруглениями и hover-эффектами

**Критерии:**
- Нет `Background="..."` в элементах — только `Style="{StaticResource ...}"`
- Работает переключение темы
- Приложение выглядит профессионально

---

## Заключение

**Дизайн-система** — это не просто "красивые картинки":

- **Consistency** — единообразие во всём приложении
- **Efficiency** — быстрая разработка на основе готовых компонентов
- **Scalability** — легко добавлять новые экраны
- **Maintainability** — изменения в одном месте применяются везде

Даже в учебном проекте стоит заложить основы дизайн-системы — это профессиональный подход.

---

## Полезные ресурсы

- [Material Design](https://material.io/design) — дизайн-система Google
- [Fluent Design](https://docs.microsoft.com/en-us/windows/apps/design/) — дизайн-система Microsoft
- [Design Tokens W3C](https://design-tokens.github.io/community-group/format/) — стандарт токенов
- [Coolors](https://coolors.co/) — генератор палитр
