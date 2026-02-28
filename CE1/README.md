\# \*\*\_CE1 — Provisionamento Automatizado com Vagrant e Ansible\_\*\*



\## \*\*Objetivos\*\*



O objetivo do CE1 consistiu na construção de um ambiente distribuído totalmente automatizado utilizando Vagrant e Ansible, permitindo a criação, configuração e orquestração de múltiplas máquinas virtuais de forma reprodutível e idempotente. A proposta visou substituir abordagens manuais por um modelo declarativo de infraestrutura, no qual toda a configuração necessária ao funcionamento da aplicação é descrita em código.



A solução desenvolvida estabelece uma arquitetura composta por três máquinas virtuais independentes, cada uma com responsabilidade bem definida. A máquina ansible é responsável pela orquestração e execução dos playbooks. A máquina app executa o servidor de aplicação baseado em Tomcat 9, onde é feito o deploy da aplicação Spring Boot. A máquina h2 executa o servidor da base de dados H2 em modo TCP, garantindo separação clara entre camada de aplicação e camada de dados.



O ambiente final permite que todo o sistema seja criado e configurado a partir de um único comando, assegurando consistência entre execuções sucessivas e eliminando dependências manuais no sistema host.



\## \*\*Arquitetura do Projeto\*\*



A estrutura inicial do repositório foi organizada de forma intencional para garantir clareza, separação de responsabilidades e compatibilidade com versionamento em Git. Desde o início, a preocupação foi estruturar o projeto de forma modular, permitindo que o CE1 estivesse isolado do restante conteúdo da unidade curricular. A criação da estrutura do projeto resultou na seguinte organização:



&nbsp;	DevSecOps-25-26-1251991

&nbsp;	│   .gitignore

&nbsp;	│

&nbsp;	└───CE1

&nbsp;	    │   README.md

&nbsp;	    │   Vagrantfile

&nbsp;	    │

&nbsp;	    └───ansible

&nbsp;	            .gitkeep

&nbsp;	

O ficheiro .gitignore foi colocado na raiz do repositório para controlar artefactos temporários e ficheiros gerados automaticamente. Dentro da pasta CE1 encontram-se o README.md específico desta fase e o Vagrantfile, responsável por descrever integralmente a infraestrutura virtual. A pasta ansible foi criada antecipadamente para receber posteriormente os ficheiros de configuração do Ansible, sendo incluído um .gitkeep apenas para garantir que a diretoria fosse versionada no GitHub mesmo antes de conter conteúdo efetivo.



A definição da infraestrutura foi realizada no ficheiro Vagrantfile, onde foram declaradas três máquinas virtuais distintas, cada uma com função bem definida dentro da arquitetura distribuída.



&nbsp;	Vagrant.configure("2") do |config|

&nbsp;	    config.vm.box = "bento/ubuntu-22.04"

&nbsp;	

&nbsp;	    config.vm.define "ansible" do |ansible|

&nbsp;	        ansible.vm.hostname = "ansible"

&nbsp;	        ansible.vm.network "private\_network", ip: "192.168.56.9"

&nbsp;	        ansible.ssh.forward\_agent = true

&nbsp;	

&nbsp;	        ansible.vm.synced\_folder ".", "/vagrant", mount\_options: \["dmode=775,fmode=600"]

&nbsp;	

&nbsp;	        ansible.vm.provider "virtualbox" do |vb|

&nbsp;	            vb.memory = 2048

&nbsp;	            vb.cpus = 2

&nbsp;	        end

&nbsp;	

&nbsp;	        ansible.vm.provision "file",

&nbsp;	            source: File.expand\_path("~/.vagrant.d/insecure\_private\_key"),

&nbsp;	            destination: "/home/vagrant/.ssh/id\_rsa"

&nbsp;	

&nbsp;	        ansible.vm.provision "shell", inline: <<-SHELL

&nbsp;	            mkdir -p /home/vagrant/.ssh

&nbsp;	            chmod 700 /home/vagrant/.ssh

&nbsp;	            chmod 600 /home/vagrant/.ssh/id\_rsa

&nbsp;	            chown -R vagrant:vagrant /home/vagrant/.ssh

