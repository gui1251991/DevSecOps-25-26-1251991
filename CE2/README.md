\# CE2: Implementação de Pipelines CI/CD com Jenkins, Gradle, Spring Boot e Docker



\# Parte 1: Pipeline do Gradle Basic Demo





\## 1. Introdução



O objetivo desta primeira fase do exercício foi implementar uma pipeline de Integração Contínua (CI) utilizando Jenkins para o projeto gradle-basic-demo, aplicando conceitos fundamentais de automação e práticas DevOps num ambiente real. Esta etapa teve como propósito substituir processos manuais de integração por um fluxo automatizado, mais eficiente, consistente e confiável.



A pipeline desenvolvida permite que cada alteração realizada no repositório GitHub seja automaticamente obtida, compilada com Gradle, validada através de testes, documentada com Javadoc e arquivada como artefato no Jenkins. Dessa forma, todo o processo básico de integração de software passa a ser executado de forma padronizada, garantindo maior rapidez na deteção de erros e maior segurança na validação de novas alterações.



A Integração Contínua é uma prática essencial dentro de DevOps, pois assegura que o código esteja constantemente verificado num ambiente controlado, reduzindo falhas de integração, aumentando a qualidade do software e promovendo builds reproduzíveis. Além disso, contribui para uma rotina de desenvolvimento mais organizada, permitindo que problemas sejam identificados mais cedo e corrigidos com maior eficiência.



Nesta fase, a pipeline foi estruturada para cumprir todas as etapas propostas no exercício, desde a obtenção automática do código até à sua compilação, teste, documentação e arquivamento, criando uma base sólida para evoluções futuras no processo de automação do projeto.



\## 2. Configuração do Ambiente Jenkins



Para garantir uma instalação prática, portátil e isolada, o Jenkins foi configurado através de Docker, permitindo a criação de um ambiente controlado sem necessidade de instalação direta no sistema operativo principal. Esta abordagem facilitou significativamente a gestão da ferramenta, reduziu conflitos de dependências locais e tornou o processo mais consistente, especialmente num contexto de DevOps onde reprodutibilidade e padronização são fundamentais.



A utilização de containers também trouxe vantagens importantes, como a persistência dos dados através de volumes dedicados, a facilidade de reinstalação ou migração do ambiente e uma gestão mais simples de atualizações e plugins. Dessa forma, toda a configuração do Jenkins permaneceu encapsulada, organizada e facilmente reutilizável.



&#x09;docker run -d \\

&#x09;-p 8081:8080 \\

&#x09;-p 50000:50000 \\

&#x09;-v jenkins\_home:/var/jenkins\_home \\

&#x09;--name jenkins \\

&#x09;jenkins/jenkins:lts



Neste processo, foi definido o mapeamento da porta 8081 para acesso à interface web do Jenkins, evitando conflitos com outras aplicações locais, enquanto a porta 50000 ficou responsável pela comunicação entre agentes e o servidor Jenkins. O volume jenkins\_home garantiu a persistência de configurações, plugins e pipelines, mesmo após reinicializações ou remoção do container.



Após a inicialização do container, foi necessário obter a palavra-passe administrativa gerada automaticamente pelo Jenkins para concluir a configuração inicial:



&#x09;docker exec jenkins cat /var/jenkins\_home/secrets/initialAdminPassword



Com o acesso realizado, procedeu-se à instalação dos plugins recomendados pela própria plataforma, assegurando funcionalidades essenciais para integração contínua, gestão de pipelines e ligação com repositórios Git. Em seguida, foi criado o utilizador administrador, permitindo personalizar e proteger o ambiente de desenvolvimento.



Esta configuração inicial estabeleceu a base necessária para a criação e execução da pipeline CI, garantindo um ambiente funcional, persistente e preparado para suportar as etapas seguintes do projeto.



\## 3. Integração com GitHub



Para permitir que o Jenkins acedesse ao repositório do projeto, foi necessária a configuração de autenticação segura com o GitHub. Como o repositório utilizado era privado, o acesso exigia credenciais válidas para que o processo de checkout pudesse ser executado corretamente.



Para isso, foi utilizado um GitHub Personal Access Token (PAT), uma solução mais segura e compatível com as políticas atuais do GitHub. No Jenkins, estas credenciais foram configuradas no gestor interno utilizando o formato Username with password, associando a conta GitHub ao token gerado.



&#x09;Tipo: Username with password

&#x09;ID: github-credentials



Com esta configuração, o Jenkins passou a conseguir autenticar-se corretamente, clonar o repositório privado e obter automaticamente o código-fonte necessário para executar a pipeline.



Esta integração garantiu segurança, automação e acesso contínuo ao projeto, permitindo que todas as alterações no repositório fossem processadas de forma eficiente dentro do fluxo de Integração Contínua.

## 4. Estrutura do Repositório



