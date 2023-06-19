# Insecure deserialisation

## Index.php

Viewing the `robots.txt` file of the website revealed a file with the `.phps` file extension, allowing us to view the source code of PHP files.

Modifying the GET request to `GET /index.phps HTTP/1.1` allowed me to view the source code of the `index.php` file.  The source code revealed a PHP file, `cookie.php`, from which the contents are retrieved from. 

**1.**  If the `$_POST["user"]` and `$_POST["pass"]` values are set, SQLite3 will run and load "users.db". The variables `$username` and `$password` are assigned the         values of `$_POST["user"]` and `$_POST["pass"]`.
**2.**  A new object `$perm_res` is constructed by creating a new instance of the permissions class with the arguments `$username` and `$password`. 
**3.**  The object is compared against the `is_guest()` or `is_admin()` methods to determine the privileges of the user logging in.
**4.**  A `login` cookie is constructed by serialising, base64 encoding and URL encoding the `$perm_res` object.