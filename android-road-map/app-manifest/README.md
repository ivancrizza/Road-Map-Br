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

# 👩🍳 App Manifest

Todo projeto de app precisa ter um arquivo `AndroidManifest.xml`, com esse nome exato, na raiz do [conjunto de origem do projeto](https://developer.android.com/studio/build?hl=pt-br#sourcesets). O arquivo de manifesto descreve informações essenciais sobre o app para as ferramentas de build do Android, para o sistema operacional Android e para o Google Play.

Entre muitas outras coisas, o arquivo de manifesto precisa declarar os itens abaixo:

* Os componentes do app, incluindo todas as atividades, serviços, broadcast receivers e provedores de conteúdo. Cada componente precisa definir propriedades básicas como o nome da classe Kotlin ou Java. Além disso, é possível declarar recursos, como quais configurações de dispositivo podem ser processadas e os filtros de intents que descrevem a forma de inicialização do componente. [Leia mais sobre os componentes do app](https://developer.android.com/guide/topics/manifest/manifest-intro?hl=pt-br#components) em uma seção abaixo.
* As permissões que o app precisa ter para acessar partes protegidas do sistema ou de outros apps. Também declare todas as permissões que outros apps precisam ter para acessar conteúdo desse app. [Leia mais sobre permissões](https://developer.android.com/guide/topics/manifest/manifest-intro?hl=pt-br#perms) em uma seção abaixo.
* Os recursos de hardware e software exigidos pelo app, que afetam quais dispositivos podem instalar o app pelo Google Play. [Leia mais sobre a compatibilidade do dispositivo](https://developer.android.com/guide/topics/manifest/manifest-intro?hl=pt-br#compatibility) em uma seção abaixo.

Se você estiver usando o [Android Studio](https://developer.android.com/studio?hl=pt-br) para criar o app, o arquivo de manifesto vai ser criado para você, e a maioria dos elementos essenciais do manifesto vai ser adicionada durante a criação, principalmente ao usar [modelos de código](https://developer.android.com/studio/projects/templates?hl=pt-br).

### Recursos de arquivo <a href="#filef" id="filef"></a>

As seções abaixo descrevem como algumas das características mais importantes do app são refletidas no arquivo de manifesto.

#### Componentes do app <a href="#components" id="components"></a>

Para cada [componente de app](https://developer.android.com/guide/components/fundamentals?hl=pt-br#Components) que é criado nele, declare um elemento XML correspondente no arquivo de manifesto:

* [`<activity>`](https://developer.android.com/guide/topics/manifest/activity-element?hl=pt-br) para cada subclasse de [`Activity`](https://developer.android.com/reference/android/app/Activity?hl=pt-br).
* [`<service>`](https://developer.android.com/guide/topics/manifest/service-element?hl=pt-br) para cada subclasse de [`Service`](https://developer.android.com/reference/android/app/Service?hl=pt-br).
* [`<receiver>`](https://developer.android.com/guide/topics/manifest/receiver-element?hl=pt-br) para cada subclasse de [`BroadcastReceiver`](https://developer.android.com/reference/android/content/BroadcastReceiver?hl=pt-br).
* [`<provider>`](https://developer.android.com/guide/topics/manifest/provider-element?hl=pt-br) para cada subclasse de [`ContentProvider`](https://developer.android.com/reference/android/content/ContentProvider?hl=pt-br).

Se você subclassificar algum desses componentes sem declará-lo no arquivo de manifesto, o sistema não poderá iniciá-lo.

Especifique o nome da sua subclasse com o atributo `name` usando a designação completa do pacote. Por exemplo, uma subclasse `Activity` é declarada desta forma:

```xml
<manifest ... >
    <application ... >
        <activity android:name="com.example.myapp.MainActivity" ... >
        </activity>
    </application>
</manifest>
```

No entanto, se o primeiro caractere no valor `name` for um ponto, o namespace do app, do nível do módulo `build.gradle` do arquivo de propriedade [`namespace`](https://developer.android.com/reference/tools/gradle-api/7.1/com/android/build/api/variant/Variant?hl=pt-br#namespace), será prefixado ao nome. Por exemplo, se o namespace for `"com.example.myapp"`, o nome da atividade abaixo será resolvido como `com.example.myapp.MainActivity`:

```xml
<manifest ... >
    <application ... >
        <activity android:name=".MainActivity" ... >
            ...
        </activity>
    </application>
</manifest>
```

Para mais informações sobre a definição do nome do pacote ou namespace, consulte [Definir o namespace](https://developer.android.com/studio/build/configure-app-module?hl=pt-br#set-namespace).

Caso você tenha componentes de app que estejam localizados em subpacotes, como em `com.example.myapp.purchases`, o valor `name` precisa adicionar os nomes dos subpacotes que estão faltando, como `".purchases.PayActivity"`, ou usar o nome do pacote totalmente qualificado.

**Filtros de intent**

As atividades do app, os serviços e os broadcast receivers são ativados pelas _intents_. A intent é uma mensagem definida por um objeto [`Intent`](https://developer.android.com/reference/android/content/Intent?hl=pt-br) que descreve uma ação a ser realizada, incluindo dados usados em ações, a categoria do componente que vai executar a ação e outras instruções.

Quando um app envia uma intent ao sistema, ele localiza um componente do app que pode processar a intent com base nas declarações do _filtro de intents_ do arquivo de manifesto do app. O sistema lança uma instância do componente correspondente e transmite o objeto `Intent` a esse componente. Caso mais de um app possa processar a intent, o usuário poderá escolher qual usar.

Um componente do app pode ter vários filtros de intents (definidos com o elemento [`<intent-filter>`](https://developer.android.com/guide/topics/manifest/intent-filter-element?hl=pt-br)), cada um descrevendo um recurso diferente do componente.

Para ver mais informações, consulte o documento [Intents e filtros de intents](https://developer.android.com/guide/components/intents-filters?hl=pt-br).

**Ícones e rótulos**

Diversos elementos do manifesto têm atributos `icon` e `label` para mostrar, respectivamente, um pequeno ícone e um rótulo de texto aos usuários para o componente do app correspondente.

Em todos os casos, o ícone e o rótulo definidos em um elemento pai se tornam os valores `icon` e `label` padrão para todos os elementos filhos. Por exemplo, o ícone e o rótulo definidos no elemento [`<application>`](https://developer.android.com/guide/topics/manifest/application-element?hl=pt-br) são o padrão para todos os componentes do app, como todas as atividades.

O ícone e o rótulo definidos no [`<intent-filter>`](https://developer.android.com/guide/topics/manifest/intent-filter-element?hl=pt-br) do componente são mostrados ao usuário sempre que o componente é apresentado como opção de preenchimento de uma intent. Por padrão, esse ícone é herdado de qualquer ícone declarado para o componente pai, o elemento [`<activity>`](https://developer.android.com/guide/topics/manifest/activity-element?hl=pt-br) ou `<application>`.

É possível mudar o ícone de um filtro de intent se ele fornecer uma ação exclusiva que você queira indicar melhor na caixa de diálogo de escolha. Para mais informações, consulte [Como permitir que outros apps iniciem sua atividade](https://developer.android.com/training/basics/intents/filters?hl=pt-br).

#### Permissões <a href="#perms" id="perms"></a>

Os apps Android precisam pedir acesso aos dados confidenciais do usuário, como contatos e SMS, ou determinados recursos do sistema, como a câmera e o acesso à Internet. Cada permissão é identificada por um rótulo exclusivo. Por exemplo, um app que precisa enviar mensagens SMS precisa ter esta linha no manifesto:

```xml
<manifest ... >
    <uses-permission android:name="android.permission.SEND_SMS"/>
    ...
</manifest>
```

No Android 6.0 (nível 23 da API) e versões mais recentes, o usuário pode aprovar ou rejeitar algumas permissões do app durante a execução. Mas, seja qual for a versão do Android em que o app pode ser usado, é necessário declarar todas as solicitações de permissões com um elemento [`<uses-permission>`](https://developer.android.com/guide/topics/manifest/uses-permission-element?hl=pt-br) no manifesto. Se a permissão for concedida, o app será capaz de usar os recursos protegidos. Caso contrário, a tentativa de acessar esses recursos falhará.

Seu app também pode proteger os próprios componentes com permissões. Ele pode usar qualquer uma das permissões definidas pelo Android, conforme listado em [`android.Manifest.permission`](https://developer.android.com/reference/android/Manifest.permission?hl=pt-br), ou uma permissão declarada em outro app. Seu app também pode definir as próprias permissões. As novas permissões são declaradas com o elemento [`<permission>`](https://developer.android.com/guide/topics/manifest/permission-element?hl=pt-br).

Para mais informações, consulte [Permissões no Android](https://developer.android.com/guide/topics/permissions/overview?hl=pt-br).

#### Compatibilidade do dispositivo <a href="#compatibility" id="compatibility"></a>

O arquivo de manifesto também é onde você pode declarar quais são os tipos de recursos de hardware ou software exigidos pelo app e, por extensão, com quais tipos de dispositivo ele é compatível. A Google Play Store não permite que os usuários instalem seu app em dispositivos que não oferecem os recursos ou a versão do sistema exigidos.

Há várias tags de manifesto que definem com quais dispositivos o app tem compatibilidade. Confira abaixo algumas das mais usadas.

**\<uses-feature>**

O elemento [`<uses-feature>`](https://developer.android.com/guide/topics/manifest/uses-feature-element?hl=pt-br) permite que você declare os recursos de hardware e software de que o app precisa. Por exemplo, se o app não consegue realizar funcionalidades básicas em um dispositivo que não tenha um sensor de bússola, é possível declarar o sensor como obrigatório com esta tag de manifesto:

```xml
<manifest ... >
    <uses-feature android:name="android.hardware.sensor.compass"
                  android:required="true" />
    ...
</manifest>
```

**Observação**: se você quiser disponibilizar o app em Chromebooks, há algumas limitações importantes de recursos de hardware e software que precisam ser consideradas. Para mais informações, consulte [Compatibilidade do manifesto do app para Chromebooks](https://developer.android.com/topic/arc/manifest?hl=pt-br).

**\<uses-sdk>**

Cada nova versão da plataforma geralmente adiciona novas APIs que não estavam disponíveis na versão anterior. Para indicar a versão mínima compatível com o app, o manifesto precisa incluir a tag [`<uses-sdk>`](https://developer.android.com/guide/topics/manifest/uses-sdk-element?hl=pt-br) e o atributo [`minSdkVersion`](https://developer.android.com/guide/topics/manifest/uses-sdk-element?hl=pt-br#min).

No entanto, não se esqueça de que os atributos no elemento `<uses-sdk>` são substituídos pelas propriedades correspondentes no arquivo [`build.gradle`](https://developer.android.com/studio/build?hl=pt-br#build-files). Se você estiver usando o Android Studio, especifique os valores `minSdkVersion` e `targetSdkVersion`:

```kotlin
android {
    defaultConfig {
        applicationId = "com.example.myapp"

        // Defines the minimum API level required to run the app.
        minSdkVersion(21)

        // Specifies the API level used to test the app.
        targetSdkVersion(33)
        ...
    }
}
```

Para mais informações sobre o arquivo `build.gradle`, consulte [como configurar seu build](https://developer.android.com/studio/build?hl=pt-br).

Para saber mais sobre como declarar o suporte do app para diferentes dispositivos, consulte a [Visão geral da compatibilidade do dispositivo](https://developer.android.com/guide/practices/compatibility?hl=pt-br).

### Convenções de arquivo <a href="#filec" id="filec"></a>

Esta seção descreve as convenções e regras que se aplicam geralmente a todos os elementos e atributos no arquivo de manifesto.

ElementosSó são obrigatórios os elementos [`<manifest>`](https://developer.android.com/guide/topics/manifest/manifest-element?hl=pt-br) e [`<application>`](https://developer.android.com/guide/topics/manifest/application-element?hl=pt-br). Ambos devem ocorrer somente uma vez. A maioria dos outros elementos pode ocorrer zero ou mais vezes. No entanto, alguns deles precisam estar presentes para tornar o arquivo de manifesto útil.

Todos os valores são definidos por atributos, e não como dados de caracteres dentro de um elemento.

Elementos de mesmo nível geralmente não são ordenados. Por exemplo, os elementos [`<activity>`](https://developer.android.com/guide/topics/manifest/activity-element?hl=pt-br), [`<provider>`](https://developer.android.com/guide/topics/manifest/provider-element?hl=pt-br), e [`<service>`](https://developer.android.com/guide/topics/manifest/service-element?hl=pt-br) podem ser posicionados em qualquer ordem. Há duas exceções principais a essa regra:

* Um elemento [`<activity-alias>`](https://developer.android.com/guide/topics/manifest/activity-alias-element?hl=pt-br) precisa seguir a `<activity>` de quem ele é um alias.
* O elemento `<application>` precisa ser o último elemento dentro do elemento `<manifest>`.

AtributosTecnicamente, os atributos são opcionais. No entanto, muitos deles precisam ser especificados para que um elemento possa cumprir sua finalidade. Para atributos realmente opcionais, consulte a [documentação de referência](https://developer.android.com/guide/topics/manifest/manifest-intro?hl=pt-br#reference) que indica os valores padrão.

Exceto por alguns atributos do elemento [`<manifest>`](https://developer.android.com/guide/topics/manifest/manifest-element?hl=pt-br) raiz, todos os nomes de atributo começam com o prefixo `android:`, como `android:alwaysRetainTaskState`. Como o prefixo é universal, a documentação geralmente o omite ao se referir a atributos pelo nome.

Vários valoresSe mais de um valor for especificado, o elemento sempre será repetido em vez de listar os vários valores dentro de um único elemento. Por exemplo, um filtro de intents pode listar algumas ações:

```xml
<intent-filter ... >
    <action android:name="android.intent.action.EDIT" />
    <action android:name="android.intent.action.INSERT" />
    <action android:name="android.intent.action.DELETE" />
    ...
</intent-filter>
```

Valores de recursoAlguns atributos têm valores que são mostrados aos usuários, como o título de uma atividade ou o ícone do app. O valor desses atributos pode divergir de acordo com o idioma do usuário ou outras configurações de dispositivos. Por exemplo, para fornecer um tamanho de ícone diferente com base na densidade de pixel do dispositivo. Os valores podem ser definidos com um recurso ou tema, em vez de serem codificados no arquivo de manifesto. O valor real pode mudar com base nos [recursos alternativos](https://developer.android.com/guide/topics/resources/providing-resources?hl=pt-br) fornecidos para diferentes configurações de dispositivos.

Os recursos são expressados como valores com este formato:

`"@[package:]type/name"`

É possível omitir o nome do package quando o recurso é fornecido pelo app, inclusive caso ele seja fornecido por uma dependência de biblioteca, porque os [recursos da biblioteca são mesclados aos seus](https://developer.android.com/studio/write/add-resources?hl=pt-br#resource\_merging). Quando você quiser usar um recurso do framework do Android, o único nome de pacote válido é `android`.

O type é um tipo de recurso, como [`string`](https://developer.android.com/guide/topics/resources/string-resource?hl=pt-br) ou [`drawable`](https://developer.android.com/guide/topics/resources/drawable-resource?hl=pt-br), e name é o nome que identifica o recurso específico. Confira um exemplo:

```xml
<activity android:icon="@drawable/smallPic" ... >
```

Para mais informações sobre como adicionar recursos ao projeto, leia [Visão geral dos recursos de app](https://developer.android.com/guide/topics/resources/providing-resources?hl=pt-br).

Para aplicar um valor definido em um [tema](https://developer.android.com/guide/topics/ui/look-and-feel/themes?hl=pt-br), o primeiro caractere precisa ser `?` em vez de `@`:

`"?[package:]type/name"`

Valores de stringQuando um valor do atributo é uma string, é preciso usar barras invertidas duplas (`\\`) para caracteres de escape, como `\` para uma nova linha ou `\\uxxxx` para um caractere Unicode.

### Referência de elementos do manifesto <a href="#reference" id="reference"></a>

A tabela abaixo fornece links para os documentos de referência para todos os elementos válidos no arquivo `AndroidManifest.xml`.

|                                                                                                                           |                                                                                                                                                                                                      |
| ------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [`<action>`](https://developer.android.com/guide/topics/manifest/action-element?hl=pt-br)                                 | Adiciona uma ação a um filtro de intents.                                                                                                                                                            |
| [`<activity>`](https://developer.android.com/guide/topics/manifest/activity-element?hl=pt-br)                             | Declara um componente da atividade.                                                                                                                                                                  |
| [`<activity-alias>`](https://developer.android.com/guide/topics/manifest/activity-alias-element?hl=pt-br)                 | Declara um alias para a atividade.                                                                                                                                                                   |
| [`<application>`](https://developer.android.com/guide/topics/manifest/application-element?hl=pt-br)                       | Declara o aplicativo.                                                                                                                                                                                |
| [`<category>`](https://developer.android.com/guide/topics/manifest/category-element?hl=pt-br)                             | Adiciona um nome de categoria a um filtro de intents.                                                                                                                                                |
| [`<compatible-screens>`](https://developer.android.com/guide/topics/manifest/compatible-screens-element?hl=pt-br)         | Especifica cada configuração de tela com que o aplicativo é compatível.                                                                                                                              |
| [`<data>`](https://developer.android.com/guide/topics/manifest/data-element?hl=pt-br)                                     | Adiciona uma especificação de dados a um filtro de intents.                                                                                                                                          |
| [`<grant-uri-permission>`](https://developer.android.com/guide/topics/manifest/grant-uri-permission-element?hl=pt-br)     | Especifica os subconjuntos de dados do app aos quais o provedor de conteúdo pai tem permissão de acesso.                                                                                             |
| [`<instrumentation>`](https://developer.android.com/guide/topics/manifest/instrumentation-element?hl=pt-br)               | `Instrumentation`Declara uma classe que permite monitorar a interação de um aplicativo com o sistema.                                                                                                |
| [`<intent-filter>`](https://developer.android.com/guide/topics/manifest/intent-filter-element?hl=pt-br)                   | Especifica os tipos de intents aos quais uma atividade, um serviço ou um broadcast receiver pode responder.                                                                                          |
| [`<manifest>`](https://developer.android.com/guide/topics/manifest/manifest-element?hl=pt-br)                             | O elemento raiz do arquivo `AndroidManifest.xml`.                                                                                                                                                    |
| [`<meta-data>`](https://developer.android.com/guide/topics/manifest/meta-data-element?hl=pt-br)                           | Um par de nome-valor para um item de dados extra e arbitrários que pode ser fornecido ao componente pai.                                                                                             |
| [`<path-permission>`](https://developer.android.com/guide/topics/manifest/path-permission-element?hl=pt-br)               | Define o caminho e as permissões necessárias para um subconjunto específico de dados em um provedor de conteúdo.                                                                                     |
| [`<permission>`](https://developer.android.com/guide/topics/manifest/permission-element?hl=pt-br)                         | Declara uma permissão de segurança que pode ser usada para limitar o acesso a componentes ou recursos específicos deste ou de outros aplicativos.                                                    |
| [`<permission-group>`](https://developer.android.com/guide/topics/manifest/permission-group-element?hl=pt-br)             | Declara um nome para um agrupamento lógico de permissões relacionadas.                                                                                                                               |
| [`<permission-tree>`](https://developer.android.com/guide/topics/manifest/permission-tree-element?hl=pt-br)               | Declara o nome base de uma árvore de permissões.                                                                                                                                                     |
| [`<provider>`](https://developer.android.com/guide/topics/manifest/provider-element?hl=pt-br)                             | Declara o componente de um provedor de conteúdo.                                                                                                                                                     |
| [`<queries>`](https://developer.android.com/guide/topics/manifest/queries-element?hl=pt-br)                               | Declara o conjunto de outros apps que seu app pretende acessar. Saiba mais no guia sobre [filtragem de visibilidade de pacotes](https://developer.android.com/training/package-visibility?hl=pt-br). |
| [`<receiver>`](https://developer.android.com/guide/topics/manifest/receiver-element?hl=pt-br)                             | Declara um componente do broadcast receiver.                                                                                                                                                         |
| [`<service>`](https://developer.android.com/guide/topics/manifest/service-element?hl=pt-br)                               | Declara um componente de serviço.                                                                                                                                                                    |
| [`<supports-gl-texture>`](https://developer.android.com/guide/topics/manifest/supports-gl-texture-element?hl=pt-br)       | Declara um único formato de compactação de textura GL que pode ser usado com o app.                                                                                                                  |
| [`<supports-screens>`](https://developer.android.com/guide/topics/manifest/supports-screens-element?hl=pt-br)             | Declara os tamanhos de tela com suporte do app e ativa o modo de compatibilidade da tela para telas maiores do que as com suporte.                                                                   |
| [`<uses-configuration>`](https://developer.android.com/guide/topics/manifest/uses-configuration-element?hl=pt-br)         | Indica os recursos de entrada específicos exigidos pelo aplicativo.                                                                                                                                  |
| [`<uses-feature>`](https://developer.android.com/guide/topics/manifest/uses-feature-element?hl=pt-br)                     | Declara um único recurso de hardware ou software usado pelo aplicativo.                                                                                                                              |
| [`<uses-library>`](https://developer.android.com/guide/topics/manifest/uses-library-element?hl=pt-br)                     | Especifica uma biblioteca compartilhada que precisa ser vinculada ao aplicativo.                                                                                                                     |
| [`<uses-native-library>`](https://developer.android.com/guide/topics/manifest/uses-native-library-element?hl=pt-br)       | Especifica uma biblioteca compartilhada nativa oferecida pelo fornecedor que precisa ser vinculada ao app.                                                                                           |
| [`<uses-permission>`](https://developer.android.com/guide/topics/manifest/uses-permission-element?hl=pt-br)               | Especifica uma permissão do sistema que precisa ser concedida pelo usuário para que o app funcione corretamente.                                                                                     |
| [`<uses-permission-sdk-23>`](https://developer.android.com/guide/topics/manifest/uses-permission-sdk-23-element?hl=pt-br) | Especifica que um app quer uma permissão específica, mas somente se o app estiver instalado em um dispositivo com Android 6.0 (nível 23 da API) ou mais recente.                                     |
| [`<uses-sdk>`](https://developer.android.com/guide/topics/manifest/uses-sdk-element?hl=pt-br)                             | Permite expressar a compatibilidade de um aplicativo com uma ou mais versões da plataforma Android usando um número inteiro de nível de API.                                                         |

### Exemplo de arquivo de manifesto <a href="#example" id="example"></a>

O XML abaixo é um exemplo simples de `AndroidManifest.xml` que declara duas atividades para o app.

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:versionCode="1"
    android:versionName="1.0">

    <!-- Beware that these values are overridden by the build.gradle file -->
    <uses-sdk android:minSdkVersion="15" android:targetSdkVersion="26" />

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:label="@string/app_name"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">

        <!-- This name is resolved to com.example.myapp.MainActivity
             based on the namespace property in the build.gradle file -->
        <activity android:name=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>

        <activity
            android:name=".DisplayMessageActivity"
            android:parentActivityName=".MainActivity" />
    </application>
</manifest>
```

\
