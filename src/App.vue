<template>
  <div id="app">
    <div class="versions">
      <div 
        @click="changeVersion(2)"
        :class="{ bold: version === 2 }">
        Wargroove v2.0
      </div>
      <div 
        @click="changeVersion(1)"
        :class="{ bold: version === 1 }">
        Wargroove v1.0
      </div>
    </div>
    <div 
      class="swap"
      @click="swapPositions()"
      >
      <i class="fas fa-exchange-alt"></i>
    </div>
      <div class="calculator">
        <UnitPanel 
          :position="'Attacker'"
          :posObject="attacker"
          :menuToggles="menuToggles"
          :factions="factions"
          :terrains="terrains"
          :unitNames="attUnits"
          :grooves="grooves"
          :damage="attDamage"
          @selectEl="selectEl"
          @toggleMenu="toggleMenu"
          @toggleCrit="toggleCrit"
          @changeGroove="changeGroove"
          @changeHp="changeHp"
          />

        <UnitPanel 
          :position="'Defender'"
          :posObject="defender"
          :menuToggles="menuToggles"
          :factions="factions"
          :terrains="terrains"
          :unitNames="defUnits"
          :grooves="grooves"
          :damage="counterDamage"
          @selectEl="selectEl"
          @toggleMenu="toggleMenu"
          @toggleCrit="toggleCrit"
          @changeHp="changeHp"
          />
      </div>
    <Footer />
  </div>
</template>

<script>
import UnitPanel from "@/components/UnitPanel.vue";
import Footer from "@/components/Footer.vue";
import damageTableV1 from "@/assets/damagetables/1.0.json";
import damageTableV2 from "@/assets/damagetables/2.0.json"

