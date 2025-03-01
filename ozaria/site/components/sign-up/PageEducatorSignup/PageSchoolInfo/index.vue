<script>
  import { mapMutations, mapGetters } from 'vuex'
  import UnitedStatesSchoolForm from './UnitedStatesSchoolForm'
  import OtherCountriesSchoolForm from './OtherCountriesSchoolForm'
  import { COUNTRIES } from '../common/constants'
  import { getSchoolFormFieldsConfig } from '../common/signUpConfig'
  import { validationMixin } from 'vuelidate'
  import { educatorOtherInfoValidations, validationMessages } from '../common/signUpValidations'
  import SecondaryButton from '../../../teacher-dashboard/common/buttons/SecondaryButton'

  export default {
    metaInfo: {
      meta: [{ vmid: 'viewport', name: 'viewport', content: 'width=device-width, initial-scale=1' }]
    },
    components: {
      'united-states-school-form': UnitedStatesSchoolForm,
      'other-countries-school-form': OtherCountriesSchoolForm,
      SecondaryButton
    },

    mixins: [validationMixin],

    props: {
      creatingTeacherAccountLoad: {
        required: false,
        default: false,
        type: Boolean
      }
    },
    data: () => ({
      phoneNumber: '',
      numStudents: '',
      marketingConsent: !me.inEU(),
      gdprConsent: !me.inEU(),
      validationMessages,
      isChinaServerSignup: me.showChinaRegistration(),
      childFormValid: false
    }),

    validations () {
      return educatorOtherInfoValidations(this.country, this.role, this.isChinaServerSignup)
    },

    computed: {
      ...mapGetters({
        trialReqProps: 'teacherSignup/getTrialRequestProperties'
      }),

      country () {
        return this.trialReqProps.country
      },

      role () {
        return this.trialReqProps.role
      },

      formFieldConfig () {
        return getSchoolFormFieldsConfig(this.country, this.role, this.isChinaServerSignup)
      },

      isUS () {
        return this.trialReqProps.country === COUNTRIES.US
      },

      isEU () {
        return me.inEU()
      },

      isFormValid () {
        return !this.$v.$invalid && this.childFormValid
      },

      doneDisabled () {
        return !this.isFormValid || !this.gdprConsent
      }
    },

    watch: {
      isFormValid (val) {
        this.$emit('validityChange', val)
      }
    },

    mounted () {
      // default country code
      if (this.isChinaServerSignup) {
        this.phoneNumber = '+86 '
      } else {
        this.phoneNumber = '+1 '
      }
    },

    methods: {
      ...mapMutations({
        updateTrialRequestProperties: 'teacherSignup/updateTrialRequestProperties',
        setMarketingConsent: 'teacherSignup/setMarketingConsent'
      }),

      onChangeValue (event = {}) {
        const attrs = {}
        if (event.target) {
          attrs[event.target.name] = event.target.value
        }
        this.updateTrialRequestProperties(attrs)
      },

      onClickNext () {
        if (this.isFormValid) {
          this.setMarketingConsent({ marketingConsent: this.marketingConsent })
          this.$emit('goToNext')
        }
      }
    }
  }
</script>

<template lang="pug">
  #school-info-component
    form.form-container(@submit.prevent="onClickNext")
      united-states-school-form(v-if="isUS" @validityChange="(val) => this.childFormValid = val")
      other-countries-school-form(v-else @validityChange="(val) => this.childFormValid = val")
      .phoneNumber.form-group.row(v-if="formFieldConfig.phoneNumber.visible" :class="{ 'has-error': $v.phoneNumber.$error }")
        .col-sm-10.col-xs-12
          span.inline-flex-form-label-div
            span.control-label {{ $t("teachers_quote.phone_number") }}
              span.control-label.optional-text(v-if="!formFieldConfig.phoneNumber.required") !{' '}({{ $t("signup.optional") }})
            span.form-error(v-if="!$v.phoneNumber.required") {{ $t(validationMessages.errorRequired.i18n) }}
          input.phone-input.form-control(name="phoneNumber" v-model="$v.phoneNumber.$model" @change="onChangeValue($event)")
      .numStudents.form-group.row(v-if="formFieldConfig.numStudents.visible" :class="{ 'has-error': $v.numStudents.$error }")
        .col-sm-10.col-xs-12
          span.inline-flex-form-label-div
            span.control-label {{ $t("teachers_quote.num_students_help") }}
              span.control-label.optional-text(v-if="!formFieldConfig.numStudents.required") !{' '}({{ $t("signup.optional") }})
            span.form-error(v-if="!$v.numStudents.required") {{ $t(validationMessages.errorRequired.i18n) }}
          select#numStudents-input.form-control(name="numStudents", v-model="$v.numStudents.$model" @change="onChangeValue($event)" :class="{ 'placeholder-text': !numStudents }")
            option(disabled selected value='') {{ $t("teachers_quote.num_students_default") }}
            option 1-10
            option 11-50
            option 51-100
            option 101-200
            option 201-500
            option 501-1000
            option 1000+
      .marketingConsent.form-group.row
        .col-sm-10.col-xs-12.form-checkbox-input
          input#marketingConsent(name="marketingConsent", type="checkbox", v-model="marketingConsent")
          label(for="marketingConsent")
            span {{ $t("signup.teacher_email_announcements") }}
      .gdprConsent.form-group.row(v-if="isEU")
        .col-sm-10.col-xs-12.form-checkbox-input
          input#gdprConsent(name="gdprConsent", type="checkbox", v-model="gdprConsent")
          label(for="gdprConsent")
            span {{ $t("signup.eu_confirmation") }}!{' '}
              a(href="https://www.ozaria.com/privacy#gdpr" target="_blank") {{ $t("signup.eu_confirmation_place_of_processing") }}
      .buttons.form-group.row
        .col-xs-offset-7
          secondary-button(v-if="!creatingTeacherAccountLoad" type="submit", :inactive="doneDisabled") {{ $t("common.done") }}
          secondary-button(v-else type="submit", :inactive="true" disabled) {{ $t("common.loading") }}
</template>

<style lang="sass" scoped>
#school-info-component
  height: 100vh
  display: flex
  flex-flow: column
  justify-content: center
  .buttons
    margin-top: 30px
    button
      width: 150px
      height: 35px
</style>
