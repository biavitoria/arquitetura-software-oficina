Condição alterada: em plugins/impostos_rj.py, na classe ImpostoRJPlugin,
troquei ALIQUOTA = 0.20 por ALIQUOTA = 0.25.

O que a saída revelou: apenas a Fatura #1002 (Rio de Janeiro) mudou. ICMS-RJ
foi de R$1.680,00 para R$2.100,00 (8.400 × 25%), e o TOTAL subiu de
R$10.080,00 para R$10.500,00. O e-mail de notificação já saiu com o valor
novo e correto (Total: R$10.500,00), confirmando que ORDEM_CATEGORIAS =
["impostos", "frete", "notificacao"] garante que a notificação sempre vê o
resultado final, depois de impostos e frete já terem sido somados. As
faturas #1001 (SP), #1003 (SP) e #1004 (MG) não mudaram em nada, porque
ImpostoRJPlugin só age quando fatura.cliente.estado == "RJ" — os outros
plugins de imposto (ICMS-SP, ISS-SP, ICMS-MG) nunca são chamados para uma
fatura de outro estado.

Responsabilidade arquitetural: a alíquota de um imposto estadual é uma regra
que pertence inteiramente ao plugin daquele estado — o núcleo (CoreFaturamento,
em nucleo.py) não sabe nem precisa saber que ALIQUOTA existe. Ele só conhece o
contrato PluginFaturamento (nome + processar(fatura, resultado)) e a ordem das
categorias. Mudar uma regra fiscal de um estado não exigiu tocar em nucleo.py,
dominio.py, nem nos plugins de outros estados — isso é a essência do estilo
MicroKernel: o núcleo é estável, e cada extensão contém sua própria mudança.