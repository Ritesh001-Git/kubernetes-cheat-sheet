## 1. What is a ConfigMap?
A ConfigMap is an API object used to store non-confidential data in key-value pairs.

In Kubernetes, you should never hardcode configuration (like API URLs, feature flags, or port numbers) inside your container image. A ConfigMap allows you to decouple your configuration from your application code.

### Why do we use it?

- Portability: You can use the same container image in Development, Staging, and Production. Only the ConfigMap changes for each environment.
- Maintainability: You can update a setting (like a log level) in one place without rebuilding your Docker image or redeploying your entire code base.

## 2. Key Benefits: Why this resource exists
- A. Environment Variable Injection
You can pull specific keys from a ConfigMap and inject them as environment variables into your Pod.

- B. Configuration as a File
You can mount an entire ConfigMap as a file inside a folder in your container (e.g., mounting a prometheus.yaml or nginx.conf).

- C. Decoupling
It separates the Artifact (the code/image) from the Environment (the settings). This is a core principle of the "Twelve-Factor App" methodology used by top tech companies.

## 3. The YAML Structure (The Blueprint)

### Step 1: Create the ConfigMap (The "Source")
```
kind: ConfigMap
apiVersion: v1
metadata:
  name: mysql-config-map
  namespace: mysql
data:
  MYSQL_DATABASE: devops
  MYSQL_USER: admin
  MYSQL_PASSWORD: secret
```
### Step 2: Use it in a Deployment (The "Consumer")
There are two ways to use these in your Deployment YAML:

#### Option A: Injecting Specific Keys
Use this when you only need a few specific variables.

```
env:
- name: MYSQL_DATABASE
    valueFrom:
        configMapKeyRef:
            name: mysql-config-map
            key: MYSQL_DATABASE
```

#### Option B: Injecting ALL Keys (The "Clean" Way)
Top tech companies use envFrom to automatically turn every key in the ConfigMap into an environment variable in the container.

```
envFrom:
  - configMapRef:
    name: mysql-config-map
```

## 4. Essential ConfigMap Commands

| Command | Purpose |
|---|---|
| `kubectl get cm` | List all ConfigMaps in the current namespace. |
| `kubectl get configmap -n mysql` | List all ConfigMaps in the `mysql` namespace. |
| `kubectl describe cm <name>` | View detailed information, including keys and values stored inside the ConfigMap. |
| `kubectl create configmap my-config --from-env-file=.env` | Create a ConfigMap directly from a local `.env` file. |
| `kubectl edit cm <name>` | Edit ConfigMap values on the fly. Pods usually need a restart to pick up updated values. |

## 5. Important: ConfigMap vs. Secret
The most common mistake beginners make is putting sensitive data in a ConfigMap.

- ConfigMap: For public info (Port numbers, URLs, Log levels). Data is stored in Plain Text.
- Secret: For private info (Passwords, API Keys, SSH Keys). Data is stored in Base64 (and can be encrypted at rest).

### What happens when you update a ConfigMap?
If you update a ConfigMap that is used as an Environment Variable, the Pod will NOT see the update until it is restarted. If you mount the ConfigMap as a File, the file inside the container will update automatically after a minute or two (though the app might still need a reload signal).