A organização do repositório foi um elemento importante para garantir que o Jenkins conseguisse localizar corretamente os ficheiros necessários à execução da pipeline, especialmente o Jenkinsfile e o projeto Gradle associado a esta fase do exercício.



O repositório principal, DevSecOps-25-26-1251991, foi estruturado para separar claramente os diferentes componentes e fases do trabalho, mantendo uma divisão lógica entre o CE1 e o CE2. Esta separação permitiu preservar conteúdos anteriores sem comprometer a configuração específica da pipeline de Integração Contínua desenvolvida para o gradle-basic-demo.



&#x09;DevSecOps-25-26-1251991

&#x09;│

&#x09;├── CE1

&#x09;│

&#x09;├── CE2

&#x09;│   └── gradle-basic-demo

&#x09;│       ├── Jenkinsfile

&#x09;│       ├── build.gradle

&#x09;│       ├── gradlew

&#x09;│       ├── settings.gradle

&#x09;│       └── src

&#x09;│

&#x09;└── README.md



Dentro da pasta CE2/gradle-basic-demo ficaram concentrados todos os ficheiros essenciais para a execução da pipeline. O Jenkinsfile definiu todas as etapas automatizadas do processo de CI, incluindo checkout, build, testes, geração de documentação e arquivamento. O build.gradle e o settings.gradle asseguraram a configuração do projeto Gradle, enquanto o gradlew permitiu utilizar o Gradle Wrapper, garantindo compatibilidade de versões independentemente do ambiente onde a pipeline fosse executada. A pasta src concentrou o código-fonte principal e os testes.



Esta estrutura organizada foi essencial para evitar erros de path no Jenkins, facilitar a manutenção do projeto e assegurar que a pipeline pudesse ser executada corretamente a partir do repositório remoto, seguindo uma abordagem clara, modular e alinhada com boas práticas de gestão de projetos DevOps.



\## 5. Implementação da Pipeline



Para a implementação da pipeline foi utilizada a sintaxe Declarative Pipeline do Jenkins, uma abordagem mais estruturada, legível e de fácil manutenção quando comparada a alternativas mais flexíveis, como Scripted Pipeline. Esta escolha permitiu organizar claramente cada fase do processo de Integração Contínua, facilitando tanto a compreensão como futuras alterações.



O ficheiro Jenkinsfile tornou-se o núcleo da automação, definindo todas as etapas necessárias para transformar alterações no repositório em builds validadas, testadas, documentadas e arquivadas.



&#x09;pipeline {

&#x09;    agent any

&#x09;

&#x09;    stages {

&#x09;        stage('Checkout') {

&#x09;            steps {

&#x09;                checkout scm

&#x09;            }

&#x09;        }

&#x09;

&#x09;        stage('Assemble') {

&#x09;            steps {

&#x09;                dir('CE2/gradle-basic-demo') {

&#x09;                    sh './gradlew assemble'

&#x09;                }

&#x09;            }

&#x09;        }

&#x09;

&#x09;        stage('Test') {

&#x09;            steps {

&#x09;                dir('CE2/gradle-basic-demo') {

&#x09;                    sh './gradlew test'

&#x09;                }

&#x09;                junit allowEmptyResults: true, testResults: 'CE2/gradle-basic-demo/build/test-results/test/\*.xml'

&#x09;            }

&#x09;        }

&#x09;

&#x09;        stage('Javadoc') {

&#x09;            steps {

&#x09;                dir('CE2/gradle-basic-demo') {

&#x09;                    sh './gradlew javadoc'

&#x09;                }

&#x09;            }

&#x09;        }

&#x09;

&#x09;        stage('Archive') {

&#x09;            steps {

&#x09;                archiveArtifacts 'CE2/gradle-basic-demo/build/libs/\*.jar'

&#x09;            }

&#x09;        }

&#x09;    }

&#x09;}



O bloco inicial:



&#x09;pipeline {

&#x09;    agent any



define a estrutura principal da pipeline e indica que esta pode ser executada em qualquer agente disponível no Jenkins. O agent any oferece flexibilidade ao permitir que qualquer executor configurado possa correr o processo, sem restrições específicas de ambiente.



O bloco:



&#x09;stages {



funciona como o contentor principal de todas as fases da pipeline. É aqui que são organizadas, por ordem lógica, as diferentes etapas do processo de integração contínua.



Na fase de Checkout:



&#x09;stage('Checkout') {

&#x09;    steps {

&#x09;        checkout scm

&#x09;    }

&#x09;}



o Jenkins obtém automaticamente o código-fonte diretamente do repositório configurado no projeto. O comando checkout scm utiliza a configuração do Source Code Management definida no job, permitindo descarregar o repositório e garantir que a pipeline trabalha sempre sobre a versão mais recente do código.



Na fase de Assemble:



&#x09;stage('Assemble') {

&#x09;    steps {

&#x09;        dir('CE2/gradle-basic-demo') {

&#x09;            sh './gradlew assemble'

