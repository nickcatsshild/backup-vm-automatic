# backup-vm-automatic
Backup de VMS Automaticamente


Para garantir que o backup tenha a data e hora no nome do arquivo VHDX, e também melhorar a confirmação do processo de backup. Dessa forma, o arquivo de backup terá um nome exclusivo com a data e hora da execução, o que ajuda a manter versões distintas de backups, além de melhorar a precisão no envio de e-mails confirmando o sucesso ou falha.

Aqui está uma versão aprimorada do script com a inclusão da data e hora no nome do arquivo VHDX, além de uma confirmação mais detalhada no envio do e-mail:

### **Script PowerShell Ajustado para Backup com Data e Hora no Nome do Arquivo**

```powershell
# Definir variáveis
$VMName = "NomeDaVM"                   # Nome da máquina virtual
$VHDXPath = "C:\Caminho\Para\VHDX"     # Caminho do VHDX da máquina virtual
$BackupPath = "D:\Backup\VHDX"         # Caminho do HD Externo
$Date = Get-Date -Format "yyyyMMdd-HHmmss"  # Formato de data para nome do backup
$EmailFrom = "seu_email@dominio.com"   # Endereço de e-mail de origem
$EmailTo = "destinatario@dominio.com"  # Endereço de e-mail do destinatário
$SMTPServer = "smtp.dominio.com"       # Servidor SMTP
$SMTPPort = 587                        # Porta do servidor SMTP (exemplo para Gmail)
$SMTPUser = "seu_email@dominio.com"    # Usuário para autenticação SMTP
$SMTPPass = "sua_senha"                # Senha para autenticação SMTP
$Subject = "Backup VHDX Concluído"     # Assunto do e-mail

# Função para enviar e-mail
Function Send-Email {
    param (
        [string]$Subject,
        [string]$Body
    )
    
    $smtp = New-Object System.Net.Mail.SmtpClient($SMTPServer, $SMTPPort)
    $smtp.Credentials = New-Object System.Net.NetworkCredential($SMTPUser, $SMTPPass)
    $smtp.EnableSsl = $true
    
    $mailmessage = New-Object system.net.mail.mailmessage
    $mailmessage.from = ($EmailFrom)
    $mailmessage.To.add($EmailTo)
    $mailmessage.Subject = $Subject
    $mailmessage.Body = $Body
    
    try {
        $smtp.Send($mailmessage)
        Write-Host "E-mail enviado com sucesso."
    }
    catch {
        Write-Host "Falha ao enviar e-mail: $_"
    }
}

# Função para realizar o backup
Function Backup-VHDX {
    # Desligar a VM para garantir que o VHDX não está em uso
    Write-Host "Desligando a VM..."
    Stop-VM -Name $VMName -Force

    # Aguardar até que a VM tenha sido totalmente desligada
    Start-Sleep -Seconds 10

    # Definir nome do arquivo de backup com data e hora
    $BackupDestination = "$BackupPath\VHDX_Backup_$Date.vhdx"
    Write-Host "Iniciando o backup do VHDX para $BackupDestination"
    
    # Realizar o backup (copiar o VHDX)
    try {
        Copy-Item -Path $VHDXPath -Destination $BackupDestination -Force -ErrorAction Stop
        Write-Host "Backup do VHDX concluído com sucesso!"
        
        # Enviar e-mail de sucesso
        Send-Email -Subject $Subject -Body "O backup do VHDX foi concluído com sucesso.\nO arquivo foi salvo em: $BackupDestination"
    } catch {
        Write-Host "Falha ao realizar o backup do VHDX: $_"
        
        # Enviar e-mail de erro
        Send-Email -Subject "$Subject - Erro" -Body "Houve um erro durante o processo de backup do VHDX.\nErro: $_"
    }

    # Iniciar a VM novamente
    Write-Host "Iniciando a VM novamente..."
    Start-VM -Name $VMName
}

# Executar o backup
Backup-VHDX
```

### **Principais Modificações:**

