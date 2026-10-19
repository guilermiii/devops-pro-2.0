cada commit da branch main tem uma versao, sempre que for criar um novo recurso, um hotfix, o que quer que seja, sempre sera uma branch de feature, tem quem separa branch fea
ture e branch hotfix
vai mais da padronizacao da equipe

o codigo nao vai pra branch main duma vez, é feito um pull request que deve ser aceito para ir para a branch principal.
nesse processo de pull request vai ter o processo de code review, qualidade de codigo, linters e etc. pipelines de verificacao de qualidade de codigo.

muito bom para equipes pequenas e para desenvolvimento solo.

essa falta de separação de ambientes, corre risco de levar mais bugs para produção, fica dificil de criar mais de um ambiente já que a branch deployavel é a main.
