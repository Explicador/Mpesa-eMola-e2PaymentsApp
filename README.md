# Mpesa-eMola-e2PaymentsApp
Aplicativo que ajuda na integração de **Mpesa** e **eMola** de forma simples usando a plataforma e2Payments.

Embora esse codigo foi exemplificado com base no Quasar ou JavaScript, poderá seguir este exemplo e fazer a implementação em qualquer linguagem de programação de sua preferência.

Esta é uma estrutura básica de um projecto baseado em Javascript, criado para facilitar os testes de integração de e2Payments para eMola e Mpesa.

Pode clonar e realizar testes como bem desejar.

## Requisitos

1. Git instalado na máquina: [instalar agora](https://git-scm.com/downloads)
2. Quasar CLI: [instalar agora](https://quasar.dev/start/quasar-cli)
3. Editor de texto á sua escolha: Sublime Text, VS Code, PHP Storm ou outro.
4. Ter uma conta no [account.explicador.co.mz](https://account.explicador.co.mz/)
5. Registar-se gratuitamente no [e2Payments](https://e2payments.explicador.co.mz/)
6. Criar pelo menos uma carteira [Mpesa](https://e2payments.explicador.co.mz/admin/wallets?w=MPesa) e [eMola](https://e2payments.explicador.co.mz/admin/wallets?w=eMola)
7. Criar as [credencias de acesso a API](https://e2payments.explicador.co.mz/admin/credentials)

## Clone e instalação:

Antes de fazer o clone, precisa certificar-se de que tem o quasar CLI instalado na sua máquina, pois caso não, poderá não finalizar com sucesso os passos abaixo!

Deseja instalar [Clique aqui para instalar QUASAR CLI](https://quasar.dev/start/quasar-cli)

Pode confirmar se o quasar está estalado, executando o comando:

```
$ quasar info
```

Se puder ver a versão de quasar, isso significa que o quasar está instalado na sua maquina!! Se der algum erro do comando, é sinal de que precisa instalar quasar para correr este projecto.

Assim, pode iniciar os passos abaixo:

### 01. Clone do repositório:

```
$ git clone https://github.com/Explicador/Mpesa-eMola-e2PaymentsApp.git
```

### 02. Instalação de dependências:

```
$ cd Mpesa-eMola-e2PaymentsApp && yarn install
```


### 03. Inicialização do projecto:

```
$ quasar dev
```

### 04. Chaves necessarias:

Precisa de navegar até a pasta `src\pages\Index.vue` e no método `async handlePayUsingEMola(payload) {}` alterar as chaves colocando as suas para que tudo funcione.

Se deseja usar `.env` para guardar as suas chaves, e só criar o ficheiro no directório raiz do projecto.

Se quiser sem `.env`, pode remover a referencia ao `process.env`, e colocar as strings das chaves normalmente seguindo o formato indicado.


### 05. Método que integra Mpesa e eMola

Execute este projecto para ver este método em funcionamento. Contudo, pode copiar e fazer as adptações necessarias em qualquer linguagem de programação de sua preferência. Só precisa de fazer as chamadas de API de forma corresponde na sua linguagem.

Este método resume a integração de Mpesa e eMola usando e2Payments.

Veja o quanto é simples:

```javascript
    async function handlePayUsingEMola(payload) {

      // Variaveis necessarios
      const credentials = {
        grant_type: 'client_credentials', //nao altere

        // Crie e copie as credenciais aqui: https://e2payments.explicador.co.mz/admin/credentials
        client_id: process.env.CLIENT_ID, // neste formato '99227735-dd04-4538-bf50-27eeed2bbbb'
        client_secret: process.env.CLIENT_SECRET, // neste formato 'EuLHLIyhm9mFEgZPBKrBdZHXKuyygfRHZWelscTx'

        // Use este link para criar e copiar o id para a sua carteira Mpesa: https://e2payments.explicador.co.mz/admin/wallets?w=MPesa
        //nao eh preciso enviar para requisitar token
        mpesa_wallet: process.env.MPESA_WALLET, //formato 123456

        // Use este link para criar e copiar o id para a sua carteira eMola: https://e2payments.explicador.co.mz/admin/wallets?w=eMola
        //nao eh preciso enviar para requisitar token
        emola_wallet: process.env.EMOLA_WALLET, //formato 123456
      }

      // Requisicao do token
      let token = await axios.post('https://e2payments.explicador.co.mz/oauth/token', credentials)
        .then(response => {
          return response.data.token_type + ' ' + response.data.access_token
        }).catch(error => {
          return null
        })

      // Composicao do fomulario. Inclui: client_id, phone, amount e reference.
      const formData = {
        client_id: credentials.client_id,
        ...payload
      }

      // Composicao do Header
      // Usar esta estrutura do Header para que a requisicao seja permitida pelo servidor de e2Payments
      const headers = {
        headers: {
          'Authorization': token,
          'Content-Type' : 'application/json',
          'Accept': 'application/json'
        }
      }

      // Endpoint para pagamento via Mpesa ou eMola
      const paymentEndpoint =
          (this.paymentOptSelected === 'Mpesa') ? 'https://e2payments.explicador.co.mz/v1/c2b/mpesa-payment/' + credentials.mpesa_wallet :
          this.paymentOptSelected === 'eMola' ? 'https://e2payments.explicador.co.mz/v1/c2b/emola-payment/' + credentials.emola_wallet :
          null // retorna-se null caso nao se tenha selecionado Mpesa ou eMola

      // Realizacao da transacao usando axios.post
      // Pode usar o fetch() se preferir.
      return axios.post(
        paymentEndpoint,
        formData,
        headers
      ).then(response => {

        const isPaymentSuccess = response.status === 200
        this.showPaymentResponse(isPaymentSuccess, 'Parabens, pagamento realizado com sucesso!')

      }).catch(error => {

        console.log('e2Payments Payment error: ', {error})

        if (error.response.data && error.response.data.message) {

          this.showPaymentResponse(false,  error.response.data.message)

        } else {

          this.showPaymentResponse(false, 'Pagamento nao realizado, ocorreu um erro.')

        }

        return false;
      })

    }

```

### 06. Suporte técnico:

No caso caso de dúvidas, não hesite em contactar-nos.

Meios de contactos:

* Nome: Explicador Inc
* Contacto: 842512502 / 862502502
* WhatsApp Chat: 842512502 / 862502502
* Email: info@explicador.co.mz / developers@explicador.co.mz
* Website: https://e2payments.explicador.co.mz/
* Comunidade WhatsApp: https://chat.whatsapp.com/H3yb0KLD3IO50e1lyDBtwd
