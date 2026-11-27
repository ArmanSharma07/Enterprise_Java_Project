# Student Management — Enterprise Java Project

Welcome to **Student Management**, a Spring Boot REST API built step by step across nine hands-on lab experiments for the Enterprise Java course (`com.example.ucs067`).

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
In this experiment, essentially built out a complete set of CRUD (Create, Read, Update, Delete) features for a Spring Boot student management app.

step-by-step process:
1. Prepared the Student Model: updated Student.java with a default constructor and setters so it can easily handle incoming JSON data.
2. Set Up a Temporary Database: Instead of connecting to a real database right away, you created an in-memory list inside HelloController.java to hold some dummy student records while the app runs.
3. Built the REST APIs: Gradually added endpoints to fetch all students, look up a specific student by ID, add new students via POST, update existing details using PUT, and delete records.
4. Tested Everything: Used curl commands in the command line to test out each endpoint and make sure everything—including error handling like 404 responses—was working properly.

### Experiment 4 — MySQL Integration with Spring Data JPA
In the previous step, all our student data disappeared every time the app restarted because it only lived in memory. To make our data stick around permanently, we connected the project to a real MySQL database:

1. Set up MySQL & Project Dependencies
We installed MySQL, created a new database called studentdb, and added spring-boot-starter-data-jpa and mysql-connector-j to our project so Spring Boot knows how to talk to MySQL.
2. Connected the Database in application.properties
We added our MySQL login details and database URL to application.properties, along with spring.jpa.hibernate.ddl-auto=update so Hibernate can automatically create and update our database tables for us.
3. Turned Student into a Database Table (@Entity)
We annotated the Student class with @Entity, @Table(name = "students"), and @Id. This tells Spring and Hibernate: "This Java class represents the students table in MySQL, and id is its primary key."
4. Created StudentRepository
We created an interface StudentRepository that extends JpaRepository<Student, Integer>. Without writing a single line of SQL, Spring Data JPA gives us ready-to-use database operations out of the box.
5. Swapped the In-Memory List for the Repository
In HelloController, we replaced our temporary ArrayList with StudentRepository methods—using findAll(), save(), findById(), existsById(), and deleteById() to handle all CRUD requests directly in MySQL.
6. Verified Everything Persists
Finally, we restarted the application and ran SELECT * FROM students; in MySQL. Seeing our saved students still sitting safely in the table proved that our data now survives restarts!

### Experiment 5 — Derived Queries, Pagination & Sorting
Once our students were safely stored in MySQL, simply fetching all of them at once wasn't enough. Next, we made it easy to search, break results into pages, and sort them—without having to write any SQL manually:

1. Added Smart Search Methods (Derived Queries)
In StudentRepository, we declared three simple method names: findByName, findByNameContaining, and findByEmail. Just by reading the method names, Spring Data JPA automatically figures out and generates the right SQL queries behind the scenes.
2. Exposed Search Endpoints in the Controller
We connected those repository methods to new API routes in HelloController so users can search for a student by exact name, partial keyword, or email address directly through URLs.
3. Added Pagination (Pageable)
Instead of dumping every student onto one screen, we added /students/page?page=0&size=2 using Spring's Pageable. This lets us fetch records in neat chunks (for example, 2 students per page), just like pages in a real web app.
4. Added Sorting (Sort)
We added /students/sort/name using Sort.by("name") so the API can return all students cleanly arranged in alphabetical order by their name.

### Experiment 6 — One-to-Many Relationship (`Student` ↔ `Course`)
In real life, a student doesn't exist in isolation—they enroll in courses. In this step, we taught our application how to connect one student to multiple courses:

1. Created the Course Entity
We built a new Course class with id, courseName, and credits, letting the database generate each course's id automatically.
2. Linked Student and Course Together
We set up a two-way relationship between the two entities:
--> On Student, we added @OneToMany(mappedBy = "student", cascade = CascadeType.ALL, orphanRemoval = true) so one student can hold a list of courses (and deleting or updating a student cleans up their courses automatically).
-->On Course, we added @ManyToOne, making Course the owning side of the relationship that stores the student_id foreign key in the database.
3. Stopped Infinite JSON Loops
Because Student points to Course and Course points right back to Student, converting them to JSON would loop forever. Adding @JsonManagedReference on Student and @JsonBackReference on Course cleanly broke that loop.
4. Added CourseRepository and Course Endpoints
We created CourseRepository and added two new endpoints in the controller: GET /courses to view all courses, and POST /students/{studentId}/courses to enroll a student in a new course.
5. Verified Multiple Enrollments
We tested adding more than one course to the same student and confirmed that one student can successfully hold multiple courses in both the API response and MySQL.

