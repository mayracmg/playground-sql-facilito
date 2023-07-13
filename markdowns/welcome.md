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
Ver Diagrama ER con scripts.
https://github.com/mayracmg/playground-sql-facilito/blob/f5753c08814bb14e3ecf1db49f46995fde1dbcc9/markdowns/ER%20con%20Scripts.png
:::

Script para la **creación** de un **trigger** el cual va a ejecutar automaticamente justo **antes de insertar** un registro en la **tabla Usuario**, va a verificar si en el registro que se esta insertado, el campo Email esta vacio (NULL) o no, si esta vacio entonces va a asignarle un estado inactivo (Activo = 0).
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
:::

::: DCL (Data Control Language)

Creación de una base de datos llamada Autenticación.

```sql
CREATE DATABASE Autenticacion;
```


:::

::: DML (Data Manipulation Language) 
Creación de una base de datos llamada Autenticación.

```sql
CREATE DATABASE Autenticacion;
```
:::
