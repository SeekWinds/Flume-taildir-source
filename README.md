# Flume-taildir-source
**修改Flume taildir source源码,使其支持递归读取指定目录下的文件变化**

**问题引入**

> 之前的Flume在使用Flume taildir作为source的时候,只能监听文件夹下面的指定后缀的文件的变化,但是如果文件夹下面有递归文件夹,就不能监听递归文件夹中指定后缀的文件的变化.

**例如:**
test目录下只有a.txt一个文件,我们在flume的配置文件配置监听,在不修改源码的情况下,只会监听a.txt文件的变化

```shell
[hadoop@hadoop001 /opt/scripts/producer/test] 10.25.13.21 
$ ll
total 0
-rw-rw-r--. 1 hryadm hryadm 0 Jun 24 09:59 a.txt
```

但是如果test文件夹下出现了新的文件夹而不是文件,就不会监听到

```shell
[hadoop@hadoop001 /opt/scripts/producer/test] 10.25.13.21 
$ ll
total 0
-rw-rw-r--. 1 hryadm hryadm 0 Jun 24 09:59 a.txt
-rw-rw-r--. 1 hryadm hryadm 0 Jun 24 10:01 file
```

**修改源码支持递归**

主要是修改**org.apache.flume.source.taildir.TaildirMatcher**这个类中的getMatchingFilesNoCache方法,使其支持递归操作

```java
private List<File> getMatchingFilesNoCache() {
    List<File> result = Lists.newArrayList();
    try (DirectoryStream<Path> stream = Files.newDirectoryStream(parentDir.toPath(), fileFilter)) {
      for (Path entry : stream) {
        result.add(entry.toFile());
      }
    } catch (IOException e) {
      logger.error("I/O exception occurred while listing parent directory. " +
                   "Files already matched will be returned. " + parentDir.toPath(), e);
    }
    return result;
  }
```

**修改后代码如下**

```java
/**
   * 根据上边原本的的非递归方法,写一个递归的方法
   */
  private List<File> getMatchingRecurseFilesNoCache() {
    List<File> result = Lists.newArrayList();

    List<Path> paths = recurseFolder(parentDir);
    for(Path path:paths){
      try (DirectoryStream<Path> stream = Files.newDirectoryStream(path, fileFilter)) {
        for (Path entry : stream) {
          result.add(entry.toFile());
        }
      } catch (IOException e) {
        e.printStackTrace();
      }
    }

    return result;
  }

  /**
   * 通过父文件夹路径遍历出父路径下所有的文件夹路径
   * 然后把文件夹路径集合返回回去
   * @param parentDir
   * @return
   */
  public List<Path> recurseFolder(File parentDir) {
    List<Path> allParentFolders = new ArrayList<>();
    allParentFolders.add(parentDir.toPath());

    if (parentDir.exists()) {
      File[] files = parentDir.listFiles();
      if (files == null || files.length == 0) {
        return allParentFolders;
      } else {
        for (File subFile : files) {
          if (subFile.isDirectory()) {
            allParentFolders.addAll(recurseFolder(subFile));
          }
        }
      }
    }
    return allParentFolders;
  }
```

