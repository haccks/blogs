---
title: What is the difference between DBMS and RDBMS?
description: RDMBS stores data in tabular form and have relations between those tables while DBMS stores data in flat files or tables (without any relations between them)
date: 2023-04-17 11:00:00 +0530
categories: [database, dbms]
tags: [dbms, rdbms]     # TAG names should always be lowercase
render_with_liquid: false
---
## What is a database management system? 

Database management system (DBMS) is a software to manage large set of data on a computer hardware. RDBMS stands for relational database management system. You can say that a RDBMS is an subset/extension of a DBMS.

## What is the key difference between DBMS and RDBMS? 

DBMS stores data as tables (without any relations between tables) or flat files. Document databses ([Neo4j](https://neo4j.com/)) or graph databases ([MongoDB](https://www.mongodb.com/)) are examples of DBMS.   

On the other hand a RDBMS application store data in tabular form with relations between the tables. MySQL, PostgresSQL are examples of RDBMS. Tables in RDBMS can have a unique identifies (called primary key). Data values are stored in the tables and each of these values can be accecible through structured query language (SQL). 