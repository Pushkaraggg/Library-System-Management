# Library-System-Management
**--Task 1. Create a New Book Record -- "('978-1-60129-456-2', 'To Kill a Mockingbird', 'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.')"**
```sql
INSERT INTO Books
VALUES
('978-1-60129-456-2', 'To Kill a Mockingbird', 'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co')
```


**--Task 2: Update an Existing Member's Address**
```sql
UPDATE members
SET member_address = '125 Oak st'
WHERE member_id = 'C103'
```


**--Task 3: Delete a Record from the Issued Status Table -- Objective: Delete the record with issued_id = 'IS121' from the issued_status table.**
```sql
DELETE FROM issued_status
WHERE issued_id='IS121'
```


**--Task 4: Retrieve All Books Issued by a Specific Employee -- Objective: Select all books issued by the employee with emp_id = 'E101'.**
```sql
SELECT issued_book_name 
FROM issued_status
WHERE issued_emp_id='E101'
```


**--Task 5: List Members Who Have Issued More Than One Book**
```sql
SELECT issued_member_id, COUNT(issued_book_name) AS total_issued_books
FROM issued_status
GROUP BY 1
HAVING COUNT(issued_book_name) >'1'
ORDER BY 1
```


**--Task 6: Create Summary Tables: Used CTAS to generate new tables based on query results - each book and total book_issued_cnt**
```sql
CREATE VIEW total_book_issued_cnt
AS SELECT b.isbn,b.book_title,i.issued_book_isbn,i.issued_book_name
FROM issued_status AS i
INNER JOIN Books AS b
ON i.issued_book_isbn = b.isbn

SELECT isbn,book_title ,COUNT(*) AS book_issued_count
FROM total_book_issued_cnt
GROUP BY isbn, book_title
ORDER BY book_issued_count DESC
```


**--Task 7: Find Total Rental Income of issued Books by Category**
```sql
SELECT b.category,SUM(b.rental_price),COUNT(issued_book_name) AS cnt_issued_books
FROM Books AS b
INNER JOIN issued_status AS i
ON i.issued_book_isbn=b.isbn
GROUP BY b.category
```


**--Task 8: List Members Who Registered in the Last 180 Days**
```sql
SELECT member_id,member_name,reg_date, DATE_PART('month', reg_date) AS Month
FROM members
WHERE reg_date >= '2024-04-01'
```


**--Task 9: Retrieve the List of Books Not Yet Returned**
```sql
SELECT r.return_date,r.return_id,i.issued_date,i.issued_id,i.issued_book_name
FROM issued_status AS i
LEFT JOIN return_status AS r
ON i.issued_id=r.issued_id
WHERE return_date IS NULL
```


**--Task 10: List Employees with Their Branch Manager's Name and their branch details**
```sql
SELECT e1.emp_name,e2.emp_name AS manager,b.branch_address,b.contact_no
FROM employees AS e1
INNER JOIN Branch AS b
ON e1.branch_id=b.branch_id
INNER JOIN employees AS e2
ON e2.emp_id=b.manager_id
```


**--Task 11: Identify Members with Overdue Books**
**--Write a query to identify members who have overdue books (assume a 30-day return period). Display the member's_id, member's name, book title, issue date, and days overdue.**
```sql
SELECT i.issued_book_name,i.issued_date,
r.return_date,
m.member_id,m.member_name,
CASE 
WHEN (r.return_date-i.issued_date) IS NULL THEN (CURRENT_DATE - issued_date) 
WHEN (r.return_date-i.issued_date) IS NOT NULL THEN (r.return_date-i.issued_date) 
END AS overdue_days
FROM issued_status AS i
LEFT JOIN return_status AS r
ON i.issued_id=r.issued_id
LEFT JOIN members AS m
ON i.issued_member_id=m.member_id
ORDER BY r.return_date
```


**--Task 12: Update Book Status on Return**
**--Write a query to update the status of books in the books table to "Yes" when they are returned (based on entries in the return_status table).**
```sql
CREATE TABLE new_books_table AS 
SELECT i.issued_id,i.issued_book_name,i.issued_date,b.status,r.return_book_name,r.return_date
FROM return_status AS r
RIGHT JOIN issued_status AS i
ON r.issued_id=i.issued_id
INNER JOIN books AS b
ON i.issued_book_name=b.book_title

UPDATE new_books_table
SET status= 'No' 
WHERE return_date IS NULL
```


**--Task 13: Branch Performance Report**
**--Create a query that generates a performance report for each branch, showing the number of books issued, the number of books returned, and the total revenue generated from book rentals.**
```sql
SELECT e.branch_id,
COUNT(i.issued_id) AS total_book_issued,
COUNT(r.return_id) AS total_book_returned,
SUM(b.rental_price) AS total_revenue_generated
FROM return_status AS r
RIGHT JOIN issued_status AS i
ON r.issued_id=i.issued_id
INNER JOIN books AS b
ON i.issued_book_name=b.book_title
INNER JOIN employees AS e
ON e.emp_id=i.issued_emp_id
GROUP BY 1
ORDER BY 1
```

**--Task 14: CTAS: Create a Table of Active Members**
**--Use the CREATE TABLE AS (CTAS) statement to create a new table active_members containing members who have issued at least one book in the last 2 months.[ASSUMING April(04) to be Current Month to get data]**
```sql
CREATE TABLE active_members_since_april AS
SELECT m.member_id,m.member_name,i.issued_member_id,i.issued_book_name,i.issued_date
FROM members AS m
LEFT JOIN issued_status AS i
ON m.member_id=i.issued_member_id
WHERE issued_date >='2024-04-01' 

SELECT * FROM active_members_since_april
```


**--Task 15: Find Employees with the Most Book Issues Processed**
**--Write a query to find the top 3 employees who have processed the most book issues. Display the employee name, number of books processed, and their branch.**
```sql
SELECT  e.emp_name,b.branch_id,branch_address,COUNT(i.issued_id) AS total_book_issued
FROM employees AS e
INNER JOIN issued_status AS i
ON e.emp_id=i.issued_emp_id 
INNER JOIN branch AS b
ON e.branch_id=b.branch_id
GROUP BY 1,2,3
ORDER BY total_book_issued DESC
LIMIT 3;
```

