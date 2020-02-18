
# Padrões para desenvolvimento
    
- Aplicar ao máximo os princípios do S.O.L.I.D. Ver [SOLID Principles: Explanation and examples].  
- Seguir ao máximo o Git Flow. Evitar trabalhar 2 ou mais pessoas ao mesmo tempo em uma mesma branch. Ver [Git Flow].

# Criação de arquivos na pasta res

- Aplique nomes significativos aos arquivos. Ajude o colega a não precisar abrir arquivos por arquivos para saber se já existe algo que atenda. Guia de recomendaçaõ: [A successful XML naming convention]
- Nos `styles.xml` aplique nomes que descrevam o estilo. Assim o colega não precisa criar um parecido. Ex:
```
<style name="textWhite26spBold">
    <item name="android:textColor">@android:color/white</item>
    <item name="android:textSize">26sp</item>
    <item name="android:textStyle">bold</item>
</style>
```
- Quando trabalhando com layout utilizar as seguintes configurações no editor:
    - No *Theme for Preview* mudar de `Material.Light [default]` para `AppTheme`. Assim é possível visualizar como o layout ficará utilizando o tema global da aplicação.
    - Quando aplicar estilos em campos que contém textos, utilizar `style` em vez de `android:textAppearance`, pois *style* consegue herdar configurações do tema global e assim o layout fica mais fiel ao que será apresentado no dispositivo.
    - Adote um dispositivo que tenha uma resolução baixa como, por exemplo, o Nexus 5. Crie um AVD do Nexus 5 e escolha ele no editor na opção Device for Preview.
- Mais recomendações em breve...

# Fluxo de Testes
- Em breve...

# Recomendações Pré-Push

- Aplicar o [ktlint] no projeto inteiro e deixar todo o código formatado antes de subir. Códigos não formatados terão os Pull Request cancelados.
    - Rodando o [ktlint gradle] em todo o projeto:
        - Via terminal: `./gradlew ktlintFormat`
        - Via AS: ` Aba do Gradle -> root project -> Tasks -> formatting -> ktlintFormat`
    - Rodando o `ktlint` apenas no módulo modificado para ser mais rápido:
        - Via terminal: `./gradlew :features:nome-da-camada:nome-do-modulo:ktlintFormat`
        - Via AS: ` Aba do Gradle -> nome-do-modulo -> Tasks -> formatting -> ktlintFormat`
    - Obs.: **Na maioria dos casos o `ktlint` vai conseguir aplicar a formatação. Mas tem cenários que ele não consegue precisando formatar manualmente. O `ktlint` avisa quando não consegue** :D
- Aplicar o [detekt] para identificar possíveis problemas com o códgo escrito. Pull Requests sem ter aplicado o detekt serão reprovados imediatamente.
    - Rodando o [detekt gradle] em todo o projeto:
        - Via terminal: `./gradlew detekt`
        - Via AS: ` Aba do Gradle -> root project -> Tasks -> verification -> detekt`
    - Rodando o `detekt` apenas no módulo modificado para ser mais rápido:
        - Via terminal: `./gradlew :features:nome-da-camada:nome-do-modulo:detekt`
        - Via AS: ` Aba do Gradle -> nome-do-modulo -> Tasks -> verification -> detekt`
    - Obs.: **O detekt vai printar todas as informações no console. Mas se precisar ter um relatório mais amigável, o mesmo gera em vários formatos e vai imprimir o link para eles no console** :D

- #### Adicionar os pacotes de classe `*Raw.kt` dos módulos `-data` no arquivo `kotlinserialization.pro` antes de subir para produção. **Se não feito, o app vai quebrar em produção, pois a serialização não vai funcionar**

# Integração com o Sonar
- Em breve...





[SOLID Principles: Explanation and examples]: <https://itnext.io/solid-principles-explanation-and-examples-715b975dcad4>

[Git Flow]: <https://danielkummer.github.io/git-flow-cheatsheet/index.pt_BR.html>

[ktlint]: <https://github.com/pinterest/ktlint>

[ktlint gradle]: <https://github.com/JLLeitschuh/ktlint-gradle>

[detekt]: <https://arturbosch.github.io/detekt/>

[detekt gradle]: <https://github.com/arturbosch/detekt>

[A successful XML naming convention]: <https://jeroenmols.com/blog/2016/03/07/resourcenaming/>
