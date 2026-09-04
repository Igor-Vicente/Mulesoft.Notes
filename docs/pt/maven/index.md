# Maven

O **Apache Maven** é uma ferramenta de automação de build e gerenciamento de projetos. Ele é muito utilizado no ecossistema Java e também participa do processo de construção e publicação de aplicações Mule.

Em termos simples, o Maven recebe uma descrição do projeto, normalmente armazenada no arquivo `pom.xml`, e executa tarefas como:

- baixar as bibliotecas necessárias;
- compilar o código;
- executar testes;
- empacotar a aplicação;
- disponibilizar o artefato para outros projetos ou ambientes.

O Maven é uma ferramenta de **build**. Ele pode ser executado em uma pipeline de CI/CD, mas não é, por si só, uma plataforma de CI/CD.

## Por que utilizar Maven?

Sem uma ferramenta de build, cada projeto pode adotar comandos, diretórios e maneiras diferentes de localizar bibliotecas. O Maven reduz essa variação por meio de convenções.

Se um projeto segue essas convenções, outro desenvolvedor consegue reconhecer rapidamente sua estrutura e executar o build com poucos comandos. Isso torna os builds mais previsíveis tanto na máquina do desenvolvedor quanto em uma pipeline.

## Instalação

Antes de instalar o Maven, instale um JDK compatível com o projeto e configure a variável `JAVA_HOME`.

No Windows:

