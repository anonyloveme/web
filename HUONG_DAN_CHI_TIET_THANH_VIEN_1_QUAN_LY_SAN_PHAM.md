# 📘 HƯỚNG DẪN CHI TIẾT - THÀNH VIÊN 1: QUẢN LÝ SẢN PHẨM

> **Mục tiêu**: Học từng bước, hiểu từng dòng code, tự tin giải thích cho thầy  
> **Phong cách**: Theo đúng source code News mẫu của thầy  
> **Thời gian học**: 2-3 tuần (3-4 giờ/ngày)

---

## 📑 MỤC LỤC

### **PHẦN A: KIẾN THỨC NỀN TẢNG**
1. [JSP/Servlet & Request-Response Cycle](#phan-a1)
2. [Mô hình MVC trong dự án](#phan-a2)
3. [JDBC & DAO Pattern](#phan-a3)
4. [JSTL & Expression Language](#phan-a4)

### **PHẦN B: CODE CHI TIẾT - MODULE SẢN PHẨM**
1. [Model: Product.java](#phan-b1)
2. [DAO: ProductDAO.java](#phan-b2)
3. [Servlet: ProductServlet.java](#phan-b3)
4. [View: JSP Pages](#phan-b4)

### **PHẦN C: LUỒNG XỬ LÝ HOÀN CHỈNH**
1. [Kịch bản 1: Xem danh sách sản phẩm](#phan-c1)
2. [Kịch bản 2: Thêm sản phẩm mới](#phan-c2)
3. [Kịch bản 3: Sửa sản phẩm](#phan-c3)
4. [Kịch bản 4: Xóa sản phẩm](#phan-c4)

### **PHẦN D: CHUẨN BỊ BÁO CÁO**
1. [Câu hỏi thầy thường hỏi & Cách trả lời](#phan-d1)
2. [Demo & Giải thích code trực tiếp](#phan-d2)

---

---

# PHẦN A: KIẾN THỨC NỀN TẢNG

<a name="phan-a1"></a>
## A1. JSP/Servlet & Request-Response Cycle

### **1.1. Servlet là gì?**

**Định nghĩa** (theo thầy dạy):
```
Servlet là một lớp Java chạy trên server, nhận request từ client (browser),
xử lý logic nghiệp vụ, và trả về response (HTML, JSON...).
```

**Vị trí trong kiến trúc**:
```
Browser (Client)  →  HTTP Request  →  Tomcat Server  →  Servlet
                                                         ↓
                                                    Xử lý logic
                                                         ↓
                   HTML Response  ←  Forward/Redirect  ←  JSP
```

### **1.2. JSP là gì?**

**Định nghĩa**:
```
JSP (JavaServer Pages) là file HTML có thể nhúng code Java.
Server sẽ biên dịch JSP thành Servlet, sau đó thực thi.
```

**Ví dụ đơn giản**:
```jsp
<!-- File: hello.jsp -->
<!DOCTYPE html>
<html>
<head><title>Xin chào</title></head>
<body>
    <h1>Xin chào <%= request.getParameter("name") %></h1>
    <!-- Khi truy cập: hello.jsp?name=Minh -->
    <!-- Kết quả: Xin chào Minh -->
</body>
</html>
```

### **1.3. Request-Response Cycle**

**Bước 1**: User nhập URL vào browser
```
http://localhost:8080/SnackShop/ProductServlet?action=list
```

**Bước 2**: Browser gửi HTTP Request đến Server
```
GET /SnackShop/ProductServlet?action=list HTTP/1.1
Host: localhost:8080
```

**Bước 3**: Tomcat nhận request, chuyển đến Servlet
```java
// Tomcat gọi method này trong ProductServlet
protected void doGet(HttpServletRequest request, HttpServletResponse response) {
    String action = request.getParameter("action"); // "list"
    // Xử lý...
}
```

**Bước 4**: Servlet xử lý logic, gọi DAO lấy dữ liệu từ DB

**Bước 5**: Servlet đưa dữ liệu vào request scope, forward sang JSP

**Bước 6**: JSP render HTML

**Bước 7**: Server gửi HTML response về browser

**Bước 8**: Browser hiển thị trang web

---

<a name="phan-a2"></a>
## A2. Mô hình MVC trong dự án

### **2.1. MVC là gì?**

**MVC = Model - View - Controller**

```
┌─────────────────────────────────────────────────┐
│                   BROWSER                       │
│            (User nhập URL, click button)        │
└──────────────────┬──────────────────────────────┘
                   │ HTTP Request
                   ↓
┌─────────────────────────────────────────────────┐
│              CONTROLLER (Servlet)               │
│  - Nhận request                                 │
│  - Phân tích action (list, add, edit, delete)   │
│  - Gọi Model/DAO xử lý                          │
│  - Chuẩn bị dữ liệu                             │
│  - Chọn View phù hợp                            │
└──────────────────┬──────────────────────────────┘
                   │ Gọi method
                   ↓
┌─────────────────────────────────────────────────┐
│               MODEL + DAO                       │
│  - Model (Product.java): Đại diện cho đối tượng │
│  - DAO (ProductDAO.java): Truy vấn database     │
│  - Thực thi SQL: SELECT, INSERT, UPDATE, DELETE │
└──────────────────┬──────────────────────────────┘
                   │ Trả về dữ liệu
                   ↓
┌─────────────────────────────────────────────────┐
│              VIEW (JSP)                         │
│  - Nhận dữ liệu từ Controller                   │
│  - Render HTML                                  │
│  - Sử dụng JSTL, EL để hiển thị                 │
└──────────────────┬──────────────────────────────┘
                   │ HTML Response
                   ↓
              BROWSER (Hiển thị)
```

### **2.2. Ví dụ trong module Quản lý Sản phẩm**

**Model**:
- `Product.java`: Lớp đại diện cho 1 sản phẩm (id, name, price, image...)

**View**:
- `product_list.jsp`: Hiển thị danh sách sản phẩm
- `product_add.jsp`: Form thêm sản phẩm mới
- `product_edit.jsp`: Form sửa sản phẩm

**Controller**:
- `ProductServlet.java`: Nhận request, xử lý action (list/add/edit/delete), gọi DAO, forward sang View

**DAO**:
- `ProductDAO.java`: Kết nối database, thực thi SQL

---

<a name="phan-a3"></a>
## A3. JDBC & DAO Pattern

### **3.1. JDBC là gì?**

**JDBC = Java Database Connectivity**

Là API chuẩn của Java để kết nối và thao tác với database (MySQL, Oracle, SQL Server...).

**Các bước kết nối DB bằng JDBC**:

```java
// Bước 1: Load driver (MySQL)
Class.forName("com.mysql.cj.jdbc.Driver");

// Bước 2: Tạo connection
Connection conn = DriverManager.getConnection(
    "jdbc:mysql://localhost:3306/snack_shop", 
    "root", 
    "password"
);

// Bước 3: Tạo Statement/PreparedStatement
PreparedStatement ps = conn.prepareStatement("SELECT * FROM products");

// Bước 4: Thực thi query
ResultSet rs = ps.executeQuery();

// Bước 5: Xử lý kết quả
while(rs.next()) {
    int id = rs.getInt("id");
    String name = rs.getString("name");
    // ...
}

// Bước 6: Đóng kết nối
rs.close();
ps.close();
conn.close();
```

### **3.2. DAO Pattern là gì?**

**DAO = Data Access Object**

**Mục đích**: Tách biệt logic truy cập database khỏi logic nghiệp vụ.

**Lợi ích**:
- Code sạch, dễ bảo trì
- Thay đổi DB (MySQL → Oracle) chỉ cần sửa DAO
- Servlet không cần biết SQL

**Cấu trúc DAO Pattern**:

```
┌─────────────────────────────────────┐
│        ProductServlet.java          │
│   (Controller - không chứa SQL)     │
└───────────────┬─────────────────────┘
                │ Gọi method
                ↓
┌─────────────────────────────────────┐
│        ProductDAO.java              │
│   - getAll()                        │
│   - getById(int id)                 │
│   - insert(Product p)               │
│   - update(Product p)               │
│   - delete(int id)                  │
└───────────────┬─────────────────────┘
                │ JDBC
                ↓
┌─────────────────────────────────────┐
│          MySQL Database             │
│        Table: products              │
└─────────────────────────────────────┘
```

### **3.3. Tại sao dùng PreparedStatement thay vì Statement?**

**Statement** (KHÔNG AN TOÀN):
```java
String sql = "SELECT * FROM products WHERE name = '" + userInput + "'";
Statement stmt = conn.createStatement();
ResultSet rs = stmt.executeQuery(sql);
```

**Vấn đề**: SQL Injection  
User nhập: `' OR '1'='1`  
SQL thành: `SELECT * FROM products WHERE name = '' OR '1'='1'`  
→ Lấy hết dữ liệu!

**PreparedStatement** (AN TOÀN):
```java
String sql = "SELECT * FROM products WHERE name = ?";
PreparedStatement ps = conn.prepareStatement(sql);
ps.setString(1, userInput); // Tự động escape ký tự đặc biệt
ResultSet rs = ps.executeQuery();
```

**Ưu điểm PreparedStatement**:
1. **Bảo mật**: Chống SQL Injection
2. **Hiệu năng**: SQL được compile trước, tái sử dụng
3. **Dễ đọc**: Tách biệt SQL và tham số

---

<a name="phan-a4"></a>
## A4. JSTL & Expression Language

### **4.1. JSTL là gì?**

**JSTL = JavaServer Pages Standard Tag Library**

Là tập hợp các thẻ (tag) chuẩn để thay thế scriptlet Java trong JSP.

**Thay vì viết**:
```jsp
<%
    List<Product> products = (List<Product>) request.getAttribute("products");
    for(Product p : products) {
        out.println("<p>" + p.getName() + "</p>");
    }
%>
```

**Dùng JSTL**:
```jsp
<c:forEach var="p" items="${products}">
    <p>${p.name}</p>
</c:forEach>
```

### **4.2. Các thẻ JSTL thường dùng**

**1. Thư viện Core** (prefix: `c`)
```jsp
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
```

**Thẻ `<c:forEach>` - Vòng lặp**:
```jsp
<c:forEach var="product" items="${productList}">
    <tr>
        <td>${product.id}</td>
        <td>${product.name}</td>
        <td>${product.price}</td>
    </tr>
</c:forEach>
```

**Thẻ `<c:if>` - Điều kiện**:
```jsp
<c:if test="${product.stock > 0}">
    <span class="in-stock">Còn hàng</span>
</c:if>
<c:if test="${product.stock == 0}">
    <span class="out-of-stock">Hết hàng</span>
</c:if>
```

**Thẻ `<c:choose>` - Nhiều điều kiện (như switch-case)**:
```jsp
<c:choose>
    <c:when test="${product.price < 10000}">
        <span>Giá rẻ</span>
    </c:when>
    <c:when test="${product.price < 50000}">
        <span>Giá trung bình</span>
    </c:when>
    <c:otherwise>
        <span>Giá cao</span>
    </c:otherwise>
</c:choose>
```

**Thẻ `<c:out>` - Xuất dữ liệu an toàn (escape HTML)**:
```jsp
<c:out value="${product.description}" />
```

### **4.3. Expression Language (EL)**

**EL** là cú pháp ngắn gọn để truy cập dữ liệu trong JSP.

**Cú pháp**: `${expression}`

**Ví dụ**:
```jsp
<!-- Lấy attribute từ request -->
${productName}  <!-- request.getAttribute("productName") -->

<!-- Lấy thuộc tính của object -->
${product.name}  <!-- product.getName() -->
${product.price}  <!-- product.getPrice() -->

<!-- Lấy parameter từ URL -->
${param.action}  <!-- request.getParameter("action") -->

<!-- Toán tử -->
${product.price * 0.9}  <!-- Giảm 10% -->
${product.stock > 0}  <!-- true/false -->
```

**Các scope trong EL**:
- `${pageScope.var}` - Page scope
- `${requestScope.var}` - Request scope
- `${sessionScope.var}` - Session scope
- `${applicationScope.var}` - Application scope

---

---

# PHẦN B: CODE CHI TIẾT - MODULE SẢN PHẨM

<a name="phan-b1"></a>
## B1. Model: Product.java

### **1.1. Vai trò của Model**

Model đại diện cho **đối tượng dữ liệu** trong ứng dụng. Mỗi instance của `Product.java` tương ứng với **1 dòng dữ liệu** trong bảng `products` của database.

**Quan hệ với Database**:
```
Database Table: products
┌────┬──────────┬───────┬─────────────┬───────┬──────────────────┬─────────┐
│ id │   name   │ price │ description │ stock │      image       │  cat_id │
├────┼──────────┼───────┼─────────────┼───────┼──────────────────┼─────────┤
│ 1  │ Bánh quy │ 15000 │ Ngon lành   │  100  │ banhquy.jpg      │    1    │
└────┴──────────┴───────┴─────────────┴───────┴──────────────────┴─────────┘
                              ↕
                    Product.java (Object)
┌─────────────────────────────────────────────────────────────────┐
│ Product p = new Product();                                      │
│ p.setId(1);                                                     │
│ p.setName("Bánh quy");                                          │
│ p.setPrice(15000);                                              │
│ p.setDescription("Ngon lành");                                  │
│ p.setStock(100);                                                │
│ p.setImage("banhquy.jpg");                                      │
│ p.setCategoryId(1);                                             │
└─────────────────────────────────────────────────────────────────┘
```

### **1.2. Code đầy đủ: Product.java**

```java
package model;

import java.io.Serializable;
import java.math.BigDecimal;

/**
 * Lớp Product đại diện cho một sản phẩm trong hệ thống.
 * Mỗi đối tượng Product tương ứng với 1 dòng trong bảng 'products'.
 * 
 * @author ThanhVien1
 * @version 1.0
 */
public class Product implements Serializable {
    
    // ============================================
    // PHẦN 1: KHAI BÁO THUỘC TÍNH (Properties)
    // ============================================
    
    /**
     * Mỗi thuộc tính tương ứng với 1 cột trong bảng database.
     * Quy tắc đặt tên: camelCase (Java) vs snake_case (SQL)
     * 
     * Ví dụ:
     *   - SQL: category_id (snake_case)
     *   - Java: categoryId (camelCase)
     */
    
    private int id;                    // Khóa chính (PRIMARY KEY)
    private String name;               // Tên sản phẩm (NOT NULL)
    private BigDecimal price;          // Giá (DECIMAL(10,2))
    private String description;        // Mô tả
    private int stock;                 // Số lượng tồn kho
    private String image;              // Tên file ảnh
    private int categoryId;            // Khóa ngoại đến bảng categories
    private String categoryName;       // Tên danh mục (JOIN từ bảng categories)
    
    // Tại sao dùng BigDecimal cho price?
    // - float/double có lỗi làm tròn: 0.1 + 0.2 = 0.30000000000000004
    // - BigDecimal chính xác tuyệt đối, phù hợp với tiền tệ
    
    // Tại sao có categoryName nếu đã có categoryId?
    // - categoryId: Lưu vào DB (foreign key)
    // - categoryName: Hiển thị trên giao diện (JOIN từ bảng categories)
    
    
    // ============================================
    // PHẦN 2: CONSTRUCTOR (Hàm khởi tạo)
    // ============================================
    
    /**
     * Constructor mặc định (no-argument constructor).
     * Cần thiết cho:
     *   - Framework (JSP, Servlet) tự động tạo object
     *   - Reflection API
     */
    public Product() {
        // Constructor rỗng
        // Java sẽ tự động gán giá trị mặc định:
        // - int: 0
        // - String: null
        // - BigDecimal: null
    }
    
    /**
     * Constructor đầy đủ tham số (dùng khi lấy từ DB).
     * Thứ tự tham số nên giống thứ tự cột trong SQL SELECT.
     * 
     * @param id           ID sản phẩm
     * @param name         Tên sản phẩm
     * @param price        Giá
     * @param description  Mô tả
     * @param stock        Số lượng
     * @param image        Tên file ảnh
     * @param categoryId   ID danh mục
     */
    public Product(int id, String name, BigDecimal price, String description, 
                   int stock, String image, int categoryId) {
        this.id = id;
        this.name = name;
        this.price = price;
        this.description = description;
        this.stock = stock;
        this.image = image;
        this.categoryId = categoryId;
    }
    
    /**
     * Constructor cho INSERT (không có id vì DB tự tăng).
     * Dùng khi thêm sản phẩm mới.
     */
    public Product(String name, BigDecimal price, String description, 
                   int stock, String image, int categoryId) {
        this.name = name;
        this.price = price;
        this.description = description;
        this.stock = stock;
        this.image = image;
        this.categoryId = categoryId;
    }
    
    
    // ============================================
    // PHẦN 3: GETTER & SETTER
    // ============================================
    
    /**
     * Getter/Setter tuân theo JavaBeans convention.
     * Quy tắc đặt tên:
     *   - Getter: get + TênThuộcTính (ví dụ: getName)
     *   - Setter: set + TênThuộcTính (ví dụ: setName)
     * 
     * Tại sao cần Getter/Setter?
     * 1. Encapsulation (đóng gói): Ẩn thuộc tính private
     * 2. Validation: Kiểm tra dữ liệu trước khi gán
     * 3. Compatibility: JSP/JSTL dùng getter để lấy dữ liệu
     *    Ví dụ: ${product.name} → gọi product.getName()
     */
    
    // --- Getter/Setter cho ID ---
    
    /**
     * Lấy ID sản phẩm.
     * @return ID sản phẩm (PRIMARY KEY)
     */
    public int getId() {
        return id;
    }
    
    /**
     * Gán ID sản phẩm.
     * Chú ý: Thường không gọi setter cho ID vì DB tự động tạo (AUTO_INCREMENT).
     * @param id ID mới
     */
    public void setId(int id) {
        this.id = id;
    }
    
    // --- Getter/Setter cho NAME ---
    
    /**
     * Lấy tên sản phẩm.
     * @return Tên sản phẩm
     */
    public String getName() {
        return name;
    }
    
    /**
     * Gán tên sản phẩm.
     * Validation: Không cho phép null hoặc rỗng.
     * @param name Tên sản phẩm
     */
    public void setName(String name) {
        // Kiểm tra validation
        if (name == null || name.trim().isEmpty()) {
            throw new IllegalArgumentException("Tên sản phẩm không được rỗng!");
        }
        this.name = name.trim();  // Loại bỏ khoảng trắng thừa
    }
    
    // --- Getter/Setter cho PRICE ---
    
    /**
     * Lấy giá sản phẩm.
     * @return Giá (BigDecimal)
     */
    public BigDecimal getPrice() {
        return price;
    }
    
    /**
     * Gán giá sản phẩm.
     * Validation: Giá phải > 0.
     * @param price Giá mới
     */
    public void setPrice(BigDecimal price) {
        if (price == null || price.compareTo(BigDecimal.ZERO) <= 0) {
            throw new IllegalArgumentException("Giá phải lớn hơn 0!");
        }
        this.price = price;
    }
    
    // --- Getter/Setter cho DESCRIPTION ---
    
    /**
     * Lấy mô tả sản phẩm.
     * @return Mô tả
     */
    public String getDescription() {
        return description;
    }
    
    /**
     * Gán mô tả sản phẩm.
     * @param description Mô tả mới
     */
    public void setDescription(String description) {
        this.description = description;
    }
    
    // --- Getter/Setter cho STOCK ---
    
    /**
     * Lấy số lượng tồn kho.
     * @return Số lượng
     */
    public int getStock() {
        return stock;
    }
    
    /**
     * Gán số lượng tồn kho.
     * Validation: Không âm.
     * @param stock Số lượng mới
     */
    public void setStock(int stock) {
        if (stock < 0) {
            throw new IllegalArgumentException("Số lượng không được âm!");
        }
        this.stock = stock;
    }
    
    // --- Getter/Setter cho IMAGE ---
    
    /**
     * Lấy tên file ảnh.
     * @return Tên file (ví dụ: "banhquy.jpg")
     */
    public String getImage() {
        return image;
    }
    
    /**
     * Gán tên file ảnh.
     * @param image Tên file mới
     */
    public void setImage(String image) {
        this.image = image;
    }
    
    // --- Getter/Setter cho CATEGORY_ID ---
    
    /**
     * Lấy ID danh mục (foreign key).
     * @return ID danh mục
     */
    public int getCategoryId() {
        return categoryId;
    }
    
    /**
     * Gán ID danh mục.
     * @param categoryId ID danh mục mới
     */
    public void setCategoryId(int categoryId) {
        this.categoryId = categoryId;
    }
    
    // --- Getter/Setter cho CATEGORY_NAME ---
    
    /**
     * Lấy tên danh mục (từ JOIN query).
     * Không lưu vào DB, chỉ dùng hiển thị.
     * @return Tên danh mục
     */
    public String getCategoryName() {
        return categoryName;
    }
    
    /**
     * Gán tên danh mục.
     * @param categoryName Tên danh mục
     */
    public void setCategoryName(String categoryName) {
        this.categoryName = categoryName;
    }
    
    
    // ============================================
    // PHẦN 4: PHƯƠNG THỨC TIỆN ÍCH
    // ============================================
    
    /**
     * Phương thức toString() để debug.
     * In ra thông tin object dạng chuỗi.
     * 
     * Ví dụ:
     *   System.out.println(product);
     *   → Product[id=1, name=Bánh quy, price=15000, ...]
     */
    @Override
    public String toString() {
        return "Product [id=" + id + ", name=" + name + ", price=" + price 
                + ", description=" + description + ", stock=" + stock 
                + ", image=" + image + ", categoryId=" + categoryId 
                + ", categoryName=" + categoryName + "]";
    }
    
    /**
     * Kiểm tra sản phẩm có còn hàng không.
     * @return true nếu stock > 0
     */
    public boolean isInStock() {
        return this.stock > 0;
    }
    
    /**
     * Định dạng giá tiền (ví dụ: 15000 → "15,000 đ").
     * Dùng trong JSP để hiển thị đẹp.
     * @return Chuỗi giá đã định dạng
     */
    public String getFormattedPrice() {
        if (price == null) return "0 đ";
        return String.format("%,.0f đ", price);
    }
    
    /**
     * Lấy đường dẫn ảnh đầy đủ.
     * Ví dụ: "banhquy.jpg" → "images/products/banhquy.jpg"
     * @return Đường dẫn ảnh
     */
    public String getImagePath() {
        if (image == null || image.isEmpty()) {
            return "images/products/default.jpg";  // Ảnh mặc định
        }
        return "images/products/" + image;
    }
}
```

### **1.3. Giải thích chi tiết**

#### **Q: Tại sao cần implements Serializable?**

**A**: 
```java
public class Product implements Serializable
```

**Serializable** cho phép object được:
1. **Lưu vào Session**: `session.setAttribute("product", product)`
2. **Truyền qua network**: RMI, EJB
3. **Lưu vào file**: ObjectOutputStream

**Trong dự án này**: Cần thiết khi lưu object vào Session (ví dụ: giỏ hàng).

---

#### **Q: Tại sao dùng private cho thuộc tính?**

**A**: 
```java
private int id;
private String name;
```

**Encapsulation** (Đóng gói):
- Ẩn dữ liệu, chỉ cho phép truy cập qua getter/setter
- Kiểm soát validation
- Dễ bảo trì: Thay đổi logic bên trong không ảnh hưởng code bên ngoài

**Ví dụ sai**:
```java
public class Product {
    public String name;  // KHÔNG TỐT!
}

// Code khác có thể gán lung tung:
product.name = "";  // Tên rỗng!
product.name = null;  // Null!
```

**Ví dụ đúng**:
```java
public void setName(String name) {
    if (name == null || name.trim().isEmpty()) {
        throw new IllegalArgumentException("Tên không được rỗng!");
    }
    this.name = name.trim();
}
```

---

#### **Q: Tại sao cần nhiều Constructor?**

**A**:

**Constructor 1: Mặc định (no-arg)**
```java
public Product() {}
```
- **Dùng khi**: Framework tự tạo object (JSP `<jsp:useBean>`)
- **Ví dụ**: Tạo object rỗng, sau đó dùng setter để gán dữ liệu

**Constructor 2: Đầy đủ tham số**
```java
public Product(int id, String name, BigDecimal price, ...) {}
```
- **Dùng khi**: Lấy dữ liệu từ ResultSet (DB)
- **Ví dụ trong DAO**:
```java
ResultSet rs = ps.executeQuery();
while (rs.next()) {
    Product p = new Product(
        rs.getInt("id"),
        rs.getString("name"),
        rs.getBigDecimal("price"),
        // ...
    );
    list.add(p);
}
```

**Constructor 3: Không có ID**
```java
public Product(String name, BigDecimal price, ...) {}
```
- **Dùng khi**: INSERT sản phẩm mới (ID do DB tự tăng)
- **Ví dụ**:
```java
Product newProduct = new Product("Bánh mì", new BigDecimal(5000), ...);
dao.insert(newProduct);  // DB sẽ tự tạo ID
```

---

#### **Q: ${product.name} trong JSP hoạt động như thế nào?**

**A**:

JSP sử dụng **Expression Language (EL)** để gọi getter:

```jsp
<!-- Trong JSP -->
<p>${product.name}</p>

<!-- JSP Engine tự động gọi -->
<p><%= product.getName() %></p>
```

**Quy tắc**:
- `${product.name}` → gọi `getName()`
- `${product.price}` → gọi `getPrice()`
- `${product.categoryId}` → gọi `getCategoryId()`

**Nếu không có getter** → Lỗi!

---

### **1.4. Bài tập tự thực hành**

**Bài 1**: Thêm thuộc tính `createdDate` (ngày tạo) vào Product
- Kiểu dữ liệu: `java.time.LocalDate`
- Getter/Setter
- Constructor

**Bài 2**: Viết method `getDiscountedPrice(int percent)` tính giá sau giảm
```java
// Ví dụ: Giá 100,000đ, giảm 20% → 80,000đ
public BigDecimal getDiscountedPrice(int percent) {
    // Code của bạn
}
```

**Bài 3**: Viết method `isExpensive()` kiểm tra sản phẩm có giá > 50,000đ không

---

<a name="phan-b2"></a>
## B2. DAO: ProductDAO.java

### **2.1. Vai trò của DAO**

**DAO (Data Access Object)** là lớp chịu trách nhiệm **giao tiếp với database**.

**Nhiệm vụ**:
1. Kết nối database
2. Thực thi SQL (SELECT, INSERT, UPDATE, DELETE)
3. Chuyển đổi ResultSet → Object
4. Xử lý exception
5. Đóng connection

**Tại sao tách DAO ra khỏi Servlet?**

**KHÔNG TỐT** (Code SQL trực tiếp trong Servlet):
```java
// ProductServlet.java
protected void doGet(HttpServletRequest request, HttpServletResponse response) {
    Connection conn = DriverManager.getConnection(...);
    PreparedStatement ps = conn.prepareStatement("SELECT * FROM products");
    ResultSet rs = ps.executeQuery();
    // ... xử lý ResultSet
}
```

**Vấn đề**:
- Code dài, khó đọc
- Trộn lẫn logic nghiệp vụ và truy cập DB
- Khó test
- Thay đổi DB phải sửa nhiều nơi

**TỐT** (Dùng DAO Pattern):
```java
// ProductServlet.java
protected void doGet(HttpServletRequest request, HttpServletResponse response) {
    ProductDAO dao = new ProductDAO();
    List<Product> products = dao.getAll();  // Gọn gàng!
    request.setAttribute("products", products);
}
```

**Lợi ích**:
- Servlet ngắn gọn, tập trung vào logic
- DAO có thể tái sử dụng
- Dễ test từng phần
- Thay đổi DB chỉ sửa DAO

---

### **2.2. Cấu trúc thư mục**

```
src/
  └── dao/
      ├── DBConnection.java       ← Lớp quản lý kết nối DB
      └── ProductDAO.java         ← Lớp truy vấn bảng products
```

---

### **2.3. Code: DBConnection.java**

**File này quản lý kết nối database** (Singleton Pattern).

```java
package dao;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

/**
 * Lớp DBConnection quản lý kết nối đến MySQL database.
 * Sử dụng Singleton Pattern để tránh tạo nhiều connection không cần thiết.
 * 
 * @author ThanhVien1
 */
public class DBConnection {
    
    // ============================================
    // PHẦN 1: THÔNG TIN KẾT NỐI
    // ============================================
    
    // Thông tin database (nên lưu trong file config hoặc environment variable)
    private static final String DB_URL = "jdbc:mysql://localhost:3306/snack_shop";
    private static final String DB_USER = "root";
    private static final String DB_PASSWORD = "your_password";  // Đổi password thật
    
    // Driver class của MySQL
    private static final String DB_DRIVER = "com.mysql.cj.jdbc.Driver";
    
    // Singleton instance
    private static DBConnection instance;
    private Connection connection;
    
    
    // ============================================
    // PHẦN 2: CONSTRUCTOR (Private)
    // ============================================
    
    /**
     * Constructor private để ngăn tạo instance từ bên ngoài.
     * Đây là đặc trưng của Singleton Pattern.
     */
    private DBConnection() {
        try {
            // Bước 1: Load MySQL JDBC Driver
            Class.forName(DB_DRIVER);
            
            // Bước 2: Tạo connection
            this.connection = DriverManager.getConnection(DB_URL, DB_USER, DB_PASSWORD);
            
            System.out.println("✅ Kết nối database thành công!");
            
        } catch (ClassNotFoundException e) {
            System.err.println("❌ Không tìm thấy MySQL Driver!");
            System.err.println("📦 Hãy thêm mysql-connector-java.jar vào project!");
            e.printStackTrace();
        } catch (SQLException e) {
            System.err.println("❌ Lỗi kết nối database!");
            System.err.println("🔍 Kiểm tra:");
            System.err.println("   - MySQL có đang chạy?");
            System.err.println("   - Database 'snack_shop' đã tạo chưa?");
            System.err.println("   - Username/Password đúng chưa?");
            e.printStackTrace();
        }
    }
    
    
    // ============================================
    // PHẦN 3: SINGLETON PATTERN
    // ============================================
    
    /**
     * Lấy instance duy nhất của DBConnection.
     * Thread-safe: Synchronized để tránh tạo nhiều instance khi có nhiều thread.
     * 
     * @return Instance DBConnection
     */
    public static synchronized DBConnection getInstance() {
        if (instance == null) {
            instance = new DBConnection();
        }
        return instance;
    }
    
    /**
     * Lấy Connection object.
     * Kiểm tra connection có còn sống không, nếu đóng thì tạo lại.
     * 
     * @return Connection object
     * @throws SQLException Nếu không thể kết nối
     */
    public Connection getConnection() throws SQLException {
        // Kiểm tra connection có bị đóng không
        if (connection == null || connection.isClosed()) {
            // Tạo lại connection
            connection = DriverManager.getConnection(DB_URL, DB_USER, DB_PASSWORD);
        }
        return connection;
    }
    
    
    // ============================================
    // PHẦN 4: ĐÓNG KẾT NỐI
    // ============================================
    
    /**
     * Đóng connection (gọi khi tắt ứng dụng).
     */
    public void closeConnection() {
        try {
            if (connection != null && !connection.isClosed()) {
                connection.close();
                System.out.println("✅ Đã đóng kết nối database");
            }
        } catch (SQLException e) {
            System.err.println("❌ Lỗi khi đóng connection!");
            e.printStackTrace();
        }
    }
}
```

**Giải thích chi tiết**:

**Q: Tại sao dùng Singleton Pattern?**

**A**: 
- **Vấn đề**: Mỗi lần gọi `new DBConnection()` sẽ tạo 1 kết nối mới → Tốn tài nguyên
- **Giải pháp**: Singleton đảm bảo chỉ có **1 instance duy nhất** trong toàn ứng dụng
- **Cách hoạt động**:
  1. Constructor private → Không thể `new DBConnection()` từ bên ngoài
  2. Method `getInstance()` static → Truy cập từ mọi nơi
  3. Kiểm tra `if (instance == null)` → Chỉ tạo 1 lần duy nhất

**Q: Tại sao cần synchronized?**

**A**:
```java
public static synchronized DBConnection getInstance()
```
- **Vấn đề**: Nhiều thread cùng gọi `getInstance()` cùng lúc → Có thể tạo nhiều instance
- **synchronized** đảm bảo chỉ 1 thread được vào method tại 1 thời điểm
- **Thread-safe**: An toàn trong môi trường multi-thread (Servlet container)

**Q: Class.forName() làm gì?**

**A**:
```java
Class.forName("com.mysql.cj.jdbc.Driver");
```
- **Load MySQL JDBC Driver** vào bộ nhớ
- **Chỉ cần gọi 1 lần** (thường trong static block hoặc constructor)
- **Từ JDBC 4.0**: Không bắt buộc phải gọi (auto-load), nhưng nên gọi để chắc chắn

---

### **2.4. Code đầy đủ: ProductDAO.java**

```java
package dao;

import model.Product;
import java.sql.*;
import java.util.ArrayList;
import java.util.List;
import java.math.BigDecimal;

/**
 * Lớp ProductDAO chịu trách nhiệm truy vấn bảng 'products'.
 * Cung cấp các phương thức CRUD (Create, Read, Update, Delete).
 * 
 * @author ThanhVien1
 */
public class ProductDAO {
    
    // Lấy connection từ DBConnection Singleton
    private Connection connection;
    
    /**
     * Constructor: Khởi tạo connection.
     */
    public ProductDAO() {
        try {
            this.connection = DBConnection.getInstance().getConnection();
        } catch (SQLException e) {
            System.err.println("❌ Lỗi khởi tạo ProductDAO!");
            e.printStackTrace();
        }
    }
    
    
    // ============================================
    // METHOD 1: getAll() - LẤY TẤT CẢ SẢN PHẨM
    // ============================================
    
    /**
     * Lấy danh sách TẤT CẢ sản phẩm từ database.
     * JOIN với bảng categories để lấy tên danh mục.
     * 
     * @return List<Product> danh sách sản phẩm
     */
    public List<Product> getAll() {
        List<Product> products = new ArrayList<>();
        
        // SQL Query: JOIN để lấy cả tên danh mục
        String sql = "SELECT p.id, p.name, p.price, p.description, " +
                     "       p.stock, p.image, p.category_id, c.name AS category_name " +
                     "FROM products p " +
                     "LEFT JOIN categories c ON p.category_id = c.id " +
                     "ORDER BY p.id DESC";  // Sản phẩm mới nhất lên đầu
        
        PreparedStatement ps = null;
        ResultSet rs = null;
        
        try {
            // Bước 1: Tạo PreparedStatement
            ps = connection.prepareStatement(sql);
            
            // Bước 2: Thực thi query (SELECT → executeQuery)
            rs = ps.executeQuery();
            
            // Bước 3: Duyệt ResultSet, chuyển từng dòng thành object Product
            while (rs.next()) {
                // Cách 1: Dùng constructor đầy đủ
                Product p = new Product(
                    rs.getInt("id"),
                    rs.getString("name"),
                    rs.getBigDecimal("price"),
                    rs.getString("description"),
                    rs.getInt("stock"),
                    rs.getString("image"),
                    rs.getInt("category_id")
                );
                
                // Gán category_name (từ JOIN)
                p.setCategoryName(rs.getString("category_name"));
                
                // Thêm vào danh sách
                products.add(p);
            }
            
            System.out.println("✅ Lấy được " + products.size() + " sản phẩm");
            
        } catch (SQLException e) {
            System.err.println("❌ Lỗi trong getAll()!");
            e.printStackTrace();
        } finally {
            // Bước 4: QUAN TRỌNG! Đóng resource
            closeResources(rs, ps);
        }
        
        return products;
    }
    
    
    // ============================================
    // METHOD 2: getById(int id) - LẤY 1 SẢN PHẨM THEO ID
    // ============================================
    
    /**
     * Lấy thông tin chi tiết 1 sản phẩm theo ID.
     * 
     * @param id ID sản phẩm cần tìm
     * @return Product object, hoặc null nếu không tìm thấy
     */
    public Product getById(int id) {
        Product product = null;
        
        String sql = "SELECT p.id, p.name, p.price, p.description, " +
                     "       p.stock, p.image, p.category_id, c.name AS category_name " +
                     "FROM products p " +
                     "LEFT JOIN categories c ON p.category_id = c.id " +
                     "WHERE p.id = ?";  // Dấu ? là placeholder
        
        PreparedStatement ps = null;
        ResultSet rs = null;
        
        try {
            ps = connection.prepareStatement(sql);
            
            // Gán giá trị cho placeholder (index bắt đầu từ 1)
            ps.setInt(1, id);  // Thay ? đầu tiên bằng giá trị id
            
            rs = ps.executeQuery();
            
            // ResultSet chỉ có 0 hoặc 1 dòng (vì id là PRIMARY KEY)
            if (rs.next()) {
                product = new Product(
                    rs.getInt("id"),
                    rs.getString("name"),
                    rs.getBigDecimal("price"),
                    rs.getString("description"),
                    rs.getInt("stock"),
                    rs.getString("image"),
                    rs.getInt("category_id")
                );
                product.setCategoryName(rs.getString("category_name"));
                
                System.out.println("✅ Tìm thấy sản phẩm: " + product.getName());
            } else {
                System.out.println("⚠️ Không tìm thấy sản phẩm ID = " + id);
            }
            
        } catch (SQLException e) {
            System.err.println("❌ Lỗi trong getById()!");
            e.printStackTrace();
        } finally {
            closeResources(rs, ps);
        }
        
        return product;
    }
    
    
    // ============================================
    // METHOD 3: getByCategoryId() - LỌC THEO DANH MỤC
    // ============================================
    
    /**
     * Lấy danh sách sản phẩm theo danh mục.
     * 
     * @param categoryId ID danh mục cần lọc
     * @return List<Product> danh sách sản phẩm thuộc danh mục đó
     */
    public List<Product> getByCategoryId(int categoryId) {
        List<Product> products = new ArrayList<>();
        
        String sql = "SELECT p.id, p.name, p.price, p.description, " +
                     "       p.stock, p.image, p.category_id, c.name AS category_name " +
                     "FROM products p " +
                     "LEFT JOIN categories c ON p.category_id = c.id " +
                     "WHERE p.category_id = ? " +
                     "ORDER BY p.name ASC";  // Sắp xếp theo tên A-Z
        
        PreparedStatement ps = null;
        ResultSet rs = null;
        
        try {
            ps = connection.prepareStatement(sql);
            ps.setInt(1, categoryId);
            rs = ps.executeQuery();
            
            while (rs.next()) {
                Product p = new Product(
                    rs.getInt("id"),
                    rs.getString("name"),
                    rs.getBigDecimal("price"),
                    rs.getString("description"),
                    rs.getInt("stock"),
                    rs.getString("image"),
                    rs.getInt("category_id")
                );
                p.setCategoryName(rs.getString("category_name"));
                products.add(p);
            }
            
            System.out.println("✅ Tìm thấy " + products.size() + " sản phẩm trong danh mục " + categoryId);
            
        } catch (SQLException e) {
            System.err.println("❌ Lỗi trong getByCategoryId()!");
            e.printStackTrace();
        } finally {
            closeResources(rs, ps);
        }
        
        return products;
    }
    
    
    // ============================================
    // METHOD 4: search() - TÌM KIẾM SẢN PHẨM
    // ============================================
    
    /**
     * Tìm kiếm sản phẩm theo từ khóa (trong tên hoặc mô tả).
     * 
     * @param keyword Từ khóa tìm kiếm
     * @return List<Product> danh sách sản phẩm phù hợp
     */
    public List<Product> search(String keyword) {
        List<Product> products = new ArrayList<>();
        
        // LIKE '%keyword%': Tìm chuỗi con
        // Ví dụ: keyword = "bánh" → Tìm "Bánh quy", "Bánh mì", "Kẹo bánh"...
        String sql = "SELECT p.id, p.name, p.price, p.description, " +
                     "       p.stock, p.image, p.category_id, c.name AS category_name " +
                     "FROM products p " +
                     "LEFT JOIN categories c ON p.category_id = c.id " +
                     "WHERE p.name LIKE ? OR p.description LIKE ? " +
                     "ORDER BY p.name ASC";
        
        PreparedStatement ps = null;
        ResultSet rs = null;
        
        try {
            ps = connection.prepareStatement(sql);
            
            // Thêm % để tìm chuỗi con
            String searchPattern = "%" + keyword + "%";
            
            // Gán cùng 1 giá trị cho 2 placeholder (name và description)
            ps.setString(1, searchPattern);  // Tìm trong name
            ps.setString(2, searchPattern);  // Tìm trong description
            
            rs = ps.executeQuery();
            
            while (rs.next()) {
                Product p = new Product(
                    rs.getInt("id"),
                    rs.getString("name"),
                    rs.getBigDecimal("price"),
                    rs.getString("description"),
                    rs.getInt("stock"),
                    rs.getString("image"),
                    rs.getInt("category_id")
                );
                p.setCategoryName(rs.getString("category_name"));
                products.add(p);
            }
            
            System.out.println("✅ Tìm thấy " + products.size() + " sản phẩm với từ khóa: " + keyword);
            
        } catch (SQLException e) {
            System.err.println("❌ Lỗi trong search()!");
            e.printStackTrace();
        } finally {
            closeResources(rs, ps);
        }
        
        return products;
    }
    
    
    // ============================================
    // METHOD 5: insert() - THÊM SẢN PHẨM MỚI
    // ============================================
    
    /**
     * Thêm sản phẩm mới vào database.
     * 
     * @param product Object Product cần thêm
     * @return true nếu thành công, false nếu thất bại
     */
    public boolean insert(Product product) {
        // SQL INSERT: Không cần truyền id (AUTO_INCREMENT)
        String sql = "INSERT INTO products (name, price, description, stock, image, category_id) " +
                     "VALUES (?, ?, ?, ?, ?, ?)";
        
        PreparedStatement ps = null;
        
        try {
            ps = connection.prepareStatement(sql);
            
            // Gán giá trị cho các placeholder (thứ tự phải đúng)
            ps.setString(1, product.getName());
            ps.setBigDecimal(2, product.getPrice());
            ps.setString(3, product.getDescription());
            ps.setInt(4, product.getStock());
            ps.setString(5, product.getImage());
            ps.setInt(6, product.getCategoryId());
            
            // Thực thi INSERT (executeUpdate trả về số dòng bị ảnh hưởng)
            int rowsAffected = ps.executeUpdate();
            
            if (rowsAffected > 0) {
                System.out.println("✅ Thêm sản phẩm thành công: " + product.getName());
                return true;
            } else {
                System.out.println("⚠️ Không thêm được sản phẩm!");
                return false;
            }
            
        } catch (SQLException e) {
            System.err.println("❌ Lỗi trong insert()!");
            
            // Kiểm tra lỗi foreign key
            if (e.getMessage().contains("foreign key")) {
                System.err.println("🔍 category_id không tồn tại trong bảng categories!");
            }
            
            e.printStackTrace();
            return false;
        } finally {
            closeResources(null, ps);
        }
    }
    
    
    // ============================================
    // METHOD 6: update() - CẬP NHẬT SẢN PHẨM
    // ============================================
    
    /**
     * Cập nhật thông tin sản phẩm.
     * 
     * @param product Object Product chứa dữ liệu mới
     * @return true nếu thành công, false nếu thất bại
     */
    public boolean update(Product product) {
        String sql = "UPDATE products " +
                     "SET name = ?, price = ?, description = ?, " +
                     "    stock = ?, image = ?, category_id = ? " +
                     "WHERE id = ?";
        
        PreparedStatement ps = null;
        
        try {
            ps = connection.prepareStatement(sql);
            
            // Gán giá trị (6 cột cần update + 1 id trong WHERE)
            ps.setString(1, product.getName());
            ps.setBigDecimal(2, product.getPrice());
            ps.setString(3, product.getDescription());
            ps.setInt(4, product.getStock());
            ps.setString(5, product.getImage());
            ps.setInt(6, product.getCategoryId());
            ps.setInt(7, product.getId());  // Điều kiện WHERE
            
            int rowsAffected = ps.executeUpdate();
            
            if (rowsAffected > 0) {
                System.out.println("✅ Cập nhật sản phẩm thành công: ID = " + product.getId());
                return true;
            } else {
                System.out.println("⚠️ Không tìm thấy sản phẩm ID = " + product.getId());
                return false;
            }
            
        } catch (SQLException e) {
            System.err.println("❌ Lỗi trong update()!");
            e.printStackTrace();
            return false;
        } finally {
            closeResources(null, ps);
        }
    }
    
    
    // ============================================
    // METHOD 7: delete() - XÓA SẢN PHẨM
    // ============================================
    
    /**
     * Xóa sản phẩm khỏi database.
     * Chú ý: Kiểm tra ràng buộc foreign key (order_items, cart_items).
     * 
     * @param id ID sản phẩm cần xóa
     * @return true nếu thành công, false nếu thất bại
     */
    public boolean delete(int id) {
        String sql = "DELETE FROM products WHERE id = ?";
        
        PreparedStatement ps = null;
        
        try {
            ps = connection.prepareStatement(sql);
            ps.setInt(1, id);
            
            int rowsAffected = ps.executeUpdate();
            
            if (rowsAffected > 0) {
                System.out.println("✅ Xóa sản phẩm thành công: ID = " + id);
                return true;
            } else {
                System.out.println("⚠️ Không tìm thấy sản phẩm ID = " + id);
                return false;
            }
            
        } catch (SQLException e) {
            System.err.println("❌ Lỗi trong delete()!");
            
            // Kiểm tra lỗi foreign key constraint
            if (e.getMessage().contains("foreign key") || e.getMessage().contains("CONSTRAINT")) {
                System.err.println("🔍 Không thể xóa vì sản phẩm đang được tham chiếu ở bảng khác!");
                System.err.println("💡 Giải pháp: Xóa các bản ghi liên quan trong order_items, cart_items trước");
            }
            
            e.printStackTrace();
            return false;
        } finally {
            closeResources(null, ps);
        }
    }
    
    
    // ============================================
    // METHOD 8: getTotalCount() - ĐẾM TỔNG SỐ SẢN PHẨM
    // ============================================
    
    /**
     * Đếm tổng số sản phẩm trong database.
     * Dùng cho phân trang (pagination).
     * 
     * @return Tổng số sản phẩm
     */
    public int getTotalCount() {
        String sql = "SELECT COUNT(*) AS total FROM products";
        
        PreparedStatement ps = null;
        ResultSet rs = null;
        int total = 0;
        
        try {
            ps = connection.prepareStatement(sql);
            rs = ps.executeQuery();
            
            if (rs.next()) {
                total = rs.getInt("total");  // Hoặc rs.getInt(1)
            }
            
            System.out.println("✅ Tổng số sản phẩm: " + total);
            
        } catch (SQLException e) {
            System.err.println("❌ Lỗi trong getTotalCount()!");
            e.printStackTrace();
        } finally {
            closeResources(rs, ps);
        }
        
        return total;
    }
    
    
    // ============================================
    // HELPER METHOD: ĐÓNG RESOURCE
    // ============================================
    
    /**
     * Đóng ResultSet và PreparedStatement.
     * QUAN TRỌNG: Luôn đóng trong block finally để tránh memory leak.
     * 
     * @param rs ResultSet cần đóng
     * @param ps PreparedStatement cần đóng
     */
    private void closeResources(ResultSet rs, PreparedStatement ps) {
        try {
            if (rs != null) {
                rs.close();
            }
            if (ps != null) {
                ps.close();
            }
        } catch (SQLException e) {
            System.err.println("❌ Lỗi khi đóng resource!");
            e.printStackTrace();
        }
    }
}
```

---

### **2.5. Giải thích chi tiết từng method**

#### **METHOD 1: getAll()**

**Mục đích**: Lấy tất cả sản phẩm từ DB, hiển thị danh sách.

**SQL Query giải thích**:
```sql
SELECT p.id, p.name, p.price, p.description, 
       p.stock, p.image, p.category_id, c.name AS category_name
FROM products p
LEFT JOIN categories c ON p.category_id = c.id
ORDER BY p.id DESC;
```

**Phân tích**:
- `p.id, p.name, ...`: Lấy các cột từ bảng `products` (alias `p`)
- `c.name AS category_name`: Lấy tên danh mục, đổi tên cột thành `category_name`
- `LEFT JOIN`: Lấy tất cả products, kể cả khi không có danh mục (category_id = NULL)
- `ORDER BY p.id DESC`: Sắp xếp giảm dần theo id (sản phẩm mới nhất lên đầu)

**Tại sao dùng LEFT JOIN thay vì INNER JOIN?**
- **INNER JOIN**: Chỉ lấy products CÓ danh mục
- **LEFT JOIN**: Lấy TẤT CẢ products (kể cả không có danh mục)
- Trong trường hợp này: Nếu `category_id = NULL` hoặc không tồn tại trong bảng `categories`, vẫn lấy sản phẩm

**Code Java giải thích**:
```java
while (rs.next()) {
    Product p = new Product(
        rs.getInt("id"),           // Lấy cột "id" kiểu int
        rs.getString("name"),       // Lấy cột "name" kiểu String
        rs.getBigDecimal("price"),  // Lấy cột "price" kiểu DECIMAL
        // ...
    );
    
    p.setCategoryName(rs.getString("category_name"));  // Từ JOIN
    products.add(p);
}
```

**Quy trình**:
1. Tạo `PreparedStatement` với SQL
2. `executeQuery()` → Nhận `ResultSet`
3. Duyệt `ResultSet` bằng `while (rs.next())`
4. Mỗi dòng → Tạo 1 `Product` object
5. Thêm vào `List<Product>`
6. Đóng resource

---

#### **METHOD 2: getById(int id)**

**Mục đích**: Lấy 1 sản phẩm cụ thể (xem chi tiết, chỉnh sửa).

**SQL với PreparedStatement**:
```sql
SELECT ... WHERE p.id = ?
```

**Tại sao dùng dấu `?` (placeholder)?**
- **An toàn**: Chống SQL Injection
- **Tái sử dụng**: SQL được compile trước, thay đổi tham số nhanh hơn
- **Dễ đọc**: Tách biệt logic và dữ liệu

**Gán giá trị cho placeholder**:
```java
ps.setInt(1, id);  // Gán giá trị id cho placeholder thứ 1
```
**Chú ý**: Index bắt đầu từ **1** (không phải 0 như mảng).

**Xử lý kết quả**:
```java
if (rs.next()) {
    // Tìm thấy → Tạo object
    product = new Product(...);
} else {
    // Không tìm thấy → Trả về null
    product = null;
}
```

---

#### **METHOD 3: getByCategoryId()**

**Mục đích**: Lọc sản phẩm theo danh mục (ví dụ: chỉ hiển thị "Bánh kẹo").

**SQL**:
```sql
WHERE p.category_id = ?
ORDER BY p.name ASC  -- Sắp xếp A-Z
```

**Sử dụng trong Servlet**:
```java
// User click danh mục "Bánh kẹo" (ID = 2)
int categoryId = Integer.parseInt(request.getParameter("categoryId"));
List<Product> products = dao.getByCategoryId(categoryId);
```

---

#### **METHOD 4: search(String keyword)**

**Mục đích**: Tìm kiếm sản phẩm theo tên hoặc mô tả.

**SQL với LIKE**:
```sql
WHERE p.name LIKE ? OR p.description LIKE ?
```

**Sử dụng % (wildcard)**:
```java
String searchPattern = "%" + keyword + "%";
ps.setString(1, searchPattern);  // Tìm trong name
ps.setString(2, searchPattern);  // Tìm trong description
```

**Ví dụ**:
- User nhập: `"bánh"`
- SQL thành: `WHERE name LIKE '%bánh%' OR description LIKE '%bánh%'`
- Kết quả: "**Bánh** quy", "**Bánh** mì", "Kẹo **bánh**", ...

**Chú ý**:
- `%` = Bất kỳ ký tự nào (0 hoặc nhiều ký tự)
- `_` = Đúng 1 ký tự
- Ví dụ: `"b_nh"` → Tìm "bành", "bình", "bênh", ...

---

#### **METHOD 5: insert(Product product)**

**Mục đích**: Thêm sản phẩm mới vào DB.

**SQL INSERT**:
```sql
INSERT INTO products (name, price, description, stock, image, category_id)
VALUES (?, ?, ?, ?, ?, ?)
```

**Chú ý**: KHÔNG có cột `id` vì DB tự động tạo (AUTO_INCREMENT).

**executeUpdate() vs executeQuery()**:

| Method | Dùng cho | Trả về |
|--------|----------|--------|
| `executeQuery()` | SELECT | ResultSet (dữ liệu) |
| `executeUpdate()` | INSERT, UPDATE, DELETE | int (số dòng bị ảnh hưởng) |

**Code**:
```java
int rowsAffected = ps.executeUpdate();
if (rowsAffected > 0) {
    // Thành công
    return true;
} else {
    // Thất bại (không có dòng nào bị ảnh hưởng)
    return false;
}
```

**Xử lý lỗi Foreign Key**:
```java
catch (SQLException e) {
    if (e.getMessage().contains("foreign key")) {
        System.err.println("category_id không tồn tại!");
    }
}
```

---

#### **METHOD 6: update(Product product)**

**Mục đích**: Cập nhật thông tin sản phẩm hiện có.

**SQL UPDATE**:
```sql
UPDATE products
SET name = ?, price = ?, description = ?, stock = ?, image = ?, category_id = ?
WHERE id = ?
```

**Tham số**:
- 6 tham số đầu: Dữ liệu mới
- Tham số thứ 7: ID sản phẩm cần update (điều kiện WHERE)

**Code**:
```java
ps.setString(1, product.getName());      // SET name = ?
ps.setString(2, product.getPrice());     // SET price = ?
// ...
ps.setInt(7, product.getId());           // WHERE id = ?
```

---

#### **METHOD 7: delete(int id)**

**Mục đích**: Xóa sản phẩm khỏi DB.

**SQL DELETE**:
```sql
DELETE FROM products WHERE id = ?
```

**Vấn đề Foreign Key Constraint**:
- Nếu sản phẩm đã có trong `order_items` hoặc `cart_items` → **KHÔNG thể xóa**!
- MySQL sẽ throw `SQLException` với message "foreign key constraint fails"

**Giải pháp**:
1. **Xóa cascade** (cấu hình DB):
   ```sql
   FOREIGN KEY (product_id) REFERENCES products(id) ON DELETE CASCADE
   ```
2. **Xóa thủ công**: Xóa các bản ghi liên quan trước
3. **Soft delete**: Thêm cột `deleted` (boolean), đánh dấu xóa thay vì xóa thật

**Xử lý lỗi**:
```java
if (e.getMessage().contains("foreign key")) {
    System.err.println("Không thể xóa vì sản phẩm đang được sử dụng!");
}
```

---

#### **METHOD 8: getTotalCount()**

**Mục đích**: Đếm tổng số sản phẩm (dùng cho phân trang).

**SQL COUNT**:
```sql
SELECT COUNT(*) AS total FROM products
```

**Code**:
```java
if (rs.next()) {
    total = rs.getInt("total");  // Lấy giá trị cột "total"
    // Hoặc: total = rs.getInt(1);  // Lấy cột đầu tiên
}
```

**Sử dụng trong phân trang**:
```java
int total = dao.getTotalCount();        // Ví dụ: 100 sản phẩm
int pageSize = 10;                      // Mỗi trang 10 sản phẩm
int totalPages = (int) Math.ceil(total / (double) pageSize);  // 10 trang
```

---

### **2.6. Tại sao cần đóng resource trong finally?**

**Vấn đề**: Connection là tài nguyên giới hạn.

**SAI**:
```java
PreparedStatement ps = connection.prepareStatement(sql);
ResultSet rs = ps.executeQuery();
// ... xử lý

// Nếu có exception → Không đóng được resource → Memory leak!
rs.close();
ps.close();
```

**ĐÚNG**:
```java
PreparedStatement ps = null;
ResultSet rs = null;
try {
    ps = connection.prepareStatement(sql);
    rs = ps.executeQuery();
    // ... xử lý
} catch (SQLException e) {
    e.printStackTrace();
} finally {
    // Luôn chạy dù có exception hay không
    closeResources(rs, ps);
}
```

**Finally block**:
- **Luôn được thực thi**, kể cả khi có exception hoặc return
- Đảm bảo resource được giải phóng

---

### **2.7. Bài tập tự thực hành**

**Bài 1**: Viết method `getByPriceRange(BigDecimal min, BigDecimal max)`
```java
// Lấy sản phẩm có giá từ 10,000 đến 50,000
List<Product> products = dao.getByPriceRange(
    new BigDecimal(10000), 
    new BigDecimal(50000)
);
```

**Bài 2**: Viết method `getTopSelling(int limit)` lấy top N sản phẩm bán chạy
- JOIN với bảng `order_items`
- ORDER BY số lượng bán
- LIMIT N

**Bài 3**: Viết method `updateStock(int productId, int quantity)` cập nhật số lượng tồn kho
```sql
UPDATE products SET stock = stock + ? WHERE id = ?
```

---

(Phần tiếp theo sẽ là **B3. Servlet: ProductServlet.java** - tôi sẽ tiếp tục trong message kế tiếp)
<a name="phan-b3"></a>
## B3. Servlet: ProductServlet.java

### **3.1. Vai trò của Servlet**

**Servlet** là **Controller** trong mô hình MVC, chịu trách nhiệm:

1. **Nhận request** từ client (browser)
2. **Phân tích action** (list, add, edit, delete, ...)
3. **Gọi DAO** để truy vấn/cập nhật database
4. **Chuẩn bị dữ liệu** cho View (đưa vào request scope)
5. **Chọn View phù hợp** (forward/redirect)

**Servlet KHÔNG làm**:
- Truy cập database trực tiếp (→ Gọi DAO)
- Viết HTML trực tiếp (→ Forward sang JSP)
- Chứa logic phức tạp (→ Đưa vào Service layer nếu cần)

---

### **3.2. Cấu trúc ProductServlet**

```
ProductServlet.java
├── doGet()              ← Xử lý GET request (hiển thị)
│   ├── action=list      → Danh sách sản phẩm
│   ├── action=detail    → Chi tiết 1 sản phẩm
│   ├── action=add       → Form thêm mới
│   └── action=edit      → Form sửa
└── doPost()             ← Xử lý POST request (submit form)
    ├── action=add       → Thực hiện thêm
    ├── action=update    → Thực hiện sửa
    └── action=delete    → Thực hiện xóa
```

---

### **3.3. Code đầy đủ: ProductServlet.java**

```java
package controller;

import dao.ProductDAO;
import model.Product;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.math.BigDecimal;
import java.util.List;

/**
 * Servlet xử lý các request liên quan đến sản phẩm.
 * URL pattern: /ProductServlet
 * 
 * @author ThanhVien1
 */
@WebServlet("/ProductServlet")  // Annotation mapping (thay cho web.xml)
public class ProductServlet extends HttpServlet {
    
    private static final long serialVersionUID = 1L;
    
    // DAO object - khởi tạo 1 lần, dùng cho tất cả request
    private ProductDAO productDAO;
    
    
    // ============================================
    // INIT METHOD - KHỞI TẠO SERVLET
    // ============================================
    
    /**
     * Method init() được gọi 1 lần duy nhất khi Servlet được tạo.
     * Dùng để khởi tạo các resource (DAO, connection pool, ...).
     */
    @Override
    public void init() throws ServletException {
        super.init();
        this.productDAO = new ProductDAO();
        System.out.println("✅ ProductServlet đã khởi tạo!");
    }
    
    
    // ============================================
    // PHẦN 1: doGet() - XỬ LÝ GET REQUEST
    // ============================================
    
    /**
     * Xử lý GET request (hiển thị dữ liệu, form).
     * GET request không thay đổi dữ liệu trên server.
     * 
     * Các action hỗ trợ:
     * - list: Danh sách tất cả sản phẩm
     * - detail: Chi tiết 1 sản phẩm
     * - add: Form thêm sản phẩm mới
     * - edit: Form sửa sản phẩm
     */
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        
        // Bước 1: Lấy action từ URL parameter
        // Ví dụ: ProductServlet?action=list
        String action = request.getParameter("action");
        
        // Nếu không có action → Mặc định là "list"
        if (action == null || action.isEmpty()) {
            action = "list";
        }
        
        System.out.println("📥 GET request - Action: " + action);
        
        // Bước 2: Phân luồng xử lý theo action
        switch (action) {
            case "list":
                showProductList(request, response);
                break;
            case "detail":
                showProductDetail(request, response);
                break;
            case "add":
                showAddForm(request, response);
                break;
            case "edit":
                showEditForm(request, response);
                break;
            default:
                // Action không hợp lệ → Trả về list
                showProductList(request, response);
        }
    }
    
    
    // --------------------------------------------
    // ACTION 1: HIỂN THỊ DANH SÁCH SẢN PHẨM
    // --------------------------------------------
    
    /**
     * Hiển thị danh sách tất cả sản phẩm.
     * URL: ProductServlet?action=list
     */
    private void showProductList(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        
        // Bước 1: Gọi DAO lấy dữ liệu
        List<Product> productList = productDAO.getAll();
        
        // Bước 2: Đưa dữ liệu vào request scope
        // JSP sẽ lấy ra bằng ${productList}
        request.setAttribute("productList", productList);
        
        // (Optional) Đưa thêm thông tin khác
        request.setAttribute("totalProducts", productList.size());
        
        // Bước 3: Forward sang JSP
        // Forward: Server xử lý, URL không thay đổi
        request.getRequestDispatcher("views/product_list.jsp")
               .forward(request, response);
        
        System.out.println("✅ Đã forward sang product_list.jsp");
    }
    
    
    // --------------------------------------------
    // ACTION 2: HIỂN THỊ CHI TIẾT SẢN PHẨM
    // --------------------------------------------
    
    /**
     * Hiển thị chi tiết 1 sản phẩm.
     * URL: ProductServlet?action=detail&id=5
     */
    private void showProductDetail(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        
        try {
            // Bước 1: Lấy id từ URL parameter
            String idStr = request.getParameter("id");
            int id = Integer.parseInt(idStr);  // Chuyển String → int
            
            // Bước 2: Gọi DAO lấy sản phẩm
            Product product = productDAO.getById(id);
            
            // Bước 3: Kiểm tra tồn tại
            if (product != null) {
                // Tìm thấy → Đưa vào request scope
                request.setAttribute("product", product);
                
                // Forward sang trang chi tiết
                request.getRequestDispatcher("views/product_detail.jsp")
                       .forward(request, response);
            } else {
                // Không tìm thấy → Hiển thị lỗi
                request.setAttribute("errorMessage", "Không tìm thấy sản phẩm ID = " + id);
                request.getRequestDispatcher("views/error.jsp")
                       .forward(request, response);
            }
            
        } catch (NumberFormatException e) {
            // URL không hợp lệ (id không phải số)
            request.setAttribute("errorMessage", "ID sản phẩm không hợp lệ!");
            request.getRequestDispatcher("views/error.jsp")
                   .forward(request, response);
        }
    }
    
    
    // --------------------------------------------
    // ACTION 3: HIỂN THỊ FORM THÊM SẢN PHẨM
    // --------------------------------------------
    
    /**
     * Hiển thị form thêm sản phẩm mới.
     * URL: ProductServlet?action=add
     */
    private void showAddForm(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        
        // Lấy danh sách categories (cho dropdown)
        // Giả sử có CategoryDAO
        // List<Category> categories = categoryDAO.getAll();
        // request.setAttribute("categories", categories);
        
        // Forward sang form thêm
        request.getRequestDispatcher("views/product_add.jsp")
               .forward(request, response);
    }
    
    
    // --------------------------------------------
    // ACTION 4: HIỂN THỊ FORM SỬA SẢN PHẨM
    // --------------------------------------------
    
    /**
     * Hiển thị form sửa sản phẩm.
     * URL: ProductServlet?action=edit&id=5
     */
    private void showEditForm(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        
        try {
            // Lấy id sản phẩm cần sửa
            String idStr = request.getParameter("id");
            int id = Integer.parseInt(idStr);
            
            // Lấy thông tin sản phẩm hiện tại
            Product product = productDAO.getById(id);
            
            if (product != null) {
                // Đưa vào request scope để JSP hiển thị sẵn
                request.setAttribute("product", product);
                
                // Lấy danh sách categories (cho dropdown)
                // request.setAttribute("categories", categoryDAO.getAll());
                
                // Forward sang form sửa
                request.getRequestDispatcher("views/product_edit.jsp")
                       .forward(request, response);
            } else {
                // Không tìm thấy
                request.setAttribute("errorMessage", "Không tìm thấy sản phẩm cần sửa!");
                request.getRequestDispatcher("views/error.jsp")
                       .forward(request, response);
            }
            
        } catch (NumberFormatException e) {
            request.setAttribute("errorMessage", "ID không hợp lệ!");
            request.getRequestDispatcher("views/error.jsp")
                   .forward(request, response);
        }
    }
    
    
    // ============================================
    // PHẦN 2: doPost() - XỬ LÝ POST REQUEST
    // ============================================
    
    /**
     * Xử lý POST request (submit form, thay đổi dữ liệu).
     * POST request thường thay đổi dữ liệu trên server.
     * 
     * Các action hỗ trợ:
     * - add: Thực hiện thêm sản phẩm mới
     * - update: Thực hiện cập nhật sản phẩm
     * - delete: Thực hiện xóa sản phẩm
     */
    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        
        // Bước 1: Set encoding để xử lý tiếng Việt
        request.setCharacterEncoding("UTF-8");
        response.setCharacterEncoding("UTF-8");
        
        // Bước 2: Lấy action từ form
        String action = request.getParameter("action");
        
        System.out.println("📤 POST request - Action: " + action);
        
        // Bước 3: Phân luồng xử lý
        switch (action) {
            case "add":
                addProduct(request, response);
                break;
            case "update":
                updateProduct(request, response);
                break;
            case "delete":
                deleteProduct(request, response);
                break;
            default:
                // Action không hợp lệ
                response.sendRedirect("ProductServlet?action=list");
        }
    }
    
    
    // --------------------------------------------
    // ACTION 5: THÊM SẢN PHẨM MỚI
    // --------------------------------------------
    
    /**
     * Xử lý thêm sản phẩm mới.
     * Nhận dữ liệu từ form, gọi DAO insert vào DB.
     */
    private void addProduct(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        
        try {
            // Bước 1: Lấy dữ liệu từ form
            String name = request.getParameter("name");
            String priceStr = request.getParameter("price");
            String description = request.getParameter("description");
            String stockStr = request.getParameter("stock");
            String image = request.getParameter("image");
            String categoryIdStr = request.getParameter("categoryId");
            
            // Bước 2: Validate dữ liệu
            if (name == null || name.trim().isEmpty()) {
                throw new IllegalArgumentException("Tên sản phẩm không được rỗng!");
            }
            
            // Chuyển đổi kiểu dữ liệu
            BigDecimal price = new BigDecimal(priceStr);
            int stock = Integer.parseInt(stockStr);
            int categoryId = Integer.parseInt(categoryIdStr);
            
            // Validate giá trị
            if (price.compareTo(BigDecimal.ZERO) <= 0) {
                throw new IllegalArgumentException("Giá phải lớn hơn 0!");
            }
            if (stock < 0) {
                throw new IllegalArgumentException("Số lượng không được âm!");
            }
            
            // Bước 3: Tạo object Product
            Product newProduct = new Product(name, price, description, stock, image, categoryId);
            
            // Bước 4: Gọi DAO insert
            boolean success = productDAO.insert(newProduct);
            
            // Bước 5: Xử lý kết quả
            if (success) {
                // Thành công → Redirect về danh sách
                // Redirect: Browser gửi request mới, URL thay đổi
                response.sendRedirect("ProductServlet?action=list&message=add_success");
            } else {
                // Thất bại → Hiển thị lỗi
                request.setAttribute("errorMessage", "Thêm sản phẩm thất bại!");
                request.getRequestDispatcher("views/error.jsp")
                       .forward(request, response);
            }
            
        } catch (NumberFormatException e) {
            // Lỗi định dạng số
            request.setAttribute("errorMessage", "Dữ liệu nhập không hợp lệ! Kiểm tra lại giá, số lượng.");
            request.getRequestDispatcher("views/error.jsp")
                   .forward(request, response);
        } catch (IllegalArgumentException e) {
            // Lỗi validation
            request.setAttribute("errorMessage", e.getMessage());
            request.getRequestDispatcher("views/error.jsp")
                   .forward(request, response);
        }
    }
    
    
    // --------------------------------------------
    // ACTION 6: CẬP NHẬT SẢN PHẨM
    // --------------------------------------------
    
    /**
     * Xử lý cập nhật sản phẩm.
     * Nhận dữ liệu từ form edit, gọi DAO update.
     */
    private void updateProduct(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        
        try {
            // Lấy dữ liệu từ form (bao gồm cả id)
            String idStr = request.getParameter("id");
            String name = request.getParameter("name");
            String priceStr = request.getParameter("price");
            String description = request.getParameter("description");
            String stockStr = request.getParameter("stock");
            String image = request.getParameter("image");
            String categoryIdStr = request.getParameter("categoryId");
            
            // Chuyển đổi kiểu dữ liệu
            int id = Integer.parseInt(idStr);
            BigDecimal price = new BigDecimal(priceStr);
            int stock = Integer.parseInt(stockStr);
            int categoryId = Integer.parseInt(categoryIdStr);
            
            // Validate
            if (name == null || name.trim().isEmpty()) {
                throw new IllegalArgumentException("Tên sản phẩm không được rỗng!");
            }
            if (price.compareTo(BigDecimal.ZERO) <= 0) {
                throw new IllegalArgumentException("Giá phải lớn hơn 0!");
            }
            
            // Tạo object Product với dữ liệu mới
            Product product = new Product(id, name, price, description, stock, image, categoryId);
            
            // Gọi DAO update
            boolean success = productDAO.update(product);
            
            if (success) {
                // Thành công → Redirect về danh sách
                response.sendRedirect("ProductServlet?action=list&message=update_success");
            } else {
                // Thất bại
                request.setAttribute("errorMessage", "Cập nhật sản phẩm thất bại!");
                request.getRequestDispatcher("views/error.jsp")
                       .forward(request, response);
            }
            
        } catch (NumberFormatException e) {
            request.setAttribute("errorMessage", "Dữ liệu nhập không hợp lệ!");
            request.getRequestDispatcher("views/error.jsp")
                   .forward(request, response);
        } catch (IllegalArgumentException e) {
            request.setAttribute("errorMessage", e.getMessage());
            request.getRequestDispatcher("views/error.jsp")
                   .forward(request, response);
        }
    }
    
    
    // --------------------------------------------
    // ACTION 7: XÓA SẢN PHẨM
    // --------------------------------------------
    
    /**
     * Xử lý xóa sản phẩm.
     * Nhận id từ request, gọi DAO delete.
     */
    private void deleteProduct(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        
        try {
            // Lấy id sản phẩm cần xóa
            String idStr = request.getParameter("id");
            int id = Integer.parseInt(idStr);
            
            // Gọi DAO delete
            boolean success = productDAO.delete(id);
            
            if (success) {
                // Thành công
                response.sendRedirect("ProductServlet?action=list&message=delete_success");
            } else {
                // Thất bại (có thể do foreign key constraint)
                request.setAttribute("errorMessage", 
                    "Không thể xóa sản phẩm! Có thể sản phẩm đang được sử dụng trong đơn hàng.");
                request.getRequestDispatcher("views/error.jsp")
                       .forward(request, response);
            }
            
        } catch (NumberFormatException e) {
            request.setAttribute("errorMessage", "ID không hợp lệ!");
            request.getRequestDispatcher("views/error.jsp")
                   .forward(request, response);
        }
    }
    
    
    // ============================================
    // DESTROY METHOD - HỦY SERVLET
    // ============================================
    
    /**
     * Method destroy() được gọi khi Servlet bị hủy (server tắt).
     * Dùng để giải phóng resource.
     */
    @Override
    public void destroy() {
        // Đóng connection (nếu cần)
        System.out.println("❌ ProductServlet đã bị hủy!");
        super.destroy();
    }
}
```

---

### **3.4. Giải thích chi tiết**

#### **Q: @WebServlet là gì?**

**A**:
```java
@WebServlet("/ProductServlet")
```

**Annotation mapping** (từ Servlet 3.0):
- **Thay thế web.xml**: Không cần khai báo trong file cấu hình
- **URL pattern**: Servlet này xử lý URL `http://localhost:8080/SnackShop/ProductServlet`

**Tương đương với web.xml**:
```xml
<servlet>
    <servlet-name>ProductServlet</servlet-name>
    <servlet-class>controller.ProductServlet</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>ProductServlet</servlet-name>
    <url-pattern>/ProductServlet</url-pattern>
</servlet-mapping>
```

**Ưu điểm Annotation**:
- Ngắn gọn
- Code và config ở cùng 1 chỗ
- Dễ bảo trì

---

#### **Q: init() và destroy() hoạt động thế nào?**

**A**:

**Lifecycle của Servlet**:
```
1. Server khởi động
   ↓
2. init() được gọi (1 lần duy nhất)
   ↓
3. Servlet sẵn sàng xử lý request
   ↓
4. Mỗi request → doGet() hoặc doPost()
   ↓
5. Server tắt
   ↓
6. destroy() được gọi (1 lần duy nhất)
```

**init()**:
- Gọi 1 lần khi Servlet được tạo
- Dùng để khởi tạo DAO, connection pool, ...
- Không nên làm công việc nặng (chặn request khác)

**destroy()**:
- Gọi 1 lần khi Servlet bị hủy
- Dùng để đóng connection, giải phóng resource
- Không được throw exception

---

#### **Q: doGet() vs doPost()?**

**A**:

| Đặc điểm | GET | POST |
|----------|-----|------|
| **Mục đích** | Lấy dữ liệu (không thay đổi server) | Gửi dữ liệu (thay đổi server) |
| **Dữ liệu** | Trong URL (query string) | Trong request body |
| **Giới hạn** | ~2KB (tùy browser) | Không giới hạn |
| **Bookmark** | Có thể bookmark | Không thể |
| **Cache** | Có thể cache | Không cache |
| **Bảo mật** | Kém (URL hiển thị) | Tốt hơn (body ẩn) |
| **Ví dụ** | Xem danh sách, tìm kiếm | Đăng nhập, thêm/sửa/xóa |

**Trong ProductServlet**:
- **doGet()**: list, detail, add (form), edit (form)
- **doPost()**: add (submit), update (submit), delete (submit)

---

#### **Q: Forward vs Redirect?**

**A**:

**Forward** (`request.getRequestDispatcher(...).forward(...)`):
```
Browser  →  Servlet  →  JSP  →  (same request)  →  Browser
```
- **Server-side**: Server xử lý nội bộ
- **URL không đổi**: Vẫn giữ URL ban đầu
- **Request scope giữ nguyên**: JSP có thể lấy attribute từ Servlet
- **Dùng khi**: Hiển thị dữ liệu, chuyển sang View

**Redirect** (`response.sendRedirect(...)`):
```
Browser  →  Servlet  →  (response: redirect)  →  Browser
                                                    ↓
                                            (new request)  →  Servlet/JSP
```
- **Client-side**: Browser gửi request mới
- **URL thay đổi**: Browser nhảy sang URL mới
- **Request scope mất**: Không giữ attribute
- **Dùng khi**: Sau khi thêm/sửa/xóa (tránh double submit)

**Ví dụ**:
```java
// Forward (hiển thị danh sách)
request.setAttribute("productList", products);
request.getRequestDispatcher("views/product_list.jsp").forward(request, response);
// URL: ProductServlet?action=list (không đổi)

// Redirect (sau khi thêm thành công)
response.sendRedirect("ProductServlet?action=list&message=add_success");
// URL: ProductServlet?action=list&message=add_success (đổi)
```

**Tại sao redirect sau khi thêm/sửa/xóa?**
- **Tránh double submit**: User bấm F5 → Không submit lại form
- **PRG Pattern** (Post-Redirect-Get):
  1. POST: Thêm/sửa/xóa
  2. REDIRECT: Chuyển sang URL mới
  3. GET: Hiển thị danh sách

---

#### **Q: request.getParameter() lấy gì?**

**A**:

**Lấy tham số từ URL (GET) hoặc form (POST)**:

**URL (GET)**:
```
ProductServlet?action=detail&id=5
```
```java
String action = request.getParameter("action");  // "detail"
String id = request.getParameter("id");          // "5"
```

**Form (POST)**:
```html
<form action="ProductServlet" method="post">
    <input type="hidden" name="action" value="add">
    <input type="text" name="name" value="Bánh quy">
    <input type="number" name="price" value="15000">
    <button type="submit">Thêm</button>
</form>
```
```java
String action = request.getParameter("action");  // "add"
String name = request.getParameter("name");      // "Bánh quy"
String price = request.getParameter("price");    // "15000"
```

**Chú ý**:
- **Luôn trả về String** (cần parse sang int, BigDecimal, ...)
- **Trả về null** nếu không tồn tại
- **Cần validate**: Kiểm tra null, empty, format

---

#### **Q: request.setAttribute() làm gì?**

**A**:

**Đưa dữ liệu vào request scope để JSP lấy ra**:

**Servlet**:
```java
List<Product> products = dao.getAll();
request.setAttribute("productList", products);  // Key: "productList", Value: products
request.getRequestDispatcher("views/product_list.jsp").forward(request, response);
```

**JSP**:
```jsp
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>

<!-- Lấy ra bằng EL -->
<c:forEach var="product" items="${productList}">
    <p>${product.name}</p>
</c:forEach>
```

**Scope trong JSP/Servlet**:

| Scope | Phạm vi | Lifetime | Dùng khi |
|-------|---------|----------|----------|
| **Page** | Trong 1 JSP | 1 request | Biến tạm |
| **Request** | 1 request (Servlet → JSP) | 1 request | Truyền dữ liệu từ Servlet sang JSP |
| **Session** | 1 user session | Đến khi logout | Lưu thông tin user đăng nhập |
| **Application** | Toàn ứng dụng | Đến khi server tắt | Config toàn cục |

---

#### **Q: Tại sao cần setCharacterEncoding()?**

**A**:
```java
request.setCharacterEncoding("UTF-8");
response.setCharacterEncoding("UTF-8");
```

**Vấn đề**: Tiếng Việt bị lỗi font (?????, �����).

**Nguyên nhân**: 
- Default encoding của Tomcat thường là ISO-8859-1 (không hỗ trợ Unicode)
- Dữ liệu từ form (POST) gửi lên bị encode sai

**Giải pháp**:
- **Request**: Đọc dữ liệu từ form với UTF-8
- **Response**: Gửi HTML về browser với UTF-8

**Chú ý**:
- **Phải gọi trước** `request.getParameter()`
- **Chỉ áp dụng cho POST** (GET cần config Tomcat server.xml)

---

### **3.5. Xử lý lỗi trong Servlet**

**Try-catch pattern**:
```java
try {
    // Code có thể lỗi
    int id = Integer.parseInt(request.getParameter("id"));
    Product product = dao.getById(id);
    // ...
} catch (NumberFormatException e) {
    // Lỗi parse số
    request.setAttribute("errorMessage", "ID không hợp lệ!");
    request.getRequestDispatcher("views/error.jsp").forward(request, response);
} catch (SQLException e) {
    // Lỗi database
    request.setAttribute("errorMessage", "Lỗi database: " + e.getMessage());
    request.getRequestDispatcher("views/error.jsp").forward(request, response);
}
```

**Trang error.jsp**:
```jsp
<!DOCTYPE html>
<html>
<head><title>Lỗi</title></head>
<body>
    <h1>⚠️ Đã xảy ra lỗi!</h1>
    <p>${errorMessage}</p>
    <a href="ProductServlet?action=list">Về trang chủ</a>
</body>
</html>
```

---

### **3.6. Bài tập tự thực hành**

**Bài 1**: Thêm action "search" trong doGet()
- URL: `ProductServlet?action=search&keyword=bánh`
- Gọi `dao.search(keyword)`
- Forward sang `product_list.jsp`

**Bài 2**: Thêm phân trang (pagination)
- URL: `ProductServlet?action=list&page=2&pageSize=10`
- Tính offset: `(page - 1) * pageSize`
- Viết method `dao.getByPage(int offset, int limit)`

**Bài 3**: Thêm confirm trước khi xóa (JavaScript)
```html
<a href="ProductServlet?action=delete&id=5" 
   onclick="return confirm('Bạn chắc chắn muốn xóa?')">
   Xóa
</a>
```

---

(Tiếp tục Phần B4 - JSP Views trong message kế tiếp...)

<a name="phan-b4"></a>
## B4. View: JSP Pages

### **4.1. Vai trò của JSP**

**JSP (JavaServer Pages)** là **View** trong mô hình MVC, chịu trách nhiệm:

1. **Nhận dữ liệu** từ Servlet (qua request scope)
2. **Render HTML** dựa trên dữ liệu
3. **Hiển thị** giao diện cho user
4. **KHÔNG chứa logic nghiệp vụ** (để trong Servlet/DAO)

---

### **4.2. Cấu trúc thư mục JSP**

```
WebContent/
  └── views/
      ├── product_list.jsp      ← Danh sách sản phẩm
      ├── product_detail.jsp    ← Chi tiết sản phẩm
      ├── product_add.jsp       ← Form thêm sản phẩm
      ├── product_edit.jsp      ← Form sửa sản phẩm
      └── error.jsp             ← Trang lỗi
```

---

### **4.3. Code JSP 1: product_list.jsp**

**Chức năng**: Hiển thị danh sách sản phẩm dạng bảng.

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
<%@ taglib uri="http://java.sun.com/jsp/jstl/fmt" prefix="fmt" %>

<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Danh sách sản phẩm</title>
    <style>
        body { 
            font-family: Arial, sans-serif; 
            margin: 20px;
            background-color: #f5f5f5;
        }
        h1 { 
            color: #333; 
            text-align: center;
        }
        .message {
            padding: 10px;
            margin: 20px 0;
            border-radius: 5px;
            text-align: center;
        }
        .success { 
            background-color: #d4edda; 
            color: #155724; 
        }
        .btn {
            display: inline-block;
            padding: 10px 20px;
            background-color: #28a745;
            color: white;
            text-decoration: none;
            border-radius: 5px;
            margin-bottom: 20px;
        }
        .btn:hover {
            background-color: #218838;
        }
        table {
            width: 100%;
            border-collapse: collapse;
            background-color: white;
            box-shadow: 0 2px 5px rgba(0,0,0,0.1);
        }
        th, td {
            padding: 12px;
            text-align: left;
            border-bottom: 1px solid #ddd;
        }
        th {
            background-color: #007bff;
            color: white;
        }
        tr:hover {
            background-color: #f1f1f1;
        }
        .actions a {
            margin-right: 10px;
            padding: 5px 10px;
            border-radius: 3px;
            text-decoration: none;
            color: white;
        }
        .btn-detail { background-color: #17a2b8; }
        .btn-edit { background-color: #ffc107; }
        .btn-delete { background-color: #dc3545; }
        .product-image {
            width: 80px;
            height: 80px;
            object-fit: cover;
            border-radius: 5px;
        }
        .in-stock {
            color: green;
            font-weight: bold;
        }
        .out-of-stock {
            color: red;
            font-weight: bold;
        }
    </style>
</head>
<body>
    <h1>🛒 DANH SÁCH SẢN PHẨM</h1>
    
    <!-- Hiển thị thông báo (nếu có) -->
    <c:if test="${param.message == 'add_success'}">
        <div class="message success">✅ Thêm sản phẩm thành công!</div>
    </c:if>
    <c:if test="${param.message == 'update_success'}">
        <div class="message success">✅ Cập nhật sản phẩm thành công!</div>
    </c:if>
    <c:if test="${param.message == 'delete_success'}">
        <div class="message success">✅ Xóa sản phẩm thành công!</div>
    </c:if>
    
    <!-- Nút thêm sản phẩm mới -->
    <a href="ProductServlet?action=add" class="btn">➕ Thêm sản phẩm mới</a>
    
    <!-- Hiển thị tổng số sản phẩm -->
    <p>Tổng số sản phẩm: <strong>${totalProducts}</strong></p>
    
    <!-- Bảng danh sách sản phẩm -->
    <table>
        <thead>
            <tr>
                <th>ID</th>
                <th>Ảnh</th>
                <th>Tên sản phẩm</th>
                <th>Giá</th>
                <th>Danh mục</th>
                <th>Số lượng</th>
                <th>Trạng thái</th>
                <th>Thao tác</th>
            </tr>
        </thead>
        <tbody>
            <!-- Duyệt danh sách sản phẩm bằng JSTL -->
            <c:forEach var="product" items="${productList}">
                <tr>
                    <!-- ID -->
                    <td>${product.id}</td>
                    
                    <!-- Ảnh sản phẩm -->
                    <td>
                        <c:choose>
                            <c:when test="${not empty product.image}">
                                <img src="images/products/${product.image}" 
                                     alt="${product.name}" 
                                     class="product-image">
                            </c:when>
                            <c:otherwise>
                                <img src="images/products/default.jpg" 
                                     alt="No image" 
                                     class="product-image">
                            </c:otherwise>
                        </c:choose>
                    </td>
                    
                    <!-- Tên sản phẩm -->
                    <td><strong>${product.name}</strong></td>
                    
                    <!-- Giá (định dạng tiền tệ) -->
                    <td>
                        <fmt:formatNumber value="${product.price}" 
                                          type="number" 
                                          pattern="#,##0"/> đ
                    </td>
                    
                    <!-- Danh mục -->
                    <td>${product.categoryName}</td>
                    
                    <!-- Số lượng tồn kho -->
                    <td>${product.stock}</td>
                    
                    <!-- Trạng thái -->
                    <td>
                        <c:choose>
                            <c:when test="${product.stock > 0}">
                                <span class="in-stock">Còn hàng</span>
                            </c:when>
                            <c:otherwise>
                                <span class="out-of-stock">Hết hàng</span>
                            </c:otherwise>
                        </c:choose>
                    </td>
                    
                    <!-- Các nút thao tác -->
                    <td class="actions">
                        <a href="ProductServlet?action=detail&id=${product.id}" 
                           class="btn-detail">
                           👁️ Xem
                        </a>
                        <a href="ProductServlet?action=edit&id=${product.id}" 
                           class="btn-edit">
                           ✏️ Sửa
                        </a>
                        <a href="ProductServlet?action=delete&id=${product.id}" 
                           class="btn-delete"
                           onclick="return confirm('Bạn chắc chắn muốn xóa sản phẩm này?')">
                           🗑️ Xóa
                        </a>
                    </td>
                </tr>
            </c:forEach>
            
            <!-- Nếu không có sản phẩm nào -->
            <c:if test="${empty productList}">
                <tr>
                    <td colspan="8" style="text-align: center; padding: 40px;">
                        <p style="font-size: 18px; color: #999;">
                            📭 Chưa có sản phẩm nào
                        </p>
                        <a href="ProductServlet?action=add" class="btn">
                            Thêm sản phẩm đầu tiên
                        </a>
                    </td>
                </tr>
            </c:if>
        </tbody>
    </table>
</body>
</html>
```

**Giải thích từng phần**:

**1. Khai báo JSP**:
```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
```
- `contentType="text/html; charset=UTF-8"`: Loại nội dung và mã hóa
- `pageEncoding="UTF-8"`: Encoding của file JSP (hỗ trợ tiếng Việt)

**2. Import JSTL**:
```jsp
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
<%@ taglib uri="http://java.sun.com/jsp/jstl/fmt" prefix="fmt" %>
```
- `core`: Các thẻ cơ bản (forEach, if, choose...)
- `fmt`: Định dạng số, ngày tháng

**3. Hiển thị thông báo**:
```jsp
<c:if test="${param.message == 'add_success'}">
    <div class="message success">✅ Thêm sản phẩm thành công!</div>
</c:if>
```
- `${param.message}`: Lấy parameter `message` từ URL
- Ví dụ: `ProductServlet?action=list&message=add_success`

**4. Vòng lặp forEach**:
```jsp
<c:forEach var="product" items="${productList}">
    <tr>
        <td>${product.id}</td>
        <td>${product.name}</td>
        <!-- ... -->
    </tr>
</c:forEach>
```
- `items="${productList}"`: Lấy List<Product> từ request scope
- `var="product"`: Mỗi vòng lặp, product là 1 object Product
- `${product.id}`: Gọi product.getId()

**5. Định dạng tiền tệ**:
```jsp
<fmt:formatNumber value="${product.price}" 
                  type="number" 
                  pattern="#,##0"/> đ
```
- Input: 15000 (BigDecimal)
- Output: "15,000 đ"

**6. Điều kiện choose**:
```jsp
<c:choose>
    <c:when test="${product.stock > 0}">
        <span class="in-stock">Còn hàng</span>
    </c:when>
    <c:otherwise>
        <span class="out-of-stock">Hết hàng</span>
    </c:otherwise>
</c:choose>
```
- Giống switch-case
- `<c:when>`: Trường hợp điều kiện đúng
- `<c:otherwise>`: Trường hợp mặc định

**7. JavaScript confirm**:
```jsp
<a href="..." onclick="return confirm('Bạn chắc chắn?')">Xóa</a>
```
- Hiển thị hộp thoại xác nhận
- `return false`: Hủy link, không chuyển trang
- `return true`: Tiếp tục chuyển trang

---

### **4.4. Code JSP 2: product_add.jsp**

**Chức năng**: Form thêm sản phẩm mới.

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>

<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Thêm sản phẩm mới</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            max-width: 600px;
            margin: 50px auto;
            padding: 20px;
            background-color: #f5f5f5;
        }
        h1 {
            text-align: center;
            color: #333;
        }
        form {
            background-color: white;
            padding: 30px;
            border-radius: 10px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
        }
        .form-group {
            margin-bottom: 20px;
        }
        label {
            display: block;
            margin-bottom: 5px;
            font-weight: bold;
            color: #555;
        }
        input[type="text"],
        input[type="number"],
        textarea,
        select {
            width: 100%;
            padding: 10px;
            border: 1px solid #ddd;
            border-radius: 5px;
            font-size: 14px;
            box-sizing: border-box;
        }
        textarea {
            resize: vertical;
            min-height: 100px;
        }
        .required {
            color: red;
        }
        .buttons {
            display: flex;
            gap: 10px;
            margin-top: 30px;
        }
        button, .btn-cancel {
            flex: 1;
            padding: 12px;
            border: none;
            border-radius: 5px;
            font-size: 16px;
            cursor: pointer;
            text-decoration: none;
            text-align: center;
            display: inline-block;
        }
        button[type="submit"] {
            background-color: #28a745;
            color: white;
        }
        button[type="submit"]:hover {
            background-color: #218838;
        }
        .btn-cancel {
            background-color: #6c757d;
            color: white;
        }
        .btn-cancel:hover {
            background-color: #5a6268;
        }
    </style>
</head>
<body>
    <h1>➕ THÊM SẢN PHẨM MỚI</h1>
    
    <form action="ProductServlet" method="post">
        <!-- Hidden field: action -->
        <input type="hidden" name="action" value="add">
        
        <!-- Tên sản phẩm -->
        <div class="form-group">
            <label for="name">
                Tên sản phẩm <span class="required">*</span>
            </label>
            <input type="text" 
                   id="name" 
                   name="name" 
                   placeholder="Ví dụ: Bánh quy Oreo" 
                   required>
        </div>
        
        <!-- Giá -->
        <div class="form-group">
            <label for="price">
                Giá (VNĐ) <span class="required">*</span>
            </label>
            <input type="number" 
                   id="price" 
                   name="price" 
                   placeholder="Ví dụ: 15000" 
                   min="1"
                   step="1000"
                   required>
        </div>
        
        <!-- Danh mục -->
        <div class="form-group">
            <label for="categoryId">
                Danh mục <span class="required">*</span>
            </label>
            <select id="categoryId" name="categoryId" required>
                <option value="">-- Chọn danh mục --</option>
                <!-- Giả sử có categories từ Servlet -->
                <option value="1">Bánh kẹo</option>
                <option value="2">Snack</option>
                <option value="3">Nước giải khát</option>
                <option value="4">Mì gói</option>
            </select>
        </div>
        
        <!-- Mô tả -->
        <div class="form-group">
            <label for="description">Mô tả</label>
            <textarea id="description" 
                      name="description" 
                      placeholder="Nhập mô tả chi tiết về sản phẩm..."></textarea>
        </div>
        
        <!-- Số lượng -->
        <div class="form-group">
            <label for="stock">
                Số lượng <span class="required">*</span>
            </label>
            <input type="number" 
                   id="stock" 
                   name="stock" 
                   placeholder="Ví dụ: 100" 
                   min="0"
                   required>
        </div>
        
        <!-- Hình ảnh -->
        <div class="form-group">
            <label for="image">Tên file ảnh</label>
            <input type="text" 
                   id="image" 
                   name="image" 
                   placeholder="Ví dụ: oreo.jpg">
            <small style="color: #666; display: block; margin-top: 5px;">
                📷 Ảnh phải được upload vào thư mục <code>images/products/</code>
            </small>
        </div>
        
        <!-- Nút submit -->
        <div class="buttons">
            <button type="submit">✅ Thêm sản phẩm</button>
            <a href="ProductServlet?action=list" class="btn-cancel">❌ Hủy</a>
        </div>
    </form>
</body>
</html>
```

**Giải thích**:

**1. Form POST**:
```jsp
<form action="ProductServlet" method="post">
```
- `action="ProductServlet"`: Gửi đến servlet
- `method="post"`: Dùng POST (thay đổi dữ liệu)

**2. Hidden field**:
```jsp
<input type="hidden" name="action" value="add">
```
- Truyền action không hiển thị trên giao diện
- Servlet nhận: `request.getParameter("action")` = "add"

**3. Validation HTML5**:
```jsp
<input type="text" name="name" required>
<input type="number" name="price" min="1" step="1000" required>
```
- `required`: Bắt buộc nhập
- `min="1"`: Giá trị tối thiểu
- `step="1000"`: Bước nhảy (1000, 2000, 3000...)

**4. Textarea**:
```jsp
<textarea name="description"></textarea>
```
- Nhập văn bản nhiều dòng
- Không có thuộc tính `value`, nội dung nằm giữa `<textarea>...</textarea>`

**5. Select dropdown**:
```jsp
<select name="categoryId">
    <option value="1">Bánh kẹo</option>
    <option value="2">Snack</option>
</select>
```
- Chọn 1 trong nhiều option
- `value`: Giá trị gửi lên server

---

### **4.5. Code JSP 3: product_edit.jsp**

**Chức năng**: Form sửa sản phẩm (giống add nhưng có dữ liệu sẵn).

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>

<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Sửa sản phẩm</title>
    <style>
        /* Style giống product_add.jsp */
        body {
            font-family: Arial, sans-serif;
            max-width: 600px;
            margin: 50px auto;
            padding: 20px;
            background-color: #f5f5f5;
        }
        /* ... (copy style từ add) ... */
    </style>
</head>
<body>
    <h1>✏️ SỬA SẢN PHẨM</h1>
    
    <form action="ProductServlet" method="post">
        <!-- Hidden fields: action và id -->
        <input type="hidden" name="action" value="update">
        <input type="hidden" name="id" value="${product.id}">
        
        <!-- Tên sản phẩm -->
        <div class="form-group">
            <label for="name">Tên sản phẩm <span class="required">*</span></label>
            <input type="text" 
                   id="name" 
                   name="name" 
                   value="${product.name}"
                   required>
        </div>
        
        <!-- Giá -->
        <div class="form-group">
            <label for="price">Giá (VNĐ) <span class="required">*</span></label>
            <input type="number" 
                   id="price" 
                   name="price" 
                   value="${product.price}"
                   min="1"
                   step="1000"
                   required>
        </div>
        
        <!-- Danh mục -->
        <div class="form-group">
            <label for="categoryId">Danh mục <span class="required">*</span></label>
            <select id="categoryId" name="categoryId" required>
                <option value="1" ${product.categoryId == 1 ? 'selected' : ''}>Bánh kẹo</option>
                <option value="2" ${product.categoryId == 2 ? 'selected' : ''}>Snack</option>
                <option value="3" ${product.categoryId == 3 ? 'selected' : ''}>Nước giải khát</option>
                <option value="4" ${product.categoryId == 4 ? 'selected' : ''}>Mì gói</option>
            </select>
        </div>
        
        <!-- Mô tả -->
        <div class="form-group">
            <label for="description">Mô tả</label>
            <textarea id="description" name="description">${product.description}</textarea>
        </div>
        
        <!-- Số lượng -->
        <div class="form-group">
            <label for="stock">Số lượng <span class="required">*</span></label>
            <input type="number" 
                   id="stock" 
                   name="stock" 
                   value="${product.stock}"
                   min="0"
                   required>
        </div>
        
        <!-- Hình ảnh -->
        <div class="form-group">
            <label for="image">Tên file ảnh</label>
            <input type="text" 
                   id="image" 
                   name="image" 
                   value="${product.image}">
        </div>
        
        <!-- Nút submit -->
        <div class="buttons">
            <button type="submit">💾 Lưu thay đổi</button>
            <a href="ProductServlet?action=list" class="btn-cancel">❌ Hủy</a>
        </div>
    </form>
</body>
</html>
```

**Điểm khác biệt so với add.jsp**:

**1. Action là "update"**:
```jsp
<input type="hidden" name="action" value="update">
```

**2. Có thêm id**:
```jsp
<input type="hidden" name="id" value="${product.id}">
```
- Servlet cần id để biết sản phẩm nào cần update

**3. Hiển thị sẵn dữ liệu hiện tại**:
```jsp
<input type="text" name="name" value="${product.name}">
```
- `value="${product.name}"`: Hiển thị tên cũ

**4. Select với selected**:
```jsp
<option value="1" ${product.categoryId == 1 ? 'selected' : ''}>Bánh kẹo</option>
```
- **EL ternary operator**: `condition ? value_if_true : value_if_false`
- Nếu `product.categoryId == 1` → Thêm attribute `selected`

**5. Textarea content**:
```jsp
<textarea name="description">${product.description}</textarea>
```
- Nội dung giữa thẻ mở và đóng

---

### **4.6. Tóm tắt JSTL Tags**

| Tag | Mục đích | Ví dụ |
|-----|----------|-------|
| `<c:if>` | Điều kiện đơn | `<c:if test="${user != null}">Xin chào</c:if>` |
| `<c:choose>` | Nhiều điều kiện | `<c:when test="...">, <c:otherwise>` |
| `<c:forEach>` | Vòng lặp | `<c:forEach var="p" items="${list}">` |
| `<c:out>` | Xuất dữ liệu an toàn | `<c:out value="${text}" />` |
| `<fmt:formatNumber>` | Định dạng số | `<fmt:formatNumber value="${price}" pattern="#,##0"/>` |
| `<fmt:formatDate>` | Định dạng ngày | `<fmt:formatDate value="${date}" pattern="dd/MM/yyyy"/>` |

---


---

# PHẦN C: LUỒNG XỬ LÝ HOÀN CHỈNH

<a name="phan-c1"></a>
## C1. Kịch bản 1: Xem danh sách sản phẩm

### **User Story**:
> "Là người quản lý, tôi muốn xem danh sách tất cả sản phẩm để nắm được hàng hóa hiện có."

### **Luồng xử lý chi tiết**:

```
┌─────────────────────────────────────────────────────────────────┐
│ BƯỚC 1: User mở trình duyệt                                     │
└─────────────────────────────────────────────────────────────────┘
                              ↓
          Nhập URL: http://localhost:8080/SnackShop/ProductServlet
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│ BƯỚC 2: Browser gửi HTTP GET request                           │
│ GET /SnackShop/ProductServlet HTTP/1.1                         │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│ BƯỚC 3: Tomcat nhận request, tìm Servlet phù hợp               │
│ - Kiểm tra @WebServlet("/ProductServlet")                      │
│ - Khởi tạo Servlet nếu chưa tạo (gọi init())                   │
│ - Gọi method doGet()                                            │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│ BƯỚC 4: ProductServlet.doGet() xử lý                           │
│                                                                 │
│ String action = request.getParameter("action");                │
│ // action = null (không có parameter)                          │
│                                                                 │
│ if (action == null) {                                          │
│     action = "list";  // Mặc định                              │
│ }                                                              │
│                                                                 │
│ switch (action) {                                              │
│     case "list":                                               │
│         showProductList(request, response);                    │
│         break;                                                 │
│ }                                                              │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│ BƯỚC 5: showProductList() thực thi                             │
│                                                                 │
│ // Gọi DAO lấy dữ liệu                                         │
│ List<Product> productList = productDAO.getAll();               │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│ BƯỚC 6: ProductDAO.getAll() truy vấn database                  │
│                                                                 │
│ String sql = "SELECT p.id, p.name, p.price, ... " +           │
│              "FROM products p " +                              │
│              "LEFT JOIN categories c ON p.category_id = c.id"; │
│                                                                 │
│ PreparedStatement ps = connection.prepareStatement(sql);       │
│ ResultSet rs = ps.executeQuery();                             │
│                                                                 │
│ List<Product> products = new ArrayList<>();                    │
│ while (rs.next()) {                                            │
│     Product p = new Product(                                   │
│         rs.getInt("id"),                                       │
│         rs.getString("name"),                                  │
│         rs.getBigDecimal("price"),                             │
│         ...                                                    │
│     );                                                         │
│     products.add(p);                                           │
│ }                                                              │
│                                                                 │
│ return products;  // Trả về List<Product>                     │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│ BƯỚC 7: Servlet nhận List<Product>, đưa vào request scope     │
│                                                                 │
│ request.setAttribute("productList", productList);              │
│ request.setAttribute("totalProducts", productList.size());     │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│ BƯỚC 8: Forward sang JSP                                       │
│                                                                 │
│ request.getRequestDispatcher("views/product_list.jsp")        │
│        .forward(request, response);                            │
│                                                                 │
│ - Server xử lý nội bộ (không qua browser)                     │
│ - URL không thay đổi                                           │
│ - Request scope giữ nguyên                                     │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│ BƯỚC 9: product_list.jsp render HTML                           │
│                                                                 │
│ <c:forEach var="product" items="${productList}">              │
│     <tr>                                                       │
│         <td>${product.id}</td>                                 │
│         <td>${product.name}</td>                               │
│         <td>                                                   │
│             <fmt:formatNumber value="${product.price}" />     │
│         </td>                                                  │
│     </tr>                                                      │
│ </c:forEach>                                                   │
│                                                                 │
│ → Duyệt qua từng Product, tạo HTML cho mỗi dòng              │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│ BƯỚC 10: Gửi HTML response về browser                          │
│                                                                 │
│ HTTP/1.1 200 OK                                                │
│ Content-Type: text/html; charset=UTF-8                         │
│                                                                 │
│ <!DOCTYPE html>                                                │
│ <html>                                                         │
│ <body>                                                         │
│     <table>                                                    │
│         <tr><td>1</td><td>Bánh quy</td><td>15,000 đ</td></tr> │
│         <tr><td>2</td><td>Snack</td><td>10,000 đ</td></tr>    │
│         ...                                                    │
│     </table>                                                   │
│ </body>                                                        │
│ </html>                                                        │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│ BƯỚC 11: Browser nhận HTML, hiển thị trang web                │
│                                                                 │
│ - Parse HTML                                                   │
│ - Render CSS                                                   │
│ - Hiển thị bảng sản phẩm                                       │
└─────────────────────────────────────────────────────────────────┘
```

### **Điểm cần nhớ để giải thích cho thầy**:

**Q: "Em giải thích luồng xử lý khi user xem danh sách sản phẩm?"**

**A**: 
```
1. User truy cập URL ProductServlet
2. Tomcat nhận request GET, gọi doGet() trong Servlet
3. Servlet lấy action = "list" (hoặc mặc định)
4. Servlet gọi ProductDAO.getAll() để lấy dữ liệu từ database
5. DAO thực thi SQL JOIN giữa products và categories
6. DAO chuyển ResultSet thành List<Product>
7. Servlet đưa List vào request scope bằng setAttribute
8. Servlet forward sang product_list.jsp
9. JSP dùng JSTL <c:forEach> duyệt List, render HTML
10. HTML được gửi về browser, user thấy bảng sản phẩm
```

**Q: "Tại sao dùng forward chứ không redirect?"**

**A**:
```
- Forward: Server xử lý nội bộ, URL không đổi, giữ request scope
- Dùng forward vì cần truyền dữ liệu (productList) từ Servlet sang JSP
- Nếu redirect: request scope mất, JSP không nhận được dữ liệu
```

---

<a name="phan-c2"></a>
## C2. Kịch bản 2: Thêm sản phẩm mới

### **User Story**:
> "Là người quản lý, tôi muốn thêm sản phẩm mới vào hệ thống để mở rộng danh mục hàng hóa."

### **Luồng xử lý (2 bước)**:

#### **Bước A: Hiển thị form thêm (GET request)**

```
User click "Thêm sản phẩm mới"
  ↓
GET /ProductServlet?action=add
  ↓
Servlet: doGet() → showAddForm()
  ↓
Forward: product_add.jsp
  ↓
Browser hiển thị form rỗng
```

#### **Bước B: Submit form (POST request)**

```
┌─────────────────────────────────────────────────────────────────┐
│ BƯỚC 1: User điền form và bấm "Thêm sản phẩm"                  │
│                                                                 │
│ - Tên: Bánh quy Oreo                                           │
│ - Giá: 15000                                                   │
│ - Danh mục: 1 (Bánh kẹo)                                       │
│ - Mô tả: Bánh ngọt thơm ngon                                   │
│ - Số lượng: 100                                                │
│ - Ảnh: oreo.jpg                                                │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│ BƯỚC 2: Browser gửi POST request                               │
│                                                                 │
│ POST /SnackShop/ProductServlet HTTP/1.1                        │
│ Content-Type: application/x-www-form-urlencoded                │
│                                                                 │
│ action=add&name=Bánh+quy+Oreo&price=15000&categoryId=1&...     │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│ BƯỚC 3: Servlet.doPost() xử lý                                 │
│                                                                 │
│ // Set encoding để xử lý tiếng Việt                           │
│ request.setCharacterEncoding("UTF-8");                         │
│                                                                 │
│ String action = request.getParameter("action"); // "add"       │
│                                                                 │
│ switch (action) {                                              │
│     case "add":                                                │
│         addProduct(request, response);                         │
│         break;                                                 │
│ }                                                              │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│ BƯỚC 4: addProduct() lấy dữ liệu từ form                       │
│                                                                 │
│ String name = request.getParameter("name");                    │
│ String priceStr = request.getParameter("price");               │
│ String categoryIdStr = request.getParameter("categoryId");     │
│ // ... lấy các field khác ...                                 │
│                                                                 │
│ // Chuyển đổi kiểu dữ liệu                                     │
│ BigDecimal price = new BigDecimal(priceStr);  // String → BigDecimal│
│ int categoryId = Integer.parseInt(categoryIdStr);  // String → int │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│ BƯỚC 5: Validate dữ liệu                                       │
│                                                                 │
│ if (name == null || name.trim().isEmpty()) {                  │
│     throw new IllegalArgumentException("Tên không được rỗng!"); │
│ }                                                              │
│                                                                 │
│ if (price.compareTo(BigDecimal.ZERO) <= 0) {                  │
│     throw new IllegalArgumentException("Giá phải > 0!");      │
│ }                                                              │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│ BƯỚC 6: Tạo object Product                                     │
│                                                                 │
│ Product newProduct = new Product(                              │
│     name,           // "Bánh quy Oreo"                         │
│     price,          // 15000                                   │
│     description,    // "Bánh ngọt thơm ngon"                   │
│     stock,          // 100                                     │
│     image,          // "oreo.jpg"                              │
│     categoryId      // 1                                       │
│ );                                                             │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│ BƯỚC 7: Gọi DAO insert vào database                            │
│                                                                 │
│ boolean success = productDAO.insert(newProduct);               │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│ BƯỚC 8: ProductDAO.insert() thực thi SQL                       │
│                                                                 │
│ String sql = "INSERT INTO products " +                         │
│              "(name, price, description, stock, image, category_id) " + │
│              "VALUES (?, ?, ?, ?, ?, ?)";                      │
│                                                                 │
│ PreparedStatement ps = connection.prepareStatement(sql);       │
│                                                                 │
│ // Gán giá trị cho các placeholder                            │
│ ps.setString(1, product.getName());           // ?1 = "Bánh quy Oreo" │
│ ps.setBigDecimal(2, product.getPrice());      // ?2 = 15000    │
│ ps.setString(3, product.getDescription());    // ?3 = "Bánh ngọt..." │
│ ps.setInt(4, product.getStock());             // ?4 = 100      │
│ ps.setString(5, product.getImage());          // ?5 = "oreo.jpg" │
│ ps.setInt(6, product.getCategoryId());        // ?6 = 1        │
│                                                                 │
│ int rowsAffected = ps.executeUpdate();  // Thực thi INSERT    │
│                                                                 │
│ return (rowsAffected > 0);  // true nếu thành công           │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│ BƯỚC 9: Servlet xử lý kết quả                                  │
│                                                                 │
│ if (success) {                                                 │
│     // Thành công → REDIRECT về danh sách                     │
│     response.sendRedirect(                                     │
│         "ProductServlet?action=list&message=add_success"       │
│     );                                                         │
│ } else {                                                       │
│     // Thất bại → Forward sang error.jsp                      │
│     request.setAttribute("errorMessage", "Thêm thất bại!");   │
│     request.getRequestDispatcher("views/error.jsp")           │
│            .forward(request, response);                        │
│ }                                                              │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│ BƯỚC 10: Redirect - Browser nhận response                      │
│                                                                 │
│ HTTP/1.1 302 Found                                             │
│ Location: ProductServlet?action=list&message=add_success       │
│                                                                 │
│ → Browser tự động gửi request mới đến URL trong Location      │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│ BƯỚC 11: Browser gửi GET request mới                           │
│                                                                 │
│ GET /ProductServlet?action=list&message=add_success            │
│                                                                 │
│ → Quay lại luồng "Xem danh sách sản phẩm"                     │
│ → JSP hiển thị thông báo thành công                            │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│ BƯỚC 12: product_list.jsp kiểm tra message                     │
│                                                                 │
│ <c:if test="${param.message == 'add_success'}">               │
│     <div class="message success">                              │
│         ✅ Thêm sản phẩm thành công!                          │
│     </div>                                                     │
│ </c:if>                                                        │
└─────────────────────────────────────────────────────────────────┘
```

### **Điểm cần nhớ**:

**Q: "Tại sao POST thêm sản phẩm lại redirect về list, không forward?"**

**A**:
```
- PRG Pattern (Post-Redirect-Get):
  1. POST: Thêm sản phẩm vào DB
  2. REDIRECT: Chuyển hướng về danh sách (URL mới)
  3. GET: Hiển thị danh sách

- Lý do:
  + Tránh double submit: User bấm F5 không submit lại form
  + URL sạch: /ProductServlet?action=list thay vì /ProductServlet (POST)
  + Bookmark: User có thể bookmark trang danh sách
```

**Q: "executeUpdate() trả về gì?"**

**A**:
```
- executeUpdate(): Dùng cho INSERT, UPDATE, DELETE
- Trả về: int (số dòng bị ảnh hưởng)
  + INSERT 1 dòng → trả về 1
  + UPDATE 3 dòng → trả về 3
  + DELETE 0 dòng (không tìm thấy) → trả về 0
  
- Khác với executeQuery():
  + executeQuery(): Dùng cho SELECT
  + Trả về: ResultSet (dữ liệu)
```

---

---

# PHẦN D: CHUẨN BỊ BÁO CÁO CHO THẦY

<a name="phan-d1"></a>
## D1. Câu hỏi thầy thường hỏi & Cách trả lời

### **NHÓM 1: KIẾN TRÚC MVC**

#### **Q1: "Em giải thích mô hình MVC trong phần của em?"**

**Trả lời**:
```
Thưa thầy, em áp dụng mô hình MVC trong module Quản lý Sản phẩm như sau:

1. MODEL (Dữ liệu):
   - Product.java: Lớp đại diện cho sản phẩm, có 7 thuộc tính (id, name, price, ...)
   - ProductDAO.java: Lớp truy cập database, có 8 methods (getAll, getById, insert, ...)

2. VIEW (Giao diện):
   - product_list.jsp: Hiển thị danh sách sản phẩm
   - product_add.jsp: Form thêm sản phẩm
   - product_edit.jsp: Form sửa sản phẩm
   
3. CONTROLLER (Điều khiển):
   - ProductServlet.java: Nhận request, gọi DAO, chọn View
   - doGet(): Xử lý hiển thị (list, detail, add form, edit form)
   - doPost(): Xử lý submit (add, update, delete)

Luồng hoạt động:
User → Servlet (Controller) → DAO (Model) → Database
                ↓
           Forward sang JSP (View) → HTML → User
```

---

#### **Q2: "Tại sao tách DAO ra khỏi Servlet?"**

**Trả lời**:
```
Thưa thầy, em tách DAO để:

1. Tách biệt trách nhiệm (Separation of Concerns):
   - Servlet: Xử lý logic nghiệp vụ, điều hướng
   - DAO: Chỉ lo truy vấn database
   
2. Dễ bảo trì:
   - Thay đổi SQL chỉ cần sửa trong DAO
   - Servlet không cần biết chi tiết SQL

3. Tái sử dụng:
   - DAO có thể dùng cho nhiều Servlet khác nhau
   - Ví dụ: ProductDAO được dùng cả trong AdminServlet và UserServlet

4. Dễ test:
   - Test riêng từng lớp
   - Mock DAO để test Servlet mà không cần DB thật
```

---

### **NHÓM 2: DATABASE & JDBC**

#### **Q3: "Tại sao dùng PreparedStatement thay vì Statement?"**

**Trả lời**:
```
Thưa thầy, em dùng PreparedStatement vì:

1. BẢO MẬT - Chống SQL Injection:
   Statement (KHÔNG AN TOÀN):
   String sql = "SELECT * FROM products WHERE name = '" + userInput + "'";
   → Nếu user nhập: ' OR '1'='1
   → SQL thành: SELECT * FROM products WHERE name = '' OR '1'='1'
   → Lấy hết dữ liệu!
   
   PreparedStatement (AN TOÀN):
   String sql = "SELECT * FROM products WHERE name = ?";
   ps.setString(1, userInput);  // Tự động escape ký tự đặc biệt
   
2. HIỆU NĂNG:
   - PreparedStatement: SQL được compile trước, tái sử dụng
   - Statement: Compile lại mỗi lần thực thi
   
3. DỄ ĐỌC:
   - Tách biệt SQL và tham số
   - Code rõ ràng hơn
```

---

#### **Q4: "executeQuery() khác executeUpdate() như thế nào?"**

**Trả lời**:
```
Thưa thầy, sự khác biệt:

┌───────────────┬───────────────────┬──────────────────┐
│               │ executeQuery()    │ executeUpdate()  │
├───────────────┼───────────────────┼──────────────────┤
│ Dùng cho      │ SELECT            │ INSERT, UPDATE,  │
│               │                   │ DELETE           │
├───────────────┼───────────────────┼──────────────────┤
│ Trả về        │ ResultSet         │ int (số dòng bị  │
│               │ (dữ liệu)         │ ảnh hưởng)       │
├───────────────┼───────────────────┼──────────────────┤
│ Ví dụ         │ rs = ps.execute   │ int rows = ps.   │
│               │ Query();          │ executeUpdate(); │
└───────────────┴───────────────────┴──────────────────┘

Trong code của em:
- getAll(), getById(): dùng executeQuery() (SELECT)
- insert(), update(), delete(): dùng executeUpdate()
```

---

### **NHÓM 3: SERVLET**

#### **Q5: "Forward khác Redirect thế nào?"**

**Trả lời**:
```
Thưa thầy, sự khác biệt:

FORWARD (request.getRequestDispatcher(...).forward(...)):
- Server xử lý nội bộ
- URL KHÔNG thay đổi
- Request scope GIỮ NGUYÊN
- Dùng khi: Truyền dữ liệu từ Servlet sang JSP

Ví dụ trong code em:
request.setAttribute("productList", products);
request.getRequestDispatcher("views/product_list.jsp").forward(...);
→ JSP nhận được products từ request scope

REDIRECT (response.sendRedirect(...)):
- Browser gửi request MỚI
- URL THAY ĐỔI
- Request scope MẤT
- Dùng khi: Sau khi thêm/sửa/xóa (tránh double submit)

Ví dụ trong code em:
response.sendRedirect("ProductServlet?action=list&message=add_success");
→ Browser nhảy sang URL mới, F5 không submit lại form
```

---

#### **Q6: "doGet() và doPost() khác nhau gì?"**

**Trả lời**:
```
Thưa thầy, sự khác biệt:

doGet():
- Xử lý GET request (lấy dữ liệu)
- Không thay đổi server
- Dữ liệu trong URL (query string)
- Có thể bookmark, cache

Trong module em:
- action=list: Xem danh sách
- action=detail: Xem chi tiết
- action=add: Hiển thị form thêm
- action=edit: Hiển thị form sửa

doPost():
- Xử lý POST request (gửi dữ liệu)
- Thay đổi dữ liệu trên server
- Dữ liệu trong request body
- Không cache

Trong module em:
- action=add: Thực hiện thêm sản phẩm
- action=update: Thực hiện sửa sản phẩm
- action=delete: Thực hiện xóa sản phẩm
```

---

### **NHÓM 4: JSP & JSTL**

#### **Q7: "Tại sao dùng JSTL thay vì scriptlet Java?"**

**Trả lời**:
```
Thưa thầy, em dùng JSTL vì:

1. CODE DỄ ĐỌC:
   Scriptlet (KHÔNGTỐT):
   <%
       List<Product> products = (List<Product>) request.getAttribute("products");
       for (Product p : products) {
           out.println("<tr><td>" + p.getName() + "</td></tr>");
       }
   %>
   
   JSTL (TỐT):
   <c:forEach var="p" items="${products}">
       <tr><td>${p.name}</td></tr>
   </c:forEach>
   
2. TÁCH BIỆT LOGIC VÀ HIỂN THỊ:
   - JSP chỉ lo render HTML
   - Logic xử lý để trong Servlet

3. BẢO MẬT:
   - <c:out> tự động escape HTML
   - Chống XSS (Cross-Site Scripting)
```

---

#### **Q8: "${product.name} hoạt động như thế nào?"**

**Trả lời**:
```
Thưa thầy, đây là Expression Language (EL).

${product.name} tương đương:
<%= product.getName() %>

Cách hoạt động:
1. JSP Engine tìm object "product" trong các scope (page, request, session, application)
2. Gọi method getName() của object
3. Hiển thị giá trị trả về

Quy tắc:
- ${product.name} → gọi getName()
- ${product.price} → gọi getPrice()
- ${product.categoryId} → gọi getCategoryId()

Nếu không có getter → Lỗi!
```

---

<a name="phan-d2"></a>
## D2. Kịch bản Demo cho thầy

### **CHUẨN BỊ TRƯỚC KHI DEMO**:

1. ✅ Khởi động MySQL, import database
2. ✅ Khởi động Tomcat
3. ✅ Mở Eclipse, clean + rebuild project
4. ✅ Mở browser, test tất cả chức năng
5. ✅ Chuẩn bị sẵn data mẫu (ảnh, thông tin sản phẩm)

---

### **BƯỚC 1: GIỚI THIỆU MODULE**

**Nói**: 
```
"Thưa thầy, em xin trình bày module Quản lý Sản phẩm.
Module này bao gồm 4 chức năng chính:
1. Xem danh sách sản phẩm
2. Xem chi tiết sản phẩm
3. Thêm sản phẩm mới
4. Sửa/Xóa sản phẩm
```

---

### **BƯỚC 2: DEMO CHỨC NĂNG**

#### **Demo 1: Xem danh sách sản phẩm**

**Hành động**: Truy cập `http://localhost:8080/SnackShop/ProductServlet`

**Nói**:
```
"Em vào URL này, browser gửi GET request đến ProductServlet.
Servlet gọi ProductDAO.getAll() lấy dữ liệu từ database.
Sau đó forward sang product_list.jsp để hiển thị."
```

**Chỉ vào màn hình**:
```
"Thầy thấy đây là bảng danh sách, có 7 cột:
- ID, Ảnh, Tên, Giá, Danh mục, Số lượng, Trạng thái
- Mỗi dòng có 3 nút: Xem, Sửa, Xóa"
```

---

#### **Demo 2: Thêm sản phẩm mới**

**Hành động**: Click "Thêm sản phẩm mới"

**Nói**:
```
"Em click nút này, gửi GET request với action=add.
Servlet forward sang product_add.jsp, hiển thị form rỗng."
```

**Điền form**:
```
- Tên: Snack Oishi
- Giá: 12000
- Danh mục: Snack
- Mô tả: Snack khoai tây vị phô mai
- Số lượng: 50
- Ảnh: oishi.jpg
```

**Nói khi bấm "Thêm"**:
```
"Em bấm Thêm, browser gửi POST request với tất cả dữ liệu này.
Servlet gọi ProductDAO.insert() thực thi SQL INSERT.
Sau khi thành công, redirect về danh sách với message=add_success."
```

**Chỉ vào thông báo xanh**:
```
"Thầy thấy đây, hệ thống hiển thị thông báo 'Thêm thành công'.
Sản phẩm mới đã xuất hiện trong bảng."
```

---

### **BƯỚC 3: GIẢI THÍCH CODE**

**Mở Eclipse, chỉ vào ProductServlet.java**:

```
"Thầy thấy đây là ProductServlet, lớp Controller trong MVC.
Em có method doGet() xử lý hiển thị, doPost() xử lý submit form.
```

**Chỉ vào đoạn code**:
```java
protected void doGet(...) {
    String action = request.getParameter("action");
    switch (action) {
        case "list":
            showProductList(request, response);
            break;
    }
}
```

**Nói**:
```
"Em lấy parameter 'action' từ URL, dùng switch-case để phân luồng.
Ví dụ action=list gọi method showProductList()."
```

---

**Chỉ vào ProductDAO.java**:
```
"Đây là lớp DAO, chịu trách nhiệm truy vấn database.
Em có 8 methods: getAll, getById, insert, update, delete, ...
```

**Chỉ vào method getAll()**:
```java
public List<Product> getAll() {
    String sql = "SELECT p.id, p.name, ... FROM products p " +
                 "LEFT JOIN categories c ON p.category_id = c.id";
    PreparedStatement ps = connection.prepareStatement(sql);
    ResultSet rs = ps.executeQuery();
    
    while (rs.next()) {
        Product p = new Product(...);
        products.add(p);
    }
    return products;
}
```

**Nói**:
```
"Em dùng LEFT JOIN để lấy cả tên danh mục từ bảng categories.
PreparedStatement để bảo mật, chống SQL Injection.
ResultSet chứa dữ liệu, em duyệt từng dòng tạo object Product."
```

---

### **BƯỚC 4: TRẢ LỜI CÂU HỎI**

**Thầy hỏi**: "Tại sao em dùng forward ở đây?"

**Trả lời**:
```
"Thưa thầy, em dùng forward vì cần truyền dữ liệu từ Servlet sang JSP.
Forward giữ nguyên request scope, JSP mới lấy được productList.
Nếu em dùng redirect thì request scope mất, JSP không nhận được dữ liệu."
```

---

**Thầy hỏi**: "PreparedStatement có lợi gì?"

**Trả lời**:
```
"Thưa thầy, PreparedStatement có 3 lợi ích:
1. Bảo mật: Chống SQL Injection bằng cách escape ký tự đặc biệt
2. Hiệu năng: SQL được compile trước, tái sử dụng nhanh hơn
3. Dễ đọc: Tách biệt SQL và tham số, code rõ ràng

Em có demo SQL Injection nếu thầy muốn xem."
```

---

### **BƯỚC 5: KẾT THÚC**

**Nói**:
```
"Em xin tóm tắt:
- Module Quản lý Sản phẩm áp dụng MVC pattern
- Model: Product.java + ProductDAO.java
- View: 4 file JSP (list, detail, add, edit)
- Controller: ProductServlet.java

- Em dùng PreparedStatement để bảo mật
- Forward để truyền dữ liệu, Redirect để tránh double submit
- JSTL để render HTML dễ đọc

Em xin kết thúc phần trình bày. Cảm ơn thầy!"
```

---

## 🎓 LỜI KHUYÊN CUỐI CÙNG

### **TRƯỚC KHI BÁO CÁO**:
1. ✅ Đọc lại tài liệu này 2-3 lần
2. ✅ Chạy thử code, test tất cả chức năng
3. ✅ Ghi nhớ luồng xử lý từng chức năng
4. ✅ Chuẩn bị trả lời các câu hỏi phổ biến
5. ✅ Demo thử trước 1-2 lần
6. ✅ Backup code và database

### **TRONG KHI BÁO CÁO**:
1. 🎤 Nói to, rõ ràng, tự tin
2. 👀 Nhìn thầy khi giải thích
3. 🖱️ Thao tác chậm rãi, chỉ rõ từng bước
4. 📊 Vẽ diagram nếu cần (MVC, luồng xử lý)
5. 💬 Dùng thuật ngữ chính xác (Servlet, DAO, forward, redirect...)
6. ⏸️ Dừng lại sau mỗi phần để thầy hỏi

### **KHI THẦY HỎI**:
1. 🤔 Suy nghĩ 2-3 giây trước khi trả lời
2. 📝 Trả lời có cấu trúc (1, 2, 3...)
3. 💻 Demo code nếu cần minh họa
4. 🙏 Không biết thì thành thật "Em chưa tìm hiểu kỹ phần này, em sẽ học thêm"
5. ✅ Xác nhận lại câu hỏi nếu không chắc

---

## 📚 TÀI LIỆU THAM KHẢO THÊM

1. **Servlet & JSP Tutorial**: https://www.javatpoint.com/servlet-tutorial
2. **JSTL Guide**: https://www.tutorialspoint.com/jsp/jsp_standard_tag_library.htm
3. **JDBC Tutorial**: https://docs.oracle.com/javase/tutorial/jdbc/
4. **MVC Pattern**: https://www.geeksforgeeks.org/mvc-design-pattern/

---

## ✅ CHECKLIST HOÀN THÀNH

Đánh dấu ✅ khi bạn đã làm xong:

- [ ] Đọc hết Phần A (Kiến thức nền tảng)
- [ ] Đọc hết Phần B (Code chi tiết Model, DAO, Servlet, JSP)
- [ ] Đọc hết Phần C (Luồng xử lý hoàn chỉnh)
- [ ] Đọc hết Phần D (Chuẩn bị báo cáo)
- [ ] Viết lại code Product.java
- [ ] Viết lại code ProductDAO.java
- [ ] Viết lại code ProductServlet.java
- [ ] Viết lại code product_list.jsp
- [ ] Test chức năng Xem danh sách
- [ ] Test chức năng Thêm sản phẩm
- [ ] Test chức năng Sửa sản phẩm
- [ ] Test chức năng Xóa sản phẩm
- [ ] Trả lời được 8 câu hỏi trong Phần D1
- [ ] Demo thử 1 lần hoàn chỉnh
- [ ] Tự tin giải thích cho thầy

---

🎉 **CHÚC BẠN THÀNH CÔNG VỚI BÀI BÁO CÁO!** 🎉

Nếu có thắc mắc, đọc lại tài liệu hoặc xem phần Q&A trong từng section.

**Liên hệ**: [Tên bạn] - ThanhVien1 - Module Quản lý Sản phẩm

---

**HẾT TÀI LIỆU**

