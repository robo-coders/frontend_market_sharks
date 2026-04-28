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

interface MeResponse {
  user?: {
    id?: number
    name?: string
    email?: string
    status?: string
    subscription_status?: string
    whatsapp_number?: string
  }
  plan?: string | null
  expires_at?: string | null
  payment_request_status?: string | null
}

const plans: Plan[] = [
  {
    slug: "basic",
    title: "Basic",
    price: 25,
    period: "month",
    description:
      "Perfect for beginners seeking guided trades, live sessions, and essential market insights.",
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
    description:
      "Ideal for active traders who want full-day live trading, premium alerts, and deeper market analysis.",
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
    description:
      "Built for serious traders who need complete access, advanced tools, and priority-level support.",
    benefitList: [
      "1:1 trading session (60 minutes)",
      "Strategy development",
      "Risk management planning",
      "Advanced Bookmap training",
      "Order flow concepts",
    ],
  },
]

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
  general: "",
})

const isLoading = ref(false)
const isBooting = ref(true)
const success = ref("")
const me = ref<MeResponse | null>(null)

const payGroup = ref<PayGroup>("card")
const manualType = ref<ManualType>("bank")
const proofFile = ref<File | null>(null)

const cardForm = reactive({
  number: "",
  name: "",
  exp: "",
  cvc: "",
})

const bankDetails = reactive({
  bankName: "ABC Bank",
  accountName: "Market Sharks",
  iban: "XX00 0000 0000 0000 0000",
  referenceHint: "Use your email as reference",
})

const jazzCashDetails = reactive({
  number: "+923027743700",
  name: "Ali Raza",
})

const easyPaisaDetails = reactive({
  number: "+923429883708",
  name: "Ali Raza",
})

const btcDetails = reactive({
  address: "bc1q0xtas7rdmenepy6h8pqmyn6835ewqxr4rrzztf",
  network: "BTC (Bitcoin)",
})

const usdtDetails = reactive({
  address: "TLUVYTLiKkQ6RpSD5dMFUNzuJ49aWHQTB1",
  network: "TRC20 (Tron)",
})

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

const isLoggedIn = computed(() => !!me.value?.user?.email)

const hasActivePlan = computed(() => {
  const status = me.value?.user?.status
  const subscriptionStatus = me.value?.user?.subscription_status
  const plan = me.value?.plan
  return status === "active" && subscriptionStatus !== "none" && !!plan
})

const isPaymentReview = computed(() => {
  return (
    me.value?.user?.status === "payment_review" ||
    me.value?.payment_request_status === "pending"
  )
})

const isBlockedFromCheckout = computed(() => {
  return hasActivePlan.value || isPaymentReview.value
})

const blockTitle = computed(() => {
  if (hasActivePlan.value) return "You already have an active plan"
  if (isPaymentReview.value) return "Your payment is already under review"
  return ""
})

const blockDescription = computed(() => {
  if (hasActivePlan.value) {
    return "Only one plan can be active at a time right now. You can manage your current subscription from billing."
  }
  if (isPaymentReview.value) {
    return "Please wait for the current payment request to be reviewed before submitting another one."
  }
  return ""
})

const accountName = computed(() => {
  const fullName = me.value?.user?.name?.trim() || ""
  return fullName
})

const showDetailsStep = computed(() => !isLoggedIn.value)
const stepTwoLabel = computed(() => (showDetailsStep.value ? "2) Payment method" : "1) Payment method"))
const stepThreeLabel = computed(() => (showDetailsStep.value ? "3) Complete payment" : "2) Complete payment"))
const isProofRequired = computed(() => payGroup.value !== "card")

const changePlan = () => {
  window.location.href = "/#pricing"
}

const goToDashboard = () => {
  window.location.href = "/app/dashboard"
}

const goToBilling = () => {
  window.location.href = "/app/billing"
}

const splitName = (fullName: string) => {
  const parts = fullName.trim().split(/\s+/).filter(Boolean)
  return {
    firstName: parts[0] || "",
    lastName: parts.slice(1).join(" "),
  }
}

const parseWhatsapp = (value: string | undefined) => {
  if (!value) return
  const matchedCountry =
    countryOptions.value.find((c) => value.startsWith(c.dial)) || null

  if (matchedCountry) {
    form.countryCode = matchedCountry.dial
    form.whatsapp_number = value.slice(matchedCountry.dial.length)
    return
  }

  form.whatsapp_number = value.replace(/^\+/, "")
}

