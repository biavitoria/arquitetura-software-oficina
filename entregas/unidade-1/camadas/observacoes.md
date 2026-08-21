Condição alterada: em dominio.py, no método Consulta.cancelar(), troquei
"if self.status != 'agendada'" por "if self.status == 'cancelada'".

O que a saída revelou: antes, cancelar a consulta #1 (já realizada) não era
possível. Depois da mudança, a consulta #1 foi cancelada com sucesso (seção 5),
mas isso apagou o registro de que ela tinha sido realizada — no histórico da
paciente (seção 8), o status virou "cancelada" em vez de "realizada". A regra
que bloqueia cancelar algo já cancelado continuou funcionando (seção 6).

Responsabilidade arquitetural: a invariante de negócio "o que pode acontecer
com uma consulta" mora na entidade Consulta, em dominio.py — não em
servicos.py nem em apresentacao.py. Alterar essa regra num único lugar mudou
o comportamento em toda a aplicação (na ação de cancelar e no relatório de
histórico), sem precisar tocar em mais nenhum arquivo. Isso mostra que a
camada de domínio concentra as regras que todas as outras camadas obedecem.