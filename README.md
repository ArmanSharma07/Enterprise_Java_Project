# Student Management — Enterprise Java Project

Welcome to **Student Management**, a Spring Boot REST API built step by step across nine hands-on lab experiments for the Enterprise Java course .

What started as a simple `"Hello, Spring Boot!"` endpoint gradually evolved into a complete, database-backed backend service. Along the way, it picked up full CRUD operations, MySQL persistence with Spring Data JPA, dynamic search, pagination and sorting, a one-to-many Student–Course relationship, bean validation with centralized error handling, automated integration tests, and HTTP Basic authentication.


## Why This Project Exists

Rather than jumping straight into a finished codebase, this project tells the story of how a real Spring Boot application grows:

1. Start with a minimal web service that responds to a browser or `curl` request.
2. Model data in Java and test endpoints in memory to understand HTTP verbs and JSON.
3. Notice that restarting the app wipes in-memory data—and solve that by wiring up MySQL and JPA.
4. Add real-world querying (search, pagination, sorting) and relational modeling (`Student` → `Course`).
5. Protect data integrity with validation and clean error responses, verify behavior with tests, and lock down endpoints with Spring Security.


## Tech Stack

| Layer / Tool | Details |
| :--- | :--- |
| **Language** | Java 21 |
| **Framework** | Spring Boot `4.1.1` (Web MVC, Data JPA, Validation, Security, DevTools) |
| **Database** | MySQL 8.x (schema: `studentdb`) |
| **Build Tool** | Maven (includes `mvnw` / `mvnw.cmd` wrapper) |
| **Testing** | JUnit 5, Spring MockMvc, cURL |
| **IDE** | IntelliJ IDEA |
| **Coordinates** | `com.example.ucs067` : `student-management` |


## Project Structure
```
student-management/
├── pom.xml
├── mvnw / mvnw.cmd
└── src/
    ├── main/
    │   ├── java/com/example/ucs067/studentmanagement/
    │   │   ├── StudentManagementApplication.java   # Application entry point
    │   │   ├── Student.java                        # Student JPA entity & validation rules
    │   │   ├── Course.java                         # Course JPA entity (linked to Student)
    │   │   ├── config/
    │   │   │   └── SecurityConfig.java             # HTTP Basic auth & endpoint access rules
    │   │   ├── controller/
    │   │   │   └── HelloController.java            # REST endpoints for students & courses
    │   │   ├── exception/
    │   │   │   └── GlobalExceptionHandler.java     # Centralized 400 validation error formatting
    │   │   └── repository/
    │   │       ├── StudentRepository.java          # JPA repository + derived search queries
    │   │       └── CourseRepository.java           # JPA repository for courses
    │   └── resources/
    │       └── application.properties              # Port, MySQL connection, Hibernate & Security config
    └── test/
        └── java/com/example/ucs067/studentmanagement/
            └── StudentManagementApplicationTests.java
```



## How to Run the Application

### Prerequisites
Make sure you have the following installed locally:
- **JDK 21**
- **MySQL 8.x** running on port `3306`
- **Maven** (optional, since the Maven Wrapper `mvnw` / `mvnw.cmd` is included)

### 1. Clone the Repository
git clone https://github.com/ArmanSharma07/Enterprise_Java_Project.git
cd Enterprise_Java_Project

*(If your local checkout places the project inside a `student-management` subdirectory, `cd` into it before running Maven commands.)*

### 2. Create the MySQL Database
Open your MySQL CLI or client and create the schema:
CREATE DATABASE studentdb;

### 3. Configure `application.properties`
Open `src/main/resources/application.properties` and update the MySQL username/password to match your local setup:

```properties
spring.application.name=student-management
server.port=8081
spring.datasource.url=jdbc:mysql://localhost:3306/studentdb
spring.datasource.username=root
spring.datasource.password=Arman@2007
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.security.user.name=admin
spring.security.user.password=Project@2007
```

