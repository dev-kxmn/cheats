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

### Wordpress Cleanup

```sql
DELETE FROM `wp_options` WHERE `option_name` LIKE ('%\_transient\_%');
UPDATE wp_comments SET comment_agent ='' ;
DELETE FROM wp_comments WHERE comment_approved = 'spam';
UPDATE wp_posts SET ping_status = 'closed';

-- Delete post revisions
DELETE a,b,c
 FROM wp_posts a
 LEFT JOIN wp_term_relationships b ON ( a.ID = b.object_id)
 LEFT JOIN wp_postmeta c ON ( a.ID = c.post_id )
 LEFT JOIN wp_term_taxonomy d ON ( b.term_taxonomy_id = d.term_taxonomy_id)
 WHERE a.post_type = 'revision'
 AND d.taxonomy != 'link_category';
```

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
