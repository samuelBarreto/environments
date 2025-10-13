# ⚡ Kustomize - Quick Start

## 🎯 Testar Localmente

### **Ver o que será aplicado (sem aplicar):**

```bash
# Dev
kubectl kustomize environments/dev/claims/

# Prod
kubectl kustomize environments/prod/claims/

# HLM
kubectl kustomize environments/hlm/claims/
```

### **Aplicar claims:**

```bash
# Dev
kubectl apply -k environments/dev/claims/

# Prod
kubectl apply -k environments/prod/claims/

# HLM
kubectl apply -k environments/hlm/claims/
```

**Nota:** Use `-k` (kustomize) em vez de `-f`!

---

## ✏️ Controlar o que Sobe

### **Exemplo: Dev**

Edite `environments/dev/claims/kustomization.yaml`:

```yaml
resources:
- example-bucket.yaml      # ✅ Vai subir
- example-vpc.yaml         # ✅ Vai subir
# - example-database.yaml  # ❌ NÃO vai subir (comentado)
```

**Para habilitar database:**
1. Remove o `#` da frente
2. Commit & Push (GitOps)
3. Ou aplica local: `kubectl apply -k environments/dev/claims/`

---

## 🔄 Workflow GitOps

### **1. Editar kustomization.yaml no Git:**

```bash
cd environments
vim dev/claims/kustomization.yaml

# Habilitar database
resources:
- example-bucket.yaml
- example-vpc.yaml
- example-database.yaml  # ← Habilitar

git add dev/claims/kustomization.yaml
git commit -m "feat: enable database in dev"
git push origin main
```

### **2. ArgoCD Sincroniza Automaticamente:**

```bash
# ArgoCD detecta mudança
# Roda: kubectl apply -k dev/claims/
# Database é criado!
```

---

## 📊 Status

### **Ver o que está ativo em cada ambiente:**

```bash
# Dev
cat environments/dev/claims/kustomization.yaml

# Prod  
cat environments/prod/claims/kustomization.yaml

# HLM
cat environments/hlm/claims/kustomization.yaml
```

---

## 🎯 Configuração Atual

### **DEV:**
- ✅ `example-bucket.yaml` - Habilitado
- ✅ `example-vpc.yaml` - Habilitado
- ❌ `example-database.yaml` - **Desabilitado** (comentado)

### **PROD:**
- ✅ `example-bucket.yaml` - Habilitado
- ✅ `example-database.yaml` - Habilitado

### **HLM:**
- ✅ `example-bucket.yaml` - Habilitado
- ✅ `example-database.yaml` - Habilitado

---

## 🚀 Comandos Rápidos

```bash
# Aplicar tudo do dev
kubectl apply -k environments/dev/claims/

# Preview
kubectl kustomize environments/dev/claims/ | less

# Deletar tudo
kubectl delete -k environments/dev/claims/

# Diff
kubectl diff -k environments/dev/claims/
```

---

**Simples assim!** 🎉

