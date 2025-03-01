<!--
  - coding=utf-8
  - Copyright 2021-present, the Recognai S.L. team.
  -
  - Licensed under the Apache License, Version 2.0 (the "License");
  - you may not use this file except in compliance with the License.
  - You may obtain a copy of the License at
  -
  -     http://www.apache.org/licenses/LICENSE-2.0
  -
  - Unless required by applicable law or agreed to in writing, software
  - distributed under the License is distributed on an "AS IS" BASIS,
  - WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  - See the License for the specific language governing permissions and
  - limitations under the License.
  -->

<template>
  <div class="container">
    <BaseLoading v-if="hasAuthToken" />

    <div class="login--left">
      <div class="login--left__header">
        <brand-logo class="form__logo" />
      </div>

      <div class="login--left__form">
        <form class="form" @submit.prevent="onLoginUser">
          <div>
            <p class="form__title" v-text="$t('login.title')" />
            <div class="form__input" :class="{ active: login.username }">
              <label class="form__label">Username</label>
              <input
                v-model="login.username"
                type="text"
                placeholder="Enter your username"
              />
            </div>
            <div class="form__input" :class="{ active: login.password }">
              <label class="form__label">Password</label>
              <input
                v-model="login.password"
                type="password"
                placeholder="Enter your password"
              />
            </div>
            <p
              v-if="deployment == 'quickstart'"
              v-html="
                $t('login.quickstart', {
                  link: $config.documentationSiteQuickStart,
                })
              "
            />
            <base-button
              type="submit"
              :disabled="!isButtonEnabled"
              class="form__button primary"
              >{{ $t("button.login") }}</base-button
            >
            <p class="form__error" v-if="error">{{ formattedError }}</p>
          </div>
        </form>

        <OAuthLogin />
      </div>
    </div>

    <div class="login--right">
      <p class="login__claim" v-html="$t('login.claim')" />
      <geometric-shape-a />
      <p
        class="login__text"
        v-html="$t('login.support', { link: $config.slackCommunity })"
      />
    </div>
  </div>
</template>

<script>
import { Notification } from "@/models/Notifications";

export default {
  layout: "app",
  data() {
    return {
      error: undefined,
      login: {
        username: "",
        password: "",
      },
      deployment: false,
      hasAuthToken: false,
    };
  },
  async created() {
    const rawAuthToken = this.$route.query.auth;

    if (!rawAuthToken) return;

    try {
      const [username, password] = atob(rawAuthToken).split(":");

      if (username && password) {
        this.hasAuthToken = true;

        try {
          await this.loginUser({ username, password });
        } catch {
          this.hasAuthToken = false;
        }
      }
    } catch {
      /* lint:disable:no-empty */
    }
  },
  async mounted() {
    try {
      const response = await fetch("deployment.json");

      const { deployment } = await response.json();

      this.deployment = deployment;
    } catch {
      this.deployment = null;
    }
  },
  computed: {
    formattedError() {
      if (this.error) {
        return this.error.toString().includes("401")
          ? "Wrong username or password. Try again"
          : this.error;
      }
    },
    isButtonEnabled() {
      return !!this.login.username && !!this.login.password;
    },
  },
  methods: {
    nextRedirect() {
      const redirect_url = this.$nuxt.$route.query.redirect || "/";
      this.$router.push({
        path: redirect_url,
      });
    },
    async loginUser(authData) {
      await this.$auth.logout();
      await this.$store.dispatch("entities/deleteAll");
      await this.$auth.loginWith("basic", {
        data: this.encodedLoginData(authData),
      });

      Notification.dispatch("clear");

      this.nextRedirect();
    },
    async onLoginUser() {
      try {
        await this.loginUser(this.login);
      } catch (err) {
        this.error = err;
      }
    },
    encodedLoginData({ username, password }) {
      return `username=${encodeURIComponent(
        username
      )}&password=${encodeURIComponent(password)}`;
    },
  },
};
</script>

<style lang="scss" scoped>
.container {
  background: $brand-secondary-color;
  display: flex;
  min-height: 100vh;
  a,
  :deep(a) {
    outline: none;
    color: $brand-primary-color;
    text-decoration: none;
    &:hover {
      color: darken($brand-primary-color, 10%);
    }
  }
}
.form {
  display: flex;
  flex-flow: column;

  &__logo {
    text-align: left;
    max-width: 120px;
    height: auto;
  }
  &__label {
    display: block;
    margin-bottom: $base-space;
    font-weight: 500;
  }
  &__title {
    @include font-size(36px);
    line-height: 1.2em;
    margin: 0 auto $base-space * 5 auto;
    color: $black-87;
    font-weight: 500;
    letter-spacing: 0.03em;
    font-family: "raptor_v2_premiumbold", "Helvetica", "Arial", sans-serif;
  }
  &__button {
    margin: 2em auto 0 auto;
    justify-content: center;
    width: 100%;
  }
  &__input {
    position: relative;
    display: block;
    margin-bottom: 1em;
    input {
      border: 1px solid palette(grey, 600);
      border-radius: $border-radius;
      padding: 0 1em;
      outline: none;
      background: transparent;
      min-height: 40px;
      width: 100%;
    }
  }
  &__error {
    color: #ff4f46;
  }
}

input:-webkit-autofill {
  box-shadow: 0 0 0px 1000px palette(white) inset;
}
.login {
  &--left {
    background: palette(white);
    padding: $base-space * 5;
    z-index: 1;
    width: 50vw;
    display: flex;
    flex-direction: column;
    justify-content: center;

    &__header {
      height: 10%;
    }

    &__form {
      width: 340px;
      margin: auto;
      height: 90%;
      display: flex;
      flex-direction: column;
      justify-content: center;
    }
  }

  &--right {
    display: flex;
    flex-flow: column;
    position: relative;
    width: 50vw;
    svg {
      position: absolute;
      max-width: 380px;
      margin: auto;
      top: 0;
      bottom: 0;
      left: 0;
      right: 0;
    }
  }
  &__claim {
    margin: auto auto;
    max-width: 500px;
    z-index: 1;
    @include font-size(40px);
    line-height: 1.3em;
    color: palette(white);
    font-family: "raptor_v2_premiumbold", "Helvetica", "Arial", sans-serif;
    transform: translateX(-1.5em);
    padding-top: 1em;
  }
  &__text {
    margin-top: 0;
    text-align: center;
    margin-bottom: $base-space * 3;
    @include font-size(16px);
    line-height: 1.4em;
    font-weight: 400;
  }
}
</style>
