<script setup lang="ts">
import { computed, ref, onMounted } from "vue";
import AppLayout from "@/layouts/AppLayout.vue";
import { Card, CardContent, CardHeader, CardTitle, CardDescription } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Badge } from "@/components/ui/badge";
import { Separator } from "@/components/ui/separator";
import api from "@/lib/axios";

type PaymentStatus = "approved" | "pending" | "rejected";

interface Payment {
  id?: number;
  date: string;
  plan: string;
  amount: string;
  status: PaymentStatus;
}

const pageUser  = ref<any>(null);
const isLoading = ref(true);
const payments  = ref<Payment[]>([]);

// ── Helpers ───────────────────────────────────────────────
function friendlyDate(dateStr: string | null | undefined): string {
  if (!dateStr) return "—";
  return new Date(dateStr).toLocaleDateString("en-US", {
    year: "numeric", month: "long", day: "numeric",
  });
}

// ── Invoice number: MS-{USERID}-{PLANCODE}{AMOUNT}-{YYYYMMDD} ──
function generateInvoiceNumber(): string {
  const userId  = String(pageUser.value?.user?.id ?? 0).padStart(5, "0");
  const plan    = (pageUser.value?.plan ?? "basic").toLowerCase();
  const amount  = (payments.value[0]?.amount ?? "$0").replace(/\D/g, "");
  const dateStr = payments.value[0]?.date ?? new Date().toISOString().split("T")[0];

  const planCode: Record<string, string> = {
    basic: "BA", premium: "PR", enterprise: "EN",
  };

  return `MS-${userId}-${planCode[plan] ?? "XX"}${amount}-${dateStr.replace(/-/g, "")}`;
}

