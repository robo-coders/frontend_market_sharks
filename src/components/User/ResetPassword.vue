<script setup lang="ts">
import { ref, onMounted } from "vue";

import { Button } from "@/components/ui/button";
import {
  Card,
  CardHeader,
  CardTitle,
  CardDescription,
  CardContent,
  CardFooter,
} from "@/components/ui/card";

const email = ref("");
const token = ref("");

const password = ref("");
const passwordConfirmation = ref("");

const showPassword = ref(false);
const showPasswordConfirmation = ref(false);

const isLoading = ref(false);

const errors = ref({
  password: "",
  passwordConfirmation: "",
  general: "",
});

const success = ref("");

onMounted(() => {
  const params = new URLSearchParams(window.location.search);
  email.value = params.get("email") || "";
  token.value = params.get("token") || "";
});

const validate = () => {
  errors.value = { password: "", passwordConfirmation: "", general: "" };

  if (!token.value) {
    errors.value.general = "Reset link is invalid or expired.";
  }

  if (!email.value) {
    errors.value.general = "Reset link is invalid or expired.";
  }

  if (!password.value.trim()) {
    errors.value.password = "New password is required";
  }

  if (!passwordConfirmation.value.trim()) {
    errors.value.passwordConfirmation = "Please confirm your new password";
  }

  if (
    password.value.trim() &&
    passwordConfirmation.value.trim() &&
    password.value !== passwordConfirmation.value
  ) {
    errors.value.passwordConfirmation = "Passwords do not match";
  }

  return (
    !errors.value.general &&
    !errors.value.password &&
    !errors.value.passwordConfirmation
  );
};

const submit = async () => {
  if (!validate()) return;

  isLoading.value = true;
  success.value = "";
  errors.value.general = "";

  try {
    // 🔌 Backend hookup later:
    // POST /reset-password { token, email, password, password_confirmation }
    console.log("Reset password payload:", {
      token: token.value,
      email: email.value,
      password: password.value,
      password_confirmation: passwordConfirmation.value,
    });

    success.value = "Password reset successful. You can now login.";

    // Optional: redirect after success
    // setTimeout(() => (window.location.href = "/login"), 800);
  } catch (e: any) {
    errors.value.general =
      e?.message || "Reset failed. Please request a new reset link.";
  } finally {
    isLoading.value = false;
  }
};
</script>

<template>
  <section class="min-h-screen flex items-center justify-center px-4">
    <Card class="w-full max-w-md">
      <CardHeader class="text-center space-y-2">
        <CardTitle class="text-2xl font-bold">Reset password</CardTitle>
        <CardDescription>
          Choose a new password for your account.
        </CardDescription>
      </CardHeader>

      <CardContent class="space-y-4">
        <p v-if="success" class="text-sm text-green-600 text-center">
          {{ success }}
        </p>

        <p v-if="errors.general" class="text-sm text-destructive text-center">
          {{ errors.general }}
        </p>

        <!-- New password -->
        <div class="space-y-2">
          <label class="text-sm font-medium">New password</label>
          <div class="relative">
            <input
              v-model="password"
              :type="showPassword ? 'text' : 'password'"
              class="w-full rounded-md border px-3 py-2 pr-10 text-sm"
              placeholder="••••••••"
            />
            <button
              type="button"
              class="absolute inset-y-0 right-3 flex items-center text-muted-foreground"
              @click="showPassword = !showPassword"
            >
              <span class="text-xs">
                {{ showPassword ? "Hide" : "Show" }}
              </span>
            </button>
          </div>
          <p v-if="errors.password" class="text-sm text-destructive">
            {{ errors.password }}
          </p>
        </div>

        <!-- Confirm password -->
        <div class="space-y-2">
          <label class="text-sm font-medium">Confirm new password</label>
          <div class="relative">
            <input
              v-model="passwordConfirmation"
              :type="showPasswordConfirmation ? 'text' : 'password'"
              class="w-full rounded-md border px-3 py-2 pr-10 text-sm"
              placeholder="••••••••"
            />
            <button
              type="button"
              class="absolute inset-y-0 right-3 flex items-center text-muted-foreground"
              @click="showPasswordConfirmation = !showPasswordConfirmation"
            >
              <span class="text-xs">
                {{ showPasswordConfirmation ? "Hide" : "Show" }}
              </span>
            </button>
          </div>
          <p
            v-if="errors.passwordConfirmation"
            class="text-sm text-destructive"
          >
            {{ errors.passwordConfirmation }}
          </p>
        </div>

        <p v-if="email" class="text-xs text-muted-foreground text-center">
          Resetting password for <span class="font-medium">{{ email }}</span>
        </p>
      </CardContent>

      <CardFooter class="flex flex-col gap-3">
        <Button class="w-full" :disabled="isLoading" @click="submit">
          {{ isLoading ? "Resetting..." : "Reset password" }}
        </Button>

        <a href="/login" class="text-sm text-primary hover:underline text-center">
          Back to login
        </a>
      </CardFooter>
    </Card>
  </section>
</template>
