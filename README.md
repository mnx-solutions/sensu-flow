# sensuflow-action
Github Action for Sensu flow resource management pattern.

## Introduction
This github action will allow you to manage Sensu resources for a namespaces by making use of sensuctl prune and create commands. The `sensu prune` command is scope by the action's `workflow_label_selector` and `managed_resources`settings. Please see the sensuctl command documentation for information.

In order to use this action, you'll need to define a Sensu user and associated role based access control.

## Capabilities

### Namespace Resource Management
This action uses a special directory structure, mapping subdirectory names to Sensu namespaces to process. By default the directory processed is `namespaces/`  but this can be overridden in the action configuration. If this directory exists, each sub directory will be processed as separate Sensu namespace. Example directory structure:
```
namespaces
└── test-namespace
    ├── checks
    │   ├── check-cpu.yaml
    │   ├── check-http.yaml
    │   ├── false.yaml
    │   └── true.yaml
    ├── filters
    │   └── fatigue-check.yaml
    ├── handlers
    │   └── alert
    │       ├── aws-sns.yaml
    │       ├── pushover.yaml
    │       └── set.yaml
    └── mutators
        └── check-status.yaml
```

Using this example, this action would process the `test-namespace`, pruning the namespace resources according to `workflow_label_selector` and `managed_resources`settings

### Optionally Preparing namespaces
If the `namespaces.yaml` file exists in the working directory (normally the top level of your repository) then this action will be used to create sensu namespaces before attempting to process the namespaces directory.

## Configuration
### Required settings
#### sensu_backend_url 
  The Sensu backend url
#### sensu_user 
  The Sensu user to auth 
#### sensu_password
  The Sensu user password

### Optional settings
####  configure_args:
    description: "optional arguments to pass to sensuctl configure"
####  sensu_ca:
    description: 'Optional Custom CA pem string'
####  namespaces_dir:
    description: "Optional directory to process default: 'namespaces' "
####  workflow_label_selector:
    description: "Option Sensu label selector, default: 'sensu.io/workflow == sensu_flow'"
####  managed_resources:
    description: 'Optional comma seperated list of managed resources'

### Github Action Workflow Example
```
jobs:
  sensuflow:
    runs-on: ubuntu-latest

    steps:
    # Checks-out your repository under $GITHUB_WORKSPACE, so your job can access it
    - name: Checkout
      uses: actions/checkout@v2

    - name: Sensuflow with optional settings
      uses: jspaleta/sensuflow-action@v0.0.1
      with:
        sensu_backend_url: ${{ secrets.SENSU_BACKEND_URL }}
        sensu_user: ${{ secrets.SENSU_USER }}
        sensu_password: ${{ secrets.SENSU_PASSWORD }} 
        namespaces_dir: namespaces
        workflow_label_selector: "sensu.io/workflow == sensu_flow"

```
### RBAC Policy
Here is an example of an expansive Sensu RBAC policy and user definition  configured to use this action with default settings. You may want to use a more restrictive RBAC policy to meet your requirements. 

```
---
type: ClusterRole
api_version: core/v2
metadata:
  name: sensu_flow
spec:
  rules:
  - resources:
    - namespaces
    - roles
    - rolebindings
    - assets
    - handlers
    - checks
    - filters
    - mutators
    - secrets
    verbs:
    - get
    - list
    - create
    - update
    - delete
---
type: ClusterRoleBinding
api_version: core/v2
metadata:
  name: sensu_flow
spec:
  role_ref:
    type: ClusterRole
    name: sensu_flow
  subjects:
  - type: Group
    name: sensu_flow
---
type: User 
api_version: core/v2 
metadata:
  name: sensu_flow
spec:
  disabled: false
  username: sensu_flow
  groups: 
  - sensu_flow
```
