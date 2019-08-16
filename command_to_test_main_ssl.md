---
title: "[筆記] 測試mail server 的SSL憑證的指令 Command to test mailserver SSL"
date: 2019-03-20T11:42:47+08:00
noSummary: false
featuredImage: "https://h.cowbay.org/images/post-default-10.jpg"
categories: []
tags: []
author: "Eric Chang"
---

今天老闆出國，發slack說手機不能寄信，看了一下，似乎是因為用GMAIL的APP來收信

然後google 不知道跟人家改了什麼，結果不接受原本的認證了... WTF ....

然後，這問題應該很久了，結果現在才在講 ....


<!--more-->
底下都是用linux 主機來進行測試

windows環境應該也可以，只是要自己去安裝 openssl 軟體


To verify SSL, connect to any Linux server via SSH and use the instructions below:

**測試 SSL-IMAP 993 port**

```
openssl s_client -showcerts -connect mail.example.com:993
```

結果應該會像是這樣

```
2019-03-20 11:21:02 [changch@hqdc034 ~]$ openssl s_client -showcerts -connect mail.abc.com:993
CONNECTED(00000003)
depth=0 C = TW, ST = Taipei, L = Taipei, O = iredmail02.abc.com, OU = IT, CN = iredmail02.abc.com, emailAddress = root@iredmail02.abc.com
verify error:num=18:self signed certificate
verify return:1
depth=0 C = TW, ST = Taipei, L = Taipei, O = iredmail02.abc.com, OU = IT, CN = iredmail02.abc.com, emailAddress = root@iredmail02.abc.com
verify return:1
---
Certificate chain
 0 s:/C=TW/ST=Taipei/L=Taipei/O=iredmail02.abc.com/OU=IT/CN=iredmail02.abc.com/emailAddress=root@iredmail02.abc.com
   i:/C=TW/ST=Taipei/L=Taipei/O=iredmail02.abc.com/OU=IT/CN=iredmail02.abc.com/emailAddress=root@iredmail02.abc.com
-----BEGIN CERTIFICATE-----
MIIEQTCCAymgAwIBAgIJAJksSxoDSMTLMA0GCSqGSIb3DQEBCwUAMIG2MQswCQYD
VQQGEwJUVzEPMA0GA1UECAwGVGFpcGVpMQ8wDQYDVQQHDAZUYWlwZWkxIzAhBgNV
BAoMGmlyZWRtYWlsMDIuZW1hdHRlcnMuY29tLnR3MQswCQYDVQQLDAJJVDEjMCEG
A1UEAwwaaXJlZG1haWwwMi5lbWF0dGVycy5jb20udHcxLjAsBgkqhkiG9w0BCQEW
H3Jvb3RAaXJlZG1haWwwMi5lbWF0dGVycy5jb20udHcwHhcNMTQwOTI0MTA1MjA2
WhcNMjQwOTIxMTA1MjA2WjCBtjELMAkGA1UEBhMCVFcxDzANBgNVBAgMBlRhaXBl
aTEPMA0GA1UEBwwGVGFpcGVpMSMwIQYDVQQKDBppcmVkbWFpbDAyLmVtYXR0ZXJz
LmNvbS50dzELMAkGA1UECwwCSVQxIzAhBgNVBAMMGmlyZWRtYWlsMDIuZW1hdHRl
cnMuY29tLnR3MS4wLAYJKoZIhvcNAQkBFh9yb290QGlyZWRtYWlsMDIuZW1hdHRl
cnMuY29tLnR3MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAvwNtZSdi
tDxSLNtFiIFv4Ww0+TwtuJAs6yUjQmfHa8CQDoSO/5EmUE2n4VA/78Qb/l/Si5tR
1FcTY2fifzXJfKkC5kMfoxkevyVqGCw3ZKku/QclaGunmLr/xCndbxbmx28T6PYJ
73ZrM9viRsE9Cn397IXZWOK+YFIxShhUFZrz1RxDk3cIU+MubBq7/A9Af8Izq9RK
w1vYK8+Jg81rOerQy71THxuUO5t+uFRzHuKGl8w8TnTt+Gnqx02UWFyHHtlVvhYV
Bi985OtvP0liApUQM7X4FPK/9cgNIlnXIi+SzQ2qtjqLNUHfO4P7GZ0IezIlFCHJ
ylx7GAj/tXm7HQIDAQABo1AwTjAdBgNVHQ4EFgQUgedf/rCzYD2FkR9C+kurh3Tj
2jcwHwYDVR0jBBgwFoAUgedf/rCzYD2FkR9C+kurh3Tj2jcwDAYDVR0TBAUwAwEB
/zANBgkqhkiG9w0BAQsFAAOCAQEAdFwoT/Y+76bDVxKn6ZW2hY0zcRe71MV4M6L1
AaObMEEBFg3C4oDZJTEyLItT2YNQ8SyNue9GstTwoZeMBiJ+sTt82f4d3KuaPynO
3ArnEGEektuaOs449eLHqLZVyGgMl7LMjThLntlTfBqD6cnmrcRpGdMUffCVAL9o
xzT9cgS6nOQIDo4uTPi7V8hqo+ioBzQXernomETW8TYocezw+XoaZB0rx4oo7uj9
zBih574GxNMcLreus5b8shEVWWlvReYkGjiHO70j818zOv4pmeJgNmkIEUW068TV
EybQFb6GZt3n3HWAbCc/ohaqUAoHc6S0P38xWRpLKyqvKEtjzw==
-----END CERTIFICATE-----
---
Server certificate
subject=/C=TW/ST=Taipei/L=Taipei/O=iredmail02.abc.com/OU=IT/CN=iredmail02.abc.com/emailAddress=root@iredmail02.abc.com
issuer=/C=TW/ST=Taipei/L=Taipei/O=iredmail02.abc.com/OU=IT/CN=iredmail02.abc.com/emailAddress=root@iredmail02.abc.com
---
No client certificate CA names sent
---
SSL handshake has read 1784 bytes and written 453 bytes
---
New, TLSv1/SSLv3, Cipher is ECDHE-RSA-AES256-GCM-SHA384
Server public key is 2048 bit
Secure Renegotiation IS supported
Compression: NONE
Expansion: NONE
SSL-Session:
    Protocol  : TLSv1.2
    Cipher    : ECDHE-RSA-AES256-GCM-SHA384
    Session-ID: 44738AF637D8531FE4554D30FC2D33428A17BD3B63C38C9D51EA18567D77F9F4
    Session-ID-ctx: 
    Master-Key: BEC2C876D0BE5066B7A1EC6BC06B8B72169FF6D6ADC77C45A080F6B7FCB23911134F815802BA80FC106C1E39F5FD28C3
    Key-Arg   : None
    PSK identity: None
    PSK identity hint: None
    SRP username: None
    TLS session ticket lifetime hint: 300 (seconds)
    TLS session ticket:
    0000 - 22 77 be c5 f2 e7 14 e1-b0 dc 1d 0b 00 a1 11 47   "w.............G
    0010 - a0 c5 b5 26 fb 15 b7 07-60 9a 79 8a a6 a5 45 77   ...&....`.y...Ew
    0020 - 74 de 1f 1e 8f b6 4d 29-66 6e 07 38 f3 d5 d8 35   t.....M)fn.8...5
    0030 - 2a 83 36 56 6f d7 0e c6-19 95 60 c5 3f 3c 9b 25   *.6Vo.....`.?<.%
    0040 - 75 0e 2a b6 57 cf 74 ad-36 e8 60 ee 37 30 ca e9   u.*.W.t.6.`.70..
    0050 - e1 42 b2 28 7e 03 df 1a-50 0c 31 ce d1 97 f2 84   .B.(~...P.1.....
    0060 - 2a 89 e1 c9 79 37 e1 37-9a f8 7f 8b 54 e0 ef 72   *...y7.7....T..r
    0070 - 4f 97 f1 92 24 b1 c5 9c-97 e1 03 cf 93 7b d8 e7   O...$........{..
    0080 - 72 6e 3d 33 a2 84 ea c3-9f 26 7f ae 99 29 88 04   rn=3.....&...)..
    0090 - 20 86 63 d2 7d ef b1 da-46 6f 3b 3c 4d dc 39 f2    .c.}...Fo;<M.9.

    Start Time: 1553052073
    Timeout   : 300 (sec)
    Verify return code: 18 (self signed certificate)
