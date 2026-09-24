### Arquitetura e Padrões de Pods

No Kubernetes, existem diversos _design patterns_. Um dos mais utilizados é o **Sidecar**, composto por um Pod contendo o container principal da aplicação e um ou mais containers auxiliares. Esses containers secundários assumem tarefas de apoio — como coleta de logs, monitoramento/métricas ou proxy de rede — sem inflar o código da aplicação principal.

- **Papel do Pod:** A menor unidade de implantação no Kubernetes. Sua função primária é encapsular e executar containers, compartilhando a pilha de rede (mesmo IP e `localhost`) e volumes de armazenamento entre eles.
    
- **Abordagem Declarativa:** O gerenciamento dos recursos é feito de forma declarativa via manifestos YAML. O usuário define o estado desejado da aplicação, e o Kubernetes se encarrega de convergir o estado atual para o desejado, viabilizando automação e infraestrutura como código (IaC).
    

### Labels e Selectors

- **Labels:** Pares de chave-valor (`key: value`) anexados aos objetos (Pods, Services, Deployments) para identificação, agrupamento e categorização.
    
- **Selectors:** Mecanismos de consulta e filtragem baseados em labels. Permitem associar recursos entre si — por exemplo, permitindo que um Service localize os Pods para os quais deve rotear o tráfego.
    

### ReplicaSet vs. Deployment

- **ReplicaSet:** Garante a resiliência e a escalabilidade, mantendo em execução o número exato de réplicas desejado (ex.: escalando de 3 para 15 Pods sob demanda). **Limitação:** Não gerencia o ciclo de vida de atualização da aplicação (rollout de novas versões).
    
- **Deployment:** Objeto de nível superior que gerencia e versiona ReplicaSets. Ele viabiliza atualizações contínuas (_rolling updates_), pausa de deploy e reversão de versões (_rollbacks_).
    
- **Estrutura dos Manifestos:** A especificação do Pod (`template`) dentro de um Deployment é idêntica à de um ReplicaSet. Na prática, altera-se apenas o campo `kind: Deployment`, dispensando a criação direta de ReplicaSets.
    

### Services

Abstração que define um conjunto lógico de Pods e a política de acesso a eles.

- **ClusterIP (Padrão):**
    
    - Provê um IP virtual estável e exclusivo para comunicação interna no cluster.
        
    - Atua como balanceador de carga interno entre os Pods selecionados.
        
    - **Validação e Diagnóstico:** Para testar a resolução DNS e a conectividade interna:
        
        Bash
        
        ```
        # Executa um Pod de diagnóstico temporário
        kubectl run debug-pod --rm -it --image ubuntu -- /bin/bash
        
        # Dentro do container:
        apt update && apt install -y curl
        curl <nome-do-service>
        ```
        
- **NodePort:**
    
    - Expõe o serviço externamente reservando uma porta dedicada em cada nó do cluster (faixa padrão: `30000` a `32767`).
        
    - A porta pode ser alocada aleatoriamente ou fixada estaticamente no manifesto via campo `nodePort`.
        
    - O acesso externo é feito diretamente através de `<IP-de-qualquer-Node>:<NodePort>`.
        
- _(Nota: Os tipos `LoadBalancer` e `ExternalName` serão abordados nas próximas seções)._
    

### Endpoints e EndpointSlices

- **Endpoints:** Sempre que um Service com seletores é criado, o Kubernetes gera automaticamente um recurso de _Endpoints_ com os IPs e portas dos Pods aptos a receber requisições (`kubectl get endpoints`).
    
- **EndpointSlice:** Evolução arquitetural desenvolvida para clusters de grande porte. Em vez de concentrar centenas de IPs em um único objeto monolítico de Endpoints, o EndpointSlice particiona esses endereços em blocos menores (por padrão, até 100 alvos por slice), reduzindo o consumo de rede e a sobrecarga no `kube-proxy`.
    

### ConfigMaps

Objeto utilizado para desacoplar configurações e variáveis de ambiente do código da imagem do container.

- **Segurança:** Indicado exclusivamente para dados **não sensíveis** (URLs, portas, parâmetros gerais).
    
- **Ciclo de Atualização:** Atualizar um ConfigMap não reinicia automaticamente os Pods que o utilizam. Para que novas variáveis entrem em vigor, é necessário realizar um rollout do Deployment (ex.: `kubectl rollout restart deployment/<nome>`) ou utilizar ferramentas automatizadas (como o _Reloader_).
    
