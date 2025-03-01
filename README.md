# Library Management System using SQL Project --P2

## Project Overview

**Project Title**: Library Management System  
**Level**: Intermediate  
**Database**: `library_managment_system'`

This project demonstrates the implementation of a Library Management System using SQL. It includes creating and managing tables, performing CRUD operations, and executing advanced SQL queries. The goal is to showcase skills in database design, manipulation, and querying.


## Objectives

1. **Set up the Library Management System Database**: Create and populate the database with tables for branches, employees, members, books, issued status, and return status.
2. **CRUD Operations**: Perform Create, Read, Update, and Delete operations on the data.
3. **CTAS (Create Table As Select)**: Utilize CTAS to create new tables based on query results.
4. **Advanced SQL Queries**: Develop complex queries to analyze and retrieve specific data.

## Project Structure

### 1. Database Setup
- **Database Creation**: Created a database named `library_managment_system`.
- **Table Creation**: Created tables for branches, employees, members, books, issued status, and return status. Each table includes relevant columns and relationships.

```sql
-- BRANCH TABLE
CREATE TABLE BRANCH
(
	branch_id VARCHAR(20) PRIMARY KEY,
	manager_id VARCHAR(20),
	branch_address VARCHAR(50),
	contact_no VARCHAR(15)
);
SELECT* FROM employees;
--EMPLOYEE TABLE
CREATE TABLE employees
(
	emp_id VARCHAR(10) PRIMARY KEY,
	emp_name VARCHAR (25),
	position VARCHAR(20),
	salary INT,
	branch_id VARCHAR(20)     --FK
);
ALTER TABLE employees
ALTER COLUMN salary TYPE FLOAT;

-- BOOKS TABLE
CREATE TABLE books 
(
	isbn VARCHAR(20) PRIMARY KEY,
	book_title VARCHAR(75),
	category VARCHAR(10),
	rental_price FLOAT ,
	status VARCHAR(15),
	author VARCHAR(35),
	publisher VARCHAR(55)
	
);

ALTER TABLE books 
ALTER COLUMN category TYPE VARCHAR(20);

--MEMBERS TABLE
CREATE TABLE members
(
	member_id VARCHAR(10) PRIMARY KEY,
	member_name VARCHAR(20),
	member_address  VARCHAR(70),
	reg_date DATE

);
SELECT * FROM return_status;
-- ISSUED STATUS TABLE
CREATE TABLE issued_status 
(
	issued_id VARCHAR(10) PRIMARY KEY,
	issued_member_id VARCHAR(10),     --FK
	issued_book_name VARCHAR(75),
	issued_date DATE,
	issued_book_isbn VARCHAR(25),   --FK
	issued_emp_id VARCHAR (10)     --FK
);

-- RETURN STATUS TABLE

CREATE TABLE return_status
(
	return_id VARCHAR(15) PRIMARY KEY,
	issued_id VARCHAR(20),            -- FK
	return_book_name VARCHAR(75),
	return_date DATE,
	return_book_isbn VARCHAR(20)
	
);

-- FOREIGN KEYS
ALTER TABLE issued_status
ADD CONSTRAINT fk_members 
FOREIGN KEY(issued_member_id)
REFERENCES members(member_id);

ALTER TABLE issued_status
ADD CONSTRAINT fk_books
FOREIGN KEY (issued_book_isbn)
REFERENCES books(isbn);

ALTER TABLE issued_status
ADD CONSTRAINT fk_employees
FOREIGN KEY (issued_emp_id)
REFERENCES employees(emp_id);

ALTER TABLE employees
ADD CONSTRAINT fk_branch
FOREIGN KEY (branch_id)
REFERENCES branch(branch_id);

ALTER TABLE return_status 
ADD CONSTRAINT fk_issued_status
FOREIGN KEY (issued_id)
REFERENCES issued_status(issued_id);

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
UPDATE members SET 
member_address='125 East' 
WHERE member_id='C103';
```

**Task 3: Delete a Record from the Issued Status Table**
-- Objective: Delete the record with issued_id = 'IS121' from the issued_status table.

```sql
SELECT * FROM issued_status ;
DELETE FROM 
issued_status 
WHERE issued_id ='IS121';
```

**Task 4: Retrieve All Books Issued by a Specific Employee**
-- Objective: Select all books issued by the employee with emp_id = 'E101'.
```sql
SELECT * FROM issued_status
WHERE issued_emp_id= 'E101';
```


**Task 5: List Members Who Have Issued More Than One Book**
-- Objective: Use GROUP BY to find members who have issued more than one book.

```sql
SELECT issued_member_id,count(*) 
FROM issued_status 
GROUP BY 1
HAVING count(*) > 1;
```

