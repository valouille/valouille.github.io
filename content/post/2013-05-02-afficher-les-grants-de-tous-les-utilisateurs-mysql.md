---
title: Afficher les « grants » de tous les utilisateurs mySQL
author: VaLouille
type: post
date: 2013-05-02T13:49:38+00:00
url: /2013/05/afficher-les-grants-de-tous-les-utilisateurs-mysql/
categories:
  - linux
  - sysadmin
tags:
  - mysql
  - show all grants

---
Pour afficher tous les droits de tous les utilisateur d&rsquo;un serveur mySQL, il faut créer une procédure. Pour cela, dans le prompt de mySQL, il suffit de taper :

```
USE mysql;

DELIMITER //
CREATE PROCEDURE showAllGrants() BEGIN
    DECLARE done INT DEFAULT 0;
    DECLARE theUser CHAR(16);
    DECLARE theHost CHAR(60);
    DECLARE cur1 CURSOR FOR SELECT user, host FROM mysql.user;
    DECLARE CONTINUE HANDLER FOR SQLSTATE '02000' SET done = 1;
    OPEN cur1;

    REPEAT
        FETCH cur1 INTO theUser, theHost;
        IF NOT done THEN
            SET @sql := CONCAT('SHOW GRANTS FOR ', QUOTE(theUser), '@', QUOTE(theHost));
            PREPARE grantStatement FROM @sql;
            EXECUTE grantStatement;
            DROP PREPARE grantStatement;
        END IF;
    UNTIL done END REPEAT;

    CLOSE cur1;
END//
DELIMITER ;
```

Pour lancer la procédure, on tape :


```
mysql> USE mysql; CALL showAllGrants();
```

```
+------------------------------------------------------------------------------------------------------------------------+
| Grants FOR dump@localhost                                                                                            |
+------------------------------------------------------------------------------------------------------------------------+
| GRANT ALL PRIVILEGES ON *.* TO 'dump'@'localhost' IDENTIFIED BY PASSWORD '*CAFB74DD90C39GSB644B614EE99H53E24622041E' |
+------------------------------------------------------------------------------------------------------------------------+
1 ROW IN SET (0.00 sec);

+----------------------------------------------------------------------------------------------------------------------------------------+
| Grants FOR root@localhost                                                                                                              |
+----------------------------------------------------------------------------------------------------------------------------------------+
| GRANT ALL PRIVILEGES ON *.* TO 'root'@'localhost' IDENTIFIED BY PASSWORD '*C9449BKSJ56127FD1F19FF194JBC01AFF68E4686' WITH GRANT OPTION |
+----------------------------------------------------------------------------------------------------------------------------------------+
1 ROW IN SET (0.00 sec);

Query OK,  ROWS affected, 1 warning (0.00 sec);
```

On peut du coup utiliser sed, pour changer des noms d&rsquo;utilisateurs ou des IP par exemple, et générer des lignes qu&rsquo;il suffira de copier/coller dans mySQL :

```
mysql -e "USE mysql; CALL showAllGrants();" | grep "10\.0\.1" | grep GRANT | sed <span class="st_h">'s/10\.0\.1/10\.0\.2/' | sed <span class="st_h">'s/$/;/' && echo "FLUSH PRIVILEGES;"
```

```
GRANT USAGE ON *.* TO 'dump'@'10.0.2.%' IDENTIFIED BY PASSWORD '*462C4380AEF7E9GKD25507FB9AF9GK23867DC5';
GRANT ALL PRIVILEGES ON `database`.* TO 'user'@'10.0.2.%';
[...]
FLUSH PRIVILEGES;
```
