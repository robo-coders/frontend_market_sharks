<script setup lang="ts">
import { computed, reactive, ref, onMounted } from "vue"
import { countries } from "countries-list"

import { Button } from "@/components/ui/button"
import { Check, Copy } from "lucide-vue-next"
import {
  Card,
  CardContent,
  CardDescription,
  CardHeader,
  CardTitle,
} from "@/components/ui/card"

type PlanSlug = "basic" | "premium" | "enterprise"
type PayGroup = "card" | "manual" | "crypto"
type ManualType = "bank" | "jazzcash" | "easypaisa"

interface Plan {
  slug: PlanSlug
  title: string
  price: number
  period: "month" | "quarter" | "hour"
  description: string
  benefitList: string[]
}

/* ---------------- PLANS ---------------- */
const plans: Plan[] = [
  {
    slug: "basic",
    title: "Basic",
    price: 25,
    period: "month",
    description: "Perfect for beginners seeking guided trades, live sessions, and essential market insights.",
    benefitList: [
      "Live 90-minute trading session",
      "Exclusive chart alerts",
      "Level 2 data insights",
      "Access to premium learning content",
    ],
  },
  {
    slug: "premium",
    title: "Premium",
    price: 60,
    period: "quarter",
    description: "Ideal for active traders who want full-day live trading, premium alerts, and deeper market analysis.",
    benefitList: [
      "Extended live trading sessions",
      "Premium chart alerts",
      "Advanced Level 2 data insights",
      "Full access to premium learning content",
    ],
  },
  {
    slug: "enterprise",
    title: "Enterprise",
    price: 100,
    period: "hour",
    description: "Built for serious traders who need complete access, advanced tools, and priority-level support.",
    benefitList: [
      "1:1 trading session (60 minutes)",
      "Strategy development",
      "Risk management planning",
      "Advanced Bookmap training",
      "Order flow concepts",
    ],
  },
]

/* ---------------- PLAN FROM URL ---------------- */
const getPlanFromUrl = (): PlanSlug | null => {
  const params = new URLSearchParams(window.location.search)
  const plan = (params.get("plan") || "").toLowerCase()
  if (plan === "basic" || plan === "premium" || plan === "enterprise") return plan
  return null
}

const selectedPlan = computed(() => {
  const slug = getPlanFromUrl()
  if (!slug) return null
  return plans.find((p) => p.slug === slug) || null
})

const planLabel = computed(() => {
  if (!selectedPlan.value) return ""
  return selectedPlan.value.period === "hour" ? "/hourly" : `/${selectedPlan.value.period}`
})

const changePlan = () => {
  window.location.href = "/#pricing"
}

/* ---------------- FORM ---------------- */
const form = reactive({
  firstName: "",
  lastName: "",
  nickname: "",
  isAnonymous: false,
  email: "",
  countryCode: "+971",
  whatsapp_number: "",
})

const errors = reactive({
  firstName: "",
  lastName: "",
  nickname: "",
  email: "",
  whatsapp_number: "",
  proof: "",
  cardNumber: "",
  cardName: "",
  cardExp: "",
  cardCvc: "",
})

const isLoading = ref(false)

/* ---------------- COUNTRIES ---------------- */
const countryOptions = computed(() => {
  const dn = new Intl.DisplayNames([navigator.language], { type: "region" })
  return Object.entries(countries)
    .map(([iso2, c]) => ({
      iso2,
      name: dn.of(iso2) || (c.name as string),
      dial: `+${c.phone}`,
    }))
    .filter((c) => c.dial !== "+undefined")
    .sort((a, b) => a.name.localeCompare(b.name))
})

onMounted(() => {
  const lang = navigator.language || ""
  const region = lang.includes("-") ? lang.split("-")[1].toUpperCase() : ""
  const match = countryOptions.value.find((c) => c.iso2 === region)
  form.countryCode = match?.dial || "+971"
})

/* ---------------- PAYMENT ---------------- */
const payGroup = ref<PayGroup>("card")
const manualType = ref<ManualType>("bank")
const proofFile = ref<File | null>(null)

const cardForm = reactive({
  number: "",
  name: "",
  exp: "",
  cvc: "",
})

const isProofRequired = computed(() => payGroup.value !== "card")

