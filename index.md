---
title: Projeto S2C2
layout: home
---

# INTRODUÇÃO {#sec-01}

O desenvolvimento de uma rede de comunicação capaz de atender às necessidades das aplicações da Família de Aplicativos de Comando e Controle da Força Terrestre (FAC2FTer) envolve explorar diferentes alternativas de projeto para garantir eficiência na troca de dados entre tropas em variados cenários operacionais. Essas soluções devem representar com fidelidade o contexto militar, incluindo movimentação e posicionamento de unidades.

Com a evolução das aplicações de comando e controle e os investimentos do Exército Brasileiro em novos rádios táticos e no projeto de Rádio Definido por Software (RDS), torna-se essencial aprimorar protocolos e funcionalidades que suportem comunicações mais robustas. Nesse contexto, simulações que reproduzem o comportamento das tropas e o fluxo típico de informações em operações militares tornam-se ferramentas fundamentais para orientar o desenvolvimento dessas capacidades.

Este documento apresenta a visão geral do Sistema de Simulação no âmbito do projeto Sistema de Sistemas de Comando e Controle (S2C2), destacando sua arquitetura, seus principais componentes e o manual de utilização da ferramenta. O foco está nos módulos responsáveis pela simulação da rede de comunicação e pelo comportamento dos agentes que representam as tropas no ambiente simulado, bem como nas orientações para uso da solução desenvolvida.

# VISÃO ARQUITETURAL {#sec-02}

Para representar um ambiente operacional próximo ao real, adotou-se uma abordagem integrada que combina um Simulador de Sistema Multiagente (MAS) com um Emulador de Rede. Essa combinação permite modelar tanto o comportamento das tropas quanto os desafios de comunicação característicos de cenários militares. A Figura 1 apresenta a arquitetura geral do sistema S2C2 EmuSim, responsável por configurar e orquestrar as simulações.

O sistema é composto por módulos que se complementam. A interface gráfica (GUI) e o componente lógico S2C2 permitem ao operador criar, carregar e gerenciar cenários de simulação. O conhecimento doutrinário militar é fornecido por uma ontologia externa em OWL, utilizada pelo OWL Manager para manter a consistência lógica dos cenários e apoiar a criação e edição de instâncias de simulação.

Parâmetros operacionais adicionais — como seleção do mapa, modo de execução e opções de visualização — são controlados pelo Parameters Manager, que repassa a configuração final ao orquestrador EmuSim. Este é o núcleo do sistema, responsável por coordenar a execução conjunta do Simulador MAS e do Emulador de Rede, garantindo a sincronização entre o comportamento dos agentes e o fluxo de comunicação.

A arquitetura também inclui um módulo de gerenciamento de dados, responsável por registrar resultados das simulações, manter o modelo de dados e interagir com a ontologia. Para análise e visualização, métricas e indicadores são disponibilizados por meio de dashboards integrados ao Grafana, permitindo observar o desempenho dos cenários simulados.

