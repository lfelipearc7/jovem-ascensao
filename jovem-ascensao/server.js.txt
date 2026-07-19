// server.js
const express = require('express');
const sqlite3 = require('sqlite3').verbose();
const cors = require('cors');
const app = express();
app.use(cors());
app.use(express.json());
app.use(express.static('public'));

const db = new sqlite3.Database('./game.db');
db.run(`CREATE TABLE IF NOT EXISTS players(
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  name TEXT, patri INTEGER, cargo TEXT, month INTEGER, createdAt DATETIME DEFAULT CURRENT_TIMESTAMP
)`);

// Ranking global
app.get('/api/ranking', (req,res)=>{
  db.all("SELECT name,patri,cargo,month FROM players ORDER BY patri DESC LIMIT 20",(e,r)=>res.json(r));
});

// Salvar score
app.post('/api/save',(req,res)=>{
  const {name,patri,cargo,month}=req.body;
  db.run("INSERT INTO players(name,patri,cargo,month) VALUES(?,?,?,?)",[name,patri,cargo,month]);
  res.json({ok:true});
});

// IA de mercado (simulada no servidor)
app.get('/api/market',(req,res)=>{
  const cycle = Math.sin(Date.now()/86400000)*10; // ciclo 10%
  const inflation = 0.003 + Math.random()*0.004;
  const sectors = {
    Tech: 1+cycle/100,
    RealEstate: 1-cycle/200,
    Finance: 1+Math.random()*0.02
  };
  res.json({inflation,sectors,cycle});
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => console.log("Server on " + PORT));
