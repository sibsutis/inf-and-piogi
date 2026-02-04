
## Определение
`OpenFileDialog` и `SaveFileDialog` — это стандартные компоненты в WPF (и Windows Forms), которые позволяют пользователю взаимодействовать с файловой системой для выбора файлов для открытия или указания имени и места для сохранения файла. Это очень распространенная задача в настольных приложениях.

## **1. Введение: Зачем нужны диалоги файлов?**

Когда вы создаете приложение, которому нужно работать с файлами (например, текстовый редактор, *проводник* изображений, программа для импорта/экспорта данных), вам нужен способ, чтобы пользователь мог:
*   Указать, какой файл он хочет **открыть**.
*   Указать, под каким именем и в какую папку он хочет **сохранить** данные.

Пытаться реализовать такой интерфейс выбора файлов самостоятельно — очень сложная задача. К счастью, операционная система Windows предоставляет стандартные диалоговые окна для этих целей, а .NET Framework (и .NET Core/.NET 5+) дают к ним удобный доступ через классы `OpenFileDialog` и `SaveFileDialog`.
Пользователи уже знают, как с ними работать, так как это стандартное окно из Windows.

**Важно:** Эти диалоги **не выполняют** фактического чтения или записи файла. Они лишь предоставляют приложению путь к файлу, выбранному пользователем. Операции чтения/записи вы должны реализовать сами, используя классы из пространства имен `System.IO` (например, `File.ReadAllText`, `File.WriteAllText`, `StreamReader`, `StreamWriter` и т.д.).

## **2. `OpenFileDialog`**

Класс `OpenFileDialog` используется для отображения диалогового окна, в котором пользователь может выбрать один или несколько файлов для открытия.

Для начала нужно создать переменную класса `OpenFileDialog`:

```csharp
OpenFileDialog openFileDialog = new OpenFileDialog();
```

**Ключевые свойства и методы:**

*   **`Title` (string):** Заголовок окна диалога. По умолчанию это "Open" или "Открыть".

```csharp
openFileDialog.Title = "Выберите текстовый файл для открытия";
```

*   **`InitialDirectory` (string):** Путь к каталогу, который будет открыт по умолчанию при первом отображении диалога.

```csharp
openFileDialog.InitialDirectory = Environment.GetFolderPath(Environment.SpecialFolder.MyDocuments);
```

*   **`Filter` (string):** Строка, определяющая фильтры файлов, которые пользователь может выбрать (например, только текстовые файлы, только изображения). Формат строки:
`"Описание1|*.расширение1|Описание2|*.расширение2;*.расширение3"`

Пример:

```csharp
openFileDialog.Filter = "Текстовые файлы (*.txt)|*.txt|Документы Word (*.docx)|*.docx|Все файлы (*.*)|*.*";
// Разделитель - вертикальная черта |
// Сначала идет описание, видимое пользователю, потом маска файлов.
// Несколько масок для одного описания разделяются точкой с запятой.
```

*   **`FilterIndex` (int):** Индекс (начиная с 1) фильтра, который будет выбран по умолчанию.

```csharp
openFileDialog.FilterIndex = 1; // По умолчанию будет выбран первый фильтр ("Текстовые файлы (*.txt)")
```

*   **`Multiselect` (bool):** Позволяет ли пользователю выбирать несколько файлов. По умолчанию `false`.

```csharp
openFileDialog.Multiselect = true;
```

*   **`CheckFileExists` (bool):** Проверять ли, существует ли выбранный файл. По умолчанию `true`. Если `false`, пользователь сможет ввести имя несуществующего файла.

*   **`ShowDialog()` (метод):** Отображает диалоговое окно. Этот метод **модальный**, т.е. он блокирует остальную часть интерфейса вашего приложения, пока пользователь не закроет диалог (выбрав файл или нажав "Отмена").

