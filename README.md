# mikrotik-sstp-letsencrypt-renew
Script for mikrotik scheduler to renew sstp letsencrypt cert and set it to sstp server
```rsc
:local domain "your.domain.com";
/ip service enable [find name="www"];
/ip firewall filter set [find comment="for_letsencrypt"] disabled=no;
/certificate enable-ssl-certificate dns-name=$domain;
/ip service disable [find name="www"];
/ip firewall filter set [find comment="for_letsencrypt"] disabled=yes;

:local newestCert "";
:local newestDate [:totime "jan/01/1970"];
:local newestDateUntil "";

:foreach cert in=[/certificate find] do={
    :local validFrom [:totime [/certificate get $cert invalid-before]];
    :local certName [/certificate get $cert name];
    :local validAfter [/certificate get $cert invalid-after];
    :if ($validFrom > $newestDate) do={
        :set newestDate $validFrom;
        :set newestCert $certName;
        :set newestDateUntil $validAfter;
    }
}

#:put ("The newest certificate is: " . $newestCert);
#:log info "The newest certificate is: $newestCert";
#:log info "It's valid until: $newestDateUntil";

/interface sstp-server server set enabled=yes certificate=$newestCert

:local Mes
:local smtpserv [:resolve "mail.domain.com"];
:local Eaccount "router@domain.com";
:local passwd "router_password";
:set Mes "Letsencrypt cert for SSTP is updated to $newestCert. It's valid until $newestDateUntil."

do {/tool e-mail send from=$Eaccount to=root@domain.com server=$smtpserv port=587 tls=starttls user=$Eaccount password=$passwd subject="Letsencrypt cert for SSTP is updated!" body=$Mes} on-error={};
 }
```
