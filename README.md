Got it! You're asking for a **Model-Context-Protocol (MCP)** based **Client-Server** project to **rent houses in a metropolitan city**.

---

### 💡 What is MCP?

MCP (Model-Context-Protocol) is a software architectural pattern used to separate concerns in distributed systems, often in **client-server** or **microservices** architectures. Here's how it breaks down:

- **Model**: Domain-specific data and logic (e.g. houses, users, bookings).
- **Context**: Defines use-cases and workflows (e.g. searching, booking a house).
- **Protocol**: Communication rules between components (e.g. HTTP/REST APIs, gRPC, etc.).

---

### 🎯 Project: **MetroHouse Rentals**  
A simple **client-server system** that lets users browse and rent houses in a metro city.

---

## 🔧 Tech Stack
- **Backend**: Python (Flask)
- **Frontend**: HTML + JS (can be extended to React)
- **Database**: SQLite (for demo)
- **Protocol**: REST API (over HTTP)

---

## 🧱 Project Structure

```
metrohouse/
│
├── server/                    # Server-side code
│   ├── model/                # Domain models
│   │   ├── house.py
│   │   └── user.py
│   ├── context/              # Use cases
│   │   ├── rent_house.py
│   │   └── search_house.py
│   ├── protocol/             # API endpoints
│   │   └── routes.py
│   ├── app.py                # Flask app entry point
│   └── db.sqlite3            # SQLite DB
│
├── client/                   # Frontend
│   ├── index.html
│   ├── rent.html
│   └── js/
│       └── api.js
│
└── README.md
```

---

## 🔧 Backend Breakdown

### ✅ Model: `model/house.py`
```python
class House:
    def __init__(self, id, address, price, available):
        self.id = id
        self.address = address
        self.price = price
        self.available = available
```

### ✅ Context: `context/search_house.py`
```python
from db import get_db

def search_houses():
    db = get_db()
    return db.execute("SELECT * FROM houses WHERE available = 1").fetchall()
```

### ✅ Protocol: `protocol/routes.py`
```python
from flask import Blueprint, jsonify, request
from context.search_house import search_houses
from context.rent_house import rent_house

bp = Blueprint('routes', __name__)

@bp.route('/api/houses', methods=['GET'])
def get_houses():
    return jsonify([dict(row) for row in search_houses()])

@bp.route('/api/rent/<int:house_id>', methods=['POST'])
def rent(house_id):
    result = rent_house(house_id)
    return jsonify({"status": result})
```

### ✅ Entry Point: `app.py`
```python
from flask import Flask
from protocol.routes import bp
from db import init_db

app = Flask(__name__)
app.register_blueprint(bp)

@app.before_first_request
def setup():
    init_db()

if __name__ == "__main__":
    app.run(debug=True)
```

---

## 🎨 Frontend Sample: `client/index.html`

```html
<!DOCTYPE html>
<html>
<head>
    <title>MetroHouse Rentals</title>
</head>
<body>
    <h1>Available Houses</h1>
    <ul id="houseList"></ul>

    <script src="js/api.js"></script>
</body>
</html>
```

### `client/js/api.js`
```javascript
fetch('http://localhost:5000/api/houses')
  .then(res => res.json())
  .then(data => {
    const list = document.getElementById('houseList');
    data.forEach(house => {
      let li = document.createElement('li');
      li.innerText = `${house.address} - $${house.price}`;
      list.appendChild(li);
    });
  });
```

---

## 🧪 Example Use Case (Context): `context/rent_house.py`
```python
from db import get_db

def rent_house(house_id):
    db = get_db()
    house = db.execute("SELECT * FROM houses WHERE id = ? AND available = 1", (house_id,)).fetchone()
    if house:
        db.execute("UPDATE houses SET available = 0 WHERE id = ?", (house_id,))
        db.commit()
        return "success"
    return "unavailable"
```

---

## 🗃️ Database Init: `db.py`
```python
import sqlite3
from flask import g

DATABASE = 'db.sqlite3'

def get_db():
    db = getattr(g, '_database', None)
    if db is None:
        db = g._database = sqlite3.connect(DATABASE)
        db.row_factory = sqlite3.Row
    return db

def init_db():
    db = get_db()
    with open('schema.sql', mode='r') as f:
        db.cursor().executescript(f.read())
    db.commit()
```

### `schema.sql`
```sql
CREATE TABLE IF NOT EXISTS houses (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    address TEXT NOT NULL,
    price INTEGER NOT NULL,
    available INTEGER NOT NULL
);

INSERT INTO houses (address, price, available) VALUES
('123 Main St', 1500, 1),
('456 Broadway', 2000, 1),
('789 Elm St', 1800, 1);
```

---

## ✅ How It Works

- **Client** requests `/api/houses` → protocol layer handles it.
- Protocol delegates to `context/search_houses`.
- Context uses `model` + `db` to fetch available houses.
- Data returned to client → rendered in browser.
- Renting works similarly via POST to `/api/rent/:id`.

---

Would you like me to zip this up into a downloadable starter project? Or maybe you'd like a React-based frontend too?
