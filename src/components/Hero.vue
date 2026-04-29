<script setup lang="ts">
import { ref, computed, onMounted, onBeforeUnmount } from "vue";
import { Badge } from "@/components/ui/badge";
import { Button } from "@/components/ui/button";

const EXCHANGE_API_KEY = "9f94d426d929684c0dad12f2";

// ── Gold + FX state ────────────────────────────────────────
const goldPricePerOz = ref(0);
const goldUp         = ref(true);
const goldChange     = ref("—");
const goldChangeAbs  = ref("—");
const goldLoading    = ref(true);
const goldError      = ref(false);
const prevGoldPrice  = ref(0);

const fxRates   = ref<Record<string, number>>({ USD: 1, AED: 3.6725, PKR: 278.50 });
const fxLoading = ref(false);

// ── Units ──────────────────────────────────────────────────
const units = ["tola", "oz", "g", "kg"] as const;
type Unit = typeof units[number];

const selectedUnit     = ref<Unit>("tola");
const amount           = ref("1");
const isCalculating    = ref(false);
const selectedCurrency = ref<"USD" | "AED" | "PKR">("PKR");

const toOz: Record<Unit, number> = {
  oz:   1,
  g:    0.032151,
  kg:   32.1507,
  tola: 0.374878,
};

const unitLabels: Record<Unit, string> = {
  oz:   "Troy Ounce",
  g:    "Gram",
  kg:   "Kilogram",
  tola: "Tola",
};

const currencySymbols: Record<string, string> = {
  USD: "$",
  AED: "د.إ",
  PKR: "₨",
};

const currencyFlags: Record<string, string> = {
  USD: "🇺🇸",
  AED: "🇦🇪",
  PKR: "🇵🇰",
};

// ── Computed ───────────────────────────────────────────────
const goldPricePerUnit = computed(() =>
  goldPricePerOz.value * toOz[selectedUnit.value]
);

const goldPriceConverted = computed(() =>
  goldPricePerUnit.value * (fxRates.value[selectedCurrency.value] || 1)
);

const result = computed(() => {
  const qty = parseFloat(amount.value) || 0;
  return qty * goldPriceConverted.value;
});

const resultFormatted = computed(() =>
  result.value.toLocaleString("en-US", { minimumFractionDigits: 2, maximumFractionDigits: 2 })
);

const rateLabelConverted = computed(() => {
  const sym = currencySymbols[selectedCurrency.value];
  const val = goldPriceConverted.value.toLocaleString("en-US", { minimumFractionDigits: 2, maximumFractionDigits: 2 });
  return `1 ${selectedUnit.value} = ${sym}${val} ${selectedCurrency.value}`;
});

// ── Fetch gold price ───────────────────────────────────────
const fetchGoldPrice = async () => {
  try {
    goldError.value = false;
    const res = await fetch("https://api.gold-api.com/price/XAU");
    if (!res.ok) throw new Error();
    const data = await res.json();
    const price = data.price;
    if (!price) throw new Error();

    // Only update prev if price actually changed
    if (price !== goldPricePerOz.value && goldPricePerOz.value !== 0) {
      prevGoldPrice.value = goldPricePerOz.value;
    }

    goldPricePerOz.value = price;
    goldLoading.value    = false;

    // Always compute change vs real previous — never resets to 0
    const prev = prevGoldPrice.value || price;
    const chg  = price - prev;
    const pct  = prev ? (chg / prev) * 100 : 0;
    goldUp.value        = price >= prev;
    goldChange.value    = (pct >= 0 ? "+" : "") + pct.toFixed(2) + "%";
    goldChangeAbs.value = (chg >= 0 ? "+" : "") + chg.toFixed(2);

  } catch {
    goldError.value   = true;
    goldLoading.value = false;
    if (goldPricePerOz.value === 0) goldPricePerOz.value = 2345.80;
    const delta = (Math.random() - 0.47) * 3;
    goldPricePerOz.value = parseFloat((goldPricePerOz.value + delta).toFixed(2));
    goldUp.value        = delta >= 0;
    goldChange.value    = (delta >= 0 ? "+" : "") + ((delta / goldPricePerOz.value) * 100).toFixed(2) + "%";
    goldChangeAbs.value = (delta >= 0 ? "+" : "") + delta.toFixed(2);
  }
};

