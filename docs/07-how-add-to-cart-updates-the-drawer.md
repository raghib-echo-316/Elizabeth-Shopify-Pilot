# How "Add to Cart" Updates the Cart Drawer

When you click **Add to Cart** on a product page, the page does **not** reload. Instead:

1. JavaScript stops the normal form submit.
2. It sends the product to Shopify in the background.
3. Shopify sends back **ready-made HTML** for the cart drawer.
4. JavaScript pastes that HTML into the drawer and opens it.

That's the whole idea. The rest of this page walks through each step with the real code.

Files involved:
- `sections/template-product.liquid`: the Add to Cart form (`<product-form>`)
- `sections/cart-drawer.liquid`: the drawer HTML and the `renderContents` function

---

## The big picture

```
 [ Click "Add to Cart" ]
          │
          ▼
 product-form stops the page reload
          │
          ▼
 POST /cart/add.js   ──►  Shopify adds the item
 (also asks for the        and renders cart-drawer.liquid
  "cart-drawer" section)    with the NEW cart
          │
          ▼
 Response = item info + drawer HTML
          │
          ▼
 cartDrawer.renderContents(response)
          │
          ▼
 Old items/subtotal replaced with new ones → drawer slides open
```

---

## Step 1: Stop the page from reloading

File: `sections/template-product.liquid`

```js
this.form.addEventListener('submit', this.onSubmit.bind(this));

onSubmit(event) {
    event.preventDefault();          // ← no page reload
    if(!this.button || this.button.disabled) return;

    this.showError(null);
    const originalLabel = this.label ? this.label.textContent.trim() : '';
    this.setLoading(true);           // button disabled, label = "Adding…"
    ...
}
```

Normally a form submit takes you to the `/cart` page. `preventDefault()` cancels that so we can do it with JavaScript instead.

---

## Step 2: Send the product to Shopify, and ask for the drawer HTML too

```js
const formData = new FormData(this.form);          // variant id + quantity from the form
formData.append('sections', 'cart-drawer');        // "please also render this section"
formData.append('sections_url', window.location.pathname);

fetch(`${root}cart/add.js`, {
    method: 'POST',
    headers: { 'Accept': 'application/json', 'X-Requested-With': 'XMLHttpRequest' },
    body: formData
})
```

- `FormData(this.form)` collects everything in the form, mainly the variant `id` and the `quantity`.
- `/cart/add.js` is Shopify's endpoint for adding an item to the cart.
- **The key line is `sections: 'cart-drawer'`.** It tells Shopify:
  *"After you add the item, render `cart-drawer.liquid` with the updated cart and send me its HTML."*

This is called the **Section Rendering API**. It means we never have to build the cart HTML in JavaScript, because Liquid does it for us.

### What Shopify sends back (simplified)

```json
{
  "id": 123456789,
  "title": "Blue Dress - M",
  "quantity": 1,
  "sections": {
    "cart-drawer": "<div id=\"shopify-section-cart-drawer\"> ...full drawer HTML... </div>"
  }
}
```

So in **one request** we get both the added item and the fresh drawer HTML.

---

## Step 3: Check the response

```js
.then((response) => response.json())
.then((response) => {
    // Error (e.g. out of stock) → Shopify sends a "status" field
    if(response.status) {
        this.setLoading(false, originalLabel);
        this.showError(response.description || response.message);
        return;
    }

    // Tell the rest of the page an item was added (optional hook)
    document.dispatchEvent(new CustomEvent('cart:item-added', {
        detail: { item: response, sections: response.sections }
    }));

    // Hand the response to the cart drawer
    const cartDrawer = document.querySelector('cart-drawer');
    if(cartDrawer && typeof cartDrawer.renderContents === 'function') {
        this.setLoading(false, originalLabel);
        cartDrawer.renderContents(response);
    }
})
.catch(() => {
    this.form.submit();   // something broke → fall back to a normal page submit
});
```

