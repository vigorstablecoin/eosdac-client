<template>
  <q-page class="q-pa-md">
    <!-- {{ getVotesTable }} -->

    <template>
      <div class="round-borders shadow-4">
        <q-table
          v-if="getVotesTable.length"
          ref="table"
          class="bg-bg1 "
          color="primary-light"
          :dark="getIsDark"
          title="Votes"
          :data="getData"
          :columns="getColumns()"
          row-key="name"
          :rows-per-page-options="[8, 10, 15, 20, 0]"
        >
          <q-td slot="body-cell-voter" slot-scope="props" :props="props">
            <router-link class="" :to="{ path: '/profile/' + props.value }">{{
              props.value
            }}</router-link>
          </q-td>

          <template slot="top-right" slot-scope="props">
            <q-search v-model="filter" dark placeholder="Voter" />
            <q-btn
              flat
              round
              dense
              color="primary-light"
              :icon="props.inFullscreen ? 'fullscreen_exit' : 'fullscreen'"
              @click="props.toggleFullscreen"
            />
          </template>
        </q-table>
      </div>
    </template>
  </q-page>
</template>

<script>
import { mapGetters } from "vuex";
export default {
  name: "Votes",
  data() {
    return {
      filter: ""
    };
  },
  computed: {
    ...mapGetters({
      getIsCustodian: "user/getIsCustodian",
      getVotesTable: "dac/getVotesTable",
      getIsDark: "ui/getIsDark"
    }),
    getData() {
      if (!this.getVotesTable.length) return [];
      let res = this.getVotesTable;
      if (this.filter && this.filter.length >= 3) {
        res = res.filter(r => r.voter.includes(this.filter));
      }
      return res;
    }
  },
  methods: {
    getColumns() {
      return [
        {
          name: "voter",
          required: true,
          label: "Voter",
          align: "left",
          field: row => row.voter,
          //format: val => `${new Date(val+'.000Z')}`,
          sortable: true,
          style: "width: 150px"
        },
        {
          name: "numbervotes",
          label: "#",
          field: row => row.candidates.length,
          // format: val => `${val.join(", ")}`,
          align: "left",
          sortable: true,
          style: "width: 80px"
        },
        {
          name: "candidates",
          label: "Voted For",
          field: "candidates",
          format: val => `${val.join(", ")}`,
          align: "left"
          // searchable: true
        }
      ];
    }
  },
  mounted() {}
};
</script>

<style>
.q-table td,
.q-table th {
  /* don't shorten cell contents */
  white-space: normal !important;
}
</style>
