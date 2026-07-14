<template>
  <q-page class="column" :padding="$q.screen.gt.sm">
    <q-expansion-item :default-opened="!app.char.kin"
      :label="`${app.char.name}${app.char.kin ? ', ' + t('kins.' + app.char.kin, app.char.kin) : ''}${app.char.profession ? ', ' + t('professions.' + app.char.profession, app.char.profession) : ''}`"
      header-class="text-h5">
      <div class="row justify-between q-gutter-sm q-px-sm">
        <div class="col">
          <q-input class="row" :label="t('ui.name')" v-model="app.char.name" dense />

          <div class="row">
            <q-select class="col" options-selected-class="text-purple-2" :label="t('ui.age')" v-model="app.char.age"
              :options="Object.values(Ages).map(age => ({ label: t('ages.' + age), value: age }))" dense emit-value map-options />
            <q-input class="col" :label="t('ui.movement')" type="number" v-model.number="app.char.movement" dense />
          </div>
        </div>

        <div class="col">
          <div class="row">
            <q-select class="col" options-selected-class="text-purple-2" :label="t('ui.kin')" v-model="app.char.kin"
              :options="Kins.map(k => ({ label: t('kins.' + k, k), value: k })).sort((a, b) => a.label.localeCompare(b.label))" dense emit-value map-options />
            <q-select class="col" options-selected-class="text-purple-2" :label="t('ui.profession')" v-model="app.char.profession"
              :options="Professions.map(p => ({ label: t('professions.' + p, p), value: p })).sort((a, b) => a.label.localeCompare(b.label))" dense emit-value map-options />
          </div>
          <q-input class="row" :label="t('ui.weakness')" v-model="app.char.weakness" dense />
        </div>
      </div>

      <q-input class="row q-px-sm" :label="t('ui.appearance')" v-model="app.char.appearance" dense autogrow borderless />
    </q-expansion-item>

    <q-separator />
    <div class="row justify-start q-px-sm q-mt-md q-mb-sm items-center">
      <div class="col-xs-4 col-sm-4 col-md-3 q-pr-sm">
        <points-block v-model="app.char.hp" :label="t('ui.hp')" />
      </div>
      <div class="col-xs-4 col-sm-4 col-md-3 q-px-sm">
        <points-block v-model="app.char.wp" :label="t('ui.wp')" />
      </div>
      <div class="col-xs-4 col-sm-4 col-md-5 flex items-center justify-start q-pl-md">
        <div class="text-bold text-h6">
          {{ t('ui.dmgBonusLabel') }}:
          <span class="q-ml-sm text-subtitle1 text-normal text-grey-5">
            {{ t('attributes.STR') }}: <span class="text-bold text-black">{{ app.dmgBonus(Attrs.STR) }}</span>, 
            {{ t('attributes.AGL') }}: <span class="text-bold text-black">{{ app.dmgBonus(Attrs.AGL) }}</span>
          </span>
        </div>
      </div>
    </div>


    <div class="row justify-evenly q-mb-md">
      <div class="col-12 text-center q-mb-sm text-h6 text-bold">
        {{ t('ui.sum') }}: {{ attributeSum }}/78
      </div>
      <div class="col-xs-4 col-sm-2 col-md-2">
        <char-attr :label="Attrs.STR" v-model="app.char.attributes.STR" />
      </div>

      <div class="col-xs-4 col-sm-2 col-md-2">
        <char-attr :label="Attrs.CON" v-model="app.char.attributes.CON" />
      </div>

      <div class="col-xs-4 col-sm-2 col-md-2">
        <char-attr :label="Attrs.AGL" v-model="app.char.attributes.AGL" />
      </div>

      <div class="col-xs-4 col-sm-2 col-md-2">
        <char-attr :label="Attrs.INT" v-model="app.char.attributes.INT" />
      </div>

      <div class="col-xs-4 col-sm-2 col-md-2">
        <char-attr :label="Attrs.WIL" v-model="app.char.attributes.WIL" />
      </div>

      <div class="col-xs-4 col-sm-2 col-md-2">
        <char-attr :label="Attrs.CHA" v-model="app.char.attributes.CHA" />
      </div>
    </div>

    <q-separator />
    <q-tabs v-model="tab" align="justify" dense>
      <q-tab name="skills" :label="t('ui.skills')" />
      <q-tab name="combat" :label="t('ui.combat')" />
      <q-tab name="abilities" :label="t('ui.abilitiesSpells')" />
      <q-tab name="gear" :label="t('ui.gear')" />
      <q-tab name="log" :label="t('ui.log')" />
    </q-tabs>

    <q-tab-panels v-model="tab" class="rounded-borders" swipeable>
      <!--SKILLS-->
      <q-tab-panel name="skills" class="q-pa-none">
        <skills-tab />
      </q-tab-panel>

      <!--COMBAT-->
      <q-tab-panel name="combat" class="q-pa-none">
        <combat-tab />
      </q-tab-panel>

      <!--ABILITIES & SPELLS-->
      <q-tab-panel name="abilities" class="q-pa-none">
        <abilities-tab />
      </q-tab-panel>

      <!--GEAR-->
      <q-tab-panel name="gear" class="q-pa-none">
        <gear-tab />
      </q-tab-panel>

      <!--LOG-->
      <q-tab-panel name="log">
        <log-tab />
      </q-tab-panel>
    </q-tab-panels>
  </q-page>
