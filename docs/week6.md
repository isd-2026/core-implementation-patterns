# Database & E2E Testing

## Database

Databases are used to permanently store data, which is often required in a full-stack web application. Databases typically run on their own servers, separate from the backend application. Communication between the backend and the database server is handled through a database driver or client library using a database-specific protocol (unlike HTTP requests). For commercial use, there are hosted (managed) database services such as Azure SQL and many more.

### Relational Database

A relational database stores data in tables that are related to each other. Each row represents an entity (or record), and each column represents an attribute defined by the table’s schema. The schema describes the structure of the table, including the attributes, data types, and constraints, and is often documented in a data dictionary. Relationships between tables can be expressed using an Entity–Relationship Diagram (ERD).

Non-relational (NoSQL) databases do not require a strict, fixed schema and often store data in formats such as JSON documents or graphs. NoSQL databases are out of scope for ISD.

### [SQLite](https://sqlite.org/)

SQLite is a lightweight SQL database that does not require a dedicated database server. It stores the entire database in a single file, which makes it suitable for the development stage of software. The query syntax is mostly the same as that of other SQL databases, with small differences in specific areas. For example, in traditional SQL databases (e.g., MySQL, PostgreSQL), defining a column as VARCHAR(50) enforces a maximum length of 50 characters; while SQLite ignores the length in types like VARCHAR(50), treating it as TEXT without enforcing a maximum length.

As the database is file-based, the same file can be shared within a group, allowing each student to work on different features while updating the same database during testing.

The Python module `sqlite3` provides methods to connect to a SQLite database, create tables, and perform CRUD (create, read, update, delete) operations. 

> Connect to a database with the following core syntax (defined as a function `get_connection()`). You'll need to specify the path and name of your db file.

```
def get_connection():
    conn = sqlite3.connect(DB_NAME)
    conn.row_factory = sqlite3.Row
    return conn
```

`conn.row_factory = sqlite3.Row` affects the result representation. By default the rows are returned as tuples, e.g., `("Alice Y", "a.y@example.com")`; with `sqlite3.Row` we return dictionary-like objects and we can access the fields with `u["name"]`, `u["email"]` where `u` is a `sqlite3.Row` object. The returning data format affects how you implement the backend.

The returned `conn` represents a connection session to the db, during which you can create cursors (objects that enable interactions with the database), execute queries, the session lasts until you close it with `conn.close()`.

> Creating a table 

```
conn = get_connection()
cursor = conn.cursor()
cursor.execute("""
    CREATE TABLE IF NOT EXISTS users (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        name TEXT NOT NULL,
        email TEXT NOT NULL UNIQUE,
        password TEXT NOT NULL,
        gender TEXT,
        favcol TEXT
    )
""")
conn.commit()
```

`cursor = conn.cursor()` creates a cursor for this connection session. `conn.commit()` applies all changes made during a transaction (actually modifies the db), with a transaction representing a sequence of one or more database operations (or database-modifying SQL statement, e.g., `INSERT`, `UPDATE`, `DELETE`). 

Relating it back to what we briefly covered in Week 2-database query protocols are stateful. When you connect to a database, the connection maintains the state. This state can include current transaction state (started, uncommitted changes), temporary tables, prepared statements, and cursor position in a query result. 

When a transaction is started (automatically when the first database-modifying SQL statement is executed), it temporarily holds the intermediate state of the database until it is finalised (or committed, e.g., with `conn.commit()`). If you call `conn.close()` without committing, SQLite will automatically roll back uncommitted changes (stored in memory), and no actual modifications are made on your db. 

> Think: How would you design your tests when testing databases? Choices to make between temporily or permanently changing the database state. 

## Database Design

### Entity-Relationship-Diagram (ERD)
To enable the above, we now need to extend our data model for it to capture project and allocation. You can refer to the [Example ERD](https://studentutsedu-my.sharepoint.com/:f:/g/personal/yining_hu_uts_edu_au/IgASfeLMaVrsRbJqYaubMP1NAe0HFMJDZ-jYJL4U1IPzCTU?e=Ud5XlC).

Note that there's a one-to-many relationship between the project and students, meaning that one project can have multiple students allocated. This complicates the database design as we want to avoid storing multiple values in one cell. Hence we create a new table to store allocations. Allocations, which are uniquely identified pairs combining a student and a project. 

You need to create corresponding tables for the two new entities. For simplicity we are not storing the admin details in the database, but only on the backend hard-coded in `models/admin.py`. 

<!-- TODO: refine -->
### Data Flow Diagram (DFD)
[Example DFD](https://miro.com/app/board/uXjVGCAiuyA=/?share_link_id=10398528666)

## Extensions
- [Normal Forms](https://www.geeksforgeeks.org/dbms/normal-forms-in-dbms/) in database design.