`DisplayMemberPath` — это свойство, которое указывает, какое свойство объекта из коллекции будет отображаться в элементе управления. Это необходимо, когда вы хотите отобразить не весь объект, а только одно из его свойств.

#### Пример использования `DisplayMemberPath`:

Допустим, вы хотите отобразить в `ComboBox` только имена комнат, а не все их свойства.

```xml
<ComboBox x:Name="comboBox" DisplayMemberPath="Name" />
```

или

```csharp
// Установка ItemsSource
comboBox.ItemsSource = Rooms;

comboBox.DisplayMemberPath = "Name";
```

В этом примере свойство `Name` каждого объекта `Room` будет отображено в `ComboBox`.

1. **`ItemsSource`** — привязывает коллекцию данных (`Rooms`) к элементу управления (`ComboBox`).
2. **`DisplayMemberPath`** — указывает, что из каждого объекта в коллекции нужно отображать только свойство `Name`. В данном случае в `ComboBox` будут показаны только названия комнат (например, "Living Room", "Bedroom", "Kitchen").

### 3. Дополнительные параметры

- Если вы хотите отображать несколько свойств объекта, можно использовать `ItemTemplate` для более гибкой настройки отображения. Например:

```xml
<ComboBox x:Name="comboBox" ItemsSource="{Binding Rooms}">
    <ComboBox.ItemTemplate>
        <DataTemplate>
            <StackPanel>
                <TextBlock Text="{Binding Name}" />
                <TextBlock Text="{Binding Temperature}" />
            </StackPanel>
        </DataTemplate>
    </ComboBox.ItemTemplate>
</ComboBox>
```

В этом случае в `ComboBox` будут отображены и название, и температура каждой комнаты.