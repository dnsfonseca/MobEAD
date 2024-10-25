
# Projeto DevOps: Pipelines com Jenkins, Docker e SonarQube

## Descrição
Este projeto configura um pipeline completo de DevOps usando **Jenkins** para gerenciar deploys de ambientes de desenvolvimento e produção, além de monitorar o código com **SonarQube** para garantir a qualidade. Todo o ambiente é configurado em uma **máquina virtual** (VM) que utiliza **Docker** para containers de desenvolvimento, produção e SonarQube, enquanto o Jenkins está instalado diretamente na VM.

## Ferramentas e Tecnologias Utilizadas
- **Jenkins**: Ferramenta de automação para integração contínua e gerenciamento de pipelines.
- **Docker**: Containerização dos ambientes de desenvolvimento, produção e SonarQube.
- **SonarQube**: Plataforma de análise contínua da qualidade de código.
- **SonarQube Scanner**: Ferramenta CLI para análise de código no pipeline de Jenkins.
- **Slack**: Utilizado para notificar sobre o progresso dos deploys e solicitar aprovações manuais.
- **Hadolint**: Linter para Dockerfiles utilizado no pipeline.

## Estrutura de Pipelines no Jenkins
O pipeline é dividido em três jobs principais no Jenkins: **Monitoramento**, **Desenvolvimento**, e **Produção**. Cada job gerencia uma parte do ciclo de desenvolvimento e deploy, com integração contínua e análise de código.

### Pipeline de Monitoramento
Nome do job: `MonitoramentoPipelineUnyleyaMobEAD-danielfns`

1. **Monitorar Repositório Git**: Verifica se há mudanças no repositório Git para iniciar os pipelines de desenvolvimento e produção.
2. **Trigger para Pipeline de Desenvolvimento**: Após detectar uma alteração, o pipeline de desenvolvimento é acionado automaticamente.

### Pipeline de Desenvolvimento
Nome do job: `DevPipelineUnyleyaMobEAD-danielfns`

1. **Lint Dockerfile**: Verifica o Dockerfile com **Hadolint** para garantir conformidade.
2. **Build da Imagem Docker**: Constrói a imagem Docker e faz o push para o repositório Docker Hub.
3. **Deploy de Desenvolvimento**: Realiza o deploy do ambiente de desenvolvimento na porta 81.
4. **Notificação via Slack**: Envia uma mensagem no canal do Slack para confirmar que o deploy de desenvolvimento foi concluído com sucesso.
5. **Solicitação de Aprovação para Produção**: Envia uma mensagem ao canal do Slack solicitando aprovação para o deploy de produção.

### Pipeline de Produção
Nome do job: `ProdPipelineUnyleyaMobEAD-danielfns`

1. **Execução Condicional após Aprovação Manual**: O deploy de produção só é executado após aprovação manual no Slack.
2. **Deploy de Produção**: Realiza o deploy do ambiente de produção na porta 82.
3. **Notificação via Slack**: Envia uma mensagem ao canal do Slack informando que o deploy de produção foi concluído com sucesso.

### Integração com Slack
As notificações e solicitações de aprovação são enviadas para o canal **pós-graduação-engenharia-devops** no Slack. Após o deploy de desenvolvimento, uma mensagem é enviada ao Slack com a URL do ambiente DEV. Para o deploy de produção, uma aprovação manual é solicitada, e o deploy só é realizado após a confirmação via Slack.

## Configuração do Ambiente
### Instalação do Docker
Para instalar o Docker no Linux (Ubuntu), siga as instruções na documentação oficial:
- [Instalar Docker no Ubuntu](https://docs.docker.com/desktop/install/linux/ubuntu/)

### Instalação do Jenkins
O Jenkins está instalado diretamente na VM. Para instalar o Jenkins no Linux (Ubuntu), siga as instruções na documentação oficial:
- [Instalar Jenkins no Ubuntu](https://www.jenkins.io/doc/book/installing/linux/)

### Instalação do SonarQube
Para configurar o **SonarQube** no Docker, utilize a imagem oficial e siga as instruções na documentação oficial:
- [Guia de instalação do SonarQube](https://docs.sonarsource.com/sonarqube/latest/try-out-sonarqube/)

### Configuração do SonarQube Scanner no Jenkins
O **SonarQube Scanner** é instalado diretamente na VM e integrado com o Jenkins para análise de código.

#### Passos de Configuração:
1. **Instalar o SonarQube Scanner**:
   ```bash
   wget https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-4.6.2.2472-linux.zip -O sonar-scanner-cli.zip
   sudo unzip sonar-scanner-cli.zip -d /opt/sonar-scanner
   ```

2. **Adicionar o Scanner ao `PATH`**:
   Adicione o diretório `/opt/sonar-scanner/bin` ao `PATH` do usuário Jenkins:
   ```bash
   export PATH=$PATH:/opt/sonar-scanner/bin
   ```

3. **Pipeline no Jenkins**:
   Configure o pipeline para executar o SonarQube Scanner:
   ```bash
   #!/bin/bash
   export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
   export PATH=$JAVA_HOME/bin:$PATH:/opt/sonar-scanner/bin

   sonar-scanner      -Dsonar.projectKey=jenkins      -Dsonar.sources=.      -Dsonar.host.url=http://192.168.1.8:9000      -X      -Dsonar.login=<SEU_TOKEN_AQUI>
   ```

### Deploy de Ambientes
- **Ambiente de Desenvolvimento**: Rodando na porta 81.
- **Ambiente de Produção**: Rodando na porta 82.
- **SonarQube**: Rodando na porta 9000 no Docker.

## Links Úteis
- [Docker Hub](https://hub.docker.com/)
- [Instalação do Docker no Linux (Ubuntu)](https://docs.docker.com/desktop/install/linux/ubuntu/)
- [Instalação do Jenkins no Linux](https://www.jenkins.io/doc/book/installing/linux/)
- [Guia de instalação do SonarQube](https://docs.sonarsource.com/sonarqube/latest/try-out-sonarqube/)

## Logs e Solução de Problemas
### Logs do Jenkins
Os logs do Jenkins podem ser acessados no servidor em:
```bash
/var/log/jenkins/jenkins.log
```

### Verificação do `PATH`
Para verificar o `PATH` do Jenkins, adicione um comando no pipeline:
```bash
echo $PATH
```

### Debugging do SonarQube Scanner
Verifique se o SonarQube Scanner está disponível no sistema:
```bash
sonar-scanner --version
```
