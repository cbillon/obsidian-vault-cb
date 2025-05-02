---
link: https://fabianlee.org/2024/09/05/bash-calculating-number-of-days-till-certificate-expiration-using-openssl/
site: "Fabian Lee : Software Engineer"
excerpt: "The openssl utility can be used to show the details of a certificate,
  including its ‘Not After’ expiration date in string format.  This can be
  transformed into “how many days till expiration” with a bit of Bash date math.
  Create test certificate and key Using a line provided by Diego Woitasen for
  non-interactive self-signed certification ... Bash: calculating number of days
  till certificate expiration using openssl"
twitter: "https://twitter.com/Fabian Lee : Software Engineer"
tags:
  - slurp/cert
  - slurp/expiration
  - slurp/expire
  - slurp/not-after
  - slurp/openssl
slurped: 2025-03-08T08:02
title: "Bash: calculating number of days till certificate expiration using openssl"
---

![](https://fabianlee.org/wp-content/uploads/2018/10/gnu-logo.gif)The [openssl](https://linux.die.net/man/1/openssl) utility can be used to show the details of a certificate, including its ‘Not After’ expiration date in string format.  This can be transformed into “how many days till expiration” with a bit of Bash date math.

## Create test certificate and key

Using a line provided by [Diego Woitasen](https://stackoverflow.com/questions/10175812/how-to-generate-a-self-signed-ssl-certificate-using-openssl) for non-interactive self-signed certification creation, let’s create a 90 day certificate.

openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -sha256 -days 90 -nodes -subj "/C=XX/ST=StateName/L=CityName/O=CompanyName/OU=CompanySectionName/CN=CommonNameOrHostname"

## Show ‘Not After’ certificate date string

Using openssl, we can show the expiration date of that certificate in string format.

$ openssl x509 -in cert.pem -text -noout | grep 'Not After :'
Not After : Sep 7 10:13:57 2024 GMT

## Calculate the number of days till expiration

Using a custom datediff function provided by [camh](https://unix.stackexchange.com/questions/24626/quickly-calculate-date-differences/24636#24636), this date string can be turned into an integer representing how many days until expiration of the certificate.

function datediff() {
    d1=$(date -d "$1" +%s)
    d2=$(date -d "$2" +%s)
    echo $(( (d1 - d2) / 86400 + 1 ))
}

cert_expiration_str=$(openssl x509 -in cert.pem -text -noout | grep -Po "Not After : \K.*" | head -n1)
echo "days till expiration:" $(datediff "$cert_expiration_str" "$(date -u)")

I have the full [openssl_cert_days_till_expiration.sh](https://github.com/fabianlee/blogcode/blob/master/bash/openssl_cert_days_till_expiration.sh) posted on github.

REFERENCES

[openssl man page](https://linux.die.net/man/1/openssl)

[unixexchange, user ‘camh’ provides datediff function](https://unix.stackexchange.com/questions/24626/quickly-calculate-date-differences/24636#24636)

[stackoverflow, diego woitasen provides non-interactive creation of self-signed cert](https://stackoverflow.com/questions/10175812/how-to-generate-a-self-signed-ssl-certificate-using-openssl)

NOTES

As contributed by [Christian Kugler](https://github.com/Syphdias), another method of checking expiration is the openssl ‘checkend’ flag

# will cert expire in 5 days?
$ openssl s_client -connect fabianlee.org:443 </dev/null 2> /dev/null|openssl x509 -noout -checkend $((86400*5)); echo $?
Certificate will not expire