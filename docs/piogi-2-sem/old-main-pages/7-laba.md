---
title: Шаг за шагом 7 лаба
---


## Вступление

Данный текст содержит описание хода выполнения лабораторной работы.

В [лабораторной работе №5](../../main-pages/5-laba) мы создали приложение, которое использует компоненты для отображения комнат, а в [лабораторной работе №6](../../main-pages/6-laba) – программу для взаимодействия с базой данных, где хранятся данные о комнатах и квартирах.

Теперь мы свяжем лабораторную работу 5 и лабораторную работу 6 с помощью простого [TCP Server] и [TCP Client].

**Зачем нам это нужно связывать?** Данные из вашей базы (ЛР6) должны быть доступны не только локально для программы ЛР6, но и другим программам (ЛР5), возможно, даже работающим на других компьютерах. Или, например, ваше приложение из ЛР5 должно получать или отправлять информацию, которая обрабатывается и хранится централизованно.

В этой лабораторной работе мы сделаем шаг к созданию более сложных и распределенных систем. Мы объединим предыдущие лабораторные работы:

- Функционал работы с базой данных (из ЛР6) мы "выставим наружу" с помощью простого **TCP-сервера**. Это позволит другим приложениям запрашивать данные или отправлять команды на их изменение.
- Наше приложение (из ЛР5) выступит в роли **HTTP-клиента** (или TCP-клиента, если взаимодействие будет проще), который будет обращаться к этому серверу для получения или отправки информации.

Далее в тексте:

- Лабораторная работа №6 будет называться **база данных**;
- Лабораторная работа №5 будет называться **клиент умного дома**.

## Цель данной работы
Освоить базовые принципы клиент-серверного взаимодействия. В данной работе мы:

1. Создавать простой TCP-сервер на C#, способный принимать запросы и отправлять ответы.
2. Реализовывать клиентскую часть, которая сможет подключаться к серверу, обмениваться с ним данными по сети.
3. Расширить функционал 5 лабораторной работы, для возможности получения данных с сервера и отправки их на сервер.

## Структура проекта

Общую структуру проекта, состоящую из двух программ, можно описать следующим образом:

- **Сервер**: отдает программе умного дома (клиенту) инициализационные данные (всю информацию о квартирах и комнатах) и принимает запросы на обновление данных в базе данных.
- **Клиент**: принимает инициализационные данные от сервера, отображает эту информацию на графическом интерфейсе и отправляет изменения в базу данных.

Примерная диаграмма работы приложений:

```kroki-mermaid
flowchart LR
    LR5["ЛР5 (Умный дом)"]
    LR6["ЛР6 (База данных)"]
    subgraph "Инициализация"
        direction LR
        LR6_init["ЛР6 (БД)"] --"Передача начальных данных"--> LR5_init["ЛР5 (Умный дом)"]
    end
    subgraph "Операционный режим"
        direction LR
        LR5_op["ЛР5 (Умный дом)"]
        LR6_op["ЛР6 (БД)"]
        LR5_op --"Запрос данных о комнатах/квартирах"--> LR6_op
        LR6_op --"Ответ: данные о комнатах/квартирах"--> LR5_op
        LR5_op --"Отправка обновленной информации"--> LR6_op
    end
```


Также для выполнения работы нужно дополнить класс `Room` и `Apartment` необходимыми свойствами, также класс `Room` в **клиенте умного дома** должен соответствовать классу из **базы данных**.

## Пример сервера

[Подробно про TCP Server.]

Начнем с разработки простого TCP сервера для работы с данными.

Для этого мы используем класс `TcpListener` для прослушивания подключений, класс `TcpClient` - для хранения информации о полученном из подключения клиенте и класс `NetworkStream` для осуществления чтения и записи в открытом подключении.

Весь код **сервера** должен быть написан в приложении **базы данных**.
### Код

В приложении **базы данных** создадим новый класс (пример `TcpServer.cs`) и объявим в нем глобальные переменные:

