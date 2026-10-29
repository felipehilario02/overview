# Histórico completo da conversa: `REMOVEFILTERS`, `TREATAS` e `Plan Total` no Power BI

> **Contexto:** investigação de uma medida DAX que deveria ignorar a segmentação de `Month`, continuar respeitando o filtro de `FY` e preservar as linhas da matriz, como `Revenue`, `Gross Margin`, `SG&A` e `SINC`.
>
> **Observação sobre as imagens:** as capturas de tela enviadas na conversa foram registradas no texto por meio da transcrição dos elementos relevantes, fórmulas, mensagens de erro e valores visíveis.

---

## 1. Pedido inicial

### Usuário

> Travar somente uma medida no PBI para não ser atualizada por segmentação de dados.

### Assistente

Foi explicado que uma medida pode ignorar uma segmentação usando `REMOVEFILTERS()` ou `ALL()`.

```DAX
Faturamento Total =
CALCULATE(
    SUM(Vendas[Valor]),
    REMOVEFILTERS(Vendas[Cliente])
)
```

Também foi mencionada a alternativa visual:

1. Selecionar a segmentação.
2. Acessar **Formato > Editar interações**.
3. No visual desejado, escolher **Nenhum**.

---

## 2. Diferença entre os relatórios BRL e USD

### Usuário

> Cara eu preciso de ajuda, eu tenho dois relatório iguais, as bases têm a mesma coluna, tratei da mesma maneira no Power Query, mesmas medidas, cada vírgula está igual, até o nome das medidas. A única diferença é que um está em dólar e outro em real, mas é literalmente tudo igual. O `REMOVEFILTERS` funcionou no BRL, porém o outro não funcionou, e é muita loucura, pois está tudo igual até a fórmula.

### Assistente

Foi levantada a hipótese de diferença em:

- contexto de filtro;
- coluna realmente usada no slicer;
- relacionamento ativo ou inativo;
- cardinalidade;
- direção de filtro;
- parâmetro de campo;
- grupo de cálculo;
- medida intermediária.

Testes sugeridos:

```DAX
Teste Sem Calendario =
CALCULATE(
    [Valor Total],
    REMOVEFILTERS('Calendario')
)
```

```DAX
Teste Totalmente Travado =
CALCULATE(
    [Valor Total],
    REMOVEFILTERS()
)
```

Medida de diagnóstico:

```DAX
Diagnostico Filtro =
VAR FiltroDireto =
    ISFILTERED('Calendario'[AnoMes])
VAR FiltroCruzado =
    ISCROSSFILTERED('Calendario'[AnoMes])
RETURN
    "Direto: "
        & IF(FiltroDireto, "SIM", "NÃO")
        & " | Cruzado: "
        & IF(FiltroCruzado, "SIM", "NÃO")
```

---

## 3. Fórmula que funcionava no relatório BRL

### Usuário

> Vou te enviar a fórmula que funciona e que não funciona, essa é a que funciona.

### Fórmula identificada na imagem

```DAX
Plan Total =
CALCULATE(
    SUM('DETAILS - DB'[Plan]),
    REMOVEFILTERS('Dataset'[Month])
)
```

### Assistente

Foi observado que:

- a segmentação de mês utilizava `'Dataset'[Month]`;
- a soma vinha de `'DETAILS - DB'[Plan]`;
- no relatório BRL, limpar `'Dataset'[Month]` era suficiente;
- os filtros de FY e moeda continuavam funcionando porque apenas `Month` era removido.

---

## 4. Fórmula que não funcionava no relatório USD

### Usuário

> Aqui é a que não funciona.

### Fórmula

```DAX
Plan Total =
CALCULATE(
    SUM('DETAILS - DB'[Plan]),
    REMOVEFILTERS('Dataset'[Month])
)
```

### Assistente

Como a fórmula era idêntica, foi sugerido testar a tabela `Dataset` inteira:

```DAX
Plan Total Teste =
CALCULATE(
    SUM('DETAILS - DB'[Plan]),
    REMOVEFILTERS('Dataset')
)
```

Outro teste sugerido:

```DAX
Plan Total Teste DB =
CALCULATE(
    SUM('DETAILS - DB'[Plan]),
    REMOVEFILTERS('DETAILS - DB')
)
```

Também foi sugerido testar a possível influência de `'Dataset'[List]`:

```DAX
Plan Total =
CALCULATE(
    SUM('DETAILS - DB'[Plan]),
    REMOVEFILTERS('Dataset'[Month]),
    REMOVEFILTERS('Dataset'[List])
)
```

---

## 5. `REMOVEFILTERS('Dataset')` também não resolveu

### Usuário

> Fiz pelo Dataset e ainda continua atualizando. E no BI que deu certo também está com todas as colunas selecionadas.

