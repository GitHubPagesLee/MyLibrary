<template>
    <div class="company_search_wrap">
        <div class="container">
        <!-- 职位 -->
            <!-- 职位搜索标签 -->
            <div>
                <p style="padding: .2rem 0 .05rem .2rem; font-size: .16rem; font-weight: 700;" v-if="tabCheck == 0">最新职位</p>
                <p style="padding: .2rem 0 .05rem .2rem; font-size: .16rem; font-weight: 700;" v-if="tabCheck == 1">推荐职位</p>
                <!-- 职位搜索栏 -->
                <ul class="search_wrap">
                    <li class="keyword">
                        <p>关&ensp;键&ensp;词：</p>
                        <input type="text" v-model="postKeywordModel" maxlength="38">
                        <p>当前关键词：{{this.$route.query.postKeyword}}</p>
                    </li>
                    <li>
                        <p>所在地区：</p>
                        <div v-for="(city, index) in filter.city" :key="index">
                            <input type="radio" name="city" :id="city.id" :value="city.value" v-model="cityModel">
                            <label :for="city.id">{{city.name}}</label>
                        </div>
                    </li>
                    <li>
                        <p>所属行业：</p>
                        <div v-for="(industry, index) in filter.industry" :key="index">
                            <input type="radio" name="industry" :id="industry.id" :value="industry.value" v-model="industryModel">
                            <label :for="industry.id">{{industry.name}}</label>
                        </div>
                    </li>
                    <li>
                        <p>学历要求：</p>
                        <div v-for="(education, index) in filter.education" :key="index">
                            <input type="radio" name="education" :id="education.id" :value="education.value" v-model="educationModel">
                            <label :for="education.id">{{education.name}}</label>
                        </div>
                    </li>
                    <li>
                        <p>工作经验：</p>
                        <div v-for="(experience, index) in filter.experience" :key="index">
                            <input type="radio" name="experience" :id="experience.id" :value="experience.value" v-model="experienceModel">
                            <label :for="experience.id">{{experience.name}}</label>
                        </div>
                    </li>
                    <li>
                        <p>薪资水平：</p>
                        <div v-for="(compensation, index) in filter.compensation" :key="index">
                            <input type="radio" name="compensation" :id="compensation.id" :value="compensation.value" v-model="compensationModel">
                            <label :for="compensation.id">{{compensation.name}}</label>
                        </div>
                    </li>
                    <li>
                        <p>发布时间：</p>
                        <div v-for="(updateAt, index) in filter.updateAt" :key="index">
                            <input type="radio" name="updateAt" :id="updateAt.id" :value="updateAt.value" v-model="updateAtModel">
                            <label :for="updateAt.id">{{updateAt.name}}</label>
                        </div>
                    </li>
                    <li class="buttons">
                        <button class="clear" @click="clear ()">清空</button>
                        <button class="search" @click="search ()">搜索</button>
                    </li>
                </ul>
                <div class="post_wrap clearfix" v-if="postLists.length != 0">
                    <div class="post_inner" v-for="(post, index) in postLists" :key="index" @click="detailsPost(post.id, post.companyId)">
                        <div class="post_inner_left">
                            <ul class="inner_left_top">
                                <li>{{post.name}}</li>
                                <li>发布时间：{{post.updateAt | updateAtFilter}}</li>
                            </ul>
                            <p>{{post.compensation | compensationFilter}}</p>
                            <ul class="inner_left_bottom">
                                <li>{{post.city | cityFilter}}-{{post.county | countyFilter}}&emsp;/</li>
                                <li>&emsp;{{post.education | educationFilter}}&emsp;/</li>
                                <li>&emsp;{{post.experience | experienceFilter}}</li>
                            </ul>
                        </div>
                        <div class="post_inner_right">
                            <div class="inner_right_content">
                                <p>{{post.companyName}}</p>
                                <ul>
                                    <li v-for="(industry, index) in post.industryList" :key="index" v-if="index <= 2">{{industry.industry | industryFilter}}</li>
                                </ul>
                                <ul class="right_content_tags">
                                    <li v-for="(tag, index) in post.tags" :key="index">{{tag.tag}}</li>
                                </ul>
                            </div>
                            <img :src="post.logo">
                        </div>
                    </div>
                    <!--分页控件-->
                    <div class="post_pagination">
                        <div>
                            <label for="perpage">每页显示数：</label>
                            <input id="perpage" type="text" v-model="inputPerPage" maxlength="2">
                            <label for="goto">去第几页：</label>
                            <input id="goto" type="text" v-model="inputGotoPage" maxlength="3">
                            <button @click="confirm ()">确认</button>
                            <button @click="reset ()">重置</button>
                        </div>
                        <b-pagination v-model="currentPage" :per-page="perPage" :total-rows="totalRows" first-text="首页" last-text="末页"></b-pagination>
                    </div>
                </div>
                <!-- 搜索为空页面 -->
                <div class="ufo" v-if="postLists.length === 0">
                    <img src="/static/ufo.png">
                    <ul>
                        <li>暂时没有符合该搜索条件的职位</li>
                        <li>请重新修改搜索条件后再进行搜索</li>
                    </ul>
                </div>
                <!-- 搜索为空页面-推荐职位 -->
                <div class="company_list row suggestCompany" v-if="postLists.length === 0">
                    <p class="suggestCompany_title">推荐职位</p>
                    <div class="company_content col-md-4" v-for="(post, index) in suggestPostLists" :key="index" @click="detailsPost(post.id, post.companyId)">
                        <div class="company_details clearfix">
                            <div class="company_content_top clearfix">
                                <img class="company_logo" :src="post.logo">
                                <div class="company_content">
                                    <p class="company_name">{{post.name}}</p>
                                    <p class="company_slogan">{{post.compensation | compensationFilter}}</p>
                                    <p class="company-job">{{post.companyName}}</p>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<style lang="less">
    .company_search_wrap {
        background: #f6f7f8;
        .nav-tabs {
            border-bottom: 2px solid #fc2b47;
            li {
                a {
                    padding: .07rem .2rem;
                    border: none;
                    font-size: .16rem;
                    color: #536a82;
                    &:hover {
                        background: none;
                        border: none;
                    }
                }
            }
            li.active {
                a {
                    border: none;
                    border-radius: 0;
                    background: #fc2b47;
                    color: #fff;
                }
            }
        }
        .search_wrap {
            display: flex;
            flex-direction: column;
            padding: .15rem .2rem;
            background: #fff;
            li {
                display: flex;
                align-items: center;
                font-size: .14rem;
                p {
                    color: #2d3e50;
                    font-weight: 700;
                }
                input[type = radio] {
                    line-height: 1;
                    -webkit-appearance: none;
                    outline: none;
                    &:checked + label {
                        background: #fc2b47;
                        color: #fff;
                    }
                }
                label {
                    padding: .05rem .13rem;
                    font-weight: 400;
                    color: #536a82;
                }
            }
            .keyword {
                margin-bottom: .1rem;
                input {
                    margin-right: .2rem;
                    width: 4rem;
                    height: .4rem;
                }
                input + p {
                    font-weight: 400;
                    color: #999;
                }
            }
            .buttons {
                align-self: flex-end;
            }
        }
    }
    .buttons {
        button {
            margin-left: .1rem;
            width: .6rem;
            height: .25rem;
            font-size: .14rem;
        }
        .search {
            background: #fc2b47;
            color: #fff;
        }
        .clear {
            background: #fff;
            color: #536a82;
        }
    }
    .company_wrap {
        margin: .2rem 0;
        padding-bottom: .65rem;
        background: #fff;
    }
    .company_list {
        padding: .05rem;
        margin: 0;
    }
    .company_content {
        margin: .15rem 0;
    }
    .company_details {
        border: .01rem solid #d9d9d9;
        border-radius: .05rem;
        &:hover {
         box-shadow: 0 0 .1rem #d9d9d9;
        }
    }
    .company_logo {
        float: left;
        margin: .23rem;
        width: 1.15rem;
        height: 1.15rem;
    }
    .company_content {
        float: left;
        margin-top: .23rem;
    }
    .company_name {
        margin-bottom: .14rem;
        font-size: .16rem;
        font-weight: 600;
        color: #2d3e50;
    }
    .company_slogan {
        margin-bottom: .13rem;
        font-size: .14rem;
        color: #536a82;
    }
    .company-job {
        font-size: .14rem;
        color: #536a82;
        span {
            color: #fc2a48;
        }
    }
    .company_content_bottom {
        padding: .1rem 0;
        border-bottom-right-radius: .05rem;
        border-bottom-left-radius: .05rem;
        background: #fafafa;
        ul {
            display: flex;
            justify-content: space-around;
        }
        li {
            float: left;
            text-align: center;
            i, span {
                vertical-align: middle;
            }
            span {
                margin-left: .05rem;
            }
        }
    }
    .company_industry {
        display: inline-block;
        width: .14rem;
        height: .13rem;
        background: url("../assets/findElite/img-phone.png");
    }
    .company_financing {
        display: inline-block;
        width: .15rem;
        height: .13rem;
        background: url("../assets/findElite/img-up.png");
    }
    .company_city {
        display: inline-block;
        width: .12rem;
        height: .13rem;
        background: url("../assets/findElite/img-address.png");
    }
    .ufo {
        display: flex;
        justify-content: center;
        margin-top: .2rem;
        padding: 1.7rem 0 1.45rem;
        border-bottom: 1px solid #e8e8e8;
        height: 4.95rem;
        background: #fff;
        ul {
            display: flex;
            flex-direction: column;
            justify-content: center;
            li {
                margin-top: .1rem;
                font-size: .2rem;
                font-weight: 700;
                color: #2b3d51;
            }
            li + li {
                font-size: .18rem;
                color: #536a82;
            }
        }
    }
    .suggestCompany {
        margin-bottom: .2rem;
        border-bottom: 1px solid #e8e8e8;
        background: #fff;
        .suggestCompany_title {
            padding: .2rem 0 0 .15rem;
            font-size: .16rem;
            font-weight: 700;
            color: #2b3d51;
        }
    }
    .company_pagination {
        float: right;
        margin: 0 .15rem;
        > div {
            float: left;
            margin-right: .5rem;
            input {
                width: .5rem;
                border: 1px solid #e8e8e8;
                border-radius: .03rem;
            }
            button {
                margin: 0 .05rem;
                padding: .06rem .12rem;
                border: 1px solid #e8e8e8;
                border-radius: .03rem;
                background: #f5f5f7;
                color: #666;
            }
        }
        .pagination {
            margin: 0;
            .page-item {
                .page-link {
                    margin: 0 .05rem;
                    border: 1px solid #e8e8e8;
                    border-radius: .03rem;
                    background: #f5f5f7;
                    color: #666;
                    &:focus {
                        outline: none;
                    }
                }
            }
            .page-item.disabled {
                .page-link {
                    background: #f5f5f7;
                    color: #bbb;
                }
            }
            .page-item.active {
                .page-link {
                    background: #fc2b47;
                    color: #fff;
                }
            }
        }
    }
    .post_wrap {
        padding: 0 .2rem .6rem;
        margin: .2rem 0;
        background: #fff;
        .post_inner {
            display: flex;
            justify-content: space-between;
            margin: .2rem 0;
            padding: .12rem .3rem;
            height: 1.2rem;
            border: 1px solid #e8e8e8;
            border-radius: .04rem;
            &:hover {
                box-shadow: 0 0 .1rem #d9d9d9;
            }
            .post_inner_left {
                display: flex;
                flex-direction: column;
                justify-content: space-between;
                border-right: 1px solid #e8e8e8;
                width: 5.25rem;
                .inner_left_top {
                    display: flex;
                    align-items: center;
                    li {
                        margin-right: .1rem;
                        font-size: .16rem;
                        font-weight: 700;
                        color: #2b3d51;
                    }
                    li + li {
                        font-size: .12rem;
                        font-weight: 400;
                        color: #999;
                    }
                }
                p {
                    font-size: .14rem;
                    font-weight: 700;
                    color: #fc2a48;
                }
                .inner_left_bottom {
                    display: flex;
                    font-size: .14rem;
                    color: #536a82;
                }
            }
            .post_inner_right {
                display: flex;
                justify-content: space-between;
                padding-left: .6rem;
                width: calc(100% - 5.25rem);
                .inner_right_content {
                    display: flex;
                    flex-direction: column;
                    justify-content: space-between;
                    font-size: .14rem;
                    p {
                        font-weight: 700;
                        color: #2d3e50;
                    }
                    ul {
                        display: flex;
                        color: #536a82;
                        li {
                            padding: 0 .1rem;
                            border-left: 1px solid #e8e8e8;
                            border-right: 1px solid #e8e8e8;
                        }
                    }
                    .right_content_tags {
                        li {
                            padding: .05rem .1rem;
                            border: 1px solid #e8e8e8;
                            border-radius: .1rem;
                        }
                    }
                }
                img {
                    width: .94rem;
                    height: 100%;
                }
            }
        }
    }
    .post_pagination {
        .company_pagination;
    }
    @media screen and (max-width: 1200px) {
        .post_wrap {
            .post_inner {
                padding-left: 0;
                padding-right: .2rem;
                .post_inner_left {
                    width: 80%;
                    padding-left: .2rem;
                    .inner_left_bottom {
                        flex-wrap: wrap;
                    }
                }
                .post_inner_right {
                   .post_inner_left;
                   width: 20%;
                   img {
                       display: none;
                   }
                }
            }
        }
    }
    @media screen and (max-width: 768px) {
        .company_search_wrap {
            .search_wrap {
                li {
                    flex-wrap: wrap;
                }
            }
        }
        .company_logo {
            width: .8rem;
            height: .8rem;
        }
        .ufo {
            flex-wrap: wrap;
        }
    }
