# Strapi Catalog Utility – Retail Catalog Import

This utility is built to help you upload retail catalog data (like store providers and retail products) into a Strapi backend using CSV files. It targets the `deg:retail` domain specifically.

---

## 📁 Folder Structure

This guide assumes you're in the `strapi-catalog-utility/deg-retail-catalog-utility` folder inside the [beckn-utilities](https://github.com/beckn/beckn-utilities) repository:

```
beckn-utilities/
└── strapi-catalog-utility/
    └── deg-retail-catalog-utility/
        ├── build/
        ├── samples/
        │   ├── providers.csv
        │   └── items.csv
        ├── .env
        └── ...
```

---

## ⚙️ Clone & Setup

### Step 1: Clone the Repository

```bash
git clone https://github.com/beckn/beckn-utilities.git
cd strapi-catalog-utility/deg-retail-catalog-utility
```

### Step 2: Install Required Libraries

```bash
npm install
```

### Step 3: Build the Project

```bash
npm run build
```

### Step 4: Configure `.env`

```bash
cp .samplenv .env
```

Then edit `.env` and set:

```env
STRAPI_BASE_URL=https://your-strapi-url.com
STRAPI_API_TOKEN=your_strapi_api_token
```

### Step 5: Run the Retail Catalog Uploader

```bash
node build/index.js providers.csv items.csv
```

---

## 📄 CSV File Format & Details

### ✅ `providers.csv`

Each row defines a retail provider (store or business). Columns:

| DOMAIN     | provider_name  | provider_short_desc   | provider_long_desc                        | provider_Logo_image_url        | provider_id      | provider_uri          | Address       | City   | State | Country | zip   | gps              |
| ---------- | -------------- | --------------------- | ----------------------------------------- | ------------------------------ | ---------------- | --------------------- | ------------- | ------ | ----- | ------- | ----- | ---------------- |
| deg:retail | VoltTrade Inc. | Battery Trade Experts | Specialized in battery purchase and sales | https://volttrade.com/logo.png | volttrade.retail | https://volttrade.com | 101 Power Ave | Austin | Texas | USA     | 73301 | 30.2672,-97.7431 |

### ✅ `items.csv`

Each row defines a product linked to a provider. Columns:

| provider_name  | Item_name              | short_desc   | long_desc                            | Logo_image_url                | max_quantity | min_quantity | tag_name         | code             | value | category name      | category code | sku      | min_price | max_price | stock_quantity | stock_status | fulfillments       | base_price | tax | currency | quantity_unit |
| -------------- | ---------------------- | ------------ | ------------------------------------ | ----------------------------- | ------------ | ------------ | ---------------- | ---------------- | ----- | ------------------ | ------------- | -------- | --------- | --------- | -------------- | ------------ | ------------------ | ---------- | --- | -------- | ------------- |
| VoltTrade Inc. | 80Ah Lithium Battery 1 | 80Ah battery | Rechargeable 80Ah lithium battery... | https://example.com/image.jpg | 1            | 1            | Battery Capacity | Battery Capacity | 80    | DEG Retail Battery | DEG_RETAIL_1  | BAT-9500 | 129       | 129       | 1000           | In Stock     | Delivery in 5 days | 124        | 5   | USD      | USD           |

> 🔁 **Ensure that `provider_name` in items matches exactly with `provider_name` in providers** to avoid upload errors.

---

## 💡 Developer Tips

- This branch handles the `deg:retail` domain.
- If your Strapi model changes (fields, collections), update the utility logic accordingly.
- You can customize relationships, tag mapping, or media handling in `recordCreator.ts`.

---

## 📜 License

This tool is a part of the [Beckn Protocol](https://github.com/beckn/beckn-utilities) suite, licensed under [MIT License](https://github.com/beckn/beckn-utilities/blob/main/LICENSE).