&nbsp;	            export DEBIAN\_FRONTEND=noninteractive

&nbsp;	            apt-get update --yes

&nbsp;	            apt-get install --yes software-properties-common

&nbsp;	            apt-add-repository --yes ppa:ansible/ansible

&nbsp;	            apt-get update --yes

&nbsp;	            apt-get install --yes ansible

&nbsp;	        SHELL

&nbsp;	    end

&nbsp;	

&nbsp;	    config.vm.define "app" do |app|

&nbsp;	        app.vm.hostname = "app"

&nbsp;	        app.vm.network "private\_network", ip: "192.168.56.10"

&nbsp;	        app.vm.network "forwarded\_port", guest: 8080, host: 8080

&nbsp;	

&nbsp;	        app.vm.provider "virtualbox" do |vb|

&nbsp;	            vb.memory = 4096

&nbsp;	            vb.cpus = 2

&nbsp;	        end

&nbsp;	

&nbsp;	        app.vm.provision "shell", inline: <<-SHELL

&nbsp;	            export DEBIAN\_FRONTEND=noninteractive

&nbsp;	            apt-get update --yes

&nbsp;	            apt-get install --yes python3

&nbsp;	        SHELL

&nbsp;	    end

&nbsp;	

&nbsp;	    config.vm.define "h2" do |h2|

&nbsp;	        h2.vm.hostname = "h2"

&nbsp;	        h2.vm.network "private\_network", ip: "192.168.56.11"

&nbsp;	        h2.vm.network "forwarded\_port", guest: 8082, host: 8082

&nbsp;	

&nbsp;	        h2.vm.provider "virtualbox" do |vb|

&nbsp;	            vb.memory = 2048

&nbsp;	            vb.cpus = 1

&nbsp;	        end

&nbsp;	

&nbsp;	        h2.vm.provision "shell", inline: <<-SHELL

&nbsp;	            export DEBIAN\_FRONTEND=noninteractive

&nbsp;	            apt-get update --yes

&nbsp;	            apt-get install --yes python3

&nbsp;	        SHELL

&nbsp;	    end

&nbsp;	end



A seguir apresenta-se a descrição individual de cada máquina virtual e os procedimentos de validação realizados após o provisionamento.



\*\*Máquina Virtual ansible\*\*



A máquina ansible foi definida como nó de controlo da infraestrutura. O seu objetivo principal é executar remotamente os playbooks responsáveis pela configuração automática das máquinas app e h2. Foi-lhe atribuído o endereço IP fixo `192.168.56.9`, garantindo previsibilidade na comunicação interna da rede privada.



Após a criação do Vagrantfile e a inicialização do ambiente com:



&nbsp;	vagrant up



foi realizado o acesso à máquina ansible através de:



&nbsp;	vagrant ssh ansible



O objetivo foi confirmar se o Ansible havia sido corretamente instalado durante o provisionamento automático. A validação foi realizada executando:



&nbsp;	ansible --version



O sistema apresentou a seguinte saída:



&nbsp;	ansible \[core 2.17.14]

&nbsp;	  config file = /etc/ansible/ansible.cfg

&nbsp;	  configured module search path = \['/home/vagrant/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']

&nbsp;	  ansible python module location = /usr/lib/python3/dist-packages/ansible

&nbsp;	  ansible collection location = /home/vagrant/.ansible/collections:/usr/share/ansible/collections

&nbsp;	  executable location = /usr/bin/ansible

&nbsp;	  python version = 3.10.12 (main, Aug 15 2025, 14:32:43) \[GCC 11.4.0] (/usr/bin/python3)

&nbsp;	  jinja version = 3.0.3

&nbsp;	  libyaml = True



A saída confirma que o Ansible Core 2.17.14 foi instalado com sucesso, incluindo todas as dependências necessárias para execução de playbooks. Esta validação foi essencial para assegurar que o nó de controlo estava pronto para orquestrar a infraestrutura.



\*\*Máquina Virtual app\*\*



A máquina app foi projetada para executar o servidor de aplicação e receber o deploy da aplicação Spring Boot. Foi configurado o endereço IP fixo `192.168.56.10`, bem como o redirecionamento da porta 8080 para o host.