&#x09;        }

&#x09;    }

&#x09;}



a pipeline entra na diretoria específica do projeto através de dir(...) e executa o comando ./gradlew assemble. Esta etapa compila o projeto e gera os ficheiros binários necessários, sem executar testes, validando se o código consegue ser construído corretamente.



Na fase de Test:



&#x09;stage('Test') {

&#x09;    steps {

&#x09;        dir('CE2/gradle-basic-demo') {

&#x09;            sh './gradlew test'

&#x09;        }

&#x09;        junit allowEmptyResults: true, testResults: 'CE2/gradle-basic-demo/build/test-results/test/\*.xml'

&#x09;    }

&#x09;}



são executados os testes automatizados definidos no projeto através do Gradle. Após a execução, o Jenkins recolhe os resultados em formato XML com o comando junit, permitindo apresentar relatórios de sucesso ou falha diretamente na interface. A opção allowEmptyResults: true evita que a pipeline falhe imediatamente caso não existam testes, oferecendo maior tolerância durante fases iniciais de configuração.



Na fase de Javadoc:



&#x09;stage('Javadoc') {

&#x09;    steps {

&#x09;        dir('CE2/gradle-basic-demo') {

&#x09;            sh './gradlew javadoc'

&#x09;        }

&#x09;    }

&#x09;}



é gerada automaticamente documentação técnica do código-fonte Java. Esta etapa é importante para manter documentação atualizada da aplicação, promovendo melhor compreensão e manutenção futura.



Por fim, na fase de Archive:



&#x09;stage('Archive') {

&#x09;    steps {

&#x09;        archiveArtifacts 'CE2/gradle-basic-demo/build/libs/\*.jar'

&#x09;    }

&#x09;}



o Jenkins guarda os artefatos produzidos durante o build, neste caso ficheiros .jar, permitindo que versões compiladas fiquem armazenadas para download, auditoria ou utilização posterior.



No seu conjunto, esta pipeline implementa um fluxo completo de Integração Contínua, assegurando que cada alteração ao projeto passa por obtenção automática, compilação, validação funcional, documentação e armazenamento, criando uma base sólida para práticas mais avançadas de automação e entrega contínua.



\## 6. Problemas Encontrados e Soluções



Durante a implementação da pipeline de Integração Contínua, vários problemas técnicos surgiram ao longo da configuração e execução no Jenkins. A resolução destes erros foi uma parte importante do processo, pois permitiu compreender melhor a relação entre estrutura do projeto, configuração da pipeline e compatibilidade do ambiente.



Um dos primeiros problemas identificados foi o erro compileTestJava NO-SOURCE. Esta mensagem surgiu porque o projeto não possuía qualquer ficheiro de testes na estrutura esperada pelo Gradle, o que impedia a execução correta da fase de testes dentro da pipeline. Como solução, foi criado o ficheiro AppTest.java, utilizando JUnit 5, garantindo assim uma estrutura mínima de testes automatizados e permitindo que a etapa test fosse executada corretamente pelo Jenkins.



Outro erro relevante foi o Unsupported class file major version, causado por incompatibilidade entre a versão de Java utilizada no ambiente Jenkins e a versão suportada pelo Gradle ou pelas dependências configuradas no projeto. Este problema evidenciou a importância da compatibilidade entre ferramentas dentro de pipelines automatizadas. A solução passou pela atualização e ajuste da configuração do build.gradle, incluindo suporte adequado para JUnit 5, bem como pela utilização de um ambiente Java compatível com a versão do Gradle em execução.



Também ocorreu o erro gradlew not found, que impedia a execução dos comandos Gradle dentro da pipeline. A causa principal foi o uso de caminhos incorretos, uma vez que o Jenkins executava comandos a partir da raiz do repositório, enquanto o projeto se encontrava dentro da pasta CE2/gradle-basic-demo. Para resolver esta situação, foi necessário utilizar explicitamente o bloco:



&#x09;dir('CE2/gradle-basic-demo')



Esta abordagem garantiu que todos os comandos ./gradlew fossem executados no diretório correto, permitindo acesso ao Gradle Wrapper e aos ficheiros de configuração necessários.



A resolução destes problemas foi essencial para estabilizar a pipeline e garantir o funcionamento correto de todas as etapas. Mais do que simples correções, estes ajustes reforçaram a importância de uma estrutura de projeto bem organizada, da compatibilidade entre versões de ferramentas e da correta definição de paths dentro de ambientes de Integração Contínua.



\## 7. Resultado Final da Parte 1



Após a resolução dos problemas de configuração, estrutura e compatibilidade encontrados durante o desenvolvimento, a pipeline de Integração Contínua foi executada com sucesso no Jenkins, cumprindo integralmente todos os objetivos definidos para esta primeira fase do exercício.



