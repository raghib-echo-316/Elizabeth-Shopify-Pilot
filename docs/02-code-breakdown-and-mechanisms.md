# 🔍 কোড ব্রেকডাউন: মেকানিজম, অবজেক্ট ও ফিল্টার সমাচার

> **উদ্দেশ্য:** এই ডকুমেন্টে আমাদের Elizabeth থিমে ব্যবহৃত প্রতিটি প্রধান ফাইলের কাজের পেছনের মেকানিজম এবং কোডে ব্যবহৃত Shopify Objects, Tags ও Filters-কে সহজ বাংলায় ব্যাখ্যা করা হয়েছে, যেন যে কেউ কোডের প্রতি লাইনের কার্যকারিতা সহজে বুঝতে পারে।

---

## ১. মাস্টার লেআউট (`layout/theme.liquid`)

যেকোনো শপিফাই থিমের ভিত্তি হলো `theme.liquid`। এটি এমন একটি ফ্রেম যার ভেতরে সব পেজ রেন্ডার হয়।

```liquid
<!doctype html>
<html>
  <head>
    <title>{{ page_title }}</title>
    <meta name="description" content="{{ page_description | escape }}">
    <link rel="canonical" href="{{ canonical_url }}">
    
    {{ content_for_header }} <!-- অতি গুরুত্বপূর্ণ -->
    
    {{ 'application.css' | asset_url | stylesheet_tag }}
    <script src="{{ 'application.js' | asset_url }}" defer></script>
    <script defer src="https://unpkg.com/alpinejs@3.x.x/dist/cdn.min.js"></script>
  </head>
  <body class="min-h-screen">
    {% section 'header' %}
    <main role="main">
      {{ content_for_layout }} <!-- পেজের কন্টেন্ট এখানে লোড হয় -->
    </main>
  </body>
</html>
```

### 💡 এখানে ব্যবহৃত গুরুত্বপূর্ণ মেকানিজম ও ফিল্টার:
* **`page_title`**: বর্তমান পেজের নাম স্বয়ংক্রিয়ভাবে দেখায় (যেমন: হোমপেজের নাম, বা প্রোডাক্টের নাম)।
* **`page_description | escape`**: পেজের মেটা ডেসক্রিপশন প্রিন্ট করে এবং `escape` ফিল্টার HTML ক্যারেক্টার সমস্যা দূর করে।
* **`canonical_url`**: সার্চ ইঞ্জিনের ডুপ্লিকেট কন্টেন্ট সমস্যা রোধ করতে পেজের মূল URL প্রদান করে।
* **`{{ content_for_header }}`**: শপিফাইয়ের নিজস্ব স্ক্রিপ্ট, থিম প্রিভিউ বার এবং থার্ড-পার্টি অ্যাপের স্ক্রিপ্ট ইনজেক্ট করার জন্য এটি **বাধ্যতামূলক**।
* **`asset_url | stylesheet_tag`**: `application.css` ফাইলটির CDN লিংক জেনারেট করে শপিফাইয়ের উপযুক্ত `<link rel="stylesheet">` ট্যাগ তৈরি করে।
* **`{{ content_for_layout }}`**: ভিজিটর যে পেজে আছেন (হোমপেজ, প্রোডাক্ট বা কার্ট), সেই পেজের টেমপ্লেট এই ট্যাগের জায়গায় এসে বসে।

---

## ২. গ্লোবাল হেডার ও নেভিগেশন (`sections/header.liquid`)

স্টোরের হেডার, ডেস্কটপ নেভিগেশন ড্রপডাউন এবং মোবাইল সাইড মেনু এখানে রয়েছে।

### ক. লিকুইড কোড মেকানিজম:
```liquid
{% assign main_menu_linklist = linklists[section.settings.menu].links %}

<nav class="hidden md:flex space-x-10">
  {% for link in main_menu_linklist %}
    {% if link.links != blank %}
      <!-- সাবমেনু থাকলে ড্রপডাউন বাটন -->
      <div class="relative" x-data="{ open: false }">
        <button x-on:click="open = !open" type="button">
          {{ link.title }}
          {% render 'icon-dropdown' %}
        </button>
        <div x-show="open" @click.away="open = false">
          {% for childlink in link.links %}
            <a href="{{ childlink.url }}">{{ childlink.title }}</a>
          {% endfor %}
        </div>
      </div>
    {% else %}
      <!-- সাধারণ লিঙ্ক -->
      <a href="{{ link.url }}">{{ link.title }}</a>
    {% endif %}
  {% endfor %}
</nav>
```

