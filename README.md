# Docker Scanner

This repository contains scripts to easily configure a docker daemon
with the Cyberwatch application before allowing to scan for vulnerabilities
in docker images.

Two bash scripts are available:

- `generate-certs`: used to generate the certificates;
- `upload-certs`: used to upload the generated certificates to Cyberwatch using the API with [cbw-api-auth module](https://github.com/Cyberwatch/httpie-cbw-api-auth).

## Generate a PKI for the docker daemon

Run the script:

```shell
./generate-certs
```

Follow the steps given by the script:
- specify subject alternative names;
- configure dockerd using the output given by the script.

This step will generate 2 files:

- `certs/ca/cert.pem`:
  This is the public certificate of the CA. It is required by both servers and
  clients to mutually verifiy their authenticy. **It is not sensitive**.
- `certs/ca/key.pem`:
  This is the private key of the CA. It is required for generating new server
  or client certificates. **It is sensitive: anyone with this key may
  authenticate against your Docker runners**.

At the end, the script will prompt if you would like to proceed to the automatic configuration wizard using the script `upload-certs`.

If you have already installed the cbw-api-auth module you can accept and proceed to automatically upload the generated certificates to Cyberwatch.
If not, you can decline and prepare the prerequisites to run the script later, or upload the generated PKI manually in Cyberwatch.

## Upload the PKI on Cyberwatch using the script (optional, can be done manually through your Cyberwatch web interface)

Run the script:

```shell
./upload-certs
```

This script will prompt for the following information:
- the Docker URL to reach your server;
- API URL of your Cyberwatch instance;
- API key of your Cyberwatch user;
- Secret key of your Cyberwatch user.

API credentials require full access, as described when executing the script.

If the credentials provided are correct and the cbw-api-auth module is correctly installed, a "Docker Engine" type stored credential will be created in your Cyberwatch interface.

## Configure and make sure the docker daemon is listening

N.B. Modifying docker configuration requires sudoer rights, the following procedure assumes these rights.

### Apply the configuration

First, make sure you have added the generated certificates in the file `/etc/docker/daemon.json`.

Then, apply the configuration and restart docker :

```shell
systemctl daemon-reload
systemctl restart docker
```

### (OPTIONNAL) Troubleshooting if docker cannot restart

A configuration problem can stop docker from restarting, as explained here: https://github.com/moby/moby/issues/22339#issuecomment-225437878
If you do encounter this problem, the following workaround can be used.

Create the file `/etc/systemd/system/docker.service.d/docker.conf`:

```shell
mkdir -p /etc/systemd/system/docker.service.d
cd /etc/systemd/system/docker.service.d

cat >> docker.conf <<EOL
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd
EOL
```

Then reload the configuration and restart docker again:

```shell
systemctl daemon-reload
systemctl restart docker
```

### Check if the docker daemon is listening

To make sure the docker daemon is listening, check that the port 2376 is open on the server after applying the configuration.

## References

- [Cyberwatch documentation](https://docs.cyberwatch.fr/en/2_use_assets/docker_images.html)
- [official Docker documentation](https://docs.docker.com/engine/security/https/)
