# ERROR 1698 (28000): Access denied for user 'root'@'localhost'



I just installed MySQL server (version 8.0) into my Ubuntu 20.04 with an empty password for the root user. It has occurred to me that the Access denied for user issue kept spinning my head.

Because for every project we use MySQL we always provide username and password to authenticate the query. I needed to change the password and get rid of the Access denied issue.

Some internet user suggests that changing the password can solve the issue, like the following

```
alter user 'root'@'localhost' identified by 'newPasswd';
```

But it didn't in my case.

I want to see if the password was really set.

I know that the MySQL user information is stored in **mysql.user** table.

Let's see what is inside mysql.user table.

```bash
sudo mysql -u root -p
use mysql;
desc mysql.user;
select Host, User, authentication_string from mysql.user;
```

The out put of the last query says that the password for the user root is empty. So changing password didn't work. But why?

After doing some research, I found out that MySQL server is using something called authentication plugin to authenticate a MySQL client when it tries to connect to the server. In my case, the authentication plugin being used doesn't support a username/password authentication method.

Lets see what plugin is being used.

```bash
select User, Host, plugin from mysql.user;
```

### The out put is

```
+------------------+-----------------------+
| User             | plugin                |
+------------------+-----------------------+
| root             | auth_socket           |
| mysql.sys        | caching_sha2_password |
| mysql.infoschema | caching_sha2_password |
| mysql.session    | caching_sha2_password |
| debian-sys-maint | caching_sha2_password |
+------------------+-----------------------+
```

OK, the plugin is **auth_socket**. It means that MySQL server will check if the client is using a Unix socket and then will compare the username. It just ignores any password provided in the connection URL.

After some time, I figured that the correct way to change the password is to change it and update the authentication plugin at the same time, like in the following:

```bash
alter user 'root'@'localhost' identified with mysql_native_password by 'password';
```

After that, if we double check the plugin we will see something like

```
+------------------+-----------------------+
| User             | plugin                |
+------------------+-----------------------+
| root             | mysql_native_password |
| mysql.sys        | caching_sha2_password |
| mysql.infoschema | caching_sha2_password |
| mysql.session    | caching_sha2_password |
| debian-sys-maint | caching_sha2_password |
+------------------+-----------------------+
```

**mysql_native_password** came to the light, that is the familiar old username/password way of authentication.

That's all. Happy coding!

