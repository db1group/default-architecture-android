
# Estrutura do projeto  
  
O projeto está estruturado para garantir uma boa escalabilidade e fornecer suporte futuro para recursos como [Dynamic Delivery] que ajudam a reduzir o tamanho do app e entregar features sobre demanda. Para facilitar encontrar as coisas, projetos que são features tem sufixos `-ui`, `-domain` e `-data` e projetos que tem coisas compartilháveis para outros projetos tem o prefixo `shared-`. Com isso, outros projetos podem escolher o que utilizar referenciando coisas específicas e melhorando o build do projeto.

> **Aviso**: esse projeto não nasceu do zero. Algumas das melhores práticas, estruturas e comportamentos foram obtidas combinando outras referências da internet. Todas elas serão referenciadas durante toda a documentação.
> "Na natureza nada se cria, nada se perde, tudo se transforma." - **Antoine Lavoisier**

## buildSrc

**Projeto com as configurações de todos os outros projetos**.
*A estrutura de buildSrc foi baseada no [Norris] com algumas modificações*.

 - **configs** - Pacote com arquivos que contém configurações de projeto Android, Kotlin, etc.
    - `AndroidConfig.kt` - Configurações para projetos Android. Ver [Configure your build].
    - `AndroidNavigationConfig.kt` - Configurações do Android Navigation.
    - `FlavorConfig.kt` - Configurações de product flavors. Ver [Configure build variants].
    - `KotlinConfig.kt` - Configurações do Kotlin como versão e JVM target.
    - `ProguardConfig.kt` - Configurações de proguard. Usada para aplicar arquivos proguard nos projetos.
    - `SigningConfig.kt` - Configurações de assinatura usadas para gerar a versão de produção do aplicativo.
 - **dependencies** - Pacote com as dependências e configurações para desenvolvimento e testes.
    - `Libraries.kt` - Contém as dependências de desenvolvimento e testes usadas no projeto.
    - `UnitTestDependencies.kt` - Classe helper que ajudar a adicionar dependências de testes unitários nos módulos usando uma lista pré-definida de dependências.
    - `InstrumentationTestsDependencies.kt` - Classe helper que ajudar a adicionar dependências de testes instrumentados nos módulos Android usando uma lista pré-definida de dependências.
 - **modules** - Pacotes com os nomes dos módulos existentes no projeto.
    - `ProjectModules.kt` - Contém os nomes dos módulos existentes no projeto. Todas vez que um módulo for adicionado ao projeto é necessário registrar nesse object. 😛
    - `LibraryType.kt` - Uma sealed class usada para carregar configurações específicas de cada módulo.
    - `LibraryModule.kt` - Classe helper que carrega as configurações do módulo que estão em arquivos `.gradle` com base no tipo informado pela `LibraryType`.
 - `BuildPlugins.kt` - Arquivos com os plugins que determinam o tipo de cada módulo e os que são usados para desenvolvimento e testes.
 - `Version.kt` - Uma data class usada para auxiliar na geração do número de versão.
 - `Versioning.kt` - Contém a lógica para versionamento do app. Usada para gerar o `versionCode` e `versionName` do aplicativo.

## shared-core

**Projeto Android Library com conteúdos source compartilhados por todos os outros módulos**.
*Geralmente todos os outros módulos Android terão referência para esse módulo*.

Esse módulo tem apenas códigos Android usados em outros módulos. Aqui não contém os resources (drawables, strings, etc) deixa o projeto responsável apenas por fazer build de códigos.

## shared-data

**Projeto Kotlin com conteúdos de data compartilhados por todos os outros módulos**.
*Apenas módulos de data terão referência direta/indireta para esse módulo*.

## shared-domain

**Projeto Kotlin com conteúdos de domínio compartilhados por todos os outros módulos**.
*Geralmente todos os outros módulos domain, data e UI terão referência direta/indireta para esse módulo*.

## shared-network

**Projeto Kotlin com recursos para consumir API. A ideia é centralizar recursos que são necessários para consumir API Rest**.
*Apenas módulos que consomem API ou configuração de DI terão referência para esse projeto*.

## shared-resources

**Projeto Android Library com conteúdos de resources compartilhados por todos os outros módulos**.
*Geralmente todos os outros módulos de UI terão referência para esse módulo*.

Esse módulo tem apenas arquivos de resources (drawable, strings, etc.) que são configurações para todo o projeto. Está separado pois em alguns módulos apenas o resources são necessários e vice versa com o *shared-core*.

## Adicionando um novo módulo

### Módulo Kotlin

