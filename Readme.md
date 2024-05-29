# Amplify Integration DB API Project Notes

This project demonstrates how to create powerful APIs that access Database data.

## Database Setup

In this section we'll setup two tables, `employees` and `titles` that can be joined

* I used [**Neon Postgres**](https://neon.tech/) as the Database

* Create Tables

  ```SQL
  CREATE TABLE employees (
    id              SERIAL PRIMARY KEY,
    name            VARCHAR(100) NOT NULL,
    email           VARCHAR(100) NOT NULL,
    title           INTEGER NOT NULL,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
  );
  ```

  ```SQL
  CREATE TABLE titles (
    id              SERIAL PRIMARY KEY,
    name            VARCHAR(100) NOT NULL
  );
  ```

* Create Function and Trigger if they don't exist

  ```SQL
  CREATE OR REPLACE FUNCTION trigger_set_timestamp()
    RETURNS TRIGGER AS $$
    BEGIN
      NEW.updated_at = NOW();
      RETURN NEW;
    END;
    $$ LANGUAGE plpgsql;
  ```

  ```SQL
  CREATE TRIGGER set_timestamp
  BEFORE UPDATE ON employees
  FOR EACH ROW
  EXECUTE PROCEDURE trigger_set_timestamp();
  ```

* Insert Rows

  ```SQL
  INSERT INTO employees (name, email, title)
  VALUES
    ('Harry Smith','hsmith@gmail.com',1),
    ('John Doe','jdoe@outlook.com',2),
    ('Patty French','pfrench@hotmail.com',3);
  ```

  ![](https://i.imgur.com/mkU31kK.png)

  ```SQL
  INSERT INTO titles (name)
  VALUES
    ('President'),
    ('CEO'),
    ('CFO'),
    ('CMO'),
    ('Solution Architect');
  ```

  ![](https://i.imgur.com/R3MaQiX.png)

* Modify Row

```SQL
UPDATE employees
SET
    name = 'Leor Brenman'
WHERE
    id = 1;
```

* Search By Date

  ```SQL
  SELECT * FROM employees WHERE created_at >= '2023-01-13 02:10:28.088649+00'
  ```

* Delete Row

```SQL
DELETE FROM employees
WHERE id = 4;
```

* Join Example

```SQL
SELECT employees.name AS employee, employees.email, titles.name AS title
FROM employees
INNER JOIN titles ON employees.title = titles.id;
```

![](https://i.imgur.com/9TmjIX6.png)

## API's

* Get All - `API_GetAllEmployees`

  `GET /employee`

  Response:

  ```JSON
  [
      {
          "Id": 1,
          "Name": "Harry Smith",
          "Email": "hsmith@gmail.com",
          "Title": 1,
          "Created_at": "2024-05-22 17:16:33.634848",
          "Updated_at": "2024-05-22 17:16:33.634848"
      },
      .
      .
      .
      {
          "Id": 3,
          "Name": "Patty French",
          "Email": "pfrench@hotmail.com",
          "Title": 3,
          "Created_at": "2024-05-22 17:16:33.634848",
          "Updated_at": "2024-05-22 17:16:33.634848"
      }
  ]
  ```

* Get By Id - `API_GetEmployeeById`

  `GET /employee/:id`

  Response:

  ```JSON
  [
      {
          "Id": 5,
          "Name": "John Doe",
          "Email": "jdoe@outlook.com",
          "Title": 2,
          "Created_at": "2024-05-23 16:00:50.678359",
          "Updated_at": "2024-05-23 16:00:50.678359"
      }
  ]
  ```

  * Create - `API_CreateEmployee`

  `POST /employee`

  ```JSON
  {
      "Title": 2,
      "Name": "Jane Doe",
      "Email": "jdoe@outlook.com"
  }
  ```
  