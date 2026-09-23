# 🧠 শপিফাই থিম ডেভেলপমেন্ট চিটশিট (Liquid Cheat Sheet)

> **পড়ার ও মুখস্থ রাখার সহজ নির্দেশিকা:** এটি শপিফাই থিম ডেভেলপমেন্টের জন্য তৈরি করা একটি কুইক-রেফারেন্স চিটশিট। এতে এই প্রজেক্টে ব্যবহৃত সমস্ত অবজেক্ট, ফিল্টার এবং ট্যাগের পাশাপাশি ভবিষ্যতে আপনার অন্যান্য প্রজেক্টের জন্য যে বিষয়গুলো জানা আবশ্যক, সেগুলো গুছিয়ে দেওয়া হয়েছে।

---

## ১. লিকুইড কোডিংয়ের ৩টি সোনার নিয়ম (Core Syntax)

| সিনট্যাক্স | নাম | কী করে? | সহজ উদাহরণ |
|---|---|---|---|
| `{{ ... }}` | **Output** | ব্রাউজারে কোনো ডেটা প্রিন্ট বা প্রদর্শন করে | `{{ product.title }}` |
| `{% ... %}` | **Logic / Tag** | লজিক, লুপ, কন্ডিশন বা ফাংশন এক্সিকিউট করে | `{% if product.available %}` |
| `{{ ... \| ... }}` | **Filter** | কোনো আউটপুটের রূপ বা ফরম্যাট পরিবর্তন করে | `{{ product.price \| money }}` |

---

## ২. শপিফাই গ্লোবাল অবজেক্টস (Shopify Objects)

### ক. এই প্রজেক্টে ব্যবহৃত অবজেক্টসমূহ (Used in Elizabeth Project)

| অবজেক্ট | গুরুত্বপূর্ণ প্রোপার্টি (Properties) | কাজ ও ব্যবহারিক উদাহরণ |
|---|---|---|
| `shop` | `.name`, `.secure_url`, `.customer_accounts_enabled` | স্টোরের সাধারণ তথ্য।<br>`<h1>{{ shop.name }}</h1>` |
| `product` | `.title`, `.price`, `.featured_image`, `.media`, `.variants`, `.url`, `.description`, `.vendor` | নির্দিষ্ট প্রোডাক্টের যাবতীয় ডেটা।<br>`<a href="{{ product.url }}">{{ product.title }}</a>` |
| `variant` | `.id`, `.title`, `.price`, `.compare_at_price`, `.available` | প্রোডাক্টের নির্দিষ্ট সাইজ/রঙের ভ্যারিয়েন্ট।<br>`<input name="id" value="{{ variant.id }}">` |
| `cart` | `.item_count`, `.items`, `.total_price` | গ্রাহকের শপিং কার্ট।<br>`{% if cart.item_count > 0 %}` |
| `line_item` (কার্ট আইটেম) | `.image`, `.title`, `.quantity`, `.line_price`, `.product`, `.variant` | কার্টের ভেতরে থাকা প্রতি লাইনের প্রোডাক্ট।<br>`{{ item.line_price \| money }}` |
| `collection` | `.title`, `.products`, `.all_products_count`, `.featured_image`, `.url` | নির্দিষ্ট প্রোডাক্ট কালেকশন।<br>`{% for product in collection.products %}` |
| `collections` | `collections['featured-collection']` | স্টোরের সমস্ত কালেকশনের তালিকা। |
| `blog` | `.title`, `.articles`, `.comments_enabled?` | ব্লগ ক্যাটাগরি।<br>`<h1>{{ blog.title }}</h1>` |
| `article` | `.title`, `.author`, `.published_at`, `.content`, `.excerpt`, `.image`, `.comments` | ব্লগের নির্দিষ্ট আর্টিকেল।<br>`<p>{{ article.author }}</p>` |
| `comment` | `.author`, `.content`, `.created_at` | ব্লগের নিচে ইউজারের কমেন্ট। |
| `section` | `.id`, `.settings`, `.blocks` | সেকশনের ইউনিক আইডি ও কাস্টমাইজার সেটিংস।<br>`{{ section.settings.title }}` |
| `block` | `.id`, `.type`, `.settings` | সেকশনের ভেতরের ব্লক।<br>`{% case block.type %}` |
| `routes` | `.cart_url`, `.cart_change_url`, `.all_products_collection_url`, `.account_login_url` | শপিফাইয়ের ডায়নামিক লিংক।<br>`<a href="{{ routes.cart_url }}">Cart</a>` |
| `linklists` | `linklists['main-menu'].links` | স্টোরের নেভিগেশন মেনু ডেটা। |
| `paginate` | `.pages`, `.previous`, `.next`, `.current_page` | পেজিনেশনের তথ্য।<br>`{% if paginate.pages > 1 %}` |
| `form` | `.errors`, `.posted_successfully?`, `.author`, `.email`, `.body` | ফর্ম সাবমিশন ফলাফল ও ইনপুট মান। |
| `search` | `.results`, `.terms`, `.performed` | সার্চ পেজের তথ্য ও ফলাফল তালিকা। |