*Esse módulo deveria ser criado apenas quando os conteúdos NÃO tem dependência do framework Android*.

 1. `File` -> `New` -> `New Module...` -> `Java Library`
 2. Se o módulo tem conteúdo compartilhados para outros módulos, adicionar o prefixo `shared-` no campo `Library name`. Se precisa criar o módulo diretamente em uma pasta, seguir a nomeclatura com o prefixo sendo o nome da pasta. Do contrário, o AS cria na pasta root por padrão.
 Ex: Criando um módulo data: `:features:data:login-data`
 Ex: Criando um módulo domain: `:features:domain:login-domain`
**Escrevendo dessa forma no Library Name o AS cria o projeto automaticamente na pasta features/data ou features/domain**.
 3. O AS vai criar o arquivo `settings.gradle`, pois ele ainda não tem suporte a Kotlin DSL. **EXCLUIR O ARQUIVO**.
 4. Abrir o arquivo `ProjectModules.kt` e criar uma variável para armazenar o novo módulo criado. O nome do módulo sempre começa com `:`. Ex: `:shared-mymodule`. Se o módulo estiver em uma pasta, usar a nomeclatura explicada no passo 2.
 5. Abrir o arquivo `settings.gradle.kts` e no `include()` adicionar o novo módulo registrado no `ProjectModules.kt` feito no passo anterior.
 6. Nesse momento o **AS nem sincroniza o projeto** por causa do suporte limitado a Kotlin DSL. Feche o projeto (não a IDE, pois não precisa), exclua a pasta `.idea` e abra o projeto novamente que o AS vai carregar o novo módulo.
 7. Agora, renomeie o `build.gradle` do novo módulo para `build.gradle.kts` e aplique a seguinte configuração:

```
import dependencies.Libraries
import dependencies.UnitTestDependencies.Companion.unitTest
import modules.LibraryModule
import modules.LibraryType

// Carrega as configuracoes padroes para modulos Kotlin
val module = LibraryModule(rootDir, LibraryType.Kotlin)
// Aplica a configuracao no modulo corrente
apply(from = module.script())

plugins {
  // Necessario aplicar o plugin especifico para o projeto
  id(PluginIds.kotlinJVM)
}

dependencies {
  implementation(Libraries.kotlinStdlib)
  // Outras dependencias necessárias ...

  // Rotina para aplicar dependencias para testes unitarios
  unitTest {
    forEachDependency { testImplementation(it) }
  }
}
```

### Módulo Android Library

*Esse módulo deveria ser criado para compartilhar recursos Android para outros projetos. Esse projeto vira um `.aar` que funciona como um `.jar`. Assim, não ter um módulo só para conter todos os recursos pois pode aumentar o tamanho do apk final*.

 1. `File` -> `New` -> `New Module...` -> `Android Library`
 2. Se o módulo tem conteúdo compartilhados para outros módulos, adicionar o prefixo `shared-` no campo `Module name`. Se precisa criar o módulo diretamente em uma pasta, seguir a nomeclatura com o prefixo sendo o nome da pasta. Do contrário, o AS cria na pasta root por padrão.
 Ex: Criando um módulo ui: `:features:ui:login-ui`
**Escrevendo dessa forma no Module Name o AS cria o projeto automaticamente na pasta features/ui**.
 3. Não use `_` ou `-` no `Package name`. Também não junte nomes no pacote. Separe por pontos. Manter o padrão no nomes de pacotes é essencial.
 4. O AS vai criar os arquivos `build.gradle`(não é o referente ao módulo. É que ele não reconhece o arquivo global) e o `settings.gradle`, pois ele ainda não tem suporte a Kotlin DSL. **EXCLUIR AMBOS**.
 5. Abrir o arquivo `ProjectModules.kt` e criar uma variável para armazenar o novo módulo criado. O nome do módulo sempre começa com `:`. Ex: `:shared-mymodule`. Se o módulo estiver em uma pasta, usar a nomeclatura explicada no passo 2.
 6. Abrir o arquivo `settings.gradle.kts` e no `include()` adicionar o novo módulo registrado no `ProjectModules.kt`
 7. Agora, renomeie o `build.gradle` do novo módulo para `build.gradle.kts` e aplique a configuração abaixo
 8. Geralmente, para módulos Android, o AS consegue sincronizar sem dificuldades. Mas, se o AS não sincronizar, realizar o passo 6 da seção de adicionar um módulo Kotlin.

