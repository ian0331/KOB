<template>
    <div class="container">
        <div class="row">
            <div class="col-3">
                <div class="card" style="margin-top: 20px;">
                    <div class="card-body">
                        <img :src="$store.state.user.photo" alt="" style="width: 100%; ">
                    </div>
                </div>
            </div>
            <div class="col-9">
                <div class="card" style="margin-top: 20px;">
                    <div class="card-header">
                        <span style="font-size: 130%;">我的Bot</span>
                        <button type="button" class="btn btn-primary float-end" data-bs-toggle="modal"
                            data-bs-target="#add-bot-btn">
                            创建Bot
                        </button>

                        <!-- Modal -->
                        <div class="modal fade" id="add-bot-btn" tabindex="-1">
                            <div class="modal-dialog modal-lg">
                                <div class="modal-content">
                                    <div class="modal-header">
                                        <h1 class="modal-title fs-5">创建Bot</h1>
                                        <button type="button" class="btn-close" data-bs-dismiss="modal"
                                            aria-label="Close"></button>
                                    </div>
                                    <div class="modal-body">
                                        <div class="mb-3">
                                            <label for="add-bot-title" class="form-label">名称</label>
                                            <input type="text" class="form-control" id="add-bot-title"
                                                placeholder="请输入Bot的名称" v-model="botadd.title">
                                        </div>
                                        <div class="mb-3">
                                            <label for="add-bot-description" class="form-label">简介</label>
                                            <textarea class="form-control" id="add-bot-description" rows="3"
                                                placeholder="请输入Bot的简介" v-model="botadd.description"></textarea>
                                        </div>
                                        <div class="mb-3">
                                            <label for="add-bot-code" class="form-label">代码</label>
                                            <VAceEditor v-model:value="botadd.content" @init="editorInit" lang="c_cpp"
                                                theme="textmate" style="height: 300px" />
                                        </div>
                                    </div>
                                    <div class="modal-footer">
                                        <div class="error-msg">{{ botadd.error_message }}</div>
                                        <button type="button" class="btn btn-secondary" data-bs-dismiss="modal"
                                            @click="add_bot">提交</button>
                                        <button type="button" class="btn btn-primary" data-bs-dismiss="modal">取消</button>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                    <div class="card-body">
                        <table class="table table-striped table-hover">
                            <thead>
                                <tr>
                                    <th>名称</th>
                                    <th>创建时间</th>
                                    <th>操作</th>
                                </tr>
                            </thead>
                            <tbody>
                                <tr v-for="bot in bots" :key="bot.id">
                                    <td>{{ bot.title }}</td>
                                    <td>{{ bot.createtime }}</td>
                                    <td>
                                        <button type="button" class="btn btn-secondary" style="margin-right: 10px;"
                                            :data-bs-target="'#update-bot-modal-' + bot.id"
                                            data-bs-toggle="modal">修改</button>
                                        <button type="button" class="btn btn-danger"
                                            @click="remove_bot(bot)">删除</button>


                                        <!-- Modal -->
                                        <div class="modal fade" :id="'update-bot-modal-' + bot.id" tabindex="-1">
                                            <div class="modal-dialog modal-lg">
                                                <div class="modal-content">
                                                    <div class="modal-header">
                                                        <h1 class="modal-title fs-5">修改Bot</h1>
                                                        <button type="button" class="btn-close" data-bs-dismiss="modal"
                                                            aria-label="Close"></button>
                                                    </div>
                                                    <div class="modal-body">
                                                        <div class="mb-3">
                                                            <label for="add-bot-title" class="form-label">名称</label>
                                                            <input type="text" class="form-control" id="add-bot-title"
                                                                placeholder="请输入Bot的名称" v-model="bot.title">
                                                        </div>
                                                        <div class="mb-3">
                                                            <label for="add-bot-description"
                                                                class="form-label">简介</label>
                                                            <textarea class="form-control" id="add-bot-description"
                                                                rows="3" placeholder="请输入Bot的简介"
                                                                v-model="bot.description"></textarea>
                                                        </div>
                                                        <div class="mb-3">
                                                            <label for="add-bot-code" class="form-label">代码</label>
                                                            <VAceEditor v-model:value="bot.content" @init="editorInit"
                                                                lang="c_cpp" theme="textmate" style="height: 300px" />
                                                        </div>
                                                    </div>
                                                    <div class="modal-footer">
                                                        <div class="error-msg">{{ bot.error_message }}</div>
                                                        <button type="button" class="btn btn-secondary"
                                                            data-bs-dismiss="modal"
                                                            @click="update_bot(bot)">保存修改</button>
                                                        <button type="button" class="btn btn-primary"
                                                            data-bs-dismiss="modal">取消</button>
                                                    </div>
                                                </div>
                                            </div>
                                        </div>

                                    </td>
                                </tr>
                            </tbody>
                        </table>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import { reactive, ref } from 'vue';
