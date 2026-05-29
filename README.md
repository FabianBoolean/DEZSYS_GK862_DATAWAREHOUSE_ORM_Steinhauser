# DEZSYS_GK862_DATAWAREHOUSE_ORM_Steinhauser

Fabian Steinhauser

## Lösungsweg

### Schritt 1

Zuerst wurde das Projekt in IntelliJ geöffnet und mit meinem eigenen GitHub-Repository verbunden.

```bash
git remote remove origin
git remote add origin git@github.com:FabianBoolean/DEZSYS_GK862_DATAWAREHOUSE_ORM_Steinhauser.git
git push -u origin main
```

Danach wurde das Projekt auf GitHub gepusht.

---

### Schritt 2

MySQL wurde installiert und gestartet, da beim ersten Start der Anwendung noch keine Datenbankverbindung funktioniert hat.

```bash
brew install mysql
brew services start mysql
```

Danach wurde überprüft, ob MySQL funktioniert:

```bash
mysql --version
```

Anschließend wurde eine Verbindung zu MySQL hergestellt:

```bash
mysql -u root
```

Die Datenbank wurde erstellt:

```sql
CREATE DATABASE example;
SHOW DATABASES;
```

Danach konnte die Spring Boot Anwendung erfolgreich mit MySQL verbunden werden.

---

### Schritt 3

Die Datei `application.properties` wurde für die Datenbankverbindung verwendet.

```properties
spring.application.name=demo
spring.jpa.hibernate.ddl-auto=update
spring.datasource.url=jdbc:mysql://${MYSQL_HOST:localhost}:3306/example
spring.datasource.username=root
spring.datasource.password=
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
```

Mit `spring.jpa.hibernate.ddl-auto=update` werden die Tabellen automatisch aus den Entity-Klassen erstellt bzw. aktualisiert.

---

### Schritt 4

Danach wurde zuerst das vorhandene User-Beispiel getestet.

User hinzufügen:

```bash
curl -X POST "http://localhost:8080/demo/add" -d "name=Fabian" -d "email=fabian@test.at"
```

Ausgabe:

```text
Saved
```

User anzeigen:

```bash
curl "http://localhost:8080/demo/all"
```

Ausgabe:

```json
[{"id":1,"name":"Fabian","email":"fabian@test.at"}]
```

Damit wurde getestet, ob Spring Boot, MySQL und das Repository richtig funktionieren.

---

### Schritt 5

Danach wurden die Klassen für die Warehouse-Anwendung erstellt.

Es wurden folgende Klassen ergänzt:

- `Warehouse`
- `Product`
- `WarehouseRepository`
- `ProductRepository`

### Warehouse

```java
@Entity
public class Warehouse {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Integer id;

    private String warehouseName;
    private String warehouseAddress;
    private String warehouseCity;
    private String warehouseCountry;

    @OneToMany
    private List<Product> products;
}
```

### Product

```java
@Entity
public class Product {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Integer id;

    private String productName;
    private String productCategory;
    private Integer productQuantity;
    private String productUnit;

    @ManyToOne
    private Warehouse warehouse;
}
```

Die Beziehung wurde mit `@OneToMany` und `@ManyToOne` umgesetzt.  
Ein Warehouse kann mehrere Produkte haben, ein Produkt gehört zu einem Warehouse.

---

### Schritt 6

Für beide Entities wurden eigene Repositories erstellt.

```java
public interface WarehouseRepository extends CrudRepository<Warehouse, Integer> {
}
```

```java
public interface ProductRepository extends CrudRepository<Product, Integer> {
}
```

Dadurch stehen einfache CRUD-Methoden automatisch zur Verfügung.

---

### Schritt 7

Der `MainController` wurde erweitert.  
Die vorhandenen User-Endpunkte wurden behalten und neue Endpunkte für Warehouse und Product ergänzt.

```text
POST /demo/warehouse/add
GET  /demo/warehouse/all
POST /demo/product/add
GET  /demo/product/all
```

Mit diesen Endpunkten können Warehouses und Produkte gespeichert und abgefragt werden.

---

### Schritt 8

Es wurden zwei Warehouses eingefügt.

```bash
curl -X POST "http://localhost:8080/demo/warehouse/add" \
-d "warehouseName=Linz Bahnhof" \
-d "warehouseAddress=Bahnhofsstrasse 27/9" \
-d "warehouseCity=Linz" \
-d "warehouseCountry=Austria"
```

