<template>
    <div>
        <dynamic-table ref="table" :pageConfig="config" @handleClick="handleClick">

        </dynamic-table>

        <a-modal dialogClass="form-modal" v-model="visible" centered
                 title="评价列表" @cancel="handleClose"
                 :footer="null">
            <a-spin :spinning="loading" style="width: 100%;">
                <a-icon slot="indicator" type="loading" style="font-size: 30px; margin: 0 auto;" spin />
                <a-list
                    class="comment-list"
                    :header="`${this.commentList.length}条评论`"
                    item-layout="horizontal"
                    :data-source="this.commentList"
                >
                    <a-list-item slot="renderItem" slot-scope="item, index">
                        <a-comment>
                            <template slot="author">
                                <p style="font-size: 16px;">{{ item.nickname ? item.nickname : '匿名用户' }}</p>
                            </template>
                            <template slot="avatar">
                                <a-avatar
                                    shape="circle"
                                    size="default"
                                    :style="{ backgroundColor: colorList[(index+1)%4], verticalAlign: 'middle' }">
                                    {{ item.nickname ? item.nickname[0] : '用户' }}
                                </a-avatar>
                            </template>
                            <p slot="content">
                                {{ item.comments }}
                            </p>
                            <template slot="actions">
                                <p style="font-size: 13px;">{{ item.createTime }}</p>
                            </template>
                            <template slot="datetime">
                                <a-rate :default-value="item.score" disabled />
                            </template>
                        </a-comment>
                    </a-list-item>
                </a-list>
            </a-spin>


        </a-modal>
    </div>
</template>

<script>
import {pageConfig} from "./driverManagementConfig";

export default {
    name: "clientManagement",
    data() {
        return {
            config: pageConfig,

            commentList: [],
            loading: true,

            visible: false,
            labelCol: {span: 6},
            wrapperCol: {span: 17},
            layout: 'horizontal',

            colorList: ['#f56a00', '#7265e6', '#ffbf00', '#00a2ae']
        }
    },
    methods: {
        handleClick(event, row) {
            switch (event) {
                case 'comment':
                    this.loading = true;
                    this.visible = true;
                    this.getCommentData(row.id)
                    break;
                case 'open':
                    this.$modal.confirm({
                        title: '启用代驾',
                        content: '确认要启用代驾【'+row.name+'】吗？',
                        onOk: () => {
                            this.updateState(row.id, 0)
                        }
                    })
                    break;
                case 'close':
                    this.$modal.confirm({
                        title: '禁用代驾',
                        content: '确认要禁用代驾【'+row.name+'】吗？',
                        onOk: () => {
                            this.updateState(row.id, 1)
                        }
                    })
                    break;
            }
        },
        handleClose() {
            this.commentList = []
        },
        getCommentData(driverId) {
            this.$request({
                url: '/drivercomment/selectPage',
                method: 'POST',
                data: {
                    driverId: driverId
                }
            }).then(res => {
                if (res.data.status == '200') {
                    this.commentList = res.data.data.list;
                    console.log(this.commentList)
                    this.loading = false;
                }else {
                    this.$message.warning(res.data.desc);
                }
            })
        },
        updateState(id, state) {
            this.$request({
                url: '/driver/updateState',
                method: 'POST',
                data: {
                    id: id,
                    state: state
                }
            }).then(res => {
                if (res.data.status == '200') {
                    this.$message.success('操作成功');
                    this.$refs.table.refresh();
                }else {
                    this.$message.warning(res.data.desc);
                }
            })
        },
    }
}
</script>

<style>
.form-modal .ant-modal-content .ant-modal-body {
    padding: 5px 23px;
    max-height: calc(100vh - 160px);
    overflow-y: auto;
}

.form-modal {
    width: 80%!important;
}

.form-modal .ant-rate {
    font-size: 15px;
}

.form-modal .ant-rate .ant-rate-star:not(:last-child) {
    margin: 1px;
}

.form-modal .ant-comment {
    width: 100%;
}

.form-modal .ant-comment-content-author-time {
    color: rgba(0, 0, 0, 0.65);
}

.form-modal .ant-comment-actions {
    display: flex;
    flex-direction: row-reverse;
}

.main-box .login-box input::-webkit-outer-spin-button,
.main-box .login-box input::-webkit-inner-spin-button {
    -webkit-appearance: none;
}

.main-box .login-box input[type="number"] {
    -moz-appearance: textfield;
}
</style>