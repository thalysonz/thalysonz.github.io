---
title: "Pix Pirate Analysis"
date: 2023-02-24
tags: ["mobile", "malware", "en"]
author: "Me"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "pix pirate analysis"
disableHLJS: true # to disable highlightjs
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: true
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: true
UseHugoToc: true
---

### Resumo

1. **Nome do pacote:** com.dam.ninth

2. **SHA256 hash:** 911fa473f75d48e28384023b4a008d6d0877ad0fa58f9ebf38da1b02a1481314

![logo](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*JlxcoyQTN0YkYLc-.png)

Pixpirate é uma família de malware mobile que começou a ficar mais famosa entre o primeiro e segundo semestre de 2022. Ele faz uso das funções de acessibilidade do Android para obter informações do dispositivo, mapeando teclas e processos abertos. Entre as diversas funções desse novo malware, destacam-se as seguintes:

- Observar/interagir com SMS;
- Observar/interagir com notificações;
- Evitar desinstalação;
- Observar aplicativos em execução no dispositivo;
- Uso das funções de acessibilidade;
- Etc.

### Inicio

Vamos comecar pelo arquivo AndroidManifest.xml que é obrigatório em todos os aplicativos Android. Nele são descritas informações básicas do aplicativo, como nome do pacote, versão do aplicativo, permissões necessárias, entre outras informações.

A documentação oficial de desenvolvimento sobre o Android Manifest pode ser encontrada aqui, mas resumindo, neste arquivo conseguimos ter uma noção sobre como o aplicativo vai funcionar. Podemos visualizar quais eventos ele está escutando e como responde a eles, quais serviços ele roda em segundo plano e também observar se existe alguma classe que ele chamou mas não está sendo carregada na análise estática.

Para quem não conhece muito de desenvolvimento Android, pode ter ficado confuso como tudo isso acontece. Vou tentar explicar de uma maneira um pouco mais clara.

No sistema Android, temos o modelo pub/sub (publicação/assinatura) para comunicação, onde é possível que aplicativos comuniquem informações uns com os outros sem que precisem estar em execução simultaneamente. Isso é possível por causa de um recurso chamado Broadcasts, que são basicamente mensagens enviadas entre aplicativos. Com isso, um aplicativo pode se registrar, usando um elemento chamado Receiver, definido no AndroidManifest.xml, para receber um broadcast e definir tarefas para serem executadas quando ouvirem esse broadcast.

No AndroidManifest, também temos os Services e os Providers, onde o primeiro é responsável por executar tarefas em segundo plano e o outro por permitir que o aplicativo compartilhe dados com outros.

Observando tudo isso nesse arquivo conseguimos ter uma nocao inicial sobre como o aplicativo vai funcionar ao ser instalado e ter uma nocao do fluxo de codigo dele.

### Permissões:

![perm](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*atwm-8GCK-4S675O.png)

Aqui temos a lista de permissoes que aplicativo pede ao usuario, abaixo listei algumas que nao sao tao comuns e podem ser prejudiciais ao usuario.

Aqui um link da documentacao do android a respeito das permissoes: https://developer.android.com/guide/topics/permissions/overview

