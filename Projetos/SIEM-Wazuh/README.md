# 🛡️ Implementação de Laboratório SIEM/XDR com Wazuh

Este repositório documenta o processo passo a passo da implementação de uma solução de monitorização de segurança **SIEM/XDR** utilizando a plataforma **Wazuh** (versão 4.14) num ambiente virtual. O objetivo deste projeto é demonstrar competências práticas em administração de sistemas Linux, virtualização, resolução de problemas (troubleshooting) e segurança defensiva.

---

## 🏗️ Arquitetura do Laboratório

*   **Hipervisor:** Oracle VM VirtualBox
*   **Servidor SIEM:** Ubuntu Server 22.04 LTS (Jammy Jellyfish)
    *   **Configuração:** Instalação All-in-One (Wazuh Indexer, Wazuh Server e Wazuh Dashboard na mesma máquina)
    *   **Recursos Alocados:** 2 vCPUs, 8 GB de RAM, 50 GB de Armazenamento
    *   **Rede:** Adaptador em Ponte (Bridged Adapter) para obter IP na rede local

---

## 🚀 Fase 1: Configuração da Infraestrutura e Servidor

### 1. Configuração da Máquina Virtual (VM)
1. Criação de uma nova VM no VirtualBox com o nome `Wazuh_Server`.
2. Seleção da ISO do **Ubuntu Server 22.04 LTS**.
3. **Configuração Crítica de Rede:** Alteração do adaptador de rede de *NAT* para *Bridged Adapter (Adaptador em Ponte)*. Isto garante que a VM recebe um IP direto do router, permitindo o acesso à interface web a partir do computador anfitrião (host).
4. Inicialização da VM e instalação padrão do Ubuntu Server, garantindo a ativação do **OpenSSH Server** para gestão remota.

### 2. Acesso Remoto (SSH)
Após a instalação, o acesso foi feito remotamente através do terminal do sistema anfitrião para facilitar a execução de comandos:
```bash
ssh o_teu_utilizador@IP_DO_SERVIDOR
```

---

## 🛠️ Fase 2: Resolução de Problemas (Troubleshooting de Disco)

Durante a primeira tentativa de instalação do painel do Wazuh, o processo falhou com a seguinte mensagem de erro no log `/var/log/wazuh-install.log`:
> `dpkg-deb: erro: o subprocesso colar foi morto pelo sinal (Broken pipe) / erro de disco cheio`

**Análise do Problema:** Por padrão, o instalador do Ubuntu Server utiliza LVM (Logical Volume Manager) mas não aloca 100% do espaço do disco virtual para a partição principal (`/`), deixando-a sem espaço suficiente para descompactar os pacotes do Wazuh Dashboard.

**Solução Aplicada:**
1. Limpeza dos pacotes corrompidos:
   ```bash
   sudo apt-get clean
   sudo apt-get autoremove -y
   ```
2. Expansão do volume lógico para ocupar a totalidade dos 50 GB alocados no VirtualBox:
   ```bash
   sudo lvextend -l +100%FREE /dev/ubuntu-vg/ubuntu-lv && sudo resize2fs /dev/ubuntu-vg/ubuntu-lv
   ```
3. Confirmação do espaço livre:
   ```bash
   df -h /
   ```

---

## ⚙️ Fase 3: Instalação do Wazuh Server

Com o problema de espaço resolvido, os resíduos da instalação anterior foram limpos e o script oficial foi executado com sucesso.

1. Atualização do sistema e instalação de dependências:
   ```bash
   sudo apt update && sudo apt upgrade -y && sudo apt install curl libcap2-bin -y
   ```
2. Download do script de instalação do Wazuh:
   ```bash
   curl -sO https://wazuh.com
   ```
3. Execução da instalação automatizada **All-in-One**:
   ```bash
   sudo bash wazuh-install.sh -a
   ```

Após a conclusão (cerca de 10 minutos), o script gerou com sucesso as credenciais de administrador (`admin`) e as chaves de encriptação únicas para o ambiente.

---

## 🖥️ Fase 4: Acesso à Interface Web

1. Abertura do navegador no sistema anfitrião através do endereço: `https://IP_DO_SERVIDOR`
2. Aceitação do aviso de segurança do navegador (normal devido à utilização de certificados SSL autoassinados gerados pelo instalador).
3. Autenticação bem-sucedida no painel de controlo do Wazuh.

---

## 📈 Próximos Passos (Em Desenvolvimento)
* [ ] Criação da máquina virtual para o Agente.
* [ ] Instalação e configuração do Wazuh Agent num endpoint.
* [ ] Simulação de ataques e validação da receção de alertas no Dashboard.

---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

