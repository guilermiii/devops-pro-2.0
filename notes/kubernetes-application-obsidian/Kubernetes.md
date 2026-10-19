# Anotações de Estudos: Kubernetes

---

## 1. Arquitetura e Componentes do Cluster

O Kubernetes é uma plataforma open-source de orquestração de containers que automatiza a implantação, o dimensionamento e o gerenciamento de aplicações distribuídas. Arquiteturalmente, o cluster é dividido em duas camadas principais: o **Control Plane** (plano de controle) e os **Worker Nodes** (nós de trabalho).

![Arquitetura do Kubernetes - Control Plane e Worker Nodes](imagens/arquitetura-kubernetes.png)

### 1.1 Control Plane
Responsável pelo gerenciamento do cluster, tomada de decisões globais (como agendamento de pods) e detecção/resposta a eventos:

- **kube-apiserver:** Ponto central de entrada do cluster. Expõe a API REST do Kubernetes e valida/configura dados para os objetos (Pods, Services, ReplicationControllers, etc.). É o único componente que se comunica diretamente com o `etcd`.
- **etcd:** Banco de dados chave-valor distribuído, consistente e altamente disponível, que armazena todo o estado do cluster e suas configurações.
- **kube-scheduler:** Monitora Pods recém-criados que ainda não possuem um nó atribuído e seleciona o melhor Worker Node para executá-los, com base em afinidade, restrições de recursos (CPU/memória), taints/tolerations e políticas de QoS.
- **kube-controller-manager:** Executa os processos de controle em loop contínuo, comparando o estado atual com o estado desejado e efetuando correções. Inclui: Node Controller, ReplicaSet Controller, EndpointSlice Controller e ServiceAccount Controller.
- **cloud-controller-manager:** Permite integrar o cluster com APIs dos provedores de nuvem (AWS, GCP, Azure), gerenciando recursos específicos como Load Balancers de nuvem, rotas de rede e instâncias de nós.

### 1.2 Worker Nodes
Máquinas (físicas ou virtuais) onde as cargas de trabalho (cargas de aplicação nos Pods) de fato são executadas:

- **kubelet:** Agente principal em cada nó. Garante que os containers descritos nas especificações dos Pods (`PodSpecs`) estejam em execução e saudáveis. Realiza a supervisão e executa as sondas de saúde (*health checks/probes*).
- **kube-proxy:** Responsável pelo gerenciamento de regras de rede no nó (utilizando `iptables` ou `IPVS`). Reflete os serviços de rede definidos na API e viabiliza o roteamento de tráfego para os Pods corretos.
- **Container Runtime (CR):** Software fundamental encarregado de baixar as imagens e gerenciar a execução dos containers (ex.: `containerd`, `CRI-O`).

---

## 2. Interfaces Padronizadas do Kubernetes

O Kubernetes desacopla componentes de baixo nível por meio de três interfaces padronizadas baseadas em plugins:

![Interfaces do Kubernetes - CRI, CNI e CSI](imagens/interfaces-kubernetes.png)

- **CRI (Container Runtime Interface):** Interface que permite ao `kubelet` interagir com diferentes motores de containers sem necessidade de recompilação. Principais implementações: `containerd` e `CRI-O`.
- **CNI (Container Network Interface):** Gerencia a conectividade de rede dos Pods, alocação de IPs e comunicação cross-node. Principais plugins: `Calico`, `Flannel`, `WeaveNet`, `Cilium`.
- **CSI (Container Storage Interface):** Padroniza a conexão de volumes de armazenamento persistente externos aos containers (ex.: AWS EBS, Google Persistent Disk, Azure Disk, NFS, Ceph, Longhorn).

---

## 3. Pods e Padrões de Design

- **Papel do Pod:** A menor unidade de implantação no Kubernetes. Encapsula um ou mais containers que compartilham a mesma pilha de rede (mesmo endereço IP e porta via `localhost`), namespaces IPC e volumes de armazenamento.
- **Abordagem Declarativa vs. Imperativa:**
  - *Declarativa (Recomendada):* O usuário escreve manifestos YAML descrevendo o estado desejado (`kubectl apply -f manifesto.yaml`). O Kubernetes converge o estado atual até alcançar o desejado. Viabiliza versionamento e Infraestrutura como Código (IaC).
  - *Imperativa:* Comandos diretos de ação (`kubectl run`, `kubectl expose`, `kubectl scale`), úteis para testes rápidos ou diagnósticos pontuais.

