<script setup lang="ts">
import { ref, onMounted, computed } from "vue";
import AppLayout from "@/layouts/AppLayout.vue";
import { Card, CardContent, CardHeader, CardTitle, CardDescription } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Badge } from "@/components/ui/badge";
import { TrendingUp, Video, BookOpen, MessageCircle, Clock, XCircle, ShieldOff } from "lucide-vue-next";
import api from "@/lib/axios";

const pageUser = ref<any>(null);
const isLoading = ref(true);

const stats = {
  liveNow: true,
  nextSession: "18:00 UTC",
  todaySignals: 4,
  communityMembers: 312,
};

onMounted(async () => {
  try {
    const { data } = await api.get("/api/me");
    pageUser.value = data;
  } catch {
    window.location.href = "/login";
  } finally {
    isLoading.value = false;
  }
});

const hasActivePlan = computed(() =>
  pageUser.value?.user?.status === "active" &&
  pageUser.value?.user?.subscription_status !== "none" &&
  pageUser.value?.plan
);

// Payment submitted, waiting for admin approval
const isPending = computed(() =>
  pageUser.value?.payment_request_status === "pending"
);

// Payment was rejected by admin
const isRejected = computed(() =>
  pageUser.value?.payment_request_status === "rejected"
);

// Account blocked
const isBlocked = computed(() =>
  pageUser.value?.user?.status === "blocked"
);

// No payment at all
const hasNoPlan = computed(() =>
  !hasActivePlan.value && !isPending.value && !isRejected.value && !isBlocked.value
);
</script>