// ── Build invoice HTML string ─────────────────────────────
function buildInvoiceHtml(invoiceNo: string): string {
  const payment   = payments.value[0];
  const userName  = pageUser.value?.user?.name  ?? "Customer";
  const userEmail = pageUser.value?.user?.email ?? "";
  const today     = friendlyDate(new Date().toISOString().split("T")[0]);
  const paidDate  = friendlyDate(payment?.date);
  const renewDate = expiresAt.value ? friendlyDate(expiresAt.value) : "—";
  const plan      = payment?.plan ?? "—";
  const amount    = payment?.amount ?? "—";

  return `<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <title>Invoice ${invoiceNo} — Market Sharks</title>
  <style>
    @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap');
    * { margin:0; padding:0; box-sizing:border-box; }
    body { font-family:'Inter',sans-serif; background:#fff; color:#111; padding:56px; font-size:14px; }
    .header { display:flex; justify-content:space-between; align-items:flex-start; margin-bottom:48px; }
    .brand-name { font-size:20px; font-weight:700; letter-spacing:-0.5px; }
    .brand-name span { color:#f97316; }
    .brand-url { font-size:12px; color:#999; margin-top:2px; }
    .invoice-label { text-align:right; }
    .invoice-label h1 { font-size:32px; font-weight:800; letter-spacing:-1.5px; }
    .invoice-label .inv-number { font-size:13px; color:#f97316; font-weight:600; margin-top:4px; }
    .invoice-label .inv-date { font-size:12px; color:#999; margin-top:2px; }
    .status-pill { display:inline-flex; align-items:center; gap:6px; background:#fff7ed;
      border:1px solid #fed7aa; color:#c2410c; font-size:11px; font-weight:600;
      padding:4px 12px; border-radius:99px; text-transform:uppercase; letter-spacing:0.5px; margin-bottom:32px; }
    .dot { width:6px; height:6px; background:#f97316; border-radius:50%; }
    hr { border:none; border-top:1px solid #f0f0f0; margin:32px 0; }
    .bill-grid { display:grid; grid-template-columns:1fr 1fr 1fr; gap:32px; margin-bottom:40px; }
    .bill-block h4 { font-size:10px; text-transform:uppercase; letter-spacing:1.5px;
      color:#aaa; font-weight:600; margin-bottom:8px; }
    .bill-block p { font-size:14px; color:#111; line-height:1.7; }
    table { width:100%; border-collapse:collapse; margin-bottom:24px; }
    th { font-size:10px; text-transform:uppercase; letter-spacing:1px; color:#999;
      font-weight:600; padding:12px 16px; text-align:left; background:#fafafa; }
    th:last-child { text-align:right; }
    td { padding:16px; border-bottom:1px solid #f5f5f5; font-size:14px; vertical-align:top; }
    td:last-child { text-align:right; font-weight:600; }
    .desc-sub { font-size:12px; color:#999; margin-top:3px; }
    .totals { display:flex; justify-content:flex-end; }
    .totals-box { min-width:260px; background:#fafafa; border-radius:12px; padding:20px 24px; }
    .total-line { display:flex; justify-content:space-between; font-size:13px; color:#666; padding:4px 0; }
    .total-line.grand { font-size:16px; font-weight:700; color:#111;
      border-top:2px solid #f0f0f0; margin-top:12px; padding-top:14px; }
    .notes { margin-top:40px; padding:20px 24px; background:#fafafa; border-radius:12px; }
    .notes h4 { font-size:10px; text-transform:uppercase; letter-spacing:1px;
      color:#aaa; font-weight:600; margin-bottom:8px; }
    .notes p { font-size:13px; color:#666; line-height:1.7; }
    .footer { margin-top:48px; padding-top:24px; border-top:1px solid #f0f0f0;
      display:flex; justify-content:space-between; align-items:center; }
    .footer-left { font-size:12px; color:#aaa; line-height:1.8; }
    .footer-right { font-size:12px; color:#aaa; text-align:right; }
    .footer-right a { color:#f97316; text-decoration:none; font-weight:500; }
  </style>
</head>
<body>
  <div class="header">
    <div>
      <div class="brand-name">Market <span>Sharks</span></div>
      <div class="brand-url">marketsharks.live</div>
    </div>
    <div class="invoice-label">
      <h1>Invoice</h1>
      <div class="inv-number">#${invoiceNo}</div>
      <div class="inv-date">Issued: ${today}</div>
    </div>
  </div>

  <div class="status-pill"><span class="dot"></span> Payment Confirmed</div>

  <hr />

  <div class="bill-grid">
    <div class="bill-block">
      <h4>Billed To</h4>
      <p><strong>${userName}</strong><br><span style="color:#999">${userEmail}</span></p>
    </div>
    <div class="bill-block">
      <h4>Payment Date</h4><p>${paidDate}</p>
      <br>
      <h4>Next Renewal</h4><p>${renewDate}</p>
    </div>
    <div class="bill-block">
      <h4>Invoice Number</h4>
      <p style="color:#f97316;font-weight:600">${invoiceNo}</p>
      <br>
      <h4>Status</h4>
      <p style="color:#16a34a;font-weight:600">✓ Paid</p>
    </div>
  </div>

  <hr />

  <table>
    <thead>
      <tr><th>Description</th><th>Plan</th><th>Period</th><th>Amount</th></tr>
    </thead>
    <tbody>
      <tr>
        <td>Market Sharks Subscription<div class="desc-sub">Access to all ${plan} plan features</div></td>
        <td>${plan}</td>
        <td>${paidDate}<div class="desc-sub">→ ${renewDate}</div></td>
        <td>${amount}</td>
      </tr>
    </tbody>
  </table>

  <div class="totals">
    <div class="totals-box">
      <div class="total-line"><span>Subtotal</span><span>${amount}</span></div>
      <div class="total-line"><span>Tax (0%)</span><span>$0.00</span></div>
      <div class="total-line grand"><span>Total Paid</span><span>${amount}</span></div>
    </div>
  </div>

  <div class="notes">
    <h4>Notes</h4>
    <p>Thank you for subscribing to Market Sharks. This invoice confirms your payment.
    Keep this receipt for your records. For billing queries contact
    <strong>support@marketsharks.live</strong>.</p>
  </div>

  <div class="footer">
    <div class="footer-left">Market Sharks Ltd.<br>support@marketsharks.live</div>
    <div class="footer-right">
      <a href="https://marketsharks.live">marketsharks.live</a><br>
      Invoice ${invoiceNo}
    </div>
  </div>
</body>
</html>`;
}

// ── Direct download (no popup) ────────────────────────────
function downloadInvoice() {
  const payment = payments.value[0];
  if (!payment) return;

  const invoiceNo = generateInvoiceNumber();
  const html      = buildInvoiceHtml(invoiceNo);
  const blob      = new Blob([html], { type: "text/html" });
  const url       = URL.createObjectURL(blob);
  const a         = document.createElement("a");
  a.href          = url;
  a.download      = `MarketSharks-Invoice-${invoiceNo}.html`;
  a.click();
  URL.revokeObjectURL(url);
}