1. QUERY_ALL_PACKAGES:
permite que um aplicativo obtenha informações sobre todos os outros aplicativos instalados no dispositivo, incluindo detalhes como o nome do pacote, a versão e a data de instalação
Essa permissão é considerada uma permissão perigosa e, portanto, deve ser usada com cautela. Isso ocorre porque, se um aplicativo mal-intencionado obtiver essa permissão, poderá coletar informações pessoais ou confidenciais dos usuários, como as informações de login salvas em outros aplicativos. Por esse motivo, os usuários devem tomar cuidado ao conceder essa permissão a um aplicativo.
2. READ_INSTALL_SESSIONS:
permite que um aplicativo leia as informações sobre sessões de instalação de outros aplicativos.
Uma sessão de instalação é criada quando um usuário inicia o processo de instalação de um aplicativo na Google Play Store ou por meio de outros meios, como o download de um arquivo de instalação APK. Essa permissão permite que um aplicativo obtenha informações sobre as sessões de instalação em andamento, incluindo o estado da instalação, como progresso e status de conclusão.
3. READ_SYNC_SETTINGS:
permite que um aplicativo leia as configurações de sincronização da conta do usuário. A sincronização é o processo pelo qual os dados da conta do usuário são transferidos entre o dispositivo Android e os servidores da conta na nuvem, para garantir que as informações do usuário estejam atualizadas e disponíveis em todos os dispositivos.
é importante notar que, se um aplicativo malicioso obtiver essa permissão, ele poderá ler informações confidenciais da conta do usuário, como senhas e informações pessoais. Por esse motivo, os usuários devem conceder essa permissão apenas a aplicativos confiáveis.
4. AUTHENTICATE_ACCOUNTS:
permite que um aplicativo solicite ao usuário que autentique uma conta. Ela é usada por aplicativos que precisam de acesso a uma conta do usuário para fornecer recursos ou funcionalidades personalizadas.
Essa permissão é considerada uma permissão perigosa e, portanto, deve ser usada com cuidado. Isso ocorre porque, se um aplicativo mal-intencionado obtiver essa permissão, poderá solicitar ao usuário que autentique uma conta falsa, com o objetivo de roubar informações pessoais ou destruir dados.
Os aplicativos que desejam acessar essa permissão devem declará-la explicitamente em seu manifesto e solicitar a permissão do usuário em tempo de execução. Além disso, a partir do Android 6.0 (nível de API 23), essa permissão é protegida por um modelo de permissão mais restritivo, chamado de “permissão de tempo de execução”. Isso significa que os usuários precisam conceder a permissão explicitamente em tempo de execução para que o aplicativo possa acessá-la.
5. WRITE_SYNC_SETTINGS:
permite que um aplicativo altere as configurações de sincronização da conta do usuário. A sincronização é o processo pelo qual os dados da conta do usuário são transferidos entre o dispositivo Android e os servidores da conta na nuvem, para garantir que as informações do usuário estejam atualizadas e disponíveis em todos os dispositivos.
Essa permissão é considerada uma permissão normal e não é particularmente perigosa por si só. Ela é usada por aplicativos que precisam alterar as configurações de sincronização para permitir que o usuário controle as configurações de sincronização da conta.
No entanto, é importante notar que, se um aplicativo malicioso obtiver essa permissão, ele poderá alterar as configurações de sincronização sem o conhecimento do usuário, o que pode resultar em perda de dados ou outras consequências indesejadas. Por esse motivo, os usuários devem conceder essa permissão apenas a aplicativos confiáveis.

### Acessibilidade(Arquivo XML):

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*aKMomWmY_J7SyJ8M.png)

Explicando um pouco sobre, aqui esta sendo definido um servico a respeito das questoes de acessibilidade como a permissao BIND_ACCESSIBILITY_SERVICE, e o arquivo xml com as opcoes de acessibildiade que a aplicacao vai usar.

Segundo a documentacao do android, usando um arquivo de configuracao xml conseguimos definir uma gama maior de opcoes de acessibilidade para a aplicacao. Assim o malware utiliza disso para definir o que vai ser usado, abaixo expliquei tudo que foi definido no xml config do malware.

1. android:description: Define a descrição do serviço de acessibilidade. Geralmente é usado para informar aos usuários sobre as funções e permissões do serviço.

2. android:accessibilityFlags: Define as configurações adicionais para o serviço de acessibilidade. flagRetrieveInteractiveWindows indica que o serviço pode recuperar janelas interativas, flagRequestFilterKeyEvents indica que o serviço pode filtrar eventos de teclado, flagReportViewIds indica que o serviço pode relatar IDs de visualização, flagRequestEnhancedWebAccessibility indica que o serviço pode solicitar recursos aprimorados de acessibilidade da Web e flagIncludeNotImportantViews indica que o serviço pode incluir visualizações não importantes.

### Analise estatica #1

O malware possui uma forte ofuscação, o que dificulta uma análise inicial e pode ser assustador à primeira vista. No entanto, seguindo alguns passos já mencionados, é possível ter sucesso na compreensão do fluxo do aplicativo.

![](https://miro.medium.com/v2/resize:fit:1348/format:webp/0*8VC6ogUzljSx6OW9.png)

Quase todas as strings importantes do malware foram modificadas utilizando uma operação XOR, mas é possível recuperar facilmente o valor original usando o “decode” que a aplicação chama e executando-o em alguma plataforma adequada.

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*mzqKoEF2sCM0yh9S.png)

Abaixo temos a classe responsável por inserir algumas das classes da aplicação na engine JS. Ao lado, comentei o significado de cada string.

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*Bp5vzHoQTpVEMRrp.png)

