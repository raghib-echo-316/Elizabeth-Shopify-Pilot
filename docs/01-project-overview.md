# 🛍️ Elizabeth থিম: প্রজেক্ট ওভারভিউ এবং স্টেপ-বাই-স্টেপ গাইড

> **উদ্দেশ্য:** এই গাইডটি তৈরি করা হয়েছে শপিফাই থিম ডেভেলপমেন্ট রিভিশন এবং ভবিষ্যতে নিজের নতুন থিম নিজে থেকে শুরু করার জন্য। এখানে পুরো প্রজেক্টের ব্যাকগ্রাউন্ড, টেকনোলজি স্ট্যাক এবং ধাপে ধাপে কাজ করার সম্পূর্ণ প্রক্রিয়া সহজ বাংলায় ব্যাখ্যা করা হয়েছে।

---

## ১. প্রজেক্ট পরিচিতি (Project Overview)

**Elizabeth** হলো শপিফাইয়ের আধুনিক **Online Store 2.0 (OS 2.0)** আর্কিটেকচার অনুযায়ী তৈরি একটি কাস্টম থিম। এটি মূলত একটি ই-কমার্স স্টোরের সম্পূর্ণ ফাংশনালিটি কাভার করে—হেডার, প্রোডাক্ট গ্যালারি ও ভ্যারিয়েন্ট সিলেক্টর, কার্ট, কালেকশন, ব্লগ/আর্টিকেল, কন্টাক্ট ফর্ম এবং কাস্টমার একাউন্ট।

### 🛠️ টেকনোলজি স্ট্যাক (Tech Stack)
1. **Shopify Liquid**: শপিফাইয়ের অফিসিয়াল টেমপ্লেটিং ইঞ্জিন (Ruby-ভিত্তিক)।
2. **Tailwind CSS (v4)**: আধুনিক ও দ্রুত ইউটিলিটি-ফার্স্ট CSS ফ্রেমওয়ার্ক ডিজাইন ও রেসপনসিভ লেআউটের জন্য।
3. **Alpine.js**: একটি লাইটওয়েট জাভাস্ক্রিপ্ট ফ্রেমওয়ার্ক—ড্রপডাউন মেনু, মোবাইল মেনু এবং মডাল হ্যান্ডেল করার জন্য।
4. **Vanilla JS & Web Components**: প্রোডাক্ট পেজের ভ্যারিয়েন্ট সিলেকশন এবং সেকশন রেন্ডারিং API-এর জন্য।
5. **Swiper.js**: মোবাইলে প্রোডাক্ট ইমেজের টাচ স্লাইডার তৈরির জন্য।
6. **Shopify CLI**: লোকাল ডেভেলপমেন্ট এবং শপিফাই স্টোরের সাথে রিয়েল-টাইম সিনক্রোনাইজেশনের জন্য।

---

## ২. শপিফাই থিমের ফোল্ডার আর্কিটেকচার (Theme Folder Structure)

শপিফাই থিমের প্রতিটি ফোল্ডারের নির্দিষ্ট দায়িত্ব থাকে:

```text
Elizabeth/
├── assets/          👉 স্ট্যাটিক ফাইলসমূহ (CSS, JS, ইমেজ, লোগো, ফন্ট)
├── config/          👉 থিম কাস্টমাইজারের গ্লোবাল সেটিংস ডাটা (settings_schema.json, settings_data.json)
├── layout/          👉 মাস্টার এইচটিএমএল ফ্রেম (theme.liquid)
├── locales/         👉 মাল্টি-ল্যাঙ্গুয়েজ ও ট্রান্সলেশন টেক্সট ফাইল (en.default.json)
├── sections/        👉 রি-ইউজেবল এবং ডায়নামিক কম্পোনেন্ট (Header, Product, Cart, Collection ইত্যাদি)
├── snippets/        👉 ছোট রি-ইউজেবল কোড ব্লক (Icons, Product Card, Pagination)
├── templates/       👉 বিভিন্ন পেজের জন্য OS 2.0 JSON টেমপ্লেট (index.json, product.json ইত্যাদি)
│   └── customers/   👉 লগইন, রেজিস্টার, একাউন্ট পেজের লিকুইড টেমপ্লেট
└── src/             👉 সোর্স কোড (যেমন: tailwind.css) যা বিল্ড হয়ে assets-এ যায়
```

