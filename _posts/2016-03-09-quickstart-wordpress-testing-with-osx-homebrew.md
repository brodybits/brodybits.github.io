---
layout: post
title: Quickstart WordPress testing with OSX Homebrew
categories: wordpress osx homebrew
---

Here is a quickstart procedure for testing WordPress with OSX Homebrew. This procedure needs a recent version of PHP with `php-cli` and MySQL to be installed as described below. In addition, this procedure uses Node.js with npm to run WordPress for testing.

The Apache `httpd` is *not* included due to its level complexity. WordPress unfortunately requires MySQL by default installation, and there is no easy workaround.

## Recommended packages

It is recommended to install the following packages using Homebrew:

- Node.js with npm
- Recent version of PHP such as `php70` with `php-cli`
- MySQL installation

To install Node.js with npm: `brew install node`

To install PHP: `brew install php70` (will also install `jpeg`, `unixodbc`, and `openssl`)

To install MySQL: `brew install mysql`

This will put the MySQL commands in your path as well.

**NOTE:** It should be theoretically possible to remove MySQL using `brew uninstall mysql`. But I have read that running MySQL will leave a number of artifacts. I found a nice description in the answer to: <http://superuser.com/questions/678324/osx-mavericks-mysql-homebrew-after-native-install>

## Local WordPress workarea

Download a recent version of WordPress from <https://wordpress.org/download/> and extract the contents into your workarea directory.

In your WordPress directory copy or rename `wp-config-sample.php` file to `wp-config.php`.

Try `php <path-to-your-wordpress-installation>/index.php` and it should output some HTML with an error message regarding database connectivity.

You can also try running a local PHP server and check it from a browser. For example (assuming you have Node and npm installed):

- `npm install -g express-php-serve`
- `express-php-serve --php-cgi /usr/local/bin/php-cgi --port 8000 ./wordpress`
- Check from a web browser

THANKS for guidance: <http://dogwood.skr.jp/wordpress/sqlite-integration/>

**NOTE:** `express-php-serve` is a Node.js wrapper to make it easier to serve PHP from a local directory. Unfortunately it is using `express` internally, which takes longer to install using `npm`. In the future I would like to provide a similar tool *without* the `express` dependency.

**Alternative:** Use `wp-cli` (*not covered here*, needs immediate database configuration)

### Setup MySQL with root user

As described above, using Homebrew to install MySQL will put the MySQL commands in your path.

To check installed MySQL version: `mysql --version`

Start MySQL: `mysql.server start`

Try MySQL shell (root user): `mysql -u root`

To quit MySQL shell: `quit` or `exit`

**IMPORTANT NOTE:** The MySQL root user is *not* the same thing as an OSX root user.

**Alternative:** It is possible to use the MySQL shell using `mysql -v` but it gave me an access denied error (using MySQL 5.7).

To set MySQL root password: `mysqladmin -u root password 'your_password'`

**NOTE:** Please be sure to use single quotes around the password!

**Troubleshooting:** In case of an access denied error, I found the following resources related to MySQL 5.7:

- <http://stackoverflow.com/a/33924648/1283667>
- <https://github.com/Homebrew/homebrew/issues/46174>

**To remove MySQL:** `brew remove mysql` then `brew cleanup` or `brew cleanup --force` (*not tested*). I found the following resources in case of artifacts that may be left behind:

- <http://superuser.com/questions/678324/osx-mavericks-mysql-homebrew-after-native-install>
- some answers to: <http://stackoverflow.com/questions/4359131/brew-install-mysql-on-mac-os/33924648>
- old: <http://soatechlab.blogspot.com/2011/01/completely-remove-mysql-on-mac-os-x.html>

THANKS for guidance: <http://coolestguidesontheplanet.com/get-apache-mysql-php-phpmyadmin-working-osx-10-10-yosemite/>

### Setup WordPress database

To create the WordPress database: `mysql -u root -pyour_password -e 'create database YourWPDatabaseName;'`

To show databases: `mysql -u root -pyour_password -e 'show databases';`

### Configure WordPress

Edit the `DB_NAME`, `DB_USER`, and `DB_PASSWORD` entries in `wp-config.php`.

**Quick check:** `express-php-serve --php-cgi /usr/local/bin/php-cgi --port 8000 ./wordpress`

It should now redirect to the `wp-admin/install.php` page.

Follow these steps to configure:

- Select the language
- Fill in the basic site tile, username, password, and e-mail information

It should succeed. Try browsing <http://localhost:8000>.

It is *recommended* to setup new authentication keys. In short:

- To geneate the salt key configuation simply navigate to: <https://api.wordpress.org/secret-key/1.1/salt/>
- Copy-paste the salt key values into `wp-config.php`

In case of problems logging in, a quick and *extremely insecure* fix is to add the following lines to `wp-config.php`:

{% highlight php %}
function wp_validate_auth_cookie($cookie = '', $scheme = '') {
	$user = get_user_by('login', 'testuser');

	return $user->ID;
}
{% endhighlight %}

## Future ideas

Here are some resources for running WordPress with another database, such as MS SQL Server, PDO/MySQLi, or SQLite:

- <https://wordpress.org/plugins/wp-db-driver/>
- <https://wordpress.org/support/topic/wordpress-and-sql-server-1>
- <http://dogwood.skr.jp/wordpress/sqlite-integration/>
- <https://wordpress.org/support/plugin/wordpress-database-abstraction>

## References

- <https://www.greengeeks.com/kb/6050/what-are-wordpress-security-keys/>
- <https://digwp.com/2010/09/wordpress-security-keys/>
- <http://dogwood.skr.jp/wordpress/sqlite-integration/>
- <https://wordpress.org/plugins/wp-db-driver/>
- <http://coolestguidesontheplanet.com/fastest-way-to-install-wordpress-on-osx-10-6/>
- <http://coolestguidesontheplanet.com/get-apache-mysql-php-phpmyadmin-working-osx-10-10-yosemite/> (decent description of how to install MySQL though it refers to using a DMG download)
- <http://stackoverflow.com/questions/4359131/brew-install-mysql-on-mac-os/33924648>
- <http://superuser.com/questions/678324/osx-mavericks-mysql-homebrew-after-native-install>
- <http://soatechlab.blogspot.com/2011/01/completely-remove-mysql-on-mac-os-x.html> (old)
- <http://www.hongkiat.com/blog/wordpress-command-line/> (old post with some out-of-date information)
- <http://www.morearty.com/blog/2013/02/03/the-hackers-way-to-install-apache-php-mysql-and-or-wordpress-on-osx-mountain-lion/>