### 💡 অবজেক্ট ও কনসেপ্ট:
1. **`section.settings.menu`**: সেকশন স্কিমা থেকে অ্যাডমিনের পছন্দ করা মেনুর হ্যান্ডেল (যেমন: `main-menu`)।
2. **`linklists[...]`**: শপিফাইয়ের সমস্ত নেভিগেশন মেনুর গ্লোবাল অবজেক্ট।
3. **`link.links != blank`**: মেনু আইটেমের কোনো চাইল্ড লিঙ্ক (ড্রপডাউন সাব-মেনু) আছে কিনা পরীক্ষা করে।
4. **`routes.cart_url`**: কোনো হার্ডকোডেড `/cart` লিঙ্কের বদলে ডাইনামিক রুট অবজেক্ট।
5. **`shop.customer_accounts_enabled`**: স্টোরে কাস্টমার একাউন্ট অন করা আছে কিনা তা চেক করে লগইন/রেজিস্টার লিংক দেখায়।
6. **Alpine.js (`x-data`, `x-show`, `@click.away`)**: জাভাস্ক্রিপ্টে ক্লাস টগল করার ঝামেলা ছাড়া এক লাইনে ড্রপডাউন খোলা এবং বাইরে ক্লিক করলে বন্ধ হওয়ার লজিক।

---

## ৩. প্রোডাক্ট পেজ ডিপ ডাইভ (`sections/template-product.liquid`)

এটি থিমের সবচেয়ে বড় ও পাওয়ারফুল সেকশন।

### ক. ডায়নামিক ব্লক আর্কিটেকচার (`section.blocks`):
```liquid
{% for block in section.blocks %}
  {% case block.type %}
    {% when 'title' %}
      <h1>{{ product.title }}</h1>
    {% when 'price' %}
      <div id="price-{{ section.id }}">
        <span class="line-through">{{ selected_variant.compare_at_price | money }}</span>
        <span>{{ selected_variant.price | money }}</span>
      </div>
    {% when 'variant_selector' %}
      ...
    {% when 'checkout_buttons' %}
      <button type="submit" name="add">Add to cart</button>
  {% endcase %}
{% endfor %}
```
> **কেন এই পদ্ধতি?** শপিফাই অ্যাডমিন কাস্টমাইজারে ক্লায়েন্ট যেন টাইটেলের নিচে প্রাইস না রেখে ইচ্ছে করলে আগে ভেন্ডর বা কোয়ান্টিটি ড্র্যাগ করে সাজাতে পারে।

### খ. ভ্যারিয়েন্ট সিলেকশন ও সেকশন রেন্ডারিং API:
প্রোডাক্ট পেজে একাধিক সাইজ বা কালার সিলেক্ট করলে পেজ রিলোড না হয়ে ডাটা আপডেট করতে একটি কাস্টম HTML এলিমেন্ট ব্যবহার করা হয়েছে:
```javascript
class VariantSelector extends HTMLElement {
  constructor() {
    super();
    this.addEventListener("change", this.onVariantChange);
  }

  onVariantChange() {
    this.getSelectedOptions();
    this.getSelectedVariant();

    if (this.currentVariant) {
      this.updateURL();     // ব্রাউজারের URL পরিবর্তন করে ?variant=12345
      this.updateFormID();  // হিডেন ইনপুটে ভ্যারিয়েন্ট ID বসায়
      this.updatePrice();   // Section Rendering API দিয়ে প্রাইস আপডেট করে
    }
  }

  updatePrice() {
    // শপিফাই সার্ভার থেকে পুরো পেজ নয়, শুধু এই সেকশনের ফ্রেশ HTML আনা হচ্ছে
    fetch(`${this.dataset.url}?variant=${this.currentVariant.id}&section_id=${this.dataset.section}`)
      .then(res => res.text())
      .then(htmlText => {
        const id = `price-${this.dataset.section}`;
        const html = new DOMParser().parseFromString(htmlText, 'text/html');
        document.getElementById(id).innerHTML = html.getElementById(id).innerHTML;
      });
  }
}
customElements.define("variant-selector", VariantSelector);
```

