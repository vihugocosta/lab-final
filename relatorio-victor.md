# Relatorio - Victor

## 1.1 - Projeto de desenvolvimento

Projeto ativo:

`Using project "victor-dev" on server "https://api.riopomba.aopatricio.com:6443".`

## 1.2 - Descobrir a API

1. Grupo da API do recurso `Route`: `route.openshift.io/v1`.
2. Valores aceitos para terminacao TLS de uma Route:
   - `edge`
   - `passthrough`
   - `reencrypt`
3. O campo `type` de um Service determina como ele sera exposto. O valor padrao e `ClusterIP`. Os tipos disponiveis sao `ClusterIP`, `ExternalName`, `NodePort` e `LoadBalancer`.

## 1.3 - Console

- Topology do projeto `victor-dev`, perspectiva Developer: abra o menu `Developer`, selecione o projeto `victor-dev` e acesse `Topology`.
- Download da CLI `oc`: abra `?` ou `Help` no canto superior direito e escolha `Command Line Tools`.
- Definicao do recurso `Deployment`: no menu `Home` ou `Search`, abra `API Explorer`, procure por `Deployment`; a versao listada e `apps/v1`.

Console: https://console-openshift-console.apps.riopomba.aopatricio.com

## 2.1 - Nos

- Quantidade de nós: 1.
- Nós control-plane/master/worker: 1.
- Nome do nó: `cluster-riopomba`.
- O nó possui simultaneamente os papéis `control-plane`, `master` e `worker`.

## 2.2 - Saude dos Cluster Operators

- Nenhum Cluster Operator com `DEGRADED=True`.

## 2.3 - MachineConfigPools e Operators

- MachineConfigPool `master`: `UPDATED=True`.
- MachineConfigPool `worker`: `UPDATED=True`.
- Operator OpenShift Pipelines: `Succeeded`.
- Operator OpenShift GitOps: `Succeeded`.

## 2.4 - Node x Machine

Um `Node` e o objeto do Kubernetes que representa um no registrado e disponivel para executar cargas no cluster. Uma `Machine` e a descricao da infraestrutura subjacente gerenciada pelo OpenShift, usada pelo Machine API para provisionar ou substituir esse no.

## 3.1 - Pod avulso relogio-agenda

- Status: `Running`, `Ready 1/1`, `RESTARTS 0`.
- Linha nos logs: `agenda pronta`.
- Nome do pod retornado pela variavel de ambiente e pelo kernel: `relogio-agenda`.
- Nota: a imagem `ubi-minimal` nao inclui o executavel `hostname`; os metodos equivalentes usando `$HOSTNAME` e `/proc/sys/kernel/hostname` confirmaram o nome do pod.
- Limpeza: o pod foi removido ao fim da tarefa.

## 3.2 - Deployment e autorrecuperacao

- Pod removido: `vitrine-7ccd547546-qbt8f`.
- Pod substituto: `vitrine-7ccd547546-hjwl8`.
- Contagem final: 3 podos `Running`, prontos, com replicas disponiveis iguais a 3.

## 3.3 - Diagnostico de imagem

- Estado final do pod `vitrine-antiga`: `ImagePullBackOff`.
- Erro relevante: `manifest unknown` para a tag `versao-antiga`.
- Causa: a tag `versao-antiga` nao existe no registry da imagem `registry.access.redhat.com/ubi9/nginx-126`.
- Limpeza: o Deployment `vitrine-antiga` foi removido.

## 3.4 - Diagnostico de reinicio

- Pod: `instavel`.
- Estado: `CrashLoopBackOff`.
- Codigo de saida do ultimo estado: `4`.
- Numero de reinicios: `5`.
- Frase nos logs: `iniciando`.
- Limpeza: o pod `instavel` foi removido.

## 3.5 - Acesso local

- Encaminhamento: porta local `9100` para a porta do container `8080`.
- Resposta de `http://localhost:9100`: `200`.

## Questoes do bloco 3

1. Um pod avulso nao possui controlador que recrie ou substitua o pod apos sua remocao ou falha. Um pod gerenciado por um Deployment e observado por seu ReplicaSet, que mantem a quantidade desejada de replicas e cria substitutos.
2. Em `ImagePullBackOff`, a causa esta na obtencao da imagem; primeiro consultar eventos e a mensagem de pull, indicando tag ou permissao ausente. Em `CrashLoopBackOff`, a imagem foi obtida e o processo continua terminando; primeiro olhar o codigo de saida no ultimo estado e os logs da execucao anterior.

## 4.1 - Permissoes

