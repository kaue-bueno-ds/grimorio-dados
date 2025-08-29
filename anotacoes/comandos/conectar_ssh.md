Conectar VS Code via SSH (com chave) — Passo a passo resumido

Cenário: Windows com VS Code, já com a extensão Remote – SSH. Vamos adicionar um novo servidor, configurar a chave e conectar.

1) Pré-requisitos rápidos

OpenSSH no Windows:
Abra o PowerShell e verifique:

ssh -V

2) Adicionar o host ao ~/.ssh/config

Edite C:\Users\<seu_usuário>\.ssh\config e adicione um bloco (ajuste IP/usuário/porta se preciso):

Host 10.8.21.24
  HostName 10.8.21.24
  User udslinux
  IdentityFile ~/.ssh/id_rsa
  IdentitiesOnly yes
  Port 22
  ServerAliveInterval 30
  ServerAliveCountMax 3


💡 Pode reutilizar a mesma chave (id_rsa) em vários servidores.

3) Instalar sua chave pública no servidor

No PowerShell (local), enviando sua id_rsa.pub para authorized_keys do servidor:

$HOST = "udslinux@10.8.21.24"
$PORT = 22

Get-Content "$env:USERPROFILE\.ssh\id_rsa.pub" | `
ssh -p $PORT $HOST "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"


Nesta etapa ele pede senha do servidor (só uma vez).

4) Testar o SSH por chave (terminal)
ssh -p 22 udslinux@10.8.21.24
# (opcional, forçar chave)
# ssh -i $env:USERPROFILE\.ssh\id_rsa -p 22 udslinux@10.8.21.24


Se logar sem pedir senha (pode pedir passphrase se sua chave tiver), está ok.

5) Conectar pelo VS Code

Ctrl+Shift+P → Remote-SSH: Connect to Host… → 10.8.21.24

Selecione Linux quando solicitado.

Aguarde instalar o VS Code Server e abrir a janela remota.

6) Criar/editar arquivos

O Explorador do VS Code opera como o usuário remoto (ex.: udslinux).

Crie/edite arquivos em diretórios onde esse usuário tem escrita (p. ex., sua HOME).

(Opcional) Caso você tenha erro de permissão ao salvar pela UI

Ex.: EACCES: permission denied em uma pasta pertencente ao root.

No servidor, como root, crie/prepare um grupo para colaborar naquele diretório:

groupadd -f hdmdev
usermod -aG hdmdev udslinux


Ajuste o diretório alvo para permitir escrita pelo grupo e herdar o grupo:

chgrp hdmdev /home/hadoop/codes/bradesco/hdm/python
chmod 2775  /home/hadoop/codes/bradesco/hdm/python


Aplique recursivo na subpasta de trabalho (se já existir conteúdo):

chgrp -R hdmdev /home/hadoop/codes/bradesco/hdm/python/report_mentorai
chmod -R g+rwX /home/hadoop/codes/bradesco/hdm/python/report_mentorai
find /home/hadoop/codes/bradesco/hdm/python/report_mentorai -type d -exec chmod g+s {} \;


Atualize a sessão do usuário para “enxergar” o novo grupo:

# como udslinux (sem sudo)
newgrp hdmdev


Alternativas: desconectar/reconectar o Remote-SSH ou reiniciar a sessão.

Se a UI ainda falhar, reinicie o VS Code Server remoto:

Ctrl+Shift+P → Remote-SSH: Kill VS Code Server on Host…, reconecte.

Tente criar/editar novamente pela UI no diretório configurado.

Dicas rápidas

Root no terminal ≠ root na UI: sudo su afeta o terminal, não o Explorador do VS Code.

Prefira grupo + permissões (como acima) em vez de rodar a UI como root.

Se precisar isolar chaves por host, gere outra com ssh-keygen e aponte via IdentityFile no bloco do Host.

Comandos principais (para colar rápido)
# instalar chave pública
$HOST="udslinux@10.8.21.24"; $PORT=22
Get-Content "$env:USERPROFILE\.ssh\id_rsa.pub" | ssh -p $PORT $HOST "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"

# testar
ssh -p 22 udslinux@10.8.21.24

# (permissões por grupo)
sudo groupadd -f hdmdev
sudo usermod -aG hdmdev udslinux
sudo chgrp hdmdev /home/hadoop/codes/bradesco/hdm/python
sudo chmod 2775  /home/hadoop/codes/bradesco/hdm/python
sudo chgrp -R hdmdev /home/hadoop/codes/bradesco/hdm/python/report_mentorai
sudo chmod -R g+rwX /home/hadoop/codes/bradesco/hdm/python/report_mentorai
sudo find /home/hadoop/codes/bradesco/hdm/python/report_mentorai -type d -exec chmod g+s {} \;

# (ativar grupo na sessão)
newgrp hdmdev