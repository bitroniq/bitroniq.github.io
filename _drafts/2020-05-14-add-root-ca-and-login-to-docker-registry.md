# How to install self-signet root CA and allow to login to Docker Registry

First we need get and import Root CA of/from our local registry server.

There is a bug in Docker project (5 years now) so
`--insecure-registry` doesn't work.

* https://github.com/moby/moby/issues/8849#issuecomment-584835957

So we need to do steps on Ubuntu/Debian:
1. Copy CA cert to `/usr/local/share/ca-certificates`
2. `sudo update-ca-certificates --verbose --fresh`
3. `sudo service docker restart`

This ca-cert should be located here:

```
ubuntu@src:~/$ cat /usr/local/share/ca-certificates/my-root.crt
-----BEGIN CERTIFICATE-----
MIIC6DCCAkqgAwIBAgIJAIi060AAF2TrMAoGCCqGSM49BAMCMIGMMQswCQYDVQQG
EwJVUzETMBEGA1UECAwKTmV3IEplcnNleTEUMBIGA1UEBwwLSmVyc2V5IENpdHkx
GzAZBgNVBAoMEkFjcmV0byBDbG91ZCBDb3JwLjEQMA4GA1UECwwHUm9vdCBDQTEj
MCEGA1UEAwwaQWNyZXRvIENsb3VkIENvcnAuIFJvb3QgQ0EwHhcNMTkwNzMwMTAw
MDI1WhcNMzQwNzI2MTAwMDI1WjCBjDELMAkGA1UEBhMCVVMxEzARBgNVBAgMCk5l
dyBKZXJzZXkxFDASBgNVBAcMC0plcnNleSBDaXR5MRswGQYDVQQKDBJBY3JldG8g
Q2xvdWQgQ29ycC4xEDAOBgNVBAsMB1Jvb3QgQ0ExIzAhBgNVBAMMGkFjcmV0byBD
bG91ZCBDb3JwLiBSb290IENBMIGbMBAGByqGSM49AgEGBSuBBAAjA4GGAAQAQfru
FiPb0xrz2rD40q77WgcT1zI5sz7X5q6HEcAyUNArYWV/9X380H61kRCagMbNgNQW
eXQtk745nNJlLamuOXsAKqHp0X1HCJsuPjmF9BFiXBV4ucTqkTbw4dhY8pkPiPHs
SAaYVrWh0sfGs72QcDMKoeBfCda80OfRFC4MY0ik9DqjUDBOMB0GA1UdDgQWBBRB
fWOAea3KAyr3Vws4OGhM744BnjAfBgNVHSMEGDAWgBRBfWOAea3KAyr3Vws4OGhM
744BnjAMBgNVHRMEBTADAQH/MAoGCCqGSM49BAMCA4GLADCBhwJCAN2UuepyyrTr
/8iis2/W/dtzqvOle4H7+XOR32BpjNmLYhREeAyQtJTBk3Uyo9GtsoiDbnHp2v3V
HrBFeF7gzAWjAkFbTdDfDcCBP3n7zNoR2+3hEJqFIZw9UCi7hR9dDPPEplXZ/p4C
Tw2bDOqT8dsnskvVu+JeKw0Clr1vwmq7ENKVBg==
-----END CERTIFICATE-----
```

Listing below shows a one-liner that allows to download root CA**,
update local certificates to install the CA and restart docker.

```shell
 openssl s_client \
  -showcerts \
  -connect infra-aws1-gitlab-01.infra-aws1.ns.acreto.net:4567 \
   </dev/null \
  | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' \
  | sudo tee /usr/local/share/ca-certificates/acreto-root.crt

sudo update-ca-certificates --verbose --fresh

sudo service docker restart
```

# References
