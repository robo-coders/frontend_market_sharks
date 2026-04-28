<script setup lang="ts">
import { ref } from "vue";
import api from "@/lib/axios";
import { Button } from "@/components/ui/button";
import { Card, CardHeader, CardTitle, CardDescription, CardContent, CardFooter } from "@/components/ui/card";
import { Eye, EyeOff } from "lucide-vue-next";

const email         = ref("");
const password      = ref("");
const showPassword  = ref(false);
const isLoading     = ref(false);

const errors = ref({ email: "", password: "", general: "" });

const validate = () => {
  errors.value = { email: "", password: "", general: "" };
  if (!email.value.trim())    errors.value.email    = "Email is required";
  if (!password.value.trim()) errors.value.password = "Password is required";
  return !errors.value.email && !errors.value.password;
};

const submit = async () => {
  if (!validate()) return;
  isLoading.value = true;
  errors.value.general = "";
  try {
    const response = await api.post("/api/login", {
      email: email.value,
      password: password.value,
    });
    const token = response.data.token;
    if (token) localStorage.setItem("auth_token", token);
    window.location.href = "/app/dashboard";
  } catch (e: any) {
    errors.value.general = e.response?.status === 422
      ? "Invalid email or password."
      : e.response?.data?.message || "Login failed. Please try again.";
  } finally {
    isLoading.value = false;
  }
};
</script>

<template>
  <section class="min-h-screen bg-background flex flex-col items-center justify-center px-4">

    <!-- Top logo linking back to main site -->
    <a
      href="https://marketsharks.live"
      class="flex items-center gap-2 mb-8 group"
    >
      <div class="flex h-8 w-8 items-center justify-center rounded-lg bg-primary">
        <span class="text-white font-black text-sm">MS</span>
      </div>
      <span class="text-sm font-semibold text-foreground group-hover:text-primary transition-colors">
        Market Sharks
      </span>
    </a>

    <!-- Login card -->
    <form @submit.prevent="submit" class="w-full max-w-md">
      <Card class="w-full">
        <CardHeader class="text-center space-y-1 pb-4">
          <CardTitle class="text-xl font-bold tracking-tight">
            Welcome back
          </CardTitle>
          <CardDescription>
            Sign in to your Market Sharks account.
          </CardDescription>
        </CardHeader>

        <CardContent class="space-y-4">
          <!-- General error -->
          <div
            v-if="errors.general"
            class="rounded-lg bg-destructive/10 border border-destructive/20 px-4 py-3 text-sm text-destructive text-center"
          >
            {{ errors.general }}
          </div>

          <!-- Email -->
          <div class="space-y-1.5">
            <label class="text-sm font-medium">Email</label>
            <input
              v-model="email"
              type="email"
              class="w-full rounded-md border bg-background px-3 py-2 text-sm transition-colors focus:outline-none focus:ring-2 focus:ring-primary/50 focus:border-primary"
              :class="errors.email ? 'border-destructive' : 'border-input'"
              placeholder="you@email.com"
              autocomplete="email"
            />
            <p v-if="errors.email" class="text-xs text-destructive">{{ errors.email }}</p>
          </div>

          <!-- Password -->
          <div class="space-y-1.5">
            <label class="text-sm font-medium">Password</label>
            <div class="relative">
              <input
                v-model="password"
                :type="showPassword ? 'text' : 'password'"
                class="w-full rounded-md border bg-background px-3 py-2 pr-10 text-sm transition-colors focus:outline-none focus:ring-2 focus:ring-primary/50 focus:border-primary"
                :class="errors.password ? 'border-destructive' : 'border-input'"
                placeholder="••••••••"
                autocomplete="current-password"
              />
              <button
                type="button"
                class="absolute inset-y-0 right-3 flex items-center text-muted-foreground hover:text-foreground transition-colors"
                @click="showPassword = !showPassword"
                aria-label="Toggle password visibility"
              >
                <Eye v-if="!showPassword" class="size-4" />
                <EyeOff v-else class="size-4" />
              </button>
            </div>
            <p v-if="errors.password" class="text-xs text-destructive">{{ errors.password }}</p>
          </div>

          <!-- Forgot password -->
          <div class="text-right">
            <a href="/forgot-password" class="text-xs text-primary hover:underline underline-offset-2">
              Forgot password?
            </a>
          </div>
        </CardContent>

        <CardFooter class="flex flex-col gap-3 pt-2">
          <Button class="w-full" :disabled="isLoading" @click="submit">
            {{ isLoading ? "Signing in..." : "Sign In" }}
          </Button>

          <p class="text-xs text-muted-foreground text-center">
            Don't have an account?
            <a href="/register" class="text-primary hover:underline underline-offset-2 font-medium">
              Register here
            </a>
          </p>
        </CardFooter>
      </Card>
    </form>

    <!-- Back to website -->
    <a
      href="https://marketsharks.live"
      class="mt-6 text-xs text-muted-foreground hover:text-foreground transition-colors"
    >
      ← Back to marketsharks.live
    </a>

  </section>
</template>