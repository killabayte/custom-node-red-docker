# Build your own Docker image

This directory contains files you need to build your own images.

The follow steps describe in short which steps to take to build your own images.

## 1. git clone

Clone the Node-RED Docker project from github
```shell script
git clone https://github.com/killabayte/custom-node-red-docker.git
```

Change dir to docker-custom
```shell script
cd custom-node-red-docker
```

## 1. **package.json**

   - Change the node-red version in package.json (from the docker-custom directory) to the version you require
   - Add optionally packages you require

## 2. **flows.json**

   - The `flows.json` file is the default flow that will be used if no external volume is mounted to `/data`. You can replace this by a preconfigured flow and launch it by not mounting a /data volume, but most users will mount and save data and flows externally.

## 3. **docker-alpine.sh**

The `docker-alpine.sh` is helper scripts to build a custom Node-RED docker image. The docker-alpine script is based on Alpine as per the default docker package.

Change the build arguments as needed:

   - `--build-arg ARCH=amd64` : architecture your are building for (arm32v6, arm32v7, arm64v8, amd64)
   - `--build-arg NODE_VERSION=10` : NodeJS version you like to use
   - `--build-arg NODE_RED_VERSION=${NODE_RED_VERSION}` : don't change this, ${NODE_RED_VERSION} gets populated from package.json
   - `--build-arg OS=alpine` : the linux distro to use (alpine)
   - `--build-arg BUILD_DATE="$(date +"%Y-%m-%dT%H:%M:%SZ")"` : don't change this
   - `--build-arg TAG_SUFFIX=default` : to build the default or minimal image
   - `--file Dockerfile.custom` : Dockerfile to use to build your image.
   - `--tag pilot:node-red-build` : set the image name and tag

## 4. **Run docker-alpine.sh**

Run `docker-alpine.sh`

```shell script
$ ./docker-alpine.sh
```

This starts building your custom image and might take a while depending on the system you are running on.

When building is done you can run the custom image by the following command:

```shell script
$ docker run -it -p 1880:1880 -e "NODE_RED_CREDENTIAL_SECRET=YOUR_BCRYPT_HASH" -v node_red_data:/data --name myNRtest pilot:node-red-build
```

Please note in setting.js enabled Admin auth and password must be provided as a dynamic variable, for more details please see [official documentation](https://nodered.org/docs/getting-started/docker#credentials-secrets-and-environment-variables)

With the following command you can verify your docker image:

```shell script
$ docker inspect pilot:node-red-build
```

## 5. **Advanced Configuration**

The relevant `Dockerfile` can be modified as required.

## 6. **Securing your node red instance**

To do that you will need to install a certificate and key on your node-red server.
In this tutorial we will be using a self signed certificate which we will create ourselves using openssl.

1.Create a private key

```shell script
openssl genrsa -out node-key.pem 2048
```

2. Create a certificate Request

```shell script
openssl req -new -sha256 -key node-key.pem -out node-csr.pem
```

You will need to fill out a form the most important entry is near the end and is the common name field.
This should be the FQDN of the server hosting nod-red or the IP address. I used *.nodered.test

3. Sign the Certificate with the Private key to create a self signed Certificate

```shell script
openssl x509 -req -in node-csr.pem -signkey node-key.pem -out node-cert.pem
```

To access your server this time  you have to go to https://127.0.0.1:1880

At some point, you will get annoyed by the prompt, so if you spend a few more minutes, you can add the SSL certificate to your browser.
Open the certification.pem file and copy the content to a text file on your computer. Save it using the same name. Then:

Open Chrome settings page chrome://settings
Search for “certificates” and add newly created node-cert.pem to trusted certificates

You are ready to roll a new server with the SSL certificate to NodeRED
