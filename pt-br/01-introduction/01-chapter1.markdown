# Primeiros passos #

Esse capítulo trata dos primeiros passos usando o Git. Inicialmente explicaremos alguns fundamentos sobre ferramentas de controle de versão, e passaremos ao tópico de como instalar o Git no seu sistema e finalmente em como configurá-lo para começar a trabalhar. Ao final do capítulo você entenderá porque o Git está por aí, porque usá-lo e como usá-lo.

## Sobre Controle de Versão ##

O que é controle de versão, e por que você deve se importar? O controle de versão é um sistema que registra as mudanças feitas em um arquivo ou um conjunto de arquivos ao longo do tempo de forma que você possa recuperar versões específicas. Nos exemplos desse livro você colocará arquivos de código fonte sob controle de versão, embora você pudesse fazê-lo com praticamente qualquer tipo de arquivo de um computador.

Se você é um designer gráfico ou um web designer e quer manter todas as versões de uma imagem ou layout (o que você deve querer, com certeza), É uma sábia decisão usar um Sistema de Controle de Versão ou Version Control System (VCS). Ele permitirá reverter arquivos ou um projeto inteiro a um estado anterior. Comparar mudanças que foram feitas ao decorrer do tempo, ver quem foi o último a modificar alguma coisa que pode estar causando problemas, quem introduziu um bug, e quando, e muito mais. Usar um VCS normalmente significa que se você estragou algo ou perdeu arquivos, poderá facilmente reavê-los. Além disso, você pode controlar tudo sem maiores esforços.

### Sistemas de Controle de Versão Locais ###

O método mais escolhido por muitas pessoas de controlar versões é copiar arquivos e guardá-los em outro diretório (talvez um diretório com data/hora, se forem espertos). Esta abordagem é muito comum por ser tão simples, mas é também muito suscetível a erros. É facíl esquecer em qual diretório você estava e gravar acidentalmente sobre o arquivo errado ou sobrescrever arquivos sem querer.

Para lidar com esse problema, alguns programadores desenvolveram há muito tempo VCS's locais que armazenavam todas as alterações dos arquivos sob controle de versão/revisão (ver Figura 1-1).

Insert 18333fig0101.png 
Figura 1-1. Diagrama de controle de versão local

Uma das ferramentas de VCS mais populares foi um sistema chamado RCS, que ainda é distribuído em muitos computadores até hoje. Até o o popular Mac OS X inclui o comando rcs quando se instala o kit de ferramentas para desenvolvedores. Basicamente, essa ferramenta mantém conjuntos de patches (ou seja, as diferenças entre os arquivos) entre cada mudança em um arquivo especial; a partir daí qualquer arquivo em qualquer ponto na linha do tempo pode ser recriado ao juntar-se todos os patches.

### Sistemas de Controle de Versão Centralizado ###

Outro grande problema que as pessoas encontram está na necessidade de trabalhar em conjunto com outros desenvolvedores, que usam outros sistemas. Para lidar com isso, foram desenvolvidos Sistemas de Controle de Versão Centralizado (Centralized Version Control Systems, ou CVCS). Esses sistemas - como por exemplo o CVS, Subversion e Perforce - possuem um único servidor central que contém todos os arquivos versionados, e vários clients que podem resgatar (check out) os arquivos do servidor. Por muitos anos, esse foi o modelo padrão para controle de versão.

Insert 18333fig0102.png 
Figura 1-2. Diagrama de Controle de Versão Centralizado

Esse arranjo oferece muitas vantagens, especialmente sobre VCS locais. Por exemplo, todo mundo tem conhecimento razoável sobre o que todos os outros no projeto estão fazendo. Administradores têm controle especifíco sobre quem-faz-o-quê; sem falar que é bem mais fácil administrar um CVCS do que é lidar com bancos de dados locais em todo cliente.

Entretanto, esse arranjo também possui sérias desvantagens. O mais óbvio é que o servidor central é unm ponto único de falha. Se o servidor fica fora do ar por uma hora, então ninguém pode trabalhar em conjunto ou salvar novas versões dos arquivos durante esse período. Se o disco do servidor do banco de dados for corrompido e não existir um backup adequado, perde-se todo o histórico de mudanças no projeto exceto pelas cópias momentâneas que os desenvolvedores possuem em suas cópias locais. VCSs locais também sofrem desse problema - sempre que se tem o histórico em um único local, corre-se o risco de perder tudo.