```
import dependencies.InstrumentationTestsDependencies.Companion.instrumentationTest
import dependencies.Libraries
import dependencies.UnitTestDependencies.Companion.unitTest
import modules.LibraryModule
import modules.LibraryType

// Carrega as configuracoes padroes para modulos Android Library
val module = LibraryModule(rootDir, LibraryType.Android)
// Aplica a configuracao no modulo corrente
apply(from = module.script())

plugins {
  // Necessario aplicar o plugin especifico para o projeto
  id(PluginIds.androidLibrary)
}

dependencies {
  implementation(Libraries.kotlinStdlib)
  implementation(Libraries.appCompatX)
  // Outras dependencias ...

  // Rotina para aplicar dependencias para testes unitarios
  unitTest {
    forEachDependency { testImplementation(it) }
  }

  // Rotina para aplicar dependencias para testes instrumentados
  instrumentationTest {
    forEachDependency { androidTestImplementation(it) }
  }
}
```

### Módulo Android Dynamic Feature  (NÃO APLICADO NO MOMENTO)

*Cada feature do projeto é ou será um módulo dynamic feature. Mesmo não entregando funcionalidades sob demanda, o projeto já estará adequado com o suporte.

 1. `File` -> `New` -> `New Module...` -> `Dynamic Feature Module`
 2. Criar o módulo diretamente em uma pasta e seguir a nomeclatura com o prefixo sendo o nome da pasta. Do contrário, o AS cria na pasta root por padrão.
 Ex: Criando um módulo ui: `:features:ui:login-ui`
**Escrevendo dessa forma no Module Name o AS cria o projeto automaticamente na pasta features/ui**.
 3. Não use `_` ou `-` no `Package name`. Também não junte nomes no pacote. Separe por pontos. Manter o padrão no nomes de pacotes é essencial.
 4. Em `Module Title` coloque um nome amigável. Esse nome é apresentado para o usuário final caso seja fornecido sob demanda
 5. Em `Install-time inclusion` escolha `Include module at install-time` já que **ainda** não terá suporta para entregas sob demanda
 6. Mesmo com `minSdk 21`, deixe a opção `Fusing` marcada
 7. O AS vai criar os arquivos `build.gradle`(não é o referente ao módulo. É que ele não reconhece o arquivo global) e o `settings.gradle`, pois ele ainda não tem suporte a Kotlin DSL. **EXCLUIR AMBOS**.
 8. Abrir o arquivo `ProjectModules.kt` e criar uma variável para armazenar o novo módulo criado. O nome do módulo sempre começa com `:`. Ex: `:features-myfeature`. Se o módulo estiver em uma pasta, usar a nomeclatura explicada no passo 2.
 9. Abrir o arquivo `settings.gradle.kts` e no `include()` adicionar o novo módulo registrado no `ProjectModules.kt`
 10. Abrir o arquivo `AndroidConfig.kt`  e adicionar o módulo na lista de `dynamicFeatures`. Aqui basta usar o mesmo procecdimento aplicado no passo anterior.
 11. Agora, renomeie o `build.gradle` do novo módulo para `build.gradle.kts` e aplique a configuração abaixo
 12. Geralmente, para módulos Android, o AS consegue sincronizar sem dificuldades. Mas, se o AS não sincronizar, realizar o passo 6 da seção de adicionar um módulo Kotlin.
  
```  
import dependencies.InstrumentationTestsDependencies.Companion.instrumentationTest  
import dependencies.Libraries  
import dependencies.UnitTestDependencies.Companion.unitTest  
import modules.LibraryModule  
import modules.LibraryType  
import modules.ProjectModules  
  
// Carrega as configuracoes padroes para modulos Dynamic Feature  
val module = LibraryModule(rootDir, LibraryType.DynamicFeature)  
// Aplica a configuracao no modulo corrente  
apply(from = module.script())  
  
plugins {  
    // Necessario aplicar o plugin especifico para o projeto  
    id(PluginIds.androidDynamicFeature)  
}  
  
dependencies {  
    implementation(Libraries.kotlinStdlib)  
    implementation(Libraries.appCompatX)  
    // Outras dependencias ...  
  
    unitTest {  
        forEachDependency { testImplementation(it) }  
    }  
  
    instrumentationTest {  
        forEachDependency { androidTestImplementation(it) }  
    }  
}  
```  






[Migrating Android build scripts from Groovy to Kotlin DSL]: <https://proandroiddev.com/migrating-android-build-scripts-from-groovy-to-kotlin-dsl-f8db79dd6737>  
  
[Dynamic Delivery]: <https://developer.android.com/studio/projects/dynamic-delivery>  
  
[Norris]: <https://github.com/dotanuki-labs/norris>  
  
[Configure your build]: <https://developer.android.com/studio/build>  
  
[Configure build variants]: <https://developer.android.com/studio/build/build-variants>