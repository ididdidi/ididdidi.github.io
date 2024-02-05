---
title:  "Unity3d: Хранение данных на устройстве"
gist: 2c9e1e20cb5d585ecaa8012fc6a0c78a
preview: /assets/images/posts/2021-04-03-unity3d-caching-resources/preview.jpg
date:   2021-04-03 10:00:00 +0300
categories: ru cases
tags: [c#, unity3d]
lang: Ru
layout: post
---

В этой статье речь идёт о сохранении данных в память устройства и организация доступа к ним. Она призвана подготовить Вас, дорогой читатель, к более сложной статье про загрузку ресурсов из сети. По сути это её вводная часть, которая была выделена в отдельный пост, потому что тема оказалась очень объёмной.

## Зачем хранить ресурсы в памяти устройства
Думаю многие разработчики прежде всего мобильных приложений сталкивались с тем, что ресурсы не помещаются в установочные файлы приложения. Особенно актуально это для тех кто использует фотореалистичный контент(панорамы 360). Небольшим подспорьем в этом выглядит использование файлов расширения, но они тоже ограничены в размере. Кроме того загрузка дополнительного контента требует обновления приложения через магазин.

Можно загружать из сети все необходимые ресурсы при каждом запуске, но несмотря на стремительное развитие интернета, это всё ещё довольно длительный, ресурсозатратны и ненадёжный процесс. Поэтому рационально будет ресурсы загружаемые из сети кэшировать в память устройства и при необходимости обращаться к ним.

## Настройка кэширования
Прежде чем что то сохранять надо настроить путь для хранения данных. Для этого я определил метод `ConfiguringCaching`:
```csharp
private static string cachingDirectory = "cache";

public static void ConfiguringCaching(string directoryName)
{
    cachingDirectory = directoryName;
    var path = Path.Combine(Application.persistentDataPath, cachingDirectory);
    if (!Directory.Exists(path))
    {
        Directory.CreateDirectory(path);
    }
    UnityEngine.Caching.currentCacheForWriting = UnityEngine.Caching.AddCache(path);
}
```
Метод на вход получает имя директории в которую мы будем сохранять загруженные данные. По умолчанию я назвал эту папку _data_. Полный путь к этой директории формируется с помощью [Application.persistentDataPath](https://docs.unity3d.com/ScriptReference/Application-persistentDataPath.html). Это значение представляет собой путь к каталогу, в котором можно сохранять данные, между запусками. При публикации на iOS и Android persistentDataPath указывает на общедоступный каталог на устройстве. Файлы в этом месте не удаляются обновлениями приложений. Файлы по-прежнему могут быть удалены непосредственно пользователями.

> Так же в этом методе путь к папке добавляется в `Caching.currentCacheForWriting`, для того чтобы настроить кэширование **AssetBundles**. **AssetBundles** обладает своими механизмами кэширования и нужно лишь правильно их использовать.

## Путь к директории с закэшированными ресурсами
Для того чтобы сохранить файл и потом к нему обращаться необходимо получить путь к нему в памяти устройства. Путь к директории с закэшированными ресурсами формируется из `Application.persistentDataPath`, имени папки для кэширования и специального хэша полученного из **url**.
```csharp
public static string GetCachedDirectory(this string url)
{
    if (!string.IsNullOrEmpty(url))
    {
        string[] path = {
            Application.persistentDataPath,
            cachingDirectory,
            Hash128.Compute(url).ToString()
        };
        return Path.Combine(path).Replace("\\", "/");
    }
    else
    {
        throw new Exception($"ConvertToCachedPath - Url address was entered incorrectly {url}");
    }
}
```

## Получение пути к сохранённому файлу
Для того чтобы сохранить файл и потом к нему обращаться необходимо получить путь к нему в памяти устройства. Для этого служит метод `GetCachedPath` В качестве аргументов он принимает строку с url-адресом файла и его версию виде `Hash128`.
```csharp
public static string GetCachedPath(this string url, Hash128 version)
{
    return Path.Combine(url.GetCachedDirectory(), version.ToString(), Path.GetFileName(url)).Replace("\\", "/");
}
```

## Версия закэшированного файла
Метод `GetCachedVersion` возвращает версию закэшированого файла, если таковой имеется или default `Hash128`.
```csharp
public static Hash128 GetCachedVersion(string url)
{
    Hash128 version = default;
    DirectoryInfo dir = new DirectoryInfo(url.GetCachedDirectory());
    if (dir.Exists)
    {
        System.DateTime lastWriteTime = default;
        var dirs = dir.GetDirectories();
        for (int i = 0; i < dirs.Length; i++)
        {
            if (lastWriteTime < dirs[i].LastWriteTime)
            {
                lastWriteTime = dirs[i].LastWriteTime;
                version = Hash128.Parse(dirs[i].Name);
            }
        }
    }
    return version;
}
```

## Есть ли файл в кэше?
Метод `IsCached` проверяет, находится ли файл заданной версии в кеше.
```csharp
public static bool IsCached(string url, Hash128 version) => new FileInfo(url.GetCachedPath(version)).Exists;
```

### Сохранение данных
Сохранение данных в постоянную память устройства реализует метод `SeveToCache`. На вход он принимает url-адрес файла в виде строки, актуальную версию файла в виде `Hash128`, массив типа `byte` с данными и флаг для удаления старых версий(по умолчанию  `true`).
```csharp
public static string SeveToCache(string url, Hash128 version, byte[] data, bool clearOldVersions = true)
{
    if (CheckFreeSpace(data.Length))
    {
        DirectoryInfo dirInfo = new DirectoryInfo(url.GetCachedDirectory());
        if (clearOldVersions && dirInfo.Exists) { dirInfo.Delete(true); }
        dirInfo.Create();

        string path = url.GetCachedPath(version);
        dirInfo.CreateSubdirectory(Directory.GetParent(path).FullName);
        File.WriteAllBytes(path, data);
        return path;
    }
    else { throw new Exception(string.Format("Caching - Not available space to download {0}Mb", data.Length / MIB)); }
}
```
Прежде всего метод проверяет, есть ли место для представленного массива с данными на устройстве. Если место есть, url-адрес конвертируется в путь файлу на устройстве с помощью метода `ConvertToCachedPath(string url)`, который мы рассмотрим далее. После чего создаются необходимые репозитории и файл сохраняется по предоставленному пути, с помощью стандартного метода `File.WriteAllBytes(string path, byte[] data)`. Если места нет - генерируется исключение.

## Наличие свободного места в памяти устройства
Прежде чем сохранять данные в памяти устройства, необходимо удостовериться в том, что данные на него поместятся. Для этого предварительно в проект был добавлен плагин **SimpleDiskUtils**. В нём есть библиотеки для разных ОС, с помощью которых можно выяснить сколько свободного места в памяти есть на устройстве. В следующем методе размер загружаемого файла сравнивается со свободным местом оставшимся на устройстве. В качестве аргумента метод принимает размер файла в байтах.
```csharp
private const float MIB = 1048576f;

public static bool CheckFreeSpace(float sizeInBytes)
        {
#if UNITY_EDITOR_WIN
            var logicalDrive = Path.GetPathRoot(Application.persistentDataPath);
            var availableSpace = SimpleDiskUtils.DiskUtils.CheckAvailableSpace(logicalDrive);
#elif UNITY_EDITOR_OSX
        var availableSpace = SimpleDiskUtils.DiskUtils.CheckAvailableSpace();
#elif UNITY_IOS
        var availableSpace = SimpleDiskUtils.DiskUtils.CheckAvailableSpace();
#elif UNITY_ANDROID
        var availableSpace = SimpleDiskUtils.DiskUtils.CheckAvailableSpace(true);
#endif
            return availableSpace > sizeInBytes / MIB;
        }
```
> При генерации **AssetBundle** могут возникнуть проблемы с обращением к разным методам несмотря на наличие директивы `#if-#else`, чтобы этого избежать лишние вызовы можно закомментировать или удалить.

## Вместо заключения
Как можно заметить предоставленный код не отличается какой либо сложностью и при этом выполняет важную функцию сохранения данных на устройстве.
В следующем посте мы воспользуемся им для кэширования ресурсов загружаемых из сети интернет.

Спасибо что дочитали до конца, надеюсь это Вам пригодится :)
