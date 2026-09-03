# Observações — Oficina de ferramentas (dois serviços, dois bancos, falha parcial)

## O que foi observado

- **Estado nominal**: os 4 contêineres (elegibilidade, exames, db_elegibilidade,
  db_exames) subiram healthy. Os bancos não têm porta publicada para a máquina
  host (só 5432/tcp interno) — só quem está na rede interna deles chega lá.
  As aplicações publicam em 0.0.0.0:18001 e 0.0.0.0:18002.
- **Operação bem-sucedida**: POST /exames com Elegibilidade no ar retornou
  201 Created com solicitacao_id e situacao "solicitado" — prova de que Exames
  chamou Elegibilidade por HTTP e só gravou depois da aprovação.
- **Falha parcial**: ao parar o contêiner elegibilidade, os outros três
  continuaram de pé. GET /exames/health seguiu retornando 200 OK
  ({"status":"ok","servico":"exames"}) no mesmo instante em que
  POST /exames retornava 503 Service Unavailable com código
  dependencia_indisponivel. Essa contradição — processo saudável, capacidade
  indisponível — é a evidência central desta oficina: "estar no ar" e
  "conseguir atender" não são a mesma coisa.
- **Recuperação**: subir elegibilidade de novo (up -d --wait) restaurou os
  dois /health para 200, e os 4 testes automatizados de
  tests/test_service_boundaries.py passaram, incluindo
  test_exames_makes_its_own_database_failure_observable.

## Qual dependência permanece saudável e qual capacidade deixa de ser concluída

O processo de Exames e o banco próprio dele (db_exames) permanecem saudáveis
durante toda a falha — Exames nunca perde acesso à sua própria base. A
capacidade que deixa de ser concluída é criar uma nova solicitação de exame,
porque essa operação especificamente depende de consultar Elegibilidade antes
de gravar. Qualquer operação de Exames que não dependesse do vizinho teria
continuado funcionando; a falha ficou contida no único caminho que atravessa
a fronteira entre os dois serviços.

## Por que 503 e não 500

O código em src/hospital/servicos/exames.py trata falha de rede/timeout ao
chamar Elegibilidade como 503 dependencia_indisponivel, não como 500. A
diferença de significado importa: 500 diria que Exames quebrou por dentro;
503 diz que o problema é temporário e externo, e que vale tentar de novo mais
tarde — informação verdadeira e acionável para quem consome a API de Exames.
Também confirmamos que Exames nunca repassa o status HTTP do vizinho: um 404
de Elegibilidade não vira 404 de Exames, porque significam coisas diferentes
(o recurso /exames existe; quem não existe é o beneficiário).

## Evidências nesta pasta

- config-validado.txt — validação estática do Compose (config --quiet + --services)
- ps-inicial.txt — os 4 contêineres healthy no estado nominal
- health-inicial-elegibilidade.txt / health-inicial-exames.txt — 200 OK antes da falha
- post-201-exame.txt — criação de solicitação bem-sucedida
- ps-degradado.txt — elegibilidade parado, os outros três de pé
- post-503-falha-parcial.txt — POST /exames falhando com dependencia_indisponivel
- health-exames-durante-falha.txt — GET /exames/health ainda 200 durante a falha
- health-recuperado-elegibilidade.txt / health-recuperado-exames.txt — 200 OK após recuperação
- testes-fronteiras.txt — 4 passed em tests/test_service_boundaries.py
