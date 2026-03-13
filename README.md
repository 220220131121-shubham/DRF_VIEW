## DRF Views — Introduction

Ab tak humne **Model** aur **Serializer** dekha.
Next step hai **View**, jahan API request handle hoti hai.

Framework: **Django REST Framework**

Basic pipeline:

```text
Client Request
      ↓
DRF View
      ↓
Serializer
      ↓
Model / Database
      ↓
JSON Response
```

---

# 1️⃣ DRF View kya karta hai

View ka main kaam:

1. Request receive karna
2. Serializer use karna
3. Database se data lena / save karna
4. Response return karna

Example flow:

```text
GET /products/
        ↓
View
        ↓
Product.objects.all()
        ↓
Serializer
        ↓
JSON response
```

---

# 2️⃣ DRF me Views ke Types

DRF me multiple view abstractions hain (complexity gradually kam hoti hai).

| Type           | Use                  |
| -------------- | -------------------- |
| APIView        | basic DRF view       |
| GenericAPIView | reusable logic       |
| Mixins         | CRUD helpers         |
| Generic Views  | mixin + generic view |
| ViewSets       | full CRUD automation |

Learning order usually:

```text
APIView
   ↓
GenericAPIView
   ↓
ViewSets
```

---

# 3️⃣ Basic APIView Example

```python
from rest_framework.views import APIView
from rest_framework.response import Response
from .models import Product
from .serializers import ProductSerializer


class ProductView(APIView):

    def get(self, request):

        products = Product.objects.all()

        serializer = ProductSerializer(products, many=True)

        return Response(serializer.data)
```

---

# 4️⃣ Important Components

### `APIView`

DRF ka base view class.

```python
from rest_framework.views import APIView
```

Ye provide karta hai:

* request parsing
* authentication
* permission handling
* response formatting

---

### `Response`

DRF response object.

```python
from rest_framework.response import Response
```

Ye automatically:

```text
Python data → JSON
```

convert karta hai.

Example:

```python
return Response(serializer.data)
```

---

# 5️⃣ Why APIView instead of Django View?

Normal **Django** view:

```python
def view(request):
```

DRF view:

```python
class View(APIView):
```

APIView extra features deta hai:

* JSON parsing
* API authentication
* content negotiation
* standardized responses

---

# 6️⃣ Minimal GET API Flow

```text
Client request
      ↓
APIView.get()
      ↓
Queryset
      ↓
Serializer
      ↓
Response()
      ↓
JSON output
```

Example output:

```json
[
  {
    "id": 1,
    "name": "Laptop",
    "price": 50000
  }
]
```

---

✅ **Short Summary**

```text
APIView = DRF ka base API view
Response = JSON response generator
View = request handling layer
```

---
## APIView Methods — `GET`, `POST`, `PUT`, `DELETE`

In **Django REST Framework**, `APIView` HTTP methods ko class methods ke through handle karta hai.
Har method ek specific **HTTP request type** ko map karta hai.

Basic structure:

```python
class ProductView(APIView):

    def get(self, request):
        ...

    def post(self, request):
        ...

    def put(self, request, pk):
        ...

    def delete(self, request, pk):
        ...
```

---

# 1️⃣ GET — Data Fetch Karna

Use case:

```text
GET /products/
GET /products/1/
```

Example:

```python
def get(self, request):

    products = Product.objects.all()

    serializer = ProductSerializer(products, many=True)

    return Response(serializer.data)
```

Flow:

```text
Client GET request
      ↓
Database query
      ↓
Serializer
      ↓
JSON response
```

---

# 2️⃣ POST — New Data Create Karna

Use case:

```text
POST /products/
```

Client JSON:

```json
{
 "name": "Phone",
 "price": 20000
}
```

Example:

```python
def post(self, request):

    serializer = ProductSerializer(data=request.data)

    if serializer.is_valid():
        serializer.save()
        return Response(serializer.data)

    return Response(serializer.errors)
```

Flow:

```text
Client JSON
     ↓
Serializer(data=request.data)
     ↓
is_valid()
     ↓
save()
     ↓
Database insert
```

---

# 3️⃣ PUT — Full Update

Use case:

```text
PUT /products/1/
```

Example:

```python
def put(self, request, pk):

    product = Product.objects.get(id=pk)

    serializer = ProductSerializer(product, data=request.data)

    if serializer.is_valid():
        serializer.save()
        return Response(serializer.data)

    return Response(serializer.errors)
```

Flow:

```text
Find object
     ↓
Serializer(instance, data)
     ↓
Validation
     ↓
Update object
```

---

# 4️⃣ DELETE — Data Remove

Use case:

```text
DELETE /products/1/
```

Example:

```python
def delete(self, request, pk):

    product = Product.objects.get(id=pk)

    product.delete()

    return Response({"message": "Deleted"})
```

Flow:

```text
Find object
     ↓
delete()
     ↓
Response
```

---

# Full CRUD View Example

```python
from rest_framework.views import APIView
from rest_framework.response import Response


class ProductView(APIView):

    def get(self, request):

        products = Product.objects.all()

        serializer = ProductSerializer(products, many=True)

        return Response(serializer.data)


    def post(self, request):

        serializer = ProductSerializer(data=request.data)

        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)

        return Response(serializer.errors)
```

---

# HTTP Method Mapping

| Method | Purpose     |
| ------ | ----------- |
| GET    | read data   |
| POST   | create data |
| PUT    | update data |
| DELETE | remove data |

---

✅ **Short Summary**

```text
APIView methods map to HTTP requests

GET    → read
POST   → create
PUT    → update
DELETE → remove
```

---