export default {
  name: 'app',
  components: {
    UnitPanel,
    Footer
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
      damageTables: {
        //Versions
        1: damageTableV1,
        2: damageTableV2
      },
      defender: {
        crit: 1,
        critCondition: false,
        faction: "fellheim",
        groove: "",
        hp: 100,
        terrain: "road",
        unit: "soldier"
      },
      defUnits: {},
      factions: ["cherrystone", "fellheim", "floran", "heavensong"],
      grooves: {
        Ryota: {
          name: "Ryota",
          setHp: 45
        },
        Ragna: {
          name: "Ragna",
          setHp: 65
        }
      },
      lastMenu: {
        menu: "",
        position: ""
      },
      //List used to ignore defense for damage calculation
      noDefUnits: {
        aeronaut: true,
        balloon: true,
        skyrider: true,
        dragon: true,
        hq: true,
        building: true
      },
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
      //Default to WG 2.0 for damage values
      version: 2
    }
  },
  methods: {
    selectEl({ value, position, menu }) {
      this[position][menu] = value;
      this.menuToggles[position][menu] = false;
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

    calcAttack(attacker, defender, luckDamage, remainingHp) {
      if(remainingHp === 0) return 0;
      let { unit: attUnit, groove: attGroove, crit: attCrit, hp: attHp } = attacker;
      let { unit: defUnit, terrain: defTerrain, hp: defHp } = defender;

      //Game works based on average damage so add 5
      const baseDamage = (!attGroove ? this.units[attUnit].damage[defUnit] : this.units.grooves.damage[defUnit]) + 5;

      //Set defense to 0 for air units and buildings
      let terrainDef = this.noDefUnits[defUnit] ? 0 : this.terrainValues[defTerrain];
      
      //Set def HP to 100 for negative defense
      if(terrainDef <= 0) {
        defHp = 100;
      }

      //Set proper hp for grooves
      if(attGroove) {
        attHp = this.grooves[attGroove].setHp;
      }

      //Set remaining hp for counter attacks
      if(remainingHp) {
        attHp = remainingHp
      }

      const damageFunctions = {
        1: this.calcV1,
        2: this.calcV2
      }
      const calcDamage = damageFunctions[this.version];

      return calcDamage(attCrit, baseDamage, luckDamage, attHp, terrainDef, defHp).toFixed(2);
    },
    //attLuckDamage is the luck damage the attacker had during his attack (min, max)
    calcCounter(attLuckDamage, luckDamage) {
      //Set the remaining hp of the defender after the attack and run calcAttack
      const { hp: defHp } = this.defender;

      const { max, min, avg } = this.attDamage;

      const remainingHps = {
        min: defHp - max,
        max: defHp - min,
        avg: defHp - avg
      }
      const remainingHp = remainingHps[attLuckDamage];

      let counterDmg = this.calcAttack(this.defender, this.attacker, luckDamage, remainingHp);

      if(counterDmg < 0) counterDmg = 0;

      return counterDmg;
    },
    //Formula prior to 2.0: crit multiplies both base damage and luck damage
    calcV1(attCrit, baseDamage, luckDamage, attHp, terrainDef, defHp) {
      return attCrit * (baseDamage + luckDamage) * attHp / 100 * (1 - 0.1 * terrainDef * defHp / 100);
    },
    //Formula after 2.0: crit multiplies only base damage
    calcV2(attCrit, baseDamage, luckDamage, attHp, terrainDef, defHp) {
      return (attCrit * baseDamage + luckDamage) * attHp / 100 * (1 - 0.1 * terrainDef * defHp / 100);
    },

    swapPositions() {
      this.attacker.groove = "";
      const tempPosition = this.attacker;
      this.attacker = this.defender;
      this.defender = tempPosition;
    },

    toggleMenu({ position, menu }) {
      if(this.lastMenu.menu) {
        this.menuToggles[this.lastMenu.position][this.lastMenu.menu] = false;
      }
      if(position != this.lastMenu.position || menu != this.lastMenu.menu) {
        this.menuToggles[position][menu] = true;
        this.lastMenu.position = position;
        this.lastMenu.menu = menu;
      } else {
        this.lastMenu.position = "";
        this.lastMenu.menu = "";
      }
    },

    updateCrit(position) {
      const { unit: unitName, critCondition } = this[position];
      if(unitName && critCondition) {
        this[position].crit = this.units[unitName].crit;
      } else {
        this[position].crit = 1;
      }
    },
    toggleCrit(position) {
      this[position].critCondition = !this[position].critCondition;
    },

    changeGroove({ position, co }) {
      if(this[position].groove === co) co = "";
      this[position].groove = co;
    },

    changeHp({ position, newHp }) {
      this[position].hp = newHp;
    },

    changeVersion(version) {
      this.version = version;
    }
  },

  computed: {
    attDamage() {
      const { unit: attUnit, groove: attGroove } = this.attacker;
      const { unit: defUnit } = this.defender;
      
      const units = this.units;

      if(attUnit && defUnit) {
        const baseDamage = !attGroove ? units[attUnit].damage[defUnit] : units.grooves.damage[defUnit];

        if(typeof(baseDamage) === "number") {
          const avg = this.calcAttack(this.attacker, this.defender, 0);
          const min = this.calcAttack(this.attacker, this.defender, -5);
          const max = this.calcAttack(this.attacker, this.defender, 5)
          
          return { min, max, avg };

        }
        else {
          return {
            avg: 0,
            min: 0,
            max: 0
          }
        }
      }
      return 0;
    },
    counterDamage() {
      const { unit: attUnit, groove: attGroove } = this.attacker;
      const { unit: defUnit } = this.defender;

      const units = this.units;

      if(attUnit && defUnit && units[defUnit] && !attGroove) {
        const baseDamage = units[defUnit].damage[attUnit];

        if(typeof(baseDamage) === "number" && units[defUnit].canCounter) {
          const avg = this.calcCounter("avg", 0);
          const min = this.calcCounter("min", -5);
          const max = this.calcCounter("max", 5);

          return { avg, min, max };
        }
        else {
          return {
            avg: 0,
            min: 0,
            max: 0
          }
        }
      }
      return {
        avg: 0,
        min: 0,
        max: 0
      }
    },
    units() {
      return this.damageTables[this.version];
    }
  
  },
  watch: {
    "attacker.critCondition": function() {
      this.updateCrit("attacker")
    },
    "attacker.unit": function() {
      this.updateCrit("attacker");
      if(this.attacker.unit !== "commander") this.attacker.groove = "";
    },
    /* "attacker.hp": function() {
      this.adjustHP("attacker");
    }, */
    "defender.critCondition": function() {
      this.updateCrit("defender");
    },
    "defender.unit": function() {
      this.updateCrit("defender");
    },
    version() {
      //Store only units that can attack in an array
      this.attUnits = Object.keys(this.units).slice(0, -3);
      //Store every unit's name in defenders
      this.defUnits = Object.keys(this.units.soldier.damage);
    }
    /* "defender.hp": function() {
      this.adjustHP("defender");
    } */
  },
  mounted() {
    //Store only units that can attack in an array
    this.attUnits = Object.keys(this.units).slice(0, -3);
    //Store every unit's name in defenders
    this.defUnits = Object.keys(this.units.soldier.damage);
  }
  
}
</script>

<style>
  body {
    background: url("assets/images/wargroove-bg.png") no-repeat fixed center center / cover;
  }
</style>
