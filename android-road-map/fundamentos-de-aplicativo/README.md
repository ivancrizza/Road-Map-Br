---
layout:
  title:
    visible: true
  description:
    visible: true
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: false
---

# 📲 Fundamentos de Aplicativo

Os apps Android podem ser escritos usando-se [Kotlin](linguagens/kotlin.md), [Java](linguagens/java.md) e linguagens [C++](linguagens/c++-c.md). As ferramentas do Android SDK compilam o código em conjunto com todos os arquivos de dados e recursos em um APK, um _pacote Android_, que é um arquivo de sufixo `.apk`. Os arquivos de[ APK](apk.md) contêm todo o conteúdo de um app Android e são os arquivos que os dispositivos desenvolvidos para Android usam para instalar o aplicativo.

Cada app Android é ativado na própria sandbox de segurança, protegido pelos seguintes recursos de segurança do Android:

* O sistema operacional Android é um sistema Linux multiusuário em que cada aplicativo é um usuário diferente.
* Por padrão, o sistema atribui a cada aplicativo um código de usuário do Linux exclusivo (o código é usado somente pelo sistema e é desconhecido para o aplicativo). O sistema define permissões para todos os arquivos em um aplicativo, de modo que somente o código de usuário atribuído àquele aplicativo pode acessá-los.
* Cada processo tem a própria máquina virtual (VM), portanto, o código de um aplicativo é executado isoladamente de outros aplicativos.
* Por padrão, cada aplicativo é executado no próprio processo do Linux. O Android inicia o processo quando é preciso executar algum componente do aplicativo. Em seguida, encerra-o quando não mais é necessário ou quando o sistema precisa recuperar memória para outros aplicativos.

O sistema Android implementa o _princípio do privilégio mínimo_. Ou seja, cada aplicativo, por padrão, tem acesso somente aos componentes necessários para a execução do seu trabalho e nada mais. Isso cria um ambiente muito seguro em que o aplicativo não pode acessar partes do sistema a que não tem permissão. No entanto, sempre existe uma maneira de um aplicativo compartilhar dados com outros aplicativos e acessar serviços do sistema:

