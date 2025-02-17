<template>
  <v-container>
    <!-- アイテム情報 -->
    <v-row class="justify-center">
      <v-col>
        <div @click="doEdit">
          名前:
          <div v-if="!edit" @click="doEdit" v-text="item.name"></div>
          <v-text-field
            v-if="edit"
            v-model="tempName"
            type="text"
            outlined
            hide-details
            dense
            row-height="1"
          />
          内容:
          <div
            v-if="!edit"
            @click="doEdit"
            v-html="item.content.replace(/\n/g, '<br/>')"
          ></div>
          <v-textarea
            v-if="edit"
            v-model="tempContent"
            type="text"
            auto-grow
            clearable
            rows="2"
            outlined
            hide-details
          />
        </div>
        <div v-if="edit">
          <v-row class="justify-end">
            <v-col>
              <v-btn small @click="updateItem">保存</v-btn>
              <v-btn small @click="doneEdit">キャンセル</v-btn>
            </v-col>
          </v-row>
        </div>
        <v-divider></v-divider>
      </v-col>
    </v-row>

    <!-- QRコード生成ボタン -->
    <v-row class="justify-end">
      <v-col cols="12">
        <v-slider v-model="wid" max="400" min="100" label="サイズ変更"></v-slider>
      </v-col>
      <v-col>
        <v-text-field
          v-model="wid"
          label="サイズ変更"
          dense
          outlined
        ></v-text-field>
      </v-col>
      <v-col>
        <v-btn @click="qrCodeUpdateConfirm">
          QRCodeを
          <span v-if="!item.qr_code">作成</span>
          <span v-else>アップデート</span>
        </v-btn>
        <v-dialog v-model="qrCodeUpdateDialog" max-width="800px">
          <v-card>
            <v-card-title class="text-h5"
              >QRコードが書き変わります。よろしいですか？</v-card-title
            >
            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="blue darken-1" text @click="createQRCode(wid)"
                >OK</v-btn
              >
              <v-btn
                color="blue darken-1"
                text
                @click="qrCodeUpdateDialog = false"
                >キャンセル</v-btn
              >
              <v-spacer></v-spacer>
            </v-card-actions>
          </v-card>
        </v-dialog>
      </v-col>
    </v-row>

    <!-- QRCODE -->
    <v-row class="justify-center">
      <div v-if="item.qr_code" >
        <v-img :src="item.qr_code" :max-height="wid" :max-width="wid" />
      </div>
    </v-row>

    <!-- 落とし物記録一覧 -->
    <v-divider></v-divider>
    <v-data-table
      :headers="headers"
      :items="lostItemInfomationsTemp"
      class="elevation-3"
      disable-sort
      mobile-breakpoint="100"
      :items-per-page="4"
      no-data-text="記録はまだありません"
      :footer-props="{
        itemsPerPageText: pageText,
        itemsPerPageOptions: [3, 5, 10],
      }"
    >
      <template #top>
        <v-toolbar flat>
          <v-toolbar-title>落とし物記録</v-toolbar-title>
          <v-spacer></v-spacer>
          <v-dialog v-model="lostItemInfoDialogDelete" max-width="800px">
            <v-card>
              <v-card-title class="text-h5"
                >記録が削除されます。よろしいですか？</v-card-title
              >
              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn color="blue darken-1" text @click="closeDelete"
                  >キャンセル</v-btn
                >
                <v-btn color="blue darken-1" text @click="deleteItemConfirm"
                  >削除</v-btn
                >
                <v-spacer></v-spacer>
              </v-card-actions>
            </v-card>
          </v-dialog>
        </v-toolbar>
      </template>
      <template #[`item.actions`]="{ item }">
        <v-icon small @click="deleteLostItemInfomation(item)">
          mdi-delete
        </v-icon>
      </template>
      <template #[`item.created_at`]="{ item }">
        {{ item.created_at.slice(0, 4) }}/{{ item.created_at.slice(5, 7) }}/{{
          item.created_at.slice(8, 10)
        }}
      </template>
    </v-data-table>

    <!-- アイテム削除ボタン -->
    <div>
      <v-btn block class="error mt-10" @click="deleteItemDialog = true"
        >削除</v-btn
      >
    </div>
    <v-dialog v-model="deleteItemDialog" max-width="800px">
      <v-card>
        <v-card-title class="text-h5"
          >アイテムが削除されます。よろしいですか？</v-card-title
        >
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="blue darken-1" text @click="deleteItem">OK</v-btn>
          <v-btn color="blue darken-1" text @click="deleteItemDialog = false"
            >キャンセル</v-btn
          >
          <v-spacer></v-spacer>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-container>