```csharp
using System.IO;
using System.Net;
using System.Net.Sockets;
//Необходимые библиотеки для работы с сетью

namespace название_Вашего_проекта_БД
{
    public class TcpServer
    {
	    //Вот тут будем писать функции и переменные
    }
}
```

```csharp
TcpListener server = null; //Для прослушки порта
TcpClient client = null; //Для информации о клиенте
NetworkStream stream = null; //Для получения\отправки информации
```

Также объявим события для возможности ведения логов происходящего в нашем сервере:

```csharp
// События для обратной связи
public event Action<string> MessageReceived;
public event Action<string> Log;
```

Теперь объявим функцию `StartServer`:

```csharp
public class TcpServer
    {
	    TcpListener server = null;
		TcpClient client = null;
		NetworkStream stream = null;
		
		// События для обратной связи
		public event Action<string> MessageReceived;
		public event Action<string> Log;
		
		public void StartServer()
		{
		
		}
    }
```

Внутри этой функции:

```csharp
// 1. Запуск сервера 
server = new TcpListener(IPAddress.Parse("127.0.0.1"), 8888); //на локальном адресе, порт 8888
server.Start(); //Стартуем сервер
Log?.Invoke("Сервер запущен и ждет подключения..."); //Кидаем в обработчик сообщение (позже напишем обработчик)
```

```csharp
// 2. Ожидание подключения клиента (блокирует выполнение)
client = server.AcceptTcpClient(); 
//Именно тут программа остановится ("Не отвечает") и будет ждать подключения клиента
//Мы будем запускать эту функцию в задаче Task
Log?.Invoke($"Клиент {client.Client.LocalEndPoint} подключен!");
//Когда клиент подключится - мы кинем сообщение об этом
```

```csharp
// 3. Получение потока для обмена данными
stream = client.GetStream(); //Создаем переменную сетевого потока
StreamReader reader = new StreamReader(stream, Encoding.UTF8); //Переменная для получения из потока
StreamWriter writer = new StreamWriter(stream, Encoding.UTF8); //Переменная для отправки в поток
```

```csharp
// 4. Чтение сообщения от клиента
string clientMessage = reader.ReadLine(); //Получаем ТОЛЬКО 1 строку из потока (чуть позже об этом)
Log?.Invoke($"Клиент: {clientMessage}"); //Выводим в лог что нам прислал клиент
```

```csharp
// 5. Отправка ответа клиенту
string serverResponse = "Привет от сервера!";
writer.WriteLine(serverResponse); //Записать строку выше в "отправитель" (см. 3 блок кода)
writer.Flush(); // Отправить данные немедленно
Log?.Invoke($"Сервер: {serverResponse}"); //Выводим что там сказал сервер
```

Весь этот код будет находится внутри блока `try-catch` для обработки исключений:

```csharp
try
{
	//Весь код, описанный выше находится тут
}
catch (Exception e) //Если вдруг было брошено исключение
{
    Log?.Invoke("Ошибка: " + e.Message); //Выводим в лог текст ошибки этого исключения
}
finally //Блок кода, выполняющийся всегда после try-catch или исключения
{
    // 6. Закрытие соединений и остановка сервера
    stream?.Close(); // с помощью ? мы проверяем переменную на "равность" null
    //if(stream == nul) - аналог
    client?.Close();
    server?.Stop();
    Log?.Invoke("Сервер остановлен.");
}
```

Вот так будет выглядеть простой `TcpServer`, который:

1. Открывает прослушивание порта;
2. Принимает подключение клиента;
3. Получает сообщение от клиента;
4. Сразу отправляет ответ клиенту;
5. Закрывает подключение.

**То есть, наш сервер перестанет принимать сообщения после 1 полученного подключения.**

### Общая диаграмма класса `TcpServer`

```kroki-mermaid
classDiagram
        class TcpServer {
        +StartServer() void
        +TcpServer()
        +Action~string~ MessageReceived
        +Action~string~ Log
    }

```


## Пример клиента

Код клиента будет схож с кодом сервера. Мы также будем использовать класс `TcpClient` для самого клиента и `NetworkStream` для чтения и записи из потока.

