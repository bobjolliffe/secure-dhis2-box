# Web application firewall

## Install
apt-get install libapache2-modsecurity

mv /etc/modsecurity/modsecurity.conf-recommended /etc/modsecurity/modsecurity.conf

rm -rf /usr/share/modsecurity-crs

cd /usr/share/modsecurity-crs

edit /etc/apache2/mods-available/security2.conf

```apache
<IfModule security2_module>
        # Default Debian dir for modsecurity's persistent data
        SecDataDir /var/cache/modsecurity

        # Include all the *.conf files in /etc/modsecurity.
        # Keeping your local configuration in that directory
        # will allow for an easy upgrade of THIS file and
        # make your life easier
        IncludeOptional /etc/modsecurity/*.conf
        IncludeOptional "/usr/share/modsecurity-crs/*.conf"
        IncludeOptional "/usr/share/modsecurity-crs/rules/*.conf"
</IfModule>
```

Blocks funny things like:
curl -k "https://slart.dhis2.org/dhis/dhis-web-commons/security/login.action/?><script>'alert(1)'</script>"

```apache
## some false positives and dhis2 tweaks

# libinjection doesn't like to see rename()
# for example: GET /dhis/api/26/dimensions.json?fields=id,displayName~rename(name),dimensionType
# disable rule 942100 when we see rename in the argument list
SecRule &ARGS:fields "!@eq 0" "id:10002,phase:1,pass,nolog,chain"
  SecRule ARGS:fields "@rx \brename\(" "ctl:ruleRemoveTargetById=942100;ARGS:fields"

# /api/i18n causes weird parsing error of json.  disable for now
<Location "/dhis/api/i18n" >
  <IfModule mod_security2.c>
    SecRuleEngine Off
  </IfModule>
</Location>
```