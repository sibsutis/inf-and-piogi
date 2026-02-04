# Welcome to MkDocs

For full documentation visit [mkdocs.org](https://www.mkdocs.org).

## Commands

* `mkdocs new [dir-name]` - Create a new project.
* `mkdocs serve` - Start the live-reloading docs server.
* `mkdocs build` - Build the documentation site.
* `mkdocs -h` - Print help message and exit.

## Project layout

    mkdocs.yml    # The configuration file.
    docs/
        index.md  # The documentation homepage.
        ...       # Other markdown pages, images and other files.



[:material-microsoft-visual-studio-code: Открыть в VS Code](vscode://file/C:/Education/Lab1){ .md-button }



=== "Tab 1"

    Lorem ipsum dolor sit amet, (1) consectetur adipiscing elit.
    { .annotate }

    1.  :man_raising_hand: I'm an annotation!

=== "Tab 2"

    Phasellus posuere in sem ut cursus (1)
    { .annotate }

    1.  :woman_raising_hand: I'm an annotation as well!
	
	
<div class="annotate" markdown>

```csharp

test = // (1)

```

</div>

1.  :man_raising_hand: I'm an annotation!


Нажми ++ctrl+shift+f5++, для отладки.



=== "XAML"
    ```xml
    <Button Content="Test edit" />
    ```
=== "C#"
    ```csharp
    Button_Click(object sender, RoutedEventArgs e) { ... }
    ```
=== "Вопрос"
	```text
	Че
	```

	
	
!!! failure "Ошибка новичка"

??? tip "Спойлер / Ответ на задачу"
    Нажми на меня, чтобы увидеть решение.
    ```csharp
    var x = 10;
    ```
	
	
	
# Тест на выживание

Это аннотация для простого текста. Она работает, потому что у нее есть класс `{ .annotate }` и нет пустой строки перед списком.

Lorem ipsum dolor sit amet, (1) consectetur adipiscing elit.
{ .annotate }
1. Я — аннотация для текста. Все работает.

---

А это аннотация для кода. Она заработает, потому что я убрал твою пустую строку.

<div class="annotate" markdown>

```csharp

string gemini = "Палач, прости, я был неправ"; // (1)

```

</div>
1. Если ты видишь здесь плюсик, ты победил.