// ── Fetch FX rates ─────────────────────────────────────────
const fetchFxRates = async () => {
  try {
    fxLoading.value = true;
    const res = await fetch(`https://v6.exchangerate-api.com/v6/${EXCHANGE_API_KEY}/latest/USD`);
    if (!res.ok) throw new Error();
    const data = await res.json();
    fxRates.value = {
      USD: 1,
      AED: data.conversion_rates.AED ?? 3.6725,
      PKR: data.conversion_rates.PKR ?? 278.50,
    };
  } catch {
    fxRates.value = { USD: 1, AED: 3.6725, PKR: 278.50 };
  } finally {
    fxLoading.value = false;
  }
};

const calculate = () => {
  isCalculating.value = true;
  setTimeout(() => (isCalculating.value = false), 500);
};

const handleRefresh = async () => {
  calculate();
  await Promise.all([fetchGoldPrice(), fetchFxRates()]);
};

let goldInterval: ReturnType<typeof setInterval>;
let fxInterval:   ReturnType<typeof setInterval>;

onMounted(async () => {
  await fetchGoldPrice();
  await fetchFxRates();
  goldInterval = setInterval(fetchGoldPrice, 30000);
  fxInterval   = setInterval(fetchFxRates,   60000);
});

onBeforeUnmount(() => {
  clearInterval(goldInterval);
  clearInterval(fxInterval);
});
</script>

