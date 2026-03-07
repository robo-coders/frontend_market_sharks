<script setup lang="ts">
import AppLayout from "@/layouts/AppLayout.vue";

import { Card, CardContent, CardHeader, CardTitle, CardDescription } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Badge } from "@/components/ui/badge";
import { Separator } from "@/components/ui/separator";

interface Payment {
  date: string;
  plan: string;
  amount: string;
  status: "approved" | "pending" | "rejected";
}

/* ---------------- DUMMY DATA ---------------- */

const user = {
  name: "Ahmed",
  status: "active", // change to: pending | blocked
  plan: "premium",
  expires_at: "2026-03-25",
};

const payments: Payment[] = [
  {
    date: "Jan 12, 2026",
    plan: "Premium",
    amount: "$60",
    status: "approved",
  },
  {
    date: "Oct 12, 2025",
    plan: "Premium",
    amount: "$60",
    status: "approved",
  },
  {
    date: "Jul 12, 2025",
    plan: "Premium",
    amount: "$60",
    status: "pending",
  },
];
</script>

<template>
  <AppLayout title="Billing" :user="user">
    <template #default="{ user }">

      <!-- Current Plan -->
      <Card class="mb-6">
        <CardHeader>
          <CardTitle>Current Plan</CardTitle>
          <CardDescription>
            Manage your subscription and renewal details.
          </CardDescription>
        </CardHeader>

        <CardContent class="space-y-4">

          <!-- Active -->
          <div v-if="user.status === 'active'" class="flex items-center justify-between flex-wrap gap-4">
            <div>
              <p class="text-lg font-semibold">
                {{ user.plan.charAt(0).toUpperCase() + user.plan.slice(1) }} Plan
              </p>
              <p class="text-sm text-muted-foreground">
                Renews on {{ user.expires_at }}
              </p>
            </div>

            <div class="flex gap-3">
              <Badge variant="default">Active</Badge>
              <Button variant="secondary">
                Upgrade Plan
              </Button>
            </div>
          </div>

          <!-- Pending -->
          <div v-if="user.status === 'pending'" class="space-y-3">
            <Badge variant="secondary">Pending Approval</Badge>
            <p class="text-sm text-muted-foreground">
              Your payment has been submitted and is under review.
            </p>
          </div>

          <!-- Blocked -->
          <div v-if="user.status === 'blocked'" class="space-y-3">
            <Badge variant="destructive">Account Suspended</Badge>
            <p class="text-sm text-muted-foreground">
              Please contact support for assistance.
            </p>
          </div>

        </CardContent>
      </Card>

      <!-- Payment History -->
      <Card>
        <CardHeader>
          <CardTitle>Payment History</CardTitle>
          <CardDescription>
            View your past transactions.
          </CardDescription>
        </CardHeader>

        <CardContent>

          <div class="space-y-4">

            <div
              v-for="payment in payments"
              :key="payment.date"
              class="flex items-center justify-between p-4 rounded-xl bg-muted"
            >
              <div>
                <p class="font-medium">
                  {{ payment.plan }} — {{ payment.amount }}
                </p>
                <p class="text-sm text-muted-foreground">
                  {{ payment.date }}
                </p>
              </div>

              <Badge
                :variant="
                  payment.status === 'approved'
                    ? 'default'
                    : payment.status === 'pending'
                    ? 'secondary'
                    : 'destructive'
                "
              >
                {{ payment.status.toUpperCase() }}
              </Badge>
            </div>

            <Separator />

            <div class="flex justify-end">
              <Button variant="secondary">
                Download Invoice
              </Button>
            </div>

          </div>

        </CardContent>
      </Card>

    </template>
  </AppLayout>
</template>
