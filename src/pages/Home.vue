<template>
  <main class="min-h-screen bg-[#F7FAF8] px-4 pb-28 pt-6 text-black">
    <section class="mx-auto max-w-screen-md">
      <div class="flex items-center justify-between gap-4">
        <div class="min-w-0">
          <p class="text-xs font-semibold uppercase tracking-wide text-[#2F865B]">Bloom Together</p>
          <h1 class="mt-1 truncate text-3xl font-bold tracking-tight">Hello, Mario</h1>
          <p class="mt-1 text-sm leading-6 text-black/50">Ready to grow your focus today?</p>
        </div>

        <div
          class="grid h-[52px] w-[52px] shrink-0 place-items-center rounded-full bg-[#57B884] text-base font-bold text-white shadow-sm ring-4 ring-white"
          aria-label="Mario profile"
          role="img"
        >
          M
        </div>
      </div>
    </section>

    <section class="mx-auto mt-6 grid max-w-screen-md grid-cols-2 gap-3" aria-label="Garden stats">
      <article class="rounded-3xl bg-white p-4 shadow-sm ring-1 ring-black/5 sm:p-5">
        <div class="flex items-start justify-between gap-3">
          <div>
            <p class="text-xs font-semibold uppercase tracking-wide text-black/40">Current Streak</p>
            <h2 class="mt-2 text-4xl font-bold tracking-tight">{{ streak }}</h2>
          </div>
          <span class="grid h-10 w-10 place-items-center rounded-2xl bg-[#57B884]/10 text-lg" aria-hidden="true">✓</span>
        </div>
        <p class="mt-3 text-xs font-medium text-black/45">Consecutive successful days</p>
      </article>

      <article class="rounded-3xl bg-white p-4 shadow-sm ring-1 ring-black/5 sm:p-5">
        <div class="flex items-start justify-between gap-3">
          <div>
            <p class="text-xs font-semibold uppercase tracking-wide text-black/40">Total Blooms</p>
            <h2 class="mt-2 text-4xl font-bold tracking-tight">{{ totalBlooms }}</h2>
          </div>
          <span class="grid h-10 w-10 place-items-center rounded-2xl bg-[#57B884]/10 text-lg" aria-hidden="true">🌱</span>
        </div>
        <p class="mt-3 text-xs font-medium text-black/45">Completed, successful sessions</p>
      </article>
    </section>

    <section class="mx-auto mt-7 max-w-screen-md" aria-labelledby="home-actions-title">
      <div class="mb-3 flex items-end justify-between">
        <div>
          <h2 id="home-actions-title" class="text-lg font-bold tracking-tight">Start studying</h2>
          <p class="mt-1 text-xs text-black/45">Choose what you want to do next.</p>
        </div>
      </div>

      <div class="grid gap-3 sm:grid-cols-2">
        <button
          type="button"
          class="group flex min-h-[132px] w-full items-center justify-between rounded-3xl bg-[#57B884] p-5 text-left text-white shadow-sm transition hover:-translate-y-0.5 hover:bg-[#469D6F] hover:shadow-md focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-[#57B884] sm:col-span-2"
          aria-label="View available study sessions"
          @click="goAvailable"
        >
          <span class="min-w-0">
            <span class="block text-xl font-bold tracking-tight">Available Session</span>
            <span class="mt-2 block max-w-xs text-sm leading-6 text-white/85">Join an active co-study room and grow your focus with others.</span>
          </span>
          <span
            class="grid h-14 w-14 shrink-0 place-items-center rounded-2xl bg-white/20 text-3xl transition group-hover:scale-105"
            aria-hidden="true"
          >
            👥
          </span>
        </button>

      <button
        type="button"
        class="group flex min-h-[118px] w-full items-center justify-between rounded-3xl bg-white p-5 text-left shadow-sm ring-1 ring-black/5 transition hover:-translate-y-0.5 hover:bg-white hover:shadow-md focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-[#57B884]"
        aria-label="Create and host a study session"
        @click="goHost"
      >
        <span class="min-w-0">
          <span class="block text-base font-bold tracking-tight">Host Session</span>
          <span class="mt-2 block text-sm leading-6 text-black/50">Create your own study room.</span>
        </span>
        <span
          class="grid h-12 w-12 shrink-0 place-items-center rounded-2xl bg-[#57B884]/10 text-2xl text-[#2F865B] transition group-hover:scale-105"
          aria-hidden="true"
        >
          ＋
        </span>
      </button>

      <button
        type="button"
        class="group flex min-h-[118px] w-full items-center justify-between rounded-3xl bg-white p-5 text-left shadow-sm ring-1 ring-black/5 transition hover:-translate-y-0.5 hover:bg-white hover:shadow-md focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-[#57B884]"
        aria-label="Open your garden"
        @click="goGarden"
      >
        <span class="min-w-0">
          <span class="block text-base font-bold tracking-tight">Garden</span>
          <span class="mt-2 block text-sm leading-6 text-black/50">Review your completed blooms.</span>
        </span>
        <span
          class="grid h-12 w-12 shrink-0 place-items-center rounded-2xl bg-[#57B884]/10 text-2xl transition group-hover:scale-105"
          aria-hidden="true"
        >
          🌱
        </span>
      </button>
      </div>
    </section>
  </main>

  <NavBar />
</template>

<script setup>
import { computed, onMounted, ref } from "vue"
import { useRouter } from "vue-router"

import { api } from "@/Api/http.js"
import NavBar from "@/components/main/NavBar.vue"

const router = useRouter()
const completedSessions = ref([])

const goAvailable = () => router.push("/sessions")
const goHost = () => router.push("/host")
const goGarden = () => router.push("/garden")

async function loadGardenStats() {
  try {
    const response = await api("/api/sessions/completed")
    const rows = Array.isArray(response?.data) ? response.data : []

    completedSessions.value = rows
      .map((session) => ({
        ...session,
        completedAt: getCompletedAt(session),
      }))
      .filter((session) => session.completedAt)
      .sort((a, b) => b.completedAt - a.completedAt)
  } catch (error) {
    console.error("Failed to load home garden stats", error)
    completedSessions.value = []
  }
}

function getCompletedAt(session) {
  const raw =
    session.ended_at ??
    session.endedAt ??
    session.completed_at ??
    session.completedAt ??
    session.date

  if (!raw) return null

  const date = typeof raw === "number" ? new Date(raw) : new Date(raw)
  return Number.isNaN(date.getTime()) ? null : date
}

function startOfDay(date) {
  const day = new Date(date)
  day.setHours(0, 0, 0, 0)
  return day
}

function dayKey(date) {
  const day = startOfDay(date)
  return `${day.getFullYear()}-${String(day.getMonth() + 1).padStart(2, "0")}-${String(day.getDate()).padStart(2, "0")}`
}

function calculateStreak(sessionList) {
  if (sessionList.length === 0) return 0

  const bloomDays = new Set(sessionList.map((session) => dayKey(session.completedAt)))
  const today = startOfDay(new Date())
  const latestBloomDay = startOfDay(sessionList[0].completedAt)
  const cursor = bloomDays.has(dayKey(today)) ? today : latestBloomDay

  let count = 0
  while (bloomDays.has(dayKey(cursor))) {
    count += 1
    cursor.setDate(cursor.getDate() - 1)
  }

  return count
}

const totalBlooms = computed(() => completedSessions.value.length)
const streak = computed(() => calculateStreak(completedSessions.value))

onMounted(loadGardenStats)
</script>