### Design Pattern: Sidecar
Composto por um Pod contendo o container principal da aplicação e um ou mais containers auxiliares. Esses containers secundários assumem tarefas de suporte — como coleta e envio de logs (ex.: Fluentbit), exportação de métricas para o Prometheus ou proxy de rede/mTLS (ex.: Envoy em Service Meshes) — sem poluir ou alterar o código da aplicação principal.

---

## 4. Labels e Selectors

Mecanismo fundamental de identificação e agrupamento flexível de recursos no cluster.

![Labels e Selectors associando Pods a ReplicaSets e Services](imagens/labels-selectors.png)

- **Labels:** Pares de chave-valor (`key: value`) anexados aos metadados dos objetos (Pods, Services, Deployments, Nodes) para fins de identificação, controle e categorização (ex.: `app: web`, `ambiente: producao`, `versao: "2.0"`).
- **Selectors:** Mecanismos de consulta e filtragem baseados em labels.
  - Permitem que um **Service** descubra para quais Pods direcionar as requisições.
  - Permitem que um **Deployment/ReplicaSet** saiba quais Pods estão sob sua gestão através do campo `matchLabels` ou `matchExpressions`.

---

## 5. ReplicaSet vs. Deployment

![Deployment gerenciando ReplicaSets e Pods](imagens/deployment.png)

- **ReplicaSet:** Garante resiliência e escalabilidade horizontal, mantendo em execução o número exato de réplicas de Pods desejado (`spec.replicas`). **Limitação:** Não gerencia estratégias de atualização nem ciclo de vida de novas versões da aplicação.
- **Deployment:** Objeto de nível superior que gerencia e versiona ReplicaSets. Permite:
  - Atualizações contínuas sem indisponibilidade (*rolling updates*).
  - Estratégias Blue/Green e Canary.
  - Reversão rápida para versões anteriores (*rollbacks*).
  - Pausa e retomada de deploys.
- **Estrutura dos Manifestos:** A especificação do Pod (`spec.template`) dentro de um Deployment é idêntica à de um ReplicaSet. Na prática, altera-se apenas `kind: Deployment`, sendo uma boa prática evitar criar ReplicaSets diretamente.

---

## 6. Services

Abstração que define uma política de acesso estável e um conjunto lógico de Pods destino.

![Roteamento de tráfego com Service no Kubernetes](imagens/service.png)

### 6.1 Tipos de Services

1. **ClusterIP (Padrão):**
   - Cria um IP virtual exclusivo e fixo para comunicação interna dentro do cluster.
   - Fornece resolução DNS interna automática no formato `<nome-do-service>.<namespace>.svc.cluster.local`.
   - Atua como balanceador de carga interno entre os Pods selecionados.
   - **Diagnóstico e Teste de Conectividade Interna:**
     ```bash
     # Executa um Pod de diagnóstico temporário
     kubectl run debug-pod --rm -it --image ubuntu -- /bin/bash

     # Dentro do container do Pod temporário:
     apt update && apt install -y curl
     curl http://<nome-do-service>
     ```

2. **NodePort:**
   - Expõe a aplicação externamente alocando uma porta dedicada em cada nó do cluster (faixa padrão: `30000` a `32767`).
   - O tráfego externo acessa diretamente `<IP-de-qualquer-Node>:<NodePort>`.
   - Pode ser alocada aleatoriamente pelo K8s ou fixada no manifesto via `spec.ports[].nodePort`.

3. **LoadBalancer:**
   - Cria um balanceador de carga gerenciado no provedor de nuvem (AWS ELB/ALB, Google Cloud LB, Azure Load Balancer).
   - Encaminha o tráfego externo para o NodePort e, por consequência, para o ClusterIP e Pods.

4. **ExternalName:**
   - Não usa seletores nem aloca IPs virtuais; mapeia o serviço para um registro CNAME DNS externo (ex.: `meu-db.rds.amazonaws.com`).

---

## 7. Endpoints e EndpointSlices

- **Endpoints:** Recurso gerado automaticamente sempre que um Service possui seletores de labels. Lista os endereços IP e portas dos Pods saudáveis que estão prontos para receber requisições (`kubectl get endpoints`).
- **EndpointSlice:** Evolução arquitetural do Kubernetes criada para garantir escalabilidade em clusters de grande porte. Em vez de concentrar centenas ou milhares de IPs em um único objeto de Endpoints (o que causava alto consumo de rede e sobrecarga no `kube-proxy`), o EndpointSlice divide os alvos em fatias menores (por padrão, até 100 endpoints por slice).

