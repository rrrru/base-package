Base Helm Chart
This repository contains a Helm chart for deploying [Your Application] using Helm. The chart is designed to be hosted in a Nexus repository.

Prerequisites
Ensure that you have a Helm hosted repository set up in Nexus.
Create a Nexus user.
Create a role with write permissions for the Helm repository.
Assign the role to the created user.
Packaging and Uploading to Nexus
On your local machine or CI job:

```bash
# Add Helm repo
helm repo add ${helm_repo_name} "https://${your_domain}/repository/${helm_repo_name}/" --username ${nexus_user} --password ${nexus_pass}

# Update Helm repo
helm repo update

# Package the current version
helm package .

# Upload the version to Nexus
# Note: 'helm push' is unsupported, using curl instead
curl --insecure -v -F file=@base-package-0.0.1.tgz -u ${nexus_user}:${nexus_pass} "https://${your_domain}/service/rest/v1/components?repository=${helm_repo_name}"
```

#### Usage in CI/CD or Local Environment
### Local Environment
Add the remote repo and update:
```bash
helm repo add ${helm_repo_name} "https://${your_domain}/repository/${helm_repo_name}/" --username ${nexus_user} --password ${nexus_pass}
helm repo update
```

Save values.yaml:
```bash
helm show values ${helm_repo_name}/base-package > /tmp/values.yaml
```

Edit /tmp/values.yaml:
```bash
sed -i 's/tag: 0.6.0/tag: 0.7.0/' /tmp/values.yaml
```

Apply the chart (rollback if necessary with --atomic --timeout 1m):
```bash
helm upgrade --install --atomic --timeout 1m ${your_application_name} --wait ${helm_repo_name}/base-package -f /tmp/values.yaml \
--set nameOverride=${your_application_name} --set fullnameOverride=${your_application_name} \
-n ${your_namespace} --create-namespace --debug
```

Feel free to customize the chart values in /tmp/values.yaml according to your specific requirements.


#### You can also use this repository on GitHub as a Helm repository
https://rrrru.github.io/base-package

```bash
helm repo add base-package https://rrrru.github.io/base-package
helm repo update
helm show values base-package/base-package > /tmp/values.yaml

## use values.yaml
helm upgrade --install --atomic \
    --timeout 1m test-app --wait base-package/base-package -f /tmp/values.yaml \
-n default --create-namespace --debug --dry-run

## just set
helm upgrade -i test-app base-package/base-package \
    --namespace default \
    --set "ingress.hosts[0].host=test-app.example.com,ingress.hosts[0].paths[0].path=/" \
    --set "ingress.tls[0].hosts={test-app.example.com}" \
    --set "ingress.tls[0].secretName=letsencrypt-test-app" \
    --set "image.repository=ealen/echo-server" \
    --set "image.tag=0.6.0" \
    --set "service.port=80" \
    --set nameOverride=test-app --set fullnameOverride=test-app \
    --version 0.0.1 \
    --debug --dry-run
```

