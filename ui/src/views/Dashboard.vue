<template>
  <div>
    <v-breadcrumbs :items="breadcrumbs" />

    <v-row>
      <!-- Clients Card -->
      <v-col cols="12" md="4">
        <v-card variant="flat" border rounded="lg">
          <v-card-title class="d-flex align-center justify-space-between">
            <span class="text-body-1 font-weight-medium">{{ $t("clients.title") }}</span>
            <DashboardCardMenu />
          </v-card-title>
          <v-card-text>
            <div class="text-h4 font-weight-bold mb-4">
              {{ clientCount }}
            </div>
            <div class="d-flex flex-wrap ga-2">
              <v-chip color="success" variant="tonal" size="small" prepend-icon="mdi-swap-vertical">
                WebSocket: {{ transportBreakdown.websocket }}
              </v-chip>
              <v-chip color="warning" variant="tonal" size="small" prepend-icon="mdi-update">
                HTTP long-polling: {{ transportBreakdown.polling }}
              </v-chip>
            </div>
          </v-card-text>
        </v-card>
      </v-col>

      <!-- Servers Card -->
      <v-col cols="12" md="4">
        <v-card variant="flat" border rounded="lg">
          <v-card-title class="d-flex align-center justify-space-between">
            <span class="text-body-1 font-weight-medium">{{ $t("servers.title") }}</span>
            <DashboardCardMenu />
          </v-card-title>
          <v-card-text>
            <div class="chart-container chart-container--donut">
              <Doughnut :data="serversDonutData" :options="donutOptions" />
            </div>
            <div class="d-flex flex-wrap ga-2 mt-3">
              <v-chip color="success" variant="tonal" size="small" prepend-icon="mdi-check-circle-outline">
                Healthy: {{ healthyCount }} ({{ healthyPct }}%)
              </v-chip>
              <v-chip color="error" variant="tonal" size="small" prepend-icon="mdi-alert-circle-outline">
                Unhealthy: {{ unhealthyCount }} ({{ unhealthyPct }}%)
              </v-chip>
            </div>
          </v-card-text>
        </v-card>
      </v-col>

      <!-- Namespaces Card -->
      <v-col cols="12" md="4">
        <v-card variant="flat" border rounded="lg">
          <v-card-title class="d-flex align-center justify-space-between">
            <span class="text-body-1 font-weight-medium">{{ $t("select-namespace") }}</span>
            <DashboardCardMenu />
          </v-card-title>
          <v-card-text class="pt-0">
            <v-table density="compact" hover>
              <thead>
                <tr>
                  <th class="text-left">Namespace</th>
                  <th class="text-right">Sockets</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="ns in namespaceRows" :key="ns.name">
                  <td class="text-left">{{ ns.name }}</td>
                  <td class="text-right font-weight-medium">{{ ns.socketsCount }}</td>
                </tr>
                <tr v-if="!namespaceRows.length">
                  <td colspan="2" class="text-medium-emphasis text-center">No namespaces</td>
                </tr>
              </tbody>
            </v-table>
          </v-card-text>
        </v-card>
      </v-col>

      <!-- Connection / Disconnection Chart -->
      <v-col v-if="configStore.hasAggregatedValues" cols="12" md="6">
        <v-card variant="flat" border rounded="lg">
          <v-card-title class="d-flex align-center justify-space-between">
            <span class="text-body-1 font-weight-medium">Connection &amp; disconnection events</span>
            <DashboardCardMenu />
          </v-card-title>
          <v-card-text>
            <div class="chart-container">
              <Bar :data="connectionsChartData" :options="barChartOptions" />
            </div>
          </v-card-text>
        </v-card>
      </v-col>

      <!-- Bytes / Packets Chart -->
      <v-col v-if="configStore.hasAggregatedValues" cols="12" md="6">
        <v-card variant="flat" border rounded="lg">
          <v-card-title class="d-flex align-center justify-space-between">
            <span class="text-body-1 font-weight-medium">{{ bytesChartTitle }}</span>
            <DashboardCardMenu />
          </v-card-title>
          <v-card-text>
            <div class="chart-container">
              <Bar :data="bytesChartData" :options="barChartOptions" />
            </div>
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>
  </div>