const bankDetails = reactive({
  bankName: "ABC Bank",
  accountName: "Market Sharks",
  iban: "XX00 0000 0000 0000 0000",
  referenceHint: "Use your email as reference",
})

const jazzCashDetails = reactive({ number: "+923027743700", name: "Ali Raza" })
const easyPaisaDetails = reactive({ number: "+923429883708", name: "Ali Raza" })

const btcDetails = reactive({ address: "bc1q0xtas7rdmenepy6h8pqmyn6835ewqxr4rrzztf", network: "BTC (Bitcoin)" })
const usdtDetails = reactive({ address: "TLUVYTLiKkQ6RpSD5dMFUNzuJ49aWHQTB1", network: "TRC20 (Tron)" })

const selectPayGroup = (g: PayGroup) => {
  payGroup.value = g
  errors.proof = ""
  proofFile.value = null
  errors.cardNumber = ""
  errors.cardName = ""
  errors.cardExp = ""
  errors.cardCvc = ""
}

/* ---------------- COPY FEEDBACK ---------------- */
const btcCopied = ref(false)
let btcCopyTimer: number | null = null

const copyBtc = async () => {
  await navigator.clipboard.writeText(btcDetails.address)
  btcCopied.value = true
  if (btcCopyTimer) clearTimeout(btcCopyTimer)
  btcCopyTimer = window.setTimeout(() => {
    btcCopied.value = false
    btcCopyTimer = null
  }, 1500)
}

const usdtCopied = ref(false)
let usdtCopyTimer: number | null = null

const copyUsdt = async () => {
  await navigator.clipboard.writeText(usdtDetails.address)
  usdtCopied.value = true
  if (usdtCopyTimer) clearTimeout(usdtCopyTimer)
  usdtCopyTimer = window.setTimeout(() => {
    usdtCopied.value = false
    usdtCopyTimer = null
  }, 1500)
}

const onProofChange = (e: Event) => {
  const input = e.target as HTMLInputElement
  proofFile.value = input.files?.[0] || null
  errors.proof = ""
}

/* ---------------- VALIDATION ---------------- */
const validate = () => {
  errors.firstName = ""
  errors.lastName = ""
  errors.nickname = ""
  errors.email = form.email.trim() ? "" : "Email is required"
  errors.whatsapp_number = form.whatsapp_number.trim() ? "" : "WhatsApp number is required"

  if (!form.isAnonymous) {
    errors.firstName = form.firstName.trim() ? "" : "First name is required"
    errors.lastName = form.lastName.trim() ? "" : "Last name is required"
  } else {
    errors.nickname = form.nickname.trim() ? "" : "Nickname is required"
  }

  errors.proof = isProofRequired.value && !proofFile.value ? "Payment proof is required" : ""

  if (payGroup.value === "card") {
    errors.cardNumber = cardForm.number.trim() ? "" : "Card number is required"
    errors.cardName = cardForm.name.trim() ? "" : "Name on card is required"
    errors.cardExp = cardForm.exp.trim() ? "" : "Expiry is required"
    errors.cardCvc = cardForm.cvc.trim() ? "" : "CVC is required"
  } else {
    errors.cardNumber = ""
    errors.cardName = ""
    errors.cardExp = ""
    errors.cardCvc = ""
  }

  return Object.values(errors).every((e) => !e)
}

/* ---------------- SUBMIT ---------------- */
const submitCheckout = async () => {
  if (!selectedPlan.value) return
  if (!validate()) return

  isLoading.value = true

  // ✅ Fix: map "crypto" → "btc" for backend
  const methodValue =
    payGroup.value === "manual"
      ? manualType.value
      : payGroup.value === "crypto"
        ? "btc"
        : payGroup.value

  const formData = new FormData()
  formData.append("plan", selectedPlan.value.slug)
  formData.append("first_name", form.isAnonymous ? "" : form.firstName)
  formData.append("last_name", form.isAnonymous ? "" : form.lastName)
  formData.append("nickname", form.isAnonymous ? form.nickname : "")
  formData.append("is_anonymous", form.isAnonymous ? "1" : "0")
  formData.append("email", form.email)
  formData.append("whatsapp_number", `${form.countryCode}${form.whatsapp_number}`)
  formData.append("payment_method", methodValue)
  if (proofFile.value) formData.append("proof", proofFile.value)

  try {
    const res = await fetch(`${import.meta.env.VITE_API_URL}/api/payment-requests`, {
      method: "POST",
      headers: { Accept: "application/json" },
      body: formData,
    })

    const json = await res.json()

    // ✅ Fix: stop the loop — only redirect on actual success
    if (!res.ok || !json.login_token) {
      console.error("Submission failed:", json)
      alert(json.message || "Something went wrong. Please try again.")
      return
    }

    localStorage.setItem("auth_token", json.login_token)
    window.location.href = "/app/dashboard"
  } catch (e) {
    console.error(e)
    alert("Network error. Please try again.")
  } finally {
    isLoading.value = false
  }
}
</script>

