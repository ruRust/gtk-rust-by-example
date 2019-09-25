# Программирование UI

На этом этапе, мы сможем соединить всё вместе. Сначала мы установим стандартное значение здоровья для программы. Это значение будет использоваться для инициализации состояния структуры приложения. Затем, мы напишем код для кнопки удара и лечения, которые будут должны изменять значение содержимого в главном окне.

## Перед тем, как мы начнём

В нашем распоряжении будет несколько строк, которые будут использованы в зависимости от действия. Это массив **MESSAGES**, к которому мы будем обращаться с помощью типажа с типом **u8**, который будет использован для получения индексов в массиве.

```rust
// Заданные сообщения, которые будут использоваться в UI
// при определённых условиях.
const MESSAGES: [&str; 3] = ["Ой! Ты ударил меня!", "...", "Спасибо!"];

#[repr(u8)]
// Типаж с типом `u8`, который используется как индекс в массиве `MESSAGES`.
enum Message { Hit, Dead, Heal }
```

Для тех, кто плохо разбирается в Rust, атрибут `#[repr(u8)]` определяет, что следующие элементы будут представлены типом **u8** в памяти. По умолчанию, варианты для типажей начинаются с нуля, поэтому **Hit** это `0`, тогда как **Heal** это `2`. Если вы хотите сделать это явным, вы можете написать это как:

```rust
#[repr(u8)]
enum Message { Hit = 0, Dead = 1, Heal = 2 }
```

## Инициализация компонента Health и структурирование приложения

После инициализации GTK, мы можем создать наш компонент `health`, который будет обёрнут внутри атомарного счётчика (**Arc**). Если вы запомнили предыдущий код, то на самом деле внутреннее значение это **AtomicUsize**, который служит нашим счетчиком `health`. Это значение будет передаваться через несколько замыканий, следовательно требуется для счётчика ссылок.

```rust
let health = Arc::new(HealthComponent::new(10));
```

Используя это значение, мы создадим структуру UI нашего приложения. Обратите внимание, что `&health` автоматически ссылается как **&HealthComponent**, даже если завёрнут в **Arc**.

```rust
let app = App::new(&health);
```

## Запрограммируем кнопку удара

Находясь здесь, всё что нам надо - это написать код наших виджетов. Именно здесь мы будем передавать оба компонента `health` и другие различные виджеты UI через замыкания. Начнём с кнопки лечения. Нам просто нужно сказать программе: "Что произойдет при нажатии на кнопку" ?
Типаж **ButtonExt** предоставляет метод **connect_clicked()** именно для этого.

> Обратите внимание, что виджеты в GTK обычно проходят через их замыкания, поэтому, если
> вы хотите управлять вызовом виджета, вы можете сделать это используя выбранное значение
> через замыкание. Мы не нуждаемся в этой функциональности, поэтому просто проигнорируем
> значение.
> ```rust
> widget.connect_action(move |widget| {});
> ```

```rust
{
    // Запрограммируем кнопку `Ударить` чтобы уменьшить здоровье.
    let health = health.clone();
    let message = app.content.message.clone();
    let info = app.content.health.clone();
    app.header.hit.clone().connect_clicked(move |_| {
        let new_health = health.subtract(1);
        let action = if new_health == 0 { Message::Dead } else { Message::Hit };
        message.set_label(MESSAGES[action as usize]);
        info.set_label(new_health.to_string().as_str());
    });
}
```

В коде выше, мы создали анонимную область, чтобы мы могли содержать наши клонированные ссылки.
Каждый вызов **clone()** просто увеличивает счётчик ссылок и делает значение доступным, чтобы использовать его еще раз позже.

После вычитания из компонента health, если health равен `0`, то мы должны вернуть **Message::Dead**, иначе, сообщением будет **MessageHit**. После того, как мы овладели этой информацией, это просто вопрос обновления метки с новым значением.

## Запрограммируем кнопку лечения

Это работает почти также, поэтому мы можем скопировать и вставить код выше, а затем изменить его, чтобы удовлетворить наши потребности.

```rust
{
    // Запрограммируем кнопку `Лечить`, чтобы вернуть очки здоровья.
    let health = health.clone();
    let message = app.content.message.clone();
    let info = app.content.health.clone();
    app.header.heal.clone().connect_clicked(move |_| {
        let new_health = health.heal(5);
        message.set_label(MESSAGES[Message::Heal as usize]);
        info.set_label(new_health.to_string().as_str());
    });
}
```

## В общей сложности

После программирования UI, вы можете завершить код, выполнив следующее:

```rust
// Сделаем все виджеты видимыми в UI.
app.window.show_all();

// Запуск основного цикла GTK.
gtk::main();
```

Ваш исходный код должен быть таким:

```rust
// Заданные сообщения, которые будут использоваться в UI
// при определённых условиях.
const MESSAGES: [&str; 3] = ["Ouch! You hit me!", "...", "Thanks!"];

#[repr(u8)]
// Типаж с типом `u8`, который используется как индекс в массиве `MESSAGES`.
enum Message { Hit, Dead, Heal }

fn main() {
    // Инициализируем GTK перед продолжением.
    if gtk::init().is_err() {
        eprintln!("failed to initialize GTK Application");
        process::exit(1);
    }

    /*  Установим начальное состояние для нашего компонента - `health`.
     *   Воспользуемся `Arc`, для того, чтобы мы могли
     *   использовать несколько programmable замыканий.
     */
    let health = Arc::new(HealthComponent::new(10));

    // Инициализируем начальное состояние UI.
    let app = App::new(&health);

    {
        // Запрограммируем кнопку `Ударить` чтобы уменьшить здоровье.
        let health = health.clone();
        let message = app.content.message.clone();
        let info = app.content.health.clone();
        app.header.hit.clone().connect_clicked(move |_| {
            let new_health = health.subtract(1);
            let action = if new_health == 0 { Message::Dead } else { Message::Hit };
            message.set_label(MESSAGES[action as usize]);
            info.set_label(new_health.to_string().as_str());
        });
    }

    {
        // Запрограммируем кнопку `Лечить`, чтобы вернуть очки здоровья.
        let health = health.clone();
        let message = app.content.message.clone();
        let info = app.content.health.clone();
        app.header.heal.clone().connect_clicked(move |_| {
            let new_health = health.heal(5);
            message.set_label(MESSAGES[Message::Heal as usize]);
            info.set_label(new_health.to_string().as_str());
        });
    }

    // Сделаем все виджеты видимыми в UI.
    app.window.show_all();

    // Запуск основного цикла GTK.
    gtk::main();
}
```