3. Modelagem do Sistema {#sec-03}

Esta seção descreve como os principais componentes do S2C2 se organizam e contribuem para a execução das simulações, aprofundando a visão geral apresentada na Seção 2.

3.1 Componentes GUI, S2C2 e OWL Manager {#subsec-03-1}

A aplicação inicia pela GUI, responsável pela interação com o usuário. As ações realizadas na interface são encaminhadas ao componente S2C2, que concentra a lógica do sistema e coordena as operações internas.

O OWL Manager gerencia a ontologia OWL IME S2C2 Base, que define regras da doutrina militar utilizadas na construção dos cenários (tipos de agentes, restrições, parâmetros de comunicações etc.). A partir dessa ontologia:

OWL Create Instance gera as instâncias do cenário.

OWL Update Instance atualiza essas instâncias conforme alterações feitas pelo usuário.

O Parameters Manager organiza e exibe os parâmetros disponíveis, respeitando as regras da ontologia.

3.2 Componente EmuSim e suas Relações {#subsec-03-2}

O EmuSim integra a simulação multiagente e a emulação de rede. Implementado em Python, ele sincroniza o Simulador MAS (NetLogo) e o Emulador de Redes (Mininet-WiFi), mantendo alinhados o movimento dos agentes e o comportamento das comunicações.

Durante a execução:

O NetLogo controla os agentes e o ambiente do terreno.

O Mininet-WiFi executa as aplicações C2 e simula enlaces de comunicação isolados.

Interfaces como PyNetLogo e MN_Wifi permitem o envio contínuo de dados entre os sistemas.

Cada tropa simulada corresponde a uma estação virtual com sua própria pilha de rede. Os dados coletados são armazenados pelo DataManager para análise posterior.

3.3 Modelagem dos Mapas e Ambiente de Simulação {#subsec-03-3}

A modelagem dos mapas utiliza arquivos Shapefile (SHP) provenientes de bases oficiais, como o BDGEx. Esses arquivos são processados em SIG (ex.: QGIS) para gerar camadas compatíveis com o NetLogo.

A criação de novos cenários requer:

filtragem das camadas vetoriais,

definição do tamanho real do terreno,

geração da imagem PNG,

criação do arquivo de configuração do mapa.

A escolha do cenário é feita por um menu dropdown (Figura abaixo).

<figure id="fig:dropdown"> <p><img src="assets/images/fig11.mapas_dropdown.jpeg" style="width:80%" /></p> <figcaption>Dropdown para escolha de mapas.</figcaption> </figure>

A simulação pode ocorrer:

Sobre o mapa vetorial, com interação direta com o grid;

Sobre a imagem PNG, privilegiando visualização.

<figure id="fig:mapa_netlogo"> <p><img src="assets/images/fig12.mapa_simulação.jpeg" style="width:80%" /></p> <figcaption>Simulação com mapa vetorial.</figcaption> </figure> <figure id="fig:mapa_png"> <p><img src="assets/images/fig13.mapa_PNG.jpeg" style="width:80%" /></p> <figcaption>Simulação com mapa PNG.</figcaption> </figure>
Granularidade

A granularidade do grid impacta o desempenho: patches muito pequenos aumentam significativamente o custo computacional. O sistema permite ajustar esse valor por meio de um controle deslizante (Figura abaixo), recalculando automaticamente as dimensões da grade.

<figure id="fig:patch_slider"> <p><img src="assets/images/fig14.slider_patch.jpeg" style="width:80%" /></p> <figcaption>Ajuste de tamanho de patches.</figcaption> </figure>

# MODELOS DE FLUXO DO SISTEMA {#sec-04}

A execução do sistema comporta uma série de modelos de fluxo, que vão
desde o fluxo da aplicação como um todo até a modelagem do comportamento
de agentes e da identificação de situações de fogo amigo. Estes
diferentes fluxos são apresentados nas subseções a seguir. Importante
frisar que, apesar de se mencionar ao longo desse capítulo a aplicação
C2 Blue Force Tracking (*BFT*), o sistema desenvolvido é genérico para
qualquer aplicação C2 e essa aplicação em especifico é usada apenas para
facilitar a compreensão de como o sistema funciona de forma geral.

## Modelagem do Fluxo da aplicação {#sec-04.1}

O modelo de fluxo da aplicação está detalhado na
Figura [8](#fig:8.simulatio.flow){reference-type="ref"
reference="fig:8.simulatio.flow"}, sendo detalhado a seguir.

1.  O usuário inicia o sistema utilizando o componente "GUI" e seleciona
    iniciar uma simulação, configurandoo os paramêtros que lhe são
    apresentados.

2.  O componente lógico S2C2 Menu valida a solicitação do usuário dentro
    das restrições estabelecidas pela doutrina militar, estipuladas
    pelos componentes de *OWL Manager* e solicita o disparo da simulação
    ao componente EmuSim.

3.  O "EmuSim" inicializa os componente *Simulador MAS*.

4.  O "EmuSim" inicializa os componente *Emulador de Redes*.

5.  O "EmuSim" instancia os componente *BFT* implementado dentro do
    componente *Emulador de Redes*.

6.  O sistema executa o loop da simulação até que o objetivo da mesma
    seja alcançado:

    1.  O *EmuSim* recupera do *Simulador MAS* as posições e dados dos
        nós da simulação por meio da interface interface PyNetLogo.

    2.  O *EmuSim* envia os dados coletados para o *Emulador de Redes*
        por meio da interface interface MN_Wifi.

    3.  O *EmuSim* envia os mesmos dados para a aplicação *BFT* por meio
        da interface interface MQTT.

    4.  A aplicação *BFT* devolve os dados processados para o componente
        *EmuSim*.

    5.  O componente *EmuSim* envia para o *Simulador SMA* os dados
        processados pelo *BFT*, que são usados para determinar e
        representar situações de fogo amigo na simulação.

    6.  Os dados resultantes são escritos pelo *EmuSim* no banco de
        dados.

7.  O *EmuSim* envia uma mensagem para interromper tanto o *Emulador de
    Redes*.

8.  O *EmuSim* envia uma mensagem para interromper o *Simulador SMA*,
    finalizando a simulação.

<figure id="fig:8.simulatio.flow" data-latex-placement="!ht">
<p><img src="assets/images/fig8.simulation.flow.png" alt="image" /> <span
id="fig:8.simulatio.flow" data-label="fig:8.simulatio.flow"></span></p>
<p>Fonte: os autores.</p>
<figcaption>Diagrama de sequência da aplicação para a execução de
cenários de simulação</figcaption>
</figure>

## Modelagem do Sistema Multiagente {#sec-04.2}

O cenário de simulação do sistema consiste em um mapa de batalha 2D de
tamanho $N \times M$, composto por uma grade $p_{x,y}$ de caminhos
possíveis, para o qual é possível personalizar a geografia do mapa, o
número de unidades aliadas e inimigas, a posição inicial e final para os
mesmos e também a localização de diferentes pontos de controle.

Um exemplo disso está ilustrado na Figura
[9](#fig:4.simulation){reference-type="ref"
reference="fig:4.simulation"}. Nela, trinta (30) agentes aliados são
representados por hexágonos azuis, que simbolizam soldados autônomos a
pé. Esses agentes devem atravessar o mapa, partindo da posição inicial
na base aliada (A), indicada por um quadrado branco próximo ao canto
inferior esquerdo do mesmo, até o destino (C), na base inimiga,
representada por um quadrado vermelho próximo ao canto superior direito
do mapa. Durante o trajeto, cada agente deve passar por pelo menos um
dos diferentes pontos de controle (B), representados por quadrados
amarelos ou laranjas, ao mesmo tempo em que confrontam dez (10) agentes
inimigos, simbolizados por hexágonos vermelhos, que se movem na direção
oposta, em direção à base aliada.

<figure id="fig:4.simulation" data-latex-placement="ht">
<p><img src="assets/images/fig4.butiaSimulation.jpeg" style="width:60.0%"
alt="image" /> <span id="fig:4.simulation"
data-label="fig:4.simulation"></span></p>
<p>Fonte: os autores.</p>
<figcaption>Simulação de ataque mostrando incidentes e baixas das
unidades.</figcaption>
</figure>

Para navegar pelo mapa, tanto unidades aliadas come inimigas navegam
utilizam o algoritmo A\*, que emprega uma função de avaliação, $f$, para
encontrar o caminho mais curto entre dois nós, minimizando a soma da
função de custo e dos valores heurísticos [@li2020path]. A equação
[\[a_star_function\]](#a_star_function){reference-type="ref"
reference="a_star_function"} define $f$ para qualquer agente $a_{i}$ na
simulação.

$$\begin{equation}
    f(a_{i}) = c_{a_{i}} + h_{a_{i}}
    \label{a_star_function}
\end{equation}$$

Por requisição dos stakeholders, a função de custo não leva em conta
nuances de decisão comportamental humana, pois o foco está em resolver
questões de comunicação, especialmente situações de fogo amigo (seção
[5.2](#sec-04.3){reference-type="ref" reference="sec:04.3"}). Portanto,
a equação [\[cost_function\]](#cost_function){reference-type="ref"
reference="cost_function"},que calcula a função de custo $c$ para
qualquer agente $a_{i}$ em direção ao objetivo $g$, tem como objetivo
encontrar a soma ótima para cada patch $p_{x,y}$ desde a posição atual
$pos$ do agente até o destino $g$, multiplicada pelo peso do tipo de
terreno $w_{p_{x,y}}$ para o patch atual. Os valores dos pesos
$w_{p_{x,y}}$ para cada patch $p_{x,y}$ do mapa são:

- Planícies: 1

- Áreas alagadas/terrenos de baixa elevação: 2

- Terrenos de média elevação: 3

- Terrenos intransponíveis (águas profunda ou terrenos elevados): 4

$$\begin{equation}
    c_{a_{i}}=\sum_{g}^{pos}\left ( p_{x,y} \times w_{p_{x,y}}\right )
    \label{cost_function}
\end{equation}$$

Por fim, o valor heurístico $h$ para o caminho de qualquer agente
$a_{i}$ até o seu objetivo é a distância Euclidiana entre sua posição
atual $(x_{a_{i}}, y_{a_{i}})$ e o patch do objetivo $g$, ou mais
precisamente $(x_{g}, y_{g})$. Assim,
$h(a_{i})=\sqrt{(x_{a_{i}} - x_{g})^2 + (y_{a_{i}} - y_{g})^2}$.

### Modelagem do Estado dos Agentes Sob Staque {#sec-04.1.1}

Ainda que as unidades aliadas e inimigas representadas por cada
simulação sejam inicialmente representadas como hexágonos de cor azul ou
vemelha, essas cores são dinamicamente alteradas pela simulação para
refletir o estado atual de cada agente, como é possível ver na
Figura [9](#fig:4.simulation){reference-type="ref"
reference="fig:4.simulation"} apresentada anteriormente.
Particularmente, referente a aliados e inimigos essas cores podem ser,
respectivamente:

- **Saudável**: azul/vermelho

- **Ferido**: roxo/rosa

- **Assistência médica urgente**: amarelo/laranja

- **Morto**: cinza/preto

Esses estados e suas devidas transições são definidos a partir de uma
Máquina de Estados Finitos (Finite State Machine, FSM), um tipo de
sistema estruturado que se caracteriza por um número finito de estados
interconectados por meio de transições. Cada estado abrange
comportamentos ou algoritmos específicos que são ativados ao entrar no
estado ou durante sua fase ativa. Os estados são representados como nós
conectados por transições, garantindo acessibilidade a todos os estados,
de forma direta ou indireta [@jagdale2021finite].

Entidades que operam dentro de uma FSM transitam de seu estado atual
$s_i$ para outro estado $s_j$ com base em condições predefinidas
associadas aquele estado. O cumprimento dessas condições aciona uma
transição para o estado conectado correspondente, permitindo mudanças
dinâmicas de estado dentro da FSM [@jagdale2021finite].

A FSM empregada para os agentes desta aplicação está ilustrada na Figura
[10](#fig:5.fsm){reference-type="ref" reference="fig:5.fsm"}, que
identifica o estado *Saudável* como o estado inicial dos agentes
enquanto que *Assistência médica urgente* e *Morto* são estados finais
deste fluxo. A adoção de *Assistência médica urgente* como estado final,
além de *Morto*, vai ao encontro da Convenção de Genebra, que protege
indivíduos nesse estado contra novos danos.

<figure id="fig:5.fsm" data-latex-placement="ht">
<p><img src="assets/images/fig5.fsm.png" style="width:80.0%" alt="image" /> <span
id="fig:5.fsm" data-label="fig:5.fsm"></span></p>
<p>Fonte: os autores.</p>
<figcaption>FSM para os estados dos agentes da aplicação sob
ataque.</figcaption>
</figure>

Agentes mudam de estado quando atacados. O estado resultante é
determinado por um valor aleatório $d$, usado para avaliar a dificuldade
do ataque. Se $d > 0.8$, nenhum ataque ocorre, semelhante a uma unidade
decidindo não engajar um alvo. No entanto, se $d \leq 0.8$, o atacante
dispara, acionando uma transição de estado.

A modelagem desta FSM assume que a probabilidade de acertar um ponto
específico do alvo diminui à medida que a letalidade do impacto aumenta.
Por exemplo, tiros direcionados a áreas altamente letais, como a cabeça,
são mais difíceis de atingir. Perspectiva que foi incorporada à FSM ao
estabelecer limites para que $d$ acione uma transição.

Esses limites incluem uma função $precisao$ relacionada ao ataque $atq$,
que tem como dois fatores principais o alcance da arma do atacante e a
distância em linha reta entre o atacante e o alvo. Função que é
construída de forma que $precisao(atq) \leq 0.8$.

A transição de estado resultante de um ataque $(d \leq 0.8)$ depende do
estado atual do agente e do valor aleatório de dificuldade sorteado para
$d$. Por exemplo, o estado inicial *Saudável* conta com as seguintes
possíveis transições:

- $d > precisao(atq)$: O ataque erra, e a unidade alvo permanece
  *Saudável*.

- $0.25 \leq d \leq precisao(atq)$: O ataque acerta, a unidade alvo
  permanece em combate, transita para o estado *Ferido* e tem sua
  mobilidade reduzida.

- $0.25 < d \leq 0.5$: O ataque é grave, a unidade alvo é forçada a
  evacuar e transita para o estado *Assistência médica urgente*.

- $0.5 < d \leq 0.8$: O ataque é crítico, a unidade alvo é abatida e
  transita para o estado *Morto*.

A quantidade de informação que uma unidade possui sobre outra influencia
diretamente a decisão de atacar. Por isso, a comunicação é vital para
soldados a pé. O reconhecimento visual permite identificar aliados sem
depender da rede, porém só é possível dentro de uma distância máxima em
linha reta; ainda assim, continua essencial. Falhas nesse reconhecimento
--- por limitação de alcance, condições ambientais ou erro de percepção
--- podem gerar incidentes de fogo amigo, que são discutidos na próxima
seção.

# APLICAÇÕES DE S2C2 {#sec-05}

Para analisar o desempenho do simulador, desenvolveram-se diferentes
aplicações de S2C2 ao longo do projeto. Estas aplicaçãoes e os
resultados obtidos a partir das mesmas são discutidas a seguir.

## Modelos de Dados da Aplicação {#subsec:3.3}

Para melhor organizar as informações geradas pela simulação, bem como,
para facilitar futuras consultas a dados e a execução em tempo real,
foram criados diferentes conjuntos de dados para a troca de informações
entre agentes do sistema, simulador e emulador. Estes conjuntos são
divididos em Mensagens, Posições e Colinas.

- Mensagens: registra tentativas de comunicação e comunicações
  bem-sucedidas.

- Posições: registra o log da trajetória seguida por cada unidade.

- Colinas: registra a presença de elevações do terreno (colinas) que
  possam interferir na comunicação entre pares de unidades durante a
  simulação.

### Posições

O conjunto de dados Posições identifica os agentes individuais do
sistema multiagentes do simulador, cada um representando uma unidade
militar. Estes dados são gerados e processados pelo NetLogo para
questões de deslocamento das unidades, e utilizados pelo Mininet Wi-fi
para avaliar as chances de sucesso ou falha de comunicações. Os
seguintes campos compõem o conjunto Posições.

- node: tipo inteiro. Indica o nó ao qual as informações se referem.

- x: tipo float. Indica a coordenada x da unidade referida no nó.

- y: tipo float. Indica a coordenada y da unidade referida no nó.

- tick: tipo inteiro. Indica o momento (em ticks) dos dados.

- round_id: tipo inteiro. Indica o número do ciclo atual.

### Colinas

O conjunto de dados Colinas identifica o nível de transposição de cada
patch individual do mapa da simulação, sendo um patch uma célula
individual que pode ser ocupada por um agente. Os dados do conjunto
impactam tanto o algoritmo de deslocamento dos agentes no NetLogo, como
o nível de sucesso de entrega do envio de comunicações de rádio entre as
unidades militares no Mininet Wi-fi. Os seguintes campos compõem o
conjunto Colinas.

- nodea: tipo inteiro. Indica um dos nós do par analisado.

- nodeb: tipo inteiro. Indica o outro nó do par analisado.

- hill: tipo inteiro. Indica o nível de interferência entre o par de
  nós.

- tick: tipo inteiro. Indica o momento (em ticks) das informações.

- round_id: tipo inteiro. Indica o número da rodada atual.

## Modelagem de ocorrências de Fogo Amigo {#sec-04.3}

A comunicação é um recurso vital para os soldados. O reconhecimento
visual permite que as unidades militares, dentro de uma distância reta
máxima, identifiquem aliados sem depender de comunicações por rede. No
entanto, além desse limite ou quando na presença de obstáculos visuais,
o uso de communicação por rede é essencial para identificar unidades
amigas.

Situações de fogo amigo ocorrem justamente quando uma unidade militar é
incapaz de identificar outra unidade como aliada e a ataca por engano.
Para monitorar e evitar esses incidentes, a aplicação desenvolvida
utiliza um sistema BFT, um tipo de sistema conhecido por empregar
dispositivos GPS para rastrear e exibir as posições de forças aliadas
(azuis) no campo de batalha. Esse sistema aprimora a consciência
situacional e facilita as comunicações de comando e controle entre
unidades dispersas [@sweeney2008blue; @chevli2006blue].

A eficácia da aplicação BFT depende de uma comunicação de rede robusta
para identificar aliados com precisão. Interrupções na rede,
frequentemente causadas por obstáculos no campo de batalha, como
terrenos elevados, podem gerar dados BFT incompletos ou a total perda da
informação, assim, aumentando o risco de fogo amigo. Equipes militares
geralmente preveem esses obstáculos por meio de missões de
reconhecimento de terreno, característica que reforça a escolha do
algoritmo A\* para a simulação do movimento das unidades. Para este
sistema, a aplicação BFT foi construída como um modulo dentro do
componente *Emulador de Redes*, no caso, o Mininet Wi-fi, sendo acionado
junto com o início da simulação de acordo com o fluxo do diagrama de
sequência da Figura [11](#fig:9.diagrama.bft){reference-type="ref"
reference="fig:9.diagrama.bft"}.

<figure id="fig:9.diagrama.bft" data-latex-placement="ht">
<p><img src="assets/images/fig9.bft.flow.png" alt="image" /> <span
id="fig:9.diagrama.bft" data-label="fig:9.diagrama.bft"></span></p>
<p>Fonte: os autores.</p>
<figcaption>Diagrama de sequência da aplicação BFT.</figcaption>
</figure>

De acordo com o diagrama da aplicação BFT, cada nó $i$, que simula uma
unidade militar, possui uma aplicação cliente-servidor. A cada tick do
ambiente de simulação, o nó $i$ recebe uma mensagem do ambiente
ordenando que avance pelo mapa do cenário. Após receber essa mensagem, o
cliente do nó $i$ envia uma nova mensagem com seus dados de localização
para todos o nó $n$ da população de agentes $P$ próximo a ele, assumindo
que $\exists n \in P | n \neq i \wedge (0 \leq n < P)$. Igualmente, todo
nó $n$ da aplicação que recebe esta nova mensagem a reencaminha para
todo nó $m$ próximo a ele, de forma que
$\exists m \in P | m \neq i \wedge m \neq n \wedge (0 \leq m < P)$.

Em paralelo a isso, todo nó $i$ da simulação que recebe uma mensagem de
outro nó $k$ qualquer em sua aplicação servidor envia para sua interface
MQTT os dados de localização recebidos deste nó $k$, junto com seu
próprio identificador, a informação de data e hora do recebimento dessa
informação e o identificador do tick atual. Ao final, a interface MQTT
do nó $i$ devolve para o ambiente da aplicação os dados de localização
do nó $k$, que utiliza essas informações para atualizar o cenário da
simulação.

O modelo BFT construído assume que, no início da simulação, os agentes
aliados compartilham informações mútuas, permitindo a troca de dados
BFT, que é ilustrada pelas linhas brancas na
Figura [12](#fig:6.simulation.lines){reference-type="ref"
reference="fig:6.simulation.lines"}. No entanto, obstáculos no terreno,
assim como interrupções na rede, podem levar a perda parcial de
comunicação, de forma que apenas um dos agentes conectados possui dados
BFT sobre o outro (ilustrado pela linha laranja ligando os agentes em
questão), ou a perda completa de comunicação (reresentada pela ausência
de linha conectando os agentes).

<figure id="fig:6.simulation.lines" data-latex-placement="!h!t">
<p><img src="assets/images/fig6.simulation.link_lines.png" style="width:50.0%"
alt="image" /> <span id="fig:6.simulation.lines"
data-label="fig:6.simulation.lines"></span></p>
<p>Fonte: os autores.</p>
<figcaption>Monitoramento de comunicação entre unidades
aliadas.</figcaption>
</figure>

Unidades aliadas e inimigas disparam automaticamente contra unidades não
identificadas dentro de seu campo de visão. Além do campo de visão ---
em formato de cone --- a precisão do disparo também considera o alcance
efetivo da arma portada pelo agente. A cada segundo, o MAS escaneia o
cenário de batalha, iterando sobre os agentes e registrando, para cada
um, todas as unidades dentro do seu campo de visão que atendem às
condições de ataque e não se encontram em um estado final da FSM.

Por outro lado, conforme requisitado pelos stakeholders, as unidades
inimigas seguem um comportamento de ataque simplificado: elas nunca
causam fogo amigo e sempre disparam contra unidades aliadas dentro do
seu alcance de precisão.

Isso contrasta com as unidades aliadas, que seguem um processo decisório
mais complexo, detalhado no diagrama de atividades [@OMG2017] da
Figura [13](#fig:7.friendly.fire){reference-type="ref"
reference="fig:7.friendly.fire"}. Esse modelo trata cada agente $a_{i}$
como um possível atacante direcionado a outros agentes. Contudo, tanto
unidades inimigas quanto aliadas estão sujeitas a erros de disparo.

<figure id="fig:7.friendly.fire" data-latex-placement="!h!t">
<p><img src="assets/images/fig7.friendly_fire.png" alt="image" /> <span
id="fig:7.friendly.fire" data-label="fig:7.friendly.fire"></span></p>
<p>Fonte: os autores.</p>
<figcaption>Fluxo de ataque e fogo amigo entre unidades
aliadas.</figcaption>
</figure>

A confirmação de um alvo segue condições específicas --- apresentadas no
diagrama da Figura [13](#fig:7.friendly.fire){reference-type="ref"
reference="fig:7.friendly.fire"} e listadas a seguir. Esse processo é
repetido para cada agente aliado com status válido na simulação,
considerando todas as combinações possíveis entre agentes atacante e
alvo, até a conclusão da simulação.

- Atacante e alvo não se encontram em um estado final.

- A distância em linha reta entre o atacante e o alvo está fora da
  distância de "reconhecimento visual", exigindo o feedback da aplicação
  BFT para identificação mútua.

- O atacante não recebeu da rede BFT comunicação de retorno do alvo.

- O atacante decidiu atirar e tem o alvo dentro do campo de visão e
  alcance da arma $(0<d \leq \text{min}(0.8, precisao(atq)))$.

- A unidade atacante respeita um atraso de, pelo menos, 6 segundos entre
  a visualização do alvo e o disparo. Tempo requisitado pelos
  stakeholders com base nos tempos médios para a tomada de decisão de
  disparo por soldados.

- A arma está pronta para disparar, sem restrições em vigor, como o
  tempo entre os disparos ou necessidade de recarga.

Quando todas as condições são atendidas, o atacante realiza o disparo.
Se tanto o atacante quanto o alvo forem unidades aliadas,
independentemente de o tiro acertar, o algoritmo incrementa a contagem
de "fogo amigo". Caso o alvo seja inimigo, a contagem de "inimigo
atacado" é aumentada.

## Blue Force Tracking {#subsec:5.3}

Para analisar o desempenho do simulador, conforme descrito na Seção 4.3,
foi criada a aplicação **Blue Force Tracking (BFT)**. Essa aplicação é
executada em cada nó controlado pelo **Emulador de Rede**. Em cada nó
rodam os módulos `Client` e `Server` do BFT, que utilizam a pilha de
rede isolada do respectivo *namespace* --- ou seja, o conjunto de
protocolos e interfaces de rede independentes daquele nó --- para se
comunicarem. Nesta seção, são apresentados os principais componentes do
BFT, ilustrados na Figura [14](#fig:a.diagrama.bft){reference-type="ref"
reference="fig:a.diagrama.bft"}.

<figure id="fig:a.diagrama.bft" data-latex-placement="ht">
<p><img src="assets/images/fig.a.BFT.application.png" alt="image" /> <span
id="fig:a.diagrama.bft" data-label="fig:a.diagrama.bft"></span></p>
<p>Fonte: os autores.</p>
<figcaption>Diagrama de Componentes da Aplicação Blue Force
Tracking.</figcaption>
</figure>

### MQTT Interface

O componente MQTT interface fornece uma interface para a troca de dados
entre o orquestrador da simulação e a aplicação BFT que executa dentro
de cada agente simulados do cenário de campo de batalha.

### Client

O componente Client define a interface de envio de dados da aplicação
através de pacotes UDP.

### Server

O componente Server define a inferface para recebimento de dados da
aplicação oriundo de outros agentes, pacotes UDP.

### Configuration

O componente Configuration contém informações de configuração da
aplicação, como o as portas utilizadas pelos componentes Client e
Server, as portas utilizadas para o MQTT, entre outras configuraçõs de
rede. Também no componente Configuration é especificado o ttl, um
inteiro que define o número de vezes que uma mensagem recebida por um
agente será retransmitida para os demais agentes no seu alcance de
comunicação. Este parâmetro visa ampliar a área de cobertura do BFT.

### Data

O componente Data define o modelo de dados da aplicação BFT, composto
por:

- source_ip: endereço de IP de origem;

- receiver_ip: endereço IP de destino;

- tick_sent: inteiro indicando o momento (em ticks) a que as informações
  se referem.

- position_x: float indicando a coordenada $x$ do agente.

- position_y: float indicando a coordenada $y$ do agente.

O diagrama representado na Figura [15](#fig:b.bft){reference-type="ref"
reference="fig:b.bft"} apresenta a comunicação da aplicação BFT com o
simulador S2C2 através das interfaces AppController e IDatabase, também
representadas na
Figura [1](#fig:1.arquitetura.s2c2){reference-type="ref"
reference="fig:1.arquitetura.s2c2"}.

<figure id="fig:b.bft" data-latex-placement="ht">
<p><img src="assets/images/fig.b.BFT.Database.Report.png" alt="image" /> <span
id="fig:b.bft" data-label="fig:b.bft"></span></p>
<p>Fonte: os autores.</p>
<figcaption>Interação do BFT com Simulador S2C2.</figcaption>
</figure>

Durante a simulação, a aplicação BFT troca mensagens de controle com o
orquestrador da simulação, o EmuSim, e armazena as mensagems trocadas no
banco de dados através da interface IDataBase. No final da simulação, a
aplicação BFT gera um relatório contendo as estatísticas de fogo amigo
do cenário de simulação executado, de modo a validar o impacto das
alterações no cenário de simulação no contexto do fogo amigo.

# CENÁRIOS DE TESTES E RESULTADOS {#sec-06}

Ao longo do projeto, diferentes cenários de simulação foram
desenvolvidos para representar as diversas necessidades apresentadas
pelas Forças Armadas. A
Figura [16](#fig:10b.simulation){reference-type="ref"
reference="fig:10b.simulation"} e a
Figura [17](#fig:10c.simulation){reference-type="ref"
reference="fig:10c.simulation"} mostram os computadores do laboratório
S2C2 executando alguns desses diferentes cenários.

<figure id="fig:10b.simulation" data-latex-placement="!ht">
<p><img src="assets/images/simulacoes-no-lab.png" style="width:80.0%"
alt="image" /> <span id="fig:10b.simulation"
data-label="fig:10b.simulation"></span></p>
<p>Fonte: os autores.</p>
<figcaption>Diferentes cenários de simulação em execução no laboratório
S2C2</figcaption>
</figure>

<figure id="fig:10c.simulation" data-latex-placement="!ht">
<p><img src="assets/images/fig.scenario4.jpeg" style="width:55.0%" alt="image" />
<span id="fig:10c.simulation"
data-label="fig:10c.simulation"></span></p>
<p>Fonte: os autores.</p>
<figcaption>Simulação de ataque executando no simulador.</figcaption>
</figure>

A seguir, são detalhados dois cenários que melhor ilustram o progresso
obtido no desenvolvimento das simulações. Ambos compartilham as mesmas
características de mapa, tropas aliadas, parâmetros de comunicação e
condições climáticas, diferindo apenas quanto à presença ou ausência de
inimigos. Na sequência, é apresentado um comparativo entre as duas
propostas, a fim de compreender melhor como essa diferença afeta as
ocorrências de fogo amigo entre unidades aliadas.

## Cenário de Simulação BFT 01: Ausência de inimigos

O cenário de simulação analisado é apresentado na
Figura [18](#fig:10.simulation){reference-type="ref"
reference="fig:10.simulation"}. Esse cenário consiste em 21 unidades
aliadas de soldados a pé, divididas em três grupos de combate.

<figure id="fig:10.simulation" data-latex-placement="!ht">
<p><img src="assets/images/fig.scenario3.png" style="width:55.0%" alt="image" />
<span id="fig:10.simulation" data-label="fig:10.simulation"></span></p>
<p>Fonte: os autores.</p>
<figcaption>Simulação de ataque sob análise.</figcaption>
</figure>

Para alcançar resultados estatisticamente consistentes entre as
configurações, foram realizadas 43 execuções de simulação por intervalo.
Este experimento foi conduzido com seis durações distintas de
comunicação --- 30, 60, 90, 120, 150 e 180 *ticks* (representando
aproximadamente um segundo do mundo real por *tick*) --- a fim de
explorar as complexidades da comunicação. Além disso, o período de
validade de cada mensagem foi definido como o dobro do respectivo
intervalo de comunicação (por exemplo, para um intervalo de comunicação
de 60 segundos, uma mensagem recebida por um aliado permanece válida por
120 segundos).

Para isolar melhor os resultados de fogo amigo do BFT, não foram
incluídas unidades inimigas para a execução desta primeira análise. Pelo
mesmo motivo, a simulação foi executada sob condições climáticas claras,
sem considerar falhas de equipamento ou recursos de guerra eletrônica.

As unidades aliadas navegam continuamente em direção aos destinos
designados, passando pelos pontos intermediários indicados pela cor do
círculo que destaca cada unidade na
Figura [18](#fig:10.simulation){reference-type="ref"
reference="fig:10.simulation"}, até alcançar o destino destacado em
amarelo. À medida que as tropas se deslocam pelo campo, as forças
aliadas podem passar pelas zonas de linha de visão, alcance de
comunicação e alcance de disparo. Elas classificam como aliados tanto as
unidades corretamente identificadas dentro de seu campo de visão quanto
aquelas que recebem com sucesso as informações transmitidas dentro do
período de validade. Qualquer unidade que não atenda a um desses
critérios é classificada como inimiga --- mesmo que o cenário simulado
não inclua entidades inimigas.

Entretanto, essa classificação permanece suscetível a mudanças
topológicas que podem afetar o campo de visão ou o alcance de ataque.
Tais mudanças também podem influenciar a taxa de sucesso na entrega de
mensagens pela rede, fator que, juntamente com a presença de unidades
inimigas e condições climáticas diversas, será abordado em futuros
cenários de simulação, a fim de reduzir a diferença entre simulação e
realidade.

<figure id="fig:11.ff_progression" data-latex-placement="h!t!">
<img src="assets/images/fig10.simulation_data_chart_v5.png"
style="width:100.0%" />
<figcaption>Evolução da taxa de fogo amigo com a melhoria na comunicação
do BFT.</figcaption>
</figure>

Os resultados dos conjuntos de simulação nos seis diferentes intervalos
de comunicação são mostrados no gráfico da
Figura [19](#fig:11.ff_progression){reference-type="ref"
reference="fig:11.ff_progression"}. Dependendo da gravidade do ataque,
as unidades atingidas podem transitar entre estados anteriormente
definidos na FSM da Figura [10](#fig:5.fsm){reference-type="ref"
reference="fig:5.fsm"}, como por exemplo, como "Ferido" ou "Assistência
médica urgente".

Como mostra o gráfico, os dados do BFT contribuem significativamente
para a redução do fogo amigo em comparação com cenários sem seu uso. A
comunicação em intervalos mais curtos melhora ainda mais a prevenção de
fogo amigo em relação a intervalos mais longos. No entanto, embora o BFT
seja eficaz para intervalos curtos, o aumento do intervalo não produz
uma tendência linear, como evidenciado pelos intervalos de comunicação
T120, T150 e T180 na
Figura [19](#fig:11.ff_progression){reference-type="ref"
reference="fig:11.ff_progression"}.

Apesar da tendência geral de que intervalos maiores apresentem maior
incidência média de fogo amigo, o intervalo T150 registrou uma média
inferior à do intervalo imediatamente anterior (T120). Supõe-se que esse
resultado tenha ocorrido porque, dentro do cenário de movimentação
mostrado na Figura [18](#fig:10.simulation){reference-type="ref"
reference="fig:10.simulation"}, um pequeno número de unidades aliadas
provavelmente atingiu o limite de comunicação com outras unidades entre
os *ticks* 0 e 119.

Por outro lado, a partir de 150 *ticks*, houve um número maior de
unidades aliadas dentro do alcance de comunicação, resultando em um
aumento na troca de mensagens e, consequentemente, em uma redução nos
incidentes de fogo amigo, uma vez que mais unidades aliadas
estabeleceram comunicações bem-sucedidas antes de entrarem no raio de
tiro ou de visão de outras unidades.

## Cenário de Simulação BFT 02: Presença de inimigos

## Cenário de Simulação GCB

O cenário de simulação com o GCB foi executado em conjunto com o cenário
de simulação BFT. Essa integração teve como objetivo permitir a troca de
mensagens entre os sistemas, de modo que os dados fornecidos pela
aplicação pudessem apoiar a tomada de decisão do comandante.

Nesse contexto, o GCB atua em complemento às características já
descritas para um cenário de simulação executado na aplicação. Assim,
após a seleção do cenário, a parametrização de suas características e o
início da execução, as estações GCB são inicializadas nos nós simulados.

As Figuras [20](#fig:S2C2-CGB01){reference-type="ref"
reference="fig:S2C2-CGB01"}, [21](#fig:S2C2-CGB02){reference-type="ref"
reference="fig:S2C2-CGB02"} e [22](#fig:S2C2-CGB03){reference-type="ref"
reference="fig:S2C2-CGB03"} ilustram essa integração. A Figura
[20](#fig:S2C2-CGB01){reference-type="ref" reference="fig:S2C2-CGB01"}
mostra o cenário de simulação em execução, com os respectivos agentes em
azul, os logs das estações GCB exibidos no prompt de comando e o mapa da
aplicação GCB em operação. Neste cenário, cinco estações são simuladas.

<figure id="fig:S2C2-CGB01" data-latex-placement="!ht">
<img src="assets/images/S2C2-GCB-02.png" />
<figcaption>Inicialização das estações GCB com o cenário de
simulação.</figcaption>
</figure>

A Figura [21](#fig:S2C2-CGB02){reference-type="ref"
reference="fig:S2C2-CGB02"} apresenta a inserção de um novo nó no campo
de batalha, que é automaticamente identificado como inimigo pelos demais
devido à ausência de informações sobre ele.

<figure id="fig:S2C2-CGB02" data-latex-placement="!ht">
<img src="assets/images/S2C2-GCB-04.png" />
<figcaption>Inserção de novo nó no campo de batalha e resposta dos
outros nós da simulação.</figcaption>
</figure>

O sucesso da comunicação faz com que o novo elemento seja reconhecido
como parte do mesmo grupo de batalha, sendo inserido de forma apropriada
na hierarquia de comando, conforme ilustrado na Figura
[22](#fig:S2C2-CGB03){reference-type="ref" reference="fig:S2C2-CGB03"}.

<figure id="fig:S2C2-CGB03" data-latex-placement="!ht">
<img src="assets/images/S2C2-GCB-06.png" />
<figcaption>Troca de mensagens bem-sucedida entre os nós.</figcaption>
</figure>

## Cenário de Simulação BRAVO (Extensão Projeto S2C2)

# PUBLICAÇÕES GERADAS {#sec-07}

- CARVALHO, Leonardo Filipe Batista Silva de; DE SOUZA, Vitor Simon;
  BONATTO, Alisson Nunes; PEREZ, Thales Junqueira Albergaria Moraes; DE
  FREITAS, Edison Pignaton; BARONE, Dante Augusto Couto; ZIBETTI,
  Guilherme Rotth; DOS ANJOS, Julio C. S.; DE ARAUJO FERNANDES, Ricardo
  Queiroz. Multi-Agent Systems Modeling of Command and Control Systems:
  A Metrics-Driven Approach to Simulator Evaluation and Co-Simulation.
  Journal of Simulation, \[S. l.\], no prelo, 2025. DOI:
  10.1080/17477778.2025.2584542.

- GOMES, João Eduardo Costa; EHLERT, Ricardo Rodrigues; BOESCHE, Rodrigo
  Murillo; SANTOS DE LIMA, Vinicius; STOCCHERO, Jorgito Matiuzzi;
  BARONE, Dante Augusto Couto; WICKBOLDT, Juliano Araujo; FREITAS,
  Edison Pignaton de; ANJOS, Julio C. S. dos; ARAUJO FERNANDES, Ricardo
  Queiroz de. Surveying emerging network approaches for military command
  and control systems. ACM Computing Surveys, v. 56, p. 1--38, 2024.

- CARVALHO, Leonardo Filipe Batista Silva de; SOUZA, Vitor Simon de;
  BONATTO, Alisson Nunes; PEREZ, Thales Junqueira Albergaria Moraes;
  FREITAS, Edison Pignaton de; BARONE, Dante Augusto Couto; ZIBETTI,
  Guilherme Rotth; ANJOS, Julio C. S. dos; ARAUJO FERNANDES, Ricardo
  Queiroz de. A multi-agent system approach for Blue Force Tracking C2
  application modeling. In: Lecture Notes in Networks and Systems. 1.
  ed. Cham: Springer Nature Switzerland, 2024. v. 2, p. 161--181.

- BARONE, Dante A. C. et al. Integrated multi-agent system simulator and
  network emulator framework to realistically exercise networked command
  and control application scenarios. In: INTERNATIONAL CONFERENCE ON
  SIMULATION AND MODELING METHODOLOGIES, TECHNOLOGIES AND APPLICATIONS,
  2023, Cham. Proceedings\... Cham: Springer Nature
  Switzerland, 2023. p. 9--28.

- BARONE, Dante Augusto Couto; WICKBOLDT, Juliano Araujo; CAVALCANTI,
  Maria Cristina Rosa; MOURA, David; TESOLIN, Julio Cesar Costa; DEMORI,
  André Marques; ANJOS, Julio C. dos; CARVALHO, Leonardo Filipe Batista
  Silva de; GOMES, João Eduardo Costa; FREITAS, Edison Pignaton de.
  Integrating a multi-agent system simulator and a network emulator to
  realistically exercise military network scenarios. In: INTERNATIONAL
  CONFERENCE ON SIMULATION AND MODELING METHODOLOGIES, TECHNOLOGIES AND
  APPLICATIONS (SIMULTECH 2023), 13., 2023, Roma. Proceedings\...
  Roma, 2023. p. 194--201.

- DEMORI, André; TESOLIN, Julio; MOURA, David; GOMES, João; PEDROSO,
  Gabriel; CARVALHO, Leonardo Silva de; FREITAS, Edison Pignaton de;
  CAVALCANTI, Maria. A semantic web approach for military operation
  scenarios development for simulation. In: INTERNATIONAL CONFERENCE ON
  DATA SCIENCE, TECHNOLOGY AND APPLICATIONS, 12., 2023, Rome.
  Proceedings\... Rome, 2023. p. 390.

# CONSIDERAÇÕES FINAIS {#sec-08}

Este relatório apresenta o simulador desenvolvido no projeto S2C2,
visando melhorar o processo de desenvolvimento de sistema e aplicativos
de C2. O simulador desenvolvido oferece um ambiente que possibilita a
execução de exercícios militares a partir da abordagem de co-simulação,
permitindo testar protocolos de comunicação e sistemas táticos em
ambientes simulados.

Além do simulador apresentado, também foram desenvolvidas as aplicações
indicadas na Seção [5](#sec-05){reference-type="ref"
reference="sec:05"}, todas, visando a diminuição dos incidentes de
fratricídio nos cenários de campo de batalha. Essas aplicações,
constituiem os recursos que permitiram a restagem das *features*
implementadas no simulador, de modo a demonstrar sua aplicabilidade ao
executar aplicações que fornecem métricas relevantes para o contexto
militar. Portanto, destacam-se como principais contribuições deste
trabalho:

- Simulador S2C2: Um simulador baseado no paradigma de co-simulação que
  permite a integração com sistemas táticos para a realização de
  exercícios militares em ambientes simulados;

- Adequação à doutrina: Conforme exposto na
  subseção [4.2](#sec-04.2){reference-type="ref" reference="sec:04.2"} e
  [5.2](#sec-04.3){reference-type="ref" reference="sec:04.3"}, o sistema
  desenvolvido buscou representar com maior fidelidade a doutrina
  militar, desenvolvendo e implementando comportamentos relacionados à
  exercícios e operações militares;

- Validação com métricas de negócio: De modo a observar o impacto que
  diferentes parâmetros de configuração possam ter no sucesso da
  operação, foi desenvolvida a aplicação BFT, que fornece ao agente
  simulado ciência dos aliados no campo de batalha, visando que a tomada
  de decisão de tiro esteja apoiada em aplicações de C2.

Como trabalhos futuros, pretende-se expandir as capacidades do simulador
para interagir com a Família de Aplicativos de Comando e Controle da
Força Terrestre (FAC2FTer), bem como criar cenários de simulação para
apoiar as equipes de desenvolvimento de aplicativos de C2, com objetivo
de melhorar os processos de desenvolvimento, testes e homologação da
FAC2FTer e demais aplicativos C2.

Além disso, pretende-se adaptar a aplicação **Blue Force Tracking
(BFT)** --- já desenvolvida para análise de desempenho do simulador,
conforme descrito na Seção [5.3](#subsec:5.3){reference-type="ref"
reference="subsec:5.3"} --- para ser executada em nós-contêiner
controlados pelo **Emulador de Rede**. Nessa proposta, cada contêiner
docker deverá instanciar os módulos `Client` e `Server` do BFT,
utilizando a pilha de rede isolada do respectivo *namespace* --- ou
seja, o conjunto de protocolos e interfaces de rede independentes
daquele contêiner --- para realizar a comunicação.

Por fim, o Apêndice [\[appen:a\]](#appen:a){reference-type="ref"
reference="appen:a"} contém o manual de operação do simulador S2C2, e o
Apêndice [\[appen:b\]](#appen:b){reference-type="ref"
reference="appen:b"} apresenta o relatório de um cenário de simulação
executando o BFT.
