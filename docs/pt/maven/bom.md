# BOM: gerenciamento centralizado de versões

Um **BOM** (*Bill of Materials*) é um POM especializado em centralizar versões e outras informações de dependências. Ele é útil quando vários projetos precisam utilizar um conjunto coerente de componentes.

Embora a expressão `bom.xml` seja usada informalmente, no Maven 3 um BOM comum continua sendo um arquivo chamado `pom.xml`, com `packaging` igual a `pom`.

## O problema que um BOM resolve

Imagine dez aplicações Mule que usam os conectores HTTP e Sockets. Se cada projeto escolher suas versões isoladamente, surgem alguns riscos:

- aplicações equivalentes passam a usar versões diferentes;
- uma correção de vulnerabilidade exige alterações dispersas;
- combinações de versões deixam de ser testadas em conjunto;
- torna-se difícil saber quais versões foram aprovadas pela organização.

Um BOM cria uma fonte central para essas versões. Cada aplicação ainda declara os componentes que realmente utiliza, mas pode omitir as versões gerenciadas pelo BOM.

```text
BOM ── fornece versões ──► POM da aplicação
                              │
                              └── declara as dependências utilizadas
```

## `dependencies` não é `dependencyManagement`

Essa distinção é essencial:

- uma dependência em `dependencies` é adicionada ao projeto;
- uma dependência em `dependencyManagement` apenas fornece valores padrão, principalmente a versão, caso o projeto declare essa dependência.

Portanto, **importar um BOM não adiciona conectores ou bibliotecas à aplicação**.

## Exemplo de BOM para componentes Mule

O exemplo abaixo centraliza versões de dois conectores e de um módulo. Os números de versão são ilustrativos e devem ser substituídos por versões compatíveis com o Mule Runtime usado pela organização.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>SEU_ORG_ID</groupId>
    <artifactId>mulesoft-bom</artifactId>
    <version>1.0.0</version>
    <packaging>pom</packaging>
    <name>BOM de componentes Mule</name>

    <properties>
        <mule.http.connector.version>VERSAO_APROVADA</mule.http.connector.version>
        <mule.sockets.connector.version>VERSAO_APROVADA</mule.sockets.connector.version>
        <mule.apikit.version>VERSAO_APROVADA</mule.apikit.version>
    </properties>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.mule.connectors</groupId>
                <artifactId>mule-http-connector</artifactId>
                <version>${mule.http.connector.version}</version>
                <classifier>mule-plugin</classifier>
            </dependency>

            <dependency>
                <groupId>org.mule.connectors</groupId>
                <artifactId>mule-sockets-connector</artifactId>
                <version>${mule.sockets.connector.version}</version>
                <classifier>mule-plugin</classifier>
            </dependency>

            <dependency>
                <groupId>org.mule.modules</groupId>
                <artifactId>mule-apikit-module</artifactId>
                <version>${mule.apikit.version}</version>
                <classifier>mule-plugin</classifier>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>
```

O conjunto usado para localizar uma dependência gerenciada inclui `groupId`, `artifactId`, `type` e `classifier`. Por isso, dependências Mule com `classifier` igual a `mule-plugin` devem conservar esse classifier tanto no BOM quanto na aplicação consumidora.

## Disponibilizando o BOM

Para ser importado, o BOM deve estar disponível como artefato Maven. Durante testes locais, execute no diretório do projeto do BOM:

```bash
mvn install
```

Isso instala o BOM em `.m2/repository`, permitindo que projetos na mesma máquina o importem.

Para compartilhá-lo entre equipes ou usá-lo em CI/CD, publique-o em um repositório Maven remoto. O BOM e os projetos consumidores **não precisam estar no mesmo diretório**. A resolução ocorre pelas coordenadas Maven e pelos repositórios configurados.

### Publicação no Anypoint Exchange

No Exchange, Parent POMs e outros arquivos genéricos são publicados como *custom assets*. A documentação atual da MuleSoft orienta usar o `exchange-mule-maven-plugin` para esse tipo de publicação.

Uma configuração básica do plugin é:

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.mule.tools.maven</groupId>
            <artifactId>exchange-mule-maven-plugin</artifactId>
            <version>VERSAO_ATUAL_DO_PLUGIN</version>
            <inherited>false</inherited>
            <executions>
                <execution>
                    <id>exchange-pre-deploy</id>
                    <phase>validate</phase>
                    <goals>
                        <goal>exchange-pre-deploy</goal>
                    </goals>
                </execution>
                <execution>
                    <id>exchange-deploy</id>
                    <phase>deploy</phase>
                    <goals>
                        <goal>exchange-deploy</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

Use a versão indicada na [documentação de publicação do Exchange](https://docs.mulesoft.com/exchange/to-publish-assets-maven), pois requisitos e versões do plugin podem mudar.

O destino da publicação é informado em `distributionManagement`:

```xml
<distributionManagement>
    <repository>
        <id>anypoint-exchange</id>
        <name>Anypoint Exchange</name>
        <url>https://maven.anypoint.mulesoft.com/api/v3/organizations/SEU_ORG_ID/maven</url>
        <layout>default</layout>
    </repository>
