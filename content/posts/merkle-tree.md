---
publish: false
date: 2021-02-08
title: 'Merkle Tree e o Bitcoin'
slug: 'merkle_tree_and_bitcoin'
image: '/images/header-post-1.jpg'
timetoread: '50 minutos'
description: 'Nesse post gostaria de explicar um conceito muito importante na criptografia e que é aplicado na blockchain.'
---

# Introdução

A árvore de Merkle (Merkle tree) é uma estrutura de dados proposta por Ralph Merkle na qual consiste em uma árvore binária de hashes (hash é o resumo de um conjunto de dados de tamanhos variados para uma cadeia de caracteres de tamanho fixo) onde cada folha tem o hash do bloco de dados e cada nó tem o hash dos hashs dos seus nós filhos. Esse tipo de estrutura é eficiente e ideal para garantir a consistência e ordem dos dados. Caso não conheça árvores binárias pode conhecer mais [aqui](https://www.geeksforgeeks.org/binary-tree-data-structure/)

Para uma fácil visualização de como essa estrutura funciona, imagine que vc e uma quantidade de pessoas compartilham uma receita de fazer bolo de cenoura, e o contrato que todos definiram segue a seguinte ordem:

```
fazer_bolo_contrato := [
    "pegar ovos, açucar, oléo e cenoura",
    "bater tudo no liquidificador",
    "misturar com trigo e fermento",
    "bater a massa por 3m"
    "untar a forma",
    "jogar a massa na forma",
    "preaquecer o forno à 180 graus por 10m",
    "assar a massa que está na forma por 45m"
]
```

Uma vez definido o contrato que todas as pessoas aceitam agora temos uma rede que compartilha desse contrato e confia nele para ter um bolo de cenoura impecável. 

# O Problema

A nossa rede não é centralizada, não existe um ponto central que define como o bolo de cenoura deve ser feito, o nosso contrato é público para escrita e leitura e quem alterar o contrato poderá fazer isso, por um lado isso é bom pois podemos ter variações deliciosas de bolo de cenoura sendo compartilhadas pela rede, entretanto podemos ter pessoas mal intencionadas que podem sabotar o bolo ed cenoura e compartilhar receitas fajutas estragando a confiabilidade da rede. Com isso entra a estrutura que vem solucionar o nosso problema: a **árvore de Merkle** e também teremos a **raiz de Merkle**

# Aplicando a solução na rede

Agora que nossa rede está sucetível a ameaças de fraude devemos fazer com que todo contrato compartilhado passe por uma validação, e essa validação tem por objetivo assegurar que o contrato que estou recebendo está intacto e é a receita de bolo que eu quero.

Para isso toda a rede deve encontrar a **raiz de Merkle** (_A merkle tree é uma árvore binária feita das folhas até chegar na raiz_) do contrato da receita de bolo de cenoura que é confiável, o passo a passo para fazer isso é:

1. Na lista **fazer_bolo_contrato** encontrar o hash de cada item e armazena-los em uma nova lista chamada **fazer_bolo_hashes**

```go
fazer_bolo_hashes := make([]string, 0)

for _, passo := range fazer_bolo_contrato {
    fazer_bolo_hashes = append(fazer_bolo_hashes, encontrar_hash(passo))
}
```

2. A nova lista **fazer_bolo_hashes** deverá ter o mesmo tamanho da lista **fazer_bolo_contrato** só que os valores serão cadeias de caracteres de tamanho fixo parecida com isso:

```
[
    "73AF4C1F9D22DAD36E3610AAD14E13E8058C82A2ABDEC296F345C2DB8B6A7EA5",
    "843F1A289B4767B126EF7B090D183611616311BC1A032591FD6F51211974AAEB",
    "00E002A31CBA1E85B382F3D559B8BFEE2DEE3E53D1521DD1E9CD920C52A8BA70",
    ...
]
```

3. Se algum passo no contrato da receita de fazer bolo for alterado a lista de hashes será completamente diferente, dessa forma já garantimos a