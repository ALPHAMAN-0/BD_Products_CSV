# BD Products CSV

Product lists from online grocery stores and pharmacies in Bangladesh. Each store has its own folder with a CSV of every product's name, price and picture, plus a folder of the pictures.

## Stores

| Store | Website | Folder | Products | Collected | Status |
|---|---|---|---|---|---|
| Chaldal | [chaldal.com](https://chaldal.com) | [`chaldal-all/`](chaldal-all/) | 3,374 | 10 Nov 2026 | Uploaded |
| Shwapno | [shwapno.com](https://www.shwapno.com) | [`shwapno-all/`](shwapno-all/) | 9,366 | 11 Nov 2026 | Uploaded |
| Prince Bazar | [princebazarbd.com](https://princebazarbd.com) | [`prince-bazar-all/`](prince-bazar-all/) | 7,791 | 11 Nov 2026 | Uploaded |
| Arogga | [arogga.com](https://www.arogga.com) | [`arogga-all/`](arogga-all/) | 21,660 | 11–12 Nov 2026 | Uploaded |
| Agora | | `agora-all/` | | | Not uploaded yet |

Prices are in Bangladeshi taka (BDT) and are what each site showed on the day it was collected. They aren't kept up to date.

## What's in a store folder

| File | What it holds |
|---|---|
| `products.csv` | One row per product, with three columns (below). |
| `images/` | The product pictures, one per row, all in WebP format. |

| Column | Meaning | Example |
|---|---|---|
| `name` | Product name, as written on the site | `Himalaya Baby Cream 100gm` |
| `price` | Price in BDT | `164` |
| `image_path` | The product's picture, relative to the store folder | `images/himalaya-baby-cream-100g-6a61cfbb881648e68a2ea806.webp` |

Every row has all three values, and every `image_path` points to a file that exists. Every picture in `images/` is used by at least one row. Products that had no picture or no price on the site are left out.

All CSV files are UTF-8 with a byte order mark, so Excel shows Bangla text correctly when you open them directly.

Pictures that the stores served as JPG, PNG or GIF were converted to WebP at quality 82, at their original size. Pictures the stores already served as WebP are unchanged. Every current browser opens WebP, and so does Python's Pillow.

## Notes per store

### Chaldal

- Only products that were in stock on 10 Nov 2026 are included. Chaldal's category pages don't list out-of-stock products.
- Names don't include the pack size, so two sizes of the same product share a name. The picture file name does include the size, for example `images/johnsons-baby-jelly-lightly-fragranced-100-ml-129599.webp`.
- Price is what a shopper who isn't logged in pays.
- Two pictures are very large, so Python's Pillow won't open them with its default settings. Set `PIL.Image.MAX_IMAGE_PIXELS = None` before opening them:
  - `images/aquafresh-little-teeth-toothpaste-3-5-years-50-ml-65746.webp` (13888×13888)
  - `images/savlon-twinkle-baby-belt-diaper-m-6-11-kg-40-pcs-67841.webp` (10000×10000)

### Shwapno

- Includes out-of-stock products, which were most of the list.
- Prices are for the store the site picks when no delivery area is set (Banani). Other branches may differ. A few prices are per kg rather than per piece.
- 5,073 products had only Shwapno's "no picture" stand-in on the site, so they're left out.

### Prince Bazar

- Prince Bazar has two branches (Mirpur 1 and Mirpur 14), each with its own prices. An item with the same price at both branches has one row. An item with different prices has one row per price, with the same name.
- One price is a typo on the site: `university Khata-05` (an exercise book) is listed at 800,000. Every other price is under 8,100.

### Arogga

- Arogga is an online pharmacy that also sells beauty, supplement and household products. About two-thirds of the list is medicine.
- The picture download was stopped partway, so only 21,660 of the 58,219 products on the site are included.
- Medicine names are usually just the brand name, such as `Moxin`, without the strength or form. The same name can appear several times, once for each strength, with different prices and pictures.
- Products listed at price 0 (not for sale) are left out.

## Using the data

Python with pandas:

```python
import pandas as pd

df = pd.read_csv("chaldal-all/products.csv", encoding="utf-8-sig")
print(df["price"].describe())
```

Open a product's picture:

```python
from PIL import Image

row = df.iloc[0]
Image.open(f"chaldal-all/{row['image_path']}").show()
```

## How the data was collected

Each store was read by its own crawler script, kept in a separate project. The crawlers follow each site's `robots.txt`, send one request at a time with at least one second between requests, and stop if the site starts blocking them. They don't log in or get around any blocks.

The crawlers save more columns than this repository keeps (category, stock, product links and more), along with a log and a cache. Those were removed to keep only the name, price and picture.

## Adding a store

The crawler output is too big to push as a single zip, because GitHub allows at most 100 MB per file. Unzip it here instead, trim it to `products.csv` (`name`, `price`, `image_path`) and `images/`, convert the pictures to WebP, then commit the folder:

```bash
unzip -q Store_All.zip -x '__MACOSX/*'
git add store-all
git commit -m "Add Store product data"
git push
```

`.gitignore` already skips `.zip` files, `.DS_Store` and `__MACOSX/`. GitHub also limits each push to 2 GB, so push one store at a time, or split a large images folder across several pushes. Then add the store to the table at the top.

## Terms of use

The product names, prices and pictures belong to each store, and the pictures may also be copyrighted by the brands. Chaldal's Terms of Use (section 6) and Shwapno's Terms & Conditions (sections 11.5, 11.6 and 14.1) don't allow scraping or copying their content without permission. This data is shared for personal and research use only. Don't resell it.
