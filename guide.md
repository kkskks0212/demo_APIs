# Developer's Guide to E-commerce Mock Data Generator

## System Overview

The E-commerce Mock Data Generator is a FastAPI application that creates realistic, interconnected datasets simulating a complete e-commerce ecosystem. This guide will walk you through the codebase structure, key components, and how they work together.

## Project Structure

```
app/
├── core/               # Core configuration
├── generators/         # Data generation modules
├── routers/           # API endpoint handlers
└── schemas/           # Pydantic models
```

## Core Components

### Configuration (`app/core/config.py`)

The configuration module defines global settings using Pydantic:
- Default data ranges (2023-01-01 to 2023-12-31)
- API limits (1,000 default, 10,000 maximum)
- Supported output formats (JSON, CSV, XML)

### Base Generator (`app/generators/generator.py`)

The `DataGenerator` class is the foundation of all data generation:
- Manages entity relationships through UUID storage
- Handles date generation within configured ranges
- Provides output formatting (JSON, CSV, XML)
- Maintains referential integrity between entities

Example usage:
```python
generator = DataGenerator(seed=42)  # Optional seed for reproducibility
data = generator.generate_users(limit=1000)
output = generator.to_json(data)  # or to_csv(), to_xml()
```

## Data Generation Flow

### 1. Entity Generation Order

The system follows a specific order to maintain referential integrity:

1. Independent Entities (no foreign keys):
   - Users
   - Categories
   - Vendors
   - Warehouses

2. Product-Related:
   - Products (needs categories, vendors)
   - Inventory (needs products, warehouses)

3. Order-Related:
   - Orders (needs users, products)
   - Payments (needs orders)
   - Shipping (needs orders, warehouses)
   - Returns (needs orders)

4. Engagement:
   - Reviews (needs users, products, orders)
   - Support Tickets (needs users, orders)
   - Promotions (needs products)

5. Analytics:
   - User Analytics (needs users)
   - Product Analytics (needs products)

### 2. Data Relationships

Example of how entities are connected:

```python
# Generate users first
user_generator = UserGenerator(seed=42)
users = user_generator.generate_users(limit=1000)
user_ids = [u["user_id"] for u in users]

# Generate products with categories and vendors
product_generator = ProductGenerator(seed=42)
products = product_generator.generate_products(limit=2000)
product_ids = [p["product_id"] for p in products]

# Generate orders using user and product IDs
order_generator = OrderGenerator(seed=42)
order_generator.user_ids = user_ids      # Set reference IDs
order_generator.product_ids = product_ids
orders = order_generator.generate_orders(limit=500)
```

## API Endpoints

### Common Features

All endpoints support:
- Format selection (JSON/CSV/XML)
- Record limit (1-10,000)
- Random seed for reproducibility

Example request:
```bash
curl "http://localhost:8000/api/v1/users?format=json&limit=1000&seed=42"
```

### Key Endpoints

1. User Management:
   - `/api/v1/users`: Basic user profiles
   - `/api/v1/wishlists`: User wishlists
   - `/api/v1/notifications`: User notifications

2. Product Catalog:
   - `/api/v1/products`: Product listings
   - `/api/v1/categories`: Product categories
   - `/api/v1/vendors`: Product vendors

3. Order Processing:
   - `/api/v1/orders`: Order details with items
   - `/api/v1/payments`: Payment transactions
   - `/api/v1/shipping`: Shipping information

4. Inventory & Logistics:
   - `/api/v1/warehouses`: Warehouse locations
   - `/api/v1/inventory`: Stock levels
   - `/api/v1/returns`: Return processing

5. Customer Engagement:
   - `/api/v1/reviews`: Product reviews
   - `/api/v1/support-tickets`: Customer support
   - `/api/v1/promotions`: Marketing campaigns

6. Analytics:
   - `/api/v1/analytics/users`: User behavior
   - `/api/v1/analytics/products`: Product performance

## Data Generation Examples

### 1. Generating Users