### 💡 অবজেক্ট ও ফিল্টার:
* **`product.selected_or_first_available_variant`**: পেজ লোড হওয়ার সময় ইউজার যে ভ্যারিয়েন্ট সিলেক্ট করে এসেছে, অথবা স্টকের প্রথম অ্যাভেইলেবল ভ্যারিয়েন্ট এনে দেয়।
* **`product.variants | json`**: সমস্ত ভ্যারিয়েন্টের ডেটা JSON আকারে পেজে প্রিন্ট করে জাভাস্ক্রিপ্টকে দেয়।
* **`money`**: সেন্ট বা পয়সার সংখ্যাকে সঠিক কারেন্সি চিহ্নে রূপান্তর করে (যেমন: `2000` রূপান্তর হয়ে হবে `$20.00`)।
* **`product.media`**: ইমেজ ছাড়াও ভিডিও ও 3D মডেল অবজেক্ট হ্যান্ডেল করে।

---

## ৪. শপিং কার্ট পেজ (`sections/template-cart.liquid`)

কার্ট পেজ ব্যবহারকারীর নির্বাচিত পণ্যগুলো টেবিল আকারে দেখায় এবং পরিমাণ পরিবর্তন বা চেকআউটে যাওয়ার সুযোগ দেয়।

```liquid
{% if cart.item_count > 0 %}
  <form action="{{ routes.cart_url }}" method="POST">
    {% for item in cart.items %}
      <img src="{{ item.image | img_url: 'small' }}">
      <a href="{{ item.url }}">{{ item.product.title | truncatewords: 5 }}</a>
      
      <!-- কোয়ান্টিটি ইনপুট অ্যারে -->
      <input type="number" name="updates[]" value="{{ item.quantity }}">
      
      <!-- রিমুভ বাটন -->
      <a href="{{ routes.cart_change_url }}?line={{ forloop.index }}&quantity=0">Remove</a>
      
      <span>{{ item.line_price | money }}</span>
    {% endfor %}

    <!-- সাবটোটাল ও বাটন -->
    <h3>{{ cart.total_price | money }}</h3>
    <button type="submit" name="update">Update</button>
    <button type="submit" name="checkout">Checkout</button>
  </form>
{% else %}
  <h1>Your cart is empty</h1>
{% endif %}
```

### 💡 গুরুত্বপূর্ণ মেকানিজম:
1. **`name="updates[]"`**: শপিফাই এই বিশেষ ইনপুট অ্যারে দিয়ে প্রতিটি লাইনের কোয়ান্টিটি এক ক্লিকে আপডেট করে নেয়।
2. **`routes.cart_change_url`**: `?line={{ forloop.index }}&quantity=0` পাঠিয়ে দিলে নির্দিষ্ট লাইনের আইটেম কার্ট থেকে মুছে যায়।
3. **`truncatewords: 5`**: বড় প্রোডাক্ট টাইটেলকে সর্বোচ্চ ৫টি শব্দে কেটে বাকি অংশে `...` জুড়ে দেয়।
4. **`item.line_price`**: ওই নির্দিষ্ট প্রোডাক্টটির মোট দাম (`quantity * unit price`)।
5. **`cart.total_price`**: সম্পূর্ণ কার্টের সর্বমোট মূল্য।

---

## ৫. কালেকশন পেজ ও পেজিনেশন (`sections/template-collection.liquid` & `snippets/pagination.liquid`)

একটি নির্দিষ্ট ক্যাটাগরির সমস্ত প্রোডাক্ট গ্রিড আকারে সাজানো।

```liquid
{% paginate collection.products by section.settings.number_of_products_per_page %}
  <h1>{{ collection.title }}</h1>
  <div class="grid grid-cols-1 md:grid-cols-3">
    {% for product in collection.products %}
      {% render 'product-card', product_product: product %}
    {% else %}
      <p>There are no products inside this collection</p>
    {% endfor %}
  </div>
  
  {% render 'pagination', pagination: paginate %}
{% endpaginate %}
```

