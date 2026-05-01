# Sistema de Gestión "Mundo Llantas & Aros"
Sistema web integral para el control de asistencia biométrica, gestión de inventarios y facturación electrónica.
Desarrollado como solución tecnológica para modernizar la operatividad del sector automotriz en Pucallpa

## Descripción del Negocio
Nombre: Mundo Llantas & Aros Pucallpa.
Giro: Comercialización de neumáticos y aros, con servicios técnicos de balanceo, alineamiento y reparación.
Tamaño: Empresa con personal técnico especializado en diversas áreas operativas.
Contexto: Actualmente operan con registros manuales en papel, lo que genera errores, pérdida de datos y riesgos de suplantación.
Justificación: Se requiere digitalizar el control de personal y el stock para evitar retrasos por medidas incorrectas de llantas y falta de veracidad en la asistencia

## Identificar el Problema y Solución

Problema: Gestión manual de entradas/salidas, errores en el conteo de inventario y
lentitud en el despacho de productos hacia los clientes
Solución Tecnológica: Implementación de un Software ERP integrado con un lector
biométrico ZKTeco y dispositivos PDA para el escaneo de códigos de barras/QR

## Requerimientos Funcionales

| Codigo | Descripcion |
|---|---|
| RF01 |El sistema debe capturar la huella dactilar (ZKTeco) para marcar entrada, salida y refrigerio|
| RF02 | Cada venta realizada debe descontar automáticamente el stock del inventario  |
| RF03 | Generación de boletas y facturas electrónicas (PDF/XML) vinculadas a SUNAT|
| RF04 | El administrador debe poder justificar tardanzas o faltas de forma manual|
| RF05 |Emisión de reportes: ventas diarias, ranking de productos y récord de asistencia|


## Requerimientos No Funcionales