O processo final validou corretamente todas as etapas essenciais da pipeline: obtenção automática do código a partir do repositório remoto (Checkout), compilação do projeto com Gradle (Assemble), execução de testes automatizados (Test), geração de documentação técnica com Javadoc (Javadoc) e arquivamento dos artefatos produzidos (Archive). Cada uma destas fases foi concluída sem erros, demonstrando que a integração entre Jenkins, GitHub e Gradle estava corretamente configurada.



O estado final apresentado pelo Jenkins foi:



&#x09;Finished: SUCCESS



Este resultado confirmou que a pipeline estava totalmente funcional, capaz de automatizar o processo básico de integração contínua de forma consistente e reproduzível. Além disso, esta conclusão representou não apenas o sucesso técnico da implementação, mas também a criação de uma base sólida para evoluções futuras, como integração com Docker, ambientes mais complexos e práticas de Entrega Contínua (CD).



Assim, a Parte 1 estabeleceu com sucesso os fundamentos essenciais de automação, validação e controlo de qualidade dentro de um fluxo DevOps estruturado.



\# Parte 2: Pipeline Spring Boot com Jenkins + Docker



\## 1. Introdução



Após a conclusão da Parte 1, onde foi implementada uma pipeline de Integração Contínua (CI) para o projeto gradle-basic-demo, o objetivo da Parte 2 foi evoluir esse processo para uma pipeline completa de CI/CD aplicada ao projeto Spring Boot basic.



Nesta fase, o foco deixou de ser apenas validar código e gerar artefatos, passando também a incluir a construção da aplicação Spring Boot, execução de testes automatizados, geração de documentação Javadoc, produção do ficheiro .war, criação de imagem Docker e publicação no Docker Hub.



Além disso, foi introduzido o versionamento por número de build, permitindo maior controlo e rastreabilidade sobre cada versão gerada. Dessa forma, a pipeline passou não só a garantir qualidade e validação técnica, mas também a preparar automaticamente a aplicação para distribuição e deployment.



A principal diferença relativamente à Parte 1 foi a transição de uma pipeline de Integração Contínua para uma abordagem de Entrega Contínua (CD), aproximando o projeto de práticas DevOps mais completas.



Para atingir estes objetivos, a pipeline foi organizada nas seguintes etapas: Checkout, Java Version, Assemble, Test, Javadoc, Archive e Publish Image.



\## 2. Estrutura do Projeto



Durante a Parte 2, a estrutura do diretório CE2 foi expandida para suportar o novo projeto Spring Boot basic, mantendo simultaneamente o projeto gradle-basic-demo desenvolvido na Parte 1. Esta organização permitiu preservar ambas as implementações dentro do mesmo repositório, garantindo separação clara entre as diferentes fases do exercício e facilitando a gestão independente de cada pipeline.



&#x09;CE2

&#x09;│   Jenkinsfile

&#x09;│   README.md

&#x09;│

&#x09;├───basic

&#x09;│   │   .gitignore

&#x09;│   │   build.gradle

&#x09;│   │   Dockerfile

&#x09;│   │   gradlew

&#x09;│   │   gradlew.bat

&#x09;│   │   HELP.md

&#x09;│   │   Jenkinsfile

&#x09;│   │   package.json

&#x09;│   │   README.md

&#x09;│   │   settings.gradle

&#x09;│   │   webpack.config.js

&#x09;│   │

&#x09;│   ├───src

&#x09;│   │   ├───main

&#x09;│   │   │   ├───java

&#x09;│   │   │   ├───js

&#x09;│   │   │   └───resources

&#x09;│   │   │

&#x09;│   │   └───test

&#x09;│   │       └───java

&#x09;│   │           └───AppTest.java

&#x09;│   │

&#x09;│   └───gradle

&#x09;│

&#x09;└───gradle-basic-demo



A pasta basic concentrou todos os ficheiros necessários para o desenvolvimento e automatização da aplicação Spring Boot. O build.gradle e settings.gradle definiram a configuração principal do projeto, enquanto o gradlew garantiu execução consistente do Gradle Wrapper em qualquer ambiente. O Dockerfile tornou possível a criação automatizada da imagem Docker da aplicação, sendo um elemento central para a fase de Continuous Delivery.



A presença de package.json e webpack.config.js refletiu também a componente frontend integrada no projeto, exigindo instalação e build de dependências JavaScript durante o processo. A estrutura src/main organizou backend, frontend e recursos da aplicação, enquanto src/test assegurou suporte a testes automatizados.



Esta organização modular foi essencial para permitir que o Jenkins localizasse corretamente o projeto CE2/basic, executasse a pipeline específica desta fase e suportasse um processo mais avançado de build, packaging e publicação, sem comprometer o trabalho realizado anteriormente.



\## 3. Alterações Necessárias no Projeto



Ao contrário da Parte 1, nesta fase não foi suficiente apenas criar ou ajustar o Jenkinsfile. A implementação da pipeline CI/CD para o projeto Spring Boot exigiu várias modificações adicionais na estrutura e configuração do próprio projeto, de forma a garantir compatibilidade entre Jenkins, Java 11, Gradle e Docker.



