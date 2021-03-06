---
layout: post
title: Visão geral sobre a integração web
categories: [integração, ecommerce]
tags: [web, ecommerce, stone]
fullview: true
---

## Índice

1. [Integração web e e-commerce](#integrao-web-e-e-commerce)
2. [Protocolo](#protocolo)
3. [Mensagens](#mensagens)
   1. [Mensagens de Autorização](#mensagens-de-autorizao)
   2. [Autorização com Captura](#autorizao-com-captura)
   3. [Autorização com Captura posterior](#autorizao-com-captura-posterior)
   4. [Reversão](#reverso)
   5. [Cancelamento](#cancelamento)
   6. [Cancelamento com Captura](#cancelamento-com-captura)
   7. [Cancelamento com Captura posterior](#cancelamento-com-captura-posterior)
4. [Códigos de Exemplo de integração](#cdigos-de-exemplo-de-integrao)


## Integração web e e-commerce

O processo de compra em uma loja virtual começa sempre com o cliente consumidor escolhendo um ou mais produtos, adicionando-os ao carrinho, fazendo revisão, até chegar ao pagamento por aqueles produtos. O cliente consumidor é sempre o ponto de partida de qualquer integração web com sistemas de pagamento. Independentemente de ser um pagamento com recorrência, compra com um clique, autorização ou, até mesmo, uma captura futura, sempre haverá o participante cliente consumidor tomando a decisão por adquirir um produto ou serviço.

Uma vez tomada a decisão, por parte do cliente consumidor, por efetuar o pagamento através de cartão de crédito, débito, voucher, etc, dois novos participantes iniciam uma comunicação através de trocas de mensagens, para autorizar a transação e fazer sua captura.

1. Adquirente
2. Banco emissor do cartão

Adquirente é a empresa responsável por fazer a comunicação com o banco emissor. Cada banco emissor possui seu próprio fluxo, meio de comunicação, tecnologia de segurança, protocolo, etc. Quando uma loja virtual faz a integração através de um adquirente, está se beneficiando de todo um investimento em tecnologias e segurança, além de um mecanismo único para se comunicar com o banco emissor, independentemente da tecnologia utilizada pelo banco.

Se a loja virtual precisasse aceitar pagamentos através de três bandeiras de cartão de crédito diferentes, três processos de integração diferentes seriam necessários. Como o adquirente já fez esse investimento no processo de integração, o custo de integração través adquirente é muito menor para a loja, do que seria se a integração fosse feita diretamente com as bandeiras.

## Fluxo de comunicação

Toda a comunicação começa com o cliente consumidor, portador do cartão, optando por fazer o pagamento com um cartão aceito pela loja virtual. A loja virtual, por sua vez, envia uma mensagem para o adquirente, solicitando o processamento do pagamento. Assim como ocorre com pagamentos em maquinetas, o cliente precisará informar seus dados de autenticação e o pagamento poderá ser autorizado ou negado pelo adquirente, seja por informações incorretas, ou algum eventual problema.

![fluxo](/assets/media/fluxo.png)

## Protocolo

Durante o fluxo de comunicação, algumas mensagens serão enviadas a partir da loja online, para o Adquirente. Essas mensagens são enviadas no formato XML através do protocolo HTTP, utilizando a arquitetura REST. Essa arquitetura permite que representações da transação possam ser enviadas ou recebidas pela loja online, segundo seu estado atual.

Por exemplo, para criar uma autorização, a loja deverá enviar uma mensagem [AcceptorAuthorisationRequest](#) para o adquirente, que irá checar com o banco emissor se o cliente consumidor possui recursos para financiar o pagamento. Nessa mensagem, além dos dados de credenciamento da loja, a loja enviará os dados do dono do cartão e os dados da transação, como valor total, número de parcelas, etc.

## Mensagens

Todas as mensagens serão enviadas através do protocolo HTTP. Para o ambiente de produção, as mensagens deverão ser enviadas utilizando o método **POST** para [http://pos.stone.com.br:3989](http://pos.stone.com.br:3989). O endpoint de testes será liberado e definido de acordo com cada integrador.

### Mensagens de Autorização

É o processo de troca de mensagens entre o estabelecimento e o adquirente onde é verificado se portador do cartão possui ou não saldo suficiente para a realização de um pagamento. A diferença entre a mensagem de autorização com captura automática e com captura posterior, está no elemento `TxCaptr` que deve ser informado `true` para Autorização com Captura, ou `false` para Autorização com Captura posterior.

#### XML de requisição comentado
A mensagem de `AcceptorAuthorisationRequest` é enviada pelo estabelecimento para o adquirente, para checar junto ao banco que a conta associada ao cartão possui recursos para financiar o pagamento. Este controle inclui a validação dos dados do cartão e todos os dados adicionais previstos.

    <Document xmlns="urn:AcceptorAuthorisationRequestV02.1">
        <AccptrAuthstnReq>
            <!-- Cabeçalho da requisição -->
            <Hdr>
                <!-- Identifica o tipo de processo em
                     que a mensagem se propõe.
                     Valor Fixo: “AUTQ” = AuthorisationRequest. -->
                <MsgFctn>AUTQ</MsgFctn>
                <!-- Versão do protocolo utilizado na mensagem. -->
                <PrtcolVrsn>2.0</PrtcolVrsn>
            </Hdr>
            <!-- Dados da requisição de autorização. -->
            <AuthstnReq>
                <!-- Ambiente da transação. -->
                <Envt>
                    <!-- Dados do estabelecimento. -->
                    <Mrchnt>
                        <!-- Identificação do estabelecimento. -->
                        <Id>
                            <!-- Identificação do estabelecimento comercial no adquirente.
                                 Também conhecido internamente como “SaleAffiliationKey”. -->
                            <Id>BFDB58AB9A8A48828C2647E18B7F1114</Id>
                        </Id>
                    </Mrchnt>
                    <!-- Dados do ponto de interação -->
                    <POI>
                        <!-- Identificação do ponto de interação -->
                        <Id>
                            <!-- Código de identificação do ponto de interação
                                 atribuído pelo estabelecimento. -->
                            <Id>2FB4C89A</Id>
                        </Id>
                        <!-- Capacidades do Ponto de interação. -->
                        <Cpblties>
                            <!-- Número máximo de colunas de cada linha a ser impressa
                                 no cupom. A quantidade mínima de colunas é de 38.
                                 Se o POI enviar menos do que 38, o Host Stone não irá
                                 retornar os dados do recibo. -->
                            <PrtLineWidth>50</PrtLineWidth>
                        </Cpblties>
                    </POI>
                    <!-- Dados do cartão utilizado na transação. -->
                    <Card>
                        <!-- Dados não criptografados do cartão utilizado na transação. -->
                        <PlainCardData>
                            <!-- Número do cartão. (Primary Account Number) -->
                            <PAN>4066559930861909</PAN>
                            <!-- Data de validade do cartão no formato “yyyy-MM”. -->
                            <XpryDt>2017-10</XpryDt>
                        </PlainCardData>
                    </Card>
                </Envt>
                <!-- Informações da transação a ser realizada. -->
                <Cntxt>
                    <!-- Informações sobre o pagamento. -->
                    <PmtCntxt>
                        <!-- Modo da entrada dos dados do cartão.
                             PHYS = Ecommerce ou Digitada; -->
                        <CardDataNtryMd>PHYS</CardDataNtryMd>
                        <!-- Tipo do canal de comunicação utilizado na transação.
                             ECOM = Ecommerce ou Digitada; -->
                        <TxChanl>ECOM</TxChanl>
                    </PmtCntxt>
                </Cntxt>
                <!-- Informações da transação. -->
                <Tx>
                    <!-- Identificação da transação definida pelo sistema que se
                         comunica com o Host Stone. -->
                    <InitrTxId>123123123</InitrTxId>
                    <!-- Indica se os dados da transação devem ser capturados (true)
                         ou não (false) imediatamente. -->
                    <TxCaptr>false</TxCaptr>
                    <!-- Dados de identificação da transação atribuída pelo
                         POI (Ponto de interação). -->
                    <TxId>
                        <!-- Data local e hora da transação atribuídas pelo
                             POI (ponto de interação). -->
                        <TxDtTm>2014-03-12T15:11:06</TxDtTm>
                        <!-- Identificação da transação definida pelo ponto de interação (POI,
                             estabelecimento, lojista, etc). O formato é livre contendo no
                             máximo 32 caracteres. -->
                        <TxRef>06064f516a50483da7f189243c95ccca</TxRef>
                    </TxId>
                    <!-- Detalhes da transação. -->
                    <TxDtls>
                        <!-- Moeda utilizada na transação em conformidade com a ISO 4217.
                             986 = BRL = Real Brasileiro
                             http://pt.wikipedia.org/wiki/ISO_4217 -->
                        <Ccy>986</Ccy>
                        <!-- Valor total da transação em centavos. -->
                        <TtlAmt>100</TtlAmt>
                        <!-- Modalidade do cartão utilizado na transação.
                             CHCK = Débito;
                             CRDT = Crédito. -->
                        <AcctTp>CRDT</AcctTp>
                        <!-- Os dados relativos à(s) parcela(s) ou a uma transação recorrente. -->
                        <RcrngTx>
                            <!-- Tipo de parcelamento.
                                 NONE = Nenhum,
                                 MCHT = Lojista -->
                            <InstlmtTp>NONE</InstlmtTp>
                            <!-- Número do total de parcelas. -->
                            <TtlNbOfPmts>0</TtlNbOfPmts>
                        </RcrngTx>
                    </TxDtls>
                </Tx>
            </AuthstnReq>
        </AccptrAuthstnReq>
    </Document>

#### XML de resposta comentado
A mensagem `AcceptorAuthorisationResponse` é enviada pelo adquirente, para retornar o resultado da validação realizada pelo emissor sobre a operação de pagamento.

    <Document xmlns="urn:AcceptorAuthorisationResponseV02.1">
        <AccptrAuthstnRspn>
            <!-- Cabeçalho da mensagem -->
            <Hdr>
                <!-- Identifica o tipo de processo em que a mensagem se propõe.
                     AUTP = AuthorisationResponse. -->
                <MsgFctn>AUTP</MsgFctn>
                <!-- Versão do protocolo utilizado na mensagem. -->
                <PrtcolVrsn>2.0</PrtcolVrsn>
                <!-- Dados para rastreamento da mensagem. -->
                <Tracblt>
                    <!-- Data e hora da saída da mensagem no Host Stone. -->
                    <TracDtTmOut>2014-03-12T18:10:58</TracDtTmOut>
                </Tracblt>
            </Hdr>
            <!-- Informações relacionadas à resposta da autorização. -->
            <AuthstnRspn>
                <!-- Ambiente da transação. -->
                <Envt>
                    <!-- Dados do estabelecimento. -->
                    <MrchntId>
                        <!-- dentificação do estabelecimento comercial no adquirente. Também
                             conhecido internamente como “SaleAffiliationKey”. -->
                        <Id>BFDB58AB9A8A48828C2647E18B7F1114</Id>
                    </MrchntId>
                    <!-- Dados do ponto de interação. -->
                    <PoiId>
                        <!-- Identificador do ponto de interação -->
                        <Id>2FB4C89A</Id>
                    </PoiId>
                </Envt>
                <!-- Informações da transação. -->
                <Tx>
                    <!--- Dados de identificação da transação atribuída pelo POI -->
                    <TxId>
                        <!-- Data local e hora da transação atribuído pelo POI.
                             Este campo será ecoado pelo adquirente. -->
                        <TxDtTm>2014-03-12T15:11:06</TxDtTm>
                        <!-- Identificação da transação atribuída pelo POI.
                             Este campo será ecoado pelo adquirente. -->
                        <TxRef>06064f516a50483da7f189243c95ccca</TxRef>
                    </TxId>
                    <!-- Identificação da transação definida pela Stone. -->
                    <RcptTxId>00000034071000000215353</RcptTxId>
                    <!-- Detalhes da transação. -->
                    <TxDtls>
                        <!-- Moeda utilizada na transação em conformidade com a ISO 4217.
                             986 = BRL = Real Brasileiro
                             http://pt.wikipedia.org/wiki/ISO_4217 -->
                        <Ccy>986</Ccy>
                        <!-- Valor total autorizado em centavos. -->
                        <TtlAmt>100</TtlAmt>
                        <!-- Modalidade do cartão utilizado na transação.
                             CHCK = Débito;
                             CRDT = Crédito. -->
                        <AcctTp>CHCK</AcctTp>
                    </TxDtls>
                </Tx>
                <!-- Dados de resposta da transação. -->
                <TxRspn>
                    <!-- Resultado da autorização. -->
                    <AuthstnRslt>
                        <!-- Dados da resposta da autorização. -->
                        <RspnToAuthstn>
                            <!-- Resposta da transação.
                                 DECL: Declined
                                 APPR: Aproved
                                 PART: PartialApproved
                                 TECH: TechinicalError -->
                            <Rspn>APPR</Rspn>
                            <!-- Código de resposta da autorização
                                 (equivalente ao campo 39 da ISO 8583 de 2003). -->
                            <RspnRsn>0000</RspnRsn>
                        </RspnToAuthstn>
                        <!-- Código de autorização retornado pelo emissor. -->
                        <AuthstnCd>007091</AuthstnCd>
                        <!-- Indica se a mensagem precisa ser capturada posteriormente. -->
                        <CmpltnReqrd>false</CmpltnReqrd>
                    </AuthstnRslt>
                    <!-- Informações sobre as ações a serem realizadas pelo POI.
                         Neste caso, as informações a serem impressas no comprovante. -->
                    <Actn>
                        <!-- Tipo da ação a ser realizada pelo POI.
                             PRNT = Imprimir mensagem;
                             DISP = Exibir mensagem. ->
                        <ActnTp>PRNT</ActnTp>
                        <!-- Mensagem a ser impressa no terminal. -->
                        <MsgToPres>
                            <!-- A quem se destina a mensagem no contexto.
                                 CRCP = CardholderReceipt;
                                 MRCP = MerchantReceipt. -->
                            <MsgDstn>CRCP</MsgDstn>
                            <!-- Texto ou dados gráficos a serem impressos na tela ou em papel
                            no terminal. A mensagem será formatada conforme o campo PrintLineWidth
                            da requisição. -->
                            <MsgCntt>Informacoes formatadas do recibo aqui</MsgCntt>
                        </MsgToPres>
                    </Actn>
                </TxRspn>
            </AuthstnRspn>
        </AccptrAuthstnRspn>
    </Document>

### Autorização com Captura

É o processo de autorização de uma transação onde a captura financeira é realizada no momento do pedido de autorização.

![autorização com captura](/assets/media/autorizacao-e-captura.png)

1. O estabelecimento envia uma mensagem `AcceptorAuthorisationRequest` para o adquirente solicitando autorização, informando que deseja realizar a captura financeira atribuindo o valor da tag `<TxCaptr>` como `true`
2. `AcceptorAuthorisationResponse` é devolvido pelo adquirente, informando ao estabelecimento sobre o êxito do pedido, uma vez que o adquirente tenha autorizado a transação sem a necessidade de envio de captura.

### Autorização com Captura posterior

O estabelecimento pode escolher enviar ou não a mensagem de captura para uma transação de crédito. Para as transações de débito, a transação de autorização já é realizada sem a necessidade da mensagem de captura, mesmo que a tag `<TxCaptr>` esteja preenchida como `false`.

![autorização com captura posterior](/assets/media/autorizacao-com-captura-posterior.png)

1. O estabelecimento envia uma `AcceptorAuthorisationRequest` ao adquirente solicitando a autorização da transação.
2. Uma `AcceptorAuthorisationResponse` é devolvida pelo adquirente, informando ao estabelecimento sobre o êxito da autorização.
3. Se a transação tiver sido concluída com êxito no lado do estabelecimento, o estabelecimento envia uma `AcceptorCompletionAdvice` para capturar a transação.
4. O adquirente retorna uma `AcceptorCompletionAdviceResponse`, reconhecendo o resultado e captura financeira da transação.

#### XML de requisição de Captura comentado
A mensagem `AcceptorCompletionAdvice` é enviada pelo estabelecimento para confirmar uma transação previamente autorizada. Esta mensagem também é utilizada como um pedido de desfazimento de transações.

    <Document xmlns="urn:AcceptorCompletionAdviceV02.1">
        <AccptrCmpltnAdvc>
            <!-- Cabeçalho da mensagem. -->
            <Hdr>
                <!-- Identifica o tipo de processo em que a mensagem se propõe.
                     CMPV = Completion Advice
                     RVRA = ReversalAdvice. -->
                <MsgFctn>CMPV</MsgFctn>
                <!-- Versão do protocolo utilizado na mensagem. -->
                <PrtcolVrsn>2.0</PrtcolVrsn>
            </Hdr>
            <!-- Informações relacionadas ao processo de captura ou desfazimento de uma autorização. -->
            <CmpltnAdvc>
                <!-- Ambiente da transação. -->
                <Envt>
                    <!-- Dados do estabelecimento. -->
                    <Mrchnt>
                        <!-- Identificação do estabelecimento comercial no adquirente.
                             Também conhecido internamente como “SaleAffiliationKey”. -->
                        <Id>00000000000000000000000000000321</Id>
                    </Mrchnt>
                </Envt>
                <!-- Dados da transação. -->
                <Tx>
                    <!-- Identificação da transação definida pelo sistema que se comunica com
                         o Host Stone. -->
                    <InitrTxId>123123123</InitrTxId>
                    <!-- Dados de identificação da transação atribuída pelo POI. -->
                    <TxId>
                        <!-- Data local e hora da transação atribuído pelo POI (ponto de
                             interação). Este campo será ecoado pelo adquirente. -->
                        <TxDtTm>2014-06-11T17:15:44</TxDtTm>
                        <!-- Identificação da transação atribuída pelo POI (Ponto de
                             interação). Este campo será ecoado pelo adquirente. -->
                        <TxRef>1111</TxRef>
                    </TxId>
                    <!-- Identificação da transação original -->
                    <OrgnlTx>
                        <!-- Identificação da transação definida pelo adquirente. -->
                        <RcptTxId>9CDF257AQKR</RcptTxId>
                    </OrgnlTx>
                    <!-- Detalhes da transação. -->
                    <TxDtls>
                        <!-- Moeda utilizada na transação em conformidade com a ISO 4217. -->
                        <Ccy>986</Ccy>
                        <!-- Valor total da transação em centavos. -->
                        <TtlAmt>100</TtlAmt>
                    </TxDtls>
                </Tx>
            </CmpltnAdvc>
        </AccptrCmpltnAdvc>
    </Document>

#### XML de resposta de Captura comentado
A mensagem `AcceptorCompletionAdviceResponse` é enviada pelo adquirente para avisar o estabelecimento sobre o reconhecimento do resultado da operação de pagamento, bem como a transferência dos dados financeiros da transação contidas no `AcceptorCompletionAdvice`. Esta mensagem também é utilizada como resposta para o processo de “desfazimento de transações”.

    <Document xmlns="urn:AcceptorCompletionAdviceResponseV02.1">
        <AccptrCmpltnAdvcRspn>
            <!-- Cabeçalho da mensagem. -->
            <Hdr>
                <!-- Identifica o tipo de processo em que a mensagem se propõe.
                     CMPK = CompletionAdviceResponse ou
                     RVRR = ReversalAdviceResponse. -->
                <MsgFctn>CMPK</MsgFctn>
                <!-- Versão do protocolo utilizado na mensagem. -->
                <PrtcolVrsn>2.0</PrtcolVrsn>
            </Hdr>
            <!-- Informações sobre a resposta da captura ou desfazimento de uma autorização. -->
            <CmpltnAdvcRspn>
                <!-- Ambiente da transação. -->
                <Envt>
                    <!-- Dados do estabelecimento. -->
                    <Mrchnt>
                        <!-- Identificação do estabelecimento comercial no adquirente.
                             Também conhecido internamente como “SaleAffiliationKey”. -->
                        <Id>BFDB58AB9A8A48828C2647E18B7F1114</Id>
                    </Mrchnt>
                </Envt>
                <!-- Dados da transação. -->
                <Tx>
                    <!-- Dados da identificação da transação definida pelo POI. -->
                    <TxId>
                        <!-- Data e hora da transação -->
                        <TxDtTm>2014-03-12T15:17:59</TxDtTm>
                        <!-- Identificação da transação definida pelo ponto de interação (POI,
                             estabelecimento, lojista, etc). Este campo será ecoado pelo adquirente. -->
                        <TxRef>7ca686eb242b4c0482c58961f5d3aac7</TxRef>
                    </TxId>
                    <!-- Resultado da transação.
                         DECL = Declined,
                         APPR = Approved,
                         PART = Partial Approved,
                         TECH = Technical Error. -->
                    <Rspn>APPR</Rspn>
                </Tx>
            </CmpltnAdvcRspn>
        </AccptrCmpltnAdvcRspn>
    </Document>

### Reversão

A reversão ou desfazimento de uma transação é realizado quando há alguma falha na transação. O prazo para a realização do desfazimento é de 5 dias e o tempo máximo de espera para de uma reposta `AcceptorCompletionAdvice` é de 50 segundos.

![reversão](/assets/media/reversao.png)

1. Uma `AcceptorAuthorisationRequest` é enviada pelo estabelecimento para o adquirente solicitando autorização.
2. Uma `AcceptorAuthorisationResponse` é devolvida pelo adquirente informando ao estabelecimento sobre o êxito do pedido, uma vez que o adquirente tenha autorizado a transação.
3. No caso de falha no estabelecimento, uma `AcceptorCompletionAdvice` é então enviada para o adquirente para desfazer a transação.
4. O adquirente envia de volta uma `AcceptorCompletionAdviceResponse` reconhecendo o desfazimento.

#### XML de requisição de Desfazimento comentado

    <Document xmlns="urn:AcceptorCompletionAdviceV02.1">
        <AccptrCmpltnAdvc>
            <!-- Cabeçalho da mensagem. -->
            <Hdr>
                <!-- Identifica o tipo de processo em que a mensagem se propõe.
                CMPK = CompletionAdviceResponse ou
                RVRR = ReversalAdviceResponse. -->
                <MsgFctn>RVRA</MsgFctn>
                <!-- Versão do protocolo utilizado na mensagem. -->
                <PrtcolVrsn>2.0</PrtcolVrsn>
            </Hdr>
            <!-- Informações relacionadas ao processo de captura ou desfazimento de uma autorização. -->
            <CmpltnAdvc>
                <!-- Ambiente da transação. -->
                <Envt>
                    <!-- Dados do estabelecimento. -->
                    <Mrchnt>
                        <!-- Container contendo o identificador do lojista. -->
                        <Id>
                            <!-- Identificação do estabelecimento comercial no adquirente.
                                 Também conhecido internamente como “SaleAffiliationKey”. -->
                            <Id>00000000000000000000000000000321</Id>
                        </Id>
                    </Mrchnt>
                </Envt>
                <!-- Dados da transação. -->
                <Tx>
                    <!-- Dados de identificação da transação atribuída pelo POI -->
                    <TxId>
                        <!-- Data local e hora da transação atribuído pelo POI.
                             Este campo será ecoado pelo adquirente. -->
                        <TxDtTm>2014-06-11T17:15:44</TxDtTm>
                        <!-- Identificação da transação atribuída pelo POI.
                             Este campo será ecoado pelo adquirente. -->
                        <TxRef>1111</TxRef>
                    </TxId>
                    <!-- Identificação da transação original -->
                    <OrgnlTx>
                        <!-- Identificação da transação definida pelo adquirente. -->
                        <RcptTxId>9CDF257AQKR</RcptTxId>
                    </OrgnlTx>
                    <!-- Detalhes da transação. -->
                    <TxDtls>
                        <!-- Moeda utilizada na transação em conformidade com a ISO 4217. -->
                        <Ccy>986</Ccy>
                        <!-- Valor total da transação em centavos. -->
                        <TtlAmt>100</TtlAmt>
                    </TxDtls>
                </Tx>
            </CmpltnAdvc>
        </AccptrCmpltnAdvc>
    </Document>

#### XML de resposta de Desfazimento comentado
A mensagem `AcceptorCompletionAdviceResponse` é enviada pelo adquirente para avisar o estabelecimento sobre o reconhecimento do resultado da operação de pagamento, bem como a transferência dos dados financeiros da transação contidas no AcceptorCompletionAdvice. Esta mensagem também é utilizada como resposta para o processo de “desfazimento de transações”.

    <Document xmlns="urn:AcceptorCompletionAdviceResponseV02.1">
        <AccptrCmpltnAdvcRspn>
            <!-- Cabeçalho da mensagem. -->
            <Hdr>
                <!-- dentifica o tipo de processo em que a mensagem se propõe.
                CMPK = CompletionAdviceResponse ou
                RVRR = ReversalAdviceResponse. -->
                <MsgFctn>RVRR</MsgFctn>
                <!-- Versão do protocolo utilizado na mensagem. -->
                <PrtcolVrsn>2.0</PrtcolVrsn>
            </Hdr>
            <!-- Informações sobre a resposta da captura ou desfazimento de uma autorização. -->
            <CmpltnAdvcRspn>
                <!-- Ambiente da transação. -->
                <Envt>
                    <!-- Dados do estabelecimento. -->
                    <Mrchnt>
                        <!-- Identificação do estabelecimento comercial no adquirente.
                             Também conhecido internamente como “SaleAffiliationKey”. -->
                        <Id>BFDB58AB9A8A48828C2647E18B7F1114</Id>
                    </Mrchnt>
                </Envt>
                <!-- Dados da transação. -->
                <Tx>
                    <!-- Dados da identificação da transação definida pelo POI. -->
                    <TxId>
                        <!-- Data e hora da transação -->
                        <TxDtTm>2014-03-12T15:17:59</TxDtTm>
                        <!-- Identificação da transação definida pelo ponto de interação.
                             Este campo será ecoado pelo adquirente. -->
                        <TxRef>7ca686eb242b4c0482c58961f5d3aac7</TxRef>
                    </TxId>
                    <!-- Resultado da transação.
                         DECL = Declined,
                         APPR = Approved,
                         PART = Partial Approved,
                         TECH = Technical Error. -->
                    <Rspn>APPR</Rspn>
                </Tx>
            </CmpltnAdvcRspn>
        </AccptrCmpltnAdvcRspn>
    </Document>

### Cancelamento
O cancelamento é o serviço que permite que um estabelecimento cancele uma transação concluída com êxito. É por vezes chamado de “desfazimento manual”. O prazo para é 180 dias para o cancelamento.

#### XML de requisição de Cancelamento comentado
A mensagem `AcceptorCancellationRequest` é utilizada para realizar um pedido de cancelamento de uma transação autorizada. Se faz necessária uma mensagem de `AcceptorCancellationAdvice` para que o cancelamento seja confirmado caso o valor da tag `TransactionCapture<TxCaptr>` seja `false` .

    <Document xmlns="urn:AcceptorCancellationRequestV02.1">
        <AccptrCxlReq>
            <!-- Cabeçalho da mensagem -->
            <Hdr>
                <!-- Identifica o tipo de processo em que a mensagem se propõe.
                     CCAQ = Cancellation Request. -->
                <MsgFctn>CCAQ</MsgFctn>
                <!-- Versão do protocolo utilizado na mensagem. -->
                <PrtcolVrsn>2.0</PrtcolVrsn>
            </Hdr>
            <!-- Informações relacionadas à requisição de cancelamento. -->
            <CxlReq>
                <!-- Ambiente da transação. -->
                <Envt>
                    <!-- Dados do estabelecimento. -->
                    <Mrchnt>
                        <!-- Identificação do estabelecimento comercial. -->
                        <Id>
                            <!-- Identificação do estabelecimento comercial no adquirente.
                                 Também conhecido internamente como “SaleAffiliationKey”. -->
                            <Id>BFDB58AB9A8A48828C2647E18B7F1114</Id>
                        </Id>
                    </Mrchnt>
                    <!-- Dados do Ponto de Interação. -->
                    <POI>
                        <!-- Identificação do ponto de interação -->
                        <Id>
                            <!-- Código de identificação do ponto de interação atribuído
                                 pelo estabelecimento. -->
                            <Id>2FB4C89A</Id>
                        </Id>
                        <!-- Capacidades do Ponto de interação. -->
                        <Cpblties>
                            <!-- Número máximo de colunas de cada linha a ser impressa no cupom.
                                 A quantidade mínima de colunas é de 38.
                                 Se o POI enviar menos do que38, o Host Stone não irá retornar
                                 os dados do recibo. -->
                            <PrtLineWidth>50</PrtLineWidth>
                        </Cpblties>
                    </POI>
                </Envt>
                <!-- Dados da transação. -->
                <Tx>
                    <!-- Indica se os dados da transação devem ser capturados `true` ou não
                         `false` imediatamente. -->
                    <TxCaptr>true</TxCaptr>
                    <!-- Identificação da transação atribuída pelo POI. -->
                    <TxId>
                        <!-- Data e hora local da transação definidas pelo ponto de interação. -->
                        <TxDtTm>2014-03-12T15:09:00</TxDtTm>
                        <!-- Identificação da transação definida pelo ponto de interação.
                             O formato é livre contendo no máximo 32 caracteres. -->
                        <TxRef>12345ABC</TxRef>
                    </TxId>
                    <!-- Detalhes da transação -->
                    <TxDtls>
                        <!-- Moeda utilizada na transação em conformidade com a ISO 4217.-->
                        <Ccy>986</Ccy>
                        <!-- Valor total da transação em centavos. -->
                        <TtlAmt>100</TtlAmt>
                    </TxDtls>
                    <!-- Dados da transação original -->
                    <OrgnlTx>
                        <!-- Identificação da transação definida pelo sistema que se comunica
                             com o Host Stone. -->
                        <InitrTxId>123123123</InitrTxId>
                        <!-- Identificação da transação definida pelo adquirente. -->
                        <RcptTxId>00000034071000000215346</RcptTxId>
                    </OrgnlTx>
                </Tx>
            </CxlReq>
        </AccptrCxlReq>
    </Document>

#### XML de resposta de Cancelamento comentado
A mensagem AcceptorCancellationResponse é respondida pelo adquirente com as informações sobre a requisição de cancelamento `AcceptorCancellationRequest`. Importante informar que os dados do recibo de cancelamento são enviados somente nesta mensagem.

    <Document xmlns="urn:AcceptorCancellationResponseV02.1">
        <AccptrCxlRspn>
            <!-- Cabeçalho da mensagem -->
            <Hdr>
                <!-- Identifica o tipo de processo em que a mensagem se propõe.
                     CCAP = Cancellation Response. -->
                <MsgFctn>CCAP</MsgFctn>
                <!-- Versão do protocolo utilizado na mensagem. -->
                <PrtcolVrsn>2.0</PrtcolVrsn>
            </Hdr>
            <!-- Informações relacionadas à resposta de cancelamento. -->
            <CxlRspn>
                <!-- Ambiente da transação. -->
                <Envt>
                    <!-- Dados do estabelecimento. -->
                    <MrchntId>
                        <!-- Identificação do estabelecimento comercial no adquirente.
                             Também conhecido internamente como “SaleAffiliationKey”. -->
                        <Id>BFDB58AB9A8A48828C2647E18B7F1114</Id>
                    </MrchntId>
                </Envt>
                <!-- Dados da transação. -->
                <Tx>
                    <!-- Indica se os dados da transação devem ser capturados `true`
                         ou não `false` imediatamente. -->
                    <TxCaptr>true</TxCaptr>
                    <!-- Identificação da transação atribuída pelo POI. -->
                    <TxId>
                        <!-- Data e hora local da transação definidas pelo ponto de interação. -->
                        <TxDtTm>2014-03-12T15:09:00</TxDtTm>
                        <!-- Identificação da transação definida pelo ponto de interação.
                             Este campo será ecoado pelo adquirente. -->
                        <TxRef>123456798</TxRef>
                    </TxId>
                    <!-- Detalhes da transação -->
                    <TxDtls>
                        <!-- Moeda utilizada na transação em conformidade com a ISO 4217. -->
                        <Ccy>986</Ccy>
                        <!-- Valor total da transação em centavos. -->
                        <TtlAmt>100</TtlAmt>
                    </TxDtls>
                </Tx>
                <!-- Dados de resposta da transação. -->
                <TxRspn>
                    <!-- Informações sobre o resultado da autorização a ser cancelada. -->
                    <AuthstnRslt>
                        <!-- Dados de resposta da autorização a ser cancelada. -->
                        <RspnToAuthstn>
                            <!-- Resposta da transação.
                                 DECL: Declined
                                 APPR: Aproved
                                 PART: PartialApproved
                                 TECH: TechinicalError -->
                            <Rspn>APPR</Rspn>
                            <!-- Código de resposta da autorização
                                 equivalente ao campo 39 da ISO 8583 de 2003. -->
                            <RspnRsn>0000</RspnRsn>
                        </RspnToAuthstn>
                    </AuthstnRslt>
                    <!-- Informações sobre as ações a serem realizadas pelo POI.
                         Neste caso, as informações a serem impressas no comprovante. -->
                    <Actn>
                        <!-- Tipo da ação a ser realizada pelo POI.
                             PRNT = Imprimir mensagem;
                             DISP = Exibir mensagem. -->
                        <ActnTp>PRNT</ActnTp>
                        <!-- Mensagem a ser impressa no terminal. -->
                        <MsgToPres>
                            <!-- A quem se destina a mensagem no contexto.
                                 CRCP = CardholderReceipt;
                                 MRCP = MerchantReceipt. -->
                            <MsgDstn>CRCP</MsgDstn>
                            <!-- Texto ou dados gráficos a serem impressos na tela ou em papel
                                 no terminal. A mensagem será formatada conforme o campo
                                 PrintLineWidth da requisição. -->
                            <MsgCntt>Informacoes formatadas do recibo aqui</MsgCntt>
                        </MsgToPres>
                    </Actn>
                </TxRspn>
            </CxlRspn>
        </AccptrCxlRspn>
    </Document>

### Cancelamento com Captura

![cancelamento com captura](/assets/media/cancelamento-com-captura.png)

1. O estabelecimento envia uma `AcceptorAuthorisationRequest` para o adquirente para realizar um pedido de autorização
2. Uma `AcceptorAuthorisationResponse` é enviada pelo adquirente confirmando e aprovando o pedido de autorização
3. Uma `AcceptorCompletionAdvice` é utilizada para informar ao adquirente sobre a captura da transação
4. O adquirente envia uma `AcceptorCompletionAdviceResponse` reconhecendo o pedido de cancelamento por parte do estabelecimento;

Nesta transação o estabelecimento não necessita de nenhum pedido de cancelamento prévio pois supomos que ele já possui as informações de que o cancelamento pode realmente ser realizado.

5. Por sua vez, o estabelecimento envia uma `AcceptorCancellationAdvice` para informar ao adquirente que a transação autorizada previamente deve ser cancelada
6. O adquirente envia uma `AcceptorCancellationAdviceResponse` reconhecendo o cancelamento, e realiza os ajustes necessários.

### Cancelamento com Captura posterior

![cancelamento com captura posterior](/assets/media/cancelamento-com-captura-posterior.png)

1. O estabelecimento envia uma `AcceptorAuthorisationRequest` para o adquirente para realizar um pedido de autorização
2. Uma `AcceptorAuthorisationResponse` é enviada pelo adquirente confirmando e aprovando o pedido de autorização
3. Uma `AcceptorCompletionAdvice` é utilizada para informar ao adquirente sobre a captura da transação
4. O adquirente envia uma `AcceptorCompletionAdviceResponse` reconhecendo o pedido de captura da transação por parte do estabelecimento.

Neste momento, por algum motivo o estabelecimento precisa cancelar a transação. Para isso ele deve seguir o fluxo a seguir:

5. O estabelecimento envia uma `AcceptorCancellationRequest` para saber se o cancelamento pode ou não ser realizado. **O adquirente pode ou não permitir este cancelamento**. A tag `<TxCaptr>` deve possuir o valor `false`
6. O adquirente envia uma `AcceptorCancellationResponse` informando que o cancelamento pode ser realizado
7. Uma vez que o estabelecimento considere que o cancelamento deve ser efetivado, ele deve enviar uma mensagem `AcceptorCancellationAdvice` para que seja realizada a captura deste cancelamento
8. O adquirente envia `AcceptorCancellationAdviceResponse` efetivando o cancelamento.

## Códigos de Exemplo de integração

Os códigos de exemplo abaixo, ilustram a troca de mensagens, com requisição e resposta, para Autorização com Captura automática, Autoricação com Captura posterior, Captura, Reversão de transações e Cancelamento. Os códigos estão disponíveis no Github Stone e podem ser acessados através dos links externos abaixo.

[Códigos de exemplo em PHP](https://github.com/stone-pagamentos/code-sample-php)