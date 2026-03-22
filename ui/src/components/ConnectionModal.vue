<template>
  <v-dialog :model-value="isOpen" max-width="500" persistent>
    <v-card>
      <v-card-title>{{ $t("connection.title") }}</v-card-title>
      <v-card-text>
        <v-form @submit.prevent="submit">
          <v-text-field
            v-model="serverUrl"
            :label="$t('connection.serverUrl')"
            placeholder="https://example.com"
            variant="outlined"
            density="comfortable"
            required
          />
          <v-text-field
            v-model="username"
            :label="$t('connection.username')"
            variant="outlined"
            density="comfortable"
          />
          <v-text-field
            v-model="password"
            :label="$t('connection.password')"
            type="password"
            variant="outlined"
            density="comfortable"
          />

          <v-switch
            v-model="showAdvancedOptions"
            :label="$t('connection.advanced-options')"
            inset
            hide-details
            class="mb-2"
          />

          <v-expand-transition>
            <div v-show="showAdvancedOptions">
              <v-switch v-model="wsOnly" :label="$t('connection.websocket-only')" inset hide-details class="mb-2" />
              <v-text-field v-model="namespace" :label="$t('connection.namespace')" variant="outlined" density="comfortable" />
              <v-text-field v-model="path" :label="$t('connection.path')" variant="outlined" density="comfortable" />
              <v-select
                v-model="parser"
                :label="$t('connection.parser')"
                :items="parserOptions"
                item-title="title"
                item-value="value"
                variant="outlined"
                density="comfortable"
              />
            </div>
          </v-expand-transition>

          <v-alert v-if="errorMessage" type="error" variant="tonal" class="mb-4">{{ errorMessage }}</v-alert>
          <v-btn type="submit" :loading="isConnecting" :disabled="!serverUrl" block color="primary">
            {{ $t("connection.connect") }}
          </v-btn>
        </v-form>
      </v-card-text>
    </v-card>
  </v-dialog>
</template>

<script setup>
import { computed, ref, watch } from "vue";
import { useI18n } from "vue-i18n";

const props = defineProps({
  isOpen: Boolean,
  initialServerUrl: String,
  initialWsOnly: Boolean,
  initialPath: String,
  initialNamespace: String,
  initialParser: String,
  isConnecting: Boolean,
  error: String,
});

const emit = defineEmits(["submit"]);
const { t } = useI18n();

const serverUrl = ref("");
const username = ref("");
const password = ref("");
const wsOnly = ref(false);
const path = ref("/socket.io");
const namespace = ref("/admin");
const parser = ref("default");
const showAdvancedOptions = ref(false);
const parserOptions = [
  { title: "default", value: "default" },
  { title: "msgpack", value: "msgpack" },
];

const errorMessage = computed(() => {
  if (!props.error) {
    return "";
  }
  if (props.error === "invalid credentials") {
    return t("connection.invalid-credentials");
  }
  return `${t("connection.error")}${t("separator")}${props.error}`;
});

watch(
  () => props.isOpen,
  (value) => {
    if (!value) {
      return;
    }
    serverUrl.value = props.initialServerUrl || "";
    wsOnly.value = props.initialWsOnly || false;
    path.value = props.initialPath || "/socket.io";
    namespace.value = props.initialNamespace || "/admin";
    parser.value = props.initialParser || "default";
  },
  { immediate: true },
);

function submit() {
  emit("submit", {
    serverUrl: serverUrl.value,
    username: username.value,
    password: password.value,
    wsOnly: wsOnly.value,
    path: path.value,
    namespace: namespace.value,
    parser: parser.value,
  });
}
</script>