> **Note:** The credentials configured in `application.properties` are strictly for local coursework and demonstration—never commit or reuse development credentials in a deployed environment.

### 4. Start the Application
Run the app using the Maven Wrapper:

**On Windows (PowerShell / CMD):**
```powershell
.\mvnw.cmd spring-boot:run
```

**On macOS / Linux:**
```bash
./mvnw spring-boot:run
```

Alternatively, open the project in **IntelliJ IDEA** and run `StudentManagementApplication.java`.

The server starts on **`http://localhost:8081`**.

### 5. Verify It Works
Try the public endpoint (no login needed):
```bash
curl http://localhost:8081/hello
# Expected output: Hello, Spring Boot!
```

Try a protected endpoint using the configured HTTP Basic credentials:
```bash
curl -u admin:Project@2007 http://localhost:8081/students
```

Create a new student:
```bash
curl -u admin:Project@2007 -X POST http://localhost:8081/students \
  -H "Content-Type: application/json" \
  -d '{"name":"Aarav","email":"aarav@example.com"}'
```

### 6. Run the Automated Tests
```bash
./mvnw test
```
*(On Windows: `.\mvnw.cmd test`. Note that `@SpringBootTest` loads the full application context, so MySQL should be running with `studentdb` available when running tests.)*

---

## API Reference

All endpoints except `GET /hello` are protected by **HTTP Basic Authentication** (default local user: `admin` / `Project@2007`).

| Method | Endpoint | Access | Description |
| :--- | :--- | :--- | :--- |
| `GET` | `/hello` | Public | Health/greeting check returning `"Hello, Spring Boot!"` |
| `GET` | `/student` | Protected | Returns a sample hardcoded `Student` object |
| `GET` | `/students` | Protected | Fetch all students |
| `GET` | `/students/{id}` | Protected | Fetch one student by ID (`404` if not found) |
| `POST` | `/students` | Protected | Create a new student (validated) |
| `PUT` | `/students/{id}` | Protected | Update an existing student's name and email (validated, `404` if missing) |
| `DELETE` | `/students/{id}` | Protected | Delete a student (`404` if missing) |
| `GET` | `/students/search/name/{name}` | Protected | Exact match search by student name |
| `GET` | `/students/search/name/contains/{keyword}` | Protected | Substring search on student name |
| `GET` | `/students/search/email/{email}` | Protected | Search students by email |
| `GET` | `/students/page?page=0&size=2` | Protected | Paginated list of students (`Pageable`) |
| `GET` | `/students/sort/name` | Protected | Students sorted alphabetically by name |
| `GET` | `/courses` | Protected | Fetch all courses |
| `POST` | `/students/{studentId}/courses` | Protected | Enroll/attach a new course to a specific student |

---

## Implementation  (Experiment by Experiment)

Each experiment built naturally on the limitations or questions uncovered in the previous one:

### Experiment 1 — Project Setup
1. Downloaded IntelliJ IDEA .
2. Created a Maven project named student-management with Git repository enabled, Java 21 (Oracle OpenJDK), Jar packaging, and Properties configuration.
3. Added the Spring Web and Spring Boot DevTools dependencies.

### Experiment 2 — First REST Endpoint
1. Confirmed the main class StudentManagementApplication with @SpringBootApplication.
2. Changed the port to 8081 in application.properties (server.port=8081).
3. Created the controller package and HelloController (@RestController) with a @GetMapping("/hello") method returning "Hello, Spring Boot!".
4. Verified at http://localhost:8081/hello

### Experiment 3 — CRUD with JSON & In-Memory Storage
1. Made a Student class with an id, name, and email.
2. Kept the students in a plain list while the app was running, so everything reset on restart.
3. Wrote endpoints to view, add, update, and delete students, and returned a 404 when a student wasn't found.
4. Learned how Spring turns JSON into Java objects and how URLs carry values like an id.

