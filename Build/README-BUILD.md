
# Configurações de Build

Como todo projeto Android em sua maioria, este projeto está configurado e usando o Gradle como ferramenta de gerenciamento de projeto.

## Realizando um Clean no projeto

Caso seja necessário realizar um clean em todos os módulos, removendo as pastas `build`, execute: `./gradlew clean`
Isso limparar os caches existentes nas pastas `build`. Caso precise limpar o cache de um módulo apenas execute: `./gradlew :path:para:o:modulo:clean`. Se não sabe o path para o módulo, veja o `ProjectModules.kt`.

## Validando formatação de códigos

O projeto utiliza `ktlint` para manter o código formatado e padronizado. Para validar se o padrão está sendo mantido execute a ação: `./gradlew ktlintCheck`. Caso precise checar apenas módulos específicos execute: `./gradlew :path:para:o:modulo:ktlintCheck`. Se não sabe o path para o módulo, veja o `ProjectModules.kt`.

## Verificando possíveis problemas nos códigos

O projeto utiliza `detekt` como ferramenta de análise de código. Ela ajuda a identificar possíveis problemas e sugerir melhores práticas na escrita dos códigos. Para validar execute: `./gradlew detekt`. Caso precise checar apenas módulos específicos execute: `./gradlew :path:para:o:modulo:detekt`. Se não sabe o path para o módulo, veja o `ProjectModules.kt`.

## Tipos de build do projeto

O projeto hoje suporta 2 Flavors e 3 tipos de build que podem ser combinados para gerar versões específicas do projeto.

- Flavors
    - `dev` - contém configurações relacionadas ao ambiente de desenvolvimento.
    - `prod` - contém configurações relacionadas ao ambiente de produção.
- Build Types
    - `debug` - contém configurações que são usadas durante o desenvolvimento, desabilitando coisas que só precisam ser habilitadas em produção.
    - `release` - contém configurações que são usadas em produção, minificando e ofuscando o bytecode e aplicando a chave de geração de apk (necessária para gerar uma versão de produção e subir na loja).
    - `staging` - mescla configurações de debug e release. Usado para testar como se comportaria a versão do app em produção. **Tem coisas que só são aplicadas em release como proguard e minify, por isso é bom ter uma versão com as configurações de release e mesclando coisas de debug para poder rastrear problemas.**

## Gerando versões...

Abaixo segue os scripts combinando os tipos de build do projeto e os Flavors

### ... de Desenvolvimento
```
./gradlew :app:assembleDevDebug
```

### ... de Desenvolvimento e sem suporte a debug
Nesse cenário, o bytecode está ofuscado dificultando realizar engenharia reversa e identificar possíveis problemas
```
./gradlew :app:assembleDevRelease
```

### ... de Desenvolvimento, com suporte a debug mas com bytecode ofuscado
Nesse cenário, o bytecode está ofuscado, dificultando realizar engenharia reversa e identificar possíveis problemas, mas com a possibilidade de realizar debug do app. Isso ajuda a identificar problemas que aconteceriam em produção e joga os problemas encontrados no crashlytics do ambiente de desenvolvimento ;P
```
./gradlew :app:assembleDevStaging
```

### ... de Produção (esse é a versão final que vai para a loja)
Nesse cenário, o bytecode está ofuscado dificultando realizar engenharia reversa e identificar possíveis problemas
```
./gradlew :app:assembleProdRelease
```

### ... de Produção e com suporte a debug
Nesse cenário, o bytecode não está ofuscado, pois aplica as configurações de debug
```
./gradlew :app:assembleProdDebug
```

### ... de Produção, com suporte a debug mas com bytecode ofuscado
Nesse cenário, o bytecode está ofuscado, dificultando realizar engenharia reversa e identificar possíveis problemas, mas com a possibilidade de realizar debug do app. Isso ajuda a identificar problemas que aconteceriam em produção e joga os problemas encontrados no crashlytics do ambiente de desenvolvimento ;P
```
./gradlew :app:assembleProdStaging
```

### ... de QA com versão de produção
Nesse cenário, o bytecode está ofuscado dificultando realizar engenharia reversa e identificar possíveis problemas
```
./gradlew :app:assembleQaRelease
```

### ... de QA e com suporte a debug
Nesse cenário, o bytecode não está ofuscado, pois aplica as configurações de debug
```
./gradlew :app:assembleQaDebug
```

