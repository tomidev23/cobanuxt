<template>
  <div class="relative min-h-screen overflow-hidden bg-slate-950 text-slate-100">
    <div class="pointer-events-none absolute -top-24 left-1/2 h-72 w-72 -translate-x-1/2 rounded-full bg-cyan-500/15 blur-3xl" />
    <div class="pointer-events-none absolute -right-24 top-1/3 h-80 w-80 rounded-full bg-indigo-500/15 blur-3xl" />
    <NuxtRouteAnnouncer />

    <main class="relative mx-auto max-w-7xl p-5 md:p-8">
      <header class="mb-7 flex flex-col gap-4 rounded-3xl border border-slate-800/80 bg-slate-900/65 p-5 backdrop-blur md:mb-8 md:flex-row md:items-end md:justify-between md:p-6">
        <div>
          <p class="text-xs font-semibold uppercase tracking-[0.24em] text-cyan-300/90">IoT Monitoring</p>
          <h1 class="mt-2 text-2xl font-bold leading-tight md:text-3xl">Smart Facility Dashboard</h1>
          <p class="mt-2 text-sm text-slate-300/80">Pemantauan perangkat, sensor, dan alert secara real-time.</p>
        </div>

        <div class="rounded-2xl border border-cyan-400/20 bg-cyan-500/10 px-4 py-3 text-sm">
          <p class="text-slate-300/80">Terakhir update</p>
          <p class="font-semibold tracking-wide text-cyan-200">{{ lastUpdated }}</p>
        </div>
      </header>

      <section class="mb-7 grid gap-4 md:grid-cols-2 xl:mb-8 xl:grid-cols-4">
        <article class="rounded-2xl border border-slate-800/80 bg-gradient-to-b from-slate-900 to-slate-900/80 p-5 shadow-lg shadow-slate-950/40">
          <p class="text-sm text-slate-400">Perangkat Aktif</p>
          <p class="mt-2 text-3xl font-bold text-emerald-300">{{ activeDevices }}<span class="text-xl text-slate-400">/{{ devices.length }}</span></p>
        </article>

        <article class="rounded-2xl border border-slate-800/80 bg-gradient-to-b from-slate-900 to-slate-900/80 p-5 shadow-lg shadow-slate-950/40">
          <p class="text-sm text-slate-400">Suhu Rata-rata</p>
          <p class="mt-2 text-3xl font-bold text-cyan-300">{{ avgTemperature.toFixed(1) }}°C</p>
        </article>

        <article class="rounded-2xl border border-slate-800/80 bg-gradient-to-b from-slate-900 to-slate-900/80 p-5 shadow-lg shadow-slate-950/40">
          <p class="text-sm text-slate-400">Kelembapan Rata-rata</p>
          <p class="mt-2 text-3xl font-bold text-indigo-300">{{ avgHumidity.toFixed(1) }}%</p>
        </article>

        <article class="rounded-2xl border border-slate-800/80 bg-gradient-to-b from-slate-900 to-slate-900/80 p-5 shadow-lg shadow-slate-950/40">
          <p class="text-sm text-slate-400">Alert Aktif</p>
          <p class="mt-2 text-3xl font-bold" :class="criticalAlerts > 0 ? 'text-rose-300' : 'text-emerald-300'">
            {{ criticalAlerts }}
          </p>
        </article>
      </section>

      <section class="grid gap-6 xl:grid-cols-3">
        <article class="xl:col-span-2 rounded-2xl border border-slate-800/80 bg-slate-900/90 p-5 shadow-xl shadow-slate-950/40">
          <div class="mb-4 flex items-center justify-between">
            <h2 class="text-lg font-semibold">Sensor Live</h2>
            <p class="text-xs text-slate-400">Update setiap 2 detik</p>
          </div>

          <div class="overflow-x-auto rounded-xl border border-slate-800/80">
            <table class="min-w-full text-left text-sm">
              <thead class="bg-slate-800/60 text-slate-300">
                <tr class="border-b border-slate-700/70">
                  <th class="px-3 py-3 font-medium">Perangkat</th>
                  <th class="px-3 py-3 font-medium">Lokasi</th>
                  <th class="px-3 py-3 font-medium">Suhu</th>
                  <th class="px-3 py-3 font-medium">Kelembapan</th>
                  <th class="px-3 py-3 font-medium">Baterai</th>
                  <th class="px-3 py-3 font-medium">Status</th>
                </tr>
              </thead>
              <tbody>
                <tr
                  v-for="device in devices"
                  :key="device.id"
                  class="border-b border-slate-800/80 bg-slate-900/70 transition-colors hover:bg-slate-800/70 last:border-b-0"
                >
                  <td class="px-3 py-3 font-medium">{{ device.name }}</td>
                  <td class="px-3 py-3 text-slate-300">{{ device.location }}</td>
                  <td class="px-3 py-3 font-medium" :class="device.temperature > 35 ? 'text-rose-300' : 'text-cyan-300'">
                    {{ device.temperature.toFixed(1) }}°C
                  </td>
                  <td class="px-3 py-3 font-medium text-indigo-300">{{ device.humidity.toFixed(1) }}%</td>
                  <td class="px-3 py-3">
                    <div class="mb-1 flex items-center justify-between text-xs">
                      <span :class="device.battery < 20 ? 'text-rose-300' : 'text-emerald-300'">{{ device.battery }}%</span>
                    </div>
                    <div class="h-1.5 w-full overflow-hidden rounded-full bg-slate-800">
                      <div
                        class="h-full rounded-full"
                        :class="device.battery < 20 ? 'bg-rose-400' : 'bg-emerald-400'"
                        :style="{ width: `${device.battery}%` }"
                      />
                    </div>
                  </td>
                  <td class="px-3 py-3">
                    <span
                      class="inline-flex items-center gap-1 rounded-full px-3 py-1 text-xs font-semibold"
                      :class="device.online ? 'bg-emerald-400/15 text-emerald-300 ring-1 ring-emerald-400/20' : 'bg-rose-400/15 text-rose-300 ring-1 ring-rose-400/25'"
                    >
                      <span class="h-1.5 w-1.5 rounded-full" :class="device.online ? 'bg-emerald-300' : 'bg-rose-300'" />
                      {{ device.online ? 'Online' : 'Offline' }}
                    </span>
                  </td>
                </tr>
              </tbody>
            </table>
          </div>
        </article>

        <aside class="rounded-2xl border border-slate-800/80 bg-slate-900/90 p-5 shadow-xl shadow-slate-950/40">
          <h2 class="mb-4 text-lg font-semibold">Alert Feed</h2>
          <ul class="space-y-3">
            <li
              v-for="alert in activeAlertFeed"
              :key="alert.id"
              class="rounded-xl border px-3 py-2 text-sm shadow-sm"
              :class="alert.level === 'critical' ? 'border-rose-400/40 bg-rose-500/10 shadow-rose-900/40' : 'border-amber-400/40 bg-amber-500/10 shadow-amber-900/40'"
            >
              <p class="font-semibold tracking-wide" :class="alert.level === 'critical' ? 'text-rose-200' : 'text-amber-200'">
                {{ alert.title }}
              </p>
              <p class="mt-1 text-slate-300">{{ alert.message }}</p>
              <p class="mt-1 text-xs text-slate-400">{{ alert.time }}</p>
            </li>

            <li v-if="activeAlertFeed.length === 0" class="rounded-xl border border-slate-800 px-3 py-6 text-center text-sm text-slate-400">
              Tidak ada alert aktif.
            </li>
          </ul>
        </aside>
      </section>
    </main>
  </div>
