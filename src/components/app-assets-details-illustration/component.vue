<style lang="scss">
@include b(assets-details-illustration) {
}
</style>

<template>
  <div
    class="app-assets-details-illustration px-5 py-2"
    flex="dir:left box:first">
    <div class="mr-5">
      <v-img :aspect-ratio="1" :src="fileSvg | https" width="200px" contain/>
    </div>
    <v-card flat>
      <v-list-item>
        <app-avatar
          :avatar="createrAvatar"
          :name="createrNickname"
          :size="50"
          @click="onAvatarClick"
          class="mr-4"/>
        <v-list-item-content>
          <v-list-item-title class="headline">{{ createrNickname }}</v-list-item-title>
          <v-list-item-subtitle>更新日期：{{ displayUpdatetime }}</v-list-item-subtitle>
        </v-list-item-content>
      </v-list-item>
      <v-card-text>
        <p class="text--primary">"{{ displayImageName }}"</p>
        <p>所属插画库</p>
        <p>
          <v-btn
            v-for="collection of collections"
            :key="collection.id"
            small
            depressed
            color="primary">
            {{ collection.name }}
          </v-btn>
        </p>
      </v-card-text>
      <v-card-actions>
        <v-btn depressed @click="onClickDownloadSvg">SVG 下载</v-btn>
        <v-btn depressed @click="onClickDownloadPng">PNG 下载</v-btn>
        <v-btn depressed @click="onClickCopySvg">复制 SVG</v-btn>
      </v-card-actions>
    </v-card>
  </div>
</template>

<script>
import dayjs from 'dayjs'
import { get } from 'lodash-es'
import { mapState, mapActions } from 'vuex'

export default {
  name: 'app-assets-details-illustration',
  inject: {
    assetsDetailsDialogClose: {
      default: () => {}
    }
  },
  props: {
    value: {
      type: Object,
      default: () => ({})
    },
    idKey: {
      type: String,
      default: 'id'
    }
  },
  data () {
    return {
      info: {}
    }
  },
  computed: {
    ...mapState('sdk', [
      'sdk'
    ]),
    displayImageName () {
      return get(this.info, 'name', '') || get(this.value, 'name', '')
    },
    displayUpdatetime () {
      return dayjs(get(this.info, 'updated_at', '') || get(this.value, 'updated_at', '')).format('YYYY-MM-DD')
    },
    createrAvatar () {
      return get(this.info, 'creater.avatar', '')
    },
    createrNickname () {
      return get(this.info, 'creater.nickname', '')
    },
    createrId () {
      return get(this.info, 'creater.id', 0)
    },
    collections () {
      return get(this.info, 'collections', [])
    },
    fileSvg () {
      return get(this.info, 'origin_file', '') || get(this.value, 'origin_file', '')
    },
    filePng () {
      return get(this.info, 'file', '') || get(this.value, 'file', '')
    }
  },
  created () {
    this.fetch()
  },
  methods: {
    ...mapActions('download', [
      'downloadIllustrations'
    ]),
    async fetch () {
      this.info = await this.sdk.svgInfo({ id: this.value[this.idKey] })
    },
    async onClickDownloadSvg () {
      this.downloadIllustrations([{
        url: this.fileSvg,
        name: this.displayImageName
      }, {
        url: this.fileSvg,
        name: this.displayImageName
      }, {
        url: this.fileSvg,
        name: this.displayImageName
      }, {
        url: this.fileSvg,
        name: this.displayImageName
      }])
    },
    async onClickDownloadPng () {
      this.downloadIllustrations([{
        url: this.filePng,
        name: this.displayImageName
      }])
    },
    async onClickCopySvg () {
      this.$clipboard.writeText(await this.sdk.getFile(this.fileSvg))
      this.$toasted.global.app_success({ message: '复制到剪贴板' })
    },
    onAvatarClick () {
      this.assetsDetailsDialogClose()
      this.$go.user.detail(this.createrId)
    }
  }
}
</script>
