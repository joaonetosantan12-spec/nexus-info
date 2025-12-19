# nexus-info
Site com notícias, vídeos e boletim escolar (TESTE)
<!DOCTYPE html>
<html lang="pt-br">
<head>
<meta charset="UTF-8">
<title>NEXUS INFO</title>

<style>
body{
    margin:0;
    font-family:Arial, sans-serif;
    background:linear-gradient(135deg,#0f2027,#203a43,#2c5364);
    color:white;
}

nav{
    display:flex;
    justify-content:space-between;
    align-items:center;
    padding:15px 30px;
    background:rgba(0,0,0,0.6);
    position:sticky;
    top:0;
}

nav ul{
    list-style:none;
    display:flex;
    gap:18px;
}

nav li{
    cursor:pointer;
    font-weight:bold;
}

nav li:hover{color:#00c6ff;}

.container{
    max-width:1200px;
    margin:auto;
    padding:20px;
}

.box{
    background:rgba(0,0,0,0.45);
    padding:20px;
    border-radius:10px;
    margin-bottom:20px;
}

input, textarea, select, button{
    width:100%;
    padding:10px;
    margin:6px 0;
    border:none;
    border-radius:6px;
}

button{
    background:#00c6ff;
    font-weight:bold;
    cursor:pointer;
}

button:hover{
    background:#0072ff;
    color:white;
}

.hidden{display:none;}

table{
    width:100%;
    border-collapse:collapse;
}

th,td{
    border:1px solid rgba(255,255,255,0.2);
    padding:10px;
    text-align:center;
}

th{background:rgba(0,0,0,0.4);}

footer{
    text-align:right;
    padding:10px 20px;
    font-size:0.85em;
    opacity:0.7;
}
</style>
</head>

<body>

<nav>
<h2>🌌 NEXUS INFO</h2>
<ul>
<li onclick="setSection('Notícias')">Notícias</li>
<li onclick="setSection('Vídeos')">Vídeos</li>
<li onclick="setSection('Programação')">Programação</li>
<li onclick="setSection('Ideias')">Ideias</li>
<li onclick="setSection('Boletim')">Boletim Escolar</li>
</ul>
</nav>

<div class="container">

<!-- LOGIN -->
<div class="box" id="loginBox">
<h3>Login</h3>
<input id="loginUser" placeholder="Usuário">
<input id="loginPass" type="password" placeholder="Senha">
<button onclick="login()">Entrar</button>
<button onclick="showRegister()">Criar conta</button>
</div>

<!-- REGISTRO -->
<div class="box hidden" id="registerBox">
<h3>Criar conta</h3>
<input id="regUser" placeholder="Usuário">
<input id="regPass" type="password" placeholder="Senha">
<button onclick="register()">Registrar</button>
<button onclick="backLogin()">Voltar</button>
</div>

<!-- SITE -->
<div class="hidden" id="siteBox">

<h2 id="sectionTitle">Notícias</h2>

<!-- BOLETIM -->
<div class="box hidden" id="boletimBox">

<!-- ADMIN -->
<div id="adminBoletim" class="hidden">
<h3>Alunos cadastrados</h3>
<div id="userList"></div>

<hr>

<h3 id="selectedUser"></h3>
<input type="number" id="notaRed" placeholder="Redação (0–10)">
<input type="number" id="notaSim" placeholder="Simulado (0–10)">
<input type="number" id="notaAtv" placeholder="Atividades (0–2)">
<input type="number" id="notaProva" placeholder="Prova (0–8)">
<button onclick="salvarNotas()">Salvar notas</button>
<button onclick="toggleVisibilidade()">Bloquear / Desbloquear visualização</button>
</div>

<!-- USUÁRIO -->
<div id="userBoletim" class="hidden">
<table>
<tr>
<th>Redação</th>
<th>Simulado</th>
<th>Atividades</th>
<th>Prova</th>
<th>Média</th>
</tr>
<tr id="linhaNotas"></tr>
</table>
</div>

</div>

<button onclick="logout()">Sair</button>

</div>
</div>

<footer>Nexus Info — v1.3</footer>

<script>
const ADMIN_USER="admin_nexus";
const ADMIN_PASS="N3xus@2025!Ultra";

let currentUser="";
let isAdmin=false;
let selectedAluno="";

function showRegister(){
loginBox.classList.add("hidden");
registerBox.classList.remove("hidden");
}

function backLogin(){
registerBox.classList.add("hidden");
loginBox.classList.remove("hidden");
}

function register(){
if(!regUser.value || !regPass.value) return alert("Preencha tudo");
localStorage.setItem("user_"+regUser.value, regPass.value);
alert("Conta criada!");
backLogin();
}

function login(){
if(loginUser.value===ADMIN_USER && loginPass.value===ADMIN_PASS){
enter(true,loginUser.value);
}else if(localStorage.getItem("user_"+loginUser.value)===loginPass.value){
enter(false,loginUser.value);
}else alert("Login inválido");
}

function enter(admin,user){
isAdmin=admin;
currentUser=user;
loginBox.classList.add("hidden");
registerBox.classList.add("hidden");
siteBox.classList.remove("hidden");
}

function logout(){
location.reload();
}

function setSection(sec){
sectionTitle.innerText=sec;
boletimBox.classList.toggle("hidden",sec!=="Boletim");
if(sec==="Boletim") loadBoletim();
}

function loadBoletim(){
if(isAdmin){
adminBoletim.classList.remove("hidden");
userBoletim.classList.add("hidden");

const users=Object.keys(localStorage)
.filter(k=>k.startsWith("user_"))
.map(u=>u.replace("user_",""));

userList.innerHTML="";
users.forEach(u=>{
const b=document.createElement("button");
b.innerText=u;
b.onclick=()=>selectAluno(u);
userList.appendChild(b);
});

}else{
adminBoletim.classList.add("hidden");
userBoletim.classList.remove("hidden");

const data=JSON.parse(localStorage.getItem("boletim_"+currentUser)||"{}");

if(data.visible===false){
userBoletim.innerHTML="<p>🔒 Boletim temporariamente bloqueado.</p>";
return;
}

linhaNotas.innerHTML=`
<td>${data.red ?? 0}</td>
<td>${data.sim ?? 0}</td>
<td>${data.atv ?? 0}</td>
<td>${data.prova ?? 0}</td>
<td>${data.media ?? 0}</td>`;
}
}

function selectAluno(u){
selectedAluno=u;
selectedUser.innerText="Aluno: "+u;
const d=JSON.parse(localStorage.getItem("boletim_"+u)||"{}");
notaRed.value=d.red ?? 0;
notaSim.value=d.sim ?? 0;
notaAtv.value=d.atv ?? 0;
notaProva.value=d.prova ?? 0;
}

function salvarNotas(){
const r=+notaRed.value;
const s=+notaSim.value;
const a=+notaAtv.value;
const p=+notaProva.value;
const m=((r+s+a+p)/3).toFixed(2);

localStorage.setItem("boletim_"+selectedAluno,JSON.stringify({
red:r,
sim:s,
atv:a,
prova:p,
media:m,
visible:true
}));

alert("Notas salvas!");
loadBoletim();
}

function toggleVisibilidade(){
const d=JSON.parse(localStorage.getItem("boletim_"+selectedAluno)||"{}");
d.visible = d.visible === false ? true : false;
localStorage.setItem("boletim_"+selectedAluno,JSON.stringify(d));
alert(d.visible ? "Boletim liberado" : "Boletim bloqueado");
loadBoletim();
}
</script>

</body>
</html>