1. **Nome do Arquivo com Data e Hora**:
   - O arquivo de backup agora é nomeado com base na data e hora da execução, o que garante que cada backup tenha um nome único. A variável `$Date` usa o formato `yyyyMMdd-HHmmss`, garantindo que o arquivo seja salvo com um nome exclusivo para cada execução, como `VHDX_Backup_20250203-153045.vhdx`.

2. **Mensagem de E-mail**:
   - Se o backup for bem-sucedido, o e-mail incluirá o caminho do arquivo de backup com data e hora no nome.
   - Se ocorrer uma falha durante o processo de backup (por exemplo, se o arquivo não puder ser copiado), o script envia um e-mail informando o erro, incluindo a mensagem de erro detalhada.

3. **Tratamento de Erros**:
   - A opção `-ErrorAction Stop` foi adicionada ao comando `Copy-Item` para garantir que qualquer erro durante a cópia do arquivo interrompa o processo e acione a captura de exceções (dentro do bloco `try-catch`).
   - Caso algum erro aconteça, o script envia um e-mail de erro com a mensagem detalhada do problema, o que torna o processo mais transparente e fácil de diagnosticar.

4. **Confirmação no Console**:
   - O script também exibe mensagens no console, informando o status do processo: se o backup foi realizado com sucesso ou se ocorreu uma falha.

### **Considerações de Envio de E-mail**:
- **Servidor SMTP**: Se você estiver usando um servidor de e-mail como o Gmail ou Outlook, certifique-se de usar os servidores SMTP corretos:
  - Para **Gmail**: `smtp.gmail.com`, porta `587`, e você precisará de uma senha de aplicativo gerada na sua conta do Google.
  - Para **Outlook**: `smtp-mail.outlook.com`, porta `587`.
  
- **Autenticação SMTP**: O PowerShell usa um par de credenciais (usuário e senha) para autenticar o envio do e-mail. Certifique-se de que o servidor SMTP permita conexões de aplicativos (caso contrário, você pode precisar configurar uma senha de aplicativo).

- **Segurança das Senhas**: Em vez de armazenar a senha diretamente no script, você pode considerar o uso de variáveis de ambiente ou um método de criptografia para proteger informações sensíveis.

### **Agendamento com o Task Scheduler**:
Uma vez que o script esteja pronto e testado, você pode agendá-lo para rodar automaticamente todos os dias no **Task Scheduler** do Windows, seguindo os mesmos passos mencionados antes.

---


Sim, você pode adicionar um envio de e-mail ao script PowerShell para confirmar que o backup foi concluído com sucesso (ou com falha). Para isso, você pode usar o cmdlet `Send-MailMessage` do PowerShell, que permite enviar e-mails via SMTP (protocolo de envio de e-mails).

Aqui está como você pode modificar o script para incluir o envio de um e-mail após a execução do backup:

### **Modificando o Script PowerShell para Enviar E-mail**

1. **Adicionar variáveis para o e-mail**:
   - Você precisará configurar o servidor SMTP (por exemplo, Gmail, Outlook, ou qualquer outro servidor SMTP que você tenha acesso).
   - Adicione variáveis com os dados do servidor SMTP, remetente e destinatário.

2. **Adicionar a função para enviar e-mail** após o backup ser concluído.

Aqui está um exemplo de como adicionar o envio de e-mail ao seu script:

