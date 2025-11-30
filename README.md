<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<title>Mini Chat Hugging Face</title>
<style>
body { background:#111; color:white; font-family:Arial; padding:20px; }
#chat { border:1px solid #444; padding:10px; height:400px; overflow-y:auto; margin-top:10px; }
.msg { margin:8px 0; }
.user { color:#4da3ff; }
.ai { color:#7dff9b; }
input, select { width:70%; padding:10px; margin-top:10px; }
button { padding:10px; margin-left:5px; margin-top:10px; }
</style>
</head>
<body>

<h2>Mini Chat Hugging Face (один файл)</h2>

<label>Ваш Hugging Face API Token (демо):</label><br>
<input type="text" id="token" placeholder="Введите токен"><br>

<label>Выберите модель:</label><br>
<select id="model">
  <option value="sberbank-ai/rugpt3small_based_on_gpt2">RuGPT-3 Small</option>
  <option value="sberbank-ai/rugpt3medium_based_on_gpt2">RuGPT-3 Medium</option>
  <option value="gpt2">GPT-2 (англ.)</option>
  <option value="EleutherAI/gpt-neo-125M">GPT-Neo 125M</option>
  <option value="flan-t5-small">FLAN-T5 Small</option>
</select>

<div id="chat"></div>

<input id="input" placeholder="Напиши что-нибудь...">
<button onclick="send()">Отправить</button>
<button onclick="voiceInput()">🎤 Говорить</button>

<script>
const chat = document.getElementById("chat");

function addMsg(t,s){
    let d=document.createElement("div");
    d.className="msg "+s;
    d.textContent=(s==="user"?"Ты: ":"ИИ: ")+t;
    chat.appendChild(d);
    chat.scrollTop=chat.scrollHeight;
}

// Запрос к Hugging Face Router API
async function queryHF(message){
    const token = document.getElementById("token").value.trim();
    if(!token) return "Ошибка: токен не введён";
    const model = document.getElementById("model").value;

    try{
        const response = await fetch(https://api-inference.huggingface.co/models/${model}, {
            method: "POST",
            headers: {
                "Authorization": "Bearer " + token,
                "Content-Type": "application/json"
            },
            body: JSON.stringify({ inputs: message })
        });
        const data = await response.json();
        if(data.error) return "Ошибка API: " + data.error;
        if(Array.isArray(data)) return data[0]?.generated_text || "Нет ответа";
        if(data.generated_text) return data.generated_text;
        return "Нет ответа";
    }catch(e){
        return "Ошибка запроса: " + e.message;
    }
}

async function send(){
    const input = document.getElementById("input");
    const text = input.value.trim();
    if(!text) return;
    addMsg(text,"user");
    input.value="";
    const reply = await queryHF(text);
    addMsg(reply,"ai");
    speak(reply);
}

function speak(text){
    let u = new SpeechSynthesisUtterance(text);
    u.lang="ru-RU";
    u.rate=1.05;
    speechSynthesis.speak(u);
}

function voiceInput(){
    let rec = new (window.SpeechRecognition || window.webkitSpeechRecognition)();
    rec.lang="ru-RU";
    rec.onresult = (e)=>{
        let text = e.results[0][0].transcript;
        document.getElementById("input").value = text;
        send();
    };
    rec.start();
}
</script>

</body>
</html>
