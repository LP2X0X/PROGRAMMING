- Object Relational Mapping (ORM) is a technique used in creating a "bridge" between object-oriented programs and, in most cases, relational databases.
- When interacting with a database using OOP languages, you'll have to perform different operations like creating, reading, updating, and deleting (CRUD) data from a database. By design, you use SQL for performing these operations in relational databases.
- While using SQL for this purpose isn't necessarily a bad idea, the ORM and ORM tools help simplify the interaction between relational databases and different OOP languages. 
- An ORM tool is software designed to help OOP developers interact with relational databases. So instead of creating your own ORM software from scratch, you can make use of these tools.

---
### Popular ORM Tools for .NET

1. Entity Framework: Entity Framework is a multi-database object-database mapper. It supports SQL, SQLite, MySQL, PostgreSQL, and Azure Cosmos DB.
2. NHibernate: NHibernate is an open source object relational mapper with tons of plugins and tools to make development easier and faster.
3. [[Dapper]]: Dapper is a micro-ORM. It is mainly used to map queries to objects. This tool doesn't do most of the things an ORM tool would do like SQL generation, caching results, lazy loading, and so on.
4. Base One Foundation Component Library (BFC): BFC is a framework for creating networked database applications with Visual Studio and DBMS software from Microsoft, Oracle, IBM, Sybase, and MySQL

You can see more ORM tools [here](https://en.wikipedia.org/wiki/List_of_object%E2%80%93relational_mapping_software).

---
### Advantages of Using ORM Tools
- It speeds up development time for teams.
- Decreases the cost of development.
- Handles the logic required to interact with databases.
- Improves security. ORM tools are built to eliminate the possibility of SQL injection attacks.
- You write less code when using ORM tools than with SQL. 

### Disadvantages of Using ORM Tools
- Learning how to use ORM tools can be time consuming.
- They are likely not going to perform better when very complex queries are involved.
- ORMs are generally slower than using SQL.