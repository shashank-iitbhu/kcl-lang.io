# CRD to KCL

命令

```shell
kcl import -m crd -o ${the_kcl_files_output_dir} -s ${your_CRD.yaml} 
```

# 示例

- 输入文件：test_crontab_CRD.yaml:

```yaml
# Deprecated in v1.16 in favor of apiextensions.k8s.io/v1
apiVersion: apiextensions.k8s.io/v1beta1
kind: CustomResourceDefinition
metadata:
  # name must match the spec fields below, and be in the form: <plural>.<group>
  name: crontabs.stable.example.com
spec:
  # group name to use for REST API: /apis/<group>/<version>
  group: stable.example.com
  # list of versions supported by this CustomResourceDefinition
  versions:
    - name: v1
      # Each version can be enabled/disabled by Served flag.
      served: true
      # One and only one version must be marked as the storage version.
      storage: true
  # either Namespaced or Cluster
  scope: Namespaced
  names:
    # plural name to be used in the URL: /apis/<group>/<version>/<plural>
    plural: crontabs
    # singular name to be used as an alias on the CLI and for display
    singular: crontab
    # kind is normally the CamelCased singular type. Your resource manifests use this.
    kind: CronTab
    # shortNames allow shorter string to match your resource on the CLI
    shortNames:
      - ct
  preserveUnknownFields: false
  validation:
    openAPIV3Schema:
      type: object
      properties:
        spec:
          type: object
          properties:
            cronSpec:
              type: string
            image:
              type: string
            replicas:
              type: integer
```

- 命令

```shell
kcl-openapi generate model -f test_crontab_CRD.yaml -t ~/ --skip-validation --crd
```

- 输出文件： ~/models/stable_example_com_v1_cron_tab.k

```python
"""
This file was generated by the KCL auto-gen tool. DO NOT EDIT.
Editing this file might prove futile when you re-run the KCL auto-gen generate command.
"""
import kusion_kubernetes.apimachinery.apis


schema CronTab:
    """stable example com v1 cron tab

    Attributes
    ----------
    apiVersion : str, default is "stable.example.com/v1", required
         APIVersion defines the versioned schema of this representation of an object. Servers should convert recognized schemas to the latest internal value, and may reject unrecognized values. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources
    kind : str, default is "CronTab", required
         Kind is a string value representing the REST resource this object represents. Servers may infer this from the endpoint the client submits requests to. Cannot be updated. In CamelCase. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds
    metadata : apis.ObjectMeta, default is Undefined, optional
        metadata
    spec : StableExampleComV1CronTabSpec, default is Undefined, optional
        spec
    """


    apiVersion: "stable.example.com/v1" = "stable.example.com/v1"

    kind: "CronTab" = "CronTab"

    metadata?: apis.ObjectMeta

    spec?: StableExampleComV1CronTabSpec


schema StableExampleComV1CronTabSpec:
    """stable example com v1 cron tab spec

    Attributes
    ----------
    cronSpec : str, default is Undefined, optional
        cron spec
    image : str, default is Undefined, optional
        image
    replicas : int, default is Undefined, optional
        replicas
    """


    cronSpec?: str

    image?: str

    replicas?: int
```