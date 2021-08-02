<template>
    <div id="main" class="container-fluid">
        <div class="row d-flex justify-content-center">
            <!-- 參數 -->
            <div class="skin-form">
                <div>
                    <span>範圍下限</span>
                    <input type="number" v-model="parameters.min" />
                </div>
                <div>
                    <span>範圍上限</span>
                    <input type="number" v-model="parameters.max" />
                </div>
                <div>
                    <span>資金</span>
                    <input type="number" v-model="parameters.funds" />
                </div>
                <div>
                    <span>網格數</span>
                    <input type="number" v-model="parameters.grids" />
                </div>
                <div>
                    <span>手續費</span>
                    <input type="number" v-model="parameters.fee" />
                </div>
                <div>
                    <button class="btn btn-green btn-block" @click="calcAll()">計算</button>
                </div>
            </div>
        </div>
        <div class="row d-flex justify-content-center mt-1">
            <span class="font-weight-bold">(此表僅供參考用)</span>
        </div>
        <div class="row d-flex justify-content-center mt-1">
            <div class="extra-board">
                <div>
                    <span>網格等差</span>
                    <span>{{ extraParamters.gridDifference }}</span>
                </div>
                <div>
                    <span>範圍平均</span>
                    <span>{{ extraParamters.rangeAvg }}</span>
                </div>
                <div>
                    <span>每格買入數量</span>
                    <span>{{ extraParamters.buyCount }}</span>
                </div>
            </div>
        </div>
        <div class="row mt-1">
            <table>
                <thead>
                    <tr>
                        <td>買入價格</td>
                        <td>買入金額</td>
                        <td>買入手續費</td>
                        <td>賣出價格</td>
                        <td>賣出金額</td>
                        <td>賣出手續費</td>
                        <td>總收益</td>
                        <td>淨收益</td>
                    </tr>
                </thead>
                <tbody>
                    <tr v-for="(data, index) in dataList" :key="index">
                        <td class="bg-green">{{ data.buyprice }}</td>
                        <td class="bg-green">{{ data.buycost }}</td>
                        <td class="bg-green">{{ data.buyfee }}</td>
                        <td class="bg-red">{{ data.sellprice }}</td>
                        <td class="bg-red">{{ data.sellcost }}</td>
                        <td class="bg-red">{{ data.sellfee }}</td>
                        <td class="bg-orange">{{ data.totalincome }}</td>
                        <td class="bg-orange">{{ data.netincome }}</td>
                    </tr>
                </tbody>
            </table>
        </div>
    </div>
</template>

<script>
export default {
    name: 'main',
    data() {
        return {
            parameters: {
                min: 0,
                max: 0,
                funds: 0,
                grids: 0,
                fee: 0
            },
            extraParamters: {
                //網格等差
                gridDifference: 0,
                //範圍平均
                rangeAvg: 0,
                //買入數量
                buyCount: 0
            },
            dataList: []
        };
    },
    methods: {
        calcAll() {
            this.clearList();
            //轉換成數字
            this.parameters.min = parseFloat(this.parameters.min);
            this.parameters.max = parseFloat(this.parameters.max);
            this.parameters.funds = parseFloat(this.parameters.funds);
            this.parameters.grids = parseFloat(this.parameters.grids);
            this.parameters.fee = parseFloat(this.parameters.fee);
            //網格等差
            this.extraParamters.gridDifference = this.calcGridDiff();
            //範圍平均
            this.extraParamters.rangeAvg = this.calcRangeAvg();
            //買入數量
            this.extraParamters.buyCount = this.calcBuyCount();
            //表格計算
            let totalPrice = parseFloat(this.parameters.min);
            for (let i = 0; i <= this.parameters.grids; i++) {
                //買入價格(Coin)
                let buyPrice = totalPrice;
                //買入金額(資金)
                let buyCost = buyPrice * this.extraParamters.buyCount;
                //買入手續費
                let buyFee = buyCost * parseFloat(this.parameters.fee);

                //賣出價格(Coin)
                let sellPrice = buyPrice + this.extraParamters.gridDifference;
                //賣出金額(資金)
                let sellCost = sellPrice * this.extraParamters.buyCount;
                //買入手續費
                let sellFee = sellCost * parseFloat(this.parameters.fee);

                //總收益
                let totalIncome = sellCost - buyCost;
                //淨收益
                let netIncome = totalIncome - (buyFee + sellFee);

                this.dataList.push({
                    buyprice: buyPrice.toFixed(5),
                    buycost: buyCost.toFixed(5),
                    buyfee: buyFee.toFixed(5),
                    sellprice: sellPrice.toFixed(5),
                    sellcost: sellCost.toFixed(5),
                    sellfee: sellFee.toFixed(5),
                    totalincome: totalIncome.toFixed(5),
                    netincome: netIncome.toFixed(5)
                });
                totalPrice += this.extraParamters.gridDifference;
            }
        },
        calcGridDiff() {
            //(上限 - 下限) / 網格數
            return (this.parameters.max - this.parameters.min) / this.parameters.grids;
        },
        calcRangeAvg() {
            //(上限 + 下限) / 2
            return (this.parameters.max + this.parameters.min) / 2;
        },
        calcBuyCount() {
            //(資金 / 網格數) / 範圍平均
            return (this.parameters.funds / this.parameters.grids / this.extraParamters.rangeAvg).toFixed(7);
        },
        clearList() {
            this.dataList.splice(0, this.dataList.length);
        }
    }
};
</script>