```powershell
# Definir variáveis
$VMName = "NomeDaVM"                   # Nome da máquina virtual
$VHDXPath = "C:\Caminho\Para\VHDX"     # Caminho do VHDX da máquina virtual
$BackupPath = "D:\Backup\VHDX"         # Caminho do HD Externo
$Date = Get-Date -Format "yyyyMMdd-HHmmss"  # Formato de data para nome do backup
$EmailFrom = "seu_email@dominio.com"   # Endereço de e-mail de origem
$EmailTo = "destinatario@dominio.com"  # Endereço de e-mail do destinatário
$SMTPServer = "smtp.dominio.com"       # Servidor SMTP
$SMTPPort = 587                        # Porta do servidor SMTP (exemplo para Gmail)
$SMTPUser = "seu_email@dominio.com"    # Usuário para autenticação SMTP
$SMTPPass = "sua_senha"                # Senha para autenticação SMTP
$Subject = "Backup VHDX Concluído"     # Assunto do e-mail

# Função para enviar e-mail
Function Send-Email {
    param (
        [string]$Subject,
        [string]$Body
    )
    
    $smtp = New-Object System.Net.Mail.SmtpClient($SMTPServer, $SMTPPort)
    $smtp.Credentials = New-Object System.Net.NetworkCredential($SMTPUser, $SMTPPass)
    $smtp.EnableSsl = $true
    
    $mailmessage = New-Object system.net.mail.mailmessage
    $mailmessage.from = ($EmailFrom)
    $mailmessage.To.add($EmailTo)
    $mailmessage.Subject = $Subject
    $mailmessage.Body = $Body
    
    try {
        $smtp.Send($mailmessage)
        Write-Host "E-mail enviado com sucesso."
    }
    catch {
        Write-Host "Falha ao enviar e-mail: $_"
    }
}

# Função para realizar o backup
Function Backup-VHDX {
    # Desligar a VM para garantir que o VHDX não está em uso
    Write-Host "Desligando a VM..."
    Stop-VM -Name $VMName -Force

    # Aguardar até que a VM tenha sido totalmente desligada
    Start-Sleep -Seconds 10

    # Realizar o backup (copiar o VHDX)
    $BackupDestination = "$BackupPath\VHDX_Backup_$Date.vhdx"
    Write-Host "Iniciando o backup do VHDX para $BackupDestination"
    Copy-Item -Path $VHDXPath -Destination $BackupDestination -Force

    # Verificar se o backup foi concluído com sucesso
    if (Test-Path $BackupDestination) {
        Write-Host "Backup do VHDX concluído com sucesso!"
        Send-Email -Subject $Subject -Body "O backup do VHDX foi concluído com sucesso. O arquivo foi salvo em: $BackupDestination"
    } else {
        Write-Host "Falha ao realizar o backup do VHDX."
        Send-Email -Subject "$Subject - Erro" -Body "Houve um erro durante o processo de backup do VHDX. Não foi possível concluir a operação."
    }

    # Iniciar a VM novamente
    Write-Host "Iniciando a VM novamente..."
    Start-VM -Name $VMName
}

# Executar o backup
Backup-VHDX
```

### **Explicação das Modificações**
1. **Configurações do E-mail**:
   - As variáveis `$EmailFrom`, `$EmailTo`, `$SMTPServer`, `$SMTPPort`, `$SMTPUser`, e `$SMTPPass` contêm os dados necessários para configurar o envio de e-mail.
   - O `Send-Email` usa esses dados para se conectar ao servidor SMTP e enviar o e-mail.

2. **Função `Send-Email`**:
   - O e-mail é enviado após o backup ser concluído. O corpo do e-mail varia dependendo do sucesso ou falha do processo:
     - Se o backup for bem-sucedido, o e-mail informará que o processo foi concluído com sucesso.
     - Se houver algum erro, o e-mail indicará que houve uma falha.

3. **Envio de E-mail Após o Backup**:
   - Se o arquivo VHDX for copiado com sucesso, o script envia um e-mail confirmando o sucesso.
   - Caso contrário, um e-mail informando que houve uma falha é enviado.

### **Considerações Importantes:**
- **SMTP Server**: Caso você esteja utilizando um servidor de e-mail como o Gmail, por exemplo, o servidor SMTP seria `smtp.gmail.com`, e a porta seria `587`. Você também precisaria habilitar a autenticação de dois fatores e gerar uma senha de aplicativo (se necessário).
- **SSL**: O parâmetro `EnableSsl = $true` garante que a conexão com o servidor de e-mail seja feita de maneira segura.
- **Segurança**: Não é recomendado deixar a senha do e-mail visível no script. Uma boa prática seria armazená-la em uma variável de ambiente ou em um arquivo seguro.

### **Testando o Envio de E-mail**
Antes de automatizar com o agendador de tarefas, teste o envio do e-mail executando o script diretamente no PowerShell para garantir que as configurações de SMTP e as credenciais estão corretas.

