```js
// layout/index.vue
/*
* -class="classObj"
* -- device sidebar @click="handleClickOutSide"
* -- class="hasTagsView:needTagsView"
* --- class="{fixed-header:fixedHeader}"
* ---- needTagsView
*/

computed:{
    ...mapState({
        theme: state => state.setting.theme,
        sideTheme: state=> state.setting.sideTheme,
        sidebar: state=> state.app.sidebar,
        device: state=>state.app.device,
        needTagsView: state => state.setting.tagsView,
        fixedHeader: state => state.settings.fixedHeader
    }),
    classObj(){
        return {
            hideSidebar: !this.sidebar.opened,
            openSidebar: this.sidebar.opened,
            withoutAnimation: this.sidebar.withoutAnimation,
            mobile: this.device=== 'mobile'
        }
    }
}
```