Após o provisionamento, foi realizado o acesso à VM através de:



&nbsp;	vagrant ssh app



A primeira validação consistiu em confirmar a instalação correta do python3:



&nbsp;	python3 --version



Resultado obtido:



&nbsp;	Python 3.10.12



Em seguida, foi testada a conectividade de rede com a máquina h2:



&nbsp;	ping 192.168.56.11



O sistema apresentou resposta positiva:



&nbsp;	PING 192.168.56.11 (192.168.56.11) 56(84) bytes of data.

&nbsp;	64 bytes from 192.168.56.11: icmp\_seq=1 ttl=64 time=0.023 ms

&nbsp;	64 bytes from 192.168.56.11: icmp\_seq=2 ttl=64 time=0.025 ms

&nbsp;	64 bytes from 192.168.56.11: icmp\_seq=3 ttl=64 time=0.046 ms

&nbsp;	64 bytes from 192.168.56.11: icmp\_seq=4 ttl=64 time=0.034 ms

&nbsp;	

&nbsp;	--- 192.168.56.11 ping statistics ---

&nbsp;	4 packets transmitted, 4 received, 0% packet loss, time 3089ms



A ausência de perda de pacotes confirmou que a rede privada estava corretamente configurada e funcional.



\*\*Máquina Virtual h2\*\*



A máquina h2 foi criada com a finalidade exclusiva de hospedar o servidor de base de dados H2 em modo TCP. Foi-lhe atribuído o IP fixo 192.168.56.11 e configurado o redirecionamento da porta 8082.



Após o provisionamento, foi realizado o acesso à VM e validada a instalação do python3:



&nbsp;	python3 --version



Resultado obtido:



&nbsp;	Python 3.10.12



A conectividade foi igualmente testada através de:



&nbsp;	ping 192.168.56.10



Obtendo-se resposta:



&nbsp;	PING 192.168.56.10 (192.168.56.10) 56(84) bytes of data.

&nbsp;	64 bytes from 192.168.56.10: icmp\_seq=1 ttl=64 time=17.9 ms

&nbsp;	64 bytes from 192.168.56.10: icmp\_seq=2 ttl=64 time=2.69 ms

&nbsp;	64 bytes from 192.168.56.10: icmp\_seq=3 ttl=64 time=1.11 ms

&nbsp;	64 bytes from 192.168.56.10: icmp\_seq=4 ttl=64 time=3.06 ms

&nbsp;	

&nbsp;	--- 192.168.56.10 ping statistics ---

&nbsp;	4 packets transmitted, 4 received, 0% packet loss, time 3005ms

&nbsp;	

Esta validação confirmou que a rede interna estava operacional e que as máquinas conseguiam comunicar entre si sem falhas.



\## \*\*Criação e Configuração do Ficheiro ansible.cfg\*\*



Após a criação da estrutura base do projeto e validação do funcionamento das máquinas virtuais, tornou-se necessário configurar explicitamente o comportamento do Ansible. Embora o Ansible possua valores padrão, a utilização de um ficheiro ansible.cfg personalizado garante previsibilidade, controlo e independência relativamente a configurações globais do sistema.



O ficheiro ansible.cfg foi criado dentro da diretoria CE1/ansible com o seguinte conteúdo:



&nbsp;	\[defaults]

&nbsp;	inventory = /vagrant/ansible/hosts

&nbsp;	remote\_user = vagrant

&nbsp;	interpreter\_python = /usr/bin/python3

&nbsp;	

&nbsp;	\[ssh\_connection]

&nbsp;	ssh\_args = -o StrictHostKeyChecking=no



Na secção `\[defaults]`, a diretiva `inventory` define o caminho absoluto do ficheiro de inventário que será utilizado pelo Ansible. Ao especificar `/vagrant/ansible/hosts`, garante-se que o Ansible utiliza sempre o ficheiro de inventário localizado na pasta sincronizada entre o host e a máquina ansible. Esta decisão elimina ambiguidades associadas a possíveis ficheiros de inventário existentes noutras localizações do sistema.



