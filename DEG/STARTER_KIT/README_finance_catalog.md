# Strapi Catalog Utility – Finance Catalog Import

This utility helps you insert catalog data (like providers and financial items) into a Strapi instance using CSV files. It supports domains like `deg:finance`, making it easier to populate your backend database reliably.

---

## 📁 Folder Structure

This guide assumes you are in the `strapi-catalog-utility/deg-finance-catalog-utility` directory within the [beckn-utilities](https://github.com/beckn/beckn-utilities) repo:

```
beckn-utilities/
└── strapi-catalog-utility/
    └── deg-finance-catalog-utility/
        ├── build/
        ├── samples/
        │   ├── providers.csv
        │   └── items.csv
        ├── .env
        └── ...
```

---

## ⚡ Quick Start – Finance Flow

### 1. Clone and Enter Directory

```bash
git clone https://github.com/beckn/beckn-utilities.git
cd strapi-catalog-utility/deg-finance-catalog-utility
```

### 2. Install Dependencies

```bash
npm install
```

### 3. Build the Project

```bash
npm run build
```

### 4. Run the Catalog Uploader for Finance

```bash
node build/index.js providers.csv items.csv
```

This command will read your provider and item CSVs and push the finance-related catalog data into your Strapi server.

---

## ⚙️ Configuration

Before running, make sure:

1. You have a `.env` file in the root (copy from `.samplenv` if needed).
2. Edit the file and fill in:
   - `STRAPI_BASE_URL=https://your-strapi-instance.com`
   - `STRAPI_API_TOKEN=your_strapi_token_here`

---

## 📄 CSV File Format & Details

The utility expects **two CSV files** as input:

### ✅ `providers.csv`

Each row defines a provider (e.g., a bank, insurer, or financial service). Columns:

| DOMAIN      | provider_name | provider_short_desc | provider_long_desc                                      | provider_Logo_image_url       | provider_id | provider_uri         | Address          | City   | State       | Country | zip    | gps                 |
| ----------- | ------------- | ------------------- | ------------------------------------------------------- | ----------------------------- | ----------- | -------------------- | ---------------- | ------ | ----------- | ------- | ------ | ------------------- |
| deg:finance | Idfc Bank     | 299                 | Idfc Bank is committed to playing a significant role... | https://Idfcbank.com/logo.png | Idfc-bank   | https://Idfcbank.com | 101 Green Avenue | Mumbai | Maharashtra | India   | 400001 | 19.076090,72.877426 |

### ✅ `items.csv`

Each row defines a financial item (e.g., loan plan, insurance policy) linked to a provider. Columns:

| provider_name | Item_name | short_desc | long_desc        | Logo_image_url                | max_quantity | min_quantity | tag_name | code | value | category name     | category code | sku  | min_price | max_price | stock_quantity | stock_status | fulfillments | base_price | tax | currency | quantity_unit |
| ------------- | --------- | ---------- | ---------------- | ----------------------------- | ------------ | ------------ | -------- | ---- | ----- | ----------------- | ------------- | ---- | --------- | --------- | -------------- | ------------ | ------------ | ---------- | --- | -------- | ------------- |
| Idfc Bank     | 9         | 299        | idfc finace bank | https://example.com/image.jpg | 1            | 1            |          |      |       | loan_type_battery | loan_type     | lt12 | 16        | 16        | 10             |              |              |            |     | INR      |               |
| Idfc Bank     | 12        | 399        | idfc finace bank | https://example.com/image.jpg | 1            | 1            |          |      |       | loan_type_battery | loan_type     | lt9  | 16        | 16        | 10             |              |              |            |     | INR      |               |

> 🔁 **Ensure that `provider_name` in items matches `provider_name` in providers**, or the data linkage will fail.

You can find editable sample files in the `/samples` directory.

---

## 🧠 Developer Tips

- Always verify the Strapi domain you're working with (e.g., `deg:finance`) aligns with your collections.
- This utility is **tightly coupled with your Strapi schema**. If you change collections or field names, update the code accordingly.
- Some fields (like fulfillments) may be hardcoded — feel free to enhance support for dynamic attributes.

---

## 📜 License

This utility is part of the [Beckn Protocol](https://github.com/beckn/beckn-utilities) and licensed under [MIT](https://github.com/beckn/beckn-utilities/blob/main/LICENSE).
