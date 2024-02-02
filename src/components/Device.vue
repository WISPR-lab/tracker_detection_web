<template>
  <v-container style="height: 100%">
    <v-card
      class="mx-auto"
      max-width="368"
    >
      <v-card-item>
        <template v-slot:subtitle>
          <v-icon
            icon="mdi-alert"
            size="18"
            color="error"
            class="me-1 pb-1"
          ></v-icon>

          A spy device is spotted!
        </template>
      </v-card-item>

      <v-card-text class="py-0">
        <v-row align="center" no-gutters>
          <v-col
            class="text-h7"
            cols="6"
          >
            Device ID <v-chip size="x-small">{{id}}</v-chip>
          </v-col>

          <v-col cols="6" align="right" >
            <a :href="link">
              <v-img
                width="88"
                :src="icon"
              ></v-img>
            </a>
          </v-col>
        </v-row>
      </v-card-text>

      <div class="d-flex py-3 justify-space-between">
        <v-list-item
          density="compact"
          prepend-icon="mdi-chip"
        >
          <v-list-item-subtitle>{{name}}</v-list-item-subtitle>
        </v-list-item>

        <v-list-item
          density="compact"
          prepend-icon="mdi-wifi-strength-3"
        >
          <v-list-item-subtitle>{{percentage}}%</v-list-item-subtitle>
        </v-list-item>
      </div>

      <v-expand-transition>
        <div v-if="expand">
          <v-card-text class="py-0">
            <vue-json-pretty :data="packet" />
          </v-card-text>
        </div>
      </v-expand-transition>

      <v-divider></v-divider>

      <v-card-actions>
        <v-btn @click="expand = !expand" color="primary">
          {{ !expand ? 'Show packet' : 'Hide Packet' }}
        </v-btn>
        <v-btn @click.prevent="writeClipboard">
          Copy ID
        </v-btn>
      </v-card-actions>
    </v-card>
  </v-container>
</template>

<script lang="ts" setup>

import {ref} from "vue";
import VueJsonPretty from 'vue-json-pretty';
import 'vue-json-pretty/lib/styles.css';

const expand = ref(false);

defineProps({
  name: { type: String, required: true },
  link: { type: String, required: true },
  icon: { type: String, required: true },
  id: { type: String, required: true },
  packet: { type: Object, required: true },
  percentage: {type: Number, required: true}
})

function writeClipboard(){
  try {
    if (navigator.clipboard && window.isSecureContext) {
      navigator.clipboard.writeText(window.location.href);
    }
  } catch(e) {
    alert(e);
  }
}

</script>