A diretiva `remote\_user = vagrant` estabelece o utilizador padrão que será utilizado nas ligações SSH às máquinas remotas. Como todas as máquinas foram criadas pelo Vagrant e utilizam o utilizador vagrant por omissão, esta definição evita a necessidade de especificar o utilizador manualmente em cada comando ou tarefa do playbook.



A diretiva `interpreter\_python = /usr/bin/python3` assume particular importância, pois define explicitamente o interpretador Python que deverá ser utilizado nas máquinas geridas. Dado que o Ansible executa módulos remotos utilizando Python, a indicação explícita do caminho evita problemas relacionados com deteção automática incorreta do interpretador, especialmente em sistemas onde múltiplas versões possam coexistir.



Na secção `\[ssh\_connection]`, a configuração `ssh\_args = -o StrictHostKeyChecking=no` desativa a verificação interativa de chaves SSH. Em ambientes criados dinamicamente pelo Vagrant, esta configuração impede que o processo de ligação seja interrompido pela confirmação manual da autenticidade do host. Esta decisão foi tomada para assegurar execução totalmente automatizada do playbook, sem necessidade de intervenção do utilizador.



A criação deste ficheiro foi essencial para garantir consistência na execução do comando `ansible-playbook playbook.yml` sem depender de configurações externas ou variáveis de ambiente.



\## \*\*Criação e Configuração do Ficheiro hosts\*\* 



Após definir o comportamento global do Ansible através do ansible.cfg, foi necessário criar o ficheiro de inventário `hosts`, responsável por identificar as máquinas alvo e os respetivos parâmetros de ligação.



O ficheiro hosts, localizado em CE1/ansible, apresenta o seguinte conteúdo:



&nbsp;	\[app]

&nbsp;	app ansible\_host=192.168.56.10 ansible\_user=vagrant ansible\_ssh\_private\_key\_file=/vagrant/.vagrant/machines/app/virtualbox/private\_key

&nbsp;	

&nbsp;	\[h2]

&nbsp;	h2 ansible\_host=192.168.56.11 ansible\_user=vagrant ansible\_ssh\_private\_key\_file=/vagrant/.vagrant/machines/h2/virtualbox/private\_key



Neste ficheiro foram definidos dois grupos distintos: `app` e `h2`. Cada grupo representa uma máquina virtual específica da infraestrutura.



No grupo app, a máquina é identificada pelo nome lógico app, sendo associado o parâmetro `ansible\_host=192.168.56.10`, que corresponde ao endereço IP fixo definido no Vagrantfile. Esta associação permite que o Ansible saiba exatamente para onde estabelecer a ligação SSH.



A variável `ansible\_user=vagrant` reforça explicitamente o utilizador remoto, garantindo coerência com a configuração estabelecida no ansible.cfg.



A variável `ansible\_ssh\_private\_key\_file` indica o caminho absoluto da chave privada gerada automaticamente pelo Vagrant para essa máquina específica. Este detalhe é fundamental, pois cada VM possui a sua própria chave privada armazenada dentro da diretoria `.vagrant`. Ao fornecer explicitamente o caminho correto, o Ansible consegue autenticar-se automaticamente sem necessidade de configuração adicional.



O mesmo padrão foi aplicado ao grupo h2, onde `ansible\_host=192.168.56.11` corresponde ao IP da máquina de base de dados, sendo igualmente especificada a chave privada correspondente.



A definição explícita destes parâmetros elimina dependências implícitas, assegura ligações SSH consistentes e garante que a execução do playbook ocorre de forma totalmente automatizada e previsível.



A combinação entre o ansible.cfg e o ficheiro hosts estabelece assim a base da orquestração remota, permitindo que o Ansible identifique corretamente as máquinas alvo, utilize as credenciais apropriadas e execute as tarefas definidas no playbook.yml sem intervenção manual.



\## \*\*Implementação do Playbook (playbook.yml)\*\*



Após a definição do inventário e configuração do Ansible, foi desenvolvido o ficheiro playbook.yml, responsável por automatizar toda a configuração das máquinas app e h2. O playbook foi estruturado em blocos distintos, respeitando a separação de responsabilidades entre aplicação e base de dados.



