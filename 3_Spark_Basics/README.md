<h1 align="center">Spark Basics</h1>


## Description

In this lesson we will describe how Spark work. We will build our own distributed system.

## Pre-requirements

Spark, Hadoop, Java JDK.

## Теория о кластере

Для того чтобы можно было приступить к оптимизациям и устройству самого Spark внутри, необходимо для начала ознакомиться с 
концепциями распределенной вычислительной системы, которой Spark и является.
По сути в устройстве системы лежит 6 понятий:

- Executor
- Worker
- Driver
- Master
- Cluster Manager
- Cluster 

Вот пример всей системы на картинке ниже.

<p align="center">
<img src="https://miro.medium.com/max/3334/1*9rdJjMwXXaBXDddxRLPFlw.jpeg" width="80%"></p>

Обо всём по порядку.

### Cluster

Кластером называют все вычислительные средства(будь то сервера или просто отдельно стоящие компьютеры), входящие в систему. 
A cluster is a set of tightly or loosely coupled computers connected through LAN (Local Area Network). The computers in the cluster are usually called nodes. 
Each node in the cluster can have a separate hardware and Operating System or can share the same among them. Resource (Node) management and task execution in the nodes 
is controlled by a software called Cluster Manager.

### Worker

Компьютер (сервер) ресурсы которого будут использоваться для работы Spark. Например в том же databricks при создании кластера вы видете выбор worker-node и после выбора
можно установить их количество. По сути машины объединенные по сети для совместной работы.

### Executor

Executor это ничто иное как процесс ранящийся в JVM(Java Virtual Machine). На каждом worker может быть несколько executors, о том как выбрать количество будет рассмотренно ниже.
Сам worker как вы понимаете ничего не делает, делают всю работу executors которые испольуют ресурсы worker. P.s. По сути worker железяка с ПО, а executor процесс выполняющий работу 
используя ресурсы железяки.

### Master

Машина с которой отправляется код на выполнение. По сути место где вы пишите код, ничего более.

### Driver

Driver это тоже процесс, который в зависимости от типа запуска кода может быть как на машине Master(client mode), так и на одной из worker-node(cluster mode).
Очевидно что продакшен cluster mode, ибо driver постоянно общается с executors, и если вы сидите в Минске а сервер в Москве то задержка будет больше чем время выполнения.

В driver живет main() метод, который и создаёт spark-context(Spark-session начиная с dataframe API), который и является логическим центром Spark. В его обязанности входит:

- Нарезает код на  job, stage, task(о том что это чуть ниже) 
- Создает Logical Plan, Physical Plan и т.д.(тоже чуть ниже объяснение)
- Координирует с cluster manager для отправки задач на executors для их выполнения 
- Отслеживает прогресс выполнения(что, где и на каком этапе ранится)

### Cluster manager

Существует несколько видов Cluster Manager:

- Spark Standalone Cluster Manager. Самый обычный и поставляется с самим спарком. Не требует настройки))
- Apache Mesos. Чуть сложнее и круче, но если предыдущий используется в тестовых целях, то этот я вообще не видел чтобы юзался.
- Hadoop YARN. До прихода на рынок k8s был одним из лучших решений, однако из-за особенностей устройства сейчас юзается реже, например в облачных решениях(тот же AWS EMR).
- k8s. Самый лучший и чаще всего будет он. Об устройстве k8s можно почитать в интернете, я лишь скажу что в рамках спарка один executor=один pod. Это добавляет дополнительную гибкость.

Вообще Cluster Manager отвечает за выделяемые ресурсы. Если мы выполняем задачу в cluster режиме, то сначала это ПО(Помните cluster manager это ПО) поднимает driver, driver говорит
мол надо столько executors с такими-то ресурсами и кидает запрос на cluster manager, тот же идёт и поднимает всё что нужно. В Client режиме driver поднимается сам,
но вот executors всё ещё лежат на плечах cluster manager.

Вот красивый пример как это всё работает:

Working Process

spark-submit –master <Spark master URL> –executor-memory 2g –executor-cores 4 WordCount-assembly-1.0.jar

 
1) Let’s say a user submits a job using “spark-submit”.
2) “spark-submit” will in-turn launch the Driver which will execute the main() method of our code.
3) Driver contacts the cluster manager and requests for resources to launch the Executors.
4) The cluster manager launches the Executors on behalf of the Driver.
5) Once the Executors are launched, they establish a direct connection with the Driver.
6) he driver determines the total number of Tasks by checking the Lineage.
7) The driver creates the Logical and Physical Plan.
8) Once the Physical Plan is generated, Spark allocates the Tasks to the Executors.
9) Task runs on Executor and each Task upon completion returns the result to the Driver.
10) Finally, when all Task is completed, the main() method running in the Driver exits, i.e. main() method invokes sparkContext.stop().
11) Finally, Spark releases all the resources from the Cluster Manager.

