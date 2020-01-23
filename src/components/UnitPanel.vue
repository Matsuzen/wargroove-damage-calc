<template>
  <div class="unit-panel">
    <div :class="`${position.toLowerCase()}-panel`">
      <div :class="`${position-toLoserCase()}`">
        <div class="panel-header">
          <h2>{{ position }}</h2>
          <div 
            class="current-faction"
            @click="toggleMenu(position.toLowerCase(), 'faction')">
            <img 
              :src="`public/images/${posObject.faction}/emblem.png`"
              :alt="`${posObject.faction}'s emblem'`">
          </div>
          <div
            v-show="menuToggles[position.toLowerCase()]"
            class="faction-menu">
              <img 
                v-for="faction in factions"
                :key="faction"
                :src="`public/images/${faction}/emblem.png`"
                :alt="`${faction}'s emblem'`"

                @click="selectEl($event.currentTarget, faction, position.toLowerCase(), 'faction')">

          </div>
        </div>

        <div class="panel-unit-terrain">
          <div
            v-if="position === 'Attacker'"
            v-show="posObject.unit === 'commander'"
            class="groove-cos">
            <img
              v-for="groove in grooves"
              :key="groove.co"
              :src="`public/images/icons/${groove.co.toLowerCase()}.png`"
              :class="{ grayscale: posObject.groove !== groove.co }"
              @click="$emit('changeGroove', groove.co)"
              @mouseover="displayGrooveTip(groove.co)"
              @mouseleave="showGrooveTip = false"
              :alt="`${groove.co}'s groove'`">
          </div>
          <div 
            v-show="showGrooveTip"
            class="groove-tip bold">
            Display {{ tipGroove }}'s Groove damage.
          </div>

          <div
            v-show="menuToggles[position.toLowerCase()].unit"
            class="unit-menu">
            <img 
              v-for="unitName in unitNames"
              :key="unitName"
              :src="unitName !== 'commander' ? 
                (`public/images/${posObject.faction}/${unitName}.png`) : 
                (`public/images/${posObject.faction}/commander.png`)"
              :class="`${unitName}, ${position.toLowerCase()}`"
              @click="selectEl($event.currentTarget, unitName, position.toLowerCase(), 'unit')"
              :alt="`${unitName}'s sprite'`">
          </div>

          <div class="panel-unit">
            <div
              class="current-unit"
              @click="toggleMenu(position.toLowerCase(), 'unit')">
              <img 
                :src="`public/images/${posObject.faction}/${posObject.unit}.png`" 
                :alt="`${posObject.unit}'s sprite'`"
                :class="{ grayscale: posObject.groove }">
            </div>
          </div>

          <div class="panel-terrain">
            <div
              @click="toggleMenu(position.toLowerCase(), 'terrain')"
              class="current-terrain">
              <img 
                :src="`public/images/terrain/${posObject.terrain}.png`" 
                alt="Unit's terrain">

            </div>
            <div
              v-show="menuToggles[position.toLowerCase()].terrain"
              class="terrain-menu">
              <img 
                v-for="terrain in terrains"
                :key="terrain"
                @click="selectEl($event.currentTarget, terrain, position.toLowerCase(), 'terrain')"
                :src="`@/assets/terrain/${terrain}.png`"
                :class="terrain">
            </div>
          </div>
        </div>
      </div>
      <!-- HP and crit -->
      <div class="panel-others">
        <div class="panel-hp">
          <label :for="`${position.toLowerCase()}-hp`">
            <img src="@/assets/icons/heart.png" alt="Unit's HP">
          </label>
          <input 
            type="number"
            :id="`${posLower}-hp`"
            v-model="changeHp"
            min=0
            max=100>
        </div>

        <div class="panel-crit">
          <label :for="`${posLower}-crit`">
            <img 
              src="@/assets/icons/critical.png" 
              alt="Critical toggle">
          </label>
          <input 
            type="checkbox"
            :id="`${posLower}-crit`"
            @click="$emit('toggleCrit', !posObject.critCondition)">
          <span
            class="crit-modifier"
            :class="{ bold: posObjet.critCondition }">
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
    }
  },
  computed: {
    changeHp: {
      get() {
        return this.posObject.hp;
      },
      set(newHp) {
        this.$emit("changeHp", newHp)
      }
    }
  },
  methods: {
    selectEl(el, value, position, menu) {
      this.$emit("selectEl", { el, value, position, menu })
    },
    toggleMenu(position, menu) {
      this.$emit("toggleMenu", { position, menu })
    }
    
  }
}
</script>
<style>

</style>