### 💡 মেকানিজম:
* **`{% paginate ... by ... %}`**: শপিফাই পেজিনেশন ট্যাগ। এর ভেতরে স্বয়ংক্রিয়ভাবে একটি `paginate` অবজেক্ট তৈরি হয় যাতে `paginate.pages`, `paginate.previous.url`, `paginate.next.url` থাকে।
* **`{% for ... %} {% else %} {% endfor %}`**: যদি কালেকশনটি সম্পূর্ণ খালি থাকে, তবে `else` ব্লকের টেক্সট রেন্ডার হবে।
* **`{% render 'product-card', product_product: product %}`**: কার্ডের সম্পূর্ণ ডিজাইন `snippets/product-card.liquid`-এ রয়েছে। সেটিকে এই লুপে ভ্যারিয়েবল পাস করে রিইউজ করা হয়েছে।

---

## ৬. ব্লগ এবং কমেন্ট হ্যান্ডলিং (`sections/template-article.liquid`)

ব্লগ আর্টিকেলের কন্টেন্ট প্রদর্শন এবং ব্যবহারকারীর কমেন্ট পোস্টিং।

```liquid
{% form 'new_comment', article %}
  {% if form.errors %}
    <p>ভুল হয়েছে!</p>
    {{ form.errors.messages[error] }}
  {% elsif form.posted_successfully? %}
    <p>আপনার কমেন্টটি সফলভাবে পোস্ট হয়েছে!</p>
  {% endif %}

  <input type="text" name="comment[author]" value="{{ form.author }}">
  <input type="email" name="comment[email]" value="{{ form.email }}">
  <textarea name="comment[body]">{{ form.body }}</textarea>
  <input type="submit" value="Post comment">
{% endform %}
```

### 💡 মেকানিজম:
* **`{% form 'new_comment', article %}`**: শপিফাইয়ের নিজস্ব ফর্ম মেথড। এটি স্বয়ংক্রিয়ভাবে সঠিক POST অ্যাকশন, CSRF টোকেন এবং শপিফাই ব্যাকএন্ড হ্যান্ডলার সেট করে।
* **`form.posted_successfully?`**: কমেন্ট সফলভাবে শপিফাই ডাটাবেজে সেভ হলে ট্রু (true) রিটার্ন করে।
* **`form.errors`**: নাম বা ইমেইল ভুল থাকলে শপিফাই সার্ভার যে এরর মেসেজ দেয় তা প্রদর্শন করে।
* **`strip_html`**: আর্টিকেলের সারসংক্ষেপ দেখানোর সময় সমস্ত এইচটিএমএল ট্যাগ রিমুভ করে প্লেইন টেক্সট বানায়।

---

## ৭. কন্টাক্ট ফর্ম (`sections/contact-form.liquid`)

```liquid
{% form 'contact' %}
  <input type="text" name="contact[name]">
  <input type="email" name="contact[email]">
  <textarea name="contact[body]"></textarea>
  <input type="submit" value="Submit message">
{% endform %}
```
* কোনো সার্ভার বা পিএইচপি ব্যাকএন্ড কোড ছাড়াই `{% form 'contact' %}` ব্যবহার করলে মেসেজটি সরাসরি স্টোর ওনারের ইমেইলে চলে যায়।

---

## ৮. Online Store 2.0 (OS 2.0) JSON টেমপ্লেট মেকানিজম

আমাদের প্রজেক্টে `templates/index.json`, `product.json`, `cart.json` ইত্যাদি রয়েছে।

```json
{
  "sections": {
    "featured-collection": {
      "type": "featured-collection"
    }
  },
  "order": [
    "featured-collection"
  ]
}
```

### কেন পুরনো `.liquid` টেমপ্লেটের চেয়ে JSON টেমপ্লেট সেরা?
1. পুরনো আর্কিটেকচারে পেজের সেকশনগুলো ফিক্সড কোড করা থাকত।
2. JSON টেমপ্লেটের কারণে শপিফাই থিম কাস্টমাইজারে গিয়ে মার্চেন্ট যেকোনো সেকশন যুক্ত করতে পারেন, মুছে ফেলতে পারেন এবং মাউস দিয়ে টেনে উপরে-নিচে সরাতে পারেন (`order` অ্যারে স্বয়ংক্রিয়ভাবে আপডেট হয়)।

