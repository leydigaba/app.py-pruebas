from flask import Flask, request, render_template_string
import sqlite3

app = Flask(__name__)

# Base de datos insegura
def get_user(username, password):
    conn = sqlite3.connect("users.db")
    cursor = conn.cursor()

    # ❌ Vulnerabilidad SQL Injection
    query = f"SELECT * FROM users WHERE username='{username}' AND password='{password}'"
    cursor.execute(query)

    return cursor.fetchone()

@app.route("/")
def index():
    return """
        <h2>Login</h2>
        <form method='POST' action='/login'>
            <input name='username' placeholder='User'>
            <input name='password' placeholder='Pass'>
            <button>Login</button>
        </form>
    """

@app.route("/login", methods=["POST"])
def login():
    username = request.form.get("username")
    password = request.form.get("password")

    user = get_user(username, password)

    if user:
        # ❌ Vulnerabilidad XSS reflejado
        return render_template_string(f"<h1>Bienvenido {username}</h1>")
    else:
        return "Credenciales incorrectas", 401

# ❌ Endpoint expuesto sin autenticación
@app.route("/admin")
def admin():
    return "Panel de administración (sin protección)"

app.run(debug=True)
