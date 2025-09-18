# Guia de Uso do OWASP ZAP no Linux

![OWASP ZAP Logo](https://www.zaproxy.org/img/zap32x32.png)

## Índice
1. [Introdução](#introdução)
2. [Pré-requisitos](#pré-requisitos)
3. [Métodos de Instalação](#métodos-de-instalação)
   - [Instalação via Pacote Debian/Ubuntu](#instalação-via-pacote-debianubuntu)
   - [Instalação via Snap](#instalação-via-snap)
   - [Instalação via Docker](#instalação-via-docker)
   - [Instalação Manual via Java JAR](#instalação-manual-via-java-jar)
4. [Primeiros Passos](#primeiros-passos)
   - [Interface Gráfica](#interface-gráfica)
   - [Linha de Comando](#linha-de-comando)
   - [Modo Daemon/API](#modo-daemonapi)
5. [Tipos de Varredura](#tipos-de-varredura)
   - [Varredura Passiva](#varredura-passiva)
   - [Varredura Ativa](#varredura-ativa)
   - [Varredura Autenticada](#varredura-autenticada)
6. [Guia Passo a Passo para Testes](#guia-passo-a-passo-para-testes)
   - [Teste Básico com Navegação Automática](#teste-básico-com-navegação-automática)
   - [Teste com Sitemap/Spider](#teste-com-sitemapspider)
   - [Teste com API](#teste-com-api)
7. [Configurações Avançadas](#configurações-avançadas)
   - [Configuração de Contextos](#configuração-de-contextos)
   - [Políticas de Varredura Personalizadas](#políticas-de-varredura-personalizadas)
   - [Scripts Personalizados](#scripts-personalizados)
8. [Integração com CI/CD](#integração-com-cicd)
9. [Dicas e Melhores Práticas](#dicas-e-melhores-práticas)
10. [Solução de Problemas](#solução-de-problemas)
11. [Recursos Adicionais](#recursos-adicionais)

## Introdução

O OWASP ZAP (Zed Attack Proxy) é uma ferramenta de segurança de aplicações web de código aberto, mantida pela OWASP (Open Web Application Security Project). É uma das ferramentas de segurança mais populares do mundo, adequada tanto para testes manuais quanto automatizados.

Este guia abordará a instalação, configuração e uso do ZAP em sistemas Linux, com foco em técnicas avançadas e práticas recomendadas.

## Pré-requisitos

- Sistema Linux (Ubuntu, Debian, CentOS, etc.)
- Java Runtime Environment (JRE) 11 ou superior
- Navegador web moderno (Firefox, Chrome, etc.)
- Conexão com a internet (para download e atualizações)

Verifique se o Java está instalado:
```bash
java -version
```

Se não estiver instalado, no Ubuntu/Debian:
```bash
sudo apt update
sudo apt install openjdk-11-jre
```

## Métodos de Instalação

### Instalação via Pacote Debian/Ubuntu

1. Adicione o repositório do ZAP:
```bash
sudo apt install -y software-properties-common
sudo add-apt-repository ppa:zaproxy/stable
sudo apt update
```

2. Instale o ZAP:
```bash
sudo apt install -y zaproxy
```

### Instalação via Snap

```bash
sudo snap install zaproxy --classic
```

### Instalação via Docker

1. Instale o Docker (se não estiver instalado):
```bash
sudo apt install docker.io
```

2. Execute o ZAP em container:
```bash
docker run -u zap -p 8080:8080 -i owasp/zap2docker-stable zap.sh -daemon -host 0.0.0.0 -port 8080 -config api.addrs.addr.name=.* -config api.addrs.addr.regex=true -config api.key=changeme
```

### Instalação Manual via Java JAR

1. Baixe a versão mais recente:
```bash
wget https://github.com/zaproxy/zaproxy/releases/download/v2.14.0/ZAP_2.14.0_Linux.tar.gz
```

2. Extraia o arquivo:
```bash
tar -xzf ZAP_2.14.0_Linux.tar.gz
```

3. Mova para o diretório de aplicações:
```bash
sudo mv ZAP_2.14.0 /opt/zaproxy
```

4. Crie um link simbólico para facilitar o acesso:
```bash
sudo ln -s /opt/zaproxy/zap.sh /usr/local/bin/zap
```

## Primeiros Passos

### Interface Gráfica

Para iniciar a interface gráfica:

```bash
zap.sh &
# ou
zaproxy
```

Na primeira execução, o ZAP perguntará se deseja persistir a sessão. Para testes simples, selecione "No, I do not want to persist this session".

### Linha de Comando

Para executar uma varredura rápida via linha de comando:

```bash
zap.sh -cmd -quickurl https://exemplo.com -quickout /tmp/report.html
```

### Modo Daemon/API

Inicie o ZAP em modo daemon com API habilitada:

```bash
zap.sh -daemon -host 0.0.0.0 -port 8080 -config api.disablekey=true
```

A API estará acessível em http://localhost:8080

## Tipos de Varredura

### Varredura Passiva

A varredura passiva ocorre enquanto você navega pela aplicação através do ZAP. Para configurar:

1. Inicie o ZAP
2. Configure seu navegador para usar o ZAP como proxy (127.0.0.1:8080)
3. Navegue pelo site normalmente

### Varredura Ativa

A varredura ativa envolve enviar requisições maliciosas para identificar vulnerabilidades.

Para executar uma varredura ativa:

1. Clique com o botão direito no site na aba "Sites"
2. Selecione "Attack" > "Active Scan"

### Varredura Autenticada

Para testar áreas restritas:

1. Vá para "File" > "New Session"
2. Navegue até a página de login através do ZAP
3. Realize o login manualmente
4. Clique com o botão direito na sessão na aba "Sites"
5. Selecione "Include in Context" > "New Context"
6. Configure a autenticação no contexto criado

## Guia Passo a Passo para Testes

### Teste Básico com Navegação Automática

1. Inicie o ZAP
2. Digite a URL alvo na "Quick Start" tab e clique em "Attack"
3. O ZAP executará automaticamente o spider e uma varredura ativa

### Teste com Sitemap/Spider

1. Adicione a URL alvo na aba "Sites" (botão direito > "Add Node")
2. Clique com o botão direito no site > "Attack" > "Spider"
3. Aguarde o spider mapear a aplicação
4. Execute uma varredura ativa nas URLs descobertas

### Teste com API

1. Inicie o ZAP em modo daemon:
```bash
zap.sh -daemon -host 0.0.0.0 -port 8080 -config api.disablekey=true
```

2. Execute um spider via API:
```bash
curl "http://localhost:8080/JSON/spider/action/scan/?url=https://exemplo.com&maxChildren=10"
```

3. Execute uma varredura ativa:
```bash
curl "http://localhost:8080/JSON/ascan/action/scan/?url=https://exemplo.com"
```

4. Gere um relatório:
```bash
curl "http://localhost:8080/OTHER/core/other/htmlreport/?apikey=" -o report.html
```

## Configurações Avançadas

### Configuração de Contextos

Contextos permitem definir escopos e configurações específicas para cada aplicação testada.

1. Clique com o botão direito no site > "Include in Context" > "New Context"
2. Defina as URLs incluídas e excluídas
3. Configure usuários e métodos de autenticação
4. Defina tecnologias utilizadas pela aplicação

### Políticas de Varredura Personalizadas

1. Vá para "Analyze" > "Scan Policy Manager"
2. Crie uma nova política ou edite uma existente
3. Ajuste as regras e thresholds conforme necessário
4. Aplique a política nas varreduras

### Scripts Personalizados

O ZAP suporta scripts em ZAP (JavaScript), Python e outros.

1. Vá para "Options" > "Scripts"
2. Clique no ícone "+" para adicionar um novo script
3. Selecione o tipo (Standalone, Passive Rule, etc.)
4. Escreva ou cole o código do script

Exemplo de script simples para adicionar um header personalizado:
```javascript
// Example script to add a custom header to all requests
function sendingRequest(msg, initiator, helper) {
    msg.getRequestHeader().setHeader('X-Custom-Header', 'value');
}

function responseReceived(msg, initiator, helper) {
    // Do nothing
}
```

## Integração com CI/CD

### Exemplo com GitHub Actions

```yaml
name: Security Scan with ZAP

on: [push, pull_request]

jobs:
  zap_scan:
    runs-on: ubuntu-latest
    steps:
    - name: ZAP Scan
      uses: zaproxy/action-baseline@v0.10.0
      with:
        target: 'https://exemplo.com'
        rules_file_name: '.zap/rules.tsv'
        cmd_options: '-a'
```

### Exemplo com Jenkins

```groovy
pipeline {
    agent any
    stages {
        stage('ZAP Scan') {
            steps {
                sh 'docker run -v $(pwd):/zap/wrk:rw -t owasp/zap2docker-stable zap-baseline.py -t https://exemplo.com -g gen.conf -r zap_report.html'
            }
        }
        stage('Archive Report') {
            steps {
                archiveArtifacts artifacts: 'zap_report.html', fingerprint: true
            }
        }
    }
}
```

## Dicas e Melhores Práticas

1. **Sempre teste em ambientes de desenvolvimento/staging** - nunca em produção
2. **Obtenha autorização por escrito** antes de testar qualquer aplicação
3. **Configure corretamente o contexto** para evitar falsos positivos
4. **Use políticas de varredura personalizadas** adaptadas à sua aplicação
5. **Mantenha o ZAP atualizado** para detecção de vulnerabilidades recentes
6. **Revise os resultados manualmente** para evitar falsos positivos/negativos
7. **Documente os processos** para reproduzibilidade dos testes

## Solução de Problemas

### Problema: ZAP não inicia
**Solução:** Verifique se o Java está instalado e se a versão é compatível.

### Problema: Conexão SSL falha
**Solução:** Importe o certificado raiz do ZAP em seu navegador/navegador automatizado.

### Problema: Spider não encontra URLs
**Solução:** Verifique se a aplicação não depende heavily de JavaScript. Considere usar o AJAX Spider.

### Problema: Varredura muito lenta
**Solução:** Ajuste a política de varredura para limitar threads e escopo.

### Problema: Falsos positivos
**Solução:** Ajuste thresholds das regras e revise manualmente os resultados.

## Recursos Adicionais

- [Site Oficial do OWASP ZAP](https://www.zaproxy.org/)
- [Documentação Oficial](https://www.zaproxy.org/docs/)
- [ZAP User Group](https://groups.google.com/group/zaproxy-users)
- [ZAP GitHub Repository](https://github.com/zaproxy/zaproxy)
- [OWASP Testing Guide](https://owasp.org/www-project-web-security-testing-guide/)

---

**Aviso Legal:** Este guia é apenas para fins educacionais. Sempre obtenha permissão explícita antes de testar qualquer aplicação web. O uso não autorizado de ferramentas de segurança pode violar leis locais e políticas organizacionais.