### Assistente

Foi concluído que o filtro não estava chegando somente por `Dataset`. Foi solicitado o teste:

```DAX
Plan Total Debug =
CALCULATE(
    SUM('DETAILS - DB'[Plan]),
    REMOVEFILTERS()
)
```

E também:

```DAX
Plan Total Debug DB =
CALCULATE(
    SUM('DETAILS - DB'[Plan]),
    REMOVEFILTERS('DETAILS - DB')
)
```

A hipótese era de caminho alternativo por tabelas como:

- `DEPARA - FY`;
- `DEPARA - RIF`;
- `BASE RIF`;
- `Dataset`;
- `DETAILS - DB`.

---

## 6. Comparação dos relacionamentos

### Usuário

> Esses são os relacionamentos do BI que deu certo.

### Assistente

Foi observado um relacionamento direto entre `Dataset` e `DETAILS - DB`, além de caminhos envolvendo:

- `DEPARA - FY`;
- `BASE RIF`;
- `DEPARA - RIF`;
- `BASE TAXA INSTALAÇÃO`.

Foi solicitado comparar:

1. colunas relacionadas;
2. relacionamento ativo ou inativo;
3. cardinalidade `1:*` ou `*:*`;
4. direção de filtro única ou ambas;
5. caminhos alternativos até `DETAILS - DB`.

Teste sugerido:

```DAX
Plan Total Debug =
CALCULATE(
    SUM('DETAILS - DB'[Plan]),
    REMOVEFILTERS('Dataset'),
    REMOVEFILTERS('DEPARA - FY'),
    REMOVEFILTERS('DETAILS - DB')
)
```

### Usuário

> Essa é a do BI que deu errado, lembrando que eu deixei as colunas relacionadas com o mesmo nome nas tabelas, em ambos os relatórios.

### Assistente

Foi explicado que nomes iguais não garantem o mesmo comportamento. Foram sugeridos testes removendo mês diretamente da fato:

```DAX
Plan Total =
CALCULATE(
    SUM('DETAILS - DB'[Plan]),
    REMOVEFILTERS('Dataset'),
    REMOVEFILTERS('DETAILS - DB'[Month])
)
```

E removendo também de `BASE RIF`:

```DAX
Teste Plan Sem Mes =
CALCULATE(
    SUM('DETAILS - DB'[Plan]),
    REMOVEFILTERS('Dataset'),
    REMOVEFILTERS('DETAILS - DB'[Month]),
    REMOVEFILTERS('BASE RIF'[Month])
)
```

---

## 7. Teste removendo `Dataset` e `DETAILS - DB[Month]`

### Usuário

> Não deu certo:

```DAX
Plan Total =
CALCULATE(
    SUM('DETAILS - DB'[Plan]),
    REMOVEFILTERS('Dataset'),
    REMOVEFILTERS('DETAILS - DB'[Month])
)
```

### Assistente

Foi reforçado o teste totalmente sem filtros:

```DAX
Plan Total Debug =
CALCULATE(
    SUM('DETAILS - DB'[Plan]),
    REMOVEFILTERS()
)
```

Também foi apresentada uma tentativa com `ALLEXCEPT`:

```DAX
Plan Total =
CALCULATE(
    SUM('DETAILS - DB'[Plan]),
    ALLEXCEPT(
        'DETAILS - DB',
        'DETAILS - DB'[Class],
        'DETAILS - DB'[FY],
        'DETAILS - DB'[Currency]
    ),
    REMOVEFILTERS('Dataset')
)
```

Sem moeda:

```DAX
Plan Total =
CALCULATE(
    SUM('DETAILS - DB'[Plan]),
    ALLEXCEPT(
        'DETAILS - DB',
        'DETAILS - DB'[Class],
        'DETAILS - DB'[FY]
    ),
    REMOVEFILTERS('Dataset')
)
```

Também foi mencionada a possibilidade de desligar temporariamente um relacionamento com `CROSSFILTER(..., NONE)`.

---

## 8. `REMOVEFILTERS()` funcionou, mas removeu o FY

### Usuário

> Deu certo, mas aí para de filtrar o FY. Eu não quero que o filtro de FY pare de funcionar.

### Assistente

Foi tentado remover somente colunas de mês:

```DAX
Plan Total =
CALCULATE(
    SUM('DETAILS - DB'[Plan]),
    REMOVEFILTERS('Dataset'[Month]),
    REMOVEFILTERS('DETAILS - DB'[Month]),
    REMOVEFILTERS('BASE RIF'[Month])
)
```

Também foi sugerido guardar e reaplicar FY:

