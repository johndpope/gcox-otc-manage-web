<!-- 广告管理 -->
<template>
    <Card>
        <p slot="title">{{$t('nav.gggl')}}
            <span class="refresh" @click="getAdsList"></span>
        </p>
        <Row>
            <Select v-model="symbol" style="width:200px" @on-change="curPage=1;getAdsList()">
                <Option value="0">{{$t('common.qb')}}</Option>
                <Option v-for="(item,i) in symbolTypeList" :value="item.symbol" :key="i">{{item.symbol}}</Option>
                <!-- <Option value="BTC">BTC</Option>
                <Option value="ETH">ETH</Option>
                <Option value="ATN">ATN</Option>
                <Option value="MECoin">MECoin</Option>
                <Option value="USDT">USDT</Option> -->
            </Select>
        </Row>
        <Tabs style="margin-top:10px;" @on-click="changeTab">
            <TabPane :label="$t('otc.mbgg')" name="2">
                <Table :columns="columns1" :data="data1" @on-sort-change="setAdsSaleSort"></Table>
                <Page :current="curPage" :total="total" @on-change="changePage"
                      style="text-align:center;margin-top:20px;"></Page>
            </TabPane>
            <TabPane :label="$t('otc.maibgg')" name="1">
                <Table :columns="columns1" :data="data1" @on-sort-change="setAdsBuySort"></Table>
                <Page :current="curPage" :total="total" @on-change="changePage1"
                      style="text-align:center;margin-top:20px;"></Page>
            </TabPane>
        </Tabs>
    </Card>
</template>

