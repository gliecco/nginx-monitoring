# Monitoramento Automatizado do nginx no Ubuntu 20.04 LTS

## Sobre o projeto

A prática consiste na instalação do WSL (Subsistema do Windows para Linux) no Windows e na criação de uma instância Ubuntu 20.04 LTS. Inclui também a configuração de um servidor nginx, o monitoramento do status do serviço por meio de um script personalizado e a automatização da execução deste script a cada 5 minutos. O script deve conter a data, hora, o nome do serviço, o status e uma mensagem personalizada de ONLINE ou OFFLINE. O objetivo é garantir a continuidade e a disponibilidade do serviço, registrando seu status em arquivos separados.

### Índice

1. [Pré-requisitos](#1-pré-requisitos)
2. [Ativação e configuração do WSL](#2-ativação-e-configuração-do-wsl)
3. [Instalação e Configuração do Ubuntu 20.04 LTS](#3-instalação-e-configuração-do-ubuntu-2004-lts)
4. [Instalação e Configuração do nginx](#4-instalação-e-configuração-do-nginx)
5. [Criação do script de Monitoramento](#5-criação-do-script-de-monitoramento-do-status-do-nginx)
   - 5.1 [Configuração do Diretório](#51-configuração-do-diretório)
   - 5.2 [Criação do Script](#52-criação-do-script)

## 1. Pré-requisitos

- Windows 10 versão 2004 e superior ou Windows 11
- Conhecimento básico do terminal Linux

## 2. Ativação e configuração do WSL

Clique com o botão direito do mouse sobre o PowerShell ou o Prompt de Comando do Windows e selecione **"Executar como administrador"**.

Depois, execute o comando:

```powershell
wsl --install
```

Após executado o comando e terminado a instalação do WSL, reinicie o computador.

## 3. Instalação e Configuração do Ubuntu 20.04 LTS

Abra o PowerShell como administrador e execute o comando:

```powershell
wsl --install -d Ubuntu-20.04
```

<details>
<summary>**Instalação pela Microsoft Store**</summary>
Alternativamente, você pode abrir a Microsoft Store, buscar por "Ubuntu 20.04 LTS", clicar em adquirir e instalar a distribuição.
</details>
Terminado o processo de instalação do Ubuntu no WSL, você será solicitado a criar um nome de usuário e senha. Esta conta será o **usuário padrão e administrador da distribuição**, com permissões para executar comandos de super usuário (`sudo`).

## 4. Instalação e Configuração do nginx

Abra o terminal do Ubuntu e execute o seguinte comando para garantir a instalação do pacote correto e sua versão mais recente:

```bash
sudo apt update
```

Após isso, instale o nginx:

```bash
sudo apt install nginx
```

Inicie e verifique o status do nginx:

```bash
sudo systemctl start nginx
```

```bash
sudo systemctl status nginx
```

Caso o nginx esteja rodando corretamente, o comando retornará uma saída como esta:

![Status do nginx Ativo](imgs/nginx_status_ativo.jpeg)

Para verificar se o servidor está funcionando, abra o navegador e digite "localhost" ("localhost" é como um atalho que aponta para o seu próprio computador, permitindo que você verifique se o servidor está respondendo normalmente) na barra de endereços. Se tudo estiver certo, o servidor vai mostrar a página padrão do nginx:

![Página Padrão do nginx](imgs/nginx_via_localhost.jpeg)

## 5. Criação do script de monitoramento do status do nginx

### 5.1 Configuração do Diretório de Logs

Antes de criar o script, criaremos o diretório onde serão armazenados os logs de monitoramento do nginx:

```bash
sudo mkdir /var/log/nginx_status
```

Por padrão, o diretório será de propriedade do root, e seu usuário não terá permissão para gravar nele. Para corrigir isso, usamos:

```bash
sudo chown seu_usuario:seu_usuario /var/log/nginx_status
```

Isso altera a propriedade para o seu usuário e permite que você crie, modifique e exclua arquivos no diretório.

Em seguida, ajustamos as permissões:

```bash
sudo chmod 755 /var/log/nginx_status
```

Essa configuração garante que você tenha acesso total ao diretório, enquanto outros usuários podem apenas ler e executar os arquivos dentro dele.

### 5.2 Criação do Script

Armazenanaremos o script dentro do diretório `/usr/local/bin`, que é o diretório destinado a programas e scripts locais. Esse diretório já está incluído no PATH por padrão, permitindo a execução do script de qualquer lugar, sem precisar especificar o caminho completo.

Para criar o script, utilize o seguinte comando:

```bash
sudo nano /usr/local/bin/nginx_status_monitor.sh
```

Digite o script:

```bash
#!/bin/bash

# obtém a data e hora atuais
data_hora=$(date "+%d-%m-%Y %H:%M:%S")

# nome do serviço
servico="nginx"

# status do serviço
status=$(systemctl is-active $servico)

# caminhos para os arquivos de log
log_online="/var/log/nginx_status/nginx_online.log"
log_offline="/var/log/nginx_status/nginx_offline.log"

# verifica o status do serviço nginx e escreve nos arquivos de log
if [ "$status" = "active" ]; then
    echo "Data e Hora: $data_hora | Serviço: $servico | Status: $status | O serviço $servico está ONLINE." >> "$log_online"
else
    echo "Data e Hora: $data_hora | Serviço: $servico | Status: $status | O serviço $servico está OFFLINE" >> "$log_offline"
fi
```

Pressione `CTRL + O` e `ENTER` para salvar e `CTRL + X` para sair. Após isso, garanta permissão de execução ao script para que você possa rodá-lo:

```bash
sudo chmod +x /usr/local/bin/nginx_status_monitor.sh
```
