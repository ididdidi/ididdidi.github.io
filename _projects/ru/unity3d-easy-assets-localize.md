---
title: "Unity3d: Простая локализация ассетов"
repository: Unity3d-EasyAssetsLocalize
preview: ./assets/images/projects/easy-assets-localize/preview.jpg
excerpt: Пакет для легкой локализации ассетов в Unity3d...
date: 10-09-2023
categories: ru projects
tags: [c#, unity3d, work]
layout: project
lang: Ru
---

![Pic. 1. View](/assets/images/projects/easy-assets-localize/view.jpg?raw=true "Pic. 1. View") 
Пакет для легкой локализации ассетов в Unity3d.

## 1. Установка

*   Откройте диспетчер пакетов: `Window` -> `Package Manager`.
*   Нажмите на __`+`__ вверху.
*   В раскрывающемся списке выберите `Add package from git URL...`
*   Вставьте ссылку на этот репозиторий в появившееся поле и нажмите `Enter`.
*   Дождитесь завершения генерации кода.

## 2. Настройка

В верхнем меню выберите: `Window` -> `Localization Storage`.
В открывшемся окне Хранилище локализаций нажмите на __`⚙`__ рядом с полем поиска.
![Pic. 2. Кнопка настроек](/assets/images/projects/easy-assets-localize/settings-button.jpg?raw=true "Pic. 2. Кнопка настроек]") 

В открывшейся вкладке вы можете добавить языки и типы ресурсов, необходимые для локализации.
![Pic. 3. Настройки](/assets/images/projects/easy-assets-localize/settings.jpg?raw=true "Pic. 3. Настройки") 

Чтобы добавить язык, нажмите __`+`__ внизу списка и выберите его из списка основных языков системы Unity.
![Pic. 4. Добавить язык](/assets/images/projects/easy-assets-localize/languages.jpg?raw=true "Pic. 4. Добавить язык")

Чтобы добавить новый тип ассета для локализации, нажмите __`+`__ внизу списка с типами.
![Pic. 5. Добавить тип ассета](/assets/images/projects/easy-assets-localize/types.jpg?raw=true "Pic. 5. AДобавить тип ассета") 

Перетащите образец актива по умолчанию в появившееся поле.
![Pic. 6. Настройки](/assets/images/projects/easy-assets-localize/settings-result.jpg?raw=true "Pic. 6. Настройки") 
Дождитесь завершения генерации кода.

## 3. Использование

* Выберите объект, который собираетесь локализовать.
* Добавьте к нему компонент локализации ресурса: `Add component` -> `Localize` -> `[Resource type]Localization`.
* Появившийся компонент будет ссылаться на локализацию по умолчанию. Чтобы изменить локализацию, нажмите кнопку `Change Localization`.
* Для локализации для изменения ресурсов объекта добавьте объект в список обработчиков и выберите свойство, соответствующее типу ресурса (обычно они находятся вверху выпадающего списка)

_Спасибо за внимание!_
