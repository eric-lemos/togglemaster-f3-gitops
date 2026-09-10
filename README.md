# ToggleMaster GitOps

Manifests Kubernetes gerenciados pelo Argo CD. O Terraform instala o Argo CD e o Metrics Server no cluster; esta aplicação é registrada no Argo CD uma única vez:

```bash
kubectl apply -f argocd/application.yaml
```

Depois disso, o Argo CD acompanha a branch `main` deste repositório, aplica a raiz `kustomization.yaml` e mantém o namespace `togglemaster` sincronizado.

## Fluxo de imagens

Cada pipeline de aplicação publica a imagem no ECR e atualiza o campo `image` do manifesto correspondente. O commit no GitOps dispara a sincronização automática do Argo CD.

O repositório de apps precisa destas GitHub Variables:

```text
GITOPS_REPOSITORY=https://github.com/eric-lemos/togglemaster-f3-gitops
GITOPS_BRANCH=main
GITOPS_AUTH_MANIFEST_PATH=apps/auth-service.yaml
GITOPS_EVALUATION_MANIFEST_PATH=apps/evaluation-service.yaml
GITOPS_ANALYTICS_MANIFEST_PATH=apps/analytics-service.yaml
GITOPS_FLAG_MANIFEST_PATH=apps/flag-service.yaml
GITOPS_TARGETING_MANIFEST_PATH=apps/targeting-service.yaml
```

Configure também o secret `GITOPS_TOKEN` com permissão de escrita no repositório GitOps. As pipelines usam esse token somente no push para `main`.

## Requisitos AWS

Os nodes do EKS precisam conseguir fazer pull dos repositórios privados ECR. A role dos nodes deve permitir `ecr:GetAuthorizationToken` e as ações de leitura de camadas e imagens do ECR.

## Workflow manual

O workflow `.github/workflows/gitops.yml` pode ser executado em **Actions > GitOps Deploy > Run workflow** com dois inputs independentes:

- `K8s Apply`: aplica a raiz Kustomize (`kubectl apply -k .`).
- `Run Postgres Schema Job`: remove e recria `postgres-schema-init`, aguarda sua conclusão e exibe os logs.

Configure no repositório GitOps:

### Variables

```text
AWS_REGION=us-east-1
EKS_CLUSTER_NAME=togglemaster-eks-cluster
```

### Secrets

```text
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
AWS_SESSION_TOKEN
```

As credenciais devem estar ativas no momento da execução. Antes de usar `K8s Apply`, substitua os placeholders de `base/secrets.yaml` por valores Base64 válidos ou adicione uma etapa de renderização de secrets. O Kubernetes rejeita valores literais como `${AUTH_DATABASE_URL_BASE64_ENCODED}`.