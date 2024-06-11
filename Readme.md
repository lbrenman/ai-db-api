# Amplify Integration DB API Project Notes

This readme describes an Amplify Integration project that implements APIs that expose PostgreSQL Database.

The [**Github repo**](https://github.com/lbrenman/ai-db-api) includes the following:

* This readme with instructions for setting up the PostgreSQL Database and how to make the API calls and what to expect for a response
* An export of the Amplify Integration project that you can import in your tenant
* An OAS 3.0 Document created in Stoplight
  * Note that this OAS Doc only defines a subset of the APIs. You will find more in the Amplify Integration project

> Note that this project was created in version Amplify Integration v1.6.1. Major changes in how to use Amplify Integration to create API's using OAS Docs are planned for Amplify Integration v1.8

> Note that the methods exposed by the OAS Doc have a Access-Control-Allow-Origin header set to * to overcome CORS errors whrn accessing from a web page but currently only GETs are working from a web page. All are working form CURL, etc....

A demo video of the APIs in action can be found below:

[![](https://markdown-videos-api.jorgenkh.no/youtube/GzLiWOQ9wN4)](https://youtu.be/GzLiWOQ9wN4)

A demo video of how the APIs were built can be found below:

[![](https://markdown-videos-api.jorgenkh.no/youtube/vCWyU-EWiRU)](https://youtu.be/vCWyU-EWiRU)

## Database Setup

In this section we'll set up two tables, `employees` and `titles` that can be joined

* Note that I used [**Neon Postgres**](https://neon.tech/) for the Database
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

  ![Image](https://i.imgur.com/9TmjIX6.png)

* Drop Table (for testing error responses)

  ```SQL
  DROP table employees
  ```

## Amplify Integration Project Setup

* Import the API2DB.zip file into your Amplify Integration tenant.
![Image](https://i.imgur.com/1KOt6CR.png)
* Update the Connections and Plugs
* Enable the Integrations and get your HTTP/S Server URL
* Update the OAS Doc with your HTTP/S Server URL
* Test your API's

## Calling API's

* Get All - `API_GetAllEmployees`

  `GET /employee`

  Response:

  200
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

  503
  The server is not ready to handle the request - DB Error

* Get By Id - `API_GetEmployeeById`

  `GET /employee/:id`

  Response:

  200
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

  503
  The server is not ready to handle the request - DB Error

* Create - `API_CreateEmployee`

`POST /employee`

```JSON
{
    "Title": 2,
    "Name": "Jane Doe",
    "Email": "jdoe@outlook.com"
}
```

Response

204
<empty>

503
The server is not ready to handle the request - DB Error

* Create Bulk - `API_CreateEmployees_Bulk`

`POST /employeesbulk`

```JSON
[
    {
        "Name": "Ossie Farrell",
        "Email": "ofarrell0@apache.org",
        "Title": 1
    },
    {
        "Name": "Sam Myhill",
        "Email": "smyhill1@examiner.com",
        "Title": 2
    },
    .
    .
    .
    {
        "Name": "Seymour Pyke",
        "Email": "spyke4@buzzfeed.com",
        "Title": 3
    }
]
```

Response

204
<empty>

503
The server is not ready to handle the request - DB Error

* Delete by Id - `API_DeleteEmployeeById` and `API_CustomQueryDeleteEmployeeById`

`DELETE /employee/:id`

Response

204
<empty>

404
No record found

503
The server is not ready to handle the request - DB Error

* Put by Id - `API_PUTEmployee`

`PUT /employee/:id`

```JSON
{
    "Title": 2,
    "Name": "Jane Doe",
    "Email": "jdoe@outlook.com"
}
```

Response

204
<empty>

404
No record found

503
The server is not ready to handle the request - DB Error