---

## 8. ConfigMaps

Objeto utilizado para desacoplar configurações e parâmetros operacionais do código da imagem do container, seguindo as boas práticas dos *Twelve-Factor Apps*.

- **Segurança:** Indicado exclusivamente para dados **não sensíveis** (URLs de APIs, portas, nomes de bancos, flags de feature).
- **Ciclo de Atualização:** Modificar um ConfigMap não reinicia automaticamente os Pods que o utilizam. Para que as novas configurações entrem em vigor, é necessário reiniciar os Pods (ex.: `kubectl rollout restart deployment/<nome>`) ou empregar ferramentas de sincronização como o *Reloader*.
- **Formas de Configuração:**
  - **Declarativa (YAML):** Manifesto com pares chave-valor.
  - **Imperativa:** Criado via linha de comando (`kubectl create configmap config-app --from-literal=PORTA=3000`).
  - **Injeção como Variáveis (`valueFrom.configMapKeyRef`):** Injeta chaves pontuais do ConfigMap em variáveis específicas do container.
  - **Injeção em Bloco (`envFrom`):** Injeta todas as chaves do ConfigMap de uma vez como variáveis de ambiente.
  - **Montagem como Volume:** O ConfigMap é montado como diretório, onde cada chave se transforma em um arquivo no sistema de arquivos do container.

---

## 9. Secrets

Recurso voltado ao armazenamento e gestão de dados sensíveis (senhas, tokens de autenticação, chaves de API, certificados TLS).

- **Codificação vs. Criptografia:**
  - Por padrão, os dados em um Secret no `etcd` são apenas codificados em **Base64**, o que **NÃO** é criptografia (qualquer um com acesso ao manifesto ou comando `base64 -d` consegue ler o valor original).
  - Para criptografia real em repouso (*Encryption at Rest*), é necessário habilitar o provedor de criptografia no `kube-apiserver` com integração a um KMS (*Key Management Service*).
- **Sintaxe no Manifesto YAML:**
  - `data:` Requer que os valores sejam previamente convertidos para Base64 antes de salvar o arquivo (ex.: `echo -n 'senha123' | base64`).
  - `stringData:` Aceita valores em texto puro entre aspas. O próprio Kubernetes converte automaticamente os valores para Base64 ao aplicar o manifesto no cluster.
- **Boas Práticas de Segurança:**
  - Restringir o acesso aos Secrets via **RBAC** e **ServiceAccounts** com princípio do menor privilégio.
  - **Nunca comitar** manifestos de Secrets com valores reais em repositórios Git públicos ou privados.
  - Em ambientes produtivos, utilizar ferramentas dedicadas como **HashiCorp Vault**, **OpenBao**, ou operadores de segredos como o **External Secrets Operator (ESO)**.

---

## 10. Gerenciamento de Imagens nos Pods (`imagePullPolicy`)

Controla como e quando o `kubelet` deve baixar (*pull*) a imagem do container do registry de imagens (ex.: Docker Hub, GitHub Container Registry, AWS ECR).

### Políticas Disponíveis
```yaml
spec:
  containers:
    - name: app
      image: kubedevio/simulador-do-caos:v1
      imagePullPolicy: IfNotPresent # Opções: Always | IfNotPresent | Never
```

1. **`Always`:**
   - Em toda inicialização ou recriação do container, o nó contata o registry para verificar e baixar a imagem mais recente.
   - *Impacto:* Garante que a imagem esteja atualizada, mas consome mais banda de rede e atrasa o tempo de startup do Pod.

2. **`IfNotPresent` (Recomendado para produção):**
   - O Kubelet só baixa a imagem se ela não existir no cache local do Worker Node.
   - *Impacto:* Reduz o tempo de inicialização dos Pods e o tráfego de rede entre o cluster e o registry.

3. **`Never`:**
   - O Kubelet nunca tenta baixar a imagem do registry. Assume obrigatoriamente que a imagem já foi pré-carregada localmente no nó (cenário comum em testes offline, minikube com Docker local ou ambientes restritos).