```python
from app.generators.user_generator import UserGenerator

generator = UserGenerator(seed=42)
users = generator.generate_users(limit=1000)

# Example user record:
{
    "user_id": "uuid-here",
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
}
```

### 2. Generating Orders

```python
from app.generators.order_generator import OrderGenerator

generator = OrderGenerator(seed=42)
generator.user_ids = [...]  # Set from previously generated users
generator.product_ids = [...] # Set from previously generated products

orders = generator.generate_orders(limit=500)

# Example order record:
{
    "order_id": "uuid-here",
    "user_id": "user-uuid-here",
    "status": "processing",
    "created_at": "2023-03-15T09:30:00",
    "shipping_address": "123 Main St",
    "shipping_method": "Express Shipping",
    "subtotal": "99.99",
    "tax": "8.00",
    "total": "107.99",
    "items": [
        {
            "order_item_id": "uuid-here",
            "product_id": "product-uuid-here",
            "quantity": 2,
            "unit_price": "49.99",
            "total": "99.98"
        }
    ]
}
```

## Best Practices

1. Data Generation:
   - Always set a seed for reproducible data
   - Generate prerequisite entities first
   - Maintain realistic relationships between entities
   - Use appropriate date ranges for timestamps

2. API Usage:
   - Start with smaller limits while testing
   - Use CSV format for large datasets
   - Include seed value in requests for consistent data
   - Handle pagination for large data sets

3. Performance:
   - Generate related entities in batches
   - Use appropriate limits based on entity relationships
   - Consider data volume when generating nested structures

## Common Use Cases

1. Building Tableau Dashboards:
```bash
# Generate sales data
curl -o sales.csv "http://localhost:8000/api/v1/orders?format=csv&limit=10000"

# Generate analytics
curl -o analytics.csv "http://localhost:8000/api/v1/analytics/products?format=csv&limit=10000"
```

2. Testing ETL Pipelines:
```bash
# Generate consistent datasets using seeds
curl "http://localhost:8000/api/v1/users?seed=42" > users.json
curl "http://localhost:8000/api/v1/orders?seed=42" > orders.json
```

3. Development Testing:
```bash
# Generate small dataset for testing
curl "http://localhost:8000/api/v1/products?limit=100"
```

## Extending the System

### Adding New Entities

1. Create generator class:
```python
class NewEntityGenerator(DataGenerator):
    def generate_entities(self, limit: int = 1000) -> List[Dict]:
        entities = []
        for _ in range(limit):
            entity = {
                "id": uuid4(),
                "name": self.fake.name(),
                "created_at": self._get_random_date()
            }
            entities.append(entity)
        return entities
```

2. Create router:
```python
@router.get("/new-entity", summary="Generate new entity data")
async def get_entities(format_params: FormatParams = Depends(get_format_params)):
    generator = NewEntityGenerator(seed=format_params.seed)
    entities = generator.generate_entities(limit=format_params.limit)
    return create_response(generator, entities, format_params,
                         root_name="entities", item_name="entity")
```

### Modifying Existing Entities

1. Add new fields to generator:
```python
def generate_users(self, limit: int = 1000) -> List[Dict]:
    users = super().generate_users(limit)
    for user in users:
        user["new_field"] = self.fake.word()
    return users
```

2. Update schema if using Pydantic:
```python
class UserBase(BaseModel):
    new_field: str
```

## Troubleshooting

Common issues and solutions:

1. Missing Foreign Keys:
   - Ensure prerequisite entities are generated first
   - Check ID storage in generator instances
   - Verify ID passing between generators

2. Inconsistent Data:
   - Use same seed value across related generators
   - Check date range configuration
   - Verify relationship logic in generators

3. Performance Issues:
   - Reduce limit for complex entities
   - Generate prerequisite data in smaller batches
   - Use appropriate format for data volume

## API Reference

For detailed API documentation, visit the Swagger UI at `http://localhost:8000/docs` after starting the server.
