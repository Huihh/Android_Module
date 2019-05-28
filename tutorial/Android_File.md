# 1. 文件保存在设备存储器（Save files on device storage）

安卓使用的文件系统类似于其它平台上基于次磁盘的文件系统。该章节将描述如何使用文件系统的 [File](https://developer.android.com/reference/java/io/File.html) APIs 读写文件。



一个 [File](https://developer.android.com/reference/java/io/File.html) 对象适用于从开始到结束顺序的读写大量数据，而不能跳跃读写。举个例子，它适用于通过网络交换的图像文件或任何东西。



文件保存的实际位置可能因设备不同而不同，你可以使用该章节描述的方法访问 **内/外存储** 路径，而不是使用绝对路径。



查看设备上的文件，可以使用方法 [File.getAbsolutePath( )](https://developer.android.com/reference/java/io/File.html#getAbsolutePath())  记录文件的位置。然后通过 [Android Studio's Device File Explorer ](https://developer.android.com/studio/debug/device-file-explorer.html) 浏览设备文件。



 ## 1.1 选择内部或外部存储（Choose internal or external storage）

所有的安卓设备都有连个存储区域：“内部” 和 “外部”存储。名称来自安卓的早期，当时大多数设备都提供内置非易失性内存（内部存储），以及一个可移动存储介质，如 micro SD 卡（外部存储）。



这些名字来自Android的早期，当时大多数设备都提供内置非易失性内存(内部存储)，以及可移动存储介质，如micro SD卡(外部存储)。许多设备现在是将永久存储空间划分为单独的 “内部” 和 “外部” 区域。所以，即使没有可移动存储介质，这两个区域一直存在。无论外部存储是否可移动，**API** 行为都是相同的。



因为外部存储可能是可移动的，以下是这两个选项之间的一些区别。

**内部存储：**

- 总是可用的。
- 保存在这里的文件只能通过你的应用程序访问。
- 当用户卸载应用程序时，系统将从内部存储中删除应用程序的所有文件。

当你想确保用户或其他应用程序都无法访问你的文件时，存放在内部存储。

**外部存储：**

- 它并不总是可用的，因为用户可以将外部存储挂载为 USB 存储，并在某些情况下从设备中删除它。
- 它是所有人都可读的，所以保存在这里的文件可以在你的控制之外读取。
- 当用户卸载应用程序时，只有将应用程序的文件保存在 [getExternalFilesDir( )](https://developer.android.com/reference/android/content/Context.html#getExternalFilesDir(java.lang.String)) 的目录中，系统才会从这里删除这些文件。

对于不需要访问限制的文件，以及您希望与其他应用程序共享或允许用户使用计算机访问的文件，存放在外部存储。



**Tip：** 尽管应用程序默认是安装安装在 **内部存储** 中，你也可以在你的配置文件中通过指定 [android:installLocation](https://developer.android.com/guide/topics/manifest/manifest-element.html#install) 属性安装在 **外部存储** 中。当 **APK** 非常大，并且 **外部存储** 空间大于 **内部存储** 空间时，设置该属性安装在 **外部存储** 中。更多详细内容，查看 [App Install Location](https://developer.android.com/guide/topics/data/install-location.html)。



## 1.2 保存文件在内部存储空间（Save a file on internal storage）

应用程序的 **内部存储目录** 由应用程序的包名在安卓文件系统的一个特殊位置指定，可以使用以下 **APIs** 访问该位置。

**Tip：** 不同于 **外部存储目录**， 你的应用程序不需要任何系统权限来读写这些方法返回的内部目录。



### 1.2.1 写文件（Write a file）

当将文件保存到内部存储器时，可以通过调用以下两种方法之一来获取适当的目录作为 [File](https://developer.android.com/reference/java/io/File.html):

[getFilesDir( )](https://developer.android.com/reference/android/content/Context.html#getFilesDir())

​	返回应用程序在内部存储的 [File](https://developer.android.com/reference/java/io/File.html)。



[getCacheDir( )](https://developer.android.com/reference/android/content/Context.html#getCacheDir())

​	返回应用程序的临时缓存文件在内部存储的 [File](https://developer.android.com/reference/java/io/File.html)。确保在不再需要每个文件时删除它，并在任何给定时间（比如1MB）为所使用的内存量实现合理的大小限制。

**警告：**当系统内存不足时，它可能会在没有警告的情况下删除你的缓存文件。



在这些目录下创建一个新文件，你可以使用 [File( )](https://developer.android.com/reference/java/io/File.html#File(java.io.File,%20java.lang.String)) 构造器，传递上述方法之一返回的内部存储的 [File](https://developer.android.com/reference/java/io/File.html)。例如:

```java
File file = new File(context.getFilesDir(), filename);
```



或者，你可以调用 [openFileOutput( )](https://developer.android.com/reference/android/content/Context.html#openFileOutput(java.lang.String,%20int)) 来获得一个 [FileOutputStream](https://developer.android.com/reference/java/io/FileOutputStream.html) ，它将写入你内部目录中的一个文件。例如，下面是如何将一些内容写入文件中：

```java
String filename = "myfile";
String fileContents = "Hello world!";
FileOutputStream outputStream;

try {
    outputStream = openFileOutput(filename, Context.MODE_PROVATE);
    outputStream.write(fileContents.getBytes());
    outputStream.close();
} catch (Exception e) {
    e.printStackTrace();
}
```



注意，[openFileOutput( )](https://developer.android.com/reference/android/content/Context.html#openFileOutput(java.lang.String,%20int)) 方法要求传一个文件类型参数。传递 [MODE_PRIVATE](https://developer.android.com/reference/android/content/Context.html#MODE_PRIVATE) 使其成为你的应用程序私有的。其它的类型，[MODE_WORLD_READABLE](https://developer.android.com/reference/android/content/Context.html#MODE_WORLD_READABLE) 和 [MODE_WORLD_WRITEABLE](https://developer.android.com/reference/android/content/Context.html#MODE_WORLD_WRITEABLE) ，已经在 API level 17 之前弃用了。从安卓 7.0 （API level 24）开始，如果你使用它们，系统将抛出一个 [SecurityException](https://developer.android.com/reference/java/lang/SecurityException.html) 异常。如果你的应用程序需要共享私有文件给其它应用程序，你应该使用带有 [FLAG_GRANT_READ_URI_PERMISSION](https://developer.android.com/reference/android/content/Intent.html#FLAG_GRANT_READ_URI_PERMISSION) 的 [FileProvider](https://developer.android.com/reference/androidx/core/content/FileProvider.html]) 。更多详细内容，查看 [Sharing Files](https://developer.android.com/training/secure-file-sharing/index.html) 。



在安卓 6.0（API level 23）及以下版本，如果你将文件模式设置为世界可读，其他应用程序可以读取你的内部文件。可是，其它应用程序必须要支持你的包名和文件名。其他应用程序无法浏览你的内部目录，并且没有读或写访问权限，除非您显式地将文件设置为可读或可写。因此，只要你对内部存储中的文件使用 [MODE_PRIVATE](https://developer.android.com/reference/android/content/Context.html#MODE_PRIVATE) ，其他应用程序就永远无法访问它们。



### 1.2.2 写缓存文件（Write a cache file）

如果需要缓存一些文件，你应该使用 [createTempFile](https://developer.android.com/reference/java/io/File.html#createTempFile(java.lang.String,%20java.lang.String)) 。例如，下面的方法从 [URL](https://developer.android.com/reference/java/net/URL.html) 中提取文件名，并在应用程序的内部缓存目录中创建一个文件。

```java
private File getTempFile(Context context, String url) {
    File file;
    try {
        String fileName = Uri.parse(url).getLastPathSegment();
        file = File.createTempFile(fileName, null, context.getCacheDir());
    } catch (IOException e) {
        // Error while creating file
    }
    
    return file;
}
```



使用  [createTempFile( )](https://developer.android.com/reference/java/io/File.html#createTempFile(java.lang.String,%20java.lang.String)) 创建的文件位于应用程序私有目录的缓冲目录下。 你应该定期的 [delete the files](https://developer.android.com/training/data-storage/files#DeleteFile) 不再需要的目录。

**警告：** 当系统内存不足时，它可能会在没有警告的情况下删除你的缓存文件。因此，在你读之前应先检查它是否存在。



### 1.2.3 打开已存在文件（Open an existing file）











