CREATE TABLE suppliers (
    supplier_id NUMBER PRIMARY KEY,
    supplier_name VARCHAR2(100) NOT NULL,
    contact VARCHAR2(100)
);

CREATE TABLE products (
    product_id NUMBER PRIMARY KEY,
    product_name VARCHAR2(100) NOT NULL,
    price NUMBER(10,2),
    stock_quantity NUMBER DEFAULT 0,
    supplier_id NUMBER,
    CONSTRAINT fk_supplier
        FOREIGN KEY (supplier_id)
        REFERENCES suppliers(supplier_id)
);

CREATE TABLE orders (
    order_id NUMBER PRIMARY KEY,
    product_id NUMBER,
    quantity NUMBER,
    order_date DATE DEFAULT SYSDATE,
    CONSTRAINT fk_product
        FOREIGN KEY (product_id)
        REFERENCES products(product_id)
);



CREATE OR REPLACE TRIGGER check_stock
BEFORE UPDATE OF stock_quantity ON products
FOR EACH ROW
BEGIN
    IF :NEW.stock_quantity < 0 THEN
        RAISE_APPLICATION_ERROR(-20002, 'Stock cannot be negative');
    END IF;
END;
/



CREATE OR REPLACE PROCEDURE add_supplier (
    p_id NUMBER,
    p_name VARCHAR2,
    p_contact VARCHAR2
)
IS
BEGIN
    INSERT INTO suppliers VALUES (p_id, p_name, p_contact);
    COMMIT;
END;
/


CREATE OR REPLACE PROCEDURE add_product (
    p_id NUMBER,
    p_name VARCHAR2,
    p_price NUMBER,
    p_stock NUMBER,
    p_supplier NUMBER
)
IS
BEGIN
    INSERT INTO products
    VALUES (p_id, p_name, p_price, p_stock, p_supplier);
    COMMIT;
END;
/


CREATE OR REPLACE PROCEDURE update_stock (
    p_id NUMBER,
    p_quantity NUMBER
)
IS
BEGIN
    UPDATE products
    SET stock_quantity = stock_quantity + p_quantity
    WHERE product_id = p_id;

    COMMIT;
END;
/


CREATE OR REPLACE PROCEDURE sell_product (
    p_order_id NUMBER,
    p_product_id NUMBER,
    p_quantity NUMBER
)
IS
    v_stock NUMBER;
BEGIN
    SELECT stock_quantity INTO v_stock
    FROM products
    WHERE product_id = p_product_id
    FOR UPDATE;

    IF v_stock < p_quantity THEN
        RAISE_APPLICATION_ERROR(-20001, 'Not enough stock available');
    END IF;

    INSERT INTO orders(order_id, product_id, quantity)
    VALUES (p_order_id, p_product_id, p_quantity);

    UPDATE products
    SET stock_quantity = stock_quantity - p_quantity
    WHERE product_id = p_product_id;

    COMMIT;
END;
/


BEGIN
    add_supplier(1, 'ABC Supplier', 'abc@email.com');
    add_product(101, 'Laptop', 50000, 10, 1);
    add_product(102, 'Mouse', 500, 50, 1);
END;
/


BEGIN
    update_stock(101, 5);
END;
/


BEGIN
    sell_product(1, 101, 2);
END;
/

SELECT * FROM suppliers;
SELECT * FROM products;
SELECT * FROM orders;


