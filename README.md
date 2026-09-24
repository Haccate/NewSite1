<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Hero Grid ASCII Editor — Dota 2</title>
<style>
:root{--bg:#eceae4;--pn:#f8f7f3;--tx:#1d1f22;--mu:#6b6e72;--ln:#b9b6ac;--ac:#2f6f5e;--acx:#fff;--hv:#ffe08a;--cell:#fff;--cw:18px;--ch:26px}
@media(prefers-color-scheme:dark){:root{--bg:#16181a;--pn:#202326;--tx:#e4e5e6;--mu:#8d9297;--ln:#3b4045;--ac:#5fb59c;--acx:#0f1a17;--hv:#6b5510;--cell:#181a1c}}
*{box-sizing:border-box}
body{margin:0;background:var(--bg);color:var(--tx);font:14px/1.4 system-ui,"Segoe UI",sans-serif}
.app{display:flex;gap:16px;padding:16px;align-items:flex-start;flex-wrap:wrap}
.pane{background:var(--pn);border:1px solid var(--ln);padding:14px}
.left{flex:1 1 640px;min-width:0}
.right{flex:0 1 440px;width:440px;max-width:100%}
h2{margin:0 0 10px;font-size:15px;font-weight:600}
.row{display:flex;gap:12px;flex-wrap:wrap;align-items:end;margin-bottom:10px}
label{display:flex;flex-direction:column;gap:3px;color:var(--mu);font-size:12px}
input,textarea,button,select{font:inherit;color:var(--tx);background:var(--cell);border:1px solid var(--ln);padding:6px 8px;border-radius:3px}
input[type=number]{width:90px}
input.full{width:100%}
#sym{width:54px;text-align:center;font:600 18px ui-monospace,Consolas,"DejaVu Sans Mono","Courier New",monospace}
button{cursor:pointer}
button:hover{border-color:var(--ac)}
button.on{background:var(--ac);color:var(--acx);border-color:var(--ac)}
button:focus-visible,input:focus-visible,textarea:focus-visible{outline:2px solid var(--ac);outline-offset:1px}
.seg{display:flex}.seg button{border-radius:0}.seg button:first-child{border-radius:3px 0 0 3px}.seg button:last-child{border-radius:0 3px 3px 0;border-left-width:0}
#status{font:600 14px ui-monospace,Consolas,"DejaVu Sans Mono",monospace;min-height:22px;margin-bottom:8px}
#info{color:var(--mu);font-size:12px;margin-top:8px}
.wrap{overflow:auto;max-height:78vh;border:1px solid var(--ln);padding:8px;background:var(--bg)}
#grid{display:grid;width:max-content;border-left:1px solid var(--ln);border-top:1px solid var(--ln);background:var(--cell);touch-action:none;user-select:none;cursor:crosshair;
 font:16px/1 ui-monospace,"Cascadia Mono",Consolas,"DejaVu Sans Mono","Liberation Mono","Courier New",monospace}
.c{width:var(--cw);height:var(--ch);border-right:1px solid var(--ln);border-bottom:1px solid var(--ln);display:flex;align-items:center;justify-content:center;white-space:pre;overflow:hidden}
.c.h{background:var(--hv)}
#out{width:100%;height:340px;resize:vertical;font:12.5px/1.4 ui-monospace,Consolas,"DejaVu Sans Mono",monospace;white-space:pre;overflow:auto}
.btns{display:flex;gap:8px;flex-wrap:wrap;margin-top:10px}
#toast{color:var(--ac);min-height:20px;margin-top:8px;font-size:13px}
#toast.err{color:#c0392b}
</style>
</head>
<body>
<div class="app">
 <section class="pane left">
  <h2>Сетка: 1 клетка = 1 символ category_name</h2>
  <div class="row">
   <label>Columns X <input type="number" id="cols" min="1" max="300" value="60"></label>
   <label>Rows Y <input type="number" id="rows" min="1" max="200" value="20"></label>
   <label>Symbol <input id="sym" value="%" maxlength="2" autocomplete="off" spellcheck="false"></label>
   <div class="seg" role="group" aria-label="Режим">
    <button id="bDraw" class="on">Draw</button><button id="bErase">Erase</button>
   </div>
   <button id="bFill">Fill</button>
   <button id="bClear">Clear</button>
  </div>
  <div id="status">X=– Y=– | Symbol=%</div>
  <div class="wrap"><div id="grid"></div></div>
  <div id="info"></div>
  <details id="pasteBox" open style="margin-top:12px">
   <summary style="cursor:pointer;font-weight:600;margin-bottom:8px">Вставить ASCII-картинку</summary>
   <textarea id="art" wrap="off" spellcheck="false" placeholder="Вставьте сюда готовый ASCII-арт (Ctrl+V)…" style="width:100%;height:120px;white-space:pre;overflow:auto;font:13px/1.2 ui-monospace,Consolas,'DejaVu Sans Mono',monospace"></textarea>
   <div class="btns" style="align-items:center">
    <button id="bPaste">Вставить в сетку</button>
    <label style="flex-direction:row;align-items:center;gap:6px"><input type="checkbox" id="fit" checked> Подогнать размер сетки под картинку</label>
   </div>
   <div style="color:var(--mu);font-size:12px;margin-top:6px">Ctrl+V прямо на странице (вне полей ввода) тоже вставляет картинку в сетку. Символы не меняются; табуляция заменяется одним пробелом.</div>
  </details>
 </section>

 <section class="pane right">
  <h2>hero_grid_config.json</h2>
  <div class="row" id="selRow" style="display:none">
   <label style="flex:1 1 100%">Конфиг в файле <select id="cfgSel"></select></label>
  </div>
  <div class="row">
   <label style="flex:1 1 100%">config_name <input class="full" id="cfg" value="Пользовательская сетка"></label>
   <label>x_position <input type="number" step="any" id="px" value="0"></label>
   <label>y_position <input type="number" step="any" id="py" value="0"></label>
   <label>width <input type="number" step="any" id="pw" value="300"></label>
   <label>height <input type="number" step="any" id="ph" value="100"></label>
   <label>Шаг по Y (px на строку) <input type="number" step="any" id="ys" value="20" min="0.1"></label>
   <label style="flex:1 1 100%;flex-direction:row;align-items:center;gap:6px"><input type="checkbox" id="ah" checked> Авто-высота: height = шаг по Y (иначе height задаётся вручную)</label>
   <label>px на символ <input type="number" step="any" id="cpx" value="12" min="1"></label>
   <label style="flex:1 1 100%;flex-direction:row;align-items:center;gap:6px"><input type="checkbox" id="aw" checked> Авто-ширина: width = Columns × px на символ (чтобы название влезло в одну строку)</label>
   <label style="flex:1 1 100%;flex-direction:row;align-items:center;gap:6px"><input type="checkbox" id="skip"> Пропускать пустые строки (их позиция сохраняется через y_position)</label>
   <div style="flex:1 1 100%;color:var(--mu);font-size:12px">Строка сетки с номером Y → категория с y_position = y_position + Y × шаг по Y.</div>
  </div>
  <textarea id="out" readonly spellcheck="false"></textarea>
  <div class="btns">
   <button id="bCopy">Copy JSON</button>
   <button id="bSave">Save hero_grid_config.json</button>
   <button id="bLoad">Load JSON</button>
   <input type="file" id="file" accept=".json,application/json" hidden>
  </div>
  <div id="toast" role="status"></div>
 </section>
</div>

<script>
(function(){
const $=id=>document.getElementById(id);
const CW=18,CH=26,g=$('grid');
let cols=60,rows=20,data=[],cells=[],mode='draw',drawing=false,last=null,hover=null,brush='%';

const isSym=s=>s.length===1&&s>' '&&s<='~';
const clamp=(v,a,b)=>{v=parseInt(v,10);return isNaN(v)?a:Math.min(b,Math.max(a,v))};
const num=id=>{const v=parseFloat($(id).value);return isFinite(v)?v:0};
const blank=(c,r)=>Array.from({length:r},()=>Array(c).fill(' '));

function build(){
 g.style.gridTemplateColumns=`repeat(${cols},${CW}px)`;
 g.style.gridAutoRows=CH+'px';
 g.textContent='';cells=[];hover=null;
 const f=document.createDocumentFragment();
 for(let y=0;y<rows;y++)for(let x=0;x<cols;x++){
  const d=document.createElement('div');d.className='c';d.textContent=data[y][x];
  cells.push(d);f.appendChild(d);
 }
 g.appendChild(f);update();
}
function resize(c,r){
 const nd=blank(c,r);
 for(let y=0;y<Math.min(r,rows);y++)for(let x=0;x<Math.min(c,cols);x++)nd[y][x]=data[y][x];
 data=nd;cols=c;rows=r;build();
}
function setCell(x,y,ch){
 if(data[y][x]===ch)return;
 data[y][x]=ch;cells[y*cols+x].textContent=ch;
}
function pos(e){
 const r=g.getBoundingClientRect();
 const x=Math.floor((e.clientX-r.left-g.clientLeft)/CW),y=Math.floor((e.clientY-r.top-g.clientTop)/CH);
 return x>=0&&y>=0&&x<cols&&y<rows?[x,y]:null;
}
function line(a,b,f){
 let [x0,y0]=a;const [x1,y1]=b;
 const dx=Math.abs(x1-x0),dy=-Math.abs(y1-y0),sx=x0<x1?1:-1,sy=y0<y1?1:-1;let e=dx+dy;
 for(;;){f(x0,y0);if(x0===x1&&y0===y1)break;const e2=2*e;if(e2>=dy){e+=dy;x0+=sx}if(e2<=dx){e+=dx;y0+=sy}}
}
const paintCh=()=>mode==='draw'?brush:' ';
function status(p){
 const s=mode==='draw'?brush:'[space] (Erase)';
 $('status').textContent=p?`X=${p[0]} Y=${p[1]} | Symbol=${s}`:`X=– Y=– | Symbol=${s}`;
}
function setHover(p){
 if(hover)hover.classList.remove('h');
 hover=p?cells[p[1]*cols+p[0]]:null;
 if(hover)hover.classList.add('h');
}
g.addEventListener('pointerdown',e=>{
 if(e.button!==0)return;
 e.preventDefault();g.setPointerCapture(e.pointerId);drawing=true;
 const p=pos(e);last=p;
 if(p){setCell(p[0],p[1],paintCh());update()}
});
g.addEventListener('pointermove',e=>{
 const p=pos(e);setHover(p);status(p);
 if(!drawing)return;
 if(p){
  const ch=paintCh();
  line(last||p,p,(x,y)=>setCell(x,y,ch));update();
 }
 last=p;
});
const stop=()=>{drawing=false;last=null};
g.addEventListener('pointerup',stop);
g.addEventListener('pointercancel',stop);
g.addEventListener('pointerleave',()=>{if(!drawing){setHover(null);status(null)}});

/* JSON */
const n6=v=>Number(v).toFixed(6);
function ser(c,ind){
 const p=' '.repeat(ind),q=p+'    ';
 return `${p}{\n${q}"category_name": ${JSON.stringify(String(c.category_name))},\n`+
  `${q}"x_position": ${n6(c.x_position||0)},\n${q}"y_position": ${n6(c.y_position||0)},\n`+
  `${q}"width": ${n6(c.width||0)},\n${q}"height": ${n6(c.height||0)},\n`+
  `${q}"hero_ids": ${JSON.stringify(c.hero_ids||[])}\n${p}}`;
}
let cfgs=[null],cur=0,ver=3;
function curConfig(){
 const x=num('px'),y=num('py'),w=num('pw'),st=num('ys'),h=$('ah').checked?st:num('ph'),skip=$('skip').checked,cats=[];
 data.forEach((r,i)=>{
  const t=r.join('');
  if(skip&&!t.trim())return;
  cats.push({category_name:t,x_position:x,y_position:y+i*st,width:w,height:h,hero_ids:[]});
 });
 return {config_name:$('cfg').value,categories:cats};
}
function serCfg(c){
 return `\t\t{\n    "config_name": ${JSON.stringify(String(c.config_name==null?'':c.config_name))},\n    "categories": [\n`+
  (c.categories||[]).map(k=>ser(k,8)).join(',\n')+`\n    ]\n\t\t}`;
}
function json(){
 const all=cfgs.slice();all[cur]=curConfig();
 return `{\n\t"version": ${JSON.stringify(ver)},\n\t"configs":\n\t[\n`+all.map(serCfg).join(',\n')+`\n\t]\n}`;
}
function update(){
 const aw=$('aw').checked;$('pw').disabled=aw;
 if(aw)$('pw').value=+(cols*num('cpx')).toFixed(2);
 const ah=$('ah').checked;$('ph').disabled=ah;
 if(ah)$('ph').value=num('ys');
 $('out').value=json();
 $('info').textContent=`Каждая строка сетки (Y) = отдельная категория, в ней ${cols} символов в одну линию, без переносов. Переносов строк в названиях: 0. Всего строк: ${rows}, ширина категории: ${num('pw')}.`;
}

/* controls */
function setMode(m){mode=m;$('bDraw').classList.toggle('on',m==='draw');$('bErase').classList.toggle('on',m==='erase');status(null)}
$('bDraw').onclick=()=>setMode('draw');
$('bErase').onclick=()=>setMode('erase');
$('bFill').onclick=()=>{data=blank(cols,rows).map(r=>r.map(()=>brush));build()};
$('bClear').onclick=()=>{data=blank(cols,rows);build()};
$('sym').addEventListener('input',e=>{
 const c=Array.from(e.target.value).pop()||'';
 if(isSym(c))brush=c;
 e.target.value=brush;status(null);
});
$('sym').addEventListener('focus',e=>e.target.select());
const rs=()=>{const c=clamp($('cols').value,1,300),r=clamp($('rows').value,1,200);$('cols').value=c;$('rows').value=r;if(c!==cols||r!==rows)resize(c,r)};
$('cols').onchange=rs;$('rows').onchange=rs;
['cfg','px','py','pw','ph','skip','aw','cpx','ys','ah'].forEach(id=>$(id).addEventListener('input',update));

function toast(t,err){const el=$('toast');el.textContent=t;el.className=err?'err':''}
$('bCopy').onclick=async()=>{
 const t=$('out').value;
 try{await navigator.clipboard.writeText(t)}
 catch(_){$('out').select();document.execCommand('copy');getSelection().removeAllRanges()}
 toast('JSON скопирован в буфер обмена')
};
$('bSave').onclick=()=>{
 const a=document.createElement('a');
 a.href=URL.createObjectURL(new Blob([$('out').value],{type:'application/json'}));
 a.download='hero_grid_config.json';document.body.appendChild(a);a.click();a.remove();
 setTimeout(()=>URL.revokeObjectURL(a.href),1000);toast('Сохранено: hero_grid_config.json')
};
$('bLoad').onclick=()=>$('file').click();
function parseCfgs(t){
 t=t.replace(/^\uFEFF/,'');
 try{
  const j=JSON.parse(t);
  if(j&&Array.isArray(j.configs))return {version:j.version,cfgs:j.configs};
  if(j&&Array.isArray(j.categories))return {version:j.version,cfgs:[j]};
 }catch(_){}
 const N='(-?[\\d.]+(?:[eE][+-]?\\d+)?)';
 const re=new RegExp('"category_name"\\s*:\\s*("(?:[^"\\\\]|\\\\.)*")\\s*,\\s*"x_position"\\s*:\\s*'+N+'\\s*,\\s*"y_position"\\s*:\\s*'+N+'\\s*,\\s*"width"\\s*:\\s*'+N+'\\s*,\\s*"height"\\s*:\\s*'+N,'g');
 const cats=[];let m;
 while((m=re.exec(t))){try{cats.push({category_name:JSON.parse(m[1]),x_position:+m[2],y_position:+m[3],width:+m[4],height:+m[5],hero_ids:[]})}catch(_){}}
 if(!cats.length)return null;
 const n=t.match(/"config_name"\s*:\s*("(?:[^"\\]|\\.)*")/);let name='';
 try{if(n)name=JSON.parse(n[1])}catch(_){}
 return {version:3,cfgs:[{config_name:name,categories:cats}]};
}
function loadConfig(c){
 const cs=Array.isArray(c.categories)?c.categories:[];
 let nr=rows,nc=cols,base=0,h=100,first={};
 data=null;
 if(cs.length){
  first=cs.reduce((a,b)=>(Number(b.y_position)||0)<(Number(a.y_position)||0)?b:a);
  h=Number(first.height)||0;base=Number(first.y_position)||0;
  const yv=[...new Set(cs.map(k=>Number(k.y_position)||0))].sort((a,b)=>a-b);
  let st=h;
  if(yv.length>1){st=Infinity;for(let i=1;i<yv.length;i++)st=Math.min(st,yv[i]-yv[i-1])}
  const map=new Map();
  cs.forEach((k,i)=>{
   const idx=st>0?Math.max(0,Math.round(((Number(k.y_position)||0)-base)/st)):i;
   String(k.category_name==null?'':k.category_name).split(/\r\n|\n|\r/).forEach((l,j)=>map.set(idx+j,Array.from(l)));
  });
  nr=clamp(Math.max(...map.keys())+1,1,200);
  nc=clamp(Math.max(1,...[...map.values()].map(l=>l.length)),1,300);
  data=blank(nc,nr);
  map.forEach((l,y)=>{if(y<nr)for(let x=0;x<Math.min(nc,l.length);x++)data[y][x]=l[x]});
  $('skip').checked=map.size<nr;
  $('aw').checked=false;
  $('px').value=Number(first.x_position)||0;$('py').value=base;
  $('pw').value=Number(first.width)||0;$('ph').value=h;$('ys').value=st>0?st:20;$('ah').checked=false;
 }else data=blank(nc,nr);
 cols=nc;rows=nr;$('cols').value=nc;$('rows').value=nr;
 $('cfg').value=c.config_name==null?'':c.config_name;
 build();
 return {n:cs.length,nr};
}
function fillSel(){
 const s=$('cfgSel');s.textContent='';
 cfgs.forEach((c,i)=>{const o=document.createElement('option');o.value=i;o.textContent=(i+1)+'. '+((c&&c.config_name)||'(без названия)');s.appendChild(o)});
 s.value=cur;$('selRow').style.display=cfgs.length>1?'':'none';
}
function loadText(t,name){
 const p=parseCfgs(t);
 if(!p||!p.cfgs.length)return false;
 const list=p.cfgs.filter(c=>c&&typeof c==='object');
 let i=list.findIndex(c=>Array.isArray(c.categories)&&c.categories.length);
 if(i<0)return false;
 cfgs=list;cur=i;ver=p.version==null?3:p.version;
 const r=loadConfig(cfgs[cur]);fillSel();
 toast(`Загружено${name?': '+name:''} (конфигов в файле: ${cfgs.length}, категорий: ${r.n}, строк сетки: ${r.nr})`);
 return true;
}
$('cfgSel').onchange=e=>{
 cfgs[cur]=curConfig();cur=+e.target.value;
 loadConfig(cfgs[cur]);toast('Открыт конфиг: '+(cfgs[cur].config_name||'(без названия)'));
};
$('file').onchange=async e=>{
 const f=e.target.files[0];if(!f)return;
 try{
  if(!loadText(await f.text(),f.name))throw new Error('не найдены categories с category_name');
 }catch(err){toast('Не удалось загрузить JSON: '+err.message,true)}
 e.target.value='';
};

function pasteArt(t){
 if(/"category_name"|"configs"/.test(t)&&loadText(t,'вставленный JSON'))return;
 const L=t.replace(/\t/g,' ').split(/\r\n|\n|\r/).map(l=>Array.from(l));
 while(L.length>1&&!L[L.length-1].length)L.pop();
 const ac=Math.max(1,...L.map(l=>l.length)),ar=L.length;
 if($('fit').checked){cols=clamp(ac,1,300);rows=clamp(ar,1,200);$('cols').value=cols;$('rows').value=rows}
 data=blank(cols,rows);
 for(let y=0;y<Math.min(rows,ar);y++)for(let x=0;x<Math.min(cols,L[y].length);x++)data[y][x]=L[y][x];
 build();
 toast(ac>cols||ar>rows?`Картинка ${ac}×${ar} обрезана до сетки ${cols}×${rows}`:`Вставлено: ${ac}×${ar} символов`,ac>cols||ar>rows);
}
$('bPaste').onclick=()=>{const t=$('art').value;if(t.trim())pasteArt(t);else toast('Сначала вставьте ASCII-арт в поле',true)};
document.addEventListener('paste',e=>{
 const a=document.activeElement;
 if(a&&(a.tagName==='INPUT'||a.tagName==='TEXTAREA'))return;
 const t=e.clipboardData&&e.clipboardData.getData('text');
 if(t&&t.trim()){e.preventDefault();pasteArt(t)}
});
data=blank(cols,rows);build();status(null);
})();
</script>
</body>
</html>