### Experiment 4 — MySQL Integration with Spring Data JPA
1. Set up MySQL and created a studentdb database for the project.
2. Added JPA and the MySQL driver, then turned Student into a database entity.
3. Created a repository so Spring handles the SQL for me.
4. Restarted the app to confirm the data was still there, which the old list couldn't do.

### Experiment 5 — Derived Queries, Pagination & Sorting
1. Added search methods just by naming them properly, such as finding students by name or email.
2. Built a "name contains" search for partial matches.
3. Added a paged endpoint so large lists come back in small chunks.
4. Added an endpoint that returns students in alphabetical order.

### Experiment 6 — One-to-Many Relationship (`Student` ↔ `Course`)
1. Created a Course entity and its repository.
2. Connected the two so that one student can have many courses.
3. Added an endpoint to attach a new course to a particular student.
4. Fixed an endless JSON loop between the two classes using Jackson annotations.

### Experiment 7 — Bean Validation & Centralized Error Handling
1. Added rules so a student can't be saved with an empty name or a badly formatted email.
2. Turned on validation in the create and update endpoints.
3. Wrote one central error handler that returns a clear 400 message instead of a messy error page.
4. Checked that a rejected update leaves the saved record untouched.

### Experiment 8 — API Testing with cURL & JUnit / MockMvc
1. Went through every endpoint using cURL and checked the status codes (200, 400, 404).
2. Let the database generate student IDs automatically instead of typing them in.
3. Wrote my first automated tests with JUnit and MockMvc to check /hello and the validation errors.

### Experiment 9 — Securing the API with Spring Security
1. Added Spring Security and saw how it blocks access by default.
2. Set a fixed username and password for local testing.
3. Wrote a security config where /hello stays public and everything else needs a login.
4. Tested with the right password, the wrong one, and none at all to see the 200 and 401 responses.
---

## Key Concepts Explored

1. **Spring Boot fundamentals:** auto-configuration, @SpringBootApplication, embedded server, externalised configuration.
2. **REST design:** @RestController, @GetMapping/@PostMapping/@PutMapping/@DeleteMapping, @PathVariable, @RequestBody, and ResponseEntity with proper HTTP status codes.
3. **JSON binding:** why the default constructor and setters are needed for deserialisation.
4. **ORM with JPA/Hibernate:** @Entity, @Id, @GeneratedValue, and automatic table creation.
5. **Spring Data JPA:** JpaRepository, derived query methods, Pageable, and Sort.
6. **Entity relationships:** @OneToMany / @ManyToOne, owning side vs. inverse side, cascade and orphan removal, and avoiding JSON recursion.
7. **Bean Validation:** @NotBlank, @Email, @Valid.
8. **Centralized error handling:** @RestControllerAdvice and @ExceptionHandler.
9. **Testing:** JUnit 5, MockMvc, and manual API testing with cURL.
10. **Security:** authentication vs. protected resources, HTTP Basic, SecurityFilterChain, and the CSRF trade-off.
11. **Tools:** Maven dependency management, MySQL command line, Git and GitHub.
---

## Learning Outcomes Achieved

By completing this project from an empty repository to a secured REST service, I can now:

1. Set up and run a Spring Boot project from scratch.
2. Build a complete CRUD REST API and test it with cURL.
3. Persist data in MySQL using JPA and Spring Data repositories without hand-writing SQL.
4. Implement search, pagination, and sorting.
5. Model and query a one-to-many relationship between entities.
6. Validate input and return meaningful, consistent error responses.
7. Write automated API tests with JUnit and MockMvc.
8. Secure an API with HTTP Basic authentication and distinguish public from protected endpoints.
9. Organise code into controller, entity, repository, config, and exception layers.
10. Use Git and GitHub to version and publish a project.

### Conclusion
This project helped me understand how a simple Java program can grow step by step into a real, database-backed REST API—complete with data validation, automated testing, and user authentication.

## Author
Arman Sharma 
GitHub: ArmanSharma07
B.Tech – Computer Science and Engineering
IIIT Sonepat