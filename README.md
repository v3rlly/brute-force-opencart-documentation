# BRUTE FORCE OPENCART PASSWORDS TUTORIAL
Nesse pequeno tutorial eu explico como realizar ataques de força bruta em senhas opencart


## CENÁRIO:
- VOCÊ CONSEGUIU ACESSO AO BANCO DE DADOS OPENCART.
- VOCÊ CONSULTA A SENHA E O SALT DA SENHA NA TABELA DO ADMIN.

| password      						  | salt    | email             | status | username    | lastname | ip        | firstname |
|-----------------------------------------|---------|-------------------|--------|-------------|----------|-----------|-----------|
|f2e9efd4a366507c5b1cba7749659d93d61ae335 |oInuc412L| admin@victim.com 	| 1      | admin       | das ganbi| 127.0.0.1 | developer |


## O PROBLEMA:
Se vc não conhece o sistema de criptografia de senhas do OpenCart, Então você vai tentar desvendar essa HASH com força bruta comum usando criptografia SHA1.

```
INICIO:

$wordlist_string[] = WORDLIST.TXT

$HASH = "f2e9efd4a366507c5b1cba7749659d93d61ae335";
$comparative = SHA1($wordlist_string[0]);

SE $comparative IGUAL A $HASH
	ENTÃO: A SENHA É : $comparative

SE NÃO
	$comparative = SHA1($wordlist_string[++]);
	COMPARA A $comparative COM A $HASH
FIM.

ASSIM SUCESSIVAMENTE ATÉ TERMOS $comparative == $HASH
```


O problema é que `$password` (A senha decimal do admin) não é simplesmente uma string criptografada com SHA1.
Logo um programa de força bruta baseada nesse tipo de criptografia nunca iria gerar uma HASH igual a HASH do admin que você quer "desvendar".

PORQUE?

SE OLHARMOS O CÓDIGO QUE FAZ A AUTENTICAÇÃO DO USUÁRIO NO OPENCART
`\opencart\system\library\customer.php`

TEMOS O SEGUINTE CÓDIGO:
```
$customer_query = $this->db->query("SELECT * FROM " . DB_PREFIX . "customer WHERE LOWER(email) ='" . $this->db->escape(strtolower($email)) . "' AND (password = SHA1(CONCAT(salt, SHA1(CONCAT(salt, SHA1('" . $this->db->escape($password) . "'))))) OR password = '" . $this->db->escape(md5($password)) . "') AND status = '1' AND approved = '1'");
```

A PARTE QUE NOS INTERESSA:
```
password = SHA1(CONCAT(salt, SHA1(CONCAT(salt, SHA1('" . $this->db->escape($password) . "'))))) OR password = '" . $this->db->escape(md5($password)) . "')
```

TRANSFORMANDO PRA NOSSA LINGUA FICA ALGO COMO:
```
$password=SHA1($salt.SHA1($salt.SHA1($password)));
```

obs: ignorei a parte `OR password = '" . $this->db->escape(md5($password)) . "')` já que testamos nossa hash e já sabemos que é SHA1, certo?

## Agora já sabemos como devemos criptografar a strings da nossa wordlist e comparar com a HASH do ADMIN

###### Um script php de força bruta em senhas OpenCart Ficaria algo do tipo:


```
INICIO:

$wordlist_string[] = WORDLIST.TXT

$salt = "oInuc412L";
$password = "f2e9efd4a366507c5b1cba7749659d93d61ae335";
$comparative = SHA1($salt.SHA1($salt.SHA1($wordlist_string[0])));

SE $comparative IGUAL A $password
	ENTÃO: A SENHA É : $comparative

SE NÃO
	$comparative = SHA1($salt.SHA1($salt.SHA1($wordlist_string[++])));;
	COMPARA A $comparative COM A $password
FIM.

ASSIM SUCESSIVAMENTE ATÉ TERMOS $comparative == $password
```

###### _DUVIDAS? SUGESTÕES?: [pablov3rlly gmail com]_
