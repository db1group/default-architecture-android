# Configurações mínimas

- Gradle 5.4.1;
- Android Studio 3.5.0 (mesmo não tendo suporte total a Kotlin DSL);
- File -> Settings -> Editor -> Code Style ->
	- Kotlin ->
		- Set from... -> Predefined Style -> Kotlin Style guide
		- Imports ->
			- Top-level Symbols -> Use single name import
			- Java Statics and Enum members -> Use single name import
			- Packages to Use import with '*' -> Uncheck all checkbox
	- XML ->
		- Set from... -> Predefined Style -> Android

# Estrutura básica

Este projeto foi elaborado com o intuito de criar um padrão de desenvolvimento de projetos Android que seja fácil de manter, testar e que possar facilmente escalar independente do tamanho do projeto.

## Projeto com Kotlin DSL

Kotlin DSL oferece um mecanismo mais amigável para configurações de projetos Gradle, além de outros suportes como:

- A statically typed and a type-safe DSL (inherited from Kotlin);
- An enhanced IDE editing experience (inherited from Kotlin);
- Interoperability with existing scripts (as a JVM language);
- Maximum readability (as any DSL);
- Consistency and super power (by using the same language across the project — source code and configuration 💯 Kotlin);

Referências: [Migrating Android build scripts from Groovy to Kotlin DSL], [Kotlin + buildSrc for Better Gradle Dependency Management]

## Problemas com Gradle DSL

-   Very poor IDE assistant when writing a Groovy DSL script
-   Performance issues*
-   Errors at build runtime instead of compile time
-   Painful build script debugging experience
-   Refactoring is a pain in 🤬

Referência: [Migrating Android build scripts from Groovy to Kotlin DSL]

# Estrutura do projeto

Ler [Estrutura do Projeto](README-STRUCTURE.md)

[Migrating Android build scripts from Groovy to Kotlin DSL]: <https://proandroiddev.com/migrating-android-build-scripts-from-groovy-to-kotlin-dsl-f8db79dd6737>

[Kotlin + buildSrc for Better Gradle Dependency Management]: <https://handstandsam.com/2018/02/11/kotlin-buildsrc-for-better-gradle-dependency-management/>