| Codigo | Tipo | Descripcion |
|---|---|---|
| RNF01 | Seguridad | Impedir acceso no autorizado mediante roles (Vendedor, Almacenero, AdminImpedir acceso no autorizado mediante roles (Vendedor, Almacenero, Admin)|
| RNF02 | Disponibilidad  | Sistema basado en la nube (Cloud) accesible 24/7 vía web o móvil |
| RNF03 |Integridad | Los registros de huellas no deben ser alterables por los trabajadores |
| RNF04 |Rendimiento | La validación de huella y emisión de boletas debe tardar entre 3 a 5 segundos |

----

## Base de datos
El sistema cuenta con 3 tablas principales identificadas 

|Tabla |Descripción |
|---|---|
|ASISTENCIA | Registro de marcas biométricas (entrada/salida) de los trabajadores |
|INVENTARIO_VENTAS | Control de stock de llantas/aros y registro de transacciones comerciales |
|USUARIO_TURNO | Gestión de accesos, roles de usuario y horarios asignados | 


## Diagrama Entidad - Relación (DER)

   sistema mundo_llantas_aros_pucallpa
   
   <img width="700" height="367" alt="image" src="https://github.com/user-attachments/assets/92979d69-1ae4-49dd-87b4-2aff5cfbe8e0" />



## Modelo Relacional (MR)

MODELO RELACIONAL- sistema mundo_llantas_aros_pucallpa
<img width="708" height="514" alt="image" src="https://github.com/user-attachments/assets/05e3e28e-c1e5-485d-bd6d-b00a2084fea0" />



## Cardinalidades

USUARIO_TURNO — VENTA_FACTURACION (1:N)
Un usuario puede realizar muchas ventas, pero una venta es realizada por un solo usuario.
INVENTARIO_PRODUCTO — VENTA_FACTURACION (1:N)
Un producto puede estar en muchas ventas, pero una venta registra un solo producto.
USUARIO_TURNO — ASISTENCIA (1:N)
Un usuario puede tener muchas asistencias, pero una asistencia pertenece a un solo usuario.

<img width="653" height="200" alt="image" src="https://github.com/user-attachments/assets/0c6db399-d2f6-4dee-aecd-b600390e17b5" />


# Base de datos- De mundo_llantas_y_aros_Pucallpa

El sistema cuenta con 4 tablas principales:


```sql
CREATE DATABASE IF NOT EXISTS mundo_llantas_pucallpa;

USE mundo_llantas_pucallpa;



-- Tabla usuario_turno

CREATE TABLE usuario_turno (

  usuario_id INT AUTO_INCREMENT PRIMARY KEY,

  nombre VARCHAR(100) NOT NULL,

  apellido VARCHAR(100) NOT NULL,

  dni VARCHAR(8) NOT NULL UNIQUE,

  rol ENUM('Admin', 'Vendedor', 'Almacenero', 'Tecnico') NOT NULL,

  telefono VARCHAR(15),

  email VARCHAR(100),

  contrasena VARCHAR(255) NOT NULL,

  activo TINYINT(1) NOT NULL DEFAULT 1

);



-- Tabla asistencia

CREATE TABLE asistencia (

  asistencia_id INT AUTO_INCREMENT PRIMARY KEY,

  usuario_id INT NOT NULL,

  fecha DATE NOT NULL,

  hora_entrada TIME NOT NULL,

  hora_refrigerio TIME,

  hora_salida TIME,

  justificacion TEXT,

  FOREIGN KEY (usuario_id) REFERENCES usuario_turno(usuario_id),

  UNIQUE KEY uq_usuario_fecha (usuario_id, fecha)

);



-- Tabla inventario_producto

CREATE TABLE inventario_producto (

  producto_id INT AUTO_INCREMENT PRIMARY KEY,

  descripcion VARCHAR(255) NOT NULL,

  categoria ENUM('Llanta', 'Aro') NOT NULL,

  medida VARCHAR(50) NOT NULL,

  marca VARCHAR(100),

  stock_actual INT NOT NULL DEFAULT 0,

  precio_unitario DECIMAL(10,2) NOT NULL

);



-- Tabla venta_facturacion

CREATE TABLE venta_facturacion (

  venta_id INT AUTO_INCREMENT PRIMARY KEY,

  usuario_id INT NOT NULL,

  producto_id INT NOT NULL,

  fecha_venta DATETIME DEFAULT CURRENT_TIMESTAMP,

  cantidad INT NOT NULL,

  precio_venta DECIMAL(10,2) NOT NULL,

  tipo_comprobante ENUM('Boleta', 'Factura') NOT NULL,

  codigo_sunat_xml VARCHAR(255),

  FOREIGN KEY (usuario_id) REFERENCES usuario_turno(usuario_id),

  FOREIGN KEY (producto_id) REFERENCES inventario_producto(producto_id)

);





-- Trigger para descontar stock al registrar una venta

DELIMITER $$



CREATE TRIGGER trg_descontar_stock

AFTER INSERT ON venta_facturacion

FOR EACH ROW

BEGIN



  UPDATE inventario_producto

  SET  stock_actual = stock_actual - NEW.cantidad

  WHERE producto_id = NEW.producto_id;



END$$



DELIMITER ;



-- Datos de ejemplo: usuario_turno

INSERT INTO usuario_turno (nombre, apellido, dni, rol, telefono, email, contrasena, activo) VALUES

('Carlos', 'Ramirez', '12345678', 'Admin',   '987654321', 'carlos@gmail.com', '1234', 1),

('Lucia',  'Torres', '87654321', 'Vendedor',  '912345678', 'lucia@gmail.com', '1234', 1),

('Miguel', 'Flores', '11223344', 'Almacenero', '923456789', 'miguel@gmail.com', '1234', 1),

('Ana',   'Chavez', '44332211', 'Tecnico',  '934567890', 'ana@gmail.com',  '1234', 1);



-- Datos de ejemplo: inventario_producto

INSERT INTO inventario_producto (descripcion, categoria, medida, marca, stock_actual, precio_unitario) VALUES

('Llanta EfficientGrip',   'Llanta', '205/55R16', 'Goodyear', 40, 350.00),

('Aro de Lujo Deportivo',   'Aro',  'R17',    'Nabil',  12, 450.00),

('Llanta Adventure AT',    'Llanta', '265/70R16', 'Goodyear', 20, 580.00),

('Aro de Repuesto Reforzado', 'Aro',  'R15',    'Generico', 15, 220.00),

('Llanta Cargo G32',     'Llanta', '195/70R15', 'Goodyear', 30, 410.00);



-- Datos de ejemplo: asistencia

INSERT INTO asistencia (usuario_id, fecha, hora_entrada, hora_refrigerio, hora_salida) VALUES

(1, '2025-04-01', '08:00:00', '13:00:00', '17:00:00'),

(2, '2025-04-01', '08:05:00', '13:00:00', '17:00:00'),

(3, '2025-04-01', '07:55:00', '13:00:00', '17:00:00'),

(4, '2025-04-01', '08:10:00', '13:00:00', '17:00:00');



-- Datos de ejemplo: venta_facturacion

INSERT INTO venta_facturacion (usuario_id, producto_id, cantidad, precio_venta, tipo_comprobante) VALUES

(2, 1, 2, 350.00, 'Boleta'),

(2, 3, 1, 580.00, 'Factura'),

(1, 5, 4, 410.00, 'Boleta');

  

 select * from usuario_turno



## FIGMA

https://www.figma.com/design/HbDLsjrG3iVDceRRa4rj2f/Sin-t%C3%ADtulo?node-id=0-1&p=f&t=CLjCuSuJsNpFjcd8-0


## Imagen del Negocio

![Mundo Llantas y Aros](img/negocio.png)

  



