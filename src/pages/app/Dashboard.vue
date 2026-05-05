<script setup lang="ts">
import { ref, onMounted, computed } from "vue";
import AppLayout from "@/layouts/AppLayout.vue";
import { Card, CardContent, CardHeader, CardTitle, CardDescription } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Badge } from "@/components/ui/badge";
import {
  TrendingUp, Video, BookOpen, MessageCircle,
  Clock, XCircle, ShieldOff, Lock, CalendarDays, Zap
} from "lucide-vue-next";
import api from "@/lib/axios";

const pageUser = ref<any>(null);
const isLoading = ref(true);

onMounted(async () => {
  try {
    const { data } = await api.get("/api/me");
    pageUser.value = data;
    
    console.log("hasActivePlan:", 
      data?.user?.status, 
      data?.payment_request_status, 
      data?.plan, 
      data?.expires_at
    );
    
  } catch {
    window.location.href = "/login";
  } finally {
    isLoading.value = false;
  }
});

function friendlyDate(d: string | null | undefined) {
  if (!d) return "Not set";
  return new Date(d).toLocaleDateString("en-US", { year: "numeric", month: "long", day: "numeric" });
}

function daysRemaining(d: string | null | undefined): number {
  if (!d) return 0;
  return Math.max(0, Math.ceil((new Date(d).getTime() - Date.now()) / 86_400_000));
}

const hasActivePlan = computed(() => {
  if (!pageUser.value) return false;
  const status = pageUser.value?.user?.status;
  const prStatus = pageUser.value?.payment_request_status;
  const plan = pageUser.value?.plan;
  const expires = pageUser.value?.expires_at;
  return (status === "active" || prStatus === "approved") && !!plan && !!expires;
});

const isPending = computed(() => pageUser.value?.payment_request_status === "pending");
const isRejected = computed(() => pageUser.value?.payment_request_status === "rejected");
const isBlocked = computed(() => pageUser.value?.user?.status === "blocked");

const hasNoPlan = computed(() => {
  if (!pageUser.value) return false;
  return !hasActivePlan.value && !isPending.value && !isRejected.value && !isBlocked.value;
});

const planLabel = computed(() => {
  const p = pageUser.value?.plan ?? "";
  return p.charAt(0).toUpperCase() + p.slice(1);
});

const days = computed(() => daysRemaining(pageUser.value?.expires_at));
</script>

