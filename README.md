# Insecure deserialisation

Serialisation involves converting objects into a string or byte stream for storage in files or databases; the reverse process, deserialisation, reconstructs an object from a string or byte stream.

Insecure deserialisation vulnerabilities allow attackers to manipulate serialised objects because of inadequate validation mechanisms provided by applications. As a result, the server reconstructs the serialised object's arbitrary code.

## Index.php

Viewing the `robots.txt` file of the website revealed a file with the `.phps` file extension, which outputs the source code of PHP files.

Modifying the GET request to `GET /index.phps HTTP/1.1` allowed me to view the source code of the `index.php` file.  

1.  The source code revealed a PHP file, `cookie.php`, from which the contents are retrieved. 
2.  If the `$_POST["user"]` and `$_POST["pass"]` values are set, SQLite3 will run and load "users.db". The variables `$username` and `$password` are assigned the         values of `$_POST["user"]` and `$_POST["pass"]`.
3.  A new object `$perm_res` is constructed by creating a new instance of the permissions class with the arguments `$username` and `$password`.
4.  The object is compared against the `is_guest()` or `is_admin()` methods to determine the privileges of the user logging in.
5.  A `login` cookie is constructed by serialising, base64 encoding and URL encoding the `$perm_res` object.
6.  The user is redirected to `authentication.php`
7.  If authentication fails, the user is presented with "_Invalid login._" on the webpage.

## Cookie.php

1.  The `permissions` class is defined with the following properties: `$username` and `$password` and methods: `__construct()`, `__toString()`, `is_guest()` and `is_admin()`.
2.  The `is_guest()` method connects to and queries `users.db` to identify if the username and password correspond to a guest user. If yes, the function returns 'true' otherwise 'false'. 
3.  The `is_admin()` method does the same but identifies whether the username and password correspond to an admin user.
4.  If `$_COOKIE["login"]` is set, it is URL decoded, base64 decoded, unserialised and assigned to the `$perm` object.
5.  If an error occurs with deserialisation, this is reported to the user.

## Authentication.php

1.  The `access_log` class is defined with the property `$log_file` and methods `__construct()`, `__toString()`, `append_to_log()`, and `read_log()`
2.  With the `require_once()` statement, the contents of the `cookie.php` file are retrieved.
3.  If the $perm variable is set and the user is an admin, the user is presented with _"Welcome admin"_.
4.  An instance of the access_log class is created with the log file path `access.log`.
5.  Otherwise, if the user has been identified as a guest, they are presented with _"Welcome guest"_.
