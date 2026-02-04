---
title: Лекция 5.1 - Алгоритмическое мышление
---

# Алгоритмическое мышление

## Введение

В лабораторной работе 4 вам предстоит сгенерировать карту зала кинотеатра из данных API. Это не просто "вставить картинку" — нужно превратить массив данных в сетку кнопок.

Это требует **алгоритмического мышления** — способности разбить сложную задачу на простые шаги и превратить их в код.

---

## Матрицы и координатные системы

### Зал как двумерный массив

Зал кинотеатра — это **матрица** (двумерный массив):

```
        Колонка
        1   2   3   4   5
      ┌───┬───┬───┬───┬───┐
Ряд 1 │ 1 │ 2 │ 3 │ 4 │ 5 │
      ├───┼───┼───┼───┼───┤
Ряд 2 │ 6 │ 7 │ 8 │ 9 │10 │
      ├───┼───┼───┼───┼───┤
Ряд 3 │11 │12 │13 │14 │15 │
      └───┴───┴───┴───┴───┘
```

Каждое место имеет **координаты**: `[ряд, место]` или `[row, column]`.

### Индексация: с 0 или с 1?

**В программировании** индексы начинаются с 0:
```csharp
int[,] seats = new int[3, 5];  // 3 ряда по 5 мест
seats[0, 0] = 1;  // Ряд 0, Место 0 — первое место первого ряда
seats[2, 4] = 15; // Ряд 2, Место 4 — последнее место последнего ряда
```

**Для пользователя** нумерация с 1 удобнее:
- Билет "Ряд 1, Место 1" понятнее, чем "Ряд 0, Место 0"

**Конверсия:**
```csharp
// Из индекса в номер для пользователя
int displayRow = arrayIndex + 1;

// Из номера пользователя в индекс
int arrayIndex = displayNumber - 1;
```

### Линейный индекс vs координаты

Иногда данные приходят "плоским" списком:

```json
{
  "rows": 3,
  "seatsPerRow": 5,
  "seats": [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15]
}
```

**Преобразование линейного индекса в координаты:**
```csharp
int linearIndex = 7;  // 8-е место (индексация с 0)
int seatsPerRow = 5;

int row = linearIndex / seatsPerRow;     // 7 / 5 = 1
int column = linearIndex % seatsPerRow;  // 7 % 5 = 2

// Результат: Ряд 1, Место 2 (индексация с 0)
// Для пользователя: Ряд 2, Место 3
```

**Преобразование координат в линейный индекс:**
```csharp
int row = 1;
int column = 2;
int seatsPerRow = 5;

int linearIndex = row * seatsPerRow + column;  // 1 * 5 + 2 = 7
```

---

## Алгоритмы обхода

### Вложенные циклы

Для обхода двумерного массива используются вложенные циклы:

```csharp
// Генерация зала 3x5
int rows = 3;
int columns = 5;

for (int row = 0; row < rows; row++)
{
    for (int col = 0; col < columns; col++)
    {
        // Создаём место для каждой комбинации row/col
        CreateSeat(row, col);
    }
}
```

**Порядок обхода (по строкам):**
```
(0,0) → (0,1) → (0,2) → (0,3) → (0,4) → 
(1,0) → (1,1) → (1,2) → (1,3) → (1,4) → 
(2,0) → (2,1) → (2,2) → (2,3) → (2,4)
```

### foreach vs for

**foreach** — когда индекс не нужен:
```csharp
foreach (var seat in seats)
{
    // seat — текущий элемент
    // Индекс недоступен напрямую
    ProcessSeat(seat);
}
```

**for** — когда нужен индекс:
```csharp
for (int i = 0; i < seats.Count; i++)
{
    var seat = seats[i];
    // i — индекс текущего элемента
    PlaceSeatAtPosition(seat, i);
}
```

**LINQ с индексом:**
```csharp
var seatsWithIndex = seats.Select((seat, index) => new { seat, index });
foreach (var item in seatsWithIndex)
{
    PlaceSeatAtPosition(item.seat, item.index);
}
```

---

## Сложность алгоритмов O(n)

### Зачем это знать

Когда зал маленький (5x5 = 25 мест), любой алгоритм работает быстро. Но что, если залов 10, и в каждом 500 мест?

**Сложность алгоритма** показывает, как время выполнения растёт с увеличением входных данных.

### Основные классы сложности

