# Install

## Database

### Supported databases

OpenBudgeteer requires a connection to a database which can be established using various variables. Currently, the following database servers are supported:

| CONNECTION_PROVIDER | Database system                                      |
|---------------------|------------------------------------------------------|
| TEMPDB              | SQLite in a temp file                                |
| SQLITE              | SQLite. Use CONNECTION_DATABASE to specify file name |
| MYSQL               | Oracle MySQL                                         |
| MARIADB             | MariaDB (FOSS MySQL fork)                            |
| POSTGRES            | PostgreSQL (starting with Version 1.7)               |
| POSTGRESQL          | PostgreSQL (starting with Version 1.7)               |

Automated database initialization is only supported for MySQL, SQLite and MariaDB.

### Database variables

| Variable                 | Description                                           | Used for database provider            | Example                 |
|--------------------------|-------------------------------------------------------|---------------------------------------|-------------------------|
| CONNECTION_PROVIDER      | Type of database that should be used                  | All (mandatory)                       | MYSQL                   |
| CONNECTION_SERVER        | IP Address/FQDN of the database Server                | MySQL, MariaDB, PostgreSQL (optional) | 192.168.178.100         |
| CONNECTION_PORT          | Port to database Server                               | MySQL, MariaDB, PostgreSQL (optional) | 3306                    |
| CONNECTION_DATABASE      | Database name, for SQLite full path and database name | All (optional)                        | MyOpenBudgeteerDb       |
| CONNECTION_USER          | Database user                                         | MySQL, MariaDB, PostgreSQL (optional) | MyOpenBudgeteerUser     |
| CONNECTION_PASSWORD      | Database password                                     | MySQL, MariaDB, PostgreSQL (optional) | MyOpenBudgeteerPassword |
| CONNECTION_ROOT_PASSWORD | Root Password                                         | MySQL, MariaDB (optional)             | MyRootPassword          |

### Default values for variables

| Variable                 | SQLite                         | MySQL, MariaDB | PostgreSQL |
|--------------------------|--------------------------------|----------------|------------|
| CONNECTION_PROVIDER      |                                |                |            |
| CONNECTION_SERVER        | localhost                      | localhost      | localhost  |
| CONNECTION_PORT          |                                | 3306           | 5432       |
| CONNECTION_DATABASE      | /app/database/openbudgeteer.db | openbudgeteer  | postgres   |
| CONNECTION_USER          |                                | openbudgeteer  | postgres   |
| CONNECTION_PASSWORD      |                                |                |            |
| CONNECTION_ROOT_PASSWORD |                                |                |            |

### Additional comments

- `CONNECTION_PROVIDER` is case-insensitive, so you can use for example `mysql` or `MYSQL`
- Using MySQL, MariaDB or PostgreSQL parameter `CONNECTION_DATABASE` can have maximum lenght of 64 chars using below character sets:
  - `0-9`
  - `a-z`
  - `A-Z`
  - `$`, `_`
  - `-` (not for PostgreSQL)
- The usage of `CONNECTION_ROOT_PASSWORD` is optional in case user and database for MariaDB or MySQL are not existing and should be created by OpenBudgeteer.

### Database setup

For all supported database provider, once the database is running and accessible for OpenBudgeteer the required tables and initial data will be created on first startup. Database schema changes after an update will be also applied automatically. Please consider a database backup before updating OpenBudgeteer. 

#### Sqlite

Not much to do here. Set the path to the database file via `CONNECTION_DATABASE`, everything else will be done by OpenBudgeteer.

#### MariaDB / MySQL

There are two ways to setup MariaDB / MySQL database:

Use `CONNECTION_ROOT_PASSWORD` to let OpenBudgeteer create user and database. The variable is only required for the first startup and can be removed later.

Alternative approach is to create user and database on your own. Important is that OpenBudgeteer has access full write access to the database as well as rights for changing the schema (like creating or changing tables). 

An easy way to do that would be to use something like `phpmyadmin`. Create a new user and ensure that you activate both options

- Create database with same name and grant all privileges.
- Grant all privileges on wildcard name (username\_%).

#### PostgreSQL

Please consider the container-per database PostgreSQL pattern and let container init take care of the database creation, or create the role and database yourself. In this case, the database created by you must be empty, the role must exist, and should have CREATE permission for all objects in the public schema of the target database.

## Docker

