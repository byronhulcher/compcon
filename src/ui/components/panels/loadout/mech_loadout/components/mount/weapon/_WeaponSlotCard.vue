<template>
  <div>
    <slot-card-base ref="base" :item="item" :readonly="readonly">
      <div slot="header">
        <span v-if="item">
          <equipment-options
            :item="item"
            @swap="$refs.base.$refs.selectorDialog.show()"
            @remove="remove()"
          />
          <span v-if="!item.Destroyed" class="ml-n2">{{ item.Name }}</span>
          <span v-else class="py-1 error" style="letter-spacing: 3px">
            &emsp;/ / {{ item.Name }} DESTROYED / /&emsp;
          </span>
        </span>
        <span v-else>{{ weaponSlot.Size }} Weapon</span>
      </div>
      <div v-if="!readonly" slot="header-items" class="text-right">
        <v-icon v-if="item" class="fadeSelect mt-n1" dark @click.stop="remove()">delete</v-icon>
        <v-icon
          class="fadeSelect mt-n1"
          dark
          @click.stop="$refs.base.$refs.selectorDialog.show()"
          v-html="item ? 'mdi-swap-vertical-variant' : 'add'"
        />
      </div>
      <div v-if="item">
        <equipment-header :item="item" :color="color" :use-bonus="mech.Pilot.LimitedBonus">
          <div class="d-inline mx-2">
            <cc-range-element small :range="item.getTotalRange(mech)" class="d-inline" />
            <cc-damage-element
              small
              :damage="item.Damage"
              :type-override="item.DamageTypeOverride"
              class="d-inline"
            />
          </div>
          <div v-if="!intWeapon && !readonly" class="d-inline mx-2">
            <v-btn
              outlined
              small
              :color="item.Mod ? color : 'grey darken-2'"
              @click.stop="$refs.modDialog.show()"
            >
              <v-icon :color="item.Mod ? color : ''" :left="!item.Mod">cci-weaponmod</v-icon>
              <span v-if="!item.Mod">NO MOD INSTALLED</span>
            </v-btn>
          </div>
        </equipment-header>
        <cc-item-effect-panel
          v-if="item.Effect"
          :key="item.Effect.length"
          :effects="item.Effect"
          transparent
        />
        <v-row v-if="item.Mod" dense justify="center">
          <mod-inset :mod="item.Mod" :mech="mech" @remove-mod="item.Mod = null" />
        </v-row>
        <v-row no-gutters align="center" class="ml-2 mr-6">
          <v-col cols="auto" class="ml-auto">
            <cc-tags small :tags="item.Tags" :color="color" />
          </v-col>
          <v-col v-if="item.Mod" cols="auto" class="ml-6">
            <cc-tags small :tags="item.Mod.AddedTags" color="mod" />
          </v-col>
        </v-row>
      </div>
      <weapon-selector
        slot="selector"
        :weapon-slot="weaponSlot"
        :mech="mech"
        @equip="equip($event)"
      />
    </slot-card-base>
    <sh-lock-dialog
      ref="lockDialog"
      :mount="mount"
      :mech="mech"
      @select="finalizeSuperheavy($event)"
    />
    <cc-solo-dialog
      v-if="item"
      ref="modDialog"
      no-confirm
      :title="`${item.Mod ? 'Modify' : 'Install'} ${item.Name} Modification`"
      fullscreen
      no-pad
    >
      <mod-selector :weapon="item" :mech="mech" @install="install($event)" />
    </cc-solo-dialog>
  </div>
</template>

<script lang="ts">
import Vue from 'vue'
import SlotCardBase from '../../_SlotCardBase.vue'
import WeaponSelector from './_WeaponSelector.vue'
import ModSelector from './_ModSelector.vue'
import ModInset from './_ModInset.vue'
import EquipmentOptions from '../../_EquipmentOptions.vue'
import EquipmentHeader from '../../_EquipmentHeader.vue'
import ShLockDialog from '../_ShLockDialog.vue'
import { MechWeapon, WeaponMod, WeaponSize, EquippableMount } from '@/class'

export default Vue.extend({
  name: 'weapon-slot-card',
  components: {
    SlotCardBase,
    WeaponSelector,
    ModSelector,
    ModInset,
    EquipmentOptions,
    EquipmentHeader,
    ShLockDialog,
  },
  props: {
    weaponSlot: {
      type: Object,
      required: true,
    },
    mech: {
      type: Object,
      required: true,
    },
    mount: {
      type: Object,
      required: true,
    },
    readonly: {
      type: Boolean,
    },
    intWeapon: {
      type: Boolean,
    },
  },
  data: () => ({
    stagedSH: null,
  }),
  computed: {
    item() {
      return this.weaponSlot.Weapon
    },
    color() {
      return this.mech.Frame.Manufacturer.Color
    },
  },
  methods: {
    equip(item: MechWeapon) {
      this.$refs.base.$refs.selectorDialog.hide()
      if (item.Size === WeaponSize.Superheavy) {
        this.equipSuperheavy(item)
      } else {
        this.weaponSlot.EquipWeapon(item, this.mech.Pilot)
      }
    },
    equipSuperheavy(item: MechWeapon) {
      this.stagedSH = item
      this.$refs.lockDialog.show()
    },
    finalizeSuperheavy(lockTarget: EquippableMount) {
      lockTarget.Lock(this.mount)
      this.weaponSlot.EquipWeapon(this.stagedSH, this.mech.Pilot)
      this.$refs.lockDialog.hide()
      this.stagedSH = null
    },
    install(mod: WeaponMod) {
      this.item.Mod = mod
      this.$refs.modDialog.hide()
    },
    uninstall() {
      this.item.Mod = null
    },
    remove() {
      if (this.item.Size === WeaponSize.Superheavy) {
        this.mech.ActiveLoadout.UnequipSuperheavy()
      }
      this.weaponSlot.UnequipWeapon()
    },
  },
})
</script>