---

### খ. ভবিষ্যতের প্রজেক্টের জন্য সবচেয়ে দরকারি অন্যান্য অবজেক্টসমূহ (Commonly Used in Real Projects)

| অবজেক্ট | প্রোপার্টি | কেন প্রয়োজন? |
|---|---|---|
| `customer` | `.first_name`, `.email`, `.orders`, `.addresses`, `.default_address` | লগইন থাকা কাস্টমারের প্রোফাইল ও ড্যাশবোর্ড বানাতে। |
| `order` | `.name`, `.total_price`, `.line_items`, `.financial_status`, `.fulfillment_status` | কাস্টমারের অর্ডার হিস্ট্রি ও ইনভয়েস পেজ তৈরিতে। |
| `settings` | `settings.brand_color`, `settings.logo` | থিমের গ্লোবাল সেটিংস (Theme Settings) থেকে ডেটা পেতে। |
| `metafields` | `product.metafields.custom.fabric_care` | শপিফাই ডিফল্ট ফিল্ডের বাইরে ক্লায়েন্টের কাস্টম ফিল্ড দেখাতে। |
| `request` | `.page_type`, `.locale.iso_code`, `.path` | বর্তমানে ভিজিটর কোন পেজে আছেন তা বুঝতে (`request.page_type == 'product'`)। |
| `localization`| `.country`, `.language`, `.currency` | মাল্টি-কারেন্সি (ডলার, ইউরো, টাকা) এবং বহুভাষিক ড্রপডাউনের জন্য। |

---

## ৩. শপিফাই লিকুইড ফিল্টারসমূহ (Shopify Liquid Filters)

ফিল্টার মূলত ডেটাকে সুন্দরভাবে রূপান্তর করতে পাইপ চিহ্নের (`|`) পর ব্যবহার করা হয়।

### ক. এই প্রজেক্টে ব্যবহৃত ফিল্টার (Used in Elizabeth)

| ফিল্টার | কাজ ও সহজ উদাহরণ | আউটপুট রেজাল্ট |
|---|---|---|
| `money` | পয়সাকে কারেন্সি ফরম্যাটে দেখায় | `{{ 1999 \| money }}` ➔ `$19.99` |
| `image_url` | ইমেজের সাইজ নিয়ন্ত্রণ করে CDN লিংক দেয় | `{{ product.featured_image \| image_url: width: 500 }}` |
| `img_url` | ইমেজের শর্টকাট সাইজ ফিল্টার | `{{ article.image \| img_url: 'medium' }}` |
| `asset_url` | assets ফোল্ডারের ফাইলের সঠিক লিংক তৈরি করে | `{{ 'application.css' \| asset_url }}` |
| `stylesheet_tag` | CSS ফাইলের জন্য `<link>` ট্যাগ তৈরি করে | `{{ 'app.css' \| asset_url \| stylesheet_tag }}` |
| `placeholder_svg_tag` | ডামি প্রোডাক্ট বা প্লেসহোল্ডার ইমেজ তৈরি করে | `{{ 'product-1' \| placeholder_svg_tag: 'w-20' }}` |
| `strip_html` | টেক্সটের ভেতরের সব HTML ট্যাগ ফেলে দেয় | `{{ '<p>Hello</p>' \| strip_html }}` ➔ `Hello` |
| `truncatewords: 5` | সর্বোচ্চ ৫টি শব্দ রেখে বাকি অংশ `...` করে | `{{ "A quick brown fox jumps" \| truncatewords: 3 }}` ➔ `A quick brown...` |
| `truncate: 20` | অক্ষরের ভিত্তিতে দৈর্ঘ্য সংক্ষেপ করে | `{{ "Shopify Theme Development" \| truncate: 10 }}` ➔ `Shopify Th...` |
| `escape` | স্পেশাল ক্যারেক্টারকে সুরক্ষিত করে | `{{ page_description \| escape }}` |
| `date: '%Y-%m-%d'` | তারিখকে নির্দিষ্ট ফরম্যাটে সাজায় | `{{ article.published_at \| date: '%B %d, %Y' }}` |
| `json` | অবজেক্টকে JSON স্ট্রিং-এ রূপান্তর করে (JS-এর জন্য) | `{{ product.variants \| json }}` |
| `within: collection` | প্রোডাক্ট লিঙ্কের সাথে কালেকশনের পাথ যোগ করে | `{{ product.url \| within: collection }}` |
| `link_to` | টাইটেলকে স্বয়ংক্রিয় লিঙ্কে রূপান্তর করে | `{{ item.title \| link_to: item.url }}` |
| `default_errors` | শপিফাই ফর্মের এররগুলো সুন্দর লিস্ট আকারে দেখায় | `{{ form.errors \| default_errors }}` |

