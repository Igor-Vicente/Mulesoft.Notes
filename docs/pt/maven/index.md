# Maven

[Apache Maven](https://maven.apache.org/) é uma ferramenta de automação de _build_ e gerenciamento de projetos, usada principalmente no ecossistema Java. Ela padroniza como um projeto é compilado, testado, empacotado e publicado.

O Maven é uma ferramenta de _build_. Ele pode ser usado em uma pipeline de CI/CD, mas não é, por si só, uma plataforma de CI/CD.

## O que o Maven faz na prática

- **Gerencia dependências:** no `pom.xml`, você declara as bibliotecas e versões necessárias. O Maven baixa essas bibliotecas — inclusive suas dependências transitivas — de repositórios como o Maven Central.
- **Padroniza o ciclo de build:** compilação, testes, empacotamento, instalação local e publicação seguem fases bem definidas, como `compile`, `test`, `package`, `install` e `deploy`.
- **Adota convenção sobre configuração:** usa uma estrutura de diretórios conhecida, como `src/main/java`, `src/test/java` e `src/main/resources`. Isso torna projetos Maven mais previsíveis para quem os mantém.
- **Centraliza a configuração:** o arquivo `pom.xml` reúne informações do projeto, dependências, plugins, versão, repositórios e regras de build.

Antes do Maven, era comum que cada projeto adotasse um processo próprio para compilar, organizar bibliotecas e gerar artefatos. As convenções do Maven reduzem essa inconsistência.

## Instalação

Consulte as [instruções oficiais de instalação](https://maven.apache.org/install.html).

No Windows, o processo básico é:

1. Instalar um JDK compatível com o projeto e configurar a variável de ambiente `JAVA_HOME`.
2. Baixar e descompactar a distribuição do Maven no diretório desejado.
3. Adicionar o diretório `bin` à variável de ambiente `Path`, por exemplo: `C:\apache-maven-3.8.8\bin`.
4. Abrir um novo terminal e validar a instalação:

```bash
mvn --version
```

!!! tip "Prefira o Maven Wrapper quando o projeto o fornecer"
Arquivos como `mvnw`, `mvnw.cmd` e `.mvn/wrapper` permitem que a equipe use a versão de Maven definida pelo projeto. No Windows, execute `./mvnw.cmd package` em vez de `mvn package`.

## Estrutura convencional de um projeto

Um projeto Java Maven normalmente segue esta estrutura:

```text
meu-projeto/
├── pom.xml
└── src/
    ├── main/
    │   ├── java/
    │   └── resources/
    └── test/
        ├── java/
        └── resources/
```

- `src/main/java`: código-fonte da aplicação;
- `src/main/resources`: arquivos de configuração e outros recursos;
- `src/test/java`: código dos testes;
- `src/test/resources`: recursos usados pelos testes;
- `target`: saída gerada pelo build; normalmente não deve ser versionada.

Projetos Mule possuem uma estrutura própria, reconhecida pelo Mule Maven Plugin, mas continuam usando o `pom.xml` e os conceitos centrais do Maven.

## O papel do `pom.xml`

O `pom.xml` (_Project Object Model_) é o arquivo central de um projeto Maven. Ele descreve o que o projeto é, de quais componentes precisa e como deve ser construído. Sem esse arquivo, o Maven não tem as informações necessárias para executar o build do projeto.

### Informações que o POM declara

1. **Identidade do projeto:** as coordenadas GAV — `groupId`, `artifactId` e `version` — identificam unicamente um artefato Maven.
2. **Dependências:** bibliotecas que devem estar disponíveis no classpath do projeto.
3. **Build:** plugins, _goals_, compilação, testes e empacotamento, como `jar`, `war` ou `pom`.
4. **Herança e composição:** um POM pode herdar configurações de um `parent` e agregar projetos em `modules`.
5. **Gerenciamento de versões:** `dependencyManagement` pode centralizar versões, inclusive pela importação de um BOM.

Em termos simples, o Maven é o motor do build; o `pom.xml` é o conjunto de instruções que informa o que construir, quais componentes usar e quais regras seguir.

### Elementos mais comuns

| Elemento       | Finalidade                                                                       |
| -------------- | -------------------------------------------------------------------------------- |
| `groupId`      | Identifica a organização ou o grupo responsável, por exemplo `com.minhaempresa`. |
| `artifactId`   | Nome técnico do projeto ou artefato.                                             |
| `version`      | Versão do artefato.                                                              |
| `dependencies` | Bibliotecas efetivamente usadas pela aplicação.                                  |
| `plugins`      | Ferramentas usadas no build, nos testes, no empacotamento ou na publicação.      |
| `properties`   | Valores reutilizáveis, como a versão do Java.                                    |
| `repositories` | Locais onde o Maven procura dependências.                                        |
| `build`        | Configurações do processo de construção.                                         |

Uma estrutura simplificada é:

```xml
<project>
    <!-- Identificação do projeto -->
    <properties>
        <!-- Valores reutilizáveis -->
    </properties>
    <build>
        <plugins>
            <!-- Ferramentas utilizadas pelo Maven -->
        </plugins>
    </build>
    <dependencies>
        <!-- Componentes utilizados pela aplicação -->
    </dependencies>
    <repositories>
        <!-- Locais para procurar dependências -->
    </repositories>
    <pluginRepositories>
        <!-- Locais para procurar plugins Maven -->
    </pluginRepositories>
</project>
```

Um POM mínimo, por sua vez, precisa da versão do modelo e das coordenadas do projeto:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>meu-projeto</artifactId>
    <version>1.0.0-SNAPSHOT</version>
</project>
```

O sufixo `SNAPSHOT` indica uma versão em desenvolvimento; uma versão sem esse sufixo normalmente representa uma entrega estável e imutável.

Ao executar `mvn package`, o Maven lê o `pom.xml`, resolve as dependências, compila o código, executa os testes e gera o artefato final — normalmente um `.jar` ou `.war`.

Em resumo: o POM declara a aplicação e suas necessidades; os plugins definem como construí-la; as dependências fornecem os componentes utilizados; e os repositórios indicam onde encontrar esses componentes.

## Dependências

Dependências são bibliotecas externas, ou seja, código de terceiros de que o projeto precisa para compilar ou executar. Em vez de implementar do zero funcionalidades comuns — como ler JSON, conectar a um banco de dados ou fazer chamadas HTTP — uma aplicação pode utilizar bibliotecas já existentes.

Por exemplo, para trabalhar com JSON em Java, uma aplicação pode usar Jackson:

```xml
<dependencies>
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
        <version>2.17.0</version>
    </dependency>
</dependencies>
```

O Maven localiza o JAR correspondente em um repositório remoto, como o Maven Central, e o disponibiliza para o projeto.

O Maven também resolve dependências transitivas. Se uma biblioteca depende de outra, essa segunda biblioteca normalmente também é obtida. Ainda assim, declare explicitamente no POM as dependências que seu código usa diretamente.

### Escopos mais comuns

O `scope` informa **em que momento uma dependência deve estar disponível**. Pense em três momentos diferentes:

- **compilação:** quando o Java transforma o código-fonte em classes;
- **execução:** quando a aplicação já empacotada está rodando;
- **testes:** quando os testes são compilados e executados.

Por exemplo, Jackson normalmente é usado tanto pelo código quanto pela aplicação em execução; por isso, seu escopo padrão é `compile`. Já o JUnit é usado para testar, mas não deve fazer parte da aplicação entregue; por isso, usa o escopo `test`.

| Escopo | Disponível em | Quando usar |
| --- | --- | --- |
| `compile` | Compilação, execução e testes. | É o padrão. Use para bibliotecas que o código da aplicação importa e utiliza em produção, como Jackson. |
| `provided` | Compilação e testes; não em execução. | Use quando o ambiente fornece a biblioteca em produção, como uma API fornecida pelo servidor de aplicações. |
| `runtime` | Execução e testes; não na compilação do código principal. | Use quando a aplicação não importa diretamente a API, mas precisa da implementação ao rodar, como determinados drivers JDBC. |
| `test` | Somente compilação e execução dos testes. | Use para ferramentas de teste, como JUnit e Mockito. Não acompanha o artefato de produção. |

Exemplo com JUnit:

```xml
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter</artifactId>
    <version>VERSAO_COMPATIVEL</version>
    <scope>test</scope>
</dependency>
```

Assim, o Maven disponibiliza o JUnit para `mvn test`, mas não o inclui como dependência necessária para executar a aplicação.

O escopo `import` é diferente dos demais: ele não adiciona uma biblioteca ao classpath. Só pode ser usado com uma dependência do tipo `pom`, dentro de `dependencyManagement`, para importar as versões gerenciadas por um BOM.

### Repositório local

Depois do primeiro download, o Maven guarda os artefatos em um repositório local e os reutiliza em outros projetos que precisem da mesma versão:

| Sistema operacional | Local padrão                          |
| ------------------- | ------------------------------------- |
| Linux e macOS       | `~/.m2/repository`                    |
| Windows             | `C:\Users\SEU_USUARIO\.m2\repository` |

Os arquivos são organizados pelas coordenadas GAV. Para o exemplo do Jackson, o caminho é semelhante a:

```text
~/.m2/repository/com/fasterxml/jackson/core/jackson-databind/2.17.0/
├── jackson-databind-2.17.0.jar
├── jackson-databind-2.17.0.pom
└── ...
```

Em outras palavras: `groupId` (com pontos convertidos em diretórios) → `artifactId` → `version` → arquivos do artefato.

### Como a resolução funciona

1. Você executa um comando, como `mvn compile` ou `mvn install`.
2. O Maven lê o `pom.xml` para identificar as dependências necessárias.
3. Para cada dependência, verifica primeiro o repositório local.
4. Se ela não estiver disponível localmente, busca-a em um repositório remoto configurado e a armazena no diretório `.m2`.
5. Quando o artefato já existe localmente, ele é reutilizado, sem novo download.

Além do Maven Central, organizações podem utilizar repositórios como Nexus, Artifactory e Anypoint Exchange.

Todo projeto Maven resolve as dependências declaradas quando necessário. Em modo _offline_ (`mvn -o ...`), porém, o Maven só usa o que já existe no repositório local e falha se um artefato estiver ausente.

!!! tip
Em caso de dependência corrompida ou conflito de resolução, inspecione a pasta específica do artefato em `~/.m2/repository`. Evite apagar todo o cache sem necessidade.

## Ciclo de vida

O ciclo de vida (_lifecycle_) do Maven é a sequência padronizada de fases que um projeto percorre durante o build: da validação inicial à publicação do artefato. Assim, os projetos adotam um fluxo previsível para compilar, testar e empacotar.

### Os três ciclos de vida

| Ciclo     | Finalidade                                                                            |
| --------- | ------------------------------------------------------------------------------------- |
| `default` | Constrói e publica o projeto: compilação, testes, empacotamento, instalação e deploy. |
| `clean`   | Remove artefatos gerados por builds anteriores.                                       |
| `site`    | Gera a documentação ou o site do projeto.                                             |

Cada ciclo tem suas próprias fases. O `default` é o mais usado no dia a dia e, por isso, costuma receber mais atenção. As fases principais dos outros dois ciclos são:

| Ciclo | Fases | Resultado |
| --- | --- | --- |
| `clean` | `pre-clean` → `clean` → `post-clean` | Remove a saída de builds anteriores; normalmente, a fase `clean` remove o diretório `target`. |
| `site` | `pre-site` → `site` → `post-site` → `site-deploy` | Prepara, gera, finaliza e, quando configurado, publica a documentação do projeto. |

Ao executar uma fase, o Maven também executa as fases anteriores **do mesmo ciclo**. Por exemplo, `mvn clean` executa `pre-clean` e `clean`; `mvn site` executa `pre-site` e `site`. No uso diário, o ciclo `default` é o mais frequente.

### Fases do ciclo `default`

| Fase       | O que faz                                                                                          |
| ---------- | -------------------------------------------------------------------------------------------------- |
| `validate` | Verifica se o projeto está correto e se as informações necessárias, como o POM, estão disponíveis. |
| `compile`  | Compila o código-fonte.                                                                            |
| `test`     | Executa testes unitários, sem empacotar o projeto.                                                 |
| `package`  | Empacota o código compilado no formato definido, como `.jar` ou `.war`.                            |
| `verify`   | Executa verificações adicionais, geralmente relacionadas a testes de integração.                   |
| `install`  | Instala o pacote no repositório local, tornando-o disponível para outros projetos locais.          |
| `deploy`   | Publica o pacote em um repositório remoto, para consumo por outros times ou projetos.              |

As fases são cumulativas. Ao pedir uma fase, o Maven executa todas as anteriores na ordem. Portanto:

```bash
mvn install
```

executa `validate` → `compile` → `test` → `package` → `verify` → `install`.

Exemplos práticos:

- `mvn compile`: compila o projeto;
- `mvn test`: compila e executa os testes;
- `mvn package`: compila, testa e gera o `.jar` ou `.war`;
- `mvn install`: realiza o processo anterior e instala o artefato em `~/.m2/repository`;
- `mvn deploy`: realiza o processo anterior e publica o artefato em um repositório remoto.

Uma **fase** é uma etapa do ciclo de vida. Um **goal** é uma tarefa específica fornecida por um plugin. No exemplo abaixo, `clean` e `package` são fases, enquanto `dependency:tree` é um goal:

```bash
mvn clean dependency:tree package
```

## Plugins e comandos úteis

O Maven coordena o build, mas os plugins realizam o trabalho concreto, como compilar, testar e criar pacotes. O `mule-maven-plugin`, por exemplo, permite que o Maven reconheça e construa projetos com `packaging` igual a `mule-application`.

```bash
# Remove a saída do build anterior e cria um novo pacote
mvn clean package

# Exibe a árvore de dependências resolvida
mvn dependency:tree

# Exibe o POM efetivo, incluindo heranças, perfis e valores padrão
mvn help:effective-pom

# Exibe os perfis disponíveis e aqueles ativos
mvn help:active-profiles
```

## Maven em aplicações MuleSoft

Em aplicações Mule, o `pom.xml` também define o `packaging` como `mule-application`, usa o `mule-maven-plugin` e declara conectores, módulos e especificações de API como dependências. Um BOM pode ser importado em `dependencyManagement` para centralizar as versões desses componentes.

O exemplo completo contém:

- configuração do `mule-maven-plugin` e do `maven-clean-plugin`;
- importação de um BOM publicado no Anypoint Exchange;
- dependências para HTTP Connector, Sockets Connector, APIkit e uma especificação RAML;
- repositórios de dependências, de plugins e configuração de publicação;
- explicações de cada seção do POM e correções aplicadas ao documento de origem.

Consulte [BOM e gerenciamento de versões](bom.md) para entender a centralização de versões e o [exemplo completo com MuleSoft](exemplo-bom-mulesoft.md) para o POM comentado da aplicação e do BOM.

## Referências

- [Apache Maven](https://maven.apache.org/)
- [Instalação do Maven](https://maven.apache.org/install.html)
- [Referência do POM](https://maven.apache.org/pom.html)
- [Mecanismo de dependências do Maven](https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html)
