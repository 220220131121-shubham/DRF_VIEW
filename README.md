```text
Class Based View kya hota hai (basic concept)
```

Agar ye clear ho gaya to baaki DRF CBV automatically samajh aata hai.

---

# 1️⃣ Pehle Problem Samjho — Function Based View

Django me traditionally views function hote the.

Example:

```python
from django.http import HttpResponse

def hello_view(request):
    return HttpResponse("Hello")
```

URL mapping:

```python
path("hello/", hello_view)
```

Flow:

```
request → function → response
```

Ye simple hai lekin ek limitation hai.

Agar multiple HTTP methods handle karne ho:

```
GET
POST
PUT
DELETE
```

to ek hi function me sab handle karna padta hai.

Example:

```python
def product_view(request):

    if request.method == "GET":
        return HttpResponse("Get Products")

    elif request.method == "POST":
        return HttpResponse("Create Product")
```

Problem:

```
1 function
multiple responsibilities
code messy ho jata hai
```

---

# 2️⃣ Solution — Class Based View

Class Based View me ek **class API endpoint ko represent karti hai**.

Example concept:

```
Class = API endpoint
Methods = HTTP actions
```

Example:

```python
class ProductView:

    def get(self, request):
        pass

    def post(self, request):
        pass
```

Ab har HTTP method ke liye **separate method** hai.

```
GET  → get()
POST → post()
PUT  → put()
DELETE → delete()
```

Result:

```
code clean
responsibility separated
maintainable
```

---

# 3️⃣ Real Django Example

Django ka base class:

```python
from django.views import View
```

Example:

```python
from django.http import HttpResponse
from django.views import View


class HelloView(View):

    def get(self, request):
        return HttpResponse("Hello GET")

    def post(self, request):
        return HttpResponse("Hello POST")
```

---

# 4️⃣ URL Mapping (Important)

Class ko directly URL me pass nahi kar sakte.

Instead use:

```python
as_view()
```

Example:

```python
from django.urls import path
from .views import HelloView

urlpatterns = [
    path("hello/", HelloView.as_view())
]
```

Reason:

```
Django expects a callable function
```

`as_view()` internally class ko function me convert karta hai.

Flow:

```
Request
   ↓
HelloView.as_view()
   ↓
HelloView instance
   ↓
dispatch()
   ↓
get() or post()
```

---

# 5️⃣ Simple Mental Model

CBV ko aise socho:

```
Class = Controller
Methods = HTTP handlers
```

Example:

```
ProductView

   get()    → fetch products
   post()   → create product
   put()    → update product
   delete() → delete product
```

---

# 6️⃣ Why DRF Uses CBV

DRF ka pura architecture classes par based hai.

Example:

```
APIView
GenericAPIView
ListAPIView
ModelViewSet
```

Ye sab **class inheritance chain** follow karte hain.

---

# 7️⃣ Short Summary

```
Function Based View
request → function → response

Class Based View
request → class → method → response
```

HTTP mapping:

```
GET    → get()
POST   → post()
PUT    → put()
DELETE → delete()
```

Aur URL me use karte hain:

```
as_view()
```

---

Agar tum bolo to **next topic: `as_view()` internally kaise kaam karta hai** samjha deta hoon — ye CBV ka **sabse confusing but important concept** hota hai.
