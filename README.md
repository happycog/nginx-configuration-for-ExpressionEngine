# nginx configuration for ExpressionEngine

At [Vector Media Group](http://www.vectormediagroup.com) we're big fans of nginx. It's often a much faster and more stable alternative to Apache -- especially on projects getting heavy concurrent traffic. We use it on as many ExpressionEngine (and other CMS) sites as possible, so have created a template to use as a starting point on our projects.

This configuration has support and working code for:

* SSL/TLS (and has been patched to prevent the POODLE attack)
* ExpressionEngine `index.php` rewriting
* CE Cache static caching
* Arbitrary domain and URL path 301 redirects
* Forced SSL on certain sections or all pages of the site
* Cross-origin resource sharing (CORS)
* Far-future `expires` headers on static assets

## Instructions to deploy this nginx setup

* Run `yum install php54-fpm nginx` or its equivalent for your server OS and desired versions to install nginx and PHP-FPM.
* Place this repository's files in `/etc/nginx` and `/etc/nginx/conf.d`, leaving any other existing files.
* Rename `domain-redirects.conf` and `domain-rules.conf` to use your actual domain (e.g. `vectormediagroup.com-rules.conf`)
* Change "domain" in the three files to the main domain of the site
* In nginx.conf:
  * Update the `server_name` directive for the real domain
  * Update the `root` directive to point to the real docroot (e.g. `/var/www/html/vectormediagroup`)
  * Update the `include` paths for `domain-redirects.conf` and `domain-rules.conf` to match their new names 
  * Update the `fastcgi_param  SCRIPT_FILENAME` path
    * You only need to change the `/domain` part of this line to match the real docroot. Here's a working example of what this line can ultimately look like:
     ```fastcgi_param  SCRIPT_FILENAME  /var/www/html/vectormediagroup$fastcgi_script_name;```
  * If you're using SSL:
    * Update the SSL certificate paths
    * Uncomment `listen 443 ssl` and the SSL certificate paths
    * Replace `ssl_certificate` with your SSL certificate file. If you have a CA Bundle, [concatenate it after your main certificate file](http://nginx.org/en/docs/http/configuring_https_servers.html#chains)
    * Replace `ssl_certificate_key` with your SSL private key.
* In domain-redirects.conf:
  * Add domain redirects if needed by uncommenting lines and updating the examples as appropriate.
  * Uncomment the last directive and replace `www.domain.com` with the production domain to force `www` in URLs
* In domain-rules.conf
  * If you're using CE Cache static caching, uncomment the top "CE Cache static" section (everything through "`# End CE Cache static rewrites`") and replace `xxxxxx` in the cache path with the actual value provided by CE Cache.
  * Look for the rule that replaces `.htaccess` `Deny` files. Replace `system` with the site's actual `system`/control panel path.
  * Add any other folders/paths that would usually need `Deny from all` to this directive (remember, nginx doesn't parse `.htaccess` files)
  * Uncomment any other commented blocks if you need any of the other rules in place. We've left a few examples to follow.
  
### Server notes
  * On CentOS or RHEL run the following to start nginx and PHP-FPM, and make sure they start on reboot:

```bash
service nginx start
chkconfig nginx on
service php-fpm start
chkconfig php-fpm on
```

  * You'll need to run `service nginx reload` for any config changes to take effect.
 
#### Warranty/Notes

There's no warranty of any kind. If you have a suggestion, please open a ticket and we'll take a look. This configuration is provided completely as-is; if something breaks, you lose data, or something else bad happens, the author(s) and owner(s) of this code are in no way responsible.

The SSL cipher choices are as listed in [Mozilla's TLS suggestions](https://wiki.mozilla.org/Security/Server_Side_TLS) as of this document's writing. We strongly suggest keeping up with that list and making updates as needed, even if not mentioned here.
