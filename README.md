# hater-muthoy
<html lang="bn">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width,initial-scale=1">
  <title>SimpleShop — Responsive E‑commerce (Demo)</title>
  <meta name="description" content="Responsive E‑commerce demo. Static single-file storefront with cart (localStorage) and mock checkout.">
  <style>
    :root{
      --accent:#0ea5a4; --bg:#f7f7fb; --card:#ffffff; --muted:#6b7280;
      --maxw:1100px;
      font-family: Inter, system-ui, -apple-system, 'Segoe UI', Roboto, 'Helvetica Neue', Arial;
    }
    *{box-sizing:border-box}
    body{margin:0;background:var(--bg);color:#111;line-height:1.4}
    .wrap{max-width:var(--maxw);margin:18px auto;padding:16px}
    header{display:flex;align-items:center;justify-content:space-between;gap:12px}
    .brand{display:flex;gap:12px;align-items:center}
    .logo{width:42px;height:42px;background:linear-gradient(135deg,var(--accent),#60a5fa);border-radius:10px;display:flex;align-items:center;justify-content:center;color:white;font-weight:700}
    h1{font-size:18px;margin:0}
    .search{flex:1;display:flex;gap:8px;margin-left:12px}
    input[type=search]{flex:1;padding:10px 12px;border-radius:10px;border:1px solid #e6e6ef;background:white}
    .actions{display:flex;gap:8px;align-items:center}
    .btn{background:var(--accent);color:white;padding:8px 12px;border-radius:10px;border:0;cursor:pointer}
    .btn.ghost{background:transparent;color:var(--accent);border:1px solid var(--accent)}

    main{display:grid;grid-template-columns:1fr 360px;gap:18px;margin-top:18px}
    @media (max-width:1000px){main{grid-template-columns:1fr 320px}}
    @media (max-width:820px){main{grid-template-columns:1fr} .cart-panel{order:2}}

    .filters{display:flex;gap:8px;flex-wrap:wrap;margin-bottom:8px}
    .chip{padding:8px 10px;background:var(--card);border-radius:999px;border:1px solid #eef2ff;cursor:pointer}

    .products{display:grid;grid-template-columns:repeat(3,1fr);gap:12px}
    @media (max-width:900px){.products{grid-template-columns:repeat(2,1fr)}}
    @media (max-width:520px){.products{grid-template-columns:repeat(1,1fr)}}

    .card{background:var(--card);border-radius:12px;padding:12px;box-shadow:0 4px 18px rgba(15,23,42,0.05)}
    .prod-img{width:100%;height:160px;border-radius:8px;background-size:cover;background-position:center;margin-bottom:10px}
    .prod-title{font-weight:600;margin:0 0 6px 0}
    .prod-price{color:var(--accent);font-weight:700}
    .prod-meta{color:var(--muted);font-size:13px}

    .cart-panel{position:relative}
    .cart-panel .card{padding:12px}
    .cart-items{max-height:420px;overflow:auto;margin-bottom:10px}
    .cart-row{display:flex;gap:8px;align-items:center;padding:8px 0;border-bottom:1px solid #f1f3f7}
    .cart-row img{width:56px;height:56px;border-radius:8px;object-fit:cover}
    .qty{display:flex;gap:6px;align-items:center}
    .small{font-size:13px;color:var(--muted)}

    footer{margin-top:24px;text-align:center;color:var(--muted);font-size:13px}

    /* modal */
    .modal{position:fixed;inset:0;display:none;align-items:center;justify-content:center;background:rgba(0,0,0,0.4);z-index:60}
    .modal.open{display:flex}
    .modal .box{width:100%;max-width:780px;background:var(--card);border-radius:12px;padding:16px}
    .row{display:flex;gap:12px}
    .col{flex:1}
    label{display:block;font-size:13px;margin-bottom:6px}
    input,textarea,select{width:100%;padding:10px;border-radius:8px;border:1px solid #e6e6ef}

    /* tiny admin bar */
    .admin-toggle{position:fixed;right:12px;bottom:12px;z-index:80}
  </style>
</head>
<body>
  <div class="wrap">
    <header>
      <div class="brand">
        <div class="logo">SS</div>
        <div>
          <h1>SimpleShop</h1>
          <div class="small">Demo responsive e‑commerce — ডেমো স্টোর</div>
        </div>
      </div>

      <div class="search">
        <input id="q" type="search" placeholder="পণ্য খুঁজুন — Search products...">
        <button id="searchBtn" class="btn">Search</button>
      </div>

      <div class="actions">
        <button id="viewOrders" class="btn ghost">Orders</button>
        <button id="openCart" class="btn">Cart (<span id="cartCount">0</span>)</button>
      </div>
    </header>

    <main>
      <section>
        <div class="filters card" style="display:flex;align-items:center">
          <div class="chip" data-cat="all">সব</div>
          <div class="chip" data-cat="electronics">ইলেকট্রনিকস</div>
          <div class="chip" data-cat="fashion">ফ্যাশন</div>
          <div class="chip" data-cat="home">হোম</div>
        </div>

        <div id="products" class="products"></div>
      </section>

      <aside class="cart-panel">
        <div class="card">
          <h3 style="margin-top:0">কার্ট</h3>
          <div class="cart-items" id="cartItems"></div>
          <div style="display:flex;justify-content:space-between;align-items:center">
            <div>
              <div class="small">মোট</div>
              <div id="cartTotal" style="font-weight:700;font-size:18px">৳0</div>
            </div>
            <div style="display:flex;flex-direction:column;gap:8px">
              <button id="checkoutBtn" class="btn">Checkout</button>
              <button id="clearCart" class="btn ghost">Clear</button>
            </div>
          </div>
        </div>
      </aside>
    </main>

    <footer>
      ডিজাইন ও ডেমো — SimpleShop • আপনি ডোমেইন ও হোস্টিং কিনে এখানে আপলোড করুন।
    </footer>
  </div>

  <!-- modal: product details & checkout -->
  <div id="modal" class="modal">
    <div class="box">
      <div id="modalContent"></div>
      <div style="text-align:right;margin-top:12px">
        <button id="closeModal" class="btn ghost">Close</button>
      </div>
    </div>
  </div>

  <!-- admin toggle -->
  <div class="admin-toggle">
    <button id="adminBtn" class="btn ghost">Admin</button>
  </div>

  <script>
    /* ===== sample products ===== */
    const SAMPLE_PRODUCTS = [
      {id:'p1',title:'Wireless Headphones',price:3599,cat:'electronics',img:'https://images.unsplash.com/photo-1518444022007-7e0b2f6b2e4e?q=80&w=800&auto=format&fit=crop&ixlib=rb-4.0.3&s=1d0b9c9a8b1d4d3d'},
      {id:'p2',title:'Classic Watch',price:2499,cat:'fashion',img:'https://images.unsplash.com/photo-1518544887600-0b2e5b0e5f6a?q=80&w=800&auto=format&fit=crop&ixlib=rb-4.0.3&s=2c4f3e2a'},
      {id:'p3',title:'Coffee Maker',price:5399,cat:'home',img:'https://images.unsplash.com/photo-1541167760496-1628856ab772?q=80&w=800&auto=format&fit=crop&ixlib=rb-4.0.3&s=3c9b1f9a'},
      {id:'p4',title:'Running Shoes',price:2999,cat:'fashion',img:'https://images.unsplash.com/photo-1600180758890-6b7b2f6b45a1?q=80&w=800&auto=format&fit=crop&ixlib=rb-4.0.3&s=4f'},
      {id:'p5',title:'Bluetooth Speaker',price:1899,cat:'electronics',img:'https://images.unsplash.com/photo-1585386959984-a4155222b66a?q=80&w=800&auto=format&fit=crop&ixlib=rb-4.0.3&s=5a'},
      {id:'p6',title:'Desk Lamp',price:1299,cat:'home',img:'https://images.unsplash.com/photo-1545239351-1141bd82e8a6?q=80&w=800&auto=format&fit=crop&ixlib=rb-4.0.3&s=6b'}
    ];

    /* ===== storage keys ===== */
    const KEY_PRODUCTS = 'simpleshop_products_v1';
    const KEY_CART = 'simpleshop_cart_v1';
    const KEY_ORDERS = 'simpleshop_orders_v1';

    /* ===== init ===== */
    let products = JSON.parse(localStorage.getItem(KEY_PRODUCTS) || 'null') || SAMPLE_PRODUCTS;
    let cart = JSON.parse(localStorage.getItem(KEY_CART) || '{}');
    let orders = JSON.parse(localStorage.getItem(KEY_ORDERS) || '[]');

    const productsEl = document.getElementById('products');
    const cartItemsEl = document.getElementById('cartItems');
    const cartTotalEl = document.getElementById('cartTotal');
    const cartCountEl = document.getElementById('cartCount');
    const modal = document.getElementById('modal');
    const modalContent = document.getElementById('modalContent');

    function saveState(){
      localStorage.setItem(KEY_PRODUCTS, JSON.stringify(products));
      localStorage.setItem(KEY_CART, JSON.stringify(cart));
      localStorage.setItem(KEY_ORDERS, JSON.stringify(orders));
    }

    function formatPrice(n){return '৳' + n.toLocaleString('en-US')}

    /* ===== render products ===== */
    function renderProducts(list = products){
      productsEl.innerHTML = '';
      list.forEach(p=>{
        const div = document.createElement('div'); div.className='card';
        div.innerHTML = `
          <div class="prod-img" style="background-image:url('${p.img}')"></div>
          <div class="prod-title">${p.title}</div>
          <div class="prod-meta small">Category: ${p.cat}</div>
          <div style="display:flex;justify-content:space-between;align-items:center;margin-top:8px">
            <div class="prod-price">${formatPrice(p.price)}</div>
            <div>
              <button class="btn" data-action="add" data-id="${p.id}">Add</button>
              <button class="btn ghost" data-action="view" data-id="${p.id}">View</button>
            </div>
          </div>
        `;
        productsEl.appendChild(div);
      })
    }

    /* ===== cart functions ===== */
    function addToCart(id, qty=1){
      cart[id] = (cart[id]||0) + qty;
      saveState(); renderCart();
    }
    function removeFromCart(id){ delete cart[id]; saveState(); renderCart(); }
    function changeQty(id, delta){ cart[id] = Math.max(0,(cart[id]||0)+delta); if(cart[id]===0) delete cart[id]; saveState(); renderCart(); }
    function clearCart(){ cart={}; saveState(); renderCart(); }

    function renderCart(){
      cartItemsEl.innerHTML = '';
      let total = 0; let count = 0;
      for(const id in cart){
        const p = products.find(x=>x.id===id); if(!p) continue;
        const qty = cart[id];
        const row = document.createElement('div'); row.className='cart-row';
        row.innerHTML = `
          <img src="${p.img}" alt="">
          <div style="flex:1">
            <div style="font-weight:600">${p.title}</div>
            <div class="small">${formatPrice(p.price)} × ${qty}</div>
          </div>
          <div class="qty small">
            <button data-id="${id}" class="btn ghost" data-action="dec">-</button>
            <div>${qty}</div>
            <button data-id="${id}" class="btn ghost" data-action="inc">+</button>
            <button data-id="${id}" class="btn ghost" data-action="del">Remove</button>
          </div>
        `;
        cartItemsEl.appendChild(row);
        total += p.price * qty; count += qty;
      }
      cartTotalEl.textContent = formatPrice(total);
      cartCountEl.textContent = count;
      if(Object.keys(cart).length===0){ cartItemsEl.innerHTML = '<div class="small">কার্ট খালি</div>' }
    }

    /* ===== events ===== */
    document.addEventListener('click', e=>{
      const btn = e.target.closest('button'); if(!btn) return;
      const action = btn.dataset.action;
      const id = btn.dataset.id;
      if(action==='add') addToCart(id);
      if(action==='view') openProductModal(id);
      if(action==='inc') changeQty(id,1);
      if(action==='dec') changeQty(id,-1);
      if(action==='del') removeFromCart(id);
    });

    document.getElementById('searchBtn').addEventListener('click', ()=>{
      const q = document.getElementById('q').value.trim().toLowerCase();
      if(!q){ renderProducts(); return; }
      const res = products.filter(p=>p.title.toLowerCase().includes(q) || p.cat.toLowerCase().includes(q));
      renderProducts(res);
    });

    document.querySelectorAll('.chip').forEach(c=>c.addEventListener('click', ()=>{
      const cat = c.dataset.cat; if(cat==='all'){ renderProducts(); return; }
      renderProducts(products.filter(p=>p.cat===cat));
    }));

    document.getElementById('openCart').addEventListener('click', ()=>{
      window.scrollTo({top:0,behavior:'smooth'});
    });

    document.getElementById('clearCart').addEventListener('click', ()=>{ if(confirm('Clear cart?')) clearCart(); });

    document.getElementById('checkoutBtn').addEventListener('click', ()=>{
      if(Object.keys(cart).length===0){ alert('কার্ট খালি।'); return; }
      openCheckoutModal();
    });

    document.getElementById('closeModal').addEventListener('click', ()=>{ modal.classList.remove('open'); });

    document.getElementById('viewOrders').addEventListener('click', ()=>{ showOrders(); });

    /* ===== modal helpers ===== */
    function openProductModal(id){
      const p = products.find(x=>x.id===id); if(!p) return;
      modalContent.innerHTML = `
        <div style="display:flex;gap:12px;flex-wrap:wrap">
          <div style="flex:1;min-width:260px"><img src="${p.img}" style="width:100%;border-radius:8px;object-fit:cover"></div>
          <div style="flex:1;min-width:260px">
            <h2 style="margin-top:0">${p.title}</h2>
            <div class="small">Category: ${p.cat}</div>
            <p style="font-weight:700;margin-top:8px">${formatPrice(p.price)}</p>
            <p class="small">ডেমো পণ্যের বর্ণনা। এখানে পণ্যের তথ্য, রঙ, মাপ ইত্যাদি যোগ করুন।</p>
            <div style="margin-top:12px"><button class="btn" data-action="add" data-id="${p.id}">Add to cart</button></div>
          </div>
        </div>
      `;
      modal.classList.add('open');
    }

    function openCheckoutModal(){
      const total = Object.keys(cart).reduce((s,id)=>{
        const p = products.find(x=>x.id===id); return s + (p? p.price*cart[id]:0);
      },0);
      modalContent.innerHTML = `
        <h2>Checkout</h2>
        <div class="row" style="margin-top:8px">
          <div class="col">
            <label>নাম</label>
            <input id="c_name" placeholder="আপনার নাম">
            <label>ইমেইল/ফোন</label>
            <input id="c_contact" placeholder="ইমেইল বা ফোন নম্বর">
            <label>ঠিকানা</label>
            <textarea id="c_addr" rows="3" placeholder="ডেলিভারি ঠিকানা"></textarea>
          </div>
          <div style="width:220px">
            <div class="card" style="padding:12px">
              <div class="small">Order Total</div>
              <div style="font-weight:700;font-size:20px;margin-top:6px">${formatPrice(total)}</div>
              <div class="small" style="margin-top:8px">ডেমো মোড — পেমেন্ট একত্রীকরণ যোগ করার জন্য নীচের নির্দেশ দেখুন।</div>
              <div style="margin-top:12px"><button id="placeOrder" class="btn">Place order (Demo)</button></div>
            </div>
          </div>
        </div>
      `;
      modal.classList.add('open');

      document.getElementById('placeOrder').addEventListener('click', ()=>{
        const name = document.getElementById('c_name').value.trim();
        const contact = document.getElementById('c_contact').value.trim();
        const addr = document.getElementById('c_addr').value.trim();
        if(!name || !contact || !addr){ alert('Please fill name, contact and address.'); return; }
        // build order
        const order = {id:'ord_'+Date.now(),created:new Date().toISOString(),name,contact,addr,items:[],total:0};
        for(const id in cart){ const p = products.find(x=>x.id===id); if(!p) continue; order.items.push({id,pid:id,title:p.title,price:p.price,qty:cart[id]}); order.total += p.price * cart[id]; }
        orders.push(order); saveState(); clearCart(); modal.classList.remove('open'); alert('Order placed (demo). Order id: '+order.id);
      });
    }

    function showOrders(){
      modalContent.innerHTML = `<h2>Orders</h2><div id="ordersList"></div>`; modal.classList.add('open');
      const ol = document.getElementById('ordersList'); ol.innerHTML='';
      if(orders.length===0) ol.innerHTML = '<div class="small">কোন অর্ডার নেই</div>';
      orders.slice().reverse().forEach(o=>{
        const d = document.createElement('div'); d.className='card'; d.style.marginBottom='8px';
        d.innerHTML = `<div style="display:flex;justify-content:space-between"><div><b>${o.name}</b><div class="small">${new Date(o.created).toLocaleString()}</div></div><div><b>${formatPrice(o.total)}</b></div></div>`;
        const items = document.createElement('div'); items.className='small'; items.style.marginTop='8px'; items.innerHTML = o.items.map(i=>`${i.title} × ${i.qty}`).join('<br>');
        d.appendChild(items); ol.appendChild(d);
      });
    }

    /* ===== tiny admin: add product (client-side only) ===== */
    document.getElementById('adminBtn').addEventListener('click', ()=>{
      modalContent.innerHTML = `
        <h2>Admin — Add product</h2>
        <label>Title</label><input id="a_title">
        <label>Category</label><input id="a_cat" placeholder="electronics|fashion|home">
        <label>Price (number)</label><input id="a_price" type="number">
        <label>Image URL</label><input id="a_img" placeholder="https://...jpg">
        <div style="margin-top:12px"><button id="addProd" class="btn">Add</button></div>
      `; modal.classList.add('open');
      document.getElementById('addProd').addEventListener('click', ()=>{
        const t = document.getElementById('a_title').value.trim();
        const c = document.getElementById('a_cat').value.trim()||'other';
        const p = Number(document.getElementById('a_price').value)||0; const img = document.getElementById('a_img').value.trim()||'https://images.unsplash.com/photo-1523275335684-37898b6baf30?q=80&w=800&auto=format&fit=crop&ixlib=rb-4.0.3&s=placeholder';
        if(!t || p<=0){ alert('Title and positive price required'); return; }
        const id = 'p'+(Date.now()); products.push({id,title:t,price:p,cat:c,img}); saveState(); renderProducts(); modal.classList.remove('open');
      });
    });

    /* ===== init render ===== */
    renderProducts(); renderCart();

  </script>

