<!doctype html>
<html lang="pt-BR">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Interface de Chat — Prompt rápido</title>
  <style>
    :root{--bg:#0f172a;--card:#0b1220;--accent:#06b6d4;--muted:#94a3b8; --bubble:#111827}
    *{box-sizing:border-box;font-family:Inter, system-ui, -apple-system, 'Segoe UI', Roboto, 'Helvetica Neue', Arial}
    body{margin:0;background:linear-gradient(180deg,#061233 0%, #071b2b 100%);color:#e6eef8;min-height:100vh;display:flex;align-items:center;justify-content:center;padding:24px}
    .app{width:100%;max-width:960px;background:var(--card);border-radius:12px;box-shadow:0 8px 30px rgba(2,6,23,.6);overflow:hidden;display:grid;grid-template-columns:360px 1fr}
    .sidebar{padding:18px;border-right:1px solid rgba(255,255,255,0.03)}
    h1{font-size:18px;margin:0 0 12px}
    .samples{display:flex;flex-direction:column;gap:8px;}
    .sample{background:linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01));padding:10px;border-radius:8px;cursor:pointer;border:1px solid rgba(255,255,255,0.02)}
    .sample small{color:var(--muted);display:block;margin-top:6px}
    .main{display:flex;flex-direction:column;height:640px}
    .chat{flex:1;padding:20px;overflow:auto;background:linear-gradient(180deg, rgba(255,255,255,0.01), transparent)}
    .message{max-width:72%;padding:10px 12px;border-radius:12px;margin-bottom:12px;line-height:1.35}
    .user{background:#0b1228;margin-left:auto;border-bottom-right-radius:4px}
    .bot{background:#0f1726;border-bottom-left-radius:4px}
    .composer{padding:12px;border-top:1px solid rgba(255,255,255,0.03);display:flex;gap:8px;align-items:center}
    textarea{flex:1;min-height:64px;max-height:160px;background:transparent;border:1px solid rgba(255,255,255,0.04);color:inherit;padding:10px;border-radius:8px;resize:vertical}
    button{background:var(--accent);border:none;color:#07212b;padding:10px 12px;border-radius:8px;cursor:pointer;font-weight:600}
    .controls{display:flex;gap:8px;margin-top:12px;align-items:center}
    .small{font-size:13px;color:var(--muted)}
    .copy{background:transparent;border:1px solid rgba(255,255,255,0.04);color:var(--muted);padding:8px;border-radius:8px;cursor:pointer}
    footer{font-size:12px;color:var(--muted);padding:10px 18px;border-top:1px solid rgba(255,255,255,0.02)}
  </style>
</head>
<body>
  <div class="app" role="application" aria-label="Interface de Chat com prompts">
    <aside class="sidebar">
      <h1>Prompts de exemplo</h1>
      <div class="samples" id="samples">
        <div class="sample" data-prompt="Você é um assistente simpático que responde em português. Me explique como fazer um site simples em HTML e CSS, passo a passo.">
          <strong>Explicar criação de site (passo a passo)</strong>
          <small>Peça para gerar um tutorial prático, com código.</small>
        </div>
        <div class="sample" data-prompt="Me ajude a escrever uma carta formal em português para solicitar uma certidão de nascimento.">
          <strong>Carta formal — certidão</strong>
          <small>Útil para documentos oficiais.</small>
        </div>
        <div class="sample" data-prompt="Crie um prompt curto para redes sociais que venda um beat de hip-hop de 90 BPM com vibe sombria.">
          <strong>Texto para vender beat (social)</strong>
          <small>Convincente e direto.</small>
        </div>
        <div class="sample" data-prompt="Atue como um tutor de Python. Explique o que são listas e dê 3 exercícios com soluções comentadas.">
          <strong>Tutor de Python — listas</strong>
          <small>Inclui exercícios e respostas.</small>
        </div>
      </div>
      <div style="margin-top:14px">
        <label class="small">Prompt atual (edite se quiser)</label>
        <textarea id="promptEditor" style="width:100%;height:92px;margin-top:8px;border-radius:8px;padding:10px">Você é um assistente simpático que responde em português. Seja objetivo e educado.</textarea>
        <div class="controls">
          <button id="sendBtn">Enviar</button>
          <button class="copy" id="copyBtn">Copiar prompt</button>
          <button class="copy" id="resetBtn">Resetar</button>
        </div>
      </div>
    </aside>

    <section class="main">
      <div class="chat" id="chat" aria-live="polite">
        <div class="message bot">Olá! Cole um prompt à esquerda e clique em "Enviar" para começar a simular uma conversa.</div>
      </div>

      <div class="composer">
        <textarea id="userInput" placeholder="Digite sua mensagem aqui..."></textarea>
        <div style="display:flex;flex-direction:column;gap:8px">
          <button id="sendMsg">Enviar</button>
          <button class="copy" id="clearChat">Limpar</button>
        </div>
      </div>
      <footer>Use este HTML como modelo — integre ao backend que preferir para transformar em chat real.</footer>
    </section>
  </div>

  <script>
    const samples = document.getElementById('samples')
    const promptEditor = document.getElementById('promptEditor')
    const sendBtn = document.getElementById('sendBtn')
    const copyBtn = document.getElementById('copyBtn')
    const resetBtn = document.getElementById('resetBtn')
    const chat = document.getElementById('chat')
    const userInput = document.getElementById('userInput')
    const sendMsg = document.getElementById('sendMsg')
    const clearChat = document.getElementById('clearChat')

    samples.addEventListener('click', e => {
      const el = e.target.closest('.sample')
      if(!el) return
      promptEditor.value = el.dataset.prompt
    })

    copyBtn.addEventListener('click', () => {
      navigator.clipboard.writeText(promptEditor.value).then(()=>{
        alert('Prompt copiado para a área de transferência!')
      })
    })

    resetBtn.addEventListener('click', ()=>{
      promptEditor.value = 'Você é um assistente simpático que responde em português. Seja objetivo e educado.'
    })

    sendBtn.addEventListener('click', ()=>{
      botReply('Prompt aplicado: ' + promptEditor.value)
    })

    sendMsg.addEventListener('click', ()=>{
      const text = userInput.value.trim()
      if(!text) return
      appendMessage(text,'user')
      userInput.value = ''
      // Simulação de resposta — substitua por chamada ao seu backend/OpenAI
      setTimeout(()=>{
        botReply('Resposta simulada para: "' + text + '" — usando prompt: "' + (promptEditor.value).slice(0,120) + (promptEditor.value.length>120? '...':'') + '"')
      },500)
    })

    clearChat.addEventListener('click', ()=>{
      chat.innerHTML = ''
    })

    function appendMessage(text, who){
      const div = document.createElement('div')
      div.className = 'message ' + (who==='user' ? 'user' : 'bot')
      div.textContent = text
      chat.appendChild(div)
      chat.scrollTop = chat.scrollHeight
    }

    function botReply(text){
      appendMessage(text,'bot')
    }
  </script>
</body>
</html>