const setDefaultCountryCode = () => {
  const lang = navigator.language || ""
  const region = lang.includes("-") ? lang.split("-")[1].toUpperCase() : ""
  const match = countryOptions.value.find((c) => c.iso2 === region)
  form.countryCode = match?.dial || "+971"
}

const hydrateLoggedInUser = (data: MeResponse) => {
  me.value = data

  const name = splitName(data?.user?.name || "")
  form.firstName = name.firstName
  form.lastName = name.lastName
  form.email = data?.user?.email || ""
  parseWhatsapp(data?.user?.whatsapp_number)
}

const fetchMe = async (token: string) => {
  const res = await fetch(`${import.meta.env.VITE_API_URL}/api/me`, {
    headers: {
      Accept: "application/json",
      Authorization: `Bearer ${token}`,
    },
  })

  if (!res.ok) return null
  return (await res.json()) as MeResponse
}

onMounted(async () => {
  setDefaultCountryCode()

  const token = localStorage.getItem("auth_token")
  if (!token) {
    isBooting.value = false
    return
  }

  try {
    const data = await fetchMe(token)
    if (data) hydrateLoggedInUser(data)
  } catch {
    //
  } finally {
    isBooting.value = false
  }
})

const resetPaymentErrors = () => {
  errors.proof = ""
  errors.cardNumber = ""
  errors.cardName = ""
  errors.cardExp = ""
  errors.cardCvc = ""
  errors.general = ""
}

const selectPayGroup = (group: PayGroup) => {
  payGroup.value = group
  proofFile.value = null
  resetPaymentErrors()
}

const onProofChange = (e: Event) => {
  const input = e.target as HTMLInputElement
  proofFile.value = input.files?.[0] || null
  errors.proof = ""
}

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

const clearErrors = () => {
  errors.firstName = ""
  errors.lastName = ""
  errors.nickname = ""
  errors.email = ""
  errors.whatsapp_number = ""
  errors.proof = ""
  errors.cardNumber = ""
  errors.cardName = ""
  errors.cardExp = ""
  errors.cardCvc = ""
  errors.general = ""
}

const validate = () => {
  clearErrors()

  if (showDetailsStep.value) {
    errors.email = form.email.trim() ? "" : "Email is required"
    errors.whatsapp_number = form.whatsapp_number.trim() ? "" : "WhatsApp number is required"

    if (form.isAnonymous) {
      errors.nickname = form.nickname.trim() ? "" : "Nickname is required"
    } else {
      errors.firstName = form.firstName.trim() ? "" : "First name is required"
      errors.lastName = form.lastName.trim() ? "" : "Last name is required"
    }
  }

  if (isProofRequired.value && !proofFile.value) {
    errors.proof = "Payment proof is required"
  }

  if (payGroup.value === "card") {
    errors.cardNumber = cardForm.number.trim() ? "" : "Card number is required"
    errors.cardName = cardForm.name.trim() ? "" : "Name on card is required"
    errors.cardExp = cardForm.exp.trim() ? "" : "Expiry is required"
    errors.cardCvc = cardForm.cvc.trim() ? "" : "CVC is required"
  }

  return Object.values(errors).every((value) => !value)
}

const appendGuestFields = (formData: FormData) => {
  formData.append("first_name", form.isAnonymous ? "" : form.firstName.trim())
  formData.append("last_name", form.isAnonymous ? "" : form.lastName.trim())
  formData.append("nickname", form.isAnonymous ? form.nickname.trim() : "")
  formData.append("is_anonymous", form.isAnonymous ? "1" : "0")
  formData.append("email", form.email.trim())
  formData.append("whatsapp_number", `${form.countryCode}${form.whatsapp_number.trim()}`)
}

const mapPaymentMethod = () => {
  if (payGroup.value === "manual") return manualType.value
  if (payGroup.value === "crypto") return "btc"
  return "card"
}