### 3. CTAS (Create Table As Select)

- **Task 6: Create Summary Tables**: Used CTAS to generate new tables based on query results - each book and total book_issued_cnt**

```sql
CREATE TABLE book_counts
 AS
SELECT
	b.isbn,
	b.book_title,
	count(ist.issued_id)  AS no_issued
FROM 
books as b
JOIN 
issued_status as ist
ON 
ist.issued_book_isbn = b.isbn
GROUP BY 1,2

select * from book_counts;
```


### 4. Data Analysis & Findings

The following SQL queries were used to address specific questions:

Task 7. **Retrieve All Books in a Specific Category**:

```sql
SELECT * FROM books
WHERE category ='History';
```

8. **Task 8: Find Total Rental Income by Category**:

```sql
SELECT 
	b.category,
	SUM(rental_price),
	COUNT(*)
FROM books AS b
JOIN 
issued_status as ist
ON
ist.issued_book_isbn = b.isbn
GROUP BY 1;
```

9. **List Members Who Registered in the Last 180 Days**:
```sql
SELECT* FROM members
WHERE reg_date >= CURRENT_DATE - INTERVAL '180 DAYS'

INSERT INTO members 
VALUES 
('C120','pavan','144 st','2025-02-15'),
('c121','maxy','211 main st','2024-12-20');
```

10. **List Employees with Their Branch Manager's Name and their branch details**:

```sql
SELECT 
e1.emp_id,
e1.emp_name,
e1.position,
e1.salary,
b.*,
e2.emp_name as manager
FROM 
	employees AS e1
JOIN 
	branch AS b
ON  e1.branch_id = b.branch_id
JOIN 
  employees AS e2
ON
	e2.emp_id = b.manager_id
```

Task 11. **Create a Table of Books with Rental Price Above a Certain Threshold**:
```sql
CREATE TABLE books_price_greater_than_7
AS
select * from books
where rental_price >7

SELECT * FROM books_price_greater_than_7
```

Task 12: **Retrieve the List of Books Not Yet Returned**
```sql
select 
ist.issued_book_name
from issued_status AS ist
LEFT JOIN
return_status AS rs
ON 
ist.issued_id = rs.issued_id
WHERE 
rs.return_id Is null
```

## Advanced SQL Operations

**Task 13: Identify Members with Overdue Books**  
Write a query to identify members who have overdue books (assume a 30-day return period). Display the member's_id, member's name, book title, issue date, and days overdue.

```sql
select 
ist.issued_member_id,
m.member_name,
bk.book_title,
ist.issued_date,
CURRENT_DATE -ist.issued_date as overdue_days
from 
issued_status as ist
JOIN
 members as m
ON
	m.member_id = ist.issued_member_id
JOIN 
	books as bk
ON
	bk.isbn = ist.issued_book_isbn
LEFT JOIN 
return_status as rs
ON
rs.issued_id = ist.issued_id
WHERE 
	rs.return_date is null
	AND
	(CURRENT_DATE -ist.issued_date) > 30 
ORDER BY 1

```


**Task 14: Update Book Status on Return**  
Write a query to update the status of books in the books table to "Yes" when they are returned (based on entries in the return_status table).


```sql

CREATE OR REPLACE PROCEDURE add_return_records(p_return_id VARCHAR (10),p_issued_id VARCHAR(10),p_book_quality VARCHAR(15))
LANGUAGE plpgsql
AS $$

DECLARE
	v_isbn VARCHAR (50);
	v_book_name varchar (80);

BEGIN
	-- all your logic and code
	--inserting into returns based on users input
	
	INSERT INTO return_status(return_id,issued_id,return_date,book_quality)
	VALUES
	(p_return_id,p_issued_id,CURRENT_DATE,p_book_quality);
	SELECT 
	issued_book_isbn,
	issued_book_name
	INTO
	v_isbn,
	v_book_name
	FROM issued_status
	WHERE issued_id =p_issued_id;

	UPDATE books
	SET status ='yes'
	WHERE isbn = v_isbn;

	RAISE NOTICE ' THANK YOU FOR RETURNING THE BOOK %'  ,v_book_name;

END;
$$

CALL add_return_records()

CALL  add_return_records()

-- testing functions add_ return _records()

issued_id = IS136
ISSUED_BOOK_ISBN='978-0-7432-7357-1'

select* from issued_Status


select * from return_status 
where issued_id ='IS136'

select * from books
where isbn ='978-0-7432-7357-1'

update books 
set status = 'no'
where isbn='978-0-7432-7357-1'

select * from return_status 
where return_book_isbn ='978-0-7432-7357-1'


CALL add_return_records('RS135','IS151','good');

CALL add_return_records ('RS136','IS136','Damaged');
```




