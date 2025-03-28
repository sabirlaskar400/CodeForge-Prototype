# CodeForge-Prototype
Its is a prototype for codeforge hackathon , its describe the implementation of the passGraud project idea. 


# PassGuard - AI-Powered Breach Alert System

## Project Structure

/passguard
├── backend  # Flask Backend
│   ├── app.py  # Main API Server
│   ├── models.py  # Database Models
│   ├── routes.py  # API Routes
│   ├── requirements.txt  # Dependencies
│   ├── config.py  # Configurations (Database, API Keys)
│   ├── encryption.py  # AES Encryption for Passwords
│   └── breach_check.py  # Integrates Have I Been Pwned API
│
├── frontend  # React Frontend
│   ├── src
│   │   ├── components
│   │   │   ├── Dashboard.js
│   │   │   ├── BreachCheck.js
│   │   │   ├── PasswordGenerator.js
│   │   │   └── Navbar.js
│   │   ├── App.js
│   │   ├── index.js
│   │   └── styles.css
│   ├── package.json  # React Dependencies
│   └── .env  # API Keys and Environment Variables
│
└── README.md  # Project Documentation

---

## Backend (Flask API)

### **1. app.py (Main API Server)**
```python
from flask import Flask
from routes import setup_routes
from flask_cors import CORS

app = Flask(__name__)
CORS(app)  # Enable CORS for frontend integration
setup_routes(app)

if __name__ == "__main__":
    app.run(debug=True)
```

### **2. routes.py (API Routes)**
```python
from flask import request, jsonify
from breach_check import check_breach
from encryption import encrypt_password

def setup_routes(app):
    @app.route("/check_breach", methods=["POST"])
    def breach_check():
        data = request.json
        email = data.get("email")
        result = check_breach(email)
        return jsonify(result)
    
    @app.route("/generate_password", methods=["GET"])
    def generate_password():
        import secrets, string
        characters = string.ascii_letters + string.digits + "!@#$%^&*()"
        password = ''.join(secrets.choice(characters) for i in range(12))
        return jsonify({"password": password})
```

### **3. breach_check.py (Check Breach via API)**
```python
import requests

def check_breach(email):
    url = f"https://haveibeenpwned.com/api/v3/breachedaccount/{email}"
    headers = {"hibp-api-key": "YOUR_API_KEY"}
    response = requests.get(url, headers=headers)
    if response.status_code == 200:
        return {"breached": True, "details": response.json()}
    else:
        return {"breached": False}
```

### **4. encryption.py (AES Encryption for Passwords)**
```python
from cryptography.fernet import Fernet

KEY = Fernet.generate_key()
cipher_suite = Fernet(KEY)

def encrypt_password(password):
    return cipher_suite.encrypt(password.encode()).decode()

def decrypt_password(encrypted_password):
    return cipher_suite.decrypt(encrypted_password.encode()).decode()
```

---

## Frontend (React.js)

### **1. App.js (Main Component)**
```javascript
import React from "react";
import Dashboard from "./components/Dashboard";
import Navbar from "./components/Navbar";

function App() {
    return (
        <div>
            <Navbar />
            <Dashboard />
        </div>
    );
}
export default App;
```

### **2. Dashboard.js**
```javascript
import React, { useState } from "react";
import BreachCheck from "./BreachCheck";
import PasswordGenerator from "./PasswordGenerator";

function Dashboard() {
    return (
        <div>
            <h2>Welcome to PassGuard</h2>
            <BreachCheck />
            <PasswordGenerator />
        </div>
    );
}
export default Dashboard;
```

### **3. BreachCheck.js**
```javascript
import React, { useState } from "react";

function BreachCheck() {
    const [email, setEmail] = useState("");
    const [result, setResult] = useState(null);

    const checkBreach = async () => {
        const response = await fetch("http://localhost:5000/check_breach", {
            method: "POST",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify({ email })
        });
        const data = await response.json();
        setResult(data);
    };

    return (
        <div>
            <input type="email" value={email} onChange={(e) => setEmail(e.target.value)} placeholder="Enter email" />
            <button onClick={checkBreach}>Check Breach</button>
            {result && <p>{result.breached ? "Breached!" : "Safe!"}</p>}
        </div>
    );
}
export default BreachCheck;
```

### **4. PasswordGenerator.js**
```javascript
import React, { useState } from "react";

function PasswordGenerator() {
    const [password, setPassword] = useState("");

    const generatePassword = async () => {
        const response = await fetch("http://localhost:5000/generate_password");
        const data = await response.json();
        setPassword(data.password);
    };

    return (
        <div>
            <button onClick={generatePassword}>Generate Secure Password</button>
            {password && <p>Generated: {password}</p>}
        </div>
    );
}
export default PasswordGenerator;
```

---

## **Deployment & Setup**

1. Install backend dependencies:
   ```sh
   pip install flask flask-cors cryptography requests
   ```
2. Install frontend dependencies:
   ```sh
   npm install
   ```
3. Run the backend:
   ```sh
   python app.py
   ```
4. Run the frontend:
   ```sh
   npm start
   ```
