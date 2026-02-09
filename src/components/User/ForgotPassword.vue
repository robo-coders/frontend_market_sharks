<script setup lang="ts">
import { ref } from "vue";

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
const isLoading = ref(false);
const error = ref("");
const success = ref("");

const validate = () => {
  error.value = "";
  if (!email.value.trim()) {
    error.value = "Email is required";
    return false;
  }
  return true;
};

const submit = async () => {
  if (!validate()) return;

  isLoading.value = true;
  error.value = "";
  success.value = "";

  try {
    // 🔌 Backend hookup later:
    // POST /forgot-password { email }
    console.log("Forgot password for:", email.value);

    // Always show success message (security best practice)
    success.value =
      "If an account exists for this email, we’ve sent a password reset link.";
  } catch (e: any) {
    error.value = e?.message || "Something went wrong. Please try again.";
  } finally {
    isLoading.value = false;
  }
};
</script>

<template>
  <section class="min-h-screen flex items-center justify-center px-4">
    <Card class="w-full max-w-md">
      <CardHeader class="text-center space-y-2">
        <CardTitle class="text-2xl font-bold">Forgot password</CardTitle>
        <CardDescription>
          Enter your email and we’ll send you a reset link.
        </CardDescription>
      </CardHeader>

      <CardContent class="space-y-4">
        <p v-if="success" class="text-sm text-green-600 text-center">
          {{ success }}
        </p>

        <p v-if="error" class="text-sm text-destructive text-center">
          {{ error }}
        </p>

        <div class="space-y-2">
          <label class="text-sm font-medium">Email</label>
          <input
            v-model="email"
            type="email"
            class="w-full rounded-md border px-3 py-2 text-sm"
            placeholder="you@email.com"
          />
        </div>
      </CardContent>

      <CardFooter class="flex flex-col gap-3">
        <Button class="w-full" :disabled="isLoading" @click="submit">
          {{ isLoading ? "Sending..." : "Send reset link" }}
        </Button>

        <a href="/login" class="text-sm text-primary hover:underline text-center">
          Back to login
        </a>
      </CardFooter>
    </Card>
  </section>
</template>