```DAX
Plan Total =
VAR FYSelecionado =
    VALUES('DEPARA - FY'[FY])
RETURN
    CALCULATE(
        SUM('DETAILS - DB'[Plan]),
        REMOVEFILTERS('Dataset'[Month]),
        REMOVEFILTERS('DETAILS - DB'[Month]),
        REMOVEFILTERS('BASE RIF'[Month]),
        KEEPFILTERS(
            TREATAS(
                FYSelecionado,
                'DETAILS - DB'[FY]
            )
        )
    )
```

### Usuário

> Não funcionou. Se eu seleciono o mês, ainda muda o Plan Total.

---

## 9. Diferença entre cartão e matriz

### Usuário

Foi confirmado que esta medida funcionava:

```DAX
DEBUG Plan Fixo =
CALCULATE(
    SUM('DETAILS - DB'[Plan]),
    REMOVEFILTERS()
)
```

Mas havia uma diferença:

- no cartão, o valor não mudava conforme a segmentação;
- na matriz, a mesma medida apresentava comportamento diferente;
- `REMOVEFILTERS()` também removia FY.

### Assistente

Foi explicado que o cartão avalia a medida uma vez, enquanto a matriz avalia cada célula dentro do contexto da linha. Por isso foi proposta a captura e reaplicação de FY e `Class`:

```DAX
Plan Total =
VAR _FYSelecionado =
    VALUES('DEPARA - FY'[FY])

VAR _IndicadorAtual =
    VALUES('DETAILS - DB'[Class])

RETURN
    CALCULATE(
        SUM('DETAILS - DB'[Plan]),
        REMOVEFILTERS(),
        TREATAS(
            _FYSelecionado,
            'DETAILS - DB'[FY]
        ),
        TREATAS(
            _IndicadorAtual,
            'DETAILS - DB'[Class]
        )
    )
```

Versão com moeda:

```DAX
Plan Total =
VAR _FYSelecionado =
    VALUES('DEPARA - FY'[FY])

VAR _IndicadorAtual =
    VALUES('DETAILS - DB'[Class])

VAR _MoedaAtual =
    VALUES('DETAILS - DB'[Currency])

RETURN
    CALCULATE(
        SUM('DETAILS - DB'[Plan]),
        REMOVEFILTERS(),
        TREATAS(
            _FYSelecionado,
            'DETAILS - DB'[FY]
        ),
        TREATAS(
            _IndicadorAtual,
            'DETAILS - DB'[Class]
        ),
        TREATAS(
            _MoedaAtual,
            'DETAILS - DB'[Currency]
        )
    )
```

### Usuário

> Tira o Currency, que na Costa Rica não tem. É só em dólar.

### Assistente

Versão sem `Currency`:

```DAX
Plan Total =
VAR FYSelecionado =
    VALUES('DETAILS - DB'[FY])

VAR IndicadorAtual =
    VALUES('DETAILS - DB'[Class])

RETURN
    CALCULATE(
        SUM('DETAILS - DB'[Plan]),
        REMOVEFILTERS(),
        TREATAS(
            FYSelecionado,
            'DETAILS - DB'[FY]
        ),
        TREATAS(
            IndicadorAtual,
            'DETAILS - DB'[Class]
        )
    )
```

### Usuário

> Continua não funcionando.

---

## 10. Tentativas preservando FY e indicador

Foram propostas variações usando:

```DAX
CALCULATETABLE(
    VALUES('DEPARA - FY'[FY])
)
```

ou:

```DAX
CALCULATETABLE(
    VALUES('Dataset'[List FY])
)
```

seguido de:

```DAX
TREATAS(
    FYSelecionado,
    'DETAILS - DB'[FY]
)
```

Também foi discutida possível diferença de formato:

- `26`;
- `FY26`;
- `2026`;
- texto versus número inteiro.

### Usuário

> Continua não funcionando, eu estou selecionando o mês e está mudando o valor.

### Assistente

Foi novamente sugerida a captura do FY e do indicador antes de `REMOVEFILTERS()`, com atenção ao campo exato do slicer FY.

### Usuário

> Não resolveu o mês, burro.

### Assistente

Foi reconhecida a interpretação incorreta. A ideia continuou sendo remover todo o contexto e reaplicar FY e indicador.

---

## 11. Pergunta sobre diferenças nos relacionamentos

### Usuário

> Tem alguma diferença nos relacionamentos das tabelas?

### Assistente

Foi explicado que as imagens sugeriam diferenças possíveis, mas não permitiam confirmar todos os detalhes. Os itens recomendados para comparação foram:

- coluna de origem e destino;
- cardinalidade;
- direção do filtro cruzado;
- relacionamento ativo;
- duplicidades no lado `1`;
- caminhos alternativos entre `Dataset`, `BASE RIF`, `DEPARA - FY` e `DETAILS - DB`.

