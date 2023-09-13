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

The package for easy localization of assets in Unity3d.

## Installation
* Open the __Package Manager__: `Window -> Package Manager`.
* Click on the __`+`__ at the top.
* Select `Add package from git URL...` in the drop-down list.
* Paste the link to this repository into the field that appears and press `Enter`.
* Wait for the code generation to finish.

## Setup
* Select from the top menu: `Window -> Localization Storage`.
* In the Localization Storage window that opens, click on the __`⚙`__ next to the search field.
* In the tab that opens, you can add the languages and resource types needed for localization.
* To add a language, click on the __`+`__ at the bottom of the list and select it from the list of Unity's base system languages.
* To add a new type of assets for localization, click the __`+`__ at the end of the list with types and drag the default asset sample into the field that appears. Wait for the code generation to finish.

## Usage
* Select the object you are going to localize.
* Add a resource localization component to it: `Add component -> Localize -> [Resource type]Localization`.
* The component that appears will reference the default localization. To change localization, click the `Change Localization` button.
* For localization to change the resources of an object, add the object to the list of handlers and select the property corresponding to the resource type (usually they are at the top of the drop-down list)


_Thank you for your interest! :)_
