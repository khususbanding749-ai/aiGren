<!DOCTYPE html>
<html lang="id">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>GrenXharimau AI — Voice Chat</title>
<style>
  * { margin: 0; padding: 0; box-sizing: border-box; }
  body {
    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
    background: #0d1117;
    color: #e6edf3;
    height: 100vh;
    display: flex;
    flex-direction: column;
    overflow: hidden;
  }

  header {
    background: linear-gradient(90deg, #1f6feb, #8957e5);
    padding: 12px 16px;
    display: flex;
    align-items: center;
    gap: 12px;
    box-shadow: 0 2px 10px rgba(0,0,0,0.4);
    flex-shrink: 0;
  }
  .avatar {
    width: 40px; height: 40px;
    border-radius: 50%;
    background: #fff;
    display: flex; align-items: center; justify-content: center;
    font-size: 22px;
    flex-shrink: 0;
  }
  .info h1 { font-size: 15px; font-weight: 600; }
  .info p { font-size: 11px; opacity: 0.9; display: flex; align-items: center; gap: 5px; }
  .dot {
    width: 7px; height: 7px;
    background: #3fb950;
    border-radius: 50%;
    animation: pulse 1.5s infinite;
  }
  @keyframes pulse { 0%,100% { opacity: 1; } 50% { opacity: 0.3; } }

  .toggle-voice {
    margin-left: auto;
    background: rgba(255,255,255,0.2);
    border: none;
    color: #fff;
    width: 36px; height: 36px;
    border-radius: 50%;
    font-size: 16px;
    cursor: pointer;
    transition: background 0.2s;
  }
  .toggle-voice.on { background: #3fb950; }
  .toggle-voice.off { background: #f85149; }

  #chat {
    flex: 1;
    overflow-y: auto;
    padding: 14px;
    display: flex;
    flex-direction: column;
    gap: 10px;
  }
  #chat::-webkit-scrollbar { width: 5px; }
  #chat::-webkit-scrollbar-thumb { background: #30363d; border-radius: 4px; }

  .msg {
    max-width: 85%;
    padding: 11px 15px;
    border-radius: 18px;
    font-size: 14.5px;
    line-height: 1.5;
    white-space: pre-wrap;
    word-wrap: break-word;
    animation: fade 0.25s ease;
  }
  @keyframes fade { from { opacity: 0; transform: translateY(6px); } to { opacity: 1; transform: translateY(0); } }

  .user {
    align-self: flex-end;
    background: linear-gradient(135deg, #1f6feb, #388bfd);
    color: #fff;
    border-bottom-right-radius: 5px;
  }
  .ai {
    align-self: flex-start;
    background: #161b22;
    border: 1px solid #30363d;
    border-bottom-left-radius: 5px;
  }
  .ai .meta {
    font-size: 10px;
    color: #7d8590;
    margin-top: 7px;
    padding-top: 6px;
    border-top: 1px solid #21262d;
    font-style: italic;
  }
  .error {
    align-self: flex-start;
    background: #2d1418;
    border: 1px solid #f85149;
    color: #ff7b72;
    border-bottom-left-radius: 5px;
  }

  .typing {
    align-self: flex-start;
    background: #161b22;
    border: 1px solid #30363d;
    padding: 12px 16px;
    border-radius: 18px;
    border-bottom-left-radius: 5px;
    display: none;
    gap: 5px;
    align-items: center;
  }
  .typing.active { display: flex; }
  .typing span {
    width: 7px; height: 7px;
    background: #7d8590;
    border-radius: 50%;
    animation: bounce 1.3s infinite;
  }
  .typing span:nth-child(2) { animation-delay: 0.15s; }
  .typing span:nth-child(3) { animation-delay: 0.3s; }
  @keyframes bounce {
    0%,60%,100% { transform: translateY(0); opacity: 0.5; }
    30% { transform: translateY(-7px); opacity: 1; }
  }

  .welcome {
    text-align: center;
    color: #7d8590;
    font-size: 13px;
    margin: 30px auto;
    padding: 0 20px;
    max-width: 400px;
  }
  .welcome .big {
    font-size: 52px;
    margin-bottom: 12px;
    animation: float 3s ease-in-out infinite;
  }
  @keyframes float { 0%,100% { transform: translateY(0); } 50% { transform: translateY(-8px); } }
  .welcome .title {
    font-size: 17px;
    color: #e6edf3;
    font-weight: 600;
    margin-bottom: 6px;
  }

  footer {
    background: #161b22;
    border-top: 1px solid #30363d;
    padding: 10px 12px;
    display: flex;
    gap: 8px;
    align-items: center;
    flex-shrink: 0;
  }
  #input {
    flex: 1;
    background: #0d1117;
    border: 1px solid #30363d;
    border-radius: 22px;
    padding: 12px 16px;
    color: #e6edf3;
    font-size: 15px;
    outline: none;
    min-width: 0;
  }
  #input:focus { border-color: #1f6feb; }
  #input::placeholder { color: #6e7681; }

  #mic, #send {
    border: none;
    width: 44px; height: 44px;
    border-radius: 50%;
    color: #fff;
    font-size: 18px;
    cursor: pointer;
    display: flex; align-items: center; justify-content: center;
    flex-shrink: 0;
    transition: transform 0.15s;
  }
  #mic {
    background: #30363d;
    font-size: 20px;
  }
  #mic.recording {
    background: #f85149;
    animation: pulse-red 1s infinite;
  }
  @keyframes pulse-red {
    0%,100% { box-shadow: 0 0 0 0 rgba(248,81,73,0.7); }
    50% { box-shadow: 0 0 0 10px rgba(248,81,73,0); }
  }
  #send {
    background: linear-gradient(135deg, #1f6feb, #8957e5);
  }
  #send:hover:not(:disabled), #mic:hover { transform: scale(1.08); }
  #send:disabled { opacity: 0.4; cursor: not-allowed; }