You can use the pre-built Docker image from [Docker Hub](https://hub.docker.com/r/axelander/openbudgeteer).

### docker run

To start a container use a `docker run` command like below. Please note that user and database need to be available, otherwise the container will not work. See [Database](#database) for more details.

```bash
docker run -d --name='openbudgeteer' \
    -e 'CONNECTION_PROVIDER'='MYSQL' \
    -e 'CONNECTION_SERVER'='192.168.178.100' \
    -e 'CONNECTION_PORT'='3306' \
    -e 'CONNECTION_DATABASE'='MyOpenBudgeteerDb' \
    -e 'CONNECTION_USER'='MyOpenBudgeteerUser' \
    -e 'CONNECTION_PASSWORD'='MyOpenBudgeteerPassword' \
    -e 'CONNECTION_MYSQL_ROOT_PASSWORD'='MyRootPassword' \
    -p '6100:80/tcp' \
    'axelander/openbudgeteer:latest'
```

Alternatively you can use a local `Sqlite` database using the below settings:

```bash
docker run -d --name='openbudgeteer' \
    -e 'CONNECTION_PROVIDER'='SQLITE' \
    -e 'CONNECTION_DATABASE'='/srv/openbudgeteer.db' \
    -v '/my/local/path:/srv'  \
    -p '6100:80/tcp' \
    'axelander/openbudgeteer:latest'
```

If you don't change the Port Mapping you can access the App with Port `80`. Otherwise like above example it can be accessed with Port `6100`

### docker compose (recommended) 

Below an example how to deploy OpenBudgeteer together with MySql Server and phpMyAdmin for administration.  Please note that user and database need to be available, otherwise the container will not work. See [Database](#database) for more details.

```yml
version: "3"

networks:
  app-global:
    external: true
  db-internal:


services:
  openbudgeteer:
    image: axelander/openbudgeteer
    container_name: openbudgeteer
    ports:
      - 8081:80
    environment:
      - CONNECTION_PROVIDER=MYSQL
      - CONNECTION_SERVER=openbudgeteer-mysql
      - CONNECTION_PORT=3306
      - CONNECTION_DATABASE=openbudgeteer
      - CONNECTION_USER=openbudgeteer
      - CONNECTION_PASSWORD=openbudgeteer
      - APPSETTINGS_CULTURE=en-US
      - APPSETTINGS_THEME=solar
    depends_on:
      - mysql
    networks:
      - app-global
      - db-internal

  mysql:
    image: mysql
    container_name: openbudgeteer-mysql
    environment:
      MYSQL_ROOT_PASSWORD: myRootPassword
    volumes:
      - data:/var/lib/mysql
    networks:
      - db-internal

  phpmyadmin:
    image: phpmyadmin/phpmyadmin
    container_name: openbudgeteer-phpmyadmin
    links:
      - mysql:db
    ports:
      - 8080:80
    networks:
      - app-global
      - db-internal

volumes:
  data:
```

Below another example how to deploy OpenBudgeteer together with PostgreSQL Server.
Please note that role and database `openbudgeteer` will be created with full authority on the `db` container on the first initialization of the database.

```yml
version: "3"

networks:
  app-global:
    external: true
  db-internal:


services:
  openbudgeteer:
    image: axelander/openbudgeteer
    container_name: openbudgeteer
    ports:
      - 8081:80
    environment:
      - CONNECTION_PROVIDER=postgres
      - CONNECTION_SERVER=openbudgeteer-db
      - CONNECTION_DATABASE=openbudgeteer
      - CONNECTION_USER=openbudgeteer
      - CONNECTION_PASSWORD=My$uP3rS3creTanDstr0ngP4ssw0rD!!!
      - APPSETTINGS_CULTURE=en-US
      - APPSETTINGS_THEME=solar
    depends_on:
      - db
    networks:
      - app-global
      - db-internal

  db:
    image: postgres:alpine
    container_name: openbudgeteer-db
    environment:
      - POSTGRES_USER=openbudgeteer
      - POSTGRES_PASSWORD=My$uP3rS3creTanDstr0ngP4ssw0rD!!!
      - POSTGRES_DB=openbudgeteer
    volumes:
      - data:/var/lib/postgresql/data
    networks:
      - db-internal

volumes:
  data:
```

### Docker tags

Beside the default `latest` tag there is also a `pre-release` tag available which includes the latest developments. Please note that `pre-release` can contain bugs on web frontend and also on database side.

Like for every update it is recommended to make a backup of the database before pulling new docker images.

In case you want to stick to a specific version there are also tags for each release available, like `1.4`, `1.5`, `1.5.1` etc.

## Build and deploy

If you don't want to use Docker you can also build the project on your own and deploy it on a web server like nginx.

Install .NET SDK 7 for your respective Linux distribution. See [here](https://docs.microsoft.com/en-us/dotnet/core/install/linux) for more details. Below example is for Debian 11

```bash
wget https://packages.microsoft.com/config/debian/11/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
sudo dpkg -i packages-microsoft-prod.deb
rm packages-microsoft-prod.deb

sudo apt-get update; \
  sudo apt-get install -y apt-transport-https && \
  sudo apt-get update && \
  sudo apt-get install -y dotnet-sdk-7.0 
```

Install nginx

```bash
sudo apt install nginx

sudo systemctl start nginx 
```

Clone git Repository and Build project

```bash
git clone https://github.com/TheAxelander/OpenBudgeteer.git
cd OpenBudgeteer/OpenBudgeteer.Blazor

dotnet publish -c Release --self-contained -r linux-x64
```

Modify `appsettings.json` and enter credentials for a running database server, or use sqlite

```bash
cd bin/Release/net7.0/linux-x64/publish

nano appsettings.json
```

For MySQL:

```json
{
  "CONNECTION_PROVIDER": "mysql",
  "CONNECTION_DATABASE": "openbudgeteer",
  "CONNECTION_SERVER": "192.168.178.100",
  "CONNECTION_PORT": "3306",
  "CONNECTION_USER": "openbudgeteer",
  "CONNECTION_PASSWORD": "openbudgeteer",
  "CONNECTION_ROOT_PASSWORD": "myRootPassword",
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning",
      "Microsoft.Hosting.Lifetime": "Information"
    }
  },
  "AllowedHosts": "*"
}
```

For Postgres:

```json
{
  "CONNECTION_PROVIDER": "postgresql",
  "CONNECTION_DATABASE": "openbudgeteer",
  "CONNECTION_SERVER": "192.168.178.100",
  "CONNECTION_USER": "openbudgeteer",
  "CONNECTION_PASSWORD": "openbudgeteer",
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning",
      "Microsoft.Hosting.Lifetime": "Information"
    }
  },
  "AllowedHosts": "*"
}
```

For Sqlite:

```json
{
  "CONNECTION_PROVIDER": "sqlite", 
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning",
      "Microsoft.Hosting.Lifetime": "Information"
    }
  },
  "AllowedHosts": "*"
}
```

Start server running on port 5000

```bash
./OpenBudgeteer --urls http://0.0.0.0:5000
```