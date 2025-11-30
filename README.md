<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<title>Гибридный мини ИИ</title>
<style>
body { background:#111; color:white; font-family:Arial; padding:20px; }
#chat { border:1px solid #444; padding:10px; height:400px; overflow-y:auto; margin-top:10px; }
.msg { margin:8px 0; }
.user { color:#4da3ff; }
.ai { color:#7dff9b; }
input { width:70%; padding:10px; margin-top:10px; }
button { padding:10px; margin-left:5px; margin-top:10px; }
</style>
</head>
<body>

<h2>Гибридный мини ИИ</h2>

<div id="chat"></div>

<input id="input" placeholder="Напиши что-нибудь...">
<button onclick="send()">Отправить</button>
<button onclick="voiceInput()">🎤 Говорить</button>

<script>
const chat = document.getElementById("chat");

// ==========================
// Шаблонные ответы офлайн
// ==========================
const responses = {
  "привет": ["Привет! Как дела?", "Здравствуйте!", "Привет, рад тебя видеть!"],
  "как дела": ["Всё отлично, а у тебя?", "Нормально, спасибо!", "Хорошо, спасибо за вопрос."],
  "что умеешь": ["Я могу общаться и отвечать на вопросы.", "Я умею вести беседу и рассказывать истории."],
  "пока": ["Пока! До встречи.", "До свидания!", "Увидимся позже!"]
};

function addMsg(t,s){
    let d=document.createElement("div");
    d.className="msg "+s;
    d.textContent=(s==="user"?"Ты: ":"ИИ: ")+t;
    chat.appendChild(d);
    chat.scrollTop=chat.scrollHeight;
}

function isQuestion(input){
    const qwords = ["что","почему","как","когда","зачем","где"];
    input = input.toLowerCase();
    return qwords.some(w => input.includes(w));
}

// ==========================
// Имитация поиска (здесь нужно будет сервер для реального Google)
// ==========================
async function searchGoogle(query){
    // Демо: возвращаем фразу "нашёл в интернете"
    // На реальном сервере можно делать fetch к Google Custom Search API
    return "Поиск в интернете: " + query;
}

// ==========================
// Генерация ответа
// ==========================
async function generateAnswer(input){
    input = input.toLowerCase();
    for(let key in responses){
        if(input.includes(key)){
            return responses[key][Math.floor(Math.random()*responses[key].length)];
        }
    }
    if(isQuestion(input)){
        return await searchGoogle(input);
    }
    return "Интересно! Расскажи подробнее.";
}

// ==========================
// Отправка сообщения
// ==========================
async function send(){
    const input = document.getElementById("input");
    const text = input.value.trim();
    if(!text) return;
    addMsg(text,"user");
    input.value="";
    const reply = await generateAnswer(text);
    addMsg(reply,"ai");
    speak(reply);
}

// ==========================
// Голосовой синтез
// ==========================
function speak(text){
    let u = new SpeechSynthesisUtterance(text);
    u.lang="ru-RU";
    u.rate=1.05;
    speechSynthesis.speak(u);
}

// ==========================
// Голосовой ввод
// ==========================
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
