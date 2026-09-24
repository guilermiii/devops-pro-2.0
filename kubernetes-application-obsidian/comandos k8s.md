kubectl api-resources - lista todos objetos do k8s

k get pods -o wide - eh um k get pods mais detalhado

k get pods -l label:exemplo - busca por labels

k rollout history deployment - lista as ultimas atualizacoes do deployment

k rollout undo deployment nome-do-deploy - volta pra ultima versao do deployment antes da atual utilizada.
exemplo pods rodando na v3 dando problema, usando o rollout ele voltaria para a v2.

