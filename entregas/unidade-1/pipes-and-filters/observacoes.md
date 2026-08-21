Condição alterada: em main.py, na construção da Vaga, troquei
salario_maximo=18_000.0 por salario_maximo=23_000.0.

O que a saída revelou: antes, 3 candidatos eram aprovados (Ana Lima, Elena
Souza, Diego Faria) e Clara Mendes era reprovada por FiltroPorPretensaoSalarial
(pretensão R$22.000 > máximo R$18.000). Depois da mudança, Clara passou pelo
filtro e entrou na lista final com score 75% (3 de 4 habilidades compatíveis:
python, rest, docker), ficando empatada com Diego Faria. O total de aprovados
subiu de 3 para 4. As duas rejeições que continuaram (currículo id=3 sem nome,
Bruno Rocha com 1 ano de experiência) não foram afetadas, porque acontecem em
filtros anteriores (ValidadorDeCurriculo e FiltroPorExperienciaMinima), que não
dependem do critério salarial.

Responsabilidade arquitetural: cada filtro do pipeline (em filtros/testers.py)
julga um único critério, isolado dos demais — FiltroPorPretensaoSalarial não
sabe nada sobre experiência ou nome válido, e vice-versa. Mudar o parâmetro de
um filtro (o salario_maximo da Vaga) afetou apenas a decisão daquele filtro
específico, sem exigir nenhuma mudança em ValidadorDeCurriculo,
FiltroPorExperienciaMinima, NormalizadorDeCampos ou CalculadorDeScore. Isso
evidencia a responsabilidade única de cada filtro no estilo Pipes and Filters:
a mudança fica contida onde a regra vive.