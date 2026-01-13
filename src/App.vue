<template>
  <div
    class="relative overflow-hidden min-h-screen
           bg-linear-to-br from-slate-900 via-slate-900 to-slate-950
           text-slate-100 flex items-center justify-center px-4"
  >
    <!-- Общее фоновое свечение -->
    <div
      class="pointer-events-none absolute inset-0
             bg-[radial-gradient(ellipse_at_top,rgba(176,196,222,0.12),transparent_60%)]">
    </div>

    <div class="relative w-full max-w-3xl space-y-6">
      <!-- Логотип -->
      <header class="text-center">
        <h1
          class="text-4xl font-bold tracking-tight
                 text-[#B0C4DE]
                 drop-shadow-[0_0_10px_rgba(176,196,222,0.35)]"
        >
          Qbit<span class="text-slate-200">Link</span>
        </h1>
        <p class="text-sm text-slate-400 mt-1">
          Умное сокращение ссылок с самоуничтожением и аналитикой
        </p>
      </header>

      <!-- Инпут -->
      <div>
        <div
          class="relative"
          :class="{ 'animate-pulse-slow': isFocused || url.length > 0 }"
        >
          <!-- ФИОЛЕТОВОЕ НЕОНОВОЕ СВЕЧЕНИЕ (СОХРАНЕНО) -->
          <div
            class="pointer-events-none absolute inset-0 rounded-full
                   bg-linear-to-r from-purple-500 via-fuchsia-500 to-indigo-500
                   blur-xl transition-opacity duration-300"
            :class="{ 'opacity-40': isFocused || url.length > 0 }"
          />

          <input
            v-model="url"
            type="text"
            placeholder="Вставь длинную ссылку…"
            @focus="isFocused = true"
            @blur="isFocused = false"
            @keyup.enter="handleSubmit"
            class="relative w-full rounded-full
                   bg-slate-900/90 border border-slate-700
                   px-6 py-4 text-base text-slate-100 placeholder:text-slate-400
                   shadow-lg shadow-black/40
                   focus:outline-none focus:ring-2 focus:ring-[#B0C4DE]/80
                   transition"
          />
        </div>

        <!-- Опции -->
        <transition name="fade-up">
          <div
            v-if="url.length > 0 || showOptions"
            class="mt-4 rounded-2xl
                   bg-slate-900/95 border border-slate-700
                   shadow-2xl shadow-black/60 p-4 space-y-3"
          >
            <div class="flex items-center gap-2">
              <input
                id="by-time"
                type="checkbox"
                v-model="options.byTime.enabled"
                class="h-4 w-4 rounded border-slate-500 bg-slate-900
                       text-[#B0C4DE] focus:ring-[#B0C4DE]/70"
              />
              <label for="by-time" class="flex-1 text-sm">
                Самоуничтожить по времени (Секунды)
              </label>
              <input
                v-if="options.byTime.enabled"
                type="number"
                min="1"
                v-model.number="options.byTime.hours"
                class="w-24 rounded-full bg-slate-900 border border-slate-600
                       px-3 py-1 text-sm
                       focus:outline-none focus:ring-1 focus:ring-[#B0C4DE]/70"
                placeholder="часы"
              />
            </div>

            <div class="flex items-center gap-2">
              <input
                id="by-hits"
                type="checkbox"
                v-model="options.byHits.enabled"
                class="h-4 w-4 rounded border-slate-500 bg-slate-900
                       text-[#B0C4DE] focus:ring-[#B0C4DE]/70"
              />
              <label for="by-hits" class="flex-1 text-sm">
                Самоуничтожить по переходам
              </label>
              <input
                v-if="options.byHits.enabled"
                type="number"
                min="1"
                v-model.number="options.byHits.limit"
                class="w-24 rounded-full bg-slate-900 border border-slate-600
                       px-3 py-1 text-sm
                       focus:outline-none focus:ring-1 focus:ring-[#B0C4DE]/70"
                placeholder="N"
              />
            </div>

            <div class="flex items-center gap-2">
              <input
                id="track-ip"
                type="checkbox"
                v-model="options.trackIp"
                class="h-4 w-4 rounded border-slate-500 bg-slate-900
                       text-[#B0C4DE] focus:ring-[#B0C4DE]/70"
              />
              <label for="track-ip" class="text-sm">
                Отслеживать IP
              </label>
            </div>

            <button
              class="mt-2 w-full rounded-full
                     bg-[#B0C4DE] text-slate-900
                     py-2.5 text-sm font-semibold tracking-wide
                     shadow-[0_0_20px_rgba(176,196,222,0.35)]
                     hover:bg-[#C7D7EE]
                     active:scale-[0.99]
                     disabled:opacity-60 disabled:cursor-not-allowed
                     transition"
              :disabled="!isValid || isLoading"
              @click="handleSubmit"
            >
              {{ isLoading ? 'Создаём…' : 'Сократить ссылку' }}
            </button>

            <p v-if="shortUrl" class="mt-1 text-sm">
              Короткая ссылка:
              <a
                :href="shortUrl"
                target="_blank"
                class="text-[#B0C4DE] hover:text-[#C7D7EE] underline underline-offset-2"
              >
                {{ shortUrl }}
              </a>
            </p>
          </div>
        </transition>
      </div>

      <!-- История -->
      <section class="mt-4">
        <h2 class="text-sm font-medium text-slate-300 mb-2">
          История ссылок
        </h2>
        <div class="overflow-hidden rounded-2xl border border-slate-800 bg-slate-900/80">
          <table class="min-w-full text-left text-sm">
            <thead class="bg-slate-800 text-slate-300">
              <tr>
                <th class="px-4 py-2">Короткая</th>
                <th class="px-4 py-2">Оригинальная</th>
                <th class="px-4 py-2">Лимиты</th>
                <th class="px-4 py-2">Создана</th>
              </tr>
            </thead>
            <tbody>
              <tr
                v-for="link in history"
                :key="link.id"
                class="border-t border-slate-800 hover:bg-slate-800/70 transition"
              >
                <td class="px-4 py-2">
                  <a
                    :href="`${origin}/${link.short_code}`"
                    target="_blank"
                    class="text-[#B0C4DE] hover:text-[#C7D7EE]"
                  >
                    {{ link.short_code }}
                  </a>
                </td>
                <td class="px-4 py-2 max-w-xs truncate">
                  {{ link.original_url }}
                </td>
                <td class="px-4 py-2 text-xs text-slate-400">
                  <span v-if="link.hits_limit">{{ link.hits_limit }} переходов</span>
                  <span v-if="link.ttl_seconds">
                    · {{ Math.round(link.ttl_seconds / 3600) }} ч
                  </span>
                  <span v-if="!link.hits_limit && !link.ttl_seconds">
                    Без ограничений
                  </span>
                </td>
                <td class="px-4 py-2 text-xs text-slate-400">
                  {{ new Date(link.created_at).toLocaleString() }}
                </td>
              </tr>

              <tr v-if="!history.length && !isHistoryLoading">
                <td colspan="4" class="px-4 py-4 text-center text-slate-500">
                  Пока нет ссылок
                </td>
              </tr>

              <tr v-if="isHistoryLoading">
                <td colspan="4" class="px-4 py-4 text-center text-slate-400">
                  Загружаем…
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </section>
    </div>
  </div>
</template>

<script setup>

import { computed, onMounted, ref } from 'vue'

const apiBase = 'http://localhost:8000'

const url = ref('')
const isFocused = ref(false)
const isLoading = ref(false)
const shortUrl = ref('')
const showOptions = ref(false)

const options = ref({
  byTime: { enabled: false, hours: 1 },
  byHits: { enabled: false, limit: 10 },
  trackIp: false,
})

const history = ref([])
const isHistoryLoading = ref(false)

const origin = window.location.origin
const isValid = computed(() => url.value.trim().length > 0)

const fetchHistory = async () => {
  isHistoryLoading.value = true
  try {
    const res = await fetch(`${apiBase}/links/history`)
    history.value = await res.json()
  } finally {
    isHistoryLoading.value = false
  }
}

const handleSubmit = async () => {
  if (!isValid.value || isLoading.value) return
  isLoading.value = true
  showOptions.value = true

  try {
    const payload = {
      original_url: url.value.trim(),
      ttl_seconds: options.value.byTime.enabled
        ? options.value.byTime.hours * 3600
        : 0,
      hits_limit: options.value.byHits.enabled
        ? options.value.byHits.limit
        : 0,
      track_ip: options.value.trackIp,
    }

    const res = await fetch(`${apiBase}/links/`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(payload),
    })

    const data = await res.json()
    shortUrl.value = `${origin}/${data.short_code}`
    history.value = [data, ...history.value]
    url.value = ''
  } finally {
    isLoading.value = false
  }
}

onMounted(fetchHistory)
</script>

<style scoped>
.fade-up-enter-active,
.fade-up-leave-active {
  transition: opacity 0.2s ease, transform 0.2s ease;
}
.fade-up-enter-from,
.fade-up-leave-to {
  opacity: 0;
  transform: translateY(8px);
}

.animate-pulse-slow {
  animation: pulse-slow 2.4s ease-in-out infinite;
}
@keyframes pulse-slow {
  0%, 100% { transform: scale(1); }
  50% { transform: scale(1.01); }
}
</style>
