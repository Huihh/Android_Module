# 1. 文件保存在设备存储器（Save files on device storage）

安卓使用的文件系统类似于其它平台上基于磁盘的文件系统。该章节将描述如何使用文件系统的 [File](https://developer.android.com/reference/java/io/File.html) APIs 读写文件。



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

读一个已存在的文件，通过调用 [openFileInput(name)](https://developer.android.com/reference/android/content/Context.html#openFileInput(java.lang.String)) ，传入参数为文件名称。



你可以通过调用 [fileList( )](https://developer.android.com/reference/android/content/Context.html#fileList()) 获取应用程序所有文件名的数组。



**Tip：** 如果你需要在应用程序中打包一个在安装时可以访问的文件，将文件保存在工程 **res/raw/** 目录下。你可调用 [openRawResource( )](https://developer.android.com/reference/android/content/res/Resources.html#openRawResource(int)) ，传入参数为 **R.raw.ID**（文件的 id）。该方法返回一个 [InputStream](https://developer.android.com/reference/java/io/InputStream.html) ，你可以通过它读取文件。但是你不能对这些文件进行写入。



### 1.2.4 打开目录（Open a directory）

你可以使用以下方法打开内部文件系统上的目录。

[getFilesDir( )](https://developer.android.com/reference/android/content/Context.html#getFilesDir())

​	返回应用程序在内部存储的 [File](https://developer.android.com/reference/java/io/File.html)。



[getDir(name, mode)](https://developer.android.com/reference/android/content/Context.html#getDir(java.lang.String,%20int)) 

​	在你应用程序的唯一文件系统目录下创建（或打开已存在）一个目录。这个新目录可以通过 [getFilesDir( )](https://developer.android.com/reference/android/content/Context.html#getFilesDir()) 获取 。

[getCacheDir( )](https://developer.android.com/reference/android/content/Context.html#getCacheDir())

​	返回你的应用程序的唯一文件系统上的缓存目录关联的 [File](https://developer.android.com/reference/java/io/File.html) 。这个目录用于存在临时文件，应该定清除它。如果系统内存不足时，它会被清除，因此需要在读它之前先检查其是否存在。



在这些目录中创建一个新文件，你可以使用 [file( )](https://developer.android.com/reference/java/io/File.html#File(java.io.File,%20java.lang.String)) 构造器，传入参数为通过上述方法获取的 [File](https://developer.android.com/reference/java/io/File.html) 对象。例如：

```java
File directory = context.getFilesDir();
File file = new File(directory, filename);
```



## 1.3 保存文件在外部存储空间（Save a file on external storage）

如果你的文件需要被其它应用程序访问或者允许用户通过电脑访问，将文件保存在外部存储空间再好不过了。



在你请求了权限 [request storage permissions](https://developer.android.com/training/data-storage/files#ExternalStoragePermissions) 和验证了存储可用 [verify that storage is available](https://developer.android.com/training/data-storage/files#CheckExternalAvail) 之后，你可以保存两种不同类型的文件：

- [Public files](https://developer.android.com/training/data-storage/files#PublicFiles) :  用户和其它应用程序可以自由访问的文件。当你的应用程序被用户卸载时，这些文件将依然对用户有效。例如，你的应用程序获取的图片会其它下载的文件应该被保存为该类型。
- [Private files](https://developer.android.com/training/data-storage/files#PrivateFiles) : 完全属于你的应用程序的文件，当用户卸载你的应用程序时这些文件将被删除。虽然这些文件在技术上是可以被用户和其它应用程序访问的，因为它们位于外部存储中，但是它们不向应用程序外部的用户提供价值。



**警告：** 如果用户移除 SD 卡或将设备连接到计算机，外部存储可能不可用。这些文件 对于用户和具有 [READ_EXTERNAL_STORAGE](https://developer.android.com/reference/android/Manifest.permission.html#READ_EXTERNAL_STORAGE) 权限的其它应用程序仍然可见。因此，如果你的应用程序的功能依赖于这些文件，或者你需要完全限制访问，你应该把你的文件写入内部存储。



### 1.3.1 请求外部存储权限（Request external storage permissions）

要写外部存储区域，必须要在你的配置文件 [manifest file](https://developer.android.com/guide/topics/manifest/manifest-intro.html) 中请求 [WRITE_EXTERNAL_STORAGE](https://developer.android.com/reference/android/Manifest.permission.html#WRITE_EXTERNAL_STORAGE) 权限:

```xml
<manifest ...>
   <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
    ...
</manifest>   
```



**注意：** 如果你使用 [WRITE_EXTERNAL_STORAGE](https://developer.android.com/reference/android/Manifest.permission.html#WRITE_EXTERNAL_STORAGE) 权限，则它还隐式地拥有读取外部存储的权限。



如果你的应用程序只需要读外部存储（不需要写），则你只需请求 [READ_EXTERNAL_STORAGE](https://developer.android.com/reference/android/Manifest.permission.html#READ_EXTERNAL_STORAGE) 权限：

```xml
<manifest ...>
	<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
    ...
</manifest>
```



从安卓 4.4（API level 19）开始，读取或写入应用程序的外部存储目录中的私有文件，需要调用 [getExternalFilesDir( )](https://developer.android.com/reference/android/content/Context.html#getExternalFilesDir(java.lang.String)) 方法，不再需要  [WRITE_EXTERNAL_STORAGE](https://developer.android.com/reference/android/Manifest.permission.html#WRITE_EXTERNAL_STORAGE) 和 [READ_EXTERNAL_STORAGE](https://developer.android.com/reference/android/Manifest.permission.html#READ_EXTERNAL_STORAGE) 权限。如果你的应用程序需要支持安卓 4.3（API level 18）及以下， 如果您只想访问私有的外部存储目录，那么您应该通过添加 [maxSdkVersion](https://developer.android.com/guide/topics/manifest/uses-permission-element.html#maxSdk) 属性来声明只在较低版本的安卓上请求权限：

```xml
<manifest ...>
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"
                     android:maxSdkVersion="18" />
</manifest>
```



### 1.3.2 验证外部存储是否有效（Verify that external storage is available）

当用户已将存储安装到PC或已移除提供外部存储的SD卡时，外部存储可能无效。在访问它之前，应该始终验证它是否可用。你可以调用 [getExternalStorageState( )](https://developer.android.com/reference/android/os/Environment.html#getExternalStorageState()) 查看外部存储的状态。如果返回的状态是 [MEDIA_MOUNTED](https://developer.android.com/reference/android/os/Environment.html#MEDIA_MOUNTED) ，则可以读写你的文件。如果返回的状态是 [MEDIA_MOUNTED_READ_ONLY](https://developer.android.com/reference/android/os/Environment.html#MEDIA_MOUNTED_READ_ONLY) ，则你只能读你的文件。

 

例如，以下的方法可以判断外部存储是否有效：

```java
/* 检查外部存储是否可以读写 */
public boolean isExternalStorageWriteable() {
    String state = Enviroment.getExternalStorageState();
    if (Enviroment.MEDIA_MOUNTED.equals(state)) {
        return true;
    }
    return false;
}

/* 检查外部存储是否至少可读 */
public boolean isExternalStorageReadable() {
    String state = Enviroment.getExternalStorageState();
    if (Enviroment.MEDIA_MOUNTED.equals(state) || 
        Enviroment.MEDIA_MOUNTED_READ_ONLY.equals(state)) {
        return true;
    }
    return false;
}
```



### 1.3.3 保存到公共目录（Save to a public directory）

如果你需要保存公共文件到外部存储上，通过调用 [getExternalStoragePublicDirectory( )](https://developer.android.com/reference/android/os/Environment.html#getExternalStoragePublicDirectory(java.lang.String)) 获取外部存储上的 [File](https://developer.android.com/reference/java/io/File.html) 代表的目录。该方法接受一个参数，该参数指定要保存的文件类型，以便它们可以与其它公共文件逻辑地组织在一起，如 [DIRECTORY_MUSIC](https://developer.android.com/reference/android/os/Environment.html#DIRECTORY_MUSIC) 或 [DIRECTORY_PICTURES](https://developer.android.com/reference/android/os/Environment.html#DIRECTORY_PICTURES) 。例如：

```java
public File getPublicAlbumStorageDir(String albumName) {
    // 获取用户公共图片目录
    File file = new File(Enviroment.getExternalStoragePublicDirectory(Enviroment.DIRECTORY_PICTURES), albumName);
    if (!file.mkdirs()) {
        Log.e(LOG_TAG, "Directory not created");
    }
    
    return file;
}
```



如果你想对媒体扫描器隐藏你的文件，请在上层文件目录中包含一个 **.nomedia** 的空文件（注意有点前缀）。这样可以防止媒体扫描器读取你的媒体文件，并通过 [MediaStore](https://developer.android.com/reference/android/provider/MediaStore.html) 内容提供者将其提供给其它应用程序。



### 1.3.4 保存到私有目录（Save to a private directory）

如果你想将文件保存在外部存储中，让这些文件对你的应用程序是私有的，并且不能被 [MediaStore](https://developer.android.com/reference/android/provider/MediaStore.html) 内容提供者访问，您可以通过调用 [getExternalFilesDir( )](https://developer.android.com/reference/android/content/Context.html#getExternalFilesDir(java.lang.String)) 并向它传递一个名称，指定您想要的目录类型，从而获得一个仅供你的应用程序使用的目录。以这种方式创建的每个目录都被添加到一个父目录中，该目录包含了应用程序的所有外部存储文件，当用户卸载应用程序时，系统将删除这些文件。



**警告：** 外部存储区域上的文件并不总是可访问的，因为用户可以将外部存储挂载到计算机上作为存储设备使用。因此，如果需要存储对应用程序功能至关重要的文件，应该将它们存储在 [内部存储](https://developer.android.com/training/data-storage/files#WriteInternalStorage) 中。



例如，使用下面方法，你可以为单个相册创建目录：

```java
public File getPrivateAlbumStorageDir(Context context, String albumName) {
    //获取应用程序私有相册目录
    File file = new File(context.getExternalFilesDir(Enviroment.DIRECTORY_PICTURES), albumName);
    if (!file.mkdirs()) {
        Log.e(LOG_TAG, "Directory not created");
    }
    return file;
}
```



如果预定义的子目录名不适合你的文件，你在调用 [getExternalFilesDir( )](https://developer.android.com/reference/android/content/Context.html#getExternalFilesDir(java.lang.String)) 方法时传入 **null** 。这将返回应用程序在外部存储上的私有目录的根目录。



记住，[getExternalFilesDir( )](https://developer.android.com/reference/android/content/Context.html#getExternalFilesDir(java.lang.String)) 方法创建的目录，当用户卸载应用程序时，该目录将被删除。如果你需要在用户卸载应用程序后，保存的文件仍然可用。如：用户想要保存你的应用程序获取的照片，因此，你应该将文件保存到公共目录中。



不管你使用 [getExternalStoragePublicDirectory( )](https://developer.android.com/reference/android/os/Environment.html#getExternalStoragePublicDirectory(java.lang.String)) 方法保存共享文件或使用 [getExternalFilesDir( )](https://developer.android.com/reference/android/content/Context.html#getExternalFilesDir(java.lang.String)) 方法保存应用程序私有文件，使用 API 提供的目录名是很重要的，如 [DIRECTORY_PICTURES](https://developer.android.com/reference/android/os/Environment.html#DIRECTORY_PICTURES) 。这些目录名确保系统正确地处理这些文件。举个例子，保存在 [DIRECTORY_RINGTONES](https://developer.android.com/reference/android/os/Environment.html#DIRECTORY_RINGTONES) 中的文件被系统媒体扫描器分类为铃声，而非音乐。



### 1.3.5 在多个存储区域间选择（Selecct between multiple storage locations）

有时，分配内部内存分区作为外部存储的设备也提供SD卡插槽。这意味着设备有两个不同的外部存储目录，因此，在将 “私有” 文件写入外部存储时，需要选择使用哪一个。



从安卓 4.4（API level 19）开始，您可以通过调用 [getExternalFilesDirs( )](https://developer.android.com/reference/android/content/Context.html#getExternalFilesDirs(java.lang.String)) 来访问这两个位置，将返回一个表示每个存储位置的 [File](https://developer.android.com/reference/java/io/File.html) 数组。数组中的第一个条目被认为是主要的外部存储，您应该使用该位置，除非它已满或不可用。



如果你的应用程序支持安卓 4.3（API level 18）及以下，你需要使用支持库的静态方法，[ContextCompat.getExternalFilesDirs( )](https://developer.android.com/reference/androidx/core/content/ContextCompat.html#getExternalFilesDirs(android.content.Context,%20java.lang.String)) 。它也返回一个  [File](https://developer.android.com/reference/java/io/File.html) 数组。如果设备运行在安卓 4.3（API level 18）及以下，它只包含主外部存储这一个条目（如果存在第二块存储，你不能在安卓 4.3（API level 18）及以下访问它）。



## 1.4 查看可用空间（Query free space）

如果你提前知道你要保存多少数据，通过调用 [getFreeSpace( )](https://developer.android.com/reference/java/io/File.html#getFreeSpace()) 或 [getTotalSpace( )](https://developer.android.com/reference/java/io/File.html#getTotalSpace()) 方法，可以在不引发 [IOException](https://developer.android.com/reference/java/io/IOException.html) 的情况下确定是否有足够的空间。 这些方法分别提供存储区域的当前剩余空间和总空间。这些信息对于避免将存储容量填满到某个阈值之上也很有用。



但是，系统并不保证您可以写入 [getFreeSpace( )](https://developer.android.com/reference/java/io/File.html#getFreeSpace()) 所指示的字节数。如果返回的数字比要保存的数据大几 MB，或者如果文件系统的容量不足 90%，那就可以继续。否则，你应该不能写入存储区域。



**注意：** 在保存文件之前，不需要检查可用空间的大小。你可以直接写文件，捕获 [IOException](https://developer.android.com/reference/java/io/IOException.html) 异常。如果不知道需要多少空间，可能需要这样做。例如，如果在将 PNG 图像转换为 JPEG 保存之前更改了文件的编码，你不会预先知道文件的大小。



## 1.5 删除文件（Delete a file）

你应该总是删除你的应用程序不再需要的文件。最简单的方式是 [File](https://developer.android.com/reference/java/io/File.html) 对象调用 [delete( )](https://developer.android.com/reference/java/io/File.html#delete()) 方法删除文件 。

```java
myFile.delete();
```



如果文件保存在内部存储区域，通过 [Context](https://developer.android.com/reference/android/content/Context.html) 调用 [deleteFile( )](https://developer.android.com/reference/android/content/Context.html#deleteFile(java.lang.String)) 定位和删除一个文件。

```java
myContext.deleteFile(fileName);
```



**注意：** 当用户卸载应用程序时，Android系统会删除以下内容：

- 应用程序保存在内部存储区域的所有文件。
- 应用程序调用 [getExternalFilesDir( )](https://developer.android.com/reference/android/content/Context.html#getExternalFilesDir(java.lang.String)) 保存在外部存储区域的所有文件。

但是，你应该定期手动删除使用 [getCacheDir( )](https://developer.android.com/reference/android/content/Context.html#getCacheDir()) 创建的所有缓存文件，并定期删除不再需要的其他文件



## 1.6 附加资源（Additional resources）

有关将文件保存到设备存储区域的更多信息，请参阅以下参考资料。



实验示例代码：

[keep Sensitive Data Safe and Private](https://codelabs.developers.google.com/codelabs/android-storage-permissions/) 。

































