<!DOCTYPE html>
<html lang="fa">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>قیمت لحظه‌ای ارز، طلا و سکه</title>

<!-- فونت فارسی Vazirmatn -->
<link href="https://fonts.googleapis.com/css2?family=Vazirmatn:wght@400;700&display=swap" rel="stylesheet">

<!-- Chart.js -->
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>

<style>
/* --- پایه --- */
* { box-sizing: border-box; margin: 0; padding: 0; }
body { font-family: 'Vazirmatn', sans-serif; background: linear-gradient(135deg, #0f2027, #203a43, #2c5364); color: #fff; text-align: center; min-height: 100vh; display: flex; flex-direction: column; }

/* --- هدر --- */
header { padding: 20px; }
h1 { font-size: 2em; margin-bottom: 10px; }

/* --- کارت قیمت‌ها --- */
.prices { display: flex; justify-content: center; flex-wrap: wrap; gap: 15px; margin: 25px 0; }
.card { background: rgba(255,255,255,0.12); padding: 15px 20px; border-radius: 12px; box-shadow: 0 0 12px rgba(0,255,204,0.3); width: 150px; transition: transform 0.2s; }
.card:hover { transform: translateY(-5px); }
.card h3 { font-size: 1.2em; margin-bottom: 5px; }
.card p { font-size: 1.4em; font-weight: bold; }

/* --- دکمه بروزرسانی --- */
#refreshBtn { background: #00ffcc; border: none; padding: 12px 30px; border-radius: 30px; font-size: 16px; cursor: pointer; margin: 10px 0; transition: 0.2s; }
#refreshBtn:hover { background: #00e6b8; }

/* --- نمودار --- */
.chart-container { width: 90%; max-width: 800px; margin: 30px auto; }

/* --- پیشنهاد ویژه --- */
.affiliate-box { margin: 30px auto; width: 90%; max-width: 380px; background: linear-gradient(135deg, #ffdd00, #ff8800); color: #000; padding: 20px; border-radius: 15px; }
.affiliate-box a { display: inline-block; text-decoration: none; background: #000; color: #fff; padding: 10px 15px; border-radius: 10px; font-weight: bold; margin-top: 10px; transition: 0.2s; }
.affiliate-box a:hover { background: #333; }

/* --- تبلیغ پاپ‌آپ --- */
.ad-box { position: fixed; bottom: 15px; left: 50%; transform: translateX(-50%); width: 90%; max-width: 400px; background: #000; border-radius: 15px; display: none; z-index: 999; overflow: hidden; box-shadow: 0 4px 20px rgba(0,0,0,0.5); }
.ad-box img { width: 100%; display: block; }
#adBox span { position: absolute; top: 6px; right: 10px; cursor: pointer; color: #fff; font-size: 20px; }

/* --- فوتر --- */
footer { margin: 30px 0; font-size: 14px; color: #ccc; }

/* --- واکنش‌گرایی --- */
@media(max-width:600px){
  .prices { flex-direction: column; align-items: center; }
  .card { width: 80%; }
}
</style>
</head>
<body>

<header>
  <h1>📈 قیمت لحظه‌ای ارز، طلا و سکه</h1>
</header>

<div class="prices">
  <div class="card"><h3>💵 دلار</h3><p id="usd">---</p></div>
  <div class="card"><h3>🏆 طلا ۱۸</h3><p id="gold">---</p></div>
  <div class="card"><h3>🥇 سکه امامی</h3><p id="coin">---</p></div>
</div>

<button id="refreshBtn">🔄 بروزرسانی</button>

<div class="chart-container">
  <canvas id="priceChart"></canvas>
</div>

<div class="affiliate-box">
  <h3>🔥 پیشنهاد ویژه</h3>
  <p>افزایش سرمایه با ابزارهای مالی</p>
  <a href="https://example.com" target="_blank">همین حالا شروع کن</a>
</div>

<div id="adBox" class="ad-box">
  <span id="closeAd">✖</span>
  <a id="adLink" href="#" target="_blank">
    <img id="adImg">
  </a>
</div>

<footer>🔔 داده‌ها از وب‌سرویس‌های معتبر</footer>

<script>
// --- المان‌ها ---
const usdEl = document.getElementById("usd");
const goldEl = document.getElementById("gold");
const coinEl = document.getElementById("coin");
const ctx = document.getElementById("priceChart").getContext("2d");

// --- داده نمودار ---
const labels = [], usdData = [], goldData = [], coinData = [];
const chart = new Chart(ctx, {
  type: "line",
  data: {
    labels,
    datasets: [
      { label: "دلار", data: usdData, borderColor: "#00ffcc", tension: 0.3, fill: false },
      { label: "طلا", data: goldData, borderColor: "#ffd700", tension: 0.3, fill: false },
      { label: "سکه", data: coinData, borderColor: "#ff69b4", tension: 0.3, fill: false }
    ]
  },
  options: {
    plugins: { legend: { labels: { color: "#fff" } } },
    scales: {
      x: { ticks: { color: "#fff" } },
      y: { ticks: { color: "#fff" } }
    }
  }
});

// --- API واقعی ---
const IR_API = "https://nerkh.io/api/v1/today";
const EX_API = "https://api.exchangerate-api.com/v4/latest/USD";

// --- دریافت داده‌ها ---
async function fetchPrices() {
  try {
    const [irRes, exRes] = await Promise.all([fetch(IR_API), fetch(EX_API)]);
    const irData = await irRes.json();
    const exData = await exRes.json();

    const usdVal = exData.rates.IRR || 0;
    const goldVal = irData.rate?.gold18 || 0;
    const coinVal = irData.rate?.sekkeh || 0;

    usdEl.textContent = Number(usdVal).toLocaleString() + " ریال";
    goldEl.textContent = Number(goldVal).toLocaleString() + " تومان";
    coinEl.textContent = Number(coinVal).toLocaleString() + " تومان";

    addToChart(usdVal, goldVal, coinVal);
  } catch (e) {
    console.error("خطا در دریافت داده‌ها:", e);
  }
}

// --- افزودن به نمودار ---
function addToChart(u, g, c) {
  const time = new Date().toLocaleTimeString("fa-IR");
  labels.push(time); usdData.push(u); goldData.push(g); coinData.push(c);
  if (labels.length > 8) { labels.shift(); usdData.shift(); goldData.shift(); coinData.shift(); }
  chart.update();
}

// --- بروزرسانی دستی ---
document.getElementById("refreshBtn").onclick = fetchPrices;

// --- تبلیغ پاپ‌آپ ---
const ads = [{ img: "https://via.placeholder.com/400x200/00ffcc/000?text=تبلیغ+ویژه", link: "https://example.com" }];
const adBox = document.getElementById("adBox");
const adImg = document.getElementById("adImg");
const adLink = document.getElementById("adLink");
setTimeout(() => {
  const ad = ads[0];
  adImg.src = ad.img;
  adLink.href = ad.link;
  adBox.style.display = "block";
}, 5000);
document.getElementById("closeAd").onclick = () => adBox.style.display = "none";

// --- PWA ساده ---
if ("serviceWorker" in navigator) {
  const swCode = `
    self.addEventListener('install', e => {
      e.waitUntil(caches.open('market-cache').then(c => c.addAll(['./', './index.html'])))
    });
  `;
  const blob = new Blob([swCode], { type: "application/javascript" });
  navigator.serviceWorker.register(URL.createObjectURL(blob));
}

// --- اجرای اولیه ---
fetchPrices();
setInterval(fetchPrices, 60000);
</script>

</body>
</html>
