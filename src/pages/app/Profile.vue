<script setup lang="ts">
import AppLayout from "@/layouts/AppLayout.vue";

import { Card, CardContent, CardHeader, CardTitle, CardDescription } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Badge } from "@/components/ui/badge";
import { Separator } from "@/components/ui/separator";
import { Input } from "@/components/ui/input";

import { LogOut, Save } from "lucide-vue-next";

/* ---------------- DUMMY USER ---------------- */
const user = {
  name: "Ahmed",
  email: "ahmed@example.com",
  whatsapp: "+971 50 123 4567",
  status: "active", // pending | active | blocked
  plan: "premium",
  expires_at: "2026-03-25",
};
</script>

<template>
  <AppLayout title="Profile" :user="user">
    <template #default="{ user }">
      <div class="grid lg:grid-cols-3 gap-6">

        <!-- Profile Card -->
        <Card class="lg:col-span-2">
          <CardHeader>
            <CardTitle>Account Details</CardTitle>
            <CardDescription>
              Update your profile information.
            </CardDescription>
          </CardHeader>

          <CardContent class="space-y-5">
            <div class="grid md:grid-cols-2 gap-4">
              <div>
                <label class="text-sm text-muted-foreground">Full Name</label>
                <Input :model-value="user.name" disabled />
              </div>

              <div>
                <label class="text-sm text-muted-foreground">Email</label>
                <Input :model-value="user.email" disabled />
              </div>

              <div>
                <label class="text-sm text-muted-foreground">WhatsApp</label>
                <Input :model-value="user.whatsapp" placeholder="+971..." disabled />
              </div>

              <div>
                <label class="text-sm text-muted-foreground">Region</label>
                <Input model-value="UAE" disabled />
              </div>
            </div>

            <Separator />

            <div class="flex justify-end gap-2">
              <Button variant="secondary" class="gap-2" disabled>
                <Save class="size-4" />
                Save Changes
              </Button>

              <Button variant="ghost" class="gap-2" disabled>
                Change Password
              </Button>
            </div>

            <p class="text-xs text-muted-foreground">
              (Dummy UI) These fields will become editable after you connect real auth + API.
            </p>
          </CardContent>
        </Card>

        <!-- Status Card -->
        <Card>
          <CardHeader>
            <CardTitle>Membership</CardTitle>
            <CardDescription>
              Current status and plan.
            </CardDescription>
          </CardHeader>

          <CardContent class="space-y-4">
            <div class="flex flex-wrap gap-2">
              <Badge
                :variant="user.status === 'active' ? 'default' : user.status === 'pending' ? 'secondary' : 'destructive'"
              >
                {{ user.status.toUpperCase() }}
              </Badge>

              <Badge variant="outline">
                {{ user.plan.charAt(0).toUpperCase() + user.plan.slice(1) }}
              </Badge>
            </div>

            <div v-if="user.status === 'active'" class="text-sm text-muted-foreground">
              Expires: <span class="text-foreground font-medium">{{ user.expires_at }}</span>
            </div>

            <div v-else-if="user.status === 'pending'" class="text-sm text-muted-foreground">
              Your payment is under review.
            </div>

            <div v-else class="text-sm text-muted-foreground">
              Your account is restricted. Contact support.
            </div>

            <Separator />

            <Button class="w-full gap-2" variant="secondary" @click="window.location.href = '/pricing'">
              Manage Plan
            </Button>

            <Button
              class="w-full gap-2"
              variant="ghost"
              @click="window.location.href = '/login'"
            >
              <LogOut class="size-4" />
              Logout
            </Button>
          </CardContent>
        </Card>

      </div>
    </template>
  </AppLayout>
</template>