### ... de QA, com suporte a debug mas com bytecode ofuscado
Nesse cenário, o bytecode está ofuscado, dificultando realizar engenharia reversa e identificar possíveis problemas, mas com a possibilidade de realizar debug do app. Isso ajuda a identificar problemas que aconteceriam em produção e joga os problemas encontrados no crashlytics do ambiente de desenvolvimento ;P
```
./gradlew :app:assembleQaStaging
```

### ... de *nome_do_produto* com versão de produção
Nesse cenário, o bytecode está ofuscado dificultando realizar engenharia reversa e identificar possíveis problemas
```
./gradlew :app:assemble*nome_do_produto*Release
```

### ... de *nome_do_produto* e com suporte a debug
Nesse cenário, o bytecode não está ofuscado, pois aplica as configurações de debug
```
./gradlew :app:assemble*nome_do_produto*Debug
```

### ... de *nome_do_produto*, com suporte a debug mas com bytecode ofuscado
Nesse cenário, o bytecode está ofuscado, dificultando realizar engenharia reversa e identificar possíveis problemas, mas com a possibilidade de realizar debug do app. Isso ajuda a identificar problemas que aconteceriam em produção e joga os problemas encontrados no crashlytics do ambiente de desenvolvimento ;P
```
./gradlew :app:assemble*nome_do_produto*Staging
```

## Fluxo básico de CI

Abaixo seguem fluxos básicos de execução nas etapas de CI. Lembrando que é necessário ter o SDK Tools instalado e com a versão igual ao compileSdk definido no projeto.

**Obs 1**.: No momento de escrita desse README, a versão do SDK Tools era `4333796` e pode ser encontrada na [Página de Downloads do Site Oficial], bastando colocar nas variáveis de ambiente: `ANDROID_SDK_TOOLS = 4333796`.

**Obs 2**.: Para otimizar o build, realize cache dos arquivos do Gradle como explicado aqui: [Cache your Gradle files]

```
before_script:
    - export ANDROID_COMPILE_SDK=`egrep 'const val compileSdkVersion' buildSrc/src/main/java/configs/AndroidConfig.kt | awk '{print $NF}'`
    - echo $ANDROID_SDK_TOOLS
    - echo $ANDROID_COMPILE_SDK
    - wget --quiet --output-document=/tmp/sdk-tools-linux.zip https://dl.google.com/android/repository/sdk-tools-linux-${ANDROID_SDK_TOOLS}.zip
    - unzip /tmp/sdk-tools-linux.zip -d .android
    - export ANDROID_HOME=$PWD/.android
    - export PATH=$PATH:$PWD/.android/platform-tools/
    - echo y | .android/tools/bin/sdkmanager "platforms;android-${ANDROID_COMPILE_SDK}"
    - chmod +x ./gradlew

```

### Build com Pull Request

A cada Pull Request para a branch de desenvolvimento `develop`, deveria ser executado o seguinte fluxo:

```
script:
    - ./gradlew clean
    - ./gradlew ktlintCheck
    - ./gradlew detekt
    - ./gradlew test
    - # Algum dos comandos na seção **Gerando Versões...**
```

### Build com Tag

A cada Tag gerada a partir da branch de desenvolvimento `develop`, deveria ser executado o seguinte fluxo:

```
script:
    - ./gradlew :app:clean
    - ./gradlew :app:assembleProdRelease     # VERIFICAR OBSERVAÇÃO ABAIXO
```

**Obs**.: Verificar a possibilidade de identificar por tag e subir versões para os respectivos públicos como:

- `qa0.0.1` - para versões testadas pelo público de QA
    - `./gradlew assembleQaStaging appDistributionUploadQaStaging`
- `pg0.0.1` - para versões testadas pelo público da *nome_do_produto*
    - `./gradlew assemble*nome_do_produto*Release appDistributionUpload*nome_do_produto*Release`
- `v0.0.1` - para versões diretamente em produção
    - `./gradlew assembleProdRelease upload-to-play-store`





[Página de Downloads do Site Oficial]: <https://developer.android.com/studio#downloads>

[Cache your Gradle files]: <https://medium.com/@chrisbanes/circleci-cache-key-over-many-files-c9e07f4d471a>