---

### খ. ভবিষ্যতে নিয়মিত কাজে লাগবে এমন গুরুত্বপূর্ণ ফিল্টার (Pro Cheat List)

| ফিল্টার | কাজ | উদাহরণ |
|---|---|---|
| `money_with_currency` | কারেন্সি কোডসহ দাম দেখায় | `{{ 2500 \| money_with_currency }}` ➔ `$25.00 USD` |
| `default` | ভ্যালু না থাকলে ফলব্যাক ভ্যালু দেয় | `{{ product.vendor \| default: 'Brand' }}` |
| `t` (translate) | `locales/*.json` ফাইল থেকে অনুবাদ আনে | `{{ 'general.newsletter.submit' \| t }}` |
| `handleize` / `handle` | যেকোনো স্ট্রিংকে URL ফ্রেন্ডলি স্লাগে বানায় | `{{ "T-Shirt Summer Edition" \| handleize }}` ➔ `t-shirt-summer-edition` |
| `where` | অ্যারে থেকে শর্ত মিলিয়ে ফিল্টার করে | `{% assign red_variants = product.variants \| where: "option1", "Red" %}` |
| `pluralize` | ১টি হলে একবচন, বেশি হলে বহুবচন টেক্সট দেয় | `{{ cart.item_count \| pluralize: 'item', 'items' }}` |
| `append` / `prepend` | সামনে বা পেছনে শব্দ যোগ করে | `{{ 'style' \| append: '.css' }}` ➔ `style.css` |
| `split` | স্ট্রিং ভেঙে অ্যারে তৈরি করে | `{% assign tags = product.tags \| split: ',' %}` |
| `map` | অবজেক্টের নির্দিষ্ট প্রোপার্টির তালিকা আলাদা করে | `{% assign titles = product.variants \| map: 'title' %}` |

---

## ৪. লিকুইড ট্যাগস ও কন্ট্রোল ফ্লো (Tags & Logic)

### ১. কন্ডিশনাল ট্যাগস (Conditions)
```liquid
{% if product.available %}
  <span>In Stock</span>
{% elsif product.tags contains 'coming-soon' %}
  <span>Coming Soon</span>
{% else %}
  <span>Out of Stock</span>
{% endif %}

<!-- unless হলো if-not এর বিপরীত -->
{% unless product.has_only_default_variant %}
  <!-- ভ্যারিয়েন্ট ড্রপডাউন দেখাও -->
{% endunless %}
```

### ২. লুপ এবং ফলব্যাক (Loops)
```liquid
{% for item in collection.products limit: 4 %}
  <p>{{ forloop.index }}: {{ item.title }}</p>
{% else %}
  <p>কোনো প্রোডাক্ট পাওয়া যায়নি!</p>
{% endfor %}
```
* **`forloop` এর জাদুকরী প্রোপার্টি**:
  * `forloop.index` (১ থেকে শুরু), `forloop.index0` (০ থেকে শুরু)
  * `forloop.first` (প্রথম আইটেম হলে true)
  * `forloop.last` (শেষ আইটেম হলে true)