### Sistemas de Controle de Versão Distribuídos ###

É aqui que surgem os Sistemas de Controle de Versão Distribuídos (Distributed Version Control Systems, ou DVCSs). Em um DVCS (tais como Git, Mercurial, Bazaar or Darcs), os clientes não apenas fazem cópias momentâneas dos arquivos: eles são cópias completas do repositório. Assim, se um servidor falha, qualquer um dos repositórios dos clientes pode ser copiado de volta para o servidor para restaurá-lo. Cada checkout é na prática um backup completo de todos os dados (veja Figura 1-3).

Insert 18333fig0103.png 
Figura 1-3. Diagrama de Controle de Versão Distribuído

Além disso, muitos desses sistemas lidam muito bem com o aspecto de ter vários repositórios remotos com os quais eles podem colaborar, permitindo que vc trabalhe em conjunto com diferentes grupos de pessoas, de diversas maneiras, no mesmo projeto, simultaneamente. Isso permite que você estabeleça diferentes tipos de workflow que não são possíveis em sistemas centralizados, como exemplo direto o uso de modelos heiráquicos.

## Uma Breve História do Git ##

Assim como muitas coisas importantes na vida, o Git começou com um tanto de destruição criativa e acirrada controvérsia. O kernel do Linux é um projeto de software de código aberto de escopo razoavelmente grande. Durante a maior parte de sua existência (1991-2002), as mudanças no software eram repassadas como patches e arquivos compactados. Em 2002, o projeto do kernel do Linux começou a usar um sistema proprietário distribuído chamado BitKeeper.

Em 2005, o relacionamento entre a comunidade que desenvolvia o kernel e a empresa que desenvolvia comercialmente o BitKeeper se desfez, e o status de isento-de-pagamento da ferramenta foi revogado. Isso levou a comunidade de desenvolvedores do Linux (em particular Linus Torvalds, o criador do Linux) a desenvolver sua própria ferramenta baseada nas lições que eles aprenderam ao usar o BitKeeper. Alguns dos objetivos do novo sistema eram:

*	Velocidade
*	Design Simples
*	Robusto Suporte a desenvolvimento não linear (milhares de branches paralelos)
*	Totalmente distribuído
*	Capaz de lidar eficientemente com grandes projetos como o kernel do Linux (velocidade e volume de dados)

Desde sua concepção em 2005, o Git evoluiu e amadureceu a ponto de ser um sistema fácil de ser usado e ainda mantém suas características iniciais. É incrivelmente rápido, bastante eficiente com grandes projetos e possui um sistema impressionante para desenvolvimento não-linear (Veja no Capítulo 3).

## Git Básico ##

Enfim, como fazer uma descrição sucinta do Git? Essa é uma seção importante para assimiliar, já que usá-lo será muito mais fácil se você entender o que é o Git e os fundamentos de sua operação. À medida que você aprende a usar o Git, tente não pensar no que você já sabe sobre outros VCS como Subversion e Perforce; assim você consegue escapar de pequenas confusões que podem surgir usando a ferramenta. Git armazena e pensa sobre informação de uma forma totalmente diferente desses outros sistemas, apesar de possuir uma interface similar; entendendo essas diferenças lhe auxiliarão a ficar confuso.

### Capturas Instantâneas, ao invés de diferenças  ###

A maior diferença entre Git e qualquer outro VCS (Subversion e similares incluso) estão nos conceitos que o Git tem sobre os dados. Conceitualmente, a maior parte dos outros sistemas armazena informação como uma lista de mudanças por arquivo. Esses sistemas (CVS, Subversion, Perforce, Bazaar, etc) tratam a informação que eles mantém como um conjunto de arquivos e as mudanças feitas a cada arquivo ao longo do tempo, conforme ilustrado na Figura 1.4.

Insert 18333fig0104.png 
Figura 1-4. Outros sistemas costumam armazenar dados como mudanças em uma versão-base de cada arquivo.

