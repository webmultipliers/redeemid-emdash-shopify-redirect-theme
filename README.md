# RedeemID redirect theme for EmDash

A Shopify theme that sends visitors from your store's online store pages to your [EmDash](https://github.com/emdash-cms/emdash) site, while checkout, order status, gift cards and customer accounts stay on Shopify.

Use it with [RedeemID for EmDash](https://github.com/webmultipliers/redeemid-emdash) or the [RedeemID starter](https://github.com/webmultipliers/redeemid-emdash-starter). Your site runs the shop, the cart and the content, so the store's own pages have nothing left to show. Without this theme they still answer at the store's address: shoppers who follow an old link, or the store link at the top of checkout, land on a second, bare shop, and search engines find two pages for every product.

With the theme published:

- `shop.example.com/products/gold` opens `example.com/shop/gold/`, and every other store page opens its page on the site. The query string comes along.
- Shopify's discount links (`shop.example.com/discount/SAVE10?redirect=/products/gold`) land on the site with the code, and the site adds it to the cart.
- A customer who is signed in to the store is signed in on the site too, without a form.
- Checkout, order status pages, customer accounts, gift cards, bot protection and app proxies (`/apps/…`) are not redirected.
- The store's pages carry `noindex`, a canonical link to the site's page, and a `robots.txt` that disallows everything.

## Install

1. Download this repository as a zip: **Code → Download ZIP** on GitHub, or [this link](https://github.com/webmultipliers/redeemid-emdash-shopify-redirect-theme/archive/refs/heads/main.zip).
2. In Shopify admin, open **Online Store → Themes**.
3. Under **Theme library**, choose **Add theme → Upload zip file** and pick the zip.

   Or, to get updates from your own copy, fork this repository and choose **Add theme → Connect from GitHub** instead.
4. On the new theme, choose **Customize**, then open **Theme settings → Your site**.
5. In **Send visitors to**, enter your site's domain, such as `example.com` (no `https://`).
6. Check **Paths on your site** (below), then **Save**.
7. Back in **Themes**, choose **Publish** on the theme.

Try it: open your store's address, such as `https://shop.example.com/products/<a product handle>`. You should land on the product's page on your site.

The store's address is its primary domain in Shopify (**Settings → Domains**), or `my-store.myshopify.com` if it has none. Keep your site's own domain off Shopify; the [starter's domain table](https://github.com/webmultipliers/redeemid-emdash-starter#2-domains) shows the usual split.

## Paths on your site

**Paths on your site** maps store paths to site paths, one rule per line:

```
store path > site path
```

The first rule that matches wins. A path that no rule matches keeps its own path, so `shop.example.com/cart` opens `example.com/cart`.

- `:name` matches one part of the path and puts it in the site path.
- `*` at the end of the store path matches the rest of the path, which may be empty. Put `*` in the site path to keep the rest.
- Anything else must match exactly. Trailing slashes on the store side don't matter. The site path keeps the one you write.
- A market's language prefix, such as `/fr`, is dropped before the rules run.

The default rules match the [RedeemID starter](https://github.com/webmultipliers/redeemid-emdash-starter):

```
/products/:handle > /shop/:handle/
/collections/:collection/products/:handle > /shop/:handle/
/collections/* > /shop/
/search > /shop/search
/pages/:page > /:page/
/blogs/:blog/:article > /posts/:article/
/blogs/* > /posts/
/account/* > /account
```

| Store page | Opens |
| --- | --- |
| `/` | `/` |
| `/products/gold` | `/shop/gold/` |
| `/collections/frontpage/products/gold` | `/shop/gold/` |
| `/collections/all` | `/shop/` |
| `/search?q=gold` | `/shop/search?q=gold` |
| `/cart` | `/cart` |
| `/pages/about` | `/about/` |
| `/blogs/news/hello` | `/posts/hello/` |
| `/account/login` | `/account` |

If your site uses other paths, change the rules to match. For example, a site with a page for each collection at `/collections/:collection/` would use:

```
/collections/:collection/products/:handle > /shop/:handle/
/collections/:collection > /collections/:collection/
```

Policy pages (`/policies/…`) keep their path, so add a rule if your site has them elsewhere, such as `/policies/privacy-policy > /privacy/`.

## Discount links

Shopify's discount links set a `discount_code` cookie on the store and then open a store page. The theme moves the code to the site as `?discount=SAVE10` and deletes the cookie. [RedeemID for EmDash](https://github.com/webmultipliers/redeemid-emdash/blob/main/docs/shop.md#discount-links) keeps the code for the visitor and adds it to their cart, now or when they add their first item, and Shopify applies it at checkout.

## Sign-in

**Carry sign-in over to the site** (on by default) adds `logged_in=true` when the visitor is signed in to the store's customer account. RedeemID for EmDash then signs them in on the site without showing a form, and they land on the page they asked for. If the store session has ended by then, they stay signed out.

For B2B stores, a signed-in B2B customer's chosen location is added as `company_location_id`. RedeemID doesn't use it; the store still checks it at checkout.

## Gift cards and bot protection

Shopify shows gift cards on the store, so `templates/gift_card.liquid` keeps a gift card page, with **Continue shopping** pointing at your site. Change it like any theme template.

`templates/checkpoint.liquid` keeps Shopify's bot protection page for flash sales. The theme never redirects `/checkpoint`, `/challenge` or `/throttle/queue`.

## How it works

`layout/theme.liquid` works out the site address on the server, from **Send visitors to** and the rules in `snippets/site-path.liquid`, and uses it for the canonical link and the fallback link. A short script then moves the visitor there, adding the query string, the discount code and the sign-in flag. Every template uses the empty `main-redirect` section. The theme editor never redirects, so you can still customize the theme.

## License

MIT. Based on Shopify's Hydrogen redirect theme (the Sell on WordPress theme), © Shopify, Inc., and Instant Commerce, also MIT. See [LICENSE.md](LICENSE.md).
