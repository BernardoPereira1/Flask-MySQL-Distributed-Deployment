# Configuração das Vms e Execução da Aplicação

Se **ainda não tem os ficheiros das máquinas virtuais**, segue os passos abaixo para instalar e configurá-las corretamente.  
➡️ [Já tem os ficheiros das VMs? Passe direto para iniciar a aplicação.](#execução-da-aplicação)

## Instalação inicial e configuração das máquinas virtuais com o WSL 

1. **Instale as máquinas:**
   
   ```bash
   wsl --install
   wsl --install -d "Ubuntu 24.04"
   ```

Uma vez que as máquinas foram instaladas deve definir o nome de utilizador e password.

---

## Instalação e configuração do MySQL Server e da base de dados para Acesso Externo

Na primeira máquina siga os seguintes passos:

1. **Instale o mysql-server:**
   ```bash
   sudo apt install mysql-server
   ```
   
2. **Corra o serviço mysql:**
   ```bash
   sudo service mysql start
   ```
   
3. **Abra o arquivo de configuração do MySQL:**
   ```bash
   sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
   ```
   
4. **Encontre a linha contendo `bind-address` e altere para:**
   ```bash
   bind-address = 0.0.0.0
   ```

5. **Aceda ao mysql:**
   ```bash
   sudo mysql
   ```
   
6. **Crie um usuário remoto:**
   ```sql
   CREATE USER 'admin'@'%' IDENTIFIED BY 'admin';
   ```
   
7. **Conceda permissões ao usuário:**
   ```sql
   GRANT ALL PRIVILEGES ON *.* TO 'admin'@'%' WITH GRANT OPTION;
   ```
   
8. **Aplique as alterações:**
   ```sql
   FLUSH PRIVILEGES;
   ```
   
9. **Saia do MySQL:**
   ```sql
   EXIT;
   ```
10. **Reenicie o serviço mysql:**
   ```bash
   sudo service mysql restart
   ```

11. **Volte a aceder ao mysql:**
   ```bash
   sudo mysql
   ```

12. **Corra o script para criar a base de dados:**
   ```sql
      CREATE DATABASE IF NOT EXISTS todo;
      USE todo;
      
      -- Criar a tabela de usuários
      CREATE TABLE IF NOT EXISTS user (
          id INT AUTO_INCREMENT PRIMARY KEY,
          email VARCHAR(100) NOT NULL UNIQUE,
          password VARCHAR(255) NOT NULL
      );
      
      -- Criar a tabela de tarefas
      CREATE TABLE IF NOT EXISTS task (
          id INT AUTO_INCREMENT PRIMARY KEY,
          descricao VARCHAR(255) NOT NULL,
          data_criacao TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
          user_id INT,
          FOREIGN KEY (user_id) REFERENCES user(id) ON DELETE CASCADE
      );
   ```

13. **Saia do MySQL:**
   ```sql
   EXIT;
   ```

---

## Acesso e teste à base de dados pela segunda máquina

Já na segunda máquina siga os seguintes passos:

1. **Instale o mysql client:**
   ```bash
   sudo apt install mysql-client
   ```

2. **Descubra o ip da primeira máquina:**
   **Este comando deve ser executado na primera maquina!**
   ```bash
   hostname -I
   ```

3. **Corra o comando de acesso ao mysql server:**
   ```bash
   mysql -h ip_PrimeiraMaquina -u admin -p
   ```
   **O sistema vai pedir para inserir a password que será a que foi definida na primeira máquina no mysql ao criar o utilizador!**

Se o mysql for aberto é porque a conexão foi concluida com sucesso e temos acesso à base de dados. Podemos também voltar a entrar no mysql (utilizando o comando de cima) e fazer **SHOW DATABASES;** e verificar que a base de dados **todo** está na lista.

---

## Configuração da aplicação

   **Esta configuração deve ser feita na segunda máquina uma vez que só utilizamos a primeira para a base de dados**

   1. **Clone o repositório:**
   ```bash
   git clone https://github.com/BernardoPereira1/Flask-MySQL-Distributed-Deployment.git
   cd CompNuvem
   ```

   2. **Após entrar na pasta aceda ao projeto pelo vscode:**
   ```bash
   code .
   ```

   3. **Instale as dependências:**
   ```bash
   pip install -r requirements.txt
   ```

   4. **Crie um arquivo `.env` na raiz do projeto com as configurações da base de dados:**
   ```ini
   SECRET_KEY=sua-chave-secreta-aqui
   DB_USER=admin
   DB_PASSWORD=admin
   DB_HOST=ip_PrimeiraMaquina
   DB_NAME=todo
   ```

   5. **Execute a aplicação:**
   ```bash
   python3 app.py
   ```

---

# Execução da Aplicação
## Caso tenha optado por utilizar as vms disponibilizadas, siga os seguintes passos:

Para importar uma distribuição do WSL a partir de um arquivo `.tar`, execute o seguinte comando no PowerShell (como administrador):

```bash
wsl --import <nome-da-distribuicao> <diretório-de-destino> <caminho-do-arquivo-tar>
```

Após importar as máquinas, abra a máquina que contem a base de dados **(Ubuntu)** e execute os seguintes comandos:

- **A password da máquina é "berna"**

1. **Corra o serviço mysql:**
   ```bash
   sudo service mysql start
   ```

2. **Verifique o estado do serviço mysql:**
   ```bash
   sudo service mysql status
   ```

---

De seguida abra a máquina que contém a aplicação **(Ubuntu 24.04)** e execute os seguintes comandos:

- **A password da máquina é "berna"**

1. **Entre na pasta do projeto:**
   ```bash
   cd CompNuvem
   ```

2. **Execute o vscode:**
   ```bash
   code .
   ```

3. **Corra a aplicação:**
   ```bash
   python3 app.py
   ```
   
Em caso de falha no ultimo passo pode optar por abrir o ficheiro **app.py** e clicar no run button do vs code.

![VsCode](https://i.sstatic.net/qqsMY.png)



# Acesso Remoto à Aplicação no WSL 2 a partir de outra máquina na mesma rede

Este passo explica como configurar o acesso remoto a uma aplicação que está a rodar no WSL 2 a partir de outra máquina.

## 1. Configurar Redirecionamento de Porta

O WSL 2 não permite acesso direto de outras máquinas, então é necessário configurar um proxy de porta no Windows. Execute o seguinte comando no PowerShell como administrador:

```powershell
netsh interface portproxy add v4tov4 listenport=5000 listenaddress=0.0.0.0 connectport=5000 connectaddress=WSL_IP
```

Substitua `WSL_IP` pelo IP da sua máquina dentro do WSL. Você pode obter esse IP ao utilizar o seguinte comando:

```bash
ip a
```

## 2. Abrir a Porta 5000 na Firewall do Windows

Para permitir conexões externas, abra a porta 5000 na firewall do Windows:

1. Abra o **Prompt de Comando** como administrador e execute:
   ```powershell
   netsh advfirewall firewall add rule name="WSL Flask" dir=in action=allow protocol=TCP localport=5000
   ```

2. Ou, manualmente:
   - Vá até **Painel de Controle** → **Firewall do Windows Defender** → **Configurações Avançadas**.
   - Clique em **Regras de Entrada** → **Nova Regra**.
   - Escolha **Porta**, selecione **TCP** e insira **5000**.
   - Marque **Permitir a Conexão** e conclua a configuração.

## 3. Executar a Aplicação Flask no WSL 2

## 4. Aceder a Aplicação a Partir de Outra Máquina

Agora, em qualquer dispositivo na mesma rede, aceda à aplicação através do browser ou de uma API utilizando:

```
http://<IP_DO_HOST_WINDOWS>:5000
```

Para descobrir o IP da máquina Windows, utilize o seguinte comando no cmd:

```powershell
ipconfig
```

O IP será exibido na interface de rede ativa, como "Adaptador Ethernet" ou "Wi-Fi".
---
Seguindo estes passos, a aplicação Flask que está no WSL 2 poderá ser acedida remotamente a partir de outra máquina.