| Situation | What happens |
|---|---|
| Error (out of stock, etc.) | Error message shows under the button |
| Success | The drawer gets updated (Step 4) |
| No drawer on the page | Button says "Added!" for 2 seconds |
| Network failure | Normal form submit (page goes to `/cart`) |

---

## Step 4: The drawer swaps in the new HTML

File: `sections/cart-drawer.liquid` (the `<script>` at the bottom)

```js
const cartDrawer = document.querySelector('cart-drawer');

cartDrawer.renderContents = (response) => {
  const applyHtml = (html) => {
    // 1. Turn the HTML string into a document we can search
    const doc = new DOMParser().parseFromString(html, 'text/html');

    // 2. Replace only the body (items) and footer (subtotal)
    ['.cart-modal__body', '.cart-modal__footer'].forEach((selector) => {
      const fresh = doc.querySelector(selector);          // new version
      const current = cartDrawer.querySelector(selector); // what's on screen now
      if (fresh && current) current.innerHTML = fresh.innerHTML;
    });

    // 3. Update the cart count bubble in the header
    const freshFooter = doc.querySelector('.cart-modal__footer');
    if (freshFooter) {
      const count = Number(freshFooter.dataset.itemCount) || 0;
      document.querySelectorAll('[data-cart-count]').forEach((bubble) => {
        bubble.textContent = count;
        bubble.hidden = count === 0;
      });
    }

    // 4. Open the drawer
    cartDrawer.querySelector('.cart-drawer-wrapper').classList.add('is-active');
  };

  if (response && response.sections && response.sections['cart-drawer']) {
    applyHtml(response.sections['cart-drawer']);   // normal case
  } else {
    // Backup: HTML wasn't in the response, so fetch it separately
    fetch(`${window.location.pathname}?sections=cart-drawer`)
      .then((res) => res.json())
      .then((sections) => applyHtml(sections['cart-drawer']));
  }
};
```

What each part does:

1. **`DOMParser`** turns the HTML text into something we can search with `querySelector`.
2. **Swap body + footer.** The items list and the subtotal are replaced with the new versions. The **header is not touched**, so the close (X) button keeps working.
3. **Cart count.** The footer has `data-item-count="{{ cart.item_count }}"`. We read that number and put it in the header bubble.
4. **Open the drawer.** Adding the `is-active` class makes the CSS slide it in:

```css
.cart-drawer-wrapper.is-active {
  opacity: 1;
  visibility: visible;
}
```

### Where does the new HTML come from?

It's the same Liquid you already wrote in `cart-drawer.liquid`:

```liquid
<div class="cart-modal__body">
  {% for item in cart.items %}
    <div class="cart-item" data-line="{{ forloop.index }}"> ... </div>
  {% endfor %}
</div>

<div class="cart-modal__footer" data-item-count="{{ cart.item_count }}">
  Subtotal: <span class="subtotal-price">{{ cart.total_price | money }}</span>
</div>
```

Shopify runs this Liquid again on the server with the **updated** cart and sends us the result.

---

## Deep dive: `renderContents` line by line

Here is the full function again, followed by an explanation of each piece.

```js
// Swap in fresh body + footer HTML (header is left alone so the close button listener survives)
cartDrawer.renderContents = (response) => {
  const applyHtml = (html) => {
    const doc = new DOMParser().parseFromString(html, 'text/html');
    ['.cart-modal__body', '.cart-modal__footer'].forEach((selector) => {
      const fresh = doc.querySelector(selector);
      const current = cartDrawer.querySelector(selector);
      if (fresh && current) current.innerHTML = fresh.innerHTML;
    });

    // Keep the header cart bubble in sync with the freshly rendered cart
    const freshFooter = doc.querySelector('.cart-modal__footer');
    if (freshFooter) {
      const count = Number(freshFooter.dataset.itemCount) || 0;
      document.querySelectorAll('[data-cart-count]').forEach((bubble) => {
        bubble.textContent = count;
        bubble.hidden = count === 0;
      });
    }
    cartDrawer.querySelector('.cart-drawer-wrapper').classList.add('is-active');
  };

  if (response && response.sections && response.sections['cart-drawer']) {
    applyHtml(response.sections['cart-drawer']);
  } else {
    fetch(`${window.location.pathname}?sections=cart-drawer`)
      .then((res) => res.json())
      .then((sections) => applyHtml(sections['cart-drawer']));
  }
};
```

