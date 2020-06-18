---
layout: default
title: "Processamento Assíncrono e Java"
tags: programacao linguagens
---

# Processamento Assíncrono e Java

* [Threads](#threads)
* [Processamento Síncrono](#sync)
* [Processamento Assíncrono](#async)

Saber como utilizar todo recurso disponível de uma máquina é importante para que uma aplicação possa processar requisições mais rapidamente e, principalmente, escalar quando necessário.  
Antes de entender como aproveitar 100% dos recursos, é importante saber diferenciar processamentos *Síncrono* e  *Assíncrono*, *Concorrência* e *Paralelismo*. Nesse primeiro post eu vou falar sobre processamentos síncrono e assíncrono, que talvez sejam os mais importantes de saber no início.

## Threads <a name="threads"></a>

Threads são linhas de instruções utilizadas pelo processador para rodar diferentes programas. Quando você abre o aplicativo do Spotify e o Google Chrome ao mesmo tempo, esses dois programas estão rodando em Threads diferentes, e um não vai interferir no funcionamento do outro.

Todo programa Java é processado na Thread principal, chamada **main**. Essa Thread é criada automaticamente pela JVM quando o programa começa a rodar.

## Processamentos Síncrono <a name="sync"></a>

Imagina que você entre numa cafeteria e peça um café e um milk shake. O atendente então pega põe o pó na cafeteira, esquenta a água, pega uma xícara, espera que o café fique pronto e chama seu nome. Então ele pega o leite, sorvete, põe no liquidificador, prepara o seu milk shake e chama novamente o seu nome.  
Assim funciona o processamento **síncrono**, no qual uma coisa ocorre depois da outra, como numa fila.

Em Java, é apenas um programa normal, lido de cima a baixo.
Então se eu tenho uma classe `Cafeteria`, com um método para fazer o pedido: 
```Java
public class CafeteriaSync {

    public void fazerPedido(String pedido, String atendente) throws InterruptedException {
        System.out.println("O atendente " + atendente + " está preparando o seu " + pedido);

        -- O sleep substitui o processo de preparação do pedido.
        Thread.sleep(1000);

        System.out.println("O atendente " + atendente + " finalizou o seu " + pedido);
    }
}
```

Na hora de chamar esse método com meus dois pedidos, todo o processo vai acontecer na thread principal, e o tempo de espera será de 2 segundos, pois o programa inteiro será lido de cima a baixo.

```Java
public static void main(String[] args) throws InterruptedException {

    long start = System.currentTimeMillis();

    CafeteriaSync cafe = new CafeteriaSync();
    CafeteriaSync milkShake = new CafeteriaSync();

    cafe.fazerPedido("Café", "1");
    milkShake.fazerPedido("Milk Shake", "2");

    long end = System.currentTimeMillis();

    double total = (end - start) / 1000.0;
    System.out.println("Tempo de espera: " + total + " segundos");

}

> O atendente 1 está preparando o seu Café
> O atendente 1 finalizou o seu Café
> O atendente 2 está preparando o seu Milk Shake
> O atendente 2 finalizou o seu Milk Shake
> Tempo de espera: 2.015 segundos
```

## Processamentos Assíncrono <a name="async"></a>

Neste método de processamento, um atendente faria o seu milk shake enquanto outro faria o seu café. Um processo não interferiria no outro, e os dois seriam feitos ao mesmo tempo. No fim, o seu nome seria chamado apenas uma vez, e as duas bebidas seriam servidas ao mesmo tempo.  

O processamento assíncrono é utilizado apenas quando uma tarefa de **I/O**(Input/Output) é necessária, como fazer uma requisição http, escrever num arquivo ou ler dados de um banco de dados.  

Nesse exemplo, a classe `Cafeteria` precisará implementar a interface `Runnable`. Ela possui apenas um método chamado `run()`, que é responsável por possibilitar o processo em uma thread diferente.

```Java
public class CafeteriaAsync implements Runnable {

    private String pedido;

    public CafeteriaAsync(String pedido) {
        this.pedido = pedido;
    }

    @Override
    public void run() {
        
        System.out.println("O atendente " + Thread.currentThread().getName() + " está preparando seu " + pedido);

        try {
            Thread.sleep(1000);
        }
        catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println("O atendente "+ Thread.currentThread().getName() + " preparou o seu " + pedido);
    }

}
```
&nbsp;
Para poder processar um pedido numa thread diferente, é preciso instanciar um objeto **Thread**, que aceita um objeto da classe **Runnable** e uma `String`, que será o nome da nova Thread, como argumentos.

```Java
public static void main(String[] args) {

    long start = System.currentTimeMillis();

    CafeteriaAsync cafe = new CafeteriaAsync("Café");
    CafeteriaAsync milkShake = new CafeteriaAsync("Milk Shake");

    Thread pedidoCafe = new Thread(cafe, "Thread-1");
    Thread pedidoMilkShake = new Thread(milkShake, "Thread-2");

    pedidoMilkShake.start(); // O método start() inicia uma nova Thread.
    pedidoCafe.run(); // O método run() processa o pedido na Thread main


    long end = System.currentTimeMillis();
    double total = (end - start) / 1000.0;

    System.out.println("Tempo de espera: " + total);
}

> O atendente main está preparando seu Café
> O atendente Thread-2 está preparando seu Milk Shake
> O atendente Thread-2 preparou o seu Milk Shake
> O atendente main preparou o seu Café
> Tempo de espera: 1.014
```

Como dá pra perceber, processar um programa de forma assíncrona não garante que a ordem do resultado será exatamente igual à ordem na qual os objetos foram chamados. O milk shake foi o primeiro pedido a ser chamado no código, mas o segundo na hora de rodar o programa e ainda terminou mais rápido.  
De qualquer forma, todo o programa rodou em apenas 1 segundo, pois os dois pedidos foram processados ao mesmo tempo, sem a interferência de um no outro.

