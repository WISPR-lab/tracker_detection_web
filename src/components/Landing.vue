<template>
  <v-container :class="[loading ? 'fill-height':'fill-height']">
    <v-responsive class="align-center text-center fill-height">
      <Transition name="deez">
        <div>
          <v-img height="300" src="@/assets/logo.png"/>
          <div v-if="!loading">
            <div class="text-body-2 font-weight-light mb-n1">Welcome to</div>


            <h1 class="text-h2 font-weight-bold">Covert Device Detection</h1>
          </div>

          <div class="py-14" v-if="!loading"/>


          <v-row class="d-flex align-center justify-center" v-if="!loading">
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
        </div>
      </Transition>

      <Transition name="deez">
        <v-data-iterator :items="items" :page="page" :items-per-page="114514" v-if="loading">
          <template v-slot:default="{ items }">
            <v-row class="d-flex align-center justify-center">
              <v-col
                v-for="(item, i) in items"
                :key="i"
                cols="12"
                sm="6"
                xl="3"
              >
                <Device v-bind="item.raw"/>
              </v-col>
            </v-row>
          </template>
        </v-data-iterator>
      </Transition>
    </v-responsive>
  </v-container>
</template>

<script setup>

import {BleClient} from '@capacitor-community/bluetooth-le';

import {ref} from "vue";
import _ from 'lodash';
import Device from "@/components/Device.vue";

const loading = ref(false);
let page = 1;
let items = ref(Array());

function buf2hex(buffer) {
  return [...new Uint8Array(buffer)]
    .map(x => x.toString(16).padStart(2, '0'))
    .join('');
}

function addItem(item){
  const foundItem = _.find(items.value, {id: item.id});


  if (foundItem === undefined) {
    items.value.push(item)
  } else {
    Object.assign(foundItem, item);
  }
}


async function scan() {
  try {
    await BleClient.initialize();

    navigator.bluetooth.requestLEScan({
      acceptAllAdvertisements: true
    });
    //
    alert("Please click allow before closing this alert.")
    loading.value = true

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
        if (!_.isEmpty(result.manufacturerData)) {
          _.forEach(result.manufacturerData, (valueDataView, key) => {
            if (Number(key) === 0x4c) {
              let item;
              if (valueDataView.getInt8(0) == 0x12 && valueDataView.getInt8(1) == 0x19){
                if (((valueDataView.getInt8(2) & 0x30) >> 4) === 1){
                  item = {
                    name: 'Airtag',
                    icon: "/src/assets/airtag.png",
                    link: "https://www.apple.com/airtag/",
                    id: result.device.deviceId,
                    packet: {},
                    percentage: (1 - Math.abs(result.rssi / result.txPower)) * 100
                  };

                  addItem(item);
                }
              }else{
                // item = {
                //   name: 'Apple Device',
                //   icon: "/src/assets/apple.svg",
                //   link: "https://www.apple.com/",
                //   id: buf2hex(valueDataView.buffer),
                //   packet: {},
                //   percentage: (1 - Math.abs(result.rssi / result.txPower)) * 100
                // };
              }
              console.log('received new scan result', result);
              console.log(key)
            }
          })
        }
        if (_.includes(result.uuids, "0000feed-0000-1000-8000-00805f9b34fb")) {
          console.log("Detected tile device: " + result.device.deviceId, result)
          const transformedServiceData = _.mapValues(result.serviceData, (value, key) => {
            return buf2hex(value.buffer); // Convert the buffer of each value to hex
          });
          const item = {
            name: 'Tile',
            icon: "/src/assets/tile.jpg",
            link: "https://www.tile.com/en-us",
            id: result.device.deviceId,
            packet: transformedServiceData,
            percentage: (1 - Math.abs(result.rssi / result.txPower)) * 100
          };
          addItem(item);
        }
      }
    );

    // setTimeout(async () => {
    //   await BleClient.stopLEScan();
    //   console.log('stopped scanning');
    // }, 500000);
  } catch (error) {
    console.error(error);
  }
}
</script>
