# app.js-and-package.json
// OY Bot - single file. Run: node app.js  (needs env TOKEN and METAAPI_TOKEN)
const ASSETS = {"/index.html":"<!doctype html>\n<html lang=\"en\">\n<head>\n<meta charset=\"utf-8\">\n<meta name=\"viewport\" content=\"width=device-width,initial-scale=1,viewport-fit=cover,user-scalable=no\">\n<meta name=\"apple-mobile-web-app-capable\" content=\"yes\">\n<meta name=\"apple-mobile-web-app-status-bar-style\" content=\"black-translucent\">\n<meta name=\"apple-mobile-web-app-title\" content=\"OY Bot\">\n<meta name=\"theme-color\" content=\"#0b0f14\">\n<link rel=\"manifest\" href=\"manifest.json\">\n<link rel=\"apple-touch-icon\" href=\"/icon-180.png\">\n<title>OY Bot</title>\n<style>\n:root{--bg:#0b0f14;--card:#141b24;--line:#223041;--tx:#e8eef5;--mut:#7d8ea1;--g:#22c55e;--r:#ef4444;--y:#f59e0b;--b:#3b82f6}\n*{box-sizing:border-box;-webkit-tap-highlight-color:transparent}\nbody{margin:0;background:var(--bg);color:var(--tx);font:16px -apple-system,system-ui,sans-serif;\n padding:env(safe-area-inset-top) 16px calc(env(safe-area-inset-bottom) + 16px)}\nh1{font-size:20px;margin:18px 0 4px}\n.sub{color:var(--mut);font-size:13px;margin-bottom:14px}\n.card{background:var(--card);border:1px solid var(--line);border-radius:16px;padding:14px;margin-bottom:12px}\n.pill{display:inline-flex;align-items:center;gap:8px;font-weight:600;font-size:14px}\n.dot{width:10px;height:10px;border-radius:50%;background:var(--mut)}\n.on .dot{background:var(--g);box-shadow:0 0 8px var(--g)}\n.off .dot{background:var(--r)}\n.warn .dot{background:var(--y)}\n.grid{display:grid;grid-template-columns:1fr 1fr;gap:10px}\n.k{color:var(--mut);font-size:12px}\n.v{font-size:20px;font-weight:700;margin-top:2px}\n.pos{color:var(--g)}.neg{color:var(--r)}\n.btns{display:grid;grid-template-columns:1fr 1fr;gap:10px}\nbutton{border:0;border-radius:14px;padding:16px;font-size:16px;font-weight:700;color:#fff;background:#27364a}\nbutton:active{opacity:.7}\n.start{background:#16a34a}.stop{background:#b91c1c}\n.wide{grid-column:1/3}\ninput,select{width:100%;padding:14px;border-radius:12px;border:1px solid var(--line);background:#0e141b;color:var(--tx);font-size:16px;margin:6px 0}\n.msg{color:var(--mut);font-size:13px;margin-top:6px;word-break:break-word}\n.hide{display:none}\n.tag{font-size:11px;font-weight:800;padding:3px 8px;border-radius:8px;letter-spacing:.5px}\n.demo{background:#1e3a8a;color:#bfdbfe}.real{background:#7f1d1d;color:#fecaca}\n.acc{display:flex;align-items:center;justify-content:space-between;gap:10px;padding:12px 0;border-bottom:1px solid var(--line)}\n.acc:last-child{border:0}\n.acc small{color:var(--mut);display:block;font-size:12px}\n.seg{display:grid;grid-template-columns:1fr 1fr;gap:8px;margin:6px 0}\n.seg button{padding:12px}\n.seg .sel.demo{background:#1d4ed8}.seg .sel.real{background:#b91c1c}\n.link{color:var(--mut);background:none;padding:6px;font-size:14px;font-weight:600}\n.err{color:var(--r)}\n</style>\n</head>\n<body>\n<div id=\"login\" class=\"hide\">\n  <h1>OY Bot</h1>\n  <div class=\"sub\">Enter your app token to connect.</div>\n  <div class=\"card\">\n    <input id=\"tok\" type=\"password\" placeholder=\"App token\" autocomplete=\"off\">\n    <button class=\"start\" style=\"width:100%\" onclick=\"saveTok()\">Connect</button>\n  </div>\n</div>\n\n<!-- DASHBOARD -->\n<div id=\"main\" class=\"hide\">\n  <h1>OY Bot</h1>\n  <div class=\"sub\" id=\"acct\">No account linked</div>\n\n  <div class=\"card\">\n    <div id=\"conn\" class=\"pill\"><span class=\"dot\"></span><span id=\"connTxt\">Checking…</span></div>\n    <div id=\"runTxt\" class=\"msg\"></div>\n  </div>\n\n  <div class=\"card btns\">\n    <button class=\"start\" onclick=\"startBot()\">▶ START</button>\n    <button class=\"stop\" onclick=\"cmd('stop')\">■ STOP</button>\n    <button class=\"wide\" onclick=\"closeAll()\">Close all &amp; stop</button>\n    <button class=\"wide\" onclick=\"show('accounts')\">⇄ Accounts (demo / real)</button>\n  </div>\n\n  <div class=\"card grid\">\n    <div><div class=\"k\">Balance</div><div class=\"v\" id=\"bal\">—</div></div>\n    <div><div class=\"k\">Equity</div><div class=\"v\" id=\"eq\">—</div></div>\n    <div><div class=\"k\">Open P/L</div><div class=\"v\" id=\"op\">—</div></div>\n    <div><div class=\"k\">Today P/L</div><div class=\"v\" id=\"tp\">—</div></div>\n    <div><div class=\"k\">Open trades</div><div class=\"v\" id=\"pos\">—</div></div>\n    <div><div class=\"k\">Trades today</div><div class=\"v\" id=\"tt\">—</div></div>\n    <div><div class=\"k\">Symbol</div><div class=\"v\" id=\"sym\">—</div></div>\n    <div><div class=\"k\">Spread (pts)</div><div class=\"v\" id=\"spr\">—</div></div>\n    <div><div class=\"k\">Peak equity</div><div class=\"v\" id=\"hwm\">—</div></div>\n    <div><div class=\"k\">Protection</div><div class=\"v\" id=\"prot\">—</div></div>\n  </div>\n\n  <div class=\"card\">\n    <div class=\"k\">Open trades</div>\n    <div id=\"trades\" class=\"msg\">None</div>\n    <div class=\"k\" style=\"margin-top:10px\">Pending chase orders</div>\n    <div id=\"pends\" class=\"msg\">None</div>\n  </div>\n\n  <div class=\"card\">\n    <div class=\"k\" style=\"margin-bottom:6px\">Chaser: size &amp; safety</div>\n    <div class=\"grid\">\n      <div><div class=\"k\">Risk per trade %</div><input id=\"sRisk\" inputmode=\"decimal\"></div>\n      <div><div class=\"k\">Total risk cap %</div><input id=\"sTotal\" inputmode=\"decimal\"></div>\n      <div><div class=\"k\">Add-ons (0 = off)</div><input id=\"sAdds\" inputmode=\"numeric\"></div>\n      <div><div class=\"k\">Max spread % of stop</div><input id=\"sSpread\" inputmode=\"decimal\"></div>\n      <div><div class=\"k\">Stop all if down % in a day</div><input id=\"sDD\" inputmode=\"decimal\"></div>\n      <div><div class=\"k\">Lock if equity falls to % of peak</div><input id=\"sProt\" inputmode=\"decimal\"></div>\n    </div>\n    <button style=\"width:100%;margin-top:6px\" onclick=\"saveSettings()\">Save settings</button>\n    <button class=\"wide\" style=\"width:100%;margin-top:8px\" onclick=\"resetProt()\">Reset protection lock</button>\n  </div>\n\n  <div class=\"card\">\n    <div class=\"k\">Symbol to trade (exactly as your account shows it, e.g. EURUSD or EURUSDc)</div>\n    <input id=\"symIn\" placeholder=\"EURUSD\" autocapitalize=\"characters\" autocomplete=\"off\">\n    <button style=\"width:100%\" onclick=\"setSym()\">Save symbol</button>\n  </div>\n\n  <div class=\"card\"><div class=\"k\">Bot status</div><div class=\"msg\" id=\"msg\">—</div></div>\n  <div class=\"sub\" style=\"text-align:center\"><a href=\"#\" onclick=\"logout();return false\" style=\"color:var(--mut)\">Change app token</a></div>\n</div>\n\n<!-- ACCOUNTS -->\n<div id=\"accounts\" class=\"hide\">\n  <h1>Accounts</h1>\n  <div class=\"sub\">Link your Exness MT5 demo or real account (hedging account). Tap one to trade on it.</div>\n  <div class=\"card\" id=\"accList\"><div class=\"msg\">Loading…</div></div>\n\n  <div class=\"card\">\n    <div class=\"k\" style=\"margin-bottom:6px\">Link a new account</div>\n    <div class=\"seg\">\n      <button id=\"kDemo\" class=\"sel demo\" onclick=\"setKind('demo')\">DEMO</button>\n      <button id=\"kReal\" onclick=\"setKind('real')\">REAL</button>\n    </div>\n    <input id=\"fLogin\" inputmode=\"numeric\" placeholder=\"MT5 login (number)\" autocomplete=\"off\">\n    <input id=\"fPass\" type=\"password\" placeholder=\"MT5 trading password\" autocomplete=\"new-password\">\n    <input id=\"fServer\" placeholder=\"Server, e.g. Exness-MT5Trial9\" autocapitalize=\"none\" autocomplete=\"off\">\n    <button class=\"start\" style=\"width:100%;margin-top:6px\" onclick=\"link()\">Link account</button>\n    <div class=\"msg\" id=\"linkMsg\"></div>\n    <div class=\"msg\">Find login and server in Exness app: Accounts &gt; your account &gt; ⋯ &gt; Account information. Use the MT5 trading password (not your Exness login password). Your password is sent over HTTPS straight to MetaApi and is not saved by this app.</div>\n  </div>\n  <button style=\"width:100%\" onclick=\"show('main')\">← Back</button>\n</div>\n\n<script>\nconst $ = id => document.getElementById(id);\nlet tok = localStorage.getItem('tok') || '';\nlet timer, view = 'main', kind = 'demo', last = null, accs = [];\n\nfunction show(v){\n  view = v;\n  for(const id of ['login','main','accounts']) $(id).classList.toggle('hide', id!==v);\n  if(v==='accounts') loadAccounts();\n}\nfunction saveTok(){ tok = $('tok').value.trim(); if(!tok) return; localStorage.setItem('tok', tok); init(); }\nfunction logout(){ localStorage.removeItem('tok'); tok=''; clearInterval(timer); init(); }\n\nasync function api(path, opt={}){\n  const r = await fetch(path, {...opt, headers:{'X-Token':tok,'Content-Type':'application/json'}});\n  if(r.status===401){ logout(); throw new Error('auth'); }\n  const j = await r.json().catch(()=>({}));\n  if(!r.ok) throw new Error(j.error||'Request failed');\n  return j;\n}\nconst post = (p,b) => api(p,{method:'POST',body:JSON.stringify(b||{})});\n\nasync function cmd(c, extra){\n  try{ await post('/api/cmd',{cmd:c,...extra}); if(navigator.vibrate)navigator.vibrate(30); $('runTxt').textContent='Sent: '+c; poll(); }\n  catch(e){ $('runTxt').innerHTML='<span class=\"err\">'+e.message+'</span>'; }\n}\nfunction startBot(){\n  const s = last && last.status;\n  if(s && s.kind==='real'){\n    if(!confirm('START trading with REAL money on account #'+s.login+'?')) return;\n    return cmd('start',{confirmReal:true});\n  }\n  cmd('start');\n}\nfunction resetProt(){ if(confirm('Clear the protection lock? The peak is reset to the current equity.')) cmd('reset'); }\nfunction closeAll(){ if(confirm('Close ALL bot positions and stop the bot?')) cmd('closeall'); }\nasync function setSym(){\n  const v = $('symIn').value.trim(); if(!v) return;\n  try{ await post('/api/symbol',{symbol:v}); $('runTxt').textContent='Symbol set: '+v; poll(); }\n  catch(e){ $('runTxt').innerHTML='<span class=\"err\">'+e.message+'</span>'; }\n}\n\n/* ---- accounts ---- */\nfunction setKind(k){\n  kind = k;\n  $('kDemo').className = k==='demo' ? 'sel demo' : '';\n  $('kReal').className = k==='real' ? 'sel real' : '';\n  $('fServer').placeholder = k==='demo' ? 'Server, e.g. Exness-MT5Trial9' : 'Server, e.g. Exness-MT5Real8';\n}\nasync function loadAccounts(){\n  try{\n    const d = await api('/api/accounts'); accs = d.accounts;\n    const act = last && last.status && last.status.activeId;\n    $('accList').innerHTML = accs.length ? accs.map(a=>`\n      <div class=\"acc\">\n        <div onclick=\"pick('${a.id}')\" style=\"flex:1\">\n          <span class=\"tag ${a.kind}\">${a.kind.toUpperCase()}</span> #${a.login}${a.id===act?' ✓':''}\n          <small>${a.server}</small>\n        </div>\n        <button class=\"link\" onclick=\"del('${a.id}')\">Remove</button>\n      </div>`).join('') : '<div class=\"msg\">No accounts linked yet.</div>';\n  }catch(e){ $('accList').innerHTML='<div class=\"msg err\">'+e.message+'</div>'; }\n}\nasync function pick(id){\n  try{ await post('/api/select',{id}); show('main'); $('runTxt').textContent='Switching account…'; poll(); }catch(e){ alert(e.message); }\n}\nasync function del(id){\n  if(!confirm('Remove this account from the bot? (Your Exness account itself is untouched.)')) return;\n  try{ await post('/api/remove',{id}); loadAccounts(); }catch(e){ alert(e.message); }\n}\nasync function link(){\n  const login=$('fLogin').value.trim(), password=$('fPass').value, server=$('fServer').value.trim();\n  if(!login||!password||!server){ $('linkMsg').innerHTML='<span class=\"err\">Fill in login, password and server.</span>'; return; }\n  if(kind==='real' && !confirm('Link a REAL-money account? The bot can open real trades when you press START.')) return;\n  try{\n    await post('/api/link',{login,password,server,kind});\n    $('fPass').value='';\n    $('linkMsg').textContent='Linking… this can take up to a minute.';\n    const t = setInterval(async()=>{\n      await poll(); const s=last&&last.status;\n      if(s && !s.linking){\n        clearInterval(t);\n        if(s.connected){ show('main'); } else { $('linkMsg').innerHTML='<span class=\"err\">'+s.msg+'</span>'; }\n      } else if(s){ $('linkMsg').textContent = s.msg; }\n    },2000);\n  }catch(e){ $('linkMsg').innerHTML='<span class=\"err\">'+e.message+'</span>'; }\n}\n\n/* ---- status ---- */\nconst money = n => (n==null?'—':Number(n).toLocaleString(undefined,{minimumFractionDigits:2,maximumFractionDigits:2}));\nfunction pl(el,n){ el.textContent=(n>0?'+':'')+money(n); el.className='v '+(n>0?'pos':n<0?'neg':''); }\n\nasync function poll(){\n  try{\n    const d = await api('/api/status');\n    last = d;\n    try{ localStorage.setItem('last', JSON.stringify(d)); }catch(e){}\n    render(d);\n  }catch(e){}\n}\nfunction render(d){\n  last = last || d;\n  const s = d.status; if(!s) return;\n  const c = $('conn');\n  if(s.linking){ c.className='pill warn'; $('connTxt').textContent='Linking account…'; }\n  else if(!s.activeId){ c.className='pill off'; $('connTxt').textContent='No account linked'; }\n  else if(!d.online){ c.className='pill off'; $('connTxt').textContent='Broker not connected'; }\n  else if(s.running){ c.className='pill on'; $('connTxt').textContent='Connected · Bot RUNNING'; }\n  else { c.className='pill warn'; $('connTxt').textContent='Connected · Bot STOPPED'; }\n  $('acct').innerHTML = s.activeId ? '<span class=\"tag '+(s.kind||'demo')+'\">'+(s.kind||'').toUpperCase()+'</span> '+s.broker+' · #'+s.login : 'No account linked. Tap Accounts.';\n  $('bal').textContent = money(s.balance); $('eq').textContent = money(s.equity);\n  pl($('op'), s.openProfit); pl($('tp'), s.todayPL);\n  $('pos').textContent = s.positions; $('tt').textContent = s.todayTrades;\n  $('sym').textContent = s.symbol; $('spr').textContent = s.spread;\n  $('msg').textContent = s.msg;\n  if(document.activeElement!==$('symIn') && !$('symIn').value) $('symIn').value = s.symbol;\n  $('tt').textContent = s.todayTrades+' ('+(s.wins||0)+'W '+(s.losses||0)+'L)';\n  $('hwm').textContent = money(s.hwm); $('prot').textContent = s.locked ? 'LOCKED' : 'OK'; $('prot').className = 'v '+(s.locked?'neg':'pos');\n  $('pends').innerHTML = (s.pendings&&s.pendings.length) ? s.pendings.map(o=>'<div class=\"acc\"><span class=\"tag '+(o.side.startsWith('BUY')?'demo':'real')+'\">'+o.side+'</span><b>'+o.price+'</b></div>').join('') : 'None';\n  $('trades').innerHTML = (s.open&&s.open.length) ? s.open.map(t=>'<div class=\"acc\"><span><span class=\"tag '+(t.side==='BUY'?'demo':'real')+'\">'+t.side+'</span> '+t.volume+' lot</span><b class=\"'+(t.profit>0?'pos':t.profit<0?'neg':'')+'\">'+(t.profit>0?'+':'')+money(t.profit)+'</b></div>').join('') : 'None';\n  if(s.settings && !settingsLoaded){\n    settingsLoaded = true;\n    $('sRisk').value=s.settings.riskPerTradePct; $('sTotal').value=s.settings.totalRiskPct; $('sAdds').value=s.settings.pyramid?s.settings.maxAdds:0;\n    $('sSpread').value=s.settings.maxSpreadPct; $('sDD').value=s.settings.maxDrawdownPct; $('sProt').value=s.settings.protectEquityPct;\n  }\n}\nlet settingsLoaded = false;\nasync function saveSettings(){\n  try{\n    const ad=+$('sAdds').value||0; await post('/api/settings',{riskPerTradePct:$('sRisk').value,totalRiskPct:$('sTotal').value,pyramid:ad>0,maxAdds:ad,maxSpreadPct:$('sSpread').value,maxDrawdownPct:$('sDD').value,protectEquityPct:$('sProt').value});\n    settingsLoaded=false; $('runTxt').textContent='Settings saved'; poll();\n  }catch(e){ $('runTxt').innerHTML='<span class=\"err\">'+e.message+'</span>'; }\n}\n\nfunction init(){\n  if(!tok){ show('login'); return; }\n  if(view==='login') view='main';\n  show(view);\n  try{ const c = JSON.parse(localStorage.getItem('last')||'null'); if(c){ last=c; render(c); $('connTxt').textContent='Updating…'; } }catch(e){}\n  poll(); clearInterval(timer); timer = setInterval(poll, 3000);\n}\nif(location.hash.length>1){ tok = decodeURIComponent(location.hash.slice(1)); localStorage.setItem('tok', tok); history.replaceState(null,'',location.pathname); }\ndocument.addEventListener('visibilitychange',()=>{ if(document.hidden){ clearInterval(timer); } else if(tok){ poll(); clearInterval(timer); timer=setInterval(poll,3000); } });\ninit();\nif('serviceWorker' in navigator) navigator.serviceWorker.register('sw.js');\n</script>\n</body>\n</html>\n","/manifest.json":"{\n  \"name\": \"OY Bot\",\n  \"short_name\": \"OY Bot\",\n  \"start_url\": \"/\",\n  \"display\": \"standalone\",\n  \"background_color\": \"#0b0f14\",\n  \"theme_color\": \"#0b0f14\",\n  \"icons\": [\n    {\n      \"src\": \"/icon-180.png\",\n      \"sizes\": \"180x180\",\n      \"type\": \"image/png\"\n    },\n    {\n      \"src\": \"/icon-512.png\",\n      \"sizes\": \"512x512\",\n      \"type\": \"image/png\",\n      \"purpose\": \"any\"\n    },\n    {\n      \"src\": \"/icon.svg\",\n      \"sizes\": \"any\",\n      \"type\": \"image/svg+xml\"\n    }\n  ]\n}","/sw.js":"// App shell is cached so the app opens instantly, even on a weak connection.\n// Live data (/api/*) is never cached.\nconst CACHE = 'oy-bot-v1';\nconst SHELL = ['/', '/index.html', '/manifest.json', '/icon.svg', '/icon-180.png'];\n\nself.addEventListener('install', e => {\n  e.waitUntil(caches.open(CACHE).then(c => c.addAll(SHELL)).then(() => self.skipWaiting()));\n});\nself.addEventListener('activate', e => {\n  e.waitUntil(caches.keys().then(ks => Promise.all(ks.filter(k => k !== CACHE).map(k => caches.delete(k)))).then(() => clients.claim()));\n});\nself.addEventListener('fetch', e => {\n  const url = new URL(e.request.url);\n  if (e.request.method !== 'GET' || url.pathname.startsWith('/api/')) return;\n  // cache-first for instant open, refresh in background\n  e.respondWith(caches.match(e.request, { ignoreSearch: true }).then(hit => {\n    const net = fetch(e.request).then(r => {\n      if (r.ok) caches.open(CACHE).then(c => c.put(e.request, r.clone()));\n      return r;\n    }).catch(() => hit);\n    return hit || net;\n  }));\n});\n","/icon.svg":"<svg xmlns=\"http://www.w3.org/2000/svg\" viewBox=\"0 0 180 180\"><rect width=\"180\" height=\"180\" rx=\"40\" fill=\"#0b0f14\"/><path d=\"M30 120 L65 80 L95 105 L150 45\" fill=\"none\" stroke=\"#22c55e\" stroke-width=\"12\" stroke-linecap=\"round\" stroke-linejoin=\"round\"/><circle cx=\"150\" cy=\"45\" r=\"9\" fill=\"#22c55e\"/></svg>\n","/icon-180.png":{"b64":"iVBORw0KGgoAAAANSUhEUgAAALQAAAC0CAIAAACyr5FlAAAPf0lEQVR4Aexda2wc1RWeO+u1s+vX2o4JAQJJiEJNqUqC8qBEoq3UVvRHaZtGVKoKAir1D1VftLFCChTCq0ARUlqpErRShfqHUqmVaEVBtFXTECdAStLG5ZmGVx6Ovev1Y+Nd7w5nZtbr8e7szL0z987cmT2r6/HMnXPPPee7n+9cnzMzq3b2LhdX0j0DS0pv7VBgp+LcCUtzumf5EhgtqAo1SVUEfLSFD+gm1o9SO4AzWGgRIGQROGL5QPsFpDXY5164kcOwUjG3FvsJd4tRYQ0BK86aBuCbhRtROJDDIIRuECEmwWvG405wCBjgA/56j7UR0Q98/PglB9gBvRPjAztYQkfAGAoCZphDAzuei19y1EzxbAE2FIEAl3HxQg6gJFzhRLiEOkUgYIyXft1nVe6FHAYrWTuKk3zEfDHGS7/QsNrNQA7PBGS1CeUFIcA6ggzk8ExAQa6iWlYEWEeQgRyspqB81BFAckR9BAXaj+QQCG7UVSM5oj6CAu13IgdE6wX2jKolQMB5iJ3IQRTCZj9KRw0B4jjETuSImqdoL2cEkBycAY2TOhtyQBwtTh6iLzQI2A66DTkgjkajDmXihIDtoNeTA9OtcRpyJl8aJ48l5ABmEMKkEIXjgwAhBAhg9WcJOQgyw4pNMPsS97JIjsZZRWKz0TQhCMDsYKXBIjkIwXlDCOLRUkrIIg2q5LDyJVrOoLXcEaiRoUoO7h2gwhgggOSIwSCKcqFKDkIWrzSiukK9EUGAkCoZdHLUrjERMR7NFI6ASQmdHMK78tQBNgodAZ0chFSnkdCtQQMkQYAQnRKqOYFIYhOaIQ8CQAx95pDHILREKgSQHFINh1zGqI43EcplK1oTKAJEURUvj18r+AkXgcQlXanta3p2bVj+289CgZ309tVQydMqDciBUwdPRMXrSqpdN182sHdbz61DqW0rE8uXQYGd7lsvH9i7rfOm9VpC/0eDhx1EVXDq4AFkMDoSa7r7927r3HEpScDA1fcJlV03rBt47FOJS3vqz3k51mz68KIH2wSAwLJEZvfG5Kou566S63ozuzZobRzmD5UQDlqczcWzXBDounF928pOGlUg1v3N9TSSDjKE6JcVBwE8JQsCbUOZ9JdW01uT3r627fI+enlbSbys2MIiXWX6+tVEZZjjQTj9hVU+3UBy+AQwoOZtF1JdUKzWJFYxN7E2h30kB4AQgZJYkWa1su0il6Wrq8IWIYcrDtILJBmuKaYzWqls7njeIjk8Qxdcw9SOtaQjwdrf/Dt51iZ18kiOOkCkO+y4dmXPzR9TFOaZY/6dKcXfB8nhDz/Brds3D/befqXeif6kIkMaDK4phefe0xv6+EFy+ABPcNPkUCZzx1XEyJVASEphmTymn3qz/OGs4u+D5PCHn7DWiYu7MvduJknrAOmzB02HpbcmZ555h0bSWcbat7Mkng0OgcT5qf4Ht6jptqVd6ssO51e8gbxWrkw+fpRUYNdvQXL4RZB7e9Lb3vfQVjXT0aiZ6FVEg49dLh1oMfP026e//Fz5bb//p+j9KPr9HOYObqVAgKQSMGckBlPNrAF+GOsPYAdcZaAo5XyxsO9k/snR8dv2Tf/mdVJmWLc268Wsx5nDxEGObVKFdUbbJd2u1hCFmGXyocNnv/5C/v7DhWeOl09MuzZkEkByMMElUlhVMndd1c6SSs3/enTuHyfF2YTkEIctm2aIZ3RsHKRvM/vH44XfH6eX9yCJ5PAAGv8m3d8eWvbpC+j1nvv7h1O/GqWX9yaJ5PCGG89WkDpJX7+GXuPcq2OTj/ybXt6zJJLDM3R8Gi6kTmi1FY9lcz99ReERxnDtEsnhClGDAL+KxdQJnc75E1O5nxxUSoFQQ8E4B92oiJBKDi2mTmj0l8cKE8MjWsHvXRo0fZkyOHOYOAS9tUudONlQyc1ldx7QJotOQrzPITl4I0qhr0nqpGnLyuw8zBnlU4WmEmJOIDnE4NpcK2meOrFtpJUqsM4ov8s5+mnbV10lkqMOELGHrqmTuu61spa775XSaK6uPphDJEcwOBu9JGlTJ4a0voF4RvHgmL4Xxg+SIyjU5UuduHrOkxyunbWygISpE9fhQHK4QsRBQM7UiatjSA5XiPwKSJs6cXUMyeEKkS8BmVMnro4hOVwh8i4geerE1TEkhytEHgXkT524OhZZciTVzpvW9z28dcWfvzj49Of7f7Et/dU1rt4GJpBY2525Z9PSp06cOi/n5iZ2jgScOnEyyDgXSXKQ/vb+R6/uumFd+8f7wQu1sy25pqf7W0OZPZtIdxJqwi3q+em+PZvVTipLwFRInWR3H6yc8vuAGqjiW6JHDtKTHHj0muS63kYgOjYODuzdlrjI70tLGjXT10DqpP+hLQm7p04Uu081deL7oWc73X7rokaOdhX+KBMrmj7WkRhM9T9+TfuVA36B8dQ+WqkTVxcjRQ6iZO7YaDtnWP1UU22ZPZtDWIIkI5Y6sYJmux8lcnTfdkXHpvNs3airJCqBJUjPjz6pqKTulKjDCKZOXKGIDDnS21enr7vY1R+rQOozF/b9bGswS9Qopk6sWNnuR4McHVev6LplyNYB58r2y/sCWKJGNHXiDB2cjQA5kpdleoc3EOLxAiF6iRrd1AkMv3ORlhxVsxMr00zRpGqzpb/ELVEjnTpZCpLNkdTkgJBGZs8mlUdcq7pE/f4nbDDwWtW+ZTCzcwN969LxfDbAp07oDWsmKTE5jJBGG92r4Ju5V1ef+tyqvkeu5rJE1VMnu66q0+9wWB4rZHcdVAJ86sTBGMpTspKDLqRB6aRVjMsSNRJPnVi99rYvKTnoQxoe3E74i6Im7F/Y1dQQSJ1MDI8E/9RJU4OoT8hIDtaQxrl9J2HSpnZZF4Qlat/9W1JfY07kqr3tmfs2q9SpE+gsd+ehUJ46ga59FunIwRrSmDt0ZvKBw+O37Ssey7Ji0XPLEFMUlaQSfQ9uoV8GaWUte/ehErthrI4IkpeLHKwhjdJbk7n7XtXfnTZVyv74QOFvH7DCxBBFjUDqhNV7F3mJyJFgDGmUTxeyuw8qxYX3EVS0/MOvTT15TKuwvU6Paokax9SJCzUURRZyQEgDFgH0IY0KTBW7RrR8qc7D2Wf+n7v75Uphvq7e+dB1iRrL1IkzJnBWDnIYIQ2HuzTAUGvRb5CBVd5J+1unii+PTXz3Xx6WqM0S/XFNnVghtd2XgByMIQ1N0yYfPFx63enZ4vL7M+PsS9RqFHVpoj/GqRNbQlgrwycHa0hj+sn/zb102uqD7b4G1x3fS9R4p05scbNWhkwO1pDGzJ+Oz/6B+uWbFWOJ+sSo1WGafVii9j9+TXrHWtbUSe6ug4G9sIvGEZ8yYZLDQ0hjmv3lm0Cm7J2HWJeobeenu/XvR6KFF5Y4kDrRZoJ7YRetZT7kQiOH55CGB2eLnpao9B2F8sIuevM8S4ZDDr8hDXZ3vS1RafqJburE1bsQyAEhDaa7NMqwtLzDJqTh6ludgL5Evf2lwvPv19X7PIxu6sTV8cDJYYQ0GNITpcokhDR8f11ZDYj8Y0em2KOotebWnainTqy+2O4HSw4BIQ1br5wrvUVRG3WG+8KuRnu41wRKDkEhDQ+gFH0vUfOCv+vEg1PcmwRHDtaQxuxf3p2lD2mwA+NniRrAd52wO8S/RUDk8BDSmNr7H/7uLtWoL1HZo6jBfNfJUkvDOAomKxtkSIMZRTOKSr1EDey7TpgdEdBA+MwRfEjDA0qUS9TiseC+68SDF9ybiCUHhDS43KXB3e1Gha5L1IC/66TRwuBrRJIDQhr3bKK/SwOcz959qNzkLg04K7qYS9Rz+081dlR44f3xH+wP8rtOGm0IvkYgOXqHNyTXZ+hdyu15ZX7U6S4NelWeJWGJOrnn1YnhA7MvflA+e64yM194/r2JnQfyPz8SreeRPCNgbSiKHN3fuWLZ1hXWnpz3p54YndvvfpeGsxJeZ0tHJqYeee3sjS+O7fhr/rGjpaMTvDRHS48QcsgW0ojWkMhjLX9yyBnS4Ih466jiTA6pQxqtM6qcPOVJjkiENDjh1hJqeJKjd/dG/w+etATqEXGSGzmWXbcquaaH0mvnB08olaCYaAT4kePaCyhtpXnwhFIViglFgBs5EuelKA2dpnvwhFIbiolDgBs5KFcbs8+eEHqXhjikWlAzN3IU35h0hW/u0JmpX/7XVQwFJEGAGzlKr407u1R7l4azGJ6VBwFu5Cg8ewKS2s0cq3+XRjM5rJcJAW7k0Gbms8Mjc0ds5o/iG7nxH+5vfJeGTDigLTYIcCMH6K5MFnPDI9l7Xy7882Q5e64yV547Oj791JvZ7+3XJuZAAEu0EOBJDtPz4ktn8g8cPvuNF8e+8lxu58jM794063EbOQT4kyNyEKDBzRBAcjRDJph6qXtBckg9POEah+QIF3+pe0dySD084RqH5AgXf6l7R3JIPTzhGofkCBd/qXtHckg9POEaJys5wkUFezcQQHIYMODGDgEkhx0qWGcggOQwYMCNHQKqprF9dY2dEqyLIQJADJw5YjiuvFwCchBeulBPvBAgQA68rMRlTDn7oQE5OKtEdbFBAMkRm6Hk7wiSgz+msdGI5IjNUHJ3hKiE4H8r3GGNg0LghT5zQLgjDt6gD/wQMCmhk4OfTtQUKwRaghyxGrFAnCFEX2zo5CCEmNNIIP1iJ7IjUCODTg7dWJ0o+m/8QQRqCCyQA2PoNUhwZwGBKjngyrJQg79bGgG4ptTIUCUH4AG1sMWCCNQQWCQH8AXv+6nh0po7QACgQc33RXJAFcFlKaDQwsVKAIBhCTngGC8uAEJrlsahrycHITh7tCY3FELqh76eHABMI4OgEku8EbAddBtyEFLPoHjjgt4BAoTYDLoNOUAUCyIACCA5AAQs9gg4kUNTMKhuj1psap2H2IkcRCGxQYGfI7HSRByH2IkcsYIBnWFHAMnBjlnLtEBytMxQsztKSw4IkjgvXti7xhZhIKAPJO3/GbTkIPBxXLyE4Sj2yY4AjCMhlM1oyUGpDsXihIAXcsD1BRL/MD/FCYi4+gLDpBdPISsv5CAKIUQHE3rVf5k/uJUMARgdKGAUgY9iDBgcsBQv5DD1Q49QzH3cSogAjI5ZPNvmnRy1LoGeZqnV4E6ICJhjAVv/NnAgR42eYJBZ/JuFGlgRgFWgYqAPDWsjAvt+CgdymN2bBplb3UjTUt1k8zxu+SOg46z/d6BjTWBRQaofXj1xI4fVIN1Gxdgout2GD8bGcmiVx31nBAzsjI0FQDiGVoQA0FAI7HMvHwEAAP//ONNcygAAAAZJREFUAwDF34PaB4z3FQAAAABJRU5ErkJggg=="},"/icon-512.png":{"b64":"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"}};
for (const k in ASSETS) if (ASSETS[k] && ASSETS[k].b64) ASSETS[k] = Buffer.from(ASSETS[k].b64, 'base64');
const __m = { exports: {} };
(function (module, exports) {
// OY Bot engine: price action chaser (stop and reverse) for Exness MT5 via MetaApi.
// Buy Stop + Sell Stop chase price on the 1-minute chart. When one triggers, the other becomes the reversal
// order, parked just beyond the group's stop-loss. Winners get breakeven then a trailing stop, so profit is
// banked before the reversal can fire. While in profit it adds trades paid for by secured profit.
// Lot size comes from the current balance, so it grows with the account. Hedging account required.
const MetaApi = require('metaapi.cloud-sdk').default;
const fs = require('fs');

const env = (k, d) => (process.env[k] !== undefined && process.env[k] !== '' ? process.env[k] : d);
const apiToken = env('METAAPI_TOKEN', '');
const PREFIX = 'OyBot';
const MAGIC = 26110301;
const STATE_FILE = env('STATE_FILE', './oy-state.json');

// Settings (clamped). Percent values are percent; the ATR values are in units of 1-minute ATR.
const S = {
  riskPerTradePct: 2, totalRiskPct: 6, maxDrawdownPct: 15, protectEquityPct: 60,
  pyramid: true, maxAdds: 3, addStepAtr: 0.6, addRiskFracOfProfit: 0.5, addRiskCapPct: 10,
  chaseDistAtr: 0.8, chaseStepAtr: 0.2, reverseAtr: 1.0, reverseGapAtr: 0.1, lockStartAtr: 0.5, trailAtr: 0.7,
  maxSpreadPct: 15, maxTradesPerDay: 150, maxMarginUsePct: 50, microBalanceUsd: 50, microMaxRiskPct: 8,
};
const LIMITS = {
  riskPerTradePct: [0.25, 5], totalRiskPct: [0.5, 20], maxDrawdownPct: [2, 50], protectEquityPct: [30, 95],
  maxAdds: [0, 6], addStepAtr: [0.2, 3], addRiskFracOfProfit: [0.1, 1], addRiskCapPct: [1, 20],
  chaseDistAtr: [0.3, 3], chaseStepAtr: [0.05, 1], reverseAtr: [0.4, 4], reverseGapAtr: [0.02, 1], lockStartAtr: [0.2, 3], trailAtr: [0.2, 3],
  maxSpreadPct: [2, 60], maxTradesPerDay: [1, 1000], maxMarginUsePct: [5, 90], microBalanceUsd: [0, 1000], microMaxRiskPct: [1, 20],
};

const st = {
  running: false, connected: false, linking: false, activeId: null, kind: null,
  symbol: env('SYMBOL', 'EURUSD'), broker: '', login: 0, currency: '',
  balance: 0, equity: 0, openProfit: 0, todayPL: 0, todayTrades: 0, wins: 0, losses: 0,
  positions: 0, open: [], pendings: [], spread: 0, atr: 0, locked: false, hwm: 0, calibrated: false,
  msg: apiToken ? 'Link an account to begin' : 'Server missing METAAPI_TOKEN',
};
let api = null, conn = null, account = null, loop = null, busy = false;
let lastStats = 0, lastRun = 0, atrMin = -1, atrVal = 0, volOk = true, dayKey = '', dayStartEquity = 0;
let gPeak = 0, gPeakDir = 0, mppObs = 0, nextAttempt = 0;
let P = {};                                   // persisted per-account protection state
try { P = JSON.parse(fs.readFileSync(STATE_FILE, 'utf8')); } catch { P = {}; }
let saveTimer = null;
const persist = () => { if (saveTimer) return; saveTimer = setTimeout(() => { saveTimer = null; try { fs.writeFileSync(STATE_FILE, JSON.stringify(P)); } catch {} }, 2000); };
const pst = () => (P[st.activeId] = P[st.activeId] || { hwm: 0, locked: false });

const errText = e => String((e && e.message) || e).slice(0, 140);
const isBuy = p => String(p.type).includes('BUY');
const cents = () => (/^[A-Za-z]{2}C$/.test(st.currency || '') ? 100 : 1);
const mine = p => p.symbol === st.symbol && (p.magic === MAGIC || String(p.comment || '').startsWith('OY'));

// ---------- accounts ----------
const getApi = () => { if (!apiToken) throw new Error('Server missing METAAPI_TOKEN'); return api || (api = new MetaApi(apiToken)); };
const parseName = n => { const p = String(n || '').split('|'); return { kind: p[1] === 'real' ? 'real' : 'demo', login: p[2] || '' }; };

async function listAccounts() {
  const all = await getApi().metatraderAccountApi.getAccountsWithInfiniteScrollPagination({});
  return all.filter(a => String(a.name || '').startsWith(PREFIX + '|')).map(a => ({
    id: a.id, login: a.login, server: a.server, state: a.state, kind: parseName(a.name).kind, active: a.id === st.activeId,
  }));
}
async function subscribe() { try { await conn.subscribeToMarketData(st.symbol); } catch (e) { st.msg = 'Symbol error: ' + errText(e); } }

async function useAccount(id) {
  st.running = false; st.connected = false; st.linking = true; st.activeId = id;
  atrMin = -1; mppObs = 0; st.calibrated = false; dayKey = ''; gPeak = 0; gPeakDir = 0;
  Object.assign(st, { balance: 0, equity: 0, openProfit: 0, todayPL: 0, todayTrades: 0, wins: 0, losses: 0, positions: 0, open: [], pendings: [], spread: 0 });
  try {
    if (conn) { try { await conn.close(); } catch {} conn = null; }
    st.msg = 'Connecting to account…';
    account = await getApi().metatraderAccountApi.getAccount(id);
    if (account.state !== 'DEPLOYED') await account.deploy();
    await account.waitConnected(90);
    conn = account.getStreamingConnection();
    await conn.connect();
    await conn.waitSynchronized({ timeoutInSeconds: 120 });
    await subscribe();
    st.kind = parseName(account.name).kind;
    st.broker = account.server || 'Exness'; st.login = account.login || 0;
    const ps = pst(); st.locked = !!ps.locked; st.hwm = ps.hwm || 0;
    st.connected = true; st.msg = st.locked ? 'PROTECTION LOCK is on. Use Reset protection to resume.' : 'Connected (' + st.kind.toUpperCase() + '). Press START.';
    if (!loop) loop = setInterval(tick, 700);
  } catch (e) { st.connected = false; st.msg = 'Could not connect: ' + errText(e); throw e; }
  finally { st.linking = false; }
}
async function addAccount({ login, password, server, kind }) {
  if (!login || !password || !server) throw new Error('Login, password and server are required');
  kind = kind === 'real' ? 'real' : 'demo';
  st.linking = true; st.running = false; st.msg = 'Linking ' + kind.toUpperCase() + ' account…';
  let created;
  try {
    created = await getApi().metatraderAccountApi.createAccount({
      name: PREFIX + '|' + kind + '|' + login, type: 'cloud', login: String(login), password, server, platform: 'mt5', magic: MAGIC,
    });
  } catch (e) { st.linking = false; st.msg = 'Link failed: ' + errText(e); throw e; }
  try { await useAccount(created.id); }
  catch (e) {
    try { const a = await getApi().metatraderAccountApi.getAccount(created.id); await a.remove(); } catch {}
    st.activeId = null; st.msg = 'Login failed. Check login, password and server name.'; throw new Error(st.msg);
  }
}
async function removeAccount(id) {
  const a = await getApi().metatraderAccountApi.getAccount(id);
  if (id === st.activeId) { st.running = false; st.connected = false; st.activeId = null; if (conn) { try { await conn.close(); } catch {} conn = null; } }
  await a.remove(); st.msg = 'Account removed';
}

// ---------- helpers ----------
const ts = () => conn.terminalState;
let spec = null;
const digits = () => (spec && spec.digits) || 5;
const tickSz = () => (spec && (spec.tickSize || spec.point)) || Math.pow(10, -digits());
const snap = p => { const t = tickSz(); return +(Math.round(p / t) * t).toFixed(digits()); };
const mpp = () => mppObs > 0 ? mppObs : ((spec && spec.contractSize) || 1) * cents();     // money per 1.0 price move per 1.0 lot
function minStopDist(px) {
  const pt = Math.pow(10, -digits());
  const lv = (spec.stopsLevel || spec.stopLevel || 0) * pt;
  return lv * 1.2 + (px.ask - px.bid) * 1.5 + pt;
}
function lotsFor(slDist, allowed, hardCapFn) {
  const step = spec.volumeStep || 0.01, minV = spec.minVolume || 0.01, maxV = spec.maxVolume || 100;
  const perLot = slDist * mpp();
  if (!(perLot > 0) || !(allowed > 0)) return { lot: 0, why: 'Bad stop distance' };
  let lot = Math.floor(allowed / perLot / step + 1e-9) * step;
  if (lot < minV) {
    if (!hardCapFn) return { lot: 0, why: 'Secured profit is too small for the next lot' };
    const cap = hardCapFn(allowed);
    if (minV * perLot > cap) return { lot: 0, why: 'Min lot risks ' + (minV * perLot).toFixed(2) + ' vs limit ' + allowed.toFixed(2) + '. Use a smaller-contract symbol or a Cent account.' };
    lot = minV;
  }
  lot = Math.min(maxV, lot);
  const dec = Math.max(0, Math.ceil(-Math.log10(step) - 1e-9));
  lot = +lot.toFixed(dec);
  return { lot, risk: lot * perLot };
}
const baseLot = slDist => lotsFor(slDist, st.balance * S.riskPerTradePct / 100,
  allowed => st.balance < S.microBalanceUsd * cents() ? st.balance * S.microMaxRiskPct / 100 : allowed * 1.5);
const addLot = (slDist, money) => lotsFor(slDist, money, null);
function marginOk(lot, price) {
  const T = ts().accountInformation || {};
  const lev = T.leverage || 100, free = T.freeMargin;
  if (!free) return true;
  const need = lot * ((spec.contractSize) || 1) * price / lev * cents();     // approximation; broker re-checks
  return need <= free * S.maxMarginUsePct / 100;
}
let failedFlag = false;
async function safe(label, fn) { try { return await fn(); } catch (e) { failedFlag = true; st.msg = label + ': ' + errText(e); return null; } }

async function dayStats() {
  try {
    const d0 = new Date(); d0.setUTCHours(0, 0, 0, 0);
    const deals = await Promise.resolve(conn.historyStorage.getDealsByTimeRange(d0, new Date(Date.now() + 6e4)));
    let pl = 0, n = 0, w = 0, l = 0;
    for (const x of deals || []) {
      if (x.symbol !== st.symbol) continue;
      if (x.entryType === 'DEAL_ENTRY_IN') n++;
      if (x.entryType === 'DEAL_ENTRY_OUT') { const net = (x.profit || 0) + (x.commission || 0) + (x.swap || 0); pl += net; net > 0 ? w++ : l++; }
    }
    st.todayPL = +pl.toFixed(2); st.todayTrades = n; st.wins = w; st.losses = l;
  } catch {}
}

async function refreshAtr() {
  const m = Math.floor(Date.now() / 6e4);
  if (m === atrMin) return true;
  const cs = await account.getHistoricalCandles(st.symbol, '1m', undefined, 130);
  if (!cs || cs.length < 40) { st.msg = 'Not enough 1-minute candles for ' + st.symbol; return false; }
  cs.sort((a, b) => new Date(a.time) - new Date(b.time));
  const c = cs.slice(0, -1), p = 14;
  const tr = c.map((x, i) => i === 0 ? x.high - x.low : Math.max(x.high - x.low, Math.abs(x.high - c[i - 1].close), Math.abs(x.low - c[i - 1].close)));
  let a = tr.slice(0, p).reduce((s, v) => s + v, 0) / p; const arr = [a];
  for (let i = p; i < tr.length; i++) { a = (a * (p - 1) + tr[i]) / p; arr.push(a); }
  const avg = arr.slice(-50).reduce((s, v) => s + v, 0) / Math.min(50, arr.length);
  atrVal = arr.at(-1); volOk = atrVal >= 0.6 * avg && atrVal <= 2.5 * avg; st.atr = atrVal; atrMin = m;
  return atrVal > 0;
}

// learn the real money-per-price figure from a live position (guards against wrong contract-size / cent maths)
function calibrate(pos, px) {
  const obs = [];
  for (const p of pos) {
    const move = (isBuy(p) ? px.bid - p.openPrice : p.openPrice - px.ask);
    if (Math.abs(move) >= 8 * tickSz() && p.profit !== undefined && p.volume > 0) {
      const v = p.profit / (move * p.volume);
      if (v > 0 && isFinite(v)) obs.push(v);
    }
  }
  if (obs.length) { obs.sort((a, b) => a - b); mppObs = obs[obs.length >> 1]; st.calibrated = true; }
}

function buildGroup(pos) {
  const g = { n: 0, dir: 0, mixed: false, lastOpen: 0, gsl: 0, secured: 0, allSl: true };
  let bestT = -1, have = false; const k = mpp();
  for (const p of pos) {
    const d = isBuy(p) ? 1 : -1;
    if (g.n === 0) g.dir = d; else if (d !== g.dir) g.mixed = true;
    g.n++;
    const t = +new Date(p.time || 0); if (t >= bestT) { bestT = t; g.lastOpen = p.openPrice; }
    if (!p.stopLoss) { g.allSl = false; continue; }
    g.secured += (p.stopLoss - p.openPrice) * d * p.volume * k;
    if (!have) { g.gsl = p.stopLoss; have = true; } else g.gsl = d === 1 ? Math.min(g.gsl, p.stopLoss) : Math.max(g.gsl, p.stopLoss);   // the stop FARTHEST from price: reversal fires only after every stop is hit
  }
  return g;
}
const worstLoss = p => { if (!p.stopLoss) return st.balance * S.riskPerTradePct / 100; const d = Math.max(0, isBuy(p) ? p.openPrice - p.stopLoss : p.stopLoss - p.openPrice); return d * p.volume * mpp(); };

async function trailAll(pos, dir, atr, minD, step, px) {
  const price = dir === 1 ? px.bid : px.ask;
  if (gPeakDir !== dir || gPeak <= 0) { gPeakDir = dir; gPeak = price; } else gPeak = dir === 1 ? Math.max(gPeak, price) : Math.min(gPeak, price);
  const T = Math.max(S.trailAtr * atr, minD * 1.5), R0 = Math.max(S.reverseAtr * atr, minD * 2), half = tickSz() / 2;
  for (const p of pos) {
    if ((isBuy(p) ? 1 : -1) !== dir) continue;
    const op = p.openPrice; let nsl = p.stopLoss || (dir === 1 ? op - R0 : op + R0);
    const profit = dir === 1 ? px.bid - op : op - px.ask;
    if (profit >= S.lockStartAtr * atr) {
      const be = dir === 1 ? op + 0.1 * atr : op - 0.1 * atr, tr = dir === 1 ? gPeak - T : gPeak + T;
      let cand = dir === 1 ? Math.max(be, tr) : Math.min(be, tr);
      cand = dir === 1 ? Math.min(cand, px.bid - minD) : Math.max(cand, px.ask + minD);
      if (dir === 1 ? cand > nsl + step : cand < nsl - step) nsl = cand;
    }
    nsl = snap(nsl);
    if (Math.abs(nsl - (p.stopLoss || 0)) > half) {
      const ok = await safe('Modify stop', () => conn.modifyPosition(p.id, nsl, p.takeProfit || undefined));
      if (ok !== null) p.stopLoss = nsl;
    }
  }
}

function mayOpen(R, px) {
  if (st.todayTrades >= S.maxTradesPerDay) { st.msg = 'Daily trade limit reached'; return false; }
  const sp = px.ask - px.bid;
  if (sp > S.maxSpreadPct / 100 * R) { st.msg = 'Spread ' + (sp / Math.pow(10, -digits())).toFixed(0) + ' pts is over ' + S.maxSpreadPct + '% of the stop, waiting'; return false; }
  return true;
}

async function scaleIn(pos, g, atr, minD, px) {
  if (!S.pyramid || g.mixed || !g.n || g.n >= 1 + S.maxAdds || !g.allSl || g.secured <= 0) return;
  const adv = g.dir === 1 ? px.bid - g.lastOpen : g.lastOpen - px.ask;
  if (adv < S.addStepAtr * atr) return;
  const entry = g.dir === 1 ? px.ask : px.bid;
  let sl = g.dir === 1 ? Math.min(g.gsl, px.bid - minD * 1.5) : Math.max(g.gsl, px.ask + minD * 1.5);
  sl = snap(sl); const dist = Math.abs(entry - sl);
  if (dist < minD) return;
  const allowed = Math.min(S.addRiskFracOfProfit * g.secured, st.equity * S.addRiskCapPct / 100);
  const L = addLot(dist, allowed);
  if (!L.lot) { st.msg = 'Scale-in waiting: secured profit ' + g.secured.toFixed(2) + ' is too small for the next lot'; return; }
  const open = pos.reduce((s, p) => s + worstLoss(p), 0);
  if (open + L.risk > st.balance * S.totalRiskPct / 100) { st.msg = 'Scale-in skipped: total risk cap'; return; }
  if (!mayOpen(dist, px)) return;
  if (!marginOk(L.lot, entry)) { st.msg = 'Scale-in skipped: not enough free margin'; return; }
  const o = { comment: 'OY-add', magic: MAGIC };
  const r = await safe('Scale-in failed', () => g.dir === 1 ? conn.createMarketBuyOrder(st.symbol, L.lot, sl, undefined, o) : conn.createMarketSellOrder(st.symbol, L.lot, sl, undefined, o));
  if (r !== null) { st.msg = 'SCALE-IN ' + (g.dir === 1 ? 'BUY ' : 'SELL ') + L.lot + ' | secured ' + g.secured.toFixed(2) + ', add risks ' + L.risk.toFixed(2); lastStats = 0; }
}

async function placeStop(buy, price, sl, R) {
  const L = baseLot(R);
  if (!L.lot) { failedFlag = true; st.msg = L.why; return false; }
  if (!marginOk(L.lot, price)) { failedFlag = true; st.msg = 'Not enough free margin for this trade'; return false; }
  const o = { comment: 'OY-C', magic: MAGIC };
  const r = await safe('Stop order failed', () => buy ? conn.createStopBuyOrder(st.symbol, L.lot, price, sl, undefined, o) : conn.createStopSellOrder(st.symbol, L.lot, price, sl, undefined, o));
  return r !== null;
}

async function closeEverything() {
  const T = ts();
  for (const o of (T.orders || []).filter(mine)) await safe('Cancel', () => conn.cancelOrder(o.id));
  for (const p of (T.positions || []).filter(mine)) await safe('Close', () => conn.closePosition(p.id));
}

// ---------- the chaser ----------
async function chase(T, px, allowNew) {
  failedFlag = false;
  const pos = (T.positions || []).filter(mine).map(p => ({ ...p }));
  const ords = (T.orders || []).filter(mine);
  let b = null, s = null;
  for (const o of ords) {
    const ty = String(o.type);
    if (ty.includes('BUY_STOP') && !ty.includes('LIMIT')) { if (!b) b = o; else await safe('Cancel', () => conn.cancelOrder(o.id)); }
    else if (ty.includes('SELL_STOP') && !ty.includes('LIMIT')) { if (!s) s = o; else await safe('Cancel', () => conn.cancelOrder(o.id)); }
  }
  st.pendings = ords.map(o => ({ side: String(o.type).includes('BUY') ? 'BUY STOP' : 'SELL STOP', price: o.openPrice }));
  const atr = atrVal, minD = minStopDist(px);
  const D = Math.max(S.chaseDistAtr * atr, minD * 1.5), R = Math.max(S.reverseAtr * atr, minD * 2), step = Math.max(S.chaseStepAtr * atr, minD * 0.5), half = tickSz() / 2;
  let g = buildGroup(pos);
  if (g.mixed) { st.msg = 'Reversal in progress'; return; }

  if (g.n > 0) {                                           // trades open: lock profit, scale in, keep the reversal order on the stop
    const dir = g.dir;
    if (dir === 1 && b) { await safe('Cancel', () => conn.cancelOrder(b.id)); b = null; }
    if (dir === -1 && s) { await safe('Cancel', () => conn.cancelOrder(s.id)); s = null; }
    await trailAll(pos, dir, atr, minD, step, px);
    g = buildGroup(pos);
    if (allowNew) await scaleIn(pos, g, atr, minD, px);
    if (!(g.gsl > 0)) return;
    const gap = Math.max(S.reverseGapAtr * atr, minD * 0.5);
    const rev = snap(dir === 1 ? g.gsl - gap : g.gsl + gap);
    const revOk = dir === 1 ? rev <= px.bid - minD : rev >= px.ask + minD;
    const r = dir === 1 ? s : b;
    if (!allowNew || !revOk || !mayOpen(R, px)) { if (r) await safe('Cancel', () => conn.cancelOrder(r.id)); }
    else {
      const rsl = snap(dir === 1 ? rev + R : rev - R);
      if (r) { if (Math.abs(r.openPrice - rev) > half) await safe('Modify reversal', () => conn.modifyOrder(r.id, rev, rsl, undefined)); }
      else await placeStop(dir !== 1, rev, rsl, R);
    }
    if (!failedFlag && !/scale-in/i.test(st.msg)) st.msg = (dir === 1 ? 'LONG x' : 'SHORT x') + g.n + ' | stop ' + g.gsl.toFixed(digits()) + (g.secured > 0 ? ' | profit secured ' + g.secured.toFixed(2) : ' | initial stop');
    return;
  }

  gPeak = 0; gPeakDir = 0;                                 // flat: keep a Buy Stop and a Sell Stop chasing price
  if (!allowNew || !volOk) {
    for (const o of ords) await safe('Cancel', () => conn.cancelOrder(o.id));
    if (st.locked) st.msg = 'PROTECTION LOCK active. Use Reset protection to resume.';
    else if (!st.running) st.msg = 'Stopped: no orders placed.';
    else st.msg = 'Market too quiet or too wild, orders removed';
    return;
  }
  if (!mayOpen(R, px)) { for (const o of ords) await safe('Cancel', () => conn.cancelOrder(o.id)); return; }
  const bp = snap(px.ask + D), sp = snap(px.bid - D);
  if (!b) await placeStop(true, bp, snap(bp - R), R);
  else if (b.openPrice - px.ask > D + step) await safe('Modify', () => conn.modifyOrder(b.id, bp, snap(bp - R), undefined));
  if (!s) await placeStop(false, sp, snap(sp + R), R);
  else if (px.bid - s.openPrice > D + step) await safe('Modify', () => conn.modifyOrder(s.id, sp, snap(sp + R), undefined));
  if (!failedFlag) st.msg = 'Chasing price: Buy Stop and Sell Stop set';
}

// ---------- main loop ----------
async function tick() {
  if (busy || !conn || !st.connected || st.linking) return;
  if (Date.now() < nextAttempt) return;
  busy = true;
  try {
    const T = ts(), info = T.accountInformation;
    if (info) { st.balance = info.balance; st.equity = info.equity; st.currency = info.currency || st.currency; }
    spec = T.specification(st.symbol); const px = T.price(st.symbol);
    const pos = (T.positions || []).filter(mine);
    st.positions = pos.length;
    st.open = pos.map(p => ({ id: p.id, side: isBuy(p) ? 'BUY' : 'SELL', volume: p.volume, profit: +((p.profit || 0) + (p.swap || 0)).toFixed(2) }));
    st.openProfit = +st.open.reduce((a, p) => a + p.profit, 0).toFixed(2);
    if (Date.now() - lastStats > 15000) { lastStats = Date.now(); await dayStats(); }
    if (!spec || !px || !px.ask || !px.bid) { st.msg = 'Waiting for ' + st.symbol + ' prices (check the symbol name)'; return; }
    st.spread = Math.round((px.ask - px.bid) / Math.pow(10, -digits()));
    if (pos.length) calibrate(pos, px);

    const dk = new Date().toISOString().slice(0, 10);
    if (dk !== dayKey) { dayKey = dk; dayStartEquity = st.equity; }
    const ps = pst();
    if (st.equity > (ps.hwm || 0)) { ps.hwm = st.equity; persist(); }
    st.hwm = ps.hwm || 0; st.locked = !!ps.locked;

    if (!st.locked && st.hwm > 0 && st.equity > 0 && st.equity <= st.hwm * S.protectEquityPct / 100) {      // equity protection lock
      await closeEverything(); ps.locked = true; st.locked = true; st.running = false; persist();
      st.msg = 'PROTECTION LOCK: equity fell to ' + S.protectEquityPct + '% of its peak (' + st.hwm.toFixed(2) + '). All closed, bot stopped.'; return;
    }
    if (st.running && dayStartEquity > 0 && st.equity <= dayStartEquity * (1 - S.maxDrawdownPct / 100)) {  // daily drawdown guard
      await closeEverything(); st.running = false;
      st.msg = 'Daily drawdown ' + S.maxDrawdownPct + '% hit: all trades closed, bot stopped.'; return;
    }
    if (Date.now() - lastRun < 600) return; lastRun = Date.now();
    if (!(await refreshAtr())) return;
    await chase(T, px, st.running && !st.locked);
  } catch (e) {
    st.msg = 'Error: ' + errText(e);
    if (/429|too many|rate/i.test(String(e && e.message))) nextAttempt = Date.now() + 5000;
  } finally { busy = false; }
}

async function start() {
  if (!apiToken) return;
  try { const l = await listAccounts(); if (l.length === 1) await useAccount(l[0].id); }
  catch (e) { st.msg = 'Startup: ' + errText(e); }
}

module.exports = {
  start, listAccounts, addAccount, removeAccount, useAccount,
  status: () => ({ ...st, settings: { ...S } }),
  isOnline: () => st.connected,
  async setSymbol(x) {
    x = String(x || '').trim(); if (!/^[A-Za-z0-9._#-]{3,20}$/.test(x)) throw new Error('Bad symbol');
    if (st.positions > 0) throw new Error('Close open trades before changing the symbol');
    if (conn) for (const o of (ts().orders || []).filter(mine)) await safe('Cancel', () => conn.cancelOrder(o.id));
    st.symbol = x; atrMin = -1; mppObs = 0; st.calibrated = false; if (conn) await subscribe();
  },
  setSettings(o) {
    if (o.pyramid !== undefined) S.pyramid = !!o.pyramid;
    for (const k of Object.keys(LIMITS)) {
      if (o[k] === undefined || o[k] === '') continue;
      const v = +o[k]; if (!isFinite(v)) throw new Error('Bad value for ' + k);
      S[k] = Math.min(LIMITS[k][1], Math.max(LIMITS[k][0], v));
    }
    st.msg = 'Settings saved';
  },
  async command(c, o = {}) {
    if (c === 'start') {
      if (!st.connected) throw new Error('Link an account first');
      if (st.locked) throw new Error('Protection lock is on. Reset protection first.');
      if (st.kind === 'real' && !o.confirmReal) throw new Error('Real account: confirmation required');
      st.running = true; atrMin = -1; st.msg = 'Started on ' + st.kind.toUpperCase() + ' account';
    } else if (c === 'stop') {
      st.running = false;
      if (conn) for (const x of (ts().orders || []).filter(mine)) await safe('Cancel', () => conn.cancelOrder(x.id));
      st.msg = 'Stopped. Pending orders removed; open trades keep their trailing stops.';
    } else if (c === 'closeall') {
      st.running = false; if (conn) { await closeEverything(); st.msg = 'Everything closed, bot stopped'; }
    } else if (c === 'reset') {
      if (!st.activeId) throw new Error('No account'); const ps = pst(); ps.locked = false; ps.hwm = st.equity; st.locked = false; persist();
      st.msg = 'Protection reset. Press START to resume.';
    } else throw new Error('unknown command');
  },
  _t: { tick, st, S, setConn(c, a) { conn = c; account = a; st.connected = true; st.activeId = 'test'; st.kind = 'demo'; }, resetClock() { atrMin = -1; lastRun = 0; lastStats = 0; }, resetState() { P = {}; gPeak = 0; gPeakDir = 0; mppObs = 0; st.calibrated = false; nextAttempt = 0; dayKey = ''; st.locked = false; st.hwm = 0; } },
};

})(__m, __m.exports);
const bot = __m.exports;
// OY Bot: single-file website (server + app + trading engine).
// Server: serves the website + API, runs the engine. (Single-file build injects ASSETS and `bot`.)
const http = require('http');

const TOKEN = process.env.TOKEN || 'CHANGE_ME';
const PORT = process.env.PORT || 3000;
const TYPES = { html: 'text/html; charset=utf-8', js: 'text/javascript', json: 'application/json', svg: 'image/svg+xml', png: 'image/png' };

const body = req => new Promise(res => {
  let d = ''; req.on('data', c => { d += c; if (d.length > 1e5) req.destroy(); });
  req.on('end', () => { try { res(JSON.parse(d || '{}')); } catch { res({}); } });
});
const send = (res, code, obj) => { res.writeHead(code, { 'Content-Type': 'application/json', 'Cache-Control': 'no-store' }); res.end(JSON.stringify(obj)); };
const authed = req => req.headers['x-token'] === TOKEN;
const em = e => String((e && e.message) || e).slice(0, 160);

http.createServer(async (req, res) => {
  const url = req.url.split('?')[0];
  if (url.startsWith('/api/')) {
    if (!authed(req)) return send(res, 401, { error: 'bad token' });
    try {
      if (url === '/api/status' && req.method === 'GET') return send(res, 200, { online: bot.isOnline(), status: bot.status() });
      if (url === '/api/accounts' && req.method === 'GET') return send(res, 200, { accounts: await bot.listAccounts() });
      if (req.method === 'POST') {
        const b = await body(req);
        if (url === '/api/cmd') { await bot.command(b.cmd, b); return send(res, 200, { ok: b.cmd }); }
        if (url === '/api/symbol') { await bot.setSymbol(b.symbol); return send(res, 200, { ok: true }); }
        if (url === '/api/settings') { bot.setSettings(b); return send(res, 200, { ok: true }); }
        if (url === '/api/select') { bot.useAccount(b.id).catch(() => {}); return send(res, 202, { queued: true }); }
        if (url === '/api/remove') { await bot.removeAccount(b.id); return send(res, 200, { ok: true }); }
        if (url === '/api/link') { bot.addAccount(b).catch(() => {}); return send(res, 202, { queued: true }); }
      }
      return send(res, 404, { error: 'not found' });
    } catch (e) { return send(res, 400, { error: em(e) }); }
  }
  const key = url === '/' ? '/index.html' : url;
  const a = ASSETS[key];
  if (!a) { res.writeHead(404); return res.end('Not found'); }
  res.writeHead(200, { 'Content-Type': TYPES[key.split('.').pop()] || 'text/plain', 'Cache-Control': key === '/sw.js' ? 'no-cache' : 'no-cache' });
  res.end(a);
}).listen(PORT, () => { console.log('Server on ' + PORT); bot.start(); });
{
  "name": "oy-phone-bot",
  "version": "1.0.0",
  "main": "app.js",
  "scripts": {
    "start": "node app.js"
  },
  "dependencies": {
    "metaapi.cloud-sdk": "^29.0.0"
  },
  "engines": {
    "node": ">=18"
  }
}
