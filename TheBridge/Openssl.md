Cifrar:
openssl rc4 -in password.txt -out password.rc4 -pbkdf2

Descifrar:
openssl rc4 -d -in password.txt -out password.rc4 -pbkdf2

