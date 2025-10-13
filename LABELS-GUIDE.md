# Labels vs Annotations - Guia de Boas Práticas

## 🏷️ Labels

### O que são?
Labels são pares chave-valor usados para **identificar e selecionar** recursos no Kubernetes.

### Quando usar?
- ✅ Seleção de recursos (kubectl get, selectors)
- ✅ Organização e agrupamento
- ✅ Filtros e queries
- ✅ Routing e scheduling

### Restrições

#### Sintaxe da chave:
- Prefixo (opcional): `example.com/`
- Nome: max 63 caracteres
- Caracteres permitidos: `a-z`, `A-Z`, `0-9`, `-`, `_`, `.`
- Deve começar e terminar com alfanumérico

#### Sintaxe do valor:
- Max 63 caracteres
- Caracteres permitidos: `a-z`, `A-Z`, `0-9`, `-`, `_`, `.`
- Pode ser vazio
- ❌ **NÃO pode conter**: `@`, espaços, caracteres especiais

### Exemplos VÁLIDOS ✅

```yaml
labels:
  app: my-app
  environment: prod
  team: backend-team
  cost-center: CC-PRODUCTION
  version: v1.2.3
  tier: frontend
  release: stable
  track: canary
```

### Exemplos INVÁLIDOS ❌

```yaml
labels:
  owner: team@example.com        # ❌ Contém @
  description: "My App Server"   # ❌ Contém espaços
  url: https://example.com       # ❌ Contém ://
  email: dev@company.com         # ❌ Contém @
```

---

## 📝 Annotations

### O que são?
Annotations são metadados descritivos **não usados para seleção**.

### Quando usar?
- ✅ Emails e contatos
- ✅ URLs e links
- ✅ Descrições longas
- ✅ Timestamps
- ✅ Configurações de ferramentas (ArgoCD, Prometheus, etc)
- ✅ Documentação e notas

### Restrições
- Chave: mesmas regras das labels
- Valor: **QUALQUER STRING** (até 256KB)

### Exemplos ✅

```yaml
annotations:
  owner: team@example.com
  contact: "John Doe <john@example.com>"
  description: "Production database for customer API"
  documentation: "https://wiki.company.com/my-app"
  created-by: "platform-team"
  last-modified: "2024-10-13T10:30:00Z"
  jira-ticket: "PLAT-1234"
  argocd.argoproj.io/sync-wave: "1"
  prometheus.io/scrape: "true"
```

---

## 🎯 Exemplos Práticos

### Database Claim (Dev)

```yaml
apiVersion: platform.example.com/v1alpha1
kind: Database
metadata:
  name: dev-app-database
  namespace: dev
  
  # Labels: para seleção e organização
  labels:
    app: my-app
    environment: dev
    team: backend-team
    cost-center: CC-DEVELOPMENT
    tier: data
    managed-by: crossplane
  
  # Annotations: metadados descritivos
  annotations:
    owner: team@example.com
    contact: "Backend Team <backend@example.com>"
    description: "PostgreSQL database for development environment"
    created-date: "2024-10-13"
    documentation: "https://wiki.company.com/databases/dev-app"
    jira-ticket: "DEV-123"
spec:
  engine: postgresql
  engineVersion: "15"
  size: small
```

### S3 Bucket (Prod)

```yaml
apiVersion: platform.example.com/v1alpha1
kind: Bucket
metadata:
  name: prod-app-storage
  namespace: prod
  
  labels:
    app: my-app
    environment: prod
    team: frontend-team
    cost-center: CC-PRODUCTION
    data-classification: sensitive
  
  annotations:
    owner: frontend@example.com
    compliance-officer: "security@example.com"
    retention-policy: "365 days"
    backup-schedule: "daily at 03:00 UTC"
    last-audit: "2024-10-01"
spec:
  storageClass: intelligent-tiering
```

---

## 🔍 Comandos Úteis

### Filtrar por Labels

```bash
# Listar por environment
kubectl get database -l environment=prod

# Listar por múltiplas labels
kubectl get database -l environment=prod,team=backend-team

# Listar com label selector
kubectl get database -l 'environment in (dev, staging)'

# Excluir com label
kubectl get database -l environment!=prod
```

### Ver Annotations

```bash
# Ver todas annotations
kubectl get database my-db -o jsonpath='{.metadata.annotations}'

# Ver annotation específica
kubectl get database my-db -o jsonpath='{.metadata.annotations.owner}'

# Adicionar annotation
kubectl annotate database my-db contact="team@example.com"

# Remover annotation
kubectl annotate database my-db contact-
```

---

## ✅ Checklist

Antes de criar um Claim, verifique:

### Labels ✅
- [ ] Sem caractere `@`
- [ ] Sem espaços
- [ ] Sem URLs (://)
- [ ] Max 63 caracteres
- [ ] Apenas: `a-z A-Z 0-9 - _ .`

### Annotations ✅
- [ ] Emails em annotations
- [ ] URLs em annotations
- [ ] Descrições longas em annotations
- [ ] Contatos em annotations

---

## 🚨 Erro Comum

### ❌ Erro

```bash
The Database "my-db" is invalid: metadata.labels: Invalid value: "team@example.com": 
a valid label must be an empty string or consist of alphanumeric characters, '-', '_' 
or '.', and must start and end with an alphanumeric character
```

### ✅ Solução

```yaml
# ANTES (Errado)
metadata:
  labels:
    owner: team@example.com  # ❌

# DEPOIS (Correto)
metadata:
  labels:
    team: backend-team  # ✅
  annotations:
    owner: team@example.com  # ✅
```

---

## 📚 Referências

- [Kubernetes Labels](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/)
- [Kubernetes Annotations](https://kubernetes.io/docs/concepts/overview/working-with-objects/annotations/)
- [Well-known Labels](https://kubernetes.io/docs/reference/labels-annotations-taints/)

---

**Dica**: Quando em dúvida, use annotations! Elas são mais flexíveis e não quebram seu YAML.

