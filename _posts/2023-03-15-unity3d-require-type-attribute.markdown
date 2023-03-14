---
title:  "Unity3d: Attribute for displaying fields with interface"
repository: Unity3d-Attributes 
preview: /assets/images/posts/2023-03-15-unity3d-require-type-attribute/preview.jpg
date:   2023-03-15 10:00:00 +0300
categories: cases
tags: [c#, unity3d]
lang: En
layout: post
---

There are already quite a few solutions on the web for visualizing fields with an interface in the inspector. But those that I came across allowed adding either only a component to the field, or only an object with a suitable component, which I tried to fix by making this argument more universal.

## Implementation
Let's start with the attribute class. It looks about the same in all implementations.
```csharp
[System.AttributeUsage(System.AttributeTargets.Field)]
public class RequireTypeAttribute : PropertyAttribute
{
    // Interface type.
    public System.Type RequiredType { get; }

    public RequireTypeAttribute(System.Type type) => this.RequiredType = type;
}
```
It takes the type of the encapsulated object as an argument. Sometimes a flag is present to allow assignment of scene objects. We will have this feature by default.

The most interesting "magic" will happen in the driver class, specifically in the `OnGUI()` method.
```csharp
[CustomPropertyDrawer(typeof(RequireTypeAttribute))]
public class RequireTypeDrawer : PropertyDrawer
{
    public override void OnGUI(Rect position, SerializedProperty property, GUIContent label)
    {
        // Check if this is reference type property
        if (property.propertyType != SerializedPropertyType.ObjectReference)
        {
            // If field is not reference, show error message.            
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
In the `OnGUI()` method, we check whether the field is intended for a reference type or not. If not, we show an error and exit the method.

The next, in the same method, we call a method that checks the objects moved to the area intended for our field.
```csharp
private void ChecDragAndDrops(Rect position, System.Type requiredType)
{
    // If the cursor is in the area of the rendered field
    if (position.Contains(Event.current.mousePosition))
    {
        // Iterate over all draggable references
        foreach (var @object in DragAndDrop.objectReferences)
        {
            // If we do not find the required type
            if (!IsValidObject(@object, requiredType))
            {
                // Disable drag and drop
                DragAndDrop.visualMode = DragAndDropVisualMode.Rejected;
                break;
            }
        }
    }
}
```
The method first checks whether the cursor gets into the field area when moving objects, and then in a loop checks them for compliance with the specified interface. If at least one of them does not fit the specified interface, adding objects to the field is prohibited.

The type checking method first checks the object with the `GameObject`, if it is, then using the `GetComponent()` method, we try to find the object of the given interface and compare it with `null`.
```csharp
private bool IsValidObject(Object @object, System.Type requiredType)
{
    // If the object is a GameObject
    if (@object is GameObject go)
    {
        // Check if it has a component of the required type and return result
        return go.GetComponent(requiredType) != null;
    }

    // Check the reference itself for compliance with the required type
    return requiredType.IsAssignableFrom(@object.GetType());
}
```
If it wasn't a `GameObject`, then we just check to see if it's possible to cast it to the desired interface and return that as the result.

In the `DrawObjectField()` method, as you might guess, we draw a field for our object.
```csharp
private void DrawObjectField(Rect position, SerializedProperty property, System.Type requiredType, GUIContent label)
{
    // Start blocking change checks
    EditorGUI.BeginChangeCheck();
    // Display a ObjectField
    EditorGUI.ObjectField(position, property, label);
    // If changes were made to the contents of the field and a GameObject was added to the field
    if (EditorGUI.EndChangeCheck() && property.objectReferenceValue is GameObject @object)
    {
        // Get component of the required type on the object and save a reference to it in a property
        property.objectReferenceValue = @object.GetComponent(requiredType);
    }
}
```
If the moved object was a `GameObject`, use `GetComponent()` to find a component suitable for our interface.

The last method was added in case the objects were added via code but without a corresponding interface.
```csharp
private void ChecValues(SerializedProperty property, System.Type requiredType)
{
    // If the reference is not null and points to an object that does not match the type
    if (property.objectReferenceValue != null && !IsValidObject(property.objectReferenceValue, requiredType))
    {
        // Nullify the reference
        property.objectReferenceValue = null;
    }
}
```

## Usage
To use this attribute, you must declare the Object field, and pass the required interface type to the attribute
```csharp
[SerializeField, RequireType(typeof(IExampleInterface))] private Object exampleObject;
```

## Conclusion
Using this attribute, you can change the contents of the field by dragging both entire objects from the hierarchy window and individual components from the inspector, the field can also work with `ScriptableObject` and with an array of objects.

You can use this and other attributes in your project by adding [this repository](https://github.com/{{ site.github.owner_name }}/{{ page.repository }}) as a [submodule](https://git-scm.com/book/en/v2/Git-Tools-Submodules):

    git submodule add https://github.com/{{ site.github.owner_name }}/{{ page.repository }}

Thank you for your attention :)
