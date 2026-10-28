<!doctype html>
<html lang="en"><head><meta charset="utf-8"><meta name="viewport" content="width=device-width,initial-scale=1">
<title>DC Maker</title>
<link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Bricolage+Grotesque:opsz,wght@12..96,500;12..96,700;12..96,800&family=Figtree:wght@400;500;600;700&display=swap">
<script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
<style>
/* Layout: sticky top bar with company switcher + tabs; New DC is a two-pane workbench (form left, live A4 preview right); other tabs are single-column lists. */
:root{
  --bg:#f1f3f0; --surface:#ffffff; --surface2:#e9ede9; --ink:#15201c; --muted:#5d6b65; --line:#d5dcd7;
  --accent:#1b4f8f; --accent-ink:#ffffff; --accent-soft:#dce8f6; --danger:#c4381f; --ok:#1d7a4b;
  --display:'Bricolage Grotesque','Segoe UI',system-ui,sans-serif; --body:'Figtree','Segoe UI',system-ui,sans-serif;
}
@media (prefers-color-scheme:dark){:root:not([data-theme="light"]){
  --bg:#0e1412; --surface:#161e1b; --surface2:#1e2824; --ink:#e6ece8; --muted:#97a69f; --line:#2a3631;
  --accent:#6aaaf0; --accent-ink:#0b1420; --accent-soft:#1a2c44; --danger:#ff7a5f; --ok:#5fcf93; color-scheme:dark}}
:root[data-theme="dark"]{
  --bg:#0e1412; --surface:#161e1b; --surface2:#1e2824; --ink:#e6ece8; --muted:#97a69f; --line:#2a3631;
  --accent:#6aaaf0; --accent-ink:#0b1420; --accent-soft:#1a2c44; --danger:#ff7a5f; --ok:#5fcf93; color-scheme:dark}
*{box-sizing:border-box}
body{background:var(--bg);color:var(--ink);font:15px/1.45 var(--body);margin:0;
  padding-left:max(16px,env(safe-area-inset-left,0px));padding-right:max(16px,env(safe-area-inset-right,0px));padding-block:0 40px}
button,input,select,textarea{font:inherit;color:inherit}
.top{position:sticky;top:env(safe-area-inset-top,0px);z-index:5;background:var(--bg);padding-block:12px 0;border-bottom:1px solid var(--line);margin-inline:-4px;padding-inline:4px}
.row{display:flex;gap:12px;align-items:center;flex-wrap:wrap}
.brand{font:800 22px/1 var(--display);letter-spacing:-.02em}
.brand small{display:block;font:500 11px var(--body);letter-spacing:.12em;text-transform:uppercase;color:var(--muted);margin-top:4px}
.cos{display:flex;gap:6px;flex-wrap:wrap;margin-left:auto}
.pill{border:1px solid var(--line);background:var(--surface);border-radius:999px;padding:6px 14px;cursor:pointer;font-weight:600;font-size:13px}
.pill[aria-pressed="true"]{background:var(--accent);color:var(--accent-ink);border-color:var(--accent)}
.tabs{display:flex;gap:2px;margin-top:10px;overflow-x:auto}
.tab{border:0;background:none;padding:10px 16px;cursor:pointer;font-weight:600;color:var(--muted);border-bottom:3px solid transparent;white-space:nowrap}
.tab[aria-selected="true"]{color:var(--ink);border-bottom-color:var(--accent)}
main{max-width:1320px;margin:20px auto 0}
.bench{display:grid;grid-template-columns:minmax(0,520px) minmax(0,1fr);gap:24px;align-items:start}
@media(max-width:980px){.bench{grid-template-columns:minmax(0,1fr)}}
.card{background:var(--surface);border:1px solid var(--line);border-radius:10px;padding:18px;min-width:0}
.card h2{font:700 13px var(--body);text-transform:uppercase;letter-spacing:.1em;color:var(--muted);margin:0 0 12px}
.stack{display:flex;flex-direction:column;gap:16px;min-width:0}
.grid2{display:grid;grid-template-columns:1fr 1fr;gap:12px}
@media(max-width:480px){.grid2{grid-template-columns:1fr}}
label.f{display:flex;flex-direction:column;gap:4px;font-size:12px;font-weight:600;color:var(--muted);min-width:0}
input,select,textarea{background:var(--bg);border:1px solid var(--line);border-radius:7px;padding:9px 10px;color:var(--ink);width:100%;min-width:0}
textarea{resize:vertical;min-height:64px}
input:focus,select:focus,textarea:focus,button:focus-visible{outline:2px solid var(--accent);outline-offset:1px}
.seg{display:flex;border:1px solid var(--line);border-radius:8px;overflow:hidden}
.seg button{flex:1;border:0;background:var(--bg);padding:9px;cursor:pointer;font-weight:600;color:var(--muted)}
.seg button[aria-pressed="true"]{background:var(--accent);color:var(--accent-ink)}
.btn{border:1px solid var(--line);background:var(--surface);border-radius:8px;padding:9px 16px;font-weight:600;cursor:pointer}
.btn.pri{background:var(--accent);color:var(--accent-ink);border-color:var(--accent)}
.btn.danger{color:var(--danger)}
.btn.sm{padding:5px 10px;font-size:13px}
.btn[disabled]{opacity:.55;cursor:wait}
.items{display:flex;flex-direction:column;gap:10px}
.item{border:1px solid var(--line);border-radius:8px;padding:10px;display:grid;grid-template-columns:28px 1fr 78px 78px 104px 30px;gap:8px;align-items:start;background:var(--bg)}
.item .n{font-weight:700;color:var(--muted);padding-top:9px;text-align:center}
.item .desc{grid-column:2/-2}
.item .x{border:0;background:none;color:var(--danger);cursor:pointer;font-size:20px;padding:4px}
@media(max-width:520px){.item{grid-template-columns:24px 1fr 1fr}.item .desc{grid-column:2/-1}.item .x{grid-column:3;grid-row:1;justify-self:end}}
.sum{display:flex;justify-content:space-between;font-weight:700;padding:10px 4px 0;font-variant-numeric:tabular-nums}
.actions{display:flex;gap:10px;flex-wrap:wrap;position:sticky;bottom:0;background:var(--surface);padding-top:12px}
.pvbar{display:flex;gap:6px;margin-bottom:12px;flex-wrap:wrap;align-items:center}
.pvbar .lbl{font-size:12px;color:var(--muted);margin-left:auto}
.pvwrap{position:relative;width:100%;background:var(--surface2);border:1px solid var(--line);border-radius:10px;overflow:hidden}
.pvscale{transform-origin:0 0;position:absolute;left:0;top:0}
table.t{width:100%;border-collapse:collapse;font-variant-numeric:tabular-nums}
.tw{overflow-x:auto}
.t th{text-align:left;font-size:12px;text-transform:uppercase;letter-spacing:.08em;color:var(--muted);padding:8px 10px;border-bottom:1px solid var(--line);white-space:nowrap}
.t td{padding:10px;border-bottom:1px solid var(--line);vertical-align:top}
.t tr:hover td{background:var(--surface2)}
.t td.r,.t th.r{text-align:right}
.tag{display:inline-block;font-size:11px;font-weight:700;padding:2px 8px;border-radius:999px;background:var(--accent-soft);color:var(--accent)}
.tag.nr{background:var(--surface2);color:var(--muted)}
.muted{color:var(--muted)}
.toolbar{display:flex;gap:10px;margin-bottom:14px;flex-wrap:wrap}
.toolbar input,.toolbar select{width:auto;min-width:150px;flex:1}
.empty{padding:36px 16px;text-align:center;color:var(--muted)}
.empty b{display:block;color:var(--ink);font:700 18px var(--display);margin-bottom:6px}
.toast{position:fixed;left:50%;bottom:calc(20px + env(safe-area-inset-bottom,0px));transform:translateX(-50%);background:var(--ink);color:var(--bg);padding:10px 18px;border-radius:8px;font-weight:600;z-index:20;max-width:90vw}
.cogrid{display:grid;grid-template-columns:repeat(auto-fill,minmax(320px,1fr));gap:16px}
@media screen{#printarea{display:none}}
@media print{@page{size:A4;margin:0}html,body{background:#fff!important;margin:0!important;padding:0!important}
 body>*:not(#printarea){display:none!important}#printarea{display:block}
 #printarea .sheet{page-break-after:always;break-after:page;min-height:1120px;max-height:1122px;overflow:hidden}
 #printarea .sheet,#printarea .sheet *{-webkit-print-color-adjust:exact;print-color-adjust:exact}}
.hostoff{position:fixed;left:-9999px;top:0;width:794px;pointer-events:none}

/* ===== A4 delivery note sheet (always paper-white; same markup is previewed and exported to PDF) ===== */
.sheet{--sa:#10263f;--sl:#7f8b97;--ss:#1a232c;width:794px;min-height:1123px;background:#fff;color:#000;font:12px/1.4 'Figtree','Segoe UI',sans-serif;padding:34px 42px 28px;display:flex;flex-direction:column;position:relative}
.sheet *{box-sizing:border-box}
.sheet .stripe{position:absolute;left:0;right:0;top:0;height:10px;background:var(--cc)}
.sheet .hd{display:flex;justify-content:space-between;gap:20px;padding-top:6px}
.sheet .co{font:800 25px/1.05 'Bricolage Grotesque','Segoe UI',sans-serif;letter-spacing:-.01em;color:var(--sa);text-transform:uppercase}
.sheet .ad{margin-top:7px;color:#0b1117;font-size:12.5px;font-weight:500}
.sheet .gst{margin-top:5px;font-weight:700;font-size:11.5px;letter-spacing:.03em}
.sheet .ti{text-align:right;flex:none}
.sheet .ti h1{margin:0;font:800 23px/1 'Bricolage Grotesque','Segoe UI',sans-serif;letter-spacing:.14em;color:var(--sa)}
.sheet .ti .chip{display:inline-block;margin-top:9px;background:var(--cc);color:#fff;font-weight:700;font-size:10.5px;letter-spacing:.16em;padding:4px 11px;border-radius:3px}
.sheet .law{margin-top:12px;border-top:1.5px solid var(--sa);padding-top:6px;text-align:center;font-size:10px;font-weight:500;color:var(--ss);line-height:1.45}
.sheet .law b{color:#000;font-size:10.5px;display:block;letter-spacing:.03em}
.sheet .meta{display:grid;grid-template-columns:repeat(4,1fr);border:1px solid var(--sl);border-radius:4px;margin-top:12px}
.sheet .meta div{padding:7px 11px;border-right:1px solid var(--sl)}
.sheet .meta div:last-child{border-right:0}
.sheet .k{display:block;font-size:9.5px;font-weight:800;letter-spacing:.12em;text-transform:uppercase;color:var(--ss);margin-bottom:2px}
.sheet .v{font-weight:700;font-size:14px}
.sheet .two{display:grid;grid-template-columns:1fr 1fr;gap:12px;margin-top:12px}
.sheet .box{border:1px solid var(--sl);border-radius:4px;padding:9px 12px;min-height:84px}
.sheet .box .nm{font-weight:700;font-size:13px;margin-bottom:2px}
.sheet .box .tx{color:#0b1117;font-size:12px;font-weight:500}
.sheet .plc{display:grid;grid-template-columns:1fr 1fr;gap:12px;margin-top:12px;font-size:11.5px}
.sheet .plc div{background:#f3f5f7;border-radius:4px;padding:7px 12px}
.sheet table{width:100%;border-collapse:collapse;margin-top:14px;table-layout:fixed}
.sheet th{background:var(--sa);color:#fff;font-size:9.5px;letter-spacing:.1em;text-transform:uppercase;padding:7px 9px;text-align:left}
.sheet th.r,.sheet td.r{text-align:right}
.sheet td{padding:6px 9px;border-bottom:1px solid var(--sl);vertical-align:top;height:29px;font-variant-numeric:tabular-nums;overflow-wrap:anywhere}
.sheet td .sub{display:block;color:#0b1117;font-size:11px;font-weight:500}
.sheet td.nt{font-style:italic;font-weight:600;color:#0b1117;border-bottom-color:transparent}
.sheet tfoot td{border-top:2px solid var(--sa);border-bottom:0;font-weight:800;font-size:13px;height:auto;padding:8px 9px}
.sheet .fill{flex:1}
.sheet .dt{display:grid;grid-template-columns:1fr 1fr;gap:6px 24px;margin-top:12px;font-size:11.5px}
.sheet .dt span{color:var(--ss);font-weight:800;font-size:9.5px;letter-spacing:.1em;text-transform:uppercase;display:block}
.sheet .dt b{font-weight:700}
.sheet .sg{display:grid;grid-template-columns:1fr 1fr;gap:28px;margin-top:14px}
.sheet .sg>div{display:flex;flex-direction:column;justify-content:space-between;min-height:86px;font-size:10.5px}
.sheet .sg .l{border-top:1.2px solid #000;padding-top:5px;color:#0b1117;font-size:10px}
.sheet .sg .fo{font-weight:700;font-size:11.5px}
.sheet .sg .ver{font-size:10.5px;color:#0b1117}
.sheet .nt2{margin-top:12px;border-top:1px solid var(--sl);padding-top:6px;font-size:8.8px;line-height:1.4;color:#0b1117;font-weight:500;text-align:justify}
</style>
<script>
/* Standalone storage: everything is saved in this browser (localStorage). Use Setup > Download backup to move data between devices. */
(function(){
  const KEY='dcmaker.v1',SEED={"companies/jp": {"name": "J.P INDUSTRIES", "short": "J.P Industries", "addr": "326, VILLANKURICHI ROAD,\nPEELAMEDU, COIMBATORE - 641004.", "gstin": "33AGBPL9727C1ZM", "signatory": "J.P INDUSTRIES", "fromPlace": "PEELAMEDU, COIMBATORE - 04.", "startNo": 1, "order": 0}, "companies/vsark": {"name": "VSARK CONTROL PANELS", "short": "Vsark", "addr": "258, V.R. LAYOUT, V.K. ROAD,\nPEELAMEDU, COIMBATORE - 641004.", "gstin": "", "signatory": "VSARK CONTROL PANELS", "fromPlace": "PEELAMEDU, COIMBATORE - 04.", "startNo": 1, "order": 1}, "companies/akshara": {"name": "AKSHARA", "short": "Akshara", "addr": "", "gstin": "", "signatory": "AKSHARA", "fromPlace": "PEELAMEDU, COIMBATORE - 04.", "startNo": 1, "order": 2}, "config/app": {"preparers": ["KRISHNA", "POONGODI", "BALA CHANDAR"], "purpose": "POWDER COATING / JOB WORK", "remarks": "NOT FOR SALE"}, "parties/p01": {"name": "V.M.POWDER COATING", "address": "337,KARUPPANNA GOUNDER LAY-OUT,\nVILANKURICHI ROAD, PEELAMEDU,CBE-04.\nGSTIN NO:33DDAPM0060C1ZD"}, "parties/p02": {"name": "MAYA TECH", "address": "73/1, LGB NAGAR 2ND STREET, SARAVANANMPATTI, COIMBATORE\nGSTIN : 33BT0PP7305K1ZA"}, "parties/p03": {"name": "Emerald Industries", "address": "3, Nethaji Nagar, Palanikavundan Pudur, K.Vadamadurai, Coimbatore - 641 017\nGSTIN/UIN: 33CWPPS3162J1Z2"}, "parties/p04": {"name": "J C ELECTRIC", "address": "NO.18, NEAR LAKSHMI HOSPITAL, METTUPALAYAM ROAD, COIMBATORE\nGSTIN NO : 33CWPPS3162J2Z1"}, "parties/p05": {"name": "AQUASUB ENGINEERING", "address": "THUDIYALUR POST, MEETUPALAYAM ROAD, COIMBATORE\nGSTIN/UIN NO:33AADFA8028P1Z9"}, "parties/p06": {"name": "SELVAM PRESS TOOLS", "address": "20,PERUMAL KOVIL STREET, K.R PURAM ROAD, GANAPATHY, COIMBATORE -641006\nGSTIN NO:33BJWPP9052M1ZA"}, "parties/p07": {"name": "Sri Balamurugan Surface Coatings", "address": "No.7/10-D Na Sri Ayyappa Industrial Estate, Keeranatham Village, Kondayampalayam Road, Saravanampatti Post, Coimbatore - 641035\nGSTIN: 33AATPU7828H1ZZ"}, "parties/p08": {"name": "GENERAL ELECTROPLATING WORKS", "address": "48-B, SSF CAMPUS, NEAR SNR SNS COLLEGE, NAVA INDIA ROAD, AVARAMPALAYAM, PEELAMEDU, COIMBATORE."}, "parties/p09": {"name": "POWERTECH CALIBRATION SERVICES", "address": "2/15, CHERAN MANAGER, VILANKURICHI MAIN ROAD, PEELAMEDU, CBE-035."}, "parties/p10": {"name": "LAKSHMI ENGINEEING INDUSTRIES", "address": "84/12, ATHIPALAYAM ROAD, CHINNAVEDAMPATTI, CBE-033.\nGSTIN NO:33AABFL9585C1ZB"}, "parties/p11": {"name": "M P V INDUSTRIES", "address": "13/2-C, D.REDDIYAPATTI, CHINNAMPATTI, Mavathur, Karur, Tamil Nadu, 621311\nGSTIN/UIN : 33CCQPV8385M1ZE"}, "parties/p12": {"name": "BALU", "address": "HOPE COLLEGE,"}, "parties/p13": {"name": "DECCAN PUMPS PRIVATE LIMITED", "address": "43, VILANKURICHI MAIN ROAD, PEELAMEDU, CBE-035."}, "parties/p14": {"name": "EKKI PUMPS", "address": "67-A-10, ATHIPALAYAM ROAD, VENKITASAMY NAGAR, GANAPATHY, COIMBATORE.\nGSTIN: 33AAEFE2643P1ZB"}, "parties/p15": {"name": "M K S Loom Modernizer", "address": "5/218-6 Konguma Nagar, Kalipalayam Privu, Karuvalur Avinashi, Tiruppur - 641670\nGSTIN : 33CDXPK5223R2ZQ"}, "parties/p16": {"name": "PRABHU INDUSTRIES", "address": "B-6 SASTHA NAGAR, OPP.INDIAN OIL PETROL BUNK, FCI ROAD, PEELAMEDU, CBE-04.\nGSTIN NO:33ADTPG1966F1ZG"}, "parties/p17": {"name": "NEWTECH FABRICATION", "address": "163/1, AMMAN KOVIL THOTTAM, GANAPATHY, COIMBATORE-641 006.\nGSTIN NO:33BDLPS3113F1ZS"}, "parties/p18": {"name": "BLN INDUSTRIES", "address": "37, VENKATALAKSHMI NAGER, ASHOK LAY LAND SHOW ROOM BACK SIDE, SINGANALLUR, COIMBATORE -641005\nGSTIN NO:33AISPM9163E1ZX"}, "parties/p19": {"name": "Vijay and Co", "address": "126, Masakkalipalayam Road, Sowripalayam, Coimbatore, Tamil Nadu\nGSTIN: 33ALEPK1096K1Z2"}, "parties/p20": {"name": "JEEVA DHARANI INDUSTRIES", "address": "5/21, ELLAPPAN ROAD, PEELAMEDU, COIMBATORE - 641004"}, "parties/p21": {"name": "Renuka Industries", "address": "7,8 R.K. Nagar, Ingur Road, Irugur, Nursery School, Coimbatore - 641103\nGSTIN : 33ABIFR1055P1ZW"}, "parties/p22": {"name": "RR POWDER COATINGS", "address": "NO.46, V.K.ROAD, THANNERPANDAL, PEELAMEDU INDUSTRIAL ESTATE, PEELAMEDU, COIMBATORE -641004\nGSTIN NO:33AAYFR5920Q1ZA"}, "parties/p23": {"name": "THAAI COATS", "address": "18/2, K.K.NAGAR, BHARTHI NAGAR, GANAPATHY, COIMBATORE-641006\nGSTIN NO:33APCPM7955F1ZT"}, "parties/p24": {"name": "K.R.SYSTEMS", "address": "132/1, KALEESWARAR STREET, RAMNAGAR, KATTOOR, COIMBATORE\nGSTIN NO: 33AHZPM5859P1Z3"}, "parties/p25": {"name": "DEEPAK COATING", "address": "COIMBATORE"}, "parties/p26": {"name": "NOAH ENGINEERING", "address": "1/237, D-16, ARINGNAR ANNA COLONY, POLLACHI MAIN ROAD, SIDCO(PO), COIMBATORE - 641021\nGSTIN : 33AZNPS3423M1ZY"}, "parties/p27": {"name": "AVM INDUSTRIAL SAFETY TRADERS", "address": "NO. 42-5, 1ST FLOOR, VENKATASAMY LAYOUT, KALLAPPA LANE, KATTOR, COIMBATORE.\nGSTIN/UIN NO: 33BYZPB9224P1ZN"}};
  let data;try{data=JSON.parse(localStorage.getItem(KEY)||'null')}catch(e){}
  if(!data){data=SEED;try{localStorage.setItem(KEY,JSON.stringify(data))}catch(e){}}
  const subs=new Set();
  const persist=()=>{try{localStorage.setItem(KEY,JSON.stringify(data))}catch(e){}subs.forEach(f=>f())};
  const meta={fromCache:false,hasPendingWrites:false};
  const colDocs=c=>Object.keys(data).filter(p=>p.startsWith(c+'/')&&p.split('/').length===2).map(p=>({id:p.split('/')[1],exists:true,data:()=>JSON.parse(JSON.stringify(data[p])),metadata:meta}));
  const db={
    doc:p=>({id:p.split('/').pop(),path:p,
      async set(d){data[p]=JSON.parse(JSON.stringify(d));persist()},
      async delete(){delete data[p];persist()},
      onSnapshot(next){const f=()=>next({id:p.split('/').pop(),exists:p in data,data:()=>p in data?JSON.parse(JSON.stringify(data[p])):undefined,metadata:meta});subs.add(f);setTimeout(f,0);return()=>subs.delete(f)}}),
    collection:c=>({onSnapshot(next){const f=()=>{const docs=colDocs(c);next({docs,size:docs.length,empty:!docs.length,docChanges:()=>[],metadata:meta})};subs.add(f);setTimeout(f,0);return()=>subs.delete(f)}})
  };
  const downloads={async save({filename,data:d}){const b=d instanceof Blob?d:new Blob([d],{type:/\.csv$/.test(filename)?'text/csv':/\.json$/.test(filename)?'application/json':'application/octet-stream'});
    const a=document.createElement('a');a.href=URL.createObjectURL(b);a.download=filename;document.body.appendChild(a);a.click();setTimeout(()=>{URL.revokeObjectURL(a.href);a.remove()},1500);return{status:'saved'}}};
  window.claude={use:async n=>n==='db'?db:n==='downloads'?downloads:null};
})();
</script>
</head><body>
<div class="top">
  <div class="row">
    <div class="brand">DC Maker<small>Delivery Note Form JJ</small></div>
    <div class="cos" id="cos"></div>
  </div>
  <div class="tabs" role="tablist" id="tabs"></div>
</div>
<main id="main"><div class="empty"><b>Loading…</b>Connecting to your saved data.</div></main>
<div class="hostoff" id="host" aria-hidden="true"></div>
<div id="printarea"></div>
<div id="toast"></div>

<script>
const $=s=>document.querySelector(s);
const esc=s=>String(s??'').replace(/[&<>"]/g,c=>({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;'}[c]));
const nl=s=>esc(s).replace(/\n/g,'<br>');
const inr=n=>Number(n||0).toLocaleString('en-IN');
const today=()=>{const d=new Date();return d.getFullYear()+'-'+String(d.getMonth()+1).padStart(2,'0')+'-'+String(d.getDate()).padStart(2,'0')};
const fmtDate=s=>{if(!s)return '';const[y,m,d]=s.split('-');return d+'.'+m+'.'+y};
const fyStart=s=>{const[y,m]=s.split('-').map(Number);return m>=4?y:y-1};
const fyKey=s=>{const a=fyStart(s);return a+'-'+String(a+1).slice(2)};
const COPIES=[{n:'ORIGINAL',c:'#6b7784'},{n:'DUPLICATE',c:'#2f6fe0'},{n:'TRIPLICATE',c:'#d93c3c'},{n:'QUADRUPLICATE',c:'#e0a100'}];
const UNITS=['Nos','Kgs','Sets','Mtrs','Ltrs'];
const NOTE='NOTE: 1. Original to be furnished to the officer in-charge of the first check post or barrier, the second copy to such officer in the last check post or barrier and the third copy shall be retained by the owner or other person in charge of the goods vehicle or boat. 2. The entries in this form should be made in ink or by copying pencil using double side carbon paper. In the case of delivery notes issued in respect of goods intended to be transferred to a place outside the state of Tamil Nadu entry should be made in English. In other cases of transport such entries may be made either in English or in official regional language of the state, namely, Tamil. 3. Any corrections made in the form should be duly attested by the person signing the form. 4. The original, duplicate, triplicate and quadruplicate copies are to be printed in white, blue, red and yellow colours respectively for easy identification.';

const S={db:null,loaded:{c:0,p:0,d:0,a:0},companies:[],parties:[],dcs:[],app:{preparers:['KRISHNA'],purpose:'POWDER COATING / JOB WORK',remarks:'NOT FOR SALE'},
  view:'new',copies:[1,1,0,0],companyId:null,draft:null,copy:0,regQ:'',regCo:'all',regFy:'all',partyQ:'',editParty:null,delId:null,busy:false};
try{const c=JSON.parse(localStorage.getItem('dc.copies')||'null');if(Array.isArray(c)&&c.length===4&&c.some(Boolean))S.copies=c}catch(e){}
try{S.view=localStorage.getItem('dc.view')||'new';S.companyId=localStorage.getItem('dc.co')}catch(e){}

function toast(m){const t=$('#toast');t.innerHTML='<div class="toast">'+esc(m)+'</div>';clearTimeout(toast.t);toast.t=setTimeout(()=>t.innerHTML='',3200)}
const selIdx=()=>S.copies.map((v,i)=>v?i:-1).filter(i=>i>=0);
const cpTxt=()=>{const n=selIdx().length;return n+(n>1?' copies':' copy')};
const co=id=>S.companies.find(c=>c.id===id);
const curCo=()=>co(S.companyId)||S.companies[0];

function totals(items){const q={};let v=0;for(const i of items){const n=parseFloat(i.q);if(n){q[i.u||'Nos']=(q[i.u||'Nos']||0)+n}v+=parseFloat(i.v)||0}return{q,v}}
const qtyText=q=>Object.entries(q).map(([u,n])=>inr(n)+' '+u).join(' + ')||'—';

function nextNo(cid,date){const fy=fyKey(date);const nums=S.dcs.filter(d=>d.company===cid&&d.fy===fy).map(d=>+d.no);const c=co(cid);return nums.length?Math.max(...nums)+1:(+(c&&c.startNo)||1)}
function newDraft(){const c=curCo();const date=today();return{id:null,company:c?c.id:'',no:c?nextNo(c.id,date):1,date,returnable:true,partyName:'',partyAddr:'',fromPlace:c?.fromPlace||'PEELAMEDU, COIMBATORE - 04.',toPlace:'COIMBATORE',
  items:[{d:'',x:'',q:'',u:'Nos',v:''}],notes:'',purpose:S.app.purpose||'',vehicle:'',remarks:S.app.remarks||'',preparedBy:(S.app.preparers||[])[0]||''}}

/* ---------- sheet ---------- */
function sheet(dc,ci){
  const c=co(dc.company)||{name:'',addr:'',gstin:'',signatory:''};const cp=COPIES[ci];const t=totals(dc.items);
  const rows=dc.items.filter(i=>i.d||i.q||i.v);
  const need=Math.max(0,9-rows.length-(dc.notes?dc.notes.split('\n').filter(Boolean).length:0));
  const body=rows.map((i,n)=>`<tr><td>${n+1}</td><td><b>${esc(i.d)}</b>${i.x?`<span class="sub">${esc(i.x)}</span>`:''}</td><td class="r">${i.q?esc(i.q)+' '+esc(i.u):''}</td><td class="r">${i.v?inr(i.v):''}</td></tr>`).join('')
   +(dc.notes||'').split('\n').filter(Boolean).map(n=>`<tr><td></td><td class="nt" colspan="3">${esc(n)}</td></tr>`).join('')
   +Array.from({length:need},()=>'<tr><td></td><td></td><td></td><td></td></tr>').join('');
  return `<div class="sheet" style="--cc:${cp.c}"><div class="stripe"></div>
  <div class="hd"><div><div class="co">${esc(c.name)}</div><div class="ad">${nl(c.addr)}</div>${c.gstin?`<div class="gst">GSTIN/UIN: ${esc(c.gstin)}</div>`:''}</div>
  <div class="ti"><h1>DELIVERY NOTE</h1><span class="chip">${cp.n}</span></div></div>
  <div class="law"><b>(For Sales / Stock Transfer / Works Contract / Labour)</b>[See rule 15(3), 15(18), 15(19), 15(20), 15(21)]<br>(For movement of inputs of partially processed goods under rule 57F(3) and for under notification No.214/86 dated 25.03.86 from one factory to another for further processing / operation and subsequent return the parent factory)</div>
  <div class="meta"><div><span class="k">DC No.</span><span class="v">${esc(dc.no)}</span></div><div><span class="k">Date</span><span class="v">${fmtDate(dc.date)}</span></div><div><span class="k">Asst. Year</span><span class="v">${dc.date?fyStart(dc.date)+' - '+(fyStart(dc.date)+1):''}</span></div><div><span class="k">Type</span><span class="v">${dc.returnable?'Returnable':'Non-Returnable'}</span></div></div>
  <div class="two"><div class="box"><span class="k">1. Name &amp; address of the consignor</span><div class="nm">${esc(c.name)}</div><div class="tx">${nl(c.addr)}${c.gstin?'<br>GSTIN/UIN: '+esc(c.gstin):''}</div></div>
  <div class="box"><span class="k">2. Name &amp; address of the consignee</span><div class="nm">${esc(dc.partyName)}</div><div class="tx">${nl(dc.partyAddr)}</div></div></div>
  <div class="plc"><div><span class="k">3. (i) Goods consigned from</span><b>${esc(dc.fromPlace)}</b></div><div><span class="k">(ii) Goods consigned to</span><b>${esc(dc.toPlace)}</b></div></div>
  <table><colgroup><col style="width:9%"><col style="width:55%"><col style="width:17%"><col style="width:19%"></colgroup>
  <thead><tr><th>Sl. No.</th><th>4. Particulars</th><th class="r">Qty (Nos. / Kgs.)</th><th class="r">Appr. Value (Rs.)</th></tr></thead><tbody>${body}</tbody>
  <tfoot><tr><td></td><td>TOTAL</td><td class="r">${qtyText(t.q)}</td><td class="r">${t.v?'₹ '+inr(t.v):''}</td></tr></tfoot></table>
  <div class="fill"></div>
  <div class="dt"><div><span>5. Purpose of transport</span><b>${esc(dc.purpose)||'&nbsp;'}</b></div><div><span>6. Delivered for transport &amp; vehicle no.</span><b>${esc(dc.vehicle)||'&nbsp;'}</b></div>
  <div><span>7. Remarks, if any</span><b>${esc(dc.remarks)||'&nbsp;'}</b></div><div><span>8. Prepared by</span><b>${esc(dc.preparedBy)||'&nbsp;'}</b></div></div>
  <div class="sg"><div><div>Received the above goods in order &amp; condition</div><div class="l">Name and signature of the person to whom the goods were delivered</div></div>
  <div><div><div class="fo">For ${esc(c.signatory||c.name)}</div><div class="ver">I / We verify that to the best of my / our knowledge the particulars furnished are true, correct and complete.</div></div><div class="l">Name and signature of the consignor / his employee / his representative</div></div></div>
  <div class="nt2">${esc(NOTE)}</div></div>`}

/* ---------- shell ---------- */
function renderTop(){
  $('#cos').innerHTML=S.companies.map(c=>`<button class="pill" data-act="co" data-id="${esc(c.id)}" aria-pressed="${c.id===(curCo()||{}).id}">${esc(c.short||c.name)}</button>`).join('');
  const T=[['new','New DC'],['register','Register'],['parties','Parties'],['setup','Setup']];
  $('#tabs').innerHTML=T.map(([k,l])=>`<button class="tab" role="tab" data-act="tab" data-id="${k}" aria-selected="${S.view===k}">${l}</button>`).join('');
}
function render(){
  if(!S.db){$('#main').innerHTML='<div class="empty"><b>Storage is blocked</b>Allow site data for this page in your browser settings and reload.</div>';return}
  if(S.loaded.c<1||S.loaded.p<1||S.loaded.d<1){$('#main').innerHTML='<div class="empty"><b>Loading…</b>Connecting to your saved data.</div>';return}
  renderTop();
  ({new:rNew,register:rReg,parties:rParties,setup:rSetup})[S.view]();
}

/* ---------- New DC ---------- */
function itemRow(i,n){return `<div class="item"><div class="n">${n+1}</div>
 <label class="f desc">Item<input data-i="${n}" data-k="d" value="${esc(i.d)}" placeholder="e.g. SHEET METAL ENCLOSURE"></label>
 <span></span>
 <label class="f desc">Size / detail (optional)<input data-i="${n}" data-k="x" value="${esc(i.x)}" placeholder="e.g. 205 X 180 X 90 - BINDHU CON"></label>
 <label class="f">Qty<input data-i="${n}" data-k="q" inputmode="decimal" value="${esc(i.q)}"></label>
 <label class="f">Unit<select data-i="${n}" data-k="u">${UNITS.map(u=>`<option${u===i.u?' selected':''}>${u}</option>`).join('')}</select></label>
 <label class="f">Value ₹<input data-i="${n}" data-k="v" inputmode="decimal" value="${esc(i.v)}"></label>
 <button class="x" data-act="delitem" data-id="${n}" aria-label="Remove item ${n+1}">×</button></div>`}
function rNew(){
  if(!S.companies.length){$('#main').innerHTML='<div class="empty"><b>Add your company first</b>Open Setup and enter your company name, address and GSTIN.</div>';return}
  if(!S.draft)S.draft=newDraft();const d=S.draft;
  $('#main').innerHTML=`<div class="bench"><div class="stack">
  <div class="card"><h2>${d.id?'Editing DC '+esc(d.no):'Delivery note details'}</h2>
   <div class="grid2"><label class="f">DC No.<input data-f="no" inputmode="numeric" value="${esc(d.no)}"></label><label class="f">Date<input type="date" data-f="date" value="${esc(d.date)}"></label></div>
   <div class="seg" style="margin-top:12px"><button data-act="ret" data-id="1" aria-pressed="${d.returnable}">Returnable</button><button data-act="ret" data-id="0" aria-pressed="${!d.returnable}">Non-Returnable</button></div></div>
  <div class="card"><h2>Consignee (party)</h2>
   <label class="f">Party name<input data-f="partyName" list="plist" value="${esc(d.partyName)}" placeholder="Type or pick a party" autocomplete="off"></label>
   <datalist id="plist">${S.parties.map(p=>`<option value="${esc(p.name)}">`).join('')}</datalist>
   <label class="f" style="margin-top:12px">Address &amp; GSTIN<textarea data-f="partyAddr" id="paddr">${esc(d.partyAddr)}</textarea></label>
   <div class="grid2" style="margin-top:12px"><label class="f">Goods from<input data-f="fromPlace" value="${esc(d.fromPlace)}"></label><label class="f">Goods to<input data-f="toPlace" value="${esc(d.toPlace)}"></label></div></div>
  <div class="card"><h2>Goods</h2><div class="items" id="items">${d.items.map(itemRow).join('')}</div>
   <div class="row" style="margin-top:12px"><button class="btn sm" data-act="additem">+ Add item</button></div>
   <div class="sum"><span>Total</span><span id="tot"></span></div>
   <label class="f" style="margin-top:12px">Lines below the items (e.g. GST 18% EXTRA)<textarea data-f="notes" placeholder="One line per row">${esc(d.notes)}</textarea></label></div>
  <div class="card"><h2>Transport</h2>
   <div class="grid2"><label class="f">Purpose<input data-f="purpose" value="${esc(d.purpose)}"></label><label class="f">Vehicle no.<input data-f="vehicle" value="${esc(d.vehicle)}" placeholder="TN37CQ9645"></label>
   <label class="f">Remarks<input data-f="remarks" value="${esc(d.remarks)}"></label>
   <label class="f">Prepared by<input data-f="preparedBy" list="prlist" value="${esc(d.preparedBy)}"></label></div>
   <datalist id="prlist">${(S.app.preparers||[]).map(p=>`<option value="${esc(p)}">`).join('')}</datalist></div>
  <div class="card"><h2>Copies to print</h2><div class="row" style="gap:8px">${COPIES.map((c,i)=>`<button class="pill" data-act="cp" data-id="${i}" aria-pressed="${!!S.copies[i]}">${c.n[0]+c.n.slice(1).toLowerCase()}${i>1?' (extra)':''}</button>`).join('')}</div><p class="muted" style="margin:10px 0 0;font-size:13px">Original and Duplicate are included by default. Tap Triplicate or Quadruplicate to add an extra copy.</p></div>
  <div class="card actions"><button class="btn pri" data-act="save">${d.id?'Update DC':'Save DC'}</button><button class="btn pri" data-act="printdraft">Save &amp; print (${cpTxt()})</button><button class="btn" data-act="pdfdraft" ${S.busy?'disabled':''}>${S.busy?'Preparing PDF…':`Save & download PDF (${cpTxt()})`}</button><button class="btn" data-act="newdc">${d.id?'Cancel edit':'Clear'}</button></div>
  </div>
  <div style="min-width:0;position:sticky;top:calc(120px + env(safe-area-inset-top,0px))"><div class="pvbar">${COPIES.map((c,i)=>`<button class="pill" data-act="copy" data-id="${i}" aria-pressed="${S.copy===i}">${c.n[0]+c.n.slice(1).toLowerCase()}</button>`).join('')}<span class="lbl">A4 preview</span></div>
   <div class="pvwrap" id="pvwrap"><div class="pvscale" id="pv"></div></div></div></div>`;
  updPreview();
}
function updPreview(){
  const d=S.draft,pv=$('#pv');if(!pv)return;pv.innerHTML=sheet(d,S.copy);
  const w=$('#pvwrap'),s=w.clientWidth/794;pv.style.transform=`scale(${s})`;w.style.height=(pv.firstElementChild.offsetHeight*s)+'px';
  const t=totals(d.items);const e=$('#tot');if(e)e.textContent=qtyText(t.q)+'   ·   ₹ '+inr(t.v);
}
new ResizeObserver(()=>{if(S.view==='new'&&$('#pv')&&S.draft)updPreview()}).observe(document.body);

async function saveDraft(silent){
  const d=S.draft;
  if(!d.no||!/^\d+$/.test(String(d.no))){toast('Enter the DC number as a whole number.');return null}
  if(!d.date){toast('Choose the DC date.');return null}
  if(!d.partyName.trim()){toast('Enter the party name.');return null}
  const fy=fyKey(d.date),id=`${d.company}_${fy}_${d.no}`;
  if(id!==d.id&&S.dcs.some(x=>x.id===id)){toast(`DC ${d.no} already exists for ${fy}. Pick another number.`);return null}
  const rec={company:d.company,no:+d.no,fy,date:d.date,returnable:d.returnable,partyName:d.partyName.trim(),partyAddr:d.partyAddr,fromPlace:d.fromPlace,toPlace:d.toPlace,
    items:d.items.filter(i=>i.d||i.q||i.v),notes:d.notes,purpose:d.purpose,vehicle:d.vehicle,remarks:d.remarks,preparedBy:d.preparedBy,updatedAt:Date.now()};
  if(!rec.items.length){toast('Add at least one item.');return null}
  try{
    if(d.id&&d.id!==id)await S.db.doc('dcs/'+d.id).delete();
    await S.db.doc('dcs/'+id).set(rec);
    if(!S.parties.some(p=>p.name.toLowerCase()===rec.partyName.toLowerCase())){await S.db.doc('parties/p'+Date.now().toString(36)).set({name:rec.partyName,address:rec.partyAddr||''})}
    const known=(S.app.preparers||[]);if(d.preparedBy&&!known.includes(d.preparedBy)){S.app.preparers=[...known,d.preparedBy];await S.db.doc('config/app').set(S.app)}
  }catch(e){toast('Could not save: '+(e.message||e.code));return null}
  d.id=id;if(!silent)toast('DC '+d.no+' saved');return {...rec,id};
}

/* ---------- PDF ---------- */
async function makePdf(dc){
  await document.fonts.ready;const{jsPDF}=window.jspdf;const pdf=new jsPDF({unit:'mm',format:'a4',orientation:'p'});const host=$('#host');
  for(const [k,i] of selIdx().entries()){
    host.innerHTML=sheet(dc,i);const el=host.firstElementChild;
    const cv=await html2canvas(el,{scale:2.5,backgroundColor:'#ffffff',useCORS:true,logging:false});
    let w=210,h=210*cv.height/cv.width;if(h>297){w=w*297/h;h=297}
    if(k)pdf.addPage();pdf.addImage(cv.toDataURL('image/jpeg',.93),'JPEG',(210-w)/2,0,w,h);
  }
  host.innerHTML='';return pdf.output('arraybuffer');
}
async function savePdf(dc){
  S.busy=true;render();
  try{
    const dl=await claude.use('downloads');if(!dl){toast('Downloads are not available in this view.');return}
    const buf=await makePdf(dc);const c=co(dc.company);
    await dl.save({filename:`DC-${dc.no}_${(dc.partyName||'party').replace(/[^\w]+/g,'_').slice(0,24)}_${(c.short||c.name).replace(/\W+/g,'')}.pdf`,data:buf});
    toast('PDF downloaded. Open it and print on A4.');
  }catch(e){if(e&&e.code!=='declined')toast('PDF failed: '+(e.message||e.code))}
  finally{S.busy=false;render()}
}

function doPrint(dc){
  const pa=$('#printarea');pa.innerHTML=selIdx().map(i=>sheet(dc,i)).join('');
  document.fonts.ready.then(()=>setTimeout(()=>{window.print();setTimeout(()=>pa.innerHTML='',500)},150));
}
/* ---------- Register ---------- */
function rReg(){
  const fys=[...new Set(S.dcs.map(d=>d.fy))].sort().reverse();
  let list=S.dcs.filter(d=>(S.regCo==='all'||d.company===S.regCo)&&(S.regFy==='all'||d.fy===S.regFy));
  const q=S.regQ.toLowerCase();if(q)list=list.filter(d=>(d.partyName+' '+d.no+' '+d.items.map(i=>i.d+' '+i.x).join(' ')).toLowerCase().includes(q));
  list.sort((a,b)=>b.date.localeCompare(a.date)||b.no-a.no);
  const rows=list.map(d=>{const t=totals(d.items),c=co(d.company);return `<tr><td><b>${d.no}</b></td><td>${fmtDate(d.date)}</td><td>${esc(c?(c.short||c.name):'')}</td><td>${esc(d.partyName)}</td>
   <td class="muted">${esc(d.items.map(i=>i.d).join(', ')).slice(0,70)}</td><td class="r">${esc(qtyText(t.q))}</td><td class="r">${t.v?'₹ '+inr(t.v):''}</td><td><span class="tag ${d.returnable?'':'nr'}">${d.returnable?'R':'NR'}</span></td><td>${esc(d.preparedBy)}</td>
   <td style="white-space:nowrap">${S.delId===d.id?`<button class="btn sm danger" data-act="delyes" data-id="${esc(d.id)}">Delete</button> <button class="btn sm" data-act="delno">Keep</button>`:
   `<button class="btn sm" data-act="edit" data-id="${esc(d.id)}">Edit</button> <button class="btn sm" data-act="print" data-id="${esc(d.id)}">Print</button> <button class="btn sm" data-act="pdf" data-id="${esc(d.id)}">PDF</button> <button class="btn sm" data-act="del" data-id="${esc(d.id)}" aria-label="Delete DC ${d.no}">✕</button>`}</td></tr>`}).join('');
  $('#main').innerHTML=`<div class="card"><h2>DC register</h2><div class="toolbar"><input id="rq" placeholder="Search party, DC no. or item" value="${esc(S.regQ)}">
   <select id="rco"><option value="all">All companies</option>${S.companies.map(c=>`<option value="${esc(c.id)}"${S.regCo===c.id?' selected':''}>${esc(c.name)}</option>`).join('')}</select>
   <select id="rfy"><option value="all">All years</option>${fys.map(f=>`<option${S.regFy===f?' selected':''}>${f}</option>`).join('')}</select>
   <button class="btn" data-act="csv">Export CSV</button></div>
   ${list.length?`<div class="tw"><table class="t"><thead><tr><th>DC</th><th>Date</th><th>Company</th><th>Party</th><th>Items</th><th class="r">Qty</th><th class="r">Value</th><th>R/NR</th><th>By</th><th></th></tr></thead><tbody>${rows}</tbody></table></div>`:
   `<div class="empty"><b>${S.dcs.length?'No DC matches':'No delivery notes yet'}</b>${S.dcs.length?'Try a different search or year.':'Saved delivery notes appear here. Create your first one in New DC.'}</div>`}</div>`;
}

/* ---------- Parties ---------- */
function rParties(){
  const q=S.partyQ.toLowerCase();const list=[...S.parties].filter(p=>(p.name+p.address).toLowerCase().includes(q)).sort((a,b)=>a.name.localeCompare(b.name));const e=S.editParty;
  $('#main').innerHTML=`<div class="bench" style="grid-template-columns:minmax(0,420px) minmax(0,1fr)"><div class="card" style="position:sticky;top:calc(120px + env(safe-area-inset-top,0px))"><h2>${e&&e.id?'Edit party':'Add party'}</h2>
   <label class="f">Party name<input id="pn" value="${esc(e?.name||'')}"></label><label class="f" style="margin-top:12px">Address &amp; GSTIN<textarea id="pa" style="min-height:110px">${esc(e?.address||'')}</textarea></label>
   <div class="row" style="margin-top:14px"><button class="btn pri" data-act="saveparty">${e&&e.id?'Update':'Add party'}</button>${e?'<button class="btn" data-act="cancelparty">Cancel</button>':''}</div></div>
   <div class="card"><h2>${S.parties.length} parties</h2><div class="toolbar"><input id="pq" placeholder="Search parties" value="${esc(S.partyQ)}"></div>
   ${list.length?`<div class="tw"><table class="t"><tbody>${list.map(p=>`<tr><td><b>${esc(p.name)}</b><div class="muted" style="font-size:13px">${esc(p.address)}</div></td><td style="white-space:nowrap;text-align:right">
   ${S.delId===p.id?`<button class="btn sm danger" data-act="delpyes" data-id="${p.id}">Delete</button> <button class="btn sm" data-act="delno">Keep</button>`:`<button class="btn sm" data-act="editparty" data-id="${p.id}">Edit</button> <button class="btn sm" data-act="delp" data-id="${p.id}" aria-label="Delete ${esc(p.name)}">✕</button>`}</td></tr>`).join('')}</tbody></table></div>`:'<div class="empty"><b>No parties found</b>Add a party on the left.</div>'}</div></div>`;
}

/* ---------- Setup ---------- */
function coCard(c){return `<div class="card" data-co="${esc(c.id)}"><h2>${esc(c.short||c.name||'New company')}</h2><div class="stack" style="gap:12px">
 <label class="f">Company name (printed on DC)<input data-c="name" value="${esc(c.name)}"></label>
 <label class="f">Short name (button)<input data-c="short" value="${esc(c.short||'')}"></label>
 <label class="f">Address (one line per row)<textarea data-c="addr">${esc(c.addr||'')}</textarea></label>
 <label class="f">GSTIN / UIN<input data-c="gstin" value="${esc(c.gstin||'')}"></label>
 <label class="f">Signatory line (“For …”)<input data-c="signatory" value="${esc(c.signatory||'')}" placeholder="${esc(c.name)}"></label>
 <div class="grid2"><label class="f">Default “goods from”<input data-c="fromPlace" value="${esc(c.fromPlace||'')}"></label><label class="f">First DC no. of a new year<input data-c="startNo" inputmode="numeric" value="${esc(c.startNo||1)}"></label></div>
 <div class="row"><button class="btn pri" data-act="saveco" data-id="${esc(c.id)}">Save company</button></div></div></div>`}
function rSetup(){
  $('#main').innerHTML=`<div class="stack"><div class="cogrid">${S.companies.map(coCard).join('')}</div>
  <div class="row"><button class="btn" data-act="addco">+ Add company</button></div>
  <div class="card"><h2>Defaults</h2><div class="grid2"><label class="f">Prepared-by names (comma separated)<input id="apre" value="${esc((S.app.preparers||[]).join(', '))}"></label>
   <label class="f">Default purpose<input id="apur" value="${esc(S.app.purpose||'')}"></label><label class="f">Default remarks<input id="arem" value="${esc(S.app.remarks||'')}"></label></div>
   <div class="row" style="margin-top:14px"><button class="btn pri" data-act="saveapp">Save defaults</button></div></div>
  <div class="card"><h2>Backup</h2><p class="muted" style="margin-top:0">Download a copy of every company, party and DC, or load one back.</p>
   <div class="row"><button class="btn" data-act="backup">Download backup</button><label class="btn" style="cursor:pointer">Restore backup<input type="file" id="restore" accept=".json,application/json" hidden></label></div></div></div>`;
}

/* ---------- events ---------- */
document.addEventListener('click',async ev=>{
  const b=ev.target.closest('[data-act]');if(!b||!S.db)return;const a=b.dataset.act,id=b.dataset.id;
  const go=v=>{S.view=v;try{localStorage.setItem('dc.view',v)}catch(e){}render()};
  if(a==='tab')return go(id);
  if(a==='co'){S.companyId=id;try{localStorage.setItem('dc.co',id)}catch(e){}
    if(S.draft&&!S.draft.id){S.draft.company=id;const c=co(id);S.draft.fromPlace=c.fromPlace||S.draft.fromPlace;S.draft.no=nextNo(id,S.draft.date)}
    return render()}
  if(a==='copy'){S.copy=+id;return render()}
  if(a==='cp'){S.copies[+id]=S.copies[+id]?0:1;if(!S.copies.some(Boolean))S.copies[+id]=1;try{localStorage.setItem('dc.copies',JSON.stringify(S.copies))}catch(e){}return render()}
  if(a==='ret'){S.draft.returnable=id==='1';return render()}
  if(a==='additem'){S.draft.items.push({d:'',x:'',q:'',u:'Nos',v:''});render();const f=document.querySelectorAll('[data-k="d"]');f[f.length-1]?.focus();return}
  if(a==='delitem'){S.draft.items.splice(+id,1);if(!S.draft.items.length)S.draft.items.push({d:'',x:'',q:'',u:'Nos',v:''});return render()}
  if(a==='newdc'){S.draft=newDraft();return render()}
  if(a==='save'){await saveDraft();return render()}
  if(a==='pdfdraft'){const r=await saveDraft(true);if(r)await savePdf(r);return}
  if(a==='edit'){const d=S.dcs.find(x=>x.id===id);S.draft={...d,no:String(d.no),items:d.items.map(i=>({...i}))};S.companyId=d.company;S.copy=0;return go('new')}
  if(a==='printdraft'){const r=await saveDraft(true);if(r)doPrint(r);return render()}
  if(a==='print'){return doPrint(S.dcs.find(x=>x.id===id))}
  if(a==='pdf'){return savePdf(S.dcs.find(x=>x.id===id))}
  if(a==='del'||a==='delp'){S.delId=id;return render()}
  if(a==='delno'){S.delId=null;return render()}
  if(a==='delyes'){await S.db.doc('dcs/'+id).delete();S.delId=null;toast('DC deleted');return}
  if(a==='delpyes'){await S.db.doc('parties/'+id).delete();S.delId=null;toast('Party deleted');return}
  if(a==='csv'){
    const dl=await claude.use('downloads');if(!dl)return toast('Downloads are not available in this view.');
    const q=v=>'"'+String(v??'').replace(/"/g,'""')+'"';const rows=[['DC No','Date','Company','Party','Item','Size','Qty','Unit','Value','Returnable','Prepared by']];
    for(const d of [...S.dcs].sort((x,y)=>x.date.localeCompare(y.date)||x.no-y.no))for(const i of d.items)rows.push([d.no,fmtDate(d.date),co(d.company)?.name||'',d.partyName,i.d,i.x,i.q,i.u,i.v,d.returnable?'Returnable':'Non-Returnable',d.preparedBy]);
    try{await dl.save({filename:'DC-register.csv',data:'﻿'+rows.map(r=>r.map(q).join(',')).join('\n')})}catch(e){}
    return}
  if(a==='saveparty'){const n=$('#pn').value.trim();if(!n)return toast('Enter the party name.');const pid=S.editParty?.id||'p'+Date.now().toString(36);
    await S.db.doc('parties/'+pid).set({name:n,address:$('#pa').value});S.editParty=null;toast('Party saved');return}
  if(a==='editparty'){S.editParty=S.parties.find(p=>p.id===id);return render()}
  if(a==='cancelparty'){S.editParty=null;return render()}
  if(a==='addco'){const nid='c'+Date.now().toString(36);await S.db.doc('companies/'+nid).set({name:'NEW COMPANY',short:'New',addr:'',gstin:'',signatory:'',fromPlace:'',startNo:1,order:S.companies.length});return}
  if(a==='saveco'){const card=b.closest('[data-co]');const o={};card.querySelectorAll('[data-c]').forEach(i=>o[i.dataset.c]=i.value);
    if(!o.name.trim())return toast('Enter the company name.');o.startNo=+o.startNo||1;o.order=co(id).order||0;await S.db.doc('companies/'+id).set(o);toast('Company saved');return}
  if(a==='saveapp'){S.app={preparers:$('#apre').value.split(',').map(s=>s.trim()).filter(Boolean),purpose:$('#apur').value,remarks:$('#arem').value};await S.db.doc('config/app').set(S.app);toast('Defaults saved');return}
  if(a==='backup'){const dl=await claude.use('downloads');if(!dl)return toast('Downloads are not available in this view.');
    try{await dl.save({filename:'dc-maker-backup-'+today()+'.json',data:JSON.stringify({companies:S.companies,parties:S.parties,dcs:S.dcs,app:S.app},null,1)})}catch(e){}return}
});
document.addEventListener('input',ev=>{
  const t=ev.target,d=S.draft;
  if(t.dataset.f&&d){const f=t.dataset.f;d[f]=t.value;
    if(f==='partyName'){const p=S.parties.find(x=>x.name.toLowerCase()===t.value.trim().toLowerCase());if(p){d.partyAddr=p.address||'';const ta=$('#paddr');if(ta)ta.value=d.partyAddr}}
    if(f==='date'&&!d.id&&t.value)d.no=nextNo(d.company,t.value),document.querySelector('[data-f="no"]').value=d.no;
    return updPreview()}
  if(t.dataset.k&&d){d.items[+t.dataset.i][t.dataset.k]=t.value;return updPreview()}
  if(t.id==='rq'){S.regQ=t.value;const p=t.selectionStart;rReg();const n=$('#rq');n.focus();n.setSelectionRange(p,p)}
  if(t.id==='pq'){S.partyQ=t.value;const p=t.selectionStart;rParties();const n=$('#pq');n.focus();n.setSelectionRange(p,p)}
});
document.addEventListener('change',async ev=>{
  const t=ev.target;
  if(t.id==='rco'){S.regCo=t.value;rReg()}if(t.id==='rfy'){S.regFy=t.value;rReg()}
  if(t.dataset.k&&S.draft)updPreview();
  if(t.id==='restore'&&t.files[0]){try{const j=JSON.parse(await t.files[0].text());const w=[];
    (j.companies||[]).forEach(c=>{const{id,...r}=c;w.push(S.db.doc('companies/'+id).set(r))});(j.parties||[]).forEach(c=>{const{id,...r}=c;w.push(S.db.doc('parties/'+id).set(r))});
    (j.dcs||[]).forEach(c=>{const{id,...r}=c;w.push(S.db.doc('dcs/'+id).set(r))});if(j.app)w.push(S.db.doc('config/app').set(j.app));
    await Promise.all(w);toast('Backup restored')}catch(e){toast('That file is not a DC Maker backup.')}}
});

/* ---------- data ---------- */
(async()=>{
  render();
  const db=await claude.use('db');if(!db){render();return}S.db=db;
  const err=e=>toast('Data error: '+(e.message||e.code));
  db.collection('companies').onSnapshot(s=>{S.companies=s.docs.map(x=>({id:x.id,...x.data()})).sort((a,b)=>(a.order||0)-(b.order||0));S.loaded.c=1;
    if(!co(S.companyId))S.companyId=S.companies[0]?.id;if(S.draft&&!S.draft.id&&!co(S.draft.company))S.draft=null;
    if(S.view==='setup'&&document.activeElement&&document.activeElement.closest('[data-co]'))return;render()},err);
  db.collection('parties').onSnapshot(s=>{S.parties=s.docs.map(x=>({id:x.id,...x.data()}));S.loaded.p=1;if(S.view==='new'&&S.draft)return refreshParty();render()},err);
  db.collection('dcs').onSnapshot(s=>{S.dcs=s.docs.map(x=>({id:x.id,...x.data()}));S.loaded.d=1;if(S.view==='new'&&S.draft){if(!S.draft.id)S.draft.no=S.draft.no;if(S.loaded.d&&!S.draft._init){S.draft._init=1;if(!S.draft.id)S.draft.no=nextNo(S.draft.company,S.draft.date)}return render()}render()},err);
  db.doc('config/app').onSnapshot(s=>{if(s.exists)S.app={...S.app,...s.data()};S.loaded.a=1},err);
})();
function refreshParty(){const dl=$('#plist');if(dl)dl.innerHTML=S.parties.map(p=>`<option value="${esc(p.name)}">`).join('')}
</script>
</body></html>

