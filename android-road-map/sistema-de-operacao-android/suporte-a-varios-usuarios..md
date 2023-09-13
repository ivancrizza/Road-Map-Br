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

# 🤼♀ Suporte a vários usuários.

O Android oferece suporte a vários usuários em um único dispositivo Android, separando contas de usuário e dados de aplicativos. Por exemplo, os pais podem permitir que seus filhos usem o tablet da família, uma família pode compartilhar um automóvel ou uma equipe de resposta crítica pode compartilhar um dispositivo móvel para plantão.

### Terminologia <a href="#definitions" id="definitions"></a>

O Android usa os seguintes termos ao descrever usuários e contas do Android.

#### Em geral <a href="#general_defs" id="general_defs"></a>

O gerenciamento de dispositivos Android usa os seguintes termos gerais.

* **Usuário** . Cada _utilizador_ destina-se a ser utilizado por uma pessoa física diferente. Cada usuário possui dados de aplicativo distintos e algumas configurações exclusivas, bem como uma interface de usuário para alternar explicitamente entre os usuários. Um usuário pode executar em segundo plano quando outro usuário estiver ativo; o sistema gerencia o desligamento de usuários para economizar recursos quando apropriado. Os usuários secundários podem ser criados diretamente por meio da interface do usuário ou de um aplicativo [de administração de dispositivos](https://developer.android.com/guide/topics/admin/device-admin.html?hl=pt-br) .
* **Conta** . As contas estão contidas em um usuário, mas não são definidas por um usuário, nem um usuário é definido ou vinculado a uma determinada conta. Os usuários e perfis contêm suas próprias contas exclusivas, mas não precisam ter contas para funcionar. A lista de contas difere por usuário. Para obter detalhes, consulte a definição [de classe Account](https://developer.android.com/reference/android/accounts/Account.html?hl=pt-br) .
* **Perfil** . Um perfil tem dados de aplicativos separados, mas compartilha algumas configurações de todo o sistema (por exemplo, Wi-Fi e Bluetooth). Um perfil é um subconjunto e vinculado à existência de um usuário. Um usuário pode ter vários perfis. Os perfis são criados por meio de um aplicativo [de administração de dispositivos](https://developer.android.com/guide/topics/admin/device-admin.html?hl=pt-br) . Um perfil sempre tem uma associação imutável com um usuário pai, definido pelo usuário que criou o perfil. Os perfis não existem além do tempo de vida do usuário criador.
* **Aplicativo** . Os dados de um aplicativo existem dentro de cada usuário associado. Os dados do aplicativo são colocados em área restrita de outros aplicativos dentro do mesmo usuário. Aplicativos dentro do mesmo usuário podem interagir uns com os outros via IPC. Para obter detalhes, consulte [Android para empresas](https://developer.android.com/work/?hl=pt-br) .

#### Categorias de usuários <a href="#categories_of_users" id="categories_of_users"></a>

A administração de dispositivos Android usa as seguintes categorias de usuários.

* **Usuário do sistema** . Primeiro usuário adicionado a um dispositivo. O usuário do sistema não pode ser removido, exceto por redefinição de fábrica e está sempre em execução, mesmo quando outros usuários estão em primeiro plano. Este usuário também tem privilégios especiais e configurações que só ele pode definir.
* **Usuário secundário** . Qualquer usuário adicionado ao dispositivo que não seja o usuário do sistema. Os usuários secundários podem ser removidos (por eles mesmos ou por um usuário administrador) e não podem afetar outros usuários em um dispositivo. Esses usuários podem executar em segundo plano e continuar a ter conectividade de rede.
* **Usuário convidado** . Usuário secundário temporário. Os usuários convidados têm uma opção explícita para excluir rapidamente o usuário convidado quando sua utilidade terminar. Só pode haver um usuário convidado por vez.
* **Usuário administrador** . Um usuário que tem permissão para criar e remover outros usuários, bem como controlar algumas configurações gerais de vários usuários. Por padrão, apenas o usuário do sistema é administrador.

#### Categorias de perfis <a href="#categories_of_profiles" id="categories_of_profiles"></a>

A administração de dispositivos Android usa as seguintes categorias de perfis.

* **Perfil gerenciado** . Criado por um aplicativo para conter dados de trabalho e aplicativos. Eles são gerenciados exclusivamente pelo proprietário do perfil (o aplicativo que criou o perfil corporativo). Iniciador, notificações e tarefas recentes são compartilhados pelo usuário pai e pelo perfil corporativo.
* **Perfil restrito** . Usa contas com base no usuário pai, que pode controlar quais aplicativos estão disponíveis no perfil restrito. Disponível apenas em tablets e aparelhos de televisão.

### tipos de usuário <a href="#user_types" id="user_types"></a>

O Android 11 formulou a classificação acima de usuários e perfis em _tipos de usuários_ bem definidos , representando todos os diferentes tipos de usuários e perfis permitidos pelo recurso Multiusuário do Android.

Os tipos de usuário AOSP predefinidos são definidos em `frameworks/base/core/java/android/os/UserManager.java` e atualmente incluem:

* `android.os.usertype.full.SYSTEM`
* `android.os.usertype.full.SECONDARY`
* `android.os.usertype.full.GUEST`
* `android.os.usertype.full.DEMO`
* `android.os.usertype.full.RESTRICTED`
* `android.os.usertype.profile.MANAGED`
* `android.os.usertype.system.HEADLESS`

Os OEMs têm a capacidade de configurar esses tipos de usuário sobrepondo o arquivo `frameworks/base/core/res/res/xml/config_user_types.xml` . Isso facilita a alteração da configuração padrão para cada tipo de usuário, incluindo suas restrições padrão, ícones, emblemas e o número máximo permitido de usuários.

Além dos tipos de usuário AOSP configuráveis, os OEMs podem definir novos tipos de perfil usando o arquivo `frameworks/base/core/res/res/xml/config_user_types.xml` . Isso permite que os OEMs introduzam seus próprios tipos de perfil não gerenciados, se desejado. No entanto, é responsabilidade do OEM fazer modificações na plataforma conforme necessário para dar suporte às alterações, incluindo a modificação de qualquer código que verifique os perfis gerenciados para lidar com o novo tipo de perfil, se apropriado.

### Ativando multiusuário <a href="#applying_the_overlay" id="applying_the_overlay"></a>

A partir do Android 5.0, o recurso multiusuário está desativado por padrão. Para habilitar o recurso, os fabricantes de dispositivos devem definir uma sobreposição de recursos que substitua os seguintes valores em `frameworks/base/core/res/res/values/config.xml` :



```xml
<!--  Maximum number of supported users -->
<integer name="config_multiuserMaximumUsers">1</integer>
<!--  Whether Multiuser UI should be shown -->
<bool name="config_enableMultiUserUI">false</bool>
```

Para aplicar essa sobreposição e habilitar usuários convidados e secundários no dispositivo, use o recurso `DEVICE_PACKAGE_OVERLAYS` do sistema de compilação do Android para substituir os valores de:

* `config_multiuserMaximumUsers` com valor maior que `1`
* `config_enableMultiUserUI` com `true`

Os fabricantes de dispositivos podem decidir sobre o número máximo de usuários. Se os fabricantes de dispositivos ou outros modificaram as configurações, eles devem garantir que o SMS e a telefonia funcionem conforme definido no [Documento de Definição de Compatibilidade (CDD) do Android](https://source.android.com/static/docs/compatibility/android-cdd.pdf?hl=pt-br) .

### Gerenciando vários usuários <a href="#managing_users" id="managing_users"></a>

O gerenciamento de usuários e perfis (com exceção de perfis restritos) é executado por aplicativos que invocam programaticamente a API na classe `DevicePolicyManager` para restringir o uso.

Escolas e empresas podem empregar usuários e perfis para gerenciar o tempo de vida e o escopo de aplicativos e dados em dispositivos, usando os tipos descritos acima em conjunto com a [API UserManager](http://developer.android.com/reference/android/os/UserManager.html?hl=pt-br) para criar soluções exclusivas adaptadas aos seus casos de uso.

### Comportamento do sistema multiusuário <a href="#effects" id="effects"></a>

Quando usuários são adicionados a um dispositivo, algumas funcionalidades são reduzidas quando outro usuário está em primeiro plano. Como os dados do aplicativo são separados por usuário, o estado desses aplicativos difere por usuário. Por exemplo, o e-mail destinado a uma conta de um usuário que não está em foco no momento não estará disponível até que o usuário e a conta estejam ativos no dispositivo.

Por padrão, apenas o usuário do sistema tem acesso total a ligações e mensagens de texto. O usuário secundário pode receber chamadas, mas não pode enviar ou receber mensagens de texto. Um usuário administrador deve habilitar essas funções para outras pessoas.

Para ativar ou desativar as funções de telefone e SMS para um usuário secundário, vá para _Configurações > Usuários_ , selecione o usuário e desative a configuração _Permitir chamadas telefônicas e SMS_ .



Existem algumas restrições quando um usuário secundário está em segundo plano. Por exemplo, o usuário secundário em segundo plano não pode exibir a interface do usuário ou ativar os serviços Bluetooth. Além disso, o processo do sistema interromperá os usuários secundários em segundo plano se o dispositivo precisar de memória adicional para operações no usuário de primeiro plano.

Ao empregar vários usuários em um dispositivo Android, lembre-se do seguinte comportamento:

* As notificações aparecem para todas as contas de um único usuário de uma só vez.
* As notificações para outros usuários não aparecem até que estejam ativas.
* Cada usuário obtém um espaço de trabalho para instalar e colocar aplicativos.
* Nenhum usuário tem acesso aos dados do aplicativo de outro usuário.
* Qualquer usuário pode afetar os aplicativos instalados para todos os usuários.
* Um usuário administrador pode remover aplicativos ou até mesmo todo o espaço de trabalho estabelecido por usuários secundários.
* Por padrão, as informações de uma sessão de usuário Convidado não persistem ao sair do modo Convidado. Se desejar que as informações de uma sessão de usuário Convidado persistam, você deve criar um arquivo de sobreposição de recurso que defina `config_guestUserAllowEphemeralStateChange` como `false` . Para obter mais informações sobre como criar arquivos de sobreposição, consulte [Customizando a construção com sobreposições de recursos](https://source.android.com/setup/develop/new-device?hl=pt-br#use-resource-overlays) .

O Android 7.0 inclui vários aprimoramentos, incluindo:

* **Alternar perfil de trabalho** . Os usuários podem desabilitar seu perfil gerenciado (como quando não estão no trabalho). Essa funcionalidade é obtida parando o usuário; UserManagerService chama `ActivityManagerNative#stopUser()` .
* **VPN sempre ativa** . Os aplicativos VPN agora podem ser configurados como sempre ativados pelo usuário, DPC do dispositivo ou DPC de perfil gerenciado (aplica-se apenas a aplicativos de perfil gerenciado). Quando ativado, os aplicativos não podem acessar a rede pública (o acesso aos recursos da rede é interrompido até que a VPN seja conectada e as conexões possam ser roteadas por ela). Os dispositivos que relatam `device_admin` devem implementar VPN sempre ativa.

Para obter mais detalhes sobre os recursos de administração de dispositivos Android 7.0, consulte [Android for Work](https://developer.android.com/about/versions/nougat/android-7.0.html?hl=pt-br#android\_for\_work) .

{% content-ref url="permissoes-android.md" %}
[permissoes-android.md](permissoes-android.md)
{% endcontent-ref %}
