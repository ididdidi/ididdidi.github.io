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

## 1. Настройка

В верхнем меню выберите: `Window` -> `Localization Storage`.

![Pic. 2. Кнопка настроек](/assets/images/projects/easy-assets-localize/settings-button.jpg?raw=true "Pic. 2. Кнопка настроек]")
<center>В открывшемся окне Хранилище локализаций нажмите на ⚙ рядом с полем поиска.</center>

![Pic. 3. Настройки](/assets/images/projects/easy-assets-localize/settings.jpg?raw=true "Pic. 3. Настройки")
<center>В открывшейся вкладке вы можете добавить языки и типы ресурсов, необходимые для локализации.</center>

![Pic. 4. Добавить язык](/assets/images/projects/easy-assets-localize/languages.jpg?raw=true "Pic. 4. Добавить язык")
<center>Чтобы добавить язык, нажмите <b>+</b> внизу списка и выберите его из списка основных языков системы Unity.</center>

![Pic. 5. Добавить тип ассета](/assets/images/projects/easy-assets-localize/types.jpg?raw=true "Pic. 5. AДобавить тип ассета")
<center>Чтобы добавить новый тип ассета для локализации, нажмите <b>+</b> внизу списка с типами.</center>

![Pic. 6. Настройки](/assets/images/projects/easy-assets-localize/settings-result.jpg?raw=true "Pic. 6. Настройки") 
<center>Перетащите образец актива по умолчанию в появившееся поле. Дождитесь завершения генерации кода.</center>

![Pic. 7. Новый тип ресурсов](/assets/images/projects/easy-assets-localize/new-resource-type.jpg?raw=true "Pic. 7. Новый тип ресурсов") 
<center>В списке локализаций появится новый пункт соответствующего типа, при клике по которому откроется список локализаций данного типа.</center>

![Pic. 8. Локализация по умолчанию](/assets/images/projects/easy-assets-localize/default-localization.jpg?raw=true "Pic. 8. Локализация по умолчанию") 
<center>В списке локализаций данного типа будет локализация по умолчанию. Вы можете редактировать локализацию по умолчанию, выбрав её и кликнув на иконку с инструментами в верхнем правом углу окна локализации.</center>

![Pic. 9. Редактирование локализации по умолчанию](/assets/images/projects/easy-assets-localize/edit-default-localization.jpg?raw=true "Pic. 9. Редактирование локализации по умолчанию")
<center>Это разблокирует ресурсы используемых языков для изменения.</center>

## 2. Применение локализации к объекту

Выберите объект, который собираетесь локализовать.
Добавьте к нему компонент локализации ресурса: `Add component` -> `Localize` -> `[Resource type]Localization`.
![Pic. 10. Add Localization Component](/assets/images/projects/easy-assets-localize/localization-component.png?raw=true "Pic. 10. Add Localization Component")

![Pic. 11. Localization Component](/assets/images/projects/easy-assets-localize/localization-component-view.jpg?raw=true "Pic. 11. Localization Component")
<center>Добавьте объект в список обработчиков и выберите свойство, соответствующее типу ресурса (обычно они находятся вверху выпадающего списка)</center>

![Pic. 12. Add handler](/assets/images/projects/easy-assets-localize/add-handler.jpg?raw=true "Pic. 12. Add handler")
<center>Компонент будет устанавливать локализацию по умолчанию.</center>

## 3. Изменение локализации

Вы можете изменить локализацию прямо на объекте, для этого нажмите кнопку `Change Localization` в окне инспектора компоненте локализации.

![Pic. 13. Изменение Локализации](/assets/images/projects/easy-assets-localize/change-localization.jpg?raw=true "Pic. 13. Изменение Локализации")
<center>В открывшемся списке вы можете выбрать уже существующую локализацию для данного типа или добавить новую.</center>

Новая локализация будет ссылаться на те же ресурсы что и локализация по умолчанию. Вы можете изменить их на другие ресурсы того же типа.

> Внимание! При редактировании локализации используемой несколькими объектами, изменения коснутся всех объектов, использующих экземпляр данной локализации.

## 4. Управление локализациями в Runtime

Для управления локализацией в Runtime необходимо добавить на сцену компонент `LocalizationController`.

![Pic. 14. LocalizationController](/assets/images/projects/easy-assets-localize/localization-controller.jpg?raw=true "Pic. 14. LocalizationController")

В поле `Localization Storage` должен быть добавлен **Scriptable Object**  типа `LocalizationStorage`.
В поле события `On Chage Language` вы можете добавить обработчик принимающий строку с названием языка.

Для взаимодействия с объектом типа `LocalizationController` в коде вы можете использовать следующие методы:

Метод           | Описание
----------------|---------
GetInstance     | Статический метод, который создает или возвращает готовый экземпляр со сцены. Принимает аргумент `dontDestroy` который позволяет сохранить экземпляр объекта при смене сцены. Возвращает экземпляр `LocalizationController`.
Subscribe       | Позволяет экземпляру типа производного от `LocalizationComponent`, переданому в качестве аргумента, подписаться на изменения локализации.
Unsubscribe     | Позволяет экземпляру типа производного от `LocalizationComponent`, переданому в качестве аргумента, подписаться на изменения локализации.
SetNextLanguage | Меняет язык на следующий в списке локализации.
SetPrevLanguage | Меняет язык на предыдущий в списке локализации.
SetLanguage     | Устанавливает текущий язык и загружает локализованные ресурсы. В качестве аргумента принимает объект типа Language

Если у вас есть вопросы или предложения, смело пишите их на электронную почту, указанную на странице [Контакты](/ru/contacts)!
