<template>
<div class="ic-container forum-box">
    <div class="wrapper">
        <div class="left-nav">
            <div class="left-nav-box">
                <!-- <span class="post-new-topic">板块列表</span> -->
                <router-link class="ic-btn primary post-new-topic" @mouseover.native="mouseOverPostNewBtn = true" @mouseleave.native="mouseOverPostNewBtn = false" :style="postNewTopicStyle" :to="{ name: 'forum_topic_new', params: {'board_id': boardId } }">发表主题</router-link>
                <div class="ul-boards">
                    <router-link :to="{ name: 'index', query: $route.query}" class="item" :class="{'showAll': !isBoard}" style="margin-top: 10px">
                        <div class="sign"></div>
                        <span class="title">全部主题</span>
                    </router-link>
                    <label v-if="isBoard" class="with-subboard-topic">
                        <input type="checkbox" v-model="withSubBoardTopic"/>
                        <span>包含子板块内容</span>
                    </label>
                    <div style="margin-bottom: 3px;"></div>
                    <router-link v-for="j in dymBoardList" :key="j.id" :class="{'subboard': j.parent_id}" class="item" :to="{ name: 'forum_board', params: {id: j.id}, query: $route.query }">
                        <div v-if="j.parent_id === null" class="sign" :style="lineStyleBG(j.id)"></div>
                        <span class="title" :style="boardNavStyle(j)">{{boardNavTitle(j)}}</span>
                    </router-link>
                </div>
            </div>
        </div>
        <div class="right" id="board-list">
            <top-btns></top-btns>
            <loading v-if="loading"/>
            <div style="flex: 1 0 0%" v-else-if="topics && topics.items.length">
                <!-- 放在这是为了让标题正确刷新，毕竟这部分DOM总会在发生变化时重构 -->
                <template v-if="isBoard">
                    <div v-title v-if="$route.params.page && $route.params.page > 1">{{ board.name }} - 第{{$route.params.page}}页 - {{state.config.title}}</div>
                    <div v-title v-else>{{ board.name }} - {{state.config.title}}</div>
                </template>
                <div v-else v-title>全部主题 - {{state.config.title}}</div>

                <div class="board-item-box" :key="i.id" v-for="i in topics.items"  @mouseover="itemHover(i.id)" @mouseout="itemHover(null)">
                    <router-link :to="{ name: 'forum_topic', params: {id: i.id} }" class="board-item" :class="{'top-post': i.sticky_weight}">
                        <div class="title-recent" style="flex: 10 0 0%">
                            <avatar style="margin-right: 10px;" :user="i.user_id" :size="32" class="avatar"></avatar>
                            <div class="right">
                                <h2>
                                    <router-link :title="i.title" :to="{ name: 'forum_topic', params: {id: i.id} }">
                                        <span>{{i.title}}</span>
                                        <span v-if="i.state === state.misc.POST_STATE.CLOSE">[关闭]</span>
                                    </router-link>
                                    <span class="icons">
                                        <i v-if="i.awesome == 1" class="mdi-icarus icon-diamond" title="优秀" style="color: #e57272" @click.prevent></i>
                                        <i v-if="false" class="mdi-icarus icon-crown" title="精华" style="color: #e8a85d"></i>
                                        <i v-if="isAdmin() && i.id === hoverId" class="mdi-icarus icon-sword-cross animated rotateIn" title="管理" style="color: #71c1ef; cursor: pointer" @click.prevent="setTopicManage(i)"></i>
                                    </span>
                                </h2>
                                <p>
                                    <router-link class="board-badge" :style="lineStyleBG(i.board_id)" :to="{ name: 'forum_board', params: {id: i.board_id} }">{{getBoardInfo(i.board_id).name}}</router-link>
                                    <user-link :user="i.user_id" />
                                    <span> 发布于 <ic-time :timestamp="i.time" /></span>
                                </p>
                            </div>
                            <div class="append-icons">
                                <i v-if="i.sticky_weight" class="mdi-icarus icon-pin" title="置顶" />
                            </div>
                        </div>
                        <div class="detail ic-xs-hidden" style="flex: 9 0 0%">
                            <div class="count-block" style="flex: 4 0 0;">
                                <div class="count">
                                    <p class="num">{{i.s.click_count}}</p>
                                    <p class="txt">点击</p>
                                </div>
                                <div class="count">
                                    <p class="num">{{i.s.comment_count}}</p>
                                    <p class="txt">回复</p>
                                </div>
                            </div>
                            <div class="recent ic-xs-hidden ic-sm-hidden" style="flex: 5 0 0;">
                                <span class="line" :style="lineStyle(i.board_id)"></span>
                                <div class="post" v-if="i.s.last_comment_id && i.s.last_comment_id.id">
                                    <strong><user-link :user="i.s.last_comment_id.user_id" /></strong>
                                    <router-link tag="div" class="post-content" :to="{ name: 'forum_topic', params: {id: i.s.last_comment_id.related_id} }">{{atConvert(i.s.last_comment_id.content)}}</router-link>
                                </div>
                                <div class="post" v-else>○ ○ ○ ○ ○</div>
                                <ic-time v-if="i.s.last_comment_id" class="time" :timestamp="i.s.last_comment_id.time" />
                                <div v-else class="time">从未</div>
                            </div>
                        </div>
                    </router-link>
                </div>
                <paginator v-if="isBoard" :page-info='topics' :route-name='"forum_board"' />
                <paginator v-else :page-info='topics' :route-name='"forum_main"' />
            </div>
            <div style="flex: 1 0 0%" v-else>
                <div style="margin-left: 10px;">尚未有人发言……</div>
            </div>
        </div>
    </div>
    <dialog-topic-manage />