### সহজভাবে বুঝুন:
* **Layout (`theme.liquid`)**: বাড়ির মূল ভিত্তি এবং ছাদ।
* **Templates (`product.json`, `index.json`)**: একেকটি আলাদা রুমের প্ল্যান।
* **Sections (`header.liquid`, `template-product.liquid`)**: রুমের ভেতরের আসবাবপত্র (যা ইচ্ছেমতো সাজানো যায়)।
* **Snippets (`product-card.liquid`, `icon-shopping-bag.liquid`)**: ছোট ছোট সাজসজ্জার জিনিস যা একাধিক রুমে ব্যবহার করা যায়।

---

## ৩. স্টেপ-বাই-স্টেপ ডেভেলপমেন্ট প্রসেস (Step-by-Step Workflow)

আপনি যদি ভবিষ্যতে একটি নতুন শপিফাই থিম স্ক্র্যাচ থেকে শুরু করতে চান, তাহলে নিচের ক্রম অনুযায়ী কাজ করবেন:

### ধাপ ১: ডেভেলপমেন্ট পরিবেশ তৈরি (Environment Setup)
1. শপিফাই পার্টনার একাউন্টে একটি **Development Store** তৈরি করুন।
2. মেশিনে **Node.js** এবং **Shopify CLI** ইনস্টল থাকতে হবে।
3. টার্মিনালে নতুন থিম ইনিশিয়ালাইজ করুন:
   ```bash
   shopify theme init my-new-theme
   ```
4. লোকাল সার্ভার রান করে স্টোরের সাথে কানেক্ট করুন:
   ```bash
   shopify theme dev --store your-dev-store.myshopify.com
   ```
5. Tailwind CSS সেটআপ করুন যাতে ক্লাসগুলো সরাসরি বিল্ড হয়ে `assets/application.css`-এ জমা হয়।

---

### ধাপ ২: মাস্টার লেআউট ফাইল তৈরি (`layout/theme.liquid`)
* সব পেজ যে মূল HTML কাঠামোর ভেতর রেন্ডার হবে সেটিই `theme.liquid`।
* `<head>` ট্যাগে শপিফাইয়ের প্রয়োজনীয় হুক যুক্ত করুন:
  * `{{ content_for_header }}` (শপিফাই সিস্টেম এবং অ্যাপের জন্য বাধ্যতামূলক)।
  * `application.css`, `application.js` এবং Alpine.js যুক্ত করুন।
* `<body>` ট্যাগে:
  * গ্লোবাল হেডার সেকশন রেন্ডার করুন: `{% section 'header' %}`
  * পেজের মূল কন্টেন্ট লোড করার জন্য বসান: `{{ content_for_layout }}`

---

### ধাপ ৩: নেভিগেশন বার ও হেডার তৈরি (`sections/header.liquid`)
1. **শপিফাই নেভিগেশন লিঙ্ক লিস্ট আনা**:
   `{% assign main_menu_linklist = linklists[section.settings.menu].links %}`
2. **ডেস্কটপ নেভিগেশন ও ড্রপডাউন**:
   * লুপ চালিয়ে মেনু লিঙ্ক দেখানো।
   * ড্রপডাউন মেনুর জন্য চেক করা: `{% if link.links != blank %}`।
   * Alpine.js দিয়ে ড্রপডাউন খোলা ও বন্ধ করা (`x-data="{ open: false }"`).
3. **মোবাইল মেনু ও হ্যামবার্গার আইকন**:
   * স্ক্রিন সাইজ ছোট হলে হ্যামবার্গার বাটন দেখানো।
   * Alpine.js দিয়ে স্লাইড-ইন/মডাল মোবাইল মেনু হ্যান্ডেল করা।
