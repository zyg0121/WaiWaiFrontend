<template>
    <div id="HomeView">
        <el-row :gutter="24">
            <!-- 滚动banner -->
            <el-col :span="20" :offset="2">
                <el-carousel :interval="3000" arrow="always">
                    <el-carousel-item v-for="item in itemList" :key="item">
                        <el-image class="bannerimage" :src=item fit="fill"></el-image>
                    </el-carousel-item>
                </el-carousel>
            </el-col>
            <!-- 最新商品 -->
            <el-col :span="20" :offset="2" style="padding-top:30px;">
                <h2>最新商品</h2>
                <el-row>
                    <el-col :span="5" v-for="(item, index) in data1" :key="item.goosid" :offset="index > 0 ? 1 : 0">
                        <el-card :body-style="{ padding: '0px' }">
                            <img :src="urlimg + item.goodsimg" class="image" width="30%">
                            <div style="padding: 14px;">
                                <span>{{ item.goodsname }}</span>
                                <div class="bottom clearfix">
                                    <span class="info-title">{{ item.goodsdesc }}</span>
                                    <el-button type="text" class="button" @click="clickGoodsInfoButton(item.goosid)">
                                        查看详情</el-button>
                                </div>
                            </div>
                        </el-card>
                    </el-col>
                </el-row>
            </el-col>

            <!-- 低价好物 -->
            <el-col :span="20" :offset="2" style="padding-top:30px;">
                <h2>低价好物</h2>
                <el-row>
                    <el-col :span="5" v-for="(item, index) in data2" :key="item.goosid" :offset="index > 0 ? 1 : 0">
                        <el-card :body-style="{ padding: '0px' }">
                            <img :src="urlimg + item.goodsimg" class="image" width="30%">
                            <div style="padding: 14px;">
                                <span>{{ item.goodsname }}</span>
                                <div class="bottom clearfix">
                                    <span class="info-title">{{ item.goodsdesc }}</span>
                                    <el-button type="text" class="button" @click="clickGoodsInfoButton(item.goosid)">
                                        查看详情</el-button>
                                </div>
                            </div>
                        </el-card>
                    </el-col>
                </el-row>
            </el-col>

            <!-- 单个商品的详情 -->
            <el-col :span="20" :offset="2" style="padding-top:30px;">
                <el-dialog :title="goodsInfoName" :visible.sync="centerDialogVisible" width="418px" center>
                    <div style="text-align: center">
                        <!--<img src="https://shadow.elemecdn.com/app/element/hamburger.9cf7b091-55e9-11e9-a976-7f4d0b07eef6.png"
                            class="image" width="365px" height="300px">-->
                        <el-carousel direction="horizontal" :autoplay="true">
                            <el-carousel-item v-for="item in goodsInfoImg" :key="item.imgid"
                                style="width: 365px;height: 300px;">
                                <img :src="urlimg + item.imgurl" alt="请检查网络连接" width="93%">
                            </el-carousel-item>
                        </el-carousel>
                        <h2><img src="@/assets/yuan.png" alt="￥" width="30px">:
                            {{ parseInt(goodsInfoPrice) / 100 }}</h2>
                        <p>{{ goodsInfoDscrip }}</p>
                        <span slot="footer" class="dialog-footer">
                            <el-button type="primary" style="margin-top: 30px;" @click="addWant()">加入我想要</el-button>
                        </span>
                    </div>
                </el-dialog>
            </el-col>
        </el-row>
    </div>
</template>

<script>
// @ is an alias to /src
import request from '@/utils/request';

export default {
    name: 'HomeView',
    data() {
        return {
            urlimg: 'http://localhost:9002/upload/',
            itemList: {
                img1: require("@/assets/school1.jpg"),
                img2: require("@/assets/school2.jpg"),
                img3: require("@/assets/school3.jpg"),
                img4: require("@/assets/school4.jpg")
            },
            data1: [], //最新发布
            data2: [], //低价好物
            dataSearch: [], //用来保存搜索的全部结果
            goodsInfoId: "",
            goodsInfoImg: {},
            goodsInfoName: "",
            goodsInfoDscrip: "",
            goodsInfoPrice: 0,
            centerDialogVisible: false,
            dialogValue: "",
            centerDialogVisible2: false,
            goToShopCar: true,
            currentPage: 1, //初始页
            pageSize: 12, //每页的数据
            paginationShow: false, //默认不显示分页
            searchFlag: false, //用来避免频繁向服务器发送数据
            fullscreenLoading: false, //模拟加载
        }
    },
    components: {

    },
    mounted() {
        request.get("/api/good/select/new?num=4")
            .then(res => {
                console.log(res);
                this.data1 = res.data;
            })
        request.get("/api/good/select/price?num=4")
            .then(res => {
                console.log(res);
                this.data2 = res.data;
            })
    },
    methods: {
        clickGoodsInfoButton(goodsId) {
            this.centerDialogVisible = true;
            this.goodsInfoId = goodsId;
            let self = this;
            console.log(goodsId)
            request.get("/api/good/select/goodsid?goodsid=" + goodsId).then(res => {
                console.log(res);
                var data = res.data
                console.log(data)
                self.goodsInfoName = data.goodsname;
                self.goodsInfoPrice = data.goodsprice;
                self.goodsInfoDscrip = data.goodsdesc;
            })

            request.get("/api/img/all?id=" + goodsId).then(res => {
                console.log(res);
                self.goodsInfoImg = res.data
                console.log(self.goodsInfoImg);
            })


        },
        addWant() {
            var goodsId = this.goodsInfoId;
            var userId = window.sessionStorage.getItem("user");
            request.get("/api/want/insert?goodsid=" + goodsId +"&userid="+userId).then(res => {
                console.log(res);
                if (res.state == '0') {
                    this.$message({
                        type: "success",
                        message: "加入我想要成功"
                    })
                    this.$router.push("/wantview");
                }
                else {
                    this.$message({
                        type: "error",
                        message: res.msg
                    })
                }
            })
        },
    },
}
</script>

<style scoped>
.bottom {
    margin-top: 13px;
    line-height: 12px;
}

.button {
    padding: 0;
    float: right;
}

.image {
    width: 100%;
    display: block;
}

.clearfix:before,
.clearfix:after {
    display: table;
    content: "";
}

.clearfix:after {
    clear: both
}

.info-title {
    font-size: 13px;
    color: #999;
    width: 100%;
    /*一定要设置宽度，或者元素内含的百分比*/
    overflow: hidden;
    /*溢出的部分隐藏*/
    white-space: nowrap;
    /*文本不换行*/
    text-overflow: ellipsis;
    /*ellipsis:文本溢出显示省略号（...）；clip：不显示省略标记（...），而是简单的裁切*/
}
</style>