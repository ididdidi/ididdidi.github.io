---
title: "Unity3d: Easy Assets Localize"
repository: Unity3d-EasyAssetsLocalize
preview: ./assets/images/projects/easy-assets-localize/preview.jpg
excerpt: The package for easy localization of assets in Unity3d...
date: 10-09-2023
categories: projects
tags: [c#, unity3d, work]
layout: project
lang: En
---

![Pic. 1. View](/assets/images/projects/easy-assets-localize/view.jpg?raw=true "Pic. 1. View") 
The package for easy localization of assets in Unity3d.

## Installation
* Open the __Package Manager__: `Window` -> `Package Manager`.
* Click on the __`+`__ at the top.
* Select `Add package from git URL...` in the drop-down list.
* Paste the link to this repository into the field that appears and press `Enter`.
* Wait for the code generation to finish.

## Setup
Select from the top menu: `Window` -> `Localization Storage`.

![Pic. 2. Settings button](/assets/images/projects/easy-assets-localize/settings-button.jpg?raw=true "Pic. 2. Settings button")
<center>In the Localization Storage window, click on the ⚙ button near the search field.</center>

![Pic. 3. Settings](/assets/images/projects/easy-assets-localize/settings.jpg?raw=true "Pic. 3. Settings")
<center>In the tab, you can add the languages and resource types needed for localization.</center>

![Pic. 4. Add language](/assets/images/projects/easy-assets-localize/languages.jpg?raw=true "Pic. 4. Add language")
<center>Click on the __`+`__ at the bottom of the list and select it from the list of Unity's base system languages to add a language.</center>

![Pic. 5. Add resource type](/assets/images/projects/easy-assets-localize/types.jpg?raw=true "Pic. 5. Add resource type")
<center>Click the __`+`__ at the end of the list with types, to add a new type of assets for localization.</center>

![Pic. 6. Settings](/assets/images/projects/easy-assets-localize/settings-result.jpg?raw=true "Pic. 6. Settings")
<center>Drag the default asset sample into the field that appears. Wait for the code generation to finish.</center>

## Usage
Select the object that needs localization.
Add a resource localization component to it: `Add component` -> `Localize` -> `[Resource type]Localization`.
![Pic. 7. Add Localization Component](/assets/images/projects/easy-assets-localize/localization-component.png?raw=true "Pic. 7. Add Localization Component")

![Pic. 8. Localization Component](/assets/images/projects/easy-assets-localize/localization-component-view.jpg?raw=true "Pic. 8. Localization Component")
<center>For localization to change the resources of an object, add the object to the list of handlers and select the property corresponding to the resource type (usually they are at the top of the drop-down list)</center>

![Pic. 9. Add handler](/assets/images/projects/easy-assets-localize/add-handler.jpg?raw=true "Pic. 9. Add handler")

The component that appears will reference the default localization. To change localization, click the `Change Localization` button.

![Pic. 10. Change Localization](/assets/images/projects/easy-assets-localize/change-localization.jpg?raw=true "Pic. 10. Change Localization")
<center>In the list that appears, you can select an existing localization or add a new one.<center>

The new localization will link to the same resources as the standard one. They can be changed directly on the component.

> Attention! The changes will affect all objects using an instance of this localization.

_Thank you for your interest! :)_