Весь код **клиента** должен быть написан в приложении **умного дома**.

### Код

В приложении **умного дома** создадим новый класс (пример `TcpClient.cs`) и объявим в нем глобальные переменные:

```csharp
using System.Net.Sockets;
using System.IO;

namespace название_Вашего_проекта_умного_дома
{
    class Client
    {
		//Вот тут будем писать функции и переменные
    }
}
```


```csharp
	TcpClient client = null; //Для информации о клиенте
	NetworkStream stream = null; //Для получения\отправки информации
```

Теперь объявим функцию `TryConnecty` для подключения к серверу:

```csharp
class Client
{

    TcpClient client = null;
    NetworkStream stream = null;

    public event Action<string> Log;

    public void TryConnect()
    {
        
    }
}
```

Внутри этой функции:

```csharp
// 1. Подключение к серверу
client = new TcpClient("127.0.0.1", 8888);
//127.0.0.1 - адрес этого же компьютера
//8888 - порт, указан как в сервере
Log?.Invoke("Клиент подключен к серверу.");
//Кидаем в лог сообщение
```

```csharp
// 2. Получение потока для обмена данными
stream = client.GetStream(); //Создаем переменную сетевого потока
StreamReader reader = new StreamReader(stream, Encoding.UTF8); //Переменная для получения из потока
StreamWriter writer = new StreamWriter(stream, Encoding.UTF8); //Переменная для отправки в поток
```

```csharp
// 3. Отправка сообщения серверу
string clientMessage = "Привет, сервер!\nА вот это ты не видишь!"; 
//Формируем строку
writer.WriteLine(clientMessage); //Записываем данные в поток
writer.Flush(); // Отправить данные немедленно
Log?.Invoke($"Клиент: {clientMessage}");
```

```csharp
// 4. Чтение ответа от сервера
string serverResponse = reader.ReadLine(); //Получаем ответ от сервера
Log?.Invoke($"Сервер: {serverResponse}"); //Выводим на экран
```

Весь этот код будет находится внутри блока `try-catch` для обработки исключений:

```csharp
try
{
	//Весь код, описанный выше находится тут
}
catch (Exception e) //Если вдруг было брошено исключение
{
    Log?.Invoke("Ошибка: " + e.Message); //Выводим в лог текст ошибки этого исключения
}
finally //Блок кода, выполняющийся всегда после try-catch или исключения
{
    // 6. Закрытие соединений и остановка сервера
    stream?.Close(); // с помощью ? мы проверяем переменную на "равность" null
    //if(stream == nul) - аналог
    client?.Close();
    Log?.Invoke("Сервер остановлен.");
}
```

Вот так будет выглядеть простой `TcpClient`, который:

1. Подключается к серверу;
2. Отправляет сообщение;
3. Получает ответ от сервера;
4. Закрывает подключение.

**То есть, наш клиент отправляет только 1 сообщение за 1 подключение.**

### Общая диаграмма класса TcpClient

```kroki-mermaid
classDiagram
        class TcpClient {
        +Connect() void
        +TcpClient()
        +Action~string~ Log
    }

```

## Запуск сервера

В основной программе **базы данных** (`MainWindow.cs`) создадим функцию для запуска сервера.

Добавим кнопку для запуска в `Menu`:

```xml
<!--Добавьте этот MenuItem в ваше меню-->
<MenuItem Height="25" Header="Сервер">
    <MenuItem Header="Запустить" Click="StartServer_Click"/>
</MenuItem>
```

```csharp
//Обработчик нажатия на кнопку
private void StartServer_Click(object sender, RoutedEventArgs e)
{
	
}
```

Внутри этого обработчика следующий код:

