# E-Commerce Mock Data Generator API

A comprehensive FastAPI application that generates realistic mock data for an e-commerce ecosystem, including users, products, orders, payments, logistics, and analytics.

## Features

- 18 RESTful API endpoints for different e-commerce entities
- Realistic data generation using Faker library
- Support for multiple output formats (JSON, CSV, XML)
- Configurable data volume (up to 10,000 records per endpoint)
- Proper relational integrity between entities
- Reproducible data generation using seed values
- Swagger UI documentation

## Getting Started

### Prerequisites

- Python 3.7+
- pip

### Installation

1. Clone the repository
2. Install the required dependencies:

```bash
pip install fastapi uvicorn faker python-multipart jinja2 python-dateutil
```

3. Run the server:

```bash
uvicorn main:app --reload
```

The API will be available at `http://localhost:8000`

## API Endpoints

The API offers the following endpoints:

| Endpoint | Description |
|----------|-------------|
| `/api/v1/users` | Generate user data |
| `/api/v1/products` | Generate product data |
| `/api/v1/categories` | Generate product category data |
| `/api/v1/vendors` | Generate vendor data |
| `/api/v1/orders` | Generate order data with items |
| `/api/v1/carts` | Generate shopping cart data |
| `/api/v1/payments` | Generate payment data |
| `/api/v1/shipping` | Generate shipping data |
| `/api/v1/warehouses` | Generate warehouse data |
| `/api/v1/inventory` | Generate inventory data |
| `/api/v1/returns` | Generate return data |
| `/api/v1/reviews` | Generate product review data |
| `/api/v1/support-tickets` | Generate support ticket data |
| `/api/v1/promotions` | Generate promotion data |
| `/api/v1/subscriptions` | Generate subscription data |
| `/api/v1/wishlists` | Generate wishlist data |
| `/api/v1/notifications` | Generate notification data |
| `/api/v1/analytics/users` | Generate user analytics data |
| `/api/v1/analytics/products` | Generate product analytics data |

## Query Parameters

All endpoints support the following query parameters:

- `format`: Response format (json, csv, xml), default: json
- `limit`: Number of records to generate (1-10,000), default: 1000
- `seed`: Random seed for reproducible data

Example:
```
GET /api/v1/users?format=csv&limit=5000&seed=12345
```

## Example Outputs

### JSON (default)

```json
[
  {
    "user_id": "f47ac10b-58cc-4372-a567-0e02b2c3d479",
    "email": "john.doe@example.com",
    "username": "johndoe",
    "first_name": "John",
    "last_name": "Doe",
    "phone_number": "+1-555-123-4567",
    "address": "123 Main St",
    "city": "New York",
    "state": "NY",
    "postal_code": "10001",
    "country": "US",
    "created_at": "2023-01-15T10:30:45",
    "updated_at": "2023-02-20T14:22:10",
    "is_active": true,
    "loyalty_points": 325
  },
  ...
]
```

### CSV

```
user_id,email,username,first_name,last_name,phone_number,address,city,state,postal_code,country,created_at,updated_at,is_active,loyalty_points
f47ac10b-58cc-4372-a567-0e02b2c3d479,john.doe@example.com,johndoe,John,Doe,+1-555-123-4567,123 Main St,New York,NY,10001,US,2023-01-15T10:30:45,2023-02-20T14:22:10,true,325
...
```

### XML

```xml
<?xml version="1.0" ?>
<users>
  <user>
    <user_id>f47ac10b-58cc-4372-a567-0e02b2c3d479</user_id>
    <email>john.doe@example.com</email>
    <username>johndoe</username>
    <first_name>John</first_name>
    <last_name>Doe</last_name>
    <phone_number>+1-555-123-4567</phone_number>
    <address>123 Main St</address>
    <city>New York</city>
    <state>NY</state>
    <postal_code>10001</postal_code>
    <country>US</country>
    <created_at>2023-01-15T10:30:45</created_at>
    <updated_at>2023-02-20T14:22:10</updated_at>
    <is_active>true</is_active>
    <loyalty_points>325</loyalty_points>
  </user>
  ...
</users>
```

## Entity Relationships

The generated data maintains proper relational integrity:

- Users → Orders → Order Items → Products
- Users → Carts → Cart Items → Products
- Users → Wishlists → Wishlist Items → Products
- Products → Categories
- Products → Vendors
- Orders → Payments
- Orders → Shipping
- Products → Warehouses → Inventory
- Orders → Returns
- Users → Reviews → Products
- Users → Support Tickets
- Products → Promotions

This makes the data suitable for building complex Tableau dashboards or testing ETL pipelines.

## License

This project is licensed under the MIT License
