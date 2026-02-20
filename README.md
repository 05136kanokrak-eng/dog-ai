[index.html](https://github.com/user-attachments/files/25430413/index.html)
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>Dog Breed AI Pro</title>

<script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs@latest/dist/tf.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/@teachablemachine/image@latest/dist/teachablemachine-image.min.js"></script>

<style>
body{
  font-family: Arial;
  background: linear-gradient(to right,#eef2ff,#f8fafc);
  text-align:center;
}
h1{margin-top:20px}
.card{
  background:white;
  width:380px;
  margin:auto;
  padding:20px;
  border-radius:15px;
  box-shadow:0 8px 20px rgba(0,0,0,0.1);
}
button{
  padding:10px 15px;
  border:none;
  border-radius:8px;
  margin:5px;
  cursor:pointer;
  color:white;
}
.open{background:#4f46e5;}
.capture{background:#16a34a;}
.reset{background:#dc2626;}
canvas,img{border-radius:12px;margin-top:10px;}
.progress{
  background:#eee;
  border-radius:10px;
  height:10px;
  margin:5px 0;
}
.bar{
  height:10px;
  border-radius:10px;
  background:#4f46e5;
}
#breed{
  font-size:22px;
  font-weight:bold;
  margin-top:10px;
}
#desc{
  font-size:14px;
  color:#444;
  margin-top:8px;
}
.stats{
  font-size:13px;
  margin-top:10px;
  color:#555;
}
</style>
</head>

<body>
<h1>🐶 Dog Breed Classifier</h1>

<div class="card">
<button class="open" onclick="init()">📷 เปิดกล้อง</button>
<button class="capture" onclick="capture()">📸 ถ่ายภาพ</button>
<button class="reset" onclick="resetStats()">🗑️ ล้างสถิติ</button>

<div id="webcam-container"></div>
<img id="snapshot" width="250">

<div id="breed"></div>
<div id="desc"></div>
<div id="bars"></div>
<div class="stats" id="stats"></div>
</div>

<script>

const URL = "https://teachablemachine.withgoogle.com/models/FqRUS6uDf/";

let model, webcam;

const dogInfo = {
  "American bulldog":
    "สุนัขแข็งแรง กล้ามเนื้อแน่น ซื่อสัตย์ เหมาะเฝ้าบ้าน",
  "Beagle":
    "ร่าเริง ดมกลิ่นเก่ง ใช้ล่าสัตว์",
  "Labrador":
    "เป็นมิตร ฉลาด เหมาะกับครอบครัว",
  "Chihuahua":
    "ตัวเล็ก กล้าหาญ ติดเจ้าของ",
  "Siberian Husky":
    "พลังงานสูง ขนหนา ชอบอากาศเย็น"
};

async function init(){
  const modelURL = URL + "model.json";
  const metadataURL = URL + "metadata.json";
  model = await tmImage.load(modelURL, metadataURL);

  webcam = new tmImage.Webcam(250,250,true);
  await webcam.setup();
  await webcam.play();

  document.getElementById("webcam-container").innerHTML="";
  document.getElementById("webcam-container").appendChild(webcam.canvas);
}

async function capture(){
  if(!model || !webcam) return alert("กรุณาเปิดกล้องก่อน");

  // แสดงภาพ freeze
  const snapshot = document.getElementById("snapshot");
  snapshot.src = webcam.canvas.toDataURL("image/png");

  const prediction = await model.predict(webcam.canvas);
  prediction.sort((a,b)=>b.probability-a.probability);

  const best = prediction[0].className;
  const percent = (prediction[0].probability*100).toFixed(1);

  document.getElementById("breed").innerText =
    `พันธุ์: ${best} (${percent}%)`;

  document.getElementById("desc").innerText =
    dogInfo[best] || "ไม่มีข้อมูล";

  // progress bar
  let barsHTML="";
  prediction.forEach(p=>{
    const pr=(p.probability*100).toFixed(1);
    barsHTML+=`
      <div>${p.className} ${pr}%</div>
      <div class="progress">
        <div class="bar" style="width:${pr}%"></div>
      </div>
    `;
  });
  document.getElementById("bars").innerHTML=barsHTML;

  saveStats(best,parseFloat(percent));
  showStats();
}

function saveStats(breed,score){
  let data=JSON.parse(localStorage.getItem("dogStats"))||{};
  if(!data[breed]){
    data[breed]={count:0,total:0};
  }
  data[breed].count+=1;
  data[breed].total+=score;
  localStorage.setItem("dogStats",JSON.stringify(data));
}

function showStats(){
  let data=JSON.parse(localStorage.getItem("dogStats"))||{};
  let html="";
  for(let b in data){
    let avg=(data[b].total/data[b].count).toFixed(1);
    html+=`${b} → ${data[b].count} ครั้ง | เฉลี่ย ${avg}%<br>`;
  }
  document.getElementById("stats").innerHTML=html;
}

function resetStats(){
  localStorage.removeItem("dogStats");
  showStats();
}

showStats();

</script>
</body>
</html>
