# Code Execution Flow: From Startup to Frontend

## 1. Application Entry Point (main.py)

### Initial Setup
```python
import uvicorn
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
```
- Imports essential libraries
- uvicorn: ASGI server implementation
- FastAPI: Web framework
- CORSMiddleware: Handles Cross-Origin Resource Sharing

### FastAPI App Configuration
```python
app = FastAPI(
    title="E-Commerce Mock Data API",
    description="A powerful API that generates realistic mock data...",
    version="1.0.0",
    docs_url="/",
)
```
- Creates FastAPI application instance
- Sets API metadata
- Configures Swagger UI as root endpoint

### CORS Configuration
```python
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```
- Enables cross-origin requests
- Allows all origins, methods, and headers
- Essential for frontend-backend communication

### Router Registration
```python
app.include_router(users.router, prefix="/api/v1", tags=["Users"])
app.include_router(products.router, prefix="/api/v1", tags=["Products"])
# ... more routers
```
- Registers all API endpoint routers
- Adds "/api/v1" prefix to all routes
- Groups endpoints by tags for documentation

## 2. Core Configuration (app/core/config.py)

### Settings Class
```python
class Settings(BaseModel):
    APP_NAME: str = "E-Commerce Mock Data API"
    DEBUG: bool = True
    HOST: str = "0.0.0.0"
    PORT: int = 8000
    DEFAULT_LIMIT: int = 1000
    MAX_LIMIT: int = 10000
    DATE_START: str = "2023-01-01"
    DATE_END: str = "2023-12-31"
    SUPPORTED_FORMATS: list = ["json", "csv", "xml"]
    DEFAULT_FORMAT: str = "json"
```
- Defines global application settings
- Uses Pydantic for type validation
- Sets default values for configuration

## 3. Base Generator (app/generators/generator.py)

### DataGenerator Class
```python
class DataGenerator:
    def __init__(self, seed: Optional[int] = None):
        self.seed = seed if seed is not None else random.randint(1, 10000)
        self.fake = Faker()
        Faker.seed(self.seed)
        random.seed(self.seed)
```
- Initializes base generator with optional seed
- Sets up Faker instance for data generation
- Ensures reproducible data with seed

### ID Management
```python
def _store_ids(self, entity_type: str, ids: List[UUID]):
    setattr(self, f"{entity_type}_ids", ids)

def _get_random_id(self, entity_type: str) -> UUID:
    ids = getattr(self, f"{entity_type}_ids", [])
    return random.choice(ids) if ids else uuid4()
```
- Manages entity relationships
- Stores generated IDs for reference
- Retrieves random IDs for relationships

### Data Format Conversion
```python
def to_json(self, data: List[Dict]) -> str:
    return json.dumps(data, default=str, indent=2)

def to_csv(self, data: List[Dict]) -> str:
    output = io.StringIO()
    writer = csv.DictWriter(output, fieldnames=data[0].keys())
    writer.writeheader()
    for row in data:
        processed_row = {}
        for k, v in row.items():
            if isinstance(v, (dict, list)):
                processed_row[k] = json.dumps(v, default=str)
            elif isinstance(v, (datetime, UUID)):
                processed_row[k] = str(v)
            else:
                processed_row[k] = v
        writer.writerow(processed_row)
    return output.getvalue()
```
- Converts data to different formats
- Handles complex data types
- Ensures proper string representation

## 4. Entity Generators

### User Generator (app/generators/user_generator.py)
```python
def generate_users(self, limit: int = 1000) -> List[Dict]:
    users = []
    for _ in range(limit):
        created_at = self._get_random_date()
        user = {
            "user_id": uuid4(),
            "email": self.fake.email(),
            "username": self.fake.user_name(),
            # ... more fields
        }
        users.append(user)
    self._store_ids("user", [u["user_id"] for u in users])
    return users
```
- Generates user data
- Stores user IDs for relationships
- Creates realistic user profiles