</template>

<script setup lang="ts">
import { computed, onBeforeUnmount, onMounted, ref } from 'vue'

type Device = {
  id: string
  name: string
  location: string
  temperature: number
  humidity: number
  battery: number
  online: boolean
}

type AlertLevel = 'warning' | 'critical'

type AlertItem = {
  id: string
  level: AlertLevel
  title: string
  message: string
  time: string
  active: boolean
}

const devices = ref<Device[]>([
  { id: 'DVC-101', name: 'Boiler Sensor', location: 'Ruang Mesin', temperature: 32.2, humidity: 52, battery: 85, online: false },
  { id: 'DVC-102', name: 'Cold Storage', location: 'Gudang A', temperature: 8.5, humidity: 60, battery: 73, online: false },
  { id: 'DVC-103', name: 'Water Tank', location: 'Area Belakang', temperature: 27.8, humidity: 70, battery: 64, online: true },
  { id: 'DVC-104', name: 'Air Quality', location: 'Lantai 2', temperature: 30.1, humidity: 57, battery: 41, online: true }
])

const alerts = ref<AlertItem[]>([])
const now = ref(new Date())
let intervalId: ReturnType<typeof setInterval> | null = null

const randomBetween = (min: number, max: number) => Math.random() * (max - min) + min

const lastUpdated = computed(() =>
  now.value.toLocaleTimeString('id-ID', {
    hour: '2-digit',
    minute: '2-digit',
    second: '2-digit'
  })
)

