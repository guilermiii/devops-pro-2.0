# Cheat Sheet: Comandos Essenciais do Kubernetes

Guia prático de comandos diários e atalhos de linha de comando (`kubectl` / `k`) para administração, diagnóstico e automação no cluster Kubernetes.

---

## 0. Produtividade e Alias

Para agilizar a digitação no terminal diário:

```bash
# Configurar alias rápido no ~/.bashrc ou ~/.zshrc
alias k=kubectl
complete -o default -F __start_kubectl k
```

---

## 1. Descoberta e Informações do Cluster

- **Listar todos os recursos e objetos suportados pela API do cluster:**
  ```bash
  kubectl api-resources
  ```
- **Consultar a documentação e campos de qualquer recurso:**
  ```bash
  kubectl explain deployment.spec.template.spec.containers
  ```
- **Visualizar informações do cluster e control plane:**
  ```bash
  kubectl cluster-info
  ```

---

## 2. Inspeção e Listagem de Pods

- **Listagem detalhada de pods com IP e Nó de execução:**
  ```bash
  k get pods -o wide
  ```
- **Listar pods exibindo todas as labels atribuídas:**
  ```bash
  k get pods --show-labels
  ```
- **Filtrar pods por label específica:**
  ```bash
  # Filtro por igualdade:
  k get pods -l app=minha-app

  # Filtro por múltiplas labels:
  k get pods -l app=minha-app,ambiente=producao
  ```
- **Listar pods em todos os namespaces:**
  ```bash
  k get pods -A
  # ou:
  k get pods --all-namespaces
  ```

---

## 3. Gestão de Rollouts e Versionamento de Deployments

- **Verificar o histórico de revisões de um Deployment:**
  ```bash
  k rollout history deployment/nome-do-deploy
  ```
- **Verificar detalhes de uma revisão específica:**
  ```bash
  k rollout history deployment/nome-do-deploy --revision=2
  ```
- **Acompanhar o status em tempo real do rollout:**
  ```bash
  k rollout status deployment/nome-do-deploy
  ```
- **Reverter (Rollback) para a versão imediatamente anterior:**
  ```bash
  k rollout undo deployment/nome-do-deploy
  # Exemplo: se os pods rodando na v3 apresentarem falhas, o comando reverte o deployment para a v2.
  ```
- **Reverter para uma revisão específica do histórico:**
  ```bash
  k rollout undo deployment/nome-do-deploy --to-revision=1
  ```
- **Reiniciar os pods de um Deployment (força recarga de ConfigMaps/Secrets):**
  ```bash
  k rollout restart deployment/nome-do-deploy
  ```

---

## 4. Diagnóstico, Logs e Depuração

- **Inspecionar eventos detalhados e causas de erro de um recurso:**
  ```bash
  k describe pod <nome-do-pod>
  ```
- **Acompanhar logs de um pod em tempo real (follow):**
  ```bash
  k logs -f <nome-do-pod>
  ```
- **Visualizar logs do container anterior (em caso de crash/CrashLoopBackOff):**
  ```bash
  k logs -p <nome-do-pod>
  ```
- **Acessar o terminal interativo de um container:**
  ```bash
  k exec -it <nome-do-pod> -- /bin/sh
  # ou:
  k exec -it <nome-do-pod> -- /bin/bash
  ```
- **Encaminhar porta local diretamente para um pod ou service (Port-Forward):**
  ```bash
  k port-forward pod/<nome-do-pod> 8080:80
  k port-forward service/<nome-do-service> 8080:80
  ```

---

## 5. Criação Rápida de Manifestos Declarativos (Dry-run)

Gere a estrutura inicial de arquivos YAML sem aplicar diretamente no cluster:

- **Gerar manifesto de Pod:**
  ```bash
  k run meu-pod --image=nginx --dry-run=client -o yaml > pod.yaml
  ```
- **Gerar manifesto de Deployment:**
  ```bash
  k create deployment meu-deploy --image=nginx --replicas=3 --dry-run=client -o yaml > deployment.yaml
  ```
- **Gerar manifesto de Service ClusterIP:**
  ```bash
  k expose deployment meu-deploy --port=80 --target-port=8080 --dry-run=client -o yaml > service.yaml
  ```
- **Gerar manifesto de ConfigMap a partir de arquivo ou literal:**
  ```bash
  k create configmap app-config --from-literal=PORTA=3000 --dry-run=client -o yaml > configmap.yaml
  ```
