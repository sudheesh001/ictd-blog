---
layout: default
title: Measuring the Adoption of HTTPS by Governments Worldwide
subtitle: Results of the web measurement of HTTPS adoption in Government websites worldwide
nav_order: 10
date: 06-10-2020
categories: ICTD, Security, Governance, Web Measurement
nav_exclude: true
summary: Across the world, government websites are expected to be reliable sources of information, regardless of their average daily view count. Further, interactions with these websites can contain sensitive identity, medical, or legal data, which could be lucrative to steal. Have you ever wondered whether government websites enforce better security practices than other sites due to the sensitive personal data they handle, or whether a malicious hacker could post false information on these websites to mislead the public and affect election results? In this blog post we try to summarize our key findings from the research work accepted to IMC’20. Please find our accepted paper here.

---

# {{page.title}}
### {{page.subtitle}}

{% if page.categories %}
{% assign categories = page.categories | split: ',' %}
{% for category in categories %}
{{category}}
{: .label }
{% endfor %}
{% endif %}

##### By: [Sudheesh Singanamalla](https://sudheesh.info/), [Esther Jang](https://homes.cs.washington.edu/~infrared/) on {{page.date | date_to_string }}

---

Across the world, government websites are expected to be reliable sources of information, regardless of their average daily view count. Further, interactions with these websites can contain sensitive identity, medical, or legal data, which could be lucrative to steal. Have you ever wondered whether government websites enforce better security practices than other sites due to the sensitive personal data they handle, or whether a malicious hacker could post false information on these websites to mislead the public and affect election results? In this blog post we try to summarize our key findings from the research work accepted to [IMC’20](https://conferences.sigcomm.org/imc/2020/){:target="_blank"}. Please find our accepted [paper here](https://ictd.cs.washington.edu/docs/papers/2020/IMC2020-Government-HTTPS-Measurement.pdf){:target="_blank"}.

One well-established technology for preventing data theft or modification during these sensitive web interactions is HTTPS, an encrypted data transfer protocol between a client and a server providing a secure version of the older HTTP protocol. HTTPS uses Transport Layer Security (TLS), the successor to the now deprecated Secure Sockets Layer (SSL), to establish a secure communication channel using asymmetric key cryptography. HTTPS uses uniquely-identifying server certificates issued by certificate authorities (CAs) to (1) verify the identity of the server the client is trying to communicate with, and (2) set up encryption to maintain confidentiality (privacy of message content) and integrity (non-tampering) of messages between the client and server.

The adoption of the HTTPS protocol on the web has been increasing worldwide every year. Modern browsers have moved to explicitly indicate webpages as insecure if they do not have valid certificates. Free certificate-issuing tools like [certbot](https://certbot.eff.org/){:target="_blank"} from the EFF have made it easier to issue [Let’s Encrypt](https://letsencrypt.org/){:target="_blank"} certificates. Search engines lower the page ranking of non-HTTPS websites. Recent estimates from [netmarketshare](https://meterpreter.org/https-encryption-traffic/){:target="_blank"} indicate that HTTPS adoption overall is above 90%, but we find that the adoption among government websites is less than 30%.

### What happens if a website is not secured with HTTPS?

A website not using HTTPS can be spoofed via a [Man-in-the-Middle (MITM)](https://en.wikipedia.org/wiki/Man-in-the-middle_attack){:target="_blank"} attack, where the attacker is able to actively modify the content of the website and serve this information to users. As a specific example, if a website hosting a travel visa application does not use HTTPS, an attacker could selectively change the served website so that the POST message endpoint redirects sensitive user information to a hacker’s database instead of the government’s, leading to identity theft.


### What did we find?

Government websites in general have low popularity (i.e. total traffic or visits) as compared to commercial websites like Netflix, Google or Facebook; less than 1.5% of entries in public Top Millions datasets (such as [Censys](https://censys.io/){:target="_blank"}, [Majestic Million](https://majestic.com/reports/majestic-million){:target="_blank"}, and [Tranco](https://tranco-list.eu/){:target="_blank"}) are government websites, although as noted before most citizens will be required to interact with government websites at some point, e.g. for taxes, utilities, identity, electoral or legal purposes. For our measurements, we start with this 1.5% and use a number of techniques to retrieve more entries, including web crawling, crowdsourcing website search using Mechanical Turk, and manually curating entries from underrepresented countries to create an extensive dataset of **135,408 government websites** across the world. In the process, we found that **72% of the websites do not use valid HTTPS**. You could find our dataset published in a [Github repository](https://github.com/uw-ictd/GovHTTPS-Data){:target="_blank"}.

<p align="center">
<img src="/img/world_map.png" alt="A picture showing 3 different world maps in 3 rows colored by percentage of availability, https support and valid https.">
</p>

### What were the common errors in HTTPS?

There were various reasons for HTTPS invalidity on these websites. 36% of invalid websites had a misconfigured hostname, rendering the certificate invalid. 24% used a certificate chain where the root certificate was not present in the list of trusted certificate authorities when validated using OpenSSL. 13% used a self-signed certificate with no chain of trust, which is invalid.

We also noticed a large variation in the validity duration of issued certificates that were invalid. Valid certificates tend to follow the rules agreed upon in the CA/B forums for a fixed 2–3 year validity period, after which they would need to be renewed.

<p align="center">
<img src="/img/cert_validity_global.png" alt="A scatter plot with an inset indicating certificate issuance and expiration dates along with validity. Inset is only valid.">
</p>

We found that insecure cryptographic signing algorithms like SHA1 or MD5 were used to issue some certificates, which is considered invalid. Additionally, websites using key sizes greater than 4096 bit RSA are also invalid, due to lack of support to validate such large key sizes.

We performed deeper case studies on the United States of America (USA) and South Korea (Republic of Korea or ROK), as both have authoritative public lists of their government websites. We found that despite having similarly high technology adoption and human development index scores, HTTPS adoption is very different for these countries. While **81.12% of USA** government websites have valid HTTPS, only **37.95% of ROK** government websites do. Since its decision in 2018 to move away from their old NPKI infrastructure, Korea continues to use CA authorities which are considered untrusted and are not present in the trust stores of major browsers and operating systems.

### Are popular government websites in the Top Millions as good as similarly popular commercial websites?

We find a positive correlation between website popularity (as ranking in the Tranco Top Million) and valid HTTPS. Using the 1.5% of popularity-ranked top million websites belonging to governments, we group them into buckets based on ranking. We compare this distribution to 1) a random sample of all the listed non-government websites and 2) a sample matching the ranking distribution of the government websites. We find that **government websites within the top million also have much lower rates of HTTPS adoption compared to non-government websites.**

<p align="center">
<img src="/img/gov_ranking.png" alt="A scatter plot and regression line indicating fit & percentage of https validity for government and non-government websites.">
</p>

### Does hosting service provider correlate with certificate validity?

Government websites can have stringent regulations for using public cloud services (eg. FedRAMP certification in the USA) and tend to be privately hosted. However, the usage of cloud host services or CDNs offering TLS/SSL certificates tends to result in higher HTTPS validity compared to private hosting. This could be due to the reduced effort needed to maintain and renew the certificates.

<p align="center">
<img src="/img/gov_hosting.png" alt="Bar charts indicating the validity of https in Gov worldwide, in USA, in ROK based on hosting type of the website.">
</p>

### Did you responsibly inform governments about their insecure websites?

**Yes.** As a part of the study, we provided detailed reports to the various government domain registrars of each country indicating the websites which did not support HTTPS, had invalid HTTPS, or did not upgrade HTTP to HTTPS. In a follow-up scan after 2 months, we noticed an improvement in 18.7% of the websites, which were either taken offline or had a new valid certificate. This of course considers websites taken offline to have been shut down on purpose by the webmasters, and is an optimistic result. If we do not consider these websites, we find only an 8.3% improvement after our disclosure. However, some countries (Bahrain, Burkina Faso, Cuba, Honduras, Portugal, Libya and Vietnam) did show improvement above 40%, suggesting that disclosure can have positive impacts. The US government also issued a statement after our disclosure mandating HSTS preloading for all ".gov" websites by September 1, 2020. However, we cannot definitely attribute these positive changes to our notifications.

### What does this mean for governments?

Even with valid HTTPS, interactions with government websites are not guaranteed to be safe, private, or accessible.

For example, HTTPS does block governments from censoring or tampering with other nations' web content for their own citizens' consumption. However, the inability to censor specific content could result in banning entire websites over letting the content remain uncensored, as seen in the Russian Wikipedia ban in 2015. Further, a disproportionate number of certificate authorities worldwide are US organizations, which could be legally compelled by the US government to perform a [compelled certificate creation attack](https://s3.amazonaws.com/files.cloudprivacy.net/ssl-mitm.pdf){:target="_blank"}. Finally, the easy, free access to services like Let’s Encrypt also makes it easy for phishing websites to obtain valid certificates and pose as governments using non-official domains. For example, *etagov.sl* was a carefully crafted phishing attempt on *eta.gov.lk*, the official electronic travel authorization website for the Govt. of Sri Lanka. Anyone can currently use commercial domain registrars to register *abcgov.ccTLD* (e.g. *"kingcountyseattlegov.us"*), which poses a significant phishing risk. Thus, we believe domain registrars should perform additional checks to periodically verify government-related domain name registrations.

We also propose that adding a local government certificate authority as a sub CA in the certification process might add significant value to the process, and reduce the concerns of a compelled certificate creation attack. Free services like Let’s Encrypt could use different sub-CAs for different government websites, which could be especially effective because such free services are very popular among government websites.

In conclusion, many (\~72% of) government sites still do not use HTTPS, either due to lack of TLS infrastructure or a large variety of certificate errors. Through our study of 135,139 government websites across the Internet from almost every country in the world, we identified major categories and frequencies of these errors, privately disclosed our findings to the sites' web administrators, and measured the effects. Common errors included: the misconfiguration of hostnames, certificate expiry, reuse of keys and serial numbers between sites likely to be hosted on different servers, use of default certificates, use of insecure cryptographic algorithms such as MD5 or SHA1, certificate self-signing, and others.

In the US, we believe our recommendations can be used to improve compliance with the [DOTGOV Online Trust in Government Act of 2019](https://www.congress.gov/bill/116th-congress/senate-bill/2749/text){:target="_blank"}, which outlines technical security requirements (including HTTPS) for the .gov domain needed to maintain public trust in our government.

---

