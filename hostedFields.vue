<template>
  <div :class="['HostedFields', wrapperClass]">

    <div v-show="!hostedFieldsInstance" class="HostedFields__loader">
      <i class="text-primary fa fa-cog fa-spin fa-3x"></i>
    </div>

    <div v-show="hostedFieldsInstance" class="HostedFields__form">
      <div class="row">
        <div v-if="collectCardHolderName" class="HostedFields__field HostedFields__field--cardHolder form-group col-sm-12">
          <label class="control-label">{{ cardHolderLabel }}</label>
          <input type="text" class="form-control" id="card-holder" name="cardholder" :placeholder="cardHolderPlaceholder">
        </div>
        <div class="HostedFields__field HostedFields__field--cardNumber form-group col-sm-12">
          <label class="control-label">{{ cardNumberLabel }}</label>
          <div class="form-control" id="card-number"></div>
        </div>
      </div>
      <div class="row">
        <div class="HostedFields__field HostedFields__field--expiry form-group col-xs-6">
          <label class="control-label">{{ cardExpiryLabel }}</label>
          <div class="form-control" id="card-expiry"></div>
        </div>
        <div class="HostedFields__field HostedFields__field--cvv form-group col-xs-6">
          <label class="control-label">{{ cardCvvLabel }}</label>
          <div class="form-control" id="card-cvv"></div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  export default {
    props: {
      cardHolderLabel: {
        default: 'Card holder',
        type: String
      },
      cardNumberLabel: {
        default: 'Card number',
        type: String
      },
      cardExpiryLabel: {
        default: 'Expiry',
        type: String
      },
      cardCvvLabel: {
        default: 'CVV',
        type: String
      },
      cardHolderPlaceholder: {
        default: '',
        type: String
      },
      cardNumberPlaceholder: {
        default: '',
        type: String
      },
      cardExpiryPlaceholder: {
        default: '',
        type: String
      },
      cardCvvPlaceholder: {
        default: '',
        type: String
      },
      authToken: {
        value: String,
      },
      wrapperClass: {
        value: String,
      },
      collectCardHolderName: {
        value: Boolean,
        default: false
      }
    },
    created() {
      this.createBT();

      this.$parent.$on('tokenize', () => {
        this.tokenizeHF();
      });
    },
    data () {
      return {
        errorMessage: '',
        clientInstance: '',
        hostedFieldsInstance: '',
        tokenizePayload: '',
        dataCollectorPayload: '',
      }
    },
    methods: {
      createBT () {
        const client = require('braintree-web/client');
        client.create({
          authorization: this.authToken
        }, (clientErr, clientInstance) => {
          if (clientErr) {
            this.errorMessage = 'There was an error setting up the client instance. Message: ' + clientErr.message;
            this.$emit('bthferror', this.errorMessage);
            return;
          } else {
            this.clientInstance = clientInstance
            this.createHF();
          }
        });
      },
      createHF () {
        const hostedFields = require('braintree-web/hosted-fields');
        hostedFields.create({
          client: this.clientInstance,
          styles: {
            'input': {
              'font-size': '14px'
            },
            'input.invalid': {
              'color': '#D90000'
            },
            'input.valid': {
              'color': '#51A351'
            }
          },
          fields: {
            number: {
              selector: '#card-number',
              placeholder: this.cardHolderPlaceholder,
            },
            cvv: {
              selector: '#card-cvv',
              placeholder: this.cardCvvPlaceholder,
            },
            expirationDate: {
              selector: '#card-expiry',
              placeholder: this.cardExpiryPlaceholder,
            },
          },
        }, (hostedFieldsErr, hostedFieldsInstance) => {
          if (hostedFieldsErr) {
            // Handle error in Hosted Fields creation
            this.errorMessage = 'There was an error setting up the hosted fields! Message: ' + hostedFieldsErr.message;
            this.$emit('bthferror', this.errorMessage);
            return;
          } else {
            this.$emit('bthfready');
            this.hostedFieldsInstance = hostedFieldsInstance;
          }

        });
      },
      tokenizeHF () {
        const additionalFields = {
          cardholderName: ''
        };
        if (this.collectCardHolderName) {
          additionalFields.cardholderName = document.querySelector('#card-holder').value;
        }

        this.hostedFieldsInstance.tokenize(additionalFields, (tokenizeErr, payload) => {
          if (tokenizeErr) {
            this.errorMessage = 'There was an error tokenizing! Message: ' + tokenizeErr.message;
            this.$emit('bthferror', this.errorMessage);
            return;
          }

          this.tokenizePayload = payload;
          this.$emit('bthfpayload', payload);
          this.teardownHF();

        });
      },
      teardownHF () {
        this.hostedFieldsInstance.teardown( (teardownErr) => {
            if (teardownErr) {
              this.errorMessage = 'There was an error tearing it down! Message: ' + teardownErr.message;
              this.$emit('bthferror', this.errorMessage);
              return;
            } else {
              this.hostedFieldsInstance = '';
              return;
            }
        });
      }
    },
  };
</script>

<style>
  .HostedFields__loader {
    width: 100%;
    text-align: center;
    height: 50px;
  }

  /* LOADER 1 */

  #loader-1:before, #loader-1:after{
    content: "";
    position: absolute;
    top: -10px;
    left: -10px;
    width: 100%;
    height: 100%;
    border-radius: 100%;
    border: 10px solid transparent;
    border-top-color: #3498db;
  }

  #loader-1:before{
    z-index: 100;
    animation: spin 1s infinite;
  }

  #loader-1:after{
    border: 10px solid #ccc;
  }

  @keyframes spin{
    0%{
      -webkit-transform: rotate(0deg);
      -ms-transform: rotate(0deg);
      -o-transform: rotate(0deg);
      transform: rotate(0deg);
    }

    100%{
      -webkit-transform: rotate(360deg);
      -ms-transform: rotate(360deg);
      -o-transform: rotate(360deg);
      transform: rotate(360deg);
    }
  }
</style>
