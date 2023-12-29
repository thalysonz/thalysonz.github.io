---
title: "Dicas para análise de código ofuscado"
date: 2023-12-29
# weight: 1
# aliases: ["/first"]
tags: ["code"]
author: "Me"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Analisando código do malware pixpirate"
disableHLJS: true # to disable highlightjs
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: true
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: true
UseHugoToc: true
cover:
    image: "https://cdn-images-1.medium.com/max/800/1*GWZv7nO5EqwGKzWNLJCJ8Q.jpeg" # image path/url
    alt: "<alt text>" # alt text
    caption: "<text>" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: true # only hide on current single page
editPost:
    URL: "https://github.com/thalysonz.github.io/posts/dicas-para-análise-de-código-obfuscado"
    Text: "s" # edit text
    appendFilePath: true # to append file path to Edit link
---


## Introdução

Finalizei minha análise estática no código do malware PixPirate. Uma parte significativa desse período foi dedicada a desofuscar os arquivos JavaScript utilizados pelo malware, que se empenhou bastante em dificultar a análise por parte de terceiros.

Como pode ser visto nas imagens abaixo, todos os arquivos presentes em *"project.zip"*, que é executado pelo malware usando o AutoJS (ferramenta responsável por cuidar da automação JS), estão ofuscados. Então, decidi aproveitar que estou focado nisso e trazer algumas dicas que podem ajudar caso vocês se deparem com algum código JavaScript obfuscado de agora em diante.


![IMG1](https://cdn-images-1.medium.com/max/800/0*H1U5znQyzgslZECB.png)

A ofuscação de código é o processo de torná-lo menos legível para humanos. Ela complica a compreensão das funcionalidades do código, dificultando a sua cópia ou alteração. Isso é alcançado por meio da aplicação de técnicas de ofuscação, que englobam a renomeação de variáveis, modificação da estrutura do código, inserção de código desnecessário ou irrelevante e a remoção de comentários.

Nosso trabalho aqui é tornar esse código acima legível novamente, para que possamos entender como o *malware* atua. Então, vamos lá.


### **1. Formatando o código**

Nas imagens acima, os três arquivos estão basicamente ilegíveis. Porém, formatar ou reestruturar o código já ajudaria muito nosso processo, pois assim conseguimos identificar onde função termina ou começa, separar funções, e assim, facilitar os próximos passos.

Abaixo, segue um exemplo de um dos arquivos antes e depois da formatação.



![IMG2](https://cdn-images-1.medium.com/max/800/0*eSw5f1W4ejdxkLib.png)
![IMG3](https://cdn-images-1.medium.com/max/800/0*VbPWBj-w8vcsHB95.png)

### **2. Alterando os nomes**

O próximo passo será alterar os nomes das funções/classes/variáveis/interfaces(seja lá o que encontrarmos pela frente) de algo estranho para algo que possamos facilmente identificar.


No exemplo a seguir, apresentamos algumas variáveis, constantes e funções, entre outros elementos, com nomes em hexadecimal ou valores aleatórios, visando dificultar a compreensão do código.


![IMG4](https://cdn-images-1.medium.com/max/800/1*r3KG0kugsPm4OZyjCcQ3eA.png)

Aqui temos uma função que basicamente retorna um array de strings, então vamos começar alterando o nome da função de a1_0x13b5 para mw_FunctionReturnStrings. No **VS CODE** apertando o botão direito do mouse e depois “Change all references”, alteramos todas as referências a essa função.

![IMG5](https://cdn-images-1.medium.com/max/800/1*PPkedJg0CioISgRWsfWORA.png)


Podemos seguir renomeando o código de acordo com seu funcionamento e vamos ter algo como as imagens abaixo no final.

![IMG6](https://cdn-images-1.medium.com/max/800/1*LW_Ol5LGvj5QS0bGZooZ2A.png)

![IMG6](https://cdn-images-1.medium.com/max/800/0*qkTWmDifL23lvSuI.png)

![IMG6](https://cdn-images-1.medium.com/max/800/1*vn9Abi_ECIKsHzwah_tBYA.png)

![IMG6](https://cdn-images-1.medium.com/max/800/0*ozB6hWO2b4SMg_qK.png)


### **3. Entendendo o código**

Agora que temos um código legível só precisamos entender o que todo esse código está fazendo. Para isso, podemos seguir as chamadas de funções pela aplicação. Na imagem abaixo, utilizei o [playcode.io](https://playcode.io) para visualizar o resultado da chamada desse array, que é um elemento importante desse código.


![IMG6](https://cdn-images-1.medium.com/max/800/1*tehhaJXOk_vtyXF17CnHvg.png)


### **4. Fluxo da aplicação**


Este é um processo demorado em que você vai juntar tudo o que foi escrito aqui e tentar entender o fluxo como um todo, percebendo que um arquivo pode chamar outro arquivo e assim por diante. Em alguns casos, vamos encontrar código que é apenas lixo, afinal estamos falando de ofuscação. Mas com as dicas acima, todo o entendimento dos próximos arquivos ofuscados que você pegar será mais fácil.

Na imagem abaixo demonstro o final do nosso código JS, onde basicamente todo este código foi usado para chamar o arquivo "permission/empower.js". Assim, demonstro que saímos de algo ilegível para um código muito mais legível e conseguimos entender o fluxo que a aplicação faz com este código. Agora, só precisamos fazer isso com o restante do código.

![IMG6](https://cdn-images-1.medium.com/max/800/0*CsffVrvvv_vRJ9NB.png)