Возвращает `bool?` (nullable boolean):
*   `true`, если пользователь выбрал файл(ы) и нажал "Открыть".
*   `false`, если пользователь нажал "Отмена" или закрыл диалог.
*   `null` в некоторых редких случаях (хотя обычно это `true` или `false`).

```csharp
bool? result = openFileDialog.ShowDialog();
if (result == true)
{
    string fn = openFileDialog.FileName;
    string[] fns = openFileDialog.FileNames;
    string sfn = openFileDialog.SafeFileName;
    string[] sfns = openFileDialog.SafeFileNames;
	
    // Пользователь выбрал файл(ы)
}
```

*   **`FileName` (string):** Полный путь к выбранному файлу (если `Multiselect = false` или выбран один файл при `Multiselect = true`).
*   **`FileNames` (string[]):** Массив полных путей ко всем выбранным файлам (если `Multiselect = true`).
*   **`SafeFileName` (string):** Только имя файла с расширением, без пути (если `Multiselect = false` или выбран один файл).
*   **`SafeFileNames` (string[]):** Массив имен файлов с расширениями (если `Multiselect = true`).

## **3. `SaveFileDialog`**

Класс `SaveFileDialog` используется для отображения диалогового окна, в котором пользователь может указать имя файла и каталог для сохранения данных.

**Ключевые свойства и методы (многие схожи с `OpenFileDialog`):**

*   **`Title` (string):** Заголовок окна диалога. По умолчанию "Save As" или "Сохранить как".
*   **`InitialDirectory` (string):** Начальный каталог.
*   **`Filter` (string):** Фильтры файлов (помогают пользователю выбрать правильное расширение).
*   **`FilterIndex` (int):** Индекс фильтра по умолчанию.
*   **`DefaultExt` (string):** Расширение файла по умолчанию, которое будет добавлено, если пользователь введет имя файла без расширения. Например, `".txt"`.

```csharp
saveFileDialog.DefaultExt = ".txt";
```

*   **`FileName` (string):** Может использоваться для предложения имени файла по умолчанию. После закрытия диалога содержит полный путь к файлу, указанному пользователем.

```csharp
saveFileDialog.FileName = "МойДокумент"; // Предложит это имя
```

*   **`OverwritePrompt` (bool):** Показывать ли предупреждение, если пользователь выбирает существующий файл для перезаписи. По умолчанию `true`.
*   **`CreatePrompt` (bool):** Показывать ли запрос на создание файла, если пользователь вводит имя несуществующего файла. Обычно не используется для `SaveFileDialog`, так как цель — именно создать/перезаписать. По умолчанию `false`.
*   **`ShowDialog()` (метод):** Отображает диалог. Возвращает `bool?` (аналогично `OpenFileDialog`).

```csharp
bool? result = saveFileDialog.ShowDialog();
if (result == true)
{
	// Пользователь указал имя файла и нажал "Сохранить"
	string filePath = saveFileDialog.FileName;
	// Здесь код для сохранения данных в filePath
}
```

## **4. Практический пример для WPF**

Давайте создадим простое WPF-приложение с кнопками для открытия и сохранения текстового файла и `TextBox` для отображения/редактирования его содержимого.

**XAML (MainWindow.xaml):**

```xml
<Window x:Class="WpfFileDialogsExample.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        xmlns:local="clr-namespace:WpfFileDialogsExample"
        mc:Ignorable="d"
        Title="File Dialog Demo" Height="450" Width="600">
    <Grid>
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="*"/>
            <RowDefinition Height="Auto"/>
        </Grid.RowDefinitions>

        <StackPanel Grid.Row="0" Orientation="Horizontal" Margin="5">
            <Button Content="Открыть файл" Click="OpenFileButton_Click" Margin="5"/>
            <Button Content="Сохранить файл как..." Click="SaveFileButton_Click" Margin="5"/>
        </StackPanel>

        <TextBox x:Name="FileContentTextBox" Grid.Row="1" Margin="5"
                 AcceptsReturn="True" TextWrapping="Wrap" VerticalScrollBarVisibility="Auto"/>

        <StatusBar Grid.Row="2">
            <StatusBarItem>
                <TextBlock x:Name="StatusTextBlock" Text="Готово"/>
            </StatusBarItem>
        </StatusBar>
    </Grid>
</Window>
```