</style>
</head>
<body>

<header>
  <div class="avatar">🐯</div>
  <div class="info">
    <h1>GrenXharimau AI</h1>
    <p><span class="dot"></span> Voice Mode Aktif</p>
  </div>
  <button class="toggle-voice on" id="voiceToggle" title="Voice ON/OFF">🔊</button>
</header>

<div id="chat">
  <div class="welcome" id="welcome">
    <div class="big">🐯</div>
    <div class="title">Halo! Aku GrenXharimau AI</div>
    <div>Ketik atau tekan 🎤 buat ngomong</div>
    <div style="margin-top:8px;font-size:12px;opacity:0.7;">AI bakal jawab pakai suara 🔊</div>
  </div>
  <div class="typing" id="typing">
    <span></span><span></span><span></span>
  </div>
</div>

<footer>
  <button id="mic" title="Bicara">🎤</button>
  <input id="input" type="text" placeholder="Tulis pesan..." autocomplete="off" />
  <button id="send" title="Kirim">➤</button>
</footer>

<script>
  // ===== KONFIGURASI =====
  const API_URL = "http://ryzzgantenk.ryuuuprvt.my.id:4318/salam";

  // ===== ELEMENT =====
  const chat     = document.getElementById("chat");
  const input    = document.getElementById("input");
  const sendBtn  = document.getElementById("send");
  const micBtn   = document.getElementById("mic");
  const typing   = document.getElementById("typing");
  const welcome  = document.getElementById("welcome");
  const toggle   = document.getElementById("voiceToggle");

  // ===== STATE =====
  let voiceEnabled = true;

  // ===== TEXT-TO-SPEECH =====
  function speak(text) {
    if (!voiceEnabled) return;
    if (!("speechSynthesis" in window)) {
      console.warn("Browser tidak support Text-to-Speech");
      return;
    }

    // Stop suara sebelumnya
    window.speechSynthesis.cancel();

    // Bersihin teks dari emoji & markdown biar suara natural
    const clean = text
      .replace(/[\u{1F300}-\u{1FAFF}]/gu, "")  // hapus emoji
      .replace(/[*_~`#]/g, "")                  // hapus markdown
      .replace(/\n+/g, ". ")                    // newline jadi jeda
      .trim();

    const utter = new SpeechSynthesisUtterance(clean);
    utter.lang = "id-ID";        // Bahasa Indonesia
    utter.rate = 1.0;            // kecepatan normal
    utter.pitch = 1.1;           // agak tinggi biar ramah
    utter.volume = 1.0;

    // Pilih voice Indonesia kalau ada
    const voices = window.speechSynthesis.getVoices();
    const idVoice = voices.find(v => v.lang === "id-ID" || v.lang.startsWith("id"));
    if (idVoice) utter.voice = idVoice;

    window.speechSynthesis.speak(utter);
  }

  // Load voices (kadang async di beberapa browser)
  if ("speechSynthesis" in window) {
    window.speechSynthesis.onvoiceschanged = () => {
      window.speechSynthesis.getVoices();
    };
  }

  // ===== SPEECH-TO-TEXT (Voice Input) =====
  let recognition = null;
  let isRecording = false;

  if ("webkitSpeechRecognition" in window || "SpeechRecognition" in window) {
    const SR = window.SpeechRecognition || window.webkitSpeechRecognition;
    recognition = new SR();
    recognition.lang = "id-ID";
    recognition.continuous = false;
    recognition.interimResults = false;

    recognition.onstart = () => {
      isRecording = true;
      micBtn.classList.add("recording");
      micBtn.textContent = "⏺";
      input.placeholder = "Mendengarkan...";
    };

    recognition.onresult = (e) => {
      const text = e.results[0][0].transcript;
      input.value = text;
      sendMessage();
    };

    recognition.onerror = (e) => {
      console.warn("Speech recognition error:", e.error);
      input.placeholder = "Tulis pesan...";
    };

    recognition.onend = () => {
      isRecording = false;
      micBtn.classList.remove("recording");
      micBtn.textContent = "🎤";
      input.placeholder = "Tulis pesan...";
    };
  } else {
    micBtn.disabled = true;
    micBtn.title = "Browser tidak support voice input";
  }

  // ===== UTILITY =====
  function scrollDown() {
    setTimeout(() => { chat.scrollTop = chat.scrollHeight; }, 50);
  }

  function removeWelcome() {
    if (welcome && welcome.parentNode) welcome.remove();
  }

  function addMessage(text, sender, meta = "") {
    const div = document.createElement("div");
    div.className = "msg " + sender;
    div.textContent = text;
    if (meta) {
      const m = document.createElement("div");
      m.className = "meta";
      m.textContent = meta;
      div.appendChild(m);
    }
    chat.insertBefore(div, typing);
    scrollDown();
    return div;
  }

  // ===== KIRIM PESAN =====
  async function sendMessage(text) {
    text = (text || input.value).trim();
    if (!text) return;

    // Stop suara sebelumnya kalau masih jalan
    if ("speechSynthesis" in window) window.speechSynthesis.cancel();

    removeWelcome();
    addMessage(text, "user");
    input.value = "";
    sendBtn.disabled = true;
    typing.classList.add("active");
    scrollDown();

    const startTime = Date.now();

    try {
      const url = `${API_URL}?pesan=${encodeURIComponent(text)}`;
      const res = await fetch(url);

      if (!res.ok) throw new Error(`HTTP ${res.status}`);

      const data = await res.json();
      const elapsed = ((Date.now() - startTime) / 1000).toFixed(2);

      typing.classList.remove("active");

      if (!data.status) {
        addMessage("⚠️ " + (data.result || "Server menolak request"), "error");
        return;
      }

      const reply = data.result || "(kosong)";
      addMessage(reply, "ai", `${elapsed}s · 🔊`);

      // 🔊 JAWAB PAKAI SUARA
      speak(reply);

    } catch (err) {
      typing.classList.remove("active");
      addMessage(
        "⚠️ Gagal konek ke server.\n\nError: " + err.message +
        "\n\nCek:\n1. Server panel masih running?\n2. URL API benar?",
        "error"
      );
    } finally {
      sendBtn.disabled = false;
      input.focus();
    }
  }

  // ===== EVENT LISTENERS =====

  // Kirim pesan
  sendBtn.addEventListener("click", () => sendMessage());
  input.addEventListener("keydown", (e) => {
    if (e.key === "Enter" && !e.shiftKey) {
      e.preventDefault();
      sendMessage();
    }
  });

  // Mic button
  micBtn.addEventListener("click", () => {
    if (!recognition) return;
    if (isRecording) {
      recognition.stop();
    } else {
      if ("speechSynthesis" in window) window.speechSynthesis.cancel();
      try { recognition.start(); } catch (e) { console.warn(e); }
    }
  });

  // Voice ON/OFF
  toggle.addEventListener("click", () => {
    voiceEnabled = !voiceEnabled;
    if (!voiceEnabled && "speechSynthesis" in window) {
      window.speechSynthesis.cancel();
    }
    toggle.classList.toggle("on", voiceEnabled);
    toggle.classList.toggle("off", !voiceEnabled);
    toggle.textContent = voiceEnabled ? "🔊" : "🔇";
  });

  input.focus();
</script>
</body>
</html>
