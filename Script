const KEY = "casas-membros-v1";
let db = { casas: [], membros: [] };

// ---------- Salvar e carregar ----------
function salvar() {
  try { localStorage.setItem(KEY, JSON.stringify(db)); } catch (e) { aviso("Não foi possível salvar (foto muito grande?)"); }
}
function carregar() {
  try { const t = localStorage.getItem(KEY); if (t) db = JSON.parse(t); } catch (e) {}
  if (!db || !Array.isArray(db.casas) || !Array.isArray(db.membros)) db = { casas: [], membros: [] };
}
const id = () => Math.random().toString(36).slice(2, 9);
const $ = s => document.getElementById(s);
const esc = s => s.replace(/[&<>"]/g, c => ({ "&": "&amp;", "<": "&lt;", ">": "&gt;", '"': "&quot;" }[c]));
function aviso(msg) {
  const t = $("toast"); t.textContent = msg; t.classList.add("on");
  clearTimeout(aviso.t); aviso.t = setTimeout(() => t.classList.remove("on"), 2200);
}

// ---------- Regras ----------
const ocupadas = casaId => db.membros.filter(m => m.casa === casaId).length;
function mover(membroId, casaId) {
  const m = db.membros.find(x => x.id === membroId);
  if (!m || m.casa === casaId) return;
  if (casaId) {
    const c = db.casas.find(x => x.id === casaId);
    if (ocupadas(casaId) >= c.vagas) { aviso("A casa de " + c.dono + " está lotada"); return; }
  }
  m.casa = casaId || "";
  salvar(); desenhar();
}

// ---------- Desenho ----------
function cartaoMembro(m, vertical) {
  const foto = m.foto ? `<img src="${m.foto}" alt="">` : `<div class="av">${esc(m.nome[0] || "?").toUpperCase()}</div>`;
  const opts = `<option value="">Mover…</option><option value="">Sem casa</option>` +
    db.casas.map(c => `<option value="${c.id}">${esc(c.dono)}</option>`).join("");
  return `<div class="m ${vertical ? "v" : ""}" draggable="true" data-id="${m.id}">${foto}
    <span class="n" title="${esc(m.nome)}">${esc(m.nome)}</span>
    <select data-mv="${m.id}">${opts}</select><button class="x" data-delm="${m.id}" title="Excluir">✕</button></div>`;
}

function desenhar() {
  const livres = db.membros.filter(m => !m.casa);
  $("qtdLivres").textContent = livres.length;
  $("livres").innerHTML = livres.length ? livres.map(m => cartaoMembro(m, false)).join("") : `<div class="empty">Nenhum membro sem casa</div>`;

  $("casas").innerHTML = db.casas.length ? db.casas.map((c, i) => {
    const dentro = db.membros.filter(m => m.casa === c.id);
    const cheia = dentro.length >= c.vagas;
    const vazias = Math.max(0, c.vagas - dentro.length);
    const hue = (i * 47 + 12) % 360;
    return `<div class="house" style="--h:${hue}">
      <div class="chimney"></div><div class="roof"></div>
      <div class="sign" title="Casa de ${esc(c.dono)}">Casa de ${esc(c.dono)}</div>
      <div class="wall zone ${cheia ? "full" : ""}" data-casa="${c.id}">
        <div class="hh"><span class="cnt ${cheia ? "full" : ""}">${dentro.length}/${c.vagas} vagas</span>
          <button class="x" data-delc="${c.id}" title="Excluir casa">🗑️</button></div>
        <div class="cells">
          ${dentro.map(m => cartaoMembro(m, true)).join("")}
          ${Array.from({ length: vazias }, () => `<div class="cell win"></div>`).join("")}
        </div>
        <div class="door"></div>
      </div></div>`;
  }).join("") : `<div class="box">Nenhuma casa ainda. Construa a primeira acima! 🏗️</div>`;
}

// ---------- Cadastros ----------
$("addCasa").onclick = () => {
  const dono = $("dono").value.trim(), vagas = parseInt($("vagas").value, 10);
  if (!dono) return aviso("Digite o nome do dono");
  if (!(vagas >= 1)) return aviso("Informe as vagas");
  db.casas.push({ id: id(), dono, vagas });
  $("dono").value = ""; salvar(); desenhar();
};
function lerFoto(file) {
  return new Promise(res => {
    if (!file) return res("");
    const r = new FileReader();
    r.onload = () => {
      const img = new Image();
      img.onload = () => {
        const s = 120, cv = document.createElement("canvas");
        cv.width = cv.height = s;
        const lado = Math.min(img.width, img.height);
        cv.getContext("2d").drawImage(img, (img.width - lado) / 2, (img.height - lado) / 2, lado, lado, 0, 0, s, s);
        res(cv.toDataURL("image/jpeg", .8));
      };
      img.onerror = () => res("");
      img.src = r.result;
    };
    r.onerror = () => res("");
    r.readAsDataURL(file);
  });
}
$("addMembro").onclick = async () => {
  const nome = $("nome").value.trim();
  if (!nome) return aviso("Digite o nome do membro");
  const foto = await lerFoto($("foto").files[0]);
  db.membros.push({ id: id(), nome, foto, casa: "" });
  $("nome").value = ""; $("foto").value = ""; salvar(); desenhar();
};
["dono", "vagas"].forEach(i => $(i).addEventListener("keydown", e => e.key === "Enter" && $("addCasa").click()));
$("nome").addEventListener("keydown", e => e.key === "Enter" && $("addMembro").click());

// ---------- Excluir / mover pelo select ----------
document.addEventListener("click", e => {
  const dm = e.target.dataset.delm, dc = e.target.dataset.delc;
  if (dm) { db.membros = db.membros.filter(m => m.id !== dm); salvar(); desenhar(); }
  if (dc) {
    if (!confirm("Excluir esta casa? Os membros voltam para \"sem casa\".")) return;
    db.membros.forEach(m => { if (m.casa === dc) m.casa = ""; });
    db.casas = db.casas.filter(c => c.id !== dc); salvar(); desenhar();
  }
});
document.addEventListener("change", e => {
  if (e.target.dataset.mv && e.target.selectedIndex > 0) mover(e.target.dataset.mv, e.target.value);
});
$("limpar").onclick = () => {
  if (confirm("Apagar todas as casas e membros?")) { db = { casas: [], membros: [] }; salvar(); desenhar(); }
};

// ---------- Arrastar e soltar ----------
let arrastando = null;
document.addEventListener("dragstart", e => {
  const m = e.target.closest && e.target.closest(".m");
  if (!m) return;
  arrastando = m.dataset.id; m.classList.add("drag");
  e.dataTransfer.effectAllowed = "move";
  e.dataTransfer.setData("text/plain", arrastando);
});
document.addEventListener("dragend", () => {
  arrastando = null;
  document.querySelectorAll(".drag,.over").forEach(el => el.classList.remove("drag", "over"));
});
document.addEventListener("dragover", e => {
  const z = e.target.closest && e.target.closest(".zone");
  if (!z || !arrastando) return;
  e.preventDefault();
  document.querySelectorAll(".over").forEach(el => el !== z && el.classList.remove("over"));
  z.classList.add("over");
});
document.addEventListener("drop", e => {
  const z = e.target.closest && e.target.closest(".zone");
  if (!z || !arrastando) return;
  e.preventDefault();
  const alvo = arrastando;
  document.querySelectorAll(".over").forEach(el => el.classList.remove("over"));
  mover(alvo, z.dataset.casa);
});

carregar();
desenhar();
