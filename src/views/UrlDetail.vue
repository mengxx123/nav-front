<template>
    <my-page :title="title" :page="page" backable>
        <ui-article v-if="link">
            <div>
                <ui-text-field v-model="link.title" label="标题" />
            </div>
            <div>
                <ui-text-field v-model="link.url" label="网址" hintText="http://" />
            </div>
            <div>
                <ui-text-field v-model="link.icon" label="图标网址（不填则显示默认图标）" />
            </div>
            <div class="btns">
                <ui-raised-button class="btn" label="保存" primary @click="finish" />
                <!-- <ui-raised-button class="btn" label="取消" @click="cancel" /> -->
            </div>
        </ui-article>
    </my-page>
</template>

<script>
    /* eslint-disable */
    const saveAs = window.saveAs

    export default {
        data () {
            return {
                title: '链接编辑',
                link: {
                    title: '',
                    url: '',
                    icon: ''
                },
                links: [
                    {
                        id: '1',
                        icon: 'https://tool.yunser.com/static/img/app-tool.png',
                        title: '云设工具',
                        url: 'https://tool.yunser.com/'
                    },
                    {
                        id: '2',
                        icon: 'https://tool.yunser.com/static/img/app-tool.png',
                        title: '百度',
                        url: 'https://www.baidu.com/'
                    }
                ],
                page: {
                    menu: [
                    ]
                }
            }
        },
        mounted() {
            this.init()
            // this.debug()
        },
        methods: {
            init() {
                let { id } = this.$route.params
                if (id) {
                    this.editType = 'update'
                    let url = `/urls/${id}`
                    this.$http.get(url)
                        .then(response => {
                            console.log('个人信息', response.data)
                            this.link = response.data
    
                        },
                        response => {
                            console.log(response)
                        })
                } else {
                    this.editType = 'add'
                    this.title = '添加链接'
                }
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
                // if (!this.link.icon) {
                //     this.link.icon = 'https://tool.yunser.com/static/img/app-tool.png'
                // }
                if (this.editType === 'update') {
                    this.$http.put(`/urls/${this.link.id}`, this.link)
                        .then(response => {
                            this.$router.go(-1)
                        },
                        response => {
                            console.log(response)
                        })
                } else {
                    this.$http.post(`/urls`, this.link)
                        .then(response => {
                            this.$router.go(-1)
                        },
                        response => {
                            console.log(response)
                        })
                }

                // if (this.isAdd) {
                //     this.link.id = new Date().getTime()
                //     this.links.unshift(this.link)
                //     this.$storage.set('links', this.links)
                // } else {
                //     for (let i = 0; i < this.links.length; i++) {
                //         if (this.links[i].id === this.link.id) {
                //             this.links.splice(i, 1, this.link)
                //             this.$storage.set('links', this.links)
                //             break
                //         }
                //     }
                // }
                // this.addBoxVisible = false
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

    .icon {
        width: 40px;
        height: 40px;
    }

    .nav-list {
        @include clearfix;
        .item {
            float: left;
            width: 120px;
            height: 120px;
            padding: 16px;
            margin-right: 16px;
            margin-bottom: 16px;
            background-color: #fff;
            box-shadow: 0 1px 6px rgba(0,0,0,.117647), 0 1px 4px rgba(0,0,0,.117647);
        }
        .link {
            display: block;
            // height: 100%;
            
            // text-align: center;
        }
        .logo {
            width: 40px;
            height: 40px;
            margin-bottom: 8px;
        }
        .title {
            color: #333;
            max-height: 40px;
            overflow: hidden;
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
