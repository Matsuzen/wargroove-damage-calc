<template>
  <div class="unit-panel">
    <div :class="`${posLower}-panel`">
      <div :class="`${posLower}`">
        <div class="panel-header">
          <h2>{{ position }}</h2>
          <div 
            class="current-faction"
            @click="toggleMenu(posLower, 'faction')">
            <img 
              :src="require(`@/assets/images/${posObject.faction}/emblem.png`)"
              :alt="`${posObject.faction}'s emblem'`">
          </div>
          <div
            v-show="menuToggles[posLower].faction"
            class="faction-menu">
              <img 
                v-for="faction in factions"
                :key="faction"
                :src="require(`@/assets/images/${faction}/emblem.png`)"
                :alt="`${faction}'s emblem'`"

                @click="selectEl(faction, posLower, 'faction')">

          </div>
        </div>

        <div class="panel-unit-terrain">
          <div 
            v-show="showGrooveTip"
            class="groove-tip bold">
            Display {{ tipGroove }}'s Groove damage.
          </div>

          <div
            v-show="menuToggles[posLower].unit"
            class="unit-menu">
            <img 
              v-for="unitName in unitNames"
              :key="unitName"
              :src="unitName !== 'commander' ? 
                (require(`@/assets/images/${posObject.faction}/${unitName}.png`)) : 
                (require(`@/assets/images/${posObject.faction}/commander.png`))"
              :class="`${unitName}, ${posLower}`"
              @click="selectEl(unitName, posLower, 'unit')"
              :alt="`${unitName}'s sprite'`">
              
          </div>

          <div class="panel-unit">
            <div
              class="current-unit"
              @click="toggleMenu(posLower, 'unit')">
              <img 
                :src="require(`@/assets/images/${posObject.faction}/${posObject.unit}.png`)" 
                :alt="`${posObject.unit}'s sprite'`"
                :class="{ grayscale: posObject.groove }">
            </div>

            <div
              v-if="position === 'Attacker' && posObject.unit === 'commander'"
              class="groove-cos">
              <img
                :src="require(`@/assets/images/grooves/${posObject.groove.toLowerCase() || 'ryota'}.png`)"
                @click="toggleMenu(posLower, 'groove')"
                :alt="`${posLower.groove}'s groove'`"
                class="current-groove hover-scale"
                :class="{ grayscale: !posObject.groove }">

              <div 
                class="groove-menu"
                v-show="menuToggles[posLower].groove">
                <img
                  v-for="groove in grooves"
                  :key="groove.name"
                  :src="require(`@/assets/images/grooves/${groove.name.toLowerCase()}.png`)"
                  @click="selectEl(groove.name, posLower, 'groove')"
                  :alt="`${groove.name}'s groove'`">
              </div>
            </div>

          </div>

          <div class="panel-terrain">
            <div
              @click="toggleMenu(posLower, 'terrain')"
              class="current-terrain">
              <img 
                :src="require(`@/assets/images/terrain/${posObject.terrain}.png`)" 
                alt="Unit's terrain">

            </div>
            <div
              v-show="menuToggles[posLower].terrain"
              class="terrain-menu">
              <img 
                v-for="terrain in terrains"
                :key="terrain"
                @click="selectEl(terrain, posLower, 'terrain')"
                :src="require(`@/assets/images/terrain/${terrain}.png`)"
                :class="terrain">
            </div>
          </div>
        </div>
      </div>
      <!-- HP, crit and blade dash count, Emeric crystal -->
      <div class="panel-others">
        <div 
          class="panel-hp"
          v-if="posObject.groove !== 'Ryota'">
          <label :for="`${posLower}-hp`">
            <img src="@/assets/images/icons/heart.png" alt="Unit's HP">
          </label>
          <input 
            type="number"
            :id="`${posLower}-hp`"
            v-model="changeHp"
            min=0
            max=100>
        </div>

        <div
          class="panel-dash"
          v-else-if="position === 'Attacker' && posObject.groove === 'Ryota'">
          <label :for="`${posLower}-dash`">
            <img src="" alt="Dash">
          </label>
          <input 
            type="number"
            :id="`${posLower}-dash`"
            v-model="changeDashes"
            min=1>
        </div>

        <div class="panel-crystal">
          <label :for="`${posLower}-crystal`">
            <img 
              src="@/assets/images/icons/crystal.png"
              :class="{ selected: posObject.emericCrystal}"
              class="clickable crystal"
              alt="Emeric Crystal">
          </label>
          <input 
            type="checkbox"
            :id="`${posLower}-crystal`"
            class="hidden-input"
            @click="$emit('toggleAttribute', { position: posLower, attribute: 'emericCrystal' })">
        </div>

        <div class="panel-crit">
          <label :for="`${posLower}-crit`">
            <img 
              src="@/assets/images/icons/critical.png"
              :class="{ selected: posObject.critCondition }"
              class="clickable"
              alt="Critical toggle">
          </label>
          <input 
            type="checkbox"
            :id="`${posLower}-crit`"
            class="hidden-input"
            @click="$emit('toggleAttribute', { position: posLower, attribute: 'critCondition' })">
          <span
            class="crit-modifier"
            :class="{ bold: posObject.critCondition }">
            {{ posObject.crit }}
          </span>
        </div>
      </div>

      <div
        class="panel-damage"
        v-show="damage"> 
        <div class="damage-display">
          {{ damage.min }} - {{ damage.max }} <br>
          AVG: {{ damage.avg }}
        </div>
      </div>

    </div>
  </div>
</template>
<script>
export default {

  props: ["position", "posObject", "menuToggles", "factions" , "terrains" , "unitNames", "grooves", "damage"],
  data() {
    return {
      posLower: this.position.toLowerCase(),
      showGrooveTip: false,
      tipGroove: ""
    }
  },
  computed: {
    changeHp: {
      get() {
        return this.posObject.hp;
      },
      set(newHp) {
        this.$emit("changeHp", { position: this.posLower, newHp })
      }
    },
    changeDashes: {
      get() {
        return this.posObject.dashes;
      },
      set(newDashes) {
        this.$emit("changeDashes", newDashes)
      }
    }
  },
  methods: {
    selectEl(value, position, menu) {
      this.$emit("selectEl", { value, position, menu })
    },
    toggleMenu(position, menu) {
      this.$emit("toggleMenu", { position, menu })
    },
    displayGrooveTip(groove) {
      this.tipGroove = groove;
      this.showGrooveTip = true;
    }
  }
}
</script>
<style>

</style>


