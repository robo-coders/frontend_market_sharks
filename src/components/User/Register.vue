<script setup lang="ts">
import { ref, reactive, onMounted, computed } from "vue";
import { Button } from "@/components/ui/button";
import api from "@/lib/axios";

import {
  Card,
  CardHeader,
  CardTitle,
  CardDescription,
  CardContent,
  CardFooter,
} from "@/components/ui/card";
import { countries } from "countries-list";

const form = reactive({
  firstName: "",
  lastName: "",
  nickname: "",
  isAnonymous: false,
  email: "",
  whatsapp_number: "",
  countryCode: "+971",
  password: "",
  password_confirmation: "",
});

const errors = reactive({
  firstName: "",
  lastName: "",
  nickname: "",
  email: "",
  whatsapp_number: "",
  password: "",
  password_confirmation: "",
  general: "",
});

const showPassword = ref(false);
const isLoading = ref(false);
const success = ref("");

const countryOptions = computed(() => {
  const dn = new Intl.DisplayNames([navigator.language], { type: "region" });
  return Object.entries(countries)
    .map(([iso2, c]) => ({
      iso2,
      name: dn.of(iso2) || (c.name as string),
      dial: `+${c.phone}`,
    }))
    .filter((c) => c.dial !== "+undefined")
    .sort((a, b) => a.name.localeCompare(b.name));
});

onMounted(() => {
  const lang = navigator.language || "";
  const region = lang.includes("-") ? lang.split("-")[1].toUpperCase() : "";
  const match = countryOptions.value.find((c) => c.iso2 === region);
  form.countryCode = match?.dial || "+971";
});

const validate = () => {
  Object.keys(errors).forEach((key) => ((errors as any)[key] = ""));

  errors.email = form.email.trim() ? "" : "Email required";
  errors.whatsapp_number = form.whatsapp_number.trim() ? "" : "WhatsApp required";

  if (!form.isAnonymous) {
    errors.firstName = form.firstName.trim() ? "" : "First name required";
    errors.lastName = form.lastName.trim() ? "" : "Last name required";
  } else {
    errors.nickname = form.nickname.trim() ? "" : "Nickname required";
  }

  if (!form.password.trim()) errors.password = "Password required";
  if (form.password !== form.password_confirmation) {
    errors.password_confirmation = "Passwords don't match";
  }

  return Object.values(errors).every((e) => !e);
};

const submit = async () => {
  if (!validate()) return;

  isLoading.value = true;
  errors.general = "";

  const payload = {
    is_anonymous: form.isAnonymous,
    first_name: form.isAnonymous ? null : form.firstName,
    last_name: form.isAnonymous ? null : form.lastName,
    nickname: form.isAnonymous ? form.nickname : null,
    email: form.email,
    whatsapp_number: `${form.countryCode}${form.whatsapp_number}`,
    password: form.password,
    password_confirmation: form.password_confirmation,
  };

  try {
    const response = await api.post("/api/register", payload);

    const token = response.data.token;
    if (token) {
      localStorage.setItem("auth_token", token);
    }

    success.value = "registered";
    setTimeout(() => {
      window.location.href = "/app/dashboard";
    }, 2500);
  } catch (e: any) {
    if (e.response?.status === 422) {
      const laravelErrors = e.response.data.errors;
      if (laravelErrors.first_name)      errors.firstName = laravelErrors.first_name[0];
      if (laravelErrors.last_name)       errors.lastName = laravelErrors.last_name[0];
      if (laravelErrors.nickname)        errors.nickname = laravelErrors.nickname[0];
      if (laravelErrors.email)           errors.email = laravelErrors.email[0];
      if (laravelErrors.whatsapp_number) errors.whatsapp_number = laravelErrors.whatsapp_number[0];
      if (laravelErrors.password)        errors.password = laravelErrors.password[0];
    } else {
      errors.general = e.response?.data?.message || "Network error. Please try again.";
    }
  } finally {
    isLoading.value = false;
  }
};
</script>

