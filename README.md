<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>SNK Foods — Indian Grocery</title>
<style>
  :root {
    --turmeric: #C8860D; --leaf: #3C5A3E; --vermilion: #A63A2E; 
    --rice: #FBF4E4; --rice-2: #F3E9D2; --charcoal: #241C15; --paper: #FFFDF7;
  }
  * { box-sizing: border-box; font-family: sans-serif; margin: 0; padding: 0; }
  body { background: var(--rice); color: var(--charcoal); padding-bottom: 80px; }
  header { background: var(--charcoal); color: var(--rice); padding: 15px; text-align: center; font-weight: bold; position: sticky; top: 0; z-index: 10; }
  .hero { text-align: center; padding: 30px 15px; background: var(--rice-2); }
  .hero h2 { color: var(--vermilion); font-size: 24px; }
  .hero p { font-size: 14px; margin-top: 5px; opacity: 0.8; }
  .cat-title { background: var(--turmeric); color: white; padding: 8px 15px; font-size: 16px; font-weight: bold; margin-top: 20px; }
  .grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(160px, 1fr)); gap: 10px; padding: 10px; }
  .card { background: var(--paper); border: 1px solid var(--rice-2); border-radius: 8px; padding: 12px; display: flex; flex-direction: column; justify-content: space-between; }
  .pname { font-weight: bold; font-size: 14px; min-height: 36px; }
  .punit { font-size: 12px; color: #666; margin-bottom: 8px; }
  .prow { display: flex; align-items: center; justify-content: space-between; margin-top: auto; }
  .price { font-weight: bold; color: var(--vermilion); font-size: 14px; }
  .stepper { display: flex; align-items: center; border: 1px solid var(--turmeric); border-radius: 4px; overflow: hidden; background: white; }
  .stepper button { width: 28px; height: 28px; border: none; background: var(--rice-2); font-weight: bold; font-size: 16px; cursor: pointer; }
  .stepper .qty { width: 28px; text-align: center; font-size: 13px; font-weight: bold; }
  .float-cart { position: fixed; bottom: 15px; left: 50%; transform: translateX(-50%); background: var(--charcoal); color: var(--rice); padding: 12px 25px; border-radius: 30px; display: none; align-items: center; gap: 15px; box-shadow: 0 4px 15px rgba(0,0,0,0.3); z-index: 20; font-weight: bold; cursor: pointer; width: 90%; max-width: 340px; justify-content: space-between; }
  .float-cart .badge { background: var(--vermilion); padding: 2px 8px; border-radius: 10px; font-size: 12px; }
  .drawer { position: fixed; inset: 0; background: rgba(0,0,0,0.5); z-index: 30; display: none; justify-content: flex-end; }
  .drawer.open { display: flex; }
  .drawer-content { background: var(--paper); width: 100%; max-width: 360px; height: 100%; display: flex; flex-direction: column; padding: 15px; }
  .drawer-head { display: flex; justify-content: space-between; align-items: center; border-bottom: 1px solid var(--rice-2); padding-bottom: 10px; margin-bottom: 10px; }
  .ledger { flex: 1; overflow-y: auto; }
  .lrow { display: flex; justify-content: space-between; padding: 8px 0; border-bottom: 1px dashed var(--rice-2); font-size: 14px; }
  .ltotal { font-weight: bold; font-size: 16px; padding: 15px 0; display: flex; justify-content: space-between; border-top: 2px solid var(--turmeric); }
  .drawer-form { display: flex; flex-direction: column; gap: 10px; background: white; padding: 10px; border-radius: 8px; box-shadow: 0 2px 8px rgba(0,0,0,0.05); }
  .drawer-form input, .drawer-form select { padding: 10px; border: 1px solid var(--rice-2); border-radius: 5px; font-size: 14px; }
  .send-btn { background: var(--leaf); color: white; border: none; padding: 12px; border-radius: 5px; font-weight: bold; cursor: pointer; font-size: 15px; }
</style>
</head>
<body>

<header>🛒 SNK Foods</header>

<section class="hero">
  <h2>Fresh Indian Groceries Delivered</h2>
  <p>Den Haag • Rotterdam • Delft • Leiden</p>
</section>

<main id="catalog"></main>

<div class="float-cart" id="floatCart" onclick="openCart()">
  <span>View Order Chit 🧾</span>
  <span class="badge" id="floatBadge">0 items - €0.00</span>
</div>

<div class="drawer" id="drawer" onclick="closeCart(event)">
  <div class="drawer-content" onclick="event.stopPropagation()">
    <div class="drawer-head"><h3>Your Order Chit</h3><button onclick="closeCart(null)" style="font-size:24px; background:none; border:none; cursor:pointer;">×</button></div>
    <div class="ledger" id="ledger"></div>
    <div class="drawer-form">
      <input id="cname" placeholder="Your Full Name">
      <select id="area"><option>Den Haag</option><option>Rotterdam</option><option>Delft</option><option>Leiden</option></select>
      <input id="caddr" placeholder="Delivery Address">
      <button class="send-btn" onclick="sendOrder()">📲 Send via WhatsApp</button>
    </div>
  </div>
</div>

<script>
const WA_NUMBER = "31685259659";
const catalog = [
  { id:"veg", name:"🍅 Fresh Vegetables", items:[["Onion", "1kg", 1.99], ["Tomato", "1kg", 2.49], ["Potato", "1kg", 1.79], ["Green Chilli", "250g", 1.49]]},
  { id:"rice", name:"🌾 Rice & Flour", items:[["Idli Rice", "5kg", 9.99], ["Ponni Rice", "5kg", 11.49]]},
  { id:"dal", name:"🥣 Dals & Pulses", items:[["Toor Dal", "1kg", 3.49], ["Moong Dal", "1kg", 3.29]]}
];
let cart = {};

function renderCatalog() {
  const mainEl = document.getElementById('catalog');
  let html = '';
  catalog.forEach(cat => {
    html += `<div class="cat-title">${cat.name}</div><div class="grid">`;
    cat.items.forEach(([name, unit, price]) => {
      const id = `${cat.id}-${name.replace(/\s+/g, '')}`;
      html += `<div class="card">
        <div class="pname">${name}</div>
        <div class="punit">${unit}</div>
        <div class="prow">
          <div class="price">€${price.toFixed(2)}</div>
          <div class="stepper">
            <button onclick="updateQty('${id}','${name}',${price},-1)">−</button>
            <div class="qty" id="qty-${id}">0</div>
            <button onclick="updateQty('${id}','${name}',${price},1)">+</button>
          </div>
        </div>
      </div>`;
    });
    html += `</div>`;
  });
  mainEl.innerHTML = html;
}

function updateQty(id, name, price, delta) {
  if (!cart[id]) cart[id] = { name, price, qty: 0 };
  cart[id].qty += delta;
  if (cart[id].qty <= 0) delete cart[id];
  
  const qtyEl = document.getElementById(`qty-${id}`);
  if (qtyEl) qtyEl.textContent = cart[id] ? cart[id].qty : 0;
  
  updateCartUI();
}

function updateCartUI() {
  let count = 0, total = 0;
  Object.values(cart).forEach(item => { count += item.qty; total += item.qty * item.price; });
  
  const fc = document.getElementById('floatCart');
  const fb = document.getElementById('floatBadge');
  
  if (count > 0) {
    fb.textContent = `${count} items - €${total.toFixed(2)}`;
    fc.style.display = 'flex';
  } else {
    fc.style.display = 'none';
  }
}

function openCart() { document.getElementById('drawer').classList.add('open'); renderLedger(); }
function closeCart(e) { document.getElementById('drawer').classList.remove('open'); }

function renderLedger() {
  const ledgerEl = document.getElementById('ledger');
  let html = '', total = 0;
  Object.values(cart).forEach(item => {
    total += item.qty * item.price;
    html += `<div class="lrow"><span>${item.name} (x${item.qty})</span><span>€${(item.qty * item.price).toFixed(2)}</span></div>`;
  });
  html += `<div class="ltotal"><span>Total Amount</span><span>€${total.toFixed(2)}</span></div>`;
  ledgerEl.innerHTML = html;
}

function sendOrder() {
  const name = document.getElementById('cname').value;
  const area = document.getElementById('area').value;
  const addr = document.getElementById('caddr').value;
  if(!name || !addr) { alert("Please fill Name and Address"); return; }
  
  let msg = `*New Order from SNK Foods*\n\n*Name:* ${name}\n*Area:* ${area}\n*Address:* ${addr}\n\n*Items:*\n`;
  let total = 0;
  Object.values(cart).forEach(item => {
    msg += `- ${item.name} x ${item.qty} (€${(item.qty * item.price).toFixed(2)})\n`;
    total += item.qty * item.price;
  });
  msg += `\n*Total Bill:* €${total.toFixed(2)}`;
  window.open(`https://wa.me/${WA_NUMBER}?text=${encodeURIComponent(msg)}`, '_blank');
}

renderCatalog();
</script>
</body>
</html>
