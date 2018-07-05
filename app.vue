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
            <Item></Item>
        </div>
        <daily-article></daily-article>
    </div>
</template>

<script>
    import $ from './libs/util.js';

    export default{
        data(){
            return{
                themes: [],
                showThemes: false,
                type: 'recommend',
                recommendList: [],
                dailyTime: $.getTodayTime(),
                isLoading: false,
                list: [],
                themeId: 0
            }
        },
        methods: {
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
