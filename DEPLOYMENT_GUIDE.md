# Library Management System - Deployment Guide

## Complete Step-by-Step Instructions

---

## **METHOD 1: Deploy on PythonAnywhere (EASIEST - Recommended)**

**Why PythonAnywhere?**
- Free tier available
- No installation needed on user's machine
- Accessible from any browser
- Automatic SSL/HTTPS
- MySQL database included
- Easy to manage

### **Step 1: Prepare the Flask Application**

First, convert the Tkinter app to a Flask web app:

1. Create a new file: `app.py`

```python
from flask import Flask, render_template, request, jsonify
import pymysql
from datetime import datetime

app = Flask(__name__)

# Database Configuration
db_config = {
    'host': 'localhost',
    'user': 'root',
    'password': 'your_password',  # Change this
    'database': 'librarydb',
    'charset': 'utf8mb4',
    'cursorclass': pymysql.cursors.DictCursor
}

def get_db_connection():
    return pymysql.connect(**db_config)

@app.route('/')
def index():
    return render_template('index.html')

@app.route('/api/books', methods=['GET'])
def get_books():
    conn = get_db_connection()
    cursor = conn.cursor()
    cursor.execute('SELECT bid, title, author, status FROM books')
    books = cursor.fetchall()
    cursor.close()
    conn.close()
    return jsonify(books)

@app.route('/api/books/add', methods=['POST'])
def add_book():
    data = request.json
    try:
        conn = get_db_connection()
        cursor = conn.cursor()
        cursor.execute(
            'INSERT INTO books (title, author, status) VALUES (%s, %s, %s)',
            (data['title'], data['author'], data['status'])
        )
        conn.commit()
        cursor.close()
        conn.close()
        return jsonify({'status': 'success', 'message': 'Book added successfully'})
    except Exception as e:
        return jsonify({'status': 'error', 'message': str(e)}), 400

@app.route('/api/books/delete/<int:bid>', methods=['DELETE'])
def delete_book(bid):
    try:
        conn = get_db_connection()
        cursor = conn.cursor()
        cursor.execute('DELETE FROM books_issued WHERE bid = %s', (bid,))
        cursor.execute('DELETE FROM books WHERE bid = %s', (bid,))
        conn.commit()
        cursor.close()
        conn.close()
        return jsonify({'status': 'success', 'message': 'Book deleted successfully'})
    except Exception as e:
        return jsonify({'status': 'error', 'message': str(e)}), 400

@app.route('/api/books/issue', methods=['POST'])
def issue_book():
    data = request.json
    try:
        conn = get_db_connection()
        cursor = conn.cursor()
        # Check if book is available
        cursor.execute('SELECT status FROM books WHERE bid = %s', (data['bid'],))
        result = cursor.fetchone()
        if not result or result['status'] != 'avail':
            return jsonify({'status': 'error', 'message': 'Book not available'}), 400
        
        # Issue the book
        cursor.execute('INSERT INTO books_issued (bid, issuedto) VALUES (%s, %s)', 
                      (data['bid'], data['issuedto']))
        cursor.execute('UPDATE books SET status = %s WHERE bid = %s', ('issued', data['bid']))
        conn.commit()
        cursor.close()
        conn.close()
        return jsonify({'status': 'success', 'message': 'Book issued successfully'})
    except Exception as e:
        return jsonify({'status': 'error', 'message': str(e)}), 400

@app.route('/api/books/return', methods=['POST'])
def return_book():
    data = request.json
    try:
        conn = get_db_connection()
        cursor = conn.cursor()
        cursor.execute('DELETE FROM books_issued WHERE bid = %s', (data['bid'],))
        cursor.execute('UPDATE books SET status = %s WHERE bid = %s', ('avail', data['bid']))
        conn.commit()
        cursor.close()
        conn.close()
        return jsonify({'status': 'success', 'message': 'Book returned successfully'})
    except Exception as e:
        return jsonify({'status': 'error', 'message': str(e)}), 400

if __name__ == '__main__':
    app.run(debug=True)
```