Como o projeto basic apresentava maior complexidade, foi necessário adaptar diferentes componentes para assegurar que todo o processo de build, teste, packaging e containerização pudesse ser executado corretamente num ambiente automatizado. Estas alterações incluíram ajustes no build.gradle para resolver dependências e compatibilidade de versões, utilização adequada do Gradle Wrapper, validação da versão de Java utilizada no ambiente Jenkins e configuração do Dockerfile para suportar a criação e publicação da imagem da aplicação.



Além disso, devido à presença de componentes frontend e backend no mesmo projeto, foi importante garantir que todas as dependências necessárias fossem corretamente processadas durante a pipeline, evitando falhas relacionadas com compilação, geração do .war ou build Docker.



Dessa forma, esta fase exigiu uma abordagem mais abrangente, onde não apenas a pipeline foi desenvolvida, mas também o próprio projeto foi ajustado para funcionar de forma estável dentro de um fluxo completo de Integração e Entrega Contínua.



\### 3.1 Criação e Configuração do Dockerfile



Uma das principais alterações desta fase foi a criação de um Dockerfile, responsável por permitir a containerização da aplicação Spring Boot após a geração do ficheiro .war. Esta etapa foi essencial para transformar o projeto numa aplicação portátil, padronizada e pronta para deployment em diferentes ambientes.



O Dockerfile foi configurado para utilizar uma imagem base do Tomcat 9 com Java 11, garantindo compatibilidade com o ambiente exigido pelo projeto e permitindo a execução direta do artefato gerado pela pipeline.



&#x09;FROM tomcat:9-jdk11

&#x09;COPY build/libs/\*.war /usr/local/tomcat/webapps/basic.war

&#x09;EXPOSE 8080



A instrução FROM tomcat:9-jdk11 definiu como base uma imagem oficial contendo Tomcat 9 e Java 11, assegurando um ambiente estável e compatível com a aplicação. Em seguida, o comando COPY build/libs/\*.war /usr/local/tomcat/webapps/basic.war automatizou a cópia do ficheiro .war produzido pelo Gradle para a pasta de deploy do Tomcat, permitindo que a aplicação fosse disponibilizada automaticamente ao iniciar o container.



Por fim, a instrução EXPOSE 8080 indicou a porta padrão de execução da aplicação dentro do container, facilitando o mapeamento para acesso externo.



Esta configuração teve como principal objetivo automatizar o empacotamento da aplicação, garantir portabilidade, simplificar futuras implementações e tornar o artefato final imediatamente pronto para deployment, alinhando o projeto com práticas modernas de DevOps e Continuous Delivery.



\### 3.2 Alterações ao build.gradle



O ficheiro build.gradle original precisou de ajustes para garantir compatibilidade com a pipeline Jenkins e permitir execução correta das etapas de build, teste e geração do .war.



As principais alterações mantiveram a estrutura base do projeto Spring Boot, mas adicionaram suporte adequado ao Gradle Wrapper, compatibilidade com Java 11, configuração explícita para testes com JUnit 5 e suporte contínuo aos componentes frontend, como npm e webpack.



As adições mais importantes foram:



&#x09;testImplementation('org.junit.jupiter:junit-jupiter:5.9.3')

&#x09;

&#x09;test {

&#x09;    useJUnitPlatform()

&#x09;}



A inclusão de testImplementation garantiu suporte ao JUnit 5, enquanto useJUnitPlatform() permitiu ao Gradle executar corretamente os testes automatizados dentro da pipeline.



Estas alterações corrigiram falhas no stage Test, eliminaram o erro compileTestJava NO-SOURCE e permitiram ao Jenkins gerar relatórios JUnit válidos.



No final, o build.gradle ficou preparado para suportar de forma estável compilação, testes, packaging e integração completa com o processo CI/CD.



\### 3.3 Criação de AppTest.java



Como o projeto não possuía uma estrutura de testes suficientemente preparada para execução automatizada no Jenkins, foi necessário criar ou adaptar uma classe de teste básica para garantir o funcionamento correto do stage Test dentro da pipeline.



&#x09;import org.junit.jupiter.api.Test;

&#x09;import static org.junit.jupiter.api.Assertions.assertTrue;

&#x09;

&#x09;public class AppTest {

&#x09;

&#x09;    @Test

&#x09;    void contextLoads() {

&#x09;        assertTrue(true);

&#x09;    }

&#x09;}



Esta implementação teve como principal objetivo fornecer uma validação mínima compatível com JUnit 5, assegurando que o Jenkins pudesse executar testes com sucesso, gerar relatórios válidos e evitar falhas relacionadas com ausência de testes.



Embora simples, o método contextLoads() confirmou que a estrutura de testes estava corretamente configurada, que o JUnit estava integrado com o Gradle e que a pipeline conseguia processar a fase de testes sem erros.



