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

import { Eye, EyeOff } from "lucide-vue-next";

const email = ref("");
const password = ref("");
const showPassword = ref(false);

const errors = ref({
  email: "",
  password: "",
  general: "",
});

const isLoading = ref(false);

const validate = () => {
  errors.value = { email: "", password: "", general: "" };

  if (!email.value.trim()) {
    errors.value.email = "Email is required";
  }

  if (!password.value.trim()) {
    errors.value.password = "Password is required";
  }

  return !errors.value.email && !errors.value.password;
};

const submit = async () => {
  if (!validate()) return;

  isLoading.value = true;

  try {
    // 🔌 Backend hookup later
    console.log({
      email: email.value,
      password: password.value,
    });

    // redirect to dashboard later
  } catch (e: any) {
    errors.value.general =
      e?.message || "Invalid email or password.";
  } finally {
    isLoading.value = false;
  }
};
</script>

<template>
  <section class="min-h-screen flex items-center justify-center px-4">
    <Card class="w-full max-w-md">
      <CardHeader class="text-center space-y-2">
        <CardTitle class="text-2xl font-bold">
          Login to Market Sharks
        </CardTitle>
        <CardDescription>
          Enter your credentials to continue.
        </CardDescription>
      </CardHeader>

      <CardContent class="space-y-4">
        <!-- General error -->
        <p
          v-if="errors.general"
          class="text-sm text-destructive text-center"
        >
          {{ errors.general }}
        </p>

        <!-- Email -->
        <div class="space-y-2">
          <label class="text-sm font-medium">Email</label>
          <input
            v-model="email"
            type="email"
            class="w-full rounded-md border px-3 py-2 text-sm"
            placeholder="you@email.com"
          />
          <p v-if="errors.email" class="text-sm text-destructive">
            {{ errors.email }}
          </p>
        </div>

        <!-- Password -->
        <div class="space-y-2">
          <label class="text-sm font-medium">Password</label>

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
              <Eye v-if="!showPassword" class="size-4" />
              <EyeOff v-else class="size-4" />
            </button>
          </div>

          <p v-if="errors.password" class="text-sm text-destructive">
            {{ errors.password }}
          </p>
        </div>

        <!-- Forgot password -->
        <div class="text-right">
          <a
            href="/forgot-password"
            class="text-sm text-primary hover:underline"
          >
            Forgot password?
          </a>
        </div>
      </CardContent>

      <CardFooter class="flex flex-col gap-3">
        <Button
          class="w-full"
          :disabled="isLoading"
          @click="submit"
        >
          {{ isLoading ? "Logging in..." : "Login" }}
        </Button>
      </CardFooter>
    </Card>
  </section>
</template>