2. Create `requirements.txt`:

```
Flask==2.3.0
PyMySQL==1.1.0
cryptography==40.0.0
```

3. Create folder: `templates/`

4. Create `templates/index.html`:

```html
<!DOCTYPE html>
<html>
<head>
    <title>Library Management System</title>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }
        
        body {
            font-family: Arial, sans-serif;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            min-height: 100vh;
            padding: 20px;
        }
        
        .container {
            max-width: 1000px;
            margin: 0 auto;
            background: white;
            border-radius: 10px;
            box-shadow: 0 10px 40px rgba(0,0,0,0.2);
            overflow: hidden;
        }
        
        .header {
            background: #333;
            color: white;
            padding: 30px;
            text-align: center;
        }
        
        .header h1 {
            font-size: 2.5em;
            margin-bottom: 10px;
        }
        
        .nav {
            background: #FFBB00;
            padding: 10px;
            display: flex;
            gap: 10px;
            flex-wrap: wrap;
            justify-content: center;
        }
        
        button {
            background: black;
            color: white;
            border: none;
            padding: 12px 24px;
            cursor: pointer;
            border-radius: 5px;
            font-weight: bold;
            transition: 0.3s;
        }
        
        button:hover {
            background: #333;
        }
        
        .content {
            padding: 30px;
        }
        
        .section {
            display: none;
        }
        
        .section.active {
            display: block;
        }
        
        .form-group {
            margin-bottom: 15px;
        }
        
        label {
            display: block;
            margin-bottom: 5px;
            font-weight: bold;
        }
        
        input, select {
            width: 100%;
            padding: 10px;
            border: 1px solid #ddd;
            border-radius: 5px;
            font-size: 1em;
        }
        
        .message {
            padding: 15px;
            margin-bottom: 15px;
            border-radius: 5px;
            display: none;
        }
        
        .message.success {
            background: #d4edda;
            color: #155724;
            display: block;
        }
        
        .message.error {
            background: #f8d7da;
            color: #721c24;
            display: block;
        }
        
        table {
            width: 100%;
            border-collapse: collapse;
            margin-top: 20px;
        }
        
        table th, table td {
            border: 1px solid #ddd;
            padding: 12px;
            text-align: left;
        }
        
        table th {
            background: #f5f5f5;
            font-weight: bold;
        }
        
        .btn-delete {
            background: red;
            padding: 5px 10px;
            font-size: 0.9em;
        }
        
        .btn-delete:hover {
            background: darkred;
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="header">
            <h1>📚 Library Management System</h1>
            <p>Manage your library efficiently</p>
        </div>
        
        <div class="nav">
            <button onclick="showSection('view')">View Books</button>
            <button onclick="showSection('add')">Add Book</button>
            <button onclick="showSection('issue')">Issue Book</button>
            <button onclick="showSection('return')">Return Book</button>
            <button onclick="showSection('delete')">Delete Book</button>
        </div>
        
        <div class="content">
            <div id="message" class="message"></div>
            
            <!-- View Books Section -->
            <div id="view" class="section active">
                <h2>📖 View All Books</h2>
                <div id="booksList" style="margin-top: 20px;"></div>
            </div>
            
            <!-- Add Book Section -->
            <div id="add" class="section">
                <h2>➕ Add New Book</h2>
                <form onsubmit="addBook(event)">
                    <div class="form-group">
                        <label>Title:</label>
                        <input type="text" id="add_title" required>
                    </div>
                    <div class="form-group">
                        <label>Author:</label>
                        <input type="text" id="add_author" required>
                    </div>
                    <div class="form-group">
                        <label>Status:</label>
                        <select id="add_status" required>
                            <option value="avail">Available</option>
                            <option value="issued">Issued</option>
                        </select>
                    </div>
                    <button type="submit">Add Book</button>
                </form>
            </div>
            
            <!-- Issue Book Section -->
            <div id="issue" class="section">
                <h2>📤 Issue Book to Student</h2>
                <form onsubmit="issueBook(event)">
                    <div class="form-group">
                        <label>Book ID:</label>
                        <input type="number" id="issue_bid" required>
                    </div>
                    <div class="form-group">
                        <label>Student Name:</label>
                        <input type="text" id="issue_name" required>
                    </div>
                    <button type="submit">Issue Book</button>
                </form>
            </div>
            
            <!-- Return Book Section -->
            <div id="return" class="section">
                <h2>📥 Return Book</h2>
                <form onsubmit="returnBook(event)">
                    <div class="form-group">
                        <label>Book ID:</label>
                        <input type="number" id="return_bid" required>
                    </div>
                    <button type="submit">Return Book</button>
                </form>
            </div>
            
            <!-- Delete Book Section -->
            <div id="delete" class="section">
                <h2>🗑️ Delete Book</h2>
                <form onsubmit="deleteBook(event)">
                    <div class="form-group">
                        <label>Book ID:</label>
                        <input type="number" id="delete_bid" required>
                    </div>
                    <button type="submit">Delete Book</button>
                </form>
            </div>
        </div>
    </div>
    
    <script>
        // Show/Hide sections
        function showSection(sectionId) {
            document.querySelectorAll('.section').forEach(s => s.classList.remove('active'));
            document.getElementById(sectionId).classList.add('active');
            if (sectionId === 'view') loadBooks();
        }
        
        // Show message
        function showMessage(message, type) {
            const msg = document.getElementById('message');
            msg.textContent = message;
            msg.className = 'message ' + type;
            setTimeout(() => msg.className = 'message', 5000);
        }
        
        // Load and display all books
        function loadBooks() {
            fetch('/api/books')
                .then(r => r.json())
                .then(books => {
                    if (books.length === 0) {
                        document.getElementById('booksList').innerHTML = '<p>No books found</p>';
                        return;
                    }
                    let html = '<table><tr><th>ID</th><th>Title</th><th>Author</th><th>Status</th></tr>';
                    books.forEach(b => {
                        html += `<tr><td>${b.bid}</td><td>${b.title}</td><td>${b.author}</td><td>${b.status}</td></tr>`;
                    });
                    html += '</table>';
                    document.getElementById('booksList').innerHTML = html;
                });
        }
        
        // Add book
        function addBook(e) {
            e.preventDefault();
            const data = {
                title: document.getElementById('add_title').value,
                author: document.getElementById('add_author').value,
                status: document.getElementById('add_status').value
            };
            
            fetch('/api/books/add', {
                method: 'POST',
                headers: {'Content-Type': 'application/json'},
                body: JSON.stringify(data)
            })
            .then(r => r.json())
            .then(data => {
                showMessage(data.message, data.status);
                if (data.status === 'success') {
                    e.target.reset();
                    loadBooks();
                }
            });
        }
        
        // Issue book
        function issueBook(e) {
            e.preventDefault();
            const data = {
                bid: document.getElementById('issue_bid').value,
                issuedto: document.getElementById('issue_name').value
            };
            
            fetch('/api/books/issue', {
                method: 'POST',
                headers: {'Content-Type': 'application/json'},
                body: JSON.stringify(data)
            })
            .then(r => r.json())
            .then(data => {
                showMessage(data.message, data.status);
                if (data.status === 'success') {
                    e.target.reset();
                    loadBooks();
                }
            });
        }
        
        // Return book
        function returnBook(e) {
            e.preventDefault();
            const data = {
                bid: document.getElementById('return_bid').value
            };
            
            fetch('/api/books/return', {
                method: 'POST',
                headers: {'Content-Type': 'application/json'},
                body: JSON.stringify(data)
            })
            .then(r => r.json())
            .then(data => {
                showMessage(data.message, data.status);
                if (data.status === 'success') {
                    e.target.reset();
                    loadBooks();
                }
            });
        }
        
        // Delete book
        function deleteBook(e) {
            e.preventDefault();
            const bid = document.getElementById('delete_bid').value;
            
            fetch(`/api/books/delete/${bid}`, {
                method: 'DELETE'
            })
            .then(r => r.json())
            .then(data => {
                showMessage(data.message, data.status);
                if (data.status === 'success') {
                    e.target.reset();
                    loadBooks();
                }
            });
        }
        
        // Load books on page load
        loadBooks();
    </script>
</body>
</html>
```