Abaixo temos uma interface responsável por alguns endpoints do C2, que posteriormente o AutoJS irá interagir.

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*F1nZGpEgg6Ui9-L0.png)

Boa parte do código é utilizado para descompactar o arquivo project.zip, que contém o projeto JavaScript do AutoJS.

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*duC0jbhpHC6IjuB8.png)

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*JayEEG9JIZjPE9uO.png)

### Analise estatica #2 (Javascript)

Utilizando o AutoJS, o malware consegue realizar uma série de atividades de forma furtiva, como observar as aplicações que o usuário utiliza (visualizando aplicativos que são abertos, aplicativos que estão abertos no modo janela, aplicativo na tela atual), observar e interagir com SMS e notificações que o aparelho recebe e verificar o status da SafeNet.

Abaixo está uma explicação sobre o que é o AutoJS:

1. O AutoJS é uma ferramenta que permite a automação de tarefas em dispositivos Android. Ela foi projetada para permitir que os usuários criem scripts personalizados que podem ser executados em seus dispositivos para automatizar várias tarefas.

2. Com o AutoJS, é possível escrever scripts que automatizam interações com aplicativos Android, como abrir e fechar aplicativos, preencher formulários, clicar em botões e até mesmo interagir com o sistema operacional do dispositivo. Esses scripts podem ser executados automaticamente em um horário agendado ou por meio de gatilhos específicos, como quando o dispositivo é conectado a uma rede Wi-Fi específica.

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*WpnuYlqO_k_JOIW4.png)

Na imagem acima, temos a ordem de funcionamento do malware, retirado do arquivo main.js.

O script começa verificando as permissões de acessibilidade, algo obrigatório devido às funções do malware. Após isso, são chamados alguns eventos que o malware utiliza para verificar as aplicações e eventos em execução.

Em seguida, o malware executa algumas funções para verificar permissões em geral e também acessar eventos do dispositivo, ou fechar aplicativos de acordo com uma lista de strings.

![](https://miro.medium.com/v2/resize:fit:816/format:webp/0*zL2wzP66bznuNm2A.png)

Também podemos observar uma função chamada antiRemove, responsável por evitar remoções e paradas na execução do malware.

As duas últimas funções, startHijarkSms e startBank, o malware utiliza para acessar e interagir com o dispositivo permitem, desde acessar e excluir SMS e notificações até interagir com as aplicações bancárias.

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*-EEjZQRC_E3mm7eL.png)

Aqui temos a função responsável pelo SMS. Podemos observar a chamada ao serviço de acessibilidade logo no início do arquivo.

![](https://miro.medium.com/v2/resize:fit:492/format:webp/0*zQvIkebBZDJYnaoe.png)

Este arquivo é responsável por observar, acessar, filtrar e excluir SMS do dispositivo da vítima.

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*ZtKMaWqfj5saXwn-.png)

A função chamada “annihilateSms” é a responsável por excluir os SMS.

O malware suporta vários bancos, como podemos ver na lista de arquivos abaixo. Ao analisarmos um desses arquivos, podemos entender um pouco mais sobre como o malware atua.

**Arquivo: BrazilBank.js**

![](https://miro.medium.com/v2/resize:fit:770/format:webp/0*5hSn0zlau93Nd0W4.png)

![](https://miro.medium.com/v2/resize:fit:1320/format:webp/0*hP8TCbz_z3WvFxNP.png)

Aqui podemos ver que no início do arquivo é definido o banco que a vítima está usando.

A seguir, vou listar algumas funções com o comportamento do malware.

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*U2MVJsx9v3aqM5-b.png)

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*rldvDb0ng1rghGXH.png)

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*UImyBk1_d-fGPcpN.png)

1. GleanBalance: responsável por coletar o saldo da vítima, utilizando um filtro disponível nas funções do AutoJS, onde filtra pelo valor de “Saldo disponível”.

2. Transfer: chama a função sendBalance, responsável por iniciar o processo de PIX na interface do usuário.”

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*F3hX3bKD_kklhR9K.png)

Aqui, o malware faz algumas verificações antes de prosseguir. Primeiro, ele verifica se o valor do saldo é válido.

