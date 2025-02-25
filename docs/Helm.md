# Helm

Interacts with Helm. Note that you should set the environment variable `KUBECONFIG` in your pipeline with `environment { KUBECONFIG = '/path/to/.kube/config' }` as the `jenkins` user probably does not have one in its home directory, and Helm requires a valid kube config for all commands. Alternatively, you can use the `kubeconfigFile` or `kubeconfigContent` bindings for the Credentials Binding plugin, and then wrap code within a `withCredentials` block as per normal. Also alternatively, you can combine the two like `environment { KUBECONFIG = credentials('my-kubeconfig') }`

Also please note direct Kubernetes support will never exist in this library. The reason for this is that the Kubernetes Java client requires exactly specifying the API version and object type altered during modifications. This limitation is not encountered in the Golang client and its associated tools (e.g. Kubectl), and the reason is [well explained here](https://github.com/kubernetes-client/java/issues/611#issuecomment-509106822). This makes the implementation significantly more cumbersome. The workaround for this has already been excellently implemented by Fabric8 in their library, so no compelling reason exists to attempt to implement something similar in this library and global variable.

### Dependencies

- Helm CLI binary executable >= 3.0
- Kubernetes cluster

### helm.install()
Performs an installation with Helm onto the Kubernetes cluster.

```groovy
helm.install(
  bin:       '/usr/bin/helm', // optional executable path for helm
  chart:     'chart', // chart repository, local archive, directory, or url to install
  context:   'default', // optional kube-context from kube config
  dryRun:    false, // optional simulate an install
  name:      'happy-panda', // required name for the installed release object
  namespace: 'default', // optional namespace for the installed release object
  values:    ['config.yaml'], // optional value overrides yaml file or url
  verify:    true, // optional verify the provenance of the chart
  wait:      false, // optional wait until everything is in a ready state
  set:       ['foo':'bar', 'bar':'baz'] // optional value override
)
```

### helm.lint()
Runs a series of tests to verify that the chart is well-formed.

```groovy
helm.lint(
  bin:       '/usr/bin/helm', // optional executable path for helm
  chart:     'chart', // chart repository, local archive, directory, or url to install
  context:   'default', // optional kube-context from kube config
  namespace: 'default', // optional namespace for the installed release object
  values:    ['config.yaml'], // optional value overrides yaml file or url
  set:       ['foo':'bar', 'bar':'baz'], // optional value override
  strict:    false // optional fail on warnings
)
```

### helm.packages()
Package a chart directory into a chart archive.

```groovy
helm.packages(
  bin:        '/usr/bin/helm', // optional executable path for helm
  chart:      'path/to/chart', // absolute or relative path to chart
  dest:       '.', // optional location to write the chart
  key:        'foo', // optional sign the package with this key name (mutually exclusive with keyring)
  keyring:    '/home/dir/.gnupg/pubring.gpg', // optional sign the package with the public keyring at this location (mutually exclusive with key)
  updateDeps: false, // optional update dependencies from requirements prior to packaging
  version:    '1.0.0' // optional version set for the chart
)
```

### helm.plugin()
Manage client-side Helm plugins.

```groovy
helm.packages(
  bin:     '/usr/bin/helm', // optional executable path for helm
  command: 'install', // plugin command; one of 'install', 'list', 'uninstall', or 'update'
  plugin:  'https://github.com/adamreese/helm-env' // targeted plugin (unless 'list' command)
)
```

### helm.repo()
Add a Helm chart repository. The repository will update if it has already been added.

```groovy
helm.repo(
  bin:      '/usr/local/bin/helm', // optional executable path
  ca:       '/path/to/crt.ca', // optional path to CA bundle to verify certificates of HTTPS servers
  cert:     '/path/to/ca.crt', // optional path to HTTPS client SSL certificate file
  insecure: false, // optional skip tls certificate checks
  key:      '/path/to/rsa.key', // optional path to HTTPS client SSL key file
  password: 'mypassword', // optional chart repository password
  repo:     'stable', // name of the chart repository
  url:      'https://kubernetes-charts.storage.googleapis.com', // url of the chart repository
  user:     'myuser' // optional chart repository username
)
```

### helm.rollback()
Roll back the release object to a previous release with Helm.

```groovy
helm.rollback(
  bin:       '/usr/local/bin/helm', // optional executable path for helm
  context:   'default', // optional kube-context from kube config
  name:      'happy-panda', // release object name to rollback
  namespace: 'default', // optional namespace for the rolled back release object
  version:   '1' // version of release-object to rollback to
)
```

### helm.show()
Show information about a chart.

```groovy
helm.show(
  bin:   '/usr/local/bin/helm', // optional executable path for helm
  chart: 'chart', // chart repository, local archive, directory, or url to display
  info:  'all', // info to display; one of 'all', 'chart', 'readme', or 'values'
)
```

### helm.status()
Shows the status of a named release.

```groovy
helm.status(
  bin:       '/usr/bin/helm', // optional executable path for helm
  name:      'happy-panda', // name for the release object to be queried
  context:   'default' // optional kube-context from kube config
  namespace: 'default' // optional namespace for the queried release object
)
```

### helm.test()
Executes the tests for a release.

```groovy
helm.test(
  bin:       '/usr/bin/helm', // optional executable path for helm
  cleanup:   false, // optional delete test pods upon completion
  context:   'default', // optional kube-context from kube config
  kubectl:   '/usr/bin/kubectl', // optional executable path for kubectl
  name:      'happy-panda', // name of a deployed release
  namespace: 'default' // optional namespace for the queried release object
  parallel:  false // optional run test pods in parallel
)
```

### helm.uninstall()
Uninstall the release object from Kubernetes with Helm.

```groovy
helm.uninstall(
  bin:       '/usr/bin/helm', // optional executable path for helm
  name:      'happy-panda', // name for the release object to be deleted
  context:   'default' // optional kube-context from kube config
  namespace: 'default' // optional namespace for the uninstalled release object
)
```

### helm.upgrade()
Updates and/or changes the configuration of a release with Helm.

```groovy
helm.upgrade(
  bin:       '/usr/bin/helm', // optional executable path for helm
  chart:     'chart', // chart repository, local archive, directory, or url to upgrade
  context:   'default', // optional kube-context from kube config
  dryRun:    false, // optional simulate an upgrade
  install:   false, // optional install if release not already present
  name:      'happy-panda', // name of the upgraded release object
  namespace: 'default', // optional namespace for the upgraded release object
  values:    ['config.yaml'], // optional value overrides yaml file or url
  verify:    true, // optional verify the provenance of the chart
  wait:      false, // optional wait until everything is in a ready state
  set:       ['foo':'bar', 'bar':'baz'] // optional value override
)
```

### helm.verify(String chartPath, String helmPath = 'helm')
Verify that the given chart has a valid provenance file. This can be used to verify a local chart. This method will return a `Boolean` type indicating whether the verification was successful (`true`) or not (`false`).

```groovy
helm.verify('/path/to/chart.tar.gz', '/usr/bin/helm')
```