---

## 12. Medida com FY, Class e Currency

### Usuário

Foi testada a seguinte medida:

```DAX
Plan Total =
VAR FYAtual =
    VALUES('DETAILS - DB'[FY])
VAR IndicadorAtual =
    VALUES('DETAILS - DB'[Class])
VAR MoedaAtual =
    VALUES('DETAILS - DB'[Currency])
RETURN
    CALCULATE(
        SUM('DETAILS - DB'[Plan]),
        REMOVEFILTERS(),
        TREATAS(
            FYAtual,
            'DETAILS - DB'[FY]
        ),
        TREATAS(
            IndicadorAtual,
            'DETAILS - DB'[Class]
        ),
        TREATAS(
            MoedaAtual,
            'DETAILS - DB'[Currency]
        )
    )
```

> Ao mudar a seleção Month, continua alterando a matriz.

### Assistente

Foi identificado que FY, indicador e moeda estavam sendo capturados da própria fato, já contaminada por `Month`. Foi sugerido capturar FY em `DEPARA - FY`.

Exemplo:

```DAX
Plan Total =
VAR FYSelecionado =
    SELECTEDVALUE('DEPARA - FY'[FY])

VAR IndicadorSelecionado =
    SELECTEDVALUE('DETAILS - DB'[Class])

RETURN
    CALCULATE(
        SUM('DETAILS - DB'[Plan]),
        REMOVEFILTERS(),
        TREATAS(
            { FYSelecionado },
            'DETAILS - DB'[FY]
        ),
        TREATAS(
            { IndicadorSelecionado },
            'DETAILS - DB'[Class]
        )
    )
```

Também foi levantada a possibilidade de o relacionamento utilizar `List` e `Year`, em vez de `[FY]`.

---

## 13. Confirmação dos campos usados nos slicers

### Usuário

> É o Month utilizado no Dataset que está na segmentação.

### Assistente

Foi explicada a estrutura de `Dataset`, em que cada mês também determina valores de `List` e `List FY`, por exemplo:

- January → `List = 1`, `List FY = 4`;
- September → `List = 9`, `List FY = 12`;
- October → `List = 10`, `List FY = 1`;
- November → `List = 11`, `List FY = 2`;
- December → `List = 12`, `List FY = 3`.

Foi sugerida a medida:

```DAX
Plan Total =
VAR FYSelecionado =
    CALCULATETABLE(
        VALUES('DEPARA - FY'[FY]),
        REMOVEFILTERS('Dataset')
    )

VAR IndicadorAtual =
    VALUES('DETAILS - DB'[Class])

RETURN
    CALCULATE(
        SUM('DETAILS - DB'[Plan]),
        REMOVEFILTERS(),
        TREATAS(
            FYSelecionado,
            'DETAILS - DB'[FY]
        ),
        TREATAS(
            IndicadorAtual,
            'DETAILS - DB'[Class]
        )
    )
```

### Usuário

> O FY está numa tabela usada chamada `DEPARA - FY`.

### Assistente

Versão proposta:

```DAX
Plan Total =
VAR FYSelecionado =
    CALCULATETABLE(
        VALUES('DEPARA - FY'[FY]),
        REMOVEFILTERS('Dataset')
    )

VAR IndicadorAtual =
    CALCULATETABLE(
        VALUES('DETAILS - DB'[Class]),
        REMOVEFILTERS('Dataset')
    )

RETURN
    CALCULATE(
        SUM('DETAILS - DB'[Plan]),
        REMOVEFILTERS(),
        TREATAS(
            FYSelecionado,
            'DETAILS - DB'[FY]
        ),
        TREATAS(
            IndicadorAtual,
            'DETAILS - DB'[Class]
        )
    )
```

### Usuário

> Continua mudando, ainda está do mesmo jeito.

---

## 14. Medida que finalmente travou o mês

### Assistente

Foi proposta uma captura isolada, removendo influências das tabelas relacionadas:

```DAX
Plan Total =
VAR FYSelecionado =
    CALCULATETABLE(
        VALUES('DEPARA - FY'[FY]),
        REMOVEFILTERS('Dataset'),
        REMOVEFILTERS('DETAILS - DB'),
        REMOVEFILTERS('BASE RIF')
    )

VAR IndicadorAtual =
    CALCULATETABLE(
        VALUES('DETAILS - DB'[Class]),
        REMOVEFILTERS('Dataset'),
        REMOVEFILTERS('BASE RIF'),
        REMOVEFILTERS('DEPARA - FY')
    )

RETURN
    CALCULATE(
        SUM('DETAILS - DB'[Plan]),
        REMOVEFILTERS(),
        TREATAS(
            FYSelecionado,
            'DETAILS - DB'[FY]
        ),
        TREATAS(
            IndicadorAtual,
            'DETAILS - DB'[Class]
        )
    )
```

