# Django訂單管理系統web開發

## 這是什麼&&我學到了什麼
這是我開發一個複雜的訂單管理網站的學習歷程

在這個個人專案中我學到了：
- 資料庫（sqlite3）基礎sql用法
- 網路協議（HTTP、HTTPS）
- 版本控制工具（Git）
- 作業系統操作（Linux）
- **Django框架**的深入應用，包括模型（Models）、視圖（Views）、模板（Templates）、表單（Forms）、用戶認證與權限管理等

這是我第一個大型且完整的個人專案，從0開始學習一個後端框架，自己寫前端html/css/js程式碼，自己寫後端/資料庫，五種程式語言我都是自己完成的，自己部署，自己維護。我在其中學到了非常多很重要東西、觀念，我現在會用到，我覺得我之後也離不開他們，比如**我學會了基礎且流行的開發流程，從建立專案，版本管理，分支管理，再到部署，另外還有網路協議的運作方式等等**。我之後參加的ais3 junior(教育部新心態資安暑期課程，我會放在另一個自主學習)的奪冠之旅也是基於這個專案，我覺得這可以算是我個人開發者生涯的起點。

## 前言

### 動機

在暑假前一陣子，我從母親那接到了一份不錯的委託：幫他們設計一個訂單管理系統，主要目的是取代目前使用的 Google表單和Excel。這個項目的最吸引我的是我可以獲得一定的零用錢，每當我有一個目標的時候我寫程式的動力就會大增，因此我在開發前期獲得了很大的效率提升，甚至很多時候都是熬夜開發

### 目標功能

- **用戶劃分**
我預設的目標是完成一整個可用的動態網站的前後端，並且有用戶權限劃分功能：普通用戶可以查看自己的訂單，而管理員用戶則可以看到所有人的訂單。
- **訂單管理 **
每個普通用戶都可以新增某個商品的訂單，並且可以打上備註，而管理員用戶則可以刪除編輯這些訂單
- **商品管理**
每個訂單都是綁定商品物件的，然後管理員用戶可以自行新增商品且打上品名、價格、上市日期、備註
- **用戶管理**
管理員用戶可以自行新增門市用戶，並且給他們帳號名、密碼、門市名

## 詳細開發過程

### 選擇後端語言與框架

在多方比較各個語言跟框架後，我決定選擇用Python語言搭配Django框架來開發這個訂單管理系統。理由是：

- **Django** 提供了完整的功能，如 ORM（物件關聯映射）、用戶認證、管理介面等，適合快速開發較複雜的項目。
- 相較於 Flask，Django 更適合應對較為複雜的應用需求。
- **Python** 的語法簡潔，易於學習與維護，而且我對python比較熟悉
- 
### 學習與準備

選定框架後，我開始進行學習，首先透過線上教學筆記學習 Django 的基本概念，發現文字教學很難理解後，轉向觀看一系列教學影片，跟隨影片一步步實作，並邊學邊啟動專案，跟著影片寫一些基礎的東西。

### 初步實作
#### 啓動項目
#### 設計靜態網頁

我在國小國中有上過靜態網站設計的課程，所以我可以先簡單設計幾個網站頁面，我先設定 URL 路徑，搞定不同頁面之間的順暢切換。

```html
<!-- templates/base.html -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>訂單管理系統</title>
</head>
<body>
    <nav>
        <a href="{% url 'home' %}">首頁</a>
        <a href="{% url 'product_manage' %}">商品管理</a>
        <a href="{% url 'order_manage' %}">訂單管理</a>
        <a href="{% url 'user_manage' %}">用戶管理</a>
    </nav>
    <div>
        {% block content %}
        {% endblock %}
    </div>
</body>
</html>
```

這裡可以看到我已經開始使用模板了，原因是我在剛開始實作的時候還沒使用git版本管理，所以我現在手邊能找到的程式代碼只有我後來更新的版本，這裡的url是透過後端傳輸給前端再透過模板引擎寫入html的。

#### 設計資料庫
我首先研究了sql的基本語法，然後在django内建的sqlite資料庫中建立了幾個表：

- **商品資料表 (Product)**

```sql
CREATE TABLE Product (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    release_date DATE,
    platform VARCHAR(10),
    product_name VARCHAR(50),
    suggested_price DECIMAL(7, 0),
    notes TEXT
);
```

