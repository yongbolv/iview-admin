<template>
    <Menu ref="sideMenu" :active-name="currentPageName" :open-names="openedSubmenuArr" :theme="$store.state.menuTheme" width="auto" @on-select="changeMenu">
        <slot name="top"></slot>
        <template v-for="item in routers">
            <MenuItem v-if="item.children.length<=1" :name="item.children[0].name" :key="item.path">
                <Icon :type="item.icon" :size="iconSize" :key="item.path"></Icon>
                <span class="layout-text" :key="item.path">{{ item.title }}</span>
            </MenuItem>

            <Submenu v-if="item.children.length>1" :name="item.name" :key="item.path">
                <template slot="title">
                    <Icon :type="item.icon" :size="iconSize"></Icon>
                    <span class="layout-text">{{ item.title }}</span>
                </template>
                <template v-for="child in item.children">
                    <MenuItem :name="child.name" :key="child.name">
                        <Icon :type="child.icon" :size="iconSize" :key="child.name"></Icon>
                        <span class="layout-text" :key="child.name">{{ child.title }}</span>
                    </MenuItem>
                </template>
            </Submenu>
        </template>
    </Menu>
</template>

<script>
export default {
    data () {
        return {
            currentPageName: this.$route.name,
            tagsList: this.$store.state.tagsList,
            openedSubmenuArr: this.$store.state.openedSubmenuArr
        };
    },
    name: 'sidebarMenu',
    props: {
        slotTopClass: String,
        routers: Array,
        iconSize: Number
    },
    methods: {
        changeMenu (active) {
            let pageOpenedList = this.$store.state.pageOpenedList;
            let openedPageLen = pageOpenedList.length;
            let i = 0;
            let tagHasOpened = false;
            while (i < openedPageLen) {
                if (active === pageOpenedList[i].name) {  // 页面已经打开
                    this.$store.commit('moveToSecond', i);
                    tagHasOpened = true;
                    break;
                }
                i++;
            }
            if (!tagHasOpened) {
                let tag = this.tagsList.filter((item) => {
                    if (item.children) {
                        return active === item.children[0].name;
                    } else {
                        return active === item.name;
                    }
                });
                tag = tag[0];
                tag = tag.children ? tag.children[0] : tag;
                this.$store.commit('increateTag', tag);
            }
            this.$store.commit('setCurrentPageName', active);
            this.$router.push({
                name: active
            });
        }
    },
    watch: {
        '$route' (to) {
            this.currentPageName = to.name;
            localStorage.currentPageName = to.name;
        },
        currentPageName () {
            this.openedSubmenuArr = this.$store.state.openedSubmenuArr;
            this.$nextTick(
                this.$refs.sideMenu.updateOpened()
            );
        }
    },
    updated () {
        this.$nextTick(
            this.$refs.sideMenu.updateOpened()
        );
    }

};
</script>
