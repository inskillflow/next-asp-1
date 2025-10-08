# Phase 1 — Développement des APIs (Next.js + Prisma/PostgreSQL)

<ul>
  <li><a href="#sec-phase1">Phase 1 — Développement des APIs</a></li>
  <li><a href="#sec-master">1 – Tableau récapitulatif EXHAUSTIF des endpoints</a></li>
  <li><a href="#sec-conventions">2 – Conventions HTTP (communes)</a></li>
  <li>
    <a href="#sec-endpoints">3 – Endpoints (détails, I/O & liens)</a>
    <ul>
      <li><a href="#sec-courses">3.1 Courses — /api/courses</a> (<a href="#ep-courses-list">liste</a>, <a href="#ep-courses-create">create</a>, <a href="#ep-courses-get">get</a>, <a href="#ep-courses-update">update</a>, <a href="#ep-courses-delete">delete</a>)</li>
      <li><a href="#sec-instructors">3.2 Instructors — /api/instructors</a> (<a href="#ep-instructors-list">liste</a>, <a href="#ep-instructors-create">create</a>, <a href="#ep-instructors-get">get</a>, <a href="#ep-instructors-update">update</a>, <a href="#ep-instructors-delete">delete</a>)</li>
      <li><a href="#sec-students">3.3 Students — /api/students</a> (<a href="#ep-students-list">liste</a>, <a href="#ep-students-create">create</a>, <a href="#ep-students-get">get</a>, <a href="#ep-students-update">update</a>, <a href="#ep-students-delete">delete</a>)</li>
      <li><a href="#sec-enrollments">3.4 Enrollments — /api/enrollments</a> (<a href="#ep-enrollments-list">liste</a>, <a href="#ep-enrollments-create">create</a>, <a href="#ep-enrollments-get">get</a>, <a href="#ep-enrollments-update">update</a>, <a href="#ep-enrollments-delete">delete</a>)</li>
      <li><a href="#sec-health">3.5 Health — /api/health</a></li>
    </ul>
  </li>
  <li><a href="#sec-schemas">4 – Schémas de données (types de réponse)</a></li>
  <li><a href="#sec-validation">5 – Règles de validation (résumé)</a></li>
  <li><a href="#sec-errors">6 – Erreurs & format commun</a></li>
  <li><a href="#sec-openapi">7 – OpenAPI (extrait prêt à coller)</a></li>
  <li><a href="#sec-compat">8 – Compatibilité Phase 3 (ajout auth/rôles sans casser)</a></li>
</ul>


<br/>
<br/>


<h1 id="sec-phase1">Phase 1 — Développement des APIs</h1>

<h2 id="sec-master">1 – Tableau récapitulatif EXHAUSTIF des endpoints</h2>

> **Phase 1 :** tous publics (pas d’auth). **Phase 3 :** mêmes endpoints, on ajoute 401/403 via guards.

