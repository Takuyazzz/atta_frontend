<template>
  <v-container>
    <Notification
      :msg="dlg.msg"
      :is-open="dlg.isOpen"
      :type="dlg.type"
    ></Notification>
    <v-row>
      <v-col cols="12">
        <v-card>
          <v-card-subtitle>落とし物詳細情報</v-card-subtitle>持ち主:
          <div v-text="lost_item.user.name"></div>
          落とし物:
          <div v-text="lost_item.item.name"></div>
          落とし物詳細:
          <div v-html="lost_item.item.content.replace(/\n/g, '<br/>')"></div>
        </v-card>
      </v-col>
    </v-row>

    <ValidationObserver ref="lostItemForm">
      <v-form @submit.prevent="submit">
        <!-- 見つけた場所用のマップ -->
        <v-dialog v-model="dialog1" max-width="600px">
          <template #activator="{ on, attrs }">
            <v-row row justify-center align="center">
              <v-col>
                <ValidationProvider
                  v-slot="{ errors }"
                  name="見つけた場所"
                  rules="required"
                >
                  <v-text-field
                    v-model="foundLocation"
                    prepend-icon="mdi-map-marker"
                    label="1.見つけた場所"
                    class="centered-input"
                    placeholder="(例)〇〇のマクドナルド... "
                    :hint="foundLocation"
                    persistent-hint
                    required
                    :error-messages="errors"
                  >
                    <template #prepend>
                      <v-btn
                        icon
                        color="primary"
                        v-bind="attrs"
                        rounded
                        small
                        @click="showMapAction1"
                        v-on="on"
                        ><v-icon>mdi-map-search-outline</v-icon></v-btn
                      >
                    </template>
                  </v-text-field>
                </ValidationProvider>
              </v-col>
            </v-row>
          </template>
          <div>
            <v-row justify="center" align="center">
              <v-col cols="12">
                <v-card>
                  <v-card-text>
                    Mapから場所を指定することもできます。<br />
                    クリックorタッチすると指定できます。
                  </v-card-text>
                  <GmapMap
                    map-type-id="roadmap"
                    :center="maplocation1"
                    :zoom="zoom"
                    :style="styleMap"
                    :options="mapOptions"
                    @click="placeMarker1($event)"
                  >
                    <GmapMarker
                      :title="marker1.title"
                      :position="marker1.position"
                      :clickable="true"
                      :draggable="false"
                      :icon="marker1.pinicon"
                    />
                  </GmapMap>
                </v-card>
              </v-col>
            </v-row>
          </div>
        </v-dialog>

        <!-- 現在地ボタン -->
        <v-row justify="end">
          <v-btn @click="inputCurrentAddress1"
            ><v-icon left>mdi-map-marker-radius-outline</v-icon
            >現在地から入力</v-btn
          >
        </v-row>

        <!-- 落とし物現在地欄 -->
        <v-dialog v-model="dialog2" max-width="600px">
          <template #activator="{ on, attrs }">
            <v-row row justify-center align="center">
              <v-col>
                <ValidationProvider
                  v-slot="{ errors }"
                  name="見つけた場所"
                  rules="required"
                >
                  <v-text-field
                    v-model="itemDestination"
                    label="2.落とし物の現在地"
                    class="centered-input"
                    placeholder="(例)〇〇の交番,,,見つけた場所にある"
                    :hint="itemDestination"
                    persistent-hint
                    required
                    :error-messages="errors"
                  >
                    <template #prepend>
                      <v-btn
                        icon
                        color="primary"
                        v-bind="attrs"
                        small
                        @click="showMapAction2"
                        v-on="on"
                        ><v-icon>mdi-map-search-outline</v-icon></v-btn
                      >
                    </template>
                  </v-text-field>
                </ValidationProvider>
              </v-col>
            </v-row>
          </template>
          <div>
            <v-row justify="center" align="center">
              <v-col cols="12">
                <v-card>
                  <v-card-text>
                    Mapから場所を指定することもできます。<br />
                    クリックorタッチすると指定できます。
                  </v-card-text>
                  <GmapMap
                    map-type-id="roadmap"
                    :center="maplocation2"
                    :zoom="zoom"
                    :style="styleMap"
                    :options="mapOptions"
                    @click="placeMarker2($event)"
                  >
                    <GmapMarker
                      :title="marker2.title"
                      :position="marker2.position"
                      :clickable="true"
                      :draggable="false"
                      :icon="marker2.pinicon"
                    />
                  </GmapMap>
                </v-card>
              </v-col>
            </v-row>
          </div>
        </v-dialog>

        <!-- 見つけた場所に同じボタン -->
        <v-row justify="end">
          <v-btn @click="sameWithCurrentAddress"
            ><v-icon left>mdi-map-marker-multiple-outline</v-icon
            >見つけた場所に同じ</v-btn
          >
        </v-row>

        <!-- 落とし物の現在地入力欄 -->
        <v-row row justify-center align-center>
          <v-col class="justify-center">
            <ValidationProvider
              v-slot="{ errors }"
              name="落とし物の詳細位置情報"
              rules="required"
            >
              <v-textarea
                v-model="itemDestinationDetails"
                label="3.落とし物の詳細位置情報"
                class="centered-input"
                hint="具体的な発見場所を書いてください"
                persistent-hint
                auto-grow
                clearable
                rows="3"
                outlined
                placeholder="(例)マクドナルドの２階の窓側席...〇〇の近くにあるよ"
                required
                :error-messages="errors"
              />
            </ValidationProvider>
          </v-col>
        </v-row>

        <!-- 持ち主に知らせるボタン -->
        <v-row justify="center">
          <v-col cols="1">
            <v-btn color="primary" type="submit"
              ><v-icon left>mdi-account-arrow-left</v-icon
              >持ち主に知らせる!</v-btn
            >
          </v-col>
        </v-row>
      </v-form>
    </ValidationObserver>
  </v-container>
