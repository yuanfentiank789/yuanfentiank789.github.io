---
layout: post
title:  "Android DiskLruCache完全解析，硬盘缓存的最佳方案"
date:   2016-06-13 1:05:00
catalog:  true
tags:
    - android

---

## 概述

记得在很早之前，我有写过一篇文章[**Android高效加载大图、多图解决方案，有效避免程序OOM**](http://blog.csdn.net/guolin_blog/article/details/9316683)，这篇文章是翻译自Android Doc的，其中防止多图OOM的核心解决思路就是使用LruCache技术。但LruCache只是管理了内存中图片的存储与释放，如果图片从内存中被移除的话，那么又需要从网络上重新加载一次图片，这显然非常耗时。对此，Google又提供了一套硬盘缓存的解决方案：DiskLruCache(非Google官方编写，但获得官方认证)。只可惜，Android Doc中并没有对DiskLruCache的用法给出详细的说明，而网上关于DiskLruCache的资料也少之又少，因此今天我准备专门写一篇博客来详细讲解DiskLruCache的用法，以及分析它的工作原理，这应该也是目前网上关于DiskLruCache最详细的资料了。

那么我们先来看一下有哪些应用程序已经使用了DiskLruCache技术。在我所接触的应用范围里，Dropbox、Twitter、网易新闻等都是使用DiskLruCache来进行硬盘缓存的，其中Dropbox和Twitter大多数人应该都没用过，那么我们就从大家最熟悉的网易新闻开始着手分析，来对DiskLruCache有一个最初的认识吧。
## 初探

相信所有人都知道，网易新闻中的数据都是从网络上获取的，包括了很多的新闻内容和新闻图片，如下图所示：

![](http://img.blog.csdn.net/20140803100719140?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZ3VvbGluX2Jsb2c=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

但是不知道大家有没有发现，这些内容和图片在从网络上获取到之后都会存入到本地缓存中，因此即使手机在没有网络的情况下依然能够加载出以前浏览过的新闻。而使用的缓存技术不用多说，自然是DiskLruCache了，那么首先第一个问题，这些数据都被缓存在了手机的什么位置呢？

其实DiskLruCache并没有限制数据的缓存位置，可以自由地进行设定，但是通常情况下多数应用程序都会将缓存的位置选择为 /sdcard/Android/data/&lt;application package&gt;/cache 这个路径。选择在这个位置有两点好处：第一，这是存储在SD卡上的，因此即使缓存再多的数据也不会对手机的内置存储空间有任何影响，只要SD卡空间足够就行。第二，这个路径被Android系统认定为应用程序的缓存路径，当程序被卸载的时候，这里的数据也会一起被清除掉，这样就不会出现删除程序之后手机上还有很多残留数据的问题。

那么这里还是以网易新闻为例，它的客户端的包名是com.netease.newsreader.activity，因此数据缓存地址就应该是&nbsp;/sdcard/Android/data/com.netease.newsreader.activity/cache ，我们进入到这个目录中看一下，结果如下图所示：

![](http://img.blog.csdn.net/20140803104231765?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZ3VvbGluX2Jsb2c=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

可以看到有很多个文件夹，因为网易新闻对多种类型的数据都进行了缓存，这里简单起见我们只分析图片缓存就好，所以进入到bitmap文件夹当中。然后你将会看到一堆文件名很长的文件，这些文件命名没有任何规则，完全看不懂是什么意思，但如果你一直向下滚动，将会看到一个名为journal的文件，如下图所示：

![](http://img.blog.csdn.net/20140803140924754?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZ3VvbGluX2Jsb2c=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

那么这些文件到底都是什么呢？看到这里相信有些朋友已经是一头雾水了，这里我简单解释一下。上面那些文件名很长的文件就是一张张缓存的图片，每个文件都对应着一张图片，而journal文件是DiskLruCache的一个日志文件，程序对每张图片的操作记录都存放在这个文件中，基本上看到journal这个文件就标志着该程序使用DiskLruCache技术了。

## 下载

好了，对DiskLruCache有了最初的认识之后，下面我们来学习一下DiskLruCache的用法吧。由于DiskLruCache并不是由Google官方编写的，所以这个类并没有被包含在Android API当中，我们需要将这个类从网上下载下来，然后手动添加到项目当中。DiskLruCache的源码在Google Source上，地址如下：

[android.googlesource.com/platform/libcore/+/jb-mr2-release/luni/src/main/java/libcore/io/DiskLruCache.java](https://android.googlesource.com/platform/libcore/+/jb-mr2-release/luni/src/main/java/libcore/io/DiskLruCache.java)

如果Google Source打不开的话，也可以[**点击这里**](http://download.csdn.net/detail/sinyu890807/7709759)下载DiskLruCache的源码。下载好了源码之后，只需要在项目中新建一个libcore.io包，然后将DiskLruCache.java文件复制到这个包中即可。

## 打开缓存

这样的话我们就把准备工作做好了，下面看一下DiskLruCache到底该如何使用。首先你要知道，DiskLruCache是不能new出实例的，如果我们要创建一个DiskLruCache的实例，则需要调用它的open()方法，接口如下所示：

```
public static DiskLruCache open(File directory, int appVersion, int valueCount, long maxSize)
```
open()方法接收四个参数，第一个参数指定的是数据的缓存地址，第二个参数指定当前应用程序的版本号，第三个参数指定同一个key可以对应多少个缓存文件，基本都是传1，第四个参数指定最多可以缓存多少字节的数据。

其中缓存地址前面已经说过了，通常都会存放在&nbsp;/sdcard/Android/data/&lt;application package&gt;/cache 这个路径下面，但同时我们又需要考虑如果这个手机没有SD卡，或者SD正好被移除了的情况，因此比较优秀的程序都会专门写一个方法来获取缓存地址，如下所示：

```
public File getDiskCacheDir(Context context, String uniqueName) {
	String cachePath;
	if (Environment.MEDIA_MOUNTED.equals(Environment.getExternalStorageState())
			|| !Environment.isExternalStorageRemovable()) {
		cachePath = context.getExternalCacheDir().getPath();
	} else {
		cachePath = context.getCacheDir().getPath();
	}
	return new File(cachePath + File.separator + uniqueName);
}
```
可以看到，当SD卡存在或者SD卡不可被移除的时候，就调用getExternalCacheDir()方法来获取缓存路径，否则就调用getCacheDir()方法来获取缓存路径。前者获取到的就是&nbsp;/sdcard/Android/data/&lt;application package&gt;/cache 这个路径，而后者获取到的是 /data/data/&lt;application package&gt;/cache 这个路径。

接着又将获取到的路径和一个uniqueName进行拼接，作为最终的缓存路径返回。那么这个uniqueName又是什么呢？其实这就是为了对不同类型的数据进行区分而设定的一个唯一值，比如说在网易新闻缓存路径下看到的bitmap、object等文件夹。

接着是应用程序版本号，我们可以使用如下代码简单地获取到当前应用程序的版本号：

```
	public int getAppVersion(Context context) {
		try {
			PackageInfo info = context.getPackageManager().getPackageInfo(context.getPackageName(), 0);
			return info.versionCode;
		} catch (NameNotFoundException e) {
			e.printStackTrace();
		}
		return 1;
	}
```
需要注意的是，每当版本号改变，缓存路径下存储的所有数据都会被清除掉，因为DiskLruCache认为当应用程序有版本更新的时候，所有的数据都应该从网上重新获取。

后面两个参数就没什么需要解释的了，第三个参数传1，第四个参数通常传入10M的大小就够了，这个可以根据自身的情况进行调节。

因此，一个非常标准的open()方法就可以这样写：

```
DiskLruCache mDiskLruCache = null;
try {
	File cacheDir = getDiskCacheDir(context, "bitmap");
	if (!cacheDir.exists()) {
		cacheDir.mkdirs();
	}
	mDiskLruCache = DiskLruCache.open(cacheDir, getAppVersion(context), 1, 10 * 1024 * 1024);
} catch (IOException e) {
	e.printStackTrace();
}
```
首先调用getDiskCacheDir()方法获取到缓存地址的路径，然后判断一下该路径是否存在，如果不存在就创建一下。接着调用DiskLruCache的open()方法来创建实例，并把四个参数传入即可。

有了DiskLruCache的实例之后，我们就可以对缓存的数据进行操作了，操作类型主要包括写入、访问、移除等，我们一个个进行学习。

## DiskLruCache#open源码分析
     ```
	public static DiskLruCache open(File directory, int appVersion, int valueCount, long maxSize)
      throws IOException {

    // If a bkp file exists, use it instead.
    File backupFile = new File(directory, JOURNAL_FILE_BACKUP);
    if (backupFile.exists()) {
      File journalFile = new File(directory, JOURNAL_FILE);
      // If journal file also exists just delete backup file.
      if (journalFile.exists()) {
        backupFile.delete();
      } else {
        renameTo(backupFile, journalFile, false);
      }
    }

    // Prefer to pick up where we left off.
    DiskLruCache cache = new DiskLruCache(directory, appVersion, valueCount, maxSize);
    if (cache.journalFile.exists()) {
      try {
        cache.readJournal();
        cache.processJournal();
        return cache;
      } catch (IOException journalIsCorrupt) {
        System.out
            .println("DiskLruCache "
                + directory
                + " is corrupt: "
                + journalIsCorrupt.getMessage()
                + ", removing");
        cache.delete();
      }
    }

    // Create a new empty cache.
    directory.mkdirs();
    cache = new DiskLruCache(directory, appVersion, valueCount, maxSize);
    cache.rebuildJournal();
    return cache;
    }
     
    ```

首先检查存不存在journal.bkp（journal的备份文件）

如果存在：然后检查journal文件是否存在，如果正主在，bkp文件就可以删除了。 
如果不存在，将bkp文件重命名为journal文件。

接下里判断journal文件是否存在：

**如果不存在**

创建directory；重新构造disklrucache；调用rebuildJournal建立journal文件 
  
     ```
       /**
     * Creates a new journal that omits redundant information. This replaces the
     * current journal if it exists.
     */
    private synchronized void rebuildJournal() throws IOException {
        if (journalWriter != null) {
            journalWriter.close();
        }

        Writer writer = new BufferedWriter(new FileWriter(journalFileTmp), IO_BUFFER_SIZE);
        writer.write(MAGIC);
        writer.write("\n");
        writer.write(VERSION_1);
        writer.write("\n");
        writer.write(Integer.toString(appVersion));
        writer.write("\n");
        writer.write(Integer.toString(valueCount));
        writer.write("\n");
        writer.write("\n");

        for (Entry entry : lruEntries.values()) {
            if (entry.currentEditor != null) {
                writer.write(DIRTY + ' ' + entry.key + '\n');
            } else {
                writer.write(CLEAN + ' ' + entry.key + entry.getLengths() + '\n');
            }
        }

        writer.close();
        journalFileTmp.renameTo(journalFile);
        journalWriter = new BufferedWriter(new FileWriter(journalFile, true), IO_BUFFER_SIZE);
    }
        
     ```
  可以看到首先构建一个journal.tmp文件，然后写入文件头（5行），然后遍历lruEntries（lruEntries = 
new LinkedHashMap<String, Entry>(0, 0.75f, true);），当然我们这里没有任何数据。接下来将tmp文件重命名为journal文件。

**如果存在**

如果已经存在，那么调用readJournal。

    ```
      private void readJournal() throws IOException {
        InputStream in = new BufferedInputStream(new FileInputStream(journalFile), IO_BUFFER_SIZE);
        try {
            String magic = readAsciiLine(in);
            String version = readAsciiLine(in);
            String appVersionString = readAsciiLine(in);
            String valueCountString = readAsciiLine(in);
            String blank = readAsciiLine(in);
            if (!MAGIC.equals(magic)
                    || !VERSION_1.equals(version)
                    || !Integer.toString(appVersion).equals(appVersionString)
                    || !Integer.toString(valueCount).equals(valueCountString)
                    || !"".equals(blank)) {
                throw new IOException("unexpected journal header: ["
                        + magic + ", " + version + ", " + valueCountString + ", " + blank + "]");
            }

            while (true) {
                try {
                    readJournalLine(readAsciiLine(in));
                } catch (EOFException endOfJournal) {
                    break;
                }
            }
        } finally {
            closeQuietly(in);
        }
    }
    ```
    
  首先校验文件头，接下来调用readJournalLine按行读取内容。我们来看看readJournalLine中的操作。         
     
        ```
         private void readJournalLine(String line) throws IOException {
        String[] parts = line.split(" ");
        if (parts.length < 2) {
            throw new IOException("unexpected journal line: " + line);
        }

        String key = parts[1];
        if (parts[0].equals(REMOVE) && parts.length == 2) {
            lruEntries.remove(key);
            return;
        }

        Entry entry = lruEntries.get(key);
        if (entry == null) {
            entry = new Entry(key);
            lruEntries.put(key, entry);
        }

        if (parts[0].equals(CLEAN) && parts.length == 2 + valueCount) {
            entry.readable = true;
            entry.currentEditor = null;
            entry.setLengths(copyOfRange(parts, 2, parts.length));
        } else if (parts[0].equals(DIRTY) && parts.length == 2) {
            entry.currentEditor = new Editor(entry);
        } else if (parts[0].equals(READ) && parts.length == 2) {
            // this work was already done by calling lruEntries.get()
        } else {
            throw new IOException("unexpected journal line: " + line);
        }
    }
        ```
        
   大家可以回忆下：每个记录至少有一个空格，有的包含两个空格。首先，拿到key，如果是REMOVE的记录呢，会调用lruEntries.remove(key);

如果不是REMOVE记录，继续往下，如果该key没有加入到lruEntries，则创建并且加入。

接下来，如果是CLEAN开头的合法记录，初始化entry，设置readable=true,currentEditor为null，初始化长度等。

如果是DIRTY，设置currentEditor对象。

如果是READ，那么直接不管。

ok，经过上面这个过程，大家回忆下我们的记录格式，一般DIRTY不会单独出现，会和REMOVE、CLEAN成对出现（正常操作）；也就是说，经过上面这个流程，基本上加入到lruEntries里面的只有CLEAN且没有被REMOVE的key。

好了，回到readJournal方法，在我们按行读取的时候，会记录一下lineCount，然后最后给redundantOpCount赋值，这个变量记录的应该是没用的记录条数（文件的行数-真正可以的key的行数）。

最后，如果读取过程中发现journal文件有问题，则重建journal文件。没有问题的话，初始化下journalWriter，关闭reader。

readJournal完成了，会继续调用processJournal()这个方法内部：

       ```
       /**
        * Computes the initial size and collects garbage as a part of opening the
        * cache. Dirty entries are assumed to be inconsistent and will be deleted.
        */
    private void processJournal() throws IOException {
        deleteIfExists(journalFileTmp);
        for (Iterator<Entry> i = lruEntries.values().iterator(); i.hasNext(); ) {
            Entry entry = i.next();
            if (entry.currentEditor == null) {
                for (int t = 0; t < valueCount; t++) {
                    size += entry.lengths[t];
                }
            } else {
                entry.currentEditor = null;
                for (int t = 0; t < valueCount; t++) {
                    deleteIfExists(entry.getCleanFile(t));
                    deleteIfExists(entry.getDirtyFile(t));
                }
                i.remove();
            }
        }
    }       
       ```      
   统计所有可用的cache占据的容量，赋值给size；对于所有非法DIRTY状态（就是DIRTY单独出现的）的entry，如果存在文件则删除，并且从lruEntries中移除。此时，剩的就真的只有CLEAN状态的key记录了。

ok，到此就初始化完毕了，太长了，根本记不住，我带大家总结下上面代码。

根据我们传入的dir，去找journal文件，如果找不到，则创建个，只写入文件头(5行)。 
如果找到，则遍历该文件，将里面所有的CLEAN记录的key，存到lruEntries中。

这么长的代码，其实就两句话的意思。经过open以后，journal文件肯定存在了；lruEntries里面肯定有值了；size存储了当前所有的实体占据的容量；。

## 写入缓存

先来看写入，比如说现在有一张图片，地址是http://img.my.csdn.net/uploads/201309/01/1378037235_7476.jpg，那么为了将这张图片下载下来，就可以这样写：

```
private boolean downloadUrlToStream(String urlString, OutputStream outputStream) {
	HttpURLConnection urlConnection = null;
	BufferedOutputStream out = null;
	BufferedInputStream in = null;
	try {
		final URL url = new URL(urlString);
		urlConnection = (HttpURLConnection) url.openConnection();
		in = new BufferedInputStream(urlConnection.getInputStream(), 8 * 1024);
		out = new BufferedOutputStream(outputStream, 8 * 1024);
		int b;
		while ((b = in.read()) != -1) {
			out.write(b);
		}
		return true;
	} catch (final IOException e) {
		e.printStackTrace();
	} finally {
		if (urlConnection != null) {
			urlConnection.disconnect();
		}
		try {
			if (out != null) {
				out.close();
			}
			if (in != null) {
				in.close();
			}
		} catch (final IOException e) {
			e.printStackTrace();
		}
	}
	return false;
}
```
这段代码相当基础，相信大家都看得懂，就是访问urlString中传入的网址，并通过outputStream写入到本地。有了这个方法之后，下面我们就可以使用DiskLruCache来进行写入了，写入的操作是借助DiskLruCache.Editor这个类完成的。类似地，这个类也是不能new的，需要调用DiskLruCache的edit()方法来获取实例，接口如下所示：
```
public Editor edit(String key) throws IOException
```
可以看到，edit()方法接收一个参数key，这个key将会成为缓存文件的文件名，并且必须要和图片的URL是一一对应的。那么怎样才能让key和图片的URL能够一一对应呢？直接使用URL来作为key？不太合适，因为图片URL中可能包含一些特殊字符，这些字符有可能在命名文件时是不合法的。其实最简单的做法就是将图片的URL进行MD5编码，编码后的字符串肯定是唯一的，并且只会包含0-F这样的字符，完全符合文件的命名规则。

那么我们就写一个方法用来将字符串进行MD5编码，代码如下所示：

```
public String hashKeyForDisk(String key) {
	String cacheKey;
	try {
		final MessageDigest mDigest = MessageDigest.getInstance("MD5");
		mDigest.update(key.getBytes());
		cacheKey = bytesToHexString(mDigest.digest());
	} catch (NoSuchAlgorithmException e) {
		cacheKey = String.valueOf(key.hashCode());
	}
	return cacheKey;
}
private String bytesToHexString(byte[] bytes) {  
    StringBuilder sb = new StringBuilder();  
    for (int i = 0; i < bytes.length; i++) {  
        String hex = Integer.toHexString(0xFF & bytes[i]);  
        if (hex.length() == 1) {  
            sb.append('0');  
        }  
        sb.append(hex);  
    }  
    return sb.toString();  
}
```
代码很简单，现在我们只需要调用一下hashKeyForDisk()方法，并把图片的URL传入到这个方法中，就可以得到对应的key了。

因此，现在就可以这样写来得到一个DiskLruCache.Editor的实例：

```
String imageUrl = "http://img.my.csdn.net/uploads/201309/01/1378037235_7476.jpg";
String key = hashKeyForDisk(imageUrl);
DiskLruCache.Editor editor = mDiskLruCache.edit(key);
```
有了DiskLruCache.Editor的实例之后，我们可以调用它的newOutputStream()方法来创建一个输出流，然后把它传入到downloadUrlToStream()中就能实现下载并写入缓存的功能了。注意newOutputStream()方法接收一个index参数，由于前面在设置valueCount的时候指定的是1，所以这里index传0就可以了。在写入操作执行完之后，我们还需要调用一下commit()方法进行提交才能使写入生效，调用abort()方法的话则表示放弃此次写入。

因此，一次完整写入操作的代码如下所示：

```
new Thread(new Runnable() {
	@Override
	public void run() {
		try {
			String imageUrl = "http://img.my.csdn.net/uploads/201309/01/1378037235_7476.jpg";
			String key = hashKeyForDisk(imageUrl);
			DiskLruCache.Editor editor = mDiskLruCache.edit(key);
			if (editor != null) {
				OutputStream outputStream = editor.newOutputStream(0);
				if (downloadUrlToStream(imageUrl, outputStream)) {
					editor.commit();
				} else {
					editor.abort();
				}
			}
			mDiskLruCache.flush();
		} catch (IOException e) {
			e.printStackTrace();
		}
	}
}).start();
```
由于这里调用了downloadUrlToStream()方法来从网络上下载图片，所以一定要确保这段代码是在子线程当中执行的。注意在代码的最后我还调用了一下flush()方法，这个方法并不是每次写入都必须要调用的，但在这里却不可缺少，我会在后面说明它的作用。

现在的话缓存应该是已经成功写入了，我们进入到SD卡上的缓存目录里看一下，如下图所示：

![](http://img.blog.csdn.net/20140803220637609?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZ3VvbGluX2Jsb2c=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

可以看到，这里有一个文件名很长的文件，和一个journal文件，那个文件名很长的文件自然就是缓存的图片了，因为是使用了MD5编码来进行命名的。

## 存入缓存源码分析

还记得，我们前面说过是怎么存的么？
    
    ```
       String key = generateKey(url);  
       DiskLruCache.Editor editor = mDiskLruCache.edit(key); 
       OuputStream os = editor.newOutputStream(0); 
       //...after op
       editor.commit()；
    ```
  那么首先就是editor方法；
  
     ```
        /**
     * Returns an editor for the entry named {@code key}, or null if another
     * edit is in progress.
     */
    public Editor edit(String key) throws IOException {
        return edit(key, ANY_SEQUENCE_NUMBER);
    }

    private synchronized Editor edit(String key, long expectedSequenceNumber) throws IOException {
        checkNotClosed();
        validateKey(key);
        Entry entry = lruEntries.get(key);
        if (expectedSequenceNumber != ANY_SEQUENCE_NUMBER
                && (entry == null || entry.sequenceNumber != expectedSequenceNumber)) {
            return null; // snapshot is stale
        }
        if (entry == null) {
            entry = new Entry(key);
            lruEntries.put(key, entry);
        } else if (entry.currentEditor != null) {
            return null; // another edit is in progress
        }

        Editor editor = new Editor(entry);
        entry.currentEditor = editor;

        // flush the journal before creating files to prevent file leaks
        journalWriter.write(DIRTY + ' ' + key + '\n');
        journalWriter.flush();
        return editor;
    }

     ```
   首先验证key，可以必须是字母、数字、下划线、横线（-）组成，且长度在1-120之间。

然后通过key获取实体，因为我们是存，只要不是正在编辑这个实体，理论上都能返回一个合法的editor对象。

所以接下来判断，如果不存在，则创建一个Entry加入到lruEntries中（如果存在，直接使用），然后为entry.currentEditor进行赋值为new Editor(entry);，最后在journal文件中写入一条DIRTY记录，代表这个文件正在被操作。

注意，如果entry.currentEditor != null不为null的时候，意味着该实体正在被编辑，会retrun null ;

拿到editor对象以后，就是去调用newOutputStream去获得一个文件输入流了。
    
    ```
        /**
         * Returns a new unbuffered output stream to write the value at
         * {@code index}. If the underlying output stream encounters errors
         * when writing to the filesystem, this edit will be aborted when
         * {@link #commit} is called. The returned output stream does not throw
         * IOExceptions.
         */
        public OutputStream newOutputStream(int index) throws IOException {
            if (index < 0 || index >= valueCount) {
                throw new IllegalArgumentException("Expected index " + index + " to "
                        + "be greater than 0 and less than the maximum value count "
                        + "of " + valueCount);
            }
            synchronized (DiskLruCache.this) {
                if (entry.currentEditor != this) {
                    throw new IllegalStateException();
                }
                if (!entry.readable) {
                    written[index] = true;
                }
                File dirtyFile = entry.getDirtyFile(index);
                FileOutputStream outputStream;
                try {
                    outputStream = new FileOutputStream(dirtyFile);
                } catch (FileNotFoundException e) {
                    // Attempt to recreate the cache directory.
                    directory.mkdirs();
                    try {
                        outputStream = new FileOutputStream(dirtyFile);
                    } catch (FileNotFoundException e2) {
                        // We are unable to recover. Silently eat the writes.
                        return NULL_OUTPUT_STREAM;
                    }
                }
                return new FaultHidingOutputStream(outputStream);
            }
        }
    ```
  首先校验index是否在valueCount范围内，一般我们使用都是一个key对应一个文件所以传入的基本都是0。接下来就是通过entry.getDirtyFile(index);拿到一个dirty File对象，为什么叫dirty file呢，其实就是个中转文件，文件格式为key.index.tmp。 
将这个文件的FileOutputStream通过FaultHidingOutputStream封装下传给我们。

最后，别忘了我们通过os写入数据以后，需要调用commit方法。

    ```
      public void commit() throws IOException {
            if (hasErrors) {
                completeEdit(this, false);
                remove(entry.key); // the previous entry is stale
            } else {
                completeEdit(this, true);
            }
        }
    ```
   首先通过hasErrors判断，是否有错误发生，如果有调用completeEdit(this, false)且调用remove(entry.key);。如果没有就调用completeEdit(this, true);。

那么这里这个hasErrors哪来的呢？还记得上面newOutputStream的时候，返回了一个os，这个os是FileOutputStream，但是经过了FaultHidingOutputStream封装么，这个类实际上就是重写了FilterOutputStream的write相关方法，将所有的IOException给屏蔽了，如果发生IOException就将hasErrors赋值为true.

这样的设计还是很nice的，否则直接将OutputStream返回给用户，如果出错没法检测，还需要用户手动去调用一些操作。

接下来看completeEdit方法。

    ```
        private synchronized void completeEdit(Editor editor, boolean success) throws IOException {
        Entry entry = editor.entry;
        if (entry.currentEditor != editor) {
            throw new IllegalStateException();
        }

        // if this edit is creating the entry for the first time, every index must have a value
        if (success && !entry.readable) {
            for (int i = 0; i < valueCount; i++) {
                if (!entry.getDirtyFile(i).exists()) {
                    editor.abort();
                    throw new IllegalStateException("edit didn't create file " + i);
                }
            }
        }

        for (int i = 0; i < valueCount; i++) {
            File dirty = entry.getDirtyFile(i);
            if (success) {
                if (dirty.exists()) {
                    File clean = entry.getCleanFile(i);
                    dirty.renameTo(clean);
                    long oldLength = entry.lengths[i];
                    long newLength = clean.length();
                    entry.lengths[i] = newLength;
                    size = size - oldLength + newLength;
                }
            } else {
                deleteIfExists(dirty);
            }
        }

        redundantOpCount++;
        entry.currentEditor = null;
        if (entry.readable | success) {
            entry.readable = true;
            journalWriter.write(CLEAN + ' ' + entry.key + entry.getLengths() + '\n');
            if (success) {
                entry.sequenceNumber = nextSequenceNumber++;
            }
        } else {
            lruEntries.remove(entry.key);
            journalWriter.write(REMOVE + ' ' + entry.key + '\n');
        }

        if (size > maxSize || journalRebuildRequired()) {
            executorService.submit(cleanupCallable);
        }
    }

    ```
  首先判断if (success && !entry.readable)是否成功，且是第一次写入（如果以前这个记录有值，则readable=true），内部的判断，我们都不会走，因为written[i]在newOutputStream的时候被写入true了。而且正常情况下，getDirtyFile是存在的。

接下来，如果成功，将dirtyFile 进行重命名为 cleanFile，文件名为：key.index。然后刷新size的长度。如果失败，则删除dirtyFile.

接下来，如果成功或者readable为true，将readable设置为true，写入一条CLEAN记录。如果第一次提交且失败，那么就会从lruEntries.remove(key)，写入一条REMOVE记录。

写入缓存，肯定要控制下size。于是最后，判断是否超过了最大size，或者需要重建journal文件，什么时候需要重建呢？

    ```
    /**
     * We only rebuild the journal when it will halve the size of the journal
     * and eliminate at least 2000 ops.
     */
    private boolean journalRebuildRequired() {
        final int REDUNDANT_OP_COMPACT_THRESHOLD = 2000;
        return redundantOpCount >= REDUNDANT_OP_COMPACT_THRESHOLD
                && redundantOpCount >= lruEntries.size();
    }
    ```
   如果redundantOpCount达到2000，且超过了lruEntries.size()就重建，这里就可以看到redundantOpCount的作用了。防止journal文件过大。

ok，到此我们的存入缓存就分析完成了。再次总结下，首先调用editor，拿到指定的dirtyFile的OutputStream，你可以尽情的进行写操作，写完以后呢，记得调用commit. 
commit中会检测是你是否发生IOException，如果没有发生，则将dirtyFile->cleanFile，将readable=true，写入CLEAN记录。如果发生错误，则删除dirtyFile,从lruEntries中移除，然后写入一条REMOVE记录。
## 读取缓存

缓存已经写入成功之后，接下来我们就该学习一下如何读取了。读取的方法要比写入简单一些，主要是借助DiskLruCache的get()方法实现的，接口如下所示：

```
public synchronized Snapshot get(String key) throws IOException
```
很明显，get()方法要求传入一个key来获取到相应的缓存数据，而这个key毫无疑问就是将图片URL进行MD5编码后的值了，因此读取缓存数据的代码就可以这样写：

```
String imageUrl = "http://img.my.csdn.net/uploads/201309/01/1378037235_7476.jpg";
String key = hashKeyForDisk(imageUrl);
DiskLruCache.Snapshot snapShot = mDiskLruCache.get(key);
```
很奇怪的是，这里获取到的是一个DiskLruCache.Snapshot对象，这个对象我们该怎么利用呢？很简单，只需要调用它的getInputStream()方法就可以得到缓存文件的输入流了。同样地，getInputStream()方法也需要传一个index参数，这里传入0就好。有了文件的输入流之后，想要把缓存图片显示到界面上就轻而易举了。所以，一段完整的读取缓存，并将图片加载到界面上的代码如下所示：

```
try {
	String imageUrl = "http://img.my.csdn.net/uploads/201309/01/1378037235_7476.jpg";
	String key = hashKeyForDisk(imageUrl);
	DiskLruCache.Snapshot snapShot = mDiskLruCache.get(key);
	if (snapShot != null) {
		InputStream is = snapShot.getInputStream(0);
		Bitmap bitmap = BitmapFactory.decodeStream(is);
		mImage.setImageBitmap(bitmap);
	}
} catch (IOException e) {
	e.printStackTrace();
}
```

我们使用了BitmapFactory的decodeStream()方法将文件流解析成Bitmap对象，然后把它设置到ImageView当中。如果运行一下程序，将会看到如下效果：

![](http://img.blog.csdn.net/20140803235312515?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZ3VvbGluX2Jsb2c=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

OK，图片已经成功显示出来了。注意这是我们从本地缓存中加载的，而不是从网络上加载的，因此即使在你手机没有联网的情况下，这张图片仍然可以显示出来。

## 读取缓存源码分析

    ```
       DiskLruCache.Snapshot snapShot = mDiskLruCache.get(key);  
       if (snapShot != null) {  
           InputStream is = snapShot.getInputStream(0);  
        }
    ```
 那么首先看get方法：
 
     ```
     /**
     * Returns a snapshot of the entry named {@code key}, or null if it doesn't
     * exist is not currently readable. If a value is returned, it is moved to
     * the head of the LRU queue.
     */
    public synchronized Snapshot get(String key) throws IOException {
        checkNotClosed();
        validateKey(key);
        Entry entry = lruEntries.get(key);
        if (entry == null) {
            return null;
        }

        if (!entry.readable) {
            return null;
        }

        /*
         * Open all streams eagerly to guarantee that we see a single published
         * snapshot. If we opened streams lazily then the streams could come
         * from different edits.
         */
        InputStream[] ins = new InputStream[valueCount];
        try {
            for (int i = 0; i < valueCount; i++) {
                ins[i] = new FileInputStream(entry.getCleanFile(i));
            }
        } catch (FileNotFoundException e) {
            // a file must have been deleted manually!
            return null;
        }

        redundantOpCount++;
        journalWriter.append(READ + ' ' + key + '\n');
        if (journalRebuildRequired()) {
            executorService.submit(cleanupCallable);
        }

        return new Snapshot(key, entry.sequenceNumber, ins);
    }
     ```
     
   get方法比较简单，如果取到的为null，或者readable=false，则返回null.否则将cleanFile的FileInputStream进行封装返回Snapshot，且写入一条READ语句。 
然后getInputStream就是返回该FileInputStream了。

好了，到此，我们就分析完成了创建DiskLruCache，存入缓存和取出缓存的源码。
## 移除缓存

学习完了写入缓存和读取缓存的方法之后，最难的两个操作你就都已经掌握了，那么接下来要学习的移除缓存对你来说也一定非常轻松了。移除缓存主要是借助DiskLruCache的remove()方法实现的，接口如下所示：

```
public synchronized boolean remove(String key) throws IOException</pre>相信你已经相当熟悉了，remove()方法中要求传入一个key，然后会删除这个key对应的缓存图片，示例代码如下：<pre code_snippet_id="444567" snippet_file_name="blog_20140807_14_2509132" name="code" class="java">try {
	String imageUrl = "http://img.my.csdn.net/uploads/201309/01/1378037235_7476.jpg";  
	String key = hashKeyForDisk(imageUrl);  
	mDiskLruCache.remove(key);
} catch (IOException e) {
	e.printStackTrace();
}
```

用法虽然简单，但是你要知道，这个方法我们并不应该经常去调用它。因为你完全不需要担心缓存的数据过多从而占用SD卡太多空间的问题，DiskLruCache会根据我们在调用open()方法时设定的缓存最大值来自动删除多余的缓存。只有你确定某个key对应的缓存内容已经过期，需要从网络获取最新数据的时候才应该调用remove()方法来移除缓存。
## 其它API

除了写入缓存、读取缓存、移除缓存之外，DiskLruCache还提供了另外一些比较常用的API，我们简单学习一下。

1.&nbsp;size()

这个方法会返回当前缓存路径下所有缓存数据的总字节数，以byte为单位，如果应用程序中需要在界面上显示当前缓存数据的总大小，就可以通过调用这个方法计算出来。比如网易新闻中就有这样一个功能，如下图所示：

![](http://img.blog.csdn.net/20140804204157148?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZ3VvbGluX2Jsb2c=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

2.flush()

这个方法用于将内存中的操作记录同步到日志文件（也就是journal文件）当中。这个方法非常重要，因为DiskLruCache能够正常工作的前提就是要依赖于journal文件中的内容。前面在讲解写入缓存操作的时候我有调用过一次这个方法，但其实并不是每次写入缓存都要调用一次flush()方法的，频繁地调用并不会带来任何好处，只会额外增加同步journal文件的时间。比较标准的做法就是在Activity的onPause()方法中去调用一次flush()方法就可以了。

3.close()

这个方法用于将DiskLruCache关闭掉，是和open()方法对应的一个方法。关闭掉了之后就不能再调用DiskLruCache中任何操作缓存数据的方法，通常只应该在Activity的onDestroy()方法中去调用close()方法。

4.delete()

这个方法用于将所有的缓存数据全部删除，比如说网易新闻中的那个手动清理缓存功能，其实只需要调用一下DiskLruCache的delete()方法就可以实现了。

## 解读journal

前面已经提到过，DiskLruCache能够正常工作的前提就是要依赖于journal文件中的内容，因此，能够读懂journal文件对于我们理解DiskLruCache的工作原理有着非常重要的作用。那么journal文件中的内容到底是什么样的呢？我们来打开瞧一瞧吧，如下图所示：

![](http://img.blog.csdn.net/20140804233158296?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZ3VvbGluX2Jsb2c=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

由于现在只缓存了一张图片，所以journal中并没有几行日志，我们一行行进行分析。第一行是个固定的字符串“libcore.io.DiskLruCache”，标志着我们使用的是DiskLruCache技术。第二行是DiskLruCache的版本号，这个值是恒为1的。第三行是应用程序的版本号，我们在open()方法里传入的版本号是什么这里就会显示什么。第四行是valueCount，这个值也是在open()方法中传入的，通常情况下都为1。第五行是一个空行。前五行也被称为journal文件的头，这部分内容还是比较好理解的，但是接下来的部分就要稍微动点脑筋了。

第六行是以一个DIRTY前缀开始的，后面紧跟着缓存图片的key。通常我们看到DIRTY这个字样都不代表着什么好事情，意味着这是一条脏数据。没错，每当我们调用一次DiskLruCache的edit()方法时，都会向journal文件中写入一条DIRTY记录，表示我们正准备写入一条缓存数据，但不知结果如何。然后调用commit()方法表示写入缓存成功，这时会向journal中写入一条CLEAN记录，意味着这条“脏”数据被“洗干净了”，调用abort()方法表示写入缓存失败，这时会向journal中写入一条REMOVE记录。也就是说，每一行DIRTY的key，后面都应该有一行对应的CLEAN或者REMOVE的记录，否则这条数据就是“脏”的，会被自动删除掉。

如果你足够细心的话应该还会注意到，第七行的那条记录，除了CLEAN前缀和key之外，后面还有一个152313，这是什么意思呢？其实，DiskLruCache会在每一行CLEAN记录的最后加上该条缓存数据的大小，以字节为单位。152313也就是我们缓存的那张图片的字节数了，换算出来大概是148.74K，和缓存图片刚刚好一样大，如下图所示：

![](http://img.blog.csdn.net/20140805223723516?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZ3VvbGluX2Jsb2c=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

前面我们所学的size()方法可以获取到当前缓存路径下所有缓存数据的总字节数，其实它的工作原理就是把journal文件中所有CLEAN记录的字节数相加，求出的总合再把它返回而已。

除了DIRTY、CLEAN、REMOVE之外，还有一种前缀是READ的记录，这个就非常简单了，每当我们调用get()方法去读取一条缓存数据时，就会向journal文件中写入一条READ记录。因此，像网易新闻这种图片和数据量都非常大的程序，journal文件中就可能会有大量的READ记录。

那么你可能会担心了，如果我不停频繁操作的话，就会不断地向journal文件中写入数据，那这样journal文件岂不是会越来越大？这倒不必担心，DiskLruCache中使用了一个redundantOpCount变量来记录用户操作的次数，每执行一次写入、读取或移除缓存的操作，这个变量值都会加1，当变量值达到2000的时候就会触发重构journal的事件，这时会自动把journal中一些多余的、不必要的记录全部清除掉，保证journal文件的大小始终保持在一个合理的范围内。

好了，这样的话我们就算是把DiskLruCache的用法以及简要的工作原理分析完了。至于DiskLruCache的源码还是比较简单的， 限于篇幅原因就不在这里展开了，感兴趣的朋友可以自己去摸索。下一篇文章中，我会带着大家通过一个项目实战的方式来更加深入地理解DiskLruCache的用法。感兴趣的朋友请继续阅读 [**Android照片墙完整版，完美结合LruCache和DiskLruCache**](http://blog.csdn.net/guolin_blog/article/details/34093441)&nbsp;。