| #  | Ressource   | Méthode | URL                     | Query (liste)                                         | Body (req)                                                          | Réponse succès              | Codes succès | Erreurs typiques | Ancre                                                       |
| -- | ----------- | ------- | ----------------------- | ----------------------------------------------------- | ------------------------------------------------------------------- | --------------------------- | ------------ | ---------------- | ----------------------------------------------------------- |
| 1  | Courses     | GET     | `/api/courses`          | `page,size,sort,level,instructorId,priceMin,priceMax` | —                                                                   | `{items[],page,size,total}` | 200          | 500              | <a href="#ep-courses-list">#ep-courses-list</a>             |
| 2  | Courses     | POST    | `/api/courses`          | —                                                     | `{title,description,instructorId,duration,maxStudents,price,level}` | `Course`                    | 201          | 422,409,500      | <a href="#ep-courses-create">#ep-courses-create</a>         |
| 3  | Courses     | GET     | `/api/courses/{id}`     | —                                                     | —                                                                   | `Course`                    | 200/404      | 404,500          | <a href="#ep-courses-get">#ep-courses-get</a>               |
| 4  | Courses     | PUT     | `/api/courses/{id}`     | —                                                     | (mêmes champs, partiels ok)                                         | `Course`                    | 200/404      | 422,404,409,500  | <a href="#ep-courses-update">#ep-courses-update</a>         |
| 5  | Courses     | DELETE  | `/api/courses/{id}`     | —                                                     | —                                                                   | —                           | 204/404      | 404,500          | <a href="#ep-courses-delete">#ep-courses-delete</a>         |
| 6  | Instructors | GET     | `/api/instructors`      | `page,size,sort,email,specialty`                      | —                                                                   | `{items[],page,size,total}` | 200          | 500              | <a href="#ep-instructors-list">#ep-instructors-list</a>     |
| 7  | Instructors | POST    | `/api/instructors`      | —                                                     | `{firstName,lastName,specialty,email,hireDate}`                     | `Instructor`                | 201          | 422,409,500      | <a href="#ep-instructors-create">#ep-instructors-create</a> |
| 8  | Instructors | GET     | `/api/instructors/{id}` | —                                                     | —                                                                   | `Instructor`                | 200/404      | 404,500          | <a href="#ep-instructors-get">#ep-instructors-get</a>       |
| 9  | Instructors | PUT     | `/api/instructors/{id}` | —                                                     | (champs du POST)                                                    | `Instructor`                | 200/404      | 422,404,409,500  | <a href="#ep-instructors-update">#ep-instructors-update</a> |
| 10 | Instructors | DELETE  | `/api/instructors/{id}` | —                                                     | —                                                                   | —                           | 204/404      | 404,500          | <a href="#ep-instructors-delete">#ep-instructors-delete</a> |
| 11 | Students    | GET     | `/api/students`         | `page,size,sort,email,level`                          | —                                                                   | `{items[],page,size,total}` | 200          | 500              | <a href="#ep-students-list">#ep-students-list</a>           |
| 12 | Students    | POST    | `/api/students`         | —                                                     | `{firstName,lastName,email,registrationDate,level}`                 | `Student`                   | 201          | 422,409,500      | <a href="#ep-students-create">#ep-students-create</a>       |
| 13 | Students    | GET     | `/api/students/{id}`    | —                                                     | —                                                                   | `Student`                   | 200/404      | 404,500          | <a href="#ep-students-get">#ep-students-get</a>             |
| 14 | Students    | PUT     | `/api/students/{id}`    | —                                                     | (champs du POST)                                                    | `Student`                   | 200/404      | 422,404,409,500  | <a href="#ep-students-update">#ep-students-update</a>       |
| 15 | Students    | DELETE  | `/api/students/{id}`    | —                                                     | —                                                                   | —                           | 204/404      | 404,500          | <a href="#ep-students-delete">#ep-students-delete</a>       |
| 16 | Enrollments | GET     | `/api/enrollments`      | `page,size,sort,status,courseId,studentId`            | —                                                                   | `{items[],page,size,total}` | 200          | 500              | <a href="#ep-enrollments-list">#ep-enrollments-list</a>     |
| 17 | Enrollments | POST    | `/api/enrollments`      | —                                                     | `{courseId,studentId,status,grade?}`                                | `Enrollment`                | 201          | 422,409,500      | <a href="#ep-enrollments-create">#ep-enrollments-create</a> |
| 18 | Enrollments | GET     | `/api/enrollments/{id}` | —                                                     | —                                                                   | `Enrollment`                | 200/404      | 404,500          | <a href="#ep-enrollments-get">#ep-enrollments-get</a>       |
| 19 | Enrollments | PUT     | `/api/enrollments/{id}` | —                                                     | `{status?,grade?}`                                                  | `Enrollment`                | 200/404      | 422,404,500      | <a href="#ep-enrollments-update">#ep-enrollments-update</a> |
| 20 | Enrollments | DELETE  | `/api/enrollments/{id}` | —                                                     | —                                                                   | —                           | 204/404      | 404,500          | <a href="#ep-enrollments-delete">#ep-enrollments-delete</a> |
| 21 | Health      | GET     | `/api/health`           | —                                                     | —                                                                   | `{status,db,time}`          | 200/500      | 500              | <a href="#ep-health">#ep-health</a>                         |






<br/>
<br/>

<h2 id="sec-conventions">2 – Conventions HTTP (communes)</h2>

JSON, pagination `page/size`, tri `sort=field:asc|desc`, dates ISO-UTC, 2xx constants; **401/403 seront ajoutés en Phase 3 sans changer cette table**.


