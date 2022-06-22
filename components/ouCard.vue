<template>
    <div class="ou-wrap">
      <img src="logo-ou.svg" alt="">
          <h1>新光OU點點卡回饋計算</h1>
      <div class="warning">
        <div class="warning-title">注意事項</div>
        <div>適用於有申辦OU帳戶並已申請自動扣款</div>
        <div>有任何權益變更以<a class="link" href="https://www.skbank.com.tw/campaign/skbankOU/index.html" target=”_blank”>官網</a>為主</div>
      </div>
        <div class="input-content">
            <div class="input-wrap">
                <el-input
                    v-model="datas.amount"
                    placeholder="請輸入消費金額"
                    type="tel"
                />
            </div>
            <div class="input-wrap">
                <div>是否為新戶且消費日為核卡三十天內</div>
                <el-radio-group v-model="datas.newClient">
                    <el-radio
                        v-for="item in clientOptions"
                        :key="`client-${item.value}`"
                        :label="item.value"
                    >
                        {{ item.text }}
                    </el-radio>
                </el-radio-group>
            </div>
            <div class="input-wrap">
                <el-select v-model="datas.channel" placeholder="請選擇消費通路">
                    <el-option
                        v-for="item in allChannels"
                        :key="`channel-${item}`"
                        :label="item"
                        :value="item"
                    />
                </el-select>
            </div>
            <div class="input-wrap">
                <el-select v-model="datas.payDay" placeholder="請選擇消費日期">
                    <el-option
                        v-for="item in dayOfWeek"
                        :key="`day-${item.value}`"
                        :label="item.text"
                        :value="item.value"
                    />
                </el-select>
            </div>
            <div class="input-wrap">
                <el-select v-model="datas.payment" placeholder="請選擇支付方式">
                    <el-option
                        v-for="item in paymentList"
                        :key="`payment-${item}`"
                        :label="item"
                        :value="item"
                    />
                </el-select>
            </div>
        </div>
        <div class="re-caculate">
            <el-button type="primary" plain @click="dataReste">我要重算</el-button>
        </div>
        <div class="result-msg">
            <h3>您可獲得的回饋試算</h3>
            <el-table
              :data="tableResult"
              border
              class="tableStyle">
                <el-table-column
                  prop="project"
                  label="回饋項目"
                  width="180">
                </el-table-column>
                <el-table-column
                  prop="offer"
                  label="回饋金額"
                  width="100">
                </el-table-column>
                <el-table-column
                  prop="left"
                  label="剩餘優惠"
                  width="100">
                </el-table-column>
                <el-table-column
                  label="回饋通路"
                  width="100">
                  <template slot-scope="scope">
                    <el-popover
                        ref="offerList"
                        placement="right"
                        title="回饋通路"
                        :width="popSize"
                        trigger="hover"
                        :content="scope.row.shops"
                      />
                      <i v-if="scope.row.shops.length" v-popover:offerList class="el-icon-info icon-style" />
                  </template>
                </el-table-column>
            </el-table>
            <!-- <div v-show="datas.newClient">
                新戶首月加碼 5%: {{ newClient }} (剩餘：{{ leftOver('newClient') }})
                <el-popover
                    ref="newClientList"
                    placement="right"
                    title="回饋通路"
                    :width="popSize"
                    trigger="hover"
                    :content="newClientChannelsList"
                />
                <i v-popover:newClientList class="el-icon-info icon-style" />
            </div>
            <div>
                指定通路加碼 4.7%: {{ specialChannel }} (剩餘：{{ leftOver('specialChannel') }}) <el-popover
                    ref="specialList"
                    placement="right"
                    title="回饋通路"
                    :width="popSize"
                    trigger="hover"
                    :content="specialChannelsList"
                />
                <i v-popover:specialList class="el-icon-info icon-style" />
            </div>
            <div>基本回饋 0.3%: {{ noLimit }}</div>
            <div>
                OU加碼 {{ isOuDay ? '17%' : '12%' }}: {{ ouOffer }} (剩餘：{{ leftOver('ouOffer') }})  <el-popover
                    ref="ouOfferList"
                    placement="right"
                    title="回饋通路"
                    :width="popSize"
                    trigger="hover"
                    :content="ouOfferList"
                />
                <i v-popover:ouOfferList class="el-icon-info icon-style" />
            </div>
			      <div>此次可獲得優惠: {{ totalOffer }}</div> -->
        </div>
        <div class="instructions">
          <div class="title">參考資料</div>
          <a class="link" href="https://www.skbank.com.tw/campaign/skbankOU/notice.html" target=”_blank”>新光OU點點卡回饋細則</a>
        </div>
        <div class="instructions">
          <div class="title">特別說明</div>
          <div>「OU數位帳戶加碼回饋」以<span class="red-text">月</span>為計算週期，消費之<span class="red-text">次次月底前</span>以現金存入至<span class="red-text">OU數位存款帳戶</span></div>
        </div>
        <div class="instructions">
          <div class="title">作者</div>
          <div>JY</div>
        </div>
    </div>
