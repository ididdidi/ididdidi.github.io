---
title:  "Unity: Displaying object properties"
repository: Unity3d-ExtendedEditor 
preview: /assets/images/posts/2023-02-11-unity3d-object-property-view/preview.png
date:   2023-02-11 10:00:00 +0300
categories: cases
tags: [c#, unity3d]
lang: En
layout: post
---

When working with fields for objects in **Unity3d**, at some point you realize that it would be nice to change the serialized properties of an object right in the inspector window of the object that encapsulates it. You can implement such a feature using a custom editor.

## Implementation
First, we need a class that will inherit from `Editor`.
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
I made it abstract so that I can later reuse it in derived classes. Also, I added a method to it to display an error message. In the future, the class may be extended with new methods.

>Attention! This class must be placed in the *Editor* folder to avoid errors when building the project.

In the class created above, I added an inner class that encapsulates the `SerializedProperty` and `Editor` of the object whose properties we want to display, and also provides a method for rendering these properties to the editor.
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
            // If field is not reference, show error message.            
            var style = new GUIStyle(EditorStyles.objectField);
            style.normal.textColor = Color.red;

            // Display label with error message
            EditorGUILayout.LabelField(label, new GUIContent($"{serializedProperty.propertyType} is not a reference type"), style);
            return;
        }
		
        // We draw a spoiler
        foldout = EditorGUILayout.Foldout(foldout, new GUIContent());

        EditorGUI.BeginChangeCheck();
        // Draw the field of the object
        EditorGUI.ObjectField(GUILayoutUtility.GetLastRect(), serializedProperty, label);

        // If there are changes
        if (EditorGUI.EndChangeCheck() || (serializedProperty.objectReferenceValue ^ editor))
        {
            // Update the object editor
            UpdateEditor();
        }

        if (foldout)
        {
            GUILayout.BeginVertical(EditorStyles.helpBox);
            // Display the properties of object or display an error if they are not there
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

    // Update the object editor
    private void UpdateEditor()
    {
        if (serializedProperty.propertyType == SerializedPropertyType.ObjectReference)
        {
            editor = Editor.CreateEditor(serializedProperty.objectReferenceValue);
        }
    }
}
```
In the constructor of this class, an editor is created that will draw the properties of the object in the `Draw()` method, immediately below the field with the given object. To hide properties - use `Foldout()`.
In case of any changes in the field where the link to the object is stored, the editor is recreated anew.

Next, in the `ExtendedEditor` class, we declare a list of `ObjectPropertyView` objects and methods for working with it.
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
The `AddPropertyView()` methods are required to add serialized object properties for later display.
The `DrawProperties()` method draws the properties of objects from the list with `ObjectPropertyView`.

Finally, let's override the `OnInspectorGUI()` method.
```csharp
// Method for drawing in the Inspector window
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
In it, we first use the default inspector draw method, and then draw the properties from the `ObjectPropertyView` list using the `DrawProperties()` method.

As a result, our `ExtendedEditor` class should look something like [like this](https://github.com/{{ site.github.owner_name }}/{{ page.repository }}/blob/main/Editor/ExtendedEditor.cs ).

## Usage
Let's create a simple Test class as an example.
```csharp
using UnityEngine;
using UnityEngine.UI;

public class Test : MonoBehaviour
{
    [SerializeField, HideInInspector] private Text text;
    [SerializeField, HideInInspector] private int integer;
}
```
The `[HideInInspector]` attribute is needed so that the necessary fields with objects are hidden in the editor by default.

To use `ExtendedEditor`, you need to create a class `TestEditor` derived from it in the *Editor* folder with the attribute `[CustomEditor(typeof(Test))]`
`[CustomEditor(typeof(Test))]`
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
In the `OnEnable()` method we find the `SerializedProperty` we need and add them to the list for display.
A field of type `Integer` was added to visualize the error associated with using a type not derived from `Object`.

## Result
If the field is empty, there will be an error message under the spoiler.
![Display error](/assets/images/posts/2023-02-11-unity3d-object-property-view/empty_field.jpg)

If you add a link to an object in the field, the properties of that object will be displayed in the inspector.
![Display object properties](/assets/images/posts/2023-02-11-unity3d-object-property-view/display_object_properties.jpg)

This method is also applicable for displaying serialized fields of `ScriptableObject`.
![Display scriptableObject properties](/assets/images/posts/2023-02-11-unity3d-object-property-view/display_scriptableObject_Properties.jpg)

You can use `ExtendedEditor` in your project by adding [this repository](https://github.com/{{ site.github.owner_name }}/{{ page.repository }}) as a [submodule](https://git-scm.com/book/en/v2/Git-Tools-Submodules):

    git submodule add https://github.com/{{ site.github.owner_name }}/{{ page.repository }}

Thank you for your attention :)