* **Format** : `application/json; charset=utf-8` (requêtes & réponses).
* **Dates** : ISO-8601 en **UTC** (ex. `2025-01-01T10:00:00Z`).
* **Pagination** : `page` (≥1), `size` (1..100).
* **Tri** : `sort=<champ>:asc|desc` (ex. `sort=createdAt:desc`).
* **Filtres** : paramètres dédiés par ressource (voir chaque endpoint).
* **Codes succès** : `200 OK`, `201 Created`, `204 No Content`.
* **Erreurs Phase 1** : `400/409/422/404/500`.
* **Compat Phase 3** : on ajoutera `401 Unauthorized` / `403 Forbidden` **sans changer** les URLs ni les payloads.



<br/>
<br/>


<h2 id="sec-endpoints">3 – Endpoints (détails, I/O & liens)</h2>

> Tous **publics** en Phase 1. En Phase 3, on activera des guards sans modifier ces sections.



<h3 id="sec-courses">3.1 Courses — <code>/api/courses</code></h3>

<h4 id="ep-courses-list">GET /api/courses</h4>

**Query** : `page`, `size`, `sort`, `level`, `instructorId`, `priceMin`, `priceMax`
**200 OK**

```json
{
  "items": [
    {
      "id": "uuid",
      "title": "Intro C#",
      "level": "Beginner",
      "price": 299.99,
      "instructorId": "uuid",
      "createdAt": "2025-01-01T10:00:00Z"
    }
  ],
  "page": 1,
  "size": 20,
  "total": 42
}
```

**Erreurs** : `500`



<h4 id="ep-courses-create">POST /api/courses</h4>

**Body**

```json
{
  "title": "Intro C#",
  "description": "...",
  "instructorId": "uuid",
  "duration": 40,
  "maxStudents": 30,
  "price": 299.99,
  "level": "Beginner"
}
```

**201 Created** → objet `Course`
**Erreurs** : `422` (validation), `409` (conflit), `500`



<h4 id="ep-courses-get">GET /api/courses/{id}</h4>

**200 OK** → `Course` • **404 Not Found**
**Erreurs** : `404`, `500`



<h4 id="ep-courses-update">PUT /api/courses/{id}</h4>

**Body** : mêmes champs que POST (autorisez partiels si vous gérez “PATCH-like”).
**200 OK** → `Course` • **404 Not Found**
**Erreurs** : `422`, `404`, `409`, `500`



<h4 id="ep-courses-delete">DELETE /api/courses/{id}</h4>

**204 No Content** • **404 Not Found**
**Erreurs** : `404`, `500`



<h3 id="sec-instructors">3.2 Instructors — <code>/api/instructors</code></h3>

<h4 id="ep-instructors-list">GET /api/instructors</h4>

**Query** : `page`, `size`, `sort`, `email`, `specialty`
**200 OK**

```json
{
  "items": [
    {
      "id": "uuid",
      "firstName": "Ada",
      "lastName": "Lovelace",
      "email": "ada@ex.com",
      "specialty": "Informatique",
      "hireDate": "2019-08-20",
      "createdAt": "2025-01-01T10:00:00Z"
    }
  ],
  "page": 1,
  "size": 20,
  "total": 3
}
```

**Erreurs** : `500`


<h4 id="ep-instructors-create">POST /api/instructors</h4>

**Body**

```json
{
  "firstName": "Ada",
  "lastName": "Lovelace",
  "specialty": "Informatique",
  "email": "ada@ex.com",
  "hireDate": "2019-08-20"
}
```

**201 Created** → `Instructor`
**Erreurs** : `422`, `409`, `500`



<h4 id="ep-instructors-get">GET /api/instructors/{id}</h4>

**200 OK** → `Instructor` • **404 Not Found**
**Erreurs** : `404`, `500`

---

<h4 id="ep-instructors-update">PUT /api/instructors/{id}</h4>

**Body** : champs du POST
**200 OK** • **404 Not Found**
**Erreurs** : `422`, `404`, `409`, `500`

---

<h4 id="ep-instructors-delete">DELETE /api/instructors/{id}</h4>

**204 No Content** • **404 Not Found**
**Erreurs** : `404`, `500`



<h3 id="sec-students">3.3 Students — <code>/api/students</code></h3>

<h4 id="ep-students-list">GET /api/students</h4>

**Query** : `page`, `size`, `sort`, `email`, `level`
**200 OK**

