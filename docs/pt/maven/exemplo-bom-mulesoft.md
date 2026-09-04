# Exemplo completo: BOM com MuleSoft

Este exemplo reúne o cenário completo apresentado nos estudos:

1. um projeto Maven funciona como BOM;
2. o BOM gerencia versões de conectores, módulos e de uma especificação RAML;
3. o BOM é publicado como *custom asset* no Anypoint Exchange;
4. uma aplicação Mule importa o BOM;
5. a aplicação declara as dependências sem repetir suas versões.

Os valores abaixo são exemplos. Antes de utilizá-los, substitua:

- `SEU_ORG_ID` pelo ID da organização ou do Business Group proprietário dos assets;
- `VERSAO_APROVADA` por versões compatíveis com o Mule Runtime utilizado;
- versões do runtime e dos plugins pelas versões homologadas no seu ambiente.

## Visão geral do fluxo

```text
Projeto do BOM
    │
    ├── mvn install ──► repositório local (.m2)
    │
    └── mvn deploy ───► Anypoint Exchange
                              │
                              ▼
                       Aplicação Mule
                       importa o BOM
                              │
                              ▼
                   versões aplicadas às dependências
```

## 1. POM completo do BOM

O projeto abaixo produz apenas um POM, pois sua finalidade é fornecer gerenciamento de versões. Ele não produz uma aplicação Mule, um JAR ou um WAR.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <!-- Coordenadas do próprio BOM -->
    <groupId>SEU_ORG_ID</groupId>
    <artifactId>mulesoft-bom</artifactId>
    <version>1.0.0</version>
    <packaging>pom</packaging>
    <name>My Company MuleSoft BOM</name>

    <!-- Versões centralizadas -->
    <properties>
        <mule.http.connector.version>VERSAO_APROVADA</mule.http.connector.version>
        <mule.sockets.connector.version>VERSAO_APROVADA</mule.sockets.connector.version>
        <mule.apikit.version>VERSAO_APROVADA</mule.apikit.version>
        <weather.api.version>1.0.1</weather.api.version>
    </properties>

    <!-- Plugin usado para publicar o POM como custom asset no Exchange -->
    <build>
        <plugins>
            <plugin>
                <groupId>org.mule.tools.maven</groupId>
                <artifactId>exchange-mule-maven-plugin</artifactId>
                <version>0.1.7</version>
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

    <!-- Fornece versões; não adiciona dependências aos consumidores -->
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
                <groupId>SEU_ORG_ID</groupId>
                <artifactId>weather</artifactId>
                <version>${weather.api.version}</version>
                <classifier>raml</classifier>
                <type>zip</type>
            </dependency>

            <dependency>
                <groupId>org.mule.modules</groupId>
                <artifactId>mule-apikit-module</artifactId>
                <version>${mule.apikit.version}</version>
                <classifier>mule-plugin</classifier>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <!-- Repositórios usados para resolver dependências -->
    <repositories>
        <repository>
            <id>anypoint-exchange</id>
            <name>Anypoint Exchange</name>
            <url>https://maven.anypoint.mulesoft.com/api/v3/organizations/SEU_ORG_ID/maven</url>
            <layout>default</layout>
            <releases>
                <enabled>true</enabled>
            </releases>
            <snapshots>
                <enabled>true</enabled>
            </snapshots>
        </repository>

        <repository>
            <id>mulesoft-releases</id>
            <name>MuleSoft Releases Repository</name>
            <url>https://repository.mulesoft.org/nexus/repository/releases/</url>
            <layout>default</layout>
            <releases>
                <enabled>true</enabled>
            </releases>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </repository>
    </repositories>

    <!-- Repositório usado para resolver plugins Maven da MuleSoft -->
    <pluginRepositories>
        <pluginRepository>
            <id>mulesoft-plugin-releases</id>
            <name>MuleSoft Plugin Releases Repository</name>
            <url>https://repository.mulesoft.org/nexus/repository/releases/</url>
            <layout>default</layout>
            <releases>
                <enabled>true</enabled>
            </releases>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </pluginRepository>
    </pluginRepositories>

    <!-- Destino usado por mvn deploy -->
    <distributionManagement>
        <repository>
            <id>anypoint-exchange</id>
            <name>Anypoint Exchange</name>
            <url>https://maven.anypoint.mulesoft.com/api/v3/organizations/SEU_ORG_ID/maven</url>
            <layout>default</layout>
        </repository>
    </distributionManagement>
