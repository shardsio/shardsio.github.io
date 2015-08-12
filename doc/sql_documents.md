---
layout: index
title: Evolution from SQL scripts to SQL Documents
---
## Evolution from SQL scripts to SQL Documents

Traditionally SQL used only for queriyng data and then the obtained results passed to futher programs for displaying results.
The idea described in this article aims to keep entire data visualization within a SQL context. 
It is highly inspired by the existing software from related programming languages and 
products like [IPyhton Notebooks](http://ipython.org/notebook.html) and [R Markdown](http://rmarkdown.rstudio.com).

### SQL Blocks

First of all lets introduce the term **SQL block**.
From client perspective a SQL statement is passed to the server for processing and server returns some dataset back to the client.
In some implementations like in Postgresql it's possible to pass to server several statements at once and get several datasets.
So **SQL block** is just a set of SQL statements passed to the server at once. 

In order to designate a set of statements withing a SQL script let's use a single comment line started with 3 dashes:

``` sql
--- here a SQL block starts

SELECT 1;

SELECT 2;

--- here another block starts

SELECT 3;

```

Most of the existing tools will pass such script on the server either entirely or statement by statement.
Nevertheless the main idea is not passing statements to the server by blocks but bringing extra semantics to the script.
Lets see how it can work further.

### SQL Annotations

Now we have blocks semantics within SQL scripts. Since these are just comments a database server and existing client tools will ignore them.
But we can add there something useful. For example we can tell to a client programm how to interpret the results of SQL block execution.

Thus in order to build a chart from query result we use **chart** annotation:

``` sql
--- chart pie
SELECT city, COUNT(*) FROM employees GROUP BY city
```

Here the query will return a datasaet with number of employees in each city

city	 | count
---------|---------
London	 | 245
Paris	 | 153
New York | 323

And the following chart will be displayed:

![](/images/pie_chart.png)


### Markdown comments

Another nice thing we can add is an arbitrary content added to SQL script output.
Thus we can add extra formatting options such as titles, output description and images.

For this purpose we use another special comments enclosed into /** **/

All the content within such comment should be formatted with [Markdown](http://daringfireball.net/projects/markdown/) syntax.

This type of comments in the beginning or in the end of each SQL block will be translated into formatted text in the output.


``` sql
/**
## Number of employees by cities
**/
--- chart pie
select city, COUNT(*) from employees
group by city

/**
This is an example of SQL document
**/
```

The complete output of the above SQL document is available [here](http://www.sqltabs.com/api/1.0/docs/6b28b79e0357cce98eb1f1403b1e7075)

### 


### Conclusion

By adding extra semantics to SQL scripts it's possible to generate pretty rich documents based on data from databases.
SQL annotations make possible to do quick visual analisys without the need to export data to 3d party tools.
Easy syntax allows to create reports and store them in source control systems for tracking history of changes.


A proof of concept implementation for Postgresql is available on [www.sqltabs.com](http://www.sqltabs.com).
In case you are a happy Postgresql user and have Ubuntu/Debian or Mac OS X client machine welcome to try it entirely for free.



