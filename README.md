# bikestores_raw
[![Ask DeepWiki](https://deepwiki.com/badge.svg)](https://deepwiki.com/cutiepie25/bikestores_raw)

Capa **raw** (datos crudos) de la base de datos de ejemplo **BikeStores**, exportada a CSV. Sirve como punto de partida para ejercicios de ingeniería de datos (ETL/ELT, modelado dimensional, data warehouse, dashboards).

Cada tabla trae dos columnas de auditoría de la ingesta:

- `_ingestion_ts`: fecha y hora en que se cargó el registro.
- `_source`: origen del dato (`BikeStores_LoadData.sql`).

## Tablas

| Archivo | Registros | Columnas |
|---------|----------:|----------|
| `brands.csv` | 9 | `brand_id`, `brand_name` |
| `categories.csv` | 7 | `category_id`, `category_name` |
| `products.csv` | 321 | `product_id`, `product_name`, `brand_id`, `category_id`, `model_year`, `list_price` |
| `stores.csv` | 1 | `store_id`, `store_name`, `phone`, `email`, `street`, `city`, `state`, `zip_code` |
| `staffs.csv` | 10 | `staff_id`, `first_name`, `last_name`, `email`, `phone`, `active`, `store_id`, `manager_id` |
| `stocks.csv` | 939 | `store_id`, `product_id`, `quantity` |
| `customers.csv` | 1445 | `customer_id`, `first_name`, `last_name`, `phone`, `email`, `street`, `city`, `state`, `zip_code` |
| `orders.csv` | 1615 | `order_id`, `customer_id`, `order_status`, `order_date`, `required_date`, `shipped_date`, `store_id`, `staff_id` |
| `order_items.csv` | 4722 | `order_id`, `item_id`, `product_id`, `quantity`, `list_price`, `discount` |

## Relaciones

```
brands ──┐
         ├──< products >──┬──< stocks >── stores ──< staffs (manager_id → staffs)
categories┘               │                  │
                          └──< order_items >── orders >── customers
                                               │
                                               └── staffs
```

- `products.brand_id` → `brands.brand_id`
- `products.category_id` → `categories.category_id`
- `stocks.(store_id, product_id)` → `stores`, `products`
- `orders.customer_id` → `customers`; `orders.store_id` → `stores`; `orders.staff_id` → `staffs`
- `order_items.order_id` → `orders`; `order_items.product_id` → `products`
- `staffs.manager_id` → `staffs.staff_id`

## Ejemplo de uso

```python
import pandas as pd

orders = pd.read_csv("orders.csv", parse_dates=["order_date"])
items = pd.read_csv("order_items.csv")

items["total"] = items["quantity"] * items["list_price"] * (1 - items["discount"])
ventas = items.merge(orders, on="order_id")
print(ventas.groupby(ventas["order_date"].dt.year)["total"].sum())
```
