# Library Management System - Python

## Project Overview

A desktop-based **Library Management System** built with Python and Tkinter. The application provides a graphical user interface (GUI) for managing books in a library, including adding, deleting, viewing, issuing, and returning books. Data is persisted in a MySQL database.

## Technology Stack

- **Programming Language:** Python 3.12+
- **GUI Framework:** Tkinter (built-in with Python)
- **Database:** MySQL 8.0+
- **Database Connector:** PyMySQL
- **Image Processing:** Pillow (PIL)
- **Architecture:** Modular Python with functional programming

## Project Structure

```
library-management-project-python/
├── main.py                  # Entry point - Main library window
├── AddBook.py              # Add new books to library
├── DeleteBook.py           # Delete books from library
├── ViewBooks.py            # View all books in library
├── IssueBook.py            # Issue books to students
├── ReturnBook.py           # Accept returned books
├── lib.jpg                 # Background image
└── .venv/                  # Virtual environment (optional)
```

## Database Schema

### Table: `books`
```sql
CREATE TABLE books (
    bid INT PRIMARY KEY AUTO_INCREMENT,
    title VARCHAR(100) NOT NULL,
    author VARCHAR(50) NOT NULL,
    status VARCHAR(20) NOT NULL DEFAULT 'avail'
);
```

### Table: `books_issued`
```sql
CREATE TABLE books_issued (
    bid INT,
    issuedto VARCHAR(100)
);
```

## Application Flow

### 1. **Main Menu (main.py)**
   - Displays the main library window
   - Shows 5 buttons for different operations
   - Background image and navigation interface

### 2. **Add Book (AddBook.py)**
   - User enters: Book ID, Title, Author, Status
   - Status options: "avail" or "issued"
   - Data is inserted into the `books` table
   - Shows success/error message

### 3. **View Books (ViewBooks.py)**
   - Displays all books in the library
   - Shows columns: Book ID, Title, Author, Status
   - Fetches data from `books` table

### 4. **Delete Book (DeleteBook.py)**
   - User enters Book ID to delete
   - Removes book from `books` table
   - Also removes from `books_issued` table if issued
   - Shows confirmation message

### 5. **Issue Book (IssueBook.py)**
   - User enters: Book ID, Student Name
   - Validates if book is available
   - Inserts record in `books_issued` table
   - Updates `books` table status to "issued"
   - Shows confirmation message

### 6. **Return Book (ReturnBook.py)**
   - User enters Book ID to return
   - Validates if book was issued
   - Removes from `books_issued` table
   - Updates `books` table status back to "avail"
   - Shows confirmation message

## Setup Instructions

### Prerequisites
- Python 3.12+ installed
- MySQL Server 8.0+ installed and running
- pip package manager

### Installation Steps

1. **Clone/Download the project**
   ```bash
   cd d:\LMS(PY)\library-management-project-python
   ```

2. **Create a virtual environment (optional but recommended)**
   ```bash
   python -m venv .venv
   .venv\Scripts\Activate.ps1  # On Windows PowerShell
   ```

3. **Install dependencies**
   ```bash
   pip install pillow pymysql cryptography
   ```

4. **Create the database and tables in MySQL**
   ```sql
   CREATE DATABASE librarydb;
   USE librarydb;

   CREATE TABLE books (
       bid INT PRIMARY KEY AUTO_INCREMENT,
       title VARCHAR(100) NOT NULL,
       author VARCHAR(50) NOT NULL,
       status VARCHAR(20) NOT NULL DEFAULT 'avail'
   );

   CREATE TABLE books_issued (
       bid INT,
       issuedto VARCHAR(100)
   );
   ```

5. **Configure database credentials**
   - Edit all `.py` files and update:
   ```python
   mypass = "your_mysql_password"
   mydatabase = "librarydb"
   ```

6. **Ensure `lib.jpg` is in the project folder**

7. **Run the application**
   ```bash
   python main.py
   ```

## Features

✅ Add new books to the library  
✅ Delete books from the library  
✅ View all books with details  
✅ Issue books to students  
✅ Accept returned books  
✅ Track book availability status  
✅ MySQL database persistence  
✅ Tkinter GUI interface  
✅ Error handling and validation  
✅ Success/Error message boxes  

## Deployment & Hosting Options

