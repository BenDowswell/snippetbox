# Snippetbox

A web application for creating and sharing short text snippets, built while following Alex Edwards' [Let's Go](https://lets-go.alexedwards.net/) course. Written in Go with MySQL for storage.

## Prerequisites

- [Go](https://go.dev/dl/) (see `go.mod` for the required version)
- [MySQL](https://dev.mysql.com/downloads/)

## Database setup

Connect to MySQL as an admin user and run the following.

### Create the database and table

```sql
CREATE DATABASE snippetbox CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

USE snippetbox;

CREATE TABLE snippets (
    id INTEGER NOT NULL PRIMARY KEY AUTO_INCREMENT,
    title VARCHAR(100) NOT NULL,
    content TEXT NOT NULL,
    created DATETIME NOT NULL,
    expires DATETIME NOT NULL
);

CREATE INDEX idx_snippets_created ON snippets(created);
```

### Add some placeholder snippets (optional)

```sql
INSERT INTO snippets (title, content, created, expires) VALUES (
    'An old silent pond',
    'An old silent pond...\nA frog jumps into the pond,\nsplash! Silence again.\n\n– Matsuo Bashō',
    UTC_TIMESTAMP(),
    DATE_ADD(UTC_TIMESTAMP(), INTERVAL 365 DAY)
);

INSERT INTO snippets (title, content, created, expires) VALUES (
    'Over the wintry forest',
    'Over the wintry\nforest, winds howl in rage\nwith no leaves to blow.\n\n– Natsume Soseki',
    UTC_TIMESTAMP(),
    DATE_ADD(UTC_TIMESTAMP(), INTERVAL 365 DAY)
);

INSERT INTO snippets (title, content, created, expires) VALUES (
    'First autumn morning',
    'First autumn morning\nthe mirror I stare into\nshows my father''s face.\n\n– Murakami Kijo',
    UTC_TIMESTAMP(),
    DATE_ADD(UTC_TIMESTAMP(), INTERVAL 7 DAY)
);
```

### Create a restricted database user

The web app shouldn't connect as `root`, so create a user with limited privileges:

```sql
CREATE USER 'web'@'localhost';
GRANT SELECT, INSERT, UPDATE, DELETE ON snippetbox.* TO 'web'@'localhost';
-- Swap 'pass' for a password of your own choosing.
ALTER USER 'web'@'localhost' IDENTIFIED BY 'pass';
```

## Running the app

```sh
go run ./cmd/web
```

The server starts on [http://localhost:4000](http://localhost:4000).

### Configuration

| Flag    | Default                                  | Description             |
| ------- | ---------------------------------------- | ----------------------- |
| `-addr` | `:4000`                                  | HTTP network address    |
| `-dsn`  | `web:<password>@/snippetbox?parseTime=true` | MySQL data source name |

If your `web` user's password differs from the default in `cmd/web/main.go`, pass your own DSN:

```sh
go run ./cmd/web -dsn "web:yourpassword@/snippetbox?parseTime=true"
```

## Routes

| Method | Path                   | Description            |
| ------ | ---------------------- | ---------------------- |
| GET    | `/`                    | Home page              |
| GET    | `/snippet/view/{id}`   | View a snippet         |
| GET    | `/snippet/create`      | Snippet creation form  |
| POST   | `/snippet/create`      | Create a new snippet   |
| GET    | `/static/`             | Static files           |

## Project structure

```
cmd/web/          Application entry point, handlers, routes, helpers
internal/models/  Database models and errors
```