### Usuário

> Deu certo agora, mas por que no outro relatório de real funciona e nesse não, sendo que é tudo igual?

### Assistente

Foi explicado que a diferença provavelmente estava no contexto interno do modelo USD, como:

- direção de filtro cruzado;
- relacionamento muitos para muitos;
- relacionamento ativo ou inativo;
- chave diferente;
- duplicidade no lado `1`;
- caminho adicional por `BASE RIF` ou `DEPARA - FY`.

A medida robusta funcionou porque removeu todo o contexto e reconstruiu apenas FY e indicador.

---

## 15. Possível influência do tipo de dado

### Usuário

> Será que tem a ver com o tipo de dado?

### Assistente

Foi explicado que o tipo de dado pode influenciar especialmente relacionamentos e `TREATAS`. Foram citadas as colunas:

| Coluna | Tipo esperado |
|---|---|
| `'Dataset'[Month]` | Texto |
| `'Dataset'[List]` | Número inteiro |
| `'Dataset'[List FY]` | Número inteiro |
| `'DEPARA - FY'[FY]` | Mesmo tipo do destino |
| `'DETAILS - DB'[FY]` | Mesmo tipo da origem |

Exemplo de incompatibilidade:

```text
DEPARA - FY[FY] = número 26
DETAILS - DB[FY] = texto "26"
```

Medida de teste sugerida:

```DAX
Teste Compatibilidade FY =
VAR FYDepara =
    SELECTEDVALUE('DEPARA - FY'[FY])

VAR EncontroCorrespondencia =
    CALCULATE(
        COUNTROWS('DETAILS - DB'),
        TREATAS(
            { FYDepara },
            'DETAILS - DB'[FY]
        )
    )

RETURN
    EncontroCorrespondencia
```

---

## 16. O que é `TREATAS`

### Usuário

> Tretas serve para quê?

### Assistente

Foi esclarecido que o nome correto é `TREATAS`. A função pega valores de uma coluna e aplica esses valores como filtro em outra coluna.

Exemplo:

```DAX
TREATAS(
    VALUES('DEPARA - FY'[FY]),
    'DETAILS - DB'[FY]
)
```

Significado:

> Pegue os FY selecionados em `'DEPARA - FY'[FY]` e aplique-os como filtro em `'DETAILS - DB'[FY]`.

Foi descrito como um relacionamento virtual usado somente durante o cálculo.

---

## 17. Por que usar a coluna `Class`

### Usuário

> Mas por que a coluna Class?

### Assistente

Foi explicado que `Class` mantém as linhas da matriz separadas:

- Revenue;
- Gross Margin;
- SG&A;
- SINC.

Como `REMOVEFILTERS()` remove todos os filtros, também apaga o filtro da linha atual. Portanto, é necessário guardar e reaplicar `Class`:

```DAX
VAR IndicadorAtual =
    CALCULATETABLE(
        VALUES('DETAILS - DB'[Class]),
        REMOVEFILTERS('Dataset'),
        REMOVEFILTERS('BASE RIF'),
        REMOVEFILTERS('DEPARA - FY')
    )
```

```DAX
TREATAS(
    IndicadorAtual,
    'DETAILS - DB'[Class]
)
```

Sem isso, todas as linhas poderiam mostrar o mesmo total geral.

---

## 18. Explicação linha por linha da solução

### Usuário

> Me explique cada linha dessa solução, por favor, pois não entendi direito ainda.

### Medida explicada

```DAX
Plan Total =
VAR FYSelecionado =
    CALCULATETABLE(
        VALUES('DEPARA - FY'[FY]),
        REMOVEFILTERS('Dataset'),
        REMOVEFILTERS('DETAILS - DB'),
        REMOVEFILTERS('BASE RIF')
    )

VAR IndicadorAtual =
    CALCULATETABLE(
        VALUES('DETAILS - DB'[Class]),
        REMOVEFILTERS('Dataset'),
        REMOVEFILTERS('BASE RIF'),
        REMOVEFILTERS('DEPARA - FY')
    )

RETURN
    CALCULATE(
        SUM('DETAILS - DB'[Plan]),
        REMOVEFILTERS(),
        TREATAS(
            FYSelecionado,
            'DETAILS - DB'[FY]
        ),
        TREATAS(
            IndicadorAtual,
            'DETAILS - DB'[Class]
        )
    )
```

Resumo da explicação:

