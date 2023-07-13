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

##Script para creación de las tablas del diagrama ER.
::: 
La tabla **Usuario** tendrá como llave primaria el campo ID, los campos ID y Nombre tendran un **constraint** de tipo **NOT NULL** el cual sirve para garantizar que esos campos siempre tendrán valor.

La sentencia **AUTO_INCREMENT** sirve para indicar que el campo ID, será un correlativo automatico, manejado por la base de datos, es decir no es necesario especificar el valor cuando se inserten valores.

La sentencia **PRIMARY KEY** sirve para indicar que campo o campos serán utilizados como llave primaria.

+ El tipo de dato INT permite almacenar números desde -2147483648 hasta 2147483647.
+ El tipo de dato VARCHAR permite almacenar una cadena de caracteres de longitud 50 y/o 100 segun se especifique.
+ El tipo de dato DATE permiate almacenar fechas (sin hora).
+ El tipo de dato TINYINT permite almacenar números desde -128 hasta 127.

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
```

La tabla **Historial_Conexion** tendrá como llave primaria el campo ID, los campos ID, ID_Usuario y Fecha_Hora tendrán un **constraint** de tipo **NOT NULL** el cual sirve para garantizar que esos campos siempre tendrán valor.

La sentencia **AUTO_INCREMENT** sirve para indicar que el campo ID, será un correlativo automatico, manejado por la base de datos, es decir no es necesario especificar el valor cuando se inserten valores.

La sentencia **PRIMARY KEY** sirve para indicar que campo o campos serán utilizados como llave primaria.

La sentencia **FOREIGN KEY** sirve parea indicar que campo o campos será uitilizados como llave foranea o referencia a otra tabla. El campo ID_Usuario corresponde al campo ID de la tabla Usuario.

```sql
CREATE TABLE Historial_Conexion (
	ID INT NOT NULL AUTO_INCREMENT,
	ID_Usuario INT NOT NULL,
	Fecha_Hora DATE NOT NULL,
	IP VARCHAR(15), 
	Navegador VARCHAR(50), 
	PRIMARY KEY (ID),
	FOREIGN KEY (ID_Usuario) REFERENCES Usuario(ID)
);
```
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