<script>
    import util from '../../libs/util';
    import vieworder from './ads/vieworder';
    import otcApi from '../../api/otc';
    import Cookies from 'js-cookie';
    import currenyApi from '@/api/currency';

    export default {
        data () {
            return {
                curPage: 1,
                total: 0,
                curPage1: 1,
                total1: 0,
                columns1: [
                    {title: this.$t('otc.fbz'), key: 'username'},
                    {
                        title: this.$t('otc.zsl'), key: 'symbolCount', sortable: 'custom',
                        render: (h, params) => {
                            return h('div', [params.row.symbolCount, '', params.row.symbol]);
                        }
                    },
                    {
                        title: this.$t('otc.ye'), key: 'remainCount', sortable: 'custom',
                        render: (h, params) => {
                            return h('div', [params.row.remainCount, params.row.symbol]);
                        }
                    },
                    {
                        title: this.$t('otc.yj'), key: 'priceRate',
                        render: (h, params) => {
                            return h('div', [params.row.priceRate, '%']);
                        }
                    },
                    {
                        title: this.$t('otc.zgdjsj'), key: 'acceptPrice', sortable: 'custom',
                        render: (h, params) => {
                            return h('div', [params.row.acceptPrice, params.row.currency]);
                        }
                    },
                    {
                        title: this.$t('otc.xe'), key: 'minAmount', sortable: 'custom',
                        render: (h, params) => {
                            return h('div', [params.row.minAmount, params.row.adType === 2 ?
                                params.row.currency : params.row.symbol, ' - ',
                                params.row.maxAmount, params.row.adType === 2 ?
                                    params.row.currency : params.row.symbol]);
                        }
                    },
                    {title: this.$t('otc.xs'), key: 'payLimitTime'},
                    {title: this.$t('otc.ddxzs'), key: 'maxProcessNum'},
                    {title: this.$t('common.cjsj'), key: 'createdAt', sortable: 'custom'},
                    {
                        title: this.$t('common.cz'), key: 'action', width: 200, render: (h, params) => {
                            return h('div', [
                                h('Button', {
                                    props: {type: 'primary', size: 'small'},
                                    style: {marginRight: '10px'},
                                    on: {
                                        click: () => {
                                            util.setDialog(vieworder, {
                                                adId: params.row.adId,
                                                username: params.row.username,
                                                symbol: params.row.symbol
                                            });
                                        }
                                    }
                                }, this.$t('otc.ckdd')),
                                h('Button', {
                                    props: {
                                        type: 'primary',
                                        size: 'small',
                                        disabled: !util.showThisRoute(['ROLE_ADMIN', 'ROLE_OPERATION', 'ROLE_OTC_AUDIT'], Cookies.get('roles')) && util.showThisRoute(['ROLE_OTC_APPEAL', 'ROLE_CUSTOMER'], Cookies.get('roles'))
                                    },
                                    on: {
                                        click: () => {
                                            this.$Modal.confirm({
                                                content: '<p style="font-size:18px;">' + this.$t('otc.gggsfxj') + '？</p>',
                                                onOk: () => {
                                                    otcApi.soldOutAd({
                                                        adId: params.row.adId,
                                                        userId: params.row.userId
                                                    }, (res) => {
                                                        if ((this.total > 10) && (this.total % 10 == 1)) {
                                                            this.curPage = this.curPage - 1;
                                                        }
                                                        this.$Message.success({content: this.$t('otc.xjcg')});
                                                        this.getAdsList();
                                                    }, (msg) => {
                                                        this.$Message.error({content: msg || this.$t('ieo.sb')});
                                                    });
                                                },
                                                onCancel: () => {
                                                    this.$emit('removeDialog');
                                                }
                                            });
                                        }
                                    }
                                }, this.$t('otc.qzxj'))
                            ]);
                        }
                    }
                ],
                data1: [],
                symbol: '0',
                ad_type: 2,
                symbolTypeList: null
            };
        },
        created () {
            this.getAdsList();
            this.getdataSymbol();
            // this.symbolTypeList = JSON.parse(window.localStorage.symbolTypes);
        },
        methods: {
            getdataSymbol () {
                currenyApi.findAllValidSymbolList((res) => {
                    this.symbolTypeList = res;
                });
            },
            setAdsSaleSort (column) {
                this.curPage = 1;
                if (column.order !== 'normal') {
                    this.AdsSaleSortKey = column.key;
                    this.AdsSaleSortVal = column.order;
                } else {
                    this.AdsSaleSortKey = null;
                }
                this.getAdsList();
            },
            setAdsBuySort (column) {
                this.curPage = 1;
                if (column.order !== 'normal') {
                    this.AdsBuySortKey = column.key;
                    this.AdsBuySortVal = column.order;
                } else {
                    this.AdsBuySortKey = null;
                }
                this.getAdsList();
            },
            changeTab (name) {
                this.curPage = 1;
                this.ad_type = name;
                console.log(name);
                this.symbol = this.symbol;
                this.getAdsList();
            },
            getAdsList () {
                this.data1 = [];
                this.total = null;
                let data = {
                    symbol: this.symbol,
                    adType: this.ad_type
                };
                if(data.symbol == '0'){
                    data.symbol = null
                }
                if (Number(this.ad_type) === 1 && this.AdsBuySortKey) {
                    data.sortKey = `${this.AdsBuySortKey} ${this.AdsBuySortVal}`;
                }
                if (Number(this.ad_type) === 2 && this.AdsSaleSortKey) {
                    data.sortKey = `${this.AdsSaleSortKey} ${this.AdsSaleSortVal}`;
                }
                otcApi.findAdInfoOtc(this.curPage, data,
                    (res, total) => {
                        this.total = total;
                        this.data1 = res;
                    }, (msg) => {
                        this.$Message.error({content: msg});
                    });
            },
            changePage (page) {
                this.curPage = page;
                this.getAdsList();
            },
            changePage1 (page) {
                this.curPage = page;
                console.log(this.curPage1);
                this.getAdsList();
            }
        }
    };
</script>

<style lang="less" scoped>
    .refresh {
        width: 49px;
        height: 24px;
        background: url(../../images/frsh.png) center/cover no-repeat;
        background-size: contain;
        cursor: pointer;
        color: #2d8cf0;
    }

    .ivu-card-head-inner, .ivu-card-head p {
        display: flex !important;
        justify-content: space-between !important;
        height: 40px !important;
        line-height: 40px !important;
    }
</style>
