---

title: postgresql hstore is easy to compare
author: ytjohn
date: 2015-08-23 17:58:10

layout: post

slug: postgresql-hstore-is-easy-to-compare

---
hstore is an option key=>value column type that's been around in postgresql for a long time. I was looking at it for a project where I want to compare "new data" to old, so I can approve it. There is a `hstore-hstore` option that compares two hstore collections and shows the differences.

In reality, an hstore column looks like text. It's just in a format that postgresql understands.

Here, we have an existing record with some network information.

```
hs1=# select id, data::hstore from d1 where id = 3;
 id |                          data                          
----+--------------------------------------------------------
  3 | &quot;ip&quot;=&gt;&quot;192.168.219.2&quot;, &quot;fqdn&quot;=&gt;&quot;hollaback.example.com&quot;
(1 row)
```

Let's say I submitted a form with slightly changed network information. I can do a select statement to get the differences.

```
hs1=# select id, hstore(&#039;&quot;ip&quot;=&gt;&quot;192.168.219.2&quot;, &quot;fqdn&quot;=&gt;&quot;hollaback01.example.com&quot;&#039;)-data from d1 where id =3;
 id |             ?column?              
----+-----------------------------------
  3 | &quot;fqdn&quot;=&gt;&quot;hollaback01.example.com&quot;
(1 row)
```

This works just as well if we're adding a new key.

```
hs1=# select id, hstore(&#039;&quot;ip&quot;=&gt;&quot;192.168.219.2&quot;, &quot;fqdn&quot;=&gt;&quot;hollaback01.example.com&quot;, &quot;netmask&quot;=&gt;&quot;255.255.255.0&quot;&#039;)-data from d1 where id =3;
 id |                           ?column?                            
----+---------------------------------------------------------------
  3 | &quot;fqdn&quot;=&gt;&quot;hollaback01.example.com&quot;, &quot;netmask&quot;=&gt;&quot;255.255.255.0&quot;
(1 row)
```

This information could be displayed on a confirmation page. Ideally, a proposed dataset would be placed somewhere, and a page could be rendered on the fly showing any changes an approval would create within the database.

Then we can update with the newly submitted form.

```
hs1=# update d1 set data = data || hstore(&#039;&quot;ip&quot;=&gt;&quot;192.168.219.2&quot;, &quot;fqdn&quot;=&gt;&quot;hollaback01.example.com&quot;, &quot;netmask&quot;=&gt;&quot;255.255.255.0&quot;&#039;) where id = 3;
UPDATE 3

hs1=# select id, data::hstore from d1 where id = 3; id |                                         data                                         
----+--------------------------------------------------------------------------------------
  3 | &quot;ip&quot;=&gt;&quot;192.168.219.2&quot;, &quot;fqdn&quot;=&gt;&quot;hollaback01.example.com&quot;, &quot;netmask&quot;=&gt;&quot;255.255.255.0&quot;
(1 row)

```


Note that if I wanted to delete a key instead of just setting it to NULL, that would be a separate operation.

```
update d1 SET data = delete(data, &#039;ip&#039;) where id = 3;
UPDATE 1
```

http://stormatics.com/howto-handle-key-value-data-in-postgresql-the-hstore-contrib/