### Read it bottom-up

The function has two parts:

- **Top part:** `applyHtml` is a helper that is only *defined* here. It doesn't run yet.
- **Bottom part:** the `if / else` is what actually runs first. It finds the HTML and then calls `applyHtml` with it.

So it's easiest to read the bottom part first.

---

### Part A: Attach the function to the drawer

```js
cartDrawer.renderContents = (response) => { ... };
```

- `cartDrawer` is the `<cart-drawer>` element on the page.
- We add our own method called `renderContents` to it.
- Any other code can now call `document.querySelector('cart-drawer').renderContents(...)`. That's exactly what the product form does after adding an item.
- `response` is the JSON that Shopify sent back from `/cart/add.js` (or `/cart/change.js`, or `/cart/clear.js`).

---

### Part B: Find the HTML (the `if / else` at the bottom)

```js
if (response && response.sections && response.sections['cart-drawer']) {
  applyHtml(response.sections['cart-drawer']);
}
```

**Normal case.** We asked Shopify for `sections: 'cart-drawer'`, so the HTML should be in `response.sections['cart-drawer']`.

The three checks joined by `&&` stop the code from crashing if something is missing:

| Check | It protects against |
|---|---|
| `response` | No response at all |
| `response.sections` | Shopify didn't include any sections |
| `response.sections['cart-drawer']` | The sections came back, but the drawer HTML isn't there |

If all three exist, we pass the HTML string to `applyHtml`.

```js
else {
  fetch(`${window.location.pathname}?sections=cart-drawer`)
    .then((res) => res.json())
    .then((sections) => applyHtml(sections['cart-drawer']));
}
```

**Backup case.** If the HTML wasn't in the response, we ask Shopify for it directly.

- `window.location.pathname` is the current page, for example `/products/blue-dress`.
- Adding `?sections=cart-drawer` makes Shopify return **only** that section as JSON, instead of the whole page:
  ```json
  { "cart-drawer": "<div id=\"shopify-section-cart-drawer\">...</div>" }
  ```
- Then we pass `sections['cart-drawer']` to `applyHtml`, the same as in the normal case.

---

### Part C: `applyHtml`, which puts the new HTML on screen

#### C1. Turn the text into a searchable document

```js
const doc = new DOMParser().parseFromString(html, 'text/html');
```

`html` is just a long **string** of text. You can't call `querySelector` on a string.
`DOMParser` converts it into a mini document (`doc`) that lives in memory and isn't shown on screen. Now we can search inside it like a normal page.

#### C2. Swap the body and the footer

```js
['.cart-modal__body', '.cart-modal__footer'].forEach((selector) => {
  const fresh = doc.querySelector(selector);           // new version (from Shopify)
  const current = cartDrawer.querySelector(selector);  // old version (on screen)
  if (fresh && current) current.innerHTML = fresh.innerHTML;
});
```

This loop runs **twice**, once for each selector:

| Run | `selector` | What gets replaced |
|---|---|---|
| 1 | `.cart-modal__body` | The list of cart items |
| 2 | `.cart-modal__footer` | Clear cart button + subtotal |

- `fresh` is the element in the new HTML.
- `current` is the element on the page right now.
- `current.innerHTML = fresh.innerHTML` copies the **inside** of the new element into the old one. The outer `<div>` stays in place and only its content changes.
- `if (fresh && current)` skips the swap if either element is missing, so nothing crashes.

