# Impactos do Choque Geopolítico de 2026 nos Preços de Combustíveis no Brasil

MVP de Engenharia de Dados desenvolvido como parte da Pós-Graduação em Ciência de Dados e Analytics da PUC-Rio.

## 1. Contexto de Negócios e Perguntas

### Contexto de Negócios

O mercado brasileiro de combustíveis está sujeito tanto a fatores internos quanto a movimentos do mercado internacional de petróleo. Alterações no preço do Brent podem afetar os preços de referência dos derivados e, posteriormente, chegar aos preços pagos pelo consumidor brasileiro. Esse processo, no entanto, não ocorre necessariamente de forma imediata ou uniforme entre diferentes combustíveis e regiões do país.

Em 28 de fevereiro de 2026, ataques militares dos Estados Unidos e de Israel contra o Irã, seguidos por ataques iranianos na região, marcaram uma escalada do conflito no Oriente Médio. Nos dias e semanas seguintes, o mercado internacional de petróleo registrou forte elevação dos preços em um contexto de redução do tráfego pelo Estreito de Ormuz, interrupções na produção regional e aumento das incertezas sobre a oferta de petróleo. 
A partir desse contexto, o projeto tem como objetivo analisar como o choque observado no mercado internacional de petróleo se relacionou com o comportamento dos preços da gasolina e do diesel no Brasil, considerando tanto os preços de referência dos derivados quanto os preços efetivamente observados ao consumidor.

Para isso, foram integradas três fontes de dados: a série histórica do preço internacional do petróleo Brent, disponibilizada pela U.S. Energy Information Administration (EIA); os dados de Preço de Paridade de Importação (PPI) de gasolina e diesel; e os dados de preços de combustíveis ao consumidor disponibilizados pela Agência Nacional do Petróleo, Gás Natural e Biocombustíveis (ANP).

A análise foi estruturada em uma arquitetura de dados com camadas Raw, Bronze, Silver e Gold. As etapas envolveram ingestão dos arquivos originais, tratamento e validação da qualidade dos dados, padronização das séries temporais, criação de agregações semanais e integração das diferentes fontes para análise.

O dia 28 de fevereiro de 2026 foi utilizado como referência para comparar o comportamento das séries antes e depois da escalada do conflito. Além da comparação entre os períodos, foram analisadas correlações e defasagens temporais entre Brent, PPI e preços ao consumidor, assim como diferenças entre combustíveis, estados e regiões brasileiras.


### Perguntas do projeto

1. Como o preço internacional do petróleo se comportou antes e depois da escalada do conflito no Oriente Médio em 2026?
2. Qual foi o comportamento dos preços de paridade de importação de gasolina e diesel durante o período?
3. As oscilações internacionais foram repassadas imediatamente aos preços dos combustíveis no Brasil?
4. Qual combustível apresentou maior sensibilidade às oscilações internacionais: gasolina ou diesel?
5. Existe defasagem temporal entre movimentos do Brent/PPI e alterações nos preços ao consumidor brasileiro?
6. O comportamento foi diferente entre estados e regiões brasileiras?
7. Houve alteração relevante no volume comercializado de gasolina, diesel e etanol após o choque?
8. O etanol apresentou comportamento diferente dos combustíveis mais diretamente relacionados ao petróleo durante o período?