4. **কার্ট আইকন ও লিঙ্ক**: `{{ routes.cart_url }}` ব্যবহার করা।
5. **সেকশন স্কিমা (`{% schema %}`)**:
   * স্টোর ওনার যেন শপিফাই অ্যাডমিন থেকে যেকোনো মেনু সিলেক্ট করতে পারেন তার জন্য `link_list` ইনপুট টাইপ দেওয়া।

---

### ধাপ ৪: হোমপেজ এবং ফিচারড কালেকশন (`templates/index.json`)
1. **Online Store 2.0 কনসেপ্ট**:
   * `templates/index.json` ফাইলে সেকশন ডিফাইন করা হয়। এর ফলে শপিফাই অ্যাডমিন কাস্টমাইজার থেকে সেকশন ড্র্যাগ-অ্যান্ড-ড্রপ করা যায়।
2. **ফিচারড কালেকশন সেকশন (`sections/featured-collection.liquid`)**:
   * অ্যাডমিন থেকে সিলেক্ট করা কালেকশনের প্রোডাক্টগুলোকে লুপ করা:
     `{% for product in section.settings.featured_collection.products %}`
   * প্রতি প্রোডাক্টের জন্য একটি রিইউজেবল স্নsnippet কল করা:
     `{% render 'product-card', product_product: product %}`
   * যদি কোনো প্রোডাক্ট না থাকে, তবে `placeholder_svg_tag` দিয়ে সুন্দর ডামি কার্ড দেখানো।

---

### ধাপ ৫: প্রোডাক্ট পেজ তৈরি (`templates/product.json` & `sections/template-product.liquid`)
এটি এই থিমের সবচেয়ে অ্যাডভান্সড অংশ:
1. **মিডিয়া গ্যালারি (Product Medias)**:
   * ডেস্কটপে গ্রিড লেআউট এবং মোবাইলে **Swiper.js** ক্যারোজেল।
   * 3D মডেল ভিউয়ার সাপোর্ট (`model-viewer`)।
2. **ডায়নামিক ব্লক আর্কিটেকচার (`section.blocks`)**:
   * টাইটেল, ভেন্ডর, প্রাইস, ভ্যারিয়েন্ট সিলেক্টর, কোয়ান্টিটি, বাই বাটন—সবকিছু আলাদা আলাদা ব্লকে রাখা হয়েছে যাতে অ্যাডমিন ইচ্ছেমতো অর্ডার পরিবর্তন করতে পারেন।
3. **ভ্যারিয়েন্ট সিলেক্টর (Vanilla JS Web Component)**:
   * ড্রপডাউনে সাইজ/কালার সিলেক্ট করলে পেজ রিলোড না হয়ে URL আপডেট হয় (`history.replaceState`)।
   * শপিফাইয়ের **Section Rendering API** দিয়ে শুধু প্রাইস অংশটুকু অ্যাজাক্সে ফেচ করে আপডেট করা হয়।

---

### ধাপ ৬: কালেকশন ও পেজিনেশন (`sections/template-collection.liquid`)
1. কালেকশনের সমস্ত প্রোডাক্ট দেখাতে `{% paginate collection.products by 6 %}` ব্যবহার করা।
2. গ্রিডে প্রোডাক্ট কার্ড দেখানো।
3. `snippets/pagination.liquid`-এ পেজিনেশন অবজেক্ট পাঠিয়ে প্রিভিয়াস/নেক্সট বাটন তৈরি করা।

---

### ধাপ ৭: শপিং কার্ট পেজ (`sections/template-cart.liquid`)
1. কার্টে কোনো প্রোডাক্ট আছে কিনা চেক করা: `{% if cart.item_count > 0 %}`।
2. থাকলে ফর্ম শুরু করা: `<form action="{{ routes.cart_url }}" method="POST">`।
3. প্রতিটি লাইনের জন্য `cart.items` লুপ করা:
   * ইমেজ, টাইটেল, ভ্যারিয়েন্ট নাম।
   * কোয়ান্টিটি ইনপুট: `name="updates[]"`।
   * রিমুভ লিঙ্ক: `{{ routes.cart_change_url }}?line={{ forloop.index }}&quantity=0`।