<template>
  <section class="container py-16">
    <div v-if="!selectedPlan" class="max-w-xl mx-auto text-center space-y-6">
      <h1 class="text-3xl md:text-4xl font-bold">Checkout</h1>
      <p class="text-muted-foreground">No plan selected.</p>
      <Button @click="changePlan">Go to Pricing</Button>
    </div>

    <div v-else class="max-w-2xl mx-auto">
      <Card>
        <CardHeader>
          <CardTitle class="text-2xl font-bold">Checkout</CardTitle>
          <CardDescription>We'll create your account after payment.</CardDescription>
        </CardHeader>

        <CardContent class="space-y-6">

          <!-- ORDER SUMMARY -->
          <div class="rounded-md border p-4">
            <div class="flex items-start justify-between gap-4">
              <div class="min-w-0">
                <div class="text-sm text-muted-foreground">Order summary</div>
                <div class="text-base font-semibold">
                  {{ selectedPlan.title }}
                  <span class="text-muted-foreground font-normal">
                    · ${{ selectedPlan.price }}{{ planLabel }}
                  </span>
                </div>
              </div>
              <button
                type="button"
                class="text-sm underline text-muted-foreground hover:text-foreground"
                @click="changePlan"
              >
                Change
              </button>
            </div>
            <div class="mt-3 grid grid-cols-2 gap-x-6 gap-y-2 text-sm">
              <div
                v-for="b in selectedPlan.benefitList"
                :key="b"
                class="flex items-start gap-2 text-muted-foreground"
              >
                <Check class="mt-0.5 h-4 w-4" />
                <span>{{ b }}</span>
              </div>
            </div>
          </div>

          <!-- 1) YOUR DETAILS -->
          <div class="rounded-md border p-4 space-y-4">
            <label class="flex items-center gap-3 p-3 border rounded-md cursor-pointer hover:bg-muted/30 transition">
              <input
                type="checkbox"
                v-model="form.isAnonymous"
                class="w-4 h-4 rounded text-primary focus:ring-primary"
              />
              <div>
                <span class="text-sm font-medium">Register anonymously</span>
                <p class="text-xs text-muted-foreground">Your name will not be shown publicly</p>
              </div>
            </label>
            <p class="text-sm font-medium">1) Your details</p>

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

            <div v-else class="space-y-2">
              <label class="text-sm font-medium">Nickname</label>
              <input
                v-model="form.nickname"
                class="w-full rounded-md border px-3 py-2 text-sm"
                placeholder="TraderJohn"
              />
              <p v-if="errors.nickname" class="text-sm text-destructive">{{ errors.nickname }}</p>
            </div>

            <div class="space-y-2">
              <label class="text-sm font-medium">Email</label>
              <input
                v-model="form.email"
                type="email"
                class="w-full rounded-md border px-3 py-2 text-sm"
                placeholder="you@email.com"
              />
              <p v-if="errors.email" class="text-sm text-destructive">{{ errors.email }}</p>
            </div>

            <div class="space-y-2">
              <label class="text-sm font-medium">WhatsApp number</label>
              <div class="grid grid-cols-3 gap-2 sm:flex sm:gap-2">
                <select
                  v-model="form.countryCode"
                  class="col-span-1 sm:w-56 sm:shrink-0 rounded-md border px-2 py-2 text-sm"
                >
                  <option v-for="c in countryOptions" :key="c.iso2" :value="c.dial">
                    {{ c.dial }} — {{ c.name }}
                  </option>
                </select>
                <input
                  v-model="form.whatsapp_number"
                  type="tel"
                  class="col-span-2 sm:min-w-0 sm:flex-1 rounded-md border px-3 py-2 text-sm"
                  placeholder="50xxxxxxx"
                />
              </div>
              <p v-if="errors.whatsapp_number" class="text-sm text-destructive">{{ errors.whatsapp_number }}</p>
            </div>
          </div>

          <!-- 2) PAYMENT METHOD -->
          <div class="rounded-md border p-4 space-y-3">
            <p class="text-sm font-medium">2) Payment method</p>
            <div class="grid gap-3 sm:grid-cols-3">
              <button
                type="button"
                @click="selectPayGroup('card')"
                :class="['text-left rounded-md border p-3 transition', payGroup === 'card' ? 'border-primary ring-1 ring-primary' : 'hover:bg-muted/40']"
              >
                <div class="text-sm font-medium">Card</div>
                <div class="text-xs text-muted-foreground">Instant</div>
              </button>
              <button
                type="button"
                @click="selectPayGroup('manual')"
                :class="['text-left rounded-md border p-3 transition', payGroup === 'manual' ? 'border-primary ring-1 ring-primary' : 'hover:bg-muted/40']"
              >
                <div class="text-sm font-medium">Manual</div>
                <div class="text-xs text-muted-foreground">Bank / JazzCash / EasyPaisa</div>
              </button>
              <button
                type="button"
                @click="selectPayGroup('crypto')"
                :class="['text-left rounded-md border p-3 transition', payGroup === 'crypto' ? 'border-primary ring-1 ring-primary' : 'hover:bg-muted/40']"
              >
                <div class="text-sm font-medium">Crypto</div>
                <div class="text-xs text-muted-foreground">BTC / USDT</div>
              </button>
            </div>
          </div>

          <!-- 3) COMPLETE PAYMENT -->
          <div class="rounded-md border p-4 space-y-3">
            <p class="text-sm font-medium">3) Complete payment</p>

            <!-- CARD -->
            <div v-if="payGroup === 'card'" class="space-y-4">
              <div class="grid gap-3 sm:grid-cols-2">
                <div class="sm:col-span-2 space-y-2">
                  <label class="text-sm font-medium">Card number</label>
                  <input v-model="cardForm.number" inputmode="numeric" autocomplete="cc-number" class="w-full rounded-md border px-3 py-2 text-sm" placeholder="1234 5678 9012 3456" />
                  <p v-if="errors.cardNumber" class="text-sm text-destructive">{{ errors.cardNumber }}</p>
                </div>
                <div class="sm:col-span-2 space-y-2">
                  <label class="text-sm font-medium">Name on card</label>
                  <input v-model="cardForm.name" autocomplete="cc-name" class="w-full rounded-md border px-3 py-2 text-sm" placeholder="John Doe" />
                  <p v-if="errors.cardName" class="text-sm text-destructive">{{ errors.cardName }}</p>
                </div>
                <div class="space-y-2">
                  <label class="text-sm font-medium">Expiry (MM/YY)</label>
                  <input v-model="cardForm.exp" inputmode="numeric" autocomplete="cc-exp" class="w-full rounded-md border px-3 py-2 text-sm" placeholder="MM/YY" />
                  <p v-if="errors.cardExp" class="text-sm text-destructive">{{ errors.cardExp }}</p>
                </div>
                <div class="space-y-2">
                  <label class="text-sm font-medium">CVC</label>
                  <input v-model="cardForm.cvc" inputmode="numeric" autocomplete="cc-csc" class="w-full rounded-md border px-3 py-2 text-sm" placeholder="123" />
                  <p v-if="errors.cardCvc" class="text-sm text-destructive">{{ errors.cardCvc }}</p>
                </div>
              </div>
              <Button class="w-full" :disabled="isLoading" @click="submitCheckout">
                Pay with Card
              </Button>
            </div>

            <!-- MANUAL -->
            <div v-else-if="payGroup === 'manual'" class="space-y-4">
              <div class="flex gap-2">
                <button type="button" @click="manualType = 'bank'" :class="['px-3 py-2 text-sm rounded-md border transition', manualType === 'bank' ? 'border-primary ring-1 ring-primary' : 'hover:bg-muted/40']">Bank</button>
                <button type="button" @click="manualType = 'jazzcash'" :class="['px-3 py-2 text-sm rounded-md border transition', manualType === 'jazzcash' ? 'border-primary ring-1 ring-primary' : 'hover:bg-muted/40']">JazzCash</button>
                <button type="button" @click="manualType = 'easypaisa'" :class="['px-3 py-2 text-sm rounded-md border transition', manualType === 'easypaisa' ? 'border-primary ring-1 ring-primary' : 'hover:bg-muted/40']">EasyPaisa</button>
              </div>

              <div v-if="manualType === 'bank'" class="text-sm space-y-1">
                <div><span class="font-medium">Bank:</span> {{ bankDetails.bankName }}</div>
                <div><span class="font-medium">Account:</span> {{ bankDetails.accountName }}</div>
                <div><span class="font-medium">IBAN:</span> {{ bankDetails.iban }}</div>
                <div class="text-muted-foreground">{{ bankDetails.referenceHint }}</div>
              </div>
              <div v-else-if="manualType === 'jazzcash'" class="text-sm space-y-1">
                <div><span class="font-medium">Send to:</span> {{ jazzCashDetails.number }}</div>
                <div><span class="font-medium">Name:</span> {{ jazzCashDetails.name }}</div>
              </div>
              <div v-else class="text-sm space-y-1">
                <div><span class="font-medium">Send to:</span> {{ easyPaisaDetails.number }}</div>
                <div><span class="font-medium">Name:</span> {{ easyPaisaDetails.name }}</div>
              </div>

              <div class="space-y-2 pt-2">
                <label class="text-sm font-medium">Upload payment proof (screenshot)</label>
                <input type="file" accept="image/*" class="block w-full text-sm" @change="onProofChange" />
                <p v-if="proofFile" class="text-sm text-muted-foreground">Selected: {{ proofFile.name }}</p>
                <p v-if="errors.proof" class="text-sm text-destructive">{{ errors.proof }}</p>
              </div>
              <Button class="w-full" :disabled="isLoading" @click="submitCheckout">Submit for Review</Button>
            </div>

            <!-- CRYPTO -->
            <div v-else class="space-y-4">

              <!-- BTC -->
              <div class="flex items-start justify-between gap-3">
                <div class="min-w-0">
                  <div class="text-sm font-medium">BTC Address</div>
                  <div class="text-xs text-muted-foreground mb-1">Network: {{ btcDetails.network }}</div>
                  <div class="text-sm text-muted-foreground break-all">{{ btcDetails.address }}</div>
                </div>
                <Button variant="secondary" size="sm" @click="copyBtc">
                  <template v-if="btcCopied">✓ Copied</template>
                  <template v-else><Copy class="w-4 h-4 mr-2" /> Copy</template>
                </Button>
              </div>

              <!-- USDT -->
              <div class="flex items-start justify-between gap-3">
                <div class="min-w-0">
                  <div class="text-sm font-medium">USDT Address</div>
                  <div class="text-xs text-muted-foreground mb-1">Network: {{ usdtDetails.network }}</div>
                  <div class="text-sm text-muted-foreground break-all">{{ usdtDetails.address }}</div>
                </div>
                <Button variant="secondary" size="sm" @click="copyUsdt">
                  <template v-if="usdtCopied">✓ Copied</template>
                  <template v-else><Copy class="w-4 h-4 mr-2" /> Copy</template>
                </Button>
              </div>

              <div class="space-y-2">
                <label class="text-sm font-medium">Upload payment proof (screenshot)</label>
                <input type="file" accept="image/*" class="block w-full text-sm" @change="onProofChange" />
                <p v-if="proofFile" class="text-sm text-muted-foreground">Selected: {{ proofFile.name }}</p>
                <p v-if="errors.proof" class="text-sm text-destructive">{{ errors.proof }}</p>
              </div>
              <Button class="w-full" :disabled="isLoading" @click="submitCheckout">Submit for Review</Button>
            </div>
          </div>

          <p class="text-sm text-muted-foreground">
            After payment, login details will be emailed to you.
          </p>

        </CardContent>
      </Card>
    </div>
  </section>
</template>