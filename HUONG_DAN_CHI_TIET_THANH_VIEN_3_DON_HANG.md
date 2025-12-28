# 📘 HƯỚNG DẪN CHI TIẾT - THÀNH VIÊN 3: QUẢN LÝ ĐƠN HÀNG

> **Mục tiêu**: Học từng bước, hiểu từng dòng code, tự tin giải thích cho thầy  
> **Module**: Quản lý Đơn hàng (Orders & Order Items)  
> **Thời gian học**: 2-3 tuần

---

## 📑 MỤC LỤC

### PHẦN A: KIẾN THỨC NỀN TẢNG
1. [Transaction trong Database](#a1)
2. [Master-Detail Pattern](#a2)
3. [Order Status Workflow](#a3)

### PHẦN B: CODE CHI TIẾT
1. [Model: Order.java & OrderItem.java](#b1)
2. [DAO: OrderDAO.java](#b2)
3. [Servlet: OrderServlet.java](#b3)
4. [View: JSP Pages](#b4)

### PHẦN C: LUỒNG XỬ LÝ
1. [Đặt hàng từ giỏ hàng](#c1)
2. [Xem danh sách đơn hàng](#c2)
3. [Xem chi tiết đơn hàng](#c3)

### PHẦN D: CÂU HỎI & DEMO
1. [Câu hỏi thầy thường hỏi](#d1)
2. [Kịch bản demo](#d2)

---

# PHẦN A: KIẾN THỨC NỀN TẢNG

<a name="a1"></a>
## A1. Transaction trong Database

### **Transaction là gì?**

**Định nghĩa**: Một nhóm các thao tác DB phải thực thi **trọn vẹn hoặc không thực thi gì cả**.

**Ví dụ: Đặt hàng**
```
Bước 1: INSERT vào bảng orders (tạo đơn hàng)
Bước 2: INSERT vào bảng order_items (lưu chi tiết)
Bước 3: UPDATE bảng products (giảm stock)

Nếu Bước 2 LỖI → Phải hủy Bước 1 (rollback)
Không thể có đơn hàng mà không có chi tiết!
```

### **4 tính chất ACID**

```
A - Atomicity (Nguyên tử): Tất cả hoặc không có gì
C - Consistency (Nhất quán): DB luôn ở trạng thái hợp lệ
I - Isolation (Độc lập): Transaction không ảnh hưởng lẫn nhau
D - Durability (Bền vững): Dữ liệu không mất sau commit
```

### **Sử dụng Transaction trong JDBC**

```java
Connection conn = null;
try {
    conn = DBConnection.getInstance().getConnection();
    
    // BẮT ĐẦU TRANSACTION
    conn.setAutoCommit(false);
    
    // Thao tác 1: INSERT order
    PreparedStatement ps1 = conn.prepareStatement(
        "INSERT INTO orders (user_id, total_price, status) VALUES (?, ?, ?)",
        Statement.RETURN_GENERATED_KEYS
    );
    ps1.setInt(1, userId);
    ps1.setBigDecimal(2, totalPrice);
    ps1.setString(3, "PENDING");
    ps1.executeUpdate();
    
    // Lấy orderId vừa tạo
    ResultSet rs = ps1.getGeneratedKeys();
    int orderId = 0;
    if (rs.next()) {
        orderId = rs.getInt(1);
    }
    
    // Thao tác 2: INSERT order_items
    PreparedStatement ps2 = conn.prepareStatement(
        "INSERT INTO order_items (order_id, product_id, quantity, price) VALUES (?, ?, ?, ?)"
    );
    for (CartItem item : cart.getItems()) {
        ps2.setInt(1, orderId);
        ps2.setInt(2, item.getProduct().getId());
        ps2.setInt(3, item.getQuantity());
        ps2.setBigDecimal(4, item.getProduct().getPrice());
        ps2.executeUpdate();
    }
    
    // COMMIT: Lưu tất cả thay đổi
    conn.commit();
    System.out.println("✅ Đặt hàng thành công!");
    
} catch (SQLException e) {
    // ROLLBACK: Hủy tất cả thay đổi
    if (conn != null) {
        try {
            conn.rollback();
            System.out.println("❌ Rollback do lỗi!");
        } catch (SQLException ex) {
            ex.printStackTrace();
        }
    }
    e.printStackTrace();
} finally {
    // Trả về chế độ auto-commit
    if (conn != null) {
        try {
            conn.setAutoCommit(true);
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```

---

<a name="a2"></a>
## A2. Master-Detail Pattern

### **Pattern là gì?**

```
Master Table (orders)          Detail Table (order_items)
┌────┬─────────┬────────┐      ┌────┬──────────┬────────────┬──────┐
│ id │ user_id │  total │      │ id │ order_id │ product_id │ qty  │
├────┼─────────┼────────┤      ├────┼──────────┼────────────┼──────┤
│ 1  │   101   │ 60000  │◄─────┤ 1  │    1     │     5      │  2   │
└────┴─────────┴────────┘      ├────┼──────────┼────────────┼──────┤
                                │ 2  │    1     │     7      │  3   │
                                └────┴──────────┴────────────┴──────┘
                                     (Foreign Key)
```

**Đặc điểm**:
- 1 Master có nhiều Detail (1-N relationship)
- Detail chứa foreign key trỏ về Master
- Xóa Master phải xóa Detail trước (hoặc CASCADE)

---

<a name="a3"></a>
## A3. Order Status Workflow

```
┌─────────┐   Đặt hàng   ┌─────────┐   Xác nhận   ┌──────────┐
│ PENDING ├─────────────►│CONFIRMED├─────────────►│SHIPPING  │
└─────────┘              └─────────┘              └────┬─────┘
                                                        │
                          ┌──────────┐    Giao hàng    │
                          │DELIVERED │◄─────────────────┘
                          └──────────┘
                                │
                          ┌─────▼────┐
                          │COMPLETED │
                          └──────────┘
```

---

# PHẦN B: CODE CHI TIẾT

<a name="b1"></a>
## B1. Model: Order.java & OrderItem.java

### **Order.java**

```java
package model;

import java.io.Serializable;
import java.math.BigDecimal;
import java.time.LocalDateTime;
import java.util.List;

public class Order implements Serializable {
    
    private int id;
    private int userId;
    private BigDecimal totalPrice;
    private String status;  // PENDING, CONFIRMED, SHIPPING, DELIVERED
    private LocalDateTime createdAt;
    
    // Quan hệ với OrderItem (1-N)
    private List<OrderItem> orderItems;
    
    // Constructors, Getters/Setters...
    
    public Order() {}
    
    public Order(int userId, BigDecimal totalPrice, String status) {
        this.userId = userId;
        this.totalPrice = totalPrice;
        this.status = status;
    }
    
    // Getters/Setters (tự bổ sung đầy đủ)
}
```

### **OrderItem.java**

```java
package model;

import java.io.Serializable;
import java.math.BigDecimal;

public class OrderItem implements Serializable {
    
    private int id;
    private int orderId;      // Foreign key
    private int productId;
    private String productName;
    private int quantity;
    private BigDecimal price;
    
    // Constructors, Getters/Setters...
    
    public BigDecimal getSubtotal() {
        return price.multiply(new BigDecimal(quantity));
    }
}
```

---

<a name="b2"></a>
## B2. DAO: OrderDAO.java

```java
package dao;

import model.Order;
import model.OrderItem;
import helper.Cart;
import helper.CartItem;

import java.sql.*;
import java.util.*;

public class OrderDAO {
    
    private Connection connection;
    
    public OrderDAO() {
        try {
            this.connection = DBConnection.getInstance().getConnection();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
    
    /**
     * Tạo đơn hàng từ giỏ hàng.
     * Sử dụng TRANSACTION để đảm bảo tính nhất quán.
     */
    public boolean createOrder(int userId, Cart cart) {
        
        if (cart == null || cart.isEmpty()) {
            return false;
        }
        
        Connection conn = null;
        PreparedStatement psOrder = null;
        PreparedStatement psItem = null;
        
        try {
            conn = DBConnection.getInstance().getConnection();
            conn.setAutoCommit(false);  // Bắt đầu transaction
            
            // Bước 1: INSERT vào bảng orders
            String sqlOrder = "INSERT INTO orders (user_id, total_price, status, created_at) " +
                             "VALUES (?, ?, ?, NOW())";
            psOrder = conn.prepareStatement(sqlOrder, Statement.RETURN_GENERATED_KEYS);
            psOrder.setInt(1, userId);
            psOrder.setBigDecimal(2, cart.getTotalPrice());
            psOrder.setString(3, "PENDING");
            psOrder.executeUpdate();
            
            // Lấy orderId vừa tạo
            ResultSet rs = psOrder.getGeneratedKeys();
            int orderId = 0;
            if (rs.next()) {
                orderId = rs.getInt(1);
            }
            
            // Bước 2: INSERT vào bảng order_items
            String sqlItem = "INSERT INTO order_items (order_id, product_id, quantity, price) " +
                            "VALUES (?, ?, ?, ?)";
            psItem = conn.prepareStatement(sqlItem);
            
            for (CartItem cartItem : cart.getItems()) {
                psItem.setInt(1, orderId);
                psItem.setInt(2, cartItem.getProduct().getId());
                psItem.setInt(3, cartItem.getQuantity());
                psItem.setBigDecimal(4, cartItem.getProduct().getPrice());
                psItem.addBatch();  // Thêm vào batch
            }
            psItem.executeBatch();  // Thực thi tất cả
            
            // COMMIT
            conn.commit();
            System.out.println("✅ Đặt hàng thành công! Order ID: " + orderId);
            return true;
            
        } catch (SQLException e) {
            // ROLLBACK
            if (conn != null) {
                try {
                    conn.rollback();
                    System.out.println("❌ Rollback transaction!");
                } catch (SQLException ex) {
                    ex.printStackTrace();
                }
            }
            e.printStackTrace();
            return false;
        } finally {
            try {
                if (conn != null) conn.setAutoCommit(true);
                if (psOrder != null) psOrder.close();
                if (psItem != null) psItem.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }
    
    /**
     * Lấy danh sách đơn hàng của user.
     */
    public List<Order> getOrdersByUserId(int userId) {
        List<Order> orders = new ArrayList<>();
        String sql = "SELECT * FROM orders WHERE user_id = ? ORDER BY created_at DESC";
        
        try (PreparedStatement ps = connection.prepareStatement(sql)) {
            ps.setInt(1, userId);
            ResultSet rs = ps.executeQuery();
            
            while (rs.next()) {
                Order order = new Order();
                order.setId(rs.getInt("id"));
                order.setUserId(rs.getInt("user_id"));
                order.setTotalPrice(rs.getBigDecimal("total_price"));
                order.setStatus(rs.getString("status"));
                order.setCreatedAt(rs.getTimestamp("created_at").toLocalDateTime());
                orders.add(order);
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
        
        return orders;
    }
    
    /**
     * Lấy chi tiết đơn hàng (order + items).
     */
    public Order getOrderDetail(int orderId) {
        Order order = null;
        
        // Lấy thông tin order
        String sqlOrder = "SELECT * FROM orders WHERE id = ?";
        try (PreparedStatement ps = connection.prepareStatement(sqlOrder)) {
            ps.setInt(1, orderId);
            ResultSet rs = ps.executeQuery();
            
            if (rs.next()) {
                order = new Order();
                order.setId(rs.getInt("id"));
                order.setUserId(rs.getInt("user_id"));
                order.setTotalPrice(rs.getBigDecimal("total_price"));
                order.setStatus(rs.getString("status"));
                order.setCreatedAt(rs.getTimestamp("created_at").toLocalDateTime());
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
        
        // Lấy order_items
        if (order != null) {
            List<OrderItem> items = getOrderItems(orderId);
            order.setOrderItems(items);
        }
        
        return order;
    }
    
    /**
     * Lấy danh sách order_items của 1 đơn hàng.
     */
    private List<OrderItem> getOrderItems(int orderId) {
        List<OrderItem> items = new ArrayList<>();
        String sql = "SELECT oi.*, p.name AS product_name " +
                    "FROM order_items oi " +
                    "JOIN products p ON oi.product_id = p.id " +
                    "WHERE oi.order_id = ?";
        
        try (PreparedStatement ps = connection.prepareStatement(sql)) {
            ps.setInt(1, orderId);
            ResultSet rs = ps.executeQuery();
            
            while (rs.next()) {
                OrderItem item = new OrderItem();
                item.setId(rs.getInt("id"));
                item.setOrderId(rs.getInt("order_id"));
                item.setProductId(rs.getInt("product_id"));
                item.setProductName(rs.getString("product_name"));
                item.setQuantity(rs.getInt("quantity"));
                item.setPrice(rs.getBigDecimal("price"));
                items.add(item);
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
        
        return items;
    }
}
```

---

# PHẦN C: LUỒNG XỬ LÝ

<a name="c1"></a>
## C1. Đặt hàng từ giỏ hàng

```
1. User ở trang giỏ hàng, click "Đặt hàng"
2. POST: OrderServlet?action=checkout
3. Servlet lấy Cart từ Session
4. Servlet gọi orderDAO.createOrder(userId, cart)
5. DAO bắt đầu transaction:
   5a. INSERT vào orders → Lấy orderId
   5b. INSERT nhiều dòng vào order_items (batch)
6. COMMIT transaction
7. Xóa giỏ hàng (cart.clear())
8. Redirect sang trang "Đặt hàng thành công"
```

---

# PHẦN D: CÂU HỎI THẦY

<a name="d1"></a>
## Q1: "Em giải thích Transaction?"

**Trả lời**:
```
Thưa thầy, Transaction là nhóm thao tác DB phải thực thi trọn vẹn.

Ví dụ đặt hàng:
1. INSERT orders
2. INSERT order_items
→ Nếu bước 2 lỗi, phải hủy bước 1 (rollback)

Trong code em:
- conn.setAutoCommit(false): Bắt đầu transaction
- conn.commit(): Lưu tất cả
- conn.rollback(): Hủy tất cả nếu lỗi
```

## Q2: "Tại sao dùng executeBatch()?"

**Trả lời**:
```
Thưa thầy, executeBatch() gộp nhiều INSERT thành 1 lần gửi.

Không dùng batch:
for (item : items) {
    ps.executeUpdate();  // Gửi DB 10 lần (nếu 10 items)
}

Dùng batch:
for (item : items) {
    ps.addBatch();  // Gom lại
}
ps.executeBatch();  // Gửi DB 1 lần duy nhất

→ Nhanh hơn nhiều!
```

---

**HẾT TÀI LIỆU THÀNH VIÊN 3**