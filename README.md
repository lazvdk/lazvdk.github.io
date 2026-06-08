<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0">
<title>Reminders</title>
<link href="https://fonts.googleapis.com/css2?family=DM+Sans:wght@300;400;500;600&family=DM+Mono:wght@400;500&display=swap" rel="stylesheet">
<style>
  *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

  :root {
    --green: #4ade80;
    --green-dark: #16a34a;
    --green-mid: #22c55e;
    --white: #ffffff;
    --text: #1a1a1a;
    --muted: #6b7280;
    --card-bg: #ffffff;
  }

  html, body {
    min-height: 100vh;
    background: #1a1a1a;
    font-family: 'DM Sans', sans-serif;
    color: var(--text);
    display: flex;
    align-items: center;
    justify-content: center;
  }

  /* ── PHONE SHELL (shared) ── */
  .phone-shell {
    width: 320px;
    height: 640px;
    border-radius: 40px;
    border: 6px solid #333;
    overflow: hidden;
    display: flex;
    flex-direction: column;
    box-shadow: 0 24px 70px rgba(0,0,0,0.6);
    position: relative;
    flex-shrink: 0;
  }

  .phone-notch {
    width: 90px; height: 22px; background: #222;
    border-radius: 0 0 14px 14px; margin: 0 auto;
    flex-shrink: 0; position: relative; z-index: 2;
  }

  .home-bar {
    width: 100px; height: 4px;
    background: rgba(0,0,0,0.15); border-radius: 2px;
    margin: 10px auto 6px; flex-shrink: 0;
  }
  .home-bar.light { background: rgba(255,255,255,0.35); }

  /* ── ALL SCREENS live inside .phone-shell ── */
  .p-screen {
    flex: 1; display: none; flex-direction: column;
    align-items: center; justify-content: center;
    padding: 16px 18px 10px; overflow: hidden;
  }
  .p-screen.active { display: flex; }

  /* ── HOME LIST SCREEN ── */
  #ps-home {
    background: #f0fdf4;
    justify-content: flex-start;
    align-items: stretch;
    padding: 0;
  }

  .status-bar {
    display: flex; justify-content: space-between; align-items: center;
    padding: 10px 18px 4px;
    font-size: 12px; font-weight: 500; color: var(--text);
    font-family: 'DM Mono', monospace;
    flex-shrink: 0;
  }

  .list-header {
    padding: 6px 18px 12px;
    display: flex; justify-content: space-between; align-items: flex-end;
    flex-shrink: 0;
  }
  .list-header h1 { font-size: 26px; font-weight: 600; letter-spacing: -0.5px; }
  .header-add {
    width: 30px; height: 30px; background: var(--green-dark); color: white;
    border: none; border-radius: 50%; font-size: 20px; cursor: pointer;
    display: flex; align-items: center; justify-content: center; line-height: 1;
  }

  .section-label {
    padding: 0 18px 8px;
    font-size: 11px; font-weight: 600; letter-spacing: 0.08em;
    text-transform: uppercase; color: var(--muted);
    flex-shrink: 0;
  }

  .reminders-list {
    padding: 0 12px;
    display: flex; flex-direction: column; gap: 8px;
    overflow-y: auto; flex: 1;
    padding-bottom: 10px;
  }

  .reminder-card {
    background: var(--card-bg);
    border-radius: 16px;
    padding: 12px 14px;
    box-shadow: 0 1px 4px rgba(0,0,0,0.07);
    display: flex; align-items: center; gap: 10px;
  }

  .reminder-icon {
    width: 38px; height: 38px; border-radius: 10px;
    display: flex; align-items: center; justify-content: center;
    font-size: 18px; flex-shrink: 0;
  }
  .icon-kitchen { background: #dcfce7; }
  .icon-meds    { background: #fef9c3; }
  .icon-water   { background: #dbeafe; }

  .reminder-info { flex: 1; min-width: 0; }
  .reminder-name { font-size: 13px; font-weight: 500; margin-bottom: 1px; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }
  .reminder-sub  { font-size: 11px; color: var(--muted); }

  .reminder-controls { display: flex; align-items: center; gap: 6px; flex-shrink: 0; }

  .test-btn {
    background: var(--green); color: var(--green-dark);
    border: none; border-radius: 20px;
    padding: 4px 9px; font-size: 11px; font-weight: 600;
    cursor: pointer; white-space: nowrap;
  }
  .test-btn:active { background: #86efac; transform: scale(0.94); }

  .toggle { position: relative; width: 40px; height: 24px; flex-shrink: 0; }
  .toggle input { opacity: 0; width: 0; height: 0; }
  .toggle-slider {
    position: absolute; inset: 0;
    background: #d1d5db; border-radius: 12px; cursor: pointer;
    transition: background 0.2s;
  }
  .toggle-slider::before {
    content: ''; position: absolute;
    width: 18px; height: 18px; border-radius: 50%;
    background: white; top: 3px; left: 3px;
    transition: transform 0.2s; box-shadow: 0 1px 3px rgba(0,0,0,0.2);
  }
  .toggle input:checked + .toggle-slider { background: var(--green-mid); }
  .toggle input:checked + .toggle-slider::before { transform: translateX(16px); }

  /* ── TASK FLOW SCREENS ── */
  .bg-green { background: #4ade80; }
  .bg-red   { background: #fca5a5; }

  .p-btn {
    background: white; border: none; border-radius: 24px;
    padding: 13px 40px; font-size: 16px; font-weight: 600;
    cursor: pointer; margin-top: 18px; width: 100%;
    color: var(--green-dark);
    transition: transform 0.1s, opacity 0.1s;
  }
  .p-btn:active { transform: scale(0.97); opacity: 0.9; }
  .p-btn.secondary { background: rgba(255,255,255,0.25); color: white; }

  .task-box {
    background: rgba(255,255,255,0.18); border-radius: 16px;
    padding: 18px 16px; width: 100%; margin-bottom: 8px; text-align: center;
  }
  .task-title { color: white; font-size: 20px; font-weight: 500; margin-bottom: 14px; }

  .mic-ring {
    width: 100px; height: 100px; border-radius: 50%;
    background: rgba(255,255,255,0.25); border: 3px solid rgba(255,255,255,0.5);
    display: flex; align-items: center; justify-content: center; margin: 0 auto 18px;
  }
  .mic-ring svg { width: 46px; height: 46px; fill: white; }

  .p-h2 { color: white; font-size: 22px; font-weight: 600; text-align: center; margin-bottom: 8px; }
  .p-p  { color: rgba(255,255,255,0.85); font-size: 14px; text-align: center; }
</style>
</head>
<body>
<div class="phone-shell">
  <div class="phone-notch"></div>

  <!-- HOME: reminder list -->
  <div class="p-screen active" id="ps-home" style="justify-content:flex-start; align-items:stretch; padding:0;">
    <div class="status-bar">
      <span id="clock">9:41</span>
      <span style="letter-spacing:2px;">●●●</span>
    </div>
    <div class="list-header">
      <h1>Reminders</h1>
      <button class="header-add" aria-label="Add">+</button>
    </div>
    <div class="section-label">Active</div>
    <div class="reminders-list">

      <div class="reminder-card">
        <div class="reminder-icon icon-kitchen">🍳</div>
        <div class="reminder-info">
          <div class="reminder-name">Go to the kitchen</div>
          <div class="reminder-sub">Daily · Morning</div>
        </div>
        <div class="reminder-controls">
          <button class="test-btn" onclick="startFlow()">Test</button>
          <label class="toggle"><input type="checkbox" checked><span class="toggle-slider"></span></label>
        </div>
      </div>

      <div class="reminder-card">
        <div class="reminder-icon icon-meds">💊</div>
        <div class="reminder-info">
          <div class="reminder-name">Take medication</div>
          <div class="reminder-sub">Daily · 8:00 AM</div>
        </div>
        <div class="reminder-controls">
          <button class="test-btn" onclick="startFlow()">Test</button>
          <label class="toggle"><input type="checkbox" checked><span class="toggle-slider"></span></label>
        </div>
      </div>

      <div class="reminder-card">
        <div class="reminder-icon icon-water">💧</div>
        <div class="reminder-info">
          <div class="reminder-name">Drink water</div>
          <div class="reminder-sub">Every 2 hours</div>
        </div>
        <div class="reminder-controls">
          <button class="test-btn" onclick="startFlow()">Test</button>
          <label class="toggle"><input type="checkbox"><span class="toggle-slider"></span></label>
        </div>
      </div>

    </div>
    <div class="home-bar"></div>
  </div>

  <!-- TASK: go / help -->
  <div class="p-screen bg-green" id="ps-task">
    <div class="task-box">
      <div class="task-title">Go to kitchen</div>
      <button class="p-btn" onclick="showP('ps-speak-go')">Go</button>
      <button class="p-btn secondary" onclick="showP('ps-pills')">Help</button>
    </div>
    <div class="home-bar light"></div>
  </div>

  <!-- PILLS reminder -->
  <div class="p-screen bg-red" id="ps-pills">
    <div style="font-size:54px; margin-bottom:12px;">💊</div>
    <div class="p-h2" style="color:#7f1d1d; font-size:26px;">Take your pills!</div>
    <button class="p-btn" style="color:#991b1b;" onclick="showP('ps-task')">OK</button>
    <div class="home-bar"></div>
  </div>

  <!-- SPEAK after Go -->
  <div class="p-screen bg-green" id="ps-speak-go">
    <div class="mic-ring"><svg viewBox="0 0 24 24"><path d="M12 1a4 4 0 0 1 4 4v6a4 4 0 0 1-8 0V5a4 4 0 0 1 4-4z"/><path d="M19 10a7 7 0 0 1-14 0"/><line x1="12" y1="19" x2="12" y2="23" stroke="white" stroke-width="2"/><line x1="8" y1="23" x2="16" y2="23" stroke="white" stroke-width="2"/></svg></div>
    <div class="p-h2">Speak</div>
    <div class="p-p">Tell us why you're going to the kitchen!</div>
    <button class="p-btn" onclick="showP('ps-goodjob')">Done</button>
    <div class="home-bar light"></div>
  </div>

  <!-- SPEAK after Help -->
  <div class="p-screen bg-green" id="ps-speak-help">
    <div class="mic-ring"><svg viewBox="0 0 24 24"><path d="M12 1a4 4 0 0 1 4 4v6a4 4 0 0 1-8 0V5a4 4 0 0 1 4-4z"/><path d="M19 10a7 7 0 0 1-14 0"/><line x1="12" y1="19" x2="12" y2="23" stroke="white" stroke-width="2"/><line x1="8" y1="23" x2="16" y2="23" stroke="white" stroke-width="2"/></svg></div>
    <div class="p-h2">Speak</div>
    <div class="p-p">Why?</div>
    <button class="p-btn" onclick="showP('ps-task')">OK</button>
    <div class="home-bar light"></div>
  </div>

  <!-- GOOD JOB -->
  <div class="p-screen" id="ps-goodjob" style="background:#86efac;">
    <div style="font-size:54px; margin-bottom:10px;">🎉</div>
    <div class="p-h2">Congratulations!</div>
    <div style="color:rgba(255,255,255,0.9); font-size:18px; font-weight:500; margin-top:6px;">+20 tokens</div>
    <button class="p-btn" onclick="showP('ps-home')" style="margin-top:28px;">Back to reminders</button>
    <div class="home-bar light"></div>
  </div>

</div>

<script>
  function tick() {
    const n = new Date();
    document.getElementById('clock').textContent =
      n.getHours().toString().padStart(2,'0') + ':' + n.getMinutes().toString().padStart(2,'0');
  }
  tick(); setInterval(tick, 10000);

  function showP(id) {
    document.querySelectorAll('.p-screen').forEach(s => s.classList.remove('active'));
    document.getElementById(id).classList.add('active');
  }

  function startFlow() {
    showP('ps-task');
  }
</script>
</body>
</html>