- **用戶資料表 (ExtendedUser)**

```sql
CREATE TABLE ExtendedUser (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    user_id INTEGER UNIQUE,
    name VARCHAR(20),
    user_class VARCHAR(20),
    FOREIGN KEY (user_id) REFERENCES User(id)
);
```
- **訂單資料表 (Order)**

```sql
CREATE TABLE Order (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    product_id INTEGER,
    create_at DATETIME,
    notes TEXT,
    quantity INTEGER,
    user_id INTEGER,
    FOREIGN KEY (product_id) REFERENCES Product(id),
    FOREIGN KEY (user_id) REFERENCES ExtendedUser(id)
);
```

其實我換過資料庫結構，原本結構不是這樣的，不過如同前面，我沒有保留以前的開發紀錄，所以我只能用現在在用的django model請ai幫我生成同功能的sql指令當作範例，我現在很後悔沒有保持版本管理跟開發筆記，現在過了半年再來寫筆記好多東西都忘記了。

---

#### 引入 Django ORM 與 Models

在實作過程中，我發現使用原生SQL進行資料庫操作過於繁瑣，後來我在論壇上發現Django有Models這個東西，與django的契合度更好，且更安全且方便。

**Django ORM**：

- **模型定義**：使用 Django 的模型系統定義資料庫結構，簡化資料操作。
- **關聯設計**：設計模型之間的關聯，如訂單與商品、訂單與用戶的多對一關係，提升資料的一致性與完整性。

#### 1. **商品模型 (Product)**

```python
class Product(models.Model):
    id = models.AutoField(primary_key=True)
    release_date = models.DateField(blank=True, null=True)
    platform = models.CharField(max_length=10)
    product_name = models.CharField(max_length=50)
    suggested_price = models.DecimalField(max_digits=7, decimal_places=0, blank=True, null=True)
    notes = models.TextField(blank=True, null=True)
```

- **id**: 自動遞增的主鍵，Django 會自動處理。
- **release_date**: 商品的發佈日期，允許為空。
- **platform**: 商品的平台，最大長度為 10 個字符。
- **product_name**: 商品名稱，最大長度為 50 個字符。
- **suggested_price**: 商品的建議售價，最大位數為 7 位，小數位數為 0，允許為空。
- **notes**: 商品的備註，允許為空。

#### 2. **用戶模型 (ExtendedUser)**

```python
class ExtendedUser(models.Model):
    user = models.OneToOneField(User, on_delete=models.CASCADE)
    name = models.CharField(max_length=20)
    user_class = models.CharField(max_length=20)
```

- **user**: 與 Django 內建的 `User` 模型建立一對一關係，當 `User` 被刪除時，`ExtendedUser` 也會被刪除。
- **name**: 用戶的名稱，最大長度為 20 個字符。
- **user_class**: 用戶的類別，最大長度為 20 個字符。

#### 3. **訂單模型 (Order)**

```python
class Order(models.Model):
    id = models.AutoField(primary_key=True)
    product = models.ForeignKey(Product, on_delete=models.CASCADE, related_name='orders')
    create_at = models.DateTimeField(auto_now_add=True)
    notes = models.TextField(blank=True, null=True)
    quantity = models.IntegerField()
    user = models.ForeignKey(ExtendedUser, on_delete=models.CASCADE, related_name='orders', blank=True, null=True)
```

- **id**: 自動遞增的主鍵，Django 會自動處理。
- **product**: 與 `Product` 模型建立多對一關係，表示該訂單購買的商品。
- **create_at**: 訂單的創建時間，自動記錄當前時間。
- **notes**: 訂單的備註，允許為空。
- **quantity**: 訂單中的商品數量。
- **user**: 與 `ExtendedUser` 模型建立多對一關係，表示下單的用戶。

#### 4. **公告模型 (Announcement)**

```python
class Announcement(models.Model):
    id = models.AutoField(primary_key=True)
    notes = models.TextField(blank=True, null=True)
```

- **id**: 自動遞增的主鍵，Django 會自動處理。
- **notes**: 公告的內容，允許為空。

#### 開始自己手搓
我看完大部分的影片教學內容後，開始靠自己實作訂單管理網站

---

## 功能實現方法&&程式解說


