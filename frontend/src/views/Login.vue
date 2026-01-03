<template>
  <div id="login" :class="{ recaptcha: recaptcha }">
    <div class="login-container">
      <div class="login-card">
        <!-- Logo from Branding folder -->
        <div class="login-logo">
          <img :src="logoURL" alt="File Browser" />
        </div>

        <!-- Title -->
        <h1 class="login-title">{{ createMode ? 'Créer un compte' : 'Connexion' }}</h1>
        <p class="login-subtitle">{{ createMode ? 'File Browser' : 'Utilisez votre compte File Browser' }}</p>

        <!-- Error/Logout Messages -->
        <div v-if="reason != null" class="message-box message-warning">
          {{ t(`login.logout_reasons.${reason}`) }}
        </div>
        <div v-if="error !== ''" class="message-box message-error">
          {{ error }}
        </div>

        <!-- Login Form -->
        <form @submit="submit" class="login-form">
          <!-- Username Field -->
          <div class="input-group">
            <input
              autofocus
              class="google-input"
              type="text"
              autocapitalize="off"
              v-model="username"
              :placeholder="t('login.username')"
              required
            />
          </div>

          <!-- Password Field -->
          <div class="input-group">
            <input
              class="google-input"
              type="password"
              v-model="password"
              :placeholder="t('login.password')"
              required
            />
          </div>

          <!-- Password Confirm (Signup mode) -->
          <div v-if="createMode" class="input-group">
            <input
              class="google-input"
              type="password"
              v-model="passwordConfirm"
              :placeholder="t('login.passwordConfirm')"
              required
            />
          </div>

          <!-- Recaptcha -->
          <div v-if="recaptcha" id="recaptcha" style="margin-top: 16px;"></div>

          <!-- Action Buttons -->
          <div class="login-actions">
            <button 
              type="button" 
              class="text-button" 
              v-if="signup"
              @click="toggleMode"
            >
              {{ createMode ? t("login.loginInstead") : t("login.createAnAccount") }}
            </button>
            <button type="submit" class="submit-button">
              {{ createMode ? t('login.signup') : t('login.submit') }}
            </button>
          </div>
        </form>
      </div>

      <!-- Footer -->
      <div class="login-footer">
        <span>Français (France)</span>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { StatusError } from "@/api/utils";
import * as auth from "@/utils/auth";
import {
  name,
  logoURL,
  recaptcha,
  recaptchaKey,
  signup,
} from "@/utils/constants";
import { inject, onMounted, ref } from "vue";
import { useI18n } from "vue-i18n";
import { useRoute, useRouter } from "vue-router";

// Define refs
const createMode = ref<boolean>(false);
const error = ref<string>("");
const username = ref<string>("");
const password = ref<string>("");
const passwordConfirm = ref<string>("");

const route = useRoute();
const router = useRouter();
const { t } = useI18n({});
// Define functions
const toggleMode = () => (createMode.value = !createMode.value);

const $showError = inject<IToastError>("$showError")!;

const reason = route.query["logout-reason"] ?? null;

const submit = async (event: Event) => {
  event.preventDefault();
  event.stopPropagation();

  const redirect = (route.query.redirect || "/files/") as string;

  let captcha = "";
  if (recaptcha) {
    captcha = window.grecaptcha.getResponse();

    if (captcha === "") {
      error.value = t("login.wrongCredentials");
      return;
    }
  }

  if (createMode.value) {
    if (password.value !== passwordConfirm.value) {
      error.value = t("login.passwordsDontMatch");
      return;
    }
  }

  try {
    if (createMode.value) {
      await auth.signup(username.value, password.value);
    }

    await auth.login(username.value, password.value, captcha);
    router.push({ path: redirect });
  } catch (e: any) {
    // console.error(e);
    if (e instanceof StatusError) {
      if (e.status === 409) {
        error.value = t("login.usernameTaken");
      } else if (e.status === 403) {
        error.value = t("login.wrongCredentials");
      } else if (e.status === 400) {
        const match = e.message.match(/minimum length is (\d+)/);
        if (match) {
          error.value = t("login.passwordTooShort", { min: match[1] });
        } else {
          error.value = e.message;
        }
      } else {
        $showError(e);
      }
    }
  }
};

// Run hooks
onMounted(() => {
  if (!recaptcha) return;

  window.grecaptcha.ready(function () {
    window.grecaptcha.render("recaptcha", {
      sitekey: recaptchaKey,
    });
  });
});
</script>
