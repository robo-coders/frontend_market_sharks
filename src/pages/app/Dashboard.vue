<script setup lang="ts">
import AppLayout from "@/layouts/AppLayout.vue";

import {
  Card,
  CardContent,
  CardHeader,
  CardTitle,
  CardDescription,
} from "@/components/ui/card";

import { Button } from "@/components/ui/button";
import { Badge } from "@/components/ui/badge";

import { TrendingUp, Video, BookOpen, MessageCircle } from "lucide-vue-next";

/**
 * Dummy dashboard data
 */
const stats = {
  liveNow: true,
  nextSession: "18:00 UTC",
  todaySignals: 4,
  communityMembers: 312,
};

const pageUser = {
  name: "Ahmed",
  email: "ahmed@example.com",
  status: "active" as const, // try: pending | blocked
  plan: "premium",
  expires_at: "2026-03-25",
};
</script>

<template>
  <AppLayout title="Dashboard" :user="pageUser">
    <template #default="{ user }">
      <!-- Welcome Card -->
      <Card class="mb-6">
        <CardHeader>
          <CardTitle>Welcome back, {{ user.name }} 👋</CardTitle>
          <CardDescription>Here’s your current account overview.</CardDescription>
        </CardHeader>

        <CardContent class="space-y-3">
          <div class="flex flex-wrap items-center gap-3">
            <Badge variant="default">
              Plan: {{ user.plan.charAt(0).toUpperCase() + user.plan.slice(1) }}
            </Badge>

            <span v-if="user.status === 'active'" class="text-sm text-muted-foreground">
              Expires: {{ user.expires_at }}
            </span>
          </div>

          <div v-if="user.status === 'pending'" class="p-4 rounded-xl bg-muted text-sm">
            Your payment is under review. You’ll receive full access once approved.
          </div>

          <div v-if="user.status === 'blocked'" class="p-4 rounded-xl bg-destructive/10 text-destructive text-sm">
            Your account has been suspended. Please contact support.
          </div>
        </CardContent>
      </Card>

      <!-- Quick Actions -->
      <div class="grid md:grid-cols-2 lg:grid-cols-4 gap-6 mb-6">
        <Card>
          <CardHeader>
            <CardTitle class="flex items-center gap-2">
              <Video class="size-5 text-primary" />
              Live Session
            </CardTitle>
          </CardHeader>
          <CardContent class="space-y-3">
            <div v-if="stats.liveNow" class="text-sm">🔴 Live now</div>
            <div v-else class="text-sm text-muted-foreground">Next: {{ stats.nextSession }}</div>
            <Button class="w-full" :disabled="user.status !== 'active'">Join Now</Button>
          </CardContent>
        </Card>

        <Card>
          <CardHeader>
            <CardTitle class="flex items-center gap-2">
              <TrendingUp class="size-5 text-primary" />
              Today’s Signals
            </CardTitle>
          </CardHeader>
          <CardContent class="space-y-3">
            <div class="text-sm">{{ stats.todaySignals }} high-probability setups today.</div>
            <Button variant="secondary" class="w-full" :disabled="user.status !== 'active'">
              View Signals
            </Button>
          </CardContent>
        </Card>

        <Card>
          <CardHeader>
            <CardTitle class="flex items-center gap-2">
              <BookOpen class="size-5 text-primary" />
              Learning Content
            </CardTitle>
          </CardHeader>
          <CardContent class="space-y-3">
            <div class="text-sm text-muted-foreground">
              Premium lessons, order flow, and risk management.
            </div>
            <Button variant="secondary" class="w-full" :disabled="user.status !== 'active'">
              Access Library
            </Button>
          </CardContent>
        </Card>

        <Card>
          <CardHeader>
            <CardTitle class="flex items-center gap-2">
              <MessageCircle class="size-5 text-primary" />
              Community
            </CardTitle>
          </CardHeader>
          <CardContent class="space-y-3">
            <div class="text-sm">{{ stats.communityMembers }} traders inside.</div>
            <Button variant="secondary" class="w-full" :disabled="user.status !== 'active'">
              Join WhatsApp
            </Button>
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
          <div v-if="user.status === 'active'" class="flex items-center justify-between flex-wrap gap-4">
            <div>
              <p class="font-medium">Premium Plan</p>
              <p class="text-sm text-muted-foreground">Renews on {{ user.expires_at }}</p>
            </div>
            <Button variant="secondary">Upgrade Plan</Button>
          </div>

          <div v-else-if="user.status === 'pending'" class="text-sm">
            Payment submitted and awaiting approval.
          </div>

          <div v-else class="text-sm text-destructive">
            Account access restricted.
          </div>
        </CardContent>
      </Card>
    </template>
  </AppLayout>
</template>