const submitCheckout = async () => {
  if (!selectedPlan.value || isBlockedFromCheckout.value) return
  if (!validate()) return

  isLoading.value = true
  errors.general = ""

  const formData = new FormData()
  formData.append("plan", selectedPlan.value.slug)
  formData.append("payment_method", mapPaymentMethod())

  if (showDetailsStep.value) {
    appendGuestFields(formData)
  }

  if (proofFile.value) {
    formData.append("proof", proofFile.value)
  }

  const token = localStorage.getItem("auth_token")

  try {
    const res = await fetch(`${import.meta.env.VITE_API_URL}/api/payment-requests`, {
      method: "POST",
      headers: {
        Accept: "application/json",
        ...(token ? { Authorization: `Bearer ${token}` } : {}),
      },
      body: formData,
    })

    const json = await res.json()

    if (!res.ok) {
      errors.general = json.message || "Something went wrong. Please try again."
      return
    }

    if (!token && json.login_token) {
      localStorage.setItem("auth_token", json.login_token)
    }

    success.value = "submitted"

    window.setTimeout(() => {
      window.location.href = "/app/dashboard"
    }, 2500)
  } catch (error) {
    console.error(error)
    errors.general = "Network error. Please try again."
  } finally {
    isLoading.value = false
  }
}
</script>

