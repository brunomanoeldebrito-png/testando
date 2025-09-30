# rubisxxx.py - RubisXXX Completão com limites e bloqueio automático
import os
import sqlite3
from flask import Flask, request, session, redirect, url_for, render_template_string, jsonify
from datetime import datetime, timedelta

app = Flask(__name__)
app.secret_key = "rubisxxx-secret"

DB_NAME = "rubisxxx.db"

# --- Banco de dados ---
def init_db():
    conn = sqlite3.connect(DB_NAME)
    c = conn.cursor()
    # tabela de usuários
    c.execute("""
        CREATE TABLE IF NOT EXISTS users (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            email TEXT UNIQUE,
            password TEXT,
            full_name TEXT,
            nickname TEXT,
            plan TEXT DEFAULT 'free',
            hours_used INTEGER DEFAULT 0,
            images_used INTEGER DEFAULT 0,
            last_login DATETIME
        )
    """)
    # tabela de mensagens
    c.execute("""
        CREATE TABLE IF NOT EXISTS messages (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            user_id INTEGER,
            role TEXT,
            content TEXT,
            timestamp DATETIME DEFAULT CURRENT_TIMESTAMP
        )
    """)
    conn.commit()
    conn.close()

init_db()

# --- Limites ---
PLAN_LIMITS = {
    "free": {"hours":24, "images":5},
    "plus": {"hours":80, "images":20},
    "pro": {"hours":10000, "images":1000} # basicamente ilimitado
}

# --- Templates (HTML embutido) ---
login_template = """ ... mesmo template anterior ... """
chat_template = """ ... mesmo template anterior ... """

# --- Funções auxiliares ---
def get_user_by_email(email):
    conn = sqlite3.connect(DB_NAME)
    c = conn.cursor()
    c.execute("SELECT * FROM users WHERE email=?",(email,))
    user = c.fetchone()
    conn.close()
    return user

def add_user(full_name,nickname,email,password):
    conn = sqlite3.connect(DB_NAME)
    c = conn.cursor()
    try:
        c.execute("INSERT INTO users (full_name,nickname,email,password) VALUES (?,?,?,?)",(full_name,nickname,email,password))
        conn.commit()
    except:
        conn.close()
        return False
    conn.close()
    return True

def add_message(user_id,role,content):
    conn = sqlite3.connect(DB_NAME)
    c = conn.cursor()
    c.execute("INSERT INTO messages (user_id,role,content) VALUES (?,?,?)",(user_id,role,content))
    conn.commit()
    conn.close()

def get_messages(user_id):
    conn = sqlite3.connect(DB_NAME)
    c = conn.cursor()
    c.execute("SELECT role,content FROM messages WHERE user_id=? ORDER BY timestamp ASC",(user_id,))
    msgs = c.fetchall()
    conn.close()
    return msgs

def generate_answer(msg):
    preview=""
    if "<html" in msg or "<body" in msg:
        preview=msg
    return {"answer": f"(IA Respondeu): {msg}", "preview": preview}

def check_limits(user):
    plan=user[5]
    hours_used=user[6]
    images_used=user[7]
    limits=PLAN_LIMITS[plan]
    if hours_used >= limits["hours"] or images_used >= limits["images"]:
        return False
    return True

def increment_usage(user_id, images=0, hours=0.1):
    conn = sqlite3.connect(DB_NAME)
    c = conn.cursor()
    c.execute("UPDATE users SET hours_used=hours_used+?, images_used=images_used+? WHERE id=?",(hours,images,user_id))
    conn.commit()
    conn.close()

# --- Rotas ---
@app.route("/", methods=["GET"])
def index():
    if "user_id" in session:
        return redirect(url_for("chat"))
    return render_template_string(login_template)

@app.route("/register", methods=["POST"])
def register():
    full_name=request.form["full_name"]
    nickname=request.form["nickname"]
    email=request.form["email"]
    password=request.form["password"]
    if add_user(full_name,nickname,email,password):
        return "Cadastro realizado! <a href='/'>Voltar para login</a>"
    else:
        return "Erro: email já cadastrado! <a href='/'>Voltar</a>"

@app.route("/login", methods=["POST"])
def login():
    email=request.form["email"]
    password=request.form["password"]
    user=get_user_by_email(email)
    if user and user[3]==password:
        session["user_id"]=user[0]
        session["nickname"]=user[2]
        session["plan"]=user[5]
        # zera a conversa se limite atingido
        if not check_limits(user):
            session["blocked"]=True
        else:
            session["blocked"]=False
        return redirect(url_for("chat"))
    return "Login ou senha inválidos! <a href='/'>Voltar</a>"

@app.route("/chat", methods=["GET"])
def chat():
    if "user_id" not in session:
        return redirect(url_for("index"))
    if session.get("blocked",False):
        return "Você atingiu o limite do seu plano. <br>Atualize para Plus/Pro via Pix 71982513027 e inicie uma nova conversa. <a href='/logout'>Sair</a>"
    return render_template_string(chat_template,nickname=session["nickname"],plan=session["plan"])

@app.route("/send_message", methods=["POST"])
def send_message():
    if "user_id" not in session:
        return jsonify({"answer":"Faça login primeiro"})
    user=get_user_by_email_by_id(session["user_id"])
    if not check_limits(user):
        session["blocked"]=True
        return jsonify({"answer":"Você atingiu o limite do seu plano. Inicie nova conversa."})
    data=request.get_json()
    msg=data["message"]
    add_message(session["user_id"],"user",msg)
    increment_usage(session["user_id"])
    ans=generate_answer(msg)
    add_message(session["user_id"],"bot",ans["answer"])
    return jsonify(ans)

def get_user_by_email_by_id(user_id):
    conn = sqlite3.connect(DB_NAME)
    c = conn.cursor()
    c.execute("SELECT * FROM users WHERE id=?",(user_id,))
    user=c.fetchone()
    conn.close()
    return user

@app.route("/logout")
def logout():
    session.clear()
    return redirect(url_for("index"))

# --- Rodar ---
if __name__=="__main__":
    app.run(debug=True)
