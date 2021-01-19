# CREATE USER

```sql
CREATE USER "username" IDENTIFIED BY password;
    GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, DROP,
    REFERENCES, INDEX, ALTER, CREATE TEMPORARY TABLES,
    LOCK TABLES, EXECUTE, CREATE VIEW, SHOW VIEW,
    CREATE ROUTINE, ALTER ROUTINE, EVENT, TRIGGER
ON `dbname`.* TO "username"@"%"
WITH GRANT OPTION;
```


# BACKUP

### Creating backup

```sh
mysqldump \
	--add-drop-table \
	--add-drop-database \
	--skip-comments \
	--lock-tables \
	--extended-insert \
	--skip-log-queries \
	--databases $DATABASE [$TABLELIST]
```

### Applying backup

```sh
mysql -u username -p database_name < file.sql
```

# OPTIMIZING


### Optimize Tables**

Desfragmenta e libera espaço em tabelas. Observe que InnoDB cria uma nova
tabela temporária com os dados sem fragmentação e depois apaga a antiga
renomeando a temporária em seu lugar.

Por isso é _importante neste processo parar o banco de dados e fazer um backup_.

```sh
{ echo 'USE loja; ';
  for i in $(echo 'USE loja; SHOW TABLES;' | mysql --) ; do
    echo "OPTIMIZE TABLE $i;" ;
  done;
} | mysql
```