| Обозначение | Название | Пример |
|-------------|----------|--------|
| O(1) | Константная | Доступ к элементу массива по индексу |
| O(log n) | Логарифмическая | Бинарный поиск |
| O(n) | Линейная | Проход по всем элементам |
| O(n log n) | Линейно-логарифмическая | Эффективная сортировка |
| O(n²) | Квадратичная | Вложенные циклы по одним данным |
| O(2ⁿ) | Экспоненциальная | Полный перебор комбинаций |

### Примеры в контексте приложения

**O(1) — Константная:**
```csharp
// Получить место по номеру — мгновенно
var seat = seats[42];
```

**O(n) — Линейная:**
```csharp
// Подсчитать занятые места — пропорционально количеству мест
int occupied = 0;
foreach (var seat in seats)
{
    if (seat.IsOccupied) occupied++;
}
// Или: int occupied = seats.Count(s => s.IsOccupied);
```

**O(n²) — Квадратичная:**
```csharp
// Найти все пары соседних занятых мест
for (int i = 0; i < seats.Count; i++)
{
    for (int j = i + 1; j < seats.Count; j++)
    {
        if (AreNeighbors(seats[i], seats[j]) && 
            seats[i].IsOccupied && seats[j].IsOccupied)
        {
            // Нашли пару
        }
    }
}
```

### Практическое правило

- **O(n)** с n = 1000 → ~1000 операций → мгновенно
- **O(n²)** с n = 1000 → ~1 000 000 операций → заметная задержка
- **O(n²)** с n = 10 000 → ~100 000 000 операций → секунды ожидания

**Вывод:** Избегайте O(n²) для больших данных, если возможно.

---

## Динамическая генерация UI

### Задача

Превратить данные API в визуальные элементы:

```json
{
  "rows": 5,
  "seatsPerRow": 10,
  "seats": [
    {"row": 1, "number": 1, "isOccupied": false},
    {"row": 1, "number": 2, "isOccupied": true},
    // ...
  ]
}
```

### Решение с UniformGrid

**UniformGrid** — контейнер, который автоматически раскладывает элементы в сетку:

```xml
<UniformGrid x:Name="SeatsGrid" Rows="5" Columns="10"/>
```

```csharp
private async Task LoadHall(int sessionId)
{
    var hall = await _apiService.GetSeatsAsync(sessionId);
    
    // Настраиваем сетку
    SeatsGrid.Rows = hall.Rows;
    SeatsGrid.Columns = hall.SeatsPerRow;
    SeatsGrid.Children.Clear();
    
    // Создаём места
    foreach (var seatData in hall.Seats)
    {
        var seatControl = new SeatControl();
        seatControl.Initialize(seatData.Row, seatData.Number, seatData.IsOccupied);
        seatControl.SeatClicked += OnSeatClicked;
        SeatsGrid.Children.Add(seatControl);
    }
}
```

### Алгоритм генерации

```
Входные данные: массив мест
Выход: визуальная сетка

1. Очистить контейнер
2. Установить размеры сетки (rows × columns)
3. Для каждого места в массиве:
   a. Создать визуальный элемент
   b. Настроить его свойства (номер, состояние)
   c. Подписаться на события
   d. Добавить в контейнер
```

### Важно: порядок элементов

UniformGrid заполняется **слева направо, сверху вниз**:

```
Порядок добавления:  1  2  3  4  5
                     6  7  8  9 10
                    11 12 13 14 15
```

Если данные API приходят в другом порядке — нужно отсортировать:

```csharp
var sortedSeats = hall.Seats
    .OrderBy(s => s.Row)
    .ThenBy(s => s.Number)
    .ToList();
```

---

## Практические задачи

### Задача 1: Шахматная раскладка

Некоторые кинотеатры располагают места в шахматном порядке для лучшего обзора:

```
    1   2   3   4   5
  ┌───┬───┬───┬───┬───┐
1 │ ● │   │ ● │   │ ● │
  ├───┼───┼───┼───┼───┤
2 │   │ ● │   │ ● │   │
  ├───┼───┼───┼───┼───┤
3 │ ● │   │ ● │   │ ● │
  └───┴───┴───┴───┴───┘
```

**Алгоритм:**
```csharp
for (int row = 0; row < rows; row++)
{
    for (int col = 0; col < columns; col++)
    {
        // Шахматный порядок: (row + col) % 2 == 0
        bool hasSeat = (row + col) % 2 == 0;
        
        if (hasSeat)
        {
            CreateSeat(row, col);
        }
        else
        {
            CreateEmptySpace();  // Placeholder для сохранения сетки
        }
    }
}
```

