# Danira Competitor Product Monitoring

A lightweight n8n workflow for monitoring competitor product prices and stock availability, detecting meaningful changes, and sending Telegram alerts.

Built as a practical Danira case study for online stores and e-commerce operations.

## What It Does

The workflow:

1. Runs on a schedule.
2. Reads active competitor products from Google Sheets.
3. Processes products one by one.
4. Opens each product page using Airtop.
5. Extracts:
   - Product name
   - Current price
   - Stock status
6. Compares the current values with the previously stored values.
7. Detects one of these events:
   - `INITIAL`
   - `PRICE_DOWN`
   - `PRICE_UP`
   - `OUT_OF_STOCK`
   - `BACK_IN_STOCK`
   - `NO_CHANGE`
8. Sends a Telegram alert only when an important change is detected.
9. Updates the product row in Google Sheets with the latest price, stock status, and check time.

## Workflow Architecture

```text
Schedule Trigger
    ↓
Read Products From Google Sheets
    ↓
Keep Active Products Only
    ↓
Process Each Product Individually
    ↓
Extract Product Data With Airtop
    ↓
Structure Extracted Product Data
    ↓
Compare With Previous Values
    ↓
Is Alert Event?
   ↙        ↘
Yes         No
 ↓           ↓
Telegram     Update Google Sheets
 ↓
Update Google Sheets
    ↓
Next Product
```

## Tech Stack

- n8n
- Google Sheets
- Airtop
- Telegram Bot

No AI model is used for price comparison or event detection. The comparison logic is deterministic.

## Google Sheet Structure

Create a sheet with these columns:

```text
id
competitor_name
product_name
product_url
last_price
last_stock_status
last_checked
active
```

Example:

```text
1 | Ulta Beauty | Example Product | https://example.com/product | 25 | in stock | 2026-10-12T10:00:00Z | TRUE
```

## Setup

### 1. Import the Workflow

Import:

```text
danira-competitor-monitoring-sanitized.json
```

into your n8n instance.

### 2. Connect Credentials

You will need to configure your own credentials for:

- Google Sheets
- Airtop
- Telegram

No API keys or credentials are included in the public workflow file.

### 3. Configure Google Sheets

In both Google Sheets nodes:

- Select your Google account.
- Select your spreadsheet.
- Select the worksheet containing the product data.

The workflow matches rows using the `id` column.

### 4. Configure Airtop

Connect your Airtop API credential.

The Airtop node receives `product_url` and extracts structured product data from the product page.

Expected output:

```json
{
  "product_name": "Example Product",
  "price": 25,
  "stock_status": "in stock"
}
```

### 5. Configure Telegram

Create a Telegram bot using BotFather and connect the bot credential in n8n.

Set your Telegram Chat ID in the `Send Telegram Alert` node.

Example alert:

```text
Competitor Alert

Event: PRICE_DOWN
Competitor: Store A
Product: Example Product

Previous price: 30
Current price: 25
Price change: -16.67%

Previous stock status: in stock
Current stock status: in stock

URL: https://example.com/product
```

## Event Detection Logic

The workflow detects:

### `INITIAL`
No previous price or stock status exists.

### `PRICE_DOWN`
The current price is lower than the previous price.

### `PRICE_UP`
The current price is higher than the previous price.

### `OUT_OF_STOCK`
The product was previously available and is now unavailable.

### `BACK_IN_STOCK`
The product was previously unavailable and is now available.

### `NO_CHANGE`
No meaningful price or stock change was detected.

Telegram alerts are sent only for:

```text
PRICE_DOWN
PRICE_UP
OUT_OF_STOCK
BACK_IN_STOCK
```

## Error Handling

The Airtop node is configured so that a failed product does not necessarily stop the entire workflow.

For production use, additional logging and retry policies may be added depending on the target websites and monitoring frequency.

## Current Scope

This repository contains the MVP version.

Included:

- Competitor product monitoring
- Price extraction
- Stock status extraction
- Price change percentage
- Change detection
- Telegram alerts
- Google Sheets state updates

Not included:

- WooCommerce price comparison
- Automatic price changes
- Dashboard
- SaaS
- Multi-tenant architecture
- AI-generated recommendations
- Large-scale monitoring infrastructure

## Limitations

- Product page structures differ between websites.
- Some websites may block automated browser access.
- Extraction quality depends on the target page and Airtop's ability to read it.
- Stock wording may vary between stores.
- This MVP stores only the latest known state in Google Sheets; it does not maintain a full historical database.

## Suggested Test

Before using the workflow regularly:

1. Add at least 5 real product URLs from at least 2 websites.
2. Run the workflow once to establish the baseline.
3. Manually change a stored `last_price` value.
4. Run the workflow again.
5. Confirm that:
   - The change is detected.
   - The correct event type is generated.
   - Telegram receives the alert.
   - Google Sheets is updated with the latest value.

## Repository Files

```text
README.md
danira-competitor-monitoring-sanitized.json
```

## About Danira

Danira is a Business Automation & AI Operations Studio focused on practical automation for WordPress, WooCommerce, online businesses, integrations, and operational workflows.

The principle behind this project is:

> Problem first. Tool second.

The goal is not to demonstrate n8n or Airtop for their own sake, but to remove repetitive competitor-monitoring work from e-commerce operations.

## License

You may use and adapt this workflow for learning, internal automation, and client projects.

If you publish a modified version, attribution to Danira is appreciated.