### 訂單管理功能
#### 顧客下單頁面

- 首先我先處理用戶的http get請求，當用戶get後，後端會調用product django表單，表單會綁定所有商品，然後渲染到前端。
- 再來我是http post，當用戶post上來資料後，我會先驗證表單的合法性，假如合法，就處理表單資料，創建訂單的資料，以及綁定訂單到下單的帳號，最後重新url導向回訂單頁面

```python
class OrderView(LoginRequiredMixin, TemplateView):
    template_name = 'order.html'
    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        platforms = Product.objects.values_list('platform', flat=True).distinct()
        product = Product.objects.all()
        OrderFormSet = formset_factory(OrderForm, extra=len(Product.objects.all()))
        formset_instance = OrderFormSet()
        combined = zip(formset_instance.forms,product)
        context['platforms'] = platforms
        context['formset'] = OrderFormSet
        context['combined'] = combined
        return context
    
    def post(self, request, *args, **kwargs):
        OrderFormSet = formset_factory(OrderForm, extra=len(Product.objects.all()))
        formset = OrderFormSet(request.POST)
        if formset.is_valid():
            product = Product.objects.all()
            combined = zip(formset.forms,product)

            for form,product in combined:
                quantity = form.cleaned_data.get('quantity')
                if quantity is None or quantity==0:
                    continue
                notes = form.cleaned_data.get("input_notes")
                try:
                    order = Order.objects.create(product=product, notes=notes, quantity=quantity)
                    user = request.user
                    extendUser = ExtendedUser.objects.get(user=user)
                    extendUser.orders.add(order)
                except:
                    print("gg")
                    return redirect('/customer/order/')

            return redirect('/customer/order/')  # 重定向到訂單頁面
        else:
            print("not valid")
            
        return redirect('/customer/')
```

#### 使用 Django Forms

原本我是使用 HTML 表單與 HTTP update進行資料傳輸，但後來我發現有django Forms這個東西，他更加契合框架與模型，但他的相關教學比較少，我到最後都是直接看官方文檔，也就是在這時候我了解到官方文檔的好

- **表單定義**：使用 Django Forms 定義表單結構，與模型自動綁定，減少重複代碼。
- **表單驗證**：利用 Django Forms 提供的驗證機制，確保資料的合法性與完整性。
- **表單渲染**：在模板中直接渲染表單，提升開發效率與頁面一致性。

其實後來我有點後悔太早全部改成使用django form，首先他的寫法就是跟著框架走，我只能看著文檔照抄，雖然他的確跟框架的搭配比較好，但我每次想新增一些功能就要重新查找資料，我的學習成本、時間也因此提高，這也是我這個專案第二大的問題，我最後會統一總結，但現在想想可能還是自己寫html表單會比較好。


###  用戶管理功能

#### **註冊視圖（RegistView）**：提供管理者註冊新用戶的介面。

```python
class RegistView(LoginRequiredMixin, OpRequiredMixin, TemplateView):
    template_name = 'regist.html'
    
    def get(self, request, *args, **kwargs):
        form = RegistrationForm()
        return render(request, self.template_name, {'form': form})
    
    def post(self, request, *args, **kwargs):
        form = RegistrationForm(request.POST)
        if form.is_valid():
            form.save()
            return redirect("/op/userManage/?page=1")
        return render(request, self.template_name, {'form': form})
```
- get方法回傳html跟django form
- post方法處理回傳的表單資料，先檢查表單是否合法，再將資料儲存到database

#### django form code:
```python
# op/forms.py
class RegistrationForm(forms.ModelForm):
    userName = forms.CharField(max_length=150, required=True, label='帳號', widget=forms.TextInput(attrs={'class': 'form-control', 'placeholder': '請輸入帳號'}))
    password = forms.CharField(required=True, label='密碼', widget=forms.TextInput(attrs={'class': 'form-control', 'placeholder': '請輸入密碼'}))
    class Meta:
        model = ExtendedUser
        fields = ['user_class', 'name']
        labels = {
            'user_class': '用戶類別',
            'name': '用戶名',
        }
        widgets = {
            'user_class': forms.TextInput(attrs={'placeholder': '輸入用戶類別','class': 'form-control'}),
            'name': forms.TextInput(attrs={'placeholder': '輸入用戶名','class': 'form-control'}),
        }
    def clean_userName(self):
        userName = self.cleaned_data.get('userName')
        if User.objects.filter(username=userName).exists():
            raise forms.ValidationError('用戶名已經存在')
        return userName

    def save(self, commit=True):
        user = User.objects.create_user(username=self.cleaned_data['userName'], password=self.cleaned_data['password'])
        extended_user = ExtendedUser(user=user, user_class=self.cleaned_data['user_class'], name=self.cleaned_data['name'])
        if commit:
            extended_user.save()
        return extended_user
    
```
- Meta是渲染表單的資料，他可以在這設定html class/id，給css/js使用
- clean_userName方法是清理用戶上傳的表單的資料
- save是當確認資料合法後，會把資料存進資料庫