</div>
</template>

<style scoped lang="scss">
.wrapper {
    display: flex;

    .left-nav {
        flex: 5 0 0%;
    }

    .right {
        flex: 19 0 0%;
    }
}

.board-badge {
    padding: 8px;
    color: $light;
    opacity: 0.6;
    // background-color: $gray-400;

    &:hover {
        color: $light;
    }
}

.ul-boards {
    margin: 0;
    list-style: none;

    .item {
        display: flex;
        align-items: center;
        padding: 7px 0;
        font-size: 14px;
        font-weight: bolder;
        color: lighten($gray-600, 10%);

        &.subboard {
            padding-left: 16px;
        }

        &.showAll {
            font-weight: bold;
            color: $primary;
        }

        .sign {
            width: 1em;
            height: 1em;
            background-color: #000;
            border-radius: 3px;
            flex-shrink: 0;
        }

        .title {
            margin-left: 3px;
        }
    }
}

$left-nav-padding-right: 30px;

.left-nav-box {
    padding: 0 $left-nav-padding-right 0 0;
}

.post-new-topic {
    width: 100%;
    display: block;
}

.with-subboard-topic {
    display:block;
    font-size: 14px;
    user-select: none;
    margin-bottom: 7px;
    color: $gray-600;
}
</style>

<script>
import api from '@/netapi.js'
import state from '@/state.js'
import '@/assets/css/_forum.scss'
import TopBtns from './topbtns2.vue'
import nprogress from 'nprogress/nprogress.js'

let pageOneHack = function (to, from, next) {
    // 这一hack的目标是抹除 /r/1 的存在，使其与 / 看起来完全一致
    // 但似乎由于 nprogress 的存在，显得有点僵硬
    if (to.name === 'forum_main' && (to.params.page === '1' || to.params.page === 1)) {
        if (from.name === 'index') {
            state.loading = 0
            nprogress.done()
            return next(false)
        }
        return next({name: 'index', query: to.query})
    }
    next()
}

