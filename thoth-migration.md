# Migration Operation

Migration of Thoth-station would require migration of thoth-station module and thoth-station bots.

## Thoth Bots Migration

We have a few of our very helpful cyborg team members helping thoth-team with our Thoth project development.

- **Kebechet**: keeps all the thoth-station repos fresh with appropriate packages.
- **Nepthys**: keeps documentation of the thoth-station fresh.
- **Sesheta Webhook**: keeps thoth-team updated with work done on Thoth-station Github.
- **Sesheta Scrum Bot**: keeps thoth-team updated with scrum meeting reminders.
- **Sesheta Label checker**: keeps updating new repos in github with the custom labels.
- **Sesheta Approver**: updates approved label to an open pull request if the ci test is successful.
- **Review Manager**:
- **Webhook2kafka & Normalizers**: updates Kafka topics for thoth-team

### Cyborg teams Deployment

Thoth Bots are to deployed in `aicoe-prod-bots` namespace.

**Kebechet**: Kebechet is a SourceOps bot that automates updating dependencies of the thoth-station.

- Source code for Kebechet bot could be found at: [thoth-station/kebechet](https://github.com/thoth-station/kebechet)
- Kebechet Secrets: Kebechet ssh secrets are in GOPASS. (cmd: `gopass show aicoe/thoth/kebechet/kebechet_github`)
- Deployment code: Use the ansible provision playbook from the [thoth-station/kebechet](https://github.com/thoth-station/kebechet) repository.

**Nepthys**: Nepthys is a SourceOps bot that automates documentation of the thoth-station.

- Source code for Nepthys could be found at [thoth-station/nepthys](https://github.com/thoth-station/nepthys)
- Nepthys uses kebechet ssh secrets.
- Deployment code: Use the ansible provision playbook from the [thoth-station/nepthys](https://github.com/thoth-station/nepthys) repository

**Sesheta**:

- Sesheta webhook bot is used for updating the thoth-team on the operations and works thats been done on the thoth-station Github and alerts of thoth metrics monitored by Prometheus via posts on Google Chat.
- Sesheta scrum bot is keeping the thoth-team updated with scrum meeting reminders.
- Sesheta label checker keeps updating new repositories in the thoth-station github with the custom labels.
- Sesheta Approver updates approved label to an open pull request if continuous integration tests are successful.

All Sesheta bot can be deployed in one go, if the requirement is to deploy individual bot, please use suitable tasks in the ansible provision script.

**NOTE**: Sesheta needs a configuration file with google chat developer information to be able to raise messages to a repository. Please make sure to create a file `sesheta-chatbot-968e13a86991.json` in the sesheta directory before running ansible provision playbook, get the content of the file from gopass. It could be found by:<br>
`gopass show aicoe/thoth/sesheta-chatbot-968e13a86991`

- Source code for Sesheta bot could be found at: [AICOE/sesheta](https://github.com/AICoE/sesheta)
- Sesheta Secrets: Few secrets are in ansible-vault and sesheta ssh secrets are in GOPASS.(cmd: `gopass show aicoe/thoth/sesheta-ssh/sesheta`)
- Deployment code: Use the ansible provision playbook from the [AICOE/sesheta](https://github.com/AICoE/sesheta) repository.

```
ansible-playbook \
   --extra-vars=OCP_URL=<openshift_cluster_url> \
   --extra-vars=OCP_TOKEN=<openshift_cluster_token> \
   --extra-vars=SESHETA_CONFIG_FILE=<config_json> \
   --extra-vars=SESHETA_SCRUM_SPACE=<google-chat-space> \
   --extra-vars=SESHETA_SCRUM_URL=<scurm_meeting_url> \
   --extra-vars=SESHETA_APPLICATION_NAMESPACE=<openshift_cluster_namespace> \
   --extra-vars=SESHETA_GITHUB_OAUTH_TOKEN=<github_oauth_token> \
   --extra-vars=SESHETA_SSH_PRIVATE_KEY_PATH=<github_ssh_private_key_path> \
   --extra-vars=SESHETA_GITHUB_WEBHOOK_SECRET=<github_webhook_secret> \
   --extra-vars=SESHETA_GOOGLE_CHAT_ENDPOINT_URL=<google_chat_incoming_webhook_url> \
   playbooks/provision.yaml --vault-password-file=.vault_pass
```

**Review Manager**: Contact the thoth-station admin for information on the deployment of the review manager.

**Webhook2kafka & Normalizers**: Webhook2kafka is a SourceOps bot that forwards the work by the developer done on Github and Trello to kafka topics. It would be used for assigning badges for the constructive work of the developers.

- Source code for Webhook2kafka & Normalizers could be found at: [cyborg-regidores](https://github.com/goern/cyborg-regidores)
- Secrets: secrets are in ansible-vault
- Deployment code: Use the ansible playbook from the [cyborg-regidores playbook](https://github.com/goern/cyborg-regidores/tree/master/playbooks) repository

  ```
  ansible-playbook --extra-vars="APPLICATION_NAMESPACE=aicoe-prod-bots" --vault-password=.vault-pass playbooks/playbook.yaml
  ```

--------------------------------------------------------------------------------

### Ultrahooks Deployment

Ultrahooks are setup in `aicoe` namespace.

Ultrahook is used for connecting public webhook endpoints with the development environment, as our resource is deployed in a restricted network, public webhooks can not trigger directly.

- **ultrahook-cyborg-regidores**: Directs Webhooks for cyborg-regidores to service route of deployed cyborg-regidores on the aicoe-prod-bots namespace.

  - ULTRAHOOK_SECRET= ultrahook-aicoe
  - ULTRAHOOK_SUBDOMAIN=cyborg-regidores
  - ULTRAHOOK_DESTINATION='<http://webhook2kafka-aicoe-prod-bots.cloud.paas.psi.redhat.com/>'

- **ultrahook-webhooks**: Directs Webhooks for sesheta-webhook to service route of deployed sesheta-webhook on the aicoe-prod-bots namespace.

  - ULTRAHOOK_SECRET= ultrahook-thoth
  - ULTRAHOOK_SUBDOMAIN=webhooks
  - ULTRAHOOK_DESTINATION='<http://sesheta-webhooks-aicoe-prod-bots.cloud.paas.psi.redhat.com>'

- **ultrahook-review-manager**: Directs Webhooks for review-manager to service route of deployed review-manager on the aicoe-prod-bots namespace.

  - ULTRAHOOK_SECRET= ultrahook-aicoe
  - ULTRAHOOK_SUBDOMAIN=review-manager
  - ULTRAHOOK_DESTINATION='<http://review-manager-webhooks-aicoe-prod-bots.cloud.paas.psi.redhat.com/github>'

- **ultrahook-zuul-thoth-test-core**: Directs Webhooks for zuul to service route of deployed zuul on the different namespace.

  - ULTRAHOOK_SECRET= ultrahook-aicoe
  - ULTRAHOOK_SUBDOMAIN=zuul-thoth-test-core
  - ULTRAHOOK_DESTINATION='<https://managesf.thoth-station.ninja>'

Source Code for Ultrahook can be found at [Ultrahook](https://github.com/AICoE/ultrahook-openshift)<br>

The thoth-station public webhook is created with Subdomain, Destination and a Secret.

- Use the [ultrahook secret template](https://github.com/AICoE/ultrahook-openshift/blob/master/openshift/ultrahook.secret.yaml) to create the required secrets.
- Use `gopass show aicoe/thoth/ultrahook` to get ULTRAHOOK_API_KEY for different secrets.

  - Edit [secret-name](https://github.com/AICoE/ultrahook-openshift/blob/72cb24bd5bd604f126d9bf53eacdfdaab8661ed0/openshift/ultrahook.secret.yaml#L9) to ultrahook-aicoe and pass ULTRAHOOK_API_KEY of the ultrahook-aicoe present in gopass with the following code.
  - Edit [secret-name](https://github.com/AICoE/ultrahook-openshift/blob/72cb24bd5bd604f126d9bf53eacdfdaab8661ed0/openshift/ultrahook.secret.yaml#L9) to ultrahook-thoth and pass ULTRAHOOK_API_KEY of the ultrahook-thoth present in gopass with the following code.

  `oc process -p ULTRAHOOK_API_KEY={{ ULTRAHOOK_API_KEY }} -f openshift/ultrahook.secret.yaml --namespace aicoe | oc apply -f - --namespace aicoe`

Deployment the ultrahook with the following code and passing appropriate values to the parameter for each ultrahook:

```
oc process -p ULTRAHOOK_SUBDOMAIN=<subdomain> -p ULTRAHOOK_DESTINATION=<webhook> -f openshift/ultrahook.app.yaml --namespace aicoe | oc apply -f - --namespace aicoe
```

### Bot Secret Keys Generation

**NOTE**: The Secret key for thoth bots are already created and are present in `gopass aicoe/thoth`. Generate new secrets only when necessary and update them in gopass along with informing the team about the new changes.

**Sesheta**:<br>
sesheta has a standard account in github and bot account in google chat. Sesheta _credentails_, _ssh keys_ and _github oauth token_ are used by **sesheta**, **kebechet**, and **nepthys** for thoth-station.

_Generation of keys_: use sesheta credentails present in GOPASS to login to a sesheta github account and generate neccessary keys and token. `gopass show aicoe/thoth/sesheta`<br>
**NOTE**: Do Not forget to update the new keys to GOPASS and informing the team about the changes.

**Ultrahooks**:<br>
All the ultrahook for webhook have been already created and ultrahook api key is stored in GOPASS.<br>
use `gopass show aicoe/thoth/ultrahook`<br>
_Generation of keys_: To generate ultrahook api key for new webhook, use [Ultrahook Client](http://www.ultrahook.com/register) and required credentails are present in GOPASS

--------------------------------------------------------------------------------

## Thoth Station Migration

Thoth Station is to be setup in environments: `stage` and `test`.

- The **stage** environment whole deployment is divided into multiple namespaces (or OpenShift projects) - thoth-frontend, thoth-middletier, thoth-backend, inspection and tensorflow build.<br>
  [thoth-station Architecture Diagram](https://raw.githubusercontent.com/thoth-station/core/master/doc/architecture.png)<br>
  Details about the setup itself, please read [thoth-station Artichecture](https://github.com/thoth-station/core#architecture-overview)
- The **test** environment deployment is amalgamation of all stage namespace into one test namespace `thoth-test-core`.

### Deployement

The Thoth-station deployment in OpenShift is managed by thoth-ops. The playbooks in the [thoth-ops](https://github.com/thoth-station/thoth-ops/tree/master/ansible) repository can be used to easily deploy and manage the thoth-station module.

The deployment of the thoth-station module involves creating the template, deploymentconfigs, buildconfigs, imagestreams, jobs, and cronjobs in openshift along with setting up appropriate service accounts, routes, configmaps, secrets, and services.

The ansible playbook [provision](https://github.com/thoth-station/thoth-ops/blob/master/ansible/provision_stage_environment.yaml) and [prepare](https://github.com/thoth-station/thoth-ops/blob/master/ansible/prepare_environment.yaml) environment are developed to deploy the appropriate module to their specific namespaces/project and to rolebind the service account accordingly. The ansible playbook uses the ansible vault to secure the secret.

**NOTE**: Ansible vault password is present in GOPASS.<br>
`gopass show aicoe/thoth/ansible-vault-password`

Familiarize with thoth-ops and how are ansible playbook executes.

- [Using the Ops Container](https://github.com/thoth-station/thoth-ops#using-the-ops-container)
- [Stage Operations](https://github.com/thoth-station/thoth-ops/blob/master/docs/stage_operations.adoc)

**NOTE**: stable images of thoth-modules present in quay.io are used by **stage** environment but no builds are explicitly triggered in **stage** environment.

- Step for Migration of **stage** env:

  - Make sure [thoth_environment](https://github.com/thoth-station/thoth-ops/blob/ab168d587d3db67d3c4ff411ced55fff97f4b622/ansible/provision_stage_environment.yaml#L14) variable is set to **stage**.
  - Make sure for your username you have _admin_ or _edit_ access rights the stage environment namespaces.
  - Execute [prepare](https://github.com/thoth-station/thoth-ops/blob/master/ansible/prepare_environment.yaml) ansible playbook to setup the environment with required service account, configmaps, secrets using [Stage Operations](https://github.com/thoth-station/thoth-ops/blob/master/docs/stage_operations.adoc).
  - Execute [provision](https://github.com/thoth-station/thoth-ops/blob/master/ansible/provision_stage_environment.yaml) ansible playbook to deploy the thoth-module to there respective namespaces using [Stage Operations](https://github.com/thoth-station/thoth-ops/blob/master/docs/stage_operations.adoc).
  - Verify the deployment by executing [verfiy](https://github.com/thoth-station/thoth-ops/blob/master/ansible/verify_environment.yaml) playbook using [Stage Operations](https://github.com/thoth-station/thoth-ops/blob/master/docs/stage_operations.adoc)

- Step for Migration of **test** env:

  - Make sure [thoth_environment](https://github.com/thoth-station/thoth-ops/blob/ab168d587d3db67d3c4ff411ced55fff97f4b622/ansible/provision_stage_environment.yaml#L14) variable is set to **test**.
  - Make sure for your username you have _admin_ or _edit_ access rights to test environment namespace.
  - The ansible vault [vars](https://github.com/thoth-station/thoth-ops/blob/master/ansible/group_vars/all/vars):

    - backend_namespace
    - middletier_namespace
    - frontend_namespace
    - infra_namespace
    - amun_api_namespace
    - amun_inspection_namespace<br>
      are to be set to **thoth-test-core** to point to test environment

  - Execute [prepare](https://github.com/thoth-station/thoth-ops/blob/master/ansible/prepare_environment.yaml) ansible playbook to setup the environment with required service account, configmaps, secrets using [Stage Operations](https://github.com/thoth-station/thoth-ops/blob/master/docs/stage_operations.adoc).

  - Execute [provision](https://github.com/thoth-station/thoth-ops/blob/master/ansible/provision_stage_environment.yaml) ansible playbook to deploy the thoth-module to there respective namespaces using [Stage Operations](https://github.com/thoth-station/thoth-ops/blob/master/docs/stage_operations.adoc).

  - Verify the deployment by executing [verfiy](https://github.com/thoth-station/thoth-ops/blob/master/ansible/verify_environment.yaml) playbook using [Stage Operations](https://github.com/thoth-station/thoth-ops/blob/master/docs/stage_operations.adoc)

  - Once the setup is successfully executed copy the secret token of service account **thoth-ops** from **thoth-test-core** namespace secrets in openshift and update that to **vault_thoth_test_core_registry_access_token** variable in ansible-vault.

**NOTE**: Deployments like workload operator are openshift event based application i.e they watch the events in openshift to proceed with the further execution. There is a bug in the python sdk which caused stopping receiving of events in the cluster after some 30mins so operators are to be restarted. We have a patch, which will enable the deployment to restart the deployment so that they start to receive events. To add this patch to relevant deployments follow the steps:

- Clone the [thoth-station/misc](https://github.com/thoth-station/misc)
- oc login to OpenShift cluster where deployment is done.
- we need to run `./auto-kill.sh <args>` script present in the misc repo.
- The argument to pass are **deployment_name**, for example, do:

  ```
  - `oc project thoth-frontend-stage`
  - `./autokill_patch.sh workload-operator-thoth-middletier-stage workload-operator-thoth-backend-stage`
  ```

- Deployment which is event based is **workload-operator** and **backend-operator**.

### Dgraph

Thoth uses Dgraph for a graph database. The dgraph is deployment in both environment **stage** and **test** for respective usage.

The Deployment of the dgraph uses openshift templates present in [dgraph](https://github.com/thoth-station/dgraph-thoth-config).

Dgraph certificates in a test environment would have automatically setup with setup of the test environment with the thoth-ops ansible playbook. Dgraph certificates for stage environment can be setup using the individual task of [thoth-ops prepare](https://github.com/thoth-station/thoth-ops/blob/ab168d587d3db67d3c4ff411ced55fff97f4b622/ansible/prepare_environment.yaml#L224) playbook in the required namespace.

**Certificates generation for dgraph**:<br>
The dgraph certificates are already present in ansible-vault. If new certificates are to be generated use the following command with appropriate hostname(check routes of dgraph to get the hostname) and user. `dgraph cert -n <all thoth-station dgraph hostname> -c user`

### Zuul
`Thoth` useds [`Zuul`](https://zuul-ci.org/) as its `CI/CD` tool for all the testing and pipelining jobs. Migrate `Zuul` is another step during the migration process of `thoth`. First, let cover some basics of `Zuul` in thoth. There is a `Zuul` server running on `OpenStack` which has all the data for `thoth` `Zuul` project. This [link](managesf.thoth-station.ninja) shows you which `Zuul` job is current avaiable on `thoth` `Zuul` project. To migrate `Zuul` to your new cluster, follow the steps to allow `Zuul` running its test on you new cluster.

**First Step**
Log in to the `Zuul` server and create a new `ServiceAccount` on the `Zuul` Server. Config your new `OpenShift` cluster to allow this `ServiceAccount` access the resource and `namespace` on the new cluster, so it can create context in the new cluster.

**Second Step**
At this step, we need to configure the `Zuul`'s `nodepool` configuration so `Zuul` can use the new cluster environment to do those tests. Go to this [repo](https://github.com/thoth-station/zuul-config/), then location the file under this path `zuul-config/nodepool/thoth.yaml`. Add the new image for `Zuul` which the new label follow the same format inside this file.
```
---
  labels:
    - name: "thoth-coala"
      min-ready: 1
    ...

  providers:
    - name: "psi"
      driver: openshiftpods
      context: "thoth-zuul/paas-psi-redhat-com:443/system:serviceaccount:thoth-zuul:nodepool"
      pools:
        - name: "thoth-zuul"
          labels:
            - name: "thoth-coala"
              image: "thoth-coala:ubi8"
              cpu: 1
              memory: 1024
            ...
```
Once the new `nodepool` configuration is merged, `Zuul` will automatically use the new `nodepool` configuration to create new `image` and `pod` in the new cluster for testing. To use the `pod` in new cluster for testing, make sure specify the new `label` in your local `.zuul.yaml` file.
