# Wordpress Database

Some queries to clean and improve database performance in Wordpress

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
