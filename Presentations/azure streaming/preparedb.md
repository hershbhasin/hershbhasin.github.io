Now we have a database called mydb. Let's create a table that will hold extracted data from the  event hub created earlier.  We will use the Azure query explorer to create our database but before we can do that, we need to add a Firewall rule in SQL Server that will allow our workstation to connect to SQL Server.

This can be done from the Firewalls and virtual networks tab of our sql server. I'll just click on the add client ip link and my local ip will be able to access sql server.

Now I can go to the Query explorer and run my create TSQL script. My table will have a column to store the brand of the tshirt and another column to store the color of the tshirt.  I will also have a data time column called EventEnqueuedUtcTime that will store the date and time a row was created.

The columns in this table will match exactly the fields in the json payload that will be pumped into the event hub. The event hub will receive data messages as a json payload, an azure stream analytics job will transform the data from the event hub into the column structure defined in this sql table and insert a row in this table