4. সাবটোটাল দেখানো: `{{ cart.total_price | money }}`।
5. আপডেট বাটন (`name="update"`) এবং চেকআউট বাটন (`name="checkout"`).

---

### ধাপ ৮: ব্লগ ও আর্টিকেল পেজ (`sections/template-blog.liquid`, `template-article.liquid`)
1. **ব্লগ পেজ**: ব্লগের সমস্ত আর্টিকেল গ্রিড আকারে পেজিনেশন সহ প্রদর্শন করা।
2. **আর্টিকেল পেজ**:
   * টাইটেল, লেখক, প্রকাশের তারিখ, আর্টিকেল কন্টেন্ট।
   * **কমেন্ট সিস্টেম**: শপিফাইয়ের কমেন্ট ফর্ম `{% form 'new_comment', article %}` ব্যবহার করে ভ্যালিডেশন এবং সাকসেস মেসেজ দেখানো।
   * **SEO মাইক্রোডাটা**: গুগল সার্চ ইঞ্জিনের জন্য `itemscope itemtype="http://schema.org/BlogPosting"` স্কিমা যুক্ত করা।

---

### ধাপ ৯: অন্যান্য জরুরি পেজসমূহ
1. **৪০৪ পেজ (`sections/template-404.liquid`)**: ভাঙা লিঙ্কের জন্য ইউজার ফ্রেন্ডলি পেজ এবং হোমপেজে ফিরে যাওয়ার বাটন।
2. **কন্টাক্ট ফর্ম (`sections/contact-form.liquid`)**: শপিফাইয়ের নেটিভ কন্টাক্ট হ্যান্ডলার `{% form 'contact' %}` ব্যবহার করে ফর্ম ডেটা সরাসরি স্টোর ওনারের ইমেইলে পাঠানো।
3. **সার্চ পেজ (`templates/search.liquid`)**: স্টোরে প্রোডাক্ট বা আর্টিকেল খোঁজার জন্য সার্চ বার এবং রেজাল্ট লিস্ট।
4. **কাস্টমার একাউন্ট (`templates/customers/*`)**: লগইন, রেজিস্টার, এড্রেস ও অর্ডার হিস্ট্রি।

---

## ৪. নতুন থিম নিজে থেকে শুরু করার কুইক রোডম্যাপ (Self-Practice Checklist)

ভবিষ্যতে যখন আপনি নতুন একটি প্রজেক্ট শুরু করবেন, তখন নিচের চেকলিস্টটি ফলো করুন:

| ধাপ | কাজ | কী কী ফাইল তৈরি করবেন |
|---|---|---|
| **১** | বেস সেটআপ | `shopify theme init`, `package.json`, Tailwind সেটআপ |
| **২** | গ্লোবাল লেআউট | `layout/theme.liquid`, `assets/application.css`, `assets/application.js` |
| **৩** | গ্লোবাল হেডার ও ফুটার | `sections/header.liquid`, `sections/footer.liquid` |
| **৪** | হোমপেজ | `templates/index.json`, `sections/featured-collection.liquid`, `snippets/product-card.liquid` |
| **৫** | কালেকশন ও পেজিনেশন | `templates/collection.json`, `sections/template-collection.liquid`, `snippets/pagination.liquid` |
| **৬** | প্রোডাক্ট পেজ | `templates/product.json`, `sections/template-product.liquid`, `snippets/product-media.liquid` |
| **৭** | কার্ট পেজ | `templates/cart.json`, `sections/template-cart.liquid` |
| **৮** | কন্টেন্ট ও ইনফো পেজ | `templates/page.json`, `templates/page.contact.json`, `sections/contact-form.liquid` |
| **৯** | ব্লগ ও আর্টিকেল | `templates/blog.json`, `templates/article.json`, `sections/template-article.liquid` |
| **১০** | সিস্টেম ও ইউটিলিটি পেজ | `templates/404.json`, `sections/template-404.liquid`, `templates/search.liquid` |

