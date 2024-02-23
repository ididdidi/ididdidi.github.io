---
title:  "Unity: Saving data on the device"
gist: 2c9e1e20cb5d585ecaa8012fc6a0c78a
preview: /assets/images/posts/2021-04-03-unity3d-caching-resources/preview.jpg
date:   2021-04-03 10:00:00 +0300
categories: cases
tags: [c#, unity3d]
lang: En
layout: post
---

In this article, we are talking about saving data to the device's memory and organizing access to it. It is intended to prepare you, dear reader, for a more complex article about downloading resources from the network. In fact, this is its introductory part, which was allocated in a separate post, because the topic turned out to be very voluminous.

## Why store resources in the device memory
I think many developers primarily of mobile applications have encountered the fact that resources do not fit in the application installation files. This is especially true for those who use photorealistic content(360 panoramas). A small help in this is the use of extension files, but they are also limited in size. In addition, downloading additional content requires updating the app through the store.

You can download all the necessary resources from the network at each launch, but despite the rapid development of the Internet, this is still quite a long, resource-consuming and unreliable process. Therefore, it is rational to cache the resources downloaded from the network to the device's memory and, if necessary, access them.

## Configuring caching
In order to save a file and access it later, you need to get the path to it in the device memory. The path to the directory with cached resources is formed from `Application.persistentDataPath`, the name of the folder for caching and a special hash obtained from **url**.
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
The input method gets the name of the directory where we will save the uploaded data. By default, I named this folder _data_. The full path to this directory is formed using [Application.persistentDataPath](https://docs.unity3d.com/ScriptReference/Application-persistentDataPath.html). This value is the path to the directory where you can save data between runs. When publishing on iOS and Android, persistentDataPath points to the public directory on the device. Files in this location are not deleted by app updates. Files can still be deleted directly by users.

> Also in this method, the path to the folder is added to `Caching.currentCacheForWriting`, to configure caching **AssetBundles**. **AssetBundles** it has its own caching mechanisms and you only need to use them correctly.

## Path to the directory with cached resources
The path to the directory with cached resources is formed from `Application.persistentDataPath`, `cachingDirectory` and a special hash obtained from **url**.
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

## Getting the path to a saved file
In order to save a file and access it later, you need to get the path to it in the device memory. To do this, use the `GetCachedPath` method. As arguments, it takes a string with the url address of the file and its version in the form `Hash128`.
```csharp
public static string GetCachedPath(this string url, Hash128 version)
{
    return Path.Combine(url.GetCachedDirectory(), version.ToString(), Path.GetFileName(url)).Replace("\\", "/");
}
```

## Cached file version
`GetCachedVersion` method returns the version of the cached file, if any, or default Hash128.
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

## Is the file in the cache?
The `IsCached` method checks whether a file of a given version is in the cache.
```csharp
public static bool IsCached(string url, Hash128 version) => new FileInfo(url.GetCachedPath(version)).Exists;
```

## Saving data
Saving data to the device's permanent memory is implemented by the `SeveToCache` method. As input, it takes the url of the file as a string, the current version of the file as `Hash128`, an array of `byte` type with data and a flag for deleting old versions (default `true`).
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
First of all, the method checks whether there is a place for the presented array with data on the device. If there is space, the url is converted to the path of the file on the device using the `GetCachedDirectory()` method, which we will discuss later. After that, the necessary repositories are created and the file is saved to the provided path using the standard `File.WriteAllBytes(string path, byte[] data)` method. If there is no space, an exception is thrown.

## Availability of free space in the device memory
Before storing data in the device's memory, you need to make sure that the data will fit on it. To do this, the **SimpleDiskUtils** plugin was previously added to the project. It has libraries for different operating systems, with which you can find out how much free memory space there is on the device. In the following method, the size of the downloaded file is compared with the free space remaining on the device. The method takes the file size in bytes as an argument.
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
> When generating **AssetBundle**, there may be problems with accessing different methods, despite the presence of the `#if-#else` directive, to avoid this, you can comment out or delete unnecessary calls.

### Getting the path to a saved file
In order to save a file and then access it, you need to get the path to it in the device's memory. To do this, use the `ConvertToCachedPath` method Accepts a string with the file url as the only argument.
```csharp
public static string ConvertToCachedPath(this string url)
{
    try
    {
        if (!string.IsNullOrEmpty(url))
        {
            var path = Path.Combine(Application.persistentDataPath, cachingDirectory + new System.Uri(url).LocalPath);
            return path.Replace("\\", "/");
        }
        else
        {
            throw new Exception("[Caching] error: Url address was entered incorrectly " + url); ;
        }
    }
    catch (System.UriFormatException e)
    {
        throw new Exception("[Caching] error: " + url + " " + e.Message);
    }
}
```
The file path is generated from `Application.persistentDataPath`,  the name of the folder to cache, and the local path obtained from the url.
It is important to note that the method does not guarantee that the file will be found at the specified path. To check this, use: `File.Exists(path)`.

## Instead of a conclusion
As you can see, the provided code does not differ in any complexity and at the same time performs an important function of saving data on the device.
In the next post, we will use it to cache resources downloaded from the Internet.

Thank you for reading to the end, I hope this will be useful to you :)