<template>
  <div v-if="isLoading" class="min-h-screen flex items-center justify-center">
    <p class="text-muted-foreground text-sm">Loading...</p>
  </div>

  <AppLayout v-else-if="pageUser" title="Dashboard">

    <!-- BLOCKED -->
    <template v-if="isBlocked">
      <Card class="mb-6 border-destructive/40">
        <CardHeader>
          <CardTitle class="flex items-center gap-2">
            <ShieldOff class="size-5 text-destructive" />
            Account Suspended
          </CardTitle>
          <CardDescription>Your account has been restricted.</CardDescription>
        </CardHeader>
        <CardContent>
          <div class="p-4 rounded-xl bg-destructive/10 text-destructive text-sm">
            Your account has been suspended. Please contact support for assistance.
          </div>
        </CardContent>
      </Card>
    </template>

    <!-- PAYMENT PENDING -->
    <template v-else-if="isPending">
      <Card class="mb-6">
        <CardHeader>
          <CardTitle class="flex items-center gap-2">
            <Clock class="size-5 text-primary" />
            Welcome, {{ pageUser.user.name }} 👋
          </CardTitle>
          <CardDescription>Your payment is being reviewed.</CardDescription>
        </CardHeader>
        <CardContent class="space-y-4">
          <div class="p-4 rounded-xl bg-muted text-sm space-y-1">
            <p class="font-medium">Your <span class="capitalize">{{ pageUser.plan }}</span> plan request has been submitted.</p>
            <p class="text-muted-foreground">Our team is reviewing your payment. You'll receive full access once approved — this usually takes a few hours.</p>
          </div>
          <Badge variant="secondary">⏳ Pending Approval</Badge>
        </CardContent>
      </Card>

      <!-- Locked preview cards -->
      <div class="grid md:grid-cols-2 lg:grid-cols-4 gap-6 mb-6">
        <Card v-for="item in ['Live Session', 'Today\'s Signals', 'Learning Content', 'Community']" :key="item" class="opacity-50 pointer-events-none">
          <CardHeader>
            <CardTitle class="text-sm text-muted-foreground">{{ item }}</CardTitle>
          </CardHeader>
          <CardContent>
            <p class="text-xs text-muted-foreground">Available after approval.</p>
          </CardContent>
        </Card>
      </div>
    </template>

    <!-- PAYMENT REJECTED -->
    <template v-else-if="isRejected">
      <Card class="mb-6 border-destructive/40">
        <CardHeader>
          <CardTitle class="flex items-center gap-2">
            <XCircle class="size-5 text-destructive" />
            Payment Rejected
          </CardTitle>
          <CardDescription>Unfortunately your payment could not be approved.</CardDescription>
        </CardHeader>
        <CardContent class="space-y-4">
          <div class="p-4 rounded-xl bg-destructive/10 text-destructive text-sm">
            Your payment proof was rejected. Please try again with a valid payment screenshot.
          </div>
          <a href="/pricing">
            <Button>Try Again</Button>
          </a>
        </CardContent>
      </Card>
    </template>

    <!-- NO PLAN -->
    <template v-else-if="hasNoPlan">
      <Card class="mb-6">
        <CardHeader>
          <CardTitle>Welcome, {{ pageUser.user.name }} 👋</CardTitle>
          <CardDescription>You don't have an active plan yet.</CardDescription>
        </CardHeader>
        <CardContent class="space-y-4">
          <div class="p-4 rounded-xl bg-muted text-sm">
            You're registered but haven't activated a subscription. Choose a plan to unlock live sessions, signals, and the trading community.
          </div>
          <a href="/pricing">
            <Button>View Plans & Activate</Button>
          </a>
        </CardContent>
      </Card>

      <div class="grid md:grid-cols-2 lg:grid-cols-4 gap-6 mb-6">
        <Card v-for="item in ['Live Session', 'Today\'s Signals', 'Learning Content', 'Community']" :key="item" class="opacity-50 pointer-events-none">
          <CardHeader>
            <CardTitle class="text-sm text-muted-foreground">{{ item }}</CardTitle>
          </CardHeader>
          <CardContent>
            <p class="text-xs text-muted-foreground">Available after activating a plan.</p>
          </CardContent>
        </Card>
      </div>
    </template>

    <!-- ACTIVE PLAN -->
    <template v-else-if="hasActivePlan">
      <Card class="mb-6">
        <CardHeader>
          <CardTitle>Welcome back, {{ pageUser.user.name }} 👋</CardTitle>
          <CardDescription>Here's your current account overview.</CardDescription>
        </CardHeader>
        <CardContent class="space-y-3">
          <div class="flex flex-wrap items-center gap-3">
            <Badge variant="default">
              Plan: {{ pageUser.plan.charAt(0).toUpperCase() + pageUser.plan.slice(1) }}
            </Badge>
            <span class="text-sm text-muted-foreground">
              Expires: {{ pageUser.expires_at ?? 'Not set' }}
            </span>
          </div>
        </CardContent>
      </Card>

      <!-- Quick Actions -->
      <div class="grid md:grid-cols-2 lg:grid-cols-4 gap-6 mb-6">
        <Card>
          <CardHeader>
            <CardTitle class="flex items-center gap-2">
              <Video class="size-5 text-primary" /> Live Session
            </CardTitle>
          </CardHeader>
          <CardContent class="space-y-3">
            <div v-if="stats.liveNow" class="text-sm">🔴 Live now</div>
            <div v-else class="text-sm text-muted-foreground">Next: {{ stats.nextSession }}</div>
            <Button class="w-full">Join Now</Button>
          </CardContent>
        </Card>

        <Card>
          <CardHeader>
            <CardTitle class="flex items-center gap-2">
              <TrendingUp class="size-5 text-primary" /> Today's Signals
            </CardTitle>
          </CardHeader>
          <CardContent class="space-y-3">
            <div class="text-sm">{{ stats.todaySignals }} high-probability setups today.</div>
            <Button variant="secondary" class="w-full">View Signals</Button>
          </CardContent>
        </Card>

        <Card>
          <CardHeader>
            <CardTitle class="flex items-center gap-2">
              <BookOpen class="size-5 text-primary" /> Learning Content
            </CardTitle>
          </CardHeader>
          <CardContent class="space-y-3">
            <div class="text-sm text-muted-foreground">Premium lessons, order flow, and risk management.</div>
            <Button variant="secondary" class="w-full">Access Library</Button>
          </CardContent>
        </Card>

        <Card>
          <CardHeader>
            <CardTitle class="flex items-center gap-2">
              <MessageCircle class="size-5 text-primary" /> Community
            </CardTitle>
          </CardHeader>
          <CardContent class="space-y-3">
            <div class="text-sm">{{ stats.communityMembers }} traders inside.</div>
            <Button variant="secondary" class="w-full">Join WhatsApp</Button>
          </CardContent>
        </Card>
      </div>

      <!-- Subscription Overview -->
      <Card>
        <CardHeader>
          <CardTitle>Subscription Overview</CardTitle>
          <CardDescription>Manage your plan and renewal status.</CardDescription>
        </CardHeader>
        <CardContent class="space-y-4">
          <div class="flex items-center justify-between flex-wrap gap-4">
            <div>
              <p class="font-medium">
                {{ pageUser.plan.charAt(0).toUpperCase() + pageUser.plan.slice(1) }} Plan
              </p>
              <p class="text-sm text-muted-foreground">Renews on {{ pageUser.expires_at ?? 'Not set' }}</p>
            </div>
            <Button variant="secondary" @click="window.location.href = '/app/billing'">
              Upgrade Plan
            </Button>
          </div>
        </CardContent>
      </Card>
    </template>

  </AppLayout>
</template>