- Criar Deployments em `victor-dev`: `yes`.
- Criar namespaces no cluster: `no`.
- Ler Secrets em `openshift-gitops`: `no`.
- RoleBindings do projeto: `admin`, `openshift-pipelines-edit`, `pipelines-scc-rolebinding`, `system:deployers`, `system:image-builders`, `system:image-pullers`.

## 4.2 - Identidade da aplicacao

- ServiceAccount do Deployment `vitrine`: `agenda-sa`.

## 4.3 - UID e SCC

- Faixa de UID do projeto: `1001750000/10000`.
- SCC aplicada ao pod: `restricted-v2`.
- UID efetivo do processo principal: `1001750000`.
- O UID nao e zero porque a SCC restrita atribuiu ao pod o primeiro UID aleado da faixa do projeto.

## 4.4 - Segredo

- Base64 nao protege o valor: ele apenas codifica os dados e qualquer usuario com acesso ao Secret pode decodifica-lo.
- O Secret `conexao-agenda` foi criado com uma senha de teste, decodificado e removido ao final do bloco.

## Questoes do bloco 4

1. Usuario representa uma identidade humana ou externa que faz autenticacao no cluster. ServiceAccount representa uma identidade para processos e workloads dentro do Kubernetes.
2. `view` permite ler recursos, mas nao altera-los. `edit` acrescenta permissao para criar, alterar e remover a maioria dos recursos do namespace, exceto configuracoes de RBAC.
3. `anyuid` pode ser necessario quando uma aplicacao exige um UID especifico e nao respeita a faixa do projeto. O impacto deve ser limitado por ServiceAccount ou namespace, usando o minimo de pods e uma SCC customizada ou vinculacao restrita.

## 5.1 - BuildConfig com estrategia Docker

- BuildConfig: `agenda`.
- Fork usado: `https://github.com/vihugocosta/lab3-openshift-app.git`.
- Build concluido: `agenda-1`, com status `Complete`.
- ImageStream: `agenda`, tag `v1`.

## 5.2 - Gatilhos

- Gatilho real configurado no BuildConfig: `ConfigChange`.
- Novo build disparado: `agenda-3`.
- Status do novo build: `Complete`.

## 5.3 - Tag e digest

- Digest da tag `v1`: `sha256:171ea97d55249893a4cee6f071abfeb5d819ad43cd259e879240bbcf3666fe87`.
- O digest e mais reprodutivel que uma tag porque identifica exatamente o conteudo da imagem e nao pode mudar para apontar para outra imagem da mesma forma.

## 5.4 - S2I

Eu escolheria S2I para uma linguagem ou framework suportado quando nao for necessario controlar manualmente todas as etapas da imagem. A aplicacao `agenda` ja fornece um Dockerfile funcional, entao a estrategia Docker foi adequada para este laboratorio.

## 6.1 - Route HTTP

- Host: `vitrine-victor-dev.apps.riopomba.aopatricio.com`.
- Resposta HTTP: `200`.

## 6.2 - Route com TLS

- Host: `vitrine-tls-victor-dev.apps.riopomba.aopatricio.com`.
- Terminacao: `edge`.
- Politica HTTP: `Redirect`.
- Resposta HTTPS: `200`.

## 6.3 - Armazenamento persistente

- PVC: `arquivos-agenda-pvc`.
- Estado: `Bound`.
- StorageClass: `local-path`.
- Capacidade: `2Gi`.
- Modo de acesso: `RWO`.
- Caminho de montagem: `/opt/app-root/src/arquivos`.
- O Deployment foi reduzido temporariamente para 1 replica e restaurado para 3.

## 6.4 - Template

- Parametro de tamanho do volume: `VOLUME_CAPACITY`.
- Valor padrao: `1Gi`.

## Questoes do bloco 6

1. Um Service fornece acesso de rede interno aos pods e faz balanceamento por seletores. Uma Route expoe esse Service externamente por hostname HTTP/HTTPS.
2. A StorageClass define como o cluster provisiona PVs. O PVC solicita armazenamento a partir dela. O PV e o volume fisico efetivamente provisionado que atende a reclamacao.
3. Template e um modelo OpenShift parametrizado que gera varios recursos a partir de valores fornecidos. E util para padronizar implantacoes repetiveis de aplicacoes.

## 7.1 - Consumo de recursos

- Maior consumo de memoria entre os pods consultados: os tres pods `vitrine` empatam em `9Mi`.
- Pod registrado: `vitrine-5c76469f5b-58qx2`, com `9Mi` e `0m` de CPU.
- A consulta de consumo do node por `oc adm top nodes` foi negada pela permisao do usuario para `nodes.metrics.k8s.io`; por isso nao inventei um valor de CPU do node.