</template>
<script>
import QRCode from 'qrcode'

export default {
  data() {
    return {
      qrCode: '',
      isCreated: false,
      wid: 100,
      edit: false,
      tempName: '',
      tempContent: '',
      headers: [
        { text: '日付', value: 'created_at', width: '10%' },
        { text: '削除', value: 'actions', width: '10%' },
        { text: '落とした場所', value: 'found_location', width: '40%' },
        { text: '詳細な場所', value: 'item_destination_details', width: '40%' },
      ],
      dialog: false,
      lostItemInfoDialogDelete: false,
      infoId: 999999,
      lostItemInfomationsTemp: [],
      qrCodeUpdateDialog: false,
      deleteItemDialog: false,
      pageText: '閲覧数',
    }
  },
  async fetch({ store, params }) {
    await store.dispatch('user/getUser')
    await store.dispatch('item/getItem', params.id)
    await store.dispatch(
      'lost_item_infomation/getLostItemInfomations',
      params.id
    )
  },
  computed: {
    item() {
      return this.$store.getters['item/itemGetter']
    },
  },

  mounted() {
    this.$store.commit('isLoggedIn')
    const Item = this.$store.getters['item/itemGetter']
    this.tempName = Item.name
    this.tempContent = Item.content
    this.lostItemInfomationsTemp =
      this.$store.getters['lost_item_infomation/lostItemInfomationsGetter']
  },

  methods: {
    doEdit() {
      this.edit = true
    },
    closeDelete() {
      this.infoId = 999999
      this.lostItemInfoDialogDelete = false
    },
    deleteItemConfirm() {
      this.$store.dispatch(
        'lost_item_infomation/deleteLostItemInfomation',
        this.infoId
      )
      this.$store.dispatch('lost_item_infomation/getLostItemInfomations')
      this.lostItemInfomationsTemp.some((v, i) => {
        if (v.id === this.infoId) this.lostItemInfomationsTemp.splice(i, 1)
        return null
      })
      this.closeDelete()
    },
    deleteLostItemInfomation(info) {
      this.infoId = info.id
      this.lostItemInfoDialogDelete = true
    },
    deleteItem() {
      this.$store.dispatch('item/deleteItem', this.$route.params.id)
      this.$router.push('/items')
    },
    async updateItem() {
      await this.$store.dispatch('item/updateItem', {
        itemId: this.$route.params.id,
        item: { name: this.tempName, content: this.tempContent },
      })
      this.doneEdit()
    },
    doneEdit() {
      this.edit = false
    },
    qrCodeUpdateConfirm() {
      this.qrCodeUpdateDialog = true
    },
    createQRCode(wid) {
      QRCode.toDataURL(
        this.$config.VUE_APP_ATTA_FRONTEND +
          `/lost-items/${this.$route.params.id}/${this._uid}`,
        { width: wid }
      )
        .then(async (qrCode) => {
          // QRcodeをimgタグのsrcに入れる
          this.qrCode = qrCode
          await this.$store.dispatch('item/updateItem', {
            itemId: this.$route.params.id,
            item: {
              qr_code: this.qrCode,
              verification_id: this._uid,
            },
          })
          await this.$store.dispatch('item/getItem', this.$route.params.id)
          this.qrCodeUpdateDialog = false
        })
        .catch((err) => {
          console.error(err)
        })
    },
  },
}
</script>
