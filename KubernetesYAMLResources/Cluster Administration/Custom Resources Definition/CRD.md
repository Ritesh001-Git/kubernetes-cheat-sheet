# Custom Resources Definition (CRD)
In Kubernetes, out-of-the-box resources like Pods, Deployments, Services, and ConfigMaps are incredibly powerful. But what if your company builds a highly specific platform—like a database automation engine, a custom internal developer portal, or an AI model runner—and you want to manage it using native Kubernetes commands?

That is where a Custom Resource Definition (CRD) comes in. It allows you to extend the Kubernetes API by creating your own entirely unique resource types.

## 1. What is a CRD?
A CRD is a blueprint that tells the Kubernetes API server: "Hey, look out for this brand-new resource type. It has these specific fields, validation rules, and structural schemas."

Once you apply a CRD, it is no longer just "plain text." You can run commands like kubectl get <your-custom-resource> just like you would for a standard Pod.

### The CRD Framework: Custom Resource vs. Controller
Just like an Ingress, a custom setup requires two distinct parts:

- Custom Resource Definition (CRD): The structural schema applied by the admin (The "Blueprint").
- Custom Resource (CR): The actual instance of the object written by developers (The "Building").
- Custom Controller/Operator: The custom code (usually written in Go or Python) running in a background pod that watches your Custom Resources and actually executes the logic to make things happen.

## 2. Real-World Use Case: The Operator Pattern
Most major open-source tools use CRDs to build "Operators" (software that mimics human operational knowledge).

For example, if you use Prometheus, you don't write hundreds of individual config lines. You install the Prometheus CRDs, and then you just deploy a small object called a ServiceMonitor. The Prometheus Operator reads that object and automatically configures the metrics collection for you.

## 3. The YML Structure (Blueprint)
Here is how an administrator defines a brand-new resource called a Database:
```
apiVersion: rbac.authorization.k8s.io/v1 # Note: crds use the apiextensions group
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  # Name must follow the pattern: <plural>.<group>
  name: databases.company.com
spec:
  group: company.com
  versions:
    - name: v1alpha1
      served: true
      storage: true
      # OpenAPI schema enforces validation rules on inputs
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              required: ["engine", "sizeGb"]
              properties:
                engine:
                  type: string
                  enum: ["postgres", "mysql", "redis"]
                sizeGb:
                  type: integer
                  minimum: 10
                  maximum: 1000
  scope: Namespaced
  names:
    plural: databases
    singular: database
    kind: Database
    shortNames:
    - db
```

## 4. Using the Custom Resource (The "Instance")
Once the CRD above is applied to the cluster, any developer in the company can now write a native Kubernetes file using that schema.
```
apiVersion: company.com/v1alpha1
kind: Database
metadata:
  name: user-profile-db
  namespace: production
spec:
  engine: postgres
  sizeGb: 250
```

## 5. Essential CRD Commands

| Command | Purpose |
|----------|---------|
| `kubectl get crd` | List all Custom Resource Definitions (CRDs) installed in the cluster. |
| `kubectl get db` | Native Kubernetes integration: Query your custom resources using their configured short names. |
| `kubectl explain db.spec` | Display the schema, fields, and definitions of the custom resource directly from the terminal, similar to native Kubernetes resources. |

## 6. When to use CRDs (vs. ConfigMaps)
A common point of confusion is choosing between a ConfigMap and a Custom Resource.

- Use a ConfigMap if: You are simply passing standard environment variables, strings, or configuration files into an existing application container.
- Use a CRD if: You want to leverage the core strength of Kubernetes—its declarative control loop. You want to enforce structured validation, tracking statuses via kubectl, and pairing it with custom automated controllers to build automated self-healing platforms.
