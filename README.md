# Database UMKM: baksoserbaada

Repositori ini berisi skrip SQL untuk sistem manajemen UMKM "Baksoserbaada", termasuk struktur database, contoh data, trigger, procedure, dan view.

---

## 1. Buat Database

```sql
CREATE DATABASE IF NOT EXISTS baksoserbaada;
USE baksoserbaada;

2. Tabel
2.1. Tabel category
CREATE TABLE category (
  category_id INT AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(100) NOT NULL,
  description TEXT
) ENGINE=InnoDB;

2.2. Tabel menu
CREATE TABLE menu (
  menu_id INT AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(150) NOT NULL,
  description TEXT,
  price DECIMAL(12,2) NOT NULL CHECK (price >= 0),
  stock INT NOT NULL DEFAULT 0 CHECK (stock >= 0),
  category_id INT,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  updated_at DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  FOREIGN KEY (category_id) REFERENCES category(category_id)
) ENGINE=InnoDB;

2.3. Tabel customer
CREATE TABLE customer (
  customer_id INT AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(150) NOT NULL,
  email VARCHAR(150) UNIQUE,
  phone VARCHAR(30),
  address TEXT,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB;

2.4. Tabel user
CREATE TABLE user (
  user_id INT AUTO_INCREMENT PRIMARY KEY,
  username VARCHAR(100) NOT NULL UNIQUE,
  password_hash VARCHAR(255) NOT NULL,
  role ENUM('admin','staff') DEFAULT 'staff',
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB;

2.5. Tabel order
CREATE TABLE `order` (
  order_id INT AUTO_INCREMENT PRIMARY KEY,
  customer_id INT NOT NULL,
  order_date DATETIME DEFAULT CURRENT_TIMESTAMP,
  status ENUM('pending','paid','processing','shipped','completed','cancelled') DEFAULT 'pending',
  total_price DECIMAL(12,2) NOT NULL DEFAULT 0,
  shipping_address TEXT,
  FOREIGN KEY (customer_id) REFERENCES customer(customer_id)
) ENGINE=InnoDB;

2.6. Tabel order_item
CREATE TABLE order_item (
  order_item_id INT AUTO_INCREMENT PRIMARY KEY,
  order_id INT NOT NULL,
  menu_id INT NOT NULL,
  qty INT NOT NULL CHECK (qty > 0),
  unit_price DECIMAL(12,2) NOT NULL CHECK (unit_price >= 0),
  subtotal DECIMAL(12,2) AS (qty * unit_price) STORED,
  FOREIGN KEY (order_id) REFERENCES `order`(order_id) ON DELETE CASCADE,
  FOREIGN KEY (menu_id) REFERENCES menu(menu_id)
) ENGINE=InnoDB;

2.7. Tabel payment
CREATE TABLE payment (
  payment_id INT AUTO_INCREMENT PRIMARY KEY,
  order_id INT UNIQUE,
  metode ENUM('bank_transfer','cod','qris') NOT NULL,
  amount DECIMAL(12,2) NOT NULL,
  payment_date DATETIME DEFAULT CURRENT_TIMESTAMP,
  status ENUM('waiting','paid','failed') DEFAULT 'waiting',
  FOREIGN KEY (order_id) REFERENCES `order`(order_id) ON DELETE CASCADE
) ENGINE=InnoDB;

2.8. Tabel delivery
CREATE TABLE delivery (
  delivery_id INT AUTO_INCREMENT PRIMARY KEY,
  order_id INT UNIQUE,
  kurir VARCHAR(100),
  tracking_no VARCHAR(100),
  status ENUM('ready','in_transit','delivered','failed') DEFAULT 'ready',
  send_date DATETIME,
  FOREIGN KEY (order_id) REFERENCES `order`(order_id) ON DELETE CASCADE
) ENGINE=InnoDB;

2.9. Tabel stock_change
CREATE TABLE stock_change (
  stock_change_id INT AUTO_INCREMENT PRIMARY KEY,
  menu_id INT NOT NULL,
  change_qty INT NOT NULL,
  reason VARCHAR(150),
  change_date DATETIME DEFAULT CURRENT_TIMESTAMP,
  user_id INT,
  FOREIGN KEY (menu_id) REFERENCES menu(menu_id),
  FOREIGN KEY (user_id) REFERENCES user(user_id)
) ENGINE=InnoDB;

3. Contoh Data
3.1. Category
INSERT INTO category (name, description) VALUES
('Bakso', 'Aneka menu bakso kuah'),
('Minuman', 'Aneka minuman pendamping'),
('Frozen', 'Bakso beku siap masak');

3.2. Customer
INSERT INTO customer (name, phone, address) VALUES
('Rudi Hartono', '081234567801', 'Jl. Melati No. 12, Jakarta'),
('Siti Aminah', '081234567802', 'Jl. Mawar No. 8, Bekasi'),
('Dimas Andika', '081234567803', 'Jl. Cemara No. 5, Depok'),
('Mega Arumsari', '081234567804', 'Jl. Kenanga No. 33, Tangerang'),
('Budi Setiawan', '081234567805', 'Jl. Anggrek No. 77, Jakarta'),
('Nita Sari', '081234567806', 'Jl. Sakura No. 27, Bogor'),
('Ahmad Fauzan', '081234567807', 'Jl. Mangga No. 19, Bekasi'),
('Rina Kartika', '081234567808', 'Jl. Teratai No. 21, Depok'),
('Bayu Pramana', '081234567809', 'Jl. Mawar No. 4, Jakarta'),
('Ayu Puspita', '081234567810', 'Jl. Durian No. 55, Tangerang');

3.3. User
INSERT INTO user (username, password_hash, role) VALUES
('admin', 'admin123', 'admin'),
('kasir1', 'kasir123', 'cashier'),
('kasir2', 'kasir456', 'cashier'),
('owner', 'ownerbos', 'owner'),
('gudang1', 'gudang123', 'inventory'),
('kurir1', 'antar123', 'courier');

3.4. Order
INSERT INTO `order` (customer_id, order_date, total_price, status, shipping_address) VALUES
(1, '2025-12-01 10:21:00', 67000, 'paid', 'Jl. Melati No. 12, Jakarta Selatan, DKI Jakarta'),
(2, '2025-12-01 10:45:00', 47000, 'paid', 'Jl. Anggrek Blok B3 No. 5, Bandung, Jawa Barat'),
(3, '2025-12-01 11:03:00', 95000, 'paid', 'Jl. Kencana Raya No. 44, Surabaya, Jawa Timur'),
(4, '2025-12-01 11:30:00', 30000, 'pending', 'Jl. Mawar Indah No. 8, Yogyakarta'),
(5, '2025-12-01 12:10:00', 55000, 'paid', 'Jl. Cemara No. 77, Malang, Jawa Timur'),
(6, '2025-12-01 12:41:00', 82000, 'paid', 'Jl. Anyelir No. 23, Semarang, Jawa Tengah'),
(7, '2025-12-01 13:15:00', 25000, 'cancelled', 'Jl. Teratai No. 2, Depok, Jawa Barat'),
(8, '2025-12-01 13:33:00', 75000, 'paid', 'Jl. Kenanga No. 6, Bekasi, Jawa Barat'),
(9, '2025-12-01 13:50:00', 50000, 'paid', 'Jl. Flamboyan No. 19, Tangerang, Banten'),
(10, '2025-12-01 14:05:00', 65000, 'paid', 'Jl. Dahlia No. 14, Bogor, Jawa Barat');

3.5. Order Item
INSERT INTO order_item (order_id, menu_id, qty, unit_price) VALUES
(1, 1, 2, 15000),
(1, 4, 1, 12000),
(2, 2, 3, 13000),
(3, 3, 2, 17500),
(3, 5, 1, 18000),
(4, 1, 1, 15000),
(5, 2, 2, 13000),
(5, 6, 1, 29000),
(6, 3, 2, 17500),
(6, 4, 2, 12000),
(7, 4, 1, 12000),
(8, 6, 1, 29000),
(8, 5, 2, 18000),
(9, 1, 2, 15000),
(9, 4, 1, 12000),
(10, 2, 1, 13000),
(10, 3, 1, 17500),
(10, 4, 2, 12000);

3.6. Payment
INSERT INTO payment (order_id, payment_date, amount, metode, status) VALUES
(1, '2025-12-01 10:25:00', 67000, 'cod', 'paid'),
(2, '2025-12-01 10:48:00', 47000, 'qris', 'paid'),
(3, '2025-12-01 11:05:00', 95000, 'bank_transfer', 'paid'),
(4, NULL, 30000, 'cod', 'waiting'),
(5, '2025-12-01 12:12:00', 55000, 'qris', 'paid'),
(6, '2025-12-01 12:45:00', 82000, 'cod', 'paid'),
(8, '2025-12-01 13:35:00', 75000, 'cod', 'paid'),
(9, '2025-12-01 13:53:00', 50000, 'qris', 'paid'),
(10, '2025-12-01 14:07:00', 65000, 'bank_transfer', 'paid');

3.7. Delivery
INSERT INTO delivery (order_id, kurir, tracking_no, status, send_date) VALUES
(1, 'Kurir1', 'TRK20251201-001', 'delivered', '2025-12-01 11:00:00'),
(2, 'Kurir1', 'TRK20251201-002', 'delivered', '2025-12-01 11:30:00'),
(3, 'Kurir2', 'TRK20251201-003', 'delivered', '2025-12-01 12:15:00'),
(5, 'Kurir2', 'TRK20251201-004', 'delivered', '2025-12-01 13:05:00'),
(6, 'Kurir3', 'TRK20251201-005', 'in_transit', '2025-12-01 13:45:00'),
(8, 'Kurir3', 'TRK20251201-006', 'in_transit', '2025-12-01 14:10:00'),
(9, 'Kurir1', 'TRK20251201-007', 'ready', '2025-12-01 14:30:00'),
(10, 'Kurir2', 'TRK20251201-008', 'ready', '2025-12-01 14:50:00');

3.8. Stock Change
INSERT INTO stock_change (menu_id, change_qty, reason, change_date, user_id) VALUES
(1, 20, 'Restok awal Bakso Urat Jumbo', '2025-12-01 08:00:00', 11),
(2, 15, 'Restok Bakso Mercon', '2025-12-01 08:10:00', 11),
-- tambahkan sisanya sesuai skrip awal
;

4. Trigger
DELIMITER $$
CREATE TRIGGER tr_update_total_order
AFTER INSERT ON order_item
FOR EACH ROW
BEGIN
    UPDATE `order`
    SET total_price = total_price + (NEW.unit_price * NEW.qty)
    WHERE order_id = NEW.order_id;
END$$
DELIMITER ;

5. Procedure
DELIMITER $$
CREATE PROCEDURE laporan_pengiriman(IN tanggal DATE)
BEGIN
    SELECT d.delivery_id,
           o.order_id,
           c.name,
           d.kurir,
           d.status
    FROM delivery d
    JOIN `order` o ON d.order_id = o.order_id
    JOIN customer c ON o.customer_id = c.customer_id
    WHERE DATE(d.send_date) = tanggal;
END$$
DELIMITER ;

6. View
CREATE VIEW view_pembayaran AS
SELECT o.order_id, c.name, p.metode, p.amount, p.status
FROM payment p
JOIN `order` o ON p.order_id = o.order_id
JOIN customer c ON o.customer_id = c.customer_id;
