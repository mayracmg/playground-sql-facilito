# SQL Facilito

En esta sección encontrarás ejemplos de consultas SQL, clasificas en los 3 tipos que existen, DDL, DCL y DML.
Los queries estan preparados para funcionar en una base de datos **MySQL**, puedes copiar los queries y ejecutarlos en tu propia base de datos.

Vamos a crear una base de datos llamada Autenticación, la cual tendra el siguiente diagrama ER (Entidad Relacion).

![ER](https://raw.githubusercontent.com/mayracmg/playground-sql-facilito/master/markdowns/Autenticacion%20ER.png)

::: DDL (Data Definition Language)
Creación de una base de datos llamada Autenticación.

```sql
CREATE DATABASE Autenticacion;
```

## Creación de las tablas del diagrama ER
::: Scripts
La tabla **Usuario** tendrá como llave primaria el campo ID, los campos ID y Nombre tendran un **constraint** de tipo **NOT NULL** el cual sirve para garantizar que esos campos siempre tendrán valor.

La sentencia **AUTO_INCREMENT** sirve para indicar que el campo ID, será un correlativo automatico, manejado por la base de datos, es decir no es necesario especificar el valor cuando se inserten valores.

La sentencia **PRIMARY KEY** sirve para indicar que campo o campos serán utilizados como llave primaria.

+ El tipo de dato INT permite almacenar números desde -2147483648 hasta 2147483647.
+ El tipo de dato VARCHAR permite almacenar una cadena de caracteres de longitud 50 y/o 100 segun se especifique.
+ El tipo de dato DATE permite almacenar fechas (sin hora).
+ El tipo de dato TINYINT permite almacenar números desde -128 hasta 127.
+ **ALTER TABLE** permite modificar la tabla Usuario y agregarle un contraint del tipo **UNIQUE** de esa forma, los valores almacenados en el campo Email no podrán repetirse.
```sql
CREATE TABLE Usuario (
	ID INT NOT NULL AUTO_INCREMENT, 
	Nombre VARCHAR(50) NOT NULL, 
	Apellido VARCHAR(50), 
	Email VARCHAR(100), 
	Fecha_Nacimiento DATE,
	Activo TINYINT,
	PRIMARY KEY (ID)
);
ALTER TABLE Usuario ADD CONSTRAINT UK_Email UNIQUE (Email);
```
___
La tabla **Historial_Conexion** tendrá como llave primaria el campo ID, los campos ID, ID_Usuario y Fecha_Hora tendrán un **constraint** de tipo **NOT NULL** el cual sirve para garantizar que esos campos siempre tendrán valor.

La sentencia **AUTO_INCREMENT** sirve para indicar que el campo ID, será un correlativo automatico, manejado por la base de datos, es decir no es necesario especificar el valor cuando se inserten valores.

La sentencia **PRIMARY KEY** sirve para indicar que campo o campos serán utilizados como llave primaria.

La sentencia **FOREIGN KEY** sirve parea indicar que campo o campos será uitilizados como llave foranea o referencia a otra tabla. El campo ID_Usuario corresponde al campo ID de la tabla Usuario.
+ El tipo de dato DATETIME permite almacenar fecha y hora.
```sql
CREATE TABLE Historial_Conexion (
	ID INT NOT NULL AUTO_INCREMENT,
	ID_Usuario INT NOT NULL,
	Fecha_Hora DATETIME NOT NULL,
	IP VARCHAR(15), 
	Navegador VARCHAR(50), 
	PRIMARY KEY (ID),
	FOREIGN KEY (ID_Usuario) REFERENCES Usuario(ID)
);
```
---
La tabla **Menu** tendrá como llave primaria el campo ID, los campos ID, Titulo y Activo tendrán un **constraint** de tipo **NOT NULL** el cual sirve para garantizar que esos campos siempre tendrán valor.

El campo Activo tendrá un **contraint** de tipo **DEFAULT** el cúal asignará el valor 0 de forma predeterminada cuando en la instrucción insert no se especifique el campo Activo. Ademas el campo Activa tendrá un **CHECK CONTRAINT** el cúal solo permitirá que el campo acepte los valores 0 o 1, ningún otro valor podrá ser almacenado en ese campo.

La sentencia **AUTO_INCREMENT** sirve para indicar que el campo ID, será un correlativo automatico, manejado por la base de datos, es decir no es necesario especificar el valor cuando se inserten valores.

La sentencia **PRIMARY KEY** sirve para indicar que campo o campos serán utilizados como llave primaria.

```sql
CREATE TABLE Menu (
	ID INT NOT NULL AUTO_INCREMENT,
	Titulo VARCHAR(50) NOT NULL, 
	Descripcion VARCHAR(150), 
	URL VARCHAR(150), 
	Activo TINYINT DEFAULT 0 NOT NULL,
	PRIMARY KEY (ID),
	CONSTRAINT CHK_Menu_Activo CHECK (Activo IN (0, 1))
);
```
---
El script de creación de la tabla **Menu_Usuario** no contiene ningun contraint, ni llave primaria y el tipo de dato DATE "es incorrecto". La tabla será modificada con scripts separados para llegar a la estructura correcta.

+ La sentencia **ALTER TABLE** permite modificar la estructura de la tabla Menu_Usuario.
+ **MODIFY** permite modificar el tipo de dato del campo Fecha_Habilitado.
+ **ADD CONSTRAINT** permite agregar la llave primaria a la tabla, entre los parentesis se agregan los 2 campos ID_Usuario y ID_Menu, que juntos forman la llave primaria.
+ **ADD FOREIGN KEY** permite agregar la llave foranea a la tabla, se crean dos queries debido a que son dos refencias a dos tablas disintas.

```sql
CREATE TABLE Menu_Usuario (
	ID_Usuario INT,
	ID_Menu INT,
	Fecha_Habilitado DATE
);

ALTER TABLE Menu_Usuario MODIFY Fecha_Habilitado DATETIME;
ALTER TABLE Menu_Usuario ADD CONSTRAINT PK_Menu_Usuario PRIMARY KEY (ID_Usuario, ID_Menu);
ALTER TABLE Menu_Usuario ADD FOREIGN KEY (ID_Usuario) REFERENCES Usuario(ID);
ALTER TABLE Menu_Usuario ADD FOREIGN KEY (ID_Menu) REFERENCES Menu(ID);
```
[Ver Diagrama ER con scripts](https://github.com/mayracmg/playground-sql-facilito/blob/f5753c08814bb14e3ecf1db49f46995fde1dbcc9/markdowns/ER%20con%20Scripts.png)
:::

## Creación de otros objetos
::: Triggers
Script para la _creación_ de un _trigger_ el cual va a ejecutar automaticamente justo _antes de insertar_ un registro en la _tabla Usuario_, va a verificar si en el registro que se esta insertado, el campo Email esta vacio (NULL) o no, si esta vacio entonces va a asignarle un estado inactivo (Activo = 0).
+ **DELIMITER** Sirve para indicarle a MySQL donde finaliza el bloque de código.
```sql
DELIMITER $$

CREATE TRIGGER BI_Usuario
    BEFORE INSERT ON Usuario 
	FOR EACH ROW
BEGIN
    IF (NEW.Email IS NULL) THEN
		SET NEW.Activo = 0;
	ELSE
		SET NEW.Activo = 1;
	END IF;
END$$
```

El funcionamento del trigger puede ser probado con la ejecución de estos scripts. Al no especificar el campo Email, el trigger va a asignar un valor 0 al campo Activo. Cuando el campo Email si se especifica entonces el campo Activo tendrá valor 1.
```sql
INSERT INTO Usuario (Nombre, Apellido)
VALUES ('Tu_Nombre', 'Tu_Apellido')

INSERT INTO Usuario (Nombre, Apellido, Email)
VALUES ('Tu_Nombre', 'Tu_Apellido', 'nombre@gmail.com')
```
Luego se comprobar el resultado con un query **SELECT** en la tabla Usuario.
```sql
SELECT *
FROM Usuario;
```
El cual generará un resultado similar al siguiente:

| ID | Nombre | Apellido | Email | Fecha_Nacimiento | Activo |
| ------ | ----------- | ----------- | ----------- | ----------- | ----------- |
| 1   | Tu_Nombre | Tu_Apellido | NULL | NULL | 0 |
| 2 | Tu_Nombre | Tu_Apellido | nombre@gmail.com | NULL | 1 |

:::

::: Funciones

Script para la _creación_ de una _función_ la cual va a concatenar el nombre y apellido del usuario en un solo campo y lo convertirá a mayusculas. El _parametro de entrada_ es el ID de la tabla Usuario. 

La función podrá ser invocada desde cualquier consulta DML, por ejemplo una consulta <span style="color:blue">SELECT</span>.

+ **DELIMITER** Sirve para indicarle a MySQL donde finaliza el bloque de código.
+ **DROP FUNCTION IF EXISTS** borrará la función si es que ya existe, para luego crearla nuevamente. Sin esta instrucción, el script **CREATE FUNCTION** solo funcionaría una vez y para modificaciones posteriores sería necesario utilizar un **ALTER FUNCTION**.
+ **RETURNS VARCHAR(101)** indica que el tipo de dato del resultado de la función será una cadena de texto de longitud maxima 101 caracteres.
+ **READS SQL DATA** sin esta instrucción la instrucción <span style="color:blue">SELECT</span> daría error.
+ El parametro de entrada va dentro de parentesis justo despues del nombre la función.
+ **DECLARE** permite declarar la variable que será utilizada para almacenar el nombre completo del usuario.
+ **CONCAT** concatena el primer nombre, un espacio en blanco y el appelido del usuario en un solo campo.
+ **UPPER** convierte el resultado de la concatenación en mayusculas.
+ **INTO** permiate almacenar el resultado del select en la variable declarada.
+ **RETURN** la última instrucción de una función debe ser devolver el resultado generado.
```sql
DELIMITER $$

DROP FUNCTION IF EXISTS Fn_Nombre_Completo;
CREATE FUNCTION Fn_Nombre_Completo(
    ID_Usuario INT
)
RETURNS VARCHAR(101)
READS SQL DATA
BEGIN
	DECLARE Nombre_Completo VARCHAR(101);
	
	SELECT UPPER(CONCAT(Nombre, ' ', Apellido))
	INTO Nombre_Completo
	FROM Usuario
	WHERE ID = ID_Usuario;

	RETURN Nombre_Completo;
END $$
```
En el siguiente ejemplo, la función es invocada desde la instrucción <span style="color:blue">SELECT</span> y el resultado se verá desplegado como cualquier campo.
+ _Nombre_Completo_ es un alias, los alias son opcionales, pero mejoran la visualización de los datos.
```sql
SELECT ID, Fn_Nombre_Completo(id) Nombre_Completo
FROM Usuario;
```
El cual generará un resultado similar al siguiente:

| ID | Nombre_Completo |
| ------ | ----------- |
| 1   | MAY CODE |
:::

::: Procedimientos (Stored Procedures)
Script para la creación de un procedimiento, parentesis vacios indica que no recibe parametros. Cada vez que se ejecute, leerá la tabla _Historial_conexion_, filtará las filas que tengan el campo _IP_ con valor _NULL_ y les asignará el valor '0.0.0.0'.
+ **DELIMITER** Sirve para indicarle a MySQL donde finaliza el bloque de código.
+ **DROP PROCEDURE IF EXISTS** borrará el procedimiento si es que ya existe, para luego crearla nuevamente. Sin esta instrucción, el script **CREATE PROCEDURE** solo funcionaría una vez y para modificaciones posteriores sería necesario utilizar un **ALTER PROCEDURE**.

```sql
DELIMITER //

DROP PROCEDURE IF EXISTS SP_Actualizar_IP_Vacia;
CREATE PROCEDURE SP_Actualizar_IP_Vacia()
BEGIN
	UPDATE Historial_conexion
	SET IP = '0.0.0.0'
	WHERE IP IS NULL;
END //
```
+ **CALL** ejecuta el procedimiento, entre los parentesis se especificarían los parametros, si los hubiera.

```sql
CALL SP_Actualizar_IP_Vacia();
```
:::
:::

::: DCL (Data Control Language)

Creación de 3 usuarios, el proposito del usuario _WebSite_ es ser utilizado desde un API o sistema que se conecte a la base de datos.
El proposito de los usuarios _Developer1_ y _Developer2_ es ser utilizados por 2 personas distintas y de acuerdo a sus responsabilidades serán asignados sus permisos.
*CREATE USER* son sentencias DDL, es necesario que existan usuarios previo a asignarles permisos.
```sql
CREATE USER WebSite IDENTIFIED BY 'Contraseña#1';
CREATE USER Developer1 IDENTIFIED BY 'Contraseña#2';
CREATE USER Developer2 IDENTIFIED BY 'Contraseña#3';
```

## Asignación de permisos:
+ Al usuario _WebSite_, el usuario tendrá todos los permisos sobre las tablas, excepto la tabla _Usuario_, sobre la tabla _Usuario_ no podrá eliminar usuario.
+ Al usuario _Developer1_ se la asignará todos los permisos sobre las tablas _Menu_ y _Menu_Usuario_, sobre las tablas _Usuario_ y _Historial_Conexion_ solo tendrá permiso de lectura, para evitar que Developer1 haga modificaciones sobre esas tablas.
+ Al usuario _Developer2_ solo se le asignará permiso de lectura sobre las tablas _Menu_ y _Menu_Usuario_, sobre las tablas _Usuario_ y _Historial_Conexion_ no tendrá ningun acceso.

+ <span style="color:blue">GRANT</span> sentencia para otorgar permisos.
+ <span style="color:blue">SELECT</span> para asignar permiso de lectura sobre la tabla especificada.
+ <span style="color:blue">INSERT</span> para asignar permiso de inserción sobre la tabla especificada.
+ <span style="color:blue">UPDATE</span> para asignar permiso de actualización sobre la tabla especificada.
+ <span style="color:blue">DELETE</span> para asignar permiso de eliminación sobre la tabla especificada.
+ <span style="color:blue">ON</span> luego de este texto se indica el nombre de la tabla a la cual se le quiere asignar los permisos.
+ <span style="color:blue">TO</span> luego de este texto se indica el nombre del usuario al cual se le quiere asignar los permisos.

```sql
GRANT SELECT, INSERT, UPDATE ON Usuario TO WebSite;
GRANT SELECT, INSERT, UPDATE, DELETE ON Historial_Conexion TO WebSite;
GRANT SELECT, INSERT, UPDATE, DELETE ON Menu TO WebSite;
GRANT SELECT, INSERT, UPDATE, DELETE ON Menu_Usuario TO WebSite;

GRANT SELECT ON Usuario TO Developer1;
GRANT SELECT ON Historial_Conexion TO Developer1;
GRANT SELECT, INSERT, UPDATE, DELETE ON Menu TO Developer1;
GRANT SELECT, INSERT, UPDATE, DELETE ON Menu_Usuario TO Developer1;

GRANT SELECT ON Menu TO Developer2;
GRANT SELECT ON Menu_Usuario TO Developer2;
```

+ Al usuario _WebSiste_ se le quitará el acceso de eliminar sobre la tabla _Menu_.

+ <span style="color:blue">REVOKE</span> sentencia para revocar o quitar permisos.
+ <span style="color:blue">SELECT</span> para quitar permiso de lectura sobre la tabla especificada.
+ <span style="color:blue">INSERT</span> para quitar permiso de inserción sobre la tabla especificada.
+ <span style="color:blue">UPDATE</span> para quitar permiso de actualización sobre la tabla especificada.
+ <span style="color:blue">DELETE</span> para quitar permiso de eliminación sobre la tabla especificada.
+ <span style="color:blue">ON</span> luego de este texto se indica el nombre de la tabla a la cual se le quiere quitar los permisos.
+ <span style="color:blue">FROM</span> luego de este texto se indica el nombre del usuario al cual se le quiere quitar los permisos.
```sql
REVOKE DELETE ON Menu FROM WebSite;
```
:::

::: DML (Data Manipulation Language) 
::: SELECTS

Queries que permiten leer todos los campos para cada una de la tablas creadas previamente.
+ <span style="color:blue">SELECT *</span> Indica que se leerán todos los campos de la tabla.
+ <span style="color:blue">FROM</span> Luego de este texto se especifica el nombre de la tabla (o tablas) a leer.

```sql
SELECT *
FROM Usuario;

SELECT *
FROM Historial_Conexion;

SELECT *
FROM Menu;

SELECT *
FROM Menu_Usuario;
```

+ <span style="color:blue">SELECT Nombre, Apellido</span> para leer unicamente los campos Nombre y Apellido. Los campos se separan por comas.

```sql
SELECT Nombre, Apellido
FROM Usuario;
```

+ <span style="color:blue">Nombre Primer_Nombre</span> _Nombre_ es el nombre del campo en la tabla, _Primer_Nombre es un *alias* para el campo, los alias es otra forma de referise al campo, son opcionales.
+ <span style="color:blue">Usuario U</span> La letra _U_ es un alias para la tabla _Usuario_, los alias son muy comunes cuando en una misma consulta hay varias tablas.
+ <span style="color:blue">WHERE</span> Aca se especifican los filtros, para solo obtener las filas que cumplan las condiciones especificadas.
+ **Activo = 1** para filtrar solo las filas que en la columna _Activo_ tengan valor igual a 1.
+ <span style="color:blue">AND</span> para unir varios filtros dentro de la clausula <span style="color:blue">WHERE</span>.
+ **Fecha_Nacimiento IS NULL** para filtrar solo las filas que en la columna Fecha_Nacimiento tengan valor igual a _NULL_.

```sql
SELECT Nombre Primer_Nombre, Apellido
FROM Usuario U
WHERE Activo = 1
AND Fecha_Nacimiento IS NULL;
```

+ <span style="color:blue">ORDER BY</span> Luego de este texto se especifica el nombre del campo (o campos) para aplicar el ordenamiento.
Se leer el hsitorial de conexion ordenados por dirección IP de forma ascendente.
```sql
SELECT *
FROM Historial_Conexion
ORDER BY IP;
```

+ <span style="color:blue">DESC</span> Indica que el ordenamiento será descendente.
+ <span style="color:blue">ASC</span> Indica que el ordenamiento será ascendente, si no se especifica **ASC** o **DESC**, por default el ordenamiento será ascendente.
Se leer el hsitorial de conexion ordenados por Fecha y Hora de forma descendente.
```sql
SELECT *
FROM Historial_Conexion
ORDER BY Fecha_Hora DESC;
```
:::
:::
