<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Clear Dictation Tool (Exam Friendly)</title>

<style>
body{
  font-family: system-ui, sans-serif;
  background:#0f172a;
  color:#e5e7eb;
  margin:0;
  padding:16px;
}
h1{text-align:center;font-size:22px;}
textarea{
  width:100%;min-height:180px;
  background:#020617;color:#e5e7eb;
  border:1px solid #334155;border-radius:12px;
  padding:12px;font-size:15px;line-height:1.7;
}
.controls{display:flex;gap:10px;margin-top:14px;}
button{
  flex:1;padding:12px;font-size:15px;
  border-radius:14px;border:2px solid #38bdf8;
  background:transparent;color:#38bdf8;
}
button.primary{border-color:#22c55e;color:#22c55e;}
button.danger{border-color:#ef4444;color:#ef4444;}
.status{text-align:center;margin-top:10px;font-size:14px;}
</style>
</head>

<body>

<h1>✍️ Clear Dictation (Writing Mode)</h1>

<textarea id="inputText" placeholder="Paste text here. Clear pronunciation, slow speed, writing-friendly."></textarea>

<div class="controls">
  <button class="primary" onclick="startDictation()">▶ Start</button>
  <button onclick="pauseDictation()">⏸ Pause</button>
  <button onclick="resumeDictation()">▶ Resume</button>
  <button class="danger" onclick="stopDictation()">⛔ Stop</button>
</div>

<div class="status" id="status">Idle</div>

<script>
let state="IDLE", lines=[], li=0, phrases=[], pi=0, timer=null;

speechSynthesis.getVoices();

function splitPhrases(line){
  let w=line.trim().split(/\s+/), r=[];
  for(let i=0;i<w.length;i+=3){ // 🔴 max 3 words
    r.push(w.slice(i,i+3).join(" "));
  }
  return r;
}

function isHardWord(text){
  return /\b[A-Za-z]{9,}\b/.test(text);
}

function speak(text, cb){
  let u=new SpeechSynthesisUtterance(text);

  let v=speechSynthesis.getVoices()
    .find(v=>v.lang==="en-IN") ||
    speechSynthesis.getVoices().find(v=>v.lang.startsWith("en"));

  if(v) u.voice=v;

  u.rate=0.65;        // 🔴 VERY SLOW
  u.pitch=0.95;      // 🔴 flat teacher tone
  u.volume=1;

  u.onend=cb;
  speechSynthesis.speak(u);
}

function nextPhrase(){
  if(state!=="PLAYING") return;
  if(pi>=phrases.length){li++;pi=0;nextLine();return;}

  let p=phrases[pi++];
  speak(p,()=>{
    let extra = isHardWord(p) ? 4000 : 0; // hard word extra pause
    timer=setTimeout(nextPhrase,5000+extra);
  });
}

function nextLine(){
  if(state!=="PLAYING") return;
  if(li>=lines.length){state="IDLE";status("Done");return;}

  let l=lines[li].trim();
  if(l===""){li++;timer=setTimeout(nextLine,2000);return;}

  phrases=splitPhrases(l);
  pi=0;
  nextPhrase();
}

function status(t){document.getElementById("status").innerText=t;}

function startDictation(){
  stopDictation();
  let t=inputText.value;
  if(!t.trim()) return alert("Paste text first");
  lines=t.split(/\n/);
  li=0;pi=0;state="PLAYING";
  status("Dictating...");
  nextLine();
}
function pauseDictation(){
  if(state!=="PLAYING") return;
  speechSynthesis.cancel();clearTimeout(timer);
  state="PAUSED";status("Paused");
}
function resumeDictation(){
  if(state!=="PAUSED") return;
  state="PLAYING";status("Resumed");
  nextPhrase();
}
function stopDictation(){
  speechSynthesis.cancel();clearTimeout(timer);
  state="STOPPED";status("Stopped");
}
</script>

</body>
</html>