**Task 15: Branch Performance Report**  
Create a query that generates a performance report for each branch, showing the number of books issued, the number of books returned, and the total revenue generated from book rentals.

```sql


CREATE TABLE branch_reports
AS
	SELECT 
		b.branch_id,
		b.manager_id,
		count(ist.issued_id) as number_books_issued,
		count(rs.return_id) as number_books_returned,
		sum(bk.rental_price) as total_revenue
	FROM issued_status AS ist
	JOIN employees AS e
	ON 
	e.emp_id = ist.issued_emp_id
	JOIN
	branch as b
	ON 
	e.branch_id = b.branch_id
	 LEFT JOIN 
	return_status as rs
	ON 
	rs.issued_id =ist.issued_id
	JOIN 
	books as bk
	ON
	ist.issued_book_isbn = bk.isbn
	GROUP BY 1,2;

SELECT * FROM branch_reports;
```

**Task 16: CTAS: Create a Table of Active Members**  
Use the CREATE TABLE AS (CTAS) statement to create a new table active_members containing members who have issued at least one book in the last 2 months.

```sql

CREATE TABLE active_members
AS
SELECT *  FROM members
WHERE member_id IN (	SELECT issued_member_id
                        FROM issued_status
                         WHERE 
                         issued_date >= CURRENT_DATE - INTERVAL'2 MONTH' 
                    )
;
SELECT * FROM  active_members;


```


**Task 17: Find Employees with the Most Book Issues Processed**  
Write a query to find the top 3 employees who have processed the most book issues. Display the employee name, number of books processed, and their branch.

```sql


select 
	e.emp_name,
	b.*,
	count(ist.issued_id) as no_books_issued 

FROM 
	issued_status AS ist
JOIN
	employees AS e
ON
	e.emp_id = ist.issued_emp_id
JOIN 
	 branch as b
ON
	e.branch_id = b.branch_id
GROUP BY 1,2

```

**Task 18: Identify Members Issuing High-Risk Books**  
Write a query to identify members who have issued books more than twice with the status "damaged" in the books table. Display the member name, book title, and the number of times they've issued damaged books.    


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

CREATE OR REPLACE PROCEDURE  book_issue_status (p_issued_id VARCHAR(10),p_issued_member_id VARCHAR(10),p_issued_book_isbn VARCHAR(25),
p_issued_emp_id VARCHAR(10))
LANGUAGE plpgsql
AS $$

DECLARE
	--ALL THE VARIABLE DECLARATION
		v_status VARCHAR(10);

BEGIN
	--	ALL THE CODE HERE
	-- CHECKING THE STATUS 
	SELECT status
	INTO
		v_status
	FROM books 
	WHERE isbn = P_issued_book_isbn;

	IF v_status ='yes' THEN
			INSERT INTO issued_status (issued_id,issued_member_id,issued_date,issued_book_isbn,issued_emp_id)
			VALUES
				(p_issued_id ,p_issued_member_id,CURRENT_DATE,p_issued_book_isbn,p_issued_emp_id);
				UPDATE books
				SET status ='no'
				WHERE isbn = p_issued_book_isbn;
				RAISE NOTICE 'Book records added succesfully for book isbn :% ', p_issued_book_isbn;
	
	ELSE
	
		RAISE NOTICE 'sorry to inform you the book you have requested is currently unavailble : %' ,p_issued_book_isbn; 
	END IF;
	
END ;
$$



-- Testing The function
SELECT * FROM books;
--978-0-330-25864-8 --yes --IS155 --E105
--978-0-375-41398-8--no --IS156 --E106

SELECT * FROM issued_status;


CALL book_issue_status('IS1155','C108','978-0-330-25864-8','E105');

CALL book_issue_status('IS156','C107','978-0-375-41398-8','E106');

SELECT * FROM books
WHERE isbn = '978-0-375-41398-8'

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



## Reports

- **Database Schema**: Detailed table structures and relationships.
- **Data Analysis**: Insights into book categories, employee salaries, member registration trends, and issued books.
- **Summary Reports**: Aggregated data on high-demand books and employee performance.

## Conclusion

This project demonstrates the application of SQL skills in creating and managing a library management system. It includes database setup, data manipulation, and advanced querying, providing a solid foundation for data management and analysis.


2. **Set Up the Database**: Execute the SQL scripts in the `database_setup.sql` file to create and populate the database.
3. **Run the Queries**: Use the SQL queries in the `analysis_queries.sql` file to perform the analysis.
4. **Explore and Modify**: Customize the queries as needed to explore different aspects of the data or answer additional questions.


Thank you for your interest in this project!
