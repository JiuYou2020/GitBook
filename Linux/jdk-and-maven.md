---
description: 下载配置jdk和maven
---

# JDK & MAVEN

全局卸载jdk

```bash
sudo apt-get purge openjdk-\*
```

重新安装jdk

```bash
sudo apt-get update
sudo apt-get install openjdk-8-jdk
sudo apt-get install openjdk-17-jdk
```

设置默认java版本

```bash
sudo update-alternatives --config java
```



1. **下载和安装Maven**在您的终端中执行以下命令来下载和安装Maven：

```bash
   sudo apt-get update
   sudo apt-get install maven
```

2. **检查安装**安装完成后，您可以使用以下命令来检查其版本，以确认安装成功：

```bash
   mvn --version
```

3. **配置Maven的阿里镜像源**Maven的配置文件位于安装目录下的`conf`文件夹内，名为`settings.xml`。在此文件中，您可以配置阿里云镜像。首先，使用编辑器打开此文件，例如：

```bash
   sudo nano /etc/maven/settings.xml
```

然后，查找`<mirrors></mirrors>`标签，并在其中添加以下配置：

```xml
   <mirror>
     <id>alimaven</id>
     <mirrorOf>central</mirrorOf>
     <name>aliyun maven</name>
     <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
   </mirror>
```

添加后，保存并关闭文件。

4. **配置下载仓库地址:** 如果您还需要配置下载仓库的地址，您可以在`settings.xml`文件的`<localRepository></localRepository>`中配置，例如：

```xml
   <localRepository>/path/to/your/repo</localRepository>
```

其中，`/path/to/your/repo`是您希望设置的仓库地址。







