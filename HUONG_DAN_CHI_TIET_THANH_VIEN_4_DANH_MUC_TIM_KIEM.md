# 📘 HƯỚNG DẪN CHI TIẾT - THÀNH VIÊN 4: DANH MỤC & TÌM KIẾM

> **Mục tiêu**: Học từng bước, hiểu từng dòng code, tự tin giải thích cho thầy  
> **Module**: Quản lý Danh mục & Tìm kiếm sản phẩm  
> **Thời gian học**: 2-3 tuần

---

## 📑 MỤC LỤC

### PHẦN A: KIẾN THỨC NỀN TẢNG
1. [SQL LIKE & Full-text Search](#a1)
2. [Pagination (Phân trang)](#a2)
3. [Filter & Sort](#a3)

### PHẦN B: CODE CHI TIẾT
1. [Model: Category.java](#b1)
2. [DAO: CategoryDAO.java](#b2)
3. [Servlet: CategoryServlet & SearchServlet](#b3)
4. [View: JSP Pages](#b4)

### PHẦN C: LUỒNG XỬ LÝ
1. [Lọc sản phẩm theo danh mục](#c1)
2. [Tìm kiếm sản phẩm](#c2)
3. [Phân trang kết quả](#c3)

### PHẦN D: CÂU HỎI & DEMO
1. [Câu hỏi thầy thường hỏi](#d1)
2. [Kịch bản demo](#d2)

---

# PHẦN A: KIẾN THỨC NỀN TẢNG

<a name="a1"></a>
## A1. SQL LIKE & Full-text Search

### **1.1. SQL LIKE**

**Tìm kiếm cơ bản**:
```sql
-- Tìm sản phẩm có tên chứa "bánh"
SELECT * FROM products 
WHERE name LIKE '%bánh%';

-- Kết quả:
-- "Bánh quy"
-- "Bánh mì"
-- "Kẹo bánh"
```

**Wildcard characters**:
```
% : Bất kỳ chuỗi nào (0 hoặc nhiều ký tự)
_ : Đúng 1 ký tự

Ví dụ:
LIKE 'b%'     → "bánh", "bim bim"
LIKE '%nh'    → "bánh", "chanh"
LIKE 'b_nh'   → "bành", "bình"
LIKE '%bánh%' → "Bánh quy", "Kẹo bánh"
```

**Tìm trong nhiều cột**:
```sql
SELECT * FROM products 
WHERE name LIKE '%bánh%' 
   OR description LIKE '%bánh%';
```

### **1.2. Tối ưu tìm kiếm**

**Vấn đề LIKE**:
```
LIKE '%keyword%' không dùng index → Chậm với DB lớn
```

**Giải pháp 1: FULLTEXT Index** (MySQL):
```sql
-- Tạo FULLTEXT index
ALTER TABLE products ADD FULLTEXT(name, description);

-- Tìm kiếm FULLTEXT
SELECT * FROM products 
WHERE MATCH(name, description) AGAINST('bánh quy' IN NATURAL LANGUAGE MODE);
```

**Giải pháp 2: Elasticsearch** (Production):
```
Dùng Elasticsearch cho search phức tạp:
- Tìm kiếm mờ (fuzzy search)
- Xử lý tiếng Việt có dấu
- Autocomplete
- Ranking kết quả
```

---

<a name="a2"></a>
## A2. Pagination (Phân trang)

### **2.1. Tại sao cần phân trang?**

**Vấn đề**: 
```
Có 1000 sản phẩm → Hiển thị hết trên 1 trang:
- Tải chậm (query 1000 rows)
- Browser treo (render 1000 elements)
- UX tệ (scroll mãi mới hết)
```

**Giải pháp**: Chia nhỏ thành nhiều trang
```
Trang 1: Sản phẩm 1-20
Trang 2: Sản phẩm 21-40
Trang 3: Sản phẩm 41-60
...
```

### **2.2. SQL LIMIT & OFFSET**

**Công thức**:
```
LIMIT: Số dòng lấy ra (pageSize)
OFFSET: Bỏ qua bao nhiêu dòng

OFFSET = (pageNumber - 1) * pageSize
```

**Ví dụ**:
```sql
-- Trang 1: Lấy 20 sản phẩm đầu
SELECT * FROM products 
ORDER BY id DESC 
LIMIT 20 OFFSET 0;

-- Trang 2: Bỏ qua 20 sản phẩm đầu, lấy 20 sản phẩm tiếp
SELECT * FROM products 
ORDER BY id DESC 
LIMIT 20 OFFSET 20;

-- Trang 3: Bỏ qua 40, lấy 20
SELECT * FROM products 
ORDER BY id DESC 
LIMIT 20 OFFSET 40;
```

### **2.3. Code Java**

```java
/**
 * Lấy sản phẩm theo trang.
 * 
 * @param pageNumber Trang số mấy (bắt đầu từ 1)
 * @param pageSize   Số sản phẩm mỗi trang
 */
public List<Product> getProductsByPage(int pageNumber, int pageSize) {
    List<Product> products = new ArrayList<>();
    
    // Tính OFFSET
    int offset = (pageNumber - 1) * pageSize;
    
    String sql = "SELECT * FROM products ORDER BY id DESC LIMIT ? OFFSET ?";
    
    try (PreparedStatement ps = connection.prepareStatement(sql)) {
        ps.setInt(1, pageSize);
        ps.setInt(2, offset);
        
        ResultSet rs = ps.executeQuery();
        while (rs.next()) {
            // Chuyển ResultSet → Product
            products.add(mapResultSetToProduct(rs));
        }
    } catch (SQLException e) {
        e.printStackTrace();
    }
    
    return products;
}

/**
 * Tính tổng số trang.
 */
public int getTotalPages(int pageSize) {
    int totalProducts = getTotalCount();  // Tổng số sản phẩm
    return (int) Math.ceil((double) totalProducts / pageSize);
}
```

**Ví dụ sử dụng trong Servlet**:
```java
// Lấy page từ URL (mặc định page 1)
String pageStr = request.getParameter("page");
int pageNumber = (pageStr != null) ? Integer.parseInt(pageStr) : 1;
int pageSize = 20;

// Lấy dữ liệu trang hiện tại
List<Product> products = productDAO.getProductsByPage(pageNumber, pageSize);

// Tính tổng số trang
int totalPages = productDAO.getTotalPages(pageSize);

// Đưa vào request scope
request.setAttribute("products", products);
request.setAttribute("currentPage", pageNumber);
request.setAttribute("totalPages", totalPages);
```

---

<a name="a3"></a>
## A3. Filter & Sort

### **3.1. Filter (Lọc)**

**Lọc theo danh mục**:
```sql
SELECT * FROM products 
WHERE category_id = ?;
```

**Lọc theo khoảng giá**:
```sql
SELECT * FROM products 
WHERE price BETWEEN ? AND ?;
```

**Lọc nhiều điều kiện**:
```sql
SELECT * FROM products 
WHERE category_id = ? 
  AND price BETWEEN ? AND ?
  AND stock > 0
ORDER BY price ASC;
```

### **3.2. Sort (Sắp xếp)**

**Các kiểu sắp xếp**:
```sql
-- Tên A-Z
ORDER BY name ASC

-- Tên Z-A
ORDER BY name DESC

-- Giá thấp → cao
ORDER BY price ASC

-- Giá cao → thấp
ORDER BY price DESC

-- Mới nhất
ORDER BY created_at DESC

-- Bán chạy nhất (cần JOIN với order_items)
SELECT p.*, COUNT(oi.id) AS sold_count
FROM products p
LEFT JOIN order_items oi ON p.id = oi.product_id
GROUP BY p.id
ORDER BY sold_count DESC;
```

**Dynamic ORDER BY trong Java**:
```java
public List<Product> getProducts(String sortBy) {
    // Validate sortBy để tránh SQL Injection
    String orderClause;
    switch (sortBy) {
        case "name_asc":
            orderClause = "name ASC";
            break;
        case "name_desc":
            orderClause = "name DESC";
            break;
        case "price_asc":
            orderClause = "price ASC";
            break;
        case "price_desc":
            orderClause = "price DESC";
            break;
        default:
            orderClause = "id DESC";  // Mặc định
    }
    
    String sql = "SELECT * FROM products ORDER BY " + orderClause;
    // ... executeQuery
}
```

---

# PHẦN B: CODE CHI TIẾT

<a name="b1"></a>
## B1. Model: Category.java

```java
package model;

import java.io.Serializable;

/**
 * Lớp Category đại diện cho danh mục sản phẩm.
 * 
 * @author ThanhVien4
 */
public class Category implements Serializable {
    
    private int id;
    private String name;
    private String description;
    private int productCount;  // Số sản phẩm trong danh mục
    
    // Constructors
    public Category() {}
    
    public Category(int id, String name, String description) {
        this.id = id;
        this.name = name;
        this.description = description;
    }
    
    // Getters & Setters
    public int getId() { return id; }
    public void setId(int id) { this.id = id; }
    
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    
    public String getDescription() { return description; }
    public void setDescription(String description) { this.description = description; }
    
    public int getProductCount() { return productCount; }
    public void setProductCount(int productCount) { this.productCount = productCount; }
    
    @Override
    public String toString() {
        return "Category [id=" + id + ", name=" + name + ", productCount=" + productCount + "]";
    }
}
```

---

<a name="b2"></a>
## B2. DAO: CategoryDAO.java

```java
package dao;

import model.Category;
import java.sql.*;
import java.util.*;

public class CategoryDAO {
    
    private Connection connection;
    
    public CategoryDAO() {
        try {
            this.connection = DBConnection.getInstance().getConnection();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
    
    /**
     * Lấy tất cả danh mục (kèm số lượng sản phẩm).
     */
    public List<Category> getAll() {
        List<Category> categories = new ArrayList<>();
        
        String sql = "SELECT c.id, c.name, c.description, COUNT(p.id) AS product_count " +
                    "FROM categories c " +
                    "LEFT JOIN products p ON c.id = p.category_id " +
                    "GROUP BY c.id, c.name, c.description " +
                    "ORDER BY c.name ASC";
        
        try (PreparedStatement ps = connection.prepareStatement(sql);
             ResultSet rs = ps.executeQuery()) {
            
            while (rs.next()) {
                Category cat = new Category();
                cat.setId(rs.getInt("id"));
                cat.setName(rs.getString("name"));
                cat.setDescription(rs.getString("description"));
                cat.setProductCount(rs.getInt("product_count"));
                categories.add(cat);
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
        
        return categories;
    }
    
    /**
     * Lấy 1 danh mục theo ID.
     */
    public Category getById(int id) {
        Category category = null;
        String sql = "SELECT * FROM categories WHERE id = ?";
        
        try (PreparedStatement ps = connection.prepareStatement(sql)) {
            ps.setInt(1, id);
            ResultSet rs = ps.executeQuery();
            
            if (rs.next()) {
                category = new Category();
                category.setId(rs.getInt("id"));
                category.setName(rs.getString("name"));
                category.setDescription(rs.getString("description"));
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
        
        return category;
    }
    
    /**
     * Thêm danh mục mới.
     */
    public boolean insert(Category category) {
        String sql = "INSERT INTO categories (name, description) VALUES (?, ?)";
        
        try (PreparedStatement ps = connection.prepareStatement(sql)) {
            ps.setString(1, category.getName());
            ps.setString(2, category.getDescription());
            return ps.executeUpdate() > 0;
        } catch (SQLException e) {
            e.printStackTrace();
            return false;
        }
    }
    
    /**
     * Cập nhật danh mục.
     */
    public boolean update(Category category) {
        String sql = "UPDATE categories SET name = ?, description = ? WHERE id = ?";
        
        try (PreparedStatement ps = connection.prepareStatement(sql)) {
            ps.setString(1, category.getName());
            ps.setString(2, category.getDescription());
            ps.setInt(3, category.getId());
            return ps.executeUpdate() > 0;
        } catch (SQLException e) {
            e.printStackTrace();
            return false;
        }
    }
    
    /**
     * Xóa danh mục (kiểm tra foreign key).
     */
    public boolean delete(int id) {
        String sql = "DELETE FROM categories WHERE id = ?";
        
        try (PreparedStatement ps = connection.prepareStatement(sql)) {
            ps.setInt(1, id);
            return ps.executeUpdate() > 0;
        } catch (SQLException e) {
            // Kiểm tra lỗi foreign key
            if (e.getMessage().contains("foreign key")) {
                System.err.println("Không thể xóa: Danh mục đang có sản phẩm!");
            }
            e.printStackTrace();
            return false;
        }
    }
}
```

---

# PHẦN C: LUỒNG XỬ LÝ

<a name="c1"></a>
## C1. Lọc sản phẩm theo danh mục

```
1. User click vào danh mục "Bánh kẹo"
2. GET: ProductServlet?action=category&categoryId=1
3. Servlet gọi productDAO.getByCategoryId(1)
4. DAO thực thi: SELECT * FROM products WHERE category_id = 1
5. Trả về List<Product>
6. Forward sang product_list.jsp
7. JSP hiển thị sản phẩm + breadcrumb "Bánh kẹo"
```

---

<a name="c2"></a>
## C2. Tìm kiếm sản phẩm

```
1. User nhập "bánh" vào ô tìm kiếm, bấm Enter
2. GET: SearchServlet?keyword=bánh
3. Servlet gọi productDAO.search("bánh")
4. DAO thực thi: 
   SELECT * FROM products 
   WHERE name LIKE '%bánh%' OR description LIKE '%bánh%'
5. Trả về List<Product> (kết quả tìm kiếm)
6. Forward sang search_results.jsp
7. JSP hiển thị: "Tìm thấy 5 sản phẩm cho 'bánh'"
```

---

# PHẦN D: CÂU HỎI THẦY

<a name="d1"></a>
## Q1: "LIKE '%keyword%' có vấn đề gì?"

**Trả lời**:
```
Thưa thầy, LIKE '%keyword%' có 2 vấn đề:

1. KHÔNG DÙNG INDEX:
   - Index chỉ hoạt động với: LIKE 'keyword%'
   - LIKE '%keyword%' phải scan toàn bộ bảng → Chậm

2. CASE SENSITIVE (tùy database):
   - MySQL: Mặc định không phân biệt hoa thường
   - PostgreSQL: Phân biệt → Phải dùng ILIKE

Giải pháp cho DB lớn:
- Dùng FULLTEXT Index
- Hoặc Elasticsearch
```

## Q2: "Em giải thích phân trang?"

**Trả lời**:
```
Thưa thầy, phân trang chia dữ liệu thành nhiều trang nhỏ.

Công thức:
OFFSET = (pageNumber - 1) * pageSize

Ví dụ: 100 sản phẩm, mỗi trang 20
- Trang 1: LIMIT 20 OFFSET 0  (sản phẩm 1-20)
- Trang 2: LIMIT 20 OFFSET 20 (sản phẩm 21-40)
- Trang 3: LIMIT 20 OFFSET 40 (sản phẩm 41-60)

Tổng số trang = CEIL(100 / 20) = 5 trang
```

## Q3: "Dynamic ORDER BY có nguy hiểm không?"

**Trả lời**:
```
Thưa thầy, CÓ nguy hiểm SQL Injection nếu không validate!

SAI (nguy hiểm):
String sql = "SELECT * FROM products ORDER BY " + sortBy;
→ User gửi: sortBy = "name; DROP TABLE products"
→ SQL thành: ... ORDER BY name; DROP TABLE products

ĐÚNG (an toàn):
- Dùng switch-case validate sortBy
- Chỉ cho phép các giá trị cố định
- Không nối chuỗi trực tiếp
```

---

**HẾT TÀI LIỆU THÀNH VIÊN 4**