// ── Single onMounted ──────────────────────────────────────
onMounted(async () => {
  try {
    const { data } = await api.get("/api/me");
    pageUser.value = data;

    if (data.payment_request_status && data.plan) {
      let paidOn = new Date().toISOString().split("T")[0];
      if (data.expires_at) {
        const d = new Date(data.expires_at);
        d.setMonth(d.getMonth() - 1);
        paidOn = d.toISOString().split("T")[0];
      }

      payments.value = [
        {
          date:   paidOn,
          plan:   data.plan.charAt(0).toUpperCase() + data.plan.slice(1),
          amount: data.plan === "basic" ? "$25" : data.plan === "premium" ? "$60" : "$100",
          status: data.payment_request_status,
        },
      ];
    }
  } catch {
    window.location.href = "/login";
  } finally {
    isLoading.value = false;
  }
});

// ── Computed ──────────────────────────────────────────────
const userStatus           = computed(() => pageUser.value?.user?.status ?? "pending");
const subscriptionStatus   = computed(() => pageUser.value?.user?.subscription_status ?? "none");
const currentPlan          = computed(() => pageUser.value?.plan ?? null);
const expiresAt            = computed(() => pageUser.value?.expires_at ?? null);
const paymentRequestStatus = computed(() => pageUser.value?.payment_request_status ?? null);

const hasApprovedSubscription = computed(() =>
  userStatus.value === "active" && !!currentPlan.value && !!expiresAt.value
);
const isPaymentReview = computed(() =>
  userStatus.value === "payment_review" || paymentRequestStatus.value === "pending"
);
const isRejected  = computed(() =>
  userStatus.value === "rejected" || paymentRequestStatus.value === "rejected"
);
const isBlocked   = computed(() => userStatus.value === "blocked");
const isPending   = computed(() =>
  userStatus.value === "pending" && !paymentRequestStatus.value && subscriptionStatus.value === "none"
);
const isExpiringSoon = computed(() => {
  if (!expiresAt.value) return false;
  const diff = (new Date(expiresAt.value).getTime() - Date.now()) / (1000 * 60 * 60 * 24);
  return diff >= 0 && diff <= 7;
});

const goToPricing  = () => { window.location.href = "/pricing"; };
const goToCheckout = () => { window.location.href = `/checkout?plan=${currentPlan.value || "basic"}`; };
</script>