```json
{
  "items": [
    {
      "id": "uuid",
      "firstName": "Sophie",
      "lastName": "Bernard",
      "email": "sophie@ex.com",
      "registrationDate": "2025-02-15",
      "level": "Intermediate",
      "createdAt": "2025-01-01T10:00:00Z"
    }
  ],
  "page": 1,
  "size": 20,
  "total": 5
}
```

**Erreurs** : `500`



<h4 id="ep-students-create">POST /api/students</h4>

**Body**

```json
{
  "firstName": "Sophie",
  "lastName": "Bernard",
  "email": "sophie@ex.com",
  "registrationDate": "2025-02-15",
  "level": "Intermediate"
}
```

**201 Created** → `Student`
**Erreurs** : `422`, `409`, `500`



<h4 id="ep-students-get">GET /api/students/{id}</h4>

**200 OK** → `Student` • **404 Not Found**
**Erreurs** : `404`, `500`

---

<h4 id="ep-students-update">PUT /api/students/{id}</h4>

**Body** : champs du POST
**200 OK** • **404 Not Found**
**Erreurs** : `422`, `404`, `409`, `500`



<h4 id="ep-students-delete">DELETE /api/students/{id}</h4>

**204 No Content** • **404 Not Found**
**Erreurs** : `404`, `500`



<h3 id="sec-enrollments">3.4 Enrollments — <code>/api/enrollments</code></h3>

<h4 id="ep-enrollments-list">GET /api/enrollments</h4>

**Query** : `page`, `size`, `sort`, `status`, `courseId`, `studentId`
**200 OK**

```json
{
  "items": [
    {
      "id": "uuid",
      "courseId": "uuid",
      "studentId": "uuid",
      "status": "Active",
      "grade": 85.5,
      "enrollmentDate": "2025-01-12T09:30:00Z",
      "createdAt": "2025-01-12T09:30:00Z"
    }
  ],
  "page": 1,
  "size": 20,
  "total": 8
}
```

**Erreurs** : `500`



<h4 id="ep-enrollments-create">POST /api/enrollments</h4>

**Body**

```json
{
  "courseId": "uuid",
  "studentId": "uuid",
  "status": "Active",
  "grade": 88.5
}
```

**201 Created** → `Enrollment`
**Erreurs** : `422`, `409` (doublon `courseId`+`studentId`), `500`



<h4 id="ep-enrollments-get">GET /api/enrollments/{id}</h4>

**200 OK** → `Enrollment` • **404 Not Found**
**Erreurs** : `404`, `500`


<h4 id="ep-enrollments-update">PUT /api/enrollments/{id}</h4>

**Body**

```json
{ "status": "Completed", "grade": 90 }
```

**200 OK** → `Enrollment` • **404 Not Found**
**Erreurs** : `422`, `404`, `500`




<h4 id="ep-enrollments-delete">DELETE /api/enrollments/{id}</h4>

**204 No Content** • **404 Not Found**
**Erreurs** : `404`, `500`




<h3 id="sec-health">3.5 Health — <code>/api/health</code></h3>

<h4 id="ep-health">GET /api/health</h4>

**200 OK**

```json
{ "status": "ok", "db": "up", "time": "2025-01-01T10:00:00Z" }
```

**500**

```json
{ "status": "degraded", "db": "down" }
```

















<br/>
<br/>


<h2 id="sec-schemas">4 – Schémas de données (types de réponse)</h2>

**Course**

```json
{ "id":"uuid","title":"string(5..100)","description":"string(10..500)","instructorId":"uuid","duration":40,"maxStudents":30,"price":299.99,"level":"Beginner|Intermediate|Advanced","createdAt":"ISO-UTC" }
```

**Instructor**

```json
{ "id":"uuid","firstName":"string(2..50)","lastName":"string(2..50)","specialty":"string(3..100)","email":"email unique","hireDate":"YYYY-MM-DD","createdAt":"ISO-UTC" }
```

**Student**

```json
{ "id":"uuid","firstName":"string(2..50)","lastName":"string(2..50)","email":"email unique","registrationDate":"YYYY-MM-DD","level":"Beginner|Intermediate|Advanced","createdAt":"ISO-UTC" }
```

**Enrollment**

````json
{ "id":"uuid","courseId":"uuid","studentId":"uuid","enrollmentDate":"ISO-UTC","status":"Active|Completed|Cancelled","grade":85.5,"createdAt":"ISO-UTC" }
````