### ৩. ভ্যারিয়েবল ডিক্লেয়ারেশন
* **`assign`**: সাধারণ মান অ্যাসাইন করতে:
  `{% assign my_price = product.price | money %}`
* **`capture`**: একাধিক লাইন বা ব্লক একসাথে স্টোর করতে:
  ```liquid
  {% capture full_title %}
    {{ product.vendor }} - {{ product.title }}
  {% endcapture %}
  ```

### ৪. কম্পোনেন্ট ও রেন্ডারিং ট্যাগস
* `{% render 'snippet-name', custom_var: value %}`: স্নিপেট ইনক্লুড করার আধুনিক উপায় (আইসোলেটেড স্কোপ)।
* `{% section 'header' %}`: ফিক্সড সেকশন লোড করতে।
* `{% sections 'header-group' %}`: OS 2.0 সেকশন গ্রুপ লোড করতে।
* `{% paginate items by 10 %}` ... `{% endpaginate %}`: পেজিনেশনের জন্য।

---

## ৫. সেকশন স্কিমা কাঠামোর কুইক ফর্মুলা (`{% schema %}`)

```json
{% schema %}
{
  "name": "Featured Products",
  "tag": "section",
  "class": "featured-section py-8",
  "settings": [
    {
      "type": "text",
      "id": "title",
      "label": "Heading",
      "default": "Featured Collection"
    },
    {
      "type": "collection",
      "id": "featured_collection",
      "label": "Select Collection"
    },
    {
      "type": "range",
      "id": "products_to_show",
      "min": 2,
      "max": 12,
      "step": 2,
      "default": 4,
      "label": "Number of products"
    }
  ],
  "blocks": [
    {
      "type": "slide",
      "name": "Slide Item",
      "limit": 5,
      "settings": [
        { "type": "image_picker", "id": "image", "label": "Image" }
      ]
    }
  ],
  "presets": [
    {
      "name": "Featured Products"
    }
  ]
}
{% endschema %}
```

### 🎯 স্কিমা মনে রাখার ৩টি প্রধান অংশ:
1. **`settings`**: সম্পূর্ণ সেকশনের গ্লোবাল কন্ট্রোল (যেমন: টাইটেল, ব্যাকগ্রাউন্ড কালার, কালেকশন পিকার)।
2. **`blocks`**: সেকশনের ভেতরের রিপিটেবল উপাদান (যেমন: স্লাইডার স্লাইড, অ্যাকর্ডিয়ন রো, প্রোডাক্ট ট্যাবস)।
3. **`presets`**: এটি দিলে মার্চেন্ট থিম কাস্টমাইজারে "Add Section" বাটনে এই সেকশনটি খুঁজে পাবেন। (টেমপ্লেট স্পেসিফিক সেকশনে presets লাগে না)।

---

## ৬. মুখস্থ ও রিভিশন টিপস (Study & Memory Hacks)

1. **ফর্মুলা ১ (Displaying Anything)**:
   স্ক্রিনে যাই দেখতে চান, আগে ভাবুন অবজেক্ট কী। দাম হলে `{{ product.price }}`, নাম হলে `{{ product.title }}`। এরপর ভাবুন ফিল্টার লাগবে কিনা—টাকা হলে `| money`, টেক্সট কাটতে হলে `| truncatewords: 5`।
2. **ফর্মুলা ২ (Loops & Ifs)**:
   কখনোই `{% include %}` ব্যবহার করবেন না; আধুনিক শপিফাই থিমে সবসময় **`{% render %}`** ব্যবহার করবেন।
3. **ফর্মুলা ৩ (Routes Object)**:
   কখনোই হার্ডকোডেড `/cart` বা `/account/login` লিখবেন না। সবসময় **`{{ routes.cart_url }}`** বা **`{{ routes.account_login_url }}`** ব্যবহার করবেন। এর ফলে মাল্টি-কান্ট্রি বা মাল্টি-ল্যাঙ্গুয়েজ স্টোরে লিঙ্ক কখনো ভাঙবে না।

