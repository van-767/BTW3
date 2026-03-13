# BÁO CÁO KẾT QUẢ TÌM VÀ SỬA LỖI - CAMPUS BOOKSTORE LAB

## 1. Tổng quan
Hệ thống quản trị nhà sách Campus Bookstore được phát triển dựa trên ngôn ngữ PHP. Qua quá trình kiểm tra, tôi đã phát hiện và khắc phục tổng cộng **12 lỗi**, bao gồm 3 lỗi cú pháp (Syntax) và 9 lỗi logic (Business Logic).

---

## 2. Danh sách tổng hợp lỗi

| STT | Loại lỗi | Trang/File | Mô tả ngắn gọn |
|:---:|:---:|:---|:---|
| 1 | **Syntax** | `pages/customers.php` | Thiếu dấu ngoặc vuông `]` trong điều kiện `if`. |
| 2 | **Syntax** | `pages/reports.php` | Thiếu dấu chấm phẩy `;` kết thúc khai báo mảng. |
| 3 | **Syntax** | `pages/settings.php` | Sai cú pháp kết thúc mảng (Thiếu dấu `]`). |
| 4 | **Logic** | `pages/dashboard.php` | Sai điều kiện lọc đơn hàng "Completed". |
| 5 | **Logic** | `pages/orders.php` | Lọc ngược trạng thái đơn hàng trong hàng đợi. |
| 6 | **Logic** | `pages/checkout.php` | Phép gán đè thay vì cộng dồn khi tính tổng tiền. |
| 7 | **Logic** | `pages/reports.php` | Gom nhóm báo cáo theo tên thay vì theo danh mục. |
| 8 | **Logic** | `includes/data.php` | Bỏ quên số lượng (quantity) khi tính tổng đơn hàng. |
| 9 | **Logic** | `includes/data.php` | Sử dụng phép cộng thay vì phép nhân cho giá trị tồn kho. |
| 10 | **Logic** | `pages/dashboard.php` | Xác định sai ngưỡng "Sắp hết hàng" (Low Stock). |
| 11 | **Logic** | `pages/checkout.php` | Thuế VAT tính trên giá gốc (trước discount) là sai quy tắc. |
| 12 | **Logic** | `pages/dashboard.php` | Doanh thu tổng hợp bị lọc thiếu do đặt sai vị trí code. |

---

## 3. Chi tiết các lỗi Syntax (Cú pháp)

### 1. Thiếu ngoặc vuông `]` trong mảng
*   **Trang/File:** `pages/customers.php` (Dòng 6)
*   **Chi tiết:** Lỗi gọi key mảng không hợp lệ.
*   **Code sai:** `if ($customer['active')`
*   **Cách khắc phục:** Thêm dấu `]` để đóng key mảng: `if ($customer['active'])`

### 2. Thiếu dấu chấm phẩy `;`
*   **Trang/File:** `pages/reports.php` (Dòng 3)
*   **Chi tiết:** Không kết thúc câu lệnh khai báo biến, gây crash trang.
*   **Code sai:** `$reportRows = []`
*   **Cách khắc phục:** Thêm dấu `;` ở cuối: `$reportRows = [];`

### 3. Sai cú pháp đóng mảng
*   **Trang/File:** `pages/settings.php` (Dòng 7)
*   **Chi tiết:** Khai báo mảng đa dòng nhưng quên đóng ngoặc vuông `]`.
*   **Code sai:** `;`
*   **Cách khắc phục:** Thay bằng `];`

---

## 4. Chi tiết các lỗi Logic 

### 4. Nhầm lẫn trạng thái đơn hàng (Dashboard)
*   **Trang/File:** `pages/dashboard.php` (Dòng 7)
*   **Chi tiết:** Mục "Packed orders" cần đếm đơn đã hoàn thành (`completed`), nhưng lại đang kiểm tra đơn `pending`.
*   **Code sai:** `if ($order['status'] === 'pending')`
*   **Cách khắc phục:** Đổi thành `if ($order['status'] === 'completed')`

