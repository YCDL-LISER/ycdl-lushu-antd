<template>

  <div class="">
    <a-row :gutter="16">
      <a-col :span="20">
        <div>
          <slot>
            {{ description }}
          </slot>
        </div>
        <div>
          <a-avatar :src="avatar"/>
          <router-link :to="{path:'/account/center/'+href}">{{ owner }}</router-link> 上传于
          <em>{{ updateAt | formatUptime }}</em>
        </div>
      </a-col>
      <a-col :span="4" v-if="owner">
        <div style="cursor: pointer;text-align: center;margin-top: 2px">
          <span @click="onPraise()">
            <a-icon type="like" theme="twoTone" :twoToneColor="praiseTwoTone.color"/>{{ praiseTwoTone.text }}
          </span>
          &nbsp;
          <span @click="onCollection()">
            <a-icon type="heart" theme="twoTone" :twoToneColor="collectionTwoTone.color"/>{{ collectionTwoTone.text }}
          </span>
        </div>
      </a-col>
    </a-row>
  </div>
</template>

<script>
import moment from 'moment'
import ARow from 'ant-design-vue/es/grid/Row'
import ACol from 'ant-design-vue/es/grid/Col'
import { getSocialStatus, postSocial, deleteSocial } from '@/api/lushu/common'
export default {
  name: 'Uploader',
  components: { ACol, ARow },
  props: {
    description: {
      type: String,
      default: ''
    },
    owner: {
      type: String,
      required: true
    },
    avatar: {
      type: String,
      required: true
    },
    href: {
      type: String,
      required: true
    },
    updateAt: {
      type: String,
      default: ''
    },
    routeId: {
      type: [String, Number],
      required: true
    },
    routeType: {
      type: [String, Number],
      required: true
    }
  },
  data () {
    return {
      praiseTwoTone: {
        color: '',
        text: '点赞'
      },
      collectionTwoTone: {
        color: '',
        text: '收藏'
      },
      routeSocial: {
        socialType: '',
        routeId: '',
        routeType: '',
        userId: ''
      },
      routeSocialStatus: {
        praise: false,
        collection: false
      }
    }
  },
  filters: {
    formatUptime (date) {
      if (date) {
        return moment(Number(date)).format('YYYY-MM-DD HH:mm:ss')
      }
    }
  },
  created () {
    this.routeSocial.routeType = this.routeType
    const { userId } = this.$store.getters
    if (userId) {
      this.routeSocial.userId = userId
    }
  },
  watch: {
    routeId (val) {
      this.routeSocial.routeId = val
      this.querySocialStatus()
    }
  },
  methods: {
    needLoginIn () {
      if (!this.$store.getters.loadedUserDetails) {
        const router = this.$router
        const fullPath = this.$route.fullPath
        this.$confirm({
          title: '提示',
          content: '是否立即登录?',
          onOk () {
            router.push({ path: '/user/login', query: { redirect: fullPath } })
          },
          onCancel () {
          }
        })
        return true
      } else {
        return false
      }
    },
    querySocialStatus () {
      if (!this.routeSocial.routeId) {
        return
      }
      if (!this.$store.getters.loadedUserDetails) {
        return
      }
      getSocialStatus(this.routeSocial.routeId).then(res => {
        const { praise, collection } = res.data
        this.routeSocialStatus.praise = praise
        this.routeSocialStatus.collection = collection
        this.twoToneColorChange()
      })
    },
    twoToneColorChange () {
      const { praise, collection } = this.routeSocialStatus
      if (praise) {
        this.praiseTwoTone.color = '#eb2f96'
        this.praiseTwoTone.text = '已点赞'
      } else {
        this.praiseTwoTone.color = ''
        this.praiseTwoTone.text = '点赞'
      }
      if (collection) {
        this.collectionTwoTone.color = '#eb2f96'
        this.collectionTwoTone.text = '已收藏'
      } else {
        this.collectionTwoTone.color = ''
        this.collectionTwoTone.text = '收藏'
      }
    },
    onPraise () {
      const { praise } = this.routeSocialStatus
      this.routeSocial.socialType = 1
      if (praise) {
        // 取消点赞
        this.routeSocialStatus.praise = false
        this.cancelSocial()
      } else {
        // 点赞
        this.routeSocialStatus.praise = true
        this.commitSocial()
      }
    },
    onCollection () {
      const { collection } = this.routeSocialStatus
      this.routeSocial.socialType = 2
      if (collection) {
        // 取消收藏
        this.routeSocialStatus.collection = false
        this.cancelSocial()
      } else {
        // 收藏
        this.routeSocialStatus.collection = true
        this.commitSocial()
      }
    },
    commitSocial () {
      if (this.needLoginIn()) {
        return
      }
      postSocial(this.routeSocial).then(() => {
        this.twoToneColorChange()
      })
    },
    cancelSocial () {
      if (this.needLoginIn()) {
        return
      }
      deleteSocial(this.routeSocial).then(() => {
        this.twoToneColorChange()
      })
    }
  }
}
</script>

<style lang="less" scoped>
</style>
