<style lang="less">
    @import "./main.less";
</style>
<template>
    <div class="main" :class="{'main-hide-text': hideMenuText}">
        <div class="lock-screen-back" id="lock_screen_back"></div>
        <div class="sidebar-menu-con" :style="{width: hideMenuText?'80px':'200px', background: $store.state.menuTheme === 'dark'?'#495060':'white'}">
            <sidebar-menu :routers="menuList" :iconSize="14">
                <div slot="top" class="logo-con">i<i>V</i>iew admin</div>
            </sidebar-menu>
        </div>
        <div class="main-content-container":style="{left: hideMenuText?'80px':'200px'}">
            <div class="main-content-out-container">
                <div class="main-header">
                    <div class="navicon-con">
                        <Button type="text" @click="toggleClick">
                            <Icon type="navicon" size="32"></Icon>
                        </Button>
                    </div>
                    <div class="header-middle-con">
                        <div class="main-breadcrumb">
                            <breadcrumb-nav :currentPath="currentPath"></breadcrumb-nav>
                        </div>
                    </div>
                    <div class="header-avator-con">
                        <div @click="handleFullScreen" v-if="showFullScreenBtn" class="full-screen-btn-con">
                            <Tooltip :content="isFullScreen ? '退出全屏' : '全屏'" placement="bottom">
                                <Icon :type="isFullScreen ? 'arrow-shrink' : 'arrow-expand'" :size="23"></Icon>
                            </Tooltip>
                        </div>
                        <div @click="lockScreen" class="lock-screen-btn-con">
                            <Tooltip content="锁屏" placement="bottom">
                                <Icon type="locked" :size="20"></Icon>
                            </Tooltip>
                        </div>
                        <div class="switch-theme-con">
                            <Row class="switch-theme" type="flex" justify="center" align="middle">
                                <theme-dropdown-menu></theme-dropdown-menu>
                            </Row>
                        </div>
                        <div @click="showMessage" class="message-con">
                            <Tooltip :content="messageCount > 0 ? '有' + messageCount + '条未读消息' : '无未读消息'" placement="bottom">
                                <Badge :count="messageCount" dot>
                                    <Icon type="ios-bell-outline" size="20"></Icon>
                                </Badge>
                            </Tooltip>
                        </div>
                        <div class="user-dropdown-menu-con">
                            <Row type="flex" justify="end" align="middle" class="user-dropdown-innercon">
                                <Dropdown trigger="click" @on-click="handleClickUserDropdown">
                                    <a href="javascript:void(0)">
                                        <span class="main-user-name">{{ userName }}</span>
                                        <Icon type="arrow-down-b"></Icon>
                                    </a>
                                    <DropdownMenu slot="list">
                                        <DropdownItem name="ownSpace">个人中心</DropdownItem>
                                        <DropdownItem name="loginout" divided>退出登录</DropdownItem>
                                    </DropdownMenu>
                                </Dropdown>
                                <Avatar src="https://i.loli.net/2017/08/21/599a521472424.jpg" style="background: #619fe7;margin-left: 10px;"></Avatar>
                            </Row>
                        </div>
                    </div>
                </div>
                
                <div class="main-content">
                    <div class="tags-con">
                        <tags-page-opened :pageTagsList="pageTagsList"></tags-page-opened>
                    </div>
                    <div class="single-page-con">
                        <router-view></router-view>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>