<template>
  <section class="min-h-screen flex items-center justify-center px-4">
    <form @submit.prevent="submit" class="w-full max-w-md">
      <Card>
        <CardHeader class="text-center space-y-2">
          <CardTitle class="text-2xl font-bold">Join Market Sharks</CardTitle>
          <CardDescription>Create your account to get started.</CardDescription>
        </CardHeader>

        <CardContent class="space-y-4">
          <p v-if="errors.general" class="text-sm text-destructive text-center">
            {{ errors.general }}
          </p>

          <!-- Anonymous toggle -->
          <label class="flex items-center gap-3 p-3 border rounded-md cursor-pointer hover:bg-muted/30 transition">
            <input
              type="checkbox"
              v-model="form.isAnonymous"
              class="w-4 h-4 rounded text-primary focus:ring-primary"
            />
            <div>
              <span class="text-sm font-medium">Anonymous account</span>
              <p class="text-xs text-muted-foreground">Your name will not be shown publicly</p>
            </div>
          </label>

          <!-- First + Last -->
          <div v-if="!form.isAnonymous" class="grid grid-cols-2 gap-3">
            <div class="space-y-2">
              <label class="text-sm font-medium">First name</label>
              <input
                v-model="form.firstName"
                class="w-full rounded-md border px-3 py-2 text-sm"
                placeholder="John"
              />
              <p v-if="errors.firstName" class="text-sm text-destructive">{{ errors.firstName }}</p>
            </div>
            <div class="space-y-2">
              <label class="text-sm font-medium">Last name</label>
              <input
                v-model="form.lastName"
                class="w-full rounded-md border px-3 py-2 text-sm"
                placeholder="Doe"
              />
              <p v-if="errors.lastName" class="text-sm text-destructive">{{ errors.lastName }}</p>
            </div>
          </div>

          <!-- Nickname -->
          <div v-else class="space-y-2">
            <label class="text-sm font-medium">Nickname</label>
            <input
              v-model="form.nickname"
              class="w-full rounded-md border px-3 py-2 text-sm"
              placeholder="TraderJohn"
            />
            <p v-if="errors.nickname" class="text-sm text-destructive">{{ errors.nickname }}</p>
          </div>

          <!-- Email -->
          <div class="space-y-2">
            <label class="text-sm font-medium">Email</label>
            <input
              v-model="form.email"
              type="email"
              class="w-full rounded-md border px-3 py-2 text-sm"
              placeholder="you@email.com"
              autocomplete="email"
            />
            <p v-if="errors.email" class="text-sm text-destructive">{{ errors.email }}</p>
          </div>

          <!-- WhatsApp -->
          <div class="space-y-2">
            <label class="text-sm font-medium">WhatsApp</label>
            <div class="grid grid-cols-3 gap-2">
              <select
                v-model="form.countryCode"
                class="col-span-1 rounded-md border px-2 py-2 text-sm"
              >
                <option v-for="c in countryOptions" :key="c.iso2" :value="c.dial">
                  {{ c.dial }} — {{ c.name }}
                </option>
              </select>
              <input
                v-model="form.whatsapp_number"
                type="tel"
                class="col-span-2 rounded-md border px-3 py-2 text-sm"
                placeholder="50xxxxxxx"
              />
            </div>
            <p v-if="errors.whatsapp_number" class="text-sm text-destructive">{{ errors.whatsapp_number }}</p>
          </div>

          <!-- Password -->
          <div class="space-y-2">
            <label class="text-sm font-medium">Password</label>
            <div class="relative">
              <input
                :type="showPassword ? 'text' : 'password'"
                v-model="form.password"
                class="w-full rounded-md border px-3 py-2 pr-16 text-sm"
                placeholder="••••••••"
                autocomplete="new-password"
              />
              <button
                type="button"
                class="absolute right-3 top-1/2 -translate-y-1/2 text-xs text-muted-foreground hover:text-foreground transition"
                @click="showPassword = !showPassword"
              >
                {{ showPassword ? "Hide" : "Show" }}
              </button>
            </div>
            <p v-if="errors.password" class="text-sm text-destructive">{{ errors.password }}</p>
          </div>

          <!-- Confirm Password -->
          <div class="space-y-2">
            <label class="text-sm font-medium">Confirm Password</label>
            <input
              v-model="form.password_confirmation"
              type="password"
              class="w-full rounded-md border px-3 py-2 text-sm"
              placeholder="••••••••"
              autocomplete="new-password"
            />
            <p v-if="errors.password_confirmation" class="text-sm text-destructive">
              {{ errors.password_confirmation }}
            </p>
          </div>
        </CardContent>

        <CardFooter class="flex flex-col gap-3">
          <Button class="w-full" :disabled="isLoading" @click="submit">
            {{ isLoading ? "Creating..." : "Create Account" }}
          </Button>
          <p class="text-xs text-muted-foreground text-center">
            Already have an account?
            <a href="/login" class="text-primary hover:underline font-medium">Sign in</a>
          </p>
        </CardFooter>
      </Card>
    </form>
  </section>

  <!-- Success Modal -->
  <Teleport to="body">
    <Transition
      enter-active-class="transition duration-300 ease-out"
      enter-from-class="opacity-0 scale-95"
      enter-to-class="opacity-100 scale-100"
      leave-active-class="transition duration-200 ease-in"
      leave-from-class="opacity-100 scale-100"
      leave-to-class="opacity-0 scale-95"
    >
      <div
        v-if="success"
        class="fixed inset-0 z-50 flex items-center justify-center px-4"
      >
        <!-- Backdrop -->
        <div class="absolute inset-0 bg-black/60 backdrop-blur-sm" />

        <!-- Modal -->
        <div class="relative z-10 w-full max-w-sm rounded-2xl bg-card border border-border shadow-2xl p-8 flex flex-col items-center gap-5 text-center">

          <!-- Check icon -->
          <div class="flex items-center justify-center w-16 h-16 rounded-full bg-primary/10 border border-primary/20">
            <svg class="w-8 h-8 text-primary" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round">
              <path d="M20 6L9 17l-5-5"/>
            </svg>
          </div>

          <div class="space-y-1">
            <h3 class="text-lg font-semibold">You're in! 🎉</h3>
            <p class="text-sm text-muted-foreground">
              Account created successfully.<br/>Taking you to your dashboard…
            </p>
          </div>

          <!-- Progress bar -->
          <div class="w-full h-1 rounded-full bg-muted overflow-hidden">
            <div class="progress-bar" />
          </div>

        </div>
      </div>
    </Transition>
  </Teleport>
</template>

<style scoped>
.progress-bar {
  height: 100%;
  background: hsl(var(--primary));
  border-radius: 9999px;
  animation: progress 2.5s ease-in forwards;
}

@keyframes progress {
  from { width: 0% }
  to   { width: 100% }
}
</style>