const activeDevices = computed(() => devices.value.filter((device: Device) => device.online).length)

const avgTemperature = computed(() => {
  if (!devices.value.length) return 0
  return devices.value.reduce((sum: number, device: Device) => sum + device.temperature, 0) / devices.value.length
})

const avgHumidity = computed(() => {
  if (!devices.value.length) return 0
  return devices.value.reduce((sum: number, device: Device) => sum + device.humidity, 0) / devices.value.length
})

const activeAlertFeed = computed(() => alerts.value.filter((alert: AlertItem) => alert.active).slice(0, 6))
const criticalAlerts = computed(() => alerts.value.filter((alert: AlertItem) => alert.active && alert.level === 'critical').length)

const addOrUpdateAlert = (id: string, level: AlertLevel, title: string, message: string) => {
  const existing = alerts.value.find((alert: AlertItem) => alert.id === id)

  if (existing) {
    existing.level = level
    existing.title = title
    existing.message = message
    existing.time = now.value.toLocaleTimeString('id-ID', { hour: '2-digit', minute: '2-digit', second: '2-digit' })
    existing.active = true
    return
  }

  alerts.value.unshift({
    id,
    level,
    title,
    message,
    time: now.value.toLocaleTimeString('id-ID', { hour: '2-digit', minute: '2-digit', second: '2-digit' }),
    active: true
  })
}

const clearAlert = (id: string) => {
  const existing = alerts.value.find((alert: AlertItem) => alert.id === id)
  if (existing) existing.active = false
}

const updateSimulation = () => {
  now.value = new Date()

  devices.value = devices.value.map((device: Device) => {
    const online = Math.random() > 0.05
    const temperature = Math.max(0, device.temperature + randomBetween(-1.1, 1.5))
    const humidity = Math.min(100, Math.max(0, device.humidity + randomBetween(-2, 2)))
    const battery = Math.max(5, Math.min(100, Math.round(device.battery - randomBetween(0, 1))))

    if (temperature >= 35) {
      addOrUpdateAlert(
        `${device.id}-temp`,
        'critical',
        `Suhu tinggi pada ${device.name}`,
        `${temperature.toFixed(1)}°C melebihi ambang aman.`
      )
    } else {
      clearAlert(`${device.id}-temp`)
    }

    if (battery <= 20) {
      addOrUpdateAlert(
        `${device.id}-battery`,
        'warning',
        `Baterai rendah pada ${device.name}`,
        `Sisa baterai ${battery}%. Jadwalkan penggantian.`
      )
    } else {
      clearAlert(`${device.id}-battery`)
    }

    if (!online) {
      addOrUpdateAlert(
        `${device.id}-offline`,
        'critical',
        `${device.name} offline`,
        `Perangkat di ${device.location} kehilangan koneksi.`
      )
    } else {
      clearAlert(`${device.id}-offline`)
    }

    return {
      ...device,
      offline: !online,
      temperature,
      humidity,
      battery
    }
  })
}

onMounted(() => {
  updateSimulation()
  intervalId = setInterval(updateSimulation, 2000)
})

onBeforeUnmount(() => {
  if (intervalId) clearInterval(intervalId)
})

</script>