### 5. Lọc sai queue cho Pending Orders
*   **Trang/File:** `pages/orders.php` (Dòng 6)
*   **Chi tiết:** Trang hàng đợi lấy nhầm danh sách đơn `completed` để hiển thị trong khi yêu cầu là đơn chưa xử lý.
*   **Code sai:** `if ($order['status'] === 'completed')`
*   **Cách khắc phục:** Đổi thành `if ($order['status'] === 'pending')`

### 6. Ghi đè Subtotal thay vì cộng dồn (Checkout)
*   **Trang/File:** `pages/checkout.php` (Dòng 11)
*   **Chi tiết:** Trong vòng lặp, giá trị `subtotal` liên tục bị ghi đè bởi món hàng cuối cùng.
*   **Code sai:** `$subtotal = $products[$item['sku']]['price'] * $item['qty'];`
*   **Cách khắc phục:** Sử dụng toán tử `+=`: `$subtotal += $products[$item['sku']]['price'] * $item['qty'];`

### 7. Gom nhóm sai giá trị trong Reports
*   **Trang/File:** `pages/reports.php` (Dòng 7)
*   **Chi tiết:** Báo cáo cần theo Category nhưng code lại lấy Tên sản phẩm làm khóa gom nhóm.
*   **Code sai:** `$category = $product['name'];`
*   **Cách khắc phục:** Đổi lấy `category`: `$category = $product['category'];`

### 8. Quên nhân số lượng (Quantity) khi tính tổng bill
*   **Trang/File:** `includes/data.php` (Dòng 12 - Hàm `calculate_order_total`)
*   **Chi tiết:** Chỉ cộng giá tiền mặt hàng mà quên không nhân với số lượng khách mua.
*   **Code sai:** `$subtotal += $products[$item['sku']]['price'];`
*   **Cách khắc phục:** `$subtotal += $products[$item['sku']]['price'] * $item['qty'];`

### 9. Cộng thay vì nhân khi tính giá trị tồn kho
*   **Trang/File:** `includes/data.php` (Dòng 23 - Hàm `calculate_inventory_value`)
*   **Chi tiết:** Tính tổng giá trị Shelf Value bằng cách lấy `Price + Stock` là sai logic toán học.
*   **Code sai:** `$value += $product['price'] + $product['stock'];`
*   **Cách khắc phục:** Phải dùng phép nhân: `$value += $product['price'] * $product['stock'];`

### 10. Xác định ngưỡng "Low Stock" không hợp lý
*   **Trang/File:** `pages/dashboard.php` (Dòng 16)
*   **Chi tiết:** "Low stock" nên bao gồm các mặt hàng có số lượng ít (ví dụ <= 5), thay vì chỉ đếm mặt hàng đã hết nhẵn (< 1).
*   **Code sai:** `if ($product['stock'] < 1)`
*   **Cách khắc phục:** Cập nhật ngưỡng: `if ($product['stock'] <= 5)`

### 11. Tính thuế VAT sai quy trình kế toán
*   **Trang/File:** `pages/checkout.php` (Dòng 17)
*   **Chi tiết:** Thuế nên được tính trên số tiền thực thu (sau giảm giá) thay vì giá niêm yết ban đầu.
*   **Code sai:** `$vat = $subtotal * 0.1;`
*   **Cách khắc phục:** `$vat = ($subtotal - $discountValue) * 0.1;`

### 12. Lỗi báo cáo doanh thu không gom đủ toàn bộ đơn hàng
*   **Trang/File:** `pages/dashboard.php` (Dòng 9)
*   **Chi tiết:** Phép cộng `totalRevenue` bị kẹp trong điều kiện `if` chỉ xét một trạng thái đơn hàng.
*   **Code sai (lồng ghép sai):**
    ```php
    if ($order['status'] === 'completed') {
        $completedOrders++;
        $totalRevenue += calculate_order_total($order, $products); 
    }
    ```
*   **Cách khắc phục:** Tách việc tính doanh thu ra ngoài để tính cho toàn bộ queue:
    ```php
    $totalRevenue += calculate_order_total($order, $products);
    if ($order['status'] === 'completed') {
        $completedOrders++;
    }
    ```

