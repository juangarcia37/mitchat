<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Chat Español ↔ Vietnamita</title>
<style>
  body { font-family: Arial, sans-serif; max-width: 600px; margin: 30px auto; padding: 10px; background: #f7f7f7; }
  h1 { text-align: center; }
  #chat { border: 1px solid #ccc; height: 400px; overflow-y: auto; background: white; padding: 10px; }
  .mensaje { margin-bottom: 15px; }
  .idioma1 { color: #222; }
  .idioma2 { color: #0066cc; margin-left: 20px; font-style: italic; }
  #inputArea { margin-top: 15px; display: flex; }
  #inputTexto { flex-grow: 1; padding: 8px; font-size: 16px; }
  #btnEnviar { padding: 8px 15px; font-size: 16px; margin-left: 8px; cursor: pointer; }
</style>
</head>
<body>

<h1>Chat Español ↔ Vietnamita</h1>

<div id="chat"></div>

<div id="inputArea">
  <input type="text" id="inputTexto" placeholder="Escribe aquí..." />
  <button id="btnEnviar">Enviar</button>
</div>

<script src="https://www.gstatic.com/firebasejs/8.10.1/firebase-app.js"></script>
<script src="https://www.gstatic.com/firebasejs/8.10.1/firebase-database.js"></script>
<script>
  // 🔹 CONFIGURA TU FIREBASE AQUÍ 🔹
  const firebaseConfig = {
    apiKey: "TU_API_KEY",
    authDomain: "TU_DOMINIO.firebaseapp.com",
    databaseURL: "https://TU_ID.firebaseio.com",
    projectId: "TU_ID",
    storageBucket: "TU_BUCKET.appspot.com",
    messagingSenderId: "TU_MESSAGING_ID",
    appId: "TU_APP_ID"
  };

  firebase.initializeApp(firebaseConfig);
  const db = firebase.database();

  const chatDiv = document.getElementById('chat');
  const inputTexto = document.getElementById('inputTexto');
  const btnEnviar = document.getElementById('btnEnviar');

  // 🔹 Traducción automática
  async function traducir(texto, origen, destino) {
    try {
      const url = `https://api.mymemory.translated.net/get?q=${encodeURIComponent(texto)}&langpair=${origen}|${destino}`;
      const res = await fetch(url);
      const data = await res.json();
      return data.responseData.translatedText;
    } catch (e) {
      return '[Error de traducción]';
    }
  }

  // 🔹 Escuchar mensajes nuevos
  db.ref("mensajes").on("child_added", (snap) => {
    const msg = snap.val();
    const mensajeDiv = document.createElement("div");
    mensajeDiv.className = "mensaje";
    mensajeDiv.innerHTML = `
      <div class="idioma1"><b>${msg.origen}:</b> ${msg.texto}</div>
      <div class="idioma2"><b>${msg.destino}:</b> ${msg.traduccion}</div>
    `;
    chatDiv.appendChild(mensajeDiv);
    chatDiv.scrollTop = chatDiv.scrollHeight;
  });

  // 🔹 Enviar mensaje
  btnEnviar.addEventListener("click", async () => {
    const texto = inputTexto.value.trim();
    if (!texto) return;

    // Configura tu rol aquí: "es" si escribes en español, "vi" si escribes en vietnamita
    const miIdioma = "es";
    const otroIdioma = "vi";

    const traduccion = await traducir(texto, miIdioma, otroIdioma);

    db.ref("mensajes").push({
      origen: miIdioma === "es" ? "Español" : "Vietnamita",
      destino: miIdioma === "es" ? "Vietnamita" : "Español",
      texto: texto,
      traduccion: traduccion
    });

    inputTexto.value = "";
    inputTexto.focus();
  });

  inputTexto.addEventListener("keydown", (e) => {
    if (e.key === "Enter") btnEnviar.click();
  });
</script>
</body>
</html>