```bash
curl -X POST "http://localhost:8080/demo/warehouse/add" \
-d "warehouseName=Wien Zentrum" \
-d "warehouseAddress=Hauptstrasse 10" \
-d "warehouseCity=Wien" \
-d "warehouseCountry=Austria"
```

Danach wurden zehn Produkte eingefügt und jeweils einem Warehouse zugeordnet.

Beispiel:

```bash
curl -X POST "http://localhost:8080/demo/product/add" \
-d "productName=Bio Orangensaft Sonne" \
-d "productCategory=Getraenk" \
-d "productQuantity=2500" \
-d "productUnit=Packung 1L" \
-d "warehouseId=1"
```

---

### Schritt 9

Die Daten wurden über die REST-Endpunkte überprüft.

Warehouses anzeigen:

```bash
curl "http://localhost:8080/demo/warehouse/all"
```

Produkte anzeigen:

```bash
curl "http://localhost:8080/demo/product/all"
```

Dabei wurden alle Produkte inklusive zugehörigem Warehouse zurückgegeben.

---

### Schritt 10

Zusätzlich wurden die Daten direkt in MySQL kontrolliert.

Tabellen anzeigen:

```sql
USE example;
SHOW TABLES;
```

Dabei wurden unter anderem diese Tabellen erstellt:

```text
product
warehouse
warehouse_products
user
```

Produkte anzeigen:

```sql
SELECT id, product_name, product_category, product_quantity, product_unit, warehouse_id FROM product;
```

Dabei wurden 10 Produkte angezeigt.

Warehouses anzeigen:

```sql
SELECT * FROM warehouse;
```

Dabei wurden 2 Warehouses angezeigt.

---

## Aufgetretene Probleme

### MySQL war nicht installiert

Beim Prüfen von MySQL kam zuerst:

```text
zsh: command not found: mysql
```

Das Problem wurde mit Homebrew gelöst:

```bash
brew install mysql
```

---

### Spring Boot konnte MySQL nicht erreichen

Beim Start der Anwendung kam zuerst:

```text
Communications link failure
Connection refused
```

Der Grund war, dass MySQL noch nicht gestartet war und die Datenbank `example` noch nicht existiert hat.

Lösung:

```bash
brew services start mysql
mysql -u root
CREATE DATABASE example;
```

Danach konnte die Anwendung erfolgreich starten.

---

### Falscher HTTP-Request

Beim Öffnen von `/demo/add` im Browser kam:

```text
Method Not Allowed
```

Der Grund war, dass der Endpoint nur POST unterstützt.  
Gelöst wurde es mit `curl -X POST`.

---

### GitHub Login mit Passwort funktionierte nicht

Beim Pushen kam:

```text
Password authentication is not supported for Git operations.
```

Das Problem wurde mit einem SSH-Key gelöst.

```bash
ssh-keygen -t ed25519 -C "FabianBoolean"
ssh -T git@github.com
```

Danach wurde das Repository per SSH verbunden.

---

## Git Commits

Es wurden mehrere Commits gemacht:

```text
Setup Spring Boot and MySQL connection
Add project gitignore
Add warehouse and product entities
Add warehouse and product endpoints
```

---

## Questions

### What is ORM and how is JPA used?

ORM bedeutet Object Relational Mapping.  
Dabei werden Java-Klassen auf Tabellen in einer relationalen Datenbank abgebildet.

JPA wird verwendet, um diese Zuordnung mit Annotationen zu beschreiben. Hibernate übernimmt dann die Umsetzung in der Datenbank.

---

### What is the application.properties used for and where must it be stored?

Die Datei `application.properties` enthält Konfigurationen für die Spring Boot Anwendung.

Zum Beispiel:

- Datenbank-URL
- Benutzername
- Passwort
- JPA-Einstellungen

Die Datei muss hier liegen:

```text
src/main/resources/application.properties
```

---

### Which annotations are frequently used for entity types?

Wichtige Annotationen sind:

- `@Entity`
- `@Id`
- `@GeneratedValue`
- `@OneToMany`
- `@ManyToOne`

Wichtig ist, dass jede Entity einen Primary Key besitzt.

---

### What methods do you need for CRUD operations?

Wichtige Methoden aus `CrudRepository` sind:

- `save()`
- `findAll()`
- `findById()`
- `deleteById()`

---

## Links

https://spring.io/guides/gs/accessing-data-mysql

https://spring.io/guides/gs/accessing-data-jpa

https://docs.spring.io/spring-framework/reference/data-access/orm.html

https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/repository/CrudRepository.html
