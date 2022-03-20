<template>
  <div class="form-container">
    <b-alert variant="success" :show="isPaymentSuccess">
      Payment Successful !!</b-alert
    >
    <b-form @submit.prevent="onSubmit" name="creditCardForm">
      <div class="left-container">
        <b-form-group
          label="CARD NUMBER"
          id="cardNumber"
          :invalid-feedback="errors.cardNumber"
          :state="getValidityState(errors.cardNumber)"
        >
          <b-row>
            <b-col v-for="(item, index) in card.cardNumber" :key="index">
              <b-form-input
                :class="`inputs-${index}`"
                type="text"
                maxlength="4"
                :id="`${index}`"
                v-model="card.cardNumber[index]"
                v-numericOnly
                v-autoTab
                required
                placeholder="0000"
                :state="getValidityState(errors.cardNumber)"
              >
              </b-form-input>
            </b-col>
          </b-row>
        </b-form-group>
        <b-row>
          <b-col cols="4">
            <b-form-group
              class="card-firstname"
              label="FIRST NAME"
              :invalid-feedback="errors.cardName"
              :state="getValidityState(errors.cardName)"
            >
              <b-form-input
                type="text"
                maxlength="20"
                v-autoTab
                v-alphabetsOnly
                v-model="card.name.firstName"
                required
                :state="getValidityState(errors.cardName)"
              >
              </b-form-input>
            </b-form-group>
          </b-col>
          <b-col cols="6">
            <b-form-group class="card-lastname" label="LAST NAME">
              <b-form-input
                type="text"
                maxlength="20"
                v-autoTab
                v-alphabetsOnly
                v-model="card.name.lastName"
                required
                :state="getValidityState(errors.cardName)"
              >
              </b-form-input>
            </b-form-group>
          </b-col>
          <b-col> </b-col>
        </b-row>
        <b-row>
          <b-col>
            <b-form-group
              label="MONTH"
              :invalid-feedback="errors.expiryDate"
              :state="getValidityState(errors.expiryDate)"
            >
              <b-form-input
                type="text"
                id="expiryMonth"
                v-numericOnly
                maxlength="2"
                v-model="card.expiry.month"
                v-autoTab
                required
                :state="getValidityState(errors.expiryDate)"
                placeholder="00"
              >
              </b-form-input>
            </b-form-group>
          </b-col>
          <b-col>
            <b-form-group
              label="YEAR"
              :state="getValidityState(errors.expiryDate)"
            >
              <b-form-input
                type="text"
                id="expiryYear"
                v-numericOnly
                maxlength="4"
                v-model="card.expiry.year"
                v-autoTab
                required
                :state="getValidityState(errors.expiryDate)"
                placeholder="0000"
              >
              </b-form-input>
            </b-form-group>
          </b-col>
          <b-col> </b-col>
          <b-col>
            <b-form-group
              label="CVC"
              :invalid-feedback="errors.cardCVC"
              :state="getValidityState(errors.cardCVC)"
            >
              <b-form-input
                type="text"
                id="cvc"
                v-numericOnly
                maxlength="3"
                v-model="card.cvc"
                v-autoTab
                required
                :state="getValidityState(errors.cardCVC)"
                placeholder="000"
              >
              </b-form-input>
            </b-form-group>
          </b-col>
        </b-row>
      </div>
      <div class="right-container">
        <div>
          <img :src="getImgUrl(cardType)" alt="card type" />
        </div>
      </div>
      <div class="card-footer">
        <b-row>
          <b-col>
            <div class="total-amount-info">
              <span>TOTAL: </span><span class="amount">{{ card.amount }}</span>
            </div>
          </b-col>
          <b-col cols="4"></b-col>
          <b-col>
            <b-button
              class="button-paynow"
              :disabled="isPaymentSuccess"
              type="submit"
              variant="primary"
              >PAY NOW</b-button
            >
          </b-col>
        </b-row>
      </div>
    </b-form>
  </div>
</template>

<script>
import Payment from "payment/lib";
import autoTab from "@/helpers/autotab.js";