```csharp
//Создаем объект сервера
server = new TcpServer();

//Для события Log назначим обработчик вывода в TextBlock
server.Log += (logString) =>
{
    Dispatcher.Invoke(() =>
    {
        LogTb.Text += ($"{logString}\n");
    });
};

//Для события MessageReceived назначим обработчик вывода в TextBlock
//Далее добавим еще функционал
server.MessageReceived += (message) =>
{
    Dispatcher.Invoke(() =>
    {
        LogTb.Text += ($"{message}\n");
    });
};

//Запускаем сервер в потоке
Task.Run(() =>
    {
        server.StartServer();
    }
);
```

## Запуск клиента

В основной программе **клиента** (`MainWindow.cs`) создадим функцию для подключения к серверу.

Добавим кнопку для подключения в `Menu`:

```xml
<!--Добавьте этот MenuItem в ваше меню-->
<MenuItem Header="Сеть">
    <MenuItem Header="Подключится к серверу" Click="ConnectToServer_Click"/>
</MenuItem>
```

```csharp
private void ConnectToServer_Click(object sender, RoutedEventArgs e)
{

}
```

Внутри этого обработчика

```csharp
//Создаем объект клиента
client = new Client();

//Добавим обработчик для вывода сообщений
client.Log += (message) =>
{
    LogTextBlock.Text += $"{message}\n";
};

//Подключаемся к серверу
client.TryConnect();
```

## Демонстрация соединения

Теперь мы можем запустить **сервер** и подключится к нему **клиентом**.

![Пример подключения](../img/7-laba-server-client-stream.gif)

## Связь сервера и базы данных