</style>

<script>
    //导入时间戳过滤器
    import {formatDate} from '../filter.js';
    export default {
        data() {
            return {
                //职位列表
                postLists: [{id: '', companyId: '', logo: '', name: '', updateAt: '', compensation: '', city: '', county: '', education: '', experience: '', companyName: '', industryList: {industry: ''}, tags: {tag: ''}}],
                suggestPostLists: [{id: '', companyId: '', logo: '', name: '', companyName: '', compensation: ''}],
                //职位翻页
                currentPage: 1,
                totalRows: 20,
                perPage: 10,
                //搜索
                filter: {
                    // 职位搜索
                    city: [{name: '不限', id: 'city', value: ''}, {name: '北京', id: 'city_0', value: 1}],
                    industry: [{name: '不限', id: 'industry', value: ''}, {name: '移动互联网', id: 'industry_0', value: 0}, {name: '电子商务', id: 'industry_1', value: 1}, {name: '企业服务', id: 'industry_2', value: 2}, {name: 'O2O', id: 'industry_3', value: 3}, {name: '教育', id: 'industry_4', value: 4}, {name: '金融', id: 'industry_5', value: 5}, {name: '游戏', id: 'industry_6', value: 6}],
                    financing: [{name: '不限', id: 'financing', value: ''}, {name: '无需融资', id: 'financing_0', value: 0}, {name: '天使轮', id: 'financing_1', value: 1}, {name: 'A轮', id: 'financing_2', value: 2}, {name: 'B轮', id: 'financing_3', value: 3}, {name: 'C轮', id: 'financing_4', value: 4}, {name: 'D轮及以上', id: 'financing_5', value: 5}, {name: '上市公司', id: 'financing_6', value: 6}],
                    education: [{name: '不限', id: 'education', value: ''}, {name: '大专', id: 'education_0', value: 0}, {name: '本科', id: 'education_1', value: 1}, {name: '硕士', id: 'education_2', value: 2}, {name: '博士及以上', id: 'education_3', value: 3}],
                    experience: [{name: '不限', id: 'experience', value: ''}, {name: '应届', id: 'experience_0', value: 0}, {name: '1~2年', id: 'experience_1', value: 1}, {name: '3~5年', id: 'experience_2', value: 2}, {name: '6~9年', id: 'experience_3', value: 3}, {name: '10年及以上', id: 'experience_4', value: 4}],
                    compensation: [{name: '不限', id: 'compensation', value: ''}, {name: '8K', id: 'compensation_0', value: 0}, {name: '8-15K', id: 'compensation_1', value: 1}, {name: '16-25K', id: 'compensation_2', value: 2}, {name: '26K及以上', id: 'compensation_3', value: 3}],
                    updateAt: [{name: '不限', id: 'updateAt', value: ''}, {name: '今天', id: 'updateAt_0', value: 0}, {name: '3天内', id: 'updateAt_1', value: 1}, {name: '7天内', id: 'updateAt_2', value: 2}]
                },
                //职位搜索绑定数据
                cityModel: '',
                industryModel: '',
                financingModel: '',
                experienceModel: '',
                educationModel: '',
                compensationModel: '',
                updateAtModel: '',
                companyKeywordModel: '',
                postKeywordModel: '',
                //标签判断
                tabCheck: window.sessionStorage.getItem('post'),
                //跳转页面输入
                inputGotoPage: '',
                //每页显示输入
                inputPerPage: ''
            }
        },
        mounted() {
            this.pageFrom ();
            this.getPostLists (3, 1, '', '', '', '', '', '', 1);
        },
        methods: {
        //职位
            //职位列表请求
            getPostLists: function (size, page, name, industry, education, experience, compensation, updateAt, recommend) {
                this.$http.get("/carrots-ajax/a/profession/search", {params: {size: size, page: page, name: name, industry: industry, education: education, experience: experience, compensation: compensation, updateAt: updateAt, recommend: recommend}})
                .then((resp) => {
                    if (size == 3) {
                        this.suggestPostLists = resp.data.data;
                    }
                    else {
                        this.postLists = resp.data.data;
                        this.totalRows = resp.data.total;
                        console.log('职位列表');
                        console.log(this.postLists);
                    }
                })
                .catch((resp) => {
                    console.log("服务器无响应");
                })
            },
            //职业翻页
            page () {
                let post = window.sessionStorage.getItem('post');
                if (post == 0) {
                    this.getPostLists (this.perPage, this.currentPage, this.postKeywordModel, this.industryModel, this.educationModel, this.experienceModel, this.compensationModel, this.updateAtModel, 0);
                }
                else if (post == 1) {
                    this.getPostLists (this.perPage, this.currentPage, this.postKeywordModel, this.industryModel, this.educationModel, this.experienceModel, this.compensationModel, this.updateAtModel, 1);
                }
            },
            //点击职位跳转
            detailsPost (id, companyId) {
                this.$router.push({path: 'detailsPost', query: {id: id, companyId: companyId}});
            },
            //职位搜索按钮
            search () {
                this.page ();
                this.$router.push({query: {postKeyword: this.postKeywordModel}});
                this.postKeywordModel = null;
            },
            //职位清空按钮
            clear () {
                this.postKeywordModel = '';
                this.cityModel = '';
                this.industryModel = '';
                this.educationModel = '';
                this.experienceModel = '';
                this.compensationModel = '';
                this.updateAtModel = '';
                this.search ();
            },
            //判断来源
            pageFrom () {
                let pageCheck = window.sessionStorage.getItem('post');
                if (pageCheck == 0) {
                    this.getPostLists (10, 1, this.$route.query.postKeyword, '', '', '', '', '', 0);
                }
                else if (pageCheck == 1) {
                    this.getPostLists (10, 1, this.$route.query.postKeyword, '', '', '', '', '', 1);
                }
            },
            //页码确认按钮
            confirm () {
                //正整数正则
                let num = /^\d+$/;
                //取得最大页码
                let max = Math.ceil(this.totalRows / this.perPage);
                //最小页码和最小每页显示个数
                let min = 1;
                //确认
                if ((num.test(this.inputGotoPage) && this.inputGotoPage <= max && this.inputGotoPage >= min) || (num.test(this.inputPerPage)  && this.inputPerPage >= min)) {
                    if (!this.inputGotoPage) {
                        this.inputGotoPage = 1;
                    }
                    else if (!this.inputPerPage) {
                        this.inputPerPage = this.perPage;
                    }
                    this.perPage = parseInt(this.inputPerPage);
                    this.currentPage = parseInt(this.inputGotoPage);
                    this.inputGotoPage = '';
                    this.inputPerPage = '';
                    console.log(this.perPage);
                    console.log(this.currentPage);
                }
            },
            //页码重置
            reset () {
                this.perPage = 10;
                this.currentPage = 1;
            }
        },
        watch: {
            //监听职位翻页
            currentPage: function() {
                this.page ();
            },
            //监听每页显示
            perPage: function() {
                this.page ();
            },
            //监听翻页滚动
            currentPage: function () {
                $('html').scrollTop(0);
            }
        },
        filters: {
            //时间戳过滤
            updateAtFilter (time) {
                let date = new Date(time);
                return formatDate(date, 'MM-dd');
            }
        }
    }
</script>