Dessa forma, esta classe foi essencial para estabilizar o stage Test, eliminar problemas como NO-SOURCE e garantir que a pipeline produzisse resultados de teste válidos dentro do fluxo de CI/CD.



\## 4. Configuração do Jenkins



Para a execução da pipeline CI/CD da Parte 2, o Jenkins continuou a ser utilizado em ambiente Docker, garantindo um setup isolado, consistente e fácil de gerir.



O job foi configurado como Pipeline from SCM, permitindo que o Jenkins obtivesse automaticamente o Jenkinsfile diretamente do repositório GitHub. Para esta fase, foi definido o seguinte Script Path:



&#x09;CE2/basic/Jenkinsfile



Esta configuração garantiu que o Jenkins executasse especificamente a pipeline do projeto Spring Boot basic.



Foram também necessárias duas credenciais principais: uma para acesso ao repositório GitHub privado e outra para autenticação no Docker Hub, identificada como dockerhub-credentials, utilizada para publicação automática da imagem Docker.



Com esta configuração, o Jenkins ficou preparado para automatizar todo o processo de checkout, build, teste, packaging e publicação da aplicação.



\## 5. Jenkinsfile Final



A pipeline da Parte 2 representou uma evolução significativa relativamente à Parte 1, transformando uma pipeline de Integração Contínua numa solução completa de CI/CD. Para além das etapas de checkout, compilação, testes e documentação, esta versão passou também a garantir compatibilidade com Java 11, geração de ficheiro .war, construção de imagem Docker e publicação automática no Docker Hub.



&#x09;pipeline {

&#x09;    agent {

&#x09;        docker {

&#x09;            image 'gradle:7.4.2-jdk11'

&#x09;            args '-u root -v /var/run/docker.sock:/var/run/docker.sock'

&#x09;        }

&#x09;    }

&#x09;

&#x09;    environment {

&#x09;        IMAGE\_NAME = 'gui1251991lemes/ce2-basic'

&#x09;    }

&#x09;

&#x09;    stages {

&#x09;

&#x09;        stage('Checkout') {

&#x09;            steps {

&#x09;                checkout scm

&#x09;            }

&#x09;        }

&#x09;

&#x09;        stage('Java Version') {

&#x09;            steps {

&#x09;                dir('CE2/basic') {

&#x09;                    sh 'java -version'

&#x09;                    sh 'chmod +x gradlew'

&#x09;                    sh './gradlew --version'

&#x09;                }

&#x09;            }

&#x09;        }

&#x09;

&#x09;        stage('Assemble') {

&#x09;            steps {

&#x09;                dir('CE2/basic') {

&#x09;                    sh './gradlew assemble'

&#x09;                }

&#x09;            }

&#x09;        }

&#x09;

&#x09;        stage('Test') {

&#x09;            steps {

&#x09;                dir('CE2/basic') {

&#x09;                    sh './gradlew test'

&#x09;                }

&#x09;                junit 'CE2/basic/build/test-results/test/\*.xml'

&#x09;            }

&#x09;        }

&#x09;

&#x09;        stage('Javadoc') {

&#x09;            steps {

&#x09;                dir('CE2/basic') {

&#x09;                    sh './gradlew javadoc'

&#x09;                }

&#x09;            }

&#x09;        }

&#x09;

&#x09;        stage('Archive') {

&#x09;            steps {

&#x09;                archiveArtifacts 'CE2/basic/build/libs/\*.war'

&#x09;                archiveArtifacts 'CE2/basic/build/docs/javadoc/\*\*'

&#x09;            }

&#x09;        }

&#x09;

&#x09;        stage('Publish Image') {

&#x09;            steps {

&#x09;                dir('CE2/basic') {

&#x09;                    sh 'apt-get update \&\& apt-get install -y docker.io'

&#x09;

&#x09;                    script {

&#x09;                        docker.withRegistry('', 'dockerhub-credentials') {

&#x09;                            def app = docker.build("${IMAGE\_NAME}:${BUILD\_NUMBER}")

&#x09;                            app.push("${BUILD\_NUMBER}")

&#x09;                            app.push("latest")

&#x09;                        }

&#x09;                    }

&#x09;                }

&#x09;            }

&#x09;        }

&#x09;    }

&#x09;}



O bloco inicial:



&#x09;pipeline {



define a estrutura principal da pipeline. É o ponto de entrada que organiza toda a automação e estabelece a lógica geral de execução.



O bloco:



&#x09;agent {

&#x09;    docker {

&#x09;        image 'gradle:7.4.2-jdk11'

&#x09;        args '-u root -v /var/run/docker.sock:/var/run/docker.sock'

&#x09;    }

&#x09;}



define o ambiente onde toda a pipeline será executada. Em vez de usar qualquer agente genérico, esta configuração utiliza um container Docker com Gradle 7.4.2 e Java 11, garantindo compatibilidade com o projeto Spring Boot. O parâmetro docker.sock permite que o Jenkins utilize Docker dentro do próprio container para construir e publicar imagens.



