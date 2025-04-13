## How application set is work

The ApplicationSet controller is a Kubernetes controller that adds support for an ApplicationSet CustomResourceDefinition (CRD). This controller/CRD enables both automation and greater flexibility managing Argo CD Applications across a large number of clusters and within monorepos, plus it makes self-service usage possible on multitenant Kubernetes clusters.

1.The ability to use a single Kubernetes manifest to target multiple Kubernetes clusters with Argo CD
2.The ability to use a single Kubernetes manifest to deploy multiple applications from one or multiple Git repositories with Argo CD
3.Improved support for monorepos: in the context of Argo CD, a monorepo is multiple Argo CD Application resources defined within a single Git repository
4.Within multitenant clusters, improves the ability of individual cluster tenants to deploy applications using Argo CD (without needing to involve privileged cluster administrators in enabling the destination clusters/namespaces)

## Generator 

`list generator`:
The List generator allows you to target Argo CD Applications to clusters based on a fixed list of any chosen key/value element pairs.
```
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: guestbook
  namespace: argocd
spec:
  goTemplate: true
  goTemplateOptions: ["missingkey=error"]
  generators:
  - list:
      elements:
      - cluster: engineering-dev
        url: https://kubernetes.default.svc
      # - cluster: engineering-prod
      #   url: https://kubernetes.default.svc
  template:
    metadata:
      name: '{{.cluster}}-guestbook'
    spec:
      project: "my-project"
      source:
        repoURL: https://github.com/argoproj/argo-cd.git
        targetRevision: HEAD
        path: applicationset/examples/list-generator/guestbook/{{.cluster}}
      destination:
        server: '{{.url}}'
        namespace: guestbook
```

`Cluster generator` 
```
`apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: guestbook
  namespace: argocd
spec:
  goTemplate: true
  goTemplateOptions: ["missingkey=error"]
  generators:
  - clusters:
      selector:
        matchLabels:
          staging: "true"
        # The cluster generator also supports matchExpressions.
        #matchExpressions:
        #  - key: staging
        #    operator: In
        #    values:
        #      - "true"
  template:
  # (...)
```


`Git generator`

`Matrix generator `