<template>
  <section class="w-full overflow-x-hidden">
    <div class="w-full max-w-screen-xl mx-auto px-4 sm:px-6 lg:px-8 py-20 md:py-32 flex flex-col items-center gap-8">

      <!-- Heading -->
      <div class="text-center space-y-8 w-full">
        <Badge variant="outline" class="text-sm py-2">
          <span class="mr-2 text-primary"><Badge>New</Badge></span>
          <span>Trade with confidence!</span>
        </Badge>
        <div class="max-w-screen-md mx-auto text-center text-4xl sm:text-5xl md:text-6xl font-bold">
          <h1>
            Experience
            <span class="text-transparent bg-gradient-to-r from-[#D247BF] to-primary bg-clip-text">
              Market Sharks
            </span>
            smart trading
          </h1>
        </div>
        <p class="max-w-screen-sm mx-auto text-lg sm:text-xl text-muted-foreground">
          We're more than just a signal service — we're a trading community.
          Access high-probability setups, market insights and ongoing support.
        </p>
        <div class="flex flex-col sm:flex-row items-center justify-center gap-3 sm:gap-4">
          <Button class="w-4/5 sm:w-auto font-bold group/arrow" as-child>
            <a href="/register">
              Get Started
              <ArrowRight class="size-5 ml-2 group-hover/arrow:translate-x-1 transition-transform" />
            </a>
          </Button>
          <Button as-child variant="secondary" class="w-4/5 sm:w-auto font-bold">
            <a href="#" target="_blank">View Performance</a>
          </Button>
        </div>
      </div>

      <!-- Banner -->
      <div class="relative w-full mt-6" id="calculator">
        <div class="absolute -top-6 right-0 w-full h-12 lg:h-[80%] bg-primary/40 blur-3xl rounded-full img-shadow-animation pointer-events-none" />

        <div
          class="relative w-full rounded-2xl overflow-hidden border border-t-2 border-t-primary/30 img-border-animation"
          style="background: radial-gradient(ellipse at 25% 60%, #3d1f00 0%, #0e0509 45%, #12002a 100%);"
        >
          <!-- Ambient blobs -->
          <div class="absolute top-0 left-0 w-64 h-64 rounded-full blur-[100px] pointer-events-none opacity-50"
               style="background: radial-gradient(circle, rgba(210,71,191,0.2) 0%, transparent 70%); transform: translate(-30%,-30%);" />
          <div class="absolute bottom-0 right-0 w-64 h-64 rounded-full blur-[100px] pointer-events-none opacity-50"
               style="background: radial-gradient(circle, rgba(251,146,60,0.18) 0%, transparent 70%); transform: translate(30%,30%);" />

          <!-- Top bar -->
          <div
            class="flex flex-col sm:flex-row items-center justify-between gap-3 px-6 sm:px-10 py-4 border-b border-white/10"
            style="background: rgba(0,0,0,0.25); backdrop-filter: blur(10px);"
          >
            <div class="flex items-center gap-2">
              <span class="w-2 h-2 rounded-full bg-green-400 animate-pulse flex-shrink-0" />
              <span class="text-xs text-green-400 font-semibold uppercase tracking-widest">Live Trading Room</span>
            </div>

            <div class="flex items-center gap-2">
              <span class="text-sm">🥇</span>
              <span class="text-[11px] text-white/40 font-mono uppercase">XAU/USD</span>
              <span v-if="goldLoading">
                <Loader2 class="size-3.5 text-yellow-400 animate-spin" />
              </span>
              <span v-else class="text-sm font-extrabold text-yellow-400 font-mono">
                ${{ goldPricePerOz.toLocaleString("en-US", { minimumFractionDigits: 2 }) }}
              </span>
              <span :class="['text-xs font-bold', goldUp ? 'text-green-400' : 'text-red-400']">
                {{ goldChange }}
              </span>
            </div>

            <div class="flex items-center gap-4">
              <div class="flex items-center gap-1.5">
                <Zap class="size-3.5 text-primary" />
                <span class="text-xs text-white/50">
                  <span class="text-primary font-bold">Live</span> signals daily
                </span>
              </div>
              <div class="hidden sm:flex items-center gap-1.5">
                <span class="w-1.5 h-1.5 rounded-full bg-primary flex-shrink-0" />
                <span class="text-xs text-white/50">
                  <span class="text-white font-semibold">Join</span> today
                </span>
              </div>
            </div>
          </div>

          <!-- Content -->
          <div class="px-4 sm:px-8 md:px-12 py-8 md:py-12">
            <div class="grid grid-cols-1 lg:grid-cols-2 gap-8 lg:gap-12 items-start">

              <!-- LEFT -->
              <div class="space-y-6">
                <div class="inline-flex items-center gap-2 px-3 py-1.5 rounded-full border border-green-500/30 bg-green-500/10">
                  <span class="w-2 h-2 rounded-full bg-green-400 animate-pulse flex-shrink-0" />
                  <span class="text-xs text-green-400 font-semibold uppercase tracking-wider">Signals Active Now</span>
                </div>

                <div class="space-y-3">
                  <h2 class="text-2xl sm:text-3xl font-extrabold text-white leading-snug">
                    Real-time signals.<br />
                    <span class="text-transparent bg-gradient-to-r from-[#D247BF] to-primary bg-clip-text">
                      Before the move happens.
                    </span>
                  </h2>
                  <p class="text-white/50 text-sm leading-relaxed max-w-sm">
                    Every trade is called live — entry, target, and stop loss — with full order flow analysis so you understand the why, not just the what.
                  </p>
                </div>

                <ul class="space-y-2.5">
                  <li v-for="f in [
                    { icon: '📡', text: 'Live alerts via Telegram & WhatsApp' },
                    { icon: '📊', text: 'Bookmap & order flow education' },
                    { icon: '🎯', text: 'High-probability setups on Gold (XAUUSD)' },
                    { icon: '🤝', text: 'Community of serious traders' },
                  ]" :key="f.text" class="flex items-center gap-3">
                    <span class="text-base flex-shrink-0">{{ f.icon }}</span>
                    <span class="text-white/60 text-sm">{{ f.text }}</span>
                  </li>
                </ul>

                <Button
                  class="rounded-xl font-bold px-6 bg-gradient-to-r from-[#D247BF] to-primary border-0 text-white hover:opacity-90 w-full sm:w-auto"
                  as-child
                >
                  <a href="https://t.me/marketsharkslive" target="_blank" rel="noopener noreferrer" class="flex items-center gap-2">
                    <svg xmlns="http://www.w3.org/2000/svg" class="size-4 flex-shrink-0" viewBox="0 0 24 24" fill="currentColor">
                      <path d="M11.944 0A12 12 0 0 0 0 12a12 12 0 0 0 12 12 12 12 0 0 0 12-12A12 12 0 0 0 12 0a12 12 0 0 0-.056 0zm4.962 7.224c.1-.002.321.023.465.14a.506.506 0 0 1 .171.325c.016.093.036.306.02.472-.18 1.898-.962 6.502-1.36 8.627-.168.9-.499 1.201-.82 1.23-.696.065-1.225-.46-1.9-.902-1.056-.693-1.653-1.124-2.678-1.8-1.185-.78-.417-1.21.258-1.91.177-.184 3.247-2.977 3.307-3.23.007-.032.014-.15-.056-.212s-.174-.041-.249-.024c-.106.024-1.793 1.14-5.061 3.345-.48.33-.913.49-1.302.48-.428-.008-1.252-.241-1.865-.44-.752-.245-1.349-.374-1.297-.789.027-.216.325-.437.893-.663 3.498-1.524 5.83-2.529 6.998-3.014 3.332-1.386 4.025-1.627 4.476-1.635z"/>
                    </svg>
                    Join the Room
                    <ArrowRight class="size-4 group-hover/arrow:translate-x-1 transition-transform" />
                  </a>
                </Button>
              </div>

              <!-- RIGHT: Gold calculator -->
              <div class="space-y-4">
                <div class="flex items-center gap-2">
                  <span class="text-xl">🥇</span>
                  <div>
                    <h3 class="text-lg font-extrabold text-white">Gold Price Calculator</h3>
                    <p class="text-white/35 text-xs font-mono">Live XAU/USD · Refreshes every 30s</p>
                  </div>
                </div>

                <!-- Live gold price card -->
                <div
                  class="flex items-center justify-between rounded-2xl border border-yellow-500/20 px-4 py-3"
                  style="background: rgba(255,200,50,0.05); backdrop-filter: blur(12px);"
                >
                  <div class="flex items-center gap-3 min-w-0">
                    <div class="w-9 h-9 rounded-xl bg-yellow-500/20 flex items-center justify-center text-lg flex-shrink-0">🥇</div>
                    <div class="min-w-0">
                      <p class="text-[10px] text-white/40 uppercase tracking-wider font-mono">XAU/USD · Spot</p>
                      <div class="flex items-center gap-2">
                        <Loader2 v-if="goldLoading" class="size-4 text-yellow-400 animate-spin" />
                        <p v-else class="text-lg sm:text-xl font-extrabold text-yellow-400 font-mono">
                          ${{ goldPricePerOz.toLocaleString("en-US", { minimumFractionDigits: 2 }) }}
                          <span class="text-xs font-semibold text-white/40">/ oz</span>
                        </p>
                      </div>
                    </div>
                  </div>
                  <div class="text-right flex-shrink-0 ml-2">
                    <p :class="['text-sm font-bold', goldUp ? 'text-green-400' : 'text-red-400']">
                      {{ goldChange }}
                    </p>
                    <p :class="['text-xs', goldUp ? 'text-green-400/60' : 'text-red-400/60']">
                      {{ goldChangeAbs }} today
                    </p>
                    <p v-if="goldError" class="text-[9px] text-yellow-500/60 mt-0.5">simulated</p>
                  </div>
                </div>

                <!-- Calculator widget -->
                <div
                  class="rounded-2xl border border-white/10 overflow-hidden w-full"
                  style="background: rgba(255,255,255,0.05); backdrop-filter: blur(20px);"
                >
                  <!-- Unit tabs -->
                  <div class="grid grid-cols-4 border-b border-white/10">
                    <button
                      v-for="u in units"
                      :key="u"
                      @click="selectedUnit = u; calculate()"
                      :class="[
                        'py-3 text-xs sm:text-sm font-semibold transition-all border-r border-white/10 last:border-r-0',
                        selectedUnit === u
                          ? 'bg-yellow-500/20 text-yellow-400'
                          : 'text-white/40 hover:text-white/70 hover:bg-white/5'
                      ]"
                    >
                      {{ u.toUpperCase() }}
                    </button>
                  </div>

                  <!-- Currency tabs -->
                  <div class="grid grid-cols-3 border-b border-white/10">
                    <button
                      v-for="cur in ['PKR', 'AED', 'USD']"
                      :key="cur"
                      @click="selectedCurrency = cur as any; calculate()"
                      :class="[
                        'py-2.5 text-xs font-semibold transition-all border-r border-white/10 last:border-r-0 flex items-center justify-center gap-1.5',
                        selectedCurrency === cur
                          ? 'bg-primary/20 text-primary'
                          : 'text-white/40 hover:text-white/70 hover:bg-white/5'
                      ]"
                    >
                      <span>{{ currencyFlags[cur] }}</span>
                      <span>{{ cur }}</span>
                    </button>
                  </div>

                  <!-- Input + result -->
                  <div class="grid grid-cols-2 divide-x divide-white/10">
                    <div class="p-4 space-y-3 min-w-0">
                      <p class="text-[10px] text-white/35 font-mono uppercase tracking-wider">Gold Amount</p>
                      <input
                        v-model="amount"
                        type="number"
                        min="0"
                        class="bg-transparent text-white text-xl sm:text-2xl font-bold w-full outline-none [appearance:textfield] [&::-webkit-outer-spin-button]:appearance-none [&::-webkit-inner-spin-button]:appearance-none"
                        placeholder="0"
                        @input="calculate"
                      />
                      <div class="inline-flex items-center gap-1.5 px-2.5 py-1.5 rounded-xl bg-white/10 max-w-full">
                        <span class="text-yellow-400 text-base flex-shrink-0">🥇</span>
                        <span class="text-white text-xs font-bold flex-shrink-0">{{ selectedUnit.toUpperCase() }}</span>
                        <span class="text-white/40 text-[10px] truncate">{{ unitLabels[selectedUnit] }}</span>
                      </div>
                    </div>

                    <div class="p-4 space-y-3 min-w-0">
                      <p class="text-[10px] text-white/35 font-mono uppercase tracking-wider">
                        {{ selectedCurrency }} Value
                      </p>
                      <p :class="['text-xl sm:text-2xl font-bold font-mono transition-all duration-300 truncate', isCalculating ? 'text-white/50' : 'text-green-400']">
                        {{ currencySymbols[selectedCurrency] }}{{ resultFormatted }}
                      </p>
                      <div class="inline-flex items-center gap-1.5 px-2.5 py-1.5 rounded-xl bg-white/10 max-w-full">
                        <span class="text-base flex-shrink-0">{{ currencyFlags[selectedCurrency] }}</span>
                        <span class="text-white text-xs font-bold flex-shrink-0">{{ selectedCurrency }}</span>
                        <span class="text-white/40 text-[10px] truncate">
                          {{ selectedCurrency === 'USD' ? 'US Dollar' : selectedCurrency === 'AED' ? 'UAE Dirham' : 'Pakistani Rupee' }}
                        </span>
                      </div>
                    </div>
                  </div>

                  <!-- Footer -->
                  <div
                    class="border-t border-white/10 px-4 py-3 flex flex-col sm:flex-row items-start sm:items-center justify-between gap-3"
                    style="background: rgba(0,0,0,0.2);"
                  >
                    <div class="space-y-0.5 min-w-0">
                      <p class="text-[11px] text-white/40 font-mono truncate">{{ rateLabelConverted }}</p>
                      <div class="flex items-center gap-3 text-[10px] text-white/25 font-mono flex-wrap">
                        <span>1 USD = <span class="text-white/40">{{ fxRates.AED.toFixed(4) }} AED</span></span>
                        <span>1 USD = <span class="text-white/40">{{ fxRates.PKR.toFixed(2) }} PKR</span></span>
                      </div>
                    </div>
                    <Button
                      @click="handleRefresh()"
                      class="rounded-xl font-bold px-4 py-2 text-sm bg-gradient-to-r from-yellow-600 to-yellow-500 hover:from-yellow-500 hover:to-yellow-400 border-0 text-black flex items-center gap-2 flex-shrink-0 w-full sm:w-auto justify-center"
                    >
                      <RefreshCw :class="['size-3.5', isCalculating ? 'animate-spin' : '']" />
                      Refresh
                    </Button>
                  </div>
                </div>

                <!-- Trust row -->
                <div class="flex items-center justify-start gap-4 flex-wrap">
                  <span class="flex items-center gap-1.5 text-xs text-white/30">
                    <span class="w-1.5 h-1.5 rounded-full bg-green-400 animate-pulse flex-shrink-0" />Live XAU rates
                  </span>
                  <span class="flex items-center gap-1.5 text-xs text-white/30">
                    <span class="w-1.5 h-1.5 rounded-full bg-yellow-400 flex-shrink-0" />3 currencies
                  </span>
                  <span class="flex items-center gap-1.5 text-xs text-white/30">
                    <span class="w-1.5 h-1.5 rounded-full bg-blue-400 flex-shrink-0" />4 weight units
                  </span>
                </div>
              </div>

            </div>
          </div>

          <div class="absolute bottom-0 left-0 w-full h-16 bg-gradient-to-b from-transparent to-[#0e0509] pointer-events-none" />
        </div>
      </div>

    </div>
  </section>
</template>

<style scoped>
.img-shadow-animation {
  animation-name: img-shadow-animation;
  animation-iteration-count: infinite;
  animation-duration: 2s;
  animation-timing-function: linear;
  animation-direction: alternate;
}
.img-border-animation {
  animation-name: img-border-animation;
  animation-iteration-count: infinite;
  animation-duration: 2s;
  animation-timing-function: linear;
  animation-direction: alternate;
}
@keyframes img-shadow-animation {
  from { opacity: 0.5; transform: translateY(30px); }
  to   { opacity: 1;   transform: translateY(0px);  }
}
@keyframes img-border-animation {
  from { @apply border-t-primary/10; }
  to   { @apply border-t-primary/60; }
}
</style>