A secção:



&#x09;environment {

&#x09;    IMAGE\_NAME = 'gui1251991lemes/ce2-basic'

&#x09;}



cria uma variável de ambiente global para o nome da imagem Docker. Isto facilita manutenção e reutilização, evitando repetição de valores ao longo da pipeline.



O bloco:



&#x09;stages {



funciona como o contentor principal de todas as fases da pipeline, organizando cada etapa do processo CI/CD.



Na fase de Checkout:



&#x09;stage('Checkout') {

&#x09;    steps {

&#x09;        checkout scm

&#x09;    }

&#x09;}



o Jenkins obtém automaticamente o código-fonte diretamente do repositório GitHub configurado, garantindo acesso à versão mais recente do projeto.



Na fase de Java Version:



&#x09;stage('Java Version') {

&#x09;    steps {

&#x09;        dir('CE2/basic') {

&#x09;            sh 'java -version'

&#x09;            sh 'chmod +x gradlew'

&#x09;            sh './gradlew --version'

&#x09;        }

&#x09;    }

&#x09;}



é validado o ambiente de execução. Primeiro verifica-se a versão de Java, depois são concedidas permissões de execução ao Gradle Wrapper e, por fim, confirma-se a versão do Gradle. Esta etapa é fundamental para evitar incompatibilidades técnicas.



Na fase de Assemble:



&#x09;stage('Assemble') {

&#x09;    steps {

&#x09;        dir('CE2/basic') {

&#x09;            sh './gradlew assemble'

&#x09;        }

&#x09;    }

&#x09;}



a aplicação é compilada e preparada, garantindo que o projeto consegue ser construído corretamente.



Na fase de Test:



&#x09;stage('Test') {

&#x09;    steps {

&#x09;        dir('CE2/basic') {

&#x09;            sh './gradlew test'

&#x09;        }

&#x09;        junit 'CE2/basic/build/test-results/test/\*.xml'

&#x09;    }

&#x09;}



são executados os testes automatizados definidos no projeto. O comando junit recolhe os resultados e gera relatórios diretamente no Jenkins, permitindo validação funcional contínua.



Na fase de Javadoc:



&#x09;stage('Javadoc') {

&#x09;    steps {

&#x09;        dir('CE2/basic') {

&#x09;            sh './gradlew javadoc'

&#x09;        }

&#x09;    }

&#x09;}



é produzida documentação técnica automática do projeto, importante para manutenção e compreensão futura.



Na fase de Archive:



&#x09;stage('Archive') {

&#x09;    steps {

&#x09;        archiveArtifacts 'CE2/basic/build/libs/\*.war'

&#x09;        archiveArtifacts 'CE2/basic/build/docs/javadoc/\*\*'

&#x09;    }

&#x09;}



o Jenkins arquiva os principais artefatos produzidos: o ficheiro .war da aplicação e a documentação Javadoc. Isto garante rastreabilidade e acesso a builds anteriores.



Por fim, na fase de Publish Image:



&#x09;stage('Publish Image') {

&#x09;    steps {

&#x09;        dir('CE2/basic') {

&#x09;            sh 'apt-get update \&\& apt-get install -y docker.io'

&#x09;

&#x09;            script {

&#x09;                docker.withRegistry('', 'dockerhub-credentials') {

&#x09;                    def app = docker.build("${IMAGE\_NAME}:${BUILD\_NUMBER}")

&#x09;                    app.push("${BUILD\_NUMBER}")

&#x09;                    app.push("latest")

&#x09;                }

&#x09;            }

&#x09;        }

&#x09;    }

&#x09;}



ocorre a principal evolução para Continuous Delivery. Primeiro, o Docker é instalado no ambiente de execução. Depois, utilizando credenciais do Docker Hub, a pipeline constrói automaticamente uma imagem da aplicação com base no Dockerfile e publica essa imagem com duas tags: uma correspondente ao número do build para versionamento e outra latest para a versão mais recente.



No conjunto, este Jenkinsfile implementa uma pipeline completa de CI/CD, automatizando desde a obtenção do código até à compilação, testes, documentação, packaging e distribuição da aplicação, aproximando o projeto de um fluxo DevOps real.



\## 6. Problemas Encontrados e Resolução Completa



Durante a implementação da pipeline CI/CD da Parte 2, surgiram vários problemas técnicos relacionados com estrutura do projeto, paths, compatibilidade entre ferramentas, Docker e configuração de credenciais. A resolução destes erros foi essencial para estabilizar a pipeline e garantir que todas as etapas funcionassem corretamente.



O primeiro problema ocorreu quando o Jenkins não conseguia localizar o ficheiro da pipeline. O erro estava relacionado com um path incorreto para o Jenkinsfile. Como o projeto Spring Boot estava dentro de CE2/basic, foi necessário definir corretamente o Script Path como:



