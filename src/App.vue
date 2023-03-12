<template>
  <v-app>
    <v-layout>
      <v-navigation-drawer v-model="drawer">
        <v-list density="compact" >
          <v-list-subheader>
            Фильтры
          </v-list-subheader>
          <v-divider></v-divider>
          <v-list-item>
            <v-combobox
                v-model="filterClasses"
                :items="classesNameArray"
                density="compact"
                label="Классы"
                item-title="title"
                item-value="id"
                :return-object="false"
                multiple
                hide-details
            ></v-combobox>
          </v-list-item>
          <v-list-item>
            <v-combobox
                v-model="filterMinLvl"
                :items="['0', 1, 2, 3, 4, 5, 6]"
                density="compact"
                label="Мин. уровень заклинания"
                hide-details
                clearable
            ></v-combobox>
          </v-list-item>
          <v-list-item>
            <v-combobox
                v-model="filterMaxLvl"
                :items="['0', 1, 2, 3, 4, 5, 6]"
                density="compact"
                label="Макс. уровень заклинания"
                hide-details
                clearable
            ></v-combobox>
          </v-list-item>
        </v-list>
      </v-navigation-drawer>
      <v-app-bar
          color="primary">
        <template v-slot:prepend>
          <v-app-bar-nav-icon @click.stop="drawer = !drawer"></v-app-bar-nav-icon>
        </template>
        <v-app-bar-title>
          <v-text-field
              clearable
              label="Поиск"
              variant="underlined"
              v-model="filterTitle"
          ></v-text-field>
        </v-app-bar-title>
        <v-icon></v-icon>
      </v-app-bar>
      <v-main>
        <v-container>
          <v-row>
            <v-col
                v-for="spell in spellsFiltred" v-bind:key="`spell-${spell.id}`"
                cols="12"
                md="6"
            >
              <v-card class="spell-card" style="overflow: auto;">
                <template v-slot:title>
                  <span class="spell-title">{{ spell.title }}</span>
                  <span class="spell-levels" v-for="className in spell.classesName" v-bind:key="`class-${className.id}`">
                    <v-icon
                        size="x-small"
                        :icon="className.id ? 'mdi-flash':'mdi-star'"
                        :color="className.id ? '#5700a7':'#cb0000'"
                    /><template v-if="className.level === undefined">{{ className.levelMin
                    }}<template v-if="className.levelMax">-{{ className.levelMax }}</template></template>
                    <template v-else>{{ className.level }}</template>
                  </span>
                </template>
                <template v-slot:text>
                  <strong>Школа:</strong> {{schools.get(spell.school).title}}
                  <template v-if="descriptors.get(spell.descriptor)">[{{descriptors.get(spell.descriptor).title}}]</template><br />
                  <template v-if="spell.castTime"><strong>Время сотворения:</strong> {{spell.castTime}}<br /></template>
                  <template v-if="spell.distance">
                    <strong>Дистанция:</strong> {{distances.get(spell.distance.id).title
                    }}<template v-if="spell.distance.value">, {{spell.distance.value}}</template>
                    <br />
                  </template>
                  <template v-if="spell.form"><strong>Область:</strong> {{spell.form}}<br /></template>
                  <template v-if="spell.effect"><strong>Эффект":</strong> {{spell.effect}}<br /></template>
                  <template v-if="spell.target"><strong>Цель:</strong> {{spell.target}}<br /></template>
                  <template v-if="spell.workTime"><strong>Длительность:</strong> {{spell.workTime}}<br /></template>
                  <template v-if="spell.challenge"><strong>Испытание:</strong> {{spell.challenge}}<br /></template>
                  <template v-if="spell.resist"><strong>Устойчивость к магии:</strong> {{spell.resist}}<br /></template>
                  <span v-if="spell.text" v-html="spell.text"></span>
                </template>
              </v-card>
            </v-col>
          </v-row>
        </v-container>
      </v-main>
    </v-layout>
  </v-app>
</template>

<script>
import axios from "axios";
export default {
  name: "App",
  data() {
    return {
      drawer: undefined,
      spells: [],
      classesName: undefined,
      classesNameArray: undefined,
      schools: undefined,
      descriptors: undefined,
      distances: undefined,
      filterTitle: '',
      filterMinLvl: undefined,
      filterMaxLvl: undefined,
      filterClasses: [0, 1],
    }
  },
  computed: {
    title() {
      return this.spells.length ? `Заклинаний: ${this.spells.length}`:'Загрузка...'
    },
    spellsFiltred() {
      return this.spells.filter(spell => {
        function prepareStr(str) {
          return str.toLowerCase().replace(/[\s.,%]/g,'');
        }
        if (prepareStr(spell.title).search(prepareStr(this.filterTitle)) === -1) {
          return false;
        }
        if (spell.classesName.every((classObj) => {
          if (this.filterClasses.find(crnClass => classObj.id === crnClass) !== undefined) {
            if (classObj.level === undefined) {
              console.log(spell.id, classObj.levelMin, this.filterMinLvl);
              if (this.filterMinLvl !== undefined && this.filterMinLvl !== null && classObj.levelMax < this.filterMinLvl) {
                return true;
              }
              if (this.filterMaxLvl !== undefined && this.filterMaxLvl !== null && classObj.levelMin > this.filterMaxLvl) {
                return true;
              }
            } else {
              if (classObj.level < this.filterMinLvl) {
                return true;
              }
              if (classObj.level > this.filterMaxLvl) {
                return true;
              }
            }
          } else {
            return true;
          }
        })) {
          return false;
        }
        return true;
      });
    },
  },
  mounted() {
    axios.get('/spells.json').then(res => {
      this.spells = res.data.spells;
      this.classesName = new Map(res.data.classesName.map((obj) => [obj.id, obj]));
      this.classesNameArray = res.data.classesName;
      this.schools = new Map(res.data.schools.map((obj) => [obj.id, obj]));
      this.descriptors = new Map(res.data.descriptors.map((obj) => [obj.id, obj]));
      this.distances = new Map(res.data.distances.map((obj) => [obj.id, obj]));
    });
  }
}
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Roboto+Condensed&family=Roboto:wght@300;400;500&display=swap');
div {
  font-family: 'Roboto', sans-serif;
  font-weight: 300;
}
strong {
  font-weight: 500;
}
.spell-title {
  font-family: 'Roboto Condensed', sans-serif;
  text-transform: uppercase;
  font-weight: 400;
  margin-right: 8px;
}
.spell-levels {
  font-weight: 300;
  font-size: 20px;
}
.spell-card {
  min-height: 500px;
  max-height: 500px;
  overflow: auto;
}
</style>