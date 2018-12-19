# Execução de programas

### Privilégios de execução

Para impedir que os programas do usuário acessem ou modifiquem dados críticos do sistema operacional, o Windows suporta dois níveis de execução de código: modo de usuário e modo de kernel, mais conhecidos por suas variante em inglês: _user mode_ e _kernel mode_.

Os programas comuns rodam em _user mode_, enquanto os serviços internos do SO e _drivers_ rodam em _kernel mode_.

Apesar de o Windows outros sistemas operacionais modernos trabalharem com somente estes dois níveis de privilégios de execução, os processadores Intel e compatíveis suportam quatro níveis, também chamado de anéis \(_rings_\), numerados de 0 a 4. Para _kernel mode_ é utilizado o _ring_ 0 e para _user mode_, o _ring_ 3.

### Dependências

Quando um programador cria um programa, em muitos casos ele utiliza funções de bibliotecas \(ou _libraries_ em inglês\), também chamadas de DLL \(_Dynamic-Link Library_\). Sendo assim, analise o seguinte simples programa em C:

```c
#include <stdio.h>

int main(void) {
    printf("Olá, mundo!\n");
    return 0;
}
```

Este programa utiliza a função _printf\(\)_, que não precisou ser implementada pelo programador. Ao contrário, ele simplesmente a chamou, já que esta está definida no arquivo _stdio.h_.

Quando compilado, este programa terá uma dependência da biblioteca de C \(arquivo **msvcrt.dll** no Windows\) pois o código para a _printf\(\)_ está nela.

Tudo isso garante que vários programadores usem tal função, que tenha sempre o mesmo comportamento se usada da mesma forma. Mas já parou para pensar como a função _printf\(\)_ de fato escreve na tela? Como ela lidaria com as diferentes placas de vídeo, por exemplo?

O fato é que a _printf\(\)_ não escreve diretamente na tela. Ao contrário, a biblioteca de C pede ao _kernel_ através de uma função de sua API que seja feito. Sendo assim, temos, neste caso um EXE que chama uma função de uma DLL que chama o _kernel_. Estudaremos mais a frente como isso é feito.

### Loader

Quando um programa é executado, ele é copiado para a memória e um **processo** é criado para ele. Dizemos então que um processo está rodando. Quem faz este carregamento de um processo em memória é um componente do sistema operacional chamado de _image loader_, presente na biblioteca **ntdll.dll**.

{% hint style="warning" %}
O código do _loader_  roda **antes** do código do programa a ser carregado. É um código comum a todos os processos executados no sistema operacional.
{% endhint %}

Dentre as funções do _loader_ estão:

* Ler os cabeçalhos do arquivo PE a ser executado e alocar a memória necessária para a imagem como um todo, suas seções, etc.
  * As seções são mapeadas para a memória, respeitando-se suas permissões.
* Ler a tabela de importações do arquivo PE a fim de carregar as DLLs requeridas por este e que ainda não foram carregas em memória. Esse processo também é chamado de **resolução de dependências**.
* Preencher a IAT com os endereços das funções importadas.
* Carregar módulos adicionais em tempo de execução, se assim for pedido pelo executável principal \(também chamado de módulo principal\).
* Manter uma lista de módulos carregados por um processo.
* Transferir a execução para o _entrypoint_ \(EP\) do programa.