</template>

<script>

export default {
  data() {
    return {
      /* 新戶加碼 5%：邏輯使用 */
      newClientChannels: ['富邦momo', '蝦皮', 'PChome24h購物', 'Yahoo奇摩', '淘寶', 'foodpanda', 'Uber Eats'],
      /* 4.7%指定加碼通路 */
      specifyChannels: ['富邦momo', '蝦皮', 'PChome24h購物', 'PChome商店街', 'Yahoo奇摩', '淘寶', '東森/森森購物', 'viva', '博客來', '生活市集', 'friDay購物', '夠麻吉GOMAJI', 'ASOS', 'Shopbop', 'IHERB', 'Amazon', '17Life', 'TENSO', 'App Store', 'Google Play', 'foodpanda', 'Uber Eats'],
      ouChannels: ['富邦momo', 'PChome24h購物', '蝦皮', 'Yahoo奇摩', '淘寶', 'foodpanda', 'Uber Eats', 'My FamiPay', 'HiPay', 'LINE Pay', 'PXPAY', 'OPEN錢包'],
      /* 符合的話 加碼 5% */
      ouDay: {
        monday: ['My FamiPay'],
        tuesday: ['OPEN錢包'],
        wednesday: ['HiPay', 'My FamiPay'],
        thursday: ['LINE Pay'],
        friday: ['PXPAY'],
        saturday: [],
        sunday: ['OPEN錢包']
      },
      /* 所有通路選擇 */
      allChannels: ['富邦momo', '蝦皮', 'PChome24h購物', 'PChome商店街', 'Yahoo奇摩', '淘寶', '東森/森森購物', 'viva', '博客來', '生活市集', 'friDay購物', '夠麻吉GOMAJI', 'ASOS', 'Shopbop', 'IHERB', 'Amazon', '17Life', 'TENSO', 'App Store', 'Google Play', 'foodpanda', 'Uber Eats', '其他'],
      paymentList: ['網購刷卡', 'My FamiPay', 'HiPay', 'LINE Pay', 'PXPAY', 'OPEN錢包'],
      dayOfWeek: [
        { value: 'monday', text: '週一' },
        { value: 'tuesday', text: '週二' },
        { value: 'wednesday', text: '週三' },
        { value: 'thursday', text: '週四' },
        { value: 'friday', text: '週五' },
        { value: 'saturday', text: '週六' },
        { value: 'sunday', text: '週日' }
      ],
      clientOptions: [
        { value: true, text: '是' },
        { value: false, text: '否' }
      ],
      /* 綁定資料 */
      datas: {
        channel: null,
        newClient: true,
        payment: '',
        payDay: '',
        amount: ''
      },
      /* 優惠整理 */
      offer: {
        /* 信用卡新戶 */
        newClient: {
          percent: 0.05,
          limit: 250
        },
        /* 保底 */
        noLimit: {
          percent: 0.003,
          limit: null
        },
        /* 指定通路加碼 */
        specialChannel: {
          percent: 0.047,
          limit: 250
        },
        /* OU數位帳戶加碼回饋 */
        ouOffer: {
          percent: 0.11,
          limit: 250
        },
        /* OU數位帳戶加碼回饋(ouDay) */
        ouday: {
          percent: 0.16,
          limit: 250
        },
        /* OU數位帳戶加碼回饋(綁定自動扣繳) */
        ouAccount: {
          percent: 0.01,
          limit: 100
        }
      },
      windowWidth: window.innerWidth
    };
  },
  computed: {
    newClient() {
      if (this.newClientChannels.includes(this.datas.channel) && this.datas.newClient) {
        const { percent, limit } = this.offer.newClient;
        const amount = +this.datas.amount * percent;
        return Math.round(amount >= limit ? limit : amount);
      }
      return 0;
    },
    noLimit() {
      return Math.round(this.datas.amount * this.offer.noLimit.percent);
    },
    specialChannel() {
      if (this.specifyChannels.includes(this.datas.channel)) {
        const { percent, limit } = this.offer.specialChannel;
        const amount = +this.datas.amount * percent;
        return Math.round(amount >= limit ? limit : amount);
      }
      return 0;
    },
    ouOffer() {
      if (this.ouChannels.includes(this.datas.channel)
        || (this.datas.payment && this.datas.payment !== '網購刷卡')) {
        const { percent, limit } = this.isOuDay ? this.offer.ouday : this.offer.ouOffer;
        const amount = +this.datas.amount * percent;
        return Math.round(amount >= limit ? limit : amount);
      }
      return 0;
    },
    ouAccount() {
      if (this.ouChannels.includes(this.datas.channel)
      || (this.datas.payment && this.datas.payment !== '網購刷卡')) {
        const { percent, limit } = this.offer.ouAccount;
        const amount = +this.datas.amount * percent;
        return Math.round(amount >= limit ? limit : amount);
      }
      return 0
    },
    isOuDay() {
      if (!this.datas.payment || !this.datas.payDay || this.datas.payDay === 'saturday') {
        return false
      }
      return this.datas.payment && this.datas.payDay ? this.ouDay[this.datas.payDay].includes(this.datas.payment) : false;
    },
    newClientChannelsList() {
      return this.newClientChannels.join('、');
    },
    specialChannelsList() {
      return this.specifyChannels.join('、');
    },
    ouOfferList() {
      return this.ouChannels.join('、');
    },
    totalOffer() {
      return this.newClient + this.noLimit + this.specialChannel + this.ouOffer + this.ouAccount
    },
    popSize() {
      if (this.windowWidth < 900) {
        return '300'
      }
      if (this.windowWidth < 400) {
        return '250'
      }
      return '500'
    },
    tableData() {
      return [{
        id: 'newClient',
        project: '新戶首月加碼 5%',
        offer: this.newClient,
        left: this.leftOver('newClient'),
        shops: this.newClientChannelsList
      }, {
        id: 'specialChannel',
        project: '指定通路加碼 4.7%',
        offer: this.specialChannel,
        left: this.leftOver('specialChannel'),
        shops: this.specialChannelsList
      }, {
        id: 'basic',
        project: '基本回饋 0.3%',
        offer: this.noLimit,
        left: '回饋無上限',
        shops: ''
      }, {
        id: 'ouOffer',
        project: `OU加碼 ${this.isOuDay ? '17%' : '12%'}`,
        offer: this.ouOffer,
        left: this.leftOver('ouOffer'),
        shops: this.ouOfferList
      },
      {
        id: 'ouAccount',
        project: 'OU自動扣繳 1%',
        offer: this.ouAccount,
        left: this.leftOver('ouAccount'),
        shops: this.ouOfferList
      },
      {
        id: 'totalOffer',
        project: '回饋總金額',
        offer: this.totalOffer,
        left: '',
        shops: ''
      }]
    },
    tableResult() {
      if (!this.datas.newClient) {
        return this.tableData.filter(item => item.id !== 'newClient')
      }
      return this.tableData
    }
  },
  methods: {
    leftOver(type) {
      return this.offer[type].limit - +this[type];
    },
    dataReste() {
      this.datas = {
        channel: null,
        newClient: true,
        payment: '',
        payDay: '',
        amount: ''
      }
    }
  }

};
</script>