1. `FYSelecionado` guarda o FY da segmentação, tentando eliminar a contaminação de `Dataset`, `DETAILS - DB` e `BASE RIF`.
2. `IndicadorAtual` guarda a `Class` da linha da matriz, como Revenue ou SG&A.
3. `REMOVEFILTERS()` remove todo o contexto, inclusive Month, FY e Class.
4. O primeiro `TREATAS` reaplica FY.
5. O segundo `TREATAS` reaplica Class.
6. Month não é reaplicado, portanto deveria ser ignorado.

---

## 19. Medida do RIF

### Usuário

> No total plan do RIF funcionou de primeira a minha fórmula.

### Fórmula mostrada na imagem

```DAX
RIF TOTAL PLAN =
CALCULATE(
    SUM('BASE RIF'[Data]),
    'BASE RIF'[Status] = "Plan",
    REMOVEFILTERS('Dataset'[Month])
)
```

### Assistente

Foi explicado que a medida parecia funcionar porque:

- o filtro de Month provavelmente chegava à `BASE RIF` por um caminho mais simples;
- `'BASE RIF'[Status] = "Plan"` fixava explicitamente o tipo de valor;
- apenas `Dataset[Month]` era removido;
- FY e o indicador da linha permaneciam no contexto.

---

## 20. Correção: a medida do RIF também não estava correta

### Usuário

> Fui fazer os cálculos e, na verdade, não deu certo a medida do Plan Total. Repare do jeito que tinha que ficar na imagem. Logo após vou enviar o valor que está ficando. De alguma forma ele está impactando no valor das outras colunas/linhas.

### Assistente

Foi pedido comparar:

- resultado esperado;
- resultado atual;
- FY selecionado;
- meses selecionados;
- `Act-YTD`;
- `Plan-YTD`;
- `Plan-Total`;
- demais colunas dependentes.

---

## 21. Resultado esperado

### Usuário

> Esse é o jeito que tem que ficar. Pense: o Plan YTD pega o acumulado do ano até agora e o Plan Total é o total do ano. Ele tem que ficar fixo porque ele é o total. Os dois números estão iguais, Plan YTD e Plan Total, porque já foi o ano inteiro. Mas, caso não fosse um ano completo, ele teria que ficar fixo e a única coluna que mudasse tinha que ser Plan YTD.

### Valores visíveis na imagem

| Linha | Plan-YTD | Plan-Total |
|---|---:|---:|
| 1 | 18.527 | 18.527 |
| 2 | 10.917 | 10.917 |
| 3 | 4.375 | 4.375 |
| 4 | 6.542 | 6.542 |

Com todos os meses do FY selecionados, `Plan-YTD` e `Plan-Total` são iguais. Ao reduzir os meses:

- `Plan-YTD` deve mudar;
- `Plan-Total` deve permanecer fixo;
- FY deve continuar funcionando;
- cada linha deve manter seu próprio total.

---

## 22. Resultado incorreto da medida robusta

### Usuário

Medida utilizada:

```DAX
Plan Total =
VAR FYSelecionado =
    CALCULATETABLE(
        VALUES('DEPARA - FY'[FY]),
        REMOVEFILTERS('Dataset'),
        REMOVEFILTERS('DETAILS - DB'),
        REMOVEFILTERS('BASE RIF')
    )

VAR IndicadorAtual =
    CALCULATETABLE(
        VALUES('DETAILS - DB'[Class]),
        REMOVEFILTERS('Dataset'),
        REMOVEFILTERS('BASE RIF'),
        REMOVEFILTERS('DEPARA - FY')
    )

RETURN
    CALCULATE(
        SUM('DETAILS - DB'[Plan]),
        REMOVEFILTERS(),
        TREATAS(
            FYSelecionado,
            'DETAILS - DB'[FY]
        ),
        TREATAS(
            IndicadorAtual,
            'DETAILS - DB'[Class]
        )
    )
```

### Valores visíveis na imagem

| Plan-YTD | Plan-Total incorreto |
|---:|---:|
| 6.542 | 18.254 |
| 10.917 | 31.081 |
| 4.375 | 12.829 |
| 18.527 | 55.893 |

### Assistente

Foi concluído que:

- `Class` parecia estar sendo preservada, pois as linhas tinham valores diferentes;
- FY aparentemente não estava sendo aplicado corretamente;
- `Plan-Total` parecia somar a mesma `Class` em vários anos fiscais.

Foi proposta esta fórmula:

```DAX
Plan Total =
VAR FYSelecionado =
    SELECTEDVALUE('DEPARA - FY'[FY])

VAR IndicadorAtual =
    CALCULATETABLE(
        SELECTEDVALUE('DETAILS - DB'[Class]),
        REMOVEFILTERS('Dataset'),
        REMOVEFILTERS('BASE RIF'),
        REMOVEFILTERS('DEPARA - FY')
    )

RETURN
    CALCULATE(
        SUM('DETAILS - DB'[Plan]),
        REMOVEFILTERS(),
        FILTER(
            ALL('DETAILS - DB'[FY]),
            'DETAILS - DB'[FY] = FYSelecionado
        ),
        TREATAS(
            { IndicadorAtual },
            'DETAILS - DB'[Class]
        )
    )
```

