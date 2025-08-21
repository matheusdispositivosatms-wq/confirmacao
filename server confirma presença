const express = require('express');
const bodyParser = require('body-parser');
const session = require('express-session');
const sqlite3 = require('sqlite3').verbose();
const QRCode = require('qrcode');
require('dotenv').config();

const app = express();
const port = process.env.PORT || 3000;

app.use(bodyParser.urlencoded({ extended: true }));
app.use(bodyParser.json());
app.use(express.static('public'));

app.use(session({
    secret: process.env.SESSION_SECRET || 'secret',
    resave: false,
    saveUninitialized: true
}));

// Banco de dados
const db = new sqlite3.Database('./db.sqlite');

db.serialize(() => {
    db.run(`CREATE TABLE IF NOT EXISTS users (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        name TEXT UNIQUE,
        email TEXT UNIQUE,
        phone TEXT UNIQUE,
        password TEXT,
        approved INTEGER DEFAULT 0,
        token TEXT
    )`);
});

// Rotas
app.post('/register', (req, res) => {
    const { name, email, phone, password } = req.body;
    if(!name || !email || !phone || !password) return res.json({status:'error', msg:'Preencha todos os campos'});

    db.get("SELECT * FROM users WHERE name=? OR email=? OR phone=?", [name,email,phone], (err,row)=>{
        if(row) return res.json({status:'error', msg:'Usuário já cadastrado'});
        const token = Math.random().toString(36).substring(2,10);
        db.run("INSERT INTO users(name,email,phone,password,token) VALUES(?,?,?,?,?)", [name,email,phone,password,token], function(err){
            if(err) return res.json({status:'error', msg:'Erro ao registrar'});
            res.json({status:'ok', msg:'Aguardando aprovação do admin', token});
        });
    });
});

app.post('/login', (req,res)=>{
    const { user, password } = req.body;
    db.get("SELECT * FROM users WHERE (email=? OR phone=?) AND password=?", [user,user,password], (err,row)=>{
        if(!row) return res.json({status:'error', msg:'Usuário ou senha inválidos'});
        if(!row.approved) return res.json({status:'pending', msg:'Aguardando aprovação'});
        res.json({status:'ok', token: row.token});
    });
});

app.post('/admin/login', (req,res)=>{
    const { username, password } = req.body;
    if(username === process.env.ADMIN_USER && password === process.env.ADMIN_PASS){
        req.session.admin = true;
        res.json({status:'ok'});
    } else {
        res.json({status:'error', msg:'Usuário ou senha inválidos'});
    }
});

app.get('/admin/users', (req,res)=>{
    if(!req.session.admin) return res.json({status:'error', msg:'Não autorizado'});
    db.all("SELECT * FROM users", (err, rows)=>{
        res.json(rows);
    });
});

app.post('/admin/approve', (req,res)=>{
    if(!req.session.admin) return res.json({status:'error', msg:'Não autorizado'});
    const { id } = req.body;
    db.run("UPDATE users SET approved=1 WHERE id=?", [id], function(err){
        if(err) return res.json({status:'error'});
        res.json({status:'ok'});
    });
});

app.listen(port, ()=>console.log(`Servidor rodando em http://localhost:${port}`));
