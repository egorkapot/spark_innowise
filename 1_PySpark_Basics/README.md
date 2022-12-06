<h1 align="center">PySpark Basics</h1>


## Description

In this task you will create local SparkSession and work with main functionality of PySpark.
All useful information you can find in "Useful articles". Tasks will be in in PySpark_Basics notebook and answers in Answer_PySpark_Basics notebook.

## Additional Requirements

- Jupyter Notebook
- download and unpack this dataset https://www.kaggle.com/datasets/ruchi798/data-science-job-salaries?resource=download, put in folder PySpark_Basics. You can find all info about it in kaggle.

## Useful articles (must read)

Note 1: to display in your code the time that a cell is running, write at the beginning of cell %%time(in the first row)
Note 2: to use PySpark from Jupyter Notebook, add 
```
import findspark
findspark.init()
```
in your code.

- transformations vs actions (https://www.bigdataschool.ru/blog/transformations-and-actions-in-spark.html?ysclid=l84yx1b0bf998273623)
- sparksession (https://sparkbyexamples.com/pyspark/pyspark-what-is-sparksession/)
- local[*] (https://stackoverflow.com/questions/32356143/what-does-setmaster-local-mean-in-spark)
- read and write (https://www.linkedin.com/posts/activity-6972796904187260928-xSLs?utm_source=share&utm_medium=member_desktop)
- read csv (https://sparkbyexamples.com/pyspark/pyspark-read-csv-file-into-dataframe/)
- print Schema (https://sparkbyexamples.com/pyspark/pyspark-find-datatype-column-names-of-dataframe/)
- creating schema (https://sparkbyexamples.com/pyspark/pyspark-structtype-and-structfield/)
- difference between inferschema and pre-defined schema (https://www.learntospark.com/2020/10/spark-optimization-technique-inferschema.html)
- withColumn (https://sparkbyexamples.com/pyspark/pyspark-withcolumn/)
- select (https://sparkbyexamples.com/pyspark/select-columns-from-pyspark-dataframe/)
- distinct, dropDuplicates (https://sparkbyexamples.com/pyspark/pyspark-distinct-to-drop-duplicates/)
- group by (https://sparkbyexamples.com/pyspark/pyspark-groupby-explained-with-example/)
- window (https://sparkbyexamples.com/pyspark/pyspark-window-functions/)
- filter (https://sparkbyexamples.com/pyspark/pyspark-where-filter/)
- when (https://sparkbyexamples.com/pyspark/pyspark-when-otherwise/)
- join (https://sparkbyexamples.com/pyspark/pyspark-join-explained-with-examples/)
- join on multiple columns (https://stackoverflow.com/questions/33745964/how-to-join-on-multiple-columns-in-pyspark)
- cast (https://sparkbyexamples.com/pyspark/pyspark-cast-column-type/...)
- write csv (https://sparkbyexamples.com/pyspark/pyspark-write-dataframe-to-csv-file/)