## 7.2 - Eventos de aviso

- Quantidade de eventos `Warning` no projeto: `11`.
- Mensagem de um evento: `Back-off restarting failed container instavel`.

## 7.3 - Metricas no console

- Perspectiva: `Administrator > Observe > Metrics`.
- Metrica: `kube_deployment_status_replicas_available`.
- Deployment `vitrine`: desejadas `3`, disponiveis `3`.

## 7.4 - Logs e inspeção

- Coleta de logs kubelet negada pela falta de acesso a `nodes/proxy`.
- Pasta da coleta `inspect` gerada: `C:\Users\RPV02\Documents\lab-final\inspect-victor-dev`.
- A coleta foi parcial por falta de permissao para recursos OVN e ServiceMonitor, mas os recursos principais do namespace foram inspectados.

## Questoes do bloco 7

1. Uso `oc adm top` para uma leitura rapida no terminal e continuo no console de metricas para historico, filtros, PromQL e correlacao visual.
2. `must-gather` coleta logs e artefatos do cluster para diagnostico ampliado e normalmente e enviado ao suporte da Red Hat ou ao instrutor/equipe responsavel.
3. Evento `Normal` informa uma atividade esperada ou concluida com sucesso. Evento `Warning` informa falha, problema ou condicao anormal.

## 8.1 - Ambiente de CI

- Operator OpenShift Pipelines: `Succeeded`.
- ServiceAccount `pipeline`: existe em `victor-dev`.
- A lista de pods de `openshift-pipelines` foi negada pela RBAC deste usuario.

## 8.2 - Projeto e credencial

- Projeto criado: `victor-prod`.
- Secret criado: `registry-agenda`, tipo `docker-registry`.
- Registry: `image-registry.openshift-image-registry.svc:5000`.
- O secret aparece em `secrets` e `imagePullSecrets` da ServiceAccount `pipeline`.

## 8.3 - Recursos do pipeline

- PVC: `fonte-agenda-pvc`, `1Gi`, `RWO`.
- Pipeline: `agenda-pipeline`.
- Workspaces: `source`.
- Resultado: `image-digest`, ligado a `$(tasks.build.results.IMAGE_DIGEST)`.
- Task `clone`: resolver `cluster`, task `git-clone`, parametro real de entrada `URL`, workspace `output`.
- Task `build`: resolver `cluster`, task `buildah`, dependente de `clone`, parametro `IMAGE`, workspace `source`.
- Arquivos:
  - `C:\Users\RPV02\Documents\lab-final\fonte-agenda-pvc.yaml`
  - `C:\Users\RPV02\Documents\lab-final\agenda-pipeline.yaml`

## 8.4 - Execucao

- PipelineRun: `agenda-pipeline-run-f6bx2`.
- Status: `Succeeded`.
- Task `clone`: `Succeeded`.
- Task `build`: `Succeeded`.
- A primeira tentativa `agenda-pipeline-run-w9bkz` falhou na validacao porque o nome real do parametro da task era `URL`; ela foi corrigida no Pipeline.

## 8.5 - Digest

- `sha256:ec9075745fb5d98ac07d7081d1213456495eb01e72ec7099d701e9f685e3acd1`.

## 8.6 - Console

- No console Developer, em `Pipelines`, a execucao fica aberta para mostrar a visualizacao das tasks e o log de cada task em seu proprio card.

## 8.7 - Discussao

Um `git push` pode ser recebido por uma configuracao de Trigger/Interceptor ou pelo Pipelines as Code. A configuracao valida o evento e gera um EventListener/PipelineRun com os parametros do repositorio. O Tekton inicia o PipelineRun, resolve as tasks e executa clone e build. A assinatura ou registro pode ser realizado depois, conforme o pipeline.

## Questoes do bloco 8

1. Task e uma unidade reutilizavel de passos. Pipeline organiza tarefas e fluxos entre elas. TaskRun executa uma Task. PipelineRun executa um Pipeline.
2. Uma task publica um `results` e outra Pipeline Task o usa com uma referencia como `$(tasks.nome.results.RESULTADO)`.
3. As credenciais ficam na ServiceAccount para serem entregues ao processo de execucao sem serem codificadas nos steps ou no Pipeline.
4. CI nao deve fazer deploy direto em producao porque separa verificacao e construcao da decisao de promocao, preservando aprovacao, rastreabilidade e recuperacao pelo Git.
