# Strapi Catalog Utility – EV Catalog Import

This utility is designed to upload EV-related catalog data (like charging station providers and items) into a Strapi instance. It uses CSVs and targets the `deg:ev` domain.

---

## 📁 Folder Structure

This guide assumes you're in the `strapi-catalog-utility/deg-ev-catalog-utility` folder inside the [beckn-utilities](https://github.com/beckn/beckn-utilities) repository:

```
beckn-utilities/
└── strapi-catalog-utility/
    └── deg-ev-catalog-utility/
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
cd strapi-catalog-utility/deg-ev-catalog-utility
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

Defines a **charging provider** (EV station/service). Example:

| DOMAIN | provider_name    | provider_short_desc         | provider_long_desc                                                                       | provider_Logo_image_url               | provider_id      | provider_uri                 | Address         | City    | State    | Country | zip   | gps                  |
| ------ | ---------------- | --------------------------- | ---------------------------------------------------------------------------------------- | ------------------------------------- | ---------------- | ---------------------------- | --------------- | ------- | -------- | ------- | ----- | -------------------- |
| deg:ev | MotorCity Charge | Leading EV charging network | MotorCity Charge provides efficient and fast EV charging solutions in Detroit, Michigan. | https://motorcity-charge.com/logo.png | motorcity-charge | https://motorcity-charge.com | 448 Main Street | Detroit | Michigan | USA     | 50189 | 41.299211,-75.718899 |

### ✅ `items.csv`

Defines a **charging product/item linked to a provider**. Example:

| provider_name    | item_name                       | short_desc      | long_desc                                                                                      | logo_image_url                            | max_quantity | min_quantity | tag_name    | code      | value       | category_name | category_code | sku   | min_price | max_price | stock_quantity | stock_status | fulfillments | base_price | tax   | currency | quantity_unit |
| ---------------- | ------------------------------- | --------------- | ---------------------------------------------------------------------------------------------- | ----------------------------------------- | ------------ | ------------ | ----------- | --------- | ----------- | ------------- | ------------- | ----- | --------- | --------- | -------------- | ------------ | ------------ | ---------- | ----- | -------- | ------------- |
| MotorCity Charge | MotorCity Charge Fast Charger 1 | Fast EV Charger | MotorCity Charge Fast Charger 1 supports high-speed charging with multiple port compatibility. | https://motorcity-charge.com/charger1.png | 1            | 1            | Type2, GB/T | port_type | Type2, GB/T | EV Chargers   | EV-CAT        | MOT-1 | 160       | 160       | 10             | In Stock     | Onsite       | 148.15     | 11.85 | USD      | per kWh       |

> 🔁 **`provider_name` must match across both files to link items correctly.**

---

## 💡 Developer Tips

- This script is tailored for `deg:ev` domain
- You may customize logic for EV-specific fulfillments, tags, or relations in `recordCreator.ts`
- Watch for schema updates in Strapi

---

## 📜 License

This tool is a part of the [Beckn Protocol](https://github.com/beckn/beckn-utilities) suite, licensed under [MIT License](https://github.com/beckn/beckn-utilities/blob/main/LICENSE).