<template>
  <div v-if="isLoading" class="min-h-screen flex items-center justify-center">
    <p class="text-muted-foreground text-sm">Loading...</p>
  </div>

  <AppLayout v-else-if="pageUser" title="Billing">
    <template #default="{ user }">

      <!-- ── Current Plan ── -->
      <Card class="mb-6">
        <CardHeader>
          <CardTitle>Current Plan</CardTitle>
          <CardDescription>
            Manage your subscription and renewal details.
          </CardDescription>
        </CardHeader>

        <CardContent class="space-y-4">

          <!-- Active -->
          <div v-if="hasApprovedSubscription" class="flex items-center justify-between flex-wrap gap-4">
            <div class="flex items-start gap-3">
              <!-- Plan icon -->
              <div class="mt-0.5 flex h-9 w-9 shrink-0 items-center justify-center rounded-lg bg-primary/10">
                <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24"
                  fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"
                  class="text-primary">
                  <path d="M12 2l3.09 6.26L22 9.27l-5 4.87 1.18 6.88L12 17.77l-6.18 3.25L7 14.14 2 9.27l6.91-1.01L12 2z"/>
                </svg>
              </div>
              <div>
                <p class="text-lg font-semibold leading-tight">
                  {{ currentPlan.charAt(0).toUpperCase() + currentPlan.slice(1) }} Plan
                  <span class="text-sm font-normal text-muted-foreground ml-1">
                    · {{ currentPlan === 'basic' ? '$25' : currentPlan === 'premium' ? '$60' : '$100' }}/mo
                  </span>
                </p>
                <p class="text-sm text-muted-foreground flex items-center gap-1 mt-0.5">
                  <!-- Calendar icon -->
                  <svg xmlns="http://www.w3.org/2000/svg" width="13" height="13" viewBox="0 0 24 24"
                    fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                    <rect width="18" height="18" x="3" y="4" rx="2" ry="2"/>
                    <line x1="16" x2="16" y1="2" y2="6"/><line x1="8" x2="8" y1="2" y2="6"/>
                    <line x1="3" x2="21" y1="10" y2="10"/>
                  </svg>
                  Renews on {{ friendlyDate(expiresAt) }}
                </p>
              </div>
            </div>

            <div class="flex gap-3 items-center">
              <Badge variant="default">Active</Badge>
              <Button v-if="isExpiringSoon" variant="secondary" @click="goToCheckout">
                Renew Plan
              </Button>
            </div>
          </div>

          <!-- Pending -->
          <div v-else-if="isPending" class="space-y-3">
            <Badge variant="secondary">Pending</Badge>
            <p class="text-sm text-muted-foreground">
              You don't have an active subscription yet. Submit payment to activate your plan.
            </p>
            <div><Button @click="goToPricing">Submit Payment</Button></div>
          </div>

          <!-- Payment Review -->
          <div v-else-if="isPaymentReview" class="space-y-3">
            <Badge variant="secondary">Payment Under Review</Badge>
            <p class="text-sm text-muted-foreground">
              Your payment request has been submitted and is currently being reviewed by our team.
            </p>
            <div><Button disabled>Payment Under Review</Button></div>
          </div>

          <!-- Rejected -->
          <div v-else-if="isRejected" class="space-y-3">
            <Badge variant="destructive">Rejected</Badge>
            <p class="text-sm text-muted-foreground">
              Your payment request was rejected. Please resubmit your payment with valid proof.
            </p>
            <div><Button variant="destructive" @click="goToCheckout">Resubmit Payment</Button></div>
          </div>

          <!-- Blocked -->
          <div v-else-if="isBlocked" class="space-y-3">
            <Badge variant="destructive">Account Suspended</Badge>
            <p class="text-sm text-muted-foreground">
              Your account is suspended. Please contact support for assistance.
            </p>
            <div><Button variant="destructive" disabled>Account Suspended</Button></div>
          </div>

        </CardContent>
      </Card>

      <!-- ── Payment History ── -->
      <Card>
        <CardHeader>
          <CardTitle>Payment History</CardTitle>
          <CardDescription>View your past transactions.</CardDescription>
        </CardHeader>

        <CardContent>
          <div class="space-y-3">

            <div v-if="payments.length === 0" class="text-sm text-muted-foreground py-2">
              No payment history yet.
            </div>

            <!-- Payment row -->
            <div
              v-for="payment in payments"
              :key="`${payment.date}-${payment.plan}-${payment.status}`"
              class="flex items-center justify-between p-4 rounded-xl bg-muted gap-3"
            >
              <!-- Left: icon + details -->
              <div class="flex items-center gap-3 min-w-0">
                <div class="flex h-8 w-8 shrink-0 items-center justify-center rounded-md bg-background/60">
                  <svg xmlns="http://www.w3.org/2000/svg" width="15" height="15" viewBox="0 0 24 24"
                    fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"
                    class="text-muted-foreground">
                    <path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"/>
                    <polyline points="14 2 14 8 20 8"/>
                    <line x1="16" x2="8" y1="13" y2="13"/><line x1="16" x2="8" y1="17" y2="17"/>
                    <polyline points="10 9 9 9 8 9"/>
                  </svg>
                </div>
                <div>
                  <p class="font-medium text-sm leading-tight">
                    {{ payment.plan }} Plan
                    <span class="text-muted-foreground font-normal">— {{ payment.amount }}</span>
                  </p>
                  <p class="text-xs text-muted-foreground mt-0.5">{{ friendlyDate(payment.date) }}</p>
                </div>
              </div>

              <!-- Right: status badge -->
              <Badge
                :variant="
                  payment.status === 'approved' ? 'default'
                  : payment.status === 'pending' ? 'secondary'
                  : 'destructive'
                "
                class="shrink-0"
              >
                {{ payment.status.toUpperCase() }}
              </Badge>
            </div>

            <!-- Download Invoice -->
            <template v-if="hasApprovedSubscription && payments.length > 0">
              <Separator class="mt-1" />
              <div class="flex justify-end pt-1">
                <Button variant="secondary" @click="downloadInvoice" class="gap-2">
                  <svg xmlns="http://www.w3.org/2000/svg" width="14" height="14" viewBox="0 0 24 24"
                    fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                    <path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"/>
                    <polyline points="7 10 12 15 17 10"/>
                    <line x1="12" x2="12" y1="15" y2="3"/>
                  </svg>
                  Download Invoice
                </Button>
              </div>
            </template>

          </div>
        </CardContent>
      </Card>

    </template>
  </AppLayout>
</template>