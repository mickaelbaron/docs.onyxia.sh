---
description: Unserstand how Onyxia catalogs work and potentially create your own!
---

# 🔬 Catalog of services

Every Onyxia instance may or may not have it's own catalog. There are four default catalogs :

{% embed url="https://github.com/inseefrlab/helm-charts-interactive-services" %}

This collection of charts helps users to launch many IDE with various binary stacks (python , R) with or without GPU support. Docker images are built [here](https://github.com/inseefrlab/images-datascience) and help us to give a homogeneous stack.

{% embed url="https://github.com/inseefrlab/helm-charts-databases" %}

This collection of charts helps users to launch many databases system. Most of them are based on [bitnami/charts](https://github.com/bitnami/charts).

{% embed url="https://github.com/InseeFrLab/helm-charts-automation/" %}

This collection of charts helps users to start automation tools for their datascience activity.

{% embed url="https://github.com/InseeFrLab/helm-charts-datavisualization" %}

This collection of charts helps users to launch tools to visualize and share data insights.

You can always find the source of the catalog by clicking on the "contribute to the... " link.

#### How to overwrite a schema or create a new one ?

You can directly create file in the values of onyxia helm charts

{% code title="onyxia-values.yaml" %}
```yaml
onyxia:
  web:
    # ...
  api:
    # ...
    schemas:
      enabled: true
      files:
        - relativePath: ide/resources.json
          content: |
            {
                "$schema": "http://json-schema.org/draft-07/schema#",
                "title": "Resources",
                "description": "Your service will have at least the requested resources and never more than its limits. No limit for a resource and you can consume everything left on the host machine.",
                "type": "object",
                "properties": {
                    "requests": {
                        "description": "Guaranteed resources",
                        "type": "object",
                        "properties": {
                            "cpu": {
                                "description": "The amount of cpu guaranteed",
                                "title": "CPU",
                                "type": "string",
                                "default": "100m",
                                "render": "slider",
                                "sliderMin": 50,
                                "sliderMax": 10000,
                                "sliderStep": 50,
                                "sliderUnit": "m",
                                "sliderExtremity": "down",
                                "sliderExtremitySemantic": "guaranteed",
                                "sliderRangeId": "cpu"
                            },
                            "memory": {
                                "description": "The amount of memory guaranteed",
                                "title": "memory",
                                "type": "string",
                                "default": "2Gi",
                                "render": "slider",
                                "sliderMin": 1,
                                "sliderMax": 200,
                                "sliderStep": 1,
                                "sliderUnit": "Gi",
                                "sliderExtremity": "down",
                                "sliderExtremitySemantic": "guaranteed",
                                "sliderRangeId": "memory"
                            }
                        }
                    },
                    "limits": {
                        "description": "max resources",
                        "type": "object",
                        "properties": {
                            "cpu": {
                                "description": "The maximum amount of cpu",
                                "title": "CPU",
                                "type": "string",
                                "default": "5000m",
                                "render": "slider",
                                "sliderMin": 50,
                                "sliderMax": 10000,
                                "sliderStep": 50,
                                "sliderUnit": "m",
                                "sliderExtremity": "up",
                                "sliderExtremitySemantic": "Maximum",
                                "sliderRangeId": "cpu"
                            },
                            "memory": {
                                "description": "The maximum amount of memory",
                                "title": "Memory",
                                "type": "string",
                                "default": "50Gi",
                                "render": "slider",
                                "sliderMin": 1,
                                "sliderMax": 200,
                                "sliderStep": 1,
                                "sliderUnit": "Gi",
                                "sliderExtremity": "up",
                                "sliderExtremitySemantic": "Maximum",
                                "sliderRangeId": "memory"
                            }
                        }
                    }
                }
            }
        - relativePath: nodeSelector.json
          content: |
            {
              "$schema": "http://json-schema.org/draft-07/schema#",
              "title": "Node Selector",
              "type": "object",
              "properties": {
                "disktype": {
                  "description": "The type of disk",
                  "type": "string",
                  "enum": ["ssd", "hdd"]
                },
                "gpu": {
                  "description": "The type of GPU",
                  "type": "string",
                  "enum": ["A2", "H100"]
                }
              },
              "additionalProperties": false
            }
        - relativePath: ide/role.json
          content: |
            {
              "$schema": "http://json-schema.org/draft-07/schema#",
              "title": "Role",
              "type": "object",
              "properties": {
                  "enabled": {
                      "type": "boolean",
                      "description": "allow your service to access your namespace ressources",
                      "default": true
                  },
                  "role": {
                      "type": "string",
                      "description": "bind your service account to this kubernetes default role",
                      "default": "view",
                      "hidden": {
                          "value": false,
                          "path": "kubernetes/enabled"
                      },
                      "enum": [
                          "view"
                      ]
                  }
              }
            }
```
{% endcode %}