---

## 23. Erro do `CALCULATETABLE`

### Usuário

Foi mostrada a mensagem:

> A função CALCULATETABLE esperava uma expressão de tabela para o argumento '1', mas foi usada uma expressão de cadeia de caracteres ou numérica.

O erro estava nesta construção:

```DAX
CALCULATETABLE(
    SELECTEDVALUE('DETAILS - DB'[Class]),
    ...
)
```

### Assistente

Foi explicado que:

- `SELECTEDVALUE` retorna um valor escalar;
- `CALCULATETABLE` exige uma tabela;
- a correção era trocar `CALCULATETABLE` por `CALCULATE`.

Fórmula corrigida:

```DAX
Plan Total =
VAR FYSelecionado =
    SELECTEDVALUE('DEPARA - FY'[FY])

VAR IndicadorAtual =
    CALCULATE(
        SELECTEDVALUE('DETAILS - DB'[Class]),
        REMOVEFILTERS('Dataset'),
        REMOVEFILTERS('BASE RIF'),
        REMOVEFILTERS('DEPARA - FY')
    )

RETURN
    CALCULATE(
        SUM('DETAILS - DB'[Plan]),
        REMOVEFILTERS(),
        FILTER(
            ALL('DETAILS - DB'[FY]),
            'DETAILS - DB'[FY] = FYSelecionado
        ),
        TREATAS(
            { IndicadorAtual },
            'DETAILS - DB'[Class]
        )
    )
```

---

## 24. A medida voltou a mudar com Month

### Usuário

> Dessa maneira voltou no mesmo erro de antes. Quando eu seleciono o mês, ele muda o Plan Total.

Fórmula testada:

```DAX
Plan Total =
VAR FYSelecionado =
    SELECTEDVALUE('DEPARA - FY'[FY])
VAR IndicadorAtual =
    CALCULATE(
        SELECTEDVALUE('DETAILS - DB'[Class]),
        REMOVEFILTERS('Dataset'),
        REMOVEFILTERS('BASE RIF'),
        REMOVEFILTERS('DEPARA - FY')
    )
RETURN
    CALCULATE(
        SUM('DETAILS - DB'[Plan]),
        REMOVEFILTERS(),
        FILTER(
            ALL('DETAILS - DB'[FY]),
            'DETAILS - DB'[FY] = FYSelecionado
        ),
        TREATAS(
            { IndicadorAtual },
            'DETAILS - DB'[Class]
        )
    )
```

### Assistente

Foi sugerido usar `FILTERS` para capturar filtros diretos:

```DAX
Plan Total =
VAR FYSelecionado =
    FILTERS('DEPARA - FY'[FY])

VAR IndicadorAtual =
    FILTERS('DETAILS - DB'[Class])

RETURN
    CALCULATE(
        SUM('DETAILS - DB'[Plan]),
        REMOVEFILTERS(),
        TREATAS(
            FYSelecionado,
            'DETAILS - DB'[FY]
        ),
        TREATAS(
            IndicadorAtual,
            'DETAILS - DB'[Class]
        )
    )
```

### Usuário

> Continua do mesmo jeito.

### Assistente

A conclusão final dessa etapa foi:

- não adiantava continuar apenas alternando `VALUES`, `SELECTEDVALUE` e `FILTERS`;
- `Class` aparentemente estava sendo mantida;
- a aplicação do FY continuava inconsistente;
- era necessário confirmar os valores, tipos e as colunas exatas do relacionamento entre `DEPARA - FY` e `DETAILS - DB`.

Medidas de diagnóstico sugeridas:

```DAX
Debug FY Segmentação =
CONCATENATEX(
    VALUES('DEPARA - FY'[FY]),
    'DEPARA - FY'[FY],
    " | "
)
```

```DAX
Debug FY Details =
CONCATENATEX(
    ALL('DETAILS - DB'[FY]),
    'DETAILS - DB'[FY],
    " | "
)
```

Também foi sugerido usar as colunas reais do relacionamento, por exemplo:

```DAX
Plan Total =
VAR ChaveFYSelecionada =
    FILTERS('DEPARA - FY'[List])

VAR IndicadorAtual =
    FILTERS('DETAILS - DB'[Class])

RETURN
    CALCULATE(
        SUM('DETAILS - DB'[Plan]),
        REMOVEFILTERS(),
        TREATAS(
            ChaveFYSelecionada,
            'DETAILS - DB'[Year]
        ),
        TREATAS(
            IndicadorAtual,
            'DETAILS - DB'[Class]
        )
    )
```

