<!doctype html><html lang="en"><head><meta charset="utf-8"/><meta name="viewport" content="width=device-width,initial-scale=1"/><title>The Darpan Wears</title>
<style>
:root{--bg:#fafafa;--card:#fff;--muted:#6b7280;--accent:#ff6b00;--radius:12px;--shadow:0 6px 18px rgba(15,23,36,0.06);--text:#0f1724}
*{box-sizing:border-box}body{margin:0;background:var(--bg);font-family:Inter,system-ui,Roboto,-apple-system;color:var(--text);-webkit-font-smoothing:antialiased}
.app{max-width:480px;margin:0 auto;padding:12px}
.header{display:flex;justify-content:space-between;align-items:center;gap:10px}
.brand{display:flex;gap:10px;align-items:center}
.logo{width:46px;height:46px;border-radius:10px;background:linear-gradient(135deg,var(--accent),#06b6d4);display:flex;align-items:center;justify-content:center;color:#fff;font-weight:800}
.title{font-weight:800}
.subtitle{font-size:12px;color:var(--muted)}
.card{background:var(--card);padding:12px;border-radius:12px;box-shadow:var(--shadow);margin-top:12px}
.chips{display:flex;gap:8px;margin-top:8px;flex-wrap:wrap}
.chip{background:#f3f4f6;padding:8px 12px;border-radius:999px;font-weight:700;color:var(--muted);font-size:13px;border:1px solid rgba(0,0,0,0.03)}
.products{display:grid;grid-template-columns:1fr;gap:10px;margin-top:12px}
.product{display:flex;gap:10px;align-items:center;padding:10px;border-radius:10px;background:var(--card);box-shadow:var(--shadow)}
.product img{width:86px;height:86px;object-fit:cover;border-radius:8px}
.prod-body{flex:1}
.prod-title{font-weight:700}
.prod-desc{font-size:13px;color:var(--muted);margin-top:6px}
.price{margin-top:8px;font-weight:800}
.actions{display:flex;gap:8px;margin-top:8px}
.btn{background:var(--accent);color:#fff;border:0;padding:8px 10px;border-radius:10px;font-weight:700}
.btn.out{background:#fff;color:var(--text);border:1px solid rgba(0,0,0,0.06)}
.footer{margin-top:18px;text-align:center;color:var(--muted);font-size:13px}
.small{font-size:13px;color:var(--muted)}
.admin-modal, .cart-drawer{position:fixed;left:0;right:0;bottom:0;background:rgba(0,0,0,0.4);display:none;align-items:flex-end;justify-content:center;padding:10px}
.panel{width:100%;max-width:480px;background:#fff;border-radius:12px;padding:12px}
.file{border:2px dashed rgba(0,0,0,0.06);padding:8px;border-radius:8px;text-align:center;margin-top:6px}
input,textarea,select{width:100%;padding:8px;margin-top:6px;border-radius:8px;border:1px solid rgba(0,0,0,0.06)}
.canvas-wrap{overflow:auto}
.chart{width:100%;height:140px;background:#fff;border-radius:8px;padding:6px;margin-top:8px}
@media(min-width:481px){.app{margin:18px auto}}
</style>
</head><body>
<div class="app">
  <div class="header">
    <div class="brand"><div class="logo">DW</div><div><div class="title">The Darpan Wears</div><div class="subtitle">T-Shirts • Jeans • Jackets • Shoes • Electronic Gadgets</div></div></div>
    <div style="display:flex;gap:8px;align-items:center">
      <button id="adminBtn" class="btn out">Admin</button>
      <button id="cartBtn" class="btn out">Cart (<span id="cartCount">0</span>)</button>
    </div>
  </div>

  <div class="card">
    <h2>Fresh styles. Honest prices.</h2>
    <div class="small">Fast ordering • Cash on delivery available</div>
    <div class="chips" id="cats"></div>
  </div>

  <div class="products card" id="productGrid"></div>

  <div class="card">
    <div style="display:flex;justify-content:space-between;align-items:center">
      <div><strong>Total</strong> <span id="totalPrice">₹0</span></div>
      <div><button id="checkoutBtn" class="btn">Order</button></div>
    </div>
  </div>

  <div class="footer">© <span id="year"></span> The Darpan Wears</div>
</div>

<!-- Admin modal -->
<div class="admin-modal" id="adminModal">
  <div class="panel">
    <div style="display:flex;justify-content:space-between;align-items:center">
      <strong>Admin Panel</strong>
      <div>
        <button id="closeAdmin" class="btn out">Close</button>
      </div>
    </div>
    <div id="adminArea" style="margin-top:8px"></div>
  </div>
</div>

<!-- Cart drawer / checkout -->
<div class="cart-drawer" id="cartModal">
  <div class="panel">
    <div style="display:flex;justify-content:space-between;align-items:center">
      <strong>Cart / Checkout</strong>
      <div><button id="closeCart" class="btn out">Close</button></div>
    </div>
    <div id="cartList" style="margin-top:8px"></div>
    <div style="margin-top:8px">
      <button id="proceedBtn" class="btn">Proceed to Order Form</button>
    </div>
  </div>
</div>

<script>
/* compact single-file app (front-end-only) */
/* stored data keys: dw_products, dw_orders, dw_cart */
const CATS = ['T-Shirts','Jeans','Jackets','Shoes','Electronic Gadgets'];
const HEX_ADMIN_HASH = '0412d1616b1bfa094eef7a24818416487283351f88cc2675eb7177ffdf75cf19'; // sha256('darpan2025')
const q = s=>document.querySelector(s), id=s=>document.getElementById(s);

function sha256hex(msg){
  const enc = new TextEncoder().encode(msg);
  return crypto.subtle.digest('SHA-256', enc).then(buf=>{
    return [...new Uint8Array(buf)].map(b=>b.toString(16).padStart(2,'0')).join('');
  });
}

/* storage helpers */
const read=(k)=>{try{return JSON.parse(localStorage.getItem(k)||'[]')}catch(e){return[]}};
const write=(k,v)=>localStorage.setItem(k,JSON.stringify(v));

/* products init */
if(!localStorage.getItem('dw_products')) write('dw_products',[
  {id:'p_sample1',name:'Basic T-Shirt',desc:'Comfort cotton',sell:399,special:299,size:'M',material:'Cotton',category:'T-Shirts',img:'',sold:0},
  {id:'p_sample2',name:'Slim Jeans',desc:'Blue denim',sell:1199,special:'',size:'32',material:'Denim',category:'Jeans',img:'',sold:0}
]);

/* render categories */
const cats = id('cats');
CATS.forEach(c=>{const d=document.createElement('div');d.className='chip';d.textContent=c;d.onclick=()=>renderProducts(c);cats.appendChild(d)});

/* product rendering */
function renderProducts(filter){
  const grid = id('productGrid'); grid.innerHTML='';
  const products = read('dw_products');
  const list = filter ? products.filter(p=>p.category===filter) : products;
  list.forEach(p=>{
    const card=document.createElement('div');card.className='product';
    const img=document.createElement('img'); img.src = p.img || '/uploads/placeholder.png'; img.alt=p.name;
    const body=document.createElement('div'); body.className='prod-body';
    body.innerHTML = '<div class="prod-title">'+escapeHtml(p.name)+'</div><div class="prod-desc">'+escapeHtml(p.desc)+'</div><div class="price">'+(p.special ? '<span style="color:var(--accent)">₹'+p.special+'</span> <span class="small" style="text-decoration:line-through;margin-left:8px">₹'+p.sell+'</span>' : '₹'+p.sell)+'</div>';
    const actions=document.createElement('div');actions.className='actions';
    const add=document.createElement('button');add.className='btn';add.innerText='Add';add.onclick=()=>addToCart(p.id);
    const ord=document.createElement('button');ord.className='btn out';ord.innerText='Order';ord.onclick=()=>{addToCart(p.id); openCart(); openCheckout();};
    actions.appendChild(add);actions.appendChild(ord); body.appendChild(actions);
    const left=document.createElement('div'); left.appendChild(img);
    card.appendChild(left); card.appendChild(body); grid.appendChild(card);
  });
  updateCartUI();
}

/* cart */
function getCart(){return read('dw_cart')}
function saveCart(c){write('dw_cart',c)}
function addToCart(id){const c=getCart(); const it=c.find(x=>x.id===id); if(it)it.qty++; else c.push({id,qty:1}); saveCart(c); updateCartUI()}
function removeFromCart(id){let c=getCart(); c=c.filter(x=>x.id!==id); saveCart(c); updateCartUI()}
function updateCartUI(){
  const cart = getCart(); id('cartCount').innerText = cart.reduce((s,i)=>s+i.qty,0);
  let total = 0; const products=read('dw_products');
  const node = id('cartList'); node.innerHTML='';
  cart.forEach(ci=>{ const p = products.find(x=>x.id===ci.id); if(!p) return; total += (p.special||p.sell)*ci.qty;
    const r=document.createElement('div'); r.style.display='flex'; r.style.justifyContent='space-between'; r.style.marginBottom='8px'; r.innerHTML = '<div><strong>'+escapeHtml(p.name)+'</strong><div class="small">₹'+(p.special||p.sell)+' × '+ci.qty+'</div></div>';
    const rm = document.createElement('button'); rm.className='btn out'; rm.innerText='Remove'; rm.onclick=()=>{ removeFromCart(ci.id) };
    r.appendChild(rm); node.appendChild(r);
  });
  id('totalPrice').innerText = '₹'+total;
}

/* checkout (order form via prompts for compactness) */
async function openCheckout(){
  const cart = getCart(); if(cart.length===0){ alert('Cart is empty'); return; }
  const name = prompt('Name'); if(!name)return; const phone = prompt('Phone number'); if(!phone)return;
  const state = prompt('State'); if(!state)return; const city = prompt('City'); if(!city)return;
  const village = prompt('Village (optional)') || ''; const pincode = prompt('Pincode'); if(!pincode) return;
  const payment = confirm('OK = Cash on Delivery, Cancel = Online Payment') ? 'COD' : 'Online';
  const products = read('dw_products');
  const items = cart.map(ci=>{ const p=products.find(x=>x.id===ci.id); return {id:p.id,name:p.name,qty:ci.qty,unitPrice:(p.special||p.sell)}});
  const total = items.reduce((s,i)=>s+i.qty*i.unitPrice,0);
  const orders = read('dw_orders'); const order = {id:'ord_'+Date.now().toString(36),items,customer:{name,phone,state,city,village,pincode},payment,total,date:new Date().toISOString()};
  orders.unshift(order); write('dw_orders',orders);
  // increment sold
  const prods = read('dw_products'); items.forEach(it=>{ const p=prods.find(x=>x.id===it.id); if(p) p.sold=(p.sold||0)+it.qty; }); write('dw_products',prods);
  saveCart([]); updateCartUI(); renderProducts(); alert('Order placed — saved to admin panel');
  renderAdminOrders(); renderChart();
}

/* admin password flow */
id('adminBtn').onclick=async()=>{
  const p = prompt('Enter admin password'); if(!p) return;
  const h = await sha256hex(p);
  if(h!==HEX_ADMIN_HASH){ alert('Wrong password'); return; }
  openAdmin();
}

/* admin UI */
function openAdmin(){ id('adminModal').style.display='flex'; renderAdminArea(); }
id('closeAdmin').onclick=()=>id('adminModal').style.display='none';

function renderAdminArea(){
  const area = id('adminArea'); area.innerHTML = '';
  // add product form
  area.innerHTML = `
    <div><input id="admName" placeholder="Name"></div>
    <div><textarea id="admDesc" placeholder="Description"></textarea></div>
    <div style="display:flex;gap:8px"><input id="admSell" placeholder="Selling Price"><input id="admSpecial" placeholder="Special Price"></div>
    <div style="display:flex;gap:8px"><input id="admSize" placeholder="Size"><input id="admMaterial" placeholder="Material"></div>
    <div><select id="admCategory">${CATS.map(c=>' <option>'+c+'</option>').join('')}</select></div>
    <div class="file"><input id="admFile" type="file" accept="image/*"></div>
    <div id="admPreview"></div>
    <div style="margin-top:8px"><button id="admAdd" class="btn">Add product</button></div>
    <hr/><div id="admList"></div><hr/><div><strong>Orders</strong><div id="admOrders"></div><div class="chart"><canvas id="salesChart" width="360" height="120"></canvas></div></div>
  `;
  const file = id('admFile'); file.onchange=()=>{ const f=file.files[0]; if(!f) return; const r=new FileReader(); r.onload=e=>id('admPreview').innerHTML='<img src="'+e.target.result+'" style="max-width:120px;border-radius:8px">'; r.readAsDataURL(f) };
  id('admAdd').onclick=async()=>{
    const name=id('admName').value.trim(); if(!name){ alert('Name required'); return; }
    const desc=id('admDesc').value; const sell=Number(id('admSell').value)||0; const special=id('admSpecial').value||''; const size=id('admSize').value||''; const material=id('admMaterial').value||''; const category=id('admCategory').value;
    let img=''; if(file.files[0]){ img = await new Promise(r=>{ const fr=new FileReader(); fr.onload=e=>r(e.target.result); fr.readAsDataURL(file.files[0]); }) }
    const prods = read('dw_products'); const idv='p_'+Date.now().toString(36);
    prods.unshift({id:idv,name,desc,sell,special,size,material,category,img,sold:0}); write('dw_products',prods);
    id('admName').value=''; id('admDesc').value=''; id('admSell').value=''; id('admSpecial').value=''; id('admPreview').innerHTML=''; renderProducts(); renderAdminList(); renderChart();
  };
  renderAdminList(); renderAdminOrders(); renderChart();
}

function renderAdminList(){
  const wrap=id('admList'); wrap.innerHTML=''; const prods=read('dw_products');
  prods.forEach(p=>{ const r=document.createElement('div'); r.style.display='flex'; r.style.justifyContent='space-between'; r.style.marginBottom='8px'; r.innerHTML = '<div><strong>'+escapeHtml(p.name)+'</strong><div class="small">₹'+p.sell+' • sold:'+ (p.sold||0) +'</div></div>';
    const right=document.createElement('div'); const del=document.createElement('button'); del.className='btn out'; del.innerText='Delete'; del.onclick=()=>{ if(confirm('Delete?')){ write('dw_products', read('dw_products').filter(x=>x.id!==p.id)); renderProducts(); renderAdminList(); renderChart(); } };
    right.appendChild(del); r.appendChild(right); wrap.appendChild(r);
  });
}

function renderAdminOrders(){
  const out=id('admOrders'); out.innerHTML=''; const orders=read('dw_orders');
  orders.forEach(o=>{ const d=document.createElement('div'); d.style.marginBottom='8px'; d.innerHTML = '<strong>'+escapeHtml(o.customer.name)+'</strong> <div class="small">'+new Date(o.date).toLocaleString()+' • ₹'+o.total+'</div><div class="small">Phone: '+escapeHtml(o.customer.phone)+'</div><div class="small">Address: '+escapeHtml(o.customer.village||'')+', '+escapeHtml(o.customer.city)+', '+escapeHtml(o.customer.state)+' - '+escapeHtml(o.customer.pincode)+'</div>';
    out.appendChild(d);
  });
}

/* sales chart */
function renderChart(){
  const prods = read('dw_products'); const canvas = id('salesChart'); if(!canvas) return;
  const ctx = canvas.getContext('2d'); ctx.clearRect(0,0,canvas.width,canvas.height);
  const vals = prods.map(p=>p.sold||0); const names = prods.map(p=>p.name.slice(0,8));
  const max = Math.max(1, ...vals); const barW=20; const gap=8;
  canvas.width = Math.max(360, (barW+gap)*vals.length + 20); const H=120; canvas.height=H;
  vals.forEach((v,i)=>{ const x = 10 + i*(barW+gap); const h = Math.round((v/max)*(H-30)); ctx.fillStyle='#ff6b00'; ctx.fillRect(x, H-20-h, barW, h); ctx.fillStyle='#6b7280'; ctx.font='10px sans-serif'; ctx.fillText(names[i], x, H-4); ctx.fillText(String(v), x, H-26-h); });
}

/* misc UI */
id('cartBtn').onclick=()=>{ id('cartModal').style.display='flex'; }
id('closeCart').onclick=()=>{ id('cartModal').style.display='none'; }
id('proceedBtn').onclick=()=>{ id('cartModal').style.display='none'; openCheckout(); }
id('checkoutBtn').onclick=()=>{ openCart(); openCheckout(); }
function openCart(){ id('cartModal').style.display='flex' }
function openCheckout(){ openCheckout } // placeholder (actual handled above in quickOrder or checkoutBtn)

/* helper */
function escapeHtml(s){ return String(s||'').replace(/[&<>"]/g,c=>({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;'}[c])); }

/* init */
renderProducts(); updateCartUI(); document.getElementById('year').innerText=new Date().getFullYear();

/* placeholder image route: if /uploads/placeholder.png fails, browser will just show broken image — acceptable for compact file */
</script>
</body></html>