O conteúdo completo do playbook é o seguinte:



&nbsp;	- hosts: app,h2

&nbsp;	  become: yes

&nbsp;	  tasks:

&nbsp;	    - name: Update apt cache

&nbsp;	      apt:

&nbsp;	        update\_cache: yes

&nbsp;	

&nbsp;	    - name: Install OpenJDK 11

&nbsp;	      apt:

&nbsp;	        name: openjdk-11-jdk

&nbsp;	        state: present

&nbsp;	

&nbsp;	    - name: Set Java 11 as default (java)

&nbsp;	      alternatives:

&nbsp;	        name: java

&nbsp;	        path: /usr/lib/jvm/java-11-openjdk-amd64/bin/java

&nbsp;	

&nbsp;	    - name: Set Java 11 as default (javac)

&nbsp;	      alternatives:

&nbsp;	        name: javac

&nbsp;	        path: /usr/lib/jvm/java-11-openjdk-amd64/bin/javac

&nbsp;	

&nbsp;	- hosts: app

&nbsp;	  become: yes

&nbsp;	  tasks:

&nbsp;	    - name: Install Tomcat

&nbsp;	      apt:

&nbsp;	        name: tomcat9

&nbsp;	        state: present

&nbsp;	

&nbsp;	- hosts: h2

&nbsp;	  become: yes

&nbsp;	  tasks:

&nbsp;	    - name: Create H2 directory

&nbsp;	      file:

&nbsp;	        path: /opt/h2

&nbsp;	        state: directory

&nbsp;	        owner: root

&nbsp;	        group: root

&nbsp;	        mode: '0755'

&nbsp;	

&nbsp;	    - name: Download H2 jar

&nbsp;	      get\_url:

&nbsp;	        url: https://repo1.maven.org/maven2/com/h2database/h2/1.4.200/h2-1.4.200.jar

&nbsp;	        dest: /opt/h2/h2.jar

&nbsp;	        mode: '0644'

&nbsp;	

&nbsp;	    - name: Create systemd service for H2

&nbsp;	      copy:

&nbsp;	        dest: /etc/systemd/system/h2.service

&nbsp;	        content: |

&nbsp;	          \[Unit]

&nbsp;	          Description=H2 Database Server

&nbsp;	          After=network.target

&nbsp;	

&nbsp;	          \[Service]

&nbsp;	          Type=simple

&nbsp;	          ExecStart=/usr/bin/java -cp /opt/h2/h2.jar org.h2.tools.Server -baseDir /opt/h2 -tcp -tcpAllowOthers -tcpPort 9092 -web -	webAllowOthers -webPort 8082

&nbsp;	          Restart=always

&nbsp;	

&nbsp;	          \[Install]

&nbsp;	          WantedBy=multi-user.target

&nbsp;	

&nbsp;	    - name: Reload systemd

&nbsp;	      systemd:

&nbsp;	        daemon\_reload: yes

&nbsp;	

&nbsp;	    - name: Restart H2

&nbsp;	      systemd:

&nbsp;	        name: h2

&nbsp;	        enabled: yes

&nbsp;	        state: restarted

&nbsp;	

&nbsp;	    - name: Wait for H2 to be available

&nbsp;	      wait\_for:

&nbsp;	        port: 9092

&nbsp;	        host: 127.0.0.1

&nbsp;	        delay: 2

&nbsp;	        timeout: 30

&nbsp;	

&nbsp;	    - name: Create H2 database via TCP

&nbsp;	      command: >

&nbsp;	        /usr/bin/java -cp /opt/h2/h2.jar org.h2.tools.Shell

&nbsp;	        -url jdbc:h2:tcp://localhost:9092/./jpadb

&nbsp;	        -user sa

&nbsp;	        -password ""

&nbsp;	        -sql "CREATE TABLE IF NOT EXISTS TEST(ID INT)"

&nbsp;	      ignore\_errors: yes

&nbsp;	

&nbsp;	- hosts: app

&nbsp;	  become: yes

&nbsp;	  tasks:

&nbsp;	    - name: Install git

&nbsp;	      apt:

&nbsp;	        name: git

&nbsp;	        state: present

&nbsp;	