#### **用戶編輯視圖（UserEditView）**：提供編輯現有用戶資訊的功能。
- 這邊是用http協議的參數來傳輸用戶id給後端，後端收到id後會把用戶資料先打在表格裡方便管理帳號更改
- 當更改完後表單post上來，一樣會驗證表單合法性後儲存
```python
class UserEditView(LoginRequiredMixin, OpRequiredMixin, TemplateView):
    template_name = 'userEdit.html'

    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        userId = self.request.GET.get('userId')
        user = ExtendedUser.objects.get(user__id=userId)
        initial_data = {
            'userId': user.user.id,
            'userName': user.user.username,
            'user_class': user.user_class,
            'name': user.name,
        }
        form = UserEditForm(initial=initial_data)
        context['form'] = form
        return context
    
    def post(self, request):
        form = UserEditForm(request.POST)
        if form.is_valid():
            form.save()
            return redirect("/op/userManage/?page=1")
        return render(request, self.template_name, {'form': form})
```
#### **用戶刪除視圖（UserDelView）**：提供刪除指定用戶的功能。
- 也是用http參數傳輸帳號id，然後直接把帳號從資料庫中刪除
```python
class UserDelView(LoginRequiredMixin, View, OpRequiredMixin):
    def get(self, request):
        userId = request.GET['userId']
        try:
            user = User.objects.get(id=userId)
            user.delete()
            return redirect("/op/userManage/?page=1")
        except:
            return redirect("/op/userManage/?page=1")
```
#### **用戶管理列表（UserManageView）**：顯示所有用戶的列表，並實現分頁功能。

###  用戶認證與權限管理

我除了區分管理員帳號以及普通帳號，還有不管進入任何頁面都需要先登入，而普通帳號跟管理員帳號是綁定在django內建的帳號系統之外的extended user，內建帳號系統提供方便的登入驗證，我是把帳號跟密碼存在這邊，而extended user可以讓我識別是管理員帳號還是普通帳號。
```python
class ExtendedUser(models.Model):
    user = models.OneToOneField(User, on_delete=models.CASCADE)
    name = models.CharField(max_length=20)
    user_class = models.CharField(max_length=20)
```
#### 登入驗證

- 使用 Django 提供的 **LoginRequiredMixin**，確保部分視圖僅對已登入用戶開放。
- 自定義 **OpRequiredMixin**，進一步檢查用戶是否具備管理員的權限，防止未授權的存取。

```python
# base/mixins.py
from django.http import HttpResponseForbidden
from base.models import ExtendedUser

class OpRequiredMixin:
    def dispatch(self, request, *args, **kwargs):
        try:
            extended_user = ExtendedUser.objects.get(user=request.user)
            if extended_user.user_class != 'op':
                return HttpResponseForbidden("You do not have permission to access this page.")
        except ExtendedUser.DoesNotExist:
            return HttpResponseForbidden("You do not have permission to access this page.")
        return super().dispatch(request, *args, **kwargs)
```

```python
# base/views.py (更新其他視圖以使用 OpRequiredMixin)
from .mixins import OpRequiredMixin

class HomeView(LoginRequiredMixin, OpRequiredMixin, ListView):
    template_name = 'opHome.html'
    model = Announcement
    context_object_name = 'notes'
```


### 5. 訂單與商品管理
### 訂單管理視圖

