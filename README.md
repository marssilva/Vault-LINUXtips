### Start vault is mod dev

`vault server -dev -dev-listen-address=0.0.0.0:8200 -dev-root-token-id=suporte`

### Autocomplete vault

`vault -autocomplete-install`

### Para listar o secret é preciso colocar kv e depois list, key/value

`vault kv list secret/`

### Criando um secret chave valor

`vault kv put secret/TesteSuporte value=1234@mudar`

### Get Data value secret

`vault kv get secret/TesteSuporte`

### Mostrar a ultima version

`vault kv get --field=teste2 secret/TesteSuporte`

### Mostrar o valor a partir da versão 

`vault kv get --version=2 --field=teste2 secret/TesteSuporte`

### Deletando uma versão do secret

`vault kv delete -versions 1 secret/TesteSuporte`

### Recuperando uma chave deletada

`vault kv undelete -versions 1 secret/TesteSuporte`

### Destruindo a chave, Adeus

`vault kv destroy -versions 1 secret/TesteSuporte`

### Listando as Secrets

`vault secrets list`

### Criando uma nova secrets no path diferente do secret

`vault secrets enable -path=ASCES-UNITA kv`

### Movendo uma Secret para outro patch 

`vault secrets move ssh/ ASCES-UNITA_ssh`

### Inserindo vários Chave/Valor em uma secret 

`vault kv put ASCES-UNITA/ADM/NTI/API_MOODLE api_user='moodle' api_version=1 api_ip='10.34.36.7' api_pass='1233k4m5t5t5 rrgtgt'`

### Habilitando um tipo de autenticação e criando o usuário 

`vault auth enable userpass`
`vault write auth/userpass/users/marssilva password=suporte`

### Leitura do contéudo 

`vault read auth/userpass/users/marssilva`

### Autenticando via CLI 

`vault login -method userpass -path userpass username=marssilva password=suporte`

token 
```
Key                    Value
---                    -----
token                  s.49more5LvPcM5bsgFWngzokT
token_accessor         DcEykfh36rlFb65JZq2oQPZK
token_duration         768h
token_renewable        true
token_policies         ["default"]
identity_policies      []
policies               ["default"]
token_meta_username    root
```

# Para realizar a autenticação via CLI com a variável de ambiente VAULT_TOKEN

`export VAULT_TOKEN=s.2lC7Rv4qTbdG2CxDMU8yxAMJ`

# Import arquivo de policies via CLI

`vault  policy write asces-unita first-policy.hcl`

## Instalando o Vault em modo Server

- Criando as pastas
```
mkdir /var/log/vault
mkdir -p /var/liv/vault/data
mkdir /et/vault
touch /etc/vault/config.json
```
- Criando o usuário
```
useradd -r vault
```
- Alterando as permissões 
```
chown -Rv <coloque todas as pastas da parte de cima>
```
- Criando o arquivo no Systemd
```
Em /etc/systemd/system/vault.service
```

```
[Unit]
Description = Vault Server
Requires = network-online.target
After = network-online.target
ConditionFileNotEmpty = /etc/vault/config.json

[Service]
User = vault
Group = vault
Restart = on-failure
ExecStart = /usr/local/bin/vault server -config=/etc/vault/config.json -log-level="trace" 
StandardOutput = /var/log/vault/output.log
StandardError = /var/log/vault/error.log
ExecReload = /bin/kill -HUP $MAINPID
KillSignal = SIGTERM
LimitMEMLOCK = infinity

[Install]
WantedBy = multi-user.target

```

Habilitando o serviço e reboot daemon-reload

`systemctl enable vault.service`
`systemctl daemon-reload`

Exportando a variable of conection

`export VAULT_ADDR="http://127.0.0.1:8200"`

### Inicializando o vault, ele vai mostrar as chaves, então guarde com carinho :)
### Caso você inicie assim, ele vai iniciar com 5 chaves e faz o unseal threshold com 3 chaves, podemos alterar isso setando estes valores.

`vault operator init -key-shares 10 -key-threshold 6`

### Tornando o vault unsel, tecle enter para colocar as chaves, precisa de 3 das 5 criadas  

`vault operator unseal`

### Como gerar uma nova chave do root token, Segura, vai ser longo.....

`vault operator generate-root -init`

### Após rodar este comando, salve o valor de Nonce e OTP, vamos utilizar.
### Agora vamos rodar o unseal da chave, com o valor do nonce obtido do comando acima.

`operator generate-root -nonce 6fb1dee6-7552-feae-aff6-d818aaac5f73`
 ### Repita este processo com a key 6 vezes :)

 ### Agora vamos fazer o decode após o ultimo unseal progress, no final ele vai mostrar o Encoded Token, com este token junto com o OTP gerado no init, vamos obter o token.

 `vault operator generate-root -decode NWsCNjluAEkQIAUTLyJVJlE0MwdZBiN1Kzo -otp FEmwp8mzvJHbysdw9EgdoeI6ft`

