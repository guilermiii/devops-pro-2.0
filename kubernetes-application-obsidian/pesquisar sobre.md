# Tópicos de Estudo e Pesquisa: DevOps & Cloud Native

Backlog de tecnologias, conceitos e ferramentas do ecossistema para aprofundamento e estudos futuros.

---

## 1. CNCF (Cloud Native Computing Foundation)
- **O que é:** Fundação parte da Linux Foundation que hospeda e governa os principais projetos de código aberto do ecossistema cloud-native (Kubernetes, Prometheus, Envoy, Helm, Cilium, etc.).
- **O que explorar:**
  - O **CNCF Landscape** (mapa interativo de soluções).
  - Níveis de maturidade dos projetos: *Sandbox*, *Incubating* e *Graduated*.
  - Padrões abertos e especificações como OpenTelemetry, OCI (Open Container Initiative) e CNI/CSI/CRI.

---

## 2. PoC (Proof of Concept - Prova de Conceito)
- **O que é:** Implementação experimental e reduzida com o objetivo de validar a viabilidade técnica, arquitetural e financeira de uma ideia ou ferramenta antes da adoção em larga escala na organização.
- **Boas Práticas para PoCs no Kubernetes:**
  - Definir critérios de sucesso e métricas objetivas (latência, consumo de memória, tempo de deploy).
  - Testar cenários de falha real (chaos engineering, quebra de nós, estresse de rede).
  - Planejar o descarte ou a esteira de migração segura para o ambiente produtivo.

---

## 3. OpenTofu (Fork Open Source do Terraform)
- **O que é:** Bifurcação (*fork*) totalmente aberta e orientada pela comunidade do Terraform, mantida sob a tutela da Linux Foundation.
- **Contexto:** Criado após a HashiCorp alterar a licença do Terraform de MPL-2.0 (código aberto) para BSL/BUSL-1.1 (Business Source License).
- **Importância:** Garante continuidade para esteiras de IaC (Infraestrutura como Código) com suporte comunitário e compatibilidade com provedores e módulos existentes.

---

## 4. OpenBao (Fork Open Source do HashiCorp Vault)
- **O que é:** Fork comunitário do HashiCorp Vault mantido pela Linux Foundation.
- **Contexto:** Também originado após a mudança de licenciamento da HashiCorp em 2023.
- **Importância:** Solução open-source para gerenciamento seguro de segredos, certificados, criptografia em trânsito e controle de acesso sensível em pipelines DevOps e clusters Kubernetes.
