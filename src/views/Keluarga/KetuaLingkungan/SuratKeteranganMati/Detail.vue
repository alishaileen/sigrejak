<template>
  <div>
    <btn-kembali path="/keluarga/ketua/surat/surat-keterangan-mati" />
    
    <h1 class="mb-5">Detail Surat Keterangan Kematian</h1>

    <v-row>
      <v-col>
        <v-card flat>
          <v-card-title>
            <h3>Detail Informasi</h3>
            <v-spacer></v-spacer>
            <button-chat
              :countChatUnread="countChatUnread"
              :chatPageUrl="`/keluarga/ketua/surat/surat-keterangan-mati/chat/${data.id}`"
              :detailPageUrl="`/keluarga/ketua/surat/surat-keterangan-mati/detail/${data.id}`"
              :endpointUrl="url"
            ></button-chat>
            <v-chip
              v-if="data.ketua_lingkungan_approval === 1"
              :color="data.ketua_lingkungan_approval === 1 ? 'green' : 'grey lighten-2'"
              class="ml-2"
            >
              <span class="color-white">
                Terverifikasi
              </span>
            </v-chip>

          </v-card-title>

          <v-divider></v-divider>

          <v-card-text class="pa-6">
            <div class="mb-15">
              <h2 class="mb-5">Informasi Surat</h2>
              <v-divider class="mb-5"></v-divider>

              <label>No. surat</label>
              <p>{{ data.no_surat }}</p>

              <label>Tanggal surat</label>
              <p>{{ data.created_at }}</p>
            </div>

            <div class="mb-15">
              <h2 class="mb-5">Informasi Umat</h2>

              <v-divider class="mb-5"></v-divider>

              <label>Nama</label>
              <p>{{ data.nama }}</p>

              <label>Nama baptis</label>
              <p>{{ data.nama_baptis }}</p>

              <label>Tempat, tanggal lahir</label>
              <p>{{ `${data.tempat_lahir}, ${data.tgl_lahir}` }}</p>

              <label>Alamat</label>
              <p>{{ data.alamat }}</p>

              <label>Tempat meninggal</label>
              <p>{{ data.tempat_meninggal }}</p>

              <label>Tanggal meninggal</label>
              <p>{{ data.tgl_meninggal }}</p>

              <label>Tempat pemakaman/kremasi</label>
              <p>{{ data.tempat_makam_kremasi }}</p>

              <label>Tanggal pemakaman/kremasi</label>
              <p>{{ data.tgl_makam_kremasi }}</p>
            </div>

            <div class="mb-15">
              <h2 class="mb-5">Informasi Keluarga</h2>
              
              <v-divider class="mb-5"></v-divider>

              <label>Nama orang tua</label>
              <p>{{ data.nama_ortu }}</p>

              <label>Nama pasangan</label>
              <p>{{ data.nama_pasangan }}</p>
            </div>

            <div class="mb-15">
              <h2 class="mb-5">Sakramen Yang Diterima Sebelum Meninggal</h2>

              <v-divider class="mb-5"></v-divider>

              <ol type="a">
                <h3><li>Viaticum (Komuni)</li></h3>
                <label>Tanggal</label>
                <p>{{ data.tgl_komuni }}</p>

                <label>Nama pelayan sakramen</label>
                <p>{{ data.pelayan_komuni }}</p>


                <h3><li>Pengampunan dosa</li></h3>
                <label>Tanggal</label>
                <p>{{ data.tgl_pengampunan_dosa }}</p>

                <label>Nama pelayan sakramen</label>
                <p>{{ data.pelayan_pengampunan_dosa }}</p>


                <h3><li>Perminyakan orang sakit</li></h3>
                <label>Tanggal</label>
                <p>{{ data.tgl_perminyakan }}</p>

                <label>Nama pelayan sakramen</label>
                <p>{{ data.pelayan_perminyakan }}</p>


                <h3><li>Baptis darurat</li></h3>
                <label>Tanggal</label>
                <p>{{ data.tgl_baptis_darurat }}</p>

                <label>Nama pelayan sakramen</label>
                <p>{{ data.pelayan_baptis_darurat }}</p>
              </ol>
            </div>

            <h2 class="mb-5">Informasi lain</h2>

            <v-divider class="mb-5"></v-divider>

            <label>Imam yang memberkati</label>
            <p>{{ data.nama_imam }}</p>

            <label>Nama pelapor</label>
            <p>{{ data.nama_pelapor }}</p>

            <label>Nomor HP pelapor</label>
            <p>{{ data.no_hp_pelapor }}</p>

            <label>Nomor HP keluarga/penanggung jawab yang bisa dihubungi</label>
            <p>{{ data.no_hp_penanggungjawab }}</p>
          </v-card-text>
          <v-card-actions v-if="data.ketua_lingkungan_approval === 0" class="py-3 px-5">
            <v-spacer></v-spacer>
            <v-btn
              class="btn text-none"
              color="blue accent-4"
              dark
              depressed
              v-if="data.ketua_lingkungan_approval === 0"
              @click="verify"
            >
              Verifikasi
            </v-btn>
          </v-card-actions>
        </v-card>    
      </v-col>
    </v-row>
    <snackbar />
  </div>
</template>

<script>
import { getOneData, changeDateFormat } from '@/utils'
import { verifySurat } from '@/utils/pengurus'
import ButtonChat from '@/components/ButtonChat.vue'

export default {
  components: {
    ButtonChat,
  },
  data: () => ({
    url: '/surat-keterangan-mati',
    data: {},
    countChatUnread: 0,
  }),
  async mounted() {
    this.data = await getOneData(`${this.url}/${this.$route.params.id}`)
    this.data.tgl_lahir = changeDateFormat(this.data.tgl_lahir)
    this.data.tgl_meninggal = changeDateFormat(this.data.tgl_meninggal)
    this.data.tgl_makam_kremasi = changeDateFormat(this.data.tgl_makam_kremasi)
    this.data.tgl_komuni = changeDateFormat(this.data.tgl_komuni)
    this.data.tgl_pengampunan_dosa = changeDateFormat(this.data.tgl_pengampunan_dosa)
    this.data.tgl_perminyakan = changeDateFormat(this.data.tgl_perminyakan)
    this.data.tgl_baptis_darurat = changeDateFormat(this.data.tgl_baptis_darurat)

    // Get jumlah chat yg belum read
    this.countChatUnread = await getOneData(`/chat/count-unread/${this.$route.params.id}`)
    this.countChatUnread = this.countChatUnread.count_unread
  },
  computed: {
    isVerifyDisabled() {
      return this.data.ketua_lingkungan_approval ? true : false
    }
  },
  methods: {
    async verify() {
      let snackbar
      
      this.$store.dispatch('loading/openLoading')
      this.$store.commit('snackbar/resetSnackbar')

      let verifikasi = {
        role: 'ketua lingkungan',
        ketua_lingkungan: this.$store.state.keluarga.nama_kepala_keluarga
      }
      snackbar = await verifySurat(this.url, this.data.id, verifikasi)

      if (snackbar.color === 'success') {
        this.$router.push('/keluarga/ketua/surat/surat-keterangan-mati')
      }

      this.data = await getOneData(`${this.url}/${this.$route.params.id}`)

      this.$store.dispatch('snackbar/openSnackbar', snackbar)
      this.$store.dispatch('loading/closeLoading')
    },
  }
}
</script>

<style>

</style>