export default {
  name: "CardThings",
  props: {
    cardInfo: {
      type: Object,
      default: function () {
        return {
          amount: "",
          name: {
            firstName: "",
            lastName: "",
          },
          cardNumber: ["", "", "", ""],
          expiry: {
            month: "",
            year: "",
          },
          cvc: "",
        };
      },
    },
  },
  data() {
    return {
      card: this.cardInfo,
      cardTypes: ["mastercard", "visa"],
      isError: false,
      errors: {
        cardNumber: "",
        expiryDate: "",
        cardCVC: "",
        cardName: "",
      },
      isPaymentSuccess: false,
    };
  },
  created() {
    // set amount value once card instance is created
    this.card.amount = "100$";
  },
  methods: {
    /**
     * Display card type icon on the screen
     * @param {String} type - type of the card
     */
    getImgUrl(type) {
      var images = require.context("../assets/", false, /\.png$/);
      return images("./" + type + ".png");
    },
    /**
     * if the errorString value exists, then input state is invalid
     * @param {String} errorString - error string
     */
    getValidityState(errorString) {
      return errorString.length > 0 ? false : null;
    },
    /**
     * Submit the credit card form
     */
    onSubmit() {
      // validate provided input values
      this.validateForm(this.card);
      // if no errors, display `success` message and
      // disable the 'pay now' button.
      if (!this.isError) {
        this.isPaymentSuccess = true;
      } else {
        this.isPaymentSuccess = false;
      }
    },
    /**
     * Validates user provided values using a library called 'payment'
     * https://www.npmjs.com/package/payment
     * @param {Object} card - card object containing input values
     */
    validateForm(card) {
      const isValidNumber = Payment.fns.validateCardNumber(
        card.cardNumber.join("")
      );
      const isValidCVC = Payment.fns.validateCardCVC(card.cvc);
      const isValidExpiry = Payment.fns.validateCardExpiry(
        card.expiry.month,
        card.expiry.year
      );
      const fullName = card.name.firstName + card.name.lastName;
      const isValidName =
        fullName.length > 0 && /^[a-zA-Z]*$/.test(fullName) ? true : false;

      this.handleErrors(isValidNumber, isValidCVC, isValidExpiry, isValidName);
    },
    /**
     * Generate error messages based on provided input values
     * @param {Boolean} isValidNumber - validity result for card number
     * @param {Boolean} isValidCVC - validity result for card cvc
     * @param {Boolean} isValidExpiry - validity result for card expiration
     */
    handleErrors(isValidNumber, isValidCVC, isValidExpiry, isValidName) {
      this.errors = {
        cardNumber: "",
        expiryDate: "",
        cardCVC: "",
        cardName: "",
      };
      this.isError = false;
      if (!isValidNumber) {
        this.errors.cardNumber = "Please correct your card number";
        this.isError = true;
      }
      if (!isValidCVC) {
        this.errors.cardCVC = "Please enter a valid CVC number";
        this.isError = true;
      }
      if (!isValidExpiry) {
        this.errors.expiryDate = "Please enter a valid month and date";
        this.isError = true;
      }
      if (!isValidName) {
        this.errors.cardName = "Please enter a valid firstname and lastname";
        this.isError = true;
      }
    },
  },
  directives: {
    // allows only numeric values in an input field
    numericOnly: {
      bind(el) {
        el.addEventListener("keyup", () => {
          let regex = /^[0-9]*$/;
          if (!regex.test(el.value)) {
            el.value = el.value.replace(/\D/g, "");
          }
        });
      },
    },
    // allow only alphbet values in an input field
    alphabetsOnly: {
      bind(el) {
        el.addEventListener("keyup", () => {
          let regex = /^[a-zA-Z]*$/;
          if (!regex.test(el.value)) {
            el.value = el.value.replace(/[^A-Za-z]/g, "");
          }
        });
      },
    },
    // enable auto tabs between input fields based on maxLength value
    autoTab: {
      bind(el) {
        el.addEventListener("keyup", () => {
          if (el.value.length === el.maxLength) {
            autoTab.focusNextElement();
          }
        });
      },
    },
  },
  computed: {
    // TODO: currently defaults to 'mastercard' if card type can not
    // be identified, which needs to be revisited and fixed to display
    // correct card type (NOTE)

    // card type get computed based on card number value
    // default value is 'mastercard'
    // https://www.npmjs.com/package/payment#paymentfnscardtypenumber
    cardType: function () {
      const isValidCardType = this.cardTypes.includes(
        Payment.fns.cardType(this.card.cardNumber.join(""))
      );
      if (isValidCardType) {
        return Payment.fns.cardType(this.card.cardNumber.join(""));
      } else {
        return "mastercard";
      }
    },
  },
};
</script>
<style src="./Card.scss" lang="scss" scoped></style>
