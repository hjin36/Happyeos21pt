<template>
    <div>
      <el-card class="box-card">
        <h1 class="title"> EOS 市场 </h1>
        <el-row class="eos-orders-list" :gutter="20">
                <el-col :span="8" class="order-info-section" v-for="order in eosOrdersList" :key="order.id">
                  <OrderView :order="order" />
                </el-col>
        </el-row>
      </el-card>
      <el-card class="box-card">
        <h1 class="title"> HPY 市场 </h1>
        <el-row class="hpy-orders-list" :gutter="20">
                <el-col :span="8" class="order-info-section" v-for="order in hpyOrdersList" :key="order.id">
                  <OrderView :order="order" />
                </el-col>
            </el-row>
      </el-card>
      <el-card class="box-card">
        <h1 class="title"> BET 市场 </h1>
        <el-row class="hpy-orders-list" :gutter="20">
                <el-col :span="8" class="order-info-section" v-for="order in betOrdersList" :key="order.id">
                  <OrderView :order="order" />
                </el-col>
            </el-row>
      </el-card>      
    </div>
</template>

<script>
import * as store from "../../store.js";
import { getOrders } from "./orders";
import OrderView from "./order";
export default {
  components: {
    OrderView
  },
  data() {
    return {
      store: store.store,
      eosOrdersList: [],
      hpyOrdersList: [],
      betOrdersList: [],
    };
  },
  computed: {
  },
  async created() {
    this.eosOrdersList = await getOrders();
    this.hpyOrdersList = await getOrders("happyeosslot");
    this.betOrdersList = await getOrders("betdividends");
  },
  watch: {
    range(newRange, oldRange) {
      if (newRange < 6) {
        this.range = 6;
      } else if (newRange > 93) {
        this.range = 93;
      }
    }
  },
  methods: {
    initIdentity() {
      store.initIdentity();
    },

    async ask_order() {
      const {target_token_contract, ask, bid} = this
      const memo = `ask,${ask},${target_token_contract}`

      try {
        await this.store.eos.transfer(
          this.store.account.name,
          "eosotcbackup",
          `${bid}`,
          `${memo}`
        );
        this.$notify.success({
          title: '挂单成功',
          message: "请耐心等待"
        });
      } catch (error) {
        this.$notify.error({
          title: '交易失败',
          message: error.message
        });
      }
    },

    roll: function() {
      this.loading = true;
      let memo = `bet ${
        this.choose === "small" ? this.range + 100 : this.range
      } ${this.store.seed}`;
      const referral = this.store.referral;
      if (referral) {
        memo += ` ${referral}`;
      }
      this.store.eos
        .transfer(
          this.store.account.name,
          "happyeosdice",
          `${this.betAmount.toFixed(4)} EOS`,
          memo
        )
        .then(() => {
          // 轮询查找结果
          const r = setInterval(() => {
            this.store.eos
              .getTableRows(
                true,
                "happyeosdice",
                this.store.account.name,
                "result",
                "0"
              )
              .then(data => {
                const ans = data.rows[0].roll_number;
                // roll点值为0-99
                if (ans < 100) {
                  clearInterval(r);
                  this.loading = false;
                  if (
                    (this.choose === "small" && ans < this.range) ||
                    (this.choose === "big" && ans > this.range)
                  ) {
                    this.roll_success(ans);
                  } else {
                    this.roll_fail(ans);
                  }
                }
              });
          }, 1000);
        })
        .catch(err => {
          console.error(err);
          alert("项目出错了，快联系开发者！");
        });
    }
  }
};
</script>

<style>

</style>
