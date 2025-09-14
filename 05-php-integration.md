# PHP OIDC App (Keycloak @8443) behind Apache with Drupal (/), Django (/django), PHP (/phpapp)

A minimal PHP app that authenticates users via Keycloak (OIDC) and runs behind Apache alongside Drupal (port 80, root) and Django (proxied at **/django/** → 8000). The PHP app is mounted at **/phpapp** and uses jumbojett/openid-connect-php.

### Topology
+ Keycloak: **https://<host>:8443/** (realm: **<your-realm>**)
![alt text](/images/image-9.png)
+ Apache vhost (:80)
    + Drupal: **http://<host>/**
    ![alt text](/images/image-10.png)
    + Django: **http://<host>/django/** → proxy to **127.0.0.1:8000**
    ![alt text](/images/image-11.png)
    + PHP app: **http://<host>/phpapp/** (served via PHP-FPM)

### Prerequisites
+ Rocky Linux (or RHEL/CentOS-like)
+ Apache (**httpd**) with modules: **rewrite, proxy, proxy_http, proxy_fcgi, headers**
+ PHP + PHP-FPM
+ Composer
+ Keycloak up on **8443** with a realm you control
+ SELinux enabled (we set a boolean to allow proxying)

### Install & Setup

#### 1. PHP app code

```bash
sudo mkdir -p /var/www/php_app
sudo chown -R apache:apache /var/www/php_app
cd /var/www/php_app
composer require jumbojett/openid-connect-php
```
Create these files:
```bash
/var/www/php_app/
  index.php
  login.php
  callback.php
  profile.php
  logout.php
  vendor/...
```

index.php
```php
<?php
header('Location: /phpapp/login.php');
```

login.php
```php
<?php
require __DIR__.'/vendor/autoload.php';
use Jumbojett\OpenIDConnectClient;
session_start();
$oidc = new OpenIDConnectClient(
  'https://<host>:8443/realms/<your-realm>',
  'php-app',
  '<your-secret>'
);
$oidc->setRedirectURL('http://<host>/phpapp/callback.php');
$oidc->addScope(['openid','profile','email']);
$oidc->authenticate();
$_SESSION['id_token'] = $oidc->getIdToken();
$_SESSION['user'] = $oidc->requestUserInfo();
header('Location: /phpapp/profile.php'); exit;
```

callback.php
```php
<?php
require __DIR__.'/vendor/autoload.php';
use Jumbojett\OpenIDConnectClient;
session_start();
$oidc = new OpenIDConnectClient(
  'https://<host>:8443/realms/<your-realm>',
  'php-app',
  '<your-secret>'
);
$oidc->setRedirectURL('http://<host>/phpapp/callback.php');
$oidc->addScope(['openid','profile','email']);
$oidc->authenticate();
$_SESSION['id_token'] = $oidc->getIdToken();
$_SESSION['user'] = $oidc->requestUserInfo();
header('Location: /phpapp/profile.php'); exit;
```

profile.php
```php
<?php
session_start();
if (empty($_SESSION['user'])) { header('Location: /phpapp/login.php'); exit; }
$u = $_SESSION['user'];
?>
<h1>Welcome, <?=htmlspecialchars($u->name ?? $u->preferred_username ?? 'User')?></h1>
<p>Email: <?=htmlspecialchars($u->email ?? '')?></p>
<p><a href="/phpapp/logout.php">Logout</a></p>
```

logout.php
```php
<?php
session_start();
$issuer = 'https://<host>:8443/realms/<your-realm>';
$redirect = 'http://<host>/phpapp/';
session_destroy();
header('Location: '.$issuer.'/protocol/openid-connect/logout?post_logout_redirect_uri='.urlencode($redirect));
exit;
```

Replace **< host >, < your-realm >, < your-secret >** with real values.
![alt text](/images/image-12.png)
Optional: store these in env vars instead of hardcoding.

#### 2. Apache vhost

**/etc/httpd/conf.d/drupal.conf:**

```apache
<VirtualHost *:80>
    ServerName <host>

    DocumentRoot /var/www/drupal/web
    <Directory /var/www/drupal/web>
        AllowOverride All
        Require all granted
        DirectoryIndex index.php index.html
        Options -Indexes
    </Directory>

    RewriteEngine On
    RewriteCond %{REQUEST_URI} ^/django/ [OR]
    RewriteCond %{REQUEST_URI} ^/phpapp/
    RewriteRule ^ - [L]

    Alias /django/static/ /var/www/django_project/static/
    <Directory /var/www/django_project/static/>
        Require all granted
        Options -Indexes
    </Directory>

    ProxyPreserveHost On
    RequestHeader set X-Forwarded-Proto "http"
    ProxyPass        /django/static/ !
    ProxyPass        /django/ http://127.0.0.1:8000/
    ProxyPassReverse /django/ http://127.0.0.1:8000/
    # ProxyPassReverseCookiePath / /django/  (optional)

    Alias /phpapp/ /var/www/php_app/
    <Directory /var/www/php_app>
        AllowOverride All
        Require all granted
        DirectoryIndex login.php index.php
        Options -Indexes
    </Directory>

    <FilesMatch "\.php$">
        SetHandler "proxy:unix:/run/php-fpm/www.sock|fcgi://localhost/"
    </FilesMatch>

    ErrorLog  /var/log/httpd/drupal_error.log
    CustomLog /var/log/httpd/drupal_access.log combined
</VirtualHost>
```

Enable and restart:

```bash
sudo dnf install -y php php-fpm
sudo systemctl enable --now php-fpm
sudo setsebool -P httpd_can_network_connect 1
sudo apachectl -t && sudo systemctl reload httpd
```

#### 3. Keycloak client (realm: < your-realm >)

+ Client ID: **php-app**
+ Client authentication: ON (confidential)
+ Valid Redirect URIs:
    + **http://< host >/phpapp/callback.php**
+ Post-logout redirect URIs:
    + **http://<host>/phpapp/**
+ Web Origins:
 + **http://< host >**

4) Verify
+ Visit **http://< host >/phpapp/** → redirect to Keycloak → login → land on **profile.php**.
+ Visit **http://< host >/django/** to verify Django proxy.
+ Visit **http://< host >/** for Drupal root.

#### Troubleshooting

+ ERR_TOO_MANY_REDIRECTS: Callback not exchanging code or session not set. Ensure callback.php calls **$oidc->authenticate()** and that PHP sessions are writable **(/var/lib/php/session)**.

+ “invalid redirect_uri”: Keycloak Redirect URIs must match exactly (scheme, host, path).

+ “token not active yet”: Check server time (NTP).

+ 403/404 under **/phpapp**: Confirm **Alias /phpapp/** path, Directory block, and PHP-FPM handler.

+ Django cookies losing path: Add **ProxyPassReverseCookiePath / /django/** if needed.

#### Security Notes

+ Don’t commit real secrets. Prefer env vars for issuer, client ID, secret, redirect.

+ Keep Keycloak on HTTPS (8443). App can stay on HTTP for this exercise, but production should be HTTPS end-to-end.

+ Limit Apache indexes and set proper ownership: **chown -R apache:apache /var/www/php_app.**