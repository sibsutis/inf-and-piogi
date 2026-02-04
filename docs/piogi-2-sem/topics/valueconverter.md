---
title: Converter это весело
---
>**Проблема:** В переменной — `true` или `false`, а в интерфейсе — нужен красивый текст.  
>Решение — **конвертер**.

**Value converter** в C# WPF — это класс, который реализует [интерфейс](../topics/interfaces) `IValueConverter` и предоставляет методы для преобразования данных из одной формы в другую при привязке данных в XAML.

**Некоторые возможности использования value converters:**

- форматирование дат;
- преобразование между единицами измерения;
- реализация условной логики в привязках.

Для примера мы реализуем конвертацию логического значения в строковое (`bool` -> `string`)

Для этого в файле `UserControl.xaml.cs` нужно создать еще один класс:

```csharp
public partial class RoomControl : UserControl
{
	//Наш компонент тут
}
public class BoolToString : IValueConverter
{
	//Наш класс конверт, который будет иметь 2 функции.
	//IValueConverter - интерфейс, : значит что мы наследуемся от него.
}
```

>На начальном этапе проще всего писать дополнительный класс конвертора в классе (файлике) того окна, в котором вы хотите его использовать.
>
>**Предпочтительнее завести отдельный файл-класс под методы-конверторы.** (Просто создать еще один класс).
## Пример Convert

Наш класс **ДОЛЖЕН** реализовывать 2 метода из интерфейса:

```csharp
        public object Convert(object value, Type targetType, object parameter, CultureInfo culture)
        {
            return ((bool)(value) ? "Свет включен" : "Свет выключен");

        }

        public object ConvertBack(object value, Type targetType, object parameter, CultureInfo culture)
        {
            return DependencyProperty.UnsetValue;
        }

```

| Метод         | Что делает                                                                                                                                                                                                                                                                             |
| ------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Convert`     | Преобразует значение **из данных** (например, `bool`) **в то, что надо показать в UI** (например, текст "Свет включен").                                                                                                                                                               |
| `ConvertBack` | Преобразует **обратно**, из того, что выбрал пользователь в интерфейсе, **обратно в данные**. (Здесь мы вернем `DependencyProperty.UnsetValue`, значит обратного преобразования у нас нет. Но если мы зачем-то как-то захотим преобразовывать значения обратно - мы можем сделать это) |

## Пример ConverBack

Представим, что мы осуществили привязку не в текст `Button`, а в поле для ввода - `TextBox`. Тогда метод обратной конвертации будем иметь смысл и выглядеть так:

```csharp
// Конвертирует обратно: из текста в bool
    public object ConvertBack(object value, Type targetType, object parameter, CultureInfo culture)
    {
        // Если пользователь ввел "Свет включен", возвращаем true, если "Свет выключен" - false
        if (value.ToString() == "Свет включен")
            return true;
        else if (value.ToString() == "Свет выключен")
            return false;

        // Если введено что-то другое, возвращаем UnsetValue (значение не установлено)
        return DependencyProperty.UnsetValue;
    }
```

## Использование Convert

В файле ваших стилей нужно объявить названия конверторов для `xaml`:

```xml
    <local:BoolToString x:Key="myBoolConverter" />
```
Здесь: 
- `BoolToString` - название класса конвертора.
- `x:Key` - название вашего конвертора для файлов `xaml`.

Также в заголовок файла нужно прописать следующее:

```xml

<ResourceDictionary xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
                    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml" 
                    
                    xmlns:local="clr-namespace:ваш_проект"> вот это прописать

```

Пример привязки данных и использование конвертора:

```xml
<Button Click="ButtonBase_OnClick" Content="{Binding Path=IsLightOn, Converter={StaticResource myBoolConverter}}" d:Content="Тут будет состояние света" Style="{StaticResource GrayButton}" />
```

```text
Content="{Binding Path=IsLightOn, Converter={StaticResource myBoolConverter}}"
```

- Указываем свойство для привязки: `IsLightOn`;
- Указываем конверт данных.