**C# (MainWindow.xaml.cs):**
(Не забудьте добавить `using Microsoft.Win32;` и `using System.IO;`)

```csharp
using Microsoft.Win32; // Для OpenFileDialog и SaveFileDialog
using System;
using System.IO;       // Для работы с файлами (File.ReadAllText, File.WriteAllText)
using System.Windows;

namespace WpfFileDialogsExample
{
    public partial class MainWindow : Window
    {
        private string currentFilePath = null; // Для хранения пути к текущему открытому файлу

        public MainWindow()
        {
            InitializeComponent();
        }

        private async void OpenFileButton_Click(object sender, RoutedEventArgs e)
        {
            OpenFileDialog openFileDialog = new OpenFileDialog();
            openFileDialog.Title = "Выберите текстовый файл";
            openFileDialog.Filter = "Текстовые файлы (*.txt)|*.txt|Все файлы (*.*)|*.*";
            openFileDialog.InitialDirectory = Environment.GetFolderPath(Environment.SpecialFolder.MyDocuments);

            if (openFileDialog.ShowDialog() == true)
            {
                try
                {
                    currentFilePath = openFileDialog.FileName;
                    // Чтение файла лучше делать асинхронно, чтобы не блокировать UI
                    string fileText = await File.ReadAllTextAsync(currentFilePath);
                    FileContentTextBox.Text = fileText;
                    StatusTextBlock.Text = $"Файл открыт: {currentFilePath}";
                }
                catch (Exception ex)
                {
                    MessageBox.Show($"Ошибка при открытии файла: {ex.Message}", "Ошибка", MessageBoxButton.OK, MessageBoxImage.Error);
                    StatusTextBlock.Text = "Ошибка открытия файла.";
                }
            }
            else
            {
                StatusTextBlock.Text = "Открытие файла отменено.";
            }
        }

        private async void SaveFileButton_Click(object sender, RoutedEventArgs e)
        {
            SaveFileDialog saveFileDialog = new SaveFileDialog();
            saveFileDialog.Title = "Сохранить текстовый файл как...";
            saveFileDialog.Filter = "Текстовые файлы (*.txt)|*.txt|Все файлы (*.*)|*.*";
            saveFileDialog.DefaultExt = ".txt";
            saveFileDialog.InitialDirectory = Environment.GetFolderPath(Environment.SpecialFolder.MyDocuments);

            // Если файл уже был открыт или сохранен, предложим его имя
            if (!string.IsNullOrEmpty(currentFilePath))
            {
                saveFileDialog.FileName = Path.GetFileName(currentFilePath); // Только имя файла
                saveFileDialog.InitialDirectory = Path.GetDirectoryName(currentFilePath); // Его директория
            }
            else
            {
                saveFileDialog.FileName = "НовыйДокумент.txt";
            }


            if (saveFileDialog.ShowDialog() == true)
            {
                try
                {
                    currentFilePath = saveFileDialog.FileName;
                    // Запись файла лучше делать асинхронно
                    await File.WriteAllTextAsync(currentFilePath, FileContentTextBox.Text);
                    StatusTextBlock.Text = $"Файл сохранен: {currentFilePath}";
                }
                catch (Exception ex)
                {
                    MessageBox.Show($"Ошибка при сохранении файла: {ex.Message}", "Ошибка", MessageBoxButton.OK, MessageBoxImage.Error);
                    StatusTextBlock.Text = "Ошибка сохранения файла.";
                }
            }
            else
            {
                StatusTextBlock.Text = "Сохранение файла отменено.";
            }
        }
    }
}
```