</template>

<script lang="ts" setup>
import { ref, computed, watch } from 'vue';
import { useI18n } from 'vue-i18n';

import type { Attr } from 'src/components/models';
import { Ages, Attrs } from 'src/components/models';

import { useQuasar } from 'quasar';
import { useCharacterStore } from 'src/stores/character';

import CharAttr from 'src/components/CharAttr.vue';
import PointsBlock from 'src/components/PointsBlock.vue';
import SkillsTab from 'src/components/SkillsTab.vue';
import CombatTab from 'src/components/CombatTab.vue';
import AbilitiesTab from 'src/components/AbilitiesTab.vue';
import GearTab from 'src/components/GearTab.vue';
import LogTab from 'src/components/LogTab.vue';

const app = useCharacterStore();
const { t, locale } = useI18n();
const tab = ref('skills');

const Kins = ['Human', 'Halfling', 'Dwarf', 'Elf', 'Mallard', 'Wolfkin'];
const Professions = [
  'Artisan',
  'Bard',
  'Fighter',
  'Hunter',
  'Knight',
  'Mage',
  'Merchant',
  'Scholar',
  'Thief',
  'Mariner',
  'Barbarian',
  'Demon Hunter',
  'Raider',
  'Assassin',
  'Swordsman',
  'Ranger',
];

const $q = useQuasar();
const attributeSum = computed((): number => {
  let total = 0;
  Object.keys(Attrs).forEach((attr) => (total += app.char.attributes[attr as Attr].score));
  return total;
});

watch(
  () => app.char?.attributes,
  (attrs) => {
    if (app.char && attrs) {
      const conScore = Number(attrs[Attrs.CON]?.score) || 0;
      const wilScore = Number(attrs[Attrs.WIL]?.score) || 0;

      if (app.char.hp.max !== conScore) {
        const diff = conScore - app.char.hp.max;
        app.char.hp.max = conScore;
        app.char.hp.current = Math.max(0, app.char.hp.current + diff);
      }

      if (app.char.wp.max !== wilScore) {
        const diff = wilScore - app.char.wp.max;
        app.char.wp.max = wilScore;
        app.char.wp.current = Math.max(0, app.char.wp.current + diff);
      }
    }
  },
  { deep: true, immediate: true }
);

const KinMovement: Record<string, number> = {
  Human: 10,
  Elf: 10,
  Dwarf: 8,
  Halfling: 8,
  Mallard: 8,
  Wolfkin: 12,
};

watch(
  () => app.char?.kin,
  (newKin) => {
    if (app.char && newKin && KinMovement[newKin] !== undefined) {
      app.char.movement = KinMovement[newKin];
    }
  }
);

interface AbilityTemplate {
  nameSv: string;
  nameEn: string;
  textSv: string;
  textEn: string;
  wp: number;
}

