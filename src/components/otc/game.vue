<template>
    <div>
        <div class="game-container">
            <!-- 第四行 -->
			<span class="display-text" style="margin-left: 30px;">
				Bid: <el-input v-model="bid" class="bet-amount-input"></el-input>
			</span>
			<span class="display-text" style="margin-left: 30px;">
				Ask: <el-input v-model="ask" class="bet-amount-input"></el-input>
				Target Token Contract: <el-input v-model="target_token_contract" class="bet-amount-input"></el-input>
			</span>								

            <el-row class="account-info">
                <el-col :span="8" class="account-info-section">
                    <div class="account-container">
                        <img class="navbar-coin" src="../../assets/eos-logo.png">
                        <span class="display-text">{{store.balance}}</span>
                    </div>
                </el-col>
                <el-col :span="8" class="account-info-section">
                  <el-button type="primary" class="login-button" @click="initIdentity()" v-if="!store.account">{{$t('LOGIN')}}</el-button>
                  <el-button type="primary" class="login-button" @click="ask_order()" v-else v-loading="loading">{{$t('ROLL DICE')}}</el-button>
                </el-col>
                <el-col :span="8" class="account-info-section">
                    <div class="account-container">
                        <img class="navbar-coin" src="../../assets/HPY_Token.png">
                        <span class="display-text">{{store.hpyBalance}}</span>
                        <i class="el-icon-question" @click="isShowBetDialog = !isShowBetDialog" style="cursor: pointer;"></i>
                    </div>
                </el-col>
            </el-row>

        </div>
        <MarketView class="market-container" />
    </div>
</template>

<script>
import * as store from "../../store.js";
import { getOrders } from "./orders";
import OrderView from "./order";
import MarketView from "./market";
export default {
  components: {
    OrderView,
    MarketView
  },
  data() {
    return {
      store: store.store,
      range: 50,
      betAmount: 1,
      isShowBetDialog: false,
      loading: false,
      choose: "small",
      ask: "1.0000 HPY",
      bid: "1.0000 EOS",
      target_token_contract: 'happyeosslot'
    };
  },
  computed: {
  },
  async created() {
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
.game-container {
  width: 655px;
  height: 386px;
  margin: 60px auto 20px auto;
  font-size: 18px;
  border-radius: 5px;
  background-color: #4b4848;
  align-items: center;
}
.slider-wrapper {
  padding: 10px;
  background-color: #4b484888;
  border-radius: 30px;
  align-items: center;
  width: 655px;
  margin: 0 auto;
  display: flex;
  flex-direction: row;
  color: white;
  justify-content: space-between;
}
.slider-wrapper .range {
  width: 90%;
}
.bet-amount-container {
  background-color: #3f3e3e;
  height: 43px;
  border-radius: 0.3em;
  padding: 4px;
  display: flex;
  justify-content: center;
  align-items: center;
}
.bet-amount {
  width: 100%;
  height: 100%;
  display: flex;
}
.bet-amount .amount {
  background-color: #4b4848;
  display: flex;
  align-items: center;
  border-radius: 0.3em;
}
.bet-amount .amount-button {
  display: flex;
  align-items: center;
  justify-content: center;
  height: 100%;
  cursor: pointer;
}
.navbar-coin {
  height: 22px;
  margin-left: 10px;
  vertical-align: middle;
}
.roll-info {
  background-color: #3f3e3e;
  border-radius: 0.3em;
  height: 87px;
  margin: 0 20px;
}
.roll-info-section {
  text-align: center;
  padding: 10px;
}
.choose-info {
  margin-bottom: 20px;
}
.market-container {
  max-width: 90%;
  margin: 0 auto;
}
.choose-info-section {
  background-color: #3f3e3e;
  height: 70px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 0.3em;
  padding: 4px;
}
.account-info {
  height: 87px;
  margin: 0 20px;
  font-size: 20px;
  display: flex;
  align-items: center;
}
.account-container {
  display: flex;
  align-items: center;
}
.account-container span {
  display: block;
  width: 100%;
  text-align: center;
}
.bet-amount-input input {
  background-color: #4b4848;
  border: 0;
  color: white;
  font-size: 1.2em;
}
</style>