### Experiment 7 — Bean Validation & Centralized Error Handling
Up until now, someone could save a student with an empty name or a fake email address. In this step, we added simple safety rules so bad data gets blocked and explained clearly:

1. Added the Validation Tool
We added spring-boot-starter-validation to the project so Spring Boot can automatically check incoming data before saving it.

2. Set Rules on Student Fields
We added @NotBlank and @Email on the Student fields (so names and emails cannot be blank, and emails must look like real emails), and placed @Valid on the POST and PUT request bodies to make sure those rules are checked every time someone adds or edits a student.

3. Created a Central Error Handler (GlobalExceptionHandler)
Instead of letting Spring throw ugly default errors, we created GlobalExceptionHandler using @RestControllerAdvice and @ExceptionHandler(MethodArgumentNotValidException.class). Whenever validation fails, it catches the mistake in one place and sends back an HTTP 400 (Bad Request) along with a clean, readable JSON list showing exactly which field failed and why.

4. Tested Bad Data Safely
We sent invalid POST and PUT requests and confirmed they were blocked right away—and most importantly, when an invalid PUT update failed, the existing student data stayed untouched and safe.

### Experiment 8 — API Testing with cURL & JUnit / MockMvc
In this step, we tested everything properly—both by hand and automatically—and fixed student IDs so we no longer have to type them ourselves:

1. Tested Everything by Hand with cURL
We ran cURL commands against our live MySQL database to check every CRUD action (Create, Read, Update, Delete) and made sure the right HTTP status codes came back: 200 for success, 400 for bad input, and 404 when a student isn't found.

2. Let MySQL Generate Student IDs Automatically
We updated Student to use @GeneratedValue(strategy = GenerationType.IDENTITY) with an Integer id, added a constructor without the id, and removed @Positive from id. Now we only send name and email, and MySQL picks the next ID automatically.

3. Fixed the students Table in MySQL
Because course was linked to students.id with a foreign key, we temporarily dropped that foreign key, updated students.id to AUTO_INCREMENT, and then restored the foreign key so everything stayed safely connected.

4. Wrote Automated Tests with JUnit and MockMvc
Using @SpringBootTest, @AutoConfigureMockMvc, and MockMvc, we added automated tests to check three things automatically:
-->The Spring Boot app starts up cleanly (contextLoads).
-->Calling /hello returns the expected greeting message.
-->Sending invalid student data returns 400 Bad Request along with the expected validation error messages.

### Experiment 9 — Securing the API with Spring Security
Up to this point, anyone could view, edit, or delete students without logging in. In the final step, we added basic username-and-password protection to keep our API safe:

1. Added Spring Security and Saw How It Works by Default
We added spring-boot-starter-security to the project. Right away, Spring Boot locked every endpoint automatically—using a default username (user) and printing a temporary random password in the console.

2. Set Our Own Fixed Username and Password
So we wouldn't have to copy a new random password every time the app restarted, we set a fixed local username and password inside application.properties.

3. Created SecurityConfig to Control Who Can Access What
We added a SecurityConfig class with a SecurityFilterChain bean to set clear rules:

-->/hello stays public (permitAll) so anyone can check if the app is running.
-->Every other request requires a login (anyRequest().authenticated()).
-->Enabled HTTP Basic authentication so tools like curl or a browser can send a username and password.
-->Turned off CSRF since this is just a simple, stateless local API exercise using Basic auth rather than browser forms.

4. Tested the Lock
We tested protected endpoints three ways to be sure it worked:
-->No login details → blocked with 401 Unauthorized
-->Wrong username/password → blocked with 401 Unauthorized
-->Correct username/password → allowed through with 200 OK
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