const ProfessionAbilities: Record<string, AbilityTemplate[]> = {
  Barbarian: [{ nameSv: 'Härdad', nameEn: 'Hardened', wp: 0, textSv: 'Kostnad: - (Passiv). Du kan motstå smärta och skada. Din maximala KP ökar med 2.', textEn: 'Cost: - (Passive). You can resist pain and injury. Your maximum HP increases by 2.' }],
  Bard: [{ nameSv: 'Tonkonst', nameEn: 'Music Talent', wp: 0, textSv: 'Kostnad: - (Passiv). Du kan spela instrument och sjunga för att inspirera andra.', textEn: 'Cost: - (Passive). You can play instruments and sing to inspire others.' }],
  'Demon Hunter': [{ nameSv: 'Skuggfjättrare', nameEn: 'Shadow Binder', wp: 0, textSv: 'Kostnad: - (Passiv). Förmåga att binda eller försvaga demoner.', textEn: 'Cost: - (Passive). Ability to bind or weaken demons.' }],
  Artisan: [
    { nameSv: 'Garverimästare', nameEn: 'Master Tanner', wp: 0, textSv: 'Garverimästare specialisering.', textEn: 'Master Tanner specialization.' },
    { nameSv: 'Mästersmed', nameEn: 'Master Blacksmith', wp: 0, textSv: 'Mästersmed specialisering.', textEn: 'Master Blacksmith specialization.' },
    { nameSv: 'Mästersnikare', nameEn: 'Master Carpenter', wp: 0, textSv: 'Mästersnikare specialisering.', textEn: 'Master Carpenter specialization.' },
    { nameSv: 'Stenhuggare', nameEn: 'Stonemason', wp: 0, textSv: 'Stenhuggare specialisering.', textEn: 'Stonemason specialization.' }
  ],
  Raider: [{ nameSv: 'Född på slagfältet', nameEn: 'Born on the Battlefield', wp: 0, textSv: 'Kostnad: - (Passiv). Ökar din stridsvana.', textEn: 'Cost: - (Passive). Increases your combat readiness.' }],
  Hunter: [
    { nameSv: 'Följeslagare', nameEn: 'Companion', wp: 0, textSv: 'Du har ett troget djur som följeslagare.', textEn: 'You have a faithful animal companion.' },
    { nameSv: 'Blodhund', nameEn: 'Bloodhound', wp: 0, textSv: 'Förmåga att spåra fiender över långa avstånd.', textEn: 'Ability to track enemies over long distances.' }
  ],
  Fighter: [{ nameSv: 'Stridsvana', nameEn: 'Combat Ready', wp: 0, textSv: 'Kostnad: - (Passiv). Du får dra initiativkort med fördel.', textEn: 'Cost: - (Passive). You draw initiative cards with advantage.' }],
  Scholar: [{ nameSv: 'Intuition', nameEn: 'Intuition', wp: 0, textSv: 'Kostnad: - (Passiv). Förmåga att ana dolda sammanhang.', textEn: 'Cost: - (Passive). Ability to sense hidden connections.' }],
  Assassin: [
    { nameSv: 'Alltid en flyktväg', nameEn: 'Always an Escape Route', wp: 0, textSv: 'Du hittar alltid en väg ut ur farliga situationer.', textEn: 'You always find a way out of dangerous situations.' },
    { nameSv: 'Blottlägga svagheter', nameEn: 'Expose Weaknesses', wp: 0, textSv: 'Du kan hitta och utnyttja fiendens svagheter.', textEn: 'You can find and exploit enemy weaknesses.' }
  ],
  Mage: [{ nameSv: 'Magisk talang', nameEn: 'Magic Talent', wp: 0, textSv: 'Kostnad: - (Passiv). Krävs för att kunna kasta besvärjelser.', textEn: 'Cost: - (Passive). Required to cast spells.' }],
  Merchant: [{ nameSv: 'Skattletare', nameEn: 'Treasure Hunter', wp: 0, textSv: 'Kostnad: - (Passiv). Förmåga att finna värdefulla ting.', textEn: 'Cost: - (Passive). Ability to find valuable things.' }],
  Knight: [{ nameSv: 'Förkämpe', nameEn: 'Champion', wp: 0, textSv: 'Kostnad: - (Passiv). Du kan ta en smäll för en allierad.', textEn: 'Cost: - (Passive). You can take a hit for an ally.' }],
  Mariner: [{ nameSv: 'Sjöben', nameEn: 'Sea Legs', wp: 0, textSv: 'Kostnad: - (Passiv). Får aldrig nackdel av att stå på gungande däck.', textEn: 'Cost: - (Passive). Never take a bane from standing on a swaying deck.' }],
  Swordsman: [{ nameSv: 'Finess', nameEn: 'Finesse', wp: 0, textSv: 'Kostnad: - (Passiv). Du kan använda SMI istället för STY för vissa vapen.', textEn: 'Cost: - (Passive). You can use AGL instead of STR for certain weapons.' }],
  Thief: [{ nameSv: 'Tjuvhugg', nameEn: 'Backstab', wp: 0, textSv: 'Kostnad: - (Passiv). Gör extra skada vid överraskningsanfall.', textEn: 'Cost: - (Passive). Deal extra damage during surprise attacks.' }],
  Ranger: [{ nameSv: 'Vildmarksexpert', nameEn: 'Wilderness Expert', wp: 0, textSv: 'Kostnad: - (Passiv). Underlättar överlevnad i vildmarken.', textEn: 'Cost: - (Passive). Facilitates survival in the wilderness.' }]
};

