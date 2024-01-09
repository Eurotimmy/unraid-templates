**Important notes for using Odoo17 on an Unraid server via the CA installation**

1. odoo.conf to be copied in "Config Location" path on Unraid server 
2. odoo.conf edits required
3. The default database name in the Odoo docker container is 'db', so on PostgreSQL (postgresql15 in CA) installation you can set the variable POSTGRES_DB to 'db'
4. The CA template can be viewed in 'Advanced mode' and the 'Post Arguments' can be removed AFTER the initial first boot and successful login to the Odoo Web UI 
