# Python Security Challenge: "SecureVault" 🔓

## 🎯 Mission Brief
You've been hired as a security consultant to test a new password manager called "SecureVault". 
The junior developer who built it claims it's "totally secure", but you suspect otherwise. 
Your job is to find the vulnerabilities and fix them!

Read the entire material first BEFORE you go into working through it. 

Make notes. Everything that you learn, and everything you found wrong. 
** Writing helps you remember things. **

## 🛠️ Setup Instructions

### 1. Create Your Workspace
```
# Install Python (3.11 is recommended)
https://www.pythontutorial.net/getting-started/install-python/

# Install virtualenv
https://virtualenv.pypa.io/en/latest/installation.html

# Create a new directory for the challenge
mkdir securevault-challenge
cd securevault-challenge

# Initialize a git repository
git init
```

### 2. Set Up Virtual Environment
```
# Create virtual environment
# make sure you are in the folder securevault-challenge
virtualenv -p Python3.11 .

# Activate it (Windows)
Scripts\activate

# Activate it (Mac/Linux)
source bin/activate

# Install required packages
pip install flask

# Make the installed packages part of the requirements.txt file where all the needed packages are bundled.
pip freeze > requirements.txt
```

### 3. Create the Vulnerable Application

Create a file called `app.py`:

```
from flask import Flask, request, render_template_string, session, redirect, url_for
import os

app = Flask(__name__)
app.secret_key = "supersecretkey123"  # 🚩 Red flag #1

# "Secure" user database
users = {
    "admin": "password123",
    "user": "qwerty", 
    "guest": "guest"
}

# Password vault
vault = {
    "admin": ["gmail:admin@email.com:MySecureP@ssw0rd", "bank:admin:VerySecret123"],
    "user": ["facebook:user@email.com:password123"],
    "guest": []
}

@app.route('/')
def home():
    if 'username' in session:
        return f'''
        <h1>Welcome to SecureVault, {session['username']}!</h1>
        <a href="/vault">View Your Passwords</a> | 
        <a href="/admin">Admin Panel</a> | 
        <a href="/logout">Logout</a>
        '''
    return '''
    <h1>SecureVault - Ultra Secure Password Manager</h1>
    <form method="POST" action="/login">
        Username: <input name="username" required><br><br>
        Password: <input name="password" type="password" required><br><br>
        <button>Login</button>
    </form>
    <p><small>Try: admin/password123 or user/qwerty</small></p>
    '''

@app.route('/login', methods=['POST'])
def login():
    username = request.form['username']
    password = request.form['password']
    
    # Super secure authentication logic
    if username in users and users[username] == password:
        session['username'] = username
        return redirect('/')
    
    return "Invalid credentials! <a href='/'>Try again</a>"

@app.route('/vault')
def vault_view():
    if 'username' not in session:
        return redirect('/')
    
    username = session['username']
    user_passwords = vault.get(username, [])
    
    html = f"<h1>{username}'s Password Vault</h1>"
    if user_passwords:
        html += "<ul>"
        for pwd in user_passwords:
            html += f"<li>{pwd}</li>"
        html += "</ul>"
    else:
        html += "<p>No passwords stored yet.</p>"
    
    html += "<br><a href='/'>Home</a>"
    return html

@app.route('/admin')
def admin_panel():
    if 'username' not in session:
        return redirect('/')
    
    # 🚩 Red flag #2 - What's wrong with this check?
    if session['username'] == 'admin':
        return '''
        <h1>Admin Panel - TOP SECRET</h1>
        <h2>All User Data:</h2>
        <pre>''' + str(vault) + '''</pre>
        <h2>System Info:</h2>
        <p>Flag: CHALLENGE_COMPLETED</p>
        <a href="/">Home</a>
        '''
    
    return "Access Denied! Admins only! <a href='/'>Home</a>"

@app.route('/search')
def search():
    query = request.args.get('q', '')
    if 'username' not in session:
        return redirect('/')
    
    # 🚩 Red flag #3 - This looks suspicious...
    template = f'''
    <h1>Search Results for: {query}</h1>
    <p>No results found.</p>
    <form>
        <input name="q" placeholder="Search..." value="{query}">
        <button>Search</button>
    </form>
    <a href="/">Home</a>
    '''
    
    return render_template_string(template)

@app.route('/logout')
def logout():
    session.clear()
    return redirect('/')

if __name__ == '__main__':
    app.run(debug=True, port=5000)
```

## 🎯 Your Challenges

### Level 1: Basic Reconnaissance
- [ ] Run the application and explore all pages
- [ ] Try different login combinations
- [ ] Note any suspicious code patterns

### Level 2: Find the Vulnerabilities
Can you identify and exploit these security flaws?

1. **Authentication Bypass**: Can you access the admin panel without being admin?
2. **Template Injection**: Can you execute code through user input?
3. **Session Security**: What's wrong with the session management?
4. **Information Disclosure**: Can you see other users' passwords?
5. install bandit and scan the code with bandit

### Level 3: Exploitation
Try these attack vectors:

1. **Admin Panel Access**: 
   - Hint: Look at how the session is managed
   - Goal: Access `/admin` without admin credentials

2. **Template Injection**:
   - Hint: Check the `/search` endpoint
   - Try: `{{7*7}}` or `{{config}}`
   - Goal: Execute Python code

3. **Session Manipulation**:
   - Hint: Check browser dev tools → Application → Cookies
   - Goal: Modify your session to become admin

### Level 4: Fix the Code
Once you've found the vulnerabilities, fix them:

1. Implement proper session security
2. Add input validation and sanitization
3. Fix the authentication logic
4. Add proper access controls
5. Commit your fixes as separate files in the repo, like app-fixed.py


## 💡 Hints

1. **Session Cookies**: Check your browser's developer tools
2. **Template Injection**: Flask templates can execute Python expressions
3. **Authentication**: Sometimes the simplest approach works
4. **Admin Check**: Is there more than one way to become "admin"?

## 🔧 Running the Challenge

```
# Make sure your virtual environment is activated
source bin/activate  # or Scripts\activate on Windows

# Run the app
python app.py

# Visit http://localhost:5000 in your browser
```

## 📚 Learning Objectives

By completing this challenge, you'll learn:
- Setting up Python virtual environments
- Git repository basics
- Common web application vulnerabilities
- Session management security
- Input validation and sanitization
- Template injection attacks (SSTI)
- Basic penetration testing mindset

## 🎉 Bonus Points

- Document your findings in a markdown file
- Create a fixed version of the application
- Add proper logging and error handling
- Implement rate limiting for login attempts

---

Good luck, and happy hacking! 🕵️‍♂️
