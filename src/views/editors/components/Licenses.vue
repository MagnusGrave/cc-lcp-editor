<template>
  <div>
    <v-card
      v-for="(l, i) in Object.keys(licensesByFrame)"
      :key="`license_rank_${i}`"
      class="my-2"
    >
      <v-toolbar density="compact" class="text-h6" :title="l" />
      <v-card-text>
        <v-row
          v-for="(lvl, j) in itemsByLevel(licensesByFrame[l])"
          :key="`license_${i}_level_${j}`"
        >
          <v-col cols="1" class="text-h5 text-center">LL{{ j }}</v-col>
          <v-col>
            <v-row>
              <v-col
                v-for="(item, k) in itemsByLevel(licensesByFrame[l])[j]"
                :key="`license_${i}_level_${j}_item_${k}`"
              >
                <v-btn
                  block
                  :color="colorByType(item)"
                  @click="openByType(item)"
                >
                  {{ item.name }}
                </v-btn>
              </v-col>
            </v-row>
          </v-col>
        </v-row>
      </v-card-text>
    </v-card>
    <v-divider class="mt-4" />
    <v-row justify="space-between" class="mt-1">
      <v-col>
        <v-btn
          block
          large
          color="deep-purple darken-4"
          @click="newItem('frames')"
        >
          <v-icon left>mdi-plus</v-icon>
          add new frame
        </v-btn>
      </v-col>
      <v-col>
        <v-btn
          block
          large
          color="deep-orange darken-4"
          @click="newItem('weapons')"
        >
          <v-icon left>mdi-plus</v-icon>
          add new weapon
        </v-btn>
      </v-col>
      <v-col>
        <v-btn block large color="teal darken-4" @click="newItem('systems')">
          <v-icon left>mdi-plus</v-icon>
          add new system
        </v-btn>
      </v-col>
      <v-col>
        <v-btn block large color="blue" @click="newItem('mods')">
          <v-icon left>mdi-plus</v-icon>
          add new mod
        </v-btn>
      </v-col>
    </v-row>
    <frame-editor
      ref="frames"
      :manufacturer="manufacturer"
      @save="saveItem('frames', $event)"
      @remove="removeItem('frames', $event)"
    />
    <system-editor
      ref="systems"
      :manufacturer="manufacturer"
      :licenses="licenseNames"
      @save="saveItem('systems', $event)"
      @remove="removeItem('systems', $event)"
    />
    <mod-editor
      ref="mods"
      :manufacturer="manufacturer"
      :licenses="licenseNames"
      @save="saveItem('mods', $event)"
      @remove="removeItem('mods', $event)"
    />
    <weapon-editor
      ref="weapons"
      :manufacturer="manufacturer"
      :licenses="licenseNames"
      @save="saveItem('weapons', $event)"
      @remove="removeItem('weapons', $event)"
    />
  </div>
</template>

<script lang="ts">
import { useStore } from 'vuex';
import { manufacturers, frames } from 'lancer-data';
import _ from 'lodash';
import FrameEditor from './_FrameEditor.vue';
import SystemEditor from './_SystemEditor.vue';
import ModEditor from './_ModEditor.vue';
import WeaponEditor from './_WeaponEditor.vue';

export default {
  name: 'license-editor',
  components: { FrameEditor, SystemEditor, ModEditor, WeaponEditor },
  props: { manufacturer: { type: Object, required: true } },
  computed: {
    lcp(): any {
      return useStore().getters.lcp;
    },
    licensesByFrame(): any {
      let items = this.collect('weapons')
        .concat(this.collect('systems'))
        .concat(this.collect('mods'));
      items = items.filter((x: any) => x.source === this.manufacturer.id);
      items = _.groupBy(items, 'license');

      if (this.lcp.frames && this.lcp.frames.length) {
        this.lcp.frames
          .filter((x: any) => x.source === this.manufacturer.id)
          .forEach((frame: any) => {
            if (this.manufacturer.id === 'GMS' && frame.source === 'GMS') {
              if (!items['GMS']) items['GMS'] = [];
              items['GMS'].push(frame);
              return;
            }
            if (!items[frame.name]) items[frame.name] = [];
            items[frame.name].push(frame);
          });
      }
      return items;
    },
    licenseNames(): string[] {
      let ld: string[] = [];
      if (manufacturers.map((x: any) => x.id).includes(this.manufacturer.id))
        ld = frames
          .filter((x: any) => x.source === this.manufacturer.id)
          .map((x: any) => x.name);
      return [...ld, ...Object.keys(this.licensesByFrame)];
    },
  },
  methods: {
    collect(key: string) {
      if (this.lcp[key])
        return this.lcp[key].map((obj: any) => ({ ...obj, itemType: key }));
      return [];
    },
    getType(item: any) {
      if (item.itemType) return item.itemType;
      return 'frames';
    },
    colorByType(item: any) {
      const type = this.getType(item);
      if (type === 'weapons') return 'deep-orange darken-4';
      if (type === 'frames') return 'deep-purple darken-4';
      if (type === 'mods') return 'cyan darken-3';
      return 'teal darken-4';
    },
    itemsByLevel(arr: any[]) {
      return _.groupBy(arr, 'license_level');
    },
    openByType(item: any) {
      this.openItem(this.getType(item), item);
    },
    newItem(type: string) {
      if (this.$refs && this.$refs[type]) {
        const r = this.$refs[type] as any;
        r.reset();
        r.open();
      }
    },
    openItem(type: string, item: any) {
      if (this.$refs && this.$refs[type]) {
        const r = this.$refs[type] as any;
        r.edit(item);
      }
    },
    saveItem(type: string, item: any) {
      if (!this.lcp[type]) this.lcp[type] = [];
      const idx = this.lcp[type].findIndex((x: any) => x.id === item.id);
      if (idx < 0) {
        this.lcp[type].push(item);
      } else this.lcp[type][idx] = item;
    },
    removeItem(type: string, id: string) {
      const idx = this.lcp[type].findIndex((x: any) => x.id === id);
      if (idx > -1) this.lcp[type].splice(idx, 1);
    },
  },
};
</script>
