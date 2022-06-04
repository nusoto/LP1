# Crime and Dna Database

### Table of Contents
1. [Introdução](#1-introdução)
2. [Background](#2-background)
3. [Entradas](#3-entradas)
4. [Interface](#4-interface)
5. [Modelagem do Problema](#5-modelagem-do-problema)
6. [Dados para testes](#6-dados-para-testes)
7. [Método de Testes](#7-método-de-testes)

#  1. Introdução

Neste trabalho iremos desenvolver um programa de processamento de DNA. O programa, chamado **crime_n_dna_db**, que será responsável por guardar e processar
informações de sequencias de DNA especialmente para aplicações relacionadas com processamento de perfis de DNA associados à suspeitos e vítimas em crimes.

O programa deverá fazer as seguintes funções:
1. Armazenar informações de DNA
    1. Informações que são adicionadas em tempo de execução
    2. Informações que são adicionadas na inicialização do programa
2. Cadastrar informações de cenas de crimes
    1. Informaçoes que são adicionadas em tempo de execução
    2. Informações que são carregadas na inicialização do programa
    3. Informações de DNA que aparecem em uma mesma cena de crime devem ser "linkadas"
3. Realizar consultas no banco de dados

Na inicialização o programa receberá como entrada dois arquivos CSV, o primeiro representando as informações de DNA cadastradas e o segundo representando
as cenas de crime cadastradas. O programa deverá carregar os dois arquivos e inicializar a base de dados. Aós o carregamento dos arquivos iniciais o programa
deverá receber do usuário comandos para realizar o restante das funcionalidades listadas, a lista de comandos a ser implementada será descrita à seguir.

# 2. Background

O DNA, que carrega as informações genéticas dos seres vivos, é usado pela justiça há décadas. Mas como, exatamente, funciona o processo de criação de perfis de DNA?
Dada uma sequencia de DNA, como os investigadores identificam à quem aquele DNA pertence?

Bem, na verdade o DNA é apenas um conjunto de moléculas chamadas nulceotídeos, agrupdadas em um formato particular (uma hélice dupla). Cada nucleotídeo do DNA contém
um de quatro diferente bases: adenina (`A`), citosina (`C`), guanina (`G`) e timina (`T`). Cada célula humana tem milhões desses nucleotídeos agrupados em sequência.
Algumas porções dessa sequencia (por exemplo, o genoma) são iguais, ou ao menos muito similares, entre todos os humanos, mas outras partes tem uma alta diversidade
genética, logo variando entre as pessoas.

Um ponto onde o DNA tende a ser muito diverso é nos [**Short Tandem Repeats** (STRs)](https://en.wikipedia.org/wiki/STR_analysis). Um STR é uma sequencia curta de DNA
que se repete uma determinada quantidade de vezes em lugares específicos do DNA. O número de vezes que um STR se repete varia muito entre as pessoas. Abaixo,
por exemplo, Alice o STR `AGAT` repetido 4 vezes em seu DNA, enquanto bob tem o mesmo STR repetido 6 vezes.

<img src="./pics/ALICE_BOB_DNA.png" width="350">

Dessa forma podemos usar os STRs para realizar uma "comparação" entre os DNAs de Bob e Alice, de forma que é possível distinguir os dois. Se usarmos múltimos STRs, 
ao invés de apenas 1, podemos aumentar a acurácia do perfil. Se a probabilidade de duas pessoas terem o mesmo número de repetições de um STR
é de 5%, e os analistas usam 10 diferentes STR, a problabilidade de duas amostras de DNA serem iguais, apenas por sorte é de 1 em um quadrilhão (assumindo que cada
STR é independente um do outro). Assim se duas amostras de DNA tem o mesmo número de repetições de STRs, o analista é bem confiante que o DNA vem da mesma pessoa. 
O [CODIS](https://www.fbi.gov/services/laboratory/biometric-analysis/codis/codis-and-ndis-fact-sheet), a base de dados do FBI, usa 20 STRs diferentes como parte
da análise de amostras de DNA.

Dessa forma os investigadores podem realizar coletas de traços de DNA em cenas de crimes, e realizar a operação de "matching" onde as informações de DNA são processadas
e comparadas com um conjunto conhecido de STRs que ficam cadastrados em um banco de dados especial. Embora possa parecer simples pela descrição, a informação
de DNA é muito vasta e apenas processar STRs para uma pessoa pode demorar uma quantidade relevante de tempo. Para atacar esse problema os investigadores usam segmentos
conhecidos do DNA que são selecionados especialmente para essa tarefa.


# 3. Entradas

O programa receberá duas entradas, inicialmente, um arquivo CSV contendo as bases de DNA e outro arquivo CSV contendo as informações das cenas de crime. O programa
deve então ficar esperando por comandos do usuário para realizar operações nas bases de dados, não há comando de finalização o programa termina quando EOF é lido
ou quando o usário digitar <kbd>Ctrl</kbd> + <kbd>d</kbd>.

## 3.1 Arquivos CSV

Um exemplo do arquivo de base de dados de dna pode ser encontrado em [./data/dnadb.csv](./data/dnadb.csv). Cada linha do arquivo representa as seguintes informações, 
separadas por vírgula:
- Identificador: Uma string com ou sem espaços que define um identificador para aquela entrada de DNA.
- Raw Data: Uma informação de DNA que é definida por uma string composta por A, C, G ou T.
- STRS: Uma lista composta pelos STRs processados naquele DNA junto com seus valores no formato STR:NUMERO, a lista é separada por espaços

Um exemplo do arquivo de base de dados de cenas do crime pode ser encontrado em [./data/crimedb.csv](./data/crimedb.csv). Cada linha do arquivo tem os seguintes 
campos, separados por vírgula:
- Identificador: Uma string com ou sem espaços que define um identificador para aquela cena de crime.
- DNA Samples: Um conjunto de DNAs que foram encontrados naquela cena de crime separados por espaço

## 3.2 Comandos de manipulação das bases

Após o carregamento das bases de dados o programa deverá receber e interpretar os comandos listados abaixo:

### Comandos relacionados à DNA

- **add_dna \<identificador_dna\> arquivo.csv**: Adiciona uma informação de dna no banco de dados. O identificador precisa ser único no banco, o arquivo csv deve conter o
dado relativo ao _Raw Data_, ou seja apenas uma string composta por A, C, G, ou T;
  - Caso de erro: identificador inválido (duplicado); arquivo inexistente.
- **del_dna \<identificador_dna\>**: Remove uma informação de DNA do banco de dados.
  - Caso de erro: identificador inválido (não encontrado).
- **proc_dna_str \<identificador_dna\> str**: Processa um STR de um DNA representado por \<identificador\>. As informações do
STR, como sua string e seu valor calculado devem ser salvas na informação de DNA correspondente.
  - Caso de erro: identificador inválido (não encontrado); str com tamanho diferente de 4.
- **del_dna_str \<identificador_dna\> str**: Remove uma informação de STR no DNA do banco de dados.
  - Caso de erro: identificador inválido (não encontrado); str inválido (não encontrado).

### Comandos Relacionados à Cenas de Crime

- **add_cena \<identificador_cena\> arquivo.csv**: Adiciona uma cena de crime ao banco. O identificador precisa ser único no banco, o arquivo csv deve conter informações de
_Raw Data_ relativas aos DNA's presentes cena do crime (separados por vírgula);
  - Caso de erro: identificador inválido (duplicado); arquivo inexistente.
- **del_cena \<identificador_cena\>**: Remove uma informação de cena do crime do banco de dados.
  - Caso de erro: identificador inválido (não encontrado).

### Comandos de consulta

- **get_dna_strs \<identificador_dna\>**: Imprime os STRs correspondentes à um DNA passado
  - Caso de erro: identificador inválido (não encontrado).
- **get_involved \<identificador_cena\>**: Imprie os possíveis envolvidos em uma cena do crime usando o Perfil de DNA dos DNAs da cena como parâmetro para encontrar
os possíveis envolvidos.
  - Caso de erro: identificador inválido (não encontrado).
- **get_scene_involved \<identificador_dna\>**: Imprime todas as cenas de crime em que o DNA passado é encontrado.
  - Caso de erro: identificador inválido (não encontrado).
- **get_dna_involved \<identificador_dna\>**: Imprime todos os identificadores de DNA que estão presentes em alguma cena do crime que o DNA passado é envolvido.
  - Caso de erro: identificador inválido (não encontrado).

### Validação dos comandos

Seu programa deve validar os comandos em cada um dos casos de erro listados neles, bem como verificar se o comando existe e se a **lista de argumentos está correta**.

# 4. Interface

Seu programa, chamado `crime_n_dna_db`, deve ler da linha de comando da seguinte forma:
```
% ./crime_n_dna_db
Usage: crime_n_dna_db -d <dnadb_file> -s <crimedb_file>
  Onde os argumentos são:
      <dnadb_file>    A base de dados contendo DNAs já cadastrados no banco.
      <crimedb_file>  A base de dados contendo cenas de crime já existentes no banco.
```

O restante da execução do programa é a leitura dos comandos vindos do usuário e a saída no terminal correspondente a cada comando. A forma como a saída
será apresentada pode ser escolhida por você.

# 5. Modelagem do Problema

Você pode criar quantas classes achar necessário, mas faça ao menos 3:

+ Uma classe para armazernar as informações relativas ao DNA de uma pessoa bem como realizar o processamento de STRs.
+ Uma classe para armazenar as informações relativas à cenas de crime (de preferencia referenciando a classe anterior)
+ Uma classe para realizar o processamento dos comandos

# 6. Dados para testes

Um dos problemas que você irá encontrar aqui é como testar seu programa, uma boa base de dados é a base do [National Institute of Standards and Technology (Nist)](https://strbase.nist.gov/seq_info.htm). Nesta base você encontra uma tabela com uma coluna denominada "Locus", que é a parte do dna onde aquele str é encontrado. Clicando nos links você é levado
à uma página contendo a informação de DNA com o STR em questão destacado.

Outra forma de encontrar dados é usando a [National Library of Medicine](https://www.ncbi.nlm.nih.gov/), nos links abaixo você encontra algumas páginas com amostras:

1. [CSF1PO](https://www.ncbi.nlm.nih.gov/nuccore?term=380561%5BBioProject%5D)
2. [D2S1338](https://www.ncbi.nlm.nih.gov/nuccore?term=380556%5BBioProject%5D)
3. [D3S1358](https://www.ncbi.nlm.nih.gov/nuccore/?term=D3S1358)
4. [D5S818](https://www.ncbi.nlm.nih.gov/nuccore/?term=D5S818)
5. [D8S1179](https://www.ncbi.nlm.nih.gov/nuccore/?term=D8S1179)

Toso esses links foram encontrados usando a página da [National Library of Medicine (nucleotide)](https://www.ncbi.nlm.nih.gov/nuccore/), buscando pelo locus contido na página do Nist. Veja que nem todos os loci possuem exemplos de nucleotídeos para podermos analisar, embora existam bastante exemplos em alguns casos.

Outra forma (não aconselhada), é você usar alguma função aleatória para gerar os DNAs, eu realmente não aconselho essa direção uma vez que os dados podem não fazer muito sentido. Por fim você também pode usar STRs não padronizados (não obrigatoriamente usados pelo Nist), como forma de testar seu algoritmo.

# 7. Método de Testes

Neste trabalho o onus de testar e mostrar as implementações das funcionalidades é totalmente seu! Invista em criar testes válidos e em automatizar o processo de testes uma vez
que você irá precisar demonstrar a implementação dos comandos, casos de erro e as funcionalidades.

Tente combinar os comandos e principalmente tente verificar os casos relacionados aos _Deletes_, por exemplo: O que ocorre se eu deleto um DNA que está envolvido em uma cena e depois chamo o comando **get_involved**? 

## Authorship

**This assignment is inspired by the DNA Profiler assignment from Brian Yu, Harvard University, [brian@cs.harvard.edu](http://nifty.stanford.edu/2020/dna/brian@cs.harvard.edu)**

**Autor: Julio Melo, [julio@imd.ufrn.br](mailto:julio@imd.ufrn.br)**