</template>
<script>
export default {
  components: {},
  data() {
    return {
      foundLocation: '',
      itemDestination: '',
      maplocation1: { lng: 0, lat: 0 },
      maplocation2: { lng: 0, lat: 0 },
      currentLocation1: { lng: 0, lat: 0 },
      currentLocation2: { lng: 0, lat: 0 },
      zoom: 6,
      styleMap: {
        width: '100%',
        height: '400px',
      },
      mapOptions: {
        streetViewControl: false,
        styles: [],
      },
      marker1: {
        title: '見つけた場所',
        position: { lat: 0, lng: 0 },
      },
      marker2: {
        title: '落とし物の現在地',
        position: { lat: 0, lng: 0 },
      },
      showMap: false,
      itemDestinationDetails: '',
      dlg: {
        msg: '',
        isOpen: false,
        mode: '',
      },
      dialog1: false,
      dialog2: false,
      item: {},
    }
  },
  async fetch({ store, redirect }) {
    const Key = localStorage.getItem('item_referer')
    const ItemId = await JSON.parse(Key)
    if (Key == null || ItemId == null) {
      redirect('/')
    }
    await store.dispatch('lost_item/getLostItem', ItemId)
  },
  computed: {
    lost_item() {
      return this.$store.getters['lost_item/lostItemDataGetter']
    },
  },
  destroyed() {
    localStorage.removeItem('item_referer')
  },
  mounted() {
    this.$store.commit('setShowUserInfo', false)
  },
  methods: {
    async showMapAction1() {
      if (
        // 以前に位置情報を取得したことがあるか
        this.currentLocation1.lat !== 0 ||
        this.currentLocation1.lng !== 0
      ) {
        // ある
        this.maplocation1 = this.currentLocation1
      } else if (
        // 位置情報があるか
        this.maplocation1.lat === 0 ||
        this.maplocation1.lng === 0
      ) {
        // ない
        const currentPosTmp = await this.getCurrentPosition()
        const currentPos = {
          lat: currentPosTmp.coords.latitude,
          lng: currentPosTmp.coords.longitude,
        }
        this.maplocation1 = currentPos
        this.currentLocation1 = currentPos
      }
      this.showMap = true
    },
    async showMapAction2() {
      if (
        // 以前に位置情報を取得したことがあるか
        this.currentLocation2.lat !== 0 ||
        this.currentLocation2.lng !== 0
      ) {
        // ある
        this.maplocation2 = this.currentLocation2
      } else if (
        // 位置情報があるか
        this.maplocation2.lat === 0 ||
        this.maplocation2.lng === 0
      ) {
        // ない
        const currentPosTmp = await this.getCurrentPosition()
        const currentPos = {
          lat: currentPosTmp.coords.latitude,
          lng: currentPosTmp.coords.longitude,
        }
        this.maplocation2 = currentPos
        this.currentLocation2 = currentPos
      }
      this.showMap = true
    },
    closeMapAction() {
      // バックエンドに送らないようにするために、undefind
      this.maplocation1.lat = 0
      this.maplocation1.lng = 0
      this.showMap = false
    },
    async inputCurrentAddress1() {
      const CurrentPosTmp = await this.getCurrentPosition()
      const CurrentPos = {
        lat: CurrentPosTmp.coords.latitude,
        lng: CurrentPosTmp.coords.longitude,
      }
      const Address = await this.getAddress(CurrentPos)
      this.foundLocation = Address
      this.maplocation1 = CurrentPos
      this.marker1.position = CurrentPos
    },
    async inputCurrentAddress2() {
      const CurrentPosTmp = await this.getCurrentPosition()
      const CurrentPos = {
        lat: CurrentPosTmp.coords.latitude,
        lng: CurrentPosTmp.coords.longitude,
      }
      const Address = await this.getAddress(CurrentPos)
      this.itemDestination = Address
      this.maplocation2 = CurrentPos
      this.marker2.position = CurrentPos
    },
    async placeMarker1(event) {
      if (event) {
        this.maplocation1.lat = event.latLng.lat()
        this.maplocation1.lng = event.latLng.lng()
      }
      this.foundLocation = await this.getAddress(this.maplocation1)
      this.marker1.position = this.maplocation1
    },
    async placeMarker2(event) {
      if (event) {
        this.maplocation2.lat = event.latLng.lat()
        this.maplocation2.lng = event.latLng.lng()
      }
      this.itemDestination = await this.getAddress(this.maplocation2)
      this.marker2.position = this.maplocation2
    },
    async getAddress(location) {
      let requestURL =
        'https://maps.googleapis.com/maps/api/geocode/json?language=ja&sensor=false'
      requestURL += '&latlng=' + location.lat + ',' + location.lng
      requestURL += `&key=${this.$config.VUE_APP_GOOGLE_API_KEY}`
      // + process.env.VUE_APP_GOOGLE_API_KEY
      const Response = await this.$axios.$get(requestURL)
      return Response.results[0].formatted_address
    },
    getCurrentPosition() {
      return new Promise(function (resolve, reject) {
        navigator.geolocation.getCurrentPosition(resolve, reject)
      })
    },
    sameWithCurrentAddress() {
      this.maplocation2 = this.maplocation1
      this.marker2.position = this.marker1.position
      this.itemDestination = this.foundLocation
    },
    submit() {
      this.$refs.lostItemForm.validate().then(async (success) => {
        if (!success) {
          return null
        }
        const Response = await this.$store.dispatch(
          'lost_item/postNotification',
          {
            lostItemData: {
              foundLocation: this.foundLocation,
              itemDestination: this.itemDestination,
              itemDestinationDetails: this.itemDestinationDetails,
              mapLocationLat: this.maplocation2.lat,
              mapLocationLng: this.maplocation2.lng,
              email: this.lost_item.user.email,
              user_name: this.lost_item.user.name,
              item_name: this.lost_item.item.name,
            },
          }
        )
        if (Response.notificated) {
          const Key = localStorage.getItem('item_referer')
          const ItemID = await JSON.parse(Key)
          await this.$store.dispatch('lost_item/updateItem', {
            itemId: ItemID,
            item: {
              isValid: false,
            },
          })
          await this.$store.dispatch(
            'lost_item_infomation/postLostItemInfomation',
            {
              lostItemInfomation: {
                item_id: ItemID,
                found_location: this.foundLocation,
                item_destination_details: this.itemDestinationDetails,
              },
            }
          )
          this.dlg.isOpen = true
          this.dlg.msg =
            '持ち主へ通知が完了しました! ご協力ありがとうございます！'
          this.dlg.type = 'success'
          setTimeout(() => {
            this.$router.push('/')
          }, 5000)
        } else {
          this.dlg.isOpen = true
          this.dlg.msg = 'メールを送れませんでした. もう一度お試しください'
          this.dlg.type = 'error'
          setTimeout(() => {
            this.dlg.isOpen = false
          }, 5000)
        }
      })
    },
  },
}
</script>