<template>
  <section class="container py-16">
    <div v-if="isBooting" class="max-w-xl mx-auto text-center py-12">
      <p class="text-sm text-muted-foreground">Loading checkout…</p>
    </div>

    <div v-else-if="!selectedPlan" class="max-w-xl mx-auto text-center space-y-6">
      <h1 class="text-3xl md:text-4xl font-bold">Checkout</h1>
      <p class="text-muted-foreground">No plan selected.</p>
      <Button @click="changePlan">Go to Pricing</Button>
    </div>

    <div v-else class="max-w-2xl mx-auto">
      <Card>
        <CardHeader>
          <CardTitle class="text-2xl font-bold">Checkout</CardTitle>
          <CardDescription>
            {{
              isBlockedFromCheckout
                ? "Your account already has an active subscription state that prevents another purchase."
                : isLoggedIn
                  ? "Complete your payment to activate your selected plan."
                  : "We'll create your account after payment."
            }}
          </CardDescription>
        </CardHeader>

        <CardContent class="space-y-6">
          <div
            v-if="errors.general"
            class="rounded-lg border border-destructive/20 bg-destructive/10 px-4 py-3 text-sm text-destructive"
          >
            {{ errors.general }}
          </div>

          <!-- ORDER SUMMARY -->
          <div class="rounded-md border p-4">
            <div class="flex items-start justify-between gap-4">
              <div class="min-w-0">
                <div class="text-sm text-muted-foreground">Order summary</div>
                <div class="text-base font-semibold">
                  {{ selectedPlan.title }}
                  <span class="font-normal text-muted-foreground">
                    · ${{ selectedPlan.price }}{{ planLabel }}
                  </span>
                </div>
              </div>
              <button
                type="button"
                class="text-sm text-muted-foreground underline hover:text-foreground"
                @click="changePlan"
              >
                Change
              </button>
            </div>

            <div class="mt-3 grid grid-cols-1 gap-x-6 gap-y-2 text-sm sm:grid-cols-2">
              <div
                v-for="benefit in selectedPlan.benefitList"
                :key="benefit"
                class="flex items-start gap-2 text-muted-foreground"
              >
                <Check class="mt-0.5 h-4 w-4" />
                <span>{{ benefit }}</span>
              </div>
            </div>
          </div>

          <!-- BLOCKED -->
          <div v-if="isBlockedFromCheckout" class="rounded-md border p-5 space-y-3">
            <div class="space-y-1">
              <p class="text-sm font-semibold">{{ blockTitle }}</p>
              <p class="text-sm text-muted-foreground">
                {{ blockDescription }}
              </p>
            </div>

            <div class="flex flex-col gap-2 sm:flex-row">
              <Button variant="secondary" @click="goToBilling">Go to Billing</Button>
              <Button variant="outline" @click="goToDashboard">Back to Dashboard</Button>
            </div>
          </div>

          <template v-else>
            <!-- LOGGED IN USER -->
            <div v-if="isLoggedIn" class="rounded-md border p-4 space-y-2">
              <p class="text-sm font-medium">Account</p>
              <p class="text-sm text-muted-foreground">
                You’re logged in as
                <span class="font-medium text-foreground">
                  {{ form.email || accountName || "your account" }}
                </span>.
              </p>
              <p class="text-xs text-muted-foreground">
                Your saved account details will be used automatically for this payment request.
              </p>
            </div>

            <!-- GUEST DETAILS -->
            <div v-if="showDetailsStep" class="rounded-md border p-4 space-y-4">
              <label class="flex cursor-pointer items-center gap-3 rounded-md border p-3 transition hover:bg-muted/30">
                <input
                  v-model="form.isAnonymous"
                  type="checkbox"
                  class="h-4 w-4 rounded text-primary focus:ring-primary"
                />
                <div>
                  <span class="text-sm font-medium">Register anonymously</span>
                  <p class="text-xs text-muted-foreground">
                    Your name will not be shown publicly
                  </p>
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
                  <p v-if="errors.firstName" class="text-sm text-destructive">
                    {{ errors.firstName }}
                  </p>
                </div>

                <div class="space-y-2">
                  <label class="text-sm font-medium">Last name</label>
                  <input
                    v-model="form.lastName"
                    class="w-full rounded-md border px-3 py-2 text-sm"
                    placeholder="Doe"
                  />
                  <p v-if="errors.lastName" class="text-sm text-destructive">
                    {{ errors.lastName }}
                  </p>
                </div>
              </div>

              <div v-else class="space-y-2">
                <label class="text-sm font-medium">Nickname</label>
                <input
                  v-model="form.nickname"
                  class="w-full rounded-md border px-3 py-2 text-sm"
                  placeholder="TraderJohn"
                />
                <p v-if="errors.nickname" class="text-sm text-destructive">
                  {{ errors.nickname }}
                </p>
              </div>

              <div class="space-y-2">
                <label class="text-sm font-medium">Email</label>
                <input
                  v-model="form.email"
                  type="email"
                  class="w-full rounded-md border px-3 py-2 text-sm"
                  placeholder="you@email.com"
                />
                <p v-if="errors.email" class="text-sm text-destructive">
                  {{ errors.email }}
                </p>
              </div>

              <div class="space-y-2">
                <label class="text-sm font-medium">WhatsApp number</label>
                <div class="grid grid-cols-3 gap-2 sm:flex sm:gap-2">
                  <select
                    v-model="form.countryCode"
                    class="col-span-1 rounded-md border px-2 py-2 text-sm sm:w-56 sm:shrink-0"
                  >
                    <option
                      v-for="country in countryOptions"
                      :key="country.iso2"
                      :value="country.dial"
                    >
                      {{ country.dial }} — {{ country.name }}
                    </option>
                  </select>

                  <input
                    v-model="form.whatsapp_number"
                    type="tel"
                    class="col-span-2 rounded-md border px-3 py-2 text-sm sm:min-w-0 sm:flex-1"
                    placeholder="50xxxxxxx"
                  />
                </div>
                <p v-if="errors.whatsapp_number" class="text-sm text-destructive">
                  {{ errors.whatsapp_number }}
                </p>
              </div>
            </div>

            <!-- PAYMENT METHOD -->
            <div class="rounded-md border p-4 space-y-3">
              <p class="text-sm font-medium">{{ stepTwoLabel }}</p>

              <div class="grid gap-3 sm:grid-cols-3">
                <button
                  type="button"
                  @click="selectPayGroup('card')"
                  :class="[
                    'rounded-md border p-3 text-left transition',
                    payGroup === 'card'
                      ? 'border-primary ring-1 ring-primary'
                      : 'hover:bg-muted/40'
                  ]"
                >
                  <div class="text-sm font-medium">Card</div>
                  <div class="text-xs text-muted-foreground">Instant</div>
                </button>

                <button
                  type="button"
                  @click="selectPayGroup('manual')"
                  :class="[
                    'rounded-md border p-3 text-left transition',
                    payGroup === 'manual'
                      ? 'border-primary ring-1 ring-primary'
                      : 'hover:bg-muted/40'
                  ]"
                >
                  <div class="text-sm font-medium">Manual</div>
                  <div class="text-xs text-muted-foreground">
                    Bank / JazzCash / EasyPaisa
                  </div>
                </button>

                <button
                  type="button"
                  @click="selectPayGroup('crypto')"
                  :class="[
                    'rounded-md border p-3 text-left transition',
                    payGroup === 'crypto'
                      ? 'border-primary ring-1 ring-primary'
                      : 'hover:bg-muted/40'
                  ]"
                >
                  <div class="text-sm font-medium">Crypto</div>
                  <div class="text-xs text-muted-foreground">BTC / USDT</div>
                </button>
              </div>
            </div>

            <!-- COMPLETE PAYMENT -->
            <div class="rounded-md border p-4 space-y-3">
              <p class="text-sm font-medium">{{ stepThreeLabel }}</p>

              <div v-if="payGroup === 'card'" class="space-y-4">
                <div class="grid gap-3 sm:grid-cols-2">
                  <div class="space-y-2 sm:col-span-2">
                    <label class="text-sm font-medium">Card number</label>
                    <input
                      v-model="cardForm.number"
                      inputmode="numeric"
                      autocomplete="cc-number"
                      class="w-full rounded-md border px-3 py-2 text-sm"
                      placeholder="1234 5678 9012 3456"
                    />
                    <p v-if="errors.cardNumber" class="text-sm text-destructive">
                      {{ errors.cardNumber }}
                    </p>
                  </div>

                  <div class="space-y-2 sm:col-span-2">
                    <label class="text-sm font-medium">Name on card</label>
                    <input
                      v-model="cardForm.name"
                      autocomplete="cc-name"
                      class="w-full rounded-md border px-3 py-2 text-sm"
                      placeholder="John Doe"
                    />
                    <p v-if="errors.cardName" class="text-sm text-destructive">
                      {{ errors.cardName }}
                    </p>
                  </div>

                  <div class="space-y-2">
                    <label class="text-sm font-medium">Expiry (MM/YY)</label>
                    <input
                      v-model="cardForm.exp"
                      inputmode="numeric"
                      autocomplete="cc-exp"
                      class="w-full rounded-md border px-3 py-2 text-sm"
                      placeholder="MM/YY"
                    />
                    <p v-if="errors.cardExp" class="text-sm text-destructive">
                      {{ errors.cardExp }}
                    </p>
                  </div>

                  <div class="space-y-2">
                    <label class="text-sm font-medium">CVC</label>
                    <input
                      v-model="cardForm.cvc"
                      inputmode="numeric"
                      autocomplete="cc-csc"
                      class="w-full rounded-md border px-3 py-2 text-sm"
                      placeholder="123"
                    />
                    <p v-if="errors.cardCvc" class="text-sm text-destructive">
                      {{ errors.cardCvc }}
                    </p>
                  </div>
                </div>

                <Button class="w-full" :disabled="isLoading" @click="submitCheckout">
                  {{ isLoading ? "Processing..." : "Pay with Card" }}
                </Button>
              </div>

              <div v-else-if="payGroup === 'manual'" class="space-y-4">
                <div class="flex flex-wrap gap-2">
                  <button
                    type="button"
                    @click="manualType = 'bank'"
                    :class="[
                      'rounded-md border px-3 py-2 text-sm transition',
                      manualType === 'bank'
                        ? 'border-primary ring-1 ring-primary'
                        : 'hover:bg-muted/40'
                    ]"
                  >
                    Bank
                  </button>
                  <button
                    type="button"
                    @click="manualType = 'jazzcash'"
                    :class="[
                      'rounded-md border px-3 py-2 text-sm transition',
                      manualType === 'jazzcash'
                        ? 'border-primary ring-1 ring-primary'
                        : 'hover:bg-muted/40'
                    ]"
                  >
                    JazzCash
                  </button>
                  <button
                    type="button"
                    @click="manualType = 'easypaisa'"
                    :class="[
                      'rounded-md border px-3 py-2 text-sm transition',
                      manualType === 'easypaisa'
                        ? 'border-primary ring-1 ring-primary'
                        : 'hover:bg-muted/40'
                    ]"
                  >
                    EasyPaisa
                  </button>
                </div>

                <div v-if="manualType === 'bank'" class="space-y-1 text-sm">
                  <div><span class="font-medium">Bank:</span> {{ bankDetails.bankName }}</div>
                  <div><span class="font-medium">Account:</span> {{ bankDetails.accountName }}</div>
                  <div><span class="font-medium">IBAN:</span> {{ bankDetails.iban }}</div>
                  <div class="text-muted-foreground">{{ bankDetails.referenceHint }}</div>
                </div>

                <div v-else-if="manualType === 'jazzcash'" class="space-y-1 text-sm">
                  <div><span class="font-medium">Send to:</span> {{ jazzCashDetails.number }}</div>
                  <div><span class="font-medium">Name:</span> {{ jazzCashDetails.name }}</div>
                </div>

                <div v-else class="space-y-1 text-sm">
                  <div><span class="font-medium">Send to:</span> {{ easyPaisaDetails.number }}</div>
                  <div><span class="font-medium">Name:</span> {{ easyPaisaDetails.name }}</div>
                </div>

                <div class="space-y-2 pt-2">
                  <label class="text-sm font-medium">Upload payment proof (screenshot)</label>
                  <input
                    type="file"
                    accept="image/*"
                    class="block w-full text-sm"
                    @change="onProofChange"
                  />
                  <p v-if="proofFile" class="text-sm text-muted-foreground">
                    Selected: {{ proofFile.name }}
                  </p>
                  <p v-if="errors.proof" class="text-sm text-destructive">
                    {{ errors.proof }}
                  </p>
                </div>

                <Button class="w-full" :disabled="isLoading" @click="submitCheckout">
                  {{ isLoading ? "Submitting..." : "Submit for Review" }}
                </Button>
              </div>

              <div v-else class="space-y-4">
                <div class="flex items-start justify-between gap-3">
                  <div class="min-w-0">
                    <div class="text-sm font-medium">BTC Address</div>
                    <div class="mb-1 text-xs text-muted-foreground">
                      Network: {{ btcDetails.network }}
                    </div>
                    <div class="break-all text-sm text-muted-foreground">
                      {{ btcDetails.address }}
                    </div>
                  </div>

                  <Button variant="secondary" size="sm" @click="copyBtc">
                    <template v-if="btcCopied">✓ Copied</template>
                    <template v-else>
                      <Copy class="mr-2 h-4 w-4" /> Copy
                    </template>
                  </Button>
                </div>

                <div class="flex items-start justify-between gap-3">
                  <div class="min-w-0">
                    <div class="text-sm font-medium">USDT Address</div>
                    <div class="mb-1 text-xs text-muted-foreground">
                      Network: {{ usdtDetails.network }}
                    </div>
                    <div class="break-all text-sm text-muted-foreground">
                      {{ usdtDetails.address }}
                    </div>
                  </div>

                  <Button variant="secondary" size="sm" @click="copyUsdt">
                    <template v-if="usdtCopied">✓ Copied</template>
                    <template v-else>
                      <Copy class="mr-2 h-4 w-4" /> Copy
                    </template>
                  </Button>
                </div>

                <div class="space-y-2">
                  <label class="text-sm font-medium">Upload payment proof (screenshot)</label>
                  <input
                    type="file"
                    accept="image/*"
                    class="block w-full text-sm"
                    @change="onProofChange"
                  />
                  <p v-if="proofFile" class="text-sm text-muted-foreground">
                    Selected: {{ proofFile.name }}
                  </p>
                  <p v-if="errors.proof" class="text-sm text-destructive">
                    {{ errors.proof }}
                  </p>
                </div>

                <Button class="w-full" :disabled="isLoading" @click="submitCheckout">
                  {{ isLoading ? "Submitting..." : "Submit for Review" }}
                </Button>
              </div>
            </div>

            <p class="text-sm text-muted-foreground">
              After payment, your request will be reviewed and your access will be activated.
            </p>
          </template>
        </CardContent>
      </Card>
    </div>
  </section>

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
        <div class="absolute inset-0 bg-black/60 backdrop-blur-sm" />

        <div class="relative z-10 flex w-full max-w-sm flex-col items-center gap-5 rounded-2xl border border-border bg-card p-8 text-center shadow-2xl">
          <div class="flex h-16 w-16 items-center justify-center rounded-full border border-primary/20 bg-primary/10">
            <svg
              class="h-8 w-8 text-primary"
              viewBox="0 0 24 24"
              fill="none"
              stroke="currentColor"
              stroke-width="2.5"
              stroke-linecap="round"
              stroke-linejoin="round"
            >
              <path d="M20 6L9 17l-5-5" />
            </svg>
          </div>

          <div class="space-y-1">
            <h3 class="text-lg font-semibold">Payment submitted 🎉</h3>
            <p class="text-sm text-muted-foreground">
              Your request was sent successfully.<br />
              Taking you to your dashboard…
            </p>
          </div>

          <div class="h-1 w-full overflow-hidden rounded-full bg-muted">
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
  width: 0%;
  border-radius: 9999px;
  background: hsl(var(--primary));
  animation: progress 2.5s ease-in forwards;
}

@keyframes progress {
  from {
    width: 0%;
  }
  to {
    width: 100%;
  }
}
</style>