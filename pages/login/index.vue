<template>
  <div class="container">
    <div class="row">
      <div class="col-md-6 col-md-offset-4 mx-auto">
        <div class="box">
          <div class="col-12">
            <div class="row mt-2">
              <a
                class="btn btn-lg btn-block btn-twitter"
                href="#"
                @click="loginGoogle"
              >
                <span class="addon-twitter">
                  <i class="fab fa-fw fa-2x fa-google fa-fw"></i>
                </span>
                Login with Google</a
              >
            </div>
          </div>

          <ValidationObserver ref="form" v-slot="{ passes, errors }">
            <form id="login-form" role="form" @submit.prevent="onLoginSubmit">
              <div class="divider-form"></div>

              <b-form-group
                id="Username-group"
                label="Username:"
                label-for="Username"
              >
                <ValidationProvider
                  vid="Username"
                  name="Username"
                  v-slot="validationContext"
                  rules="required"
                >
                  <b-form-input
                    id="Username"
                    :state="getValidationState(validationContext)"
                    placeholder="Enter username"
                    v-model="form.username"
                    type="Username"
                  >
                  </b-form-input>
                  <b-form-invalid-feedback>
                    <div v-for="error in validationContext.errors">
                      {{ error }}
                    </div>
                  </b-form-invalid-feedback>
                </ValidationProvider>
              </b-form-group>

              <b-form-group
                id="password-group"
                label="Password:"
                label-for="password"
              >
                <ValidationProvider
                  vid="password"
                  name="password"
                  v-slot="validationContext"
                  rules="required"
                >
                  <b-form-input
                    id="password"
                    :state="getValidationState(validationContext)"
                    placeholder="Enter password"
                    v-model="form.password"
                    type="password"
                  >
                  </b-form-input>
                  <b-form-invalid-feedback>
                    <div v-for="error in validationContext.errors">
                      {{ error }}
                    </div>
                  </b-form-invalid-feedback>
                </ValidationProvider>
              </b-form-group>

              <div class="divider-form"></div>
              <ValidationProvider
                vid="error"
                name="error"
                v-slot="validationContext"
              >
                <div
                  class="text-center text-danger my-2"
                  v-if="validationContext.errors[1]"
                >
                  {{ validationContext.errors }}
                </div>
              </ValidationProvider>

              <p class="text-center">
                You agree to the
                <strong>
                  <nuxt-link to="tos">Terms & Conditions</nuxt-link> </strong
                >.
              </p>

              <button type="submit" class="btn-block btn btn-lg btn-primary">
                Login
              </button>

              <p class="text-center mt-2">
                Forgot your password?
                <nuxt-link to="/login/reset_password">Reset</nuxt-link>
              </p>

              <p class="text-center">
                Need an account? <nuxt-link to="/signup">Signup</nuxt-link>
              </p>
            </form>
          </ValidationObserver>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import Vue from "vue";
import { mapMutations } from "vuex";
import { ValidationProvider, ValidationObserver } from "vee-validate";

export default {
  name: "login",
  layout: "default",
  components: {
    ValidationProvider,
    ValidationObserver
  },
  async fetch({ store, params }) {},
  data() {
    return {
      form: {}
    };
  },
  mounted() {
    if (this.$store.state.message !== null) {
      this.$bvToast.toast(this.$store.state.message, {
        title: "Heads Up!",
        autoHideDelay: 2500,
        appendToast: true
      });
      this.$store.commit("CLEAR_MESSAGE");
    }
  },
  beforeDestroy() {},
  created() {},
  updated() {},
  computed: {},
  methods: {
    getValidationState({ dirty, validated, valid = null }) {
      return dirty || validated ? valid : null;
    },
    /* Use vuex to log the user in */
    async onLoginSubmit() {
      this.$store.dispatch("login", this.form).then(status => {
        this.$router.push("/");
      }).catch(err => console.log(err))
    },
    /* Log the user in using  */
    loginGoogle() {
      console.log("Google");
      this.$gAuth.signIn().then(GoogleUser => {
        console.log("user", GoogleUser);
        // Convert the token
        this.$axios
          .$post("/api/oath/convert-token", {
            grant_type: "convert_token",
            client_id: "zBMCR5Dvoz1G0Vz7TP8rucXks1nMd7ynpS8KuqC1",
            client_secret:
              "0oin5jEiAAn8omVfrrOIz2UVD6IsYnuAkmKLLU6LMAHGF4GnHLyA0UnfvUdrlBP3jqvzUc8qm2f6Q2eUo65tEGzfHtKK0uxRWJvXJVvFB3rnJPm6PlVRr9mmJOgBQJQp",
            backend: "google-oauth2",
            token: GoogleUser.uc.access_token
          })
          .then(res => {
            // this.$axios.setHeader('Authorization', data.token_type + ' ' + data.access_token)

            this.$axios.setToken(res.access_token, res.token_type);
            this.$store.dispatch("loginGoogle", res);
            let nextUrl = "/";
            if (this.$route.query.next) {
              nextUrl = decodeURIComponent(this.$route.query.next);
            }
            if (!nextUrl.startsWith("/")) {
              nextUrl = "/";
            }
            this.$router.push(nextUrl);
          });
      });

      // this.$gAuth.getAuthCode().then(async authCode => {
      //   console.log(authCode)
      //   this.$store.dispatch('loginGoogle', authCode)
      // await this.$axios
      //   .$post("/auth/login/google", { auth: authCode })
      //   .then(async resp => {
      //     const user_details = await this.$axios.$get("/api/auth/users/me/");
      //     await this.$store.dispatch("login", user_details);
      //     let nextUrl = "/";
      //     if (this.$route.query.next) {
      //       nextUrl = decodeURIComponent(this.$route.query.next);
      //     }
      //     if (!nextUrl.startsWith("/")) {
      //       nextUrl = "/";
      //     }
      //     this.$router.push(nextUrl);
      //   })
      //   .catch(data => {
      //     this.$refs.form.setErrors(data.response.data);
      //   });
      // });
    }
  }
};
</script>

<style lang="scss">
.btn .fa-icon {
  vertical-align: middle;
  margin-right: 0.5rem;
}

.btn .fa-icon:last-child {
  margin-right: 0;
}

.box {
  background-color: #fff;
  padding: 25px 40px;
  margin-top: 30px;
  /*Remove, its example*/
  box-shadow: 0 8px 17px 0 rgba(0, 0, 0, 0.2), 0 6px 20px 0 rgba(0, 0, 0, 0.19);
}

.input-group {
  margin: 5px 0px;
}

.addon-facebook {
  vertical-align: middle;
  border: none;
  border-radius: 2px 0px 0px 2px;
  color: #fff;
}

.btn-facebook,
.btn-facebook:hover {
  background-color: #4b6ea9;
  color: #fff;
  border-radius: 0px 2px 2px 0px;
  font-size: 15px;
}

.addon-twitter {
  vertical-align: middle;
  border: none;
  border-radius: 2px 0px 0px 2px;
  color: #fff;
}

.btn-twitter,
.btn-twitter:hover {
  background-color: #00d7fa;
  color: #fff;
  border-radius: 0px;
  font-size: 15px;
}

p {
  color: #aebbcb;
}

a {
  color: #23a9f6;
}

.divider-form {
  border: 1px solid #ebeff1;
  margin: 15px -40px 10px;
}

label {
  text-transform: uppercase;
  color: #656565;
}

strong {
  color: #95a5bb;
}

.form-control:focus {
  box-shadow: none;
}
</style>
