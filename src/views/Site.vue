<template>
    <my-page title="网站大全" :page="page">
        <div class="common-container container">
            <div class="total">共收录 {{ total }} 个网站</div>
            <ul class="nav-list">
                <li class="item" v-for="link, index in links">
                    <router-link class="link" :to="`/sites/${link.id}`">
                        <img class="icon" :src="link.icon || '/static/img/nav.svg'" alt="">
                        <h3 class="title">{{ link.name }}</h3>
                    </router-link>
                    <div class="description">{{ link.description }}</div>
                    <div v-if="isSetting">
                        <a href="#" v-if="isSetting" @click="remove(link)">删除</a>
                        | 
                        <a href="#" v-if="isSetting" @click="edit(link, index)">编辑</a>
                    </div>
                    <a class="visit" :href="link.url" target="_blank">访问</a>
                    <br>
                </li>
            </ul>
            <ui-pagination :total="total" :current="curPage" :pageSize="20" @pageChange="handleClick" v-if="curPage"></ui-pagination>
        </div>
    </my-page>
</template>

<script>
    const saveAs = window.saveAs

    export default {
        data () {
            return {
                curPage: 1,
                total: 20,
                isAdd: true,
                link: {
                    title: '',
                    icon: '',
                    url: ''
                },
                links: [
                ],
                addBoxVisible: false,
                isSetting: false,
                page: {
                    menu: [
                        {
                            type: 'icon',
                            icon: 'help',
                            href: 'https://project.yunser.com/products/3733dc70702411e999b6a92ec207965f',
                            target: '_blank',
                            title: '帮助'
                        }
                    ]
                }
            }
        },
        mounted() {
            this.init()
            // this.debug()
        },
        methods: {
            handleClick(newIndex) {
                this.curPage = newIndex
                this.init()
                // this.$router.push('/sites?page=' + newIndex)
            },
            init() {
                // this.curPage = parseInt(this.$route.query.page || 1)
                this.$http.get(`/sites?page=${this.curPage}&page_size=20`)
                    .then(response => {
                        console.log('个人信息', response.data)
                        this.links = response.data
                        this.total = response.headers['x-total']
                    },
                    response => {
                        console.log(response)
                    })
            },
            debug() {
                // this.addBoxVisible = true
                // this.link = {
                //     title: '百度翻译',
                //     logo: 'https://tool.yunser.com/static/img/app-tool.png',
                //     url: 'http://fanyi.baidu.com/'
                // }
                // this.finish()
                this.isSetting = true
            },
            add() {
                this.isAdd = true
                this.addBoxVisible = true
            },
            cancel() {
                this.addBoxVisible = false
            },
            finish() {
                if (!this.link.title) {
                    alert('请输入网站名称')
                    return
                }
                if (!this.link.url) {
                    alert('请输入网址')
                    return
                }
                if (!/http/.test(this.link.url)) {
                    this.link.url = 'http://' + this.link.url
                }
                if (!this.link.icon) {
                    this.link.icon = 'https://tool.yunser.com/static/img/app-tool.png'
                }
                if (this.isAdd) {
                    this.link.id = new Date().getTime()
                    this.links.unshift(this.link)
                    this.$storage.set('links', this.links)
                } else {
                    for (let i = 0; i < this.links.length; i++) {
                        if (this.links[i].id === this.link.id) {
                            this.links.splice(i, 1, this.link)
                            this.$storage.set('links', this.links)
                            break
                        }
                    }
                }
                this.addBoxVisible = false
            },
            option() {
                this.isSetting = !this.isSetting
            },
            remove(link) {
                for (let i = 0; i < this.links.length; i++) {
                    if (this.links[i].id === link.id) {
                        this.links.splice(i, 1)
                        break
                    }
                }
                this.$storage.set('links', this.links)
            },
            edit(link, index) {
                this.addBoxVisible = true
                this.isAdd = false
                console.log('编辑', link)
                this.link.id = link.id
                this.link.title = link.title
                this.link.url = link.url
                this.link.icon = link.icon
            },
            exportData() {
                let cotent = `<!DOCTYPE NETSCAPE-Bookmark-file-1> 
<META HTTP-EQUIV="Content-Type" CONTENT="text/html; charset=UTF-8"> 
<TITLE>Bookmarks</TITLE> 
<H1>Bookmarks</H1> 
<DL> 
`
                for (let link of this.links) {
                    cotent += `    <DT><A HREF="${link.url}">${link.title}</A></DT>` + '\n'
                }
                cotent += '</DL>'
                let blob = new Blob([cotent], {type: 'text/plain;charset=utf-8'})
                saveAs(blob, 'bookmark_' + new Date().getTime() + '.html')
            }
        }
    }
</script>

<style lang="scss" scoped>
    @import '../scss/var';
    .total {
        margin-bottom: 16px;
    }
    .container {
        max-width: 1200px;
    }
    .add-box {
        max-width: 292px;
        padding: 16px;
        background-color: #fff;
        box-shadow: 0 1px 6px rgba(0,0,0,.117647), 0 1px 4px rgba(0,0,0,.117647);
        margin-bottom: 24px;
        .btns {
            .btn {
                margin-right: 8px;
            }
        }
    }
    .nav-list {
        display: flex;
        flex-wrap: wrap;
        // @include clearfix;
        .item {
            position: relative;
            // float: left;
            width: 320px;
            height: 120px;
            padding: 16px;
            margin-right: 16px;
            margin-bottom: 16px;
            background-color: #fff;
            box-shadow: 0 1px 6px rgba(0,0,0,.117647), 0 1px 4px rgba(0,0,0,.117647);
        }
        .link {
            display: block;
            display: flex;
            // height: 100%;
            align-items: center;
            
            // text-align: center;
        }

        .visit {
            position: absolute;
            top: 16px;
            right: 16px;
        }
        .icon {
            width: 40px;
            height: 40px;
            margin-right: 8px;
            // margin-bottom: 8px;
        }
        .title {
            color: #333;
            max-height: 40px;
            overflow: hidden;
        }
        .description {
            text-overflow: -o-ellipsis-lastline;
            overflow: hidden;
            text-overflow: ellipsis;
            display: -webkit-box;
            -webkit-line-clamp: 2;
            line-clamp: 2;
            -webkit-box-orient: vertical;
        }
        .item-add {
            box-shadow: none;
            border: 1px dashed #999;
            line-height: 90px;
            font-size: 60px;
            .title {
                color: #999;
            }
        }
    }
</style>
