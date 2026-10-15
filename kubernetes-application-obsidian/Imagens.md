# Galeria de Diagramas e Arquiteturas: Kubernetes

Índice visual com todos os esquemas conceituais e diagramas de estudo utilizados nas anotações.

---

### 1. Arquitetura do Cluster (Control Plane & Worker Nodes)
Visão geral dos componentes centrais do Kubernetes (API Server, Controller Manager, Scheduler, etcd, Kubelet e Kube-proxy).

![Arquitetura do Kubernetes](imagens/arquitetura-kubernetes.png)

---

### 2. Interfaces do Kubernetes (CRI, CNI e CSI)
Padrões de integração com motores de container (CRI), plugins de rede (CNI) e drivers de armazenamento (CSI).

![Interfaces do Kubernetes](imagens/interfaces-kubernetes.png)

---

### 3. Labels e Selectors
Associação dinâmica entre ReplicaSets, Services e Pods através de filtros de rótulos.

![Labels e Selectors](imagens/labels-selectors.png)

---

### 4. Deployments e ReplicaSets
Estratégia de gerenciamento, versionamento e escalabilidade de Pods.

![Deployments e ReplicaSets](imagens/deployment.png)

---

### 5. Services e Roteamento de Tráfego
Balanceamento interno e abstração de rede para comunicação estável entre Pods.

![Services no Kubernetes](imagens/service.png)

---

### 6. Probes e Ciclo de Saúde (Health Checks)
Fluxo do Load Balancer e responsabilidades de cada sonda (`startupProbe`, `livenessProbe`, `readinessProbe`).

![Probes e Health Checks](imagens/probes-health-checks.png)

---

### 7. Namespaces e Isolamento de Ambientes
Divisão lógica de recursos no cluster entre produção, homologação e projetos.

![Namespaces no Kubernetes](imagens/namespaces-k8s.png)