Git não pensa na informação dessa forma, nem a armazena com esse princípio. Ao invés disso, o Git considera que os dados são como um conjunto de capturas instântaneas (snapshots) de um mini-sistema de arquivos. Cada vez que você faz um commit ou salva o estado do seu projeto no Git, o que basicamente o Git faz é tirar uma foto de todos os seus arquivos naquele momento e armazena uma referência para essa captura. Para ser eficiente, se nenhum arquivo foi alterado, a informação não é armazenada novamente - apenas um link para o arquivo idêntico anterior que já foi armazenado. A figura 1-5 representa melhor a forma que o Git lida com dados.

Insert 18333fig0105.png 
Figura 1-5. Git armazena dados como snapshots do projeto ao longo do tempo.

Essa é uma distinção importante entre Git e quase todos os outros VCSs. Isso leva o Git a reconsiderar quase todos os aspectos de controle de versão que os outros sistemas copiaram da geração anterior. Também faz com que o Git se comporte mais como um mini-sistema de arquivos com algumas poderosas ferramentas construídas em cima dele, ao invés de um simples VCS. Nós vamos explorar alguns dos benefícios que você tem ao lidar com dados dessa forma, quando tratarmos do assunto de branching no Capítulo 3.

### Quase Toda Operação É Local ###

A maior parte das operações no Git precisa apenas de recursos e arquivos locais para seu funcionamento - geralmente nenhuma outra informação é necessária de outro computador na sua rede. Se você está acostumado a um CVCS onde a maior parte das operações possui latência por conta de comunicação com a rede, esse aspecto do Git fará com que você pense que os deuses da velocidade abençoaram Git com poderes sobrenaturais. Uma vez que todos o histórico do projeto está no seu disco local, a maior parte das operações parece ser instantânea.

Por exemplo, para navegar o histórico do projeto, o Git não precisa requisitar ao servidor o histórico para que possa apresentar a você - basta apenas uma leitura da base de dados local. Isso significa que você vê o histórico do projeto de quase instanteneamente. Se você quiser ver todas as mudanças introduzidas entre a versão atual de um arquivo e a versão de um mês atrás, o Git pode buscar o arquivo de um mês atrás e fazer um cálculo de diferenças localmente, ao invés de ter que requisitar ao servidor que faça o cálculo, ou pedir o arquivo antigo para que o cálculo possa ser feito localmente.

Isso também significa que há pouca coisa que você não pode fazer, se estiver offline ou sem acesso a uma VPN. Se você entra em um avião ou trem e quiser trabalhar, você pode fazer commits livre de preocupações até o instante que você tem acesso a rede novamente. Se você estiver indo para casa e seu cliente de VPN não estiver funcionando, você ainda pode trabalhar. Em outros sistemas, fazer isso ou é impossível ou uma tarefa árdua. No Perforce, por exemplo, você não pode fazer muita coisa quando não está conectado ao servidor; e no Subversion e CVS, você pode até editar os arquivos, mas não pode fazer commits das mudanças já que sua base de dados estará offline. Pode até parecer que não é grande coisa, mas você pode se surpreender com o tanto de diferença que pode lhe trazer.

### Git Possui Integridade ###

Tudo no Git tem seu checksum calculado antes que seja armazenado e é referenciado pelo checksum. Isso significada que é impossível mudar o conteúdo de qualquer arquivo ou diretório sem que o Git tenha conhecimento da ação tomada. Essa funcionalidade é parte fundamental do Git e é integral à sua filosofia. Você não pode perder informação em trânsito ou ter arquivos corrompidos sem que o Git seja capaz de detectar o ocorrido.

O mecanismo que o Git usa para fazer o checksum é chamado de hash SHA-1, uma string de 40 caracteres composta de characteres hexadecimais que é calculado a partir do conteúdo de um arquivo ou estrutura de um diretório no Git. Um hash SHA-1 parece com algo mais ou menos assim:

	24b9da6552252987aa493b52f8696cd6d3b00373

Você vai encontrar esses hashes em todo canto, uma vez que Git os utiliza tanto. Na verdade, tudo que o Git armazena é identificado não por nome do arquivo mas pelo valor do hash do seu conteúdo.

### Git Generally Only Adds Data ###

When you do actions in Git, nearly all of them only add data to the Git database. It is very difficult to get the system to do anything that is undoable or make it erase data in any way. As in any VCS, you can lose or mess up changes yo