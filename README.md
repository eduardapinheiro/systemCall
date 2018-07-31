# systemCall
Chamada de sistema do Kernel Linux

## Chamada de sistema no Kernel do Linux

Neste tutorial estarei ensinando como adicionar uma chamada de sistema no Kernel do Sistema operacional linux. Estarei definindo um  sistema de chamada que imprime no log do Kernel o maior número de dois informados pelo usuário.

O que foi usado?
* SO Linux 16.04.4 no sistema de 32 bits;
* Código fonte do Kernel na versão 4.16.5.

Vamos começar?

**PASSO 1: Abra o terminal e use o seguinte comando para baixar o Kernel:**

`wget https://cdn.kernel.org/pub/linux/kernel/v4.x/linux-4.16.5.tar.xz `

Obs: Você também pode baixá-lo diretamente no navegador: https://www.kernel.org/. 

**PASSO 2: Extraia o código fonte do Kernel da pasta linux-4.16.5.tar.xz usando o seguinte comando:**

`sudo tar -xvf linux-4.16.5.tar.xz -C/usr/src/ `

**PASSO 3: Depois de extrai-lo mude para o diretório de origem do Kernel:**

`cd /usr/src/linux-4.16.5/ `

**PASSO 4: Crie um diretório chamado **maior**:**

` sudo mkdir maior`

**PASSO 5: Altere para este diretório:**

`cd maior`

**PASSO 6: Crie um arquivo "maior.c":**

`sudo gedit maior.c`

Inclua a definição de chamada do sistema:

    #include<linux/kernel.h>

    // os parâmetros serão passados para este função quando o programa de usuário for 
    criado e verificará qual dos dois números é maior

    asmlinkage long sys_maior(int i, int j){

    if(i>j){
    printk("Maior %d",i);
    }

    else if(j>i) {
    printk("Maior %d",j);
    } 

    else if(i==j){
    printk("Numeros iguais %d%d",i,j);
    }

    return 0; 

    }

Salve. 

A função printk() é usada para imprimir mensagens em um arquivo de log do Kernel, e portanto só pode ser chamada do kernel.

**PASSO 7: Crie um Makefile no diretório Maior e adicione:**

`obj-y := maior.o`

Isso é para garantir que o arquivo maior.c seja compilado e incluído no código-fonte do kernel.

**PASSO 8: Volte para o diretório linux-4.16.5 e abra o Makefile:**

`sudo gedit Makefile`

**PASSO 9: Vá para a linha de número 985 que diz:**

`- “core-y + = kernel / certs/ mm / fs / ipc / security / crypto / block /“ `

Altere isto para:

`“core-y += kernel/ mm/ fs/ ipc/ security/ crypto/ block/ maior/“ `

Salve.

**PASSO 10: Vá para o diretório:**

`cd arch/x86/entry/syscalls `

E abra o arquivo:

`sudo gedit syscall_32.tbl `

Adicione a nova chamada de sistema (sys_MAIOR ()) à tabela de chamadas do sistema na seguinte linha no **fim** do arquivo:

`385    i386   hello    sys_maior `

385 - É o número da chamada do sistema. No meu caso foi 385.

**PASSO 11: Volte para o diretório linux-4.16.5 e adicione a nova chamada de sistema (sys_maior ()) no arquivo de cabeçalho de chamada do sistema:
:**

`cd  include/linux/ `

`sudo gedit syscalls.h `
 
Adicione a seguinte linha ao final do arquivo logo antes da declaração #endif:

`asmlinkage long sys_maior(int i, int j); `

**PASSO 12: Antes de compilar o Kernel, você precisa instalar:**

`sudo apt-get install gcc`

`sudo apt-get install libncurses5-dev`

`sudo apt-get update`

`sudo apt-get upgrade`

**PASSO 13: Configure o Kernel:**

`make menuconfig`

Após o comando acima, vai surgir uma janela pop-up. Certifique se o ext4 estava selecionado e, em seguida, salve.

**PASSO 14: Compile o kernel:**

`cd /usr/src/linux-4.16.5/`
`sudo make -j 5 KDEB_PKGVERSION=1.arbitrary-name deb-pkg`

 Ele irá criar alguns arquivos deb em / usr / src /.

**PASSO 15: Depois disso precisamos instalá-los:**

`dpkg -i linux*.deb`

Isto irá criar um novo kernel no seu sistema.

**PASSO 16: Reinicie o seu sistema. Para verificar se o novo Kernel foi instalado ou não:**

`uname -r`

**PASSO 17: Para fazer a chamada do sistema que criamos:**

`cat /proc/kallsyms | grep maior`

Você obterá a seguinte saída

`0000000000000000 T sys_maior`

O que indica que sua chamada de sistema foi adicionada ao Kernel com sucesso.

**PASSO 18: Para testar a chamada de sistema, crie um programa em sua pasta home chamada "maior.c" e digite o seguinte comando:**

    #include<stdio.h>
    #include<sys/syscall.h>
    #include<unistd.h>

    int main(){

    int a,b;
    printf("Informe o primeiro numero:");
    scanf("%d",&a);

    printf("Informe o segundo numero:");
    scanf("%d",&b);

    long int amma = syscall(386,a,b);
    printf(" Retorno %ld\n ", amma);

    return 0;

    }

**PASSO 19: Compile o programa usando o seguinte comando:**
  
` gcc maior.c`

` ./a.out`

Você verá a seguinte linha sendo impressa no terminal: 

`Retorno 0.` 

**PASSO 20: Agora, para verificar a mensagem do kernel, você pode executar o seguinte comando:**

`dmesg`

Isso exibirá o maior número informado.