### Задача 2: VIP-ряды

Последние ряды — VIP, места шире:

```
Обычный:  [1][2][3][4][5][6][7][8]
VIP:      [ 1 ][ 2 ][ 3 ][ 4 ]
```

**Решение с Grid:**
```xml
<Grid x:Name="HallGrid">
    <Grid.RowDefinitions>
        <!-- Генерируются динамически -->
    </Grid.RowDefinitions>
</Grid>
```

```csharp
for (int row = 0; row < totalRows; row++)
{
    bool isVip = row >= totalRows - 2;  // Последние 2 ряда — VIP
    int seatsInRow = isVip ? vipSeatsCount : normalSeatsCount;
    
    // Создаём RowDefinition
    HallGrid.RowDefinitions.Add(new RowDefinition());
    
    // Создаём UniformGrid для ряда
    var rowPanel = new UniformGrid { Columns = seatsInRow };
    Grid.SetRow(rowPanel, row);
    HallGrid.Children.Add(rowPanel);
    
    // Заполняем местами
    for (int col = 0; col < seatsInRow; col++)
    {
        var seat = CreateSeat(row, col, isVip);
        rowPanel.Children.Add(seat);
    }
}
```

### Задача 3: Проход посередине

Типичный зал имеет проход в центре:

```
[1][2][3]   [4][5][6]
[1][2][3]   [4][5][6]
```

**Решение:**
```csharp
for (int col = 0; col < columns; col++)
{
    bool isAisle = col == columns / 2;  // Проход посередине
    
    if (isAisle)
    {
        // Пустое место-разделитель
        var spacer = new Border { Width = 30, Background = Brushes.Transparent };
        SeatsGrid.Children.Add(spacer);
    }
    else
    {
        CreateSeat(row, col);
    }
}
```

---

## Отладка алгоритмов

### Console.WriteLine — ваш друг

При отладке генерации полезно выводить промежуточные значения:

```csharp
for (int row = 0; row < rows; row++)
{
    for (int col = 0; col < columns; col++)
    {
        Console.WriteLine($"Creating seat at Row={row}, Col={col}");
        CreateSeat(row, col);
    }
}
```

### Визуализация в Debug

```csharp
// Вывести "карту" зала в консоль
for (int row = 0; row < rows; row++)
{
    for (int col = 0; col < columns; col++)
    {
        var seat = seats[row * columns + col];
        Console.Write(seat.IsOccupied ? "X" : "O");
    }
    Console.WriteLine();
}

// Вывод:
// OOXOO
// OOOOO
// XOOOX
```

### Breakpoints и Watch

В Visual Studio:
1. Поставьте breakpoint внутри цикла
2. Добавьте переменные в Watch: `row`, `col`, `linearIndex`
3. Смотрите, как меняются значения на каждой итерации

---

## Связь с лабораторной работой

### Лаба 4: Генерация зала

В лабораторной работе вы:

1. **Получите данные** — массив мест с API
2. **Преобразуете** — из JSON в объекты C#
3. **Сгенерируете** — визуальные элементы в UniformGrid
4. **Обработаете** — клики и выбор мест

**Ключевой код:**
```csharp
private async Task LoadHall()
{
    var hall = await _apiService.GetSeatsAsync(_sessionId);
    
    SeatsGrid.Rows = hall.Rows;
    SeatsGrid.Columns = hall.SeatsPerRow;
    
    foreach (var seat in hall.Seats.OrderBy(s => s.Row).ThenBy(s => s.Number))
    {
        var control = new SeatControl(seat.Row, seat.Number, seat.IsOccupied);
        control.OnSeatClicked += HandleSeatClick;
        SeatsGrid.Children.Add(control);
    }
}
```

---

## Заключение

**Алгоритмическое мышление** — это:

- Умение разбить задачу на простые шаги
- Понимание структур данных (массивы, матрицы)
- Выбор правильного алгоритма обхода
- Осознание производительности (O-нотация)

Эти навыки универсальны и применимы в любой области программирования — от игр до финансовых систем.

---

## Полезные ресурсы

- [Big O Notation](https://www.bigocheatsheet.com/) — шпаргалка по сложности
- [Visualgo](https://visualgo.net/) — визуализация алгоритмов
- [LeetCode](https://leetcode.com/) — практика алгоритмов