1. Baixe o arquivo binário compactado na [página oficial do Maven](https://maven.apache.org/download.cgi).
2. Descompacte-o em um diretório permanente, por exemplo `C:\\tools\\apache-maven`.
3. Adicione o diretório `bin` do Maven à variável de ambiente `Path`.
4. Abra um novo terminal e verifique a instalação:

```powershell
mvn --version
```

O resultado deve mostrar, entre outras informações, as versões do Maven e do Java que estão sendo utilizadas. Consulte também o [guia oficial de instalação](https://maven.apache.org/install.html).

!!! tip "Prefira o Maven Wrapper quando o projeto o fornecer"
    Arquivos como `mvnw`, `mvnw.cmd` e `.mvn/wrapper` permitem que a equipe use a versão de Maven definida pelo projeto. Nesse caso, no Windows, execute `./mvnw.cmd package` em vez de `mvn package`.

## Estrutura convencional de um projeto

Um projeto Java Maven costuma ter a seguinte estrutura:

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
- `target`: saída gerada pelo build. Normalmente não deve ser versionada.

Projetos Mule possuem uma estrutura própria, reconhecida pelo Mule Maven Plugin, mas continuam usando o `pom.xml` e os conceitos centrais do Maven.

## O que é o `pom.xml`?

POM significa **Project Object Model**. O `pom.xml` descreve a identidade do projeto, suas dependências e a forma como o build deve ocorrer.

Um POM mínimo pode ser escrito assim:

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

As três coordenadas abaixo identificam o artefato. Elas são frequentemente chamadas de **GAV**:

| Elemento | Finalidade | Exemplo |
| --- | --- | --- |
| `groupId` | Identifica a organização ou o grupo responsável | `com.example` |
| `artifactId` | Identifica o projeto ou artefato dentro do grupo | `meu-projeto` |
| `version` | Identifica uma versão específica do artefato | `1.0.0-SNAPSHOT` |

`SNAPSHOT` indica uma versão ainda em desenvolvimento. Uma versão sem esse sufixo normalmente representa uma entrega estável e imutável.

Outras seções comuns do POM são:

- `properties`: valores reutilizáveis, como versões e codificação;
- `dependencies`: componentes efetivamente usados pelo projeto;
- `dependencyManagement`: regras e versões para dependências que poderão ser usadas;
- `build/plugins`: ferramentas que participam do build;
- `repositories`: repositórios nos quais procurar dependências;
- `pluginRepositories`: repositórios nos quais procurar plugins;
- `distributionManagement`: destino usado para publicar o artefato.

!!! note
    Nem todas essas seções precisam aparecer em todos os projetos. O Maven herda configurações do Super POM e pode receber outras configurações de um Parent POM.

## Dependências

Uma dependência é um componente externo de que o projeto precisa para compilar, testar ou executar. Em vez de armazenar manualmente cada arquivo JAR no projeto, declaramos suas coordenadas no POM:

```xml
<dependencies>
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
        <version>2.17.0</version>
    </dependency>
</dependencies>
```

A versão acima é apenas ilustrativa. Em um projeto real, escolha uma versão compatível e mantida.

O Maven também pode resolver **dependências transitivas**. Se a biblioteca A depende da biblioteca B, o Maven normalmente inclui B sem que ela precise ser declarada novamente. Ainda assim, dependências usadas diretamente pelo código devem ser declaradas explicitamente; isso documenta o projeto e evita que uma mudança na árvore transitiva quebre o build.

### Escopos mais comuns

O `scope` informa em quais classpaths e etapas uma dependência é necessária:

| Escopo | Uso principal |
| --- | --- |
| `compile` | Compilação e execução. É o valor padrão. |
| `provided` | Necessária para compilar, mas fornecida em execução pelo JDK ou contêiner. |
| `runtime` | Necessária para executar, mas não para compilar. |
| `test` | Utilizada apenas na compilação e execução de testes. |

O escopo `import` tem um propósito diferente: ele só pode ser usado, com uma dependência do tipo `pom`, dentro de `dependencyManagement` para importar um [BOM](bom.md).

## Repositórios

O Maven trabalha principalmente com dois tipos de repositório:

- **local**: cache existente na máquina do usuário;
- **remoto**: serviço do qual artefatos são baixados ou para o qual são publicados, como Maven Central, Nexus, Artifactory ou Anypoint Exchange.

Por padrão, o repositório local fica em:

```text
# Linux e macOS
~/.m2/repository

# Windows
C:\Users\SEU_USUARIO\.m2\repository
```

Ao precisar de uma dependência, o Maven procura primeiro no repositório local. Se ela não estiver disponível, consulta os repositórios remotos configurados e armazena o resultado localmente.

Os artefatos são organizados pelas coordenadas Maven. Por exemplo:

```text
~/.m2/repository/
└── com/fasterxml/jackson/core/jackson-databind/2.17.0/
    ├── jackson-databind-2.17.0.jar
    └── jackson-databind-2.17.0.pom
```

Use o modo offline somente quando as dependências e os plugins necessários já estiverem disponíveis localmente:

```bash
mvn -o package
```

!!! warning "Evite apagar todo o repositório local como primeira tentativa"
    A pasta `.m2/repository` é um cache compartilhado por vários projetos. Se houver suspeita de um download corrompido, remova apenas o diretório do artefato afetado e execute o build novamente.

## Ciclos de vida, fases e goals

O Maven possui três ciclos de vida nativos:

- `default`: constrói e publica o projeto;
- `clean`: remove arquivos gerados por builds anteriores;
- `site`: gera a documentação do projeto.

As principais fases do ciclo `default` são:

```text
validate → compile → test → package → verify → install → deploy
```

| Fase | Resultado esperado |
| --- | --- |
| `validate` | Verifica se o projeto possui as informações necessárias. |
| `compile` | Compila o código-fonte principal. |
| `test` | Executa os testes unitários. |
| `package` | Gera o artefato, como JAR, WAR ou pacote Mule. |
| `verify` | Executa verificações adicionais sobre o pacote e testes de integração. |
| `install` | Copia o artefato para o repositório Maven local. |
| `deploy` | Publica o artefato em um repositório remoto. |

As fases são cumulativas dentro do mesmo ciclo. Portanto:

```bash
mvn install
```

executa `validate`, `compile`, `test`, `package`, `verify` e, por fim, `install`, incluindo as fases intermediárias não exibidas nessa lista resumida.

Uma **fase** é uma etapa do ciclo de vida. Um **goal** é uma tarefa específica fornecida por um plugin. No comando abaixo, `clean` e `package` são fases, enquanto `dependency:tree` é um goal:

```bash
mvn clean dependency:tree package
```

## Plugins

O Maven coordena o build, mas são os plugins que executam o trabalho concreto. Por exemplo, plugins podem compilar código, executar testes ou criar um pacote.

Em um projeto Mule, o `mule-maven-plugin` acrescenta ao Maven o suporte necessário para reconhecer e construir tipos de empacotamento como `mule-application`:

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.mule.tools.maven</groupId>
            <artifactId>mule-maven-plugin</artifactId>
            <version>${mule.maven.plugin.version}</version>
            <extensions>true</extensions>
        </plugin>
    </plugins>
</build>
```

## Comandos úteis

```bash
# Exibe as versões do Maven e do Java
mvn --version

# Remove a saída do build anterior e cria um novo pacote
mvn clean package

# Mostra a árvore de dependências resolvida
mvn dependency:tree

# Mostra o POM efetivo após heranças, perfis e valores padrão
mvn help:effective-pom

# Mostra os perfis disponíveis e quais estão ativos
mvn help:active-profiles
```

## Próximo passo

Depois de compreender POM, dependências e repositórios, veja como um [BOM centraliza versões de dependências](bom.md), inclusive em aplicações Mule.

## Referências

- [Documentação do Apache Maven](https://maven.apache.org/guides/)
- [Introdução ao POM](https://maven.apache.org/guides/introduction/introduction-to-the-pom.html)
- [Ciclo de vida do build](https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html)
- [Mecanismo de dependências](https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html)
- [Repositórios Maven](https://maven.apache.org/guides/introduction/introduction-to-repositories.html)
