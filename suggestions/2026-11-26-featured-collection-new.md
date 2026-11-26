# 2026-11-26 — Featured Collection New only shows placeholders

## 🔴 1. The loop runs over the collection, not its products

**Where:** [sections/featured-collection-new.liquid:3](../sections/featured-collection-new.liquid#L3)

**What I noticed:** `{% for product in section.settings.featured_collection_new %}` loops over the **collection object** itself.

**Why it matters:** A `collection` setting gives you the whole collection (its title, URL, image, products and so on), not a list of products. Looping over the object finds nothing to step through, so the loop runs zero times and the `{% else %}` placeholders always show, even after you pick a collection.

**Try it yourself:** Compare with [sections/featured-collection.liquid:6](../sections/featured-collection.liquid#L6). What property does it loop over? Add the same to line 3.

**Lesson:** When a `for` loop shows nothing, ask "Am I looping over a *list*, or over an *object* that contains a list?" Writing `{{ section.settings.featured_collection_new | json }}` temporarily on the page shows you what you actually have.

- [ ] Done

---

## 🟢 2. An empty collection still shows placeholders

That's expected. The `{% else %}` runs whenever the loop has nothing to go through. That happens when no collection is selected, *or* when the selected collection has no products in it.

- [ ] Understood

---

## 🟡 3. Which card snippet should this section use?

**Where:** [snippets/wax-product-card.liquid](../snippets/wax-product-card.liquid) and [snippets/wax-product-placeholder.liquid](../snippets/wax-product-placeholder.liquid)

**What I noticed:** These two snippets are nearly the same, and both have hardcoded text ("Product Title", a fixed price). The section renders the older `product-card` snippet instead.

**Try it yourself:** If `wax-product-card` is meant to be your new design, make it read real product data the way `product-card` does with `product_product`. Then render it from the section.

- [ ] Decided

---

## 🟢 4. Why the card image isn't 500 × 500

**Where:** [snippets/product-card.liquid:5-6](../snippets/product-card.liquid#L5-L6)

**What to learn:** There are three different "sizes" in this code:
- `image_url: width: 500` sets the size of the **file** Shopify sends. Its height follows the photo's own shape.
- The `width="500" height="500"` attributes only tell the browser the image's **shape** before it loads. Any CSS size wins over them.
- The CSS sets what you actually **see**: `h-40` on the wrapper div is 40 × 0.25rem = **160px** ([assets/application.css:405](../assets/application.css#L405)). `w-full h-full object-cover` then fills that box and crops the photo.

**Try it yourself (tip 4):** Inspect the wrapper div in DevTools and check its computed height. Try `h-60`, or an aspect-ratio class like `aspect-square`. Your Tailwind build includes only the classes you've already used, so a new class may need a rebuild before it works. Afterwards, make the `width`/`height` attributes match the new shape.

- [ ] Understood

---

## 🔴 5. `product.all_available_sizes` doesn't exist

**Where:** the product card, where you wrote `{{ product_product.all_available_sizes }}`

**What I noticed:** That property isn't real. Liquid prints nothing for unknown properties and shows no error.

**Why it matters:** Sizes are **product options**. You reach them through:
- `product.options_by_name['Size']` gives the Size option. It's empty if the product has no option with that name.
- `.values` on that option gives the list of values: S, M, L…
- `value.available` is true if at least one variant with that value can be bought.
- `product.options_with_values` gives every option, which helps when you're unsure of the name.

**Try it yourself:**
1. Check the option's exact name in Shopify admin.
2. `{% assign %}` the option to a variable, then wrap the output in `{% if %}` so products without sizes show nothing.
3. Loop over `.values`, and print only the values where `.available` is true.
4. To debug, temporarily add `{{ product_product.options_with_values | json }}`.

**Bonus:** show sold-out sizes crossed out instead of hiding them.

- [ ] Done

**Follow-up: `product.options` vs `product.options_with_values`**
- `product.options` is a list of option **names** only, as plain text, for example `["Size", "Color"]`. There are no values inside it.
- `product.options_with_values` is a list of option **objects**. Each one has `.name` and `.values`, and each value has `.available`.
- So you need two loops: the outer one goes over the options, the inner one over each option's `.values`.
- Be careful with `options.size > 0`: a product with no variants still has one default option, called "Title". Test the option's **name** instead.

**Review of attempt 1** ([snippets/wax-product-card.liquid:24-31](../snippets/wax-product-card.liquid#L24-L31)): you're close. Still to fix:
- [ ] The loop still uses `options` (just text), so `option.name` is empty. Switch to `options_with_values`.
- [ ] `"size"` vs `"Size"`: Liquid compares text exactly, including capital letters.
- [ ] `{{ option.values }}` glues every value together ("SML"). Add an inner `for` loop.
- [ ] Remove the outer `options.size > 0` check. It's almost always true.