</project>
```

### O papel de cada bloco

| Bloco | Responsabilidade |
| --- | --- |
| Coordenadas GAV | Identificam o próprio BOM no repositório. |
| `packaging` | Com o valor `pom`, indica que o artefato produzido é um POM. |
| `properties` | Dá nomes reutilizáveis às versões aprovadas. |
| `dependencyManagement` | Gerencia as versões oferecidas aos projetos consumidores. |
| `build/plugins` | Executa a preparação e a publicação do custom asset. |
| `repositories` | Localiza dependências e assets usados durante o build. |
| `pluginRepositories` | Localiza plugins que não estão no Maven Central. |
| `distributionManagement` | Define o destino para o qual `mvn deploy` envia o BOM. |

O `exchange-pre-deploy` é vinculado à fase `validate` para preparar o asset no Exchange. O `exchange-deploy`, vinculado à fase `deploy`, conclui a publicação. `inherited=false` evita que projetos que usem esse POM como parent herdem acidentalmente o processo de publicação do BOM.

!!! note "Sobre o `maven-jar-plugin` do documento original"
    O exemplo original usava uma propriedade `type=custom` e o `maven-jar-plugin` com o classifier `custom`. Essa propriedade não é o elemento Maven `packaging` e, isoladamente, não muda o tipo do projeto. Para publicar um POM/Parent POM como custom asset, a documentação atual da MuleSoft apresenta o `exchange-mule-maven-plugin` diretamente. Por isso, ambos foram removidos desta versão corrigida: um projeto com `packaging=pom` não precisa gerar um JAR para que seu POM seja publicado.

## 2. Credenciais no `settings.xml`

O valor de `distributionManagement/repository/id` deve corresponder ao `server/id` do arquivo `settings.xml`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 https://maven.apache.org/xsd/settings-1.0.0.xsd">
    <servers>
        <server>
            <id>anypoint-exchange</id>
            <username>USUARIO_OU_TIPO_DE_CREDENCIAL</username>
            <password>SEGREDO</password>
        </server>
    </servers>
</settings>
```

O arquivo costuma ficar em `~/.m2/settings.xml` no Linux e macOS ou em `C:\Users\SEU_USUARIO\.m2\settings.xml` no Windows.