<style lang="scss" scoped>

.ou-wrap {
	padding: 30px 0;
	width: 80%;
	margin: 0 auto;
	text-align: center;

  h1 {
    margin-top: 0;
  }
}
.input-wrap {
	margin: 10px 0;
}

.el-input {
	width: 203px;
}

.icon-style {
	margin-left: 10px;
	font-size: 18px;
	color: coral;
}

.result-msg {
	margin-top: 20px;
}

.input-content {
	margin-top: 20px;
}

.warning {
  max-width: 300px;
  margin: 0 auto;
  border: 1px solid #F00;
  padding: 10px 5px;
  border-radius: 5px;
}

.warning-title {
  margin-bottom: 5px;
  color: #F00;
}

.re-caculate {
  margin-top: 20px;
}

.instructions {
  max-width: 500px;
  margin: 0 auto;
  margin-top: 30px;
  padding: 10px 5px;

  .title {
    font-size: 18px;
    font-weight: 500;
    margin-bottom: 10px;

    &::after {
      margin: 5px auto 0;
      display: block;
      content: '';
      width: 100px;
      height: 1px;
      background: #bdbdbd;
    }
  }
}

.link {
  color: #0066FF;
  text-decoration: none;
}

.red-text {
  font-weight: 500;
  color: #FF6666;
}

.tableStyle {
  width: 480px;
  margin: 0 auto;
}

@media (max-width: 450px) {
  h1 {
    font-size: 27px;
  }
}
</style>