### Regra de Inferência Automática do Kubernetes
- Se a tag da imagem for `:latest` ou se a tag for **omitida**, o Kubernetes define automaticamente `imagePullPolicy: Always`.
- Se a tag for específica e diferente de `:latest` (ex.: `:v1.0.0`, `:build-42`), o Kubernetes define automaticamente `imagePullPolicy: IfNotPresent`.

> [!WARNING]
> **Evite a tag `:latest` em ambientes produtivos!**
> O uso de `:latest` pode gerar nós com versões divergentes da mesma imagem, compromete a rastreabilidade e causa lentidão devido a pulls constantes com `imagePullPolicy: Always`. Prefira sempre tags imutáveis (SemVer) ou digests SHA256.

---

## 11. Comandos e Argumentos no Container (`command` e `args`)

No Kubernetes, é possível sobrescrever os comandos padrões definidos na criação da imagem do container (Dockerfile):

| Recurso no Kubernetes | Equivalente no Dockerfile | Descrição |
| :--- | :--- | :--- |
| `command` | `ENTRYPOINT` | O executável ou processo principal que será iniciado no container. |
| `args` | `CMD` | Os argumentos e parâmetros passados para o executável principal. |

### Exemplo de Manifesto
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-exec-exemplo
  labels:
    app: exec-demo
spec:
  containers:
    - name: busybox-worker
      image: busybox:latest
      # Sobrescreve o ENTRYPOINT da imagem:
      command: ["echo"]
      # Sobrescreve o CMD da imagem (argumentos passados para o echo):
      args: ["Mensagem de inicializacao processada com sucesso!"]
  restartPolicy: Never
```

---

## 12. Políticas de Reinicialização de Containers (`restartPolicy`)

A diretiva `restartPolicy` é configurada no nível de especificação do Pod (`spec.restartPolicy`) e instrui o `kubelet` sobre como proceder caso os containers do Pod encerrem ou falhem:

```yaml
spec:
  restartPolicy: Always # Opções: Always | OnFailure | Never
  containers:
    - name: web
      image: nginx:1.25
