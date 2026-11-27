# SQL Playground

> **Learn SQL visually. Build databases without writing queries.**

![Java](https://img.shields.io/badge/Java-17-orange)
![Spring MVC](https://img.shields.io/badge/Spring%20MVC-6.1-brightgreen)
![MySQL](https://img.shields.io/badge/MySQL-8-blue)
![Tomcat](https://img.shields.io/badge/Tomcat-10%2B-yellow)
![Maven](https://img.shields.io/badge/Build-Maven-red)

SQL Playground is a full-stack Java web application that lets beginners work with a real MySQL database **without typing any SQL**. You fill in forms and click buttons; the application builds the SQL in Java, shows it to you in a live preview panel, and runs it through JDBC. You perform real database operations and see the exact SQL behind each one.

Built for the *Advanced Java Programming (BCSE0651)* course at NIET, Greater Noida.

---

## Features

- **Create Table**: visual column builder (datatype, length, nullable, primary key, auto-increment, default) with a live `CREATE TABLE` preview
- **Insert / Update / Delete**: input fields are generated automatically from the table's columns
- **View Data**: search, column sorting, pagination, CSV and Excel-compatible export
- **Drop Table**: protected by a type-the-table-name confirmation
- **Join Tables**: pick two tables, a join type (INNER, LEFT, RIGHT) and the matching columns
- **Aggregate Query**: `COUNT`, `SUM`, `AVG`, `MIN`, `MAX` with optional `GROUP BY` and `HAVING`
- **Database Explorer**: browse tables, columns, datatypes, primary keys and row counts using `DatabaseMetaData`
- **Query History**: every generated statement is stored with its status and timestamp, searchable and filterable
- **Live SQL preview** on every screen, with a copy button
- Dashboard with live connection status, light and dark themes, toast notifications and responsive layout

## Tech Stack

| Layer | Technology |
|---|---|
| Language | Java 17 |
| Web framework | Spring MVC 6.1 (pure Java configuration, no XML) |
| Data access | Spring JDBC (`JdbcTemplate`), JDBC, HikariCP connection pool |
| Database | MySQL 8, MySQL Connector/J 8.4 |
| Views | JSP with JSTL (Jakarta) |
| Front end | HTML5, CSS3, vanilla JavaScript |
| Build / server | Maven (WAR), Apache Tomcat 10+ |

No Spring Boot, Hibernate, JPA or front-end frameworks are used, so the full request flow stays visible.

## Architecture

```
Browser (JSP + JavaScript)
        |
        v
Spring DispatcherServlet
        |
        v
Controller  ->  Service  ->  DAO  ->  JdbcTemplate  ->  JDBC / HikariCP  ->  MySQL
                   |
                   +-->  SQLGenerator (builds all SQL)   +   MetadataUtility (reads schema)
```

- **`SQLGenerator`** is the only class that builds SQL text. No SQL is written in controllers.
- **`MetadataUtility`** discovers tables, columns and keys at runtime, so the app works for any table you create.
- Every executed statement (success or failure) is logged to the `query_history` table.

## Screenshots

| Dashboard | Create Table |
|---|---|
| ![Dashboard](docs/screenshots/dashboard.png) | ![Create Table](docs/screenshots/create-table.png) |

| View Data | Join Tables |
|---|---|
| ![View Data](docs/screenshots/view-data.png) | ![Join Tables](docs/screenshots/join-tables.png) |

| Aggregate Query | Query History |
|---|---|
| ![Aggregate Query](docs/screenshots/aggregate-query.png) | ![Query History](docs/screenshots/query-history.png) |

## Getting Started

### Prerequisites

- JDK 17 or higher
- Apache Maven 3.9+
- MySQL 8.x (running)
- Apache Tomcat **10 or newer** (Tomcat 9 will not work because the project uses the `jakarta.*` namespace)
- IntelliJ IDEA (recommended)

### 1. Clone the repository

```bash
git clone https://github.com/<your-username>/SQLPlayground.git
cd SQLPlayground
```

### 2. Create the database

```bash
mysql -u root -p < sql/schema.sql
```

This creates the `sql_playground` database, the `query_history` and `users` tables, and a demo `student` table with a few rows.

### 3. Configure the connection

Edit `src/main/resources/db.properties`:

```properties
db.url=jdbc:mysql://localhost:3306/sql_playground?useSSL=false&serverTimezone=UTC&allowPublicKeyRetrieval=true
db.username=root
db.password=your_password
```

> Do not commit your real password to a public repository.

### 4. Run the application

**Option A: IntelliJ IDEA with the Smart Tomcat plugin**

1. Open the project and let Maven import the dependencies.
2. Create a *Smart Tomcat* run configuration pointing at your Tomcat installation.
3. Set the deployment directory to `src/main/webapp`, the context path to `/SQLPlayground`, and the module classpath to this project.
4. Click Run.

**Option B: Build a WAR and deploy it**

```bash
mvn clean package
cp target/SQLPlayground.war $CATALINA_HOME/webapps/
$CATALINA_HOME/bin/startup.sh        # startup.bat on Windows
```

Open **http://localhost:8080/SQLPlayground/**

## Quick Walkthrough

1. **Create Table**: make `enrollment` with `id` (INT, primary key, auto-increment) and `student_id` (INT).
2. **Insert Data**: add a few rows to `enrollment`, using ids from the demo `student` table.
3. **Join Tables**: join `student.id` to `enrollment.student_id` and check the generated `INNER JOIN`.
4. **Aggregate Query**: count enrollments grouped by `student_id` with a `HAVING` condition.
5. Open **Query History** to see every statement you generated.

## Project Structure

```
SQLPlayground/
├── pom.xml
├── sql/schema.sql
└── src/main/
    ├── java/com/sqlplayground/
    │   ├── config/        AppConfig, WebConfig, AppInitializer
    │   ├── controller/    Dashboard, Table, Data, QueryBuilder, History, Connection, GlobalExceptionHandler
    │   ├── service/       Table, Data, QueryBuilder, History, Connection
    │   ├── dao/           TableDAO, DataDAO, HistoryDAO
    │   ├── model/         TableModel, ColumnModel, HistoryModel, RecordModel
    │   └── util/          SQLGenerator, MetadataUtility
    ├── resources/db.properties
    └── webapp/
        ├── WEB-INF/views/ JSP pages (+ common/header.jsp, footer.jsp)
        ├── css/           theme, dashboard, forms, table, animation, responsive
        └── js/            shared helpers + one script per page
```

## Security

- Table and column names must match `[A-Za-z_][A-Za-z0-9_]{0,63}` and are wrapped in backticks.
- `INSERT`, `UPDATE` and `DELETE` bind all values as `?` parameters; values are never concatenated into executed SQL.
- Join types, aggregate functions and `HAVING` operators are restricted to fixed whitelists, and the `HAVING` value must be a plain number.
- Destructive actions (drop table, delete record) require explicit confirmation.

## Troubleshooting

| Problem | Fix |
|---|---|
| `ClassNotFoundException` for `jakarta.servlet.*` | You are on Tomcat 9 or older. Use Tomcat 10+. |
| Every page shows an error / connection pill says Disconnected | Make sure MySQL is running and `db.properties` has the right URL, user and password. |
| HTTP 500 saying parameter name information is not available | Rebuild the project; all `@RequestParam` annotations in this repo are explicitly named, so a stale build is the usual cause. |
| Port 8080 already in use | Stop the other process or change the Tomcat port. |
| CSS/JS changes not showing | Hard refresh the browser (Ctrl+Shift+R). |

## Roadmap

- Foreign keys and an ER-diagram view
- `ALTER TABLE` support
- Subqueries, views and indexes
- Transaction demo mode (`COMMIT`, `ROLLBACK`, `SAVEPOINT`)
- Authentication and per-user query history
- Schema export and import as SQL scripts

## Team

| Name | Roll No. |
|---|---|
| Aditya Kumar | 2401331690002 |
| Ankit Kumar  | 2401331690006 |


**Supervisor:** Ms. Aditee Mattoo, Assistant Professor, Department of Computer Science & Engineering, Noida Institute of Engineering and Technology, Greater Noida.

## License

Developed for academic purposes. All rights reserved by the authors.
