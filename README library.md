# Library Management System using SQL Project --P2

## Project Overview

**Project Title**: Library Management System  
**Level**: Intermediate  
**Database**: `library_db`

This project demonstrates the implementation of a Library Management System using SQL. It includes creating and managing tables, performing CRUD operations, and executing advanced SQL queries. The goal is to showcase skills in database design, manipulation, and querying.

![library](https://github.com/user-attachments/assets/06986f2e-fed1-4a67-bab2-74ed2f131146)


## Objectives

1. **Set up the Library Management System Database**: Create and populate the database with tables for branches, employees, members, books, issued status, and return status.
2. **CRUD Operations**: Perform Create, Read, Update, and Delete operations on the data.
3. **CTAS (Create Table As Select)**: Utilize CTAS to create new tables based on query results.
4. **Advanced SQL Queries**: Develop complex queries to analyze and retrieve specific data.

## Project Structure

### 1. Database Setup
**ERD**

<img width="966" height="776" alt="Screenshot 2025-09-01 163306" src="https://github.com/user-attachments/assets/0b9dd067-a8de-423b-a26b-afe910197ddb" />


- **Database Creation**: Created a database named `library_db`.
- **Table Creation**: Created tables for branches, employees, members, books, issued status, and return status. Each table includes relevant columns and relationships.

```sql
CREATE DATABASE library_db;

DROP TABLE IF EXISTS branch;
CREATE TABLE branch(
	branch_id VARCHAR(10) PRIMARY KEY,
	manager_id VARCHAR(10),
	branch_address VARCHAR(55),
	contact_no VARCHAR(10)
);

DROP TABLE IF EXISTS employees;
CREATE TABLE employees(
	emp_id VARCHAR(10) PRIMARY KEY,
	emp_name VARCHAR(25),
	position VARCHAR(15),
	salary INT,
	branch_id VARCHAR(10)
);

DROP TABLE IF EXISTS issued_status;
CREATE TABLE issued_status(
	issued_id VARCHAR(10) PRIMARY KEY,
	issued_member_id VARCHAR(10),
	issued_book_name VARCHAR(65),
	issued_date DATE,
	issued_book_isbn VARCHAR(25),
	issued_emp_id VARCHAR(10)
);

DROP TABLE IF EXISTS book;
CREATE TABLE books(
	isbn VARCHAR(25) PRIMARY KEY,
	book_title VARCHAR(65),
	category VARCHAR(25),
	rental_price FLOAT,
	status VARCHAR(5),
	author VARCHAR(15),
	publisher VARCHAR(25)
);

DROP TABLE IF EXISTS return_status;
CREATE TABLE return_status(
	return_id VARCHAR(10) PRIMARY KEY,
	issued_id VARCHAR(10),
	return_book_name VARCHAR(65),
	return_date DATE,
	return_book_isbn VARCHAR(25),
	book_quality VARCHAR(10)
);

DROP TABLE IF EXISTS members;
CREATE TABLE members(
	member_id VARCHAR(10) PRIMARY KEY,
	member_name VARCHAR(15),
	member_address VARCHAR(20),
	reg_date DATE
); 

```

### 2. CRUD Operations

- **Create**: Inserted sample records into the `books` table.
- **Read**: Retrieved and displayed data from various tables.
- **Update**: Updated records in the `employees` table.
- **Delete**: Removed records from the `members` table as needed.

**Task 1. Create a New Book Record**
-- "978-1-60129-456-2', 'To Kill a Mockingbird', 'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.')"

```sql
INSERT INTO books(isbn, book_title, category, rental_price, status, author, publisher)
VALUES('978-1-60129-456-2', 'To Kill a Mockingbird', 'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.');
SELECT * FROM books;
```
**Task 2: Update an Existing Member's Address**

```sql
UPDATE members 
SET member_address = 'Dholpur, Rajasthan'
WHERE member_id = 'C101';
```

**Task 3: Delete a Record from the Issued Status Table**
-- Objective: Delete the record with issued_id = 'IS121' from the issued_status table.

```sql
DELETE FROM issued_status
WHERE   issued_id =   'IS121';
```

**Task 4: Retrieve All Books Issued by a Specific Employee**
-- Objective: Select all books issued by the employee with emp_id = 'E101'.
```sql
SELECT * FROM issued_status
WHERE issued_emp_id = 'E101'
```


**Task 5: List Members Who Have Issued More Than One Book**
-- Objective: Use GROUP BY to find members who have issued more than one book.

```sql
SELECT issued_emp_id FROM issued_status
GROUP BY issued_emp_id
HAVING COUNT(issued_id) > 1;
```

### 3. CTAS (Create Table As Select)

- **Task 6: Create Summary Tables**: Used CTAS to generate new tables based on query results - each book and total book_issued_cnt**

```sql
CREATE TABLE book_status_count
AS 
SELECT 
   b.isbn ,
   b.book_title , 
   COUNT(ist.issued_id)
FROM issued_status AS ist
JOIN books AS b
ON 
ist.issued_book_isbn = b.isbn
GROUP BY b.isbn,b.book_title;

SELECT*FROM book_status_count;
```


### 4. Data Analysis & Findings

The following SQL queries were used to address specific questions:

Task 7. **Retrieve All Books in a Specific Category**:

```sql
SELECT * FROM books
WHERE category = 'History';
```

8. **Task 8: Find Total Rental Income by Category**:

```sql
SELECT 
   books.category ,
   SUM(books.rental_price) AS rental_income ,
   COUNT(*)
FROM issued_status
JOIN books
ON
issued_status.issued_book_isbn = books.isbn
GROUP BY 1;
```

9. **List Members Who Registered in the Last 180 Days**:
```sql
SELECT * FROM members
WHERE reg_date >= CURRENT_DATE - INTERVAL '180 days';
```

10. **List Employees with Their Branch Manager's Name and their branch details**:

```sql
SELECT 
   emp.emp_id,
   emp.emp_name AS manager_name,
   emp.position,
   emp.salary,
   branch.*
FROM branch
JOIN employees AS emp
ON 
branch.branch_id = emp.branch_id;
```

Task 11. **Create a Table of Books with Rental Price Above a Certain Threshold**:
```sql
CREATE TABLE expensive_books AS
SELECT * FROM books
WHERE rental_price > 7.00;
```

Task 12: **Retrieve the List of Books Not Yet Returned**
```sql
SELECT * FROM issued_status as ist
LEFT JOIN
return_status as rs
ON rs.issued_id = ist.issued_id
WHERE rs.return_id IS NULL;
```

## Advanced SQL Operations

**Task 13: Identify Members with Overdue Books**  
Write a query to identify members who have overdue books (assume a 30-day return period). Display the member's_id, member's name, book title, issue date, and days overdue.

```sql
SELECT m.member_id,
		m.member_name,
		ist.issued_book_name,
		ist.issued_date,
		(CURRENT_DATE - ist.issued_date) AS overdue_days
FROM members AS m
JOIN issued_status AS ist
ON 
m.member_id = ist.issued_member_id
FULL JOIN return_status AS rst
ON 
rst.issued_id = ist.issued_id
WHERE rst.return_date IS NULL
AND 
(CURRENT_DATE - ist.issued_date) >30
ORDER BY 1;

```


**Task 14: Update Book Status on Return**  
Write a query to update the status of books in the books table to "Yes" when they are returned (based on entries in the return_status table).


```sql

CREATE OR REPLACE PROCEDURE add_return_records(p_return_id VARCHAR(10),p_issued_id VARCHAR(10))
LANGUAGE plpgsql
AS $$

DECLARE
	v_isbn VARCHAR(50);
	v_book_title VARCHAR(65);
BEGIN
	--inserting into returns based on users input
	INSERT INTO return_status(return_id,issued_id,return_date)
	VALUES
	(p_return_id,p_issued_id,CURRENT_DATE); 

	SELECT issued_book_isbn,issued_book_name INTO v_isbn , v_book_title
	FROM issued_status
	WHERE issued_id = p_issued_id;

	UPDATE books
	SET status = 'yes'
	WHERE isbn = v_isbn;

	RAISE NOTICE 'THANK YOU FOR RETURNING THE BOOK : %' ,v_book_title;
END;
$$
--TESTING
SELECT * FROM issued_status
SELECT * FROM return_status
WHERE issued_id = 'IS135';

SELECT ist.*,rs.return_date,rs.return_id FROM issued_status AS ist
LEFT JOIN return_status AS rs
ON 
ist.issued_id = rs.issued_id
WHERE ist.issued_id = 'IS134';

CALL add_return_records('RS150','IS135');
CALL add_return_records('RS151','IS134');

```




**Task 15: Branch Performance Report**  
Create a query that generates a performance report for each branch, showing the number of books issued, the number of books returned, and the total revenue generated from book rentals.

```sql
SELECT * FROM branch;
SELECT * FROM books;
SELECT * FROM return_status;
SELECT* FROM issued_status;


CREATE TABLE branch_report 
AS
SELECT
            br.branch_id,
            br.manager_id, 
            COUNT(ist.issued_id) AS number_of_book_issued,
            COUNT(rs.return_id) AS number_of_book_returned,
            SUM(bk.rental_price) AS total_revenue
FROM issued_status AS ist
JOIN employees as emp
ON
ist.issued_emp_id = emp.emp_id
JOIN branch AS br
ON
emp.branch_id = br.branch_id
JOIN books AS bk
ON
ist.issued_book_isbn = bk.isbn
LEFT JOIN return_status AS rs
ON
ist.issued_id = rs.issued_id
GROUP BY 1,2;

SELECT * FROM branch_report;
```

**Task 16: CTAS: Create a Table of Active Members**  
Use the CREATE TABLE AS (CTAS) statement to create a new table active_members containing members who have issued at least one book in the last 2 months.

```sql

CREATE TABLE active_members AS
SELECT * FROM members
WHERE member_id 
	IN(
		SELECT DISTINCT(issued_member_id) 
		FROM issued_status
		WHERE issued_date >= CURRENT_DATE - INTERVAL '2 months'
		);

SELECT * FROM active_members;

```


**Task 17: Find Employees with the Most Book Issues Processed**  
Write a query to find the top 3 employees who have processed the most book issues. Display the employee name, number of books processed, and their branch.

```sql
SELECT emp.emp_name,
		br.*,
		COUNT(ist.issued_id) AS book_issued
FROM issued_status AS ist
JOIN employees AS emp
ON
emp.emp_id = ist.issued_emp_id
JOIN branch AS br
ON
br.branch_id = emp.branch_id
GROUP BY 1,2;
```

**Task 18: Identify Members Issuing High-Risk Books**  
Write a query to identify members who have issued books more than twice with the status "damaged" in the books table. Display the member name, book title, and the number of times they've issued damaged books.    

```sql

SELECT m.member_id , m.member_name ,COUNT(ist.issued_id)
FROM members AS m
JOIN issued_status AS ist
ON
ist.issued_member_id = m.member_id
JOIN books as bk
ON
ist.issued_book_isbn = bk.isbn
JOIN return_status AS rs
ON
ist.issued_id = rs.issued_id
WHERE rs.book_quality = 'Damaged'
GROUP BY m.member_id
HAVING COUNT(ist.issued_id)>=2;
```

**Task 19: Stored Procedure**
Objective:
Create a stored procedure to manage the status of books in a library system.
Description:
Write a stored procedure that updates the status of a book in the library based on its issuance. The procedure should function as follows:
The stored procedure should take the book_id as an input parameter.
The procedure should first check if the book is available (status = 'yes').
If the book is available, it should be issued, and the status in the books table should be updated to 'no'.
If the book is not available (status = 'no'), the procedure should return an error message indicating that the book is currently not available.

```sql

SELECT * FROM books
SELECT * FROM issued_status

CREATE OR REPLACE PROCEDURE issue_book(p_issued_id VARCHAR(10),p_issued_member_id VARCHAR(10),p_issued_book_name VARCHAR(65),p_issued_book_isbn VARCHAR(25),p_issued_emp_id VARCHAR(10))
LANGUAGE plpgsql
AS $$

DECLARE

	v_status VARCHAR(5);
	
BEGIN
	SELECT status INTO v_status
	FROM books
	WHERE isbn = p_issued_book_isbn;

	IF v_status = 'yes' THEN
		INSERT INTO issued_status(issued_id,issued_member_id,issued_book_name,issued_date,issued_book_isbn,issued_emp_id)
		VALUES
		(p_issued_id , p_issued_member_id , p_issued_book_name , CURRENT_DATE , p_issued_book_isbn , p_issued_emp_id);
	
		UPDATE books
		SET status = 'no'
		WHERE isbn = p_issued_book_isbn;

		RAISE NOTICE 'Book records added successfully for isbn : %',p_issued_book_isbn;
	
	ELSE
		RAISE NOTICE 'Sorry to inform you have requested an unavailable book_isbn %' , p_issued_book_isbn;
	END IF;

END;
$$
--testing issue_book with add_return_book_record
CALL issue_book('IS334','C106','Animal Farm','978-0-330-25864-8','E104');
CALL add_return_records('R15','IS334');

```



**Task 20: Create Table As Select (CTAS)**
Objective: Create a CTAS (Create Table As Select) query to identify overdue books and calculate fines.

Description: Write a CTAS query to create a new table that lists each member and the books they have issued but not returned within 30 days. The table should include:
    The number of overdue books.
    The total fines, with each day's fine calculated at $0.50.
    The number of books issued by each member.
    The resulting table should show:
    Member ID
    Number of overdue books
    Total fines


```sql

--Sample table just for checking
SELECT m.member_id,
		m.member_name ,
		ist.issued_book_name,
		ist.issued_date,
		rs.return_date, 
		(return_date-issued_date) AS days_of_return,
		((return_date-issued_date)*0.50) AS total_fine
		FROM members AS m
JOIN issued_status as ist
ON
m.member_id = ist.issued_member_id
JOIN return_status AS rs
ON ist.issued_id = rs.issued_id

--CTAS OF ABOVE TABLE ONLY WITH member_id,member_name, numbers_of_overdue_books,total_fine
CREATE TABLE due_fine
AS
SELECT
   m.member_id,
   m.member_name ,
   COUNT(ist.issued_id) AS numbers_of_overdue_books,
   SUM(((return_date-issued_date)*0.50)) AS total_fine
FROM members AS m
JOIN issued_status as ist
ON
m.member_id = ist.issued_member_id
JOIN return_status AS rs
ON
ist.issued_id = rs.issued_id
WHERE (rs.return_date-ist.issued_date) > 30
GROUP BY 1;

SELECT * FROM due_fine;

```    



## Reports

- **Database Schema**: Detailed table structures and relationships.
- **Data Analysis**: Insights into book categories, employee salaries, member registration trends, and issued books.
- **Summary Reports**: Aggregated data on high-demand books and employee performance.

## Conclusion

This project demonstrates the application of SQL skills in creating and managing a library management system. It includes database setup, data manipulation, and advanced querying, providing a solid foundation for data management and analysis.

## How to Use

1. **Clone the Repository**: Clone this repository to your local machine.
   ```sh
   git clone https://github.com/najirh/Library-System-Management---P2.git
   ```

2. **Set Up the Database**: Execute the SQL scripts in the `project2-Library_Management.sql` file to create and populate the database.
3. **Explore and Modify**: Customize the queries as needed to explore different aspects of the data or answer additional questions.

![LinkedIN:-](https://www.linkedin.com/in/prashant-jha-681a912b4/)