- **檢視訂單（CheckView）**：顯示所有訂單資訊，供管理者檢視與管理。
```python
class CheckView(LoginRequiredMixin, TemplateView):
    template_name = 'check.html'
    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        user = self.request.user
        extendUser = get_object_or_404(ExtendedUser, user=user)
        sameClass = ExtendedUser.objects.filter(user_class=extendUser.user_class)
        orders = Order.objects.filter(user__in=sameClass).distinct()

        context['orders'] = orders
        return context
```

- **刪除訂單（OrderDelView）**：提供刪除指定訂單的功能，增強訂單管理的靈活性。

```python
class OrderDelView(LoginRequiredMixin, View, OpRequiredMixin):
    def get(self, request):
        id = request.GET.get('id')
        try:
            order = Order.objects.get(id=id)
            order.delete()
        except Order.DoesNotExist:
            pass
        return redirect("/op/check/")
```

```html
<!-- templates/opCheck.html -->
{% extends 'base.html' %}

{% block content %}
  <h2>訂單檢視</h2>
  <table>
    <tr>
      <th>訂單編號</th>
      <th>商品</th>
      <th>下單時間</th>
      <th>數量</th>
      <th>備註</th>
      <th>用戶</th>
      <th>操作</th>
    </tr>
    {% for order in orders %}
    <tr>
      <td>{{ order.id }}</td>
      <td>{{ order.product.product_name }}</td>
      <td>{{ order.order_time }}</td>
      <td>{{ order.quantity }}</td>
      <td>{{ order.notes }}</td>
      <td>{{ order.user.name }}</td>
      <td>
        <a href="{% url 'order_delete' %}?id={{ order.id }}">刪除</a>
      </td>
    </tr>
    {% endfor %}
  </table>
{% endblock %}
```

#### 商品管理視圖

- **檢視商品（ProductView）**：顯示所有商品及其所屬平台資訊，供管理者檢視。
```python
class CheckView(LoginRequiredMixin, TemplateView, OpRequiredMixin):
    template_name = 'opCheck.html'    
    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        orders = Order.objects.all()
        products = Product.objects.all()
            
        context['orders'] = orders
        return context
```
- **創建商品（CreateProductView）**：提供新增商品的介面，允許管理者添加新的商品資訊。
- **編輯商品（EditProductView）**：提供編輯現有商品資訊的功能，允許管理者更新商品細節。
- **刪除商品（ProductDelView）**：提供刪除指定商品的功能，允許管理者移除不再販售的商品。

```python
# base/views.py (繼續添加商品管理視圖)
class ProductView(LoginRequiredMixin, TemplateView, OpRequiredMixin):
    template_name = 'products.html'
    
    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        platforms = Product.objects.values_list('platform', flat=True).distinct()
        products = Product.objects.all()
        context['platforms'] = platforms
        context['products'] = products
        return context

class ProductDelView(LoginRequiredMixin, View, OpRequiredMixin):
    def get(self, request):
        id = request.GET.get('id')
        try:
            product = Product.objects.get(id=id)
            product.delete()
        except Product.DoesNotExist:
            pass
        return redirect("/op/productManage/")

class EditProductView(LoginRequiredMixin, OpRequiredMixin, TemplateView):
    template_name = 'editProduct.html'
    
    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        id = self.request.GET.get('id')
        product = get_object_or_404(Product, id=id)
        
        initial_data = {
            "id": product.id,
            'release_date': product.release_date,
            'platform': product.platform,
            'product_name': product.product_name,
            'suggested_price': product.suggested_price,
            'notes': product.notes,
        }
        form = EditProductForm(initial=initial_data)
        context['form'] = form
        return context
    
    def post(self, request, *args, **kwargs):
        form = EditProductForm(request.POST)
        if form.is_valid():
            id = form.cleaned_data.get('id')
            product = get_object_or_404(Product, id=id)
            product.release_date = form.cleaned_data.get('release_date')
            product.platform = form.cleaned_data.get('platform')
            product.product_name = form.cleaned_data.get('product_name')
            product.suggested_price = form.cleaned_data.get('suggested_price')
            product.notes = form.cleaned_data.get('notes')
            product.save()
            return redirect("/op/productManage/")
        return render(request, self.template_name, {'form': form})
```