### **Option 1: Desktop Application (Current)**
- **Pros:** Works offline, no server needed, fast performance
- **Cons:** Must be installed on each machine
- **Best for:** Single library or local use

---

### **Option 2: Convert to Web Application (Recommended)**

#### Using Flask/Django with Tkinter replacement
```
Frontend: HTML/CSS/JavaScript (React, Vue, or vanilla)
Backend: Python Flask/Django
Database: MySQL
Hosting: Heroku, Render, PythonAnywhere, AWS, Azure, Google Cloud
```

**Steps:**
1. Replace Tkinter with Flask web framework
2. Create HTML templates for each page
3. Host on cloud platform
4. Access via web browser from any device

**Advantages:**
- Access from anywhere (browser-based)
- Multiple users simultaneously
- No installation required
- Easy to scale
- Remote access

---

### **Option 3: Use PythonAnywhere (Easiest)**

**Steps:**
1. Create Flask version of the app
2. Upload to [PythonAnywhere.com](https://www.pythonanywhere.com)
3. Configure MySQL database
4. Get a public URL
5. Access from any browser

**Cost:** Free tier available

**URL Example:** `https://yourusername.pythonanywhere.com`

---

### **Option 4: Docker + Cloud Deployment**

**Steps:**
1. Create Dockerfile for Python app
2. Use Docker Compose for MySQL + Flask
3. Deploy to:
   - **Docker Hub** - Container registry
   - **Heroku** - Simple deployment
   - **AWS EC2** - Full control
   - **Google Cloud Run** - Serverless
   - **DigitalOcean** - Affordable VPS

**Dockerfile Example:**
```dockerfile
FROM python:3.12
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
CMD ["python", "main.py"]
```

---

### **Option 5: Executable (.EXE) for Windows**

Convert to standalone Windows application:
```bash
pip install pyinstaller
pyinstaller --onefile --windowed main.py
```

**Pros:**
- No Python installation needed
- Can distribute to users
- Looks like normal Windows app

**Cons:**
- Database must be accessible (local or networked MySQL)
- Larger file size (~100MB+)

---

### **Recommendation for Production**

**Best Approach: Flask Web Application + PythonAnywhere/Heroku**

1. **Rewrite using Flask** (minimal changes)
2. **Host on PythonAnywhere** (easiest) or **Heroku** (more scalable)
3. **Access from any browser** (mobile, tablet, desktop)
4. **No installation** required for end-users
5. **Automatic backups** and updates

---

## Code Quality Improvements

### Security Issues (Current)
⚠️ Hardcoded database credentials  
⚠️ SQL injection vulnerable (string concatenation)  
⚠️ No user authentication  

### Improvements Needed
✓ Use environment variables for credentials  
✓ Use parameterized queries (already done partially)  
✓ Add user login system  
✓ Add role-based access (Admin, Librarian, Student)  
✓ Add logging  
✓ Add input validation  

---

## Troubleshooting

### Error: "ModuleNotFoundError: No module named 'PIL'"
```bash
pip install pillow
```

### Error: "AttributeError: module 'PIL.Image' has no attribute 'ANTIALIAS'"
Use `Image.LANCZOS` instead (newer Pillow versions)

### Error: "Lost connection to MySQL server"
- Restart MySQL Server
- Check database credentials
- Verify MySQL is running

### Error: "Unknown column 'status' in field list"
- Run ALTER TABLE command to add status column
- Verify database schema matches

---

## Future Enhancements

1. **User Authentication** - Login system for librarians
2. **Student Database** - Track student records
3. **Book Categories** - Organize books by category/genre
4. **Fine System** - Calculate overdue book fines
5. **Search & Filter** - Advanced book search
6. **Reports** - Generate library statistics
7. **Email Notifications** - Notify students of due dates
8. **Mobile App** - Native iOS/Android app
9. **Barcode Scanner** - Integration with barcode readers
10. **Analytics Dashboard** - Usage statistics and insights

---

## Author & License

**Project Type:** Educational  
**License:** Open Source (MIT)  
**Version:** 1.0  

---

## Contact & Support

For issues or questions, refer to the code comments or modify the database credentials in each `.py` file.

---

## Summary

This Library Management System is a **desktop application** best suited for small-to-medium libraries. For large-scale or remote access needs, **migrate to a web-based Flask/Django application** and host on cloud platforms like **PythonAnywhere, Heroku, or AWS** for better scalability, security, and accessibility.
