---
title:  "Unity3d: Атрибут для отображения полей с интерфейсом"
repository: Unity3d-Attributes 
preview: /assets/images/posts/2023-03-15-unity3d-require-type-attribute/preview.jpg
date:   2023-03-15 10:00:00 +0300
categories: ru cases
tags: [c#, unity3d]
lang: Ru
layout: post
---

В сети уже есть немало решений по визуализации полей с интерфейсом в инспекторе. Но те что мне попадались, позволяли добавить в поле либо только компонент, либо только объект с подходящим компонетом, как раз это я постарался исправить, сделав этот аргумент более универсальным.

## Реализация
Начнем с класса атрибутов. Он выглядит примерно одинаково во всех реализациях.
```csharp
[System.AttributeUsage(System.AttributeTargets.Field)]
public class RequireTypeAttribute : PropertyAttribute
{
    // Тип интерфейса.
    public System.Type RequiredType { get; }

    public RequireTypeAttribute(System.Type type) => this.RequiredType = type;
}
```
В качестве аргумента принимает тип инкапсулируемого объекта. Иногда присуствует флаг для разрешения назначения объектов сцены. У нас эта возможность будет по умолчанию.

Самая интересная "магия" будет происходить в классе драйвер, конкретно в методе `OnGUI()`.
```csharp
[CustomPropertyDrawer(typeof(RequireTypeAttribute))]
public class RequireTypeDrawer : PropertyDrawer
{
    public override void OnGUI(Rect position, SerializedProperty property, GUIContent label)
    {
        // Проверяет, является ли это свойством ссылочного типа
        if (property.propertyType != SerializedPropertyType.ObjectReference)
        {
            // Если поле не является ссылкой, показать сообщение об ошибке            
            var style = new GUIStyle(EditorStyles.objectField);
            style.normal.textColor = Color.red;

            EditorGUI.LabelField(position, label, new GUIContent($"{ property.propertyType } is not a reference type"), style);
            return;
        }

        var requiredType = ((RequireTypeAttribute)attribute).RequiredType;
        ChecDragAndDrops(position, requiredType);
        DrawObjectField(position, property, requiredType, label);
        ChecValues(property, requiredType);
    }
}
```
В методе `OnGUI()` проверяем, предназначено ли поле для ссылочного типа или нет. Если нет - показываем ошибку и выходим из метода.

Далее в этом же методе вызываем метод который осуществляет проверку объектов, перемещаемых в область предназначеную для нашего поля.
```csharp
private void ChecDragAndDrops(Rect position, System.Type requiredType)
{
    // Если курсор находится в области отображаемого поля
    if (position.Contains(Event.current.mousePosition))
    {
        // Перебрать все перетаскиваемые ссылки
        foreach (var @object in DragAndDrop.objectReferences)
        {
            // Если мы не найдем нужный тип
            if (!IsValidObject(@object, requiredType))
            {
                // Отключить перетаскивание
                DragAndDrop.visualMode = DragAndDropVisualMode.Rejected;
                break;
            }
        }
    }
}
```
Метод сначала проверяет, попадает ли курсор при перемещении объектов в область поля, и далее в цикле проверяет их на соответствие заданному интерфейсу. Если хоть один из них не подходит под указанный интерфейс - добавление объектов в поле запрещается.

Метод для проверки на соответствие типу сначала сверяет объект с `GameObject`, если это он, то с помощью метода `GetComponent()`, пытаемся найти объект заданного интерфейса и сравниваем его с `null`.
```csharp
private bool IsValidObject(Object @object, System.Type requiredType)
{
    // Если объект является GameObject
    if (@object is GameObject go)
    {
        // Проверить, есть ли у него компонент нужного типа и вернуть результат
        return go.GetComponent(requiredType) != null;
    }

    // Проверить саму ссылку на соответствие требуемому типу
    return requiredType.IsAssignableFrom(@object.GetType());
}
```
Если это был не `GameObject`, то мы просто проверяем, возможно ли привести его к нужному интерфейсу и возвращаем это как результат.

В методе `DrawObjectField()`, как можно догататься, отрисовываем поле для нашего объекта.
```csharp
private void DrawObjectField(Rect position, SerializedProperty property, System.Type requiredType, GUIContent label)
{
    // Начать проверку изменений
    EditorGUI.BeginChangeCheck();
    // Отобразить поле объекта
    EditorGUI.ObjectField(position, property, label);
    // Если в содержимое поля были внесены изменения и в поле был добавлен GameObject
    if (EditorGUI.EndChangeCheck() && property.objectReferenceValue is GameObject @object)
    {
        // Получить компонент нужного типа на объекте и сохранить ссылку на него в свойстве
        property.objectReferenceValue = @object.GetComponent(requiredType);
    }
}
```
Если перемещаемый объект был `GameObject` - с помощью `GetComponent()` находим компонет подходящий под наш интерфейс.

Последний метод был добавлен на случай, если объекты были добавлены через код, но без соответствующего интерфейса.
```csharp
private void ChecValues(SerializedProperty property, System.Type requiredType)
{
    // Если ссылка не нулевая и указывает на объект, который не соответствует типу
    if (property.objectReferenceValue != null && !IsValidObject(property.objectReferenceValue, requiredType))
    {
        // Удалить ссылку
        property.objectReferenceValue = null;
    }
}
```

## Использование
Чтобы воспользоваться этим атрибутом необходимо объявить поле Object, а в атрибут передать требуемый тип интерфейса
```csharp
[SerializeField, RequireType(typeof(IExampleInterface))] private Object exampleObject;
```

## Заключение
С помощью  этого атрибута можно изменять содержимое поля, перетаскивая как целые объекты из окна иерархии, так и отдельные компоненты из инспектора, так же поле может работать со `ScriptableObject` и с массивом объектов.

Этот и другие атрибуты Вы можете использовать в своем проекте, добавив [этот репозиторий](https://github.com/{{ site.github.owner_name }}/{{ page.repository }}) как [подмодуль](https://git-scm.com/book/en/v2/Git-Tools-Submodules):

    git submodule add https://github.com/{{ site.github.owner_name }}/{{ page.repository }}

Спасибо за внимание :)
