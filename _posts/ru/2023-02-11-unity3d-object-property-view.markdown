---
title:  "Unity3d: Отображение свойств объекта"
repository: Unity3d-ExtendedEditor 
preview: /assets/images/posts/2023-02-11-unity3d-object-property-view/preview.png
date:   2023-02-11 10:00:00 +0300
categories: ru cases
tags: [c#, unity3d]
lang: Ru
layout: post
---

Работая с полями для объектов в **Unity3d**, в какой-то момент понимаешь, что было бы не плохо изменять сериализованые свойства объекта непосредственно в окне инспектора объекта, который его инкапсулирует. Реализовать такую матрёшку можно с помощью кастомного редактора.

## Реализация
Во-первых нам понадобится класс, который будет наследоваться от `Editor`.
```csharp
public abstract class ExtendedEditor : Editor
{
	public static void DisplayMessage(string message, MessageType messageType = MessageType.None)
    {
        // Create content from the icon and text of the message
        GUIContent label = new GUIContent(message);
        switch (messageType)
        {
            case MessageType.Info: { label.image = EditorGUIUtility.Load("icons/console.infoicon.png") as Texture2D; break; }
            case MessageType.Warning: { label.image = EditorGUIUtility.Load("icons/console.warnicon.png") as Texture2D; break; }
            case MessageType.Error: { label.image = EditorGUIUtility.Load("icons/console.erroricon.png") as Texture2D; break; }
        }

        // Define the message display style
        var style = new GUIStyle();
        style.wordWrap = true;
        style.normal.textColor = GUI.skin.label.normal.textColor;

        // Display message
        EditorGUILayout.LabelField(label, style);
    }
}
```
Я сделал его абстрактным, чтобы потом переиспользовать в производных классах. Кроме этого добавил в него метод для отображения сообщения об ошибке. В последующем класс можно будет рассширить новыми методами. 

>Внимание! Этот класс необходимо поместить в папку *Editor*, чтобы избежать ошибок при сборке проекта.

В созданный выше класс добавил внутренний класс который инкапсулирует `SerializedProperty` и `Editor` объекта, свойства которого мы хотим отображать, а так же предоставляет метод для отрисовки этих свойств в редакторе.
```csharp
private class ObjectPropertyView
{
    private GUIContent label;
    private SerializedProperty serializedProperty;
    private Editor editor;
    private bool foldout;

    public ObjectPropertyView(SerializedProperty property, GUIContent label = null)
    {
        this.label = (label != null) ? label : new GUIContent(property.name);
        this.serializedProperty = property;
        this.foldout = false;
        UpdateEditor();
    }

    public void Draw()
    {
        if (serializedProperty.propertyType != SerializedPropertyType.ObjectReference)
        {            
            var style = new GUIStyle(EditorStyles.objectField);
            style.normal.textColor = Color.red;

            // Отобразить сообщение об ошибке
            EditorGUILayout.LabelField(label, new GUIContent($"{serializedProperty.propertyType} is not a reference type"), style);
            return;
        }
		
        // Отрисовываем спойлер
        foldout = EditorGUILayout.Foldout(foldout, new GUIContent());

        EditorGUI.BeginChangeCheck();
        // Отрисовываем поле для сериализуемого объекта
        EditorGUI.ObjectField(GUILayoutUtility.GetLastRect(), serializedProperty, label);

        // Проверяем наличие изменений
        if (EditorGUI.EndChangeCheck() || (serializedProperty.objectReferenceValue ^ editor))
        {
            // Обовляем редактор для сериализуемого объекта
            UpdateEditor();
        }

        if (foldout)
        {
            GUILayout.BeginVertical(EditorStyles.helpBox);
            // Отображаем сериализованые свойства объекта или ошибку
            if (editor != null)
            {
                editor.OnInspectorGUI();
            }
            else
            {
                ExtendedEditor.DisplayMessage($"The {serializedProperty.displayName} field must not be empty!", MessageType.Error);
            }
            GUILayout.EndVertical();
        }
    }

    // Обновляет редактор сериализуемого объекта
    private void UpdateEditor()
    {
        if (serializedProperty.propertyType == SerializedPropertyType.ObjectReference)
        {
            editor = Editor.CreateEditor(serializedProperty.objectReferenceValue);
        }
    }
}
```
В конструкторе данного класса создаётся редактор, который будет отрисовывать свойства объекта в методе `Draw()`, сразу под полем с данным объектом. Для того чтобы скрыть свойства - используем `Foldout()`.
В случае каких либо изменений в поле, где хранится ссылка на объект, редактор пересоздаётся заново.

Далее в классе `ExtendedEditor`, объявляем список объектов `ObjectPropertyView` и методы для работы с ним.
```csharp
// List of prepared object property editors
private List<ObjectPropertyView> properties = new List<ObjectPropertyView>();

// Method for adding displayable serialized object properties
protected void AddPropertyView(SerializedProperty serializedProperty, GUIContent label = null)
{
    if (target.GetType().ToString().Equals(serializedProperty.type.TrimStart("PPtr <$".ToCharArray()).TrimEnd('>')))
    {
        throw new System.ArgumentException("The type of the field must be different from the type of the parent object, otherwise recursion occurs.");
    }
    properties.Add(new ObjectPropertyView(serializedProperty, label));
}

// Method for adding displayable serialized object properties
protected void AddPropertyView(SerializedProperty serializedProperty, string label) => AddPropertyView(serializedProperty, new GUIContent(label));

// Display the properties of objects added to the list
protected void DrawProperties()
{
    for (int i = 0; i < properties.Count; i++)
    {
        properties[i].Draw();
    }
}
```
Методы `AddPropertyView()` необходимы для добаления сериализованых свойств объектов для последующего отбражения.
Метод `DrawProperties()` отрисовывает свойства объектов из списка с `ObjectPropertyView`.

В завершении переопределим метод `OnInspectorGUI()`.
```csharp
// Метод для отрисовки в окне инспектора
public override void OnInspectorGUI()
{
    serializedObject.Update();

    // Display default Inspector 
    DrawDefaultInspector();

    // Display ptoperties on Inspector
    DrawProperties();

    serializedObject.ApplyModifiedProperties();
}
```
В нём сначала используем метод отрисовки инспектора по-умолчанию, а затем отрисовываем свойства из списка `ObjectPropertyView` используя метод `DrawProperties()`.

В итоге наш класс `ExtendedEditor` должен выглядеть примерно [вот так](https://github.com/{{ site.github.owner_name }}/{{ page.repository }}/blob/main/Editor/ExtendedEditor.cs).

## Использование
Создадим для примера простой класс `Test`.
```csharp
using UnityEngine;
using UnityEngine.UI;

public class Test : MonoBehaviour
{
    [SerializeField, HideInInspector] private Text text;
    [SerializeField, HideInInspector] private int integer;
}
```
Атрибут `[HideInInspector]` нужен, чтобы необходимые поля с объектами нам скрывались в редакторе по умолчанию. 

Чтобы воспользоваться `ExtendedEditor`, нужно в папке *Editor* создать производный от него класс `TestEditor` с атрибутом `[CustomEditor(typeof(Test))]`
```csharp
using UnityExtended;
using UnityEditor;

[CanEditMultipleObjects]
[CustomEditor(typeof(Test))]
public class TestEditor : ExtendedEditor
{
    private void OnEnable()
    {
        AddPropertyView(serializedObject.FindProperty("text"));
        AddPropertyView(serializedObject.FindProperty("integer"));
    }
}
```
В методе `OnEnable()` находим необходимые нам `SerializedProperty` и добавляем их в список для отображения. 
Поле типа `Integer` добавлено для визуализации ошибки связаной с использованием типа не производного от `Object`.

## Result
Если поле пустое, под спойлером будет сообщение об ошибке.
![Сообщение об ошибке](/assets/images/posts/2023-02-11-unity3d-object-property-view/empty_field.jpg)

Если в поле добавить ссылку на объект, свойства этого объекта отобразятся в инспекторе.
![Показать свойства объекта](/assets/images/posts/2023-02-11-unity3d-object-property-view/display_object_properties.jpg)


Этот способ также применим для отображения сериализованых полей `ScriptableObject`.
![Показать свойства scriptableObject](/assets/images/posts/2023-02-11-unity3d-object-property-view/display_scriptableObject_Properties.jpg)

Спасибо за внимание :)
