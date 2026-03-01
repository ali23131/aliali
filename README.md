# aliali
a-shell
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Terminal — Saud</title>
<meta name="color-scheme" content="dark">
<link rel="preconnect" href="https://fonts.googleapis.com">
<link href="https://fonts.googleapis.com/css2?family=Fira+Code:wght@400;500;600&display=swap" rel="stylesheet">
<style>
  :root{
    --bg: #0b0f14;
    --panel: #0f141a;
    --text: #d6dee7;
    --muted: #7b8a99;
    --green: #7ee787;
    --cyan: #6fd3ff;
    --yellow:#ffd166;
    --red:#ff6b6b;
    --accent:#9aa6ff;
    --link:#9cdcfe;
  }
  *{box-sizing:border-box}
  html,body{height:100%}
  body{
    margin:0;
    background: radial-gradient(1200px 600px at 70% 10%, rgba(70,90,120,.15), transparent 60%),
               linear-gradient(160deg, #0b0f14 0%, #0a0d12 100%);
    color:var(--text);
    font-family: "Fira Code", ui-monospace, SFMono-Regular, Menlo, Consolas, monospace;
    line-height:1.5;
  }
  .wrap{
    max-width:980px;
    margin:32px auto;
    padding:0 16px;
  }
  .window{
    border-radius:14px;
    background:linear-gradient(180deg, #111820, #0d131a);
    box-shadow: 0 12px 30px rgba(0,0,0,.45), inset 0 0 0 1px rgba(255,255,255,.04);
    overflow:hidden;
    border:1px solid rgba(255,255,255,.06);
  }
  .titlebar{
    display:flex; align-items:center; gap:8px;
    padding:10px 12px;
    background:linear-gradient(180deg, rgba(255,255,255,.04), transparent);
    border-bottom:1px solid rgba(255,255,255,.06);
  }
  .dot{width:12px;height:12px;border-radius:50%}
  .dot.close{background:#ff5f56}
  .dot.min{background:#ffbd2e}
  .dot.max{background:#27c93f}
  .title{
    margin-left:8px; color:var(--muted); font-size:13px;
    user-select:none;
  }
  .screen{
    padding:18px;
    min-height:60vh;
    background:
      radial-gradient(1000px 400px at 30% 0%, rgba(80,100,140,.08), transparent 60%),
      repeating-linear-gradient(180deg, rgba(255,255,255,.02) 0 2px, transparent 2px 4px);
  }
  .line{white-space:pre-wrap; word-break:break-word}
  .prompt{color:var(--green)}
  .path{color:var(--cyan)}
  .muted{color:var(--muted)}
  .output a{color:var(--link); text-decoration:none}
  .output a:hover{ text-decoration:underline; }
  .inputline{
    display:flex; gap:10px; align-items:center; flex-wrap:wrap;
  }
  .cmd{
    flex:1 1 260px;
    background:transparent; border:none; outline:none;
    color:var(--text); font:inherit; padding:0; caret-color:var(--accent);
  }
  .cursor{
    display:inline-block; width:8px; height:1.2em; margin-left:2px;
    background:var(--accent); vertical-align:bottom;
    animation:blink 1.1s steps(1,end) infinite;
  }
  @keyframes blink { 50%{opacity:0} }
  .hint{ color: var(--yellow); }
  .tag{ display:inline-block; padding:.1rem .45rem; border:1px solid rgba(255,255,255,.15); border-radius:999px; color:#c8d3e3; }
  .section{margin:8px 0 14px}
  .hr{height:1px; background:rgba(255,255,255,.08); margin:12px 0}
  .footer{ color:var(--muted); font-size:12px; padding:10px 18px; border-top:1px solid rgba(255,255,255,.06) }
  /* Mobile tweaks */
  @media (max-width:640px){
    .screen{ padding:14px }
    .title{ display:none }
  }
</style>
</head>
<body>
  <div class="wrap">
    <div class="window">
      <div class="titlebar">
        <span class="dot close"></span><span class="dot min"></span><span class="dot max"></span>
        <span class="title">a‑Shell style — Saud@web</span>
      </div>
      <div id="screen" class="screen" aria-live="polite" aria-label="Terminal output">
        <div class="line muted">Welcome to <span class="tag">a‑Shell‑style</span> web terminal.</div>
        <div class="line muted">Type <span class="hint">help</span> to see commands. Use <span class="hint">Tab</span> to autocomplete, <span class="hint">↑/↓</span> for history.</div>
        <div class="hr"></div>
        <!-- Output will append here -->
        <div id="history" class="output" role="log"></div>

        <!-- Active input line -->
        <div class="inputline">
          <span class="prompt">saud@web</span>:<span class="path">~</span>$&nbsp;
          <span id="typed"></span><span class="cursor" aria-hidden="true"></span>
          <input id="cmd" class="cmd" type="text" aria-label="Command input" autocomplete="off" autocapitalize="off" spellcheck="false" autofocus />
        </div>
      </div>
      <div class="footer">
        Built with ❤️ by Saud — Theme inspired by a‑Shell. Press <strong>?</strong> for quick tips.
      </div>
    </div>
  </div>

<script>
(() => {
  const historyEl = document.getElementById('history');
  const input = document.getElementById('cmd');
  const typedMirror = document.getElementById('typed');
  const screen = document.getElementById('screen');

  const state = {
    cwd: '~',
    history: [],
    histIndex: -1,
    files: {
      '~': ['about.txt', 'projects/', 'contact/'],
      'projects/': ['mt5-btc-bot.md', 'arduino-labs.md', 'business-ideas.md'],
      'contact/': ['email.txt', 'telegram.link']
    }
  };

  const commands = {
    help() {
      return section('Commands',
`help               Show this help
about              About me
projects           Show featured projects
contact            How to reach me
ls [path]          List items (e.g., ls, ls projects/)
open <item>        Open a file/link (e.g., open mt5-btc-bot.md)
clear              Clear the screen
theme <name>       Switch theme: default | hacker | solar
echo <text>        Print text
date               Show date/time
whoami             Show user info`);
    },
    about() {
      return section('About',
`Name: Saud
Focus: University student, math/stat lover, building trading bots (MT5/MT5 BTCUSD), Arduino projects, and English improvement.
Goal: Ship practical tools with clean, simple UX — and keep learning every day.`);
    },
    projects() {
      return section('Projects',
`• MT5 BTCUSD Expert Advisor
  - Trend + swing logic, spread-aware, risk rules
  - Status: Iterating strategies and presets

• Arduino Lab Series
  - Traffic light controller, LED brightness sensor
  - Tinkercad + real‑world wiring tips

• Business Research Ideas
  - Customer satisfaction models, AI productivity`);
    },
    contact() {
      return section('Contact',
`Email: <a href="mailto:saud@example.com">saud@example.com</a>
Telegram: <a target="_blank" href="https://t.me/your_username">t.me/your_username</a>
GitHub: <a target="_blank" href="https://github.com/yourname">github.com/yourname</a>`);
    },
    ls(arg) {
      const path = normalize(arg || state.cwd);
      const list = state.files[path];
      if (!list) return error(`ls: cannot access '${arg||path}': No such file or directory`);
      return list.map(x => formatItem(x)).join('\n');
    },
    open(arg) {
      if (!arg) return error('open: missing <item>');
      const path = normalize(state.cwd);
      const list = state.files[path] || [];
      // allow "open x" in cwd or "open projects/mt5-btc-bot.md"
      const [dir, file] = splitPath(arg);
      let base = path;
      if (dir) base = normalize(dir.endsWith('/') ? dir : dir + '/');
      const items = state.files[base];
      if (!items) return error(`open: cannot open '${arg}'`);
      const target = items.find(x => stripSlash(x) === file);
      if (!target) return error(`open: '${arg}' not found`);

      // Behaviors
      if (target.endsWith('.md')) {
        if (target.includes('mt5-btc-bot')) {
          return section('mt5-btc-bot.md',
`# MT5 BTCUSD EA
- Mode: Trend/Swing with safe hedging rules
- Notes: Spread ~1200 pts, commission awareness, VPS ready
- Todo: Backtests with Preset 0/1, risk by balance auto 1:1000`);
        }
        if (target.includes('arduino')) {
          return section('arduino-labs.md',
`# Arduino Labs
- Traffic light FSM
- PWM brightness via LDR
- Tinkercad + real wiring`);
        }
        if (target.includes('business')) {
          return section('business-ideas.md',
`# Business Research
- AI & productivity
- Time management models
- Customer satisfaction indexes`);
        }
      }
      if (target.endsWith('.txt')) {
        if (target.includes('about')) return commands.about();
        if (target.includes('email')) return 'Email: saud@example.com';
      }
      if (target.endsWith('.link')) {
        window.open('https://t.me/your_username', '_blank');
        return info('Opening Telegram…');
      }
      if (target.endsWith('/')) {
        state.cwd = base + stripSlash(target) + '/';
        return info(`cd ${state.cwd}`);
      }
      return info(`Opened: ${target}`);
    },
    clear() {
      historyEl.innerHTML = '';
      return '';
    },
    theme(arg) {
      const t = (arg||'default').toLowerCase();
      if (t === 'hacker') {
        setTheme({bg:'#020b05', panel:'#020f08', text:'#c3ffcf', accent:'#00ff88', link:'#7fffd4'});
        return info('Theme: hacker');
      } else if (t === 'solar') {
        setTheme({bg:'#002b36', panel:'#03303b', text:'#fdf6e3', accent:'#b58900', link:'#2aa198'});
        return info('Theme: solarized');
      } else {
        setTheme(); return info('Theme: default');
      }
    },
    echo(arg) { return arg || ''; },
    date() { return new Date().toString(); },
    whoami() {
      return `user: saud
shell: web-terminal
path: ${state.cwd}`;
    }
  };

  // ---------- helpers ----------
  function section(title, body){
    return `\n\u001b[38;2;153;170;255m## ${title}\u001b[0m\n${body}`;
  }
  function error(msg){ return `\u001b[31m${msg}\u001b[0m`; }
  function info(msg){ return `\u001b[36m${msg}\u001b[0m`; }
  function normalize(p){
    if (!p || p==='~') return '~';
    if (p.startsWith('~/')) return p.replace('~/', '');
    return p;
  }
  function splitPath(p){
    const parts = p.split('/');
    const file = parts.pop();
    const dir = parts.length ? parts.join('/')+'/' : '';
    return [dir, file];
  }
  function stripSlash(x){ return x.endsWith('/') ? x.slice(0,-1) : x; }
  function formatItem(x){
    const isDir = x.endsWith('/');
    return (isDir ? '\u001b[33m' : '\u001b[32m') + x + '\u001b[0m';
  }
  function appendOutput(text){
    if (!text) return;
    const pre = document.createElement('pre');
    pre.className = 'line';
    pre.innerHTML = ansiToHtml(escapeHtml(text));
    historyEl.appendChild(pre);
    scrollToBottom();
  }
  function appendEcho(cmdText){
    const div = document.createElement('div');
    div.className = 'line';
    div.innerHTML = `<span class="prompt">saud@web</span>:<span class="path">${state.cwd}</span>$ ${escapeHtml(cmdText)}`;
    historyEl.appendChild(div);
    scrollToBottom();
  }
  function scrollToBottom(){
    setTimeout(() => screen.scrollTop = screen.scrollHeight, 0);
  }
  function escapeHtml(s){
    return s.replace(/[&<>"']/g, m => ({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;'}[m]));
  }
  // simple ANSI → HTML color
  function ansiToHtml(s){
    return s
      .replace(/\u001b\[0m/g,'</span>')
      .replace(/\u001b\[31m/g,'<span style="color: var(--red)">')
      .replace(/\u001b\[32m/g,'<span style="color: var(--green)">')
      .replace(/\u001b\[33m/g,'<span style="color: var(--yellow)">')
      .replace(/\u001b\[36m/g,'<span style="color: var(--cyan)">')
      .replace(/\u001b\[38;2;153;170;255m/g,'<span style="color: var(--accent)">');
  }
  function setTheme(preset){
    const root = document.documentElement;
    if (!preset){ root.style.cssText=''; return; }
    const map = {
      '--bg': preset.bg, '--panel': preset.panel, '--text': preset.text,
      '--accent': preset.accent, '--link': preset.link
    };
    for (const k in map) root.style.setProperty(k, map[k]);
  }

  // ---------- input/UX ----------
  function run(raw){
    const line = raw.trim();
    if (!line) return;
    // store history (dedupe consecutive)
    if (state.history[state.history.length-1] !== line) state.history.push(line);
    state.histIndex = state.history.length;

    appendEcho(line);

    const [cmd, ...rest] = tokenize(line);
    const arg = rest.join(' ');
    const fn = commands[cmd];
    if (!fn) { appendOutput(error(`command not found: ${cmd}`)); return; }

    const res = fn(arg);
    if (res !== undefined) appendOutput(res);
  }

  function tokenize(s){
    // simple quoted arg parser
    const out=[]; let cur=''; let q=null;
    for (let i=0;i<s.length;i++){
      const c=s[i];
      if (q){
        if (c===q){ q=null; continue; }
        cur+=c;
      } else {
        if (c==='"' || c==="'"){ q=c; continue; }
        if (/\s/.test(c)){ if (cur){ out.push(cur); cur=''; } }
        else cur+=c;
      }
    }
    if (cur) out.push(cur);
    return out;
  }

  // mirror visible typed text (for custom cursor)
  input.addEventListener('input', () => {
    typedMirror.textContent = input.value;
  });

  input.addEventListener('keydown', (e) => {
    // tips
    if (e.key === '?' && !e.ctrlKey && !e.metaKey) {
      e.preventDefault();
      appendOutput(info('Tips: Tab = autocomplete, ↑/↓ = history, theme hacker|solar|default'));
      return;
    }
    // run command
    if (e.key === 'Enter') {
      e.preventDefault();
      const val = input.value;
      input.value = '';
      typedMirror.textContent = '';
      run(val);
      return;
    }
    // history
    if (e.key === 'ArrowUp') {
      e.preventDefault();
      if (state.histIndex > 0) state.histIndex--;
      input.value = state.history[state.histIndex] || '';
      typedMirror.textContent = input.value;
      input.setSelectionRange(input.value.length, input.value.length);
      return;
    }
    if (e.key === 'ArrowDown') {
      e.preventDefault();
      if (state.histIndex < state.history.length) state.histIndex++;
      input.value = state.history[state.histIndex] || '';
      typedMirror.textContent = input.value;
      input.setSelectionRange(input.value.length, input.value.length);
      return;
    }
    // autocomplete
    if (e.key === 'Tab') {
      e.preventDefault();
      const val = input.value.trim();
      if (!val) { input.value = 'help'; typedMirror.textContent = input.value; return; }
      const [cmd, ...rest] = val.split(/\s+/);
      // complete command names
      if (rest.length === 0) {
        const keys = Object.keys(commands);
        const options = keys.filter(k => k.startsWith(cmd));
        if (options.length === 1) { input.value = options[0]; }
        else if (options.length > 1) { appendOutput(options.join('  ')); }
      } else {
        // complete files in cwd
        const fragment = rest.join(' ');
        const list = (state.files[state.cwd]||[]).map(stripSlash);
        const matches = list.filter(x => x.startsWith(fragment));
        if (matches.length === 1) input.value = cmd + ' ' + matches[0];
        else if (matches.length > 1) appendOutput(matches.join('  '));
      }
      typedMirror.textContent = input.value;
    }
  });

  // focus on click anywhere in screen
  screen.addEventListener('click', () => input.focus());
  // auto-focus on load
  window.addEventListener('load', () => input.focus());
})();
</script>
</body>
</html>