```

- **`Always` (Padrão):** O container é reiniciado automaticamente independentemente do motivo do encerramento (seja erro ou finalização com sucesso). Esse é o padrão em Deployments, StatefulSets e DaemonSets para manter serviços web sempre disponíveis.
- **`OnFailure`:** O container é reiniciado apenas se for encerrado com código de saída diferente de 0 (erro/crash). Se finalizar com código de saída 0 (sucesso), não há reinicialização. Padrão ideal para **Jobs** e **CronJobs**.
- **`Never`:** O container nunca é reiniciado pelo Kubelet, mantendo o status final após a conclusão ou erro. Útil para tarefas de depuração rápida e testes isolados.

---

## 13. Auto-recuperação e Probes (Health Checks)

As sondas (*probes*) permitem que o Kubernetes monitore a saúde da aplicação em tempo real, viabilizando resiliência e auto-recuperação (*self-healing*) sem intervenção humana.

![Fluxo e responsabilidades dos Probes no Kubernetes](imagens/probes-health-checks.png)

### 13.1 Os Três Tipos de Probes

1. **`startupProbe` ("A aplicação subiu e inicializou?"):**
   - Roda exclusivamente durante a inicialização do container.
   - Desativa temporariamente o `livenessProbe` e o `readinessProbe` até que o startup seja concluído com sucesso.
   - **Objetivo:** Evitar que aplicações lentas para inicializar (como aplicações corporativas em Java/JVM ou que realizam migração de banco no boot) sejam mortas precipitadamente pelo `livenessProbe`.

2. **`livenessProbe` ("A aplicação travou em deadlock/congelou?"):**
   - Roda periodicamente durante todo o ciclo de vida do Pod.
   - **Objetivo:** Detectar se a aplicação entrou em um estado irrecuperável (ex.: deadlock de threads, vazamento fatal de memória).
   - **Ação em caso de falha:** O Kubelet mata o container e dispara sua reinicialização conforme a `restartPolicy`.

3. **`readinessProbe` ("A aplicação está apta a receber tráfego agora?"):**
   - Roda periodicamente durante a execução da aplicação.
   - **Objetivo:** Verificar se o container está pronto para atender requisições de clientes (ex.: aguardando aquecimento de cache, carga inicial de dados ou alívio de sobrecarga momentânea).
   - **Ação em caso de falha:** O Kubernetes remove temporariamente o IP do Pod das listas de `Endpoints` e `EndpointSlices` do Service, interrompendo o envio de novas requisições até que a aplicação volte a responder positivamente. **Não reinicia o container!**

### 13.2 Mecanismos de Checagem
- **`httpGet`:** Dispara uma requisição HTTP GET para um endpoint do container (códigos de status entre 200 e 399 indicam sucesso).
- **`tcpSocket`:** Tenta abrir uma conexão TCP em uma porta específica (sucesso se o handshake TCP for completado).
- **`exec`:** Executa um comando internamente no container (sucesso se o exit code for 0).

### 13.3 Parâmetros de Configuração
- `initialDelaySeconds`: Tempo (em segundos) de espera após o container subir antes de disparar a primeira verificação.
- `periodSeconds`: Frequência (intervalo em segundos) com que a sonda será executada.
- `timeoutSeconds`: Tempo limite (em segundos) para considerar que a checagem falhou por timeout.
- `failureThreshold`: Número consecutivo de falhas necessárias para considerar o probe reprovado.
- `successThreshold`: Número consecutivo de sucessos necessários para considerar o probe aprovado após uma falha anterior.

> [!NOTE]
> No `livenessProbe` e no `startupProbe`, o campo `successThreshold` **deve ser estritamente igual a 1**. O Kubernetes rejeita o manifesto caso esse valor seja configurado com número superior.

### 13.4 Exemplo de Manifesto com Probes Completos
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-resiliente
  labels:
    app: app-resiliente
spec:
  replicas: 2
  selector:
    matchLabels:
      app: app-resiliente
  template:
    metadata:
      labels:
        app: app-resiliente
    spec:
      containers:
        - name: app
          image: kubedevio/simulador-do-caos:v1
          ports:
            - containerPort: 3000
          # 1. Startup Probe: aguarda até 30s no total (10 tentativas * 3s) para subir
          startupProbe:
            httpGet:
              path: /health
              port: 3000
            periodSeconds: 3
            failureThreshold: 10

          # 2. Liveness Probe: checa se travou; se falhar 2 vezes seguidas, reinicia
          livenessProbe:
            httpGet:
              path: /health
              port: 3000
              httpHeaders:
                - name: token
                  value: secret-health-token
            initialDelaySeconds: 5
            periodSeconds: 5
            timeoutSeconds: 3
            failureThreshold: 2
            successThreshold: 1

          # 3. Readiness Probe: checa se pode receber requisições; se falhar, isola do Service
          readinessProbe:
            httpGet:
              path: /ready
              port: 3000
            initialDelaySeconds: 3
            periodSeconds: 4
            timeoutSeconds: 2
            failureThreshold: 2
            successThreshold: 2
```

---

## 14. Gerenciamento de Ambientes: Namespaces

Os **Namespaces** fornecem um mecanismo de partição lógica de recursos dentro de um mesmo cluster físico de Kubernetes.

![Separação lógica de ambientes por Namespaces no Kubernetes](imagens/namespaces-k8s.png)

### 14.1 Finalidades Principais
- **Separação de Ambientes:** Isolamento entre `producao`, `homologacao` e `desenvolvimento`.
- **Organização por Equipes/Sistemas:** Divisão por squads, projetos ou domínios de negócio.
- **Governança e Cotas de Recursos:** Aplicação de políticas de consumo máximo de CPU/memória (`ResourceQuota` e `LimitRange`) e controle de acesso baseado em funções (`RBAC`) restrito por namespace.

### 14.2 Namespaces Padrão do Kubernetes
- **`default`:** Namespace padrão utilizado para objetos que não declaram um namespace específico.
- **`kube-system`:** Reservado para objetos e pods do sistema de controle do próprio Kubernetes (ex.: `CoreDNS`, `kube-proxy`).
- **`kube-public`:** Criado automaticamente e acessível publicamente (inclusive para usuários não autenticados). Usado principalmente para informações públicas do cluster.
- **`kube-node-lease`:** Abriga objetos de *Lease* que registram os heartbeats dos nós, permitindo ao control plane detectar rapidamente nós inativos ou indisponíveis.

### 14.3 Manifesto e Comandos Úteis
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: homologacao
  labels:
    ambiente: homologacao
```

```bash
# Listar todos os namespaces existentes
kubectl get namespaces

# Criar um novo namespace imperativamente
kubectl create namespace producao

# Listar pods de um namespace específico
kubectl get pods -n homologacao

# Alterar o namespace padrão ativo no contexto atual
kubectl config set-context --current --namespace=homologacao
```
