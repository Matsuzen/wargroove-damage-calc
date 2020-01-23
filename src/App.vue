<template>
  <div id="app">
    <div 
      class="swap"
      @click="swapPositions()"
      >
      <i class="fas fa-exchange-alt"></i>
    </div>
    <div>
      <div class="calculator">
        <UnitPanel 
          :position="'Attacker'"
          :posObject="attacker"
          :menuToggles="menuToggles"
          :factions="factions"
          :terrains="terrains"
          :unitNames="unitNames"
          :grooves="grooves"
          :damage="attDamage"
          @selectEl="selectEl"
          @toggleMenu="toggleMenu"
          />
      </div>
    </div>
  </div>
</template>

<script>
import UnitPanel from "@/components/UnitPanel.vue";

export default {
  name: 'app',
  components: {
    unitPanel
  },
  data() {
    return {
      attacker: {
        crit: 1,
        critCondition: false,
        faction: "cherrystone",
        groove: "",
        hp: 100,
        terrain: "road",
        unit: "soldier"
      },
      attUnits: {},
      defender: {
        crit: 1,
        critCondition: false,
        faction: "fellheim",
        groove: "",
        hp: 100,
        terrain: "road",
        unit: "soldier"
      },
      defenderUnits: {},
      factions: ["cherrystone", "fellheim", "floran", "heavensong"],
      grooves: [
        {
          co: "Ragna",
          setHp: 65
        },
        {
          co: "Ryota",
          setHp: 45
        }
      ],
      lastMenu: {
        menu: "",
        position: ""
      },
      noDefUnits: {
        aeronaut: true,
        balloon: true,
        skyrider: true,
        dragon: true,
        hq: true,
        building: true
      },
      showGrooveTip: false,
      terrains: ["river", "beach", "road", "deepsea", "plain", "sea", "flagstone", "reefs", "forest", "mountain"],
      terrainValues: {
        river: -2,
        beach: -1,
        road: 0,
        deepsea: 0,
        plain: 1,
        sea: 1,
        flagstone: 2,
        reefs: 2,
        forest: 3,
        mountain: 4
      },
      tipGroove: "",
      menuToggles: {
        attacker: {
          faction: false,
          unit: false,
          terrain: false
        },
        defender: {
          faction: false,
          unit: false,
          terrain: false
        }
      },
      units: {}
    
    }
  },
  methods: {
    selectEl({ el, value, position, menu }) {
      this[position][menu] = value;
      this.toggles[position][menu] = false;
      this.lastMenu.menu = "";
    },
    adjustHP(position) {
      if(this[position].hp > 100) {
        this[position].hp = 100;
      }
      if(this[position].hp < 0) {
        this[position].hp = 0;
      }
    },
    calcAttack(luckDamage) {
      const { attUnit, attGroove, attHp } = this.attacker;
      const { defUnit, defTerrain } = this.defender;

      let terrainDef = 
    }
  }
  
}
</script>

<style>

</style>
