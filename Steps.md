## Update ##

sudo zypper update

## Install Go ## - done

sudo zypper addrepo https://download.opensuse.org/repositories/devel:/languages:/go/15.6/devel:languages:go.repo
sudo zypper refresh
sudo zypper in go go-doc
go env | grep GOENV

## Install docker-compose standalone

curl -SL https://github.com/docker/compose/releases/download/v2.35.0/docker-compose-linux-x86_64 -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose

## Install icdiff ##

sudo zypper in python3-pip
python3 -m pip install icdiff

python3 -m icdiff <file1> <file2>

## Install SqLite ##

sudo zypper in sqlite3
sqlite3 --version

## Install Syft ##

curl -sSfL https://raw.githubusercontent.com/anchore/syft/main/install.sh | sudo sh -s -- -b /usr/local/bin
syft --version

## Install grype ##

curl -sSfL https://raw.githubusercontent.com/anchore/grype/main/install.sh | sudo sh -s -- -b /usr/local/bin
grype version

## Install Trivy ##

curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sudo sh -s -- -b /usr/local/bin v0.61.0
trivy --version

## Install Docker ##

sudo zypper -n in docker
sudo systemctl start docker
sudo systemctl enable docker
sudo gpasswd -a "${USER}" docker
sudo reboot

## Install git ## - done

sudo zypper in git

## Check an image with docker-bench ##

### Build a docker image ###

docker build . -t opensuse/leap:latest -m 256mb --no-cache=true

## Using docker bench ##

git clone https://github.com/aquasecurity/docker-bench.git
go build -o docker-bench 
./docker-bench help
./docker-bench --include-test-output >docker-bench.txt
cat docker-bench.txt | grep FAIL

#### Select three findings ####
    * 2.9 Enable user namespace support
    * 5.26 Ensure that the container is restricted from acquiring additional privileges
    * 2.7 Ensure TLS authentication for Docker daemon is configured (Manual)

From instructor:
[FAIL] 5.10 Ensure that the memory usage for container is limited (Scored)
[FAIL] 5.14 Ensure that the 'on-failure' container restart policy is set to '5' (Scored)
[FAIL] 5.22 Ensure that docker exec commands are not used with the privileged option (Scored)

#### Remidate findings ####
4.5 Ensure Content trust for Docker is Enabled (Manual)

export DOCKER_CONTENT_TRUST=1
echo $DOCKER_CONTENT_TRUST

Check if container un in privileged mode:
docker ps --quiet --all | xargs docker inspect --format '{{ .Id }}: Privileged={{ .HostConfig.Privileged }}'


5.10 Ensure that the memory usage for container is limited

docker run -u --interactive --tty --memory 256mb opensuse/leap /bin/bash
docker inspect --format='{{.Config.Memory}}' <foo>    # foo is the image identifier for example 634c53ae1eb6


5.14 Ensure that the 'on-failure' container restart policy is set to '5'

docker run -u --detach --restart=on-failure:5 opensuse/leap
nick.reva@nreva-mbp % docker run --detach --restart=on-failure:5 opensuse/leap
235133ccd54c8c2ee797ade2e4a13ea04c876ee0ed03b2d197eb553c5211104c

5.22 Ensure that docker exec commands are not used with the privileged option (Scored)

There are no hardening steps here, just make sure to not use the --privileged flag as it's very dangerous.
- use the flag --user (or –u) in the docker run command

#### Docker push ####

docker push <your_username>/udacitysecurity:hardened-v1.0

Display sha: docker images --digests | grep <your_username>/udacitysecurity:hardened-v1.0

## Setup Docker notary server ##

git clone https://github.com/theupdateframework/notary.git
cd notary
mkdir -p /home/<username>/.docker/trust
docker trust key generate <name of the key> --dir ~/.docker/trust
docker-compose up -d
docker trust signer add --key ~/.docker/trust/<key directory>.pub <name of the key> <name of docker hub repo>
export DOCKER_CONTENT_TRUST_SERVER=https://notary.docker.io

docker trust sign nickreva/udacitysecurity:hardened-v1.0
export DOCKER_CONTENT_TRUST=1
docker push nickreva/udacitysecurity:hardened-v1.0
docker pull <your_username>/udacitysecurity:hardened-v1.0
docker trust inspect --pretty nickreva/udacitysecurity:hardened-v1.0