## Практика кластер(ну и теория)

Сейчас мы создадим свой кластер на своей машине.

Заходим в cmd(если конечно вы настроили Spark как в моём гайде) и прям там пишем 
```
spark-class org.apache.spark.deploy.master.Master
```
Это ничто иное как объявление master для вашего кластера. Вообще Spark предоставляет широкий спектр скриптов для автоматического подъема всего добра, но к сожалению они не работают
на Windows(ток Линуха). Поэтому мы будем делать всё ручками.
После выполнения кода выше, должен быть такой вывод

![image](https://user-images.githubusercontent.com/113685144/192796311-57f796a6-c35e-4aac-9ee3-d9467c5a2da0.png)

Теперь вы можете взять MasterUI и зайти на эту страницу в интернете. Это ничто иное как UI где можно найти всё о вашем кластере, из чего он состоит, какую задачу выполняет и т.д.
Используется для мониторинга работы вашего кластера в real time. Сам же master поднят по адресу который указан в этой строке Starting Spark master at spark://...

Далее необходимо поднять например два worker. 
Откройте новое cmd окно и впишите туда следующее
```
spark-class org.apache.spark.deploy.worker.Worker spark://<адрес мастера> --cores 2 --memory 3g
```
Это создат worker на вашем компьютере с 2 ядрами и 3гб оперативки. Spark считает логические ядра, то есть например у меня 6 ядер по 2 потока, то есть для спарка это 12 ядер. 
Чтобы проверить что он создался, зайдите в UI и там будет Workers(1). Note: не меняйте сеть когда создаёте все это, ибо очевидно что адреса будут меняться.
Создадим ещё одного Worker но уже с 3 ядрами и 4g памяти.
```
spark-class org.apache.spark.deploy.worker.Worker spark://<адрес мастера> --cores 3 --memory 4g
```
В spark UI должен появиться второй Worker.
Теперь ваш кластер готов к боевым действиям.

## Немного(очень много) теории про job, stage, task, оптимизатор, таблицы и виды оптимизаций в оптимизаторе

Прежде чем приступить к запуску кода, необходимо для начала разобраться как всё работает на этом кластере.
В этих статьях вы познакомитесь в вышеупомянутыми job, stage, task.
Статьи(первая вводная, вторая более серьезная которая покрывает даже больше аспектов, но всё же советую сначала первая(ибо там есть job и application), а потом вторая):

- https://www.hadoopinrealworld.com/what-are-applications-jobs-stages-and-tasks-in-spark/ 
- https://habr.com/ru/company/neoflex/blog/578654/?ysclid=l8ls6p9kjq379024568

Т.к. во второй статье затронулся оптимизатор, то почему бы с ним не разобраться до конца.
Статья про оптимизатор в Spark: https://spark-school.ru/blogs/how-catalyst-works/?ysclid=l8lsfmfe4b334214965.

Уже не раз упоминались логические оптимизации, которые применяет оптимизатор когда из логического плана получает оптимизированный логический.
Все эти оптимизации называются rule-based optimizations. Что же за они пришло время узнать:

- Predicate pushdown — строки. Иными словами, если вы напишите код, в котором первой строкой считаете данные а потом где-то в конце отфильтруете по ключу(например только True 
флаг), то Spark сделает этот отбор как можно ближе к считываемому файлу(если конечно это возможно) чтобы уменьшить как можно раньше количество строк. Плюс ко всему,
этот отбор строк может вообще происходить на этапе считывания из файла(то есть прям когда он считывает данные, строки будут уже отсеиваться).
- Projection Pushdown — колонки. То же самое, только с полями. Например вспомним тот же parquet позволяющий считывать только те колонки которые мы используем.
- Partition pruning - вообще, эта штука только когда Spark использует Hive metastore db, где хранит всякую мета-информацию о таблицах. Если речь про databricks то там 
Spark использует эту бд, а вот если речь про обычный локальный Spark то он юзает дефолтную бд(если хочешь Hive metastore нужно настроить это дополнительно). Эта штука используется
только на таблицах используя sparksql, то есть к обычным датафреймам не применима(ну или я чего-то не знаю). 
Вот статья где в начале есть объяснение сути partitoning pruning: http://www.openkb.info/2021/03/spark-tuning-dynamic-partition-pruning.html.
Если вкратце, то суть в том что когда Join по условию, то можно сначала не джойнить обе таблицы, а сделать подзапрос который отсортирует одну таблицу, потом результат значений
распространить на все executors и там отфильтровать вторую таблицу, и только потом уже делать джойн из отфильтрованных таблиц. Работает круто и всё такое, только вот сразу же и 
всплывает мысль почему это ток с таблицами: а потому что как такое написать в синтаксисе dataframe API? Никак, сначала ты фильтруешь таблицу одну, потом джойнишь обе. 
Таблицами в данном случае выступают реально таблицы, сейчас будет длинная и на самом деле тяжелая часть, так что будьте внимательны.
Таблицы в Spark, не такие как в БД, ибо они хранятся в файлах. Поэтому правила ACID над ними не работают, что ещё хуже, т.к. это просто файлы то с ними нельзя делать
update, delete, ну и самое главное merge. Да вы можете писать над ними sql запросы как в обычную БД как раз за счёт Hive metastore db или же за счёт дефолтного метастора Spark,
но к сожалению или к счастью это всё ещё файлы. И тут на сцену выходит преславутый deltalake с его delta table. По сути те же файлы, но уже гораздо круче и прикольнее, ибо
на них теперь распространяется ACID, да ещё и через Delta API можно делать delete, update, merge, ну и хранить историю(позже вы узнаете всё более подробно про delta lake). Так вот,
про таблицы ещё нужно знать что они бывают managed и external. Суть в том что в Spark ещё есть Spark warehouse(Hive warehouse). Это такое хранилище, где Spark хранит managed таблицы.

Разница в следующем:

- managed таблица - это таблица которая полностью управляется спарком, а именно спарк хранит не только мета-информацию в Hive metastore, но ещё и сами файлы этой таблицы
в Spark warehouse. Удалите таблицу и удалите не только мета-информацию, но ещё и сами данные. Создаётся так: 

spark.sql("CREATE TABLE employee (name STRING, emp_id INT,salary INT, joining_date STRING)")

или так 

df= spark.read.format("csv").option("inferSchema","true").load("/FileStore/tables/Order.csv")

df.write.saveAsTable("OrderTable").

- external таблица - это таблица, данные которой(файлы) хранятся вне Spark warehouse, Spark лишь знает что она хранится там-то и знает о ней мета-информацию. Удалите такую таблицу,
и спарк лишь удалит мета-информацию, а сама таблица будет жить. Создаётся так: 

spark.sql("""CREATE TABLE OrderTable(name STRING, address STRING, salary INT) USING csv OPTIONS (PATH '/FileStore/tables/Order.csv')""").

В основном таблицы юзаются только с databricks где есть под капотом Hive и своё hdfs хранилище, ну а какую именно таблицу юзать это уже прихоть архитектора на проекте.

К сожалению это не вся теория. Далее нас ждут оптимизации которые оптимизатор юзает выбирая физический план. Называются они cost-based optimizations
Ну тут на самом деле можно разойтись на славу так, но т.к. это лучше показывать вместе с практикой, приведу лишь пример: У вас есть join. 
В спарке есть 5 видов джойн(обязательно их затронем но не сегодня) и спарк в зависимости от настройек которые вы ему задали(spark.sql.autobroadcastjointhreshold 
например) посмотрит на статистику входных данных и примет решение какой именно из 5 джойнов ему юзать. Какие именно оптимизации спарк делает на этом 
этапе вы узнаете в следующих темах, тут лишь скажу ещё что существует такая вещь как AQE, которая основывается не на входных данных, а прям на рантайме смотрит на
статистику и решает что лучше. То есть изначально план может быть один, но например потом на рантайме когда уже вы отфильтровали и что-то преобразовали, статистика же может
измениться и старый план станет уже не актуален. Тут и приходит на помощь AQE который собирает статистику после каждого shuffle и проверяет надо ли поменять план выполнения.
Появилось это чудо со Spark 3.0 и активно юзается. Также на выбор физическего плана влияет CBO(cost-based optimizations), работает только на таблицы(тобишь на
sparksql), для dataframe или RDD вещь бесполезная очевидно, но для таблиц очень помогает. Его надо включить и самому руками написать в коде чтобы Spark посчитал статистику о таблицах, 
только тогда CBO заработает. Подробнее про AQE и CBO и какие именно оптимизации они делают вы узнаете чуть позже, когда узнаете в принципе что можно улучшать в спарке и зачем это вообще
нужно.


## DAG, narrow wide transformations

узкие, широки трансформации(да они уже встречались в одной статье, надо закрепить): https://sauravomar01.medium.com/wide-vs-narrow-dependencies-in-apache-spark-2cd33bf7ed7d

DAG - видели картинку из прошлой статьи RDD Lineage? Так вот это оно самое. Направленный ациклический граф, который показывает путь ваших RDD от начала до конца, а именно
какие операции с ними происходят и т.д. Статейка: https://www.tutorialkart.com/apache-spark/dag-and-physical-execution-plan/.

Ну и как читать план запроса: https://blog.rockthejvm.com/reading-query-plans/.
Есть ещё одна статья, как читать план запроса(оч советую, реально супер крутая но посложнее): https://towardsdatascience.com/mastering-query-plans-in-spark-3-0-f4c334663aa4.


## Запуск кода на кластере

Перед тем как посмотреть на ваш кластер в бою и посмотреть на всякие метрики в Spark UI, вот статья про то что вообще можно найти в Spark UI: 
https://spark.apache.org/docs/3.0.0-preview/web-ui.html#:~:text=Apache%20Spark%20provides%20a%20suite,Jobs%20detail.
Вот небольшое пояснение на cache и persist, которые встретятся в статье: https://stackoverflow.com/questions/26870537/what-is-the-difference-between-cache-and-persist. 
Также необходимо посмотреть как запускается spark application: https://sparkbyexamples.com/spark/spark-submit-command/.

Т.к. аналитика в Spark UI доступна только в риал лайф, то необходимо настроить папку для хранения истории этой аналитики, чтобы потом можно было смотреть. Для этого вам надо
создать в этой директории(3_Spark_Basics) папку for_history. Чтобы там хранилась истории и после вы могли смотреть всю статистику в History Server вам необходимо вписать эти строки:

```
spark.eventLog.enabled true
spark.eventLog.dir file:///C:/Users/stepa/Desktop/spark_demo/3_Spark_Basics/for_history
spark.history.fs.logDirectory file:///C:/Users/stepa/Desktop/spark_demo/3_Spark_Basics/for_history
```
со своим путём разумеется на папку. Вписать их необходимо в файл spark-defaults.conf который находится по этому пути ...\spark-3.1.3-bin-hadoop2.7\conf. Также необходимо
удалить .template из имени spark-defaults.conf.template(если вы не видите template то включите вид->расширения имён файлов).
Также необходимо запустить history server. Для этого открываем cmd и там пишем:

```
spark-class org.apache.spark.deploy.history.HistoryServer
```

Эта команда запустит сервер, который будет читать историю отработанных приложений, адрес сервера будет указан после выполнения команды(так же как и было с адресом мастера). 
Поэтому после завершения выполнения приложения необходимо идти по тому адресу который будет указан при запуске history server и смотреть на ваше приложение.

Для того чтобы Spark видел Python 100%, переименуйте файл spark-env.sh.template в spark-env.cmd и добавьте туда строки ниже
(файл лежит по этому пути ...\spark-3.1.3-bin-hadoop2.7\conf). Естественно пути надо прописать свои, они будут почти такие же.

```
set PYSPARK_PYTHON=C:\users\stepa\appdata\local\programs\python\python39\python.exe
set PYSPARK_DRIVER_PYTHON=C:\users\stepa\appdata\local\programs\python\python39\python.exe
```

Файл с кодом уже готов и называется spark_basics.py, в нём необходимо поменять пути для считывания df_people, df_country, df_parquet и для записи df_last, plans1.txt И plans.txt.
После его требуется запустить и посмотреть на различные метрики в Spark UI(если не успеете за время выполнения приложения, то бегом в history server), а также 
на планы запросов которые будут сохранены в отдельный файл с названием plans.txt или plans1.txt. Сравните план выполнения с кодом в spark_basics.py, 
сравните различные планы(логический от физического, или какие-нибудь ещё). Также обязательно зайдите в history server в раздел SQL(сверху где Jobs, Stages...)
и посмотрите всё в интерактивной форме. В статье которая посложнее про чтение планов достаточно хорошо объясняется как там всё читать.

После самостоятельного разбора, обязательно загляните в "Объяснение плана запросов.docx", чтобы посмотреть на нюансы, которые надо обязательно увидеть.

Note: да, физический план можно найти в Spark UI(history server) в вкладке SQL под дагом. Но там только физический, в txt будут все.
Note 2: history server это тот же spark ui, ток хранит то что уже прошло, а Spark UI то что в данный момент
