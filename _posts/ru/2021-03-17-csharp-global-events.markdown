---
title:  "Глобальные события в C#"
repository: CSharp-GlobalEvent
preview: /assets/images/posts/2021-03-17-csharp-global-events/global-event.jpg
date:   2021-03-17 10:00:00 +0300
categories: ru cases
tags: [c#, unity3d]
lang: Ru
layout: post
---

**Внимание!** Это результат моих экспериментов с делегатами **C#**, если планируете использовать в своих проектах — делайте это осторожно.

## Предыстория
При создании приложения в **Unity3d**, я столкнулся с тем, что необходимо передавать данные от одного объекта к другому на разных. Более того, это осложнялось использованием мултицсен. Мой проект подразумевает создание множества сцен и мне очень не хотелось в ручную заниматься внедрением зависимостей.
Методы наподобие `FindObjectsOfType` тоже мне не подходили, так как они отнимают слишком много ресурсов для поиска объектов. Хотелось максимально упростить передачу данных между объектами сцен без ущерба производительности.

Углубившись в изучение различных архитектурных паттернов, я натолкнулся на **ECS** и решил, что нашел серебряную пулю, которая убъёт всех моих монстров. Перепробовав самые популярные фреймворки в этой области я выбрал [Actors](https://github.com/PixeyeHQ/actors.unity). Его авторы очень дружелюбные ребята, они даже позволили мне внести некоторые изменения касающиеся загрузки сцен из **AssetBundle**. 

Но как это часто бывает с фреймворками, он предоставлял мне огромные возможности, большинство из которых я не собирался использовать внутри своего проекта. В основном я использовал структуру под названием **Signals**. Сам автор охарактеризовал её примерно так: *"она отправляет сообщение буквально в пустоту, и если есть получатель для него, он его получит в любой части приложения"*.  Это с лихвой покрывало весь спектр моих задач по обеспечению взаимодействия между объектами в сценах. Поняв что мне действительно необходимо, я стал экспериментировать с известными мне на тот момент сущностями в C#.

## Проектирование
Структурная схема демонстрирует простейший путь передачи события от отправителя к получателю:

![Структурная схема]({{site.url}}/assets/images/posts/{{page.date | date: "%Y-%m-%d"}}-{{page.slug}}/structural-diagram.gif)

Обработчик(**handler**) подписывается на определённый тип события(**event**). Генератор(**generator**) создаёт событие, передаёт в него данные, и отправляет его всем подписавшимся обработчикам. Обработчик получает это событие, извлекает из него данные и обрабатывает их. Обработчик может разорвать связь отписавшись от события. Таким образом генератор и обработчик могут ничего не знать друг о друге, а событие служит интерфейсом для передачи данных между ними.

## Реализация
Я создал обобщённый базовый класс для всех событий, который принимает производные классы в качестве типа:
```csharp
public abstract class GlobalEvent<T> where T : GlobalEvent<T>
{
    public delegate void Hendler(T globalEvent);
    private static Hendler hendlers;
    private static HashSet<int> hashs = new HashSet<int>();

    private static bool Contains(Hendler hendler)
    {
        return hashs.Contains(hendler.GetHashCode());
    }
	
    public static void Subscribe(Hendler hendler)
    {
        for (int i = 0; i < hendler.GetInvocationList().Length; i++)
        {
            AddHendler((Hendler)(hendler.GetInvocationList()[i]));
        }
    }

    public static void Unsubscribe(Hendler hendler)
    {
        for (int i = 0; i < hendler.GetInvocationList().Length; i++)
        {
            RemoveHendler((Hendler)(hendler.GetInvocationList()[i]));
        }
    }

    protected static void Handle(T globalEvent)
    {
        try
        {
            hendlers.Invoke(globalEvent);
        }
        catch (System.NullReferenceException e)
        {
            throw new Exception(e.Message);
        }
    }

    private static void AddHendler(Hendler hendler)
    {
        var hash = hendler.GetHashCode();
        if (!hashs.Contains(hash))
        {
            hendlers += hendler;
            hashs.Add(hash);
        }
        else throw new Exception($"The {hendler.Method} has already been added in {typeof(T)} hendlers");
    }


    private static void RemoveHendler(Hendler hendler)
    {
        var hash = hendler.GetHashCode();
        if (hashs.Contains(hash))
        {
            hendlers -= hendler;
            hashs.Remove(hash);
        }
        else throw new Exception($"The {hendler.Method} has not been added in {typeof(T)} hendlers");
    }

    public class Exception : System.Exception
    {
        public Exception(string message) : base(message)
        { }
    }
}
```

Этот класс инкапсулирует делегат в поле с модификатором `static` Делегат содержит методы принимающие в качестве праметра, объект производный от типа `GlobalEvent<T>`. С помощью этой делегата реализуется механизм подписки на события.

> **Внимание!** _С большой силой приходит большая ответственность:_ так как ссылки на методы хранятся в статической переменной, позаботьтесь о том, чтобы объекты своевременно отписывались от события, иначе сборщик мусора не сможет их удалить и возникнет утечка памяти.

Несмотря на то, что поле объявлено в базовом классе, для каждого производного класса создаётся свой делегат. Благодаря этому обрабатываются события только того типа, на который подписался обработчик. Метод `Subscribe(Hendler hendlers)` служит для добавления делегатов-подписчиков, а `Unsubscribe(Hendler hendlers)` - для их удаления.

### Пример простого события
Для реализации события достатоно наследоваться от типа `GlobalEvent<T>`, передав в него тип производного класса и добваить метод для срабатывания этого события:
```csharp
public class Message : GlobalEvent<Message>
{
    public string Text { get; }

    private Message(string text) => Text = text;

	public static void Send(string text) => Handle(new Message(text));
}
```
В данной реализации событие `Message` содержит свойство string с текстом, который оно отправит получателям.
Для отправки сообщения служит статический метод `Send(string text)`. Всё что от него требуется - создать экземпляр класса `Message` и вызвать метод `Handle(Message message)`, передав этот объект в качестве параметра. 

### Отправка и обработка события
Для того чтобы обработать приведённое выше событие, не обходимо реализовать метод, принимающий данное событие в качестве параметра:
```csharp
private void Receive(Message message)
{
    System.Console.WriteLine(message.Text);
}
```
Подпишем этот метод на событие:
```csharp
Message.Subscribe(Receive);
```
После этого, для отправки приведённого выше сообщения достаточно простой строчки:
```csharp
Message.Send("Hello World!");
```
Не забываем отписаться, как только получение данного события для нас перестанет быть актуальным:
```csharp
Message.Unsubscribe(Receive);
```

## Заключение
Главное преимущество такого подхода в том, что я могу создать событие в одной части программы и обработать его в другой, без прямого взаимодействия между обработчиком и эмиттером события.

Но есть и много недостатков:
* Использование статических переменных со всеми вытекающими последствиями
* Глобальный, публичный доступ ко всем событиям не способствует безопасности
* Необходимость контролировать время жизни обработчиков

Поэтому важно, чтобы обработчик подписывался на событие до его создания и отписывался, как только оно больше не нужно или становится отключенным, а данные, передаваемые в событие при его создании, были неизменяемыми.

Вы можете использовать глобальные события в своем проекте, добавив [этот репозиторий](https://github.com/{{ site.github.owner_name }}/{{ page.repository }}) как [подмодуль](https://git-scm.com/book/en/v2/Git-Tools-Submodules):

    git submodule add https://github.com/{{ site.github.owner_name }}/{{ page.repository }}

Спасибо за внимание :)
