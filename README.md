# Görev 6 – Veritabanı Yükseltme ve Sürüm Yönetimi

Bu proje, Microsoft SQL Server Management Studio (SSMS) üzerinde AdventureWorks2022 veritabanı kullanılarak gerçekleştirilmiştir. Projede bir veritabanının yeni sürüme yükseltilmesi senaryosu simüle edilmiş ve DDL (Data Definition Language) işlemlerinin izlenmesi için bir tetikleyici (trigger) oluşturulmuştur.

---

## 📌 Kullanılan Sorgular

### 1. SQL Server Sürümünü Öğrenme

```sql
SELECT @@VERSION;
```

---

### 2. Veritabanı Uyumluluk Seviyesini Ayarlama

```sql
ALTER DATABASE AdventureWorks2022 SET COMPATIBILITY_LEVEL = 160;
```

> *Not: 160 değeri SQL Server 2022 sürümünü temsil eder.*

---

### 3. SchemaChangeLog Tablosunun Oluşturulması

```sql
CREATE TABLE SchemaChangeLog (
    ChangeID INT IDENTITY(1,1) PRIMARY KEY,
    EventType NVARCHAR(100),
    ObjectName NVARCHAR(255),
    ObjectType NVARCHAR(50),
    SQLCommand NVARCHAR(MAX),
    ChangeDate DATETIME DEFAULT GETDATE(),
    ChangedBy NVARCHAR(100)
);
```

---

### 4. DDL Trigger Oluşturulması

```sql
CREATE TRIGGER LogSchemaChanges
ON DATABASE
FOR CREATE_TABLE, ALTER_TABLE, DROP_TABLE
AS
BEGIN
    INSERT INTO SchemaChangeLog (EventType, ObjectName, ObjectType, SQLCommand, ChangedBy)
    SELECT 
        EVENTDATA().value('(/EVENT_INSTANCE/EventType)[1]', 'NVARCHAR(100)'),
        EVENTDATA().value('(/EVENT_INSTANCE/ObjectName)[1]', 'NVARCHAR(255)'),
        EVENTDATA().value('(/EVENT_INSTANCE/ObjectType)[1]', 'NVARCHAR(50)'),
        EVENTDATA().value('(/EVENT_INSTANCE/TSQLCommand/CommandText)[1]', 'NVARCHAR(MAX)'),
        SYSTEM_USER;
END;
```

---

### 5. Trigger’ı Test Etmek İçin Kullanılan Örnek DDL Sorguları

```sql
CREATE TABLE TestLog (
    id INT
);

ALTER TABLE TestLog ADD name NVARCHAR(100);

DROP TABLE TestLog;
```

---

### 6. Kayıtları Kontrol Etmek için Kullanılan Sorgu

```sql
SELECT * FROM SchemaChangeLog ORDER BY ChangeDate DESC;
```

---

### 7. Trigger’ı Devre Dışı Bırakma ve Etkinleştirme

```sql
-- Devre dışı bırakma
DISABLE TRIGGER LogSchemaChanges ON DATABASE;

-- Yeniden etkinleştirme
ENABLE TRIGGER LogSchemaChanges ON DATABASE;
```

---

## 📂 Proje Amacı

Bu proje ile aşağıdaki kazanımlar hedeflenmiştir:

- Mevcut bir veritabanının daha yeni bir sürüme yükseltilmesi için plan oluşturmak
- Veritabanı yapısal değişikliklerinin kayıt altına alınmasını sağlamak
- Trigger mekanizması ile veri bütünlüğü ve izlenebilirlik sağlamak

---

## 📌 Notlar

- Proje SQL Server 2022 Developer Edition üzerinde gerçekleştirilmiştir.
- AdventureWorks2022 örnek veritabanı kullanılmıştır.
- Ekran görüntüleri ve açıklamalı dökümantasyon raporda sunulmuştur.