![](https://miro.medium.com/v2/resize:fit:1262/format:webp/0*OX6G8ov4ANl25f0L.png)

Após isso, é feita uma verificação sobre o saldo da vítima, onde o malware verifica se o saldo é maior que o Teto (provavelmente o malware estabeleceu um limite máximo para as transações PIX a fim de evitar bloqueios por valores muito altos), sendo o valor do Teto definido como 4777.

Em seguida, podemos verificar que o malware multiplica o valor do saldo por 0.95, possivelmente uma outra medida de segurança para evitar bloqueios nas transações PIX.

![](https://miro.medium.com/v2/resize:fit:1070/format:webp/0*Km4lIDBZYvzmT5oi.png)

Após isso, o malware interage com a classe C2C, que foi mencionada na importação no código nativo da aplicação.

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*Gjm7WIH33e_qLkoI.png)

Essa interação é responsável por salvar o valor do saldo no servidor C2 do malware. Essa será a requisição que o malware executará, passando os valores em formato JSON.

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*O5QWhSYjZHM4f-dk.png)

A seguir, o malware realiza outra interação com a classe C2C. Nessa requisição, o malware envia o valor do saldo da conta da vítima, o nome do banco da vítima e um ID associado ao dispositivo da vítima.

![](https://miro.medium.com/v2/resize:fit:1376/format:webp/0*BbyLcZnj5esTRwec.png)

![](https://miro.medium.com/v2/resize:fit:924/format:webp/0*_RONnyfjPkWFM0Wj.png)

Além de tudo isso, o malware possui um arquivo para armazenar as credenciais do usuário, como mencionado em uma das imagens acima.

![](https://miro.medium.com/v2/resize:fit:1302/format:webp/0*weB_8_i8QNwZn9w9.png)

Novamente fazendo uso da acessibilidade o malware filtra os eventos de acordo com cada banco e inicia o processo de salvar as credenciais do usuario.

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*tTWmSwaLnzVWJoWU.png)

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*kQHbe8dDl5L3GkJF.png)

Na versão mais recente do malware foi implementada uma nova função para WebSocket, porém ainda não parei para analisar bem.

![](https://miro.medium.com/v2/resize:fit:1194/format:webp/0*vypF0mfjueIUcqIg.png)

### Credenciais HardCoded

1. **user:logy password:qwertyuiop**
2. **user:logz password:4ddc69691**

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*dhq15DD5wmqowIHf.png)

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*_gzabkIUCx6TXG9x.png)

![](https://miro.medium.com/v2/resize:fit:712/format:webp/0*_oKD92_UOJrY8FzN.png)

### Servidores

Analisando as requisições do malware, foi possível obter o servidor de atualizações que é utilizado para obter a versão mais recente do código JavaScript.

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*JcmyForOONlFzoIK.png)

O endereço “ https://down883[.]oss-us-east-1[.]aliyuncs[.]com" é utilizado para o versionamento do código JavaScript. Analisando essas versões, podemos ver que o malware está em constante atualização.

![](https://miro.medium.com/v2/resize:fit:1398/format:webp/0*mX_1h9ygeSFnozxn.png)

Sendo a mais recente de 22 de fevereiro de 2023, a mais antiga que consegui acesso foi de 25 de novembro de 2022. O malware também utiliza alguns servidores Elasticsearch para gravar alguns logs de uso de cada dispositivo.

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*xJGVedhVndiTbNVs.png)

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*VQt08XZHKD0uRV8A.png)

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*5Sz38Vjv3buJQxJ3.png)

* camera[.]applebalanyou[.]com
* orange[.]applebalanyou[.]com
* wc[.]werichcash[.]com

Servidor C2, utilizado para obter informações de pagamentos, salvar dados e interagir com o dispositivo infectado pelo malware.

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*JP-DVnh99j9v2Xzr.png)

O endereço desse servidor é atualizado dinamicamente pela aplicação, que recebe o endereço e salva no “prefs” do dispositivo junto com algumas outras informações. Então, esse valor sempre tem alguma atualização.

Abaixo estão algumas versões que já foram usadas:

Também foi possível obter acesso aos dados de transações salvos no C2. Foi observada uma quantidade expressiva de transações salvas, um total de 33.127 até a data de hoje.

![](https://miro.medium.com/v2/resize:fit:1144/format:webp/0*F6zVpj8ceZVXIEwf.png)

Links

1. Malware: https://bazaar.abuse.ch/sample/911fa473f75d48e28384023b4a008d6d0877ad0fa58f9ebf38da1b02a1481314/
2. JADX: https://github.com/skylot/jadx