As credenciais não devem ser versionadas. Para autenticação por token ou Connected App, siga o formato indicado na [documentação oficial do Exchange](https://docs.mulesoft.com/exchange/to-publish-assets-maven).

## 3. Instalando ou publicando o BOM

Para disponibilizar o BOM somente na máquina atual:

```bash
mvn clean install
```

Para publicá-lo no Exchange configurado em `distributionManagement`:

```bash
mvn clean deploy
```

`install` grava o artefato no repositório local. `deploy` percorre as fases anteriores e o envia ao repositório remoto. Portanto, `deploy` não é sinônimo de implantar uma aplicação em um runtime; no contexto Maven, a fase publica o artefato no repositório configurado.

## 4. POM completo da aplicação Mule consumidora

O exemplo a seguir representa a aplicação `eapi-weather`. Ela importa o BOM e utiliza os conectores HTTP e Sockets, a especificação RAML `weather` e o módulo APIkit sem declarar versões localmente.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.mycompany</groupId>
    <artifactId>eapi-weather</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <packaging>mule-application</packaging>
    <name>eapi-weather</name>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <app.runtime>VERSAO_DO_MULE_RUNTIME</app.runtime>
        <mule.maven.plugin.version>VERSAO_DO_MULE_MAVEN_PLUGIN</mule.maven.plugin.version>
    </properties>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-clean-plugin</artifactId>
                <version>VERSAO_COMPATIVEL</version>
            </plugin>

            <plugin>
                <groupId>org.mule.tools.maven</groupId>
                <artifactId>mule-maven-plugin</artifactId>
                <version>${mule.maven.plugin.version}</version>
                <extensions>true</extensions>
            </plugin>
        </plugins>
    </build>

    <!-- Importação do BOM: fornece as versões gerenciadas -->
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

    <!-- Dependências realmente utilizadas pela aplicação -->
    <dependencies>
        <dependency>
            <groupId>org.mule.connectors</groupId>
            <artifactId>mule-http-connector</artifactId>
            <classifier>mule-plugin</classifier>
        </dependency>

        <dependency>
            <groupId>org.mule.connectors</groupId>
            <artifactId>mule-sockets-connector</artifactId>
            <classifier>mule-plugin</classifier>
        </dependency>

        <dependency>
            <groupId>SEU_ORG_ID</groupId>
            <artifactId>weather</artifactId>
            <classifier>raml</classifier>
            <type>zip</type>
        </dependency>

        <dependency>
            <groupId>org.mule.modules</groupId>
            <artifactId>mule-apikit-module</artifactId>
            <classifier>mule-plugin</classifier>
        </dependency>
    </dependencies>

    <repositories>
        <repository>
            <id>anypoint-exchange</id>
            <name>Anypoint Exchange</name>
            <url>https://maven.anypoint.mulesoft.com/api/v3/organizations/SEU_ORG_ID/maven</url>
            <layout>default</layout>
            <releases>
                <enabled>true</enabled>
            </releases>
            <snapshots>
                <enabled>true</enabled>
            </snapshots>
        </repository>

        <repository>
            <id>mulesoft-releases</id>
            <name>MuleSoft Releases Repository</name>
            <url>https://repository.mulesoft.org/nexus/repository/releases/</url>
            <layout>default</layout>
            <releases>
                <enabled>true</enabled>
            </releases>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </repository>
    </repositories>

    <pluginRepositories>
        <pluginRepository>
            <id>mulesoft-plugin-releases</id>
            <name>MuleSoft Plugin Releases Repository</name>
            <url>https://repository.mulesoft.org/nexus/repository/releases/</url>
            <layout>default</layout>
            <releases>
                <enabled>true</enabled>
            </releases>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </pluginRepository>
    </pluginRepositories>
</project>
```

### O que cada dependência fornece

| Dependência | Papel na aplicação |
| --- | --- |
| `mule-http-connector` | Disponibiliza operações como HTTP Listener e HTTP Request. |
| `mule-sockets-connector` | Permite comunicação por sockets, como TCP e UDP. |
| `weather` | Disponibiliza a especificação RAML publicada no Exchange como ZIP. |
| `mule-apikit-module` | Auxilia no roteamento e na validação de requisições com base na especificação da API. |

O `mule-maven-plugin`, com `extensions=true`, estende o Maven para reconhecer o `packaging` `mule-application` e participar da validação e do empacotamento da aplicação. O `maven-clean-plugin` remove saídas de builds anteriores, normalmente o diretório `target`, quando `mvn clean` é executado.

## 5. Como o Maven encontra cada item

É importante não confundir as três seções de repositório:

| Seção | Usada para |
| --- | --- |
| `repositories` | Baixar dependências do projeto, inclusive BOM, conectores e RAML. |
| `pluginRepositories` | Baixar plugins de build, como plugins fornecidos pela MuleSoft. |
| `distributionManagement` | Publicar o artefato produzido pelo projeto. |

Uma configuração corporativa pode centralizar repositórios e mirrors no `settings.xml`. Nesse caso, nem todas essas declarações precisam ser repetidas nos POMs.

## 6. Conferindo o resultado

No diretório da aplicação Mule, execute:

```bash
mvn help:effective-pom
mvn dependency:tree
```

O primeiro comando permite verificar se o conteúdo de `dependencyManagement` do BOM foi incorporado ao POM efetivo. O segundo mostra as dependências e versões resolvidas.

Uma verificação útil é procurar, no POM efetivo ou na árvore, as versões de:

- `mule-http-connector`;
- `mule-sockets-connector`;
- `weather`;
- `mule-apikit-module`.

## 7. Correções em relação ao documento original

- O BOM é um `pom.xml`, não um formato Maven chamado `bom.xml`.
- O BOM não adiciona dependências; ele gerencia informações usadas quando a aplicação as declara.
- O BOM e a aplicação não precisam estar no mesmo nível de diretório. Depois de instalado ou publicado, o BOM é localizado por suas coordenadas GAV.
- `repositories`, `pluginRepositories` e `distributionManagement` possuem finalidades diferentes.
- O classifier e o type fazem parte da correspondência de uma dependência gerenciada; por isso foram mantidos nos dois POMs.
- Versões do Mule Runtime, conectores e plugins não foram apresentadas como universais. Elas devem ser validadas para o ambiente real.
- IDs de organização foram substituídos por placeholders para evitar que valores específicos sejam tratados como configuração reutilizável.
- O plugin de publicação foi ajustado ao exemplo atual da documentação oficial para custom assets e Parent POMs.

## Referências

- [Publicar assets usando Maven — MuleSoft](https://docs.mulesoft.com/exchange/to-publish-assets-maven)
- [Mecanismo de dependências e BOM — Apache Maven](https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html)
- [Mule Maven Plugin — MuleSoft](https://docs.mulesoft.com/mule-runtime/latest/mmp-concept)
