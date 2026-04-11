<script setup lang="ts">
import AppLayout from "@/layouts/AppLayout.vue";
import { Card, CardContent, CardHeader, CardTitle, CardDescription } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Badge } from "@/components/ui/badge";
import { Separator } from "@/components/ui/separator";
import { Input } from "@/components/ui/input";
import { KeyRound, Eye, EyeOff, ShieldCheck, CircleAlert, CalendarClock, Crown } from "lucide-vue-next";
import { ref } from "vue";

const showNew = ref(false);
const showConfirm = ref(false);

const newPassword = ref("");
const confirmPassword = ref("");

const passwordError = ref("");
const passwordSuccess = ref("");

const submitPassword = () => {
  passwordError.value = "";
  passwordSuccess.value = "";

  if (!newPassword.value || !confirmPassword.value) {
    passwordError.value = "All fields are required.";
    return;
  }
  if (newPassword.value.length < 8) {
    passwordError.value = "Password must be at least 8 characters.";
    return;
  }
  if (newPassword.value !== confirmPassword.value) {
    passwordError.value = "Passwords do not match.";
    return;
  }

  // TODO: wire up API
  passwordSuccess.value = "Password updated successfully.";
  newPassword.value = "";
  confirmPassword.value = "";
};

const getInitials = (name: string) =>
  name.split(" ").map((n) => n[0]).join("").toUpperCase().slice(0, 2);
</script>

<template>
  <AppLayout title="Profile">
    <template #default="{ user }">
      <div class="grid lg:grid-cols-3 gap-6">

        <!-- Left Column -->
        <div class="lg:col-span-2 space-y-6">

          <!-- Account Details -->
          <Card class="overflow-hidden">
            <div class="h-20 bg-gradient-to-r from-primary/20 via-primary/10 to-transparent" />
            <CardContent class="pt-0 px-6 pb-6 -mt-8 space-y-4">
              <div class="flex items-end gap-4">
                <div class="size-16 rounded-xl bg-primary text-primary-foreground flex items-center justify-center text-xl font-bold shadow-md ring-4 ring-background">
                  {{ getInitials(user.name) }}
                </div>
                <div class="pb-1">
                  <p class="text-lg font-semibold leading-tight">{{ user.name }}</p>
                  <p class="text-sm text-muted-foreground">{{ user.email }}</p>
                </div>
              </div>

              <p class="text-xs text-muted-foreground flex items-center gap-1.5 pt-1">
                <CircleAlert class="size-3.5 shrink-0" />
                To update your details, please contact support.
              </p>
            </CardContent>
          </Card>

          <!-- Change Password -->
          <Card>
            <CardHeader>
              <div class="flex items-center gap-3">
                <div class="size-9 rounded-lg bg-muted flex items-center justify-center">
                  <KeyRound class="size-4 text-muted-foreground" />
                </div>
                <div>
                  <CardTitle class="text-base">Change Password</CardTitle>
                  <CardDescription class="text-xs">Set a new password for your account.</CardDescription>
                </div>
              </div>
            </CardHeader>

            <CardContent class="space-y-4">
              <div class="grid md:grid-cols-2 gap-4">
                <div class="space-y-1.5">
                  <label class="text-xs font-medium text-muted-foreground uppercase tracking-wide">New Password</label>
                  <div class="relative">
                    <Input :type="showNew ? 'text' : 'password'" v-model="newPassword" placeholder="Min. 8 characters" class="pr-10 text-sm" />
                    <button type="button" class="absolute right-3 top-1/2 -translate-y-1/2 text-muted-foreground hover:text-foreground transition" @click="showNew = !showNew">
                      <EyeOff v-if="showNew" class="size-4" />
                      <Eye v-else class="size-4" />
                    </button>
                  </div>
                </div>

                <div class="space-y-1.5">
                  <label class="text-xs font-medium text-muted-foreground uppercase tracking-wide">Confirm Password</label>
                  <div class="relative">
                    <Input :type="showConfirm ? 'text' : 'password'" v-model="confirmPassword" placeholder="Repeat new password" class="pr-10 text-sm" />
                    <button type="button" class="absolute right-3 top-1/2 -translate-y-1/2 text-muted-foreground hover:text-foreground transition" @click="showConfirm = !showConfirm">
                      <EyeOff v-if="showConfirm" class="size-4" />
                      <Eye v-else class="size-4" />
                    </button>
                  </div>
                </div>
              </div>

              <p v-if="passwordError" class="text-sm text-destructive flex items-center gap-1.5">
                <CircleAlert class="size-4 shrink-0" /> {{ passwordError }}
              </p>
              <p v-if="passwordSuccess" class="text-sm text-green-500 flex items-center gap-1.5">
                <ShieldCheck class="size-4 shrink-0" /> {{ passwordSuccess }}
              </p>

              <div class="flex justify-end pt-1">
                <Button class="gap-2" @click="submitPassword">
                  <KeyRound class="size-4" />
                  Update Password
                </Button>
              </div>
            </CardContent>
          </Card>

        </div>

        <!-- Membership Card -->
        <div class="space-y-4">
          <Card>
            <CardHeader>
              <div class="flex items-center gap-2">
                <Crown class="size-4 text-primary" />
                <CardTitle class="text-base">Membership</CardTitle>
              </div>
              <CardDescription class="text-xs">Your current plan and status.</CardDescription>
            </CardHeader>

            <CardContent class="space-y-4">
              <div class="flex items-center justify-between p-3 rounded-lg bg-muted/40 border border-border/50">
                <span class="text-sm text-muted-foreground">Account Status</span>
                <Badge :variant="user.status === 'active' ? 'default' : user.status === 'pending' ? 'secondary' : 'destructive'">
                  {{ user.status.toUpperCase() }}
                </Badge>
              </div>

              <div class="flex items-center justify-between p-3 rounded-lg bg-muted/40 border border-border/50">
                <span class="text-sm text-muted-foreground">Current Plan</span>
                <Badge variant="outline">
                  {{ user.plan === 'none' ? 'No Active Plan' : user.plan.charAt(0).toUpperCase() + user.plan.slice(1) }}
                </Badge>
              </div>

              <div v-if="user.status === 'active' && user.expires_at" class="flex items-center gap-2 text-sm text-muted-foreground px-1">
                <CalendarClock class="size-4 shrink-0" />
                Expires: <span class="text-foreground font-medium">{{ user.expires_at }}</span>
              </div>

              <div v-else-if="user.status === 'pending'" class="text-sm text-muted-foreground flex items-center gap-2 px-1">
                <CircleAlert class="size-4 shrink-0 text-yellow-500" />
                Your payment is under review.
              </div>

              <div v-else-if="user.status === 'blocked'" class="text-sm text-destructive flex items-center gap-2 px-1">
                <CircleAlert class="size-4 shrink-0" />
                Account restricted. Contact support.
              </div>

              <Separator />

              <Button class="w-full" variant="secondary" @click="window.location.href = '/pricing'">
                Manage Plan
              </Button>
            </CardContent>
          </Card>
        </div>

      </div>
    </template>
  </AppLayout>
</template>