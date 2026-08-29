# Observações — Oficina de ferramentas (contrato, execução e comparação)

## Contrato explícito vs. execução

O contrato (openapi.yaml) documenta a regra do CPF via schema
(`pattern: "^\d{11}$"`) e via exemplo em requestBody.examples.pedidoValido.
Confirmamos que essas duas fontes são checadas em momentos diferentes:

- Spectral valida o documento contra si mesmo (exemplo vs. schema), sem
  nunca chamar o servidor. Isso é uma verificação estática.
- O servidor (FastAPI + Pydantic) valida a mesma regra em runtime, quando
  uma requisição chega de verdade — e devolve 422 com
  codigo: "dados_invalidos" e campo: "body.cpf" quando o CPF é inválido
  ou está ausente (ver resposta-post-422.txt).

Ou seja: contrato e implementação impõem a mesma regra por caminhos
independentes. Um não substitui o outro — o Spectral não prova que o
servidor obedece o contrato, e o servidor rodando não prova que o
contrato documentado está correto.

## Falha deliberada

Copiamos o contrato para evidencias/openapi-experimento.yaml e alteramos
apenas o exemplo do CPF no requestBody (paths./elegibilidades.post.
requestBody.content.application/json.examples.pedidoValido.value.cpf) de
"12345678901" para "123", sem tocar no exemplo de components.schemas.

Rodando Spectral nessa cópia, a regra oas3-valid-media-example acusou o
erro: "cpf" property must match pattern "^\d{11}$", com código de saída 1.
Essa falha é esperada e desejada — ela prova que o linter detecta um
exemplo que descumpre o próprio schema do contrato, antes de qualquer
chamada ao servidor. Não é um defeito pendente: é a evidência de que a
verificação estática funciona. O arquivo original (contratos/openapi.yaml)
não foi alterado.

## Contrato gerado (app.openapi()) vs. contrato declarado

Não aprofundamos essa comparação nesta entrega (extensão opcional do
roteiro) — os testes de tests/test_api_contract.py (7 passed) validam o
comportamento da implementação contra o contrato declarado em
contratos/openapi.yaml, que é a evidência que sustentamos aqui.

## Evidências desta pasta

- spectral-valido.txt — lint do contrato original, sem erros
- testes-contrato.txt — 7 testes de tests/test_api_contract.py, todos aprovados
- resposta-post-202.txt — POST /elegibilidades com pedido válido
- resposta-get-200.txt — GET /elegibilidades/{protocolo} do protocolo criado acima
- resposta-post-422.txt — POST /elegibilidades sem o campo cpf
- openapi-experimento.yaml — cópia com a falha deliberada no exemplo do CPF
- bruno/ — coleção importada do contrato, com os 4 exemplos testados manualmente
