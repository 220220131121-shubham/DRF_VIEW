# 1️⃣ Function Based View

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

# 1️⃣ Problem: Django URL ko function chahiye

Django URL dispatcher internally **callable view** expect karta hai.

Example (FBV):

```python
path("hello/", hello_view)
```

Yaha `hello_view` ek function hai:

```python
def hello_view(request):
    ...
```

Execution:

```
request → function(request)
```

---

# 2️⃣ CBV me problem

CBV me hum class likhte hain:

```python
class HelloView(View):

    def get(self, request):
        return HttpResponse("Hello")
```

Agar directly URL me pass karein:

```python
path("hello/", HelloView)
```

Ye kaam nahi karega.

Reason:

```
HelloView = class
Django ko function chahiye
```

---

# 3️⃣ Solution: `as_view()`

Django CBV me ek method hota hai:

```
as_view()
```

Usage:

```python
path("hello/", HelloView.as_view())
```

Ye method internally **class ko callable function me convert karta hai**.

Conceptually:

```
class → function wrapper
```

---

# 4️⃣ Conceptual Implementation (Simplified)

Actual Django source code ka simplified idea:

```python
class View:

    @classmethod
    def as_view(cls):

        def view(request, *args, **kwargs):

            self = cls()

            return self.dispatch(request, *args, **kwargs)

        return view
```

Important observations:

| Step | Action                |
| ---- | --------------------- |
| 1    | class method call     |
| 2    | view function create  |
| 3    | class instance create |
| 4    | dispatch() call       |

---

# 5️⃣ Execution Flow

Request aata hai:

```
GET /hello/
```

Execution:

```
URL Router
     ↓
HelloView.as_view()
     ↓
function view(request)
     ↓
HelloView() instance create
     ↓
dispatch()
     ↓
get()
     ↓
Response
```

---

# 6️⃣ Step-by-Step Runtime Flow

Example code:

```python
class HelloView(View):

    def get(self, request):
        return HttpResponse("Hello")
```

Request arrives:

```
GET /hello/
```

Execution sequence:

```
1 URL matched
2 HelloView.as_view() returned function
3 Django calls function(request)
4 Class instance created → HelloView()
5 dispatch() executed
6 HTTP method detected → GET
7 get() called
8 Response returned
```

---

# 7️⃣ Important Insight

`as_view()` **class ko function banata nahi hai permanently**, balki:

```
function wrapper create karta hai
```

Har request par:

```
new class instance create hota hai
```

Iska advantage:

```
request specific state store ho sakta hai
```

---

# 8️⃣ Mental Model

CBV execution ko aise visualize karo:

```
URL
 ↓
as_view()
 ↓
wrapper function
 ↓
class instance
 ↓
dispatch()
 ↓
HTTP method handler
```

---

# 9️⃣ Why `as_view()` Important for DRF

DRF ka `APIView` bhi isi mechanism par build hai.

Example:

```python
class ProductView(APIView):
```

URL:

```python
path("products/", ProductView.as_view())
```

Execution same hota hai:

```
APIView.as_view()
       ↓
dispatch()
       ↓
get()/post()
```

---

# 🔑 Short Summary

```
Django URL needs a function
CBV is a class

as_view() converts:

class → callable function wrapper
```

Execution chain:

```
as_view()
   ↓
view function
   ↓
class instance
   ↓
dispatch()
   ↓
get/post
```

---

# 1️⃣ Problem dispatch solve karta hai

Request jab server par aati hai, to usme ek **HTTP method** hota hai:

Examples:

```
GET
POST
PUT
PATCH
DELETE
```

CBV me har method ke liye alag function hota hai:

```
get()
post()
put()
delete()
```

Question:

```
Kaise decide hoga ki kaunsa method call ho?
```

Answer:

```
dispatch()
```

---

# 2️⃣ Request Lifecycle (till dispatch)

Agar user request bhejta hai:

```
GET /products/
```

Execution pipeline:

```
URL router
   ↓
ProductView.as_view()
   ↓
view(request)
   ↓
ProductView instance
   ↓
dispatch(request)
```

Yaha se **dispatch() ka role start hota hai**.

---

# 3️⃣ dispatch() ka simplified logic

Django internally kuch aisa logic use karta hai.

Simplified conceptual code:

```python
class View:

    def dispatch(self, request, *args, **kwargs):

        if request.method == "GET":
            return self.get(request, *args, **kwargs)

        elif request.method == "POST":
            return self.post(request, *args, **kwargs)

        elif request.method == "PUT":
            return self.put(request, *args, **kwargs)

        elif request.method == "DELETE":
            return self.delete(request, *args, **kwargs)
```

Actual implementation thodi different hoti hai but concept same hai.

---

# 4️⃣ Real Django Implementation Idea

Django dynamically method lookup karta hai.

Conceptually:

```python
def dispatch(self, request, *args, **kwargs):

    method = request.method.lower()

    handler = getattr(self, method)

    return handler(request, *args, **kwargs)
```

Example:

```
request.method = "GET"
```

Convert:

```
"GET" → "get"
```

Then:

```
self.get()
```

call hota hai.

---

# 5️⃣ Example CBV Execution

View:

```python
class ProductView(View):

    def get(self, request):
        return HttpResponse("GET products")

    def post(self, request):
        return HttpResponse("Create product")
```

Request:

```
GET /products/
```

Execution:

```
dispatch()
   ↓
method = "GET"
   ↓
handler = get
   ↓
get()
```

Response:

```
GET products
```

---

# 6️⃣ Example with POST

Request:

```
POST /products/
```

Execution:

```
dispatch()
   ↓
method = "POST"
   ↓
handler = post
   ↓
post()
```

Response:

```
Create product
```

---

# 7️⃣ What if Method Not Implemented?

Example view:

```python
class ProductView(View):

    def get(self, request):
        ...
```

User sends:

```
POST /products/
```

Dispatch try karega:

```
self.post()
```

Lekin `post()` exist nahi karta.

Result:

```
405 Method Not Allowed
```

---

# 8️⃣ Full CBV Execution Diagram

Complete pipeline:

```
Client Request
      ↓
URL Router
      ↓
View.as_view()
      ↓
View instance
      ↓
dispatch()
      ↓
identify HTTP method
      ↓
call handler (get/post/put/delete)
      ↓
Response returned
```

---

# 9️⃣ DRF me dispatch ka role

DRF `APIView` bhi **dispatch() mechanism use karta hai**, lekin usse pehle kuch extra steps run karta hai.

DRF pipeline:

```
request
   ↓
dispatch()
   ↓
initial()
   ↓
authentication
permissions
throttling
   ↓
get()/post()
   ↓
Response
```

Isliye DRF views me extra features automatically milte hain.

---

# 🔑 Short Mental Model

```
dispatch() = HTTP router inside class
```

Mapping:

```
GET → get()
POST → post()
PUT → put()
DELETE → delete()
```

---