const AllStartingAbilities = [
  'Härdad', 'Hardened',
  'Tonkonst', 'Music Talent',
  'Skuggfjättrare', 'Shadow Binder',
  'Garverimästare', 'Master Tanner',
  'Mästersmed', 'Master Blacksmith',
  'Mästersnikare', 'Master Carpenter',
  'Stenhuggare', 'Stonemason',
  'Född på slagfältet', 'Born on the Battlefield',
  'Följeslagare', 'Companion',
  'Blodhund', 'Bloodhound',
  'Stridsvana', 'Combat Ready',
  'Intuition', 'Intuition',
  'Alltid en flyktväg', 'Always an Escape Route',
  'Blottlägga svagheter', 'Expose Weaknesses',
  'Magisk talang', 'Magic Talent',
  'Skattletare', 'Treasure Hunter',
  'Förkämpe', 'Champion',
  'Sjöben', 'Sea Legs',
  'Finess', 'Finesse',
  'Tjuvhugg', 'Backstab',
  'Vildmarksexpert', 'Wilderness Expert'
];

watch(
  () => app.char?.profession,
  (newProf) => {
    if (!app.char) return;

    if (!app.char.abilities) {
      app.char.abilities = [];
    }

    // 1. Remove any previously added starting heroic ability
    app.char.abilities = app.char.abilities.filter(
      (a) => !AllStartingAbilities.includes(a.name)
    );

    if (!newProf) return;

    const abilities = ProfessionAbilities[newProf];
    if (abilities && abilities.length > 0) {
      if (abilities.length === 1) {
        // Only one ability, add it directly
        const chosen = abilities[0]!;
        app.char.abilities.push({
          name: locale.value === 'sv' ? chosen.nameSv : chosen.nameEn,
          wp: chosen.wp,
          text: locale.value === 'sv' ? chosen.textSv : chosen.textEn
        });
      } else {
        // Multiple options, show dialog
        const options = abilities.map((a, index) => ({
          label: locale.value === 'sv' ? a.nameSv : a.nameEn,
          value: String(index)
        }));

        $q.dialog({
          title: locale.value === 'sv' ? 'Välj startförmåga' : 'Choose Starting Ability',
          message: locale.value === 'sv' 
            ? 'Välj din startförmåga för detta yrke:' 
            : 'Choose your starting heroic ability for this profession:',
          options: {
            type: 'radio',
            model: '0',
            items: options
          },
          cancel: true,
          persistent: true
        }).onOk((selectedValue: string) => {
          const selectedIndex = Number(selectedValue);
          const chosen = abilities[selectedIndex]!;
          app.char.abilities.push({
            name: locale.value === 'sv' ? chosen.nameSv : chosen.nameEn,
            wp: chosen.wp,
            text: locale.value === 'sv' ? chosen.textSv : chosen.textEn
          });
        });
      }
    }
  }
);
</script>
