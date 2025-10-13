# 🎯 Kustomization - Gerenciamento de Claims

## 📋 Visão Geral

Cada ambiente (dev, hlm, prod) tem um arquivo `kustomization.yaml` que **controla quais claims** serão aplicados.

## 📂 Estrutura

```
environments/
├── dev/
│   └── claims/
│       ├── kustomization.yaml      ← Lista de claims para DEV
│       ├── example-bucket.yaml
│       ├── example-vpc.yaml
│       └── example-database.yaml
│
├── hlm/
│   └── claims/
│       ├── kustomization.yaml      ← Lista de claims para HLM
│       ├── example-bucket.yaml
│       └── example-database.yaml
│
└── prod/
    └── claims/
        ├── kustomization.yaml      ← Lista de claims para PROD
        ├── example-bucket.yaml
        └── example-database.yaml
```

---

## 🎯 Como Funciona

### **kustomization.yaml em DEV:**

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

namespace: dev

resources:
- example-bucket.yaml      # ← Vai subir
- example-vpc.yaml         # ← Vai subir
# - example-database.yaml  # ← Comentado, NÃO vai subir
```

**Controle fino:**
- ✅ **Listado** em `resources:` → Será aplicado
- ❌ **Comentado** (# na frente) → NÃO será aplicado
- ❌ **Não listado** → Ignorado

---

## 🚀 Como Usar

### **1. Aplicar Manualmente (kubectl):**

```bash
# Aplicar todos os claims do dev
kubectl apply -k environments/dev/claims/

# Aplicar todos os claims do prod
kubectl apply -k environments/prod/claims/

# Aplicar todos os claims do hlm
kubectl apply -k environments/hlm/claims/
```

**Nota:** Use `-k` (kustomize) em vez de `-f`!

### **2. Preview (dry-run):**

```bash
# Ver o que seria aplicado
kubectl kustomize environments/dev/claims/

# Ou com apply dry-run
kubectl apply -k environments/dev/claims/ --dry-run=client
```

### **3. Via ArgoCD:**

O ArgoCD detecta automaticamente `kustomization.yaml` e usa Kustomize!

---

## ✏️ Habilitar/Desabilitar Claims

### **Desabilitar um Claim:**

```yaml
# kustomization.yaml
resources:
- example-bucket.yaml
- example-vpc.yaml
# - example-database.yaml  ← Comentar para desabilitar
```

### **Habilitar um Claim:**

```yaml
# kustomization.yaml
resources:
- example-bucket.yaml
- example-vpc.yaml
- example-database.yaml  ← Descomentar para habilitar
```

### **Adicionar Novo Claim:**

```bash
# 1. Criar arquivo
cat > environments/dev/claims/new-redis.yaml << EOF
apiVersion: platform.example.com/v1alpha1
kind: Database
metadata:
  name: dev-redis
  namespace: dev
spec:
  engine: redis
  ...
EOF

# 2. Adicionar ao kustomization.yaml
echo "- new-redis.yaml" >> environments/dev/claims/kustomization.yaml

# 3. Aplicar
kubectl apply -k environments/dev/claims/
```

---

## 🔄 GitOps Workflow

### **Com Kustomize + ArgoCD:**

```
1. Editar kustomization.yaml (habilitar/desabilitar claims)
   ↓
2. Commit & Push
   ↓
3. ArgoCD detecta mudança
   ↓
4. ArgoCD roda: kubectl apply -k environments/dev/claims/
   ↓
5. Claims aplicados/removidos automaticamente
```

---

## 📊 Exemplo Prático

### **Cenário: Adicionar Database em Dev**

**Antes:**
```yaml
# dev/claims/kustomization.yaml
resources:
- example-bucket.yaml
- example-vpc.yaml
# - example-database.yaml  ← Comentado
```

**Depois:**
```yaml
# dev/claims/kustomization.yaml
resources:
- example-bucket.yaml
- example-vpc.yaml
- example-database.yaml  ← Habilitado!
```

**Resultado:**
- ArgoCD detecta mudança
- Aplica o database claim
- Database é provisionado na AWS

---

## 🔧 Funcionalidades Avançadas

### **1. Patches (Sobrescrever valores):**

```yaml
# kustomization.yaml
resources:
- example-bucket.yaml

patches:
- target:
    kind: Bucket
    name: dev-app-storage
  patch: |-
    - op: replace
      path: /spec/storageClass
      value: glacier
```

### **2. ConfigMap/Secret Generators:**

```yaml
# kustomization.yaml
configMapGenerator:
- name: app-config
  literals:
  - ENVIRONMENT=dev
  - LOG_LEVEL=debug
```

### **3. Name Prefix/Suffix:**

```yaml
# kustomization.yaml
namePrefix: team-a-
nameSuffix: -v2

# Resultado:
# dev-app-storage → team-a-dev-app-storage-v2
```

---

## ✅ Benefícios

| Benefício | Descrição |
|-----------|-----------|
| ✅ **Controle Granular** | Habilitar/desabilitar claims individualmente |
| ✅ **GitOps Friendly** | Versionamento claro do que está ativo |
| ✅ **DRY** | Evita duplicação com patches |
| ✅ **Safe** | Preview antes de aplicar |
| ✅ **ArgoCD Native** | Suporte nativo |

---

## 🎯 Comandos Úteis

```bash
# Ver manifests gerados (sem aplicar)
kubectl kustomize environments/dev/claims/

# Aplicar todos os claims
kubectl apply -k environments/dev/claims/

# Deletar todos os claims
kubectl delete -k environments/dev/claims/

# Diff (ver mudanças)
kubectl diff -k environments/dev/claims/

# Validar syntax
kubectl kustomize environments/dev/claims/ | kubectl apply --dry-run=client -f -
```

---

## 📚 Documentação

- [Kustomize Docs](https://kustomize.io/)
- [Kubectl Kustomize](https://kubernetes.io/docs/tasks/manage-kubernetes-objects/kustomization/)
- [ArgoCD + Kustomize](https://argo-cd.readthedocs.io/en/stable/user-guide/kustomize/)

---

**Agora você tem controle total sobre quais recursos são aplicados em cada ambiente!** 🎉

