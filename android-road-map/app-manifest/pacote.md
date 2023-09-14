---
layout:
  title:
    visible: true
  description:
    visible: true
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: false
---

# 📦 Pacote

Os objetos de pacote contêm informações de versão sobre a implementação e especificação de um pacote Java. Essas informações de controle de versão são recuperadas e disponibilizadas pela instância ClassLoader que carregou a(s) classe(s). Normalmente, ele é armazenado no manifesto distribuído com as classes.

O conjunto de classes que compõem o pacote pode implementar uma especificação específica e, nesse caso, o título da especificação, o número da versão e as strings do fornecedor identificam essa especificação. Um aplicativo pode perguntar se o pacote é compatível com uma versão específica; consulte o método isCompatibleWith para obter detalhes.

Os números de versão da especificação usam uma sintaxe que consiste em números inteiros decimais não negativos separados por pontos ".", por exemplo "2.0" ou "1.2.3.4.5.6.7". Isso permite que um número extensível seja usado para representar versões principais, secundárias, micro, etc.&#x20;

O título da implementação, a versão e as strings do fornecedor identificam uma implementação e são disponibilizados de forma conveniente para permitir relatórios precisos dos pacotes envolvidos quando ocorre um problema. O conteúdo de todas as três cadeias de implementação é específico do fornecedor. As strings de versão de implementação não têm sintaxe especificada e só devem ser comparadas quanto à igualdade com os identificadores de versão desejados.

Dentro de cada instância do ClassLoader, todas as classes do mesmo pacote java possuem o mesmo objeto Package. Os métodos estáticos permitem que um pacote seja encontrado por nome ou que o conjunto de todos os pacotes conhecidos pelo carregador de classes atual seja encontrado.