<br/>
<br/>


<h2 id="sec-validation">5 – Règles de validation (résumé)</h2>

- **Course** : `title` 5..100 · `description` 10..500 · `duration` 1..200 · `maxStudents` 5..100 · `price` 0..10000 · `level` ∈ {Beginner,Intermediate,Advanced} · `instructorId` uuid.  
- **Instructor** : `firstName/lastName` 2..50 · `specialty` 3..100 · `email` valide **unique** · `hireDate` ≤ now.  
- **Student** : `firstName/lastName` 2..50 · `email` valide **unique** · `registrationDate` ≤ now · `level` valide.  
- **Enrollment** : (`courseId`,`studentId`) requis & **unique** · `status` ∈ {Active,Completed,Cancelled} · `grade` 0..100 (optionnelle).


<br/>
<br/>



<h2 id="sec-errors">6 – Erreurs &amp; format commun</h2>

<strong>Format recommandé (422 – validation, exemple Zod)</strong>

```json
{
  "status": 422,
  "error": "Unprocessable Entity",
  "message": "Validation failed",
  "details": {
    "fieldErrors": {
      "title": ["String must contain at least 5 character(s)"],
      "duration": ["Number must be greater than or equal to 1"],
      "level": ["Invalid enum value. Expected 'Beginner' | 'Intermediate' | 'Advanced'"]
    },
    "formErrors": []
  },
  "traceId": "req-9c1f-3b2a-…"
}
```

<strong>409 – Conflict (ex. email déjà utilisé ou inscription dupliquée)</strong>

```json
{
  "status": 409,
  "error": "Conflict",
  "message": "Unique constraint violated on field(s): email",
  "traceId": "req-409c-…"
}
```

<strong>404 – Not Found</strong>

```json
{
  "status": 404,
  "error": "Not Found",
  "message": "Course not found",
  "traceId": "req-404a-…"
}
```

<strong>500 – Internal Server Error</strong>

```json
{
  "status": 500,
  "error": "Internal Server Error",
  "message": "Unexpected server error",
  "traceId": "req-500e-…"
}
```

<p><em>Phase 3</em> ajoutera simplement les codes suivants (même format) :</p>

<strong>401 – Unauthorized</strong>

```json
{
  "status": 401,
  "error": "Unauthorized",
  "message": "Authentication required",
  "traceId": "req-401u-…"
}
```

<strong>403 – Forbidden</strong>

```json
{
  "status": 403,
  "error": "Forbidden",
  "message": "Access denied for role Student on /api/courses (DELETE)",
  "traceId": "req-403f-…"
}
```



Codes usuels : **422** validation · **409** conflit (email/enrollment) · **404** not found · **500** serveur.
(**Phase 3** ajoutera **401/403** pour auth/rôles, mêmes payloads sinon.)















<br/>
<br/>



<h2 id="sec-openapi">7 – OpenAPI (extrait complet prêt à coller)</h2>

