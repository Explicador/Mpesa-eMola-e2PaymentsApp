<template>
  <q-page class="q-pa-xl">
    <div class="row">

      <div class="col-md-8 col-12">
        <div class="text-grey">
          Selecione o metodo de pagamento e preencha os campos.
        </div>

        <!-- Area com as opcoes de pagamentos -->
        <div class="text-center q-pa-md q-gutter-x-md">
          <q-btn
            @click="paymentOptSelected = 'Mpesa'"
            style="width: 100px;height: 100px;padding: 0;margin: 10px"
            flat
          >
            <img src="https://e2payments.explicador.co.mz/assets/images/mpesa.png"
             alt=""
             style="width: 100%;height: 100%"
            >
            <q-badge v-if="paymentOptSelected === 'Mpesa'" color="green" floating>
              <q-icon name="check" color="white" />
            </q-badge>
          </q-btn>

          <q-btn
            @click="paymentOptSelected = 'eMola'"
            style="width: 100px;height: 100px;padding: 0;margin: 10px"
            flat
          >
            <img src="https://e2payments.explicador.co.mz/assets/images/emola.png"
             alt=""
             style="width: 100%;height: 100%"
            >
            <q-badge v-if="paymentOptSelected === 'eMola'" color="green" floating>
              <q-icon name="check" color="white" />
            </q-badge>
          </q-btn>

        </div>
        <!-- ./Area com as opcoes de pagamentos -->

        <!--Formularios com os campos -->
        <q-form
          @submit="onSubmit"
          class="q-gutter-md"
        >
          <q-input
            filled
            v-model="payload.amount"
            label="Insira o valor a pagar *"
            type="number"
            lazy-rules
            :rules="[ val => val && val.length > 0 || 'Por favor, preencha este campo']"
          />

          <q-input
            v-if="paymentOptSelected"
            filled
            type="number"
            v-model="payload.phone"
            :label="'Número com ' + (paymentOptSelected === 'Mpesa' ? 'Mpesa 84/85' : 'eMola 86/87')"
            lazy-rules
            :rules="[
              val => val !== null && val !== '' || 'Por favor, preencha este campos',
              val => val > 0 && val.toString().length === 9 || 'O número deve ter 9 digitos',
              val => isValidPhoneNumber || 'Insira um número ' + paymentOptSelected + ' valido'
            ]"
          />

          <q-select
            filled
            v-model="payload.reference"
            :options="referenceOptions"
            label="Indique a referencia"
            lazy-rules
            :rules="[
              val => val !== null && val !== 'Selecione' || 'Por favor, selecione alguma opcao',
            ]"
          />

          <div class="text-right">
            <q-btn
              :disable="!formFieldsAreValids()"
              label="Submit"
              type="submit"
              :color="formFieldsAreValids() ? 'primary' : 'grey-2'"
              :unelevated="!formFieldsAreValids()"
              style="width: 180px;height: 40px"
              :loading="isLoading"
            />
          </div>
        </q-form>
        <!--./Formularios com os campos -->

      </div>
    </div>
  </q-page>
</template>

<script>

import axios from 'axios'

export default {
  name: 'Index',
  data () {
    return {
      payload: {
        amount: null,
        phone: null,
        reference: null,
      },
      referenceOptions: [
        'Selecione', 'Live', 'TestarPagamento', 'PagarCurso', 'DoarValor'
      ],
      paymentOptSelected: null,
      isLoading: false
    }
  },
  computed: {
    isValidPhoneNumber() {

      if (this.paymentOptSelected === null) return false

      return this.paymentOptSelected === 'Mpesa'
        ? this.payload.phone.startsWith('84') || this.payload.phone.startsWith('85')
        : this.payload.phone.startsWith('86') || this.payload.phone.startsWith('87')

    },
  },
  methods: {

    formFieldsAreValids() {

      return this.paymentOptSelected
        && this.payload.amount
        && this.payload.phone
        && this.payload.reference

    },

    onSubmit () {

      this.isLoading = true

      this.handlePayUsingEMola(this.payload)
      // Chamamos o metodo que processa o pagamento

    },

    async handlePayUsingEMola(payload) {

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

    },

    showPaymentResponse (isSuccess, feedbackMessage = '') {

      this.isLoading = false

      this.$q.notify({
        color: isSuccess ? 'positive' : 'danger',
        textColor: 'white',
        icon: 'check_circle',
        timeout: 15000,
        progress: true,
        actions: [
          { icon: 'close', color: 'dark', handler: () => { /* ... */ } }
        ],
        message: feedbackMessage
      })

    }

  },
  mounted() {

  }
}
</script>