import $ from 'jquery';
import { useStore } from 'vuex';
import { Modal } from 'bootstrap/dist/js/bootstrap';
import { VAceEditor } from "vue3-ace-editor";
import ace from "ace-builds";
import "ace-builds/src-noconflict/mode-c_cpp";
import "ace-builds/src-noconflict/theme-textmate";
export default {
    components: {
        VAceEditor
    },
    setup() {
        const store = useStore();
        let bots = ref([]);
        ace.config.set(
            "basePath",
            "https://cdn.jsdelivr.net/npm/ace-builds@" + require('ace-builds').version + "/src-noconflict/");
        ace.require("ace/mode/c_cpp");
        ace.require("ace/theme/textmate");
        const botadd = reactive({
            title: '',
            description: '',
            content: '',
            error_message: ''
        });

        const refresh_bots = () => {
            $.ajax({
                url: "http://localhost:3000/user/bot/getlist/",
                type: "GET",
                headers: {
                    "Authorization": "Bearer " + store.state.user.token
                },
                success(resp) {
                    bots.value = resp;
                }
            })
        }
        refresh_bots();
        const add_bot = () => {
            $.ajax({
                url: "http://localhost:3000/user/bot/add/",
                type: "POST",
                data: {
                    title: botadd.title,
                    description: botadd.description,
                    content: botadd.content
                },
                headers: {
                    "Authorization": "Bearer " + store.state.user.token
                },
                success(resp) {
                    if (resp.error_msg === "success") {
                        botadd.title = '';
                        botadd.description = '';
                        botadd.content = '';
                        Modal.getInstance('#add-bot-btn').hide();
                        refresh_bots();
                    } else {
                        botadd.error_message = resp.error_message;
                    }
                }
            })
        }
        const update_bot = (bot) => {
            botadd.error_message = '';
            $.ajax({
                url: "http://localhost:3000/user/bot/update/",
                type: "POST",
                data: {
                    bot_id: bot.id,
                    title: bot.title,
                    description: bot.description,
                    content: bot.content
                },
                headers: {
                    "Authorization": "Bearer " + store.state.user.token
                },
                success(resp) {
                    if (resp.error_msg === "success") {
                        Modal.getInstance('#update-bot-modal-' + bot.id).hide();
                        refresh_bots();
                    } else {
                        bot.error_message = resp.error_message;
                    }
                }
            })
        }
        const remove_bot = (bot) => {
            $.ajax({
                url: "http://localhost:3000/user/bot/remove/",
                type: "POST",
                data: {
                    bot_id: bot.id
                },
                headers: {
                    "Authorization": "Bearer " + store.state.user.token
                },
                success(resp) {
                    if (resp.error_msg === "success") {
                        refresh_bots();
                    }
                }
            })
        }
        return {
            bots,
            botadd,
            update_bot,
            add_bot,
            remove_bot
        }
    }
}
</script>

<style scoped>
.error-msg {
    color: red;
}
</style>