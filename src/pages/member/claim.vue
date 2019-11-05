<template>
  <q-page class="q-pa-md">
    <!-- <q-input stack-label="Account name" v-model="accountToCheck" dark /> -->

    <div
      class="round-borders bg-bg1 bg-logo shadow-4 overflow-hidden"
      style="max-width:400px"
    >
      <div
        class="bg-primary q-pa-sm row justify-between items-center"
        style="height:50px"
      >
        <q-icon :name="$configFile.icon.dactoken" color="text2" size="24px" />
        <span>My Claimables ({{ my_claimables.length }})</span>
        <help-btn
          content="Each week the candidates are invited to claim their rewards."
          title="My claimables"
          color="text1"
          size="sm"
        />
      </div>

      <q-scroll-area
        style="width: 100%; height: 250px;"
        :thumb-style="{
          right: '0px',
          borderRadius: '2px',
          background: '#485aa3',
          width: '10px',
          opacity: 0.8
        }"
      >
        <q-list no-border separator highlight dark>
          <div
            v-if="isLoading"
            class="row justify-center text-weight-thin q-mt-md"
          >
            <div class="column items-center">
              <q-spinner />
              <div>checking for claimables</div>
            </div>
          </div>

          <div
            class="row justify-center text-weight-thin q-mt-md"
            v-else-if="!my_claimables.length"
          >
            You don't have any claimables
          </div>
          <q-item
            v-for="(claim, i) in my_claimables"
            :key="`claim${i}`"
            class="animate-fade"
          >
            <q-item-side
              left
              icon="icon-ui-19"
              :title="`id: ${claim.distri_id}`"
            />
            <q-item-main>
              <q-item-tile label>
                <div class="overflow-hidden">
                  <span>{{ claim.receiver }}</span>
                </div>
              </q-item-tile>
              <q-item-tile sublabel>
                <div style="white-space: nowrap;" class=" overflow-hidden">
                  {{ claim.amount }}
                </div>
              </q-item-tile>
            </q-item-main>
            <q-item-side right style="min-width:65px">
              <q-btn
                color="primary"
                class="animate-fade"
                size="sm"
                label="claim"
                @click="claimpay(claim.distri_id, claim.receiver)"
              />
            </q-item-side>
          </q-item>
        </q-list>
      </q-scroll-area>
      <q-item v-if="my_claimables.length > 1">
        <q-item-main>
          <q-item-tile label>Total</q-item-tile>
          <q-item-tile sublabel>{{ getTotal }}</q-item-tile>
        </q-item-main>
        <q-item-side right>
          <q-btn
            color="positive"
            :disabled="false"
            label="claim all"
            @click="claimAll"
          />
        </q-item-side>
      </q-item>
    </div>
  </q-page>
</template>

<script>
import { mapGetters } from "vuex";
import helpBtn from "components/controls/help-btn";
export default {
  name: "claim",
  components: {
    helpBtn
  },
  data() {
    return {
      isLoading: false,
      my_claimables: [],
      accountToCheck: ""
    };
  },
  computed: {
    ...mapGetters({
      getAccountName: "user/getAccountName",
      getIsCustodian: "user/getIsCustodian",
      getIsCandidate: "user/getIsCandidate",
      getDacApi: "global/getDacApi"
    }),
    getTotal() {
      let total = 0;
      this.my_claimables.forEach(mc => {
        let a = mc.amount.split(" ")[0];
        total += Number(a);
      });
      return `${total} VIG`;
    }
  },
  methods: {
    async getDistris() {
      let res = await this.getDacApi.eos
        .get_table_rows({
          json: true,
          code: this.$configFile.get("districontract"),
          scope: this.$configFile.get("districontract"),
          table: "districonfs",
          limit: -1
        })
        .catch(e => false);

      if (!res || !res.rows.length) {
        this.isLoading = false;
        return false;
      } else {
        return res.rows;
      }
    },
    async getClaimables() {
      let accountname = this.getAccountName;
      this.isLoading = true;
      let distris = await this.getDistris();
      if (!distris) {
        this.isLoading = false;
        return;
      }
      let claims = [];
      for (let i = 0; i < distris.length; i++) {
        let res = await this.getDacApi.eos
          .get_table_rows({
            json: true,
            code: this.$configFile.get("districontract"),
            lower_bound: accountname,
            upper_bound: accountname,
            scope: distris[i].distri_id,
            table: "distris",
            limit: -1
          })
          .catch(e => false);
        if (res && res.rows.length) {
          res.rows[0].distri_id = distris[i].distri_id;
          res.rows[0].status = 0;
          claims.push(res.rows[0]);
        }
      }
      this.isLoading = false;
      this.my_claimables = claims;
      console.log("open claimables for user", accountname, claims);
    },
    async claimpay(id) {
      let actions = [
        {
          account: this.$configFile.get("districontract"),
          name: "claim",
          data: {
            distri_id: id,
            receiver: this.getAccountName
          }
        }
      ];
      let result = await this.$store.dispatch("user/transact", {
        actions: actions
      });
      if (result) {
        this.my_claimables = this.my_claimables.filter(
          mc => mc.distri_id != id
        );
      }
    },
    async claimAll() {
      let actions = this.my_claimables.map(mc => {
        return {
          account: this.$configFile.get("districontract"),
          name: "claim",
          data: {
            distri_id: mc.distri_id,
            receiver: this.getAccountName
          }
        };
      });
      let result = await this.$store.dispatch("user/transact", {
        actions: actions
      });
      if (result) {
        this.my_claimables = [];
      }
    }
  },
  mounted() {
    this.getClaimables();
  }
};
</script>

<style></style>