### Teremos o nosso root token

### Segurança re-key das chaves existentes, só consegue fazer isso, com as chaves originais.

bf12122d-cc2d-555e-cb8d-564afcc7a2ac

`vault operator rekey -init -key-shares 6 -key-threshold 3`

### Guarde a saida desse comando, se perder, não tem volta.

#### Secrets Engine com AWS, com secrets dinâmicos.

    `vault secrets enable -path AWS_Auth aw`

 #### Escrevendo a secret 

    `vault write AWS_Auth/config/root access_key=<user> secret_key=<key> region=us-east-01` 

 #### Precisamos criar uma ROLE para definir os paṕeis as permissões corretas

 - https://docs.aws.amazon.com/service-authorization/latest/reference/reference_policies_actions-resources-contextkeys.html


 ### Precisamos criar a policy do vault com a AWS, temos o arquivo policy_example_vault.json para fazer a importação, o link acima podemos visualizar o effect, action e Resource.

 - Importação da Policy 

    `vault write  AWS_Auth/roles/role-default credential_type=iam_user policy_document=@policy_example_vault.json`

 - List

    `vault read AWS_Auth/roles/role-default`   

### Vai gerar uma chave com essas permissões, podemos testar com aws-cli setando a chave e key com o comando aws configure, após isso podemos testar com o comando abaixo:

    `aws ec2 describe-instances`

### Habilitando as secrets

   `vault secrets enable -path ssh_local ssh`

### Criando o acesso com OTP com ssh 

- Fazendo o download do vault-ssh-helper, dentro do cliente, salve em /usr/local/bin
   `https://github.com/hashicorp/vault-ssh-helper`

 - Configurando a máquina com arquivo de configuração 
   `mkdir /etc/vault-ssh-helpe.d/`
   `touch /etc/vault-ssh-helpe.d/config.hcl`
   - Contéudo 
       ```
       vault_addr = "http://127.0.0.1:8201"
       tls_skip_verify = false
       ssh_mount_point = "ssh_local"
       allowed_roles = "*"
      ```
 - Configurando o pam e ssh
   - Dentro do pam (/etc/pam.d/sshd) edite a linha que tem '@include common-auth' e coloque o trecho abaixo.
   `auth requisite pam_exec.so quiet expose_authtok log=/var/log/vault-ssh.log /usr/local/bin/vault-ssh-helper -dev -config=/etc/vault-ssh-helpe.d/config.hcl`
   `auth optional pam_unix.so not_set_pass use_first_pass nodelay`
   - Dentro do sshd (/etc/ssh/sshd_config) edite essas variavéis
      - PasswordAuthentication no
      - ChallengeResponseAuthentication yes
      - UsePAM yes            
      - restart ssh

### Vamos atualizar a policy default, com este contéudo e salvar.
   ```
   # To View in web UI
   path "sys/mounts" {
         capabilities = ["read", "update"]
   }  

   # To configure the SSH secrets engine
   path "ssh_local/*" {
         capabilities = ["create", "read", "update", "delete", "list"]
   }  

   # Allow tokens to look up their own properties
   path "auth/token/lookup-self" {
      capabilities = ["read"]
   }
   ```
# Vamos criar a role, user default root, e allowed para marssilva e teste

   `vault write ssh_local/roles/otp_key_role key_type=otp default_user=root cidr_list=0.0.0.0/0 allowed_users='marssilva,teste'`

### Criar a policy com o nome teste, o arquivo é policy_teste_ssh.hcl, e vamos importar esssa policy

   `vault policy write teste policy_teste_ssh_hcl`

### Vamos testar a criação das chaves com outro usuário, vamos lá 

   - Habilitando o userpass
   `vault auth enable userpass`
   - Criando o usuário
   `vault write auth/userpass/users/marssilva password="teste" policies="teste"`
   - Fazendo o login
   `vault login -method userpass username=marssilva`

### Criando a key para o acesso remoto

   `vault write ssh_local/creds/otp_key_role ip=127.0.0.1 username=marssilva`

   - Saída 
   ```
   Key                Value
   ---                -----
   lease_id           ssh_local/creds/otp_key_role/wn2PcdKB23RcM3edjcONiSAr
   lease_duration     6h
   lease_renewable    false
   ip                 127.0.0.1
   key                ca3273de-80b1-052c-aa11-67630434fbe0
   key_type           otp
   port               22
   username           marssilva

   ```
### Podemos fazer o teste com ssh local assim 

   `ssh marssilva@127.0.0.1` # coloque a senha do campo key, só utiliza uma vez :)

### Dica do dolinho, o Vault tem um sub comando ssh para realizar o login, fica assim

   - Com esse comando ele solicita a key e já realiza o login (precisa instalar o sshpass para acessar automático)
   `vault ssh -mode=otp -mount-point ssh_local -role=otp_key_role marssilva@127.0.0.1`











