## **🔹 Caso 1: Control de Stock de Productos**

### **Escenario:**

Una tienda en línea necesita asegurarse de que los clientes no puedan comprar más unidades de un producto del stock disponible. Si intentan hacerlo, la compra debe **bloquearse**.

### **Tarea:**

1. Crear las tablas `productos` y `ventas`.

2. Implementar un trigger `BEFORE INSERT` para evitar ventas con cantidad mayor al stock disponible.

3. Probar el trigger.

   

```sql
DELIMITER //

CREATE TRIGGER verificar_stock
BEFORE INSERT ON ventas
FOR EACH ROW
BEGIN
    DECLARE stock_actual INT;

    SELECT stock INTO stock_actual
    FROM productos
    WHERE id = NEW.id_producto;

    IF NEW.cantidad > stock_actual THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'No hay suficiente stock para realizar la venta';
    END IF;
END;
//

DELIMITER ;

mysql>  insert into ventas (id_producto, cantidad) VALUES (1, 10);
Query OK, 1 row affected (0,00 sec)

mysql> insert into ventas (id_producto, cantidad) VALUES (1, 11);
ERROR 1644 (45000): No hay suficiente stock para realizar la venta

```

## 

## **🔹 Caso 2: Registro Automático de Cambios en Salarios**

### **Escenario:**

La empresa **TechCorp** desea mantener un registro histórico de todos los cambios de salario de sus empleados.

### **Tarea:**

1. Crear las tablas `empleados` y `historial_salarios`.
2. Implementar un trZ|Zigger `BEFORE UPDATE` que registre cualquier cambio en el salario.
3. Probar el trigger.

```sql
CREATE TABLE empleados (
    id INT PRIMARY KEY AUTO_INCREMENT,
    nombre VARCHAR(50),
    salario DECIMAL(10,2)
);

CREATE TABLE historial_salarios (
    id INT PRIMARY KEY AUTO_INCREMENT,
    id_empleado INT,
    salario_anterior DECIMAL(10,2),
    salario_nuevo DECIMAL(10,2),
    fecha TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (id_empleado) REFERENCES empleados(id)
);

DELIMITER //

CREATE TRIGGER before_salario_update
BEFORE UPDATE ON empleados
FOR EACH ROW
BEGIN
    INSERT INTO historial_salarios (id_empleado, salario_anterior, salario_nuevo)
    VALUES (OLD.id, OLD.salario, NEW.salario);
END //

DELIMITER ;
INSERT INTO empleados (nombre, salario) VALUES ('Carlos López', 2500.00);
UPDATE empleados SET salario = 3000.00 WHERE id = 1;
SELECT * FROM historial_salarios;
```

## 

## **🔹 Caso 3: Registro de Eliminaciones en Auditoría**

### **Escenario:**

La empresa **DataSecure** quiere registrar toda eliminación de clientes en una tabla de auditoría para evitar pérdidas accidentales de datos.

### **Tarea:**

1. Crear las tablas `clientes` y `clientes_auditoria`.
2. Implementar un trigger `AFTER DELETE` para registrar los clientes eliminados.
3. Probar el trigger.

```sql
CREATE TABLE clientes (
    id INT PRIMARY KEY AUTO_INCREMENT,
    nombre VARCHAR(50),
    email VARCHAR(50)
);

CREATE TABLE clientes_auditoria (
    id INT PRIMARY KEY AUTO_INCREMENT,
    id_cliente INT,
    nombre VARCHAR(50),
    email VARCHAR(50),
    fecha_eliminacion TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
DELIMITER //

CREATE TRIGGER auditar_eliminacion_cliente
AFTER DELETE ON clientes
FOR EACH ROW
BEGIN
    INSERT INTO clientes_auditoria (id_cliente, nombre, email)
    VALUES (OLD.id, OLD.nombre, OLD.email);
END;
//

DELIMITER ;

 INSERT INTO clientes (nombre, email) VALUES
    -> ('Ana López', 'ana@example.com'),
    -> ('Carlos Díaz', 'carlos@example.com');

DELETE FROM clientes WHERE id = 1;

SELECT * FROM clientes_auditoria;
+----+------------+------------+-----------------+---------------------+
| id | id_cliente | nombre     | email           | fecha_eliminacion   |
+----+------------+------------+-----------------+---------------------+
|  1 |          1 | Ana López  | ana@example.com | 2025-07-08 13:11:29 |
+----+------------+------------+-----------------+---------------------+
1 row in set (0,01 sec)


```

## 



##  Caso 4: Restricción de Eliminación de Pedidos Pendientes**

### **Escenario:**

En un sistema de ventas, no se debe permitir eliminar pedidos que aún están **pendientes**.

### **Tarea:**

1. Crear las tablas `pedidos`.
2. Implementar un trigger `BEFORE DELETE` para evitar la eliminación de pedidos pendientes.
3. Probar el trigger.

```sql
CREATE TABLE pedidos (
    id INT PRIMARY KEY AUTO_INCREMENT,
    cliente VARCHAR(100),
    estado ENUM('pendiente', 'completado')
);

DELIMITER //

CREATE TRIGGER no_eliminacion_pendiente
BEFORE DELETE ON pedidos
FOR EACH ROW
BEGIN
IF OLD.estado = 'pendiente' THEN 
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'No hacer la eliminacion del pedido';
    END IF;
   
END //

DELIMITER ;

INSERT INTO pedidos (cliente, estado) VALUES
('Juan Pérez', 'pendiente'),
('Laura Gómez', 'completado'),
('Andrés Martínez', 'pendiente'),
('Sofía Ramírez', 'completado'),
('Carlos Torres', 'pendiente'),
('Valentina Suárez', 'completado'),
('Luis Mendoza', 'pendiente'),
('Camila Herrera', 'completado');
	
DELETE from pedidos WHERE id = 3;
ERROR 1644 (45000): No hacer la eliminacion del pedido

DELETE from pedidos WHERE id = 2;
Query OK, 1 row affected (0,00 sec)


```
