<template>
    <div>
        <div class="arcitleTitle">
            <div>
                <el-form :inline="true" :model="formInline" class="demo-form-inline">
                    <el-form-item>
                        <el-select v-model="formInline.type" placeholder="文章类型">
                            <el-option v-for="item in options" :key="item.value" :label="item.label" :value="item.value"></el-option>
                        </el-select>
                    </el-form-item>
                    <el-form-item class='el-form-item2'>
                        <el-input v-model="formInline.title" placeholder="文章标题"></el-input>
                    </el-form-item>
                </el-form>
            </div>
        </div>
        <mavon-editor ref=md @imgAdd="$imgAdd" @imgDel="$imgDel" v-model="content" @change="change" :toolbars='toolbars' :subfield='false' style="min-height:500px">
        </mavon-editor>
        <div class="butn">
            <el-button type="primary" v-if="update2" @click='savearticle("updata")'>genx发布</el-button>
            <el-button type="primary" v-else @click='savearticle("write")'>发布</el-button>
            <el-button v-if="update2" @click="black2">genx返回</el-button>
            <el-button v-else @click="black">返回</el-button>
        </div>
    </div>
</template>
<script>
export default {

    name: 'onwrite',
    data() {
        return {
            update2: '',
            content: '', //编辑器实时显示
            Acontent: '',
            updataArcitle: '',
            updataType: '',
            img_file: {}, //图片
            toolbars: {
                bold: true, // 粗体
                italic: true, // 斜体
                header: true, // 标题
                underline: true, // 下划线
                strikethrough: true, // 中划线
                mark: true, // 标记
                superscript: true, // 上角标
                subscript: true, // 下角标
                quote: true, // 引用
                ol: true, // 有序列表
                ul: true, // 无序列表
                link: true, // 链接
                imagelink: true, // 图片链接
                code: true, // code
                table: true, // 表格
                fullscreen: true, // 全屏编辑
                readmodel: true, // 沉浸式阅读
                htmlcode: true, // 展示html源码
                help: true, // 帮助
                /* 1.3.5 */
                undo: true, // 上一步
                redo: true, // 下一步
                trash: true, // 清空
                save: true, // 保存（触发events中的save事件）
                /* 1.4.2 */
                navigation: true, // 导航目录
                /* 2.1.8 */
                alignleft: true, // 左对齐
                aligncenter: true, // 居中
                alignright: true, // 右对齐
                /* 2.2.1 */
                subfield: true, // 单双栏模式
                preview: true, // 预览
            },
            options: [{
                value: 'tj',
                label: '推荐'
            }, {
                value: 'htmlcss',
                label: 'html/css'
            }, {
                value: 'javascript',
                label: 'javascript'
            }, {
                value: 'vue',
                label: 'vue'
            }, {
                value: 'webpack',
                label: 'webpack'
            }, {
                value: 'node',
                label: 'node'
            }],
            value: '',
            formInline: {
                type: '', //文章类型
                title: ''
            }
        }
    },
    methods: {
        //接收到富文本处理后的文章携带标签
        change(val, render) {
            this.Acontent = render;
        },
        opchange(event) {
            this.d_language = event.target.value;
            alert("q")
        },

        savearticle(Type) {
            this.updataType = Type;
            var articleIfo = {
                title: this.formInline.title,
                content: this.Acontent,
                type: this.formInline.type,
                updataType: this.updataType,
                id:this.$route.query.id //这里上传需要修改文章的id
            };
            this.$http.post('/api/upData/article', articleIfo).then((data) => {
                var articleState = data.body;
                this.open();
                this.formInline.title = "";
                this.content = "";
                this.formInline.type = "";
            }, (data) => {
                console.log("err");
            })
        },
        open() {
            this.$message({
                message: '恭喜你，文章发布成功',
                type: 'success'
            });
        },
        black() {
            this.$router.push({ name: 'indexpage' });
        },
        black2() {
            this.$router.go(-1); //返回上一层
            this.update2 = false;

        },
        // 绑定@imgAdd event
        $imgAdd(pos, $file) {
            // 第一步.将图片上传到服务器.
            var img = $file;
            this.$http.post('/api/login/upImg', { img: img }).then((data) => {
                var imgageUrl = data.body;
                this.$refs.md.$img2Url(pos, imgageUrl);
            }, (data) => {
                console.log("err");
            })
        },
        $imgDel() {
            delete this.img_file[pos];
        },
        //监控路由如果用户从管理页面重新进入发布文章页面，还原发布按钮
        routequery() {
            if (JSON.stringify(this.$route.query) == "{}") {
                this.update2 = false;
            } else {
                this.update2 = this.$route.query.update
            }

        }
    },
    created() {
        this.update2 = this.$route.query.update;
        if (this.update2 == true) {
            //这里拿到this.$route.query.id发起请求将要修改的文章请求过来
            this.$http.get("/api/login/updata", { params: { updataId: this.$route.query.id } }).then((res) => {
                this.updataArcitle = res.data[0];
                this.formInline.type = this.updataArcitle.type;
                this.formInline.title = this.updataArcitle.title;
                this.content = this.updataArcitle.content;
                console.log(this.updataArcitle.content);
            });
        }
    },
    watch: {
        "$route.query": "routequery"
    }
}
</script>
<style scoped>
.demo-form-inline {
    display: flex;
    justify-content: start;
    align-items: center
}

.arcitleTitle {
    padding: 0px 0px 20px;
}

.el-form-item {
    margin-bottom: 0px;
}

.el-form-item2 .el-input input {
    width: 100%;
}

.butn {
    display: flex;
    padding: 20px 0;
}
</style>