```html
<!-- templates/products.html -->
{% extends 'base.html' %}

{% block content %}
  <h2>商品管理</h2>
  <a href="{% url 'create_product' %}">新增商品</a>
  <table>
    <tr>
      <th>商品名稱</th>
      <th>平台</th>
      <th>發售日期</th>
      <th>建議售價</th>
      <th>備註</th>
      <th>操作</th>
    </tr>
    {% for product in products %}
    <tr>
      <td>{{ product.product_name }}</td>
      <td>{{ product.platform }}</td>
      <td>{{ product.release_date }}</td>
      <td>{{ product.suggested_price }}</td>
      <td>{{ product.notes }}</td>
      <td>
        <a href="{% url 'edit_product' %}?id={{ product.id }}">編輯</a>
        <a href="{% url 'product_delete' %}?id={{ product.id }}">刪除</a>
      </td>
    </tr>
    {% endfor %}
  </table>
{% endblock %}
```

### 分頁功能實作

在用戶管理視圖（UserManageView）中，實作了分頁功能：

- **Paginator**：使用 Django 的 Paginator 類別，將用戶列表分成多頁顯示。
- **頁面導航**：在模板中實現前後頁的導航按鈕，提升管理者瀏覽大量用戶的效率。
- **動態頁碼計算**：根據當前頁數與總頁數，動態計算前一頁、下一頁、最大頁數及當前頁數，確保分頁邏輯的正確性。

```python
# base/views.py (UserManageView 已展示，重點在 get_context_data)
def get_context_data(self, **kwargs):
    context = super().get_context_data(**kwargs)
    paginator = context['paginator']
    page_obj = context['page_obj']
    pages = {
        "pre": page_obj.previous_page_number() if page_obj.has_previous() else 1,
        "next": page_obj.next_page_number() if page_obj.has_next() else paginator.num_pages,
        "max": paginator.num_pages,
        "now": page_obj.number,
    }
    context['page'] = pages
    return context
```

```html
<!-- templates/userManage.html (已展示，包含分頁導航) -->
<div>
  <a href="?page={{ page.pre }}">上一頁</a>
  <span>第 {{ page.now }} 頁，共 {{ page.max }} 頁</span>
  <a href="?page={{ page.next }}">下一頁</a>
</div>
```

### 錯誤處理與除錯

在開發過程中，遇到了多種挑戰，並學會了相應的錯誤處理與除錯方法：

- **權限驗證失敗**：
  - 使用 `LoginRequiredMixin` 與 `OpRequiredMixin` 確保用戶具備相應權限，防止未授權存取。
  - 若權限驗證失敗，返回 HTTP 403 禁止訪問的回應，提升系統安全性。

- **資料查詢失敗**：
  - 在刪除視圖（如 `UserDelView`、`OrderDelView`、`ProductDelView`）中，若查詢對應實例失敗，會捕捉異常並重定向至管理頁面。
  - 避免因資料缺失或錯誤引發的系統崩潰，提升系統穩定性。

- **表單驗證失敗**：
  - 在表單處理視圖（如 `RegistView`、`UserEditView`、`CreateProductView`、`EditProductView`）中，若表單驗證失敗，重新渲染表單頁面並顯示錯誤訊息。
  - 提升使用者體驗，幫助用戶修正錯誤輸入。


### Git 版本控制

因爲一直回檔，我發現我真的應該學學版本控制，而不是一直ctrl+z+備份
我上網簡單研究了一下git的基本指令，熟悉後便開始使用
~~雖然我後來都在用vscode的插件gitlens或git graph~~

```bash
# 初始化 Git 版本庫
git init

# 添加所有檔案
git add .

# 初始提交
git commit -m "Initial commit"

# 創建新分支
git checkout -b feature/user-management

# 完成功能後，切換回主分支
git checkout main

# 合併功能分支
git merge feature/user-management

# 推送至遠端倉庫
git remote add origin https://github.com/username/order-management.git
git push -u origin main
```

### 部署與測試

在完成開發後，進行了系統的測試與部署：
一開始我是想用heroku來雲端部署，因爲我在論壇一直看到，而且gpt跟我說他是免費的，結果上網一研究發現他今年取消免費方案了。最後我選了render，雖然功能沒有heroku那麽全面，但至少他是免費的，給我拿來測試用已經夠了。

- **本地測試**：在本地環境中進行全面測試，確保所有功能正常運作。
- **部署準備**：準備部署環境，包括伺服器配置、資料庫設置等，github資料庫，requirements.txt
- **上線部署**：將系統部署至線上伺服器，進行最後的測試與優化，確保系統在實際運作中的穩定性。



