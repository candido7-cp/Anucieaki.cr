<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>ANUNCIE AKI 🔥</title>

<script src="https://www.gstatic.com/firebasejs/10.12.2/firebase-app.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.12.2/firebase-auth.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.12.2/firebase-firestore.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.12.2/firebase-storage.js"></script>

<style>
body { font-family: Arial; background:#f2f4f5; margin:0; }
header { background:#ff6a00; color:#fff; padding:15px; text-align:center; }
.container { padding:10px; }

input {
  width:100%;
  padding:10px;
  margin:5px 0;
  border-radius:8px;
  border:1px solid #ccc;
}

.card {
  background:#fff;
  margin-top:10px;
  padding:10px;
  border-radius:10px;
  box-shadow:0 2px 10px rgba(0,0,0,0.1);
}

img {
  width:100%;
  border-radius:10px;
}

button {
  padding:10px;
  margin-top:5px;
  width:100%;
  border:none;
  border-radius:8px;
  font-weight:bold;
}

.btn {
  background:#25d366;
  color:white;
}

.btn-publicar {
  background:#00c853;
  color:white;
}
</style>
</head>

<body>

<header>ANUNCIE AKI 🔥</header>

<div class="container">

<h3>Login</h3>
<input id="email" placeholder="Email">
<input id="senha" placeholder="Senha">
<button onclick="login()">Entrar / Criar Conta</button>

<hr>

<h3>Criar anúncio</h3>
<input id="titulo" placeholder="Título">
<input id="preco" placeholder="Preço">
<input id="whatsapp" placeholder="WhatsApp (5511...)">
<input type="file" id="foto">
<button class="btn-publicar" onclick="postar()">Publicar</button>

<div id="lista"></div>

</div>

<script>
const firebaseConfig = {
  apiKey: "AIzaSyA3jnsONEgBuL5htlDEc-TA0NigBJmjOvs",
  authDomain: "anucieaki-ac5dd.firebaseapp.com",
  projectId: "anucieaki-ac5dd",
  storageBucket: "anucieaki-ac5dd.firebasestorage.app",
  messagingSenderId: "153399196349",
  appId: "1:153399196349:web:15a73d1b99c4e3d85a15d4"
};

firebase.initializeApp(firebaseConfig);

const auth = firebase.auth();
const db = firebase.firestore();
const storage = firebase.storage();

let user;

// LOGIN
function login(){
  const email = document.getElementById("email").value;
  const senha = document.getElementById("senha").value;

  auth.signInWithEmailAndPassword(email, senha)
  .catch(()=> auth.createUserWithEmailAndPassword(email, senha));
}

// POSTAR
function postar(){
  if(!user){
    alert("Faça login primeiro");
    return;
  }

  const file = document.getElementById("foto").files[0];
  if(!file){
    alert("Selecione uma imagem");
    return;
  }

  const ref = storage.ref("fotos/"+Date.now());

  ref.put(file).then(()=>{
    ref.getDownloadURL().then(url=>{

      db.collection("anuncios").add({
        titulo: titulo.value,
        preco: preco.value,
        whatsapp: whatsapp.value,
        foto: url,
        userId: user.uid,
        created: Date.now()
      });

      alert("Anúncio publicado 🚀");

    });
  });
}

// LISTAR
auth.onAuthStateChanged(u=>{
  user = u;

  db.collection("anuncios")
  .orderBy("created","desc")
  .onSnapshot(snap=>{
    lista.innerHTML="";

    snap.forEach(doc=>{
      const d = doc.data();

      lista.innerHTML += `
        <div class="card">
          <img src="${d.foto}">
          <h3>${d.titulo}</h3>
          <p><b>R$ ${d.preco}</b></p>

          <button class="btn" onclick="comprar('${d.whatsapp}','${d.titulo}')">
            Falar no WhatsApp
          </button>
        </div>
      `;
    });
  });
});

// PAGAMENTO + WHATSAPP
function comprar(wpp, titulo){
  window.open("https://pay.infinitepay.io/anderson_candido_raulino/VC0xLTAtUg-7Udis2xD1j-1,00");

  setTimeout(()=>{
    window.open(`https://wa.me/${wpp}?text=Já paguei pelo produto: ${titulo}`);
  },8000);
}
</script>

</body>
</html>
