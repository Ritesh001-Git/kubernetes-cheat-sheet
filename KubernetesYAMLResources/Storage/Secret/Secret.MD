# Secret

## 1. What is a Secret?
A Secret is an object that holds a small amount of sensitive data. It behaves like a ConfigMap but is specifically designed to handle data that shouldn't be visible in plain text in your YAML files.

In a professional environment, Secrets are used for passwords, API tokens, and—as you mentioned—binary files like SSL certificates, SSH keys, or even small image blobs.

### Why do we use it for Binary Files?
Since YAML is a text-based format, you cannot paste "raw" binary data (like a .p12 certificate or a .jpg icon) directly into it. Base64 encoding allows you to turn that binary data into a string of text that the YAML file can hold.

## 2. The Manual "Encryption" (Encoding) Commands
To put a password or a file into a Secret YAML manually, you must encode it first.

### Encoding a Password:

```
# -n ensures no 'newline' character is added to the password
echo -n "my-super-secret-password" | base64
# Output: bXktc3VwZXItc2VjcmV0LXBhc3N3b3Jk
```

### Encoding a Binary File:

```
# Convert a certificate or image into a base64 string
cat cert.crt | base64
```

### Decoding:
If you see a Secret YAML and want to know the password:

```
echo "bXktc3VwZXItc2VjcmV0LXBhc3N3b3Jk" | base64 --decode
# Output: my-super-secret-password
```

## 3. The YML file (The Blueprint)

```
apiVersion: v1
kind: Secret
metadata:
  name: mysql-secret
  namespace: mysql
data:
  MYSQL_ROOT_PASSWORD: Y2hpa3UK # base64 encoded for 'chiku'


# Command = echo "chiku" | base64
```

### How to use it in your YAML
```
- name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: MYSQL_ROOT_PASSWORD
```

## 4. If Base64 is not Secure, how is it protected?
You are right: Base64 is not encryption. Anyone with access to the YAML can decode it. In tech companies, we add "Real" security in three ways:

- RBAC: We restrict who can run kubectl get secrets. Most developers can't see them.
- Encryption at Rest: We configure the Kubernetes API to encrypt the secret data before it's saved to the etcd database.
- External Vaults: Many companies use HashiCorp Vault or AWS Secrets Manager. They store the data there and "inject" it into the Pod so it's never even saved as a K8s Secret object.

## 5. Essential Secret Commands
| Command | Purpose |
|---|---|
| `kubectl get secrets` | List all secrets. |
| `kubectl describe secret <name>` | Shows the size and metadata of the data but hides the actual values. |
| `kubectl get secret <name> -o jsonpath='{.data.DB_PASSWORD}' \| base64 --decode` | Decode and display a specific secret value from Base64. |
| `kubectl create secret generic my-auth --from-file=ssh-key=./id_rsa` | Best way for binary secrets: creates a secret directly from a local file. |