```yaml
openapi: 3.0.3
info:
  title: EduTrack API
  version: 1.0.0
servers:
  - url: http://localhost:3000
paths:
  /api/health:
    get:
      summary: Healthcheck
      responses:
        "200": { description: OK, content: { application/json: { schema: { $ref: "#/components/schemas/HealthOk" }}}}
        "500": { description: Degraded, content: { application/json: { schema: { $ref: "#/components/schemas/HealthDown" }}}}

  /api/courses:
    get:
      summary: List courses
      parameters:
        - $ref: "#/components/parameters/Page"
        - $ref: "#/components/parameters/Size"
        - $ref: "#/components/parameters/Sort"
        - name: level
          in: query
          schema: { type: string, enum: [Beginner, Intermediate, Advanced] }
        - name: instructorId
          in: query
          schema: { type: string, format: uuid }
        - name: priceMin
          in: query
          schema: { type: number, minimum: 0 }
        - name: priceMax
          in: query
          schema: { type: number, minimum: 0 }
      responses:
        "200": { description: OK, content: { application/json: { schema: { $ref: "#/components/schemas/PagedCourse" }}}}
    post:
      summary: Create course
      requestBody:
        required: true
        content:
          application/json:
            schema: { $ref: "#/components/schemas/CourseCreate" }
      responses:
        "201": { description: Created, content: { application/json: { schema: { $ref: "#/components/schemas/Course" }}}}
        "422": { description: Validation error }
        "409": { description: Conflict }

  /api/courses/{id}:
    parameters:
      - $ref: "#/components/parameters/Id"
    get:
      summary: Get course by id
      responses:
        "200": { description: OK, content: { application/json: { schema: { $ref: "#/components/schemas/Course" }}}}
        "404": { description: Not Found }
    put:
      summary: Update course
      requestBody:
        required: true
        content:
          application/json:
            schema: { $ref: "#/components/schemas/CourseUpdate" }
      responses:
        "200": { description: OK, content: { application/json: { schema: { $ref: "#/components/schemas/Course" }}}}
        "404": { description: Not Found }
        "422": { description: Validation error }
        "409": { description: Conflict }
    delete:
      summary: Delete course
      responses:
        "204": { description: No Content }
        "404": { description: Not Found }

  /api/instructors:
    get:
      summary: List instructors
      parameters:
        - $ref: "#/components/parameters/Page"
        - $ref: "#/components/parameters/Size"
        - $ref: "#/components/parameters/Sort"
        - name: email
          in: query
          schema: { type: string, format: email, maxLength: 100 }
        - name: specialty
          in: query
          schema: { type: string, maxLength: 100 }
      responses:
        "200": { description: OK, content: { application/json: { schema: { $ref: "#/components/schemas/PagedInstructor" }}}}
    post:
      summary: Create instructor
      requestBody:
        required: true
        content:
          application/json:
            schema: { $ref: "#/components/schemas/InstructorCreate" }
      responses:
        "201": { description: Created, content: { application/json: { schema: { $ref: "#/components/schemas/Instructor" }}}}
        "422": { description: Validation error }
        "409": { description: Conflict }

  /api/instructors/{id}:
    parameters: [ { $ref: "#/components/parameters/Id" } ]
    get:
      summary: Get instructor
      responses:
        "200": { description: OK, content: { application/json: { schema: { $ref: "#/components/schemas/Instructor" }}}}
        "404": { description: Not Found }
    put:
      summary: Update instructor
      requestBody:
        required: true
        content:
          application/json:
            schema: { $ref: "#/components/schemas/InstructorUpdate" }
      responses:
        "200": { description: OK, content: { application/json: { schema: { $ref: "#/components/schemas/Instructor" }}}}
        "404": { description: Not Found }
        "422": { description: Validation error }
        "409": { description: Conflict }
    delete:
      summary: Delete instructor
      responses:
        "204": { description: No Content }
        "404": { description: Not Found }

  /api/students:
    get:
      summary: List students
      parameters:
        - $ref: "#/components/parameters/Page"
        - $ref: "#/components/parameters/Size"
        - $ref: "#/components/parameters/Sort"
        - name: email
          in: query
          schema: { type: string, format: email, maxLength: 100 }
        - name: level
          in: query
          schema: { type: string, enum: [Beginner, Intermediate, Advanced] }
      responses:
        "200": { description: OK, content: { application/json: { schema: { $ref: "#/components/schemas/PagedStudent" }}}}
    post:
      summary: Create student
      requestBody:
        required: true
        content:
          application/json:
            schema: { $ref: "#/components/schemas/StudentCreate" }
      responses:
        "201": { description: Created, content: { application/json: { schema: { $ref: "#/components/schemas/Student" }}}}
        "422": { description: Validation error }
        "409": { description: Conflict }

  /api/students/{id}:
    parameters: [ { $ref: "#/components/parameters/Id" } ]
    get:
      summary: Get student
      responses:
        "200": { description: OK, content: { application/json: { schema: { $ref: "#/components/schemas/Student" }}}}
        "404": { description: Not Found }
    put:
      summary: Update student
      requestBody:
        required: true
        content:
          application/json:
            schema: { $ref: "#/components/schemas/StudentUpdate" }
      responses:
        "200": { description: OK, content: { application/json: { schema: { $ref: "#/components/schemas/Student" }}}}
        "404": { description: Not Found }
        "422": { description: Validation error }
        "409": { description: Conflict }
    delete:
      summary: Delete student
      responses:
        "204": { description: No Content }
        "404": { description: Not Found }

  /api/enrollments:
    get:
      summary: List enrollments
      parameters:
        - $ref: "#/components/parameters/Page"
        - $ref: "#/components/parameters/Size"
        - $ref: "#/components/parameters/Sort"
        - name: status
          in: query
          schema: { type: string, enum: [Active, Completed, Cancelled] }
        - name: courseId
          in: query
          schema: { type: string, format: uuid }
        - name: studentId
          in: query
          schema: { type: string, format: uuid }
      responses:
        "200": { description: OK, content: { application/json: { schema: { $ref: "#/components/schemas/PagedEnrollment" }}}}
    post:
      summary: Create enrollment
      requestBody:
        required: true
        content:
          application/json:
            schema: { $ref: "#/components/schemas/EnrollmentCreate" }
      responses:
        "201": { description: Created, content: { application/json: { schema: { $ref: "#/components/schemas/Enrollment" }}}}
        "422": { description: Validation error }
        "409": { description: Duplicate (courseId,studentId) }

  /api/enrollments/{id}:
    parameters: [ { $ref: "#/components/parameters/Id" } ]
    get:
      summary: Get enrollment
      responses:
        "200": { description: OK, content: { application/json: { schema: { $ref: "#/components/schemas/Enrollment" }}}}
        "404": { description: Not Found }
    put:
      summary: Update enrollment (status/grade)
      requestBody:
        required: true
        content:
          application/json:
            schema: { $ref: "#/components/schemas/EnrollmentUpdate" }
      responses:
        "200": { description: OK, content: { application/json: { schema: { $ref: "#/components/schemas/Enrollment" }}}}
        "404": { description: Not Found }
        "422": { description: Validation error }
    delete:
      summary: Delete enrollment
      responses:
        "204": { description: No Content }
        "404": { description: Not Found }

components:
  parameters:
    Id:
      name: id
      in: path
      required: true
      schema: { type: string, format: uuid }
    Page:
      name: page
      in: query
      schema: { type: integer, minimum: 1, default: 1 }
    Size:
      name: size
      in: query
      schema: { type: integer, minimum: 1, maximum: 100, default: 20 }
    Sort:
      name: sort
      in: query
      schema: { type: string, example: "createdAt:desc" }
  schemas:
    # Health
    HealthOk:
      type: object
      properties:
        status: { type: string, example: ok }
        db: { type: string, example: up }
        time: { type: string, format: date-time }
    HealthDown:
      type: object
      properties:
        status: { type: string, example: degraded }
        db: { type: string, example: down }

    # Core entities
    Course:
      type: object
      required: [id,title,description,instructorId,duration,maxStudents,price,level,createdAt]
      properties:
        id: { type: string, format: uuid }
        title: { type: string, minLength: 5, maxLength: 100 }
        description: { type: string, minLength: 10, maxLength: 500 }
        instructorId: { type: string, format: uuid }
        duration: { type: integer, minimum: 1, maximum: 200 }
        maxStudents: { type: integer, minimum: 5, maximum: 100 }
        price: { type: number, minimum: 0, maximum: 10000 }
        level: { type: string, enum: [Beginner, Intermediate, Advanced] }
        createdAt: { type: string, format: date-time }
    CourseCreate:
      allOf:
        - $ref: "#/components/schemas/Course"
      required: [title,description,instructorId,duration,maxStudents,price,level]
      properties:
        id: { readOnly: true }
        createdAt: { readOnly: true }
    CourseUpdate:
      type: object
      properties:
        title: { type: string, minLength: 5, maxLength: 100 }
        description: { type: string, minLength: 10, maxLength: 500 }
        instructorId: { type: string, format: uuid }
        duration: { type: integer, minimum: 1, maximum: 200 }
        maxStudents: { type: integer, minimum: 5, maximum: 100 }
        price: { type: number, minimum: 0, maximum: 10000 }
        level: { type: string, enum: [Beginner, Intermediate, Advanced] }

    Instructor:
      type: object
      required: [id,firstName,lastName,specialty,email,hireDate,createdAt]
      properties:
        id: { type: string, format: uuid }
        firstName: { type: string, minLength: 2, maxLength: 50 }
        lastName: { type: string, minLength: 2, maxLength: 50 }
        specialty: { type: string, minLength: 3, maxLength: 100 }
        email: { type: string, format: email, maxLength: 100 }
        hireDate: { type: string, format: date }
        createdAt: { type: string, format: date-time }
    InstructorCreate:
      allOf:
        - $ref: "#/components/schemas/Instructor"
      required: [firstName,lastName,specialty,email,hireDate]
      properties:
        id: { readOnly: true }
        createdAt: { readOnly: true }
    InstructorUpdate:
      type: object
      properties:
        firstName: { type: string, minLength: 2, maxLength: 50 }
        lastName: { type: string, minLength: 2, maxLength: 50 }
        specialty: { type: string, minLength: 3, maxLength: 100 }
        email: { type: string, format: email, maxLength: 100 }
        hireDate: { type: string, format: date }

    Student:
      type: object
      required: [id,firstName,lastName,email,registrationDate,level,createdAt]
      properties:
        id: { type: string, format: uuid }
        firstName: { type: string, minLength: 2, maxLength: 50 }
        lastName: { type: string, minLength: 2, maxLength: 50 }
        email: { type: string, format: email, maxLength: 100 }
        registrationDate: { type: string, format: date }
        level: { type: string, enum: [Beginner, Intermediate, Advanced] }
        createdAt: { type: string, format: date-time }
    StudentCreate:
      allOf:
        - $ref: "#/components/schemas/Student"
      required: [firstName,lastName,email,registrationDate,level]
      properties:
        id: { readOnly: true }
        createdAt: { readOnly: true }
    StudentUpdate:
      type: object
      properties:
        firstName: { type: string, minLength: 2, maxLength: 50 }
        lastName: { type: string, minLength: 2, maxLength: 50 }
        email: { type: string, format: email, maxLength: 100 }
        registrationDate: { type: string, format: date }
        level: { type: string, enum: [Beginner, Intermediate, Advanced] }

    Enrollment:
      type: object
      required: [id,courseId,studentId,enrollmentDate,status,createdAt]
      properties:
        id: { type: string, format: uuid }
        courseId: { type: string, format: uuid }
        studentId: { type: string, format: uuid }
        enrollmentDate: { type: string, format: date-time }
        status: { type: string, enum: [Active, Completed, Cancelled] }
        grade: { type: number, minimum: 0, maximum: 100, nullable: true }
        createdAt: { type: string, format: date-time }
    EnrollmentCreate:
      allOf:
        - $ref: "#/components/schemas/Enrollment"
      required: [courseId,studentId]
      properties:
        id: { readOnly: true }
        enrollmentDate: { readOnly: true }
        createdAt: { readOnly: true }
    EnrollmentUpdate:
      type: object
      properties:
        status: { type: string, enum: [Active, Completed, Cancelled] }
        grade: { type: number, minimum: 0, maximum: 100, nullable: true }

    # Paged results
    PagedCourse:
      type: object
      properties:
        items: { type: array, items: { $ref: "#/components/schemas/Course" } }
        page: { type: integer }
        size: { type: integer }
        total: { type: integer }
    PagedInstructor:
      type: object
      properties:
        items: { type: array, items: { $ref: "#/components/schemas/Instructor" } }
        page: { type: integer }
        size: { type: integer }
        total: { type: integer }
    PagedStudent:
      type: object
      properties:
        items: { type: array, items: { $ref: "#/components/schemas/Student" } }
        page: { type: integer }
        size: { type: integer }
        total: { type: integer }
    PagedEnrollment:
      type: object
      properties:
        items: { type: array, items: { $ref: "#/components/schemas/Enrollment" } }
        page: { type: integer }
        size: { type: integer }
        total: { type: integer }
```

*(Tu peux importer cet extrait dans Swagger UI/Redoc directement. Il couvre **toutes** les ressources + health, avec paramètres communs, schémas, et réponses.)*


<br/>
<br/>


<h2 id="sec-compat">8 – Compatibilité Phase 3 (ajout auth/rôles sans casser)</h2>

* **Mêmes URLs et schémas** qu’en Phase 1.
* Tu brancheras **JWT/Clerk/NextAuth** et des **guards** → seules nouveautés côté doc : **401 Unauthorized** (non connecté) et **403 Forbidden** (rôle insuffisant).
* Pas de changement des tableaux ci-dessus ; en Phase 3, complète juste la colonne “Erreurs” avec `401/403` et ajoute la section *SecuritySchemes* dans l’OpenAPI (ex. `bearerAuth: type: http, scheme: bearer`).









