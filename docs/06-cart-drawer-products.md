# Show Products in the Cart Drawer

Three small steps: list the items in Liquid, show the subtotal, then refresh the drawer after "Add to Cart".

Files involved:
- `sections/cart-drawer.liquid`
- `assets/cart-drawer.js`
- `sections/template-product.liquid` (already done, nothing to change)

---

## Step 1: List the cart items

In `sections/cart-drawer.liquid`, replace:

```liquid
<div class="cart-modal__body">body</div>
```

with:

```liquid
<div class="cart-modal__body">
  {% if cart.item_count > 0 %}
    {% for item in cart.items %}
      <div class="cart-item">
        <img src="{{ item.image | image_url: width: 120 }}" alt="{{ item.title | escape }}" width="60" height="60">
        <div>
          <a href="{{ item.url }}">{{ item.product.title }}</a>
          {% unless item.product.has_only_default_variant %}
            <p>{{ item.variant.title }}</p>
          {% endunless %}
          <p>Qty: {{ item.quantity }}</p>
          <p>{{ item.final_line_price | money }}</p>
        </div>
      </div>
    {% endfor %}
  {% else %}
    <p>Your cart is empty.</p>
  {% endif %}
</div>
```

`cart` is a global Liquid object, so it works on every page.

---

## Step 2: Show the subtotal

Replace:

```liquid
<div class="cart-modal__footer">footer</div>
```

with:

```liquid
<div class="cart-modal__footer">
  <p>Subtotal: {{ cart.total_price | money }}</p>
  <a href="{{ routes.cart_url }}">View cart</a>
  <a href="/checkout">Checkout</a>
</div>
```

Add some basic styles inside the existing `{% stylesheet %}` block:

```css
.cart-modal__body {
  flex: 1;
  overflow-y: auto;
  padding: 16px;
}
.cart-item {
  display: flex;
  gap: 12px;
  margin-bottom: 16px;
}
.cart-modal__footer {
  border-top: 1px solid #ddd;
  padding: 16px;
}
```

At this point, reloading the page shows the products in the drawer.

---

## Step 3: Update the drawer after "Add to Cart" (no reload)

The product page already does this for you in `sections/template-product.liquid`:

```js
formData.append('sections', 'cart-drawer');   // ask Shopify for fresh drawer HTML
...
cartDrawer.renderContents(response);          // hand it to the drawer
```

So you only need to add `renderContents()` to `assets/cart-drawer.js`:

```js
class CartDrawer extends HTMLElement {
  renderContents(response) {
    const html = response.sections && response.sections['cart-drawer'];
    if (!html) return;

    const newDoc = new DOMParser().parseFromString(html, 'text/html');

    // Swap only the body and footer, so the close button keeps working
    ['.cart-modal__body', '.cart-modal__footer'].forEach((selector) => {
      const fresh = newDoc.querySelector(selector);
      const current = this.querySelector(selector);
      if (fresh && current) current.innerHTML = fresh.innerHTML;
    });

    // Open the drawer
    this.querySelector('.cart-drawer-wrapper').classList.add('is-active');
  }
}

customElements.define('cart-drawer', CartDrawer);
```

---

## How it works

```text
Click "Add to Cart"
   → POST /cart/add.js  (with sections=cart-drawer)
   → Shopify returns the item + fresh cart-drawer HTML
   → renderContents() swaps body/footer and opens the drawer
```

---

## Quick test

1. Open a product page and click **Add to Cart**. The drawer should open with the product inside.
2. Reload the page and click the cart icon in the header. The product should still be there.
3. Empty the cart. The drawer should show "Your cart is empty."