---
* OK [CAPABILITY IMAP4rev1 LITERAL+ SASL-IR LOGIN-REFERRALS ID ENABLE IDLE AUTH=PLAIN AUTH=LOGIN] Dovecot (Ubuntu) ready.

2019-03-20 11:21:47 [changch@hqdc034 ~]$ 
```

**測試SMTP TLS 587 port**

```
openssl s_client -starttls smtp -showcerts -connect mail.example.com:587
```

指令有點不同，要加上 startls smtp 的參數
回應應該會是這樣
```
2019-03-20 11:50:48 [changch@hqdc034 ~]$ openssl s_client -starttls smtp -showcerts -connect mail.abc.com:587
CONNECTED(00000003)
depth=1 C = US, O = Let's Encrypt, CN = Let's Encrypt Authority X3
verify error:num=20:unable to get local issuer certificate
verify return:0
---
Certificate chain
 0 s:/CN=mail.abc.com
   i:/C=US/O=Let's Encrypt/CN=Let's Encrypt Authority X3
-----BEGIN CERTIFICATE-----
MIIFXjCCBEagAwIBAgISA1Tg+poqmT+Vztc+/L2IPUHiMA0GCSqGSIb3DQEBCwUA
MEoxCzAJBgNVBAYTAlVTMRYwFAYDVQQKEw1MZXQncyBFbmNyeXB0MSMwIQYDVQQD
ExpMZXQncyBFbmNyeXB0IEF1dGhvcml0eSBYMzAeFw0xOTAyMjAyMzM0MjZaFw0x
OTA1MjEyMzM0MjZaMB8xHTAbBgNVBAMTFG1haWwuZW1hdHRlcnMuY29tLnR3MIIB
IjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAuhyXC0sCNccrqteycHst8ykZ
JSLxXB/07GBkvpdaCSKqTFAjtUh9IpkyDna1+IMtQIX+jeDWvd/GMJ3lj1pCgM+X
je850oFBp+HxMXTSDN67RMPbZsgAt9I938uigYmICHCqn4+FrnlfEOuX24326Y5Z
qEnguvZIdXYbQ5fzdMjP4W4Byu9cQAR+IUEWpFZ+hFNETAOVlNFb5kzsESWPTwIk
XWpDeQLQN6Iixf/ISXduJZSbOsFknAk7Dfrfine2cN9uYFCaiI2MJzrZ472Gbx+d
RnR6OmCinJ6Pb4yFQl2wQfGxnNyVCKTGu8JC+DM+j/Jfg1YYkxF4r/RdepPOTQID
AQABo4ICZzCCAmMwDgYDVR0PAQH/BAQDAgWgMB0GA1UdJQQWMBQGCCsGAQUFBwMB
BggrBgEFBQcDAjAMBgNVHRMBAf8EAjAAMB0GA1UdDgQWBBSaA9/Ki7j6XwUFm5af
X4WYJQNFIDAfBgNVHSMEGDAWgBSoSmpjBH3duubRObemRWXv86jsoTBvBggrBgEF
BQcBAQRjMGEwLgYIKwYBBQUHMAGGImh0dHA6Ly9vY3NwLmludC14My5sZXRzZW5j
cnlwdC5vcmcwLwYIKwYBBQUHMAKGI2h0dHA6Ly9jZXJ0LmludC14My5sZXRzZW5j
cnlwdC5vcmcvMB8GA1UdEQQYMBaCFG1haWwuZW1hdHRlcnMuY29tLnR3MEwGA1Ud
IARFMEMwCAYGZ4EMAQIBMDcGCysGAQQBgt8TAQEBMCgwJgYIKwYBBQUHAgEWGmh0
dHA6Ly9jcHMubGV0c2VuY3J5cHQub3JnMIIBAgYKKwYBBAHWeQIEAgSB8wSB8ADu
AHUA4mlLribo6UAJ6IYbtjuD1D7n/nSI+6SPKJMBnd3x2/4AAAFpDXmYTAAABAMA
RjBEAiBwIf9s7zBBuh/HUso4GWcZcAaEp2LsqT3xuJNOt7EWDAIgXE+u1JRVVEit
vi0PWIyfmUhBV8HP9GZ7r7uv2/9vGoYAdQBj8tvN6DvMLM8LcoQnV2szpI1hd4+9
daY4scdoVEvYjQAAAWkNeZiVAAAEAwBGMEQCIAZHUtzIl+6UvusG0cpYTiN5cFav
hzFQMoGeA5L5bSRvAiARX/+xnhtF9lUcXOsJfST1Eghk4Gm7QGno8vbcD3JHnzAN
BgkqhkiG9w0BAQsFAAOCAQEAI3buq7vydxeDpwSYVToAockocOMwXTyORIo6tACg
Z8gmVVTU6jliwrGWGzsjGh/V0gU+beDtlKzJpsYqbAWGOne1uuuKoH++lHO1KrrB
Br0R+G/UlIhRIZuSCdUGylWkV1ZQbHPtfw3MBB4nNj6wU0/tluxrqHJme/tyRSCs
WiP/DSfzIUkGUSSXyd3oEjwBdmq7NAmck3FK24HgBcJ3PSExChw4+TsEmGicpCKP
Q5AKt+n4b52tnp51P7rBHSFdmNL4AZiiQtZNXVmAQ2GI9zCwKguQg2c3YJ2oF39m
1GnjpLMRQkv5x6GQi7rglwB9ROH92Lg+uSD1vzjfiFwQeQ==
-----END CERTIFICATE-----
 1 s:/C=US/O=Let's Encrypt/CN=Let's Encrypt Authority X3
   i:/O=Digital Signature Trust Co./CN=DST Root CA X3