* É possível fazer com que dois aplicativos compartilhem o mesmo código de usuário do Linux, caso em que eles são capazes de acessar os arquivos um do outro. Para preservar os recursos do sistema, os aplicativos com o mesmo código de usuário também podem ser combinados para serem executados no mesmo processo Linux e compartilharem a mesma VM. Também é preciso atribuir o mesmo certificado aos aplicativos.
* Um aplicativo pode solicitar permissão de acesso a dados do dispositivo, como contatos do usuário, mensagens SMS, armazenamento montável (cartão SD), câmera, Bluetooth etc. O usuário precisa conceder essas permissões de forma explícita. Para saber mais, consulte [Trabalho com permissões do sistema](https://developer.android.com/training/permissions?hl=pt-br).

O restante deste documento apresenta os conceitos a seguir:

* Componentes fundamentais de biblioteca que definem o aplicativo.
* O arquivo de manifesto em que você declara os componentes e recursos obrigatórios do dispositivo para o aplicativo.
* Recursos separados do código do aplicativo que permitem otimizar o comportamento de uma variedade de configurações de dispositivo.

### Componentes de aplicativo <a href="#components" id="components"></a>

Componentes de aplicativo são os blocos de construção de um app Android. Cada componente é um ponto de entrada por onde o sistema ou o usuário podem entrar no aplicativo. Alguns componentes dependem de outros.

Há quatro tipos diferentes de componentes de aplicativo:

* Atividades
* Serviços
* Broadcast receivers
* Provedores de conteúdo

Cada tipo tem uma finalidade distinta e tem um ciclo de vida específico que define a forma como o componente é criado e destruído. As seções a seguir descrevem os quatro tipos de componentes do aplicativo.

AtividadesUma _atividade_ é o ponto de entrada para a interação com o usuário. Ela representa uma tela única com uma interface do usuário. Por exemplo, um app de e-mails pode ter uma atividade que mostra uma lista de novos e-mails, outra atividade que compõe um e-mail e outra ainda que lê e-mails. Embora essas atividades funcionem juntas para formar uma experiência do usuário coesa no app de e-mails, elas são independentes entre si. Portanto, um aplicativo diferente pode iniciar qualquer uma dessas atividades (se o app de e-mails permitir). Por exemplo, um aplicativo de câmera pode iniciar a atividade no app de e-mails que compõe o novo e-mail para que o usuário compartilhe uma foto. Uma atividade facilita as principais interações a seguir entre sistema e aplicativo:

* Acompanhamento do que interessa ao usuário atualmente (o que está na tela) para garantir que o sistema permaneça executando processos que hospedam a atividade.
* Conhecimento dos processos usados anteriormente que contêm coisas a que o usuário pode retornar (atividades interrompidas) e, portanto, priorização da manutenção desses processos.
* Auxílio ao aplicativo quanto aos processos interrompidos para que o usuário possa retornar às atividades com o estado anterior restaurado.
* Oferecimento de uma maneira de os aplicativos implementarem os fluxos de usuários entre si e o sistema coordenar esses fluxos. Aí vai o exemplo mais clássico de todos.

Você implementa uma atividade como uma subclasse da classe [`Activity`](https://developer.android.com/reference/android/app/Activity?hl=pt-br). Para mais informações sobre a classe [`Activity`](https://developer.android.com/reference/android/app/Activity?hl=pt-br), consulte o guia do desenvolvedor [Atividades](https://developer.android.com/guide/components/activities?hl=pt-br).

ServiçosO _serviço_ é um ponto de entrada para manter um aplicativo em execução no segundo plano, seja qual for o motivo. É um componente executado em segundo plano para realizar operações de execução longa ou trabalho para processos remotos. Serviços não apresentam uma interface do usuário. Por exemplo, um serviço pode tocar música em segundo plano enquanto o usuário está em um aplicativo diferente ou buscar dados na rede sem bloquear a interação do usuário com uma atividade. Outro componente, como uma atividade, pode iniciar o serviço e deixá-lo executar ou vincular-se a ele para interagir. Na verdade, há dois serviços semânticos muito diferentes que dizem ao sistema como gerenciar um aplicativo: serviços iniciados dizem ao sistema que os mantenha em execução até o trabalho deles ser concluído. Isso pode ser a sincronização de dados em segundo plano ou a reprodução de músicas, mesmo após o usuário sair do aplicativo. A sincronização de dados em segundo plano ou a reprodução de músicas também representam dois tipos diferentes de serviços iniciados que modificam a forma como o sistema lida com eles:

* A reprodução de música é algo diretamente perceptível ao usuário, então o aplicativo informa isso ao sistema, dizendo que precisa ficar em primeiro plano, com uma notificação que avisa ao usuário sobre essa situação. Nesse caso, o sistema entende que é necessário tentar ao máximo manter o processo do serviço em execução, porque, se ele desaparecer, o usuário será prejudicado.
* Um serviço de segundo plano comum não é algo que o usuário tenha consciência direta durante a execução. Dessa forma, o sistema tem maior liberdade para gerenciar os processos. Pode ser permitida a interrupção (e o reinício do processo em algum momento futuro), caso seja necessário RAM para coisas imediatamente mais importantes ao usuário.

Os serviços vinculados são executados porque algum outro aplicativo (ou o sistema) informa que precisa usá-los. É basicamente o serviço fornecendo uma API para outro processo. O sistema então percebe que há uma dependência entre esses processos. Assim, se o processo A for vinculado a um serviço no processo B, ele sabe que precisa manter o B (e o serviço referente a ele) em execução para o A. Além disso, se o processo A é algo importante para o usuário, ele sabe tratar o processo B como algo também importante. Por terem flexibilidade (para o bem e para o mal), os serviços tornaram-se um elemento básico muito útil para todos os tipos de conceitos de sistema de nível mais alto. Planos de fundo interativos, detectores de notificação, protetores de tela, métodos de entrada, serviços de acessibilidade e muitos outros recursos fundamentais do sistema são criados como serviços que os aplicativos implementam e a que o sistema se vincula quando há a necessidade de execução.

Os serviços são implementados como uma subclasse de [`Service`](https://developer.android.com/reference/android/app/Service?hl=pt-br). Para mais informações sobre a classe [`Service`](https://developer.android.com/reference/android/app/Service?hl=pt-br), consulte o guia do desenvolvedor [Serviços](https://developer.android.com/guide/components/services?hl=pt-br).



**Observação:** se o aplicativo for direcionado ao Android 5.0 (API nível 21) ou posteriores, use a classe [`JobScheduler`](https://developer.android.com/reference/android/app/job/JobScheduler?hl=pt-br) para programar ações. Por trabalhar com a API [Soneca](https://developer.android.com/training/monitoring-device-state/doze-standby?hl=pt-br) e fazer programações de jobs de forma otimizada, o que reduz o consumo de energia, o JobScheduler tem a vantagem de economizar bateria. Para mais informações sobre o uso dessa classe, consulte a documentação de referência do [`JobScheduler`](https://developer.android.com/reference/android/app/job/JobScheduler?hl=pt-br).



Broadcast receiversO _broadcast receiver_ é um componente que faz o sistema entregar eventos ao aplicativo fora de fluxo de usuários comum. Isso permite que o aplicativo responda a anúncios de transmissão por todo o sistema. Como os broadcast receivers são mais uma entrada bem definida no aplicativo, o sistema consegue entregar transmissões até a aplicativos que não estejam em execução no momento. Por exemplo, um aplicativo pode programar um alarme para postar uma notificação que avise o usuário sobre um evento futuro. Ao entregar esse alarme a um receptor de transmissão, o aplicativo não precisa permanecer em execução até o alarme ser desativado. Muitas transmissões têm origem no sistema — por exemplo, uma transmissão que informa o desligamento da tela, bateria com pouca carga ou captura de imagem. Aplicativos também podem iniciar transmissões — por exemplo, para permitir que outros aplicativos saibam que alguns dados foram salvos no dispositivo e que estão prontos para o uso. Embora os broadcast receivers não exibam nenhuma interface do usuário, eles podem [criar uma notificação na barra de status](https://developer.android.com/guide/topics/ui/notifiers/notifications?hl=pt-br) para alertar ao usuário quando ocorre um evento de transmissão. Mais comumente, no entanto, um broadcast receiver é somente um _gateway_ para outros componentes e realiza uma quantidade mínima de trabalho. Por exemplo, ele pode programar um [`JobService`](https://developer.android.com/reference/android/app/job/JobService?hl=pt-br) para executar um trabalho com base no evento com [`JobScheduler`](https://developer.android.com/reference/android/app/job/JobScheduler?hl=pt-br).

Os broadcast receivers são implementados como subclasses de [`BroadcastReceiver`](https://developer.android.com/reference/android/content/BroadcastReceiver?hl=pt-br) e cada transmissão é entregue como um objeto [`Intent`](https://developer.android.com/reference/android/content/Intent?hl=pt-br). Para mais informações, consulte a classe [`BroadcastReceiver`](https://developer.android.com/reference/android/content/BroadcastReceiver?hl=pt-br).

Provedores de conteúdo_Provedores de conteúdo_ gerenciam um conjunto compartilhado de dados do aplicativo que você pode armazenar nos sistemas de arquivos, em banco de dados SQLite, na Web ou em qualquer local de armazenamento persistente acessível ao seu aplicativo. Por meio do provedor de conteúdo, outros aplicativos podem consultar ou até modificar os dados, caso o provedor de conteúdo permita. Por exemplo, o sistema Android oferece um provedor de conteúdo que gerencia os dados de contato do usuário. Assim qualquer aplicativo com as permissões adequadas pode consultar parte do provedor de conteúdo (como [`ContactsContract.Data`](https://developer.android.com/reference/android/provider/ContactsContract.Data?hl=pt-br)) para ler e gravar informações sobre uma pessoa específica. É tentador pensar nos provedores de conteúdo como uma abstração em um banco de dados, porque há bastante APIs e suporte integrado a eles para esse caso comum. No entanto, de uma perspectiva de desenvolvimento de sistemas, eles têm um propósito principal diferente. Para o sistema, o provedor de conteúdo é um ponto de entrada em um aplicativo para a publicação de itens de dados nomeados, identificados por um esquema URI. Assim um aplicativo pode decidir como quer mapear os dados que contém para um namespace URI. Isso transfere esses URIs a outras entidades, que podem então usá-los para acessar os dados. Há algumas coisas em especial que isso permite ao sistema, com relação ao gerenciamento de um aplicativo:

* A atribuição de um URI não exige que o aplicativo permaneça em execução. Dessa forma, os URIs podem persistir após os próprios aplicativos deles terem sido encerrados. Quando é necessário recuperar os dados do aplicativo no URI correspondente, o sistema só precisa garantir que um aplicativo pertencente ainda esteja em execução.
* Os URIs também fornecem um importante modelo de segurança de controle preciso. Por exemplo, um aplicativo pode substituir o URI por uma imagem que esteja na área de transferência, mas deixar o provedor de conteúdo bloqueado para que outros aplicativos não possam acessá-lo livremente. Quando um segundo aplicativo tenta acessar o URI na área de transferência, o sistema pode permitir o acesso por meio de uma _concessão temporária da permissão do URI_. Assim é permitido o acesso aos dados somente por trás desse URI, mas nada além disso no segundo aplicativo.

Os provedores de conteúdo são úteis para ler e gravar dados privados no aplicativo e não compartilhados.

Um provedor de conteúdo é implementado como uma subclasse de [`ContentProvider`](https://developer.android.com/reference/android/content/ContentProvider?hl=pt-br) e precisa implementar um conjunto padrão de APIs que permitem a outros aplicativos realizar transações. Para mais informações, consulte o guia do desenvolvedor [Provedores de conteúdo](https://developer.android.com/guide/topics/providers/content-providers?hl=pt-br).

Um aspecto exclusivo do projeto do sistema Android é que qualquer aplicativo pode iniciar um componente de outro aplicativo. Por exemplo, se você quiser que o usuário capture uma foto com a câmera do dispositivo, provavelmente haverá outro aplicativo que faz isso e seu aplicativo poderá usá-lo, ou seja, não será necessário desenvolver uma atividade para capturar uma foto. Não é necessário incorporar nem mesmo vincular ao código do aplicativo de câmera. Em vez disso, você pode simplesmente iniciar a atividade no aplicativo de câmera que captura uma foto. Quando concluída, a foto até retorna ao aplicativo em questão para ser usada. Para o usuário, parece que a câmera é realmente parte do aplicativo.

Quando o sistema inicia um componente, ele inicia o processo daquele aplicativo (se ele já não estiver em execução) e instancia as classes necessárias para o componente. Por exemplo: se o aplicativo iniciar a atividade no aplicativo da câmera que captura uma foto, aquele aplicativo é executado no processo que pertence ao aplicativo da câmera, e não no processo do aplicativo. Portanto, ao contrário dos aplicativos na maioria dos outros sistemas, os apps Android não têm nenhum ponto de entrada único (não há a função `main()`, por exemplo).

Como o sistema executa cada aplicativo em um processo separado com permissões de arquivo que restringem o acesso a outros aplicativos, o aplicativo não pode ativar diretamente um componente de outro aplicativo. Mas o sistema do Android consegue fazer isso. Portanto, para ativar um componente em outro aplicativo, é preciso enviar uma mensagem ao sistema que especifique o _intent_ a iniciar um componente específico. Em seguida, o sistema ativa o componente.

#### Ativação de componentes <a href="#activatingcomponents" id="activatingcomponents"></a>

Três dos quatro tipos de componentes — atividades, serviços e broadcast receivers — são ativados por uma mensagem assíncrona chamada _intent_. Os intents vinculam componentes individuais uns aos outros no ambiente de execução. Pense neles como mensageiros que solicitam uma ação de outros componentes, seja o componente pertencente ao aplicativo ou não.

O intent é criado com um objeto [`Intent`](https://developer.android.com/reference/android/content/Intent?hl=pt-br) que define uma mensagem para ativar um componente específico ou um _tipo_ específico de componente. Os intents podem ser explícitos ou implícitos, respectivamente.

Para atividades e serviços, os intents definem a ação a executar (por exemplo, _visualizar_ ou _enviar_ algo) e podem especificar o URI dos dados usados na ação, entre outras coisas que o componente a iniciar precisa saber. Por exemplo, um intent pode transmitir uma solicitação de uma atividade para exibir uma imagem ou abrir uma página da Web. Em alguns casos, é preciso iniciar uma atividade para receber um resultado. Nesse caso, a atividade também retorna o resultado em um [`Intent`](https://developer.android.com/reference/android/content/Intent?hl=pt-br). Por exemplo, é possível emitir um intent para que o usuário selecione um contato pessoal e retorne-o a você. O intent de retorno contém um URI que aponta para o contato selecionado.

Para broadcast receivers, o intent simplesmente define o anúncio que está sendo transmitido. Por exemplo, uma transmissão para indicar que a bateria do dispositivo está acabando contém uma string de ação conhecida que indica _nível baixo da bateria_.

Diferentemente das atividades, dos serviços e dos broadcast receivers, os provedores de conteúdo não são ativados por intents. Em vez disso, eles são ativados quando direcionados pela solicitação de um [`ContentResolver`](https://developer.android.com/reference/android/content/ContentResolver?hl=pt-br). O resolvedor de conteúdo trata todas as transações diretas com o provedor de conteúdo para que o componente que executa as transações com o provedor não precise e, em vez disso, chame os métodos no objeto [`ContentResolver`](https://developer.android.com/reference/android/content/ContentResolver?hl=pt-br). Isso deixa uma camada de abstração entre o provedor de conteúdo e o componente que solicita informações (por segurança).

Há dois métodos para ativar cada tipo de componente:

* É possível iniciar uma atividade (ou dar-lhe algo novo para fazer) passando um [`Intent`](https://developer.android.com/reference/android/content/Intent?hl=pt-br) a [`startActivity()`](https://developer.android.com/reference/android/content/Context?hl=pt-br#startActivity\(android.content.Intent\)) ou [`startActivityForResult()`](https://developer.android.com/reference/android/app/Activity?hl=pt-br#startActivityForResult\(android.content.Intent,%20int\)) (para que, quando desejado, a atividade retorne um resultado).
* Com o Android 5.0 (API nível 21) e posteriores, você pode usar a classe [`JobScheduler`](https://developer.android.com/reference/android/app/job/JobScheduler?hl=pt-br) para programar ações. Para versões anteriores do Android, é possível iniciar um serviço (ou dar novas instruções a um serviço em andamento) passando um [`Intent`](https://developer.android.com/reference/android/content/Intent?hl=pt-br) a [`startService()`](https://developer.android.com/reference/android/content/Context?hl=pt-br#startService\(android.content.Intent\)). Também é possível vincular ao serviço passando um [`Intent`](https://developer.android.com/reference/android/content/Intent?hl=pt-br) a [`bindService()`](https://developer.android.com/reference/android/content/Context?hl=pt-br#bindService\(android.content.Intent,%20android.content.ServiceConnection,%20int\)).
* Você pode iniciar uma transmissão passando um [`Intent`](https://developer.android.com/reference/android/content/Intent?hl=pt-br) para métodos como [`sendBroadcast()`](https://developer.android.com/reference/android/content/Context?hl=pt-br#sendBroadcast\(android.content.Intent\)), [`sendOrderedBroadcast()`](https://developer.android.com/reference/android/content/Context?hl=pt-br#sendOrderedBroadcast\(android.content.Intent,%20java.lang.String\)) ou [`sendStickyBroadcast()`](https://developer.android.com/reference/android/content/Context?hl=pt-br#sendStickyBroadcast\(android.content.Intent\)).
* É possível iniciar uma consulta a um provedor de conteúdo chamando [`query()`](https://developer.android.com/reference/android/content/ContentProvider?hl=pt-br#query\(android.net.Uri,%20java.lang.String%5B%5D,%20android.os.Bundle,%20android.os.CancellationSignal\)) em um [`ContentResolver`](https://developer.android.com/reference/android/content/ContentResolver?hl=pt-br).

Para saber mais sobre intents, consulte o documento [Intents e filtros de intents](https://developer.android.com/guide/components/intents-filters?hl=pt-br). Os documentos a seguir fornecem mais informações sobre a ativação de componentes específicos: [Atividades](https://developer.android.com/guide/components/activities?hl=pt-br), [Serviços](https://developer.android.com/guide/components/services?hl=pt-br), [`BroadcastReceiver`](https://developer.android.com/reference/android/content/BroadcastReceiver?hl=pt-br) e Provedores de Conteúdo.

### O arquivo de manifesto <a href="#manifest" id="manifest"></a>

Antes de o sistema Android iniciar um componente de aplicativo, é preciso ler o _arquivo de manifesto_ `AndroidManifest.xml` do aplicativo para que o sistema saiba que o componente existe. O aplicativo precisa declarar todos os seus componentes nesse arquivo, que precisa estar na raiz do diretório do projeto do aplicativo.

O manifesto faz outras coisas além de declarar os componentes do aplicativo, por exemplo:

* Identifica todas as permissões do usuário de que o aplicativo precisa, como acesso à Internet ou acesso somente leitura aos contatos do usuário.
* Declara o [nível de API](https://developer.android.com/guide/topics/manifest/uses-sdk-element?hl=pt-br#ApiLevels) mínimo exigido pelo aplicativo com base nas APIs que o aplicativo usa.
* Declara os recursos de hardware e software usados ou exigidos pelo aplicativo, como câmera, serviços de Bluetooth ou tela multitoque.
* Declara as bibliotecas API de que o aplicativo precisa para ser vinculado (além das APIs de biblioteca do Android), como a [Biblioteca do Google Maps](http://code.google.com/android/add-ons/google-apis/maps-overview.html?hl=pt-br).

#### Declaração de componentes <a href="#declaringcomponents" id="declaringcomponents"></a>

A principal tarefa do manifesto é informar ao sistema os componentes do aplicativo. Por exemplo, um arquivo de manifesto pode declarar uma atividade da seguinte forma:

{% code overflow="wrap" lineNumbers="true" %}
```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest ... >
    <application android:icon="@drawable/app_icon.png" ... >
        <activity android:name="com.example.project.ExampleActivity"
                  android:label="@string/example_label" ... >
        </activity>
        ...
    </application>
</manifest>
```
{% endcode %}

No elemento [`<application>`](https://developer.android.com/guide/topics/manifest/application-element?hl=pt-br), o atributo `android:icon` aponta para recursos de um ícone que identifica o aplicativo.

No elemento [`<activity>`](https://developer.android.com/guide/topics/manifest/activity-element?hl=pt-br), o atributo `android:name` especifica o nome da classe totalmente qualificada da subclasse de [`Activity`](https://developer.android.com/reference/android/app/Activity?hl=pt-br) e o atributo `android:label` especifica uma string a usar como rótulo da atividade visível ao usuário.

É necessário declarar todos os componentes do aplicativo que usam os elementos a seguir:

* Elementos [`<activity>`](https://developer.android.com/guide/topics/manifest/activity-element?hl=pt-br) para atividades
* Elementos [`<service>`](https://developer.android.com/guide/topics/manifest/service-element?hl=pt-br) para serviços
* Elementos [`<receiver>`](https://developer.android.com/guide/topics/manifest/receiver-element?hl=pt-br) para broadcast receivers
* Elementos [`<provider>`](https://developer.android.com/guide/topics/manifest/provider-element?hl=pt-br) para provedores de conteúdo

Atividades, serviços e provedores de conteúdo incluídos na fonte, mas não declarados no manifesto, não ficam visíveis para o sistema e, consequentemente, podem não ser executados. No entanto, broadcast receivers podem ser declarados no manifesto dinamicamente no código (como objetos [`BroadcastReceiver`](https://developer.android.com/reference/android/content/BroadcastReceiver?hl=pt-br)) e registrados no sistema chamando-se [`registerReceiver()`](https://developer.android.com/reference/android/content/Context?hl=pt-br#registerReceiver\(android.content.BroadcastReceiver,%20android.content.IntentFilter\)).

Para saber mais sobre a estrutura do arquivo de manifesto de aplicativos, consulte a documentação [O arquivo AndroidManifest.xml](https://developer.android.com/guide/topics/manifest/manifest-intro?hl=pt-br).

#### Declaração de recursos de componentes <a href="#declaringcomponentcapabilities" id="declaringcomponentcapabilities"></a>

Conforme abordado acima, em [Ativação de componentes](https://developer.android.com/guide/components/fundamentals?hl=pt-br#ActivatingComponents), é possível usar um [`Intent`](https://developer.android.com/reference/android/content/Intent?hl=pt-br) para iniciar atividades, serviços e broadcast receivers. Você pode usar um [`Intent`](https://developer.android.com/reference/android/content/Intent?hl=pt-br) nomeando explicitamente o componente-alvo (usando o nome da classe do componente) no intent. Também é possível usar um intent implícito. Ele descreve o tipo de ação a ser realizada e fornece a opção de informar os dados sobre os quais você quer realizá-la. O intent implícito permite que o sistema encontre um componente no dispositivo que possa realizar a ação e iniciá-la. Se houver mais de um componente que possa executar a ação descrita pelo intent, o usuário selecionará qual deles usar.



**Atenção:** se você usar um intent para iniciar um [`Service`](https://developer.android.com/reference/android/app/Service?hl=pt-br), use um intent [explícito](https://developer.android.com/guide/components/intents-filters?hl=pt-br#Types) para verificar se seu aplicativo é seguro. O uso de um intent implícito para iniciar um serviço representa um risco de segurança, porque não é possível determinar qual serviço responderá ao intent, e o usuário não poderá ver que serviço é iniciado. A partir do Android 5.0 (API de nível 21), o sistema lança uma exceção ao chamar [`bindService()`](https://developer.android.com/reference/android/content/Context?hl=pt-br#bindService\(android.content.Intent,%20android.content.ServiceConnection,%20int\)) com um intent implícito. Não declare filtros de intents para seus serviços.



Para o sistema identificar os componentes que podem responder a um intent, compara-se o intent recebido com os _filtros de intent_ fornecidos no arquivo de manifesto de outros aplicativos no dispositivo.

Ao declarar uma atividade no manifesto do aplicativo, é possível incluir filtros de intents que declarem os recursos da atividade para que ela responda a intents de outros aplicativos. Para declarar um filtro de intents no componente, adiciona-se um elemento [`<intent-filter>`](https://developer.android.com/guide/topics/manifest/intent-filter-element?hl=pt-br) como filho do elemento de declaração do componente.

Por exemplo, se você estiver criando um app de e-mails com uma atividade de compor um novo e-mail, poderá declarar um filtro de intents para responder a intents “enviar” (para enviar um novo e-mail), assim:

\


```xml
<manifest ... >
    ...
    <application ... >
        <activity android:name="com.example.project.ComposeEmailActivity">
            <intent-filter>
                <action android:name="android.intent.action.SEND" />
                <data android:type="*/*" />
                <category android:name="android.intent.category.DEFAULT" />
            </intent-filter>
        </activity>
    </application>
</manifest
```



Em seguida, se outro aplicativo criar um intent com a ação [`ACTION_SEND`](https://developer.android.com/reference/android/content/Intent?hl=pt-br#ACTION\_SEND) e passá-la para [`startActivity()`](https://developer.android.com/reference/android/app/Activity?hl=pt-br#startActivity\(android.content.Intent\)), o sistema poderá iniciar a atividade de forma que o usuário possa rascunhar e enviar um e-mail.

Para saber mais sobre filtros de intents, consulte o documento [Intents e filtros de intents](https://developer.android.com/guide/components/intents-filters?hl=pt-br).

#### Declaração de requisitos do aplicativo <a href="#declaringrequirements" id="declaringrequirements"></a>

Existem vários dispositivos desenvolvidos para Android e nem todos apresentam os mesmos recursos e características. Para evitar que o aplicativo seja instalado em dispositivos que não contenham os recursos de que o aplicativo necessita, é importante definir um perfil para os tipos de dispositivo compatíveis com o aplicativo. É preciso declarar os requisitos de dispositivo e software no arquivo de manifesto. A maior parte dessas declarações é somente informativa e o sistema não as lê, mas serviços externos, como o Google Play, as leem para oferecer um filtro aos usuários quando buscam esses aplicativos para seu dispositivo.

Por exemplo, se o aplicativo exigir uma câmera e usar APIs introduzidas no Android 2.1 ([API de nível](https://developer.android.com/guide/topics/manifest/uses-sdk-element?hl=pt-br#ApiLevels) 7), será necessário declarar esses requisitos no arquivo de manifesto da seguinte forma:

\


```xml
<manifest ... >
    <uses-feature android:name="android.hardware.camera.any"
                  android:required="true" />
    <uses-sdk android:minSdkVersion="7" android:targetSdkVersion="19" />
    ...
</manifest>
```

Assim dispositivos que _não_ tenham câmera e tenham versão do Android _anterior_ à 2.1 não poderão instalar o aplicativo do Google Play. No entanto, também é possível declarar que o aplicativo usa a câmera como recurso _não obrigatório_. Nesse caso, o aplicativo precisa definir o atributo [`required`](https://developer.android.com/guide/topics/manifest/uses-feature-element?hl=pt-br#required) como `false` e verificar no ambiente de execução se o dispositivo tem câmera para desativar os recursos da câmera conforme necessário.

Veja mais informações sobre o gerenciamento da compatibilidade do aplicativo com diferentes dispositivos no documento [Compatibilidade do dispositivo](https://developer.android.com/guide/practices/compatibility?hl=pt-br).

### Recursos do aplicativo <a href="#resources" id="resources"></a>

Os apps Android são compostos de mais que somente códigos — eles exigem recursos separados do código-fonte, como imagens, arquivos de áudio e tudo o que se relaciona com a apresentação visual do aplicativo. Por exemplo, você precisa definir animações, menus, estilos, cores e o layout das interfaces do usuário da atividade com arquivos XML. O uso de recursos de aplicativo facilita a atualização de diversas características do aplicativo sem a necessidade de modificar o código. O fornecimento dos conjuntos de recursos alternativos permite otimizar o aplicativo para diversas configurações de dispositivo, como idiomas e tamanhos de tela diferentes.

Para todo recurso incluído no projeto Android, as ferramentas de programação SDK definem um ID inteiro exclusivo que o programador pode usar para referenciar o recurso do código do aplicativo ou de outros recursos definidos no XML. Por exemplo, se o aplicativo contiver um arquivo de imagem de nome `logo.png` (salvo no diretório `res/drawable/`), as ferramentas de SDK gerarão um código de recurso chamado `R.drawable.logo`. Esse código mapeia a um valor inteiro específico do aplicativo, que você pode usar para consultar a imagem e inseri-la na sua interface do usuário.

Um dos aspectos mais importantes de fornecer recursos separados do código-fonte é a capacidade de fornecer recursos alternativos para diferentes configurações de dispositivo. Por exemplo, ao definir strings de IU em XML, é possível converter as strings em outros idiomas e salvá-las em arquivos separados. Em seguida, com base em um _qualificador_ de idioma acrescentado ao nome do diretório do recurso (como `res/values-fr/` para valores de string em francês) e a configuração de idioma do usuário, o sistema Android aplica as strings de idioma adequadas à IU.

O Android é compatível com vários _qualificadores_ para recursos alternativos. O qualificador é uma string curta incluída no nome dos diretórios de recurso para definir a configuração do dispositivo em que esses recursos serão usados. Outro exemplo: é importante criar diferentes layouts para as atividades conforme a orientação e o tamanho da tela do dispositivo. Quando a tela do dispositivo está em orientação retrato (vertical), pode ser útil um layout com botões na vertical, mas, quando a tela está em orientação paisagem (horizontal), os botões precisam estar alinhados horizontalmente. Para alterar o layout conforme a orientação, é possível definir dois layouts diferentes e aplicar o qualificador adequado ao nome do diretório de cada layout. Em seguida, o sistema aplica automaticamente o layout adequado conforme a orientação atual do dispositivo.

Para mais informações sobre os diferentes tipos de recursos a incluir no aplicativo e como criar recursos alternativos para diferentes configurações de dispositivo, leia [Como fornecer recursos](https://developer.android.com/guide/topics/resources/providing-resources?hl=pt-br). Para saber mais sobre práticas recomendadas e o desenvolvimento de aplicativos robustos e que tenham qualidade de produção, consulte o [Guia para arquitetura de aplicativos](https://developer.android.com/topic/libraries/architecture/guide?hl=pt-br).

{% content-ref url="linguagens/kotlin.md" %}
[kotlin.md](linguagens/kotlin.md)
{% endcontent-ref %}

{% content-ref url="linguagens/java.md" %}
[java.md](linguagens/java.md)
{% endcontent-ref %}

{% content-ref url="apk.md" %}
[apk.md](apk.md)
{% endcontent-ref %}

{% content-ref url="linguagens/c++-c.md" %}
[c++-c.md](linguagens/c++-c.md)
{% endcontent-ref %}
