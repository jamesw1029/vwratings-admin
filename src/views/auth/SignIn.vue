<template>
  <div class="auth-page">
    <div class="auth-page__signin" @keyup.enter="submitLogin()" v-if="this.signin_form">
      <b-form-group class="m-0 mb-1">
        <b-form-input
          size="lg"
          class="mb-2"
          placeholder="Username"
          type="username"
          :state="validateState('username')"
          v-model="form.username"
          autofocus
        />
      </b-form-group>

      <b-form-group class="password-group">
        <b-form-input
          size="lg"
          class="mb-2"
          placeholder="Password"
          :type="this.password_reveal ? 'text' : 'password'"
          :state="validateState('password')"
          v-model="form.password"
        />
        <div class="password-reveal" @click="passwordReveal()">
          <b-icon
            variant="primary"
            :icon="this.password_reveal ? 'eye-slash' : 'eye'"
          />
        </div>
      </b-form-group>

      <b-button size="large" variant="primary" class="w-100" type="submit" @click="submitLogin()">Login</b-button>
      <a class="text-center mt-3 d-block cursor-pointer" @click="signin_form = false">Forgot password?</a>
      <span class="error-message text-center text-danger d-block">{{ this.error }}</span>

    </div>
    <div class="auth-page__forgot" v-else>

      <div v-if="!forget_sent">
        <h2 class="font-weight-bold">Forgotten Password ?</h2>

        <span class="d-block mb-5 text-black-50">Enter your email to reset your password</span>

        <b-form-input
          class="pt-4 pb-4"
          placeholder="Email"
          type="email"
          :state="validateState('email')"
          v-model="form.email"
          @keyup.enter="submitForgetPassword()"
          autofocus
        />

        <div class="mt-3">
          <b-button variant="primary" class="mr-2" type="submit" @click="submitForgetPassword()">Send</b-button>
          <b-button @click="signin_form = true">Back</b-button>
        </div>

      </div>

      <div class="auth-page__sent mb-3" v-if="forget_sent">
        <h2 class="font-weight-bold">Check your mail</h2>
        <p>We sent the instructions to: <br> {{ this.form.email }}</p>
        <b-button @click="signin_form = true">Back</b-button>
      </div>
    </div>
  </div>
</template>
<script>
const { required } = require('vuelidate/lib/validators')
import { mapActions } from 'vuex';

export default {
  data() {
    return {
      forget_sent: false,
      signin_form: true,
      password_reveal: false,
      error: null,
      form: {
        username: '',
        password: ''
      }
    }
  },

  validations: {
    form: {
      username: {required},
      password: {required}
    }
  },

  computed: {
    appName() {
      return process.env.VUE_APP_PROJECT_NAME
    }
  },

  methods: {
    ...mapActions({
      login: 'auth/LOGIN'
    }),

    passwordReveal() {
      this.password_reveal = !this.password_reveal;
    },

    validateState(name) {
      const {$dirty, $error} = this.$v.form[name];
      return $dirty ? !$error : null;
    },

    submitForgetPassword() {
      this.$v.form.email.$touch();
      if (this.$v.form.email.$error) {
        return;
      }

      this.$api.auth.forgotPass({email: this.form.email}).then(() => {
        this.forget_sent = true;
      })
    },

    submitLogin() {
      this.$v.form.$touch();
      if (this.$v.form.$anyError) {
        return;
      }

      const formData = new FormData();
      formData.append('username', this.form.username);
      formData.append('password', this.form.password);

      this.login(formData).then(() => {
        this.$router.push({name: 'cabinet.dashboard'})
      }).catch(() => {
        this.error = 'Invalid email or password'
      })
    }
  }
}
</script>
<style lang="scss">
.auth-page {
  width: 350px;

  .password-group {
    position: relative;

    .is-valid {
      background-image: unset;
    }
  }

  .password-reveal {
    right: 0;
    top: 0;
    display: flex;
    align-items: center;
    justify-content: center;
    width: 47px;
    height: 100%;
    cursor: pointer;
    position: absolute;

    svg {
      width: 20px;
      height: 20px;
    }
  }

  .error-message {
    height: 30px;
    margin-top: 30px;
  }

  .cursor-pointer {
    cursor: pointer;
  }
}
</style>