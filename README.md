
<!DOCTYPE html>
<!--
  idea_3.html - ไซโลข้าวเปลือก: ภาพตัดไซโล สีเมล็ดตามความชื้น จุดรา และเส้นแนวโน้ม RH
  เปิดไฟล์แล้วจะเลือก team19 ให้อัตโนมัติ (เปลี่ยนทีมได้ในช่องบนหน้า)
  ฟัง bento-aiot/<ทีม>/# (บอร์ดส่ง /event เมื่อระดับเปลี่ยน) ส่งคำสั่งเข้า bento-aiot/<ทีม>/cmd
  คีย์จากบอร์ด: id crop level alert temp_c rh sun_c + wet_s rh_max (ดู project/idea_3_silo.py)
  กฎฝั่งแอป: level 2 = ป้ายแดง "เสี่ยงรา เปิดพัดลมไซโล" ค้างจนกว่าจะรับทราบหรือระดับลด
  broker สาธารณะ ไม่มีรหัสผ่าน ใครก็อ่านและสั่งได้ ห้ามส่งของลับ
-->
<html lang="th">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>ไซโลข้าวเปลือก</title>
<style>
  :root {
    color-scheme: light dark;
    --bg: #fbf6ec; --card: #fffdf8; --line: #e6d8bd; --ink: #3b2f20; --dim: #7a6548;
    --grain: #d9a83c; --ok: #5f8f52; --warn: #c9772b; --bad: #c0442a; --steel: #b9b2a6;
    --font: system-ui, -apple-system, "Segoe UI", "Noto Sans Thai", "Sarabun", sans-serif;
  }
  @media (prefers-color-scheme: dark) {
    :root { --bg: #1b150e; --card: #2a2118; --line: #5c4a33; --ink: #fff3dc; --dim: #bfa88a;
      --grain: #e9c46a; --ok: #8ab17d; --warn: #f4a261; --bad: #e76f51; --steel: #6b6257; }
  }
  * { box-sizing: border-box; }
  body { margin: 0; background: var(--bg); color: var(--ink); font-family: var(--font); line-height: 1.5; }
  .wrap { max-width: 900px; margin: 0 auto; padding: 16px; }
  header { display: flex; flex-wrap: wrap; justify-content: space-between; gap: 10px; align-items: center; }
  h1 { margin: 0; font-size: 21px; }
  h2 { margin: 0 0 8px; font-size: 16px; color: var(--grain); }
  .sub { margin: 0; color: var(--dim); font-size: 14px; }
  .layout { display: grid; grid-template-columns: 220px minmax(0, 1fr); gap: 12px; margin-top: 12px; }
  .card { background: var(--card); border: 1px solid var(--line); border-radius: 18px; padding: 14px; min-width: 0; }
  .rh { font-size: 52px; font-weight: 800; line-height: 1; }
  .kv { display: grid; grid-template-columns: auto 1fr; gap: 4px 12px; font-size: 15px; margin-top: 10px; }
  .kv dt { color: var(--dim); }
  .kv dd { margin: 0; font-weight: 600; }
  #alarm { display: none; margin-top: 12px; padding: 12px; border-radius: 18px; background: var(--bad); color: #fff; font-weight: 700; }
  #alarm.on { display: block; }
  .row { display: flex; flex-wrap: wrap; gap: 8px; align-items: center; }
  button, input { font: inherit; min-height: 40px; padding: 8px 12px; border-radius: 999px; border: 1px solid var(--line);
    background: var(--card); color: var(--ink); }
  button.main { background: var(--grain); border-color: var(--grain); color: #2a2118; font-weight: 700; }
  input[type=number] { width: 80px; }
  #trendNote { font-size: 14px; }
  .stats { display: grid; grid-template-columns: repeat(4, minmax(0, 1fr)); gap: 10px; margin-top: 12px; }
  .stat { background: var(--card); border: 1px solid var(--line); border-radius: 14px; padding: 12px; min-width: 0; }
  .stat .label { color: var(--dim); font-size: 13px; }
  .stat .value { font-size: 23px; font-weight: 800; overflow-wrap: anywhere; }
  .status-pill { display: inline-block; border: 1px solid var(--line); border-radius: 999px; padding: 3px 9px; font-size: 12px; }
  #historyMsg { font-size: 13px; color: var(--dim); }
  footer { margin: 18px 0 8px; color: var(--dim); font-size: 13px; }
  .project-footer { margin-top: 22px; padding-top: 0; font-size: 14px; }
  .footer-rule { border-top: 1px solid #aaa; margin: 18px 0 22px; }
  .project-footer h3 { font-size: 17px; color: var(--dim); margin: 0 0 16px; }
  .project-footer p { margin: 10px 0; }
  .footer-features-title { margin-top: 16px !important; }
  .footer-features { margin: 8px 0 18px; padding-left: 48px; }
  .footer-features li { padding-left: 4px; margin: 2px 0; }
  .footer-credit { text-align: center; margin-top: 18px !important; }
  .footer-note { font-size: 12px; opacity: .85; }
  @media (max-width: 640px) {
    .layout { grid-template-columns: 1fr; }
    #siloCard svg { height: 220px; }
    .stats { grid-template-columns: repeat(2, minmax(0, 1fr)); }
  }
</style>
</head>
<body>
<div class="wrap">
<header>
  <div>
    <h1>ไซโลข้าวเปลือก</h1>
    <p class="sub">ความชื้นในไซโล เวลาเสี่ยงรา และแนวโน้ม</p>
  </div>
  <div class="row">
    <input id="team" size="10" placeholder="team19" aria-label="ชื่อทีม">
    <button id="go">ต่อ</button>
    <span id="conn" class="sub">ยังไม่ต่อ</span>
  </div>
</header>
<div id="alarm" role="alert"></div>

<div class="layout">
  <section class="card" id="siloCard">
    <h2>ภาพตัดไซโล</h2>
    <svg viewBox="0 0 160 260" width="100%" height="280" aria-label="ไซโล">
      <defs><clipPath id="body"><rect x="30" y="50" width="100" height="190" rx="6"/></clipPath></defs>
      <polygon points="24,52 80,8 136,52" fill="var(--steel)"/>
      <rect x="30" y="50" width="100" height="190" rx="6" fill="none" stroke="var(--steel)" stroke-width="4"/>
      <g clip-path="url(#body)">
        <rect id="fill" x="30" y="120" width="100" height="120" fill="var(--grain)"/>
        <g id="mold"></g>
      </g>
      <line id="hiLine" x1="22" x2="138" y1="0" y2="0" stroke="var(--bad)" stroke-width="2" stroke-dasharray="5 4"/>
      <text id="hiTxt" x="140" y="0" font-size="10" fill="var(--bad)">70%</text>
      <rect x="40" y="240" width="10" height="16" fill="var(--steel)"/><rect x="110" y="240" width="10" height="16" fill="var(--steel)"/>
    </svg>
    <p class="sub">ระดับเมล็ดในภาพ = ความชื้น % (เส้นแดง = เกณฑ์ราขึ้น)</p>
  </section>

  <section class="card">
    <h2>ความชื้นตอนนี้</h2>
    <div class="rh"><span id="rh">--</span><small> %RH</small></div>
    <div id="mood" class="sub">รอข้อความจากบอร์ด</div>
    <dl class="kv">
      <dt>เวลาเสี่ยงรา</dt><dd id="wet">-</dd>
      <dt>สูงสุดตั้งแต่เริ่ม</dt><dd id="rhmax">-</dd>
      <dt>อุณหภูมิ</dt><dd id="temp">-</dd>
      <dt>เมล็ด</dt><dd id="crop">-</dd>
    </dl>
    <h2 style="margin-top:14px">แนวโน้ม RH (ข้อมูลจากบอร์ดทุก 5 วินาที)</h2>
    <svg id="trend" viewBox="0 0 300 90" width="100%" height="90" aria-label="แนวโน้มความชื้น">
      <line id="trendHi" x1="0" x2="300" y1="27" y2="27" stroke="var(--bad)" stroke-dasharray="4 3"/>
      <polyline id="trendLine" fill="none" stroke="var(--grain)" stroke-width="3" points=""/>
    </svg>
    <p id="trendNote" class="sub"></p>
    <p id="fastRiseAlert" class="sub" role="status" style="margin-top:8px;padding:10px;border-radius:10px">รอตรวจสอบแนวโน้มความชื้น</p>
  </section>
</div>

<section class="stats" aria-label="สรุปข้อมูล">
  <div class="stat"><div class="label">RH เฉลี่ย</div><div class="value" id="avgRh">--</div><div class="sub">จากข้อมูลที่บันทึก</div></div>
  <div class="stat"><div class="label">RH สูงสุด</div><div class="value" id="maxRh">--</div><div class="sub">จากข้อมูลที่บันทึก</div></div>
  <div class="stat"><div class="label">อุณหภูมิเฉลี่ย</div><div class="value" id="avgTemp">--</div><div class="sub">°C</div></div>
  <div class="stat"><div class="label">จำนวนข้อมูล</div><div class="value" id="sampleCount">0</div><div class="sub">รายการ</div></div>
  <div class="stat"><div class="label">Quality Score*</div><div class="value" id="qualityScore">--</div><div class="sub">คะแนนประเมินเบื้องต้น / 100</div></div>
  <div class="stat"><div class="label">แจ้งเตือนระดับสูง</div><div class="value" id="alertCount">0</div><div class="sub">จากประวัติที่บันทึก</div></div>
</section>
<p class="sub" style="margin-top:6px">*คะแนนเป็นเกณฑ์ประเมินเพื่อการเรียนรู้ ไม่ใช่มาตรฐานรับรองคุณภาพข้าว</p>

<section class="card" style="margin-top:12px" id="userAndThreshold">
  <h2>ออกแบบเพื่อผู้ใช้จริง</h2>
  <p><strong>ผู้ใช้เป้าหมาย:</strong> ผู้ดูแลไซโลและคนขับรถขนส่งข้าวเปลือก</p>
  <p>ภาษาที่ใช้เน้นสั้นและลงมือทำได้ทันที: <strong>“ความชื้นสูง โปรดแจ้งผู้ดูแลตรวจสอบไซโล”</strong> เมื่อระบบขึ้นระดับเสี่ยง ให้ตรวจสภาพจริงก่อนตัดสินใจเปิดพัดลมหรือย้ายข้าว</p>
  <h2 style="margin-top:14px">บันทึกหลักฐานตั้งเกณฑ์ RH ของทีม</h2>
  <p class="sub">กรอกค่าที่วัดจริง ณ จุดติดตั้ง 3 ครั้งก่อนนำเสนอ ห้ามใส่ค่าคาดเดาแทนค่าที่วัดได้</p>
  <div class="row">
    <label>ครั้งที่ 1 (%) <input id="rhRead1" type="number" min="0" max="100" step="0.1" placeholder="ค่าที่วัดจริง"></label>
    <label>ครั้งที่ 2 (%) <input id="rhRead2" type="number" min="0" max="100" step="0.1" placeholder="ค่าที่วัดจริง"></label>
    <label>ครั้งที่ 3 (%) <input id="rhRead3" type="number" min="0" max="100" step="0.1" placeholder="ค่าที่วัดจริง"></label>
    <button id="calcReadings">คำนวณค่าเฉลี่ย</button>
  </div>
  <p id="readingResult" class="sub" aria-live="polite">ยังไม่ได้บันทึกค่าที่วัดจริง 3 ครั้ง</p>
  <p class="footer-note">เหตุผลของเกณฑ์: 70% RH เป็นค่าเริ่มต้นเพื่อทดลองในชั้นเรียน ไม่ใช่มาตรฐานรับรองคุณภาพข้าว เมื่อมีผลวัดจริง ให้ทีมอธิบายว่าเกณฑ์นี้ใช้แยกสภาวะปกติออกจากช่วงที่ควรตรวจสอบเพิ่มเติม และปรับตามข้อมูลหน้างาน/คำแนะนำผู้เชี่ยวชาญ</p>
</section>

<section class="card" style="margin-top:12px">
  <h2>สั่งไซโล</h2>
  <div class="row">
    <button class="main" id="ack">รับทราบ เปิดพัดลมแล้ว (ack)</button>
    <button id="notify">เปิดการแจ้งเตือน</button>
    <button id="clearHistory">ล้างประวัติในเครื่อง</button>
    <input id="thi" type="number" min="16" max="45" value="35" aria-label="t_hi"><button id="set">ตั้งเกณฑ์ร้อน t_hi</button>
    <input id="rhhi" type="number" min="40" max="90" value="70" aria-label="rh_hi"><button id="setRh">ตั้งเกณฑ์ชื้น rh_hi</button>
    <button id="csv">ดาวน์โหลด CSV</button>
  </div>
  <p class="sub" id="cmdMsg"></p>
  <p id="historyMsg">ประวัติจะถูกเก็บไว้ในเบราว์เซอร์เครื่องนี้ สูงสุด 500 รายการต่อทีม</p>
</section>

<footer class="project-footer">
  <div class="footer-rule"></div>
  <h3>🌾 Smart Paddy Silo Monitoring System</h3>
  <p>ระบบติดตามความชื้นในไซโลข้าวเปลือกแบบเรียลไทม์</p>
  <p class="footer-features-title">ฟีเจอร์ที่พัฒนาต่อยอด</p>
  <ul class="footer-features">
    <li>✅ ปรับค่า RH Threshold ผ่าน MQTT</li>
    <li>✅ วิเคราะห์แนวโน้มความชื้นย้อนหลัง</li>
    <li>✅ คำนวณ Quality Score ของไซโล</li>
    <li>✅ นับจำนวนการแจ้งเตือนสะสม</li>
    <li>✅ ตรวจจับบอร์ดออฟไลน์อัตโนมัติ</li>
    <li>✅ ประเมินความเสี่ยงการเกิดเชื้อรา</li>
    <li>✅ บันทึกประวัติข้อมูลและดาวน์โหลด CSV</li>
  </ul>
  <p class="footer-credit">Developed by team19 | AIoT Smart Farm Project 2026</p>
  <p class="footer-note">เกณฑ์ RH เริ่มต้น 70% เป็นค่าทดลองในชั้นเรียน ไม่ใช่คำแนะนำทางเกษตร</p>
  <p class="footer-note">ผู้ใช้เป้าหมาย: ผู้ดูแลไซโลและคนขับรถขนส่งข้าวเปลือก โดยใช้คำเตือนสั้น ๆ ที่นำไปปฏิบัติได้</p>
  <p class="footer-note">ต่อยอดจากโครงงานเริ่มต้น โดยใช้ MQTT สำหรับรับข้อมูลจากบอร์ด</p>
</footer>
</div>

<script src="https://cdn.jsdelivr.net/npm/mqtt@5.16.0/dist/mqtt.min.js"></script>
<script>
const BROKER_URL = "wss://broker.hivemq.com:8884/mqtt";
const ROOT = "bento-aiot";
const LEVEL_TH = ["แห้งดี", "เริ่มชื้น", "เสี่ยงรา"];
const LEVEL_VAR = ["--ok", "--warn", "--bad"];
const ALERT_TH = { ok: "แห้งดี", cold: "ไซโลเย็น", hot: "ไซโลร้อน", dry: "แห้งไป", wet: "ชื้นเกิน" };
// ชื่อไทยและ RH สูงของแต่ละเมล็ด ตรงกับ CROPS ในบอร์ด
const CROPS = { paddy: ["ข้าวเปลือก", 70], seed: ["เมล็ดพันธุ์", 65], corn: ["ข้าวโพด", 70] };
const $ = (id) => document.getElementById(id);
let rows = [], history_rh = [];
let client = null, team = "", rhHi = 70;
const MAX_ROWS = 500;
let lastAlertKey = "";
let notificationEnabled = false;
let lastMessageAt = 0;

function storageKey(t) { return "smart-silo-history-" + t; }
function loadHistory(t) {
  try {
    const saved = JSON.parse(localStorage.getItem(storageKey(t)) || "[]");
    rows = Array.isArray(saved) ? saved.slice(-MAX_ROWS) : [];
    history_rh = rows.map(r => num(r[5])).filter(v => v !== null).slice(-100);
  } catch (_) { rows = []; history_rh = []; }
  renderStats();
  drawTrend();
}
function saveHistory() {
  try {
    localStorage.setItem(storageKey(team), JSON.stringify(rows.slice(-MAX_ROWS)));
    $("historyMsg").textContent = "บันทึกประวัติในเบราว์เซอร์แล้ว (" + rows.length + " รายการ)";
  } catch (_) {
    $("historyMsg").textContent = "บันทึกไม่ได้ อาจเป็นเพราะพื้นที่เบราว์เซอร์เต็มหรือปิดการจัดเก็บข้อมูล";
  }
}
function renderStats() {
  const rhs = rows.map(r => num(r[5])).filter(v => v !== null);
  const temps = rows.map(r => num(r[4])).filter(v => v !== null);
  $("avgRh").textContent = rhs.length ? (rhs.reduce((a,b) => a+b, 0) / rhs.length).toFixed(1) + "%" : "--";
  $("maxRh").textContent = rhs.length ? Math.max(...rhs).toFixed(1) + "%" : "--";
  $("avgTemp").textContent = temps.length ? (temps.reduce((a,b) => a+b, 0) / temps.length).toFixed(1) : "--";
  $("sampleCount").textContent = String(rows.length);
  $("alertCount").textContent = String(rows.filter(r => Number(r[2]) === 2 && String(r[1] || "").endsWith("/event")).length);
  const scored = rows.filter(r => num(r[5]) !== null && num(r[4]) !== null);
  if (scored.length) {
    const good = scored.filter(r => num(r[5]) <= rhHi && num(r[4]) >= 15 && num(r[4]) <= 35).length;
    $("qualityScore").textContent = Math.round(100 * good / scored.length);
  } else {
    $("qualityScore").textContent = "--";
  }
}
function notifyRisk(level, why) {
  const key = level + ":" + why;
  if (level !== 2 || key === lastAlertKey) return;
  lastAlertKey = key;
  if (notificationEnabled && "Notification" in window && Notification.permission === "granted") {
    new Notification("แจ้งเตือนไซโลข้าวเปลือก", { body: "เสี่ยงรา: " + why + " — กรุณาตรวจสอบไซโล" });
  }
}

function num(v) { return (typeof v === "number" && Number.isFinite(v)) ? v : null; }
function str(v, max) { return typeof v === "string" ? v.slice(0, max || 40) : ""; }
function cssVar(n) { return getComputedStyle(document.documentElement).getPropertyValue(n).trim(); }
function okTeam(t) { return /^team[0-9a-z_]{2,16}$/i.test(t) && t !== "teamXX" && t !== "team00"; }
function alertTh(code) { return str(code, 60).split("+").map((c) => ALERT_TH[c] || c).join(" + "); }
function setConn(t, ok) { $("conn").textContent = t; $("conn").style.color = cssVar(ok ? "--ok" : "--warn"); }
function siloY(rh) { return 240 - Math.max(0, Math.min(100, rh)) * 1.9; }    // 0-100 % -> y ในตัวไซโล

function drawHi() {
  const y = siloY(rhHi);
  $("hiLine").setAttribute("y1", y); $("hiLine").setAttribute("y2", y);
  $("hiTxt").setAttribute("y", y - 3); $("hiTxt").textContent = rhHi + "%";
  const ty = 90 - rhHi * 0.9;
  $("trendHi").setAttribute("y1", ty); $("trendHi").setAttribute("y2", ty);
}

function drawSilo(rh, level) {
  const y = siloY(rh);
  $("fill").setAttribute("y", y.toFixed(1));
  $("fill").setAttribute("height", (240 - y).toFixed(1));
  $("fill").setAttribute("fill", level === 1 ? "var(--warn)" : "var(--grain)");   // เสี่ยงรา = เมล็ดสีเดิม + จุดรา
  const g = $("mold");
  while (g.firstChild) g.firstChild.remove();
  if (level < 2) return;
  for (let i = 0; i < 9; i++) {                 // จุดราสุ่มในเนื้อเมล็ด เมื่อเสี่ยงรา
    const c = document.createElementNS("http://www.w3.org/2000/svg", "circle");
    c.setAttribute("cx", (38 + Math.random() * 84).toFixed(1));
    c.setAttribute("cy", (y + 8 + Math.random() * Math.max(4, 228 - y)).toFixed(1));
    c.setAttribute("r", (2 + Math.random() * 4).toFixed(1));
    c.setAttribute("fill", "#2f4f2f"); c.setAttribute("opacity", "0.7");
    g.appendChild(c);
  }
}

function drawTrend() {
  const pts = history_rh.slice(-20);
  const step = pts.length > 1 ? 300 / (pts.length - 1) : 0;
  $("trendLine").setAttribute("points", pts.map((v, i) => (i * step).toFixed(1) + "," + (90 - v * 0.9).toFixed(1)).join(" "));
  const n = pts.length;
  if (n >= 3 && pts[n - 1] > pts[n - 2] && pts[n - 2] > pts[n - 3]) {
    $("trendNote").textContent = "ชื้นขึ้นต่อเนื่อง 3 ครั้งล่าสุด: ตรวจหลังคาและพัดลมไซโล";
    $("trendNote").style.color = cssVar("--warn");
  } else { $("trendNote").textContent = n ? "ยังไม่เห็นแนวโน้มชื้นขึ้นต่อเนื่อง" : ""; $("trendNote").style.color = ""; }
}

function mmss(sec) { return Math.floor(sec / 60) + " นาที " + (sec % 60) + " วินาที"; }

// กฎฝั่งแอป: เสี่ยงรา (level 2) = ป้ายแดงค้าง
function alarm(on, why) {
  $("alarm").className = on ? "on" : "";
  $("alarm").textContent = on ? "เสี่ยงรา: " + why + "  เปิดพัดลมไซโล แล้วกดรับทราบ" : "";
  if (on && navigator.vibrate) navigator.vibrate([300, 150, 300]);
}

// ฟีเจอร์ต่อยอด: แจ้งเตือนเมื่อ RH เพิ่มขึ้นอย่างรวดเร็ว
// เปรียบเทียบกับข้อมูลก่อนหน้าในช่วง 5-15 วินาที ไม่ได้ยืนยันว่าข้าวเกิดเชื้อรา
function detectFastRise(currentRh) {
  const box = $("fastRiseAlert");
  if (currentRh === null) {
    box.textContent = "ยังไม่มีข้อมูลความชื้น";
    return;
  }

  const previous = [...rows].reverse().find(row => num(row[5]) !== null);
  if (!previous) {
    box.textContent = "กำลังเก็บข้อมูลเพื่อวิเคราะห์แนวโน้ม";
    return;
  }

  const oldTime = new Date(previous[0]).getTime();
  const now = Date.now();
  const elapsed = (now - oldTime) / 1000;
  const oldRh = num(previous[5]);
  if (!Number.isFinite(oldTime) || oldRh === null || elapsed <= 0) {
    box.textContent = "กำลังรอข้อมูลความชื้นชุดถัดไป";
    return;
  }

  const rise = currentRh - oldRh;
  if (elapsed >= 5 && elapsed <= 15 && rise >= 5) {
    box.textContent = "⚠️ RH เพิ่มขึ้น " + rise.toFixed(1) +
      " จุดใน " + elapsed.toFixed(0) + " วินาที โปรดตรวจสอบไซโล";
    box.style.background = "#fce4df";
    box.style.color = "#9b2416";
  } else {
    box.textContent = "แนวโน้มล่าสุด: " + (rise >= 0 ? "+" : "") +
      rise.toFixed(1) + " จุด RH ใน " + elapsed.toFixed(0) + " วินาที";
    box.style.background = "";
    box.style.color = "";
  }
}

function onEvent(topic, d) {
  const level = num(d.level);
  if (level === null || level < 0 || level > 2) return;
  const rh = num(d.rh), t = num(d.temp_c), wet = num(d.wet_s), rmax = num(d.rh_max), sun = num(d.sun_c);
  const crop = str(d.crop, 20);
  if (!window._rhThresholdCustomized && CROPS[crop] && CROPS[crop][1] !== rhHi) {
    rhHi = CROPS[crop][1]; $("rhhi").value = rhHi; drawHi();
  }
  $("crop").textContent = CROPS[crop] ? CROPS[crop][0] : (crop || "-");
  const why = alertTh(d.alert);
  if (rh !== null) { $("rh").textContent = rh.toFixed(0); drawSilo(rh, level); history_rh.push(rh); drawTrend(); }
  detectFastRise(rh);
  $("rh").style.color = cssVar(LEVEL_VAR[level]);
  $("mood").textContent = LEVEL_TH[level] + (level ? ": " + why : "");
  $("mood").style.color = cssVar(LEVEL_VAR[level]);
  $("wet").textContent = wet === null ? "-" : mmss(wet);
  $("rhmax").textContent = rmax === null ? "-" : rmax.toFixed(1) + " %";
  $("temp").textContent = t === null ? "-" : t.toFixed(1) + " C" + (sun ? " (แดด +" + sun + ")" : "");
  rows.push([new Date().toISOString(), topic, level, str(d.alert, 60), t, rh, sun, wet, rmax]);
  if (rows.length > MAX_ROWS) rows = rows.slice(-MAX_ROWS);
  saveHistory();
  renderStats();
  notifyRisk(level, why);
  if (level < 2) lastAlertKey = "";
  alarm(level === 2, why);
}

function onMessage(topic, payload) {
  let d = null;
  try { d = JSON.parse(payload.toString()); } catch (e) { return; }
  if (!d || typeof d !== "object" || Array.isArray(d)) return;
  lastMessageAt = Date.now();
  if (client && client.connected) setConn("ต่อแล้ว " + team + " · รับข้อมูลล่าสุด", true);
  if (topic.endsWith("/status") && d.type === "config") {
    if (num(d.rh_hi) !== null) {
      rhHi = d.rh_hi;
      $("rhhi").value = rhHi;
      window._rhThresholdCustomized = true;
      drawHi();
      renderStats();
    }
    if (num(d.t_hi) !== null) $("thi").value = d.t_hi;
    $("cmdMsg").textContent = "บอร์ดยืนยันเกณฑ์แล้ว: RH " + d.rh_hi + "%, อุณหภูมิ " + d.t_hi + "°C";
    return;
  }
  if (topic.endsWith("/cmd")) { if (d.cmd === "ack") alarm(false); return; }
  if (topic.endsWith("/event") || topic.endsWith("/telemetry")) onEvent(topic, d);
}

function connect() {
  if (typeof mqtt === "undefined") {
    setConn("โหลด MQTT ไม่สำเร็จ: ตรวจอินเทอร์เน็ต/รีเฟรชหน้า", false);
    $("cmdMsg").textContent = "ไลบรารี MQTT จาก CDN ยังไม่โหลด จึงยังเชื่อมต่อบอร์ดไม่ได้";
    return;
  }
  team = $("team").value.trim();
  if (!okTeam(team)) { setConn("ใส่ทีม เช่น team19", false); return; }
  history.replaceState(null, "", "?team=" + encodeURIComponent(team));
  if (client) client.end(true);
  client = mqtt.connect(BROKER_URL, { clientId: "idea3-web-" + Math.random().toString(16).slice(2, 10), connectTimeout: 20000 });
  setConn("กำลังต่อ...", false);
  loadHistory(team);
  client.on("connect", () => {
    lastMessageAt = Date.now();
    setConn("ต่อแล้ว " + team, true);
    const credit = document.querySelector(".footer-credit");
    if (credit) credit.textContent = "Developed by " + team + " | AIoT Smart Farm Project 2026";
    client.subscribe(ROOT + "/" + team + "/#", (err) => {
      if (err) setConn("เชื่อมต่อแล้ว แต่สมัครรับข้อมูลไม่สำเร็จ", false);
    });
  });
  client.on("close", () => setConn("หลุด กำลังต่อใหม่...", false));
  client.on("message", onMessage);
  client.on("error", (err) => {
    setConn("MQTT error: " + (err && err.message ? err.message : "เชื่อมต่อผิดพลาด"), false);
  });
}

function toBoard(obj) {
  if (!client || !client.connected) { $("cmdMsg").textContent = "ยังไม่ได้ต่อ broker"; return; }
  client.publish(ROOT + "/" + team + "/cmd", JSON.stringify(obj));
  $("cmdMsg").textContent = "ส่งแล้ว: " + JSON.stringify(obj);
}

function csvCell(v) {
  if (v === null || v === undefined) return "";
  const s = String(v);
  return /[",\r\n]/.test(s) ? '"' + s.replace(/"/g, '""') + '"' : s;
}
function downloadCsv() {
  if (!rows.length) { $("cmdMsg").textContent = "ยังไม่มีข้อมูลให้ดาวน์โหลด"; return; }
  const head = ["time", "topic", "level", "alert", "temp_c", "rh", "sun_c", "wet_s", "rh_max"];
  const text = [head.join(","), ...rows.map((r) => r.map(csvCell).join(","))].join("\r\n");
  const url = URL.createObjectURL(new Blob(["\uFEFF" + text], { type: "text/csv;charset=utf-8" }));
  const a = document.createElement("a");
  a.href = url; a.download = "silo_" + team + ".csv";
  document.body.appendChild(a); a.click(); a.remove();
  URL.revokeObjectURL(url);
}

$("calcReadings").addEventListener("click", () => {
  const ids = ["rhRead1", "rhRead2", "rhRead3"];
  const vals = ids.map(id => $(id).value.trim() === "" ? NaN : Number($(id).value));
  if (vals.some(v => !Number.isFinite(v) || v < 0 || v > 100)) {
    $("readingResult").textContent = "กรุณากรอกค่าความชื้นที่วัดจริงให้ครบทั้ง 3 ครั้ง (0-100% RH)";
    return;
  }
  const avg = vals.reduce((a, b) => a + b, 0) / vals.length;
  $("readingResult").textContent = "ค่าที่บันทึก: " + vals.map(v => v.toFixed(1) + "%").join(", ") + " | ค่าเฉลี่ย " + avg.toFixed(1) + "% RH. ใช้ค่าเฉลี่ยนี้ประกอบการอธิบายเท่านั้น ไม่ได้เปลี่ยนเกณฑ์ 70% อัตโนมัติ";
});

$("go").addEventListener("click", connect);
$("ack").addEventListener("click", () => toBoard({ cmd: "ack" }));
$("set").addEventListener("click", () => {
  const v = Number($("thi").value);
  if (!Number.isInteger(v) || v < 16 || v > 45) { $("cmdMsg").textContent = "t_hi ต้องเป็นเลขเต็ม 16-45"; return; }
  toBoard({ cmd: "set", t_hi: v });
});
$("setRh").addEventListener("click", () => {
  const v = Number($("rhhi").value);
  if (!Number.isInteger(v) || v < 40 || v > 90) {
    $("cmdMsg").textContent = "rh_hi ต้องเป็นเลขเต็ม 40-90";
    return;
  }
  // ใช้ได้เมื่ออัปเดต firmware ให้รับ cmd=set และฟิลด์ rh_hi แล้วเท่านั้น
  if (!client || !client.connected) { $("cmdMsg").textContent = "ยังไม่ได้ต่อ broker"; return; }
  toBoard({ cmd: "set", rh_hi: v });
  $("cmdMsg").textContent = "ส่งคำขอตั้ง RH " + v + "% แล้ว กำลังรอการยืนยันจากบอร์ด";
});
$("csv").addEventListener("click", downloadCsv);
$("notify").addEventListener("click", async () => {
  if (!("Notification" in window)) {
    $("cmdMsg").textContent = "เบราว์เซอร์นี้ไม่รองรับการแจ้งเตือน";
    return;
  }
  const permission = await Notification.requestPermission();
  notificationEnabled = permission === "granted";
  $("cmdMsg").textContent = notificationEnabled
    ? "เปิดการแจ้งเตือนแล้ว (ต้องอนุญาตจากเบราว์เซอร์ด้วย)"
    : "ยังไม่ได้รับอนุญาตให้แจ้งเตือน";
});
$("clearHistory").addEventListener("click", () => {
  if (!team) { $("cmdMsg").textContent = "กรุณาเลือกทีมก่อน"; return; }
  if (!confirm("ต้องการล้างประวัติของ " + team + " ในเบราว์เซอร์เครื่องนี้หรือไม่?")) return;
  rows = [];
  history_rh = [];
  try { localStorage.removeItem(storageKey(team)); } catch (_) {}
  drawTrend();
  renderStats();
  $("cmdMsg").textContent = "ล้างประวัติในเครื่องแล้ว";
  $("historyMsg").textContent = "ประวัติจะถูกเก็บไว้ในเบราว์เซอร์เครื่องนี้ สูงสุด 500 รายการต่อทีม";
});

drawHi();
// บอร์ดรุ่นที่ต่อยอดจะส่ง telemetry ทุก 5 วินาที; ถ้าเงียบเกิน 15 วินาทีให้เตือน
setInterval(() => {
  if (client && client.connected && lastMessageAt && Date.now() - lastMessageAt > 15000) {
    setConn("บอร์ดเงียบเกิน 15 วินาที: ตรวจ Wi-Fi/ไฟเลี้ยง", false);
  }
}, 3000);
team = new URLSearchParams(location.search).get("team") || "team19";
$("team").value = team;
if (okTeam(team)) connect(); else setConn("ใส่ทีมแล้วกด ต่อ", false);
</script>
</body>
</html>