> Nessa versão, `[List]` e `[Year]` eram apenas exemplos. A orientação foi substituir pelas duas colunas efetivamente utilizadas no relacionamento.

---

# Conclusões consolidadas até o fim da conversa

1. A medida simples abaixo funciona no relatório BRL, mas não se comportou da mesma forma no USD/Costa Rica:

```DAX
Plan Total =
CALCULATE(
    SUM('DETAILS - DB'[Plan]),
    REMOVEFILTERS('Dataset'[Month])
)
```

2. Esta medida comprovou que remover todo o contexto impede o Month de alterar o cálculo:

```DAX
DEBUG Plan Fixo =
CALCULATE(
    SUM('DETAILS - DB'[Plan]),
    REMOVEFILTERS()
)
```

3. Porém, `REMOVEFILTERS()` também remove FY, Class e outros filtros necessários.

4. O uso de `TREATAS` foi uma tentativa de reconstruir somente:

- o FY selecionado;
- a Class da linha da matriz.

5. A medida chegou a travar o Month, mas passou a somar valores de vários FYs, produzindo totais como `55.893`, `31.081`, `12.829` e `18.254`, em vez dos totais anuais esperados `18.527`, `10.917`, `4.375` e `6.542`.

6. O ponto ainda não resolvido ao final da conversa foi a reaplicação correta do FY. As principais suspeitas restantes eram:

- o slicer usa `'DEPARA - FY'[FY]`, mas o relacionamento pode usar outra coluna;
- os valores de FY podem estar em formatos diferentes, como `26`, `FY26` ou `2026`;
- o tipo pode ser texto em uma tabela e número na outra;
- existe caminho de filtro alternativo ou bidirecional;
- existe cardinalidade ou duplicidade diferente entre os relatórios;
- a `Class` da linha pode vir de uma coluna diferente da que está sendo reaplicada.

7. O comportamento funcional esperado ficou definido assim:

| Alteração | Plan-YTD | Plan-Total |
|---|---|---|
| Selecionar menos meses | Deve mudar | Não deve mudar |
| Trocar FY | Deve mudar | Deve mudar |
| Trocar linha/indicador | Deve respeitar a linha | Deve respeitar a linha |
| Todos os meses selecionados | Pode igualar Plan-Total | Total anual completo |

---

# Fórmulas principais reunidas

## Fórmula original

```DAX
Plan Total =
CALCULATE(
    SUM('DETAILS - DB'[Plan]),
    REMOVEFILTERS('Dataset'[Month])
)
```

## Debug que ignora todos os filtros

```DAX
DEBUG Plan Fixo =
CALCULATE(
    SUM('DETAILS - DB'[Plan]),
    REMOVEFILTERS()
)
```

## Medida robusta que chegou a travar Month, mas somou vários FYs

```DAX
Plan Total =
VAR FYSelecionado =
    CALCULATETABLE(
        VALUES('DEPARA - FY'[FY]),
        REMOVEFILTERS('Dataset'),
        REMOVEFILTERS('DETAILS - DB'),
        REMOVEFILTERS('BASE RIF')
    )

VAR IndicadorAtual =
    CALCULATETABLE(
        VALUES('DETAILS - DB'[Class]),
        REMOVEFILTERS('Dataset'),
        REMOVEFILTERS('BASE RIF'),
        REMOVEFILTERS('DEPARA - FY')
    )

RETURN
    CALCULATE(
        SUM('DETAILS - DB'[Plan]),
        REMOVEFILTERS(),
        TREATAS(
            FYSelecionado,
            'DETAILS - DB'[FY]
        ),
        TREATAS(
            IndicadorAtual,
            'DETAILS - DB'[Class]
        )
    )
```

## Fórmula do RIF testada

```DAX
RIF TOTAL PLAN =
CALCULATE(
    SUM('BASE RIF'[Data]),
    'BASE RIF'[Status] = "Plan",
    REMOVEFILTERS('Dataset'[Month])
)
```

## Diagnóstico dos valores de FY

```DAX
Debug FY Segmentação =
CONCATENATEX(
    VALUES('DEPARA - FY'[FY]),
    'DEPARA - FY'[FY],
    " | "
)
```

```DAX
Debug FY Details =
CONCATENATEX(
    ALL('DETAILS - DB'[FY]),
    'DETAILS - DB'[FY],
    " | "
)
```

---

*Documento gerado a partir da conversa sobre Power BI, DAX, segmentações, relacionamentos, `REMOVEFILTERS`, `TREATAS`, FY, Month, Class, Plan-YTD e Plan-Total.*
