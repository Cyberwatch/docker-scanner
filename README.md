# Docker Scanner

This repository contains a script to easily configure a docker daemon
with the Cyberwatch application before allowing to scan for vulnerabilities
in docker images.

## Generate a PKI for the docker daemon

Run

```shell
generate-certs
```

This step will generate 2 files:

- certs/ca/cert.pem:
  This is the public certificate of the CA. It is required by both servers and
  clients to mutually verifiy their authenticy. It is not sensitive.
- certs/ca/key.pem:
  This is the private key of the CA. It is required for generating new server
  or client certificates. It is sensitive: anyone with this key may
  authenticate against your Docker runners.

## Upload the PKI on Cyberwatch (optional)

Run :

```shell
upload-certs
```

## References

- [Cyberwatch documentation](https://docs.cyberwatch.fr/en/2_use_assets/docker_images.html#adding-a-docker-image)
- [official Docker documentation](https://docs.docker.com/engine/security/https/)
