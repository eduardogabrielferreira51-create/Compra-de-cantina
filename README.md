<!DOCTYPE html>
<html lang="pt-br">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Automatizador da Cantina</title>
</head>
<body>

<style>
  :root{--accent:#2b7a78}
  *{box-sizing:border-box;margin:0;padding:0}
  body{background:var(--color-background-tertiary);color:var(--color-text-primary);font-family:var(--font-sans)}
  header{background:var(--accent);color:#fff;padding:12px 16px;display:flex;align-items:center;justify-content:space-between;border-radius:var(--border-radius-lg) var(--border-radius-lg) 0 0}
  .container{padding:12px;max-width:900px;margin:0 auto}
  .grid{display:grid;grid-template-columns:1fr 320px;gap:12px}
  @media(max-width:700px){.grid{grid-template-columns:1fr}}
  .card{background:var(--color-background-primary);border-radius:var(--border-radius-lg);padding:12px;border:0.5px solid var(--color-border-tertiary)}
  .products{display:grid;grid-template-columns:repeat(auto-fill,minmax(140px,1fr));gap:10px}
  .product{border:0.5px solid var(--color-border-secondary);padding:8px;border-radius:var(--border-radius-md);display:flex;flex-direction:column;gap:6px}
  .product .name{font-weight:500;font-size:14px}
  .product .price{font-size:13px;color:var(--color-text-secondary)}
  .add-btn{background:var(--accent);color:#fff;border:0;padding:7px 10px;border-radius:var(--border-radius-md);cursor:pointer;font-size:13px;width:100%}
  .add-btn:hover{opacity:.88}
  .cart-item{display:flex;justify-content:space-between;gap:8px;align-items:center;padding:6px 0;border-bottom:0.5px solid var(--color-border-tertiary)}
  .qty{display:flex;gap:6px;align-items:center}
  .qty-btn{background:transparent;border:0.5px solid var(--color-border-secondary);color:var(--color-text-primary);width:26px;height:26px;border-radius:var(--border-radius-md);cursor:pointer;font-size:15px;display:flex;align-items:center;justify-content:center}
  .qty-btn:hover{background:var(--color-background-secondary)}
  .qty-input{width:44px;text-align:center;padding:4px;border-radius:var(--border-radius-md);border:0.5px solid var(--color-border-secondary);background:var(--color-background-secondary);color:var(--color-text-primary);font-size:13px}
  .action-btn{border:0.5px solid var(--color-border-secondary);background:transparent;color:var(--color-text-primary);padding:8px 14px;border-radius:var(--border-radius-md);cursor:pointer;font-size:13px;flex:1}
  .action-btn.primary{background:var(--accent);color:#fff;border-color:var(--accent)}
  .action-btn:hover{background:var(--color-background-secondary)}
  .action-btn.primary:hover{opacity:.88;background:var(--accent)}
  .name-field{display:flex;align-items:center;gap:8px;margin-bottom:10px;padding-bottom:10px;border-bottom:0.5px solid var(--color-border-tertiary)}
  .name-field label{font-size:13px;color:var(--color-text-secondary);white-space:nowrap}
  .name-field input{flex:1;padding:6px 10px;border-radius:var(--border-radius-md);border:0.5px solid var(--color-border-secondary);background:var(--color-background-secondary);color:var(--color-text-primary);font-size:13px}
  pre.summary{background:var(--color-background-secondary);border-radius:var(--border-radius-md);padding:10px;font-size:13px;white-space:pre-wrap;color:var(--color-text-primary);margin-top:10px}
  footer{text-align:center;font-size:12px;color:var(--color-text-secondary);padding:12px}
</style>

<div style="max-width:900px;margin:0 auto">
  <header>
    <div style="font-weight:500">Automatizador da Cantina</div>
    <div id="clock" style="font-size:14px"></div>
  </header>
  <div class="container">
    <div class="grid">
      <section class="card">
        <h3 style="font-size:15px;font-weight:500;margin-bottom:10px">Produtos</h3>
        <div class="products" id="products"></div>
      </section>
      <aside class="card">
        <h3 style="font-size:15px;font-weight:500;margin-bottom:10px">Carrinho</h3>

        <div class="name-field">
          <label for="buyerName">Seu nome</label>
          <input type="text" id="buyerName" placeholder="Ex: Maria" maxlength="50" />
        </div>

        <div id="cartList"></div>
        <div style="margin-top:8px;display:flex;justify-content:space-between;align-items:center">
          <span style="font-size:13px;color:var(--color-text-secondary)">Total</span>
          <strong id="total">R$ 0,00</strong>
        </div>
        <div style="margin-top:10px;display:flex;gap:8px">
          <button class="action-btn primary" id="checkout">Finalizar</button>
          <button class="action-btn" id="clear">Limpar</button>
        </div>
        <div id="orderResult"></div>
      </aside>
    </div>
  </div>
  <footer>Use no celular ou PC — os pedidos são salvos localmente. Compartilhe o resumo para a cantina.</footer>
</div>

<script>
const products=[
  {id:1,name:'Coxinha',price:4.5},
  {id:2,name:'Pão de Queijo',price:3.0},
  {id:3,name:'Suco Natural',price:5.0},
  {id:4,name:'Água 500ml',price:2.5},
  {id:5,name:'Sanduíche',price:7.0}
];

const $=id=>document.getElementById(id);
let cart=JSON.parse(localStorage.getItem('cantina_cart')||'{}');

function fmt(v){return 'R$ '+v.toFixed(2).replace('.',',')}
function save(){localStorage.setItem('cantina_cart',JSON.stringify(cart))}
function addToCart(id){cart[id]=(cart[id]||0)+1;save();renderCart()}
function changeQty(id,qty){if(qty<=0)delete cart[id];else cart[id]=qty;save();renderCart()}
function clearCart(){cart={};save();renderCart();$('orderResult').innerHTML=''}

function renderProducts(){
  const el=$('products');el.innerHTML='';
  products.forEach(p=>{
    const d=document.createElement('div');d.className='product';
    d.innerHTML=`<div class="name">${p.name}</div><div class="price">${fmt(p.price)}</div>`;
    const b=document.createElement('button');b.className='add-btn';b.textContent='Adicionar';
    b.onclick=()=>addToCart(p.id);d.appendChild(b);el.appendChild(d);
  });
}

function renderCart(){
  const el=$('cartList');el.innerHTML='';
  let total=0;
  Object.keys(cart).forEach(id=>{
    const p=products.find(x=>x.id==id);const q=cart[id];
    const row=document.createElement('div');row.className='cart-item';
    const left=document.createElement('div');
    left.innerHTML=`<div style="font-size:14px;font-weight:500">${p.name}</div><div style="font-size:12px;color:var(--color-text-secondary)">${fmt(p.price)} cada</div>`;
    const right=document.createElement('div');
    right.style.textAlign='right';
    const qtyRow=document.createElement('div');qtyRow.className='qty';
    const bm=document.createElement('button');bm.className='qty-btn';bm.textContent='−';bm.onclick=()=>changeQty(p.id,q-1);
    const inp=document.createElement('input');inp.type='number';inp.min=0;inp.value=q;inp.className='qty-input';
    inp.onchange=e=>changeQty(p.id,parseInt(e.target.value)||0);
    const bp=document.createElement('button');bp.className='qty-btn';bp.textContent='+';bp.onclick=()=>changeQty(p.id,q+1);
    qtyRow.append(bm,inp,bp);
    const sub=document.createElement('div');sub.style.cssText='font-size:13px;margin-top:4px;color:var(--color-text-secondary)';sub.textContent=fmt(p.price*q);
    right.append(qtyRow,sub);row.append(left,right);el.appendChild(row);
    total+=p.price*q;
  });
  $('total').textContent=fmt(total);
}

function checkout(){
  const items=Object.keys(cart).map(id=>{const p=products.find(x=>x.id==id);return{name:p.name,qty:cart[id],subtotal:p.price*cart[id]}});
  if(items.length===0){$('orderResult').innerHTML='<div style="font-size:13px;color:var(--color-text-secondary);margin-top:8px">Carrinho vazio.</div>';return}
  const total=items.reduce((s,i)=>s+i.subtotal,0);
  const name=($('buyerName').value||'').trim();
  const nameHeader=name?`Pedido de: ${name}\n`:'Pedido\n';
  const lines=items.map(i=>`${i.qty}x ${i.name} — R$ ${i.subtotal.toFixed(2).replace('.',',')}`);
  const summary=`${nameHeader}${lines.join('\n')}\nTotal: R$ ${total.toFixed(2).replace('.',',')}`;
  $('orderResult').innerHTML=`<pre class="summary">${summary}</pre>`;
  if(navigator.share){navigator.share({title:name?`Pedido de ${name}` : 'Pedido da Cantina',text:summary}).catch(()=>{})}
  else{navigator.clipboard?.writeText(summary).catch(()=>{})}
}

$('checkout').onclick=checkout;
$('clear').onclick=clearCart;
function tick(){$('clock').textContent=new Date().toLocaleTimeString('pt-BR')}
setInterval(tick,1000);tick();
renderProducts();renderCart();
</script>