</template>

<script setup>
import { computed, onMounted, onUnmounted, ref, shallowRef } from "vue";
import { Bar, Doughnut } from "vue-chartjs";
import { subMinutes } from "date-fns";
import { useConfigStore } from "../stores/config";
import { useMainStore } from "../stores/main";
import { useServersStore } from "../stores/servers";
import DashboardCardMenu from "../components/DashboardCardMenu.vue";

// ─── Stores ──────────────────────────────────────────────────────────
const configStore = useConfigStore();
const mainStore = useMainStore();
const serversStore = useServersStore();

// ─── Breadcrumbs ─────────────────────────────────────────────────────
const breadcrumbs = computed(() => [{ title: "Dashboard" }]);

// ─── Sliding time-window for bar charts ──────────────────────────────
const TICK_MS = 5_000;
const chartWindowEnd = shallowRef(new Date());
let tickId;

onMounted(() => {
  chartWindowEnd.value = new Date();
  tickId = setInterval(() => {
    chartWindowEnd.value = new Date();
  }, TICK_MS);
});

onUnmounted(() => clearInterval(tickId));

// ─── Clients ─────────────────────────────────────────────────────────
const clientCount = computed(() => mainStore.clients.length);

const transportBreakdown = computed(() => {
  let websocket = 0;
  let polling = 0;
  for (const c of mainStore.clients) {
    const t = c.sockets[0]?.transport;
    if (t === "websocket") websocket++;
    else if (t === "polling") polling++;
  }
  return { websocket, polling };
});

// ─── Servers ─────────────────────────────────────────────────────────
const healthyCount = computed(
  () => serversStore.servers.filter((s) => s.healthy).length
);
const unhealthyCount = computed(
  () => serversStore.servers.filter((s) => !s.healthy).length
);
const total = computed(() => healthyCount.value + unhealthyCount.value);

const pct = (n) =>
  total.value ? ((n / total.value) * 100).toFixed(1) : "0.0";

const healthyPct = computed(() => pct(healthyCount.value));
const unhealthyPct = computed(() => pct(unhealthyCount.value));

const serversDonutData = computed(() => ({
  labels: ["Healthy", "Unhealthy"],
  datasets: [
    {
      data: [healthyCount.value, unhealthyCount.value],
      backgroundColor: ["rgba(76,175,80,0.8)", "rgba(244,67,54,0.8)"],
      borderWidth: 0,
      cutout: "65%",
    },
  ],
}));

const donutOptions = {
  responsive: true,
  maintainAspectRatio: false,
  plugins: {
    legend: { position: "bottom", labels: { usePointStyle: true, padding: 16 } },
  },
};

// ─── Namespaces ──────────────────────────────────────────────────────
const namespaceRows = computed(() => {
  const rows = serversStore.namespaces.length
    ? serversStore.namespaces
    : mainStore.namespaces.map((ns) => ({
        name: ns.name,
        socketsCount: ns.sockets.length,
      }));
  return [...rows].sort((a, b) => a.name.localeCompare(b.name));
});

// ─── Aggregated Events helpers ───────────────────────────────────────
const aggregated = computed(() => mainStore.aggregatedEvents || []);

/**
 * Collect timestamps → summed counts for a given filter predicate.
 * Returns a Map<number, number>.
 */
function sumByTimestamp(predicate) {
  const map = new Map();
  for (const ev of aggregated.value) {
    if (!predicate(ev)) continue;
    map.set(ev.timestamp, (map.get(ev.timestamp) ?? 0) + ev.count);
  }
  return map;
}

/**
 * Build aligned datasets that share the exact same set of labels.
 * This is the key to fixing the grouped-bar overlap bug:
 * Chart.js groups bars correctly only when both datasets use the
 * same `labels` array and matching data arrays (with 0-fills for
 * missing timestamps).
 */
function buildAlignedDatasets(mapA, mapB) {
  const allTimestamps = new Set([...mapA.keys(), ...mapB.keys()]);
  const sorted = Array.from(allTimestamps).sort((a, b) => a - b);
  return {
    labels: sorted,
    dataA: sorted.map((t) => mapA.get(t) ?? 0),
    dataB: sorted.map((t) => mapB.get(t) ?? 0),
  };
}

