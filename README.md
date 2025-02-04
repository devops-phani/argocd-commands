# argocd-commands

Get all application names of test project

```
kubectl get applications -n argocd -o json | jq -r '.items[] | select(.spec.project == "test") | .metadata.name'
```

Get all application names and git repo names of test project

```
kubectl get applications -n argocd -o json | jq -r '.items[] | select(.spec.project == "test") | "\(.metadata.name)\t\(.spec.source.repoURL)"'
```

Redirect the above command output to a file

```
kubectl get applications -n argocd -o json | jq -r '.items[] | select(.spec.project == "test") | "\(.metadata.name)\t\(.spec.source.repoURL)"' > test.txt
```

Extract only application names from `test.txt`

```
cat test.txt | awk '{print $1}' > test-apps.txt
```

Extract only Git repo urls from `test.txt`

```
cat test.txt | awk '{print $2}' > test-repo-urls.txt
```

Extract only Git repo names from `test-repo-urls.txt`

```
sed -E 's|^.*/([^/]+)\.git$|\1|' test-repo-urls.txt > test-repo-names.txt
```

Disable the auto sync for all test project repos

Create  `test.sh`

```
vim test.sh
```


```
#!/bin/bash

# Filename to read from
FILE="test-apps.txt"

# Check if the file exists
if [[ ! -f "$FILE" ]]; then
  echo "File not found!"
  exit 1
fi

# Read the file line by line
while IFS= read -r line; do
  echo "prod app $line"
  kubectl patch application $line -n argocd --type=json -p='[{"op": "remove", "path": "/spec/syncPolicy/automated"}]'
done < "$FILE"
```


Get the autosync enabled applications

```
kubectl get applications -n argocd -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.spec.project}{"\t"}{.spec.syncPolicy.automated}{"\n"}{end}' | grep {} | awk '{print $1}'
```
```
sh test.sh
```

Argocd policies from helm chart or `argocd-rbac-cm` configmap

Refeerence link

- https://github.com/argoproj/argo-cd/blob/master/assets/builtin-policy.csv


- Provide read access to argocd-reader group
- Provide admin access to argocd-admin group
- Provide only sync access of test project to argocd-test-sync group

```
rbac:
  policy.csv: |
     g, "argocd-admin", role:admin
     g, "argocd-reader", role:readonly
     p, role:testsync, applications, sync, test/*, allow
     g, "argocd-test-sync", role:demosync
```      



