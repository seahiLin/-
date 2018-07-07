<template>
    <div class="daily">
        <div class="daily-menu">
            <div class="daily-menu-item"
                :calss="{on: type==='recommend'}"
                @click="handleToRecommend">每日推荐</div>
            <div class="daily-menu-item"
                :class="{on: type==='daily'}"
                @click="showThemes= !showThemes">主题日报</div>
            <ul v-show="showThemes">
                <li v-for="item in themes" :key="item.id">
                    <a :class="{on: item.id=== themeId &&type=== 'daily'}"
                        @click="handleToTheme(item.id)">{{item.name}}</a>
                </li>
            </ul>
        </div>
        <div class="daily-list">
            <template v-if="type=== 'recommend'">
                <div v-for="list in recommendList" :key="list.date">
                    <div class="daily-date">{{formatDay(list.date)}}</div>
                    <Item 
                        v-for="item in list.stories"
                        :key="item.id" :data="item"
                        @click.native="handleClick(item.id)">
                    </Item>
                </div>
            </template>
            <template v-if="type=== 'daily'">
                <Item 
                    v-for="item in list"
                    :key="item.id"
                    :data="item"
                    @click.native="handleClick(item.id)">
                </Item>
           </template>
        </div>
    </div>
</template>

<script>
    import $ from './libs/util.js';
    import Item from './component/item.vue';
    export default{
        components: {Item},
        data(){
            return{
                themes: [],
                showThemes: false,
                type: 'recommend',
                recommendList: [],
                dailyTime: $.getTodayTime(),
                isLoading: false,
                list: [],
                articleId: 0,
                themeId: 0
            }
        },
        methods: {
            formatDay(date){
                let month= date.substr(4,2);
                let day= date.substr(6,2);
                if(month.substr(0,1)=== '0') month= month.substr(1,1);
                if(day.substr(0,1)=== '0') day= day.substr(1,1);
                return month + "月" + day + "日";
            },
            handleClick(id){
                this.articleId= id;
            },
            getThemes(){
                $.ajax.get('themes').then(res=> {
                    this.themes= res.others;
                })
            },
            handleToTheme(id){
                //改变菜单分类
                this.type= 'daily';
                //设置当前点击子类的主题日报id
                this.themeId= id;
                //清空中间栏的数据
                this.list= [];
                $.ajax.get('theme/'+ id).then(res=> {
                    //过滤类型为1的文章，该类型下文章为空
                    this.list= res.stories.filter(item=> item.type!= 1);
                });
            },
            handleToRecommend(){
                this.type= 'recommend';
                this.recommendList= [];
                this.dailyTime= $.getTodayTime();
                this.getRecommendList();
            },
            getRecommendList(){
                this.isLoading= true;
                const preDay= $.prevDay(this.dailyTime+ 86400000);
                $.ajax.get('news/before/'+ preDay).then(res=> {
                    this.recommendList.push(res);
                    this.isLoading= false;
                })
            }
        },
        mounted(){
            this.getThemes();
            this.getRecommendList();
        }
    }
</script>

<style scoped>
    div{
        color: blueviolet;
    }
</style>