<template>
  <div v-if="isLoading" class="min-h-screen flex items-center justify-center">
    <p class="text-muted-foreground text-sm animate-pulse">Loading...</p>
  </div>

  <AppLayout v-else-if="pageUser" title="Dashboard">
    <template v-if="isBlocked">
      <Card class="border-destructive/40 bg-destructive/5">
        <CardHeader>
          <CardTitle class="flex items-center gap-2 text-destructive">
            <ShieldOff class="size-5" /> Account Suspended
          </CardTitle>
          <CardDescription>Your account has been restricted.</CardDescription>
        </CardHeader>
        <CardContent>
          <p class="text-sm text-muted-foreground">
            Your account has been suspended. Please contact
            <a href="mailto:support@marketsharks.live" class="underline underline-offset-2 text-foreground">
              support@marketsharks.live
            </a>
            for assistance.
          </p>
        </CardContent>
      </Card>
    </template>

    <template v-else-if="isPending">
      <Card class="mb-6">
        <CardContent class="pt-6">
          <div class="flex items-start gap-4">
            <div class="flex h-10 w-10 shrink-0 items-center justify-center rounded-full bg-primary/10">
              <Clock class="size-5 text-primary" />
            </div>
            <div>
              <p class="font-semibold text-base">Payment under review</p>
              <p class="text-sm text-muted-foreground mt-1">
                Your <span class="text-foreground font-medium capitalize">{{ pageUser.plan }}</span> plan
                request has been submitted. Our team is verifying your payment —
                this usually takes a few hours.
              </p>
              <Badge variant="secondary" class="mt-3">⏳ Pending Approval</Badge>
            </div>
          </div>
        </CardContent>
      </Card>

      <p class="text-xs text-muted-foreground uppercase tracking-widest font-medium mb-3 px-0.5">
        Coming with your plan
      </p>
      <div class="grid sm:grid-cols-2 lg:grid-cols-3 gap-4">
        <Card v-for="item in [
          { icon: Video, label: 'Live Sessions', desc: 'Daily live trading sessions with our analysts.' },
          { icon: TrendingUp, label: 'Today\'s Signals', desc: 'High-probability trade setups delivered daily.' },
          { icon: BookOpen, label: 'Learning Library', desc: 'Order flow, risk management, and more.' },
        ]" :key="item.label"
          class="opacity-40 pointer-events-none select-none"
        >
          <CardHeader class="pb-2">
            <CardTitle class="text-sm flex items-center gap-2">
              <component :is="item.icon" class="size-4 text-muted-foreground" />
              {{ item.label }}
              <Lock class="size-3 ml-auto text-muted-foreground/60" />
            </CardTitle>
          </CardHeader>
          <CardContent>
            <p class="text-xs text-muted-foreground">{{ item.desc }}</p>
          </CardContent>
        </Card>
      </div>
    </template>

    <template v-else-if="isRejected">
      <Card class="mb-6 border-destructive/40 bg-destructive/5">
        <CardHeader>
          <CardTitle class="flex items-center gap-2 text-destructive">
            <XCircle class="size-5" /> Payment Rejected
          </CardTitle>
          <CardDescription>Your payment proof could not be verified.</CardDescription>
        </CardHeader>
        <CardContent class="space-y-4">
          <p class="text-sm text-muted-foreground">
            Please resubmit with a clear screenshot showing the full transaction.
            If you believe this is an error contact
            <a href="mailto:support@marketsharks.live" class="underline underline-offset-2 text-foreground">
              support@marketsharks.live
            </a>.
          </p>
          <a href="/pricing">
            <Button variant="destructive">Resubmit Payment</Button>
          </a>
        </CardContent>
      </Card>
    </template>

    <template v-else-if="hasActivePlan">
      <Card class="mb-6">
        <CardContent class="pt-6">
          <div class="flex items-start justify-between flex-wrap gap-4">
            <div>
              <p class="text-lg font-bold tracking-tight">
                Welcome back, {{ pageUser.user.name }} 👋
              </p>
              <p class="text-sm text-muted-foreground mt-1">
                Here's your Market Sharks member overview.
              </p>
              <div class="flex items-center gap-2 mt-3 flex-wrap">
                <Badge variant="default">{{ planLabel }} Plan</Badge>
                <span class="text-xs text-muted-foreground flex items-center gap-1">
                  <CalendarDays class="size-3" />
                  Renews {{ friendlyDate(pageUser.expires_at) }}
                </span>
                <span
                  :class="[
                    'text-xs font-medium px-2 py-0.5 rounded-full',
                    days <= 7
                      ? 'bg-amber-500/10 text-amber-500'
                      : 'bg-emerald-500/10 text-emerald-500'
                  ]"
                >
                  {{ days }} days left
                </span>
              </div>
            </div>
          </div>
        </CardContent>
      </Card>

      <Card class="mb-6">
        <CardContent class="pt-6 pb-6">
          <div class="flex items-center justify-between flex-wrap gap-4">
            <div class="flex items-center gap-4">
              <div class="flex h-11 w-11 shrink-0 items-center justify-center rounded-full bg-primary/10">
                <MessageCircle class="size-5 text-primary" />
              </div>
              <div>
                <p class="font-semibold text-sm">Join the Traders Community</p>
                <p class="text-xs text-muted-foreground mt-0.5">
                  Active traders — signals, discussion and live calls.

                </p>
              </div>
            </div>
              <a
                v-if="pageUser.whatsapp_link"
                :href="pageUser.whatsapp_link"
                target="_blank"
                rel="noopener noreferrer"
              >
                <Button class="gap-2 shrink-0">
                  <MessageCircle class="size-4" /> Join WhatsApp Group
                </Button>
              </a>
          </div>
        </CardContent>
      </Card>

      <p class="text-xs text-muted-foreground uppercase tracking-widest font-medium mb-3 px-0.5">
        Coming in Version 2
      </p>
      <div class="grid sm:grid-cols-2 lg:grid-cols-3 gap-4">
        <Card v-for="item in [
          { icon: Video, label: 'Live Sessions', desc: 'Daily live trading sessions with our analysts.' },
          { icon: TrendingUp, label: 'Today\'s Signals', desc: 'High-probability trade setups delivered daily.' },
          { icon: BookOpen, label: 'Learning Library', desc: 'Order flow, risk management, and more.' },
        ]" :key="item.label"
          class="relative overflow-hidden opacity-50 pointer-events-none select-none"
        >
          <div class="absolute top-3 right-3">
            <span class="text-[10px] font-semibold uppercase tracking-wider bg-muted text-muted-foreground px-2 py-0.5 rounded-full border border-border/60">
              Coming Soon
            </span>
          </div>
          <CardHeader class="pb-2 pr-24">
            <CardTitle class="text-sm flex items-center gap-2">
              <component :is="item.icon" class="size-4 text-muted-foreground" />
              {{ item.label }}
            </CardTitle>
          </CardHeader>
          <CardContent>
            <p class="text-xs text-muted-foreground">{{ item.desc }}</p>
          </CardContent>
        </Card>
      </div>
    </template>

    <template v-else-if="hasNoPlan">
      <Card class="mb-6">
        <CardContent class="pt-6">
          <div class="flex items-start gap-4">
            <div class="flex h-10 w-10 shrink-0 items-center justify-center rounded-full bg-primary/10">
              <Zap class="size-5 text-primary" />
            </div>
            <div>
              <p class="font-semibold text-base">Welcome, {{ pageUser.user.name }} 👋</p>
              <p class="text-sm text-muted-foreground mt-1">
                You're registered but don't have an active subscription yet.
                Activate a plan to unlock live sessions, daily signals, and the trading community.
              </p>
              <a href="/pricing" class="inline-block mt-4">
                <Button>View Plans & Activate</Button>
              </a>
            </div>
          </div>
        </CardContent>
      </Card>

      <p class="text-xs text-muted-foreground uppercase tracking-widest font-medium mb-3 px-0.5">
        Unlock with a plan
      </p>
      <div class="grid sm:grid-cols-2 lg:grid-cols-3 gap-4">
        <Card v-for="item in [
          { icon: Video, label: 'Live Sessions', desc: 'Daily live trading sessions with our analysts.' },
          { icon: TrendingUp, label: 'Today\'s Signals', desc: 'High-probability trade setups delivered daily.' },
          { icon: BookOpen, label: 'Learning Library', desc: 'Order flow, risk management, and more.' },
        ]" :key="item.label"
          class="opacity-40 pointer-events-none select-none"
        >
          <CardHeader class="pb-2">
            <CardTitle class="text-sm flex items-center gap-2">
              <component :is="item.icon" class="size-4 text-muted-foreground" />
              {{ item.label }}
              <Lock class="size-3 ml-auto text-muted-foreground/60" />
            </CardTitle>
          </CardHeader>
          <CardContent>
            <p class="text-xs text-muted-foreground">{{ item.desc }}</p>
          </CardContent>
        </Card>
      </div>
    </template>

  </AppLayout>
</template>