&nbsp;	    - name: Install unzip

&nbsp;	      apt:

&nbsp;	        name: unzip

&nbsp;	        state: present

&nbsp;	

&nbsp;	    - name: Install wget

&nbsp;	      apt:

&nbsp;	        name: wget

&nbsp;	        state: present

&nbsp;	

&nbsp;	    - name: Clone project repository (as vagrant)

&nbsp;	      become\_user: vagrant

&nbsp;	      git:

&nbsp;	        repo: https://bitbucket.org/pssmatos/tut-basic-gradle.git

&nbsp;	        dest: /home/vagrant/tut-basic-gradle

&nbsp;	        update: yes

&nbsp;	        force: yes

&nbsp;	

&nbsp;	    - name: Fix datasource URL

&nbsp;	      replace:

&nbsp;	        path: /home/vagrant/tut-basic-gradle/src/main/resources/application.properties

&nbsp;	        regexp: 'spring.datasource.url=.\*'

&nbsp;	        replace: 'spring.datasource.url=jdbc:h2:tcp://192.168.56.11:9092//opt/h2/jpadb'

&nbsp;	

&nbsp;	    - name: Build the application with wrapper

&nbsp;	      become\_user: vagrant

&nbsp;	      shell: ./gradlew clean build --no-daemon --console=plain

&nbsp;	      args:

&nbsp;	        chdir: /home/vagrant/tut-basic-gradle

&nbsp;	

&nbsp;	    - name: Copy WAR to Tomcat webapps

&nbsp;	      copy:

&nbsp;	        src: /home/vagrant/tut-basic-gradle/build/libs/basic-0.0.1-SNAPSHOT.war

&nbsp;	        dest: /var/lib/tomcat9/webapps/basic-0.0.1-SNAPSHOT.war

&nbsp;	        remote\_src: yes

&nbsp;	        mode: '0644'

&nbsp;	

&nbsp;	    - name: Restart Tomcat

&nbsp;	      systemd:

&nbsp;	        name: tomcat9

&nbsp;	        state: restarted



\*\*Instalação do Java 11\*\*

&nbsp;	

O primeiro bloco é executado nas máquinas app e h2. Inicialmente é atualizado o cache de pacotes do sistema, garantindo que as versões mais recentes disponíveis nos repositórios sejam utilizadas.



Em seguida, é instalado o `openjdk-11-jdk`. A escolha da versão 11 não foi arbitrária. Durante o desenvolvimento verificou-se incompatibilidade entre versões mais recentes do Java e a versão do Gradle utilizada pelo projeto. O Java 11 demonstrou compatibilidade estável com o Gradle Wrapper presente no repositório.



Após a execução, o resultado foi:



&nbsp;	TASK \[Install OpenJDK 11]

&nbsp;	ok: \[app]

&nbsp;	ok: \[h2]



As tarefas seguintes configuram o Java 11 como versão padrão do sistema através do módulo alternatives, assegurando que tanto java como javac apontam para o JDK correto.



\*\*Instalação do Tomcat\*\*



O segundo bloco aplica-se apenas à máquina app, responsável por executar a aplicação.



&nbsp;	- name: Install Tomcat

&nbsp;	  apt:

&nbsp;	    name: tomcat9

&nbsp;	    state: present



Após a instalação, foi realizado o teste manual no navegador:



&nbsp;	http://localhost:8080



O servidor retornou a página padrão do Tomcat com a mensagem:



&nbsp;	It works !

&nbsp;	If you're seeing this page via a web browser, it means you've setup Tomcat successfully.



Este teste confirmou que o servidor de aplicações estava corretamente instalado e ativo.



\*\*Instalação e Configuração do H2\*\*



O terceiro bloco configura exclusivamente a máquina h2.



Primeiramente é criado o diretório `/opt/h2`, que servirá como base de armazenamento da base de dados.



O ficheiro `h2-1.4.200.jar` é então descarregado diretamente do repositório Maven Central. A escolha da versão 1.4.200 foi intencional, pois apresentou melhor compatibilidade com o projeto Spring utilizado.



Em seguida, é criado um serviço `systemd` para o H2. O serviço executa o servidor H2 utilizando:



