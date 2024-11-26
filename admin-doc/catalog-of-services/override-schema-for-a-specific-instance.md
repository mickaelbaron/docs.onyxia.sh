# 🎭 Override schema for a specific instance

When charts allow overwriteSchemaWith you can modify this schema at the instance level or for a specific role.

### For the whole instance

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
                  "enum": ["ssd", "hdd"],
                  "default": "ssd"
                },
                "gpu": {
                  "description": "The type of GPU",
                  "type": "string",
                  "enum": ["A2", "H100"],
                  "default": "A2"
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
