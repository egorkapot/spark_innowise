<h1 align="center">Spark Demo Course</h1>


## Description

This README contains instuction on how to install PySpark on your local Windows machine and start on it jobs.

<p align="center">
<img src="http://habrastorage.org/getpro/habr/post_images/550/b31/bd9/550b31bd91269dd77ce8b0041798d8f8.png" width="80%"></p>

## Pre-requirements

### Python

You need have Python on your local machine. If you don't have it, go to https://www.python.org/downloads/windows/ and install latest version.
After the installation is complete, close the Command Prompt if it was already open, reopen it and check if you can successfully run 
```
python --version
```
command.
After this install jupyter notebook, pandas, numpy, pyspark and findspark(you can easily find guides in the Web) and all other staff you need.

### Msvcr100.dll

If you have Visual Studio c++ skip this step.
If you don't have follow this guide https://www.computer-setup.ru/msvcr100-dll-chto-eto-za-oshibka-kak-ispravit?ysclid=l84uyxs1qq310355183 to install Msvcr100.dll.

### Kaggle

You also need to register on kaggle(for example using your google account) to download datasets.

### Microsoft SQL Server

You alse need to have microsoft sql server(you can find how download it in internet).

## Installation guide 

If you want to have set up same as me, you can download Spark_Demo_Course from this repository and go directly to step "Last step".
For other people you need to go throw all installation guide.

### Java

First of all you need to install Java JDK 7<=version<=11. I really recommended to install Java JDK version 8 to not have problems with versions. For the moment
you can't download Java JDK from Oracle archive from Belarus. Try to use VPN or download it from external sources.

### Spark

Go to the page https://spark.apache.org/downloads.html. Select the latest stable release of Spark. Choose a package type: select a version that is pre-built for 
the latest version of Hadoop such as Pre-built for Hadoop 3.3. If you want to have same versions as me, choose Spark 3.1.3 and Hadoop 2.7. After you choose 
package-type, you will see under it Download Spark, click on link, you will be redirecting to the next page where you need to click on the link like below.

![image](https://user-images.githubusercontent.com/113685144/190641829-10321ad3-9351-4134-b0ea-02d30496330e.png)

### Hadoop

Go to the page https://hadoop.apache.org/release/2.7.0.html. Select version of Hadoop that you choose while installing Spark(if you remember, you choose pre-built 
version of Hadoop). Choose download like below.

![image](https://user-images.githubusercontent.com/113685144/190644409-62fc1044-7ce1-4ebf-8353-4f6a6806d615.png)

### Winutils.exe

Windows users need also install winutils.exe to work with Spark. Go to this page https://github.com/steveloughran/winutils. You can find winutils.exe in hadoop-'version'\bin.
If you don't find your version here, find it in another sources.

### Unpacking 

Now you need to unpack Hadoop, Java and Spark to make just folder. After you unpack packages you need to put winutils.exe into the hadoop-2.7.0\bin and 
spark-3.1.3-bin-hadoop2.7\bin. 

### Last step

This is the last step. Use win+r and write sysdm.cpl, click ok. Click on Additionally, Environment Variables. 
You need to create 3 variables in system variables(second window):
- JAVA_HOME=...\Java\jdk-8;
- HADOOP_HOME=...\hadoop-2.7.0; 
- SPARK_HOME=...\spark-3.1.3-bin-hadoop2.7.

Then in the system variables you need to add 3 paths into path(second window):
- %HADOOP_HOME%\bin; 
- %SPARK_HOME%\bin; 
- %JAVA_HOME%\bin.

## Environment ready

Now you can open new command prompt and type there 
```
pyspark
``` 
You need to see something like below.

![image](https://user-images.githubusercontent.com/113685144/190651364-0bc8e33d-63a6-449a-9c84-94dc9a1687f0.png)

