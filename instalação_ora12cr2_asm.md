# Oracle 12cR2 com uso de ASM
## Instalação e Configuração
### Este documento visa orientar na instalação e configuração do Oracle Database 12cR2 utilizando discos ASM recomendados como boa prática pela Oracle.

#### Pré-requisitos de SO:

* Sistema operacional: **Oracle Linux 7**
* Memória RAM mínima: **2GB**
* SWAP: **1.5 * Memória RAM**
* Espaço em disco recomendado: **100G**


#### Instalando Pacotes de Pré-requisitos
Com o sistema operacional já instalado e configurado, vamos instalar os pacotes de pre-instalação do oracle database e os sistemas de configuração do oracle asm. Há algum tempo a Oracle
criou esses pacotes de instalação que simplificam todo o processo de pré-configuração do SO, para receber os softwares.

No terminal, com o usuário **root** execute:

```
$> yum -y install oracle-database-server-12cR2-preinstall
$> yum -y install oracleasm*
$> yum -y install kmod-oracleasm*
```

Atualizar o repositório:

```
$> yum -y update
```

#### Criar Grupos e Usuários
O pacote *oracle-database-server-12cR2-preinstall* já cria os grupos e o usuário oracle automaticamente, porém como também vamos instalar o grid infraestructure para configurar o ASM
alguns grupos e usuários ainda são precisos serem criados manualmente.

No terminal, com o usuário **root** execute:

```
$> groupadd -g 54327 asmdba
$> groupadd -g 54328 asmoper
$> groupadd -g 54329 asmadmin

$> usermod -a -G asmdba oracle

$> useradd -u 54331 -g oinstall -G dba,asmdba,asmoper,asmadmin,racdba grid
```

Altere as senhas do usuário oracle e usuário grid:

```
$> passwd oracle
$> passwd grid
```

> Observação: Muitas vezes a senha informada não atende aos padrões de segurança do Linux, porém se você continuar como se atendesse o Linux aceita a senha menos segura. Não é recomendado, porém como estamos apenas utilizando para laboratório pode ser útil.

#### Configurar e Iniciar o serviço oracleasm
Siga exatamente como abaixo, execute o comando oracleasm para configurar e informe as opções na ordem abaixo:

No terminal, com o usuário **root** execute:

```
$> oracleasm configure -i
grid
asmadmin
y
y
```

Iniciar o oracleasm:

```
oracleasm init
```

No final da configuração, será informado onde o Oracle ASM irá armazenar os discos:

```
/dev/oracleasm
```

#### Criar e Configurar Partições a Serem Utilizadas pelo ASM
Através do VirtualBox fiz a criação de um disco de 30G. A seguir descrevo os passos para a criação das partições no Linux.

Com o usuário **root** vamos executar os comandos abaixo. Nele vamos dividir nosso disco de 30G em 3 partições de 10G cada.

Verificar o disco disponível que acabamos de criar, o comando irá listar todos os discos:

```
$> fdisk -l
```

Com a informação do disco em mãos, vamos criar as partições seguindo exatamente a ordem abaixo:

```
$> fdisk /dev/sdb
n
p
<vazio>
<vazio>
+10G
```

Com esta execução temos o primeiro disco criado, você vai ver que ele vai continuar no utilitário do fdisk, não saia ainda. Repita o passo mais duas vezes e por fim grave as configurações com o comando abaixo:

```
w
```

> Observação: Leia no utilitário fdisk o que está pedindo, você verá e entenderá os comandos listados acima. Onde se lê \<vazio\> não informe nenhum valor apenas siga em frente.

Execute novamente o comando que lista os discos e verifique que ele criou as três partições como queremos:

```
$> fdisk -l
```

Com os partições criadas corretamente, criar os discos no ASM
No terminal, com o usuário **root** execute:

```
$> oracleasm createdisk CRS1 /dev/sdb1
$> oracleasm createdisk DATA1 /dev/sdb2
$> oracleasm createdisk FRA1 /dev/sdb3
```

#### Criando Diretórios para Instalação

No terminal, com o usuário **root** vamos criar os diretórios para o Oracle:

```
$> mkdir -p /u01/app/oracle
$> mkdir -p /u01/app/oracle/product/12.2.0/db_1
$> chown -R oracle:oinstall /u01
```

No terminal, como **root** vamos criar os diretórios para o Grid:

```
$> mkdir -p /u01/app/grid
$> mkdir -p /u01/app/grid/12.2.0/grid_1
$> chown -R grid:oinstall /u01/app/grid
```

Conceder permissão para a raiz:

```
$> chmod -R 775 /u01
```

#### Configurar Variáveis de Ambiente
No terminal, com o usuário **oracle** execute:

```
$> mkdir /home/oracle/scripts
```

```
$> cat > /home/oracle/scripts/setEnv.sh <<EOF
# Oracle Settings
export TMP=/tmp
export TMPDIR=$TMP

export ORACLE_HOSTNAME=jarvis.localdomain
export ORACLE_UNQNAME=prod
export ORACLE_BASE=/u01/app/oracle
export ORACLE_HOME=$ORACLE_BASE/product/12.2.0/db_1
export ORACLE_SID=prod

export PATH=/usr/sbin:usr/local/bin:$PATH
export PATH=$ORACLE_HOME/bin:$PATH

export LD_LIBRARY_PATH=$ORACLE_HOME/lib:lib:usr/lib
export CLASSPATH=$ORACLE_HOME/jlib:$ORACLE_HOME/rdbms/jlib
EOF
```

```
echo ". /home/oracle/scripts/setEnv.sh" >> /home/oracle/.bash_profile
```

No terminal, com o usuário **grid** execute:

```
mkdir /home/grid/scripts
```

```
cat > /home/grid/scripts/setEnv.sh <<EOF
# Grid settings
export ORACLE_SID=+ASM
export GRID_HOME=/u01/app/grid/12.2.0/grid_1
export ORACLE_HOME=$GRID_HOME
export PATH=$ORACLE_HOME/bin:$BASE_PATH:$PATH
export LD_LIBRARY_PATH=$ORACLE_HOME/lib:/lib:/usr/lib
export CLASSPATH=$ORACLE_HOME/JRE:$ORACLE_HOME/jlib:$ORACLE_HOME/rdbms/jlib
EOF
```

```
echo ". /home/grid/scripts/setEnv.sh" >> /home/grid/.bash_profile
```

#### Instalar ASM:
Com tudo pronto, irêmos finalmente realizar a instalação do ASM. Para isso você deve ter em mãos a mídia de instalação do Grid Infraestructure 12cR2.
Normalmente essa mídia é obtida em formato *.zip*, descarregue ela no *$GRID_HOME* e execute o executável:

```
$> ./gridSetup.sh
```

As telas estão no arquivo **Instalação e Configuração Grid - parte gráfica.docx**.

REFERÊNCIAS:

- https://www.youtube.com/watch?v=VFRj2QAdOAg
- http://www.nazmulhuda.info/ora-00845-memory_target-not-supported-on-this-system-during-starting-instance
