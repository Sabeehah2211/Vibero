<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Vibero | Next-Gen 3D Social</title>
    <link href="https://fonts.googleapis.com/css2?family=Orbitron:wght@400;700&family=Rajdhani:wght@300;500;700&display=swap" rel="stylesheet">
    <style>
        :root {
            --bg-color: #000000;
            --accent-cyan: #00ffff;
            --accent-green: #00ff00;
            --error-red: #ff3333;
            --panel-bg: #0d0d0d;
            --sidebar-width: 280px;
            --skin-tone: #f3c3ad;
            --hair-color: #d4a017;
            --hair-shadow: #7a5c00;
            --clothing-color: #1a1a1a;
        }

        body, html { margin: 0; padding: 0; background-color: var(--bg-color); color: white; font-family: 'Rajdhani', sans-serif; overflow: hidden; height: 100%; }
        h1, h2, h3 { font-family: 'Orbitron', sans-serif; text-transform: uppercase; letter-spacing: 2px; margin-bottom: 15px; }
        input, select { background: #1a1a1a; border: 1px solid #333; color: white; padding: 14px; border-radius: 5px; font-family: inherit; margin-bottom: 12px; width: 100%; box-sizing: border-box; font-size: 1rem; }
        label { display: block; margin-bottom: 5px; font-size: 0.8rem; color: #888; text-transform: uppercase; letter-spacing: 1px; }

        /* --- SCREEN SYSTEM --- */
        .screen { display: none; height: 100vh; width: 100vw; flex-direction: column; position: absolute; top: 0; left: 0; box-sizing: border-box; background: var(--bg-color); z-index: 10; overflow-y: auto; padding-top: 20px; }
        .active { display: flex; }
        .centered { align-items: center; justify-content: center; padding: 20px; }

        /* --- UI COMPONENTS --- */
        .card-panel { background: var(--panel-bg); border: 1px solid #222; padding: 35px; border-radius: 12px; width: 100%; max-width: 400px; box-shadow: 0 10px 30px rgba(0,0,0,0.5); position: relative; }
        .btn { background: transparent; border: 2px solid var(--accent-cyan); color: var(--accent-cyan); padding: 15px; cursor: pointer; font-family: 'Orbitron', sans-serif; font-weight: bold; transition: 0.3s; text-align: center; text-decoration: none; display: block; width: 100%; box-sizing: border-box; font-size: 0.9rem; }
        .btn:hover { background: var(--accent-cyan); color: black; box-shadow: 0 0 20px var(--accent-cyan); }
        .btn-ghost { border: none; color: #888; font-size: 0.8rem; margin-top: 10px; cursor: pointer; background: none; }
        .error-msg { color: var(--error-red); font-size: 0.9rem; margin-bottom: 15px; display: none; padding: 10px; background: rgba(255,51,51,0.1); border-radius: 5px; border: 1px solid var(--error-red); }

        /* --- TOP NAV --- */
        .top-nav { height: 75px; width: 100%; background: #000; border-bottom: 1px solid #1a1a1a; display: flex; align-items: center; justify-content: space-between; padding: 0 25px; box-sizing: border-box; z-index: 100; position: sticky; top: 0; }
        .hamburger { font-size: 30px; cursor: pointer; color: var(--accent-cyan); }
        .user-info { text-align: right; }
        .vib-bal { color: var(--accent-green); font-family: 'Orbitron'; font-size: 0.8rem; }

        /* --- SIDEBAR --- */
        .side-menu { position: fixed; top: 0; left: -300px; width: var(--sidebar-width); height: 100%; background: #050505; border-right: 1px solid var(--accent-cyan); transition: 0.3s cubic-bezier(0.4, 0, 0.2, 1); z-index: 1000; padding: 30px; box-sizing: border-box; }
        .side-menu.open { left: 0; }
        .menu-item { padding: 20px 0; border-bottom: 1px solid #111; cursor: pointer; font-family: 'Orbitron'; font-size: 0.85rem; color: #ccc; }
        .menu-item:hover { color: var(--accent-cyan); padding-left: 10px; }
        .overlay { position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.9); display: none; z-index: 900; }

        /* --- 3D ENGINE --- */
        #avatar-preview-window { width: 100%; height: 420px; background: radial-gradient(circle, #222, #000); perspective: 800px; display: flex; align-items: center; justify-content: center; border-radius: 15px; margin-bottom: 25px; overflow: hidden; }
        .scene { width: 200px; height: 350px; transform-style: preserve-3d; }
        .pivot { width: 100%; height: 100%; position: relative; transform-style: preserve-3d; transform: rotateY(0deg); transition: transform 0.1s linear; }
        
        .cube { position: absolute; transform-style: preserve-3d; }
        .face { position: absolute; background: var(--skin-tone); border: 1px solid rgba(0,0,0,0.2); box-sizing: border-box; }
        
        /* Clothing Layers */
        .clothing-layer { position: absolute; transform-style: preserve-3d; pointer-events: none; display: none; }
        .clothing-face { position: absolute; background: var(--clothing-color); border: 1px solid rgba(255,255,255,0.05); }

        /* Body Parts */
        .head { width: 50px; height: 50px; left: 75px; top: 20px; }
        .torso { width: 90px; height: 110px; left: 55px; top: 75px; }
        .arm-l { width: 35px; height: 110px; left: 15px; top: 75px; }
        .arm-r { width: 35px; height: 110px; left: 150px; top: 75px; }
        .leg-l { width: 40px; height: 120px; left: 55px; top: 185px; }
        .leg-r { width: 40px; height: 120px; left: 105px; top: 185px; }

        /* Hair System */
        #hair-system { position: absolute; width: 50px; height: 50px; transform-style: preserve-3d; display: none; top: 0; left: 0; z-index: 5; }
        .hair-clump { position: absolute; transform-style: preserve-3d; }
        .hc-face { position: absolute; background: var(--hair-color); border: 0.5px solid var(--hair-shadow); }

        /* UI TABS */
        .tabs { display: flex; overflow-x: auto; gap: 10px; margin-bottom: 20px; padding-bottom: 5px; border-bottom: 1px solid #222; }
        .tab { background: #111; border: 1px solid #333; color: white; padding: 10px 20px; cursor: pointer; font-family: 'Orbitron'; font-size: 0.7rem; white-space: nowrap; }
        .tab.active { border-color: var(--accent-cyan); color: var(--accent-cyan); }
    </style>
</head>
<body>

    <div id="overlay" class="overlay" onclick="toggleMenu()"></div>
    <div id="side-menu" class="side-menu">
        <h2 style="color: var(--accent-cyan);">VIBERO</h2>
        <div class="menu-item" onclick="showScreen('home-screen'); toggleMenu();">Home</div>
        <div class="menu-item" onclick="showScreen('friends-screen'); toggleMenu();">Squad</div>
        <div class="menu-item" onclick="showScreen('avatar-screen'); toggleMenu();">Avatar Labs</div>
        <div class="menu-item" onclick="showScreen('vibrux-screen'); toggleMenu();">Vibrux</div>
        <div class="menu-item" onclick="showScreen('viber-create-screen'); toggleMenu();">Viber Create</div>
        <div class="menu-item" onclick="showScreen('settings-screen'); toggleMenu();">Settings</div>
        <div class="menu-item" style="color:var(--error-red); margin-top:50px" onclick="logout()">Logout</div>
    </div>

    <div id="welcome-screen" class="screen active centered">
        <h1 style="font-size: 4rem; color: var(--accent-cyan);">VIBERO</h1>
        <p style="color: #666; margin-bottom: 40px;">NEXT-GEN 3D SOCIAL EXPERIENCE</p>
        <div style="width: 100%; max-width: 320px; display: flex; flex-direction: column; gap: 15px;">
            <button class="btn" onclick="showScreen('login-screen')">LOGIN</button>
            <button class="btn" style="border-color:#555; color:#fff" onclick="showScreen('signup-screen')">JOIN VIBERO</button>
        </div>
    </div>

    <div id="login-screen" class="screen centered">
        <div class="card-panel">
            <h2>ACCESS</h2>
            <div id="login-error" class="error-msg">Account not found. Advance to Join Vibero? <br><br> <button class="btn" style="padding:5px" onclick="showScreen('signup-screen')">YES</button></div>
            <label>Username</label><input type="text" id="login-user">
            <label>Password</label><input type="password" id="login-pass">
            <button class="btn" onclick="handleLogin()">ENTER</button>
            <button class="btn-ghost" onclick="showScreen('welcome-screen')">CANCEL</button>
        </div>
    </div>

    <div id="signup-screen" class="screen centered">
        <div class="card-panel" style="max-width: 450px;">
            <h2>JOIN VIBERO</h2>
            <div id="reg-error" class="error-msg"></div>
            <label>Choose Username</label><input type="text" id="reg-user">
            <label>Create Password</label><input type="password" id="reg-pass">
            <label>Birth Date</label>
            <div style="display: flex; gap: 10px;">
                <select id="reg-day"><option>Day</option></select>
                <select id="reg-month"><option>Month</option></select>
                <select id="reg-year"><option>Year</option></select>
            </div>
            <label>Gender</label>
            <select id="reg-gender"><option value="">Select Gender</option><option>Male</option><option>Female</option><option>Viber</option></select>
            <button class="btn" onclick="handleSignup()">CREATE ACCOUNT</button>
            <button class="btn-ghost" onclick="showScreen('welcome-screen')">BACK</button>
        </div>
    </div>

    <div id="home-screen" class="screen">
        <div class="top-nav">
            <div class="hamburger" onclick="toggleMenu()">☰</div>
            <div class="user-info"><div id="display-name" style="color:var(--accent-cyan); font-family: 'Orbitron';">USER</div><div class="vib-bal"><span id="bal-val">0</span> VIB</div></div>
        </div>
        <div style="padding: 25px;">
            <div style="display:grid; grid-template-columns: 1fr 1fr; gap:15px; margin-bottom:30px;">
                <button class="btn" onclick="showScreen('friends-screen')">MY SQUAD</button>
                <button class="btn" onclick="showScreen('search-screen')">SEARCH</button>
            </div>
            <h3>TRENDING NOW</h3>
            <div style="height:200px; background:linear-gradient(45deg, #111, #222); border-radius:15px; border:1px solid #333; display:flex; align-items:flex-end; padding:25px;">
                <h2 style="margin:0">CYBER CITY WORLD</h2>
            </div>
        </div>
    </div>

    <div id="friends-screen" class="screen">
        <div class="top-nav"><div class="hamburger" onclick="toggleMenu()">☰</div><h3>SQUAD</h3></div>
        <div id="squad-list" style="padding:25px;"></div>
    </div>

    <div id="search-screen" class="screen">
        <div class="top-nav"><div class="hamburger" onclick="toggleMenu()">☰</div><h3>FIND VIBERS</h3></div>
        <div style="padding:25px;">
            <input type="text" id="search-input" placeholder="Search usernames..." oninput="runSearch(this.value)">
            <div id="search-results"></div>
        </div>
    </div>

    <div id="avatar-screen" class="screen">
        <div class="top-nav"><div class="hamburger" onclick="toggleMenu()">☰</div><h3>AVATAR LABS</h3></div>
        <div style="display:flex; border-bottom:1px solid #222;">
            <button class="btn" style="border:none; flex:1" onclick="setLab('market')">MARKET</button>
            <button class="btn" style="border:none; flex:1" onclick="setLab('custom')">CUSTOMIZE</button>
        </div>
        <div id="lab-market" style="padding:25px;">
            <div style="display:grid; grid-template-columns:1fr 1fr; gap:15px;">
                <div class="card-panel"><h3>HOODIE</h3><button class="btn" onclick="buy('hoodie', 0)">FREE</button></div>
                <div class="card-panel"><h3>HAIR</h3><button class="btn" onclick="buy('hair', 150)">150 VIB</button></div>
            </div>
        </div>
        <div id="lab-custom" style="display:none; padding:25px;">
            <div id="avatar-preview-window"><div class="scene"><div class="pivot" id="char-mesh"></div></div></div>
            <div class="tabs">
                <div class="tab" onclick="loadCat('hair')">HAIR</div>
                <div class="tab" onclick="loadCat('clothing')">CLOTHING</div>
                <div class="tab" onclick="loadCat('skin')">SKIN</div>
            </div>
            <div id="custom-grid" style="display:grid; grid-template-columns:1fr 1fr; gap:10px;"></div>
        </div>
    </div>

    <div id="vibrux-screen" class="screen">
        <div class="top-nav"><div class="hamburger" onclick="toggleMenu()">☰</div><h3>VIBRUX VAULT</h3></div>
        <div style="padding:25px; display:grid; gap:20px;">
            <div class="card-panel"><h2>500 VIB</h2><p>R100</p><button class="btn" onclick="checkout(500, 100)">AUTHORIZE</button></div>
            <div class="card-panel"><h2>1000 VIB</h2><p>R500</p><button class="btn" onclick="checkout(1000, 500)">AUTHORIZE</button></div>
        </div>
    </div>

    <div id="checkout-screen" class="screen centered">
        <div class="card-panel" style="max-width:450px;">
            <h2 style="text-align:center">SECURE GATEWAY</h2>
            <p id="check-desc" style="text-align:center; color:var(--accent-cyan); margin-bottom:20px;"></p>
            <label>Card Number</label><input type="text" maxlength="16" placeholder="0000 0000 0000 0000">
            <div style="display:flex; gap:10px;">
                <div style="flex:1"><label>Expiry</label><input type="text" placeholder="MM/YY"></div>
                <div style="flex:1"><label>CVV</label><input type="password" maxlength="3"></div>
            </div>
            <button class="btn" style="color:var(--accent-green); border-color:var(--accent-green); margin-top:20px;" onclick="finishPay()">PROCESS PAYMENT</button>
            <button class="btn-ghost" onclick="showScreen('vibrux-screen')">BACK</button>
        </div>
    </div>

    <div id="viber-create-screen" class="screen">
        <div class="top-nav"><div class="hamburger" onclick="toggleMenu()">☰</div><h3>STUDIO</h3></div>
        <div class="centered" style="text-align:center; height: 80%;">
            <h1 style="font-size:3rem">VIBER CREATE</h1>
            <p style="color:#666">THE WORLD IS YOUR CANVAS</p>
            <button class="btn" style="max-width:300px; margin:30px auto">START NEW VIBE</button>
        </div>
    </div>

    <div id="settings-screen" class="screen">
        <div class="top-nav"><div class="hamburger" onclick="toggleMenu()">☰</div><h3>SETTINGS</h3></div>
        <div style="padding:25px"><button class="btn" style="color:var(--error-red)" onclick="logout()">LOGOUT</button></div>
    </div>

    <script>
        const DB = { users: { "ADMIN": { pass: "123", vib: 1000, friends: [], inv: [] } }, cur: null, pend: null };
        let eq = { hair: false, hoodie: false };

        function showScreen(id) { document.querySelectorAll('.screen').forEach(s => s.classList.remove('active')); document.getElementById(id).classList.add('active'); }
        function toggleMenu() { const m = document.getElementById('side-menu'); const o = document.getElementById('overlay'); m.classList.toggle('open'); o.style.display = m.classList.contains('open') ? 'block' : 'none'; }
        
        const d = document.getElementById('reg-day'), y = document.getElementById('reg-year'), m = document.getElementById('reg-month');
        for(let i=1; i<=31; i++) d.innerHTML += `<option>${i}</option>`;
        for(let i=2024; i>=1950; i--) y.innerHTML += `<option>${i}</option>`;
        ["Jan","Feb","Mar","Apr","May","Jun","Jul","Aug","Sep","Oct","Nov","Dec"].forEach((n,i) => m.innerHTML += `<option value="${i}">${n}</option>`);

        function handleLogin() {
            const u = document.getElementById('login-user').value.toUpperCase();
            if(DB.users[u] && DB.users[u].pass === document.getElementById('login-pass').value) { DB.cur = u; loginOk(); }
            else { document.getElementById('login-error').style.display = 'block'; }
        }
        function handleSignup() {
            const u = document.getElementById('reg-user').value.toUpperCase(), p = document.getElementById('reg-pass').value, year = parseInt(document.getElementById('reg-year').value);
            const err = document.getElementById('reg-error');
            if(!u || !p || isNaN(year)) { err.innerText = "All fields required"; err.style.display="block"; return; }
            if(2024 - year < 7) { err.innerText = "Vibero is 7+ only"; err.style.display="block"; return; }
            DB.users[u] = { pass: p, vib: 0, friends: [], inv: [] }; DB.cur = u; loginOk();
        }
        function loginOk() { document.getElementById('display-name').innerText = DB.cur; updateUI(); showScreen('home-screen'); initAvatar(); }
        function logout() { DB.cur = null; showScreen('welcome-screen'); toggleMenu(); }

        function runSearch(q) {
            const res = document.getElementById('search-results'); res.innerHTML = ""; if(!q) return;
            Object.keys(DB.users).forEach(u => { if(u !== DB.cur && u.includes(q.toUpperCase())) res.innerHTML += `<div style="background:#111; padding:15px; border:1px solid #222; margin-top:10px; display:flex; justify-content:space-between;">${u} <button class="btn" style="width:60px; padding:5px" onclick="addF('${u}')">ADD</button></div>`; });
        }
        function addF(u) { if(!DB.users[DB.cur].friends.includes(u)) { DB.users[DB.cur].friends.push(u); alert("Added!"); updateUI(); } }

        function updateUI() {
            const u = DB.users[DB.cur]; document.getElementById('bal-val').innerText = u.vib;
            const list = document.getElementById('squad-list'); list.innerHTML = "";
            u.friends.forEach(f => list.innerHTML += `<div style="background:#111; padding:15px; border:1px solid #222; margin-bottom:10px;">${f} - <span style="color:var(--accent-green)">ONLINE</span></div>`);
        }

        // --- LAYERED ENGINE ---
        function createCube(w, h, d, className) {
            const cube = document.createElement('div'); cube.className = 'cube ' + className;
            cube.style.width = w + 'px'; cube.style.height = h + 'px';
            const faces = [
                { t: `translateZ(${d/2}px)`, w, h, f:'front' }, { t: `rotateY(180deg) translateZ(${d/2}px)`, w, h, f:'back' },
                { t: `rotateY(-90deg) translateZ(${w/2}px)`, w:d, h, f:'left' }, { t: `rotateY(90deg) translateZ(${w/2}px)`, w:d, h, f:'right' },
                { t: `rotateX(90deg) translateZ(${d/2}px)`, w, h:d, f:'top' }, { t: `rotateX(-90deg) translateZ(${h - d/2}px)`, w, h:d, f:'bottom' }
            ];
            faces.forEach(faceData => {
                const face = document.createElement('div'); face.className = 'face ' + faceData.f;
                face.style.width = faceData.w + 'px'; face.style.height = faceData.h + 'px'; face.style.transform = faceData.t;
                cube.appendChild(face);
            });
            return cube;
        }

        function createClothingLayer(w, h, d, scale, className) {
            const shell = createCube(w * scale, h * scale, d * scale, 'clothing-layer ' + className);
            shell.querySelectorAll('.face').forEach(f => { f.classList.remove('face'); f.classList.add('clothing-face'); });
            shell.style.left = `-${(w * scale - w) / 2}px`; shell.style.top = `-${(h * scale - h) / 2}px`; shell.style.transform = `translateZ(0px)`;
            return shell;
        }

        function initAvatar() {
            const mesh = document.getElementById('char-mesh'); mesh.innerHTML = "";
            const bodyParts = [
                { name: 'head', w: 50, h: 50, d: 50 }, { name: 'torso', w: 90, h: 110, d: 45 },
                { name: 'arm-l', w: 35, h: 110, d: 35 }, { name: 'arm-r', w: 35, h: 110, d: 35 },
                { name: 'leg-l', w: 40, h: 120, d: 40 }, { name: 'leg-r', w: 40, h: 120, d: 40 }
            ];
            bodyParts.forEach(part => {
                const cube = createCube(part.w, part.h, part.d, part.name);
                if(part.name === 'head') { const hairSys = document.createElement('div'); hairSys.id = "hair-system"; cube.appendChild(hairSys); }
                if(['torso', 'arm-l', 'arm-r'].includes(part.name)) cube.appendChild(createClothingLayer(part.w, part.h, part.d, 1.12, part.name + '-cloth'));
                mesh.appendChild(cube);
            });
        }

        function createClump(w, h, d, x, y, z, rx, ry) {
            const clump = document.createElement('div'); clump.className = "hair-clump";
            clump.style.width = w+"px"; clump.style.height = h+"px";
            // FIXED: Removed the internal translateZ that was causing the halo offset
            clump.style.transform = `translate3d(${x}px, ${y}px, ${z}px) rotateX(${rx}deg) rotateY(${ry}deg)`;
            const fcs = [
                {t: `translateZ(${d/2}px)`, w:w, h:h}, {t: `rotateY(180deg) translateZ(${d/2}px)`, w:w, h:h},
                {t: `rotateY(-90deg) translateZ(${w/2}px)`, w:d, h:h}, {t: `rotateY(90deg) translateZ(${w/2}px)`, w:d, h:h},
                {t: `rotateX(90deg) translateZ(${h/2}px)`, w:w, h:d}, {t: `rotateX(-90deg) translateZ(${h/2}px)`, w:w, h:d}
            ];
            fcs.forEach(f => {
                const div = document.createElement('div'); div.className="hc-face";
                div.style.width = f.w+"px"; div.style.height = f.h+"px"; div.style.transform = f.t;
                clump.appendChild(div);
            });
            return clump;
        }

        function toggleEq(t) {
            eq[t] = !eq[t];
            if(t === 'hair') {
                const sys = document.getElementById('hair-system'); sys.innerHTML = ""; sys.style.display = eq.hair ? 'block' : 'none';
                if(eq.hair) {
                    // Radius set to 20px (well within the 50px head width) to ensure clump bases are buried
                    for(let i=0; i<40; i++) {
                        const angle = (i/40) * 360;
                        const rad = 20; 
                        sys.appendChild(createClump(15, 50, 10, Math.cos(angle * Math.PI/180)*rad + 17, -5, Math.sin(angle * Math.PI/180)*rad + 17, 15, angle));
                    }
                    for(let i=0; i<15; i++) {
                        const angle = (i/15) * 360;
                        sys.appendChild(createClump(25, 12, 25, 12.5, -8, 12.5, 0, angle));
                    }
                }
            }
            if(t === 'hoodie') document.querySelectorAll('.clothing-layer').forEach(l => l.style.display = eq.hoodie ? 'block' : 'none');
            loadCat(t);
        }

        function setLab(m) { document.getElementById('lab-market').style.display = m === 'market' ? 'block' : 'none'; document.getElementById('lab-custom').style.display = m === 'custom' ? 'block' : 'none'; if(m === 'custom') loadCat('hair'); }
        function buy(id, p) { const u = DB.users[DB.cur]; if(u.inv.includes(id)) return; if(u.vib >= p) { u.vib -= p; u.inv.push(id); updateUI(); alert("Success!"); } else alert("Need VIB!"); }
        function loadCat(cat) {
            const g = document.getElementById('custom-grid'); g.innerHTML = ""; const u = DB.users[DB.cur];
            if(cat === 'hair' && u.inv.includes('hair')) g.innerHTML = `<button class="btn" onclick="toggleEq('hair')">${eq.hair ? 'REMOVE HAIR' : 'EQUIP HAIR'}</button>`;
            else if(cat === 'clothing' && u.inv.includes('hoodie')) g.innerHTML = `<button class="btn" onclick="toggleEq('hoodie')">${eq.hoodie ? 'REMOVE HOODIE' : 'EQUIP HAIR'}</button>`;
        }
        function checkout(v, p) { DB.pend = { v, p }; document.getElementById('check-desc').innerText = `AUTHORIZING R${p} FOR ${v} VIB`; showScreen('checkout-screen'); }
        function finishPay() { DB.users[DB.cur].vib += DB.pend.v; updateUI(); showScreen('vibrux-screen'); alert("Authorized!"); }

        let rot = 0; setInterval(() => { rot += 1.5; const m = document.getElementById('char-mesh'); if(m) m.style.transform = `rotateY(${rot}deg)`; }, 30);
    </script>
</body>
</html>