## 遇到的挑戰與解決方法

在開發過程中，遇到了多方面的挑戰，這些挑戰促使我不斷學習與成長：

1. **權限管理的複雜性**：
   - 初期設計用戶角色時，未能充分考慮到不同用戶間的權限細節，導致部分功能出現權限漏洞。
   - 解決方案：深入研究 Django 的用戶認證與權限管理系統，使用自定義的混入類別（如 `OpRequiredMixin`）來加強權限控制。

2. **資料庫關聯設計的挑戰**：
   - 初期使用原生 SQL 進行資料庫操作，隨著系統複雜度增加，資料庫查詢與操作變得困難。
   - 解決方案：轉向使用 Django ORM，重新設計模型關聯，簡化資料庫操作，提升開發效率。

3. **表單處理與驗證**：
   - 在實作表單時，遇到表單驗證失敗的問題，導致資料無法正確保存。
   - 解決方案：深入了解 Django Forms 的使用方法，合理設計表單驗證規則，確保資料的完整性與合法性。

4. **前後端整合的困難**：
   - 初期前端與後端的數據傳輸存在不一致，導致部分功能無法正常運作。
   - 解決方案：學習並應用 Django 的模板系統與表單處理，確保前後端數據的一致性，提升系統的穩定性。


## 解決方案

針對以上挑戰，採取了以下解決方案：

1. **深入學習與研究**：
   - 利用線上資源、官方文檔與社群論壇，深入了解 Django 的各項功能與最佳實踐。
   - 參考其他開源項目的實作，學習其架構與設計思路。

2. **模塊化開發**：
   - 將系統功能劃分為不同的模塊，分步實作與測試，避免功能耦合過高。
   - 使用 Django 的類視圖（Class-Based Views）與混入類別（Mixin）實現代碼的重用與擴展。

3. **版本控制與回滾**：
   - 使用 Git 進行版本控制，保證每個開發階段都有清晰的版本記錄。
   - 在遇到重大錯誤時，能夠迅速回滾至穩定版本，減少開發風險。

4. **測試驅動開發（TDD）**：
   - 實施測試驅動開發，先撰寫測試用例，再實作功能，確保系統的穩定性與可靠性。
   - 使用 Django 的測試框架，進行單元測試與整合測試。

5. **社群支持與協作**：
   - 積極參與 Django 社群，向其他開發者請教問題，分享經驗與心得。
   - 參與線上討論，從他人的經驗中學習，提升解決問題的能力。

## 心得與反思

通過這次訂單管理系統的開發，我不僅掌握了 Django 框架的各項功能，還在實際應用中學會了如何進行資料庫設計、用戶認證與權限管理、前後端整合、API 整合等技能。這個項目不僅提升了我的全棧開發能力，還讓我體驗到了從需求分析、系統設計、功能實作到測試部署的完整開發流程。

### 成果展示

最終，我成功完成了一個功能完善的訂單管理系統，具備以下特點：

- **用戶管理**：管理者可以新增、編輯、刪除用戶，並分配相應的權限。
- **訂單管理**：管理者可以查看所有訂單，並進行必要的操作；門市用戶只能查看自己的訂單。
- **商品管理**：管理者可以新增、編輯、刪除商品，門市用戶可以根據商品進行下單。
- **安全性與穩定性**：通過嚴格的權限控制與錯誤處理，確保系統的安全性與穩定性。

### 未來展望

這個訂單管理系統只是我在 Django 開發路上的第一步，未來我計劃進一步優化系統功能，提升用戶體驗，包括：

- **前端優化**：引入前端框架（如 React 或 Vue）提升頁面互動性與響應速度。
- **功能擴展**：增加更多的業務功能，如報表生成、數據分析等。
- **性能優化**：通過緩存、資料庫優化等手段提升系統性能，應對更多用戶的同時訪問。
- **持續學習**：不斷學習最新的技術與框架，保持技術的前沿性與競爭力。

---

通過這次的專案開發，我深刻體會到理論與實踐的結合之美，也更加堅定了我在軟體開發領域不斷探索與進步的決心。未來，我將繼續挑戰更多複雜的專案，提升自己的技術能力，為實現更高的職業目標而努力。