### Referências do contexto
- [ONU - pronunciamento do Secretário-Geral ao Conselho de Segurança em 28/02/2026](https://www.un.org/sg/en/content/sg/statements/2026-02-28/secretary-generals-remarks-the-security-council-meeting-the-situation-the-middle-east-delivered)
- [EIA - análise dos preços do petróleo no primeiro trimestre de 2026](https://www.eia.gov/todayinenergy/detail.php?id=67424)
- [EIA - Weekly Europe Brent Spot Price FOB](https://www.eia.gov/dnav/pet/hist/rbrtew.htm)

## 2. Carga dos Dados

O projeto utiliza três conjuntos de dados principais, escolhidos para representar diferentes etapas da formação dos preços dos combustíveis: o preço internacional do petróleo Brent, os preços de paridade de importação (PPI) e os preços praticados ao consumidor brasileiro.

Os arquivos originais foram armazenados em um Volume do Databricks, utilizado como camada Raw do projeto. A partir dessa camada, os dados foram lidos e persistidos em tabelas Delta na camada Bronze, preservando os dados de origem e adicionando, quando necessário, informações de rastreabilidade da ingestão.

### 2.1 Preço internacional do petróleo Brent

A série histórica do Brent foi obtida a partir da U.S. Energy Information Administration (EIA). A base contém preços históricos do petróleo Brent e foi utilizada como referência para acompanhar o comportamento do mercado internacional.

A ingestão resultou em 9.973 registros, abrangendo o período de 20/05/1987 a 09/09/2026.

Os dados originais foram armazenados no diretório:

`/Volumes/workspace/raw/dados_raw/brent/`

Após a leitura e ingestão, os dados foram persistidos na camada Bronze para utilização nas etapas posteriores do pipeline.

**Tabela Bronze:** `workspace.bronze.brent_raw`

**Fonte:** [U.S. Energy Information Administration (EIA)](https://www.eia.gov/dnav/pet/hist/rbrtew.htm)

### 2.2 Preço de Paridade de Importação (PPI)

Os dados de Preço de Paridade de Importação foram obtidos a partir de arquivo disponibilizado pela Agência Nacional do Petróleo, Gás Natural e Biocombustíveis (ANP).

O arquivo contém séries semanais de PPI para gasolina e diesel, apresentadas para diferentes localidades. A base utilizada possui 409 semanas de observações na origem.

O arquivo original em formato Excel foi armazenado no diretório:

`/Volumes/workspace/raw/dados_raw/ppi/`

Durante a ingestão, as estruturas referentes à gasolina e ao diesel foram identificadas e tratadas separadamente antes da persistência na camada Bronze.

**Tabelas Bronze:**
- `workspace.bronze.ppi_gasolina_raw`
- `workspace.bronze.ppi_diesel_raw`

### 2.3 Preços de combustíveis ao consumidor

Os dados de preços ao consumidor também foram obtidos a partir da ANP. Essa base contém informações coletadas em postos revendedores de combustíveis, incluindo atributos como produto, data da coleta, município, estado, região, revenda, bandeira e valor de venda.

Foram utilizados 40 arquivos, que totalizaram 1.374.816 registros na ingestão.

Os arquivos originais foram armazenados no diretório:

`/Volumes/workspace/raw/dados_raw/anp_precos/`

Durante a ingestão, os arquivos foram consolidados em uma única estrutura e receberam informações adicionais de rastreabilidade, como arquivo de origem, data de ingestão e fonte. O resultado foi persistido na camada Bronze.

**Tabela Bronze:** `workspace.bronze.precos_anp_raw`

**Fonte:** [Agência Nacional do Petróleo, Gás Natural e Biocombustíveis (ANP)](https://www.gov.br/anp/pt-br)

### 2.4 Organização da camada Raw

Os arquivos de origem foram organizados no Volume seguindo a estrutura:

```text
/Volumes/workspace/raw/dados_raw/
├── brent/
├── ppi/
└── anp_precos/


## 3. Modelagem e Catálogo

A organização dos dados foi estruturada no Databricks utilizando o Unity Catalog e uma arquitetura em camadas Raw, Bronze, Silver e Gold. A separação das camadas permite distinguir os arquivos originais, os dados ingeridos, os dados tratados e os conjuntos preparados especificamente para análise.

### 3.1 Arquitetura de dados

A arquitetura adotada foi organizada da seguinte forma:

- **Raw:** armazenamento dos arquivos originais no Databricks Volume, separados por fonte.
- **Bronze:** persistência dos dados ingeridos, com mínima transformação e preservação das informações provenientes das fontes.
- **Silver:** tratamento, padronização, conversão de tipos, remoção de duplicidades e reorganização dos dados para análise.
- **Gold:** agregações semanais, integração entre as fontes e geração das tabelas utilizadas nas análises finais.

A estrutura implementada no catálogo foi:

```text
workspace
│
├── raw
│   └── dados_raw (Volume)
│
├── bronze
│   ├── brent_raw
│   ├── ppi_diesel_raw
│   ├── ppi_gasolina_raw
│   └── precos_anp_raw
│
├── silver
│   ├── brent
│   ├── ppi
│   └── precos_anp
│
└── gold
    ├── anp_semanal
    ├── base_integrada
    ├── brent_semanal
    ├── correlacao_brent_ppi
    ├── correlacoes_anp
    └── ppi_semanal
```

Essa organização possibilita acompanhar a evolução dos dados ao longo do pipeline e manter separadas as tabelas de ingestão, tratamento e análise.

### 3.2 Tabelas Silver

Na camada Silver foram consolidadas três tabelas principais:

- `workspace.silver.brent`: série histórica do Brent tratada e padronizada;
- `workspace.silver.ppi`: consolidação dos dados de gasolina e diesel em uma estrutura única;
- `workspace.silver.precos_anp`: preços ao consumidor tratados, com padronização dos campos utilizados na análise.

No tratamento do PPI, os dados originalmente distribuídos em diferentes colunas por localidade foram reorganizados em formato longo, permitindo identificar explicitamente data, produto, localidade, preço e variação semanal.

Nos dados de preços ao consumidor, o valor de venda foi convertido para formato numérico e as duplicidades exatas da fonte foram removidas.

### 3.3 Tabelas Gold

A camada Gold concentra os dados preparados para as análises do projeto.

As séries foram agregadas em frequência semanal para permitir a comparação entre fontes com granularidades originalmente diferentes. Foram criadas tabelas semanais para Brent, PPI e preços ANP, além de uma base integrada.

Também foram persistidos os resultados das análises de correlação entre Brent e PPI e das correlações envolvendo os preços ao consumidor.

Essa modelagem permitiu analisar tanto movimentos simultâneos quanto possíveis efeitos com defasagem temporal entre as séries.


## 4. Pipeline de Dados

O pipeline foi desenvolvido em notebooks no Databricks utilizando principalmente Python, PySpark e SQL.

Os notebooks foram separados por responsabilidade, acompanhando o fluxo dos dados desde a preparação do ambiente até a análise final:

1. **Setup do ambiente:** criação dos schemas `raw`, `bronze`, `silver` e `gold`, além do Volume utilizado para armazenamento dos arquivos originais.
2. **Ingestão do Brent:** leitura da série histórica e persistência na camada Bronze.
3. **Ingestão do PPI:** leitura do arquivo Excel, identificação das estruturas de gasolina e diesel, padronização inicial e persistência das duas tabelas Bronze.
4. **Ingestão dos preços ANP:** leitura e consolidação dos arquivos de preços ao consumidor, com inclusão de informações de rastreabilidade.
5. **Profiling e transformação Bronze → Silver:** avaliação da qualidade, conversão de tipos, tratamento das estruturas e remoção de duplicidades.
6. **Modelagem Silver → Gold:** criação das séries semanais, integração das fontes e cálculo das correlações e defasagens.
7. **Análise dos resultados:** comparação dos períodos anterior e posterior ao choque, análise dos combustíveis e avaliação das diferenças regionais.

O fluxo geral pode ser representado da seguinte forma:

```text
Arquivos originais
       │
       ▼
      Raw
       │
       ▼
    Bronze
       │
       ▼
    Silver
       │
       ▼
      Gold
       │
       ▼
Análises e resultados
```

A divisão em notebooks e camadas facilita a compreensão do fluxo e permite executar separadamente as etapas de ingestão, transformação, modelagem e análise.


## 5. Qualidade de Dados

A qualidade dos dados foi avaliada durante a transformação da camada Bronze para Silver. Foram realizadas verificações de quantidade de registros, tipos de dados, valores nulos, conversões e duplicidades.

### 5.1 Brent

A ingestão do Brent resultou em **9.973 registros**. A quantidade de registros foi comparada entre a fonte carregada e a tabela Bronze, garantindo que não houvesse perda de linhas durante a persistência.

### 5.2 PPI

Após a transformação para formato longo e consolidação de gasolina e diesel, a tabela Silver do PPI ficou com **12.142 registros**, sendo:

- **6.071 registros de gasolina**;
- **6.071 registros de diesel**.

Foram realizadas verificações nos campos essenciais `data_inicio`, `data_fim`, `produto`, `localidade` e `preco`, não sendo encontrados valores nulos nesses campos.

Também foi verificada a existência de duplicidades considerando a combinação de período, produto e localidade. O resultado foi de **zero combinações duplicadas** na tabela Silver.

### 5.3 Preços ANP

A camada Bronze dos preços ao consumidor possui **1.374.816 registros**.

Durante o tratamento, foram identificados **6 grupos com duplicidade**, correspondentes a **6 linhas duplicadas excedentes**. Após a remoção dessas duplicidades, a camada Silver passou a possuir **1.374.810 registros**.

A conversão do campo de valor de venda para formato numérico também foi validada. Não foram encontrados valores preenchidos na origem que se tornassem nulos em decorrência da conversão.

Dessa forma, a diferença entre as quantidades das camadas Bronze e Silver é explicada integralmente pela remoção das seis linhas duplicadas identificadas.


## 6. Análise de Dados

Para analisar o impacto do choque de 2026, o dia **28 de fevereiro de 2026** foi adotado como data de referência. Foram comparadas janelas anteriores e posteriores ao evento, além de serem analisadas correlações contemporâneas e com defasagens semanais.

As análises apresentadas são descritivas e correlacionais. Portanto, as associações encontradas entre as séries não são interpretadas isoladamente como evidência de causalidade.

### 6.1 Comportamento do Brent

Na janela analisada, o preço médio semanal do Brent passou de aproximadamente **US$ 66,64 por barril** no período anterior para **US$ 110,28** no período posterior à data de referência, representando uma elevação de aproximadamente **65,48%**.

O maior valor médio semanal observado no período posterior foi de aproximadamente **US$ 124,61 por barril**, na semana de 06/04/2026.

O resultado evidencia uma mudança expressiva no nível de preços internacionais após a escalada do conflito.

### 6.2 Comportamento do PPI

Os preços de paridade de importação também apresentaram aumento na comparação entre as 12 semanas anteriores e as 12 semanas posteriores à data de referência.

Para o **diesel**, a média passou de aproximadamente **3,26 para 5,64**, uma variação de **73,15%**.

Para a **gasolina**, a média passou de aproximadamente **2,42 para 3,98**, uma variação de **64,22%**.

Nesse recorte específico, portanto, o aumento percentual do PPI foi maior para o diesel.

### 6.3 Preços ao consumidor

Os preços observados pela ANP também aumentaram, mas em proporção inferior às variações observadas no Brent e no PPI.

O **Diesel S10** apresentou aumento médio de aproximadamente **18,08%**, passando de 6,14 para 7,25.

A **gasolina** apresentou aumento médio de aproximadamente **6,32%**, passando de 6,28 para 6,68.

A diferença de magnitude entre as etapas indica que as oscilações internacionais não foram reproduzidas de maneira imediata e proporcional nos preços ao consumidor.

### 6.4 Sensibilidade de gasolina e diesel

A comparação entre gasolina e diesel depende da etapa da cadeia e da janela analisada.

No choque de 2026, o diesel apresentou maior variação percentual tanto no PPI quanto nos preços ao consumidor.

Na análise histórica entre Brent e PPI, entretanto, a correlação contemporânea foi maior para a gasolina (**0,690**) do que para o diesel (**0,588**).

Esses resultados mostram que a ideia de "maior sensibilidade" depende da métrica considerada. O comportamento durante o evento específico e a associação histórica entre as séries representam perspectivas diferentes do problema.

### 6.5 Defasagens temporais

Foram calculadas correlações considerando diferentes defasagens semanais.

Na comparação entre **PPI e preços ao consumidor**, a associação mais forte ocorreu com uma semana de defasagem:

- Diesel: aproximadamente **0,656**;
- Gasolina: aproximadamente **0,479**.

Na comparação entre **Brent e preços ao consumidor**, a correlação também foi mais elevada com uma semana de defasagem:

- Diesel: aproximadamente **0,470**;
- Gasolina: aproximadamente **0,408**.

Os resultados sugerem que, na janela analisada, as alterações das referências internacionais apresentaram associação mais forte com os preços ao consumidor quando considerada aproximadamente uma semana de defasagem. Essa evidência deve ser interpretada como associação temporal, e não como demonstração de causalidade.

### 6.6 Diferenças regionais

A análise regional mostrou que o impacto não foi uniforme no território brasileiro.

Para o **Diesel S10**, as variações médias observadas foram:

- Centro-Oeste: **18,27%**;
- Norte: **15,00%**;
- Nordeste: **20,07%**;
- Sul: **17,89%**;
- Sudeste: **17,74%**.

Para a **gasolina**, foram:

- Centro-Oeste: **3,31%**;
- Norte: **6,72%**;
- Nordeste: **10,22%**;
- Sul: **4,07%**;
- Sudeste: **6,05%**.

Também foram observadas diferenças relevantes entre as unidades da federação. No Diesel S10, as variações ficaram entre aproximadamente **7,87% no Acre e 27,26% na Bahia**. Na gasolina, ficaram entre aproximadamente **1,09% no Distrito Federal e 12,85% na Bahia**.

Um exemplo da heterogeneidade entre combustíveis aparece no Distrito Federal: o Diesel S10 apresentou aumento de aproximadamente **20,73%**, enquanto a gasolina apresentou aumento de aproximadamente **1,09%**.

### 6.7 Síntese dos resultados

Em conjunto, os resultados mostram uma forte elevação do Brent após a data de referência, acompanhada por aumentos do PPI de gasolina e diesel. Os preços ao consumidor também aumentaram, mas em intensidade menor.

As análises de correlação indicaram associações temporais entre as séries e mostraram que, na janela comum analisada, as correlações com os preços ao consumidor foram mais fortes com aproximadamente uma semana de defasagem.

O comportamento também não foi homogêneo entre os combustíveis nem entre regiões e estados brasileiros. Isso reforça que a transmissão das oscilações internacionais para o mercado doméstico envolve diferentes etapas e não pode ser representada apenas pela comparação direta entre o Brent e o preço final nos postos.


## 7. Limitações e Trabalhos Futuros

O escopo originalmente proposto incluía também a análise do volume comercializado de gasolina, diesel e etanol e uma comparação específica do comportamento do etanol com os combustíveis mais diretamente relacionados ao petróleo.

Essas duas questões não foram respondidas nesta versão do MVP. Sua inclusão exigiria a incorporação de novas fontes de dados e novas etapas de ingestão, tratamento, modelagem e validação, ampliando significativamente o pipeline desenvolvido.

As perguntas foram mantidas no objetivo original para preservar o escopo inicialmente proposto e registrar possibilidades de continuidade do trabalho.

Como evolução futura, o projeto poderá incorporar dados de comercialização de combustíveis e séries específicas de etanol. Isso permitirá avaliar se o choque observado nos preços também esteve associado a mudanças no volume comercializado e investigar o comportamento de um combustível cuja dinâmica de formação de preços possui características diferentes das de gasolina e diesel.

Outras possibilidades de evolução incluem ampliar a janela temporal de análise, incorporar outras variáveis econômicas relevantes e aplicar métodos de séries temporais capazes de investigar de forma mais aprofundada as relações entre as diferentes etapas da formação dos preços.


## 8. Autoavaliação

O desenvolvimento deste MVP permitiu aplicar conceitos de Engenharia de Dados em um problema real, envolvendo fontes com formatos, granularidades e estruturas diferentes.

Um dos principais desafios foi tornar comparáveis as três bases utilizadas. Enquanto o Brent apresenta uma série histórica de preços internacionais, o PPI possui estrutura semanal distribuída por localidades e os dados da ANP possuem grande volume e granularidade de preços coletados em revendedores. A construção das camadas Silver e Gold foi importante para padronizar essas estruturas antes da análise.

Outro ponto relevante foi a preocupação com a qualidade e a rastreabilidade dos dados. A comparação das quantidades entre as etapas, a verificação de valores nulos, a validação das conversões e a identificação das duplicidades permitiram entender e justificar as alterações ocorridas entre Bronze e Silver.

A criação das séries semanais e da camada Gold também permitiu separar o tratamento dos dados das análises finais, tornando o fluxo mais organizado e reutilizável.

Nem todas as perguntas inicialmente propostas puderam ser respondidas nesta versão. As análises de volume comercializado e etanol exigiriam novas fontes e uma ampliação do pipeline. A decisão foi manter essas perguntas documentadas e priorizar a conclusão e validação das etapas já desenvolvidas, em vez de incorporar novas bases sem o mesmo nível de tratamento e verificação.

Como aprendizado, o projeto reforçou a importância de definir claramente a granularidade dos dados, validar cada etapa do pipeline e documentar as transformações realizadas. Em uma próxima evolução, além da inclusão das bases faltantes, seria interessante aprofundar a análise temporal e avaliar métodos adicionais para investigar as relações entre as séries.


## 9. Estrutura do Repositório

```text
mvp_engenharia_dados/
│
├── README.md
├── notebooks/
│   ├── 00_setup.ipynb
│   ├── 01_ingestao_brent.ipynb
│   ├── 02_ingestao_ppi.ipynb
│   ├── 03_ingestao_precos_anp.ipynb
│   ├── 04_profiling_qualidade_bronze.ipynb
│   ├── 05_modelagem_gold.ipynb
│   └── 06_analise_choque_geopolitico_resultados.ipynb
│
└── imagens/
    ├── catalogo_bronze_gold.png
    └── catalogo_raw_silver.png
```

Os notebooks seguem a ordem de execução do pipeline, desde a preparação do ambiente e ingestão das fontes até a criação das tabelas analíticas e interpretação dos resultados.  
