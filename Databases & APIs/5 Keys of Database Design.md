If you're referring to the **"5 types of keys in databases"** (often called "5 keys of database design"), here are the essential ones you should learn:

---

### 🔑 1. **Primary Key**

* **Definition:** A column (or set of columns) that uniquely identifies each row in a table.
* **Example:** In a `Students` table, `student_id` can be a primary key.
* **Properties:**

  * Cannot be `NULL`
  * Must be unique
  * Only one primary key per table

---

### 🔑 2. **Candidate Key**

* **Definition:** A column (or set of columns) that could be a primary key because it uniquely identifies rows.
* **Example:** In a `Users` table, both `email` and `username` might be candidate keys.
* **Note:** One candidate key becomes the primary key, others are called **alternate keys**.

---

### 🔑 3. **Foreign Key**

* **Definition:** A column that creates a relationship between two tables by referencing the primary key in another table.
* **Example:** `order.customer_id` in an `Orders` table may reference `customer.id` in a `Customers` table.
* **Ensures referential integrity.**

---

### 🔑 4. **Super Key**

* **Definition:** Any combination of columns that uniquely identifies rows in a table.
* **Example:** `{student_id}`, `{student_id, name}` — both are super keys if `student_id` is unique.
* **Note:** All primary keys are super keys, but not all super keys are minimal.

---

### 🔑 5. **Composite Key**

* **Definition:** A primary key made up of **two or more columns**.
* **Example:** In a `CourseEnrollment` table, a composite key could be `{student_id, course_id}`.
* **Used when a single column is not enough to uniquely identify rows.**

---