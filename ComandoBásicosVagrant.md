## Comandos Essenciais do Vagrant para Gerenciamento de Máquinas Virtuais

Aqui estão os principais comandos do Vagrant para criar, gerenciar e manipular máquinas virtuais de forma eficiente.

## 1. Inicializar um Novo Ambiente Vagrant

```bash
vagrant init
```
Este comando cria um `Vagrantfile` básico no diretório atual, que pode ser editado para configurar o ambiente desejado.

## 2. Iniciar uma Máquina Virtual

```bash
vagrant up
```
Este comando inicia a máquina virtual conforme as configurações especificadas no `Vagrantfile`. Se a VM ainda não existir, o Vagrant irá criá-la.

## 3. Acessar a Máquina Virtual via SSH

```bash
vagrant ssh
```
Permite acessar a máquina virtual em execução através do SSH, proporcionando um terminal para trabalhar diretamente dentro da VM.

## 4. Pausar a Máquina Virtual

```bash
vagrant suspend
```
Este comando suspende a máquina virtual, salvando seu estado atual em disco. Quando retomada, a VM retorna exatamente de onde parou.

## 5. Retomar uma Máquina Virtual Suspensa

```bash
vagrant resume
```
Este comando retoma a execução da VM suspensa, restaurando-a para o estado em que estava antes da suspensão.

## 6. Desligar uma Máquina Virtual

```bash
vagrant halt
```
Este comando desliga a máquina virtual de forma ordenada, semelhante ao comando de "shutdown" dentro do sistema operacional.

## 7. Excluir uma Máquina Virtual

```bash
vagrant destroy
```
Este comando remove a máquina virtual completamente do sistema, liberando o espaço em disco que ela ocupava. A configuração do `Vagrantfile` permanece intacta.

## 8. Recarregar a Máquina Virtual

```bash
vagrant reload
```
Este comando reinicia a máquina virtual, aplicando qualquer alteração feita no `Vagrantfile` sem a necessidade de destruí-la.

## 9. Verificar o Status da Máquina Virtual

```bash
vagrant status
```
Este comando exibe o estado atual da máquina virtual (por exemplo, se está "running", "halted", etc.).

## 10. Listar Máquinas Virtuais

```bash
vagrant global-status
```
Este comando mostra o status de todas as máquinas virtuais Vagrant no seu sistema, facilitando o gerenciamento de múltiplas VMs.

## 11. Sincronizar o Ambiente com o Vagrantfile

```bash
vagrant provision
```
Se você adicionou ou modificou scripts de provisionamento, este comando reaplica esses scripts sem precisar reiniciar a máquina.

---

Esses são os comandos mais importantes para gerenciar ambientes Vagrant. Eles permitem que você controle todos os aspectos do ciclo de vida da máquina virtual de maneira eficiente e automatizada.
Esse guia cobre as operações básicas e essenciais para gerenciar máquinas virtuais no Vagrant caso tenha curiosidade em saber mais clique [aqui](https://github.com/JardelSO/eng/blob/main/VagrantFile) para abrir a documentação oficial.
