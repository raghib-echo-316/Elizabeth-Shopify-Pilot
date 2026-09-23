# 🛒 Shopify Sidecart (Drawer Cart) Complete Guide - Vanilla JavaScript
> **Elizabeth Theme** | Vanilla JavaScript + Tailwind CSS + Shopify AJAX Cart API (No Alpine.js)

---

## 📌 সূচিপত্র (Table of Contents)
1. [আর্কিটেকচার ও কাজের সারসংক্ষেপ (Overview & Architecture)](#1-আর্কিটেকচার-ও-কাজের-সারসংক্ষেপ-overview--architecture)
2. [ধাপ ১: জাভাস্ক্রিপ্ট কন্ট্রোলার তৈরি (`assets/application.js`)](#ধাপ-১-জাভাস্ক্রিপ্ট-কন্ট্রোলার-তৈরি-assetsapplicationjs)
3. [ধাপ ২: সাইডকার্ট ড্রয়ার স্নিপেট তৈরি (`snippets/side-cart.liquid`)](#ধাপ-২-সাইডকার্ট-ড্রয়ার-স্নিপেট-তৈরি-snippetsside-cartliquid)
4. [ধাপ ৩: লেআউটে স্নিপেট যুক্ত করা (`layout/theme.liquid`)](#ধাপ-৩-লেআউটে-স্নিপেট-যুক্ত-করা-layoutthemeliquid)
5. [ধাপ ৪: হেডার ন্যাভিগেশন ও লাল ব্যাজ আপডেট (`sections/header.liquid`)](#ধাপ-৪-হেডার-ন্যাভিগেশন-ও-লাল-ব্যাজ-আপডেট-sectionsheaderliquid)
6. [ধাপ ৫: প্রোডাক্ট পেজে AJAX Add-to-Cart কানেক্ট করা (`sections/template-product.liquid`)](#ধাপ-৫-প্রোডাক্ট-পেজে-ajax-add-to-cart-কানেক্ট-করা-sectionstemplate-productliquid)
7. [ধাপ ৬: টেস্টিং ও ভেরিফিকেশন চেকলিস্ট (Testing Checklist)](#ধাপ-৬-টেস্টিং-ও-ভেরিফিকেশন-চেকলিস্ট-testing-checklist)
8. [সাধারণ সমস্যা ও সমাধান (Troubleshooting & FAQs)](#সাধারণ-সমস্যা-ও-সমাধান-troubleshooting--faqs)

---

## ১. আর্কিটেকচার ও কাজের সারসংক্ষেপ (Overview & Architecture)

এই গাইডে কোনো বাহ্যিক ফ্রেমওয়ার্ক (**Alpine.js বা jQuery ছাড়া**) সম্পূর্ণ **Vanilla JavaScript (পিওর জাভাস্ক্রিপ্ট)** এবং **Tailwind CSS** ব্যবহার করে সাইডকার্ট ড্রয়ার তৈরি করার প্রক্রিয়া ধাপে ধাপে দেখানো হয়েছে।

### প্রধান ৩টি ফিচার (রেফারেন্স ইমেজ অনুযায়ী):
1. **হেডার কার্ট ব্যাজ:** হেডারের শপিং ব্যাগ আইকনের উপরে লাল বৃত্তাকার ব্যাজে আইটেম সংখ্যা (`item_count`) রিয়েল-টাইমে দেখা যাবে।
2. **যেকোনো পেজে কার্ট আইকনে ক্লিকে ড্রয়ার ওপেন:** হেডারের কার্ট আইকনে ক্লিক করলে পেজ রিফ্রেশ না হয়ে ডান দিক থেকে মসৃণভাবে সাইডকার্ট ড্রয়ারটি স্লাইড হয়ে খুলে যাবে।
3. **প্রোডাক্ট পেজ থেকে Add to Cart এ সরাসরি ওপেন:** প্রোডাক্ট পেজে "Add to Cart" বাটনে চাপ দিলে Shopify AJAX API-এর মাধ্যমে ব্যাকগ্রাউন্ডে প্রোডাক্ট কার্টে যোগ হবে এবং সাথে সাথে সাইডকার্টটি ড্রয়ার আকারে ওপেন হয়ে যাবে।

### 🧩 উপাদানগুলোর মধ্যে সম্পর্ক (Data Flow Diagram):

```text
┌─────────────────────────────────────────────────────────────┐
│                    Vanilla JS SideCart Controller           │
│   • SideCart.open()         • SideCart.fetchCart()          │
│   • SideCart.close()        • SideCart.render(cartData)     │
└──────────────┬──────────────────────────────┬───────────────┘
               │                              │
     ┌─────────┴────────┐            ┌────────┴─────────┐
     ▼                  ▼            ▼                  ▼
[Header Cart Icon] [Red Badge]  [Sidecart Drawer] [Product Form]
- Click to open    - Live count - Qty, Trash, Total - AJAX Submit
```

---

## ধাপ ১: জাভাস্ক্রিপ্ট কন্ট্রোলার তৈরি (`assets/application.js`)

এই জাভাস্ক্রিপ্ট অবজেক্টটি পুরো সাইডকার্টের লজিক নিয়ন্ত্রণ করে:
- Tailwind CSS ক্লাসের মাধ্যমে ড্রয়ার ওপেন এবং ক্লোজ অ্যানিমেশন
- Shopify AJAX Cart API (`/cart.js`, `/cart/add.js`, `/cart/change.js`, `/cart/clear.js`)
- কোয়ান্টিটি বাড়ানো/কমানো, রিমুভ ও ক্লিয়ার কার্ট
- ড্রয়ারের ভেতরের কন্টেন্ট এবং হেডারের লাল ব্যাজ আপডেট

### ফাইল পাথ: `assets/application.js`
আপনার `assets/application.js` ফাইলে নিচের সম্পূর্ণ ভ্যানিলা জাভাস্ক্রিপ্ট কোডটি পেস্ট করুন:

```javascript
/**
 * Vanilla JavaScript Sidecart Controller
 * For Elizabeth Shopify Theme (No Alpine.js)
 */
const SideCart = {
  drawer: null,
  overlay: null,
  itemsContainer: null,
  subtotalEl: null,
  headerBadge: null,
  mobileBadge: null,
  itemCountEl: null,
  footerEl: null,

  init() {
    // DOM এলিমেন্টগুলো ক্যাশ করা
    this.drawer = document.getElementById('side-cart-drawer');
    this.overlay = document.getElementById('side-cart-overlay');
    this.itemsContainer = document.getElementById('side-cart-items');
    this.subtotalEl = document.getElementById('side-cart-subtotal');
    this.headerBadge = document.getElementById('header-cart-count');
    this.mobileBadge = document.getElementById('mobile-cart-count');
    this.itemCountEl = document.getElementById('side-cart-item-count');
    this.footerEl = document.getElementById('side-cart-footer');

    this.bindEvents();
  },

  bindEvents() {
    // ১. হেডারের কার্ট বাটনে ক্লিক করলে ড্রয়ার ওপেন
    document.querySelectorAll('.open-side-cart-btn').forEach(btn => {
      btn.addEventListener('click', (e) => {
        e.preventDefault();
        this.open();
      });
    });

    // ২. ড্রয়ার ক্লোজ বাটন এবং ডার্ক ওভারলে ক্লিকে ড্রয়ার বন্ধ
    const closeBtn = document.getElementById('side-cart-close');
    if (closeBtn) {
      closeBtn.addEventListener('click', () => this.close());
    }
    if (this.overlay) {
      this.overlay.addEventListener('click', () => this.close());
    }

    // ৩. কীবোর্ডের Escape কী চাপলে ড্রয়ার বন্ধ
    document.addEventListener('keydown', (e) => {
      if (e.key === 'Escape' && this.isOpen()) {
        this.close();
      }
    });

    // ৪. ড্রয়ারের ভেতরের বাটনগুলোর ইভেন্ট ডেলিগেশন (+, -, Trash)
    if (this.itemsContainer) {
      this.itemsContainer.addEventListener('click', (e) => {
        const target = e.target.closest('[data-cart-action]');
        if (!target) return;

        const action = target.dataset.cartAction;
        const key = target.dataset.key;
        const currentQty = parseInt(target.dataset.qty, 10);

        if (action === 'increase') {
          this.changeQuantity(key, currentQty + 1);
        } else if (action === 'decrease') {
          this.changeQuantity(key, currentQty - 1);
        } else if (action === 'remove') {
          this.changeQuantity(key, 0);
        }
      });
    }

    // ৫. Clear Cart বাটন হ্যান্ডলার
    const clearBtn = document.getElementById('side-cart-clear-btn');
    if (clearBtn) {
      clearBtn.addEventListener('click', () => this.clearCart());
    }

    // ৬. প্রোডাক্ট পেজের Add to Cart ফর্ম ইন্টারসেপ্ট করা
    const productForm = document.querySelector('#product-form');
    if (productForm) {
      productForm.addEventListener('submit', (e) => {
        e.preventDefault();
        this.addToCart(productForm);
      });
    }
  },

  isOpen() {
    return this.drawer && !this.drawer.classList.contains('translate-x-full');
  },

  // সাইডকার্ট ওপেন
  open() {
    if (!this.drawer || !this.overlay) return;
    
    // ব্যাকড্রপ দৃশ্যমান করা
    this.overlay.classList.remove('opacity-0', 'pointer-events-none');
    this.overlay.classList.add('opacity-100', 'pointer-events-auto');

    // ড্রয়ার স্লাইড ইন করা
    this.drawer.classList.remove('translate-x-full');
    this.drawer.classList.add('translate-x-0');

    document.body.classList.add('overflow-hidden');
    this.fetchCart();
  },

  // সাইডকার্ট ক্লোজ
  close() {
    if (!this.drawer || !this.overlay) return;

    // ব্যাকড্রপ হাইড করা
    this.overlay.classList.remove('opacity-100', 'pointer-events-auto');
    this.overlay.classList.add('opacity-0', 'pointer-events-none');

    // ড্রয়ার স্লাইড আউট করা
    this.drawer.classList.remove('translate-x-0');
    this.drawer.classList.add('translate-x-full');

    document.body.classList.remove('overflow-hidden');
  },

  // Shopify AJAX Cart ফেচ করা
  async fetchCart() {
    try {
      const res = await fetch('/cart.js');
      const cart = await res.json();
      this.render(cart);
    } catch (err) {
      console.error('Error fetching cart:', err);
    }
  },

  // প্রোডাক্ট পেজ থেকে AJAX Add to Cart
  async addToCart(form) {
    const submitBtn = form.querySelector('[type="submit"]');
    const originalText = submitBtn ? submitBtn.innerText : '';
    if (submitBtn) {
      submitBtn.disabled = true;
      submitBtn.innerText = 'Adding to cart...';
    }

    const formData = new FormData(form);

    try {
      const res = await fetch('/cart/add.js', {
        method: 'POST',
        body: formData
      });

      if (res.ok) {
        await this.fetchCart();
        this.open();
      } else {
        const errorData = await res.json();
        alert(errorData.description || 'Could not add item to cart.');
      }
    } catch (err) {
      console.error('Add to cart failed:', err);
    } finally {
      if (submitBtn) {
        submitBtn.disabled = false;
        submitBtn.innerText = originalText;
      }
    }
  },

  // কোয়ান্টিটি বৃদ্ধি/হ্রাস/রিমুভ
  async changeQuantity(key, quantity) {
    try {
      const res = await fetch('/cart/change.js', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'Accept': 'application/json'
        },
        body: JSON.stringify({ id: key, quantity: quantity })
      });
      const cart = await res.json();
      this.render(cart);
    } catch (err) {
      console.error('Change quantity failed:', err);
    }
  },

  // সম্পূর্ণ কার্ট খালি করা
  async clearCart() {
    try {
      const res = await fetch('/cart/clear.js', {
        method: 'POST',
        headers: { 'Accept': 'application/json' }
      });
      const cart = await res.json();
      this.render(cart);
    } catch (err) {
      console.error('Clear cart failed:', err);
    }
  },

  // ড্রয়ার এবং হেডারের ব্যাজ রেন্ডার করা
  render(cart) {
    // ১. হেডারের লাল ব্যাজ আপডেট
    this.updateBadges(cart.item_count);

    // ২. ড্রয়ার হেডারের আইটেম কাউন্ট
    if (this.itemCountEl) {
      this.itemCountEl.innerText = `${cart.item_count} items`;
    }

    // ৩. ফুটার শো/হাইড এবং সাবটোটাল
    if (this.footerEl) {
      if (cart.item_count > 0) {
        this.footerEl.classList.remove('hidden');
      } else {
        this.footerEl.classList.add('hidden');
      }
    }
    if (this.subtotalEl) {
      this.subtotalEl.innerText = this.formatMoney(cart.total_price);
    }

    // ৪. আইটেম লিস্ট তৈরি
    if (!this.itemsContainer) return;

    if (cart.items.length === 0) {
      this.itemsContainer.innerHTML = `
        <div class="text-center py-16">
          <svg class="mx-auto h-12 w-12 text-gray-300" fill="none" viewBox="0 0 24 24" stroke="currentColor">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1" d="M16 11V7a4 4 0 00-8 0v4M5 9h14l1 12H4L5 9z" />
          </svg>
          <p class="mt-4 text-base font-medium text-gray-900">Your cart is currently empty</p>
          <p class="mt-1 text-sm text-gray-500">Add products to your cart to see them here.</p>
          <button 
            type="button" 
            onclick="SideCart.close()" 
            class="mt-6 inline-flex items-center px-4 py-2 border border-transparent text-sm font-medium rounded-md shadow-sm text-white bg-black hover:bg-gray-800"
          >
            Continue Shopping
          </button>
        </div>
      `;
      return;
    }

    let itemsHtml = '';
    cart.items.forEach(item => {
      const variantTitle = (item.variant_title && item.variant_title !== 'Default Title')
        ? `<p class="text-xs text-gray-400 uppercase tracking-wider mt-0.5">${item.variant_title}</p>`
        : '';

      const imageUrl = item.image || (item.featured_image ? item.featured_image.url : '');

      itemsHtml += `
        <div class="border border-gray-200 rounded-lg p-3 bg-white shadow-sm flex items-start space-x-3">
          <!-- Thumbnail -->
          <div class="h-20 w-20 flex-shrink-0 overflow-hidden rounded bg-gray-50 border border-gray-100">
            <img src="${imageUrl}" alt="${item.product_title}" class="h-full w-full object-cover object-center">
          </div>

          <!-- Info & Controls -->
          <div class="flex-1 min-w-0">
            <div class="flex justify-between items-start">
              <div>
                <h3 class="text-sm font-bold text-gray-900 uppercase tracking-tight truncate max-w-[170px]">${item.product_title}</h3>
                ${variantTitle}
              </div>
              <span class="text-sm font-semibold text-gray-900 ml-2">${this.formatMoney(item.final_line_price)}</span>
            </div>

            <!-- Quantity & Delete -->
            <div class="flex items-center justify-between mt-3">
              <div class="inline-flex items-center border border-gray-300 rounded">
                <button 
                  type="button" 
                  data-cart-action="decrease" 
                  data-key="${item.key}" 
                  data-qty="${item.quantity}"
                  class="px-2.5 py-1 text-gray-500 hover:text-gray-700 hover:bg-gray-100 transition text-sm focus:outline-none"
                  aria-label="Decrease quantity"
                >−</button>
                <span class="px-3 py-1 text-xs font-semibold text-gray-900">${item.quantity}</span>
                <button 
                  type="button" 
                  data-cart-action="increase" 
                  data-key="${item.key}" 
                  data-qty="${item.quantity}"
                  class="px-2.5 py-1 text-gray-500 hover:text-gray-700 hover:bg-gray-100 transition text-sm focus:outline-none"
                  aria-label="Increase quantity"
                >+</button>
              </div>

              <!-- Trash Icon -->
              <button 
                type="button" 
                data-cart-action="remove" 
                data-key="${item.key}"
                class="text-red-500 hover:text-red-700 p-1 transition focus:outline-none"
                aria-label="Remove item"
              >
                <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.5" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16" />
                </svg>
              </button>
            </div>
          </div>
        </div>
      `;
    });

    this.itemsContainer.innerHTML = itemsHtml;
  },

  updateBadges(count) {
    const badges = [this.headerBadge, this.mobileBadge];
    badges.forEach(badge => {
      if (!badge) return;
      badge.innerText = count;
      if (count > 0) {
        badge.classList.remove('hidden');
        badge.classList.add('flex');
      } else {
        badge.classList.remove('flex');
        badge.classList.add('hidden');
      }
    });
  },

  formatMoney(cents) {
    if (cents == null || isNaN(cents)) return 'Tk 0.00';
    return `Tk ${(cents / 100).toFixed(2)}`;
  }
};

// DOM তৈরি হলে কন্ট্রোলার চালু করা
document.addEventListener('DOMContentLoaded', () => {
  SideCart.init();
});
```

---

## ধাপ ২: সাইডকার্ট ড্রয়ার স্নিপেট তৈরি (`snippets/side-cart.liquid`)

এই স্নিপেটটি সম্পূর্ণ ড্রয়ারের এইচটিএমএল এবং Tailwind CSS স্ট্রাকচার বহন করবে। এতে কোনো Alpine ডিরেক্টিভ নেই।

### ফাইল পাথ তৈরি করুন: `snippets/side-cart.liquid`
নিচের কোডটি দিয়ে ফাইলটি সেভ করুন:

```liquid
<!-- Sidecart Container -->
<div id="side-cart-wrapper" class="relative z-50">
  
  <!-- ১. ব্যাকড্রপ ওভারলে (Dark Dimmed Overlay) -->
  <div 
    id="side-cart-overlay" 
    class="fixed inset-0 bg-gray-500/75 transition-opacity duration-300 opacity-0 pointer-events-none"
    aria-hidden="true"
  ></div>

  <!-- ২. সাইডকার্ট ড্রয়ার কন্টেইনার -->
  <div class="fixed inset-0 overflow-hidden pointer-events-none">
    <div class="absolute inset-0 overflow-hidden">
      <div class="pointer-events-none fixed inset-y-0 right-0 flex max-w-full pl-10">
        
        <!-- স্লাইড ওভার প্যানেল (White Drawer Panel) -->
        <div 
          id="side-cart-drawer" 
          class="pointer-events-auto w-screen max-w-md bg-white shadow-2xl flex flex-col transform translate-x-full transition-transform duration-300 ease-in-out"
        >
          
          <!-- ড্রয়ার হেডার (Your cart, 2 items, Close X) -->
          <div class="flex items-center justify-between px-6 py-4 border-b border-gray-100">
            <h2 class="text-xl font-bold text-gray-900">Your cart</h2>
            
            <div class="flex items-center space-x-3">
              <span id="side-cart-item-count" class="text-sm text-gray-500">
                {{ cart.item_count }} items
              </span>
              <button 
                type="button" 
                id="side-cart-close" 
                class="text-gray-400 hover:text-gray-600 p-1 focus:outline-none"
                aria-label="Close cart"
              >
                <!-- Close X Icon -->
                <svg class="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.5" d="M6 18L18 6M6 6l12 12" />
                </svg>
              </button>
            </div>
          </div>

          <!-- ড্রয়ার আইটেম এরিয়া (Scrollable Items List) -->
          <div id="side-cart-items" class="flex-1 overflow-y-auto px-6 py-4 space-y-4">
            {% if cart.item_count == 0 %}
              <div class="text-center py-16">
                <svg class="mx-auto h-12 w-12 text-gray-300" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1" d="M16 11V7a4 4 0 00-8 0v4M5 9h14l1 12H4L5 9z" />
                </svg>
                <p class="mt-4 text-base font-medium text-gray-900">Your cart is currently empty</p>
                <p class="mt-1 text-sm text-gray-500">Add products to your cart to see them here.</p>
                <button 
                  type="button" 
                  onclick="SideCart.close()" 
                  class="mt-6 inline-flex items-center px-4 py-2 border border-transparent text-sm font-medium rounded-md shadow-sm text-white bg-black hover:bg-gray-800"
                >
                  Continue Shopping
                </button>
              </div>
            {% else %}
              {% for item in cart.items %}
                <div class="border border-gray-200 rounded-lg p-3 bg-white shadow-sm flex items-start space-x-3">
                  <!-- Thumbnail -->
                  <div class="h-20 w-20 flex-shrink-0 overflow-hidden rounded bg-gray-50 border border-gray-100">
                    {% if item.image %}
                      <img src="{{ item.image | img_url: 'medium' }}" alt="{{ item.title | escape }}" class="h-full w-full object-cover object-center">
                    {% else %}
                      {{ 'product-1' | placeholder_svg_tag: 'h-full w-full object-cover object-center' }}
                    {% endif %}
                  </div>

                  <!-- Info & Controls -->
                  <div class="flex-1 min-w-0">
                    <div class="flex justify-between items-start">
                      <div>
                        <h3 class="text-sm font-bold text-gray-900 uppercase tracking-tight truncate max-w-[170px]">{{ item.product.title }}</h3>
                        {% unless item.variant.title contains 'Default' %}
                          <p class="text-xs text-gray-400 uppercase tracking-wider mt-0.5">{{ item.variant.title }}</p>
                        {% endunless %}
                      </div>
                      <span class="text-sm font-semibold text-gray-900 ml-2">{{ item.final_line_price | money }}</span>
                    </div>

                    <!-- Quantity & Delete -->
                    <div class="flex items-center justify-between mt-3">
                      <div class="inline-flex items-center border border-gray-300 rounded">
                        <button 
                          type="button" 
                          data-cart-action="decrease" 
                          data-key="{{ item.key }}" 
                          data-qty="{{ item.quantity }}"
                          class="px-2.5 py-1 text-gray-500 hover:text-gray-700 hover:bg-gray-100 transition text-sm focus:outline-none"
                          aria-label="Decrease quantity"
                        >−</button>
                        <span class="px-3 py-1 text-xs font-semibold text-gray-900">{{ item.quantity }}</span>
                        <button 
                          type="button" 
                          data-cart-action="increase" 
                          data-key="{{ item.key }}" 
                          data-qty="{{ item.quantity }}"
                          class="px-2.5 py-1 text-gray-500 hover:text-gray-700 hover:bg-gray-100 transition text-sm focus:outline-none"
                          aria-label="Increase quantity"
                        >+</button>
                      </div>

                      <!-- Trash Icon -->
                      <button 
                        type="button" 
                        data-cart-action="remove" 
                        data-key="{{ item.key }}"
                        class="text-red-500 hover:text-red-700 p-1 transition focus:outline-none"
                        aria-label="Remove item"
                      >
                        <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.5" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16" />
                        </svg>
                      </button>
                    </div>
                  </div>
                </div>
              {% endfor %}
            {% endif %}
          </div>

          <!-- ড্রয়ার ফুটার (Clear Cart, Subtotal, View Cart, Checkout Buttons) -->
          <div 
            id="side-cart-footer" 
            class="border-t border-gray-200 px-6 py-5 bg-white space-y-4 {% if cart.item_count == 0 %}hidden{% endif %}"
          >
            <!-- Clear Cart Link -->
            <div class="flex justify-end">
              <button 
                type="button" 
                id="side-cart-clear-btn" 
                class="text-xs font-semibold uppercase tracking-wider text-gray-900 hover:text-red-600 underline focus:outline-none"
              >
                Clear Cart
              </button>
            </div>

            <!-- Subtotal -->
            <div class="flex justify-end items-baseline space-x-2">
              <span class="text-sm font-medium text-gray-600">Subtotal:</span>
              <span id="side-cart-subtotal" class="text-xl font-bold text-gray-900">
                {{ cart.total_price | money }}
              </span>
            </div>

            <!-- View Cart Button (White with black border) -->
            <div>
              <a 
                href="{{ routes.cart_url }}" 
                class="w-full flex items-center justify-center px-6 py-3 border border-black text-sm font-bold uppercase tracking-wider text-black bg-white hover:bg-gray-50 transition"
              >
                View Cart
              </a>
            </div>

            <!-- Checkout Button (Black button) -->
            <div>
              <a 
                href="/checkout" 
                class="w-full flex items-center justify-center px-6 py-3 border border-transparent text-sm font-bold uppercase tracking-wider text-white bg-black hover:bg-gray-800 transition"
              >
                Checkout
              </a>
            </div>
          </div>

        </div>
      </div>
    </div>
  </div>
</div>
```

---

## ধাপ ৩: লেআউটে স্নিপেট যুক্ত করা (`layout/theme.liquid`)

### ফাইল পাথ: `layout/theme.liquid`
`theme.liquid` ফাইলের শেষের দিকে `</body>` ট্যাগের ঠিক উপরে `{% render 'side-cart' %}` বসিয়ে দিন:

```liquid
    {% section 'header' %}
    <main role="main">
      {{ content_for_layout }}
    </main>

    <!-- 👉 এই লাইনটি যুক্ত করুন -->
    {% render 'side-cart' %}
  </body>
</html>
```

---

## ধাপ ৪: হেডার ন্যাভিগেশন ও লাল ব্যাজ আপডেট (`sections/header.liquid`)

হেডারের কার্ট লিঙ্কে ক্লাস `.open-side-cart-btn` এবং রেফারেন্স ইমেজের মতো লাল বৃত্তাকার ব্যাজ যোগ করুন।

### ১. ডেস্কটপ ন্যাভিগেশন:
`sections/header.liquid` ফাইলে লাইন ৬৮-৭০ এর আশেপাশে বর্তমান কোডটি খুঁজুন:

```liquid
<!-- পূর্বের কোড: -->
<a href="{{ routes.cart_url }}" class="whitespace-nowrap text-base text-gray-500 hover:text-gray-900">
  {% render 'icon-shopping-bag' %}
</a>
```

**নিচের কোডটি দিয়ে প্রতিস্থাপন করুন:**

```liquid
<!-- 👉 নতুন কোড (কার্ট বাটন + লাল নোটিফিকেশন ব্যাজ): -->
<button 
  type="button" 
  class="open-side-cart-btn relative p-1 text-gray-500 hover:text-gray-900 focus:outline-none flex items-center"
  aria-label="Open cart"
>
  {% render 'icon-shopping-bag' %}

  <!-- লাল বৃত্তাকার ব্যাজ (Red Item Count Badge) -->
  <span 
    id="header-cart-count" 
    class="absolute -top-1.5 -right-1.5 bg-red-600 text-white text-[10px] font-bold rounded-full h-4 min-w-[16px] px-1 {% if cart.item_count > 0 %}flex{% else %}hidden{% endif %} items-center justify-center leading-none shadow-sm"
  >
    {{ cart.item_count }}
  </span>
</button>
```

### ২. মোবাইল হেডারে কার্ট বাটন যুক্ত করা (ঐচ্ছিক কিন্তু বাঞ্ছনীয়):
মোবাইল ভিউতে হ্যামবার্গার মেনুর পাশে কার্ট বাটন যুক্ত করতে চাইলে (লাইন ১২-২০ এর আশেপাশে):

```liquid
<div class="flex items-center md:hidden space-x-2">
  <button 
    type="button" 
    class="open-side-cart-btn relative p-2 text-gray-500 hover:text-gray-900 focus:outline-none"
    aria-label="Open cart"
  >
    {% render 'icon-shopping-bag' %}
    <span 
      id="mobile-cart-count" 
      class="absolute top-0 right-0 bg-red-600 text-white text-[10px] font-bold rounded-full h-4 min-w-[16px] px-1 {% if cart.item_count > 0 %}flex{% else %}hidden{% endif %} items-center justify-center leading-none"
    >
      {{ cart.item_count }}
    </span>
  </button>
</div>
```

---

## ধাপ ৫: প্রোডাক্ট পেজে AJAX Add-to-Cart কানেক্ট করা (`sections/template-product.liquid`)

### ফাইল পাথ: `sections/template-product.liquid`
নিশ্চিত করুন যে আপনার প্রোডাক্ট ফর্মটির `id="product-form"` সেট করা আছে (লাইন ৫১ এ এটি ইতিমধ্যেই আছে):

```liquid
{% form 'product', product, id: 'product-form', novalidate: 'novalidate' %}
```

আমাদের `assets/application.js`-এর ভেতরে থাকা ইভেন্ট লিসেনারটি স্বয়ংক্রিয়ভাবে `#product-form`-এর সাবমিশন ইন্টারসেপ্ট করে AJAX-এর মাধ্যমে কার্টে যোগ করবে এবং ড্রয়ারটি ওপেন করবে:
```javascript
const productForm = document.querySelector('#product-form');
if (productForm) {
  productForm.addEventListener('submit', (e) => {
    e.preventDefault();
    this.addToCart(productForm);
  });
}
```

> **অতিরিক্ত সুবিধা:** বাটনে চাপ দিলে বাটনটি সাময়িকভাবে ডিজেবল হয়ে **"Adding to cart..."** টেক্সট দেখাবে, এবং ড্রয়ার ওপেন হওয়ার সাথে সাথে বাটনটি পুনরায় স্বাভাবিক হয়ে যাবে।

---

## ধাপ ৬: টেস্টিং ও ভেরিফিকেশন চেকলিস্ট (Testing Checklist)

কোড সেভ করার পর নিচের ধাপগুলো পরীক্ষা করুন:

- [ ] **১. হেডার কার্ট আইকন ক্লিক:** যেকোনো পেজে টপ ন্যাভিগেশনের শপিং ব্যাগ আইকনে ক্লিক করুন। ড্রয়ারটি ডানদিক থেকে স্লাইড হয়ে বের হচ্ছে কিনা এবং ব্যাকগ্রাউন্ডে ডার্ক ওভারলে আসছে কিনা দেখুন।
- [ ] **২. ক্লোজ বাটন ও ব্যাকড্রপ:** উপরে ডানদিকের 'X' বাটনে বা ডার্ক ব্যাকড্রপে ক্লিক করলে ড্রয়ারটি বন্ধ হয় কিনা দেখুন।
- [ ] **৩. Escape কী প্রেস:** কীবোর্ডের `Esc` চাপ দিলে ড্রয়ার বন্ধ হচ্ছে কিনা পরীক্ষা করুন।
- [ ] **৪. প্রোডাক্ট পেজ Add-to-Cart:** প্রোডাক্ট পেজে গিয়ে "Add to cart" বাটনে ক্লিক করুন। পেজ রিলোড না হয়ে সরাসরি ড্রয়ার ওপেন হচ্ছে এবং সদ্য যোগ করা আইটেমটি ড্রয়ারে দেখা যাচ্ছে কিনা দেখুন।
- [ ] **৫. হেডার লাল ব্যাজ:** প্রোডাক্ট যুক্ত করার পর হেডারের লাল বৃত্তাকার ব্যাজ স্বয়ংক্রিয়ভাবে আপডেটেড সংখ্যা (যেমন: 1, 2) দেখাচ্ছে কিনা দেখুন।
- [ ] **৬. ড্রয়ারে কোয়ান্টিটি বৃদ্ধি (+):** `+` বাটনে ক্লিক করলে কোয়ান্টিটি ও লাইন প্রাইস বাড়ছে কিনা দেখুন।
- [ ] **৭. ড্রয়ারে কোয়ান্টিটি হ্রাস (−):** `−` বাটনে ক্লিক করলে কোয়ান্টিটি কমছে কিনা দেখুন।
- [ ] **৮. আইটেম রিমুভ (Trash icon):** লাল ডাস্টবিন আইকনে ক্লিক করলে সাথে সাথে আইটেমটি ড্রয়ার থেকে ডিলিট হয়ে যাচ্ছে কিনা দেখুন।
- [ ] **৯. Clear Cart লিংক:** "CLEAR CART" এ ক্লিক করলে সব আইটেম মুছে গিয়ে "Your cart is currently empty" বার্তা দেখাচ্ছে কিনা এবং ব্যাজ লুকানো হচ্ছে কিনা দেখুন।

---

## সাধারণ সমস্যা ও সমাধান (Troubleshooting & FAQs)

### ১. ড্রয়ারটি খোলার সময় মসৃণভাবে স্লাইড হচ্ছে না কেন?
**সমাধান:** Tailwind CSS ট্রানজিশন ক্লাসগুলো (`transition-transform duration-300 ease-in-out` এবং `translate-x-full`) সঠিকভাবে `snippets/side-cart.liquid`-এর `#side-cart-drawer`-এ আছে কিনা চেক করুন।

### ২. কারেন্সিতে "Tk" এর জায়গায় অন্য সিম্বল কীভাবে দেব?
**সমাধান:** `assets/application.js`-এর একদম নিচে `formatMoney(cents)` ফাংশনে `Tk ${(cents / 100).toFixed(2)}`-এর জায়গায় আপনার কাঙ্ক্ষিত কারেন্সি সিম্বল বসিয়ে দিন।

### ৩. কনসোলে কোনো এরর আসছে কিনা কীভাবে বুঝব?
**সমাধান:** ব্রাউজারে `F12` চেপে `Console` ট্যাবে যান। কোনো ফাইল লোড হতে মিসিং হলে বা কোনো ভুল থাকলে সেখানে লাল এরর মেসেজ দেখতে পাবেন।