export default {
    data () {
        return {
            state,
            hoverId: null,
            loading: true,
            topics: null,
            withSubBoardTopic: false,
            withSubBoardTopicOptionReady: false,
            mouseOverPostNewBtn: false
        }
    },
    computed: {
        postNewTopicStyle: function () {
            let exInfo = $.getBoardExInfoById(this.boardId)
            if (this.isBoard && (!this.state.boards.loaded)) {
                return {'background-color': '#777'}
            }
            if (exInfo) {
                if (this.mouseOverPostNewBtn) {
                    return {'background-color': exInfo.colorHover}
                }
                return {'background-color': exInfo.color}
            }
        },
        isBoard: function () {
            return this.$route.name === 'forum_board'
        },
        board: function () {
            let bid = this.boardId
            if (bid) return $.getBoardInfoById(bid)
        },
        boardId: function () {
            if (this.isBoard) {
                return this.$route.params.id
            }
        },
        dymBoardList: function () {
            let lst = []
            let curBoardId = this.boardId
            let chain = $.getBoardChainById(curBoardId)
            let topBoardId = chain[chain.length - 1]

            let pushSubBoards = (i, chainIndex) => {
                let topId = chain[--chainIndex]
                // $.getBoardExInfoById(i.id)
                for (let j of this.state.boards.exInfoMap[i.id].subboards) {
                    lst.push(j)
                    if (j.id === topId) {
                        pushSubBoards(j, chainIndex)
                    }
                }
            }

            for (let i of this.state.boards.lst) {
                lst.push(i)

                if (i.id === topBoardId) {
                    pushSubBoards(i, chain.length - 1)
                }
            }

            return lst
        }
    },
    methods: {
        atConvert: $.atConvert2,
        isAdmin: function () {
            return $.isAdmin()
        },
        boardNavTitle: function (board) {
            if (board.parent_id) {
                let chain = $.getBoardChainById(board.id)
                let prefix = _.times(chain.length - 2, () => '>').join('')
                return `${prefix} ${board.name}`
            }
            return board.name
        },
        boardNavStyle: function (board) {
            if (this.isBoard) {
                let boardId = this.$route.params.id
                let chain = $.getBoardChainById(boardId)
                if (chain.indexOf(board.id) !== -1) {
                    let exInfo = $.getBoardExInfoById(board.id)
                    return {
                        'color': exInfo.color,
                        'font-weight': 'bold'
                    }
                }
            }
        },
        setTopicManage: function (topic) {
            state.dialog.topicManageData = topic
            state.dialog.topicManage = true
        },
        itemHover: function (id) {
            this.hoverId = id
        },
        lineStyle: function (boardId, key = 'border-left-color') {
            return $.lineStyleById(boardId, key)
        },
        lineStyleBG: function (boardId) {
            return $.lineStyleById(boardId, 'background-color')
        },
        getBoardInfo: function (boardId) {
            return $.getBoardInfoById(boardId)
        },
        fetchData: async function () {
            this.loading = true
            let baseQuery = {}
            let params = this.$route.params
            let page = 1

            // 包含子板块内容
            if (localStorage.getItem('sbt')) {
                this.withSubBoardTopic = true
            }
            this.$nextTick(() => {
                this.withSubBoardTopicOptionReady = true
            })

            // 获取全局板块信息
            await $.getBoardsInfo()

            // 具体板块
            if (this.isBoard) {
                // 若是要求包含子板块内容
                if (localStorage.getItem('sbt')) {
                    let lst = [params.id]
                    for (let i of $.getBoardExInfoById(params.id).subboardsAll) {
                        lst.push(i.id)
                    }
                    baseQuery['board_id.in'] = JSON.stringify(lst)
                } else {
                    baseQuery['board_id'] = params.id
                }
                page = params.page
            } else {
                baseQuery['sticky_weight.ne'] = 5
                page = params.page
            }

            let query = this.$route.query
            let order = 'weight.desc, update_time.desc' // 权重降序

            if (query.type === '2' || query.type === 2) {
                // 最近更新：更新时间降序
                order = 'update_time.desc, time.desc'
            }

            if (query.type === '3' || query.type === 3) {
                // 最近发布：发布时间降序
                order = 'time.desc'
            }

            if (query.type === '4' || query.type === 4) {
                // 最近回复：回复时间排序
                // 好吧，这个好像暂时还实现不了
                order = 'update_time.desc, time.desc'
            }

            if (this.isBoard) {
                // 在板块模式下加入当前板块的置顶
                order = 'sticky_weight.desc, ' + order
            }

            let retList = await api.topic.list(Object.assign({
                order: order,
                select: 'id, time, user_id, board_id, title, state, awesome, weight, update_time, sticky_weight',
                loadfk: {'user_id': null, 'id': {'as': 's', loadfk: {'last_comment_id': {'loadfk': {'user_id': null}}}}}
            }, baseQuery), page)
            if (retList.code === api.retcode.SUCCESS) {
                if (!this.isBoard && (!page || page === 1)) {
                    // 首页
                    let retStickyTopics = await api.topic.list({
                        sticky_weight: 5, // 全局置顶项
                        order: order,
                        select: 'id, time, user_id, board_id, title, state, awesome, weight, update_time, sticky_weight',
                        loadfk: {'user_id': null, 'id': {'as': 's', loadfk: {'last_comment_id': {'loadfk': {'user_id': null}}}}}
                    })
                    if (retStickyTopics.code === api.retcode.SUCCESS) {
                        retList.data.items = _.concat(retStickyTopics.data.items, retList.data.items)
                    }
                }

                this.topics = retList.data
                this.loading = false
                return
            } else {
                this.topics = null
            }

            // $.message_by_code(retList.code)
            this.loading = false
        }
    },
    beforeRouteEnter (to, from, next) {
        return pageOneHack(to, from, next)
    },
    beforeRouteUpdate (to, from, next) {
        return pageOneHack(to, from, next)
    },
    created () {
        this.fetchData()
    },
    watch: {
        // 如果路由有变化，会再次执行该方法
        '$route': 'fetchData',
        'withSubBoardTopic': async function (newVal, oldVal) {
            if (this.withSubBoardTopicOptionReady) {
                localStorage.removeItem('sbt', newVal)
                if (newVal) localStorage.setItem('sbt', 1)
                await this.fetchData()
            }
        }
    },
    components: {
        TopBtns
    }
}
</script>
