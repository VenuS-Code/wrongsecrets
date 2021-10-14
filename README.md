# Wrong Secrets

[![Java checkstyle and testing](https://github.com/commjoen/wrongsecrets/actions/workflows/main.yml/badge.svg)](https://github.com/commjoen/wrongsecrets/actions/workflows/main.yml) [![Terraform FMT](https://github.com/commjoen/wrongsecrets/actions/workflows/terraform.yml/badge.svg)](https://github.com/commjoen/wrongsecrets/actions/workflows/terraform.yml) [![Test minikube script](https://github.com/commjoen/wrongsecrets/actions/workflows/minikube-test.yml/badge.svg)](https://github.com/commjoen/wrongsecrets/actions/workflows/minikube-test.yml)

Welcome to the Wrong Secrets p0wnable app. With this app, we have packed various ways of how to not store your secrets. These can help you to realize whether your secret management is ok. The challenge is to find all the different secrets by means of various tools and techniques.

Can you solve all the challenges?
![screenshot.png](screenshot.png)
## Basic docker exercises

For the basic docker exercises you currently require:

- Docker [Install from here](https://docs.docker.com/get-docker/)
- Some Browser that can render HTML

You can install it by doing:

```bash
docker run -p 8080:8080 jeroenwillemsen/addo-example:0.0.2-no-vault
```

Now you can try to find the secrets by means of solving the challenge offered at:

- [localhost:8080/challenge/1](http://localhost:8080/challenge/1)
- [localhost:8080/challenge/2](http://localhost:8080/challenge/2)
- [localhost:8080/challenge/3](http://localhost:8080/challenge/3)
- [localhost:8080/challenge/4](http://localhost:8080/challenge/4)
- [localhost:8080/challenge/5](http://localhost:8080/challenge/5)
- [localhost:8080/challenge/6](http://localhost:8080/challenge/6)
- [localhost:8080/challenge/7](http://localhost:8080/challenge/7)
- [localhost:8080/challenge/8](http://localhost:8080/challenge/8)

Note that these challenges are still very basic, and so are their explanations. Feel free to file a PR to make them look better ;-). [See this issue as well.](https://github.com/commjoen/wrongsecrets/issues/23)

## Basic K8s exercise

### Minikube based

Make sure you have the following installed:

- Docker [Install from here](https://docs.docker.com/get-docker/)
- Minikube [Install from here](https://minikube.sigs.k8s.io/docs/start/)

The K8S setup currently is based on using Minikube for local fun:

```bash
    minikube start
    kubectl apply -f k8s/secrets-config.yml
    kubectl apply -f k8s/secrets-secret.yml
    kubectl apply -f k8s/secret-challenge-deployment.yml
    while [[ $(kubectl get pods -l app=secret-challenge -o 'jsonpath={..status.conditions[?(@.type=="Ready")].status}') != "True" ]]; do echo "waiting for secret-challenge" && sleep 2; done
    kubectl expose deployment secret-challenge --type=LoadBalancer --port=8080
    minikube service secret-challenge
```

now you can use the provided IP address and port to further play with the K8s variant (instead of localhost).

### EKS Based

Follow the steps in [the README in the AWS subfolder](aws/README.md).

### k8s based

Want to run vanilla on your own k8s? Use the commands below:

```bash

    kubectl apply -f k8s/secrets-config.yml
    kubectl apply -f k8s/secrets-secret.yml
    kubectl apply -f k8s/secret-challenge-deployment.yml
    while [[ $(kubectl get pods -l app=secret-challenge -o 'jsonpath={..status.conditions[?(@.type=="Ready")].status}') != "True" ]]; do echo "waiting for secret-challenge" && sleep 2; done
    kubectl port-forward \
        $(kubectl get pod -l app=secret-challenge -o jsonpath="{.items[0].metadata.name}") \
        8080:8080

```

## Vault exercises with minikube

Make sure you have the following installed:

- minikube with docker (or comment out line 8 and work at your own k8s setup),
- docker,
- helm [Install from here](https://helm.sh/docs/intro/install/),
- kubectl [Install from here](https://kubernetes.io/docs/tasks/tools/),
- jq [Install from here](https://stedolan.github.io/jq/download/),
- vault [Install from here](https://www.vaultproject.io/downloads),
- grep, Cat, and Sed

Run `./k8s-vault-minkube-start.sh`, when the script is done, then the challenge will wait for you at <http://localhost:8080>

## Cloud-based exercise

Have a look at the [aws folder](aws/README.md), Note that we are [not done yet](https://github.com/commjoen/wrongsecrets/issues/15).

## Special thanks & Contributors

With thanks to [@madhuakula](https://github.com/madhuakula) for motivating me to set up the project, and thanks to [@bendehaan](https://github.com/bendehaan) & [@nbaars](https://github.com/nbaars) for contributing to the project.

## Help Wanted

You can help us by the following methods:

- Star us
- Share this app with others
- Of course, we can always use your help to get more flavors of "wrongly" configured secrets in to spread awareness! We would love to get some help with a Google Cloud or Azure integration for instance. Do you miss something else than a cloud provider as an example? File an issue or create a PR!

## Notes on development

If you want to test against vault without K8s: start vault locally with

```shell
 export VAULT_ADDR='http://127.0.0.1:8200'
 export VAULT_API_ADDR='http://127.0.0.1:8200'
 vault server -dev
 ```

and in your next terminal, do (with the token from the previous commands):

```shell
export VAULT_ADDR='http://127.0.0.1:8200'
export VAULT_TOKEN='<TOKENHERE>'
vault token create -id="00000000-0000-0000-0000-000000000000" -policy="root"
vault kv put secret/secret-challenge vaultpassword.password="$(openssl rand -base64 16)"
```

Now use the `local-vault` profile to do your development.

If you want to dev without a Vault instance, use the `without-vault` profile to do your development.

Want to push a container? See `.github/scripts/docker-create-and-push.sh` for a script that generates and pushes all containers. Do not forget to rebuild the app before composing the container