<script>
    import sidebarMenu from './main_components/sidebarMenu.vue';
    import tagsPageOpened from './main_components/tagsPageopened.vue';
    import breadcrumbNav from './main_components/breadcrumbNav.vue';
    import themeDropdownMenu from './main_components/themeDropdownMenu.vue';
    import Cookies from 'js-cookie';
    import util from './util.js';
    
    export default {
        components: {
            sidebarMenu,
            tagsPageOpened,
            breadcrumbNav,
            themeDropdownMenu
        },
        data () {
            return {
                spanLeft: 4,
                spanRight: 20,
                menuList: this.$store.state.appRouter,
                tagsList: [],  // 所有页面的页面对象
                pageTagsList: this.$store.state.pageOpenedList,  // 打开的页面的页面对象
                currentPath: this.$store.state.currentPath,  // 当前面包屑数组
                currentPageName: '',
                hideMenuText: false,
                userName: '',
                showFullScreenBtn: window.navigator.userAgent.indexOf('MSIE') < 0,
                isFullScreen: false,
                messageCount: 0,
                lockScreenSize: 0
            };
        },
        methods: {
            init () {
                let tagsList = [];
                this.menuList.map((item) => {
                    if (item.children.length <= 1) {
                        tagsList.push(item.children[0]);
                    } else {
                        tagsList.push(...item.children);
                    }
                });
                this.$store.commit('setTagsList', tagsList);
                this.$store.commit('setCurrentPageName', this.$route.name);
                let pathArr = util.setCurrentPath(this, this.$route.name);
                if (pathArr.length > 2) {
                    this.$store.commit('addOpenSubmenu', pathArr[1].name);
                }
                this.userName = Cookies.get('user');
                let messageCount = 3;
                this.messageCount = messageCount.toString();
            },
            toggleClick () {
                this.hideMenuText = !this.hideMenuText;
            },
            handleClickUserDropdown (name) {
                if (name === 'ownSpace') {
                    util.openPage(this, 'ownspace_index', '个人中心');
                } else if (name === 'loginout') {
                    Cookies.remove('user');
                    Cookies.remove('password');
                    Cookies.remove('hasGreet');
                    this.$Notice.close('greeting');
                    this.$router.push({
                        name: 'login'
                    });
                }
            },
            handleFullScreen () {
                let main = document.getElementById('main');
                if (this.isFullScreen) {
                    if (document.exitFullscreen) {
                        document.exitFullscreen();
                    } else if (document.mozCancelFullScreen) {
                        document.mozCancelFullScreen();
                    } else if (document.webkitCancelFullScreen) {
                        document.webkitCancelFullScreen();
                    } else if (document.msExitFullscreen) {
                        document.msExitFullscreen();
                    }
                } else {
                    if (main.requestFullscreen) {
                        main.requestFullscreen();
                    } else if (main.mozRequestFullScreen) {
                        main.mozRequestFullScreen();
                    } else if (main.webkitRequestFullScreen) {
                        main.webkitRequestFullScreen();
                    } else if (main.msRequestFullscreen) {
                        main.msRequestFullscreen();
                    }
                }
            },
            showMessage () {
                util.openPage(this, 'message_index', '消息中心');
            },
            lockScreen () {
                let lockScreenBack = document.getElementById('lock_screen_back');
                lockScreenBack.style.zIndex = 10000;
                lockScreenBack.style.boxShadow = '0 0 0 ' + this.lockScreenSize + 'px #667aa6 inset';
                this.showUnlock = true;
                this.$store.commit('lock');
                setTimeout(() => {
                    this.$router.push({
                        name: 'locking'
                    });
                }, 800);
            }
        },
        watch: {
            '$route' (to) {
                this.$store.commit('setCurrentPageName', to.name);
                let currentTitle = this.$store.state.tagsList.filter(item => {
                    return item.name === to.name;
                })[0].title;
                let pathArr = util.setCurrentPath(this, to.name, currentTitle);
                if (pathArr.length > 2) {
                    this.$store.commit('addOpenSubmenu', pathArr[1].name);
                }
            }
        },
        mounted () {
            this.init();
            document.addEventListener('fullscreenchange', () => {
                this.isFullScreen = !this.isFullScreen;
            });
            document.addEventListener('mozfullscreenchange', () => {
                this.isFullScreen = !this.isFullScreen;
            });
            document.addEventListener('webkitfullscreenchange', () => {
                this.isFullScreen = !this.isFullScreen;
            });
            document.addEventListener('msfullscreenchange', () => {
                this.isFullScreen = !this.isFullScreen;
            });
            let lockScreenBack = document.getElementById('lock_screen_back');
            let x = document.body.clientWidth;
            let y = document.body.clientHeight;
            let r = Math.sqrt(x * x + y * y);
            let size = parseInt(r);
            this.lockScreenSize = size;
            window.addEventListener('resize', () => {
                let x = document.body.clientWidth;
                let y = document.body.clientHeight;
                let r = Math.sqrt(x * x + y * y);
                let size = parseInt(r);
                this.lockScreenSize = size;
                lockScreenBack.style.transition = 'all 0s';
                lockScreenBack.style.width = lockScreenBack.style.height = size + 'px';
            });
            lockScreenBack.style.width = lockScreenBack.style.height = size + 'px';

            if (!Cookies.get('hasGreet')) {
                let now = new Date();
                let hour = now.getHours();
                let greetingWord = {
                    title: '',
                    words: ''
                };
                let userName = Cookies.get('user');
                if (hour < 6) {
                    greetingWord = {title: '凌晨好~' + userName, words: '早起的鸟儿有虫吃~'};
                } else if (hour >= 6 && hour < 9) {
                    greetingWord = {title: '早上好~' + userName, words: '来一杯咖啡开启美好的一天~'};
                } else if (hour >= 6 && hour < 9) {
                    greetingWord = {title: '上午好~' + userName, words: '工作要加油哦~'};
                } else if (hour >= 12 && hour < 14) {
                    greetingWord = {title: '中午好~' + userName, words: '午饭要吃饱~'};
                } else if (hour >= 14 && hour < 17) {
                    greetingWord = {title: '下午好~' + userName, words: '下午也要活力满满哦~'};
                } else if (hour >= 17 && hour < 19) {
                    greetingWord = {title: '傍晚好~' + userName, words: '下班没事问候下爸妈吧~'};
                } else if (hour >= 19 && hour < 21) {
                    greetingWord = {title: '晚上好~' + userName, words: '工作之余品一品书香吧~'};
                } else {
                    greetingWord = {title: '深夜好~' + userName, words: '夜深了，注意休息哦~'};
                }
                this.$Notice.config({
                    top: 130
                });
                this.$Notice.info({
                    title: greetingWord.title,
                    desc: greetingWord.words,
                    duration: 4,
                    name: 'greeting'
                });
                Cookies.set('hasGreet', 1);
            }
        }
    };
</script>