&nbsp;	org.h2.tools.Server



com os seguintes parâmetros principais:



&nbsp;	 -baseDir /opt/h2

&nbsp;	 -tcpAllowOthers

&nbsp;	 -tcpPort 9092

&nbsp;	 -webPort 8082



Após ativação do serviço, foi testado no navegador:



&nbsp;	http://localhost:8082



O console do H2 foi apresentado corretamente, confirmando o funcionamento do servidor de base de dados.



A tarefa `wait\_for` garante que o porto 9092 esteja disponível antes de continuar a execução, evitando falhas por tentativa de ligação prematura.



Finalmente, é executado um comando para forçar a criação da base de dados via TCP, assegurando que o ficheiro jpadb exista antes do arranque da aplicação.



\*\*Build e Deploy da Aplicação\*\*



O último bloco executa novamente na máquina app. São instaladas ferramentas auxiliares como `git, unzip e wget`.



O repositório é clonado para:



&nbsp;	/home/vagrant/tut-basic-gradle



A opção `force: yes` foi adicionada para evitar falhas caso existam modificações locais, garantindo idempotência.



Em seguida, o ficheiro `application.properties` é modificado dinamicamente através do módulo replace. Esta etapa foi essencial para alterar a URL da base de dados para:



&nbsp;	jdbc:h2:tcp://192.168.56.11:9092//opt/h2/jpadb



Sem esta correção, a aplicação não conseguiria ligar-se à base de dados remota.



O build é então realizado através do Gradle Wrapper:



&nbsp;	./gradlew clean build --no-daemon --console=plain



O ficheiro .war gerado é copiado para:



&nbsp;	/var/lib/tomcat9/webapps/basic-0.0.1-SNAPSHOT.war



Finalmente, o Tomcat é reiniciado para efetivar o deploy.



\*\*Validação Final



Após a execução completa do playbook, o seguinte endpoint foi testado:



&nbsp;	http://localhost:8080/basic-0.0.1-SNAPSHOT/api



A resposta recebida foi:



&nbsp;	{

&nbsp;	  "\_links" : {

&nbsp;	    "employees" : {

&nbsp;	      "href" : "http://localhost:8080/basic-0.0.1-SNAPSHOT/api/employees"

&nbsp;	    },

&nbsp;	    "profile" : {

&nbsp;	      "href" : "http://localhost:8080/basic-0.0.1-SNAPSHOT/api/profile"

&nbsp;	    }

&nbsp;	  }

&nbsp;	}



Esta resposta confirma que o Tomcat está funcional, que o deploy do ficheiro WAR ocorreu corretamente, que a aplicação Spring Boot iniciou com sucesso, que a ligação à base de dados H2 está operacional e que o endpoint REST se encontra disponível.



Com esta validação, conclui-se que o ambiente foi totalmente provisionado e configurado de forma automatizada, reproduzível e funcional.



\## \*\*Conclusão\*\*



A implementação do CE1 permitiu aplicar, de forma prática, os princípios de automação de infraestrutura através da utilização conjunta de Vagrant e Ansible. Foi construída uma arquitetura distribuída composta por três máquinas virtuais com funções bem definidas: uma máquina de controlo (ansible), uma máquina de aplicação (app) e uma máquina de base de dados (h2).



O Vagrantfile assegura a criação determinística das máquinas, incluindo configuração de rede privada e definição de recursos. A partir desse ponto, toda a configuração passa a ser realizada pelo Ansible, eliminando intervenções manuais e garantindo reprodutibilidade. O ficheiro ansible.cfg e o inventário hosts estabelecem uma comunicação remota consistente e controlada, baseada em autenticação por chave SSH.



O playbook.yml centraliza a instalação e configuração dos serviços necessários, incluindo Java 11, Tomcat e H2, bem como o processo de build e deploy da aplicação. A alteração automatizada do application.properties garantiu a correta integração entre aplicação e base de dados remota.



O resultado final é um ambiente configurável exclusivamente por código, executável de forma repetível e organizada, demonstrando a aplicação prática de conceitos de Infraestrutura como Código e automação em contexto DevSecOps.