**Why not replace the whole drawer?**
The header holds the close (X) button, and a click listener is attached to that exact button:

```js
document.querySelector('#cart-modal__close-btn').addEventListener('click', ...);
```

If we replaced the header, the old button (with its listener) would be thrown away, the new button would have no listener, and X would stop working. Swapping only the body and footer avoids that problem.

#### C3. Update the cart count bubble in the header

```js
const freshFooter = doc.querySelector('.cart-modal__footer');
if (freshFooter) {
  const count = Number(freshFooter.dataset.itemCount) || 0;
  document.querySelectorAll('[data-cart-count]').forEach((bubble) => {
    bubble.textContent = count;
    bubble.hidden = count === 0;
  });
}
```

The site header (outside the drawer) shows a little number with how many items are in the cart. It needs updating too.

1. **Where does the number come from?** The footer in Liquid has:
   ```liquid
   <div class="cart-modal__footer" data-item-count="{{ cart.item_count }}">
   ```
   So the new HTML contains something like `data-item-count="3"`.

2. **`freshFooter.dataset.itemCount`** reads that attribute. JavaScript turns `data-item-count` into `dataset.itemCount` (dashes become camelCase).

3. **`Number(...) || 0`**: attributes are always text (`"3"`), so `Number` turns it into a real number (`3`). If it's missing or not a number, `|| 0` falls back to `0`.

4. **`querySelectorAll('[data-cart-count]')`** finds every element on the page with a `data-cart-count` attribute (the header bubble, or more than one if you add others).

5. For each bubble:
   - `bubble.textContent = count` shows the new number.
   - `bubble.hidden = count === 0` hides the bubble when the cart is empty and shows it otherwise.

#### C4. Open the drawer

```js
cartDrawer.querySelector('.cart-drawer-wrapper').classList.add('is-active');
```

This adds the `is-active` class, and the CSS does the rest (fade in the dark overlay, slide the panel in from the right):

```css
.cart-drawer-wrapper.is-active { opacity: 1; visibility: visible; }
.cart-drawer-wrapper.is-active .cart-drawer-modal { transform: translate(0); }
```

`classList.add` does nothing if the class is already there. So when you change a quantity while the drawer is already open, it just stays open.

---

### The whole thing in plain words

1. "Did Shopify give me the drawer HTML? If yes, use it. If no, go fetch it."
2. "Turn that HTML text into something I can search."
3. "Copy the new item list and the new footer into the drawer. Leave the header alone."
4. "Read the new item count and update the header bubble."
5. "Open the drawer."

---

## Bonus: the + / − / remove / clear buttons work the same way

Inside the drawer, changing a quantity calls `/cart/change.js` with the same `sections: 'cart-drawer'` trick, then calls the same `renderContents`:

```js
fetch(`${cartRoot}cart/change.js`, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json', 'Accept': 'application/json' },
  body: JSON.stringify({
    line: Number(item.dataset.line),
    quantity: Math.max(0, quantity),
    sections: 'cart-drawer',
    sections_url: window.location.pathname
  })
})
  .then((res) => res.json())
  .then((response) => cartDrawer.renderContents(response));
```

**Why do the buttons still work after the HTML is replaced?**
The click listener sits on `<cart-drawer>` itself, which never gets replaced, not on each button:

```js
cartDrawer.addEventListener('click', (event) => {
  const item = event.target.closest('.cart-item');
  if (event.target.closest('[data-qty-plus]')) updateLine(item, quantity + 1);
  ...
});
```

This is called **event delegation**: one listener on the parent catches clicks from any child, even children added later.

---

## Summary in one sentence

> The product form sends the item to `/cart/add.js` and asks Shopify for the `cart-drawer` section HTML in the same request. Then `renderContents` swaps the new items and subtotal into the drawer and slides it open.
