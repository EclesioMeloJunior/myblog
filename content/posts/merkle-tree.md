---
publish: true
date: 2021-03-30
title: 'Merkle Tree: Compartilhando dados de forma segura'
slug: 'merkle_tree_share_safe'
image: '/images/header-post-1.jpg'
timetoread: '10 minutos'
description: 'Nesse post gostaria de explicar uma estrutura de dados que é muito utilizada em algumas blockchains.'
---

# Introdução

[`header image created by Zunnoon Ahmed`](https://unsplash.com/photos/qqvz2MRH8_M?utm_source=unsplash&utm_medium=referral&utm_content=creditShareLink)

A árvore de Merkle (Merkle tree) é uma estrutura de dados proposta por Ralph Merkle na qual consiste em uma árvore binária de hashes (hash é o resumo de um conjunto de dados de tamanhos variados para uma cadeia de caracteres de tamanho fixo) onde cada folha tem o hash do bloco de dados e cada nó tem o hash dos hashs dos seus nós filhos. Esse tipo de estrutura é eficiente e ideal para garantir a consistência e ordem dos dados. Caso não conheça árvores binárias pode conhecer mais [aqui](https://www.geeksforgeeks.org/binary-tree-data-structure/)

Você pode acessar o código desse post [aqui](https://github.com/crypto2lab/mekle-tree-post)

![Merkle Tree structure](/static/images/merkle_tree_ex_2.png)

# O Problema

Imagine que vc e uma quantidade de pessoas compartilham uma receita de fazer bolo de cenoura, e o contrato que todos definiram segue a seguinte ordem:

```
fazer_bolo_contrato := []string{
    "pegar ovos, açucar, oléo e cenoura",
    "bater tudo no liquidificador",
    "misturar com trigo e fermento",
    "bater a massa por 3m",
    "untar a forma",
    "jogar a massa na forma",
    "preaquecer o forno à 180 graus por 10m",
    "assar a massa que está na forma por 45m",
}
```

Uma vez definido o contrato que todas as pessoas aceitam agora temos uma rede que compartilha desse contrato e confia nele para ter um bolo de cenoura impecável.

A nossa rede não é centralizada, não existe um ponto central que define como o bolo de cenoura deve ser feito, o nosso contrato é público para escrita e leitura (quem quiser poderá alterar o contrato e compartilhar com a rede), por um lado isso é bom pois podemos ter variações deliciosas de bolo de cenoura sendo compartilhadas pela rede, entretanto podemos ter pessoas mal intencionadas que podem sabotar o bolo de cenoura e compartilhar receitas fajutas estragando a confiabilidade da rede. Com isso entra a estrutura que vem solucionar o nosso problema: a **árvore de Merkle** e também teremos a **raiz de Merkle**

# Aplicando a solução na rede

Agora que nossa rede está sucetível a ameaças de fraude devemos fazer com que todo contrato compartilhado passe por uma validação, e essa validação tem por objetivo assegurar que o contrato que estou recebendo está intacto e é a receita de bolo que eu quero.

Para isso toda a rede deve encontrar a **raiz de Merkle** (_A merkle tree é uma árvore binária feita das folhas até chegar na raiz_) do contrato da receita de bolo de cenoura que é confiável, o passo a passo para fazer isso é:

1. Na lista **fazer_bolo_contrato** vamos encontrar o hash de cada item e armazena-los em uma nova lista chamada **fazer_bolo_hashes**:

```go
import (
    "crypto/sha256"
)

func encontrar_hash(passo string) []byte {
    h := sha256.Sum256([]byte(passo))
    return h[:]
}

fazer_bolo_hashes := make([][]byte, 0)

for _, passo := range fazer_bolo_contrato {
    fazer_bolo_hashes = append(fazer_bolo_hashes, encontrar_hash(passo))
}
```

2. A nova lista **fazer_bolo_hashes** deverá ter o mesmo tamanho da lista **fazer_bolo_contrato** só que os valores serão cadeias de caracteres de tamanho fixo (hashs) parecida com isso:

```
[
    "73AF4C1F9D22DAD36E3610AAD14E13E8058C82A2ABDEC296F345C2DB8B6A7EA5",
    "843F1A289B4767B126EF7B090D183611616311BC1A032591FD6F51211974AAEB",
    "00E002A31CBA1E85B382F3D559B8BFEE2DEE3E53D1521DD1E9CD920C52A8BA70",
    ...
]
```

3. Se algum passo no contrato da receita de fazer bolo for alterado a lista de hashes será completamente diferente, dessa forma já garantimos a consistência dos dados, o próximo passo é encontrar a raiz de Merkle que irá nos dizer se uma lista está de acordo ou se está alterada, para isso vamos construir a estrutura de dados de uma árvore binária:

```go
// ...
type No struct {
    hashNo []byte

    esquerda *No
    direita *No
}
```

4. Como descrito acima, a estrutura tem a propriedade **hashNo** que armazenará um hash. O primeiro passo para encontrar a raiz de Merkle é construir as folhas da árvore:

```go
// ...
func criar_folhas(hashes [][]byte) []*No {
    folhas := make([]*No, 0)

    for _, hash := range hashes {
        folhas = append(folhas, &No{
            hashNo: hash,
            esqueda: nil,
            direita: nil,
        })
    }

    return folhas
}

// ...
func criar_raiz(hashes [][]byte) *No {
    folhas := criar_folhas(hashes)
}

func main() {
    // ...

    raiz := criar_raiz(fazer_bolo_hashes)
}
```

5. No passo anterior criamos as folhas utilizando os hashes dos passos para fazer um bolo, como estamos criando folhas então os apontamentos para esquerda e direita são nulos pois não haverá nada abaixo das folhas. O proximo passo será criar os nós intermediários a partir das folhas. É importante lembrar que a árvore de merkle é uma árvore binária montada das folhas até a raiz, então, nesse passo, iremos combinar o hash de duas em duas folhas e para cada par de folhas gerar um **No** intermediário.

```go
// ...
func criar_intermediarios(nos []*No) []*No {
	intermediarios := make([]*No, 0)

	// aqui iremos ir de 2 em 2 folhas até o final
	for i := 0; i < len(nos); i += 2 {
		folhaAtual := nos[i]

		// a folhaIrma faz referência a folha que esta
		// ao lado, no caso de uma lista com numero impar
		// uma folha sempre ficará sozinha
		// exemplo: folhas A - B - C, vamos combinar A com B
		// e C ficará sozinha, então vamos combinas C com C
		var folhaIrma *No
		if i+1 >= len(nos) {
			folhaIrma = nos[i]
		} else {
			folhaIrma = nos[i+1]
		}

		// combino o hash das 2 e gero um novo hash
		combinaHash := bytes.Join(
			[][]byte{folhaAtual.hashNo, folhaIrma.hashNo}, []byte{})
		novoHash := sha256.Sum256(combinaHash)

		noIntermediario := &No{
			hashNo:  novoHash[:],
			esqueda: folhaAtual,
			direita: folhaIrma,
		}

		intermediarios = append(intermediarios, noIntermediario)
	}

	return intermediarios
}

func criar_raiz(hashes [][]byte) *No {
    folhas := criar_folhas(hashes)
    intermediarios := criar_intermediarios(folhas)
}
```

6. Veja bem que no passo anterior temos uma condição simples mas que faz toda a diferença, quando temos uma lista impar, o último elemento da lista sempre ficará sozinho e nesse caso o que fazemos e combinar o hash dele com ele mesmo. A função **criar_intermediarios** recebe uma lista de folhas e reduz o tamanho dela pela metade, dessa forma já começamos o afunilamento e estamos tendo a forma de uma árvore. Agora que temos os nós intermediários construidos e fazendo referências as folhas o próximo passo é fazer ir combinando os hashs dos nós intermediários até alcançarmos a raiz. 

```go
func combinar_intermediarios(intermediarios []*No) *No {
	for {
		quantidade_intermediarios := len(intermediarios)

		if quantidade_intermediarios == 1 {
			return intermediarios[0]
		}

		intermediarios = criar_intermediarios(intermediarios)
	}
}

func criar_raiz(hashes [][]byte) *No {
    folhas := criar_folhas(hashes)
    intermediarios := criar_intermediarios(folhas)

    raiz := combinar_intermediarios(intermediarios)
	return raiz
}
```

7. Ao combinarmos os intermediários até chegar a raiz nós reutilizamos a função **criar_intermediarios** para que ela crie intermediários a partir de outros intermediários até que nos reste um único nó que é a nossa raiz de Merkle, o hash da raiz de Merkle é o resultado da combinação de todos os hashes de nossa árvore. Para ver o hash da raiz de merkle:

```go
// ...
func main() {
    // ...

    raiz := criar_raiz(fazer_bolo_hashes)
    fmt.Printf("%x\n", raiz.hashNo)
}
```

```sh
go run main.go
```

8. Pronto!! Temos uma raiz que assegura a ordem e conteúdo de nossa lista, não importa quantas vezes você compile e execute o arquivo o resultado da raiz é sempre o mesmo para a lista **fazer_bolo_contrato**, mas se você, ou alguem mudar a lista o hash da raiz será diferente.

# Conclusão

Agora que você definiu um contrato e econtrou a raiz de merkle desse contrato basta mandar o hash para toda a sua rede! Sempre que alguem mandar uma receita de bolo para outra pessoa a parte que recebe a receita deve encontrar a raiz de Merkle da receita que recebeu e comparar com a raiz de Merkle que assegura o conteúdo e se os valores forem iguais, show de bola a receita é segura, contudo se os valores forem diferentes então quem enviou a receita está querendo sabotar a rede.

