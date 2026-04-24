<!DOCTYPE html>
<html lang="pt-br">
<head>
<meta charset="UTF-8">
<title>Marketplace App</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<style>
body {font-family: Arial; margin:0; background:#f2f2f2;}
.header {background:#2ecc71; color:#fff; padding:15px; text-align:center; font-size:20px;}
.search {padding:10px;}
input, select {width:100%; padding:10px; margin:5px 0; border-radius:8px; border:1px solid #ccc;}
.grid {display:grid; grid-template-columns:1fr 1fr; gap:10px; padding:10px;}
.card {background:#fff; border-radius:10px; padding:10px; position:relative;}
.card img, .card video {width:100%; height:160px; object-fit:cover; border-radius:8px;}
.fav {position:absolute; top:10px; right:10px; font-size:20px; cursor:pointer;}
button {padding:10px; border:none; border-radius:8px; background:#2ecc71; color:#fff; margin:5px 0;}
.modal {position:fixed; top:0; left:0; width:100%; height:100%; background:#000000aa; display:none; justify-content:center; align-items:center;}
.modal-content {background:#fff; padding:20px; width:90%; max-height:90vh; overflow:auto; border-radius:10px;}
</style>
</head>
<body>

<div class="header">📱 Marketplace</div>

<div class="search">
  <input id="busca" placeholder="Buscar..." onkeyup="buscar()">
</div>

<!-- SEM LOGIN: SEMPRE PODE ANUNCIAR -->
<div style="padding:10px;">
  <h3>Criar anúncio</h3>
  <input id="telefone" placeholder="Seu WhatsApp (ex: 85999999999)">
  <input id="titulo" placeholder="Título">
  <select id="categoria">
    <option>Hobby</option>
    <option>Saúde e Beleza</option>
    <option>Móveis</option>
    <option>Veículos</option>
    <option>Brinquedos</option>
    <option>Papelaria</option>
    <option>Variados</option>
  </select>
  <label style="display:block;background:#3498db;color:#fff;padding:10px;border-radius:8px;text-align:center;cursor:pointer;">Adicionar imagem ou vídeo<input type="file" accept="image/*,video/*" onchange="preview(event)" style="display:none;"></label>
  <textarea id="descricao" placeholder="Descrição"></textarea>
  <button onclick="publicar()">Publicar anúncio</button>
</div>

<div class="grid" id="feed"></div>

<div id="modal" class="modal">
  <div class="modal-content" id="modalContent"></div>
</div>

<script>
let anuncios = JSON.parse(localStorage.getItem('anuncios')) || [];
let favoritos = JSON.parse(localStorage.getItem('favoritos')) || [];
let midia = null;

function preview(e){
  let file = e.target.files[0];
  let r = new FileReader();
  r.onload = ev => midia = ev.target.result;
  r.readAsDataURL(file);
}

function publicar(){
  let telefone = document.getElementById('telefone').value.replace(/\D/g,'');
  let titulo = document.getElementById('titulo').value;
  let categoria = document.getElementById('categoria').value;
  let descricao = document.getElementById('descricao').value;

  if(!telefone || !titulo){
    alert('Preencha telefone e título');
    return;
  }

  let anuncio = {
    id: Date.now(),
    telefone,
    titulo,
    categoria,
    descricao,
    midia
  };

  anuncios.unshift(anuncio);
  salvar();
  carregar();

  // fluxo com pagamento
  let agora = new Date();
  let limite = new Date();
  limite.setDate(agora.getDate()+1);

  anuncios[0].ativo = false;
  anuncios[0].criado = agora.toISOString();
  anuncios[0].limitePagamento = limite.toISOString();

  window.open(`https://wa.me/${atob('NTU4NTkyMDAxMzk4NA==')}?text=APROVAR ${anuncios[0].id}`);

  alert('Anúncio criado! Você tem 1 dia para pagar.');
}

function favoritar(id){
  if(favoritos.includes(id)) favoritos = favoritos.filter(f=>f!=id);
  else favoritos.push(id);
  localStorage.setItem('favoritos', JSON.stringify(favoritos));
  carregar();
}

function carregar(){
  let hoje = new Date();
  let feed = document.getElementById('feed');
  feed.innerHTML='';

  anuncios.forEach(a=>{
    if(!a.ativo && a.limitePagamento && new Date(a.limitePagamento) < hoje) return;
    if(a.expira && new Date(a.expira) < hoje) return;
    let fav = favoritos.includes(a.id) ? '❤️' : '🤍';

    let mid = a.midia?.includes('video') ? `<video src="${a.midia}"></video>` : `<img src="${a.midia}">`;

    feed.innerHTML += `
      <div class="card$1">
        <div onclick='abrir($2)' style="cursor:pointer">
        <div class="fav" onclick="event.stopPropagation();favoritar(${a.id})">${fav}</div>
        ${mid}
        <h4>${a.titulo}</h4>
        <small>${a.categoria}</small>
      </div>
        <button onclick="event.stopPropagation();abrir($2)">🔍 Ver detalhes</button>
        <button onclick="event.stopPropagation();comprar('$2.telefone','$2.titulo')">🛒 Comprar</button>
      </div>
    `;
  });
}

function confirmarPagamento(cmd){
  let p = cmd.split(' ');
  let id = parseInt(p[1]);
  let a = anuncios.find(x=>x.id==id);
  if(a){
    a.ativo = true;
    let exp = new Date();
    exp.setDate(exp.getDate()+15);
    a.expira = exp.toISOString();
    salvar(); carregar();
  }
}

function abrir(a){
  let c = document.getElementById('modalContent');

  let mid = a.midia?.includes('video') ? `<video src="${a.midia}" controls style="max-height:300px;width:100%;object-fit:contain"></video>` : `<img src="${a.midia}" style="max-height:300px;width:100%;object-fit:contain">`;

  c.innerHTML = `
    <h2>${a.titulo}</h2>
    ${mid}
    <p>${a.descricao}</p>
    <button onclick="comprar('${a.telefone}','${a.titulo}')">🛒 Comprar</button>
    <button onclick="fechar()">Fechar</button>
  `;

  document.getElementById('modal').style.display='flex';
}

function comprar(num,titulo){
  window.open(`https://wa.me/55${num}?text=Tenho interesse em ${titulo}`);
}

function fechar(){ document.getElementById('modal').style.display='none'; }

function buscar(){
  let t = document.getElementById('busca').value.toLowerCase();
  document.querySelectorAll('.card').forEach(c=>{
    c.style.display = c.innerText.toLowerCase().includes(t)?'block':'none';
  });
}

function salvar(){ localStorage.setItem('anuncios', JSON.stringify(anuncios)); }

carregar();
</script>

</body>
</html>
