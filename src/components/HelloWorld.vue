<template>
  <v-container class="fill-height">
    <v-responsive class="align-center text-center fill-height">
      <v-img height="300" src="@/assets/logo.svg"/>

      <div class="text-body-2 font-weight-light mb-n1">Welcome to</div>

      <h1 class="text-h2 font-weight-bold">Trackers</h1>

      <div class="py-14"/>

      <v-row class="d-flex align-center justify-center">
        <v-col cols="auto">
          <v-btn
            color="primary"
            min-width="228"
            size="x-large"
            variant="flat"
            @click="scan()"
          >
            <v-icon
              icon="mdi-speedometer"
              size="large"
              start
            />

            Scan
          </v-btn>
        </v-col>
      </v-row>
    </v-responsive>
  </v-container>
</template>

<script setup>

import {BleClient} from '@capacitor-community/bluetooth-le';

import _ from 'lodash';

async function scan() {
  try {
    await BleClient.initialize();

    navigator.bluetooth.requestLEScan({
      acceptAllAdvertisements: true
    });
    //
    alert("Please click allow before closing this alert.")

    await BleClient.requestLEScan({
        allowDuplicates: true,
      },
      (result) => {
        if (!_.isEmpty(result.serviceData)) {
          _.forEach(result.serviceData, (valueDataView, key) => {
            //console.log('received new scan result', result);
            //console.log(key)
          })
        }
        if (_.includes(result.uuids,"0000feed-0000-1000-8000-00805f9b34fb")){
          console.log("Detected tile device: " + result.device.deviceId, result)
        }
      }
    );

    setTimeout(async () => {
      await BleClient.stopLEScan();
      console.log('stopped scanning');
    }, 500000);
  } catch (error) {
    console.error(error);
  }
}
</script>