---

### **Step 2: Create PythonAnywhere Account**

1. Go to [www.pythonanywhere.com](https://www.pythonanywhere.com)
2. Click **Sign Up** → Choose **Free account**
3. Enter username and password
4. Verify email
5. Log in

---

### **Step 3: Upload Your Project**

1. In PythonAnywhere dashboard, click **Files**
2. Click **Upload a file**
3. Upload:
   - `app.py`
   - `requirements.txt`
   - Create folder `templates/`
   - Upload `index.html` to `templates/`

**OR use Git:**
```bash
git clone https://github.com/yourusername/library-management.git
```

---

### **Step 4: Create Virtual Environment & Install Dependencies**

1. Click **Web** tab
2. Click **Add a new web app**
3. Choose **Flask** → **Python 3.10**
4. In **Web** tab, click **Open bash console**
5. Run:

```bash
mkvirtualenv --python=/usr/bin/python3.10 mylib
pip install -r requirements.txt
```

---

### **Step 5: Configure MySQL Database**

1. In PythonAnywhere, go to **Databases** tab
2. Click **Create a new database**
3. Database name: `librarydb`
4. MySQL password: Set a strong password
5. Click **Create**

6. Get the database connection details:
   - Host: Your PythonAnywhere MySQL host
   - User: `yourusername$librarydb`
   - Password: (the one you set)

7. Update `app.py` with correct credentials:

```python
db_config = {
    'host': 'yourusername.mysql.pythonanywhere-services.com',
    'user': 'yourusername$librarydb',
    'password': 'your_mysql_password',
    'database': 'yourusername$librarydb',
}
```

---

### **Step 6: Create Database Tables**

1. Click **Databases** → **Start MySQL console**
2. Run:

```sql
USE yourusername$librarydb;

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

---

### **Step 7: Configure Web App**

1. Go to **Web** tab
2. Under **Code**, set:
   - **Source code:** `/home/yourusername/mylib` (or your upload path)
   - **Working directory:** `/home/yourusername/mylib`

3. Click **WSGI configuration file** and replace content:

```python
import sys
path = '/home/yourusername/mylib'
if path not in sys.path:
    sys.path.insert(0, path)

from app import app
application = app
```

4. **Save and reload** the web app

---

### **Step 8: Get Your Public URL**

1. In **Web** tab, you'll see:
   ```
   Your web app is live at: https://yourusername.pythonanywhere.com
   ```

2. Click the link to access your application!

---

---

## **METHOD 2: Deploy on Heroku (More Scalable)**

### **Step 1: Create Heroku Account**

1. Go to [heroku.com](https://www.heroku.com)
2. Sign up for free account
3. Verify email
4. Install Heroku CLI: [heroku-cli](https://devcenter.heroku.com/articles/heroku-cli)

---

### **Step 2: Prepare Project**

Create `Procfile` (no extension):
```
web: gunicorn app:app
```

Update `requirements.txt`:
```
Flask==2.3.0
PyMySQL==1.1.0
cryptography==40.0.0
gunicorn==20.1.0
```

---

### **Step 3: Deploy to Heroku**

```bash
# Initialize git repository
git init
git add .
git commit -m "Initial commit"

# Login to Heroku
heroku login

# Create Heroku app
heroku create your-library-app

# Add MySQL add-on
heroku addons:create cleardb:ignite

# Get database URL
heroku config

# Deploy
git push heroku main

# View logs
heroku logs --tail
```

---

### **Step 4: Initialize Database**

```bash
heroku run python
```

Then:
```python
from app import db, get_db_connection
conn = get_db_connection()
cursor = conn.cursor()
cursor.execute('''
    CREATE TABLE books (
        bid INT PRIMARY KEY AUTO_INCREMENT,
        title VARCHAR(100),
        author VARCHAR(50),
        status VARCHAR(20)
    )
''')
conn.commit()
```

---

### **Step 5: Access Your App**

```bash
heroku open
```

Your app is now live at: `https://your-library-app.herokuapp.com`

---

---

## **METHOD 3: Deploy with Docker (Advanced)**

### **Step 1: Create Dockerfile**

```dockerfile
FROM python:3.10

WORKDIR /app

COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .

EXPOSE 5000

CMD ["gunicorn", "--bind", "0.0.0.0:5000", "app:app"]
```

### **Step 2: Create docker-compose.yml**

```yaml
version: '3.8'

services:
  web:
    build: .
    ports:
      - "5000:5000"
    environment:
      - FLASK_APP=app.py
      - DATABASE_URL=mysql://root:password@db:3306/librarydb
    depends_on:
      - db
    
  db:
    image: mysql:8.0
    environment:
      - MYSQL_ROOT_PASSWORD=password
      - MYSQL_DATABASE=librarydb
    volumes:
      - mysql_data:/var/lib/mysql
    ports:
      - "3306:3306"

volumes:
  mysql_data:
```

### **Step 3: Build and Run**

```bash
docker-compose up --build
```

App will be available at: `http://localhost:5000`

### **Step 4: Deploy to Cloud**

Push to Docker Hub:
```bash
docker build -t yourusername/library-app .
docker push yourusername/library-app
```

Deploy to:
- **AWS EC2**
- **Google Cloud Run**
- **DigitalOcean App Platform**
- **Azure Container Instances**

---

---

## **METHOD 4: Create Windows EXE (Desktop Distribution)**

### **Step 1: Install PyInstaller**

```bash
pip install pyinstaller
```

### **Step 2: Create EXE**

```bash
pyinstaller --onefile --windowed --icon=icon.ico main.py
```

### **Step 3: Distribute**

- EXE file in `dist/` folder
- Users can run without Python
- **Requires MySQL to be installed separately**

---

---

## **SUMMARY: Which Method to Choose?**

| Method | Ease | Cost | Best For | Setup Time |
|--------|------|------|----------|-----------|
| **PythonAnywhere** | ⭐⭐⭐⭐⭐ | Free/5$/mo | Small libraries, learners | 20 mins |
| **Heroku** | ⭐⭐⭐⭐ | Free/7$/mo | Growing apps, startups | 30 mins |
| **Docker** | ⭐⭐⭐ | 5-10$/mo | Enterprise, scalability | 1 hour |
| **EXE** | ⭐⭐⭐ | Free | Desktop users, offline | 15 mins |

---

## **RECOMMENDED: Use PythonAnywhere + Flask**

✅ Easiest setup  
✅ Free tier available  
✅ MySQL included  
✅ No installation needed for users  
✅ Access from any browser  
✅ Perfect for learning & small libraries  

---

## **Next Steps**

1. **Convert Tkinter to Flask** (use `app.py` provided above)
2. **Create PythonAnywhere account**
3. **Upload files**
4. **Configure MySQL**
5. **Deploy in 20 minutes!**

---

## **Support**

- PythonAnywhere Help: https://help.pythonanywhere.com
- Flask Docs: https://flask.palletsprojects.com
- Heroku Docs: https://devcenter.heroku.com