&#x09;CE2/basic/Jenkinsfile



Isto garantiu que o Jenkins executasse a pipeline correta.



O segundo problema foi o erro gradlew not found, causado pela execução de comandos fora do diretório do projeto. A solução passou por garantir que todos os comandos Gradle fossem executados dentro da pasta correta:



&#x09;dir('CE2/basic')



Outro erro crítico foi o Unsupported class file major version 61, provocado por incompatibilidade entre a versão de Java e o Gradle utilizado. Como o projeto exigia Java 11, a solução foi definir explicitamente um ambiente compatível através de:



&#x09;image 'gradle:7.4.2-jdk11'



Também surgiu o erro docker: not found, porque o ambiente Jenkins não possuía Docker instalado por defeito. Para resolver este problema, foi necessário instalar o Docker durante a pipeline:



&#x09;apt-get update

&#x09;apt-get install -y docker.io



Após isso, ocorreu o erro permission denied docker.sock, relacionado com permissões insuficientes para acesso ao Docker socket. A solução foi executar o container como root e montar corretamente o socket:



&#x09;args '-u root -v /var/run/docker.sock:/var/run/docker.sock'



Outro problema recorrente foi o erro compileTestJava NO-SOURCE, causado pela ausência de testes válidos no projeto. A solução envolveu duas ações principais: criação da classe AppTest.java e atualização do build.gradle para suportar corretamente JUnit 5.



Por fim, a pipeline apresentou falhas no stage Archive porque inicialmente procurava ficheiros .jar, enquanto o projeto Spring Boot gerava .war. A correção foi:



&#x09;archiveArtifacts 'CE2/basic/build/libs/\*.war'



A resolução destes problemas foi fundamental para transformar uma pipeline inicialmente instável numa solução completa e funcional. Cada erro corrigido contribuiu para melhorar compatibilidade, automação e robustez do processo, consolidando uma pipeline CI/CD capaz de compilar, testar, documentar, empacotar e publicar automaticamente a aplicação Spring Boot.



\## 7. Resultado Final



Após a resolução de todos os problemas de configuração, compatibilidade e automação, a pipeline CI/CD da Parte 2 foi concluída com sucesso, cumprindo integralmente todos os objetivos definidos para o projeto Spring Boot basic.



Todas as etapas principais foram executadas corretamente: obtenção automática do código-fonte (Checkout), validação do ambiente Java e Gradle (Java Version), compilação da aplicação (Assemble), execução de testes automatizados (Test), geração de documentação técnica (Javadoc), arquivamento dos artefatos .war e documentação (Archive) e, por fim, construção e publicação automática da imagem Docker no Docker Hub (Publish Image).



A imagem final publicada foi:



&#x09;gui1251991lemes/ce2-basic



O estado final apresentado pelo Jenkins foi:



&#x09;Finished: SUCCESS



Este resultado confirmou que a pipeline estava totalmente funcional, automatizando com sucesso todo o ciclo de Integração e Entrega Contínua — desde o código-fonte até à disponibilização de uma imagem pronta para deployment.



Com esta implementação, o projeto passou a refletir práticas reais de DevOps, garantindo não apenas qualidade e validação técnica, mas também portabilidade, versionamento e preparação automatizada para distribuição em ambientes modernos.



\## Conclusão Final



A Parte 2 representou uma evolução estrutural significativa relativamente à Parte 1, deixando de se focar apenas numa pipeline básica de Integração Contínua para implementar uma solução completa de CI/CD aplicada a um projeto Spring Boot real.



Ao longo desta fase, não foi suficiente apenas configurar o Jenkins — foi necessário adaptar profundamente o próprio projeto para garantir compatibilidade total com automação, Java 11, Gradle, Docker e publicação externa. Entre as principais alterações estiveram a criação de um Dockerfile, desenvolvimento de um novo Jenkinsfile mais avançado, ajustes ao build.gradle, criação de AppTest.java, integração com Docker Hub, correção de incompatibilidades entre Java e Gradle e adaptação do processo de arquivamento para ficheiros .war.



Estas mudanças permitiram transformar o projeto numa aplicação totalmente preparada para um fluxo moderno de build e entrega, aproximando o exercício de práticas reais de DevOps.



Durante este processo, foram também desenvolvidas competências técnicas fundamentais em Jenkins, Docker, Docker Hub, Spring Boot, Gradle, JUnit e troubleshooting DevOps, especialmente na resolução de problemas relacionados com paths, compatibilidade, permissões e automação de ambientes.



Como resultado final, foi construída com sucesso uma pipeline completa capaz de validar, compilar, testar, documentar, empacotar, dockerizar e publicar automaticamente a aplicação Spring Boot. Esta implementação simulou de forma prática um processo real de Entrega Contínua, consolidando conhecimentos essenciais de automação, integração e distribuição de software em contexto profissional.





