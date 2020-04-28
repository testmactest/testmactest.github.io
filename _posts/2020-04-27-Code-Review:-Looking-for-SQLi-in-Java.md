---
title: Code Review: Searching for SQLi vulnerabilities in Java
author: WHmacmac
layout: post
permalink: Code_Review_Searching_for_SQLi_vulnerabilities_in_Java
category: blog
---

Why do we need code review when we can interact with the target and finding if it is vulnerable? After all the response's behavior should help us finding if something is not configured properly. Based on the attack, there are cases where we will not can deduce so easily if we are vulnerable or not. Making code review can help us finding what the interactive analysis misses.

## Contents
* [Introduction](#introduction)
* [Java Persistence API (JPA)](#jpa)
* [Hibernate](#hibernate)
* [PreparedStatement](#preparedstatement)
* [CallableStatement](#callablestatement)
* [MyBatis](#mybatis)
* [Conclusion](#conclusion)

## Introduction {#introduction}

There are a wide variety of SQLi techniques, attacks, vulnerabilities which can occur in different situations. Some of them include:
<ul>
<li>Obtaining hidden data.</li>
<li>Unsettle the application's logic.</li>
<li>Blind SQLi, where the results of a query you contorl, are not returned or displayed in the application's responses.</li>
<li>Examining the database.</li>
<li>Union attacks</li>
</ul>

SQLi vulnerabilities can in principle occur at any location within a query and in different query types. Most of us probably found SQLi within a WHERE clause of a SELECT query.<br/>
The most common other locations where a SQLi can occur are:

<ul>
<li>In SELECT statements, within the table or column name.</li>
<li>In SELECT statements, within the ORDER BY clause.</li>
<li>In INSERT statements, within the inserted values.</li>
<li>In UPDATE statements, within the updated values or the WHERE clause.</li>
</ul>

There are cases where the application takes the user's input and store it for a future use, instead of processing the user's input from a HTTP request and adding it into a SQL query.
This is usually done by storing the input into a database, no vulnerability arises at the point where the data is stored. The stored SQLi vulnerabilities are hard to be detected through interactive analysis.

Many are thinking that today frameworks can resolve all the SQLi injection vulnerabilities, but this is wrong. Many are using SQL query and connections with the databases in a wrong manner. In what will follow, I will present some examples of how to detect SQLi in Java code and how to correctly write it.
   
## Java Persistence API (JPA) {#jpa}
Java Persistence API (JPA), is an ORM solution that is a part of the Java EE framework. It helps manage relational data in applications that use Java SE and Java EE. It is a common misconception that JPA (Java Persistence API) is SQL Injection proof. JPA allows the use of native SQL and defines its own query language, named, JPQL (Java Persistence Query Language). 

### 1. Vulnerable usage of JPA
When looking for SQLi in code, I am looking to see and understand how the queries are build and check its escape filters.

{% highlight java %}
List sql_result = EntityManager.createNativeQuery("Select * from PcComponents where component = " + component).GetResultList()
List sql_result = entityManager.createQuery("Select age from Persons age order where age.id = " + age.id).getResultList();
int sql_result = entityManager.createNativeQuery("Delete from Persons where CNP = " + CNP).executeUpdate();

{% endhighlight %}

I consider that component, age.id & CNP are user input. As you can see in the above examples, they have not been validated or escaped as required. Therefore, it leaves the above queries vulnerable to SQLi attacks.

### 2. Secure usage of JPA

The below code examples use parameter binding to set data. The JDBC driver is escaping the input data before the query is executed; making sure that data is used just as data and not as code. In case the data is contaning malicious sql code, the code will be escaped approprietely by the JBDC driver because parameterized queries are used.

<b>Native SQL</b>

{% highlight java %}
Query sql_query = entityManager.createNativeQuery("Select * from PcComponents where component = ?", PcComponents.class);
List sql_result = sql_query.setParameter(1, "AMD Ryzen 3900X").getResultList();

{% endhighlight %}

Positional parameter in JPQL
{% highlight java %}
Query jpqlQuery = entityManager.createQuery("Select emp from Employees emp where emp.id = ?1");
List sql_result = jpqlQuery.setParameter(1, "XXXXXX3000XXXXXX").getResultList();

{% endhighlight %}

Named parameter in JPQL
{% highlight java %}
Query jpqlQuery = entityManager.createQuery("Select emp from Employees emp where emp.salary > :salary");
List sql_result = jpqlQuery.setParameter("salary", new Long(1000)).getResultList();

{% endhighlight %}

Named query in JPQL - Query named "Person" being "Select c from Persons c where c.CNP = :CNP"
{% highlight java %}
Query jpqlQuery = entityManager.createNamedQuery("Person");
List sql_result = jpqlQuery.setParameter("CNP", "1940519555555").getResultList();

{% endhighlight %}

Tip for devs: If your JPA provider processes all input arguments to handle injection attacks then you should be covered. Never use concatenation in your queries.
