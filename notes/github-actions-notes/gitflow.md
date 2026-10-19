trabalhando de maneira colaborativa no git
ao desenvolver uma feature - criar uma branch
sharepick ->  pegar alteracoes especificas de um commit especifico e nao tudo.
rebase - junta de maneira desorganizada em comparacao ao merge

-----

com varios membros tem que ter uma certa organizacao para poder trabalhar de maneira padronizada evitando baguncas

alguns padroes sao o gitflow gitlabflow githubflow e outros - nao precisa seguir a risca, é um template pra vc seguir e adaptar da sua maneira, levando em conta varios fatores como maturidade da empresa, da equipe, do gestor de codigos e etc.

-----
##gitflow 
 -- main -> versao de producao
 -- develop -> versao de novas features/ desenvolvimento. aqui sao criadas varias branchs, cada dev que ta trabalhando em uma feature, precisa criar uma branch, upar suas alteracoes lá, e nao na branch develop.
todo novo recurso vai gerar uma nova breanch de feature, geralmente o nome é feature/recurso-novo.
isso ocorre em toda sprint nas empresas.

-- release -> apos algumas novas features, é feito o merge na branch de develop e enviado para uma branch intermediaria que seria a branch de release, onde será testado de maneira exaustiva na busca de erros, qualidade esperada e etc, validando tudo e tendo a nova release indo pra branch main.

a branch de nova feature e release vao ser temporarias, usadas apenas durante o periodo de usabilidade delas.
bugs em producao é preciso ter uma estratégia.
criar uma branch de hotfix e corrigir o bug, na hora de realizar o merge, tem que ser feito o merge tanto com a branch main quanto com a branch develop se nao o bug volta na nova release quando subit da devlop.

quando o time é grande, fica dificil de lidar com esse flow, geralmente sempre da problema. tem que rodar todo o fluxo de ci cd em cada branch, é o mais usado mas é o mais complexo, mas ele tem seus pontos positivos que compensam tudo isso. sua utilizam padroniza e as equipes conseguem trabalhar de maneira organizada.

----
## githubflow

