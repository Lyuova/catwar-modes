// ==UserScript==
// @name         Glowline Mod
// @version      1.0
// @description  Настраиваемая подсветка игроков по статусам
// @match        https://catwar.net/cw3/*
// @match        https://catwar.su/cw3/*
// @icon         https://catwar.net/rest/site/img/M_Ti5C8YUXEN1v_vb9jc37jLeViWeDwkac9HxMUZN8kY7CE9ZB9o9F6QULzVnDJGfBPmyFPI98kiQS0Uruq_X7Tow9420w
// @grant        none
// ==/UserScript==

const ls = 'glOnlMod';
let c = JSON.parse(localStorage.getItem(ls)) || { on: 1, o: {}, c: {}, s: { 1: 1, 2: 1, 3: 1 }, op: 30, t: 'cell' };

const toH = c => {
    if (!c) return '#ffffff';
    if (c[0] === '#') return c;
    let m = c.match(/\d+/g);
    return m ? '#' + m.slice(0,3).map(x => (+x).toString(16).padStart(2, '0')).join('') : '#ffffff';
};

function save() {
    localStorage.setItem(ls, JSON.stringify(c));
    apply();
}

function ui() {
    if (!document.getElementById('gl-st')) {
        document.head.insertAdjacentHTML('beforeend', `<style id="gl-st">.gl-glow > *:not(.cat_tooltip) { filter: drop-shadow(0 0 4px var(--gl-c)) drop-shadow(0 0 2px var(--gl-c)) !important; }</style>`);
    }

    let nav = document.querySelector('.game-topbar-nav');
    if (!nav) return;

    let row = (txt, k) => `
        <div style="display:flex; justify-content:space-between; align-items:center; margin-bottom:4px;">
            <span style="cursor:pointer; font-size:14px; color:${c.s[k] ? '#888' : '#f44'};" id="gl-x${k}" title="Вкл/Выкл">✖</span>
            <span style="flex-grow:1; margin-left:6px; opacity:${c.s[k] ? 1 : 0.4};" id="gl-tx${k}">${txt}</span>
            <input type="color" id="gl-c${k}" value="${c.c[k] || toH(c.o[k])}">
        </div>`;

    let html = `
        <span style="position:relative; display:flex;">
            <a href="#" id="gl-btn" class="kti-topbar-btn" style="margin-left:5px; text-decoration:none;"><span>Glowline</span></a>
            <div id="gl-m" style="display:none; position:absolute; top:100%; left:0; width:220px; background:#2b2b2b; color:#fff; padding:10px; border-radius:6px; border:1px solid #444; z-index:10000; box-shadow:0 4px 8px rgba(0,0,0,.3); font-size:12px;">
                <label><input type="checkbox" id="gl-on" ${c.on ? 'checked' : ''}> <b>Выделять</b></label><br><br>
                ${row('"В игре"', 1)}
                ${row('"Недавно"', 2)}
                ${row('"Спит"', 3)}<br>
                Прозрачность: <span id="gl-op-v">${c.op}%</span>
                <input type="range" id="gl-op" min="0" max="100" value="${c.op}" style="width:100%; margin-bottom:10px;">
                <label><input type="radio" name="gl-t" value="cat" ${c.t === 'cat' ? 'checked' : ''}> Под игроком</label><br>
                <label><input type="radio" name="gl-t" value="cell" ${c.t === 'cell' ? 'checked' : ''}> Клетка</label>
            </div>
        </span>
    `;

    let tgt = document.querySelector('.kti-topbar-btn');
    if (tgt) tgt.insertAdjacentHTML('afterend', html);
    else nav.insertAdjacentHTML('beforeend', html);

    let m = document.getElementById('gl-m');
    document.getElementById('gl-btn').onclick = e => {
        e.preventDefault();
        m.style.display = m.style.display === 'none' ? 'block' : 'none';
    };

    m.oninput = e => {
        let i = e.target.id, v = e.target.value;
        if (i === 'gl-on') c.on = e.target.checked;
        if (i === 'gl-c1') c.c[1] = v;
        if (i === 'gl-c2') c.c[2] = v;
        if (i === 'gl-c3') c.c[3] = v;
        if (i === 'gl-op') { c.op = v; document.getElementById('gl-op-v').innerText = v + '%'; }
        if (e.target.name === 'gl-t') c.t = v;
        save();
    };

    m.onclick = e => {
        if (e.target.id.startsWith('gl-x')) {
            let k = e.target.id[4];
            c.s[k] = c.s[k] ? 0 : 1;
            e.target.style.color = c.s[k] ? '#888' : '#f44';
            document.getElementById(`gl-tx${k}`).style.opacity = c.s[k] ? 1 : 0.4;
            save();
        }
    };
}

function apply() {
    document.querySelectorAll('td.cage').forEach(t => t.style.backgroundColor = '');
    document.querySelectorAll('.cat').forEach(t => t.classList.remove('gl-glow'));
    if (!c.on) return;

    let reqSave = 0;

    document.querySelectorAll('td.cage .online span').forEach(el => {
        if (!el.dataset.o) el.dataset.o = el.style.color;
        let raw = el.dataset.o;
        if (!raw) return;

        let txt = el.textContent;
        if (txt.includes('игре') && !c.o[1]) { c.o[1] = raw; reqSave = 1; }
        if (txt.includes('Недавно') && !c.o[2]) { c.o[2] = raw; reqSave = 1; }
        if (txt.includes('Спит') && !c.o[3]) { c.o[3] = raw; reqSave = 1; }

        let k = 0;
        if (raw === c.o[1]) k = 1;
        else if (raw === c.o[2]) k = 2;
        else if (raw === c.o[3]) k = 3;

        if (k && c.s[k]) {
            let fin = c.c[k] || raw;
            let cg = el.closest('td.cage');
            let ct = cg?.querySelector('.cat');
            if (!ct) return;

            let colOp = `color-mix(in srgb, ${fin} ${c.op}%, transparent)`;
            if (c.t === 'cell') cg.style.backgroundColor = colOp;
            else {
                ct.style.setProperty('--gl-c', colOp);
                ct.classList.add('gl-glow');
            }
        }
    });

    if (reqSave) {
        localStorage.setItem(ls, JSON.stringify(c));
        [1, 2, 3].forEach(k => {
            let inp = document.getElementById(`gl-c${k}`);
            if (inp && !c.c[k] && c.o[k]) inp.value = toH(c.o[k]);
        });
    }
}

ui();
apply();

let t;
new MutationObserver(() => {
    clearTimeout(t);
    t = setTimeout(apply, 50);
}).observe(document.body, { childList: true, subtree: true, characterData: true });