-----BEGIN CERTIFICATE-----
MIIEkjCCA3qgAwIBAgIQCgFBQgAAAVOFc2oLheynCDANBgkqhkiG9w0BAQsFADA/
MSQwIgYDVQQKExtEaWdpdGFsIFNpZ25hdHVyZSBUcnVzdCBDby4xFzAVBgNVBAMT
DkRTVCBSb290IENBIFgzMB4XDTE2MDMxNzE2NDA0NloXDTIxMDMxNzE2NDA0Nlow
SjELMAkGA1UEBhMCVVMxFjAUBgNVBAoTDUxldCdzIEVuY3J5cHQxIzAhBgNVBAMT
GkxldCdzIEVuY3J5cHQgQXV0aG9yaXR5IFgzMIIBIjANBgkqhkiG9w0BAQEFAAOC
AQ8AMIIBCgKCAQEAnNMM8FrlLke3cl03g7NoYzDq1zUmGSXhvb418XCSL7e4S0EF
q6meNQhY7LEqxGiHC6PjdeTm86dicbp5gWAf15Gan/PQeGdxyGkOlZHP/uaZ6WA8
SMx+yk13EiSdRxta67nsHjcAHJyse6cF6s5K671B5TaYucv9bTyWaN8jKkKQDIZ0
Z8h/pZq4UmEUEz9l6YKHy9v6Dlb2honzhT+Xhq+w3Brvaw2VFn3EK6BlspkENnWA
a6xK8xuQSXgvopZPKiAlKQTGdMDQMc2PMTiVFrqoM7hD8bEfwzB/onkxEz0tNvjj
/PIzark5McWvxI0NHWQWM6r6hCm21AvA2H3DkwIDAQABo4IBfTCCAXkwEgYDVR0T
AQH/BAgwBgEB/wIBADAOBgNVHQ8BAf8EBAMCAYYwfwYIKwYBBQUHAQEEczBxMDIG
CCsGAQUFBzABhiZodHRwOi8vaXNyZy50cnVzdGlkLm9jc3AuaWRlbnRydXN0LmNv
bTA7BggrBgEFBQcwAoYvaHR0cDovL2FwcHMuaWRlbnRydXN0LmNvbS9yb290cy9k
c3Ryb290Y2F4My5wN2MwHwYDVR0jBBgwFoAUxKexpHsscfrb4UuQdf/EFWCFiRAw
VAYDVR0gBE0wSzAIBgZngQwBAgEwPwYLKwYBBAGC3xMBAQEwMDAuBggrBgEFBQcC
ARYiaHR0cDovL2Nwcy5yb290LXgxLmxldHNlbmNyeXB0Lm9yZzA8BgNVHR8ENTAz
MDGgL6AthitodHRwOi8vY3JsLmlkZW50cnVzdC5jb20vRFNUUk9PVENBWDNDUkwu
Y3JsMB0GA1UdDgQWBBSoSmpjBH3duubRObemRWXv86jsoTANBgkqhkiG9w0BAQsF
AAOCAQEA3TPXEfNjWDjdGBX7CVW+dla5cEilaUcne8IkCJLxWh9KEik3JHRRHGJo
uM2VcGfl96S8TihRzZvoroed6ti6WqEBmtzw3Wodatg+VyOeph4EYpr/1wXKtx8/
wApIvJSwtmVi4MFU5aMqrSDE6ea73Mj2tcMyo5jMd6jmeWUHK8so/joWUoHOUgwu
X4Po1QYz+3dszkDqMp4fklxBwXRsW10KXzPMTZ+sOPAveyxindmjkW8lGy+QsRlG
PfZ+G6Z6h7mjem0Y+iWlkYcV4PIWL1iwBi8saCbGS5jN2p8M+X+Q7UNKEkROb3N6
KOqkqm57TH2H3eDJAkSnh6/DNFu0Qg==
-----END CERTIFICATE-----
---
Server certificate
subject=/CN=mail.abc.com
issuer=/C=US/O=Let's Encrypt/CN=Let's Encrypt Authority X3
---
No client certificate CA names sent
---
SSL handshake has read 3279 bytes and written 456 bytes
---
New, TLSv1/SSLv3, Cipher is ECDHE-RSA-AES256-GCM-SHA384
Server public key is 2048 bit
Secure Renegotiation IS supported
Compression: NONE
Expansion: NONE
SSL-Session:
    Protocol  : TLSv1.2
    Cipher    : ECDHE-RSA-AES256-GCM-SHA384
    Session-ID: F9FF6D75A3923A3B4E2CFBECC3DEAE2402FA99DA2B0746B6A2BF7E7E49F2E4D6
    Session-ID-ctx: 
    Master-Key: 62D845C4754634DCDAE1AB544B06929504C6310B3CD6E4512E3718535D871F1115A78DB46FA608159F52169DE4D9E12F
    Key-Arg   : None
    PSK identity: None
    PSK identity hint: None
    SRP username: None
    Start Time: 1553053873
    Timeout   : 300 (sec)
    Verify return code: 20 (unable to get local issuer certificate)
---
250 DSN

```

**測試 SMTP SSL port 465**
指令
<pre>
openssl s_client -showcerts -connect mail.example.com:465
</pre>

不過因為我沒開這個port ，所以就不測試了

