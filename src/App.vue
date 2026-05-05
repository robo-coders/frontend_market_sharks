<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount, computed } from "vue";

import Navbar from "@/components/Navbar.vue";
import Hero from "@/components/Hero.vue";
import Partners from "@/components/Partners.vue";
import Benefits from "@/components/Benefits.vue";
import Services from "@/components/Services.vue";
import Pricing from "@/components/Pricing.vue";
import Community from "@/components/Community.vue";
import Contact from "@/components/Contact.vue";
import FAQ from "@/components/FAQ.vue";
import Footer from "@/components/Footer.vue";

import AppDashboard from "@/pages/app/Dashboard.vue";
import AppBilling from "@/pages/app/Billing.vue";
import AppProfile from "@/pages/app/Profile.vue";

import Checkout from "@/components/Checkout.vue";
import Register from "@/components/User/Register.vue";
import Login from "@/components/User/Login.vue";
import ForgotPassword from "@/components/User/ForgotPassword.vue";
import ResetPassword from "@/components/User/ResetPassword.vue";

/* ---------------- ROUTE STATE ---------------- */

const route = ref(window.location.pathname);

const syncRoute = () => {
  route.value = window.location.pathname;
};

onMounted(() => window.addEventListener("popstate", syncRoute));
onBeforeUnmount(() => window.removeEventListener("popstate", syncRoute));

/* ---------------- AUTH STATE ---------------- */

const token = ref(localStorage.getItem("auth_token"));

window.addEventListener("storage", () => {
  token.value = localStorage.getItem("auth_token");
});

/* ---------------- ROUTE HELPERS ---------------- */

const isProtectedRoute = computed(() => route.value.startsWith("/app"));

const isAuthPage = computed(() =>
  ["/login", "/register"].includes(route.value)
);

/* ---------------- GUARDS ---------------- */

// Logged-in user hits /login or /register → send to dashboard
if (isAuthPage.value && token.value) {
  window.location.href = "/app/dashboard";
}
</script>

<template>
  <!-- Protect /app/* — no token → show login -->
  <Login v-if="isProtectedRoute && !token" />

  <!-- Checkout -->
  <Checkout v-else-if="route === '/checkout'" />

  <!-- Auth Pages -->
  <Register v-else-if="route === '/register'" />
  <Login v-else-if="route === '/login'" />

  <!-- Password -->
  <ForgotPassword v-else-if="route === '/forgot-password'" />
  <ResetPassword v-else-if="route === '/reset-password'" />

  <!-- App Pages -->
  <AppDashboard v-else-if="route === '/app/dashboard'" />
  <AppBilling v-else-if="route === '/app/billing'" />
  <AppProfile v-else-if="route === '/app/profile'" />

  <!-- Landing -->
  <template v-else>
    <Navbar />
    <Hero />
    <Partners/>
    <Benefits />
    <Services />
    <Pricing />
    <Community />
    <Contact />
    <FAQ />
    <Footer />
  </template>
</template>