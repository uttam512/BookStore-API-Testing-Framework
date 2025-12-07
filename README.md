# 📘 BookStore API Testing Framework (Postman)

A complete Postman-based API Testing Framework designed to test CRUD operations of the public FakeREST BookStore API.

This project demonstrates professional API testing practices including:

- Environment variables  
- Chained requests using `bookId`  
- Automated JavaScript tests  
- JSON Schema validation  
- CRUD validation  
- Data-driven testing with CSV  
- API defect documentation (DELETE does not persist)

---

## 📌 Project Overview

This Postman collection automates full CRUD operations for the Books resource of the FakeREST API:

| Operation | Request Name |
|----------|---------------|
| Create   | `POST – Create Book` |
| Read     | `GET – All Books`, `GET – Book by ID`, `GET – Created Book` |
| Update   | `PUT – Update Book` |
| Delete   | `DELETE – Book` |
| Verify   | `GET – Verify Book Deleted` |

The project follows real-world testing patterns and demonstrates end-to-end validation using chained variables.

---

## 🌍 Environment Setup

**Environment Name:** `Bookstore Fake API`

| Variable | Value | Purpose |
|----------|--------|----------|
| `baseUrl` | `https://fakerestapi.azurewebsites.net` | Base API endpoint |
| `booksPath` | `/api/v1/Books` | Books resource path |
| `token` | *(empty)* | Placeholder for token-based APIs |
| `bookId` | *(empty, set automatically by POST)* | Used to chain GET/PUT/DELETE |

---

## 🔁 How `bookId` Works

The `bookId` is dynamically assigned during the POST request:

```js
pm.environment.set("bookId", body.id);
```

This sets up chaining for:

```
{{bookId}}
```

---

## 📦 Collection Requests (7 Requests)

The collection contains **7 structured API requests**.

---

### 1️⃣ GET – All Books

**URL:**
```text
{{baseUrl}}{{booksPath}}
```

**Purpose:** Retrieve all books and validate API structure.

**Key Tests:**
```js
pm.test("Status is 200", () => pm.response.to.have.status(200));
pm.test("Response is an array", () =>
    pm.expect(pm.response.json()).to.be.an("array")
);
pm.test("Response time < 2000ms", () =>
    pm.expect(pm.response.responseTime).to.be.below(2000)
);
```

---

### 2️⃣ GET – Book by ID

**URL:**
```text
{{baseUrl}}{{booksPath}}/{{bookId}}
```

**Purpose:** Fetch the newly created book using dynamic `bookId`.

**Key Tests:**
```js
const res = pm.response.json();

pm.test("Status is 200", () => pm.response.to.have.status(200));

pm.test("Book ID matches", () => {
    pm.expect(res.id).to.eql(parseInt(pm.environment.get("bookId")));
});
```

---

### 3️⃣ POST – Create Book

**URL:**
```text
{{baseUrl}}{{booksPath}}
```

**Body Example:**
```json
{
  "id": 111,
  "title": "Postman Framework Practice Book",
  "description": "Book created as part of BookStore API Testing project.",
  "pageCount": 250,
  "excerpt": "This is a sample book for API testing.",
  "publishDate": "2025-01-01T00:00:00.000Z"
}
```

**Purpose:** Create a new Book and store `bookId`.

**Key Tests:**
```js
pm.test("Status is 200 or 201", () => {
    pm.expect([200, 201]).to.include(pm.response.code);
});

const body = pm.response.json();

pm.test("Book ID present", () =>
    pm.expect(body).to.have.property("id")
);

// Save ID for chaining
pm.environment.set("bookId", body.id);
```

---

### 4️⃣ GET – Created Book

**URL:**
```text
{{baseUrl}}{{booksPath}}/{{bookId}}
```

**Purpose:** Validate that the created book exists.

**Key Test:**
```js
pm.test("Status is 200", () => pm.response.to.have.status(200));
```

---

### 5️⃣ PUT – Update Book

**URL:**
```text
{{baseUrl}}{{booksPath}}/{{bookId}}
```

**Purpose:** Update the created book details.

**Key Tests:**
```js
const res = pm.response.json();

pm.test("Status is 200", () => pm.response.to.have.status(200));

pm.test("Title updated", () =>
    pm.expect(res.title).to.include("Updated")
);
```

---

### 6️⃣ DELETE – Book

**URL:**
```text
{{baseUrl}}{{booksPath}}/{{bookId}}
```

**Purpose:** Delete the created book.

**Key Test:**
```js
pm.test("Status is 200 or 204", () =>
    pm.expect([200, 204]).to.include(pm.response.code)
);
```

---

### 7️⃣ GET – Verify Book Deleted

**URL:**
```text
{{baseUrl}}{{booksPath}}/{{bookId}}
```

**Purpose:** Validate that the deleted book no longer exists.

**Expected Behavior (Real APIs):**  
`404 Not Found`

**Actual Behavior (FakeREST Limitation):**  
`200 OK` — book still returned

**Intentional Failing Test:**
```js
pm.test("Book is deleted", () =>
    pm.expect(pm.response.code).to.be.oneOf([404, 400])
);
```

---

## ⚠️ Known API Limitation

The FakeREST API does **not** actually delete resources.

After:
```
DELETE /api/v1/Books/{id}
```

A follow-up request:
```
GET /api/v1/Books/{id}
```

Still returns:

- `200 OK`  
- Full book object  

This is an **API defect**, not a test issue.

---

## 🧪 Running the Full Test Suite (Collection Runner)

1. Open **Postman → Collections**  
2. Select **BookStore API Tests**  
3. Click **Run**  
4. Select environment: `Bookstore Fake API`  

### 🔄 First run — IMPORTANT

Uncheck these two requests:

- `GET – Book by ID`  
- `GET – Created Book`  

Because `bookId` is empty until POST runs once.

After POST runs successfully, run the full suite normally.

---
