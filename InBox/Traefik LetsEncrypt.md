Since you shouldn’t have landed on this page without knowing what Traefik, Let’s Encrypt and OVH are, I won’t extend myself introducing them 😃

As you should know Let’s Encrypt is able to deliver x509 certificates for your domains after having validated your website/domain identity. This consists in resolving one of the four challenge types: http-01, tls-sni-01 (soon outdated), tls-alpn-01 or dns-01 (introduced early 2018). While most administrators uses the http or tls challenge (easier to configure), I will be writing here about the dns-01 one as it offers more possibilities (such as delivering wildcard certificates) and as it is, according to me, the most secured one.

You can actually find some other tutorials on the Internet dealing with the same subject, but I wasn’t really satisfied with them as they give Traefik too must access rights to the OVH API… That was not something suitable to me (they usually give all rights under the ‘/domain’ path, or worse under the ‘/’ path! 😱).

## Why should you consider using the dns-01 challenge?

Well I know that using the dns-01 challenge might be impossible in a lot of companies for security concerns as it requires to give rights to Traefik to create and remove some DNS records (TXT entries), but on the other hand it permits you to have a wildcard certificates (I’m not really a huge fan of them), you will be able to have certificate for internal services (that are not facing the Internet/resolves to a private IP), and more especially it will prevent your domains and certificates to be listed on some specialized websites such as [https://crt.sh](https://crt.sh) (example with the [medium.com domain](https://crt.sh/?q=%25.medium.com) 😨).

# Configuring OVH

To allow Traefik to create and remove DNS records, you first need to create an application account for Traefik to interact with the OVH API. To generate a new application account, go to the web page listed below, and follow instructions:  
Europe: [https://eu.api.ovh.com/createApp/](https://eu.api.ovh.com/createApp/)  
Canada/USA: [https://ca.api.ovh.com/createApp/](https://ca.api.ovh.com/createApp/)

![](https://miro.medium.com/v2/resize:fit:700/1*T63nw-5oqXpvnEVkzTtwmw.png)

![](https://miro.medium.com/v2/resize:fit:700/1*n7Ifg9rohUBMSySyX-l-5g.png)

You don’t really need to write down nor save those credentials as we will be using them very soon (and you will be able to find them later on your Traefik configuration). Furthermore, if you lost them you can always delete and re-create them (see at the end of this chapter for the deletion).

Now that your application account has been created, you can check its existence in the OVH API console:  
Europe: [https://eu.api.ovh.com/console](https://eu.api.ovh.com/console)  
Canada/USA: [https://ca.api.ovh.com/console](https://ca.api.ovh.com/console)

![](https://miro.medium.com/v2/resize:fit:700/1*0TiYtJ80pRUa1JB87wb40Q.png)

Yeah! We have it listed 😃

Next we need to create some access rules for this application account. To resolve the dns-01 challenge Traefik should be able to create a TXT DNS record, refresh the zone and delete the record. We can check which calls are performed by Traefik looking at the [source code](https://github.com/xenolf/lego/blob/master/providers/dns/ovh/ovh.go) of the acme library it’s using ([xenolf/lego](https://github.com/xenolf/lego)).
Methode Edouard Bzh
https://api.ovh.com/createToken/?GET=/domain/zone/domain.tld/*&POST=/domain/zone/domain.tld/*&PUT=/domain/zone/domain.tld/*&GET=/domain/zone/domain.tld&DELETE=/domain/zone/domain.tld/record/*

As giving access right to an application account can’t be done from the OVH API console, we need to do it from a terminal with a curl command (or any other program that can perform HTTP requests). The request looks as follow:

curl -XPOST -H “X-Ovh-Application: <my_application_key>” -H “Content-type: application/json” https://eu.api.ovh.com/1.0/auth/credential -d ‘{  
 “accessRules”:[  
{“method”:”POST”,”path”:”/domain/zone/<my_domain>/record”},  
 {“method”:”POST”,”path”:”/domain/zone/<my_domain>/refresh”},  
 {“method”:”DELETE”,”path”:”/domain/zone/<my_domain>/record/*”}  
 ],  
 “redirection”: “https://www.<my_domain>"  
 }’

The ‘redirection’ parameter tell which web page the application should be redirected to once logged in. This is quite useless in our case but it still must be filled.

_If you want Traefik to manage multiple domains, you may either duplicate those access rules for each domain, or replace ‘<my_domain>’ with ‘*’ which will affect all domains.  
Also, OVH sadly do not permit us restraining the entry type to TXT_ 😞

So with this tutorial data the request looks as following:

==curl -XPOST -H “X-Ovh-Application: ya9xB3cE6mRlZkhn” -H “Content-type: application/json” https://eu.api.ovh.com/1.0/auth/credential -d ‘{“accessRules”:[{“method”:”POST”,”path”:”/domain/zone/example.com/record”},{“method”:”POST”,”path”:”/domain/zone/example.com/refresh”},{“method”:”DELETE”,”path”:”/domain/zone/example.com/record/*”}], “redirection”: “https://www.example.com"}'==

If the request process correctly, it should return a JSON with following informations:

{“validationUrl”:”https://eu.api.ovh.com/auth/?credentialToken=zPIM6Qyln7q2pwzCxU26RzVvXCZo7QHXAjXMYNrRljt1EXdEq7Uk5skNSJNiNw82","state":"pendingValidation","consumerKey":"JlJ64Fwtc2yost2WjuIzxBXNiEE0QJ6C"}

_The ‘consumerKey’ will later be used in Traefik configuration._

Access rights have been created and attached to the application account but it still needs validation (notice the state as being ‘pendingValidation’). You have to click on the validation URL, and enter **your** credentials to validate the process (you should also double-check that the application name and access rights listed on the web page are correct). Remember to select the validity period to ‘Unlimited’ as we don’t want to repeat the procedure regularly.

![](https://miro.medium.com/v2/resize:fit:700/1*JeInl5XDcLua89Qt0qT2iA.png)

After hitting the “Log in” button if you get redirected to the web page you filed on your request everything went fine. You can still verify that rules have been attached to your application account by checking on the OVH API console:

![](https://miro.medium.com/v2/resize:fit:700/1*NoCBy-XiUX1RBQnS19G8sg.png)

If this is the case you are now all set on the OVH side! ✅

## Deleting OVH account for Traefik

If one day you think your OVH application account have been compromised or you just want to delete it, this also can be done from the OVH API console:

![](https://miro.medium.com/v2/resize:fit:700/1*Ymxvx2FYghLbDhXWLMEwAQ.png)

(You can verify it has correctly been deleted by checking the list of applications).

# Configuring Traefik

The Traefik part is way easier and straight forward compared to the OVH one.

Traefik configuration has to be done in two different places: in its main configuration file (traefik.toml/traefik.yaml) and providing OVH credential to the Traefik container.

Providing OVH credentials is common to every Traefik versions. Edit your docker-compose file (or any equivalent) and add the following variables to the environment section:

environment:  
- OVH_ENDPOINT=ovh-eu (or ovh-ca)  
- OVH_APPLICATION_KEY=<your_application_key>  
- OVH_APPLICATION_SECRET=<your_application_secret>  
- OVH_CONSUMER_KEY=<your_application_consumer_key>

Replacing those fields with data from this example:

environment:  
- OVH_ENDPOINT=ovh-eu  
- OVH_APPLICATION_KEY=ya9xB3cE6mRlZkhn  
- OVH_APPLICATION_SECRET=WdrqTkATVNkhVGtLaCzFWIbNclbMimz6  
- OVH_CONSUMER_KEY=JlJ64Fwtc2yost2WjuIzxBXNiEE0QJ6C

## Traefik v2.x (v1.x below)

[Traefik — Let’s Encrypt documentation](https://docs.traefik.io/https/acme/)  
_As I find the YAML format more convenient I’m be using it for this tutorials, but paths are the same for TOML files._  
Create a **certificatesResolvers** root section in your configuration file:

certificatesResolvers:  
  myResolverName:  
    acme:  
      email: acme@example.com  
      storage: /etc/traefik/acme.json  
      dnsChallenge:  
        provider: ovh  
        delayBeforeCheck: 10

👉 **_Beware that if you are running multiple Traefik instances, they cannot share the same ‘acme.json’ file for concurrency reason. You will need to use the Enterprise Edition._**  
The ‘delayBeforeCheck’ option tells Traefik how many seconds it should wait before verifying the DNS TXT entry (and then trigger Let’s Encrypt challenge verification). I had some issue setting it to 0 so I set it to 10 (time delay isn’t a real issue when it comes to certificate generation as they are renewed [30 days in advance](https://docs.traefik.io/https/acme/#automatic-renewals)).

You also have the possibility to generate [wildcard domains certificates](https://docs.traefik.io/https/acme/#wildcard-domains).

💡You do not need to restart the Traefik container for it to take config changes into account if you activate the [‘watch’ option](https://docs.traefik.io/providers/file/#watch).

To finish, you just need to follow Traefik documentation to configure your [Docker containers](https://docs.traefik.io/user-guides/docker-compose/acme-dns/)/[Kubernetes ingress](https://docs.traefik.io/user-guides/crd-acme/) to create endpoints.

## Traefik v1.x

[Traefik — Let’s Encrypt documentation](https://docs.traefik.io/v1.7/configuration/acme/)  
In the traefik.toml file, under your ‘acme’ part you need to create a ‘acme.dnsChallenge’ section containing the following:

[acme]  
email = “letsencrypt@example.com”  
storage = “/etc/traefik/acme.json”  
entryPoint = “https”  
[acme.dnsChallenge]  
provider = “ovh”  
delayBeforeCheck = 10

👉 **_Beware that if you are running multiple Traefik instances, they cannot share the same ‘acme.json’ file for concurrency reason. You will need to use a_** [**_key-value store for your configuration_**](https://docs.traefik.io/v1.7/user-guide/kv-config/)**_.  
_**The ‘delayBeforeCheck’ option tells Traefik how many seconds it should wait before verifying the DNS TXT entry (and then trigger Let’s Encrypt challenge verification ; [official documentation](https://docs.traefik.io/v1.7/configuration/acme/#delaybeforecheck)). I had some issue setting it to 0 so I set it to 10 (time delay isn’t a real issue when it comes to certificate generation as they are renewed few days in advance).

You also have the possibility to ask Let’s Encrypt to generate wildcard domains certificates by adding a ‘acme.domains’ sub-section ([official documentation](https://docs.traefik.io/v1.7/configuration/acme/#wildcard-domains)):

[[acme.domains]]  
main = “*.example.com”  
…

Restart the Traefik container and you’re done with the config 🎉

To finish, you need to configure your [Docker containers](https://docs.traefik.io/v1.7/user-guide/docker-and-lets-encrypt/#exposing-web-services-to-the-outside-world)/[Kubernetes ingresses](https://docs.traefik.io/v1.7/user-guide/kubernetes/#name-based-routing) to create endpoints.
