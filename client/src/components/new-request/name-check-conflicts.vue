<template>
  <v-data-table :headers="headers"
                fixed
                hide-default-header hide-default-footer
                :items="items"
                item-key="problem"
                show-expand>
    <template v-slot:item="{ item, expand, isExpanded }">
      <tr @click="expand(!isExpanded)">
        <td>
          <v-icon :color="item.iconColor">{{ item.icon }}</v-icon>
        </td>
        <td>
          <span v-html="item.problem"/>
          <b v-for="(word, index) in item.words" :key=index>
            <span v-if="index === 0"> {{ word }}</span>
            <span v-else>, {{ word }}</span>
          </b>
        </td>
        <td v-if="item.count">
          <v-chip :color="item.iconColor" dark>{{ item.count }}</v-chip>
        </td>
        <td v-if="!isExpanded && item.expandLabel">
          <v-row no-gutters>
            <v-col>
              <span class="expand-label">{{ item.expandLabel.closed }}</span>
            </v-col>
            <v-col>
              <v-icon color="blue">mdi-chevron-down</v-icon>
            </v-col>
          </v-row>
        </td>
        <td v-else-if="item.expandLabel">
          <v-row no-gutters>
            <v-col>
              <span class="expand-label">{{ item.expandLabel.open }}</span>
            </v-col>
            <v-col>
              <v-icon color="blue">mdi-chevron-up</v-icon>
            </v-col>
          </v-row>
        </td>
      </tr>
    </template>
    <template v-slot:expanded-item="{ headers, item }">
      <td v-if="item.expandedList" :colspan="headers.length">
        <v-row no-gutters>
          <QuickSearchNames :namesList="item.expandedList"/>
        </v-row>
      </td>
    </template>
  </v-data-table>
</template>

<script lang="ts">
import { Component, Prop, Vue } from 'vue-property-decorator'
import QuickSearchNames from '@/components/new-request/quick-search-names.vue'

@Component({
  components: { QuickSearchNames }
})
export default class NameCheckConflicts extends Vue {
  config = {
    modules: {
      toolbar: false
    },
    placeholder: '',
    scrollingContainer: false
  }
  @Prop() items: Array<{
    count: number,
    expanded: string,
    expandedList: Array<{ name: string, type: string }>,
    expandLabel: {
      open: string,
      closed: string
    }
    icon: string,
    iconColor: string,
    problem: string,
    words: Array<string>
  }>
  private headers = [
    { text: 'Icon', value: 'icon' },
    { text: 'Problem', value: 'problem' },
    { text: 'Words', value: 'words' },
    { text: '', value: 'data-table-expand' }
  ]
}
</script>

<style lang="scss">
@import '@/assets/scss/theme.scss';
.expand-label {
  color: #2196F3;
}
</style>