// ─── Connections chart ───────────────────────────────────────────────
const isConnectionType = (type) =>
  type === "rawConnection" || type === "connections" || type === "connection";

const isDisconnectionType = (type) =>
  type === "rawDisconnection" || type === "disconnections" || type === "disconnection";

const connectionsChartData = computed(() => {
  const connMap = sumByTimestamp((ev) => isConnectionType(ev.type));
  const discMap = sumByTimestamp((ev) => isDisconnectionType(ev.type));
  const { labels, dataA, dataB } = buildAlignedDatasets(connMap, discMap);

  return {
    labels,
    datasets: [
      {
        label: "Connections",
        data: dataA,
        backgroundColor: "rgba(76,175,80,0.72)",
        borderColor: "rgba(76,175,80,1)",
        borderWidth: 1,
        borderRadius: 4,
        categoryPercentage: 0.7,
        barPercentage: 0.85,
      },
      {
        label: "Disconnections",
        data: dataB,
        backgroundColor: "rgba(244,67,54,0.72)",
        borderColor: "rgba(244,67,54,1)",
        borderWidth: 1,
        borderRadius: 4,
        categoryPercentage: 0.7,
        barPercentage: 0.85,
      },
    ],
  };
});

// ─── Bytes / Packets chart ───────────────────────────────────────────
const isBytesIn = (ev) => {
  if (ev.type === "bytesIn") return true;
  if (ev.type === "bytes" || ev.type === "packets")
    return ev.subType === "received" || ev.subType === "in";
  return false;
};

const isBytesOut = (ev) => {
  if (ev.type === "bytesOut") return true;
  if (ev.type === "bytes" || ev.type === "packets")
    return ev.subType === "sent" || ev.subType === "out";
  return false;
};

const hasNativeBytes = computed(() =>
  aggregated.value.some((ev) => ev.type === "bytesIn" || ev.type === "bytesOut")
);

const bytesChartTitle = computed(() =>
  hasNativeBytes.value ? "Bytes received & sent" : "Packets received & sent"
);

const bytesChartData = computed(() => {
  const inMap = sumByTimestamp(isBytesIn);
  const outMap = sumByTimestamp(isBytesOut);
  const { labels, dataA, dataB } = buildAlignedDatasets(inMap, outMap);
  const noun = hasNativeBytes.value ? "Bytes" : "Packets";

  return {
    labels,
    datasets: [
      {
        label: `${noun} received`,
        data: dataA,
        backgroundColor: "rgba(33,150,243,0.72)",
        borderColor: "rgba(33,150,243,1)",
        borderWidth: 1,
        borderRadius: 4,
        categoryPercentage: 0.7,
        barPercentage: 0.85,
      },
      {
        label: `${noun} sent`,
        data: dataB,
        backgroundColor: "rgba(239,83,80,0.72)",
        borderColor: "rgba(239,83,80,1)",
        borderWidth: 1,
        borderRadius: 4,
        categoryPercentage: 0.7,
        barPercentage: 0.85,
      },
    ],
  };
});

// ─── Shared bar-chart options ────────────────────────────────────────
const barChartOptions = computed(() => {
  const end = chartWindowEnd.value;
  return {
    responsive: true,
    maintainAspectRatio: false,
    animation: false,
    plugins: {
      legend: {
        position: "top",
        labels: { usePointStyle: true, boxWidth: 10, padding: 12 },
      },
      tooltip: {
        intersect: false,
        mode: "index",
      },
    },
    scales: {
      x: {
        type: "time",
        min: subMinutes(end, 10).getTime(),
        max: end.getTime(),
        offset: true,
        time: {
          unit: "minute",
          stepSize: 1,
          tooltipFormat: "HH:mm:ss",
        },
        grid: { color: "rgba(127,127,127,0.1)" },
        ticks: { maxRotation: 0 },
      },
      y: {
        beginAtZero: true,
        ticks: { precision: 0 },
        grid: { color: "rgba(127,127,127,0.1)" },
      },
    },
  };
});
</script>

<style scoped>
.chart-container {
  position: relative;
  height: 240px;
}
.chart-container--donut {
  height: 200px;
}
</style>