### Product Generator (app/generators/product_generator.py)
```python
def generate_products(self, limit: int = 1000) -> List[Dict]:
    if not self.category_ids:
        self.generate_categories()
    if not self.vendor_ids:
        self.generate_vendors()
    
    products = []
    for _ in range(limit):
        price = round(Decimal(random.uniform(9.99, 999.99)), 2)
        product = {
            "product_id": uuid4(),
            "name": f"{random.choice(product_adjectives)} {random.choice(product_types)}",
            "category_id": self._get_random_id("category"),
            "vendor_id": self._get_random_id("vendor"),
            # ... more fields
        }
        products.append(product)
    self._store_ids("product", [p["product_id"] for p in products])
    return products
```
- Ensures prerequisite data exists
- Generates product data
- Maintains relationships with categories and vendors

## 5. API Routers

### Base Router Setup (app/routers/base.py)
```python
def get_format_params(
    format: str = Query("json", description="Response format (json, csv, xml)"),
    limit: int = Query(1000, description="Number of records", ge=1, le=10000),
    seed: Optional[int] = Query(None, description="Random seed")
) -> FormatParams:
    return FormatParams(format=format, limit=limit, seed=seed)

def create_response(generator: DataGenerator, data: List[Dict],
                   format_params: FormatParams, root_name: str,
                   item_name: str):
    content = generator.generate_response(data, format_params.format,
                                       root_name, item_name)
    # Set content type and filename
    return StreamingResponse(io.StringIO(content),
                           media_type=media_type,
                           headers={"Content-Disposition": f"attachment; filename={filename}"})
```
- Handles common query parameters
- Creates standardized responses
- Manages file downloads

### Entity-Specific Routers
```python
@router.get("/users", summary="Generate user data")
async def get_users(format_params: FormatParams = Depends(get_format_params)):
    generator = UserGenerator(seed=format_params.seed)
    users = generator.generate_users(limit=format_params.limit)
    return create_response(generator, users, format_params,
                         root_name="users", item_name="user")
```
- Defines API endpoints
- Handles data generation
- Returns formatted response

## 6. Frontend Integration (React)

### App Component (src/App.tsx)
```typescript
function App() {
  return (
    <div className="min-h-screen bg-gray-100 flex items-center justify-center">
      <p>Start prompting (or editing) to see magic happen :)</p>
    </div>
  );
}
```
- Basic React component
- Uses Tailwind CSS for styling

### Main Entry (src/main.tsx)
```typescript
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import App from './App.tsx';
import './index.css';

createRoot(document.getElementById('root')!).render(
  <StrictMode>
    <App />
  </StrictMode>
);
```
- Sets up React application
- Mounts App component
- Enables strict mode

## 7. Request Flow Example

1. Client makes request:
```bash
GET /api/v1/users?format=json&limit=1000&seed=42
```

2. FastAPI processes request:
- Validates query parameters
- Creates FormatParams instance
- Routes to appropriate handler

3. Router handles request:
- Initializes UserGenerator with seed
- Generates user data
- Formats response

4. Response returned:
- Proper content type set
- Data formatted as requested
- Downloaded or displayed

## 8. Data Generation Dependencies

The system follows this generation order:

1. Independent Entities:
   ```python
   users = user_generator.generate_users()
   categories = product_generator.generate_categories()
   vendors = product_generator.generate_vendors()
   warehouses = logistics_generator.generate_warehouses()
   ```

2. Products and Inventory:
   ```python
   products = product_generator.generate_products()
   inventory = logistics_generator.generate_inventory()
   ```

3. Orders and Related:
   ```python
   orders = order_generator.generate_orders()
   payments = payment_generator.generate_payments(orders)
   shipping = logistics_generator.generate_shipping(orders)
   ```

4. Engagement Data:
   ```python
   reviews = engagement_generator.generate_reviews()
   tickets = engagement_generator.generate_support_tickets()
   promotions = engagement_generator.generate_promotions()
   ```

This ensures all necessary relationships are maintained throughout the data generation process.
