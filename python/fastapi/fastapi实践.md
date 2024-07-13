# fastapi实践

## 一、fastapi 连接 MySQL 数据库

### 前置准备

1、安装驱动包

首先需要为 MySQL 安装 Python 库，FastAPI 需要使用 Python 的 MySQL 客户端库来连接到 MySQL 数据库，这些驱动包括 `mysql-connector-python` 和 `pymysql`。

```bash
pip install mysql-connector-python pymysql
```

![img](assets/111.awebp)

2、创建库表

```sql
CREATE DATABASE example_db;

CREATE TABLE
  `users` (
    `id` int unsigned NOT NULL AUTO_INCREMENT,
    `name` varchar(255) DEFAULT NULL,
    `created_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (`id`)
  ) ENGINE = InnoDB;
  
 -- 往数据库表中插入一条数据：
 INSERT INTO users (name) VALUES ("Atom");
```

### 1.2 正式使用

#### 数据库连接

例如，通常命名一个包含依赖项函数的 Python 文件，该函数定义与上面示例中所示的 MySQL 数据库的连接，以指示它具有与数据库相关的功能，例如 `db.py` 或 `database.py`。

此外，如果您有多个定义 FastAPI 依赖项的函数，或者如果您为不同功能定义了依赖项，则可以通过为每个功能指定不同的名称来提高代码的可读性。

```python
from fastapi import Depends
import mysql.connector


def get_db_connection():
    connection = mysql.connector.connect(
        host='localhost',
        port=3306,
        user="root",
        password="123456",
        database="example_db"
    )
    return connection

def get_db():
    connection = get_db_connection()
    db = connection.cursor()

    try:
        yield db
    finally:
        db.close()
        connection.close()
```

#### python操作数据库

在将 MySQL 数据库与 FastAPI 路由器一起使用的示例 Python 文件名中，通常最好根据应用程序的功能和角色对其进行命名。 你可以想到这样的文件名：

- `main.py`：包含示例代码的文件，该示例是应用程序的主要入口点，定义 FastAPI 路由器并使用 MySQL 数据库。
- `router.py`：定义 FastAPI 路由器并包含使用 MySQL 数据库的示例的代码的文件。
- `db.py`：包含用于连接和查询 MySQL 数据库的函数的文件。

例如，可以考虑以下文件名和目录结构：

- `main.py`：作为应用程序主入口点的文件，导入并使用路由器目录中的路由器模块。
- `routers/db_router.py`：定义使用 MySQL 数据库的示例路由器的模块。
- `routers/db.py`：定义用于连接和查询MySQL数据库的函数的模块。

`db_router.py` 文件写入如下内容：

```python
from fastapi import FastAPI, Depends
from mysql.connector import cursor
from db import get_db
import json
from pydantic import BaseModel

app = FastAPI()


# def get_db(db: cursor.MySQLCursor = Depends(get_db)):
#     return db

@app.get("/users/")
async def get_users(db: cursor.MySQLCursor = Depends(get_db)):
    query = "SELECT * FROM users"
    db.execute(query)
    result = db.fetchall()
    if result:
        return {"users": result}
    else:
        return {"error": "User not found"}


@app.get("/users/{user_id}")
async def get_user(user_id: int,
                   db: cursor.MySQLCursor = Depends(get_db)):
    query = "SELECT * FROM users WHERE id = %s"
    db.execute(query, (user_id,))
    result = db.fetchall()
    if result:
        return {"user_id": result[0][0], "username": result[0][1]}
    else:
        return {"error": "User not found"}


@app.get("/user_name/{user_name}")
async def insert_user(user_name: str,
                      db: cursor.MySQLCursor = Depends(get_db)):
    query = "INSERT INTO users (name) VALUES (%s)"
    db.execute(query, (user_name,))
    result = db.fetchone()
    db.execute("COMMIT")
    return {"user_name": user_name}

class User(BaseModel):
    name: str

@app.post("/users/add")
async def create_item(user_name: str,
                      db: cursor.MySQLCursor = Depends(get_db)):
    query = "INSERT INTO users (name) VALUES (%s)"
    db.execute(query, (user_name,))
    result = db.fetchone()
    db.execute("COMMIT")
    return {"user_name": user_name}

@app.post("/users/addUser")
async def create_item(user: User,
                      db: cursor.MySQLCursor = Depends(get_db)):
    query = "INSERT INTO users (name) VALUES (%s)"
    db.execute(query, (user.name,))
    result = db.fetchone()
    db.execute("COMMIT")
    return {"user_name": user.name}
```

