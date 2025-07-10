# Certificado Blockchain para AET

## Contexto do Projeto

* **Backend:** C# .NET Core
* **Escala:** Nacional (Brasil), milhões de requisições
* **Objetivo:** Garantir unicidade, integridade e inviolabilidade de certificados que associam informações de veículos (placa padrão Mercosul) e cargas (volume, identificação, origem e destino).

## Objetivos

1. Gerar um certificado digital único e à prova de adulteração.
2. Permitir verificação pública e transparente da autenticidade.
3. Suportar alta taxa de requisições (milhões) sem comprometer desempenho.
4. Manter custos operacionais baixos aproveitando rede pública de baixo gas, como Ethereum.

## Propriedades do Blockchain Utilizadas

* **Imutabilidade:** Cada bloco referencia o hash do anterior; alterações inválidas são facilmente detectáveis.
* **Descentralização:** Ledger replicado em múltiplos nós, sem ponto único de falha.
* **Consenso:** Transações validadas pela rede antes de serem confirmadas (Proof-of-Work/Proof-of-Stake).
* **Transparência:** Históricos de transações publicamente auditáveis.

## Arquitetura Proposta

1. **Geração do Payload**

   * Serialização JSON contendo:

     * Placa do veículo (formato Mercosul)
     * Volume e identificação do produto
     * Ponto de partida e destino
2. **Cálculo de Hash**

   * Aplicar SHA‑256 ao JSON serializado → `hashCertificado`
3. **Armazenamento On-Chain**

   * Publicar transação em smart contract na rede Ethereum que grava:

     * `hashCertificado`
     * `timestamp` da transação
     * Endereço da carteira responsável
   * **Opcional (off-chain):** Armazenar JSON completo em IPFS e registrar apenas o CID no contrato para redução de custos de gas.
4. **Resposta ao Cliente**

   * Retornar ao cliente o **TxHash** e número de bloco da transação, para incluir no certificado (no PDF ou QR Code).

## Fluxo de Verificação

1. **Leitura do Certificado**

   * Obter JSON original + `TxHash` (impresso ou via QR Code).
2. **Re-hash Local**

   * Calcular `sha256(JSON)` e comparar com `hashCertificado` na blockchain.
3. **Consulta On-Chain**

   * Recuperar dados do smart contract usando `TxHash`.
   * Confirmar que o `hashCertificado` armazenado bate com o calculado.
4. **Validação**

   * Se consistente → **Autêntico**; caso contrário → **Suspeito**.

## Vantagens desta Abordagem

* **Único e Não Replicável:** Hashes duplicados são detectáveis e rejeitados.
* **Invulnerável a Alterações:** Qualquer modificação nos dados rompe a consistência do hash.
* **Rastreabilidade:** Histórico público de quem e quando emitiu cada certificado.
* **Resiliência:** Mesmo que o servidor central seja comprometido, não é possível adulterar registros on-chain sem consenso.

## Tecnologias e Ferramentas

* **Rede Pública:** Ethereum (preferência por Layer 2 ou sidechains de baixo gas, se necessário).
* **Biblioteca .NET:** [Nethereum](https://nethereum.com/) para interação com contratos Ethereum em C#.
* **Off-Chain Storage (opcional):** IPFS / Filecoin para JSONs grandes.
* **Smart Contract:** Solidity, contrato simples para gravação de hashes e eventos.

## Considerações de Escalabilidade

* **Batching de Transações:** Agrupar múltiplos hashes em uma única tx usando arrays, para economizar gas.
* **Layer 2 / Sidechains:** Avaliar opções como Polygon ou Arbitrum para reduzir custos em picos de requisições.
* **Cache de Consultas:** Utilizar Redis para armazenar resultados de verificação recentes e diminuir consultas on-chain frequentes.

---

> **Próximos Passos:**
> * Escrever smart contract em Solidity e validar em testnet.
> * Integrar Nethereum no backend .NET Core.
> * Planejar estratégia de batching e cache para atender pico de milhões de requisições.