</distributionManagement>
```

As credenciais não devem ser colocadas no POM. Configure-as no `settings.xml` usando o mesmo `id`:

```xml
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 https://maven.apache.org/xsd/settings-1.0.0.xsd">
    <servers>
        <server>
            <id>anypoint-exchange</id>
            <username>USUARIO_OU_CONNECTED_APP</username>
            <password>SEGREDO</password>
        </server>
    </servers>
</settings>
```

!!! warning
    O formato exato das credenciais depende do método de autenticação adotado. Em pipelines, armazene segredos no cofre do provedor de CI/CD e gere ou injete o `settings.xml` de maneira segura.

Depois de conferir as configurações e permissões, a publicação normalmente é feita com:

```bash
mvn clean deploy
```

## Importando o BOM em uma aplicação Mule

O BOM é importado dentro de `dependencyManagement` com `type` igual a `pom` e `scope` igual a `import`:

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>SEU_ORG_ID</groupId>
            <artifactId>mulesoft-bom</artifactId>
            <version>1.0.0</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

Em seguida, a aplicação declara apenas os componentes que usa. As versões vêm do BOM:

```xml
<dependencies>
    <dependency>
        <groupId>org.mule.connectors</groupId>
        <artifactId>mule-http-connector</artifactId>
        <classifier>mule-plugin</classifier>
    </dependency>

    <dependency>
        <groupId>org.mule.modules</groupId>
        <artifactId>mule-apikit-module</artifactId>
        <classifier>mule-plugin</classifier>
    </dependency>
</dependencies>
```

Se o BOM estiver no Exchange, a aplicação também precisa conseguir localizar o repositório da organização. Isso pode ser configurado no POM ou, de forma centralizada, no `settings.xml` corporativo:

```xml
<repositories>
    <repository>
        <id>anypoint-exchange</id>
        <name>Anypoint Exchange</name>
        <url>https://maven.anypoint.mulesoft.com/api/v3/organizations/SEU_ORG_ID/maven</url>
    </repository>
</repositories>
```

## BOM, Parent POM e agregador

Esses conceitos podem aparecer no mesmo arquivo, mas resolvem problemas diferentes:

| Recurso | Finalidade |
| --- | --- |
| BOM | Centraliza versões de dependências e é importado. |
| Parent POM | Compartilha configurações por herança, inclusive plugins e propriedades. |
| POM agregador | Lista módulos para construí-los em conjunto. |

Uma organização pode usar um BOM e um Parent POM separados. Essa separação costuma ser útil porque um projeto só pode ter um `parent`, mas pode importar mais de um BOM.

## Como verificar se o BOM funcionou

Execute os comandos no diretório da aplicação consumidora:

```bash
# Exibe o POM resultante, incluindo o dependencyManagement importado
mvn help:effective-pom

# Exibe as dependências e versões efetivamente resolvidas
mvn dependency:tree
```

Se o Maven informar que uma dependência não possui versão, verifique:

1. se o BOM foi localizado no repositório;
2. se ele foi importado dentro de `dependencyManagement`;
3. se `groupId`, `artifactId`, `type` e `classifier` correspondem à declaração gerenciada;
4. se a dependência realmente aparece em `dependencies` na aplicação.

## Exemplo completo com MuleSoft

Para acompanhar o fluxo de ponta a ponta — criação do BOM, publicação no Anypoint Exchange e consumo por uma aplicação Mule — consulte o [exemplo completo de BOM com MuleSoft](exemplo-bom-mulesoft.md).

## Referências

- [Mecanismo de dependências e importação de BOMs — Apache Maven](https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html)
- [Publicação de assets com Maven — MuleSoft](https://docs.mulesoft.com/exchange/to-publish-assets-maven)
- [Referência do POM — Apache Maven](https://maven.apache.org/pom.html)