- **Formas de Configuração:**
    
    - **Declarativa:** Via arquivo de manifesto YAML.
        
    - **Imperativa:** Criado diretamente via linha de comando (`kubectl create configmap ...`).
        
    - **Referência de Chave (`valueFrom.configMapKeyRef`):** Permite injetar apenas chaves pontuais do ConfigMap em variáveis específicas do container.
        

### Secrets

Recurso voltado ao armazenamento de informações sensíveis (senhas, chaves de API, certificados TLS).

- **Codificação vs. Criptografia:** Por padrão, os dados em um Secret no etcd são apenas codificados em **Base64**, o que não equivale a criptografia.
    
- **Boas Práticas de Segurança:**
    
    - Restringir o acesso aos segredos via **RBAC** e **ServiceAccounts** com privilégios mínimos.
        
    - Evitar comitar manifestos de Secrets em repositórios Git.
        
    - Para ambientes produtivos, integrar com soluções externas de cofre de senhas (como **HashiCorp Vault**, **OpenBao** ou KMS dos provedores de nuvem).
        
- **Sintaxe no Manifesto YAML:**
    
    - `data:` Requer que os valores sejam previamente convertidos em Base64 antes de salvar o arquivo.
        
    - `stringData:` Aceita valores em texto puro (strings entre aspas). O Kubernetes converte automaticamente os valores para Base64 ao aplicar o manifesto, facilitando a escrita sem comprometer o formato interno do cluster.

######################
gerenciamento de imagens no k8s - restart police - como executar comandos e argumentos na criacao do pod - recursos de recuperacao

imagePullPolicy - gerenciamento de imagens
	aways -> todo pode que sobe, ele baixa a imagem novamente no hub,
	if not present -> so busca imagem se nao tiver no nó , acaba sendo mais uma seguranca ja que evitamos usar a tag latest nas imagens.
	a policy pode ser declarada no manifesto, por default ela fica habilitada como aways, explicitamente conseguimos utilizar dois valores alem do default (aways), que seria o if not present e o never (precisa colocar as imagens antes no cluster para funcionar, mas sao raros os casos de uso)
	```

```
imagePullPolicy: IfNotPresent | Never | Aways

``` 

restart policies - se refere ao container e nao o POD
especificado na declaracao do POD

aways - reiniciar sempre (ignorando se foi encerrado, se esta com erro)
nunca reiniciar
reiniciar somente se deu erro no container.
O pod reinicia o container individualmente, a depender da politica aplicada

```yaml
containers:
- name: exemplo
  image: nginx
  ports:
  - containerPort:80
restartPolicy: OnFailure ##restart somente com erro no container, se for encerrada sem erros nao tem o restart
restartPolicy: Aways ## restart independente do estado do container encerrado
restartPolicy: never ##nunca faz o restart, fica com status de erro ate deletar e subir novamentem

```


probe - maneira do k8s verificar a saude da aplicacao periodicamente, seja antes de subir o pod ou com o pod rodando

o kubelet é o responsável por manter o pod de pé, rodando, se no manifesto especificamos 3 pods ele tem que subir 3 pods e garantir o funcionamento deles, ele vai receber a especificação de criação e posteriormente gerenciar esses pods, então quem faz o health-check é o kubelete.


liveness x readiness 
o liveness testa periodicamente nosssa aplicacao, caso encontre erros, ele restarta o pod, (self-heling), existem parametros especificos para verificar falso positivo e falso negativo e existem diferentes maneiras de configurar o liveness e readiness, com diversos parametros, o readiness faz sua verificacao antes de subir o pod, verificando se a aplicacao esta apta a receber trafego.


```yaml
livenessProbe:
httpGet: #tipo de verificação
  port: 3000
  path: /health
  http:Headers: ## headers a serem enviados na requisição
    - name: token-exemplo
      value: sdajkldasjlkasd
initialDelaySeconds: 5 ## tempo para comecar a verificação
periodSeconds: 10 ## intervalo de tempo entre cada verificação
failureThreshold: 3 ## quantidade de falhas antes de reiniciar, 
successThreshold: 1 # caso apresente sucess reinicia o contador de falha
```
O livenessProbe funciona  com o ```sucessThreshold: 1``` (sempre 1 - geralmente nem se coloca no manifesto, se tentar aumentar o k8s bloqueia e nao deixa aplicar)

as formas de verificar sao com o httpGet , o exec (comando) e o tcp que vai ficar pingando numa porta especifica






