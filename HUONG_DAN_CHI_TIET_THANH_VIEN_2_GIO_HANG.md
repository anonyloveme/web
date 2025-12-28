# 📘 HƯỚNG DẪN CHI TIẾT - THÀNH VIÊN 2: QUẢN LÝ GIỎ HÀNG

> **Mục tiêu**: Học từng bước, hiểu từng dòng code, tự tin giải thích cho thầy  
> **Phong cách**: Theo đúng source code News mẫu của thầy  
> **Thời gian học**: 2-3 tuần (3-4 giờ/ngày)

---

## 📑 MỤC LỤC

### **PHẦN A: KIẾN THỨC NỀN TẢNG**
1. [Session trong JSP/Servlet](#phan-a1)
2. [Cookie vs Session](#phan-a2)
3. [Shopping Cart Pattern](#phan-a3)
4. [Tính toán trong giỏ hàng](#phan-a4)

### **PHẦN B: CODE CHI TIẾT - MODULE GIỎ HÀNG**
1. [Model: CartItem.java](#phan-b1)
2. [Helper: Cart.java (Shopping Cart Manager)](#phan-b2)
3. [Servlet: CartServlet.java](#phan-b3)
4. [View: JSP Pages](#phan-b4)

### **PHẦN C: LUỒNG XỬ LÝ HOÀN CHỈNH**
1. [Kịch bản 1: Thêm sản phẩm vào giỏ hàng](#phan-c1)
2. [Kịch bản 2: Cập nhật số lượng trong giỏ](#phan-c2)
3. [Kịch bản 3: Xóa sản phẩm khỏi giỏ](#phan-c3)

### **PHẦN D: CHUẨN BỊ BÁO CÁO**
1. [Câu hỏi thầy thường hỏi & Cách trả lời](#phan-d1)
2. [Demo & Giải thích code trực tiếp](#phan-d2)

---

---

# PHẦN A: KIẾN THỨC NỀN TẢNG

<a name="phan-a1"></a>
## A1. Session trong JSP/Servlet

### **1.1. Session là gì?**

**Định nghĩa** (theo thầy dạy):
```
Session là cơ chế lưu trữ thông tin của USER trên SERVER trong suốt 
phiên làm việc (từ khi truy cập đến khi đóng browser hoặc timeout).
```

**Đặc điểm**:
- **Lưu trên server** (không phải client như Cookie)
- **Mỗi user có 1 session riêng** (được nhận diện bởi Session ID)
- **Có thời gian sống** (mặc định 30 phút không hoạt động)
- **Có thể lưu object phức tạp** (List, Map, custom objects)

### **1.2. Tại sao cần Session cho Giỏ hàng?**

**Vấn đề**: HTTP là stateless (không nhớ trạng thái)
```
User thêm sản phẩm vào giỏ → Chuyển trang khác → Giỏ hàng mất!
```

**Giải pháp**: Dùng Session lưu giỏ hàng
```
Request 1: Thêm Bánh quy → Session lưu {Bánh quy: 2 cái}
Request 2: Thêm Snack → Session lưu {Bánh quy: 2, Snack: 1}
Request 3: Xem giỏ hàng → Session trả về {Bánh quy: 2, Snack: 1}
```

### **1.3. Sử dụng Session trong Servlet**

**Lấy Session (tạo mới nếu chưa có)**:
```java
HttpSession session = request.getSession();  // getSession(true)
```

**Lấy Session (không tạo mới)**:
```java
HttpSession session = request.getSession(false);  // Trả về null nếu chưa có
```

**Lưu dữ liệu vào Session**:
```java
session.setAttribute("cart", cartObject);  // Key: "cart", Value: cartObject
```

**Lấy dữ liệu từ Session**:
```java
Cart cart = (Cart) session.getAttribute("cart");
if (cart == null) {
    cart = new Cart();  // Tạo giỏ hàng mới
    session.setAttribute("cart", cart);
}
```

**Xóa dữ liệu khỏi Session**:
```java
session.removeAttribute("cart");
```

**Hủy toàn bộ Session**:
```java
session.invalidate();  // Xóa tất cả attribute, Session ID mới
```

**Các method thường dùng**:
```java
String sessionId = session.getId();                    // Lấy Session ID
long creationTime = session.getCreationTime();        // Thời gian tạo
long lastAccessTime = session.getLastAccessedTime();  // Lần truy cập cuối
int maxInactive = session.getMaxInactiveInterval();   // Timeout (giây)
session.setMaxInactiveInterval(3600);                 // Set timeout 1 giờ
```

---

<a name="phan-a2"></a>
## A2. Cookie vs Session

### **2.1. So sánh Cookie và Session**

| Đặc điểm | Cookie | Session |
|----------|--------|---------|
| **Lưu trữ** | Client (browser) | Server |
| **Dung lượng** | ~4KB/cookie | Không giới hạn |
| **Bảo mật** | Kém (user có thể xem/sửa) | Tốt (user không truy cập được) |
| **Tuổi thọ** | Có thể vĩnh viễn | Hết khi đóng browser hoặc timeout |
| **Kiểu dữ liệu** | Chỉ String | Object bất kỳ |
| **Tốc độ** | Nhanh (không cần server) | Chậm hơn (phải query server) |
| **Dùng cho** | Remember me, tracking | Giỏ hàng, đăng nhập |

### **2.2. Khi nào dùng Cookie, khi nào dùng Session?**

**Dùng Cookie**:
- Lưu ngôn ngữ (language preference)
- "Ghi nhớ đăng nhập" (Remember me)
- Tracking hành vi user (analytics)

**Dùng Session**:
- Giỏ hàng (Shopping cart)
- Thông tin đăng nhập (User info)
- Dữ liệu tạm (Temporary data)

**Trong module Giỏ hàng của em**: Dùng **Session** vì:
- Cần lưu object phức tạp (List<CartItem>)
- Bảo mật (không để user sửa giỏ hàng từ browser)
- Chỉ cần lưu trong phiên làm việc

---

<a name="phan-a3"></a>
## A3. Shopping Cart Pattern

### **3.1. Kiến trúc Giỏ hàng**

```
┌─────────────────────────────────────────────────┐
│                   SESSION                       │
│  ┌───────────────────────────────────────────┐  │
│  │ Attribute: "cart" → Cart object           │  │
│  │  ├── cartItems: Map<Integer, CartItem>    │  │
│  │  │    ├── Key: productId                  │  │
│  │  │    └── Value: CartItem object          │  │
│  │  │         ├── product (Product object)   │  │
│  │  │         └── quantity (int)             │  │
│  │  ├── getTotalItems()                      │  │
│  │  ├── getTotalPrice()                      │  │
│  │  ├── addItem(Product, quantity)           │  │
│  │  ├── updateItem(productId, quantity)      │  │
│  │  └── removeItem(productId)                │  │
│  └───────────────────────────────────────────┘  │
└─────────────────────────────────────────────────┘
```

### **3.2. Tại sao dùng Map<Integer, CartItem>?**

**Câu hỏi**: Tại sao không dùng List<CartItem>?

**Trả lời**:

**List<CartItem>** (KHÔNG TỐT):
```java
// Kiểm tra sản phẩm đã có trong giỏ chưa
for (CartItem item : cartItems) {
    if (item.getProduct().getId() == productId) {
        // Tìm thấy → Tăng số lượng
        item.setQuantity(item.getQuantity() + 1);
        found = true;
        break;
    }
}
if (!found) {
    // Chưa có → Thêm mới
    cartItems.add(new CartItem(...));
}
```
**Vấn đề**: Phải duyệt toàn bộ List → O(n)

**Map<Integer, CartItem>** (TỐT):
```java
// Kiểm tra và cập nhật trong 1 dòng
CartItem item = cartItems.get(productId);
if (item != null) {
    // Đã có → Tăng số lượng
    item.setQuantity(item.getQuantity() + 1);
} else {
    // Chưa có → Thêm mới
    cartItems.put(productId, new CartItem(...));
}
```
**Lợi ích**: Truy cập trực tiếp bằng key → O(1)

---

<a name="phan-a4"></a>
## A4. Tính toán trong Giỏ hàng

### **4.1. Các phép tính cần thiết**

**1. Tổng số lượng sản phẩm**:
```java
// Ví dụ: Bánh quy (2 cái) + Snack (3 cái) = 5 cái
public int getTotalItems() {
    int total = 0;
    for (CartItem item : cartItems.values()) {
        total += item.getQuantity();
    }
    return total;
}
```

**2. Tổng tiền giỏ hàng**:
```java
// Ví dụ: (Bánh quy: 15,000 x 2) + (Snack: 10,000 x 3) = 60,000
public BigDecimal getTotalPrice() {
    BigDecimal total = BigDecimal.ZERO;
    for (CartItem item : cartItems.values()) {
        total = total.add(item.getSubtotal());
    }
    return total;
}
```

**3. Thành tiền từng sản phẩm**:
```java
// Trong CartItem
public BigDecimal getSubtotal() {
    return product.getPrice().multiply(new BigDecimal(quantity));
}
```

### **4.2. Tại sao dùng BigDecimal cho tiền?**

**float/double có lỗi làm tròn**:
```java
double a = 0.1;
double b = 0.2;
System.out.println(a + b);  // 0.30000000000000004 ← SAI!
```

**BigDecimal chính xác tuyệt đối**:
```java
BigDecimal a = new BigDecimal("0.1");
BigDecimal b = new BigDecimal("0.2");
System.out.println(a.add(b));  // 0.3 ← ĐÚNG!
```

**Quy tắc làm việc với BigDecimal**:
```java
BigDecimal price = new BigDecimal("15000");
BigDecimal quantity = new BigDecimal("2");

// Cộng
BigDecimal sum = price.add(quantity);

// Trừ
BigDecimal diff = price.subtract(quantity);

// Nhân
BigDecimal product = price.multiply(quantity);  // 15000 * 2 = 30000

// Chia
BigDecimal quotient = price.divide(quantity, 2, RoundingMode.HALF_UP);  // 15000 / 2 = 7500.00

// So sánh
if (price.compareTo(BigDecimal.ZERO) > 0) {  // price > 0
    // ...
}
```

---

---

# PHẦN B: CODE CHI TIẾT - MODULE GIỎ HÀNG

<a name="phan-b1"></a>
## B1. Model: CartItem.java

### **1.1. Vai trò của CartItem**

**CartItem** đại diện cho **1 dòng trong giỏ hàng** (1 sản phẩm + số lượng).

**Quan hệ**:
```
Cart (Giỏ hàng)
  ├── CartItem 1: Bánh quy Oreo (2 cái)
  ├── CartItem 2: Snack Oishi (3 cái)
  └── CartItem 3: Nước ngọt Coca (1 chai)
```

### **1.2. Code đầy đủ: CartItem.java**

```java
package model;

import java.io.Serializable;
import java.math.BigDecimal;

/**
 * Lớp CartItem đại diện cho 1 dòng trong giỏ hàng.
 * Mỗi CartItem bao gồm sản phẩm (Product) và số lượng (quantity).
 * 
 * @author ThanhVien2
 */
public class CartItem implements Serializable {
    
    private static final long serialVersionUID = 1L;
    
    // ============================================
    // PHẦN 1: KHAI BÁO THUỘC TÍNH
    // ============================================
    
    private Product product;   // Sản phẩm trong giỏ
    private int quantity;      // Số lượng
    
    // Tại sao không lưu productId thay vì Product object?
    // → Vì cần thông tin đầy đủ (name, price, image...) để hiển thị trong giỏ
    // → Nếu chỉ lưu id, phải query DB mỗi lần hiển thị giỏ → Chậm!
    
    
    // ============================================
    // PHẦN 2: CONSTRUCTOR
    // ============================================
    
    /**
     * Constructor mặc định.
     */
    public CartItem() {
    }
    
    /**
     * Constructor đầy đủ tham số.
     * 
     * @param product  Sản phẩm
     * @param quantity Số lượng
     */
    public CartItem(Product product, int quantity) {
        this.product = product;
        this.quantity = quantity;
    }
    
    
    // ============================================
    // PHẦN 3: GETTER & SETTER
    // ============================================
    
    /**
     * Lấy sản phẩm.
     * @return Product object
     */
    public Product getProduct() {
        return product;
    }
    
    /**
     * Gán sản phẩm.
     * @param product Product object
     */
    public void setProduct(Product product) {
        this.product = product;
    }
    
    /**
     * Lấy số lượng.
     * @return Số lượng
     */
    public int getQuantity() {
        return quantity;
    }
    
    /**
     * Gán số lượng.
     * Validation: Số lượng phải > 0.
     * @param quantity Số lượng mới
     */
    public void setQuantity(int quantity) {
        if (quantity <= 0) {
            throw new IllegalArgumentException("Số lượng phải lớn hơn 0!");
        }
        this.quantity = quantity;
    }
    
    
    // ============================================
    // PHẦN 4: PHƯƠNG THỨC TÍNH TOÁN
    // ============================================
    
    /**
     * Tính thành tiền (subtotal) của dòng này.
     * Công thức: Giá * Số lượng
     * 
     * Ví dụ:
     *   - Bánh quy: 15,000đ
     *   - Số lượng: 2
     *   - Subtotal: 15,000 * 2 = 30,000đ
     * 
     * @return Thành tiền (BigDecimal)
     */
    public BigDecimal getSubtotal() {
        if (product == null || product.getPrice() == null) {
            return BigDecimal.ZERO;
        }
        return product.getPrice().multiply(new BigDecimal(quantity));
    }
    
    /**
     * Tăng số lượng lên 1.
     * Dùng khi user click "+"
     */
    public void increaseQuantity() {
        this.quantity++;
    }
    
    /**
     * Giảm số lượng xuống 1.
     * Dùng khi user click "-"
     * Validation: Không cho giảm xuống dưới 1.
     */
    public void decreaseQuantity() {
        if (this.quantity > 1) {
            this.quantity--;
        }
    }
    
    
    // ============================================
    // PHẦN 5: PHƯƠNG THỨC TIỆN ÍCH
    // ============================================
    
    /**
     * ToString() để debug.
     */
    @Override
    public String toString() {
        return "CartItem [product=" + product.getName() 
                + ", quantity=" + quantity 
                + ", subtotal=" + getSubtotal() + "]";
    }
}
```

### **1.3. Giải thích chi tiết**

**Q: Tại sao lưu Product object thay vì chỉ lưu productId?**

**A**:
```
Lưu Product object:
- Ưu điểm: Có đầy đủ thông tin (name, price, image...) để hiển thị
- Nhược điểm: Tốn bộ nhớ hơn

Lưu productId:
- Ưu điểm: Tiết kiệm bộ nhớ
- Nhược điểm: Mỗi lần hiển thị giỏ phải query DB → Chậm

→ Chọn lưu Product vì hiệu năng quan trọng hơn.
```

**Q: Tại sao multiply() phải new BigDecimal(quantity)?**

**A**:
```java
// SAI: Không thể nhân trực tiếp với int
subtotal = price.multiply(quantity);  // Lỗi biên dịch!

// ĐÚNG: Chuyển int → BigDecimal trước
subtotal = price.multiply(new BigDecimal(quantity));
```

---

<a name="phan-b2"></a>
## B2. Helper: Cart.java (Shopping Cart Manager)

### **2.1. Vai trò của Cart**

**Cart** là lớp **quản lý giỏ hàng**, chứa:
- Map các CartItem
- Các method thêm/sửa/xóa/tính toán

**Tại sao cần class Cart riêng?**
```
Thay vì lưu Map<Integer, CartItem> trực tiếp vào Session,
em tạo class Cart bọc ngoài để:
1. Đóng gói logic (encapsulation)
2. Dễ thêm method mới (getTotalPrice, clear, ...)
3. Code sạch hơn (Servlet không trực tiếp thao tác Map)
```

### **2.2. Code đầy đủ: Cart.java**

```java
package helper;

import model.CartItem;
import model.Product;

import java.io.Serializable;
import java.math.BigDecimal;
import java.util.*;

/**
 * Lớp Cart quản lý giỏ hàng.
 * Lưu trong Session với key "cart".
 * 
 * @author ThanhVien2
 */
public class Cart implements Serializable {
    
    private static final long serialVersionUID = 1L;
    
    // ============================================
    // PHẦN 1: KHAI BÁO THUỘC TÍNH
    // ============================================
    
    // Map lưu các CartItem
    // Key: productId, Value: CartItem
    private Map<Integer, CartItem> cartItems;
    
    
    // ============================================
    // PHẦN 2: CONSTRUCTOR
    // ============================================
    
    /**
     * Constructor: Khởi tạo giỏ hàng rỗng.
     */
    public Cart() {
        this.cartItems = new HashMap<>();
    }
    
    
    // ============================================
    // PHẦN 3: METHODS QUẢN LÝ GIỎ HÀNG
    // ============================================
    
    /**
     * METHOD 1: Thêm sản phẩm vào giỏ.
     * 
     * Logic:
     * - Nếu sản phẩm ĐÃ có trong giỏ → Tăng số lượng
     * - Nếu sản phẩm CHƯA có → Thêm mới
     * 
     * @param product  Sản phẩm cần thêm
     * @param quantity Số lượng
     */
    public void addItem(Product product, int quantity) {
        // Validate
        if (product == null) {
            throw new IllegalArgumentException("Sản phẩm không được null!");
        }
        if (quantity <= 0) {
            throw new IllegalArgumentException("Số lượng phải > 0!");
        }
        
        int productId = product.getId();
        
        // Kiểm tra sản phẩm đã có trong giỏ chưa
        CartItem existingItem = cartItems.get(productId);
        
        if (existingItem != null) {
            // ĐÃ CÓ → Tăng số lượng
            int newQuantity = existingItem.getQuantity() + quantity;
            existingItem.setQuantity(newQuantity);
            
            System.out.println("✅ Cập nhật số lượng: " + product.getName() 
                + " → " + newQuantity + " cái");
        } else {
            // CHƯA CÓ → Thêm mới
            CartItem newItem = new CartItem(product, quantity);
            cartItems.put(productId, newItem);
            
            System.out.println("✅ Thêm mới vào giỏ: " + product.getName() 
                + " (" + quantity + " cái)");
        }
    }
    
    /**
     * METHOD 2: Cập nhật số lượng sản phẩm trong giỏ.
     * 
     * @param productId   ID sản phẩm
     * @param newQuantity Số lượng mới
     * @return true nếu cập nhật thành công, false nếu không tìm thấy
     */
    public boolean updateItem(int productId, int newQuantity) {
        // Validate
        if (newQuantity <= 0) {
            throw new IllegalArgumentException("Số lượng phải > 0!");
        }
        
        CartItem item = cartItems.get(productId);
        
        if (item != null) {
            // Tìm thấy → Cập nhật
            item.setQuantity(newQuantity);
            System.out.println("✅ Cập nhật số lượng: " + item.getProduct().getName() 
                + " → " + newQuantity);
            return true;
        } else {
            // Không tìm thấy
            System.out.println("⚠️ Không tìm thấy sản phẩm ID = " + productId);
            return false;
        }
    }
    
    /**
     * METHOD 3: Xóa sản phẩm khỏi giỏ.
     * 
     * @param productId ID sản phẩm cần xóa
     * @return true nếu xóa thành công, false nếu không tìm thấy
     */
    public boolean removeItem(int productId) {
        CartItem removedItem = cartItems.remove(productId);
        
        if (removedItem != null) {
            System.out.println("✅ Đã xóa: " + removedItem.getProduct().getName());
            return true;
        } else {
            System.out.println("⚠️ Không tìm thấy sản phẩm ID = " + productId);
            return false;
        }
    }
    
    /**
     * METHOD 4: Xóa toàn bộ giỏ hàng.
     */
    public void clear() {
        cartItems.clear();
        System.out.println("✅ Đã xóa toàn bộ giỏ hàng");
    }
    
    /**
     * METHOD 5: Lấy CartItem theo productId.
     * 
     * @param productId ID sản phẩm
     * @return CartItem hoặc null nếu không tìm thấy
     */
    public CartItem getItem(int productId) {
        return cartItems.get(productId);
    }
    
    /**
     * METHOD 6: Lấy danh sách tất cả CartItem.
     * Dùng để hiển thị trong JSP.
     * 
     * @return Collection<CartItem>
     */
    public Collection<CartItem> getItems() {
        return cartItems.values();
    }
    
    
    // ============================================
    // PHẦN 4: METHODS TÍNH TOÁN
    // ============================================
    
    /**
     * METHOD 7: Tính tổng số lượng sản phẩm trong giỏ.
     * 
     * Ví dụ:
     *   - Bánh quy: 2 cái
     *   - Snack: 3 cái
     *   → Tổng: 5 cái
     * 
     * @return Tổng số lượng
     */
    public int getTotalItems() {
        int total = 0;
        for (CartItem item : cartItems.values()) {
            total += item.getQuantity();
        }
        return total;
    }
    
    /**
     * METHOD 8: Tính tổng tiền giỏ hàng.
     * 
     * Ví dụ:
     *   - Bánh quy: 15,000 x 2 = 30,000
     *   - Snack: 10,000 x 3 = 30,000
     *   → Tổng: 60,000
     * 
     * @return Tổng tiền (BigDecimal)
     */
    public BigDecimal getTotalPrice() {
        BigDecimal total = BigDecimal.ZERO;
        for (CartItem item : cartItems.values()) {
            total = total.add(item.getSubtotal());
        }
        return total;
    }
    
    /**
     * METHOD 9: Kiểm tra giỏ hàng có rỗng không.
     * 
     * @return true nếu giỏ rỗng
     */
    public boolean isEmpty() {
        return cartItems.isEmpty();
    }
    
    /**
     * METHOD 10: Đếm số loại sản phẩm khác nhau.
     * 
     * Ví dụ:
     *   - Bánh quy: 2 cái
     *   - Snack: 3 cái
     *   → Số loại: 2 (không phải 5!)
     * 
     * @return Số loại sản phẩm
     */
    public int getItemCount() {
        return cartItems.size();
    }
    
    
    // ============================================
    // PHẦN 5: PHƯƠNG THỨC TIỆN ÍCH
    // ============================================
    
    /**
     * ToString() để debug.
     */
    @Override
    public String toString() {
        return "Cart [items=" + getItemCount() 
                + ", totalQuantity=" + getTotalItems() 
                + ", totalPrice=" + getTotalPrice() + "]";
    }
}
```

### **2.3. Giải thích chi tiết**

**Q: Tại sao addItem() kiểm tra existingItem trước khi thêm?**

**A**:
```
Tránh duplicate:
- User thêm Bánh quy (2 cái)
- User thêm Bánh quy (3 cái) lần nữa
→ Không tạo 2 dòng riêng biệt
→ Tăng số lượng dòng cũ: 2 + 3 = 5 cái
```

**Q: getTotalItems() khác getItemCount() thế nào?**

**A**:
```
Ví dụ giỏ hàng:
- Bánh quy: 2 cái
- Snack: 3 cái

getTotalItems() = 2 + 3 = 5 (tổng số lượng)
getItemCount() = 2 (số loại sản phẩm)
```

---

(Tiếp tục Phần B3 - CartServlet.java trong phần tiếp theo...)
<a name="phan-b3"></a>
## B3. Servlet: CartServlet.java

### **3.1. Các action cần xử lý**

```
CartServlet
├── doGet()
│   ├── action=view     → Xem giỏ hàng
│   └── action=count    → Đếm số sản phẩm (AJAX)
└── doPost()
    ├── action=add      → Thêm vào giỏ
    ├── action=update   → Cập nhật số lượng
    ├── action=remove   → Xóa 1 sản phẩm
    └── action=clear    → Xóa toàn bộ giỏ
```

### **3.2. Code đầy đủ: CartServlet.java**

```java
package controller;

import dao.ProductDAO;
import helper.Cart;
import model.Product;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.*;
import java.io.IOException;
import java.io.PrintWriter;

@WebServlet("/CartServlet")
public class CartServlet extends HttpServlet {
    
    private ProductDAO productDAO;
    
    @Override
    public void init() throws ServletException {
        super.init();
        this.productDAO = new ProductDAO();
    }
    
    // ============================================
    // doGet() - HIỂN THỊ
    // ============================================
    
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        
        String action = request.getParameter("action");
        if (action == null) action = "view";
        
        switch (action) {
            case "view":
                viewCart(request, response);
                break;
            case "count":
                getCartCount(request, response);
                break;
            default:
                viewCart(request, response);
        }
    }
    
    /**
     * Hiển thị giỏ hàng.
     */
    private void viewCart(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        
        // Lấy Cart từ Session (tạo mới nếu chưa có)
        HttpSession session = request.getSession();
        Cart cart = (Cart) session.getAttribute("cart");
        
        if (cart == null) {
            cart = new Cart();
            session.setAttribute("cart", cart);
        }
        
        // Đưa cart vào request scope để JSP hiển thị
        request.setAttribute("cart", cart);
        
        // Forward sang JSP
        request.getRequestDispatcher("views/cart_view.jsp")
               .forward(request, response);
    }
    
    /**
     * Trả về số lượng sản phẩm trong giỏ (AJAX).
     */
    private void getCartCount(HttpServletRequest request, HttpServletResponse response)
            throws IOException {
        
        HttpSession session = request.getSession(false);
        int count = 0;
        
        if (session != null) {
            Cart cart = (Cart) session.getAttribute("cart");
            if (cart != null) {
                count = cart.getTotalItems();
            }
        }
        
        // Trả về JSON
        response.setContentType("application/json");
        PrintWriter out = response.getWriter();
        out.print("{\"count\": " + count + "}");
    }
    
    // ============================================
    // doPost() - XỬ LÝ THÊM/SỬA/XÓA
    // ============================================
    
    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        
        request.setCharacterEncoding("UTF-8");
        String action = request.getParameter("action");
        
        switch (action) {
            case "add":
                addToCart(request, response);
                break;
            case "update":
                updateCart(request, response);
                break;
            case "remove":
                removeFromCart(request, response);
                break;
            case "clear":
                clearCart(request, response);
                break;
            default:
                response.sendRedirect("CartServlet?action=view");
        }
    }
    
    /**
     * Thêm sản phẩm vào giỏ.
     */
    private void addToCart(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        
        try {
            // Lấy tham số
            int productId = Integer.parseInt(request.getParameter("productId"));
            int quantity = Integer.parseInt(request.getParameter("quantity"));
            
            // Lấy thông tin sản phẩm từ DB
            Product product = productDAO.getById(productId);
            
            if (product == null) {
                request.setAttribute("errorMessage", "Không tìm thấy sản phẩm!");
                request.getRequestDispatcher("views/error.jsp").forward(request, response);
                return;
            }
            
            // Lấy Cart từ Session
            HttpSession session = request.getSession();
            Cart cart = (Cart) session.getAttribute("cart");
            
            if (cart == null) {
                cart = new Cart();
                session.setAttribute("cart", cart);
            }
            
            // Thêm vào giỏ
            cart.addItem(product, quantity);
            
            // Redirect về giỏ hàng với thông báo
            response.sendRedirect("CartServlet?action=view&message=add_success");
            
        } catch (NumberFormatException e) {
            request.setAttribute("errorMessage", "Dữ liệu không hợp lệ!");
            request.getRequestDispatcher("views/error.jsp").forward(request, response);
        }
    }
    
    /**
     * Cập nhật số lượng.
     */
    private void updateCart(HttpServletRequest request, HttpServletResponse response)
            throws IOException {
        
        try {
            int productId = Integer.parseInt(request.getParameter("productId"));
            int newQuantity = Integer.parseInt(request.getParameter("quantity"));
            
            HttpSession session = request.getSession(false);
            if (session != null) {
                Cart cart = (Cart) session.getAttribute("cart");
                if (cart != null) {
                    cart.updateItem(productId, newQuantity);
                }
            }
            
            response.sendRedirect("CartServlet?action=view&message=update_success");
            
        } catch (NumberFormatException e) {
            response.sendRedirect("CartServlet?action=view&error=invalid_data");
        }
    }
    
    /**
     * Xóa 1 sản phẩm.
     */
    private void removeFromCart(HttpServletRequest request, HttpServletResponse response)
            throws IOException {
        
        try {
            int productId = Integer.parseInt(request.getParameter("productId"));
            
            HttpSession session = request.getSession(false);
            if (session != null) {
                Cart cart = (Cart) session.getAttribute("cart");
                if (cart != null) {
                    cart.removeItem(productId);
                }
            }
            
            response.sendRedirect("CartServlet?action=view&message=remove_success");
            
        } catch (NumberFormatException e) {
            response.sendRedirect("CartServlet?action=view&error=invalid_data");
        }
    }
    
    /**
     * Xóa toàn bộ giỏ hàng.
     */
    private void clearCart(HttpServletRequest request, HttpServletResponse response)
            throws IOException {
        
        HttpSession session = request.getSession(false);
        if (session != null) {
            Cart cart = (Cart) session.getAttribute("cart");
            if (cart != null) {
                cart.clear();
            }
        }
        
        response.sendRedirect("CartServlet?action=view&message=clear_success");
    }
}
```

---

## PHẦN C: LUỒNG XỬ LÝ

<a name="phan-c1"></a>
### C1. Kịch bản: Thêm sản phẩm vào giỏ

```
1. User click "Thêm vào giỏ" trên trang sản phẩm
2. Form gửi POST: action=add&productId=5&quantity=2
3. CartServlet.doPost() → addToCart()
4. Lấy Product từ DAO.getById(5)
5. Lấy Cart từ Session (tạo mới nếu chưa có)
6. cart.addItem(product, 2)
7. Cart kiểm tra: Đã có product này chưa?
   - Chưa có → Thêm mới CartItem
   - Đã có → Tăng quantity
8. Redirect về CartServlet?action=view
9. JSP hiển thị giỏ hàng
```

---

## PHẦN D: CÂU HỎI THẦY & ĐÁP ÁN

<a name="phan-d1"></a>
### Q1: "Em giải thích Session hoạt động như thế nào?"

**Trả lời**:
```
Thưa thầy, Session hoạt động như sau:

1. User truy cập lần đầu → Server tạo Session mới, gán Session ID
2. Session ID được gửi về browser qua Cookie (JSESSIONID)
3. Các request sau, browser tự động gửi Session ID lên
4. Server dựa vào Session ID để nhận diện user, lấy dữ liệu Session
5. Session tồn tại đến khi:
   - User đóng browser
   - Timeout (mặc định 30 phút không hoạt động)
   - Gọi session.invalidate()

Trong module em:
- Cart object được lưu trong Session với key "cart"
- Mỗi user có giỏ hàng riêng (Session riêng)
```

### Q2: "Tại sao dùng Map thay vì List cho giỏ hàng?"

**Trả lời**:
```
Thưa thầy, em dùng Map<Integer, CartItem> vì:

1. HIỆU NĂNG:
   - Map: Truy cập theo key → O(1)
   - List: Phải duyệt tìm kiếm → O(n)

2. DỄ XỬ LÝ:
   - Map: cart.get(productId) → Lấy ngay
   - List: Phải for loop tìm productId

3. VÍ DỤ:
   Kiểm tra sản phẩm đã có trong giỏ:
   - Map: CartItem item = map.get(5); // Nhanh!
   - List: for (item : list) if (item.id == 5) ... // Chậm!

Key là productId, Value là CartItem.
```

### Q3: "Forward vs Redirect trong giỏ hàng?"

**Trả lời**:
```
Thưa thầy:

FORWARD (dùng khi XEM giỏ):
- viewCart() → Forward sang cart_view.jsp
- Lý do: Cần truyền cart object sang JSP
- Request scope giữ nguyên

REDIRECT (dùng sau THÊM/SỬA/XÓA):
- addToCart() → Redirect về CartServlet?action=view
- Lý do: PRG pattern, tránh double submit
- User bấm F5 không thêm lại vào giỏ
```

---

**HẾT TÀI LIỆU THÀNH VIÊN 2**