Все [методы для взаимодействия с нашей базой данных](../../main-pages/6-laba/#_16) у нас уже реализованы. Наша задача сводится к тому, чтобы наш **клиент** мог говорить **серверу** что ему нужно сделать и передать аргументы (если потребуется).

### Предполагаемый алгоритм может быть следующим:
1. **Клиент** формирует запрос (название операции и аргумент) в формате JSON;
2. **Клиент** отправляет этот запрос на **сервер**;
3. Сервер выполняет операцию;
4. Отправляет ответ с результатом операции;
5. Клиент "реагирует" на ответ (выводит на экран список комнат, например).

## Модернизация сервера

### Бесконечный цикл

Для того, чтобы сервер не прекращал работу после одного ответа, нам нужно добавить в работу сервера бесконечный цикл, который постоянно будет ждать запрос от клиента:

```csharp
try
{
	while(true)
	{
	//Весь код для сервера, описанный выше, находится тут
	}
}
catch (Exception e) //Если вдруг было брошено исключение
{
    Log?.Invoke("Ошибка: " + e.Message); //Выводим в лог текст ошибки этого исключения
}
finally //Блок кода, выполняющийся всегда после try-catch или исключения
{
    // 6. Закрытие соединений и остановка сервера
    stream?.Close(); // с помощью ? мы проверяем переменную на "равность" null
    //if(stream == nul) - аналог
    client?.Close();
    server?.Stop();
    Log?.Invoke("Сервер остановлен.");
}
```

Теперь наш сервер будет будет:

1. Открывает прослушивание порта;
2. Принимает подключение клиента;
3. Получает сообщение от клиента;
4. Сразу отправляет ответ клиенту;
5. Закрывает подключение.

Только этот блок операций теперь находится в бесконечном цикле и работает пока мы не остановим программу.

### Класс Запроса-Ответ

Для удобства работы с запросами и ответами сервера реализуем 2 класса:

```csharp
public class Request
{
    public string MethodName { get; set; }
    public string PayloadJson { get; set; } // Аргументы метода, сериализованные в JSON
}

public class Response
{
    public bool IsSuccess { get; set; }
    public string ErrorMessage { get; set; }
    public string ResultJson { get; set; } // Результат выполнения метода, сериализованный в JSON
}
```

Это нужно сделать в обоих проектах: **базе данных** и **клиенте**.

В классе `Request` мы описываем свойства для запроса от **клиента** к **серверу**:

- `MethodName` - название операции, которую должен выполнить сервер.
- `PayloadJson` - аргумент, преобразованный в `JSON`, в нашем случае это может быть объект `Room`.

В классе `Response` описываются свойства ответа от сервера:

- `IsSuccess` - логическое значение результата операции. Если результат будет успешным - будет `true`, иначе `false`.
- `ErrorMessage` - сообщение об ошибке, если результат не будет успешным.
- `ResultJson` - результат выполнения операции, в `JSON`.

#### Пример №1:

Вызов метода для получения всех квартир:
```json
{"MethodName":"GetAparts","PayloadJson":null}
```

Пример ответа от сервера:
```json
{
  "IsSuccess": true,
  "ErrorMessage": null,
  "ResultJson": [
    {
      "Id": 1,
      "Number": 1,
      "Description": "Описание для квартиры 1"
    },
    {
      "Id": 2,
      "Number": 2,
      "Description": "Описание для квартиры 2"
    },
    {
      "Id": 3,
      "Number": 3,
      "Description": "Описание для квартиры 3"
    }
  ]
}
```

Именно `ResultJson` мы можем преобразовать в список нужных нам объектов.


#### Пример №2

Запрос на добавление новой комнаты:

```json
{
  "MethodName": "InsertRoom",
  "PayloadJson": "{\"Name\":\"Название комнаты\",\"Temperature\":20,\"IsLightOn\":false,\"Area\":0,\"Id\":0,\"ApartId\":3,\"ApartDescription\":\"Описание для квартиры 3\"}"
}
```

Пример ответа от сервера:

```json

{
  "IsSuccess": true,
  "ErrorMessage": null,
  "ResultJson": "{\"Name\":\"Название комнаты\",\"Temperature\":20,\"IsLightOn\":false,\"Area\":0,\"Id\":9,\"ApartId\":3}"
}

```


### Обработка запросов сервером

Часть с запуском сервера и получением запроса от клиента остается без изменений:

```csharp
// 1. Запуск сервера
Log?.Invoke("Сервер запущен и ждет подключения...");

// 2. Ожидание подключения клиента (блокирует выполнение)
client = server.AcceptTcpClient();
Log?.Invoke($"Клиент {client.Client.LocalEndPoint} подключен!");

// 3. Получение потока для обмена данными
stream = client.GetStream();
StreamReader reader = new StreamReader(stream, Encoding.UTF8);
StreamWriter writer = new StreamWriter(stream, Encoding.UTF8);

// 4. Читаем JSON-запрос от клиента (ОЖИДАЕМ одну строку)
string requestJson = reader.ReadLine();
if (string.IsNullOrEmpty(requestJson))
{
    Log?.Invoke("Получен пустой запрос от клиента.");
    return;
}

Log?.Invoke($"Получен JSON запрос: {requestJson}");

Request clientRequest = null;
Response serverResponse = new Response();
```

Далее нужно добавить обработчик, преобразует запрос в объект:

```csharp
try
{
    clientRequest = JsonSerializer.Deserialize<Request>(requestJson);
	
	// Здесь будет еще код самой обработки
	// 4. Обрабатываем запрос в зависимости от MethodName
    
}
catch (JsonException jsonEx)
{
    Log?.Invoke($"Ошибка десериализации JSON: {jsonEx.Message}");
    serverResponse.IsSuccess = false;
    serverResponse.ErrorMessage = "Ошибка формата JSON запроса: " + jsonEx.Message;
}
catch (Exception ex) // Другие ошибки при обработке
{
    Log?.Invoke($"Ошибка обработки запроса '{clientRequest?.MethodName}': {ex.Message}");
    serverResponse.IsSuccess = false;
    serverResponse.ErrorMessage = "Внутренняя ошибка сервера: " + ex.Message;
}

// 5. Отправляем JSON-ответ клиенту
string responseJson = JsonSerializer.Serialize(serverResponse);
writer.WriteLine(responseJson);
writer.Flush();
Log?.Invoke($"Отправлен JSON ответ: {responseJson}");
```

Код обработки и выбора нужного метода

```csharp
// 4. Обрабатываем запрос в зависимости от MethodName
    switch (clientRequest.MethodName)
    {
        case "InsertRoom":
            Room roomToInsert = JsonSerializer.Deserialize<Room>(clientRequest.PayloadJson);

            Room insertedRoom = DatabaseManager.AddRoom(roomToInsert).Result;

            serverResponse.ResultJson = JsonSerializer.Serialize(insertedRoom);
            serverResponse.IsSuccess = true;

            UpdateUI?.Invoke();

            break;

        case "UpdateRoom":

            Room roomToUpdate = JsonSerializer.Deserialize<Room>(clientRequest.PayloadJson);

            Room updatedRoom = DatabaseManager.UpdateRoom(roomToUpdate).Result;

            serverResponse.ResultJson = JsonSerializer.Serialize(updatedRoom);
            serverResponse.IsSuccess = true;
            
            UpdateUI?.Invoke();

            break;

        case "GetAparts":

            var tempAparts = DatabaseManager.GetAllApartments().Result;

            serverResponse.ResultJson = JsonSerializer.Serialize(tempAparts);
            serverResponse.IsSuccess = true;

            break;

        case "GetRooms":

            var tempRooms = DatabaseManager.GetAllRooms().Result;

            serverResponse.ResultJson = JsonSerializer.Serialize(tempRooms);
            serverResponse.IsSuccess = true;

            break;


        default:
            serverResponse.IsSuccess = false;
            serverResponse.ErrorMessage = $"Неизвестный метод: {clientRequest.MethodName}";
            break;
    }
```

После этого мы отправляем клиенту ответ с нужными данными:

```csharp
// Этот код уже есть выше, он повторяется для последовательного объяснения.
// 5. Отправляем JSON-ответ клиенту
string responseJson = JsonSerializer.Serialize(serverResponse);
writer.WriteLine(responseJson);
writer.Flush();
Log?.Invoke($"Отправлен JSON ответ: {responseJson}");
```

## Формирование запроса на клиенте (GetRooms)

Начало функции:

```csharp
public ObservableCollection<Room> GetRooms() //Указываем, что функция вернет коллекцию комнат.
{

ObservableCollection<Room> insertedRoom = null;

//Дальше код с подключением и возврат значения.

}

```

Начало подключения остается таким же:

```csharp
// 1. Подключение к серверу
client = new TcpClient("127.0.0.1", 8888);
Log?.Invoke("Клиент подключен к серверу.");

// 2. Получение потока для обмена данными
stream = client.GetStream();
StreamReader reader = new StreamReader(stream, Encoding.UTF8);
StreamWriter writer = new StreamWriter(stream, Encoding.UTF8);
```

Далее мы объявляем и инициализируем переменную типа `Request`:

```csharp
// 3. Объявляем переменную типа Request и задаем нужные свойства
// Так как метод GetRooms ничего не передает на сервер - нагрущка (Payload) будет null
Request insertRequest = new Request
{
    MethodName = "GetRooms",
    PayloadJson = null
};
string insertRequestJson = JsonSerializer.Serialize(insertRequest);
//Сериализуем переменную в текстовое представление в формате JSON
```

Далее мы отправляем запрос на сервер:

```csharp
// 4. Записываем информацию в writer и отправляем
Log?.Invoke($"Клиент: Отправка запроса: {insertRequestJson}");
writer.WriteLine(insertRequestJson);
writer.Flush();
```

Получаем ответ от сервера:

```csharp
// 5. Ждем ответ от сервера в reader
string insertResponseJson = reader.ReadLine();
Log?.Invoke($"Клиент: Получен ответ: {insertResponseJson}"); //Выводим что получили в ответе

Response insertResponse = JsonSerializer.Deserialize<Response>(insertResponseJson);
//Тут мы используем операцию десериализацию - перевод из представления текстового в объект класса Response

if (insertResponse.IsSuccess) // Если сервер сказал что успешно
{
    insertedRoom = JsonSerializer.Deserialize<ObservableCollection<Room>>(insertResponse.ResultJson);
// Десериализуем объект ResultJson в ObservableCollection объектов Room.
// Сервер отдал нам список всех комнат в виде текста, находящихся в базе данных. 
// Мы преобразуем их в коллекцию типа комнаты и зададим в нашу программу.
}
else
{
    Log?.Invoke($"Ошибка добавления: {insertResponse.ErrorMessage}");
}
```

В самом конце возвращаем `insertedRooms`:

```csharp
return insertedRoom;
```

Пример вызова такой функции:

```csharp
private void ConnectToServer_Click(object sender, RoutedEventArgs e)
{
    client = new Client();

    client.Log += (message) =>
    {
        LogTextBlock.Text += $"{message}\n";
    };

    Aparts = client.GetAparts(); //Возвращает список Apartment
    Rooms = convertRoomsToRoomControls(client.GetRooms()); //Возвращает список Apartment
}
```

Так как метод возвращает список `Room`, а в программе мы используем `RoomControl` - нам нужно сконвертировать значения:

```csharp
private ObservableCollection<RoomControl> convertRoomsToRoomControls(ObservableCollection<Room> newRooms)
{
    ObservableCollection<RoomControl> listToReturn = new ObservableCollection<RoomControl>();
    foreach (Room r in newRooms)
    {
        r.ApartDescription = Aparts.First(x => x.Id == r.ApartId).Description;
        r.OnRoomChanged += UpdateRoomOnServer;
        RoomControl newRoomControl = new RoomControl(r);
        newRoomControl.RemoveControl += RemoveRoomControlUI;
        listToReturn.Add(newRoomControl);
    }
    return listToReturn;
}
```

### Пример выполнения запроса:

![Пример запроса получить все комнаты](../img/7-laba-sample-getrooms.gif)


## Пример AddRoom

Запрос клиента выглядит так:

```csharp
public void InsertRoom(Room room)
{
    try
    {
        // 1. Подключение к серверу
        client = new TcpClient("127.0.0.1", 8888);
        Log?.Invoke("Клиент подключен к серверу.");

        // 2. Получение потока для обмена данными
        stream = client.GetStream();
        StreamReader reader = new StreamReader(stream, Encoding.UTF8);
        StreamWriter writer = new StreamWriter(stream, Encoding.UTF8);

		// 3. Формируем запрос, указываем метод и нагрузку (нужную комнату)
        Request insertRequest = new Request
        {
            MethodName = "InsertRoom",
            PayloadJson = JsonSerializer.Serialize(room)
        };
        string insertRequestJson = JsonSerializer.Serialize(insertRequest);

		// 4. Отправляем запрос
        Log?.Invoke($"Клиент: Отправка запроса: {insertRequestJson}");
        writer.WriteLine(insertRequestJson);
        writer.Flush();

		// 5. Получаем ответ и преобразуем в объект Response
        string insertResponseJson = reader.ReadLine();
        Log?.Invoke($"Клиент: Получен ответ: {insertResponseJson}");
        Response insertResponse = JsonSerializer.Deserialize<Response>(insertResponseJson);

        if (insertResponse.IsSuccess) // Если сервер сказал "все хорошо"
        {
	        //Десериализуем вставленную комнату из БД, так как в ней содержится Id
            Room insertedRoom = JsonSerializer.Deserialize<Room>(insertResponse.ResultJson);
            room = insertedRoom; //Перезаписываем старую переменную, чтобы Id был из базы данных

            Log?.Invoke($"Успешно добавлена комната: Id={insertedRoom.Id}, Name='{insertedRoom.Name}'");
        }
        else
        {
            Log?.Invoke($"Ошибка добавления: {insertResponse.ErrorMessage}");
        }


    }
    catch (Exception e)
    {
        Log?.Invoke("Ошибка: " + e.Message);
    }
    finally
    {
        // 6. Закрытие соединений
        stream?.Close();
        client?.Close();
        Log?.Invoke("Клиент отключен.");
    }

}
```

### Пример выполнения запроса:

![Пример добавления комнаты](../img/7-laba-sample-addRoom.gif)

## Пример работы программы

![Пример итоговый](../img/7-laba-sample-total.gif)